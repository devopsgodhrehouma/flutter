# PARTIE 2A — JEU 2D EN FLUTTER PUR
# CHAPITRE 21 — COORDONNÉES, CANVAS ET DESSIN 2D

> **Niveau :** intermédiaire
> **Durée estimée :** 7 h
> **Pré-requis :** chapitre 19 (Flutter en accéléré, `CustomPainter`) et chapitre 20 (boucle de jeu, `Ticker`, delta time)
> **Ce que vous saurez faire à la fin :** dessiner n'importe quelle scène 2D à la main avec `Canvas` et `Paint`, la positionner au pixel près dans le repère de l'écran, la transformer (translation, rotation, échelle) et l'animer avec le `dt` de la boucle de jeu.

---

## 21.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer pourquoi l'axe `Y` de l'écran est orienté vers le bas, contrairement au repère mathématique ;
- situer l'origine `(0, 0)` d'une zone de dessin et raisonner en coordonnées locales ;
- distinguer un pixel logique d'un pixel physique et utiliser `devicePixelRatio` ;
- lire la taille de la zone de dessin transmise à `paint()` et vous y adapter ;
- manipuler `Offset`, `Rect` et `Size`, les trois types de base de la géométrie Flutter ;
- anticiper la différence entre `Vector2` (Flame) et `Offset` (Flutter) ;
- écrire un `CustomPainter` propre, avec un `shouldRepaint()` correct ;
- configurer un objet `Paint` : couleur, style, épaisseur de trait ;
- dessiner des rectangles, cercles, lignes, ellipses, rectangles arrondis, chemins et points ;
- dessiner du texte dans un `Canvas` avec `TextPainter` ;
- construire des couleurs en ARGB et appliquer une transparence ;
- appliquer un dégradé linéaire ou radial à une forme ;
- expliquer et exploiter l'ordre de dessin (« algorithme du peintre ») ;
- utiliser `save()` / `restore()` sans jamais déséquilibrer la pile ;
- appliquer `translate()`, `rotate()` et `scale()`, et comprendre pourquoi leur ordre compte ;
- dessiner une forme autour de son centre plutôt que de son coin (notion d'ancre) ;
- limiter le dessin à une zone avec `clipRect()` ;
- expliquer le double buffering et pourquoi l'écran ne clignote pas ;
- identifier ce qui coûte cher dans `paint()` et l'éviter ;
- dessiner une grille de donjon et un personnage entièrement en formes géométriques ;
- animer un dessin avec le `dt` du chapitre 20.

---

## 21.1 — Le repère de l'écran : Y vers le bas

Vous avez appris au collège un repère où `Y` monte. En informatique graphique, `Y` **descend**. Ce n'est pas un caprice : c'est un héritage direct du matériel.

Les premiers écrans étaient des tubes cathodiques. Un faisceau d'électrons balayait la surface **ligne par ligne, de haut en bas**, et sur chaque ligne **de gauche à droite**. La mémoire vidéo était donc lue dans cet ordre : le premier octet correspondait au coin en haut à gauche. Tout le reste en découle. Aujourd'hui encore, une image en mémoire est stockée ligne du haut d'abord.

Voici les deux repères côte à côte.

```text
   REPÈRE MATHÉMATIQUE                 REPÈRE DE L'ÉCRAN
   (celui du collège)                  (celui de Flutter)

           Y                        (0,0)
           ^                          +─────────────────────> X
           │                          │
       +2  │   . (3, 2)               │
       +1  │                          │      . (3, 2)
        0  +───────────> X            │
       -1  │                          │
           │                          v
                                      Y

   Y augmente vers le HAUT            Y augmente vers le BAS
   l'origine est au CENTRE            l'origine est en HAUT À GAUCHE
```

Conséquence pratique, à graver dès maintenant :

> Pour faire **monter** un objet à l'écran, on **diminue** son `y`.
> Pour le faire **descendre**, on **augmente** son `y`.

C'est contre-intuitif pendant environ une semaine, puis cela devient un réflexe. Ce détail explique aussi la formule de la gravité que vous écrirez au chapitre 23 : la gravité est un nombre **positif** ajouté à la vitesse verticale, parce qu'elle pousse vers le bas, donc vers les `y` croissants.

Prenons notre fil rouge, le **Donjon de Dart**. Un gobelin se trouve en `(120, 80)` et une potion en `(120, 200)`. Laquelle est la plus haute à l'écran ?

```text
  (0,0)
    +───────────────────────────────────> X
    │
    │            G  gobelin  (120, 80)     <- plus HAUT
    │
    │
    │            P  potion   (120, 200)    <- plus BAS
    │
    v
    Y
```

Le gobelin, avec son `y` **plus petit**, est plus haut. Retenez : petit `y` = haut de l'écran.

Vérifions cela avec un vrai programme Flutter. Le code ci-dessous est un `main.dart` complet et copiable.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const RepereApp());
}

class RepereApp extends StatelessWidget {
  const RepereApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: const Color(0xFF14161C),
        body: Center(
          child: CustomPaint(
            size: const Size(320, 280),
            painter: RepereePainter(),
          ),
        ),
      ),
    );
  }
}

class RepereePainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    final fond = Paint()..color = const Color(0xFF20242E);
    canvas.drawRect(Offset.zero & size, fond);

    // Le gobelin : y petit, donc en haut.
    final gobelin = Paint()..color = const Color(0xFF6FCF6F);
    canvas.drawCircle(const Offset(120, 80), 18, gobelin);

    // La potion : y grand, donc en bas.
    final potion = Paint()..color = const Color(0xFFE05A78);
    canvas.drawCircle(const Offset(120, 200), 18, potion);
  }

  @override
  bool shouldRepaint(covariant RepereePainter oldDelegate) => false;
}
```

**Résultat :**

```text
Une zone gris foncé de 320 x 280.
Un disque vert (le gobelin) dans la moitié haute.
Un disque rouge (la potion) plus bas, exactement à la verticale du vert.
```

Le disque dont le `y` vaut 80 est bien au-dessus de celui dont le `y` vaut 200.

> **Remarque.** Ce repère est celui de presque tous les systèmes graphiques 2D : Canvas HTML, SDL, Cairo, Skia (le moteur de rendu de Flutter), Android, iOS. Les moteurs 3D, eux, utilisent souvent un `Y` vers le haut. Ne mélangez pas les deux mondes.

---

## 21.2 — L'origine (0, 0)

L'origine, c'est le point `(0, 0)`. En 2D écran, il est en **haut à gauche**.

Mais la question importante est : **haut à gauche de quoi ?**

La réponse est : de la zone de dessin qu'on vous confie. Et c'est une excellente nouvelle.

```text
  ÉCRAN DU TÉLÉPHONE
  ┌──────────────────────────────────────┐
  │ (0,0) de l'écran                     │
  │                                      │
  │      ┌────────────────────────┐      │
  │      │ (0,0) de VOTRE Canvas  │      │
  │      │                        │      │
  │      │   zone confiée au      │      │
  │      │   CustomPainter        │      │
  │      │                        │      │
  │      └────────────────────────┘      │
  │                                      │
  └──────────────────────────────────────┘
```

Quand Flutter appelle votre méthode `paint(Canvas canvas, Size size)`, il a **déjà** déplacé l'origine du canvas au coin haut-gauche de votre widget. Vous travaillez donc en **coordonnées locales** : si votre `CustomPaint` est centré dans l'écran, vous n'avez pas à en tenir compte. `Offset(0, 0)` désigne toujours votre propre coin.

C'est exactement le même principe que les coordonnées relatives d'une pièce dans le donjon : la case `(0, 0)` d'une salle est le coin de la salle, pas le coin du donjon entier.

Voici un programme qui matérialise l'origine et les deux axes.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const OrigineApp());

class OrigineApp extends StatelessWidget {
  const OrigineApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: const Color(0xFF14161C),
        body: Center(
          child: Container(
            color: const Color(0xFF20242E),
            child: CustomPaint(
              size: const Size(300, 220),
              painter: OriginePainter(),
            ),
          ),
        ),
      ),
    );
  }
}

class OriginePainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    final axe = Paint()
      ..color = const Color(0xFF8FA1C7)
      ..strokeWidth = 2;

    // Axe X : de l'origine vers la droite.
    canvas.drawLine(Offset.zero, Offset(size.width, 0), axe);
    // Axe Y : de l'origine vers le bas.
    canvas.drawLine(Offset.zero, Offset(0, size.height), axe);

    // Un repère visible sur l'origine elle-même.
    final origine = Paint()..color = const Color(0xFFFFC857);
    canvas.drawCircle(Offset.zero, 6, origine);

    // Quelques points de contrôle.
    final point = Paint()..color = const Color(0xFF6FCF6F);
    canvas.drawCircle(const Offset(50, 0), 4, point);
    canvas.drawCircle(const Offset(0, 50), 4, point);
    canvas.drawCircle(const Offset(50, 50), 4, point);
  }

  @override
  bool shouldRepaint(covariant OriginePainter oldDelegate) => false;
}
```

**Résultat :**

```text
Une zone gris foncé de 300 x 220.
Un quart de disque jaune au coin haut-gauche : c'est l'origine, coupée en deux
par les bords du canvas.
Une ligne claire le long du bord haut (axe X) et une le long du bord gauche (axe Y).
Trois petits points verts : à 50 px à droite, 50 px en bas, et en diagonale.
```

Remarquez que le disque jaune est **coupé**. C'est normal : son centre est exactement sur le coin, donc trois quarts du disque tombent hors de la zone. Flutter ne dessine pas au-delà des limites du `CustomPaint` par défaut lorsqu'un parent le contraint. Nous reviendrons sur ce découpage en section 21.30.

> **Point d'attention.** Rien ne vous interdit de dessiner à des coordonnées négatives, comme `Offset(-40, -40)`. Le dessin sera simplement invisible, ou partiellement visible. C'est utile pour un ennemi qui entre par la gauche de l'écran.

---

## 21.3 — Pixels logiques et `devicePixelRatio`

Voici une question piège. Vous dessinez un carré de 100 de côté. Combien de pixels physiques occupe-t-il sur l'écran ?

Réponse : **on ne sait pas**. Cela dépend du téléphone.

Flutter ne travaille pas en pixels physiques mais en **pixels logiques** (aussi appelés « logical pixels » ou « density-independent pixels »). Un pixel logique correspond à peu près à la même **taille réelle** sur tous les appareils : environ 1/160 de pouce. Un bouton de 48 pixels logiques a donc la même taille sous le doigt sur un vieil écran et sur un écran très fin.

La conversion se fait avec le `devicePixelRatio` :

```text
  pixels physiques  =  pixels logiques  x  devicePixelRatio
```

```text
  APPAREIL A : devicePixelRatio = 1.0
  ┌───────────────────────────────┐
  │  carré de 100 px logiques     │
  │  = 100 px physiques           │
  └───────────────────────────────┘

  APPAREIL B : devicePixelRatio = 3.0
  ┌───────────────────────────────┐
  │  carré de 100 px logiques     │
  │  = 300 px physiques           │
  │  (même taille pour l'oeil,    │
  │   mais 9 fois plus de pixels) │
  └───────────────────────────────┘
```

Bonne nouvelle : **pour dessiner, vous ne vous en occupez jamais**. Vous raisonnez toujours en pixels logiques, et Skia s'occupe de la mise à l'échelle. Votre gobelin de 32 de côté fera 32 pixels logiques partout.

Vous n'avez besoin du `devicePixelRatio` que dans deux cas :

1. pour afficher une information de diagnostic (utile pour comprendre les performances : un écran à ratio 3 doit remplir 9 fois plus de pixels) ;
2. pour choisir la résolution d'une image bitmap générée à la volée (chapitre 22).

On le lit dans le `MediaQuery` :

```dart
import 'package:flutter/material.dart';

void main() => runApp(const RatioApp());

class RatioApp extends StatelessWidget {
  const RatioApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: const Color(0xFF14161C),
        body: Center(child: const InfoEcran()),
      ),
    );
  }
}

class InfoEcran extends StatelessWidget {
  const InfoEcran({super.key});

  @override
  Widget build(BuildContext context) {
    final media = MediaQuery.of(context);
    final logiques = media.size;
    final ratio = media.devicePixelRatio;
    final physiquesL = logiques.width * ratio;
    final physiquesH = logiques.height * ratio;

    const style = TextStyle(color: Colors.white, fontSize: 16);

    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        Text('Taille logique  : '
            '${logiques.width.toStringAsFixed(1)} x '
            '${logiques.height.toStringAsFixed(1)}', style: style),
        Text('devicePixelRatio: ${ratio.toStringAsFixed(2)}', style: style),
        Text('Taille physique : '
            '${physiquesL.toStringAsFixed(0)} x '
            '${physiquesH.toStringAsFixed(0)}', style: style),
      ],
    );
  }
}
```

**Résultat (exemple sur un téléphone à ratio 3) :**

```text
Taille logique  : 411.4 x 866.3
devicePixelRatio: 3.00
Taille physique : 1234 x 2599
```

Sur un navigateur de bureau standard, vous liriez plutôt :

```text
Taille logique  : 1280.0 x 720.0
devicePixelRatio: 1.00
Taille physique : 1280 x 720
```

> **À retenir.** Vous codez en pixels logiques. Le `devicePixelRatio` est une information de contexte, pas un facteur à appliquer vous-même dans `paint()`.

---

## 21.4 — La taille de la zone de dessin (`Size`)

La méthode `paint()` reçoit deux paramètres :

```dart
void paint(Canvas canvas, Size size)
```

Le second, `size`, est la **taille réelle** de la zone qu'on vous confie, en pixels logiques. Elle est déterminée par le système de layout de Flutter, pas par vous.

C'est le paramètre le plus souvent ignoré par les débutants, et c'est une erreur. Écrire des coordonnées en dur comme `Offset(200, 150)` produit un dessin qui sera mal placé dès que la fenêtre change de taille, ou dès qu'on passe du téléphone à la tablette.

La bonne pratique est de **tout exprimer en fonction de `size`** :

```text
  size = Size(largeur, hauteur)

  ┌───────────────────────────────────────┐  <- y = 0
  │                                       │
  │                 x                     │  <- y = size.height / 2
  │           (centre exact)              │
  │                                       │
  └───────────────────────────────────────┘  <- y = size.height
  ^                                       ^
  x = 0                          x = size.width

  centre = Offset(size.width / 2, size.height / 2)
```

Flutter fournit d'ailleurs des raccourcis pratiques sur `Size` :

| Expression | Résultat |
| --- | --- |
| `size.width` | largeur en pixels logiques |
| `size.height` | hauteur en pixels logiques |
| `size.shortestSide` | le plus petit des deux |
| `size.longestSide` | le plus grand des deux |
| `size.aspectRatio` | `width / height` |
| `size.center(Offset.zero)` | le centre, sous forme d'`Offset` |
| `Offset.zero & size` | un `Rect` couvrant toute la zone |

Voici un programme qui s'adapte automatiquement à la taille reçue.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const TailleApp());

class TailleApp extends StatelessWidget {
  const TailleApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: const Color(0xFF14161C),
        // Pas de Center ni de taille fixe : le CustomPaint prend TOUT l'espace.
        body: CustomPaint(
          size: Size.infinite,
          painter: AdaptatifPainter(),
        ),
      ),
    );
  }
}

class AdaptatifPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    // Fond : tout l'espace disponible.
    canvas.drawRect(
      Offset.zero & size,
      Paint()..color = const Color(0xFF20242E),
    );

    // Un cadre à 10 % de marge, toujours proportionnel.
    final marge = size.shortestSide * 0.10;
    final cadre = Rect.fromLTWH(
      marge,
      marge,
      size.width - 2 * marge,
      size.height - 2 * marge,
    );
    canvas.drawRect(
      cadre,
      Paint()
        ..color = const Color(0xFF8FA1C7)
        ..style = PaintingStyle.stroke
        ..strokeWidth = 3,
    );

    // Un disque exactement au centre, de rayon proportionnel.
    canvas.drawCircle(
      size.center(Offset.zero),
      size.shortestSide * 0.15,
      Paint()..color = const Color(0xFFFFC857),
    );
  }

  @override
  bool shouldRepaint(covariant AdaptatifPainter oldDelegate) => false;
}
```

**Résultat :**

```text
Toute la fenêtre est gris foncé.
Un cadre clair suit les bords avec une marge régulière.
Un disque jaune est parfaitement centré.
Si vous redimensionnez la fenêtre, tout suit sans une seule ligne de code en plus.
```

> **Attention à `Size.infinite`.** Il signifie « prends tout ce que le parent t'accorde ». Cela ne fonctionne que si un parent contraint la taille (ici, le `Scaffold`). Si vous mettez `CustomPaint` dans une `Column` sans contrainte de hauteur, Flutter lèvera une erreur de layout. Dans ce cas, utilisez `Expanded`, `SizedBox` ou une taille explicite.

---

## 21.5 — `Offset` : un point

`Offset` est le type qui représente **un point** ou **un déplacement** dans le plan. Il contient deux `double` : `dx` et `dy`.

```dart
const Offset p = Offset(120, 80);   // dx = 120, dy = 80
```

Son nom (« décalage ») trahit sa double nature. Selon le contexte, un `Offset` est :

- **une position** : « le gobelin est à `Offset(120, 80)` » ;
- **un vecteur** : « le gobelin avance de `Offset(3, 0)` par frame ».

C'est le même type, et c'est voulu : additionner une position et un déplacement donne une nouvelle position.

```text
  position          déplacement            nouvelle position
  Offset(120, 80)  +  Offset(3, 0)     =   Offset(123, 80)

     G . . . . . . . . . . . . . . >  G'
```

`Offset` supporte les opérateurs arithmétiques, ce qui rend le code de jeu très lisible.

| Opération | Signification |
| --- | --- |
| `a + b` | somme composante par composante |
| `a - b` | différence (vecteur de `b` vers `a`) |
| `a * 2.0` | mise à l'échelle |
| `a / 2.0` | division |
| `-a` | vecteur opposé |
| `a.distance` | longueur du vecteur (depuis l'origine) |
| `a.distanceSquared` | longueur au carré (pas de racine, plus rapide) |
| `a.direction` | angle en radians |
| `Offset.zero` | le point `(0, 0)` |
| `Offset.fromDirection(angle, longueur)` | point à partir d'un angle |
| `a.scale(sx, sy)` | mise à l'échelle par axe |
| `a.translate(dx, dy)` | translation |

Un programme console pur (exécutable dans DartPad en mode Dart, avec l'import `dart:ui`) illustre tout cela. Ici, on reste dans une application Flutter minimale pour rester homogène avec le reste du chapitre.

```dart
import 'package:flutter/material.dart';

void main() {
  const gobelin = Offset(120, 80);
  const potion = Offset(120, 200);
  const pasVersLaDroite = Offset(3, 0);

  final apresUnPas = gobelin + pasVersLaDroite;
  final versLaPotion = potion - gobelin;

  debugPrint('gobelin        : $gobelin');
  debugPrint('apres un pas   : $apresUnPas');
  debugPrint('vecteur potion : $versLaPotion');
  debugPrint('distance       : ${versLaPotion.distance}');
  debugPrint('double du pas  : ${pasVersLaDroite * 2.0}');
  debugPrint('origine        : ${Offset.zero}');

  runApp(const MaterialApp(
    home: Scaffold(
      backgroundColor: Color(0xFF14161C),
      body: Center(
        child: Text(
          'Voir la console',
          style: TextStyle(color: Colors.white),
        ),
      ),
    ),
  ));
}
```

**Résultat (console) :**

```text
gobelin        : Offset(120.0, 80.0)
apres un pas   : Offset(123.0, 80.0)
vecteur potion : Offset(0.0, 120.0)
distance       : 120.0
double du pas  : Offset(6.0, 0.0)
origine        : Offset(0.0, 0.0)
```

Notez `versLaPotion` : il vaut `Offset(0, 120)`, un vecteur qui pointe **vers le bas** de 120 pixels. Le `dy` positif confirme l'orientation vue en 21.1.

> **Astuce de performance.** Pour comparer deux distances, comparez `distanceSquared` plutôt que `distance`. La racine carrée est inutile et coûteuse. Vous réutiliserez ce réflexe au chapitre 24 sur les collisions circulaires.

> **Immutabilité.** Un `Offset` est immuable, comme un `String`. `a + b` ne modifie ni `a` ni `b` : il crée un nouvel objet. C'est sûr, mais cela crée des objets. Voir la section 21.32 sur les performances.

---

## 21.6 — `Rect` : un rectangle

`Rect` décrit une zone rectangulaire alignée sur les axes. Il stocke quatre `double` : `left`, `top`, `right`, `bottom`.

```text
       left                    right
        │                        │
   top ─┼────────────────────────┼──
        │                        │
        │       le Rect          │
        │                        │
bottom ─┼────────────────────────┼──
        │                        │

   width  = right - left
   height = bottom - top
```

Il existe plusieurs constructeurs, et **choisir le bon rend le code beaucoup plus clair**. Voici les trois que l'on utilise en jeu.

### `Rect.fromLTWH(left, top, width, height)`

Le plus courant. On donne le **coin haut-gauche** et les dimensions.

```dart
final salle = Rect.fromLTWH(40, 60, 200, 120);
// left = 40, top = 60, right = 240, bottom = 180
```

C'est le constructeur naturel pour une tuile de donjon, dont on connaît la case et la taille.

### `Rect.fromCenter(center:, width:, height:)`

On donne le **centre**. C'est le constructeur naturel pour une entité de jeu : un personnage est plus naturellement décrit par sa position centrale que par son coin.

```dart
final gobelin = Rect.fromCenter(
  center: const Offset(120, 80),
  width: 32,
  height: 32,
);
// left = 104, top = 64, right = 136, bottom = 96
```

### `Rect.fromCircle(center:, radius:)`

Le carré qui contient exactement un cercle. Très utile pour dessiner un arc, une ellipse inscrite, ou une hitbox circulaire.

```dart
final aura = Rect.fromCircle(center: const Offset(120, 80), radius: 40);
// left = 80, top = 40, right = 160, bottom = 120  (côté = 2 * rayon)
```

Il en existe deux autres, plus rares mais pratiques :

| Constructeur | Usage |
| --- | --- |
| `Rect.fromLTRB(l, t, r, b)` | on connaît les quatre bords |
| `Rect.fromPoints(a, b)` | rectangle englobant deux points (ordre indifférent) |
| `Offset & Size` | raccourci : `Offset(10, 10) & Size(50, 30)` |

Et quelques propriétés très utilisées :

| Propriété | Valeur |
| --- | --- |
| `rect.width`, `rect.height` | dimensions |
| `rect.center` | centre, sous forme d'`Offset` |
| `rect.topLeft`, `rect.bottomRight` | coins |
| `rect.size` | dimensions sous forme de `Size` |
| `rect.deflate(v)` | rectangle rétréci de `v` sur chaque bord |
| `rect.inflate(v)` | rectangle agrandi de `v` sur chaque bord |
| `rect.contains(offset)` | le point est-il dedans ? |
| `rect.overlaps(autre)` | les deux rectangles se croisent-ils ? |

Voici les trois constructeurs dessinés côte à côte, avec leur centre matérialisé.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const RectApp());

class RectApp extends StatelessWidget {
  const RectApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: const Color(0xFF14161C),
        body: Center(
          child: CustomPaint(
            size: const Size(420, 200),
            painter: RectPainter(),
          ),
        ),
      ),
    );
  }
}

class RectPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
      Offset.zero & size,
      Paint()..color = const Color(0xFF20242E),
    );

    final trait = Paint()
      ..style = PaintingStyle.stroke
      ..strokeWidth = 2;
    final centre = Paint()..color = const Color(0xFFFFC857);

    // 1) fromLTWH : coin haut-gauche + dimensions
    final a = Rect.fromLTWH(20, 40, 100, 80);
    canvas.drawRect(a, trait..color = const Color(0xFF6FCF6F));
    canvas.drawCircle(a.center, 4, centre);

    // 2) fromCenter : centre + dimensions
    final b = Rect.fromCenter(
      center: const Offset(210, 80),
      width: 100,
      height: 80,
    );
    canvas.drawRect(b, trait..color = const Color(0xFF5AA9E0));
    canvas.drawCircle(b.center, 4, centre);

    // 3) fromCircle : centre + rayon (côté = 2 * rayon)
    final c = Rect.fromCircle(center: const Offset(350, 80), radius: 45);
    canvas.drawRect(c, trait..color = const Color(0xFFE05A78));
    canvas.drawCircle(c.center, 4, centre);
    canvas.drawCircle(c.center, 45, trait..color = const Color(0xFF7A4050));
  }

  @override
  bool shouldRepaint(covariant RectPainter oldDelegate) => false;
}
```

**Résultat :**

```text
Trois rectangles vides côte à côte, chacun avec un point jaune en son centre.
Le troisième contient en plus le cercle inscrit, dont le rectangle est
exactement la boîte englobante.
```

> **Piège classique.** `Rect.fromLTWH(x, y, w, h)` et `Rect.fromLTRB(l, t, r, b)` se ressemblent à l'écrit. Confondre les deux donne un rectangle démesuré ou inversé. Relisez toujours le nom du constructeur avant de compter vos arguments.

> **Rectangle vide ou inversé.** Si `right < left`, le rectangle est dit « inversé » et `isEmpty` vaut `true`. Rien ne sera dessiné. Utilisez `Rect.fromPoints(a, b)` si l'ordre des points n'est pas garanti : il normalise automatiquement.

---

## 21.7 — `Size`

`Size` ne contient que deux nombres : `width` et `height`. Pas de position. C'est une **dimension pure**.

```dart
const Size tuile = Size(32, 32);
const Size salle = Size(320, 240);
```

Pourquoi un type distinct de `Offset`, alors que les deux ne sont que deux `double` ? Pour la **sécurité de type**. Une taille n'est pas un point. Si les deux étaient interchangeables, on écrirait un jour `drawCircle(taille, ...)` sans que le compilateur bronche. Avec deux types différents, l'erreur est refusée à la compilation. C'est le même raisonnement que celui du chapitre 12 sur le null safety : faire dire « non » au compilateur plutôt qu'au programme en cours d'exécution.

Les membres utiles :

| Membre | Rôle |
| --- | --- |
| `size.width`, `size.height` | les deux dimensions |
| `size.aspectRatio` | rapport largeur / hauteur |
| `size.shortestSide`, `size.longestSide` | le plus petit / le plus grand côté |
| `size.center(origine)` | centre, à partir d'un point d'origine |
| `size.topLeft(origine)` | coin haut-gauche à partir d'une origine |
| `size.isEmpty` | vrai si une dimension est nulle ou négative |
| `Size.zero` | `Size(0, 0)` |
| `Size.square(48)` | un carré |
| `Size.infinite` | dimensions infinies (usage layout) |

Le pont entre les trois types se fait ainsi :

```text
  Offset  +  Size          ->  Rect      (opérateur &)
  Rect    .  size          ->  Size
  Rect    .  center        ->  Offset
  Size    .  center(origin)->  Offset
```

Un exemple qui utilise les trois ensemble pour placer une tuile de donjon :

```dart
import 'package:flutter/material.dart';

void main() => runApp(const SizeApp());

class SizeApp extends StatelessWidget {
  const SizeApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: const Color(0xFF14161C),
        body: Center(
          child: CustomPaint(
            size: const Size(360, 200),
            painter: SizePainter(),
          ),
        ),
      ),
    );
  }
}

class SizePainter extends CustomPainter {
  static const Size tuile = Size(48, 48);

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
      Offset.zero & size,
      Paint()..color = const Color(0xFF20242E),
    );

    final mur = Paint()..color = const Color(0xFF3C4457);

    // On pose 5 tuiles en ligne grâce à l'opérateur & (Offset & Size -> Rect).
    for (int i = 0; i < 5; i++) {
      final coin = Offset(20 + i * (tuile.width + 6), 60);
      final Rect caseDonjon = coin & tuile;
      canvas.drawRect(caseDonjon, mur);

      // Le centre de la tuile, calculé depuis la Size.
      canvas.drawCircle(
        tuile.center(coin),
        4,
        Paint()..color = const Color(0xFFFFC857),
      );
    }
  }

  @override
  bool shouldRepaint(covariant SizePainter oldDelegate) => false;
}
```

**Résultat :**

```text
Cinq carrés gris-bleu alignés horizontalement, séparés par 6 pixels.
Un point jaune au centre exact de chacun.
```

---

## 21.8 — `Vector2` de Flame contre `Offset` de Flutter

Au chapitre 27, vous installerez Flame. Vous découvrirez alors qu'il n'utilise **pas** `Offset` mais un type nommé `Vector2`, importé du package `vector_math`. Autant comprendre la différence tout de suite, cela vous évitera une bonne demi-journée de confusion.

| | `Offset` (Flutter) | `Vector2` (Flame) |
| --- | --- | --- |
| Composantes | `dx`, `dy` | `x`, `y` |
| Mutabilité | **immuable** | **mutable** |
| Modification | crée un nouvel objet | modifie l'objet existant |
| Addition | `a + b` (nouvel objet) | `a + b` ou `a.add(b)` (en place) |
| Origine | `dart:ui` | package `vector_math` |
| Usage | dessin, layout, `Canvas` | positions d'entités dans Flame |

L'écart le plus important est la **mutabilité**.

```text
  OFFSET (immuable)
  ────────────────────────────────────────
  var p = Offset(10, 20);
  p = p + Offset(3, 0);        <- on REMPLACE p par un nouvel objet

        avant :  [10, 20]      (objet 1)
        après :  [13, 20]      (objet 2, l'objet 1 est jeté)


  VECTOR2 (mutable)
  ────────────────────────────────────────
  final p = Vector2(10, 20);
  p.x += 3;                    <- on MODIFIE l'objet en place

        avant :  [10, 20]      (objet 1)
        après :  [13, 20]      (toujours l'objet 1)
```

Pourquoi Flame a-t-il fait ce choix ? Pour la **performance**. Un jeu qui bouge 500 entités à 60 images par seconde crée, avec `Offset`, 30 000 objets par seconde rien que pour les positions. Le ramasse-miettes (garbage collector) doit les nettoyer, ce qui produit de micro-saccades. Avec `Vector2`, on modifie les objets existants : zéro allocation.

Le prix à payer est un risque d'aliasing (deux variables qui désignent le même objet) :

```text
  final a = Vector2(0, 0);
  final b = a;          <- b et a désignent LE MÊME objet
  b.x = 100;
  print(a.x);           -> 100 !  a a changé aussi

  Avec Offset, ce piège n'existe pas.
```

Dans Flame, on écrit donc `b = a.clone()` quand on veut une vraie copie.

La conversion entre les deux mondes est immédiate, car Flame fournit des extensions :

```text
  Vector2  ->  Offset  :  monVecteur.toOffset()
  Offset   ->  Vector2 :  monOffset.toVector2()
  Vector2  ->  Size    :  monVecteur.toSize()
```

> **Ce chapitre reste en Flutter pur.** Nous utilisons donc `Offset` partout. Retenez simplement que `Vector2` arrive au chapitre 27 et qu'il est **mutable** : c'est la seule différence qui vous piégera.

---

## 21.9 — `CustomPaint` et `CustomPainter` : rappel et détail

Vous avez rencontré ce duo au chapitre 19. Reprenons-le en détail, car tout ce chapitre repose dessus.

Il y a **deux** classes, et on les confond souvent :

| Classe | Nature | Rôle |
| --- | --- | --- |
| `CustomPaint` | un **widget** | réserve une zone dans l'arbre des widgets |
| `CustomPainter` | une **classe abstraite** | contient le code qui dessine dans cette zone |

Le schéma de leur relation :

```text
  ARBRE DES WIDGETS                     VOTRE CODE

  MaterialApp
    └─ Scaffold
         └─ Center
              └─ CustomPaint  ──────>  painter: DonjonPainter()
                    │                        │
                    │                        │  extends CustomPainter
                    │                        v
                    │                   void paint(Canvas c, Size s) {
                    │                     c.drawRect(...);
                    │                     c.drawCircle(...);
                    │                   }
                    v
              zone rectangulaire
              de taille "size"
```

Le widget `CustomPaint` accepte trois choses à dessiner :

```dart
CustomPaint(
  painter: FondPainter(),        // dessiné AVANT l'enfant (arrière-plan)
  foregroundPainter: HudPainter(),// dessiné APRÈS l'enfant (premier plan)
  size: const Size(400, 300),    // taille souhaitée s'il n'y a pas d'enfant
  child: const Text('bonjour'),  // widget classique par-dessus le painter
)
```

L'ordre est fixe et important :

```text
  1. painter.paint()           <- le fond
  2. child (widgets normaux)   <- au milieu
  3. foregroundPainter.paint() <- par-dessus tout
```

Pour un jeu, on utilise presque toujours `painter` seul, sans `child`, et on dessine tout à la main.

Attention à la règle de taille :

- **avec un `child`** : `CustomPaint` prend la taille du `child`, et `size` est ignoré ;
- **sans `child`** : `CustomPaint` prend la valeur de `size` (par défaut `Size.zero`, donc **rien ne s'affiche** si vous l'oubliez).

C'est l'erreur numéro un des débutants : un `CustomPaint` sans `child` et sans `size` occupe zéro pixel, et l'écran reste vide alors que le code de `paint()` est parfait.

Voici le squelette complet et minimal, à recopier au début de chaque projet de ce chapitre.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const DonjonApp());

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Donjon de Dart',
      home: Scaffold(
        backgroundColor: const Color(0xFF14161C),
        body: Center(
          child: CustomPaint(
            size: const Size(400, 300),   // OBLIGATOIRE sans child
            painter: DonjonPainter(),
          ),
        ),
      ),
    );
  }
}

class DonjonPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    // Tout le dessin se passe ici.
    canvas.drawRect(
      Offset.zero & size,
      Paint()..color = const Color(0xFF20242E),
    );
    canvas.drawCircle(
      size.center(Offset.zero),
      40,
      Paint()..color = const Color(0xFFFFC857),
    );
  }

  @override
  bool shouldRepaint(covariant DonjonPainter oldDelegate) => false;
}
```

**Résultat :**

```text
Un rectangle gris foncé de 400 x 300, centré dans la fenêtre,
avec un disque jaune en son milieu.
```

> **Remarque.** `CustomPainter` étend `Listenable`-compatible via son constructeur `super(repaint: ...)`. On peut lui passer un `Animation` ou un `ValueNotifier` qui déclenchera un repeint automatique. Nous en verrons l'usage en section 21.35.

---

## 21.10 — `paint()` et `shouldRepaint()`

Un `CustomPainter` a exactement deux méthodes obligatoires. Elles ont des rôles très différents.

### `paint(Canvas canvas, Size size)`

C'est **la** méthode de dessin. Elle est appelée par le moteur de rendu quand la zone doit être redessinée. Vous n'appelez jamais `paint()` vous-même.

Deux règles absolues :

1. `paint()` ne doit **rien modifier** en dehors du dessin. Pas de `setState()`, pas de mise à jour de la logique de jeu, pas de calcul de collisions. La logique va dans `update(dt)` (chapitre 20), le dessin va dans `paint()`. C'est la séparation logique / rendu, qui sera formalisée au chapitre 26.
2. `paint()` doit être **rapide**. À 60 images par seconde, vous disposez de 16,7 millisecondes pour tout faire. Voir la section 21.32.

### `shouldRepaint(covariant CustomPainter oldDelegate)`

Cette méthode répond à une question précise :

> « L'objet `CustomPainter` vient d'être remplacé par un nouveau. Le dessin doit-il être refait ? »

Flutter la consulte lorsqu'un nouveau `CustomPainter` est fourni au même `CustomPaint`. Si vous renvoyez `false`, Flutter réutilise le dessin précédent, déjà en cache. Si vous renvoyez `true`, il rappelle `paint()`.

```text
  setState()  ->  build()  ->  nouveau CustomPainter
                                     │
                                     v
                       shouldRepaint(ancien) ?
                          │                │
                        false            true
                          │                │
                  garde l'image      appelle paint()
                     en cache          à nouveau
```

Le paramètre `oldDelegate` est **l'ancien** painter, celui qui a produit le dessin actuel. Le mot-clé `covariant` permet de le typer précisément dans votre sous-classe, ce qui évite un `as` disgracieux. C'est un usage direct du polymorphisme du chapitre 10.

La règle pratique tient en trois lignes :

| Situation | Retour |
| --- | --- |
| Le painter n'a aucun champ, le dessin est fixe | `false` |
| Le painter a des champs, comparez-les | `oldDelegate.x != x` |
| Vous ne savez pas / phase de mise au point | `true` |

Voici un painter avec état, dont le `shouldRepaint` est écrit correctement.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const RepaintApp());

class RepaintApp extends StatefulWidget {
  const RepaintApp({super.key});

  @override
  State<RepaintApp> createState() => _RepaintAppState();
}

class _RepaintAppState extends State<RepaintApp> {
  double _x = 60;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: const Color(0xFF14161C),
        body: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            CustomPaint(
              size: const Size(360, 140),
              // Un NOUVEAU painter est créé à chaque build.
              painter: HerosPainter(x: _x),
            ),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: () => setState(() => _x += 30),
              child: const Text('Avancer de 30'),
            ),
          ],
        ),
      ),
    );
  }
}

class HerosPainter extends CustomPainter {
  const HerosPainter({required this.x});

  final double x;

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
      Offset.zero & size,
      Paint()..color = const Color(0xFF20242E),
    );
    canvas.drawCircle(
      Offset(x, size.height / 2),
      20,
      Paint()..color = const Color(0xFF6FCF6F),
    );
  }

  // On compare l'ancien état au nouveau : c'est la bonne façon de faire.
  @override
  bool shouldRepaint(covariant HerosPainter oldDelegate) {
    return oldDelegate.x != x;
  }
}
```

**Résultat :**

```text
Un disque vert dans une bande grise, et un bouton dessous.
Chaque clic déplace le disque de 30 pixels vers la droite.
```

Si vous remplacez le corps de `shouldRepaint` par `return false;`, le bouton continue de fonctionner (`_x` augmente bien), mais **le disque ne bouge plus**. C'est un des bugs les plus déroutants du débutant : la logique est correcte, l'écran ment. Testez-le, cela vous vaccinera.

> **Ne renvoyez pas `true` systématiquement « au cas où ».** Cela marche, mais vous perdez le cache de rendu. Sur un écran complexe avec plusieurs painters, cela se voit.

---

## 21.11 — L'objet `Paint` : `color`, `style`, `strokeWidth`

Le `Canvas` sait dessiner des formes. Le `Paint` décrit **comment** elles sont dessinées. C'est le pinceau, et il est passé en dernier argument à toutes les méthodes de dessin.

```dart
final pinceau = Paint()
  ..color = const Color(0xFFE05A78)
  ..style = PaintingStyle.stroke
  ..strokeWidth = 4;
```

Notez l'opérateur `..` (cascade), vu au chapitre 9 : il permet de configurer un objet en une seule expression, sans variable temporaire.

Les propriétés principales :

| Propriété | Type | Rôle | Défaut |
| --- | --- | --- | --- |
| `color` | `Color` | couleur du tracé ou du remplissage | noir opaque |
| `style` | `PaintingStyle` | `fill` (rempli) ou `stroke` (contour) | `fill` |
| `strokeWidth` | `double` | épaisseur du contour | `0.0` |
| `strokeCap` | `StrokeCap` | forme des extrémités : `butt`, `round`, `square` | `butt` |
| `strokeJoin` | `StrokeJoin` | forme des angles : `miter`, `round`, `bevel` | `miter` |
| `isAntiAlias` | `bool` | lissage des bords | `true` |
| `shader` | `Shader?` | dégradé ou motif (section 21.22) | `null` |
| `blendMode` | `BlendMode` | mode de fusion avec le fond | `srcOver` |
| `maskFilter` | `MaskFilter?` | flou (halo, ombre) | `null` |

Un point souvent mal compris : **`strokeWidth = 0` ne signifie pas « invisible »**. Cela signifie « la ligne la plus fine possible », soit un pixel physique (hairline). Pour rendre une ligne invisible, réduisez l'alpha de la couleur ou ne la dessinez pas.

Autre point : le trait est **centré sur le contour**. Un `strokeWidth` de 10 déborde de 5 pixels vers l'intérieur et de 5 vers l'extérieur.

```text
  Rect demandé (strokeWidth = 10)

    <-5-><-5->
  ░░░░░████████████████████░░░░░
  ░░░░░█                  █░░░░░
        █   intérieur     █
        █                 █
  ░░░░░████████████████████░░░░░

  Le trait déborde de la moitié de son épaisseur de chaque côté.
  Un Rect.fromLTWH(0, 0, 100, 100) avec strokeWidth = 10 occupe
  en réalité de -5 à 105 : les bords sont coupés par le canvas.
```

Retenez ce comportement : c'est la raison pour laquelle un cadre dessiné exactement sur `Offset.zero & size` apparaît deux fois plus fin qu'attendu (la moitié externe est coupée). La solution est `rect.deflate(strokeWidth / 2)`.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const PaintApp());

class PaintApp extends StatelessWidget {
  const PaintApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: const Color(0xFF14161C),
        body: Center(
          child: CustomPaint(
            size: const Size(400, 220),
            painter: PaintPainter(),
          ),
        ),
      ),
    );
  }
}

class PaintPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
      Offset.zero & size,
      Paint()..color = const Color(0xFF20242E),
    );

    // Cadre NAÏF : la moitié externe du trait est coupée.
    canvas.drawRect(
      Offset.zero & size,
      Paint()
        ..color = const Color(0xFFE05A78)
        ..style = PaintingStyle.stroke
        ..strokeWidth = 12,
    );

    // Cadre CORRIGÉ : on rentre de la moitié de l'épaisseur.
    canvas.drawRect(
      (Offset.zero & size).deflate(6 + 12),
      Paint()
        ..color = const Color(0xFF6FCF6F)
        ..style = PaintingStyle.stroke
        ..strokeWidth = 12,
    );

    // Trois extrémités de ligne différentes.
    for (int i = 0; i < 3; i++) {
      final cap = [StrokeCap.butt, StrokeCap.round, StrokeCap.square][i];
      canvas.drawLine(
        Offset(120, 80.0 + i * 30),
        Offset(280, 80.0 + i * 30),
        Paint()
          ..color = const Color(0xFFFFC857)
          ..strokeWidth = 14
          ..strokeCap = cap,
      );
    }
  }

  @override
  bool shouldRepaint(covariant PaintPainter oldDelegate) => false;
}
```

**Résultat :**

```text
Un cadre rouge collé aux bords, visiblement plus fin que 12 pixels
(sa moitié externe est hors zone).
Un cadre vert à l'intérieur, d'épaisseur pleine.
Trois barres jaunes : la première à bouts francs, la deuxième à bouts arrondis
(plus longue de 14 px au total), la troisième à bouts carrés (également plus longue).
```

---

## 21.12 — `PaintingStyle.fill` contre `PaintingStyle.stroke`

C'est le réglage le plus structurant du `Paint`.

```text
  PaintingStyle.fill                PaintingStyle.stroke
  (valeur par défaut)               (contour, strokeWidth utilisé)

  ████████████████                  ┌──────────────┐
  ████████████████                  │              │
  ████████████████                  │              │
  ████████████████                  │              │
  ████████████████                  └──────────────┘

  L'intérieur est peint.            Seul le bord est peint.
  strokeWidth est IGNORÉ.           strokeWidth donne l'épaisseur.
```

Il n'existe pas de style « rempli **et** contouré » en une seule passe. Pour obtenir les deux, on dessine **deux fois** la même forme, avec deux `Paint` différents. C'est le motif standard :

```dart
canvas.drawCircle(centre, 40, remplissage);  // d'abord l'intérieur
canvas.drawCircle(centre, 40, contour);      // puis le bord par-dessus
```

L'ordre compte : le contour doit venir **après**, sinon le remplissage le recouvre à moitié (voir 21.23).

Un cas particulier utile : `PaintingStyle.stroke` avec `drawPoints` produit des points visibles, alors que `fill` ne dessine rien du tout (section 21.19).

```dart
import 'package:flutter/material.dart';

void main() => runApp(const StyleApp());

class StyleApp extends StatelessWidget {
  const StyleApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: const Color(0xFF14161C),
        body: Center(
          child: CustomPaint(
            size: const Size(420, 180),
            painter: StylePainter(),
          ),
        ),
      ),
    );
  }
}

class StylePainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
      Offset.zero & size,
      Paint()..color = const Color(0xFF20242E),
    );

    final remplissage = Paint()
      ..color = const Color(0xFF5AA9E0)
      ..style = PaintingStyle.fill;

    final contour = Paint()
      ..color = const Color(0xFFFFC857)
      ..style = PaintingStyle.stroke
      ..strokeWidth = 5;

    // 1) Uniquement rempli.
    canvas.drawCircle(const Offset(90, 90), 45, remplissage);

    // 2) Uniquement contouré.
    canvas.drawCircle(const Offset(210, 90), 45, contour);

    // 3) Les deux : deux appels, remplissage puis contour.
    canvas.drawCircle(const Offset(330, 90), 45, remplissage);
    canvas.drawCircle(const Offset(330, 90), 45, contour);
  }

  @override
  bool shouldRepaint(covariant StylePainter oldDelegate) => false;
}
```

**Résultat :**

```text
Trois disques alignés :
 - le premier bleu plein, sans bordure ;
 - le deuxième vide, avec une bordure jaune de 5 px ;
 - le troisième bleu plein avec une bordure jaune.
```

> **Erreur fréquente.** Régler `strokeWidth` en oubliant `style = PaintingStyle.stroke` n'a aucun effet : la forme reste pleine. Le compilateur ne dit rien, puisque les deux propriétés sont valides séparément.

---

## 21.13 — `drawRect()`

La méthode la plus utilisée d'un jeu 2D en formes géométriques.

```dart
void drawRect(Rect rect, Paint paint)
```

Elle dessine un rectangle aligné sur les axes. Pour un rectangle incliné, il faut passer par `rotate()` (section 21.26) ou par un `Path` (section 21.18).

Construisons le sol, les murs et un coffre du Donjon de Dart.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const RectDessinApp());

class RectDessinApp extends StatelessWidget {
  const RectDessinApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: const Color(0xFF14161C),
        body: Center(
          child: CustomPaint(
            size: const Size(420, 260),
            painter: SallePainter(),
          ),
        ),
      ),
    );
  }
}

class SallePainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    final sol = Paint()..color = const Color(0xFF2A2F3C);
    final mur = Paint()..color = const Color(0xFF454E63);
    final coffre = Paint()..color = const Color(0xFF8A6A3B);
    final ferrure = Paint()..color = const Color(0xFFFFC857);

    // Le sol de la salle : toute la zone.
    canvas.drawRect(Offset.zero & size, sol);

    // Quatre murs de 20 px d'épaisseur.
    const e = 20.0;
    canvas.drawRect(Rect.fromLTWH(0, 0, size.width, e), mur);
    canvas.drawRect(Rect.fromLTWH(0, size.height - e, size.width, e), mur);
    canvas.drawRect(Rect.fromLTWH(0, 0, e, size.height), mur);
    canvas.drawRect(Rect.fromLTWH(size.width - e, 0, e, size.height), mur);

    // Un coffre, décrit par son centre.
    final boite = Rect.fromCenter(
      center: Offset(size.width / 2, size.height / 2),
      width: 90,
      height: 60,
    );
    canvas.drawRect(boite, coffre);

    // La serrure : un petit rectangle centré sur le coffre.
    canvas.drawRect(
      Rect.fromCenter(center: boite.center, width: 14, height: 20),
      ferrure,
    );

    // La bande métallique horizontale.
    canvas.drawRect(
      Rect.fromLTWH(boite.left, boite.center.dy - 4, boite.width, 8),
      ferrure,
    );
  }

  @override
  bool shouldRepaint(covariant SallePainter oldDelegate) => false;
}
```

**Résultat :**

```text
Une salle rectangulaire au sol gris sombre, entourée de quatre murs plus clairs
de 20 pixels d'épaisseur.
Au centre, un coffre brun traversé par une bande dorée horizontale,
avec une serrure dorée verticale au milieu.
```

> **Note.** Les quatre murs se chevauchent aux angles. Ce n'est pas un problème ici puisqu'ils sont de la même couleur, mais retenez-le : dessiner deux fois le même pixel coûte du temps (section 21.32).

---

## 21.14 — `drawCircle()`

```dart
void drawCircle(Offset center, double radius, Paint paint)
```

Trois choses à retenir :

1. le premier argument est le **centre**, pas un coin ;
2. le second est le **rayon**, pas le diamètre ;
3. avec `PaintingStyle.stroke`, le trait est centré sur la circonférence.

```text
        rayon
      <────────>
          ┌ ─ ─ ─ ─ ┐
        ╱             ╲
       │       x       │   x = center
        ╲             ╱
          └ ─ ─ ─ ─ ┘

  Le disque va de (cx - r, cy - r) à (cx + r, cy + r).
  Sa boîte englobante est Rect.fromCircle(center: c, radius: r).
```

Confondre rayon et diamètre est l'erreur classique : votre potion apparaît deux fois trop grosse.

Dessinons une potion, une pièce d'or et un halo autour du héros.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const CercleApp());

class CercleApp extends StatelessWidget {
  const CercleApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: const Color(0xFF14161C),
        body: Center(
          child: CustomPaint(
            size: const Size(420, 220),
            painter: CerclePainter(),
          ),
        ),
      ),
    );
  }
}

class CerclePainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
      Offset.zero & size,
      Paint()..color = const Color(0xFF20242E),
    );

    // 1) La potion : un disque rouge avec un reflet.
    const centrePotion = Offset(90, 110);
    canvas.drawCircle(
      centrePotion,
      40,
      Paint()..color = const Color(0xFFE05A78),
    );
    canvas.drawCircle(
      centrePotion + const Offset(-13, -13),
      10,
      Paint()..color = const Color(0x88FFFFFF),
    );

    // 2) La pièce d'or : deux disques concentriques.
    const centrePiece = Offset(210, 110);
    canvas.drawCircle(
      centrePiece,
      34,
      Paint()..color = const Color(0xFFC79A2E),
    );
    canvas.drawCircle(
      centrePiece,
      26,
      Paint()..color = const Color(0xFFFFC857),
    );

    // 3) Le halo du héros : un contour épais.
    const centreHeros = Offset(330, 110);
    canvas.drawCircle(
      centreHeros,
      22,
      Paint()..color = const Color(0xFF6FCF6F),
    );
    canvas.drawCircle(
      centreHeros,
      40,
      Paint()
        ..color = const Color(0x666FCF6F)
        ..style = PaintingStyle.stroke
        ..strokeWidth = 6,
    );
  }

  @override
  bool shouldRepaint(covariant CerclePainter oldDelegate) => false;
}
```

**Résultat :**

```text
De gauche à droite :
 - un disque rouge avec un reflet blanc translucide en haut à gauche ;
 - une pièce dorée composée d'un anneau sombre et d'un coeur clair ;
 - un petit disque vert entouré d'un anneau vert translucide.
```

> **Astuce.** `drawCircle` est un cas particulier de `drawOval` où le `Rect` est carré. `drawCircle(c, r, p)` équivaut exactement à `drawOval(Rect.fromCircle(center: c, radius: r), p)`.

---

## 21.15 — `drawLine()`

```dart
void drawLine(Offset p1, Offset p2, Paint paint)
```

Deux points, un segment. Le `style` du `Paint` est ignoré : une ligne est toujours tracée en mode contour. Ce qui compte, c'est `strokeWidth` et `strokeCap`.

Une ligne d'épaisseur `strokeWidth = 0` reste visible : c'est le trait le plus fin que l'appareil sait produire.

Servons-nous des lignes pour tracer la grille du donjon et un rayon lumineux.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const LigneApp());

class LigneApp extends StatelessWidget {
  const LigneApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: const Color(0xFF14161C),
        body: Center(
          child: CustomPaint(
            size: const Size(400, 240),
            painter: LignePainter(),
          ),
        ),
      ),
    );
  }
}

class LignePainter extends CustomPainter {
  static const double pas = 40;

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
      Offset.zero & size,
      Paint()..color = const Color(0xFF20242E),
    );

    // Grille : lignes verticales puis horizontales.
    final grille = Paint()
      ..color = const Color(0xFF3A4152)
      ..strokeWidth = 1;

    for (double x = 0; x <= size.width; x += pas) {
      canvas.drawLine(Offset(x, 0), Offset(x, size.height), grille);
    }
    for (double y = 0; y <= size.height; y += pas) {
      canvas.drawLine(Offset(0, y), Offset(size.width, y), grille);
    }

    // Un rayon : ligne épaisse à bouts arrondis.
    canvas.drawLine(
      const Offset(60, 200),
      const Offset(340, 50),
      Paint()
        ..color = const Color(0xFFFFC857)
        ..strokeWidth = 8
        ..strokeCap = StrokeCap.round,
    );

    // Une croix marquant un piège.
    final piege = Paint()
      ..color = const Color(0xFFE05A78)
      ..strokeWidth = 4
      ..strokeCap = StrokeCap.round;
    const c = Offset(120, 80);
    canvas.drawLine(c + const Offset(-14, -14), c + const Offset(14, 14), piege);
    canvas.drawLine(c + const Offset(14, -14), c + const Offset(-14, 14), piege);
  }

  @override
  bool shouldRepaint(covariant LignePainter oldDelegate) => false;
}
```

**Résultat :**

```text
Une grille régulière de 40 pixels de côté sur fond sombre.
Une large diagonale dorée traverse la zone du bas-gauche vers le haut-droit.
Une croix rouge marque une case en haut à gauche.
```

> **Ligne floue ?** Une ligne d'épaisseur 1 tracée à une coordonnée entière est centrée sur la frontière entre deux pixels : elle s'étale sur deux demi-pixels et paraît grise. Pour une ligne nette d'épaisseur 1, tracez-la sur une demi-coordonnée : `Offset(x + 0.5, 0)`.

---

## 21.16 — `drawOval()`

```dart
void drawOval(Rect rect, Paint paint)
```

`drawOval` dessine l'ellipse **inscrite** dans le rectangle fourni. On ne donne donc pas un centre et deux rayons, mais une boîte englobante.

```text
    Rect.fromLTWH(0, 0, 160, 80)

    ┌──────────────────────────────┐
    │        ,-'''''''''''-.       │
    │      ,'               `.     │
    │     (                   )    │
    │      `.               ,'     │
    │        `-.........-'         │
    └──────────────────────────────┘

    L'ellipse touche les quatre bords du Rect.
    Rect carré -> cercle parfait.
```

En jeu, l'ellipse sert surtout à une chose : **l'ombre portée au sol**. Une ombre circulaire aplatie donne immédiatement l'impression qu'un personnage repose sur le sol.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const OvaleApp());

class OvaleApp extends StatelessWidget {
  const OvaleApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: const Color(0xFF14161C),
        body: Center(
          child: CustomPaint(
            size: const Size(400, 240),
            painter: OvalePainter(),
          ),
        ),
      ),
    );
  }
}

class OvalePainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
      Offset.zero & size,
      Paint()..color = const Color(0xFF20242E),
    );

    // Le sol.
    canvas.drawRect(
      Rect.fromLTWH(0, 170, size.width, size.height - 170),
      Paint()..color = const Color(0xFF2A2F3C),
    );

    // Ombre portée : ellipse large et plate.
    canvas.drawOval(
      Rect.fromCenter(
        center: const Offset(120, 172),
        width: 70,
        height: 18,
      ),
      Paint()..color = const Color(0x66000000),
    );

    // Le héros, posé juste au-dessus de son ombre.
    canvas.drawRect(
      Rect.fromCenter(center: const Offset(120, 140), width: 34, height: 60),
      Paint()..color = const Color(0xFF6FCF6F),
    );

    // Un oeil de boss : ellipse contourée + pupille.
    final oeil = Rect.fromCenter(
      center: const Offset(290, 100),
      width: 130,
      height: 70,
    );
    canvas.drawOval(oeil, Paint()..color = const Color(0xFFF0EAD8));
    canvas.drawOval(
      oeil,
      Paint()
        ..color = const Color(0xFF14161C)
        ..style = PaintingStyle.stroke
        ..strokeWidth = 4,
    );
    canvas.drawCircle(
      oeil.center,
      22,
      Paint()..color = const Color(0xFFE05A78),
    );
    canvas.drawCircle(
      oeil.center,
      9,
      Paint()..color = const Color(0xFF14161C),
    );
  }

  @override
  bool shouldRepaint(covariant OvalePainter oldDelegate) => false;
}
```

**Résultat :**

```text
Un rectangle vert (le héros) posé sur une ombre ovale sombre, sur un sol plus clair.
À droite, un grand oeil : ellipse blanche cernée de noir, iris rouge, pupille noire.
```

---

## 21.17 — `drawRRect()`

`RRect` signifie « rounded rectangle » : un rectangle à coins arrondis.

```dart
void drawRRect(RRect rrect, Paint paint)
```

On ne construit pas un `RRect` directement, mais à partir d'un `Rect` et d'un ou plusieurs rayons :

| Constructeur | Effet |
| --- | --- |
| `RRect.fromRectAndRadius(rect, Radius.circular(12))` | les 4 coins arrondis pareil |
| `RRect.fromRectXY(rect, 20, 8)` | rayons horizontal et vertical différents |
| `RRect.fromRectAndCorners(rect, topLeft: ..., bottomRight: ...)` | chaque coin séparément |

```text
   Radius.circular(0)        Radius.circular(12)      Radius.circular(40)

   ┌────────────────┐        ╭────────────────╮        ╭──────────────╮
   │                │        │                │       (                )
   │                │        │                │       (                )
   └────────────────┘        ╰────────────────╯        ╰──────────────╯

   Si le rayon dépasse la moitié du plus petit côté,
   Flutter le réduit automatiquement (pas d'erreur, mais forme "pilule").
```

C'est la forme reine des interfaces de jeu : barres de vie, boutons, cartouches de dialogue, cadre d'inventaire.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const RRectApp());

class RRectApp extends StatelessWidget {
  const RRectApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: const Color(0xFF14161C),
        body: Center(
          child: CustomPaint(
            size: const Size(420, 220),
            painter: BarreVieePainter(vie: 0.62),
          ),
        ),
      ),
    );
  }
}

class BarreVieePainter extends CustomPainter {
  const BarreVieePainter({required this.vie});

  /// Fraction de vie restante, entre 0.0 et 1.0.
  final double vie;

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
      Offset.zero & size,
      Paint()..color = const Color(0xFF20242E),
    );

    // Cadre d'inventaire, coins très arrondis.
    canvas.drawRRect(
      RRect.fromRectAndRadius(
        Rect.fromLTWH(30, 30, size.width - 60, 60),
        const Radius.circular(18),
      ),
      Paint()..color = const Color(0xFF2E3444),
    );

    // Barre de vie : fond puis remplissage.
    final fondBarre = Rect.fromLTWH(30, 130, size.width - 60, 26);
    const rayon = Radius.circular(13);

    canvas.drawRRect(
      RRect.fromRectAndRadius(fondBarre, rayon),
      Paint()..color = const Color(0xFF3A2530),
    );

    final remplie = Rect.fromLTWH(
      fondBarre.left,
      fondBarre.top,
      fondBarre.width * vie.clamp(0.0, 1.0),
      fondBarre.height,
    );
    canvas.drawRRect(
      RRect.fromRectAndRadius(remplie, rayon),
      Paint()..color = const Color(0xFFE05A78),
    );

    // Contour de la barre.
    canvas.drawRRect(
      RRect.fromRectAndRadius(fondBarre, rayon),
      Paint()
        ..color = const Color(0xFFF0EAD8)
        ..style = PaintingStyle.stroke
        ..strokeWidth = 2,
    );

    // Une "pilule" : rayon supérieur à la moitié de la hauteur.
    canvas.drawRRect(
      RRect.fromRectAndRadius(
        Rect.fromLTWH(30, 175, 120, 30),
        const Radius.circular(99),
      ),
      Paint()..color = const Color(0xFF5AA9E0),
    );
  }

  @override
  bool shouldRepaint(covariant BarreVieePainter oldDelegate) {
    return oldDelegate.vie != vie;
  }
}
```

**Résultat :**

```text
Un grand cadre arrondi gris-bleu en haut.
Une barre de vie remplie à 62 % en rouge, sur fond sombre, cerclée de blanc.
En bas à gauche, une forme "pilule" bleue aux extrémités parfaitement demi-circulaires.
```

> **Remarque.** `vie.clamp(0.0, 1.0)` protège contre une valeur hors bornes. Sans cela, une vie négative produirait un `Rect` inversé (invisible) et une vie supérieure à 1 déborderait du cadre. Le réflexe du chapitre 13 : anticiper l'entrée invalide.

---

## 21.18 — `drawPath()` et `Path`

Toutes les formes précédentes sont prédéfinies. Pour une forme quelconque — une flèche, une épée, un éclair, un polygone — on utilise un `Path` : une suite d'instructions de tracé.

```dart
void drawPath(Path path, Paint paint)
```

Un `Path` se construit comme un stylo que l'on déplace :

| Méthode | Effet |
| --- | --- |
| `moveTo(x, y)` | lever le stylo et le poser en `(x, y)` |
| `lineTo(x, y)` | tracer une ligne droite jusqu'à `(x, y)` |
| `relativeLineTo(dx, dy)` | idem, mais en coordonnées relatives |
| `quadraticBezierTo(cx, cy, x, y)` | courbe avec un point de contrôle |
| `cubicTo(c1x, c1y, c2x, c2y, x, y)` | courbe avec deux points de contrôle |
| `arcTo(rect, start, sweep, forceMoveTo)` | arc de cercle ou d'ellipse |
| `addRect(rect)`, `addOval(rect)`, `addRRect(rrect)` | ajouter une forme entière |
| `close()` | fermer la figure en revenant au point de départ |

```text
  UN TRIANGLE (pointe vers le haut)

     moveTo(60, 10)  ────>  A
                           ╱ ╲
                          ╱   ╲
   lineTo(110, 90)  ─>  C ─────  B  <─ lineTo(10, 90)
                        close() ferme de C vers A
```

**`close()` est essentiel** : sans lui, un `Path` en mode `fill` est refermé automatiquement pour le remplissage, mais en mode `stroke` le dernier côté manque.

Dessinons trois formes de jeu : une flèche, un éclair et une épée.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const PathApp());

class PathApp extends StatelessWidget {
  const PathApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: const Color(0xFF14161C),
        body: Center(
          child: CustomPaint(
            size: const Size(440, 240),
            painter: PathPainter(),
          ),
        ),
      ),
    );
  }
}

class PathPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
      Offset.zero & size,
      Paint()..color = const Color(0xFF20242E),
    );

    // 1) Un triangle : la pointe d'une flèche.
    final triangle = Path()
      ..moveTo(70, 40)
      ..lineTo(115, 120)
      ..lineTo(25, 120)
      ..close();
    canvas.drawPath(triangle, Paint()..color = const Color(0xFF6FCF6F));

    // Le même en contour, décalé vers le bas.
    final triangle2 = Path()
      ..moveTo(70, 150)
      ..lineTo(115, 220)
      ..lineTo(25, 220)
      ..close();
    canvas.drawPath(
      triangle2,
      Paint()
        ..color = const Color(0xFF6FCF6F)
        ..style = PaintingStyle.stroke
        ..strokeWidth = 4,
    );

    // 2) Un éclair : polygone à six sommets.
    final eclair = Path()
      ..moveTo(200, 30)
      ..lineTo(175, 120)
      ..lineTo(205, 120)
      ..lineTo(185, 210)
      ..lineTo(245, 100)
      ..lineTo(213, 100)
      ..lineTo(238, 30)
      ..close();
    canvas.drawPath(eclair, Paint()..color = const Color(0xFFFFC857));

    // 3) Une courbe : la garde d'une épée, avec quadraticBezierTo.
    final lame = Path()
      ..moveTo(350, 200)
      ..lineTo(350, 50)
      ..lineTo(362, 30)
      ..lineTo(374, 50)
      ..lineTo(374, 200)
      ..close();
    canvas.drawPath(lame, Paint()..color = const Color(0xFFC7CEDB));

    final garde = Path()
      ..moveTo(320, 200)
      ..quadraticBezierTo(362, 175, 404, 200)
      ..quadraticBezierTo(362, 190, 320, 200)
      ..close();
    canvas.drawPath(garde, Paint()..color = const Color(0xFFC79A2E));

    canvas.drawRect(
      Rect.fromLTWH(354, 200, 16, 40),
      Paint()..color = const Color(0xFF8A6A3B),
    );
  }

  @override
  bool shouldRepaint(covariant PathPainter oldDelegate) => false;
}
```

**Résultat :**

```text
Un triangle vert plein, et en dessous le même en contour.
Au centre, un éclair doré à six pointes.
À droite, une épée : lame grise pointue, garde dorée incurvée, poignée brune.
```

> **Optimisation.** Un `Path` est coûteux à construire. Si sa forme ne change pas, créez-le une seule fois dans un champ `final` de la classe plutôt qu'à chaque appel de `paint()` (voir 21.32).

> **Règle de remplissage.** Par défaut `path.fillType` vaut `PathFillType.nonZero`. Avec `PathFillType.evenOdd`, les zones qui se recouvrent se « creusent ». C'est la technique pour faire un anneau ou un trou dans une forme.

---

## 21.19 — `drawPoints()`

```dart
void drawPoints(PointMode pointMode, List<Offset> points, Paint paint)
```

Cette méthode dessine plusieurs points **en un seul appel**, ce qui est bien plus rapide que N appels à `drawCircle`. C'est la méthode des systèmes de particules, des étoiles, des étincelles.

`PointMode` prend trois valeurs :

```text
  PointMode.points     PointMode.lines        PointMode.polygon

   .   .   .   .        ●───●   ●───●          ●───●───●───●
                        1   2   3   4          1   2   3   4

  chaque point est     les points sont       tous les points sont
  dessiné isolément    reliés DEUX PAR       reliés à la suite
                       DEUX (1-2, 3-4)       (1-2, 2-3, 3-4)
```

Deux règles impératives :

1. `PointMode` vient de `dart:ui`. Avec le seul import `package:flutter/material.dart`, il est bien accessible (material réexporte `dart:ui`), mais si vous rencontrez une ambiguïté, ajoutez `import 'dart:ui' as ui;` et écrivez `ui.PointMode.points`.
2. **`style` doit valoir `PaintingStyle.stroke`.** Avec `fill`, `drawPoints` ne dessine rien. C'est le piège classique.

La taille de chaque point est donnée par `strokeWidth`, et sa forme par `strokeCap` (`round` pour des points ronds, `square` pour des carrés).

```dart
import 'dart:math';
import 'package:flutter/material.dart';

void main() => runApp(const PointsApp());

class PointsApp extends StatelessWidget {
  const PointsApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: const Color(0xFF14161C),
        body: Center(
          child: CustomPaint(
            size: const Size(440, 260),
            painter: PointsPainter(),
          ),
        ),
      ),
    );
  }
}

class PointsPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
      Offset.zero & size,
      Paint()..color = const Color(0xFF141824),
    );

    // 1) Un ciel étoilé : 120 points ronds, un seul appel.
    final alea = Random(42); // graine fixe -> même ciel à chaque exécution
    final etoiles = <Offset>[
      for (int i = 0; i < 120; i++)
        Offset(alea.nextDouble() * size.width, alea.nextDouble() * 120),
    ];
    canvas.drawPoints(
      PointMode.points,
      etoiles,
      Paint()
        ..color = const Color(0xFFF0EAD8)
        ..style = PaintingStyle.stroke   // OBLIGATOIRE
        ..strokeWidth = 3
        ..strokeCap = StrokeCap.round,
    );

    // 2) PointMode.lines : segments deux par deux.
    canvas.drawPoints(
      PointMode.lines,
      const [
        Offset(40, 170), Offset(140, 170),
        Offset(180, 170), Offset(280, 170),
      ],
      Paint()
        ..color = const Color(0xFF5AA9E0)
        ..style = PaintingStyle.stroke
        ..strokeWidth = 5,
    );

    // 3) PointMode.polygon : ligne brisée continue.
    canvas.drawPoints(
      PointMode.polygon,
      const [
        Offset(40, 230), Offset(90, 205), Offset(140, 235),
        Offset(190, 200), Offset(240, 230), Offset(290, 210),
      ],
      Paint()
        ..color = const Color(0xFF6FCF6F)
        ..style = PaintingStyle.stroke
        ..strokeWidth = 3,
    );
  }

  @override
  bool shouldRepaint(covariant PointsPainter oldDelegate) => false;
}
```

**Résultat :**

```text
Un ciel constellé de 120 petits points clairs dans la partie haute.
Deux traits bleus séparés au milieu (mode lines).
Une ligne brisée verte continue en bas (mode polygon).
```

> **Pourquoi `Random(42)` ?** Sans graine, le ciel changerait à chaque repeint et scintillerait de façon incontrôlée. Une graine fixe rend le dessin reproductible. C'est une bonne pratique générale du dessin procédural.

---

## 21.20 — Dessiner du texte avec `TextPainter`

Il n'existe **pas** de `canvas.drawText(...)`. Le texte est trop complexe (polices, ligatures, retours à la ligne, écriture de droite à gauche) pour tenir dans une seule méthode. Flutter impose donc un objet intermédiaire : `TextPainter`.

La procédure est toujours la même, en quatre étapes :

```text
  1. créer un TextPainter avec un TextSpan (texte + style)
  2. préciser textDirection (obligatoire, sinon exception)
  3. appeler layout()   -> le texte est mesuré, width/height deviennent lisibles
  4. appeler paint(canvas, offset)   -> dessin au coin HAUT-GAUCHE indiqué
```

`layout()` est obligatoire avant `paint()`. L'oublier lève une assertion. Et l'offset donné à `paint()` est le **coin haut-gauche** du bloc de texte, pas son centre : pour centrer, il faut soustraire la moitié des dimensions mesurées.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const TexteApp());

class TexteApp extends StatelessWidget {
  const TexteApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: const Color(0xFF14161C),
          body: Center(
            child: CustomPaint(
              size: const Size(420, 200),
              painter: TextePainter(score: 1250, vies: 3),
            ),
          ),
        ),
      );
}

class TextePainter extends CustomPainter {
  const TextePainter({required this.score, required this.vies});

  final int score;
  final int vies;

  /// Dessine [texte] centré sur le point [centre].
  void _texteCentre(Canvas canvas, String texte, Offset centre, TextStyle st) {
    final tp = TextPainter(
      text: TextSpan(text: texte, style: st),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, centre - Offset(tp.width / 2, tp.height / 2));
  }

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
        Offset.zero & size, Paint()..color = const Color(0xFF20242E));

    // Texte aligné en haut à gauche : usage direct de paint().
    final hud = TextPainter(
      text: TextSpan(
        text: 'Score : $score\nVies  : $vies',
        style: const TextStyle(
            color: Color(0xFFF0EAD8), fontSize: 18, height: 1.4),
      ),
      textDirection: TextDirection.ltr,
    )..layout();
    hud.paint(canvas, const Offset(16, 16));

    // Titre centré : on utilise les mesures de layout().
    _texteCentre(
      canvas,
      'DONJON DE DART',
      Offset(size.width / 2, size.height - 50),
      const TextStyle(
        color: Color(0xFFFFC857),
        fontSize: 28,
        fontWeight: FontWeight.bold,
        letterSpacing: 2,
      ),
    );
  }

  @override
  bool shouldRepaint(covariant TextePainter old) =>
      old.score != score || old.vies != vies;
}
```

**Résultat :**

```text
En haut à gauche, sur deux lignes :
Score : 1250
Vies  : 3
En bas, centré horizontalement, "DONJON DE DART" en doré, lettres espacées.
```

Quelques réglages utiles du `TextPainter` :

| Paramètre | Rôle |
| --- | --- |
| `textAlign` | alignement interne (`left`, `center`, `right`) |
| `maxLines` | nombre maximal de lignes |
| `ellipsis` | texte de troncature, par exemple `'...'` |
| `textScaler` | facteur d'échelle d'accessibilité |
| `layout(maxWidth: 200)` | largeur maximale, au-delà le texte passe à la ligne |

> **Coût.** Mesurer du texte est **cher**. Ne recréez pas un `TextPainter` à chaque frame pour un texte qui ne change pas : gardez-le dans un champ et n'appelez `layout()` que si le contenu a changé.

---

## 21.21 — Les couleurs : `Color`, `Colors`, `withOpacity`, ARGB

Une couleur Flutter est un entier 32 bits, découpé en quatre octets : **A**lpha, **R**ouge, **V**ert, **B**leu.

```text
   0xFF  E0  5A  78
     │    │   │   │
     │    │   │   └── Bleu   : 0x78 = 120
     │    │   └────── Vert   : 0x5A =  90
     │    └────────── Rouge  : 0xE0 = 224
     └─────────────── Alpha  : 0xFF = 255 (totalement opaque)

   Chaque composante va de 0x00 (0) à 0xFF (255).
   Alpha 0x00 = invisible, 0x80 = à moitié transparent, 0xFF = opaque.
```

Les trois façons d'écrire une couleur :

```dart
const rouge1 = Color(0xFFE05A78);              // hexadécimal ARGB
const rouge2 = Color.fromARGB(255, 224, 90, 120);  // composantes 0-255
const rouge3 = Color.fromRGBO(224, 90, 120, 1.0);  // opacité en 0.0-1.0
// rouge1 == rouge2 == rouge3
```

**L'oubli le plus fréquent est celui des deux `FF` de l'alpha.** `Color(0xE05A78)` n'est pas rouge : Dart complète à gauche par des zéros, l'alpha vaut `0x00`, et la couleur est totalement invisible. Écrivez toujours les huit chiffres.

La classe `Colors` (avec un `s`) fournit la palette Material : `Colors.red`, `Colors.blue.shade700`, `Colors.transparent`, `Colors.black54`. Pratique pour prototyper, mais pour un jeu on définit sa propre palette en constantes.

Pour rendre une couleur translucide :

```dart
const base = Color(0xFF6FCF6F);
final fantome = base.withValues(alpha: 0.35);  // API moderne
final ancien  = base.withOpacity(0.35);        // API historique, dépréciée
```

`withOpacity(x)` est la méthode que vous croiserez dans tous les tutoriels antérieurs à 2024. Depuis Flutter 3.27, elle est dépréciée au profit de `withValues(alpha: x)`, plus précis (il évite un arrondi sur 8 bits). Les deux donnent le même rendu visuel ; préférez `withValues` dans du code neuf.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const CouleurApp());

class CouleurApp extends StatelessWidget {
  const CouleurApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: const Color(0xFF14161C),
          body: Center(
            child: CustomPaint(
              size: const Size(420, 200),
              painter: CouleurPainter(),
            ),
          ),
        ),
      );
}

class CouleurPainter extends CustomPainter {
  static const Color potion = Color(0xFFE05A78);

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
        Offset.zero & size, Paint()..color = const Color(0xFF20242E));

    // Dégradé d'opacité par pas de 20 %.
    for (int i = 0; i < 5; i++) {
      final alpha = (i + 1) / 5;
      canvas.drawRect(
        Rect.fromLTWH(30.0 + i * 74, 40, 64, 60),
        Paint()..color = potion.withValues(alpha: alpha),
      );
    }

    // Superposition : trois disques translucides.
    const a = Color(0x99E05A78);
    const b = Color(0x996FCF6F);
    const c = Color(0x995AA9E0);
    canvas.drawCircle(const Offset(180, 150), 38, Paint()..color = a);
    canvas.drawCircle(const Offset(215, 150), 38, Paint()..color = b);
    canvas.drawCircle(const Offset(197, 175), 38, Paint()..color = c);
  }

  @override
  bool shouldRepaint(covariant CouleurPainter old) => false;
}
```

**Résultat :**

```text
Cinq carrés rouges du plus pâle au plus vif, de gauche à droite.
En dessous, trois disques translucides qui se recouvrent : les zones communes
prennent des teintes mélangées.
```

Une palette de donjon utilisable telle quelle dans tout le reste de la formation :

| Rôle | Constante |
| --- | --- |
| Fond de l'écran | `Color(0xFF14161C)` |
| Sol de la salle | `Color(0xFF2A2F3C)` |
| Mur | `Color(0xFF454E63)` |
| Héros | `Color(0xFF6FCF6F)` |
| Gobelin | `Color(0xFF8FBF5A)` |
| Potion / dégâts | `Color(0xFFE05A78)` |
| Or / clé | `Color(0xFFFFC857)` |
| Texte | `Color(0xFFF0EAD8)` |

---

## 21.22 — Les dégradés (`Gradient.linear`, `Gradient.radial`)

Un dégradé n'est pas une couleur, c'est un **shader** : un programme qui calcule une couleur différente pour chaque pixel. On l'installe dans le `Paint` via la propriété `shader`, et il remplace alors `color`.

**Attention à un piège d'import.** Il existe deux classes nommées `Gradient` :

- `Gradient` de `dart:ui` : celle qui a les constructeurs `linear` et `radial`, utilisable dans un `Paint` ;
- `Gradient` de `package:flutter/painting.dart` (réexportée par `material.dart`) : la classe abstraite des `LinearGradient` / `RadialGradient` utilisés par les widgets.

Comme `material.dart` importe la seconde, écrire `Gradient.linear(...)` provoque une erreur de compilation. La solution est un import préfixé :

```dart
import 'dart:ui' as ui;
...
paint.shader = ui.Gradient.linear(debut, fin, couleurs);
```

Les deux constructeurs :

```dart
ui.Gradient.linear(Offset from, Offset to, List<Color> colors, [List<double>? stops])
ui.Gradient.radial(Offset center, double radius, List<Color> colors, [List<double>? stops])
```

```text
  LINÉAIRE                              RADIAL

  from                    to                    ┌─────────────┐
   ●─────────────────────►●                     │   ,-----.   │
                                                │  /       \  │
  ████▓▓▓▓▒▒▒▒░░░░                              │ |    ●    | │  center
  couleur[0] ... couleur[n]                     │  \       /  │
                                                │   `-----'   │
  La couleur varie le long                      └─────────────┘
  de l'axe from -> to.                     La couleur varie du centre
                                           vers le bord (rayon).
```

Les `stops` sont facultatifs : ce sont les positions (entre `0.0` et `1.0`) de chaque couleur. Sans eux, les couleurs sont réparties uniformément. **Si vous les fournissez, la liste doit avoir exactement la même longueur que celle des couleurs**, sinon une assertion échoue.

```dart
import 'dart:ui' as ui;
import 'package:flutter/material.dart';

void main() => runApp(const DegradeApp());

class DegradeApp extends StatelessWidget {
  const DegradeApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: const Color(0xFF14161C),
          body: Center(
            child: CustomPaint(
              size: const Size(420, 260),
              painter: DegradePainter(),
            ),
          ),
        ),
      );
}

class DegradePainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    // 1) Ciel de donjon : dégradé linéaire vertical.
    final ciel = Rect.fromLTWH(0, 0, size.width, size.height);
    canvas.drawRect(
      ciel,
      Paint()
        ..shader = ui.Gradient.linear(
          ciel.topCenter,
          ciel.bottomCenter,
          const [Color(0xFF1B2340), Color(0xFF3B2246), Color(0xFF14161C)],
          const [0.0, 0.55, 1.0],
        ),
    );

    // 2) Une torche : dégradé radial du blanc chaud vers le transparent.
    const centreTorche = Offset(110, 130);
    canvas.drawCircle(
      centreTorche,
      80,
      Paint()
        ..shader = ui.Gradient.radial(
          centreTorche,
          80,
          const [Color(0xFFFFF0B0), Color(0x66FFC857), Color(0x00FFC857)],
          const [0.0, 0.4, 1.0],
        ),
    );
    canvas.drawCircle(
        centreTorche, 10, Paint()..color = const Color(0xFFFFF6D8));

    // 3) Une barre d'énergie avec dégradé horizontal.
    final barre = Rect.fromLTWH(240, 100, 150, 26);
    canvas.drawRRect(
      RRect.fromRectAndRadius(barre, const Radius.circular(13)),
      Paint()
        ..shader = ui.Gradient.linear(
          barre.centerLeft,
          barre.centerRight,
          const [Color(0xFF5AA9E0), Color(0xFF6FCF6F)],
        ),
    );
  }

  @override
  bool shouldRepaint(covariant DegradePainter old) => false;
}
```

**Résultat :**

```text
Le fond passe du bleu nuit au violet puis au presque noir, de haut en bas.
À gauche, un halo lumineux qui s'estompe progressivement, avec un coeur blanc.
À droite, une barre arrondie qui passe du bleu au vert.
```

> **Performance.** Créer un shader est coûteux. Si le dégradé est fixe, construisez le `Paint` une fois pour toutes. Mais attention : un shader dépendant de `size` doit être recréé si `size` change.

---

## 21.23 — L'ordre de dessin : l'algorithme du peintre

Le `Canvas` ne connaît ni profondeur ni calque. Il applique la règle la plus simple qui soit, appelée **algorithme du peintre** :

> Ce qui est dessiné **en dernier** est **par-dessus**.

Exactement comme un peintre qui pose d'abord le ciel, puis les montagnes, puis les arbres, puis le personnage.

```text
  ORDRE DES APPELS                      RENDU FINAL

  1. drawRect(fond)          ░░░░░░░░░░░░░░░░░░░░░░
  2. drawRect(sol)           ░░░░░░░░░░░░░░░░░░░░░░
  3. drawOval(ombre)         ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
  4. drawRect(heros)         ▒▒▒▒▒▒▒▒░████░▒▒▒▒▒▒▒▒
  5. drawText(score)         ▒▒Score▒░████░▒▒▒▒▒▒▒▒
                                 ^         ^
                          dessiné en 5   dessiné en 4
                          donc DEVANT    donc devant l'ombre
```

D'où l'ordre canonique d'une frame de jeu 2D :

```text
  1. fond / ciel
  2. décor lointain (parallaxe, chapitre 25)
  3. sol et murs
  4. objets au sol (pièces, potions, clés)
  5. ombres portées
  6. entités (ennemis, puis joueur)
  7. effets (particules, éclairs)
  8. HUD (score, barre de vie)
```

Il n'existe pas de `z-index` dans un `Canvas`. Si vous voulez trier vos entités par profondeur, vous devez **trier votre liste avant de dessiner**. C'est un usage direct de `sort` vu au chapitre 14 :

```dart
entites.sort((a, b) => a.y.compareTo(b.y));  // les plus hauts d'abord
for (final e in entites) { e.dessiner(canvas); }
```

Cette technique — trier par `y` croissant — donne gratuitement l'effet de perspective des jeux vus de dessus : un personnage plus bas à l'écran est plus proche, donc devant.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const OrdreApp());

class OrdreApp extends StatelessWidget {
  const OrdreApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: const Color(0xFF14161C),
          body: Center(
            child: CustomPaint(
              size: const Size(420, 220),
              painter: OrdrePainter(),
            ),
          ),
        ),
      );
}

class OrdrePainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
        Offset.zero & size, Paint()..color = const Color(0xFF2A2F3C));

    // Trois entités, décrites par leur y (profondeur).
    final entites = <(double, double, Color)>[
      (150, 90, Color(0xFF8FBF5A)),   // gobelin, plus haut donc derrière
      (110, 130, Color(0xFF6FCF6F)),  // héros
      (200, 110, Color(0xFFE05A78)),  // gobelin rouge
    ];

    // On trie par y croissant : les plus hauts sont dessinés en premier.
    entites.sort((a, b) => a.$2.compareTo(b.$2));

    for (final (x, y, couleur) in entites) {
      canvas.drawOval(
        Rect.fromCenter(center: Offset(x, y + 32), width: 56, height: 16),
        Paint()..color = const Color(0x55000000),
      );
      canvas.drawRect(
        Rect.fromCenter(center: Offset(x, y), width: 46, height: 64),
        Paint()..color = couleur,
      );
    }
  }

  @override
  bool shouldRepaint(covariant OrdrePainter old) => false;
}
```

**Résultat :**

```text
Trois rectangles colorés qui se chevauchent, chacun posé sur son ombre.
Celui dont le y est le plus grand (le vert clair, le plus bas) recouvre les autres :
l'illusion de profondeur fonctionne.
```

> **Note Dart.** `(double, double, Color)` est un **record**, disponible depuis Dart 3. `a.$2` accède au deuxième champ. La déstructuration `for (final (x, y, couleur) in ...)` évite de créer une classe pour trois valeurs.

---

## 21.24 — `save()` et `restore()`

Le `Canvas` possède un **état courant** : la matrice de transformation (translation, rotation, échelle) et la zone de découpe (clip). Toutes les méthodes de transformation modifient cet état **de façon cumulative et définitive**.

D'où le besoin de pouvoir revenir en arrière. C'est le rôle de `save()` et `restore()`, qui fonctionnent comme une **pile** :

```text
  canvas.save();          empile une copie de l'état courant
     ... transformations et dessins ...
  canvas.restore();       dépile : l'état revient exactement comme avant


  PILE D'ÉTATS

  état initial   ->  save()  ->  translate(100,0)  ->  restore()

  ┌──────────┐      ┌──────────┐   ┌──────────┐        ┌──────────┐
  │ identité │      │ identité │   │ identité │        │ identité │
  └──────────┘      ├──────────┤   ├──────────┤        └──────────┘
                    │ identité │   │ T(100,0) │
                    └──────────┘   └──────────┘
                     (copie)        (modifiée)
```

La règle absolue :

> **Chaque `save()` doit avoir son `restore()`.** Toujours. Sans exception.

Un `restore()` oublié est un bug pernicieux : la première frame semble correcte, la deuxième est décalée, la troisième deux fois plus, et l'écran part en vrille. Comme `paint()` est appelé 60 fois par seconde, une petite fuite devient énorme en une seconde.

À l'inverse, un `restore()` **en trop** (sans `save()` correspondant) lève une assertion en mode debug.

Le motif à adopter systématiquement :

```dart
canvas.save();
canvas.translate(x, y);
canvas.rotate(angle);
// ... on dessine dans le repère local ...
canvas.restore();   // toujours, même si le dessin échoue
```

Il existe deux aides :

| Méthode | Rôle |
| --- | --- |
| `canvas.getSaveCount()` | profondeur actuelle de la pile (vaut 1 au départ) |
| `canvas.restoreToCount(n)` | dépile jusqu'à revenir à la profondeur `n` |

`restoreToCount` est le filet de sécurité : on note la profondeur au début de `paint()`, et on y revient à la fin.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const SaveApp());

class SaveApp extends StatelessWidget {
  const SaveApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: const Color(0xFF14161C),
          body: Center(
            child: CustomPaint(
              size: const Size(420, 200),
              painter: SavePainter(),
            ),
          ),
        ),
      );
}

class SavePainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    final depart = canvas.getSaveCount(); // vaut 1

    canvas.drawRect(
        Offset.zero & size, Paint()..color = const Color(0xFF20242E));

    final rouge = Paint()..color = const Color(0xFFE05A78);
    final vert = Paint()..color = const Color(0xFF6FCF6F);
    const carre = Rect.fromLTWH(0, 0, 50, 50);

    // Bloc 1 : isolé par save/restore.
    canvas.save();
    canvas.translate(40, 40);
    canvas.drawRect(carre, rouge);
    canvas.restore();

    // Bloc 2 : isolé lui aussi. Il repart bien de l'origine.
    canvas.save();
    canvas.translate(140, 40);
    canvas.drawRect(carre, vert);
    canvas.restore();

    // Preuve : ce carré est dessiné à l'origine réelle, non décalé.
    canvas.drawRect(
      const Rect.fromLTWH(300, 40, 50, 50),
      Paint()..color = const Color(0xFFFFC857),
    );

    // Filet de sécurité : on garantit l'équilibre de la pile.
    canvas.restoreToCount(depart);
  }

  @override
  bool shouldRepaint(covariant SavePainter old) => false;
}
```

**Résultat :**

```text
Trois carrés de 50 px alignés horizontalement à y = 40 :
rouge à x = 40, vert à x = 140, jaune à x = 300.
Si l'on retirait les restore(), le vert serait à x = 180 et le jaune à x = 480
(donc hors de la zone).
```

---

## 21.25 — `translate()`

```dart
void translate(double dx, double dy)
```

`translate` déplace **l'origine du repère**. Ce n'est pas un déplacement de forme : c'est un déplacement du système de coordonnées lui-même.

```text
  AVANT translate(100, 50)          APRÈS translate(100, 50)

  (0,0)                              (0,0) réel
   +──────────────> X                 +──────────────> X
   │                                  │
   │                                  │      (0,0) du repère courant
   v                                  │       +──────────> X
   Y                                  │       │
                                      v       v
  drawRect(0,0,40,40) dessine        drawRect(0,0,40,40) dessine
  au coin haut-gauche.               à (100, 50) à l'écran.
```

C'est **la** technique pour dessiner une entité de jeu : au lieu d'ajouter la position à toutes les coordonnées, on translate une fois, puis on dessine la forme « à l'origine ».

```text
  SANS translate                        AVEC translate

  drawRect(x + 0,  y + 0,  40, 40)      save();
  drawCircle(x + 10, y + 12, 5)         translate(x, y);
  drawCircle(x + 30, y + 12, 5)         drawRect(0, 0, 40, 40);
  drawLine(x+10, y+28, x+30, y+28)      drawCircle(10, 12, 5);
                                        drawCircle(30, 12, 5);
  Fatiguant, source d'erreurs.          drawLine(...);
                                        restore();

                                        La forme est décrite UNE FOIS,
                                        dans son repère à elle.
```

Cette séparation « forme dans son repère local / position via `translate` » est exactement ce que Flame automatise avec les `PositionComponent` du chapitre 28. Vous en construisez ici la version manuelle.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const TranslateApp());

class TranslateApp extends StatelessWidget {
  const TranslateApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: const Color(0xFF14161C),
          body: Center(
            child: CustomPaint(
              size: const Size(420, 200),
              painter: TranslatePainter(),
            ),
          ),
        ),
      );
}

class TranslatePainter extends CustomPainter {
  /// Dessine un gobelin dont le coin haut-gauche est à l'origine locale.
  void _gobelin(Canvas canvas) {
    canvas.drawRect(
        const Rect.fromLTWH(0, 0, 44, 44), Paint()..color = const Color(0xFF8FBF5A));
    canvas.drawCircle(const Offset(13, 16), 5, Paint()..color = Colors.black);
    canvas.drawCircle(const Offset(31, 16), 5, Paint()..color = Colors.black);
    canvas.drawRect(const Rect.fromLTWH(12, 30, 20, 4),
        Paint()..color = const Color(0xFF3A2530));
  }

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
        Offset.zero & size, Paint()..color = const Color(0xFF20242E));

    // Le MÊME dessin, posé à trois endroits différents.
    for (final p in const [Offset(40, 40), Offset(180, 90), Offset(320, 40)]) {
      canvas.save();
      canvas.translate(p.dx, p.dy);
      _gobelin(canvas);
      canvas.restore();
    }
  }

  @override
  bool shouldRepaint(covariant TranslatePainter old) => false;
}
```

**Résultat :**

```text
Trois gobelins identiques (carré vert, deux yeux noirs, une bouche sombre)
placés en haut à gauche, au centre plus bas, et en haut à droite.
La méthode _gobelin ne connaît aucune position : elle dessine toujours à (0, 0).
```

---

## 21.26 — `rotate()` : attention, des radians

```dart
void rotate(double radians)
```

`rotate` fait tourner le repère **autour de son origine courante**, dans le sens des aiguilles d'une montre (parce que `Y` est vers le bas).

Deux pièges, et ce sont les deux erreurs les plus fréquentes de tout le chapitre.

### Piège 1 : l'argument est en radians, pas en degrés

`canvas.rotate(45)` ne tourne pas de 45 degrés, mais de 45 **radians**, soit environ 2578 degrés, c'est-à-dire un peu plus de sept tours complets. Le résultat est totalement imprévisible.

```text
   degrés  ->  radians  :  radians = degres * pi / 180

        0°  =  0
       45°  =  pi / 4   ≈ 0.785
       90°  =  pi / 2   ≈ 1.571
      180°  =  pi       ≈ 3.142
      360°  =  2 * pi   ≈ 6.283
```

Écrivez une fonction utilitaire et n'y pensez plus :

```dart
double rad(double degres) => degres * pi / 180;
```

(avec `import 'dart:math';` pour `pi`)

### Piège 2 : la rotation se fait autour de l'ORIGINE, pas du centre de la forme

```text
  canvas.rotate(rad(30));                  translate PUIS rotate
  drawRect(100, 60, 60, 60)                  au centre de la forme

  origine
    +                                          +
     ╲                                         │
      ╲   la forme part                        │      ╱▔▔╲
       ╲  en arc de cercle                     │     │ ▁▁ │  tourne sur place
        ╲    ╱▔╲                               │      ╲__╱
         ╲  │  │                               │
          ╲  ▔▔                                v

  Ce n'est presque jamais ce que l'on veut.   Voilà ce que l'on veut.
```

La recette correcte, à mémoriser :

```dart
canvas.save();
canvas.translate(centreX, centreY);   // 1. amener l'origine sur le centre
canvas.rotate(angle);                 // 2. tourner autour de ce centre
canvas.drawRect(                      // 3. dessiner CENTRÉ sur l'origine
  Rect.fromCenter(center: Offset.zero, width: w, height: h),
  paint,
);
canvas.restore();
```

```dart
import 'dart:math';
import 'package:flutter/material.dart';

void main() => runApp(const RotateApp());

class RotateApp extends StatelessWidget {
  const RotateApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: const Color(0xFF14161C),
          body: Center(
            child: CustomPaint(
              size: const Size(420, 220),
              painter: RotatePainter(),
            ),
          ),
        ),
      );
}

double rad(double degres) => degres * pi / 180;

class RotatePainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
        Offset.zero & size, Paint()..color = const Color(0xFF20242E));

    // MAUVAIS : rotation autour de l'origine du canvas.
    canvas.save();
    canvas.rotate(rad(20));
    canvas.drawRect(
      const Rect.fromLTWH(90, 40, 70, 70),
      Paint()..color = const Color(0x88E05A78),
    );
    canvas.restore();

    // BON : translate au centre, puis rotation sur place.
    const centre = Offset(300, 110);
    canvas.save();
    canvas.translate(centre.dx, centre.dy);
    canvas.rotate(rad(20));
    canvas.drawRect(
      Rect.fromCenter(center: Offset.zero, width: 70, height: 70),
      Paint()..color = const Color(0xFF6FCF6F),
    );
    canvas.restore();

    // Repère du centre visé, pour comparer.
    canvas.drawCircle(centre, 3, Paint()..color = const Color(0xFFFFC857));
  }

  @override
  bool shouldRepaint(covariant RotatePainter old) => false;
}
```

**Résultat :**

```text
À gauche, un carré rouge translucide incliné, mais nettement déporté vers
le bas-droite par rapport à l'endroit prévu : la rotation l'a fait glisser
le long d'un arc centré sur le coin haut-gauche du canvas.
À droite, un carré vert incliné de 20 degrés autour de son propre centre,
sur lequel un point jaune est parfaitement superposé.
```

> **Sens de rotation.** Un angle positif tourne dans le sens des aiguilles d'une montre à l'écran. C'est la conséquence directe de l'axe `Y` vers le bas : le repère est indirect.

---

## 21.27 — `scale()`

```dart
void scale(double sx, [double? sy])
```

`scale` multiplie les unités du repère. `scale(2)` double tout : les positions **et** les tailles. `scale(2, 1)` étire seulement horizontalement.

Trois usages en jeu :

| Appel | Effet |
| --- | --- |
| `scale(2)` | zoom x2 (positions et dimensions doublées) |
| `scale(0.5)` | dézoom |
| `scale(-1, 1)` | **miroir horizontal** : le personnage regarde à gauche |
| `scale(1, -1)` | miroir vertical |

Le miroir est l'usage le plus précieux : il vous évite de dessiner deux jeux de sprites pour un personnage qui marche dans les deux sens (chapitre 22).

```text
  scale(-1, 1) : l'axe X est inversé

  AVANT                        APRÈS
  origine                              origine
    +────────> X                  X <────────+
    │                                        │
    │   ▶ (le héros regarde                  │   ◀ (il regarde à gauche)
    │      à droite)                         │
    v                                        v

  Attention : tout ce que vous dessinez ensuite à x positif
  part vers la GAUCHE. D'où la nécessité de translate d'abord.
```

Point critique : `scale` multiplie aussi le `strokeWidth`. Un contour de 2 pixels dessiné dans un repère à `scale(3)` fera 6 pixels à l'écran. Si vous voulez une épaisseur constante quel que soit le zoom, divisez : `strokeWidth = 2 / zoom`.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const ScaleApp());

class ScaleApp extends StatelessWidget {
  const ScaleApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: const Color(0xFF14161C),
          body: Center(
            child: CustomPaint(
              size: const Size(440, 200),
              painter: ScalePainter(),
            ),
          ),
        ),
      );
}

class ScalePainter extends CustomPainter {
  /// Un héros asymétrique : le nez pointe vers +X.
  void _heros(Canvas canvas) {
    canvas.drawRect(const Rect.fromLTWH(-15, -25, 30, 50),
        Paint()..color = const Color(0xFF6FCF6F));
    canvas.drawRect(const Rect.fromLTWH(15, -12, 14, 6),
        Paint()..color = const Color(0xFFFFC857)); // le nez, vers la droite
  }

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
        Offset.zero & size, Paint()..color = const Color(0xFF20242E));

    // Taille normale.
    canvas.save();
    canvas.translate(70, 100);
    _heros(canvas);
    canvas.restore();

    // Deux fois plus grand.
    canvas.save();
    canvas.translate(180, 100);
    canvas.scale(2);
    _heros(canvas);
    canvas.restore();

    // Miroir horizontal : le nez pointe vers la gauche.
    canvas.save();
    canvas.translate(320, 100);
    canvas.scale(-1, 1);
    _heros(canvas);
    canvas.restore();

    // Étirement vertical seul.
    canvas.save();
    canvas.translate(400, 100);
    canvas.scale(1, 1.6);
    _heros(canvas);
    canvas.restore();
  }

  @override
  bool shouldRepaint(covariant ScalePainter old) => false;
}
```

**Résultat :**

```text
Quatre héros : le premier normal (nez à droite), le deuxième deux fois plus
grand, le troisième retourné (nez à gauche), le quatrième étiré en hauteur.
```

> **Piège.** `scale(0)` écrase tout en un point : plus rien n'est visible et la matrice n'est plus inversible. Cela arrive quand on anime une échelle qui passe par zéro. Bornez toujours : `echelle.clamp(0.01, 10.0)`.

---

## 21.28 — Combiner les transformations : l'ordre compte

Les transformations se **cumulent**, et elles ne sont **pas commutatives**. `translate` puis `rotate` ne donne pas le même résultat que `rotate` puis `translate`.

La clé pour ne plus se tromper : **chaque transformation s'applique au repère produit par les précédentes**. Lisez toujours vos appels de haut en bas comme une suite de déplacements d'un repère physique.

```text
  CAS A : translate(200, 100) PUIS rotate(45°)

  1) l'origine va en (200,100)      2) le repère tourne SUR PLACE
     +─────────────>                     +
     │                                    ╲
     │        +─────>                      ╲
     │        │                             ╲     drawRect(0,0,60,60)
     v        v                              v     est un carré incliné
                                                   AUTOUR de (200,100)


  CAS B : rotate(45°) PUIS translate(200, 100)

  1) le repère tourne d'abord        2) on avance de 200 le long de l'axe X
     +                                  DÉJÀ TOURNÉ
      ╲                                   +
       ╲                                   ╲
        ╲                                   ╲──────────► nouvelle origine
         v                                            (≈ 141, 212) à l'écran

  Le carré est incliné pareil, mais il n'est PAS au même endroit.
```

Concrètement : `translate` puis `rotate` fait tourner l'objet **sur lui-même** à sa position. `rotate` puis `translate` fait tourner l'objet **autour du point de départ**, comme une lune autour d'une planète.

Ce deuxième cas n'est pas une erreur : c'est exactement la technique pour faire orbiter un satellite, ou pour disposer des objets en cercle.

```dart
import 'dart:math';
import 'package:flutter/material.dart';

void main() => runApp(const CombineApp());

class CombineApp extends StatelessWidget {
  const CombineApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: const Color(0xFF14161C),
          body: Center(
            child: CustomPaint(
              size: const Size(440, 260),
              painter: CombinePainter(),
            ),
          ),
        ),
      );
}

class CombinePainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
        Offset.zero & size, Paint()..color = const Color(0xFF20242E));

    const carre = Rect.fromLTWH(0, 0, 50, 50);

    // CAS A : translate PUIS rotate -> tourne sur place.
    canvas.save();
    canvas.translate(100, 70);
    canvas.rotate(pi / 4);
    canvas.drawRect(carre, Paint()..color = const Color(0xFF6FCF6F));
    canvas.restore();
    canvas.drawCircle(
        const Offset(100, 70), 3, Paint()..color = const Color(0xFFFFC857));

    // CAS B : rotate PUIS translate -> orbite autour de l'origine.
    canvas.save();
    canvas.rotate(pi / 4);
    canvas.translate(100, 70);
    canvas.drawRect(carre, Paint()..color = const Color(0xFFE05A78));
    canvas.restore();

    // Application : 8 clés disposées en cercle autour d'un coffre.
    const centre = Offset(320, 130);
    canvas.drawCircle(centre, 12, Paint()..color = const Color(0xFF8A6A3B));
    for (int i = 0; i < 8; i++) {
      canvas.save();
      canvas.translate(centre.dx, centre.dy); // 1. aller au centre
      canvas.rotate(i * 2 * pi / 8);          // 2. tourner d'un huitième
      canvas.translate(70, 0);                // 3. s'éloigner sur l'axe tourné
      canvas.drawRect(
        Rect.fromCenter(center: Offset.zero, width: 22, height: 10),
        Paint()..color = const Color(0xFFFFC857),
      );
      canvas.restore();
    }
  }

  @override
  bool shouldRepaint(covariant CombinePainter old) => false;
}
```

**Résultat :**

```text
À gauche, un carré vert en losange dont le COIN haut-gauche est sur le point jaune
(translate puis rotate : la rotation a eu lieu à cet endroit).
Un carré rouge ailleurs dans la zone : même inclinaison, position différente
(rotate puis translate).
À droite, huit petites clés dorées disposées en couronne autour d'un disque brun,
chacune orientée dans sa direction.
```

> **Formule à retenir.** Pour placer un objet à un angle `a` et une distance `d` autour d'un centre : `translate(centre)` puis `rotate(a)` puis `translate(d, 0)`. Trois lignes, aucune trigonométrie à écrire soi-même.

---

## 21.29 — Dessiner autour d'un centre plutôt qu'un coin : la notion d'ancre

L'**ancre** (anchor) d'une forme, c'est le point de la forme qui coïncide avec sa position déclarée.

```text
  ANCRE = topLeft (par défaut)        ANCRE = center

  position (100, 60)                  position (100, 60)
      ●────────────┐                      ┌──────────────┐
      │            │                      │              │
      │  la forme  │                      │       ●      │
      │            │                      │              │
      └────────────┘                      └──────────────┘

  Rect.fromLTWH(100, 60, w, h)        Rect.fromCenter(
                                        center: Offset(100, 60),
                                        width: w, height: h)
```

Pourquoi préférer le centre pour une entité de jeu ? Trois raisons décisives :

1. **La rotation.** Un objet tourne naturellement autour de son centre (section 21.26).
2. **La mise à l'échelle.** Grossir un objet ancré au coin le fait glisser vers le bas-droite ; ancré au centre, il grossit sur place.
3. **La distance.** Pour savoir si deux entités se touchent (chapitre 24), on compare des distances entre **centres**. Avec des coins, il faut ajouter la moitié des tailles partout.

La règle de conversion est immédiate :

```text
  coin   = centre - Offset(largeur / 2, hauteur / 2)
  centre = coin   + Offset(largeur / 2, hauteur / 2)
```

Il existe une exception importante : les **tuiles de décor**. Une grille se raisonne naturellement en coins (`colonne * taille`, `ligne * taille`). On garde donc `topLeft` pour la tilemap, et `center` pour les entités mobiles. C'est exactement la convention retenue par Flame, dont `PositionComponent` possède une propriété `anchor` réglable (`Anchor.topLeft`, `Anchor.center`, `Anchor.bottomCenter`...).

Un quatrième cas mérite d'être connu : l'ancre **`bottomCenter`**, aux pieds du personnage. C'est celle qui simplifie la gravité et le tri par profondeur, puisque « où le personnage touche le sol » est l'information réellement utile.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AncreApp());

class AncreApp extends StatelessWidget {
  const AncreApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: const Color(0xFF14161C),
          body: Center(
            child: CustomPaint(
                size: const Size(420, 220), painter: AncrePainter()),
          ),
        ),
      );
}

class AncrePainter extends CustomPainter {
  void _repere(Canvas c, Offset p) =>
      c.drawCircle(p, 4, Paint()..color = const Color(0xFFFFC857));

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
        Offset.zero & size, Paint()..color = const Color(0xFF20242E));
    const w = 60.0, h = 80.0;
    final vert = Paint()..color = const Color(0xFF6FCF6F);

    // Ancre topLeft.
    const pA = Offset(60, 60);
    canvas.drawRect(Rect.fromLTWH(pA.dx, pA.dy, w, h), vert);
    _repere(canvas, pA);

    // Ancre center.
    const pB = Offset(210, 100);
    canvas.drawRect(
        Rect.fromCenter(center: pB, width: w, height: h), vert);
    _repere(canvas, pB);

    // Ancre bottomCenter : les pieds sur le point.
    const pC = Offset(350, 160);
    canvas.drawRect(
        Rect.fromLTWH(pC.dx - w / 2, pC.dy - h, w, h), vert);
    _repere(canvas, pC);
    canvas.drawLine(Offset(280, pC.dy), Offset(420, pC.dy),
        Paint()..color = const Color(0xFF454E63)..strokeWidth = 2);
  }

  @override
  bool shouldRepaint(covariant AncrePainter old) => false;
}
```

**Résultat :**

```text
Trois rectangles verts identiques. Le point jaune est sur le coin du premier,
au centre du deuxième, et sous les pieds du troisième, exactement sur la ligne
de sol.
```

---

## 21.30 — `clipRect()`

```dart
void clipRect(Rect rect, {ClipOp clipOp = ClipOp.intersect, bool doAntiAlias = true})
```

`clipRect` restreint la zone dans laquelle les dessins suivants seront visibles. Tout ce qui déborde est simplement ignoré.

```text
  SANS clip                          AVEC clipRect(fenetre)

  ┌──────────────────┐               ┌──────────────────┐
  │ ██████████████   │               │ ┌────────┐       │
  │ ██████████████   │               │ │████████│       │
  │ ██████████████   │               │ │████████│       │
  │ ██████████████   │               │ └────────┘       │
  └──────────────────┘               └──────────────────┘
  la forme déborde                   seule la partie dans la
  partout                            fenêtre est peinte
```

Les usages en jeu sont nombreux :

- une **minicarte** dans un coin de l'écran ;
- une **fenêtre de dialogue** dont le texte défile ;
- un **viewport** de caméra (chapitre 25) ;
- une **barre de progression** dont on masque la partie non remplie.

Le clip fait partie de l'état du canvas : il est donc annulé par `restore()`, et **c'est la seule façon de l'annuler**. Il n'existe pas de `unclip()`.

Variantes disponibles : `clipRRect(RRect)` pour une fenêtre à coins arrondis, `clipPath(Path)` pour une forme quelconque. `ClipOp.difference` inverse la logique : on masque l'intérieur du rectangle au lieu de l'extérieur (utile pour un effet de « trou »).

```dart
import 'package:flutter/material.dart';

void main() => runApp(const ClipApp());

class ClipApp extends StatelessWidget {
  const ClipApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: const Color(0xFF14161C),
          body: Center(
            child:
                CustomPaint(size: const Size(420, 230), painter: ClipPainter()),
          ),
        ),
      );
}

class ClipPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
        Offset.zero & size, Paint()..color = const Color(0xFF20242E));

    // Une minicarte : fenêtre carrée de 150 px.
    final fenetre = Rect.fromLTWH(20, 30, 150, 150);
    canvas.drawRect(fenetre, Paint()..color = const Color(0xFF11141B));

    canvas.save();
    canvas.clipRect(fenetre); // tout ce qui suit est limité à la fenêtre
    for (int l = -2; l < 8; l++) {
      for (int c = -2; c < 8; c++) {
        if ((l + c) % 3 == 0) continue;
        canvas.drawRect(
          Rect.fromLTWH(20 + c * 34.0, 30 + l * 34.0, 30, 30),
          Paint()..color = const Color(0xFF3C4457),
        );
      }
    }
    canvas.drawCircle(
        const Offset(95, 105), 9, Paint()..color = const Color(0xFF6FCF6F));
    canvas.restore(); // le clip disparaît ici

    // Preuve : ce disque déborde librement, hors clip.
    canvas.drawCircle(
        const Offset(300, 120), 70, Paint()..color = const Color(0x88E05A78));
    canvas.drawRect(
      fenetre,
      Paint()
        ..color = const Color(0xFFF0EAD8)
        ..style = PaintingStyle.stroke
        ..strokeWidth = 2,
    );
  }

  @override
  bool shouldRepaint(covariant ClipPainter old) => false;
}
```

**Résultat :**

```text
À gauche, une minicarte carrée cernée de blanc : les tuiles grises sont coupées
net aux quatre bords, un point vert marque le héros.
À droite, un grand disque rouge translucide, dessiné après le restore(),
qui n'est pas coupé.
```

> **Coût.** Un clip n'est pas gratuit : il ajoute une contrainte que le GPU doit évaluer. Un ou deux clips par frame ne se voient pas ; cinquante, si. `clipPath` avec `doAntiAlias: true` est le plus coûteux des trois.

---

## 21.31 — Le double buffering, ou pourquoi l'écran ne clignote pas

Question légitime : à chaque frame, on efface tout et on redessine. Pourquoi ne voit-on jamais l'écran vide, ne serait-ce qu'un instant ?

Parce que **vous ne dessinez jamais directement sur l'écran**. Vous dessinez dans un tampon (buffer) invisible. Quand la frame est terminée, le système **échange** les deux tampons d'un coup.

```text
  TAMPON AVANT (affiché)          TAMPON ARRIÈRE (en cours de dessin)

  ┌────────────────────┐          ┌────────────────────┐
  │  frame n           │          │  frame n+1         │
  │  visible à l'écran │          │  invisible         │
  └────────────────────┘          └────────────────────┘
            ▲                                │
            └──────────  ÉCHANGE  ───────────┘
                    (au signal VSync)

  L'utilisateur ne voit JAMAIS un dessin en cours de construction.
```

Sans ce mécanisme, on verrait l'écran s'effacer puis se remplir progressivement : c'est le **scintillement** (flicker) des vieilles applications graphiques.

Deux termes à connaître :

- **VSync** : le signal envoyé par l'écran à chaque rafraîchissement (60 fois par seconde sur un écran 60 Hz). L'échange des tampons est synchronisé dessus. C'est pour cela que la boucle de jeu du chapitre 20 tourne à 60 images par seconde et pas plus : le `Ticker` est cadencé par le VSync.
- **Tearing** (déchirure) : l'artefact qui apparaît si l'échange a lieu au milieu du balayage de l'écran, sans VSync. La moitié haute affiche la frame n, la moitié basse la frame n+1. Flutter synchronise toujours, vous ne le rencontrerez pas.

Conséquence pratique pour vous : **vous n'avez rien à faire**. Pas de `flip()`, pas de `swapBuffers()`, pas de gestion de tampon. Vous décrivez la frame complète dans `paint()`, Flutter s'occupe du reste.

Une seule chose à retenir : **`paint()` doit toujours dessiner la scène entière**. Le tampon arrière n'est pas garanti de contenir la frame précédente. Si vous ne redessinez que « ce qui a changé », vous obtiendrez des résidus imprévisibles. Commencez donc chaque `paint()` par un fond opaque couvrant toute la zone.

---

## 21.32 — Performance : ce qui coûte cher dans `paint()`

Le budget est simple et impitoyable :

```text
  60 images par seconde  ->  1 / 60 = 16,7 ms par frame

  ┌──────────────────────────────────────────────────┐
  │ 16,7 ms                                          │
  │ ├── logique du jeu (update)     ~2 ms            │
  │ ├── construction des widgets    ~2 ms            │
  │ ├── VOTRE paint()               ~4 ms            │
  │ └── rasterisation GPU           ~6 ms            │
  └──────────────────────────────────────────────────┘

  Dépasser 16,7 ms = une frame sautée = une saccade visible.
```

Voici, par ordre d'importance décroissante, ce qui coûte cher.

### 1. Créer des objets dans `paint()`

C'est le premier coupable. `paint()` tourne 60 fois par seconde. Chaque `Paint()`, chaque `Path()`, chaque `TextPainter` créé à l'intérieur est un objet à allouer puis à collecter.

```dart
// MAUVAIS : 60 x 3 objets Paint par seconde, pour rien.
void paint(Canvas canvas, Size size) {
  for (final e in ennemis) {
    canvas.drawRect(e.rect, Paint()..color = e.couleur);
  }
}

// BON : un seul objet, réutilisé et simplement reconfiguré.
final Paint _p = Paint();
void paint(Canvas canvas, Size size) {
  for (final e in ennemis) {
    _p.color = e.couleur;
    canvas.drawRect(e.rect, _p);
  }
}
```

Déclarez vos `Paint` en champs `final` de la classe, ou mieux en `static final` si les couleurs sont fixes.

### 2. Le texte

Mesurer du texte (`TextPainter.layout()`) est l'opération la plus lourde du dessin 2D. Ne remesurez que si le contenu a changé. Un score qui n'évolue pas ne doit pas être remis en page 60 fois par seconde.

### 3. Le nombre d'appels de dessin

Chaque `drawXxx` a un coût fixe. 5 000 appels à `drawCircle` sont beaucoup plus lents qu'un seul `drawPoints` avec 5 000 points. Regroupez quand c'est possible :

| Au lieu de | Utilisez |
| --- | --- |
| N `drawCircle` de petits points | un `drawPoints` |
| N `drawLine` d'une polyligne | un `drawPath` ou `drawPoints(polygon)` |
| N `drawRect` d'un décor fixe | un `Picture` pré-enregistré, ou une image |

### 4. Le surdessin (overdraw)

Peindre dix fois le même pixel coûte dix fois. Évitez d'empiler des fonds opaques inutiles, et ne dessinez pas ce qui est hors de l'écran :

```dart
final ecran = Offset.zero & size;
for (final e in entites) {
  if (!ecran.overlaps(e.rect)) continue;  // culling : on saute
  e.dessiner(canvas);
}
```

Ce filtrage (« culling ») est indispensable dès que le monde est plus grand que l'écran (chapitre 25).

### 5. Les effets coûteux

`MaskFilter.blur`, `saveLayer()`, `clipPath` avec anticrénelage et les `BlendMode` exotiques forcent le moteur à travailler hors écran. Utilisez-les avec parcimonie.

### 6. `shouldRepaint` trop permissif

Renvoyer `true` en permanence désactive le cache de rendu. Comparez toujours les champs.

### Mesurer plutôt que deviner

Trois outils, dans l'ordre :

| Outil | Usage |
| --- | --- |
| `flutter run --profile` | mesurer sur un vrai appareil, jamais en debug |
| Overlay de performance | `MaterialApp(showPerformanceOverlay: true)` |
| Flutter DevTools, onglet Performance | voir la durée réelle de chaque frame |

> **Règle d'or.** N'optimisez jamais à l'aveugle. Écrivez d'abord du code clair, mesurez, puis corrigez le point réellement lent. Les six règles ci-dessus suffisent pour 95 % des cas.

---

## 21.33 — Mini-projet : dessiner une grille de donjon

Rassemblons tout. Objectif : afficher une salle du Donjon de Dart décrite par une grille de caractères, comme au chapitre 18.

La technique est toujours la même : une `List<String>` où chaque caractère est un type de case, et une double boucle qui convertit `(colonne, ligne)` en `Rect`.

```text
  CONVERSION GRILLE -> PIXELS

  colonne c, ligne l, tuile de T pixels

     x = c * T
     y = l * T
     rect = Rect.fromLTWH(x, y, T, T)

  '#' mur      '.' sol      'P' potion
  'C' coffre   'K' clé      'H' héros
```

```dart
import 'package:flutter/material.dart';

void main() => runApp(const DonjonGrilleApp());

const List<String> carte = [
  '####################',
  '#..................#',
  '#..K....##.....C...#',
  '#.......##.........#',
  '#...H......P.......#',
  '#........####......#',
  '#..................#',
  '####################',
];

class DonjonGrilleApp extends StatelessWidget {
  const DonjonGrilleApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: const Color(0xFF14161C),
          body: Center(
            child: CustomPaint(
              size: Size(carte.first.length * 32, carte.length * 32),
              painter: GrillePainter(carte: carte, tuile: 32),
            ),
          ),
        ),
      );
}

class GrillePainter extends CustomPainter {
  const GrillePainter({required this.carte, required this.tuile});

  final List<String> carte;
  final double tuile;

  static final Paint _sol = Paint()..color = const Color(0xFF2A2F3C);
  static final Paint _mur = Paint()..color = const Color(0xFF454E63);
  static final Paint _joint = Paint()
    ..color = const Color(0x40232833)
    ..style = PaintingStyle.stroke
    ..strokeWidth = 1;
  static final Paint _or = Paint()..color = const Color(0xFFFFC857);
  static final Paint _rouge = Paint()..color = const Color(0xFFE05A78);
  static final Paint _brun = Paint()..color = const Color(0xFF8A6A3B);
  static final Paint _vert = Paint()..color = const Color(0xFF6FCF6F);

  @override
  void paint(Canvas canvas, Size size) {
    for (int l = 0; l < carte.length; l++) {
      for (int c = 0; c < carte[l].length; c++) {
        final caseRect = Rect.fromLTWH(c * tuile, l * tuile, tuile, tuile);
        final symbole = carte[l][c];

        // 1) Le fond de la case.
        canvas.drawRect(caseRect, symbole == '#' ? _mur : _sol);
        canvas.drawRect(caseRect, _joint);

        // 2) Le contenu, centré dans la case.
        final centre = caseRect.center;
        switch (symbole) {
          case 'K': // une clé : anneau + tige
            canvas.drawCircle(centre + const Offset(-6, 0), 5, _or);
            canvas.drawRect(
                Rect.fromLTWH(centre.dx - 2, centre.dy - 2, 12, 4), _or);
          case 'P': // une potion
            canvas.drawCircle(centre + const Offset(0, 3), 8, _rouge);
            canvas.drawRect(
                Rect.fromCenter(
                    center: centre - const Offset(0, 8), width: 6, height: 8),
                _rouge);
          case 'C': // un coffre
            canvas.drawRect(
                Rect.fromCenter(
                    center: centre, width: tuile - 8, height: tuile - 14),
                _brun);
            canvas.drawRect(
                Rect.fromCenter(
                    center: centre, width: tuile - 8, height: 5),
                _or);
          case 'H': // le héros
            canvas.drawRect(
                Rect.fromCenter(
                    center: centre, width: tuile - 12, height: tuile - 6),
                _vert);
            canvas.drawCircle(centre + const Offset(-5, -5), 2.5,
                Paint()..color = const Color(0xFF14161C));
            canvas.drawCircle(centre + const Offset(5, -5), 2.5,
                Paint()..color = const Color(0xFF14161C));
        }
      }
    }
  }

  @override
  bool shouldRepaint(covariant GrillePainter old) =>
      old.carte != carte || old.tuile != tuile;
}
```

**Résultat :**

```text
Une salle de 20 x 8 cases de 32 pixels, soit 640 x 256 pixels.
Un mur continu fait le tour, deux blocs de murs sont à l'intérieur.
On distingue une clé dorée, une potion rouge, un coffre brun à bande dorée
et le héros vert à deux yeux.
Un fin quadrillage sépare les cases.
```

Ce qu'il faut retenir de ce mini-projet :

1. la carte est une **donnée**, séparée du code de dessin (principe du chapitre 26) ;
2. la conversion `(colonne, ligne) -> Rect` tient en une ligne ;
3. le `switch` sur le symbole utilise la syntaxe des patterns de Dart 3 (chapitre 4) ;
4. les `Paint` sont `static final` : aucune allocation dans `paint()`.

---

## 21.34 — Mini-projet : un personnage en formes géométriques

Sans le moindre fichier image, on peut obtenir un personnage tout à fait lisible. La méthode :

```text
  DÉCOMPOSER EN FORMES SIMPLES, DU FOND VERS L'AVANT

   1. ombre au sol       drawOval
   2. jambes             drawRect x2
   3. corps              drawRRect
   4. bras               drawRect x2
   5. tête               drawCircle
   6. yeux, bouche       drawCircle / drawRect
   7. accessoire (épée)  drawPath

  Tout est décrit dans un repère local dont l'origine est
  les PIEDS du personnage (ancre bottomCenter, section 21.29).
```

Le point clé est l'ancre : en plaçant l'origine locale aux pieds, poser le héros sur un sol revient à `translate(x, ySol)`.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const HerosApp());

class HerosApp extends StatelessWidget {
  const HerosApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: const Color(0xFF14161C),
          body: Center(
            child: CustomPaint(
                size: const Size(460, 260), painter: PersonnagePainter()),
          ),
        ),
      );
}

class PersonnagePainter extends CustomPainter {
  static final Paint _peau = Paint()..color = const Color(0xFFF0C9A0);
  static final Paint _tunique = Paint()..color = const Color(0xFF6FCF6F);
  static final Paint _cuir = Paint()..color = const Color(0xFF8A6A3B);
  static final Paint _acier = Paint()..color = const Color(0xFFC7CEDB);
  static final Paint _noir = Paint()..color = const Color(0xFF14161C);
  static final Paint _ombre = Paint()..color = const Color(0x55000000);

  /// Dessine un personnage dont les PIEDS sont à l'origine locale (0, 0).
  /// [corps] permet de réutiliser la même silhouette pour un ennemi.
  void _personnage(Canvas canvas, Paint corps, {bool armee = true}) {
    // 1. Ombre au sol.
    canvas.drawOval(
      Rect.fromCenter(center: Offset.zero, width: 54, height: 14),
      _ombre,
    );
    // 2. Jambes.
    canvas.drawRect(const Rect.fromLTWH(-16, -26, 12, 26), _cuir);
    canvas.drawRect(const Rect.fromLTWH(4, -26, 12, 26), _cuir);
    // 3. Corps.
    canvas.drawRRect(
      RRect.fromRectAndRadius(
        const Rect.fromLTWH(-20, -70, 40, 46),
        const Radius.circular(8),
      ),
      corps,
    );
    // 4. Bras.
    canvas.drawRect(const Rect.fromLTWH(-28, -66, 8, 30), corps);
    canvas.drawRect(const Rect.fromLTWH(20, -66, 8, 30), corps);
    // 5. Tête.
    canvas.drawCircle(const Offset(0, -86), 17, _peau);
    // 6. Yeux et bouche.
    canvas.drawCircle(const Offset(-6, -90), 2.5, _noir);
    canvas.drawCircle(const Offset(6, -90), 2.5, _noir);
    canvas.drawRect(const Rect.fromLTWH(-5, -80, 10, 2), _noir);
    // 7. Épée, dans la main droite.
    if (armee) {
      final lame = Path()
        ..moveTo(28, -60)
        ..lineTo(34, -60)
        ..lineTo(34, -108)
        ..lineTo(31, -116)
        ..lineTo(28, -108)
        ..close();
      canvas.drawPath(lame, _acier);
      canvas.drawRect(const Rect.fromLTWH(24, -62, 14, 5), _cuir);
    }
  }

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
        Offset.zero & size, Paint()..color = const Color(0xFF20242E));
    // Le sol.
    const ySol = 210.0;
    canvas.drawRect(Rect.fromLTWH(0, ySol, size.width, size.height - ySol),
        Paint()..color = const Color(0xFF2A2F3C));

    // Le héros, debout sur le sol.
    canvas.save();
    canvas.translate(110, ySol);
    _personnage(canvas, _tunique);
    canvas.restore();

    // Un gobelin : même silhouette, autre couleur, plus petit, sans arme,
    // et retourné pour qu'il fasse face au héros.
    canvas.save();
    canvas.translate(320, ySol);
    canvas.scale(-0.8, 0.8);
    _personnage(canvas, Paint()..color = const Color(0xFF8FBF5A),
        armee: false);
    canvas.restore();
  }

  @override
  bool shouldRepaint(covariant PersonnagePainter old) => false;
}
```

**Résultat :**

```text
À gauche, un héros à tunique verte, tête ronde, deux yeux, bouche,
tenant une épée grise pointée vers le haut, posé sur son ombre.
À droite, un gobelin vert olive, 20 % plus petit, retourné (il regarde
vers la gauche, donc vers le héros), sans arme.
Les deux reposent exactement sur la ligne de sol.
```

Retenez la structure : **une méthode par entité, qui dessine dans son repère local**, et un `save/translate/.../restore` pour la poser. C'est le patron que Flame formalise avec les `Component` du chapitre 28.

---

## 21.35 — Animer le dessin avec le `dt` du chapitre 20

Il ne manque plus qu'une chose : le mouvement. Reprenons la boucle du chapitre 20.

Le principe est inchangé :

```text
  Ticker  ──> onTick(elapsed)
                  │
                  ├── dt = elapsed - precedent      (en secondes)
                  ├── update(dt)   : la LOGIQUE change les positions
                  └── setState()   : demande un nouveau dessin
                          │
                          └──> paint() : le RENDU affiche l'état courant
```

La règle d'or du chapitre 20 reste la règle d'or ici :

> Toute vitesse s'exprime en **unités par seconde**, et se multiplie par `dt`.
> Jamais « + 3 pixels par frame », qui donnerait une vitesse différente selon l'appareil.

Le programme ci-dessous fait rebondir une potion dans la salle, fait tourner une clé sur elle-même et fait pulser un halo. Trois animations différentes, un seul `dt`.

```dart
import 'dart:math';
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() => runApp(const AnimationApp());

class AnimationApp extends StatelessWidget {
  const AnimationApp({super.key});

  @override
  Widget build(BuildContext context) => const MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: Color(0xFF14161C),
          body: Center(child: SceneAnimee()),
        ),
      );
}

class SceneAnimee extends StatefulWidget {
  const SceneAnimee({super.key});

  @override
  State<SceneAnimee> createState() => _SceneAnimeeState();
}

class _SceneAnimeeState extends State<SceneAnimee>
    with SingleTickerProviderStateMixin {
  static const Size zone = Size(440, 280);

  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  // État du monde.
  Offset _potion = const Offset(80, 80);
  Offset _vitesse = const Offset(150, 110); // pixels PAR SECONDE
  double _angleCle = 0;                     // radians
  double _temps = 0;                        // secondes écoulées

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_onTick)..start();
  }

  void _onTick(Duration elapsed) {
    final dt = (elapsed - _precedent).inMicroseconds / 1e6;
    _precedent = elapsed;
    if (dt <= 0 || dt > 0.25) return; // on ignore les frames aberrantes
    setState(() => _update(dt));
  }

  void _update(double dt) {
    _temps += dt;

    // 1. Déplacement de la potion : position += vitesse * dt.
    _potion += _vitesse * dt;

    // 2. Rebonds sur les bords (rayon 18).
    const r = 18.0;
    var (vx, vy) = (_vitesse.dx, _vitesse.dy);
    if (_potion.dx < r) { _potion = Offset(r, _potion.dy); vx = -vx; }
    if (_potion.dx > zone.width - r) {
      _potion = Offset(zone.width - r, _potion.dy);
      vx = -vx;
    }
    if (_potion.dy < r) { _potion = Offset(_potion.dx, r); vy = -vy; }
    if (_potion.dy > zone.height - r) {
      _potion = Offset(_potion.dx, zone.height - r);
      vy = -vy;
    }
    _vitesse = Offset(vx, vy);

    // 3. Rotation de la clé : 90 degrés par seconde.
    _angleCle += (pi / 2) * dt;
  }

  @override
  void dispose() {
    _ticker.dispose(); // INDISPENSABLE, sinon fuite mémoire
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return CustomPaint(
      size: zone,
      painter: ScenePainter(
        potion: _potion,
        angleCle: _angleCle,
        temps: _temps,
      ),
    );
  }
}

class ScenePainter extends CustomPainter {
  const ScenePainter({
    required this.potion,
    required this.angleCle,
    required this.temps,
  });

  final Offset potion;
  final double angleCle;
  final double temps;

  static final Paint _fond = Paint()..color = const Color(0xFF20242E);
  static final Paint _rouge = Paint()..color = const Color(0xFFE05A78);
  static final Paint _or = Paint()..color = const Color(0xFFFFC857);
  static final Paint _halo = Paint()
    ..color = const Color(0x556FCF6F)
    ..style = PaintingStyle.stroke
    ..strokeWidth = 5;

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(Offset.zero & size, _fond);

    // La potion qui rebondit.
    canvas.drawCircle(potion, 18, _rouge);
    canvas.drawCircle(potion + const Offset(-6, -6), 5,
        Paint()..color = const Color(0x99FFFFFF));

    // La clé qui tourne sur elle-même : translate PUIS rotate (21.28).
    final centreCle = Offset(size.width / 2, size.height - 60);
    canvas.save();
    canvas.translate(centreCle.dx, centreCle.dy);
    canvas.rotate(angleCle);
    canvas.drawCircle(const Offset(-16, 0), 9, _or);
    canvas.drawRect(
        Rect.fromCenter(center: const Offset(4, 0), width: 32, height: 6), _or);
    canvas.drawRect(
        Rect.fromCenter(center: const Offset(16, 6), width: 6, height: 10), _or);
    canvas.restore();

    // Le halo qui pulse : sinus du temps, entre 26 et 40 pixels.
    final rayon = 33 + 7 * sin(temps * 3);
    canvas.drawCircle(const Offset(70, 220), rayon, _halo);
    canvas.drawCircle(
        const Offset(70, 220), 16, Paint()..color = const Color(0xFF6FCF6F));
  }

  // Le dessin dépend de trois champs : on les compare tous les trois.
  @override
  bool shouldRepaint(covariant ScenePainter old) =>
      old.potion != potion ||
      old.angleCle != angleCle ||
      old.temps != temps;
}
```

**Résultat :**

```text
Une potion rouge rebondit sur les quatre bords de la zone, à vitesse constante.
Au centre bas, une clé dorée tourne régulièrement sur elle-même
(un quart de tour par seconde).
En bas à gauche, un disque vert entouré d'un anneau dont le rayon
augmente et diminue doucement, en respiration.
```

Trois points méritent votre attention.

1. **La séparation logique / rendu est stricte.** `_update(dt)` change les nombres, `paint()` les affiche. Aucun calcul de jeu n'a lieu dans le painter. C'est la discipline qui rendra le chapitre 26 facile.
2. **Le garde-fou sur `dt`.** `if (dt <= 0 || dt > 0.25) return;` protège des frames aberrantes : première frame, application mise en arrière-plan puis restaurée. Sans lui, la potion se téléporterait au retour de veille.
3. **`_ticker.dispose()`.** Un `Ticker` non libéré continue de tourner après la destruction du widget et provoque une exception ou une fuite. C'est obligatoire.

> **Variante sans `setState`.** On peut aussi passer un `Listenable` au constructeur du `CustomPainter` : `ScenePainter(...) : super(repaint: monNotifier)`. Flutter repeint alors sans reconstruire l'arbre des widgets, ce qui est plus efficace. Nous l'utiliserons au chapitre 26.

---

## 21.36 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| L'écran reste vide, alors que `paint()` est correct | `CustomPaint` sans `child` **et** sans `size` : sa taille vaut `Size.zero`, donc il occupe zéro pixel. | Donnez une taille : `CustomPaint(size: const Size(400, 300), painter: ...)`, ou placez-le dans un `SizedBox.expand`, un `Expanded` ou un `LayoutBuilder`. |
| `size.width` et `size.height` valent `0.0` dans `paint()` | Même cause : le parent n'impose aucune contrainte et aucun `size` n'est fourni. | Contraignez le widget (`SizedBox.expand`, `AspectRatio`, `Expanded`) avant de lire `size`. |
| `setState()` est appelé mais rien ne bouge à l'écran | `shouldRepaint()` retourne toujours `false`, donc Flutter réutilise le rendu précédent. | Comparez les champs qui influencent le dessin : `bool shouldRepaint(covariant MonPainter old) => old.x != x;` |
| Le dessin est repeint 60 fois par seconde sans raison, les FPS chutent | `shouldRepaint()` retourne toujours `true`. | Retournez `true` uniquement quand une donnée du dessin a changé. |
| La forme part de travers, comme inclinée au hasard | `canvas.rotate(90)` : `rotate()` attend des **radians**, pas des degrés. 90 radians valent environ 5157 degrés. | `canvas.rotate(90 * pi / 180)` ou directement `canvas.rotate(pi / 2)`, avec `import 'dart:math';`. |
| Tout ce qui est dessiné **après** une forme est décalé ou tourné | `save()` appelé sans `restore()` correspondant : la transformation reste active pour le reste de `paint()`. | Un `save()` = un `restore()`. Écrivez les deux immédiatement, puis remplissez entre les deux. |
| `Unsupported operation: restore() called more times than save()` | Un `restore()` en trop, souvent après un `return` anticipé placé au milieu d'un bloc `save()` / `restore()`. | Comptez les appels. Évitez de sortir de `paint()` entre un `save()` et son `restore()`. |
| Les FPS s'effondrent quand le nombre d'entités augmente | Un objet `Paint` (ou un `Path`, ou un `TextPainter`) est reconstruit à chaque frame et pour chaque entité. | Déclarez les `Paint` en `static final` au niveau de la classe et réutilisez-les. Ne recréez que ce qui change vraiment. |
| La forme tourne autour du coin haut-gauche de l'écran au lieu de tourner sur elle-même | `rotate()` pivote autour de l'origine **du canvas**, pas autour de la forme. | `canvas.translate(cx, cy); canvas.rotate(a);` puis dessinez la forme centrée sur `Offset.zero`. |
| `'text_painter.dart': Failed assertion: ... !_needsLayout` | Vous appelez `tp.paint(canvas, offset)` sans avoir appelé `tp.layout()`. | Enchaînez toujours : `..layout()` avant `paint()`. |
| Exception au sujet de `textDirection` lors de la création d'un `TextPainter` | Le paramètre `textDirection` est obligatoire. | Ajoutez `textDirection: TextDirection.ltr`. |
| `The method 'linear' isn't defined for the class 'Gradient'` | Collision de noms : `material.dart` expose `Gradient` (la classe de `BoxDecoration`), qui n'a pas de constructeur `linear`. | `import 'dart:ui' as ui;` puis `ui.Gradient.linear(...)`. |
| Le contour dépasse du rectangle attendu de quelques pixels | Un trait `stroke` est centré sur le contour : la moitié de `strokeWidth` déborde vers l'extérieur. | Rétrécissez la zone : `rect.deflate(strokeWidth / 2)`. |
| `strokeWidth` semble ignoré, la forme est pleine | Le `Paint` est resté en `PaintingStyle.fill` (valeur par défaut). | Ajoutez `..style = PaintingStyle.stroke`. |
| Le rectangle n'apparaît pas alors que les coordonnées semblent bonnes | `Rect.fromLTRB(200, 100, 50, 60)` : `left > right` produit un rectangle de largeur négative, qui ne dessine rien. | Utilisez `Rect.fromLTWH`, `Rect.fromCenter` ou `Rect.fromPoints`, qui ne peuvent pas s'inverser. |
| La couleur est totalement invisible | `const Color(0x00FF00)` : le canal alpha vaut `0x00`, donc la couleur est transparente. Un `Color` s'écrit sur **huit** chiffres hexadécimaux, alpha compris. | `const Color(0xFF00FF00)`. Le `FF` de tête est l'opacité maximale. |
| `'withOpacity' is deprecated` | `withOpacity()` est déprécié depuis Flutter 3.27. | `couleur.withValues(alpha: 0.35)`. |
| Le personnage va deux fois plus vite sur un téléphone 120 Hz | La position est incrémentée d'une constante **par frame** au lieu d'être multipliée par `dt` (chapitre 20). | `x += vitesse * dt;` et jamais `x += 2;`. |
| Les traits fins scintillent ou paraissent flous | Coordonnées non alignées sur la grille de pixels : un trait de 1 pixel dessiné en `x = 10.0` s'étale sur deux colonnes. | Décalez d'un demi-pixel (`x = 10.5`) pour les traits d'épaisseur impaire, ou désactivez l'anticrénelage avec `..isAntiAlias = false`. |
| Le dessin déborde sur les widgets voisins | `CustomPaint` **ne coupe pas** ce qui dépasse de sa zone : rien n'oblige `paint()` à rester dans `size`. | Encadrez le dessin par `canvas.clipRect(Offset.zero & size)`, ou enveloppez le widget dans un `ClipRect`. |
| Les tailles codées en dur paraissent minuscules sur un écran haute densité, ou énormes sur un vieil écran | On raisonne en pixels physiques, ou on multiplie soi-même par `devicePixelRatio`. | Raisonnez toujours en **pixels logiques** : Flutter applique lui-même le facteur de densité. |
| Une exception apparaît après avoir quitté l'écran de jeu | Le `Ticker` du chapitre 20 n'a pas été libéré. | `@override void dispose() { _ticker.dispose(); super.dispose(); }` |

---

## 21.37 — Résumé du chapitre

### Les méthodes de dessin de `Canvas`

| Méthode `Canvas` | Ce qu'elle dessine |
| --- | --- |
| `drawColor(color, blendMode)` | remplit **toute** la surface disponible d'une couleur |
| `drawPaint(paint)` | remplit toute la surface avec un `Paint` (utile pour un dégradé de fond) |
| `drawRect(rect, paint)` | un rectangle droit |
| `drawRRect(rrect, paint)` | un rectangle à coins arrondis |
| `drawDRRect(externe, interne, paint)` | l'anneau compris entre deux rectangles arrondis (cadre, bordure) |
| `drawCircle(centre, rayon, paint)` | un disque, ou un cercle si `style = stroke` |
| `drawOval(rect, paint)` | une ellipse inscrite dans le rectangle donné |
| `drawArc(rect, debut, balayage, useCenter, paint)` | un arc d'ellipse, ou une part de camembert si `useCenter` vaut `true` |
| `drawLine(p1, p2, paint)` | un segment de droite (toujours en `stroke`) |
| `drawPath(path, paint)` | une forme libre construite avec `Path` |
| `drawPoints(mode, points, paint)` | des points, des segments deux à deux ou une ligne brisée |
| `drawShadow(path, couleur, elevation, transparentOccluder)` | l'ombre portée d'une forme |
| `drawImage(image, offset, paint)` | une image bitmap — **chapitre 22** |
| `drawImageRect(image, src, dst, paint)` | une portion d'image redimensionnée : la base des sprite sheets — **chapitre 22** |
| `TextPainter.paint(canvas, offset)` | du texte (il n'existe pas de `canvas.drawText`) |

### Les méthodes d'état de `Canvas`

| Méthode `Canvas` | Effet |
| --- | --- |
| `save()` | empile l'état courant (transformations et découpes) |
| `restore()` | dépile et restaure l'état précédent |
| `saveLayer(bounds, paint)` | dessine dans un calque séparé, fusionné au `restore()` (opacité globale, effets) |
| `translate(dx, dy)` | déplace l'origine du repère |
| `rotate(radians)` | pivote le repère autour de son origine, dans le sens horaire |
| `scale(sx, [sy])` | agrandit ou rétrécit le repère (valeur négative = miroir) |
| `skew(sx, sy)` | cisaille le repère (effet d'inclinaison) |
| `transform(matrice)` | applique une matrice 4x4 complète |
| `clipRect(rect)` | limite tout dessin ultérieur à un rectangle |
| `clipRRect(rrect)` | même chose avec des coins arrondis |
| `clipPath(path)` | même chose avec une forme libre |

### Les types géométriques

| Type | Rôle | Construction courante |
| --- | --- | --- |
| `Offset` | un point ou un vecteur : `dx`, `dy` | `Offset(12, 40)`, `Offset.zero` |
| `Size` | des dimensions : `width`, `height` | `Size(400, 300)`, `size.center(Offset.zero)` |
| `Rect` | une zone rectangulaire | `Rect.fromLTWH`, `Rect.fromCenter`, `Rect.fromCircle`, `offset & size` |
| `RRect` | un rectangle à coins arrondis | `RRect.fromRectAndRadius(rect, Radius.circular(8))` |
| `Radius` | un rayon de coin | `Radius.circular(8)`, `Radius.elliptical(12, 6)` |
| `Path` | une forme libre | `moveTo`, `lineTo`, `quadraticBezierTo`, `cubicTo`, `arcTo`, `close` |
| `Color` | une couleur ARGB 32 bits | `Color(0xFFC85A3A)`, `couleur.withValues(alpha: 0.5)` |

### Les propriétés de `Paint`

| Propriété | Rôle |
| --- | --- |
| `color` | couleur de remplissage ou de trait |
| `style` | `PaintingStyle.fill` (plein, défaut) ou `PaintingStyle.stroke` (contour) |
| `strokeWidth` | épaisseur du trait, centrée sur le contour |
| `strokeCap` | extrémité du trait : `butt`, `round`, `square` |
| `strokeJoin` | jonction entre deux segments : `miter`, `round`, `bevel` |
| `shader` | dégradé ou motif appliqué à la place de `color` |
| `maskFilter` | flou, notamment `MaskFilter.blur(BlurStyle.normal, 8)` |
| `blendMode` | mode de fusion avec ce qui est déjà dessiné |
| `isAntiAlias` | lissage des bords, `true` par défaut |

### Les cinq phrases à retenir

1. **`Y` descend.** L'origine `(0, 0)` est en haut à gauche de la zone de dessin, et le paramètre `size` de `paint()` en donne les limites.
2. **On raisonne en pixels logiques.** `devicePixelRatio` est l'affaire de Flutter, pas la vôtre.
3. **L'ordre de dessin est l'ordre d'empilement.** Ce qui est dessiné en dernier est au-dessus : c'est l'algorithme du peintre.
4. **Un `save()` = un `restore()`.** Et `rotate()` prend des radians, autour de l'origine courante du repère.
5. **`paint()` est appelé jusqu'à 120 fois par seconde.** Tout ce qui peut être calculé ailleurs doit l'être ailleurs.

---

## 21.38 — Exercices

Ces dix exercices sont progressifs et se suivent : ils construisent, morceau par morceau, les éléments visuels du Donjon de Dart. Chaque énoncé se résout dans un `main.dart` complet, à exécuter avec `flutter run`. Reprenez le squelette de la section 21.9 : `runApp`, `MaterialApp`, `Scaffold`, `Center`, `CustomPaint` avec un `size` explicite.

Une règle vaut pour tous les exercices : **aucun objet `Paint` ne doit être créé à l'intérieur de `paint()`**.

### Exercice 1 — La dalle centrale (facile)

Dessinez une zone de 360 x 240 pixels contenant :

1. un fond gris foncé qui occupe **toute** la zone, sans écrire `360` ni `240` en dur ;
2. une dalle carrée de 160 x 160 pixels, parfaitement centrée, dans un gris plus clair ;
3. le contour de cette même dalle, en vert, épaisseur 4.

Contrainte : utilisez `Offset.zero & size` pour le fond et `Rect.fromCenter` pour la dalle. Le programme doit rester correct si vous changez la taille du `CustomPaint`.

### Exercice 2 — La rangée de potions (facile)

Dans une zone de 420 x 160, dessinez cinq potions alignées horizontalement, à mi-hauteur, régulièrement espacées, y compris sur les bords.

Chaque potion est composée de trois cercles :

- un disque rouge de rayon 18 ;
- son contour, en rouge sombre, épaisseur 3 ;
- un petit reflet blanc semi-transparent de rayon 5, décalé de `(-6, -6)` par rapport au centre.

Contrainte : le nombre de potions doit être une constante `nb` ; passer `nb` de 5 à 8 ne doit demander aucune autre modification.

### Exercice 3 — Le repère de débogage (facile)

Dans une zone de 380 x 260, matérialisez le repère de l'écran :

1. l'axe `X` : un segment horizontal partant de `(0, 0)`, sur toute la largeur ;
2. l'axe `Y` : un segment vertical partant de `(0, 0)`, sur toute la hauteur ;
3. la diagonale allant de `(0, 0)` au coin opposé, en pointillé simulé (une suite de courts segments) ;
4. un petit disque plein à chaque extrémité de la diagonale.

Contrainte : utilisez uniquement `drawLine()` et `drawCircle()`, et `StrokeCap.round` pour les axes.

### Exercice 4 — Les barres de vie (moyen)

Écrivez une méthode privée `_barre(Canvas canvas, Rect zone, double ratio)` qui dessine une barre de vie à coins arrondis :

- un fond sombre occupant toute la `zone` ;
- une partie pleine dont la largeur vaut `zone.width * ratio` ;
- un contour clair sur toute la `zone`.

La couleur de la partie pleine dépend du ratio : vert au-dessus de 0,5, orange entre 0,25 et 0,5, rouge en dessous. La méthode doit accepter sans planter un `ratio` valant `-0.4` ou `1.8`.

Affichez trois barres empilées : héros à 0,82 ; gobelin à 0,35 ; boss à 0,12.

### Exercice 5 — Le coffre du donjon (moyen)

Écrivez une méthode `_coffre(Canvas canvas, Offset position, double echelle)` qui dessine un coffre **centré sur `position`**, redimensionné par `echelle`, en utilisant `save()`, `translate()`, `scale()` et `restore()`.

Le coffre, décrit dans son repère local centré sur `(0, 0)`, comporte :

- une caisse (rectangle arrondi brun) de 90 x 60, dont le haut est à `y = -10` ;
- un couvercle (rectangle arrondi brun plus clair) de 96 x 30, juste au-dessus ;
- deux bandes métalliques verticales traversant caisse et couvercle ;
- une serrure : un petit rectangle doré et un trou de serrure noir.

Affichez trois coffres à des échelles différentes : 0,7 ; 1,0 ; 1,4. Le code du coffre ne doit être écrit **qu'une seule fois**.

### Exercice 6 — Le blason de la guilde (moyen)

Construisez un `Path` en forme d'écu, décrit dans un repère local centré sur `(0, 0)` : deux angles droits en haut, deux courbes de Bézier quadratiques qui se rejoignent en pointe vers le bas.

Dessinez-le deux fois : une fois rempli, une fois en contour épais par-dessus. Ajoutez, à l'intérieur, une épée stylisée constituée d'un second `Path` (lame, garde, pommeau).

Contrainte : les deux `Path` sont construits **une seule fois**, dans des champs `static final` de la classe, jamais dans `paint()`.

### Exercice 7 — Les étiquettes de nom (moyen)

Dessinez trois entités sous forme de disques colorés, alignées horizontalement : le héros (vert), le gobelin (rouge), le boss (violet).

Au-dessus de chacune, affichez son nom avec un `TextPainter`, **centré horizontalement** sur le disque, posé sur une plaque à coins arrondis dont la taille est calculée à partir des mesures du texte plus une marge de 8 pixels.

Contrainte : la plaque doit s'adapter automatiquement à la longueur du nom, sans aucune largeur codée en dur.

### Exercice 8 — La herse à pointes (difficile)

Dessinez un piège rotatif : un moyeu central, et douze pointes triangulaires disposées en cercle autour de lui, à 70 pixels du centre, chacune orientée vers l'extérieur.

Contraintes :

- le triangle est décrit **une seule fois**, dans un `Path` orienté vers le haut, pointe en `(0, -22)` ;
- la disposition se fait avec une boucle et le trio `save()` / `rotate()` / `translate()` ;
- la pile de sauvegarde doit être exactement équilibrée à la fin de `paint()`.

### Exercice 9 — La torche : découpe et dégradé (difficile)

Dessinez une salle de donjon plus grande que la zone visible, puis limitez son affichage à une lucarne rectangulaire.

1. Remplissez le fond en noir.
2. Ouvrez un `save()`, puis un `clipRect()` sur une lucarne centrée de 300 x 200.
3. À l'intérieur, dessinez une grille de dalles de 40 pixels qui **déborde volontairement** de la lucarne : la découpe doit se voir.
4. Toujours à l'intérieur, dessinez par-dessus un dégradé radial allant du transparent au centre vers le noir presque opaque sur les bords : c'est le halo de la torche.
5. Fermez par `restore()`, puis tracez le contour doré de la lucarne.

Contrainte : utilisez `ui.Gradient.radial` avec `import 'dart:ui' as ui;`.

### Exercice 10 — La salle des pièges animée (difficile)

Réunissez tout le chapitre et le `dt` du chapitre 20 dans une scène animée, dans une zone de 480 x 320 :

- une lame de pendule accrochée au plafond, qui oscille selon `angle = amplitude * sin(2 * pi * temps / periode)` ;
- une pièce d'or qui tourne sur elle-même, simulée par une mise à l'échelle horizontale `scale(cos(a), 1)` ;
- une torche dont le halo pulse doucement ;
- un compteur de temps écoulé, en secondes avec une décimale, affiché en haut à gauche avec un `TextPainter`.

Contraintes :

- un `StatefulWidget` avec `SingleTickerProviderStateMixin`, un `Ticker`, et `dispose()` qui libère le ticker ;
- toute la logique dans une méthode `_update(double dt)`, aucun calcul de jeu dans `paint()` ;
- `dt` ignoré s'il est négatif ou supérieur à 0,25 seconde ;
- un `shouldRepaint()` correct ;
- une mise à l'échelle horizontale jamais nulle (sinon la pièce disparaît complètement).

---

## 21.39 — Corrections des exercices

### Correction 1

```dart
import 'package:flutter/material.dart';

void main() => runApp(const DalleApp());

class DalleApp extends StatelessWidget {
  const DalleApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: const Color(0xFF14161C),
          body: Center(
            child: CustomPaint(
              size: const Size(360, 240),
              painter: DallePainter(),
            ),
          ),
        ),
      );
}

class DallePainter extends CustomPainter {
  const DallePainter();

  static final Paint _fond = Paint()..color = const Color(0xFF20242E);
  static final Paint _dalle = Paint()..color = const Color(0xFF2A2F3C);
  static final Paint _joint = Paint()
    ..color = const Color(0xFF6FCF6F)
    ..style = PaintingStyle.stroke
    ..strokeWidth = 4;

  @override
  void paint(Canvas canvas, Size size) {
    // 1) Le fond : Offset.zero & size donne le Rect (0, 0, largeur, hauteur).
    canvas.drawRect(Offset.zero & size, _fond);

    // 2) La dalle, décrite par son CENTRE : aucune soustraction à faire.
    final Rect dalle = Rect.fromCenter(
      center: size.center(Offset.zero),
      width: 160,
      height: 160,
    );
    canvas.drawRect(dalle, _dalle);

    // 3) Le contour : même Rect, autre Paint.
    canvas.drawRect(dalle, _joint);
  }

  @override
  bool shouldRepaint(covariant DallePainter oldDelegate) => false;
}
```

**Résultat :**

```text
Un rectangle gris foncé de 360 x 240 au centre de la fenêtre.
En son milieu, un carré gris clair de 160 x 160, entouré d'un liseré vert.
```

**Explication :** trois points méritent d'être soulignés.

D'abord, `Offset.zero & size`. L'opérateur `&` de la classe `Offset` fabrique un `Rect` à partir d'un coin et d'une taille. C'est la façon idiomatique de désigner « toute la zone de dessin » : le code ne contient jamais `360` ni `240`, et reste donc juste si la taille change.

Ensuite, `size.center(Offset.zero)`. La méthode `center` d'un `Size` renvoie son centre en le décalant d'un `Offset` de référence ; avec `Offset.zero`, on obtient simplement `Offset(width / 2, height / 2)`. Associée à `Rect.fromCenter`, elle évite le calcul manuel `(size.width - 160) / 2`, source classique d'erreur.

Enfin, le contour. Le même `Rect` est dessiné deux fois, avec deux `Paint` différents : le second est en `PaintingStyle.stroke`. Attention, un trait est **centré** sur le contour : avec `strokeWidth = 4`, deux pixels débordent vers l'extérieur du carré et deux pixels mordent vers l'intérieur. Si vous voulez un liseré strictement intérieur, dessinez `dalle.deflate(2)`.

Les trois `Paint` sont `static final` : ils sont construits une seule fois pour toute l'application, jamais à l'intérieur de `paint()`.

---

### Correction 2

```dart
import 'package:flutter/material.dart';

void main() => runApp(const PotionsApp());

class PotionsApp extends StatelessWidget {
  const PotionsApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: const Color(0xFF14161C),
          body: Center(
            child: CustomPaint(
              size: const Size(420, 160),
              painter: PotionsPainter(),
            ),
          ),
        ),
      );
}

class PotionsPainter extends CustomPainter {
  const PotionsPainter();

  /// Nombre de potions : la seule valeur à changer.
  static const int nb = 5;

  static final Paint _fond = Paint()..color = const Color(0xFF20242E);
  static final Paint _verre = Paint()..color = const Color(0xFFC85A3A);
  static final Paint _contour = Paint()
    ..color = const Color(0xFF7A2E1C)
    ..style = PaintingStyle.stroke
    ..strokeWidth = 3;
  static final Paint _reflet = Paint()
    ..color = const Color(0xFFFFFFFF).withValues(alpha: 0.55);

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(Offset.zero & size, _fond);

    final double y = size.height / 2;

    for (int i = 1; i <= nb; i++) {
      // i / (nb + 1) : nb points régulièrement répartis, bords compris.
      final double x = size.width * i / (nb + 1);
      final Offset centre = Offset(x, y);

      canvas.drawCircle(centre, 18, _verre);
      canvas.drawCircle(centre, 18, _contour);
      canvas.drawCircle(centre + const Offset(-6, -6), 5, _reflet);
    }
  }

  @override
  bool shouldRepaint(covariant PotionsPainter oldDelegate) => false;
}
```

**Résultat :**

```text
Cinq potions rouges alignées à mi-hauteur, régulièrement espacées,
chacune cerclée de rouge sombre et portant un petit reflet blanc
en haut à gauche.
```

**Explication :** le cœur de l'exercice est la formule de répartition.

```text
  nb = 5, largeur = 420

  i :        1     2     3     4     5
  x :       70   140   210   280   350
             |     |     |     |     |
  0 ---------+-----+-----+-----+-----+--------- 420
       70    70    70    70    70    70

  x = largeur * i / (nb + 1)
```

En divisant par `nb + 1` et en faisant varier `i` de `1` à `nb`, on obtient `nb` positions séparées par des intervalles égaux, **y compris** aux extrémités. Avec `i / nb` en partant de zéro, la dernière potion serait collée au bord droit.

Le reflet utilise `withValues(alpha: 0.55)`, la forme moderne de `withOpacity(0.55)` vue en section 21.21. La valeur est calculée une seule fois puisque `_reflet` est `static final`.

Notez enfin que passer `nb` à 8 suffit : la boucle, l'espacement et le rendu s'adaptent seuls. C'est exactement ce qu'on attend d'un code de rendu paramétrable.

---

### Correction 3

```dart
import 'package:flutter/material.dart';

void main() => runApp(const RepereApp());

class RepereApp extends StatelessWidget {
  const RepereApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: const Color(0xFF14161C),
          body: Center(
            child: CustomPaint(
              size: const Size(380, 260),
              painter: ReperePainter(),
            ),
          ),
        ),
      );
}

class ReperePainter extends CustomPainter {
  const ReperePainter();

  static final Paint _fond = Paint()..color = const Color(0xFF20242E);
  static final Paint _axeX = Paint()
    ..color = const Color(0xFF6FCF6F)
    ..style = PaintingStyle.stroke
    ..strokeWidth = 3
    ..strokeCap = StrokeCap.round;
  static final Paint _axeY = Paint()
    ..color = const Color(0xFF5AA9E6)
    ..style = PaintingStyle.stroke
    ..strokeWidth = 3
    ..strokeCap = StrokeCap.round;
  static final Paint _pointille = Paint()
    ..color = const Color(0xFFFFC857)
    ..style = PaintingStyle.stroke
    ..strokeWidth = 2;
  static final Paint _borne = Paint()..color = const Color(0xFFFFC857);

  /// Nombre de tirets potentiels sur la diagonale (un sur deux est dessiné).
  static const int _segments = 14;

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(Offset.zero & size, _fond);

    // 1) Axe X : de l'origine vers la droite.
    canvas.drawLine(Offset.zero, Offset(size.width, 0), _axeX);

    // 2) Axe Y : de l'origine vers le BAS. C'est tout le sujet du chapitre.
    canvas.drawLine(Offset.zero, Offset(0, size.height), _axeY);

    // 3) Diagonale en pointillé : un segment sur deux.
    final Offset fin = Offset(size.width, size.height);
    for (int i = 0; i < _segments; i++) {
      if (i.isOdd) continue;
      final Offset a = Offset.lerp(Offset.zero, fin, i / _segments)!;
      final Offset b = Offset.lerp(Offset.zero, fin, (i + 1) / _segments)!;
      canvas.drawLine(a, b, _pointille);
    }

    // 4) Les deux bornes.
    canvas.drawCircle(Offset.zero, 6, _borne);
    canvas.drawCircle(fin, 6, _borne);
  }

  @override
  bool shouldRepaint(covariant ReperePainter oldDelegate) => false;
}
```

**Résultat :**

```text
Un trait vert longe le bord supérieur, un trait bleu longe le bord gauche :
ce sont les axes X et Y, tous deux issus du coin haut-gauche.
Une diagonale dorée en pointillé relie le coin haut-gauche au coin bas-droit,
avec un disque doré à chaque extrémité.
```

**Explication :** l'exercice a une valeur pédagogique avant d'avoir une valeur graphique. Les deux axes partent du **même point**, `Offset.zero`, et pourtant l'un longe le haut et l'autre longe la gauche : c'est la démonstration visuelle que l'origine est en haut à gauche et que `Y` croît vers le bas.

Trois détails techniques.

`Offset.lerp(a, b, t)` interpole linéairement entre deux points : `t = 0` donne `a`, `t = 1` donne `b`, `t = 0.5` donne le milieu. Comme la méthode accepte des arguments nullables, elle renvoie un `Offset?` ; le `!` est ici parfaitement sûr, les deux points n'étant jamais `null` (chapitre 12).

`i.isOdd` permet de sauter un segment sur deux : c'est la façon la plus simple de simuler un pointillé, Flutter n'ayant pas de trait discontinu natif dans `Paint`.

`StrokeCap.round` arrondit les extrémités des axes. Sans lui (`StrokeCap.butt`, la valeur par défaut), le trait s'arrête net exactement sur le point demandé. Cela se voit surtout sur les traits épais.

Notez qu'une moitié de chaque axe est invisible : le trait est centré sur la ligne `y = 0`, donc 1,5 pixel sort de la zone de dessin. C'est normal, et cela rappelle qu'un `CustomPaint` ne coupe pas ce qui dépasse.

---

### Correction 4

```dart
import 'package:flutter/material.dart';

void main() => runApp(const BarresApp());

class BarresApp extends StatelessWidget {
  const BarresApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: const Color(0xFF14161C),
          body: Center(
            child: CustomPaint(
              size: const Size(400, 220),
              painter: BarresPainter(),
            ),
          ),
        ),
      );
}

class BarresPainter extends CustomPainter {
  const BarresPainter();

  static final Paint _fond = Paint()..color = const Color(0xFF20242E);
  static final Paint _vide = Paint()..color = const Color(0xFF3A2020);
  static final Paint _contour = Paint()
    ..color = const Color(0xFFF0EAD8)
    ..style = PaintingStyle.stroke
    ..strokeWidth = 2;
  static final Paint _vert = Paint()..color = const Color(0xFF6FCF6F);
  static final Paint _orange = Paint()..color = const Color(0xFFFFC857);
  static final Paint _rouge = Paint()..color = const Color(0xFFC85A3A);

  /// Choisit le Paint selon le ratio. Aucun Paint n'est créé ici.
  Paint _couleur(double r) {
    if (r > 0.5) return _vert;
    if (r > 0.25) return _orange;
    return _rouge;
  }

  /// Dessine une barre de vie remplie à [ratio] dans la zone donnée.
  void _barre(Canvas canvas, Rect zone, double ratio) {
    final double r = ratio.clamp(0.0, 1.0);
    final Radius rayon = Radius.circular(zone.height / 2);

    // 1) Le fond, toujours sur toute la largeur.
    canvas.drawRRect(RRect.fromRectAndRadius(zone, rayon), _vide);

    // 2) La partie pleine, proportionnelle. Rien à dessiner si r vaut 0.
    if (r > 0) {
      final Rect pleine =
          Rect.fromLTWH(zone.left, zone.top, zone.width * r, zone.height);
      canvas.drawRRect(
        RRect.fromRectAndRadius(pleine, rayon),
        _couleur(r),
      );
    }

    // 3) Le contour, dessiné en dernier pour passer par-dessus.
    canvas.drawRRect(RRect.fromRectAndRadius(zone, rayon), _contour);
  }

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(Offset.zero & size, _fond);

    const double marge = 40;
    final double largeur = size.width - 2 * marge;

    _barre(canvas, Rect.fromLTWH(marge, 40, largeur, 22), 0.82);
    _barre(canvas, Rect.fromLTWH(marge, 95, largeur, 22), 0.35);
    _barre(canvas, Rect.fromLTWH(marge, 150, largeur, 22), 0.12);
  }

  @override
  bool shouldRepaint(covariant BarresPainter oldDelegate) => false;
}
```

**Résultat :**

```text
Trois barres arrondies empilées, sur fond gris foncé :
- la première remplie aux quatre cinquièmes, en vert ;
- la deuxième remplie au tiers, en orange ;
- la troisième presque vide, en rouge.
Chacune est cerclée d'un fin liseré clair.
```

**Explication :** quatre points structurent cette correction.

**Le `clamp`.** `ratio.clamp(0.0, 1.0)` ramène toute valeur dans l'intervalle autorisé : `-0.4` devient `0.0`, `1.8` devient `1.0`. Sans lui, un ratio négatif produirait un `Rect` de largeur négative (rien ne s'afficherait), et un ratio supérieur à 1 ferait dépasser la barre de son cadre. Dans un vrai jeu, les dégâts finissent toujours par produire une vie négative : la protection est indispensable côté rendu.

**Le rayon des coins.** `Radius.circular(zone.height / 2)` donne des extrémités parfaitement demi-circulaires, quelle que soit la hauteur. Quand la partie pleine devient plus étroite que sa hauteur, les rayons demandés dépassent les dimensions du rectangle : Flutter les réduit proportionnellement de lui-même, sans erreur. La barre du boss, très courte, reste donc une pastille propre.

**Le choix de la couleur.** `_couleur(r)` ne construit rien : elle **retourne** l'un des trois `Paint` préexistants. C'est la bonne façon de faire varier une couleur sans allouer à chaque frame.

**L'ordre de dessin.** Fond, remplissage, contour : l'algorithme du peintre de la section 21.23. Le contour est tracé en dernier pour recouvrir la limite de la partie pleine et masquer l'éventuel crénelage.

---

### Correction 5

```dart
import 'package:flutter/material.dart';

void main() => runApp(const CoffreApp());

class CoffreApp extends StatelessWidget {
  const CoffreApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: const Color(0xFF14161C),
          body: Center(
            child: CustomPaint(
              size: const Size(480, 220),
              painter: CoffrePainter(),
            ),
          ),
        ),
      );
}

class CoffrePainter extends CustomPainter {
  const CoffrePainter();

  static final Paint _fond = Paint()..color = const Color(0xFF20242E);
  static final Paint _caisse = Paint()..color = const Color(0xFF8A6A3B);
  static final Paint _couvercle = Paint()..color = const Color(0xFFA98450);
  static final Paint _metal = Paint()..color = const Color(0xFF6B7280);
  static final Paint _or = Paint()..color = const Color(0xFFFFC857);
  static final Paint _noir = Paint()..color = const Color(0xFF14161C);
  static final Paint _ombre = Paint()..color = const Color(0x55000000);

  /// Dessine un coffre CENTRÉ sur [position], mis à l'échelle par [echelle].
  ///
  /// Tout le dessin est décrit dans un repère local dont l'origine (0, 0)
  /// est le centre du coffre : les coordonnées ne dépendent ni de la
  /// position à l'écran, ni de la taille voulue.
  void _coffre(Canvas canvas, Offset position, double echelle) {
    canvas.save();
    canvas.translate(position.dx, position.dy);
    canvas.scale(echelle);

    // Ombre au sol.
    canvas.drawOval(
      Rect.fromCenter(center: const Offset(0, 54), width: 100, height: 16),
      _ombre,
    );

    // Caisse : 90 x 60, haut à y = -10, donc centre à y = 20.
    canvas.drawRRect(
      RRect.fromRectAndRadius(
        Rect.fromCenter(center: const Offset(0, 20), width: 90, height: 60),
        const Radius.circular(6),
      ),
      _caisse,
    );

    // Couvercle : 96 x 30, juste au-dessus, centre à y = -25.
    canvas.drawRRect(
      RRect.fromRectAndRadius(
        Rect.fromCenter(center: const Offset(0, -25), width: 96, height: 30),
        const Radius.circular(10),
      ),
      _couvercle,
    );

    // Deux bandes métalliques verticales, de -40 à 50.
    for (final double x in <double>[-28, 28]) {
      canvas.drawRect(
        Rect.fromLTWH(x - 4, -40, 8, 90),
        _metal,
      );
    }

    // Serrure : plaque dorée et trou noir.
    canvas.drawRect(
      Rect.fromCenter(center: const Offset(0, -6), width: 22, height: 26),
      _or,
    );
    canvas.drawCircle(const Offset(0, -10), 4, _noir);
    canvas.drawRect(
      Rect.fromCenter(center: const Offset(0, -2), width: 4, height: 10),
      _noir,
    );

    canvas.restore();
  }

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(Offset.zero & size, _fond);

    final double y = size.height / 2;
    _coffre(canvas, Offset(size.width * 0.18, y), 0.7);
    _coffre(canvas, Offset(size.width * 0.50, y), 1.0);
    _coffre(canvas, Offset(size.width * 0.82, y), 1.4);
  }

  @override
  bool shouldRepaint(covariant CoffrePainter oldDelegate) => false;
}
```

**Résultat :**

```text
Trois coffres bruns alignés horizontalement, de taille croissante
(petit, normal, grand), chacun avec son couvercle plus clair,
ses deux bandes métalliques, sa serrure dorée et son ombre au sol.
Le plus grand déborde légèrement en hauteur : c'est voulu.
```

**Explication :** cette correction illustre le motif le plus important du chapitre, celui que vous réutiliserez dans toute la partie 2.

```text
  canvas.save();                       <- on mémorise le repère
  canvas.translate(x, y);              <- l'origine devient le centre du coffre
  canvas.scale(echelle);               <- une unité locale vaut "echelle" pixels
      ... dessin autour de (0, 0) ...
  canvas.restore();                    <- on rend le repère intact
```

Le corps du coffre est décrit **une seule fois**, dans un repère local où `(0, 0)` est son centre. Le même code produit trois coffres différents : seules la translation et l'échelle changent. Si vous aviez écrit les coordonnées absolues, il aurait fallu recopier et recalculer tout le dessin trois fois.

`canvas.scale(echelle)` avec un seul argument applique le même facteur en `X` et en `Y`. Attention, l'échelle agit aussi sur les épaisseurs de trait : un contour de 2 pixels dessiné à l'échelle 1,4 mesurera 2,8 pixels à l'écran. C'est en général ce que l'on veut pour un objet de jeu.

Deux remarques de méthode.

La boucle `for (final double x in <double>[-28, 28])` évite de dupliquer le code des deux bandes. Le littéral de liste est reconstruit à chaque appel : pour un jeu qui dessine des centaines d'objets, on le déclarerait en `static const List<double>`.

Enfin, `restore()` est appelé **avant** de quitter la méthode, et il n'y a aucun `return` entre le `save()` et lui. C'est la discipline qui évite l'erreur la plus pénible du chapitre : un repère laissé transformé, qui décale silencieusement tout le reste de la scène.

---

### Correction 6

```dart
import 'package:flutter/material.dart';

void main() => runApp(const BlasonApp());

class BlasonApp extends StatelessWidget {
  const BlasonApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: const Color(0xFF14161C),
          body: Center(
            child: CustomPaint(
              size: const Size(320, 340),
              painter: BlasonPainter(),
            ),
          ),
        ),
      );
}

class BlasonPainter extends CustomPainter {
  const BlasonPainter();

  static final Paint _fond = Paint()..color = const Color(0xFF20242E);
  static final Paint _champ = Paint()..color = const Color(0xFF2F5D8A);
  static final Paint _bordure = Paint()
    ..color = const Color(0xFFFFC857)
    ..style = PaintingStyle.stroke
    ..strokeWidth = 6
    ..strokeJoin = StrokeJoin.round;
  static final Paint _acier = Paint()..color = const Color(0xFFC7CEDB);
  static final Paint _cuir = Paint()..color = const Color(0xFF8A6A3B);

  /// L'écu, construit UNE SEULE FOIS, centré sur l'origine locale.
  static final Path _ecu = _construireEcu(180, 230);

  /// L'épée, construite UNE SEULE FOIS, centrée sur l'origine locale.
  static final Path _epee = _construireEpee();

  static Path _construireEcu(double l, double h) {
    final Path p = Path();
    p.moveTo(-l / 2, -h / 2);          // coin haut-gauche
    p.lineTo(l / 2, -h / 2);           // coin haut-droit
    p.lineTo(l / 2, h / 6);            // descente du flanc droit
    p.quadraticBezierTo(l / 2, h / 2, 0, h / 2);   // courbe vers la pointe
    p.quadraticBezierTo(-l / 2, h / 2, -l / 2, h / 6); // remontée à gauche
    p.close();                         // referme vers le point de départ
    return p;
  }

  static Path _construireEpee() {
    final Path p = Path();
    // Lame : un long triangle pointe en haut.
    p.moveTo(0, -80);
    p.lineTo(9, -55);
    p.lineTo(9, 35);
    p.lineTo(-9, 35);
    p.lineTo(-9, -55);
    p.close();
    return p;
  }

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(Offset.zero & size, _fond);

    canvas.save();
    canvas.translate(size.width / 2, size.height / 2);

    // 1) L'écu rempli, puis son contour par-dessus.
    canvas.drawPath(_ecu, _champ);
    canvas.drawPath(_ecu, _bordure);

    // 2) L'épée : lame, garde, pommeau.
    canvas.drawPath(_epee, _acier);
    canvas.drawRect(
      Rect.fromCenter(center: const Offset(0, -45), width: 70, height: 12),
      _cuir,
    );
    canvas.drawCircle(const Offset(0, 44), 11, _cuir);

    canvas.restore();
  }

  @override
  bool shouldRepaint(covariant BlasonPainter oldDelegate) => false;
}
```

**Résultat :**

```text
Un écu bleu à bord doré, large en haut et terminé en pointe arrondie
vers le bas, centré dans la zone.
Une épée grise le traverse verticalement : lame pointée vers le haut,
garde brune horizontale, pommeau rond en bas.
```

**Explication :** l'exercice montre comment décrire une forme qui n'est ni un rectangle ni un cercle.

Le tracé de l'écu suit toujours la même logique : on **pose le crayon** avec `moveTo`, on trace des segments avec `lineTo`, des courbes avec `quadraticBezierTo`, puis on referme avec `close()`.

```text
  (-90, -115) ────────────────── (90, -115)
      │                              │
      │            ÉCU               │
      │                              │
  (-90, 38)                      (90, 38)
       \__                        __/
          \___                ___/     <- deux courbes quadratiques
              \___    ____ ___/
                  (0, 115)              <- la pointe
```

`quadraticBezierTo(cx, cy, x, y)` prend d'abord le **point de contrôle**, puis le point d'arrivée. Le point de contrôle attire la courbe sans jamais être atteint : en le plaçant dans le coin `(l / 2, h / 2)`, on obtient exactement l'arrondi d'un écu médiéval.

`close()` n'est pas cosmétique. Sans lui, une forme remplie serait tout de même fermée automatiquement, mais le contour resterait ouvert : il manquerait le segment supérieur, et la jonction des deux coins du haut serait mal dessinée.

Deux points de performance, qui sont l'objet réel de l'exercice.

Les deux `Path` sont des champs `static final`, construits **une fois pour toutes** au premier accès à la classe. Construire un `Path` est coûteux : il faut allouer, remplir un tampon de commandes, puis le convertir. Le refaire soixante fois par seconde pour une forme immobile est un gaspillage caractéristique du débutant.

`strokeJoin = StrokeJoin.round` arrondit les jonctions du contour. Avec la valeur par défaut `StrokeJoin.miter`, les angles très aigus produisent parfois des pointes disgracieuses qui dépassent largement de la forme.

---

### Correction 7

```dart
import 'package:flutter/material.dart';

void main() => runApp(const EtiquettesApp());

class EtiquettesApp extends StatelessWidget {
  const EtiquettesApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: const Color(0xFF14161C),
          body: Center(
            child: CustomPaint(
              size: const Size(520, 240),
              painter: EtiquettesPainter(),
            ),
          ),
        ),
      );
}

/// Une entité du donjon : un nom et une couleur.
class Entite {
  const Entite(this.nom, this.couleur);
  final String nom;
  final Color couleur;
}

class EtiquettesPainter extends CustomPainter {
  const EtiquettesPainter();

  static const List<Entite> entites = <Entite>[
    Entite('Héros', Color(0xFF6FCF6F)),
    Entite('Gobelin', Color(0xFFC85A3A)),
    Entite('Boss Nécromancien', Color(0xFF9B5DE5)),
  ];

  static const TextStyle _style = TextStyle(
    color: Color(0xFF14161C),
    fontSize: 15,
    fontWeight: FontWeight.bold,
  );

  static const double _marge = 8;

  static final Paint _fond = Paint()..color = const Color(0xFF20242E);
  static final Paint _plaque = Paint()..color = const Color(0xFFF0EAD8);
  static final Paint _corps = Paint();

  /// Dessine [texte] sur une plaque arrondie centrée horizontalement
  /// sur [ancre], le bas de la plaque étant posé sur ancre.dy.
  void _etiquette(Canvas canvas, String texte, Offset ancre) {
    final TextPainter tp = TextPainter(
      text: TextSpan(text: texte, style: _style),
      textDirection: TextDirection.ltr,
    )..layout(); // OBLIGATOIRE avant toute lecture de width/height.

    // La plaque est déduite des mesures du texte : rien n'est codé en dur.
    final Rect plaque = Rect.fromCenter(
      center: Offset(ancre.dx, ancre.dy - tp.height / 2 - _marge),
      width: tp.width + 2 * _marge,
      height: tp.height + 2 * _marge,
    );

    canvas.drawRRect(
      RRect.fromRectAndRadius(plaque, const Radius.circular(6)),
      _plaque,
    );

    // paint() place le COIN HAUT-GAUCHE du texte : on recentre à la main.
    tp.paint(
      canvas,
      Offset(plaque.center.dx - tp.width / 2,
          plaque.center.dy - tp.height / 2),
    );
  }

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(Offset.zero & size, _fond);

    final double y = size.height * 0.62;

    for (int i = 0; i < entites.length; i++) {
      final Entite e = entites[i];
      final double x = size.width * (i + 1) / (entites.length + 1);

      _corps.color = e.couleur;
      canvas.drawCircle(Offset(x, y), 26, _corps);

      // L'étiquette est posée 34 pixels au-dessus du centre du disque.
      _etiquette(canvas, e.nom, Offset(x, y - 34));
    }
  }

  @override
  bool shouldRepaint(covariant EtiquettesPainter oldDelegate) => false;
}
```

**Résultat :**

```text
Trois disques colorés alignés : vert, rouge, violet.
Au-dessus de chacun, une plaque claire à coins arrondis portant
son nom en gras et centré. La plaque du "Boss Nécromancien"
est nettement plus large que celle du "Héros" : sa largeur
est calculée à partir du texte mesuré.
```

**Explication :** l'exercice porte sur la seule opération de dessin qui exige une mesure préalable.

`TextPainter` fonctionne en deux temps. Tant que `layout()` n'a pas été appelé, `tp.width` et `tp.height` ne sont pas disponibles et `tp.paint()` déclenche une assertion. Une fois `layout()` appelé, le texte est mesuré, et **ces mesures deviennent la source de toutes les autres coordonnées** : la plaque n'est pas dessinée puis remplie de texte, c'est le texte qui dicte la taille de la plaque.

```text
  1. layout()          ->  tp.width = 132 , tp.height = 20
  2. plaque            ->  132 + 2*8 = 148 de large
                          20  + 2*8 = 36  de haut
  3. tp.paint(canvas, coin haut-gauche du texte)
```

Le second piège est l'origine du texte. `tp.paint(canvas, offset)` place le **coin haut-gauche** du bloc, jamais son centre. Pour centrer, on soustrait la moitié des dimensions mesurées, exactement comme en section 21.20.

Notez le `Paint _corps` unique, dont on change la propriété `color` avant chaque disque. Modifier un `Paint` existant est beaucoup moins coûteux que d'en construire un nouveau ; c'est la technique à retenir quand la couleur varie d'une entité à l'autre.

Un avertissement pour la suite : ici, trois `TextPainter` sont créés à chaque appel de `paint()`. C'est acceptable pour une scène statique, cela ne l'est plus dans une boucle de jeu. Pour un HUD animé, gardez le `TextPainter` dans un champ et n'appelez `layout()` que si le texte a réellement changé (section 21.32).

---

### Correction 8

```dart
import 'dart:math';
import 'package:flutter/material.dart';

void main() => runApp(const HerseApp());

class HerseApp extends StatelessWidget {
  const HerseApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: const Color(0xFF14161C),
          body: Center(
            child: CustomPaint(
              size: const Size(320, 320),
              painter: HersePainter(),
            ),
          ),
        ),
      );
}

class HersePainter extends CustomPainter {
  const HersePainter();

  static const int nbPointes = 12;
  static const double rayon = 70;

  static final Paint _fond = Paint()..color = const Color(0xFF20242E);
  static final Paint _acier = Paint()..color = const Color(0xFFC7CEDB);
  static final Paint _moyeu = Paint()..color = const Color(0xFF6B7280);
  static final Paint _rivet = Paint()..color = const Color(0xFF3A3F4B);
  static final Paint _cercle = Paint()
    ..color = const Color(0x336FCF6F)
    ..style = PaintingStyle.stroke
    ..strokeWidth = 2;

  /// Un triangle pointant vers le HAUT, base sur l'origine locale.
  static final Path _pointe = Path()
    ..moveTo(0, -22)
    ..lineTo(11, 0)
    ..lineTo(-11, 0)
    ..close();

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(Offset.zero & size, _fond);

    canvas.save();
    // L'origine passe au centre de la zone : tout devient symétrique.
    canvas.translate(size.width / 2, size.height / 2);

    // Cercle de repère, pour visualiser le rayon de disposition.
    canvas.drawCircle(Offset.zero, rayon, _cercle);

    for (int i = 0; i < nbPointes; i++) {
      canvas.save();

      // 1) On tourne le repère d'une fraction de tour complet.
      canvas.rotate(2 * pi * i / nbPointes);
      // 2) On avance de "rayon" vers le HAUT du repère tourné.
      canvas.translate(0, -rayon);
      // 3) On dessine toujours le MÊME triangle, à la MÊME place locale.
      canvas.drawPath(_pointe, _acier);

      canvas.restore();
    }

    // Moyeu central, dessiné en dernier pour masquer les bases des pointes.
    canvas.drawCircle(Offset.zero, rayon * 0.55, _moyeu);
    canvas.drawCircle(Offset.zero, 10, _rivet);

    canvas.restore();
  }

  @override
  bool shouldRepaint(covariant HersePainter oldDelegate) => false;
}
```

**Résultat :**

```text
Un disque gris au centre de la zone, entouré de douze pointes
métalliques triangulaires régulièrement réparties, chacune dirigée
vers l'extérieur. Un fin cercle vert translucide passe par la base
des pointes.
```

**Explication :** cette correction est la démonstration la plus claire du principe énoncé en section 21.28 : **on ne déplace pas les formes, on déplace le repère**.

Le triangle `_pointe` est décrit une fois, pointe vers le haut, base à l'origine. Il n'est jamais recalculé. Ce sont les transformations qui le placent :

```text
  i = 0   rotate(0)          translate(0, -70)   ->  pointe vers le HAUT
  i = 3   rotate(pi/2)       translate(0, -70)   ->  pointe vers la DROITE
  i = 6   rotate(pi)         translate(0, -70)   ->  pointe vers le BAS
  i = 9   rotate(3*pi/2)     translate(0, -70)   ->  pointe vers la GAUCHE
```

L'ordre `rotate` puis `translate` est essentiel. Le `translate(0, -rayon)` s'exprime dans le repère **déjà tourné** : il envoie donc la pointe dans une direction différente à chaque tour de boucle. Si vous inversiez les deux lignes, les douze pointes se retrouveraient au même endroit, toutes empilées sur le point situé 70 pixels au-dessus du centre, mais orientées différemment. Essayez : c'est la meilleure façon de graver l'ordre des transformations dans votre mémoire.

`2 * pi * i / nbPointes` répartit `nbPointes` angles sur un tour complet. Le calcul est en **radians** : `2 * pi` correspond à 360 degrés. Changer `nbPointes` de 12 à 20 suffit à densifier la herse.

Comptons enfin la pile de sauvegarde : un `save()` au début, un `save()` et un `restore()` par itération, un `restore()` à la fin. Chaque `save()` a bien son `restore()`, y compris à l'intérieur de la boucle. C'est cette symétrie qui permet de dessiner le moyeu au centre après la boucle, sans se soucier des rotations appliquées.

---

### Correction 9

```dart
import 'dart:ui' as ui;
import 'package:flutter/material.dart';

void main() => runApp(const TorcheApp());

class TorcheApp extends StatelessWidget {
  const TorcheApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: const Color(0xFF14161C),
          body: Center(
            child: CustomPaint(
              size: const Size(420, 300),
              painter: TorchePainter(),
            ),
          ),
        ),
      );
}

class TorchePainter extends CustomPainter {
  const TorchePainter();

  static const double tuile = 40;

  static final Paint _noir = Paint()..color = const Color(0xFF000000);
  static final Paint _dalleA = Paint()..color = const Color(0xFF2A2F3C);
  static final Paint _dalleB = Paint()..color = const Color(0xFF232733);
  static final Paint _joint = Paint()
    ..color = const Color(0xFF161A22)
    ..style = PaintingStyle.stroke
    ..strokeWidth = 2;
  static final Paint _cadre = Paint()
    ..color = const Color(0xFFFFC857)
    ..style = PaintingStyle.stroke
    ..strokeWidth = 3;

  // Le halo : un dégradé radial du transparent vers le noir.
  // Le shader est fixe, donc construit une seule fois.
  static final Paint _halo = Paint()
    ..shader = ui.Gradient.radial(
      const Offset(150, 100), // centre, en coordonnées de la lucarne
      150,
      const <Color>[
        Color(0x00000000), // au centre : totalement transparent
        Color(0x66000000), // à mi-chemin : pénombre
        Color(0xF2000000), // au bord : presque noir
      ],
      const <double>[0.0, 0.55, 1.0],
    );

  @override
  void paint(Canvas canvas, Size size) {
    // 1) Fond entièrement noir.
    canvas.drawRect(Offset.zero & size, _noir);

    // 2) La lucarne : 300 x 200 centrée.
    final Rect lucarne = Rect.fromCenter(
      center: size.center(Offset.zero),
      width: 300,
      height: 200,
    );

    canvas.save();
    canvas.clipRect(lucarne);
    // L'origine passe dans le coin de la lucarne : le halo est décrit
    // dans ce repère local, ce qui simplifie ses coordonnées.
    canvas.translate(lucarne.left, lucarne.top);

    // 3) Une grille qui DÉBORDE volontairement (de -2 à +2 tuiles).
    for (int l = -2; l < 2 + (200 / tuile).ceil(); l++) {
      for (int c = -2; c < 2 + (300 / tuile).ceil(); c++) {
        final Rect dalle = Rect.fromLTWH(c * tuile, l * tuile, tuile, tuile);
        canvas.drawRect(dalle, (c + l).isEven ? _dalleA : _dalleB);
        canvas.drawRect(dalle, _joint);
      }
    }

    // 4) Le halo de la torche, par-dessus les dalles.
    canvas.drawRect(const Rect.fromLTWH(0, 0, 300, 200), _halo);

    canvas.restore();

    // 5) Le cadre doré, dessiné HORS de la découpe.
    canvas.drawRect(lucarne, _cadre);
  }

  @override
  bool shouldRepaint(covariant TorchePainter oldDelegate) => false;
}
```

**Résultat :**

```text
Un rectangle noir de 420 x 300. En son centre, une lucarne de 300 x 200
bordée d'or laisse voir un damier de dalles grises.
Les dalles sont coupées net sur les quatre bords de la lucarne :
elles continuent en réalité au-delà, mais rien ne dépasse.
Le centre de la lucarne est bien éclairé, les coins s'assombrissent
progressivement jusqu'au noir.
```

**Explication :** trois mécanismes se combinent ici.

**La découpe.** `clipRect(lucarne)` installe un masque : tout ce qui est dessiné ensuite est limité à ce rectangle. La grille est volontairement calculée de `-2` à `+2` tuiles au-delà des bords ; sans le `clipRect`, elle déborderait sur le fond noir et se superposerait aux widgets voisins. La découpe fait partie de l'état du canvas : elle est posée après un `save()` et disparaît au `restore()`, ce qui permet de dessiner le cadre doré sans qu'il soit rogné.

**Le dégradé radial.** `ui.Gradient.radial(centre, rayon, couleurs, stops)` produit un `Shader`, que l'on affecte à `paint.shader`. Les `stops` indiquent où chaque couleur est atteinte, de `0.0` (le centre) à `1.0` (le bord du rayon). En allant du transparent vers le noir opaque, on n'éclaire rien : on **assombrit le pourtour**, ce qui donne exactement l'impression d'une torche. C'est l'illusion la plus économique du jeu 2D.

Attention à l'import. `material.dart` expose déjà une classe `Gradient` (celle des `BoxDecoration`), sans constructeur `linear` ni `radial`. Sans le préfixe `ui.`, le code ne compile pas. C'est le piège de la section 21.22.

**Le repère local.** Après `translate(lucarne.left, lucarne.top)`, l'origine se trouve dans le coin haut-gauche de la lucarne. Le halo et la grille sont donc décrits en coordonnées `0..300` et `0..200`, sans jamais faire intervenir la position de la lucarne dans la fenêtre. Le shader, dont le centre est fixé à `Offset(150, 100)`, reste correct même si la zone change de taille, puisqu'il est exprimé dans ce repère local. C'est ce même mécanisme qui deviendra la **caméra** au chapitre 25.

Une remarque de performance : `_halo` est un champ `static final`. Un shader de dégradé est un objet coûteux à construire ; le recréer à chaque frame est l'une des causes les plus fréquentes de chute de FPS dans un jeu Flutter.

---

### Correction 10

```dart
import 'dart:math';
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() => runApp(const SalleApp());

class SalleApp extends StatelessWidget {
  const SalleApp({super.key});

  @override
  Widget build(BuildContext context) => const MaterialApp(
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          backgroundColor: Color(0xFF14161C),
          body: Center(child: SalleDesPieges()),
        ),
      );
}

class SalleDesPieges extends StatefulWidget {
  const SalleDesPieges({super.key});

  @override
  State<SalleDesPieges> createState() => _SalleDesPiegesState();
}

class _SalleDesPiegesState extends State<SalleDesPieges>
    with SingleTickerProviderStateMixin {
  static const Size zone = Size(480, 320);

  // Réglages de l'animation, en unités PAR SECONDE.
  static const double amplitude = 0.9;   // radians (environ 51 degrés)
  static const double periode = 2.4;     // secondes pour un aller-retour
  static const double tourPiece = 2.2;   // radians par seconde

  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  // État du monde : quatre nombres, et rien d'autre.
  double _temps = 0;        // secondes écoulées
  double _angleLame = 0;    // radians
  double _anglePiece = 0;   // radians
  double _halo = 0;         // rayon du halo, en pixels

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_onTick)..start();
  }

  void _onTick(Duration elapsed) {
    final double dt = (elapsed - _precedent).inMicroseconds / 1e6;
    _precedent = elapsed;
    if (dt <= 0 || dt > 0.25) return; // frames aberrantes ignorées
    setState(() => _update(dt));
  }

  /// Toute la logique est ici. paint() ne fait que dessiner.
  void _update(double dt) {
    _temps += dt;
    _angleLame = amplitude * sin(2 * pi * _temps / periode);
    _anglePiece += tourPiece * dt;
    _halo = 30 + 6 * sin(_temps * 4);
  }

  @override
  void dispose() {
    _ticker.dispose(); // INDISPENSABLE
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => CustomPaint(
        size: zone,
        painter: SallePainter(
          temps: _temps,
          angleLame: _angleLame,
          anglePiece: _anglePiece,
          halo: _halo,
        ),
      );
}

class SallePainter extends CustomPainter {
  const SallePainter({
    required this.temps,
    required this.angleLame,
    required this.anglePiece,
    required this.halo,
  });

  final double temps;
  final double angleLame;
  final double anglePiece;
  final double halo;

  static const double longueurTige = 150;

  static final Paint _fond = Paint()..color = const Color(0xFF20242E);
  static final Paint _sol = Paint()..color = const Color(0xFF2A2F3C);
  static final Paint _tige = Paint()
    ..color = const Color(0xFF6B7280)
    ..style = PaintingStyle.stroke
    ..strokeWidth = 5
    ..strokeCap = StrokeCap.round;
  static final Paint _acier = Paint()..color = const Color(0xFFC7CEDB);
  static final Paint _pivot = Paint()..color = const Color(0xFF3A3F4B);
  static final Paint _or = Paint()..color = const Color(0xFFFFC857);
  static final Paint _orSombre = Paint()..color = const Color(0xFFB88C1F);
  static final Paint _flamme = Paint()..color = const Color(0xFFFF9A3C);
  static final Paint _lueur = Paint()
    ..color = const Color(0xFFFF9A3C).withValues(alpha: 0.18);
  static final Paint _bois = Paint()..color = const Color(0xFF8A6A3B);

  /// La lame, décrite autour de son propre centre.
  static final Path _lame = Path()
    ..moveTo(-52, 0)
    ..lineTo(0, -26)
    ..lineTo(52, 0)
    ..lineTo(0, 12)
    ..close();

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(Offset.zero & size, _fond);
    canvas.drawRect(
      Rect.fromLTWH(0, size.height - 46, size.width, 46),
      _sol,
    );

    _dessinerPendule(canvas, Offset(size.width / 2, 24));
    _dessinerPiece(canvas, Offset(90, size.height - 90));
    _dessinerTorche(canvas, Offset(size.width - 70, 110));
    _dessinerChrono(canvas);
  }

  void _dessinerPendule(Canvas canvas, Offset pivot) {
    canvas.save();
    canvas.translate(pivot.dx, pivot.dy); // l'origine devient le point d'ancrage
    canvas.rotate(angleLame);             // tout le pendule pivote autour de lui

    canvas.drawLine(Offset.zero, const Offset(0, longueurTige), _tige);

    canvas.save();
    canvas.translate(0, longueurTige);    // on descend au bout de la tige
    canvas.drawPath(_lame, _acier);
    canvas.restore();

    canvas.restore();

    // Le pivot est dessiné hors rotation : il ne bouge pas.
    canvas.drawCircle(pivot, 9, _pivot);
  }

  void _dessinerPiece(Canvas canvas, Offset centre) {
    // cos donne la largeur apparente d'un disque vu de biais.
    // On ne descend jamais à 0, sinon la pièce disparaît complètement.
    double facteur = cos(anglePiece).abs();
    if (facteur < 0.12) facteur = 0.12;

    canvas.save();
    canvas.translate(centre.dx, centre.dy);
    canvas.scale(facteur, 1); // aplatissement horizontal uniquement
    canvas.drawCircle(Offset.zero, 26, _or);
    canvas.drawCircle(Offset.zero, 18, _orSombre);
    canvas.restore();
  }

  void _dessinerTorche(Canvas canvas, Offset centre) {
    canvas.drawCircle(centre, halo * 2.2, _lueur);
    canvas.drawRect(
      Rect.fromCenter(center: centre + const Offset(0, 34), width: 10, height: 56),
      _bois,
    );
    canvas.drawCircle(centre, halo * 0.45, _flamme);
    canvas.drawCircle(centre + const Offset(0, -4), halo * 0.22, _or);
  }

  void _dessinerChrono(Canvas canvas) {
    final TextPainter tp = TextPainter(
      text: TextSpan(
        text: 'Temps : ${temps.toStringAsFixed(1)} s',
        style: const TextStyle(color: Color(0xFFF0EAD8), fontSize: 16),
      ),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, const Offset(14, 12));
  }

  // Le dessin dépend de quatre champs : on les compare tous les quatre.
  @override
  bool shouldRepaint(covariant SallePainter old) =>
      old.temps != temps ||
      old.angleLame != angleLame ||
      old.anglePiece != anglePiece ||
      old.halo != halo;
}
```

**Résultat :**

```text
Une salle de 480 x 320 avec un sol plus clair en bas.
Au plafond, une lame accrochée à une tige oscille de gauche à droite,
en ralentissant aux extrémités et en accélérant au passage vertical.
En bas à gauche, une pièce d'or s'aplatit puis s'élargit sans cesse :
elle semble tourner sur elle-même.
À droite, une torche dont la flamme et la lueur respirent doucement.
En haut à gauche, un compteur : "Temps : 12.4 s", qui avance
au rythme réel, quel que soit le nombre d'images par seconde.
```

**Explication :** cette dernière correction réunit le chapitre 20 et le chapitre 21. Cinq points sont à retenir.

**La séparation logique / rendu.** `_update(dt)` modifie quatre nombres ; `paint()` les lit et dessine. Aucun calcul de jeu n'a lieu dans le painter, aucun appel de dessin n'a lieu dans `_update`. Cette discipline paraît excessive sur une scène aussi simple ; elle devient vitale au chapitre 26, quand il faudra mettre le jeu en pause, le rejouer ou le tester sans écran.

**Le mouvement du pendule.** L'angle n'est pas incrémenté, il est **recalculé** à partir du temps absolu : `angle = amplitude * sin(2 * pi * temps / periode)`. Le sinus varie entre `-1` et `+1`, donc l'angle varie entre `-0,9` et `+0,9` radian, et l'oscillation dure exactement `periode` secondes. Cette formule ne dérive jamais, contrairement à une accumulation frame après frame. Retenez la distinction :

```text
  ACCUMULATION            angle += vitesse * dt
     - va toujours dans le même sens (rotation continue)
     - de petites erreurs s'ajoutent avec le temps

  FONCTION DU TEMPS       angle = f(temps)
     - idéale pour un va-et-vient, une pulsation, une respiration
     - exacte à tout instant, aucune dérive possible
```

La pièce d'or utilise la première méthode (elle tourne indéfiniment), la lame et le halo utilisent la seconde.

**La rotation autour du bon point.** `translate(pivot)` puis `rotate(angleLame)` place l'origine au point d'ancrage : la tige et la lame pivotent donc **autour du plafond**, exactement comme un vrai pendule. Le second couple `save()` / `translate(0, longueurTige)` descend au bout de la tige, où la lame est dessinée autour de son propre centre. Deux niveaux de repères imbriqués, deux `save()`, deux `restore()`. C'est la structure de base d'un squelette animé, et c'est ce que Flame appellera un arbre de composants au chapitre 28.

**L'aplatissement de la pièce.** `scale(facteur, 1)` ne touche qu'à l'axe `X`. Comme `cos` passe par zéro, la pièce deviendrait invisible et paraîtrait clignoter ; le plancher à `0.12` conserve une fine tranche visible, ce qui rend l'illusion bien plus lisible. C'est une astuce classique du jeu 2D : simuler une rotation en trois dimensions avec une simple échelle.

**Le garde-fou sur `dt`.** `if (dt <= 0 || dt > 0.25) return;` protège des deux cas pathologiques du chapitre 20 : la toute première frame, dont l'écart est nul ou incohérent, et le retour d'arrière-plan, où l'écart peut valoir plusieurs secondes. Sans lui, la lame ferait un bond brutal au réveil de l'application.

Un point d'amélioration, pour aller plus loin : le `TextPainter` du chronomètre est reconstruit à chaque frame. C'est inévitable ici, puisque le texte change réellement soixante fois par seconde. En revanche, un titre fixe comme « DONJON DE DART » devrait être mesuré une seule fois et conservé dans un champ.

---

## Et maintenant ?

Vous savez dessiner. Le repère de l'écran n'a plus de secret : l'origine est en haut à gauche, `Y` descend, et le paramètre `size` de `paint()` vous donne les limites de votre monde. Vous manipulez `Offset`, `Rect` et `Size` sans hésiter, vous configurez un `Paint`, vous tracez des rectangles, des cercles, des ellipses, des chemins, du texte et des dégradés. Vous savez surtout déplacer le repère plutôt que les formes, empiler proprement vos `save()` et vos `restore()`, dessiner autour d'un centre, découper une zone, et faire vivre tout cela avec le `dt` du chapitre 20.

Autrement dit : vous êtes capable de construire un jeu entier sans le moindre fichier image. Le mini-projet du personnage en formes géométriques n'était pas un exercice de style, c'est une solution de repli tout à fait viable — de nombreux jeux publiés n'utilisent rien d'autre.

Mais un donjon peuplé de rectangles atteint vite ses limites. Un héros qui marche, une torche qui vacille, un gobelin qui frappe : tout cela se dessine mal avec des formes géométriques, et surtout cela se dessine **lentement**, car chaque forme est une commande envoyée au processeur graphique. Les vrais jeux 2D emploient une autre technique : une image unique contient toutes les poses du personnage, et l'on n'affiche à chaque frame que le petit rectangle correspondant à la pose voulue.

C'est le sujet du chapitre suivant : charger une image avec `ui.Image`, découper une planche de sprites avec `drawImageRect()`, enchaîner les images d'une animation à une cadence maîtrisée, et retomber sur nos pieds avec le `dt` que vous connaissez déjà.

Rendez-vous au chapitre 22 : [22-PARTIE-2A—SPRITES-SPRITE-SHEETS-ET-ANIMATION.md](./22-PARTIE-2A—SPRITES-SPRITE-SHEETS-ET-ANIMATION.md)
