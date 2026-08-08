# PARTIE 2A — LES FONDAMENTAUX DU JEU 2D
# CHAPITRE 23 — MOUVEMENT, VÉLOCITÉ ET PHYSIQUE SIMPLE

> **Niveau :** intermédiaire
> **Durée estimée :** 8 h
> **Pré-requis :** chapitre 20 (boucle de jeu, delta time), chapitre 21 (Canvas), chapitre 22 (sprites et animations), et la POO des chapitres 08 à 11
> **Ce que vous saurez faire à la fin :** écrire votre propre classe de vecteur 2D, appliquer la gravité, faire sauter un personnage avec un saut variable, un double saut, du coyote time, du frottement et un rebond — le tout en Flutter pur, sans aucun moteur physique.

---

## 23.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- distinguer clairement **position**, **vitesse** et **accélération**, et dire dans quelle unité chacune s'exprime ;
- expliquer ce qu'est un vecteur en 2D et pourquoi un jeu en a besoin ;
- écrire de zéro une classe `Vec2` complète, immuable, testable ;
- surcharger les opérateurs `+`, `-`, `*` et `==` en Dart ;
- calculer la **longueur** d'un vecteur et la **normaliser** ;
- diagnostiquer et corriger le célèbre bug du **déplacement en diagonale plus rapide** ;
- utiliser le **produit scalaire** pour savoir si un ennemi est devant ou derrière le héros ;
- mesurer une **distance** entre deux points sans racine carrée inutile ;
- convertir un vecteur en **angle** avec `atan2`, et un angle en vecteur ;
- écrire l'**intégration** `position += vitesse * dt` et `vitesse += acceleration * dt` ;
- expliquer pourquoi **l'ordre de ces deux lignes change le résultat** ;
- comparer **Euler explicite** et **Euler semi-implicite**, et choisir le second en connaissance de cause ;
- appliquer une **gravité**, arrêter la chute sur un **sol**, déclencher un **saut** ;
- implémenter le **double saut**, le **saut variable**, le **coyote time** et le **jump buffering** ;
- ajouter du **frottement**, un **amortissement**, une **vitesse maximale** avec `clamp` ;
- régler l'**accélération et la décélération** d'un personnage pour obtenir un bon *game feel* ;
- faire **rebondir** un objet avec un coefficient de restitution ;
- écrire un **mouvement circulaire** et une **interpolation linéaire** (`lerp`) ;
- faire **suivre une cible en douceur** par une caméra ou un familier ;
- programmer un **tremblement d'écran** et un **knockback** ;
- assembler un personnage complet du **Donjon de Dart** qui court, saute, retombe et glisse.

---

## 23.1 — Position, vitesse, accélération : les trois notions

Vous n'avez pas besoin d'avoir fait de la physique. Tout ce chapitre repose sur trois idées que vous connaissez déjà dans la vie courante. Nommons-les proprement une bonne fois pour toutes.

### La position

La position, c'est **où se trouve l'objet**. Rien de plus. Dans un jeu 2D, c'est un couple de nombres : `x` et `y`. On l'exprime en **pixels**.

```text
  Le héros est à x = 120, y = 300.
  Unité : le pixel.
```

### La vitesse

La vitesse, c'est **de combien la position change en une seconde**. Ce n'est pas une position, c'est un changement de position.

```text
  Le héros va à 200 pixels par seconde vers la droite.
  Unité : le pixel PAR SECONDE (px/s).
```

Une vitesse ne dit rien sur l'endroit où se trouve l'objet. Deux héros à deux endroits différents peuvent avoir exactement la même vitesse.

### L'accélération

L'accélération, c'est **de combien la vitesse change en une seconde**. Ce n'est ni une position, ni une vitesse : c'est un changement de vitesse.

```text
  La gravité tire le héros vers le bas à 1200 pixels par seconde CARRÉE.
  Unité : le pixel PAR SECONDE PAR SECONDE (px/s²).
```

Le mot « seconde carrée » effraie souvent. Il ne veut rien dire de mystérieux : il signale simplement qu'on parle d'un changement de vitesse, et que la vitesse est déjà « par seconde ». Une accélération de 1200 px/s² veut dire : *chaque seconde qui passe, la vitesse gagne 1200 px/s*.

### Le schéma de la chaîne

Voici la chaîne complète, et c'est tout le chapitre en un dessin.

```text
  ┌──────────────────┐        ┌──────────────────┐        ┌──────────────────┐
  │  ACCÉLÉRATION    │ ────>  │     VITESSE      │ ────>  │    POSITION      │
  │    (px/s²)       │ modifie│     (px/s)       │ modifie│     (px)         │
  └──────────────────┘        └──────────────────┘        └──────────────────┘
        gravité,                 « je monte à              « je suis à
        moteur,                    300 px/s »                y = 240 »
        poussée
```

On lit ce schéma de gauche à droite : **l'accélération nourrit la vitesse, la vitesse nourrit la position**. Jamais l'inverse.

### Une analogie de voiture, à retenir

| Notion | Dans une voiture | Dans le jeu |
| --- | --- | --- |
| Position | le point sur la carte | `position` (px) |
| Vitesse | ce qu'indique le compteur | `vitesse` (px/s) |
| Accélération | l'appui sur la pédale | `acceleration` (px/s²) |
| Accélération négative | l'appui sur le frein | frottement, gravité vers le haut |

Un débutant écrit souvent : « le héros a une vitesse de 300 pixels ». C'est faux et cela finit toujours par un bug. Une vitesse ne s'exprime jamais en pixels ; elle s'exprime en pixels **par seconde**. Le chapitre 20 vous a déjà mis en garde : dès que vous voyez une valeur sans unité claire, vous devez pouvoir répondre à la question « par seconde ou pas ? ».

Vérifions cette chaîne dans un programme console très simple, sans vecteur pour l'instant : une seule dimension, la verticale.

```dart
void main() {
  // Une seule dimension : la hauteur (y augmente vers le bas, comme au chapitre 21).
  double y = 0;              // px
  double vitesseY = -400;    // px/s : négatif = vers le haut
  const double gravite = 1200; // px/s²
  const double dt = 0.1;     // on simule des pas de 100 ms pour lire les chiffres

  print('temps |     y | vitesseY');
  for (int i = 0; i <= 8; i++) {
    print('${(i * dt).toStringAsFixed(1)} s | '
        '${y.toStringAsFixed(1).padLeft(5)} | '
        '${vitesseY.toStringAsFixed(1).padLeft(7)}');

    vitesseY += gravite * dt; // l'accélération modifie la vitesse
    y += vitesseY * dt;       // la vitesse modifie la position
  }
}
```

**Résultat :**

```text
temps |     y | vitesseY
0.0 s |   0.0 |  -400.0
0.1 s | -28.0 |  -280.0
0.2 s | -44.0 |  -160.0
0.3 s | -48.0 |   -40.0
0.4 s | -40.0 |    80.0
0.5 s | -20.0 |   200.0
0.6 s |  12.0 |   320.0
0.7 s |  56.0 |   440.0
0.8 s | 112.0 |   560.0
```

Lisez la colonne `vitesseY` : elle monte régulièrement de 120 à chaque pas (1200 × 0,1). Lisez la colonne `y` : elle descend d'abord (valeurs négatives, donc au-dessus du point de départ), atteint son minimum vers 0,3 s, puis remonte. Vous venez de simuler un saut. Il n'y a rien d'autre dans ce chapitre que ces deux lignes, répétées et enrichies.

> **À retenir.** Accélération → vitesse → position. Trois notions, trois unités, deux lignes de code.

---

## 23.2 — Un vecteur en 2D

Jusqu'ici nous avons travaillé sur une seule dimension. Un jeu 2D en a deux. Or on ne veut pas écrire deux fois chaque ligne :

```dart
// Ce que l'on veut ÉVITER d'écrire partout :
positionX += vitesseX * dt;
positionY += vitesseY * dt;
vitesseX += accelerationX * dt;
vitesseY += accelerationY * dt;
```

Quatre lignes pour ce qui est conceptuellement une seule idée. Et cela empire avec les collisions, le knockback, la caméra. La solution existe depuis longtemps : le **vecteur**.

### Définition

Un vecteur 2D, c'est simplement **deux nombres regroupés dans un seul objet**, sur lesquels on peut faire des opérations d'un seul coup.

```text
  Vec2(3, 4)
     │  │
     │  └── composante y
     └───── composante x
```

### Deux lectures d'un même objet

Un vecteur se lit de deux manières, et c'est la source de la moitié des confusions des débutants.

```text
  LECTURE 1 : UN POINT (une position)

      y
      │
    4 │        ● (3, 4)
      │
      │
      └─────────────── x
      0        3

  « Le coffre est à la position (3, 4). »


  LECTURE 2 : UNE FLÈCHE (un déplacement)

      y
      │       ↗
    4 │      ╱ ● arrivée
      │     ╱
      │    ╱
      │   ● départ
      └─────────────── x

  « Déplace-toi de 3 vers la droite et de 4 vers le bas. »
```

Le même objet `Vec2(3, 4)` sert aux deux. La différence est dans **ce que vous en faites**, pas dans le type.

| Ce que le vecteur représente | Exemple de nom de variable | Unité |
| --- | --- | --- |
| Un point du monde | `position`, `cible`, `spawn` | px |
| Un déplacement | `deplacement`, `decalage` | px |
| Une vitesse | `vitesse` | px/s |
| Une accélération | `acceleration`, `gravite` | px/s² |
| Une direction pure | `direction` (longueur 1) | sans unité |

Cette dernière ligne est capitale : une **direction** est un vecteur dont la longueur vaut exactement 1. Il ne dit pas « à quelle vitesse », il dit seulement « vers où ». Nous y reviendrons en 23.7.

### Pourquoi pas simplement `Offset` ?

Flutter fournit déjà une classe `Offset` avec un `dx` et un `dy`, que vous avez utilisée au chapitre 21 pour dessiner. Pourquoi ne pas s'en servir ?

Trois raisons.

1. `Offset` appartient à `dart:ui`. Votre logique de jeu deviendrait dépendante de Flutter, donc impossible à tester dans un simple `dart test` en console.
2. `Offset` ne propose ni normalisation, ni produit scalaire, ni limitation de longueur. Vous écririez ces fonctions à côté, en dehors de la classe.
3. Et surtout : **vous ne comprendrez la physique de jeu qu'en écrivant vous-même ces opérations.** C'est tout l'objet du chapitre.

Nous écrirons donc `Vec2`, et nous le convertirons en `Offset` uniquement au moment de dessiner. La règle du chapitre 20 tient toujours : la logique ne connaît pas le rendu.

> **Remarque.** Le moteur Flame, à partir du chapitre 27, fournit une classe `Vector2` très proche de celle que nous allons écrire. Vous la comprendrez d'autant mieux que vous aurez construit la vôtre.

---

## 23.3 — Écrire sa propre classe `Vec2` (rappel POO, chapitres 08-09)

Reprenons les outils de la PARTIE 1A. Une classe, des champs, un constructeur, des méthodes. Rien de neuf, seulement une application.

Première décision, et elle est structurante : **notre `Vec2` sera immuable**. Ses champs seront `final`. Chaque opération renverra un **nouveau** vecteur au lieu de modifier l'ancien.

Pourquoi ? Parce que le bug suivant est extrêmement fréquent dans les jeux :

```text
  AVEC UN VECTEUR MUTABLE (le piège)

  final v = Vec2(0, 0);
  heros.vitesse = v;
  gobelin.vitesse = v;      // les DEUX pointent sur le MÊME objet

  heros.vitesse.x = 200;    // et le gobelin part aussi à 200 !
```

Avec un vecteur immuable, ce partage accidentel est impossible : on ne peut rien modifier en place.

Voici la première version de la classe.

```dart
import 'dart:math' as math;

/// Un vecteur 2D immuable : deux nombres et des opérations dessus.
class Vec2 {
  final double x;
  final double y;

  const Vec2(this.x, this.y);

  /// Le vecteur nul : (0, 0). Point de départ de beaucoup de calculs.
  static const Vec2 zero = Vec2(0, 0);

  @override
  String toString() =>
      '(${x.toStringAsFixed(2)}, ${y.toStringAsFixed(2)})';
}

void main() {
  const Vec2 positionHeros = Vec2(120, 300);
  const Vec2 vitesse = Vec2(200, -50);

  print('position : $positionHeros');
  print('vitesse  : $vitesse');
  print('zéro     : ${Vec2.zero}');
}
```

**Résultat :**

```text
position : (120.00, 300.00)
vitesse  : (200.00, -50.00)
zéro     : (0.00, 0.00)
```

Trois détails méritent une explication, tous vus dans les chapitres 08 et 09.

**`final double x`.** Une fois construit, un `Vec2` ne change plus. C'est le choix d'immuabilité expliqué plus haut.

**`const Vec2(this.x, this.y)`.** Le constructeur est `const`, donc Dart peut créer certains vecteurs à la compilation. `Vec2.zero` en profite : il n'existe qu'une seule fois en mémoire dans tout le programme, quel que soit le nombre d'appels.

**`toString()` avec `toStringAsFixed(2)`.** Sans cela, `Vec2(1/3, 0)` s'afficherait `(0.3333333333333333, 0.0)`, illisible dans une console de débogage. Deux décimales suffisent pour raisonner sur des pixels.

Ajoutons maintenant deux constructeurs nommés utiles, dans l'esprit du chapitre 09.

```dart
import 'dart:math' as math;

class Vec2 {
  final double x;
  final double y;

  const Vec2(this.x, this.y);

  /// Les deux composantes identiques : Vec2.tout(3) == Vec2(3, 3).
  const Vec2.tout(double valeur)
      : x = valeur,
        y = valeur;

  /// Un vecteur de longueur 1 pointant dans la direction d'un angle (radians).
  factory Vec2.depuisAngle(double radians, [double longueur = 1]) {
    return Vec2(math.cos(radians) * longueur, math.sin(radians) * longueur);
  }

  static const Vec2 zero = Vec2(0, 0);
  static const Vec2 droite = Vec2(1, 0);
  static const Vec2 gauche = Vec2(-1, 0);
  static const Vec2 haut = Vec2(0, -1);   // y vers le bas : le haut est négatif
  static const Vec2 bas = Vec2(0, 1);

  @override
  String toString() =>
      '(${x.toStringAsFixed(2)}, ${y.toStringAsFixed(2)})';
}

void main() {
  print('tout(3)          : ${const Vec2.tout(3)}');
  print('haut             : ${Vec2.haut}');
  print('angle 0          : ${Vec2.depuisAngle(0)}');
  print('angle 90° (pi/2) : ${Vec2.depuisAngle(math.pi / 2)}');
}
```

**Résultat :**

```text
tout(3)          : (3.00, 3.00)
haut             : (0.00, -1.00)
angle 0          : (1.00, 0.00)
angle 90° (pi/2) : (0.00, 1.00)
```

Notez `Vec2.haut = Vec2(0, -1)`. Ce signe moins n'est pas une erreur : c'est la conséquence directe du repère du `Canvas` vu au chapitre 21, où l'axe `y` descend. Toute la physique du chapitre repose sur cette convention, et un oubli de ce signe est la cause n° 1 des héros qui « sautent vers le sol ».

```text
  REPÈRE DU CANVAS (rappel du chapitre 21)

  (0,0) ─────────────────────> x croissant
    │
    │        y NÉGATIF = vers le HAUT
    │              ▲
    │              │
    │        ●  le héros
    │              │
    │              ▼
    │        y POSITIF = vers le BAS
    ▼
  y croissant
```

---

## 23.4 — Addition, soustraction, multiplication par un scalaire

Trois opérations suffisent à faire tourner 90 % d'un jeu. Prenons-les une par une, avec un dessin avant le code.

### L'addition : enchaîner deux déplacements

Additionner deux vecteurs, c'est **mettre les flèches bout à bout**.

```text
  a = (3, 1)   b = (1, 2)   a + b = (4, 3)

      ────────>          ────────>─────>
        a                    a       b
                             └──────────┘
                                a + b
```

En composantes : on additionne les `x` entre eux et les `y` entre eux. C'est tout.

```text
  a + b = ( a.x + b.x , a.y + b.y )
```

Usage typique : `position + deplacement`.

### La soustraction : aller de l'un vers l'autre

Soustraire, c'est répondre à la question **« quel déplacement m'emmène de A vers B ? »**.

```text
  cible - depart = le vecteur qui va de depart vers cible

      depart ●
              ╲
               ╲  (cible - depart)
                ╲
                 ▼ ● cible
```

Retenez l'ordre, car c'est l'erreur classique : **arrivée moins départ**. Dans l'autre sens, votre gobelin fuira le héros au lieu de le poursuivre.

```text
  b - a = ( b.x - a.x , b.y - a.y )
```

### La multiplication par un scalaire : allonger ou raccourcir

Un **scalaire**, en mathématiques, est simplement un nombre seul (par opposition à un vecteur). Multiplier un vecteur par un scalaire, c'est **changer sa longueur sans changer sa direction**.

```text
  v = (2, 1)

  v * 1    ──────>
  v * 2    ─────────────>          (deux fois plus long, même direction)
  v * 0.5  ───>                    (deux fois plus court)
  v * -1   <──────                 (même longueur, direction OPPOSÉE)
```

```text
  v * k = ( v.x * k , v.y * k )
```

C'est cette opération qui rend possible `vitesse * dt` : la vitesse pointe dans une direction, `dt` la raccourcit à la portion de seconde écoulée.

Écrivons ces trois opérations sous forme de méthodes classiques, avant de les transformer en opérateurs à la section suivante.

```dart
class Vec2 {
  final double x;
  final double y;

  const Vec2(this.x, this.y);

  Vec2 plus(Vec2 autre) => Vec2(x + autre.x, y + autre.y);
  Vec2 moins(Vec2 autre) => Vec2(x - autre.x, y - autre.y);
  Vec2 fois(double k) => Vec2(x * k, y * k);

  @override
  String toString() =>
      '(${x.toStringAsFixed(2)}, ${y.toStringAsFixed(2)})';
}

void main() {
  const Vec2 position = Vec2(100, 200);
  const Vec2 vitesse = Vec2(60, -30);
  const double dt = 0.5;

  final Vec2 deplacement = vitesse.fois(dt);
  final Vec2 nouvelle = position.plus(deplacement);

  print('position     : $position');
  print('vitesse      : $vitesse');
  print('déplacement  : $deplacement');
  print('nouvelle pos : $nouvelle');

  const Vec2 coffre = Vec2(340, 120);
  print('héros -> coffre : ${coffre.moins(position)}');
}
```

**Résultat :**

```text
position     : (100.00, 200.00)
vitesse      : (60.00, -30.00)
déplacement  : (30.00, -15.00)
nouvelle pos : (130.00, 185.00)
héros -> coffre : (240.00, -80.00)
```

Le code fonctionne, mais lisez la ligne clé :

```dart
final Vec2 nouvelle = position.plus(vitesse.fois(dt));
```

Comparez avec ce que vous aimeriez écrire :

```dart
final Vec2 nouvelle = position + vitesse * dt;
```

La seconde version se lit comme la formule de physique elle-même. Dart permet exactement cela.

---

## 23.5 — Surcharger les opérateurs `+`, `-`, `*` en Dart

Dart autorise une classe à définir ce que signifient `+`, `-`, `*`, `/`, `==`, `[]`, `<`, et quelques autres, pour ses propres objets. On appelle cela la **surcharge d'opérateurs**.

La syntaxe est simple : le mot-clé `operator` suivi du symbole, à la place d'un nom de méthode.

```dart
Vec2 operator +(Vec2 autre) => Vec2(x + autre.x, y + autre.y);
//   ▲        ▲      ▲
//   │        │      └── le paramètre : l'opérande de droite
//   │        └───────── le symbole de l'opérateur
//   └────────────────── le type de retour
```

L'opérande de gauche est l'objet lui-même (`this`), l'opérande de droite est le paramètre.

Voici la classe `Vec2` avec ses opérateurs.

```dart
import 'dart:math' as math;

class Vec2 {
  final double x;
  final double y;

  const Vec2(this.x, this.y);

  static const Vec2 zero = Vec2(0, 0);

  Vec2 operator +(Vec2 autre) => Vec2(x + autre.x, y + autre.y);
  Vec2 operator -(Vec2 autre) => Vec2(x - autre.x, y - autre.y);
  Vec2 operator *(double k) => Vec2(x * k, y * k);
  Vec2 operator /(double k) => Vec2(x / k, y / k);

  /// Moins unaire : -v inverse la direction.
  Vec2 operator -() => Vec2(-x, -y);

  @override
  bool operator ==(Object autre) =>
      autre is Vec2 && autre.x == x && autre.y == y;

  @override
  int get hashCode => Object.hash(x, y);

  @override
  String toString() =>
      '(${x.toStringAsFixed(2)}, ${y.toStringAsFixed(2)})';
}

void main() {
  const Vec2 position = Vec2(100, 200);
  const Vec2 vitesse = Vec2(60, -30);
  const double dt = 0.5;

  print('position + vitesse * dt = ${position + vitesse * dt}');
  print('-vitesse                = ${-vitesse}');
  print('vitesse / 2             = ${vitesse / 2}');
  print('égalité                 : ${Vec2(1, 2) == Vec2(1, 2)}');
}
```

**Résultat :**

```text
position + vitesse * dt = (130.00, 185.00)
-vitesse                = (-60.00, 30.00)
vitesse / 2             = (30.00, -15.00)
égalité                 : true
```

Quatre points d'attention.

**La priorité des opérateurs est celle de Dart, pas la vôtre.** Dans `position + vitesse * dt`, la multiplication s'évalue avant l'addition, exactement comme en arithmétique. Vous n'avez rien à déclarer : Dart applique ses règles habituelles, vues au chapitre 03.

**`operator -()` sans paramètre est le moins unaire.** Dart distingue `a - b` (binaire, un paramètre) de `-a` (unaire, zéro paramètre). Les deux peuvent coexister dans la même classe.

**`==` doit toujours aller avec `hashCode`.** C'est la règle du chapitre 10. Si vous redéfinissez l'égalité sans redéfinir `hashCode`, vos vecteurs se comporteront mal dans un `Set` ou une clé de `Map`. `Object.hash(x, y)` fait le travail correctement.

**On ne peut pas surcharger `+=`.** Et ce n'est pas nécessaire : Dart déduit automatiquement `a += b` de `a = a + b` dès que `+` existe. En revanche, cela signifie que `+=` réaffecte la variable, donc votre champ ne peut pas être `final` :

```dart
Vec2 position = const Vec2(0, 0); // PAS final : on va la réaffecter
position += const Vec2(10, 0);    // équivaut à position = position + Vec2(10, 0)
```

C'est l'immuabilité du vecteur combinée à la mutabilité de la variable qui le contient. Le vecteur ne change pas ; c'est la variable qui pointe vers un nouveau vecteur.

> **Attention.** `Vec2 * Vec2` n'existe volontairement pas dans notre classe. Multiplier deux vecteurs n'a pas une seule définition évidente (produit scalaire ? composante par composante ?), et un opérateur ambigu est pire qu'une méthode nommée. Nous écrirons `a.scalaire(b)` en 23.9.

---

## 23.6 — Longueur d'un vecteur

La **longueur** d'un vecteur, aussi appelée **norme** ou **magnitude**, répond à la question : *quelle distance cette flèche couvre-t-elle ?*

### L'explication en mots

Un vecteur `(3, 4)` décrit un déplacement de 3 vers la droite puis de 4 vers le bas. Ces deux déplacements forment les deux côtés d'un triangle rectangle, et le vecteur lui-même en est l'hypoténuse.

```text
      (0,0) ●───────────── 3 ─────────────>
            │ ╲
            │   ╲
            4      ╲  longueur = ?
            │        ╲
            ▼          ╲
                         ● (3, 4)
```

Le théorème de Pythagore, que vous avez peut-être croisé au collège, dit que le carré de l'hypoténuse vaut la somme des carrés des deux autres côtés. Autrement dit :

```text
  longueur × longueur = x × x + y × y
  longueur = racine carrée de (x × x + y × y)
```

Avec `(3, 4)` : 3 × 3 + 4 × 4 = 9 + 16 = 25, et la racine carrée de 25 vaut 5. Le vecteur `(3, 4)` a une longueur de 5.

### La longueur au carré : l'optimisation gratuite

La racine carrée est l'une des opérations les plus coûteuses du processeur. Or, très souvent, on n'a pas besoin de la longueur elle-même mais seulement de **comparer** deux longueurs. Par exemple : « le gobelin est-il à moins de 100 pixels ? »

Dans ce cas, on compare les **carrés** et on économise la racine :

```text
  longueur < 100        équivaut à        longueurCarree < 100 × 100
```

C'est vrai parce que la fonction « élever au carré » conserve l'ordre pour les nombres positifs, et une longueur est toujours positive. Retenez ce réflexe : il vous fera gagner beaucoup de temps de calcul quand vous testerez 300 ennemis par frame.

```dart
import 'dart:math' as math;

class Vec2 {
  final double x;
  final double y;
  const Vec2(this.x, this.y);

  Vec2 operator -(Vec2 a) => Vec2(x - a.x, y - a.y);

  /// Longueur au carré : pas de racine carrée, donc rapide.
  double get longueurCarree => x * x + y * y;

  /// Longueur réelle, en pixels.
  double get longueur => math.sqrt(longueurCarree);

  @override
  String toString() =>
      '(${x.toStringAsFixed(2)}, ${y.toStringAsFixed(2)})';
}

void main() {
  const Vec2 a = Vec2(3, 4);
  print('a                : $a');
  print('a.longueurCarree : ${a.longueurCarree}');
  print('a.longueur       : ${a.longueur}');

  const Vec2 diagonale = Vec2(1, 1);
  print('diagonale        : ${diagonale.longueur.toStringAsFixed(4)}');

  // Test de portée SANS racine carrée.
  const Vec2 heros = Vec2(100, 100);
  const Vec2 gobelin = Vec2(160, 180);
  const double portee = 100;
  final bool aPortee = (gobelin - heros).longueurCarree < portee * portee;
  print('gobelin à portée : $aPortee');
}
```

**Résultat :**

```text
a                : (3.00, 4.00)
a.longueurCarree : 25.0
a.longueur       : 5.0
diagonale        : 1.4142
gobelin à portée : false
```

Cette dernière ligne mérite un arrêt. Le gobelin est à (160, 180), le héros à (100, 100) : l'écart vaut (60, 80). Or 60 × 60 + 80 × 80 = 3600 + 6400 = 10000, et la portée au carré vaut aussi 100 × 100 = 10000. Le test `10000 < 10000` est `false` : le gobelin est **exactement** à la limite, et l'opérateur `<` l'exclut.

Ce n'est pas un bug, c'est une décision que vous devez prendre consciemment. Si vous voulez inclure la limite, écrivez `<=`. C'est le genre de détail qui fait qu'un coup « touche parfois » et « rate parfois » sans raison apparente pour le joueur.

> **À retenir.** `longueur` coûte une racine carrée. `longueurCarree` n'en coûte aucune. Pour comparer, utilisez toujours la seconde.

---

## 23.7 — Normaliser un vecteur

**Normaliser** un vecteur, c'est le ramener à une longueur de exactement 1 en conservant sa direction.

### L'explication en mots

Imaginez une flèche de 5 pixels de long qui pointe vers le coffre. Vous voulez garder son orientation mais vous fichez de sa longueur : vous voulez juste savoir « vers où ». Il suffit de la raccourcir 5 fois, c'est-à-dire de diviser chaque composante par 5, sa longueur.

```text
  v = (3, 4)          longueur = 5

  normalisé = (3/5, 4/5) = (0.6, 0.8)

  longueur du normalisé = racine(0.36 + 0.64) = racine(1) = 1 
```

Un vecteur de longueur 1 s'appelle un **vecteur unitaire**, ou plus simplement, dans le vocabulaire des jeux, une **direction**.

```text
  AVANT                          APRÈS NORMALISATION

  ────────────────>              ───>
  longueur 5                     longueur 1
  direction : nord-est           direction : nord-est (IDENTIQUE)
```

### Le piège du vecteur nul

Que vaut la direction du vecteur `(0, 0)` ? La question n'a pas de sens : une flèche de longueur nulle ne pointe nulle part. Mathématiquement, on diviserait par zéro.

En Dart, diviser un `double` par 0 ne lève pas d'exception : cela donne `NaN` (Not a Number) ou `Infinity`. Et un `NaN` se propage : `NaN + 5` vaut `NaN`, `NaN * 0` vaut `NaN`. Votre héros disparaîtra de l'écran sans un seul message d'erreur. C'est l'un des bugs les plus déroutants du développement de jeu.

Notre `normalise` doit donc traiter ce cas explicitement.

```dart
import 'dart:math' as math;

class Vec2 {
  final double x;
  final double y;
  const Vec2(this.x, this.y);

  static const Vec2 zero = Vec2(0, 0);

  Vec2 operator *(double k) => Vec2(x * k, y * k);

  double get longueurCarree => x * x + y * y;
  double get longueur => math.sqrt(longueurCarree);

  /// Ramène le vecteur à une longueur de 1, en gardant sa direction.
  /// Renvoie le vecteur nul si la longueur est nulle : jamais de NaN.
  Vec2 get normalise {
    final double l = longueur;
    if (l == 0) return Vec2.zero;
    return Vec2(x / l, y / l);
  }

  /// Un vecteur de longueur 1 vers la même direction, puis remis à `n` pixels.
  Vec2 avecLongueur(double n) => normalise * n;

  @override
  String toString() =>
      '(${x.toStringAsFixed(2)}, ${y.toStringAsFixed(2)})';
}

void main() {
  const Vec2 v = Vec2(3, 4);
  print('v                  : $v');
  print('v.normalise        : ${v.normalise}');
  print('longueur du normal.: ${v.normalise.longueur}');
  print('v.avecLongueur(200): ${v.avecLongueur(200)}');

  // Le cas dangereux :
  print('zero.normalise     : ${Vec2.zero.normalise}');

  // Ce qui se passerait SANS la garde :
  print('sans garde         : ${0 / 0}');
}
```

**Résultat :**

```text
v                  : (3.00, 4.00)
v.normalise        : (0.60, 0.80)
longueur du normal.: 1.0
v.avecLongueur(200): (120.00, 160.00)
zero.normalise     : (0.00, 0.00)
sans garde         : NaN
```

La ligne `v.avecLongueur(200)` est l'une des plus utiles de tout le chapitre. Elle dit : *« va vers là-bas, à 200 pixels par seconde »*. C'est exactement la formule de tout déplacement dirigé :

```text
  vitesse = (cible - position).normalise * vitesseMax
             └────────┬────────┘  └───┬──┘   └───┬───┘
                 vers où          longueur 1   combien vite
```

Mémorisez cette ligne. Vous l'écrirez dans chaque jeu, pour les projectiles, les ennemis qui poursuivent, les objets aimantés.

> **Erreur classique.** Ne testez jamais `longueur == 0` avec des vecteurs qui ont subi des calculs flottants : privilégiez `longueurCarree < 0.000001` si vous voulez être robuste. Pour un vecteur construit à la main, `== 0` suffit.

---

## 23.8 — Pourquoi normaliser : le bug de la diagonale plus rapide

Voici le bug le plus célèbre du développement de jeu 2D. Presque tout le monde l'écrit une fois. Il ne provoque aucune erreur, aucun message : le jeu tourne, mais il est cassé.

### Le code fautif

```dart
// Version naïve : chaque touche ajoute sa contribution.
if (gauche) vitesseX = -200;
if (droite) vitesseX = 200;
if (haut)   vitesseY = -200;
if (bas)    vitesseY = 200;
```

Le joueur appuie sur « droite ». Il va à 200 px/s. Parfait.

Le joueur appuie sur « droite » **et** « haut ». Le vecteur vitesse vaut alors `(200, -200)`. Quelle est sa longueur ?

```text
  racine(200² + 200²) = racine(40000 + 40000) = racine(80000) ≈ 282.84
```

Le héros va à **282,84 px/s en diagonale**, soit 41 % plus vite qu'en ligne droite.

### Le schéma du bug

```text
  CE QUE LE JOUEUR ATTEND              CE QUE LE CODE FAIT

       ↑ 200                                ↑ 200
       │                                    │
       │                                    │      ↗ 282.84 (!!)
       │    ↗ 200                           │    ↗
       │  ↗                                 │  ↗
       └──────────> 200                     └──────────> 200

   toutes les directions            la diagonale est plus rapide
   à la même vitesse                de 41 %
```

Conséquences dans un vrai jeu, toutes constatées :

- les joueurs découvrent qu'en zigzaguant ils traversent le niveau plus vite ; le *speedrun* de votre jeu se joue en diagonale permanente ;
- un ennemi qui poursuit le héros le rattrape plus vite en diagonale qu'en ligne droite, ce qui rend son comportement incohérent ;
- vos réglages de difficulté ne veulent plus rien dire, puisque la vitesse réelle dépend de la direction.

### La correction

On sépare strictement **la direction** et **la vitesse**. On construit un vecteur de direction brut, on le **normalise**, puis on le multiplie par la vitesse voulue.

```dart
import 'dart:math' as math;

class Vec2 {
  final double x;
  final double y;
  const Vec2(this.x, this.y);
  static const Vec2 zero = Vec2(0, 0);

  Vec2 operator *(double k) => Vec2(x * k, y * k);
  double get longueur => math.sqrt(x * x + y * y);
  Vec2 get normalise {
    final double l = longueur;
    return l == 0 ? Vec2.zero : Vec2(x / l, y / l);
  }

  @override
  String toString() =>
      '(${x.toStringAsFixed(2)}, ${y.toStringAsFixed(2)})';
}

/// Construit une vitesse correcte à partir de 4 touches.
Vec2 vitesseDepuisTouches({
  required bool gauche,
  required bool droite,
  required bool haut,
  required bool bas,
  required double vitesseMax,
}) {
  double dx = 0;
  double dy = 0;
  if (gauche) dx -= 1;
  if (droite) dx += 1;
  if (haut) dy -= 1;
  if (bas) dy += 1;

  // (dx, dy) vaut par exemple (1, -1) : longueur 1.41, pas 1.
  final Vec2 direction = Vec2(dx, dy).normalise;
  return direction * vitesseMax;
}

void main() {
  const double v = 200;

  final Vec2 versDroite = vitesseDepuisTouches(
      gauche: false, droite: true, haut: false, bas: false, vitesseMax: v);
  final Vec2 versDiagonale = vitesseDepuisTouches(
      gauche: false, droite: true, haut: true, bas: false, vitesseMax: v);
  final Vec2 immobile = vitesseDepuisTouches(
      gauche: false, droite: false, haut: false, bas: false, vitesseMax: v);
  final Vec2 opposees = vitesseDepuisTouches(
      gauche: true, droite: true, haut: false, bas: false, vitesseMax: v);

  print('droite     : $versDroite  longueur = ${versDroite.longueur}');
  print('diagonale  : $versDiagonale  longueur = ${versDiagonale.longueur}');
  print('immobile   : $immobile  longueur = ${immobile.longueur}');
  print('gauche+dr. : $opposees  longueur = ${opposees.longueur}');
}
```

**Résultat :**

```text
droite     : (200.00, 0.00)  longueur = 200.0
diagonale  : (141.42, -141.42)  longueur = 200.00000000000003
immobile   : (0.00, 0.00)  longueur = 0.0
gauche+dr. : (0.00, 0.00)  longueur = 0.0
```

La diagonale vaut désormais 200 px/s, comme la ligne droite. Le `200.00000000000003` est l'imprécision normale des nombres flottants vue au chapitre 02 ; elle est sans conséquence sur un écran de pixels.

Remarquez aussi les deux dernières lignes : le vecteur nul est géré (immobile) et les touches opposées s'annulent proprement grâce à la garde de `normalise`. Sans cette garde, `gauche+droite` aurait produit `(NaN, NaN)` et le héros aurait disparu.

> **À retenir.** Direction et vitesse sont deux choses différentes. On normalise la direction, puis on multiplie par la vitesse. Jamais l'inverse.

---

## 23.9 — Produit scalaire, et à quoi il sert

Le **produit scalaire** (en anglais *dot product*) prend deux vecteurs et renvoie **un seul nombre**. Sa formule est étonnamment simple :

```text
  a · b = a.x × b.x + a.y × b.y
```

Ce nombre n'est ni une position ni une vitesse : c'est une mesure d'**alignement** entre les deux vecteurs.

### La lecture qui compte vraiment

Vous n'avez pas besoin de la théorie. Retenez ce tableau, il suffit à tous les usages de jeu.

| Signe du produit scalaire | Ce que cela veut dire | Image |
| --- | --- | --- |
| Positif | les deux vecteurs pointent globalement dans le même sens | → → |
| Zéro | les deux vecteurs sont perpendiculaires | → ↑ |
| Négatif | les deux vecteurs pointent globalement en sens opposé | → ← |

```text
  a ─────────>

     b ─────────>       a · b > 0    (même sens, aligné)

     b      ↑
            │           a · b = 0    (perpendiculaire)

     b <─────────       a · b < 0    (sens opposé)
```

Et si les deux vecteurs sont **normalisés**, le produit scalaire est exactement le cosinus de l'angle entre eux : il vaut 1 pour un alignement parfait, 0 à 90°, −1 à 180°. C'est une valeur entre −1 et 1 très commode.

### Trois usages concrets dans un jeu

**1. Le gobelin voit-il le héros ?** On compare la direction dans laquelle regarde le gobelin avec la direction vers le héros. Si le produit scalaire est supérieur à un seuil, le héros est dans le cône de vision.

**2. Le joueur frappe-t-il vers l'ennemi ?** Même principe : direction de l'attaque contre direction vers l'ennemi.

**3. Le rebond sur un mur.** La composante de la vitesse le long de la normale du mur se calcule avec un produit scalaire. Nous ne détaillerons pas la formule complète ici (elle revient au chapitre 24), mais sachez qu'elle en dépend.

```dart
import 'dart:math' as math;

class Vec2 {
  final double x;
  final double y;
  const Vec2(this.x, this.y);
  static const Vec2 zero = Vec2(0, 0);

  Vec2 operator -(Vec2 a) => Vec2(x - a.x, y - a.y);
  Vec2 operator *(double k) => Vec2(x * k, y * k);
  double get longueur => math.sqrt(x * x + y * y);
  Vec2 get normalise {
    final double l = longueur;
    return l == 0 ? Vec2.zero : Vec2(x / l, y / l);
  }

  /// Produit scalaire : mesure l'alignement entre deux vecteurs.
  double scalaire(Vec2 autre) => x * autre.x + y * autre.y;

  @override
  String toString() =>
      '(${x.toStringAsFixed(2)}, ${y.toStringAsFixed(2)})';
}

void main() {
  const Vec2 droite = Vec2(1, 0);

  print('droite · droite  = ${droite.scalaire(const Vec2(1, 0))}');
  print('droite · haut    = ${droite.scalaire(const Vec2(0, -1))}');
  print('droite · gauche  = ${droite.scalaire(const Vec2(-1, 0))}');

  // Le gobelin regarde vers la droite. Voit-il le héros ?
  const Vec2 gobelin = Vec2(100, 100);
  const Vec2 regard = Vec2(1, 0); // déjà normalisé
  const double cosSeuil = 0.5;    // cône de 60° de demi-angle

  void test(String nom, Vec2 heros) {
    final Vec2 vers = (heros - gobelin).normalise;
    final double d = regard.scalaire(vers);
    final bool visible = d > cosSeuil;
    print('$nom : scalaire = ${d.toStringAsFixed(2)} -> '
        '${visible ? 'VU' : 'non vu'}');
  }

  test('héros devant       ', const Vec2(300, 100));
  test('héros derrière     ', const Vec2(20, 100));
  test('héros au-dessus    ', const Vec2(100, 20));
  test('héros devant-droite', const Vec2(300, 200));
}
```

**Résultat :**

```text
droite · droite  = 1.0
droite · haut    = 0.0
droite · gauche  = -1.0
héros devant        : scalaire = 1.00 -> VU
héros derrière      : scalaire = -1.00 -> non vu
héros au-dessus     : scalaire = 0.00 -> non vu
héros devant-droite : scalaire = 0.89 -> VU
```

Le dernier cas est intéressant : le héros est à 300 en x et 200 en y, donc en diagonale par rapport au gobelin. L'écart est (200, 100), normalisé environ (0,894 ; 0,447). Le produit scalaire avec (1, 0) vaut 0,894, supérieur au seuil de 0,5 : il est dans le cône.

Pour régler ce seuil, retenez ces valeurs de cosinus :

| Demi-angle du cône | Seuil à utiliser | Type de vision |
| --- | --- | --- |
| 30° | 0.87 | très étroite, un projecteur |
| 45° | 0.71 | garde attentif |
| 60° | 0.50 | vision normale |
| 90° | 0.00 | tout ce qui est devant |
| 180° | −1.00 | voit partout |

> **Remarque.** Un produit scalaire ne coûte que deux multiplications et une addition. C'est incomparablement moins cher qu'un calcul d'angle avec `atan2` et une comparaison d'angles. Préférez-le systématiquement quand vous testez une orientation.

---

## 23.10 — Distance entre deux points

La distance entre deux points est simplement **la longueur du vecteur qui va de l'un à l'autre**. Vous savez déjà tout faire : soustraire, puis mesurer.

```text
  distance(a, b) = longueur(b - a)
```

```text
      a ●
         ╲
          ╲     b - a
           ╲
            ▼
              ● b

  la distance est la longueur de cette flèche
```

Et, comme en 23.6, on dispose d'une version au carré qui évite la racine.

```dart
import 'dart:math' as math;

class Vec2 {
  final double x;
  final double y;
  const Vec2(this.x, this.y);

  Vec2 operator -(Vec2 a) => Vec2(x - a.x, y - a.y);
  double get longueurCarree => x * x + y * y;
  double get longueur => math.sqrt(longueurCarree);

  double distanceVers(Vec2 autre) => (autre - this).longueur;
  double distanceCarreeVers(Vec2 autre) => (autre - this).longueurCarree;

  @override
  String toString() =>
      '(${x.toStringAsFixed(2)}, ${y.toStringAsFixed(2)})';
}

class Ennemi {
  final String nom;
  final Vec2 position;
  const Ennemi(this.nom, this.position);
}

void main() {
  const Vec2 heros = Vec2(200, 200);

  const List<Ennemi> ennemis = <Ennemi>[
    Ennemi('gobelin', Vec2(260, 280)),
    Ennemi('rat', Vec2(210, 205)),
    Ennemi('squelette', Vec2(500, 200)),
    Ennemi('boss', Vec2(200, 700)),
  ];

  for (final Ennemi e in ennemis) {
    final double d = heros.distanceVers(e.position);
    print('${e.nom.padRight(10)} : ${d.toStringAsFixed(1)} px');
  }

  // Le plus proche, SANS racine carrée (rappel du chapitre 14 : reduce).
  final Ennemi plusProche = ennemis.reduce((Ennemi a, Ennemi b) =>
      heros.distanceCarreeVers(a.position) <
              heros.distanceCarreeVers(b.position)
          ? a
          : b);
  print('cible la plus proche : ${plusProche.nom}');

  // Test de portée d'attaque : 80 pixels.
  const double portee = 80;
  final List<String> aPortee = ennemis
      .where((Ennemi e) =>
          heros.distanceCarreeVers(e.position) <= portee * portee)
      .map((Ennemi e) => e.nom)
      .toList();
  print('à portée d\'épée      : $aPortee');
}
```

**Résultat :**

```text
gobelin    : 100.0 px
rat        : 11.2 px
squelette  : 300.0 px
boss       : 500.0 px
cible la plus proche : rat
à portée d'épée      : [rat]
```

Le gobelin est à exactement 100 pixels : 60² + 80² = 10000, racine 100. Il n'entre pas dans la portée de 80.

> **À retenir.** Pour trier ou comparer des distances, utilisez toujours `distanceCarreeVers`. Réservez `distanceVers` à l'affichage ou aux calculs qui ont réellement besoin de la valeur en pixels.

---

## 23.11 — Angle d'un vecteur, `atan2`

Parfois, il faut connaître **la direction sous forme d'angle** : pour faire pivoter un sprite (chapitre 21, `canvas.rotate`), pour orienter le canon d'une tourelle, pour dessiner un cône de vision.

### Le problème du quadrant

Vous pourriez penser à utiliser l'arc tangente ordinaire, `atan(y / x)`. C'est une mauvaise idée, pour deux raisons :

1. si `x` vaut 0, on divise par zéro ;
2. `atan` ne sait pas distinguer `(1, 1)` de `(-1, -1)` : le rapport `y / x` vaut 1 dans les deux cas, alors que les directions sont opposées.

La bibliothèque `dart:math` fournit `atan2(y, x)`, qui prend les **deux** composantes séparément et renvoie l'angle correct dans les quatre quadrants.

```text
  ATTENTION À L'ORDRE : atan2(y, x) — le y d'abord.
```

### Le cercle des angles dans le repère du Canvas

Comme `y` descend, les angles positifs tournent dans le sens des aiguilles d'une montre à l'écran. C'est déroutant si vous avez fait des maths, mais parfaitement cohérent une fois admis.

```text
                 -pi/2  (soit -90°)
                   ▲  HAUT
                   │
                   │
   pi (180°) ──────┼──────> 0 (droite)
   GAUCHE          │
                   │
                   ▼  BAS
                 +pi/2  (soit +90°)

  Angles renvoyés par atan2 : entre -pi et +pi.
```

### Radians et degrés

`atan2` renvoie des **radians**. Un tour complet vaut `2 × pi` radians, soit environ 6,283. Retenez les conversions :

```text
  degrés  = radians × 180 / pi
  radians = degrés × pi / 180
```

```dart
import 'dart:math' as math;

class Vec2 {
  final double x;
  final double y;
  const Vec2(this.x, this.y);

  Vec2 operator -(Vec2 a) => Vec2(x - a.x, y - a.y);

  /// Angle du vecteur, en radians, entre -pi et +pi.
  double get angle => math.atan2(y, x);

  /// Le même angle en degrés, plus lisible pour déboguer.
  double get angleDegres => angle * 180 / math.pi;

  factory Vec2.depuisAngle(double radians, [double longueur = 1]) =>
      Vec2(math.cos(radians) * longueur, math.sin(radians) * longueur);

  @override
  String toString() =>
      '(${x.toStringAsFixed(2)}, ${y.toStringAsFixed(2)})';
}

void main() {
  const List<(String, Vec2)> cas = <(String, Vec2)>[
    ('droite', Vec2(1, 0)),
    ('bas', Vec2(0, 1)),
    ('gauche', Vec2(-1, 0)),
    ('haut', Vec2(0, -1)),
    ('bas-droite', Vec2(1, 1)),
    ('haut-droite', Vec2(1, -1)),
  ];

  for (final (String nom, Vec2 v) in cas) {
    print('${nom.padRight(12)} $v -> '
        '${v.angle.toStringAsFixed(3)} rad = '
        '${v.angleDegres.toStringAsFixed(1)}°');
  }

  print('---');

  // Aller-retour : angle -> vecteur -> angle.
  final Vec2 reconstruit = Vec2.depuisAngle(math.pi / 4);
  print('depuisAngle(pi/4) = $reconstruit');
  print('son angle         = ${reconstruit.angleDegres.toStringAsFixed(1)}°');

  // Orienter une tourelle vers le héros.
  const Vec2 tourelle = Vec2(100, 100);
  const Vec2 heros = Vec2(300, 200);
  final double vise = (heros - tourelle).angleDegres;
  print('tourelle vise     : ${vise.toStringAsFixed(1)}°');
}
```

**Résultat :**

```text
droite       (1.00, 0.00) -> 0.000 rad = 0.0°
bas          (0.00, 1.00) -> 1.571 rad = 90.0°
gauche       (-1.00, 0.00) -> 3.142 rad = 180.0°
haut         (0.00, -1.00) -> -1.571 rad = -90.0°
bas-droite   (1.00, 1.00) -> 0.785 rad = 45.0°
haut-droite  (1.00, -1.00) -> -0.785 rad = -45.0°
---
depuisAngle(pi/4) = (0.71, 0.71)
son angle         = 45.0°
tourelle vise     : 26.6°
```

`Vec2.depuisAngle` est l'opération inverse de `angle`. Les deux forment un couple : vous passez d'une représentation à l'autre selon les besoins.

| Vous avez | Vous voulez | Utilisez |
| --- | --- | --- |
| Un vecteur | Un angle pour `canvas.rotate` | `v.angle` |
| Un angle | Un vecteur de déplacement | `Vec2.depuisAngle(a) * vitesse` |
| Deux points | L'angle de l'un vers l'autre | `(b - a).angle` |

> **Piège.** Comparer deux angles directement est délicat, car `-179°` et `+179°` sont voisins alors que leurs valeurs numériques sont éloignées de 358. Si vous devez comparer des orientations, revenez au produit scalaire de 23.9 : il n'a pas ce problème.

---

## 23.12 — Du vecteur à la direction de déplacement

Rassemblons tout ce que nous savons pour répondre à la question la plus pratique de la section : **comment fait-on avancer une entité vers un point ?**

Le schéma est toujours le même, en trois temps.

```text
  ÉTAPE 1 : où est la cible par rapport à moi ?
      versCible = cible - position

  ÉTAPE 2 : dans quelle direction, sans notion de distance ?
      direction = versCible.normalise

  ÉTAPE 3 : à quelle vitesse ?
      vitesse = direction * vitesseMax
```

Et en une ligne compacte, celle que vous écrirez le plus souvent :

```dart
vitesse = (cible - position).normalise * vitesseMax;
```

Il y a toutefois un piège dans ce code, et il vaut la peine d'être vu maintenant : **le tremblement autour de la cible**.

Si le gobelin va à 200 px/s et qu'il ne reste que 2 pixels à parcourir, il dépasse la cible pendant la frame, se retourne à la frame suivante, dépasse à nouveau, et vibre indéfiniment sur place. On appelle cela l'*overshoot*.

```text
  SANS GARDE                             AVEC GARDE

  frame 1   ──────>  ● cible             frame 1   ────> ● cible
  frame 2   <──────                       frame 2   arrivé, on s'arrête
  frame 3   ──────>
  frame 4   <──────
  (le gobelin vibre sur la cible)
```

La correction consiste à ne jamais avancer plus loin que la distance restante.

```dart
import 'dart:math' as math;

class Vec2 {
  final double x;
  final double y;
  const Vec2(this.x, this.y);
  static const Vec2 zero = Vec2(0, 0);

  Vec2 operator +(Vec2 a) => Vec2(x + a.x, y + a.y);
  Vec2 operator -(Vec2 a) => Vec2(x - a.x, y - a.y);
  Vec2 operator *(double k) => Vec2(x * k, y * k);

  double get longueur => math.sqrt(x * x + y * y);
  Vec2 get normalise {
    final double l = longueur;
    return l == 0 ? Vec2.zero : Vec2(x / l, y / l);
  }

  @override
  String toString() =>
      '(${x.toStringAsFixed(2)}, ${y.toStringAsFixed(2)})';
}

/// Avance `position` vers `cible` d'au plus `vitesse * dt` pixels.
Vec2 avancerVers(Vec2 position, Vec2 cible, double vitesse, double dt) {
  final Vec2 versCible = cible - position;
  final double restant = versCible.longueur;
  final double pas = vitesse * dt;

  if (restant <= pas) return cible; // on arrive pile, sans dépasser
  return position + versCible.normalise * pas;
}

void main() {
  const Vec2 cible = Vec2(100, 0);
  Vec2 gobelin = Vec2(0, 0);
  const double dt = 0.1;
  const double vitesse = 200; // 20 px par pas de 0,1 s

  for (int i = 1; i <= 7; i++) {
    gobelin = avancerVers(gobelin, cible, vitesse, dt);
    print('pas $i : $gobelin');
  }
}
```

**Résultat :**

```text
pas 1 : (20.00, 0.00)
pas 2 : (40.00, 0.00)
pas 3 : (60.00, 0.00)
pas 4 : (80.00, 0.00)
pas 5 : (100.00, 0.00)
pas 6 : (100.00, 0.00)
pas 7 : (100.00, 0.00)
```

Le gobelin s'arrête exactement sur la cible et n'en bouge plus. Aucune vibration.

> **À retenir.** `(cible - position).normalise * vitesse` est la formule du déplacement dirigé. Ajoutez toujours une garde contre le dépassement quand la cible est un point fixe à atteindre.

---

## 23.13 — Intégration : `position += vitesse * dt`

Nous entrons dans le cœur du chapitre. Le mot **intégration** peut faire peur ; il désigne ici quelque chose de très simple : *faire avancer une valeur petit à petit, une frame après l'autre*.

### La classe `Vec2` complète (référence)

Avant d'aller plus loin, voici la version définitive de `Vec2`. C'est celle qui sera utilisée dans tous les exemples suivants du chapitre. Gardez-la sous la main, ou placez-la dans un fichier `lib/moteur/vec2.dart` de votre projet (organisation du chapitre 16).

```dart
import 'dart:math' as math;

/// Vecteur 2D immuable pour la logique de jeu.
/// Ne dépend pas de Flutter : testable en Dart pur.
class Vec2 {
  final double x;
  final double y;

  const Vec2(this.x, this.y);

  const Vec2.tout(double v)
      : x = v,
        y = v;

  factory Vec2.depuisAngle(double radians, [double longueur = 1]) =>
      Vec2(math.cos(radians) * longueur, math.sin(radians) * longueur);

  static const Vec2 zero = Vec2(0, 0);
  static const Vec2 droite = Vec2(1, 0);
  static const Vec2 gauche = Vec2(-1, 0);
  static const Vec2 haut = Vec2(0, -1);
  static const Vec2 bas = Vec2(0, 1);

  // --- opérateurs ---
  Vec2 operator +(Vec2 a) => Vec2(x + a.x, y + a.y);
  Vec2 operator -(Vec2 a) => Vec2(x - a.x, y - a.y);
  Vec2 operator *(double k) => Vec2(x * k, y * k);
  Vec2 operator /(double k) => Vec2(x / k, y / k);
  Vec2 operator -() => Vec2(-x, -y);

  @override
  bool operator ==(Object o) => o is Vec2 && o.x == x && o.y == y;

  @override
  int get hashCode => Object.hash(x, y);

  // --- mesures ---
  double get longueurCarree => x * x + y * y;
  double get longueur => math.sqrt(longueurCarree);
  double get angle => math.atan2(y, x);
  double get angleDegres => angle * 180 / math.pi;

  // --- transformations ---
  Vec2 get normalise {
    final double l = longueur;
    return l == 0 ? Vec2.zero : Vec2(x / l, y / l);
  }

  Vec2 avecLongueur(double n) => normalise * n;

  /// Limite la longueur du vecteur à `max`, sans changer sa direction.
  Vec2 limite(double max) =>
      longueurCarree > max * max ? normalise * max : this;

  /// Copie en changeant une seule composante.
  Vec2 copieAvec({double? x, double? y}) => Vec2(x ?? this.x, y ?? this.y);

  // --- combinaisons ---
  double scalaire(Vec2 a) => x * a.x + y * a.y;
  double distanceVers(Vec2 a) => (a - this).longueur;
  double distanceCarreeVers(Vec2 a) => (a - this).longueurCarree;

  /// Interpolation linéaire : t = 0 renvoie this, t = 1 renvoie `a`.
  Vec2 lerp(Vec2 a, double t) => Vec2(x + (a.x - x) * t, y + (a.y - y) * t);

  @override
  String toString() =>
      '(${x.toStringAsFixed(2)}, ${y.toStringAsFixed(2)})';
}
```

### La première ligne d'intégration

Le chapitre 20 vous a déjà fait écrire, en une dimension :

```dart
x += vitesse * dt;
```

Avec `Vec2`, c'est exactement la même ligne, mais elle traite les deux axes d'un coup :

```dart
position += vitesse * dt;
```

Lisons-la lentement.

```text
  position  +=  vitesse  *  dt
     │       │      │       │
     │       │      │       └── la fraction de seconde écoulée (ex. 0.016)
     │       │      └────────── px par seconde
     │       └───────────────── « ajoute à »
     └───────────────────────── px
```

L'analyse des unités confirme la formule : px/s × s = px. On ajoute bien des pixels à des pixels. **Vérifiez toujours vos unités de cette manière** : si le résultat n'est pas homogène, votre formule est fausse.

Voici un exemple console, où l'on regarde le héros traverser la salle.

```dart
// (Collez la classe Vec2 complète au-dessus.)
void main() {
  Vec2 position = const Vec2(20, 300);
  const Vec2 vitesse = Vec2(180, -40); // px/s
  const double dt = 1 / 60;            // 60 FPS

  print('frame |          position');
  for (int frame = 0; frame <= 60; frame += 15) {
    // On saute directement à la frame voulue pour l'affichage.
    final Vec2 p = position + vitesse * (dt * frame);
    print('${frame.toString().padLeft(5)} | $p');
  }

  // Et la vraie boucle, frame par frame :
  for (int frame = 0; frame < 60; frame++) {
    position += vitesse * dt;
  }
  print('après 60 frames (1 s) : $position');
}
```

**Résultat :**

```text
frame |          position
    0 | (20.00, 300.00)
   15 | (65.00, 290.00)
   30 | (110.00, 280.00)
   45 | (155.00, 270.00)
   60 | (200.00, 260.00)
après 60 frames (1 s) : (200.00, 260.00)
```

En une seconde, le héros a parcouru 180 pixels vers la droite et 40 pixels vers le haut. C'est exactement ce qu'annonçait la vitesse. La boucle frame par frame donne le même résultat que le calcul direct : c'est la garantie que le delta time fait son travail.

> **Remarque.** L'accumulation frame par frame introduit de minuscules erreurs d'arrondi flottant. Sur 60 frames elles sont invisibles ; sur 100 000 frames elles se voient. Aucun jeu ne s'en soucie, mais sachez que c'est la raison pour laquelle on ne calcule jamais une trajectoire de tir longue distance par accumulation.

---

## 23.14 — Intégration : `vitesse += acceleration * dt`

La deuxième ligne d'intégration suit exactement la même logique, un cran plus haut dans la chaîne de 23.1.

```dart
vitesse += acceleration * dt;
```

Vérification des unités : px/s² × s = px/s. On ajoute bien une vitesse à une vitesse.

```text
  ┌───────────────┐   × dt    ┌───────────────┐   × dt    ┌───────────────┐
  │ acceleration  │ ────────> │    vitesse    │ ────────> │   position    │
  │    px/s²      │  += ...   │     px/s      │  += ...   │      px       │
  └───────────────┘           └───────────────┘           └───────────────┘
```

Deux lignes. C'est **tout** le moteur physique de ce chapitre. Le reste n'est que du réglage de valeurs et des cas particuliers.

Observons une accélération constante vers la droite, comme un chariot de mine qui prend de la vitesse.

```dart
// (Collez la classe Vec2 complète au-dessus.)
void main() {
  Vec2 position = const Vec2(0, 200);
  Vec2 vitesse = Vec2.zero;
  const Vec2 acceleration = Vec2(300, 0); // px/s² vers la droite
  const double dt = 0.25;

  print('t (s) |     vitesse |      position');
  for (int i = 0; i <= 8; i++) {
    print('${(i * dt).toStringAsFixed(2).padLeft(5)} | '
        '${vitesse.toString().padLeft(11)} | '
        '${position.toString().padLeft(13)}');

    vitesse += acceleration * dt;
    position += vitesse * dt;
  }
}
```

**Résultat :**

```text
t (s) |     vitesse |      position
 0.00 | (0.00, 0.00) | (0.00, 200.00)
 0.25 | (75.00, 0.00) | (18.75, 200.00)
 0.50 | (150.00, 0.00) | (56.25, 200.00)
 0.75 | (225.00, 0.00) | (112.50, 200.00)
 1.00 | (300.00, 0.00) | (187.50, 200.00)
 1.25 | (375.00, 0.00) | (281.25, 200.00)
 1.50 | (450.00, 0.00) | (393.75, 200.00)
 1.75 | (525.00, 0.00) | (525.00, 200.00)
 2.00 | (600.00, 0.00) | (675.00, 200.00)
```

Trois observations à faire vous-même sur ce tableau.

**La vitesse monte linéairement.** +75 px/s à chaque quart de seconde, soit +300 px/s par seconde : c'est bien la valeur de l'accélération.

**La position monte de plus en plus vite.** Entre t = 0 et t = 0,25 le chariot parcourt 18,75 px. Entre t = 1,75 et t = 2, il en parcourt 150. Une accélération constante produit un mouvement **de plus en plus rapide**, pas un mouvement uniforme.

**La courbe est une parabole.** Si vous traciez `position` en fonction du temps, vous obtiendriez une courbe qui s'incurve vers le haut. C'est la forme de toute trajectoire sous accélération constante — et c'est pour cela que les sauts dans les jeux dessinent des paraboles.

```text
  POSITION SOUS ACCÉLÉRATION CONSTANTE

  position
     │                                        ●
     │                                   ●
     │                              ●
     │                        ●
     │                  ●
     │           ●
     │      ●
     │  ●
     └───────────────────────────────────────── temps
       les intervalles s'allongent : c'est une parabole
```

---

## 23.15 — L'ordre des deux lignes compte

Nous avons maintenant deux lignes à exécuter à chaque frame :

```dart
vitesse += acceleration * dt;   // ligne A
position += vitesse * dt;       // ligne B
```

Question : peut-on les inverser ?

Techniquement oui, le programme compilera et tournera. Physiquement, **le résultat ne sera pas le même**. Et cette différence, presque invisible sur une frame, se voit très bien sur un saut.

### Pourquoi la différence existe

Dans l'ordre A puis B, la ligne B utilise la vitesse **déjà mise à jour**, celle de la fin de la frame.
Dans l'ordre B puis A, la ligne B utilise la vitesse **de la frame précédente**, celle du début de la frame.

```text
  ORDRE A → B  (vitesse d'abord)

  début de frame        fin de frame
       v0 ──────────────> v1 = v0 + a·dt
                          │
                          └── on déplace avec v1  (la NOUVELLE)


  ORDRE B → A  (position d'abord)

  début de frame        fin de frame
       v0 ──────────────> v1 = v0 + a·dt
       │
       └── on déplace avec v0  (l'ANCIENNE)
```

Comparons sur une chute libre, à la main.

```dart
// (Collez la classe Vec2 complète au-dessus.)
void main() {
  const Vec2 g = Vec2(0, 1000); // gravité, px/s²
  const double dt = 0.1;
  const int pas = 5;

  // ORDRE 1 : vitesse puis position (semi-implicite).
  Vec2 p1 = Vec2.zero;
  Vec2 v1 = Vec2.zero;

  // ORDRE 2 : position puis vitesse (explicite).
  Vec2 p2 = Vec2.zero;
  Vec2 v2 = Vec2.zero;

  print('t (s) | vitesse d\'abord | position d\'abord |  exact');
  for (int i = 1; i <= pas; i++) {
    v1 += g * dt;
    p1 += v1 * dt;

    p2 += v2 * dt;
    v2 += g * dt;

    final double t = i * dt;
    final double exact = 0.5 * 1000 * t * t;

    print('${t.toStringAsFixed(1).padLeft(5)} | '
        '${p1.y.toStringAsFixed(1).padLeft(15)} | '
        '${p2.y.toStringAsFixed(1).padLeft(16)} | '
        '${exact.toStringAsFixed(1).padLeft(6)}');
  }
}
```

**Résultat :**

```text
t (s) | vitesse d'abord | position d'abord |  exact
  0.1 |            10.0 |              0.0 |    5.0
  0.2 |            30.0 |             10.0 |   20.0
  0.3 |            60.0 |             30.0 |   45.0
  0.4 |           100.0 |             60.0 |   80.0
  0.5 |           150.0 |            100.0 |  125.0
```

La colonne « exact » vient de la formule de physique de la chute libre, `0,5 × g × t²`, qui donne la vraie position d'un corps qui tombe. Elle sert d'arbitre.

Constat :

- « vitesse d'abord » **surestime** la chute (150 au lieu de 125) ;
- « position d'abord » **sous-estime** la chute (100 au lieu de 125) ;
- les deux erreurs sont de même taille, 25 pixels, mais de signes opposés.

### Ce que cela change dans un jeu

Prenez un saut. Avec « position d'abord », la toute première frame du saut ne déplace pas le personnage : sa vitesse initiale n'a pas encore servi. Le saut a un micro-retard, et le personnage semble « coller » au sol un instant.

Pire : avec « position d'abord », l'énergie du système a tendance à augmenter frame après frame. Un objet qui rebondit indéfiniment finit par rebondir **plus haut** que son point de départ, ce qui est absurde et très visible.

> **Règle.** Mettez toujours la vitesse à jour **avant** la position. C'est ce qu'on appelle l'intégration d'Euler semi-implicite, et c'est ce qu'utilisent la quasi-totalité des moteurs de jeu 2D, Flame compris.

---

## 23.16 — Euler explicite vs semi-implicite

Donnons leurs noms officiels aux deux ordres de la section précédente. Vous les rencontrerez dans toute la documentation de physique de jeu.

| Nom | Ordre des lignes | Comportement | Verdict |
| --- | --- | --- | --- |
| Euler **explicite** (ou « avant ») | position puis vitesse | ajoute de l'énergie, diverge | à éviter |
| Euler **semi-implicite** (ou « symplectique ») | vitesse puis position | stable, conserve l'énergie | **à utiliser** |

```text
  EULER EXPLICITE                    EULER SEMI-IMPLICITE

  position += vitesse * dt;          vitesse  += acceleration * dt;
  vitesse  += acceleration * dt;     position += vitesse * dt;

  ┌───────────────────────┐          ┌───────────────────────┐
  │ un rebond monte de    │          │ un rebond retombe     │
  │ plus en plus haut     │          │ à la même hauteur     │
  │ (énergie qui gonfle)  │          │ (énergie stable)      │
  └───────────────────────┘          └───────────────────────┘
```

Le mot « symplectique » vient de la mécanique ; retenez seulement sa conséquence pratique : **le semi-implicite ne fait pas exploser vos simulations**.

Il existe des méthodes plus précises encore — Verlet, Runge-Kutta d'ordre 4 — utilisées en simulation scientifique et dans certains moteurs 3D. Elles coûtent plus cher et n'apportent rien à un jeu de plateforme 2D, où l'objectif n'est pas la fidélité physique mais la **sensation**. Un saut de Mario ne respecte aucune loi de la physique réelle : il monte vite, il retombe plus vite encore, et c'est ce qui le rend agréable.

Démontrons la divergence du semi-implicite contre l'explicite sur un ressort, le cas d'école.

```dart
import 'dart:math' as math;

void main() {
  // Un ressort : l'accélération ramène toujours vers l'origine.
  const double raideur = 40; // 1/s²
  const double dt = 0.05;

  double pExp = 100, vExp = 0;   // Euler explicite
  double pSemi = 100, vSemi = 0; // Euler semi-implicite

  // L'amplitude d'une oscillation : elle DOIT rester constante.
  double amplitude(double p, double v) =>
      math.sqrt(p * p + v * v / raideur);

  print(' t (s) | amplitude explicite | amplitude semi-implicite');
  for (int i = 0; i <= 200; i++) {
    if (i % 40 == 0) {
      print('${(i * dt).toStringAsFixed(2).padLeft(6)} | '
          '${amplitude(pExp, vExp).toStringAsFixed(1).padLeft(19)} | '
          '${amplitude(pSemi, vSemi).toStringAsFixed(1).padLeft(24)}');
    }

    // Euler explicite : les DEUX lignes utilisent les valeurs du début
    // de frame, il faut donc mémoriser l'ancienne position.
    final double pAncien = pExp;
    pExp += vExp * dt;
    vExp += -raideur * pAncien * dt;

    // Euler semi-implicite : vitesse d'abord, puis position.
    vSemi += -raideur * pSemi * dt;
    pSemi += vSemi * dt;
  }
}
```

**Résultat :**

```text
 t (s) | amplitude explicite | amplitude semi-implicite
  0.00 |               100.0 |                    100.0
  2.00 |               672.7 |                     97.9
  4.00 |              4525.9 |                     96.0
  6.00 |             30448.2 |                     94.4
  8.00 |            204840.0 |                     93.4
 10.00 |           1378061.2 |                     92.9
```

Le verdict est sans appel. En dix secondes, l'amplitude de l'oscillateur explicite est multipliée par **treize mille** : le ressort explose. Celle du semi-implicite perd 7 %, et cette légère perte se stabilise au lieu de continuer.

C'est exactement ce qui arriverait à une balle qui rebondit, à un pendule, à un grappin élastique ou à un ressort de plateforme. La formule d'accélération est pourtant identique dans les deux cas : seule change la valeur utilisée pour la calculer.

> **À retenir.** Deux lignes, un ordre, une règle : `vitesse` d'abord, `position` ensuite. Écrivez-le une fois dans votre méthode `mettreAJour` et ne le remettez jamais en question.

---

## 23.17 — La gravité

La gravité n'est **rien d'autre qu'une accélération constante vers le bas**. Il n'y a aucune classe spéciale à écrire, aucune bibliothèque à installer. Une ligne suffit.

```dart
const Vec2 gravite = Vec2(0, 1200); // px/s², y positif = vers le bas
```

Dans la boucle :

```dart
vitesse += gravite * dt;
position += vitesse * dt;
```

### Choisir la valeur

La gravité réelle de la Terre vaut environ 9,81 m/s². Elle ne vous sert à rien : votre jeu ne travaille pas en mètres mais en pixels, et un jeu réaliste est presque toujours un jeu mou.

Voici des ordres de grandeur éprouvés pour un personnage d'une cinquantaine de pixels de haut.

| Gravité (px/s²) | Sensation | Type de jeu |
| --- | --- | --- |
| 300 | flottante, lunaire | jeu spatial, plongée |
| 600 | douce, aérienne | plateforme contemplatif |
| 1200 | équilibrée | plateforme classique |
| 2000 | sèche, nerveuse | plateforme d'action rapide |
| 3500 | très lourde | *hardcore*, précision |

Retenez le principe de réglage : la gravité et la force du saut se règlent **ensemble**. Doubler la gravité sans toucher au saut donne un personnage qui ne décolle plus.

### Une relation utile

Si vous voulez qu'un saut atteigne une hauteur précise, il existe une formule simple :

```text
  vitesse de saut = racine carrée de ( 2 × gravité × hauteur voulue )
```

En mots : plus vous voulez sauter haut, plus il faut partir vite, et cette relation n'est pas proportionnelle mais en racine carrée — sauter deux fois plus haut demande environ 1,41 fois plus de vitesse initiale.

Vérifions-le, puis regardons la chute complète.

```dart
import 'dart:math' as math;

void main() {
  const double gravite = 1200; // px/s²

  for (final double hauteur in <double>[50, 100, 150, 200]) {
    final double v0 = math.sqrt(2 * gravite * hauteur);
    print('hauteur ${hauteur.toStringAsFixed(0).padLeft(3)} px '
        '-> vitesse initiale ${v0.toStringAsFixed(1)} px/s '
        '-> durée de montée ${(v0 / gravite).toStringAsFixed(3)} s');
  }
}
```

**Résultat :**

```text
hauteur  50 px -> vitesse initiale 346.4 px/s -> durée de montée 0.289 s
hauteur 100 px -> vitesse initiale 489.9 px/s -> durée de montée 0.408 s
hauteur 150 px -> vitesse initiale 600.0 px/s -> durée de montée 0.500 s
hauteur 200 px -> vitesse initiale 692.8 px/s -> durée de montée 0.577 s
```

Ce tableau est un outil de game design : vous décidez « le héros doit franchir un mur de 100 pixels », et vous en déduisez la vitesse de saut à écrire dans le code, au lieu de tâtonner au hasard.

Voici maintenant un `main.dart` complet : une pièce du Donjon de Dart, un héros, et la gravité seule. Il tombe, et il traverse le bas de l'écran — c'est volontaire, nous ajouterons le sol à la section suivante.

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';
import 'dart:math' as math;

// ---------------------------------------------------------------- Vec2
class Vec2 {
  final double x;
  final double y;
  const Vec2(this.x, this.y);
  static const Vec2 zero = Vec2(0, 0);

  Vec2 operator +(Vec2 a) => Vec2(x + a.x, y + a.y);
  Vec2 operator -(Vec2 a) => Vec2(x - a.x, y - a.y);
  Vec2 operator *(double k) => Vec2(x * k, y * k);

  double get longueur => math.sqrt(x * x + y * y);
  Vec2 get normalise {
    final double l = longueur;
    return l == 0 ? Vec2.zero : Vec2(x / l, y / l);
  }

  Vec2 copieAvec({double? x, double? y}) => Vec2(x ?? this.x, y ?? this.y);

  @override
  String toString() => '(${x.toStringAsFixed(0)}, ${y.toStringAsFixed(0)})';
}

// ---------------------------------------------------------------- App
void main() => runApp(const DonjonApp());

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: EcranGravite(),
    );
  }
}

class EcranGravite extends StatefulWidget {
  const EcranGravite({super.key});

  @override
  State<EcranGravite> createState() => _EcranGraviteState();
}

class _EcranGraviteState extends State<EcranGravite>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  static const Vec2 gravite = Vec2(0, 1200);

  Vec2 _position = const Vec2(160, 60);
  Vec2 _vitesse = Vec2.zero;
  double _temps = 0;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_frame)..start();
  }

  void _frame(Duration maintenant) {
    final double dt =
        (maintenant - _precedent).inMicroseconds / 1000000.0;
    _precedent = maintenant;
    if (dt <= 0 || dt > 0.25) return; // garde du chapitre 20

    setState(() {
      _temps += dt;
      // Les deux lignes d'intégration, dans le bon ordre.
      _vitesse += gravite * dt;
      _position += _vitesse * dt;
    });
  }

  void _relancer() {
    setState(() {
      _position = const Vec2(160, 60);
      _vitesse = Vec2.zero;
      _temps = 0;
    });
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF161320),
      body: GestureDetector(
        onTap: _relancer,
        child: CustomPaint(
          painter: PeintreGravite(
            position: _position,
            vitesse: _vitesse,
            temps: _temps,
          ),
          child: const SizedBox.expand(),
        ),
      ),
    );
  }
}

class PeintreGravite extends CustomPainter {
  final Vec2 position;
  final Vec2 vitesse;
  final double temps;

  PeintreGravite({
    required this.position,
    required this.vitesse,
    required this.temps,
  });

  @override
  void paint(Canvas canvas, Size size) {
    final Paint heros = Paint()..color = const Color(0xFF6FB3E0);
    canvas.drawRect(
      Rect.fromCenter(
        center: Offset(position.x, position.y),
        width: 28,
        height: 40,
      ),
      heros,
    );

    // Flèche de vitesse : on divise par 4 pour qu'elle tienne à l'écran.
    final Paint fleche = Paint()
      ..color = const Color(0xFFE0B36F)
      ..strokeWidth = 3;
    canvas.drawLine(
      Offset(position.x, position.y),
      Offset(position.x + vitesse.x / 4, position.y + vitesse.y / 4),
      fleche,
    );

    _texte(canvas, 'GRAVITE SEULE (touchez pour relancer)',
        const Offset(16, 16));
    _texte(canvas, 't        = ${temps.toStringAsFixed(2)} s',
        const Offset(16, 38));
    _texte(canvas, 'position = $position', const Offset(16, 56));
    _texte(canvas, 'vitesse  = $vitesse', const Offset(16, 74));
  }

  void _texte(Canvas canvas, String s, Offset o) {
    final TextPainter tp = TextPainter(
      text: TextSpan(
        text: s,
        style: const TextStyle(
          color: Color(0xFFE8E4DC),
          fontSize: 13,
          fontFamily: 'monospace',
        ),
      ),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, o);
  }

  @override
  bool shouldRepaint(PeintreGravite old) => true;
}
```

**Résultat :**

```text
GRAVITE SEULE (touchez pour relancer)
t        = 0.62 s
position = (160, 292)
vitesse  = (0, 744)

Un carré bleu part du haut de l'écran, immobile.
Il descend d'abord lentement, puis de plus en plus vite.
Une barre orange part de son centre vers le bas et s'allonge
au fur et à mesure : c'est le vecteur vitesse.
Le carré finit par sortir de l'écran par le bas.
Une pression le replace en haut, vitesse remise à zéro.
```

La flèche orange est un outil de débogage précieux : **dessinez toujours vos vecteurs**. Un bug de physique invisible dans les chiffres devient évident quand on voit une flèche pointer dans la mauvaise direction.

---

## 23.18 — Le sol : arrêter la chute

Notre héros traverse le bas de l'écran. Il lui faut un sol.

Un sol, à ce stade, n'est pas une collision au sens du chapitre 24 : c'est simplement **une hauteur limite**. Trois choses se produisent quand on la franchit.

```text
  QUAND LE HÉROS PASSE SOUS LE SOL :

  1. on le REPLACE exactement sur le sol       position.y = solY
  2. on ANNULE sa vitesse verticale            vitesse.y = 0
  3. on note qu'il est AU SOL                  auSol = true
```

Les trois sont indispensables, et oublier l'une d'elles produit un bug caractéristique.

| Étape oubliée | Symptôme |
| --- | --- |
| 1 — replacer | le héros s'enfonce d'un peu plus à chaque frame |
| 2 — annuler la vitesse | le héros vibre sur le sol, ou reste collé sans pouvoir sauter |
| 3 — noter l'état | le héros peut sauter en plein vol, indéfiniment |

Et le test lui-même s'écrit avec `>=`, pas `==` :

```dart
// FAUX : on ne tombe presque jamais pile sur la valeur du sol.
if (position.y == solY) { ... }

// JUSTE : on teste le dépassement.
if (position.y >= solY) { ... }
```

C'est une conséquence directe des nombres flottants. Avec `dt` qui varie d'une frame à l'autre, la position ne vaudra jamais exactement `solY`. Le test d'égalité serait vrai environ une fois sur un milliard.

Voici la logique, isolée dans une méthode.

```dart
// (Collez la classe Vec2 complète au-dessus.)
class Heros {
  Vec2 position;
  Vec2 vitesse = Vec2.zero;
  bool auSol = false;

  static const Vec2 gravite = Vec2(0, 1200);
  static const double demiHauteur = 20; // le héros fait 40 px de haut

  Heros(this.position);

  void mettreAJour(double dt, double solY) {
    vitesse += gravite * dt;
    position += vitesse * dt;

    final double basDuHeros = position.y + demiHauteur;
    if (basDuHeros >= solY) {
      position = position.copieAvec(y: solY - demiHauteur); // 1. replacer
      vitesse = vitesse.copieAvec(y: 0);                    // 2. annuler
      auSol = true;                                         // 3. noter
    } else {
      auSol = false;
    }
  }
}

void main() {
  final Heros h = Heros(const Vec2(100, 0));
  const double solY = 300;
  const double dt = 0.1;

  for (int i = 0; i <= 8; i++) {
    print('t=${(i * dt).toStringAsFixed(1)} '
        'y=${h.position.y.toStringAsFixed(1).padLeft(6)} '
        'vy=${h.vitesse.y.toStringAsFixed(1).padLeft(6)} '
        'auSol=${h.auSol}');
    h.mettreAJour(dt, solY);
  }
}
```

**Résultat :**

```text
t=0.0 y=   0.0 vy=   0.0 auSol=false
t=0.1 y=  12.0 vy= 120.0 auSol=false
t=0.2 y=  36.0 vy= 240.0 auSol=false
t=0.3 y=  72.0 vy= 360.0 auSol=false
t=0.4 y= 120.0 vy= 480.0 auSol=false
t=0.5 y= 180.0 vy= 600.0 auSol=false
t=0.6 y= 252.0 vy= 720.0 auSol=false
t=0.7 y= 280.0 vy=   0.0 auSol=true
t=0.8 y= 280.0 vy=   0.0 auSol=true
```

À t = 0,7 s, la position calculée aurait été 336, donc bien au-delà du sol. Le code la ramène à 280 (soit 300 − 20, le sol moins la demi-hauteur) et remet `vy` à zéro. Le héros se pose et ne bouge plus.

> **Remarque importante.** Nous replaçons le héros **au niveau du sol**, pas à sa position précédente. Ces deux corrections semblent équivalentes ; elles ne le sont pas. Replacer à la position précédente laisse un espace variable entre le héros et le sol, et le personnage semble léviter de quelques pixels au hasard.

---

## 23.19 — Le saut : une impulsion vers le haut

Un saut n'est pas une accélération. C'est une **impulsion** : un changement brutal de vitesse, appliqué une seule fois, au moment où le joueur appuie.

```dart
if (toucheSaut && auSol) {
  vitesse = vitesse.copieAvec(y: -forceSaut); // AFFECTATION, pas +=
  auSol = false;
}
```

Trois points cruciaux dans ces trois lignes.

**Le signe moins.** `y` grandit vers le bas (chapitre 21), donc sauter veut dire donner une vitesse `y` **négative**. C'est l'erreur n° 1 des débutants : sans le moins, le héros est projeté dans le sol.

**Une affectation, pas une addition.** On écrit `vitesse.y = -forceSaut`, pas `vitesse.y -= forceSaut`. Si le joueur martèle la touche pendant la montée et que vous accumulez, il s'envolera. On **remplace** la vitesse verticale par la vitesse de saut.

**La condition `auSol`.** Sans elle, le joueur saute en l'air autant qu'il veut : c'est le bug des sauts infinis. Nous verrons en 23.20 comment autoriser un nombre contrôlé de sauts.

### La forme de la trajectoire

Une impulsion vers le haut, suivie d'une gravité constante, dessine une **parabole**. C'est la trajectoire de saut de tous les jeux de plateforme.

```text
  TRAJECTOIRE DE SAUT

                    ●●●●●          sommet : vy = 0
                 ●●       ●●
              ●●             ●●
            ●                   ●
          ●                       ●
        ●                           ●
  ─────●─────────────────────────────●───────  sol
      départ                      arrivée
      vy = -600                   vy = +600

  MONTÉE : vy négatif, qui augmente vers 0
  SOMMET : vy = 0 pendant un instant
  CHUTE  : vy positif, qui augmente
```

Retenez ce point : **au sommet du saut, la vitesse verticale vaut exactement zéro**. C'est un instant précieux, souvent utilisé pour changer de frame d'animation (chapitre 22) ou pour déclencher un effet.

Voici un `main.dart` complet : le héros du Donjon de Dart saute quand on touche l'écran.

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';
import 'dart:math' as math;

class Vec2 {
  final double x;
  final double y;
  const Vec2(this.x, this.y);
  static const Vec2 zero = Vec2(0, 0);

  Vec2 operator +(Vec2 a) => Vec2(x + a.x, y + a.y);
  Vec2 operator -(Vec2 a) => Vec2(x - a.x, y - a.y);
  Vec2 operator *(double k) => Vec2(x * k, y * k);
  double get longueur => math.sqrt(x * x + y * y);
  Vec2 copieAvec({double? x, double? y}) => Vec2(x ?? this.x, y ?? this.y);

  @override
  String toString() => '(${x.toStringAsFixed(0)}, ${y.toStringAsFixed(0)})';
}

void main() => runApp(const DonjonApp());

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) => const MaterialApp(
        debugShowCheckedModeBanner: false,
        home: EcranSaut(),
      );
}

class EcranSaut extends StatefulWidget {
  const EcranSaut({super.key});

  @override
  State<EcranSaut> createState() => _EcranSautState();
}

class _EcranSautState extends State<EcranSaut>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  static const Vec2 gravite = Vec2(0, 1600);
  static const double forceSaut = 620;
  static const double demiHauteur = 22;

  Vec2 _position = const Vec2(140, 100);
  Vec2 _vitesse = Vec2.zero;
  bool _auSol = false;
  double _solY = 400;

  // Mémoire de la trajectoire, pour la dessiner.
  final List<Offset> _trace = <Offset>[];

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_frame)..start();
  }

  void _frame(Duration maintenant) {
    final double dt = (maintenant - _precedent).inMicroseconds / 1000000.0;
    _precedent = maintenant;
    if (dt <= 0 || dt > 0.25) return;

    setState(() {
      _vitesse += gravite * dt;
      _position += _vitesse * dt;

      if (_position.y + demiHauteur >= _solY) {
        _position = _position.copieAvec(y: _solY - demiHauteur);
        _vitesse = _vitesse.copieAvec(y: 0);
        _auSol = true;
      } else {
        _auSol = false;
      }

      _trace.add(Offset(_position.x, _position.y));
      if (_trace.length > 180) _trace.removeAt(0);
    });
  }

  void _sauter() {
    if (!_auSol) return; // pas de saut en l'air
    setState(() {
      _vitesse = _vitesse.copieAvec(y: -forceSaut);
      _auSol = false;
      _trace.clear();
    });
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF161320),
      body: LayoutBuilder(
        builder: (BuildContext context, BoxConstraints c) {
          _solY = c.maxHeight - 80;
          return GestureDetector(
            onTapDown: (_) => _sauter(),
            child: CustomPaint(
              painter: PeintreSaut(
                position: _position,
                vitesse: _vitesse,
                auSol: _auSol,
                solY: _solY,
                trace: _trace,
              ),
              child: const SizedBox.expand(),
            ),
          );
        },
      ),
    );
  }
}

class PeintreSaut extends CustomPainter {
  final Vec2 position;
  final Vec2 vitesse;
  final bool auSol;
  final double solY;
  final List<Offset> trace;

  PeintreSaut({
    required this.position,
    required this.vitesse,
    required this.auSol,
    required this.solY,
    required this.trace,
  });

  @override
  void paint(Canvas canvas, Size size) {
    // Le sol.
    canvas.drawRect(
      Rect.fromLTWH(0, solY, size.width, size.height - solY),
      Paint()..color = const Color(0xFF3A3350),
    );

    // La trace de la trajectoire.
    final Paint pointTrace = Paint()..color = const Color(0x55E0B36F);
    for (final Offset o in trace) {
      canvas.drawCircle(o, 2, pointTrace);
    }

    // Le héros.
    canvas.drawRect(
      Rect.fromCenter(
        center: Offset(position.x, position.y),
        width: 30,
        height: 44,
      ),
      Paint()..color = auSol
          ? const Color(0xFF6FB3E0)
          : const Color(0xFF9AD86F),
    );

    _texte(canvas, 'TOUCHEZ POUR SAUTER', const Offset(16, 16));
    _texte(canvas, 'vy    = ${vitesse.y.toStringAsFixed(0)}',
        const Offset(16, 38));
    _texte(canvas, 'auSol = $auSol', const Offset(16, 56));
    _texte(
      canvas,
      vitesse.y.abs() < 20 && !auSol ? 'SOMMET DU SAUT' : '',
      const Offset(16, 74),
    );
  }

  void _texte(Canvas canvas, String s, Offset o) {
    final TextPainter tp = TextPainter(
      text: TextSpan(
        text: s,
        style: const TextStyle(
          color: Color(0xFFE8E4DC),
          fontSize: 13,
          fontFamily: 'monospace',
        ),
      ),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, o);
  }

  @override
  bool shouldRepaint(PeintreSaut old) => true;
}
```

**Résultat :**

```text
TOUCHEZ POUR SAUTER
vy    = -318
auSol = false
SOMMET DU SAUT

Un rectangle bleu repose sur une bande violette en bas de l'écran.
Une pression le fait bondir : il devient vert tant qu'il est en l'air,
monte d'environ 120 pixels, ralentit, marque un temps d'arrêt au sommet,
puis retombe de plus en plus vite et redevient bleu en touchant le sol.
Une traînée de points orange dessine la parabole du saut.
Toucher l'écran pendant le vol ne fait rien.
```

Le changement de couleur selon `auSol` est un autre outil de débogage à adopter : **rendez visible l'état interne**. Si le héros reste vert alors qu'il est manifestement posé, vous savez immédiatement où chercher.

---

## 23.20 — Le double saut

Le double saut consiste à autoriser un second bond en plein vol. Sa mise en œuvre est un simple **compteur**, et c'est l'occasion de généraliser proprement.

```text
  IDÉE : on ne demande plus « es-tu au sol ? »
         on demande « te reste-t-il des sauts ? »

  au sol            -> sautsRestants = sautsMax
  chaque saut       -> sautsRestants -= 1
  saut autorisé si  -> sautsRestants > 0
```

Avec `sautsMax = 1`, on retrouve le saut simple. Avec 2, on a le double saut. Avec 3, le triple. Un seul paramètre gouverne tout.

```dart
// (Collez la classe Vec2 complète au-dessus.)
class Heros {
  Vec2 position;
  Vec2 vitesse = Vec2.zero;
  bool auSol = false;

  int sautsMax = 2;      // 1 = saut simple, 2 = double saut
  int sautsRestants = 2;

  static const Vec2 gravite = Vec2(0, 1600);
  static const double forceSaut = 600;
  static const double demiHauteur = 22;

  Heros(this.position);

  void mettreAJour(double dt, double solY) {
    vitesse += gravite * dt;
    position += vitesse * dt;

    if (position.y + demiHauteur >= solY) {
      position = position.copieAvec(y: solY - demiHauteur);
      vitesse = vitesse.copieAvec(y: 0);
      if (!auSol) sautsRestants = sautsMax; // rechargement à l'atterrissage
      auSol = true;
    } else {
      auSol = false;
    }
  }

  /// Renvoie true si le saut a bien été déclenché.
  bool sauter() {
    if (sautsRestants <= 0) return false;
    vitesse = vitesse.copieAvec(y: -forceSaut);
    sautsRestants -= 1;
    auSol = false;
    return true;
  }
}

void main() {
  final Heros h = Heros(const Vec2(0, 378));
  const double solY = 400;
  const double dt = 1 / 60;

  // On force l'état initial « posé ».
  h.mettreAJour(dt, solY);

  print('saut 1 : ${h.sauter()} (restants = ${h.sautsRestants})');
  print('saut 2 : ${h.sauter()} (restants = ${h.sautsRestants})');
  print('saut 3 : ${h.sauter()} (restants = ${h.sautsRestants})');

  // On laisse retomber.
  int frames = 0;
  while (!h.auSol && frames < 600) {
    h.mettreAJour(dt, solY);
    frames++;
  }
  print('atterri après $frames frames, restants = ${h.sautsRestants}');
  print('saut 4 : ${h.sauter()} (restants = ${h.sautsRestants})');
}
```

**Résultat :**

```text
saut 1 : true (restants = 1)
saut 2 : true (restants = 0)
saut 3 : false (restants = 0)
atterri après 45 frames, restants = 2
saut 4 : true (restants = 1)
```

Le troisième saut est refusé. Après l'atterrissage, le compteur repart à 2.

### Le piège du rechargement

Notez la ligne :

```dart
if (!auSol) sautsRestants = sautsMax;
```

Le `if (!auSol)` est essentiel. Sans lui, `sautsRestants` serait remis à `sautsMax` **à chaque frame passée au sol**, y compris à la frame où le joueur vient de sauter — et le saut serait gratuit.

Pour être précis : la remise à zéro doit se produire **à l'instant de l'atterrissage**, pas pendant tout le séjour au sol. La variable `auSol` de la frame précédente sert donc de détecteur de transition, une technique que vous retrouverez partout dans les jeux.

```text
  DÉTECTER UNE TRANSITION, PAS UN ÉTAT

  frames :  air  air  air  SOL  sol  sol  sol
  auSol  :   F    F    F    T    T    T    T
                          ▲
                          └── ATTERRISSAGE : ici, une seule fois
```

### Deux détails de game feel

Beaucoup de jeux à double saut appliquent deux ajustements que vous pouvez ajouter en une ligne chacun :

1. **Le second saut est plus faible** que le premier — environ 80 % — pour éviter de monter trop haut ;
2. **Le second saut remet la vitesse à zéro avant l'impulsion**, ce qui donne un rebond franc même quand le personnage tombait déjà vite.

Le second point est déjà satisfait par notre code, puisque nous **affectons** `vitesse.y` au lieu de l'additionner. C'est un exemple de choix technique qui se révèle aussi être un bon choix de design.

---

## 23.21 — Le saut variable selon la durée d'appui

Dans les jeux de plateforme soignés, une pression brève fait un petit saut et une pression maintenue fait un grand saut. Cette nuance, appelée **saut variable**, change radicalement la sensation de contrôle.

### Comment on pense que ça marche (et pourquoi c'est faux)

L'idée naïve consiste à appliquer une force vers le haut tant que la touche est maintenue. Mauvaise approche : la hauteur devient imprévisible, et un joueur qui garde la touche s'envole.

### Comment ça marche vraiment

On procède en sens inverse : **on donne toujours l'impulsion maximale, puis on coupe la montée quand le joueur relâche**.

```text
  APPUI LONG                      APPUI COURT

        ●●●●                          ●●
      ●●    ●●                      ●   ●
    ●●        ●●                   ●     ●
   ●            ●                 ●       ●
  ●              ●               ●         ●
  ────────────────               ───────────

  impulsion -600                  impulsion -600
  jamais coupée                   coupée à -600 * 0.35 = -210
                                  au moment du relâchement
```

Concrètement, au relâchement de la touche :

```dart
if (vitesse.y < 0) {
  // Encore en montée : on écrête la vitesse verticale.
  vitesse = vitesse.copieAvec(y: vitesse.y * coupureSaut);
}
```

Le test `vitesse.y < 0` est indispensable : si le joueur relâche pendant la chute, il ne faut surtout rien couper, sinon il flotterait.

Le coefficient `coupureSaut` se règle entre 0 et 1 :

| Valeur | Effet |
| --- | --- |
| 0.0 | la montée s'arrête net, saut très court, sensation brutale |
| 0.3 | franc et lisible, valeur la plus courante |
| 0.5 | doux, écart faible entre petit et grand saut |
| 1.0 | aucun effet, saut à hauteur fixe |

Voici un `main.dart` complet où l'on peut comparer les deux sauts en gardant le doigt appuyé plus ou moins longtemps.

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

class Vec2 {
  final double x;
  final double y;
  const Vec2(this.x, this.y);
  static const Vec2 zero = Vec2(0, 0);
  Vec2 operator +(Vec2 a) => Vec2(x + a.x, y + a.y);
  Vec2 operator *(double k) => Vec2(x * k, y * k);
  Vec2 copieAvec({double? x, double? y}) => Vec2(x ?? this.x, y ?? this.y);
}

void main() => runApp(const DonjonApp());

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) => const MaterialApp(
        debugShowCheckedModeBanner: false,
        home: EcranSautVariable(),
      );
}

class EcranSautVariable extends StatefulWidget {
  const EcranSautVariable({super.key});

  @override
  State<EcranSautVariable> createState() => _EcranSautVariableState();
}

class _EcranSautVariableState extends State<EcranSautVariable>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  static const Vec2 gravite = Vec2(0, 2200);
  static const double forceSaut = 760;
  static const double coupureSaut = 0.3;
  static const double demiHauteur = 22;

  Vec2 _position = const Vec2(150, 300);
  Vec2 _vitesse = Vec2.zero;
  bool _auSol = false;
  double _solY = 420;

  double _hauteurMax = 0;
  double _dureeAppui = 0;
  bool _appuyé = false;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_frame)..start();
  }

  void _frame(Duration maintenant) {
    final double dt = (maintenant - _precedent).inMicroseconds / 1000000.0;
    _precedent = maintenant;
    if (dt <= 0 || dt > 0.25) return;

    setState(() {
      if (_appuyé) _dureeAppui += dt;

      _vitesse += gravite * dt;
      _position += _vitesse * dt;

      if (_position.y + demiHauteur >= _solY) {
        _position = _position.copieAvec(y: _solY - demiHauteur);
        _vitesse = _vitesse.copieAvec(y: 0);
        _auSol = true;
      } else {
        _auSol = false;
        final double h = _solY - demiHauteur - _position.y;
        if (h > _hauteurMax) _hauteurMax = h;
      }
    });
  }

  void _appuyer() {
    _appuyé = true;
    _dureeAppui = 0;
    if (!_auSol) return;
    setState(() {
      _vitesse = _vitesse.copieAvec(y: -forceSaut);
      _auSol = false;
      _hauteurMax = 0;
    });
  }

  void _relacher() {
    _appuyé = false;
    if (_vitesse.y < 0) {
      // Encore en montée : on écrête.
      setState(() {
        _vitesse = _vitesse.copieAvec(y: _vitesse.y * coupureSaut);
      });
    }
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF161320),
      body: LayoutBuilder(
        builder: (BuildContext context, BoxConstraints c) {
          _solY = c.maxHeight - 90;
          return GestureDetector(
            onTapDown: (_) => _appuyer(),
            onTapUp: (_) => _relacher(),
            onTapCancel: _relacher,
            child: CustomPaint(
              painter: PeintreSautVariable(
                position: _position,
                vitesse: _vitesse,
                solY: _solY,
                hauteurMax: _hauteurMax,
                dureeAppui: _dureeAppui,
                auSol: _auSol,
              ),
              child: const SizedBox.expand(),
            ),
          );
        },
      ),
    );
  }
}

class PeintreSautVariable extends CustomPainter {
  final Vec2 position;
  final Vec2 vitesse;
  final double solY;
  final double hauteurMax;
  final double dureeAppui;
  final bool auSol;

  PeintreSautVariable({
    required this.position,
    required this.vitesse,
    required this.solY,
    required this.hauteurMax,
    required this.dureeAppui,
    required this.auSol,
  });

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
      Rect.fromLTWH(0, solY, size.width, size.height - solY),
      Paint()..color = const Color(0xFF3A3350),
    );

    // Repère de la hauteur maximale atteinte.
    final double yMax = solY - 22 - hauteurMax;
    canvas.drawLine(
      Offset(0, yMax),
      Offset(size.width, yMax),
      Paint()
        ..color = const Color(0x66E0B36F)
        ..strokeWidth = 1,
    );

    canvas.drawRect(
      Rect.fromCenter(
        center: Offset(position.x, position.y),
        width: 30,
        height: 44,
      ),
      Paint()..color =
          auSol ? const Color(0xFF6FB3E0) : const Color(0xFF9AD86F),
    );

    _texte(canvas, 'APPUI COURT = PETIT SAUT / APPUI LONG = GRAND SAUT',
        const Offset(16, 16));
    _texte(canvas, 'duree appui  = ${dureeAppui.toStringAsFixed(2)} s',
        const Offset(16, 38));
    _texte(canvas, 'hauteur max  = ${hauteurMax.toStringAsFixed(0)} px',
        const Offset(16, 56));
    _texte(canvas, 'vy           = ${vitesse.y.toStringAsFixed(0)}',
        const Offset(16, 74));
  }

  void _texte(Canvas canvas, String s, Offset o) {
    final TextPainter tp = TextPainter(
      text: TextSpan(
        text: s,
        style: const TextStyle(
          color: Color(0xFFE8E4DC),
          fontSize: 12,
          fontFamily: 'monospace',
        ),
      ),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, o);
  }

  @override
  bool shouldRepaint(PeintreSautVariable old) => true;
}
```

**Résultat :**

```text
APPUI COURT = PETIT SAUT / APPUI LONG = GRAND SAUT
duree appui  = 0.08 s
hauteur max  = 41 px
vy           = 122

Une pression brève fait sauter le personnage d'une quarantaine de pixels.
Une pression maintenue jusqu'au sommet le fait monter à environ 130 pixels.
Une ligne orange horizontale marque la hauteur maximale du dernier saut,
ce qui permet de comparer d'un essai à l'autre.
```

Remarquez que `onTapUp` et `onTapCancel` appellent tous deux `_relacher`. Sans `onTapCancel`, un doigt qui glisse hors de la zone ne déclencherait jamais le relâchement, et le personnage garderait un grand saut par erreur. Ce genre d'oubli produit des bugs rares et difficiles à reproduire.

---

## 23.22 — Le coyote time et le jump buffering

Ces deux techniques n'ajoutent aucune règle physique : elles ajoutent de la **tolérance**. Elles sont invisibles quand elles fonctionnent, et c'est précisément leur but. Tous les bons jeux de plateforme les emploient.

### Le coyote time

Le nom vient du coyote des dessins animés, qui court dans le vide et ne tombe qu'après s'en être aperçu.

Le problème qu'il résout : le joueur appuie sur saut **juste après** avoir quitté la plateforme, à 3 ou 4 frames près. Physiquement, il n'est plus au sol, donc rien ne se passe. Il a pourtant la certitude d'avoir appuyé à temps, et il trouve les commandes « pas réactives ».

```text
  SANS COYOTE TIME

  frames : ... SOL  SOL  air  air  air ...
                              ▲
                              └── le joueur appuie ici : REFUSÉ

  AVEC COYOTE TIME (0,1 s = 6 frames)

  frames : ... SOL  SOL  air  air  air  air  air  air  air  air ...
                         └────── fenêtre de tolérance ─────┘
                              ▲
                              └── le joueur appuie ici : ACCEPTÉ
```

La mise en œuvre tient en un chronomètre à rebours :

```dart
if (auSol) {
  coyoteRestant = coyoteDuree;  // rechargé tant qu'on touche le sol
} else {
  coyoteRestant -= dt;          // décompte dès qu'on décolle
}

final bool peutSauter = coyoteRestant > 0;
```

Il faut penser à **remettre `coyoteRestant` à zéro au moment du saut**, sinon le joueur pourrait sauter deux fois dans la fenêtre de tolérance.

### Le jump buffering

C'est le problème symétrique : le joueur appuie **juste avant** de toucher le sol, à quelques frames près. Sans tolérance, l'appui est perdu et le joueur doit ré-appuyer. Sur un enchaînement de sauts rapides, c'est extrêmement frustrant.

```text
  SANS BUFFER

  frames : ... air  air  air  SOL  SOL ...
                    ▲
                    └── appui PERDU, il faut ré-appuyer

  AVEC BUFFER (0,12 s)

  frames : ... air  air  air  SOL  SOL ...
                    ▲          ▲
                    │          └── le saut se déclenche AUTOMATIQUEMENT
                    └── appui MÉMORISÉ
```

La mise en œuvre est un second chronomètre :

```dart
// À l'appui sur la touche :
bufferRestant = bufferDuree;

// À chaque frame :
bufferRestant -= dt;
if (bufferRestant > 0 && peutSauter) {
  sauter();
  bufferRestant = 0;    // on consomme le buffer
  coyoteRestant = 0;    // et la tolérance
}
```

### Les valeurs à utiliser

| Réglage | Valeur typique | En frames à 60 FPS |
| --- | --- | --- |
| Coyote time | 0,08 à 0,12 s | 5 à 7 frames |
| Jump buffer | 0,10 à 0,15 s | 6 à 9 frames |

Au-delà de 0,2 s, le joueur perçoit le décalage et le jeu semble mou ou imprécis. En dessous de 0,05 s, l'effet ne se sent plus.

Voici les deux mécanismes réunis dans une classe complète, testée en console pour bien voir la chronologie.

```dart
// (Collez la classe Vec2 complète au-dessus.)
class HerosTolerant {
  Vec2 position;
  Vec2 vitesse = Vec2.zero;
  bool auSol = false;

  static const Vec2 gravite = Vec2(0, 1600);
  static const double forceSaut = 600;
  static const double demiHauteur = 22;
  static const double coyoteDuree = 0.10;
  static const double bufferDuree = 0.12;

  double coyoteRestant = 0;
  double bufferRestant = 0;

  HerosTolerant(this.position);

  /// Le joueur a appuyé sur la touche de saut.
  void demanderSaut() {
    bufferRestant = bufferDuree;
  }

  void mettreAJour(double dt, double solY) {
    // 1. Chronomètres.
    bufferRestant -= dt;
    if (auSol) {
      coyoteRestant = coyoteDuree;
    } else {
      coyoteRestant -= dt;
    }

    // 2. Le saut est-il possible ET demandé ?
    if (bufferRestant > 0 && coyoteRestant > 0) {
      vitesse = vitesse.copieAvec(y: -forceSaut);
      auSol = false;
      bufferRestant = 0;
      coyoteRestant = 0;
    }

    // 3. Physique.
    vitesse += gravite * dt;
    position += vitesse * dt;

    // 4. Sol.
    if (position.y + demiHauteur >= solY) {
      position = position.copieAvec(y: solY - demiHauteur);
      vitesse = vitesse.copieAvec(y: 0);
      auSol = true;
    } else {
      auSol = false;
    }
  }
}

void main() {
  const double dt = 1 / 60;
  const double solY = 400;

  // Scénario : le héros marche puis quitte une plateforme.
  // Le joueur appuie 4 frames APRÈS avoir quitté le sol.
  final HerosTolerant h = HerosTolerant(const Vec2(0, 378));
  h.mettreAJour(dt, solY); // il est au sol

  print('--- coyote time ---');
  // On simule la sortie de plateforme : on remonte le sol très bas.
  const double solLoin = 4000;
  for (int i = 1; i <= 6; i++) {
    if (i == 4) {
      h.demanderSaut();
      print('frame $i : le joueur appuie (déjà en l\'air)');
    }
    h.mettreAJour(dt, solLoin);
    print('frame $i : vy = ${h.vitesse.y.toStringAsFixed(0).padLeft(5)} '
        'coyote = ${h.coyoteRestant.toStringAsFixed(3)}');
  }
}
```

**Résultat :**

```text
--- coyote time ---
frame 1 : vy =    27 coyote = 0.083
frame 2 : vy =    53 coyote = 0.067
frame 3 : vy =    80 coyote = 0.050
frame 4 : le joueur appuie (déjà en l'air)
frame 4 : vy =  -573 coyote = 0.000
frame 5 : vy =  -547 coyote = -0.017
frame 6 : vy =  -520 coyote = -0.033
```

À la frame 4, le héros tombait déjà depuis 3 frames. L'appui est pourtant accepté : `vy` passe brusquement de +80 à −573 (soit −600 puis la gravité de la frame). Le coyote time a fait son travail, et le joueur n'a rien remarqué — ce qui est exactement l'objectif.

> **Conseil de réglage.** Ajoutez toujours ces deux tolérances **après** avoir réglé la physique de base. Elles masquent les défauts de réactivité, mais elles ne corrigent pas une gravité mal choisie.

---

## 23.23 — Le frottement et l'amortissement

Sans frottement, un personnage qui a reçu une impulsion glisse indéfiniment. Il faut donc **retirer** de la vitesse à chaque frame. Deux méthodes coexistent, et elles ne donnent pas du tout le même ressenti.

### Méthode 1 : le frottement linéaire (soustractif)

On retire une quantité fixe de vitesse par seconde, comme un patin qui frotte.

```dart
final double perte = frottement * dt; // frottement en px/s²
if (vitesse.longueur <= perte) {
  vitesse = Vec2.zero;                // on s'arrête net, sans repartir en arrière
} else {
  vitesse -= vitesse.normalise * perte;
}
```

Le test `if` est capital. Sans lui, une vitesse de 5 px/s à qui l'on retire 20 px/s devient **−15 px/s** : l'objet repart en arrière. C'est le bug du « personnage qui recule quand on lâche la touche ».

Caractéristique : l'arrêt est **franc et à date fixe**. Un objet à 400 px/s avec un frottement de 800 px/s² s'arrête en exactement 0,5 seconde.

### Méthode 2 : l'amortissement (multiplicatif)

On multiplie la vitesse par un facteur inférieur à 1 à chaque frame, comme un objet dans l'eau.

```dart
vitesse = vitesse * 0.9; // NE FAITES PAS CELA
```

Cette écriture, très répandue, est **fausse** : elle dépend du nombre de frames, donc du framerate. C'est exactement l'erreur que le chapitre 20 vous a appris à traquer.

La forme correcte utilise une puissance :

```dart
vitesse = vitesse * math.pow(facteurParSeconde, dt).toDouble();
```

où `facteurParSeconde` est la fraction de vitesse qui **reste** après une seconde complète. Avec 0,1 : il reste 10 % de la vitesse au bout d'une seconde, quel que soit le framerate.

Caractéristique : le ralentissement est **doux et n'atteint jamais zéro** mathématiquement. On ajoute donc un seuil de coupure.

### La démonstration du bug de framerate

```dart
import 'dart:math' as math;

void main() {
  // Amortissement écrit « par frame » : dépend du framerate.
  double v60 = 200, v30 = 200;
  for (int i = 0; i < 60; i++) {
    v60 *= 0.9;
  }
  for (int i = 0; i < 30; i++) {
    v30 *= 0.9;
  }

  // Amortissement écrit « par seconde » : indépendant du framerate.
  double w60 = 200, w30 = 200;
  for (int i = 0; i < 60; i++) {
    w60 *= math.pow(0.1, 1 / 60).toDouble();
  }
  for (int i = 0; i < 30; i++) {
    w30 *= math.pow(0.1, 1 / 30).toDouble();
  }

  print('après 1 seconde :');
  print('  v *= 0.9 par frame   -> 60 FPS : ${v60.toStringAsFixed(2)}'
      '   30 FPS : ${v30.toStringAsFixed(2)}');
  print('  v *= pow(0.1, dt)    -> 60 FPS : ${w60.toStringAsFixed(2)}'
      '   30 FPS : ${w30.toStringAsFixed(2)}');
}
```

**Résultat :**

```text
après 1 seconde :
  v *= 0.9 par frame   -> 60 FPS : 0.36   30 FPS : 8.48
  v *= pow(0.1, dt)    -> 60 FPS : 20.00   30 FPS : 20.00
```

La première ligne montre un écart de plus de 20 fois entre deux machines. Sur la seconde, les deux valeurs sont identiques.

### Choisir sa méthode

| Situation | Méthode | Raison |
| --- | --- | --- |
| Personnage qui s'arrête au sol | linéaire | arrêt franc, prévisible |
| Glace, patinoire | amortissement faible (0,6) | glissade longue et douce |
| Objet dans l'eau | amortissement fort (0,02) | freinage rapide et mou |
| Caisse poussée | linéaire fort | doit s'arrêter net |
| Particule, débris | amortissement | jamais besoin d'un arrêt exact |

> **À retenir.** Un frottement écrit sans `dt` est un bug, même s'il « a l'air de marcher » sur votre machine.

---

## 23.24 — Vitesse maximale (`clamp`)

Une accélération constante fait grandir la vitesse sans limite. Un héros qui accélère pendant dix secondes traverserait l'écran en une frame — et passerait à travers les murs (voir le *tunneling* au chapitre 24).

Il faut donc **plafonner** la vitesse. Deux plafonnements différents existent, et il faut choisir sciemment.

### Plafonner chaque composante séparément

```dart
vitesse = Vec2(
  vitesse.x.clamp(-vMax, vMax).toDouble(),
  vitesse.y.clamp(-vMax, vMax).toDouble(),
);
```

Cela dessine un **carré** de vitesses autorisées : la diagonale reste plus rapide que les axes. C'est le retour du bug de 23.8.

### Plafonner la longueur du vecteur

```dart
vitesse = vitesse.limite(vMax); // méthode de notre classe Vec2
```

Cela dessine un **cercle** : toutes les directions sont limitées à la même vitesse. C'est presque toujours ce que vous voulez.

```text
  PAR COMPOSANTE (carré)          PAR LONGUEUR (cercle)

  ┌─────────────┐                      ╭───────╮
  │             │                     ╱         ╲
  │      ●      │                    │     ●     │
  │             │                     ╲         ╱
  └─────────────┘                      ╰───────╯

  la diagonale atteint             toutes les directions
  vMax × 1.41                      atteignent vMax
```

### Le piège de `clamp` en Dart

`clamp` est déclaré sur `num`, pas sur `double`. Son type de retour statique est donc `num`, même si vous lui donnez des `double`.

```dart
double v = 300;
double limite = v.clamp(0, 200);        // ERREUR de compilation : num -> double
double limite = v.clamp(0, 200).toDouble(); // correct
```

Le message d'erreur (`A value of type 'num' can't be assigned to a variable of type 'double'`) déroute beaucoup de débutants. Retenez le `.toDouble()`.

### Un cas particulier : plafonner la chute seule

Dans un jeu de plateforme, on limite souvent **uniquement la vitesse de chute**, pour que la montée reste libre.

```dart
if (vitesse.y > vitesseChuteMax) {
  vitesse = vitesse.copieAvec(y: vitesseChuteMax);
}
```

Cela s'appelle la **vitesse terminale**. Une valeur de 900 à 1200 px/s convient pour un personnage de 44 px de haut.

```dart
import 'dart:math' as math;

class Vec2 {
  final double x;
  final double y;
  const Vec2(this.x, this.y);
  static const Vec2 zero = Vec2(0, 0);
  Vec2 operator +(Vec2 a) => Vec2(x + a.x, y + a.y);
  Vec2 operator *(double k) => Vec2(x * k, y * k);
  double get longueurCarree => x * x + y * y;
  double get longueur => math.sqrt(longueurCarree);
  Vec2 get normalise {
    final double l = longueur;
    return l == 0 ? Vec2.zero : Vec2(x / l, y / l);
  }

  Vec2 limite(double max) =>
      longueurCarree > max * max ? normalise * max : this;

  @override
  String toString() =>
      '(${x.toStringAsFixed(1)}, ${y.toStringAsFixed(1)})';
}

void main() {
  const double vMax = 200;
  const Vec2 v = Vec2(180, 180); // longueur 254.6

  final Vec2 parComposante = Vec2(
    v.x.clamp(-vMax, vMax).toDouble(),
    v.y.clamp(-vMax, vMax).toDouble(),
  );
  final Vec2 parLongueur = v.limite(vMax);

  print('vitesse brute   : $v  longueur = '
      '${v.longueur.toStringAsFixed(1)}');
  print('par composante  : $parComposante  longueur = '
      '${parComposante.longueur.toStringAsFixed(1)}');
  print('par longueur    : $parLongueur  longueur = '
      '${parLongueur.longueur.toStringAsFixed(1)}');

  // Un vecteur déjà court n'est pas modifié.
  const Vec2 lent = Vec2(30, 40);
  print('vecteur lent    : ${lent.limite(vMax)}  (inchangé)');
}
```

**Résultat :**

```text
vitesse brute   : (180.0, 180.0)  longueur = 254.6
par composante  : (180.0, 180.0)  longueur = 254.6
par longueur    : (141.4, 141.4)  longueur = 200.0
vecteur lent    : (30.0, 40.0)  (inchangé)
```

La ligne « par composante » ne change rien du tout : 180 est déjà dans l'intervalle [−200, 200]. Pourtant la vitesse réelle vaut 254,6 px/s, au-dessus de la limite annoncée. La démonstration est faite.

La dernière ligne montre que `limite` renvoie `this` sans rien recalculer quand la longueur est déjà sous le plafond : aucun coût inutile, et surtout aucune modification parasite d'un vecteur qui allait bien.

---

## 23.25 — L'accélération et la décélération d'un personnage (game feel)

Nous savons faire bouger un personnage. Reste à le rendre **agréable à contrôler**. C'est le domaine du *game feel*, et il repose sur deux réglages seulement.

### Les trois modèles de contrôle

**Modèle 1 : vitesse instantanée.** On affecte directement la vitesse quand la touche est enfoncée.

```dart
vitesse = Vec2(entree.x * vitesseMax, vitesse.y);
```

Réactif au maximum, mais raide et mécanique. Convient aux jeux de puzzle et aux jeux à grille.

**Modèle 2 : accélération pure.** On ajoute une accélération, on plafonne, on freine avec du frottement.

```dart
vitesse += Vec2(entree.x * acceleration, 0) * dt;
```

Réaliste mais souvent mou : le personnage patine, et le joueur trouve les commandes en retard.

**Modèle 3 : accélération asymétrique.** On distingue trois régimes — accélérer, freiner, tourner — avec trois valeurs différentes. C'est ce qu'emploient les jeux de plateforme réussis.

```text
  LE JOUEUR APPUIE                          -> ACCELERATION
  LE JOUEUR RELÂCHE                         -> DECELERATION
  LE JOUEUR APPUIE DANS L'AUTRE SENS        -> DEMI-TOUR (le plus fort)
```

Le demi-tour doit être **le plus vif des trois**. Sans cela, changer de direction donne une impression de patinage sur glace, ce qui est pénalisant quand il faut esquiver un projectile.

### Les valeurs de référence

Pour un personnage de 44 px de haut avec `vitesseMax = 220 px/s` :

| Réglage | Valeur | Temps pour atteindre la vitesse max |
| --- | --- | --- |
| `acceleration` | 1400 px/s² | environ 0,16 s |
| `deceleration` | 1800 px/s² | arrêt en environ 0,12 s |
| `demiTour` | 2600 px/s² | inversion en environ 0,17 s |

Retenez la relation : **temps ≈ vitesseMax / accélération**. C'est ainsi qu'on règle par intention et non par tâtonnement.

```dart
/// Rapproche `courant` de `cible` d'au plus `taux * dt`, sans dépasser.
double versCible(double courant, double cible, double taux, double dt) {
  final double ecart = cible - courant;
  final double pas = taux * dt;
  if (ecart.abs() <= pas) return cible;
  return courant + (ecart > 0 ? pas : -pas);
}

void main() {
  const double vitesseMax = 220;
  const double acceleration = 1400;
  const double deceleration = 1800;
  const double demiTour = 2600;
  const double dt = 1 / 60;

  double vx = 0;
  double entree = 1; // le joueur pousse à droite

  double tauxPour(double vx, double entree) {
    if (entree == 0) return deceleration;
    if (vx != 0 && entree.sign != vx.sign) return demiTour;
    return acceleration;
  }

  print('phase       | frame | vx');
  for (int i = 1; i <= 30; i++) {
    if (i == 11) entree = 0;   // relâchement
    if (i == 21) entree = -1;  // demi-tour

    final String phase = entree == 1
        ? 'accélère  '
        : entree == 0
            ? 'relâche   '
            : 'demi-tour ';
    vx = versCible(vx, entree * vitesseMax, tauxPour(vx, entree), dt);

    if (i % 5 == 0) {
      print('$phase  |    ${i.toString().padLeft(2)} | '
          '${vx.toStringAsFixed(1).padLeft(7)}');
    }
  }
}
```

**Résultat :**

```text
phase       | frame | vx
accélère    |     5 |   116.7
accélère    |    10 |   220.0
relâche     |    15 |   70.0
relâche     |    20 |     0.0
demi-tour   |    25 | -216.7
demi-tour   |    30 |  -220.0
```

Lisez les durées : le personnage atteint 220 px/s en 10 frames (0,17 s), s'arrête en 8 frames environ, et repart en sens inverse à pleine vitesse en 10 frames. C'est nerveux sans être brutal.

> **Conseil.** Réglez toujours ces trois valeurs **en jouant**, pas en lisant des chiffres. Mais partez de ces ordres de grandeur : cela vous évitera des heures de tâtonnement.

---

## 23.26 — Le rebond et le coefficient de restitution

Faire rebondir un objet consiste à **inverser la composante de vitesse perpendiculaire à la surface**, puis à la réduire.

Pour un sol horizontal, seule `y` est concernée :

```dart
vitesse = vitesse.copieAvec(y: -vitesse.y * restitution);
```

Le **coefficient de restitution** est la fraction de vitesse conservée après le choc.

| Restitution | Comportement | Objet |
| --- | --- | --- |
| 0.0 | ne rebondit pas du tout | sac de sable, corps |
| 0.3 | rebond mou | caisse de bois |
| 0.6 | rebond franc | ballon de football |
| 0.85 | rebond très vif | balle de caoutchouc |
| 1.0 | rebondit indéfiniment à la même hauteur | irréaliste, utile pour un Pong |
| > 1.0 | monte de plus en plus haut | jamais, sauf effet volontaire |

### La hauteur des rebonds

Point important pour régler un jeu : la restitution s'applique à la **vitesse**, mais la hauteur atteinte dépend du **carré** de la vitesse (relation vue en 23.17). Un coefficient de 0,7 divise donc la hauteur non pas par 1/0,7 mais par 1/0,49 : chaque rebond monte à 49 % du précédent.

```text
  restitution 0.7 sur la vitesse

  hauteur 1 : 100 %      ●●●●
  hauteur 2 :  49 %        ●●
  hauteur 3 :  24 %         ●
  hauteur 4 :  12 %
  ────────────────────────────
```

### Le piège du rebond infini

Un objet avec une restitution non nulle rebondit un nombre infini de fois, avec des hauteurs de plus en plus petites. Numériquement, cela produit un tremblement permanent sur le sol qui consomme du calcul et se voit à l'écran.

La parade est un **seuil d'endormissement** :

```dart
if (vitesse.y.abs() < seuilRepos) {
  vitesse = vitesse.copieAvec(y: 0); // l'objet se pose définitivement
}
```

Une valeur de 40 à 80 px/s convient. En dessous de ce seuil, l'œil ne distingue plus le rebond d'un simple frémissement.

```dart
// (Collez la classe Vec2 complète au-dessus.)
void main() {
  const double gravite = 1600;
  const double restitution = 0.7;
  const double seuilRepos = 50;
  const double solY = 400;
  const double dt = 1 / 240; // pas fin pour ne rater aucun rebond

  double y = 100, vy = 0;
  int rebonds = 0;
  double hautMax = 0;

  for (int i = 0; i < 4000 && rebonds < 5; i++) {
    vy += gravite * dt;
    y += vy * dt;

    if (vy < 0) {
      final double h = solY - y;
      if (h > hautMax) hautMax = h;
    }

    if (y >= solY) {
      y = solY;
      if (vy.abs() < seuilRepos) {
        vy = 0;
        print('l\'objet se pose (endormi)');
        break;
      }
      vy = -vy * restitution;
      rebonds++;
      print('rebond $rebonds : vy = ${vy.toStringAsFixed(1).padLeft(7)} '
          '-> hauteur suivante ≈ '
          '${(vy * vy / (2 * gravite)).toStringAsFixed(1)} px');
      hautMax = 0;
    }
  }
}
```

**Résultat :**

```text
rebond 1 : vy =  -685.9 -> hauteur suivante ≈ 147.0
rebond 2 : vy =  -480.1 -> hauteur suivante ≈ 72.0
rebond 3 : vy =  -336.1 -> hauteur suivante ≈ 35.3
rebond 4 : vy =  -235.2 -> hauteur suivante ≈ 17.3
rebond 5 : vy =  -164.6 -> hauteur suivante ≈ 8.5
```

Chaque hauteur vaut à peu près 49 % de la précédente : 147, 72, 35, 17, 8. La règle du carré est vérifiée.

Les décimales exactes dépendent du pas de temps : avec `dt = 1/240`, l'objet dépasse le sol de quelques dixièmes de pixel avant d'être corrigé, ce qui décale les vitesses d'impact de moins d'un pour cent. Les proportions, elles, ne changent pas.

> **Pour aller plus loin.** Un rebond sur une surface inclinée demande de réfléchir la vitesse autour de la **normale** de la surface, ce qui utilise le produit scalaire de 23.9. Le chapitre 24, consacré aux collisions, donnera la formule complète.

---

## 23.27 — Mouvement circulaire

Certains éléments ne se déplacent pas en ligne droite : une plateforme qui tourne, une chauve-souris qui décrit des cercles, un familier qui gravite autour du héros, une lame rotative de piège.

Le principe est simple : **on ne fait pas bouger la position, on fait bouger un angle**, et la position en découle.

```dart
angle += vitesseAngulaire * dt;                       // radians par seconde
position = centre + Vec2.depuisAngle(angle) * rayon;  // on RECALCULE la position
```

Le mot important est **recalcule**. On n'écrit pas `position += quelque chose` : la position est entièrement déterminée par l'angle. C'est une différence de nature avec tout ce que nous avons fait jusqu'ici.

```text
  MOUVEMENT CIRCULAIRE

              angle croissant
                   ↘
          ●─────────────●
        ╱                 ╲
       ●        + centre    ●   rayon
        ╲                 ╱
          ●─────────────●

  position = centre + (cos(angle), sin(angle)) * rayon
```

### Régler la vitesse angulaire

Un tour complet vaut `2 × pi` radians, soit environ 6,283.

| Vitesse angulaire | Durée d'un tour |
| --- | --- |
| `2 * pi` | 1 seconde (très rapide) |
| `pi` | 2 secondes |
| `pi / 2` | 4 secondes |
| `pi / 4` | 8 secondes (lent, décoratif) |

Retenez : `durée d'un tour = 2 × pi / vitesseAngulaire`.

### Ellipses et orbites décalées

Deux variantes s'obtiennent en une ligne :

```dart
// Ellipse : deux rayons différents.
position = Vec2(centre.x + math.cos(angle) * rayonX,
                centre.y + math.sin(angle) * rayonY);

// Plusieurs satellites répartis : on décale l'angle de départ.
final double angleI = angle + i * 2 * math.pi / nombre;
```

```dart
import 'dart:math' as math;

class Vec2 {
  final double x;
  final double y;
  const Vec2(this.x, this.y);
  Vec2 operator +(Vec2 a) => Vec2(x + a.x, y + a.y);
  Vec2 operator *(double k) => Vec2(x * k, y * k);
  factory Vec2.depuisAngle(double r, [double l = 1]) =>
      Vec2(math.cos(r) * l, math.sin(r) * l);
  @override
  String toString() =>
      '(${x.toStringAsFixed(1)}, ${y.toStringAsFixed(1)})';
}

void main() {
  const Vec2 centre = Vec2(200, 200);
  const double rayon = 80;
  const double vitesseAngulaire = math.pi; // un demi-tour par seconde
  const double dt = 0.25;
  const int satellites = 3;

  double angle = 0;
  for (int i = 0; i <= 4; i++) {
    final List<String> pos = <String>[];
    for (int s = 0; s < satellites; s++) {
      final double a = angle + s * 2 * math.pi / satellites;
      pos.add((centre + Vec2.depuisAngle(a) * rayon).toString());
    }
    print('t=${(i * dt).toStringAsFixed(2)} '
        'angle=${angle.toStringAsFixed(2)} : ${pos.join('  ')}');
    angle += vitesseAngulaire * dt;
  }
}
```

**Résultat :**

```text
t=0.00 angle=0.00 : (280.0, 200.0)  (160.0, 269.3)  (160.0, 130.7)
t=0.25 angle=0.79 : (256.6, 256.6)  (122.7, 220.7)  (220.7, 122.7)
t=0.50 angle=1.57 : (200.0, 280.0)  (130.7, 160.0)  (269.3, 160.0)
t=0.75 angle=2.36 : (143.4, 256.6)  (179.3, 122.7)  (277.3, 220.7)
t=1.00 angle=3.14 : (120.0, 200.0)  (240.0, 130.7)  (240.0, 269.3)
```

Trois satellites tournent, espacés de 120 degrés, et le premier a fait exactement un demi-tour en une seconde (de x = 280 à x = 120). Conforme à `vitesseAngulaire = pi`.

> **Attention à l'accumulation.** `angle` grandit indéfiniment. Après une heure de jeu il vaut plusieurs dizaines de milliers, ce qui dégrade la précision des `cos` et `sin`. Si votre jeu tourne longtemps, ramenez-le dans l'intervalle : `angle = angle % (2 * math.pi);`.

---

## 23.28 — Interpolation linéaire (`lerp`)

L'**interpolation linéaire**, abrégée *lerp* (pour *linear interpolation*), répond à la question : *quelle valeur se trouve à x % du chemin entre A et B ?*

```text
  lerp(a, b, t) = a + (b - a) × t
```

Le paramètre `t` va de 0 à 1 :

```text
  t = 0     ●───────────────────────────  on est sur A
  t = 0.25  ────●────────────────────────  un quart du chemin
  t = 0.5   ─────────────●───────────────  au milieu
  t = 1     ───────────────────────────●  on est sur B
```

C'est l'une des fonctions les plus utiles de tout le développement de jeu. Elle sert pour :

| Usage | Ce qu'on interpole |
| --- | --- |
| Barre de vie qui descend en douceur | un nombre |
| Fondu de couleur | une couleur |
| Caméra qui rattrape le héros | une position |
| Zoom progressif | un facteur d'échelle |
| Transition entre deux états d'animation | une opacité |

### La confusion à éviter

`lerp` **n'est pas un mouvement**. C'est une formule qui ne connaît pas le temps. Ce qui produit le mouvement, c'est la façon dont vous faites évoluer `t`. Deux usages très différents en découlent :

**Usage 1 — `t` progresse de 0 à 1 :** vous obtenez un déplacement à vitesse constante entre deux points fixes.

```dart
t += dt / duree;
position = depart.lerp(arrivee, t.clamp(0, 1).toDouble());
```

**Usage 2 — `t` reste constant :** vous obtenez un rapprochement asymptotique, de plus en plus lent. C'est le sujet de la section 23.29.

```dart
position = position.lerp(cible, 0.1); // 10 % de l'écart restant, chaque frame
```

Ces deux usages produisent des courbes totalement différentes.

```text
  USAGE 1 : t de 0 à 1                USAGE 2 : t constant

  ●───●───●───●───●───●               ●──────●─────●───●──●─●●
  progression régulière               grands pas puis petits pas
```

```dart
// (Collez la classe Vec2 complète au-dessus.)
double lerp(double a, double b, double t) => a + (b - a) * t;

void main() {
  print('--- usage 1 : t progresse ---');
  const Vec2 depart = Vec2(0, 0);
  const Vec2 arrivee = Vec2(100, 50);
  for (double t = 0; t <= 1.001; t += 0.25) {
    print('t=${t.toStringAsFixed(2)} -> ${depart.lerp(arrivee, t)}');
  }

  print('--- usage 2 : t constant ---');
  Vec2 p = const Vec2(0, 0);
  for (int i = 1; i <= 5; i++) {
    p = p.lerp(arrivee, 0.4);
    print('frame $i -> $p');
  }

  print('--- lerp sur un nombre ---');
  double vieAffichee = 100;
  const double vieReelle = 30;
  for (int i = 1; i <= 4; i++) {
    vieAffichee = lerp(vieAffichee, vieReelle, 0.5);
    print('barre de vie : ${vieAffichee.toStringAsFixed(1)}');
  }
}
```

**Résultat :**

```text
--- usage 1 : t progresse ---
t=0.00 -> (0.00, 0.00)
t=0.25 -> (25.00, 12.50)
t=0.50 -> (50.00, 25.00)
t=0.75 -> (75.00, 37.50)
t=1.00 -> (100.00, 50.00)
--- usage 2 : t constant ---
frame 1 -> (40.00, 20.00)
frame 2 -> (64.00, 32.00)
frame 3 -> (78.40, 39.20)
frame 4 -> (87.04, 43.52)
frame 5 -> (92.22, 46.11)
--- lerp sur un nombre ---
barre de vie : 65.0
barre de vie : 47.5
barre de vie : 38.8
barre de vie : 34.4
```

Vérifiez la dernière série à la main, c'est un réflexe à prendre : 100, puis 65, puis 47,5, puis 38,75, puis 34,375. La barre se rapproche de 30 sans jamais l'atteindre. C'est la signature de l'usage 2, et cela impose la garde d'arrivée dont nous parlons à la section suivante.

---

## 23.29 — Suivre une cible en douceur

Le lerp à `t` constant est la base du **suivi doux**, utilisé pour les caméras (chapitre 25), les familiers, les curseurs de visée et les barres d'interface.

```dart
position = position.lerp(cible, 0.1); // NE FAITES PAS CELA TEL QUEL
```

Cette ligne, extrêmement répandue, souffre du même défaut que l'amortissement de 23.23 : **elle dépend du framerate**. À 120 FPS, la caméra rattrape deux fois plus vite qu'à 60 FPS.

### La version correcte

On remplace le facteur fixe par une formule qui intègre `dt` :

```dart
final double t = 1 - math.exp(-lambda * dt);
position = position.lerp(cible, t);
```

`lambda` est une **vitesse de rattrapage**, exprimée en « unités par seconde ». Plus il est grand, plus le suivi est vif.

| `lambda` | Sensation | Usage |
| --- | --- | --- |
| 2 | très mou, traîne beaucoup | caméra contemplative |
| 5 | doux | caméra de plateforme |
| 8 | équilibré | valeur de départ recommandée |
| 15 | serré | caméra d'action, visée |
| 30 | quasi instantané | barre de vie |

Retenez une équivalence utile : avec `lambda`, l'écart restant est divisé par environ 2,7 chaque seconde... et plus concrètement, **l'objet parcourt 63 % du chemin en `1 / lambda` seconde**. Avec `lambda = 8`, cela fait 0,125 s.

### Ne jamais oublier la garde d'arrivée

Comme le lerp n'atteint jamais exactement la cible, il faut un seuil, sans quoi la caméra « travaille » indéfiniment et votre `shouldRepaint` renvoie toujours vrai.

```dart
if (position.distanceCarreeVers(cible) < 0.25) position = cible;
```

```dart
import 'dart:math' as math;

void main() {
  const double cible = 100;
  const double lambda = 8;
  const double dt = 1 / 60;

  double p = 0;
  print('frame |   position | reste');
  for (int i = 1; i <= 60; i++) {
    final double t = 1 - math.exp(-lambda * dt);
    p = p + (cible - p) * t;
    if (i == 5 || i == 10 || i == 20 || i == 40 || i == 60) {
      print('${i.toString().padLeft(5)} | '
          '${p.toStringAsFixed(2).padLeft(10)} | '
          '${(cible - p).toStringAsFixed(2)}');
    }
  }
}
```

**Résultat :**

```text
frame |   position | reste
    5 |      48.66 | 51.34
   10 |      73.64 | 26.36
   20 |      93.05 | 6.95
   40 |      99.52 | 0.48
   60 |      99.97 | 0.03
```

À la frame 8, soit 0,133 s, on est aux alentours de 65 % : conforme à la règle des 63 % en `1 / lambda` seconde. Le rattrapage est rapide au début et ralentit ensuite, ce qui est exactement la sensation attendue d'une caméra de qualité.

> **Astuce de caméra.** Beaucoup de jeux utilisent deux `lambda` différents : un petit sur l'axe vertical (pour ne pas suivre chaque saut) et un grand sur l'axe horizontal (pour ne pas perdre le héros). Vous pourrez le faire au chapitre 25.

---

## 23.30 — Le tremblement d'écran (screen shake)

Le tremblement d'écran est un effet de **rendu**, pas de logique. Le monde ne bouge pas ; c'est la caméra qui vibre. Cette distinction est fondamentale : si vous secouez les positions réelles des entités, vous casserez vos collisions.

### Le principe

```text
  1. un événement déclenche le shake  -> intensite = 12 px
  2. à chaque frame, l'intensité décroît -> intensite -= vitesseDecroissance * dt
  3. au rendu, on décale TOUT le dessin d'un offset aléatoire de cette taille
     canvas.translate(dx, dy)
```

Le point 3 doit être encadré par `canvas.save()` et `canvas.restore()`, comme vu au chapitre 21, sinon le décalage s'accumulerait sur l'interface.

### Le réglage

| Événement | Intensité initiale | Durée |
| --- | --- | --- |
| Pas du héros | 0 (jamais) | — |
| Coup d'épée qui touche | 3 px | 0,10 s |
| Le héros reçoit un dégât | 8 px | 0,20 s |
| Explosion de baril | 14 px | 0,35 s |
| Attaque du boss | 20 px | 0,50 s |

Un défaut classique consiste à secouer trop et trop souvent. L'effet perd alors tout impact et devient fatigant. Réservez-le aux événements marquants.

### Une décroissance qui a du sens

Une décroissance linéaire (`intensite -= vitesse * dt`) donne un tremblement qui s'arrête brutalement. Une décroissance quadratique (`intensite = base * (restant / duree)²`) donne une secousse forte au début et une extinction douce, bien plus satisfaisante.

```dart
/// Générateur pseudo-aléatoire déterministe (LCG).
/// On l'écrit à la main pour que la sortie de cet exemple soit
/// strictement reproductible, sur toutes les plateformes.
class Alea {
  int _etat;
  Alea(this._etat);

  /// Un nombre entre 0 (inclus) et 1 (exclu).
  double suivant() {
    _etat = (_etat * 1664525 + 1013904223) % 2147483648;
    return _etat / 2147483648;
  }

  /// Un nombre entre -1 et 1.
  double signe() => suivant() * 2 - 1;
}

class ScreenShake {
  final Alea _alea = Alea(7);

  double _base = 0;
  double _duree = 0;
  double _restant = 0;

  double dx = 0;
  double dy = 0;

  void declencher(double intensite, double duree) {
    // Un shake plus fort remplace le précédent ; un plus faible ne l'écrase pas.
    if (intensite < _base * (_duree == 0 ? 0 : _restant / _duree)) return;
    _base = intensite;
    _duree = duree;
    _restant = duree;
  }

  void mettreAJour(double dt) {
    if (_restant <= 0) {
      dx = 0;
      dy = 0;
      return;
    }
    _restant -= dt;
    if (_restant < 0) _restant = 0;

    final double facteur = _restant / _duree;
    final double intensite = _base * facteur * facteur; // quadratique
    dx = _alea.signe() * intensite;
    dy = _alea.signe() * intensite;
  }
}

void main() {
  final ScreenShake shake = ScreenShake();
  shake.declencher(14, 0.35);

  const double dt = 1 / 30; // pas grossier pour un affichage court
  for (int i = 1; i <= 12; i++) {
    shake.mettreAJour(dt);
    print('frame ${i.toString().padLeft(2)} : '
        'dx = ${shake.dx.toStringAsFixed(2).padLeft(6)}  '
        'dy = ${shake.dy.toStringAsFixed(2).padLeft(6)}');
  }
}
```

**Résultat :**

```text
frame  1 : dx =  -0.51  dy =   7.49
frame  2 : dx =  -5.05  dy =   6.49
frame  3 : dx =  -5.73  dy =   3.02
frame  4 : dx =   2.38  dy =  -4.23
frame  5 : dx =  -2.78  dy =   3.59
frame  6 : dx =   2.29  dy =  -2.25
frame  7 : dx =   1.26  dy =  -1.26
frame  8 : dx =  -0.46  dy =   0.09
frame  9 : dx =   0.03  dy =   0.19
frame 10 : dx =   0.03  dy =   0.01
frame 11 : dx =   0.00  dy =   0.00
frame 12 : dx =   0.00  dy =   0.00
```

Ce qui compte n'est pas la valeur de chaque nombre, mais la **forme** : de grandes amplitudes au début, une extinction rapide, puis un zéro franc au bout de 0,35 s. En production, remplacez `Alea` par `math.Random()` : la graine fixe ne sert ici qu'à obtenir une sortie identique à celle imprimée.

Côté rendu, l'usage se réduit à trois lignes :

```dart
canvas.save();
canvas.translate(shake.dx, shake.dy);
// ... dessiner tout le MONDE ici ...
canvas.restore();
// ... dessiner l'interface ICI, hors du shake ...
```

> **À retenir.** Le monde est dessiné dans le `save/restore`, l'interface en dehors. Un HUD qui tremble avec l'écran donne mal au cœur et rend le score illisible.

---

## 23.31 — Le knockback

Le *knockback* (recul) est l'impulsion reçue lors d'un coup. C'est l'application directe de tout ce chapitre : une direction normalisée, une force, une affectation de vitesse.

```dart
final Vec2 direction = (cible.position - source.position).normalise;
cible.vitesse = direction * forceRecul;
```

### Trois décisions à prendre

**1. Affecter ou additionner ?** Comme pour le saut (23.19), **affectez**. Un joueur touché trois fois de suite ne doit pas partir trois fois plus loin.

**2. Ajouter une composante verticale ?** Presque toujours. Un recul purement horizontal semble plat. On ajoute une petite impulsion vers le haut, que la gravité récupère ensuite.

```dart
cible.vitesse = Vec2(direction.x * forceRecul, -forceVerticale);
```

**3. Bloquer les commandes pendant le recul ?** Oui, sinon le joueur annule le recul en poussant dans l'autre sens et l'effet disparaît. On introduit un compteur `controleBloqueRestant` de 0,15 à 0,25 s.

```text
  CHRONOLOGIE D'UN COUP REÇU

  t = 0.00   impact
             │ vitesse = direction * 320, y = -220
             │ invincibilite = 1.0 s
             │ controleBloque = 0.20 s
             │ screen shake 8 px
  t = 0.20   le joueur reprend la main
  t = 1.00   fin de l'invincibilité (le clignotement s'arrête)
```

### Le cas du vecteur nul

Si les deux entités sont exactement à la même position, `(cible - source)` vaut `(0, 0)` et `normalise` renvoie le vecteur nul : le recul serait sans effet. C'est rare, mais cela arrive avec des ennemis qui se superposent. Prévoyez une direction par défaut.

```dart
// (Collez la classe Vec2 complète au-dessus.)
class Personnage {
  Vec2 position;
  Vec2 vitesse = Vec2.zero;
  int pointsDeVie = 100;
  double controleBloqueRestant = 0;
  double invincibiliteRestante = 0;

  Personnage(this.position);

  bool get controlable => controleBloqueRestant <= 0;

  void recevoirCoup(Vec2 sourcePosition, int degats) {
    if (invincibiliteRestante > 0) return; // encore invincible : rien

    pointsDeVie -= degats;

    Vec2 direction = (position - sourcePosition).normalise;
    if (direction == Vec2.zero) direction = Vec2.droite; // secours

    vitesse = Vec2(direction.x * 320, -220);
    controleBloqueRestant = 0.20;
    invincibiliteRestante = 1.0;
  }

  void mettreAJour(double dt) {
    if (controleBloqueRestant > 0) controleBloqueRestant -= dt;
    if (invincibiliteRestante > 0) invincibiliteRestante -= dt;

    vitesse += const Vec2(0, 1600) * dt;
    position += vitesse * dt;
    if (position.y > 400) {
      position = position.copieAvec(y: 400);
      vitesse = vitesse.copieAvec(y: 0);
    }
  }
}

void main() {
  final Personnage heros = Personnage(const Vec2(200, 400));
  const Vec2 gobelin = Vec2(160, 400); // à gauche du héros
  const double dt = 1 / 60;

  heros.recevoirCoup(gobelin, 12);
  print('PV = ${heros.pointsDeVie}');

  for (int i = 1; i <= 24; i++) {
    heros.mettreAJour(dt);
    if (i % 8 == 0) {
      print('frame ${i.toString().padLeft(2)} : '
          'x = ${heros.position.x.toStringAsFixed(1).padLeft(6)}  '
          'y = ${heros.position.y.toStringAsFixed(1).padLeft(6)}  '
          'contrôlable = ${heros.controlable}');
    }
  }

  // Un second coup pendant l'invincibilité ne fait rien.
  heros.recevoirCoup(gobelin, 12);
  print('PV après second coup immédiat = ${heros.pointsDeVie}');
}
```

**Résultat :**

```text
PV = 88
frame  8 : x =  242.7  y =  386.7  contrôlable = false
frame 16 : x =  285.3  y =  400.0  contrôlable = true
frame 24 : x =  328.0  y =  400.0  contrôlable = true
PV après second coup immédiat = 88
```

Le héros est projeté vers la droite, à l'opposé du gobelin. Il décolle d'une quinzaine de pixels, retombe au sol vers la frame 16 (soit 0,27 s), et reprend le contrôle un peu avant, à 0,20 s. Le second coup est ignoré grâce à l'invincibilité : les points de vie restent à 88.

> **Remarque de design.** Le recul horizontal n'est jamais freiné dans ce code : le héros glisserait indéfiniment. En pratique, on combine avec le frottement de 23.23, ce que fait le mini-projet de la section suivante.

---

## 23.32 — Mini-projet : un personnage qui court, saute et retombe

Il est temps d'assembler. Ce mini-projet réunit **toutes** les notions du chapitre dans un seul `main.dart` exécutable, sans aucune image et sans aucun package.

### Cahier des charges

Le héros du Donjon de Dart doit :

1. courir à gauche et à droite avec accélération, décélération et demi-tour distincts (23.25) ;
2. subir une gravité et se poser sur le sol (23.17, 23.18) ;
3. sauter avec un saut variable selon la durée d'appui (23.21) ;
4. disposer d'un double saut (23.20) ;
5. bénéficier du coyote time et du jump buffering (23.22) ;
6. subir un frottement au sol et une vitesse de chute plafonnée (23.23, 23.24) ;
7. être repoussé par un gobelin qui patrouille, avec invincibilité temporaire (23.31) ;
8. déclencher un tremblement d'écran à l'impact (23.30) ;
9. être jouable au clavier **et** au tactile ;
10. afficher son état interne pour pouvoir être débogué.

### Architecture

```text
  main.dart
    │
    ├── Vec2                 pur Dart, testable, aucune dépendance Flutter
    │
    ├── Entrees              ce que le joueur demande (3 booléens)
    │
    ├── Heros                position, vitesse, états, mettreAJour(dt)
    ├── Gobelin              patrouille simple, mettreAJour(dt)
    ├── ScreenShake          effet de caméra, mettreAJour(dt)
    │
    ├── EcranDonjon          StatefulWidget + Ticker : la boucle de jeu
    └── PeintreDonjon        CustomPainter : DESSINE, ne décide de rien
```

Cette séparation est celle du chapitre 20 et du chapitre 22 : la logique ignore le `Canvas`, le peintre ignore les règles.

### Le code complet

```dart
import 'dart:math' as math;

import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';
import 'package:flutter/services.dart';

// ===================================================================
// 1. LE VECTEUR
// ===================================================================

class Vec2 {
  final double x;
  final double y;

  const Vec2(this.x, this.y);

  static const Vec2 zero = Vec2(0, 0);
  static const Vec2 droite = Vec2(1, 0);

  Vec2 operator +(Vec2 a) => Vec2(x + a.x, y + a.y);
  Vec2 operator -(Vec2 a) => Vec2(x - a.x, y - a.y);
  Vec2 operator *(double k) => Vec2(x * k, y * k);
  Vec2 operator -() => Vec2(-x, -y);

  @override
  bool operator ==(Object o) => o is Vec2 && o.x == x && o.y == y;

  @override
  int get hashCode => Object.hash(x, y);

  double get longueurCarree => x * x + y * y;
  double get longueur => math.sqrt(longueurCarree);

  Vec2 get normalise {
    final double l = longueur;
    return l == 0 ? Vec2.zero : Vec2(x / l, y / l);
  }

  Vec2 limite(double max) =>
      longueurCarree > max * max ? normalise * max : this;

  Vec2 copieAvec({double? x, double? y}) => Vec2(x ?? this.x, y ?? this.y);

  Vec2 lerp(Vec2 a, double t) => Vec2(x + (a.x - x) * t, y + (a.y - y) * t);

  @override
  String toString() =>
      '(${x.toStringAsFixed(0)}, ${y.toStringAsFixed(0)})';
}

// ===================================================================
// 2. LES ENTRÉES DU JOUEUR
// ===================================================================

/// Ce que le joueur demande. Aucune logique de jeu ici.
class Entrees {
  bool gauche = false;
  bool droite = false;
  bool sautMaintenu = false;

  /// Passe à true une seule frame, au moment de l'appui.
  bool sautPresse = false;

  /// -1 vers la gauche, 0 immobile, +1 vers la droite.
  double get axeX => (droite ? 1 : 0) - (gauche ? 1 : 0);

  void finDeFrame() => sautPresse = false;
}

// ===================================================================
// 3. LE HÉROS
// ===================================================================

class Heros {
  Vec2 position;
  Vec2 vitesse = Vec2.zero;

  // --- réglages de course (23.25) ---
  static const double vitesseMax = 230;
  static const double acceleration = 1500;
  static const double deceleration = 1900;
  static const double demiTour = 2800;
  static const double frottementAir = 700;

  // --- réglages de saut (23.17 à 23.22) ---
  static const double gravite = 2000;
  static const double graviteChute = 2800; // plus forte en descente
  static const double forceSaut = 700;
  static const double coupureSaut = 0.35;
  static const double vitesseChuteMax = 1100;
  static const int sautsMax = 2;
  static const double coyoteDuree = 0.10;
  static const double bufferDuree = 0.12;

  // --- géométrie ---
  static const double largeur = 26;
  static const double hauteur = 42;
  double get demiL => largeur / 2;
  double get demiH => hauteur / 2;

  // --- états ---
  bool auSol = false;
  int sautsRestants = sautsMax;
  double coyoteRestant = 0;
  double bufferRestant = 0;
  double controleBloqueRestant = 0;
  double invincibiliteRestante = 0;
  int pointsDeVie = 5;
  int regard = 1; // 1 = droite, -1 = gauche

  Heros(this.position);

  bool get controlable => controleBloqueRestant <= 0;
  bool get invincible => invincibiliteRestante > 0;

  void mettreAJour(double dt, Entrees e, double solY, double largeurMonde) {
    _chronometres(dt, e);
    _horizontal(dt, e);
    _vertical(dt, e);
    _integrer(dt);
    _bornes(solY, largeurMonde);
  }

  // -------------------------------------------------- chronomètres
  void _chronometres(double dt, Entrees e) {
    if (controleBloqueRestant > 0) controleBloqueRestant -= dt;
    if (invincibiliteRestante > 0) invincibiliteRestante -= dt;

    if (auSol) {
      coyoteRestant = coyoteDuree;
    } else if (coyoteRestant > 0) {
      coyoteRestant -= dt;
    }

    if (e.sautPresse) bufferRestant = bufferDuree;
    if (bufferRestant > 0) bufferRestant -= dt;
  }

  // -------------------------------------------------- course
  void _horizontal(double dt, Entrees e) {
    final double axe = controlable ? e.axeX : 0;

    if (axe != 0) regard = axe > 0 ? 1 : -1;

    final double cible = axe * vitesseMax;

    double taux;
    if (axe == 0) {
      taux = auSol ? deceleration : frottementAir;
    } else if (vitesse.x != 0 && axe.sign != vitesse.x.sign) {
      taux = demiTour;
    } else {
      taux = acceleration;
    }

    final double ecart = cible - vitesse.x;
    final double pas = taux * dt;
    final double vx =
        ecart.abs() <= pas ? cible : vitesse.x + (ecart > 0 ? pas : -pas);

    vitesse = vitesse.copieAvec(x: vx);
  }

  // -------------------------------------------------- saut et gravité
  void _vertical(double dt, Entrees e) {
    // 1. Déclenchement du saut : buffer + (coyote OU saut restant).
    final bool peutSauterDuSol = coyoteRestant > 0;
    final bool peutDoubleSauter = !auSol && sautsRestants > 0;

    if (bufferRestant > 0 && (peutSauterDuSol || peutDoubleSauter)) {
      vitesse = vitesse.copieAvec(y: -forceSaut);
      auSol = false;
      bufferRestant = 0;
      if (peutSauterDuSol) {
        coyoteRestant = 0;
        sautsRestants = sautsMax - 1; // le premier saut est consommé
      } else {
        sautsRestants -= 1;
      }
    }

    // 2. Saut variable : on écrête si le joueur relâche en montée.
    if (!e.sautMaintenu && vitesse.y < 0) {
      vitesse = vitesse.copieAvec(y: vitesse.y * coupureSaut);
    }

    // 3. Gravité asymétrique : la chute est plus lourde que la montée.
    final double g = vitesse.y < 0 ? gravite : graviteChute;
    vitesse = vitesse.copieAvec(y: vitesse.y + g * dt);

    // 4. Vitesse terminale.
    if (vitesse.y > vitesseChuteMax) {
      vitesse = vitesse.copieAvec(y: vitesseChuteMax);
    }
  }

  // -------------------------------------------------- intégration
  void _integrer(double dt) {
    position += vitesse * dt; // la vitesse a DÉJÀ été mise à jour
  }

  // -------------------------------------------------- sol et murs
  void _bornes(double solY, double largeurMonde) {
    if (position.y + demiH >= solY) {
      position = position.copieAvec(y: solY - demiH);
      vitesse = vitesse.copieAvec(y: 0);
      if (!auSol) sautsRestants = sautsMax; // rechargement à l'atterrissage
      auSol = true;
    } else {
      auSol = false;
    }

    if (position.x - demiL < 0) {
      position = position.copieAvec(x: demiL);
      vitesse = vitesse.copieAvec(x: 0);
    }
    if (position.x + demiL > largeurMonde) {
      position = position.copieAvec(x: largeurMonde - demiL);
      vitesse = vitesse.copieAvec(x: 0);
    }
  }

  // -------------------------------------------------- dégâts
  /// Renvoie true si le coup a réellement été encaissé.
  bool recevoirCoup(Vec2 source) {
    if (invincible) return false;

    pointsDeVie -= 1;

    Vec2 direction = (position - source).normalise;
    if (direction == Vec2.zero) direction = Vec2.droite;

    vitesse = Vec2(direction.x.sign * 330, -260);
    controleBloqueRestant = 0.22;
    invincibiliteRestante = 1.1;
    return true;
  }

  Rect get boite => Rect.fromCenter(
        center: Offset(position.x, position.y),
        width: largeur,
        height: hauteur,
      );
}

// ===================================================================
// 4. LE GOBELIN
// ===================================================================

class Gobelin {
  Vec2 position;
  double vitesseX;
  final double xMin;
  final double xMax;

  static const double largeur = 24;
  static const double hauteur = 28;

  Gobelin(this.position, this.vitesseX, this.xMin, this.xMax);

  void mettreAJour(double dt) {
    position += Vec2(vitesseX, 0) * dt;
    if (position.x < xMin) {
      position = position.copieAvec(x: xMin);
      vitesseX = -vitesseX;
    } else if (position.x > xMax) {
      position = position.copieAvec(x: xMax);
      vitesseX = -vitesseX;
    }
  }

  Rect get boite => Rect.fromCenter(
        center: Offset(position.x, position.y),
        width: largeur,
        height: hauteur,
      );
}

// ===================================================================
// 5. LE TREMBLEMENT D'ÉCRAN
// ===================================================================

class ScreenShake {
  final math.Random _alea = math.Random();
  double _base = 0;
  double _duree = 0;
  double _restant = 0;

  Vec2 decalage = Vec2.zero;

  void declencher(double intensite, double duree) {
    _base = intensite;
    _duree = duree;
    _restant = duree;
  }

  void mettreAJour(double dt) {
    if (_restant <= 0) {
      decalage = Vec2.zero;
      return;
    }
    _restant = math.max(0, _restant - dt);
    final double f = _restant / _duree;
    final double i = _base * f * f;
    decalage = Vec2(
      (_alea.nextDouble() * 2 - 1) * i,
      (_alea.nextDouble() * 2 - 1) * i,
    );
  }
}

// ===================================================================
// 6. L'APPLICATION
// ===================================================================

void main() => runApp(const DonjonApp());

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) => const MaterialApp(
        debugShowCheckedModeBanner: false,
        home: EcranDonjon(),
      );
}

class EcranDonjon extends StatefulWidget {
  const EcranDonjon({super.key});

  @override
  State<EcranDonjon> createState() => _EcranDonjonState();
}

class _EcranDonjonState extends State<EcranDonjon>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  final FocusNode _focus = FocusNode();
  Duration _precedent = Duration.zero;

  final Entrees _entrees = Entrees();
  final ScreenShake _shake = ScreenShake();

  late Heros _heros;
  late List<Gobelin> _gobelins;

  double _solY = 420;
  double _largeurMonde = 360;
  double _fps = 0;
  bool _pret = false;

  @override
  void initState() {
    super.initState();
    _heros = Heros(const Vec2(80, 300));
    _gobelins = <Gobelin>[];
    _ticker = createTicker(_frame)..start();
  }

  void _preparerMonde(Size taille) {
    if (_pret) return;
    _pret = true;
    _solY = taille.height - 90;
    _largeurMonde = taille.width;
    _heros = Heros(Vec2(80, _solY - Heros.hauteur / 2));
    _gobelins = <Gobelin>[
      Gobelin(Vec2(taille.width * 0.55, _solY - Gobelin.hauteur / 2), 70,
          taille.width * 0.35, taille.width * 0.85),
      Gobelin(Vec2(taille.width * 0.75, _solY - Gobelin.hauteur / 2), -110,
          taille.width * 0.45, taille.width * 0.95),
    ];
  }

  void _frame(Duration maintenant) {
    final double dtBrut =
        (maintenant - _precedent).inMicroseconds / 1000000.0;
    _precedent = maintenant;
    if (dtBrut <= 0) return;

    // Garde du chapitre 20 : un dt aberrant casserait la physique.
    final double dt = dtBrut > 0.05 ? 0.05 : dtBrut;

    setState(() {
      _fps = _fps * 0.9 + (1 / dtBrut) * 0.1;

      _heros.mettreAJour(dt, _entrees, _solY, _largeurMonde);
      for (final Gobelin g in _gobelins) {
        g.mettreAJour(dt);
      }
      _shake.mettreAJour(dt);

      for (final Gobelin g in _gobelins) {
        if (_heros.boite.overlaps(g.boite)) {
          if (_heros.recevoirCoup(g.position)) {
            _shake.declencher(9, 0.25);
          }
        }
      }

      _entrees.finDeFrame();
    });
  }

  // ---------------------------------------------------------- clavier
  void _touche(KeyEvent evenement) {
    final LogicalKeyboardKey k = evenement.logicalKey;
    final bool bas = evenement is KeyDownEvent;
    final bool haut = evenement is KeyUpEvent;

    if (k == LogicalKeyboardKey.arrowLeft || k == LogicalKeyboardKey.keyQ) {
      if (bas) _entrees.gauche = true;
      if (haut) _entrees.gauche = false;
    } else if (k == LogicalKeyboardKey.arrowRight ||
        k == LogicalKeyboardKey.keyD) {
      if (bas) _entrees.droite = true;
      if (haut) _entrees.droite = false;
    } else if (k == LogicalKeyboardKey.space ||
        k == LogicalKeyboardKey.arrowUp) {
      if (bas && !_entrees.sautMaintenu) _entrees.sautPresse = true;
      if (bas) _entrees.sautMaintenu = true;
      if (haut) _entrees.sautMaintenu = false;
    }
  }

  // ---------------------------------------------------------- tactile
  Widget _bouton(String etiquette, VoidCallback debut, VoidCallback fin) {
    return Listener(
      onPointerDown: (_) => debut(),
      onPointerUp: (_) => fin(),
      onPointerCancel: (_) => fin(),
      child: Container(
        width: 76,
        height: 62,
        alignment: Alignment.center,
        decoration: BoxDecoration(
          color: const Color(0x33FFFFFF),
          borderRadius: BorderRadius.circular(10),
        ),
        child: Text(
          etiquette,
          style: const TextStyle(
            color: Color(0xFFE8E4DC),
            fontSize: 20,
            fontFamily: 'monospace',
          ),
        ),
      ),
    );
  }

  @override
  void dispose() {
    _ticker.dispose();
    _focus.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF15121F),
      body: KeyboardListener(
        focusNode: _focus,
        autofocus: true,
        onKeyEvent: _touche,
        child: LayoutBuilder(
          builder: (BuildContext context, BoxConstraints c) {
            _preparerMonde(Size(c.maxWidth, c.maxHeight));
            return Stack(
              children: <Widget>[
                CustomPaint(
                  painter: PeintreDonjon(
                    heros: _heros,
                    gobelins: _gobelins,
                    solY: _solY,
                    shake: _shake.decalage,
                    fps: _fps,
                  ),
                  child: const SizedBox.expand(),
                ),
                Positioned(
                  left: 16,
                  bottom: 16,
                  child: Row(
                    children: <Widget>[
                      _bouton('<', () => _entrees.gauche = true,
                          () => _entrees.gauche = false),
                      const SizedBox(width: 12),
                      _bouton('>', () => _entrees.droite = true,
                          () => _entrees.droite = false),
                    ],
                  ),
                ),
                Positioned(
                  right: 16,
                  bottom: 16,
                  child: _bouton(
                    '^',
                    () {
                      if (!_entrees.sautMaintenu) _entrees.sautPresse = true;
                      _entrees.sautMaintenu = true;
                    },
                    () => _entrees.sautMaintenu = false,
                  ),
                ),
              ],
            );
          },
        ),
      ),
    );
  }
}

// ===================================================================
// 7. LE RENDU
// ===================================================================

class PeintreDonjon extends CustomPainter {
  final Heros heros;
  final List<Gobelin> gobelins;
  final double solY;
  final Vec2 shake;
  final double fps;

  PeintreDonjon({
    required this.heros,
    required this.gobelins,
    required this.solY,
    required this.shake,
    required this.fps,
  });

  @override
  void paint(Canvas canvas, Size size) {
    // --- LE MONDE : soumis au tremblement ---
    canvas.save();
    canvas.translate(shake.x, shake.y);

    // Le sol.
    canvas.drawRect(
      Rect.fromLTWH(-20, solY, size.width + 40, size.height - solY + 20),
      Paint()..color = const Color(0xFF332E45),
    );
    // Les dalles.
    final Paint joint = Paint()
      ..color = const Color(0xFF262238)
      ..strokeWidth = 2;
    for (double x = -20; x < size.width + 40; x += 44) {
      canvas.drawLine(Offset(x, solY), Offset(x, size.height), joint);
    }

    // Les gobelins.
    final Paint pGobelin = Paint()..color = const Color(0xFF7FBF5A);
    for (final Gobelin g in gobelins) {
      canvas.drawRRect(
        RRect.fromRectAndRadius(g.boite, const Radius.circular(5)),
        pGobelin,
      );
      // Deux yeux, orientés selon la marche.
      final double s = g.vitesseX > 0 ? 1 : -1;
      final Paint oeil = Paint()..color = const Color(0xFF15121F);
      canvas.drawCircle(
          Offset(g.position.x + 4 * s, g.position.y - 5), 2.4, oeil);
      canvas.drawCircle(
          Offset(g.position.x + 9 * s, g.position.y - 5), 2.4, oeil);
    }

    // Le héros : il clignote pendant l'invincibilité.
    final bool cache = heros.invincible &&
        (heros.invincibiliteRestante * 14).floor() % 2 == 0;
    if (!cache) {
      final Paint pHeros = Paint()
        ..color = heros.auSol
            ? const Color(0xFF6FB3E0)
            : const Color(0xFF9AD86F);
      canvas.drawRRect(
        RRect.fromRectAndRadius(heros.boite, const Radius.circular(6)),
        pHeros,
      );
      // Le regard.
      canvas.drawCircle(
        Offset(heros.position.x + 6.0 * heros.regard, heros.position.y - 8),
        3,
        Paint()..color = const Color(0xFF15121F),
      );
    }

    canvas.restore();

    // --- L'INTERFACE : jamais soumise au tremblement ---
    _texte(canvas, 'DONJON DE DART — chapitre 23', const Offset(16, 14));
    _texte(canvas, 'fps      ${fps.toStringAsFixed(0)}',
        const Offset(16, 36));
    _texte(canvas, 'vitesse  ${heros.vitesse}', const Offset(16, 52));
    _texte(canvas, 'auSol    ${heros.auSol}', const Offset(16, 68));
    _texte(canvas, 'sauts    ${heros.sautsRestants}', const Offset(16, 84));
    _texte(canvas, 'coyote   ${heros.coyoteRestant.toStringAsFixed(2)}',
        const Offset(16, 100));
    _texte(canvas, 'buffer   ${heros.bufferRestant.toStringAsFixed(2)}',
        const Offset(16, 116));

    // Les points de vie.
    for (int i = 0; i < 5; i++) {
      canvas.drawRect(
        Rect.fromLTWH(size.width - 26.0 - i * 22, 16, 16, 16),
        Paint()
          ..color = i < heros.pointsDeVie
              ? const Color(0xFFD9615A)
              : const Color(0xFF3A3350),
      );
    }
  }

  void _texte(Canvas canvas, String s, Offset o) {
    final TextPainter tp = TextPainter(
      text: TextSpan(
        text: s,
        style: const TextStyle(
          color: Color(0xFFE8E4DC),
          fontSize: 12,
          fontFamily: 'monospace',
        ),
      ),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, o);
  }

  @override
  bool shouldRepaint(PeintreDonjon old) => true;
}
```

**Résultat :**

```text
DONJON DE DART — chapitre 23
fps      60
vitesse  (230, 0)
auSol    true
sauts    2
coyote   0.10
buffer   0.00

Une salle de donjon : un sol de dalles violettes occupe le bas de l'écran,
cinq carrés rouges en haut à droite figurent les points de vie.
Un rectangle bleu arrondi — le héros — court de gauche à droite avec une
légère montée en vitesse, et s'arrête franchement quand on relâche.
Une pression brève sur le bouton « ^ » le fait bondir d'environ 60 pixels,
une pression maintenue d'environ 120. En l'air il devient vert, et un
second appui déclenche le double saut : le compteur « sauts » tombe à 0.
Deux gobelins verts patrouillent sur le sol, les yeux tournés vers leur
direction de marche.
Au contact d'un gobelin, l'écran tremble brièvement, un cœur disparaît,
le héros est projeté en arrière et vers le haut, clignote pendant une
seconde, et ne répond plus aux commandes pendant deux dixièmes de seconde.
Les touches flèches (ou Q et D) et la barre d'espace font la même chose
que les boutons tactiles.
```

### Les points à retenir de ce code

**L'ordre dans `mettreAJour`.** Chronomètres, puis horizontal, puis vertical, puis intégration, puis bornes. Chaque étape prépare la suivante, et l'intégration est **la dernière** avant la correction des bornes. Si vous déplacez `_integrer` plus haut, le saut prendra une frame de retard.

**La gravité asymétrique.** `gravite` en montée (2000), `graviteChute` en descente (2800). Ce détail, invisible sur le papier, est ce qui distingue un saut de plateforme agréable d'un saut « lunaire ». Le personnage monte avec élan et retombe avec autorité.

**Le double saut et le coyote cohabitent.** La condition de 23.20 et celle de 23.22 sont réunies dans un seul `if`. Notez `sautsRestants = sautsMax - 1` quand le saut vient du sol : sans cela, un saut depuis le sol laisserait deux sauts en l'air, soit un triple saut.

**Le shake enveloppe le monde, pas l'interface.** Le `save`/`restore` isole strictement les deux. Le compteur de FPS et les cœurs ne bougent pas d'un pixel.

**Le `dt` est plafonné.** `dtBrut > 0.05 ? 0.05 : dtBrut` reprend la garde du chapitre 20. Sans elle, un ralentissement du système ferait traverser le sol au héros en une seule frame.

**Aucune image, aucun package.** Tout est dessiné avec des `Rect`, des `RRect` et des `Circle`. Vous pourrez remplacer chaque rectangle par un sprite du chapitre 22 sans toucher une seule ligne de physique.

---

## 23.33 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `position += vitesse;` sans `dt` | on raisonne en pixels par frame au lieu de pixels par seconde | `position += vitesse * dt;` — la vitesse est toujours en px/s (chapitre 20) |
| La diagonale est 41 % plus rapide | on additionne les contributions des touches sans normaliser | construire `Vec2(dx, dy)`, appeler `.normalise`, puis multiplier par la vitesse (23.8) |
| Le personnage rebondit de plus en plus haut | intégration d'Euler explicite : `position` mise à jour avant `vitesse` | inverser : `vitesse += acceleration * dt;` **puis** `position += vitesse * dt;` (23.15) |
| La gravité ne fait rien le premier saut | la gravité est appliquée après avoir déplacé la position | appliquer la gravité à la vitesse en tout début de `mettreAJour` |
| Le héros s'envole à l'infini | pas de `clamp` sur la vitesse de chute ni sur la vitesse horizontale | `vitesse.limite(vMax)` et un plafond de vitesse terminale (23.24) |
| Erreur de compilation `num can't be assigned to double` | `clamp` est déclaré sur `num`, il renvoie `num` | ajouter `.toDouble()` : `v.clamp(-200, 200).toDouble()` |
| Sauts infinis en plein vol | la condition `auSol` (ou le compteur de sauts) est absente | tester `sautsRestants > 0` et décrémenter à chaque saut (23.20) |
| Le double saut devient un triple saut | `sautsRestants` rechargé à chaque frame passée au sol | recharger uniquement à la **transition** d'atterrissage : `if (!auSol) sautsRestants = sautsMax;` |
| Le héros saute vers le bas | on oublie que l'axe `y` du `Canvas` descend | la force de saut est **négative** : `vitesse.y = -forceSaut` (23.19) |
| Le héros s'enfonce dans le sol frame après frame | on teste `position.y == solY` au lieu de `>=`, ou on ne replace pas la position | `if (position.y + demiH >= solY) { position.y = solY - demiH; vitesse.y = 0; }` (23.18) |
| Le héros vibre sur le sol | on replace la position mais on n'annule pas `vitesse.y` | toujours faire les trois gestes : replacer, annuler, noter l'état |
| Le personnage disparaît sans message d'erreur | division par zéro dans `normalise` : `NaN` qui se propage | garde `if (longueur == 0) return Vec2.zero;` (23.7) |
| Le frottement fait reculer le personnage | on soustrait plus de vitesse qu'il n'en reste | `if (vitesse.longueur <= perte) vitesse = Vec2.zero;` avant de soustraire (23.23) |
| L'amortissement dépend de la machine | `vitesse *= 0.9` par frame | `vitesse *= math.pow(facteurParSeconde, dt).toDouble()` (23.23) |
| La caméra rattrape plus vite en 120 FPS | `lerp` avec un `t` constant par frame | `t = 1 - math.exp(-lambda * dt)` (23.29) |
| Le gobelin vibre sur sa cible | il dépasse la cible chaque frame et se retourne | ne jamais avancer de plus que la distance restante (23.12) |
| Deux entités partagent la même vitesse par accident | vecteur mutable partagé entre deux objets | rendre `Vec2` immuable avec des champs `final` (23.3) |
| `Set<Vec2>` contient des doublons | `==` redéfini sans `hashCode` | toujours redéfinir les deux ensemble (23.5, chapitre 10) |
| Le HUD tremble avec l'écran et devient illisible | le `screen shake` englobe aussi l'interface | dessiner le monde entre `canvas.save()` et `canvas.restore()`, le HUD après (23.30) |
| Le joueur annule son propre recul | les commandes ne sont pas bloquées pendant le knockback | un compteur `controleBloqueRestant` de 0,15 à 0,25 s (23.31) |
| Un coup inflige trois fois les dégâts | pas d'invincibilité temporaire après un impact | `invincibiliteRestante` d'environ 1 s, testée avant d'appliquer les dégâts |
| Le personnage traverse le sol après un ralentissement | `dt` énorme après un pic de lag | plafonner `dt` à 0,05 s (chapitre 20), et voir le *tunneling* au chapitre 24 |

---

## 23.34 — Résumé du chapitre

| Formule | Signification |
| --- | --- |
| `position += vitesse * dt` | la vitesse déplace la position (px/s × s = px) |
| `vitesse += acceleration * dt` | l'accélération modifie la vitesse (px/s² × s = px/s) |
| vitesse **d'abord**, position **ensuite** | intégration d'Euler semi-implicite : stable, ne diverge pas |
| `a + b = (a.x + b.x, a.y + b.y)` | addition : deux déplacements mis bout à bout |
| `b - a` | le vecteur qui va de `a` vers `b` (arrivée moins départ) |
| `v * k` | allonge ou raccourcit sans changer la direction |
| `longueur = racine(x² + y²)` | la taille du vecteur, en pixels (Pythagore) |
| `longueurCarree = x² + y²` | idem sans racine carrée : pour comparer, c'est suffisant et rapide |
| `normalise = v / longueur` | ramène à une longueur de 1 : une direction pure |
| `direction * vitesseMax` | la formule universelle du déplacement dirigé |
| `a · b = a.x·b.x + a.y·b.y` | produit scalaire : positif si aligné, nul si perpendiculaire, négatif si opposé |
| `distance(a, b) = longueur(b - a)` | distance entre deux points |
| `atan2(y, x)` | angle du vecteur, entre −pi et +pi ; le `y` en premier |
| `Vec2(cos(a), sin(a)) * r` | opération inverse : d'un angle vers un vecteur |
| `gravite = Vec2(0, g)` | une accélération constante vers le bas ; `y` positif descend |
| `v0 = racine(2 × g × hauteur)` | vitesse de saut pour atteindre une hauteur donnée |
| `vitesse.y = -forceSaut` | le saut est une impulsion : on **affecte**, on n'additionne pas |
| `if (y + demiH >= solY)` | test du sol : replacer, annuler `vitesse.y`, noter `auSol` |
| `sautsRestants > 0` | double saut : un compteur, rechargé à l'atterrissage seulement |
| `vitesse.y *= coupureSaut` au relâchement | saut variable : on écrête la montée, jamais la chute |
| `coyoteRestant`, `bufferRestant` | tolérances de saut, 0,10 s environ, invisibles quand elles marchent |
| `vitesse -= vitesse.normalise * frottement * dt` | frottement linéaire : arrêt franc, avec garde contre l'inversion |
| `vitesse *= pow(facteur, dt)` | amortissement indépendant du framerate |
| `vitesse.limite(vMax)` | plafonne la **longueur** : toutes les directions égales |
| `v.clamp(a, b).toDouble()` | plafonne une composante ; `clamp` renvoie un `num` |
| `vitesse.y = -vitesse.y * restitution` | rebond ; la hauteur suivante vaut `restitution²` de la précédente |
| `angle += vitesseAngulaire * dt` puis `position = centre + Vec2.depuisAngle(angle) * rayon` | mouvement circulaire : on anime l'angle, la position se recalcule |
| `lerp(a, b, t) = a + (b - a) * t` | interpolation linéaire ; `t` entre 0 et 1 |
| `t = 1 - exp(-lambda * dt)` | suivi doux correct, indépendant du framerate |
| `canvas.save(); translate(shake); ...; restore();` | tremblement d'écran : le monde tremble, pas le HUD |
| `vitesse = (cible - source).normalise * force` | knockback ; bloquer les commandes et ajouter une invincibilité |

---

## 23.35 — Exercices

Les exercices sont progressifs. Les cinq premiers se font en Dart console (DartPad convient), les cinq suivants demandent un projet Flutter. Sauf indication contraire, réutilisez la classe `Vec2` complète de la section 23.13.

### Exercice 1 — Compléter `Vec2` (facile)

Ajoutez à la classe `Vec2` :

- une méthode `Vec2 perpendiculaire` qui renvoie le vecteur tourné de 90 degrés dans le sens des `y` croissants — autrement dit `(x, y)` devient `(-y, x)` ;
- une méthode `bool estPresqueEgal(Vec2 autre, [double tolerance = 0.001])` qui compare deux vecteurs à une tolérance près.

Vérifiez avec un `main` que `Vec2(1, 0).perpendiculaire` vaut `(0, 1)`, et que `Vec2(0.1 + 0.2, 0).estPresqueEgal(Vec2(0.3, 0))` renvoie `true` alors que l'égalité stricte renvoie `false`.

### Exercice 2 — Le radar du donjon (facile)

Écrivez une fonction qui, à partir de la position du héros et d'une liste de trois gobelins, affiche pour chacun :

- sa distance en pixels, arrondie à l'unité ;
- son angle en degrés, arrondi à l'unité ;
- s'il est à portée d'épée (80 px) ou non.

Utilisez `distanceCarreeVers` pour le test de portée et `distanceVers` pour l'affichage.

### Exercice 3 — Le déplacement à huit directions (facile)

Écrivez une fonction `Vec2 vitesseDepuis(bool g, bool d, bool h, bool b, double vMax)` qui renvoie une vitesse correcte pour les huit directions plus l'immobilité.

Affichez un tableau des neuf combinaisons possibles avec la longueur du vecteur obtenu, et vérifiez que toutes les longueurs non nulles valent `vMax`.

### Exercice 4 — La trajectoire de saut (moyen)

Simulez en console un saut avec `gravite = 1600`, `forceSaut = 620`, `dt = 1/60`, un sol à `y = 400` et un héros qui part de `y = 400`.

Affichez, tous les 5 pas :

- le temps écoulé ;
- la hauteur au-dessus du sol ;
- la vitesse verticale.

Puis affichez la hauteur maximale atteinte, l'instant du sommet et la durée totale du saut. Comparez la hauteur mesurée à la valeur théorique `forceSaut² / (2 × gravite)`.

### Exercice 5 — Le comparateur d'intégrateurs (moyen)

Écrivez un programme qui simule un projectile lancé à 45 degrés avec une vitesse de 400 px/s et une gravité de 1000 px/s², avec trois intégrateurs :

- Euler explicite (position avant vitesse) ;
- Euler semi-implicite (vitesse avant position) ;
- la solution exacte `y = v0y × t + 0,5 × g × t²`.

Affichez la portée horizontale obtenue avec chacun pour `dt = 1/30` puis `dt = 1/240`, et concluez sur l'influence du pas de temps.

### Exercice 6 — La balle rebondissante (moyen)

Écrivez un `main.dart` complet où une balle tombe, rebondit sur les quatre bords de l'écran avec un coefficient de restitution de 0,8, et subit un amortissement horizontal indépendant du framerate.

Ajoutez un compteur de rebonds affiché à l'écran et une remise à zéro au toucher.

### Exercice 7 — Le familier qui suit (moyen)

Écrivez un `main.dart` où un petit carré (le familier) suit le doigt ou la souris avec un suivi doux indépendant du framerate.

Ajoutez trois modes sélectionnables par un bouton : `lambda = 3`, `lambda = 8`, `lambda = 20`. Affichez le `lambda` courant et la distance restante.

### Exercice 8 — La tourelle du donjon (moyen)

Écrivez un `main.dart` où une tourelle placée au centre de l'écran vise en permanence la position du doigt.

- Dessinez le canon avec `canvas.rotate` et l'angle obtenu par `atan2`.
- Tirez un projectile toutes les 0,4 s dans la direction visée, à 320 px/s.
- Supprimez les projectiles sortis de l'écran.
- Affichez le nombre de projectiles vivants.

### Exercice 9 — Le réglage de saut par intention (difficile)

Écrivez un `main.dart` où l'on ne règle pas la gravité et la force de saut, mais **la hauteur voulue** et **la durée de montée voulue**, avec deux curseurs `Slider`.

Le programme doit calculer lui-même :

```text
  gravite    = 2 × hauteur / (duree × duree)
  forceSaut  = 2 × hauteur / duree
```

Vérifiez à l'écran que la hauteur atteinte correspond bien à la hauteur demandée, en traçant une ligne repère.

### Exercice 10 — Le héros complet du Donjon de Dart (difficile)

Reprenez le mini-projet de 23.32 et ajoutez trois fonctionnalités :

1. **Une plateforme mobile** qui va et vient horizontalement, sur laquelle le héros peut se poser et qui l'entraîne avec elle ;
2. **Un mur glissant** : quand le héros est en l'air et touche le bord gauche ou droit de l'écran en poussant contre lui, sa vitesse de chute est plafonnée à 120 px/s ;
3. **Le saut mural** : dans cet état, un appui sur saut le propulse à l'opposé du mur avec une composante horizontale de 280 px/s et verticale de 620 px/s, en bloquant les commandes 0,15 s.

---

## 23.36 — Corrections des exercices

### Correction 1

```dart
import 'dart:math' as math;

class Vec2 {
  final double x;
  final double y;

  const Vec2(this.x, this.y);

  static const Vec2 zero = Vec2(0, 0);

  Vec2 operator +(Vec2 a) => Vec2(x + a.x, y + a.y);
  Vec2 operator -(Vec2 a) => Vec2(x - a.x, y - a.y);
  Vec2 operator *(double k) => Vec2(x * k, y * k);

  @override
  bool operator ==(Object o) => o is Vec2 && o.x == x && o.y == y;

  @override
  int get hashCode => Object.hash(x, y);

  double get longueurCarree => x * x + y * y;
  double get longueur => math.sqrt(longueurCarree);

  Vec2 get normalise {
    final double l = longueur;
    return l == 0 ? Vec2.zero : Vec2(x / l, y / l);
  }

  /// Rotation de 90 degrés dans le sens des y croissants.
  Vec2 get perpendiculaire => Vec2(-y, x);

  /// Comparaison tolérante, indispensable avec des nombres flottants.
  bool estPresqueEgal(Vec2 autre, [double tolerance = 0.001]) {
    return (autre - this).longueurCarree <= tolerance * tolerance;
  }

  @override
  String toString() =>
      '(${x.toStringAsFixed(4)}, ${y.toStringAsFixed(4)})';
}

void main() {
  print('--- perpendiculaire ---');
  const Vec2 droite = Vec2(1, 0);
  print('droite            : $droite');
  print('perpendiculaire   : ${droite.perpendiculaire}');
  print('deux fois         : ${droite.perpendiculaire.perpendiculaire}');
  print('quatre fois       : '
      '${droite.perpendiculaire.perpendiculaire.perpendiculaire.perpendiculaire}');

  print('--- longueur conservée ---');
  const Vec2 v = Vec2(3, 4);
  print('v.longueur              : ${v.longueur}');
  print('v.perpendiculaire.long. : ${v.perpendiculaire.longueur}');
  print('produit scalaire        : '
      '${v.x * v.perpendiculaire.x + v.y * v.perpendiculaire.y}');

  print('--- comparaison tolérante ---');
  final Vec2 calcule = Vec2(0.1 + 0.2, 0);
  const Vec2 attendu = Vec2(0.3, 0);
  print('calculé          : ${calcule.x}');
  print('égalité stricte  : ${calcule == attendu}');
  print('estPresqueEgal   : ${calcule.estPresqueEgal(attendu)}');
}
```

**Résultat :**

```text
--- perpendiculaire ---
droite            : (1.0000, 0.0000)
perpendiculaire   : (-0.0000, 1.0000)
deux fois         : (-1.0000, -0.0000)
quatre fois       : (1.0000, 0.0000)
--- longueur conservée ---
v.longueur              : 5.0
v.perpendiculaire.long. : 5.0
produit scalaire        : 0.0
--- comparaison tolérante ---
calculé          : 0.30000000000000004
égalité stricte  : false
estPresqueEgal   : true
```

**Explication :** trois enseignements dans cette correction.

**La formule de la perpendiculaire.** `(x, y)` devient `(-y, x)`. Vous pouvez la retrouver sans l'apprendre : appliquez-la à `(1, 0)`, vous devez obtenir `(0, 1)`. Si vous obtenez `(0, -1)`, vous avez la rotation dans l'autre sens, qui s'écrit `(y, -x)`. Les deux sont utiles ; il faut seulement savoir laquelle on utilise.

**Le `-0.0000` affiché.** En virgule flottante IEEE 754, `-0.0` existe et est distinct de `0.0` en représentation binaire, tout en étant égal à `0.0` pour l'opérateur `==`. Ce n'est pas un bug de votre code. Cela vient du signe de `y` propagé par `-y`.

**Deux propriétés vérifiées.** La perpendiculaire conserve la longueur (5 dans les deux cas) et son produit scalaire avec le vecteur d'origine vaut 0 — ce qui est, par définition (23.9), la signature de la perpendicularité. Ces deux tests sont exactement ce que vous écririez dans un `test()` du chapitre 16.

**Le cœur de l'exercice.** `0.1 + 0.2` ne vaut pas `0.3` en virgule flottante, comme le chapitre 02 l'avait montré. C'est pourquoi une comparaison stricte de vecteurs issus de calculs est presque toujours une erreur. Notez que `estPresqueEgal` compare des **carrés** : on évite ainsi une racine carrée, exactement comme en 23.6.

---

### Correction 2

```dart
import 'dart:math' as math;

class Vec2 {
  final double x;
  final double y;
  const Vec2(this.x, this.y);

  Vec2 operator -(Vec2 a) => Vec2(x - a.x, y - a.y);

  double get longueurCarree => x * x + y * y;
  double get longueur => math.sqrt(longueurCarree);
  double get angle => math.atan2(y, x);
  double get angleDegres => angle * 180 / math.pi;

  double distanceVers(Vec2 a) => (a - this).longueur;
  double distanceCarreeVers(Vec2 a) => (a - this).longueurCarree;

  @override
  String toString() =>
      '(${x.toStringAsFixed(0)}, ${y.toStringAsFixed(0)})';
}

class Gobelin {
  final String nom;
  final Vec2 position;
  const Gobelin(this.nom, this.position);
}

void radar(Vec2 heros, List<Gobelin> gobelins, double porteeEpee) {
  final double porteeCarree = porteeEpee * porteeEpee;

  print('héros en $heros — portée d\'épée : ${porteeEpee.toStringAsFixed(0)} px');
  print('nom       | position     | distance | angle  | à portée');
  print('----------|--------------|----------|--------|---------');

  for (final Gobelin g in gobelins) {
    final double d = heros.distanceVers(g.position);
    final double a = (g.position - heros).angleDegres;
    // Test SANS racine carrée (23.6).
    final bool portee = heros.distanceCarreeVers(g.position) <= porteeCarree;

    print('${g.nom.padRight(9)} | '
        '${g.position.toString().padRight(12)} | '
        '${d.toStringAsFixed(0).padLeft(6)} px | '
        '${a.toStringAsFixed(0).padLeft(4)}°  | '
        '${portee ? 'OUI' : 'non'}');
  }
}

void main() {
  const Vec2 heros = Vec2(200, 200);

  radar(
    heros,
    const <Gobelin>[
      Gobelin('éclaireur', Vec2(260, 200)),
      Gobelin('archer', Vec2(200, 290)),
      Gobelin('brute', Vec2(140, 160)),
    ],
    80,
  );
}
```

**Résultat :**

```text
héros en (200, 200) — portée d'épée : 80 px
nom       | position     | distance | angle  | à portée
----------|--------------|----------|--------|---------
éclaireur | (260, 200)   |     60 px |    0°  | OUI
archer    | (200, 290)   |     90 px |   90°  | non
brute     | (140, 160)   |     72 px | -146°  | OUI
```

**Explication :** trois points à observer.

**Le calcul de l'angle se fait sur l'écart, pas sur la position.** On écrit `(g.position - heros).angleDegres` et non `g.position.angleDegres`. Cette seconde forme donnerait l'angle du gobelin vu depuis l'origine du repère, c'est-à-dire depuis le coin haut-gauche de l'écran — ce qui n'a aucun intérêt.

**L'archer est à 90 degrés, donc en dessous.** Le repère du `Canvas` a son `y` vers le bas (chapitre 21), donc un angle positif de 90 degrés désigne le sud de l'écran. L'archer est bien 90 pixels plus bas que le héros.

**La brute est à −146 degrés.** L'écart vaut `(-60, -40)` : à gauche et au-dessus. `atan2` renvoie une valeur négative proche de −180, ce qui correspond à « vers la gauche, légèrement vers le haut ». C'est exactement le cas que `atan(y / x)` aurait raté, puisque `-40 / -60` vaut le même rapport que `40 / 60`.

**Le test de portée utilise le carré.** Sur trois gobelins l'économie est nulle. Sur trois cents ennemis et soixante frames par seconde, cela fait dix-huit mille racines carrées évitées chaque seconde.

---

### Correction 3

```dart
import 'dart:math' as math;

class Vec2 {
  final double x;
  final double y;
  const Vec2(this.x, this.y);
  static const Vec2 zero = Vec2(0, 0);

  Vec2 operator *(double k) => Vec2(x * k, y * k);

  double get longueur => math.sqrt(x * x + y * y);

  Vec2 get normalise {
    final double l = longueur;
    return l == 0 ? Vec2.zero : Vec2(x / l, y / l);
  }

  @override
  String toString() =>
      '(${x.toStringAsFixed(2)}, ${y.toStringAsFixed(2)})';
}

Vec2 vitesseDepuis(bool g, bool d, bool h, bool b, double vMax) {
  double dx = 0;
  double dy = 0;
  if (g) dx -= 1;
  if (d) dx += 1;
  if (h) dy -= 1;
  if (b) dy += 1;

  // La normalisation traite d'un coup : les axes, les diagonales,
  // l'immobilité et les touches opposées qui s'annulent.
  return Vec2(dx, dy).normalise * vMax;
}

void main() {
  const double vMax = 200;

  const List<(String, bool, bool, bool, bool)> combinaisons =
      <(String, bool, bool, bool, bool)>[
    ('rien', false, false, false, false),
    ('gauche', true, false, false, false),
    ('droite', false, true, false, false),
    ('haut', false, false, true, false),
    ('bas', false, false, false, true),
    ('haut-gauche', true, false, true, false),
    ('haut-droite', false, true, true, false),
    ('bas-gauche', true, false, false, true),
    ('bas-droite', false, true, false, true),
    ('gauche+droite', true, true, false, false),
  ];

  print('combinaison   |          vitesse | longueur');
  print('--------------|------------------|---------');
  for (final (String nom, bool g, bool d, bool h, bool b) in combinaisons) {
    final Vec2 v = vitesseDepuis(g, d, h, b, vMax);
    print('${nom.padRight(13)} | ${v.toString().padLeft(16)} | '
        '${v.longueur.toStringAsFixed(2).padLeft(7)}');
  }
}
```

**Résultat :**

```text
combinaison   |          vitesse | longueur
--------------|------------------|---------
rien          |     (0.00, 0.00) |    0.00
gauche        |  (-200.00, 0.00) |  200.00
droite        |   (200.00, 0.00) |  200.00
haut          |  (0.00, -200.00) |  200.00
bas           |   (0.00, 200.00) |  200.00
haut-gauche   | (-141.42, -141.42) |  200.00
haut-droite   | (141.42, -141.42) |  200.00
bas-gauche    | (-141.42, 141.42) |  200.00
bas-droite    | (141.42, 141.42) |  200.00
gauche+droite |     (0.00, 0.00) |    0.00
```

**Explication :** toutes les longueurs non nulles valent exactement 200. Le bug de la diagonale de 23.8 est éliminé.

**Une seule ligne fait tout le travail.** `Vec2(dx, dy).normalise * vMax` couvre les neuf cas utiles plus le dixième. Il n'y a aucun `if` sur le nombre de touches enfoncées, aucune division par racine de 2 écrite à la main. C'est le bénéfice de la normalisation : elle est indifférente au nombre de composantes non nulles.

**Les touches opposées sont gérées gratuitement.** `gauche + droite` produit `dx = 0`, donc le vecteur nul, donc — grâce à la garde de `normalise` — un vecteur nul en sortie plutôt qu'un `NaN`. Sans cette garde, votre personnage disparaîtrait dès que le joueur appuierait sur les deux flèches, ce qui arrive constamment sur un clavier.

**Un détail de mise en forme.** Les lignes des diagonales dépassent la largeur de colonne parce que `(-141.42, -141.42)` fait 18 caractères. `padLeft(16)` ne tronque jamais : il complète seulement. C'est le comportement attendu, mais souvenez-vous-en quand vous alignez des colonnes.

---

### Correction 4

```dart
void main() {
  const double gravite = 1600;   // px/s²
  const double forceSaut = 620;  // px/s
  const double dt = 1 / 60;
  const double solY = 400;

  double y = solY;
  double vy = -forceSaut; // impulsion : NÉGATIVE (23.19)

  double hauteurMax = 0;
  double instantSommet = 0;
  int frame = 0;

  print('frame | temps |  hauteur |  vitesse vy');
  print('------|-------|----------|------------');

  while (true) {
    frame++;

    // Euler semi-implicite (23.16) : vitesse d'abord.
    vy += gravite * dt;
    y += vy * dt;

    if (y >= solY) {
      y = solY;
      break; // atterrissage
    }

    final double hauteur = solY - y;
    if (hauteur > hauteurMax) {
      hauteurMax = hauteur;
      instantSommet = frame * dt;
    }

    if (frame % 5 == 0) {
      print('${frame.toString().padLeft(5)} | '
          '${(frame * dt).toStringAsFixed(3)} | '
          '${hauteur.toStringAsFixed(2).padLeft(8)} | '
          '${vy.toStringAsFixed(2).padLeft(11)}');
    }
  }

  final double theorique = forceSaut * forceSaut / (2 * gravite);

  print('');
  print('hauteur maximale mesurée   : ${hauteurMax.toStringAsFixed(2)} px');
  print('instant du sommet          : ${instantSommet.toStringAsFixed(3)} s');
  print('durée totale du saut       : ${(frame * dt).toStringAsFixed(3)} s');
  print('hauteur théorique          : ${theorique.toStringAsFixed(2)} px');
  print('écart                      : '
      '${(theorique - hauteurMax).toStringAsFixed(2)} px');
}
```

**Résultat :**

```text
frame | temps |  hauteur |  vitesse vy
------|-------|----------|------------
    5 | 0.083 |    45.00 |     -486.67
   10 | 0.167 |    78.89 |     -353.33
   15 | 0.250 |   101.67 |     -220.00
   20 | 0.333 |   113.33 |      -86.67
   25 | 0.417 |   113.89 |       46.67
   30 | 0.500 |   103.33 |      180.00
   35 | 0.583 |    81.67 |      313.33
   40 | 0.667 |    48.89 |      446.67
   45 | 0.750 |     5.00 |      580.00

hauteur maximale mesurée   : 115.00 px
instant du sommet          : 0.383 s
durée totale du saut       : 0.767 s
hauteur théorique          : 120.13 px
écart                      : 5.13 px
```

**Explication :** la trajectoire suit exactement la parabole annoncée en 23.19.

**Le sommet est là où `vy` change de signe.** Entre la frame 20 (`vy = -86,67`) et la frame 25 (`vy = +46,67`), la vitesse traverse zéro. La hauteur maximale, 115 px, est atteinte à la frame 23, soit 0,383 s. La théorie donne `forceSaut / gravite = 620 / 1600 = 0,3875 s` : l'accord est bon.

**L'écart de 5 pixels est normal, et il s'explique.** La formule `v0² / (2g)` décrit un mouvement continu. Notre simulation avance par pas de 1/60 s : elle « rate » le vrai sommet, situé entre deux frames. L'erreur vaut environ `0,5 × g × dt × t`, soit ici `0,5 × 1600 × 0,0167 × 0,3875 ≈ 5,2` px. C'est exactement l'écart observé.

**Conséquence pratique de première importance.** Si vous réglez votre jeu pour que le héros franchisse un mur de 120 pixels en vous fiant à la formule théorique, il échouera. Réglez toujours **sur la valeur mesurée en simulation**, ou donnez-vous une marge de 10 %.

**Le saut dure 46 frames**, soit 0,767 s. La théorie donne `2 × v0 / g = 0,775 s`. Là encore, l'écart vient de la discrétisation.

---

### Correction 5

```dart
import 'dart:math' as math;

/// Simule un projectile et renvoie la portée horizontale.
double simuler({
  required bool semiImplicite,
  required double dt,
  required double v0,
  required double angleDegres,
  required double gravite,
}) {
  final double a = angleDegres * math.pi / 180;
  double x = 0, y = 0;
  double vx = math.cos(a) * v0;
  double vy = -math.sin(a) * v0; // vers le haut (23.3)

  int garde = 0;
  while (y < 0 || garde == 0) {
    garde++;
    if (garde > 1000000) break;

    if (semiImplicite) {
      vy += gravite * dt;
      y += vy * dt;
      x += vx * dt;
    } else {
      y += vy * dt;
      x += vx * dt;
      vy += gravite * dt;
    }
    if (y >= 0 && garde > 1) break;
  }
  return x;
}

void main() {
  const double v0 = 400;
  const double angle = 45;
  const double g = 1000;

  // Solution exacte : portée = v0² × sin(2a) / g
  final double exact =
      v0 * v0 * math.sin(2 * angle * math.pi / 180) / g;

  print('projectile : $v0 px/s à $angle°, gravité $g px/s²');
  print('portée exacte : ${exact.toStringAsFixed(2)} px');
  print('');
  print('   dt    | Euler explicite | Euler semi-implicite');
  print('---------|-----------------|---------------------');

  for (final (String nom, double dt) in <(String, double)>[
    ('1/30 s', 1 / 30),
    ('1/240 s', 1 / 240),
  ]) {
    final double exp = simuler(
        semiImplicite: false,
        dt: dt,
        v0: v0,
        angleDegres: angle,
        gravite: g);
    final double semi = simuler(
        semiImplicite: true,
        dt: dt,
        v0: v0,
        angleDegres: angle,
        gravite: g);
    print('${nom.padRight(8)} | ${exp.toStringAsFixed(2).padLeft(15)} | '
        '${semi.toStringAsFixed(2).padLeft(20)}');
  }
}
```

**Résultat :**

```text
projectile : 400.0 px/s à 45.0°, gravité 1000.0 px/s²
portée exacte : 160.00 px

   dt    | Euler explicite | Euler semi-implicite
---------|-----------------|---------------------
1/30 s   |          169.71 |               150.85
1/240 s  |          161.46 |               159.10
```

**Explication :** ce tableau est la démonstration chiffrée de la section 23.16.

**Les deux intégrateurs encadrent la vérité.** L'explicite surestime la portée (169,7 au lieu de 160), le semi-implicite la sous-estime (150,8). L'erreur est du même ordre de grandeur, mais de signe opposé. Ce n'est pas un hasard : l'un utilise systématiquement la vitesse du début de frame, l'autre celle de la fin.

**Diviser `dt` par 8 divise l'erreur par 8.** À `dt = 1/30`, les erreurs valent 9,7 et 9,2 pixels. À `dt = 1/240`, elles tombent à 1,5 et 0,9. C'est la signature d'une méthode d'ordre 1 : l'erreur est proportionnelle à `dt`.

**Alors pourquoi préférer le semi-implicite, s'il n'est pas plus précis ?** Parce que la précision n'est pas le critère. Le semi-implicite est **stable** : sur une simulation longue, il ne gagne pas d'énergie. L'explicite, lui, en gagne, et une balle qui rebondit finit par sortir de l'écran. Sur un tir de projectile qui dure une demi-seconde, la différence ne se voit pas ; sur une balle qui rebondit pendant une minute, elle est catastrophique.

**Le `garde` dans la boucle.** Une simulation qui ne se termine jamais gèle l'application. Écrivez systématiquement une borne de sécurité dans toute boucle `while` de simulation — c'est le genre de discipline qui vous évitera de forcer l'arrêt de votre émulateur.

---

### Correction 6

```dart
import 'dart:math' as math;

import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

class Vec2 {
  final double x;
  final double y;
  const Vec2(this.x, this.y);
  static const Vec2 zero = Vec2(0, 0);

  Vec2 operator +(Vec2 a) => Vec2(x + a.x, y + a.y);
  Vec2 operator *(double k) => Vec2(x * k, y * k);
  Vec2 copieAvec({double? x, double? y}) => Vec2(x ?? this.x, y ?? this.y);

  @override
  String toString() =>
      '(${x.toStringAsFixed(0)}, ${y.toStringAsFixed(0)})';
}

void main() => runApp(const BalleApp());

class BalleApp extends StatelessWidget {
  const BalleApp({super.key});

  @override
  Widget build(BuildContext context) => const MaterialApp(
        debugShowCheckedModeBanner: false,
        home: EcranBalle(),
      );
}

class EcranBalle extends StatefulWidget {
  const EcranBalle({super.key});

  @override
  State<EcranBalle> createState() => _EcranBalleState();
}

class _EcranBalleState extends State<EcranBalle>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  static const double rayon = 16;
  static const Vec2 gravite = Vec2(0, 1400);
  static const double restitution = 0.8;
  static const double amortissementParSeconde = 0.55; // ce qui RESTE en 1 s
  static const double seuilRepos = 45;

  Vec2 _position = const Vec2(80, 80);
  Vec2 _vitesse = const Vec2(260, 0);
  int _rebonds = 0;
  bool _endormie = false;
  Size _taille = const Size(360, 640);

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_frame)..start();
  }

  void _frame(Duration maintenant) {
    final double brut = (maintenant - _precedent).inMicroseconds / 1000000.0;
    _precedent = maintenant;
    if (brut <= 0) return;
    final double dt = brut > 0.05 ? 0.05 : brut;

    setState(() {
      // 1. Vitesse (23.15 : la vitesse AVANT la position).
      _vitesse += gravite * dt;

      // 2. Amortissement horizontal indépendant du framerate (23.23).
      final double facteur =
          math.pow(amortissementParSeconde, dt).toDouble();
      _vitesse = _vitesse.copieAvec(x: _vitesse.x * facteur);

      // 3. Position.
      _position += _vitesse * dt;

      // 4. Rebonds sur les quatre bords.
      if (_position.x - rayon < 0) {
        _position = _position.copieAvec(x: rayon);
        _vitesse = _vitesse.copieAvec(x: -_vitesse.x * restitution);
        _rebonds++;
      } else if (_position.x + rayon > _taille.width) {
        _position = _position.copieAvec(x: _taille.width - rayon);
        _vitesse = _vitesse.copieAvec(x: -_vitesse.x * restitution);
        _rebonds++;
      }

      if (_position.y - rayon < 0) {
        _position = _position.copieAvec(y: rayon);
        _vitesse = _vitesse.copieAvec(y: -_vitesse.y * restitution);
        _rebonds++;
      } else if (_position.y + rayon > _taille.height) {
        _position = _position.copieAvec(y: _taille.height - rayon);
        if (_vitesse.y.abs() < seuilRepos) {
          // Seuil d'endormissement (23.26) : on arrête le frémissement.
          _vitesse = _vitesse.copieAvec(y: 0);
          _endormie = true;
        } else {
          _vitesse = _vitesse.copieAvec(y: -_vitesse.y * restitution);
          _rebonds++;
        }
      }
    });
  }

  void _relancer() {
    setState(() {
      _position = const Vec2(80, 80);
      _vitesse = const Vec2(260, 0);
      _rebonds = 0;
      _endormie = false;
    });
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF16131F),
      body: LayoutBuilder(
        builder: (BuildContext context, BoxConstraints c) {
          _taille = Size(c.maxWidth, c.maxHeight);
          return GestureDetector(
            onTap: _relancer,
            child: CustomPaint(
              painter: PeintreBalle(
                position: _position,
                vitesse: _vitesse,
                rayon: rayon,
                rebonds: _rebonds,
                endormie: _endormie,
              ),
              child: const SizedBox.expand(),
            ),
          );
        },
      ),
    );
  }
}

class PeintreBalle extends CustomPainter {
  final Vec2 position;
  final Vec2 vitesse;
  final double rayon;
  final int rebonds;
  final bool endormie;

  PeintreBalle({
    required this.position,
    required this.vitesse,
    required this.rayon,
    required this.rebonds,
    required this.endormie,
  });

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawCircle(
      Offset(position.x, position.y),
      rayon,
      Paint()..color = endormie
          ? const Color(0xFF6B6480)
          : const Color(0xFFD9615A),
    );

    canvas.drawLine(
      Offset(position.x, position.y),
      Offset(position.x + vitesse.x / 6, position.y + vitesse.y / 6),
      Paint()
        ..color = const Color(0xFFE0B36F)
        ..strokeWidth = 2,
    );

    _texte(canvas, 'BALLE REBONDISSANTE — touchez pour relancer',
        const Offset(14, 14));
    _texte(canvas, 'rebonds  : $rebonds', const Offset(14, 34));
    _texte(canvas, 'vitesse  : $vitesse', const Offset(14, 50));
    _texte(canvas, 'endormie : $endormie', const Offset(14, 66));
  }

  void _texte(Canvas canvas, String s, Offset o) {
    final TextPainter tp = TextPainter(
      text: TextSpan(
        text: s,
        style: const TextStyle(
          color: Color(0xFFE8E4DC),
          fontSize: 12,
          fontFamily: 'monospace',
        ),
      ),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, o);
  }

  @override
  bool shouldRepaint(PeintreBalle old) => true;
}
```

**Résultat :**

```text
BALLE REBONDISSANTE — touchez pour relancer
rebonds  : 9
vitesse  : (48, -212)
endormie : false

Une balle rouge part du coin haut-gauche vers la droite.
Elle décrit une parabole, touche le sol, remonte à environ 64 % de sa
hauteur, rebondit sur les murs latéraux, et ses bonds diminuent
progressivement. Son déplacement horizontal ralentit visiblement à cause
de l'amortissement.
Au bout d'une dizaine de secondes elle se pose, devient grise et cesse
de bouger : le seuil d'endormissement a été franchi.
Une flèche orange, partant du centre de la balle, indique le vecteur
vitesse à chaque instant.
```

**Explication :** cinq points de vigilance.

**L'ordre de la mise à jour.** Vitesse, amortissement, position, puis rebonds. Les rebonds viennent **après** l'intégration : on corrige une position déjà fautive, comme pour le sol de 23.18.

**On corrige la position en même temps que la vitesse.** Sans `_position.copieAvec(x: rayon)`, la balle resterait un peu enfoncée dans le mur, et le test de rebond redeviendrait vrai à la frame suivante : elle collerait au bord en vibrant.

**Le seuil d'endormissement ne s'applique qu'au sol.** Sur les murs latéraux, la gravité ne pousse pas la balle contre la paroi ; il n'y a donc pas de frémissement à supprimer.

**L'amortissement horizontal utilise `pow`.** Écrit `_vitesse.x * 0.99` par frame, il aurait donné une balle qui roule deux fois plus loin sur un écran à 120 Hz.

**La restitution de 0,8 fait remonter à 64 %.** C'est la règle du carré de 23.26 : 0,8 × 0,8 = 0,64. En comptant les rebonds successifs, vous pouvez vérifier cette proportion à l'œil.

---

### Correction 7

```dart
import 'dart:math' as math;

import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

class Vec2 {
  final double x;
  final double y;
  const Vec2(this.x, this.y);
  static const Vec2 zero = Vec2(0, 0);

  Vec2 operator -(Vec2 a) => Vec2(x - a.x, y - a.y);
  double get longueurCarree => x * x + y * y;
  double get longueur => math.sqrt(longueurCarree);
  Vec2 lerp(Vec2 a, double t) => Vec2(x + (a.x - x) * t, y + (a.y - y) * t);
}

void main() => runApp(const FamilierApp());

class FamilierApp extends StatelessWidget {
  const FamilierApp({super.key});

  @override
  Widget build(BuildContext context) => const MaterialApp(
        debugShowCheckedModeBanner: false,
        home: EcranFamilier(),
      );
}

class EcranFamilier extends StatefulWidget {
  const EcranFamilier({super.key});

  @override
  State<EcranFamilier> createState() => _EcranFamilierState();
}

class _EcranFamilierState extends State<EcranFamilier>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  static const List<double> lambdas = <double>[3, 8, 20];
  int _indexLambda = 1;

  Vec2 _familier = const Vec2(180, 320);
  Vec2 _cible = const Vec2(180, 320);
  final List<Offset> _traine = <Offset>[];

  double get lambda => lambdas[_indexLambda];

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_frame)..start();
  }

  void _frame(Duration maintenant) {
    final double brut = (maintenant - _precedent).inMicroseconds / 1000000.0;
    _precedent = maintenant;
    if (brut <= 0) return;
    final double dt = brut > 0.05 ? 0.05 : brut;

    setState(() {
      // Suivi doux INDÉPENDANT du framerate (23.29).
      final double t = 1 - math.exp(-lambda * dt);
      _familier = _familier.lerp(_cible, t);

      // Garde d'arrivée : on colle à la cible sous un demi-pixel.
      if (_familier.distanceCarreeVersCible(_cible) < 0.25) {
        _familier = _cible;
      }

      _traine.add(Offset(_familier.x, _familier.y));
      if (_traine.length > 45) _traine.removeAt(0);
    });
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF16131F),
      body: Stack(
        children: <Widget>[
          Listener(
            onPointerDown: (PointerDownEvent e) => _cible =
                Vec2(e.localPosition.dx, e.localPosition.dy),
            onPointerMove: (PointerMoveEvent e) => _cible =
                Vec2(e.localPosition.dx, e.localPosition.dy),
            onPointerHover: (PointerHoverEvent e) => _cible =
                Vec2(e.localPosition.dx, e.localPosition.dy),
            child: CustomPaint(
              painter: PeintreFamilier(
                familier: _familier,
                cible: _cible,
                traine: _traine,
                lambda: lambda,
              ),
              child: const SizedBox.expand(),
            ),
          ),
          Positioned(
            right: 16,
            bottom: 16,
            child: ElevatedButton(
              onPressed: () => setState(() {
                _indexLambda = (_indexLambda + 1) % lambdas.length;
              }),
              child: Text('lambda = ${lambda.toStringAsFixed(0)}'),
            ),
          ),
        ],
      ),
    );
  }
}

extension on Vec2 {
  double distanceCarreeVersCible(Vec2 a) => (a - this).longueurCarree;
}

class PeintreFamilier extends CustomPainter {
  final Vec2 familier;
  final Vec2 cible;
  final List<Offset> traine;
  final double lambda;

  PeintreFamilier({
    required this.familier,
    required this.cible,
    required this.traine,
    required this.lambda,
  });

  @override
  void paint(Canvas canvas, Size size) {
    // La traîne : plus ancienne = plus transparente.
    for (int i = 0; i < traine.length; i++) {
      final double a = (i + 1) / traine.length;
      canvas.drawCircle(
        traine[i],
        3 + 5 * a,
        Paint()..color = Color.fromRGBO(111, 179, 224, a * 0.35),
      );
    }

    // La cible.
    canvas.drawCircle(
      Offset(cible.x, cible.y),
      10,
      Paint()
        ..color = const Color(0xFFE0B36F)
        ..style = PaintingStyle.stroke
        ..strokeWidth = 2,
    );

    // Le familier.
    canvas.drawRRect(
      RRect.fromRectAndRadius(
        Rect.fromCenter(
          center: Offset(familier.x, familier.y),
          width: 24,
          height: 24,
        ),
        const Radius.circular(6),
      ),
      Paint()..color = const Color(0xFF6FB3E0),
    );

    final double reste = (cible - familier).longueur;
    _texte(canvas, 'SUIVI DOUX — déplacez le doigt ou la souris',
        const Offset(14, 14));
    _texte(canvas, 'lambda : ${lambda.toStringAsFixed(0)}',
        const Offset(14, 34));
    _texte(canvas, 'reste  : ${reste.toStringAsFixed(1)} px',
        const Offset(14, 50));
    _texte(
      canvas,
      't par frame a 60 FPS : '
      '${(1 - math.exp(-lambda / 60)).toStringAsFixed(3)}',
      const Offset(14, 66),
    );
  }

  void _texte(Canvas canvas, String s, Offset o) {
    final TextPainter tp = TextPainter(
      text: TextSpan(
        text: s,
        style: const TextStyle(
          color: Color(0xFFE8E4DC),
          fontSize: 12,
          fontFamily: 'monospace',
        ),
      ),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, o);
  }

  @override
  bool shouldRepaint(PeintreFamilier old) => true;
}
```

**Résultat :**

```text
SUIVI DOUX — déplacez le doigt ou la souris
lambda : 8
reste  : 27.4 px
t par frame a 60 FPS : 0.125

Un carré bleu suit le doigt avec un léger retard, en laissant derrière lui
une traînée de cercles qui s'estompent. Un cercle orange marque la position
visée.
Avec lambda = 3, le carré traîne loin derrière et met près d'une seconde à
rejoindre le doigt immobile.
Avec lambda = 8, il suit de près sans être collé : c'est le réglage le plus
agréable.
Avec lambda = 20, il est presque toujours sous le doigt et l'effet de
lissage devient imperceptible.
```

**Explication :** quatre remarques.

**La formule `1 - exp(-lambda * dt)` est le cœur de l'exercice.** Elle transforme une « vitesse de rattrapage par seconde » en un facteur d'interpolation valable pour la frame courante. À 60 FPS et `lambda = 8`, elle vaut 0,125 : le familier comble 12,5 % de l'écart à chaque frame. À 120 FPS, elle vaudrait 0,0645 — deux fois moins par frame, donc le même résultat par seconde.

**La traîne visualise la courbe de rattrapage.** Les cercles sont serrés quand le familier est proche de la cible et espacés quand il en est loin. C'est la signature du rapprochement asymptotique décrit en 23.28, usage 2.

**La garde d'arrivée est indispensable.** Sans le test `< 0.25`, `reste` afficherait indéfiniment des valeurs comme `0.0003` et le `Ticker` continuerait de recalculer. Sur une caméra, ce résidu produit un tremblement d'un demi-pixel très visible en pixel art.

**Une extension pour une seule méthode.** `distanceCarreeVersCible` est ajoutée par une `extension` plutôt que dans la classe. C'est un choix stylistique — la classe `Vec2` de cet exemple est volontairement minimale. En production, mettez-la directement dans `Vec2`.

---

### Correction 8

```dart
import 'dart:math' as math;

import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

class Vec2 {
  final double x;
  final double y;
  const Vec2(this.x, this.y);
  static const Vec2 zero = Vec2(0, 0);

  Vec2 operator +(Vec2 a) => Vec2(x + a.x, y + a.y);
  Vec2 operator -(Vec2 a) => Vec2(x - a.x, y - a.y);
  Vec2 operator *(double k) => Vec2(x * k, y * k);

  double get longueur => math.sqrt(x * x + y * y);
  double get angle => math.atan2(y, x);

  Vec2 get normalise {
    final double l = longueur;
    return l == 0 ? Vec2.zero : Vec2(x / l, y / l);
  }

  factory Vec2.depuisAngle(double r, [double l = 1]) =>
      Vec2(math.cos(r) * l, math.sin(r) * l);
}

class Projectile {
  Vec2 position;
  final Vec2 vitesse;
  Projectile(this.position, this.vitesse);

  void mettreAJour(double dt) => position += vitesse * dt;

  bool horsEcran(Size t) =>
      position.x < -20 ||
      position.y < -20 ||
      position.x > t.width + 20 ||
      position.y > t.height + 20;
}

void main() => runApp(const TourelleApp());

class TourelleApp extends StatelessWidget {
  const TourelleApp({super.key});

  @override
  Widget build(BuildContext context) => const MaterialApp(
        debugShowCheckedModeBanner: false,
        home: EcranTourelle(),
      );
}

class EcranTourelle extends StatefulWidget {
  const EcranTourelle({super.key});

  @override
  State<EcranTourelle> createState() => _EcranTourelleState();
}

class _EcranTourelleState extends State<EcranTourelle>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  static const double cadence = 0.4;       // secondes entre deux tirs
  static const double vitesseTir = 320;    // px/s
  static const double longueurCanon = 46;

  Size _taille = const Size(360, 640);
  Vec2 _tourelle = const Vec2(180, 320);
  Vec2 _visee = const Vec2(300, 200);
  double _rechargement = 0;
  final List<Projectile> _projectiles = <Projectile>[];

  double get angle => (_visee - _tourelle).angle;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_frame)..start();
  }

  void _frame(Duration maintenant) {
    final double brut = (maintenant - _precedent).inMicroseconds / 1000000.0;
    _precedent = maintenant;
    if (brut <= 0) return;
    final double dt = brut > 0.05 ? 0.05 : brut;

    setState(() {
      _rechargement -= dt;
      if (_rechargement <= 0) {
        _rechargement = cadence;
        final Vec2 direction = (_visee - _tourelle).normalise;
        if (direction != Vec2.zero) {
          _projectiles.add(Projectile(
            _tourelle + direction * longueurCanon,
            direction * vitesseTir,
          ));
        }
      }

      for (final Projectile p in _projectiles) {
        p.mettreAJour(dt);
      }
      _projectiles.removeWhere((Projectile p) => p.horsEcran(_taille));
    });
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF16131F),
      body: LayoutBuilder(
        builder: (BuildContext context, BoxConstraints c) {
          _taille = Size(c.maxWidth, c.maxHeight);
          _tourelle = Vec2(c.maxWidth / 2, c.maxHeight / 2);
          return Listener(
            onPointerDown: (PointerDownEvent e) => _visee =
                Vec2(e.localPosition.dx, e.localPosition.dy),
            onPointerMove: (PointerMoveEvent e) => _visee =
                Vec2(e.localPosition.dx, e.localPosition.dy),
            onPointerHover: (PointerHoverEvent e) => _visee =
                Vec2(e.localPosition.dx, e.localPosition.dy),
            child: CustomPaint(
              painter: PeintreTourelle(
                tourelle: _tourelle,
                visee: _visee,
                angle: angle,
                projectiles: _projectiles,
                longueurCanon: longueurCanon,
              ),
              child: const SizedBox.expand(),
            ),
          );
        },
      ),
    );
  }
}

class PeintreTourelle extends CustomPainter {
  final Vec2 tourelle;
  final Vec2 visee;
  final double angle;
  final List<Projectile> projectiles;
  final double longueurCanon;

  PeintreTourelle({
    required this.tourelle,
    required this.visee,
    required this.angle,
    required this.projectiles,
    required this.longueurCanon,
  });

  @override
  void paint(Canvas canvas, Size size) {
    // Les projectiles.
    final Paint pTir = Paint()..color = const Color(0xFFE0B36F);
    for (final Projectile p in projectiles) {
      canvas.drawCircle(Offset(p.position.x, p.position.y), 5, pTir);
    }

    // Le canon : on tourne le repère (chapitre 21).
    canvas.save();
    canvas.translate(tourelle.x, tourelle.y);
    canvas.rotate(angle);
    canvas.drawRRect(
      RRect.fromRectAndRadius(
        Rect.fromLTWH(0, -7, longueurCanon, 14),
        const Radius.circular(4),
      ),
      Paint()..color = const Color(0xFF9AD86F),
    );
    canvas.restore();

    // Le socle.
    canvas.drawCircle(
      Offset(tourelle.x, tourelle.y),
      20,
      Paint()..color = const Color(0xFF6FB3E0),
    );

    // Le réticule.
    canvas.drawCircle(
      Offset(visee.x, visee.y),
      12,
      Paint()
        ..color = const Color(0x88E8E4DC)
        ..style = PaintingStyle.stroke
        ..strokeWidth = 2,
    );

    _texte(canvas, 'TOURELLE DU DONJON — visez avec le doigt',
        const Offset(14, 14));
    _texte(canvas, 'angle       : '
        '${(angle * 180 / math.pi).toStringAsFixed(0)}°',
        const Offset(14, 34));
    _texte(canvas, 'projectiles : ${projectiles.length}',
        const Offset(14, 50));
  }

  void _texte(Canvas canvas, String s, Offset o) {
    final TextPainter tp = TextPainter(
      text: TextSpan(
        text: s,
        style: const TextStyle(
          color: Color(0xFFE8E4DC),
          fontSize: 12,
          fontFamily: 'monospace',
        ),
      ),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, o);
  }

  @override
  bool shouldRepaint(PeintreTourelle old) => true;
}
```

**Résultat :**

```text
TOURELLE DU DONJON — visez avec le doigt
angle       : -37°
projectiles : 4

Un socle bleu au centre de l'écran, prolongé par un canon vert qui pivote
en permanence vers le doigt. Un cercle blanc marque le point visé.
Toutes les 0,4 seconde, une bille orange part de l'extrémité du canon,
file en ligne droite dans la direction visée, et disparaît en sortant de
l'écran. Le compteur oscille entre 3 et 5 projectiles vivants.
```

**Explication :** quatre mécanismes réunis.

**L'angle vient de `atan2` sur l'écart.** `(visee - tourelle).angle` : encore une fois, on calcule l'angle d'un **écart**, jamais d'une position absolue.

**Le canon est dessiné à l'horizontale, puis tourné.** `Rect.fromLTWH(0, -7, 46, 14)` décrit un canon qui part de l'origine vers la droite. Le `canvas.rotate(angle)` s'occupe du reste. C'est bien plus simple que de calculer les quatre coins du rectangle tourné, et c'est la technique du chapitre 21.

**Le projectile naît au bout du canon.** `_tourelle + direction * longueurCanon` place l'origine du tir à l'extrémité, pas au centre du socle. Sans cela, la bille apparaîtrait à l'intérieur de la tourelle et le tir semblerait sortir du mauvais endroit.

**Le nettoyage est indispensable.** `removeWhere` supprime les projectiles hors écran. Sans lui, la liste grandirait indéfiniment : après dix minutes, quinze cents objets seraient mis à jour chaque frame pour rien. C'est un des premiers réflexes de performance à acquérir.

---

### Correction 9

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

class Vec2 {
  final double x;
  final double y;
  const Vec2(this.x, this.y);
  static const Vec2 zero = Vec2(0, 0);
  Vec2 operator +(Vec2 a) => Vec2(x + a.x, y + a.y);
  Vec2 operator *(double k) => Vec2(x * k, y * k);
  Vec2 copieAvec({double? x, double? y}) => Vec2(x ?? this.x, y ?? this.y);
}

void main() => runApp(const ReglageApp());

class ReglageApp extends StatelessWidget {
  const ReglageApp({super.key});

  @override
  Widget build(BuildContext context) => const MaterialApp(
        debugShowCheckedModeBanner: false,
        home: EcranReglage(),
      );
}

class EcranReglage extends StatefulWidget {
  const EcranReglage({super.key});

  @override
  State<EcranReglage> createState() => _EcranReglageState();
}

class _EcranReglageState extends State<EcranReglage>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  // LES DEUX SEULS RÉGLAGES EXPOSÉS AU CONCEPTEUR.
  double _hauteurVoulue = 120; // px
  double _dureeMontee = 0.38;  // s

  // Les valeurs physiques en DÉCOULENT.
  double get gravite => 2 * _hauteurVoulue / (_dureeMontee * _dureeMontee);
  double get forceSaut => 2 * _hauteurVoulue / _dureeMontee;

  static const double demiHauteur = 22;

  Vec2 _position = const Vec2(120, 300);
  Vec2 _vitesse = Vec2.zero;
  bool _auSol = false;
  double _solY = 400;
  double _hauteurAtteinte = 0;
  double _hauteurMesuree = 0;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_frame)..start();
  }

  void _frame(Duration maintenant) {
    final double brut = (maintenant - _precedent).inMicroseconds / 1000000.0;
    _precedent = maintenant;
    if (brut <= 0) return;
    final double dt = brut > 0.05 ? 0.05 : brut;

    setState(() {
      _vitesse = _vitesse.copieAvec(y: _vitesse.y + gravite * dt);
      _position += _vitesse * dt;

      if (_position.y + demiHauteur >= _solY) {
        _position = _position.copieAvec(y: _solY - demiHauteur);
        _vitesse = _vitesse.copieAvec(y: 0);
        if (!_auSol) _hauteurMesuree = _hauteurAtteinte;
        _auSol = true;
      } else {
        _auSol = false;
        final double h = _solY - demiHauteur - _position.y;
        if (h > _hauteurAtteinte) _hauteurAtteinte = h;
      }
    });
  }

  void _sauter() {
    if (!_auSol) return;
    setState(() {
      _vitesse = _vitesse.copieAvec(y: -forceSaut);
      _auSol = false;
      _hauteurAtteinte = 0;
    });
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF16131F),
      body: Column(
        children: <Widget>[
          Expanded(
            child: LayoutBuilder(
              builder: (BuildContext context, BoxConstraints c) {
                _solY = c.maxHeight - 40;
                return GestureDetector(
                  onTap: _sauter,
                  child: CustomPaint(
                    painter: PeintreReglage(
                      position: _position,
                      solY: _solY,
                      hauteurVoulue: _hauteurVoulue,
                      hauteurMesuree: _hauteurMesuree,
                      gravite: gravite,
                      forceSaut: forceSaut,
                      dureeMontee: _dureeMontee,
                    ),
                    child: const SizedBox.expand(),
                  ),
                );
              },
            ),
          ),
          _curseur('hauteur voulue', _hauteurVoulue, 40, 260,
              (double v) => setState(() => _hauteurVoulue = v), 'px'),
          _curseur('durée de montée', _dureeMontee, 0.15, 0.80,
              (double v) => setState(() => _dureeMontee = v), 's'),
          const SizedBox(height: 8),
        ],
      ),
    );
  }

  Widget _curseur(String nom, double valeur, double min, double max,
      ValueChanged<double> onChanged, String unite) {
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 12),
      child: Row(
        children: <Widget>[
          SizedBox(
            width: 130,
            child: Text(
              nom,
              style: const TextStyle(
                  color: Color(0xFFE8E4DC), fontSize: 12),
            ),
          ),
          Expanded(
            child: Slider(
              value: valeur,
              min: min,
              max: max,
              onChanged: onChanged,
            ),
          ),
          SizedBox(
            width: 62,
            child: Text(
              '${valeur.toStringAsFixed(2)} $unite',
              style: const TextStyle(
                  color: Color(0xFFE0B36F), fontSize: 12),
            ),
          ),
        ],
      ),
    );
  }
}

class PeintreReglage extends CustomPainter {
  final Vec2 position;
  final double solY;
  final double hauteurVoulue;
  final double hauteurMesuree;
  final double gravite;
  final double forceSaut;
  final double dureeMontee;

  PeintreReglage({
    required this.position,
    required this.solY,
    required this.hauteurVoulue,
    required this.hauteurMesuree,
    required this.gravite,
    required this.forceSaut,
    required this.dureeMontee,
  });

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
      Rect.fromLTWH(0, solY, size.width, size.height - solY),
      Paint()..color = const Color(0xFF332E45),
    );

    // Ligne repère de la hauteur VOULUE.
    final double yVoulue = solY - 22 - hauteurVoulue;
    canvas.drawLine(
      Offset(0, yVoulue),
      Offset(size.width, yVoulue),
      Paint()
        ..color = const Color(0xFFE0B36F)
        ..strokeWidth = 2,
    );

    // Ligne de la hauteur RÉELLEMENT atteinte au dernier saut.
    final double yMesuree = solY - 22 - hauteurMesuree;
    canvas.drawLine(
      Offset(0, yMesuree),
      Offset(size.width, yMesuree),
      Paint()
        ..color = const Color(0xFF9AD86F)
        ..strokeWidth = 1,
    );

    canvas.drawRRect(
      RRect.fromRectAndRadius(
        Rect.fromCenter(
          center: Offset(position.x, position.y),
          width: 28,
          height: 44,
        ),
        const Radius.circular(6),
      ),
      Paint()..color = const Color(0xFF6FB3E0),
    );

    _texte(canvas, 'RÉGLAGE PAR INTENTION — touchez pour sauter',
        const Offset(14, 14));
    _texte(canvas, 'gravité calculée   : '
        '${gravite.toStringAsFixed(0)} px/s²', const Offset(14, 34));
    _texte(canvas, 'force de saut      : '
        '${forceSaut.toStringAsFixed(0)} px/s', const Offset(14, 50));
    _texte(canvas, 'hauteur voulue     : '
        '${hauteurVoulue.toStringAsFixed(0)} px (ligne orange)',
        const Offset(14, 66));
    _texte(canvas, 'hauteur atteinte   : '
        '${hauteurMesuree.toStringAsFixed(1)} px (ligne verte)',
        const Offset(14, 82));
  }

  void _texte(Canvas canvas, String s, Offset o) {
    final TextPainter tp = TextPainter(
      text: TextSpan(
        text: s,
        style: const TextStyle(
          color: Color(0xFFE8E4DC),
          fontSize: 12,
          fontFamily: 'monospace',
        ),
      ),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, o);
  }

  @override
  bool shouldRepaint(PeintreReglage old) => true;
}
```

**Résultat :**

```text
RÉGLAGE PAR INTENTION — touchez pour sauter
gravité calculée   : 1662 px/s²
force de saut      : 632 px/s
hauteur voulue     : 120 px (ligne orange)
hauteur atteinte   : 114.8 px (ligne verte)

Un rectangle bleu posé sur un sol violet. Une ligne orange marque la
hauteur demandée, une ligne verte la hauteur réellement atteinte au
dernier saut ; les deux sont très proches.
Deux curseurs en bas de l'écran modifient la hauteur voulue et la durée
de montée. Le personnage saute toujours à la hauteur affichée, quelle que
soit la durée choisie : une durée courte donne un saut vif et lourd, une
durée longue un saut flottant.
```

**Explication :** cet exercice inverse la logique de réglage, et c'est une méthode de travail professionnelle.

**D'où viennent les deux formules ?** Elles se déduisent des deux relations connues. Au sommet, la vitesse est nulle, donc `0 = forceSaut - gravite × duree`, ce qui donne `forceSaut = gravite × duree`. Et la hauteur parcourue vaut `hauteur = forceSaut × duree / 2` (la vitesse moyenne durant la montée étant la moitié de la vitesse initiale). En combinant les deux, on obtient `forceSaut = 2 × hauteur / duree` puis `gravite = 2 × hauteur / duree²`.

**On règle une intention, pas un nombre abstrait.** « Le héros franchit un mur de 120 px en 0,38 s » est une décision de game design. « La gravité vaut 1662 » n'en est pas une. Cette inversion est ce qui distingue un réglage maîtrisé d'un tâtonnement.

**La ligne verte reste sous l'orange.** L'écart de 4 à 5 pixels est celui, déjà identifié à la correction 4, dû à la discrétisation du temps. Il est constant en proportion : si vous voulez la précision au pixel, majorez `hauteurVoulue` de 4 %.

**Le `if (!_auSol)` à l'atterrissage.** On enregistre `_hauteurMesuree` uniquement à la frame d'atterrissage, pas pendant tout le séjour au sol. C'est encore la détection de transition de 23.20.

---

### Correction 10

```dart
import 'dart:math' as math;

import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

// ============================================================ Vec2
class Vec2 {
  final double x;
  final double y;
  const Vec2(this.x, this.y);
  static const Vec2 zero = Vec2(0, 0);

  Vec2 operator +(Vec2 a) => Vec2(x + a.x, y + a.y);
  Vec2 operator -(Vec2 a) => Vec2(x - a.x, y - a.y);
  Vec2 operator *(double k) => Vec2(x * k, y * k);

  double get longueur => math.sqrt(x * x + y * y);
  Vec2 copieAvec({double? x, double? y}) => Vec2(x ?? this.x, y ?? this.y);

  @override
  String toString() =>
      '(${x.toStringAsFixed(0)}, ${y.toStringAsFixed(0)})';
}

// ============================================================ entrées
class Entrees {
  bool gauche = false;
  bool droite = false;
  bool sautMaintenu = false;
  bool sautPresse = false;

  double get axeX => (droite ? 1 : 0) - (gauche ? 1 : 0);
  void finDeFrame() => sautPresse = false;
}

// ============================================================ plateforme
class Plateforme {
  Vec2 position;
  double vitesseX;
  final double xMin;
  final double xMax;
  static const double largeur = 120;
  static const double epaisseur = 16;

  Plateforme(this.position, this.vitesseX, this.xMin, this.xMax);

  void mettreAJour(double dt) {
    position += Vec2(vitesseX, 0) * dt;
    if (position.x < xMin) {
      position = position.copieAvec(x: xMin);
      vitesseX = -vitesseX;
    } else if (position.x > xMax) {
      position = position.copieAvec(x: xMax);
      vitesseX = -vitesseX;
    }
  }

  double get haut => position.y - epaisseur / 2;
  double get gaucheX => position.x - largeur / 2;
  double get droiteX => position.x + largeur / 2;

  Rect get boite => Rect.fromCenter(
        center: Offset(position.x, position.y),
        width: largeur,
        height: epaisseur,
      );
}

// ============================================================ héros
class Heros {
  Vec2 position;
  Vec2 vitesse = Vec2.zero;

  static const double vitesseMax = 230;
  static const double acceleration = 1500;
  static const double deceleration = 1900;
  static const double demiTour = 2800;
  static const double frottementAir = 700;

  static const double gravite = 2000;
  static const double graviteChute = 2800;
  static const double forceSaut = 700;
  static const double coupureSaut = 0.35;
  static const double vitesseChuteMax = 1100;
  static const double coyoteDuree = 0.10;
  static const double bufferDuree = 0.12;

  // Nouveautés de l'exercice 10.
  static const double glisseMurMax = 120;
  static const double sautMurX = 280;
  static const double sautMurY = 620;
  static const double blocageSautMur = 0.15;

  static const double largeur = 26;
  static const double hauteur = 42;
  double get demiL => largeur / 2;
  double get demiH => hauteur / 2;

  bool auSol = false;
  int murTouche = 0; // -1 mur à gauche, +1 mur à droite, 0 aucun
  bool glisseAuMur = false;
  double coyoteRestant = 0;
  double bufferRestant = 0;
  double controleBloqueRestant = 0;
  int regard = 1;

  Heros(this.position);

  bool get controlable => controleBloqueRestant <= 0;

  void mettreAJour(
    double dt,
    Entrees e,
    double solY,
    double largeurMonde,
    List<Plateforme> plateformes,
  ) {
    final bool etaitAuSol = auSol;
    final double yAvant = position.y;

    // --- chronomètres ---
    if (controleBloqueRestant > 0) controleBloqueRestant -= dt;
    if (auSol) {
      coyoteRestant = coyoteDuree;
    } else if (coyoteRestant > 0) {
      coyoteRestant -= dt;
    }
    if (e.sautPresse) bufferRestant = bufferDuree;
    if (bufferRestant > 0) bufferRestant -= dt;

    // --- détection du mur (avant tout déplacement) ---
    final double axe = controlable ? e.axeX : 0;
    murTouche = 0;
    if (!auSol) {
      if (position.x - demiL <= 0.5 && axe < 0) murTouche = -1;
      if (position.x + demiL >= largeurMonde - 0.5 && axe > 0) murTouche = 1;
    }
    glisseAuMur = murTouche != 0 && vitesse.y > 0;

    // --- saut ---
    if (bufferRestant > 0) {
      if (coyoteRestant > 0) {
        vitesse = vitesse.copieAvec(y: -forceSaut);
        auSol = false;
        coyoteRestant = 0;
        bufferRestant = 0;
      } else if (murTouche != 0) {
        // Saut mural : on repart à l'OPPOSÉ du mur.
        vitesse = Vec2(-murTouche * sautMurX, -sautMurY);
        controleBloqueRestant = blocageSautMur;
        bufferRestant = 0;
        murTouche = 0;
        glisseAuMur = false;
        regard = vitesse.x > 0 ? 1 : -1;
      }
    }

    // --- course ---
    final double axeEffectif = controlable ? e.axeX : 0;
    if (axeEffectif != 0) regard = axeEffectif > 0 ? 1 : -1;
    final double cible = axeEffectif * vitesseMax;
    double taux;
    if (axeEffectif == 0) {
      taux = auSol ? deceleration : frottementAir;
    } else if (vitesse.x != 0 && axeEffectif.sign != vitesse.x.sign) {
      taux = demiTour;
    } else {
      taux = acceleration;
    }
    final double ecart = cible - vitesse.x;
    final double pas = taux * dt;
    vitesse = vitesse.copieAvec(
      x: ecart.abs() <= pas ? cible : vitesse.x + (ecart > 0 ? pas : -pas),
    );

    // --- saut variable et gravité ---
    if (!e.sautMaintenu && vitesse.y < 0) {
      vitesse = vitesse.copieAvec(y: vitesse.y * coupureSaut);
    }
    final double g = vitesse.y < 0 ? gravite : graviteChute;
    vitesse = vitesse.copieAvec(y: vitesse.y + g * dt);

    // Glissade murale : la chute est fortement plafonnée.
    final double plafond = glisseAuMur ? glisseMurMax : vitesseChuteMax;
    if (vitesse.y > plafond) vitesse = vitesse.copieAvec(y: plafond);

    // --- intégration ---
    position += vitesse * dt;

    // --- plateformes mobiles : on ne les touche QUE par le dessus ---
    auSol = false;
    for (final Plateforme p in plateformes) {
      final bool auDessus = yAvant + demiH <= p.haut + 1;
      final bool traverse = position.y + demiH >= p.haut;
      final bool dansLaLargeur =
          position.x + demiL > p.gaucheX && position.x - demiL < p.droiteX;

      if (auDessus && traverse && dansLaLargeur && vitesse.y >= 0) {
        position = position.copieAvec(y: p.haut - demiH);
        vitesse = vitesse.copieAvec(y: 0);
        auSol = true;
        // La plateforme ENTRAÎNE le héros.
        position += Vec2(p.vitesseX * dt, 0);
      }
    }

    // --- sol ---
    if (position.y + demiH >= solY) {
      position = position.copieAvec(y: solY - demiH);
      vitesse = vitesse.copieAvec(y: 0);
      auSol = true;
    }

    // --- murs ---
    if (position.x - demiL < 0) {
      position = position.copieAvec(x: demiL);
      if (vitesse.x < 0) vitesse = vitesse.copieAvec(x: 0);
    }
    if (position.x + demiL > largeurMonde) {
      position = position.copieAvec(x: largeurMonde - demiL);
      if (vitesse.x > 0) vitesse = vitesse.copieAvec(x: 0);
    }

    if (auSol && !etaitAuSol) coyoteRestant = coyoteDuree;
  }

  Rect get boite => Rect.fromCenter(
        center: Offset(position.x, position.y),
        width: largeur,
        height: hauteur,
      );
}

// ============================================================ application
void main() => runApp(const DonjonApp());

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) => const MaterialApp(
        debugShowCheckedModeBanner: false,
        home: EcranDonjon(),
      );
}

class EcranDonjon extends StatefulWidget {
  const EcranDonjon({super.key});

  @override
  State<EcranDonjon> createState() => _EcranDonjonState();
}

class _EcranDonjonState extends State<EcranDonjon>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  final Entrees _entrees = Entrees();
  late Heros _heros;
  List<Plateforme> _plateformes = <Plateforme>[];

  double _solY = 460;
  double _largeurMonde = 360;
  bool _pret = false;

  @override
  void initState() {
    super.initState();
    _heros = Heros(const Vec2(60, 300));
    _ticker = createTicker(_frame)..start();
  }

  void _preparer(Size t) {
    if (_pret) return;
    _pret = true;
    _solY = t.height - 70;
    _largeurMonde = t.width;
    _heros = Heros(Vec2(60, _solY - Heros.hauteur / 2));
    _plateformes = <Plateforme>[
      Plateforme(Vec2(t.width * 0.5, _solY - 130), 90, t.width * 0.25,
          t.width * 0.75),
      Plateforme(Vec2(t.width * 0.5, _solY - 250), -60, t.width * 0.20,
          t.width * 0.80),
    ];
  }

  void _frame(Duration maintenant) {
    final double brut = (maintenant - _precedent).inMicroseconds / 1000000.0;
    _precedent = maintenant;
    if (brut <= 0) return;
    final double dt = brut > 0.05 ? 0.05 : brut;

    setState(() {
      for (final Plateforme p in _plateformes) {
        p.mettreAJour(dt);
      }
      _heros.mettreAJour(
          dt, _entrees, _solY, _largeurMonde, _plateformes);
      _entrees.finDeFrame();
    });
  }

  Widget _bouton(String s, VoidCallback debut, VoidCallback fin) {
    return Listener(
      onPointerDown: (_) => debut(),
      onPointerUp: (_) => fin(),
      onPointerCancel: (_) => fin(),
      child: Container(
        width: 74,
        height: 60,
        alignment: Alignment.center,
        decoration: BoxDecoration(
          color: const Color(0x33FFFFFF),
          borderRadius: BorderRadius.circular(10),
        ),
        child: Text(
          s,
          style: const TextStyle(
              color: Color(0xFFE8E4DC),
              fontSize: 20,
              fontFamily: 'monospace'),
        ),
      ),
    );
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF15121F),
      body: LayoutBuilder(
        builder: (BuildContext context, BoxConstraints c) {
          _preparer(Size(c.maxWidth, c.maxHeight));
          return Stack(
            children: <Widget>[
              CustomPaint(
                painter: PeintreDonjon(
                  heros: _heros,
                  plateformes: _plateformes,
                  solY: _solY,
                ),
                child: const SizedBox.expand(),
              ),
              Positioned(
                left: 14,
                bottom: 14,
                child: Row(
                  children: <Widget>[
                    _bouton('<', () => _entrees.gauche = true,
                        () => _entrees.gauche = false),
                    const SizedBox(width: 10),
                    _bouton('>', () => _entrees.droite = true,
                        () => _entrees.droite = false),
                  ],
                ),
              ),
              Positioned(
                right: 14,
                bottom: 14,
                child: _bouton('^', () {
                  if (!_entrees.sautMaintenu) _entrees.sautPresse = true;
                  _entrees.sautMaintenu = true;
                }, () => _entrees.sautMaintenu = false),
              ),
            ],
          );
        },
      ),
    );
  }
}

class PeintreDonjon extends CustomPainter {
  final Heros heros;
  final List<Plateforme> plateformes;
  final double solY;

  PeintreDonjon({
    required this.heros,
    required this.plateformes,
    required this.solY,
  });

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
      Rect.fromLTWH(0, solY, size.width, size.height - solY),
      Paint()..color = const Color(0xFF332E45),
    );

    final Paint pPlate = Paint()..color = const Color(0xFF8A7BC8);
    for (final Plateforme p in plateformes) {
      canvas.drawRRect(
        RRect.fromRectAndRadius(p.boite, const Radius.circular(5)),
        pPlate,
      );
    }

    canvas.drawRRect(
      RRect.fromRectAndRadius(heros.boite, const Radius.circular(6)),
      Paint()..color = heros.glisseAuMur
          ? const Color(0xFFE0B36F)
          : heros.auSol
              ? const Color(0xFF6FB3E0)
              : const Color(0xFF9AD86F),
    );

    _texte(canvas, 'DONJON DE DART — plateformes et saut mural',
        const Offset(14, 14));
    _texte(canvas, 'vitesse     ${heros.vitesse}', const Offset(14, 34));
    _texte(canvas, 'auSol       ${heros.auSol}', const Offset(14, 50));
    _texte(canvas, 'mur         ${heros.murTouche}', const Offset(14, 66));
    _texte(canvas, 'glisse      ${heros.glisseAuMur}', const Offset(14, 82));
    _texte(canvas, 'controlable ${heros.controlable}', const Offset(14, 98));
  }

  void _texte(Canvas canvas, String s, Offset o) {
    final TextPainter tp = TextPainter(
      text: TextSpan(
        text: s,
        style: const TextStyle(
          color: Color(0xFFE8E4DC),
          fontSize: 12,
          fontFamily: 'monospace',
        ),
      ),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, o);
  }

  @override
  bool shouldRepaint(PeintreDonjon old) => true;
}
```

**Résultat :**

```text
DONJON DE DART — plateformes et saut mural
vitesse     (-230, 120)
auSol       false
mur         -1
glisse      true
controlable true

Deux plateformes violettes vont et viennent horizontalement au-dessus d'un
sol de pierre. Le héros peut se poser dessus : il redevient bleu et se
laisse porter latéralement sans avoir à corriger sa position.
En sautant vers un bord de l'écran et en maintenant la direction contre le
mur, il devient orange et descend lentement, à 120 pixels par seconde au
lieu de 1100. Un appui sur « ^ » dans cet état le projette vers le côté
opposé, en cloche, et les commandes restent bloquées un dixième et demi
de seconde pour que l'élan ne soit pas annulé.
En alternant les deux murs, on remonte tout l'écran.
```

**Explication :** trois ajouts, trois pièges classiques évités.

**La plateforme n'est franchissable que par le dessus.** Le test combine trois conditions : le héros était **au-dessus** à la frame précédente (`yAvant + demiH <= p.haut + 1`), il est **en dessous** maintenant, et il **descend** (`vitesse.y >= 0`). Sans la première condition, il se téléporterait sur la plateforme en sautant par en dessous. C'est le principe des plateformes traversables, dites *one-way*, que le chapitre 24 formalisera.

**L'entraînement latéral se fait après le replacement.** `position += Vec2(p.vitesseX * dt, 0)` déplace le héros de la même quantité que la plateforme pendant cette frame. Sans cela, la plateforme glisserait sous ses pieds et il tomberait au bout de deux secondes. Notez qu'on ajoute un **déplacement**, pas une vitesse : le héros ne conserve pas l'élan de la plateforme quand il saute, ce qui est le comportement le plus lisible pour un joueur.

**La glissade murale plafonne la chute, elle ne l'annule pas.** `glisseMurMax = 120` remplace la vitesse terminale de 1100. Le héros descend encore, lentement. Un plafond de 0 le collerait au mur indéfiniment, ce qui casse la tension du niveau.

**Le blocage des commandes est ce qui rend le saut mural jouable.** Sans les 0,15 s de `controleBloqueRestant`, le joueur qui maintient la direction contre le mur annulerait immédiatement la composante horizontale de 280 px/s et retomberait sur place. C'est très exactement le mécanisme du knockback de 23.31, réutilisé dans un autre contexte — preuve que ces briques se recombinent.

**Le détail de `murTouche`.** On ne détecte le mur que si le joueur **pousse** contre lui (`axe < 0` à gauche, `axe > 0` à droite). Un héros qui frôle une paroi sans appuyer ne doit pas se mettre à glisser tout seul.

---

## Et maintenant ?

Votre héros possède désormais un corps. Il a une position, une vitesse, une accélération. Il tombe, il court, il saute, il glisse, il rebondit, il est repoussé. Vous savez régler ces comportements par intention et non par tâtonnement, et vous savez pourquoi l'ordre de deux lignes de code décide de la stabilité de toute une simulation.

Il lui manque pourtant l'essentiel : **le monde ne lui résiste pas encore**.

Dans tout ce chapitre, le « sol » n'était qu'un nombre, et les murs n'étaient que les bords de l'écran. Le gobelin ne fait mal que parce que deux rectangles se recouvrent, testés avec un `overlaps` que nous n'avons jamais expliqué. Il n'y a ni plafond, ni caisse, ni pic, ni porte.

Le chapitre 24 comble ce manque. Vous y apprendrez :

- ce qu'est une **hitbox** et pourquoi elle ne coïncide presque jamais avec le sprite ;
- le test **AABB**, la collision rectangle contre rectangle, écrite à la main ;
- la collision **cercle contre cercle**, où le `longueurCarree` de ce chapitre reprend du service ;
- comment **résoudre** une collision, c'est-à-dire décider où replacer l'entité et quelle composante de vitesse annuler — la généralisation directe de la section 23.18 ;
- pourquoi il faut séparer l'axe X de l'axe Y lors de la résolution, sous peine de personnages qui s'accrochent aux angles ;
- et le **tunneling** : comment un objet rapide traverse un mur mince en une seule frame, et pourquoi le plafonnement de `dt` du chapitre 20 ne suffit pas à l'éviter.

Tout ce que vous avez écrit ici sera réutilisé tel quel : `Vec2`, l'intégration semi-implicite, le `auSol`, le knockback. Le chapitre 24 ne remplace rien, il ajoute la couche qui manquait entre le mouvement et le monde.

Chapitre suivant : [24-PARTIE-2A—COLLISIONS-ET-HITBOXES.md](./24-PARTIE-2A—COLLISIONS-ET-HITBOXES.md)
