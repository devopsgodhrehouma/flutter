# PARTIE 1A — DART
# CHAPITRE 08 — INTRODUCTION À LA PROGRAMMATION ORIENTÉE OBJET

> **Niveau :** débutant
> **Durée estimée :** 6 h
> **Pré-requis :** chapitre 07 — Les fonctions
> **Ce que vous saurez faire à la fin :** écrire vos propres classes, créer des objets à partir de ces classes, lire et modifier leurs propriétés, appeler leurs méthodes, et faire interagir plusieurs objets entre eux.

---

## 08.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer pourquoi les variables et les fonctions isolées ne suffisent plus ;
- définir ce qu'est une classe ;
- définir ce qu'est un objet ;
- expliquer le mot « instance » ;
- déclarer une propriété (aussi appelée attribut) ;
- déclarer une méthode ;
- écrire votre première classe ;
- instancier un objet ;
- lire la valeur d'une propriété ;
- modifier la valeur d'une propriété ;
- appeler une méthode sur un objet ;
- comprendre et utiliser le mot-clé `this` ;
- créer plusieurs objets d'une même classe ;
- ranger des objets dans une `List` ;
- écrire une méthode qui retourne une valeur ;
- écrire une méthode qui accepte des paramètres ;
- écrire une classe `Player` complète ;
- écrire une classe `Enemy` complète ;
- écrire une classe `Product` hors du contexte du jeu ;
- faire interagir deux objets ;
- expliquer pourquoi une classe est préférable à une `Map`.

---

## 08.0.1 — Avertissement important sur la progression

Ce chapitre est une **première rencontre** avec la programmation orientée objet.

Nous n'utiliserons volontairement PAS encore :

```text
- les constructeurs personnalisés  -> chapitre 09
- l'héritage (extends)             -> chapitre 10
- l'encapsulation (_prive, get, set) -> chapitre 10
- les classes abstraites, mixins, enums -> chapitre 11
- le null safety avancé (?, !, late) -> chapitre 12
```

Conséquence directe et assumée : dans tout ce chapitre, **chaque propriété est déclarée avec une valeur par défaut**, ou bien elle reçoit sa valeur **après** la création de l'objet.

C'est volontairement un peu verbeux. Vous écrirez :

```dart
Player joueur = Player();
joueur.name = 'Alex';
```

Au chapitre 09, vous écrirez la même chose en une seule ligne :

```dart
Player joueur = Player(name: 'Alex');
```

> **Le constructeur sera vu au chapitre 09.** Ne cherchez pas à l'utiliser ici : nous construisons d'abord les fondations.

---

## 08.1 — Pourquoi la POO ?

Reprenons ce que nous savons faire à la fin du chapitre 07.

Nous savons stocker des données dans des variables :

```dart
String nomJoueur = 'Alex';
int vieJoueur = 100;
int scoreJoueur = 0;
```

Et nous savons écrire des fonctions :

```dart
void afficherJoueur(String nom, int vie, int score) {
  print('$nom | vie: $vie | score: $score');
}
```

Cela fonctionne. Mais il y a un problème.

---

## 08.1.1 — Premier problème : les données sont éparpillées

Ajoutons un deuxième joueur :

```dart
String nomJoueur1 = 'Alex';
int vieJoueur1 = 100;
int scoreJoueur1 = 0;

String nomJoueur2 = 'Sophie';
int vieJoueur2 = 100;
int scoreJoueur2 = 0;
```

Puis un troisième, un quatrième...

Représentons la situation :

```text
nomJoueur1 ─┐
vieJoueur1 ─┤  rien ne dit à Dart que
scoreJoueur1─┘  ces 3 variables vont ensemble

nomJoueur2 ─┐
vieJoueur2 ─┤  ni que celles-ci forment
scoreJoueur2─┘  un autre joueur
```

Pour l'ordinateur, ce sont **six variables sans aucun lien entre elles**. Le lien n'existe que dans votre tête et dans le nom des variables.

---

## 08.1.2 — Deuxième problème : les fonctions ne savent pas sur qui elles travaillent

Regardez cette fonction :

```dart
void perdreDesVies(int vie, int degats) {
  vie = vie - degats;
}
```

Elle reçoit une copie du nombre. Elle ne sait pas **de quel joueur** il s'agit. Il faut donc lui repasser sans arrêt toutes les informations :

```dart
afficherJoueur(nomJoueur1, vieJoueur1, scoreJoueur1);
afficherJoueur(nomJoueur2, vieJoueur2, scoreJoueur2);
```

Chaque fois que vous ajoutez une donnée (le niveau, l'arme, l'or...), vous devez modifier **toutes** les fonctions.

---

## 08.1.3 — Troisième problème : rien ne protège la cohérence

Rien n'empêche d'écrire :

```dart
afficherJoueur(nomJoueur1, vieJoueur2, scoreJoueur1);
```

Le programme compile. Il affiche la vie de Sophie sous le nom d'Alex. Le bug est silencieux.

---

## 08.1.4 — La solution : regrouper les données ET les actions

La programmation orientée objet (POO) répond à ce problème avec une idée simple :

> Mettre dans une même boîte les **données** d'une chose et les **actions** que cette chose sait faire.

Un joueur, ce n'est pas trois variables séparées. C'est **une** chose qui possède un nom, une vie, un score, et qui sait attaquer, se soigner, s'afficher.

Voici ce que nous saurons écrire à la fin de ce chapitre :

```dart
Player joueur = Player();
joueur.name = 'Alex';
joueur.showStatus();
```

Une seule variable. Toutes les données à l'intérieur. Toutes les actions attachées.

---

## 08.2 — Qu'est-ce qu'une classe ?

Une **classe** est un plan de fabrication.

Ce n'est pas un joueur. C'est la **description** de ce qu'est un joueur : quelles informations il possède, quelles actions il sait faire.

Une image très courante, et très juste :

```text
      LA CLASSE = LE MOULE                 LES OBJETS = LES PIÈCES

      ┌───────────────────┐
      │      Player       │                 ┌─────────┐  ┌─────────┐
      │  ┌─────────────┐  │                 │  Alex   │  │ Sophie  │
      │  │   name      │  │   ───────>      │ vie 100 │  │ vie 100 │
      │  │   health    │  │   fabrique      │ score 0 │  │ score 0 │
      │  │   score     │  │                 └─────────┘  └─────────┘
      │  └─────────────┘  │
      │  attack()         │                 ┌─────────┐  ┌─────────┐
      │  heal()           │                 │ Samir   │  │ Maria   │
      │  showStatus()     │                 │ vie 100 │  │ vie 100 │
      └───────────────────┘                 │ score 0 │  │ score 0 │
                                            └─────────┘  └─────────┘
      On l'écrit UNE fois.                  On en fabrique AUTANT
      On ne joue pas avec le moule.         qu'on veut.
```

Points essentiels :

- le moule est écrit **une seule fois** ;
- le moule ne contient aucune valeur réelle, seulement la forme ;
- on peut fabriquer un nombre illimité de pièces avec le même moule ;
- si on corrige le moule, toutes les pièces futures sont corrigées.

En Dart, on écrit une classe avec le mot-clé `class` :

```dart
class Player {
  // description du joueur
}
```

> Convention Dart : le nom d'une classe commence toujours par une **majuscule** et s'écrit en `UpperCamelCase` : `Player`, `Enemy`, `Product`, `GameLevel`.

---

## 08.3 — Qu'est-ce qu'un objet ?

Un **objet** est une pièce réellement fabriquée à partir du moule.

C'est une chose concrète, qui existe en mémoire, avec ses propres valeurs.

```text
CLASSE Player                      OBJET
(le concept, le plan)              (une chose réelle en mémoire)

"un joueur possède                 "CE joueur-ci s'appelle Alex,
 un nom, une vie,                   il a 100 points de vie
 un score"                          et 0 point de score"
```

Autres exemples de la vie courante :

| Classe (le concept) | Objets (les choses réelles) |
| --- | --- |
| `Player` | Alex, Sophie, Samir |
| `Enemy` | le gobelin n°1, le gobelin n°2, le boss |
| `Product` | l'épée à 150 pièces, la potion à 25 pièces |
| `Voiture` | ma voiture rouge, la voiture du voisin |

Retenez la phrase suivante :

> La classe est le concept. L'objet est l'exemplaire.

---

## 08.4 — Instance

Le mot **instance** est un synonyme d'objet, mais il insiste sur la relation avec la classe.

- « Alex est un objet. » — on parle de la chose.
- « Alex est une instance de `Player`. » — on précise de quel moule il sort.

Et le verbe associé :

> **Instancier** = fabriquer un objet à partir d'une classe.

En Dart, on instancie en écrivant le nom de la classe suivi de parenthèses :

```dart
Player joueur = Player();
```

Décomposons cette ligne, elle est fondamentale :

```text
Player      joueur       =       Player()
  │            │         │           │
  │            │         │           └─ on FABRIQUE un objet
  │            │         └───────────── on range le résultat
  │            └─────────────────────── nom de la variable
  └──────────────────────────────────── type de la variable
                                        (ici : la classe Player)
```

> **Remarque importante :** en Dart, on **n'écrit pas** `new`. Certains langages (Java, C#, l'ancien Dart) exigent `new Player()`. En Dart moderne, `new` est inutile. Écrivez simplement `Player()`.

---

## 08.5 — Propriété (attribut)

Une **propriété** est une donnée qui appartient à l'objet.

On dit aussi **attribut**, ou **variable d'instance**. Ces trois mots désignent la même chose. Dans cette formation, nous dirons « propriété ».

Exemples de propriétés pour un joueur :

```text
name    -> le nom            (String)
health  -> les points de vie (int)
score   -> le score          (int)
isAlive -> vivant ou non     (bool)
```

En Dart, une propriété se déclare **à l'intérieur** de la classe, exactement comme une variable :

```dart
class Player {
  String name = 'Inconnu';
  int health = 100;
  int score = 0;
  bool isAlive = true;
}
```

> **Pourquoi une valeur par défaut ?** Parce que Dart refuse qu'une propriété non nullable reste sans valeur. Comme nous n'utilisons pas encore de constructeur (chapitre 09), la valeur par défaut est notre solution. Sans elle, le programme ne compile pas.

---

## 08.6 — Méthode

Une **méthode** est une action que l'objet sait faire.

Techniquement, c'est une fonction écrite **à l'intérieur** d'une classe.

| Fonction | Méthode |
| --- | --- |
| Écrite en dehors de toute classe | Écrite à l'intérieur d'une classe |
| Ne connaît que ses paramètres | Connaît aussi les propriétés de l'objet |
| Appelée par son nom : `saluer()` | Appelée sur un objet : `joueur.saluer()` |

Exemple de méthode :

```dart
class Player {
  String name = 'Inconnu';
  int health = 100;

  void showStatus() {
    print('$name a $health points de vie.');
  }
}
```

Observez la ligne `print('$name a $health points de vie.');` : la méthode utilise `name` et `health` **sans qu'on les lui passe en paramètre**. C'est exactement ce que les fonctions isolées ne savaient pas faire.

---

## 08.7 — Créer une première classe

Passons à un programme complet et exécutable.

En Dart, une classe se déclare **en dehors** de `main()`, généralement en dessous.

```dart
void main() {
  print('Programme lance.');
}

class Player {
  String name = 'Inconnu';
  int health = 100;
  int score = 0;
}
```

**Résultat :**

```text
Programme lance.
```

La classe est déclarée, mais aucun objet n'a encore été fabriqué. C'est normal : écrire un moule ne fabrique pas de pièce.

---

## 08.7.1 — Où placer la classe dans le fichier ?

Les deux écritures suivantes sont valides :

```dart
// Classe après main
void main() {
  // ...
}

class Player {
  String name = 'Inconnu';
}
```

```dart
// Classe avant main
class Player {
  String name = 'Inconnu';
}

void main() {
  // ...
}
```

Dart ne lit pas le fichier de haut en bas comme une recette : il connaît tout le fichier avant de l'exécuter.

Ce qui est **interdit**, c'est de déclarer une classe **dans** `main()` :

```dart
void main() {
  class Player {   // ERREUR de compilation
    String name = 'Inconnu';
  }
}
```

Dans cette formation, nous placerons toujours les classes **après** `main()`, pour que le programme principal reste visible en premier.

---

## 08.8 — Instancier un objet

Fabriquons maintenant un joueur.

```dart
void main() {
  Player joueur = Player();

  print(joueur);
}

class Player {
  String name = 'Inconnu';
  int health = 100;
  int score = 0;
}
```

**Résultat :**

```text
Instance of 'Player'
```

L'objet existe. `print` affiche `Instance of 'Player'` parce que Dart ne sait pas encore comment décrire votre objet en texte. Ce n'est pas une erreur.

---

## 08.8.1 — Ce qui se passe en mémoire

Voici le deuxième schéma à retenir : à quoi ressemble un objet en mémoire.

```text
   Variable                       MÉMOIRE (le tas)
                            ┌──────────────────────────┐
   joueur ────référence────>│  OBJET  #1  : Player     │
                            ├──────────────────────────┤
                            │  name    = 'Inconnu'     │
                            │  health  = 100           │
                            │  score   = 0             │
                            ├──────────────────────────┤
                            │  showStatus()            │
                            │  attack()                │
                            └──────────────────────────┘

   La variable `joueur` ne CONTIENT pas l'objet.
   Elle contient l'ADRESSE de l'objet : on dit qu'elle le "référence".
```

Retenez :

- l'objet vit dans la mémoire ;
- la variable pointe vers lui ;
- deux variables peuvent pointer vers le **même** objet (nous y reviendrons en 08.22).

---

## 08.8.2 — `var` fonctionne aussi

Dart peut deviner le type :

```dart
void main() {
  var joueur = Player();

  print(joueur.health);
}

class Player {
  String name = 'Inconnu';
  int health = 100;
  int score = 0;
}
```

**Résultat :**

```text
100
```

Ici, `var joueur` est automatiquement de type `Player`. Dans ce chapitre, nous écrirons le type en toutes lettres (`Player joueur = Player();`) pour que tout reste explicite.

---

## 08.9 — Accéder aux propriétés

Pour lire une propriété d'un objet, on utilise le **point** :

```text
objet.propriete
```

Le point se lit « de » : `joueur.health` se lit « la vie du joueur ».

```dart
void main() {
  Player joueur = Player();

  print(joueur.name);
  print(joueur.health);
  print(joueur.score);
}

class Player {
  String name = 'Inconnu';
  int health = 100;
  int score = 0;
}
```

**Résultat :**

```text
Inconnu
100
0
```

Les valeurs affichées sont les valeurs par défaut écrites dans la classe. C'est logique : nous n'avons encore rien changé.

---

## 08.9.1 — Utiliser une propriété dans une interpolation

Comme n'importe quelle valeur, une propriété s'insère dans une chaîne.

Attention à une règle de syntaxe :

```dart
void main() {
  Player joueur = Player();

  print('Nom : $joueur.name');
  print('Nom : ${joueur.name}');
}

class Player {
  String name = 'Inconnu';
  int health = 100;
}
```

**Résultat :**

```text
Nom : Instance of 'Player'.name
Nom : Inconnu
```

> **Règle :** dès qu'il y a un point dans une interpolation, il faut les accolades : `${joueur.name}`. Sans accolades, Dart s'arrête au nom de la variable.

---

## 08.10 — Modifier les propriétés

Le point sert aussi à **écrire** :

```text
objet.propriete = nouvelleValeur;
```

```dart
void main() {
  Player joueur = Player();

  joueur.name = 'Alex';
  joueur.health = 80;
  joueur.score = 250;

  print(joueur.name);
  print(joueur.health);
  print(joueur.score);
}

class Player {
  String name = 'Inconnu';
  int health = 100;
  int score = 0;
}
```

**Résultat :**

```text
Alex
80
250
```

C'est ainsi que nous donnerons une identité à nos objets dans tout ce chapitre : on instancie, puis on affecte.

```text
ETAPE 1 : Player joueur = Player();   ->  name = 'Inconnu', health = 100
ETAPE 2 : joueur.name = 'Alex';       ->  name = 'Alex',    health = 100
ETAPE 3 : joueur.health = 80;         ->  name = 'Alex',    health = 80
```

> Au chapitre 09, le constructeur permettra de faire les trois étapes en une seule ligne.

---

## 08.10.1 — Modifier à partir de la valeur actuelle

Rien n'interdit de calculer à partir de l'ancienne valeur :

```dart
void main() {
  Player joueur = Player();
  joueur.name = 'Alex';

  print('Vie de depart : ${joueur.health}');

  joueur.health = joueur.health - 30;
  print('Apres un coup : ${joueur.health}');

  joueur.health -= 20;
  print('Apres un autre coup : ${joueur.health}');

  joueur.score += 100;
  print('Score : ${joueur.score}');
}

class Player {
  String name = 'Inconnu';
  int health = 100;
  int score = 0;
}
```

**Résultat :**

```text
Vie de depart : 100
Apres un coup : 70
Apres un autre coup : 50
Score : 100
```

Tous les opérateurs du chapitre 03 (`+=`, `-=`, `++`...) fonctionnent sur les propriétés.

---

## 08.11 — Appeler une méthode

Appeler une méthode, c'est demander à l'objet d'exécuter une action.

La syntaxe utilise le même point, suivi de parenthèses :

```text
objet.methode()
```

```dart
void main() {
  Player joueur = Player();
  joueur.name = 'Alex';
  joueur.health = 90;

  joueur.showStatus();
}

class Player {
  String name = 'Inconnu';
  int health = 100;
  int score = 0;

  void showStatus() {
    print('$name | vie : $health | score : $score');
  }
}
```

**Résultat :**

```text
Alex | vie : 90 | score : 0
```

Remarquez le gain : nous n'avons rien passé à `showStatus()`. La méthode a lu les propriétés de **son** objet.

---

## 08.11.1 — Les parenthèses sont obligatoires

Une erreur très fréquente chez le débutant :

```dart
void main() {
  Player joueur = Player();

  joueur.showStatus;    // ligne sans effet visible attendu
  joueur.showStatus();  // appel correct
}

class Player {
  String name = 'Alex';

  void showStatus() {
    print('Statut de $name');
  }
}
```

**Résultat :**

```text
Statut de Alex
```

Une seule ligne s'est affichée. `joueur.showStatus` sans parenthèses ne **lance** pas la méthode : cela désigne la méthode elle-même. Pour exécuter, il faut `()`.

---

## 08.11.2 — Une méthode peut en appeler une autre

À l'intérieur d'une classe, une méthode appelle une autre méthode de la même classe **sans point** :

```dart
void main() {
  Player joueur = Player();
  joueur.name = 'Alex';

  joueur.startTurn();
}

class Player {
  String name = 'Inconnu';
  int health = 100;
  int energy = 50;

  void showStatus() {
    print('$name | vie : $health | energie : $energy');
  }

  void startTurn() {
    print('--- Nouveau tour ---');
    showStatus();
  }
}
```

**Résultat :**

```text
--- Nouveau tour ---
Alex | vie : 100 | energie : 50
```

---

## 08.12 — Le mot-clé `this`

`this` désigne **l'objet courant**, c'est-à-dire l'objet sur lequel la méthode est en train de s'exécuter.

Ces deux méthodes font exactement la même chose :

```dart
void showStatus() {
  print('$name a $health points de vie.');
}
```

```dart
void showStatus() {
  print('${this.name} a ${this.health} points de vie.');
}
```

Programme complet :

```dart
void main() {
  Player joueur = Player();
  joueur.name = 'Alex';

  joueur.showStatusCourt();
  joueur.showStatusExplicite();
}

class Player {
  String name = 'Inconnu';
  int health = 100;

  void showStatusCourt() {
    print('$name a $health points de vie.');
  }

  void showStatusExplicite() {
    print('${this.name} a ${this.health} points de vie.');
  }
}
```

**Résultat :**

```text
Alex a 100 points de vie.
Alex a 100 points de vie.
```

---

## 08.12.1 — Quand `this` est-il nécessaire ?

En Dart, `this` est **facultatif** tant qu'il n'y a pas d'ambiguïté. Et l'écrire partout est considéré comme du bruit.

`this` devient **indispensable** quand un paramètre porte le même nom qu'une propriété :

```dart
void main() {
  Player joueur = Player();

  joueur.rename('Sophie');
  print(joueur.name);
}

class Player {
  String name = 'Inconnu';

  void rename(String name) {
    this.name = name;
  }
}
```

**Résultat :**

```text
Sophie
```

Lecture de la ligne `this.name = name;` :

```text
this.name  =  name;
    │           │
    │           └─ le PARAMÈTRE de la méthode ('Sophie')
    └───────────── la PROPRIÉTÉ de l'objet
```

Sans `this`, Dart comprendrait `name = name;` : le paramètre s'affecte à lui-même, et la propriété ne change jamais.

Démonstration de l'erreur :

```dart
void main() {
  Player joueur = Player();

  joueur.renameBuggy('Sophie');
  print(joueur.name);
}

class Player {
  String name = 'Inconnu';

  void renameBuggy(String name) {
    name = name; // ne fait rien d'utile
  }
}
```

**Résultat :**

```text
Inconnu
```

> **À retenir :** utilisez `this` uniquement quand il y a conflit de noms. Ailleurs, écrivez directement le nom de la propriété.

---

## 08.13 — Plusieurs objets d'une même classe

C'est ici que la POO prend tout son sens.

```dart
void main() {
  Player joueur1 = Player();
  joueur1.name = 'Alex';
  joueur1.health = 100;
  joueur1.score = 250;

  Player joueur2 = Player();
  joueur2.name = 'Sophie';
  joueur2.health = 75;
  joueur2.score = 480;

  Player joueur3 = Player();
  joueur3.name = 'Samir';
  joueur3.health = 40;
  joueur3.score = 120;

  joueur1.showStatus();
  joueur2.showStatus();
  joueur3.showStatus();
}

class Player {
  String name = 'Inconnu';
  int health = 100;
  int score = 0;

  void showStatus() {
    print('$name | vie : $health | score : $score');
  }
}
```

**Résultat :**

```text
Alex | vie : 100 | score : 250
Sophie | vie : 75 | score : 480
Samir | vie : 40 | score : 120
```

Une seule classe. Trois objets. Chacun possède **sa propre copie** des propriétés.

```text
                    class Player  (le moule, écrit 1 fois)
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
   ┌─────────┐          ┌─────────┐          ┌─────────┐
   │ joueur1 │          │ joueur2 │          │ joueur3 │
   ├─────────┤          ├─────────┤          ├─────────┤
   │ Alex    │          │ Sophie  │          │ Samir   │
   │ 100     │          │ 75      │          │ 40      │
   │ 250     │          │ 480     │          │ 120     │
   └─────────┘          └─────────┘          └─────────┘
```

---

## 08.13.1 — Les objets sont indépendants

Modifier un objet ne touche pas les autres :

```dart
void main() {
  Player joueur1 = Player();
  joueur1.name = 'Alex';

  Player joueur2 = Player();
  joueur2.name = 'Sophie';

  joueur1.health = 10;

  print('${joueur1.name} : ${joueur1.health}');
  print('${joueur2.name} : ${joueur2.health}');
}

class Player {
  String name = 'Inconnu';
  int health = 100;
}
```

**Résultat :**

```text
Alex : 10
Sophie : 100
```

Sophie garde ses 100 points de vie. Chaque objet a sa mémoire propre.

---

## 08.14 — Objets dans une `List`

Au chapitre 06, nous avons rangé des `String` et des `int` dans des listes. Une liste peut contenir **n'importe quel type**, y compris nos propres classes.

Le type s'écrit `List<Player>` : une liste de joueurs.

```dart
void main() {
  Player joueur1 = Player();
  joueur1.name = 'Alex';
  joueur1.score = 250;

  Player joueur2 = Player();
  joueur2.name = 'Sophie';
  joueur2.score = 480;

  Player joueur3 = Player();
  joueur3.name = 'Samir';
  joueur3.score = 120;

  List<Player> joueurs = [joueur1, joueur2, joueur3];

  print(joueurs.length);
}

class Player {
  String name = 'Inconnu';
  int health = 100;
  int score = 0;
}
```

**Résultat :**

```text
3
```

---

## 08.14.1 — Accéder à un objet de la liste

L'index fonctionne exactement comme au chapitre 06.

```dart
void main() {
  Player joueur1 = Player();
  joueur1.name = 'Alex';

  Player joueur2 = Player();
  joueur2.name = 'Sophie';

  List<Player> joueurs = [joueur1, joueur2];

  print(joueurs[0].name);
  print(joueurs[1].name);
}

class Player {
  String name = 'Inconnu';
  int health = 100;
}
```

**Résultat :**

```text
Alex
Sophie
```

Lecture de `joueurs[1].name` :

```text
joueurs[1]  ->  l'objet Player en position 1
       .name ->  la propriété name de cet objet
```

---

## 08.14.2 — Parcourir une liste d'objets avec `for-in`

```dart
void main() {
  Player joueur1 = Player();
  joueur1.name = 'Alex';
  joueur1.score = 250;

  Player joueur2 = Player();
  joueur2.name = 'Sophie';
  joueur2.score = 480;

  Player joueur3 = Player();
  joueur3.name = 'Samir';
  joueur3.score = 120;

  List<Player> joueurs = [joueur1, joueur2, joueur3];

  for (Player j in joueurs) {
    j.showStatus();
  }
}

class Player {
  String name = 'Inconnu';
  int health = 100;
  int score = 0;

  void showStatus() {
    print('$name | vie : $health | score : $score');
  }
}
```

**Résultat :**

```text
Alex | vie : 100 | score : 250
Sophie | vie : 100 | score : 480
Samir | vie : 100 | score : 120
```

---

## 08.14.3 — Créer les objets directement dans la liste

On peut aussi instancier à l'intérieur des crochets, puis configurer avec une boucle indexée :

```dart
void main() {
  List<Player> joueurs = [
    Player(),
    Player(),
    Player(),
  ];

  List<String> noms = ['Alex', 'Sophie', 'Samir'];

  for (int i = 0; i < joueurs.length; i++) {
    joueurs[i].name = noms[i];
    joueurs[i].score = (i + 1) * 100;
  }

  for (Player j in joueurs) {
    print('${j.name} -> ${j.score}');
  }
}

class Player {
  String name = 'Inconnu';
  int health = 100;
  int score = 0;
}
```

**Résultat :**

```text
Alex -> 100
Sophie -> 200
Samir -> 300
```

---

## 08.14.4 — Ajouter un objet à la liste

`add()` fonctionne comme au chapitre 06 :

```dart
void main() {
  List<Player> joueurs = [];

  Player nouveau = Player();
  nouveau.name = 'Maria';
  joueurs.add(nouveau);

  Player autre = Player();
  autre.name = 'Karim';
  joueurs.add(autre);

  print('Nombre de joueurs : ${joueurs.length}');

  for (Player j in joueurs) {
    print(j.name);
  }
}

class Player {
  String name = 'Inconnu';
  int health = 100;
}
```

**Résultat :**

```text
Nombre de joueurs : 2
Maria
Karim
```

---

## 08.15 — Méthode qui retourne une valeur

Jusqu'ici toutes nos méthodes étaient `void` : elles affichaient quelque chose mais ne renvoyaient rien.

Comme une fonction (chapitre 07), une méthode peut **retourner** une valeur. On remplace alors `void` par le type retourné.

```dart
void main() {
  Player joueur = Player();
  joueur.name = 'Alex';
  joueur.health = 42;

  int vie = joueur.getHealth();
  print('Vie recuperee : $vie');
}

class Player {
  String name = 'Inconnu';
  int health = 100;

  int getHealth() {
    return health;
  }
}
```

**Résultat :**

```text
Vie recuperee : 42
```

---

## 08.15.1 — Retourner un booléen

Très utile pour poser une question à l'objet :

```dart
void main() {
  Player joueur = Player();
  joueur.name = 'Alex';
  joueur.health = 0;

  if (joueur.isDead()) {
    print('${joueur.name} est mort.');
  } else {
    print('${joueur.name} est encore en vie.');
  }
}

class Player {
  String name = 'Inconnu';
  int health = 100;

  bool isDead() {
    return health <= 0;
  }
}
```

**Résultat :**

```text
Alex est mort.
```

> Notez `return health <= 0;` : la comparaison produit déjà un `bool`. Écrire `if (health <= 0) { return true; } else { return false; }` fonctionnerait, mais c'est inutilement long.

---

## 08.15.2 — Retourner une chaîne construite

```dart
void main() {
  Player joueur = Player();
  joueur.name = 'Sophie';
  joueur.health = 75;
  joueur.score = 480;

  String ligne = joueur.describe();
  print(ligne);
  print(ligne.toUpperCase());
}

class Player {
  String name = 'Inconnu';
  int health = 100;
  int score = 0;

  String describe() {
    return '$name (vie : $health, score : $score)';
  }
}
```

**Résultat :**

```text
Sophie (vie : 75, score : 480)
SOPHIE (VIE : 75, SCORE : 480)
```

L'avantage d'un `return` par rapport à un `print` : la valeur peut être réutilisée, transformée, stockée. Un `print` ne fait qu'afficher.

---

## 08.15.3 — Une méthode peut retourner un calcul

```dart
void main() {
  Player joueur = Player();
  joueur.name = 'Alex';
  joueur.health = 40;
  joueur.maxHealth = 160;

  print('Vie en pourcentage : ${joueur.healthPercent()} %');
}

class Player {
  String name = 'Inconnu';
  int health = 100;
  int maxHealth = 100;

  double healthPercent() {
    return health / maxHealth * 100;
  }
}
```

**Résultat :**

```text
Vie en pourcentage : 25.0 %
```

Rappel du chapitre 03 : la division `/` entre deux `int` produit un `double`, d'où l'affichage `25.0`.

---

## 08.16 — Méthode avec paramètres

Une méthode peut recevoir des informations de l'extérieur, exactement comme une fonction.

```dart
void main() {
  Player joueur = Player();
  joueur.name = 'Alex';

  joueur.showStatus();

  joueur.takeDamage(30);
  joueur.showStatus();

  joueur.takeDamage(45);
  joueur.showStatus();
}

class Player {
  String name = 'Inconnu';
  int health = 100;

  void takeDamage(int amount) {
    health = health - amount;
    print('$name subit $amount degats.');
  }

  void showStatus() {
    print('$name | vie : $health');
  }
}
```

**Résultat :**

```text
Alex | vie : 100
Alex subit 30 degats.
Alex | vie : 70
Alex subit 45 degats.
Alex | vie : 25
```

Le paramètre `amount` vient de l'appelant. La propriété `health` appartient à l'objet. La méthode combine les deux.

---

## 08.16.1 — Plusieurs paramètres

```dart
void main() {
  Player joueur = Player();
  joueur.name = 'Alex';

  joueur.gainReward(150, 20);
  joueur.showStatus();
}

class Player {
  String name = 'Inconnu';
  int health = 100;
  int gold = 0;
  int score = 0;

  void gainReward(int points, int pieces) {
    score = score + points;
    gold = gold + pieces;
    print('$name gagne $points points et $pieces pieces d\'or.');
  }

  void showStatus() {
    print('$name | score : $score | or : $gold');
  }
}
```

**Résultat :**

```text
Alex gagne 150 points et 20 pieces d'or.
Alex | score : 150 | or : 20
```

---

## 08.16.2 — Paramètre nommé avec valeur par défaut

Comme au chapitre 07, on peut utiliser des paramètres nommés entre accolades :

```dart
void main() {
  Player joueur = Player();
  joueur.name = 'Alex';
  joueur.health = 50;

  joueur.heal();
  joueur.heal(amount: 25);
}

class Player {
  String name = 'Inconnu';
  int health = 100;

  void heal({int amount = 10}) {
    health = health + amount;
    print('$name recupere $amount PV -> vie : $health');
  }
}
```

**Résultat :**

```text
Alex recupere 10 PV -> vie : 60
Alex recupere 25 PV -> vie : 85
```

---

## 08.16.3 — Protéger la méthode avec une condition

Une méthode bien écrite vérifie ce qu'elle reçoit :

```dart
void main() {
  Player joueur = Player();
  joueur.name = 'Alex';
  joueur.health = 20;

  joueur.takeDamage(5);
  joueur.takeDamage(100);
  joueur.takeDamage(-50);
}

class Player {
  String name = 'Inconnu';
  int health = 100;

  void takeDamage(int amount) {
    if (amount <= 0) {
      print('Degats invalides : $amount');
      return;
    }

    health = health - amount;

    if (health < 0) {
      health = 0;
    }

    print('$name | vie : $health');
  }
}
```

**Résultat :**

```text
Alex | vie : 15
Alex | vie : 0
Degats invalides : -50
```

Sans la ligne `if (health < 0) { health = 0; }`, le joueur afficherait `-85` points de vie. Une classe doit garder ses données cohérentes.

---

## 08.17 — Classe `Player` complète

Rassemblons tout ce que nous savons dans une classe `Player` utilisable.

```dart
void main() {
  Player joueur = Player();
  joueur.name = 'Alex';
  joueur.health = 100;
  joueur.maxHealth = 100;
  joueur.attackPower = 18;

  joueur.showStatus();

  joueur.takeDamage(35);
  joueur.takeDamage(40);
  joueur.heal(15);
  joueur.gainScore(200);
  joueur.levelUp();

  joueur.showStatus();

  print('Vivant ? ${joueur.isAlive()}');
  print('Resume : ${joueur.describe()}');
}

class Player {
  String name = 'Inconnu';
  int health = 100;
  int maxHealth = 100;
  int attackPower = 10;
  int level = 1;
  int score = 0;

  void showStatus() {
    print('--- $name ---');
    print('Niveau  : $level');
    print('Vie     : $health / $maxHealth');
    print('Attaque : $attackPower');
    print('Score   : $score');
  }

  void takeDamage(int amount) {
    health = health - amount;
    if (health < 0) {
      health = 0;
    }
    print('$name subit $amount degats (vie : $health).');
  }

  void heal(int amount) {
    health = health + amount;
    if (health > maxHealth) {
      health = maxHealth;
    }
    print('$name se soigne de $amount (vie : $health).');
  }

  void gainScore(int points) {
    score = score + points;
    print('$name gagne $points points (score : $score).');
  }

  void levelUp() {
    level = level + 1;
    maxHealth = maxHealth + 20;
    health = maxHealth;
    attackPower = attackPower + 5;
    print('$name passe niveau $level !');
  }

  bool isAlive() {
    return health > 0;
  }

  String describe() {
    return '$name (niv. $level) - $health/$maxHealth PV';
  }
}
```

**Résultat :**

```text
--- Alex ---
Niveau  : 1
Vie     : 100 / 100
Attaque : 18
Score   : 0
Alex subit 35 degats (vie : 65).
Alex subit 40 degats (vie : 25).
Alex se soigne de 15 (vie : 40).
Alex gagne 200 points (score : 200).
Alex passe niveau 2 !
--- Alex ---
Niveau  : 2
Vie     : 120 / 120
Attaque : 23
Score   : 200
Vivant ? true
Resume : Alex (niv. 2) - 120/120 PV
```

Observez la méthode `levelUp()` : elle modifie quatre propriétés d'un coup. Depuis `main()`, une seule ligne suffit. C'est exactement le bénéfice de la POO : **regrouper la logique là où sont les données**.

---

## 08.18 — Classe `Enemy` complète

Un ennemi ressemble à un joueur, mais ce n'est pas la même chose : il possède un type, une récompense, et il ne gagne pas de niveaux.

```dart
void main() {
  Enemy gobelin = Enemy();
  gobelin.name = 'Gobelin';
  gobelin.type = 'Gobelin';
  gobelin.health = 40;
  gobelin.attackPower = 8;
  gobelin.rewardGold = 12;
  gobelin.rewardScore = 50;

  gobelin.showStatus();

  gobelin.takeDamage(25);
  gobelin.showStatus();

  gobelin.takeDamage(25);
  gobelin.showStatus();

  print('Vaincu ? ${gobelin.isDefeated()}');
}

class Enemy {
  String name = 'Ennemi';
  String type = 'Basique';
  int health = 30;
  int attackPower = 5;
  int rewardGold = 5;
  int rewardScore = 10;
  bool isBoss = false;

  void showStatus() {
    print('[$type] $name | vie : $health | attaque : $attackPower');
  }

  void takeDamage(int amount) {
    health = health - amount;
    if (health < 0) {
      health = 0;
    }
  }

  bool isDefeated() {
    return health <= 0;
  }

  int giveReward() {
    if (isBoss) {
      return rewardScore * 3;
    }
    return rewardScore;
  }
}
```

**Résultat :**

```text
[Gobelin] Gobelin | vie : 40 | attaque : 8
[Gobelin] Gobelin | vie : 15 | attaque : 8
[Gobelin] Gobelin | vie : 0 | attaque : 8
Vaincu ? true
```

---

## 08.18.1 — Créer plusieurs ennemis d'un coup

```dart
void main() {
  List<Enemy> ennemis = [];

  List<String> types = ['Gobelin', 'Squelette', 'Orc'];

  for (int i = 0; i < types.length; i++) {
    Enemy e = Enemy();
    e.name = '${types[i]} #${i + 1}';
    e.type = types[i];
    e.health = 30 + i * 20;
    e.attackPower = 5 + i * 3;
    ennemis.add(e);
  }

  Enemy boss = Enemy();
  boss.name = 'Roi Gobelin';
  boss.type = 'Boss';
  boss.health = 200;
  boss.attackPower = 25;
  boss.isBoss = true;
  boss.rewardScore = 100;
  ennemis.add(boss);

  for (Enemy e in ennemis) {
    e.showStatus();
  }

  print('Total ennemis : ${ennemis.length}');
}

class Enemy {
  String name = 'Ennemi';
  String type = 'Basique';
  int health = 30;
  int attackPower = 5;
  int rewardGold = 5;
  int rewardScore = 10;
  bool isBoss = false;

  void showStatus() {
    print('[$type] $name | vie : $health | attaque : $attackPower');
  }

  bool isDefeated() {
    return health <= 0;
  }
}
```

**Résultat :**

```text
[Gobelin] Gobelin #1 | vie : 30 | attaque : 5
[Squelette] Squelette #2 | vie : 50 | attaque : 8
[Orc] Orc #3 | vie : 70 | attaque : 11
[Boss] Roi Gobelin | vie : 200 | attaque : 25
Total ennemis : 4
```

---

## 08.19 — Classe `Product` (exemple hors jeu)

La POO ne sert pas qu'aux jeux. Modélisons un produit d'une boutique — c'est typiquement ce que vous ferez dans une application Flutter e-commerce.

```dart
void main() {
  Product epee = Product();
  epee.name = 'Epee en fer';
  epee.price = 150.0;
  epee.stock = 3;
  epee.category = 'Arme';

  Product potion = Product();
  potion.name = 'Potion de soin';
  potion.price = 25.5;
  potion.stock = 0;
  potion.category = 'Consommable';

  epee.showCard();
  potion.showCard();

  print('Prix TTC de l\'epee : ${epee.priceWithTax()} pieces');
  print('Total pour 2 epees : ${epee.totalFor(2)} pieces');

  epee.sell(1);
  epee.showCard();

  potion.sell(1);
}

class Product {
  String name = 'Produit';
  double price = 0.0;
  int stock = 0;
  String category = 'Divers';
  double taxRate = 0.20;

  void showCard() {
    print('--- $name ---');
    print('Categorie : $category');
    print('Prix      : $price');
    print('Stock     : $stock');
    print('Dispo     : ${isAvailable()}');
  }

  bool isAvailable() {
    return stock > 0;
  }

  double priceWithTax() {
    return price + price * taxRate;
  }

  double totalFor(int quantity) {
    return price * quantity;
  }

  void sell(int quantity) {
    if (quantity > stock) {
      print('Stock insuffisant pour $name.');
      return;
    }
    stock = stock - quantity;
    print('Vente de $quantity x $name (stock restant : $stock).');
  }
}
```

**Résultat :**

```text
--- Epee en fer ---
Categorie : Arme
Prix      : 150.0
Stock     : 3
Dispo     : true
--- Potion de soin ---
Categorie : Consommable
Prix      : 25.5
Stock     : 0
Dispo     : false
Prix TTC de l'epee : 180.0 pieces
Total pour 2 epees : 300.0 pieces
Vente de 1 x Epee en fer (stock restant : 2).
--- Epee en fer ---
Categorie : Arme
Prix      : 150.0
Stock     : 2
Dispo     : true
Stock insuffisant pour Potion de soin.
```

Retenez la structure : **une classe = un concept du monde réel**. Produit, utilisateur, commande, message, article de blog... C'est ainsi que sont construites toutes les applications Flutter.

---

## 08.20 — Faire interagir deux objets

Voici le point d'orgue du chapitre : un objet peut recevoir un autre objet en paramètre.

Le joueur attaque l'ennemi. La méthode `attack()` de `Player` reçoit un `Enemy` et modifie sa vie.

```dart
void main() {
  Player joueur = Player();
  joueur.name = 'Alex';
  joueur.attackPower = 15;

  Enemy gobelin = Enemy();
  gobelin.name = 'Gobelin';
  gobelin.health = 40;
  gobelin.attackPower = 7;

  print('=== Debut du combat ===');
  joueur.showStatus();
  gobelin.showStatus();

  print('=== Tour 1 ===');
  joueur.attack(gobelin);
  gobelin.attack(joueur);

  print('=== Tour 2 ===');
  joueur.attack(gobelin);
  gobelin.attack(joueur);

  print('=== Tour 3 ===');
  joueur.attack(gobelin);

  print('=== Fin ===');
  joueur.showStatus();
  gobelin.showStatus();
}

class Player {
  String name = 'Heros';
  int health = 100;
  int attackPower = 10;
  int score = 0;

  void showStatus() {
    print('$name | vie : $health | score : $score');
  }

  void takeDamage(int amount) {
    health = health - amount;
    if (health < 0) {
      health = 0;
    }
  }

  void attack(Enemy target) {
    print('$name attaque ${target.name} pour $attackPower degats.');
    target.takeDamage(attackPower);

    if (target.isDefeated()) {
      print('${target.name} est vaincu !');
      score = score + target.rewardScore;
      print('$name gagne ${target.rewardScore} points.');
    }
  }
}

class Enemy {
  String name = 'Ennemi';
  int health = 30;
  int attackPower = 5;
  int rewardScore = 20;

  void showStatus() {
    print('$name | vie : $health');
  }

  void takeDamage(int amount) {
    health = health - amount;
    if (health < 0) {
      health = 0;
    }
  }

  bool isDefeated() {
    return health <= 0;
  }

  void attack(Player target) {
    if (isDefeated()) {
      return;
    }
    print('$name attaque ${target.name} pour $attackPower degats.');
    target.takeDamage(attackPower);
  }
}
```

**Résultat :**

```text
=== Debut du combat ===
Alex | vie : 100 | score : 0
Gobelin | vie : 40
=== Tour 1 ===
Alex attaque Gobelin pour 15 degats.
Gobelin attaque Alex pour 7 degats.
=== Tour 2 ===
Alex attaque Gobelin pour 15 degats.
Gobelin attaque Alex pour 7 degats.
=== Tour 3 ===
Alex attaque Gobelin pour 15 degats.
Gobelin est vaincu !
Alex gagne 20 points.
=== Fin ===
Alex | vie : 86 | score : 20
Gobelin | vie : 0
```

---

## 08.20.1 — Comprendre l'interaction

Détaillons la signature :

```dart
void attack(Enemy target) {
  target.takeDamage(attackPower);
}
```

```text
void attack( Enemy target )
              │      │
              │      └─ le nom du paramètre : l'objet reçu
              └──────── son TYPE : ce doit être un Enemy

Dans le corps :
  attackPower          -> propriété de CET objet   (le joueur)
  target.takeDamage()  -> méthode de l'AUTRE objet (l'ennemi)
```

Schéma de l'échange :

```text
   joueur (Player)                        gobelin (Enemy)
   ┌───────────────┐                      ┌───────────────┐
   │ name  = Alex  │                      │ name = Gobelin│
   │ health= 100   │   attack(gobelin)    │ health = 40   │
   │ power = 15    │ ───────────────────> │               │
   └───────────────┘   appelle            └───────────────┘
                       gobelin.takeDamage(15)
                                                  │
                                                  v
                                          health devient 25
```

Point clé : `attack()` reçoit **l'objet lui-même**, pas une copie. Les modifications faites sur `target` sont donc bien visibles depuis `main()`.

---

## 08.20.2 — Combat automatique avec une boucle

Combinons la POO et les boucles du chapitre 05 :

```dart
void main() {
  Player joueur = Player();
  joueur.name = 'Alex';
  joueur.health = 100;
  joueur.attackPower = 12;

  Enemy boss = Enemy();
  boss.name = 'Roi Gobelin';
  boss.health = 90;
  boss.attackPower = 14;

  int tour = 1;

  while (joueur.health > 0 && boss.health > 0) {
    print('--- Tour $tour ---');
    joueur.attack(boss);
    if (boss.health > 0) {
      boss.attack(joueur);
    }
    print('${joueur.name} : ${joueur.health} PV | ${boss.name} : ${boss.health} PV');
    tour = tour + 1;
  }

  if (joueur.health > 0) {
    print('Victoire de ${joueur.name} !');
  } else {
    print('Defaite de ${joueur.name}...');
  }
}

class Player {
  String name = 'Heros';
  int health = 100;
  int attackPower = 10;

  void takeDamage(int amount) {
    health = health - amount;
    if (health < 0) {
      health = 0;
    }
  }

  void attack(Enemy target) {
    target.takeDamage(attackPower);
  }
}

class Enemy {
  String name = 'Ennemi';
  int health = 30;
  int attackPower = 5;

  void takeDamage(int amount) {
    health = health - amount;
    if (health < 0) {
      health = 0;
    }
  }

  void attack(Player target) {
    target.takeDamage(attackPower);
  }
}
```

**Résultat :**

```text
--- Tour 1 ---
Alex : 86 PV | Roi Gobelin : 78 PV
--- Tour 2 ---
Alex : 72 PV | Roi Gobelin : 66 PV
--- Tour 3 ---
Alex : 58 PV | Roi Gobelin : 54 PV
--- Tour 4 ---
Alex : 44 PV | Roi Gobelin : 42 PV
--- Tour 5 ---
Alex : 30 PV | Roi Gobelin : 30 PV
--- Tour 6 ---
Alex : 16 PV | Roi Gobelin : 18 PV
--- Tour 7 ---
Alex : 2 PV | Roi Gobelin : 6 PV
--- Tour 8 ---
Alex : 2 PV | Roi Gobelin : 0 PV
Victoire de Alex !
```

Au tour 8, le boss tombe à 0 PV avant de pouvoir riposter : la condition `if (boss.health > 0)` l'en empêche.

---

## 08.21 — Objet vs `Map` : pourquoi la classe est meilleure

Au chapitre 06, nous représentions un joueur ainsi :

```dart
Map<String, dynamic> joueur = {
  'name': 'Alex',
  'health': 100,
  'score': 250,
};
```

C'est possible. Mais comparons honnêtement.

---

## 08.21.1 — La faute de frappe n'est pas détectée

```dart
void main() {
  Map<String, dynamic> joueur = {
    'name': 'Alex',
    'health': 100,
  };

  print(joueur['helth']);
}
```

**Résultat :**

```text
null
```

Le programme compile, s'exécute, et affiche `null`. Le bug se révélera plus tard, ailleurs, difficilement.

Avec une classe :

```dart
void main() {
  Player joueur = Player();
  print(joueur.helth); // ERREUR détectée avant même l'exécution
}

class Player {
  String name = 'Alex';
  int health = 100;
}
```

Le compilateur refuse :

```text
Error: The getter 'helth' isn't defined for the class 'Player'.
```

L'erreur est trouvée **pendant que vous écrivez le code**, pas devant l'utilisateur.

---

## 08.21.2 — Le type est perdu avec `Map<String, dynamic>`

```dart
void main() {
  Map<String, dynamic> joueur = {
    'name': 'Alex',
    'health': 100,
  };

  joueur['health'] = 'beaucoup'; // accepté !
  print(joueur['health']);
}
```

**Résultat :**

```text
beaucoup
```

La vie d'un joueur est devenue un texte. Avec une classe, `joueur.health = 'beaucoup';` est refusé immédiatement : `health` est un `int`.

---

## 08.21.3 — Une `Map` ne contient pas de comportement

Une `Map` stocke des données. Elle ne sait rien faire.

```dart
// Avec une Map : la logique est éparpillée dans des fonctions externes
void infligerDegats(Map<String, dynamic> joueur, int degats) {
  joueur['health'] = joueur['health'] - degats;
}
```

```dart
// Avec une classe : la logique est DANS l'objet
class Player {
  int health = 100;

  void takeDamage(int amount) {
    health = health - amount;
  }
}
```

Dans le second cas, impossible d'oublier la règle « la vie ne descend pas sous zéro » : elle est écrite au seul endroit qui compte.

---

## 08.21.4 — Tableau comparatif

| Critère | `Map<String, dynamic>` | Classe |
| --- | --- | --- |
| Fautes de frappe sur les noms | non détectées (`null` silencieux) | détectées à la compilation |
| Typage des valeurs | perdu (`dynamic`) | garanti (`int`, `String`...) |
| Autocomplétion de l'éditeur | quasi inexistante | complète |
| Comportement (méthodes) | impossible | naturel |
| Documentation du modèle | il faut deviner les clés | la classe liste tout |
| Refactorisation (renommer un champ) | manuelle et risquée | automatique et sûre |
| Lecture par un autre développeur | difficile | immédiate |

---

## 08.21.5 — Alors la `Map` ne sert à rien ?

Si, mais à autre chose.

```text
Une API renvoie du JSON  ->  Dart le reçoit comme Map<String, dynamic>
                                          │
                                          v
                          on le CONVERTIT en objets de nos classes
                                          │
                                          v
                        le reste de l'application manipule des objets
```

La `Map` est un **format d'échange**, pas un modèle de travail. Cette conversion s'appelle la désérialisation : elle sera traitée au chapitre 17.

---

## 08.22 — Les objets sont manipulés par référence

Un point qui surprend souvent, et qui explique de vrais bugs.

```dart
void main() {
  Player a = Player();
  a.name = 'Alex';

  Player b = a;
  b.name = 'Sophie';

  print(a.name);
  print(b.name);
}

class Player {
  String name = 'Inconnu';
  int health = 100;
}
```

**Résultat :**

```text
Sophie
Sophie
```

`Player b = a;` ne crée **pas** un deuxième joueur. Les deux variables désignent le même objet.

```text
   a ─────┐
          ├──────>  ┌──────────────────┐
   b ─────┘         │ OBJET Player #1  │
                    │ name = 'Sophie'  │
                    └──────────────────┘
```

Pour obtenir un vrai deuxième joueur, il faut instancier de nouveau :

```dart
void main() {
  Player a = Player();
  a.name = 'Alex';

  Player b = Player();
  b.name = 'Sophie';

  print(a.name);
  print(b.name);
}

class Player {
  String name = 'Inconnu';
  int health = 100;
}
```

**Résultat :**

```text
Alex
Sophie
```

```text
   a ────────────>  ┌──────────────────┐
                    │ OBJET #1 : Alex  │
                    └──────────────────┘
   b ────────────>  ┌──────────────────┐
                    │ OBJET #2 : Sophie│
                    └──────────────────┘
```

> Règle simple : **un `Player()` écrit = un objet créé**. Pas d'appel, pas d'objet.

---

## 08.23 — Vocabulaire à maîtriser avant le chapitre 09

| Mot | Définition courte | Exemple |
| --- | --- | --- |
| Classe | Le plan, le moule | `class Player { }` |
| Objet | Une chose fabriquée à partir du plan | l'exemplaire « Alex » |
| Instance | Synonyme d'objet, vu depuis sa classe | Alex est une instance de `Player` |
| Instancier | Fabriquer un objet | `Player()` |
| Propriété / attribut | Une donnée de l'objet | `health` |
| Méthode | Une action de l'objet | `attack()` |
| `this` | L'objet courant | `this.name = name;` |
| Référence | Le lien variable → objet | `Player b = a;` |

---

## 08.24 — Ce qui manque encore (et arrive au chapitre 09)

Notre code souffre encore de deux défauts visibles :

```dart
Player joueur = Player();
joueur.name = 'Alex';
joueur.health = 100;
joueur.maxHealth = 100;
joueur.attackPower = 18;
```

1. **C'est long.** Cinq lignes pour créer un joueur.
2. **Rien n'oblige à remplir les propriétés.** On peut oublier `name` et se retrouver avec `'Inconnu'` dans toute l'application.

Le **constructeur**, vu au chapitre 09, règle les deux problèmes d'un coup :

```dart
// Chapitre 09 - à ne pas écrire maintenant
Player joueur = Player(name: 'Alex', health: 100, attackPower: 18);
```

Terminez ce chapitre avec la méthode « instancier puis affecter ». Vous apprécierez d'autant plus le constructeur.

---

## 08.25 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `Error: Undefined name 'new'` ou avertissement `unnecessary_new` | Vous écrivez `new Player()` par habitude d'un autre langage | En Dart moderne, `new` est inutile : écrivez `Player()` |
| `Field 'name' should be initialized` / `Non-nullable instance field 'name' must be initialized` | Propriété déclarée sans valeur : `String name;` | Tant que le constructeur n'est pas vu (chapitre 09), donnez une valeur par défaut : `String name = 'Inconnu';` |
| `LateInitializationError: Field 'name' has not been initialized` | Vous avez écrit `late String name;` puis lu `joueur.name` avant de l'affecter | Affectez la propriété avant de la lire, ou utilisez une valeur par défaut. `late` sera traité au chapitre 12 |
| `Error: The getter 'name' isn't defined for the class 'Player'` | Faute de frappe (`nom` au lieu de `name`) ou propriété absente de la classe | Vérifiez l'orthographe exacte et la déclaration dans la classe |
| `Instance of 'Player'` s'affiche au lieu des données | `print(joueur)` affiche l'objet entier, que Dart ne sait pas décrire | Affichez une propriété (`print(joueur.name)`) ou appelez une méthode (`joueur.showStatus()`) |
| `Nom : Instance of 'Player'.name` | Interpolation `'$joueur.name'` sans accolades | Dès qu'il y a un point : `'${joueur.name}'` |
| La méthode ne s'exécute pas | Appel sans parenthèses : `joueur.showStatus;` | Ajoutez les parenthèses : `joueur.showStatus();` |
| `Error: Method not found: 'showStatus'` | Vous appelez `showStatus()` seul dans `main()` au lieu de l'appeler sur un objet | Une méthode s'appelle toujours sur un objet : `joueur.showStatus();` |
| `Error: The method 'attack' isn't defined for the class 'Player'` alors qu'elle existe | La méthode a été écrite en dehors des accolades de la classe | Vérifiez l'indentation et les accolades : la méthode doit être **dans** `class Player { ... }` |
| `Error: Instance member 'name' can't be accessed using static access` (`Player.name`) | Confusion classe / instance : vous utilisez le nom de la classe au lieu d'une variable | Créez d'abord un objet : `Player joueur = Player();` puis `joueur.name` |
| La propriété ne change jamais après un appel de méthode | Paramètre de même nom que la propriété : `name = name;` | Utilisez `this` : `this.name = name;` |
| `this` écrit partout, code illisible | Réflexe venu d'autres langages | En Dart, `this` ne sert qu'en cas de conflit de noms |
| Modifier un objet en modifie un autre | `Player b = a;` copie la **référence**, pas l'objet | Instanciez à nouveau : `Player b = Player();` |
| `Error: A class declaration must not be declared inside a function` | La classe a été écrite à l'intérieur de `main()` | Déclarez la classe en dehors de `main()`, avant ou après |
| Les points de vie deviennent négatifs | Aucune vérification dans `takeDamage()` | Ajoutez `if (health < 0) { health = 0; }` |
| `Error: The argument type 'String' can't be assigned to the parameter type 'int'` | Vous passez `joueur.takeDamage('30')` | Passez un nombre : `joueur.takeDamage(30)` |
| `Error: Too few positional arguments` sur `Player()` | Vous avez ajouté un constructeur sans le savoir, ou copié du code du chapitre 09 | Dans ce chapitre, `Player()` s'appelle toujours sans argument |

---

## 08.26 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| POO | Regrouper les données et les actions d'une même chose dans une seule structure |
| Classe | Le moule, le plan. Écrite une fois : `class Player { }` |
| Objet | Une chose fabriquée à partir du moule, vivant en mémoire |
| Instance | Synonyme d'objet, vu depuis sa classe |
| Instancier | `Player joueur = Player();` — pas de `new` en Dart |
| Propriété | Variable déclarée dans la classe : `int health = 100;` |
| Valeur par défaut | Obligatoire ici, car le constructeur n'arrive qu'au chapitre 09 |
| Méthode | Fonction déclarée dans la classe : `void attack() { }` |
| Point `.` | Sert à lire, à écrire et à appeler : `joueur.health`, `joueur.attack()` |
| Interpolation | Avec un point, il faut les accolades : `'${joueur.name}'` |
| `this` | L'objet courant. Utile seulement en cas de conflit de noms |
| Méthode `void` | N'a pas de `return`, elle agit ou affiche |
| Méthode typée | `int getHealth()`, `bool isAlive()`, `String describe()` |
| Paramètres | Une méthode combine ses paramètres et les propriétés de son objet |
| Plusieurs objets | Chaque objet possède sa propre copie des propriétés |
| `List<Player>` | Une liste peut contenir vos propres objets |
| Interaction | Une méthode peut recevoir un autre objet : `attack(Enemy target)` |
| Référence | `Player b = a;` fait pointer deux variables vers le même objet |
| Classe vs `Map` | La classe apporte le typage, l'autocomplétion, la détection d'erreurs et le comportement |
| Convention | Classe en `UpperCamelCase`, propriétés et méthodes en `lowerCamelCase` |

---

## 08.27 — Exercices

Faites-les dans l'ordre : ils sont progressifs. Chaque exercice s'écrit dans DartPad, avec `void main()` et la ou les classes en dessous.

Rappel : **aucun constructeur**. On instancie, puis on affecte.

### Exercice 1 — Première classe `Weapon` (facile)

Créez une classe `Weapon` avec trois propriétés et leurs valeurs par défaut :

- `name` (`String`), valeur par défaut `'Arme'` ;
- `damage` (`int`), valeur par défaut `10` ;
- `weight` (`double`), valeur par défaut `1.0`.

Dans `main()`, instanciez une `Weapon` et affichez ses trois propriétés, une par ligne, sans rien modifier.

### Exercice 2 — Modifier les propriétés (facile)

Reprenez la classe `Weapon` de l'exercice 1.

Dans `main()`, créez une arme, puis affectez-lui :

- `name` = `'Epee longue'` ;
- `damage` = `35` ;
- `weight` = `4.5`.

Affichez ensuite une seule ligne au format :

```text
Epee longue : 35 degats, 4.5 kg
```

### Exercice 3 — Première méthode (facile)

Ajoutez à la classe `Weapon` une méthode `void showInfo()` qui affiche exactement la ligne de l'exercice 2.

Dans `main()`, créez deux armes différentes et appelez `showInfo()` sur chacune.

### Exercice 4 — Méthode avec paramètre (facile)

Créez une classe `Potion` avec :

- `name` (`String`) = `'Potion'` ;
- `healAmount` (`int`) = `20` ;
- `quantity` (`int`) = `0`.

Ajoutez une méthode `void addQuantity(int amount)` qui augmente `quantity` de `amount` et affiche :

```text
Potion : quantite = 5
```

Dans `main()`, créez une potion, ajoutez-en 5, puis 3, et vérifiez le résultat.

### Exercice 5 — Méthode qui retourne un booléen (moyen)

Créez une classe `Chest` (coffre) avec :

- `name` (`String`) = `'Coffre'` ;
- `isLocked` (`bool`) = `true` ;
- `gold` (`int`) = `100`.

Ajoutez :

- `bool canOpen()` qui retourne `true` si le coffre n'est pas verrouillé ;
- `void open()` qui affiche le contenu si le coffre peut être ouvert, sinon affiche `'Coffre verrouille.'`.

Dans `main()`, testez les deux cas.

### Exercice 6 — Le mot-clé `this` (moyen)

Créez une classe `Player` avec `name` = `'Inconnu'` et `level` = `1`.

Ajoutez une méthode `void setName(String name)` qui affecte le paramètre à la propriété. Le paramètre doit **obligatoirement** s'appeler `name`.

Dans `main()`, créez un joueur, appelez `setName('Maria')` et affichez `joueur.name`.

### Exercice 7 — Plusieurs objets et une `List` (moyen)

Créez une classe `Enemy` avec `name` = `'Ennemi'`, `health` = `30` et `rewardScore` = `10`.

Dans `main()`, créez trois ennemis (`'Gobelin'` 30 PV, `'Squelette'` 45 PV, `'Orc'` 60 PV), rangez-les dans une `List<Enemy>`, puis parcourez la liste avec `for-in` pour afficher :

```text
Gobelin -> 30 PV
```

Affichez enfin le nombre total d'ennemis.

### Exercice 8 — Méthode qui retourne une valeur calculée (moyen)

Créez une classe `Player` avec `name`, `health` (`int` = `100`) et `maxHealth` (`int` = `100`).

Ajoutez :

- `double healthPercent()` qui retourne le pourcentage de vie restant ;
- `String describe()` qui retourne une chaîne du type `'Alex : 60/100 PV (60.0 %)'`.

Dans `main()`, créez un joueur à 60 PV et affichez le résultat de `describe()`.

### Exercice 9 — Somme des scores d'une liste d'objets (moyen)

Reprenez la classe `Enemy` de l'exercice 7.

Dans `main()`, créez quatre ennemis avec des `rewardScore` différents (10, 25, 40, 100), rangez-les dans une liste et calculez avec une boucle le score total que le joueur gagnerait s'il les vainquait tous. Affichez aussi le nom de l'ennemi qui rapporte le plus.

### Exercice 10 — Classe hors jeu : `BankAccount` (moyen)

Créez une classe `BankAccount` avec :

- `owner` (`String`) = `'Inconnu'` ;
- `balance` (`double`) = `0.0`.

Ajoutez :

- `void deposit(double amount)` : refuse les montants négatifs ou nuls ;
- `void withdraw(double amount)` : refuse si le solde est insuffisant ;
- `void showBalance()`.

Dans `main()`, testez un dépôt valide, un retrait valide, un retrait trop grand et un dépôt négatif.

### Exercice 11 — Interaction entre deux objets (difficile)

Créez une classe `Player` (`name`, `health` = 100, `attackPower` = 20, `score` = 0) et une classe `Enemy` (`name`, `health` = 50, `attackPower` = 10, `rewardScore` = 30).

Ajoutez :

- `Player.attack(Enemy target)` qui inflige `attackPower` dégâts, et si l'ennemi tombe à 0, ajoute `rewardScore` au score du joueur ;
- `Enemy.attack(Player target)` qui inflige ses dégâts, mais ne fait rien si l'ennemi est déjà vaincu.

Dans `main()`, écrivez une boucle `while` de combat au tour par tour, et annoncez le vainqueur.

### Exercice 12 — Mini-projet : fiche de personnage (difficile)

Créez une classe `Character` représentant une fiche de personnage complète, avec au minimum :

- `name` (`String`) = `'Sans nom'` ;
- `characterClass` (`String`) = `'Aventurier'` ;
- `level` (`int`) = `1` ;
- `experience` (`int`) = `0` ;
- `health` (`int`) = `100` ;
- `maxHealth` (`int`) = `100` ;
- `strength` (`int`) = `10` ;
- `agility` (`int`) = `10` ;
- `intelligence` (`int`) = `10` ;
- `gold` (`int`) = `0` ;
- `inventory` (`List<String>`) = liste vide.

Et les méthodes suivantes :

- `void showSheet()` : affiche une fiche encadrée et lisible ;
- `void gainExperience(int amount)` : ajoute l'expérience et fait monter de niveau tous les 100 points (une seule montée par appel suffit, mais gérez le cas de plusieurs niveaux si vous le souhaitez) ;
- `void levelUp()` : +1 niveau, +20 `maxHealth`, vie remise au maximum, +2 sur chaque statistique ;
- `void addItem(String item)` : ajoute un objet à l'inventaire ;
- `bool hasItem(String item)` : indique si l'objet est possédé ;
- `int powerScore()` : retourne `strength * 2 + agility + intelligence + level * 10` ;
- `String rank()` : `'Novice'` sous 50, `'Confirme'` sous 100, `'Elite'` au-delà.

Dans `main()`, créez un personnage, remplissez sa fiche, ajoutez trois objets à son inventaire, donnez-lui 250 points d'expérience et affichez la fiche avant et après.

---

## 08.28 — Corrections des exercices

### Correction 1

```dart
void main() {
  Weapon arme = Weapon();

  print(arme.name);
  print(arme.damage);
  print(arme.weight);
}

class Weapon {
  String name = 'Arme';
  int damage = 10;
  double weight = 1.0;
}
```

**Résultat :**

```text
Arme
10
1.0
```

**Explication :** la classe `Weapon` est déclarée en dehors de `main()`. `Weapon()` fabrique un objet ; comme aucune valeur n'est affectée ensuite, les trois propriétés gardent leurs valeurs par défaut. Notez `1.0` et non `1` : `weight` est un `double`.

---

### Correction 2

```dart
void main() {
  Weapon arme = Weapon();

  arme.name = 'Epee longue';
  arme.damage = 35;
  arme.weight = 4.5;

  print('${arme.name} : ${arme.damage} degats, ${arme.weight} kg');
}

class Weapon {
  String name = 'Arme';
  int damage = 10;
  double weight = 1.0;
}
```

**Résultat :**

```text
Epee longue : 35 degats, 4.5 kg
```

**Explication :** on instancie d'abord, puis on affecte chaque propriété avec le point. Les accolades sont indispensables dans l'interpolation, car chaque expression contient un point : `${arme.name}` et non `$arme.name`.

---

### Correction 3

```dart
void main() {
  Weapon epee = Weapon();
  epee.name = 'Epee longue';
  epee.damage = 35;
  epee.weight = 4.5;

  Weapon dague = Weapon();
  dague.name = 'Dague';
  dague.damage = 12;
  dague.weight = 0.8;

  epee.showInfo();
  dague.showInfo();
}

class Weapon {
  String name = 'Arme';
  int damage = 10;
  double weight = 1.0;

  void showInfo() {
    print('$name : $damage degats, $weight kg');
  }
}
```

**Résultat :**

```text
Epee longue : 35 degats, 4.5 kg
Dague : 12 degats, 0.8 kg
```

**Explication :** à l'intérieur de la méthode, on écrit directement `$name` : la méthode lit les propriétés de **son** objet. Le même code produit deux affichages différents car il s'exécute sur deux objets distincts. C'est tout l'intérêt de la POO.

---

### Correction 4

```dart
void main() {
  Potion potion = Potion();
  potion.name = 'Potion de soin';

  potion.addQuantity(5);
  potion.addQuantity(3);
}

class Potion {
  String name = 'Potion';
  int healAmount = 20;
  int quantity = 0;

  void addQuantity(int amount) {
    quantity = quantity + amount;
    print('$name : quantite = $quantity');
  }
}
```

**Résultat :**

```text
Potion de soin : quantite = 5
Potion de soin : quantite = 8
```

**Explication :** `amount` est le paramètre (l'information venue de l'extérieur) ; `quantity` est la propriété (la mémoire de l'objet). La méthode combine les deux. Comme l'objet conserve son état entre deux appels, le second appel part de 5 et non de 0.

---

### Correction 5

```dart
void main() {
  Chest coffre1 = Chest();
  coffre1.name = 'Coffre du donjon';
  coffre1.gold = 250;

  Chest coffre2 = Chest();
  coffre2.name = 'Coffre ouvert';
  coffre2.isLocked = false;
  coffre2.gold = 80;

  coffre1.open();
  coffre2.open();

  print('coffre1 ouvrable ? ${coffre1.canOpen()}');
  print('coffre2 ouvrable ? ${coffre2.canOpen()}');
}

class Chest {
  String name = 'Coffre';
  bool isLocked = true;
  int gold = 100;

  bool canOpen() {
    return isLocked == false;
  }

  void open() {
    if (canOpen()) {
      print('$name ouvert : $gold pieces d\'or.');
    } else {
      print('Coffre verrouille.');
    }
  }
}
```

**Résultat :**

```text
Coffre verrouille.
Coffre ouvert : 80 pieces d'or.
coffre1 ouvrable ? false
coffre2 ouvrable ? true
```

**Explication :** `canOpen()` retourne un `bool` et peut donc être utilisé directement dans un `if`. Remarquez que `open()` appelle `canOpen()` **sans point** : à l'intérieur d'une classe, une méthode appelle les autres méthodes du même objet directement. On aurait aussi pu écrire `return !isLocked;`.

---

### Correction 6

```dart
void main() {
  Player joueur = Player();

  print('Avant : ${joueur.name}');
  joueur.setName('Maria');
  print('Apres : ${joueur.name}');
}

class Player {
  String name = 'Inconnu';
  int level = 1;

  void setName(String name) {
    this.name = name;
  }
}
```

**Résultat :**

```text
Avant : Inconnu
Apres : Maria
```

**Explication :** le paramètre `name` masque la propriété `name`. Sans `this`, la ligne `name = name;` affecterait le paramètre à lui-même et la propriété ne changerait jamais. `this.name` désigne explicitement la propriété de l'objet courant. C'est le seul cas où `this` est obligatoire en Dart.

---

### Correction 7

```dart
void main() {
  Enemy gobelin = Enemy();
  gobelin.name = 'Gobelin';
  gobelin.health = 30;

  Enemy squelette = Enemy();
  squelette.name = 'Squelette';
  squelette.health = 45;

  Enemy orc = Enemy();
  orc.name = 'Orc';
  orc.health = 60;

  List<Enemy> ennemis = [gobelin, squelette, orc];

  for (Enemy e in ennemis) {
    print('${e.name} -> ${e.health} PV');
  }

  print('Total : ${ennemis.length} ennemis');
}

class Enemy {
  String name = 'Ennemi';
  int health = 30;
  int rewardScore = 10;
}
```

**Résultat :**

```text
Gobelin -> 30 PV
Squelette -> 45 PV
Orc -> 60 PV
Total : 3 ennemis
```

**Explication :** `List<Enemy>` annonce une liste d'objets `Enemy`. La boucle `for-in` place tour à tour chaque objet dans la variable `e`, sur laquelle on utilise le point comme sur n'importe quel objet. Toutes les techniques du chapitre 06 (`length`, `add`, index) restent valables.

---

### Correction 8

```dart
void main() {
  Player joueur = Player();
  joueur.name = 'Alex';
  joueur.health = 60;
  joueur.maxHealth = 100;

  print(joueur.describe());
  print('Pourcentage brut : ${joueur.healthPercent()}');
}

class Player {
  String name = 'Inconnu';
  int health = 100;
  int maxHealth = 100;

  double healthPercent() {
    return health / maxHealth * 100;
  }

  String describe() {
    return '$name : $health/$maxHealth PV (${healthPercent()} %)';
  }
}
```

**Résultat :**

```text
Alex : 60/100 PV (60.0 %)
Pourcentage brut : 60.0
```

**Explication :** `healthPercent()` retourne un `double` car la division `/` entre deux `int` produit toujours un `double` en Dart. `describe()` appelle `healthPercent()` à l'intérieur d'une interpolation : les accolades sont obligatoires puisque l'expression contient des parenthèses. Une méthode qui `return` est réutilisable, contrairement à une méthode qui se contente d'un `print`.

---

### Correction 9

```dart
void main() {
  Enemy gobelin = Enemy();
  gobelin.name = 'Gobelin';
  gobelin.rewardScore = 10;

  Enemy squelette = Enemy();
  squelette.name = 'Squelette';
  squelette.rewardScore = 25;

  Enemy orc = Enemy();
  orc.name = 'Orc';
  orc.rewardScore = 40;

  Enemy boss = Enemy();
  boss.name = 'Roi Gobelin';
  boss.rewardScore = 100;

  List<Enemy> ennemis = [gobelin, squelette, orc, boss];

  int total = 0;
  String meilleurNom = '';
  int meilleurScore = 0;

  for (Enemy e in ennemis) {
    total = total + e.rewardScore;

    if (e.rewardScore > meilleurScore) {
      meilleurScore = e.rewardScore;
      meilleurNom = e.name;
    }
  }

  print('Score total possible : $total');
  print('Meilleure recompense : $meilleurNom ($meilleurScore points)');
}

class Enemy {
  String name = 'Ennemi';
  int health = 30;
  int rewardScore = 10;
}
```

**Résultat :**

```text
Score total possible : 175
Meilleure recompense : Roi Gobelin (100 points)
```

**Explication :** on combine la boucle du chapitre 05, la liste du chapitre 06 et les objets de ce chapitre. La recherche du maximum suit le schéma classique : une variable témoin initialisée à 0, mise à jour chaque fois qu'on trouve mieux. Le total vaut 10 + 25 + 40 + 100 = 175.

---

### Correction 10

```dart
void main() {
  BankAccount compte = BankAccount();
  compte.owner = 'Alex';

  compte.showBalance();

  compte.deposit(500.0);
  compte.withdraw(120.0);
  compte.withdraw(1000.0);
  compte.deposit(-50.0);

  compte.showBalance();
}

class BankAccount {
  String owner = 'Inconnu';
  double balance = 0.0;

  void deposit(double amount) {
    if (amount <= 0) {
      print('Depot invalide : $amount');
      return;
    }
    balance = balance + amount;
    print('Depot de $amount accepte.');
  }

  void withdraw(double amount) {
    if (amount <= 0) {
      print('Retrait invalide : $amount');
      return;
    }
    if (amount > balance) {
      print('Solde insuffisant : $balance disponible.');
      return;
    }
    balance = balance - amount;
    print('Retrait de $amount accepte.');
  }

  void showBalance() {
    print('Compte de $owner : $balance euros');
  }
}
```

**Résultat :**

```text
Compte de Alex : 0.0 euros
Depot de 500.0 accepte.
Retrait de 120.0 accepte.
Solde insuffisant : 380.0 disponible.
Depot invalide : -50.0
Compte de Alex : 380.0 euros
```

**Explication :** la POO ne sert pas qu'aux jeux : un compte bancaire est un objet comme un autre. Le `return` anticipé dans `deposit()` et `withdraw()` interrompt la méthode dès qu'une règle est violée, ce qui évite un `else` imbriqué. Les règles de validité sont écrites **dans la classe** : impossible pour le reste du programme de les contourner par oubli.

---

### Correction 11

```dart
void main() {
  Player joueur = Player();
  joueur.name = 'Alex';
  joueur.health = 100;
  joueur.attackPower = 20;

  Enemy orc = Enemy();
  orc.name = 'Orc';
  orc.health = 50;
  orc.attackPower = 10;
  orc.rewardScore = 30;

  int tour = 1;

  while (joueur.health > 0 && orc.health > 0) {
    print('--- Tour $tour ---');
    joueur.attack(orc);
    orc.attack(joueur);
    print('${joueur.name} : ${joueur.health} PV | ${orc.name} : ${orc.health} PV');
    tour = tour + 1;
  }

  if (joueur.health > 0) {
    print('${joueur.name} remporte le combat avec ${joueur.score} points.');
  } else {
    print('${joueur.name} a ete vaincu.');
  }
}

class Player {
  String name = 'Heros';
  int health = 100;
  int attackPower = 20;
  int score = 0;

  void takeDamage(int amount) {
    health = health - amount;
    if (health < 0) {
      health = 0;
    }
  }

  void attack(Enemy target) {
    print('$name frappe ${target.name} (-${attackPower} PV).');
    target.takeDamage(attackPower);

    if (target.isDefeated()) {
      print('${target.name} est vaincu !');
      score = score + target.rewardScore;
    }
  }
}

class Enemy {
  String name = 'Ennemi';
  int health = 50;
  int attackPower = 10;
  int rewardScore = 30;

  bool isDefeated() {
    return health <= 0;
  }

  void takeDamage(int amount) {
    health = health - amount;
    if (health < 0) {
      health = 0;
    }
  }

  void attack(Player target) {
    if (isDefeated()) {
      return;
    }
    print('$name riposte sur ${target.name} (-${attackPower} PV).');
    target.takeDamage(attackPower);
  }
}
```

**Résultat :**

```text
--- Tour 1 ---
Alex frappe Orc (-20 PV).
Orc riposte sur Alex (-10 PV).
Alex : 90 PV | Orc : 30 PV
--- Tour 2 ---
Alex frappe Orc (-20 PV).
Orc riposte sur Alex (-10 PV).
Alex : 80 PV | Orc : 10 PV
--- Tour 3 ---
Alex frappe Orc (-20 PV).
Orc est vaincu !
Alex : 80 PV | Orc : 0 PV
Alex remporte le combat avec 30 points.
```

**Explication :** `attack(Enemy target)` reçoit l'objet lui-même, pas une copie : `target.takeDamage(...)` modifie donc réellement l'ennemi créé dans `main()`. Au tour 3, l'orc tombe à 0 PV ; sa méthode `attack()` commence par `if (isDefeated()) return;` et n'affiche donc rien. La boucle `while` s'arrête ensuite car `orc.health > 0` est faux.

---

### Correction 12 — Mini-projet : fiche de personnage

```dart
void main() {
  Character heros = Character();
  heros.name = 'Alex le Vagabond';
  heros.characterClass = 'Guerrier';
  heros.strength = 16;
  heros.agility = 12;
  heros.intelligence = 8;
  heros.gold = 75;

  heros.addItem('Epee longue');
  heros.addItem('Bouclier de bois');
  heros.addItem('Potion de soin');

  print('=== FICHE INITIALE ===');
  heros.showSheet();

  print('');
  print('=== PROGRESSION ===');
  heros.gainExperience(250);

  print('');
  print('=== FICHE FINALE ===');
  heros.showSheet();

  print('');
  bool aUnePotion = heros.hasItem('Potion de soin');
  bool aUnArc = heros.hasItem('Arc long');
  print('Possede une potion ? $aUnePotion');
  print('Possede un arc ?     $aUnArc');
}

class Character {
  String name = 'Sans nom';
  String characterClass = 'Aventurier';
  int level = 1;
  int experience = 0;
  int health = 100;
  int maxHealth = 100;
  int strength = 10;
  int agility = 10;
  int intelligence = 10;
  int gold = 0;
  List<String> inventory = [];

  void showSheet() {
    print('+----------------------------------------+');
    print('| $name');
    print('| Classe : $characterClass');
    print('+----------------------------------------+');
    print('| Niveau       : $level');
    print('| Experience   : $experience');
    print('| Vie          : $health / $maxHealth');
    print('| Force        : $strength');
    print('| Agilite      : $agility');
    print('| Intelligence : $intelligence');
    print('| Or           : $gold');
    print('| Puissance    : ${powerScore()} ( ${rank()} )');
    print('| Inventaire   : ${inventory.length} objet(s)');
    for (String item in inventory) {
      print('|   - $item');
    }
    print('+----------------------------------------+');
  }

  void levelUp() {
    level = level + 1;
    maxHealth = maxHealth + 20;
    health = maxHealth;
    strength = strength + 2;
    agility = agility + 2;
    intelligence = intelligence + 2;
    print('Niveau $level atteint ! Statistiques ameliorees.');
  }

  void gainExperience(int amount) {
    experience = experience + amount;
    print('$name gagne $amount points d\'experience (total : $experience).');

    while (experience >= 100) {
      experience = experience - 100;
      levelUp();
    }
  }

  void addItem(String item) {
    inventory.add(item);
    print('Objet ajoute : $item');
  }

  bool hasItem(String item) {
    return inventory.contains(item);
  }

  int powerScore() {
    return strength * 2 + agility + intelligence + level * 10;
  }

  String rank() {
    if (powerScore() < 50) {
      return 'Novice';
    } else if (powerScore() < 100) {
      return 'Confirme';
    } else {
      return 'Elite';
    }
  }
}
```

**Résultat :**

```text
Objet ajoute : Epee longue
Objet ajoute : Bouclier de bois
Objet ajoute : Potion de soin
=== FICHE INITIALE ===
+----------------------------------------+
| Alex le Vagabond
| Classe : Guerrier
+----------------------------------------+
| Niveau       : 1
| Experience   : 0
| Vie          : 100 / 100
| Force        : 16
| Agilite      : 12
| Intelligence : 8
| Or           : 75
| Puissance    : 62 ( Confirme )
| Inventaire   : 3 objet(s)
|   - Epee longue
|   - Bouclier de bois
|   - Potion de soin
+----------------------------------------+

=== PROGRESSION ===
Alex le Vagabond gagne 250 points d'experience (total : 250).
Niveau 2 atteint ! Statistiques ameliorees.
Niveau 3 atteint ! Statistiques ameliorees.

=== FICHE FINALE ===
+----------------------------------------+
| Alex le Vagabond
| Classe : Guerrier
+----------------------------------------+
| Niveau       : 3
| Experience   : 50
| Vie          : 140 / 140
| Force        : 20
| Agilite      : 16
| Intelligence : 12
| Or           : 75
| Puissance    : 98 ( Confirme )
| Inventaire   : 3 objet(s)
|   - Epee longue
|   - Bouclier de bois
|   - Potion de soin
+----------------------------------------+

Possede une potion ? true
Possede un arc ?     false
```

**Explication :** cette fiche réunit tout le chapitre.

- Les propriétés couvrent plusieurs types : `String`, `int`, `bool` (implicite via `hasItem`), et même une `List<String>` pour l'inventaire.
- `gainExperience()` illustre une méthode qui en appelle une autre : elle boucle avec `while` et déclenche `levelUp()` autant de fois que nécessaire. Avec 250 points, le personnage monte deux fois et conserve 50 points.
- `powerScore()` calcule : 20 x 2 + 16 + 12 + 3 x 10 = 40 + 16 + 12 + 30 = 98.
- `rank()` réutilise `powerScore()` : la logique de calcul n'est écrite qu'une seule fois.
- `showSheet()` ne calcule rien : elle affiche. Chaque méthode a une responsabilité unique.

Depuis `main()`, tout se pilote en quelques lignes lisibles. Comparez avec ce qu'il aurait fallu écrire au chapitre 07 avec des variables isolées : une dizaine de variables et autant de fonctions prenant six paramètres chacune.

---

## Et maintenant ?

Vous savez désormais décrire une chose avec une classe, en fabriquer autant d'exemplaires que nécessaire, leur donner un comportement et les faire dialoguer.

Il reste un défaut évident : créer un objet demande encore une ligne par propriété, et rien n'oblige à les remplir toutes. Un joueur sans nom, un produit sans prix : ces objets à moitié construits sont une source de bugs.

C'est exactement le rôle du **constructeur**, que nous allons découvrir maintenant. Vous apprendrez à écrire :

```dart
Player joueur = Player(name: 'Alex', health: 100, attackPower: 18);
```

et à rendre certaines informations obligatoires avec `required`.

Chapitre suivant : [09-PARTIE-1A—CONSTRUCTEURS-ET-MODÉLISATION.md](09-PARTIE-1A—CONSTRUCTEURS-ET-MODÉLISATION.md)
