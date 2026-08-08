# PARTIE 1A — DART
# CHAPITRE 10 — ENCAPSULATION, HÉRITAGE ET POLYMORPHISME

> **Niveau :** intermédiaire
> **Durée estimée :** 7 h
> **Pré-requis :** chapitre 09 — Constructeurs et modélisation
> **Ce que vous saurez faire à la fin :** protéger les données d'une classe avec des membres privés, des getters et des setters validés, construire une hiérarchie de classes avec `extends` et `super`, redéfinir des méthodes avec `@override`, et écrire du code polymorphe qui traite `Player`, `Enemy` et `Boss` de la même façon.

---

## 10.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- citer les trois piliers de la POO abordés ici ;
- expliquer ce qu'est l'encapsulation avec vos propres mots ;
- montrer concrètement le danger d'une propriété publique modifiable n'importe comment ;
- déclarer un membre privé avec le préfixe `_` ;
- expliquer que la portée du `_` est la bibliothèque (le fichier), pas la classe ;
- écrire un getter qui expose une valeur en lecture seule ;
- écrire un setter qui valide la valeur avant de l'accepter ;
- écrire des méthodes privées d'aide interne ;
- construire une classe `Player` entièrement encapsulée, avec des vies toujours bornées entre 0 et un maximum ;
- expliquer ce qu'est l'héritage et à quoi il sert ;
- utiliser `extends` pour créer une sous-classe ;
- dire précisément ce qui est hérité et ce qui ne l'est pas ;
- appeler le constructeur parent avec `super` ;
- appeler une méthode parente avec `super.methode()` ;
- redéfinir (override) une méthode héritée ;
- utiliser correctement l'annotation `@override` ;
- expliquer le polymorphisme sur un exemple ;
- parcourir une `List<Character>` contenant des `Player` et des `Enemy` ;
- utiliser `is` et `as` pour tester et convertir un type ;
- construire la hiérarchie `Character → Player / Enemy / Boss` ;
- reconnaître les situations où l'héritage est un mauvais choix.

---

## 10.1 — Les trois piliers abordés dans ce chapitre

La programmation orientée objet repose sur quelques grands principes. Ce chapitre en traite trois.

```text
+--------------------+-------------------------------------------+
| ENCAPSULATION      | Cacher l'intérieur, exposer une façade    |
+--------------------+-------------------------------------------+
| HÉRITAGE           | Réutiliser une classe pour en créer une   |
|                    | version plus spécialisée                  |
+--------------------+-------------------------------------------+
| POLYMORPHISME      | Traiter des objets différents avec le     |
|                    | même code                                 |
+--------------------+-------------------------------------------+
```

Reformulons chacun avec le vocabulaire du jeu vidéo.

**Encapsulation.** Le nombre de vies du joueur ne doit pas pouvoir passer à `-15` parce qu'une ligne de code ailleurs a écrit n'importe quoi. La classe `Player` protège ses propres données.

**Héritage.** Un `Player`, un `Enemy` et un `Boss` ont tous un nom et des points de vie. Plutôt que de réécrire trois fois le même code, nous écrivons une classe `Character` et les trois autres en héritent.

**Polymorphisme.** Nous voulons écrire une seule boucle qui affiche l'état de tous les personnages du niveau, sans nous demander à chaque tour s'il s'agit d'un joueur, d'un ennemi ou d'un boss.

Ces trois notions se complètent. Nous les verrons dans cet ordre, car l'encapsulation est la plus simple et sert de fondation aux deux autres.

---

## 10.2 — Qu'est-ce que l'encapsulation ?

Encapsuler, c'est mettre les données dans une capsule et ne laisser passer que ce qui doit passer.

Une classe encapsulée ressemble à une machine à café : vous voyez des boutons, vous ne voyez pas la tuyauterie.

```text
        Le monde extérieur (votre main())
                     |
                     v
   +---------------------------------------+
   |            FAÇADE PUBLIQUE            |
   |   name      hp      takeDamage()      |
   +---------------------------------------+
   |                                       |
   |   INTÉRIEUR PRIVÉ (protégé)           |
   |     _hp = 100                         |
   |     _maxHp = 100                      |
   |     _clamp()                          |
   |                                       |
   +---------------------------------------+
              classe Player
```

Le monde extérieur ne touche jamais directement `_hp`. Il passe par la façade.

L'intérêt est triple :

1. **La classe garantit ses propres règles.** Si `_hp` ne doit jamais dépasser `_maxHp`, c'est la classe qui l'impose, une fois pour toutes.
2. **Le code appelant devient plus simple.** Il n'a pas à connaître les règles internes.
3. **Vous pouvez changer l'intérieur sans casser l'extérieur.** Tant que la façade reste la même, le reste du programme continue de fonctionner.

En Dart, l'encapsulation repose sur quatre outils :

```text
1. le préfixe _   -> rendre un membre privé
2. les getters    -> exposer une lecture contrôlée
3. les setters    -> exposer une écriture contrôlée
4. les méthodes privées -> factoriser la logique interne
```

Nous allons voir les quatre.

---

## 10.3 — Le problème d'une propriété publique modifiable n'importe comment

Commençons par le problème. Voici une classe `Player` naïve, sans aucune protection.

```dart
class Player {
  String name;
  int hp;

  Player(this.name, this.hp);
}

void main() {
  Player hero = Player('Alex', 100);

  print('${hero.name} : ${hero.hp} PV');

  hero.hp = -50;
  print('${hero.name} : ${hero.hp} PV');

  hero.hp = 999999;
  print('${hero.name} : ${hero.hp} PV');
}
```

**Résultat :**

```text
Alex : 100 PV
Alex : -50 PV
Alex : 999999 PV
```

Ce programme ne plante pas. C'est justement le danger : il produit un état absurde en silence.

Un joueur avec `-50` PV devrait être mort. Un joueur avec `999999` PV est invincible. Aucune de ces deux situations n'était voulue.

Regardons où est l'erreur. Elle n'est pas dans la classe `Player` : la classe n'a rien fait de mal, elle n'a rien fait du tout. L'erreur est dans le `main()`. Mais dans un vrai projet, ce `main()` sera dispersé dans des dizaines de fichiers, écrits par plusieurs personnes, sur plusieurs mois.

Il y a deux stratégies possibles :

```text
Stratégie A : demander à tout le monde de faire attention.
Stratégie B : rendre l'erreur impossible.
```

La stratégie A ne fonctionne jamais. Ce chapitre enseigne la stratégie B.

Reformulons la règle métier que nous voulons imposer :

```text
0 <= hp <= maxHp
```

Cette règle s'appelle un **invariant** : une propriété qui doit rester vraie du début à la fin de la vie de l'objet. L'encapsulation sert à protéger les invariants.

---

## 10.4 — Membres privés avec `_` (portée : la bibliothèque, pas la classe)

En Dart, il n'existe pas de mot-clé `private`. La règle est purement typographique :

> Tout identifiant qui commence par un tiret bas `_` est privé.

Cela s'applique aux propriétés, aux méthodes, aux constructeurs nommés et même aux classes.

```dart
class Player {
  String name;
  int _hp;

  Player(this.name, this._hp);
}
```

Ici, `name` est public et `_hp` est privé.

Voici le point que presque tous les débutants comprennent de travers.

> En Dart, `_` rend le membre privé **à la bibliothèque**, c'est-à-dire au **fichier**, et non à la classe.

Concrètement, cela veut dire ceci.

```dart
class Player {
  String name;
  int _hp;

  Player(this.name, this._hp);
}

void main() {
  Player hero = Player('Alex', 100);

  print(hero._hp);
}
```

**Résultat :**

```text
100
```

Ce programme **compile et s'exécute** dans DartPad. Pourquoi ? Parce que `main()` et `Player` sont dans le même fichier, donc dans la même bibliothèque. Le `_` n'y change rien.

Schéma de la portée :

```text
fichier player.dart  (= une bibliothèque)
+--------------------------------------------+
|  class Player {                            |
|     int _hp;   <-- privé à CE fichier      |
|  }                                         |
|                                            |
|  void main() {                             |
|     hero._hp;   <-- AUTORISÉ (même fichier)|
|  }                                         |
+--------------------------------------------+

fichier game.dart  (= une autre bibliothèque)
+--------------------------------------------+
|  import 'player.dart';                     |
|                                            |
|  void main() {                             |
|     hero._hp;   <-- ERREUR de compilation  |
|  }                                         |
+--------------------------------------------+
```

Dans un vrai projet Dart ou Flutter, chaque classe vit dans son propre fichier. Le `_` protège donc réellement. Dans DartPad, tout est dans un seul fichier, et vous pouvez « tricher » sans que le compilateur proteste.

**Conséquence pratique pour ce chapitre :** nous écrirons le code comme s'il était protégé, et nous n'accèderons jamais directement à un membre `_` depuis `main()`. C'est une discipline, et c'est exactement la discipline qui vous servira en Flutter.

Deuxième conséquence, plus surprenante : deux classes écrites dans le **même** fichier voient mutuellement leurs membres privés.

```dart
class Weapon {
  int _damage;
  Weapon(this._damage);
}

class Player {
  String name;
  Player(this.name);

  void inspect(Weapon w) {
    print('$name observe une arme de ${w._damage} dégâts.');
  }
}

void main() {
  Player hero = Player('Alex');
  Weapon sword = Weapon(25);

  hero.inspect(sword);
}
```

**Résultat :**

```text
Alex observe une arme de 25 dégâts.
```

`Player` lit `_damage` qui appartient à `Weapon`. C'est légal, car même fichier. Retenez la formule :

```text
_  =  privé au fichier
```

---

## 10.5 — Getters sécurisés

Si `_hp` est privé, comment le monde extérieur peut-il l'afficher ? Avec un **getter**.

Un getter est une méthode déguisée en propriété. On l'écrit avec le mot-clé `get`.

```dart
class Player {
  final String name;
  int _hp;

  Player(this.name, this._hp);

  int get hp {
    return _hp;
  }
}

void main() {
  Player hero = Player('Alex', 100);

  print('${hero.name} a ${hero.hp} PV.');
}
```

**Résultat :**

```text
Alex a 100 PV.
```

Notez bien la syntaxe d'appel : `hero.hp`, sans parenthèses. Pour celui qui utilise la classe, cela ressemble à une propriété normale.

Comme le corps tient sur une ligne, on écrit habituellement le getter en forme abrégée avec `=>` :

```dart
int get hp => _hp;
```

Un getter n'est pas obligé de renvoyer une propriété existante. Il peut **calculer** une valeur.

```dart
class Player {
  final String name;
  int _hp;
  final int maxHp;

  Player(this.name, this._hp, this.maxHp);

  int get hp => _hp;

  bool get isAlive => _hp > 0;

  double get hpPercent => (_hp / maxHp) * 100;

  String get healthBar {
    int filled = (_hp * 10) ~/ maxHp;
    return '[${'#' * filled}${'.' * (10 - filled)}]';
  }
}

void main() {
  Player hero = Player('Alex', 70, 100);

  print(hero.hp);
  print(hero.isAlive);
  print(hero.hpPercent);
  print(hero.healthBar);
}
```

**Résultat :**

```text
70
true
70.0
[#######...]
```

Trois remarques importantes.

1. `isAlive` et `hpPercent` ne sont stockés nulle part. Ils sont recalculés à chaque lecture. Il est donc impossible qu'ils soient désynchronisés des PV.
2. Un getter seul rend la propriété **en lecture seule** vue de l'extérieur. `hero.hp = 5;` provoque une erreur de compilation, car aucun setter `hp` n'existe.
3. Un getter ne doit jamais modifier l'objet. Lire une valeur ne doit pas avoir d'effet de bord.

Vérifions le point 2 par un contre-exemple à ne pas écrire :

```text
hero.hp = 5;
```

Message du compilateur :

```text
Error: The setter 'hp' isn't defined for the class 'Player'.
```

C'est exactement le comportement recherché : le compilateur refuse la modification sauvage vue en 10.3.

---

## 10.6 — Setters avec validation

Parfois, nous voulons quand même autoriser l'écriture, mais à nos conditions. C'est le rôle du **setter**.

Un setter s'écrit avec `set`, prend exactement un paramètre et ne renvoie rien.

```dart
class Player {
  final String name;
  final int maxHp;
  int _hp;

  Player(this.name, this._hp, this.maxHp);

  int get hp => _hp;

  set hp(int value) {
    if (value < 0) {
      _hp = 0;
    } else if (value > maxHp) {
      _hp = maxHp;
    } else {
      _hp = value;
    }
  }
}

void main() {
  Player hero = Player('Alex', 100, 100);

  hero.hp = 60;
  print(hero.hp);

  hero.hp = -50;
  print(hero.hp);

  hero.hp = 999999;
  print(hero.hp);
}
```

**Résultat :**

```text
60
0
100
```

Comparez avec la sortie de la section 10.3 :

```text
Sans encapsulation :  100  ->  -50    ->  999999
Avec encapsulation :   60  ->    0    ->     100
```

Le code appelant est identique dans sa forme (`hero.hp = ...`), mais l'objet ne peut plus entrer dans un état invalide. La règle `0 <= hp <= maxHp` est désormais garantie par la classe elle-même.

Dart possède une fonction utilitaire qui fait exactement ce bornage : `clamp()`. Elle existe sur les nombres.

```dart
class Player {
  final String name;
  final int maxHp;
  int _hp;

  Player(this.name, this._hp, this.maxHp);

  int get hp => _hp;

  set hp(int value) {
    _hp = value.clamp(0, maxHp);
  }
}

void main() {
  Player hero = Player('Alex', 100, 100);

  hero.hp = -50;
  print(hero.hp);

  hero.hp = 300;
  print(hero.hp);
}
```

**Résultat :**

```text
0
100
```

Attention à un piège classique : ne jamais écrire un setter qui s'appelle lui-même.

```dart
set hp(int value) {
  hp = value;
}
```

Cette ligne ne s'affecte pas à `_hp` mais au setter `hp`, donc à elle-même. Le programme part en boucle infinie et se termine par un débordement de pile (`Stack Overflow`). Le setter doit toujours écrire dans le champ privé `_hp`.

Enfin, il est fréquent qu'un setter **refuse** simplement une valeur au lieu de la corriger :

```dart
class Player {
  final String name;
  String _weapon;

  Player(this.name, this._weapon);

  String get weapon => _weapon;

  set weapon(String value) {
    if (value.trim().isEmpty) {
      print('Nom d\'arme invalide, changement ignoré.');
      return;
    }
    _weapon = value;
  }
}

void main() {
  Player hero = Player('Alex', 'Épée courte');

  hero.weapon = 'Hache de guerre';
  print(hero.weapon);

  hero.weapon = '   ';
  print(hero.weapon);
}
```

**Résultat :**

```text
Hache de guerre
Nom d'arme invalide, changement ignoré.
Hache de guerre
```

---

## 10.7 — Méthodes privées

Une méthode aussi peut être privée. Il suffit de préfixer son nom par `_`.

Une méthode privée sert à découper la logique interne sans l'exposer. C'est de la plomberie : utile, mais on ne veut pas que le reste du programme s'en serve.

```dart
class Player {
  final String name;
  final int maxHp;
  int _hp;

  Player(this.name, this._hp, this.maxHp);

  int get hp => _hp;
  bool get isAlive => _hp > 0;

  void _setHp(int value) {
    _hp = value.clamp(0, maxHp);
  }

  void _log(String message) {
    print('[$name] $message');
  }

  void takeDamage(int amount) {
    if (amount <= 0) {
      _log('Coup sans effet.');
      return;
    }
    _setHp(_hp - amount);
    _log('Subit $amount dégâts. PV : $_hp/$maxHp');
    if (!isAlive) {
      _log('Est hors de combat.');
    }
  }

  void heal(int amount) {
    if (!isAlive) {
      _log('Impossible de soigner un personnage à terre.');
      return;
    }
    _setHp(_hp + amount);
    _log('Récupère $amount PV. PV : $_hp/$maxHp');
  }
}

void main() {
  Player hero = Player('Alex', 100, 100);

  hero.takeDamage(30);
  hero.heal(50);
  hero.takeDamage(0);
  hero.takeDamage(200);
  hero.heal(10);
}
```

**Résultat :**

```text
[Alex] Subit 30 dégâts. PV : 70/100
[Alex] Récupère 50 PV. PV : 100/100
[Alex] Coup sans effet.
[Alex] Subit 200 dégâts. PV : 0/100
[Alex] Est hors de combat.
[Alex] Impossible de soigner un personnage à terre.
```

Observez ce qui se passe :

- `_setHp()` centralise le bornage. Il n'y a qu'un seul endroit à corriger si la règle change.
- `_log()` centralise l'affichage. Demain, nous pourrons remplacer `print` par une écriture dans un fichier sans toucher au reste.
- `takeDamage()` et `heal()` sont publiques : ce sont les actions que le jeu a le droit de déclencher.

La façade publique de cette classe est donc :

```text
name        (lecture)
hp          (lecture)
isAlive     (lecture)
takeDamage()
heal()
```

Tout le reste est interne. C'est cela, encapsuler.

---

## 10.8 — Exemple complet `Player` encapsulé

Rassemblons tout ce que nous venons de voir dans une classe complète et réaliste. Les vies sont bornées entre `0` et `maxHp`, et le score ne peut jamais devenir négatif.

```dart
class Player {
  final String name;
  final int maxHp;

  int _hp;
  int _score;
  int _potions;

  Player(this.name, {int maxHealth = 100, int potions = 3})
      : maxHp = maxHealth,
        _hp = maxHealth,
        _score = 0,
        _potions = potions;

  int get hp => _hp;
  int get score => _score;
  int get potions => _potions;

  bool get isAlive => _hp > 0;
  bool get isFullHp => _hp == maxHp;

  String get status => '$name  $_hp/$maxHp PV  |  score $_score  |  $_potions potion(s)';

  void _setHp(int value) {
    _hp = value.clamp(0, maxHp);
  }

  void takeDamage(int amount) {
    if (!isAlive) {
      return;
    }
    if (amount <= 0) {
      return;
    }
    _setHp(_hp - amount);
  }

  void heal(int amount) {
    if (!isAlive) {
      return;
    }
    if (amount <= 0) {
      return;
    }
    _setHp(_hp + amount);
  }

  void drinkPotion() {
    if (_potions <= 0) {
      print('$name n\'a plus de potion.');
      return;
    }
    if (isFullHp) {
      print('$name est déjà au maximum.');
      return;
    }
    _potions--;
    heal(40);
    print('$name boit une potion.');
  }

  void addScore(int points) {
    if (points <= 0) {
      return;
    }
    _score += points;
  }

  void loseScore(int points) {
    if (points <= 0) {
      return;
    }
    _score = _score - points;
    if (_score < 0) {
      _score = 0;
    }
  }
}

void main() {
  Player hero = Player('Alex', maxHealth: 120, potions: 2);

  print(hero.status);

  hero.takeDamage(45);
  hero.addScore(150);
  print(hero.status);

  hero.drinkPotion();
  print(hero.status);

  hero.loseScore(500);
  print(hero.status);

  hero.takeDamage(1000);
  print(hero.status);
  print('Vivant : ${hero.isAlive}');

  hero.heal(50);
  print(hero.status);
}
```

**Résultat :**

```text
Alex  120/120 PV  |  score 0  |  2 potion(s)
Alex  75/120 PV  |  score 150  |  2 potion(s)
Alex boit une potion.
Alex  115/120 PV  |  score 150  |  1 potion(s)
Alex  115/120 PV  |  score 0  |  1 potion(s)
Alex  0/120 PV  |  score 0  |  1 potion(s)
Vivant : false
Alex  0/120 PV  |  score 0  |  1 potion(s)
```

Vérifions ligne par ligne que les invariants tiennent :

| Action | Attendu | Obtenu |
| --- | --- | --- |
| `takeDamage(45)` sur 120 | 75 | 75 |
| `drinkPotion()` (+40 sur 75) | 115, borné à 120 | 115 |
| `loseScore(500)` sur 150 | 0, jamais négatif | 0 |
| `takeDamage(1000)` | 0, jamais négatif | 0 |
| `heal(50)` sur un mort | refusé | 0 |

Aucun appel extérieur ne peut casser ces règles, car aucun appel extérieur n'atteint `_hp`, `_score` ni `_potions`.

Un dernier point de vocabulaire. Dans le constructeur, la ligne :

```text
: maxHp = maxHealth, _hp = maxHealth, _score = 0, _potions = potions
```

s'appelle une **liste d'initialisation**. Elle s'exécute avant le corps du constructeur et sert à donner leur valeur aux champs `final` et privés. Vous l'avez rencontrée au chapitre 09.

---

## 10.9 — Qu'est-ce que l'héritage ?

Passons au deuxième pilier.

Imaginons trois classes de notre jeu, écrites sans héritage.

```dart
class Player {
  String name;
  int hp;
  Player(this.name, this.hp);
  void describe() => print('$name : $hp PV');
}

class Enemy {
  String name;
  int hp;
  Enemy(this.name, this.hp);
  void describe() => print('$name : $hp PV');
}

class Boss {
  String name;
  int hp;
  Boss(this.name, this.hp);
  void describe() => print('$name : $hp PV');
}
```

Le même code est écrit trois fois. Si nous ajoutons un champ `level`, il faut le faire trois fois. Si nous corrigeons un bug dans `describe()`, il faut le corriger trois fois. Tôt ou tard, une des trois copies sera oubliée.

L'héritage résout ce problème. Nous écrivons une classe qui contient le tronc commun, puis nous fabriquons les trois autres à partir d'elle.

```text
                  +-----------------+
                  |    Character    |   <- classe mère (parent, super-classe)
                  |  name           |
                  |  hp             |
                  |  describe()     |
                  +-----------------+
                          ^
                          |
        +-----------------+-----------------+
        |                 |                 |
  +-----------+     +-----------+     +-----------+
  |  Player   |     |   Enemy   |     |   Boss    |   <- classes filles
  |  score    |     |  reward   |     |  phase    |   (sous-classes)
  +-----------+     +-----------+     +-----------+
```

La flèche se lit toujours de bas en haut :

```text
Player  EST UN  Character
Enemy   EST UN  Character
Boss    EST UN  Character
```

Ce test « EST UN » est la question à se poser avant d'utiliser l'héritage. Nous y reviendrons en 10.20.

Le vocabulaire, en français et en anglais :

| Français | Anglais | Rôle |
| --- | --- | --- |
| classe mère, classe parente | superclass, parent class | celle qui donne |
| classe fille, sous-classe | subclass, child class | celle qui reçoit |
| hériter | inherit | recevoir les membres du parent |
| redéfinir | override | remplacer le comportement d'une méthode héritée |

---

## 10.10 — `extends`

Le mot-clé qui crée un lien d'héritage est `extends`.

```text
class Fille extends Mere { ... }
```

Voici le plus petit exemple complet possible.

```dart
class Character {
  String name = 'Sans nom';
  int hp = 100;

  void describe() {
    print('$name : $hp PV');
  }
}

class Player extends Character {
}

void main() {
  Player hero = Player();

  hero.name = 'Alex';
  hero.describe();
}
```

**Résultat :**

```text
Alex : 100 PV
```

Regardez bien la classe `Player` : elle est **vide**. Elle ne déclare ni `name`, ni `hp`, ni `describe()`. Et pourtant les trois fonctionnent, parce qu'ils viennent de `Character`.

Ajoutons maintenant du contenu propre à `Player`.

```dart
class Character {
  String name = 'Sans nom';
  int hp = 100;

  void describe() {
    print('$name : $hp PV');
  }
}

class Player extends Character {
  int score = 0;

  void addScore(int points) {
    score += points;
  }
}

void main() {
  Player hero = Player();
  hero.name = 'Alex';

  hero.addScore(120);
  hero.addScore(30);

  hero.describe();
  print('Score : ${hero.score}');
}
```

**Résultat :**

```text
Alex : 100 PV
Score : 150
```

`Player` possède donc, au total :

```text
name        (hérité de Character)
hp          (hérité de Character)
describe()  (hérité de Character)
score       (propre à Player)
addScore()  (propre à Player)
```

Une sous-classe est toujours **plus riche** que sa classe mère, jamais plus pauvre. On ne peut pas « retirer » un membre par héritage.

Dernier point : en Dart, une classe ne peut hériter que d'**une seule** classe. `extends` n'apparaît qu'une fois.

```text
class Boss extends Enemy extends Character  // INTERDIT
class Boss extends Enemy                    // correct
```

Mais les chaînes sont autorisées : `Boss extends Enemy`, et `Enemy extends Character`. Nous construirons cette chaîne en 10.19.

---

## 10.11 — Ce qui est hérité

Il faut savoir exactement ce qui passe du parent à l'enfant.

**Est hérité :**

| Élément | Hérité ? |
| --- | --- |
| Propriétés d'instance (`name`, `hp`) | oui |
| Méthodes d'instance (`describe()`) | oui |
| Getters (`get isAlive`) | oui |
| Setters (`set hp`) | oui |

**N'est pas hérité :**

| Élément | Hérité ? | Explication |
| --- | --- | --- |
| Les constructeurs | non | chaque classe écrit les siens |
| Les membres privés vus d'une autre bibliothèque | non accessibles | le `_` reste limité au fichier du parent |

Le point le plus important est le premier : **les constructeurs ne s'héritent pas**.

```dart
class Character {
  String name;
  int hp;

  Character(this.name, this.hp);
}

class Player extends Character {
}
```

Ce code **ne compile pas**. Message :

```text
Error: The superclass 'Character' doesn't have a zero argument constructor.
```

Dart raisonne ainsi : pour construire un `Player`, il faut d'abord construire la partie `Character`. Comme `Character` n'a pas de constructeur sans paramètre, `Player` doit dire explicitement comment appeler celui du parent. C'est le rôle de `super`, que nous voyons tout de suite.

Vérifions aussi qu'un getter est bien hérité :

```dart
class Character {
  String name = 'Sans nom';
  int hp = 100;

  bool get isAlive => hp > 0;
}

class Enemy extends Character {
}

void main() {
  Enemy slime = Enemy();
  print(slime.isAlive);

  slime.hp = 0;
  print(slime.isAlive);
}
```

**Résultat :**

```text
true
false
```

---

## 10.12 — `super` dans le constructeur

`super` désigne la partie parente de l'objet. Dans un constructeur, `super(...)` appelle le constructeur du parent.

Il s'écrit dans la **liste d'initialisation**, après les deux-points.

```dart
class Character {
  final String name;
  int hp;

  Character(this.name, this.hp) {
    print('  -> constructeur Character exécuté ($name)');
  }
}

class Player extends Character {
  int score;

  Player(String name, int hp, this.score) : super(name, hp) {
    print('  -> constructeur Player exécuté (score $score)');
  }
}

void main() {
  print('Création du héros :');
  Player hero = Player('Alex', 100, 250);

  print('${hero.name} / ${hero.hp} PV / ${hero.score} pts');
}
```

**Résultat :**

```text
Création du héros :
  -> constructeur Character exécuté (Alex)
  -> constructeur Player exécuté (score 250)
Alex / 100 PV / 250 pts
```

Retenez l'ordre d'exécution :

```text
1. la liste d'initialisation de la sous-classe  (this.score = ...)
2. le constructeur du parent                    (super(name, hp))
3. le corps du constructeur de la sous-classe   { ... }
```

Le parent est toujours construit **avant** le corps de l'enfant. C'est logique : on construit les fondations avant les murs.

Trois règles à connaître :

**Règle 1.** `super(...)` doit être le **dernier** élément de la liste d'initialisation.

```text
Player(String name, int hp, this.score) : super(name, hp), _bonus = 0;  // INTERDIT
Player(String name, int hp, this.score) : _bonus = 0, super(name, hp);  // correct
```

**Règle 2.** Si le parent possède un constructeur sans paramètre, l'appel à `super()` est ajouté automatiquement. On peut donc l'omettre.

**Règle 3.** On ne peut pas utiliser `this` avant l'appel à `super`. L'objet n'est pas encore complètement construit.

Dart propose depuis peu une écriture raccourcie, les **paramètres super**. Elle évite de recopier les paramètres un par un.

```dart
class Character {
  final String name;
  int hp;

  Character(this.name, this.hp);
}

class Player extends Character {
  int score;

  Player(super.name, super.hp, this.score);
}

void main() {
  Player hero = Player('Alex', 100, 250);
  print('${hero.name} / ${hero.hp} PV / ${hero.score} pts');
}
```

**Résultat :**

```text
Alex / 100 PV / 250 pts
```

`super.name` signifie « ce paramètre est transmis tel quel au paramètre `name` du constructeur parent ». Les deux écritures sont équivalentes. Dans ce chapitre, nous utiliserons souvent la forme longue `: super(...)`, plus explicite pour apprendre, mais vous rencontrerez la forme courte en Flutter.

---

## 10.13 — `super` pour appeler une méthode parente

`super` ne sert pas qu'au constructeur. Il permet aussi d'appeler une méthode du parent depuis l'enfant.

```dart
class Character {
  final String name;
  int hp;

  Character(this.name, this.hp);

  void describe() {
    print('$name : $hp PV');
  }
}

class Player extends Character {
  int score;

  Player(String name, int hp, this.score) : super(name, hp);

  void showSheet() {
    print('--- Fiche joueur ---');
    super.describe();
    print('Score : $score');
    print('--------------------');
  }
}

void main() {
  Player hero = Player('Alex', 100, 250);
  hero.showSheet();
}
```

**Résultat :**

```text
--- Fiche joueur ---
Alex : 100 PV
Score : 250
--------------------
```

Ici, `super.describe()` et `describe()` donneraient le même résultat, puisque `Player` ne redéfinit pas `describe()`.

`super.` devient **indispensable** dès que l'enfant redéfinit la méthode. C'est le seul moyen d'atteindre la version du parent, comme nous allons le voir.

Retenez la distinction :

```text
this.describe()   -> la version de MA classe (ou héritée)
super.describe()  -> la version de la classe MÈRE, toujours
```

---

## 10.14 — Redéfinir une méthode (override)

Redéfinir, c'est fournir dans la sous-classe une méthode qui porte **le même nom** et **la même signature** qu'une méthode du parent. La version de l'enfant remplace celle du parent.

```dart
class Character {
  final String name;
  int hp;

  Character(this.name, this.hp);

  void attack() {
    print('$name attaque maladroitement.');
  }
}

class Player extends Character {
  Player(String name, int hp) : super(name, hp);

  void attack() {
    print('$name frappe avec son épée !');
  }
}

class Enemy extends Character {
  Enemy(String name, int hp) : super(name, hp);

  void attack() {
    print('$name mord sauvagement.');
  }
}

void main() {
  Player hero = Player('Alex', 100);
  Enemy slime = Enemy('Slime', 30);
  Character ghost = Character('Fantôme', 10);

  hero.attack();
  slime.attack();
  ghost.attack();
}
```

**Résultat :**

```text
Alex frappe avec son épée !
Slime mord sauvagement.
Fantôme attaque maladroitement.
```

Trois objets, trois comportements, une seule méthode `attack()`.

Très souvent, on ne veut pas jeter le comportement du parent mais **l'enrichir**. On appelle alors `super` au début de la redéfinition.

```dart
class Character {
  final String name;
  int hp;

  Character(this.name, this.hp);

  void describe() {
    print('$name : $hp PV');
  }
}

class Boss extends Character {
  int phase;

  Boss(String name, int hp, this.phase) : super(name, hp);

  void describe() {
    super.describe();
    print('BOSS - phase $phase');
  }
}

void main() {
  Boss dragon = Boss('Dragon Noir', 500, 1);
  dragon.describe();
}
```

**Résultat :**

```text
Dragon Noir : 500 PV
BOSS - phase 1
```

La signature doit rester **compatible**. En pratique, pour un débutant : gardez exactement le même nom, le même nombre de paramètres, les mêmes types et le même type de retour.

Contre-exemple à ne pas écrire :

```text
class Character {
  void attack() { ... }
}

class Player extends Character {
  void attack(int power) { ... }   // signature différente
}
```

Message du compilateur :

```text
Error: The method 'Player.attack' has more required arguments than those of
overridden method 'Character.attack'.
```

Ce n'est pas une redéfinition, c'est une tentative de remplacement incompatible, et Dart la refuse.

---

## 10.15 — L'annotation `@override`

Écrivons la même redéfinition, cette fois correctement annotée.

```dart
class Character {
  final String name;
  int hp;

  Character(this.name, this.hp);

  void attack() {
    print('$name attaque maladroitement.');
  }
}

class Player extends Character {
  Player(String name, int hp) : super(name, hp);

  @override
  void attack() {
    print('$name frappe avec son épée !');
  }
}

void main() {
  Player hero = Player('Alex', 100);
  hero.attack();
}
```

**Résultat :**

```text
Alex frappe avec son épée !
```

`@override` se place juste au-dessus de la méthode redéfinie.

Que fait cette annotation, exactement ?

1. **Elle ne change rien à l'exécution.** Le programme se comporte de façon identique avec ou sans elle.
2. **Elle documente l'intention.** Le lecteur voit immédiatement que cette méthode existe déjà dans le parent.
3. **Elle protège des fautes de frappe.** C'est son intérêt principal.

Imaginons cette erreur :

```text
class Player extends Character {
  @override
  void attak() {          // faute de frappe
    print('...');
  }
}
```

L'outil d'analyse signale :

```text
The member 'attak' overrides an inherited member, but the annotation
@override is applied to a member that does not override anything.
```

Sans `@override`, Dart aurait simplement créé une **nouvelle** méthode `attak()`, et votre `attack()` d'origine serait resté celui du parent. Le bug aurait été silencieux, et vous auriez cherché longtemps pourquoi le héros « attaque maladroitement ».

Règle d'usage, simple et sans exception :

> Chaque fois que vous redéfinissez une méthode, un getter ou un setter hérité, écrivez `@override` au-dessus.

C'est la convention officielle de Dart et de Flutter. Vous verrez `@override` sur presque chaque écran Flutter que vous écrirez.

---

## 10.16 — Le polymorphisme

Polymorphisme vient du grec : « plusieurs formes ». En POO, cela signifie qu'une même variable peut contenir des objets de types différents, et qu'un même appel de méthode déclenche un comportement différent selon l'objet réel.

```dart
class Character {
  final String name;
  int hp;

  Character(this.name, this.hp);

  void attack() {
    print('$name attaque maladroitement.');
  }
}

class Player extends Character {
  Player(String name, int hp) : super(name, hp);

  @override
  void attack() {
    print('$name lance une boule de feu !');
  }
}

class Enemy extends Character {
  Enemy(String name, int hp) : super(name, hp);

  @override
  void attack() {
    print('$name griffe violemment.');
  }
}

void main() {
  Character c1 = Player('Alex', 100);
  Character c2 = Enemy('Gobelin', 40);

  c1.attack();
  c2.attack();
}
```

**Résultat :**

```text
Alex lance une boule de feu !
Gobelin griffe violemment.
```

Regardez la ligne clé :

```text
Character c1 = Player('Alex', 100);
```

La variable est déclarée `Character`, mais l'objet créé est un `Player`. C'est autorisé, car « `Player` EST UN `Character` ». On dit que `Character` est le **type statique** (ce que le compilateur voit) et `Player` le **type dynamique** (ce qui existe réellement en mémoire).

```text
   Type statique       Type dynamique
   (déclaré)           (réel, à l'exécution)
   -----------         --------------------
   Character    c1 --> Player   'Alex'
   Character    c2 --> Enemy    'Gobelin'
```

Quand on écrit `c1.attack()` :

- le **compilateur** vérifie que `Character` possède bien une méthode `attack()` ;
- l'**exécution** cherche la version la plus spécialisée, celle de `Player`.

Ce mécanisme s'appelle la **liaison dynamique**. C'est lui qui rend le polymorphisme utile : votre code parle à des `Character`, et chaque objet répond à sa manière.

L'inverse n'est pas vrai :

```text
Player p = Character('Fantôme', 10);   // INTERDIT
```

Message :

```text
Error: A value of type 'Character' can't be assigned to a variable of type 'Player'.
```

Un `Character` n'est pas forcément un `Player`. La conversion ne fonctionne que dans le sens enfant → parent.

---

## 10.17 — Une `List<Character>` contenant des `Player` et des `Enemy`

Le polymorphisme prend toute sa valeur dans les collections.

```dart
class Character {
  final String name;
  int hp;

  Character(this.name, this.hp);

  void attack() {
    print('$name attaque maladroitement.');
  }

  void describe() {
    print('$name : $hp PV');
  }
}

class Player extends Character {
  int score;

  Player(String name, int hp, this.score) : super(name, hp);

  @override
  void attack() {
    print('$name lance une boule de feu !');
  }
}

class Enemy extends Character {
  int reward;

  Enemy(String name, int hp, this.reward) : super(name, hp);

  @override
  void attack() {
    print('$name griffe violemment.');
  }
}

void main() {
  List<Character> arene = [
    Player('Alex', 100, 0),
    Enemy('Gobelin', 40, 20),
    Enemy('Chauve-souris', 15, 5),
    Player('Sophie', 90, 120),
  ];

  print('--- Tour de table ---');
  for (Character c in arene) {
    c.describe();
  }

  print('');
  print('--- Tous attaquent ---');
  for (Character c in arene) {
    c.attack();
  }
}
```

**Résultat :**

```text
--- Tour de table ---
Alex : 100 PV
Gobelin : 40 PV
Chauve-souris : 15 PV
Sophie : 90 PV

--- Tous attaquent ---
Alex lance une boule de feu !
Gobelin griffe violemment.
Chauve-souris griffe violemment.
Sophie lance une boule de feu !
```

La boucle est écrite **une seule fois**. Elle ne contient aucun `if`. Elle ne sait pas, et n'a pas besoin de savoir, qui est joueur et qui est ennemi.

C'est le gain principal du polymorphisme : le code de haut niveau reste stable même quand vous ajoutez de nouveaux types de personnages. Ajoutez demain une classe `Marchand extends Character` : la boucle fonctionnera sans être modifiée.

Attention à ce que la liste vous laisse faire :

```text
for (Character c in arene) {
  print(c.score);   // ERREUR
}
```

```text
Error: The getter 'score' isn't defined for the class 'Character'.
```

Vu à travers le type `Character`, un `Player` n'expose que ce que `Character` déclare. Pour accéder à `score`, il faut d'abord prouver au compilateur que l'objet est bien un `Player`. C'est l'objet de la section suivante.

---

## 10.18 — `is` et `as`

Dart fournit deux opérateurs pour travailler avec les types à l'exécution.

| Opérateur | Question posée | Résultat |
| --- | --- | --- |
| `is` | cet objet est-il de ce type ? | `true` / `false` |
| `is!` | cet objet n'est-il PAS de ce type ? | `true` / `false` |
| `as` | traite cet objet comme ce type | l'objet converti, ou une exception |

Commençons par `is`.

```dart
class Character {
  final String name;
  int hp;
  Character(this.name, this.hp);
}

class Player extends Character {
  int score;
  Player(String name, int hp, this.score) : super(name, hp);
}

class Enemy extends Character {
  int reward;
  Enemy(String name, int hp, this.reward) : super(name, hp);
}

void main() {
  List<Character> arene = [
    Player('Alex', 100, 0),
    Enemy('Gobelin', 40, 20),
    Player('Sophie', 90, 120),
  ];

  int butinTotal = 0;

  for (Character c in arene) {
    if (c is Player) {
      c.score += 10;
      print('${c.name} : bonus de 10 points, total ${c.score}');
    } else if (c is Enemy) {
      butinTotal += c.reward;
      print('${c.name} : butin de ${c.reward}');
    }
  }

  print('Butin total : $butinTotal');
}
```

**Résultat :**

```text
Alex : bonus de 10 points, total 10
Gobelin : butin de 20
Sophie : bonus de 10 points, total 130
Butin total : 20
```

Observez la ligne `c.score += 10;`. La variable `c` est déclarée `Character`, et pourtant nous accédons à `score`. C'est la **promotion de type** : à l'intérieur d'un `if (c is Player)`, Dart considère automatiquement que `c` est un `Player`. Aucune conversion manuelle n'est nécessaire.

Passons à `as`. Il force la conversion, sans vérification préalable.

```dart
void main() {
  Character c = Player('Alex', 100, 250);

  Player hero = c as Player;
  print(hero.score);
}
```

**Résultat :**

```text
250
```

Mais si le type ne correspond pas, le programme s'arrête à l'exécution.

```dart
void main() {
  Character c = Enemy('Gobelin', 40, 20);

  Player hero = c as Player;
  print(hero.score);
}
```

**Résultat :**

```text
Unhandled exception:
type 'Enemy' is not a subtype of type 'Player' in type cast
```

Le compilateur n'a rien dit : `as` est une promesse que vous faites, et Dart vous croit jusqu'à l'exécution.

Règle de sécurité :

> Préférez toujours `is` à `as`. N'utilisez `as` que lorsque vous êtes absolument certain du type, ou juste après un test `is`.

Un dernier détail utile : `is` renvoie aussi `true` pour les classes parentes.

```dart
void main() {
  Player hero = Player('Alex', 100, 0);

  print(hero is Player);
  print(hero is Character);
  print(hero is Enemy);
  print(hero is! Enemy);
}
```

**Résultat :**

```text
true
true
false
true
```

`hero` est à la fois un `Player` et un `Character`. C'est exactement ce que dit la relation « EST UN ».

---

## 10.19 — Hiérarchie `Character → Player / Enemy / Boss`

Assemblons maintenant tout le chapitre dans une hiérarchie complète.

Voici l'arbre visé :

```text
                    +-------------------------+
                    |        Character        |
                    |  name, maxHp, _hp       |
                    |  get hp, get isAlive    |
                    |  get power              |
                    |  takeDamage(), attack() |
                    |  describe()             |
                    +-------------------------+
                       ^                   ^
                       |                   |
          +------------+                   +------------+
          |                                             |
  +-----------------+                         +--------------------+
  |     Player      |                         |       Enemy        |
  |  score          |                         |  reward            |
  |  power = 12     |                         |  power = 7         |
  +-----------------+                         +--------------------+
                                                        ^
                                                        |
                                              +--------------------+
                                              |        Boss        |
                                              |  phase             |
                                              |  power = 15 * phase|
                                              +--------------------+
```

`Boss` n'hérite pas directement de `Character` mais de `Enemy`, car un boss **est un** ennemi : il donne un butin, il attaque le joueur. Il ajoute simplement une notion de phase.

```dart
class Character {
  final String name;
  final int maxHp;
  int _hp;

  Character(this.name, this.maxHp) : _hp = maxHp;

  int get hp => _hp;
  bool get isAlive => _hp > 0;
  int get power => 5;

  void takeDamage(int amount) {
    if (amount <= 0) {
      return;
    }
    _hp = (_hp - amount).clamp(0, maxHp);
  }

  void attack(Character target) {
    print('$name attaque ${target.name} (-$power PV)');
    target.takeDamage(power);
  }

  String describe() => '$name [$_hp/$maxHp]';
}

class Player extends Character {
  int score;

  Player(String name, int maxHp)
      : score = 0,
        super(name, maxHp);

  @override
  int get power => 12;

  @override
  void attack(Character target) {
    super.attack(target);
    if (!target.isAlive) {
      score += 50;
      print('  $name gagne 50 points (total $score).');
    }
  }

  @override
  String describe() => '${super.describe()} JOUEUR score=$score';
}

class Enemy extends Character {
  final int reward;

  Enemy(String name, int maxHp, this.reward) : super(name, maxHp);

  @override
  int get power => 7;

  @override
  String describe() => '${super.describe()} ENNEMI (butin $reward)';
}

class Boss extends Enemy {
  int phase;

  Boss(String name, int maxHp, int reward)
      : phase = 1,
        super(name, maxHp, reward);

  @override
  int get power => 15 * phase;

  @override
  void takeDamage(int amount) {
    super.takeDamage(amount);
    if (hp <= maxHp ~/ 2 && phase == 1) {
      phase = 2;
      print('  $name entre en phase 2 !');
    }
  }

  @override
  String describe() => '${super.describe()} BOSS phase $phase';
}

void main() {
  Player hero = Player('Alex', 100);
  Enemy gobelin = Enemy('Gobelin', 30, 20);
  Boss dragon = Boss('Dragon Noir', 60, 200);

  List<Character> arene = [hero, gobelin, dragon];

  print('--- État initial ---');
  for (Character c in arene) {
    print(c.describe());
  }

  print('');
  print('--- Combat ---');
  hero.attack(gobelin);
  hero.attack(gobelin);
  hero.attack(gobelin);
  hero.attack(dragon);
  hero.attack(dragon);
  hero.attack(dragon);
  dragon.attack(hero);

  print('');
  print('--- État final ---');
  for (Character c in arene) {
    print(c.describe());
  }
}
```

**Résultat :**

```text
--- État initial ---
Alex [100/100] JOUEUR score=0
Gobelin [30/30] ENNEMI (butin 20)
Dragon Noir [60/60] ENNEMI (butin 200) BOSS phase 1

--- Combat ---
Alex attaque Gobelin (-12 PV)
Alex attaque Gobelin (-12 PV)
Alex attaque Gobelin (-12 PV)
  Alex gagne 50 points (total 50).
Alex attaque Dragon Noir (-12 PV)
Alex attaque Dragon Noir (-12 PV)
Alex attaque Dragon Noir (-12 PV)
  Dragon Noir entre en phase 2 !
Dragon Noir attaque Alex (-30 PV)

--- État final ---
Alex [70/100] JOUEUR score=50
Gobelin [0/30] ENNEMI (butin 20)
Dragon Noir [24/60] ENNEMI (butin 200) BOSS phase 2
```

Détaillons les quatre mécanismes à l'œuvre :

1. **Getter redéfini.** `power` vaut 5 dans `Character`, 12 dans `Player`, 7 dans `Enemy`, `15 * phase` dans `Boss`. La méthode `attack()` de `Character` écrit simplement `power` : c'est la version de l'objet réel qui est utilisée.
2. **`super` dans une redéfinition.** `Player.attack()` appelle `super.attack()` puis ajoute la logique de score. `Boss.takeDamage()` appelle `super.takeDamage()` puis vérifie le passage en phase 2.
3. **Chaîne de `super.describe()`.** Pour le dragon, l'appel traverse trois niveaux : `Boss` → `Enemy` → `Character`. C'est ce qui produit une ligne composée de trois morceaux.
4. **Polymorphisme dans la boucle.** `for (Character c in arene) print(c.describe());` affiche trois formats différents sans un seul `if`.

---

## 10.20 — Quand NE PAS utiliser l'héritage

L'héritage est puissant, donc dangereux. Un débutant a tendance à en mettre partout. Voici les cas où il ne faut pas.

**Cas 1 : la relation n'est pas « EST UN ».**

Le test est simple. Lisez la phrase à voix haute :

```text
Un Player EST UNE Weapon ?        -> non
Un Player A UNE Weapon ?          -> oui
```

Donc ceci est une faute de conception :

```text
class Player extends Weapon { ... }   // MAUVAIS
```

La bonne solution s'appelle la **composition** : l'objet contient un autre objet.

```dart
class Weapon {
  final String name;
  final int damage;

  Weapon(this.name, this.damage);
}

class Player {
  final String name;
  Weapon weapon;

  Player(this.name, this.weapon);

  void attack() {
    print('$name frappe avec ${weapon.name} (-${weapon.damage} PV)');
  }
}

void main() {
  Player hero = Player('Alex', Weapon('Épée courte', 10));
  hero.attack();

  hero.weapon = Weapon('Hache de guerre', 25);
  hero.attack();
}
```

**Résultat :**

```text
Alex frappe avec Épée courte (-10 PV)
Alex frappe avec Hache de guerre (-25 PV)
```

La composition offre en prime une souplesse que l'héritage n'a pas : on peut **changer** l'arme en cours de partie. On ne peut pas changer la classe mère d'un objet une fois créé.

Formule à retenir :

```text
EST UN  -> héritage  (extends)
A UN    -> composition (un champ)
```

**Cas 2 : hériter uniquement pour récupérer du code.**

Si vous écrivez `class Inventory extends List` uniquement parce que vous voulez `add()` et `length`, vous héritez aussi de tout le reste, y compris de méthodes qui n'ont aucun sens pour un inventaire. Préférez un champ `List<Item> _items` à l'intérieur de `Inventory`.

**Cas 3 : la hiérarchie devient trop profonde.**

```text
Character -> Enemy -> FlyingEnemy -> FireFlyingEnemy -> FireFlyingBossEnemy
```

Au-delà de trois niveaux, plus personne ne sait où est défini quoi. Chaque redéfinition doit être cherchée dans cinq fichiers.

**Cas 4 : la classe mère change tout le temps.**

Quand vous modifiez une classe mère, vous modifiez d'un coup toutes ses filles, parfois sans le vouloir. C'est le problème dit de la « classe de base fragile ». Plus une classe a de filles, plus elle doit être stable.

**Cas 5 : vous voulez seulement un contrat commun.**

Si vos classes n'ont **rien** à partager en code, mais doivent simplement toutes posséder une méthode `attack()`, l'héritage n'est pas le bon outil. Dart propose pour cela `abstract` et `implements`, ainsi que les mixins pour partager du comportement sans lien de parenté. C'est le sujet du chapitre 11.

Récapitulatif de décision :

| Situation | Bon outil |
| --- | --- |
| « B EST UN A », et A contient du code réutilisable | `extends` |
| « B A UN A » | composition (un champ) |
| Besoin d'un simple contrat sans code partagé | chapitre 11 |
| Besoin de partager un comportement entre classes non parentes | chapitre 11 |

---

## 10.21 — Erreurs fréquentes

Voici les fautes que commettent presque tous les débutants sur ce chapitre. Lisez ce tableau avant de faire les exercices, puis relisez-le après.

| Erreur | Cause | Correction |
| --- | --- | --- |
| `The superclass 'Character' doesn't have a zero argument constructor` | La sous-classe ne transmet rien au constructeur parent alors que le parent exige des arguments. | Ajouter l'appel explicite : `Player(String name, int maxHp) : super(name, maxHp);` |
| `super call must be last in an initializer list` | `super(...)` est écrit avant les autres initialisations. | Placer `super(...)` en **dernier** : `: score = 0, super(name, maxHp);` |
| Croire que `_hp` est privé « à la classe » | En Dart, le `_` rend le membre privé **à la bibliothèque** (le fichier), pas à la classe. | Dans un même fichier, une autre classe peut lire `_hp`. Pour une vraie séparation, placez la classe dans son propre fichier. |
| `The getter '_hp' isn't defined for the type 'Player'` depuis un autre fichier | On tente d'accéder à un membre privé importé d'une autre bibliothèque. | Passer par un getter public (`hp`) ou une méthode publique. Ne jamais retirer le `_` pour « faire compiler ». |
| `'Player.attack' has fewer positional arguments than those of overridden method` | La méthode redéfinie n'a pas la même signature que celle du parent. | Recopier exactement la signature du parent : mêmes types de paramètres, même type de retour compatible. |
| Le parent continue d'être appelé alors qu'on voulait remplacer | On a écrit une méthode au nom légèrement différent (`atack`, `Attack`, `attack2`). | Ajouter `@override` : le compilateur signale immédiatement que rien n'est redéfini. |
| `The method doesn't override an inherited member` | `@override` posé sur une méthode qui n'existe pas dans le parent. | Corriger le nom, ou retirer `@override` s'il s'agit d'une nouvelle méthode propre à la fille. |
| Le programme se fige puis plante (`Stack Overflow`) | Un setter qui s'appelle lui-même : `set hp(int v) { hp = v; }`. | Le setter doit écrire dans le **champ privé** : `set hp(int v) { _hp = v; }` |
| `'hp' is already declared in this scope` | Un champ public et un getter portent le même nom : `int hp;` et `int get hp => hp;`. | Renommer le champ en `_hp` et garder `int get hp => _hp;`. |
| `Player extends Weapon` | Héritage abusif : on hérite pour récupérer du code, pas parce que la relation est « EST UN ». | Utiliser la composition : `class Player { Weapon weapon; }`. Test : « un joueur EST une arme ? » Non. |
| `The method 'drinkPotion' isn't defined for the type 'Character'` | On appelle une méthode de la sous-classe sur une variable déclarée `Character`. | Tester le type avant : `if (c is Player) { c.drinkPotion(); }`. |
| `type 'Enemy' is not a subtype of type 'Player' in type cast` | Un `as` forcé sur le mauvais type, à l'exécution. | Toujours protéger : `if (c is Player) { ... }` plutôt que `c as Player` en aveugle. |
| Le getter redéfini semble ignoré | On croit que la méthode du parent utilise « sa » version du getter. | Non : Dart choisit toujours la version de l'objet **réel**. `attack()` défini dans `Character` utilise le `power` de `Boss` si l'objet est un `Boss`. |
| Un getter renvoie la liste privée et l'extérieur la modifie | `List<String> get items => _items;` donne la **référence** de la liste interne. | Renvoyer une copie : `List<String> get items => List<String>.from(_items);` |
| Le setter valide mais ne borne pas | On écrit `if (v < 0) return;` sans traiter le dépassement du maximum. | Traiter les **deux** bornes, ou utiliser `_hp = v.clamp(0, maxHp);`. |
| `Only static members can be accessed in initializers` | On utilise `this` ou un getter dans la liste d'initialisation. | Déplacer le calcul dans le corps du constructeur, après `{`. |

---

## 10.22 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| Encapsulation | Cacher les données, n'exposer qu'une façade contrôlée. La classe garantit ses propres règles. |
| Préfixe `_` | Rend un membre privé **à la bibliothèque** (au fichier), pas à la classe. |
| Getter | `int get hp => _hp;` — expose une valeur en lecture seule, ou calcule une valeur dérivée. |
| Getter calculé | `bool get isAlive => _hp > 0;` — ne se stocke pas, se recalcule. Une seule source de vérité. |
| Setter | `set hp(int v) { _hp = v.clamp(0, maxHp); }` — filtre et corrige avant d'écrire. |
| Méthode privée | `void _clamp()` — logique interne factorisée, invisible de l'extérieur. |
| Invariant | Une règle qui doit rester vraie en permanence (par exemple `0 <= hp <= maxHp`). L'encapsulation la protège. |
| Héritage | `class Player extends Character` — la fille reçoit tout le contenu de la mère. |
| Ce qui est hérité | Les propriétés, les getters, les setters et les méthodes. **Pas** les constructeurs. |
| `super(...)` | Dans la liste d'initialisation, appelle le constructeur du parent. Toujours en **dernier**. |
| `super.methode()` | Appelle la version parente d'une méthode redéfinie. Sert à « compléter » au lieu de « remplacer ». |
| Redéfinition (override) | Réécrire dans la fille une méthode déjà définie dans la mère, avec la **même signature**. |
| `@override` | Annotation facultative pour le compilateur, indispensable en pratique : elle attrape les fautes de frappe. |
| Polymorphisme | Une variable de type `Character` peut contenir un `Player`, un `Enemy` ou un `Boss`, et l'appel exécute la bonne version. |
| `List<Character>` | Permet de traiter des objets différents dans une seule boucle, sans `if` sur le type. |
| `is` | Teste le type et **promeut** la variable : dans le `if`, Dart la voit avec le type testé. |
| `as` | Force la conversion. Plante à l'exécution si le type est faux. À réserver aux cas certains. |
| `EST UN` / `A UN` | « EST UN » → héritage. « A UN » → composition (un champ). |
| Profondeur de hiérarchie | Trois niveaux au maximum. Au-delà, plus personne ne s'y retrouve. |
| Quand ne pas hériter | Relation qui n'est pas « EST UN », héritage pour récupérer du code, classe mère instable, besoin d'un simple contrat. |

---

## 10.23 — Exercices

Faites-les dans l'ordre : chacun réutilise ce que le précédent a mis en place. Écrivez d'abord votre solution complète dans DartPad, exécutez-la, et comparez seulement ensuite avec la correction.

### Exercice 1 — Un score que l'on ne peut pas truquer (facile)

Écrivez une classe `Score` avec :

- une propriété privée `_points`, initialisée à `0` ;
- un getter public `points` ;
- une méthode `add(int value)` qui refuse les valeurs négatives ou nulles.

Dans `main()`, créez un `Score`, affichez-le, ajoutez `120`, essayez d'ajouter `-50`, puis affichez de nouveau.

### Exercice 2 — Getters calculés (facile)

Écrivez une classe `Player` avec `name`, `maxHp` (tous deux `final`) et `_hp` privé initialisé à `maxHp`.

Ajoutez :

- un getter `hp` ;
- un getter `isAlive` qui vaut `true` si `_hp > 0` ;
- un getter `hpPercent` qui renvoie le pourcentage de vie restant, en entier ;
- une méthode `takeDamage(int amount)` qui ne descend jamais sous `0`.

Testez avec un joueur de 200 PV que vous frappez de `150`, puis de `500`.

### Exercice 3 — Un setter qui corrige (facile)

Écrivez une classe `GameSettings` avec une propriété privée `_volume` initialisée à `50`.

Ajoutez un getter `volume` et un setter `volume` qui :

- ramène à `0` toute valeur inférieure à `0`, en affichant un message ;
- ramène à `100` toute valeur supérieure à `100`, en affichant un message ;
- accepte silencieusement les valeurs comprises entre `0` et `100`.

Testez avec `80`, `250` puis `-10`.

### Exercice 4 — Une méthode privée d'aide (facile)

Écrivez une classe `Energy` avec `max` (final) et `_value` privé.

Ajoutez une **méthode privée** `_apply(int newValue)` qui borne la valeur entre `0` et `max`, puis utilisez-la dans `spend(int amount)` et `recover(int amount)`.

Ajoutez enfin un getter `bar` qui renvoie une barre de progression sur dix caractères, par exemple `[#######...] 35/50`.

### Exercice 5 — La bourse du héros (moyen)

Écrivez une classe `Wallet` totalement encapsulée :

- `_gold` privé, jamais négatif, même si on tente de créer la bourse avec un montant négatif ;
- un getter `gold` ;
- `earn(int amount)` qui refuse les montants invalides ;
- `spend(int amount)` qui renvoie `true` si la dépense a pu se faire, `false` sinon, et qui refuse de descendre sous zéro.

### Exercice 6 — Premier `extends` (moyen)

Écrivez une classe `Item` avec `name` et `price`, et une méthode `describe()`.

Écrivez une classe `Potion extends Item` qui ajoute `healing` et redéfinit `describe()` en réutilisant `super.describe()`.

Rangez une `Item` et deux `Potion` dans une `List<Item>` et affichez tout dans une boucle.

### Exercice 7 — `super` dans le constructeur (moyen)

Écrivez une classe `Character` avec `name`, `maxHp`, `_hp`, un getter `hp`, un getter `isAlive`, une méthode `takeDamage()` et une méthode `describe()`.

Écrivez `Archer extends Character` qui ajoute une propriété `arrows` (paramètre nommé, valeur par défaut `10`) et une méthode `shoot(Character target)` qui inflige `15` dégâts et consomme une flèche. Quand il n'y a plus de flèche, l'archer le signale et ne tire pas.

### Exercice 8 — Redéfinir une méthode (moyen)

Écrivez une classe `Enemy` avec une méthode `cry()`.

Écrivez `Goblin` et `Ghost` qui héritent de `Enemy` et redéfinissent `cry()` chacune à leur manière.

Placez les trois objets dans une `List<Enemy>` et appelez `cry()` dans une boucle.

### Exercice 9 — Chaîne de `super.methode()` (moyen)

Écrivez trois classes en cascade :

- `Weapon` (`name`, `damage`, `describe()`) ;
- `MagicWeapon extends Weapon` (ajoute `element` et `bonus`) ;
- `LegendaryWeapon extends MagicWeapon` (ajoute `owner`).

Chaque `describe()` doit appeler `super.describe()` et compléter la phrase. Vérifiez que l'arme légendaire produit bien une ligne composée de trois morceaux.

### Exercice 10 — Polymorphisme dans une boucle (moyen)

Écrivez `Character` avec un getter `power` valant `5` et une méthode `describe()`.

Écrivez `Player` (`power` = `12`) et `Enemy` (`power` = `7`), qui redéfinissent aussi `describe()` en réutilisant `super.describe()`.

Dans `main()`, remplissez une `List<Character>` avec un joueur et deux ennemis, affichez chaque description et calculez la somme des puissances. Le tout **sans un seul `if`**.

### Exercice 11 — `is` et `as` (difficile)

Reprenez `Character`, `Player` (avec `score`) et `Enemy` (avec `reward`).

Dans `main()`, créez une `List<Character>` contenant deux joueurs et trois ennemis. Dans une seule boucle :

- comptez les joueurs et ajoutez `10` points au score de chacun ;
- additionnez les butins de tous les ennemis.

Terminez en récupérant le premier élément de la liste avec `as` et affichez son score.

### Exercice 12 — Mini-projet : le bestiaire (difficile)

Construisez un petit bestiaire complet.

**Classe mère `Monster`** avec :

- `name`, `maxHp`, `reward` (finaux) et `_hp` privé ;
- les getters `hp`, `isAlive`, `power` (valeur `5`) et `family` (valeur `'Monstre'`) ;
- `takeDamage(int amount)` qui borne la vie entre `0` et `maxHp` ;
- `cry()` qui renvoie `'...'` ;
- `describe()` qui affiche la famille, le nom, la vie, la puissance et le butin.

**Trois sous-classes :**

1. `Slime` — 20 PV, butin 5, puissance 2, famille `'Gluant'`. Quand il meurt, il se divise en deux petits gluants (un message s'affiche, une seule fois).
2. `Skeleton` — 45 PV, butin 25, puissance 9, famille `'Squelette'`, plus une propriété `weapon` ajoutée à la description.
3. `DragonBoss` — 150 PV, butin 500, famille `'BOSS Dragon'`, avec une propriété `phase`. Sa puissance vaut `20 * phase`. Quand sa vie tombe à la moitié ou moins, il passe en phase 2 et son cri change.

**Dans `main()` :**

- rangez les trois monstres dans une `List<Monster>` ;
- affichez le bestiaire complet (description + cri) dans une boucle polymorphe ;
- simulez deux tours de combat où le héros inflige `40` dégâts à chaque monstre encore vivant ;
- affichez l'état final, le nombre de monstres encore debout et le butin total ramassé.

Contrainte : la boucle d'affichage ne doit contenir **aucun test de type**. Tout doit passer par le polymorphisme.

---

## 10.24 — Corrections des exercices

### Correction 1

```dart
class Score {
  int _points = 0;

  int get points => _points;

  void add(int value) {
    if (value <= 0) {
      print('Ajout refusé : $value');
      return;
    }
    _points += value;
  }
}

void main() {
  Score score = Score();
  print('Score de départ : ${score.points}');

  score.add(120);
  score.add(-50);

  print('Score final : ${score.points}');
}
```

**Résultat :**

```text
Score de départ : 0
Ajout refusé : -50
Score final : 120
```

**Explication :** `_points` est privé, donc `main()` ne peut pas écrire `score._points = 999999`. La seule porte d'entrée est `add()`, et cette porte contient la règle « un ajout est toujours positif ». Le getter `points` offre une lecture sans écriture : c'est exactement le sens de « lecture seule ». Remarquez qu'aucune ligne de `main()` n'a besoin de connaître la règle : la classe la porte elle-même.

---

### Correction 2

```dart
class Player {
  final String name;
  final int maxHp;

  int _hp;

  Player(this.name, this.maxHp) : _hp = maxHp;

  int get hp => _hp;
  bool get isAlive => _hp > 0;
  int get hpPercent => (_hp * 100) ~/ maxHp;

  void takeDamage(int amount) {
    if (amount <= 0) {
      return;
    }
    _hp = (_hp - amount).clamp(0, maxHp);
  }

  void show() {
    print('$name $_hp/$maxHp -> $hpPercent% | vivant : $isAlive');
  }
}

void main() {
  Player hero = Player('Alex', 200);
  hero.show();

  hero.takeDamage(150);
  hero.show();

  hero.takeDamage(500);
  hero.show();
}
```

**Résultat :**

```text
Alex 200/200 -> 100% | vivant : true
Alex 50/200 -> 25% | vivant : true
Alex 0/200 -> 0% | vivant : false
```

**Explication :** `isAlive` et `hpPercent` ne sont pas des propriétés stockées : ce sont des getters **calculés**. Ils se recalculent à chaque lecture à partir de `_hp`. C'est important : si vous aviez stocké un champ `bool estVivant`, il aurait fallu penser à le mettre à jour dans `takeDamage()`, et un jour vous auriez oublié. Une seule source de vérité, `_hp`, et tout le reste en découle. Le `clamp(0, maxHp)` garantit l'invariant `0 <= _hp <= maxHp` même face à un coup de 500 dégâts.

---

### Correction 3

```dart
class GameSettings {
  int _volume = 50;

  int get volume => _volume;

  set volume(int value) {
    if (value < 0) {
      print('Volume $value trop bas : ramené à 0.');
      _volume = 0;
      return;
    }
    if (value > 100) {
      print('Volume $value trop haut : ramené à 100.');
      _volume = 100;
      return;
    }
    _volume = value;
  }
}

void main() {
  GameSettings options = GameSettings();
  print('Volume : ${options.volume}');

  options.volume = 80;
  print('Volume : ${options.volume}');

  options.volume = 250;
  print('Volume : ${options.volume}');

  options.volume = -10;
  print('Volume : ${options.volume}');
}
```

**Résultat :**

```text
Volume : 50
Volume : 80
Volume 250 trop haut : ramené à 100.
Volume : 100
Volume -10 trop bas : ramené à 0.
Volume : 0
```

**Explication :** le setter conserve la syntaxe agréable d'une propriété (`options.volume = 250;`) tout en exécutant du code. C'est là tout son intérêt : le code appelant ne change pas, mais la classe garde le contrôle. Attention au piège classique : à l'intérieur du setter, il faut écrire `_volume = value;` et surtout pas `volume = value;`, qui rappellerait le setter à l'infini et ferait planter le programme.

---

### Correction 4

```dart
class Energy {
  final int max;

  int _value;

  Energy(this.max) : _value = max;

  int get value => _value;

  String get bar {
    int filled = (_value * 10) ~/ max;
    return '[${'#' * filled}${'.' * (10 - filled)}] $_value/$max';
  }

  void _apply(int newValue) {
    _value = newValue.clamp(0, max);
  }

  void spend(int amount) {
    if (amount <= 0) {
      return;
    }
    _apply(_value - amount);
  }

  void recover(int amount) {
    if (amount <= 0) {
      return;
    }
    _apply(_value + amount);
  }
}

void main() {
  Energy energie = Energy(50);
  print(energie.bar);

  energie.spend(15);
  print(energie.bar);

  energie.spend(100);
  print(energie.bar);

  energie.recover(30);
  print(energie.bar);

  energie.recover(1000);
  print(energie.bar);
}
```

**Résultat :**

```text
[##########] 50/50
[#######...] 35/50
[..........] 0/50
[######....] 30/50
[##########] 50/50
```

**Explication :** `_apply()` est une méthode privée. Elle contient la règle de bornage, écrite **une seule fois**. `spend()` et `recover()` se contentent de calculer une nouvelle valeur et de la lui confier. Si demain la règle change (par exemple un minimum de 5 au lieu de 0), il n'y a qu'un seul endroit à modifier. Notez aussi l'opérateur `*` appliqué à une chaîne : `'#' * 3` produit `'###'`.

---

### Correction 5

```dart
class Wallet {
  int _gold;

  Wallet({int gold = 0}) : _gold = gold < 0 ? 0 : gold;

  int get gold => _gold;

  void earn(int amount) {
    if (amount <= 0) {
      print('Gain invalide : $amount');
      return;
    }
    _gold += amount;
    print('+$amount or (total $_gold)');
  }

  bool spend(int amount) {
    if (amount <= 0) {
      print('Dépense invalide : $amount');
      return false;
    }
    if (amount > _gold) {
      print('Fonds insuffisants : $amount demandés, $_gold disponibles.');
      return false;
    }
    _gold -= amount;
    print('-$amount or (total $_gold)');
    return true;
  }
}

void main() {
  Wallet bourse = Wallet(gold: 100);

  bourse.earn(50);
  bourse.spend(30);
  bourse.spend(1000);
  bourse.earn(-5);

  Wallet triche = Wallet(gold: -500);
  print('Bourse tricheuse : ${triche.gold}');

  print('Solde final : ${bourse.gold}');
}
```

**Résultat :**

```text
+50 or (total 150)
-30 or (total 120)
Fonds insuffisants : 1000 demandés, 120 disponibles.
Gain invalide : -5
Bourse tricheuse : 0
Solde final : 120
```

**Explication :** l'invariant est ici « l'or n'est jamais négatif ». Il est protégé à trois endroits : dans la liste d'initialisation du constructeur, dans `earn()` et dans `spend()`. C'est le principe même de l'encapsulation : une règle vraie **à tout instant de la vie de l'objet**, y compris à sa naissance. `spend()` renvoie un `bool` plutôt que rien : le code appelant peut ainsi savoir si l'achat a réussi et réagir en conséquence.

---

### Correction 6

```dart
class Item {
  final String name;
  final int price;

  Item(this.name, this.price);

  String describe() => '$name ($price or)';
}

class Potion extends Item {
  final int healing;

  Potion(String name, int price, this.healing) : super(name, price);

  @override
  String describe() => '${super.describe()} soigne $healing PV';
}

void main() {
  Item epee = Item('Épée courte', 150);
  Potion petite = Potion('Petite potion', 25, 30);
  Potion grande = Potion('Grande potion', 60, 80);

  List<Item> boutique = [epee, petite, grande];

  print('--- BOUTIQUE ---');
  for (Item article in boutique) {
    print(article.describe());
  }
}
```

**Résultat :**

```text
--- BOUTIQUE ---
Épée courte (150 or)
Petite potion (25 or) soigne 30 PV
Grande potion (60 or) soigne 80 PV
```

**Explication :** `Potion` hérite de `name`, de `price` et de `describe()`. Le constructeur, lui, n'est **pas** hérité : il faut l'écrire et transmettre les valeurs au parent avec `super(name, price)`. La redéfinition de `describe()` ne recopie pas le texte du parent, elle l'appelle avec `super.describe()` puis le complète. Résultat : si un jour le format de `Item.describe()` change, `Potion` suit automatiquement. Enfin, la liste est déclarée `List<Item>` : elle accepte les `Potion`, car une potion **est un** objet.

---

### Correction 7

```dart
class Character {
  final String name;
  final int maxHp;

  int _hp;

  Character(this.name, this.maxHp) : _hp = maxHp;

  int get hp => _hp;
  bool get isAlive => _hp > 0;

  void takeDamage(int amount) {
    if (amount <= 0) {
      return;
    }
    _hp = (_hp - amount).clamp(0, maxHp);
  }

  String describe() => '$name [$_hp/$maxHp]';
}

class Archer extends Character {
  int arrows;

  Archer(String name, int maxHp, {this.arrows = 10}) : super(name, maxHp);

  void shoot(Character target) {
    if (arrows <= 0) {
      print('$name n\'a plus de flèches.');
      return;
    }
    arrows--;
    target.takeDamage(15);
    print('$name tire sur ${target.name} (-15 PV, reste $arrows flèche(s))');
  }

  @override
  String describe() => '${super.describe()} ARCHER $arrows flèche(s)';
}

void main() {
  Archer robin = Archer('Robin', 90, arrows: 2);
  Character cible = Character('Mannequin', 100);

  print(robin.describe());
  print(cible.describe());
  print('');

  robin.shoot(cible);
  robin.shoot(cible);
  robin.shoot(cible);

  print('');
  print(robin.describe());
  print(cible.describe());
}
```

**Résultat :**

```text
Robin [90/90] ARCHER 2 flèche(s)
Mannequin [100/100]

Robin tire sur Mannequin (-15 PV, reste 1 flèche(s))
Robin tire sur Mannequin (-15 PV, reste 0 flèche(s))
Robin n'a plus de flèches.

Robin [90/90] ARCHER 0 flèche(s)
Mannequin [70/100]
```

**Explication :** trois points méritent l'attention. D'abord, `super(name, maxHp)` est placé **après** la liste des paramètres, en fin de déclaration : c'est la forme obligatoire. Ensuite, `Archer` peut appeler `target.takeDamage(15)` alors que `takeDamage()` est définie dans `Character` : la méthode est héritée. Enfin, `shoot()` n'existe que dans `Archer`. Une variable déclarée `Character` ne pourrait pas l'appeler, même si l'objet réel est un archer : il faudrait d'abord tester avec `is`.

---

### Correction 8

```dart
class Enemy {
  final String name;

  Enemy(this.name);

  String cry() => '$name grogne.';
}

class Goblin extends Enemy {
  Goblin(String name) : super(name);

  @override
  String cry() => '$name ricane : "Gnahaha !"';
}

class Ghost extends Enemy {
  Ghost(String name) : super(name);

  @override
  String cry() => '$name gémit : "Ouuuuh..."';
}

void main() {
  List<Enemy> horde = [
    Enemy('Créature'),
    Goblin('Gob'),
    Ghost('Spectre'),
  ];

  for (Enemy monstre in horde) {
    print(monstre.cry());
  }
}
```

**Résultat :**

```text
Créature grogne.
Gob ricane : "Gnahaha !"
Spectre gémit : "Ouuuuh..."
```

**Explication :** la boucle est déclarée `for (Enemy monstre in horde)`. Au moment d'écrire cette ligne, Dart ne sait pas quel type réel se cachera derrière `monstre`. C'est à l'exécution qu'il regarde l'objet et choisit la bonne version de `cry()`. C'est le polymorphisme en une ligne. Retirez les `@override` : le programme fonctionne toujours, mais si vous écrivez `cri()` au lieu de `cry()` dans `Goblin`, le compilateur ne dira rien et le gobelin grognera comme une créature quelconque. Avec `@override`, l'erreur est signalée immédiatement.

---

### Correction 9

```dart
class Weapon {
  final String name;
  final int damage;

  Weapon(this.name, this.damage);

  String describe() => '$name (-$damage PV)';
}

class MagicWeapon extends Weapon {
  final String element;
  final int bonus;

  MagicWeapon(String name, int damage, this.element, this.bonus)
      : super(name, damage);

  @override
  String describe() => '${super.describe()} + $bonus dégâts de $element';
}

class LegendaryWeapon extends MagicWeapon {
  final String owner;

  LegendaryWeapon(
    String name,
    int damage,
    String element,
    int bonus,
    this.owner,
  ) : super(name, damage, element, bonus);

  @override
  String describe() => '${super.describe()} — légendaire, forgée pour $owner';
}

void main() {
  List<Weapon> armurerie = [
    Weapon('Épée courte', 10),
    MagicWeapon('Lame de givre', 18, 'glace', 6),
    LegendaryWeapon('Excalibur', 40, 'lumière', 20, 'Arthur'),
  ];

  for (Weapon arme in armurerie) {
    print(arme.describe());
  }
}
```

**Résultat :**

```text
Épée courte (-10 PV)
Lame de givre (-18 PV) + 6 dégâts de glace
Excalibur (-40 PV) + 20 dégâts de lumière — légendaire, forgée pour Arthur
```

**Explication :** la dernière ligne est produite par trois méthodes différentes qui se passent le relais. `LegendaryWeapon.describe()` appelle `MagicWeapon.describe()`, qui appelle `Weapon.describe()`. Chaque niveau ajoute son morceau, personne ne recopie le travail du voisin. Représentation de la remontée :

```text
LegendaryWeapon.describe()
        |
        +--> super.describe()  =  MagicWeapon.describe()
                     |
                     +--> super.describe()  =  Weapon.describe()
                                  |
                                  +--> "Excalibur (-40 PV)"
                     <-- "Excalibur (-40 PV) + 20 dégâts de lumière"
        <-- "Excalibur (-40 PV) + 20 dégâts de lumière — légendaire, forgée pour Arthur"
```

---

### Correction 10

```dart
class Character {
  final String name;
  int hp;

  Character(this.name, this.hp);

  int get power => 5;

  String describe() => '$name [$hp PV] puissance $power';
}

class Player extends Character {
  Player(String name, int hp) : super(name, hp);

  @override
  int get power => 12;

  @override
  String describe() => '${super.describe()} (joueur)';
}

class Enemy extends Character {
  Enemy(String name, int hp) : super(name, hp);

  @override
  int get power => 7;

  @override
  String describe() => '${super.describe()} (ennemi)';
}

void main() {
  List<Character> arene = [
    Player('Alex', 100),
    Enemy('Gobelin', 30),
    Enemy('Orc', 55),
  ];

  int total = 0;

  for (Character combattant in arene) {
    print(combattant.describe());
    total += combattant.power;
  }

  print('Puissance totale : $total');
}
```

**Résultat :**

```text
Alex [100 PV] puissance 12 (joueur)
Gobelin [30 PV] puissance 7 (ennemi)
Orc [55 PV] puissance 7 (ennemi)
Puissance totale : 26
```

**Explication :** observez la première ligne du résultat : `puissance 12`. Ce texte est produit par `Character.describe()`, qui écrit simplement `$power`. Pourtant, c'est bien le `power` de `Player` qui est utilisé, parce que l'objet réel est un `Player`. Un getter redéfini se comporte exactement comme une méthode redéfinie. C'est le point que les débutants trouvent le plus surprenant, et c'est le cœur du polymorphisme : la méthode du parent utilise la version de l'enfant. La boucle, elle, ne contient aucun `if` et n'aurait pas besoin d'être modifiée si vous ajoutiez demain une classe `Boss`.

---

### Correction 11

```dart
class Character {
  final String name;
  int hp;

  Character(this.name, this.hp);

  String describe() => '$name [$hp PV]';
}

class Player extends Character {
  int score;

  Player(String name, int hp)
      : score = 0,
        super(name, hp);
}

class Enemy extends Character {
  final int reward;

  Enemy(String name, int hp, this.reward) : super(name, hp);
}

void main() {
  List<Character> arene = [
    Player('Alex', 100),
    Enemy('Gobelin', 30, 20),
    Enemy('Orc', 55, 45),
    Player('Sophie', 100),
    Enemy('Dragon', 200, 500),
  ];

  int joueurs = 0;
  int butin = 0;

  for (Character c in arene) {
    if (c is Player) {
      joueurs++;
      c.score += 10;
      print('${c.name} : joueur, score ${c.score}');
    } else if (c is Enemy) {
      butin += c.reward;
      print('${c.name} : ennemi, butin ${c.reward}');
    }
  }

  print('');
  print('Joueurs : $joueurs');
  print('Butin total : $butin or');

  Character premier = arene[0];
  Player heros = premier as Player;
  print('Premier de la liste : ${heros.name}, score ${heros.score}');
}
```

**Résultat :**

```text
Alex : joueur, score 10
Gobelin : ennemi, butin 20
Orc : ennemi, butin 45
Sophie : joueur, score 10
Dragon : ennemi, butin 500

Joueurs : 2
Butin total : 565 or
Premier de la liste : Alex, score 10
```

**Explication :** dans `if (c is Player)`, la variable `c` est déclarée `Character`, donc `c.score` devrait être refusé. Il ne l'est pas, grâce à la **promotion de type** : à l'intérieur du `if`, Dart sait que `c` est un `Player` et vous laisse accéder aux membres de `Player`. C'est pour cela qu'on préfère toujours `is` à `as`. La dernière ligne utilise `as` : elle fonctionne ici parce que nous savons que `arene[0]` est un joueur. Si vous écriviez `arene[1] as Player`, le programme compilerait mais planterait à l'exécution avec `type 'Enemy' is not a subtype of type 'Player' in type cast`. Réservez `as` aux cas où vous êtes certain, et préférez `is` partout ailleurs.

---

### Correction 12 — Mini-projet « bestiaire »

Voici l'arbre que nous allons écrire :

```text
                     +-----------------------------+
                     |           Monster           |
                     |  name, maxHp, reward, _hp   |
                     |  get hp, isAlive            |
                     |  get power  = 5             |
                     |  get family = 'Monstre'     |
                     |  takeDamage(), cry()        |
                     |  describe()                 |
                     +-----------------------------+
                       ^          ^            ^
                       |          |            |
        +--------------+          |            +--------------+
        |                         |                           |
+-----------------+   +------------------------+   +-------------------------+
|      Slime      |   |       Skeleton         |   |       DragonBoss        |
|  20 PV, butin 5 |   |  45 PV, butin 25       |   |  150 PV, butin 500      |
|  power  = 2     |   |  power  = 9            |   |  power  = 20 * phase    |
|  family = Gluant|   |  family = Squelette    |   |  family = BOSS Dragon   |
|  splitCount     |   |  weapon                |   |  phase                  |
+-----------------+   +------------------------+   +-------------------------+
```

```dart
class Monster {
  final String name;
  final int maxHp;
  final int reward;

  int _hp;

  Monster(this.name, this.maxHp, this.reward) : _hp = maxHp;

  int get hp => _hp;
  bool get isAlive => _hp > 0;
  int get power => 5;
  String get family => 'Monstre';

  void takeDamage(int amount) {
    if (amount <= 0) {
      return;
    }
    _hp = (_hp - amount).clamp(0, maxHp);
  }

  String cry() => '...';

  String describe() =>
      '$family $name [$_hp/$maxHp] puissance $power butin $reward';
}

class Slime extends Monster {
  int splitCount;

  Slime(String name)
      : splitCount = 0,
        super(name, 20, 5);

  @override
  int get power => 2;

  @override
  String get family => 'Gluant';

  @override
  String cry() => 'Blop blop.';

  @override
  void takeDamage(int amount) {
    super.takeDamage(amount);
    if (!isAlive && splitCount == 0) {
      splitCount = 2;
      print('  $name se divise en $splitCount petits gluants !');
    }
  }
}

class Skeleton extends Monster {
  final String weapon;

  Skeleton(String name, this.weapon) : super(name, 45, 25);

  @override
  int get power => 9;

  @override
  String get family => 'Squelette';

  @override
  String cry() => 'Clac clac clac.';

  @override
  String describe() => '${super.describe()} arme=$weapon';
}

class DragonBoss extends Monster {
  int phase;

  DragonBoss(String name)
      : phase = 1,
        super(name, 150, 500);

  @override
  int get power => 20 * phase;

  @override
  String get family => 'BOSS Dragon';

  @override
  String cry() => phase == 1 ? 'Grrrrrr...' : 'RAAAAAAAH !';

  @override
  void takeDamage(int amount) {
    super.takeDamage(amount);
    if (hp <= maxHp ~/ 2 && phase == 1) {
      phase = 2;
      print('  $name entre en phase 2 !');
    }
  }

  @override
  String describe() => '${super.describe()} phase $phase';
}

void main() {
  List<Monster> bestiaire = [
    Slime('Gloubi'),
    Skeleton('Osselet', 'sabre rouillé'),
    DragonBoss('Ignis'),
  ];

  print('--- BESTIAIRE ---');
  for (Monster m in bestiaire) {
    print(m.describe());
    print('  cri : ${m.cry()}');
  }

  print('');
  print('--- COMBAT : deux frappes de 40 dégâts ---');
  for (int tour = 1; tour <= 2; tour++) {
    print('Tour $tour');
    for (Monster m in bestiaire) {
      if (!m.isAlive) {
        print('  ${m.name} est déjà vaincu.');
        continue;
      }
      m.takeDamage(40);
      print('  ${m.name} -> ${m.hp}/${m.maxHp}');
    }
  }

  print('');
  print('--- ÉTAT FINAL ---');
  int butin = 0;
  int debout = 0;
  for (Monster m in bestiaire) {
    print(m.describe());
    print('  cri : ${m.cry()}');
    if (m.isAlive) {
      debout++;
    } else {
      butin += m.reward;
    }
  }

  print('');
  print('Monstres encore debout : $debout');
  print('Butin ramassé : $butin or');
}
```

**Résultat :**

```text
--- BESTIAIRE ---
Gluant Gloubi [20/20] puissance 2 butin 5
  cri : Blop blop.
Squelette Osselet [45/45] puissance 9 butin 25 arme=sabre rouillé
  cri : Clac clac clac.
BOSS Dragon Ignis [150/150] puissance 20 butin 500 phase 1
  cri : Grrrrrr...

--- COMBAT : deux frappes de 40 dégâts ---
Tour 1
  Gloubi se divise en 2 petits gluants !
  Gloubi -> 0/20
  Osselet -> 5/45
  Ignis -> 110/150
Tour 2
  Gloubi est déjà vaincu.
  Osselet -> 0/45
  Ignis entre en phase 2 !
  Ignis -> 70/150

--- ÉTAT FINAL ---
Gluant Gloubi [0/20] puissance 2 butin 5
  cri : Blop blop.
Squelette Osselet [0/45] puissance 9 butin 25 arme=sabre rouillé
  cri : Clac clac clac.
BOSS Dragon Ignis [70/150] puissance 40 butin 500 phase 2
  cri : RAAAAAAAH !

Monstres encore debout : 1
Butin ramassé : 30 or
```

**Explication :** ce mini-projet réunit tout le chapitre. Reprenons mécanisme par mécanisme.

1. **Encapsulation.** `_hp` est privé. Aucun des trois monstres, ni le `main()`, ne l'écrit directement. Tout passe par `takeDamage()`, qui applique `clamp(0, maxHp)`. Le gluant frappé de 40 dégâts alors qu'il n'a que 20 PV tombe à `0`, jamais à `-20`.

2. **Héritage.** Les trois sous-classes ne redéclarent ni `name`, ni `maxHp`, ni `reward`, ni `_hp`, ni `isAlive`, ni `describe()`. Elles reçoivent tout de `Monster`. Chaque constructeur transmet ses valeurs fixes au parent : `super(name, 20, 5)` pour le gluant, `super(name, 45, 25)` pour le squelette, `super(name, 150, 500)` pour le dragon. Remarquez que `super(...)` est toujours écrit **en dernier** dans la liste d'initialisation.

3. **Getters redéfinis.** `power` et `family` sont des getters de `Monster` que chaque fille remplace. `describe()`, définie une seule fois dans `Monster`, écrit `$family` et `$power` : elle affiche donc « Gluant » pour un `Slime` et « BOSS Dragon » pour un `DragonBoss`, sans avoir été modifiée. Le cas du dragon est le plus parlant : son `power` vaut `20 * phase`, donc la même ligne de code affiche `20` avant la phase 2 et `40` après.

4. **`super.methode()` dans une redéfinition.** `Slime.takeDamage()` et `DragonBoss.takeDamage()` commencent par `super.takeDamage(amount)` : elles laissent le parent faire le travail de base, puis ajoutent leur réaction propre (la division, le passage en phase 2). De même, `Skeleton.describe()` et `DragonBoss.describe()` appellent `super.describe()` puis complètent la ligne.

5. **Polymorphisme.** Les trois boucles de `main()` sont écrites sur `List<Monster>`. Aucune ne contient de test de type. Pourtant chaque monstre s'affiche à sa façon, crie à sa façon et réagit aux dégâts à sa façon. Si vous ajoutez demain une classe `Zombie extends Monster`, il suffit de l'insérer dans la liste : pas une ligne de `main()` à modifier. C'est exactement ce que l'on attend d'une bonne hiérarchie.

6. **Le seul `if` de type... n'existe pas.** Le seul test présent est `if (!m.isAlive)`, qui porte sur l'**état** du monstre, pas sur sa **classe**. C'est une distinction importante : tester un état est normal, tester une classe dans une boucle d'affichage est presque toujours le signe qu'un polymorphisme a été oublié.

---

## Et maintenant ?

Vous savez désormais protéger les données d'une classe, construire une hiérarchie et écrire du code polymorphe. C'est déjà l'essentiel de la POO au quotidien, et vous retrouverez ces trois piliers dans absolument tout le code Flutter que vous lirez.

Il reste cependant plusieurs questions ouvertes que ce chapitre a délibérément laissées de côté :

- comment interdire la création d'un `Character` « générique », qui n'a aucun sens dans le jeu ?
- comment forcer toutes les sous-classes à écrire une méthode `attack()`, sans fournir de version par défaut ?
- comment partager un comportement (par exemple « peut voler ») entre des classes qui n'ont aucun parent commun ?
- comment représenter proprement une liste fermée de valeurs, comme les états `menu`, `enCours`, `pause`, `gameOver` ?

Les réponses s'appellent respectivement `abstract`, `implements`, les **mixins** et les **enums**. C'est le sujet du chapitre suivant, qui complète et termine votre apprentissage de la POO en Dart.

Rendez-vous au chapitre 11 :

[11-PARTIE-1A—POO-AVANCÉE-ABSTRACT-MIXINS-ENUMS.md](11-PARTIE-1A—POO-AVANCÉE-ABSTRACT-MIXINS-ENUMS.md)
