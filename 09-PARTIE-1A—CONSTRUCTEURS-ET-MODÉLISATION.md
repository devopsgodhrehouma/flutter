# PARTIE 1A — DART
# CHAPITRE 09 — CONSTRUCTEURS ET MODÉLISATION

> **Niveau :** débutant / intermédiaire
> **Durée estimée :** 7 h
> **Pré-requis :** chapitre 08 — Introduction à la POO (classe, objet, propriété, méthode, `this`)
> **Ce que vous saurez faire à la fin :** créer des objets complets et valides dès leur première ligne de vie grâce aux constructeurs, exposer des valeurs calculées avec des getters, et traduire un énoncé écrit en français en un ensemble de classes cohérentes.

---

## 09.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer précisément pourquoi une classe sans constructeur pose problème ;
- reconnaître et utiliser le constructeur par défaut ;
- écrire un constructeur personnalisé ;
- utiliser la syntaxe raccourcie `this.propriete` ;
- utiliser des paramètres nommés avec les accolades `{ }` ;
- rendre un paramètre nommé obligatoire avec `required` ;
- déclarer des paramètres optionnels et leur donner une valeur par défaut ;
- écrire des constructeurs nommés comme `Player.debutant()` ou `Enemy.boss()` ;
- initialiser une propriété avec une initializer list `: x = ...` ;
- protéger vos objets avec `assert` ;
- écrire un constructeur `const` et expliquer ce qu'il apporte ;
- écrire un constructeur `factory` et expliquer quand il est utile ;
- écrire un getter, un setter, et des getters calculés ;
- redéfinir `toString()` pour afficher un objet lisible ;
- transformer un énoncé écrit en français en un ensemble de classes Dart ;
- appliquer une liste de bonnes pratiques de modélisation.

---

## 09.0.1 — Avertissement sur la progression

Ce chapitre porte sur **la naissance d'un objet** et sur **la façon de le modéliser**.

Nous n'utiliserons volontairement PAS encore :

```text
- l'héritage (extends, super)           -> chapitre 10
- le polymorphisme                      -> chapitre 10
- les classes abstraites, mixins, enums -> chapitre 11
- le null safety avancé (?, !, late)    -> chapitre 12
- les exceptions (throw, try/catch)     -> chapitre 13
```

Toutes les classes de ce chapitre sont donc **indépendantes** : aucune ne prolonge une autre.

Le fil rouge reste le **jeu vidéo** (`Player`, `Enemy`, `Weapon`, `Potion`). Une classe `Product` de commerce en ligne montrera que la POO n'est pas réservée aux jeux.

---

## 09.1 — Le problème sans constructeur (rappel du chapitre 08)

Au chapitre 08, nous écrivions une classe puis nous remplissions ses propriétés une par une.

```dart
class Player {
  String name = 'Inconnu';
  int health = 100;
  int score = 0;

  void showStatus() {
    print('$name | vie: $health | score: $score');
  }
}

void main() {
  Player joueur = Player();
  joueur.name = 'Alex';
  joueur.health = 80;
  joueur.score = 250;

  joueur.showStatus();
}
```

**Résultat :**

```text
Alex | vie: 80 | score: 250
```

Observez la vie de l'objet :

```text
ÉTAPE 1  Player()            -> l'objet existe, mais il s'appelle "Inconnu"
ÉTAPE 2  joueur.name = ...   -> on corrige
ÉTAPE 3  joueur.health = ... -> on corrige
ÉTAPE 4  joueur.score = ...  -> on corrige

Entre l'étape 1 et l'étape 4, l'objet est INCOMPLET.
```

Cet intervalle produit trois défauts.

**Défaut 1 — l'objet peut vivre à moitié construit.** Si vous oubliez `joueur.score = 250;`, le programme ne dit rien et le score vaut silencieusement `0`.

**Défaut 2 — c'est verbeux.** Trois joueurs demandent douze lignes :

```dart
class Player {
  String name = 'Inconnu';
  int health = 100;
  int score = 0;
}

void main() {
  Player j1 = Player();
  j1.name = 'Alex';
  j1.health = 100;

  Player j2 = Player();
  j2.name = 'Sophie';
  j2.health = 100;

  print('${j1.name} et ${j2.name}');
}
```

**Résultat :**

```text
Alex et Sophie
```

**Défaut 3 — rien n'est vérifié.**

```dart
class Player {
  String name = 'Inconnu';
  int health = 100;
}

void main() {
  Player joueur = Player();
  joueur.name = '';
  joueur.health = -9999;

  print('"${joueur.name}" a ${joueur.health} points de vie.');
}
```

**Résultat :**

```text
"" a -9999 points de vie.
```

Un joueur sans nom, avec une vie négative, et pas le moindre avertissement.

Il nous faut un mécanisme qui garantisse :

```text
"Un objet Player n'a JAMAIS le droit d'exister
 s'il n'a pas un nom et une vie valides."
```

Ce mécanisme s'appelle le **constructeur**.

> Un constructeur est une fonction spéciale, appelée automatiquement à la création de l'objet, dont le seul rôle est de mettre cet objet dans un état valide dès sa première milliseconde d'existence.

---

## 09.2 — Le constructeur par défaut

Vous avez déjà utilisé un constructeur sans le savoir : les parenthèses de `Player()` sont un **appel de constructeur**.

Lorsque vous n'écrivez aucun constructeur, Dart en fabrique un pour vous, invisible, sans paramètre, qui ne fait rien. C'est le **constructeur par défaut**. Tout se passe comme si vous aviez écrit :

```dart
class Player {
  String name = 'Inconnu';

  Player() {
    // ce constructeur existe implicitement et ne fait rien
  }
}

void main() {
  Player joueur = Player();
  print(joueur.name);
}
```

**Résultat :**

```text
Inconnu
```

Décomposons sa forme, déroutante au premier abord :

```text
class Player {
        Player()  {  }
          │   │
          │   └── liste des paramètres (ici : aucun)
          └────── le nom du constructeur est EXACTEMENT
                  le nom de la classe

Un constructeur :
  - porte le nom de la classe
  - n'a AUCUN type de retour (ni void, ni Player)
```

> **Règle absolue :** un constructeur ne déclare jamais de type de retour. `void Player() {}` crée une méthode nommée `Player`, pas un constructeur.

### 09.2.1 — Le constructeur par défaut disparaît dès que vous en écrivez un

```dart
class Player {
  String name = 'Inconnu';

  Player(String nomRecu) {
    name = nomRecu;
  }
}

void main() {
  Player joueur = Player('Alex');
  print(joueur.name);

  // Player autre = Player();  // ERREUR DE COMPILATION
}
```

**Résultat :**

```text
Alex
```

Dès que vous écrivez **un** constructeur, Dart cesse d'en fournir un gratuitement :

```text
Error: Too few positional arguments: 1 required, 0 given.
```

---

## 09.3 — Le constructeur personnalisé

Un **constructeur personnalisé** reçoit des valeurs et remplit les propriétés.

```dart
class Player {
  String name = '';
  int health = 0;
  int score = 0;

  Player(String nomRecu, int vieRecue, int scoreRecu) {
    name = nomRecu;
    health = vieRecue;
    score = scoreRecu;
  }

  void showStatus() {
    print('$name | vie: $health | score: $score');
  }
}

void main() {
  Player joueur = Player('Alex', 80, 250);
  joueur.showStatus();
}
```

**Résultat :**

```text
Alex | vie: 80 | score: 250
```

Ce que fait Dart, dans l'ordre :

```text
1. Player('Alex', 80, 250) est rencontré
2. Dart réserve une place en mémoire pour un nouvel objet Player
3. Dart place les valeurs initiales des propriétés ('' , 0, 0)
4. Dart exécute le CORPS du constructeur : name='Alex', health=80, score=250
5. Dart renvoie l'objet, prêt à l'emploi
```

### 09.3.1 — Les paramètres positionnels

Dans `Player('Alex', 80, 250)`, c'est la **position** qui détermine le rôle.

```text
Player( 'Alex' ,  80  ,  250 )
           │       │      │
           │       │      └── 3e paramètre -> scoreRecu
           │       └───────── 2e paramètre -> vieRecue
           └───────────────── 1er paramètre -> nomRecu
```

Inversez deux nombres, et `Player('Alex', 250, 80)` compile sans broncher en donnant 250 points de vie au joueur. Nous corrigerons ce défaut en section 09.5.

### 09.3.2 — Le mot-clé `this` pour lever une ambiguïté

Il est naturel de donner au paramètre le même nom que la propriété. Il faut alors `this`.

```dart
class Player {
  String name = '';
  int health = 0;

  Player(String name, int health) {
    this.name = name;
    this.health = health;
  }
}

void main() {
  Player joueur = Player('Alex', 100);
  print('${joueur.name} - ${joueur.health}');
}
```

**Résultat :**

```text
Alex - 100
```

```text
Player(String name, int health) {
   this.name = name;
     │    │     │
     │    │     └── le PARAMÈTRE du constructeur
     │    └──────── la PROPRIÉTÉ de l'objet
     └───────────── "this" = l'objet en cours de construction
```

Sans `this`, la ligne `name = name;` affecterait le paramètre à lui-même, et la propriété resterait vide :

```dart
class Player {
  String name = '';

  Player(String name) {
    name = name; // erreur : on écrase le paramètre avec lui-même
  }
}

void main() {
  print('Nom : "${Player('Alex').name}"');
}
```

**Résultat :**

```text
Nom : ""
```

> **Retenez :** dans le corps d'un constructeur, si un paramètre porte le même nom qu'une propriété, `this.` est obligatoire pour désigner la propriété.

---

## 09.4 — La syntaxe raccourcie `this.propriete`

Écrire `this.name = name;` pour chaque propriété est fastidieux. Dart propose un raccourci officiel, utilisé partout en Dart et en Flutter.

```dart
class Player {
  String name;
  int health;
  int score;

  Player(this.name, this.health, this.score);

  void showStatus() {
    print('$name | vie: $health | score: $score');
  }
}

void main() {
  Player joueur = Player('Alex', 80, 250);
  joueur.showStatus();
}
```

**Résultat :**

```text
Alex | vie: 80 | score: 250
```

Les deux écritures sont strictement équivalentes :

```text
VERSION LONGUE                      VERSION RACCOURCIE

Player(String n, int h, int s) {    Player(this.name, this.health, this.score);
  name = n;
  health = h;                       (le corps est vide : un simple
  score = s;                         point-virgule remplace les accolades)
}
```

Points à remarquer :

- `this.name` en paramètre signifie « prends la valeur reçue et range-la dans la propriété `name` » ;
- le type du paramètre est **déduit** de celui de la propriété : on ne le réécrit pas ;
- le corps étant vide, on termine par un **point-virgule**.

### 09.4.1 — Les propriétés n'ont plus besoin de valeur initiale

```dart
// AVANT (chapitre 08) : obligatoire, sinon Dart refuse
String name = 'Inconnu';

// MAINTENANT : possible, car le constructeur garantit une valeur
String name;
```

Si vous supprimez le constructeur, Dart proteste :

```text
Error: Field 'name' should be initialized because its type 'String'
doesn't allow null.
```

### 09.4.2 — Les propriétés peuvent devenir `final`

Une propriété `final` est fixée une fois pour toutes à la construction.

```dart
class Weapon {
  final String name;
  final int damage;

  Weapon(this.name, this.damage);

  void describe() {
    print('$name inflige $damage degats.');
  }
}

void main() {
  Weapon epee = Weapon('Épée de fer', 12);
  epee.describe();

  // epee.damage = 999; // ERREUR : une propriété final ne se modifie pas
}
```

**Résultat :**

```text
Épée de fer inflige 12 degats.
```

Message de Dart si vous décommentez : `The setter 'damage' isn't defined for the class 'Weapon'.`

> **Bonne habitude :** tout ce qui ne doit jamais changer après la création doit être `final`. C'est le compilateur qui protège votre code à votre place.

---

## 09.5 — Les paramètres nommés `{ }` dans un constructeur

Le défaut des paramètres positionnels est que l'ordre est invisible à la lecture : dans `Player('Alex', 80, 250)`, impossible de savoir ce que valent `80` et `250` sans ouvrir la classe.

On entoure alors les paramètres d'**accolades** `{ }`.

```dart
class Player {
  String name;
  int health;
  int score;

  Player({this.name = 'Inconnu', this.health = 100, this.score = 0});

  void showStatus() {
    print('$name | vie: $health | score: $score');
  }
}

void main() {
  Player a = Player(name: 'Alex', health: 80, score: 250);
  Player b = Player(score: 250, health: 80, name: 'Alex'); // ordre libre

  a.showStatus();
  b.showStatus();
}
```

**Résultat :**

```text
Alex | vie: 80 | score: 250
Alex | vie: 80 | score: 250
```

À l'appel, chaque valeur est étiquetée, et l'ordre n'a plus d'importance :

```text
Player( name: 'Alex' , health: 80 , score: 250 )
          │              │             │
          └──────────────┴─────────────┴── chaque valeur est ÉTIQUETÉE
```

### 09.5.1 — Pourquoi c'est le style dominant en Dart et Flutter

```text
POSITIONNEL                       NOMMÉ

Enemy('Gobelin', 30, 5, true)     Enemy(
                                    name: 'Gobelin',
Il faut deviner :                   health: 30,
  30   = vie ? dégâts ?             damage: 5,
  5    = dégâts ? niveau ?          isBoss: true,
  true = boss ? vivant ?          )
```

Presque tous les widgets Flutter utilisent des paramètres nommés. Prenez l'habitude dès maintenant.

> **Règle pratique :** au-delà de deux paramètres, préférez les paramètres nommés.

### 09.5.2 — Un paramètre nommé est optionnel par défaut

C'est le piège de la section :

```dart
class Player {
  String name;
  int health;

  Player({this.name = 'Inconnu', this.health = 100});
}

void main() {
  Player fantome = Player(); // aucun argument, et pourtant cela compile
  print('${fantome.name} | vie: ${fantome.health}');
}
```

**Résultat :**

```text
Inconnu | vie: 100
```

Nous retombons sur le problème de la section 09.1. La solution s'appelle `required`.

---

## 09.6 — `required` : rendre un paramètre nommé obligatoire

Le mot-clé `required` force l'appelant à fournir la valeur.

```dart
class Player {
  String name;
  int health;
  int score;

  Player({
    required this.name,
    required this.health,
    this.score = 0,
  });

  void showStatus() {
    print('$name | vie: $health | score: $score');
  }
}

void main() {
  Player joueur = Player(name: 'Alex', health: 100);
  joueur.showStatus();

  // Player x = Player();             // ERREUR
  // Player y = Player(name: 'Bob');  // ERREUR
}
```

**Résultat :**

```text
Alex | vie: 100 | score: 0
```

Messages produits par les lignes commentées :

```text
Error: Required named parameter 'name' must be provided.
Error: Required named parameter 'health' must be provided.
```

L'erreur survient **à la compilation**, avant même l'exécution : un objet incomplet ne peut plus exister.

### 09.6.1 — `required` et valeur par défaut sont incompatibles

| Écriture | Signification |
| --- | --- |
| `required this.name` | l'appelant DOIT fournir `name` |
| `this.name = 'Inconnu'` | l'appelant PEUT fournir `name` |
| `required this.name = 'X'` | erreur de compilation |

C'est logique : si une valeur par défaut existe, le paramètre n'a aucune raison d'être obligatoire.

### 09.6.2 — Application à `Product` (commerce en ligne)

```dart
class Product {
  final String reference;
  final String label;
  final double price;
  int stock;

  Product({
    required this.reference,
    required this.label,
    required this.price,
    this.stock = 0,
  });

  void describe() {
    print('[$reference] $label - ${price.toStringAsFixed(2)} EUR - stock: $stock');
  }
}

void main() {
  Product clavier = Product(
    reference: 'KB-100',
    label: 'Clavier mécanique',
    price: 89.9,
    stock: 12,
  );
  Product souris = Product(
    reference: 'MS-200',
    label: 'Souris sans fil',
    price: 24.5,
  );

  clavier.describe();
  souris.describe();
}
```

**Résultat :**

```text
[KB-100] Clavier mécanique - 89.90 EUR - stock: 12
[MS-200] Souris sans fil - 24.50 EUR - stock: 0
```

Une référence, un libellé et un prix sont indispensables : ils sont `required`. Le stock peut légitimement valoir zéro : il a une valeur par défaut.

---

## 09.7 — Paramètres optionnels et valeurs par défaut

Dart distingue deux familles de paramètres optionnels.

```text
                     PARAMÈTRES D'UN CONSTRUCTEUR

     positionnels obligatoires        Player(this.name)
     positionnels optionnels    [ ]   Player(this.name, [this.level = 1])
     nommés optionnels          { }   Player({this.level = 1})
     nommés obligatoires        { }   Player({required this.name})
```

### 09.7.1 — Les positionnels optionnels avec `[ ]`

```dart
class Enemy {
  String name;
  int health;
  int level;

  Enemy(this.name, this.health, [this.level = 1]);

  void showStatus() {
    print('$name (niveau $level) - vie: $health');
  }
}

void main() {
  Enemy('Gobelin', 30).showStatus();
  Enemy('Orc', 60, 4).showStatus();
}
```

**Résultat :**

```text
Gobelin (niveau 1) - vie: 30
Orc (niveau 4) - vie: 60
```

Règles : les crochets viennent **après** les paramètres obligatoires ; un paramètre entre crochets doit avoir une valeur par défaut ; on ne peut pas sauter un paramètre.

### 09.7.2 — Les nommés optionnels avec `{ }`

Ils autorisent, eux, à sauter n'importe quel paramètre.

```dart
class Enemy {
  String name;
  int health;
  int damage;
  int level;

  Enemy({
    required this.name,
    this.health = 30,
    this.damage = 5,
    this.level = 1,
  });

  void showStatus() {
    print('$name | niv $level | vie $health | degats $damage');
  }
}

void main() {
  Enemy(name: 'Gobelin').showStatus();
  Enemy(name: 'Orc', health: 60).showStatus();
  Enemy(name: 'Dragon', level: 12, damage: 40, health: 500).showStatus();
}
```

**Résultat :**

```text
Gobelin | niv 1 | vie 30 | degats 5
Orc | niv 1 | vie 60 | degats 5
Dragon | niv 12 | vie 500 | degats 40
```

Le deuxième ennemi fournit `health` sans fournir `damage` : impossible avec des crochets, naturel avec des accolades.

> On ne peut pas mélanger `[ ]` et `{ }` dans un même constructeur. En pratique, choisissez les accolades : elles couvrent tous les besoins.

### 09.7.3 — Une valeur par défaut doit être une constante de compilation

C'est une contrainte technique importante :

```text
Inventory({this.gold = 100})              ->  autorisé : 100 est constant
Inventory({this.items = []})              ->  ERREUR
Inventory({this.date = DateTime.now()})   ->  ERREUR
```

```text
Error: Default values of an optional parameter must be constant.
```

Pour une liste vide, écrivez `const []` :

```dart
class Inventory {
  final String owner;
  int gold;
  List<String> items;

  Inventory({required this.owner, this.gold = 100, this.items = const []});
}

void main() {
  Inventory sac = Inventory(owner: 'Alex');
  print('${sac.owner} : ${sac.gold} pieces, ${sac.items.length} objet(s)');
}
```

**Résultat :**

```text
Alex : 100 pieces, 0 objet(s)
```

> **Attention :** une liste `const` est **non modifiable**. Si vous devez y ajouter des éléments, initialisez-la avec une initializer list (section 09.9).

---

## 09.8 — Les constructeurs nommés

Une classe peut posséder **plusieurs** constructeurs, à condition de leur donner des noms différents. Un **constructeur nommé** s'écrit `NomDeLaClasse.nomDuConstructeur`.

Le besoin est évident dans un jeu : certains objets reviennent toujours avec les mêmes réglages (un joueur débutant, un gobelin standard, un boss). Répéter ces valeurs à chaque appel est fastidieux et source d'incohérences.

```dart
class Player {
  String name;
  int health;
  int score;
  int level;

  Player({
    required this.name,
    required this.health,
    this.score = 0,
    this.level = 1,
  });

  Player.debutant(this.name)
      : health = 100,
        score = 0,
        level = 1;

  Player.veteran(this.name)
      : health = 200,
        score = 5000,
        level = 20;

  void showStatus() {
    print('$name | niv $level | vie $health | score $score');
  }
}

void main() {
  Player.debutant('Sophie').showStatus();
  Player.veteran('Samir').showStatus();
  Player(name: 'Alex', health: 120, score: 300, level: 4).showStatus();
}
```

**Résultat :**

```text
Sophie | niv 1 | vie 100 | score 0
Samir | niv 20 | vie 200 | score 5000
Alex | niv 4 | vie 120 | score 300
```

```text
Player.debutant(this.name)
  │      │
  │      └── nom du constructeur nommé (lowerCamelCase)
  └───────── nom de la classe, à l'identique
```

La partie après les deux-points `:` est une **initializer list**, détaillée en section 09.9.

### 09.8.1 — La redirection : un constructeur nommé appelle le principal

Recopier les mêmes affectations dans plusieurs constructeurs finit par créer des divergences. Dart permet de **rediriger** avec `: this(...)`.

```dart
class Enemy {
  String name;
  int health;
  int damage;
  bool isBoss;

  Enemy({
    required this.name,
    required this.health,
    required this.damage,
    this.isBoss = false,
  });

  Enemy.gobelin() : this(name: 'Gobelin', health: 30, damage: 5);

  Enemy.orc() : this(name: 'Orc', health: 60, damage: 12);

  Enemy.boss(String nom)
      : this(name: nom, health: 500, damage: 40, isBoss: true);

  void showStatus() {
    String marque = isBoss ? '[BOSS] ' : '';
    print('$marque$name | vie $health | degats $damage');
  }
}

void main() {
  List<Enemy> vague = [
    Enemy.gobelin(),
    Enemy.gobelin(),
    Enemy.orc(),
    Enemy.boss('Dragon Noir'),
  ];

  for (Enemy e in vague) {
    e.showStatus();
  }
}
```

**Résultat :**

```text
Gobelin | vie 30 | degats 5
Gobelin | vie 30 | degats 5
Orc | vie 60 | degats 12
[BOSS] Dragon Noir | vie 500 | degats 40
```

```text
Enemy.gobelin() : this(name: 'Gobelin', health: 30, damage: 5);
                    │
                    └── "this" désigne ici le constructeur PRINCIPAL
                        (celui qui n'a pas de nom)
```

Avantages : une seule vérité pour la construction, et tout ajout de propriété profite automatiquement à tous les constructeurs nommés. Le corps d'un constructeur redirigé doit rester **vide** (un simple point-virgule).

La lecture de `main()` est devenue une description du niveau : deux gobelins, un orc, un boss. C'est exactement l'objectif de la modélisation.

> **Conseil de nommage :** un constructeur nommé décrit un **cas d'usage**, pas une implémentation. `Player.debutant()`, `Enemy.boss()`, `Product.gratuit()` se lisent comme des phrases.

---

## 09.9 — L'initializer list `: x = ...`

L'**initializer list** est la partie située entre la parenthèse fermante du constructeur et son corps, introduite par deux-points `:`.

```text
Player(String nom) : health = 100, score = 0 {
        │                    │
        │                    └── INITIALIZER LIST
        └─────────────────────── paramètres

    // ici commence le CORPS
}
```

Elle s'exécute **avant** le corps, et c'est le seul endroit où l'on peut donner sa valeur à une propriété `final`.

```dart
class Weapon {
  final String name;
  final int damage;
  final int repairCost;

  Weapon(this.name, this.damage) : repairCost = damage * 3;

  void describe() {
    print('$name : $damage degats, reparation $repairCost pieces');
  }
}

void main() {
  Weapon('Épée de fer', 12).describe();
  Weapon('Hache', 20).describe();
}
```

**Résultat :**

```text
Épée de fer : 12 degats, reparation 36 pieces
Hache : 20 degats, reparation 60 pieces
```

### 09.9.1 — Ordre d'exécution exact

```text
1. les valeurs par défaut des paramètres sont appliquées
2. l'INITIALIZER LIST s'exécute        (: a = ..., b = ...)
3. les propriétés this.x du constructeur sont remplies
4. le CORPS du constructeur s'exécute  { ... }
```

Conséquence essentielle :

> Dans l'initializer list, `this` n'est **pas encore utilisable**. L'objet n'existe pas complètement.

```dart
class Enemy {
  final int health;
  final int maxHealth;

  // Enemy(this.health) : maxHealth = this.health;  // INTERDIT
  Enemy(int pointsDeVie)
      : health = pointsDeVie,
        maxHealth = pointsDeVie; // correct : on utilise le PARAMÈTRE

  void describe() {
    print('vie $health / $maxHealth');
  }
}

void main() {
  Enemy(30).describe();
}
```

**Résultat :**

```text
vie 30 / 30
```

Message si vous tentez la version interdite :

```text
Error: Can't access 'this' in a field initializer to read 'health'.
```

### 09.9.2 — Initializer list ou corps du constructeur ?

| Situation | Où écrire |
| --- | --- |
| remplir une propriété `final` | initializer list |
| calculer une valeur à partir d'un paramètre | initializer list |
| vérifier une valeur avec `assert` | initializer list |
| créer une liste modifiable | initializer list |
| afficher un message, journaliser | corps |
| appeler une méthode de l'objet | corps |

Exemple mêlant les deux :

```dart
class Player {
  final String name;
  final int health;
  final List<String> inventory;

  Player(this.name, this.health) : inventory = [] {
    inventory.add('Potion');
    print('Le joueur $name entre dans le jeu.');
  }
}

void main() {
  Player joueur = Player('Alex', 100);
  print(joueur.inventory);
}
```

**Résultat :**

```text
Le joueur Alex entre dans le jeu.
[Potion]
```

L'initializer list crée la liste vide (la référence est `final`, donc figée), et le corps la remplit.

---

## 09.10 — `assert` dans l'initializer list

Un `assert` est une **vérification de programmation** : « à cet endroit, cette condition doit forcément être vraie ».

Reprenons le défaut 3 de la section 09.1 et ajoutons des garde-fous.

```dart
class Player {
  final String name;
  final int health;

  Player(this.name, this.health)
      : assert(name != '', 'Le nom du joueur ne peut pas être vide'),
        assert(health > 0, 'La vie doit être strictement positive');

  void showStatus() {
    print('$name | vie: $health');
  }
}

void main() {
  Player valide = Player('Alex', 100);
  valide.showStatus();

  // Player invalide = Player('Bob', 0);
  // -> Assertion failed: "La vie doit être strictement positive"
}
```

**Résultat :**

```text
Alex | vie: 100
```

Un `assert` prend deux arguments : la condition, puis le message affiché en cas d'échec. Écrivez toujours le message : c'est lui que vous lirez à trois heures du matin.

### 09.10.1 — Point capital : `assert` disparaît en production

C'est la caractéristique la plus mal comprise d'`assert`.

```text
MODE DEBUG (DartPad, flutter run)   -> les assert sont ACTIFS
MODE RELEASE (application publiée)  -> les assert sont IGNORÉS
```

> `assert` est un outil de **détection de bugs du programmeur**, pas un outil de **validation de données utilisateur**.

| Cas | Bon outil |
| --- | --- |
| un développeur passe `health: -50` | `assert` |
| un développeur oublie `name` | `required` |
| un utilisateur saisit un prix négatif dans un formulaire | vérification explicite + exception (chapitre 13) |
| une réponse réseau contient une valeur absurde | vérification explicite + exception (chapitre 13) |

### 09.10.2 — Plusieurs `assert` sur `Product`

```dart
class Product {
  final String reference;
  final String label;
  final double price;
  final int stock;

  Product({
    required this.reference,
    required this.label,
    required this.price,
    this.stock = 0,
  })  : assert(reference != '', 'La référence est obligatoire'),
        assert(label != '', 'Le libellé est obligatoire'),
        assert(price >= 0, 'Le prix ne peut pas être négatif'),
        assert(stock >= 0, 'Le stock ne peut pas être négatif');

  void describe() {
    print('[$reference] $label - ${price.toStringAsFixed(2)} EUR');
  }
}

void main() {
  Product(
    reference: 'KB-100',
    label: 'Clavier mécanique',
    price: 89.9,
    stock: 12,
  ).describe();
}
```

**Résultat :**

```text
[KB-100] Clavier mécanique - 89.90 EUR
```

Remarquez la syntaxe : quand un constructeur possède des paramètres nommés, la parenthèse fermante `)` est suivie de `:`, puis des `assert` séparés par des virgules.

---

## 09.11 — Le constructeur `const`

Certains objets ne changent **jamais** : une position sur une grille, une couleur, une fiche de catalogue. Pour eux, Dart propose les constructeurs `const`.

```text
Pour écrire un constructeur const, il faut :

  1. que TOUTES les propriétés soient final
  2. que le constructeur n'ait PAS de corps { }
  3. que les valeurs passées soient elles-mêmes des constantes
```

```dart
class Position {
  final int x;
  final int y;

  const Position(this.x, this.y);

  void show() {
    print('($x, $y)');
  }
}

void main() {
  const Position depart = Position(0, 0);
  const Position arrivee = Position(10, 4);

  depart.show();
  arrivee.show();
}
```

**Résultat :**

```text
(0, 0)
(10, 4)
```

### 09.11.1 — Ce que `const` apporte : l'objet unique

Deux objets `const` construits avec les mêmes valeurs sont **le même objet en mémoire**. On dit qu'ils sont « canonisés ».

```dart
class Position {
  final int x;
  final int y;

  const Position(this.x, this.y);
}

void main() {
  const Position a = Position(3, 5);
  const Position b = Position(3, 5);

  Position c = Position(3, 5);
  Position d = Position(3, 5);

  print(identical(a, b));
  print(identical(c, d));
}
```

**Résultat :**

```text
true
false
```

```text
AVEC const                          SANS const

a ─┐                                c ──> [objet 1 : x=3 y=5]
   ├──> [objet unique x=3 y=5]      d ──> [objet 2 : x=3 y=5]
b ─┘
                                    Deux zones mémoire distinctes,
Une seule zone mémoire.             au contenu identique.
```

`identical()` répond à la question « est-ce littéralement le même objet ? ». Notez qu'un constructeur `const` **peut** être appelé sans `const` : vous obtenez alors un objet ordinaire.

> **Règle simple :** déclarez le constructeur `const` dès que la classe le permet. L'appelant décide ensuite s'il veut profiter de la constante.

### 09.11.2 — Où l'utiliser en pratique

| Bon candidat pour `const` | Mauvais candidat |
| --- | --- |
| `Position(x, y)` sur une grille | `Player` (vie et score changent) |
| `Color(r, g, b)` | `Enemy` (vie change) |
| une fiche d'arme figée du catalogue | `Inventory` (contenu change) |
| une configuration de niveau | un panier de commande |

En Flutter, `const` devant un widget évite de le reconstruire inutilement. C'est un gain de performance direct, et l'habitude se prend ici.

---

## 09.12 — Le constructeur `factory`

Un constructeur ordinaire crée **forcément** un nouvel objet. Un constructeur `factory` (« fabrique ») est libre : il **retourne** un objet, mais il peut choisir lequel.

```text
CONSTRUCTEUR ORDINAIRE          CONSTRUCTEUR FACTORY

Player(...)                     factory Player(...)
  - crée toujours un objet neuf   - contient du CODE avant de décider
  - ne peut pas faire return      - DOIT faire "return unObjet;"
  - remplit les propriétés        - peut renvoyer un objet déjà existant
```

### 09.12.1 — Cas 1 : choisir l'objet à créer

```dart
class Enemy {
  final String name;
  final int health;
  final int damage;

  Enemy({required this.name, required this.health, required this.damage});

  factory Enemy.selonNiveau(int niveau) {
    if (niveau >= 10) {
      return Enemy(name: 'Dragon', health: 500, damage: 40);
    }
    if (niveau >= 5) {
      return Enemy(name: 'Orc', health: 60, damage: 12);
    }
    return Enemy(name: 'Gobelin', health: 30, damage: 5);
  }

  void showStatus() {
    print('$name | vie $health | degats $damage');
  }
}

void main() {
  Enemy.selonNiveau(1).showStatus();
  Enemy.selonNiveau(6).showStatus();
  Enemy.selonNiveau(15).showStatus();
}
```

**Résultat :**

```text
Gobelin | vie 30 | degats 5
Orc | vie 60 | degats 12
Dragon | vie 500 | degats 40
```

Un constructeur ordinaire ne pourrait pas faire cela : il ne peut contenir ni `return`, ni décision sur l'objet renvoyé.

### 09.12.2 — Cas 2 : réutiliser un objet déjà créé

Dans un catalogue d'armes, inutile de recréer l'épée de fer mille fois.

```dart
class Weapon {
  final String name;
  final int damage;

  static final Map<String, Weapon> _catalogue = {};

  Weapon._interne(this.name, this.damage);

  factory Weapon(String name, int damage) {
    if (_catalogue.containsKey(name)) {
      print('Réutilisation de $name');
      return _catalogue[name]!;
    }
    print('Création de $name');
    final Weapon arme = Weapon._interne(name, damage);
    _catalogue[name] = arme;
    return arme;
  }
}

void main() {
  Weapon a = Weapon('Épée de fer', 12);
  Weapon b = Weapon('Épée de fer', 12);
  Weapon c = Weapon('Hache', 20);

  print(identical(a, b));
  print(identical(a, c));
}
```

**Résultat :**

```text
Création de Épée de fer
Réutilisation de Épée de fer
Création de Hache
true
false
```

Trois éléments nouveaux : `static` désigne une donnée qui appartient à la **classe** et non à chaque objet ; `Weapon._interne(...)` est un constructeur nommé réservé à l'usage interne ; `_catalogue[name]!` affirme à Dart que la valeur n'est pas nulle (détaillé au chapitre 12).

### 09.12.3 — Cas 3 : construire depuis un autre format

C'est l'usage le plus fréquent dans les applications réelles.

```dart
class Product {
  final String reference;
  final String label;
  final double price;

  Product({required this.reference, required this.label, required this.price});

  factory Product.depuisMap(Map<String, Object> donnees) {
    return Product(
      reference: donnees['reference'] as String,
      label: donnees['label'] as String,
      price: donnees['price'] as double,
    );
  }

  void describe() {
    print('[$reference] $label - ${price.toStringAsFixed(2)} EUR');
  }
}

void main() {
  Map<String, Object> ligne = {
    'reference': 'KB-100',
    'label': 'Clavier mécanique',
    'price': 89.9,
  };

  Product.depuisMap(ligne).describe();
}
```

**Résultat :**

```text
[KB-100] Clavier mécanique - 89.90 EUR
```

Vous retrouverez exactement ce schéma au chapitre 17, consacré au JSON.

### 09.12.4 — Quand ne PAS utiliser `factory`

| Besoin | Outil correct |
| --- | --- |
| remplir des propriétés | constructeur ordinaire |
| proposer des réglages tout faits | constructeur nommé |
| calculer une propriété | initializer list |
| décider quel objet renvoyer | `factory` |
| réutiliser un objet existant | `factory` |
| convertir une `Map` en objet | `factory` |

> **Règle de décision :** si votre constructeur n'a pas besoin d'écrire `return`, il n'a pas besoin d'être `factory`.

---

## 09.13 — Les getters

Un **getter** est une méthode déguisée en propriété. On l'appelle **sans parenthèses**.

Le besoin : connaître la vie restante en pourcentage. Avec une méthode, on écrirait `joueur.pourcentageVie()`. Mais ce n'est pas une action, c'est une **information** : les parenthèses laissent croire qu'il se passe quelque chose.

On remplace donc `type nom()` par `type get nom`.

```dart
class Player {
  final String name;
  int health;
  final int maxHealth;

  Player({required this.name, required this.maxHealth}) : health = maxHealth;

  double get pourcentageVie {
    return health / maxHealth * 100;
  }
}

void main() {
  Player joueur = Player(name: 'Alex', maxHealth: 200);
  joueur.health = 50;

  print('${joueur.pourcentageVie} %');
}
```

**Résultat :**

```text
25.0 %
```

```text
MÉTHODE                              GETTER

double pourcentageVie() { ... }      double get pourcentageVie { ... }
                     ▲                      ▲
              parenthèses               mot-clé "get",
              obligatoires              aucune parenthèse

joueur.pourcentageVie()              joueur.pourcentageVie
```

### 09.13.1 — La forme fléchée `=>`

Quand le getter tient en une expression, on utilise la flèche.

```dart
class Player {
  final String name;
  int health;
  final int maxHealth;

  Player({required this.name, required this.maxHealth}) : health = maxHealth;

  double get pourcentageVie => health / maxHealth * 100;
  bool get estVivant => health > 0;
  String get etiquette => '$name ($health/$maxHealth)';
}

void main() {
  Player joueur = Player(name: 'Alex', maxHealth: 100);
  joueur.health = 40;

  print(joueur.etiquette);
  print(joueur.pourcentageVie);
  print(joueur.estVivant);
}
```

**Résultat :**

```text
Alex (40/100)
40.0
true
```

Un getter ne stocke rien : il **recalcule** à chaque lecture. La valeur suit donc automatiquement l'état de l'objet, et il est impossible d'oublier de la mettre à jour.

### 09.13.2 — Des getters sur `Product`

```dart
class Product {
  final String label;
  final double priceHT;
  final int stock;

  Product({required this.label, required this.priceHT, this.stock = 0});

  double get priceTTC => priceHT * 1.20;
  bool get enStock => stock > 0;
  String get disponibilite => enStock ? 'Disponible' : 'Rupture';
}

void main() {
  Product clavier = Product(label: 'Clavier', priceHT: 100.0, stock: 5);
  Product souris = Product(label: 'Souris', priceHT: 20.0);

  print('${clavier.label} : ${clavier.priceTTC} EUR - ${clavier.disponibilite}');
  print('${souris.label} : ${souris.priceTTC} EUR - ${souris.disponibilite}');
}
```

**Résultat :**

```text
Clavier : 120.0 EUR - Disponible
Souris : 24.0 EUR - Rupture
```

Remarquez que `disponibilite` utilise le getter `enStock` : un getter peut librement en appeler un autre.

---

## 09.14 — Les setters

Un **setter** est le pendant du getter : une méthode déguisée en propriété que l'on **écrit**. Il s'écrit `set nom(Type valeur)` et ne retourne rien.

Le besoin : empêcher la vie de sortir de l'intervalle `[0, maxHealth]`. Pour intercepter l'écriture, la propriété réelle est renommée avec un tiret bas `_`.

```dart
class Player {
  final String name;
  final int maxHealth;
  int _health;

  Player({required this.name, required this.maxHealth}) : _health = maxHealth;

  int get health => _health;

  set health(int valeur) {
    if (valeur < 0) {
      _health = 0;
    } else if (valeur > maxHealth) {
      _health = maxHealth;
    } else {
      _health = valeur;
    }
  }
}

void main() {
  Player joueur = Player(name: 'Alex', maxHealth: 100);

  joueur.health = 60;
  print(joueur.health);

  joueur.health = 999999;
  print(joueur.health);

  joueur.health = -400;
  print(joueur.health);
}
```

**Résultat :**

```text
60
100
0
```

```text
joueur.health = 999999
        │
        ▼
  set health(valeur)      <-- le SETTER intercepte
        │
        ├── vérifie / corrige la valeur
        ▼
     _health = 100        <-- la vraie propriété, cachée

print(joueur.health)
        │
        ▼
  get health              <-- le GETTER renvoie _health
        ▼
      100
```

> **Le tiret bas `_` devant un nom** signifie « privé au fichier ». C'est la base de l'encapsulation, étudiée au chapitre 10. Retenez pour l'instant qu'il sert à cacher la vraie propriété derrière le couple getter / setter.

### 09.14.1 — Règles de syntaxe

```text
set health(int valeur) { ... }        ->  correct
void set health(int valeur) { ... }   ->  toléré mais déconseillé
int set health(int valeur) { ... }    ->  erreur
```

Un setter reçoit exactement **un** paramètre, ni zéro ni deux.

### 09.14.2 — Quand écrire un setter, et quand s'en passer

| Situation | Décision |
| --- | --- |
| la valeur ne doit jamais changer | propriété `final`, aucun setter |
| la valeur change librement | propriété publique simple |
| la valeur doit être bornée ou nettoyée | getter + setter |
| la valeur se déduit d'autres propriétés | getter seul (getter calculé) |
| l'écriture doit déclencher une action | getter + setter |

> **Piège fréquent :** un setter qui se contente de `_x = valeur;` est inutile. Utilisez alors une propriété publique ordinaire.

---

## 09.15 — Les getters calculés

Un **getter calculé** produit une information dérivée de l'état de l'objet, sans jamais la stocker.

```text
APPROCHE 1 : STOCKER              APPROCHE 2 : CALCULER

bool estVivant = true;            bool get estVivant => health > 0;

Il faut penser à écrire           Rien à maintenir.
  estVivant = health > 0;         La valeur est toujours juste.
à CHAQUE modification de health.

-> risque d'incohérence           -> aucune incohérence possible
```

Le bug de l'approche 1, en pratique :

```dart
class Player {
  int health = 100;
  bool estVivant = true;

  void subirDegats(int degats) {
    health = health - degats;
    // on a oublié de mettre à jour estVivant
  }
}

void main() {
  Player joueur = Player();
  joueur.subirDegats(150);
  print('vie: ${joueur.health} | vivant: ${joueur.estVivant}');
}
```

**Résultat :**

```text
vie: -50 | vivant: true
```

Un mort qui se croit vivant. Le getter calculé supprime ce type de bug.

### 09.15.1 — Une classe `Player` complète

```dart
class Player {
  final String name;
  final int maxHealth;
  int _health;
  int score;

  Player({required this.name, this.maxHealth = 100, this.score = 0})
      : _health = maxHealth,
        assert(maxHealth > 0, 'maxHealth doit être positif');

  int get health => _health;

  set health(int valeur) {
    _health = valeur.clamp(0, maxHealth);
  }

  bool get estVivant => _health > 0;
  bool get estMort => !estVivant;
  double get pourcentageVie => _health / maxHealth * 100;
  bool get estEnDanger => estVivant && pourcentageVie < 25;

  String get barreDeVie {
    int pleins = (pourcentageVie / 10).round();
    return '[${'#' * pleins}${'.' * (10 - pleins)}]';
  }

  void subirDegats(int degats) {
    health = _health - degats;
  }

  void seSoigner(int soin) {
    health = _health + soin;
  }

  void showStatus() {
    print('$name $barreDeVie $_health/$maxHealth '
        '| vivant: $estVivant | danger: $estEnDanger');
  }
}

void main() {
  Player joueur = Player(name: 'Alex', maxHealth: 100);
  joueur.showStatus();

  joueur.subirDegats(40);
  joueur.showStatus();

  joueur.subirDegats(45);
  joueur.showStatus();

  joueur.seSoigner(1000);
  joueur.showStatus();

  joueur.subirDegats(9999);
  joueur.showStatus();
}
```

**Résultat :**

```text
Alex [##########] 100/100 | vivant: true | danger: false
Alex [######....] 60/100 | vivant: true | danger: false
Alex [##........] 15/100 | vivant: true | danger: true
Alex [##########] 100/100 | vivant: true | danger: false
Alex [..........] 0/100 | vivant: false | danger: false
```

Points remarquables :

- `clamp(0, maxHealth)` ramène automatiquement la valeur dans l'intervalle ;
- `'#' * pleins` répète une chaîne : c'est l'opérateur `*` appliqué à un `String` ;
- `subirDegats` passe par le **setter**, donc bénéficie du bornage sans le réécrire ;
- aucune des propriétés `estVivant`, `estMort`, `pourcentageVie` n'est stockée.

### 09.15.2 — Getters calculés sur un panier de commande

```dart
class Product {
  final String label;
  final double priceHT;
  final int quantity;

  const Product({
    required this.label,
    required this.priceHT,
    this.quantity = 1,
  });

  double get totalHT => priceHT * quantity;
  double get tva => totalHT * 0.20;
  double get totalTTC => totalHT + tva;
  String get resume => '$label x$quantity = ${totalTTC.toStringAsFixed(2)} EUR';
}

void main() {
  const List<Product> panier = [
    Product(label: 'Clavier', priceHT: 100.0, quantity: 2),
    Product(label: 'Souris', priceHT: 20.0),
    Product(label: 'Écran', priceHT: 250.0),
  ];

  double total = 0;
  for (Product p in panier) {
    print(p.resume);
    total = total + p.totalTTC;
  }

  print('TOTAL : ${total.toStringAsFixed(2)} EUR');
}
```

**Résultat :**

```text
Clavier x2 = 240.00 EUR
Souris x1 = 24.00 EUR
Écran x1 = 300.00 EUR
TOTAL : 564.00 EUR
```

---

## 09.16 — Redéfinir `toString()`

Affichez un objet directement, et vous obtenez un message inutile :

```dart
class Player {
  final String name;
  final int health;

  Player(this.name, this.health);
}

void main() {
  print(Player('Alex', 100));
}
```

**Résultat :**

```text
Instance of 'Player'
```

Toute classe Dart possède déjà une méthode `toString()`. Nous la **redéfinissons**.

```dart
class Player {
  final String name;
  final int health;

  Player(this.name, this.health);

  @override
  String toString() {
    return 'Player(name: $name, health: $health)';
  }
}

void main() {
  Player joueur = Player('Alex', 100);
  print(joueur);
  print('Le joueur : $joueur');
}
```

**Résultat :**

```text
Player(name: Alex, health: 100)
Le joueur : Player(name: Alex, health: 100)
```

Trois remarques :

- `@override` est une **annotation** qui signale que l'on remplace une méthode existante. Elle est facultative mais fortement recommandée : elle permet à Dart de détecter une faute de frappe dans le nom ;
- `print(objet)` appelle automatiquement `toString()` ;
- l'interpolation `'$joueur'` l'appelle également.

### 09.16.1 — Deux styles, et l'affichage des listes

```dart
class Enemy {
  final String name;
  final int health;

  Enemy(this.name, this.health);

  @override
  String toString() => 'Enemy(name: $name, health: $health)';
}

class Weapon {
  final String name;
  final int damage;

  Weapon(this.name, this.damage);

  @override
  String toString() => '$name ($damage degats)';
}

void main() {
  print(Weapon('Épée de fer', 12));

  List<Enemy> vague = [Enemy('Gobelin', 30), Enemy('Orc', 60)];
  print(vague);
}
```

**Résultat :**

```text
Épée de fer (12 degats)
[Enemy(name: Gobelin, health: 30), Enemy(name: Orc, health: 60)]
```

Sans `toString()`, la dernière ligne aurait affiché `[Instance of 'Enemy', Instance of 'Enemy']`.

| Style | Forme | Usage |
| --- | --- | --- |
| technique | `Enemy(name: ..., health: ...)` | débogage, journaux |
| lisible | `Épée de fer (12 degats)` | affichage à l'utilisateur |

Le style technique est la convention du monde Dart.

> **Bonne pratique :** écrivez un `toString()` dans **toutes** vos classes de données. Le temps gagné en débogage est considérable.

---

## 09.17 — Modéliser un domaine : de l'énoncé aux classes

Savoir écrire un constructeur ne suffit pas. Il faut savoir **décider** quelles classes écrire.

```text
ÉNONCÉ

Dans notre jeu, un joueur possède un pseudo, des points de vie
(au maximum 100), un niveau et une bourse d'or. Il porte une arme.
Une arme a un nom, des dégâts et une durabilité. Le joueur affronte
des ennemis. Un ennemi a un nom, des points de vie, des dégâts, et
peut être un boss. Quand le joueur tue un ennemi, il gagne de l'or.
```

### 09.17.1 — Méthode en quatre étapes

```text
ÉTAPE 1  Souligner les NOMS COMMUNS      -> candidats CLASSES
ÉTAPE 2  Souligner les CARACTÉRISTIQUES  -> PROPRIÉTÉS
ÉTAPE 3  Souligner les VERBES            -> MÉTHODES
ÉTAPE 4  Repérer les LIENS entre noms    -> propriétés d'un autre type
```

**Étape 1.** Les noms communs sont : joueur, arme, ennemi, or, niveau, dégâts, durabilité, boss. Tous ne sont pas des classes : une classe est un nom qui possède **plusieurs** caractéristiques.

```text
joueur  -> pseudo, vie, niveau, or, arme       -> CLASSE Player
arme    -> nom, dégâts, durabilité             -> CLASSE Weapon
ennemi  -> nom, vie, dégâts, boss              -> CLASSE Enemy
or      -> un simple nombre                    -> propriété int
niveau  -> un simple nombre                    -> propriété int
```

**Étape 2 — les propriétés :**

```text
Player : name (String), health (int), maxHealth (int),
         level (int), gold (int), weapon (Weapon)
Weapon : name (String), damage (int), durability (int)
Enemy  : name (String), health (int), damage (int), isBoss (bool)
```

**Étape 3 — les verbes :**

```text
"affronte", "tue", "gagne de l'or"  -> Player.attaquer(Enemy cible)
"perd de la vie"                    -> Enemy.subirDegats(int)
"l'arme s'use"                      -> Weapon.utiliser()
```

**Étape 4 — les liens :**

```text
┌────────────┐   possède    ┌────────────┐
│   Player   │─────────────>│   Weapon   │
│            │              │            │
│ weapon ────┼──────────────┤ name       │
└─────┬──────┘              │ damage     │
      │                     │ durability │
      │ attaque             └────────────┘
      ▼
┌────────────┐
│   Enemy    │
│ name       │
│ health     │
│ damage     │
│ isBoss     │
└────────────┘
```

`Player` possède une propriété de type `Weapon`. C'est une **composition** : un objet contient un autre objet.

### 09.17.2 — Le code complet

```dart
class Weapon {
  final String name;
  final int damage;
  int durability;

  Weapon({required this.name, required this.damage, this.durability = 10})
      : assert(damage > 0, 'Les dégâts doivent être positifs');

  Weapon.poing() : this(name: 'Poing nu', damage: 1, durability: 9999);

  bool get estCassee => durability <= 0;

  void utiliser() {
    if (durability > 0) {
      durability = durability - 1;
    }
  }

  @override
  String toString() => 'Weapon($name, $damage degats, $durability usages)';
}

class Enemy {
  final String name;
  final int maxHealth;
  final int damage;
  final bool isBoss;
  int _health;

  Enemy({
    required this.name,
    required this.maxHealth,
    required this.damage,
    this.isBoss = false,
  }) : _health = maxHealth;

  Enemy.gobelin() : this(name: 'Gobelin', maxHealth: 30, damage: 5);
  Enemy.orc() : this(name: 'Orc', maxHealth: 60, damage: 12);
  Enemy.boss(String nom)
      : this(name: nom, maxHealth: 300, damage: 35, isBoss: true);

  int get health => _health;
  bool get estVivant => _health > 0;
  int get recompense => isBoss ? maxHealth * 3 : maxHealth;

  void subirDegats(int degats) {
    _health = (_health - degats).clamp(0, maxHealth);
  }

  @override
  String toString() => 'Enemy($name, $_health/$maxHealth PV)';
}

class Player {
  final String name;
  final int maxHealth;
  int _health;
  int level;
  int gold;
  Weapon weapon;

  Player({required this.name, this.maxHealth = 100, this.level = 1})
      : _health = maxHealth,
        gold = 0,
        weapon = Weapon.poing();

  int get health => _health;
  bool get estVivant => _health > 0;
  double get pourcentageVie => _health / maxHealth * 100;

  void equiper(Weapon arme) {
    weapon = arme;
    print('$name équipe ${arme.name}.');
  }

  void attaquer(Enemy cible) {
    if (!estVivant) {
      print('$name est hors de combat.');
      return;
    }
    if (weapon.estCassee) {
      print('${weapon.name} est cassée : $name se bat au poing.');
      weapon = Weapon.poing();
    }

    cible.subirDegats(weapon.damage);
    weapon.utiliser();
    print('$name frappe ${cible.name} pour ${weapon.damage} degats. '
        '(${cible.health} PV restants)');

    if (!cible.estVivant) {
      gold = gold + cible.recompense;
      print('${cible.name} est vaincu. $name gagne ${cible.recompense} or.');
    }
  }

  @override
  String toString() =>
      'Player($name, niv $level, $_health/$maxHealth PV, $gold or)';
}

void main() {
  Player alex = Player(name: 'Alex');
  alex.equiper(Weapon(name: 'Épée de fer', damage: 12, durability: 20));

  Enemy gobelin = Enemy.gobelin();

  while (gobelin.estVivant) {
    alex.attaquer(gobelin);
  }

  print(alex);
}
```

**Résultat :**

```text
Alex équipe Épée de fer.
Alex frappe Gobelin pour 12 degats. (18 PV restants)
Alex frappe Gobelin pour 12 degats. (6 PV restants)
Alex frappe Gobelin pour 12 degats. (0 PV restants)
Gobelin est vaincu. Alex gagne 30 or.
Player(Alex, niv 1, 100/100 PV, 30 or)
```

Chaque classe reste courte et compréhensible. Aucune ne connaît les détails internes des autres : `Player` demande simplement à `Enemy` de subir des dégâts.

---

## 09.18 — Bonnes pratiques de modélisation

```text
1.  Une classe = une responsabilité claire, exprimable en une phrase.
2.  Le constructeur doit produire un objet VALIDE, jamais à moitié rempli.
3.  Tout ce qui ne change pas est final.
4.  Au-delà de deux paramètres, utiliser des paramètres nommés.
5.  Ce qui est indispensable est required ; le reste a une valeur par défaut.
6.  Ne jamais stocker ce qui peut être calculé : préférer un getter.
7.  Un setter ne sert que s'il vérifie, borne ou normalise.
8.  Les cas d'usage répétés méritent un constructeur nommé.
9.  Toute classe de données mérite un toString().
10. Nommer avec les mots du domaine, pas avec des mots techniques.
```

### 09.18.1 — Règle 1 : une responsabilité par classe

Contre-exemple à ne pas suivre :

```dart
class Jeu {
  String nomJoueur = '';
  int vieJoueur = 0;
  String nomEnnemi = '';
  int vieEnnemi = 0;
  String nomArme = '';
  int degatsArme = 0;
}
```

Cette classe est un sac. Dès qu'un deuxième ennemi apparaît, elle explose. Test rapide : si vous ne pouvez pas décrire la classe en une phrase sans dire « et », elle en fait trop.

```text
"Player représente un joueur."                        -> bon
"Jeu représente le joueur ET l'ennemi ET l'arme..."   -> mauvais
```

### 09.18.2 — Règle 3 : `final` par défaut

Posez-vous la question pour chaque propriété : « cette valeur change-t-elle pendant la vie de l'objet ? »

| Propriété | Change ? | Déclaration |
| --- | --- | --- |
| `Player.name` | non | `final String name;` |
| `Player.health` | oui | `int _health;` |
| `Player.maxHealth` | non | `final int maxHealth;` |
| `Weapon.damage` | non | `final int damage;` |
| `Weapon.durability` | oui | `int durability;` |
| `Product.reference` | non | `final String reference;` |
| `Product.stock` | oui | `int stock;` |

Commencez toujours par `final`. Retirez-le seulement quand le compilateur vous y oblige.

### 09.18.3 — Règle 6 : calculer plutôt que stocker

```text
À STOCKER (donnée de base)        À CALCULER (donnée dérivée)

name                              estVivant       (health > 0)
health                            pourcentageVie  (health / maxHealth)
maxHealth                         estEnDanger     (pourcentage < 25)
priceHT                           priceTTC        (priceHT * 1.2)
quantity                          totalTTC        (priceTTC * quantity)
```

Question test : « puis-je retrouver cette valeur à partir des autres ? » Si oui, c'est un getter.

### 09.18.4 — Règle 10 : le vocabulaire du domaine

| À éviter | À préférer |
| --- | --- |
| `Data1`, `Info`, `Manager` | `Player`, `Weapon`, `Order` |
| `x`, `val`, `tmp` | `damage`, `price`, `stock` |
| `flag` | `isBoss`, `isDigital` |
| `doStuff()` | `attaquer()`, `seSoigner()` |

Une bonne modélisation se lit comme la description du jeu, pas comme du code.

### 09.18.5 — Modèle de classe à recopier

Voici le squelette type d'une classe de données bien écrite.

```dart
class Potion {
  // 1. propriétés immuables
  final String name;
  final int healAmount;

  // 2. propriétés mutables (cachées si contrôlées)
  int _uses;

  // 3. constructeur principal : nommé + required + assert
  Potion({required this.name, required this.healAmount, int uses = 1})
      : _uses = uses,
        assert(healAmount > 0, 'Le soin doit être positif'),
        assert(uses > 0, 'Il faut au moins un usage');

  // 4. constructeurs nommés (cas d'usage)
  Potion.petite() : this(name: 'Petite potion', healAmount: 20);
  Potion.grande() : this(name: 'Grande potion', healAmount: 75, uses: 3);

  // 5. getters
  int get uses => _uses;
  bool get estVide => _uses <= 0;

  // 6. méthodes (actions)
  int boire() {
    if (estVide) {
      print('$name est vide.');
      return 0;
    }
    _uses = _uses - 1;
    return healAmount;
  }

  // 7. toString
  @override
  String toString() => 'Potion($name, +$healAmount PV, $_uses usage(s))';
}

void main() {
  Potion grande = Potion.grande();
  print(grande);

  print('Soin recu : ${grande.boire()}');
  print('Soin recu : ${grande.boire()}');
  print('Soin recu : ${grande.boire()}');
  print('Soin recu : ${grande.boire()}');

  print(grande);
}
```

**Résultat :**

```text
Potion(Grande potion, +75 PV, 3 usage(s))
Soin recu : 75
Soin recu : 75
Soin recu : 75
Grande potion est vide.
Soin recu : 0
Potion(Grande potion, +75 PV, 0 usage(s))
```

L'ordre des sections (propriétés, constructeurs, getters, méthodes, `toString`) est une convention largement suivie dans l'écosystème Dart. Adoptez-la.
---

## 09.19 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `Error: Too few positional arguments: 1 required, 0 given.` | vous écrivez `Player()` alors qu'un constructeur personnalisé exige des arguments | fournir les arguments, ou ajouter des valeurs par défaut |
| La propriété reste vide après construction | `name = name;` dans le corps : le paramètre s'écrase lui-même | écrire `this.name = name;` ou utiliser `Player(this.name)` |
| `Error: Constructors can't have a return type.` | vous avez écrit `void Player()` ou `Player Player()` | supprimer le type de retour |
| `Error: Required named parameter 'name' must be provided.` | un paramètre `required` n'a pas été fourni à l'appel | ajouter `name: ...` à l'appel |
| `Error: Named optional parameters can't start with an underscore.` | vous avez écrit `required this._health` | exposer un paramètre public et l'affecter dans l'initializer list |
| `Error: Default values of an optional parameter must be constant.` | `this.items = []` ou `= DateTime.now()` | utiliser `const []`, ou initialiser dans l'initializer list |
| `Error: Can't access 'this' in a field initializer.` | vous utilisez `this.x` dans l'initializer list | utiliser le paramètre reçu, pas la propriété |
| `Error: 'health' can't be used as a setter because it's final.` | vous tentez `objet.health = ...` sur une propriété `final` | retirer `final`, ou passer par une méthode |
| `Error: Can't define a const constructor for a class with non-final fields.` | une propriété n'est pas `final` | rendre toutes les propriétés `final`, ou retirer `const` |
| `Error: A const constructor can't have a body.` | vous avez mis `{ ... }` après un constructeur `const` | déplacer le code dans l'initializer list, ou retirer `const` |
| `Error: Constructors can't have a body when they redirect.` | `Enemy.orc() : this(...) { print('...'); }` | laisser le corps vide, terminer par `;` |
| `Error: A factory constructor must return a value.` | un `factory` sans `return` | ajouter `return unObjet;` dans toutes les branches |
| `print(objet)` affiche `Instance of 'Player'` | `toString()` n'est pas redéfini | ajouter `@override String toString() => ...;` |
| `joueur.pourcentageVie()` ne compile pas | `pourcentageVie` est un getter, pas une méthode | appeler sans parenthèses : `joueur.pourcentageVie` |
| `Error: Stack Overflow` dans un getter | `int get health => health;` : le getter s'appelle lui-même | renvoyer la propriété privée : `=> _health;` |
| Un `assert` ne se déclenche jamais | vous êtes en mode release | utiliser une vérification explicite pour les données utilisateur |
| Deux objets identiques ne sont pas `identical` | vous avez oublié `const` à l'appel | écrire `const Position(3, 5)` |
| `Error: Can't have modifier 'required' with a default value.` | `required this.name = 'X'` | choisir : soit `required`, soit une valeur par défaut |

---

## 09.20 — Résumé du chapitre

| Notion | Syntaxe | Quand l'utiliser |
| --- | --- | --- |
| Constructeur par défaut | `Player()` | classe simple, sans donnée obligatoire |
| Constructeur personnalisé | `Player(String n) { name = n; }` | recevoir des valeurs à la création |
| `this` | `this.name = name;` | lever l'ambiguïté paramètre / propriété |
| Raccourci `this.x` | `Player(this.name);` | toujours, dès que le corps se réduit à des affectations |
| Paramètres nommés `{ }` | `Player({this.name = ''})` | dès trois paramètres, pour la lisibilité |
| `required` | `Player({required this.name})` | une donnée est indispensable à la validité |
| Positionnels optionnels `[ ]` | `Enemy(this.name, [this.level = 1])` | un ou deux paramètres facultatifs de fin |
| Valeur par défaut | `this.score = 0` | il existe une valeur raisonnable de départ |
| Constructeur nommé | `Player.debutant('Alex')` | un cas d'usage récurrent et bien identifié |
| Redirection | `Enemy.orc() : this(...)` | éviter de dupliquer la logique de construction |
| Initializer list | `Player(this.name) : health = 100;` | remplir une propriété `final` ou calculée |
| `assert` | `: assert(health > 0, '...')` | détecter une erreur de programmation en développement |
| Constructeur `const` | `const Position(this.x, this.y);` | objet totalement immuable et réutilisable |
| Constructeur `factory` | `factory Enemy.selonNiveau(int n)` | choisir, réutiliser ou convertir un objet |
| Getter | `int get health => _health;` | exposer une valeur en lecture |
| Setter | `set health(int v) { ... }` | contrôler, borner ou nettoyer une écriture |
| Getter calculé | `bool get estVivant => health > 0;` | la valeur se déduit d'autres propriétés |
| `toString()` | `@override String toString() => '...';` | toujours, sur toute classe de données |
| Composition | `Weapon weapon;` dans `Player` | un objet possède un autre objet |

---

## 09.21 — Exercices

Travaillez dans DartPad. Écrivez d'abord votre solution complète, puis seulement ensuite comparez avec la correction.

### Exercice 1 — Première potion (facile)

Écrivez une classe `Potion` avec deux propriétés `final` : `name` (`String`) et `healAmount` (`int`).
Écrivez un constructeur **positionnel** classique (avec un corps et `this.`).
Dans `main()`, créez une potion `'Potion de soin'` qui rend `25` points de vie, et affichez-la sous la forme `Potion de soin : +25 PV`.

### Exercice 2 — Le raccourci (facile)

Réécrivez l'exercice 1 en utilisant la syntaxe raccourcie `Potion(this.name, this.healAmount);`.
Ajoutez une méthode `describe()` qui affiche la même phrase.
Créez trois potions différentes et affichez-les.

### Exercice 3 — Paramètres nommés et `required` (facile)

Écrivez une classe `Player` avec `name` (`final String`), `health` (`int`), `score` (`int`).
Le constructeur utilise des **paramètres nommés** : `name` et `health` sont `required`, `score` vaut `0` par défaut.
Créez deux joueurs, l'un avec un score, l'autre sans, et affichez leur statut.

### Exercice 4 — Valeurs par défaut (facile)

Écrivez une classe `Product` (commerce en ligne) avec `reference`, `label`, `price` et `stock`.
`reference`, `label` et `price` sont obligatoires. `stock` vaut `0` par défaut.
Créez trois produits et affichez pour chacun `[reference] label - prix EUR - stock: n`, le prix étant affiché avec deux décimales.

### Exercice 5 — Constructeurs nommés (moyen)

Écrivez une classe `Enemy` avec `name`, `health`, `damage`, `isBoss`.
Ajoutez un constructeur principal nommé, puis trois constructeurs nommés **redirigés** :
`Enemy.gobelin()` (30 PV, 5 dégâts), `Enemy.orc()` (60 PV, 12 dégâts), `Enemy.boss(String nom)` (400 PV, 45 dégâts, `isBoss` à `true`).
Créez une vague de quatre ennemis dans une `List<Enemy>` et affichez-la.

### Exercice 6 — Initializer list (moyen)

Écrivez une classe `Weapon` avec `name` (`final String`), `damage` (`final int`), `maxDurability` (`final int`) et `repairCost` (`final int`).
`repairCost` n'est pas fourni par l'appelant : il vaut `damage * 5`.
`maxDurability` n'est pas fourni non plus : il vaut `100 - damage` (une arme puissante s'use plus vite).
Créez deux armes et affichez leurs caractéristiques.

### Exercice 7 — `assert` (moyen)

Reprenez la classe `Product` de l'exercice 4 et ajoutez quatre `assert` dans l'initializer list :
la référence n'est pas vide, le libellé n'est pas vide, le prix est positif ou nul, le stock est positif ou nul.
Chaque `assert` porte un message explicite.
Créez un produit valide, puis mettez en commentaire une ligne créant un produit invalide en indiquant le message attendu.

### Exercice 8 — Constructeur `const` (moyen)

Écrivez une classe `Position` avec `x` et `y` (`final int`) et un constructeur `const`.
Ajoutez un getter `origine` qui indique si la position est `(0, 0)`, et une méthode `distanceVers(Position autre)` qui renvoie la distance en ligne droite (utilisez `sqrt` de `dart:math`).
Démontrez avec `identical()` que deux positions `const` identiques sont le même objet.

### Exercice 9 — Constructeur `factory` (moyen)

Écrivez une classe `Enemy` avec `name`, `health`, `damage`.
Ajoutez un `factory Enemy.selonNiveau(int niveau)` qui renvoie un gobelin en dessous du niveau 5, un orc de 5 à 9, un dragon à partir de 10.
Ajoutez un `factory Enemy.depuisMap(Map<String, Object> donnees)`.
Testez les deux fabriques.

### Exercice 10 — Getters calculés (moyen)

Écrivez une classe `Player` avec `name`, `maxHealth` et `health`.
Ajoutez les getters calculés suivants : `estVivant`, `estMort`, `pourcentageVie`, `estEnDanger` (moins de 30 %), et `etat` qui renvoie `'En pleine forme'`, `'Blessé'`, `'Critique'` ou `'Mort'`.
Faites subir des dégâts successifs au joueur et affichez son état à chaque étape.

### Exercice 11 — Setter contrôlé et `toString()` (moyen)

Écrivez une classe `Inventory` avec `owner` (`final String`), une capacité maximale `maxGold` (`final int`, 1000 par défaut) et une propriété privée `_gold`.
Exposez `gold` par un getter et un setter. Le setter borne la valeur entre `0` et `maxGold`.
Ajoutez `ajouterOr(int montant)`, `depenser(int montant)` (qui refuse si l'or est insuffisant) et un `toString()`.
Testez les cas limites.

### Exercice 12 — Mini-projet : catalogue d'armes (difficile)

Construisez un petit catalogue d'armes réunissant toutes les notions du chapitre.

**Classe `Weapon`**

- propriétés `final` : `reference`, `name`, `damage`, `priceHT`, `rarity` (`String`) ;
- constructeur nommé principal : `reference`, `name`, `damage`, `priceHT` obligatoires, `rarity` valant `'commune'` par défaut ;
- deux `assert` : dégâts strictement positifs, prix positif ou nul ;
- constructeurs nommés redirigés : `Weapon.epeeDeFer()`, `Weapon.arcLong()`, `Weapon.batonMagique()` ;
- un `factory Weapon.depuisMap(Map<String, Object> donnees)` ;
- getters calculés : `priceTTC` (TVA 20 %), `estRare` (rareté différente de `'commune'`), `puissance` (dégâts multipliés par 2 si l'arme est rare) ;
- `toString()`.

**Classe `Catalogue`**

- propriétés : `title` (`final String`) et `_weapons` (`List<Weapon>`) ;
- constructeur avec `title` obligatoire et liste vide par défaut ;
- getters : `nombre`, `estVide`, `valeurTotaleTTC`, `plusPuissante` ;
- méthodes : `ajouter(Weapon arme)`, `afficher()` ;
- `toString()`.

**Dans `main()`**

Créez un catalogue, ajoutez les trois armes prédéfinies, une arme personnalisée et une arme construite depuis une `Map`. Affichez le catalogue complet, la valeur totale et l'arme la plus puissante.

---

## 09.22 — Corrections des exercices

### Correction 1

```dart
class Potion {
  final String name;
  final int healAmount;

  Potion(String name, int healAmount)
      : name = name,
        healAmount = healAmount;
}

void main() {
  Potion soin = Potion('Potion de soin', 25);
  print('${soin.name} : +${soin.healAmount} PV');
}
```

**Résultat :**

```text
Potion de soin : +25 PV
```

**Explication :** les propriétés sont `final`. On ne peut donc pas les affecter dans le corps du constructeur : il faut passer par l'initializer list `: name = name, healAmount = healAmount`. Ici, `name` à gauche des deux-points désigne la propriété, `name` à droite désigne le paramètre. Aucune ambiguïté n'est possible dans une initializer list, `this` n'y étant pas accessible. Si les propriétés n'étaient pas `final`, on écrirait dans le corps `this.name = name;`.

---

### Correction 2

```dart
class Potion {
  final String name;
  final int healAmount;

  Potion(this.name, this.healAmount);

  void describe() {
    print('$name : +$healAmount PV');
  }
}

void main() {
  Potion petite = Potion('Petite potion', 20);
  Potion moyenne = Potion('Potion de soin', 50);
  Potion grande = Potion('Élixir majeur', 150);

  petite.describe();
  moyenne.describe();
  grande.describe();
}
```

**Résultat :**

```text
Petite potion : +20 PV
Potion de soin : +50 PV
Élixir majeur : +150 PV
```

**Explication :** `Potion(this.name, this.healAmount);` remplace à lui seul l'initializer list de la correction 1. Dart déduit le type de chaque paramètre depuis la propriété correspondante. Le corps étant vide, le constructeur se termine par un point-virgule : ni accolades, ni deux-points. C'est la forme à privilégier dans la quasi-totalité des cas.

---

### Correction 3

```dart
class Player {
  final String name;
  int health;
  int score;

  Player({required this.name, required this.health, this.score = 0});

  void showStatus() {
    print('$name | vie: $health | score: $score');
  }
}

void main() {
  Player alex = Player(name: 'Alex', health: 100, score: 1250);
  Player sophie = Player(name: 'Sophie', health: 80);

  alex.showStatus();
  sophie.showStatus();
}
```

**Résultat :**

```text
Alex | vie: 100 | score: 1250
Sophie | vie: 80 | score: 0
```

**Explication :** les accolades `{ }` transforment les paramètres en paramètres nommés. `required` rend `name` et `health` obligatoires : les omettre provoquerait une erreur de compilation. `score` reçoit une valeur par défaut, ce qui le rend facultatif. Notez qu'un paramètre nommé sans `required` et sans valeur par défaut serait refusé pour un type non nullable comme `int`.

---

### Correction 4

```dart
class Product {
  final String reference;
  final String label;
  final double price;
  int stock;

  Product({
    required this.reference,
    required this.label,
    required this.price,
    this.stock = 0,
  });

  void describe() {
    print('[$reference] $label - ${price.toStringAsFixed(2)} EUR - stock: $stock');
  }
}

void main() {
  Product clavier = Product(
    reference: 'KB-100',
    label: 'Clavier mécanique',
    price: 89.9,
    stock: 12,
  );
  Product souris = Product(
    reference: 'MS-200',
    label: 'Souris sans fil',
    price: 24.5,
    stock: 3,
  );
  Product ecran = Product(
    reference: 'SC-300',
    label: 'Écran 27 pouces',
    price: 249.0,
  );

  clavier.describe();
  souris.describe();
  ecran.describe();
}
```

**Résultat :**

```text
[KB-100] Clavier mécanique - 89.90 EUR - stock: 12
[MS-200] Souris sans fil - 24.50 EUR - stock: 3
[SC-300] Écran 27 pouces - 249.00 EUR - stock: 0
```

**Explication :** `stock` est la seule propriété non `final`, car un stock varie. `toStringAsFixed(2)` force l'affichage à deux décimales : sans lui, `89.9` s'afficherait `89.9` et `249.0` s'afficherait `249.0`, ce qui n'est pas un format monétaire. Le troisième produit démontre l'intérêt de la valeur par défaut : aucune ligne inutile à écrire.

---

### Correction 5

```dart
class Enemy {
  final String name;
  int health;
  final int damage;
  final bool isBoss;

  Enemy({
    required this.name,
    required this.health,
    required this.damage,
    this.isBoss = false,
  });

  Enemy.gobelin() : this(name: 'Gobelin', health: 30, damage: 5);

  Enemy.orc() : this(name: 'Orc', health: 60, damage: 12);

  Enemy.boss(String nom)
      : this(name: nom, health: 400, damage: 45, isBoss: true);

  @override
  String toString() {
    String marque = isBoss ? '[BOSS] ' : '';
    return '$marque$name ($health PV, $damage degats)';
  }
}

void main() {
  List<Enemy> vague = [
    Enemy.gobelin(),
    Enemy.gobelin(),
    Enemy.orc(),
    Enemy.boss('Dragon Noir'),
  ];

  for (Enemy e in vague) {
    print(e);
  }
}
```

**Résultat :**

```text
Gobelin (30 PV, 5 degats)
Gobelin (30 PV, 5 degats)
Orc (60 PV, 12 degats)
[BOSS] Dragon Noir (400 PV, 45 degats)
```

**Explication :** chaque constructeur nommé **redirige** vers le constructeur principal grâce à `: this(...)`. Un constructeur redirigé n'a pas de corps : il se termine par un point-virgule. L'avantage est qu'il n'existe qu'un seul endroit où les propriétés sont réellement affectées. Si demain vous ajoutez une propriété `level`, il suffira de la traiter dans le constructeur principal.

---

### Correction 6

```dart
class Weapon {
  final String name;
  final int damage;
  final int maxDurability;
  final int repairCost;

  Weapon(this.name, int damage)
      : damage = damage,
        repairCost = damage * 5,
        maxDurability = 100 - damage;

  void describe() {
    print('$name : $damage degats | durabilite $maxDurability '
        '| reparation $repairCost pieces');
  }
}

void main() {
  Weapon epee = Weapon('Épée de fer', 12);
  Weapon hache = Weapon('Hache de guerre', 30);

  epee.describe();
  hache.describe();
}
```

**Résultat :**

```text
Épée de fer : 12 degats | durabilite 88 | reparation 60 pieces
Hache de guerre : 30 degats | durabilite 70 | reparation 150 pieces
```

**Explication :** on n'écrit pas `this.damage` en paramètre, car il faut réutiliser la valeur reçue dans deux autres calculs. On déclare donc un paramètre ordinaire `int damage`, puis on affecte les trois propriétés `final` dans l'initializer list. Rappel essentiel : dans l'initializer list, `damage` désigne le paramètre, jamais la propriété. Écrire `repairCost = this.damage * 5` provoquerait l'erreur « Can't access 'this' in a field initializer ».

---

### Correction 7

```dart
class Product {
  final String reference;
  final String label;
  final double price;
  int stock;

  Product({
    required this.reference,
    required this.label,
    required this.price,
    this.stock = 0,
  })  : assert(reference != '', 'La référence ne peut pas être vide'),
        assert(label != '', 'Le libellé ne peut pas être vide'),
        assert(price >= 0, 'Le prix ne peut pas être négatif'),
        assert(stock >= 0, 'Le stock ne peut pas être négatif');

  void describe() {
    print('[$reference] $label - ${price.toStringAsFixed(2)} EUR - stock: $stock');
  }
}

void main() {
  Product clavier = Product(
    reference: 'KB-100',
    label: 'Clavier mécanique',
    price: 89.9,
    stock: 12,
  );
  clavier.describe();

  // Product invalide = Product(reference: '', label: 'X', price: 10);
  // -> Assertion failed: "La référence ne peut pas être vide"

  // Product negatif = Product(reference: 'A', label: 'B', price: -1);
  // -> Assertion failed: "Le prix ne peut pas être négatif"
}
```

**Résultat :**

```text
[KB-100] Clavier mécanique - 89.90 EUR - stock: 12
```

**Explication :** lorsque le constructeur possède des paramètres nommés, la parenthèse fermante est suivie de `:` puis de la liste des `assert` séparés par des virgules. Chaque `assert` prend deux arguments : la condition à vérifier et le message affiché si elle est fausse. Rappelez-vous que les `assert` sont désactivés en mode release : ils servent à repérer une erreur de programmation pendant le développement, pas à valider la saisie d'un utilisateur.

---

### Correction 8

```dart
import 'dart:math';

class Position {
  final int x;
  final int y;

  const Position(this.x, this.y);

  bool get origine => x == 0 && y == 0;

  double distanceVers(Position autre) {
    int dx = autre.x - x;
    int dy = autre.y - y;
    return sqrt(dx * dx + dy * dy);
  }

  @override
  String toString() => '($x, $y)';
}

void main() {
  const Position depart = Position(0, 0);
  const Position arrivee = Position(3, 4);

  print('$depart origine ? ${depart.origine}');
  print('$arrivee origine ? ${arrivee.origine}');
  print('distance : ${depart.distanceVers(arrivee)}');

  const Position a = Position(3, 4);
  const Position b = Position(3, 4);
  Position c = Position(3, 4);

  print('a et b sont le même objet ? ${identical(a, b)}');
  print('a et c sont le même objet ? ${identical(a, c)}');
}
```

**Résultat :**

```text
(0, 0) origine ? true
(3, 4) origine ? false
distance : 5.0
a et b sont le même objet ? true
a et c sont le même objet ? false
```

**Explication :** le constructeur `const` exige que toutes les propriétés soient `final` et que le corps soit vide. Dart peut alors « canoniser » les objets : deux appels `const Position(3, 4)` produisent une seule et unique instance en mémoire, ce que confirme `identical(a, b)`. Sans le mot-clé `const` à l'appel (variable `c`), Dart crée un objet ordinaire, distinct. Notez aussi que `distanceVers` est une **méthode** et non un getter : elle prend un argument.

---

### Correction 9

```dart
class Enemy {
  final String name;
  final int health;
  final int damage;

  Enemy({required this.name, required this.health, required this.damage});

  factory Enemy.selonNiveau(int niveau) {
    if (niveau >= 10) {
      return Enemy(name: 'Dragon', health: 500, damage: 40);
    }
    if (niveau >= 5) {
      return Enemy(name: 'Orc', health: 60, damage: 12);
    }
    return Enemy(name: 'Gobelin', health: 30, damage: 5);
  }

  factory Enemy.depuisMap(Map<String, Object> donnees) {
    return Enemy(
      name: donnees['name'] as String,
      health: donnees['health'] as int,
      damage: donnees['damage'] as int,
    );
  }

  @override
  String toString() => 'Enemy($name, $health PV, $damage degats)';
}

void main() {
  print(Enemy.selonNiveau(2));
  print(Enemy.selonNiveau(7));
  print(Enemy.selonNiveau(14));

  Map<String, Object> ligne = {
    'name': 'Spectre',
    'health': 120,
    'damage': 25,
  };
  print(Enemy.depuisMap(ligne));
}
```

**Résultat :**

```text
Enemy(Gobelin, 30 PV, 5 degats)
Enemy(Orc, 60 PV, 12 degats)
Enemy(Dragon, 500 PV, 40 degats)
Enemy(Spectre, 120 PV, 25 degats)
```

**Explication :** un constructeur ordinaire ne peut pas contenir de `return`. Dès qu'il faut **décider** quel objet renvoyer, `factory` est obligatoire. `Enemy.selonNiveau` choisit selon une condition ; `Enemy.depuisMap` convertit une structure de données en objet. Le mot-clé `as` convertit la valeur générique `Object` vers le type attendu. Vous retrouverez très exactement ce second modèle au chapitre 17, consacré au JSON.

---

### Correction 10

```dart
class Player {
  final String name;
  final int maxHealth;
  int health;

  Player({required this.name, this.maxHealth = 100}) : health = maxHealth;

  bool get estVivant => health > 0;
  bool get estMort => !estVivant;
  double get pourcentageVie => health / maxHealth * 100;
  bool get estEnDanger => estVivant && pourcentageVie < 30;

  String get etat {
    if (estMort) return 'Mort';
    if (pourcentageVie < 30) return 'Critique';
    if (pourcentageVie < 70) return 'Blessé';
    return 'En pleine forme';
  }

  void subirDegats(int degats) {
    health = (health - degats).clamp(0, maxHealth);
    print('$name subit $degats degats -> $health PV ($etat)');
  }
}

void main() {
  Player alex = Player(name: 'Alex');
  print('${alex.name} : ${alex.health} PV (${alex.etat})');

  alex.subirDegats(25);
  alex.subirDegats(30);
  alex.subirDegats(30);
  alex.subirDegats(50);

  print('Vivant ? ${alex.estVivant} | En danger ? ${alex.estEnDanger}');
}
```

**Résultat :**

```text
Alex : 100 PV (En pleine forme)
Alex subit 25 degats -> 75 PV (En pleine forme)
Alex subit 30 degats -> 45 PV (Blessé)
Alex subit 30 degats -> 15 PV (Critique)
Alex subit 50 degats -> 0 PV (Mort)
Vivant ? false | En danger ? false
```

**Explication :** aucun de ces cinq getters ne stocke la moindre donnée : ils sont recalculés à chaque lecture, ce qui garantit qu'ils sont toujours cohérents avec `health`. Notez que `estEnDanger` combine deux autres getters, et que `etat` en utilise deux également : un getter peut librement en appeler d'autres. `clamp(0, maxHealth)` empêche la vie de descendre en dessous de zéro.

---

### Correction 11

```dart
class Inventory {
  final String owner;
  final int maxGold;
  int _gold;

  Inventory({required this.owner, this.maxGold = 1000, int gold = 0})
      : _gold = gold.clamp(0, maxGold),
        assert(maxGold > 0, 'La capacité doit être positive');

  int get gold => _gold;

  set gold(int valeur) {
    _gold = valeur.clamp(0, maxGold);
  }

  bool get estPlein => _gold >= maxGold;

  void ajouterOr(int montant) {
    gold = _gold + montant;
    print('$owner reçoit $montant or -> $_gold');
  }

  bool depenser(int montant) {
    if (montant > _gold) {
      print('$owner : fonds insuffisants ($montant demandés, $_gold disponibles)');
      return false;
    }
    gold = _gold - montant;
    print('$owner dépense $montant or -> $_gold');
    return true;
  }

  @override
  String toString() => 'Inventory($owner, $_gold/$maxGold or)';
}

void main() {
  Inventory sac = Inventory(owner: 'Alex', maxGold: 500, gold: 100);
  print(sac);

  sac.ajouterOr(250);
  sac.ajouterOr(1000);
  print('Plein ? ${sac.estPlein}');

  sac.depenser(200);
  sac.depenser(9999);

  sac.gold = -50;
  print(sac);
}
```

**Résultat :**

```text
Inventory(Alex, 100/500 or)
Alex reçoit 250 or -> 350
Alex reçoit 1000 or -> 500
Plein ? true
Alex dépense 200 or -> 300
Alex : fonds insuffisants (9999 demandés, 300 disponibles)
Inventory(Alex, 0/500 or)
```

**Explication :** la vraie donnée est `_gold`, cachée derrière le tiret bas. Toute écriture passe par le setter `gold`, qui borne systématiquement la valeur avec `clamp(0, maxGold)`. Les méthodes `ajouterOr` et `depenser` écrivent volontairement via `gold = ...` et non via `_gold = ...` : elles héritent ainsi du bornage sans le réécrire. Notez enfin que l'initializer list applique le même `clamp` à la valeur initiale, afin qu'un objet ne puisse jamais naître dans un état invalide.

---

### Correction 12 — Mini-projet : catalogue d'armes

```dart
class Weapon {
  final String reference;
  final String name;
  final int damage;
  final double priceHT;
  final String rarity;

  Weapon({
    required this.reference,
    required this.name,
    required this.damage,
    required this.priceHT,
    this.rarity = 'commune',
  })  : assert(damage > 0, 'Les dégâts doivent être strictement positifs'),
        assert(priceHT >= 0, 'Le prix ne peut pas être négatif');

  Weapon.epeeDeFer()
      : this(
          reference: 'W-001',
          name: 'Épée de fer',
          damage: 12,
          priceHT: 150.0,
        );

  Weapon.arcLong()
      : this(
          reference: 'W-002',
          name: 'Arc long',
          damage: 18,
          priceHT: 220.0,
          rarity: 'rare',
        );

  Weapon.batonMagique()
      : this(
          reference: 'W-003',
          name: 'Bâton magique',
          damage: 25,
          priceHT: 480.0,
          rarity: 'legendaire',
        );

  factory Weapon.depuisMap(Map<String, Object> donnees) {
    return Weapon(
      reference: donnees['reference'] as String,
      name: donnees['name'] as String,
      damage: donnees['damage'] as int,
      priceHT: donnees['priceHT'] as double,
      rarity: donnees['rarity'] as String,
    );
  }

  double get priceTTC => priceHT * 1.20;
  bool get estRare => rarity != 'commune';
  int get puissance => estRare ? damage * 2 : damage;

  @override
  String toString() =>
      '[$reference] $name | $rarity | $damage degats (puissance $puissance) '
      '| ${priceTTC.toStringAsFixed(2)} EUR TTC';
}

class Catalogue {
  final String title;
  final List<Weapon> _weapons;

  Catalogue({required this.title}) : _weapons = [];

  int get nombre => _weapons.length;
  bool get estVide => _weapons.isEmpty;

  double get valeurTotaleTTC {
    double total = 0;
    for (Weapon w in _weapons) {
      total = total + w.priceTTC;
    }
    return total;
  }

  Weapon get plusPuissante {
    Weapon meilleure = _weapons[0];
    for (Weapon w in _weapons) {
      if (w.puissance > meilleure.puissance) {
        meilleure = w;
      }
    }
    return meilleure;
  }

  void ajouter(Weapon arme) {
    _weapons.add(arme);
  }

  void afficher() {
    print('=== $title ($nombre armes) ===');
    for (Weapon w in _weapons) {
      print('  $w');
    }
  }

  @override
  String toString() =>
      'Catalogue($title, $nombre armes, ${valeurTotaleTTC.toStringAsFixed(2)} EUR)';
}

void main() {
  Catalogue boutique = Catalogue(title: 'Armurerie du village');

  boutique.ajouter(Weapon.epeeDeFer());
  boutique.ajouter(Weapon.arcLong());
  boutique.ajouter(Weapon.batonMagique());

  boutique.ajouter(Weapon(
    reference: 'W-004',
    name: 'Dague rouillée',
    damage: 4,
    priceHT: 15.0,
  ));

  Map<String, Object> ligne = {
    'reference': 'W-005',
    'name': 'Marteau du titan',
    'damage': 30,
    'priceHT': 900.0,
    'rarity': 'legendaire',
  };
  boutique.ajouter(Weapon.depuisMap(ligne));

  boutique.afficher();

  print('');
  print('Valeur totale : ${boutique.valeurTotaleTTC.toStringAsFixed(2)} EUR TTC');
  print('Arme la plus puissante : ${boutique.plusPuissante.name} '
      '(${boutique.plusPuissante.puissance})');
  print(boutique);
}
```

**Résultat :**

```text
=== Armurerie du village (5 armes) ===
  [W-001] Épée de fer | commune | 12 degats (puissance 12) | 180.00 EUR TTC
  [W-002] Arc long | rare | 18 degats (puissance 36) | 264.00 EUR TTC
  [W-003] Bâton magique | legendaire | 25 degats (puissance 50) | 576.00 EUR TTC
  [W-004] Dague rouillée | commune | 4 degats (puissance 4) | 18.00 EUR TTC
  [W-005] Marteau du titan | legendaire | 30 degats (puissance 60) | 1080.00 EUR TTC

Valeur totale : 2118.00 EUR TTC
Arme la plus puissante : Marteau du titan (60)
Catalogue(Armurerie du village, 5 armes, 2118.00 EUR)
```

**Explication :** ce mini-projet réunit l'ensemble du chapitre.

- `Weapon` possède un constructeur principal à paramètres nommés, protégé par deux `assert` ;
- les trois constructeurs nommés redirigent vers ce constructeur principal, ce qui évite toute duplication ;
- le `factory Weapon.depuisMap` est indispensable, car il faut écrire `return` ;
- `priceTTC`, `estRare` et `puissance` sont des getters calculés : aucune de ces valeurs n'est stockée, elles restent donc toujours cohérentes ;
- `Catalogue` illustre la **composition** : un objet contient une liste d'autres objets. Sa liste `_weapons` est créée par l'initializer list `: _weapons = []`, car une liste modifiable ne peut pas servir de valeur par défaut de paramètre ;
- `valeurTotaleTTC` et `plusPuissante` sont des getters qui parcourent la liste. Attention : `plusPuissante` suppose le catalogue non vide. Une version robuste vérifierait `estVide` et lèverait une exception, notion du chapitre 13 ;
- chaque classe redéfinit `toString()`, ce qui rend l'affichage immédiatement lisible, y compris à l'intérieur d'une interpolation.

---

## Et maintenant ?

Vous savez désormais faire naître un objet complet, valide et immuable là où c'est nécessaire. Vos classes ne sont plus des sacs de propriétés que l'on remplit après coup : elles imposent leurs règles dès la construction, exposent des valeurs calculées par des getters et se décrivent elles-mêmes avec `toString()`.

Il reste cependant un point faible. Rien n'empêche encore le code extérieur de tripoter directement les propriétés de vos objets, et surtout : nos classes sont toutes indépendantes. `Player`, `Enemy` et un futur `Npc` partagent pourtant beaucoup de choses (un nom, des points de vie, la capacité de subir des dégâts), et nous dupliquons ce code dans chacune d'elles.

Le chapitre suivant apporte les trois piliers qui règlent ces deux problèmes :

- l'**encapsulation**, pour protéger réellement l'état interne d'un objet avec le tiret bas `_` ;
- l'**héritage**, avec `extends` et `super`, pour écrire une fois ce qui est commun ;
- le **polymorphisme**, pour traiter uniformément des objets de types différents.

Rendez-vous au chapitre 10 : [10-PARTIE-1A—ENCAPSULATION-HÉRITAGE-POLYMORPHISME.md](10-PARTIE-1A—ENCAPSULATION-HÉRITAGE-POLYMORPHISME.md)
