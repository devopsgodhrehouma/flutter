# PARTIE 1A — DART
# CHAPITRE 06 — COLLECTIONS : `List`, `Set` ET `Map`

> **Niveau :** débutant
> **Durée estimée :** 6 h
> **Pré-requis :** chapitre 05 — Les boucles (`for`, `while`, `do while`)
> **Ce que vous saurez faire à la fin :** créer, parcourir et manipuler des `List`, des `Set` et des `Map`, y compris des collections imbriquées proches de ce que renvoie une API, et construire un inventaire de jeu complet.

---

## 06.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- comprendre ce qu'est une collection ;
- stocker plusieurs valeurs dans une seule variable ;
- créer une `List` ;
- accéder aux éléments d'une liste ;
- modifier une liste ;
- ajouter et supprimer des éléments ;
- parcourir une liste avec `for` ;
- parcourir une liste avec `for-in` ;
- utiliser `length`, `first`, `last`, `isEmpty` et `isNotEmpty` ;
- utiliser `contains()` et `indexOf()` ;
- créer un `Set` ;
- comprendre l'unicité des valeurs dans un `Set` ;
- effectuer des unions, intersections et différences ;
- créer une `Map` ;
- comprendre le principe clé/valeur ;
- lire, ajouter, modifier et supprimer une entrée d'une `Map` ;
- parcourir une `Map` ;
- créer des collections imbriquées ;
- manipuler des structures proches de celles reçues depuis une API JSON ;
- créer un inventaire simple de jeu.

---

## 06.1 — Pourquoi utiliser des collections ?

Jusqu'à maintenant, si nous voulions stocker plusieurs joueurs, nous pouvions écrire :

```dart
String joueur1 = 'Alex';
String joueur2 = 'Sophie';
String joueur3 = 'Samir';
String joueur4 = 'Maria';
```

Cela fonctionne.

Mais si nous avons :

```text
100 joueurs
```

cette solution devient rapidement impossible à maintenir.

Nous voulons plutôt une seule structure contenant plusieurs valeurs :

```dart
List<String> joueurs = [
  'Alex',
  'Sophie',
  'Samir',
  'Maria',
];
```

Nous pouvons représenter cette liste ainsi :

```text
joueurs
│
├── Alex
├── Sophie
├── Samir
└── Maria
```

---

## 06.2 — Les trois collections principales

Dans ce chapitre, nous allons étudier :

```text
List
Set
Map
```

Leur rôle est différent.

| Collection | Utilité principale |
|---|---|
| `List` | Liste ordonnée d'éléments |
| `Set` | Ensemble de valeurs uniques |
| `Map` | Association clé → valeur |

---

## 06.3 — Première collection : `List`

Une `List` permet de stocker plusieurs éléments dans un ordre précis.

Exemple :

```dart
List<String> joueurs = [
  'Alex',
  'Sophie',
  'Samir',
];
```

Ici :

```text
List
```

indique que nous utilisons une liste.

Et :

```text
<String>
```

indique que cette liste contient uniquement des chaînes de caractères.

---

## 06.4 — Première liste complète

```dart
void main() {
  List<String> joueurs = [
    'Alex',
    'Sophie',
    'Samir',
  ];

  print(joueurs);
}
```

Résultat :

```text
[Alex, Sophie, Samir]
```

---

## 06.5 — Liste de nombres

Nous pouvons également créer :

```dart
List<int> scores = [
  100,
  250,
  500,
  750,
];
```

Programme :

```dart
void main() {
  List<int> scores = [
    100,
    250,
    500,
    750,
  ];

  print(scores);
}
```

---

## 06.6 — Liste de nombres décimaux

```dart
List<double> prix = [
  9.99,
  15.50,
  29.99,
];
```

---

## 06.7 — Liste de booléens

```dart
List<bool> etats = [
  true,
  false,
  true,
];
```

---

## 06.8 — Inférence de type avec `var`

Dart peut aussi déduire le type.

Exemple :

```dart
var joueurs = [
  'Alex',
  'Sophie',
  'Samir',
];
```

Dart comprend que nous avons une :

```text
List<String>
```

---

## 06.9 — L'index d'une liste

Une liste possède des positions appelées :

> index.

Attention :

> le premier index est 0.

Exemple :

```dart
List<String> joueurs = [
  'Alex',
  'Sophie',
  'Samir',
];
```

Représentation :

```text
Index     Valeur

0         Alex
1         Sophie
2         Samir
```

Donc :

```dart
joueurs[0]
```

correspond à :

```text
Alex
```

---

## 06.10 — Accéder à un élément

```dart
void main() {
  List<String> joueurs = [
    'Alex',
    'Sophie',
    'Samir',
  ];

  print(joueurs[0]);
  print(joueurs[1]);
  print(joueurs[2]);
}
```

Résultat :

```text
Alex
Sophie
Samir
```

---

## 06.11 — Attention aux index

Pour une liste de 3 éléments :

```text
Index valides :

0
1
2
```

L'index :

```text
3
```

n'existe pas.

Donc :

```dart
print(joueurs[3]);
```

provoquera une erreur à l'exécution.

---

## 06.12 — Comprendre `length`

La propriété :

```dart
length
```

indique le nombre d'éléments.

Exemple :

```dart
void main() {
  List<String> joueurs = [
    'Alex',
    'Sophie',
    'Samir',
  ];

  print(joueurs.length);
}
```

Résultat :

```text
3
```

---

## 06.13 — Nombre d'éléments vs dernier index

C'est très important.

Si :

```dart
joueurs.length
```

vaut :

```text
3
```

les index valides sont :

```text
0
1
2
```

Le dernier index correspond donc généralement à :

```dart
joueurs.length - 1
```

---

## 06.14 — Accéder au dernier élément

```dart
void main() {
  List<String> joueurs = [
    'Alex',
    'Sophie',
    'Samir',
  ];

  print(joueurs[joueurs.length - 1]);
}
```

Résultat :

```text
Samir
```

Mais Dart fournit aussi une syntaxe plus simple :

```dart
print(joueurs.last);
```

---

## 06.15 — `first` et `last`

```dart
void main() {
  List<String> joueurs = [
    'Alex',
    'Sophie',
    'Samir',
  ];

  print(joueurs.first);
  print(joueurs.last);
}
```

Résultat :

```text
Alex
Samir
```

---

## 06.16 — `isEmpty`

```dart
List<String> joueurs = [];
```

Nous pouvons tester :

```dart
print(joueurs.isEmpty);
```

Résultat :

```text
true
```

---

## 06.17 — `isNotEmpty`

```dart
List<String> joueurs = [
  'Alex',
];
```

Puis :

```dart
print(joueurs.isNotEmpty);
```

Résultat :

```text
true
```

---

## 06.18 — Modifier un élément

Considérons :

```dart
List<String> joueurs = [
  'Alex',
  'Sophie',
  'Samir',
];
```

Nous pouvons remplacer :

```text
Sophie
```

par :

```text
Maria
```

avec :

```dart
joueurs[1] = 'Maria';
```

---

## 06.19 — Exemple complet

```dart
void main() {
  List<String> joueurs = [
    'Alex',
    'Sophie',
    'Samir',
  ];

  joueurs[1] = 'Maria';

  print(joueurs);
}
```

Résultat :

```text
[Alex, Maria, Samir]
```

---

## 06.20 — Ajouter un élément avec `add()`

```dart
void main() {
  List<String> joueurs = [
    'Alex',
    'Sophie',
  ];

  joueurs.add('Samir');

  print(joueurs);
}
```

Résultat :

```text
[Alex, Sophie, Samir]
```

---

## 06.21 — Ajouter plusieurs éléments avec `addAll()`

```dart
void main() {
  List<String> joueurs = [
    'Alex',
  ];

  joueurs.addAll([
    'Sophie',
    'Samir',
    'Maria',
  ]);

  print(joueurs);
}
```

---

## 06.22 — Ajouter à une position précise avec `insert()`

```dart
void main() {
  List<String> joueurs = [
    'Alex',
    'Samir',
  ];

  joueurs.insert(1, 'Sophie');

  print(joueurs);
}
```

Résultat :

```text
[Alex, Sophie, Samir]
```

---

## 06.23 — Supprimer avec `remove()`

```dart
void main() {
  List<String> joueurs = [
    'Alex',
    'Sophie',
    'Samir',
  ];

  joueurs.remove('Sophie');

  print(joueurs);
}
```

Résultat :

```text
[Alex, Samir]
```

---

## 06.24 — Supprimer avec `removeAt()`

```dart
void main() {
  List<String> joueurs = [
    'Alex',
    'Sophie',
    'Samir',
  ];

  joueurs.removeAt(1);

  print(joueurs);
}
```

Résultat :

```text
[Alex, Samir]
```

---

## 06.25 — `removeLast()`

```dart
void main() {
  List<String> joueurs = [
    'Alex',
    'Sophie',
    'Samir',
  ];

  joueurs.removeLast();

  print(joueurs);
}
```

Résultat :

```text
[Alex, Sophie]
```

---

## 06.26 — Vider une liste avec `clear()`

```dart
void main() {
  List<String> joueurs = [
    'Alex',
    'Sophie',
  ];

  joueurs.clear();

  print(joueurs);
}
```

Résultat :

```text
[]
```

---

## 06.27 — Vérifier la présence avec `contains()`

```dart
void main() {
  List<String> joueurs = [
    'Alex',
    'Sophie',
    'Samir',
  ];

  print(joueurs.contains('Sophie'));
}
```

Résultat :

```text
true
```

---

## 06.28 — Rechercher la position avec `indexOf()`

```dart
void main() {
  List<String> joueurs = [
    'Alex',
    'Sophie',
    'Samir',
  ];

  int position = joueurs.indexOf('Sophie');

  print(position);
}
```

Résultat :

```text
1
```

---

## 06.29 — Si l'élément n'existe pas

```dart
print(joueurs.indexOf('Maria'));
```

retourne :

```text
-1
```

Cela signifie :

```text
élément introuvable
```

---

## 06.30 — Parcourir une liste avec `for`

Nous avons appris :

```dart
for
```

au chapitre précédent.

Nous pouvons maintenant combiner les deux concepts.

```dart
void main() {
  List<String> joueurs = [
    'Alex',
    'Sophie',
    'Samir',
  ];

  for (int i = 0; i < joueurs.length; i++) {
    print(joueurs[i]);
  }
}
```

---

## 06.31 — Pourquoi `i < joueurs.length` ?

Si :

```text
length = 3
```

les index sont :

```text
0
1
2
```

Donc :

```dart
i < joueurs.length
```

est exactement ce qu'il faut.

Si nous écrivions :

```dart
i <= joueurs.length
```

nous finirions par essayer :

```dart
joueurs[3]
```

qui n'existe pas.

---

## 06.32 — Afficher l'index et la valeur

```dart
void main() {
  List<String> joueurs = [
    'Alex',
    'Sophie',
    'Samir',
  ];

  for (int i = 0; i < joueurs.length; i++) {
    print('Index $i : ${joueurs[i]}');
  }
}
```

Résultat :

```text
Index 0 : Alex
Index 1 : Sophie
Index 2 : Samir
```

---

## 06.33 — La boucle `for-in`

Dart propose une syntaxe plus simple lorsqu'on veut directement parcourir les valeurs.

```dart
void main() {
  List<String> joueurs = [
    'Alex',
    'Sophie',
    'Samir',
  ];

  for (String joueur in joueurs) {
    print(joueur);
  }
}
```

---

## 06.34 — Comprendre `for-in`

```dart
for (String joueur in joueurs)
```

signifie :

> Pour chaque joueur contenu dans la liste joueurs...

Puis :

```dart
print(joueur);
```

affiche le joueur actuel.

---

## 06.35 — `for` ou `for-in` ?

Utilisez `for` lorsque vous avez besoin de l'index.

```dart
for (int i = 0; i < joueurs.length; i++) {
  print('$i : ${joueurs[i]}');
}
```

Utilisez `for-in` lorsque vous avez seulement besoin des valeurs.

```dart
for (String joueur in joueurs) {
  print(joueur);
}
```

---

## 06.36 — Liste de scores

```dart
void main() {
  List<int> scores = [
    500,
    1200,
    850,
    2000,
  ];

  for (int score in scores) {
    print('Score : $score');
  }
}
```

---

## 06.37 — Calculer un total

```dart
void main() {
  List<int> scores = [
    500,
    1200,
    850,
    2000,
  ];

  int total = 0;

  for (int score in scores) {
    total += score;
  }

  print('Total : $total');
}
```

---

## 06.38 — Calculer une moyenne

```dart
void main() {
  List<double> notes = [
    75,
    80,
    95,
    90,
  ];

  double total = 0;

  for (double note in notes) {
    total += note;
  }

  double moyenne = total / notes.length;

  print('Moyenne : $moyenne');
}
```

---

## 06.39 — Trouver une valeur maximale

Approche simple :

```dart
void main() {
  List<int> scores = [
    500,
    1200,
    850,
    2000,
  ];

  int meilleurScore = scores[0];

  for (int score in scores) {
    if (score > meilleurScore) {
      meilleurScore = score;
    }
  }

  print('Meilleur score : $meilleurScore');
}
```

---

## 06.40 — Filtrer manuellement avec une condition

```dart
void main() {
  List<int> scores = [
    200,
    1500,
    700,
    2200,
  ];

  for (int score in scores) {
    if (score >= 1000) {
      print('Score élevé : $score');
    }
  }
}
```

Nous verrons plus tard une méthode beaucoup plus élégante :

```dart
where()
```

---

## 06.41 — Liste d'inventaire

```dart
void main() {
  List<String> inventaire = [
    'Épée',
    'Potion',
    'Clé',
  ];

  print('Inventaire :');

  for (String objet in inventaire) {
    print('- $objet');
  }
}
```

---

## 06.42 — Ajouter un objet à l'inventaire

```dart
inventaire.add('Bouclier');
```

---

## 06.43 — Retirer un objet

```dart
inventaire.remove('Potion');
```

---

## 06.44 — Vérifier si le joueur possède une clé

```dart
if (inventaire.contains('Clé')) {
  print('La porte peut être ouverte.');
}
```

Cela ressemble déjà beaucoup à de la logique de jeu réelle.

---

## 06.45 — Listes imbriquées

Une liste peut contenir d'autres listes.

Exemple :

```dart
List<List<int>> grille = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9],
];
```

Nous pouvons visualiser :

```text
1 2 3
4 5 6
7 8 9
```

---

## 06.46 — Accéder à une liste imbriquée

```dart
print(grille[0]);
```

Résultat :

```text
[1, 2, 3]
```

---

## 06.47 — Accéder à une case

```dart
print(grille[0][0]);
```

Résultat :

```text
1
```

Puis :

```dart
print(grille[1][2]);
```

Résultat :

```text
6
```

---

## 06.48 — Comprendre les deux index

Dans :

```dart
grille[1][2]
```

le premier index correspond à la :

```text
ligne
```

Le deuxième correspond à la :

```text
colonne
```

---

## 06.49 — Parcourir une grille

```dart
void main() {
  List<List<int>> grille = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
  ];

  for (int ligne = 0; ligne < grille.length; ligne++) {
    for (
      int colonne = 0;
      colonne < grille[ligne].length;
      colonne++
    ) {
      print(
        'Case [$ligne][$colonne] = ${grille[ligne][colonne]}',
      );
    }
  }
}
```

---

## 06.50 — Pourquoi les grilles sont intéressantes ?

Elles peuvent représenter :

```text
plateau de jeu
labyrinthe
carte
terrain
matrice
grille de cases
inventaire organisé
```

Dans un jeu futur :

```text
0 → sol
1 → mur
2 → joueur
3 → ennemi
4 → trésor
```

---

## 06.51 — Exemple de mini-carte

```dart
List<List<int>> carte = [
  [1, 1, 1, 1],
  [1, 2, 0, 1],
  [1, 0, 3, 1],
  [1, 1, 1, 1],
];
```

On pourrait décider :

```text
0 = vide
1 = mur
2 = joueur
3 = ennemi
```

Nous reviendrons sur ce type de structure lors de la création du jeu.

---

## 06.52 — Deuxième collection : `Set`

Un `Set` ressemble à une liste, mais il possède une caractéristique fondamentale :

> les valeurs sont uniques.

Exemple :

```dart
Set<String> competences = {
  'Dart',
  'Flutter',
  'Dart',
};
```

Le deuxième :

```text
Dart
```

n'est pas conservé comme doublon.

---

## 06.53 — Premier exemple `Set`

```dart
void main() {
  Set<String> langages = {
    'Dart',
    'Python',
    'Java',
    'Dart',
  };

  print(langages);
}
```

Le résultat contient une seule occurrence de :

```text
Dart
```

---

## 06.54 — Pourquoi utiliser un `Set` ?

Un `Set` est utile lorsque les doublons n'ont aucun sens.

Exemples :

```text
compétences
tags
identifiants uniques
objets déjà collectés
niveaux déjà terminés
catégories
permissions
```

---

## 06.55 — Ajouter avec `add()`

```dart
void main() {
  Set<String> objets = {
    'Clé',
    'Potion',
  };

  objets.add('Bouclier');

  print(objets);
}
```

---

## 06.56 — Ajouter un doublon

```dart
objets.add('Clé');
```

Le `Set` ne crée pas un deuxième élément identique.

---

## 06.57 — `addAll()`

```dart
objets.addAll({
  'Épée',
  'Arc',
});
```

---

## 06.58 — `remove()`

```dart
objets.remove('Potion');
```

---

## 06.59 — `contains()`

```dart
if (objets.contains('Clé')) {
  print('Clé disponible');
}
```

---

## 06.60 — Parcourir un `Set`

```dart
void main() {
  Set<String> competences = {
    'Dart',
    'Flutter',
    'Git',
  };

  for (String competence in competences) {
    print(competence);
  }
}
```

---

## 06.61 — Union de deux ensembles

Considérons :

```dart
Set<String> equipeA = {
  'Dart',
  'Flutter',
};

Set<String> equipeB = {
  'Git',
  'Flutter',
};
```

L'union contient toutes les valeurs sans doublons.

```dart
Set<String> total = equipeA.union(equipeB);
```

Résultat conceptuel :

```text
Dart
Flutter
Git
```

---

## 06.62 — Intersection

L'intersection représente les éléments présents dans les deux ensembles.

```dart
Set<String> commun = equipeA.intersection(equipeB);
```

Résultat :

```text
Flutter
```

---

## 06.63 — Différence

```dart
Set<String> uniquementA =
    equipeA.difference(equipeB);
```

Résultat :

```text
Dart
```

---

## 06.64 — Résumé des opérations sur `Set`

```text
union
→ tous les éléments

intersection
→ éléments communs

difference
→ éléments du premier Set absents du second
```

---

## 06.65 — Troisième collection : `Map`

Une `Map` stocke des associations :

```text
clé → valeur
```

Exemple :

```dart
Map<String, int> scores = {
  'Alex': 1200,
  'Sophie': 1800,
  'Samir': 950,
};
```

On peut représenter :

```text
Alex    → 1200
Sophie  → 1800
Samir   → 950
```

---

## 06.66 — Comprendre le type

Dans :

```dart
Map<String, int>
```

le premier type :

```text
String
```

correspond au type des clés.

Le deuxième :

```text
int
```

correspond au type des valeurs.

---

## 06.67 — Première Map

```dart
void main() {
  Map<String, int> scores = {
    'Alex': 1200,
    'Sophie': 1800,
    'Samir': 950,
  };

  print(scores);
}
```

---

## 06.68 — Accéder à une valeur

```dart
print(scores['Alex']);
```

Résultat :

```text
1200
```

---

## 06.69 — Pourquoi la valeur peut être nullable ?

Lorsque nous écrivons :

```dart
scores['Alex']
```

Dart doit aussi prévoir que la clé puisse ne pas exister.

Par exemple :

```dart
scores['Maria']
```

pourrait retourner :

```text
null
```

Nous approfondirons ce comportement dans le chapitre consacré à la Null Safety.

---

## 06.70 — Ajouter une nouvelle entrée

```dart
scores['Maria'] = 1400;
```

La `Map` devient :

```text
Alex    → 1200
Sophie  → 1800
Samir   → 950
Maria   → 1400
```

---

## 06.71 — Modifier une valeur

```dart
scores['Alex'] = 2000;
```

La clé existe déjà.

Sa valeur est remplacée.

---

## 06.72 — Ajouter des points à une valeur

Pour l'instant, une façon simple consiste à écrire :

```dart
scores['Alex'] = 1500;
```

Plus tard, avec la Null Safety, nous apprendrons à manipuler plus élégamment :

```dart
scores['Alex']
```

lorsqu'une clé peut être absente.

---

## 06.73 — Supprimer une entrée

```dart
scores.remove('Samir');
```

---

## 06.74 — Vérifier une clé avec `containsKey()`

```dart
if (scores.containsKey('Alex')) {
  print('Alex existe.');
}
```

---

## 06.75 — Vérifier une valeur avec `containsValue()`

```dart
if (scores.containsValue(1800)) {
  print('Score trouvé.');
}
```

---

## 06.76 — Obtenir toutes les clés

```dart
print(scores.keys);
```

Résultat conceptuel :

```text
(Alex, Sophie, Samir)
```

---

## 06.77 — Obtenir toutes les valeurs

```dart
print(scores.values);
```

Résultat conceptuel :

```text
(1200, 1800, 950)
```

---

## 06.78 — Nombre d'entrées

```dart
print(scores.length);
```

---

## 06.79 — Parcourir les clés

```dart
void main() {
  Map<String, int> scores = {
    'Alex': 1200,
    'Sophie': 1800,
    'Samir': 950,
  };

  for (String joueur in scores.keys) {
    print(joueur);
  }
}
```

---

## 06.80 — Parcourir les valeurs

```dart
for (int score in scores.values) {
  print(score);
}
```

---

## 06.81 — Parcourir clé et valeur

Une manière simple :

```dart
void main() {
  Map<String, int> scores = {
    'Alex': 1200,
    'Sophie': 1800,
    'Samir': 950,
  };

  for (var entree in scores.entries) {
    print('${entree.key} : ${entree.value}');
  }
}
```

Résultat :

```text
Alex : 1200
Sophie : 1800
Samir : 950
```

---

## 06.82 — Comprendre `entries`

Une entrée représente un couple :

```text
clé + valeur
```

Exemple :

```text
Alex → 1200
```

Donc :

```dart
entree.key
```

donne :

```text
Alex
```

et :

```dart
entree.value
```

donne :

```text
1200
```

---

## 06.83 — Map contenant plusieurs informations

Une `Map` peut aussi contenir des informations différentes.

Exemple :

```dart
Map<String, dynamic> joueur = {
  'nom': 'Alex',
  'niveau': 5,
  'score': 1250,
  'energie': 80.5,
  'vivant': true,
};
```

Pourquoi :

```dart
dynamic
```

?

Parce que les valeurs n'ont pas toutes le même type.

---

## 06.84 — Lire les informations

```dart
void main() {
  Map<String, dynamic> joueur = {
    'nom': 'Alex',
    'niveau': 5,
    'score': 1250,
    'energie': 80.5,
    'vivant': true,
  };

  print(joueur['nom']);
  print(joueur['niveau']);
  print(joueur['score']);
}
```

---

## 06.85 — Pourquoi `Map<String, dynamic>` sera important ?

Cette structure ressemble énormément à des données JSON.

Par exemple :

```json
{
  "nom": "Alex",
  "niveau": 5,
  "score": 1250
}
```

En Dart, nous retrouverons souvent quelque chose de proche de :

```dart
Map<String, dynamic>
```

C'est pourquoi ce chapitre prépare directement :

```text
Flutter
API REST
JSON
Firebase
stockage de données
```

---

## 06.86 — Une liste de Maps

Nous pouvons créer :

```dart
List<Map<String, dynamic>> joueurs = [
  {
    'nom': 'Alex',
    'score': 1200,
  },
  {
    'nom': 'Sophie',
    'score': 1800,
  },
];
```

---

## 06.87 — Représentation

```text
joueurs
│
├── joueur 0
│   ├── nom   → Alex
│   └── score → 1200
│
└── joueur 1
    ├── nom   → Sophie
    └── score → 1800
```

---

## 06.88 — Accéder au premier joueur

```dart
print(joueurs[0]);
```

Résultat conceptuel :

```text
{nom: Alex, score: 1200}
```

---

## 06.89 — Accéder au nom du premier joueur

```dart
print(joueurs[0]['nom']);
```

Résultat :

```text
Alex
```

---

## 06.90 — Accéder au score du deuxième joueur

```dart
print(joueurs[1]['score']);
```

Résultat :

```text
1800
```

---

## 06.91 — Parcourir une liste de Maps

```dart
void main() {
  List<Map<String, dynamic>> joueurs = [
    {
      'nom': 'Alex',
      'score': 1200,
    },
    {
      'nom': 'Sophie',
      'score': 1800,
    },
    {
      'nom': 'Samir',
      'score': 950,
    },
  ];

  for (Map<String, dynamic> joueur in joueurs) {
    print(
      '${joueur['nom']} : ${joueur['score']} points',
    );
  }
}
```

---

## 06.92 — Exemple produit

```dart
List<Map<String, dynamic>> produits = [
  {
    'nom': 'Clavier',
    'prix': 79.99,
    'stock': 5,
  },
  {
    'nom': 'Souris',
    'prix': 39.99,
    'stock': 10,
  },
];
```

Ce type de données sera très fréquent dans Flutter.

---

## 06.93 — Exemple d'inventaire plus structuré

```dart
List<Map<String, dynamic>> inventaire = [
  {
    'nom': 'Potion',
    'quantite': 3,
    'valeur': 50,
  },
  {
    'nom': 'Clé',
    'quantite': 1,
    'valeur': 0,
  },
];
```

---

## 06.94 — Map contenant une List

Une `Map` peut également contenir une liste.

```dart
Map<String, dynamic> joueur = {
  'nom': 'Alex',
  'score': 1500,
  'inventaire': [
    'Épée',
    'Potion',
    'Clé',
  ],
};
```

---

## 06.95 — Structure conceptuelle

```text
joueur
│
├── nom → Alex
├── score → 1500
│
└── inventaire
    ├── Épée
    ├── Potion
    └── Clé
```

---

## 06.96 — Lire l'inventaire

```dart
print(joueur['inventaire']);
```

---

## 06.97 — Pourquoi les collections imbriquées sont importantes ?

Une application réelle manipule rarement uniquement :

```text
une chaîne
un entier
un booléen
```

Elle manipule plutôt :

```text
liste de produits
liste d'utilisateurs
liste de messages
liste de joueurs
liste d'ennemis
liste de tâches
liste de niveaux
liste de réponses API
```

Chaque élément peut lui-même contenir plusieurs propriétés.

---

## 06.98 — `final` et les collections

Considérons :

```dart
final List<String> joueurs = [
  'Alex',
  'Sophie',
];
```

Une subtilité importante existe.

`final` empêche de réassigner :

```dart
joueurs = ['Maria'];
```

Mais la liste elle-même reste modifiable.

Ceci reste possible :

```dart
joueurs.add('Samir');
```

---

## 06.99 — `const` et collections immuables

Le chapitre 02 a présenté `const` comme une variante plus stricte de `final`. Avec les collections, cette différence devient très concrète.

Rappel de la section précédente : `final` empêche de réassigner la variable, mais le contenu de la liste reste modifiable.

`const` va plus loin. Une collection `const` est figée dès la compilation : ni réassignation, ni modification du contenu ne sont possibles.

```dart
const List<String> armesDeDepart = [
  'Épée',
  'Bouclier',
];
```

Tenter d'ajouter un élément :

```dart
armesDeDepart.add('Arc');
```

provoque une erreur à l'exécution, car la liste est totalement figée.

Le même principe s'applique à `Set` et à `Map` :

```dart
const Set<String> niveauxDeDepart = {
  'Niveau 1',
};
```

```dart
const Map<String, int> scoresDeDepart = {
  'Alex': 0,
  'Sophie': 0,
};
```

Dans les deux cas, `add()`, `remove()` ou toute écriture sur une clé provoquent une erreur à l'exécution.

Voici un tableau pour résumer les différences entre `var`, `final` et `const` appliqués à une collection :

| Mot-clé | Réassigner la variable | Modifier le contenu (`add`, `remove`, `[]=`) |
| --- | --- | --- |
| `var` | possible | possible |
| `final` | impossible | possible |
| `const` | impossible | impossible |

Dans notre futur jeu, `const` conviendra à des valeurs qui ne changeront jamais pendant l'exécution, par exemple :

```dart
const List<String> difficultes = [
  'Facile',
  'Normal',
  'Difficile',
];
```

À l'inverse, l'inventaire d'un joueur, qui évolue sans cesse, doit rester une collection `final` (ou `var`), jamais `const`.

---

## 06.100 — Quelle collection choisir ?

Après avoir vu `List`, `Set` et `Map`, une question revient souvent : laquelle utiliser dans telle ou telle situation ?

Le tableau suivant sert de guide de décision.

| Besoin | Collection recommandée | Pourquoi |
| --- | --- | --- |
| Garder l'ordre exact d'ajout des éléments | `List` | Une `List` conserve toujours l'ordre d'insertion |
| Autoriser des valeurs identiques (plusieurs ennemis « Zombie ») | `List` | Une `List` accepte les doublons |
| Accéder à un élément par sa position (le premier, le troisième...) | `List` | Une `List` est indexée à partir de `0` |
| Garantir des valeurs uniques (niveaux déjà terminés) | `Set` | Un `Set` refuse automatiquement les doublons |
| Vérifier rapidement si une valeur est présente, sans se soucier de l'ordre | `Set` | La recherche d'appartenance est la raison d'être du `Set` |
| Calculer des unions, intersections ou différences entre deux ensembles | `Set` | `union()`, `intersection()` et `difference()` n'existent que sur `Set` |
| Associer une information à une autre (nom → score, clé → objet) | `Map` | Une `Map` fonctionne par paires clé/valeur |
| Retrouver une valeur à partir d'un identifiant plutôt qu'une position | `Map` | La recherche par clé est directe avec `map[cle]` |
| Représenter un objet possédant plusieurs propriétés (joueur, produit) | `Map<String, dynamic>` | Chaque propriété devient une clé de la `Map` |
| Représenter une collection d'objets provenant d'une API JSON | `List<Map<String, dynamic>>` | C'est exactement la structure que renvoie la majorité des API |

En cas de doute, la question à se poser est simple :

```text
Ai-je besoin de doublons et d'un ordre ? → List
Ai-je besoin d'unicité ? → Set
Ai-je besoin d'associer une clé à une valeur ? → Map
```

---

## 06.101 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| Accéder à un index qui n'existe pas | L'index demandé est égal ou supérieur à `length`, ou négatif | Toujours vérifier `index < liste.length` avant d'accéder à l'élément, ou utiliser `first` / `last` |
| Modifier une liste pendant qu'on la parcourt avec `for-in` | Ajouter ou retirer un élément change la structure que la boucle est en train de parcourir | Parcourir une copie avec `toList()`, ou construire une nouvelle liste plutôt que de modifier l'originale pendant la boucle |
| Confondre `remove()` et `removeAt()` | `remove()` attend une valeur, `removeAt()` attend une position | Se demander : « je connais la valeur » (`remove`) ou « je connais la position » (`removeAt`) |
| Lire une clé absente d'une `Map` sans vérifier | `map[cle]` renvoie `null` si la clé n'existe pas, au lieu de provoquer une erreur immédiate | Vérifier avec `containsKey()` avant de lire, ou utiliser `??` pour fournir une valeur par défaut |
| Oublier le type générique d'une collection | Une collection sans type explicite accepte n'importe quel type, les erreurs ne sont détectées qu'à l'exécution | Toujours préciser `List<Type>`, `Set<Type>` ou `Map<KeyType, ValueType>` |
| Comparer deux listes avec `==` | `==` compare l'identité des objets, pas leur contenu ; deux listes distinctes avec les mêmes valeurs ne sont pas égales | Comparer élément par élément si le contenu doit être comparé |
| Essayer de modifier une collection `const` | Une collection `const` est figée définitivement dès la compilation | Utiliser `final` si le contenu doit encore évoluer, réserver `const` aux valeurs vraiment fixes |
| Créer un `Set` vide avec `{}` sans préciser le type | `{}` seul est interprété par défaut comme une `Map` vide, pas comme un `Set` | Écrire `<String>{}` (ou le type voulu) pour créer explicitement un `Set` vide |
| Confondre `length` et le dernier index valide | Le dernier index valide est toujours `length - 1`, jamais `length` | Vérifier ce détail avant d'écrire une condition avec `<` ou `<=` dans une boucle |
| Utiliser `indexOf()` sans vérifier le résultat | `indexOf()` renvoie `-1`, et non une erreur, si la valeur est absente | Toujours tester `if (index != -1)` avant d'utiliser la position renvoyée |
| Utiliser `+` pour combiner deux `Set` | L'opérateur `+` n'existe pas pour les `Set`, seulement pour les `List` | Utiliser la méthode `union()` pour combiner deux `Set` |

---

## 06.102 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| Collection | Structure permettant de stocker plusieurs valeurs dans une seule variable |
| `List` | Collection ordonnée, indexée à partir de `0`, doublons autorisés |
| Index | Position d'un élément dans une `List`, de `0` à `length - 1` |
| `length` | Nombre d'éléments d'une collection |
| `first` / `last` | Accès direct au premier et au dernier élément d'une `List` |
| `add()` / `addAll()` / `insert()` | Ajoutent un élément, plusieurs éléments, ou un élément à une position précise |
| `remove()` / `removeAt()` / `removeLast()` / `clear()` | Retirent un élément par sa valeur, par sa position, le dernier élément, ou vident la collection |
| `contains()` / `indexOf()` | Vérifient la présence d'une valeur, ou renvoient sa position (`-1` si absente) |
| `for` et `for-in` | Deux façons de parcourir une `List` ; `for-in` quand l'index n'est pas nécessaire |
| `Set` | Collection non ordonnée garantissant l'unicité des valeurs |
| `union()` / `intersection()` / `difference()` | Opérations ensemblistes disponibles uniquement sur `Set` |
| `Map` | Collection clé/valeur, accès direct à une valeur via sa clé |
| `containsKey()` / `containsValue()` | Vérifient la présence d'une clé ou d'une valeur dans une `Map` |
| `keys` / `values` / `entries` | Accès à toutes les clés, toutes les valeurs, ou aux paires clé/valeur d'une `Map` |
| Collections imbriquées | Une `List` ou une `Map` peut contenir d'autres collections (grille, `List<Map<String, dynamic>>`) |
| `Map<String, dynamic>` | Structure très fréquente pour représenter un objet avec plusieurs propriétés, proche du JSON |
| `final` sur une collection | Empêche la réassignation de la variable, mais pas la modification du contenu |
| `const` sur une collection | Empêche à la fois la réassignation et la modification du contenu |
| Choisir sa collection | `List` (ordre et doublons), `Set` (unicité), `Map` (association clé/valeur) |

---

## 06.103 — Exercices

### Exercice 1 — Liste d'armes (facile)

Créez une `List<String>` nommée `armes` contenant quatre armes du jeu : `'Épée'`, `'Arc'`, `'Bâton'`, `'Hache'`.

Affichez la liste entière avec `print()`.

---

### Exercice 2 — Première et dernière arme (facile)

À partir de la liste `armes` de l'exercice précédent, affichez uniquement la première et la dernière arme, en utilisant `first` et `last`.

---

### Exercice 3 — Total des scores (facile)

Créez une `List<int>` nommée `scores` contenant cinq scores : `120`, `340`, `75`, `500`, `210`.

Calculez et affichez leur somme totale à l'aide d'une boucle `for-in` et d'une variable accumulatrice.

---

### Exercice 4 — Ajouter et retirer une arme (facile)

À partir de la liste `armes`, ajoutez `'Dague'` à la fin avec `add()`, puis retirez `'Bâton'` avec `remove()`.

Affichez la liste finale.

---

### Exercice 5 — Ennemis vaincus sans doublons (moyen)

Créez un `Set<String>` nommé `ennemisVaincus`.

Ajoutez trois fois de suite la valeur `'Zombie'` avec `add()`.

Affichez le `Set`, puis affichez sa taille avec `length`. Expliquez, en commentaire, pourquoi la taille vaut `1`.

---

### Exercice 6 — Niveaux communs à deux joueurs (moyen)

Créez deux `Set<String>` :

```text
niveauxJoueur1 : Niveau 1, Niveau 2, Niveau 3
niveauxJoueur2 : Niveau 2, Niveau 3, Niveau 4
```

Affichez :

- l'intersection des deux `Set` (les niveaux terminés par les deux joueurs) ;
- l'union des deux `Set` (tous les niveaux terminés par au moins un des deux joueurs).

---

### Exercice 7 — Scores des joueurs dans une Map (moyen)

Créez une `Map<String, int>` nommée `scores` avec trois joueurs :

```text
'Alex'   : 1200
'Sophie' : 1800
'Samir'  : 950
```

Ajoutez un nouveau joueur `'Maria'` avec un score de `1400`.

Modifiez le score de `'Samir'` pour qu'il devienne `1100`.

Parcourez ensuite la `Map` avec `entries` et affichez chaque ligne au format :

```text
Nom : Score
```

---

### Exercice 8 — Compter les obstacles d'une grille (difficile)

Créez une `List<List<int>>` représentant une grille 3×3, où `0` signifie une case vide et `1` un obstacle :

```text
0 1 0
1 1 0
0 0 1
```

À l'aide de deux boucles `for` imbriquées, comptez et affichez le nombre total d'obstacles.

---

### Exercice 9 — Filtrer les ennemis dangereux (difficile)

Créez une `List<Map<String, dynamic>>` nommée `ennemis`, où chaque élément possède les clés `'nom'` et `'pointsDeVie'` :

```text
Zombie  : 40
Dragon  : 200
Gobelin : 25
Golem   : 150
```

Parcourez la liste et affichez uniquement le nom des ennemis dont les points de vie dépassent `50`.

---

### Exercice 10 — Mini-projet : inventaire de jeu (difficile)

Ce dernier exercice rassemble tout ce qui a été vu dans ce chapitre.

Créez une `Map<String, int>` nommée `inventaire`, où chaque clé est le nom d'un objet et chaque valeur sa quantité. Initialisez-la avec :

```text
'Potion' : 3
'Casque' : 2
'Clé'    : 1
```

Votre programme doit ensuite, dans cet ordre :

1. **Ajout** : le joueur trouve 2 potions supplémentaires, et une nouvelle épée (`'Epée'`, quantité `1`, nouvelle clé de la `Map`).
2. **Retrait** : le joueur utilise sa clé ; sa quantité diminue de 1, et l'entrée `'Clé'` est totalement supprimée de la `Map` si sa quantité tombe à `0`.
3. **Recherche** : le joueur cherche s'il possède un `'Bouclier'` dans son inventaire, et un message adapté est affiché selon le résultat.
4. **Affichage trié** : l'inventaire final est affiché par ordre alphabétique du nom de l'objet, au format :

```text
Nom : Quantité
```

Indice : `inventaire.keys.toList()` permet de récupérer les clés sous forme de `List`, que l'on peut ensuite trier avec `sort()`.

---

## 06.104 — Corrections des exercices

### Correction 1

```dart
void main() {
  List<String> armes = [
    'Épée',
    'Arc',
    'Bâton',
    'Hache',
  ];

  print(armes);
}
```

**Résultat :**

```text
[Épée, Arc, Bâton, Hache]
```

**Explication :** `print()` appliqué directement à une `List` affiche automatiquement tous ses éléments entre crochets, séparés par des virgules.

---

### Correction 2

```dart
void main() {
  List<String> armes = [
    'Épée',
    'Arc',
    'Bâton',
    'Hache',
  ];

  print(armes.first);
  print(armes.last);
}
```

**Résultat :**

```text
Épée
Hache
```

**Explication :** `first` et `last` évitent d'écrire `armes[0]` et `armes[armes.length - 1]`.

---

### Correction 3

```dart
void main() {
  List<int> scores = [120, 340, 75, 500, 210];

  int total = 0;

  for (int score in scores) {
    total += score;
  }

  print('Total : $total');
}
```

**Résultat :**

```text
Total : 1245
```

**Explication :** `for-in` parcourt directement chaque valeur de la liste, sans avoir besoin d'un index, ce qui convient parfaitement ici puisque la position n'est pas utile.

---

### Correction 4

```dart
void main() {
  List<String> armes = [
    'Épée',
    'Arc',
    'Bâton',
    'Hache',
  ];

  armes.add('Dague');
  armes.remove('Bâton');

  print(armes);
}
```

**Résultat :**

```text
[Épée, Arc, Hache, Dague]
```

**Explication :** `add()` insère toujours à la fin de la liste, tandis que `remove()` retire la première occurrence de la valeur donnée, quelle que soit sa position.

---

### Correction 5

```dart
void main() {
  Set<String> ennemisVaincus = {};

  ennemisVaincus.add('Zombie');
  ennemisVaincus.add('Zombie');
  ennemisVaincus.add('Zombie');

  print(ennemisVaincus);
  print(ennemisVaincus.length);
}
```

**Résultat :**

```text
{Zombie}
1
```

**Explication :** un `Set` refuse silencieusement les doublons ; ajouter trois fois `'Zombie'` ne produit qu'une seule entrée, d'où une taille de `1`.

---

### Correction 6

```dart
void main() {
  Set<String> niveauxJoueur1 = {'Niveau 1', 'Niveau 2', 'Niveau 3'};
  Set<String> niveauxJoueur2 = {'Niveau 2', 'Niveau 3', 'Niveau 4'};

  Set<String> communs = niveauxJoueur1.intersection(niveauxJoueur2);
  Set<String> tous = niveauxJoueur1.union(niveauxJoueur2);

  print(communs);
  print(tous);
}
```

**Résultat :**

```text
{Niveau 2, Niveau 3}
{Niveau 1, Niveau 2, Niveau 3, Niveau 4}
```

**Explication :** `intersection()` ne garde que les valeurs présentes dans les deux `Set`, tandis que `union()` rassemble toutes les valeurs des deux `Set`, sans jamais créer de doublon.

---

### Correction 7

```dart
void main() {
  Map<String, int> scores = {
    'Alex': 1200,
    'Sophie': 1800,
    'Samir': 950,
  };

  scores['Maria'] = 1400;
  scores['Samir'] = 1100;

  for (MapEntry<String, int> entry in scores.entries) {
    print('${entry.key} : ${entry.value}');
  }
}
```

**Résultat :**

```text
Alex : 1200
Sophie : 1800
Samir : 1100
Maria : 1400
```

**Explication :** écrire `scores['Maria'] = 1400` ajoute une nouvelle entrée si la clé n'existe pas encore, ou modifie la valeur existante sinon ; `entries` donne accès à chaque couple clé/valeur pendant le parcours.

---

### Correction 8

```dart
void main() {
  List<List<int>> grille = [
    [0, 1, 0],
    [1, 1, 0],
    [0, 0, 1],
  ];

  int totalObstacles = 0;

  for (int ligne = 0; ligne < grille.length; ligne++) {
    for (int colonne = 0; colonne < grille[ligne].length; colonne++) {
      if (grille[ligne][colonne] == 1) {
        totalObstacles++;
      }
    }
  }

  print('Obstacles : $totalObstacles');
}
```

**Résultat :**

```text
Obstacles : 4
```

**Explication :** la boucle externe parcourt les lignes, la boucle interne parcourt les colonnes de chaque ligne ; `grille[ligne][colonne]` donne accès à une case précise de la grille.

---

### Correction 9

```dart
void main() {
  List<Map<String, dynamic>> ennemis = [
    {'nom': 'Zombie', 'pointsDeVie': 40},
    {'nom': 'Dragon', 'pointsDeVie': 200},
    {'nom': 'Gobelin', 'pointsDeVie': 25},
    {'nom': 'Golem', 'pointsDeVie': 150},
  ];

  for (Map<String, dynamic> ennemi in ennemis) {
    if (ennemi['pointsDeVie'] > 50) {
      print(ennemi['nom']);
    }
  }
}
```

**Résultat :**

```text
Dragon
Golem
```

**Explication :** chaque élément de la liste est une `Map` indépendante ; on lit sa clé `'pointsDeVie'` pour filtrer manuellement avec un `if` à l'intérieur du `for-in`.

---

### Correction 10

```dart
void main() {
  Map<String, int> inventaire = {
    'Potion': 3,
    'Casque': 2,
    'Clé': 1,
  };

  // 1. Ajout
  inventaire['Potion'] = (inventaire['Potion'] ?? 0) + 2;
  inventaire['Epée'] = (inventaire['Epée'] ?? 0) + 1;

  // 2. Retrait
  inventaire['Clé'] = inventaire['Clé']! - 1;

  if (inventaire['Clé'] == 0) {
    inventaire.remove('Clé');
  }

  // 3. Recherche
  if (inventaire.containsKey('Bouclier')) {
    print('Le joueur possède un Bouclier.');
  } else {
    print('Le joueur ne possède pas de Bouclier.');
  }

  // 4. Affichage trié
  List<String> objets = inventaire.keys.toList();
  objets.sort();

  for (String objet in objets) {
    print('$objet : ${inventaire[objet]}');
  }
}
```

**Résultat :**

```text
Le joueur ne possède pas de Bouclier.
Casque : 2
Epée : 1
Potion : 5
```

**Explication :** l'ajout utilise `?? 0` pour gérer aussi bien une clé déjà existante (`'Potion'`) qu'une clé encore absente (`'Epée'`) ; le retrait diminue la quantité puis supprime totalement l'entrée si elle atteint `0` ; `containsKey()` permet une recherche sûre sans jamais provoquer d'erreur ; enfin, `keys.toList()` suivi de `sort()` transforme les clés de la `Map` en une `List` triée, utilisée ensuite pour afficher l'inventaire dans l'ordre alphabétique.

---

## Et maintenant ?

Vous savez désormais stocker, parcourir et manipuler plusieurs valeurs avec `List`, `Set` et `Map`, y compris des structures imbriquées proches de ce que renverra une véritable API. Le chapitre suivant vous apprend à regrouper du code répétitif dans des fonctions réutilisables, que vous appellerez notamment pour agir sur les collections construites ici : attaquer un ennemi, calculer des dégâts, ou ajouter un score à l'inventaire du joueur.

Direction le chapitre suivant : [07-PARTIE-1A—LES-FONCTIONS.md](07-PARTIE-1A—LES-FONCTIONS.md)
