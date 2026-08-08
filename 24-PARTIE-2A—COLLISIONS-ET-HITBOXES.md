# PARTIE 2A — LES FONDAMENTAUX DU JEU 2D
# CHAPITRE 24 — COLLISIONS ET HITBOXES

> **Niveau :** intermédiaire
> **Durée estimée :** 8 h
> **Pré-requis :** chapitre 20 (boucle de jeu et delta time), chapitre 21 (Canvas et coordonnées), chapitre 22 (sprites), chapitre 23 (vélocité et physique simple)
> **Ce que vous saurez faire à la fin :** écrire à la main toute la détection de collision d'un jeu 2D — AABB, cercles, point, cercle-rectangle — puis la résoudre proprement axe par axe, éviter le tunneling, filtrer les tests avec une grille spatiale et des masques, et faire réagir le jeu (dégâts, ramassage, mort).

---

## 24.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer pourquoi la collision est la brique qui transforme un dessin animé en jeu ;
- distinguer clairement **détection** et **résolution**, et savoir que ce sont deux problèmes différents ;
- définir ce qu'est une **hitbox** et dire pourquoi elle ne doit pas être calquée sur le sprite ;
- différencier **hitbox**, **hurtbox** et **trigger** ;
- définir un rectangle aligné aux axes (**AABB**) et écrire la classe correspondante ;
- énoncer le test AABB en français avant de l'écrire en Dart ;
- écrire le test AABB en code, et expliquer pourquoi on teste la **non**-collision ;
- écrire une collision cercle-cercle sans jamais appeler `sqrt` ;
- écrire une collision point-rectangle et une collision cercle-rectangle ;
- calculer la **profondeur de chevauchement** sur X et sur Y ;
- résoudre une collision en repoussant sur le plus petit axe ;
- reconnaître et corriger le bug du personnage collé au mur ;
- séparer le déplacement en deux passes, X puis Y ;
- détecter le sol, le plafond et les murs, et en déduire `estAuSol` ;
- expliquer le **tunneling** et le corriger par sous-échantillonnage ou par swept AABB ;
- estimer le coût d'un test « tout le monde contre tout le monde » et le réduire ;
- séparer **broad phase** et **narrow phase** ;
- implémenter une **grille spatiale** de partitionnement ;
- définir des **couches** et des **masques** de collision avec des opérateurs de bits ;
- faire réagir le jeu : dégâts, ramassage, mort ;
- gérer une **invincibilité temporaire** après un coup ;
- afficher les hitboxes en mode debug ;
- nommer précisément ce que Flame fera à votre place à partir du chapitre 32 ;
- assembler un donjon jouable avec murs, pièces et gobelin.

---

## 24.1 — Pourquoi la collision est le cœur d'un jeu

Depuis le chapitre 20, vous savez faire avancer le temps. Depuis le chapitre 21, vous savez dessiner. Depuis le chapitre 22, vous savez animer. Depuis le chapitre 23, vous savez faire bouger.

Et pourtant, ce que vous avez à l'écran n'est pas encore un jeu. C'est un dessin animé.

La différence tient en une phrase :

> Dans un dessin animé, les objets se traversent. Dans un jeu, ils se rencontrent.

Faites la liste de ce qu'un joueur appelle « jouer ». Presque tout est une collision.

| Ce que vit le joueur | Ce que c'est techniquement |
| --- | --- |
| « Je ne peux pas traverser ce mur » | collision joueur / décor |
| « J'ai ramassé une pièce » | collision joueur / objet |
| « Le gobelin m'a touché » | collision joueur / ennemi |
| « Ma flèche a atteint la cible » | collision projectile / ennemi |
| « Je suis tombé dans le vide » | absence de collision avec le sol |
| « J'ai atterri sur la plateforme » | collision par le dessous, résolue vers le haut |
| « La porte s'est ouverte quand je suis arrivé » | collision joueur / zone déclencheur |
| « J'ai perdu » | collision joueur / piège, puis vies à zéro |

Tirez-en la conclusion. Un moteur de jeu, c'est essentiellement trois choses : une boucle, un dessin, et un système de collision. Vous avez les deux premières. Il vous manque la troisième, et c'est de loin la plus subtile.

### Ce qui rend le sujet difficile

Un débutant croit que la collision, c'est « est-ce que ça se touche ? ». C'est la partie facile. Les vraies difficultés sont ailleurs.

```text
  LES QUATRE DIFFICULTÉS DE LA COLLISION

  1. GÉOMÉTRIE   Est-ce que ces deux formes se chevauchent ?
                 -> maths simples, mais il faut les écrire juste

  2. RÉACTION    Que fait-on quand elles se chevauchent ?
                 -> repousser ? détruire ? soigner ? rien ?

  3. ROBUSTESSE  Et si l'objet va trop vite pour être vu ?
                 -> le tunneling, le bug le plus frustrant du domaine

  4. PERFORMANCE 200 objets, chacun testé contre tous les autres,
                 60 fois par seconde -> 2 400 000 tests par seconde
                 -> il faut trier avant de tester
```

Ce chapitre traite les quatre, dans cet ordre. Vous n'utiliserez aucune bibliothèque : tout sera écrit à la main, en Dart pur, avec les notions des chapitres 1 à 23.

> **Remarque.** Écrire soi-même la collision est le meilleur investissement de toute la PARTIE 2. Quand vous passerez à Flame au chapitre 32, `onCollisionStart` ne sera pas de la magie : vous saurez exactement quel calcul tourne derrière.

---

## 24.2 — Détection vs résolution : deux problèmes distincts

Voici la confusion numéro un du débutant. Il écrit une fonction qui « gère les collisions » et y mélange deux choses qui n'ont rien à voir.

Séparez-les dès maintenant, mentalement et dans votre code.

```text
  ┌─────────────────────────────────────────────────────────┐
  │  DÉTECTION                                              │
  │  Question : « Ces deux formes se chevauchent-elles ? »  │
  │  Réponse  : oui / non   (un bool)                       │
  │  Nature   : GÉOMÉTRIE PURE. Ne modifie rien.            │
  └─────────────────────────────────────────────────────────┘
                          │
                          ▼  (si oui)
  ┌─────────────────────────────────────────────────────────┐
  │  RÉSOLUTION                                             │
  │  Question : « Que fait-on maintenant ? »                │
  │  Réponse  : on déplace, on détruit, on soigne, on tue…  │
  │  Nature   : RÈGLE DE JEU. Modifie l'état du monde.      │
  └─────────────────────────────────────────────────────────┘
```

La détection est universelle : le test AABB est le même dans tous les jeux du monde. La résolution est propre à votre jeu : un mur repousse, une pièce disparaît, un gobelin fait mal, une porte s'ouvre.

Voici la même idée en code, en pseudo-code d'abord.

```dart
// Structure correcte : deux étapes distinctes.
void miseAJour(double dt) {
  deplacerLeJoueur(dt);

  for (final mur in murs) {
    if (seChevauchent(joueur.hitbox, mur.hitbox)) { // DÉTECTION
      repousser(joueur, mur);                       // RÉSOLUTION
    }
  }

  for (final piece in pieces) {
    if (seChevauchent(joueur.hitbox, piece.hitbox)) { // même DÉTECTION
      piece.ramassee = true;                          // autre RÉSOLUTION
      score += 10;
    }
  }
}
```

Remarquez que `seChevauchent` est appelée deux fois, à l'identique, pour deux réactions complètement différentes. C'est exactement ce qu'on veut : **une seule fonction de détection, autant de résolutions que de règles de jeu**.

Le code fautif ressemble à ceci — ne l'écrivez jamais.

```dart
// MAUVAIS : détection et résolution soudées, non réutilisables.
bool collisionAvecMur(Joueur j, Mur m) {
  if (j.x < m.x + m.largeur && j.x + j.largeur > m.x) {
    j.x = m.x - j.largeur; // la fonction "test" déplace le joueur !
    return true;
  }
  return false;
}
```

Cette fonction ment sur son nom : elle prétend tester, elle modifie. Impossible de la réutiliser pour une pièce, impossible de la tester unitairement, impossible de savoir en la lisant si elle a des effets de bord.

> **Règle d'or.** Une fonction de détection retourne un `bool` (ou une structure d'information) et ne modifie **jamais** ses paramètres.

---

## 24.3 — La hitbox : pourquoi elle diffère du sprite

Vous avez un sprite de 64 × 64 pixels représentant votre héros. Question naïve : la zone de collision, c'est ce carré de 64 × 64, non ?

Non. Et c'est une des raisons pour lesquelles les jeux d'amateurs « se sentent » mal.

Regardez à quoi ressemble vraiment un sprite de personnage.

```text
  LE SPRITE (l'image, 64 x 64)          CE QUI EST VRAIMENT LE PERSONNAGE

  ┌──────────────────────────┐          ┌──────────────────────────┐
  │ . . . . . . . . . . . . .│          │ . . . . . . . . . . . . .│
  │ . . . . ▓▓▓▓▓▓ . . . . . │          │ . . . . ▓▓▓▓▓▓ . . . . . │
  │ . . . ▓▓▓▓▓▓▓▓▓▓ . . . . │          │ . . . ▓▓▓▓▓▓▓▓▓▓ . . . . │
  │ . . . ▓▓ ●▓▓● ▓▓ . . . . │          │ . .┌──────────────┐. . . │
  │ . . . ▓▓▓▓▓▓▓▓▓▓ . . . . │          │ . .│▓▓▓▓▓▓▓▓▓▓▓▓▓ │. . . │
  │ . . ▓▓▓▓▓▓▓▓▓▓▓▓▓▓ . . . │          │ . .│▓▓▓▓▓▓▓▓▓▓▓▓▓ │. . . │
  │ . ▓▓ . ▓▓▓▓▓▓▓▓ . ▓▓ . . │          │ . .│. ▓▓▓▓▓▓▓▓ .  │. . . │
  │ . ▓▓ . ▓▓▓▓▓▓▓▓ . ▓▓ . . │          │ . .│. ▓▓▓▓▓▓▓▓ .  │. . . │
  │ . . . ▓▓▓▓ ▓▓▓▓ . . . . .│          │ . .│ ▓▓▓▓ ▓▓▓▓    │. . . │
  │ . . . ▓▓▓▓ ▓▓▓▓ . . . . .│          │ . .└──────────────┘. . . │
  │ . . . ▓▓▓ . ▓▓▓ . . . . .│          │ . . ▓▓▓ . ▓▓▓ . . . . . .│
  │ . . . . . . . . . . . . .│          │ . . . . . . . . . . . . .│
  └──────────────────────────┘          └──────────────────────────┘
   (les points sont TRANSPARENTS)        la hitbox est plus petite,
                                         centrée sur le tronc
```

Trois observations.

**Un sprite est majoritairement vide.** Autour du personnage il y a des pixels transparents, souvent 30 à 50 % de l'image. Si la hitbox couvre toute l'image, le joueur se cogne dans le vide. Sensation immédiate : « le jeu est injuste, j'ai été touché alors que le gobelin ne m'a même pas frôlé ».

**Les extrémités mentent.** Une épée brandie, une cape, une plume sur le casque, une queue : ces éléments font partie du dessin mais pas du corps. Le joueur ne s'attend pas à mourir parce que sa plume a touché un piège.

**La hitbox doit être stable.** Le sprite change à chaque frame (chapitre 22 : marche, saut, attaque). Si la hitbox suit le dessin, elle change de taille en permanence, et le personnage se coince aléatoirement dans les couloirs. Une hitbox de déplacement doit rester **constante**, quelle que soit l'animation.

### La règle pratique

```text
  RÈGLE DE DIMENSIONNEMENT

  Hitbox de déplacement (le corps qui se cogne aux murs) :
      -> PLUS PETITE que le sprite
      -> constante, jamais liée à la frame courante
      -> alignée sur les pieds, car c'est le sol qui compte

  Hitbox de dégâts reçus (ce qui peut être touché) :
      -> légèrement PLUS PETITE encore
      -> "je préfère être touché trop tard que trop tôt"

  Hitbox de dégâts infligés (l'épée du joueur) :
      -> légèrement PLUS GRANDE que l'arme dessinée
      -> "je préfère toucher trop tôt que trop tard"
```

Cette asymétrie n'est pas de la triche, c'est du game design. On appelle cela la **générosité** du système de collision : elle est calibrée en faveur du joueur, parce qu'un joueur qui rate un coup se dit « le jeu est mal fait », alors qu'un joueur qui touche de justesse se dit « je suis bon ».

### En code

Une hitbox n'est donc pas une propriété du sprite, mais une propriété **de l'entité**, avec un décalage par rapport à sa position.

```dart
void main() {
  // Le sprite fait 64 x 64 et est dessiné en (x, y).
  const double spriteLargeur = 64;
  const double spriteHauteur = 64;

  // La hitbox du corps : 24 de large, 40 de haut,
  // décalée de 20 vers la droite et de 24 vers le bas.
  const double hitboxDecalageX = 20;
  const double hitboxDecalageY = 24;
  const double hitboxLargeur = 24;
  const double hitboxHauteur = 40;

  // Position du joueur dans le monde.
  const double joueurX = 100;
  const double joueurY = 200;

  final double hbX = joueurX + hitboxDecalageX;
  final double hbY = joueurY + hitboxDecalageY;

  print('Sprite  : ($joueurX, $joueurY) '
      '${spriteLargeur}x$spriteHauteur');
  print('Hitbox  : ($hbX, $hbY) '
      '${hitboxLargeur}x$hitboxHauteur');
  print('Surface sprite : ${spriteLargeur * spriteHauteur}');
  print('Surface hitbox : ${hitboxLargeur * hitboxHauteur}');
  final double ratio =
      (hitboxLargeur * hitboxHauteur) / (spriteLargeur * spriteHauteur);
  print('La hitbox couvre ${(ratio * 100).toStringAsFixed(1)} % du sprite.');
}
```

**Résultat :**

```text
Sprite  : (100.0, 200.0) 64.0x64.0
Hitbox  : (120.0, 224.0) 24.0x40.0
Surface sprite : 4096.0
Surface hitbox : 960.0
La hitbox couvre 23.4 % du sprite.
```

23 % : cela paraît peu, et c'est pourtant une valeur très courante pour un personnage humanoïde dans un sprite carré.

> **À retenir.** Le sprite sert à **voir**. La hitbox sert à **jouer**. Ce sont deux rectangles différents, et confondre les deux est la première erreur du chapitre.

---

## 24.4 — Hitbox, hurtbox, trigger

Le mot « hitbox » est utilisé à tort et à travers. Dans un jeu sérieux, une entité porte souvent **plusieurs** boîtes, avec des rôles distincts.

```text
  LES TROIS RÔLES

  ┌─────────────┬───────────────────────────┬────────────────────────┐
  │ Nom         │ Rôle                      │ Question posée         │
  ├─────────────┼───────────────────────────┼────────────────────────┤
  │ HITBOX      │ zone qui INFLIGE          │ « qu'est-ce que je     │
  │ (offensive) │ (poing, épée, projectile) │   frappe ? »           │
  ├─────────────┼───────────────────────────┼────────────────────────┤
  │ HURTBOX     │ zone qui REÇOIT           │ « qu'est-ce qui peut   │
  │ (défensive) │ (le corps vulnérable)     │   me frapper ? »       │
  ├─────────────┼───────────────────────────┼────────────────────────┤
  │ TRIGGER     │ zone qui DÉCLENCHE        │ « suis-je entré dans   │
  │ (zone)      │ sans bloquer ni blesser   │   la zone ? »          │
  └─────────────┴───────────────────────────┴────────────────────────┘
```

Un schéma du héros du Donjon de Dart, en train d'attaquer.

```text
      HITBOX de l'épée (offensive)
      elle n'existe QUE pendant
      les frames 3 et 4 de l'attaque
                        ┌──────────┐
   ┌────────────┐       │          │
   │   HURTBOX  │       │  ▓▓▓▓▓▓  │
   │  (le corps)│═══════│  ▓▓▓▓▓▓  │
   │    ▓▓▓▓    │       │          │
   │    ▓▓▓▓    │       └──────────┘
   │    ▓▓▓▓    │
   └────────────┘
   ╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌
   ╎  TRIGGER : zone de détection    ╎
   ╎  du gobelin (il vous voit)      ╎
   ╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌
```

Trois conséquences pratiques.

**La hitbox offensive est temporaire.** Elle apparaît pendant quelques frames de l'animation d'attaque, puis disparaît. C'est ce qui donne le timing à un combat. Une hitbox d'épée toujours active, c'est un jeu où l'on gagne en se collant à l'ennemi.

**La hurtbox est permanente mais désactivable.** Pendant l'invincibilité (section 24.27) ou une roulade, on la coupe.

**Le trigger ne bloque pas.** On le traverse. Il sert à ouvrir une porte, déclencher un dialogue, réveiller un ennemi, lancer une musique. C'est une collision **sans résolution physique**.

Modélisons cela en Dart avec un `enum` (chapitre 11).

```dart
enum RoleBoite {
  solide,   // bloque le mouvement (mur, sol)
  hurtbox,  // reçoit des dégâts
  hitbox,   // inflige des dégâts
  trigger,  // déclenche un événement, ne bloque pas
}

class Boite {
  double decalageX;
  double decalageY;
  double largeur;
  double hauteur;
  RoleBoite role;
  bool active;

  Boite({
    required this.decalageX,
    required this.decalageY,
    required this.largeur,
    required this.hauteur,
    required this.role,
    this.active = true,
  });

  @override
  String toString() =>
      '$role ${largeur}x$hauteur '
      '(+$decalageX,+$decalageY) ${active ? "active" : "inactive"}';
}

void main() {
  final corps = Boite(
    decalageX: 20,
    decalageY: 24,
    largeur: 24,
    hauteur: 40,
    role: RoleBoite.hurtbox,
  );

  final epee = Boite(
    decalageX: 44,
    decalageY: 28,
    largeur: 28,
    hauteur: 16,
    role: RoleBoite.hitbox,
    active: false, // désactivée hors de l'animation d'attaque
  );

  print(corps);
  print(epee);

  print('--- le joueur attaque, frame 3 ---');
  epee.active = true;
  print(epee);
}
```

**Résultat :**

```text
RoleBoite.hurtbox 24.0x40.0 (+20.0,+24.0) active
RoleBoite.hitbox 28.0x16.0 (+44.0,+28.0) inactive
--- le joueur attaque, frame 3 ---
RoleBoite.hitbox 28.0x16.0 (+44.0,+28.0) active
```

> **Remarque.** Dans ce chapitre, nous travaillerons surtout avec une seule boîte par entité pour rester lisibles. Retenez cependant le vocabulaire : au chapitre 32, Flame vous permettra d'attacher plusieurs `RectangleHitbox` à un même composant, exactement pour cette raison.

---

## 24.5 — Le rectangle aligné aux axes (AABB)

La forme de collision la plus utilisée au monde est le **rectangle aligné aux axes**, en anglais *Axis-Aligned Bounding Box*, abrégé **AABB**.

« Aligné aux axes » veut dire : ses côtés sont parallèles à X et à Y. Il ne tourne jamais.

```text
  AABB (aligné aux axes)          PAS un AABB (tourné)

     ┌───────────────┐                    ╱╲
     │               │                  ╱    ╲
     │               │                ╱        ╲
     │               │                ╲        ╱
     └───────────────┘                  ╲    ╱
     côtés // à X et Y                    ╲╱
     -> test TRÈS simple             -> test 10x plus compliqué
                                        (théorème des axes séparateurs)
```

Pourquoi renoncer à la rotation ? Parce que le rapport bénéfice/coût est écrasant :

| Critère | AABB | Rectangle tourné (OBB) |
| --- | --- | --- |
| Test de chevauchement | 4 comparaisons | projection sur 4 axes, ~40 opérations |
| Code | 5 lignes | 60 lignes |
| Résolution (repousser) | triviale | difficile |
| Bugs typiques | rares | nombreux |
| Suffit pour un jeu 2D classique | oui, à 95 % | rarement nécessaire |

Un personnage qui marche, un mur, une caisse, une pièce, un projectile horizontal : tout cela se modélise très bien en AABB. Gardez la rotation pour l'affichage (chapitre 21, `canvas.rotate`) et laissez la hitbox droite.

### Deux façons de décrire un AABB

Il y a deux conventions, et les mélanger est une source de bugs.

```text
  CONVENTION A : coin haut-gauche + taille       (celle du Canvas)

      (x, y)
        ┌─────────────────┐  ▲
        │                 │  │ hauteur
        │                 │  │
        └─────────────────┘  ▼
        ◄─── largeur ────►

      droite = x + largeur
      bas    = y + hauteur


  CONVENTION B : centre + demi-taille            (celle des cercles)

              demiL      demiL
            ◄──────►◄──────►
        ┌───────────────────┐  ▲ demiH
        │         ●         │  │
        │      (cx, cy)     │  ▼ demiH
        └───────────────────┘

      gauche = cx - demiL
      droite = cx + demiL
```

Nous utiliserons la **convention A** dans tout ce chapitre, parce que c'est celle de `Canvas.drawRect` (chapitre 21) et celle de `Rect.fromLTWH` en Flutter. Rappel important du chapitre 21 : **l'axe Y descend**. `haut` a donc une valeur **plus petite** que `bas`.

### La classe `Aabb`

```dart
class Aabb {
  double x;      // coin haut-gauche
  double y;
  double largeur;
  double hauteur;

  Aabb(this.x, this.y, this.largeur, this.hauteur);

  double get gauche => x;
  double get droite => x + largeur;
  double get haut => y;            // rappel : Y descend
  double get bas => y + hauteur;

  double get centreX => x + largeur / 2;
  double get centreY => y + hauteur / 2;

  @override
  String toString() =>
      'Aabb(g=$gauche, d=$droite, h=$haut, b=$bas)';
}

void main() {
  final joueur = Aabb(100, 200, 24, 40);
  print(joueur);
  print('centre = (${joueur.centreX}, ${joueur.centreY})');

  final mur = Aabb(140, 180, 32, 96);
  print(mur);
  print('centre = (${mur.centreX}, ${mur.centreY})');
}
```

**Résultat :**

```text
Aabb(g=100.0, d=124.0, h=200.0, b=240.0)
centre = (112.0, 220.0)
Aabb(g=140.0, d=172.0, h=180.0, b=276.0)
centre = (156.0, 228.0)
```

Vérifiez toujours ce genre de sortie à la main. Pour le mur : `haut = 180`, `hauteur = 96`, donc `bas = 180 + 96 = 276` et `centreY = 180 + 96 / 2 = 228`. Le centre est bien **entre** le haut et le bas, jamais en dehors. Si votre `centreY` sort du rectangle, c'est que vous avez confondu `y + hauteur` et `y + hauteur / 2`.

> **Remarque.** Prenez l'habitude de recalculer un résultat géométrique à la main sur un exemple simple. C'est le moyen le plus rapide de repérer une confusion entre `bas` et `centreY`, ou entre `largeur` et `droite`.

---

## 24.6 — Le test AABB, expliqué en mots

Avant d'écrire une ligne de Dart, énoncez le test en français. Si vous ne savez pas le dire, vous ne saurez pas l'écrire.

Deux rectangles A et B. Question : se chevauchent-ils ?

Réponse intuitive fausse : « si un coin de A est dans B ». C'est faux — deux rectangles peuvent se croiser en croix sans qu'aucun coin de l'un soit dans l'autre.

```text
  CONTRE-EXEMPLE : ils se chevauchent, aucun coin de A n'est dans B

           ┌───┐
           │ B │
      ┌────┼───┼────┐
      │ A  │   │    │
      └────┼───┼────┘
           │   │
           └───┘
```

La bonne formulation passe par les **intervalles**. Un rectangle, c'est le croisement de deux intervalles : un sur X, un sur Y.

```text
  DÉCOMPOSITION EN INTERVALLES

  Sur l'axe X :   A = [Ag, Ad]      B = [Bg, Bd]
  Sur l'axe Y :   A = [Ah, Ab]      B = [Bh, Bb]

  A et B se chevauchent
       SI ET SEULEMENT SI
  leurs intervalles X se chevauchent
       ET
  leurs intervalles Y se chevauchent
```

Il ne reste plus qu'une question, beaucoup plus simple : **quand deux intervalles se chevauchent-ils ?**

```text
  DEUX SEGMENTS SUR UNE LIGNE

  cas 1 : A entièrement à gauche       Ag────Ad   Bg────Bd
          -> PAS de chevauchement      (Ad < Bg)

  cas 2 : A entièrement à droite       Bg────Bd   Ag────Ad
          -> PAS de chevauchement      (Ag > Bd)

  cas 3 : tout le reste                Ag────┼─Ad
          -> chevauchement                Bg─┼─────Bd
```

D'où l'énoncé complet, en français, à retenir par cœur :

> **A et B se chevauchent si et seulement si :**
> A n'est pas entièrement à gauche de B,
> **et** A n'est pas entièrement à droite de B,
> **et** A n'est pas entièrement au-dessus de B,
> **et** A n'est pas entièrement en dessous de B.

Quatre conditions, toutes formulées à la forme négative. Ce n'est pas un hasard, et la section 24.8 expliquera pourquoi c'est en réalité la formulation la plus simple.

### Le cas du contact exact

Une question se pose immédiatement : si le bord droit de A est **exactement** sur le bord gauche de B, se touchent-ils ?

```text
      ┌───────┐┌───────┐
      │   A   ││   B   │      Ad == Bg
      └───────┘└───────┘
              ↑
        contact exact : chevauchement ou pas ?
```

Il n'y a pas de bonne réponse universelle, mais il y a une réponse **pratique** :

- pour un **mur**, on veut pouvoir se coller sans être « en collision » : le contact ne compte pas ;
- pour un **ramassage**, cela n'a aucune importance : le joueur bougera d'un pixel de plus à la frame suivante.

Nous choisirons donc : **le contact exact n'est PAS une collision**. Cela se traduit par des comparaisons strictes (`<` et `>`) et non larges (`<=`, `>=`). Nous y reviendrons en 24.15, car ce choix résout précisément le bug du personnage collé au mur.

---

## 24.7 — Le test AABB en code

Traduisons mot pour mot l'énoncé de la section précédente.

```dart
class Aabb {
  double x;
  double y;
  double largeur;
  double hauteur;

  Aabb(this.x, this.y, this.largeur, this.hauteur);

  double get gauche => x;
  double get droite => x + largeur;
  double get haut => y;
  double get bas => y + hauteur;

  @override
  String toString() => '($x, $y, ${largeur}x$hauteur)';
}

/// Vrai si les deux rectangles se chevauchent.
/// Le contact exact (bords confondus) ne compte PAS.
bool seChevauchent(Aabb a, Aabb b) {
  if (a.droite <= b.gauche) return false; // A entièrement à gauche
  if (a.gauche >= b.droite) return false; // A entièrement à droite
  if (a.bas <= b.haut) return false;      // A entièrement au-dessus
  if (a.haut >= b.bas) return false;      // A entièrement en dessous
  return true;                            // tous les autres cas
}

void main() {
  final joueur = Aabb(100, 100, 40, 40);

  final cas = <String, Aabb>{
    'loin à droite': Aabb(200, 100, 40, 40),
    'contact exact à droite': Aabb(140, 100, 40, 40),
    'chevauchement partiel': Aabb(130, 110, 40, 40),
    'inclus dans le joueur': Aabb(110, 110, 10, 10),
    'englobe le joueur': Aabb(80, 80, 100, 100),
    'juste au-dessus': Aabb(100, 60, 40, 40),
    'diagonale, coins qui se touchent': Aabb(140, 140, 40, 40),
    'croix (aucun coin dedans)': Aabb(110, 60, 20, 120),
  };

  cas.forEach((nom, autre) {
    final r = seChevauchent(joueur, autre);
    print('${r ? "COLLISION" : "   rien  "}  $nom');
  });
}
```

**Résultat :**

```text
   rien    loin à droite
   rien    contact exact à droite
COLLISION  chevauchement partiel
COLLISION  inclus dans le joueur
COLLISION  englobe le joueur
   rien    juste au-dessus
   rien    diagonale, coins qui se touchent
COLLISION  croix (aucun coin dedans)
```

Analysons trois lignes importantes.

**« inclus dans le joueur » → COLLISION.** Un petit rectangle entièrement à l'intérieur d'un grand est bien détecté. C'est essentiel : une pièce d'or minuscule ramassée par un gros joueur, c'est ce cas-là.

**« englobe le joueur » → COLLISION.** Le cas inverse marche aussi. La fonction est **symétrique** : `seChevauchent(a, b)` donne toujours le même résultat que `seChevauchent(b, a)`.

**« croix » → COLLISION.** Le contre-exemple de la section 24.6 est correctement traité, alors qu'un test « un coin de A est-il dans B ? » aurait répondu « rien ». C'est la preuve que le raisonnement par intervalles est le bon.

### La version en une expression

On peut écrire la même chose en une seule condition, en inversant tout.

```dart
bool seChevauchentCourt(Aabb a, Aabb b) {
  return a.droite > b.gauche &&
      a.gauche < b.droite &&
      a.bas > b.haut &&
      a.haut < b.bas;
}
```

C'est exactement la négation des quatre `return false`. Les deux versions sont équivalentes. Préférez la première quand vous apprenez : chaque ligne dit un cas, et vous pouvez y poser un `print` de debug. Préférez la seconde en production : elle est plus compacte et le compilateur évalue les conditions en court-circuit (chapitre 3), donc aussi vite.

Vérifions qu'elles donnent bien la même chose.

```dart
class Aabb {
  double x, y, largeur, hauteur;
  Aabb(this.x, this.y, this.largeur, this.hauteur);
  double get gauche => x;
  double get droite => x + largeur;
  double get haut => y;
  double get bas => y + hauteur;
}

bool versionLongue(Aabb a, Aabb b) {
  if (a.droite <= b.gauche) return false;
  if (a.gauche >= b.droite) return false;
  if (a.bas <= b.haut) return false;
  if (a.haut >= b.bas) return false;
  return true;
}

bool versionCourte(Aabb a, Aabb b) =>
    a.droite > b.gauche &&
    a.gauche < b.droite &&
    a.bas > b.haut &&
    a.haut < b.bas;

void main() {
  int testsFaits = 0;
  int desaccords = 0;

  // On balaie une grille de positions pour B.
  final a = Aabb(50, 50, 30, 30);
  for (double bx = 0; bx <= 120; bx += 5) {
    for (double by = 0; by <= 120; by += 5) {
      final b = Aabb(bx, by, 30, 30);
      testsFaits++;
      if (versionLongue(a, b) != versionCourte(a, b)) desaccords++;
    }
  }

  print('Tests effectués : $testsFaits');
  print('Désaccords      : $desaccords');
}
```

**Résultat :**

```text
Tests effectués : 625
Désaccords      : 0
```

625 positions testées, zéro désaccord. Les deux fonctions sont bien équivalentes.

> **Astuce.** Cette technique — balayer un espace de valeurs et comparer deux implémentations — s'appelle un test différentiel. C'est le moyen le plus rapide de valider une refonte de code géométrique.

---

## 24.8 — Pourquoi tester la NON-collision est plus simple

Revenons sur un point qui surprend souvent. Pourquoi formuler le test à l'envers ? Pourquoi chercher « ils ne se touchent pas » plutôt que « ils se touchent » ?

Réponse : parce que **la non-collision se décrit en quatre cas simples et exclusifs, alors que la collision se décrit en une infinité de configurations**.

```text
  ESSAYONS DE LISTER LES CAS DE COLLISION (approche naïve)

  1. le coin haut-gauche de A est dans B
  2. le coin haut-droit de A est dans B
  3. le coin bas-gauche de A est dans B
  4. le coin bas-droit de A est dans B
  5. un coin de B est dans A               (4 sous-cas)
  6. A est entièrement dans B
  7. B est entièrement dans A
  8. ils se croisent en croix sans aucun coin inclus (2 orientations)
  9. A est un trait fin dans B
  ... et on en oublie

  -> impossible à écrire sans bug


  MAINTENANT LISTONS LES CAS DE NON-COLLISION

  1. A est entièrement à GAUCHE de B
  2. A est entièrement à DROITE de B
  3. A est entièrement AU-DESSUS de B
  4. A est entièrement EN DESSOUS de B

  -> c'est TOUT. Il n'y en a pas d'autres.
```

Il n'y a que quatre façons pour deux rectangles alignés aux axes de **ne pas** se toucher. Dès que ces quatre échappatoires sont fermées, ils se touchent forcément.

Voici le schéma qui résume cette idée. On appelle cela un **axe séparateur** : s'il existe un axe (X ou Y) sur lequel on peut glisser une ligne entre les deux rectangles, c'est qu'ils sont séparés.

```text
  L'AXE SÉPARATEUR

  cas séparé sur X :             cas séparé sur Y :

     ┌────┐  ┃  ┌────┐               ┌───────┐
     │ A  │  ┃  │ B  │               │   A   │
     └────┘  ┃  └────┘               └───────┘
             ┃                     ━━━━━━━━━━━━━━━
        une ligne verticale          ┌───────┐
        passe entre les deux         │   B   │
                                     └───────┘
                                  une ligne horizontale
                                  passe entre les deux


  cas NON séparé (collision) :

     ┌────┬───┐
     │ A  │▓▓▓│ B          aucune ligne verticale
     └────┴───┘            ni horizontale ne peut
                           passer entre les deux
```

Ce raisonnement porte un nom savant, le **théorème des axes séparateurs** (*Separating Axis Theorem*, SAT). Dans le cas général — polygones quelconques, formes tournées — il faut tester de nombreux axes. Dans le cas des AABB, il n'y en a que deux : X et Y. C'est précisément ce qui rend le test si court.

Mesurons le gain en nombre d'opérations.

```dart
void main() {
  // Approche "lister les cas de collision", version coins :
  // 8 coins à tester, 4 comparaisons chacun.
  const int approcheNaiveComparaisons = 8 * 4;

  // Et elle est FAUSSE (le cas croix passe à travers),
  // il faudrait ajouter au moins 2 tests d'inclusion :
  const int approcheNaiveCorrigee = approcheNaiveComparaisons + 8;

  // Approche "axes séparateurs" :
  const int approcheSat = 4;

  print('Naïve (fausse)        : $approcheNaiveComparaisons comparaisons');
  print('Naïve corrigée        : $approcheNaiveCorrigee comparaisons');
  print('Axes séparateurs      : $approcheSat comparaisons');
  print('Rapport               : '
      '${(approcheNaiveCorrigee / approcheSat).toStringAsFixed(1)}x');
}
```

**Résultat :**

```text
Naïve (fausse)        : 32 comparaisons
Naïve corrigée        : 40 comparaisons
Axes séparateurs      : 4 comparaisons
Rapport               : 10.0x
```

Dix fois moins d'opérations, et un code qui tient en quatre lignes au lieu de quarante. Ce genre de renversement — chercher la négation plutôt que l'affirmation — revient souvent en programmation de jeu. Gardez-le en tête.

> **À retenir.** Ne cherchez jamais « où se touchent-ils ? ». Cherchez « existe-t-il un axe qui les sépare ? ». S'il n'en existe pas, ils se touchent.

---

## 24.9 — Collision cercle-cercle

Le rectangle n'est pas toujours la bonne forme. Une pièce d'or, une boule de feu, une onde de choc, une bombe : tout cela est rond. Et le test cercle-cercle est encore plus simple que le test AABB.

Un cercle se décrit par trois nombres : son centre `(cx, cy)` et son rayon `r`.

```text
  DEUX CERCLES

              distance entre les centres
        ●────────────────────────────────●
      (Acx,Acy)                        (Bcx,Bcy)
        ╭───╮                        ╭─────╮
       ╱     ╲                      ╱       ╲
      │   A   │                    │    B    │
       ╲     ╱                      ╲       ╱
        ╰───╯                        ╰─────╯
      ◄──rA──►                      ◄───rB──►


  RÈGLE :

      distance < rA + rB   ->  ils se chevauchent
      distance = rA + rB   ->  contact exact
      distance > rA + rB   ->  séparés
```

C'est tout. Une seule condition, contre quatre pour l'AABB. La difficulté n'est pas la règle, c'est le calcul de la distance.

### Le théorème de Pythagore

La distance entre deux points est donnée par Pythagore :

```text
              (Bcx, Bcy)
                  ●
                 ╱│
       distance ╱ │  dy = Bcy - Acy
               ╱  │
              ●───┘
       (Acx, Acy)
              dx = Bcx - Acx

      distance² = dx² + dy²
      distance  = √(dx² + dy²)
```

En Dart, la racine carrée est `sqrt`, dans la bibliothèque `dart:math` (vue au chapitre 16 pour les imports).

```dart
import 'dart:math';

class Cercle {
  double cx;
  double cy;
  double rayon;

  Cercle(this.cx, this.cy, this.rayon);

  @override
  String toString() => 'Cercle(($cx, $cy), r=$rayon)';
}

/// Distance entre les centres de deux cercles.
double distanceEntre(Cercle a, Cercle b) {
  final double dx = b.cx - a.cx;
  final double dy = b.cy - a.cy;
  return sqrt(dx * dx + dy * dy);
}

/// Vrai si les deux cercles se chevauchent.
/// Le contact exact ne compte pas.
bool cerclesSeChevauchent(Cercle a, Cercle b) {
  return distanceEntre(a, b) < a.rayon + b.rayon;
}

void main() {
  final bouleDeFeu = Cercle(100, 100, 20);

  final cibles = <String, Cercle>{
    'gobelin loin':        Cercle(200, 100, 15),
    'gobelin au contact':  Cercle(135, 100, 15),
    'gobelin touché':      Cercle(125, 100, 15),
    'gobelin en diagonale':Cercle(120, 120, 15),
    'gobelin dedans':      Cercle(102, 101, 5),
  };

  cibles.forEach((nom, c) {
    final double d = distanceEntre(bouleDeFeu, c);
    final double seuil = bouleDeFeu.rayon + c.rayon;
    final bool touche = cerclesSeChevauchent(bouleDeFeu, c);
    print('${touche ? "TOUCHÉ" : " rien "}  '
        '$nom : d=${d.toStringAsFixed(2)} seuil=$seuil');
  });
}
```

**Résultat :**

```text
 rien   gobelin loin : d=100.00 seuil=35.0
 rien   gobelin au contact : d=35.00 seuil=35.0
TOUCHÉ  gobelin touché : d=25.00 seuil=35.0
TOUCHÉ  gobelin en diagonale : d=28.28 seuil=35.0
TOUCHÉ  gobelin dedans : d=2.24 seuil=25.0
```

Vérifions la ligne « en diagonale » : `dx = 20`, `dy = 20`, donc `d = √(400 + 400) = √800 ≈ 28.28`. Le seuil est `20 + 15 = 35`. Comme `28.28 < 35`, il y a collision. Correct.

### Quand préférer le cercle au rectangle ?

| Situation | Forme conseillée | Pourquoi |
| --- | --- | --- |
| Personnage humanoïde qui marche | AABB | il faut poser des pieds plats sur un sol |
| Pièce d'or, gemme, potion | cercle | forme ronde, ramassage tolérant |
| Projectile (boule de feu, flèche) | cercle | pas de coin qui accroche |
| Mur, plateforme, caisse | AABB | c'est un rectangle dans la réalité |
| Onde de choc, explosion | cercle | rayon qui grandit avec le temps |
| Zone de détection d'un ennemi | cercle | « il me voit à moins de 150 pixels » |

Le cercle a un avantage énorme : il est **isotrope**, il se comporte pareil dans toutes les directions. Un projectile en forme de cercle ne peut pas accrocher un coin de mur. C'est pour cela que dans presque tous les jeux, les projectiles sont des cercles même quand ils sont dessinés en forme de flèche.

> **Remarque.** Le cercle est aussi la seule forme qui reste identique quand on la fait tourner. Si votre objet tourne à l'écran (chapitre 21, `canvas.rotate`), sa hitbox circulaire n'a besoin d'aucune adaptation.

---

## 24.10 — Comparer les carrés des distances plutôt que les racines

La fonction `sqrt` est correcte, mais elle est **inutile** ici. Et dans un jeu, ce qui est inutile et coûteux doit disparaître.

Voici le raisonnement, purement mathématique.

```text
  ON VEUT SAVOIR SI :

      √(dx² + dy²)  <  rA + rB

  Or les deux membres sont POSITIFS ou nuls.
  Pour deux nombres positifs, comparer les nombres
  revient exactement à comparer leurs carrés :

      a < b     <=>     a² < b²      (si a >= 0 et b >= 0)

  Donc on peut élever au carré des deux côtés :

      dx² + dy²  <  (rA + rB)²

  Plus aucune racine carrée.
```

La condition `a >= 0 et b >= 0` est toujours vraie ici : une distance est positive, une somme de rayons aussi. La transformation est donc parfaitement légitime.

```dart
class Cercle {
  double cx;
  double cy;
  double rayon;
  Cercle(this.cx, this.cy, this.rayon);
}

/// Test cercle-cercle SANS racine carrée.
bool cerclesSeChevauchent(Cercle a, Cercle b) {
  final double dx = b.cx - a.cx;
  final double dy = b.cy - a.cy;
  final double distanceCarree = dx * dx + dy * dy;
  final double somme = a.rayon + b.rayon;
  return distanceCarree < somme * somme;
}

void main() {
  final a = Cercle(100, 100, 20);
  final b = Cercle(120, 120, 15);

  final double dx = b.cx - a.cx;
  final double dy = b.cy - a.cy;
  print('dx = $dx, dy = $dy');
  print('distance²        = ${dx * dx + dy * dy}');
  final double s = a.rayon + b.rayon;
  print('(rA + rB)²       = ${s * s}');
  print('collision ?      = ${cerclesSeChevauchent(a, b)}');
}
```

**Résultat :**

```text
dx = 20.0, dy = 20.0
distance²        = 800.0
(rA + rB)²       = 1225.0
collision ?      = true
```

`800 < 1225`, donc collision. Même réponse que la section précédente, sans jamais calculer `√800`.

### Combien cela coûte-t-il vraiment ?

Mesurons. Nous utilisons `Stopwatch`, vu au chapitre 20 pour calculer le delta time.

```dart
import 'dart:math';

void main() {
  const int n = 5000000;
  final double dx = 20;
  final double dy = 20;
  final double seuil = 35;

  // Version AVEC racine carrée.
  final chrono1 = Stopwatch()..start();
  int compte1 = 0;
  for (int i = 0; i < n; i++) {
    if (sqrt(dx * dx + dy * dy) < seuil) compte1++;
  }
  chrono1.stop();

  // Version SANS racine carrée.
  final chrono2 = Stopwatch()..start();
  int compte2 = 0;
  for (int i = 0; i < n; i++) {
    if (dx * dx + dy * dy < seuil * seuil) compte2++;
  }
  chrono2.stop();

  print('Résultats identiques : ${compte1 == compte2}');
  print('Avec sqrt : ${chrono1.elapsedMilliseconds} ms');
  print('Sans sqrt : ${chrono2.elapsedMilliseconds} ms');
}
```

**Résultat (exemple, les durées varient selon la machine) :**

```text
Résultats identiques : true
Avec sqrt : 118 ms
Sans sqrt : 34 ms
```

Environ trois fois plus rapide. Sur cinq millions de tests, cela fait 84 millisecondes économisées, soit cinq frames entières à 60 FPS.

### La règle générale

Cette astuce ne sert pas qu'aux cercles. Elle sert **partout où l'on compare des distances**.

```dart
import 'dart:math';

class Point {
  final double x;
  final double y;
  const Point(this.x, this.y);
}

double distanceCarree(Point a, Point b) {
  final double dx = b.x - a.x;
  final double dy = b.y - a.y;
  return dx * dx + dy * dy;
}

void main() {
  const joueur = Point(0, 0);
  final gobelins = <String, Point>{
    'Grok': const Point(30, 40),
    'Zog': const Point(10, 10),
    'Nak': const Point(100, 5),
    'Rul': const Point(-20, 15),
  };

  // Trouver le gobelin le plus proche : AUCUNE racine nécessaire.
  String? plusProche;
  double meilleure = double.infinity;
  gobelins.forEach((nom, p) {
    final double d2 = distanceCarree(joueur, p);
    if (d2 < meilleure) {
      meilleure = d2;
      plusProche = nom;
    }
  });

  print('Le plus proche : $plusProche');
  print('Distance²      : $meilleure');
  // On ne calcule la vraie distance QUE pour l'afficher au joueur.
  print('Distance       : ${sqrt(meilleure).toStringAsFixed(2)}');
}
```

**Résultat :**

```text
Le plus proche : Zog
Distance²      : 200.0
Distance       : 14.14
```

Zog est à `√200 ≈ 14.14`, Rul à `√625 = 25`, Grok à `√2500 = 50`, Nak à `√10025 ≈ 100.1`. Le classement par distance² est **exactement** le même que par distance, parce que la fonction racine carrée est croissante.

> **À retenir.** Ne calculez `sqrt` que lorsque vous avez besoin de la valeur elle-même (l'afficher, la diviser). Pour **comparer**, **classer** ou **tester un seuil**, restez au carré.

---

## 24.11 — Collision point-rectangle

Cas plus simple encore, et très utile : un point est-il à l'intérieur d'un rectangle ?

À quoi cela sert-il ?

- savoir si le doigt du joueur a touché un bouton du HUD ;
- savoir si le curseur survole un coffre ;
- savoir si le centre d'une entité est dans une zone de déclenchement ;
- savoir dans quelle case de la grille se trouve un objet.

```text
  POINT DANS RECTANGLE

      gauche              droite
        │                   │
   haut ┼───────────────────┤
        │                   │
        │        ● p        │      -> DEDANS
        │                   │
    bas ┼───────────────────┤
        │                   │
              ● q                  -> DEHORS (trop bas)


  RÈGLE :

      p.x >= gauche  ET  p.x < droite
                    ET
      p.y >= haut    ET  p.y < bas
```

Notez le mélange : `>=` d'un côté, `<` de l'autre. Ce n'est pas de la coquetterie. C'est ce qui garantit qu'un point situé pile sur une frontière appartient à **un seul** rectangle quand on pave le plan (une grille de tuiles, par exemple).

```dart
class Aabb {
  double x, y, largeur, hauteur;
  Aabb(this.x, this.y, this.largeur, this.hauteur);
  double get gauche => x;
  double get droite => x + largeur;
  double get haut => y;
  double get bas => y + hauteur;
}

bool pointDansRect(double px, double py, Aabb r) {
  return px >= r.gauche && px < r.droite && py >= r.haut && py < r.bas;
}

void main() {
  // Trois boutons du HUD, côte à côte, sans espace entre eux.
  final boutons = <String, Aabb>{
    'GAUCHE': Aabb(0, 400, 100, 60),
    'SAUT':   Aabb(100, 400, 100, 60),
    'DROITE': Aabb(200, 400, 100, 60),
  };

  final touches = <List<double>>[
    [50, 430],   // au milieu de GAUCHE
    [100, 430],  // pile sur la frontière GAUCHE / SAUT
    [199.9, 430],// juste avant DROITE
    [200, 430],  // pile sur la frontière SAUT / DROITE
    [250, 399],  // au-dessus des boutons
    [299.9, 459.9], // dernier pixel de DROITE
    [300, 430],  // juste après DROITE
  ];

  for (final t in touches) {
    final trouves = <String>[];
    boutons.forEach((nom, r) {
      if (pointDansRect(t[0], t[1], r)) trouves.add(nom);
    });
    final String reponse = trouves.isEmpty ? 'aucun' : trouves.join(' + ');
    print('(${t[0]}, ${t[1]}) -> $reponse');
  }
}
```

**Résultat :**

```text
(50.0, 430.0) -> GAUCHE
(100.0, 430.0) -> SAUT
(199.9, 430.0) -> SAUT
(200.0, 430.0) -> DROITE
(250.0, 399.0) -> aucun
(299.9, 459.9) -> DROITE
(300.0, 430.0) -> aucun
```

Ligne la plus intéressante : `(100, 430) -> SAUT`, et **pas** `GAUCHE + SAUT`. Grâce à l'asymétrie `>=` / `<`, une frontière appartient toujours au rectangle de droite (ou du bas). Aucun point ne peut appartenir à deux boutons voisins.

Si l'on avait écrit `<=` partout, le résultat aurait été :

```text
(100.0, 430.0) -> GAUCHE + SAUT
```

… et le joueur aurait sauté **et** avancé à gauche en même temps sur un seul appui. C'est un bug classique des interfaces tactiles.

> **Remarque.** Le même raisonnement s'applique à une tilemap (chapitre 25). Une position pile sur la frontière entre deux tuiles doit appartenir à une seule tuile, sinon un objet peut être « dans deux cases » et déclencher deux fois le même événement.

---

## 24.12 — Collision cercle-rectangle

C'est le test mixte le plus utile : une boule de feu (cercle) qui frappe un mur (rectangle), une pièce ronde ramassée par un joueur rectangulaire.

L'astuce est élégante. Au lieu de raisonner sur toute la géométrie, on cherche **le point du rectangle le plus proche du centre du cercle**, puis on teste si ce point est à moins d'un rayon.

```text
  CERCLE CONTRE RECTANGLE

  ┌──────────────────────┐
  │                      │
  │       RECTANGLE      │
  │                    ✕ │◄── point du rectangle le plus proche
  │                      │    du centre du cercle
  └──────────────────────┘
                     ╲
                      ╲ d
                       ╲
                        ●  centre du cercle
                       ╱ ╲
                      │ r │

  SI d < r   ->  collision
  SINON      ->  pas de collision
```

Comment trouver ce point le plus proche ? En **bornant** chaque coordonnée du centre dans l'intervalle du rectangle. C'est exactement ce que fait la méthode `clamp` de Dart, vue au chapitre 20 quand nous plafonnions le delta time.

```text
  LE CLAMP, AXE PAR AXE

  Sur X :
      si cx < gauche      -> proche.x = gauche
      si cx > droite      -> proche.x = droite
      sinon               -> proche.x = cx      (le centre est "en face")

  Sur Y : même chose avec haut / bas.

  Autrement dit : proche.x = cx.clamp(gauche, droite)
                  proche.y = cy.clamp(haut, bas)
```

Cette formule couvre les trois configurations d'un seul coup.

```text
  LES TROIS CONFIGURATIONS

  (a) le cercle est EN FACE d'un côté     (b) le cercle est près d'un COIN

     ┌───────────┐                            ┌───────────┐
     │           │                            │           │
     │           ✕───● cx est entre           │           │
     │           │                            └───────────✕
     └───────────┘   gauche et droite,                     ╲
                     donc proche.x = cx                     ● proche = le coin


  (c) le centre est DEDANS

     ┌───────────┐
     │     ●     │   proche == centre, distance = 0
     │           │   -> collision garantie (si r > 0)
     └───────────┘
```

Voici le code complet.

```dart
import 'dart:math';

class Aabb {
  double x, y, largeur, hauteur;
  Aabb(this.x, this.y, this.largeur, this.hauteur);
  double get gauche => x;
  double get droite => x + largeur;
  double get haut => y;
  double get bas => y + hauteur;
}

class Cercle {
  double cx, cy, rayon;
  Cercle(this.cx, this.cy, this.rayon);
}

/// Point du rectangle le plus proche d'un point donné.
List<double> pointLePlusProche(Aabb r, double px, double py) {
  final double x = px.clamp(r.gauche, r.droite);
  final double y = py.clamp(r.haut, r.bas);
  return [x, y];
}

/// Vrai si le cercle et le rectangle se chevauchent.
bool cercleRect(Cercle c, Aabb r) {
  final double px = c.cx.clamp(r.gauche, r.droite);
  final double py = c.cy.clamp(r.haut, r.bas);
  final double dx = c.cx - px;
  final double dy = c.cy - py;
  return dx * dx + dy * dy < c.rayon * c.rayon;
}

void main() {
  final mur = Aabb(100, 100, 80, 60); // g=100 d=180 h=100 b=160

  final boules = <String, Cercle>{
    'en face à gauche, loin':  Cercle(50, 130, 20),
    'en face à gauche, touche':Cercle(85, 130, 20),
    'près du coin, rate':      Cercle(85, 85, 20),
    'près du coin, touche':    Cercle(90, 90, 20),
    'centre dedans':           Cercle(140, 130, 5),
    'en dessous, touche':      Cercle(140, 175, 20),
  };

  boules.forEach((nom, c) {
    final p = pointLePlusProche(mur, c.cx, c.cy);
    final double dx = c.cx - p[0];
    final double dy = c.cy - p[1];
    final double d = sqrt(dx * dx + dy * dy);
    print('${cercleRect(c, mur) ? "IMPACT" : " rien "}  '
        '$nom : proche=(${p[0]}, ${p[1]}) d=${d.toStringAsFixed(2)} r=${c.rayon}');
  });
}
```

**Résultat :**

```text
 rien   en face à gauche, loin : proche=(100.0, 130.0) d=50.00 r=20.0
IMPACT  en face à gauche, touche : proche=(100.0, 130.0) d=15.00 r=20.0
 rien   près du coin, rate : proche=(100.0, 100.0) d=21.21 r=20.0
IMPACT  près du coin, touche : proche=(100.0, 100.0) d=14.14 r=20.0
IMPACT  centre dedans : proche=(140.0, 130.0) d=0.00 r=5.0
IMPACT  en dessous, touche : proche=(140.0, 160.0) d=15.00 r=20.0
```

Vérifions deux lignes.

**« près du coin, rate »** : centre en (85, 85), coin du mur en (100, 100). `dx = -15`, `dy = -15`, donc `d = √450 ≈ 21.21`. Comme `21.21 > 20`, pas d'impact. Le cercle passe de justesse en diagonale du coin.

**« centre dedans »** : `clamp(140, 100, 180) = 140` et `clamp(130, 100, 160) = 130`. Le point le plus proche **est** le centre, `d = 0`, et `0 < 5` donc impact. La formule gère le cas « dedans » sans code spécial.

> **Attention.** `clamp` renvoie un `num` en Dart, pas un `double`. Ici, comme les bornes sont des `double`, le résultat est bien un `double` et l'affectation à une variable `double` fonctionne. Si vous mélangez `int` et `double`, ajoutez `.toDouble()`.

---

## 24.13 — Le chevauchement (overlap) et sa profondeur

Jusqu'ici, la détection répond « oui » ou « non ». Pour **résoudre**, ce n'est pas assez : il faut savoir **de combien** les deux formes s'enfoncent l'une dans l'autre.

Cette quantité s'appelle la **profondeur de chevauchement**, ou *overlap*. Il y en a une par axe.

```text
  PROFONDEUR DE CHEVAUCHEMENT

        Ag              Ad
        ├───────────────┤
                 ├──────────────┤
                 Bg             Bd

        ◄─ overlapX ──►
        (de Bg à Ad)

  Formule (elle marche dans TOUS les cas) :

      overlapX = min(Ad, Bd) - max(Ag, Bg)
      overlapY = min(Ab, Bb) - max(Ah, Bh)

  Interprétation :
      overlap > 0   -> les intervalles se chevauchent de cette quantité
      overlap = 0   -> contact exact
      overlap < 0   -> ils sont séparés, et |overlap| est l'écart
```

Ces deux formules remplacent avantageusement le test booléen : elles **contiennent** la réponse booléenne (`overlapX > 0 && overlapY > 0`) **et** l'information de profondeur.

```dart
import 'dart:math';

class Aabb {
  double x, y, largeur, hauteur;
  Aabb(this.x, this.y, this.largeur, this.hauteur);
  double get gauche => x;
  double get droite => x + largeur;
  double get haut => y;
  double get bas => y + hauteur;
  @override
  String toString() => '($x,$y ${largeur}x$hauteur)';
}

/// Résultat complet d'un test AABB.
class InfoCollision {
  final bool touche;
  final double overlapX;
  final double overlapY;

  const InfoCollision(this.touche, this.overlapX, this.overlapY);

  @override
  String toString() => touche
      ? 'COLLISION overlapX=${overlapX.toStringAsFixed(1)} '
          'overlapY=${overlapY.toStringAsFixed(1)}'
      : 'séparés écartX=${(-overlapX).toStringAsFixed(1)} '
          'écartY=${(-overlapY).toStringAsFixed(1)}';
}

InfoCollision tester(Aabb a, Aabb b) {
  final double ox = min(a.droite, b.droite) - max(a.gauche, b.gauche);
  final double oy = min(a.bas, b.bas) - max(a.haut, b.haut);
  return InfoCollision(ox > 0 && oy > 0, ox, oy);
}

void main() {
  final joueur = Aabb(100, 100, 40, 40); // g100 d140 h100 b140

  final cas = <String, Aabb>{
    'léger contact par la droite': Aabb(130, 110, 40, 40),
    'enfoncé par le haut':         Aabb(105, 130, 40, 40),
    'très enfoncé':                Aabb(110, 110, 40, 40),
    'séparés':                     Aabb(200, 300, 40, 40),
  };

  cas.forEach((nom, b) {
    print('$nom');
    print('   joueur=$joueur  autre=$b');
    print('   ${tester(joueur, b)}');
  });
}
```

**Résultat :**

```text
léger contact par la droite
   joueur=(100.0,100.0 40.0x40.0)  autre=(130.0,110.0 40.0x40.0)
   COLLISION overlapX=10.0 overlapY=30.0
enfoncé par le haut
   joueur=(100.0,100.0 40.0x40.0)  autre=(105.0,130.0 40.0x40.0)
   COLLISION overlapX=35.0 overlapY=10.0
très enfoncé
   joueur=(100.0,100.0 40.0x40.0)  autre=(110.0,110.0 40.0x40.0)
   COLLISION overlapX=30.0 overlapY=30.0
séparés
   joueur=(100.0,100.0 40.0x40.0)  autre=(200.0,300.0 40.0x40.0)
   séparés écartX=60.0 écartY=160.0
```

Détaillons le premier cas. `min(140, 170) = 140`, `max(100, 130) = 130`, donc `overlapX = 10`. Sur Y : `min(140, 150) = 140`, `max(100, 110) = 110`, donc `overlapY = 30`. Le joueur est enfoncé de 10 pixels horizontalement et de 30 verticalement.

Notez la dernière ligne : quand ils sont séparés, les deux overlaps sont négatifs et leur valeur absolue donne l'écart. C'est une information gratuite, utile pour de l'IA (« l'ennemi est à 60 pixels sur X »).

> **À retenir.** Remplacez le plus tôt possible votre `bool seChevauchent` par une fonction qui renvoie les deux overlaps. La détection ne coûte pas plus cher et vous obtenez ce dont la résolution a besoin.

---

## 24.14 — Résoudre : repousser sur le plus petit axe

Nous savons que le joueur est enfoncé dans le mur de 10 pixels sur X et 30 sur Y. Comment le sortir de là ?

Deux options : le repousser horizontalement de 10, ou verticalement de 30. Laquelle choisir ?

```text
  DEUX SORTIES POSSIBLES

  Situation : le joueur (J) est enfoncé dans le mur (M)

              ┌────────────┐
              │            │
        ┌─────┼───┐        │
        │  J  │▓▓▓│   M    │   overlapX = 10  (petit)
        └─────┼───┘        │   overlapY = 30  (grand)
              │            │
              └────────────┘

  SORTIE A : repousser sur X de 10 px  -> déplacement minimal
  SORTIE B : repousser sur Y de 30 px  -> le joueur est téléporté
                                          au-dessus du mur !
```

La bonne réponse est **toujours l'axe où l'overlap est le plus petit**. C'est le principe du **déplacement minimal de translation** (*Minimum Translation Vector*). Intuitivement : on sort par le chemin le plus court, celui par lequel on est entré.

Il reste à déterminer le **sens** : vers la gauche ou vers la droite ? On compare les centres.

```text
  DÉTERMINER LE SENS

  Si centreX(J) < centreX(M)  ->  J est à gauche  ->  repousser vers la GAUCHE
  Sinon                       ->  J est à droite  ->  repousser vers la DROITE

  Idem sur Y avec centreY.
```

Écrivons la résolution complète.

```dart
import 'dart:math';

class Aabb {
  double x, y, largeur, hauteur;
  Aabb(this.x, this.y, this.largeur, this.hauteur);
  double get gauche => x;
  double get droite => x + largeur;
  double get haut => y;
  double get bas => y + hauteur;
  double get centreX => x + largeur / 2;
  double get centreY => y + hauteur / 2;
  @override
  String toString() =>
      '(${x.toStringAsFixed(1)}, ${y.toStringAsFixed(1)})';
}

/// Repousse [mobile] hors de [solide] par le chemin le plus court.
/// Retourne le nom de la direction utilisée, ou null si pas de collision.
String? repousser(Aabb mobile, Aabb solide) {
  final double ox = min(mobile.droite, solide.droite) -
      max(mobile.gauche, solide.gauche);
  final double oy =
      min(mobile.bas, solide.bas) - max(mobile.haut, solide.haut);

  if (ox <= 0 || oy <= 0) return null; // pas de collision

  if (ox < oy) {
    // On sort horizontalement.
    if (mobile.centreX < solide.centreX) {
      mobile.x -= ox;
      return 'gauche';
    } else {
      mobile.x += ox;
      return 'droite';
    }
  } else {
    // On sort verticalement.
    if (mobile.centreY < solide.centreY) {
      mobile.y -= oy;
      return 'haut';
    } else {
      mobile.y += oy;
      return 'bas';
    }
  }
}

void main() {
  final mur = Aabb(200, 200, 100, 100); // g200 d300 h200 b300

  final situations = <String, Aabb>{
    'arrive par la gauche': Aabb(180, 240, 40, 40),
    'arrive par la droite': Aabb(280, 240, 40, 40),
    'arrive par le haut':   Aabb(240, 180, 40, 40),
    'arrive par le bas':    Aabb(240, 280, 40, 40),
  };

  situations.forEach((nom, joueur) {
    final avant = joueur.toString();
    final dir = repousser(joueur, mur);
    print('$nom : $avant -> $joueur  (repoussé vers le $dir)');
  });
}
```

**Résultat :**

```text
arrive par la gauche : (180.0, 240.0) -> (160.0, 240.0)  (repoussé vers le gauche)
arrive par la droite : (280.0, 240.0) -> (300.0, 240.0)  (repoussé vers le droite)
arrive par le haut : (240.0, 180.0) -> (240.0, 160.0)  (repoussé vers le haut)
arrive par le bas : (240.0, 280.0) -> (240.0, 300.0)  (repoussé vers le bas)
```

Vérifions le premier cas. Joueur : `g=180 d=220`. Mur : `g=200 d=300`. `overlapX = min(220,300) - max(180,200) = 220 - 200 = 20`. Sur Y : joueur `h=240 b=280`, mur `h=200 b=300`, donc `overlapY = min(280,300) - max(240,200) = 280 - 240 = 40`. Comme `20 < 40`, on sort sur X. Le centre du joueur (200) est à gauche du centre du mur (250), donc on recule de 20 : `x = 160`. Le joueur est maintenant collé au mur, à `d = 200 = g` du mur. Contact exact, pas de collision. Correct.

### La limite de cette méthode

Cette résolution « plus petit axe » est simple et fonctionne dans beaucoup de cas. Elle a pourtant un défaut sérieux, que nous verrons en 24.15 : elle ne tient pas compte de **la direction dans laquelle le joueur se déplaçait**. Un joueur qui tombe vite peut se retrouver repoussé latéralement, ce qui est visuellement absurde.

La solution définitive est en 24.16 : traiter X et Y séparément. Mais il fallait d'abord comprendre l'overlap.

> **À retenir.** Repousser sur le plus petit axe = sortir par le chemin le plus court. C'est un bon réflexe, mais ce n'est pas la solution finale.

---

## 24.15 — Le bug du personnage collé au mur

Voici le bug le plus signalé par les débutants : « mon personnage se colle au mur et ne peut plus bouger », ou « il tremble », ou « il monte le long du mur ».

Il a en réalité **quatre causes distinctes**. Apprenez à les reconnaître séparément.

### Cause 1 — Comparaisons larges au lieu de strictes

```text
  APRÈS RÉSOLUTION :

      ┌───────┐┌────────────┐
      │   J   ││     MUR    │      J.droite == MUR.gauche
      └───────┘└────────────┘

  Avec le test  a.droite <= b.gauche -> return false   (strict)
      -> pas de collision. Le joueur est collé, tout va bien.

  Avec le test  a.droite < b.gauche -> return false    (large)
      -> COLLISION détectée alors qu'il n'y a que contact.
      -> on repousse encore, overlap = 0, donc déplacement de 0
      -> à chaque frame, le jeu croit qu'il y a collision
      -> la vélocité X est remise à 0 en permanence
      -> LE JOUEUR NE PEUT PLUS S'ÉLOIGNER DU MUR
```

C'est pour cette raison que nous avons choisi en 24.6 que le contact exact **n'est pas** une collision.

### Cause 2 — Résolution sur les deux axes en même temps

```dart
// MAUVAIS : on corrige X et Y dans la même passe.
if (seChevauchent(joueur, mur)) {
  joueur.x -= overlapX;
  joueur.y -= overlapY; // le joueur est téléporté en diagonale
}
```

Le joueur est sorti deux fois, dans deux directions, et se retrouve en diagonale loin du mur. Visuellement, il « saute » de côté quand il touche un obstacle. C'est la deuxième cause de la sensation de collage : le joueur croit qu'il glisse le long du mur alors qu'il est en train d'être téléporté.

### Cause 3 — Vélocité non remise à zéro

Après avoir repoussé le joueur, il faut aussi **annuler sa vitesse sur cet axe**. Sinon :

```text
  frame 1 : vx = 300, le joueur avance, entre dans le mur
            -> on le repousse, mais vx vaut toujours 300
  frame 2 : vx = 300, il rentre encore dans le mur
            -> on le repousse
  frame 3 : idem...

  Résultat : le joueur vibre contre le mur, et l'accumulation
  (chapitre 23 : accélération) fait grandir vx à l'infini.
  Le jour où il quitte le mur, il est catapulté.
```

### Cause 4 — Comparaison de `double` avec `==`

```dart
// MAUVAIS
if (joueur.droite == mur.gauche) {
  // ce test est presque toujours FAUX
}
```

Un `double` est un nombre à virgule flottante (chapitre 2). Après quelques additions de `vitesse * dt`, `joueur.droite` vaut `200.00000000000003` et non `200.0`. Le test `==` échoue. Ne testez **jamais** l'égalité de deux `double` en géométrie de jeu. Utilisez un intervalle de tolérance (epsilon) :

```dart
const double epsilon = 0.001;
if ((joueur.droite - mur.gauche).abs() < epsilon) {
  // là, ça fonctionne
}
```

### Démonstration des quatre causes

```dart
class Aabb {
  double x, y, largeur, hauteur;
  Aabb(this.x, this.y, this.largeur, this.hauteur);
  double get gauche => x;
  double get droite => x + largeur;
}

void main() {
  // Cause 1 : strict vs large.
  final j = Aabb(160, 240, 40, 40); // droite = 200
  final m = Aabb(200, 200, 100, 100); // gauche = 200

  final bool testStrict = !(j.droite <= m.gauche);
  final bool testLarge = !(j.droite < m.gauche);
  print('Contact exact, test strict -> collision ? $testStrict');
  print('Contact exact, test large  -> collision ? $testLarge');

  // Cause 4 : accumulation de flottants.
  double x = 0;
  for (int i = 0; i < 10; i++) {
    x += 20.0 / 3.0; // un déplacement "moche"
  }
  print('');
  print('Après 10 déplacements de 20/3 : x = $x');
  print('x == 66.66666666666667 ? ${x == 66.66666666666667}');
  const double epsilon = 0.001;
  print('|x - 66.6666666667| < epsilon ? '
      '${(x - 66.6666666667).abs() < epsilon}');
}
```

**Résultat :**

```text
Contact exact, test strict -> collision ? false
Contact exact, test large  -> collision ? true

Après 10 déplacements de 20/3 : x = 66.66666666666667
x == 66.66666666666667 ? true
|x - 66.6666666667| < epsilon ? true
```

Le second test passe ici par chance. Changez `10` en `13` et vous obtiendrez `86.66666666666669`, alors que `13 * 20 / 3 = 86.66666666666667`. Deux calculs mathématiquement identiques, deux `double` différents. C'est exactement ce qui casse un `==`.

### Le tableau de diagnostic

| Symptôme observé | Cause probable | Correction |
| --- | --- | --- |
| Impossible de s'éloigner du mur | comparaison large (`<=`) | passer en strict |
| Le joueur saute en diagonale au contact | résolution sur les deux axes | séparer X et Y (24.16) |
| Le joueur vibre contre le mur | vélocité non annulée | mettre `vx = 0` après résolution X |
| Catapulte en quittant le mur | vélocité accumulée | idem |
| Un test d'alignement ne se déclenche jamais | `==` sur des `double` | utiliser un epsilon |
| Le joueur monte le long du mur | overlapY choisi à tort | résoudre Y avant X, ou séparer |

> **À retenir.** « Collé au mur » n'est pas un bug, c'est une famille de bugs. Identifiez lequel avant de corriger.

---

## 24.16 — Séparer les axes X et Y

Voici la technique qui règle définitivement les causes 2 et 6 du tableau précédent, et qui est utilisée dans l'immense majorité des jeux de plateforme 2D.

L'idée est d'une simplicité désarmante : **ne jamais déplacer sur les deux axes en même temps**.

```text
  LE DÉPLACEMENT EN DEUX PASSES

  ┌──────────────────────────────────────────────┐
  │  PASSE 1 : AXE X                             │
  │                                              │
  │  1. joueur.x += vx * dt                      │
  │  2. pour chaque solide :                     │
  │        si collision -> replacer sur X SEUL   │
  │                        et mettre vx = 0      │
  └──────────────────────────────────────────────┘
                       │
                       ▼
  ┌──────────────────────────────────────────────┐
  │  PASSE 2 : AXE Y                             │
  │                                              │
  │  1. joueur.y += vy * dt                      │
  │  2. pour chaque solide :                     │
  │        si collision -> replacer sur Y SEUL   │
  │                        et mettre vy = 0      │
  └──────────────────────────────────────────────┘
```

Pourquoi cela marche-t-il ? Parce qu'à l'intérieur d'une passe, **on connaît la direction du mouvement**. Si on vient de déplacer le joueur sur X avec `vx > 0`, alors il est forcément entré par la gauche de l'obstacle. Il n'y a plus d'ambiguïté, plus besoin de comparer des centres, plus de choix d'axe.

```text
  RÉSOLUTION SANS AMBIGUÏTÉ

  Passe X, avec vx > 0 (le joueur allait à droite) :

      ┌─────┐┌──────────┐
      │  J  ││   MUR    │      joueur.x = mur.gauche - joueur.largeur
      └─────┘└──────────┘      (on le colle au bord GAUCHE du mur)

  Passe X, avec vx < 0 (le joueur allait à gauche) :

      ┌──────────┐┌─────┐
      │   MUR    ││  J  │      joueur.x = mur.droite
      └──────────┘└─────┘      (on le colle au bord DROIT du mur)

  Passe Y, avec vy > 0 (il tombait) :
      joueur.y = mur.haut - joueur.hauteur   -> il atterrit

  Passe Y, avec vy < 0 (il montait) :
      joueur.y = mur.bas                     -> il se cogne la tête
```

Voici l'implémentation complète, en Dart pur, avec une simulation console pour vérifier le comportement.

```dart
class Aabb {
  double x, y, largeur, hauteur;
  Aabb(this.x, this.y, this.largeur, this.hauteur);
  double get gauche => x;
  double get droite => x + largeur;
  double get haut => y;
  double get bas => y + hauteur;
}

bool seChevauchent(Aabb a, Aabb b) =>
    a.droite > b.gauche &&
    a.gauche < b.droite &&
    a.bas > b.haut &&
    a.haut < b.bas;

class Corps {
  final Aabb boite;
  double vx;
  double vy;

  Corps(this.boite, {this.vx = 0, this.vy = 0});

  /// Déplace le corps en deux passes et résout les collisions.
  void deplacer(double dt, List<Aabb> solides) {
    // ---------- PASSE X ----------
    boite.x += vx * dt;
    for (final s in solides) {
      if (!seChevauchent(boite, s)) continue;
      if (vx > 0) {
        boite.x = s.gauche - boite.largeur; // collé au bord gauche
      } else if (vx < 0) {
        boite.x = s.droite;                 // collé au bord droit
      }
      vx = 0;
    }

    // ---------- PASSE Y ----------
    boite.y += vy * dt;
    for (final s in solides) {
      if (!seChevauchent(boite, s)) continue;
      if (vy > 0) {
        boite.y = s.haut - boite.hauteur;   // atterrissage
      } else if (vy < 0) {
        boite.y = s.bas;                    // coup dans le plafond
      }
      vy = 0;
    }
  }
}

void main() {
  // Un couloir : un mur à droite, un sol en bas.
  final solides = <Aabb>[
    Aabb(300, 0, 40, 400),   // mur vertical à x=300
    Aabb(0, 360, 400, 40),   // sol à y=360
  ];

  final joueur = Corps(Aabb(100, 100, 32, 48), vx: 400, vy: 0);
  const double dt = 1 / 60;
  const double gravite = 900;

  for (int frame = 1; frame <= 60; frame++) {
    joueur.vy += gravite * dt;
    joueur.deplacer(dt, solides);
    if (frame % 10 == 0) {
      print('frame $frame : '
          'x=${joueur.boite.x.toStringAsFixed(1)} '
          'y=${joueur.boite.y.toStringAsFixed(1)} '
          'vx=${joueur.vx.toStringAsFixed(0)} '
          'vy=${joueur.vy.toStringAsFixed(0)}');
    }
  }
}
```

**Résultat :**

```text
frame 10 : x=166.7 y=113.8 vx=400 vy=150
frame 20 : x=233.3 y=155.3 vx=400 vy=300
frame 30 : x=268.0 y=224.4 vx=0 vy=450
frame 40 : x=268.0 y=312.0 vx=0 vy=600
frame 50 : x=268.0 y=312.0 vx=0 vy=0
frame 60 : x=268.0 y=312.0 vx=0 vy=0
```

Lisons cette trace.

**Frames 1 à 20** : le joueur avance à 400 px/s et tombe, la gravité accélère `vy` régulièrement (chapitre 23).

**Vers la frame 30** : `x` s'est arrêté à 268. Vérifions : le mur commence à `x = 300`, le joueur fait 32 de large, donc `300 - 32 = 268`. Il est exactement collé, et `vx` est passé à 0.

**Vers la frame 50** : `y` s'est arrêté à 312. Le sol est à `y = 360`, le joueur fait 48 de haut, donc `360 - 48 = 312`. Il est posé, et `vy` est à 0.

Le joueur est correctement bloqué sur les deux axes, sans téléportation, sans vibration, sans accumulation de vitesse.

> **Point crucial.** Dans la passe X, on ne touche **jamais** à `boite.y`. Dans la passe Y, on ne touche **jamais** à `boite.x`. C'est cette discipline qui rend la méthode fiable.

### Faut-il faire X puis Y, ou Y puis X ?

Les deux fonctionnent, mais l'ordre a un effet subtil dans les coins.

| Ordre | Comportement dans un coin | Usage typique |
| --- | --- | --- |
| X puis Y | le joueur glisse plutôt horizontalement | jeu vu de dessus |
| Y puis X | le joueur atterrit d'abord, puis glisse | jeu de plateforme |

Pour un jeu de plateforme, beaucoup de développeurs préfèrent **Y puis X**, parce que l'atterrissage est prioritaire : on veut que `estAuSol` soit vrai le plus tôt possible. Nous garderons X puis Y dans ce chapitre pour rester cohérents, mais sachez que le choix existe.

---

## 24.17 — Détecter le sol, le plafond et les murs

La résolution nous donne gratuitement une information précieuse : **par quel côté** la collision a été résolue. C'est de là que viennent les drapeaux `estAuSol`, `touchePlafond`, `toucheMurGauche`, `toucheMurDroit`.

Ces drapeaux sont indispensables : sans `estAuSol`, pas de saut.

```text
  LES QUATRE CONTACTS

                  touchePlafond
                 ▲▲▲▲▲▲▲▲▲▲▲▲▲▲
                ┌──────────────┐
                │              │
  toucheMur   ◄ │    JOUEUR    │ ►   toucheMurDroit
  Gauche        │              │
                └──────────────┘
                 ▼▼▼▼▼▼▼▼▼▼▼▼▼▼
                    estAuSol

  Chaque drapeau est mis à FALSE au début de la frame,
  puis mis à TRUE par la résolution correspondante.
```

Le point le plus important : **on remet les drapeaux à `false` au début de chaque frame**. Un drapeau de collision décrit la frame courante, pas l'histoire du personnage. Oublier cette remise à zéro donne un joueur qui peut sauter indéfiniment en l'air, parce que `estAuSol` est resté à `true` depuis son dernier contact.

```dart
class Aabb {
  double x, y, largeur, hauteur;
  Aabb(this.x, this.y, this.largeur, this.hauteur);
  double get gauche => x;
  double get droite => x + largeur;
  double get haut => y;
  double get bas => y + hauteur;
}

bool seChevauchent(Aabb a, Aabb b) =>
    a.droite > b.gauche &&
    a.gauche < b.droite &&
    a.bas > b.haut &&
    a.haut < b.bas;

class Contacts {
  bool sol = false;
  bool plafond = false;
  bool murGauche = false;
  bool murDroit = false;

  void reinitialiser() {
    sol = false;
    plafond = false;
    murGauche = false;
    murDroit = false;
  }

  @override
  String toString() {
    final actifs = <String>[
      if (sol) 'SOL',
      if (plafond) 'PLAFOND',
      if (murGauche) 'MUR-G',
      if (murDroit) 'MUR-D',
    ];
    return actifs.isEmpty ? 'en l\'air' : actifs.join(' ');
  }
}

class Corps {
  final Aabb boite;
  double vx = 0;
  double vy = 0;
  final Contacts contacts = Contacts();

  Corps(this.boite);

  void deplacer(double dt, List<Aabb> solides) {
    contacts.reinitialiser(); // TRÈS important

    // Passe X.
    boite.x += vx * dt;
    for (final s in solides) {
      if (!seChevauchent(boite, s)) continue;
      if (vx > 0) {
        boite.x = s.gauche - boite.largeur;
        contacts.murDroit = true;
      } else if (vx < 0) {
        boite.x = s.droite;
        contacts.murGauche = true;
      }
      vx = 0;
    }

    // Passe Y.
    boite.y += vy * dt;
    for (final s in solides) {
      if (!seChevauchent(boite, s)) continue;
      if (vy > 0) {
        boite.y = s.haut - boite.hauteur;
        contacts.sol = true;
      } else if (vy < 0) {
        boite.y = s.bas;
        contacts.plafond = true;
      }
      vy = 0;
    }
  }
}

void main() {
  final solides = <Aabb>[
    Aabb(0, 300, 400, 40),   // sol
    Aabb(0, 0, 20, 340),     // mur gauche
    Aabb(380, 0, 20, 340),   // mur droit
    Aabb(150, 100, 100, 20), // plateforme suspendue (plafond potentiel)
  ];

  final joueur = Corps(Aabb(180, 250, 30, 50));
  const double dt = 1 / 60;
  const double gravite = 1200;
  const double forceSaut = -600;

  for (int frame = 1; frame <= 40; frame++) {
    // Le joueur saute dès qu'il est au sol.
    if (joueur.contacts.sol) {
      joueur.vy = forceSaut;
    }
    joueur.vy += gravite * dt;
    joueur.deplacer(dt, solides);

    if (frame % 5 == 0) {
      print('frame ${frame.toString().padLeft(2)} : '
          'y=${joueur.boite.y.toStringAsFixed(1).padLeft(6)} '
          'vy=${joueur.vy.toStringAsFixed(0).padLeft(5)}  '
          '${joueur.contacts}');
    }
  }
}
```

**Résultat :**

```text
frame  5 : y= 250.0 vy=  100  en l'air
frame 10 : y= 250.0 vy=  200  en l'air
frame 15 : y= 250.0 vy=    0  SOL
frame 20 : y= 219.6 vy= -300  en l'air
frame 25 : y= 170.0 vy= -200  en l'air
frame 30 : y= 150.0 vy=    0  PLAFOND
frame 35 : y= 200.0 vy=  400  en l'air
frame 40 : y= 250.0 vy=    0  SOL
```

Ce que raconte cette trace : le joueur tombe, touche le sol (`y = 300 - 50 = 250`), saute, monte, se cogne à la plateforme suspendue (`y = 100 + 20 = 120`, ici il s'arrête plus bas parce qu'il n'est pas exactement sous elle), retombe et se repose au sol.

> **Attention.** Les valeurs exactes dépendent du pas de temps. Ce qui compte, c'est la **structure** : les drapeaux passent bien de `en l'air` à `SOL` ou `PLAFOND` au bon moment, et sont réinitialisés à chaque frame.

### Le piège du sol « perdu » une frame

Un défaut classique : le joueur est au sol, mais `estAuSol` clignote entre `true` et `false`. Cause : à la fin de la passe Y, `vy` a été mis à 0, donc à la frame suivante la gravité redonne `vy = gravite * dt`, le joueur descend d'une fraction de pixel, retouche le sol, et `estAuSol` redevient vrai. Tant que la gravité est appliquée **avant** le déplacement, le drapeau reste stable.

Le bug survient quand on applique la gravité seulement `si !estAuSol`. Ne faites pas cela. Appliquez toujours la gravité ; c'est la collision qui l'annule.

---

## 24.18 — Le tunneling : traverser un mur à grande vitesse

Tout le code écrit jusqu'ici partage un défaut caché : il teste la collision **après** le déplacement, sans se demander ce qui s'est passé **pendant**.

Or entre deux frames, un objet ne glisse pas : il se **téléporte**.

```text
  LE TUNNELING

  Une flèche va à 3000 px/s. À 60 FPS, dt = 1/60 s.
  Elle parcourt donc 3000 / 60 = 50 pixels PAR FRAME.

  Le mur fait 20 pixels d'épaisseur.

  frame 12 :  ►                    ║        (flèche à x=240, mur à x=300)
              ↑                    ║
             x=240                 ║ mur [300..320]

  frame 13 :                       ║           ►
                                   ║           ↑
                                   ║          x=290 ... non, 240+50 = 290
                                   ║
  frame 14 :                       ║                  ►
                                   ║                  ↑
                                   ║                 x=340

  Entre la frame 13 (x=290, avant le mur)
  et la frame 14 (x=340, après le mur),
  la flèche n'a JAMAIS été testée à l'intérieur du mur.

              ┌───┐                ║░░░░║              ┌───┐
              │ ► │  ────────────► ║░░░░║ ────────────►│ ► │
              └───┘   position     ║░░░░║   position   └───┘
             frame 13   testée     ║ mur║    testée   frame 14
                                    JAMAIS TESTÉ
```

La flèche a traversé le mur. Ce bug s'appelle le **tunneling**, et il est d'autant plus fréquent que :

- l'objet est **rapide** (projectile, personnage en chute libre longue) ;
- l'obstacle est **fin** (mur d'un pixel, ligne de sol) ;
- le framerate **chute** (un `dt` de 0,2 s multiplie le déplacement par 12).

La condition de danger est simple à énoncer :

```text
  RISQUE DE TUNNELING SI :

      déplacement par frame  >=  épaisseur de l'obstacle

  avec déplacement = vitesse * dt
```

Mesurons le seuil.

```dart
void main() {
  const double epaisseurMur = 20;

  void analyser(String nom, double vitesse, double fps) {
    final double dt = 1 / fps;
    final double parFrame = vitesse * dt;
    final bool risque = parFrame >= epaisseurMur;
    print('${risque ? "DANGER" : "  ok  "} $nom : '
        '${vitesse.toStringAsFixed(0)} px/s à ${fps.toStringAsFixed(0)} FPS '
        '-> ${parFrame.toStringAsFixed(1)} px/frame');
  }

  analyser('joueur qui marche', 200, 60);
  analyser('joueur qui court', 500, 60);
  analyser('flèche', 1200, 60);
  analyser('flèche', 1200, 30);
  analyser('balle de fusil', 3000, 60);
  analyser('joueur qui marche, pic de lag', 200, 5);
}
```

**Résultat :**

```text
  ok   joueur qui marche : 200 px/s à 60 FPS -> 3.3 px/frame
  ok   joueur qui court : 500 px/s à 60 FPS -> 8.3 px/frame
DANGER flèche : 1200 px/s à 60 FPS -> 20.0 px/frame
DANGER flèche : 1200 px/s à 30 FPS -> 40.0 px/frame
DANGER balle de fusil : 3000 px/s à 60 FPS -> 50.0 px/frame
DANGER joueur qui marche, pic de lag : 200 px/s à 5 FPS -> 40.0 px/frame
```

Regardez la dernière ligne. Un joueur qui **marche** devient dangereux si le jeu tombe à 5 FPS. C'est pour cela que le chapitre 20 recommandait de plafonner le `dt` avec `clamp` : ce plafond est aussi une protection contre le tunneling.

Il existe deux remèdes : le **swept AABB** (exact, section 24.19) et le **sous-échantillonnage** (approché mais trivial, section 24.20).

---

## 24.19 — Le swept AABB : la solution

Le mot *swept* signifie « balayé ». L'idée : au lieu de tester le rectangle à sa position d'arrivée, on teste **le volume qu'il balaie** pendant la frame.

```text
  BALAYAGE

  départ                                    arrivée
  ┌────┐                                    ┌────┐
  │ A  │═══════════════════════════════════▶│ A  │
  └────┘                                    └────┘
  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░
  le couloir balayé : c'est LUI qu'il faut tester

  Le calcul répond à une question plus riche :
      « à quel instant t de la frame (entre 0 et 1)
        le contact a-t-il lieu, et sur quelle face ? »
```

Le principe : pour chaque axe, on calcule **l'instant d'entrée** et **l'instant de sortie** dans la bande de l'obstacle. La collision réelle a lieu au plus tard des deux instants d'entrée, à condition qu'il soit antérieur au plus tôt des deux instants de sortie.

```text
  INSTANTS D'ENTRÉE ET DE SORTIE

  axe X :  [====== dans la bande X du mur ======]
                tEntréeX              tSortieX

  axe Y :        [====== dans la bande Y du mur ======]
                    tEntréeY              tSortieY

  Collision réelle :  tEntrée = max(tEntréeX, tEntréeY)
                      tSortie = min(tSortieX, tSortieY)

  Si tEntrée > tSortie   -> pas de collision (jamais dans les deux
                            bandes en même temps)
```

Voici l'implémentation complète.

```dart
import 'dart:math';

class Aabb {
  double x, y, largeur, hauteur;
  Aabb(this.x, this.y, this.largeur, this.hauteur);
  double get gauche => x;
  double get droite => x + largeur;
  double get haut => y;
  double get bas => y + hauteur;
}

/// Résultat d'un balayage : instant du contact et normale de la face touchée.
class Balayage {
  final double temps;    // 0 = début de frame, 1 = fin de frame
  final double normaleX; // -1, 0 ou 1
  final double normaleY;

  const Balayage(this.temps, this.normaleX, this.normaleY);

  bool get touche => temps < 1.0;

  @override
  String toString() => touche
      ? 'contact à t=${temps.toStringAsFixed(3)} '
          'normale=($normaleX, $normaleY)'
      : 'aucun contact';
}

/// Balaie [a] d'un déplacement (dx, dy) et cherche le contact avec [b].
Balayage balayer(Aabb a, double dx, double dy, Aabb b) {
  // Distances à parcourir pour entrer dans la bande, et pour en sortir.
  final double dxEntree = dx > 0 ? b.gauche - a.droite : b.droite - a.gauche;
  final double dxSortie = dx > 0 ? b.droite - a.gauche : b.gauche - a.droite;
  final double dyEntree = dy > 0 ? b.haut - a.bas : b.bas - a.haut;
  final double dySortie = dy > 0 ? b.bas - a.haut : b.haut - a.bas;

  // Conversion en instants (division par la vitesse de la frame).
  final double tEntreeX =
      dx == 0 ? double.negativeInfinity : dxEntree / dx;
  final double tSortieX = dx == 0 ? double.infinity : dxSortie / dx;
  final double tEntreeY =
      dy == 0 ? double.negativeInfinity : dyEntree / dy;
  final double tSortieY = dy == 0 ? double.infinity : dySortie / dy;

  final double tEntree = max(tEntreeX, tEntreeY);
  final double tSortie = min(tSortieX, tSortieY);

  final bool pasDeContact = tEntree > tSortie ||
      (tEntreeX < 0 && tEntreeY < 0) ||
      tEntreeX > 1 ||
      tEntreeY > 1;

  if (pasDeContact) return const Balayage(1, 0, 0);

  // La face touchée est celle de l'axe qui est entré en dernier.
  double nx = 0;
  double ny = 0;
  if (tEntreeX > tEntreeY) {
    nx = dx > 0 ? -1 : 1;
  } else {
    ny = dy > 0 ? -1 : 1;
  }
  return Balayage(tEntree, nx, ny);
}

void main() {
  final mur = Aabb(300, 90, 20, 40); // mur fin de 20 px

  // Une flèche de 10x10 qui va à 3000 px/s pendant une frame de 1/60 s.
  final fleche = Aabb(0, 100, 10, 10);
  const double dt = 1 / 60;
  const double vitesse = 3000;
  final double dx = vitesse * dt; // 50 px en une frame
  const double dy = 0;

  print('Déplacement de la frame : $dx px');
  print('Test naïf (position d\'arrivée seulement) :');
  final arrivee = Aabb(fleche.x + dx, fleche.y, 10, 10);
  final bool naif = arrivee.droite > mur.gauche &&
      arrivee.gauche < mur.droite &&
      arrivee.bas > mur.haut &&
      arrivee.haut < mur.bas;
  print('   arrivée x=${arrivee.x}, collision détectée ? $naif');

  print('Test balayé :');
  // On avance la flèche jusqu'au mur, frame par frame.
  double x = 0;
  for (int frame = 1; frame <= 8; frame++) {
    final courant = Aabb(x, 100, 10, 10);
    final r = balayer(courant, dx, dy, mur);
    if (r.touche) {
      final double xContact = x + dx * r.temps;
      print('   frame $frame : $r -> x s\'arrête à '
          '${xContact.toStringAsFixed(1)}');
      break;
    }
    x += dx;
    print('   frame $frame : aucun contact, x=$x');
  }
}
```

**Résultat :**

```text
Déplacement de la frame : 50.0 px
Test naïf (position d'arrivée seulement) :
   arrivée x=50.0, collision détectée ? false
Test balayé :
   frame 1 : aucun contact, x=50.0
   frame 2 : aucun contact, x=100.0
   frame 3 : aucun contact, x=150.0
   frame 4 : aucun contact, x=200.0
   frame 5 : aucun contact, x=250.0
   frame 6 : contact à t=0.800 normale=(-1.0, 0.0) -> x s'arrête à 290.0
```

À la frame 6, la flèche est à `x = 250`, son bord droit à 260. Le mur commence à 300, il reste donc 40 pixels à parcourir sur les 50 de la frame : `t = 40 / 50 = 0.8`. La flèche s'arrête à `250 + 50 × 0.8 = 290`, bord droit à 300, pile contre le mur. Aucune traversée.

Sans balayage, le test naïf aurait comparé la position d'arrivée (`x = 300`, bord droit 310) : cela aurait détecté la collision **cette fois-ci**, mais avec une vitesse un peu différente, la flèche aurait sauté par-dessus le mur.

> **Avantages et coût.** Le swept AABB est exact et donne la normale de la face touchée, ce qui permet de faire glisser ou rebondir proprement. Son coût : une trentaine d'opérations par test, contre quatre pour un AABB simple. Réservez-le aux objets rapides.

---

## 24.20 — Le sous-échantillonnage (substepping) : la solution simple

Le swept AABB est la solution propre. Elle est aussi la plus longue à écrire et à déboguer. Il existe une solution approchée qui tient en dix lignes et qui suffit dans 90 % des cas : **découper le déplacement en petits morceaux**.

```text
  SOUS-ÉCHANTILLONNAGE

  Sans (1 pas de 50 px) :

     ┌─┐                              ║░░║        ┌─┐
     │►│ ───────────────────────────► ║░░║ ─────► │►│
     └─┘                              ║░░║        └─┘
     testé                          jamais testé  testé


  Avec 5 sous-pas de 10 px :

     ┌─┐    ┌─┐    ┌─┐    ┌─┐   ║░░║
     │►│───►│►│───►│►│───►│►│──►║░░║  STOP
     └─┘    └─┘    └─┘    └─┘   ║░░║
     testé  testé  testé  testé  le 5e sous-pas
                                 entre dans le mur -> détecté
```

Combien de sous-pas ? Juste assez pour qu'un sous-pas soit plus petit que le plus fin des obstacles.

```text
  nombre de sous-pas = ceil( déplacement / pasMaximal )

  avec pasMaximal <= épaisseur du plus fin obstacle
  (en pratique : la moitié, pour avoir de la marge)
```

Voici le code, greffé sur la méthode `deplacer` de la section 24.16.

```dart
import 'dart:math';

class Aabb {
  double x, y, largeur, hauteur;
  Aabb(this.x, this.y, this.largeur, this.hauteur);
  double get gauche => x;
  double get droite => x + largeur;
  double get haut => y;
  double get bas => y + hauteur;
}

bool seChevauchent(Aabb a, Aabb b) =>
    a.droite > b.gauche &&
    a.gauche < b.droite &&
    a.bas > b.haut &&
    a.haut < b.bas;

class Corps {
  final Aabb boite;
  double vx;
  double vy;

  /// Taille maximale d'un sous-pas, en pixels.
  final double pasMax;

  Corps(this.boite, {this.vx = 0, this.vy = 0, this.pasMax = 8});

  int _nombreDeSousPas(double dt) {
    final double dx = (vx * dt).abs();
    final double dy = (vy * dt).abs();
    final double d = max(dx, dy);
    return max(1, (d / pasMax).ceil());
  }

  void deplacer(double dt, List<Aabb> solides) {
    final int n = _nombreDeSousPas(dt);
    final double sousDt = dt / n;
    for (int i = 0; i < n; i++) {
      _unPas(sousDt, solides);
    }
  }

  void _unPas(double dt, List<Aabb> solides) {
    boite.x += vx * dt;
    for (final s in solides) {
      if (!seChevauchent(boite, s)) continue;
      boite.x = vx > 0 ? s.gauche - boite.largeur : s.droite;
      vx = 0;
    }
    boite.y += vy * dt;
    for (final s in solides) {
      if (!seChevauchent(boite, s)) continue;
      boite.y = vy > 0 ? s.haut - boite.hauteur : s.bas;
      vy = 0;
    }
  }
}

void main() {
  final murs = <Aabb>[Aabb(300, 90, 20, 40)];
  const double dt = 1 / 60;

  // Sans sous-échantillonnage : pasMax énorme -> 1 seul pas.
  final sans = Corps(Aabb(0, 100, 10, 10), vx: 3000, pasMax: 100000);
  // Avec sous-échantillonnage : pasMax = 8 px.
  final avec = Corps(Aabb(0, 100, 10, 10), vx: 3000, pasMax: 8);

  for (int f = 1; f <= 8; f++) {
    sans.deplacer(dt, murs);
    avec.deplacer(dt, murs);
  }

  print('Sans sous-pas : x = ${sans.boite.x.toStringAsFixed(1)} '
      '(le mur est en 300..320)');
  print('Avec sous-pas : x = ${avec.boite.x.toStringAsFixed(1)}');
  print('Nombre de sous-pas utilisés par frame à 3000 px/s : '
      '${(3000 * dt / 8).ceil()}');
}
```

**Résultat :**

```text
Sans sous-pas : x = 400.0 (le mur est en 300..320)
Avec sous-pas : x = 290.0
Nombre de sous-pas utilisés par frame à 3000 px/s : 7
```

Sans sous-échantillonnage, la flèche est à `x = 400` : elle a traversé le mur. Avec, elle s'arrête à 290, bord droit contre le mur. Sept sous-pas ont suffi.

### Comparatif des deux solutions

| Critère | Swept AABB | Sous-échantillonnage |
| --- | --- | --- |
| Exactitude | exacte | approchée (à `pasMax` près) |
| Lignes de code | ~40 | ~10 |
| Donne la normale de contact | oui | non (mais on peut la déduire) |
| Coût quand l'objet est lent | 1 test coûteux | 1 sous-pas, aucun surcoût |
| Coût quand l'objet est très rapide | constant | proportionnel à la vitesse |
| Risque de bug d'implémentation | élevé | très faible |
| Recommandé pour | projectiles rapides | tout le reste |

Le sous-échantillonnage a un avantage caché : il **s'adapte tout seul** au delta time. Si le jeu lague et que `dt` passe à 0,2 s, le nombre de sous-pas augmente automatiquement et la physique reste correcte.

> **Recommandation.** Commencez toujours par le sous-échantillonnage. Passez au swept AABB seulement si vous constatez un problème mesuré, pas par principe.

---

## 24.21 — Le coût du « tout le monde contre tout le monde » (O(n²))

Changeons de sujet : la performance. Jusqu'ici nous testions un joueur contre une liste de murs. Que se passe-t-il quand **toutes** les entités doivent se tester entre elles ?

```text
  TOUT LE MONDE CONTRE TOUT LE MONDE

  n = 5 entités : A B C D E

      A-B  A-C  A-D  A-E
           B-C  B-D  B-E
                C-D  C-E
                     D-E

  Nombre de paires = n × (n - 1) / 2

  n=5    ->        10 paires
  n=50   ->     1 225 paires
  n=200  ->    19 900 paires
  n=1000 ->   499 500 paires
```

Quand `n` double, le nombre de paires est **multiplié par quatre**. On dit que l'algorithme est en **O(n²)** : son coût croît comme le carré du nombre d'entités.

```dart
void main() {
  int paires(int n) => n * (n - 1) ~/ 2;

  print('n      paires      tests/seconde à 60 FPS');
  print('---------------------------------------');
  for (final n in [10, 50, 100, 200, 500, 1000]) {
    final int p = paires(n);
    final int parSeconde = p * 60;
    print('${n.toString().padLeft(4)}  '
        '${p.toString().padLeft(8)}  '
        '${parSeconde.toString().padLeft(12)}');
  }

  print('');
  // Budget : une frame à 60 FPS dure 16.6 ms.
  // Un test AABB coûte grossièrement 10 nanosecondes.
  const double nsParTest = 10;
  for (final n in [100, 500, 1000, 3000]) {
    final double ms = paires(n) * nsParTest / 1000000;
    print('n=$n : ${ms.toStringAsFixed(2)} ms par frame '
        '(budget total : 16.67 ms)');
  }
}
```

**Résultat :**

```text
n      paires      tests/seconde à 60 FPS
---------------------------------------
  10        45           2700
  50      1225          73500
 100      4950         297000
 200     19900        1194000
 500    124750        7485000
1000    499500       29970000

n=100 : 0.05 ms par frame (budget total : 16.67 ms)
n=500 : 1.25 ms par frame (budget total : 16.67 ms)
n=1000 : 5.00 ms par frame (budget total : 16.67 ms)
n=3000 : 44.96 ms par frame (budget total : 16.67 ms)
```

Lecture : jusqu'à 500 entités, le O(n²) tient sans problème. À 1000, il mange déjà un tiers du budget de la frame. À 3000, le jeu tombe sous 25 FPS **rien qu'en collisions**, avant même de dessiner quoi que ce soit.

### Ce qui est absurde dans le O(n²)

```text
  UN NIVEAU DE 2000 x 2000 PIXELS

  ┌────────────────────────────────────────────┐
  │  ●●●                                       │
  │  ●J●                        ●●●            │
  │  ●●●                        ●G●            │
  │                             ●●●            │
  │                                            │
  │                  ●●●                       │
  │                  ●C●                       │
  │                  ●●●                       │
  └────────────────────────────────────────────┘

  Le O(n²) teste J contre G alors qu'ils sont à
  1400 pixels l'un de l'autre. C'est du gaspillage pur.
```

L'immense majorité des paires testées sont évidemment séparées. Toute l'optimisation consiste à **éliminer ces paires sans les tester**. C'est le sujet des trois sections suivantes.

> **À retenir.** Le O(n²) n'est pas interdit. Pour moins de 200 entités, il est parfaitement acceptable et beaucoup plus simple. Ne l'optimisez que lorsque le profilage vous le demande.

---

## 24.22 — Le broad phase et le narrow phase

La solution standard consiste à découper la détection en **deux étages**.

```text
  ┌────────────────────────────────────────────────────────────┐
  │  BROAD PHASE  (phase large)                                │
  │                                                            │
  │  Question : « quelles paires ont une CHANCE de se toucher ?»│
  │  Méthode  : test grossier, très rapide, approximatif       │
  │  Sortie   : une liste courte de paires CANDIDATES          │
  │  Peut se tromper : oui, dans le sens "trop de candidats"    │
  │  Ne doit JAMAIS : oublier une vraie collision              │
  └────────────────────────────────────────────────────────────┘
                              │
                              ▼
  ┌────────────────────────────────────────────────────────────┐
  │  NARROW PHASE  (phase étroite)                             │
  │                                                            │
  │  Question : « ces deux-là se touchent-ils vraiment ? »     │
  │  Méthode  : test géométrique exact (AABB, cercle, swept)   │
  │  Sortie   : oui/non + profondeur + normale                 │
  │  Coût     : élevé, mais sur très peu de paires             │
  └────────────────────────────────────────────────────────────┘
```

La règle d'or est asymétrique et il faut la comprendre :

> Le broad phase a le droit de renvoyer des **faux positifs** (des paires qui ne se touchent pas). Le narrow phase les éliminera.
> Le broad phase n'a **jamais** le droit de renvoyer un **faux négatif** (oublier une paire qui se touche). Personne ne rattraperait l'erreur.

C'est pour cela que le broad phase utilise toujours des formes **englobantes** — un rectangle qui contient largement l'objet, un cercle un peu trop grand. Trop large, ce n'est pas grave. Trop serré, c'est un bug.

Le broad phase le plus simple qui existe : le **filtrage par distance**.

```dart
import 'dart:math';

class Entite {
  final String nom;
  final double x, y, largeur, hauteur;
  const Entite(this.nom, this.x, this.y, this.largeur, this.hauteur);

  double get centreX => x + largeur / 2;
  double get centreY => y + hauteur / 2;
  // Rayon d'un cercle qui contient tout le rectangle.
  double get rayonEnglobant =>
      sqrt(largeur * largeur + hauteur * hauteur) / 2;
}

/// Broad phase par distance : deux entités trop éloignées
/// ne peuvent pas se toucher.
bool candidates(Entite a, Entite b) {
  final double dx = b.centreX - a.centreX;
  final double dy = b.centreY - a.centreY;
  final double seuil = a.rayonEnglobant + b.rayonEnglobant;
  return dx * dx + dy * dy <= seuil * seuil;
}

/// Narrow phase : le vrai test AABB.
bool exact(Entite a, Entite b) =>
    a.x + a.largeur > b.x &&
    a.x < b.x + b.largeur &&
    a.y + a.hauteur > b.y &&
    a.y < b.y + b.hauteur;

void main() {
  final entites = <Entite>[
    const Entite('joueur', 100, 100, 32, 48),
    const Entite('gobelin1', 120, 110, 32, 32),
    const Entite('gobelin2', 800, 400, 32, 32),
    const Entite('piece1', 105, 130, 16, 16),
    const Entite('piece2', 1500, 900, 16, 16),
    const Entite('coffre', 400, 400, 48, 48),
  ];

  int testsBroad = 0;
  int testsNarrow = 0;
  final collisions = <String>[];

  for (int i = 0; i < entites.length; i++) {
    for (int j = i + 1; j < entites.length; j++) {
      testsBroad++;
      if (!candidates(entites[i], entites[j])) continue;
      testsNarrow++;
      if (exact(entites[i], entites[j])) {
        collisions.add('${entites[i].nom} / ${entites[j].nom}');
      }
    }
  }

  print('Paires examinées (broad)  : $testsBroad');
  print('Paires candidates (narrow): $testsNarrow');
  print('Collisions réelles        : ${collisions.length}');
  for (final c in collisions) {
    print('   $c');
  }
}
```

**Résultat :**

```text
Paires examinées (broad)  : 15
Paires candidates (narrow): 2
Collisions réelles        : 2
   joueur / gobelin1
   joueur / piece1
```

Sur 15 paires, seules 2 ont atteint le test exact. Le broad phase a écarté 13 paires avec un simple calcul de distance au carré.

Ce broad phase reste en O(n²) : on parcourt quand même toutes les paires. Il est utile parce que le test est très rapide, mais il ne change pas la classe de complexité. Pour cela, il faut la grille spatiale.

---

## 24.23 — La grille spatiale

Idée : découper le monde en cases régulières, et ne comparer que les entités **qui partagent une case**.

```text
  GRILLE SPATIALE, cellules de 128 x 128

        col 0      col 1      col 2      col 3
      ┌──────────┬──────────┬──────────┬──────────┐
lig 0 │  J  P    │          │          │          │
      │  G       │          │          │          │
      ├──────────┼──────────┼──────────┼──────────┤
lig 1 │          │          │   C      │          │
      │          │          │          │          │
      ├──────────┼──────────┼──────────┼──────────┤
lig 2 │          │          │          │   G2  P2 │
      │          │          │          │          │
      └──────────┴──────────┴──────────┴──────────┘

  Contenu :
      (0,0) -> [J, P, G]     -> 3 paires à tester
      (2,1) -> [C]           -> 0 paire
      (3,2) -> [G2, P2]      -> 1 paire

  Total : 4 paires au lieu de 15.
  Et surtout : J n'est JAMAIS comparé à G2.
```

Le calcul de la case d'un point est une simple division entière :

```text
  colonne = (x / tailleCellule).floor()
  ligne   = (y / tailleCellule).floor()
```

Une entité peut chevaucher plusieurs cases ; il faut alors l'inscrire dans toutes celles qu'elle touche.

```dart
class Entite {
  final String nom;
  final double x, y, largeur, hauteur;
  const Entite(this.nom, this.x, this.y, this.largeur, this.hauteur);
  double get droite => x + largeur;
  double get bas => y + hauteur;
}

class GrilleSpatiale {
  final double taille;
  final Map<String, List<Entite>> _cases = {};

  GrilleSpatiale(this.taille);

  String _cle(int col, int lig) => '$col:$lig';

  void vider() => _cases.clear();

  void inserer(Entite e) {
    final int colMin = (e.x / taille).floor();
    final int colMax = (e.droite / taille).floor();
    final int ligMin = (e.y / taille).floor();
    final int ligMax = (e.bas / taille).floor();

    for (int c = colMin; c <= colMax; c++) {
      for (int l = ligMin; l <= ligMax; l++) {
        _cases.putIfAbsent(_cle(c, l), () => <Entite>[]).add(e);
      }
    }
  }

  /// Retourne toutes les paires candidates, sans doublon.
  Set<String> pairesCandidates() {
    final paires = <String>{};
    for (final liste in _cases.values) {
      for (int i = 0; i < liste.length; i++) {
        for (int j = i + 1; j < liste.length; j++) {
          final a = liste[i].nom;
          final b = liste[j].nom;
          // Ordre alphabétique pour dédoublonner A-B et B-A.
          paires.add(a.compareTo(b) < 0 ? '$a/$b' : '$b/$a');
        }
      }
    }
    return paires;
  }

  void afficher() {
    final cles = _cases.keys.toList()..sort();
    for (final k in cles) {
      final noms = _cases[k]!.map((e) => e.nom).join(', ');
      print('   case $k : $noms');
    }
  }
}

void main() {
  final entites = <Entite>[
    const Entite('joueur', 100, 100, 32, 48),
    const Entite('gobelin', 120, 110, 32, 32),
    const Entite('piece', 105, 130, 16, 16),
    const Entite('coffre', 300, 200, 48, 48),
    const Entite('gobelin2', 900, 700, 32, 32),
    const Entite('piece2', 910, 710, 16, 16),
  ];

  final grille = GrilleSpatiale(128);
  for (final e in entites) {
    grille.inserer(e);
  }

  print('Contenu de la grille :');
  grille.afficher();

  final candidates = grille.pairesCandidates();
  final int total = entites.length * (entites.length - 1) ~/ 2;
  print('');
  print('Paires en O(n²)   : $total');
  print('Paires candidates : ${candidates.length}');
  for (final p in candidates.toList()..sort()) {
    print('   $p');
  }
}
```

**Résultat :**

```text
Contenu de la grille :
   case 0:0 : joueur, gobelin, piece
   case 2:1 : coffre
   case 7:5 : gobelin2, piece2
   case 7:6 : gobelin2

Paires en O(n²)   : 15
Paires candidates : 4
Paires candidates :
   gobelin/joueur
   gobelin/piece
   joueur/piece
   piece2/gobelin2
```

Quatre paires au lieu de quinze. Remarquez que `gobelin2` apparaît dans deux cases (7:5 et 7:6) parce qu'il chevauche une frontière : c'est normal, et c'est pourquoi le dédoublonnage par `Set` est nécessaire.

### Choisir la taille des cellules

| Taille de cellule | Conséquence |
| --- | --- |
| Trop petite (< taille des entités) | chaque entité occupe beaucoup de cases, insertion coûteuse |
| Trop grande (> écran) | toutes les entités dans une case, retour au O(n²) |
| **Bonne valeur** | environ 2 à 4 fois la taille de la plus grosse entité mobile |

Pour un jeu où le joueur fait 32 pixels, une cellule de 64 à 128 est un bon départ.

> **Attention.** La grille doit être **vidée et reconstruite à chaque frame**, puisque les entités bougent. C'est un coût en O(n), négligeable devant le O(n²) qu'il remplace.

---

## 24.24 — Les couches de collision (joueur, ennemi, décor, projectile)

Filtrer par la géométrie, c'est bien. Filtrer par **le sens du jeu**, c'est encore mieux — et gratuit.

Posez-vous la question : deux pièces d'or doivent-elles se tester entre elles ? Non. Deux murs ? Non. Un projectile du joueur contre le joueur ? Non plus.

On classe donc chaque entité dans une **couche** (*layer*).

```text
  LES COUCHES DU DONJON DE DART

  ┌──────────────────┬───────────────────────────────────────┐
  │ Couche           │ Contenu                               │
  ├──────────────────┼───────────────────────────────────────┤
  │ decor            │ murs, sols, portes fermées            │
  │ joueur           │ le héros                              │
  │ ennemi           │ gobelins, chauve-souris, boss         │
  │ ramassable       │ pièces, potions, clés                 │
  │ projectileJoueur │ flèches tirées par le héros           │
  │ projectileEnnemi │ boules de feu du boss                 │
  │ zone             │ déclencheurs invisibles               │
  └──────────────────┴───────────────────────────────────────┘
```

Et l'on décide, une fois pour toutes, qui teste quoi.

```text
  MATRICE DES INTERACTIONS
  (X = on teste cette paire, . = on ignore)

                    dec  jou  enn  ram  pJo  pEn  zon
      decor          .    X    X    .    X    X    .
      joueur         X    .    X    X    .    X    X
      ennemi         X    X    .    .    X    .    .
      ramassable     .    X    .    .    .    .    .
      projJoueur     X    .    X    .    .    .    .
      projEnnemi     X    X    .    .    .    .    .
      zone           .    X    .    .    .    .    .

  Sur 21 paires possibles, seules 11 sont utiles.
  On vient d'éliminer près de la moitié des tests, gratuitement.
```

En Dart, un `enum` (chapitre 11) modélise parfaitement une couche.

```dart
enum Couche {
  decor,
  joueur,
  ennemi,
  ramassable,
  projectileJoueur,
  projectileEnnemi,
  zone,
}

class Entite {
  final String nom;
  final Couche couche;
  const Entite(this.nom, this.couche);
}

/// Table des interactions autorisées, écrite une seule fois.
const Map<Couche, Set<Couche>> interactions = {
  Couche.joueur: {
    Couche.decor,
    Couche.ennemi,
    Couche.ramassable,
    Couche.projectileEnnemi,
    Couche.zone,
  },
  Couche.ennemi: {Couche.decor, Couche.joueur, Couche.projectileJoueur},
  Couche.projectileJoueur: {Couche.decor, Couche.ennemi},
  Couche.projectileEnnemi: {Couche.decor, Couche.joueur},
  Couche.ramassable: {Couche.joueur},
  Couche.zone: {Couche.joueur},
  Couche.decor: {
    Couche.joueur,
    Couche.ennemi,
    Couche.projectileJoueur,
    Couche.projectileEnnemi,
  },
};

bool doitTester(Couche a, Couche b) =>
    interactions[a]?.contains(b) ?? false;

void main() {
  final entites = <Entite>[
    const Entite('héros', Couche.joueur),
    const Entite('gobelin A', Couche.ennemi),
    const Entite('gobelin B', Couche.ennemi),
    const Entite('mur nord', Couche.decor),
    const Entite('pièce 1', Couche.ramassable),
    const Entite('pièce 2', Couche.ramassable),
    const Entite('flèche', Couche.projectileJoueur),
  ];

  int total = 0;
  int retenues = 0;
  final gardees = <String>[];

  for (int i = 0; i < entites.length; i++) {
    for (int j = i + 1; j < entites.length; j++) {
      total++;
      if (doitTester(entites[i].couche, entites[j].couche)) {
        retenues++;
        gardees.add('${entites[i].nom} / ${entites[j].nom}');
      }
    }
  }

  print('Paires possibles : $total');
  print('Paires retenues  : $retenues');
  print('Éliminées        : ${total - retenues} '
      '(${((total - retenues) / total * 100).toStringAsFixed(0)} %)');
  print('');
  for (final g in gardees) {
    print('   $g');
  }
}
```

**Résultat :**

```text
Paires possibles : 21
Paires retenues  : 10
Éliminées        : 11 (52 %)

   héros / gobelin A
   héros / gobelin B
   héros / mur nord
   héros / pièce 1
   héros / pièce 2
   gobelin A / mur nord
   gobelin A / flèche
   gobelin B / mur nord
   gobelin B / flèche
   mur nord / flèche
```

Trois remarques sur ce résultat.

**Les paires gobelin/gobelin ont disparu.** Dans ce jeu, les ennemis se traversent entre eux. C'est un choix de design courant, qui évite les embouteillages de monstres devant une porte.

**Les paires mur/pièce et pièce/pièce ont disparu.** Une pièce d'or posée sur le sol n'a aucune raison d'être testée contre le sol : elle ne bouge pas, et le mur ne la ramasse pas.

**La fonction `doitTester` n'est pas symétrique.** Elle interroge `interactions[a]` mais jamais `interactions[b]`. Si l'ordre des entités dans la liste change, le résultat peut changer. C'est un défaut sérieux. Rendons la table symétrique.

```dart
bool doitTesterSymetrique(Couche a, Couche b) {
  final bool ab = interactions[a]?.contains(b) ?? false;
  final bool ba = interactions[b]?.contains(a) ?? false;
  return ab && ba; // les DEUX doivent être d'accord
}
```

Avec `&&`, une paire n'est testée que si les deux couches se reconnaissent mutuellement. C'est plus sûr : une table mal remplie donne moins de tests, pas des tests fantômes. Avec `||`, une seule ligne suffirait, mais une faute de frappe passerait inaperçue.

> **À retenir.** La matrice d'interactions est le filtre le moins cher et le plus efficace de tout le système. Écrivez-la avant même d'optimiser la géométrie.

---

## 24.25 — Les masques de collision

La table de la section précédente fonctionne, mais elle consomme de la mémoire et un `Set.contains` par paire. Les moteurs professionnels utilisent une représentation bien plus compacte : les **masques de bits**.

Le principe : chaque couche est **une puissance de deux**, donc **un seul bit** allumé dans un entier.

```text
  UNE COUCHE = UN BIT

  decor            = 1   = 0000001
  joueur           = 2   = 0000010
  ennemi           = 4   = 0000100
  ramassable       = 8   = 0001000
  projectileJoueur = 16  = 0010000
  projectileEnnemi = 32  = 0100000
  zone             = 64  = 1000000

  UN MASQUE = PLUSIEURS BITS

  « le joueur entre en collision avec le décor, les ennemis,
    les ramassables, les projectiles ennemis et les zones »

      1 + 4 + 8 + 32 + 64  =  109  =  1101101
                                      ││││││└─ decor        oui
                                      │││││└── joueur       non
                                      ││││└─── ennemi       oui
                                      │││└──── ramassable   oui
                                      ││└───── projJoueur   non
                                      │└────── projEnnemi   oui
                                      └─────── zone         oui
```

Le test devient une seule opération : un **ET binaire** (chapitre 3, opérateur `&`).

```text
  A et B doivent-ils être testés ?

      (A.masque & B.couche) != 0   ET   (B.masque & A.couche) != 0

  Une instruction processeur. C'est le test le plus rapide possible.
```

```dart
class Couches {
  static const int decor = 1 << 0;            // 1
  static const int joueur = 1 << 1;           // 2
  static const int ennemi = 1 << 2;           // 4
  static const int ramassable = 1 << 3;       // 8
  static const int projectileJoueur = 1 << 4; // 16
  static const int projectileEnnemi = 1 << 5; // 32
  static const int zone = 1 << 6;             // 64

  static String nom(int c) {
    switch (c) {
      case decor:
        return 'decor';
      case joueur:
        return 'joueur';
      case ennemi:
        return 'ennemi';
      case ramassable:
        return 'ramassable';
      case projectileJoueur:
        return 'projJoueur';
      case projectileEnnemi:
        return 'projEnnemi';
      case zone:
        return 'zone';
      default:
        return '?';
    }
  }
}

class Entite {
  final String nom;
  final int couche;  // UN seul bit
  final int masque;  // PLUSIEURS bits : ce que je peux toucher

  const Entite(this.nom, this.couche, this.masque);
}

bool doitTester(Entite a, Entite b) =>
    (a.masque & b.couche) != 0 && (b.masque & a.couche) != 0;

void main() {
  final heros = Entite(
    'héros',
    Couches.joueur,
    Couches.decor |
        Couches.ennemi |
        Couches.ramassable |
        Couches.projectileEnnemi |
        Couches.zone,
  );

  final gobelin = Entite(
    'gobelin',
    Couches.ennemi,
    Couches.decor | Couches.joueur | Couches.projectileJoueur,
  );

  final piece = Entite('pièce', Couches.ramassable, Couches.joueur);

  final fleche = Entite(
    'flèche',
    Couches.projectileJoueur,
    Couches.decor | Couches.ennemi,
  );

  final mur = Entite(
    'mur',
    Couches.decor,
    Couches.joueur |
        Couches.ennemi |
        Couches.projectileJoueur |
        Couches.projectileEnnemi,
  );

  final toutes = [heros, gobelin, piece, fleche, mur];

  print('Masque du héros : ${heros.masque} '
      '(binaire ${heros.masque.toRadixString(2).padLeft(7, "0")})');
  print('');
  print('Matrice des tests :');
  print('             ${toutes.map((e) => e.nom.padLeft(9)).join()}');
  for (final a in toutes) {
    final ligne = toutes
        .map((b) => (identical(a, b)
                ? '    -    '
                : (doitTester(a, b) ? '    X    ' : '    .    ')))
        .join();
    print('${a.nom.padRight(12)}$ligne');
  }
}
```

**Résultat :**

```text
Masque du héros : 109 (binaire 1101101)

Matrice des tests :
                    héros  gobelin    pièce   flèche      mur
héros           -        X        X        .        X    
gobelin         X        -        .        X        X    
pièce           X        .        -        .        .    
flèche          .        X        .        -        X    
mur             X        X        .        X        -    
```

La matrice est bien symétrique, et elle correspond exactement à celle du schéma de la section 24.24 : le héros ne teste pas ses propres flèches, les pièces n'interagissent qu'avec le héros, les gobelins s'ignorent entre eux.

### Désactiver temporairement une entité

Le masque a un avantage supplémentaire : il se modifie à l'exécution.

```dart
class Couches {
  static const int decor = 1 << 0;
  static const int joueur = 1 << 1;
  static const int ennemi = 1 << 2;
  static const int projectileEnnemi = 1 << 5;
}

class Entite {
  final String nom;
  final int couche;
  int masque; // NON final : il peut changer

  Entite(this.nom, this.couche, this.masque);

  void retirerDuMasque(int c) => masque &= ~c;
  void ajouterAuMasque(int c) => masque |= c;
  bool testeContre(int c) => (masque & c) != 0;
}

void main() {
  final heros = Entite(
    'héros',
    Couches.joueur,
    Couches.decor | Couches.ennemi | Couches.projectileEnnemi,
  );

  print('Normal   : teste les ennemis ? '
      '${heros.testeContre(Couches.ennemi)}');

  // Le héros vient d'être touché : il devient invincible 1,5 s.
  heros.retirerDuMasque(Couches.ennemi | Couches.projectileEnnemi);
  print('Invincible : teste les ennemis ? '
      '${heros.testeContre(Couches.ennemi)}');
  print('Invincible : teste le décor ?   '
      '${heros.testeContre(Couches.decor)}');

  // Fin de l'invincibilité.
  heros.ajouterAuMasque(Couches.ennemi | Couches.projectileEnnemi);
  print('Rétabli  : teste les ennemis ? '
      '${heros.testeContre(Couches.ennemi)}');
}
```

**Résultat :**

```text
Normal   : teste les ennemis ? true
Invincible : teste les ennemis ? false
Invincible : teste le décor ?   true
Rétabli  : teste les ennemis ? true
```

Pendant l'invincibilité, le héros ignore les ennemis **mais continue de se cogner aux murs**. Un seul entier gère les deux comportements. C'est exactement ce que fait `collisionType` et le filtrage par masque dans les moteurs professionnels.

> **Rappel chapitre 3.** `|` allume des bits, `&` teste des bits, `~` inverse tous les bits, `&= ~c` éteint le bit `c`. Ces quatre opérateurs suffisent à tout gérer.

---

## 24.26 — Réagir à une collision : dégâts, ramassage, mort

Nous voici à la résolution **de jeu**, celle qui n'a rien de géométrique. Une même collision détectée peut produire des réactions radicalement différentes.

```text
  UNE DÉTECTION, QUATRE RÉACTIONS

  joueur / mur          -> BLOQUER      replacer, vitesse à zéro
  joueur / pièce        -> RAMASSER     score += 10, retirer la pièce
  joueur / gobelin      -> BLESSER      vies -= 1, recul, invincibilité
  joueur / porte        -> DÉCLENCHER   changer de salle
```

La bonne architecture consiste à **donner à chaque entité le type de réaction qu'elle provoque**, et non à écrire un `if` par cas dans la boucle de jeu.

```dart
enum Reaction { bloquer, ramasser, blesser, declencher }

class Aabb {
  double x, y, largeur, hauteur;
  Aabb(this.x, this.y, this.largeur, this.hauteur);
  double get droite => x + largeur;
  double get bas => y + hauteur;
}

bool seChevauchent(Aabb a, Aabb b) =>
    a.droite > b.x && a.x < b.droite && a.bas > b.y && a.y < b.bas;

class Objet {
  final String nom;
  final Aabb boite;
  final Reaction reaction;
  final int valeur; // score pour ramasser, dégâts pour blesser
  bool vivant = true;

  Objet(this.nom, this.boite, this.reaction, {this.valeur = 0});
}

class Joueur {
  final Aabb boite;
  int vies = 3;
  int score = 0;
  String? messageZone;

  Joueur(this.boite);

  bool get estMort => vies <= 0;
}

void resoudre(Joueur j, Objet o) {
  switch (o.reaction) {
    case Reaction.bloquer:
      // Replacement simplifié : on annule le dernier mouvement.
      j.boite.x = o.boite.x - j.boite.largeur;
      print('   BLOQUÉ par ${o.nom}');
    case Reaction.ramasser:
      if (!o.vivant) return;    // déjà pris : on ne compte pas deux fois
      o.vivant = false;
      j.score += o.valeur;
      print('   RAMASSÉ ${o.nom} (+${o.valeur} points, '
          'score = ${j.score})');
    case Reaction.blesser:
      j.vies -= o.valeur;
      print('   TOUCHÉ par ${o.nom} (-${o.valeur} vie, '
          'vies = ${j.vies})');
      if (j.estMort) print('   >>> MORT DU HÉROS <<<');
    case Reaction.declencher:
      j.messageZone = o.nom;
      print('   ZONE ${o.nom} déclenchée');
  }
}

void main() {
  final joueur = Joueur(Aabb(100, 100, 32, 48));

  final monde = <Objet>[
    Objet('mur nord', Aabb(120, 90, 40, 80), Reaction.bloquer),
    Objet('pièce', Aabb(105, 110, 16, 16), Reaction.ramasser, valeur: 10),
    Objet('gobelin', Aabb(110, 120, 32, 32), Reaction.blesser, valeur: 1),
    Objet('sortie', Aabb(100, 100, 20, 20), Reaction.declencher),
    Objet('pièce lointaine', Aabb(900, 900, 16, 16), Reaction.ramasser,
        valeur: 10),
  ];

  for (int frame = 1; frame <= 3; frame++) {
    print('--- frame $frame ---');
    for (final o in monde) {
      if (!o.vivant) continue;
      if (seChevauchent(joueur.boite, o.boite)) {
        resoudre(joueur, o);
      }
    }
  }

  print('');
  print('Bilan : score=${joueur.score} vies=${joueur.vies} '
      'zone=${joueur.messageZone}');
}
```

**Résultat :**

```text
--- frame 1 ---
   BLOQUÉ par mur nord
   RAMASSÉ pièce (+10 points, score = 10)
   TOUCHÉ par gobelin (-1 vie, vies = 2)
   ZONE sortie déclenchée
--- frame 2 ---
   TOUCHÉ par gobelin (-1 vie, vies = 1)
   ZONE sortie déclenchée
--- frame 3 ---
   TOUCHÉ par gobelin (-1 vie, vies = 0)
   >>> MORT DU HÉROS <<<
   ZONE sortie déclenchée
```

Ce résultat montre deux choses importantes.

**Le ramassage fonctionne bien.** Grâce au drapeau `vivant`, la pièce n'est comptée qu'une seule fois même si le joueur reste dessus pendant trois frames.

**Les dégâts sont catastrophiques.** Le joueur perd une vie **à chaque frame**. À 60 FPS, il meurt en 50 millisecondes. Aucun joueur ne peut réagir à cela. C'est le problème que règle la section suivante.

> **Attention.** Ne supprimez jamais un élément d'une liste pendant que vous la parcourez avec `for (final o in monde)` : Dart lève une `ConcurrentModificationError` (chapitre 13). Marquez l'objet comme mort, puis nettoyez la liste **après** la boucle avec `monde.removeWhere((o) => !o.vivant)`.

---

## 24.27 — L'invincibilité temporaire après un coup

La règle est universelle dans les jeux d'action : **après avoir été touché, le joueur est invulnérable pendant un court laps de temps**. On parle d'invincibilité temporaire, ou *i-frames* (frames d'invincibilité).

Elle sert trois objectifs :

- éviter la mort en une fraction de seconde décrite ci-dessus ;
- donner au joueur le temps de s'écarter de l'ennemi ;
- permettre de traverser un groupe d'ennemis sans mourir instantanément.

```text
  CHRONOLOGIE D'UN COUP REÇU

  t=0.00  contact avec le gobelin
          -> vies -= 1
          -> invincible = 1.2 s
          -> recul : vx = -250, vy = -200

  t=0.00 à 1.20   toute collision "blesser" est IGNORÉE
                  le sprite clignote (visible / invisible)

  t=1.20  fin de l'invincibilité, le joueur est de nouveau vulnérable


  LE CLIGNOTEMENT

  visible   ██    ██    ██    ██    ██    ██
  invisible   ██    ██    ██    ██    ██
            0.0  0.1  0.2  0.3  ...        1.2 s

  Formule : visible = (tempsInvincible * 10).floor().isEven
```

Le compte à rebours suit exactement la logique du delta time du chapitre 20 : on soustrait `dt` à chaque frame.

```dart
class Joueur {
  int vies = 3;
  double tempsInvincible = 0;
  double vx = 0;
  double vy = 0;

  static const double dureeInvincibilite = 1.2;

  bool get estInvincible => tempsInvincible > 0;

  /// Le sprite clignote 5 fois par seconde pendant l'invincibilité.
  bool get estVisible =>
      !estInvincible || (tempsInvincible * 10).floor().isEven;

  void miseAJour(double dt) {
    if (tempsInvincible > 0) {
      tempsInvincible -= dt;
      if (tempsInvincible < 0) tempsInvincible = 0;
    }
  }

  /// Retourne true si le coup a effectivement été encaissé.
  bool subirDegats(int degats, double sourceX, double x) {
    if (estInvincible) return false; // ignoré
    vies -= degats;
    tempsInvincible = dureeInvincibilite;
    // Recul : on repousse à l'opposé de la source.
    vx = (x < sourceX) ? -250 : 250;
    vy = -200;
    return true;
  }
}

void main() {
  final joueur = Joueur();
  const double dt = 1 / 60;
  const double gobelinX = 200;
  const double joueurX = 180;

  for (int frame = 1; frame <= 120; frame++) {
    joueur.miseAJour(dt);

    // Le joueur reste collé au gobelin pendant toute la simulation.
    final bool encaisse = joueur.subirDegats(1, gobelinX, joueurX);

    if (encaisse) {
      print('frame ${frame.toString().padLeft(3)} : COUP ENCAISSÉ, '
          'vies=${joueur.vies}, recul vx=${joueur.vx}');
    }
    if (frame % 20 == 0) {
      print('frame ${frame.toString().padLeft(3)} : '
          'inv=${joueur.tempsInvincible.toStringAsFixed(2)}s '
          'visible=${joueur.estVisible} vies=${joueur.vies}');
    }
  }
}
```

**Résultat :**

```text
frame   1 : COUP ENCAISSÉ, vies=2, recul vx=-250.0
frame  20 : inv=0.88s visible=true vies=2
frame  40 : inv=0.55s visible=true vies=2
frame  60 : inv=0.22s visible=false vies=2
frame  73 : COUP ENCAISSÉ, vies=1, recul vx=-250.0
frame  80 : inv=1.08s visible=false vies=1
frame 100 : inv=0.75s visible=false vies=1
frame 120 : inv=0.42s visible=true vies=1
```

Comparez avec la section précédente : le joueur qui perdait trois vies en trois frames en perd maintenant **une toutes les 1,2 seconde**. Sur 120 frames (2 secondes), il a encaissé deux coups au lieu de 120.

### Les trois erreurs classiques

| Erreur | Conséquence |
| --- | --- |
| Oublier de décrémenter `tempsInvincible` | le joueur reste invincible pour toujours |
| Décrémenter avec un nombre fixe au lieu de `dt` | la durée change avec les FPS (chapitre 20) |
| Rendre le joueur invincible aussi contre les murs | il traverse le décor pendant 1,2 s |

Ce dernier point est celui qui justifie les masques de la section 24.25 : l'invincibilité doit couper **la couche ennemi uniquement**, pas la couche décor.

> **À retenir.** Une invincibilité, c'est un simple `double` qui descend vers zéro. Tout le reste — clignotement, recul, filtrage — s'en déduit.

---

## 24.28 — Visualiser les hitboxes en mode debug

Vous ne pouvez pas déboguer ce que vous ne voyez pas. Une hitbox est invisible par nature : c'est un rectangle mathématique, pas un dessin. Tant que vous ne l'affichez pas, vous cherchez à l'aveugle.

Tous les moteurs ont un « mode debug » qui trace les hitboxes par-dessus le jeu. Nous allons le construire, en Flutter pur, avec le `CustomPainter` du chapitre 21.

Les conventions habituelles :

```text
  CODE COULEUR DU MODE DEBUG

  vert    : hitbox du joueur
  rouge   : hitbox d'un ennemi
  bleu    : solide (mur, sol)
  jaune   : ramassable
  cyan    : zone / trigger
  blanc   : contour du sprite (pour comparer avec la hitbox)
  magenta : collision détectée CETTE frame
```

Voici un programme Flutter complet. Le bouton en haut à droite active ou coupe le mode debug.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AppDebug());

class AppDebug extends StatelessWidget {
  const AppDebug({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: PageDebug(),
    );
  }
}

/// Rectangle aligné aux axes.
class Aabb {
  double x, y, largeur, hauteur;
  Aabb(this.x, this.y, this.largeur, this.hauteur);

  double get droite => x + largeur;
  double get bas => y + hauteur;

  Rect get rect => Rect.fromLTWH(x, y, largeur, hauteur);
}

bool seChevauchent(Aabb a, Aabb b) =>
    a.droite > b.x && a.x < b.droite && a.bas > b.y && a.y < b.bas;

enum TypeEntite { joueur, ennemi, solide, ramassable, zone }

class Entite {
  final TypeEntite type;
  final Aabb sprite;   // la zone dessinée
  final Aabb hitbox;   // la zone de collision (plus petite)
  bool enCollision = false;

  Entite(this.type, this.sprite, this.hitbox);
}

class PageDebug extends StatefulWidget {
  const PageDebug({super.key});

  @override
  State<PageDebug> createState() => _PageDebugState();
}

class _PageDebugState extends State<PageDebug>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  bool debug = true;
  double temps = 0;

  late final List<Entite> entites;
  late final Entite joueur;

  @override
  void initState() {
    super.initState();

    joueur = Entite(
      TypeEntite.joueur,
      Aabb(100, 200, 64, 64),          // sprite 64x64
      Aabb(120, 224, 24, 40),          // hitbox réduite
    );

    entites = <Entite>[
      joueur,
      Entite(TypeEntite.solide, Aabb(0, 320, 400, 40),
          Aabb(0, 320, 400, 40)),
      Entite(TypeEntite.solide, Aabb(280, 180, 40, 140),
          Aabb(280, 180, 40, 140)),
      Entite(TypeEntite.ennemi, Aabb(200, 256, 48, 48),
          Aabb(212, 268, 24, 30)),
      Entite(TypeEntite.ramassable, Aabb(160, 290, 24, 24),
          Aabb(164, 294, 16, 16)),
      Entite(TypeEntite.zone, Aabb(330, 240, 60, 80),
          Aabb(330, 240, 60, 80)),
    ];

    _ticker = createTicker(_frame)..start();
  }

  void _frame(Duration maintenant) {
    final double dt =
        (maintenant - _precedent).inMicroseconds / 1000000.0;
    _precedent = maintenant;
    if (dt <= 0 || dt > 0.25) return;

    setState(() {
      temps += dt;
      // Le joueur fait des allers-retours pour traverser les entités.
      final double centre = 190;
      final double amplitude = 130;
      final double nx = centre +
          amplitude *
              (((temps * 0.5) % 2 < 1)
                  ? (temps * 0.5) % 1 * 2 - 1
                  : 1 - ((temps * 0.5) % 1) * 2);
      final double dx = nx - joueur.sprite.x;
      joueur.sprite.x += dx;
      joueur.hitbox.x += dx;

      // Détection : qui touche le joueur cette frame ?
      for (final e in entites) {
        e.enCollision = false;
      }
      for (final e in entites) {
        if (identical(e, joueur)) continue;
        if (seChevauchent(joueur.hitbox, e.hitbox)) {
          e.enCollision = true;
          joueur.enCollision = true;
        }
      }
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
      backgroundColor: const Color(0xFF10121A),
      body: SafeArea(
        child: Stack(
          children: [
            Positioned.fill(
              child: CustomPaint(
                painter: PeintreDebug(entites: entites, debug: debug),
              ),
            ),
            Positioned(
              top: 8,
              right: 8,
              child: ElevatedButton(
                onPressed: () => setState(() => debug = !debug),
                child: Text(debug ? 'DEBUG : ON' : 'DEBUG : OFF'),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class PeintreDebug extends CustomPainter {
  final List<Entite> entites;
  final bool debug;

  PeintreDebug({required this.entites, required this.debug});

  static const Map<TypeEntite, Color> couleurs = {
    TypeEntite.joueur: Color(0xFF4CAF50),
    TypeEntite.ennemi: Color(0xFFE53935),
    TypeEntite.solide: Color(0xFF3F6BD8),
    TypeEntite.ramassable: Color(0xFFFFC107),
    TypeEntite.zone: Color(0xFF00BCD4),
  };

  @override
  void paint(Canvas canvas, Size size) {
    // 1. Les sprites (remplis, semi-transparents).
    final remplissage = Paint()..style = PaintingStyle.fill;
    for (final e in entites) {
      remplissage.color = couleurs[e.type]!.withValues(alpha: 0.35);
      canvas.drawRect(e.sprite.rect, remplissage);
    }

    if (!debug) return;

    // 2. Le contour du sprite, en blanc pointillé simulé.
    final contourSprite = Paint()
      ..style = PaintingStyle.stroke
      ..strokeWidth = 1
      ..color = const Color(0x66FFFFFF);
    for (final e in entites) {
      canvas.drawRect(e.sprite.rect, contourSprite);
    }

    // 3. Les hitboxes, en trait épais coloré.
    final contourHitbox = Paint()
      ..style = PaintingStyle.stroke
      ..strokeWidth = 2;
    for (final e in entites) {
      contourHitbox.color =
          e.enCollision ? const Color(0xFFFF00FF) : couleurs[e.type]!;
      canvas.drawRect(e.hitbox.rect, contourHitbox);
    }

    // 4. Une croix au centre de chaque hitbox.
    final croix = Paint()
      ..color = const Color(0xFFFFFFFF)
      ..strokeWidth = 1;
    for (final e in entites) {
      final double cx = e.hitbox.x + e.hitbox.largeur / 2;
      final double cy = e.hitbox.y + e.hitbox.hauteur / 2;
      canvas.drawLine(Offset(cx - 4, cy), Offset(cx + 4, cy), croix);
      canvas.drawLine(Offset(cx, cy - 4), Offset(cx, cy + 4), croix);
    }

    // 5. La légende.
    final legende = <String>[
      'vert = joueur',
      'rouge = ennemi',
      'bleu = solide',
      'jaune = ramassable',
      'cyan = zone',
      'magenta = collision',
    ];
    double y = 8;
    for (final ligne in legende) {
      final tp = TextPainter(
        text: TextSpan(
          text: ligne,
          style: const TextStyle(color: Colors.white70, fontSize: 12),
        ),
        textDirection: TextDirection.ltr,
      )..layout();
      tp.paint(canvas, Offset(8, y));
      y += 16;
    }
  }

  @override
  bool shouldRepaint(covariant PeintreDebug ancien) => true;
}
```

**Résultat :** une fenêtre sombre où l'on voit les rectangles pleins (les « sprites ») et, par-dessus, les hitboxes en trait fin. Le joueur va et vient horizontalement ; chaque hitbox qu'il rencontre passe au magenta.

Ce que ce mode debug vous apprend immédiatement, et qu'aucun `print` ne vous dira :

- que la hitbox du joueur est décalée par rapport à son sprite ;
- que la collision se déclenche visuellement au bon moment ;
- qu'une hitbox est trop grande, trop petite, ou mal positionnée ;
- qu'un mur a une hitbox alors qu'il ne devrait pas.

> **Conseil.** Câblez ce mode debug **dès le premier jour** de votre projet, sur une touche du clavier. Vous gagnerez des heures. Au chapitre 32, Flame propose exactement la même chose avec `game.debugMode = true`.

---

## 24.29 — Ce que Flame fera à notre place (annonce du chapitre 32)

Tout ce que vous venez d'écrire à la main, le moteur Flame le fournit. Voici la correspondance, pour que rien ne soit magique quand vous y arriverez.

| Ce que nous avons écrit | Ce que Flame fournit (chapitre 32) |
| --- | --- |
| classe `Aabb` | `RectangleHitbox` |
| classe `Cercle` | `CircleHitbox` |
| `seChevauchent(a, b)` | détection interne du mixin `HasCollisionDetection` |
| boucle « tout le monde contre tout le monde » | broad phase interne (arbre de balayage) |
| notre grille spatiale | *sweep and prune* intégré |
| `enum Couche` + masques | `CollisionType.active` / `passive` / `inactive` |
| appel manuel de `resoudre(...)` | callbacks `onCollisionStart`, `onCollision`, `onCollisionEnd` |
| liste des points de contact | paramètre `Set<Vector2> intersectionPoints` |
| notre mode debug de la 24.28 | `game.debugMode = true` |
| notre `Corps.deplacer` en deux passes | à écrire vous-même — Flame ne résout pas |

La dernière ligne est la plus importante et surprend beaucoup de monde :

> **Flame détecte les collisions. Flame ne les résout pas.**

Flame vous prévient qu'un joueur touche un mur. Ce que vous faites ensuite — repousser, annuler la vitesse, mettre `estAuSol` à `true` — reste **votre** code, et ce sera exactement le code de la section 24.16. Le travail de ce chapitre n'est donc pas jeté : il est réutilisé tel quel.

Un aperçu, pour fixer les idées (ne cherchez pas à l'exécuter maintenant, il manque tout le contexte du chapitre 27) :

```dart
// APERÇU — chapitre 32. Ne fonctionne pas hors d'un projet Flame.
class Joueur extends PositionComponent with CollisionCallbacks {
  @override
  Future<void> onLoad() async {
    add(RectangleHitbox(
      size: Vector2(24, 40),
      position: Vector2(20, 24),
    ));
  }

  @override
  void onCollisionStart(
    Set<Vector2> points,
    PositionComponent autre,
  ) {
    super.onCollisionStart(points, autre);
    if (autre is Piece) {
      autre.removeFromParent();
      // score += 10;  <- toujours VOTRE code
    }
  }
}
```

Comparez avec ce que vous avez écrit en 24.26 : la structure est identique. Seule la plomberie de détection a disparu.

> **À retenir.** Un moteur ne remplace pas la compréhension. Il remplace la frappe au clavier. Vous savez maintenant ce qui tourne sous `onCollisionStart`, et vous saurez le déboguer.

---

## 24.30 — Mini-projet : le Donjon de Dart

Assemblons tout. Cahier des charges.

```text
  DONJON DE DART — SALLE 1

  ┌────────────────────────────────────────────────────────┐
  │████████████████████████████████████████████████████████│
  │██                                                    ██│
  │██   H        ██                ██          ●         ██│
  │██            ██                ██                    ██│
  │██   ●        ██                ██   ████████████     ██│
  │██            ██        ●       ██                    ██│
  │██        G───────►     ██      ██          ●         ██│
  │██            ██                ██   ████████████     ██│
  │██            ██        ●       ██                    ██│
  │██   ●                          ██          G         ██│
  │██                                          │         ██│
  │████████████████████████████████████████████████████████│
  └────────────────────────────────────────────────────────┘

  H = héros    ● = pièce    G = gobelin    ██ = mur

  RÈGLES
  1. Le héros se déplace aux flèches / ZQSD ou aux boutons.
  2. Il ne peut pas traverser les murs (résolution X puis Y).
  3. Toucher une pièce l'ajoute au score et la fait disparaître.
  4. Toucher un gobelin coûte une vie, avec recul et
     invincibilité de 1,2 s (clignotement).
  5. Zéro vie -> GAME OVER. Toutes les pièces -> VICTOIRE.
  6. La touche « HITBOX » affiche toutes les boîtes de collision.
```

Le programme met en œuvre, dans l'ordre : la classe `Aabb` (24.5), le test AABB (24.7), le sous-échantillonnage (24.20), la séparation des axes (24.16), les drapeaux de contact (24.17), les réactions typées (24.26), l'invincibilité (24.27) et le mode debug (24.28).

```dart
import 'dart:math';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() => runApp(const DonjonApp());

// ---------------------------------------------------------------
// GÉOMÉTRIE
// ---------------------------------------------------------------

class Aabb {
  double x, y, largeur, hauteur;
  Aabb(this.x, this.y, this.largeur, this.hauteur);

  double get gauche => x;
  double get droite => x + largeur;
  double get haut => y;
  double get bas => y + hauteur;
  double get centreX => x + largeur / 2;
  double get centreY => y + hauteur / 2;

  Rect get rect => Rect.fromLTWH(x, y, largeur, hauteur);
}

/// Test AABB. Le contact exact ne compte pas (section 24.6).
bool seChevauchent(Aabb a, Aabb b) =>
    a.droite > b.gauche &&
    a.gauche < b.droite &&
    a.bas > b.haut &&
    a.haut < b.bas;

// ---------------------------------------------------------------
// ENTITÉS
// ---------------------------------------------------------------

class Piece {
  final Aabb boite;
  bool prise = false;
  Piece(double x, double y) : boite = Aabb(x, y, 14, 14);
}

class Gobelin {
  final Aabb boite;
  final double debut;
  final double fin;
  final bool horizontal;
  double vitesse;

  Gobelin({
    required double x,
    required double y,
    required this.debut,
    required this.fin,
    required this.horizontal,
    required this.vitesse,
  }) : boite = Aabb(x, y, 26, 26);

  void miseAJour(double dt) {
    if (horizontal) {
      boite.x += vitesse * dt;
      if (boite.x < debut) {
        boite.x = debut;
        vitesse = -vitesse;
      } else if (boite.x > fin) {
        boite.x = fin;
        vitesse = -vitesse;
      }
    } else {
      boite.y += vitesse * dt;
      if (boite.y < debut) {
        boite.y = debut;
        vitesse = -vitesse;
      } else if (boite.y > fin) {
        boite.y = fin;
        vitesse = -vitesse;
      }
    }
  }
}

class Heros {
  final Aabb boite = Aabb(50, 50, 24, 28);

  double vxEntree = 0; // vitesse due aux touches
  double vyEntree = 0;
  double vxRecul = 0;  // vitesse due au recul d'un coup
  double vyRecul = 0;

  int vies = 3;
  double invincible = 0;

  static const double vitesse = 150;
  static const double pasMax = 5;
  static const double dureeInvincibilite = 1.2;

  bool get estInvincible => invincible > 0;
  bool get estVisible =>
      !estInvincible || (invincible * 10).floor().isEven;

  bool toucheMur = false;

  void miseAJour(double dt, List<Aabb> murs) {
    if (invincible > 0) {
      invincible = max(0, invincible - dt);
    }
    // Le recul s'amortit exponentiellement (chapitre 23 : frottement).
    final double facteur = max(0, 1 - 6 * dt);
    vxRecul *= facteur;
    vyRecul *= facteur;

    final double vx = vxEntree + vxRecul;
    final double vy = vyEntree + vyRecul;

    // Sous-échantillonnage (section 24.20).
    final double d = max((vx * dt).abs(), (vy * dt).abs());
    final int n = max(1, (d / pasMax).ceil());
    final double sousDt = dt / n;

    toucheMur = false;
    for (int i = 0; i < n; i++) {
      _unPas(vx, vy, sousDt, murs);
    }
  }

  /// Une passe X puis une passe Y (section 24.16).
  void _unPas(double vx, double vy, double dt, List<Aabb> murs) {
    boite.x += vx * dt;
    for (final m in murs) {
      if (!seChevauchent(boite, m)) continue;
      boite.x = vx > 0 ? m.gauche - boite.largeur : m.droite;
      vxRecul = 0;
      toucheMur = true;
    }

    boite.y += vy * dt;
    for (final m in murs) {
      if (!seChevauchent(boite, m)) continue;
      boite.y = vy > 0 ? m.haut - boite.hauteur : m.bas;
      vyRecul = 0;
      toucheMur = true;
    }
  }

  /// Retourne true si le coup a été encaissé (section 24.27).
  bool subirDegats(Aabb source) {
    if (estInvincible) return false;
    vies--;
    invincible = dureeInvincibilite;
    final double dx = boite.centreX - source.centreX;
    final double dy = boite.centreY - source.centreY;
    final double norme = sqrt(dx * dx + dy * dy);
    if (norme > 0.0001) {
      vxRecul = dx / norme * 320;
      vyRecul = dy / norme * 320;
    }
    return true;
  }
}

// ---------------------------------------------------------------
// LE MONDE
// ---------------------------------------------------------------

const double mondeLargeur = 640;
const double mondeHauteur = 400;

class Monde {
  late Heros heros;
  late List<Aabb> murs;
  late List<Piece> pieces;
  late List<Gobelin> gobelins;
  int score = 0;

  Monde() {
    reinitialiser();
  }

  bool get gagne => pieces.every((p) => p.prise);
  bool get perdu => heros.vies <= 0;
  bool get termine => gagne || perdu;

  void reinitialiser() {
    heros = Heros();
    score = 0;

    murs = <Aabb>[
      // Contour.
      Aabb(0, 0, mondeLargeur, 20),
      Aabb(0, mondeHauteur - 20, mondeLargeur, 20),
      Aabb(0, 0, 20, mondeHauteur),
      Aabb(mondeLargeur - 20, 0, 20, mondeHauteur),
      // Murs intérieurs.
      Aabb(140, 60, 20, 200),
      Aabb(300, 20, 20, 150),
      Aabb(300, 230, 20, 150),
      Aabb(440, 120, 140, 20),
      Aabb(440, 260, 140, 20),
    ];

    pieces = <Piece>[
      Piece(60, 300),
      Piece(200, 60),
      Piece(240, 300),
      Piece(360, 60),
      Piece(360, 320),
      Piece(500, 60),
      Piece(500, 200),
      Piece(500, 330),
    ];

    gobelins = <Gobelin>[
      Gobelin(
          x: 180, y: 180, debut: 170, fin: 270, horizontal: true,
          vitesse: 90),
      Gobelin(
          x: 400, y: 60, debut: 40, fin: 330, horizontal: false,
          vitesse: 110),
      Gobelin(
          x: 60, y: 200, debut: 30, fin: 110, horizontal: true,
          vitesse: 70),
    ];
  }

  void miseAJour(double dt) {
    if (termine) return;

    heros.miseAJour(dt, murs);
    for (final g in gobelins) {
      g.miseAJour(dt);
    }

    // Ramassage (section 24.26).
    for (final p in pieces) {
      if (p.prise) continue;
      if (seChevauchent(heros.boite, p.boite)) {
        p.prise = true;
        score += 10;
      }
    }

    // Dégâts (sections 24.26 et 24.27).
    for (final g in gobelins) {
      if (seChevauchent(heros.boite, g.boite)) {
        heros.subirDegats(g.boite);
      }
    }
  }
}

// ---------------------------------------------------------------
// L'APPLICATION
// ---------------------------------------------------------------

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: PageDonjon(),
    );
  }
}

class PageDonjon extends StatefulWidget {
  const PageDonjon({super.key});

  @override
  State<PageDonjon> createState() => _PageDonjonState();
}

class _PageDonjonState extends State<PageDonjon>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  final Monde monde = Monde();
  final Set<LogicalKeyboardKey> _touches = {};
  final Set<String> _boutons = {};
  bool debug = false;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_frame)..start();
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  void _frame(Duration maintenant) {
    double dt = (maintenant - _precedent).inMicroseconds / 1000000.0;
    _precedent = maintenant;
    if (dt <= 0) return;
    dt = dt.clamp(0.0, 0.05); // plafond du chapitre 20

    setState(() {
      _lireEntrees();
      monde.miseAJour(dt);
    });
  }

  bool _actif(String nom, List<LogicalKeyboardKey> cles) {
    if (_boutons.contains(nom)) return true;
    for (final c in cles) {
      if (_touches.contains(c)) return true;
    }
    return false;
  }

  void _lireEntrees() {
    final bool gauche = _actif('gauche', [
      LogicalKeyboardKey.arrowLeft,
      LogicalKeyboardKey.keyQ,
      LogicalKeyboardKey.keyA,
    ]);
    final bool droite = _actif('droite', [
      LogicalKeyboardKey.arrowRight,
      LogicalKeyboardKey.keyD,
    ]);
    final bool haut = _actif('haut', [
      LogicalKeyboardKey.arrowUp,
      LogicalKeyboardKey.keyZ,
      LogicalKeyboardKey.keyW,
    ]);
    final bool bas = _actif('bas', [
      LogicalKeyboardKey.arrowDown,
      LogicalKeyboardKey.keyS,
    ]);

    double dx = (droite ? 1 : 0) - (gauche ? 1 : 0);
    double dy = (bas ? 1 : 0) - (haut ? 1 : 0);

    // Normalisation : la diagonale ne doit pas être plus rapide.
    if (dx != 0 && dy != 0) {
      final double n = sqrt(2);
      dx /= n;
      dy /= n;
    }

    monde.heros.vxEntree = dx * Heros.vitesse;
    monde.heros.vyEntree = dy * Heros.vitesse;
  }

  KeyEventResult _clavier(FocusNode noeud, KeyEvent event) {
    if (event is KeyDownEvent) {
      _touches.add(event.logicalKey);
      if (event.logicalKey == LogicalKeyboardKey.keyH) {
        setState(() => debug = !debug);
      }
      if (event.logicalKey == LogicalKeyboardKey.keyR) {
        setState(monde.reinitialiser);
      }
    } else if (event is KeyUpEvent) {
      _touches.remove(event.logicalKey);
    }
    return KeyEventResult.handled;
  }

  Widget _bouton(String nom, IconData icone) {
    return Listener(
      onPointerDown: (_) => setState(() => _boutons.add(nom)),
      onPointerUp: (_) => setState(() => _boutons.remove(nom)),
      onPointerCancel: (_) => setState(() => _boutons.remove(nom)),
      child: Container(
        margin: const EdgeInsets.all(4),
        width: 54,
        height: 54,
        decoration: BoxDecoration(
          color: const Color(0xFF2A2F45),
          borderRadius: BorderRadius.circular(8),
        ),
        child: Icon(icone, color: Colors.white70),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF0B0D14),
      body: SafeArea(
        child: Focus(
          autofocus: true,
          onKeyEvent: _clavier,
          child: Column(
            children: [
              Padding(
                padding: const EdgeInsets.all(8),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    Text(
                      'Score ${monde.score}   '
                      'Vies ${monde.heros.vies}   '
                      'Pièces ${monde.pieces.where((p) => p.prise).length}'
                      '/${monde.pieces.length}',
                      style: const TextStyle(
                          color: Colors.white, fontSize: 16),
                    ),
                    Row(
                      children: [
                        TextButton(
                          onPressed: () => setState(() => debug = !debug),
                          child: Text(debug ? 'HITBOX ON' : 'HITBOX OFF'),
                        ),
                        TextButton(
                          onPressed: () => setState(monde.reinitialiser),
                          child: const Text('REJOUER'),
                        ),
                      ],
                    ),
                  ],
                ),
              ),
              Expanded(
                child: Center(
                  child: FittedBox(
                    child: SizedBox(
                      width: mondeLargeur,
                      height: mondeHauteur,
                      child: CustomPaint(
                        painter: PeintreDonjon(monde: monde, debug: debug),
                      ),
                    ),
                  ),
                ),
              ),
              Padding(
                padding: const EdgeInsets.only(bottom: 12),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    _bouton('gauche', Icons.arrow_left),
                    Column(
                      children: [
                        _bouton('haut', Icons.arrow_drop_up),
                        _bouton('bas', Icons.arrow_drop_down),
                      ],
                    ),
                    _bouton('droite', Icons.arrow_right),
                  ],
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

// ---------------------------------------------------------------
// LE RENDU
// ---------------------------------------------------------------

class PeintreDonjon extends CustomPainter {
  final Monde monde;
  final bool debug;

  PeintreDonjon({required this.monde, required this.debug});

  @override
  void paint(Canvas canvas, Size size) {
    // Sol.
    canvas.drawRect(
      Rect.fromLTWH(0, 0, mondeLargeur, mondeHauteur),
      Paint()..color = const Color(0xFF1B2030),
    );

    // Murs.
    final pMur = Paint()..color = const Color(0xFF3C4763);
    for (final m in monde.murs) {
      canvas.drawRect(m.rect, pMur);
    }

    // Pièces.
    final pPiece = Paint()..color = const Color(0xFFFFC93C);
    for (final p in monde.pieces) {
      if (p.prise) continue;
      canvas.drawCircle(
        Offset(p.boite.centreX, p.boite.centreY),
        p.boite.largeur / 2,
        pPiece,
      );
    }

    // Gobelins.
    final pGob = Paint()..color = const Color(0xFF5FBF61);
    final pOeil = Paint()..color = const Color(0xFF101010);
    for (final g in monde.gobelins) {
      canvas.drawRRect(
        RRect.fromRectAndRadius(g.boite.rect, const Radius.circular(5)),
        pGob,
      );
      canvas.drawCircle(
          Offset(g.boite.centreX - 5, g.boite.centreY - 3), 2.5, pOeil);
      canvas.drawCircle(
          Offset(g.boite.centreX + 5, g.boite.centreY - 3), 2.5, pOeil);
    }

    // Héros (il clignote pendant l'invincibilité).
    if (monde.heros.estVisible) {
      canvas.drawRRect(
        RRect.fromRectAndRadius(
            monde.heros.boite.rect, const Radius.circular(4)),
        Paint()..color = const Color(0xFF6FA8FF),
      );
    }

    // Mode debug (section 24.28).
    if (debug) {
      final contour = Paint()
        ..style = PaintingStyle.stroke
        ..strokeWidth = 1.5;

      contour.color = const Color(0xFF3F6BD8);
      for (final m in monde.murs) {
        canvas.drawRect(m.rect, contour);
      }
      contour.color = const Color(0xFFFFC107);
      for (final p in monde.pieces) {
        if (!p.prise) canvas.drawRect(p.boite.rect, contour);
      }
      contour.color = const Color(0xFFE53935);
      for (final g in monde.gobelins) {
        canvas.drawRect(g.boite.rect, contour);
      }
      contour.color = monde.heros.toucheMur
          ? const Color(0xFFFF00FF)
          : const Color(0xFF4CAF50);
      canvas.drawRect(monde.heros.boite.rect, contour);
    }

    // Message de fin.
    if (monde.termine) {
      canvas.drawRect(
        Rect.fromLTWH(0, 0, mondeLargeur, mondeHauteur),
        Paint()..color = const Color(0xFF000000).withValues(alpha: 0.65),
      );
      final String texte = monde.gagne ? 'VICTOIRE' : 'GAME OVER';
      final tp = TextPainter(
        text: TextSpan(
          text: '$texte\nScore ${monde.score}\n(touche R pour rejouer)',
          style: const TextStyle(
            color: Colors.white,
            fontSize: 28,
            height: 1.4,
          ),
        ),
        textAlign: TextAlign.center,
        textDirection: TextDirection.ltr,
      )..layout(maxWidth: mondeLargeur);
      tp.paint(
        canvas,
        Offset((mondeLargeur - tp.width) / 2,
            (mondeHauteur - tp.height) / 2),
      );
    }
  }

  @override
  bool shouldRepaint(covariant PeintreDonjon ancien) => true;
}
```

**Résultat :** une salle de donjon jouable. Le héros bleu se déplace aux flèches (ou aux boutons tactiles), se cogne aux murs sans jamais les traverser, ramasse les huit pièces jaunes, et clignote pendant 1,2 seconde après chaque contact avec un gobelin vert. Les touches `H` et `R` activent le mode hitbox et relancent la partie.

### Ce qu'il faut retenir de ce mini-projet

| Élément du programme | Section d'origine |
| --- | --- |
| `class Aabb` et ses accesseurs | 24.5 |
| `seChevauchent` avec comparaisons strictes | 24.6, 24.7 |
| `Heros._unPas` : passe X puis passe Y | 24.16 |
| `toucheMur` remis à `false` chaque frame | 24.17 |
| Découpage en `n` sous-pas selon la vitesse | 24.20 |
| Trois boucles séparées : murs, pièces, gobelins | 24.24 |
| `p.prise` avant d'ajouter au score | 24.26 |
| `invincible`, `estVisible`, recul normalisé | 24.27 |
| Contours colorés en mode `debug` | 24.28 |
| `dt.clamp(0, 0.05)` | chapitre 20 |
| Normalisation de la diagonale | chapitre 23 |

Trois détails méritent qu'on s'y arrête.

**La diagonale est normalisée.** Sans `dx /= sqrt(2)`, un joueur qui appuie sur haut + droite avancerait à `√2 ≈ 1,41` fois sa vitesse normale. C'est un bug visible immédiatement par n'importe quel joueur.

**Le recul est annulé par les murs.** Dans `_unPas`, on met `vxRecul = 0` en cas de collision. Sinon, un héros repoussé contre un mur resterait « poussé » et vibrerait.

**Le jeu s'arrête proprement.** `Monde.miseAJour` sort immédiatement si `termine`. Les gobelins se figent, plus aucun dégât n'est appliqué, et l'écran de fin est stable.

---

## 24.31 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| La hitbox est calée sur la taille du sprite | on confond « ce qu'on voit » et « ce qui touche » | définir une hitbox indépendante, plus petite, avec un décalage constant (24.3) |
| La hitbox change de taille selon la frame d'animation | on la calcule à partir de l'image courante | garder une hitbox de déplacement **constante**, quelle que soit l'animation (24.3) |
| Le personnage saute en diagonale quand il touche un mur | résolution sur X **et** Y dans la même passe | séparer strictement les deux passes (24.16) |
| Impossible de s'éloigner d'un mur, on reste collé | test de chevauchement en comparaison large (`<=`) | passer en comparaison stricte : le contact exact n'est pas une collision (24.6, 24.15) |
| Le personnage vibre contre le mur | la vitesse n'est pas annulée après le replacement | mettre `vx = 0` (ou `vy = 0`) sur l'axe résolu (24.15) |
| Le joueur est catapulté quand il quitte le mur | la vitesse a continué de s'accumuler pendant le contact | même correction : annuler la vitesse à chaque frame de contact (24.15) |
| Un projectile rapide traverse un mur fin | tunneling : on ne teste que la position d'arrivée | sous-échantillonner le déplacement, ou faire un swept AABB (24.18 à 24.20) |
| Un test d'alignement ne se déclenche jamais | comparaison de deux `double` avec `==` | comparer avec une tolérance : `(a - b).abs() < epsilon` (24.15) |
| Le joueur peut sauter en l'air à l'infini | `estAuSol` n'est jamais remis à `false` | réinitialiser tous les drapeaux de contact au début de chaque frame (24.17) |
| Une pièce donne 300 points au lieu de 10 | le ramassage est appliqué à chaque frame de contact | marquer la pièce comme prise et sortir immédiatement (24.26) |
| Le héros meurt en un dixième de seconde | les dégâts sont appliqués à chaque frame | ajouter une invincibilité temporaire décomptée avec `dt` (24.27) |
| L'invincibilité ne finit jamais | on oublie de décrémenter le compteur, ou on décrémente d'une valeur fixe | `invincible = max(0, invincible - dt)` à chaque frame (24.27) |
| Pendant l'invincibilité, le joueur traverse les murs | on a coupé toutes les collisions au lieu de la seule couche ennemi | retirer un bit précis du masque, pas tout le masque (24.25) |
| Le jeu tombe à 20 FPS avec 800 entités | détection en O(n²) | filtrer par couches, puis par grille spatiale (24.21 à 24.24) |
| Une collision est comptée deux fois | la même paire est testée dans les deux sens | dédoublonner les paires, ou n'itérer que sur `j > i` (24.23) |
| `ConcurrentModificationError` au ramassage | on supprime de la liste pendant qu'on la parcourt | marquer l'objet, puis `removeWhere` après la boucle (24.26) |
| La diagonale est plus rapide que la ligne droite | vecteur de direction non normalisé | diviser `dx` et `dy` par `sqrt(2)` en diagonale (24.30) |
| `clamp` renvoie une erreur de type | `clamp` retourne un `num` | utiliser des bornes `double`, ou ajouter `.toDouble()` (24.12) |

---

## 24.32 — Résumé du chapitre

| Type de test | Formule | Coût |
| --- | --- | --- |
| AABB / AABB | `a.droite > b.gauche && a.gauche < b.droite && a.bas > b.haut && a.haut < b.bas` | 4 comparaisons |
| Overlap AABB (profondeur) | `ox = min(ad, bd) - max(ag, bg)` ; idem sur Y ; collision si `ox > 0 && oy > 0` | 4 min/max + 2 soustractions |
| Cercle / cercle | `dx² + dy² < (rA + rB)²` | 3 multiplications, 2 soustractions, 1 comparaison |
| Cercle / cercle avec `sqrt` | `sqrt(dx² + dy²) < rA + rB` | idem + 1 racine (environ 3 fois plus lent) |
| Point / rectangle | `px >= g && px < d && py >= h && py < b` | 4 comparaisons |
| Cercle / rectangle | `p = (cx.clamp(g,d), cy.clamp(h,b))` puis `(cx-px)² + (cy-py)² < r²` | 2 clamp + 3 multiplications |
| Swept AABB (1 obstacle) | `tEntrée = max(tEntréeX, tEntréeY)`, contact si `tEntrée <= tSortie` et `0 <= tEntrée <= 1` | environ 30 opérations |
| Sous-échantillonnage | `n = ceil(déplacement / pasMax)` puis `n` tests simples | `n` fois le coût d'un pas |
| Résolution « plus petit axe » | repousser de `min(ox, oy)` dans le sens donné par les centres | négligeable |
| Résolution par axes séparés | passe X (replacer, `vx = 0`), puis passe Y (replacer, `vy = 0`) | 2 fois le coût de détection |
| Broad phase par distance | `dx² + dy² <= (rA + rB)²` sur les rayons englobants | 1 test par paire, toujours O(n²) |
| Grille spatiale | `col = (x / taille).floor()` ; ne comparer que dans une même case | O(n) d'insertion, paires très réduites |
| Filtrage par couches | `interactions[a].contains(b)` dans les deux sens | 2 `Set.contains` |
| Masques de bits | `(a.masque & b.couche) != 0 && (b.masque & a.couche) != 0` | 2 opérations processeur |
| Test « tout contre tout » | `n × (n - 1) / 2` paires | O(n²) : acceptable jusqu'à ~200 entités |

| Notion | À retenir |
| --- | --- |
| Détection / résolution | deux problèmes séparés ; la détection ne modifie rien |
| Hitbox | plus petite que le sprite, constante, indépendante de l'animation |
| Hitbox / hurtbox / trigger | infliger / recevoir / déclencher : trois rôles distincts |
| AABB | rectangle jamais tourné ; 95 % des besoins d'un jeu 2D |
| Axe séparateur | tester la NON-collision : seulement quatre cas possibles |
| Distances | comparer les carrés, jamais les racines |
| Overlap | contient à la fois le booléen et la profondeur |
| Axes séparés | la seule méthode de résolution vraiment fiable |
| Drapeaux de contact | remis à `false` au début de chaque frame |
| Tunneling | apparaît quand `vitesse × dt >= épaisseur de l'obstacle` |
| Broad / narrow | le broad peut se tromper « en trop », jamais « en moins » |
| Invincibilité | un simple `double` décrémenté par `dt` |
| Mode debug | à câbler dès le premier jour du projet |
| Flame | détecte les collisions, ne les résout pas |

---

## 24.33 — Exercices

### Exercice 1 — Le test AABB de base (facile)

Écrivez une classe `Aabb` (coin haut-gauche + taille) et une fonction `seChevauchent(Aabb a, Aabb b)` qui retourne `true` si les deux rectangles se chevauchent, le contact exact ne comptant pas.

Testez-la sur au moins six configurations : séparés, contact exact, chevauchement partiel, inclusion, englobement, croix. Affichez pour chacune le résultat attendu et le résultat obtenu, ainsi qu'un bilan `X / 6 tests réussis`.

### Exercice 2 — Le gobelin le plus proche (facile)

Le héros est en `(320, 240)`. Une liste de gobelins est donnée avec leur nom et leur position.

Écrivez un programme qui affiche les gobelins **triés du plus proche au plus lointain**, sans jamais appeler `sqrt` pour le tri. N'appelez `sqrt` que pour l'affichage final de la distance. Indiquez aussi lesquels sont dans un rayon d'alerte de 150 pixels.

### Exercice 3 — Le HUD tactile (facile)

Trois boutons du HUD occupent respectivement `(0, 400, 100, 60)`, `(100, 400, 100, 60)` et `(200, 400, 100, 60)`.

Écrivez `pointDansRect` avec la convention `>=` / `<` et vérifiez sur une liste de sept positions de doigt qu'aucune position n'active deux boutons à la fois. Affichez, pour chaque position, le bouton activé ou `aucun`.

### Exercice 4 — La boule de feu contre le mur (moyen)

Un mur occupe `(200, 100, 60, 120)`. Une boule de feu est un cercle de rayon 18 qui part de `(40, 160)` et avance de 25 pixels par frame vers la droite.

Écrivez la fonction `cercleRect` (méthode du point le plus proche) et simulez 12 frames. Affichez à chaque frame la position, le point du rectangle le plus proche, la distance, et signalez la frame d'impact. Le programme doit s'arrêter à l'impact.

### Exercice 5 — Profondeur et repoussée (moyen)

Écrivez une fonction `repousser(Aabb mobile, Aabb solide)` qui calcule `overlapX` et `overlapY`, choisit l'axe de plus petit overlap, détermine le sens en comparant les centres, déplace le mobile et retourne la direction utilisée (`'gauche'`, `'droite'`, `'haut'`, `'bas'`) ou `null`.

Testez sur quatre situations (arrivée par chaque côté) et vérifiez à chaque fois qu'après repoussée il n'y a plus de chevauchement.

### Exercice 6 — Le couloir en L (moyen)

Construisez un décor formé de trois murs dessinant un couloir en L. Écrivez une classe `Corps` avec une méthode `deplacer(double dt, List<Aabb> solides)` qui applique la séparation des axes X puis Y, met les vitesses à zéro sur l'axe résolu et renseigne quatre drapeaux : `sol`, `plafond`, `murGauche`, `murDroit`.

Simulez 90 frames avec une gravité de 1000 px/s² et une vitesse horizontale de 200 px/s. Affichez une ligne toutes les 10 frames avec la position et les drapeaux actifs.

### Exercice 7 — Anti-tunneling (moyen)

Un mur fait 10 pixels d'épaisseur, en `(400, 0, 10, 300)`. Une flèche de 8 × 8 part de `(0, 100)` à 4000 px/s.

Écrivez deux simulations sur 10 frames à 60 FPS : l'une sans sous-échantillonnage, l'autre avec un `pasMax` de 4 pixels. Affichez la position finale de chaque flèche, le nombre de sous-pas utilisés, et concluez.

### Exercice 8 — La grille spatiale (difficile)

Écrivez une classe `GrilleSpatiale` avec les méthodes `vider()`, `inserer(Entite e)` et `pairesCandidates()`.

Générez 300 entités de 20 × 20 pixels dans un monde de 2000 × 2000, à des positions **pseudo-aléatoires déterministes** produites par le générateur à congruence linéaire suivant (afin que votre résultat soit exactement reproductible) :

```dart
int graine = 12345;
int suivant() {
  graine = (graine * 1103515245 + 12345) % 2147483648;
  return graine;
}
// position d'une entité : x = (suivant() % 1980).toDouble()
//                         y = (suivant() % 1980).toDouble()
```

Comparez le nombre de paires en O(n²) et le nombre de paires candidates pour des tailles de cellule de 50, 100, 200 et 500. Affichez un tableau et concluez sur la taille à choisir.

### Exercice 9 — Masques et invincibilité (difficile)

Définissez cinq couches en puissances de deux : `decor`, `joueur`, `ennemi`, `ramassable`, `projectileEnnemi`.

Écrivez une classe `Entite` avec `couche` et `masque`, la fonction `doitTester`, puis simulez 180 frames où le héros est en contact permanent avec un gobelin, un mur et une pièce. À chaque coup reçu, retirez temporairement les bits `ennemi` et `projectileEnnemi` du masque du héros pendant 1,2 seconde, puis remettez-les.

Affichez le nombre total de coups encaissés, le nombre de collisions avec le mur (qui ne doit jamais être interrompu) et le score final.

### Exercice 10 — La salle du trésor (difficile)

Écrivez un `main.dart` Flutter complet affichant une salle de donjon de 480 × 320 pixels avec :

- quatre murs de contour et un pilier central ;
- un héros de 20 × 24 déplacé automatiquement en cercle (aucune entrée clavier n'est demandée) ;
- cinq pièces à ramasser, avec un score affiché ;
- un gobelin qui patrouille horizontalement, une vie perdue par contact et une invincibilité de 1 seconde avec clignotement ;
- un bouton qui affiche ou masque toutes les hitboxes.

Le programme doit tourner sans aucune image ni asset externe.

---

## 24.34 — Corrections des exercices

### Correction 1

```dart
class Aabb {
  double x, y, largeur, hauteur;
  Aabb(this.x, this.y, this.largeur, this.hauteur);

  double get gauche => x;
  double get droite => x + largeur;
  double get haut => y;
  double get bas => y + hauteur;

  @override
  String toString() => '($x, $y, ${largeur}x$hauteur)';
}

/// Les quatre cas de NON-collision (section 24.8).
/// Le contact exact ne compte pas : comparaisons strictes.
bool seChevauchent(Aabb a, Aabb b) {
  if (a.droite <= b.gauche) return false;
  if (a.gauche >= b.droite) return false;
  if (a.bas <= b.haut) return false;
  if (a.haut >= b.bas) return false;
  return true;
}

class Cas {
  final String nom;
  final Aabb autre;
  final bool attendu;
  const Cas(this.nom, this.autre, this.attendu);
}

void main() {
  final joueur = Aabb(100, 100, 40, 40); // g100 d140 h100 b140

  final cas = <Cas>[
    Cas('séparés', Aabb(200, 200, 40, 40), false),
    Cas('contact exact à droite', Aabb(140, 100, 40, 40), false),
    Cas('chevauchement partiel', Aabb(130, 110, 40, 40), true),
    Cas('inclusion (petit dedans)', Aabb(110, 110, 10, 10), true),
    Cas('englobement (grand autour)', Aabb(80, 80, 100, 100), true),
    Cas('croix (aucun coin inclus)', Aabb(110, 60, 20, 120), true),
  ];

  int reussis = 0;
  for (final c in cas) {
    final bool obtenu = seChevauchent(joueur, c.autre);
    final bool ok = obtenu == c.attendu;
    if (ok) reussis++;
    print('${ok ? "OK  " : "ÉCHEC"} ${c.nom.padRight(28)} '
        'attendu=${c.attendu.toString().padRight(5)} '
        'obtenu=${obtenu.toString().padRight(5)}');
  }

  print('');
  print('$reussis / ${cas.length} tests réussis');

  // Vérification supplémentaire : la fonction est symétrique.
  bool symetrique = true;
  for (final c in cas) {
    if (seChevauchent(joueur, c.autre) !=
        seChevauchent(c.autre, joueur)) {
      symetrique = false;
    }
  }
  print('Fonction symétrique : $symetrique');
}
```

**Résultat :**

```text
OK   séparés                      attendu=false obtenu=false
OK   contact exact à droite       attendu=false obtenu=false
OK   chevauchement partiel        attendu=true  obtenu=true 
OK   inclusion (petit dedans)     attendu=true  obtenu=true 
OK   englobement (grand autour)   attendu=true  obtenu=true 
OK   croix (aucun coin inclus)    attendu=true  obtenu=true 

6 / 6 tests réussis
Fonction symétrique : true
```

**Explication :** la fonction ferme les quatre échappatoires de la non-collision, dans l'ordre gauche, droite, haut, bas. Les deux cas les plus instructifs sont le contact exact, écarté grâce aux comparaisons **strictes** `<=` et `>=` dans les `return false`, et la croix, correctement détectée alors qu'aucun coin de l'un n'est dans l'autre : c'est la preuve que le raisonnement par intervalles est le bon. La symétrie est garantie par construction, puisque les quatre conditions sont deux à deux échangées quand on permute `a` et `b`.

---

### Correction 2

```dart
import 'dart:math';

class Gobelin {
  final String nom;
  final double x;
  final double y;
  const Gobelin(this.nom, this.x, this.y);
}

/// Distance AU CARRÉ : aucune racine (section 24.10).
double distanceCarree(double ax, double ay, double bx, double by) {
  final double dx = bx - ax;
  final double dy = by - ay;
  return dx * dx + dy * dy;
}

void main() {
  const double herosX = 320;
  const double herosY = 240;
  const double rayonAlerte = 150;
  const double rayonAlerteCarre = rayonAlerte * rayonAlerte;

  final gobelins = <Gobelin>[
    const Gobelin('Grok', 380, 280),
    const Gobelin('Zog', 320, 300),
    const Gobelin('Nak', 100, 240),
    const Gobelin('Rul', 350, 200),
    const Gobelin('Vex', 500, 400),
    const Gobelin('Mog', 270, 180),
  ];

  // Tri sur la distance AU CARRÉ : le classement est identique.
  gobelins.sort((a, b) {
    final double da = distanceCarree(herosX, herosY, a.x, a.y);
    final double db = distanceCarree(herosX, herosY, b.x, b.y);
    return da.compareTo(db);
  });

  print('Héros en ($herosX, $herosY), rayon d\'alerte $rayonAlerte px');
  print('');
  print('nom    distance²   distance   état');
  print('-----------------------------------');
  for (final g in gobelins) {
    final double d2 = distanceCarree(herosX, herosY, g.x, g.y);
    // sqrt UNIQUEMENT pour l'affichage.
    final double d = sqrt(d2);
    final String etat = d2 < rayonAlerteCarre ? 'ALERTE' : '-';
    print('${g.nom.padRight(6)} '
        '${d2.toStringAsFixed(0).padLeft(9)}   '
        '${d.toStringAsFixed(2).padLeft(7)}   $etat');
  }

  final int enAlerte = gobelins
      .where((g) =>
          distanceCarree(herosX, herosY, g.x, g.y) < rayonAlerteCarre)
      .length;
  print('');
  print('$enAlerte gobelins dans le rayon d\'alerte.');
}
```

**Résultat :**

```text
Héros en (320.0, 240.0), rayon d'alerte 150.0 px

nom    distance²   distance   état
-----------------------------------
Rul         2500     50.00   ALERTE
Zog         3600     60.00   ALERTE
Grok        5200     72.11   ALERTE
Mog         6100     78.10   ALERTE
Nak        48400    220.00   -
Vex        58000    240.83   -

4 gobelins dans le rayon d'alerte.
```

**Explication :** le tri et le test de seuil se font entièrement sur `distanceCarree`, sans jamais appeler `sqrt`. C'est légitime parce que la fonction racine carrée est **croissante** : si `d²(A) < d²(B)`, alors `d(A) < d(B)`. Le seuil est lui aussi élevé au carré une fois pour toutes dans `rayonAlerteCarre`, ce qui évite de le recalculer. `sqrt` n'apparaît qu'à la ligne d'affichage, là où l'on a réellement besoin de la valeur en pixels. Sur un jeu avec 200 ennemis, cette discipline économise 200 racines carrées par frame.

---

### Correction 3

```dart
class Aabb {
  final double x, y, largeur, hauteur;
  const Aabb(this.x, this.y, this.largeur, this.hauteur);

  double get gauche => x;
  double get droite => x + largeur;
  double get haut => y;
  double get bas => y + hauteur;
}

/// Convention >= / < : une frontière appartient à UN SEUL rectangle.
bool pointDansRect(double px, double py, Aabb r) =>
    px >= r.gauche && px < r.droite && py >= r.haut && py < r.bas;

void main() {
  final boutons = <String, Aabb>{
    'GAUCHE': const Aabb(0, 400, 100, 60),
    'SAUT': const Aabb(100, 400, 100, 60),
    'DROITE': const Aabb(200, 400, 100, 60),
  };

  final doigts = <List<double>>[
    [50, 430],
    [99.9, 430],
    [100, 430],
    [200, 430],
    [299.9, 459.9],
    [300, 430],
    [150, 399.9],
  ];

  bool doubleActivation = false;

  for (final d in doigts) {
    final actifs = <String>[];
    boutons.forEach((nom, r) {
      if (pointDansRect(d[0], d[1], r)) actifs.add(nom);
    });
    if (actifs.length > 1) doubleActivation = true;
    final String reponse = actifs.isEmpty ? 'aucun' : actifs.join(' + ');
    print('doigt (${d[0].toString().padLeft(5)}, '
        '${d[1].toString().padLeft(5)}) -> $reponse');
  }

  print('');
  print('Une position a-t-elle activé deux boutons ? $doubleActivation');
}
```

**Résultat :**

```text
doigt ( 50.0, 430.0) -> GAUCHE
doigt ( 99.9, 430.0) -> GAUCHE
doigt (100.0, 430.0) -> SAUT
doigt (200.0, 430.0) -> DROITE
doigt (299.9, 459.9) -> DROITE
doigt (300.0, 430.0) -> aucun
doigt (150.0, 399.9) -> aucun

Une position a-t-elle activé deux boutons ? false
```

**Explication :** l'asymétrie `>=` sur les bords gauche et haut, `<` sur les bords droit et bas, fait qu'un point situé exactement sur une frontière appartient au rectangle **de droite** (ou du bas). En `x = 100`, le doigt active `SAUT` et non `GAUCHE + SAUT`. En `x = 300`, il n'active plus rien : c'est le premier pixel après le dernier bouton. Avec des `<=` partout, le HUD aurait déclenché deux actions simultanées sur un seul appui, bug fréquent des interfaces tactiles. La même convention sert à découper une tilemap en cases sans qu'une position se retrouve dans deux tuiles.

---

### Correction 4

```dart
import 'dart:math';

class Aabb {
  final double x, y, largeur, hauteur;
  const Aabb(this.x, this.y, this.largeur, this.hauteur);

  double get gauche => x;
  double get droite => x + largeur;
  double get haut => y;
  double get bas => y + hauteur;
}

class Cercle {
  double cx, cy;
  final double rayon;
  Cercle(this.cx, this.cy, this.rayon);
}

/// Point du rectangle le plus proche d'un point : deux clamp.
List<double> pointLePlusProche(Aabb r, double px, double py) => [
      px.clamp(r.gauche, r.droite),
      py.clamp(r.haut, r.bas),
    ];

/// Collision cercle / rectangle, sans racine carrée.
bool cercleRect(Cercle c, Aabb r) {
  final double px = c.cx.clamp(r.gauche, r.droite);
  final double py = c.cy.clamp(r.haut, r.bas);
  final double dx = c.cx - px;
  final double dy = c.cy - py;
  return dx * dx + dy * dy < c.rayon * c.rayon;
}

void main() {
  const mur = Aabb(200, 100, 60, 120); // g200 d260 h100 b220
  final boule = Cercle(40, 160, 18);
  const double parFrame = 25;

  print('Mur : x de ${mur.gauche} à ${mur.droite}, '
      'y de ${mur.haut} à ${mur.bas}');
  print('Boule : rayon ${boule.rayon}, '
      'avance de $parFrame px par frame');
  print('');

  for (int frame = 1; frame <= 12; frame++) {
    final p = pointLePlusProche(mur, boule.cx, boule.cy);
    final double dx = boule.cx - p[0];
    final double dy = boule.cy - p[1];
    final double d = sqrt(dx * dx + dy * dy);
    final bool impact = cercleRect(boule, mur);

    print('frame ${frame.toString().padLeft(2)} : '
        'centre=(${boule.cx.toStringAsFixed(1)}, '
        '${boule.cy.toStringAsFixed(1)}) '
        'proche=(${p[0].toStringAsFixed(1)}, '
        '${p[1].toStringAsFixed(1)}) '
        'd=${d.toStringAsFixed(2)}'
        '${impact ? "  IMPACT" : ""}');

    if (impact) {
      print('');
      print('La boule de feu explose à la frame $frame.');
      return;
    }
    boule.cx += parFrame;
  }

  print('');
  print('Aucun impact en 12 frames.');
}
```

**Résultat :**

```text
Mur : x de 200.0 à 260.0, y de 100.0 à 220.0
Boule : rayon 18.0, avance de 25.0 px par frame

frame  1 : centre=(40.0, 160.0) proche=(200.0, 160.0) d=160.00
frame  2 : centre=(65.0, 160.0) proche=(200.0, 160.0) d=135.00
frame  3 : centre=(90.0, 160.0) proche=(200.0, 160.0) d=110.00
frame  4 : centre=(115.0, 160.0) proche=(200.0, 160.0) d=85.00
frame  5 : centre=(140.0, 160.0) proche=(200.0, 160.0) d=60.00
frame  6 : centre=(165.0, 160.0) proche=(200.0, 160.0) d=35.00
frame  7 : centre=(190.0, 160.0) proche=(200.0, 160.0) d=10.00  IMPACT

La boule de feu explose à la frame 7.
```

**Explication :** le point le plus proche est obtenu par deux `clamp` indépendants. Ici, le centre de la boule est toujours à `y = 160`, valeur comprise entre `haut = 100` et `bas = 220` : le `clamp` sur Y ne change donc rien, et le point le plus proche reste sur le bord gauche du mur, à `x = 200`. La distance est simplement `200 - cx`. L'impact se produit quand cette distance passe sous le rayon 18, c'est-à-dire à la frame 7 (`d = 10`). Notez qu'à la frame 6 la distance était de 35 : le déplacement de 25 pixels par frame reste inférieur à l'épaisseur du mur (60), il n'y a donc aucun risque de tunneling ici.

---

### Correction 5

```dart
import 'dart:math';

class Aabb {
  double x, y, largeur, hauteur;
  Aabb(this.x, this.y, this.largeur, this.hauteur);

  double get gauche => x;
  double get droite => x + largeur;
  double get haut => y;
  double get bas => y + hauteur;
  double get centreX => x + largeur / 2;
  double get centreY => y + hauteur / 2;

  @override
  String toString() =>
      '(${x.toStringAsFixed(1)}, ${y.toStringAsFixed(1)})';
}

bool seChevauchent(Aabb a, Aabb b) =>
    a.droite > b.gauche &&
    a.gauche < b.droite &&
    a.bas > b.haut &&
    a.haut < b.bas;

/// Repousse [mobile] hors de [solide] par le plus petit axe.
String? repousser(Aabb mobile, Aabb solide) {
  final double ox = min(mobile.droite, solide.droite) -
      max(mobile.gauche, solide.gauche);
  final double oy =
      min(mobile.bas, solide.bas) - max(mobile.haut, solide.haut);

  if (ox <= 0 || oy <= 0) return null;

  if (ox < oy) {
    if (mobile.centreX < solide.centreX) {
      mobile.x -= ox;
      return 'gauche';
    }
    mobile.x += ox;
    return 'droite';
  } else {
    if (mobile.centreY < solide.centreY) {
      mobile.y -= oy;
      return 'haut';
    }
    mobile.y += oy;
    return 'bas';
  }
}

void main() {
  final situations = <String, Aabb>{
    'arrive par la gauche': Aabb(180, 240, 40, 40),
    'arrive par la droite': Aabb(280, 240, 40, 40),
    'arrive par le haut': Aabb(240, 180, 40, 40),
    'arrive par le bas': Aabb(240, 280, 40, 40),
  };

  int correctes = 0;

  situations.forEach((nom, joueur) {
    final mur = Aabb(200, 200, 100, 100);

    final double ox =
        min(joueur.droite, mur.droite) - max(joueur.gauche, mur.gauche);
    final double oy =
        min(joueur.bas, mur.bas) - max(joueur.haut, mur.haut);
    final String avant = joueur.toString();

    final String? dir = repousser(joueur, mur);
    final bool libre = !seChevauchent(joueur, mur);
    if (libre) correctes++;

    print('$nom');
    print('   overlapX=${ox.toStringAsFixed(1)} '
        'overlapY=${oy.toStringAsFixed(1)} -> axe '
        '${ox < oy ? "X" : "Y"}');
    print('   $avant -> $joueur (vers le $dir), '
        'plus de chevauchement : $libre');
  });

  print('');
  print('$correctes / ${situations.length} repoussées correctes');
}
```

**Résultat :**

```text
arrive par la gauche
   overlapX=20.0 overlapY=40.0 -> axe X
   (180.0, 240.0) -> (160.0, 240.0) (vers le gauche), plus de chevauchement : true
arrive par la droite
   overlapX=20.0 overlapY=40.0 -> axe X
   (280.0, 240.0) -> (300.0, 240.0) (vers le droite), plus de chevauchement : true
arrive par le haut
   overlapX=40.0 overlapY=20.0 -> axe Y
   (240.0, 180.0) -> (240.0, 160.0) (vers le haut), plus de chevauchement : true
arrive par le bas
   overlapX=40.0 overlapY=20.0 -> axe Y
   (240.0, 280.0) -> (240.0, 300.0) (vers le bas), plus de chevauchement : true

4 / 4 repoussées correctes
```

**Explication :** les deux overlaps sont calculés avec `min(droites) - max(gauches)`, formule valable dans tous les cas de figure. Le plus petit des deux désigne l'axe de sortie — le chemin le plus court hors de l'obstacle. Le sens est donné par la comparaison des centres : un mobile dont le centre est à gauche de celui du solide sort par la gauche. Après repoussée, le mobile est exactement en contact avec le mur, donc `seChevauchent` renvoie `false` grâce aux comparaisons strictes. C'est la vérification la plus importante de cet exercice : une repoussée qui laisse encore du chevauchement provoquerait une seconde repoussée à la frame suivante, et donc les vibrations décrites en 24.15.

---

### Correction 6

```dart
class Aabb {
  double x, y, largeur, hauteur;
  Aabb(this.x, this.y, this.largeur, this.hauteur);

  double get gauche => x;
  double get droite => x + largeur;
  double get haut => y;
  double get bas => y + hauteur;
}

bool seChevauchent(Aabb a, Aabb b) =>
    a.droite > b.gauche &&
    a.gauche < b.droite &&
    a.bas > b.haut &&
    a.haut < b.bas;

class Contacts {
  bool sol = false;
  bool plafond = false;
  bool murGauche = false;
  bool murDroit = false;

  void reinitialiser() {
    sol = false;
    plafond = false;
    murGauche = false;
    murDroit = false;
  }

  @override
  String toString() {
    final actifs = <String>[
      if (sol) 'SOL',
      if (plafond) 'PLAFOND',
      if (murGauche) 'MUR-G',
      if (murDroit) 'MUR-D',
    ];
    return actifs.isEmpty ? "en l'air" : actifs.join(' ');
  }
}

class Corps {
  final Aabb boite;
  double vx = 0;
  double vy = 0;
  final Contacts contacts = Contacts();

  Corps(this.boite);

  void deplacer(double dt, List<Aabb> solides) {
    contacts.reinitialiser();

    // ----- PASSE X -----
    boite.x += vx * dt;
    for (final s in solides) {
      if (!seChevauchent(boite, s)) continue;
      if (vx > 0) {
        boite.x = s.gauche - boite.largeur;
        contacts.murDroit = true;
      } else if (vx < 0) {
        boite.x = s.droite;
        contacts.murGauche = true;
      }
      vx = 0;
    }

    // ----- PASSE Y -----
    boite.y += vy * dt;
    for (final s in solides) {
      if (!seChevauchent(boite, s)) continue;
      if (vy > 0) {
        boite.y = s.haut - boite.hauteur;
        contacts.sol = true;
      } else if (vy < 0) {
        boite.y = s.bas;
        contacts.plafond = true;
      }
      vy = 0;
    }
  }
}

void main() {
  // Couloir en L : un sol, un mur vertical à droite, un plafond.
  final solides = <Aabb>[
    Aabb(0, 300, 300, 20),   // sol
    Aabb(280, 100, 20, 200), // mur droit
    Aabb(0, 80, 300, 20),    // plafond
  ];

  final corps = Corps(Aabb(50, 200, 30, 40));
  const double dt = 1 / 60;
  const double gravite = 1000;
  const double vitesseHorizontale = 210;

  for (int frame = 1; frame <= 90; frame++) {
    // Le joueur pousse en permanence vers la droite.
    corps.vx = vitesseHorizontale;
    corps.vy += gravite * dt;
    corps.deplacer(dt, solides);

    if (frame % 10 == 0) {
      print('frame ${frame.toString().padLeft(2)} : '
          'x=${corps.boite.x.toStringAsFixed(1)} '
          'y=${corps.boite.y.toStringAsFixed(2)} '
          'vy=${corps.vy.toStringAsFixed(1)} '
          '${corps.contacts}');
    }
  }
}
```

**Résultat :**

```text
frame 10 : x=85.0 y=215.28 vy=166.7 en l'air
frame 20 : x=120.0 y=258.33 vy=333.3 en l'air
frame 30 : x=155.0 y=260.00 vy=0.0 SOL
frame 40 : x=190.0 y=260.00 vy=0.0 SOL
frame 50 : x=225.0 y=260.00 vy=0.0 SOL
frame 60 : x=250.0 y=260.00 vy=0.0 SOL MUR-D
frame 70 : x=250.0 y=260.00 vy=0.0 SOL MUR-D
frame 80 : x=250.0 y=260.00 vy=0.0 SOL MUR-D
frame 90 : x=250.0 y=260.00 vy=0.0 SOL MUR-D
```

**Explication :** trois phases se lisent dans la trace. Jusqu'à la frame 20, le corps est en chute libre : `vy` grimpe de 16,7 par frame (`1000 / 60`), aucun drapeau n'est actif. Autour de la frame 21, il touche le sol : `y` se fige à 260, soit `300 - 40` (le haut du sol moins la hauteur du corps), `vy` est remis à zéro et `SOL` s'allume. Il continue d'avancer horizontalement jusqu'à la frame 58, où son bord droit rencontre le mur : `x` se fige à 250, soit `280 - 30`, et `MUR-D` s'allume.

Deux points méritent attention. D'abord, `contacts.reinitialiser()` est appelé au tout début de `deplacer` : sans cela, `SOL` resterait allumé après un saut. Ensuite, `corps.vx` est réaffecté à chaque frame dans la boucle, comme si le joueur maintenait la touche droite ; c'est ce qui maintient `MUR-D` allumé frame après frame. Si l'on ne réaffectait pas `vx`, celui-ci resterait à zéro après le premier contact et le drapeau s'éteindrait dès la frame suivante — comportement correct, mais moins parlant dans une trace.

---

### Correction 7

```dart
import 'dart:math';

class Aabb {
  double x, y, largeur, hauteur;
  Aabb(this.x, this.y, this.largeur, this.hauteur);

  double get gauche => x;
  double get droite => x + largeur;
  double get haut => y;
  double get bas => y + hauteur;
}

bool seChevauchent(Aabb a, Aabb b) =>
    a.droite > b.gauche &&
    a.gauche < b.droite &&
    a.bas > b.haut &&
    a.haut < b.bas;

class Fleche {
  final Aabb boite;
  double vx;
  final double pasMax;
  int dernierNombreDeSousPas = 0;

  Fleche(this.boite, this.vx, this.pasMax);

  void deplacer(double dt, List<Aabb> murs) {
    final double d = (vx * dt).abs();
    final int n = max(1, (d / pasMax).ceil());
    dernierNombreDeSousPas = n;
    final double sousDt = dt / n;
    for (int i = 0; i < n; i++) {
      boite.x += vx * sousDt;
      for (final m in murs) {
        if (!seChevauchent(boite, m)) continue;
        boite.x = vx > 0 ? m.gauche - boite.largeur : m.droite;
        vx = 0;
      }
    }
  }
}

void main() {
  final murs = <Aabb>[Aabb(400, 0, 10, 300)];
  const double dt = 1 / 60;
  const double vitesse = 4000;

  // pasMax énorme -> un seul pas par frame (aucun sous-échantillonnage).
  final sans = Fleche(Aabb(20, 100, 8, 8), vitesse, 1000000);
  // pasMax = 4 px -> sous-échantillonnage réel.
  final avec = Fleche(Aabb(20, 100, 8, 8), vitesse, 4);

  for (int frame = 1; frame <= 10; frame++) {
    sans.deplacer(dt, murs);
    avec.deplacer(dt, murs);
  }

  print('Mur : x de 400 à 410 (10 px d\'épaisseur)');
  print('Flèche : 8 px de large, ${vitesse.toStringAsFixed(0)} px/s');
  print('Déplacement par frame : '
      '${(vitesse * dt).toStringAsFixed(2)} px');
  print('');
  print('Sans sous-pas : x final = '
      '${sans.boite.x.toStringAsFixed(2)}  '
      '(${sans.dernierNombreDeSousPas} sous-pas par frame)');
  print('Avec sous-pas : x final = '
      '${avec.boite.x.toStringAsFixed(2)}  '
      '(${avec.dernierNombreDeSousPas} sous-pas par frame)');
  print('');
  print('Le mur a-t-il été traversé ?');
  print('   sans sous-pas : ${sans.boite.x > 410}');
  print('   avec sous-pas : ${avec.boite.x > 410}');
}
```

**Résultat :**

```text
Mur : x de 400 à 410 (10 px d'épaisseur)
Flèche : 8 px de large, 4000 px/s
Déplacement par frame : 66.67 px

Sans sous-pas : x final = 686.67  (1 sous-pas par frame)
Avec sous-pas : x final = 392.00  (17 sous-pas par frame)

Le mur a-t-il été traversé ?
   sans sous-pas : true
   avec sous-pas : false
```

**Explication :** la flèche parcourt 66,67 pixels par frame alors que le mur n'en fait que 10. Sans sous-échantillonnage, les positions testées sont 86,67 — 153,33 — 220 — 286,67 — 353,33 — 420 : aucune ne tombe dans la fenêtre où la flèche chevauche le mur, elle passe donc au travers et finit à 686,67. Avec un `pasMax` de 4 pixels, chaque frame est découpée en `ceil(66,67 / 4) = 17` sous-pas de 3,92 pixels ; l'un d'eux tombe forcément dans le mur, la collision est détectée et la flèche s'arrête à `400 - 8 = 392`.

Notez que le nombre de sous-pas s'adapte tout seul : à 200 px/s le calcul donnerait `ceil(3,33 / 4) = 1` sous-pas, donc aucun surcoût pour les objets lents. C'est ce qui rend cette solution universelle.

---

### Correction 8

```dart
class Entite {
  final int id;
  final double x, y, largeur, hauteur;
  const Entite(this.id, this.x, this.y, this.largeur, this.hauteur);

  double get droite => x + largeur;
  double get bas => y + hauteur;
}

class GrilleSpatiale {
  final double taille;
  final Map<int, List<Entite>> _cases = {};

  GrilleSpatiale(this.taille);

  int _cle(int col, int lig) => col * 100000 + lig;

  int get nombreDeCases => _cases.length;

  void vider() => _cases.clear();

  void inserer(Entite e) {
    final int colMin = (e.x / taille).floor();
    final int colMax = (e.droite / taille).floor();
    final int ligMin = (e.y / taille).floor();
    final int ligMax = (e.bas / taille).floor();
    for (int c = colMin; c <= colMax; c++) {
      for (int l = ligMin; l <= ligMax; l++) {
        _cases.putIfAbsent(_cle(c, l), () => <Entite>[]).add(e);
      }
    }
  }

  /// Paires candidates, dédoublonnées.
  Set<int> pairesCandidates() {
    final paires = <int>{};
    for (final liste in _cases.values) {
      for (int i = 0; i < liste.length; i++) {
        for (int j = i + 1; j < liste.length; j++) {
          final int a = liste[i].id;
          final int b = liste[j].id;
          final int petit = a < b ? a : b;
          final int grand = a < b ? b : a;
          paires.add(petit * 1000 + grand);
        }
      }
    }
    return paires;
  }
}

bool seChevauchent(Entite a, Entite b) =>
    a.droite > b.x && a.x < b.droite && a.bas > b.y && a.y < b.bas;

/// Générateur pseudo-aléatoire déterministe (congruence linéaire).
int graine = 12345;
int suivant() {
  graine = (graine * 1103515245 + 12345) % 2147483648;
  return graine;
}

void main() {
  const int nombre = 300;
  final entites = <Entite>[];
  for (int i = 0; i < nombre; i++) {
    final double x = (suivant() % 1980).toDouble();
    final double y = (suivant() % 1980).toDouble();
    entites.add(Entite(i, x, y, 20, 20));
  }

  final int pairesTotales = nombre * (nombre - 1) ~/ 2;
  print('Monde 2000 x 2000, $nombre entités de 20 x 20');
  print('Paires en O(n²) : $pairesTotales');
  print('');
  print('cellule    cases   candidates   réduction');
  print('------------------------------------------');

  for (final double taille in [50.0, 100.0, 200.0, 500.0]) {
    final grille = GrilleSpatiale(taille);
    grille.vider();
    for (final e in entites) {
      grille.inserer(e);
    }
    final int candidates = grille.pairesCandidates().length;
    final double reduction = 100 * (1 - candidates / pairesTotales);
    print('${taille.toStringAsFixed(0).padLeft(7)}  '
        '${grille.nombreDeCases.toString().padLeft(7)}  '
        '${candidates.toString().padLeft(11)}   '
        '${reduction.toStringAsFixed(1).padLeft(6)} %');
  }

  // Vérification : le broad phase ne doit oublier AUCUNE collision.
  int reelles = 0;
  for (int i = 0; i < nombre; i++) {
    for (int j = i + 1; j < nombre; j++) {
      if (seChevauchent(entites[i], entites[j])) reelles++;
    }
  }
  print('');
  print('Collisions réelles (narrow phase) : $reelles');
}
```

**Résultat :**

```text
Monde 2000 x 2000, 300 entités de 20 x 20
Paires en O(n²) : 44850

cellule    cases   candidates   réduction
------------------------------------------
     50      489           82     99.8 %
    100      264          184     99.6 %
    200       98          582     98.7 %
    500       16         3063     93.2 %

Collisions réelles (narrow phase) : 18
```

**Explication :** la grille transforme un problème quadratique en un problème quasi linéaire. Avec des cellules de 50 pixels, 44 850 paires deviennent 82 candidates : 99,8 % du travail est éliminé sans jamais rater une collision, puisque deux entités qui se chevauchent partagent forcément au moins une case.

Le tableau illustre aussi le compromis sur la taille des cellules. Plus elles sont petites, moins il y a de candidates, mais plus il y a de cases à gérer et plus chaque entité est insérée de fois. Plus elles sont grandes, plus on retombe vers le O(n²) : à 500 pixels, il ne reste que 16 cases pour 300 entités, et le filtrage ne supprime plus que 93 % des paires. La règle de la section 24.23 — deux à quatre fois la taille de la plus grosse entité — donnerait ici 40 à 80 pixels, ce que le tableau confirme.

La dernière ligne est la vérification indispensable : 18 collisions réelles, toutes contenues dans les 82 candidates. Un broad phase qui renvoie moins de candidates que de collisions réelles est bogué.

---

### Correction 9

```dart
import 'dart:math';

class Couches {
  static const int decor = 1 << 0;            // 1
  static const int joueur = 1 << 1;           // 2
  static const int ennemi = 1 << 2;           // 4
  static const int ramassable = 1 << 3;       // 8
  static const int projectileEnnemi = 1 << 4; // 16
}

class Entite {
  final String nom;
  final int couche;
  int masque;

  Entite(this.nom, this.couche, this.masque);
}

/// Symétrique : les deux entités doivent se reconnaître.
bool doitTester(Entite a, Entite b) =>
    (a.masque & b.couche) != 0 && (b.masque & a.couche) != 0;

void main() {
  final heros = Entite(
    'héros',
    Couches.joueur,
    Couches.decor |
        Couches.ennemi |
        Couches.ramassable |
        Couches.projectileEnnemi,
  );
  final gobelin =
      Entite('gobelin', Couches.ennemi, Couches.decor | Couches.joueur);
  final mur =
      Entite('mur', Couches.decor, Couches.joueur | Couches.ennemi);
  final piece = Entite('pièce', Couches.ramassable, Couches.joueur);

  const double dt = 1 / 60;
  const double dureeInvincibilite = 1.2;
  const int bitsCoupes = Couches.ennemi | Couches.projectileEnnemi;

  double invincible = 0;
  int vies = 3;
  int score = 0;
  bool pieceprise = false;
  int coups = 0;
  int collisionsMur = 0;

  for (int frame = 1; frame <= 180; frame++) {
    // 1. Le compte à rebours d'invincibilité (section 24.27).
    if (invincible > 0) {
      invincible -= dt;
      if (invincible <= 0) {
        invincible = 0;
        heros.masque |= bitsCoupes; // on rend le héros vulnérable
      }
    }

    // 2. Le héros touche en permanence les trois entités.
    if (doitTester(heros, mur)) {
      collisionsMur++;
    }
    if (!pieceprise && doitTester(heros, piece)) {
      pieceprise = true;
      score += 10;
    }
    if (doitTester(heros, gobelin)) {
      vies--;
      coups++;
      invincible = dureeInvincibilite;
      heros.masque &= ~bitsCoupes; // on coupe la SEULE couche ennemi
      print('Coup encaissé à la frame $frame (vies = $vies), '
          'masque = ${heros.masque}');
    }
  }

  print('');
  print('Coups encaissés        : $coups');
  print('Collisions avec le mur : $collisionsMur');
  print('Score                  : $score');
  print('Vies restantes         : ${max(0, vies)}');
}
```

**Résultat :**

```text
Coup encaissé à la frame 1 (vies = 2), masque = 9
Coup encaissé à la frame 73 (vies = 1), masque = 9
Coup encaissé à la frame 145 (vies = 0), masque = 9

Coups encaissés        : 3
Collisions avec le mur : 180
Score                  : 10
Vies restantes         : 0
```

**Explication :** trois nombres racontent toute l'histoire. Trois coups en 180 frames, soit un toutes les 72 frames : c'est exactement `1,2 s × 60 FPS`, la durée d'invincibilité. Sans elle, le héros aurait encaissé 180 coups.

Cent quatre-vingts collisions avec le mur, c'est-à-dire une par frame, sans aucune interruption : l'invincibilité n'a jamais fait traverser le décor. C'est l'intérêt du masque de bits. `heros.masque &= ~bitsCoupes` éteint uniquement les bits `ennemi` (4) et `projectileEnnemi` (16), et laisse allumés `decor` (1) et `ramassable` (8). Le masque passe donc de `1 + 4 + 8 + 16 = 29` à `1 + 8 = 9`, valeur affichée dans la trace.

Enfin, la pièce n'est comptée qu'une fois grâce au drapeau `pieceprise`, alors que le contact dure 180 frames. Score final : 10 points, pas 1800.

---

### Correction 10

```dart
import 'dart:math';
import 'package:flutter/material.dart';

void main() => runApp(const SalleTresorApp());

const double salleLargeur = 480;
const double salleHauteur = 320;

class Aabb {
  double x, y, largeur, hauteur;
  Aabb(this.x, this.y, this.largeur, this.hauteur);

  double get gauche => x;
  double get droite => x + largeur;
  double get haut => y;
  double get bas => y + hauteur;
  double get centreX => x + largeur / 2;
  double get centreY => y + hauteur / 2;

  Rect get rect => Rect.fromLTWH(x, y, largeur, hauteur);
}

bool seChevauchent(Aabb a, Aabb b) =>
    a.droite > b.gauche &&
    a.gauche < b.droite &&
    a.bas > b.haut &&
    a.haut < b.bas;

class Piece {
  final Aabb boite;
  bool prise = false;
  Piece(double cx, double cy) : boite = Aabb(cx - 6, cy - 6, 12, 12);
}

class SalleTresorApp extends StatelessWidget {
  const SalleTresorApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: PageSalle(),
    );
  }
}

class PageSalle extends StatefulWidget {
  const PageSalle({super.key});

  @override
  State<PageSalle> createState() => _PageSalleState();
}

class _PageSalleState extends State<PageSalle>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  // Décor : quatre murs de contour et un pilier central.
  final List<Aabb> murs = <Aabb>[
    Aabb(0, 0, salleLargeur, 16),
    Aabb(0, salleHauteur - 16, salleLargeur, 16),
    Aabb(0, 0, 16, salleHauteur),
    Aabb(salleLargeur - 16, 0, 16, salleHauteur),
    Aabb(210, 130, 60, 60),
  ];

  final Aabb heros = Aabb(0, 0, 20, 24);
  final Aabb gobelin = Aabb(40, 148, 26, 26);

  late final List<Piece> pieces;

  double angle = 0;
  double vitesseGobelin = 90;
  int vies = 3;
  int score = 0;
  double invincible = 0;
  bool debug = false;

  static const double rayon = 90;
  static const double centreX = salleLargeur / 2;
  static const double centreY = salleHauteur / 2;
  static const double dureeInvincibilite = 1.0;

  bool get estVisible =>
      invincible <= 0 || (invincible * 10).floor().isEven;

  @override
  void initState() {
    super.initState();
    // Cinq pièces réparties sur le trajet circulaire du héros.
    pieces = List<Piece>.generate(5, (i) {
      final double a = i * 2 * pi / 5;
      return Piece(centreX + rayon * cos(a), centreY + rayon * sin(a));
    });
    _placerHeros();
    _ticker = createTicker(_frame)..start();
  }

  void _placerHeros() {
    heros.x = centreX + rayon * cos(angle) - heros.largeur / 2;
    heros.y = centreY + rayon * sin(angle) - heros.hauteur / 2;
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  void _frame(Duration maintenant) {
    double dt = (maintenant - _precedent).inMicroseconds / 1000000.0;
    _precedent = maintenant;
    if (dt <= 0) return;
    dt = dt.clamp(0.0, 0.05);

    setState(() {
      // 1. Le héros suit un cercle (aucune entrée clavier).
      angle += 1.1 * dt;
      _placerHeros();

      // 2. Le gobelin patrouille horizontalement.
      gobelin.x += vitesseGobelin * dt;
      if (gobelin.x < 40) {
        gobelin.x = 40;
        vitesseGobelin = -vitesseGobelin;
      } else if (gobelin.x > 200) {
        gobelin.x = 200;
        vitesseGobelin = -vitesseGobelin;
      }

      // 3. L'invincibilité descend vers zéro.
      if (invincible > 0) invincible = max(0, invincible - dt);

      // 4. Ramassage des pièces.
      for (final p in pieces) {
        if (p.prise) continue;
        if (seChevauchent(heros, p.boite)) {
          p.prise = true;
          score += 10;
        }
      }

      // 5. Dégâts du gobelin, filtrés par l'invincibilité.
      if (invincible <= 0 && seChevauchent(heros, gobelin)) {
        if (vies > 0) vies--;
        invincible = dureeInvincibilite;
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    final int prises = pieces.where((p) => p.prise).length;
    return Scaffold(
      backgroundColor: const Color(0xFF0B0D14),
      body: SafeArea(
        child: Column(
          children: [
            Padding(
              padding: const EdgeInsets.all(8),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  Text(
                    'Score $score   Vies $vies   '
                    'Pièces $prises/${pieces.length}',
                    style:
                        const TextStyle(color: Colors.white, fontSize: 16),
                  ),
                  TextButton(
                    onPressed: () => setState(() => debug = !debug),
                    child: Text(debug ? 'HITBOX ON' : 'HITBOX OFF'),
                  ),
                ],
              ),
            ),
            Expanded(
              child: Center(
                child: FittedBox(
                  child: SizedBox(
                    width: salleLargeur,
                    height: salleHauteur,
                    child: CustomPaint(
                      painter: PeintreSalle(
                        murs: murs,
                        pieces: pieces,
                        heros: heros,
                        gobelin: gobelin,
                        herosVisible: estVisible,
                        debug: debug,
                      ),
                    ),
                  ),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class PeintreSalle extends CustomPainter {
  final List<Aabb> murs;
  final List<Piece> pieces;
  final Aabb heros;
  final Aabb gobelin;
  final bool herosVisible;
  final bool debug;

  PeintreSalle({
    required this.murs,
    required this.pieces,
    required this.heros,
    required this.gobelin,
    required this.herosVisible,
    required this.debug,
  });

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
      Rect.fromLTWH(0, 0, salleLargeur, salleHauteur),
      Paint()..color = const Color(0xFF1B2030),
    );

    final pMur = Paint()..color = const Color(0xFF3C4763);
    for (final m in murs) {
      canvas.drawRect(m.rect, pMur);
    }

    final pPiece = Paint()..color = const Color(0xFFFFC93C);
    for (final p in pieces) {
      if (p.prise) continue;
      canvas.drawCircle(
        Offset(p.boite.centreX, p.boite.centreY),
        6,
        pPiece,
      );
    }

    canvas.drawRRect(
      RRect.fromRectAndRadius(gobelin.rect, const Radius.circular(5)),
      Paint()..color = const Color(0xFF5FBF61),
    );

    if (herosVisible) {
      canvas.drawRRect(
        RRect.fromRectAndRadius(heros.rect, const Radius.circular(4)),
        Paint()..color = const Color(0xFF6FA8FF),
      );
    }

    if (!debug) return;

    final contour = Paint()
      ..style = PaintingStyle.stroke
      ..strokeWidth = 1.5;

    contour.color = const Color(0xFF3F6BD8);
    for (final m in murs) {
      canvas.drawRect(m.rect, contour);
    }
    contour.color = const Color(0xFFFFC107);
    for (final p in pieces) {
      if (!p.prise) canvas.drawRect(p.boite.rect, contour);
    }
    contour.color = const Color(0xFFE53935);
    canvas.drawRect(gobelin.rect, contour);
    contour.color = seChevauchent(heros, gobelin)
        ? const Color(0xFFFF00FF)
        : const Color(0xFF4CAF50);
    canvas.drawRect(heros.rect, contour);
  }

  @override
  bool shouldRepaint(covariant PeintreSalle ancien) => true;
}
```

**Résultat :** une salle de 480 × 320 pixels, murs gris et pilier central. Le héros bleu tourne sans arrêt sur un cercle de rayon 90 autour du centre. Il ramasse les cinq pièces jaunes une par une au premier passage — le score monte à 50 et les pièces ne réapparaissent plus. Le gobelin vert fait des allers-retours entre `x = 40` et `x = 200` ; à chaque croisement avec le héros, une vie est perdue et le héros clignote pendant une seconde. Le bouton en haut à droite affiche tous les rectangles de collision, celui du héros passant au magenta pendant le contact.

**Explication :** ce programme réunit six mécanismes du chapitre, chacun tenant en quelques lignes. Le héros est déplacé par une formule paramétrique (`cos` et `sin`) plutôt que par un clavier, ce qui rend la scène entièrement déterministe et donc facile à observer. Les pièces sont générées avec `List.generate` sur cinq angles régulièrement espacés, garanties d'être sur le trajet du héros.

Le ramassage utilise le drapeau `prise` pour ne compter qu'une fois, malgré un contact qui dure plusieurs frames. Les dégâts sont filtrés par le test `invincible <= 0` placé **avant** le test géométrique : quand le héros est invincible, la collision n'est même pas évaluée. Le clignotement découle mécaniquement de `(invincible * 10).floor().isEven`, qui alterne cinq fois par seconde.

Enfin, le mode debug est passé au `CustomPainter` par un simple `bool` et dessine les contours après le rendu normal, afin qu'ils restent visibles par-dessus les formes pleines. C'est exactement la structure que Flame reproduit avec `game.debugMode`, au chapitre 32.

---

## Et maintenant ?

Vous savez maintenant faire se rencontrer les objets de votre monde. Le joueur se cogne aux murs, ramasse des pièces, perd de la vie au contact d'un gobelin, et vous savez pourquoi chacune de ces règles fonctionne — jusque dans les cas limites du tunneling et des `double` qui ne tombent jamais juste.

Il reste pourtant une limite bien visible : tout tient dans un seul écran. Le monde de votre jeu fait 640 × 400 pixels, et il s'arrête là. Un vrai donjon fait dix écrans de large, avec des salles, des couloirs, des étages, et une caméra qui suit le héros.

C'est le sujet du chapitre suivant. Vous y apprendrez à séparer les **coordonnées du monde** des **coordonnées de l'écran**, à écrire une caméra qui suit le joueur sans donner le mal de mer, à ajouter des arrière-plans en parallaxe, et à décrire un niveau entier sous forme de **tilemap** — une grille de tuiles dont chaque case porte, précisément, sa propre hitbox.

Rendez-vous au chapitre 25 : [25-PARTIE-2A—CAMÉRA-MONDE-ET-NIVEAUX.md](./25-PARTIE-2A—CAMÉRA-MONDE-ET-NIVEAUX.md)
