# PARTIE 1A — DART
# CHAPITRE 14 — COLLECTIONS ET PROGRAMMATION FONCTIONNELLE

> **Niveau :** intermédiaire
> **Durée estimée :** 6 h
> **Pré-requis :** chapitre 05 (boucles), chapitre 06 (collections), chapitre 07 (fonctions), chapitre 13 (exceptions)
> **Ce que vous saurez faire à la fin :** transformer, filtrer, trier et résumer n'importe quelle liste de joueurs, d'ennemis ou d'objets d'inventaire en quelques lignes déclaratives, sans écrire une seule boucle `for`.

---

## 14.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- distinguer le style impératif (comment faire) du style déclaratif (quoi obtenir) ;
- écrire une fonction anonyme et une fonction fléchée pour les passer à une méthode ;
- parcourir une collection avec `forEach()` ;
- transformer une collection avec `map()`, y compris en changeant de type ;
- expliquer ce qu'est un `Iterable` et pourquoi `toList()` est souvent obligatoire ;
- filtrer une collection avec `where()` ;
- rechercher un élément avec `firstWhere()`, `lastWhere()`, `singleWhere()` et `indexWhere()` ;
- répondre à une question par oui ou non avec `any()`, `every()` et `contains()` ;
- réduire une collection à une seule valeur avec `reduce()` et `fold()` ;
- choisir entre `reduce()` et `fold()` en connaissance de cause ;
- trier une liste avec `sort()` et écrire un comparateur ;
- utiliser `compareTo()` sur des nombres et sur des textes ;
- trier des objets par score, par nom, puis par plusieurs critères ;
- trier une copie sans modifier la liste d'origine ;
- découper une collection avec `take()` et `skip()` ;
- aplatir des listes imbriquées avec `expand()` ;
- dédoublonner avec `toSet()` ;
- assembler une collection en texte avec `join()` ;
- utiliser le spread operator `...` et sa version nullable `...?` ;
- écrire un `if` et un `for` directement à l'intérieur d'une liste littérale ;
- chaîner plusieurs opérations en une seule expression lisible ;
- décider, avec des critères clairs, quand il vaut mieux revenir à une boucle `for`.

---

## 14.1 — Le style impératif (boucle `for`) vs le style déclaratif

Jusqu'ici, pour travailler sur une collection, vous avez toujours fait la même chose : créer une liste vide, la remplir dans une boucle, puis l'utiliser.

Prenons une équipe de joueurs et leurs scores. On veut la liste des scores doublés (un événement « double XP » dans le jeu).

Voici la version **impérative**, celle que vous connaissez :

```dart
void main() {
  List<int> scores = [120, 80, 340, 55];

  List<int> doubles = [];
  for (int i = 0; i < scores.length; i++) {
    doubles.add(scores[i] * 2);
  }

  print(doubles);
}
```

**Résultat :**

```text
[240, 160, 680, 110]
```

Ce code fonctionne. Mais lisez-le à voix haute et remarquez tout ce qu'il vous oblige à raconter :

```text
  1. crée une liste vide
  2. crée un compteur i qui vaut 0
  3. tant que i est plus petit que la longueur
  4. prends l'élément à l'indice i
  5. multiplie-le par 2
  6. ajoute le résultat à la liste
  7. augmente i de 1
  8. recommence
```

Sept des huit étapes parlent de **mécanique** (compteur, indice, ajout). Une seule parle de votre intention réelle : *multiplier par 2*.

C'est cela, le style impératif : vous décrivez **COMMENT** la machine doit s'y prendre.

Voici maintenant la version **déclarative** :

```dart
void main() {
  List<int> scores = [120, 80, 340, 55];

  List<int> doubles = scores.map((s) => s * 2).toList();

  print(doubles);
}
```

**Résultat :**

```text
[240, 160, 680, 110]
```

Le résultat est identique. La phrase, elle, a changé :

```text
  « à partir des scores, associe à chacun son double, et donne-moi la liste »
```

Il n'y a plus de compteur, plus d'indice, plus de liste vide à préparer. Vous décrivez **QUOI** vous voulez obtenir, et Dart s'occupe de la mécanique.

Comparons les deux approches :

```text
  ┌──────────────────────┬───────────────────────────────────────┐
  │ Style impératif      │ Style déclaratif                      │
  ├──────────────────────┼───────────────────────────────────────┤
  │ for, while           │ map, where, fold...                   │
  │ décrit COMMENT       │ décrit QUOI                           │
  │ compteur visible     │ aucun compteur                        │
  │ liste vide à remplir │ la liste est le résultat de l'appel   │
  │ erreurs d'indice     │ pas d'indice, donc pas d'erreur       │
  │ possibles            │ d'indice                              │
  └──────────────────────┴───────────────────────────────────────┘
```

Ce chapitre vous apprend le second style. Attention toutefois : le premier ne devient pas mauvais pour autant. La section 14.29 vous donnera des critères précis pour choisir.

> **Vocabulaire :** on appelle ce style « programmation fonctionnelle » parce qu'on passe des **fonctions** en argument à d'autres fonctions. Vous savez déjà faire cela depuis le chapitre 07.

---

## 14.2 — Rappel : fonction anonyme et fonction fléchée

Toutes les méthodes de ce chapitre ont un point commun : elles attendent **une fonction** en argument.

Une fonction ordinaire porte un nom :

```dart
int doubler(int valeur) {
  return valeur * 2;
}

void main() {
  print(doubler(120));
}
```

**Résultat :**

```text
240
```

Mais quand une fonction ne sert qu'une seule fois, à un seul endroit, lui donner un nom est inutile. On écrit alors une **fonction anonyme** : le même corps, sans nom.

```dart
void main() {
  // fonction anonyme rangée dans une variable
  var doubler = (int valeur) {
    return valeur * 2;
  };

  print(doubler(120));
}
```

**Résultat :**

```text
240
```

La syntaxe est celle d'une fonction normale à laquelle on a retiré le nom et le type de retour :

```text
  int doubler(int valeur) { return valeur * 2; }
  ───┬─── ──┬───  ──┬───    ────────┬─────────
  type    nom   paramètre         corps

        (int valeur) { return valeur * 2; }
        ──────┬─────   ────────┬──────────
          paramètre           corps        <- fonction anonyme
```

Quand le corps se résume à **un seul `return`**, Dart propose un raccourci : la **fonction fléchée** avec `=>`.

```dart
void main() {
  var doubler = (int valeur) => valeur * 2;

  print(doubler(120));
}
```

**Résultat :**

```text
240
```

`=>` signifie exactement `{ return ... ; }`. Retenez cette équivalence :

```text
  (x) { return x * 2; }      ===      (x) => x * 2
```

Dernier point, essentiel pour la suite : quand vous passez une fonction anonyme à `map()` ou `where()`, **le type du paramètre est deviné par Dart**. Vous n'avez pas besoin de l'écrire.

```dart
void main() {
  List<int> scores = [120, 80, 340];

  // les trois écritures suivantes sont équivalentes
  print(scores.map((int s) { return s * 2; }).toList());
  print(scores.map((s) { return s * 2; }).toList());
  print(scores.map((s) => s * 2).toList());
}
```

**Résultat :**

```text
[240, 160, 680]
[240, 160, 680]
[240, 160, 680]
```

Puisque `scores` est une `List<int>`, Dart sait que `s` est un `int`. C'est ce qu'on appelle l'inférence de type.

Dans tout le reste du chapitre, on utilisera la forme fléchée quand elle suffit, et la forme à accolades quand il faut plusieurs instructions.

> **Remarque :** le nom du paramètre est libre. `(s)`, `(score)`, `(e)`, `(joueur)` : choisissez le nom le plus parlant. Un nom d'une lettre est acceptable pour une expression très courte, pas au-delà.

---

## 14.3 — `forEach()`

`forEach()` parcourt une collection et exécute une fonction **pour chaque** élément. Il ne produit aucun résultat : on l'utilise uniquement pour son effet (afficher, enregistrer, modifier un objet).

Version avec une boucle `for` :

```dart
void main() {
  List<String> inventaire = ['Potion', 'Épée', 'Bouclier'];

  for (int i = 0; i < inventaire.length; i++) {
    print('Objet : ${inventaire[i]}');
  }
}
```

**Résultat :**

```text
Objet : Potion
Objet : Épée
Objet : Bouclier
```

Version fonctionnelle équivalente :

```dart
void main() {
  List<String> inventaire = ['Potion', 'Épée', 'Bouclier'];

  inventaire.forEach((objet) => print('Objet : $objet'));
}
```

**Résultat :**

```text
Objet : Potion
Objet : Épée
Objet : Bouclier
```

Le paramètre `objet` reçoit successivement chaque valeur de la liste. Vous n'écrivez ni compteur, ni condition d'arrêt, ni `inventaire[i]`.

Si le traitement demande plusieurs lignes, utilisez la forme à accolades :

```dart
void main() {
  List<int> degats = [12, 40, 7];

  degats.forEach((d) {
    String niveau = d >= 30 ? 'CRITIQUE' : 'normal';
    print('Dégâts $d -> $niveau');
  });
}
```

**Résultat :**

```text
Dégâts 12 -> normal
Dégâts 40 -> CRITIQUE
Dégâts 7 -> normal
```

Un point important : `forEach()` ne **retourne rien** (son type de retour est `void`). Vous ne pouvez donc pas l'utiliser pour construire une nouvelle liste. Ce code est une erreur :

```dart
// NE COMPILE PAS
List<int> doubles = scores.forEach((s) => s * 2);
```

**Résultat :**

```text
Error: A value of type 'void' can't be assigned to a variable of type 'List<int>'.
```

Pour construire une nouvelle collection, la bonne méthode est `map()`, vue à la section suivante.

> **Conseil de professionnel :** en Dart, la boucle `for (var x in liste)` est souvent préférée à `forEach()`. Elle est aussi lisible, et surtout elle autorise `break`, `continue` et `await`, ce que `forEach()` ne permet pas. Retenez `forEach()` parce que vous le lirez dans du code existant, mais ne le considérez pas comme la forme moderne par défaut.

Pour mémoire, la boucle `for-in` équivalente :

```dart
void main() {
  List<String> inventaire = ['Potion', 'Épée', 'Bouclier'];

  for (final objet in inventaire) {
    print('Objet : $objet');
  }
}
```

**Résultat :**

```text
Objet : Potion
Objet : Épée
Objet : Bouclier
```

---

## 14.4 — `map()`

`map()` est la méthode la plus utilisée du chapitre. Elle **transforme** chaque élément et produit une nouvelle collection de même taille.

```text
  [120, 80, 340]        liste de départ
     │    │    │
     ×2   ×2   ×2       la fonction est appliquée à chaque élément
     │    │    │
     v    v    v
  [240, 160, 680]       nouvelle collection, MÊME nombre d'éléments
```

Version avec une boucle `for` :

```dart
void main() {
  List<int> scoresBruts = [120, 80, 340];

  List<int> scoresBonus = [];
  for (final s in scoresBruts) {
    scoresBonus.add(s + 50);
  }

  print(scoresBonus);
}
```

**Résultat :**

```text
[170, 130, 390]
```

Version fonctionnelle équivalente :

```dart
void main() {
  List<int> scoresBruts = [120, 80, 340];

  List<int> scoresBonus = scoresBruts.map((s) => s + 50).toList();

  print(scoresBonus);
}
```

**Résultat :**

```text
[170, 130, 390]
```

Trois faits à retenir sur `map()` :

1. La collection de départ n'est **jamais modifiée**. On obtient une nouvelle collection.
2. Le nombre d'éléments est **toujours conservé**. Trois éléments en entrée, trois en sortie.
3. `map()` ne renvoie pas une `List` mais un `Iterable`. D'où le `.toList()` final, expliqué en 14.6.

Vérifions le point 1 :

```dart
void main() {
  List<String> ennemis = ['gobelin', 'orc', 'troll'];

  List<String> cris = ennemis.map((e) => e.toUpperCase()).toList();

  print('origine : $ennemis');
  print('cris    : $cris');
}
```

**Résultat :**

```text
origine : [gobelin, orc, troll]
cris    : [GOBELIN, ORC, TROLL]
```

`ennemis` est intact. C'est une propriété fondamentale du style fonctionnel : on ne détruit pas les données de départ, on en dérive de nouvelles.

Un dernier exemple, avec une transformation un peu plus riche :

```dart
void main() {
  List<String> ennemis = ['gobelin', 'orc', 'troll'];

  List<String> etiquettes = ennemis
      .map((e) => '[${e[0].toUpperCase()}] $e')
      .toList();

  etiquettes.forEach(print);
}
```

**Résultat :**

```text
[G] gobelin
[O] orc
[T] troll
```

> **Remarque :** `etiquettes.forEach(print)` passe directement la fonction `print` sans écrire `(x) => print(x)`. C'est possible parce que `print` accepte exactement un argument. On appelle cela une référence de fonction (« tear-off »).

---

## 14.5 — `map()` et changement de type

`map()` n'est pas obligée de rendre le même type que celui d'entrée. C'est même son plus grand intérêt : elle sait passer d'un `List<int>` à un `List<String>`, ou d'un `List<Joueur>` à un `List<int>`.

Premier cas : des `int` vers des `String`.

Version avec une boucle `for` :

```dart
void main() {
  List<int> vies = [3, 1, 5];

  List<String> affichage = [];
  for (final v in vies) {
    affichage.add('$v vie(s)');
  }

  print(affichage);
}
```

**Résultat :**

```text
[3 vie(s), 1 vie(s), 5 vie(s)]
```

Version fonctionnelle équivalente :

```dart
void main() {
  List<int> vies = [3, 1, 5];

  List<String> affichage = vies.map((v) => '$v vie(s)').toList();

  print(affichage);
}
```

**Résultat :**

```text
[3 vie(s), 1 vie(s), 5 vie(s)]
```

Deuxième cas, beaucoup plus utile en pratique : extraire un champ d'une liste d'objets.

```dart
class Joueur {
  final String nom;
  final int score;

  Joueur(this.nom, this.score);
}

void main() {
  List<Joueur> equipe = [
    Joueur('Alex', 120),
    Joueur('Bilal', 340),
    Joueur('Chloé', 80),
  ];

  // List<Joueur> -> List<String>
  List<String> noms = equipe.map((j) => j.nom).toList();

  // List<Joueur> -> List<int>
  List<int> scores = equipe.map((j) => j.score).toList();

  print(noms);
  print(scores);
}
```

**Résultat :**

```text
[Alex, Bilal, Chloé]
[120, 340, 80]
```

Ce schéma est à connaître par cœur : « j'ai une liste d'objets, je veux la liste d'un seul de leurs champs ».

```text
  equipe : List<Joueur>
  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐
  │ nom  : Alex   │  │ nom  : Bilal  │  │ nom  : Chloé  │
  │ score: 120    │  │ score: 340    │  │ score: 80     │
  └───────────────┘  └───────────────┘  └───────────────┘
          │ (j) => j.score    │                  │
          v                   v                  v
        120                 340                 80
  scores : List<int>
```

On peut aussi aller dans l'autre sens : partir de données brutes et **construire** des objets.

```dart
class Ennemi {
  final String nom;
  final int pv;

  Ennemi(this.nom, this.pv);

  @override
  String toString() => '$nom($pv pv)';
}

void main() {
  List<String> nomsBruts = ['gobelin', 'orc', 'troll'];

  List<Ennemi> ennemis =
      nomsBruts.map((n) => Ennemi(n, n.length * 10)).toList();

  print(ennemis);
}
```

**Résultat :**

```text
[gobelin(70 pv), orc(30 pv), troll(50 pv)]
```

Si vous voulez être explicite sur le type produit, vous pouvez l'écrire entre chevrons :

```dart
void main() {
  List<int> scores = [120, 340, 80];

  var texte = scores.map<String>((s) => 'Score : $s').toList();

  print(texte.runtimeType);
  print(texte);
}
```

**Résultat :**

```text
List<String>
[Score : 120, Score : 340, Score : 80]
```

> **Remarque :** `map<String>(...)` n'est pas obligatoire ici, Dart déduit seul le type. Cette écriture devient utile quand l'inférence donne un type trop large, par exemple `Object` au lieu de `String`.

---

## 14.6 — `toList()` et l'évaluation paresseuse (`Iterable`)

Voici le point qui déroute tous les débutants. Regardez ce que renvoie `map()` **sans** `toList()` :

```dart
void main() {
  List<int> scores = [120, 80, 340];

  var resultat = scores.map((s) => s * 2);

  print(resultat);
  print(resultat.runtimeType);
}
```

**Résultat :**

```text
(240, 160, 680)
MappedListIterable<int, int>
```

Deux surprises :

- l'affichage utilise des **parenthèses** `( )` et non des crochets `[ ]` ;
- le type n'est pas `List<int>` mais `MappedListIterable`.

`map()`, `where()`, `take()`, `skip()` et `expand()` ne rendent pas une liste : elles rendent un **`Iterable`**.

Un `Iterable` est une **recette**, pas un plat. Il décrit un calcul à faire, mais ne le fait pas tant que personne ne demande les valeurs. On appelle cela l'**évaluation paresseuse** (lazy evaluation).

```text
  scores.map((s) => s * 2)

  ┌──────────────────────────────────────────┐
  │  Iterable = « recette »                  │
  │  source : [120, 80, 340]                 │
  │  règle  : multiplier chaque élément par 2│
  │  valeurs calculées : AUCUNE pour l'instant│
  └──────────────────────────────────────────┘
              │
              │  .toList()  <- on exécute la recette
              v
  ┌──────────────────────────────────────────┐
  │  List = « plat servi »                   │
  │  [240, 160, 680]  (en mémoire)           │
  └──────────────────────────────────────────┘
```

La démonstration la plus parlante consiste à mettre un `print` dans la fonction de transformation :

```dart
void main() {
  List<int> scores = [120, 80, 340];

  print('--- avant map ---');
  var recette = scores.map((s) {
    print('  calcul de $s');
    return s * 2;
  });
  print('--- après map ---');

  print('--- avant toList ---');
  List<int> plat = recette.toList();
  print('--- après toList ---');

  print(plat);
}
```

**Résultat :**

```text
--- avant map ---
--- après map ---
--- avant toList ---
  calcul de 120
  calcul de 80
  calcul de 340
--- après toList ---
[240, 160, 680]
```

Regardez bien : **aucun calcul n'a lieu à l'appel de `map()`**. Tout se déclenche à `toList()`.

Cette paresse a une conséquence piégeuse : un `Iterable` non stocké est **recalculé à chaque parcours**.

```dart
void main() {
  List<int> scores = [10, 20];

  var recette = scores.map((s) {
    print('  calcul de $s');
    return s * 2;
  });

  print('premier parcours :');
  print(recette.toList());

  print('second parcours :');
  print(recette.toList());
}
```

**Résultat :**

```text
premier parcours :
  calcul de 10
  calcul de 20
[20, 40]
second parcours :
  calcul de 10
  calcul de 20
[20, 40]
```

Les calculs sont refaits. Si la transformation est coûteuse, c'est du temps perdu.

**Règle pratique :** appelez `toList()` dès que vous voulez :

- stocker le résultat dans une variable de type `List` ;
- parcourir le résultat plusieurs fois ;
- accéder par indice (`resultat[0]`) ;
- appeler `sort()`, `add()` ou `removeAt()`.

Un `Iterable` ne connaît pas l'accès par indice :

```dart
// NE COMPILE PAS
var r = scores.map((s) => s * 2);
print(r[0]);
```

**Résultat :**

```text
Error: The operator '[]' isn't defined for the class 'Iterable<int>'.
```

Avec `toList()`, tout rentre dans l'ordre :

```dart
void main() {
  List<int> scores = [120, 80, 340];

  List<int> doubles = scores.map((s) => s * 2).toList();

  print(doubles[0]);
  print(doubles.length);
  doubles.add(999);
  print(doubles);
}
```

**Résultat :**

```text
240
3
[240, 160, 680, 999]
```

Il existe aussi `toSet()` (voir 14.23) pour obtenir un `Set` au lieu d'une `List`.

> **Remarque :** certaines opérations forcent l'évaluation sans `toList()` : `length`, `first`, `isEmpty`, `forEach`, `join`, `reduce`, `fold`, ou une boucle `for-in`. Le `toList()` reste néanmoins la façon la plus sûre et la plus claire d'obtenir un résultat concret.

---

## 14.7 — `where()` (filtrer)

`where()` **sélectionne** les éléments qui satisfont une condition. Contrairement à `map()`, le nombre d'éléments diminue (ou reste identique), mais le type ne change pas.

```text
  [120, 80, 340, 55]         liste de départ
    │    │    │    │
   >100 >100 >100 >100       la condition est testée sur chacun
   vrai  faux vrai faux
    │         │
    v         v
  [120,     340]             seuls les éléments retenus
```

Version avec une boucle `for` :

```dart
void main() {
  List<int> scores = [120, 80, 340, 55];

  List<int> bons = [];
  for (final s in scores) {
    if (s > 100) {
      bons.add(s);
    }
  }

  print(bons);
}
```

**Résultat :**

```text
[120, 340]
```

Version fonctionnelle équivalente :

```dart
void main() {
  List<int> scores = [120, 80, 340, 55];

  List<int> bons = scores.where((s) => s > 100).toList();

  print(bons);
}
```

**Résultat :**

```text
[120, 340]
```

La fonction passée à `where()` doit obligatoirement renvoyer un `bool`. On l'appelle un **prédicat**.

Sur des objets, c'est là que `where()` prend toute sa valeur :

```dart
class Joueur {
  final String nom;
  final int score;
  final bool estConnecte;

  Joueur(this.nom, this.score, this.estConnecte);

  @override
  String toString() => '$nom:$score';
}

void main() {
  List<Joueur> equipe = [
    Joueur('Alex', 120, true),
    Joueur('Bilal', 340, false),
    Joueur('Chloé', 80, true),
    Joueur('Dan', 500, true),
  ];

  List<Joueur> connectes = equipe.where((j) => j.estConnecte).toList();
  List<Joueur> elites = equipe.where((j) => j.score >= 300).toList();

  print('connectés : $connectes');
  print('élites    : $elites');
}
```

**Résultat :**

```text
connectés : [Alex:120, Chloé:80, Dan:500]
élites    : [Bilal:340, Dan:500]
```

Une condition peut combiner plusieurs critères avec `&&` et `||` :

```dart
void main() {
  List<Joueur> equipe = [
    Joueur('Alex', 120, true),
    Joueur('Bilal', 340, false),
    Joueur('Chloé', 80, true),
    Joueur('Dan', 500, true),
  ];

  List<Joueur> elitesEnLigne =
      equipe.where((j) => j.estConnecte && j.score >= 300).toList();

  print(elitesEnLigne);
}

class Joueur {
  final String nom;
  final int score;
  final bool estConnecte;

  Joueur(this.nom, this.score, this.estConnecte);

  @override
  String toString() => '$nom:$score';
}
```

**Résultat :**

```text
[Dan:500]
```

Si aucun élément ne satisfait la condition, on obtient une liste **vide**, jamais `null` :

```dart
void main() {
  List<int> scores = [10, 20, 30];

  List<int> impossibles = scores.where((s) => s > 1000).toList();

  print(impossibles);
  print('vide ? ${impossibles.isEmpty}');
}
```

**Résultat :**

```text
[]
vide ? true
```

C'est une excellente nouvelle : pas de `null` à tester, donc pas de crash possible.

> **Remarque :** il existe aussi `whereType<T>()`, qui filtre par type. Sur une `List<Object>` contenant des `int` et des `String`, `liste.whereType<int>()` ne garde que les entiers.

---

## 14.8 — `firstWhere()` et `orElse`

`where()` rend **tous** les éléments correspondants. `firstWhere()` rend **le premier**, et un seul.

Version avec une boucle `for` :

```dart
class Joueur {
  final String nom;
  final int score;

  Joueur(this.nom, this.score);

  @override
  String toString() => '$nom:$score';
}

void main() {
  List<Joueur> equipe = [
    Joueur('Alex', 120),
    Joueur('Bilal', 340),
    Joueur('Chloé', 380),
  ];

  Joueur? trouve;
  for (final j in equipe) {
    if (j.score > 300) {
      trouve = j;
      break;
    }
  }

  print(trouve);
}
```

**Résultat :**

```text
Bilal:340
```

Version fonctionnelle équivalente :

```dart
void main() {
  List<Joueur> equipe = [
    Joueur('Alex', 120),
    Joueur('Bilal', 340),
    Joueur('Chloé', 380),
  ];

  Joueur trouve = equipe.firstWhere((j) => j.score > 300);

  print(trouve);
}

class Joueur {
  final String nom;
  final int score;

  Joueur(this.nom, this.score);

  @override
  String toString() => '$nom:$score';
}
```

**Résultat :**

```text
Bilal:340
```

`firstWhere()` s'arrête dès qu'il a trouvé. Chloé n'est même pas testée.

Voici maintenant **le piège numéro un du chapitre**. Que se passe-t-il si aucun élément ne convient ?

```dart
void main() {
  List<int> scores = [120, 80, 55];

  int trouve = scores.firstWhere((s) => s > 1000);

  print(trouve);
}
```

**Résultat :**

```text
Unhandled exception:
Bad state: No element
```

Le programme **plante**. `firstWhere()` sans filet lance une `StateError` quand rien ne correspond. Ce n'est pas `null` qui est renvoyé : c'est une exception.

La solution est le paramètre nommé `orElse`. Il attend une fonction **sans argument** qui fournit la valeur de repli.

```dart
void main() {
  List<int> scores = [120, 80, 55];

  int trouve = scores.firstWhere((s) => s > 1000, orElse: () => -1);

  print(trouve);
}
```

**Résultat :**

```text
-1
```

Avec des objets, deux stratégies existent. La première : renvoyer un objet « par défaut ».

```dart
class Joueur {
  final String nom;
  final int score;

  Joueur(this.nom, this.score);

  @override
  String toString() => '$nom:$score';
}

void main() {
  List<Joueur> equipe = [
    Joueur('Alex', 120),
    Joueur('Chloé', 80),
  ];

  Joueur champion = equipe.firstWhere(
    (j) => j.score > 300,
    orElse: () => Joueur('personne', 0),
  );

  print(champion);
}
```

**Résultat :**

```text
personne:0
```

La seconde, plus honnête : accepter que le résultat soit `null` et le déclarer nullable.

```dart
class Joueur {
  final String nom;
  final int score;

  Joueur(this.nom, this.score);

  @override
  String toString() => '$nom:$score';
}

void main() {
  List<Joueur> equipe = [
    Joueur('Alex', 120),
    Joueur('Chloé', 80),
  ];

  Joueur? champion = equipe
      .cast<Joueur?>()
      .firstWhere((j) => (j?.score ?? 0) > 300, orElse: () => null);

  if (champion == null) {
    print('Aucun champion.');
  } else {
    print('Champion : $champion');
  }
}
```

**Résultat :**

```text
Aucun champion.
```

Cette écriture avec `cast` est lourde. En pratique, on préfère très souvent cette forme, bien plus lisible :

```dart
class Joueur {
  final String nom;
  final int score;

  Joueur(this.nom, this.score);

  @override
  String toString() => '$nom:$score';
}

void main() {
  List<Joueur> equipe = [
    Joueur('Alex', 120),
    Joueur('Chloé', 80),
  ];

  final candidats = equipe.where((j) => j.score > 300).toList();
  final Joueur? champion = candidats.isEmpty ? null : candidats.first;

  print(champion == null ? 'Aucun champion.' : 'Champion : $champion');
}
```

**Résultat :**

```text
Aucun champion.
```

> **Règle à retenir :** n'écrivez **jamais** `firstWhere()` sans `orElse`, sauf si vous êtes absolument certain qu'un élément correspond. Dans le doute, ajoutez `orElse`.

---

## 14.9 — `lastWhere()`, `singleWhere()`

`lastWhere()` fonctionne comme `firstWhere()`, mais parcourt la liste et retient **le dernier** élément correspondant.

Version avec une boucle `for` :

```dart
void main() {
  List<int> degats = [12, 45, 8, 60, 3];

  int? dernierGros;
  for (final d in degats) {
    if (d > 10) {
      dernierGros = d;
    }
  }

  print(dernierGros);
}
```

**Résultat :**

```text
60
```

Notez l'absence de `break` : on continue jusqu'au bout pour garder le dernier.

Version fonctionnelle équivalente :

```dart
void main() {
  List<int> degats = [12, 45, 8, 60, 3];

  int dernierGros = degats.lastWhere((d) => d > 10, orElse: () => 0);

  print(dernierGros);
}
```

**Résultat :**

```text
60
```

Comparons les deux sur la même liste :

```dart
void main() {
  List<int> degats = [12, 45, 8, 60, 3];

  print('premier > 10 : ${degats.firstWhere((d) => d > 10, orElse: () => 0)}');
  print('dernier > 10 : ${degats.lastWhere((d) => d > 10, orElse: () => 0)}');
}
```

**Résultat :**

```text
premier > 10 : 12
dernier > 10 : 60
```

`singleWhere()` est plus strict : il exige qu'il y ait **exactement un** élément correspondant. Utilisez-le quand l'unicité est une règle de votre jeu (un seul boss, un seul joueur avec un identifiant donné).

```dart
class Ennemi {
  final String nom;
  final bool estBoss;

  Ennemi(this.nom, this.estBoss);

  @override
  String toString() => nom;
}

void main() {
  List<Ennemi> vague = [
    Ennemi('gobelin', false),
    Ennemi('Dragon Noir', true),
    Ennemi('orc', false),
  ];

  Ennemi boss = vague.singleWhere((e) => e.estBoss);

  print('Boss de la vague : $boss');
}
```

**Résultat :**

```text
Boss de la vague : Dragon Noir
```

S'il y a **deux** boss, `singleWhere()` refuse :

```dart
class Ennemi {
  final String nom;
  final bool estBoss;

  Ennemi(this.nom, this.estBoss);
}

void main() {
  List<Ennemi> vague = [
    Ennemi('Dragon Noir', true),
    Ennemi('Liche', true),
  ];

  Ennemi boss = vague.singleWhere((e) => e.estBoss);

  print(boss.nom);
}
```

**Résultat :**

```text
Unhandled exception:
Bad state: Too many elements
```

S'il n'y en a **aucun**, il refuse aussi :

```text
Unhandled exception:
Bad state: No element
```

`singleWhere()` accepte lui aussi `orElse`, mais celui-ci ne couvre que le cas « aucun élément » :

```dart
class Ennemi {
  final String nom;
  final bool estBoss;

  Ennemi(this.nom, this.estBoss);

  @override
  String toString() => nom;
}

void main() {
  List<Ennemi> vague = [
    Ennemi('gobelin', false),
    Ennemi('orc', false),
  ];

  Ennemi boss = vague.singleWhere(
    (e) => e.estBoss,
    orElse: () => Ennemi('aucun boss', false),
  );

  print(boss);
}
```

**Résultat :**

```text
aucun boss
```

Résumé des trois recherches :

| Méthode | Renvoie | Si plusieurs correspondent | Si aucun ne correspond |
| --- | --- | --- | --- |
| `firstWhere()` | le premier | prend le premier | `StateError` (sauf `orElse`) |
| `lastWhere()` | le dernier | prend le dernier | `StateError` (sauf `orElse`) |
| `singleWhere()` | l'unique | `StateError` | `StateError` (sauf `orElse`) |

---

## 14.10 — `indexWhere()`

`indexWhere()` ne rend pas l'élément, mais **sa position** dans la liste. C'est utile quand vous voulez ensuite modifier ou supprimer cet élément.

Version avec une boucle `for` :

```dart
void main() {
  List<String> inventaire = ['Potion', 'Épée', 'Bouclier', 'Élixir'];

  int position = -1;
  for (int i = 0; i < inventaire.length; i++) {
    if (inventaire[i].startsWith('É')) {
      position = i;
      break;
    }
  }

  print(position);
}
```

**Résultat :**

```text
1
```

Version fonctionnelle équivalente :

```dart
void main() {
  List<String> inventaire = ['Potion', 'Épée', 'Bouclier', 'Élixir'];

  int position = inventaire.indexWhere((o) => o.startsWith('É'));

  print(position);
}
```

**Résultat :**

```text
1
```

Point crucial : quand rien ne correspond, `indexWhere()` ne plante pas. Il renvoie **`-1`**.

```dart
void main() {
  List<String> inventaire = ['Potion', 'Épée'];

  int position = inventaire.indexWhere((o) => o == 'Arc');

  print(position);
  if (position == -1) {
    print('Objet introuvable dans le sac.');
  }
}
```

**Résultat :**

```text
-1
Objet introuvable dans le sac.
```

C'est donc une méthode plus douce que `firstWhere()` : aucune exception, jamais.

Exemple d'utilisation réelle : remplacer un objet dans l'inventaire.

```dart
void main() {
  List<String> inventaire = ['Potion', 'Épée rouillée', 'Bouclier'];

  int i = inventaire.indexWhere((o) => o.contains('rouillée'));
  if (i != -1) {
    inventaire[i] = 'Épée de feu';
  }

  print(inventaire);
}
```

**Résultat :**

```text
[Potion, Épée de feu, Bouclier]
```

Il existe aussi `lastIndexWhere()`, qui cherche depuis la fin :

```dart
void main() {
  List<int> degats = [5, 30, 7, 40, 2];

  print('premier index > 10 : ${degats.indexWhere((d) => d > 10)}');
  print('dernier index > 10 : ${degats.lastIndexWhere((d) => d > 10)}');
}
```

**Résultat :**

```text
premier index > 10 : 1
dernier index > 10 : 3
```

Et `indexOf()`, qui cherche une **valeur** plutôt qu'une condition :

```dart
void main() {
  List<String> inventaire = ['Potion', 'Épée', 'Bouclier'];

  print(inventaire.indexOf('Épée'));
  print(inventaire.indexOf('Arc'));
}
```

**Résultat :**

```text
1
-1
```

---

## 14.11 — `any()`

`any()` répond à une question par oui ou non : « **au moins un** élément satisfait-il la condition ? ». Le résultat est un `bool`.

Version avec une boucle `for` :

```dart
void main() {
  List<int> pvEnnemis = [40, 0, 25];

  bool ilResteUnVivant = false;
  for (final pv in pvEnnemis) {
    if (pv > 0) {
      ilResteUnVivant = true;
      break;
    }
  }

  print(ilResteUnVivant);
}
```

**Résultat :**

```text
true
```

Version fonctionnelle équivalente :

```dart
void main() {
  List<int> pvEnnemis = [40, 0, 25];

  bool ilResteUnVivant = pvEnnemis.any((pv) => pv > 0);

  print(ilResteUnVivant);
}
```

**Résultat :**

```text
true
```

`any()` s'arrête **dès le premier succès**. Sur une liste de 10 000 ennemis dont le premier est vivant, un seul test est effectué.

Sur des objets :

```dart
class Ennemi {
  final String nom;
  final int pv;

  Ennemi(this.nom, this.pv);
}

void main() {
  List<Ennemi> vague = [
    Ennemi('gobelin', 0),
    Ennemi('orc', 0),
    Ennemi('troll', 15),
  ];

  print('Combat en cours ? ${vague.any((e) => e.pv > 0)}');
  print('Un boss ?         ${vague.any((e) => e.nom == 'Dragon')}');
}
```

**Résultat :**

```text
Combat en cours ? true
Un boss ?         false
```

Sur une liste **vide**, `any()` renvoie toujours `false` : il n'y a aucun élément pour satisfaire la condition.

```dart
void main() {
  List<int> vide = [];
  print(vide.any((x) => x > 0));
}
```

**Résultat :**

```text
false
```

---

## 14.12 — `every()`

`every()` pose la question inverse : « **tous** les éléments satisfont-ils la condition ? ».

Version avec une boucle `for` :

```dart
void main() {
  List<int> pvEquipe = [100, 80, 100];

  bool tousEnPleineForme = true;
  for (final pv in pvEquipe) {
    if (pv < 100) {
      tousEnPleineForme = false;
      break;
    }
  }

  print(tousEnPleineForme);
}
```

**Résultat :**

```text
false
```

Version fonctionnelle équivalente :

```dart
void main() {
  List<int> pvEquipe = [100, 80, 100];

  bool tousEnPleineForme = pvEquipe.every((pv) => pv == 100);

  print(tousEnPleineForme);
}
```

**Résultat :**

```text
false
```

`every()` s'arrête **dès le premier échec**, par symétrie avec `any()`.

Un usage typique dans un jeu : décider si le niveau est terminé.

```dart
class Ennemi {
  final String nom;
  final int pv;

  Ennemi(this.nom, this.pv);
}

void main() {
  List<Ennemi> vague = [
    Ennemi('gobelin', 0),
    Ennemi('orc', 0),
    Ennemi('troll', 0),
  ];

  bool niveauTermine = vague.every((e) => e.pv == 0);

  print(niveauTermine ? 'Niveau terminé !' : 'Il reste des ennemis.');
}
```

**Résultat :**

```text
Niveau terminé !
```

Attention au comportement sur une liste vide : `every()` renvoie **`true`**.

```dart
void main() {
  List<int> vide = [];
  print(vide.every((x) => x > 1000));
}
```

**Résultat :**

```text
true
```

Cela peut surprendre, mais c'est logique : il n'existe aucun élément qui viole la condition. En mathématiques, on parle de vérité « vide ». Dans un jeu, cela signifie qu'une vague sans ennemi est considérée comme terminée, ce qui est en général le comportement souhaité.

---

## 14.13 — `contains()`

`contains()` teste la présence d'une **valeur précise**. Ce n'est pas un prédicat : on passe la valeur elle-même, pas une fonction.

Version avec une boucle `for` :

```dart
void main() {
  List<String> inventaire = ['Potion', 'Épée', 'Bouclier'];

  bool aUnePotion = false;
  for (final o in inventaire) {
    if (o == 'Potion') {
      aUnePotion = true;
      break;
    }
  }

  print(aUnePotion);
}
```

**Résultat :**

```text
true
```

Version fonctionnelle équivalente :

```dart
void main() {
  List<String> inventaire = ['Potion', 'Épée', 'Bouclier'];

  print(inventaire.contains('Potion'));
  print(inventaire.contains('Arc'));
}
```

**Résultat :**

```text
true
false
```

`contains()` s'appuie sur l'opérateur `==`. Pour les types de base (`int`, `String`, `bool`), cela fonctionne exactement comme attendu.

Sur des objets, en revanche, `==` compare par défaut les **identités**, pas les contenus :

```dart
class Objet {
  final String nom;

  Objet(this.nom);
}

void main() {
  List<Objet> sac = [Objet('Potion'), Objet('Épée')];

  print(sac.contains(Objet('Potion')));
}
```

**Résultat :**

```text
false
```

La `Potion` créée dans le `contains()` est un objet **différent** de celle du sac, même si le nom est identique. Deux solutions.

Première solution : utiliser `any()` avec un critère explicite.

```dart
class Objet {
  final String nom;

  Objet(this.nom);
}

void main() {
  List<Objet> sac = [Objet('Potion'), Objet('Épée')];

  print(sac.any((o) => o.nom == 'Potion'));
}
```

**Résultat :**

```text
true
```

Seconde solution : redéfinir `==` et `hashCode` dans la classe.

```dart
class Objet {
  final String nom;

  Objet(this.nom);

  @override
  bool operator ==(Object other) => other is Objet && other.nom == nom;

  @override
  int get hashCode => nom.hashCode;
}

void main() {
  List<Objet> sac = [Objet('Potion'), Objet('Épée')];

  print(sac.contains(Objet('Potion')));
}
```

**Résultat :**

```text
true
```

> **Remarque de performance :** sur une `List`, `contains()` parcourt les éléments un par un. Sur un `Set`, la réponse est immédiate quelle que soit la taille. Si vous testez souvent l'appartenance, stockez vos données dans un `Set`.

---

## 14.14 — `reduce()`

`reduce()` **réduit** une collection entière à une seule valeur : une somme, un maximum, un produit.

Le principe : on prend les deux premiers éléments, on les combine, puis on combine le résultat avec le troisième, et ainsi de suite.

```text
  [120, 80, 340, 55]

  120 + 80 = 200
            200 + 340 = 540
                        540 + 55 = 595
                                   ────
                                   595
```

Version avec une boucle `for` :

```dart
void main() {
  List<int> scores = [120, 80, 340, 55];

  int total = 0;
  for (final s in scores) {
    total = total + s;
  }

  print(total);
}
```

**Résultat :**

```text
595
```

Version fonctionnelle équivalente :

```dart
void main() {
  List<int> scores = [120, 80, 340, 55];

  int total = scores.reduce((a, b) => a + b);

  print(total);
}
```

**Résultat :**

```text
595
```

La fonction passée à `reduce()` reçoit **deux** paramètres :

- `a` : la valeur accumulée jusqu'ici ;
- `b` : l'élément suivant.

Le maximum s'écrit avec la même mécanique :

```dart
void main() {
  List<int> scores = [120, 80, 340, 55];

  int meilleur = scores.reduce((a, b) => a > b ? a : b);
  int pire = scores.reduce((a, b) => a < b ? a : b);

  print('meilleur : $meilleur');
  print('pire     : $pire');
}
```

**Résultat :**

```text
meilleur : 340
pire     : 55
```

Voici le piège de `reduce()` : sur une liste **vide**, il n'y a aucune valeur de départ, donc il plante.

```dart
void main() {
  List<int> scores = [];

  int total = scores.reduce((a, b) => a + b);

  print(total);
}
```

**Résultat :**

```text
Unhandled exception:
Bad state: No element
```

Il faut donc systématiquement protéger l'appel :

```dart
void main() {
  List<int> scores = [];

  int total = scores.isEmpty ? 0 : scores.reduce((a, b) => a + b);

  print(total);
}
```

**Résultat :**

```text
0
```

Autre limite : `reduce()` impose que le résultat soit **du même type** que les éléments. Additionner des `int` donne un `int`, c'est cohérent. Mais on ne peut pas réduire une `List<Joueur>` en un `int` avec `reduce()`. Pour cela, il faut `fold()`.

> **Remarque :** pour une simple somme de nombres, Dart offre aussi un raccourci via le paquet `dart:math` pour le min/max de deux valeurs, mais `reduce` reste la façon générique de résumer une collection.

---

## 14.15 — `fold()`

`fold()` fait la même chose que `reduce()`, avec **une valeur de départ fournie par vous**.

```text
  fold(0, (a, b) => a + b)   sur  [120, 80, 340]

  départ :          0
    0 + 120 =     120
  120 +  80 =     200
  200 + 340 =     540
                  ───
                  540
```

Version avec une boucle `for` :

```dart
void main() {
  List<int> scores = [120, 80, 340];

  int total = 0;
  for (final s in scores) {
    total += s;
  }

  print(total);
}
```

**Résultat :**

```text
540
```

Version fonctionnelle équivalente :

```dart
void main() {
  List<int> scores = [120, 80, 340];

  int total = scores.fold(0, (a, b) => a + b);

  print(total);
}
```

**Résultat :**

```text
540
```

Premier avantage immédiat : sur une liste vide, `fold()` **ne plante pas**. Il renvoie la valeur de départ.

```dart
void main() {
  List<int> vide = [];

  print(vide.fold(0, (a, b) => a + b));
}
```

**Résultat :**

```text
0
```

Second avantage, bien plus puissant : le type accumulé peut être **différent** du type des éléments.

Additionner les scores d'une liste de joueurs, par exemple :

```dart
class Joueur {
  final String nom;
  final int score;

  Joueur(this.nom, this.score);
}

void main() {
  List<Joueur> equipe = [
    Joueur('Alex', 120),
    Joueur('Bilal', 340),
    Joueur('Chloé', 80),
  ];

  // List<Joueur> -> int
  int totalEquipe = equipe.fold<int>(0, (somme, j) => somme + j.score);

  print('Total de l\'équipe : $totalEquipe');
}
```

**Résultat :**

```text
Total de l'équipe : 540
```

Ou construire un texte à partir de nombres :

```dart
void main() {
  List<int> degats = [12, 40, 7];

  // List<int> -> String
  String journal = degats.fold<String>(
    'Combat :',
    (texte, d) => '$texte -$d',
  );

  print(journal);
}
```

**Résultat :**

```text
Combat : -12 -40 -7
```

`fold()` sait même construire une `Map`, par exemple pour compter les ennemis par type :

```dart
void main() {
  List<String> vague = ['orc', 'gobelin', 'orc', 'troll', 'orc'];

  Map<String, int> comptage = vague.fold<Map<String, int>>(
    {},
    (acc, nom) {
      acc[nom] = (acc[nom] ?? 0) + 1;
      return acc;
    },
  );

  print(comptage);
}
```

**Résultat :**

```text
{orc: 3, gobelin: 1, troll: 1}
```

> **Remarque :** précisez le type entre chevrons (`fold<int>`, `fold<String>`) dès que l'inférence hésite. Sans cela, Dart déduit parfois `Object` ou `dynamic`, ce qui produit des erreurs difficiles à lire.

---

## 14.16 — `reduce()` vs `fold()`

Les deux méthodes résument une collection en une valeur. Voici comment choisir.

| Critère | `reduce()` | `fold()` |
| --- | --- | --- |
| Valeur de départ | aucune (le 1er élément) | fournie par vous |
| Liste vide | `StateError` | renvoie la valeur de départ |
| Type du résultat | identique aux éléments | libre |
| Écriture | plus courte | plus explicite |

Le même calcul, écrit des deux façons :

```dart
void main() {
  List<int> scores = [120, 80, 340];

  print(scores.reduce((a, b) => a + b));
  print(scores.fold<int>(0, (a, b) => a + b));
}
```

**Résultat :**

```text
540
540
```

Sur une liste vide, la différence est brutale :

```dart
void main() {
  List<int> vide = [];

  print(vide.fold<int>(0, (a, b) => a + b));

  try {
    print(vide.reduce((a, b) => a + b));
  } on StateError catch (e) {
    print('reduce a échoué : ${e.message}');
  }
}
```

**Résultat :**

```text
0
reduce a échoué : No element
```

Le cas où `reduce()` reste préférable est le maximum ou le minimum. Avec `fold()`, il faudrait inventer une valeur de départ arbitraire :

```dart
void main() {
  List<int> scores = [120, 80, 340];

  // clair
  print(scores.reduce((a, b) => a > b ? a : b));

  // possible, mais la valeur de départ est discutable
  print(scores.fold<int>(-999999, (a, b) => a > b ? a : b));
}
```

**Résultat :**

```text
340
340
```

Si les scores pouvaient être encore plus petits que `-999999`, le second calcul serait faux. C'est exactement ce genre de valeur « magique » qu'il faut éviter.

> **Règle simple :** utilisez `fold()` par défaut (il ne plante jamais et accepte tout type). Réservez `reduce()` au minimum et au maximum, sur une liste dont vous avez vérifié qu'elle n'est pas vide.

---

## 14.17 — `sort()` et le comparateur

`sort()` trie une `List`. Attention : contrairement à `map()` et `where()`, il **modifie la liste d'origine** et ne renvoie rien.

Sans argument, `sort()` utilise l'ordre naturel : croissant pour les nombres, alphabétique pour les textes.

```dart
void main() {
  List<int> scores = [120, 80, 340, 55];

  scores.sort();

  print(scores);
}
```

**Résultat :**

```text
[55, 80, 120, 340]
```

Pour tout autre ordre, on fournit un **comparateur** : une fonction qui reçoit deux éléments `a` et `b` et renvoie un `int`.

```text
  comparateur(a, b) renvoie :

    un nombre NÉGATIF  ->  a doit être placé AVANT b
    ZÉRO               ->  a et b sont équivalents
    un nombre POSITIF  ->  a doit être placé APRÈS b
```

Tri décroissant :

```dart
void main() {
  List<int> scores = [120, 80, 340, 55];

  scores.sort((a, b) => b - a);

  print(scores);
}
```

**Résultat :**

```text
[340, 120, 80, 55]
```

Pourquoi `b - a` ? Si `b` est plus grand que `a`, la soustraction est positive, donc `a` passe après `b` : les grandes valeurs remontent.

L'écriture `a - b` fonctionne pour des `int`, mais elle est risquée sur de très grands nombres (risque de débordement) et impossible sur des `String`. On préfère donc toujours `compareTo()`, présenté à la section suivante.

Le point à ne jamais oublier : `sort()` est **destructif**.

```dart
void main() {
  List<int> scores = [120, 80, 340];

  var resultat = scores.sort();

  print('liste  : $scores');
  print('retour : $resultat');
}
```

**Résultat :**

```text
liste  : [80, 120, 340]
retour : null
```

`sort()` renvoie `null` (son type est `void`). Ce code est donc une erreur fréquente :

```dart
// NE COMPILE PAS
List<int> tries = scores.sort();
```

**Résultat :**

```text
Error: A value of type 'void' can't be assigned to a variable of type 'List<int>'.
```

La section 14.20 montre comment trier sans abîmer l'original.

---

## 14.18 — `compareTo()`

`compareTo()` est la méthode standard de comparaison en Dart. Elle existe sur tous les types qui implémentent `Comparable` : `int`, `double`, `String`, `DateTime`, `Duration`, et vos propres classes si vous le décidez.

Elle renvoie exactement les trois valeurs attendues par un comparateur.

```dart
void main() {
  print(5.compareTo(10));
  print(10.compareTo(10));
  print(10.compareTo(5));

  print('alex'.compareTo('bilal'));
  print('bilal'.compareTo('alex'));
}
```

**Résultat :**

```text
-1
0
1
-1
1
```

D'où les deux formules à retenir :

```text
  croissant   :  (a, b) => a.compareTo(b)
  décroissant :  (a, b) => b.compareTo(a)
```

Sur des nombres :

```dart
void main() {
  List<int> scores = [120, 80, 340, 55];

  List<int> croissant = List<int>.from(scores)..sort((a, b) => a.compareTo(b));
  List<int> decroissant = List<int>.from(scores)..sort((a, b) => b.compareTo(a));

  print('croissant   : $croissant');
  print('décroissant : $decroissant');
}
```

**Résultat :**

```text
croissant   : [55, 80, 120, 340]
décroissant : [340, 120, 80, 55]
```

Sur des textes, `compareTo()` suit l'ordre des codes de caractères. Les **majuscules passent avant les minuscules** :

```dart
void main() {
  List<String> noms = ['bilal', 'Alex', 'chloé'];

  noms.sort((a, b) => a.compareTo(b));

  print(noms);
}
```

**Résultat :**

```text
[Alex, bilal, chloé]
```

Ici le résultat semble correct, mais ce n'est qu'une coïncidence. Regardez ce cas :

```dart
void main() {
  List<String> noms = ['bilal', 'Zoé', 'alex'];

  noms.sort((a, b) => a.compareTo(b));

  print(noms);
}
```

**Résultat :**

```text
[Zoé, alex, bilal]
```

`Zoé` passe avant `alex` parce que `Z` (code 90) est inférieur à `a` (code 97). Pour un classement affiché à l'utilisateur, ce n'est pas acceptable.

La correction consiste à comparer les versions en minuscules :

```dart
void main() {
  List<String> noms = ['bilal', 'Zoé', 'alex'];

  noms.sort((a, b) => a.toLowerCase().compareTo(b.toLowerCase()));

  print(noms);
}
```

**Résultat :**

```text
[alex, bilal, Zoé]
```

---

## 14.19 — Trier des objets (par score, par nom)

Vos vraies listes contiennent des objets, pas des nombres. Le comparateur doit alors désigner **le champ** sur lequel porte le tri.

Version avec une boucle `for` (tri par sélection, écrit à la main) :

```dart
class Joueur {
  final String nom;
  final int score;

  Joueur(this.nom, this.score);

  @override
  String toString() => '$nom($score)';
}

void main() {
  List<Joueur> equipe = [
    Joueur('Alex', 120),
    Joueur('Bilal', 340),
    Joueur('Chloé', 80),
  ];

  for (int i = 0; i < equipe.length - 1; i++) {
    for (int k = i + 1; k < equipe.length; k++) {
      if (equipe[k].score > equipe[i].score) {
        final tampon = equipe[i];
        equipe[i] = equipe[k];
        equipe[k] = tampon;
      }
    }
  }

  print(equipe);
}
```

**Résultat :**

```text
[Bilal(340), Alex(120), Chloé(80)]
```

Quinze lignes, deux boucles imbriquées, une variable tampon. Voici la version fonctionnelle équivalente :

```dart
class Joueur {
  final String nom;
  final int score;

  Joueur(this.nom, this.score);

  @override
  String toString() => '$nom($score)';
}

void main() {
  List<Joueur> equipe = [
    Joueur('Alex', 120),
    Joueur('Bilal', 340),
    Joueur('Chloé', 80),
  ];

  equipe.sort((a, b) => b.score.compareTo(a.score));

  print(equipe);
}
```

**Résultat :**

```text
[Bilal(340), Alex(120), Chloé(80)]
```

Une ligne. Et elle se lit : « trie par score décroissant ».

Tri par nom :

```dart
class Joueur {
  final String nom;
  final int score;

  Joueur(this.nom, this.score);

  @override
  String toString() => '$nom($score)';
}

void main() {
  List<Joueur> equipe = [
    Joueur('Chloé', 80),
    Joueur('Alex', 120),
    Joueur('Bilal', 340),
  ];

  equipe.sort((a, b) => a.nom.compareTo(b.nom));

  print(equipe);
}
```

**Résultat :**

```text
[Alex(120), Bilal(340), Chloé(80)]
```

Souvent, un seul critère ne suffit pas. Comment départager deux joueurs à égalité de score ? Par leur nom. On enchaîne alors les comparaisons : si la première renvoie `0`, on passe à la seconde.

```dart
class Joueur {
  final String nom;
  final int score;

  Joueur(this.nom, this.score);

  @override
  String toString() => '$nom($score)';
}

void main() {
  List<Joueur> equipe = [
    Joueur('Chloé', 120),
    Joueur('Alex', 120),
    Joueur('Bilal', 340),
  ];

  equipe.sort((a, b) {
    final parScore = b.score.compareTo(a.score);
    if (parScore != 0) return parScore;
    return a.nom.compareTo(b.nom);
  });

  print(equipe);
}
```

**Résultat :**

```text
[Bilal(340), Alex(120), Chloé(120)]
```

Alex et Chloé ont le même score ; c'est le nom qui les départage.

Vous pouvez aussi rendre la classe elle-même comparable, en implémentant `Comparable`. `sort()` sans argument saura alors la trier.

```dart
class Joueur implements Comparable<Joueur> {
  final String nom;
  final int score;

  Joueur(this.nom, this.score);

  @override
  int compareTo(Joueur autre) => autre.score.compareTo(score);

  @override
  String toString() => '$nom($score)';
}

void main() {
  List<Joueur> equipe = [
    Joueur('Alex', 120),
    Joueur('Bilal', 340),
    Joueur('Chloé', 80),
  ];

  equipe.sort();

  print(equipe);
}
```

**Résultat :**

```text
[Bilal(340), Alex(120), Chloé(80)]
```

> **Remarque :** l'ordre « naturel » d'une classe doit être évident pour tout le monde. Si vos joueurs peuvent être triés par score, par nom ou par date d'inscription selon l'écran affiché, ne choisissez pas : gardez des comparateurs explicites au moment de l'appel.

---

## 14.20 — Trier sans modifier l'original (`List.from`)

`sort()` modifie la liste sur laquelle il est appelé. Si cette liste est votre source de vérité (la liste des joueurs de la partie en cours), la trier pour afficher un classement est une mauvaise idée : vous perdez l'ordre initial.

Le problème, en clair :

```dart
void main() {
  List<int> scores = [120, 80, 340];

  scores.sort();

  print('ordre d\'arrivée perdu : $scores');
}
```

**Résultat :**

```text
ordre d'arrivée perdu : [80, 120, 340]
```

La solution : trier une **copie**. `List.from()` crée une nouvelle liste contenant les mêmes éléments.

```dart
void main() {
  List<int> scores = [120, 80, 340];

  List<int> tries = List<int>.from(scores);
  tries.sort();

  print('original : $scores');
  print('trié     : $tries');
}
```

**Résultat :**

```text
original : [120, 80, 340]
trié     : [80, 120, 340]
```

On écrit souvent cela en une seule ligne avec l'opérateur cascade `..`, qui renvoie l'objet lui-même au lieu du résultat de la méthode :

```dart
void main() {
  List<int> scores = [120, 80, 340];

  final tries = List<int>.from(scores)..sort();

  print('original : $scores');
  print('trié     : $tries');
}
```

**Résultat :**

```text
original : [120, 80, 340]
trié     : [80, 120, 340]
```

Trois façons de copier une liste, toutes équivalentes ici :

```dart
void main() {
  List<int> scores = [120, 80, 340];

  final copie1 = List<int>.from(scores);
  final copie2 = [...scores];
  final copie3 = scores.toList();

  print(copie1);
  print(copie2);
  print(copie3);
}
```

**Résultat :**

```text
[120, 80, 340]
[120, 80, 340]
[120, 80, 340]
```

Avec des objets, on obtient donc un classement propre sans toucher aux données du jeu :

```dart
class Joueur {
  final String nom;
  final int score;

  Joueur(this.nom, this.score);

  @override
  String toString() => '$nom($score)';
}

void main() {
  List<Joueur> equipe = [
    Joueur('Alex', 120),
    Joueur('Bilal', 340),
    Joueur('Chloé', 80),
  ];

  final classement = List<Joueur>.from(equipe)
    ..sort((a, b) => b.score.compareTo(a.score));

  print('ordre d\'arrivée : $equipe');
  print('classement      : $classement');
}
```

**Résultat :**

```text
ordre d'arrivée : [Alex(120), Bilal(340), Chloé(80)]
classement      : [Bilal(340), Alex(120), Chloé(80)]
```

> **Attention :** la copie est **superficielle**. Les deux listes contiennent les **mêmes** objets `Joueur`. Modifier `classement[0].score` modifierait aussi le joueur vu depuis `equipe`. Ce qui est protégé, c'est l'ordre, pas le contenu des objets.

---

## 14.21 — `take()`, `skip()`

`take(n)` garde les `n` **premiers** éléments. `skip(n)` **ignore** les `n` premiers et garde le reste.

Version avec une boucle `for` :

```dart
void main() {
  List<int> classement = [500, 340, 120, 80, 55];

  List<int> podium = [];
  for (int i = 0; i < classement.length; i++) {
    if (i < 3) {
      podium.add(classement[i]);
    }
  }

  print(podium);
}
```

**Résultat :**

```text
[500, 340, 120]
```

Version fonctionnelle équivalente :

```dart
void main() {
  List<int> classement = [500, 340, 120, 80, 55];

  List<int> podium = classement.take(3).toList();
  List<int> autres = classement.skip(3).toList();

  print('podium : $podium');
  print('autres : $autres');
}
```

**Résultat :**

```text
podium : [500, 340, 120]
autres : [80, 55]
```

Les deux méthodes se combinent pour extraire une « page » de résultats :

```dart
void main() {
  List<String> joueurs = ['Alex', 'Bilal', 'Chloé', 'Dan', 'Eva', 'Farid'];

  // page 2, avec 2 joueurs par page
  List<String> page2 = joueurs.skip(2).take(2).toList();

  print(page2);
}
```

**Résultat :**

```text
[Chloé, Dan]
```

Point rassurant : si vous demandez plus d'éléments qu'il n'en existe, il n'y a **aucune erreur**. La liste est simplement plus courte.

```dart
void main() {
  List<int> scores = [120, 80];

  print(scores.take(10).toList());
  print(scores.skip(10).toList());
}
```

**Résultat :**

```text
[120, 80]
[]
```

C'est un net avantage sur `scores.sublist(0, 10)`, qui lancerait une `RangeError`.

Il existe aussi `takeWhile()` et `skipWhile()`, qui s'arrêtent sur une condition au lieu d'un nombre :

```dart
void main() {
  List<int> degats = [50, 40, 30, 5, 60];

  print(degats.takeWhile((d) => d >= 30).toList());
  print(degats.skipWhile((d) => d >= 30).toList());
}
```

**Résultat :**

```text
[50, 40, 30]
[5, 60]
```

Notez bien : `takeWhile()` s'arrête au **premier** élément qui échoue. Le `60` final n'est pas repris, même s'il satisfait la condition.

---

## 14.22 — `expand()`

`expand()` transforme chaque élément en une **collection**, puis colle bout à bout toutes ces collections. On dit qu'il « aplatit ».

```text
  [ [Potion, Épée], [Bouclier], [Arc, Flèche] ]
        │                │           │
        v                v           v
    Potion, Épée     Bouclier     Arc, Flèche
        └────────────────┴───────────┘
                        v
    [Potion, Épée, Bouclier, Arc, Flèche]
```

Version avec une boucle `for` :

```dart
void main() {
  List<List<String>> sacs = [
    ['Potion', 'Épée'],
    ['Bouclier'],
    ['Arc', 'Flèche'],
  ];

  List<String> tout = [];
  for (final sac in sacs) {
    for (final objet in sac) {
      tout.add(objet);
    }
  }

  print(tout);
}
```

**Résultat :**

```text
[Potion, Épée, Bouclier, Arc, Flèche]
```

Version fonctionnelle équivalente :

```dart
void main() {
  List<List<String>> sacs = [
    ['Potion', 'Épée'],
    ['Bouclier'],
    ['Arc', 'Flèche'],
  ];

  List<String> tout = sacs.expand((sac) => sac).toList();

  print(tout);
}
```

**Résultat :**

```text
[Potion, Épée, Bouclier, Arc, Flèche]
```

`expand()` n'est pas limité à l'aplatissement : la fonction peut **fabriquer** une collection, et donc produire plus d'éléments qu'il n'y en avait.

```dart
void main() {
  List<String> ennemis = ['gobelin', 'orc'];

  // chaque ennemi apparaît en deux exemplaires
  List<String> vague = ennemis.expand((e) => [e, e]).toList();

  print(vague);
}
```

**Résultat :**

```text
[gobelin, gobelin, orc, orc]
```

Le cas réel le plus courant : rassembler les inventaires de tous les joueurs.

```dart
class Joueur {
  final String nom;
  final List<String> sac;

  Joueur(this.nom, this.sac);
}

void main() {
  List<Joueur> equipe = [
    Joueur('Alex', ['Potion', 'Épée']),
    Joueur('Bilal', ['Potion']),
    Joueur('Chloé', ['Bouclier', 'Arc']),
  ];

  List<String> butin = equipe.expand((j) => j.sac).toList();

  print(butin);
  print('nombre total d\'objets : ${butin.length}');
}
```

**Résultat :**

```text
[Potion, Épée, Potion, Bouclier, Arc]
nombre total d'objets : 5
```

> **À ne pas confondre :** `map()` garde toujours le même nombre d'éléments. `expand()` peut en produire plus ou moins. Avec `map((j) => j.sac)`, vous obtiendriez une liste de listes ; avec `expand`, une liste plate.

---

## 14.23 — `toSet()` pour dédoublonner

Un `Set` est une collection sans doublon. `toSet()` convertit une collection en `Set`, ce qui élimine automatiquement les répétitions.

Version avec une boucle `for` :

```dart
void main() {
  List<String> butin = ['Potion', 'Épée', 'Potion', 'Arc', 'Épée'];

  List<String> uniques = [];
  for (final o in butin) {
    if (!uniques.contains(o)) {
      uniques.add(o);
    }
  }

  print(uniques);
}
```

**Résultat :**

```text
[Potion, Épée, Arc]
```

Version fonctionnelle équivalente :

```dart
void main() {
  List<String> butin = ['Potion', 'Épée', 'Potion', 'Arc', 'Épée'];

  List<String> uniques = butin.toSet().toList();

  print(uniques);
}
```

**Résultat :**

```text
[Potion, Épée, Arc]
```

Le `Set` de Dart conserve l'**ordre d'insertion** : la première occurrence est gardée, les suivantes sont ignorées.

Si vous affichez directement le `Set`, remarquez les accolades :

```dart
void main() {
  List<String> butin = ['Potion', 'Épée', 'Potion'];

  print(butin.toSet());
  print(butin.toSet().runtimeType);
}
```

**Résultat :**

```text
{Potion, Épée}
{Potion, Épée}
```

Compter les types d'objets distincts devient immédiat :

```dart
void main() {
  List<String> butin = ['Potion', 'Épée', 'Potion', 'Arc', 'Épée', 'Potion'];

  print('objets ramassés : ${butin.length}');
  print('types distincts : ${butin.toSet().length}');
}
```

**Résultat :**

```text
objets ramassés : 6
types distincts : 3
```

Comme pour `contains()`, la déduplication d'objets repose sur `==` et `hashCode`. Sans redéfinition, deux `Joueur` de même nom sont considérés comme différents :

```dart
class Joueur {
  final String nom;

  Joueur(this.nom);

  @override
  bool operator ==(Object other) => other is Joueur && other.nom == nom;

  @override
  int get hashCode => nom.hashCode;

  @override
  String toString() => nom;
}

void main() {
  List<Joueur> presents = [Joueur('Alex'), Joueur('Bilal'), Joueur('Alex')];

  print(presents.toSet().toList());
}
```

**Résultat :**

```text
[Alex, Bilal]
```

---

## 14.24 — `join()`

`join()` assemble tous les éléments d'une collection en un seul `String`, avec un séparateur de votre choix.

Version avec une boucle `for` :

```dart
void main() {
  List<String> equipe = ['Alex', 'Bilal', 'Chloé'];

  String texte = '';
  for (int i = 0; i < equipe.length; i++) {
    texte += equipe[i];
    if (i < equipe.length - 1) {
      texte += ', ';
    }
  }

  print(texte);
}
```

**Résultat :**

```text
Alex, Bilal, Chloé
```

Le `if` sert uniquement à ne pas laisser une virgule à la fin. C'est un grand classique des bugs d'affichage.

Version fonctionnelle équivalente :

```dart
void main() {
  List<String> equipe = ['Alex', 'Bilal', 'Chloé'];

  print(equipe.join(', '));
}
```

**Résultat :**

```text
Alex, Bilal, Chloé
```

Sans argument, le séparateur est vide :

```dart
void main() {
  List<String> lettres = ['G', 'A', 'M', 'E'];

  print(lettres.join());
  print(lettres.join('-'));
  print(lettres.join(' | '));
}
```

**Résultat :**

```text
GAME
G-A-M-E
G | A | M | E
```

`join()` fonctionne aussi sur des collections qui ne contiennent pas de `String` : chaque élément est converti via sa méthode `toString()`.

```dart
void main() {
  List<int> scores = [120, 80, 340];

  print(scores.join(' + '));
}
```

**Résultat :**

```text
120 + 80 + 340
```

Combiné à `map()`, `join()` produit des affichages soignés, notamment multi-lignes avec `'\n'` :

```dart
class Joueur {
  final String nom;
  final int score;

  Joueur(this.nom, this.score);
}

void main() {
  List<Joueur> equipe = [
    Joueur('Alex', 120),
    Joueur('Bilal', 340),
  ];

  String tableau = equipe.map((j) => '${j.nom} : ${j.score} pts').join('\n');

  print(tableau);
}
```

**Résultat :**

```text
Alex : 120 pts
Bilal : 340 pts
```

---

## 14.25 — Le spread operator `...` et `...?`

Le spread operator « déverse » le contenu d'une collection à l'intérieur d'une autre littérale.

Version avec une boucle `for` :

```dart
void main() {
  List<String> baseInventaire = ['Potion', 'Épée'];
  List<String> butin = ['Arc', 'Flèche'];

  List<String> total = [];
  total.add('Sac de départ');
  for (final o in baseInventaire) {
    total.add(o);
  }
  for (final o in butin) {
    total.add(o);
  }

  print(total);
}
```

**Résultat :**

```text
[Sac de départ, Potion, Épée, Arc, Flèche]
```

Version fonctionnelle équivalente :

```dart
void main() {
  List<String> baseInventaire = ['Potion', 'Épée'];
  List<String> butin = ['Arc', 'Flèche'];

  List<String> total = ['Sac de départ', ...baseInventaire, ...butin];

  print(total);
}
```

**Résultat :**

```text
[Sac de départ, Potion, Épée, Arc, Flèche]
```

Bien distinguer les deux écritures suivantes :

```dart
void main() {
  List<String> a = ['Potion', 'Épée'];
  List<String> b = ['Arc'];

  print([a, b]);      // une liste de listes
  print([...a, ...b]); // une liste plate
}
```

**Résultat :**

```text
[[Potion, Épée], [Arc]]
[Potion, Épée, Arc]
```

Attention : `...` sur une collection qui vaut `null` fait planter le programme.

```dart
void main() {
  List<String>? butin;

  List<String> total = ['Potion', ...butin];

  print(total);
}
```

**Résultat :**

```text
Error: An expression whose value can be 'null' must be null-checked
before it can be dereferenced.
```

D'où la variante `...?`, le **spread nullable** : si la collection vaut `null`, elle est simplement ignorée.

```dart
void main() {
  List<String>? butin;
  List<String>? recompenses = ['Médaille'];

  List<String> total = ['Potion', ...?butin, ...?recompenses];

  print(total);
}
```

**Résultat :**

```text
[Potion, Médaille]
```

Le spread fonctionne aussi sur les `Set` et les `Map` :

```dart
void main() {
  Map<String, int> statsBase = {'force': 10, 'agilite': 8};
  Map<String, int> bonusArme = {'force': 15};

  Map<String, int> statsFinales = {...statsBase, ...bonusArme};

  print(statsFinales);
}
```

**Résultat :**

```text
{force: 15, agilite: 8}
```

Remarquez que `force` a été **écrasée** : dans une `Map`, la dernière valeur déversée l'emporte. C'est exactement le comportement voulu pour appliquer un bonus d'équipement.

---

## 14.26 — Collection `if`

Vous pouvez écrire un `if` **à l'intérieur** d'une liste littérale, pour n'inclure un élément que sous condition.

Version avec une boucle `for` (ou plutôt avec un `if` classique) :

```dart
void main() {
  bool aBouclier = true;
  bool aArc = false;

  List<String> equipement = ['Épée'];
  if (aBouclier) {
    equipement.add('Bouclier');
  }
  if (aArc) {
    equipement.add('Arc');
  }

  print(equipement);
}
```

**Résultat :**

```text
[Épée, Bouclier]
```

Version déclarative équivalente :

```dart
void main() {
  bool aBouclier = true;
  bool aArc = false;

  List<String> equipement = [
    'Épée',
    if (aBouclier) 'Bouclier',
    if (aArc) 'Arc',
  ];

  print(equipement);
}
```

**Résultat :**

```text
[Épée, Bouclier]
```

La liste est déclarée **complète du premier coup**, ce qui autorise le mot-clé `final` ou `const`, impossible avec la version précédente.

Le `if` de collection accepte un `else` :

```dart
void main() {
  int niveau = 1;

  List<String> menu = [
    'Continuer',
    if (niveau >= 10) 'Mode expert' else 'Mode découverte',
    'Quitter',
  ];

  print(menu);
}
```

**Résultat :**

```text
[Continuer, Mode découverte, Quitter]
```

Il fonctionne également dans les `Map` et les `Set` :

```dart
void main() {
  bool estPremium = true;

  Map<String, int> bonus = {
    'or': 100,
    if (estPremium) 'gemmes': 50,
  };

  print(bonus);
}
```

**Résultat :**

```text
{or: 100, gemmes: 50}
```

Et il se combine avec le spread, ce qui est très fréquent dans les interfaces :

```dart
void main() {
  List<String> objetsRares = ['Épée légendaire'];
  bool aTermineLeBoss = true;

  List<String> coffre = [
    'Or',
    if (aTermineLeBoss) ...objetsRares,
  ];

  print(coffre);
}
```

**Résultat :**

```text
[Or, Épée légendaire]
```

---

## 14.27 — Collection `for`

Sur le même principe, un `for` peut être écrit directement dans une littérale de collection.

Version avec une boucle `for` classique :

```dart
void main() {
  List<int> scores = [120, 80, 340];

  List<String> lignes = [];
  for (final s in scores) {
    lignes.add('Score : $s');
  }

  print(lignes);
}
```

**Résultat :**

```text
[Score : 120, Score : 80, Score : 340]
```

Version déclarative équivalente :

```dart
void main() {
  List<int> scores = [120, 80, 340];

  List<String> lignes = [
    for (final s in scores) 'Score : $s',
  ];

  print(lignes);
}
```

**Résultat :**

```text
[Score : 120, Score : 80, Score : 340]
```

Le `for` de collection joue ici le rôle de `map()`. Les deux écritures sont acceptables :

```dart
void main() {
  List<int> scores = [120, 80];

  print([for (final s in scores) s * 2]);
  print(scores.map((s) => s * 2).toList());
}
```

**Résultat :**

```text
[240, 160]
[240, 160]
```

Sa force apparaît quand on le combine avec un `if`, car il remplace alors `where()` **et** `map()` en une seule ligne, sans `toList()` :

```dart
class Joueur {
  final String nom;
  final int score;

  Joueur(this.nom, this.score);
}

void main() {
  List<Joueur> equipe = [
    Joueur('Alex', 120),
    Joueur('Bilal', 340),
    Joueur('Chloé', 80),
  ];

  List<String> elites = [
    for (final j in equipe)
      if (j.score >= 100) '${j.nom} (${j.score})',
  ];

  print(elites);
}
```

**Résultat :**

```text
[Alex (120), Bilal (340)]
```

La version classique avec compteur fonctionne aussi, ce qui est pratique pour numéroter :

```dart
void main() {
  List<String> podium = ['Bilal', 'Alex', 'Chloé'];

  List<String> classement = [
    for (int i = 0; i < podium.length; i++) '${i + 1}. ${podium[i]}',
  ];

  print(classement.join('\n'));
}
```

**Résultat :**

```text
1. Bilal
2. Alex
3. Chloé
```

Et on peut imbriquer deux `for` pour générer une grille de jeu :

```dart
void main() {
  List<String> cases = [
    for (int y = 0; y < 2; y++)
      for (int x = 0; x < 3; x++) '($x,$y)',
  ];

  print(cases);
}
```

**Résultat :**

```text
[(0,0), (1,0), (2,0), (0,1), (1,1), (2,1)]
```

---

## 14.28 — Chaîner les opérations (`where().map().toList()`)

Chaque méthode de ce chapitre renvoie une collection. On peut donc appeler la suivante directement dessus : c'est le **chaînage**.

L'objectif : à partir de l'équipe, obtenir les noms en majuscules des trois meilleurs joueurs connectés.

Version avec une boucle `for` :

```dart
class Joueur {
  final String nom;
  final int score;
  final bool enLigne;

  Joueur(this.nom, this.score, this.enLigne);
}

void main() {
  List<Joueur> equipe = [
    Joueur('Alex', 120, true),
    Joueur('Bilal', 340, false),
    Joueur('Chloé', 80, true),
    Joueur('Dan', 500, true),
    Joueur('Eva', 210, true),
  ];

  List<Joueur> connectes = [];
  for (final j in equipe) {
    if (j.enLigne) {
      connectes.add(j);
    }
  }

  connectes.sort((a, b) => b.score.compareTo(a.score));

  List<String> noms = [];
  for (int i = 0; i < connectes.length && i < 3; i++) {
    noms.add(connectes[i].nom.toUpperCase());
  }

  print(noms);
}
```

**Résultat :**

```text
[DAN, EVA, ALEX]
```

Version fonctionnelle équivalente :

```dart
class Joueur {
  final String nom;
  final int score;
  final bool enLigne;

  Joueur(this.nom, this.score, this.enLigne);
}

void main() {
  List<Joueur> equipe = [
    Joueur('Alex', 120, true),
    Joueur('Bilal', 340, false),
    Joueur('Chloé', 80, true),
    Joueur('Dan', 500, true),
    Joueur('Eva', 210, true),
  ];

  List<String> noms = (equipe.where((j) => j.enLigne).toList()
        ..sort((a, b) => b.score.compareTo(a.score)))
      .take(3)
      .map((j) => j.nom.toUpperCase())
      .toList();

  print(noms);
}
```

**Résultat :**

```text
[DAN, EVA, ALEX]
```

Cette chaîne se lit comme une phrase :

```text
  equipe
    .where(en ligne)          garder les connectés
    .toList() ..sort(score)   trier par score décroissant
    .take(3)                  garder les 3 premiers
    .map(nom en majuscules)   transformer en noms
    .toList()                 produire la liste finale
```

Deux règles pour que le chaînage reste lisible :

**Règle 1 — filtrer d'abord, transformer ensuite.** Il est inutile de transformer des éléments que vous allez jeter.

```dart
void main() {
  List<int> scores = [120, 80, 340, 55];

  // bien : on ne calcule que ce qui sert
  print(scores.where((s) => s > 100).map((s) => s * 2).toList());

  // moins bien : on double 4 valeurs pour n'en garder que 2
  print(scores.map((s) => s * 2).where((s) => s > 200).toList());
}
```

**Résultat :**

```text
[240, 680]
[240, 680]
```

**Règle 2 — un `toList()` à la fin, pas au milieu.** Les intermédiaires peuvent rester paresseux ; seul `sort()` exige une vraie `List`.

```dart
void main() {
  List<int> scores = [120, 80, 340, 55];

  // inutile : trois listes créées en mémoire
  final lourd = scores.where((s) => s > 60).toList().map((s) => s + 10).toList();

  // suffisant : une seule liste créée
  final leger = scores.where((s) => s > 60).map((s) => s + 10).toList();

  print(lourd);
  print(leger);
}
```

**Résultat :**

```text
[130, 90, 350]
[130, 90, 350]
```

Enfin, au-delà de trois maillons, coupez la chaîne en variables nommées :

```dart
class Joueur {
  final String nom;
  final int score;
  final bool enLigne;

  Joueur(this.nom, this.score, this.enLigne);
}

void main() {
  List<Joueur> equipe = [
    Joueur('Alex', 120, true),
    Joueur('Dan', 500, true),
    Joueur('Bilal', 340, false),
  ];

  final connectes = equipe.where((j) => j.enLigne).toList();
  final classement = connectes..sort((a, b) => b.score.compareTo(a.score));
  final podium = classement.take(3).map((j) => j.nom).toList();

  print(podium);
}
```

**Résultat :**

```text
[Dan, Alex]
```

Chaque ligne porte un nom, donc chaque étape s'explique toute seule.

---

## 14.29 — Lisibilité : quand revenir à une boucle `for`

Le style fonctionnel n'est pas toujours le meilleur choix. Voici cinq situations où la boucle `for` reste supérieure.

**1. Vous devez interrompre le parcours.** `forEach()` n'accepte ni `break` ni `continue`.

```dart
void main() {
  List<int> pvEnnemis = [40, 0, 25, 90];

  for (final pv in pvEnnemis) {
    if (pv == 0) {
      print('Ennemi mort rencontré, on arrête le scan.');
      break;
    }
    print('Ennemi vivant : $pv pv');
  }
}
```

**Résultat :**

```text
Ennemi vivant : 40 pv
Ennemi mort rencontré, on arrête le scan.
```

**2. Vous avez besoin de l'indice ET de la valeur.** Le style fonctionnel oblige alors à des acrobaties.

```dart
void main() {
  List<String> podium = ['Dan', 'Eva', 'Alex'];

  for (int i = 0; i < podium.length; i++) {
    print('${i + 1}. ${podium[i]}');
  }
}
```

**Résultat :**

```text
1. Dan
2. Eva
3. Alex
```

**3. Le traitement fait plusieurs choses en même temps.** Calculer un total, un maximum et un compte en un seul passage est plus clair avec une boucle qu'avec trois chaînes successives.

```dart
void main() {
  List<int> scores = [120, 80, 340, 55];

  int total = 0;
  int maximum = scores.first;
  int nbBons = 0;

  for (final s in scores) {
    total += s;
    if (s > maximum) maximum = s;
    if (s >= 100) nbBons++;
  }

  print('total : $total, max : $maximum, bons : $nbBons');
}
```

**Résultat :**

```text
total : 595, max : 340, bons : 2
```

**4. Le corps est long.** Une fonction anonyme de vingt lignes noyée au milieu d'une chaîne est illisible. Soit vous en faites une vraie fonction nommée, soit vous revenez à une boucle.

**5. Vous devez utiliser `await`.** Une opération asynchrone dans un `forEach()` ne s'attend pas correctement. Le chapitre 15 y reviendra ; retenez pour l'instant qu'un `for` est la seule forme sûre dans ce cas.

À l'inverse, le style fonctionnel gagne nettement quand la transformation est **simple et unique** :

```dart
void main() {
  List<int> scores = [120, 80, 340, 55];

  // clair
  print(scores.where((s) => s >= 100).toList());

  // même chose, trois fois plus long
  final r = <int>[];
  for (final s in scores) {
    if (s >= 100) r.add(s);
  }
  print(r);
}
```

**Résultat :**

```text
[120, 340]
[120, 340]
```

Le critère final tient en une question :

> **Un collègue comprend-il votre intention en une seule lecture ?** Si oui, gardez ce que vous avez écrit. Si non, changez de style, quel qu'il soit.

---

## 14.30 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `A value of type 'Iterable<int>' can't be assigned to 'List<int>'` | `map()` et `where()` renvoient un `Iterable`, pas une `List` | ajouter `.toList()` à la fin de la chaîne |
| `The operator '[]' isn't defined for the class 'Iterable'` | on tente `resultat[0]` sur un `Iterable` | convertir d'abord : `resultat.toList()[0]`, ou utiliser `.first` |
| `Bad state: No element` sur `firstWhere()` | aucun élément ne satisfait le prédicat | toujours fournir `orElse: () => valeurParDefaut` |
| `Bad state: Too many elements` sur `singleWhere()` | plusieurs éléments satisfont le prédicat | utiliser `firstWhere()` si l'unicité n'est pas garantie |
| `Bad state: No element` sur `reduce()` | la collection est vide | tester `isEmpty` avant, ou utiliser `fold()` avec une valeur de départ |
| `A value of type 'void' can't be assigned to 'List<int>'` | `sort()` et `forEach()` ne renvoient rien | trier sur place, ou `final t = List<int>.from(l)..sort();` |
| La liste d'origine est trouvée triée par surprise | `sort()` modifie la liste en place | trier une copie : `List<T>.from(liste)..sort(...)` |
| Le comparateur renvoie `bool` : `sort((a, b) => a > b)` | `sort()` attend un `int`, pas un `bool` | écrire `sort((a, b) => a.compareTo(b))` |
| Le tri alphabétique place `Zoé` avant `alex` | `compareTo()` compare les codes de caractères, majuscules d'abord | comparer en minuscules : `a.toLowerCase().compareTo(b.toLowerCase())` |
| Une transformation coûteuse s'exécute deux fois | un `Iterable` non matérialisé est recalculé à chaque parcours | stocker le résultat avec `.toList()` |
| `forEach()` ne construit aucune liste | `forEach()` a le type de retour `void` | utiliser `map()` pour produire une nouvelle collection |
| `contains(monObjet)` renvoie `false` alors que l'objet « existe » | `==` compare les identités par défaut | redéfinir `==` et `hashCode`, ou utiliser `any((o) => o.nom == ...)` |
| `An expression whose value can be 'null' must be null-checked` sur `...maListe` | le spread classique refuse `null` | utiliser le spread nullable `...?maListe` |
| `every()` renvoie `true` sur une liste vide et surprend | c'est la définition mathématique (vérité vide) | tester `liste.isNotEmpty && liste.every(...)` si nécessaire |
| Une liste de listes au lieu d'une liste plate | `map()` conserve la structure imbriquée | utiliser `expand()` pour aplatir |
| `break` refusé dans un `forEach()` | `forEach()` appelle une fonction, on ne peut pas en sortir | revenir à une boucle `for (final x in liste)` |

---

## 14.31 — Résumé du chapitre

| Méthode | Ce qu'elle retourne | Exemple |
| --- | --- | --- |
| `forEach()` | `void` (aucun résultat) | `sac.forEach(print)` |
| `map()` | `Iterable` de même taille | `scores.map((s) => s * 2).toList()` |
| `where()` | `Iterable` filtré | `scores.where((s) => s > 100).toList()` |
| `toList()` | `List` matérialisée | `iterable.toList()` |
| `toSet()` | `Set` sans doublon | `butin.toSet().toList()` |
| `firstWhere()` | le premier élément trouvé | `equipe.firstWhere((j) => j.score > 300, orElse: () => defaut)` |
| `lastWhere()` | le dernier élément trouvé | `degats.lastWhere((d) => d > 10, orElse: () => 0)` |
| `singleWhere()` | l'unique élément trouvé | `vague.singleWhere((e) => e.estBoss)` |
| `indexWhere()` | un `int` (`-1` si absent) | `sac.indexWhere((o) => o == 'Arc')` |
| `indexOf()` | un `int` (`-1` si absent) | `sac.indexOf('Épée')` |
| `any()` | un `bool` (au moins un) | `vague.any((e) => e.pv > 0)` |
| `every()` | un `bool` (tous) | `vague.every((e) => e.pv == 0)` |
| `contains()` | un `bool` (valeur présente) | `sac.contains('Potion')` |
| `reduce()` | un élément du même type | `scores.reduce((a, b) => a > b ? a : b)` |
| `fold()` | une valeur de type libre | `equipe.fold<int>(0, (t, j) => t + j.score)` |
| `sort()` | `void` — modifie la liste | `scores.sort((a, b) => b.compareTo(a))` |
| `compareTo()` | `int` négatif, nul ou positif | `a.score.compareTo(b.score)` |
| `List.from()` | une copie modifiable | `List<int>.from(scores)..sort()` |
| `take()` | `Iterable` des n premiers | `classement.take(3).toList()` |
| `skip()` | `Iterable` sans les n premiers | `classement.skip(3).toList()` |
| `expand()` | `Iterable` aplati | `equipe.expand((j) => j.sac).toList()` |
| `join()` | un `String` | `equipe.join(', ')` |
| `...` | déverse une collection | `[...base, ...butin]` |
| `...?` | déverse ou ignore si `null` | `[...?bonus]` |
| collection `if` | élément conditionnel | `['Épée', if (aBouclier) 'Bouclier']` |
| collection `for` | éléments générés | `[for (final j in equipe) j.nom]` |

Trois phrases à retenir :

- `map()` transforme, `where()` filtre, `fold()` résume.
- Tant qu'il n'y a pas de `toList()`, rien n'est calculé.
- `sort()` est la seule méthode du chapitre qui abîme la liste d'origine.

---

## 14.32 — Exercices

### Exercice 1 — Afficher l'inventaire (facile)

Vous disposez de `List<String> inventaire = ['Potion', 'Épée', 'Bouclier', 'Arc'];`.
Affichez chaque objet sur une ligne, précédé d'un tiret et d'une espace, en utilisant `forEach()`.

### Exercice 2 — Double XP (facile)

Vous disposez de `List<int> scores = [50, 120, 75];`.
Produisez avec `map()` une nouvelle liste contenant les scores doublés, puis affichez les deux listes pour vérifier que l'originale est intacte.

### Exercice 3 — Ennemis encore vivants (facile)

Une classe `Ennemi` a un `nom` et des `pv`. Créez quatre ennemis (dont deux à `0` pv) et affichez, avec `where()`, la liste de ceux qui sont encore vivants.

### Exercice 4 — Noms en majuscules (facile)

À partir d'une `List<Joueur>` (champ `nom`, champ `score`), produisez une `List<String>` contenant les noms en majuscules.

### Exercice 5 — Chercher un objet en boutique (moyen)

Une boutique contient des objets ayant un `nom` et un `prix`. Écrivez une recherche par nom avec `firstWhere()` qui ne plante pas quand l'objet n'existe pas : renvoyez un objet `Objet('introuvable', 0)`.

### Exercice 6 — Le niveau est-il terminé ? (moyen)

À partir d'une vague d'ennemis, affichez :
- s'il reste au moins un ennemi vivant (`any()`) ;
- si le niveau est terminé, c'est-à-dire si tous les ennemis sont à `0` pv (`every()`).
Testez sur deux vagues différentes.

### Exercice 7 — Total et maximum (moyen)

À partir de `List<int> pv = [40, 0, 25, 90];`, calculez le total avec `fold()` et le maximum avec `reduce()`.
Votre code doit fonctionner sans planter même si la liste est vide.

### Exercice 8 — Classement sans casser l'ordre d'arrivée (moyen)

À partir de `List<int> scores = [120, 80, 340];`, produisez une liste triée par ordre décroissant **sans** modifier `scores`. Affichez les deux listes.

### Exercice 9 — Butin de l'équipe (moyen)

Chaque joueur possède un `sac` (`List<String>`). Rassemblez tous les objets de l'équipe avec `expand()`, puis affichez le nombre total d'objets et le nombre de types distincts avec `toSet()`.

### Exercice 10 — Feuille de scores (moyen)

À partir d'une `List<Joueur>`, produisez un unique `String` où chaque ligne vaut `- Nom : X pts`, en combinant `map()` et `join('\n')`. Affichez-le en un seul `print`.

### Exercice 11 — Équipement dynamique (avancé)

Construisez une liste d'équipement en une seule littérale, contenant :
- toujours `'Épée'` ;
- `'Bouclier'` seulement si `aBouclier` vaut `true` ;
- toutes les potions d'une liste `potions` (spread) ;
- les objets d'une liste `bonus` qui peut valoir `null` (spread nullable).
Puis produisez, avec un collection `for`, la même liste en majuscules.

### Exercice 12 — Mini-projet : tableau des scores (projet)

Vous disposez d'une liste de joueurs ayant un `nom`, un `score` et un booléen `actif`.

Écrivez un programme qui :
1. ne garde que les joueurs actifs ;
2. calcule le nombre de joueurs retenus, le total des points et la moyenne (avec deux décimales) ;
3. identifie le meilleur et le plus faible joueur ;
4. affiche un classement numéroté par score décroissant, aligné en colonnes ;
5. affiche la liste des joueurs au-dessus de la moyenne, séparés par des virgules.

Le programme ne doit **jamais** modifier la liste d'origine et ne doit pas planter si aucun joueur n'est actif.

---

## 14.33 — Corrections des exercices

### Correction 1

```dart
void main() {
  List<String> inventaire = ['Potion', 'Épée', 'Bouclier', 'Arc'];

  inventaire.forEach((objet) => print('- $objet'));
}
```

**Résultat :**

```text
- Potion
- Épée
- Bouclier
- Arc
```

**Explication :** `forEach()` reçoit une fonction fléchée qui est appelée une fois par élément. Le paramètre `objet` prend successivement chaque valeur de la liste. On n'écrit ni compteur ni `inventaire[i]`, donc aucune erreur d'indice n'est possible. Notez qu'on ne peut rien affecter au résultat de `forEach()` : il ne renvoie rien.

---

### Correction 2

```dart
void main() {
  List<int> scores = [50, 120, 75];

  List<int> doubles = scores.map((s) => s * 2).toList();

  print('originaux : $scores');
  print('doublés   : $doubles');
}
```

**Résultat :**

```text
originaux : [50, 120, 75]
doublés   : [100, 240, 150]
```

**Explication :** `map()` applique `(s) => s * 2` à chacun des trois éléments et produit un `Iterable` de trois éléments. `toList()` matérialise ce résultat en `List<int>`, ce qui est indispensable ici puisque la variable est déclarée `List<int>`. La liste `scores` n'est pas touchée : le style fonctionnel ne modifie jamais la source, il en dérive une nouvelle collection.

---

### Correction 3

```dart
class Ennemi {
  final String nom;
  final int pv;

  Ennemi(this.nom, this.pv);

  @override
  String toString() => '$nom($pv pv)';
}

void main() {
  List<Ennemi> vague = [
    Ennemi('gobelin', 0),
    Ennemi('orc', 25),
    Ennemi('troll', 0),
    Ennemi('dragon', 300),
  ];

  List<Ennemi> vivants = vague.where((e) => e.pv > 0).toList();

  print('vague   : $vague');
  print('vivants : $vivants');
  print('nombre  : ${vivants.length}');
}
```

**Résultat :**

```text
vague   : [gobelin(0 pv), orc(25 pv), troll(0 pv), dragon(300 pv)]
vivants : [orc(25 pv), dragon(300 pv)]
nombre  : 2
```

**Explication :** le prédicat `(e) => e.pv > 0` renvoie un `bool` : c'est l'exigence de `where()`. Les éléments retenus gardent leur type `Ennemi`, seul leur nombre change. La méthode `toString()` redéfinie permet d'afficher directement la liste sans écrire de boucle d'affichage.

---

### Correction 4

```dart
class Joueur {
  final String nom;
  final int score;

  Joueur(this.nom, this.score);
}

void main() {
  List<Joueur> equipe = [
    Joueur('Alex', 120),
    Joueur('Bilal', 340),
    Joueur('Chloé', 80),
  ];

  List<String> noms = equipe.map((j) => j.nom.toUpperCase()).toList();

  print(noms);
}
```

**Résultat :**

```text
[ALEX, BILAL, CHLOÉ]
```

**Explication :** c'est l'exemple type du changement de type par `map()` : on entre avec une `List<Joueur>` et on ressort avec une `List<String>`. Dart déduit seul le type de sortie à partir de l'expression `j.nom.toUpperCase()`, qui est un `String`. Aucune annotation `map<String>` n'est donc nécessaire.

---

### Correction 5

```dart
class Objet {
  final String nom;
  final int prix;

  Objet(this.nom, this.prix);

  @override
  String toString() => '$nom ($prix or)';
}

Objet chercher(List<Objet> boutique, String nom) {
  return boutique.firstWhere(
    (o) => o.nom == nom,
    orElse: () => Objet('introuvable', 0),
  );
}

void main() {
  List<Objet> boutique = [
    Objet('Potion', 50),
    Objet('Épée', 200),
    Objet('Bouclier', 150),
  ];

  print(chercher(boutique, 'Épée'));
  print(chercher(boutique, 'Arc'));
}
```

**Résultat :**

```text
Épée (200 or)
introuvable (0 or)
```

**Explication :** sans `orElse`, le second appel lancerait `Bad state: No element` et arrêterait le programme. `orElse` attend une fonction **sans paramètre** qui fabrique la valeur de repli : d'où l'écriture `() => Objet(...)` et non `Objet(...)` seul. Cet « objet nul » est un motif classique : il évite de propager un `null` dans tout le reste du code.

---

### Correction 6

```dart
class Ennemi {
  final String nom;
  final int pv;

  Ennemi(this.nom, this.pv);
}

void analyser(String titre, List<Ennemi> vague) {
  bool combatEnCours = vague.any((e) => e.pv > 0);
  bool niveauTermine = vague.every((e) => e.pv == 0);

  print('$titre');
  print('  au moins un vivant : $combatEnCours');
  print('  niveau terminé     : $niveauTermine');
}

void main() {
  analyser('Vague 1', [
    Ennemi('gobelin', 0),
    Ennemi('orc', 15),
    Ennemi('troll', 0),
  ]);

  analyser('Vague 2', [
    Ennemi('gobelin', 0),
    Ennemi('orc', 0),
  ]);
}
```

**Résultat :**

```text
Vague 1
  au moins un vivant : true
  niveau terminé     : false
Vague 2
  au moins un vivant : false
  niveau terminé     : true
```

**Explication :** `any()` et `every()` sont des questions fermées : leur résultat est un `bool`, jamais une collection. `any()` s'interrompt au premier succès, `every()` au premier échec, ce qui les rend très rapides. Ici, les deux réponses sont opposées, mais ce n'est vrai que parce que les conditions `pv > 0` et `pv == 0` sont complémentaires.

---

### Correction 7

```dart
void resumer(String titre, List<int> pv) {
  int total = pv.fold<int>(0, (somme, v) => somme + v);
  int maximum = pv.isEmpty ? 0 : pv.reduce((a, b) => a > b ? a : b);

  print('$titre -> total : $total, maximum : $maximum');
}

void main() {
  resumer('Vague pleine', [40, 0, 25, 90]);
  resumer('Vague vide', []);
}
```

**Résultat :**

```text
Vague pleine -> total : 155, maximum : 90
Vague vide -> total : 0, maximum : 0
```

**Explication :** `fold()` reçoit la valeur de départ `0`, donc il fonctionne même sur une liste vide et renvoie `0`. `reduce()`, lui, n'a pas de valeur de départ : sur une liste vide il lancerait `Bad state: No element`. Le test `pv.isEmpty ? 0 : ...` est donc obligatoire. C'est exactement la différence expliquée en 14.16 : `fold()` par défaut, `reduce()` pour le min et le max avec une protection.

---

### Correction 8

```dart
void main() {
  List<int> scores = [120, 80, 340];

  List<int> classement = List<int>.from(scores)
    ..sort((a, b) => b.compareTo(a));

  print('ordre d\'arrivée : $scores');
  print('classement      : $classement');
}
```

**Résultat :**

```text
ordre d'arrivée : [120, 80, 340]
classement      : [340, 120, 80]
```

**Explication :** `List<int>.from(scores)` crée une nouvelle liste contenant les mêmes valeurs. L'opérateur cascade `..` applique `sort()` à cette copie et renvoie la copie elle-même, ce qui permet l'affectation en une seule expression. Sans cascade, `List<int>.from(scores).sort()` renverrait `null`. Le comparateur `(a, b) => b.compareTo(a)` inverse l'ordre naturel : les grandes valeurs passent devant.

---

### Correction 9

```dart
class Joueur {
  final String nom;
  final List<String> sac;

  Joueur(this.nom, this.sac);
}

void main() {
  List<Joueur> equipe = [
    Joueur('Alex', ['Potion', 'Épée']),
    Joueur('Bilal', ['Potion', 'Arc']),
    Joueur('Chloé', ['Épée']),
  ];

  List<String> butin = equipe.expand((j) => j.sac).toList();
  List<String> types = butin.toSet().toList();

  print('butin complet   : $butin');
  print('objets ramassés : ${butin.length}');
  print('types distincts : ${types.length} -> $types');
}
```

**Résultat :**

```text
butin complet   : [Potion, Épée, Potion, Arc, Épée]
objets ramassés : 5
types distincts : 3 -> [Potion, Épée, Arc]
```

**Explication :** `expand()` remplace chaque joueur par le contenu de son sac, puis colle toutes ces listes bout à bout : on obtient une liste plate de cinq `String`. Avec `map((j) => j.sac)`, on aurait obtenu une `List<List<String>>`, ce qui n'est pas ce que l'on veut. `toSet()` supprime ensuite les doublons en conservant l'ordre de première apparition.

---

### Correction 10

```dart
class Joueur {
  final String nom;
  final int score;

  Joueur(this.nom, this.score);
}

void main() {
  List<Joueur> equipe = [
    Joueur('Alex', 120),
    Joueur('Bilal', 340),
    Joueur('Chloé', 80),
  ];

  String feuille = equipe.map((j) => '- ${j.nom} : ${j.score} pts').join('\n');

  print(feuille);
}
```

**Résultat :**

```text
- Alex : 120 pts
- Bilal : 340 pts
- Chloé : 80 pts
```

**Explication :** `map()` transforme chaque `Joueur` en une ligne de texte, puis `join('\n')` assemble ces lignes en un seul `String` séparé par des retours à la ligne. Cette combinaison remplace avantageusement la boucle avec concaténation et test de dernier élément : il n'y a plus de séparateur en trop à la fin.

---

### Correction 11

```dart
void main() {
  bool aBouclier = true;
  List<String> potions = ['Potion', 'Potion'];
  List<String>? bonus;

  List<String> equipement = [
    'Épée',
    if (aBouclier) 'Bouclier',
    ...potions,
    ...?bonus,
  ];

  List<String> enMajuscules = [
    for (final o in equipement) o.toUpperCase(),
  ];

  print(equipement);
  print(enMajuscules);
}
```

**Résultat :**

```text
[Épée, Bouclier, Potion, Potion]
[ÉPÉE, BOUCLIER, POTION, POTION]
```

**Explication :** la liste est déclarée complète en une seule expression. Le collection `if` insère `'Bouclier'` uniquement si la condition est vraie. `...potions` déverse les deux potions. `...?bonus` ne provoque aucune erreur bien que `bonus` vaille `null` : le spread nullable ignore simplement la collection absente. Avec `...bonus`, le code ne compilerait pas. Le collection `for` joue enfin le rôle de `map()`, sans `toList()` puisque le résultat est déjà une littérale de liste.

---

### Correction 12

```dart
class Joueur {
  final String nom;
  final int score;
  final bool actif;

  Joueur(this.nom, this.score, this.actif);
}

void afficherTableau(List<Joueur> joueurs) {
  final actifs = joueurs.where((j) => j.actif).toList();

  if (actifs.isEmpty) {
    print('=== TABLEAU DES SCORES ===');
    print('Aucun joueur actif.');
    return;
  }

  final total = actifs.fold<int>(0, (somme, j) => somme + j.score);
  final moyenne = total / actifs.length;

  final classement = List<Joueur>.from(actifs)
    ..sort((a, b) => b.score.compareTo(a.score));

  final meilleur = classement.first;
  final plusFaible = classement.last;

  final auDessus =
      classement.where((j) => j.score > moyenne).map((j) => j.nom).toList();

  print('=== TABLEAU DES SCORES ===');
  print('Joueurs actifs : ${actifs.length}');
  print('Total          : $total');
  print('Moyenne        : ${moyenne.toStringAsFixed(2)}');
  print('Meilleur       : ${meilleur.nom} (${meilleur.score})');
  print('Plus faible    : ${plusFaible.nom} (${plusFaible.score})');
  print('');
  print('--- Classement ---');

  final lignes = [
    for (int i = 0; i < classement.length; i++)
      '${i + 1}. ${classement[i].nom.padRight(12)}'
          '${classement[i].score.toString().padLeft(4)}',
  ];
  print(lignes.join('\n'));

  print('');
  print('Au-dessus de la moyenne : ${auDessus.join(', ')}');
}

void main() {
  final joueurs = [
    Joueur('Alex', 120, true),
    Joueur('Bilal', 340, true),
    Joueur('Chloé', 80, false),
    Joueur('Dan', 500, true),
    Joueur('Eva', 210, true),
    Joueur('Farid', 45, true),
  ];

  afficherTableau(joueurs);

  print('');
  print('Ordre d\'origine intact : ${joueurs.map((j) => j.nom).toList()}');

  print('');
  afficherTableau([Joueur('Zoé', 10, false)]);
}
```

**Résultat :**

```text
=== TABLEAU DES SCORES ===
Joueurs actifs : 5
Total          : 1215
Moyenne        : 243.00
Meilleur       : Dan (500)
Plus faible    : Farid (45)

--- Classement ---
1. Dan          500
2. Bilal        340
3. Eva          210
4. Alex         120
5. Farid         45

Au-dessus de la moyenne : Dan, Bilal

Ordre d'origine intact : [Alex, Bilal, Chloé, Dan, Eva, Farid]

=== TABLEAU DES SCORES ===
Aucun joueur actif.
```

**Explication :** ce mini-projet enchaîne presque toutes les notions du chapitre.

- `where((j) => j.actif).toList()` filtre les joueurs retenus. Chloé, inactive, disparaît de tous les calculs.
- Le test `actifs.isEmpty` protège la suite : sans lui, `classement.first` lancerait `Bad state: No element`. C'est le réflexe à prendre avant tout `first`, `last` ou `reduce`.
- `fold<int>(0, ...)` calcule le total. On préfère `fold()` à `reduce()` car il fonctionne quelle que soit la taille de la liste et parce qu'il transforme des `Joueur` en `int`, ce que `reduce()` ne sait pas faire.
- `moyenne` est un `double` : la division `/` en Dart renvoie toujours un `double`. `toStringAsFixed(2)` l'affiche avec deux décimales.
- `List<Joueur>.from(actifs)..sort(...)` trie une **copie**. La liste `joueurs` reçue en paramètre n'est jamais modifiée, ce que prouve la dernière ligne du programme.
- Une fois la liste triée par score décroissant, `first` donne le meilleur et `last` le plus faible : inutile d'appeler `reduce()` deux fois de plus.
- Le collection `for` avec compteur produit les lignes numérotées ; `padRight(12)` et `padLeft(4)` alignent les colonnes ; `join('\n')` assemble le tout pour un seul `print`.
- `auDessus` réutilise le classement déjà trié, donc les noms sortent eux aussi du meilleur au moins bon.

---

## Et maintenant ?

Vous savez désormais traiter une collection sans écrire une seule boucle. `map()` transforme, `where()` filtre, `fold()` résume, `sort()` classe, et le spread, le collection `if` et le collection `for` vous permettent de construire une liste complète en une seule expression. Vous connaissez aussi les trois pièges qui font perdre le plus de temps aux débutants : l'oubli de `toList()`, le `firstWhere()` sans `orElse`, et le `sort()` qui abîme la liste d'origine.

Jusqu'ici, tout votre code s'exécutait **immédiatement** : chaque ligne se terminait avant que la suivante ne commence. Dans un vrai jeu, ce n'est plus vrai. Charger une sauvegarde, télécharger le classement mondial, attendre la fin d'une animation ou lire un fichier prend du temps. Si le programme s'arrêtait à chaque fois, l'image se figerait.

Le chapitre suivant introduit la programmation asynchrone : `Future`, `async`, `await`, la gestion des erreurs asynchrones et les `Stream`. Vous y verrez pourquoi un `await` dans un `forEach()` ne fonctionne pas, et comment écrire correctement une boucle qui attend.

Rendez-vous au chapitre 15 : [15-PARTIE-1A—PROGRAMMATION-ASYNCHRONE-FUTURE-ASYNC-AWAIT.md](15-PARTIE-1A—PROGRAMMATION-ASYNCHRONE-FUTURE-ASYNC-AWAIT.md)
