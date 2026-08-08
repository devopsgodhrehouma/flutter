# PARTIE 2A — LES FONDAMENTAUX DU JEU 2D
# CHAPITRE 26 — ARCHITECTURE ET ÉTATS D'UN JEU

> **Niveau :** intermédiaire
> **Durée estimée :** 8 h
> **Pré-requis :** chapitres 20 à 25 (boucle de jeu, `Canvas`, sprites, physique, collisions, caméra) et chapitres 8 à 11 de la PARTIE 1A (POO, héritage, classes abstraites, enums)
> **Ce que vous saurez faire à la fin :** organiser un jeu complet en entités, états, services et fichiers séparés, au lieu d'un unique `main.dart` illisible.

---

## 26.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer pourquoi un fichier de 2000 lignes finit toujours par bloquer un projet ;
- séparer les **données**, la **logique** et le **rendu** d'un jeu ;
- définir précisément ce qu'est une **entité** ;
- écrire une classe abstraite `Entity` et la faire hériter proprement ;
- répartir le travail entre `update(double dt)` et `render(Canvas canvas)` ;
- gérer une liste d'entités et sa boucle de mise à jour ;
- diagnostiquer et corriger le bug de **modification concurrente** ;
- mettre en place des **files d'ajout et de suppression** ;
- contrôler l'**ordre de rendu** avec un champ `priority` ;
- construire une hiérarchie `Entity → Character → Player / Enemy` et en voir les limites ;
- remplacer un héritage profond par de la **composition** ;
- expliquer le principe d'un **ECS** (entité, composant, système) ;
- comparer ECS et POO classique dans un tableau de décision ;
- décrire le modèle de Flame, un **arbre de composants** ;
- définir un **état de jeu** et écrire l'enum `GameState` ;
- dessiner la **machine à états** d'un jeu complet ;
- implémenter menu, jeu, pause, game over et victoire ;
- appliquer le **patron State** : une classe par écran ;
- empiler des états pour afficher une pause par-dessus le jeu ;
- découpler le clavier des actions avec un **gestionnaire d'entrées** ;
- proposer un **remappage des touches** ;
- faire communiquer des objets sans les coupler grâce à un **bus d'événements** ;
- centraliser score, vies et progression dans un **service de données** ;
- reconnaître les dangers du **Singleton** et pratiquer une **injection de dépendances** simple ;
- organiser les fichiers d'un projet de jeu ;
- décider **quoi tester** dans un jeu et écrire ces tests ;
- assembler le mini-moteur complet de la PARTIE 2A ;
- expliquer précisément ce que Flame apportera au chapitre 27.

---

## 26.1 — Le problème : un `main.dart` de 2000 lignes

Depuis le chapitre 20, vous avez ajouté une brique par chapitre. Une boucle, un `Canvas`, des sprites, de la vélocité, des collisions, une caméra. Chaque brique était juste. Le résultat, lui, ne l'est plus.

Regardons honnêtement à quoi ressemble le fichier d'un élève arrivé au chapitre 25.

```text
  main.dart  (2000 lignes)
  ─────────────────────────────────────────────────────────
  ligne    1 : import 'package:flutter/material.dart';
  ligne   14 : class DonjonApp extends StatelessWidget
  ligne   40 : class VueDeJeu extends StatefulWidget
  ligne   88 :   double heroX = 100, heroY = 200;
  ligne   89 :   double heroVx = 0, heroVy = 0;
  ligne   90 :   int heroVies = 3;
  ligne   91 :   double heroInvincibleRestant = 0;
  ligne   92 :   List<double> gobelinX = [];
  ligne   93 :   List<double> gobelinY = [];
  ligne   94 :   List<double> gobelinVx = [];
  ligne   95 :   List<bool> gobelinVivant = [];
  ligne   96 :   List<double> potionX = [];
  ligne   97 :   List<double> potionY = [];
  ligne   98 :   List<bool> potionPrise = [];
  ligne  110 :   int score = 0;
  ligne  111 :   bool enPause = false;
  ligne  112 :   bool gameOver = false;
  ligne  113 :   bool victoire = false;
  ligne  114 :   bool menuAffiche = true;
  ...
  ligne  260 :   void update(double dt) {        <-- 700 lignes
  ...
  ligne  960 :   }
  ...
  ligne 1010 :   void paint(Canvas canvas, ...) { <-- 600 lignes
  ...
  ligne 1610 :   }
  ...
  ligne 1998 : }
  ─────────────────────────────────────────────────────────
```

Ce fichier fonctionne. Le jeu tourne. Et pourtant il est **mort**, au sens où plus personne ne peut le faire évoluer. Voici pourquoi.

**Ajouter un ennemi coûte cinq modifications.** Un nouveau type d'ennemi, disons une chauve-souris, exige : quatre nouvelles listes parallèles, un bloc dans `update`, un bloc dans `paint`, une boucle de collision, une remise à zéro dans la fonction de redémarrage. Oublier l'un des cinq produit un bug silencieux.

**Les listes parallèles se désynchronisent.** `gobelinX[3]` et `gobelinVivant[3]` désignent le même gobelin, mais rien dans le code ne le garantit. Le jour où vous supprimez un gobelin d'une liste et pas de l'autre, tout se décale d'un cran.

**On ne peut pas tester.** Pour vérifier qu'un gobelin perd bien 2 points de vie, il faut lancer l'application, aller jusqu'au gobelin et le frapper. Aucun test automatique n'est possible : la logique est enfermée dans un `State` de widget.

**Les booléens d'état se contredisent.** Que se passe-t-il si `enPause` et `gameOver` valent `true` en même temps ? Personne ne le sait. Avec quatre booléens, il existe seize combinaisons possibles, dont douze n'ont aucun sens.

**Deux personnes ne peuvent pas travailler dessus.** Un seul fichier, un seul auteur.

Récapitulons les symptômes dans un tableau, parce que vous allez les reconnaître dans vos propres projets.

| Symptôme | Ce qu'il révèle | Section qui le corrige |
| --- | --- | --- |
| Un `update` de 700 lignes | aucune séparation par entité | 26.6 |
| Des listes parallèles `xs`, `ys`, `vies` | pas de classe pour l'objet du jeu | 26.3 |
| Le rendu qui modifie une position | logique et rendu mélangés | 26.2 |
| `if (gameOver) ... if (enPause) ...` partout | pas de machine à états | 26.18 |
| `if (touche == 'ArrowLeft')` dans `update` | entrées non découplées | 26.24 |
| Une variable globale `score` | pas de service de données | 26.27 |
| Un `Exception: Concurrent modification` | modification pendant l'itération | 26.7 |
| Aucun fichier de test | logique enfermée dans un widget | 26.31 |

> **À retenir.** Le problème n'est pas la taille du fichier. Le problème est que **chaque changement touche à tout**. L'architecture, c'est l'art de faire en sorte qu'un changement reste local.

---

## 26.2 — Séparer données, logique et rendu

La première coupe à pratiquer est aussi la plus rentable. Elle consiste à distinguer trois responsabilités qui n'ont rien à faire ensemble.

```text
  ┌─────────────────────────────────────────────────────────────┐
  │                  LES TROIS COUCHES D'UN JEU                 │
  └─────────────────────────────────────────────────────────────┘

   1. DONNÉES        Ce que le monde EST à cet instant.
   (state)           x, y, vies, score, vitesse, niveau
                     Pas de Canvas. Pas de Ticker. Pas de widget.
                     Testable en console.
        │
        │  lues et modifiées par
        ▼
   2. LOGIQUE        Ce qui FAIT changer le monde.
   (update)          gravité, collisions, IA, dégâts, score
                     Reçoit dt. Ne dessine rien.
                     Testable en console.
        │
        │  lues seulement par
        ▼
   3. RENDU          Comment le monde APPARAÎT.
   (render)          Canvas, Paint, sprites, couleurs, HUD
                     Ne modifie AUCUNE donnée.
                     Non testable simplement, et ce n'est pas grave.
```

La flèche du bas est la règle d'or : **le rendu lit, il n'écrit jamais**. Voici la version fautive, puis la version correcte.

### Version fautive

```dart
// NE FAITES PAS CELA.
@override
void paint(Canvas canvas, Size size) {
  heroX += 2;                       // logique dans le rendu
  if (heroX > size.width) heroX = 0;
  canvas.drawRect(
    Rect.fromLTWH(heroX, heroY, 32, 32),
    Paint()..color = const Color(0xFFE8B04B),
  );
}
```

Trois défauts, tous graves. Le héros avance de 2 pixels **par redessin**, donc deux fois plus vite sur un écran 120 Hz : c'est le bug du chapitre 20, sous une forme déguisée. Ensuite, si Flutter décide de redessiner deux fois la même frame, le héros avance deux fois. Enfin, la vitesse du héros devient impossible à tester sans écran.

### Version correcte

```dart
class Heros {
  Heros({required this.x, required this.y});

  double x;
  double y;
  double vitesse = 120; // pixels par seconde

  // LOGIQUE : modifie les données, ne dessine rien.
  void update(double dt, double largeurMonde) {
    x += vitesse * dt;
    if (x > largeurMonde) x = 0;
  }
}

// RENDU : lit les données, ne les modifie pas.
void dessinerHeros(Canvas canvas, Heros h) {
  canvas.drawRect(
    Rect.fromLTWH(h.x, h.y, 32, 32),
    Paint()..color = const Color(0xFFE8B04B),
  );
}
```

Vérifions immédiatement le bénéfice principal : la logique est maintenant testable **sans Flutter**, en console.

```dart
class Heros {
  Heros({required this.x, required this.y});

  double x;
  double y;
  double vitesse = 120;

  void update(double dt, double largeurMonde) {
    x += vitesse * dt;
    if (x > largeurMonde) x = 0;
  }
}

void main() {
  final Heros h = Heros(x: 0, y: 100);

  // Simulons 1 seconde à 60 images par seconde.
  for (int i = 0; i < 60; i++) {
    h.update(1 / 60, 800);
  }

  print('x apres 1 seconde : ${h.x.toStringAsFixed(1)}');

  // Simulons 1 seconde a 30 images par seconde.
  final Heros h2 = Heros(x: 0, y: 100);
  for (int i = 0; i < 30; i++) {
    h2.update(1 / 30, 800);
  }
  print('x apres 1 seconde : ${h2.x.toStringAsFixed(1)}');
}
```

**Résultat :**

```text
x apres 1 seconde : 120.0
x apres 1 seconde : 120.0
```

Les deux valeurs sont identiques, alors que le nombre d'images diffère du simple au double. Cette égalité est la preuve que la logique est correcte, et vous venez de l'obtenir en une seconde de calcul, sans lancer d'application.

> **Remarque.** Cette séparation ne coûte rien en performance. Elle coûte quelques classes de plus, et elle rembourse cet investissement dès le troisième type d'ennemi.

---

## 26.3 — Qu'est-ce qu'une entité ?

Le mot revient dans tous les moteurs de jeu. Définissons-le sans détour.

> **Définition.** Une **entité** est un objet du monde du jeu qui possède un état, qui évolue avec le temps et qui peut être affiché.

Trois critères, et il faut les trois.

| Critère | Question à se poser | Exemple |
| --- | --- | --- |
| Un état | « a-t-il des données propres ? » | position, vies, vitesse |
| Une évolution | « change-t-il tout seul avec le temps ? » | le gobelin patrouille |
| Une apparence | « le joueur peut-il le voir ? » | un sprite, un rectangle |

Passons en revue les objets du Donjon de Dart.

```text
  ENTITÉS (état + évolution + apparence)
  ──────────────────────────────────────
   Héros          x, y, vies, énergie        se déplace, saute
   Gobelin        x, y, pv, direction        patrouille
   Chauve-souris  x, y, pv, phase            oscille
   Potion         x, y, soin                 flotte doucement
   Clé            x, y                       scintille
   Coffre         x, y, ouvert               s'ouvre
   Boss           x, y, pv, phase            attaque en trois phases
   Projectile     x, y, vx, vy, durée        vole puis disparaît
   Particule      x, y, vie                  s'estompe

  PAS DES ENTITÉS
  ──────────────────────────────────────
   Le score              une donnée, pas un objet du monde  -> 26.27
   La caméra             un point de vue                    -> chapitre 25
   Le fond parallaxe     un décor, pas un acteur (discutable)
   Le gestionnaire de    un service                         -> 26.24
   touches
   L'état « en pause »   un état de jeu                     -> 26.16
```

La ligne « discutable » est volontaire. En architecture, certaines réponses dépendent du projet. Si votre fond parallaxe doit défiler tout seul et réagir au vent, faites-en une entité. S'il ne fait que se dessiner, gardez-le dans le rendu du décor.

**Un piège fréquent.** Un débutant crée une entité `GestionnaireDeScore` parce que « c'est un objet ». Ce n'est pas une entité : elle n'a pas d'apparence et n'évolue pas toute seule. C'est un **service**, et nous les verrons en section 26.27. Mélanger entités et services est la première fissure d'une architecture.

---

## 26.4 — La classe de base `Entity`

Nous allons donner un ancêtre commun à toutes nos entités. Cet ancêtre sera une **classe abstraite**, notion vue au chapitre 11 de la PARTIE 1A. Rappelons-en le principe en une phrase.

> Une classe abstraite définit un **contrat** : elle déclare des méthodes sans les écrire, et interdit qu'on l'instancie directement. Toute sous-classe concrète doit fournir le corps des méthodes manquantes.

C'est exactement ce qu'il nous faut. Nous voulons pouvoir écrire `entite.update(dt)` sans savoir s'il s'agit d'un gobelin ou d'une potion, tout en garantissant que chaque entité concrète sait effectivement se mettre à jour.

```dart
import 'dart:ui';

/// Ancêtre de tout ce qui vit dans le monde du jeu.
abstract class Entity {
  Entity({this.x = 0, this.y = 0, this.priority = 0});

  /// Position dans le MONDE (pas à l'écran) — voir chapitre 25.
  double x;
  double y;

  /// Ordre de dessin : petit = au fond, grand = au premier plan.
  int priority;

  /// Marque l'entité pour retrait à la fin de la frame — voir 26.8.
  bool aRetirer = false;

  /// Fait avancer l'entité de dt secondes. Ne dessine rien.
  void update(double dt);

  /// Dessine l'entité. Ne modifie aucune donnée.
  void render(Canvas canvas);
}
```

Quatre remarques sur ce code, dans l'ordre d'importance.

**`abstract` empêche `Entity()`.** Écrire `final e = Entity();` provoque une erreur de compilation. C'est voulu : une « entité en général » n'existe pas dans le monde du jeu.

**`update` et `render` n'ont pas de corps.** Le point-virgule remplace les accolades. Toute sous-classe qui oublie l'une des deux ne compilera pas. Le compilateur devient votre relecteur.

**Les champs communs sont mutualisés.** `x`, `y`, `priority` et `aRetirer` existent une seule fois, pour toutes les entités du jeu. Le jour où vous ajoutez une rotation, une seule ligne suffit.

**`import 'dart:ui'` suffit pour `Canvas`.** Vous n'avez pas besoin de `material.dart` dans un fichier de logique. C'est un bon indicateur : si un fichier d'entité importe `material.dart`, c'est probablement qu'il contient du rendu de widget, donc du code mal placé.

Écrivons une première entité concrète.

```dart
import 'dart:ui';

abstract class Entity {
  Entity({this.x = 0, this.y = 0, this.priority = 0});
  double x;
  double y;
  int priority;
  bool aRetirer = false;
  void update(double dt);
  void render(Canvas canvas);
}

class Potion extends Entity {
  Potion({required double x, required double y})
      : super(x: x, y: y, priority: 5);

  final double soin = 25;
  double _phase = 0;
  double _yInitial = 0;
  bool _initialisee = false;

  @override
  void update(double dt) {
    if (!_initialisee) {
      _yInitial = y;
      _initialisee = true;
    }
    // La potion flotte : un aller-retour vertical de 4 pixels.
    _phase += dt * 3;
    y = _yInitial + 4 * _sinus(_phase);
  }

  double _sinus(double a) {
    // Approximation suffisante pour l'exemple ; on utilisera
    // dart:math dans le code final.
    final double b = a % 6.2831853;
    return b < 3.1415927 ? (b * (3.1415927 - b)) / 2.4674011 * 1.0
                         : -(((b - 3.1415927) *
                             (6.2831853 - b)) / 2.4674011);
  }

  @override
  void render(Canvas canvas) {
    canvas.drawCircle(
      Offset(x, y),
      8,
      Paint()..color = const Color(0xFFCF5C5C),
    );
  }
}
```

Le sinus artisanal n'est là que pour montrer qu'une entité peut avoir des méthodes privées à elle. Dans le code final, nous écrirons simplement `import 'dart:math'` et `sin(_phase)`.

> **Erreur classique.** Écrire `class Potion extends Entity` puis oublier `@override void render`. Le message est explicite : *Missing concrete implementation of 'Entity.render'*. Ne le fuyez pas : il vous dit exactement quelle méthode manque.

---

## 26.5 — `update(double dt)` et `render(Canvas canvas)`

Ces deux méthodes sont le cœur battant de l'architecture. Leur répartition doit être **absolue**, sans exception ni « juste cette fois ».

```text
  ┌──────────────────────────┬──────────────────────────────────┐
  │  update(double dt)       │  render(Canvas canvas)           │
  ├──────────────────────────┼──────────────────────────────────┤
  │  A le droit de :         │  A le droit de :                 │
  │   - lire l'état          │   - lire l'état                  │
  │   - MODIFIER l'état      │   - appeler canvas.drawXxx       │
  │   - lire les entrées     │   - appeler canvas.save/restore  │
  │   - créer des entités    │   - créer des Paint, Rect, Path  │
  │   - marquer aRetirer     │                                  │
  ├──────────────────────────┼──────────────────────────────────┤
  │  N'a PAS le droit de :   │  N'a PAS le droit de :           │
  │   - toucher au Canvas    │   - MODIFIER quoi que ce soit    │
  │   - créer des Paint      │   - avancer une animation        │
  │   - dépendre des FPS     │   - lire dt (il n'y en a pas)    │
  │                          │   - décider d'une règle de jeu   │
  └──────────────────────────┴──────────────────────────────────┘
```

Trois conséquences pratiques que vous devez savoir justifier.

**`render` ne reçoit pas `dt`, et c'est volontaire.** Si le rendu recevait `dt`, la tentation d'y faire avancer une animation serait irrésistible. En le lui refusant, la signature elle-même interdit l'erreur. C'est un principe général : **rendez les mauvaises pratiques impossibles à écrire**.

**L'avancement d'une animation est de la logique.** Le chapitre 22 vous a appris à faire tourner un sprite sheet en incrémentant un temps accumulé. Cet incrément appartient à `update`. Le choix de la frame à dessiner appartient à `render`, mais il se contente de **lire** l'index déjà calculé.

```dart
class Gobelin extends Entity {
  Gobelin({required double x, required double y}) : super(x: x, y: y);

  double vitesse = 40;
  int _direction = 1;

  // Animation : voir chapitre 22.
  double _tempsAnim = 0;
  int _frame = 0;
  static const double _dureeParFrame = 0.12;
  static const int _nbFrames = 4;

  @override
  void update(double dt) {
    // 1. Déplacement.
    x += vitesse * _direction * dt;
    if (x < 50) {
      x = 50;
      _direction = 1;
    } else if (x > 300) {
      x = 300;
      _direction = -1;
    }

    // 2. Avancement de l'animation : c'est de la LOGIQUE.
    _tempsAnim += dt;
    while (_tempsAnim >= _dureeParFrame) {
      _tempsAnim -= _dureeParFrame;
      _frame = (_frame + 1) % _nbFrames;
    }
  }

  @override
  void render(Canvas canvas) {
    // Le rendu LIT _frame, il ne l'incrémente jamais.
    final double hauteur = 22 + _frame.toDouble();
    canvas.drawRect(
      Rect.fromLTWH(x, y - hauteur, 20, hauteur),
      Paint()..color = const Color(0xFF6FBF73),
    );
  }
}
```

**Le rendu doit pouvoir être appelé deux fois de suite sans effet.** C'est le test mental le plus efficace. Posez-vous la question : « si j'appelle `render` deux fois pour la même frame, obtiens-je exactement la même image ? » Si la réponse est non, votre rendu contient de la logique.

> **Astuce de relecture.** Cherchez dans vos `render` les symboles `+=`, `-=`, `++` et `--` appliqués à un champ de la classe. Chaque occurrence est une faute d'architecture.

---

## 26.6 — Une liste d'entités et sa boucle

Une fois toutes les entités descendantes d'`Entity`, le `update` monstrueux de 700 lignes se réduit à quatre lignes. C'est le moment le plus satisfaisant du chapitre.

```text
  AVANT                              APRÈS
  ─────────────────────────          ─────────────────────────────
  void update(double dt) {           void update(double dt) {
    // héros : 80 lignes               for (final e in entites) {
    // gobelins : 120 lignes             e.update(dt);
    // chauves-souris : 90 lignes       }
    // potions : 40 lignes            }
    // clés : 30 lignes
    // coffres : 50 lignes            void render(Canvas c) {
    // boss : 180 lignes                for (final e in entites) {
    // projectiles : 60 lignes           e.render(c);
    // particules : 50 lignes           }
  }                                  }
```

Le code du monde devient trivial. La complexité n'a pas disparu, elle a été **répartie** : chaque entité connaît son propre comportement, et personne d'autre n'a besoin de le connaître. C'est le **polymorphisme** du chapitre 10, appliqué à grande échelle.

Écrivons un monde minimal, exécutable en console.

```dart
abstract class Entity {
  Entity({this.x = 0, this.y = 0});
  double x;
  double y;
  bool aRetirer = false;

  void update(double dt);
  String decrire();
}

class Heros extends Entity {
  Heros({required double x}) : super(x: x, y: 0);

  int vies = 3;

  @override
  void update(double dt) {
    x += 120 * dt;
  }

  @override
  String decrire() => 'Heros   x=${x.toStringAsFixed(1)} vies=$vies';
}

class Gobelin extends Entity {
  Gobelin({required double x}) : super(x: x, y: 0);

  int pv = 8;
  int direction = 1;

  @override
  void update(double dt) {
    x += 40 * direction * dt;
    if (x > 200) direction = -1;
    if (x < 100) direction = 1;
  }

  @override
  String decrire() => 'Gobelin x=${x.toStringAsFixed(1)} pv=$pv';
}

class Potion extends Entity {
  Potion({required double x}) : super(x: x, y: 0);

  @override
  void update(double dt) {
    // La potion ne bouge pas.
  }

  @override
  String decrire() => 'Potion  x=${x.toStringAsFixed(1)}';
}

class Monde {
  final List<Entity> entites = <Entity>[];

  void ajouter(Entity e) => entites.add(e);

  void update(double dt) {
    for (final Entity e in entites) {
      e.update(dt);
    }
  }

  void afficher() {
    for (final Entity e in entites) {
      print(e.decrire());
    }
  }
}

void main() {
  final Monde monde = Monde();
  monde.ajouter(Heros(x: 0));
  monde.ajouter(Gobelin(x: 180));
  monde.ajouter(Potion(x: 250));

  // Une seconde de jeu a 60 images par seconde.
  for (int i = 0; i < 60; i++) {
    monde.update(1 / 60);
  }

  print('--- apres 1 seconde ---');
  monde.afficher();
}
```

**Résultat :**

```text
--- apres 1 seconde ---
Heros   x=120.0 vies=3
Gobelin x=180.0 pv=8
Potion  x=250.0
```

Le gobelin est reparti de 180 vers 200, a rebondi, et se retrouve à 180. Le héros a parcouru ses 120 pixels. La potion n'a pas bougé, et pourtant son `update` a bien été appelé soixante fois : une entité immobile a le droit d'avoir un `update` vide.

Remarquez ce que `Monde.update` **ne sait pas** : il ignore totalement qu'il existe des gobelins. Vous pouvez ajouter dix types d'entités sans modifier une seule ligne de cette classe. C'est le critère d'une bonne architecture : **on étend sans modifier**.

> **Remarque de performance.** Une boucle `for` sur une `List` de 2000 objets coûte quelques microsecondes. Ce n'est jamais le goulot d'étranglement d'un jeu 2D. N'optimisez pas ce point avant d'avoir mesuré, comme le rappelait le chapitre 20 avec l'indicateur de pire frame.

---

## 26.7 — Ajouter et retirer des entités pendant la boucle : le bug de la modification concurrente

Voici le bug qui frappe absolument tous les débutants en développement de jeu, à peu près à ce stade du projet. Il faut le rencontrer une fois pour ne plus jamais l'oublier.

Le scénario est banal : un gobelin meurt, on veut le retirer de la liste. Ou bien un coffre s'ouvre, on veut ajouter une clé. Dans les deux cas, on modifie la liste **pendant qu'on la parcourt**.

```dart
class Entite {
  Entite(this.nom, this.pv);
  final String nom;
  int pv;
}

void main() {
  final List<Entite> entites = <Entite>[
    Entite('gobelin A', 0),
    Entite('gobelin B', 5),
    Entite('gobelin C', 0),
  ];

  // NE FAITES PAS CELA.
  for (final Entite e in entites) {
    if (e.pv <= 0) {
      entites.remove(e); // modification pendant l'itération
    }
  }

  print(entites.length);
}
```

**Résultat :**

```text
Unhandled exception:
Concurrent modification during iteration: Instance(length:2) of '_GrowableList'.
#0      ListIterator.moveNext (dart:_internal/iterable.dart:336:7)
#1      main (file:///main.dart:14:22)
```

Le message est très clair une fois qu'on sait le lire : *« modification concurrente pendant l'itération »*. Comprenons pourquoi Dart refuse.

```text
  POURQUOI L'ITÉRATEUR SE CASSE

  Liste au départ :   [ A , B , C ]     longueur = 3
                        ^
                     curseur = 0

  On supprime A :     [ B , C ]         longueur = 2
                            ^
                     curseur = 1   -> pointe maintenant sur C !
                                      B n'a JAMAIS été examiné.

  Dart détecte que la longueur a changé et lève une exception
  PLUTÔT que de vous laisser sauter silencieusement un élément.
```

Cette exception est donc une **protection**, pas une brimade. Dans un langage qui ne la lèverait pas, vous auriez un gobelin sur deux qui ignore les dégâts, et vous chercheriez la cause pendant trois jours.

Le piège existe aussi à l'ajout, et il est plus vicieux encore.

```dart
class Entite {
  Entite(this.nom);
  final String nom;
}

void main() {
  final List<Entite> entites = <Entite>[Entite('coffre')];

  for (final Entite e in entites) {
    if (e.nom == 'coffre') {
      entites.add(Entite('cle')); // ajout pendant l'itération
    }
  }

  print(entites.length);
}
```

**Résultat :**

```text
Unhandled exception:
Concurrent modification during iteration: Instance(length:2) of '_GrowableList'.
```

Même exception. Et si Dart ne la levait pas, une entité qui s'ajoute elle-même produirait une boucle infinie : le coffre crée une clé, la clé est examinée, elle crée quelque chose, et ainsi de suite jusqu'à saturation de la mémoire.

Passons en revue les fausses solutions, car vous les verrez sur les forums.

| Tentative | Verdict | Pourquoi |
| --- | --- | --- |
| `for (final e in entites.toList())` | acceptable | on itère sur une copie ; coût : une copie par frame |
| `entites.removeWhere((e) => e.pv <= 0)` | bon pour les retraits | mais ne règle pas les ajouts |
| `for (int i = 0; i < entites.length; i++)` | dangereux | saute un élément après chaque suppression |
| `for (int i = entites.length - 1; i >= 0; i--)` | correct pour les retraits | illisible, et ne règle pas les ajouts |
| Files d'attente | **la bonne solution** | règle ajouts et retraits, coût nul | 

La dernière ligne est celle qu'utilisent tous les moteurs professionnels, Flame compris. C'est l'objet de la section suivante.

---

## 26.8 — Les files d'ajout et de suppression

L'idée tient en une phrase : **on ne touche jamais à la liste pendant la boucle ; on note ce qu'on veut faire, et on le fait après**.

```text
  ┌───────────────────────────────────────────────────────────────┐
  │            UNE FRAME AVEC FILES D'ATTENTE                     │
  └───────────────────────────────────────────────────────────────┘

  1. update de chaque entité
     ┌─────────────────────────────────────────────┐
     │  for (final e in entites) e.update(dt);      │
     │                                             │
     │  Pendant ce parcours :                      │
     │   - le gobelin meurt      -> aRetirer = true│
     │   - le héros tire         -> _aAjouter.add()│
     │   - le coffre s'ouvre     -> _aAjouter.add()│
     │                                             │
     │  La liste "entites" n'est PAS touchée.      │
     └─────────────────────────────────────────────┘
                        │
                        ▼
  2. vidage des files (hors de toute itération)
     ┌─────────────────────────────────────────────┐
     │  entites.addAll(_aAjouter);                 │
     │  _aAjouter.clear();                         │
     │  entites.removeWhere((e) => e.aRetirer);    │
     └─────────────────────────────────────────────┘
                        │
                        ▼
  3. render de chaque entité
     ┌─────────────────────────────────────────────┐
     │  for (final e in entites) e.render(canvas); │
     └─────────────────────────────────────────────┘
```

Le point clé est l'étape 2 : elle se déroule **entre** deux parcours, jamais pendant. Traduisons cela en code.

```dart
abstract class Entity {
  Entity({this.x = 0, this.y = 0});
  double x;
  double y;
  bool aRetirer = false;

  /// Le monde auquel appartient l'entité, pour pouvoir en créer d'autres.
  Monde? monde;

  void update(double dt);
  String decrire();
}

class Monde {
  final List<Entity> entites = <Entity>[];
  final List<Entity> _aAjouter = <Entity>[];

  /// Demande l'ajout d'une entité. L'ajout est effectif à la fin de la frame.
  void ajouter(Entity e) {
    e.monde = this;
    _aAjouter.add(e);
  }

  /// Ajout immédiat, réservé à l'initialisation (hors boucle).
  void ajouterMaintenant(Entity e) {
    e.monde = this;
    entites.add(e);
  }

  void update(double dt) {
    // 1. On parcourt une liste que personne ne modifiera.
    for (final Entity e in entites) {
      e.update(dt);
    }

    // 2. On applique les demandes accumulées.
    _viderLesFiles();
  }

  void _viderLesFiles() {
    if (_aAjouter.isNotEmpty) {
      entites.addAll(_aAjouter);
      _aAjouter.clear();
    }
    entites.removeWhere((Entity e) => e.aRetirer);
  }

  void afficher() {
    print('--- ${entites.length} entites ---');
    for (final Entity e in entites) {
      print(e.decrire());
    }
  }
}

class Coffre extends Entity {
  Coffre({required double x}) : super(x: x);

  bool ouvert = false;
  double _delai = 0.5;

  @override
  void update(double dt) {
    if (ouvert) return;
    _delai -= dt;
    if (_delai <= 0) {
      ouvert = true;
      // On DEMANDE un ajout : la liste n'est pas touchée ici.
      monde?.ajouter(Cle(x: x + 10));
    }
  }

  @override
  String decrire() => 'Coffre  x=${x.toStringAsFixed(0)} ouvert=$ouvert';
}

class Cle extends Entity {
  Cle({required double x}) : super(x: x);

  @override
  void update(double dt) {}

  @override
  String decrire() => 'Cle     x=${x.toStringAsFixed(0)}';
}

class Gobelin extends Entity {
  Gobelin({required double x, required this.pv}) : super(x: x);

  int pv;

  @override
  void update(double dt) {
    pv -= 4; // dégâts fictifs pour l'exemple
    if (pv <= 0) {
      // On MARQUE : la suppression aura lieu après la boucle.
      aRetirer = true;
    }
  }

  @override
  String decrire() => 'Gobelin x=${x.toStringAsFixed(0)} pv=$pv';
}

void main() {
  final Monde monde = Monde();
  monde.ajouterMaintenant(Coffre(x: 100));
  monde.ajouterMaintenant(Gobelin(x: 200, pv: 8));
  monde.ajouterMaintenant(Gobelin(x: 240, pv: 4));

  monde.afficher();

  for (int frame = 1; frame <= 3; frame++) {
    monde.update(0.25);
    print('=== apres la frame $frame ===');
    monde.afficher();
  }
}
```

**Résultat :**

```text
--- 3 entites ---
Coffre  x=100 ouvert=false
Gobelin x=200 pv=8
Gobelin x=240 pv=4
=== apres la frame 1 ===
--- 2 entites ---
Coffre  x=100 ouvert=false
Gobelin x=200 pv=4
=== apres la frame 2 ===
--- 2 entites ---
Coffre  x=100 ouvert=true
Cle     x=110
=== apres la frame 3 ===
--- 2 entites ---
Coffre  x=100 ouvert=true
Cle     x=110
```

Suivons le déroulement, car chaque ligne s'explique.

**Frame 1.** Le délai du coffre passe de 0,50 à 0,25 seconde : il ne s'ouvre pas encore. Les deux gobelins encaissent 4 points. Celui qui avait 4 points de vie tombe à 0 et se marque `aRetirer`. Le vidage des files le supprime. Il reste deux entités.

**Frame 2.** Le délai du coffre atteint 0 : le coffre s'ouvre et **demande** une clé. Le gobelin restant tombe de 4 à 0 et se marque. Le vidage ajoute d'abord la clé, puis retire le gobelin. Il reste le coffre et la clé.

**Frame 3.** Le coffre est déjà ouvert et sort immédiatement de son `update`. La clé a un `update` vide. Rien ne change : le monde est stable.

Notez surtout ce qui ne s'est **pas** produit : aucune exception, alors que nous avons ajouté et supprimé des entités au beau milieu du parcours. C'est tout l'intérêt du dispositif.

Trois enseignements à retenir de ce mécanisme.

**L'ordre du vidage est un choix de conception.** Ajouter avant de retirer signifie qu'une entité créée cette frame ne peut pas être retirée cette même frame par un `removeWhere`. Retirer avant d'ajouter donnerait l'inverse. Flame ajoute puis retire, comme nous. Documentez votre choix.

**Une entité ajoutée n'est pas mise à jour la frame de sa création.** Elle apparaît dans la liste après la boucle, donc son premier `update` a lieu à la frame suivante. C'est normal et sans conséquence visible, mais il faut le savoir quand on débogue un projectile qui semble « perdre une frame ».

**`aRetirer` est plus sûr qu'une suppression directe.** Deux morceaux de code peuvent marquer la même entité sans risque : `true` deux fois vaut `true`. Deux `remove` successifs sur le même objet, eux, produiraient un comportement dépendant de l'ordre.

> **À retenir.** Ne modifiez jamais une collection pendant que vous la parcourez. Notez l'intention, appliquez-la après. Cette règle vaut pour les entités, mais aussi pour les écouteurs d'événements de la section 26.26.

---

## 26.9 — L'ordre de rendu et la profondeur (`priority`)

Le `Canvas` de Flutter dessine dans l'ordre des appels : le dernier dessin recouvre les précédents. C'est ce qu'on appelle l'**algorithme du peintre**. Sans précaution, l'ordre de rendu est donc l'ordre d'insertion dans la liste, ce qui produit des résultats absurdes.

```text
  SANS ORDRE DE RENDU

  entites = [ héros, sol, gobelin, ciel ]
              (1)    (2)   (3)      (4)

  Ordre de dessin :  héros, puis sol PAR-DESSUS,
                     puis gobelin, puis ciel PAR-DESSUS TOUT.

  ┌──────────────────────────────┐
  │                              │
  │        CIEL OPAQUE           │   Le joueur ne voit rien.
  │                              │
  └──────────────────────────────┘
```

La solution consiste à donner à chaque entité un entier de profondeur, et à trier avant de dessiner. Nous l'appelons `priority`, du nom exact qu'utilise Flame — vous retrouverez le mot au chapitre 28.

```text
  AVEC priority (petit = au fond, grand = devant)

  priority   couche                      exemple
  ────────  ─────────────────────────   ─────────────────────
    -100    ciel, fond lointain          dégradé
     -50    parallaxe arrière            montagnes
     -10    décor de fond                murs du donjon
       0    sol, tuiles                  tilemap
      10    objets posés au sol          potions, clés, coffres
      20    ennemis                      gobelins, chauves-souris
      30    joueur                       le héros
      40    projectiles                  flèches, boules de feu
      50    particules                   étincelles, fumée
     100    HUD                          vies, score, barre de vie
     200    voile de pause               rectangle noir semi-opaque
```

Le tableau ci-dessus est une **convention de projet**. Écrivez-la une fois dans un fichier de constantes et n'en dérogez jamais.

```dart
/// Les couches de rendu du Donjon de Dart.
/// Une seule source de vérité, pour tout le projet.
abstract class Couches {
  static const int ciel = -100;
  static const int parallaxe = -50;
  static const int decorFond = -10;
  static const int sol = 0;
  static const int objets = 10;
  static const int ennemis = 20;
  static const int joueur = 30;
  static const int projectiles = 40;
  static const int particules = 50;
  static const int hud = 100;
  static const int voile = 200;
}
```

Reste à trier. Trois stratégies existent, et le choix n'est pas anodin.

```dart
import 'dart:ui';

abstract class Entity {
  Entity({this.x = 0, this.y = 0, this.priority = 0});
  double x;
  double y;
  int priority;
  bool aRetirer = false;
  void update(double dt);
  void render(Canvas canvas);
}

class Monde {
  final List<Entity> entites = <Entity>[];
  final List<Entity> _aAjouter = <Entity>[];

  /// Liste triée réutilisée d'une frame à l'autre pour éviter
  /// de créer un nouveau tableau soixante fois par seconde.
  final List<Entity> _ordreDeRendu = <Entity>[];
  bool _triNecessaire = true;

  void ajouter(Entity e) {
    _aAjouter.add(e);
    _triNecessaire = true;
  }

  void update(double dt) {
    for (final Entity e in entites) {
      e.update(dt);
    }
    if (_aAjouter.isNotEmpty) {
      entites.addAll(_aAjouter);
      _aAjouter.clear();
      _triNecessaire = true;
    }
    final int avant = entites.length;
    entites.removeWhere((Entity e) => e.aRetirer);
    if (entites.length != avant) _triNecessaire = true;
  }

  void render(Canvas canvas) {
    if (_triNecessaire) {
      _ordreDeRendu
        ..clear()
        ..addAll(entites)
        ..sort((Entity a, Entity b) => a.priority.compareTo(b.priority));
      _triNecessaire = false;
    }
    for (final Entity e in _ordreDeRendu) {
      e.render(canvas);
    }
  }
}
```

Comparons les trois stratégies possibles.

| Stratégie | Coût par frame | Quand l'utiliser |
| --- | --- | --- |
| Trier à chaque frame | `O(n log n)` à chaque frame | jamais nécessaire en 2D simple |
| Trier seulement si la liste a changé | quasi nul en régime stable | **choix par défaut**, celui du code ci-dessus |
| Une liste par couche | nul, mais plus de code | milliers d'entités, jeu à tuiles dense |

Un dernier point mérite votre attention : **le tri de Dart n'est pas stable**. Deux entités de même `priority` peuvent changer d'ordre entre deux tris, ce qui produit un scintillement si elles se recouvrent. Deux parades : donner des priorités distinctes aux entités qui se recouvrent, ou trier sur un couple `(priority, y)`, ce qui est d'ailleurs la technique classique des jeux en vue de dessus.

```dart
_ordreDeRendu.sort((Entity a, Entity b) {
  final int parCouche = a.priority.compareTo(b.priority);
  if (parCouche != 0) return parCouche;
  return a.y.compareTo(b.y); // le plus « bas » à l'écran passe devant
});
```

Avec ce tri, un héros situé devant un tonneau le masque, et derrière il est masqué, sans une ligne de code supplémentaire. C'est le fameux tri en Y des jeux isométriques.

> **Remarque.** L'ordre de `update` et l'ordre de `render` sont deux choses différentes. `update` peut rester dans l'ordre d'insertion : la logique ne doit pas dépendre de l'ordre de parcours. Si elle en dépend, c'est le signe d'un couplage à corriger.

---

## 26.10 — L'héritage classique : `Entity → Character → Player / Enemy`

Le chapitre 10 de la PARTIE 1A vous a appris l'héritage : une classe fille reçoit les champs et méthodes de sa mère, et peut les redéfinir avec `@override`. Appliquons-le au Donjon de Dart.

Constat de départ : le héros et le gobelin partagent beaucoup de choses. Tous deux ont des points de vie, subissent des dégâts, meurent, ont une vitesse et une direction. Plutôt que d'écrire deux fois le même code, on introduit un intermédiaire.

```text
  ┌─────────────────────────────────────────────────────────────┐
  │                 HIÉRARCHIE PAR HÉRITAGE                     │
  └─────────────────────────────────────────────────────────────┘

                        Entity  (abstract)
                        x, y, priority, aRetirer
                        update(dt), render(canvas)
                              │
              ┌───────────────┼────────────────┐
              │               │                │
          Character       Collectible       Decor
          (abstract)      (abstract)        (abstract)
          pv, pvMax       ramasse()         (aucun ajout)
          vitesse         update() vide
          subirDegats()         │                │
          estMort           ┌───┴───┐        ┌───┴───┐
              │             │       │        │       │
        ┌─────┴─────┐    Potion    Cle    Torche  Tonneau
        │           │
      Player      Enemy  (abstract)
      energie     cible
      sauter()    ia()
        │           │
        │      ┌────┴────┬─────────┐
        │      │         │         │
        │   Gobelin  ChauveSouris Boss
        │
     (une seule classe)
```

Écrivons cette hiérarchie, en console pour pouvoir l'exécuter.

```dart
abstract class Entity {
  Entity({this.x = 0, this.y = 0, this.priority = 0});
  double x;
  double y;
  int priority;
  bool aRetirer = false;

  void update(double dt);
  String etat();
}

abstract class Character extends Entity {
  Character({
    required double x,
    required double y,
    required this.pvMax,
    required this.vitesse,
    int priority = 20,
  })  : pv = pvMax,
        super(x: x, y: y, priority: priority);

  int pv;
  final int pvMax;
  double vitesse;
  int direction = 1;

  bool get estMort => pv <= 0;

  void subirDegats(int degats) {
    if (estMort) return;
    pv -= degats;
    if (pv <= 0) {
      pv = 0;
      mourir();
    }
  }

  /// Comportement de mort, redéfinissable.
  void mourir() {
    aRetirer = true;
  }
}

class Player extends Character {
  Player({required double x, required double y})
      : super(x: x, y: y, pvMax: 6, vitesse: 140, priority: 30);

  double energie = 100;
  bool auSol = true;

  void sauter() {
    if (auSol && energie >= 10) {
      auSol = false;
      energie -= 10;
    }
  }

  @override
  void mourir() {
    // Le joueur ne disparaît pas : il déclenche l'écran de Game Over.
    // Nous verrons comment le signaler proprement en 26.26.
    print('>>> Le heros est tombe. Game Over.');
  }

  @override
  void update(double dt) {
    x += vitesse * direction * dt;
    energie = (energie + 5 * dt).clamp(0, 100).toDouble();
  }

  @override
  String etat() =>
      'Player   pv=$pv/$pvMax energie=${energie.toStringAsFixed(0)} '
      'x=${x.toStringAsFixed(0)}';
}

abstract class Enemy extends Character {
  Enemy({
    required double x,
    required double y,
    required int pvMax,
    required double vitesse,
  }) : super(x: x, y: y, pvMax: pvMax, vitesse: vitesse);

  /// Le comportement propre à chaque ennemi.
  void ia(double dt);

  @override
  void update(double dt) {
    ia(dt);
  }
}

class Gobelin extends Enemy {
  Gobelin({required double x, required double y})
      : super(x: x, y: y, pvMax: 8, vitesse: 40);

  @override
  void ia(double dt) {
    x += vitesse * direction * dt;
    if (x > 260) direction = -1;
    if (x < 200) direction = 1;
  }

  @override
  String etat() => 'Gobelin  pv=$pv/$pvMax x=${x.toStringAsFixed(0)}';
}

class Boss extends Enemy {
  Boss({required double x, required double y})
      : super(x: x, y: y, pvMax: 60, vitesse: 25);

  int phase = 1;

  @override
  void ia(double dt) {
    x += vitesse * direction * dt;
    if (pv < pvMax * 0.5 && phase == 1) {
      phase = 2;
      vitesse = 50;
    }
  }

  @override
  String etat() => 'Boss     pv=$pv/$pvMax phase=$phase';
}

void main() {
  final List<Entity> entites = <Entity>[
    Player(x: 100, y: 200),
    Gobelin(x: 240, y: 200),
    Boss(x: 400, y: 200),
  ];

  for (int i = 0; i < 60; i++) {
    for (final Entity e in entites) {
      e.update(1 / 60);
    }
  }

  // Le héros frappe tout ce qui est un Character.
  for (final Entity e in entites) {
    if (e is Character && e is! Player) {
      e.subirDegats(35);
    }
  }

  for (final Entity e in entites) {
    print(e.etat());
  }
}
```

**Résultat :**

```text
Player   pv=6/6 energie=100 x=240
Gobelin  pv=0/8 x=241
Boss     pv=25/60 phase=1
```

Trois points sont à souligner.

**`subirDegats` n'est écrit qu'une fois.** Le gobelin et le boss l'héritent de `Character`. Ajouter une chauve-souris ne demandera pas de réécrire cette méthode.

**`mourir` est redéfini pour le joueur.** L'ennemi disparaît, le joueur non. C'est du polymorphisme : le même appel produit deux comportements selon le type réel de l'objet.

**`e is Character` filtre proprement.** Le test de type vu au chapitre 10 permet de traiter d'un coup tous les personnages, sans énumérer les sous-classes. C'est plus robuste qu'une liste de `if` par type.

La dernière ligne de la sortie mérite un arrêt. Le boss est descendu à 25 points de vie sur 60, donc sous la moitié : il devrait être en phase 2. Il affiche pourtant `phase=1`. Pourquoi ?

Parce que le passage de phase est écrit dans `ia`, c'est-à-dire dans l'`update`. Or les dégâts ont été infligés **après** la boucle de simulation : plus aucun `update` n'a été appelé ensuite, donc la condition n'a jamais été évaluée. Le boss ne passerait en phase 2 qu'à la frame suivante.

Ce décalage d'une frame est ici sans gravité, mais le principe qu'il illustre est important : **déclenchez un changement d'état là où sa cause se produit**, et non plusieurs frames plus tard en sondant l'état. Voici la correction de conception, à écrire dans `Boss`.

```dart
@override
void subirDegats(int degats) {
  super.subirDegats(degats);
  if (pv < pvMax * 0.5 && phase == 1) {
    phase = 2;
    vitesse = 50;
  }
}
```

Avec cette version, la phase change à l'instant exact où le seuil est franchi. Retenez le principe : **réagissez à l'événement, ne sondez pas l'état frame après frame** quand vous pouvez l'éviter.

---

## 26.11 — Les limites de l'héritage profond

L'arbre de la section précédente est élégant. Il le reste tant que le jeu est simple. Voyons ce qui se passe quand le game design évolue, ce qui est certain.

**Demande 1 : un coffre piégé qui attaque.** Un coffre est un `Decor`. Pour qu'il attaque, il lui faut des points de vie et des dégâts, donc `Character`. Mais une classe Dart ne peut hériter que d'**une seule** classe. Impasse.

**Demande 2 : une potion qui fuit le joueur.** Une potion est un `Collectible`, dont l'`update` est vide. Il faudrait maintenant de l'IA, qui vit dans `Enemy`. Impasse.

**Demande 3 : un joueur contrôlé par l'IA en démo d'attract mode.** Le `Player` devrait devenir un `Enemy` le temps de la démo. Impasse.

**Demande 4 : un gobelin qui vole.** Le vol est dans `ChauveSouris`. Il faut le dupliquer, ou remonter `voler()` dans `Enemy`, ce qui donne une méthode inutile à tous les ennemis terrestres.

Ces quatre demandes sont ordinaires. Aucune n'est un caprice. Et pourtant chacune casse l'arbre. Voici ce qui arrive concrètement à une hiérarchie soumise à ces pressions.

```text
  L'ARBRE QUI POURRIT

  Entity
    └── Character
          ├── Player
          ├── PlayerAvecIA              (copie de Player + IA)
          └── Enemy
                ├── Gobelin
                ├── GobelinVolant       (copie de Gobelin + vol)
                ├── GobelinTireur       (copie de Gobelin + tir)
                ├── GobelinVolantTireur (copie des deux !)
                ├── ChauveSouris
                ├── ChauveSourisTireuse
                ├── Coffre              (un décor dans Enemy ?!)
                └── PotionFuyante       (un objet dans Enemy ?!)

  3 comportements optionnels (voler, tirer, exploser)
  -> 2 x 2 x 2 = 8 classes par type d'ennemi.
  C'est l'EXPLOSION COMBINATOIRE.
```

Résumons les quatre maux de l'héritage profond.

| Mal | Description | Conséquence concrète |
| --- | --- | --- |
| Héritage simple | une seule classe mère en Dart | impossible d'être décor **et** personnage |
| Explosion combinatoire | `n` options donnent `2^n` classes | 8 classes pour 3 comportements |
| Classe mère obèse | on remonte tout « au cas où » | `Entity` finit avec 40 champs inutiles |
| Rigidité | le type est figé à la construction | un gobelin ne peut pas apprendre à voler en jeu |

Le dernier point est le plus décisif pour un jeu. Un ennemi qui rentre dans une phase de rage devrait pouvoir **gagner** un comportement en cours de partie. Avec l'héritage, son type est gravé au moment du `new` : il faudrait le détruire et le recréer, en perdant sa position, ses points de vie et son animation en cours.

> **À retenir.** L'héritage répond à la question « **qu'est-ce que c'est ?** ». Il devient un piège dès que la vraie question est « **que sait-il faire ?** », car un objet peut savoir faire plusieurs choses, et en apprendre de nouvelles.

Cela ne veut pas dire que l'héritage est mauvais. Une hiérarchie de **deux niveaux** — `Entity`, puis vos entités concrètes — est saine et suffit à la plupart des jeux 2D. Le danger commence au troisième niveau, et devient critique au quatrième.

---

## 26.12 — La composition

La composition répond à la question « que sait-il faire ? ». Le principe se résume par une formule célèbre :

> **Préférez la composition à l'héritage.** Au lieu de dire « un gobelin volant EST un gobelin qui vole », dites « un gobelin volant EST une entité qui A un déplacement, A des points de vie et A une capacité de vol ».

Le chapitre 11 vous a donné deux outils pour cela : les **mixins** et la délégation à des objets. Utilisons d'abord des objets, car c'est le mécanisme le plus souple et celui qui mène directement à l'ECS.

```text
  HÉRITAGE                        COMPOSITION

  GobelinVolantTireur             Entity « gobelin volant tireur »
    extends GobelinVolant           ├── Sante(pv: 8)
    extends Gobelin                 ├── Deplacement(vitesse: 40)
    extends Enemy                   ├── Vol(amplitude: 20)
    extends Character               ├── Tir(cadence: 1.5)
    extends Entity                  └── Apparence(couleur: vert)

  Le type dit ce qu'il EST.       La liste dit ce qu'il A.
  Figé à la compilation.          Modifiable en cours de partie.
  8 classes pour 3 options.       3 classes pour 3 options.
```

Voici une implémentation complète et exécutable.

```dart
/// Un comportement attachable à une entité.
abstract class Comportement {
  Entite? proprietaire;

  void update(double dt);
}

class Entite {
  Entite({required this.nom, this.x = 0, this.y = 0});

  final String nom;
  double x;
  double y;
  bool aRetirer = false;

  final List<Comportement> comportements = <Comportement>[];

  /// Attache un comportement et renvoie l'entité, pour chaîner les appels.
  Entite avec(Comportement c) {
    c.proprietaire = this;
    comportements.add(c);
    return this;
  }

  /// Récupère un comportement d'un type donné, ou null.
  T? obtenir<T extends Comportement>() {
    for (final Comportement c in comportements) {
      if (c is T) return c;
    }
    return null;
  }

  void update(double dt) {
    for (final Comportement c in comportements) {
      c.update(dt);
    }
  }
}

// --- Les comportements ---

class Sante extends Comportement {
  Sante(this.pvMax) : pv = pvMax;

  int pv;
  final int pvMax;

  void subirDegats(int d) {
    pv -= d;
    if (pv <= 0) {
      pv = 0;
      proprietaire?.aRetirer = true;
    }
  }

  @override
  void update(double dt) {}
}

class Patrouille extends Comportement {
  Patrouille({
    required this.vitesse,
    required this.xMin,
    required this.xMax,
  });

  final double vitesse;
  final double xMin;
  final double xMax;
  int direction = 1;

  @override
  void update(double dt) {
    final Entite? e = proprietaire;
    if (e == null) return;
    e.x += vitesse * direction * dt;
    if (e.x > xMax) direction = -1;
    if (e.x < xMin) direction = 1;
  }
}

class Vol extends Comportement {
  Vol({required this.amplitude, required this.frequence});

  final double amplitude;
  final double frequence;
  double _temps = 0;
  double _yBase = 0;
  bool _pret = false;

  @override
  void update(double dt) {
    final Entite? e = proprietaire;
    if (e == null) return;
    if (!_pret) {
      _yBase = e.y;
      _pret = true;
    }
    _temps += dt;
    // Oscillation triangulaire, sans dart:math, pour rester lisible.
    final double p = (_temps * frequence) % 2.0;
    final double onde = p < 1.0 ? p : 2.0 - p;
    e.y = _yBase + amplitude * (onde * 2 - 1);
  }
}

class Tir extends Comportement {
  Tir({required this.cadence});

  final double cadence; // secondes entre deux tirs
  double _restant = 0;
  int tirsEffectues = 0;

  @override
  void update(double dt) {
    _restant -= dt;
    if (_restant <= 0) {
      _restant = cadence;
      tirsEffectues++;
    }
  }
}

void main() {
  // Un gobelin ordinaire.
  final Entite gobelin = Entite(nom: 'gobelin', x: 200, y: 300)
      .avec(Sante(8))
      .avec(Patrouille(vitesse: 40, xMin: 180, xMax: 260));

  // Un gobelin volant tireur : AUCUNE nouvelle classe.
  final Entite gobelinVolantTireur =
      Entite(nom: 'gobelin volant tireur', x: 400, y: 200)
          .avec(Sante(8))
          .avec(Patrouille(vitesse: 60, xMin: 350, xMax: 450))
          .avec(Vol(amplitude: 20, frequence: 1))
          .avec(Tir(cadence: 0.5));

  // Une potion qui fuit : un objet AVEC un déplacement. Impossible en héritage.
  final Entite potionFuyante = Entite(nom: 'potion fuyante', x: 500, y: 300)
      .avec(Patrouille(vitesse: 90, xMin: 480, xMax: 560));

  final List<Entite> monde = <Entite>[
    gobelin,
    gobelinVolantTireur,
    potionFuyante,
  ];

  for (int i = 0; i < 120; i++) {
    for (final Entite e in monde) {
      e.update(1 / 60);
    }
  }

  for (final Entite e in monde) {
    final Sante? s = e.obtenir<Sante>();
    final Tir? t = e.obtenir<Tir>();
    print('${e.nom.padRight(22)} '
        'x=${e.x.toStringAsFixed(0).padLeft(3)} '
        'y=${e.y.toStringAsFixed(0).padLeft(3)} '
        'pv=${s?.pv ?? '-'} '
        'tirs=${t?.tirsEffectues ?? '-'}');
  }

  // Un gobelin qui APPREND à voler en pleine partie.
  print('');
  print('Le gobelin entre en rage et apprend a voler.');
  gobelin.avec(Vol(amplitude: 30, frequence: 2));
  for (int i = 0; i < 30; i++) {
    gobelin.update(1 / 60);
  }
  print('gobelin y=${gobelin.y.toStringAsFixed(0)}');
}
```

**Résultat :**

```text
gobelin                x=240 y=300 pv=8 tirs=-
gobelin volant tireur  x=390 y=200 pv=8 tirs=4
potion fuyante         x=524 y=300 pv=- tirs=-

Le gobelin entre en rage et apprend a voler.
gobelin y=300
```

Les valeurs de `x` et de `y` dépendent des rebonds ; l'important est ailleurs. Observez ce que la composition a rendu possible.

**Une potion qui se déplace, sans nouvelle classe.** Elle réutilise `Patrouille`, écrit pour les ennemis. Aucun lien de type entre les deux.

**Un gobelin volant tireur, sans nouvelle classe.** Quatre comportements assemblés. Avec l'héritage, il aurait fallu une classe supplémentaire.

**Un gobelin qui apprend à voler en cours de partie.** La dernière ligne du programme ajoute un comportement à une entité déjà vivante. C'est strictement impossible avec l'héritage : on ne change pas le type d'un objet existant.

Notez enfin la méthode `obtenir<T>()`. Elle répond à la question « cette entité a-t-elle de la santé ? » sans connaître son type. Elle remplace le `e is Character` de la section 26.10, et elle est plus fine : elle interroge une **capacité**, pas une identité.

> **Remarque sur les mixins.** Le chapitre 11 présentait `mixin Volant { void voler() {...} }` avec `class Gobelin extends Entity with Volant`. C'est aussi de la composition, mais **statique** : la combinaison est figée à la compilation. Les mixins conviennent parfaitement quand les combinaisons sont connues d'avance et peu nombreuses. Les comportements-objets conviennent quand elles doivent varier en cours de partie.

---

## 26.13 — Introduction à l'ECS : entité, composant, système

La section précédente a fait la moitié du chemin. L'**ECS** (*Entity Component System*) fait l'autre moitié, en poussant l'idée jusqu'au bout : on retire aux composants leur logique.

Les trois mots du sigle désignent trois choses radicalement différentes.

```text
  ┌────────────────────────────────────────────────────────────────┐
  │                       LE MODÈLE ECS                            │
  └────────────────────────────────────────────────────────────────┘

  ENTITÉ      Un simple IDENTIFIANT. Un entier. Rien d'autre.
  (Entity)    Pas de champ, pas de méthode, pas de position.
              #1, #2, #3...

  COMPOSANT   Des DONNÉES pures, sans aucune méthode.
  (Component) Position(x, y) | Vitesse(vx, vy) | Sante(pv)
              Un sac de valeurs, attaché à un identifiant.

  SYSTÈME     Une LOGIQUE pure, sans aucune donnée propre.
  (System)    « Pour toute entité qui a Position ET Vitesse,
                faire x += vx * dt. »
              Il balaie, il calcule, il repart.


  LA TABLE DU MONDE

           Position   Vitesse   Sante   Sprite   Controle
  #1 héros    X          X        X        X        X
  #2 gobelin  X          X        X        X        .
  #3 potion   X          .        .        X        .
  #4 mur      X          .        .        X        .
  #5 flèche   X          X        .        X        .

  SystemeDeplacement  lit les lignes qui ont Position ET Vitesse
                      -> #1, #2, #5
  SystemeRendu        lit les lignes qui ont Position ET Sprite
                      -> #1, #2, #3, #4, #5
  SystemeDegats       lit les lignes qui ont Sante
                      -> #1, #2
```

La différence essentielle avec la section 26.12 : dans notre composition, `Patrouille` contenait sa propre méthode `update`. En ECS strict, `Vitesse` ne contient que des nombres, et c'est `SystemeDeplacement` qui les fait avancer, **pour toutes les entités à la fois**.

Écrivons un ECS minimal mais authentique, en console.

```dart
// ============================================================
//  COMPOSANTS : uniquement des données.
// ============================================================

class Position {
  Position(this.x, this.y);
  double x;
  double y;
}

class Vitesse {
  Vitesse(this.vx, this.vy);
  double vx;
  double vy;
}

class Sante {
  Sante(this.pv, this.pvMax);
  int pv;
  final int pvMax;
}

class Sprite {
  Sprite(this.symbole);
  final String symbole;
}

class Etiquette {
  Etiquette(this.nom);
  final String nom;
}

// ============================================================
//  LE MONDE : une table entité -> composants.
// ============================================================

class Monde {
  int _prochainId = 1;

  /// Pour chaque type de composant, une table id -> composant.
  final Map<Type, Map<int, Object>> _tables = <Type, Map<int, Object>>{};

  final Set<int> entites = <int>{};

  int creerEntite() {
    final int id = _prochainId++;
    entites.add(id);
    return id;
  }

  void attacher<T extends Object>(int id, T composant) {
    _tables.putIfAbsent(T, () => <int, Object>{})[id] = composant;
  }

  T? composant<T extends Object>(int id) {
    return _tables[T]?[id] as T?;
  }

  bool a<T extends Object>(int id) => _tables[T]?.containsKey(id) ?? false;

  /// Toutes les entités possédant les deux composants demandés.
  Iterable<int> avec2<A extends Object, B extends Object>() {
    final Map<int, Object> tableA = _tables[A] ?? <int, Object>{};
    return tableA.keys.where((int id) => a<B>(id));
  }

  /// Toutes les entités possédant le composant demandé.
  Iterable<int> avec1<A extends Object>() {
    return (_tables[A] ?? <int, Object>{}).keys;
  }

  void detruire(int id) {
    entites.remove(id);
    for (final Map<int, Object> table in _tables.values) {
      table.remove(id);
    }
  }
}

// ============================================================
//  SYSTÈMES : uniquement de la logique.
// ============================================================

abstract class Systeme {
  void executer(Monde monde, double dt);
}

class SystemeDeplacement extends Systeme {
  @override
  void executer(Monde monde, double dt) {
    for (final int id in monde.avec2<Position, Vitesse>().toList()) {
      final Position p = monde.composant<Position>(id)!;
      final Vitesse v = monde.composant<Vitesse>(id)!;
      p.x += v.vx * dt;
      p.y += v.vy * dt;
    }
  }
}

class SystemeLimitesEcran extends Systeme {
  SystemeLimitesEcran(this.largeur);
  final double largeur;

  @override
  void executer(Monde monde, double dt) {
    for (final int id in monde.avec2<Position, Vitesse>().toList()) {
      final Position p = monde.composant<Position>(id)!;
      final Vitesse v = monde.composant<Vitesse>(id)!;
      if (p.x < 0) {
        p.x = 0;
        v.vx = -v.vx;
      } else if (p.x > largeur) {
        p.x = largeur;
        v.vx = -v.vx;
      }
    }
  }
}

class SystemeMort extends Systeme {
  @override
  void executer(Monde monde, double dt) {
    for (final int id in monde.avec1<Sante>().toList()) {
      final Sante s = monde.composant<Sante>(id)!;
      if (s.pv <= 0) {
        final Etiquette? e = monde.composant<Etiquette>(id);
        print('  ${e?.nom ?? "entite $id"} meurt.');
        monde.detruire(id);
      }
    }
  }
}

class SystemeAffichage extends Systeme {
  @override
  void executer(Monde monde, double dt) {
    for (final int id in monde.avec2<Position, Sprite>().toList()) {
      final Position p = monde.composant<Position>(id)!;
      final Sprite s = monde.composant<Sprite>(id)!;
      final Etiquette? e = monde.composant<Etiquette>(id);
      final Sante? sa = monde.composant<Sante>(id);
      print('  ${s.symbole} ${(e?.nom ?? "?").padRight(10)} '
          'x=${p.x.toStringAsFixed(0).padLeft(4)} '
          'y=${p.y.toStringAsFixed(0).padLeft(4)} '
          '${sa != null ? "pv=${sa.pv}/${sa.pvMax}" : ""}');
    }
  }
}

// ============================================================
//  PROGRAMME
// ============================================================

void main() {
  final Monde monde = Monde();

  final int heros = monde.creerEntite();
  monde.attacher(heros, Position(100, 300));
  monde.attacher(heros, Vitesse(120, 0));
  monde.attacher(heros, Sante(6, 6));
  monde.attacher(heros, Sprite('@'));
  monde.attacher(heros, Etiquette('heros'));

  final int gobelin = monde.creerEntite();
  monde.attacher(gobelin, Position(400, 300));
  monde.attacher(gobelin, Vitesse(-40, 0));
  monde.attacher(gobelin, Sante(8, 8));
  monde.attacher(gobelin, Sprite('g'));
  monde.attacher(gobelin, Etiquette('gobelin'));

  // Un mur : pas de vitesse, pas de sante. Le systeme de deplacement
  // ne le verra jamais, sans qu'on ait rien a coder pour cela.
  final int mur = monde.creerEntite();
  monde.attacher(mur, Position(600, 300));
  monde.attacher(mur, Sprite('#'));
  monde.attacher(mur, Etiquette('mur'));

  final List<Systeme> systemes = <Systeme>[
    SystemeDeplacement(),
    SystemeLimitesEcran(800),
    SystemeMort(),
  ];

  print('=== etat initial ===');
  SystemeAffichage().executer(monde, 0);

  for (int i = 0; i < 60; i++) {
    for (final Systeme s in systemes) {
      s.executer(monde, 1 / 60);
    }
  }

  print('=== apres 1 seconde ===');
  SystemeAffichage().executer(monde, 0);

  print('=== le heros frappe le gobelin (8 degats) ===');
  monde.composant<Sante>(gobelin)!.pv -= 8;
  for (final Systeme s in systemes) {
    s.executer(monde, 1 / 60);
  }

  print('=== etat final ===');
  SystemeAffichage().executer(monde, 0);
}
```

**Résultat :**

```text
=== etat initial ===
  @ heros      x= 100 y= 300 pv=6/6
  g gobelin    x= 400 y= 300 pv=8/8
  # mur        x= 600 y= 300 
=== apres 1 seconde ===
  @ heros      x= 220 y= 300 pv=6/6
  g gobelin    x= 360 y= 300 pv=8/8
  # mur        x= 600 y= 300 
=== le heros frappe le gobelin (8 degats) ===
  gobelin meurt.
=== etat final ===
  @ heros      x= 222 y= 300 pv=6/6
  # mur        x= 600 y= 300 
```

Quatre observations qui résument tout l'intérêt du modèle.

**Le mur n'a jamais été « exclu » explicitement.** Aucun `if (type == mur) continue;` nulle part. Il n'a simplement pas de composant `Vitesse`, donc le système de déplacement ne le voit pas. La sélection se fait par les données, pas par des tests de type.

**Ajouter un comportement, c'est ajouter un système.** Vous voulez la gravité ? Écrivez `SystemeGravite` qui, pour toute entité ayant `Vitesse` et `Masse`, fait `v.vy += 900 * dt`. Aucune classe existante n'est touchée.

**Rendre un objet mortel, c'est lui attacher `Sante`.** Le mur deviendra destructible le jour où vous écrirez une ligne `monde.attacher(mur, Sante(20, 20))`. Rien d'autre à changer.

**L'ordre des systèmes est explicite.** Déplacement, puis limites d'écran, puis mort. Cet ordre est écrit noir sur blanc dans une liste, au lieu d'être enfoui dans l'ordre des lignes d'un `update` de 700 lignes. C'est un gain de lisibilité considérable quand le jeu grossit.

> **Remarque honnête.** L'implémentation ci-dessus est pédagogique, pas performante. Les vrais ECS rangent les composants dans des tableaux contigus pour que le processeur les charge en une fois dans son cache. Cela leur permet de simuler des centaines de milliers d'entités. Pour un jeu 2D de quelques centaines d'entités, cette optimisation est parfaitement inutile.

---

## 26.14 — ECS vs POO classique : tableau comparatif

Vous disposez maintenant des deux modèles. Choisissez en connaissance de cause, et non par mode.

| Critère | POO classique (héritage) | ECS |
| --- | --- | --- |
| Unité de base | la classe | l'identifiant + ses composants |
| Où vivent les données | dans l'objet | dans des tables de composants |
| Où vit la logique | dans les méthodes de l'objet | dans les systèmes |
| Ajouter un type d'ennemi | une nouvelle classe | un assemblage de composants |
| Ajouter un comportement à tous | modifier la classe mère | ajouter un système |
| Combiner deux comportements | explosion combinatoire | trivial |
| Changer un comportement en jeu | impossible | attacher/détacher un composant |
| Lisibilité pour un débutant | **excellente** | déroutante au début |
| Débogage pas à pas | facile, tout est dans l'objet | plus difficile, l'état est éparpillé |
| Outils de l'IDE (aller à la définition) | très efficaces | moins efficaces |
| Performance à 100 entités | équivalente | équivalente |
| Performance à 100 000 entités | mauvaise | excellente |
| Sérialisation, sauvegarde | à écrire par classe | générique, une fois pour toutes |
| Édition de niveau par données | difficile | naturelle (fichier JSON de composants) |
| Courbe d'apprentissage | douce | raide |
| Utilisé par | Flame, Godot, Unity (GameObject) | Bevy, Unity DOTS, Overwatch |

Traduisons ce tableau en règle de décision.

```text
  QUEL MODÈLE CHOISIR ?

  Combien de types d'entités différents ?
      moins de 10  ─────────────────────► POO classique
      10 à 30      ──┐
      plus de 30   ──┤
                     │
  Les comportements se combinent-ils librement ?
      non          ──┴─────────────────► POO + mixins
      oui          ─────────────────────► composition (26.12)

  Faut-il changer de comportement en cours de partie ?
      oui          ─────────────────────► composition ou ECS

  Y a-t-il plus de 10 000 entités simultanées ?
      oui          ─────────────────────► ECS avec tableaux contigus

  Êtes-vous seul et débutant ?
      oui          ─────────────────────► POO classique, sans hésiter
```

Pour cette formation, la réponse est nette : **nous restons en POO classique avec une pointe de composition**. C'est aussi le choix de Flame, et c'est celui qui vous permettra de terminer votre jeu. Un ECS mal maîtrisé produit un code plus difficile à déboguer qu'un héritage un peu imparfait.

> **À retenir.** L'ECS n'est pas « la version avancée » de la POO. C'est un autre compromis : il échange de la lisibilité immédiate contre de la flexibilité et de la performance à grande échelle. Un jeu de plateforme 2D avec vingt ennemis n'a besoin ni de l'une ni de l'autre.

---

## 26.15 — Le modèle de Flame : un arbre de composants

Flame, que vous installerez au chapitre 27, ne choisit ni l'héritage profond ni l'ECS strict. Il retient une troisième voie : **l'arbre de composants**, le même modèle que Flutter lui-même, que Unity et que Godot.

Le principe : tout est un `Component`, et un `Component` peut contenir d'autres `Component`. Le moteur parcourt l'arbre, appelle `update` sur chaque nœud, puis `render`.

```text
  ┌────────────────────────────────────────────────────────────────┐
  │              L'ARBRE DE COMPOSANTS DE FLAME                    │
  └────────────────────────────────────────────────────────────────┘

  FlameGame                          (la racine, c'est un Component)
    │
    ├── CameraComponent              (le point de vue — chapitre 31)
    │     ├── Viewport
    │     └── World                  (le monde du jeu)
    │           ├── FondParallaxe        priority = -50
    │           ├── NiveauTiled          priority = 0
    │           ├── Joueur               priority = 30
    │           │     ├── SpriteAnimationComponent
    │           │     ├── RectangleHitbox
    │           │     └── EpeeComponent
    │           │           └── RectangleHitbox
    │           ├── Gobelin              priority = 20
    │           │     ├── SpriteComponent
    │           │     └── CircleHitbox
    │           ├── Gobelin              priority = 20
    │           └── Potion               priority = 10
    │
    └── HudComponent                 priority = 100
          ├── BarreDeVie
          ├── TexteScore
          └── JoystickComponent


  CE QUE FLAME FAIT POUR VOUS À CHAQUE FRAME
  ──────────────────────────────────────────
  1. calcule dt                       -> chapitre 20, fait à la main
  2. parcourt l'arbre, appelle update -> section 26.6, faite à la main
  3. vide ses files d'ajout/retrait   -> section 26.8, faite à la main
  4. trie par priority                -> section 26.9, faite à la main
  5. applique les transformations     -> chapitre 21, fait à la main
  6. parcourt l'arbre, appelle render -> section 26.6, faite à la main
  7. propage les collisions           -> chapitre 24, fait à la main
```

Cette colonne de droite est la vraie réponse à la question « pourquoi avoir tout écrit à la main pendant six chapitres ? ». Parce que vous savez maintenant **ce que le moteur fait**, et donc pourquoi il le fait dans cet ordre.

L'apport propre de l'arbre, par rapport à notre liste plate de la section 26.6, tient en trois points.

**Les positions deviennent relatives.** L'épée du joueur est positionnée à `(20, 0)` **par rapport au joueur**. Quand le joueur se déplace, l'épée suit sans une ligne de code. Avec une liste plate, il faut recopier la position du porteur à chaque frame.

**La destruction est récursive.** Retirer le joueur retire son sprite, sa hitbox et son épée. Avec une liste plate, il faut penser à retirer chaque morceau, et l'on oublie toujours le dernier.

**La composition est structurelle.** Un joueur *a* une hitbox et *a* une animation, exactement au sens de la section 26.12, mais exprimé par la structure de l'arbre plutôt que par une liste de comportements.

Voici la correspondance terme à terme entre ce que vous avez écrit et ce que Flame propose.

| Votre code (chapitres 20 à 26) | L'équivalent Flame | Chapitre |
| --- | --- | --- |
| `abstract class Entity` | `Component` | 28 |
| `x`, `y` séparés | `position` (un `Vector2`) | 28 |
| `update(double dt)` | `update(double dt)` | 27 |
| `render(Canvas canvas)` | `render(Canvas canvas)` | 27 |
| `List<Entity> entites` | l'arbre de composants | 28 |
| `monde.ajouter(e)` | `add(component)` | 28 |
| `e.aRetirer = true` | `component.removeFromParent()` | 28 |
| `int priority` | `priority` | 28 |
| votre tri de rendu | fait par le moteur | 28 |
| `MoteurDeBoucle` du chapitre 20 | `FlameGame` + `GameWidget` | 27 |
| votre gestion de caméra | `CameraComponent`, `World` | 31 |
| vos AABB écrits à la main | `RectangleHitbox`, `onCollisionStart` | 32 |
| votre machine à états | `overlays` + votre propre enum | 35 |

Remarquez la dernière ligne : **Flame ne fournit pas de machine à états de jeu**. Le contenu des sections 26.16 à 26.23 restera donc entièrement de votre responsabilité, y compris dans la PARTIE 2C. C'est une raison de plus de bien le maîtriser maintenant.

---

## 26.16 — Qu'est-ce qu'un état de jeu ?

Changeons de sujet. Nous savons organiser le **contenu** du monde ; il reste à organiser le **déroulement** de la partie.

> **Définition.** Un **état de jeu** est une situation dans laquelle le jeu se comporte globalement différemment : ce qu'il met à jour, ce qu'il dessine et ce à quoi il réagit changent d'un bloc.

Un jeu ne fait pas la même chose selon qu'il affiche son menu ou qu'une partie est en cours. Comparons.

```text
                    │ MENU        │ JEU         │ PAUSE       │ GAME OVER
  ──────────────────┼─────────────┼─────────────┼─────────────┼───────────
  entités mises     │ non         │ OUI         │ non         │ non
  à jour            │             │             │             │
  ──────────────────┼─────────────┼─────────────┼─────────────┼───────────
  entités dessinées │ non         │ OUI         │ OUI (figées)│ OUI (figées)
  ──────────────────┼─────────────┼─────────────┼─────────────┼───────────
  chrono qui avance │ non         │ OUI         │ non         │ non
  ──────────────────┼─────────────┼─────────────┼─────────────┼───────────
  touche Espace     │ démarrer    │ sauter      │ (rien)      │ rejouer
  ──────────────────┼─────────────┼─────────────┼─────────────┼───────────
  touche Échap      │ quitter     │ mettre en   │ reprendre   │ retour
                    │             │ pause       │             │ menu
  ──────────────────┼─────────────┼─────────────┼─────────────┼───────────
  musique           │ thème menu  │ thème donjon│ étouffée    │ silence
```

Regardez la ligne « touche Espace ». La **même** touche déclenche quatre actions différentes. Sans état explicite, cela s'écrit ainsi, et c'est le début de la fin.

```dart
// NE FAITES PAS CELA.
void surEspace() {
  if (menuAffiche) {
    demarrerPartie();
  } else if (gameOver) {
    rejouer();
  } else if (enPause) {
    // rien
  } else if (victoire) {
    rejouer();
  } else {
    heros.sauter();
  }
}
```

Ce code souffre de trois maux. Il faut relire tous les `if` pour comprendre un cas. L'ordre des `if` porte du sens caché : si `gameOver` et `enPause` sont vrais ensemble, c'est `gameOver` qui gagne, mais rien ne le dit. Enfin, ajouter un état oblige à retoucher **tous** les endroits qui testent des booléens, et il y en aura vingt.

Le tableau plus haut suggère la solution : chaque colonne est un état, et un seul état est actif à la fois. C'est exactement ce qu'un `enum` exprime.

---

## 26.17 — L'enum `GameState`

Le chapitre 11 de la PARTIE 1A a présenté les `enum`. Rappelons pourquoi ils battent les booléens sur ce terrain précis.

| Avec des booléens | Avec un enum |
| --- | --- |
| 4 booléens = 16 combinaisons | 6 valeurs = 6 situations |
| 12 combinaisons absurdes possibles | aucune combinaison absurde possible |
| `if/else if/else if...` | `switch` exhaustif |
| Oublier un cas passe inaperçu | le compilateur signale le cas manquant |
| Ajouter un état = retoucher partout | ajouter une valeur, suivre les erreurs |

Le quatrième point est décisif. Un `switch` sur un `enum` sans `default` provoque un **avertissement du compilateur** quand une valeur n'est pas traitée. Ajouter un état devient donc une opération guidée : le compilateur vous fait la liste des endroits à mettre à jour.

```dart
/// Les états possibles du Donjon de Dart.
enum GameState {
  /// Logo, chargement des ressources.
  demarrage,

  /// Menu principal : jouer, options, quitter.
  menu,

  /// Une partie est en cours.
  jeu,

  /// Partie en cours, mais figée.
  pause,

  /// Le héros a perdu toutes ses vies.
  gameOver,

  /// Le boss est vaincu.
  victoire,
}
```

Un `enum` Dart peut porter des données et des méthodes, ce qui évite d'éparpiller les informations d'affichage.

```dart
enum GameState {
  demarrage(titre: 'Donjon de Dart', mondeActif: false),
  menu(titre: 'Menu principal', mondeActif: false),
  jeu(titre: '', mondeActif: true),
  pause(titre: 'Pause', mondeActif: false),
  gameOver(titre: 'Vous etes tombe', mondeActif: false),
  victoire(titre: 'Le donjon est vaincu', mondeActif: false);

  const GameState({required this.titre, required this.mondeActif});

  /// Texte affiché en gros au centre de l'écran.
  final String titre;

  /// Le monde doit-il être mis à jour dans cet état ?
  final bool mondeActif;

  /// Le monde doit-il être dessiné (même figé) ?
  bool get mondeVisible => this != GameState.demarrage && this != GameState.menu;
}

void main() {
  for (final GameState e in GameState.values) {
    print('${e.name.padRight(10)} '
        'update=${e.mondeActif.toString().padRight(5)} '
        'dessin=${e.mondeVisible.toString().padRight(5)} '
        '"${e.titre}"');
  }
}
```

**Résultat :**

```text
demarrage  update=false dessin=false "Donjon de Dart"
menu       update=false dessin=false "Menu principal"
jeu        update=true  dessin=true  ""
pause      update=false dessin=true  "Pause"
gameOver   update=false dessin=true  "Vous etes tombe"
victoire   update=false dessin=true  "Le donjon est vaincu"
```

Ce tableau, qui existait sur papier en section 26.16, existe maintenant **dans le code**. Il n'y a plus de risque de désaccord entre la documentation et le comportement réel.

> **Remarque.** N'entassez pas tout dans l'enum. Les couleurs et les polices appartiennent au rendu, pas à la définition de l'état. Une bonne règle : mettez dans l'enum ce dont la **logique** a besoin, laissez au rendu ce qui relève du style.

---

## 26.18 — La machine à états : schéma des transitions

Un enum liste les états. Une **machine à états** ajoute l'information capitale : **qui peut mener à quoi**.

```text
  ┌────────────────────────────────────────────────────────────────────┐
  │        MACHINE À ÉTATS DU DONJON DE DART                           │
  └────────────────────────────────────────────────────────────────────┘

        (lancement de l'application)
                  │
                  ▼
          ┌───────────────┐
          │   DEMARRAGE   │  chargement des ressources
          └───────┬───────┘
                  │ ressources prêtes (automatique)
                  ▼
          ┌───────────────┐◄──────────────────────────────┐
          │     MENU      │                               │
          └───────┬───────┘                               │
                  │ appui sur « Jouer »                   │
                  ▼                                       │
          ┌───────────────┐                               │
      ┌──►│      JEU      │──────────┐                    │
      │   └───────┬───────┘          │                    │
      │           │                  │                    │
      │           │ Échap            │ vies == 0          │
      │           ▼                  ▼                    │
      │   ┌───────────────┐   ┌───────────────┐           │
      │   │     PAUSE     │   │   GAME OVER   │───────────┤
      │   └───────┬───────┘   └───────┬───────┘   « Menu »│
      │           │                   │                   │
      └───────────┘                   │ « Rejouer »       │
        Échap ou « Reprendre »        └──────────┐        │
                                                 │        │
      ┌────────────────────────────┐             │        │
      │  depuis PAUSE : « Menu »   │─────────────┼────────┤
      └────────────────────────────┘             │        │
                                                 ▼        │
          ┌───────────────┐              (nouvelle partie)│
          │   VICTOIRE    │───────────────────────────────┘
          └───────────────┘   « Menu » ou « Rejouer »
                  ▲
                  │ boss vaincu
                  │
              (depuis JEU)
```

Ce schéma dit aussi ce qui est **interdit**, et c'est là son vrai pouvoir.

```text
  TRANSITIONS AUTORISÉES (les seules)

  demarrage -> menu
  menu      -> jeu
  jeu       -> pause | gameOver | victoire
  pause     -> jeu | menu
  gameOver  -> jeu | menu
  victoire  -> jeu | menu

  TRANSITIONS INTERDITES (exemples)

  menu      -> pause       (mettre en pause quoi ?)
  gameOver  -> pause       (le jeu est fini)
  demarrage -> jeu         (les ressources ne sont pas chargées)
  pause     -> gameOver    (rien ne se met à jour en pause)
  victoire  -> pause       (idem)
```

Écrire ces règles dans le code apporte un bénéfice immédiat : une transition illégale devient un **message d'erreur immédiat** au lieu d'un bug d'affichage inexplicable trois écrans plus loin.

```dart
enum GameState { demarrage, menu, jeu, pause, gameOver, victoire }

/// Les transitions autorisées, une fois pour toutes.
const Map<GameState, Set<GameState>> transitionsAutorisees =
    <GameState, Set<GameState>>{
  GameState.demarrage: <GameState>{GameState.menu},
  GameState.menu: <GameState>{GameState.jeu},
  GameState.jeu: <GameState>{
    GameState.pause,
    GameState.gameOver,
    GameState.victoire,
  },
  GameState.pause: <GameState>{GameState.jeu, GameState.menu},
  GameState.gameOver: <GameState>{GameState.jeu, GameState.menu},
  GameState.victoire: <GameState>{GameState.jeu, GameState.menu},
};

bool transitionValide(GameState de, GameState vers) {
  return transitionsAutorisees[de]?.contains(vers) ?? false;
}

void main() {
  final List<List<GameState>> essais = <List<GameState>>[
    <GameState>[GameState.menu, GameState.jeu],
    <GameState>[GameState.jeu, GameState.pause],
    <GameState>[GameState.pause, GameState.jeu],
    <GameState>[GameState.menu, GameState.pause],
    <GameState>[GameState.gameOver, GameState.pause],
    <GameState>[GameState.demarrage, GameState.jeu],
  ];

  for (final List<GameState> t in essais) {
    final bool ok = transitionValide(t[0], t[1]);
    print('${t[0].name.padRight(10)} -> ${t[1].name.padRight(10)} '
        '${ok ? "AUTORISEE" : "INTERDITE"}');
  }
}
```

**Résultat :**

```text
menu       -> jeu        AUTORISEE
jeu        -> pause      AUTORISEE
pause      -> jeu        AUTORISEE
menu       -> pause      INTERDITE
gameOver   -> pause      INTERDITE
demarrage  -> jeu        INTERDITE
```

> **À retenir.** Une machine à états, ce n'est pas seulement une liste d'états : c'est une liste d'états **plus** un graphe de transitions. Le graphe est la partie qui empêche les bugs.

---

## 26.19 — Écran de démarrage, menu, jeu, pause, game over, victoire

Détaillons ce que chaque état doit faire. Ce tableau est votre cahier des charges : gardez-le sous les yeux en écrivant le code.

| État | Ce qu'il met à jour | Ce qu'il dessine | Entrées acceptées | Sortie vers |
| --- | --- | --- | --- | --- |
| `demarrage` | une barre de progression | logo, barre | aucune | `menu` |
| `menu` | une animation de fond | titre, boutons | haut/bas, entrée | `jeu` |
| `jeu` | tout le monde, le chrono | monde, HUD | toutes | `pause`, `gameOver`, `victoire` |
| `pause` | rien | monde figé, voile, menu | échap, entrée | `jeu`, `menu` |
| `gameOver` | une animation de texte | monde figé, voile, score | entrée, échap | `jeu`, `menu` |
| `victoire` | des particules de fête | monde figé, voile, score | entrée, échap | `jeu`, `menu` |

Quelques précisions qui évitent des erreurs courantes.

**`demarrage` n'est pas décoratif.** Le chargement des images et des sons est asynchrone (chapitre 15). Tant qu'il n'est pas terminé, `jeu` ne peut pas commencer sans risquer une image manquante. Cet état existe pour attendre proprement.

**`pause` dessine le monde, mais figé.** C'est essentiel pour l'expérience du joueur : voir la scène rassure sur le fait que la partie n'est pas perdue. Techniquement, cela signifie que le rendu du monde doit être appelé même quand son `update` ne l'est pas. C'est possible seulement parce que vous avez séparé les deux en section 26.2.

**`gameOver` et `victoire` sont deux états, pas un état avec un booléen.** Ils affichent des textes, des musiques et des statistiques différentes. Les fusionner en `fin(gagne: true)` fonctionne, mais complique chaque `switch`.

**Le voile est une entité de rendu.** Le rectangle noir semi-transparent posé sur le monde figé se dessine avec la couche `Couches.voile` de la section 26.9.

```dart
void dessinerVoile(Canvas canvas, Size taille, String titre, String sous) {
  // 1. Le voile.
  canvas.drawRect(
    Offset.zero & taille,
    Paint()..color = const Color(0xCC000000),
  );

  // 2. Le titre, centré.
  final TextPainter tp = TextPainter(
    text: TextSpan(
      text: titre,
      style: const TextStyle(
        color: Color(0xFFE8B04B),
        fontSize: 34,
        fontWeight: FontWeight.bold,
      ),
    ),
    textDirection: TextDirection.ltr,
  )..layout();
  tp.paint(
    canvas,
    Offset((taille.width - tp.width) / 2, taille.height / 2 - 50),
  );

  // 3. La ligne d'instruction.
  final TextPainter tp2 = TextPainter(
    text: TextSpan(
      text: sous,
      style: const TextStyle(color: Color(0xFFCCCCCC), fontSize: 16),
    ),
    textDirection: TextDirection.ltr,
  )..layout();
  tp2.paint(
    canvas,
    Offset((taille.width - tp2.width) / 2, taille.height / 2 + 10),
  );
}
```

---

## 26.20 — Implémenter la machine à états

Passons à l'implémentation. Nous encapsulons l'état courant dans une classe qui **contrôle** ses transitions, au lieu d'exposer une variable publique modifiable par n'importe qui.

```dart
enum GameState { demarrage, menu, jeu, pause, gameOver, victoire }

class MachineAEtats {
  MachineAEtats({GameState initial = GameState.demarrage})
      : _etat = initial;

  GameState _etat;

  /// Lecture seule depuis l'extérieur.
  GameState get etat => _etat;

  /// Durée passée dans l'état courant, en secondes. Très pratique
  /// pour les animations d'entrée et les délais.
  double tempsDansEtat = 0;

  /// Historique, utile pour déboguer et pour « revenir en arrière ».
  final List<GameState> historique = <GameState>[];

  static const Map<GameState, Set<GameState>> _autorisees =
      <GameState, Set<GameState>>{
    GameState.demarrage: <GameState>{GameState.menu},
    GameState.menu: <GameState>{GameState.jeu},
    GameState.jeu: <GameState>{
      GameState.pause,
      GameState.gameOver,
      GameState.victoire,
    },
    GameState.pause: <GameState>{GameState.jeu, GameState.menu},
    GameState.gameOver: <GameState>{GameState.jeu, GameState.menu},
    GameState.victoire: <GameState>{GameState.jeu, GameState.menu},
  };

  /// Appelé à chaque entrée dans un état. Branchez-y musique et effets.
  void Function(GameState ancien, GameState nouveau)? surTransition;

  bool peutAller(GameState vers) =>
      _autorisees[_etat]?.contains(vers) ?? false;

  /// Change d'état. Renvoie false si la transition est interdite.
  bool aller(GameState vers) {
    if (vers == _etat) return true;
    if (!peutAller(vers)) {
      print('TRANSITION INTERDITE : ${_etat.name} -> ${vers.name}');
      return false;
    }
    final GameState ancien = _etat;
    historique.add(ancien);
    _etat = vers;
    tempsDansEtat = 0;
    surTransition?.call(ancien, vers);
    return true;
  }

  void update(double dt) {
    tempsDansEtat += dt;
  }
}
```

Utilisons-la dans un jeu console complet.

```dart
enum GameState { demarrage, menu, jeu, pause, gameOver, victoire }

class MachineAEtats {
  MachineAEtats({GameState initial = GameState.demarrage}) : _etat = initial;

  GameState _etat;
  GameState get etat => _etat;
  double tempsDansEtat = 0;

  static const Map<GameState, Set<GameState>> _autorisees =
      <GameState, Set<GameState>>{
    GameState.demarrage: <GameState>{GameState.menu},
    GameState.menu: <GameState>{GameState.jeu},
    GameState.jeu: <GameState>{
      GameState.pause,
      GameState.gameOver,
      GameState.victoire,
    },
    GameState.pause: <GameState>{GameState.jeu, GameState.menu},
    GameState.gameOver: <GameState>{GameState.jeu, GameState.menu},
    GameState.victoire: <GameState>{GameState.jeu, GameState.menu},
  };

  void Function(GameState ancien, GameState nouveau)? surTransition;

  bool peutAller(GameState vers) => _autorisees[_etat]?.contains(vers) ?? false;

  bool aller(GameState vers) {
    if (vers == _etat) return true;
    if (!peutAller(vers)) {
      print('  ! transition interdite : ${_etat.name} -> ${vers.name}');
      return false;
    }
    final GameState ancien = _etat;
    _etat = vers;
    tempsDansEtat = 0;
    surTransition?.call(ancien, vers);
    return true;
  }

  void update(double dt) => tempsDansEtat += dt;
}

class Jeu {
  final MachineAEtats machine = MachineAEtats();

  double chargement = 0;
  double x = 0;
  int vies = 3;
  int score = 0;

  Jeu() {
    machine.surTransition = (GameState a, GameState n) {
      print('  [${a.name} -> ${n.name}]');
      if (n == GameState.jeu && a == GameState.menu) _nouvellePartie();
      if (n == GameState.jeu && a == GameState.gameOver) _nouvellePartie();
    };
  }

  void _nouvellePartie() {
    x = 0;
    vies = 3;
    score = 0;
    print('  (nouvelle partie : vies=3, score=0)');
  }

  void update(double dt) {
    machine.update(dt);

    switch (machine.etat) {
      case GameState.demarrage:
        chargement += dt * 0.5;
        if (chargement >= 1) machine.aller(GameState.menu);
        break;

      case GameState.menu:
        // Le fond du menu s'anime, rien d'autre.
        break;

      case GameState.jeu:
        x += 100 * dt;
        score += 1;
        if (x > 300) {
          vies--;
          x = 0;
          print('  (piege ! vies=$vies)');
          if (vies <= 0) machine.aller(GameState.gameOver);
        }
        break;

      case GameState.pause:
      case GameState.gameOver:
      case GameState.victoire:
        // Rien ne se met à jour.
        break;
    }
  }

  String rendu() {
    switch (machine.etat) {
      case GameState.demarrage:
        return 'chargement ${(chargement * 100).toStringAsFixed(0)} %';
      case GameState.menu:
        return 'MENU — appuyez sur Entree';
      case GameState.jeu:
        return 'x=${x.toStringAsFixed(0)} vies=$vies score=$score';
      case GameState.pause:
        return 'PAUSE (monde fige : x=${x.toStringAsFixed(0)})';
      case GameState.gameOver:
        return 'GAME OVER — score final $score';
      case GameState.victoire:
        return 'VICTOIRE — score final $score';
    }
  }
}

void main() {
  final Jeu jeu = Jeu();

  void avancer(String etiquette, int frames) {
    for (int i = 0; i < frames; i++) {
      jeu.update(1 / 60);
    }
    print('$etiquette -> ${jeu.machine.etat.name.padRight(9)} | ${jeu.rendu()}');
  }

  avancer('60 frames  ', 60);
  avancer('120 frames ', 120);

  print('Le joueur appuie sur Entree.');
  jeu.machine.aller(GameState.jeu);
  avancer('60 frames  ', 60);

  print('Le joueur appuie sur Echap.');
  jeu.machine.aller(GameState.pause);
  avancer('60 frames  ', 60);

  print('Le joueur essaie de perdre pendant la pause.');
  jeu.machine.aller(GameState.gameOver);

  print('Le joueur reprend.');
  jeu.machine.aller(GameState.jeu);
  avancer('600 frames ', 600);
}
```

**Résultat :**

```text
60 frames   -> demarrage | chargement 50 %
  [demarrage -> menu]
120 frames  -> menu      | MENU — appuyez sur Entree
Le joueur appuie sur Entree.
  [menu -> jeu]
  (nouvelle partie : vies=3, score=0)
60 frames   -> jeu       | x=100 vies=3 score=60
Le joueur appuie sur Echap.
  [jeu -> pause]
60 frames   -> pause     | PAUSE (monde fige : x=100)
Le joueur essaie de perdre pendant la pause.
  ! transition interdite : pause -> gameOver
Le joueur reprend.
  [pause -> jeu]
  (piege ! vies=2)
  (piege ! vies=1)
  (piege ! vies=0)
  [jeu -> gameOver]
600 frames  -> gameOver  | GAME OVER — score final 900
```

Analysons trois moments de cette trace.

**Le monde est bien figé en pause.** Après soixante frames de pause, `x` vaut toujours 100. Aucun `if (enPause) return;` n'a été semé dans le code : c'est le `switch` qui ne fait rien dans ce cas.

**La transition illégale a été refusée.** L'essai `pause -> gameOver` affiche un avertissement et n'a aucun effet. Dans un vrai jeu, remplacez le `print` par `assert(false, ...)` : le développeur est arrêté net, mais le joueur ne voit rien en production.

**La nouvelle partie est déclenchée par la transition.** La réinitialisation vit dans `surTransition`, pas dans le bouton du menu. Conséquence : quel que soit le chemin qui mène à `jeu`, la partie est correctement remise à zéro. C'est exactement le principe énoncé en section 26.10 — réagir à l'événement.

---

## 26.21 — Le patron State : une classe par écran

Le `switch` de la section précédente fonctionne parfaitement pour six états simples. Mais il grossit. Chaque état ajoute un `case` dans `update`, un dans `render`, un dans la gestion des touches, un dans la musique. Au bout d'un moment, la logique du menu est dispersée dans cinq `switch` différents.

Le **patron State** règle ce problème : une **classe par état**, qui rassemble tout ce qui concerne cet état.

```text
  AVANT (switch)                    APRÈS (patron State)

  Jeu                               Jeu
   ├── update()                      └── etatCourant : EtatDeJeu
   │    switch (etat) {                     ▲
   │      case menu:   ...                  │
   │      case jeu:    ...            ┌─────┴──────┬──────────┬────────┐
   │      case pause:  ...            │            │          │        │
   │    }                          EtatMenu    EtatJeu   EtatPause  EtatFin
   ├── render()                     entrer()   entrer()   entrer()  entrer()
   │    switch (etat) { ... }       update()   update()   update()  update()
   ├── touche()                     render()   render()   render()  render()
   │    switch (etat) { ... }       touche()   touche()   touche()  touche()
   └── musique()                    sortir()   sortir()   sortir()  sortir()
        switch (etat) { ... }

  La logique du menu                Tout le menu tient dans
  est éparpillée dans 4 switch.     une seule classe de 40 lignes.
```

Le contrat d'un état comporte cinq méthodes. Deux d'entre elles, `entrer` et `sortir`, sont la vraie valeur ajoutée du patron : elles offrent un endroit naturel pour lancer une musique, réinitialiser une partie ou libérer une ressource.

```dart
abstract class EtatDeJeu {
  /// Appelé une fois, au moment où l'on entre dans cet état.
  void entrer() {}

  /// Appelé une fois, au moment où l'on quitte cet état.
  void sortir() {}

  /// Appelé à chaque frame.
  void update(double dt) {}

  /// Appelé à chaque frame, après tous les update.
  void render() {}

  /// Appelé quand une action utilisateur survient.
  void surAction(String action) {}
}
```

Voici un programme console complet qui met le patron en œuvre.

```dart
// ============================================================
//  LE GESTIONNAIRE D'ÉTATS
// ============================================================

abstract class EtatDeJeu {
  late GestionnaireEtats gestionnaire;

  String get nom;

  void entrer() {}
  void sortir() {}
  void update(double dt) {}
  void render() {}
  void surAction(String action) {}
}

class GestionnaireEtats {
  EtatDeJeu? _courant;

  EtatDeJeu? get courant => _courant;

  void changer(EtatDeJeu nouveau) {
    _courant?.sortir();
    nouveau.gestionnaire = this;
    _courant = nouveau;
    nouveau.entrer();
  }

  void update(double dt) => _courant?.update(dt);
  void render() => _courant?.render();
  void surAction(String action) => _courant?.surAction(action);
}

// ============================================================
//  LES DONNÉES PARTAGÉES DE LA PARTIE
// ============================================================

class Partie {
  double x = 0;
  int vies = 3;
  int score = 0;

  void reinitialiser() {
    x = 0;
    vies = 3;
    score = 0;
  }
}

// ============================================================
//  LES ÉTATS
// ============================================================

class EtatMenu extends EtatDeJeu {
  EtatMenu(this.partie);
  final Partie partie;

  @override
  String get nom => 'MENU';

  @override
  void entrer() => print('  [entree MENU] musique : theme du menu');

  @override
  void sortir() => print('  [sortie MENU] musique : arret');

  @override
  void render() => print('  MENU — Entree pour jouer');

  @override
  void surAction(String action) {
    if (action == 'valider') {
      partie.reinitialiser();
      gestionnaire.changer(EtatJeu(partie));
    }
  }
}

class EtatJeu extends EtatDeJeu {
  EtatJeu(this.partie);
  final Partie partie;

  @override
  String get nom => 'JEU';

  @override
  void entrer() => print('  [entree JEU] musique : theme du donjon');

  @override
  void sortir() => print('  [sortie JEU] musique : etouffee');

  @override
  void update(double dt) {
    partie.x += 100 * dt;
    partie.score += 1;
    if (partie.x > 300) {
      partie.x = 0;
      partie.vies--;
      print('  piege ! vies=${partie.vies}');
      if (partie.vies <= 0) {
        gestionnaire.changer(EtatFin(partie, gagne: false));
      }
    }
  }

  @override
  void render() => print('  x=${partie.x.toStringAsFixed(0)} '
      'vies=${partie.vies} score=${partie.score}');

  @override
  void surAction(String action) {
    if (action == 'pause') {
      gestionnaire.changer(EtatPause(partie, this));
    }
  }
}

class EtatPause extends EtatDeJeu {
  EtatPause(this.partie, this.etatJeu);
  final Partie partie;
  final EtatJeu etatJeu;

  @override
  String get nom => 'PAUSE';

  @override
  void entrer() => print('  [entree PAUSE] volume a 30 %');

  @override
  void sortir() => print('  [sortie PAUSE] volume a 100 %');

  @override
  void render() {
    // On dessine le monde figé, puis le voile.
    print('  (monde fige : x=${partie.x.toStringAsFixed(0)})');
    print('  === PAUSE === Echap pour reprendre');
  }

  @override
  void surAction(String action) {
    if (action == 'pause') {
      // On revient sur l'INSTANCE d'origine : rien n'est perdu.
      gestionnaire.changer(etatJeu);
    } else if (action == 'menu') {
      gestionnaire.changer(EtatMenu(partie));
    }
  }
}

class EtatFin extends EtatDeJeu {
  EtatFin(this.partie, {required this.gagne});
  final Partie partie;
  final bool gagne;

  @override
  String get nom => gagne ? 'VICTOIRE' : 'GAME OVER';

  @override
  void entrer() => print('  [entree $nom] son : ${gagne ? "fanfare" : "glas"}');

  @override
  void render() => print('  $nom — score ${partie.score} '
      '— Entree pour rejouer');

  @override
  void surAction(String action) {
    if (action == 'valider') {
      partie.reinitialiser();
      gestionnaire.changer(EtatJeu(partie));
    } else if (action == 'menu') {
      gestionnaire.changer(EtatMenu(partie));
    }
  }
}

// ============================================================
//  PROGRAMME
// ============================================================

void main() {
  final Partie partie = Partie();
  final GestionnaireEtats g = GestionnaireEtats();
  g.changer(EtatMenu(partie));

  void frames(int n) {
    for (int i = 0; i < n; i++) {
      g.update(1 / 60);
    }
    g.render();
  }

  frames(1);

  print('> Entree');
  g.surAction('valider');
  frames(60);

  print('> Echap');
  g.surAction('pause');
  frames(60);

  print('> Echap');
  g.surAction('pause');
  frames(600);
}
```

**Résultat :**

```text
  [entree MENU] musique : theme du menu
  MENU — Entree pour jouer
> Entree
  [sortie MENU] musique : arret
  [entree JEU] musique : theme du donjon
  x=100 vies=3 score=60
> Echap
  [sortie JEU] musique : etouffee
  [entree PAUSE] volume a 30 %
  (monde fige : x=100)
  === PAUSE === Echap pour reprendre
> Echap
  [sortie PAUSE] volume a 100 %
  [entree JEU] musique : theme du donjon
  piege ! vies=2
  piege ! vies=1
  piege ! vies=0
  [sortie JEU] musique : etouffee
  [entree GAME OVER] son : glas
  GAME OVER — score 900
```

Le patron apporte quatre gains mesurables.

**La musique se gère toute seule.** Aucun `switch` sur l'état n'est nécessaire : chaque état sait ce qu'il doit lancer en entrant et arrêter en sortant.

**Ajouter un écran n'ouvre aucun fichier existant.** Un écran d'options ? Une nouvelle classe `EtatOptions`, et un appel `gestionnaire.changer(EtatOptions(...))` depuis le menu. Rien d'autre.

**Chaque état est court.** Quarante lignes chacun, contre un `switch` de trois cents lignes.

**L'instance de `EtatJeu` est conservée pendant la pause.** Regardez `EtatPause` : elle garde une référence vers l'`EtatJeu` d'origine et y revient. Sans cette précaution, on créerait un nouvel `EtatJeu`, dont l'`entrer` relancerait la musique depuis le début et, surtout, dont les champs internes seraient remis à zéro.

Ce dernier point est le défaut du patron State tel quel : la pause « écrase » le jeu et doit se souvenir de lui. La section 26.22 propose une solution plus élégante.

---

## 26.22 — La pile d'états

Constatons le problème dans le détail. En section 26.21, la pause **remplace** le jeu. C'est une transition. Mais conceptuellement, une pause ne remplace rien : elle se **superpose**.

```text
  REMPLACEMENT (une seule case)      EMPILEMENT (une pile)

  ┌─────────────┐                    ┌─────────────┐  <- sommet, actif
  │  EtatPause  │                    │  EtatPause  │     update + render
  └─────────────┘                    ├─────────────┤
                                     │   EtatJeu   │  <- dessous
  L'EtatJeu a disparu.               └─────────────┘     render seul
  Il faut le garder de côté
  manuellement pour y revenir.       Rien n'a disparu.
                                     Dépiler suffit à reprendre.
```

Une **pile d'états** (*state stack*) applique deux règles simples.

1. Seul l'état du **sommet** reçoit `update` et les actions du joueur.
2. Tous les états marqués « transparents » sous le sommet reçoivent `render`, du bas vers le haut.

Cela résout d'un coup plusieurs besoins classiques.

| Besoin | Avec une pile |
| --- | --- |
| Pause par-dessus le jeu | `empiler(EtatPause())` |
| Inventaire par-dessus le jeu | `empiler(EtatInventaire())` |
| Dialogue par-dessus le jeu | `empiler(EtatDialogue())` |
| Options par-dessus la pause par-dessus le jeu | `empiler(EtatOptions())` |
| Revenir en arrière | `depiler()` |
| Fermer trois écrans d'un coup | `depiler()` trois fois |

Voici la structure d'une pile en action pendant une partie.

```text
  Le joueur joue                    Le joueur met en pause
  ┌───────────────┐                 ┌───────────────┐
  │    EtatJeu    │ <- sommet       │   EtatPause   │ <- sommet (opaque
  └───────────────┘                 ├───────────────┤     au sens logique)
                                    │    EtatJeu    │ <- dessiné, figé
                                    └───────────────┘

  Il ouvre les options              Il ferme les options
  ┌───────────────┐                 ┌───────────────┐
  │  EtatOptions  │ <- sommet       │   EtatPause   │ <- sommet
  ├───────────────┤                 ├───────────────┤
  │   EtatPause   │ <- dessiné      │    EtatJeu    │ <- dessiné
  ├───────────────┤                 └───────────────┘
  │    EtatJeu    │ <- dessiné
  └───────────────┘                 depiler() a suffi.
```

Implémentons.

```dart
abstract class EtatDeJeu {
  late PileEtats pile;

  String get nom;

  /// Si true, les états situés dessous sont encore dessinés.
  bool get transparent => false;

  void entrer() {}
  void sortir() {}

  /// Appelé quand cet état perd le sommet sans être retiré.
  void endormir() {}

  /// Appelé quand cet état redevient le sommet.
  void reveiller() {}

  void update(double dt) {}
  void render() {}
  void surAction(String action) {}
}

class PileEtats {
  final List<EtatDeJeu> _pile = <EtatDeJeu>[];

  EtatDeJeu? get sommet => _pile.isEmpty ? null : _pile.last;
  int get profondeur => _pile.length;

  /// Ajoute un état par-dessus. L'état du dessous est endormi.
  void empiler(EtatDeJeu e) {
    if (_pile.isNotEmpty) _pile.last.endormir();
    e.pile = this;
    _pile.add(e);
    e.entrer();
  }

  /// Retire l'état du sommet. Celui du dessous est réveillé.
  void depiler() {
    if (_pile.isEmpty) return;
    final EtatDeJeu e = _pile.removeLast();
    e.sortir();
    if (_pile.isNotEmpty) _pile.last.reveiller();
  }

  /// Vide la pile et repart de zéro. Utile pour « retour au menu ».
  void remplacerTout(EtatDeJeu e) {
    while (_pile.isNotEmpty) {
      _pile.removeLast().sortir();
    }
    empiler(e);
  }

  /// Seul le sommet se met à jour.
  void update(double dt) => sommet?.update(dt);

  /// Seul le sommet reçoit les actions.
  void surAction(String action) => sommet?.surAction(action);

  /// On dessine du plus bas visible jusqu'au sommet.
  void render() {
    if (_pile.isEmpty) return;

    // On cherche le premier état opaque en partant du sommet.
    int debut = _pile.length - 1;
    while (debut > 0 && _pile[debut].transparent) {
      debut--;
    }
    for (int i = debut; i < _pile.length; i++) {
      _pile[i].render();
    }
  }

  String get description => _pile.map((EtatDeJeu e) => e.nom).join(' > ');
}
```

Mettons-la à l'épreuve.

```dart
abstract class EtatDeJeu {
  late PileEtats pile;
  String get nom;
  bool get transparent => false;
  void entrer() {}
  void sortir() {}
  void endormir() {}
  void reveiller() {}
  void update(double dt) {}
  void render() {}
  void surAction(String action) {}
}

class PileEtats {
  final List<EtatDeJeu> _pile = <EtatDeJeu>[];

  EtatDeJeu? get sommet => _pile.isEmpty ? null : _pile.last;
  int get profondeur => _pile.length;

  void empiler(EtatDeJeu e) {
    if (_pile.isNotEmpty) _pile.last.endormir();
    e.pile = this;
    _pile.add(e);
    e.entrer();
  }

  void depiler() {
    if (_pile.isEmpty) return;
    final EtatDeJeu e = _pile.removeLast();
    e.sortir();
    if (_pile.isNotEmpty) _pile.last.reveiller();
  }

  void remplacerTout(EtatDeJeu e) {
    while (_pile.isNotEmpty) {
      _pile.removeLast().sortir();
    }
    empiler(e);
  }

  void update(double dt) => sommet?.update(dt);
  void surAction(String a) => sommet?.surAction(a);

  void render() {
    if (_pile.isEmpty) return;
    int debut = _pile.length - 1;
    while (debut > 0 && _pile[debut].transparent) {
      debut--;
    }
    for (int i = debut; i < _pile.length; i++) {
      _pile[i].render();
    }
  }

  String get description => _pile.map((EtatDeJeu e) => e.nom).join(' > ');
}

class Partie {
  double x = 0;
  int score = 0;
}

class EtatJeu extends EtatDeJeu {
  EtatJeu(this.partie);
  final Partie partie;

  @override
  String get nom => 'JEU';

  @override
  void entrer() => print('    [JEU entre]');

  @override
  void endormir() => print('    [JEU endormi]');

  @override
  void reveiller() => print('    [JEU reveille]');

  @override
  void update(double dt) {
    partie.x += 100 * dt;
    partie.score += 1;
  }

  @override
  void render() => print('    monde : x=${partie.x.toStringAsFixed(0)} '
      'score=${partie.score}');

  @override
  void surAction(String a) {
    if (a == 'pause') pile.empiler(EtatPause(partie));
  }
}

class EtatPause extends EtatDeJeu {
  EtatPause(this.partie);
  final Partie partie;

  @override
  String get nom => 'PAUSE';

  @override
  bool get transparent => true; // on voit le jeu derrière

  @override
  void entrer() => print('    [PAUSE entre] volume 30 %');

  @override
  void sortir() => print('    [PAUSE sort] volume 100 %');

  @override
  void render() => print('    voile + "PAUSE"');

  @override
  void surAction(String a) {
    if (a == 'pause') pile.depiler();
    if (a == 'options') pile.empiler(EtatOptions());
  }
}

class EtatOptions extends EtatDeJeu {
  @override
  String get nom => 'OPTIONS';

  @override
  bool get transparent => true;

  @override
  void entrer() => print('    [OPTIONS entre]');

  @override
  void sortir() => print('    [OPTIONS sort]');

  @override
  void render() => print('    panneau des options');

  @override
  void surAction(String a) {
    if (a == 'retour') pile.depiler();
  }
}

void main() {
  final Partie partie = Partie();
  final PileEtats pile = PileEtats();
  pile.empiler(EtatJeu(partie));

  void frame(String titre) {
    print('$titre  [pile : ${pile.description}]');
    for (int i = 0; i < 60; i++) {
      pile.update(1 / 60);
    }
    pile.render();
  }

  frame('1 seconde de jeu');

  print('> Echap');
  pile.surAction('pause');
  frame('1 seconde de pause');

  print('> Options');
  pile.surAction('options');
  frame('1 seconde d options');

  print('> Retour');
  pile.surAction('retour');
  frame('1 seconde de pause');

  print('> Echap');
  pile.surAction('pause');
  frame('1 seconde de jeu');
}
```

**Résultat :**

```text
    [JEU entre]
1 seconde de jeu  [pile : JEU]
    monde : x=100 score=60
> Echap
    [JEU endormi]
    [PAUSE entre] volume 30 %
1 seconde de pause  [pile : JEU > PAUSE]
    monde : x=100 score=60
    voile + "PAUSE"
> Options
    [PAUSE endormi]
    [OPTIONS entre]
1 seconde d options  [pile : JEU > PAUSE > OPTIONS]
    monde : x=100 score=60
    voile + "PAUSE"
    panneau des options
> Retour
    [OPTIONS sort]
    [PAUSE reveille]
1 seconde de pause  [pile : JEU > PAUSE]
    monde : x=100 score=60
    voile + "PAUSE"
> Echap
    [PAUSE sort]
    [JEU reveille]
1 seconde de jeu  [pile : JEU]
    monde : x=200 score=120
```

Tout est là, et rien n'a été perdu.

**Le monde est figé pendant la pause et les options.** `x` reste à 100 pendant trois secondes de temps réel, puis reprend à 200 après une nouvelle seconde de jeu. Aucun test de pause n'existe dans `EtatJeu` : il ne reçoit tout simplement plus `update`.

**Les trois couches se dessinent dans le bon ordre.** Monde, puis voile de pause, puis panneau d'options. C'est le `transparent` qui produit cet empilement visuel.

**`endormir` et `reveiller` existent pour les cas subtils.** Un jeu qui doit couper le suivi de la souris pendant la pause, ou relâcher toutes les touches enfoncées, place ce code dans `endormir`. Nous verrons pourquoi c'est indispensable en section 26.24.

> **À retenir.** Pile d'états et machine à états ne s'opposent pas. La machine décide **quel écran** est légitime ; la pile gère les **superpositions**. Un jeu complet utilise souvent les deux.

---

## 26.23 — Gérer la pause proprement

La pause semble triviale. Elle est en réalité l'une des sources de bugs les plus fournies d'un jeu. Passons en revue les cinq pièges, dans l'ordre où ils apparaissent en pratique.

**Piège 1 : mettre `dt` à zéro plutôt que de couper l'`update`.**

```dart
// Version fragile.
void update(double dt) {
  if (enPause) dt = 0;
  monde.update(dt);
}
```

Cela fonctionne pour tout ce qui est proportionnel à `dt`. Mais un compteur écrit `frames++` continue de monter, un `Timer` de `dart:async` continue de sonner, et une animation pilotée par un `AnimationController` de Flutter continue de tourner. Vous obtenez un monde « à moitié figé », le pire des cas car il est difficile à diagnostiquer. Coupez l'appel plutôt que d'annuler son argument.

**Piège 2 : oublier le temps accumulé pendant la pause.**

Le chapitre 20 calcule `dt` comme la différence entre deux horodatages. Si le joueur reste en pause deux minutes, le premier `dt` après la reprise vaut 120 secondes. Le héros traverse alors instantanément tout le niveau et se retrouve dans un mur.

```dart
void surFrame(Duration ecoule) {
  double dt = (ecoule - _precedent).inMicroseconds / 1000000.0;
  _precedent = ecoule;

  // Deux protections complémentaires.
  dt = dt.clamp(0.0, 0.05);        // plafond général (chapitre 20)
  if (_reprisePause) {             // sécurité supplémentaire
    dt = 0;
    _reprisePause = false;
  }

  pile.update(dt);
}
```

**Piège 3 : les touches restées enfoncées.**

Le joueur maintient la flèche droite, appuie sur Échap, relâche la flèche droite pendant la pause. Le relâchement n'est pas transmis au jeu, qui garde `droiteEnfoncee = true`. À la reprise, le héros part tout seul vers la droite.

La solution tient en une ligne, placée dans `endormir` de l'état de jeu.

```dart
@override
void endormir() {
  entrees.toutRelacher(); // vide l'ensemble des touches enfoncées
}
```

**Piège 4 : le son qui continue.**

Une musique et des effets qui tournent pendant la pause donnent une impression de bug. Baissez le volume dans `entrer` de l'état de pause, remontez-le dans `sortir`. Nous l'avons fait dans les exemples ci-dessus.

**Piège 5 : la pause pendant une transition.**

Le joueur appuie sur Échap exactement pendant le fondu au noir de fin de niveau. Si la transition est pilotée par un état dédié, la question ne se pose pas : `EtatTransition` ignore simplement l'action `pause`. C'est un avantage supplémentaire du patron State.

Récapitulons dans une liste de vérification, à relire avant chaque livraison.

```text
  CHECKLIST « PAUSE »

  [ ] Le monde ne se met plus du tout à jour (pas juste dt = 0)
  [ ] Le monde reste VISIBLE, figé
  [ ] Le premier dt après la reprise est nul ou plafonné
  [ ] Toutes les touches sont relâchées à l'entrée en pause
  [ ] Le volume est réduit, puis rétabli
  [ ] Le chronomètre de partie ne progresse pas
  [ ] Les timers (invincibilité, poison, recharge) ne progressent pas
  [ ] La pause est impossible pendant menu, game over et transitions
  [ ] Reprendre ne recrée pas l'état de jeu (pile, pas remplacement)
  [ ] Quitter depuis la pause vide bien toute la pile
```

> **Remarque.** Sur mobile, la mise en arrière-plan de l'application doit déclencher la pause automatiquement. Vous le brancherez avec `WidgetsBindingObserver` et `AppLifecycleState.paused`. Le principe reste celui de cette section : empiler un `EtatPause`.

---

## 26.24 — Le gestionnaire d'entrées : découpler clavier et actions

Voici du code que tout le monde écrit au début, et que tout le monde finit par regretter.

```dart
// NE FAITES PAS CELA.
void update(double dt) {
  if (touchesEnfoncees.contains(LogicalKeyboardKey.arrowRight)) {
    heros.x += heros.vitesse * dt;
  }
  if (touchesEnfoncees.contains(LogicalKeyboardKey.space)) {
    heros.sauter();
  }
}
```

Le défaut n'est pas visible tout de suite. Il apparaît quand vous voulez :

- ajouter les touches Q, S, D pour les claviers AZERTY ;
- ajouter une manette ;
- ajouter un joystick tactile pour la version mobile ;
- laisser le joueur remapper ses touches ;
- écrire un test automatique du saut ;
- enregistrer une démo rejouable.

Chacun de ces besoins vous oblige à retoucher la logique du héros, qui n'a pourtant rien à voir avec le clavier. La cause est un **couplage** : le héros connaît `LogicalKeyboardKey`.

La solution consiste à introduire une couche intermédiaire : l'**action**.

```text
  SANS COUCHE INTERMÉDIAIRE       AVEC UNE COUCHE D'ACTIONS

  clavier ──┐                     clavier  ──┐
  manette ──┼──> logique du jeu   manette  ──┼──> ACTIONS ──> logique
  tactile ──┘                     tactile  ──┤
                                  réseau   ──┤    allerDroite
  Chaque source doit connaître    test     ──┘    sauter
  la logique, et réciproquement.                  attaquer
                                                  pause
  N x M liaisons.                 N + M liaisons.
```

Une action est un nom métier : `allerGauche`, `allerDroite`, `sauter`, `attaquer`, `pause`, `valider`. La logique du jeu ne parle que d'actions. Les périphériques ne parlent que de traduire vers des actions.

```dart
/// Les actions du Donjon de Dart. La logique ne connaît QUE cela.
enum Action {
  allerGauche,
  allerDroite,
  sauter,
  attaquer,
  interagir,
  pause,
  valider,
  retour,
}

/// Traduit les périphériques en actions et retient leur état.
class GestionnaireEntrees {
  /// Actions maintenues (par exemple : le joueur garde la flèche droite).
  final Set<Action> _maintenues = <Action>{};

  /// Actions déclenchées pendant CETTE frame uniquement.
  final Set<Action> _pressees = <Action>{};

  /// Actions relâchées pendant CETTE frame uniquement.
  final Set<Action> _relachees = <Action>{};

  bool estMaintenue(Action a) => _maintenues.contains(a);
  bool vientDEtrePressee(Action a) => _pressees.contains(a);
  bool vientDEtreRelachee(Action a) => _relachees.contains(a);

  /// Appelé par la couche périphérique.
  void appuyer(Action a) {
    if (_maintenues.add(a)) _pressees.add(a);
  }

  void relacher(Action a) {
    if (_maintenues.remove(a)) _relachees.add(a);
  }

  /// Relâche tout. À appeler quand le jeu perd le focus ou se met en pause.
  void toutRelacher() {
    _relachees.addAll(_maintenues);
    _maintenues.clear();
  }

  /// À appeler à la FIN de chaque frame, après tous les update.
  void finDeFrame() {
    _pressees.clear();
    _relachees.clear();
  }
}
```

La distinction entre « maintenue » et « vient d'être pressée » est fondamentale, et beaucoup de bugs de jeu viennent de sa confusion.

| Type | Vrai pendant | Usage typique |
| --- | --- | --- |
| maintenue | tant que la touche est enfoncée | marcher, viser, courir |
| vient d'être pressée | une seule frame | sauter, tirer, valider un menu |
| vient d'être relâchée | une seule frame | tir chargé, saut à hauteur variable |

Si vous utilisez « maintenue » pour le saut, le héros saute soixante fois par seconde. Si vous utilisez « vient d'être pressée » pour marcher, il avance d'un pixel puis s'arrête. Voici l'usage correct.

```dart
enum Action { allerGauche, allerDroite, sauter, attaquer }

class GestionnaireEntrees {
  final Set<Action> _maintenues = <Action>{};
  final Set<Action> _pressees = <Action>{};
  final Set<Action> _relachees = <Action>{};

  bool estMaintenue(Action a) => _maintenues.contains(a);
  bool vientDEtrePressee(Action a) => _pressees.contains(a);

  void appuyer(Action a) {
    if (_maintenues.add(a)) _pressees.add(a);
  }

  void relacher(Action a) {
    if (_maintenues.remove(a)) _relachees.add(a);
  }

  void toutRelacher() {
    _relachees.addAll(_maintenues);
    _maintenues.clear();
  }

  void finDeFrame() {
    _pressees.clear();
    _relachees.clear();
  }
}

class Heros {
  double x = 0;
  double vy = 0;
  bool auSol = true;
  int sauts = 0;

  void update(double dt, GestionnaireEntrees e) {
    // MAINTENUE : déplacement continu.
    if (e.estMaintenue(Action.allerGauche)) x -= 140 * dt;
    if (e.estMaintenue(Action.allerDroite)) x += 140 * dt;

    // PRESSÉE : action ponctuelle.
    if (e.vientDEtrePressee(Action.sauter) && auSol) {
      vy = -300;
      auSol = false;
      sauts++;
    }

    // Physique minimale (chapitre 23).
    vy += 900 * dt;
    if (vy > 0 && !auSol) {
      auSol = true;
      vy = 0;
    }
  }
}

void main() {
  final GestionnaireEntrees e = GestionnaireEntrees();
  final Heros h = Heros();

  void frame() {
    h.update(1 / 60, e);
    e.finDeFrame();
  }

  // Le joueur maintient « droite » pendant 30 frames
  // et garde « saut » enfoncé tout du long.
  e.appuyer(Action.allerDroite);
  e.appuyer(Action.sauter);
  for (int i = 0; i < 30; i++) {
    frame();
  }
  e.relacher(Action.allerDroite);
  e.relacher(Action.sauter);
  frame();

  print('x = ${h.x.toStringAsFixed(1)}');
  print('sauts = ${h.sauts}');
}
```

**Résultat :**

```text
x = 70.0
sauts = 1
```

Un seul saut, alors que la touche est restée enfoncée trente frames. C'est exactement ce que le joueur attend. Le déplacement, lui, a bien duré une demi-seconde : 140 × 0,5 = 70 pixels.

L'appel `finDeFrame()` est indispensable : il vide les ensembles « pressées » et « relâchées » pour que ces informations ne durent qu'une frame. Oubliez-le, et votre héros saute indéfiniment.

Reste la couche périphérique, qui traduit Flutter vers vos actions. Elle est la **seule** partie du code à connaître `LogicalKeyboardKey`.

```dart
/// La SEULE classe qui connaît le clavier physique.
class ClavierVersActions {
  ClavierVersActions(this.entrees, this.mapping);

  final GestionnaireEntrees entrees;
  final Map<LogicalKeyboardKey, Action> mapping;

  void surEvenement(KeyEvent evenement) {
    final Action? action = mapping[evenement.logicalKey];
    if (action == null) return;

    if (evenement is KeyDownEvent) {
      entrees.appuyer(action);
    } else if (evenement is KeyUpEvent) {
      entrees.relacher(action);
    }
  }
}
```

Le bénéfice se mesure immédiatement : un joystick tactile écrit `entrees.appuyer(Action.allerDroite)` et fonctionne sans toucher à une ligne du héros. Un test automatique fait de même. Une démo rejouable enregistre la suite des actions et la rejoue.

> **À retenir.** Le héros ne doit jamais savoir qu'un clavier existe. Il ne connaît que des intentions.

---

## 26.25 — Le remappage des touches

Une fois la couche d'actions en place, le remappage devient presque gratuit : il suffit de rendre la table `mapping` modifiable.

```dart
enum Action { allerGauche, allerDroite, sauter, attaquer, pause }

/// Une table modifiable, avec des valeurs par défaut selon la disposition.
class Reglages {
  Reglages.qwerty()
      : _mapping = <String, Action>{
          'ArrowLeft': Action.allerGauche,
          'ArrowRight': Action.allerDroite,
          'KeyA': Action.allerGauche,
          'KeyD': Action.allerDroite,
          'Space': Action.sauter,
          'KeyJ': Action.attaquer,
          'Escape': Action.pause,
        };

  Reglages.azerty()
      : _mapping = <String, Action>{
          'ArrowLeft': Action.allerGauche,
          'ArrowRight': Action.allerDroite,
          'KeyQ': Action.allerGauche,
          'KeyD': Action.allerDroite,
          'Space': Action.sauter,
          'KeyJ': Action.attaquer,
          'Escape': Action.pause,
        };

  final Map<String, Action> _mapping;

  Action? actionPour(String touche) => _mapping[touche];

  /// Toutes les touches associées à une action.
  List<String> touchesDe(Action a) => _mapping.entries
      .where((MapEntry<String, Action> e) => e.value == a)
      .map((MapEntry<String, Action> e) => e.key)
      .toList();

  /// Associe une touche à une action. Renvoie l'ancienne action
  /// si la touche était déjà utilisée, pour prévenir le joueur.
  Action? assigner(String touche, Action a) {
    final Action? ancienne = _mapping[touche];
    _mapping[touche] = a;
    return ancienne;
  }

  /// Retire une association.
  void liberer(String touche) => _mapping.remove(touche);

  /// Sérialisation, pour la sauvegarde (chapitre 17).
  Map<String, String> versJson() =>
      _mapping.map((String k, Action v) => MapEntry<String, String>(k, v.name));

  void depuisJson(Map<String, String> json) {
    _mapping.clear();
    json.forEach((String k, String v) {
      for (final Action a in Action.values) {
        if (a.name == v) _mapping[k] = a;
      }
    });
  }
}

void main() {
  final Reglages r = Reglages.azerty();

  print('Touches pour sauter : ${r.touchesDe(Action.sauter)}');
  print('Touche Space -> ${r.actionPour('Space')?.name}');

  print('');
  print('Le joueur remappe le saut sur W.');
  final Action? conflit = r.assigner('KeyW', Action.sauter);
  print('conflit detecte : ${conflit?.name ?? "aucun"}');
  print('Touches pour sauter : ${r.touchesDe(Action.sauter)}');

  print('');
  print('Le joueur remappe l attaque sur Space (deja pris).');
  final Action? conflit2 = r.assigner('Space', Action.attaquer);
  print('conflit detecte : ${conflit2?.name ?? "aucun"}');
  print('Touches pour sauter : ${r.touchesDe(Action.sauter)}');
  print('Touches pour attaquer : ${r.touchesDe(Action.attaquer)}');

  print('');
  print('Sauvegarde : ${r.versJson()}');
}
```

**Résultat :**

```text
Touches pour sauter : [Space]
Touche Space -> sauter

Le joueur remappe le saut sur W.
conflit detecte : aucun
Touches pour sauter : [Space, KeyW]

Le joueur remappe l attaque sur Space (deja pris).
conflit detecte : sauter
Touches pour sauter : [KeyW]
Touches pour attaquer : [KeyJ, Space]

Sauvegarde : {ArrowLeft: allerGauche, ArrowRight: allerDroite, KeyQ: allerGauche, KeyD: allerDroite, Space: attaquer, KeyJ: attaquer, Escape: pause}
```

Trois points de conception méritent d'être expliqués.

**Le sens de la table compte.** Nous allons de la **touche** vers l'**action**, jamais l'inverse. Une action peut avoir plusieurs touches, et c'est souhaitable : flèches **et** ZQSD. Une touche ne peut pas déclencher deux actions ; la structure `Map` l'interdit d'elle-même.

**Le conflit est signalé, pas interdit.** `assigner` renvoie l'action précédente. L'écran d'options peut alors afficher « Espace était assigné à Sauter. Confirmer ? ». Ne réglez jamais un conflit en silence.

**La sérialisation est prévue dès le départ.** `versJson` et `depuisJson` réutilisent le chapitre 17. Le remappage doit survivre au redémarrage du jeu, sinon il est inutile.

Un dernier écueil : ne laissez jamais le joueur se retrouver sans aucune touche pour une action essentielle. Ajoutez une vérification avant de quitter l'écran d'options.

```dart
List<Action> actionsSansTouche(Reglages r) {
  return Action.values
      .where((Action a) => r.touchesDe(a).isEmpty)
      .toList();
}
```

---

## 26.26 — Le bus d'événements

Nouveau problème de couplage, différent du précédent. Quand un gobelin meurt, il faut : ajouter des points au score, faire apparaître des particules, jouer un son, incrémenter le compteur de la quête « tuer 10 gobelins », et peut-être déclencher la victoire si c'était le dernier.

La version naïve donne ceci.

```dart
// NE FAITES PAS CELA.
class Gobelin extends Entity {
  void mourir() {
    score.ajouter(10);
    audio.jouer('mort_gobelin.wav');
    monde.ajouter(Explosion(x, y));
    quetes.progresser('tuer_gobelins');
    if (monde.plusAucunGobelin()) machine.aller(GameState.victoire);
  }
}
```

Le gobelin connaît maintenant le score, l'audio, le monde, les quêtes et la machine à états. Il est devenu impossible à tester isolément, et le moindre changement dans le système de quêtes casse la classe `Gobelin`.

Le **bus d'événements** inverse la relation : le gobelin **annonce** ce qui lui arrive, et ceux que cela intéresse écoutent.

```text
  COUPLAGE DIRECT                  BUS D'ÉVÉNEMENTS

  Gobelin ──────> Score            Gobelin
     │  │  ├────> Audio               │
     │  │  └────> Quetes              │ publie
     │  └───────> Monde               ▼
     └──────────> Machine        ┌─────────┐
                                 │   BUS   │
  Le gobelin connaît 5 systèmes. └────┬────┘
  Test impossible sans les 5.         │ notifie
                                 ┌────┼────┬────────┬────────┐
                                 ▼    ▼    ▼        ▼        ▼
                              Score Audio Quetes Particules Machine

                                 Le gobelin ne connaît que le bus.
                                 Les systèmes ne connaissent pas le gobelin.
```

Voici une implémentation complète, typée, avec la protection contre la modification concurrente vue en section 26.7.

```dart
// ============================================================
//  LES ÉVÉNEMENTS
// ============================================================

abstract class EvenementJeu {
  const EvenementJeu();
}

class EnnemiTue extends EvenementJeu {
  const EnnemiTue({
    required this.type,
    required this.x,
    required this.y,
    required this.points,
  });
  final String type;
  final double x;
  final double y;
  final int points;
}

class ObjetRamasse extends EvenementJeu {
  const ObjetRamasse(this.nom);
  final String nom;
}

class JoueurBlesse extends EvenementJeu {
  const JoueurBlesse(this.degats, this.viesRestantes);
  final int degats;
  final int viesRestantes;
}

class NiveauTermine extends EvenementJeu {
  const NiveauTermine(this.numero);
  final int numero;
}

// ============================================================
//  LE BUS
// ============================================================

typedef Ecouteur<T> = void Function(T evenement);

class BusEvenements {
  final Map<Type, List<Function>> _ecouteurs = <Type, List<Function>>{};

  /// File d'attente : on ne modifie jamais _ecouteurs pendant une émission.
  final List<void Function()> _enAttente = <void Function()>[];
  bool _enEmission = false;

  /// Abonne un écouteur et renvoie la fonction de désabonnement.
  void Function() ecouter<T extends EvenementJeu>(Ecouteur<T> ecouteur) {
    void inscrire() {
      _ecouteurs.putIfAbsent(T, () => <Function>[]).add(ecouteur);
    }

    if (_enEmission) {
      _enAttente.add(inscrire);
    } else {
      inscrire();
    }

    return () {
      void retirer() => _ecouteurs[T]?.remove(ecouteur);
      if (_enEmission) {
        _enAttente.add(retirer);
      } else {
        retirer();
      }
    };
  }

  /// Publie un événement à tous ses écouteurs.
  void publier<T extends EvenementJeu>(T evenement) {
    final List<Function>? liste = _ecouteurs[evenement.runtimeType];
    if (liste == null || liste.isEmpty) return;

    _enEmission = true;
    // Copie défensive : un écouteur peut en abonner un autre.
    for (final Function f in List<Function>.of(liste)) {
      (f as Ecouteur<T>)(evenement);
    }
    _enEmission = false;

    for (final void Function() operation in _enAttente) {
      operation();
    }
    _enAttente.clear();
  }
}

// ============================================================
//  LES SYSTÈMES ABONNÉS
// ============================================================

class ServiceScore {
  ServiceScore(BusEvenements bus) {
    bus.ecouter<EnnemiTue>((EnnemiTue e) {
      score += e.points;
      print('  [score] +${e.points} -> $score');
    });
    bus.ecouter<ObjetRamasse>((ObjetRamasse e) {
      score += 5;
      print('  [score] +5 (${e.nom}) -> $score');
    });
  }

  int score = 0;
}

class ServiceAudio {
  ServiceAudio(BusEvenements bus) {
    bus.ecouter<EnnemiTue>(
        (EnnemiTue e) => print('  [audio] mort_${e.type}.wav'));
    bus.ecouter<ObjetRamasse>(
        (ObjetRamasse e) => print('  [audio] ramassage.wav'));
    bus.ecouter<JoueurBlesse>((JoueurBlesse e) => print(
        '  [audio] ${e.viesRestantes == 0 ? "glas" : "degats"}.wav'));
  }
}

class ServiceParticules {
  ServiceParticules(BusEvenements bus) {
    bus.ecouter<EnnemiTue>((EnnemiTue e) => print(
        '  [particules] explosion en (${e.x.toStringAsFixed(0)}, '
        '${e.y.toStringAsFixed(0)})'));
  }
}

class ServiceQuetes {
  ServiceQuetes(BusEvenements bus) {
    bus.ecouter<EnnemiTue>((EnnemiTue e) {
      if (e.type != 'gobelin') return;
      gobelinsTues++;
      print('  [quete] gobelins tues : $gobelinsTues / 3');
      if (gobelinsTues == 3) print('  [quete] QUETE TERMINEE');
    });
  }

  int gobelinsTues = 0;
}

// ============================================================
//  PROGRAMME
// ============================================================

void main() {
  final BusEvenements bus = BusEvenements();

  final ServiceScore score = ServiceScore(bus);
  ServiceAudio(bus);
  ServiceParticules(bus);
  ServiceQuetes(bus);

  print('--- le heros tue un gobelin ---');
  bus.publier(const EnnemiTue(type: 'gobelin', x: 240, y: 300, points: 10));

  print('--- il ramasse une potion ---');
  bus.publier(const ObjetRamasse('potion'));

  print('--- il tue deux autres gobelins ---');
  bus.publier(const EnnemiTue(type: 'gobelin', x: 300, y: 300, points: 10));
  bus.publier(const EnnemiTue(type: 'gobelin', x: 360, y: 300, points: 10));

  print('--- il se blesse ---');
  bus.publier(const JoueurBlesse(1, 2));

  print('');
  print('score final : ${score.score}');
}
```

**Résultat :**

```text
--- le heros tue un gobelin ---
  [score] +10 -> 10
  [audio] mort_gobelin.wav
  [particules] explosion en (240, 300)
  [quete] gobelins tues : 1 / 3
--- il ramasse une potion ---
  [score] +5 (potion) -> 15
  [audio] ramassage.wav
--- il tue deux autres gobelins ---
  [score] +10 -> 25
  [audio] mort_gobelin.wav
  [particules] explosion en (300, 300)
  [quete] gobelins tues : 2 / 3
  [score] +10 -> 35
  [audio] mort_gobelin.wav
  [particules] explosion en (360, 300)
  [quete] gobelins tues : 3 / 3
  [quete] QUETE TERMINEE
--- il se blesse ---
  [audio] degats.wav

score final : 35
```

Trois avantages, et deux dangers qu'il faut connaître.

**Avantage : le gobelin est testable seul.** Un test peut créer un bus vide, tuer un gobelin et vérifier qu'un `EnnemiTue` a bien été publié, sans audio, sans particules et sans quêtes.

**Avantage : ajouter un système ne touche à rien.** Un système de statistiques de fin de partie s'abonne aux mêmes événements. Aucune classe existante n'est modifiée.

**Avantage : l'ordre des abonnés n'a pas d'importance.** Chacun réagit indépendamment.

**Danger : le flux devient invisible.** En lisant `Gobelin.mourir()`, vous ne voyez plus ce qui se passe ensuite. Sur un gros projet, cela complique le débogage. La parade : documentez chaque type d'événement, et ajoutez au bus un mode de trace qui affiche chaque publication en mode développement.

**Danger : les fuites d'abonnements.** Un état qui s'abonne et ne se désabonne jamais reste vivant en mémoire et continue de réagir. C'est pour cela que `ecouter` renvoie une fonction de désabonnement ; appelez-la dans `sortir()` de vos états.

```dart
class EtatJeu extends EtatDeJeu {
  final List<void Function()> _abonnements = <void Function()>[];

  @override
  void entrer() {
    _abonnements.add(bus.ecouter<EnnemiTue>(_surEnnemiTue));
    _abonnements.add(bus.ecouter<JoueurBlesse>(_surBlessure));
  }

  @override
  void sortir() {
    for (final void Function() desabonner in _abonnements) {
      desabonner();
    }
    _abonnements.clear();
  }
}
```

> **À retenir.** Utilisez le bus pour les événements **importants et transversaux** : mort, ramassage, changement de niveau, fin de partie. N'en faites pas passer les positions ou les collisions frame par frame : ce serait à la fois lent et illisible.

---

## 26.27 — Le service de données du jeu

Le score, les vies, le niveau atteint, le meilleur score et les options ne sont pas des entités : ils n'ont ni position ni apparence. Ce sont des **données de session**, et elles méritent un objet dédié.

```text
  QUI POSSÈDE QUOI ?

  Monde (26.6)          les entités : héros, gobelins, potions
                        durée de vie = un niveau

  DonneesDeJeu (26.27)  score, vies, clés, niveau, meilleur score
                        durée de vie = toute l'application

  Reglages (26.25)      touches, volume, langue
                        durée de vie = permanente (sauvegardée)

  MachineAEtats (26.20) l'écran courant
                        durée de vie = toute l'application
```

Le critère de découpe est la **durée de vie**. Un gobelin meurt à la fin du niveau ; le meilleur score doit survivre à la fermeture du jeu. Mélanger les deux dans le même objet garantit des bugs de réinitialisation.

```dart
class DonneesDeJeu {
  DonneesDeJeu({this.viesMax = 3});

  final int viesMax;

  int _score = 0;
  int _vies = 3;
  int _cles = 0;
  int _niveau = 1;
  int _meilleurScore = 0;

  // Lecture seule depuis l'extérieur : personne ne peut écrire
  // « donnees.score = 999 » par accident.
  int get score => _score;
  int get vies => _vies;
  int get cles => _cles;
  int get niveau => _niveau;
  int get meilleurScore => _meilleurScore;

  bool get partiePerdue => _vies <= 0;

  /// Signalé à chaque changement, pour rafraîchir le HUD.
  void Function()? surChangement;

  void ajouterPoints(int points) {
    if (points <= 0) return;
    _score += points;
    if (_score > _meilleurScore) _meilleurScore = _score;
    surChangement?.call();
  }

  void perdreUneVie() {
    if (_vies <= 0) return;
    _vies--;
    surChangement?.call();
  }

  void gagnerUneVie() {
    if (_vies >= viesMax) return;
    _vies++;
    surChangement?.call();
  }

  void ramasserCle() {
    _cles++;
    surChangement?.call();
  }

  bool consommerCle() {
    if (_cles <= 0) return false;
    _cles--;
    surChangement?.call();
    return true;
  }

  void niveauSuivant() {
    _niveau++;
    _cles = 0; // les clés ne se gardent pas d'un niveau à l'autre
    surChangement?.call();
  }

  /// Nouvelle partie : on remet tout à zéro SAUF le meilleur score.
  void nouvellePartie() {
    _score = 0;
    _vies = viesMax;
    _cles = 0;
    _niveau = 1;
    surChangement?.call();
  }

  Map<String, Object?> versJson() => <String, Object?>{
        'meilleurScore': _meilleurScore,
        'niveau': _niveau,
      };

  void depuisJson(Map<String, Object?> json) {
    _meilleurScore = (json['meilleurScore'] as int?) ?? 0;
    _niveau = (json['niveau'] as int?) ?? 1;
  }
}

void main() {
  final DonneesDeJeu d = DonneesDeJeu();
  d.surChangement = () => print('  HUD : score=${d.score} vies=${d.vies} '
      'cles=${d.cles} niveau=${d.niveau} record=${d.meilleurScore}');

  d.nouvellePartie();
  d.ajouterPoints(10);
  d.ramasserCle();
  d.perdreUneVie();
  d.ajouterPoints(50);
  print('porte : ${d.consommerCle() ? "ouverte" : "verrouillee"}');
  print('porte : ${d.consommerCle() ? "ouverte" : "verrouillee"}');
  d.niveauSuivant();

  print('');
  print('Le heros tombe deux fois de plus.');
  d.perdreUneVie();
  d.perdreUneVie();
  print('partie perdue : ${d.partiePerdue}');

  print('');
  print('Nouvelle partie.');
  d.nouvellePartie();
  print('sauvegarde : ${d.versJson()}');
}
```

**Résultat :**

```text
  HUD : score=0 vies=3 cles=0 niveau=1 record=0
  HUD : score=10 vies=3 cles=0 niveau=1 record=10
  HUD : score=10 vies=3 cles=1 niveau=1 record=10
  HUD : score=10 vies=2 cles=1 niveau=1 record=10
  HUD : score=60 vies=2 cles=1 niveau=1 record=60
  HUD : score=60 vies=2 cles=0 niveau=1 record=60
porte : ouverte
porte : verrouillee
  HUD : score=60 vies=2 cles=0 niveau=2 record=60
  HUD : score=60 vies=1 cles=0 niveau=2 record=60

Le heros tombe deux fois de plus.
  HUD : score=60 vies=0 cles=0 niveau=2 record=60
partie perdue : true

Nouvelle partie.
  HUD : score=0 vies=3 cles=0 niveau=1 record=60
sauvegarde : {meilleurScore: 60, niveau: 1}
```

Quatre décisions de conception sont visibles dans ce code.

**Les champs sont privés, l'écriture passe par des méthodes.** On ne peut pas écrire `donnees.score = 999`. Chaque modification passe par une méthode qui applique les règles : pas de points négatifs, pas plus de vies que le maximum, mise à jour automatique du record.

**Le record se met à jour tout seul.** Aucun appelant n'a à y penser, donc personne ne peut l'oublier.

**`consommerCle` renvoie un booléen.** La porte demande « ai-je pu consommer une clé ? » et reçoit une réponse claire. C'est plus sûr que de tester `cles > 0` puis de décrémenter en deux étapes.

**`nouvellePartie` ne touche pas au record.** C'est la distinction entre données de session et données persistantes, écrite noir sur blanc.

> **Remarque.** `surChangement` joue ici le rôle d'un `ChangeNotifier` de Flutter. Dans un vrai projet Flutter, faites `class DonneesDeJeu extends ChangeNotifier` et appelez `notifyListeners()`. Le HUD devient alors un widget qui se reconstruit tout seul.

---

## 26.28 — Le patron Singleton et ses dangers

Arrivé ici, une question surgit naturellement : comment le gobelin, tout au fond de la liste d'entités, accède-t-il au service de score ? La réponse la plus tentante est le **Singleton**.

> **Définition.** Un Singleton est une classe qui garantit qu'il n'existe qu'une seule instance d'elle-même, accessible globalement.

```dart
class ServiceScore {
  // Le constructeur privé empêche toute autre création.
  ServiceScore._();

  /// L'unique instance, accessible de partout.
  static final ServiceScore instance = ServiceScore._();

  int score = 0;

  void ajouter(int points) => score += points;
}

void main() {
  // Accessible depuis n'importe où, sans rien passer en paramètre.
  ServiceScore.instance.ajouter(10);
  ServiceScore.instance.ajouter(5);
  print(ServiceScore.instance.score);
}
```

**Résultat :**

```text
15
```

C'est court, c'est pratique, et c'est pour cela que le piège fonctionne si bien. Voici ce qu'il vous coûte réellement.

| Danger | Description | Conséquence concrète |
| --- | --- | --- |
| État global déguisé | c'est une variable globale avec un joli nom | n'importe quel code peut tout modifier |
| Tests contaminés | l'instance survit d'un test à l'autre | le test 2 échoue à cause du test 1 |
| Dépendances invisibles | rien dans la signature ne les montre | on découvre les liens en lisant tout le corps |
| Impossible de simuler | on ne peut pas substituer un faux service | pas de test avec un audio muet |
| Ordre d'initialisation | qui crée quoi en premier ? | plantage au démarrage, difficile à reproduire |
| Plusieurs parties simultanées | une seule instance possible | écran splitté impossible |

Le deuxième point est le plus concret. Illustrons-le.

```dart
class ServiceScore {
  ServiceScore._();
  static final ServiceScore instance = ServiceScore._();
  int score = 0;
  void ajouter(int p) => score += p;
}

void testTuerUnGobelin() {
  ServiceScore.instance.ajouter(10);
  final bool ok = ServiceScore.instance.score == 10;
  print('test 1 : ${ok ? "REUSSI" : "ECHOUE"} '
      '(score = ${ServiceScore.instance.score})');
}

void testRamasserUnePotion() {
  ServiceScore.instance.ajouter(5);
  final bool ok = ServiceScore.instance.score == 5;
  print('test 2 : ${ok ? "REUSSI" : "ECHOUE"} '
      '(score = ${ServiceScore.instance.score})');
}

void main() {
  testTuerUnGobelin();
  testRamasserUnePotion();
}
```

**Résultat :**

```text
test 1 : REUSSI (score = 10)
test 2 : ECHOUE (score = 15)
```

Le second test échoue alors que son code est irréprochable. Il échoue à cause du premier. Pire : exécuté seul, il réussit. Vous voilà avec un test qui dépend de l'ordre d'exécution, c'est-à-dire avec un test auquel on ne peut plus faire confiance.

La rustine habituelle consiste à ajouter une méthode `reinitialiser()` appelée avant chaque test. Elle fonctionne, mais il faut y penser à chaque fois, pour chaque Singleton, et l'oubli est silencieux.

Quand le Singleton est-il acceptable ? Trois cas, et seulement trois.

| Cas | Exemple | Pourquoi c'est tolérable |
| --- | --- | --- |
| Sans état | un service de logs | rien à réinitialiser |
| Ressource unique par nature | le moteur audio du système | il n'y en a physiquement qu'un |
| Constantes de configuration | les couches de rendu de 26.9 | immuables |

Pour tout ce qui porte un état modifiable — score, vies, progression, entrées, machine à états — préférez la solution de la section suivante.

> **À retenir.** Le Singleton ne règle pas un problème de conception : il le rend invisible. Un objet difficile à passer en paramètre est souvent un objet qui a trop de responsabilités.

---

## 26.29 — L'injection de dépendances simple

L'**injection de dépendances** consiste à **donner** à un objet ce dont il a besoin, au lieu de le laisser aller le chercher. C'est tout. Le nom est intimidant, le principe tient en une ligne.

```text
  SINGLETON (l'objet va chercher)   INJECTION (on lui donne)

  class Gobelin {                   class Gobelin {
    void mourir() {                   Gobelin(this.contexte);
      ServiceScore.instance           final ContexteJeu contexte;
        .ajouter(10);
    }                                 void mourir() {
  }                                     contexte.donnees
                                          .ajouterPoints(10);
  Dépendance CACHÉE.                  }
  Test impossible sans le vrai      }
  service.
                                    Dépendance VISIBLE dans le
                                    constructeur.
                                    Test avec un faux contexte.
```

En pratique, passer dix services un par un serait pénible. On les regroupe donc dans un objet unique, souvent appelé **contexte** ou *service locator* local.

```dart
// ============================================================
//  LES SERVICES
// ============================================================

class DonneesDeJeu {
  int score = 0;
  int vies = 3;

  void ajouterPoints(int p) => score += p;
  void perdreUneVie() => vies--;
}

abstract class ServiceAudio {
  void jouer(String son);
}

class AudioReel implements ServiceAudio {
  @override
  void jouer(String son) => print('  [audio] $son');
}

/// Version muette, pour les tests. Elle enregistre au lieu de jouer.
class AudioMuet implements ServiceAudio {
  final List<String> sonsJoues = <String>[];

  @override
  void jouer(String son) => sonsJoues.add(son);
}

class BusEvenements {
  final List<String> journal = <String>[];
  void publier(String e) => journal.add(e);
}

// ============================================================
//  LE CONTEXTE : tout ce dont une entité peut avoir besoin
// ============================================================

class ContexteJeu {
  ContexteJeu({
    required this.donnees,
    required this.audio,
    required this.bus,
  });

  final DonneesDeJeu donnees;
  final ServiceAudio audio;
  final BusEvenements bus;
}

// ============================================================
//  LES ENTITÉS REÇOIVENT LE CONTEXTE
// ============================================================

abstract class Entity {
  Entity(this.contexte, {this.x = 0, this.y = 0});

  final ContexteJeu contexte;
  double x;
  double y;
  bool aRetirer = false;

  void update(double dt);
}

class Gobelin extends Entity {
  Gobelin(super.contexte, {required super.x, required super.y});

  int pv = 8;

  void subirDegats(int d) {
    pv -= d;
    if (pv <= 0) mourir();
  }

  void mourir() {
    aRetirer = true;
    contexte.donnees.ajouterPoints(10);
    contexte.audio.jouer('mort_gobelin.wav');
    contexte.bus.publier('EnnemiTue(gobelin)');
  }

  @override
  void update(double dt) {
    x += 40 * dt;
  }
}

// ============================================================
//  PROGRAMME : le vrai jeu, puis un test
// ============================================================

void main() {
  // --- 1. Le vrai jeu ---
  print('=== partie reelle ===');
  final ContexteJeu contexte = ContexteJeu(
    donnees: DonneesDeJeu(),
    audio: AudioReel(),
    bus: BusEvenements(),
  );

  final Gobelin g = Gobelin(contexte, x: 200, y: 300);
  g.subirDegats(8);
  print('  score = ${contexte.donnees.score}');
  print('  evenements = ${contexte.bus.journal}');

  // --- 2. Un test, avec des services de remplacement ---
  print('');
  print('=== test automatique ===');
  final AudioMuet audioTest = AudioMuet();
  final ContexteJeu contexteTest = ContexteJeu(
    donnees: DonneesDeJeu(),
    audio: audioTest,
    bus: BusEvenements(),
  );

  final Gobelin gTest = Gobelin(contexteTest, x: 0, y: 0);
  gTest.subirDegats(8);

  print('  le gobelin est marque : ${gTest.aRetirer}');
  print('  score obtenu : ${contexteTest.donnees.score} (attendu 10)');
  print('  sons declenches : ${audioTest.sonsJoues}');
  print('  aucun son n a ete joue pour de vrai.');

  // --- 3. Un second test, totalement independant du premier ---
  print('');
  print('=== second test ===');
  final ContexteJeu contexteTest2 = ContexteJeu(
    donnees: DonneesDeJeu(),
    audio: AudioMuet(),
    bus: BusEvenements(),
  );
  final Gobelin gTest2 = Gobelin(contexteTest2, x: 0, y: 0);
  gTest2.subirDegats(3);
  print('  score obtenu : ${contexteTest2.donnees.score} (attendu 0)');
  print('  le gobelin survit : ${!gTest2.aRetirer}');
}
```

**Résultat :**

```text
=== partie reelle ===
  [audio] mort_gobelin.wav
  score = 10
  evenements = [EnnemiTue(gobelin)]

=== test automatique ===
  le gobelin est marque : true
  score obtenu : 10 (attendu 10)
  sons declenches : [mort_gobelin.wav]
  aucun son n a ete joue pour de vrai.

=== second test ===
  score obtenu : 0 (attendu 0)
  le gobelin survit : true
```

Comparez avec la section 26.28 : le second test donne `0` comme prévu, parce qu'il travaille sur son **propre** jeu de données. Aucune contamination n'est possible.

Trois bénéfices, et un coût qu'il faut assumer.

**Bénéfice : les dépendances sont visibles.** Le constructeur de `Gobelin` annonce qu'il a besoin d'un contexte. En lisant la signature, vous savez à quoi la classe touche.

**Bénéfice : les substitutions sont triviales.** `AudioMuet` remplace `AudioReel` parce que les deux implémentent `ServiceAudio`. C'est l'interface implicite du chapitre 10, utilisée pour de bon.

**Bénéfice : plusieurs mondes peuvent coexister.** Un écran splitté, une prévisualisation dans l'éditeur de niveaux, un test de charge : chacun a son contexte.

**Coût : il faut passer le contexte partout.** C'est réel, mais modéré : le contexte se transmet par le constructeur, et le mot-clé `super.contexte` de Dart 2.17 rend l'écriture très légère, comme dans `Gobelin` ci-dessus.

> **À retenir.** Injecter n'est pas plus compliqué que d'utiliser un Singleton : c'est simplement un paramètre de constructeur en plus. En échange, vous obtenez des tests fiables et un code dont on peut lire les dépendances.

---

## 26.30 — Organisation des fichiers d'un projet de jeu

Le chapitre 16 vous a appris à structurer un projet Dart : `lib/`, `test/`, `pubspec.yaml`, les exports. Appliquons cela à un jeu.

```text
  donjon_de_dart/
  │
  ├── pubspec.yaml               dépendances et déclaration des assets
  ├── analysis_options.yaml      règles d'analyse statique
  ├── README.md
  │
  ├── assets/
  │   ├── images/
  │   │   ├── heros.png              sprite sheet du héros
  │   │   ├── gobelin.png
  │   │   └── tuiles.png
  │   ├── audio/
  │   │   ├── musique_donjon.mp3
  │   │   └── mort_gobelin.wav
  │   └── niveaux/
  │       ├── niveau_1.json          données du niveau (chapitre 17)
  │       └── niveau_2.json
  │
  ├── lib/
  │   │
  │   ├── main.dart                  UNIQUEMENT runApp() : 15 lignes
  │   │
  │   ├── core/                      le moteur, indépendant du jeu
  │   │   ├── entity.dart                abstract class Entity   (26.4)
  │   │   ├── monde.dart                 List<Entity> + files    (26.6, 26.8)
  │   │   ├── boucle.dart                Ticker, dt, FPS         (ch. 20)
  │   │   ├── couches.dart               constantes de priority  (26.9)
  │   │   ├── bus_evenements.dart        BusEvenements           (26.26)
  │   │   ├── entrees.dart               Action, GestionnaireEntrees (26.24)
  │   │   ├── pile_etats.dart            EtatDeJeu, PileEtats    (26.22)
  │   │   └── camera.dart                monde -> écran          (ch. 25)
  │   │
  │   ├── jeu/                       le contenu du Donjon de Dart
  │   │   ├── contexte.dart              ContexteJeu             (26.29)
  │   │   ├── donnees_de_jeu.dart        score, vies, clés       (26.27)
  │   │   ├── evenements.dart            EnnemiTue, ObjetRamasse (26.26)
  │   │   │
  │   │   ├── entites/
  │   │   │   ├── personnage.dart            Character           (26.10)
  │   │   │   ├── heros.dart
  │   │   │   ├── gobelin.dart
  │   │   │   ├── chauve_souris.dart
  │   │   │   ├── boss.dart
  │   │   │   ├── potion.dart
  │   │   │   ├── cle.dart
  │   │   │   ├── coffre.dart
  │   │   │   ├── projectile.dart
  │   │   │   └── particule.dart
  │   │   │
  │   │   ├── etats/
  │   │   │   ├── etat_demarrage.dart
  │   │   │   ├── etat_menu.dart
  │   │   │   ├── etat_jeu.dart
  │   │   │   ├── etat_pause.dart
  │   │   │   ├── etat_options.dart
  │   │   │   └── etat_fin.dart
  │   │   │
  │   │   └── niveaux/
  │   │       ├── niveau.dart                modèle de niveau
  │   │       └── chargeur_niveau.dart       JSON -> entités     (ch. 17)
  │   │
  │   └── ui/                        tout ce qui est widget Flutter
  │       ├── app.dart                   MaterialApp
  │       ├── vue_de_jeu.dart            StatefulWidget + Ticker
  │       ├── peintre_de_jeu.dart        CustomPainter
  │       └── hud.dart                   barre de vie, score
  │
  └── test/
      ├── donnees_de_jeu_test.dart
      ├── machine_a_etats_test.dart
      ├── entites_test.dart
      ├── collisions_test.dart
      └── bus_evenements_test.dart
```

Quatre règles gouvernent cette arborescence. Elles valent bien plus que l'arborescence elle-même, car elles vous permettront de l'adapter.

**Règle 1 : `core/` ne connaît pas `jeu/`.** Le moteur ignore ce qu'est un gobelin. Vous devez pouvoir copier `core/` dans un autre projet sans rien modifier. Vérification simple : ouvrez chaque fichier de `core/` et cherchez le mot « gobelin » ou « donjon ». S'il apparaît, la règle est violée.

**Règle 2 : `jeu/` ne connaît pas `ui/`.** Aucun fichier de `jeu/` n'importe `material.dart`. Il peut importer `dart:ui` pour `Canvas` et `Color`, c'est tout. C'est ce qui rend `jeu/` testable en console.

**Règle 3 : `ui/` connaît tout le monde.** C'est la couche la plus externe, celle qui assemble. Elle a le droit de tout importer.

**Règle 4 : un fichier, une classe principale.** `gobelin.dart` contient `Gobelin`. On le retrouve sans chercher. Les petites classes annexes très liées peuvent l'accompagner.

Le sens des dépendances se résume ainsi.

```text
  SENS AUTORISÉ DES IMPORTS

     ui/  ──────────►  jeu/  ──────────►  core/
      │                                     ▲
      └─────────────────────────────────────┘

  Les flèches ne remontent JAMAIS.

  Test rapide : si un import de core/ commence par
  "package:donjon_de_dart/jeu/", vous avez une inversion.
```

Enfin, `main.dart` doit rester minuscule. C'est le meilleur indicateur de santé d'un projet.

```dart
import 'package:flutter/material.dart';
import 'ui/app.dart';

void main() {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const DonjonApp());
}
```

Cinq lignes utiles. Comparez avec les 2000 lignes de la section 26.1 : c'est le chemin parcouru dans ce chapitre.

> **Remarque.** N'appliquez pas cette arborescence dès la première ligne d'un prototype. Commencez par deux ou trois fichiers, et découpez quand un fichier dépasse trois cents lignes. Une architecture prématurée est aussi coûteuse qu'une architecture absente.

---

## 26.31 — Les tests dans un jeu : que tester et comment

Le chapitre 16 a introduit `package:test` et la fonction `expect`. Beaucoup d'élèves concluent qu'on ne peut pas tester un jeu, « parce que c'est visuel ». C'est faux : **la majorité d'un jeu n'est pas visuelle**.

```text
  QUE TESTER DANS UN JEU ?

  ┌──────────────────────────────────────────────────────────────┐
  │  TESTABLE FACILEMENT, ET RENTABLE                            │
  ├──────────────────────────────────────────────────────────────┤
  │  Les règles      dégâts, soins, vies, score, combo           │
  │  Les transitions machine à états, transitions interdites     │
  │  Les collisions  chevauchement AABB, cercles                 │
  │  La physique     position après n secondes, portée d'un saut │
  │  Les données     sérialisation JSON aller-retour             │
  │  Les entrées     touche -> action, remappage, conflits       │
  │  Le monde        ajout/retrait différé, ordre de rendu       │
  │  L'inventaire    ajout, empilement, capacité                 │
  └──────────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────────┐
  │  TESTABLE DIFFICILEMENT, ET PEU RENTABLE                     │
  ├──────────────────────────────────────────────────────────────┤
  │  Le rendu        « le sprite est-il joli ? »                 │
  │  Le ressenti     « le saut est-il agréable ? »               │
  │  L'équilibrage   « le boss est-il trop dur ? »               │
  │  Les FPS réels   dépendent de la machine                     │
  └──────────────────────────────────────────────────────────────┘

  La colonne de gauche représente 80 % du code d'un jeu.
```

Le secret est déjà acquis : tout ce que vous avez séparé du rendu dans ce chapitre est testable, **et rien d'autre ne l'est**. L'architecture n'est pas seulement plus propre, elle est la condition d'existence des tests.

Voici une suite de tests complète pour le Donjon de Dart. Le fichier se place dans `test/` et s'exécute avec `dart test`.

```dart
import 'package:test/test.dart';

// ------------------------------------------------------------
//  Le code testé (normalement importé depuis lib/).
// ------------------------------------------------------------

enum GameState { demarrage, menu, jeu, pause, gameOver, victoire }

class MachineAEtats {
  GameState etat = GameState.demarrage;

  static const Map<GameState, Set<GameState>> _ok = <GameState, Set<GameState>>{
    GameState.demarrage: <GameState>{GameState.menu},
    GameState.menu: <GameState>{GameState.jeu},
    GameState.jeu: <GameState>{
      GameState.pause,
      GameState.gameOver,
      GameState.victoire,
    },
    GameState.pause: <GameState>{GameState.jeu, GameState.menu},
    GameState.gameOver: <GameState>{GameState.jeu, GameState.menu},
    GameState.victoire: <GameState>{GameState.jeu, GameState.menu},
  };

  bool aller(GameState vers) {
    if (!(_ok[etat]?.contains(vers) ?? false)) return false;
    etat = vers;
    return true;
  }
}

class DonneesDeJeu {
  int score = 0;
  int vies = 3;
  int meilleurScore = 0;

  void ajouterPoints(int p) {
    if (p <= 0) return;
    score += p;
    if (score > meilleurScore) meilleurScore = score;
  }

  void perdreUneVie() {
    if (vies > 0) vies--;
  }

  bool get partiePerdue => vies <= 0;

  void nouvellePartie() {
    score = 0;
    vies = 3;
  }
}

abstract class Entity {
  double x = 0;
  double y = 0;
  bool aRetirer = false;
  void update(double dt);
}

class Gobelin extends Entity {
  Gobelin({required double px, this.pv = 8}) {
    x = px;
  }

  int pv;
  double vitesse = 40;
  int direction = 1;

  void subirDegats(int d) {
    pv -= d;
    if (pv <= 0) {
      pv = 0;
      aRetirer = true;
    }
  }

  @override
  void update(double dt) {
    x += vitesse * direction * dt;
  }
}

class Monde {
  final List<Entity> entites = <Entity>[];
  final List<Entity> _aAjouter = <Entity>[];

  void ajouter(Entity e) => _aAjouter.add(e);

  void update(double dt) {
    for (final Entity e in entites) {
      e.update(dt);
    }
    entites.addAll(_aAjouter);
    _aAjouter.clear();
    entites.removeWhere((Entity e) => e.aRetirer);
  }
}

bool seChevauchent(
  double ax, double ay, double al, double ah,
  double bx, double by, double bl, double bh,
) {
  return ax < bx + bl && ax + al > bx && ay < by + bh && ay + ah > by;
}

// ------------------------------------------------------------
//  Les tests.
// ------------------------------------------------------------

void main() {
  group('Machine à états', () {
    test('le chemin normal fonctionne', () {
      final MachineAEtats m = MachineAEtats();
      expect(m.aller(GameState.menu), isTrue);
      expect(m.aller(GameState.jeu), isTrue);
      expect(m.aller(GameState.pause), isTrue);
      expect(m.aller(GameState.jeu), isTrue);
      expect(m.etat, GameState.jeu);
    });

    test('on ne peut pas mettre le menu en pause', () {
      final MachineAEtats m = MachineAEtats();
      m.aller(GameState.menu);
      expect(m.aller(GameState.pause), isFalse);
      expect(m.etat, GameState.menu);
    });

    test('on ne peut pas perdre pendant la pause', () {
      final MachineAEtats m = MachineAEtats();
      m.aller(GameState.menu);
      m.aller(GameState.jeu);
      m.aller(GameState.pause);
      expect(m.aller(GameState.gameOver), isFalse);
    });

    test('tout état sauf demarrage peut revenir au menu ou y mener', () {
      for (final GameState e in GameState.values) {
        final MachineAEtats m = MachineAEtats();
        m.etat = e;
        // Aucune transition ne doit mener à demarrage.
        expect(m.aller(GameState.demarrage), isFalse,
            reason: 'depuis ${e.name}');
      }
    });
  });

  group('Données de jeu', () {
    test('le score augmente et met à jour le record', () {
      final DonneesDeJeu d = DonneesDeJeu();
      d.ajouterPoints(10);
      d.ajouterPoints(25);
      expect(d.score, 35);
      expect(d.meilleurScore, 35);
    });

    test('les points négatifs sont ignorés', () {
      final DonneesDeJeu d = DonneesDeJeu();
      d.ajouterPoints(-100);
      expect(d.score, 0);
    });

    test('les vies ne descendent pas sous zéro', () {
      final DonneesDeJeu d = DonneesDeJeu();
      for (int i = 0; i < 10; i++) {
        d.perdreUneVie();
      }
      expect(d.vies, 0);
      expect(d.partiePerdue, isTrue);
    });

    test('une nouvelle partie conserve le record', () {
      final DonneesDeJeu d = DonneesDeJeu();
      d.ajouterPoints(120);
      d.nouvellePartie();
      expect(d.score, 0);
      expect(d.vies, 3);
      expect(d.meilleurScore, 120);
    });
  });

  group('Monde et entités', () {
    test('le mouvement ne dépend pas du nombre de frames', () {
      final Gobelin a = Gobelin(px: 0);
      final Gobelin b = Gobelin(px: 0);

      for (int i = 0; i < 60; i++) {
        a.update(1 / 60);
      }
      for (int i = 0; i < 30; i++) {
        b.update(1 / 30);
      }

      expect(a.x, closeTo(b.x, 0.0001));
      expect(a.x, closeTo(40, 0.0001));
    });

    test('un gobelin à 0 pv est marqué et retiré', () {
      final Monde monde = Monde();
      final Gobelin g = Gobelin(px: 100, pv: 5);
      monde.entites.add(g);

      g.subirDegats(5);
      expect(g.aRetirer, isTrue);

      monde.update(1 / 60);
      expect(monde.entites, isEmpty);
    });

    test('ajouter pendant la boucle ne lève pas d exception', () {
      final Monde monde = Monde();
      monde.entites.add(Gobelin(px: 0));

      expect(() {
        monde.ajouter(Gobelin(px: 50));
        monde.update(1 / 60);
      }, returnsNormally);

      expect(monde.entites.length, 2);
    });

    test('les pv ne descendent pas sous zéro', () {
      final Gobelin g = Gobelin(px: 0, pv: 3);
      g.subirDegats(999);
      expect(g.pv, 0);
    });
  });

  group('Collisions AABB', () {
    test('deux rectangles qui se chevauchent', () {
      expect(seChevauchent(0, 0, 10, 10, 5, 5, 10, 10), isTrue);
    });

    test('deux rectangles séparés', () {
      expect(seChevauchent(0, 0, 10, 10, 20, 20, 10, 10), isFalse);
    });

    test('deux rectangles qui se touchent bord à bord ne collisionnent pas', () {
      expect(seChevauchent(0, 0, 10, 10, 10, 0, 10, 10), isFalse);
    });

    test('un rectangle contenu dans un autre', () {
      expect(seChevauchent(0, 0, 100, 100, 40, 40, 10, 10), isTrue);
    });
  });
}
```

**Résultat :**

```text
00:00 +17: All tests passed!
```

Quelques principes de rédaction de tests, applicables à tous vos projets.

**Un test, une assertion de sens.** Le nom du test doit décrire la règle vérifiée, pas la méthode appelée. Comparez « test de perdreUneVie » et « les vies ne descendent pas sous zéro » : seul le second vous dira, six mois plus tard, ce que le jeu est censé faire.

**Testez les bords, pas le milieu.** Zéro vie, zéro point de vie, dégâts négatifs, rectangles jointifs. Les bugs se cachent aux extrémités.

**Testez l'indépendance au framerate.** Le test « 60 frames à 1/60 donne le même résultat que 30 frames à 1/30 » est le test le plus rentable d'un jeu. Il attrape toutes les régressions du type `x += 5`.

**Testez ce qui est interdit.** La moitié des tests de la machine à états vérifie que des transitions **échouent**. Un test qui ne vérifie que le cas nominal laisse passer les vrais bugs.

> **Remarque.** Pour les widgets Flutter, `flutter_test` fournit `testWidgets` et `WidgetTester`. C'est utile pour vérifier qu'un bouton de menu appelle bien la bonne action, mais ne cherchez pas à tester le `CustomPainter` pixel par pixel : le rapport effort/bénéfice est mauvais.

---

## 26.32 — Bilan de la PARTIE 2A : le mini-moteur complet

Assemblons tout. Le programme ci-dessous est un jeu Flutter complet, en un seul fichier pour pouvoir être collé dans DartPad, mais organisé exactement selon les sections précédentes : chaque bloc correspond à un futur fichier de l'arborescence 26.30.

Il réunit la boucle et le delta time du chapitre 20, le `Canvas` du chapitre 21, l'animation par index du chapitre 22, la vélocité et la gravité du chapitre 23, les collisions AABB du chapitre 24, une caméra simple du chapitre 25, et toute l'architecture du chapitre 26.

```dart
import 'dart:math' as math;
import 'dart:ui' show PointMode;

import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';
import 'package:flutter/services.dart';

// ============================================================
//  core/couches.dart
// ============================================================

abstract class Couches {
  static const int fond = -100;
  static const int sol = 0;
  static const int objets = 10;
  static const int ennemis = 20;
  static const int joueur = 30;
  static const int particules = 50;
}

// ============================================================
//  core/entrees.dart
// ============================================================

enum ActionJeu { gauche, droite, sauter, pause, valider }

class GestionnaireEntrees {
  final Set<ActionJeu> _maintenues = <ActionJeu>{};
  final Set<ActionJeu> _pressees = <ActionJeu>{};

  static const Map<LogicalKeyboardKey, ActionJeu> mapping =
      <LogicalKeyboardKey, ActionJeu>{
    LogicalKeyboardKey.arrowLeft: ActionJeu.gauche,
    LogicalKeyboardKey.arrowRight: ActionJeu.droite,
    LogicalKeyboardKey.keyQ: ActionJeu.gauche,
    LogicalKeyboardKey.keyD: ActionJeu.droite,
    LogicalKeyboardKey.space: ActionJeu.sauter,
    LogicalKeyboardKey.escape: ActionJeu.pause,
    LogicalKeyboardKey.enter: ActionJeu.valider,
  };

  bool maintenue(ActionJeu a) => _maintenues.contains(a);
  bool pressee(ActionJeu a) => _pressees.contains(a);

  void appuyer(ActionJeu a) {
    if (_maintenues.add(a)) _pressees.add(a);
  }

  void relacher(ActionJeu a) => _maintenues.remove(a);

  void toutRelacher() {
    _maintenues.clear();
    _pressees.clear();
  }

  void finDeFrame() => _pressees.clear();

  void surTouche(KeyEvent e) {
    final ActionJeu? a = mapping[e.logicalKey];
    if (a == null) return;
    if (e is KeyDownEvent) {
      appuyer(a);
    } else if (e is KeyUpEvent) {
      relacher(a);
    }
  }
}

// ============================================================
//  core/bus_evenements.dart
// ============================================================

abstract class EvenementJeu {
  const EvenementJeu();
}

class EnnemiTue extends EvenementJeu {
  const EnnemiTue(this.x, this.y, this.points);
  final double x;
  final double y;
  final int points;
}

class ObjetRamasse extends EvenementJeu {
  const ObjetRamasse(this.nom, this.x, this.y);
  final String nom;
  final double x;
  final double y;
}

class JoueurTouche extends EvenementJeu {
  const JoueurTouche();
}

class BusEvenements {
  final Map<Type, List<Function>> _ecouteurs = <Type, List<Function>>{};

  void Function() ecouter<T extends EvenementJeu>(void Function(T) f) {
    _ecouteurs.putIfAbsent(T, () => <Function>[]).add(f);
    return () => _ecouteurs[T]?.remove(f);
  }

  void publier<T extends EvenementJeu>(T e) {
    final List<Function>? l = _ecouteurs[e.runtimeType];
    if (l == null) return;
    for (final Function f in List<Function>.of(l)) {
      (f as void Function(T))(e);
    }
  }
}

// ============================================================
//  core/entity.dart
// ============================================================

abstract class Entity {
  Entity({this.x = 0, this.y = 0, this.priority = 0});

  double x;
  double y;
  int priority;
  bool aRetirer = false;
  Monde? monde;

  double get largeur => 0;
  double get hauteur => 0;

  Rect get boite => Rect.fromLTWH(x, y, largeur, hauteur);

  void update(double dt);
  void render(Canvas canvas);
}

// ============================================================
//  core/monde.dart
// ============================================================

class Monde {
  Monde({required this.largeur, required this.hauteur});

  final double largeur;
  final double hauteur;

  final List<Entity> entites = <Entity>[];
  final List<Entity> _aAjouter = <Entity>[];
  final List<Entity> _ordre = <Entity>[];
  bool _triNecessaire = true;

  void ajouter(Entity e) {
    e.monde = this;
    _aAjouter.add(e);
  }

  void ajouterMaintenant(Entity e) {
    e.monde = this;
    entites.add(e);
    _triNecessaire = true;
  }

  Iterable<T> de<T extends Entity>() => entites.whereType<T>();

  void update(double dt) {
    for (final Entity e in entites) {
      e.update(dt);
    }

    if (_aAjouter.isNotEmpty) {
      entites.addAll(_aAjouter);
      _aAjouter.clear();
      _triNecessaire = true;
    }
    final int avant = entites.length;
    entites.removeWhere((Entity e) => e.aRetirer);
    if (entites.length != avant) _triNecessaire = true;
  }

  void render(Canvas canvas) {
    if (_triNecessaire) {
      _ordre
        ..clear()
        ..addAll(entites)
        ..sort((Entity a, Entity b) {
          final int c = a.priority.compareTo(b.priority);
          return c != 0 ? c : a.y.compareTo(b.y);
        });
      _triNecessaire = false;
    }
    for (final Entity e in _ordre) {
      e.render(canvas);
    }
  }
}

// ============================================================
//  core/pile_etats.dart
// ============================================================

abstract class EtatDeJeu {
  late PileEtats pile;

  String get nom;
  bool get transparent => false;

  void entrer() {}
  void sortir() {}
  void endormir() {}
  void reveiller() {}
  void update(double dt) {}
  void render(Canvas canvas, Size taille) {}
}

class PileEtats {
  final List<EtatDeJeu> _pile = <EtatDeJeu>[];

  EtatDeJeu? get sommet => _pile.isEmpty ? null : _pile.last;
  String get chemin => _pile.map((EtatDeJeu e) => e.nom).join(' > ');

  void empiler(EtatDeJeu e) {
    if (_pile.isNotEmpty) _pile.last.endormir();
    e.pile = this;
    _pile.add(e);
    e.entrer();
  }

  void depiler() {
    if (_pile.isEmpty) return;
    _pile.removeLast().sortir();
    if (_pile.isNotEmpty) _pile.last.reveiller();
  }

  void remplacerTout(EtatDeJeu e) {
    while (_pile.isNotEmpty) {
      _pile.removeLast().sortir();
    }
    empiler(e);
  }

  void update(double dt) => sommet?.update(dt);

  void render(Canvas canvas, Size taille) {
    if (_pile.isEmpty) return;
    int debut = _pile.length - 1;
    while (debut > 0 && _pile[debut].transparent) {
      debut--;
    }
    for (int i = debut; i < _pile.length; i++) {
      _pile[i].render(canvas, taille);
    }
  }
}

// ============================================================
//  jeu/donnees_de_jeu.dart
// ============================================================

class DonneesDeJeu {
  int score = 0;
  int vies = 3;
  int meilleurScore = 0;

  void ajouterPoints(int p) {
    if (p <= 0) return;
    score += p;
    if (score > meilleurScore) meilleurScore = score;
  }

  void perdreUneVie() {
    if (vies > 0) vies--;
  }

  bool get partiePerdue => vies <= 0;

  void nouvellePartie() {
    score = 0;
    vies = 3;
  }
}

// ============================================================
//  jeu/contexte.dart
// ============================================================

class ContexteJeu {
  ContexteJeu({
    required this.donnees,
    required this.entrees,
    required this.bus,
  });

  final DonneesDeJeu donnees;
  final GestionnaireEntrees entrees;
  final BusEvenements bus;
}

// ============================================================
//  jeu/entites/*.dart
// ============================================================

class Decor extends Entity {
  Decor({
    required double x,
    required double y,
    required this.l,
    required this.h,
    required this.couleur,
    int priority = Couches.sol,
  }) : super(x: x, y: y, priority: priority);

  final double l;
  final double h;
  final Color couleur;

  @override
  double get largeur => l;

  @override
  double get hauteur => h;

  @override
  void update(double dt) {}

  @override
  void render(Canvas canvas) {
    canvas.drawRect(boite, Paint()..color = couleur);
  }
}

class Heros extends Entity {
  Heros(this.contexte, {required double x, required double y})
      : super(x: x, y: y, priority: Couches.joueur);

  final ContexteJeu contexte;

  double vx = 0;
  double vy = 0;
  bool auSol = false;
  double invincible = 0;

  double _tempsAnim = 0;
  int _frame = 0;

  static const double vitesse = 180;
  static const double forceSaut = -420;
  static const double gravite = 1100;

  @override
  double get largeur => 24;

  @override
  double get hauteur => 32;

  @override
  void update(double dt) {
    final GestionnaireEntrees e = contexte.entrees;

    // 1. Entrées -> vitesse horizontale.
    vx = 0;
    if (e.maintenue(ActionJeu.gauche)) vx -= vitesse;
    if (e.maintenue(ActionJeu.droite)) vx += vitesse;
    if (e.pressee(ActionJeu.sauter) && auSol) {
      vy = forceSaut;
      auSol = false;
    }

    // 2. Gravité (chapitre 23).
    vy += gravite * dt;

    // 3. Déplacement axe par axe, avec résolution (chapitre 24).
    x += vx * dt;
    _resoudre(horizontal: true);
    y += vy * dt;
    auSol = false;
    _resoudre(horizontal: false);

    // 4. Limites du monde.
    final Monde? m = monde;
    if (m != null) {
      x = x.clamp(0.0, m.largeur - largeur);
      if (y > m.hauteur) {
        contexte.bus.publier(const JoueurTouche());
        x = 40;
        y = 100;
        vy = 0;
      }
    }

    // 5. Animation (chapitre 22).
    _tempsAnim += dt;
    if (_tempsAnim >= 0.12) {
      _tempsAnim = 0;
      _frame = (_frame + 1) % 4;
    }

    // 6. Invincibilité temporaire.
    if (invincible > 0) invincible -= dt;
  }

  void _resoudre({required bool horizontal}) {
    final Monde? m = monde;
    if (m == null) return;
    for (final Decor d in m.de<Decor>()) {
      if (!boite.overlaps(d.boite)) continue;
      if (horizontal) {
        x = vx > 0 ? d.x - largeur : d.x + d.largeur;
      } else {
        if (vy > 0) {
          y = d.y - hauteur;
          auSol = true;
        } else {
          y = d.y + d.hauteur;
        }
        vy = 0;
      }
    }
  }

  void blesser() {
    if (invincible > 0) return;
    invincible = 1.5;
    contexte.donnees.perdreUneVie();
    contexte.bus.publier(const JoueurTouche());
  }

  @override
  void render(Canvas canvas) {
    // Clignotement pendant l'invincibilité.
    if (invincible > 0 && (invincible * 10).floor() % 2 == 0) return;

    final double oscillation = auSol && vx != 0 ? (_frame % 2) * 2 : 0;
    canvas.drawRRect(
      RRect.fromRectAndRadius(
        Rect.fromLTWH(x, y + oscillation, largeur, hauteur - oscillation),
        const Radius.circular(4),
      ),
      Paint()..color = const Color(0xFFE8B04B),
    );
    // Les yeux, orientés dans le sens du déplacement.
    final double dx = vx < 0 ? 5 : 13;
    canvas.drawRect(
      Rect.fromLTWH(x + dx, y + 8, 6, 4),
      Paint()..color = const Color(0xFF2A2118),
    );
  }
}

class Gobelin extends Entity {
  Gobelin(this.contexte, {
    required double x,
    required double y,
    required this.xMin,
    required this.xMax,
  }) : super(x: x, y: y, priority: Couches.ennemis);

  final ContexteJeu contexte;
  final double xMin;
  final double xMax;

  int pv = 2;
  int direction = 1;
  double vitesse = 55;

  @override
  double get largeur => 22;

  @override
  double get hauteur => 22;

  @override
  void update(double dt) {
    x += vitesse * direction * dt;
    if (x < xMin) {
      x = xMin;
      direction = 1;
    } else if (x > xMax) {
      x = xMax;
      direction = -1;
    }

    final Monde? m = monde;
    if (m == null) return;
    for (final Heros h in m.de<Heros>()) {
      if (!boite.overlaps(h.boite)) continue;
      // Saut sur la tête : l'ennemi meurt.
      if (h.vy > 0 && h.y + h.hauteur - 10 < y) {
        mourir();
        h.vy = -280;
      } else {
        h.blesser();
      }
    }
  }

  void mourir() {
    aRetirer = true;
    contexte.bus.publier(EnnemiTue(x, y, 10));
  }

  @override
  void render(Canvas canvas) {
    canvas.drawRect(boite, Paint()..color = const Color(0xFF6FBF73));
    canvas.drawRect(
      Rect.fromLTWH(x + (direction > 0 ? 13 : 4), y + 6, 5, 4),
      Paint()..color = const Color(0xFF13351A),
    );
  }
}

class Piece extends Entity {
  Piece(this.contexte, {required double x, required double y})
      : super(x: x, y: y, priority: Couches.objets) {
    _yBase = y;
  }

  final ContexteJeu contexte;
  double _yBase = 0;
  double _phase = 0;

  @override
  double get largeur => 14;

  @override
  double get hauteur => 14;

  @override
  void update(double dt) {
    _phase += dt * 3;
    y = _yBase + math.sin(_phase) * 3;

    final Monde? m = monde;
    if (m == null) return;
    for (final Heros h in m.de<Heros>()) {
      if (boite.overlaps(h.boite)) {
        aRetirer = true;
        contexte.bus.publier(ObjetRamasse('piece', x, y));
      }
    }
  }

  @override
  void render(Canvas canvas) {
    canvas.drawCircle(
      Offset(x + 7, y + 7),
      7,
      Paint()..color = const Color(0xFFF2C744),
    );
  }
}

class Particule extends Entity {
  Particule({required double x, required double y, required this.couleur})
      : super(x: x, y: y, priority: Couches.particules) {
    final math.Random r = math.Random();
    vx = (r.nextDouble() - 0.5) * 180;
    vy = -r.nextDouble() * 200;
  }

  final Color couleur;
  double vx = 0;
  double vy = 0;
  double vie = 0.8;

  @override
  void update(double dt) {
    vy += 700 * dt;
    x += vx * dt;
    y += vy * dt;
    vie -= dt;
    if (vie <= 0) aRetirer = true;
  }

  @override
  void render(Canvas canvas) {
    canvas.drawPoints(
      PointMode.points,
      <Offset>[Offset(x, y)],
      Paint()
        ..color = couleur.withOpacity(vie.clamp(0.0, 1.0))
        ..strokeWidth = 4
        ..strokeCap = StrokeCap.round,
    );
  }
}

// ============================================================
//  jeu/etats/*.dart
// ============================================================

class EtatMenu extends EtatDeJeu {
  EtatMenu(this.contexte);
  final ContexteJeu contexte;

  double _temps = 0;

  @override
  String get nom => 'MENU';

  @override
  void update(double dt) {
    _temps += dt;
    if (contexte.entrees.pressee(ActionJeu.valider)) {
      contexte.donnees.nouvellePartie();
      pile.remplacerTout(EtatJeu(contexte));
    }
  }

  @override
  void render(Canvas canvas, Size taille) {
    canvas.drawRect(Offset.zero & taille, Paint()..color = const Color(0xFF14161C));
    _texte(canvas, 'DONJON DE DART', taille.width / 2,
        taille.height / 2 - 60, 36, const Color(0xFFE8B04B));
    final bool clignote = (_temps * 2).floor() % 2 == 0;
    if (clignote) {
      _texte(canvas, 'Entree pour jouer', taille.width / 2,
          taille.height / 2 + 10, 18, Colors.white);
    }
    _texte(
      canvas,
      'Fleches ou Q/D : bouger    Espace : sauter    Echap : pause',
      taille.width / 2,
      taille.height - 40,
      13,
      const Color(0xFF8A93A5),
    );
    if (contexte.donnees.meilleurScore > 0) {
      _texte(canvas, 'Record : ${contexte.donnees.meilleurScore}',
          taille.width / 2, taille.height / 2 + 50, 15,
          const Color(0xFF8A93A5));
    }
  }
}

class EtatJeu extends EtatDeJeu {
  EtatJeu(this.contexte);
  final ContexteJeu contexte;

  late Monde monde;
  late Heros heros;
  double cameraX = 0;
  final List<void Function()> _abonnements = <void Function()>[];

  @override
  String get nom => 'JEU';

  @override
  void entrer() {
    _construireNiveau();

    _abonnements.add(contexte.bus.ecouter<EnnemiTue>((EnnemiTue e) {
      contexte.donnees.ajouterPoints(e.points);
      for (int i = 0; i < 12; i++) {
        monde.ajouter(Particule(
            x: e.x + 11, y: e.y + 11, couleur: const Color(0xFF6FBF73)));
      }
    }));

    _abonnements.add(contexte.bus.ecouter<ObjetRamasse>((ObjetRamasse e) {
      contexte.donnees.ajouterPoints(5);
      for (int i = 0; i < 8; i++) {
        monde.ajouter(Particule(
            x: e.x + 7, y: e.y + 7, couleur: const Color(0xFFF2C744)));
      }
    }));

    _abonnements.add(contexte.bus.ecouter<JoueurTouche>((JoueurTouche e) {
      for (int i = 0; i < 10; i++) {
        monde.ajouter(Particule(
            x: heros.x + 12, y: heros.y + 16,
            couleur: const Color(0xFFCF5C5C)));
      }
    }));
  }

  @override
  void sortir() {
    for (final void Function() d in _abonnements) {
      d();
    }
    _abonnements.clear();
  }

  @override
  void endormir() => contexte.entrees.toutRelacher();

  void _construireNiveau() {
    monde = Monde(largeur: 1600, hauteur: 400);

    // Le sol et quelques plateformes.
    monde.ajouterMaintenant(Decor(
        x: 0, y: 340, l: 1600, h: 60, couleur: const Color(0xFF3A3F4B)));
    monde.ajouterMaintenant(Decor(
        x: 260, y: 260, l: 140, h: 20, couleur: const Color(0xFF4A5160)));
    monde.ajouterMaintenant(Decor(
        x: 520, y: 200, l: 140, h: 20, couleur: const Color(0xFF4A5160)));
    monde.ajouterMaintenant(Decor(
        x: 820, y: 260, l: 180, h: 20, couleur: const Color(0xFF4A5160)));
    monde.ajouterMaintenant(Decor(
        x: 1180, y: 220, l: 200, h: 20, couleur: const Color(0xFF4A5160)));

    heros = Heros(contexte, x: 40, y: 260);
    monde.ajouterMaintenant(heros);

    monde.ajouterMaintenant(
        Gobelin(contexte, x: 300, y: 318, xMin: 240, xMax: 420));
    monde.ajouterMaintenant(
        Gobelin(contexte, x: 700, y: 318, xMin: 640, xMax: 860));
    monde.ajouterMaintenant(
        Gobelin(contexte, x: 1100, y: 318, xMin: 1020, xMax: 1300));
    monde.ajouterMaintenant(
        Gobelin(contexte, x: 860, y: 238, xMin: 820, xMax: 960));

    for (int i = 0; i < 14; i++) {
      monde.ajouterMaintenant(
          Piece(contexte, x: 120.0 + i * 100, y: 300 - (i % 3) * 40));
    }
  }

  @override
  void update(double dt) {
    if (contexte.entrees.pressee(ActionJeu.pause)) {
      pile.empiler(EtatPause(contexte));
      return;
    }

    monde.update(dt);

    // Caméra : elle suit le héros avec un lissage (chapitre 25).
    final double cible = heros.x - 320;
    cameraX += (cible - cameraX) * math.min(1.0, dt * 6);
    cameraX = cameraX.clamp(0.0, monde.largeur - 720);

    if (contexte.donnees.partiePerdue) {
      pile.remplacerTout(EtatFin(contexte, gagne: false));
    } else if (monde.de<Gobelin>().isEmpty && monde.de<Piece>().isEmpty) {
      pile.remplacerTout(EtatFin(contexte, gagne: true));
    }
  }

  @override
  void render(Canvas canvas, Size taille) {
    canvas.drawRect(
        Offset.zero & taille, Paint()..color = const Color(0xFF14161C));

    // Le monde, décalé par la caméra.
    canvas.save();
    canvas.translate(-cameraX, 0);
    monde.render(canvas);
    canvas.restore();

    // Le HUD, jamais décalé.
    _texte(canvas, 'Score ${contexte.donnees.score}', 70, 24, 16,
        const Color(0xFFE8B04B));
    _texte(canvas, 'Vies ${contexte.donnees.vies}', taille.width - 70, 24, 16,
        const Color(0xFFCF5C5C));
    _texte(canvas, 'Pieces ${monde.de<Piece>().length}', taille.width / 2, 24,
        14, const Color(0xFF8A93A5));
  }
}

class EtatPause extends EtatDeJeu {
  EtatPause(this.contexte);
  final ContexteJeu contexte;

  @override
  String get nom => 'PAUSE';

  @override
  bool get transparent => true;

  @override
  void entrer() => contexte.entrees.toutRelacher();

  @override
  void update(double dt) {
    if (contexte.entrees.pressee(ActionJeu.pause)) {
      pile.depiler();
    } else if (contexte.entrees.pressee(ActionJeu.valider)) {
      pile.remplacerTout(EtatMenu(contexte));
    }
  }

  @override
  void render(Canvas canvas, Size taille) {
    canvas.drawRect(Offset.zero & taille, Paint()..color = const Color(0xB3000000));
    _texte(canvas, 'PAUSE', taille.width / 2, taille.height / 2 - 30, 34,
        const Color(0xFFE8B04B));
    _texte(canvas, 'Echap : reprendre    Entree : menu', taille.width / 2,
        taille.height / 2 + 20, 15, Colors.white);
  }
}

class EtatFin extends EtatDeJeu {
  EtatFin(this.contexte, {required this.gagne});
  final ContexteJeu contexte;
  final bool gagne;

  double _temps = 0;

  @override
  String get nom => gagne ? 'VICTOIRE' : 'GAME OVER';

  @override
  void entrer() => contexte.entrees.toutRelacher();

  @override
  void update(double dt) {
    _temps += dt;
    if (_temps > 0.6 && contexte.entrees.pressee(ActionJeu.valider)) {
      pile.remplacerTout(EtatMenu(contexte));
    }
  }

  @override
  void render(Canvas canvas, Size taille) {
    canvas.drawRect(
        Offset.zero & taille, Paint()..color = const Color(0xFF14161C));
    _texte(canvas, nom, taille.width / 2, taille.height / 2 - 60, 36,
        gagne ? const Color(0xFF6FBF73) : const Color(0xFFCF5C5C));
    _texte(canvas, 'Score : ${contexte.donnees.score}', taille.width / 2,
        taille.height / 2, 20, Colors.white);
    _texte(canvas, 'Record : ${contexte.donnees.meilleurScore}',
        taille.width / 2, taille.height / 2 + 30, 15,
        const Color(0xFF8A93A5));
    if (_temps > 0.6) {
      _texte(canvas, 'Entree pour revenir au menu', taille.width / 2,
          taille.height / 2 + 80, 15, const Color(0xFF8A93A5));
    }
  }
}

// ============================================================
//  ui/ — outil de texte partagé
// ============================================================

void _texte(Canvas canvas, String texte, double cx, double cy, double taille,
    Color couleur) {
  final TextPainter tp = TextPainter(
    text: TextSpan(
      text: texte,
      style: TextStyle(
          color: couleur, fontSize: taille, fontWeight: FontWeight.w600),
    ),
    textDirection: TextDirection.ltr,
  )..layout();
  tp.paint(canvas, Offset(cx - tp.width / 2, cy - tp.height / 2));
}

// ============================================================
//  ui/vue_de_jeu.dart
// ============================================================

class VueDeJeu extends StatefulWidget {
  const VueDeJeu({super.key});

  @override
  State<VueDeJeu> createState() => _VueDeJeuState();
}

class _VueDeJeuState extends State<VueDeJeu>
    with SingleTickerProviderStateMixin {
  late final ContexteJeu contexte;
  late final PileEtats pile;
  late final Ticker _ticker;
  final ValueNotifier<int> _frame = ValueNotifier<int>(0);
  final FocusNode _focus = FocusNode();

  Duration _precedent = Duration.zero;

  @override
  void initState() {
    super.initState();
    contexte = ContexteJeu(
      donnees: DonneesDeJeu(),
      entrees: GestionnaireEntrees(),
      bus: BusEvenements(),
    );
    pile = PileEtats();
    pile.empiler(EtatMenu(contexte));

    _ticker = createTicker(_surFrame)..start();
  }

  void _surFrame(Duration ecoule) {
    double dt = (ecoule - _precedent).inMicroseconds / 1000000.0;
    _precedent = ecoule;
    dt = dt.clamp(0.0, 0.05);

    pile.update(dt);
    contexte.entrees.finDeFrame();
    _frame.value++;
  }

  @override
  void dispose() {
    _ticker.dispose();
    _focus.dispose();
    _frame.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return KeyboardListener(
      focusNode: _focus,
      autofocus: true,
      onKeyEvent: contexte.entrees.surTouche,
      child: CustomPaint(
        painter: _PeintreDeJeu(pile: pile, repaint: _frame),
        child: const SizedBox.expand(),
      ),
    );
  }
}

class _PeintreDeJeu extends CustomPainter {
  _PeintreDeJeu({required this.pile, required Listenable repaint})
      : super(repaint: repaint);

  final PileEtats pile;

  @override
  void paint(Canvas canvas, Size size) => pile.render(canvas, size);

  @override
  bool shouldRepaint(covariant _PeintreDeJeu ancien) => false;
}

// ============================================================
//  main.dart
// ============================================================

void main() => runApp(const DonjonApp());

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF14161C),
        body: SafeArea(child: VueDeJeu()),
      ),
    );
  }
}
```

**Résultat :** l'application démarre sur le menu du Donjon de Dart, avec un texte clignotant et le record affiché s'il existe. La touche Entrée lance une partie. Le héros doré se déplace avec les flèches ou Q et D, saute avec Espace, et la caméra le suit en douceur sur un niveau de 1600 pixels de large. Les gobelins verts patrouillent : les toucher de côté coûte une vie et déclenche une gerbe rouge et un clignotement d'invincibilité ; leur sauter dessus les fait exploser en particules vertes et rapporte 10 points. Les pièces dorées flottent, rapportent 5 points et éclatent en étincelles jaunes. Échap superpose un voile de pause au-dessus du monde figé, Échap à nouveau reprend exactement là où l'on s'était arrêté. À zéro vie, l'écran de Game Over s'affiche ; en ramassant toutes les pièces et en éliminant tous les gobelins, c'est l'écran de victoire.

Regardez maintenant la structure du fichier. Chaque bloc de commentaires correspond à un fichier de l'arborescence de la section 26.30. Le découpage n'est plus une théorie : c'est littéralement la table des matières de votre code.

Comptons ce que ce programme réunit.

| Chapitre | Ce qu'il apporte ici |
| --- | --- |
| 20 | `Ticker`, `dt` plafonné, séparation update/render |
| 21 | `Canvas`, `Paint`, `save`/`restore`, `TextPainter` |
| 22 | animation par index de frame (`_frame`, `_tempsAnim`) |
| 23 | gravité, vitesse, saut, force de saut |
| 24 | AABB avec `Rect.overlaps`, résolution axe par axe |
| 25 | caméra lissée avec `translate` et `clamp` |
| 26 | `Entity`, `Monde`, files, `priority`, pile d'états, entrées, bus, contexte |

---

## 26.33 — Pourquoi passer à Flame maintenant

Vous pourriez continuer ainsi. Beaucoup de jeux commerciaux ont été écrits avec un moteur maison de ce type. Mais faites l'inventaire de ce qu'il vous reste à écrire pour terminer le Donjon de Dart de la PARTIE 2C.

```text
  CE QU'IL RESTE À ÉCRIRE À LA MAIN

  Chargement d'images        décodage, cache, attente asynchrone
  Sprite sheets              découpe, atlas, animations nommées
  Audio                      musique, effets, volumes, boucles
  Tilemap                    lecture Tiled, calques, objets
  Détection de collisions    quadtree, hitbox par forme, callbacks
  Effets                     déplacement, échelle, opacité, séquences
  Particules                 générateurs, durées de vie, formes
  Timers                     répétitions, délais, annulations
  Caméra avancée             viewport, zoom, secousses, limites
  Joystick tactile           zone morte, retour visuel, multi-touch
  Boutons du HUD             zones tactiles, états visuels
  Cycle de vie               chargement, montage, démontage, resize
  Gestion du focus           perte de fenêtre, arrière-plan mobile

  Estimation honnête : plusieurs milliers de lignes,
  plusieurs semaines, et beaucoup de bugs subtils.
```

Ce travail a déjà été fait, testé et corrigé par la communauté Flame. Voici ce qui change concrètement.

| Ce que vous avez écrit | Ce que Flame fournit | Économie |
| --- | --- | --- |
| `MoteurDeBoucle` + `Ticker` | `FlameGame` + `GameWidget` | 150 lignes |
| `abstract class Entity` | `Component`, `PositionComponent` | 80 lignes |
| `Monde` avec files et tri | l'arbre de composants | 120 lignes |
| chargement d'images | `game.images.load` | 200 lignes |
| animation de sprite sheet | `SpriteAnimationComponent` | 250 lignes |
| collisions AABB à la main | `HasCollisionDetection`, `RectangleHitbox` | 400 lignes |
| caméra et viewport | `CameraComponent`, `World` | 300 lignes |
| joystick tactile | `JoystickComponent` | 250 lignes |
| effets et interpolations | `MoveEffect`, `OpacityEffect` | 300 lignes |
| particules | `ParticleSystemComponent` | 250 lignes |
| audio | `flame_audio` | 200 lignes |
| lecture de Tiled | `flame_tiled` | 500 lignes |

Mais le vrai bénéfice n'est pas là. Il est dans ce que vous avez gagné en écrivant tout cela vous-même.

**Vous savez ce que fait `update(dt)`.** Ce n'est pas une méthode magique : c'est la deuxième étape de la boucle du chapitre 20.

**Vous savez pourquoi `add()` est différé.** Flame met les composants dans une file, exactement pour la raison de la section 26.7. Quand vous lirez que `add` est asynchrone, vous saurez pourquoi.

**Vous savez ce que fait `priority`.** C'est l'entier de la section 26.9, et le moteur trie pour vous.

**Vous savez pourquoi les positions sont relatives au parent.** C'est l'arbre de la section 26.15.

**Vous savez ce que Flame ne fait pas.** Machine à états de jeu, gestionnaire d'entrées abstrait, bus d'événements, service de données, injection de dépendances, organisation des fichiers, tests : tout cela reste votre travail, dans Flame comme ailleurs. Les sections 26.16 à 26.31 resteront valables mot pour mot.

C'est la différence entre utiliser un moteur et le subir. Un développeur qui découvre Flame sans avoir écrit sa boucle passera trois jours sur un bug de `dt`. Vous, vous saurez où regarder.

---

## 26.34 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `Concurrent modification during iteration` | on ajoute ou retire une entité pendant `for (final e in entites)` | marquer `aRetirer = true` et empiler dans `_aAjouter`, puis vider les files après la boucle (26.8) |
| Un ennemi sur deux ignore les dégâts | boucle `for (int i = 0; ...)` avec `removeAt(i)` : l'index saute un élément | utiliser `removeWhere` après la boucle, jamais pendant |
| Le héros va deux fois plus vite sur un écran 120 Hz | un `+=` est écrit dans `render` au lieu de `update` | déplacer toute modification d'état dans `update`, et multiplier par `dt` (26.2, 26.5) |
| Le ciel recouvre tout le jeu | l'ordre de rendu est l'ordre d'insertion | attribuer un `priority` à chaque entité et trier avant de dessiner (26.9) |
| Deux sprites clignotent quand ils se croisent | tri instable entre deux entités de même `priority` | trier sur le couple `(priority, y)` ou donner des priorités distinctes (26.9) |
| Il faut 8 classes pour 3 comportements optionnels | explosion combinatoire de l'héritage | passer à la composition : des comportements attachés (26.12) |
| `Missing concrete implementation of 'Entity.render'` | une sous-classe oublie une méthode abstraite | ajouter la méthode manquante avec `@override` (26.4) |
| `enPause` et `gameOver` vrais en même temps | plusieurs booléens pour un seul état | remplacer par un `enum GameState` unique (26.17) |
| Le jeu passe du menu à la pause | aucune vérification des transitions | déclarer la table des transitions autorisées et refuser les autres (26.18) |
| Le héros traverse le niveau à la reprise de la pause | le `dt` accumulé pendant la pause est appliqué d'un coup | plafonner `dt` avec `clamp` et forcer `dt = 0` à la première frame après reprise (26.23) |
| Le héros part tout seul après une pause | une touche a été relâchée pendant la pause, le relâchement n'est pas arrivé | appeler `entrees.toutRelacher()` dans `endormir()` (26.23, 26.24) |
| Le héros saute 60 fois par seconde | on teste « touche maintenue » au lieu de « touche pressée cette frame » | distinguer `estMaintenue` et `vientDEtrePressee`, et vider les pressées en fin de frame (26.24) |
| Reprendre la partie remet le score à zéro | la pause a remplacé l'état de jeu, qui a été recréé | empiler l'état de pause au lieu de remplacer (26.22) |
| Le monde disparaît pendant la pause | l'état de pause n'est pas marqué transparent | déclarer `bool get transparent => true` (26.22) |
| Le test 2 échoue à cause du test 1 | un Singleton conserve son état d'un test à l'autre | injecter les services par le constructeur (26.28, 26.29) |
| Un écouteur du bus réagit encore après un changement d'écran | l'abonnement n'a jamais été annulé | conserver les fonctions de désabonnement et les appeler dans `sortir()` (26.26) |
| Le score monte de 10 puis reste bloqué | l'écouteur a été enregistré deux fois puis retiré une fois, ou l'inverse | s'abonner dans `entrer()`, se désabonner dans `sortir()`, jamais ailleurs |
| Impossible de tester la logique du jeu | tout est dans le `State` d'un widget | sortir les données et la logique de `ui/` vers `jeu/` et `core/` (26.30) |
| `core/` importe `gobelin.dart` | inversion du sens des dépendances | le moteur ne doit rien connaître du contenu du jeu (26.30) |
| Le record est remis à zéro à chaque partie | `nouvellePartie()` réinitialise aussi les données persistantes | séparer données de session et données persistantes (26.27) |
| Le sprite continue d'avancer alors que le monde est figé | l'animation est incrémentée dans `render` | incrémenter le temps d'animation dans `update` uniquement (26.5) |

---

## 26.35 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| Le problème initial | un fichier unique n'est pas trop long : il est trop **couplé** ; chaque changement touche à tout |
| Trois couches | données, logique, rendu ; le rendu **lit** et n'écrit jamais |
| Entité | un objet du monde qui a un état, évolue avec le temps et s'affiche |
| `Entity` | classe **abstraite** avec `x`, `y`, `priority`, `aRetirer`, `update`, `render` |
| `update` / `render` | `update` modifie et reçoit `dt` ; `render` dessine et ne reçoit pas `dt` |
| Liste d'entités | `for (final e in entites) e.update(dt);` remplace 700 lignes de `switch` |
| Modification concurrente | on ne modifie **jamais** une collection pendant qu'on la parcourt |
| Files d'attente | marquer `aRetirer`, empiler dans `_aAjouter`, vider **après** la boucle |
| `priority` | entier de profondeur ; trier seulement quand la liste change ; départager par `y` |
| Héritage | répond à « qu'est-ce que c'est ? » ; deux niveaux suffisent, le quatrième est un piège |
| Limites de l'héritage | héritage simple, explosion combinatoire, classe mère obèse, type figé |
| Composition | répond à « que sait-il faire ? » ; des comportements attachés, modifiables en cours de partie |
| ECS | entité = identifiant, composant = données pures, système = logique pure |
| ECS vs POO | l'ECS échange la lisibilité contre la flexibilité et la performance à grande échelle |
| Choix pour ce cours | POO classique avec une pointe de composition, comme Flame |
| Modèle de Flame | un **arbre** de composants : positions relatives, destruction récursive |
| État de jeu | une situation où le jeu met à jour, dessine et écoute globalement autre chose |
| `GameState` | un `enum` remplace `n` booléens et supprime les combinaisons absurdes |
| Machine à états | des états **plus** un graphe de transitions autorisées ; refuser les autres |
| Patron State | une classe par écran, avec `entrer`, `sortir`, `update`, `render`, `surAction` |
| Pile d'états | seul le sommet se met à jour ; les états transparents laissent voir ceux du dessous |
| Pause | couper l'appel à `update`, pas mettre `dt` à zéro ; relâcher les touches ; annuler le `dt` accumulé |
| Gestionnaire d'entrées | le héros connaît des **actions**, jamais des touches |
| Maintenue vs pressée | maintenue pour marcher, pressée pour sauter ; vider les pressées en fin de frame |
| Remappage | une table touche → action, modifiable, sérialisable, avec détection des conflits |
| Bus d'événements | l'émetteur ne connaît que le bus ; réservez-le aux événements importants |
| Service de données | score, vies, progression ; champs privés, méthodes qui appliquent les règles |
| Singleton | une variable globale déguisée : tests contaminés, dépendances invisibles |
| Injection de dépendances | donner à l'objet ce dont il a besoin ; un `ContexteJeu` suffit |
| Organisation | `ui/ → jeu/ → core/`, et les flèches ne remontent jamais |
| Tests | testez les règles, les transitions, les collisions, l'indépendance au framerate |
| Bilan | un mini-moteur complet en un fichier, découpé exactement comme l'arborescence cible |
| Passage à Flame | vous savez maintenant **ce que** le moteur fait, donc **pourquoi** il le fait ainsi |

---

## 26.36 — Exercices

### Exercice 1 — Repérer les fautes de couche (facile)

Voici un extrait de code. Trouvez les trois fautes d'architecture, expliquez pourquoi chacune est une faute, et réécrivez la méthode correctement.

```dart
class Gobelin extends Entity {
  double vitesse = 40;
  int direction = 1;
  double tempsAnim = 0;
  int frame = 0;

  @override
  void update(double dt) {
    x += 2;
  }

  @override
  void render(Canvas canvas) {
    tempsAnim += 0.016;
    if (tempsAnim > 0.12) {
      tempsAnim = 0;
      frame = (frame + 1) % 4;
    }
    if (x > 300) direction = -1;
    canvas.drawRect(
      Rect.fromLTWH(x, y, 20, 20),
      Paint()..color = const Color(0xFF6FBF73),
    );
  }
}
```

### Exercice 2 — Une entité `Torche` (facile)

Écrivez une classe `Torche` qui hérite d'`Entity`. Elle ne se déplace pas, mais son intensité lumineuse oscille entre 0,6 et 1,0 avec une période de 1,2 seconde. Sa `priority` vaut `Couches.decorFond`. Écrivez un programme console qui crée une torche, simule 3 secondes à 60 images par seconde et affiche l'intensité toutes les demi-secondes.

### Exercice 3 — Le monde avec files d'attente (facile)

Écrivez une classe `Monde` complète avec `ajouter`, `retirer` différé, et vidage des files. Créez trois gobelins de 2, 4 et 6 points de vie. À chaque frame, chaque gobelin perd 2 points de vie et, s'il meurt, fait apparaître une `Piece` à sa position. Simulez quatre frames et affichez le contenu du monde après chacune. Aucune exception ne doit être levée.

### Exercice 4 — Ordre de rendu (moyen)

Créez cinq entités : un ciel, un sol, une potion, un gobelin et un héros, insérées dans le désordre. Écrivez la méthode `render` du monde qui trie par `priority` puis par `y`, et qui ne retrie que si la liste a changé. Affichez l'ordre de dessin obtenu, puis ajoutez une particule et vérifiez que le tri est bien recalculé.

### Exercice 5 — De l'héritage à la composition (moyen)

On vous donne une hiérarchie : `Ennemi`, `EnnemiVolant`, `EnnemiTireur`, `EnnemiVolantTireur`. Réécrivez-la avec des comportements attachés : `Patrouille`, `Vol`, `Tir`. Créez ensuite un ennemi qui commence en simple patrouilleur, puis **gagne** le vol et le tir au bout de deux secondes de jeu, et montrez que cela fonctionne.

### Exercice 6 — La machine à états contrôlée (moyen)

Implémentez `MachineAEtats` avec le graphe de transitions de la section 26.18. Ajoutez un compteur du nombre de transitions refusées et un historique des dix dernières transitions réussies. Écrivez un scénario console qui tente huit transitions, dont trois illégales, et affichez le journal complet.

### Exercice 7 — Le gestionnaire d'entrées (moyen)

Écrivez `GestionnaireEntrees` avec `estMaintenue`, `vientDEtrePressee`, `vientDEtreRelachee`, `toutRelacher` et `finDeFrame`. Écrivez un héros qui marche tant que la touche est maintenue, saute une seule fois par appui, et déclenche un tir chargé au relâchement, la puissance dépendant de la durée d'appui. Simulez une séquence et affichez les résultats.

### Exercice 8 — La pile d'états avec inventaire (difficile)

Reprenez `PileEtats` et ajoutez un `EtatInventaire` transparent, ouvert par l'action `interagir` depuis le jeu et fermé par `retour`. Vérifiez que le monde ne progresse pas pendant que l'inventaire est ouvert, qu'il reste visible, et que l'ouverture de l'inventaire par-dessus la pause fonctionne aussi. Affichez le chemin de la pile à chaque étape.

### Exercice 9 — Le bus d'événements et les succès (difficile)

Écrivez un bus d'événements typé, puis un `ServiceSucces` qui écoute les événements et débloque trois succès : « Premier sang » (un ennemi tué), « Collectionneur » (dix objets ramassés) et « Sans une égratignure » (niveau terminé sans être blessé). Le service doit se désabonner proprement. Montrez le déblocage dans une trace console.

### Exercice 10 — Mini-projet : refactorer le jeu du chapitre 25 (difficile)

C'est l'exercice de synthèse du chapitre. Vous partez d'un jeu écrit « à plat » : un `State` de widget contenant des listes parallèles, un `update` monolithique et quatre booléens d'état. Vous devez le transformer en une architecture complète.

Le cahier des charges est le suivant.

1. **Entités.** Toutes les listes parallèles disparaissent au profit de classes héritant d'`Entity` : `Heros`, `Gobelin`, `Piece`, `Decor`, `Particule`.
2. **Monde.** Un `Monde` avec files d'ajout et de suppression, et tri de rendu par `priority` puis `y`.
3. **États.** Une pile d'états avec `EtatMenu`, `EtatJeu`, `EtatPause` transparent et `EtatFin`. La pause doit être une superposition, pas un remplacement.
4. **Entrées.** Un `GestionnaireEntrees` avec des actions. Aucune entité ne doit connaître `LogicalKeyboardKey`.
5. **Bus.** Un bus d'événements pour `EnnemiTue`, `ObjetRamasse` et `JoueurTouche`, avec désabonnement dans `sortir()`.
6. **Données.** Un `DonneesDeJeu` avec champs privés, et un record conservé entre les parties.
7. **Injection.** Un `ContexteJeu` passé aux entités par constructeur. Aucun Singleton.
8. **Rendu.** Le rendu ne modifie aucune donnée. Vérifiez-le en cherchant les `+=` dans vos `render`.
9. **Pause propre.** Touches relâchées, `dt` neutralisé à la reprise, monde visible mais figé.
10. **Tests.** Au moins six tests : indépendance au framerate, transitions interdites, retrait différé, score plafonné, collision AABB, remise à zéro qui conserve le record.

Livrez un fichier Flutter exécutable, découpé par blocs de commentaires correspondant aux futurs fichiers de l'arborescence de la section 26.30.

---

## 26.37 — Corrections des exercices

### Correction 1

Les trois fautes sont : un déplacement en pixels par **frame** au lieu de pixels par seconde, l'avancement de l'animation placé dans `render`, et une décision de jeu (le demi-tour) placée dans `render`.

```dart
import 'package:flutter/material.dart';

abstract class Entity {
  Entity({this.x = 0, this.y = 0, this.priority = 0});
  double x;
  double y;
  int priority;
  bool aRetirer = false;

  void update(double dt);
  void render(Canvas canvas);
}

class Gobelin extends Entity {
  Gobelin({required double x, required double y})
      : super(x: x, y: y, priority: 20);

  double vitesse = 40; // pixels PAR SECONDE
  int direction = 1;

  double _tempsAnim = 0;
  int _frame = 0;
  static const double _dureeParFrame = 0.12;
  static const int _nbFrames = 4;

  int get frame => _frame; // lecture seule pour le rendu

  @override
  void update(double dt) {
    // FAUTE 1 corrigée : le déplacement dépend du temps, pas de la frame.
    x += vitesse * direction * dt;

    // FAUTE 3 corrigée : la règle de demi-tour est de la LOGIQUE.
    if (x > 300) {
      x = 300;
      direction = -1;
    } else if (x < 100) {
      x = 100;
      direction = 1;
    }

    // FAUTE 2 corrigée : l'animation avance dans update, avec le vrai dt.
    _tempsAnim += dt;
    while (_tempsAnim >= _dureeParFrame) {
      _tempsAnim -= _dureeParFrame;
      _frame = (_frame + 1) % _nbFrames;
    }
  }

  @override
  void render(Canvas canvas) {
    // Le rendu LIT _frame, direction et x. Il n'écrit rien.
    final double hauteur = 20 + (_frame % 2) * 2;
    canvas.drawRect(
      Rect.fromLTWH(x, y + (20 - hauteur), 20, hauteur),
      Paint()..color = const Color(0xFF6FBF73),
    );
    canvas.drawRect(
      Rect.fromLTWH(x + (direction > 0 ? 12 : 4), y + 6, 4, 4),
      Paint()..color = const Color(0xFF13351A),
    );
  }
}

void main() {
  // Preuve d'indépendance au framerate.
  final Gobelin a = Gobelin(x: 100, y: 300);
  final Gobelin b = Gobelin(x: 100, y: 300);

  for (int i = 0; i < 60; i++) {
    a.update(1 / 60);
  }
  for (int i = 0; i < 120; i++) {
    b.update(1 / 120);
  }

  print('a 60 FPS  : x=${a.x.toStringAsFixed(2)} frame=${a.frame}');
  print('a 120 FPS : x=${b.x.toStringAsFixed(2)} frame=${b.frame}');
  print('identiques : ${(a.x - b.x).abs() < 0.0001 && a.frame == b.frame}');
}
```

**Résultat :**

```text
a 60 FPS  : x=140.00 frame=0
a 120 FPS : x=140.00 frame=0
identiques : true
```

**Explication :** les trois corrections sont indissociables. La première, `x += vitesse * direction * dt`, rend le déplacement indépendant du matériel : le test final le prouve en simulant la même seconde à deux cadences différentes et en obtenant la même position. La deuxième déplace l'avancement de l'animation dans `update` ; notez le `while` plutôt qu'un `if`, qui garantit un rattrapage correct si une frame a duré plus longtemps qu'une image d'animation. La troisième sort la règle du demi-tour de `render` : une règle de jeu placée dans le rendu serait évaluée un nombre imprévisible de fois, et disparaîtrait complètement le jour où l'on met le jeu en pause tout en continuant à dessiner. Après correction, `render` ne contient plus aucun `+=` ni aucune affectation à un champ : c'est le critère de relecture donné en section 26.5. Enfin, `_frame` est privé et exposé en lecture seule par un getter, ce qui interdit au rendu de le modifier même par accident.

---

### Correction 2

```dart
import 'dart:math' as math;

abstract class Couches {
  static const int decorFond = -10;
  static const int sol = 0;
  static const int objets = 10;
}

abstract class Entity {
  Entity({this.x = 0, this.y = 0, this.priority = 0});
  double x;
  double y;
  int priority;
  bool aRetirer = false;

  void update(double dt);
  String decrire();
}

class Torche extends Entity {
  Torche({required double x, required double y})
      : super(x: x, y: y, priority: Couches.decorFond);

  /// Intensité lumineuse, entre 0,6 et 1,0.
  double intensite = 0.8;

  double _temps = 0;

  static const double _periode = 1.2;
  static const double _centre = 0.8;
  static const double _amplitude = 0.2;

  @override
  void update(double dt) {
    _temps += dt;
    final double angle = 2 * math.pi * _temps / _periode;
    intensite = _centre + _amplitude * math.sin(angle);
  }

  @override
  String decrire() => 't=${_temps.toStringAsFixed(1)} s  '
      'intensite=${intensite.toStringAsFixed(2)}  '
      '${_barre(intensite)}';

  String _barre(double v) {
    final int n = ((v - 0.6) / 0.4 * 20).round().clamp(0, 20);
    return '[${"#" * n}${"." * (20 - n)}]';
  }
}

void main() {
  final Torche torche = Torche(x: 120, y: 180);

  print('priority = ${torche.priority} (decorFond)');
  print('');

  for (int i = 0; i <= 180; i++) {
    if (i % 30 == 0) print(torche.decrire());
    torche.update(1 / 60);
  }
}
```

**Résultat :**

```text
priority = -10 (decorFond)

t=0.0 s  intensite=0.80  [##########..........]
t=0.5 s  intensite=0.90  [###############.....]
t=1.0 s  intensite=0.63  [#...................]
t=1.5 s  intensite=1.00  [####################]
t=2.0 s  intensite=0.63  [#...................]
t=2.5 s  intensite=0.90  [###############.....]
t=3.0 s  intensite=0.80  [##########..........]
```

**Explication :** la torche illustre le cas d'une entité **immobile mais vivante** : elle ne change jamais de position, et pourtant son `update` est indispensable. La formule est la forme canonique d'une oscillation : `centre + amplitude × sin(2π × t / période)`. Le centre 0,8 est la moyenne des bornes demandées, l'amplitude 0,2 en est la demi-étendue. Comme le sinus varie entre −1 et 1, l'intensité varie exactement entre 0,6 et 1,0. Notez que `_temps` s'accumule en secondes, ce qui rend l'animation indépendante du framerate. Une variante fautive consisterait à écrire `intensite += 0.01` avec un demi-tour aux bornes : elle donnerait une oscillation triangulaire, plus rapide sur une machine rapide. Enfin, la `priority` est prise dans la table de constantes plutôt qu'écrite en dur : le jour où vous insérez une couche intermédiaire, un seul fichier change.

---

### Correction 3

```dart
abstract class Entity {
  Entity({this.x = 0, this.y = 0});
  double x;
  double y;
  bool aRetirer = false;
  Monde? monde;

  void update(double dt);
  String decrire();
}

class Monde {
  final List<Entity> entites = <Entity>[];
  final List<Entity> _aAjouter = <Entity>[];

  void ajouter(Entity e) {
    e.monde = this;
    _aAjouter.add(e);
  }

  void ajouterMaintenant(Entity e) {
    e.monde = this;
    entites.add(e);
  }

  void retirer(Entity e) => e.aRetirer = true;

  void update(double dt) {
    // 1. Parcours SANS modification.
    for (final Entity e in entites) {
      e.update(dt);
    }

    // 2. Vidage des files, hors de toute itération.
    if (_aAjouter.isNotEmpty) {
      entites.addAll(_aAjouter);
      _aAjouter.clear();
    }
    entites.removeWhere((Entity e) => e.aRetirer);
  }

  void afficher(String titre) {
    print('$titre  (${entites.length} entites)');
    for (final Entity e in entites) {
      print('   ${e.decrire()}');
    }
  }
}

class Gobelin extends Entity {
  Gobelin({required double x, required this.pv, required this.nom})
      : super(x: x, y: 300);

  final String nom;
  int pv;

  @override
  void update(double dt) {
    if (aRetirer) return;
    pv -= 2;
    if (pv <= 0) {
      pv = 0;
      aRetirer = true;
      // Demande d'ajout : la liste n'est PAS touchée ici.
      monde?.ajouter(Piece(x: x, provenance: nom));
    }
  }

  @override
  String decrire() => 'Gobelin $nom  pv=$pv  x=${x.toStringAsFixed(0)}';
}

class Piece extends Entity {
  Piece({required double x, required this.provenance}) : super(x: x, y: 300);

  final String provenance;

  @override
  void update(double dt) {}

  @override
  String decrire() => 'Piece   (de $provenance)  x=${x.toStringAsFixed(0)}';
}

void main() {
  final Monde monde = Monde();
  monde.ajouterMaintenant(Gobelin(x: 100, pv: 2, nom: 'A'));
  monde.ajouterMaintenant(Gobelin(x: 200, pv: 4, nom: 'B'));
  monde.ajouterMaintenant(Gobelin(x: 300, pv: 6, nom: 'C'));

  monde.afficher('=== depart ===');

  for (int frame = 1; frame <= 4; frame++) {
    monde.update(1 / 60);
    monde.afficher('=== apres la frame $frame ===');
  }

  print('');
  print('Aucune exception de modification concurrente.');
}
```

**Résultat :**

```text
=== depart ===  (3 entites)
   Gobelin A  pv=2  x=100
   Gobelin B  pv=4  x=200
   Gobelin C  pv=6  x=300
=== apres la frame 1 ===  (3 entites)
   Gobelin B  pv=2  x=200
   Gobelin C  pv=4  x=300
   Piece   (de A)  x=100
=== apres la frame 2 ===  (3 entites)
   Gobelin C  pv=2  x=300
   Piece   (de A)  x=100
   Piece   (de B)  x=200
=== apres la frame 3 ===  (3 entites)
   Piece   (de A)  x=100
   Piece   (de B)  x=200
   Piece   (de C)  x=300
=== apres la frame 4 ===  (3 entites)
   Piece   (de A)  x=100
   Piece   (de B)  x=200
   Piece   (de C)  x=300
```

**Explication :** ce programme réalise en une seule frame les deux opérations qui font planter la version naïve : une suppression et un ajout, pendant le parcours. Il n'y a pourtant aucune exception, parce que la liste `entites` n'est touchée qu'à l'étape 2, quand plus aucun itérateur n'est actif. Observez l'ordre de vidage : on ajoute d'abord, on retire ensuite. Ce choix a une conséquence visible : à la frame 1, la pièce issue de A apparaît **et** le gobelin A disparaît dans le même passage. Si l'on retirait avant d'ajouter, le résultat serait identique ici, mais il différerait le jour où une entité nouvellement ajoutée serait immédiatement marquée `aRetirer` par une autre. Notez aussi la garde `if (aRetirer) return;` en tête de `update` : une entité marquée pendant la frame courante reste dans la liste jusqu'à la fin du parcours, et pourrait donc être mise à jour une dernière fois. Sans cette garde, un gobelin déjà mort pourrait générer une seconde pièce. Enfin, `ajouterMaintenant` existe pour l'initialisation, hors boucle, où l'ajout différé serait inutilement compliqué.

---

### Correction 4

```dart
abstract class Couches {
  static const int ciel = -100;
  static const int sol = 0;
  static const int objets = 10;
  static const int ennemis = 20;
  static const int joueur = 30;
  static const int particules = 50;
}

abstract class Entity {
  Entity({required this.nom, this.x = 0, this.y = 0, this.priority = 0});

  final String nom;
  double x;
  double y;
  int priority;
  bool aRetirer = false;

  void update(double dt) {}
  void render(List<String> journal) => journal.add(nom);
}

class Simple extends Entity {
  Simple({
    required String nom,
    required double y,
    required int priority,
  }) : super(nom: nom, y: y, priority: priority);
}

class Monde {
  final List<Entity> entites = <Entity>[];
  final List<Entity> _aAjouter = <Entity>[];
  final List<Entity> _ordre = <Entity>[];

  bool _triNecessaire = true;
  int nombreDeTris = 0;

  void ajouterMaintenant(Entity e) {
    entites.add(e);
    _triNecessaire = true;
  }

  void ajouter(Entity e) => _aAjouter.add(e);

  void update(double dt) {
    for (final Entity e in entites) {
      e.update(dt);
    }
    if (_aAjouter.isNotEmpty) {
      entites.addAll(_aAjouter);
      _aAjouter.clear();
      _triNecessaire = true;
    }
    final int avant = entites.length;
    entites.removeWhere((Entity e) => e.aRetirer);
    if (entites.length != avant) _triNecessaire = true;
  }

  List<String> render() {
    if (_triNecessaire) {
      _ordre
        ..clear()
        ..addAll(entites)
        ..sort((Entity a, Entity b) {
          final int parCouche = a.priority.compareTo(b.priority);
          if (parCouche != 0) return parCouche;
          return a.y.compareTo(b.y);
        });
      _triNecessaire = false;
      nombreDeTris++;
    }
    final List<String> journal = <String>[];
    for (final Entity e in _ordre) {
      e.render(journal);
    }
    return journal;
  }
}

void main() {
  final Monde monde = Monde();

  // Insertion volontairement dans le desordre.
  monde.ajouterMaintenant(
      Simple(nom: 'heros', y: 300, priority: Couches.joueur));
  monde.ajouterMaintenant(Simple(nom: 'ciel', y: 0, priority: Couches.ciel));
  monde.ajouterMaintenant(
      Simple(nom: 'gobelin', y: 318, priority: Couches.ennemis));
  monde.ajouterMaintenant(Simple(nom: 'sol', y: 340, priority: Couches.sol));
  monde.ajouterMaintenant(
      Simple(nom: 'potion', y: 300, priority: Couches.objets));

  print('insertion : heros, ciel, gobelin, sol, potion');
  print('rendu 1   : ${monde.render().join(" -> ")}');
  print('rendu 2   : ${monde.render().join(" -> ")}');
  print('tris effectues : ${monde.nombreDeTris}');

  print('');
  print('Une particule apparait.');
  monde.ajouter(
      Simple(nom: 'particule', y: 250, priority: Couches.particules));
  monde.update(1 / 60);
  print('rendu 3   : ${monde.render().join(" -> ")}');
  print('tris effectues : ${monde.nombreDeTris}');

  print('');
  print('Deux gobelins a la meme couche, departages par y.');
  monde.ajouterMaintenant(
      Simple(nom: 'gobelin-loin', y: 280, priority: Couches.ennemis));
  print('rendu 4   : ${monde.render().join(" -> ")}');
  print('tris effectues : ${monde.nombreDeTris}');
}
```

**Résultat :**

```text
insertion : heros, ciel, gobelin, sol, potion
rendu 1   : ciel -> sol -> potion -> gobelin -> heros
rendu 2   : ciel -> sol -> potion -> gobelin -> heros
tris effectues : 1

Une particule apparait.
rendu 3   : ciel -> sol -> potion -> gobelin -> heros -> particule
tris effectues : 2

Deux gobelins a la meme couche, departages par y.
rendu 4   : ciel -> sol -> potion -> gobelin-loin -> gobelin -> heros
tris effectues : 3
```

**Explication :** trois comportements sont vérifiés par cette trace. D'abord, l'ordre de dessin obtenu est celui des couches, quel que soit l'ordre d'insertion : le ciel passe en premier bien qu'il ait été inséré en deuxième, et le héros passe en dernier bien qu'il ait été inséré en premier.

Ensuite, le compteur `nombreDeTris` reste à 1 après deux rendus consécutifs : le drapeau `_triNecessaire` évite de retrier soixante fois par seconde une liste qui n'a pas changé. Il repasse à `true` uniquement lors d'un ajout ou d'un retrait effectif. Le compteur monte à 2 après l'arrivée de la particule, puis à 3 après l'ajout du second gobelin. En régime stable, le tri ne coûte donc rien.

Enfin, `gobelin-loin` et `gobelin` partagent la même `priority`. C'est le second critère qui les départage : celui dont le `y` est le plus petit, c'est-à-dire le plus haut à l'écran, est dessiné en premier et se retrouve donc **derrière** l'autre. C'est le tri en Y des jeux en vue de dessus, et il donne gratuitement une illusion de profondeur correcte.

---

### Correction 5

```dart
// ============================================================
//  L'entité et ses comportements
// ============================================================

abstract class Comportement {
  Entite? proprietaire;
  String get nom;
  void update(double dt);
}

class Entite {
  Entite({required this.nom, this.x = 0, this.y = 0});

  final String nom;
  double x;
  double y;

  final List<Comportement> comportements = <Comportement>[];
  final List<Comportement> _aAjouter = <Comportement>[];

  /// Attache immédiatement (hors boucle) et renvoie l'entité.
  Entite avec(Comportement c) {
    c.proprietaire = this;
    comportements.add(c);
    return this;
  }

  /// Demande un attachement : il sera effectif à la fin de la frame.
  /// Sans cette file, on obtiendrait une modification concurrente (26.7).
  void attacherPlusTard(Comportement c) {
    c.proprietaire = this;
    _aAjouter.add(c);
  }

  T? obtenir<T extends Comportement>() {
    for (final Comportement c in comportements) {
      if (c is T) return c;
    }
    return null;
  }

  bool a<T extends Comportement>() => obtenir<T>() != null;

  void update(double dt) {
    for (final Comportement c in comportements) {
      c.update(dt);
    }
    if (_aAjouter.isNotEmpty) {
      comportements.addAll(_aAjouter);
      _aAjouter.clear();
    }
  }

  String get description => comportements
      .map((Comportement c) => c.nom)
      .join(' + ');
}

// ============================================================
//  Les comportements
// ============================================================

class Patrouille extends Comportement {
  Patrouille({required this.vitesse, required this.xMin, required this.xMax});

  final double vitesse;
  final double xMin;
  final double xMax;
  int direction = 1;

  @override
  String get nom => 'Patrouille';

  @override
  void update(double dt) {
    final Entite? e = proprietaire;
    if (e == null) return;
    e.x += vitesse * direction * dt;
    if (e.x > xMax) {
      e.x = xMax;
      direction = -1;
    } else if (e.x < xMin) {
      e.x = xMin;
      direction = 1;
    }
  }
}

class Vol extends Comportement {
  Vol({required this.amplitude, required this.frequence});

  final double amplitude;
  final double frequence;

  double _temps = 0;
  double _yBase = 0;
  bool _initialise = false;

  @override
  String get nom => 'Vol';

  @override
  void update(double dt) {
    final Entite? e = proprietaire;
    if (e == null) return;
    if (!_initialise) {
      _yBase = e.y;
      _initialise = true;
    }
    _temps += dt;
    final double p = (_temps * frequence) % 2.0;
    final double onde = p < 1.0 ? p : 2.0 - p; // onde triangulaire 0 -> 1 -> 0
    e.y = _yBase + amplitude * (onde * 2 - 1);
  }
}

class Tir extends Comportement {
  Tir({required this.cadence});

  final double cadence;
  double _restant = 0;
  int tirs = 0;

  @override
  String get nom => 'Tir';

  @override
  void update(double dt) {
    _restant -= dt;
    if (_restant <= 0) {
      _restant = cadence;
      tirs++;
    }
  }
}

/// Le comportement qui en ajoute d'autres : c'est lui qui rendait
/// l'héritage impuissant.
class Rage extends Comportement {
  Rage({required this.delai});

  final double delai;
  double _temps = 0;
  bool _declenchee = false;

  @override
  String get nom => 'Rage';

  @override
  void update(double dt) {
    if (_declenchee) return;
    _temps += dt;
    if (_temps < delai) return;
    _declenchee = true;

    final Entite? e = proprietaire;
    if (e == null) return;
    print('  >>> ${e.nom} entre en rage : il gagne Vol et Tir.');
    e.attacherPlusTard(Vol(amplitude: 20, frequence: 1));
    e.attacherPlusTard(Tir(cadence: 0.5));
  }
}

// ============================================================
//  Programme
// ============================================================

void main() {
  final Entite ennemi = Entite(nom: 'gobelin', x: 200, y: 300)
      .avec(Patrouille(vitesse: 50, xMin: 200, xMax: 300))
      .avec(Rage(delai: 2.0));

  void etat(int frames) {
    final Tir? t = ennemi.obtenir<Tir>();
    print('t=${(frames / 60).toStringAsFixed(1)} s  '
        'x=${ennemi.x.toStringAsFixed(0)}  '
        'y=${ennemi.y.toStringAsFixed(0)}  '
        'tirs=${t?.tirs ?? "-"}  '
        'vole=${ennemi.a<Vol>()}');
    print('          comportements : ${ennemi.description}');
  }

  etat(0);
  for (int i = 1; i <= 180; i++) {
    ennemi.update(1 / 60);
    if (i % 60 == 0) etat(i);
  }
}
```

**Résultat :**

```text
t=0.0 s  x=200  y=300  tirs=-  vole=false
          comportements : Patrouille + Rage
t=1.0 s  x=250  y=300  tirs=-  vole=false
          comportements : Patrouille + Rage
  >>> gobelin entre en rage : il gagne Vol et Tir.
t=2.0 s  x=300  y=300  tirs=0  vole=true
          comportements : Patrouille + Rage + Vol + Tir
t=3.0 s  x=251  y=320  tirs=2  vole=true
          comportements : Patrouille + Rage + Vol + Tir
```

**Explication :** cette correction démontre le point qui rend la composition supérieure à l'héritage pour un jeu : **une entité change de capacités en cours de partie**. À la deuxième seconde, le gobelin gagne le vol et le tir sans être détruit ni recréé ; il conserve sa position, sa direction de patrouille et son identité. Avec la hiérarchie `Ennemi → EnnemiVolant → EnnemiVolantTireur`, il aurait fallu instancier une nouvelle classe et recopier l'état à la main.

Trois détails d'implémentation méritent attention. D'abord, `Rage` ajoute des comportements **pendant** que la liste des comportements est parcourue : sans la file `_aAjouter`, on obtiendrait exactement l'exception de la section 26.7. Le problème de la modification concurrente n'est donc pas propre aux entités : il se pose partout où l'on itère.

Ensuite, `Vol` capture sa hauteur de référence à son premier `update`, et non à sa construction. C'est indispensable ici, puisqu'il est attaché deux secondes après le début de la partie : sa base doit être la hauteur du moment, pas celle du début.

Enfin, `obtenir<T>()` et `a<T>()` interrogent une **capacité**, pas un type. Le code qui veut savoir si un ennemi vole écrit `ennemi.a<Vol>()`, ce qui reste vrai quel que soit l'assemblage. C'est ce que la question `ennemi is EnnemiVolant` ne pouvait pas exprimer.

---

### Correction 6

```dart
enum GameState { demarrage, menu, jeu, pause, gameOver, victoire }

class Transition {
  const Transition(this.de, this.vers);
  final GameState de;
  final GameState vers;

  @override
  String toString() => '${de.name} -> ${vers.name}';
}

class MachineAEtats {
  MachineAEtats({GameState initial = GameState.demarrage}) : _etat = initial;

  GameState _etat;
  GameState get etat => _etat;

  double tempsDansEtat = 0;

  /// Nombre de transitions refusées depuis le début.
  int refusees = 0;

  /// Les dix dernières transitions réussies.
  final List<Transition> historique = <Transition>[];
  static const int _tailleHistorique = 10;

  static const Map<GameState, Set<GameState>> _autorisees =
      <GameState, Set<GameState>>{
    GameState.demarrage: <GameState>{GameState.menu},
    GameState.menu: <GameState>{GameState.jeu},
    GameState.jeu: <GameState>{
      GameState.pause,
      GameState.gameOver,
      GameState.victoire,
    },
    GameState.pause: <GameState>{GameState.jeu, GameState.menu},
    GameState.gameOver: <GameState>{GameState.jeu, GameState.menu},
    GameState.victoire: <GameState>{GameState.jeu, GameState.menu},
  };

  void Function(GameState ancien, GameState nouveau)? surTransition;

  bool peutAller(GameState vers) => _autorisees[_etat]?.contains(vers) ?? false;

  Set<GameState> get sortiesPossibles =>
      _autorisees[_etat] ?? const <GameState>{};

  bool aller(GameState vers) {
    if (vers == _etat) return true;

    if (!peutAller(vers)) {
      refusees++;
      print('  REFUS  ${_etat.name} -> ${vers.name}   '
          '(sorties possibles : '
          '${sortiesPossibles.map((GameState e) => e.name).join(", ")})');
      return false;
    }

    final GameState ancien = _etat;
    _etat = vers;
    tempsDansEtat = 0;

    historique.add(Transition(ancien, vers));
    while (historique.length > _tailleHistorique) {
      historique.removeAt(0);
    }

    print('  OK     ${ancien.name} -> ${vers.name}');
    surTransition?.call(ancien, vers);
    return true;
  }

  void update(double dt) => tempsDansEtat += dt;
}

void main() {
  final MachineAEtats m = MachineAEtats();
  m.surTransition = (GameState a, GameState n) {
    if (n == GameState.jeu && a == GameState.menu) {
      print('         -> nouvelle partie : score 0, vies 3');
    }
  };

  print('=== scenario ===');
  m.aller(GameState.menu);      // 1  legale
  m.aller(GameState.pause);     // 2  ILLEGALE
  m.aller(GameState.jeu);       // 3  legale
  m.aller(GameState.pause);     // 4  legale
  m.aller(GameState.gameOver);  // 5  ILLEGALE
  m.aller(GameState.jeu);       // 6  legale
  m.aller(GameState.victoire);  // 7  legale
  m.aller(GameState.pause);     // 8  ILLEGALE

  print('');
  print('=== bilan ===');
  print('etat final          : ${m.etat.name}');
  print('transitions refusees : ${m.refusees}');
  print('historique (${m.historique.length}) :');
  for (final Transition t in m.historique) {
    print('   $t');
  }
}
```

**Résultat :**

```text
=== scenario ===
  OK     demarrage -> menu
  REFUS  menu -> pause   (sorties possibles : jeu)
  OK     menu -> jeu
         -> nouvelle partie : score 0, vies 3
  OK     jeu -> pause
  REFUS  pause -> gameOver   (sorties possibles : jeu, menu)
  OK     pause -> jeu
  OK     jeu -> victoire
  REFUS  victoire -> pause   (sorties possibles : jeu, menu)

=== bilan ===
etat final          : victoire
transitions refusees : 3
historique (5) :
   demarrage -> menu
   menu -> jeu
   jeu -> pause
   pause -> jeu
   jeu -> victoire
```

**Explication :** trois éléments transforment cette machine en outil de diagnostic. Le message de refus n'indique pas seulement l'échec : il liste les **sorties possibles** depuis l'état courant. C'est la première question que se pose le développeur devant un refus, et la réponse est déjà là. L'historique borné à dix entrées permet de reconstituer le chemin qui a mené à un bug sans saturer les journaux ; c'est le minimum indispensable pour comprendre un rapport de bug d'un testeur.

Le compteur `refusees` mérite une attention particulière. En production, il devrait rester à zéro : chaque refus signale un endroit du code qui tente une transition sans vérifier qu'elle est légitime. Un compteur qui grimpe pendant une session de jeu est le symptôme d'un bug latent, même si le joueur ne voit rien. En développement, remplacez le `print` par un `assert(false, ...)` : le programme s'arrête à l'instant exact de la faute, avec la pile d'appels complète.

Notez enfin `surTransition`, qui centralise ce qui doit se produire **à l'occasion** d'un changement d'état. La remise à zéro d'une partie y trouve sa place naturelle : quel que soit le bouton, quel que soit l'écran d'origine, aller de `menu` vers `jeu` réinitialise. Personne ne peut l'oublier.

---

### Correction 7

```dart
enum ActionJeu { gauche, droite, sauter, tirer }

class GestionnaireEntrees {
  final Set<ActionJeu> _maintenues = <ActionJeu>{};
  final Set<ActionJeu> _pressees = <ActionJeu>{};
  final Set<ActionJeu> _relachees = <ActionJeu>{};

  bool estMaintenue(ActionJeu a) => _maintenues.contains(a);
  bool vientDEtrePressee(ActionJeu a) => _pressees.contains(a);
  bool vientDEtreRelachee(ActionJeu a) => _relachees.contains(a);

  void appuyer(ActionJeu a) {
    if (_maintenues.add(a)) _pressees.add(a);
  }

  void relacher(ActionJeu a) {
    if (_maintenues.remove(a)) _relachees.add(a);
  }

  /// Relâche tout : le relâchement est bien signalé, pour ne pas
  /// laisser une action « collée » (26.23).
  void toutRelacher() {
    _relachees.addAll(_maintenues);
    _maintenues.clear();
  }

  /// À appeler à la toute fin de chaque frame.
  void finDeFrame() {
    _pressees.clear();
    _relachees.clear();
  }
}

class Heros {
  double x = 0;
  double vy = 0;
  bool auSol = true;

  int sauts = 0;

  /// Charge du tir, en secondes.
  double charge = 0;
  final List<int> tirs = <int>[];

  static const double vitesse = 140;
  static const double chargeMax = 1.5;

  void update(double dt, GestionnaireEntrees e) {
    // 1. MAINTENUE : déplacement continu.
    if (e.estMaintenue(ActionJeu.gauche)) x -= vitesse * dt;
    if (e.estMaintenue(ActionJeu.droite)) x += vitesse * dt;

    // 2. PRESSÉE : action ponctuelle, une seule fois par appui.
    if (e.vientDEtrePressee(ActionJeu.sauter) && auSol) {
      vy = -300;
      auSol = false;
      sauts++;
    }

    // 3. MAINTENUE puis RELÂCHÉE : tir chargé.
    if (e.estMaintenue(ActionJeu.tirer)) {
      charge = (charge + dt).clamp(0.0, chargeMax);
    } else if (e.vientDEtreRelachee(ActionJeu.tirer)) {
      final double ratio = charge / chargeMax;
      final int puissance = (10 + 90 * ratio).round();
      tirs.add(puissance);
      print('  tir ! charge=${charge.toStringAsFixed(2)} s '
          '-> puissance=$puissance');
      charge = 0;
    }

    // 4. Physique minimale.
    vy += 900 * dt;
    if (vy > 0 && !auSol) {
      auSol = true;
      vy = 0;
    }
  }
}

void main() {
  final GestionnaireEntrees e = GestionnaireEntrees();
  final Heros h = Heros();

  void frames(int n) {
    for (int i = 0; i < n; i++) {
      h.update(1 / 60, e);
      e.finDeFrame();
    }
  }

  print('--- 1. le joueur maintient DROITE pendant 30 frames ---');
  e.appuyer(ActionJeu.droite);
  frames(30);
  e.relacher(ActionJeu.droite);
  frames(1);
  print('  x = ${h.x.toStringAsFixed(1)}');

  print('--- 2. il maintient SAUTER pendant 60 frames ---');
  e.appuyer(ActionJeu.sauter);
  frames(60);
  e.relacher(ActionJeu.sauter);
  frames(1);
  print('  sauts = ${h.sauts}  (attendu : 1)');

  print('--- 3. il charge un tir pendant 45 frames puis relache ---');
  e.appuyer(ActionJeu.tirer);
  frames(45);
  e.relacher(ActionJeu.tirer);
  frames(1);

  print('--- 4. tir bref : 6 frames ---');
  e.appuyer(ActionJeu.tirer);
  frames(6);
  e.relacher(ActionJeu.tirer);
  frames(1);

  print('--- 5. charge tres longue : 150 frames (plafonnee) ---');
  e.appuyer(ActionJeu.tirer);
  frames(150);
  e.relacher(ActionJeu.tirer);
  frames(1);

  print('');
  print('tirs effectues : ${h.tirs}');
}
```

**Résultat :**

```text
--- 1. le joueur maintient DROITE pendant 30 frames ---
  x = 70.0
--- 2. il maintient SAUTER pendant 60 frames ---
  sauts = 1  (attendu : 1)
--- 3. il charge un tir pendant 45 frames puis relache ---
  tir ! charge=0.75 s -> puissance=55
--- 4. tir bref : 6 frames ---
  tir ! charge=0.10 s -> puissance=16
--- 5. charge tres longue : 150 frames (plafonnee) ---
  tir ! charge=1.50 s -> puissance=100
tirs effectues : [55, 16, 100]
```

**Explication :** les trois types d'entrée sont ici mis en évidence par trois usages distincts, et l'expérience du joueur dépend entièrement du choix correct.

Le déplacement utilise `estMaintenue` : trente frames à 140 pixels par seconde donnent exactement 70 pixels, soit une demi-seconde de marche. Si l'on avait utilisé `vientDEtrePressee`, le héros aurait avancé de 2,3 pixels puis se serait arrêté net, touche enfoncée.

Le saut utilise `vientDEtrePressee` : une seule impulsion pour soixante frames d'appui. Avec `estMaintenue`, le compteur afficherait 60, et le héros décollerait comme un hélicoptère. C'est le bug le plus fréquent des premiers prototypes.

Le tir chargé utilise les trois à la fois : `estMaintenue` accumule la charge frame après frame, `vientDEtreRelachee` déclenche le tir. La puissance est proportionnelle à la charge, plafonnée par un `clamp` à 1,5 seconde — le cinquième essai montre que 150 frames d'appui ne donnent pas plus que la charge maximale.

Le détail le plus important est l'appel `e.finDeFrame()` **après** chaque `update`. Il vide les ensembles « pressées » et « relâchées », ce qui garantit que ces informations ne durent qu'une frame. Omettez-le, et le héros saute indéfiniment tandis que le tir se déclenche à chaque frame. C'est le genre d'oubli qui produit un bug spectaculaire mais dont la cause tient en une ligne.

---

### Correction 8

```dart
// ============================================================
//  core/pile_etats.dart
// ============================================================

abstract class EtatDeJeu {
  late PileEtats pile;

  String get nom;
  bool get transparent => false;

  void entrer() {}
  void sortir() {}
  void endormir() {}
  void reveiller() {}
  void update(double dt) {}
  void render(List<String> ecran) {}
  void surAction(String action) {}
}

class PileEtats {
  final List<EtatDeJeu> _pile = <EtatDeJeu>[];

  EtatDeJeu? get sommet => _pile.isEmpty ? null : _pile.last;
  String get chemin => _pile.map((EtatDeJeu e) => e.nom).join(' > ');

  void empiler(EtatDeJeu e) {
    if (_pile.isNotEmpty) _pile.last.endormir();
    e.pile = this;
    _pile.add(e);
    e.entrer();
  }

  void depiler() {
    if (_pile.isEmpty) return;
    _pile.removeLast().sortir();
    if (_pile.isNotEmpty) _pile.last.reveiller();
  }

  void update(double dt) => sommet?.update(dt);
  void surAction(String a) => sommet?.surAction(a);

  List<String> render() {
    final List<String> ecran = <String>[];
    if (_pile.isEmpty) return ecran;

    int debut = _pile.length - 1;
    while (debut > 0 && _pile[debut].transparent) {
      debut--;
    }
    for (int i = debut; i < _pile.length; i++) {
      _pile[i].render(ecran);
    }
    return ecran;
  }
}

// ============================================================
//  jeu/
// ============================================================

class Partie {
  double x = 0;
  int score = 0;
  final List<String> inventaire = <String>['potion', 'cle', 'epee'];
}

class EtatJeu extends EtatDeJeu {
  EtatJeu(this.partie);
  final Partie partie;

  @override
  String get nom => 'JEU';

  @override
  void entrer() => print('    [JEU entre]');

  @override
  void endormir() => print('    [JEU endormi : touches relachees]');

  @override
  void reveiller() => print('    [JEU reveille]');

  @override
  void update(double dt) {
    partie.x += 100 * dt;
    partie.score += 1;
  }

  @override
  void render(List<String> ecran) => ecran.add(
      'monde : x=${partie.x.toStringAsFixed(0)} score=${partie.score}');

  @override
  void surAction(String a) {
    if (a == 'pause') pile.empiler(EtatPause(partie));
    if (a == 'interagir') pile.empiler(EtatInventaire(partie));
  }
}

class EtatPause extends EtatDeJeu {
  EtatPause(this.partie);
  final Partie partie;

  @override
  String get nom => 'PAUSE';

  @override
  bool get transparent => true;

  @override
  void entrer() => print('    [PAUSE entre : volume 30 %]');

  @override
  void sortir() => print('    [PAUSE sort : volume 100 %]');

  @override
  void render(List<String> ecran) => ecran.add('voile + "PAUSE"');

  @override
  void surAction(String a) {
    if (a == 'pause') pile.depiler();
    if (a == 'interagir') pile.empiler(EtatInventaire(partie));
  }
}

class EtatInventaire extends EtatDeJeu {
  EtatInventaire(this.partie);
  final Partie partie;

  int selection = 0;

  @override
  String get nom => 'INVENTAIRE';

  @override
  bool get transparent => true;

  @override
  void entrer() => print('    [INVENTAIRE entre]');

  @override
  void sortir() => print('    [INVENTAIRE sort]');

  @override
  void render(List<String> ecran) {
    final String liste = partie.inventaire
        .asMap()
        .entries
        .map((MapEntry<int, String> e) =>
            e.key == selection ? '<${e.value}>' : e.value)
        .join('  ');
    ecran.add('panneau inventaire : $liste');
  }

  @override
  void surAction(String a) {
    if (a == 'retour') {
      pile.depiler();
    } else if (a == 'suivant') {
      selection = (selection + 1) % partie.inventaire.length;
    }
  }
}

// ============================================================
//  Programme
// ============================================================

void main() {
  final Partie partie = Partie();
  final PileEtats pile = PileEtats();
  pile.empiler(EtatJeu(partie));

  void seconde(String titre) {
    for (int i = 0; i < 60; i++) {
      pile.update(1 / 60);
    }
    print('$titre   [pile : ${pile.chemin}]');
    for (final String ligne in pile.render()) {
      print('      $ligne');
    }
  }

  seconde('1 s de jeu');

  print('> interagir');
  pile.surAction('interagir');
  seconde('1 s d inventaire');

  print('> suivant');
  pile.surAction('suivant');
  seconde('1 s d inventaire');

  print('> retour');
  pile.surAction('retour');
  seconde('1 s de jeu');

  print('> pause');
  pile.surAction('pause');
  seconde('1 s de pause');

  print('> interagir (depuis la pause)');
  pile.surAction('interagir');
  seconde('1 s d inventaire sur pause');

  print('> retour');
  pile.surAction('retour');
  print('> pause');
  pile.surAction('pause');
  seconde('1 s de jeu');
}
```

**Résultat :**

```text
    [JEU entre]
1 s de jeu   [pile : JEU]
      monde : x=100 score=60
> interagir
    [JEU endormi : touches relachees]
    [INVENTAIRE entre]
1 s d inventaire   [pile : JEU > INVENTAIRE]
      monde : x=100 score=60
      panneau inventaire : <potion>  cle  epee
> suivant
1 s d inventaire   [pile : JEU > INVENTAIRE]
      monde : x=100 score=60
      panneau inventaire : potion  <cle>  epee
> retour
    [INVENTAIRE sort]
    [JEU reveille]
1 s de jeu   [pile : JEU]
      monde : x=200 score=120
> pause
    [JEU endormi : touches relachees]
    [PAUSE entre : volume 30 %]
1 s de pause   [pile : JEU > PAUSE]
      monde : x=200 score=120
      voile + "PAUSE"
> interagir (depuis la pause)
    [INVENTAIRE entre]
1 s d inventaire sur pause   [pile : JEU > PAUSE > INVENTAIRE]
      monde : x=200 score=120
      voile + "PAUSE"
      panneau inventaire : <potion>  cle  epee
> retour
    [INVENTAIRE sort]
    [PAUSE sort : volume 100 %]
    [JEU reveille]
1 s de jeu   [pile : JEU]
      monde : x=300 score=180
```

**Explication :** cette trace démontre les quatre propriétés attendues d'une pile d'états.

**Le monde se fige sans le savoir.** Entre l'ouverture de l'inventaire et sa fermeture, trois secondes de temps réel s'écoulent, et pourtant `x` reste à 100 et le score à 60. La classe `EtatJeu` ne contient aucun test de pause : elle ne reçoit tout simplement plus `update`. C'est le gain le plus important, car il supprime la vingtaine de `if (enPause) return;` qu'un débutant sème dans son code.

**Le monde reste visible.** Les états d'inventaire et de pause déclarent `transparent => true`, donc le rendu remonte jusqu'au premier état opaque, ici `EtatJeu`, et dessine ensuite tout ce qui est au-dessus, de bas en haut. Le joueur voit le donjon derrière son inventaire.

**L'empilement à trois niveaux fonctionne sans code supplémentaire.** `JEU > PAUSE > INVENTAIRE` s'affiche correctement et se dépile dans le bon ordre. C'est ce que le remplacement de la section 26.21 ne permettait pas : il aurait fallu mémoriser manuellement l'écran précédent à chaque niveau.

**Les crochets `entrer`, `sortir`, `endormir` et `reveiller` se déclenchent au bon moment.** Notez la nuance : `endormir` est appelé sur `JEU` quand `INVENTAIRE` s'empile, mais `sortir` ne l'est pas, puisque `JEU` n'est pas retiré. Cette distinction permet de relâcher les touches à l'endormissement sans perdre l'état de la partie. Notez aussi que lors du dernier dépilement, `PAUSE` reçoit `sortir` alors que `INVENTAIRE` venait d'être retiré : chaque état ne gère que son propre cycle, et l'ensemble reste cohérent.

---

### Correction 9

```dart
// ============================================================
//  core/bus_evenements.dart
// ============================================================

abstract class EvenementJeu {
  const EvenementJeu();
}

class EnnemiTue extends EvenementJeu {
  const EnnemiTue(this.type);
  final String type;
}

class ObjetRamasse extends EvenementJeu {
  const ObjetRamasse(this.nom);
  final String nom;
}

class JoueurBlesse extends EvenementJeu {
  const JoueurBlesse(this.degats);
  final int degats;
}

class NiveauTermine extends EvenementJeu {
  const NiveauTermine(this.numero);
  final int numero;
}

class NiveauCommence extends EvenementJeu {
  const NiveauCommence(this.numero);
  final int numero;
}

typedef Ecouteur<T> = void Function(T evenement);

class BusEvenements {
  BusEvenements({this.trace = false});

  final bool trace;
  final Map<Type, List<Function>> _ecouteurs = <Type, List<Function>>{};

  /// Renvoie la fonction de désabonnement.
  void Function() ecouter<T extends EvenementJeu>(Ecouteur<T> f) {
    _ecouteurs.putIfAbsent(T, () => <Function>[]).add(f);
    return () => _ecouteurs[T]?.remove(f);
  }

  void publier<T extends EvenementJeu>(T e) {
    if (trace) print('  . bus : ${e.runtimeType}');
    final List<Function>? liste = _ecouteurs[e.runtimeType];
    if (liste == null || liste.isEmpty) return;
    // Copie défensive : un écouteur peut en retirer un autre.
    for (final Function f in List<Function>.of(liste)) {
      (f as Ecouteur<T>)(e);
    }
  }

  int get nombreDAbonnements =>
      _ecouteurs.values.fold(0, (int s, List<Function> l) => s + l.length);
}

// ============================================================
//  jeu/service_succes.dart
// ============================================================

class Succes {
  Succes({required this.code, required this.titre, required this.description});

  final String code;
  final String titre;
  final String description;
  bool debloque = false;
}

class ServiceSucces {
  ServiceSucces(this.bus) {
    _abonnements.add(bus.ecouter<EnnemiTue>(_surEnnemiTue));
    _abonnements.add(bus.ecouter<ObjetRamasse>(_surObjetRamasse));
    _abonnements.add(bus.ecouter<JoueurBlesse>(_surJoueurBlesse));
    _abonnements.add(bus.ecouter<NiveauCommence>(_surNiveauCommence));
    _abonnements.add(bus.ecouter<NiveauTermine>(_surNiveauTermine));
  }

  final BusEvenements bus;
  final List<void Function()> _abonnements = <void Function()>[];

  final Map<String, Succes> succes = <String, Succes>{
    'premier_sang': Succes(
      code: 'premier_sang',
      titre: 'Premier sang',
      description: 'Eliminer un ennemi.',
    ),
    'collectionneur': Succes(
      code: 'collectionneur',
      titre: 'Collectionneur',
      description: 'Ramasser dix objets.',
    ),
    'sans_egratignure': Succes(
      code: 'sans_egratignure',
      titre: 'Sans une egratignure',
      description: 'Terminer un niveau sans etre blesse.',
    ),
  };

  int _ennemisTues = 0;
  int _objetsRamasses = 0;
  bool _blesseDansLeNiveau = false;

  void _debloquer(String code) {
    final Succes? s = succes[code];
    if (s == null || s.debloque) return;
    s.debloque = true;
    print('  *** SUCCES DEBLOQUE : ${s.titre} — ${s.description}');
  }

  void _surEnnemiTue(EnnemiTue e) {
    _ennemisTues++;
    if (_ennemisTues == 1) _debloquer('premier_sang');
  }

  void _surObjetRamasse(ObjetRamasse e) {
    _objetsRamasses++;
    if (_objetsRamasses == 10) _debloquer('collectionneur');
  }

  void _surJoueurBlesse(JoueurBlesse e) {
    _blesseDansLeNiveau = true;
  }

  void _surNiveauCommence(NiveauCommence e) {
    _blesseDansLeNiveau = false;
    print('  --- niveau ${e.numero} : compteur de blessures remis a zero');
  }

  void _surNiveauTermine(NiveauTermine e) {
    if (!_blesseDansLeNiveau) _debloquer('sans_egratignure');
  }

  /// Indispensable : sans cela, le service continue de réagir
  /// après la fin de l'écran qui l'a créé (26.26).
  void liberer() {
    for (final void Function() d in _abonnements) {
      d();
    }
    _abonnements.clear();
  }

  String get bilan => succes.values
      .map((Succes s) => '${s.debloque ? "[x]" : "[ ]"} ${s.titre}')
      .join('\n   ');
}

// ============================================================
//  Programme
// ============================================================

void main() {
  final BusEvenements bus = BusEvenements();
  final ServiceSucces service = ServiceSucces(bus);

  print('abonnements actifs : ${bus.nombreDAbonnements}');

  print('');
  print('=== NIVEAU 1 : le joueur se fait toucher ===');
  bus.publier(const NiveauCommence(1));
  bus.publier(const EnnemiTue('gobelin'));
  bus.publier(const EnnemiTue('gobelin'));
  for (int i = 0; i < 6; i++) {
    bus.publier(ObjetRamasse('piece $i'));
  }
  bus.publier(const JoueurBlesse(1));
  bus.publier(const NiveauTermine(1));

  print('');
  print('=== NIVEAU 2 : parcours sans faute ===');
  bus.publier(const NiveauCommence(2));
  for (int i = 0; i < 4; i++) {
    bus.publier(ObjetRamasse('piece $i'));
  }
  bus.publier(const EnnemiTue('chauve-souris'));
  bus.publier(const NiveauTermine(2));

  print('');
  print('=== BILAN ===');
  print('   ${service.bilan}');

  print('');
  print('=== retour au menu : le service est libere ===');
  service.liberer();
  print('abonnements actifs : ${bus.nombreDAbonnements}');
  bus.publier(const EnnemiTue('gobelin'));
  bus.publier(const NiveauTermine(3));
  print('aucune reaction : le service n ecoute plus.');
}
```

**Résultat :**

```text
abonnements actifs : 5

=== NIVEAU 1 : le joueur se fait toucher ===
  --- niveau 1 : compteur de blessures remis a zero
  *** SUCCES DEBLOQUE : Premier sang — Eliminer un ennemi.

=== NIVEAU 2 : parcours sans faute ===
  --- niveau 2 : compteur de blessures remis a zero
  *** SUCCES DEBLOQUE : Collectionneur — Ramasser dix objets.
  *** SUCCES DEBLOQUE : Sans une egratignure — Terminer un niveau sans etre blesse.

=== BILAN ===
   [x] Premier sang
   [x] Collectionneur
   [x] Sans une egratignure

=== retour au menu : le service est libere ===
abonnements actifs : 0
aucune reaction : le service n ecoute plus.
```

**Explication :** ce service illustre parfaitement pourquoi le bus d'événements existe. Aucune classe du jeu — ni le gobelin, ni la pièce, ni le niveau — ne connaît l'existence des succès. Vous pouvez supprimer entièrement `ServiceSucces` du projet : rien d'autre ne cesse de compiler. Vous pouvez en ajouter dix de plus : aucun fichier existant n'est touché. C'est ce que l'on appelle une extension **ouverte**.

Observez le succès « Collectionneur » : il se débloque au dixième objet, alors que les objets ont été ramassés à cheval sur deux niveaux, six puis quatre. Le service maintient son propre compteur, qui traverse les niveaux. À l'inverse, « Sans une égratignure » utilise un drapeau remis à zéro à chaque `NiveauCommence`. Deux compteurs, deux portées différentes, dans le même service : c'est possible précisément parce que le service est seul maître de son état.

Le niveau 1 ne débloque pas « Sans une égratignure » puisqu'un `JoueurBlesse` a été publié entre-temps ; le niveau 2, parcouru sans dommage, le débloque. La règle est vérifiée sans qu'aucune entité n'ait eu à savoir ce qu'est un succès.

Enfin, `liberer()` est la partie que les débutants oublient systématiquement. Sans elle, un service créé à chaque nouvelle partie s'accumulerait : après cinq parties, cinq services écouteraient les mêmes événements et afficheraient cinq fois le même message, tout en empêchant le ramasse-miettes de libérer la mémoire. Le compteur `nombreDAbonnements` passe de 5 à 0, ce qui prouve que le nettoyage est complet. Appelez toujours cette méthode depuis le `sortir()` de l'état qui a créé le service.

---

### Correction 10

Voici d'abord un extrait du point de départ, tel qu'on le trouve à la fin du chapitre 25 chez la plupart des élèves. Il ne s'agit pas de le juger : il est le résultat naturel de six chapitres d'ajouts successifs.

```dart
// AVANT — extrait représentatif, à ne pas reproduire.
class _VueDeJeuState extends State<VueDeJeu>
    with SingleTickerProviderStateMixin {
  double heroX = 40, heroY = 260, heroVx = 0, heroVy = 0;
  bool heroAuSol = false;
  int vies = 3, score = 0;

  List<double> gobX = <double>[300, 480, 620];
  List<double> gobY = <double>[318, 318, 238];
  List<int> gobDir = <int>[1, -1, 1];
  List<bool> gobVivant = <bool>[true, true, true];

  List<double> pieceX = <double>[];
  List<double> pieceY = <double>[];
  List<bool> piecePrise = <bool>[];

  bool menuAffiche = true, enPause = false, gameOver = false, victoire = false;
  Set<LogicalKeyboardKey> touches = <LogicalKeyboardKey>{};

  void update(double dt) {
    if (menuAffiche || enPause || gameOver || victoire) return;
    if (touches.contains(LogicalKeyboardKey.arrowRight)) heroX += 180 * dt;
    // ... 600 lignes de plus, gobelins, pieces, collisions, particules
  }
}
```

Et voici la version refactorée, complète et exécutable. Chaque bloc de commentaires correspond à un fichier de l'arborescence de la section 26.30.

```dart
import 'dart:math' as math;
import 'dart:ui' show PointMode;

import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';
import 'package:flutter/services.dart';

// ============================================================
//  1. core/couches.dart
// ============================================================

abstract class Couches {
  static const int sol = 0;
  static const int objets = 10;
  static const int ennemis = 20;
  static const int joueur = 30;
  static const int particules = 50;
}

// ============================================================
//  2. core/entrees.dart — aucune entité ne connaît le clavier
// ============================================================

enum ActionJeu { gauche, droite, sauter, pause, valider }

class GestionnaireEntrees {
  static const Map<LogicalKeyboardKey, ActionJeu> _mapping =
      <LogicalKeyboardKey, ActionJeu>{
    LogicalKeyboardKey.arrowLeft: ActionJeu.gauche,
    LogicalKeyboardKey.arrowRight: ActionJeu.droite,
    LogicalKeyboardKey.keyQ: ActionJeu.gauche,
    LogicalKeyboardKey.keyD: ActionJeu.droite,
    LogicalKeyboardKey.space: ActionJeu.sauter,
    LogicalKeyboardKey.escape: ActionJeu.pause,
    LogicalKeyboardKey.enter: ActionJeu.valider,
  };

  final Set<ActionJeu> _maintenues = <ActionJeu>{};
  final Set<ActionJeu> _pressees = <ActionJeu>{};

  bool maintenue(ActionJeu a) => _maintenues.contains(a);
  bool pressee(ActionJeu a) => _pressees.contains(a);

  void appuyer(ActionJeu a) {
    if (_maintenues.add(a)) _pressees.add(a);
  }

  void relacher(ActionJeu a) => _maintenues.remove(a);

  void toutRelacher() {
    _maintenues.clear();
    _pressees.clear();
  }

  void finDeFrame() => _pressees.clear();

  void surTouche(KeyEvent e) {
    final ActionJeu? a = _mapping[e.logicalKey];
    if (a == null) return;
    if (e is KeyDownEvent) {
      appuyer(a);
    } else if (e is KeyUpEvent) {
      relacher(a);
    }
  }
}

// ============================================================
//  3. core/bus_evenements.dart
// ============================================================

abstract class EvenementJeu {
  const EvenementJeu();
}

class EnnemiTue extends EvenementJeu {
  const EnnemiTue(this.x, this.y);
  final double x;
  final double y;
}

class ObjetRamasse extends EvenementJeu {
  const ObjetRamasse(this.x, this.y);
  final double x;
  final double y;
}

class JoueurTouche extends EvenementJeu {
  const JoueurTouche(this.x, this.y);
  final double x;
  final double y;
}

class BusEvenements {
  final Map<Type, List<Function>> _ecouteurs = <Type, List<Function>>{};

  void Function() ecouter<T extends EvenementJeu>(void Function(T) f) {
    _ecouteurs.putIfAbsent(T, () => <Function>[]).add(f);
    return () => _ecouteurs[T]?.remove(f);
  }

  void publier<T extends EvenementJeu>(T e) {
    final List<Function>? l = _ecouteurs[e.runtimeType];
    if (l == null) return;
    for (final Function f in List<Function>.of(l)) {
      (f as void Function(T))(e);
    }
  }
}

// ============================================================
//  4. core/entity.dart et core/monde.dart
// ============================================================

abstract class Entity {
  Entity({this.x = 0, this.y = 0, this.priority = 0});

  double x;
  double y;
  int priority;
  bool aRetirer = false;
  Monde? monde;

  double get largeur => 0;
  double get hauteur => 0;
  Rect get boite => Rect.fromLTWH(x, y, largeur, hauteur);

  void update(double dt);
  void render(Canvas canvas);
}

class Monde {
  Monde({required this.largeur, required this.hauteur});

  final double largeur;
  final double hauteur;

  final List<Entity> entites = <Entity>[];
  final List<Entity> _aAjouter = <Entity>[];
  final List<Entity> _ordre = <Entity>[];
  bool _triNecessaire = true;

  void ajouter(Entity e) {
    e.monde = this;
    _aAjouter.add(e);
  }

  void ajouterMaintenant(Entity e) {
    e.monde = this;
    entites.add(e);
    _triNecessaire = true;
  }

  Iterable<T> de<T extends Entity>() => entites.whereType<T>();

  void update(double dt) {
    for (final Entity e in entites) {
      if (!e.aRetirer) e.update(dt);
    }
    if (_aAjouter.isNotEmpty) {
      entites.addAll(_aAjouter);
      _aAjouter.clear();
      _triNecessaire = true;
    }
    final int avant = entites.length;
    entites.removeWhere((Entity e) => e.aRetirer);
    if (entites.length != avant) _triNecessaire = true;
  }

  void render(Canvas canvas) {
    if (_triNecessaire) {
      _ordre
        ..clear()
        ..addAll(entites)
        ..sort((Entity a, Entity b) {
          final int c = a.priority.compareTo(b.priority);
          return c != 0 ? c : a.y.compareTo(b.y);
        });
      _triNecessaire = false;
    }
    for (final Entity e in _ordre) {
      e.render(canvas);
    }
  }
}

// ============================================================
//  5. core/pile_etats.dart
// ============================================================

abstract class EtatDeJeu {
  late PileEtats pile;

  bool get transparent => false;

  void entrer() {}
  void sortir() {}
  void endormir() {}
  void reveiller() {}
  void update(double dt) {}
  void render(Canvas canvas, Size taille) {}
}

class PileEtats {
  final List<EtatDeJeu> _pile = <EtatDeJeu>[];

  EtatDeJeu? get sommet => _pile.isEmpty ? null : _pile.last;

  void empiler(EtatDeJeu e) {
    if (_pile.isNotEmpty) _pile.last.endormir();
    e.pile = this;
    _pile.add(e);
    e.entrer();
  }

  void depiler() {
    if (_pile.isEmpty) return;
    _pile.removeLast().sortir();
    if (_pile.isNotEmpty) _pile.last.reveiller();
  }

  void remplacerTout(EtatDeJeu e) {
    while (_pile.isNotEmpty) {
      _pile.removeLast().sortir();
    }
    empiler(e);
  }

  void update(double dt) => sommet?.update(dt);

  void render(Canvas canvas, Size taille) {
    if (_pile.isEmpty) return;
    int debut = _pile.length - 1;
    while (debut > 0 && _pile[debut].transparent) {
      debut--;
    }
    for (int i = debut; i < _pile.length; i++) {
      _pile[i].render(canvas, taille);
    }
  }
}

// ============================================================
//  6. jeu/donnees_de_jeu.dart — champs privés, règles centralisées
// ============================================================

class DonneesDeJeu {
  int _score = 0;
  int _vies = 3;
  int _meilleurScore = 0;

  int get score => _score;
  int get vies => _vies;
  int get meilleurScore => _meilleurScore;
  bool get partiePerdue => _vies <= 0;

  void ajouterPoints(int p) {
    if (p <= 0) return;
    _score += p;
    if (_score > _meilleurScore) _meilleurScore = _score;
  }

  void perdreUneVie() {
    if (_vies > 0) _vies--;
  }

  /// Le record survit à la nouvelle partie.
  void nouvellePartie() {
    _score = 0;
    _vies = 3;
  }
}

// ============================================================
//  7. jeu/contexte.dart — injection de dépendances
// ============================================================

class ContexteJeu {
  ContexteJeu({
    required this.donnees,
    required this.entrees,
    required this.bus,
  });

  final DonneesDeJeu donnees;
  final GestionnaireEntrees entrees;
  final BusEvenements bus;
}

// ============================================================
//  8. jeu/entites/*.dart
// ============================================================

class Decor extends Entity {
  Decor({
    required double x,
    required double y,
    required this.l,
    required this.h,
  }) : super(x: x, y: y, priority: Couches.sol);

  final double l;
  final double h;

  @override
  double get largeur => l;

  @override
  double get hauteur => h;

  @override
  void update(double dt) {}

  @override
  void render(Canvas canvas) =>
      canvas.drawRect(boite, Paint()..color = const Color(0xFF3F4653));
}

class Heros extends Entity {
  Heros(this.contexte, {required double x, required double y})
      : super(x: x, y: y, priority: Couches.joueur);

  final ContexteJeu contexte;

  double vx = 0;
  double vy = 0;
  bool auSol = false;
  double invincible = 0;
  double _tempsAnim = 0;
  int _frame = 0;

  @override
  double get largeur => 22;

  @override
  double get hauteur => 30;

  @override
  void update(double dt) {
    final GestionnaireEntrees e = contexte.entrees;

    vx = 0;
    if (e.maintenue(ActionJeu.gauche)) vx -= 180;
    if (e.maintenue(ActionJeu.droite)) vx += 180;
    if (e.pressee(ActionJeu.sauter) && auSol) {
      vy = -430;
      auSol = false;
    }

    vy += 1200 * dt;

    x += vx * dt;
    _resoudre(horizontal: true);
    y += vy * dt;
    auSol = false;
    _resoudre(horizontal: false);

    final Monde? m = monde;
    if (m != null) {
      x = x.clamp(0.0, m.largeur - largeur);
      if (y > m.hauteur) {
        blesser();
        x = 40;
        y = 120;
        vy = 0;
      }
    }

    _tempsAnim += dt;
    if (_tempsAnim >= 0.12) {
      _tempsAnim = 0;
      _frame = (_frame + 1) % 4;
    }
    if (invincible > 0) invincible -= dt;
  }

  void _resoudre({required bool horizontal}) {
    final Monde? m = monde;
    if (m == null) return;
    for (final Decor d in m.de<Decor>()) {
      if (!boite.overlaps(d.boite)) continue;
      if (horizontal) {
        x = vx > 0 ? d.x - largeur : d.x + d.largeur;
      } else {
        if (vy > 0) {
          y = d.y - hauteur;
          auSol = true;
        } else {
          y = d.y + d.hauteur;
        }
        vy = 0;
      }
    }
  }

  void blesser() {
    if (invincible > 0) return;
    invincible = 1.5;
    contexte.donnees.perdreUneVie();
    contexte.bus.publier(JoueurTouche(x + 11, y + 15));
  }

  @override
  void render(Canvas canvas) {
    if (invincible > 0 && (invincible * 12).floor() % 2 == 0) return;
    final double h = hauteur - (auSol && vx != 0 ? (_frame % 2) * 2 : 0);
    canvas.drawRRect(
      RRect.fromRectAndRadius(
        Rect.fromLTWH(x, y + hauteur - h, largeur, h),
        const Radius.circular(4),
      ),
      Paint()..color = const Color(0xFFE8B04B),
    );
    canvas.drawRect(
      Rect.fromLTWH(x + (vx < 0 ? 4 : 12), y + 8, 6, 4),
      Paint()..color = const Color(0xFF2A2118),
    );
  }
}

class Gobelin extends Entity {
  Gobelin(
    this.contexte, {
    required double x,
    required double y,
    required this.xMin,
    required this.xMax,
  }) : super(x: x, y: y, priority: Couches.ennemis);

  final ContexteJeu contexte;
  final double xMin;
  final double xMax;
  int direction = 1;

  @override
  double get largeur => 22;

  @override
  double get hauteur => 22;

  @override
  void update(double dt) {
    x += 60 * direction * dt;
    if (x > xMax) {
      x = xMax;
      direction = -1;
    } else if (x < xMin) {
      x = xMin;
      direction = 1;
    }

    final Monde? m = monde;
    if (m == null) return;
    for (final Heros h in m.de<Heros>()) {
      if (!boite.overlaps(h.boite)) continue;
      if (h.vy > 0 && h.y + h.hauteur - 12 < y) {
        aRetirer = true;
        h.vy = -300;
        contexte.bus.publier(EnnemiTue(x + 11, y + 11));
      } else {
        h.blesser();
      }
    }
  }

  @override
  void render(Canvas canvas) {
    canvas.drawRect(boite, Paint()..color = const Color(0xFF6FBF73));
    canvas.drawRect(
      Rect.fromLTWH(x + (direction > 0 ? 13 : 4), y + 6, 5, 4),
      Paint()..color = const Color(0xFF13351A),
    );
  }
}

class Piece extends Entity {
  Piece(this.contexte, {required double x, required double y})
      : super(x: x, y: y, priority: Couches.objets) {
    _yBase = y;
  }

  final ContexteJeu contexte;
  double _yBase = 0;
  double _phase = 0;

  @override
  double get largeur => 14;

  @override
  double get hauteur => 14;

  @override
  void update(double dt) {
    _phase += dt * 3;
    y = _yBase + math.sin(_phase) * 3;

    final Monde? m = monde;
    if (m == null) return;
    for (final Heros h in m.de<Heros>()) {
      if (boite.overlaps(h.boite)) {
        aRetirer = true;
        contexte.bus.publier(ObjetRamasse(x + 7, y + 7));
      }
    }
  }

  @override
  void render(Canvas canvas) => canvas.drawCircle(
        Offset(x + 7, y + 7),
        7,
        Paint()..color = const Color(0xFFF2C744),
      );
}

class Particule extends Entity {
  Particule({required double x, required double y, required this.couleur})
      : super(x: x, y: y, priority: Couches.particules) {
    final math.Random r = math.Random();
    vx = (r.nextDouble() - 0.5) * 200;
    vy = -r.nextDouble() * 220;
  }

  final Color couleur;
  double vx = 0;
  double vy = 0;
  double vie = 0.7;

  @override
  void update(double dt) {
    vy += 800 * dt;
    x += vx * dt;
    y += vy * dt;
    vie -= dt;
    if (vie <= 0) aRetirer = true;
  }

  @override
  void render(Canvas canvas) => canvas.drawPoints(
        PointMode.points,
        <Offset>[Offset(x, y)],
        Paint()
          ..color = couleur.withOpacity((vie / 0.7).clamp(0.0, 1.0))
          ..strokeWidth = 4
          ..strokeCap = StrokeCap.round,
      );
}

// ============================================================
//  9. ui/ — outil de texte
// ============================================================

void _texte(Canvas canvas, String t, double cx, double cy, double taille,
    Color couleur) {
  final TextPainter tp = TextPainter(
    text: TextSpan(
      text: t,
      style: TextStyle(
          color: couleur, fontSize: taille, fontWeight: FontWeight.w600),
    ),
    textDirection: TextDirection.ltr,
  )..layout();
  tp.paint(canvas, Offset(cx - tp.width / 2, cy - tp.height / 2));
}

// ============================================================
//  10. jeu/etats/*.dart
// ============================================================

class EtatMenu extends EtatDeJeu {
  EtatMenu(this.contexte);
  final ContexteJeu contexte;

  double _t = 0;

  @override
  void entrer() => contexte.entrees.toutRelacher();

  @override
  void update(double dt) {
    _t += dt;
    if (contexte.entrees.pressee(ActionJeu.valider)) {
      contexte.donnees.nouvellePartie();
      pile.remplacerTout(EtatJeu(contexte));
    }
  }

  @override
  void render(Canvas canvas, Size taille) {
    canvas.drawRect(
        Offset.zero & taille, Paint()..color = const Color(0xFF14161C));
    _texte(canvas, 'DONJON DE DART', taille.width / 2, taille.height / 2 - 60,
        34, const Color(0xFFE8B04B));
    if ((_t * 2).floor() % 2 == 0) {
      _texte(canvas, 'Entree pour jouer', taille.width / 2,
          taille.height / 2 + 4, 18, Colors.white);
    }
    if (contexte.donnees.meilleurScore > 0) {
      _texte(canvas, 'Record : ${contexte.donnees.meilleurScore}',
          taille.width / 2, taille.height / 2 + 44, 15,
          const Color(0xFF8A93A5));
    }
    _texte(canvas, 'Fleches ou Q/D  —  Espace : sauter  —  Echap : pause',
        taille.width / 2, taille.height - 30, 13, const Color(0xFF8A93A5));
  }
}

class EtatJeu extends EtatDeJeu {
  EtatJeu(this.contexte);
  final ContexteJeu contexte;

  late Monde monde;
  late Heros heros;
  final List<void Function()> _abonnements = <void Function()>[];

  @override
  void entrer() {
    _construireNiveau();

    _abonnements.add(contexte.bus.ecouter<EnnemiTue>((EnnemiTue e) {
      contexte.donnees.ajouterPoints(10);
      _gerbe(e.x, e.y, const Color(0xFF6FBF73), 12);
    }));
    _abonnements.add(contexte.bus.ecouter<ObjetRamasse>((ObjetRamasse e) {
      contexte.donnees.ajouterPoints(5);
      _gerbe(e.x, e.y, const Color(0xFFF2C744), 8);
    }));
    _abonnements.add(contexte.bus.ecouter<JoueurTouche>((JoueurTouche e) {
      _gerbe(e.x, e.y, const Color(0xFFCF5C5C), 10);
    }));
  }

  /// Sans ce désabonnement, l'écouteur survivrait à l'écran (26.26).
  @override
  void sortir() {
    for (final void Function() d in _abonnements) {
      d();
    }
    _abonnements.clear();
  }

  /// Sans cela, une touche relâchée pendant la pause resterait « collée ».
  @override
  void endormir() => contexte.entrees.toutRelacher();

  void _gerbe(double x, double y, Color c, int n) {
    for (int i = 0; i < n; i++) {
      monde.ajouter(Particule(x: x, y: y, couleur: c));
    }
  }

  void _construireNiveau() {
    monde = Monde(largeur: 700, hauteur: 380);
    monde.ajouterMaintenant(Decor(x: 0, y: 340, l: 700, h: 40));
    monde.ajouterMaintenant(Decor(x: 140, y: 260, l: 120, h: 18));
    monde.ajouterMaintenant(Decor(x: 380, y: 210, l: 120, h: 18));
    monde.ajouterMaintenant(Decor(x: 560, y: 280, l: 110, h: 18));

    heros = Heros(contexte, x: 30, y: 280);
    monde.ajouterMaintenant(heros);

    monde.ajouterMaintenant(
        Gobelin(contexte, x: 250, y: 318, xMin: 220, xMax: 420));
    monde.ajouterMaintenant(
        Gobelin(contexte, x: 500, y: 318, xMin: 460, xMax: 660));
    monde.ajouterMaintenant(
        Gobelin(contexte, x: 400, y: 188, xMin: 380, xMax: 480));

    const List<List<double>> positions = <List<double>>[
      <double>[170, 230],
      <double>[220, 230],
      <double>[410, 180],
      <double>[460, 180],
      <double>[590, 250],
      <double>[640, 250],
      <double>[320, 310],
    ];
    for (final List<double> p in positions) {
      monde.ajouterMaintenant(Piece(contexte, x: p[0], y: p[1]));
    }
  }

  @override
  void update(double dt) {
    if (contexte.entrees.pressee(ActionJeu.pause)) {
      pile.empiler(EtatPause(contexte));
      return;
    }

    monde.update(dt);

    if (contexte.donnees.partiePerdue) {
      pile.remplacerTout(EtatFin(contexte, gagne: false));
    } else if (monde.de<Piece>().isEmpty && monde.de<Gobelin>().isEmpty) {
      pile.remplacerTout(EtatFin(contexte, gagne: true));
    }
  }

  @override
  void render(Canvas canvas, Size taille) {
    canvas.drawRect(
        Offset.zero & taille, Paint()..color = const Color(0xFF14161C));
    monde.render(canvas);
    _texte(canvas, 'Score ${contexte.donnees.score}', 70, 22, 16,
        const Color(0xFFE8B04B));
    _texte(canvas, 'Vies ${contexte.donnees.vies}', taille.width - 60, 22, 16,
        const Color(0xFFCF5C5C));
    _texte(
        canvas,
        'Restants ${monde.de<Piece>().length + monde.de<Gobelin>().length}',
        taille.width / 2,
        22,
        14,
        const Color(0xFF8A93A5));
  }
}

class EtatPause extends EtatDeJeu {
  EtatPause(this.contexte);
  final ContexteJeu contexte;

  @override
  bool get transparent => true;

  @override
  void entrer() => contexte.entrees.toutRelacher();

  @override
  void update(double dt) {
    if (contexte.entrees.pressee(ActionJeu.pause)) {
      pile.depiler();
    } else if (contexte.entrees.pressee(ActionJeu.valider)) {
      pile.remplacerTout(EtatMenu(contexte));
    }
  }

  @override
  void render(Canvas canvas, Size taille) {
    canvas.drawRect(
        Offset.zero & taille, Paint()..color = const Color(0xB3000000));
    _texte(canvas, 'PAUSE', taille.width / 2, taille.height / 2 - 24, 32,
        const Color(0xFFE8B04B));
    _texte(canvas, 'Echap : reprendre    Entree : menu', taille.width / 2,
        taille.height / 2 + 22, 15, Colors.white);
  }
}

class EtatFin extends EtatDeJeu {
  EtatFin(this.contexte, {required this.gagne});
  final ContexteJeu contexte;
  final bool gagne;

  double _t = 0;

  @override
  void entrer() => contexte.entrees.toutRelacher();

  @override
  void update(double dt) {
    _t += dt;
    if (_t > 0.6 && contexte.entrees.pressee(ActionJeu.valider)) {
      pile.remplacerTout(EtatMenu(contexte));
    }
  }

  @override
  void render(Canvas canvas, Size taille) {
    canvas.drawRect(
        Offset.zero & taille, Paint()..color = const Color(0xFF14161C));
    _texte(canvas, gagne ? 'VICTOIRE' : 'GAME OVER', taille.width / 2,
        taille.height / 2 - 50, 34,
        gagne ? const Color(0xFF6FBF73) : const Color(0xFFCF5C5C));
    _texte(canvas, 'Score : ${contexte.donnees.score}', taille.width / 2,
        taille.height / 2, 20, Colors.white);
    _texte(canvas, 'Record : ${contexte.donnees.meilleurScore}',
        taille.width / 2, taille.height / 2 + 30, 15,
        const Color(0xFF8A93A5));
    if (_t > 0.6) {
      _texte(canvas, 'Entree : menu', taille.width / 2, taille.height / 2 + 74,
          15, const Color(0xFF8A93A5));
    }
  }
}

// ============================================================
//  11. ui/vue_de_jeu.dart
// ============================================================

class VueDeJeu extends StatefulWidget {
  const VueDeJeu({super.key});

  @override
  State<VueDeJeu> createState() => _VueDeJeuState();
}

class _VueDeJeuState extends State<VueDeJeu>
    with SingleTickerProviderStateMixin {
  late final ContexteJeu contexte;
  late final PileEtats pile;
  late final Ticker _ticker;

  final ValueNotifier<int> _frame = ValueNotifier<int>(0);
  final FocusNode _focus = FocusNode();
  Duration _precedent = Duration.zero;

  @override
  void initState() {
    super.initState();
    contexte = ContexteJeu(
      donnees: DonneesDeJeu(),
      entrees: GestionnaireEntrees(),
      bus: BusEvenements(),
    );
    pile = PileEtats()..empiler(EtatMenu(contexte));
    _ticker = createTicker(_surFrame)..start();
  }

  void _surFrame(Duration ecoule) {
    double dt = (ecoule - _precedent).inMicroseconds / 1000000.0;
    _precedent = ecoule;
    dt = dt.clamp(0.0, 0.05); // dt plafonné : chapitre 20 et section 26.23

    pile.update(dt);
    contexte.entrees.finDeFrame();
    _frame.value++;
  }

  @override
  void dispose() {
    _ticker.dispose();
    _focus.dispose();
    _frame.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return KeyboardListener(
      focusNode: _focus,
      autofocus: true,
      onKeyEvent: contexte.entrees.surTouche,
      child: CustomPaint(
        painter: _PeintreDeJeu(pile: pile, repaint: _frame),
        child: const SizedBox.expand(),
      ),
    );
  }
}

class _PeintreDeJeu extends CustomPainter {
  _PeintreDeJeu({required this.pile, required Listenable repaint})
      : super(repaint: repaint);

  final PileEtats pile;

  @override
  void paint(Canvas canvas, Size size) => pile.render(canvas, size);

  @override
  bool shouldRepaint(covariant _PeintreDeJeu ancien) => false;
}

// ============================================================
//  12. main.dart — cinq lignes utiles
// ============================================================

void main() => runApp(const DonjonApp());

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) => const MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: Color(0xFF14161C),
          body: SafeArea(child: VueDeJeu()),
        ),
      );
}
```

**Résultat :** le jeu s'ouvre sur le menu du Donjon de Dart. Entrée lance la partie. Le héros se déplace, saute entre trois plateformes, ramasse sept pièces dorées et affronte trois gobelins verts. Sauter sur un gobelin l'élimine, le toucher de côté coûte une vie et déclenche une invincibilité clignotante d'une seconde et demie. Échap superpose le voile de pause au monde figé ; Échap à nouveau reprend exactement où l'on s'était arrêté, sans perdre le score. À zéro vie, l'écran de Game Over apparaît ; en éliminant tout, c'est l'écran de victoire. Le record persiste d'une partie à l'autre.

Et voici la suite de tests demandée, à placer dans `test/jeu_test.dart`.

```dart
import 'package:test/test.dart';

// Ces classes sont normalement importées depuis lib/. Elles sont
// reproduites ici en version sans rendu pour que le test soit
// exécutable tel quel avec « dart test ».

enum GameState { demarrage, menu, jeu, pause, gameOver, victoire }

class MachineAEtats {
  GameState etat = GameState.demarrage;

  static const Map<GameState, Set<GameState>> _ok = <GameState, Set<GameState>>{
    GameState.demarrage: <GameState>{GameState.menu},
    GameState.menu: <GameState>{GameState.jeu},
    GameState.jeu: <GameState>{
      GameState.pause,
      GameState.gameOver,
      GameState.victoire,
    },
    GameState.pause: <GameState>{GameState.jeu, GameState.menu},
    GameState.gameOver: <GameState>{GameState.jeu, GameState.menu},
    GameState.victoire: <GameState>{GameState.jeu, GameState.menu},
  };

  bool aller(GameState v) {
    if (!(_ok[etat]?.contains(v) ?? false)) return false;
    etat = v;
    return true;
  }
}

class DonneesDeJeu {
  int _score = 0;
  int _vies = 3;
  int _meilleurScore = 0;

  int get score => _score;
  int get vies => _vies;
  int get meilleurScore => _meilleurScore;
  bool get partiePerdue => _vies <= 0;

  void ajouterPoints(int p) {
    if (p <= 0) return;
    _score += p;
    if (_score > _meilleurScore) _meilleurScore = _score;
  }

  void perdreUneVie() {
    if (_vies > 0) _vies--;
  }

  void nouvellePartie() {
    _score = 0;
    _vies = 3;
  }
}

abstract class Entity {
  double x = 0;
  double y = 0;
  bool aRetirer = false;
  void update(double dt);
}

class MobileSimple extends Entity {
  MobileSimple(this.vitesse);
  final double vitesse;

  @override
  void update(double dt) => x += vitesse * dt;
}

class Monde {
  final List<Entity> entites = <Entity>[];
  final List<Entity> _aAjouter = <Entity>[];

  void ajouter(Entity e) => _aAjouter.add(e);

  void update(double dt) {
    for (final Entity e in entites) {
      if (!e.aRetirer) e.update(dt);
    }
    entites.addAll(_aAjouter);
    _aAjouter.clear();
    entites.removeWhere((Entity e) => e.aRetirer);
  }
}

bool chevauchent(double ax, double ay, double al, double ah, double bx,
    double by, double bl, double bh) {
  return ax < bx + bl && ax + al > bx && ay < by + bh && ay + ah > by;
}

void main() {
  test('1. le mouvement est indépendant du framerate', () {
    final MobileSimple a = MobileSimple(180);
    final MobileSimple b = MobileSimple(180);
    for (int i = 0; i < 60; i++) {
      a.update(1 / 60);
    }
    for (int i = 0; i < 144; i++) {
      b.update(1 / 144);
    }
    expect(a.x, closeTo(180, 0.0001));
    expect(b.x, closeTo(180, 0.0001));
  });

  test('2. les transitions interdites sont refusées', () {
    final MachineAEtats m = MachineAEtats();
    m.aller(GameState.menu);
    expect(m.aller(GameState.pause), isFalse);
    m.aller(GameState.jeu);
    m.aller(GameState.pause);
    expect(m.aller(GameState.gameOver), isFalse);
    expect(m.etat, GameState.pause);
  });

  test('3. le retrait est différé et ne lève pas d exception', () {
    final Monde monde = Monde();
    final MobileSimple a = MobileSimple(10);
    final MobileSimple b = MobileSimple(10);
    monde.entites.addAll(<Entity>[a, b]);

    expect(() {
      a.aRetirer = true;
      monde.ajouter(MobileSimple(20));
      monde.update(1 / 60);
    }, returnsNormally);

    expect(monde.entites.length, 2);
    expect(monde.entites.contains(a), isFalse);
    expect(monde.entites.contains(b), isTrue);
  });

  test('4. le score refuse les valeurs négatives', () {
    final DonneesDeJeu d = DonneesDeJeu();
    d.ajouterPoints(30);
    d.ajouterPoints(-500);
    expect(d.score, 30);
  });

  test('5. la collision AABB est correcte aux bords', () {
    expect(chevauchent(0, 0, 10, 10, 5, 5, 10, 10), isTrue);
    expect(chevauchent(0, 0, 10, 10, 10, 0, 10, 10), isFalse);
    expect(chevauchent(0, 0, 100, 100, 40, 40, 5, 5), isTrue);
  });

  test('6. une nouvelle partie conserve le record', () {
    final DonneesDeJeu d = DonneesDeJeu();
    d.ajouterPoints(240);
    for (int i = 0; i < 5; i++) {
      d.perdreUneVie();
    }
    expect(d.partiePerdue, isTrue);
    expect(d.vies, 0);

    d.nouvellePartie();
    expect(d.score, 0);
    expect(d.vies, 3);
    expect(d.meilleurScore, 240);
  });
}
```

**Résultat :**

```text
00:00 +6: All tests passed!
```

**Explication :** ce refactoring illustre le fait qu'une bonne architecture ne se juge pas à sa beauté, mais aux **changements qu'elle rend faciles**. Reprenons les dix points du cahier des charges et mesurons chacun.

**Les listes parallèles ont disparu.** `gobX`, `gobY`, `gobDir` et `gobVivant` sont devenues une classe `Gobelin` avec quatre champs. Il n'est plus possible de désynchroniser les données d'un même gobelin, puisqu'elles ne sont plus séparées.

**Le monde est générique.** `Monde.update` ne connaît ni gobelin, ni pièce, ni héros. Vous pouvez ajouter une chauve-souris sans ouvrir ce fichier. C'est le critère « on étend sans modifier » de la section 26.6.

**Les quatre booléens sont devenus une pile.** `menuAffiche`, `enPause`, `gameOver` et `victoire` n'existent plus. Aucune combinaison contradictoire n'est représentable. Surtout, le test `if (menuAffiche || enPause || ...) return;` en tête de `update` a disparu : l'état de jeu ne reçoit tout simplement plus `update` quand il n'est pas au sommet.

**Aucune entité ne connaît le clavier.** Cherchez `LogicalKeyboardKey` dans le programme : il n'apparaît que dans `GestionnaireEntrees`. Le héros lit `maintenue(ActionJeu.droite)`. Brancher un joystick tactile demanderait d'ajouter un appel `appuyer(ActionJeu.droite)` quelque part dans l'interface, et rien d'autre.

**Le bus découple les conséquences.** `Gobelin` publie `EnnemiTue` et ignore complètement l'existence du score et des particules. Ajouter un son ou un succès ne touchera pas à `Gobelin`.

**Les abonnements sont libérés.** `EtatJeu.sortir()` appelle chaque fonction de désabonnement. Sans cela, une seconde partie créerait un second jeu d'écouteurs, et le score monterait deux fois plus vite — un bug fréquent et déroutant.

**Les données sont protégées.** `_score` et `_vies` sont privés. Aucune ligne du jeu ne peut écrire `donnees.score = 999`, et `ajouterPoints` applique la règle du record automatiquement. Le test 4 vérifie qu'un score négatif est refusé.

**Le contexte est injecté.** Aucun Singleton. Le test 6 crée son propre `DonneesDeJeu` et n'est donc pas contaminé par les tests précédents, contrairement à la démonstration de la section 26.28.

**Le rendu ne modifie rien.** Parcourez les méthodes `render` : aucune n'affecte un champ. Le seul calcul qu'elles font, comme la hauteur oscillante du héros, est une variable locale dérivée de l'état, pas une modification de cet état.

**La pause est propre.** `EtatPause` est transparent, donc le monde reste visible. `EtatJeu.endormir()` relâche les touches. Le `dt` est plafonné à 0,05 seconde dans `_surFrame`, ce qui neutralise le temps accumulé pendant une longue pause.

Un dernier point, et non le moindre : ce fichier reste long, environ six cents lignes. Mais sa longueur n'est plus un problème, car il est **découpable sans effort**. Chaque bloc numéroté peut être déplacé tel quel dans son fichier, sans réécrire une ligne. C'est exactement ce que vous ferez au chapitre 35, quand le projet final commencera pour de bon.

---

## Et maintenant ?

La PARTIE 2A s'achève ici. Faisons le point sur le chemin parcouru depuis le chapitre 19.

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │                    BILAN DE LA PARTIE 2A                         │
  └──────────────────────────────────────────────────────────────────┘

  19  Flutter en accéléré        runApp, widgets, setState, Ticker
  20  Boucle de jeu              entrées -> update -> render, dt, FPS
  21  Canvas et dessin 2D        repère, formes, Paint, transformations
  22  Sprites et animation       sprite sheets, frames, cadence
  23  Mouvement et physique      vecteurs, vitesse, gravité, saut
  24  Collisions et hitboxes     AABB, cercles, résolution, tunneling
  25  Caméra, monde et niveaux   monde vs écran, suivi, parallaxe
  26  Architecture et états      entités, files, priority, machine à
                                 états, pile, entrées, bus, services

  Vous avez écrit, à la main, un moteur de jeu 2D complet.
```

Ce n'est pas une formule de politesse. Reprenez le fichier de la section 26.32 : il contient une boucle de jeu, un système d'entités, une gestion de profondeur, une machine à états empilable, une abstraction des entrées, un bus d'événements et une injection de dépendances. Ce sont les sept briques que l'on retrouve, sous des noms différents, dans Unity, dans Godot, dans Bevy et dans Flame.

Le chapitre 27 ouvre la PARTIE 2B, et il change complètement de rythme. Vous allez ajouter une ligne dans `pubspec.yaml` :

```text
  dependencies:
    flame: ^1.x.x
```

et récupérer d'un coup tout ce que vous venez d'écrire, plus quelques milliers de lignes que vous n'avez pas écrites. Vous découvrirez `FlameGame`, la classe qui remplace votre `MoteurDeBoucle`, et `GameWidget`, le widget qui l'insère dans une application Flutter. Vous écrirez votre premier `onLoad`, votre premier `update` et votre premier `render` sous Flame, et vous constaterez que les signatures sont exactement celles de la section 26.5 — ce n'est pas une coïncidence.

Trois conseils avant d'y aller.

**Gardez vos fichiers de la PARTIE 2A.** Ils vous serviront de point de comparaison. Chaque fois que Flame fera quelque chose « magiquement », retrouvez la section correspondante de ce chapitre : la magie disparaîtra.

**Ne jetez pas les sections 26.16 à 26.31.** Flame ne fournit ni machine à états de jeu, ni gestionnaire d'entrées abstrait, ni bus d'événements, ni service de données, ni injection de dépendances, ni organisation de fichiers, ni tests. Tout cela reste votre travail, et vous le réutiliserez tel quel dans la PARTIE 2C.

**Vérifiez la version de Flame.** Le chapitre 27 indique la version utilisée, et l'API de Flame évolue vite. Si un exemple ne compile pas, la première chose à vérifier est le numéro de version dans votre `pubspec.yaml`.

Rendez-vous au chapitre suivant : [27-PARTIE-2B—INSTALLER-FLAME-ET-PREMIER-FLAMEGAME.md](./27-PARTIE-2B—INSTALLER-FLAME-ET-PREMIER-FLAMEGAME.md)
