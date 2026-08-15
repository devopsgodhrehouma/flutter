# PARTIE 2C — LE JEU COMPLET « DONJON DE DART »
# CHAPITRE 37 — LES ENNEMIS : IA ET COMBAT

> **Niveau :** intermédiaire / avancé
> **Durée estimée :** 10 h
> **Pré-requis :** chapitres 35 et 36 (architecture du jeu, joueur), chapitre 11 (mixins, classes abstraites, enums), chapitre 23 (vecteurs, normalisation, knockback), chapitre 26 (machines à états), chapitre 32 (collisions, raycast), chapitre 33 (effets, timers, cooldowns)
> **Version de Flame utilisée :** **1.38.0**
> **Ce que vous saurez faire à la fin :** peupler la salle du « Donjon de Dart » d'ennemis qui patrouillent, vous repèrent, vous poursuivent, vous frappent, vous tirent dessus, encaissent vos coups et meurent proprement — le tout sans une seule image.

---

## 37.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- décrire précisément l'état du projet à la fin du chapitre 36 et ce que le chapitre 37 y ajoute ;
- écrire un mixin `Sante` réutilisable et justifier ce choix face à une classe mère ;
- appliquer `Sante` au joueur en remplacement du champ `pv` écrit à la main au chapitre 36 ;
- concevoir une classe abstraite `Ennemi` qui factorise tout ce que les ennemis ont en commun ;
- déclarer et utiliser une méthode abstraite `mettreAJourIA(double dt)` ;
- écrire un ennemi terrestre `Gobelin` soumis à la gravité ;
- faire patrouiller un ennemi entre deux bornes calculées à partir de son point d'apparition ;
- détecter le vide devant un ennemi et lui faire faire demi-tour au bord d'une plateforme ;
- lancer des rayons avec `raycast` pour sonder le sol, les murs et la ligne de vue ;
- implémenter un rayon d'aggro et une ligne de vue non traversante ;
- écrire une poursuite qui ne tremble pas et qui s'arrête au bord du vide ;
- construire une machine à états à cinq états : `patrouille`, `poursuite`, `attaque`, `retour`, `mort` ;
- dessiner et lire un schéma de transitions ;
- écrire un ennemi volant `Chauvesouris` affranchi de la gravité ;
- produire un mouvement sinusoïdal stable, sans dérive ;
- déclencher un piqué vers une position mémorisée du joueur ;
- écrire un composant `Projectile` autonome ;
- calculer une direction de tir normalisée (rappel du chapitre 23) ;
- limiter la durée de vie d'un projectile de trois manières différentes ;
- écrire un cooldown de tir et expliquer pourquoi il ne doit jamais vivre dans `onCollision` ;
- établir un tableau des couches de collision et le traduire en `CollisionType` ;
- créer une hitbox d'attaque temporaire qui ne blesse chaque ennemi qu'une fois ;
- infliger des dégâts de contact sans vider la barre de vie en trois frames ;
- appliquer un knockback cohérent des deux côtés ;
- déclencher un flash blanc et un clignotement d'invincibilité ;
- faire mourir un ennemi avec un effet, un gain de score et un retrait propre ;
- implémenter `perdreUneVie()` et le respawn du joueur ;
- définir et utiliser un point de réapparition ;
- équilibrer le combat à l'aide d'un tableau de PV et de dégâts ;
- livrer un jeu jouable de bout en bout, sans aucun fichier image.

---

## 37.1 — Où on en est et ce qu'on ajoute

### L'état du projet à la fin du chapitre 36

À la fin du chapitre 36, vous disposez d'un projet qui compile, se lance et se joue déjà un peu.

```text
  CE QUI EXISTE À LA FIN DU CHAPITRE 36

  donjon_de_dart/
  ├── pubspec.yaml
  └── lib/
      ├── main.dart                     GameWidget + overlayBuilderMap
      ├── donjon_game.dart              DonjonGame : FlameGame, monde, camera, etat
      ├── config/
      │   ├── constantes.dart           gravite, vitesseJoueur, forceSaut, pvJoueurMax...
      │   └── palette.dart              les couleurs du jeu
      ├── core/
      │   ├── game_state.dart           enum GameState
      │   └── entite.dart               base commune (ch. 36)
      ├── composants/
      │   ├── joueur.dart               Joueur + enum EtatJoueur
      │   └── plateforme.dart           Plateforme (hitbox passive)
      └── ecrans/
          └── menu_principal.dart       le menu (overlay Flutter)
```

Concrètement, quand vous lancez `flutter run` :

1. le menu principal s'affiche par-dessus le canvas ;
2. « Jouer » appelle `demarrerPartie()`, qui construit une salle de plateformes et y place le joueur ;
3. le joueur court à gauche et à droite, saute, retombe, ne traverse pas les plateformes ;
4. il possède une animation d'état (`immobile`, `marche`, `saut`, `chute`, `attaque`) rendue avec des rectangles colorés ;
5. la touche d'attaque joue l'état `attaque`… **mais ne frappe rien**, puisqu'il n'y a rien à frapper ;
6. le champ `pv` existe sur le joueur, mais **personne ne lui retire jamais un point de vie**.

Autrement dit : vous avez un pantin très agréable à manipuler dans une pièce vide.

### Ce que le chapitre 37 ajoute

Ce chapitre remplit la pièce et donne un sens au bouton d'attaque.

| Fichier | Statut | Contenu |
| --- | --- | --- |
| `lib/core/sante.dart` | **créé** | le mixin `Sante` : `pvMax`, `pv`, `estVivant`, `subirDegats`, `soigner` |
| `lib/composants/ennemi.dart` | **créé** | l'enum `EtatEnnemi` et la classe abstraite `Ennemi` |
| `lib/composants/gobelin.dart` | **créé** | `Gobelin`, l'ennemi terrestre qui patrouille et poursuit |
| `lib/composants/chauvesouris.dart` | **créé** | `Chauvesouris`, l'ennemi volant qui ondule, tire et pique |
| `lib/composants/projectile.dart` | **créé** | `Projectile`, la boule tirée par la chauve-souris |
| `lib/composants/joueur.dart` | **modifié** | applique `Sante`, ajoute la hitbox d'attaque, les dégâts, la mort |
| `lib/donjon_game.dart` | **modifié** | `perdreUneVie()`, `ajouterScore()`, le point de réapparition, le peuplement de la salle |
| `lib/config/constantes.dart` | **modifié** | les constantes de combat |
| `lib/config/palette.dart` | **modifié** | les couleurs des ennemis et des projectiles |
| `lib/main.dart` | **modifié** | un overlay « Game Over » provisoire |

Trois fichiers du chapitre 36 ne bougent pas du tout : `core/game_state.dart`, `core/entite.dart` et `composants/plateforme.dart`.

### La règle qui gouverne tout le chapitre

> **Aucun fichier image, aucun fichier son.**
>
> Chaque ennemi est un `PositionComponent` invisible qui porte deux enfants : un `RectangleComponent` coloré (le corps) et une hitbox. Le jour où vous aurez de vrais sprites, vous remplacerez le `RectangleComponent` par un `SpriteAnimationComponent` et **rien d'autre ne changera**. Cette séparation entre le squelette logique et l'habillage est la seule raison pour laquelle ce chapitre peut être écrit sans assets.

### Rappel des deux fichiers du chapitre 36 dont nous allons nous servir

Le premier est la plateforme. C'est le décor solide : elle n'a aucune logique, seulement une hitbox **passive** (chapitre 32.9).

```dart
// lib/composants/plateforme.dart  (chapitre 36 — INCHANGÉ)
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flutter/material.dart';

import '../config/palette.dart';

class Plateforme extends PositionComponent {
  Plateforme({required Vector2 position, required Vector2 size})
      : super(position: position, size: size, anchor: Anchor.topLeft);

  @override
  Future<void> onLoad() async {
    add(
      RectangleComponent(
        size: size.clone(),
        paint: Paint()..color = Palette.plateforme,
      ),
    );
    add(RectangleHitbox(collisionType: CollisionType.passive));
  }
}
```

Le second est l'ossature de `DonjonGame`, dont voici les parties qui nous concernent.

```dart
// lib/donjon_game.dart  (extrait de l'état chapitre 35-36)
class DonjonGame extends FlameGame
    with HasCollisionDetection, HasKeyboardHandlerComponents {
  // `FlameGame` fournit déjà `world` et `camera`.
  // `monde` est simplement l'alias imposé par le cahier des charges du jeu.
  late final World monde = world;

  GameState etat = GameState.chargement;

  int score = 0;
  int vies = Constantes.viesDepart;
  int niveauCourant = 0;
  int meilleurScore = 0;

  late Joueur joueur;

  void changerEtat(GameState nouvelEtat) { /* ch. 35 */ }
  Future<void> demarrerPartie() async { /* ch. 35 */ }
}
```

> **Note d'architecture.** `FlameGame` possède déjà un champ `world` et un champ `camera`. Redéclarer `late final CameraComponent camera;` masquerait celui du moteur et casserait la caméra. Le cahier des charges du jeu parle de `monde` : nous en faisons donc un **alias** (`late final World monde = world;`), ce qui coûte zéro ligne à l'exécution et respecte les deux contraintes.

---

## 37.2 — `lib/core/sante.dart` : le mixin `Sante`

### Le besoin

Trois familles d'objets vont avoir des points de vie : le joueur, les ennemis, et plus tard le boss du chapitre 39. Toutes les trois ont besoin exactement des mêmes cinq choses :

```text
  pvMax          combien de points de vie au maximum
  pv             combien de points de vie maintenant
  estVivant      pv > 0 ?
  subirDegats()  retirer des points, sans jamais descendre sous 0
  soigner()      rendre des points, sans jamais dépasser pvMax
```

Écrire ces cinq membres trois fois serait une faute. Les mettre dans une classe mère serait une autre faute (section 37.3). La bonne réponse est un **mixin**, revu au chapitre 11.

### Le code

```dart
// lib/core/sante.dart
import 'package:flame/components.dart';

/// Points de vie d'une entité du donjon.
///
/// `on PositionComponent` : ce mixin ne peut être appliqué qu'à un composant
/// qui a une position — ce qui garantit l'accès à `position`, `size` et
/// `absoluteCenter` dans les classes qui l'utilisent.
mixin Sante on PositionComponent {
  late double pvMax;
  late double pv;

  /// Vrai tant qu'il reste au moins une fraction de point de vie.
  bool get estVivant => pv > 0;

  /// Rapport entre 0.0 et 1.0, utilisé par la barre de vie du chapitre 38.
  double get ratioPv => pvMax <= 0 ? 0 : (pv / pvMax).clamp(0.0, 1.0);

  /// À appeler UNE fois, dans `onLoad`, avant toute lecture de `pv`.
  void initialiserSante(double valeurMax) {
    pvMax = valeurMax;
    pv = valeurMax;
  }

  /// Retire des points de vie. Ignore les dégâts nuls, négatifs,
  /// et ceux infligés à un mort.
  void subirDegats(double degats) {
    if (degats <= 0 || !estVivant) {
      return;
    }
    pv -= degats;
    if (pv < 0) {
      pv = 0;
    }
    onDegatsRecus(degats);
    if (!estVivant) {
      onMort();
    }
  }

  /// Rend des points de vie, sans jamais dépasser `pvMax`.
  void soigner(double points) {
    if (points <= 0 || !estVivant) {
      return;
    }
    pv += points;
    if (pv > pvMax) {
      pv = pvMax;
    }
    onSoinRecu(points);
  }

  // ---- Points d'accroche, redéfinis par les classes qui utilisent le mixin --

  /// Appelé après chaque perte de points de vie effective.
  void onDegatsRecus(double degats) {}

  /// Appelé après chaque soin effectif.
  void onSoinRecu(double points) {}

  /// Appelé UNE SEULE FOIS, à l'instant exact où `pv` atteint 0.
  void onMort() {}
}
```

### Le patron « méthode modèle »

Ce mixin applique un patron classique, la **méthode modèle** (*template method*) : `subirDegats` fixe le déroulé une fois pour toutes — vérifier, retrancher, borner, prévenir — et laisse trois trous que chaque entité remplit à sa guise.

```text
  subirDegats(12)
      |
      +-- degats <= 0 ou déjà mort ?  -> on sort
      +-- pv -= 12
      +-- pv < 0 ?                    -> pv = 0
      +-- onDegatsRecus(12)  <---- le gobelin flashe en blanc,
      |                             le joueur clignote et recule
      +-- pv == 0 ?
              +-- onMort()   <---- le gobelin explose et donne du score,
                                   le joueur déclenche perdreUneVie()
```

Résultat : la règle « on ne descend jamais sous zéro » est écrite **une seule fois** dans tout le projet. Le jour où vous ajoutez une armure qui divise les dégâts par deux, vous la mettez ici et les quatre entités en bénéficient.

**Résultat attendu, en pseudo-console :**

```text
gobelin.pvMax = 30, pv = 30, estVivant = true
gobelin.subirDegats(25) -> pv = 5   -> onDegatsRecus(25)
gobelin.subirDegats(25) -> pv = 0   -> onDegatsRecus(25) puis onMort()
gobelin.subirDegats(25) -> ignoré   (déjà mort)
gobelin.soigner(10)     -> ignoré   (on ne soigne pas un mort)
```

### Le piège du `late`

Le cahier des charges impose `late double pvMax;` et `late double pv;`. Le mot-clé `late` (chapitre 12) signifie « je promets d'affecter cette variable avant de la lire ». Si vous lisez `pv` avant d'avoir appelé `initialiserSante`, Dart lève une exception à l'exécution :

```text
LateInitializationError: Field 'pv' has not been initialized.
```

D'où la règle absolue du chapitre :

> **Toute classe qui utilise `Sante` appelle `initialiserSante(...)` en toute première ligne de son `onLoad()`.**

---

## 37.3 — Pourquoi un mixin plutôt qu'une classe mère

La question mérite d'être posée sérieusement, parce que la réponse « parce que c'est plus joli » est fausse. Il existe trois raisons techniques.

### Raison 1 — L'héritage est déjà pris

`Joueur` et `Ennemi` héritent tous les deux de `PositionComponent`, imposé par Flame. Dart n'a pas d'héritage multiple (chapitre 11) : impossible d'écrire `class Joueur extends PositionComponent, EtreVivant`. Une classe mère `EtreVivant` obligerait à écrire :

```dart
// Ce qu'il faudrait faire avec une classe mère :
abstract class EtreVivant extends PositionComponent { /* pv, subirDegats... */ }

class Joueur extends EtreVivant { }
class Ennemi extends EtreVivant { }
```

Cela **marche**, mais crée un étage supplémentaire dans la hiérarchie, et la hiérarchie est la ressource la plus rare d'un projet de jeu : on n'en a qu'une.

### Raison 2 — La santé n'est pas une identité, c'est une capacité

Un gobelin **est** un ennemi. Un gobelin **a** de la santé. La phrase « un gobelin est un être vivant qui est un composant positionné » est un mensonge de modélisation : elle range dans l'arbre d'identité quelque chose qui relève d'une capacité transverse.

```text
  IDENTITÉ (héritage, "est un")        CAPACITÉ (mixin, "sait faire")

  PositionComponent                    Sante              -> a des PV
        |                              CollisionCallbacks -> réagit aux contacts
     Ennemi                            KeyboardHandler    -> lit le clavier
      /    \                           HasGameReference   -> connaît le jeu
  Gobelin  Chauvesouris
```

Le classement est net : à gauche ce qui définit **ce que la chose est**, à droite ce qu'elle **sait faire**. Flame lui-même est construit sur ce principe : `CollisionCallbacks`, `KeyboardHandler`, `HasGameReference` et `HasPaint` sont tous des mixins.

### Raison 3 — Un mixin se combine, une classe mère s'exclut

Au chapitre 38 vous allez écrire un coffre destructible, au chapitre 39 une porte qui casse. Ni l'un ni l'autre n'est un ennemi. Avec un mixin, il suffit d'écrire `with Sante` :

```dart
class Coffre extends PositionComponent with Sante { }      // possible
class Coffre extends EtreVivant { }                        // absurde
```

### La contrainte `on PositionComponent`

```dart
mixin Sante on PositionComponent { }
```

La clause `on` déclare une **contrainte de superclasse** : `Sante` ne peut être appliqué qu'à un `PositionComponent` ou l'un de ses descendants. Deux bénéfices :

- une erreur de compilation immédiate si vous l'appliquez à un `Component` sans position ;
- à l'intérieur du mixin, vous pouvez utiliser `position`, `size`, `absoluteCenter`, `add(...)` comme si vous étiez dans la classe.

### Le tableau de décision

| Question | Réponse | Outil |
| --- | --- | --- |
| Est-ce que ça définit ce que l'objet **est** ? | oui | classe mère (`Ennemi`) |
| Est-ce que ça définit ce que l'objet **sait faire** ? | oui | mixin (`Sante`) |
| Est-ce que plusieurs branches non apparentées en ont besoin ? | oui | mixin |
| Est-ce que la classe hérite déjà d'autre chose d'imposé ? | oui | mixin |
| Faut-il un constructeur, des paramètres obligatoires ? | oui | classe mère (un mixin n'a pas de constructeur) |
| Faut-il stocker de l'état ? | possible dans les deux | mixin (avec `late` ou une valeur par défaut) |

Dernière ligne à retenir : **un mixin ne peut pas avoir de constructeur**. C'est précisément pour cela que `Sante` expose `initialiserSante(...)` au lieu d'initialiser `pv` dans un constructeur.

---

## 37.4 — Appliquer `Sante` au joueur

Le chapitre 36 avait donné au joueur un champ écrit à la main :

```dart
// Chapitre 36 — AVANT
class Joueur extends PositionComponent
    with HasGameReference<DonjonGame>, KeyboardHandler, CollisionCallbacks {
  double pv = Constantes.pvJoueurMax;

  void subirDegats(double degats) {
    pv -= degats;              // rien ne borne à zéro
    if (pv <= 0) mourir();
  }
}
```

Trois lignes de diff suffisent à passer au mixin.

```dart
// Chapitre 37 — APRÈS
class Joueur extends PositionComponent
    with HasGameReference<DonjonGame>,
         Sante,                    // <-- 1. le mixin
         KeyboardHandler,
         CollisionCallbacks {
  // 2. le champ `pv` écrit à la main DISPARAÎT : il vient du mixin.

  @override
  Future<void> onLoad() async {
    initialiserSante(Constantes.pvJoueurMax);   // 3. obligatoire
    // ... le reste du onLoad du chapitre 36
  }
}
```

### Ce que le code appelant devient

Rien ne casse. `joueur.pv` existe toujours, `joueur.subirDegats(10)` aussi. En prime, trois nouveautés arrivent gratuitement :

| Expression | Avant (ch. 36) | Après (ch. 37) |
| --- | --- | --- |
| `joueur.pv` | champ de la classe | champ du mixin |
| `joueur.pvMax` | n'existait pas | disponible |
| `joueur.estVivant` | n'existait pas | disponible |
| `joueur.ratioPv` | n'existait pas | disponible (barre de vie du ch. 38) |
| `joueur.soigner(20)` | n'existait pas | disponible (potions du ch. 38) |
| `pv` négatif | possible | impossible |

### La redéfinition de `subirDegats` chez le joueur

Le joueur a une contrainte que le mixin ignore : l'**invincibilité temporaire**. Il redéfinit donc `subirDegats` pour filtrer, puis délègue au mixin.

```dart
@override
void subirDegats(double degats, {Vector2? direction}) {
  // Filtre 1 : pendant l'invincibilité, aucun dégât ne passe.
  if (invincible || !estVivant || etat == EtatJoueur.mort) {
    return;
  }

  super.subirDegats(degats);   // <-- le mixin fait le travail comptable

  if (!estVivant) {
    return;                    // onMort() a déjà été déclenché par le mixin
  }

  invincible = true;
  _tempsInvincible = Constantes.dureeInvincibilite;
  etat = EtatJoueur.touche;
  _tempsTouche = 0.25;
  _reculer(direction);
  _clignoter();
}
```

Deux points de langage méritent une remarque.

**Ajouter un paramètre optionnel dans une redéfinition est légal.** La signature du mixin est `void subirDegats(double degats)`. Celle du joueur est `void subirDegats(double degats, {Vector2? direction})`. Dart accepte une redéfinition qui **ajoute des paramètres optionnels** (nommés ou positionnels), parce que tout appel valide sur le type parent reste valide sur le type enfant. L'inverse — rendre un paramètre obligatoire — serait refusé.

**`super.subirDegats(...)` appelle bien le mixin.** Dans une application de mixins, `super` désigne le membre suivant dans la chaîne de linéarisation, ici celui de `Sante`. C'est exactement le mécanisme du chapitre 11.

### L'ordre des mixins

```dart
with HasGameReference<DonjonGame>, Sante, KeyboardHandler, CollisionCallbacks
```

L'ordre compte : le dernier mixin de la liste est le plus proche de la classe, donc « gagne » en cas de membres homonymes. Ici aucun des quatre ne déclare de membre commun, l'ordre est donc libre ; on garde celui du cahier des charges en insérant `Sante` juste après `HasGameReference`.

---

## 37.5 — `lib/composants/ennemi.dart` : la classe abstraite

### Ce qui est commun à tous les ennemis

Avant d'écrire une ligne, listons ce qu'un gobelin et une chauve-souris partagent réellement.

| Élément commun | Justification |
| --- | --- |
| des points de vie | `Sante` |
| une vitesse de déplacement | `double vitesse` |
| des dégâts infligés au contact | `double degatsContact` |
| des points de score à la mort | `int pointsScore` |
| un état d'IA | `EtatEnnemi etat` |
| une vélocité | `Vector2 velocite` |
| un sens de déplacement | `int sens` (+1 ou -1) |
| un point d'ancrage (là où il est né) | `Vector2 ancre` |
| un rayon d'aggro / d'abandon / d'attaque | trois `double` |
| un corps visible et une hitbox | deux composants enfants |
| la résolution des collisions avec les plateformes | une méthode |
| le knockback | une méthode |
| le flash blanc quand il est touché | `onDegatsRecus` |
| la mort : effet, score, retrait | `mourir()` |
| **la façon de décider où aller** | **différente pour chacun** |

La dernière ligne est celle qui justifie le mot `abstract` : tout est commun **sauf** la décision. Le gobelin marche au sol ; la chauve-souris ondule dans les airs. On factorise les quatorze premières lignes et on laisse un trou pour la quinzième.

### L'enum des états

```dart
// lib/composants/ennemi.dart (extrait)
enum EtatEnnemi { patrouille, poursuite, attaque, retour, mort }
```

### Le squelette de la classe

```dart
abstract class Ennemi extends PositionComponent
    with HasGameReference<DonjonGame>, Sante, CollisionCallbacks {
  Ennemi({
    required Vector2 position,
    required Vector2 taille,
    required this.vitesse,
    required this.degatsContact,
    required this.pointsScore,
    required this.couleur,
    required double pvDepart,
  })  : _pvDepart = pvDepart,
        super(position: position, size: taille, anchor: Anchor.topLeft);

  // --- Réglages de l'espèce, fournis par la sous-classe ---
  double vitesse;
  double degatsContact;
  int pointsScore;
  final Color couleur;
  final double _pvDepart;

  // --- État courant ---
  EtatEnnemi etat = EtatEnnemi.patrouille;
  Vector2 velocite = Vector2.zero();
  int sens = 1;
  bool auSol = false;
  late Vector2 ancre;

  // --- Portées, réglables par espèce dans onLoad ---
  double rayonAggro = 140;
  double rayonAbandon = 220;
  double rayonAttaque = 26;
  double demiPatrouille = 3 * Constantes.tailleTuile;

  /// Le gobelin tombe, la chauve-souris non.
  bool get subitGravite => true;

  // --- La décision : chaque espèce l'écrit à sa façon ---
  void mettreAJourIA(double dt);
}
```

Notez le paramètre `taille` plutôt que `size` : `size` est déjà le nom du champ de `PositionComponent`, et l'écrire `required super.size` empêcherait de le cloner. On préfère un nom local, transmis explicitement au `super`.

### Pourquoi `abstract` et non une classe normale

Trois effets concrets, tous vérifiables par le compilateur :

1. `Ennemi(...)` ne peut pas être instancié. Écrire `world.add(Ennemi(...))` est une **erreur de compilation**, pas un bug à l'exécution.
2. Toute sous-classe **doit** fournir `mettreAJourIA`. Oublier l'IA d'un nouvel ennemi devient impossible.
3. Le type `Ennemi` reste utilisable partout : `other is Ennemi`, `List<Ennemi>`, `game.monde.children.whereType<Ennemi>()`. C'est le polymorphisme du chapitre 10.

```text
                 PositionComponent   (Flame)
                          |
                       Ennemi        abstract  -- ne s'instancie pas
                       /     \
                Gobelin       Chauvesouris     -- concrètes
```

---

## 37.6 — `mettreAJourIA(double dt)` : la méthode abstraite

### La déclaration

```dart
void mettreAJourIA(double dt);   // pas de corps, pas d'accolades, un point-virgule
```

C'est la forme d'une **méthode abstraite** vue au chapitre 11 : une signature suivie d'un point-virgule, dans une classe `abstract`. Elle décrit un contrat sans l'honorer.

### Qui l'appelle, et quand

Le point capital est que `mettreAJourIA` n'est **pas** `update`. La classe `Ennemi` garde la main sur `update` et n'appelle l'IA qu'au bon moment, entouré de garde-fous.

```dart
@override
void update(double dt) {
  super.update(dt);                 // 1. les enfants (corps, hitbox, effets)

  if (etat == EtatEnnemi.mort) {
    return;                         // 2. un mort ne décide plus rien
  }

  if (_tempsKnockback > 0) {
    _tempsKnockback -= dt;          // 3. repoussé : l'IA est suspendue
    velocite = velocite * 0.88;     //    la vélocité s'amortit toute seule
  } else {
    mettreAJourIA(dt);              // 4. LA DÉCISION (sous-classe)
  }

  if (subitGravite) {               // 5. la physique, identique pour tous
    velocite.y += Constantes.gravite * dt;
    if (velocite.y > Constantes.vitesseMaxChute) {
      velocite.y = Constantes.vitesseMaxChute;
    }
  }

  auSol = false;                    // 6. remis à true par onCollision
  position += velocite * dt;        // 7. intégration (chapitre 23.13)

  oeil.position.x = sens > 0 ? size.x - 7 : 3;   // 8. l'oeil regarde devant
}
```

Ce découpage porte un nom : **inversion de contrôle**. La sous-classe ne pilote pas la boucle, elle est appelée par elle.

```text
  update(dt)      <- appelé par Flame, écrit UNE FOIS dans Ennemi
     |
     +-- garde : mort ?
     +-- garde : repoussé ?
     +-- mettreAJourIA(dt)   <- écrit par Gobelin, par Chauvesouris, par Boss...
     +-- gravité
     +-- intégration
```

### Le contrat de `mettreAJourIA`

Pour que le mécanisme tienne, la méthode abstraite doit avoir un contrat clair, que toutes les sous-classes respectent :

> **`mettreAJourIA(dt)` a le droit de modifier `velocite`, `sens` et `etat`. Elle n'a pas le droit de modifier `position` directement, ni d'ajouter la gravité, ni de résoudre une collision.**

Cette discipline a une conséquence agréable : pour écrire un nouvel ennemi, vous n'avez plus à penser à la physique. Le boss du chapitre 39 se résumera à une trentaine de lignes de décision.

### Le squelette d'une implémentation

```dart
// Dans Gobelin
@override
void mettreAJourIA(double dt) {
  switch (etat) {
    case EtatEnnemi.patrouille:
      _patrouiller(dt);
      break;
    case EtatEnnemi.poursuite:
      _poursuivre(dt);
      break;
    case EtatEnnemi.attaque:
      _attaquer(dt);
      break;
    case EtatEnnemi.retour:
      _revenir(dt);
      break;
    case EtatEnnemi.mort:
      break;
  }
}
```

Un `switch` sur un enum, un cas par état, une méthode privée par cas. C'est la forme canonique d'une machine à états (chapitre 26), et c'est exactement ce que nous allons remplir dans les sections 37.8 à 37.14.

> **Astuce du compilateur.** Un `switch` sur un enum **sans clause `default`** déclenche un avertissement de l'analyseur dès que vous ajoutez une valeur à l'enum sans traiter le cas correspondant. Ne mettez donc jamais de `default` dans une machine à états : laissez l'analyseur travailler pour vous.

---

## 37.7 — `lib/composants/gobelin.dart` : l'ennemi terrestre

Le gobelin est le premier ennemi du donjon, et le plus simple à comprendre : il marche, il tombe, il vous court après.

### La fiche technique

```text
  GOBELIN
  taille          24 x 30 pixels
  couleur         vert
  PV              30
  vitesse         60 px/s en patrouille, 60 px/s en poursuite
  dégâts contact  12
  score           50
  gravité         OUI
  aggro           140 px, avec ligne de vue
  abandon         220 px
  attaque         28 px
```

### Le squelette

```dart
// lib/composants/gobelin.dart
import 'package:flame/components.dart';

import '../config/constantes.dart';
import '../config/palette.dart';
import 'ennemi.dart';

class Gobelin extends Ennemi {
  Gobelin({required Vector2 position})
      : super(
          position: position,
          taille: Vector2(24, 30),
          vitesse: Constantes.vitesseGobelin,
          degatsContact: Constantes.degatsContactGobelin,
          pointsScore: Constantes.scoreGobelin,
          couleur: Palette.gobelin,
          pvDepart: Constantes.pvGobelin,
        );

  @override
  Future<void> onLoad() async {
    await super.onLoad();                 // OBLIGATOIRE : corps, hitbox, santé
    rayonAggro = Constantes.rayonAggroGobelin;
    rayonAbandon = Constantes.rayonAbandonGobelin;
    rayonAttaque = Constantes.rayonAttaqueGobelin;
    demiPatrouille = 3 * Constantes.tailleTuile;
  }

  @override
  void mettreAJourIA(double dt) {
    // rempli aux sections 37.8 à 37.13
  }
}
```

Tout tient dans un constructeur qui remplit les cases de la classe mère, un `onLoad` qui ajuste trois portées, et une méthode de décision. **Aucune ligne de rendu, aucune ligne de physique, aucune ligne de collision.** C'est le bénéfice direct de la section 37.5.

> **L'oubli qui coûte une heure.** `await super.onLoad();` en première ligne. Sans lui, ni le corps, ni la hitbox, ni `pvMax` ne sont initialisés : le gobelin est invisible, intraversable et lève une `LateInitializationError` au premier coup reçu.

---

## 37.8 — La patrouille entre deux bornes

### Le principe

Un ennemi qui patrouille fait des allers-retours autour de son point de naissance. On mémorise ce point dans `ancre` (rempli par `onLoad` de la classe mère : `ancre = position.clone()`), et on définit deux bornes.

```text
        ancre.x - demiPatrouille        ancre.x        ancre.x + demiPatrouille
                |                          |                       |
                v                          v                       v
    ############|##########################|#######################|############
                [<---------- zone de patrouille du gobelin -------->]
                          <- sens = -1        sens = +1 ->
```

### Le code

```dart
void _patrouiller(double dt) {
  velocite.x = sens * vitesse;

  final borneGauche = ancre.x - demiPatrouille;
  final borneDroite = ancre.x + demiPatrouille;

  if (sens > 0 && position.x >= borneDroite) {
    sens = -1;
  } else if (sens < 0 && position.x <= borneGauche) {
    sens = 1;
  }
}
```

### Trois erreurs classiques, dans l'ordre de fréquence

**Erreur 1 : comparer sans tenir compte du sens.**

```dart
// FAUX
if (position.x >= borneDroite || position.x <= borneGauche) {
  sens = -sens;
}
```

Si le gobelin dépasse la borne droite d'un demi-pixel, il fait demi-tour. La frame suivante, il est encore au-delà de la borne (il n'a reculé que d'un pixel) : il refait demi-tour. Il **vibre** sur place indéfiniment. Le test `sens > 0 &&` supprime le problème : on ne se retourne que si on va vers la borne qu'on dépasse.

**Erreur 2 : oublier `.clone()` sur l'ancre.**

```dart
ancre = position;          // FAUX : `ancre` et `position` sont le MÊME objet
ancre = position.clone();  // JUSTE
```

`Vector2` est **mutable** (rappel du chapitre 23 et de la fiche Flame) : sans `clone()`, l'ancre suit le gobelin et la zone de patrouille se déplace avec lui. Le gobelin part alors tout droit jusqu'au bout du niveau.

**Erreur 3 : patrouiller en modifiant `position` au lieu de `velocite`.**

```dart
position.x += sens * vitesse * dt;   // FAUX ICI
```

Ça fonctionne… jusqu'à ce que le knockback, la gravité ou la résolution de collision entrent en jeu : deux codes déplacent alors le même composant et se contredisent. Le contrat de la section 37.6 est clair : l'IA écrit dans `velocite`, la classe mère intègre.

**Résultat attendu :**

```text
t = 0.0 s   x = 200   sens = +1
t = 1.0 s   x = 260   sens = +1
t = 1.6 s   x = 296   sens = -1   (borne droite = 200 + 96)
t = 3.2 s   x = 200   sens = -1
t = 4.8 s   x = 104   sens = +1   (borne gauche = 200 - 96)
```

---

## 37.9 — Détecter le vide et faire demi-tour

### Le problème

La patrouille de la section 37.8 ignore complètement le décor. Placez le gobelin sur une plateforme de deux tuiles avec `demiPatrouille = 96` et il marchera dans le vide au bout d'une seconde.

```text
  AVANT (patrouille aveugle)              APRÈS (patrouille informée)

     g ->                                    g ->
  ########                                ########
          |                                       |
          v  il tombe                             +-- il fait demi-tour
       .....                                   .....
```

Trois obstacles doivent provoquer un demi-tour :

| Obstacle | Comment le savoir |
| --- | --- |
| la borne de patrouille | comparaison sur `position.x` (fait en 37.8) |
| un mur devant | un rayon horizontal, ou la collision elle-même |
| **le vide devant** | un rayon vertical vers le bas, devant les pieds |

Le troisième cas est le seul qui ne peut pas être détecté par une collision : **l'absence de contact n'est pas un événement**. Aucun `onCollision` ne se déclenchera jamais pour dire « il n'y a rien sous mon pied avant ». Il faut aller chercher l'information soi-même.

### La solution naïve, et pourquoi elle ne suffit pas

Beaucoup de tutoriels proposent de tester le contact avec le sol :

```dart
// Insuffisant
if (!auSol) {
  sens = -sens;   // il fait demi-tour APRÈS avoir quitté la plateforme
}
```

Le gobelin fait demi-tour, mais **en l'air** : il a déjà basculé dans le vide, la gravité l'emporte, et le demi-tour ne sert qu'à le faire tomber en marche arrière. Le retournement doit avoir lieu **avant** que le pied ne quitte le sol : il faut donc interroger un point qui est encore **devant** le gobelin.

### La bonne approche

On sonde un point situé un peu en avant du corps, au niveau des pieds, et on regarde vers le bas.

```text
        +--------+
        | GOBELIN|
        |        |
        +--------+
                  o  <- origine du rayon : centre + (sens * largeur * 0.6,
                  |                                  hauteur * 0.5 - 1)
                  |     longueur du rayon : 0.75 tuile
                  v
  ################        <- si le rayon touche une plateforme : il y a du sol
                          <- s'il ne touche rien : c'est le vide, demi-tour
```

L'outil qui fait cela s'appelle un **raycast**, et c'est l'objet de la section suivante.

---

## 37.10 — Le raycast pour voir devant soi (rappel chapitre 32)

### Rappel de l'API

Vous avez découvert le lancer de rayons à la section 32.23. Voici le rappel exact, vérifié pour Flame 1.38.0.

```dart
final rayon = Ray2(
  origin: Vector2(100, 50),      // point de départ, en coordonnées du monde
  direction: Vector2(0, 1),      // direction NORMALISÉE (longueur 1)
);

final RaycastResult<ShapeHitbox>? resultat = game.collisionDetection.raycast(
  rayon,
  maxDistance: 24,                                   // optionnel
  hitboxFilter: (candidate) => candidate.parent is Plateforme,  // optionnel
  ignoreHitboxes: const [],                          // optionnel
);
```

| Membre du résultat | Type | Contenu |
| --- | --- | --- |
| `distance` | `double?` | distance parcourue jusqu'au point d'impact |
| `intersectionPoint` | `Vector2?` | le point touché, en coordonnées du monde |
| `normal` | `Vector2?` | la normale de la surface touchée |
| `hitbox` | `ShapeHitbox?` | la hitbox touchée ; `hitbox.parent` donne le composant |
| `isActive` | `bool` | le résultat contient-il des données valides |
| `reflectionRay` | `Ray2?` | le rayon réfléchi (utile pour un ricochet) |

`raycast` renvoie `null` quand rien n'est touché. C'est précisément la réponse « il n'y a rien devant », celle qui nous manquait à la section 37.9.

### Trois précautions

**1. La direction doit être normalisée.** `Vector2(0, 1)` et `Vector2(-1, 0)` le sont déjà. Pour une direction quelconque, `direction.normalized()` (chapitre 23.7). Une direction de longueur 2 ferait mentir `maxDistance` d'un facteur deux.

**2. Le rayon part de l'intérieur de l'ennemi.** Sans précaution, il touche d'abord la hitbox de l'ennemi lui-même. Deux parades : `ignoreHitboxes: [hitbox]`, ou un `hitboxFilter` qui ne retient que le décor. Nous utilisons la seconde, plus sûre : elle ignore aussi les projectiles, les pièces et les autres gobelins.

**3. `hitboxFilter` retient ce qui renvoie `true`.** La fonction reçoit chaque hitbox candidate et répond « faut-il en tenir compte ». `(candidate) => candidate.parent is Plateforme` signifie donc « ne considère que les plateformes ».

### Les trois sondes de l'ennemi

Ces trois méthodes vont dans la classe `Ennemi` : les deux espèces s'en servent.

```dart
/// Y a-t-il du sol juste devant les pieds ?
bool solDevant() {
  final origine = absoluteCenter +
      Vector2(sens * size.x * 0.6, size.y * 0.5 - 1);

  final resultat = game.collisionDetection.raycast(
    Ray2(origin: origine, direction: Vector2(0, 1)),
    maxDistance: Constantes.tailleTuile * 0.75,
    hitboxFilter: (candidate) => candidate.parent is Plateforme,
  );

  return resultat != null;
}

/// Y a-t-il un mur juste devant ?
bool murDevant() {
  final origine = absoluteCenter + Vector2(0, size.y * 0.25);

  final resultat = game.collisionDetection.raycast(
    Ray2(origin: origine, direction: Vector2(sens.toDouble(), 0)),
    maxDistance: size.x * 0.5 + 4,
    hitboxFilter: (candidate) => candidate.parent is Plateforme,
  );

  return resultat != null;
}

/// Le joueur est-il visible, sans mur entre nous deux ?
bool voitLeJoueur() {
  final cible = game.joueur;
  if (!cible.isMounted || !cible.estVivant) {
    return false;
  }

  final vers = cible.absoluteCenter - absoluteCenter;
  final distance = vers.length;
  if (distance > rayonAggro) {
    return false;               // trop loin : inutile de lancer un rayon
  }
  if (distance < 1) {
    return true;                // collés l'un à l'autre
  }

  final resultat = game.collisionDetection.raycast(
    Ray2(origin: absoluteCenter.clone(), direction: vers.normalized()),
    maxDistance: distance,
    hitboxFilter: (candidate) => candidate.parent is Plateforme,
  );

  return resultat == null;      // aucun mur sur le trajet
}
```

### Pourquoi tester la distance AVANT de lancer le rayon

```dart
if (distance > rayonAggro) return false;   // <-- ce test, en premier
```

Un raycast n'est pas gratuit : il parcourt la structure de broadphase du moteur. Avec vingt ennemis à soixante images par seconde, un raycast systématique représente mille deux cents lancers par seconde. Le test de distance, lui, coûte une soustraction et une racine carrée. On filtre donc **du moins cher au plus cher** :

```text
  1. distance > rayonAggro ?            soustraction + racine        très rapide
  2. le joueur est-il vivant/monté ?    deux booléens                très rapide
  3. raycast                            parcours du broadphase       coûteux
```

C'est la règle générale de l'IA de jeu : **le test le moins cher passe en premier**.

---

## 37.11 — La détection du joueur : le rayon d'aggro

### Trois rayons, pas un

Un ennemi bien réglé utilise **trois** distances différentes, et c'est ce qui donne l'impression qu'il « réfléchit ».

```text
                        rayonAbandon (220)
        +-------------------------------------------------+
        |            rayonAggro (140)                     |
        |    +-----------------------------+              |
        |    |     rayonAttaque (28)       |              |
        |    |        +-------+            |              |
        |    |        |   G   |            |              |
        |    |        +-------+            |              |
        |    +-----------------------------+              |
        +-------------------------------------------------+

  distance <= 28    -> il frappe
  distance <= 140   -> il poursuit (s'il vous VOIT)
  distance >  220   -> il abandonne et rentre
```

### Pourquoi `rayonAbandon` doit être plus grand que `rayonAggro`

C'est le principe de l'**hystérésis**, déjà rencontré au chapitre 26. Si les deux rayons étaient égaux (disons 140), un joueur qui reste à 140 pixels ferait osciller l'ennemi entre poursuite et retour soixante fois par seconde : l'ennemi tremblerait sur place.

```text
  SANS hystérésis (aggro = abandon = 140)      AVEC hystérésis (140 / 220)

  d = 139.9  -> poursuite                       d = 139.9 -> poursuite
  d = 140.1  -> retour                          d = 150   -> poursuite
  d = 139.8  -> poursuite                       d = 210   -> poursuite
  d = 140.2  -> retour            (tremblote)   d = 221   -> retour
```

La règle empirique : `rayonAbandon` vaut environ 1,5 fois `rayonAggro`.

### Le code de détection

```dart
double get distanceAuJoueur {
  final cible = game.joueur;
  if (!cible.isMounted) {
    return double.infinity;
  }
  return (cible.absoluteCenter - absoluteCenter).length;
}
```

Renvoyer `double.infinity` quand le joueur n'est pas monté est un choix délibéré : toutes les comparaisons `distance > rayonX` deviennent vraies, donc tous les ennemis rentrent chez eux. Aucun cas particulier à écrire ailleurs.

### Un détail qui change tout : `absoluteCenter`

```dart
(cible.position - position).length          // FAUX si les ancres diffèrent
(cible.absoluteCenter - absoluteCenter).length   // JUSTE
```

`position` est la position de l'**ancre** dans le repère du **parent**. Avec `Anchor.topLeft`, la distance entre les positions est la distance entre les coins supérieurs gauches : un gobelin de 24 pixels de large et un joueur de 24 pixels paraissent à 24 pixels l'un de l'autre alors qu'ils se touchent. `absoluteCenter` renvoie le centre en coordonnées du monde, quels que soient l'ancre et la hiérarchie de parents.

### Aggro à travers un mur : le bug qui ruine l'ambiance

Sans ligne de vue, un gobelin situé derrière un mur épais vous poursuivra en se cognant contre la pierre pour l'éternité. Le joueur ne voit qu'un ennemi coincé qui tremble.

```text
  SANS ligne de vue                     AVEC ligne de vue (voitLeJoueur)

     G ->|########|  J                     G  |########|  J
         mur                                  mur
     le gobelin fonce dans le mur         le gobelin continue sa patrouille
```

D'où la condition d'entrée en poursuite : **distance ET vision**.

```dart
if (voitLeJoueur()) {
  etat = EtatEnnemi.poursuite;
}
```

`voitLeJoueur()` contient déjà le test de distance (section 37.10), ce qui évite de le répéter.

---

## 37.12 — La poursuite

### Le code

```dart
void _poursuivre(double dt) {
  final cible = game.joueur;
  final ecartX = cible.absoluteCenter.x - absoluteCenter.x;

  // 1. Se tourner vers le joueur, avec une zone morte de 2 pixels.
  if (ecartX.abs() > 2) {
    sens = ecartX > 0 ? 1 : -1;
  }

  // 2. Avancer, SAUF si le sol manque ou si un mur bloque.
  if (solDevant() && !murDevant()) {
    velocite.x = sens * vitesse;
  } else {
    velocite.x = 0;         // il piétine au bord du gouffre
  }

  // 3. Transitions.
  final distance = distanceAuJoueur;
  if (distance <= rayonAttaque) {
    etat = EtatEnnemi.attaque;
    _tempsAttaque = Constantes.dureeAttaqueEnnemi;
  } else if (distance > rayonAbandon || !voitLeJoueur()) {
    etat = EtatEnnemi.retour;
  }
}
```

### La zone morte de 2 pixels

```dart
if (ecartX.abs() > 2) {
  sens = ecartX > 0 ? 1 : -1;
}
```

Sans elle, quand le gobelin est pile sous le joueur, `ecartX` oscille entre `+0.3` et `-0.4` d'une frame à l'autre. Le gobelin change de sens soixante fois par seconde et **vibre**. Deux pixels de tolérance suffisent à éliminer complètement le phénomène. C'est la même idée que l'hystérésis de la section 37.11, appliquée à l'orientation.

### Le bord du gouffre

L'appel à `solDevant()` **dans la poursuite** est ce qui distingue un ennemi crédible d'un ennemi suicidaire.

```text
   J (le joueur est en bas à droite)

   G ->                          Sans solDevant() : le gobelin saute dans le vide
  ######                         et meurt sans que le joueur ait rien fait.
        ......
              ######             Avec solDevant() : il s'arrête au bord et
                    J            attend. Le joueur doit venir le chercher.
```

Selon le jeu que vous voulez, le comportement au bord change :

| Comportement | Code | Ressenti |
| --- | --- | --- |
| il s'arrête | `velocite.x = 0` | prudent, classique |
| il saute | `if (auSol) velocite.y = -320;` | agressif, dangereux |
| il tombe quand même | ne rien tester | idiot, à éviter |

Nous choisissons « il s'arrête », et l'exercice 3 vous fera coder « il saute ».

---

## 37.13 — La machine à états de l'ennemi

### Les cinq états et leur rôle

| État | Ce que l'ennemi fait | Ce qui le fait sortir |
| --- | --- | --- |
| `patrouille` | va-et-vient autour de `ancre` | il voit le joueur -> `poursuite` |
| `poursuite` | se dirige vers le joueur | joueur à portée -> `attaque` ; trop loin ou perdu de vue -> `retour` |
| `attaque` | s'immobilise et frappe pendant un court instant | fin du temps d'attaque -> `poursuite` |
| `retour` | revient vers `ancre` | arrivé -> `patrouille` ; revoit le joueur -> `poursuite` |
| `mort` | plus rien | **aucune sortie** (état terminal) |

### Le code de l'IA du gobelin

Le répartiteur, puis les deux états qui n'ont pas encore été écrits. `_patrouiller` est celui de la section 37.8, complété par les deux sondes de la section 37.10 ; `_poursuivre` est celui de la section 37.12.

```dart
@override
void mettreAJourIA(double dt) {
  switch (etat) {
    case EtatEnnemi.patrouille:
      _patrouiller(dt);
      if (voitLeJoueur()) {
        etat = EtatEnnemi.poursuite;
      }
      break;
    case EtatEnnemi.poursuite:
      _poursuivre(dt);
      break;
    case EtatEnnemi.attaque:
      _attaquer(dt);
      break;
    case EtatEnnemi.retour:
      _revenir(dt);
      break;
    case EtatEnnemi.mort:
      velocite.x = 0;
      break;
  }
}

void _patrouiller(double dt) {
  velocite.x = sens * vitesse;

  final borneGauche = ancre.x - demiPatrouille;
  final borneDroite = ancre.x + demiPatrouille;

  final doitSeRetourner = (sens > 0 && position.x >= borneDroite) ||
      (sens < 0 && position.x <= borneGauche) ||
      murDevant() ||
      !solDevant();          // <-- les deux sondes de la section 37.10

  if (doitSeRetourner) {
    sens = -sens;
    velocite.x = sens * vitesse;
  }
}

void _attaquer(double dt) {
  velocite.x = 0;                       // il plante ses pieds
  _tempsAttaque -= dt;
  if (_tempsAttaque <= 0) {
    etat = EtatEnnemi.poursuite;        // il enchaîne ou il abandonne
  }
}

void _revenir(double dt) {
  final ecartX = ancre.x - position.x;

  if (ecartX.abs() < 4) {
    velocite.x = 0;
    etat = EtatEnnemi.patrouille;
    return;
  }

  sens = ecartX > 0 ? 1 : -1;
  velocite.x = (solDevant() && !murDevant()) ? sens * vitesse * 0.8 : 0;

  if (voitLeJoueur()) {
    etat = EtatEnnemi.poursuite;        // il se ravise
  }
}
```

Le fichier complet figure en section 37.34.

### L'état `mort` est terminal

Aucune transition ne sort de `mort`. C'est une propriété importante : elle garantit qu'un ennemi mort ne peut plus rien faire, même si un projectile le touche pendant son animation de disparition. La garde placée en tête de `update` (section 37.6) rend d'ailleurs la branche `case EtatEnnemi.mort` inatteignable ; on l'écrit quand même, pour que le `switch` soit exhaustif et que l'analyseur reste silencieux.

### Où mettre les transitions

Deux écoles s'affrontent, et l'une des deux est mauvaise.

| Approche | Description | Verdict |
| --- | --- | --- |
| transitions **dans** chaque cas | chaque état décide de sa propre sortie | recommandée |
| transitions **avant** le `switch` | une grosse cascade de `if` centralisée | à fuir |

La seconde produit très vite un pavé de conditions imbriquées où l'on ne sait plus quel test a la priorité. La première garde le raisonnement local : pour comprendre l'état `poursuite`, on lit uniquement `_poursuivre`.

### Le journal de debug qui sauve des heures

Ajoutez un setter au lieu d'un champ nu et vous verrez chaque transition :

```dart
EtatEnnemi _etat = EtatEnnemi.patrouille;

EtatEnnemi get etat => _etat;

set etat(EtatEnnemi nouveau) {
  if (nouveau == _etat) {
    return;              // on ne trace pas les non-transitions
  }
  if (debugMode) {
    // ignore: avoid_print
    print('$runtimeType : $_etat -> $nouveau');
  }
  _etat = nouveau;
}
```

**Résultat attendu dans la console :**

```text
Gobelin : EtatEnnemi.patrouille -> EtatEnnemi.poursuite
Gobelin : EtatEnnemi.poursuite -> EtatEnnemi.attaque
Gobelin : EtatEnnemi.attaque -> EtatEnnemi.poursuite
Gobelin : EtatEnnemi.poursuite -> EtatEnnemi.retour
Gobelin : EtatEnnemi.retour -> EtatEnnemi.patrouille
```

Si vous voyez une ligne qui se répète soixante fois par seconde, vous venez de trouver un défaut d'hystérésis.

---

## 37.14 — Le schéma des transitions

Le voici en entier. Gardez-le sous les yeux pendant que vous écrivez l'IA du boss au chapitre 39 : il ne changera pas.

```text
                      +---------------------------------------------+
                      |                                             |
                      v                                             |
              +---------------+                                     |
   depart --->|  PATROUILLE   |                                     |
              +---------------+                                     |
                      |  voitLeJoueur()                             |
                      v                                             |
              +---------------+   distance <= rayonAttaque   +--------------+
              |   POURSUITE   |----------------------------->|   ATTAQUE    |
              +---------------+                              +--------------+
                   |     ^                                          |
                   |     |  voitLeJoueur()                          |
                   |     +------------------------------------------+
                   |     |            fin du temps d'attaque
                   |     |
   distance > rayonAbandon                                          |
   OU !voitLeJoueur()    |                                          |
                   |     |                                          |
                   v     |                                          |
              +---------------+                                     |
              |    RETOUR     |  arrive sur ancre                   |
              +---------------+-------------------------------------+
                      |                          (retour vers PATROUILLE)
                      |
                      |  pv <= 0    (depuis N'IMPORTE QUEL état)
                      v
              +---------------+
              |     MORT      |   état terminal : aucune sortie
              +---------------+
```

### La table de transitions

Le même schéma, sous forme de table — la forme que l'on relit le plus vite quand on cherche un bug.

| État courant | Condition | Nouvel état |
| --- | --- | --- |
| `patrouille` | `voitLeJoueur()` | `poursuite` |
| `patrouille` | borne, mur ou vide devant | `patrouille` (demi-tour, pas de transition) |
| `poursuite` | `distance <= rayonAttaque` | `attaque` |
| `poursuite` | `distance > rayonAbandon` ou `!voitLeJoueur()` | `retour` |
| `attaque` | `_tempsAttaque <= 0` | `poursuite` |
| `retour` | `voitLeJoueur()` | `poursuite` |
| `retour` | `abs(ancre.x - position.x) < 4` | `patrouille` |
| **tous** | `pv <= 0` | `mort` |
| `mort` | — | — |

### La transition universelle vers `mort`

`mort` ne figure dans aucune des méthodes `_patrouiller`, `_poursuivre`, `_attaquer`, `_revenir`. Elle est déclenchée **ailleurs**, par le mixin `Sante` :

```text
  subirDegats(25)          [mixin Sante]
      -> pv atteint 0
      -> onMort()          [redéfini dans Ennemi]
      -> mourir()          [Ennemi]
      -> etat = EtatEnnemi.mort
```

C'est le grand avantage d'une transition centralisée : peu importe l'origine des dégâts — épée du joueur, piège du chapitre 39, chute —, la mort se déclenche toujours au même endroit et de la même manière. Ne dupliquez jamais `if (pv <= 0) etat = mort;` dans les états : ce serait quatre occasions de se tromper.

---

## 37.15 — `lib/composants/chauvesouris.dart` : l'ennemi volant

### Pourquoi un deuxième ennemi

Le gobelin ne menace que le sol. Un joueur qui a compris le jeu passe sa partie sur les plateformes hautes, hors de portée. Il faut donc une menace **verticale**, et c'est le rôle de la chauve-souris.

| | Gobelin | Chauve-souris |
| --- | --- | --- |
| gravité | oui | **non** |
| déplacement | horizontal, au sol | libre, en ondulant |
| attaque | contact | **projectile** + piqué |
| PV | 30 | 15 |
| vitesse | 60 px/s | 90 px/s |
| dégâts contact | 12 | 8 |
| score | 50 | 35 |
| menace | zones basses | partout |

### S'affranchir de la gravité

Toute la différence physique tient dans une ligne, grâce au découpage de la section 37.6 :

```dart
@override
bool get subitGravite => false;
```

`update` de la classe `Ennemi` teste ce getter avant d'ajouter `Constantes.gravite * dt` à `velocite.y`. Redéfinir un getter au lieu de dupliquer `update` : c'est du polymorphisme au sens strict (chapitre 10).

### Le squelette

```dart
// lib/composants/chauvesouris.dart
class Chauvesouris extends Ennemi {
  Chauvesouris({required Vector2 position})
      : super(
          position: position,
          taille: Vector2(22, 16),
          vitesse: Constantes.vitesseChauvesouris,
          degatsContact: Constantes.degatsContactChauvesouris,
          pointsScore: Constantes.scoreChauvesouris,
          couleur: Palette.chauvesouris,
          pvDepart: Constantes.pvChauvesouris,
        );

  @override
  bool get subitGravite => false;

  double _phase = 0;                 // avance du temps dans le sinus
  double _cooldownTir = 0;           // secondes avant le prochain tir
  double _tempsPique = 0;            // secondes restantes de piqué

  double amplitude = 18;             // hauteur de l'ondulation, en pixels
  double frequence = 2.4;            // ondulations par seconde x 2 pi

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    rayonAggro = Constantes.rayonAggroChauvesouris;
    rayonAbandon = Constantes.rayonAbandonChauvesouris;
    rayonAttaque = Constantes.rayonAttaqueChauvesouris;
    demiPatrouille = 4 * Constantes.tailleTuile;
  }

  @override
  void mettreAJourIA(double dt) { /* sections 37.16 et 37.17 */ }
}
```

### La hitbox : rectangle ou cercle ?

Une chauve-souris est plus proche d'un disque que d'une boîte. Nous lui donnons donc une `CircleHitbox`, ce qui adoucit les contacts en diagonale. La classe `Ennemi` créant une `RectangleHitbox` par défaut, on rend la fabrication redéfinissable :

```dart
// Dans Ennemi
ShapeHitbox creerHitbox() => RectangleHitbox();

// Dans Chauvesouris
@override
ShapeHitbox creerHitbox() => CircleHitbox();
```

`ShapeHitbox` est le type parent commun de `RectangleHitbox`, `CircleHitbox` et `PolygonHitbox` : c'est lui qui porte `collisionType`, que nous manipulerons à la mort (section 37.27).

---

## 37.16 — Le mouvement sinusoïdal

### L'idée

Une chauve-souris qui vole en ligne droite ressemble à un carré qui glisse. Une ondulation verticale change tout, pour trois lignes de code.

```text
   y

   ancre.y + A  -    .-.         .-.         .-.
                    /   \       /   \       /   \
   ancre.y      -  /     \     /     \     /     \      ----> x
                          \   /       \   /       \   /
   ancre.y - A  -          '-'         '-'         '-'

   y(t) = ancre.y + A * sin(w * t)
     A = amplitude (pixels)
     w = frequence (radians par seconde)
```

### Deux façons de coder un sinus

**Méthode 1 — écrire la position directement.**

```dart
_phase += frequence * dt;
position.y = ancre.y + sin(_phase) * amplitude;
```

Simple, exact, sans dérive… mais elle **écrase** toute autre influence sur `y` : le knockback n'a plus aucun effet vertical, et le piqué devient impossible. Elle viole en outre le contrat de la section 37.6 (« l'IA n'écrit pas dans `position` »).

**Méthode 2 — écrire la vitesse, c'est-à-dire la dérivée.**

La dérivée de `A · sin(ω·t)` par rapport au temps est `A · ω · cos(ω·t)`.

```dart
_phase += frequence * dt;
velocite.y = cos(_phase) * amplitude * frequence;
```

Compatible avec tout le reste, mais l'intégration d'Euler (chapitre 23.16) introduit une petite dérive : au bout de deux minutes, la chauve-souris a lentement glissé de quelques pixels vers le haut ou le bas.

**Méthode 3 — la dérivée plus un rappel élastique.** C'est celle que nous retenons.

```dart
_phase += frequence * dt;

final cibleY = ancre.y + sin(_phase) * amplitude;

velocite.y = cos(_phase) * amplitude * frequence   // le mouvement voulu
    + (cibleY - position.y) * 4.0;                  // le rappel qui corrige
```

Le second terme est un ressort : plus la chauve-souris s'écarte de la trajectoire idéale, plus il la ramène fort. Il annule la dérive, absorbe le knockback en douceur et laisse le piqué reprendre la main quand on court-circuite le calcul.

```text
  position réelle
       o                        (cibleY - position.y) est grand
       |                        -> rappel fort
       v
  - - -X- - - -  trajectoire idéale : ancre.y + A sin(phase)
```

### Le code de la patrouille volante

```dart
void _voler(double dt) {
  velocite.x = sens * vitesse;

  final borneGauche = ancre.x - demiPatrouille;
  final borneDroite = ancre.x + demiPatrouille;

  if ((sens > 0 && position.x >= borneDroite) ||
      (sens < 0 && position.x <= borneGauche) ||
      murDevant()) {
    sens = -sens;
    velocite.x = sens * vitesse;
  }

  _ondulation();
}

void _ondulation() {
  final cibleY = ancre.y + sin(_phase) * amplitude;
  velocite.y = cos(_phase) * amplitude * frequence + (cibleY - position.y) * 4.0;
}
```

Remarquez qu'on ne teste pas `solDevant()` : une chauve-souris n'a que faire du vide. En revanche `murDevant()` reste utile, sinon elle traverserait les piliers en vibrant contre eux.

---

## 37.17 — Le piqué vers le joueur

### Le principe

Le piqué (*dive*) est l'attaque signature de la chauve-souris : elle fond en ligne droite vers l'endroit où se trouvait le joueur, dépasse, puis remonte.

```text
       C
        \                     1. elle mémorise la position du joueur
         \                    2. elle fonce en LIGNE DROITE, vite
          \                   3. au bout de 0,45 s, elle remonte
           \
            v
             x  <- la cible mémorisée (le joueur a peut-être bougé)
                J
```

### Pourquoi mémoriser la cible

Le point capital est là.

| Comportement | Effet sur le joueur |
| --- | --- |
| la chauve-souris **suit** le joueur pendant le piqué | impossible à esquiver, injuste |
| elle **fonce sur la position mémorisée** | esquivable, lisible, satisfaisant |

Un ennemi qui recalcule sa trajectoire à chaque frame est mathématiquement imparable : il colle au joueur. Un ennemi qui s'engage sur une trajectoire décidée à l'avance offre au joueur une **fenêtre de réaction**. C'est une des règles les plus universelles du game design d'action : la puissance d'une attaque doit être compensée par la possibilité de la lire.

### Le code

```dart
void _piquer(double dt) {
  _tempsPique -= dt;

  if (_tempsPique <= 0 || murDevant()) {
    etat = EtatEnnemi.retour;          // elle remonte
    return;
  }

  // La vélocité a été fixée UNE FOIS au déclenchement : on ne la retouche pas.
}

void _declencherPique() {
  final cible = game.joueur.absoluteCenter;
  final direction = (cible - absoluteCenter).normalized();

  velocite = direction * (vitesse * 2.4);
  sens = direction.x >= 0 ? 1 : -1;

  _tempsPique = Constantes.dureePiqueChauvesouris;   // 0.45 s
  etat = EtatEnnemi.attaque;
}
```

La direction est **normalisée** (chapitre 23.7), puis multipliée par une vitesse : la chauve-souris se déplace à `vitesse * 2.4` quelle que soit la distance à la cible. Sans normalisation, elle irait dix fois plus vite sur une cible dix fois plus lointaine — le fameux bug de la diagonale du chapitre 23.8, dans sa version radiale.

### L'IA complète de la chauve-souris

```dart
@override
void mettreAJourIA(double dt) {
  _phase += frequence * dt;
  if (_cooldownTir > 0) {
    _cooldownTir -= dt;
  }

  switch (etat) {
    case EtatEnnemi.patrouille:
      _voler(dt);
      if (voitLeJoueur()) {
        etat = EtatEnnemi.poursuite;
      }
      break;

    case EtatEnnemi.poursuite:
      _poursuivreEnVol(dt);
      break;

    case EtatEnnemi.attaque:
      _piquer(dt);
      break;

    case EtatEnnemi.retour:
      _remonter(dt);
      break;

    case EtatEnnemi.mort:
      velocite.setZero();
      break;
  }
}

void _poursuivreEnVol(double dt) {
  final vers = game.joueur.absoluteCenter - absoluteCenter;
  final distance = vers.length;

  if (distance > 1) {
    velocite = vers.normalized() * (vitesse * 0.55);
    sens = vers.x >= 0 ? 1 : -1;
  }
  _ondulation();                       // elle continue d'onduler en approchant

  if (distance <= rayonAttaque && _cooldownTir <= 0) {
    _declencherPique();
    _cooldownTir = Constantes.cooldownTirChauvesouris;
    return;
  }

  if (_cooldownTir <= 0 && voitLeJoueur()) {
    tirer();
  }

  if (distance > rayonAbandon || !voitLeJoueur()) {
    etat = EtatEnnemi.retour;
  }
}

// `_remonter` ramène la chauve-souris vers son ancre puis repasse en
// patrouille ; il figure en section 37.34.
```

Notez la condition de reprise dans `_remonter` : `rayonAggro * 0.7`, et non `rayonAggro`. Encore de l'hystérésis : sans ce facteur, une chauve-souris qui vient d'abandonner reprend immédiatement la poursuite et fait du surplace.

**Résultat attendu :**

```text
La chauve-souris ondule autour de (300, 90), amplitude 18 px.
Le joueur entre à 170 px, ligne de vue dégagée.
  etat : patrouille -> poursuite
  elle tire un projectile, cooldown = 1.8 s
  elle approche à 50 px/s en continuant d'onduler
À 34 px : etat -> attaque, piqué à 216 px/s vers la position mémorisée.
0,45 s plus tard : etat -> retour, elle remonte vers son ancre.
Arrivée : etat -> patrouille.
```

---

## 37.18 — `lib/composants/projectile.dart`

### Le cahier des charges d'un projectile

Un projectile est le composant le plus simple du jeu, et pourtant celui qui provoque le plus de fuites de mémoire quand il est mal écrit. Il doit :

1. partir d'un point, dans une direction, à une vitesse ;
2. avancer tout seul, sans que personne ne s'en occupe ;
3. infliger des dégâts à ce qu'il touche ;
4. **disparaître**, toujours, quoi qu'il arrive.

Le point 4 est le seul qui compte vraiment. Un projectile qui sort de l'écran sans être retiré reste dans l'arbre de composants, continue d'être mis à jour, continue d'être testé en collision, pour toujours.

```text
  SANS durée de vie                     AVEC durée de vie

  t = 10 s    12 projectiles vivants     t = 10 s    3 projectiles vivants
  t = 60 s    72 projectiles vivants     t = 60 s    3 projectiles vivants
  t = 300 s   360 projectiles vivants    t = 300 s   3 projectiles vivants
              (le jeu rame)                          (le jeu tourne)
```

### Le code

```dart
// lib/composants/projectile.dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flutter/material.dart';

import '../config/constantes.dart';
import '../config/palette.dart';
import '../donjon_game.dart';
import 'joueur.dart';
import 'plateforme.dart';

class Projectile extends CircleComponent
    with HasGameReference<DonjonGame>, CollisionCallbacks {
  Projectile({
    required Vector2 position,
    required Vector2 direction,
    this.vitesse = Constantes.vitesseProjectile,
    this.degats = Constantes.degatsProjectile,
    this.dureeVie = Constantes.dureeVieProjectile,
  })  : direction = direction.normalized(),      // sécurité : toujours normalisée
        super(
          position: position,
          radius: 4,
          anchor: Anchor.center,
          priority: 5,
        );

  final Vector2 direction;
  final double vitesse;
  final double degats;
  final double dureeVie;

  double _tempsRestant = 0;

  @override
  Future<void> onLoad() async {
    _tempsRestant = dureeVie;
    paint = Paint()..color = Palette.projectile;
    add(CircleHitbox());
  }

  @override
  void update(double dt) {
    super.update(dt);

    _tempsRestant -= dt;
    if (_tempsRestant <= 0) {
      removeFromParent();
      return;
    }

    position += direction * (vitesse * dt);
  }

  @override
  void onCollisionStart(
    Set<Vector2> intersectionPoints,
    PositionComponent other,
  ) {
    super.onCollisionStart(intersectionPoints, other);

    if (other is Joueur) {
      other.subirDegats(degats, direction: direction.clone());
      removeFromParent();
    } else if (other is Plateforme) {
      removeFromParent();
    }
  }
}
```

### Trois décisions expliquées

**`extends CircleComponent`.** `CircleComponent` est un `PositionComponent` qui sait se dessiner. On récupère `paint`, `radius` et le rendu gratuitement. Pour un vrai sprite, on passerait à `SpriteComponent` sans toucher au reste.

**`direction.normalized()` dans la liste d'initialisation.** L'appelant peut passer n'importe quel vecteur ; la classe garantit elle-même l'invariant. Un appelant distrait ne peut plus créer un projectile qui va trois fois trop vite.

**`removeFromParent()` puis `return`.** Sans le `return`, la ligne suivante déplacerait un composant en cours de retrait. Ce n'est pas fatal (Flame tolère un retrait différé), mais c'est un déplacement inutile et surtout une mauvaise habitude : après avoir demandé un retrait, on sort.

### La collision avec un autre projectile

Deux projectiles qui se croisent ne se détruisent pas : `onCollisionStart` ne teste que `Joueur` et `Plateforme`. C'est voulu. Si vous voulez qu'ils s'annulent, la ligne à ajouter tient sur un ternaire — mais souvenez-vous que cela crée une mécanique de jeu (« je peux annuler les tirs ») que le joueur devra comprendre.

---

## 37.19 — Tirer vers le joueur : direction normalisée

### Le calcul, en trois lignes

```dart
void tirer() {
  final vers = game.joueur.absoluteCenter - absoluteCenter;   // 1. le vecteur
  if (vers.length < 1) {
    return;                                                   // 2. garde-fou
  }

  game.monde.add(                                             // 3. le tir
    Projectile(
      position: absoluteCenter.clone(),
      direction: vers.normalized(),
    ),
  );

  _cooldownTir = Constantes.cooldownTirChauvesouris;
}
```

### Le rappel du chapitre 23

```text
  vers    = cible - source              (soustraction de vecteurs, 23.4)
  |vers|  = sqrt(vers.x² + vers.y²)     (longueur, 23.6)
  u       = vers / |vers|               (normalisation, 23.7)

  Exemple :
    source = (100, 100)   cible = (160, 180)
    vers   = (60, 80)
    |vers| = sqrt(3600 + 6400) = sqrt(10000) = 100
    u      = (0.6, 0.8)                       |u| = 1
    vitesse 170 px/s  ->  velocite = (102, 136)
```

Le vecteur `u` porte **uniquement** l'information de direction. La vitesse est ensuite un facteur indépendant : c'est exactement pour cela qu'on normalise.

### Les quatre erreurs autour de `normalized()`

**1. Normaliser le vecteur nul.** `Vector2.zero().normalized()` produit `(NaN, NaN)`, et à partir de là toutes les positions calculées deviennent `NaN` : le projectile disparaît de l'écran sans erreur ni message. D'où le garde-fou `if (vers.length < 1) return;`.

**2. Confondre `normalized()` et `normalize()`.**

```dart
final u = vers.normalized();   // renvoie un NOUVEAU vecteur, `vers` intact
vers.normalize();              // MODIFIE `vers` sur place et renvoie sa longueur
```

Le second est une source de bugs quand `vers` est réutilisé plus loin.

**3. Partager le vecteur direction.**

```dart
// DANGEREUX : les deux projectiles partagent le MÊME Vector2
final d = vers.normalized();
monde.add(Projectile(position: p, direction: d));
monde.add(Projectile(position: p, direction: d));
```

`Vector2` est mutable. Si un projectile modifie sa direction (ricochet, tête chercheuse), l'autre change aussi. Le constructeur de `Projectile` appelle `normalized()`, qui renvoie une copie fraîche : notre code est donc protégé. Dans le doute, `.clone()`.

**4. Tirer depuis `position` au lieu de `absoluteCenter`.** Le projectile sortirait du coin supérieur gauche de la chauve-souris, visiblement décalé.

---

## 37.20 — La durée de vie d'un projectile

Il existe trois façons correctes de faire disparaître un projectile. Elles ne sont pas équivalentes.

### Méthode 1 — Le compte à rebours manuel (celle du chapitre)

```dart
double _tempsRestant = 0;

@override
Future<void> onLoad() async {
  _tempsRestant = dureeVie;
}

@override
void update(double dt) {
  super.update(dt);
  _tempsRestant -= dt;
  if (_tempsRestant <= 0) {
    removeFromParent();
    return;
  }
  position += direction * (vitesse * dt);
}
```

Avantages : aucune dépendance, lisible, et surtout **on peut lire `_tempsRestant`** pour faire pâlir le projectile en fin de vie :

```dart
paint.color = Palette.projectile.withValues(
  alpha: (_tempsRestant / dureeVie).clamp(0.2, 1.0),
);
```

### Méthode 2 — `RemoveEffect` (chapitre 33.18)

```dart
@override
Future<void> onLoad() async {
  add(RemoveEffect(delay: dureeVie));
}
```

Une ligne, zéro logique dans `update`. C'est la version la plus élégante quand aucun retour visuel progressif n'est nécessaire.

### Méthode 3 — `TimerComponent` (chapitre 33.33)

```dart
add(TimerComponent(period: dureeVie, removeOnFinish: true, onTick: removeFromParent));
```

Utile lorsque le même minuteur doit faire autre chose au passage (jouer un son, poser une flaque).

### La méthode incorrecte : la sortie d'écran

```dart
// PIÈGE
if (position.x < 0 || position.x > game.size.x) {
  removeFromParent();
}
```

Elle échoue dans trois cas au moins :

| Cas | Pourquoi ça échoue |
| --- | --- |
| la caméra suit le joueur | `game.size` est la taille de la **fenêtre**, pas du monde |
| le niveau est plus grand que l'écran | le projectile survit hors champ, à jamais |
| le projectile part exactement à la verticale | il ne franchit jamais les bornes en X |

Une durée de vie est une garantie **absolue** : au bout de N secondes, l'objet est parti, quelles que soient la caméra, la taille du niveau et la trajectoire.

---

## 37.21 — Le cooldown de tir (rappel chapitre 33)

### Le problème

`mettreAJourIA` est appelée à chaque frame. Sans limitation, une chauve-souris qui voit le joueur crée **soixante projectiles par seconde**. Au bout de deux secondes, cent vingt projectiles saturent l'écran, le joueur est mort et le framerate au sol.

```text
  SANS cooldown                              AVEC cooldown de 1,8 s

  frame 1  : tir                             t = 0.0  : tir, cooldown = 1.8
  frame 2  : tir                             t = 0.5  : cooldown = 1.3, rien
  frame 3  : tir                             t = 1.0  : cooldown = 0.8, rien
  ...                                        t = 1.8  : cooldown = 0.0, tir
  60 tirs par seconde                        0,55 tir par seconde
```

C'est très exactement le mécanisme d'attaque du joueur vu à la section 33.35, appliqué à l'ennemi.

### La version manuelle

```dart
double _cooldownTir = 0;

@override
void mettreAJourIA(double dt) {
  if (_cooldownTir > 0) {
    _cooldownTir -= dt;             // 1. on décrémente TOUJOURS
  }
  // ...
  if (_cooldownTir <= 0 && voitLeJoueur()) {
    tirer();                        // 2. `tirer()` remet le cooldown à 1.8
  }
}
```

Deux règles, et elles se ressemblent trompeusement :

> **Le cooldown se décrémente dans `update` / `mettreAJourIA`, jamais ailleurs.**
> **Le cooldown se recharge dans l'action, jamais dans le test.**

### La version `Timer` de Flame

```dart
final Timer _rechargement = Timer(
  Constantes.cooldownTirChauvesouris,
  autoStart: false,
);

@override
void update(double dt) {
  super.update(dt);
  _rechargement.update(dt);        // NE JAMAIS OUBLIER
}

bool get peutTirer => !_rechargement.isRunning();

void tirer() {
  if (!peutTirer) {
    return;
  }
  // ... création du projectile ...
  _rechargement.start();
}
```

Un `Timer` non mis à jour ne s'écoule jamais : `isRunning()` reste vrai pour l'éternité et la chauve-souris ne tire plus qu'une seule fois de toute sa vie. C'est le bug le plus fréquent de la section 33.32.

### Où le cooldown NE doit jamais vivre

```dart
// FAUX : le cooldown lié à la collision
@override
void onCollision(Set<Vector2> points, PositionComponent other) {
  _cooldownTir -= 0.016;   // et si aucune collision n'a lieu ?
}
```

Les callbacks de collision ne sont appelés **que s'il y a contact**. Tout ce qui doit s'écouler avec le temps appartient à `update`, sans exception.

---

## 37.22 — Les couches de collision : qui touche qui

### Le problème

À partir de maintenant, six familles d'objets portent une hitbox : joueur, zone d'attaque, plateforme, gobelin, chauve-souris, projectile. Cela fait quinze paires possibles, dont la plupart n'ont aucun sens. Il faut décider explicitement, une fois pour toutes, qui réagit à qui.

Flame n'offre pas de système de couches et de masques comme celui que vous aviez écrit à la main au chapitre 24. Il propose seulement `CollisionType` (chapitre 32.9) :

```text
  active   <-> active    : testé
  active   <-> passive   : testé
  passive  <-> passive   : NON testé
  inactive <-> quoi que ce soit : NON testé
```

Le filtrage fin se fait donc en deux temps :

1. `CollisionType` élimine les paires les plus nombreuses (tout le décor entre lui-même) ;
2. l'opérateur `is` dans les callbacks élimine le reste, à coût nul.

### La table d'attribution

| Composant | Hitbox | `CollisionType` | Justification |
| --- | --- | --- | --- |
| `Plateforme` | `RectangleHitbox` | **passive** | le décor ne cherche personne ; il est cherché |
| `Joueur` | `RectangleHitbox` | active | il se déplace et doit tout tester |
| `ZoneAttaque` | `RectangleHitbox` | active | elle ne vit que 0,16 s |
| `Gobelin` | `RectangleHitbox` | active | il se déplace, tombe, frappe |
| `Chauvesouris` | `CircleHitbox` | active | elle se déplace et frappe |
| `Projectile` | `CircleHitbox` | active | il file vite et doit tout tester |
| ennemi **mort** | (sa hitbox) | **inactive** | un cadavre ne blesse plus et ne bloque plus |

Une seule famille est passive, et c'est la plus nombreuse : une salle contient des dizaines de plateformes et une poignée d'entités. C'est exactement le bon arbitrage de performance (chapitre 32.11).

### La matrice « qui touche qui »

```text
                     |  Plate | Joueur | ZoneAtt | Gobelin | Chauve | Projec |
                     | forme  |        |         |         | souris | tile   |
  -------------------+--------+--------+---------+---------+--------+--------+
  Plateforme         |   --   | bloque |    .    | bloque  |   .    | detruit|
  Joueur             | bloque |   --   |    .    | degats  | degats | degats |
  ZoneAttaque        |   .    |   .    |   --    | frappe  | frappe |   .    |
  Gobelin            | bloque | degats |  frappe |    .    |   .    |   .    |
  Chauvesouris       |   .    | degats |  frappe |    .    |   --   |   .    |
  Projectile         |detruit | degats |    .    |    .    |   .    |   .    |
  -------------------+--------+--------+---------+---------+--------+--------+

  bloque  = résolution physique (repousser)
  degats  = perte de points de vie
  frappe  = dégâts infligés par l'attaque du joueur
  detruit = le projectile disparaît
  .       = contact ignoré
```

### Qui écrit la réaction : le côté unique

La matrice est symétrique, mais **le code ne doit l'être qu'une fois**. Si le gobelin inflige des dégâts au joueur dans son `onCollision` **et** que le joueur s'inflige des dégâts au contact d'un gobelin dans le sien, le joueur perd deux fois les points. C'est le patron « double dispatch » de la section 32.17, et la règle est simple :

> **Un effet, un seul propriétaire.**

| Interaction | Qui écrit le code | Pourquoi |
| --- | --- | --- |
| joueur ↔ plateforme | **Joueur** | c'est lui qui doit être repoussé |
| ennemi ↔ plateforme | **Ennemi** | idem |
| ennemi → joueur (contact) | **Ennemi** | il connaît `degatsContact` |
| zone d'attaque → ennemi | **ZoneAttaque** | elle connaît les dégâts de l'arme |
| projectile → joueur | **Projectile** | il connaît ses dégâts |
| projectile → plateforme | **Projectile** | c'est lui qui disparaît |

Aucune ligne de ce tableau n'apparaît deux fois. Écrivez-le avant de coder, relisez-le quand un chiffre de dégâts vous paraît doublé.

---

## 37.23 — Le joueur touche l'ennemi : la hitbox d'attaque

### Le principe

L'attaque du joueur n'est pas un test de distance : c'est une **hitbox temporaire** qui apparaît devant lui pendant quelques images.

```text
  sens = +1                                  sens = -1

   +--------+ +-------+                    +-------+ +--------+
   | JOUEUR | | ZONE  |                    | ZONE  | | JOUEUR |
   |        | |ATTAQUE|                    |ATTAQUE| |        |
   +--------+ +-------+                    +-------+ +--------+
   x=0        x=size.x                     x=-18     x=0

  Chronologie :
    t = 0.00 s   touche pressée -> la zone est ajoutée au joueur
    t = 0.16 s   la zone est retirée
    t = 0.45 s   fin du cooldown, on peut réattaquer
```

### Le composant

```dart
class ZoneAttaque extends PositionComponent with CollisionCallbacks {
  ZoneAttaque({
    required this.degats,
    required Vector2 position,
    required Vector2 size,
  }) : super(position: position, size: size, anchor: Anchor.topLeft);

  final double degats;

  /// Chaque ennemi ne peut être touché QU'UNE FOIS par un même coup.
  final Set<Ennemi> _dejaTouches = {};

  @override
  Future<void> onLoad() async {
    add(RectangleHitbox());
    add(
      RectangleComponent(
        size: size.clone(),
        paint: Paint()..color = Palette.attaque.withValues(alpha: 0.35),
      ),
    );
  }

  @override
  void onCollisionStart(
    Set<Vector2> intersectionPoints,
    PositionComponent other,
  ) {
    super.onCollisionStart(intersectionPoints, other);

    if (other is! Ennemi || !other.estVivant) {
      return;
    }
    if (!_dejaTouches.add(other)) {
      return;              // add() renvoie false : il était déjà dans le Set
    }

    other.subirDegats(degats);

    if (other.estVivant) {
      final versEnnemi = other.absoluteCenter.x - absoluteCenter.x;
      other.appliquerKnockback(Vector2(versEnnemi >= 0 ? 1 : -1, 0));
    }
  }
}
```

### `Set.add()` renvoie un booléen

```dart
if (!_dejaTouches.add(other)) {
  return;
}
```

`Set<T>.add(T)` renvoie `true` si l'élément **a été ajouté**, `false` s'il y était déjà (chapitre 06). Cette seule ligne remplace le classique `if (_dejaTouches.contains(x)) return; _dejaTouches.add(x);` et supprime le risque d'oublier la seconde moitié.

Pourquoi ce filtre est indispensable : `onCollisionStart` n'est appelé qu'une fois par contact, mais un ennemi peut **sortir puis rentrer** dans la zone pendant les 0,16 seconde qu'elle dure — par exemple parce que le knockback l'éjecte et qu'il revient. Sans le `Set`, un seul coup d'épée infligerait deux ou trois fois les dégâts.

### Le déclenchement, côté joueur

```dart
void attaquer() {
  if (_cooldownAttaque > 0 || etat == EtatJoueur.mort) {
    return;
  }

  _cooldownAttaque = Constantes.cooldownAttaqueJoueur;   // 0.45 s
  _tempsAttaque = Constantes.dureeAttaqueJoueur;         // 0.16 s
  etat = EtatJoueur.attaque;

  _zone = ZoneAttaque(
    degats: Constantes.degatsAttaqueJoueur,
    position: Vector2(sens > 0 ? size.x : -18, 4),
    size: Vector2(18, 22),
  );
  add(_zone!);
}

void _finirAttaque() {
  _zone?.removeFromParent();
  _zone = null;
  if (etat == EtatJoueur.attaque) {
    etat = EtatJoueur.immobile;
  }
}
```

La zone est un **enfant du joueur** : elle suit ses déplacements sans une ligne de code. Comme le joueur a `Anchor.topLeft`, la position `x = size.x` place la zone juste à droite du corps, et `x = -18` juste à gauche.

### Pourquoi pas un simple test de distance

```dart
// L'approche naïve
void attaquer() {
  for (final e in game.monde.children.whereType<Ennemi>()) {
    if (e.absoluteCenter.distanceTo(absoluteCenter) < 40) {
      e.subirDegats(25);
    }
  }
}
```

Elle fonctionne… et elle frappe **derrière** le joueur, à travers les murs, au-dessus et en dessous. Une hitbox orientée donne gratuitement : la direction, la portée, la hauteur du coup, et la cohérence avec le rendu — le joueur voit exactement la zone qui blesse.

---

## 37.24 — L'ennemi touche le joueur : dégâts au contact

### Le code, côté ennemi

```dart
// Dans Ennemi
@override
void onCollision(Set<Vector2> intersectionPoints, PositionComponent other) {
  super.onCollision(intersectionPoints, other);

  if (etat == EtatEnnemi.mort) {
    return;
  }

  if (other is Plateforme) {
    _resoudreCollision(other);
    return;
  }

  if (other is Joueur) {
    final versJoueur = other.absoluteCenter - absoluteCenter;
    versJoueur.y = 0;
    if (versJoueur.length2 < 0.01) {
      versJoueur.setValues(sens.toDouble(), 0);
    }
    other.subirDegats(degatsContact, direction: versJoueur.normalized());
  }
}
```

### `onCollision` et non `onCollisionStart` : pourquoi

C'est contre-intuitif, alors prenons le temps.

| Callback | Appelé | Conséquence pour les dégâts de contact |
| --- | --- | --- |
| `onCollisionStart` | une fois, au premier contact | si le joueur reste collé à l'ennemi après la fin de l'invincibilité, **plus rien ne se passe** |
| `onCollision` | à chaque frame de contact | les dégâts repartent dès que l'invincibilité expire |

Avec `onCollisionStart` seul, un joueur coincé dans un couloir contre un gobelin ne perdrait qu'une seule fois 12 PV, puis serait à l'abri en restant immobile — un comportement absurde. Avec `onCollision`, l'invincibilité de 1,2 seconde suffit à réguler la cadence : **12 PV toutes les 1,2 s**, soit environ 10 PV par seconde. Le filtre n'est pas dans le callback, il est dans `Joueur.subirDegats`.

```text
  t = 0.00  contact -> subirDegats(12)  pv 100 -> 88   invincible jusqu'à 1.2
  t = 0.02  contact -> ignoré (invincible)
  t = 0.04  contact -> ignoré
  ...       (72 appels ignorés)
  t = 1.22  contact -> subirDegats(12)  pv  88 -> 76   invincible jusqu'à 2.42
```

### Le vecteur de recul : pourquoi annuler Y

```dart
versJoueur.y = 0;
```

Sans cette ligne, un joueur qui saute sur la tête d'un gobelin serait projeté vers le bas, à travers le sol. En n'utilisant que la composante horizontale, on garantit un recul latéral, et c'est `Joueur._reculer` qui ajoutera la composante verticale (toujours vers le haut).

### Le cas du contact parfaitement centré

```dart
if (versJoueur.length2 < 0.01) {
  versJoueur.setValues(sens.toDouble(), 0);
}
```

Si les deux centres coïncident exactement, `normalized()` produirait `NaN`. On se rabat alors sur le sens de l'ennemi. `length2` (longueur au carré) évite une racine carrée inutile : c'est l'optimisation classique de toute comparaison de distances.

### Sauter sur la tête de l'ennemi : la variante « Mario »

Beaucoup de plateformers font mourir l'ennemi quand le joueur lui retombe dessus. La condition tient en deux tests :

```dart
final vientDuDessus = other.absoluteCenter.y < absoluteCenter.y - size.y * 0.25;
final tombe = other.velocite.y > 0;

if (vientDuDessus && tombe) {
  subirDegats(pvMax);            // l'ennemi meurt
  other.velocite.y = Constantes.forceSaut * 0.7;   // le joueur rebondit
  return;
}
```

Nous ne l'activons pas dans le jeu de base — le donjon repose sur l'épée — mais l'exercice 8 vous propose de l'ajouter.

---

## 37.25 — Le knockback (rappel chapitre 23)

### Rappel de la section 23.31

Le knockback est une **impulsion** : on écrase la vélocité au lieu de l'accumuler, puis on laisse l'amortissement ramener l'entité au repos.

```text
  impulsion :  velocite = direction * force        (on ÉCRASE)
  et non    :  velocite += direction * force       (on accumule -> ça s'emballe)
```

### Côté ennemi

```dart
// Dans Ennemi
double _tempsKnockback = 0;

bool get estRepousse => _tempsKnockback > 0;

void appliquerKnockback(
  Vector2 direction, [
  double force = Constantes.forceKnockback,
]) {
  velocite = direction.normalized() * force;
  velocite.y = -force * 0.35;              // un léger soulèvement
  _tempsKnockback = Constantes.dureeKnockback;   // 0.18 s
}
```

Et, dans `update` (section 37.6) :

```dart
if (_tempsKnockback > 0) {
  _tempsKnockback -= dt;
  velocite = velocite * 0.88;     // amortissement
} else {
  mettreAJourIA(dt);              // l'IA ne reprend qu'après le recul
}
```

### Le point le plus important : suspendre l'IA

Sans la suspension, `mettreAJourIA` réécrit `velocite.x = sens * vitesse` à la frame suivante et **efface le knockback**. Le joueur voit alors son coup ne produire aucun recul, ce qui donne l'impression que l'attaque n'a pas porté.

```text
  SANS suspension                        AVEC suspension

  frame 1  knockback : vx = +220         frame 1  knockback : vx = +220
  frame 2  IA        : vx = -60          frame 2  amorti    : vx = +194
  frame 3  IA        : vx = -60          frame 3  amorti    : vx = +171
           (recul invisible)             ...
                                         frame 12 l'IA reprend la main
```

### Côté joueur

```dart
// Dans Joueur
void _reculer(Vector2? direction) {
  final horizontal = direction == null || direction.x.abs() < 0.01
      ? -sens.toDouble()                  // repoussé vers l'arrière par défaut
      : (direction.x > 0 ? 1.0 : -1.0);

  velocite.x = horizontal * Constantes.forceKnockback;
  velocite.y = -Constantes.forceKnockback * 0.5;
}
```

Le joueur repoussé garde le contrôle après `_tempsTouche` (0,25 s) : pendant ce quart de seconde, `update` n'écrase pas `velocite.x` avec l'entrée clavier.

```dart
if (_tempsTouche <= 0) {
  velocite.x = _entreeX * Constantes.vitesseJoueur;
}
```

---

## 37.26 — Le flash blanc et le clignotement d'invincibilité

Deux retours visuels, deux rôles différents. Ne les confondez pas.

| Retour | Durée | Message envoyé au joueur |
| --- | --- | --- |
| **flash blanc** | 0,06 s | « ce coup a porté » |
| **clignotement** | 1,2 s | « je suis invulnérable en ce moment » |

### Le flash blanc, sur l'ennemi

Le mixin `Sante` appelle `onDegatsRecus` à chaque perte de PV : c'est l'endroit exact où brancher le flash.

```dart
// Dans Ennemi
@override
void onDegatsRecus(double degats) {
  corps.add(
    ColorEffect(
      Palette.blanc,
      EffectController(duration: 0.06, alternate: true),
      opacityTo: 1.0,
    ),
  );
  corps.add(
    ScaleEffect.by(
      Vector2(1.18, 0.86),
      EffectController(duration: 0.06, alternate: true),
    ),
  );
}
```

Deux détails techniques importants.

**L'effet s'applique à `corps`, pas à `this`.** `ColorEffect`, `OpacityEffect` et `ScaleEffect` exigent une cible qui possède un `Paint`, c'est-à-dire le mixin `HasPaint`. Notre `Ennemi` est un `PositionComponent` nu : il n'a pas de peinture. Son enfant `corps`, lui, est un `RectangleComponent`, qui en a une. C'est la raison profonde de la structure « conteneur logique + enfant visuel » adoptée dès la section 37.1.

```text
  Ennemi (PositionComponent)      <- position, hitbox, IA : PAS de Paint
    ├── corps (RectangleComponent) <- couleur, effets visuels : HasPaint
    │      └── oeil (RectangleComponent)
    └── hitbox (ShapeHitbox)
```

**`alternate: true` garantit le retour à l'état initial.** L'effet va de 0 % à 100 % puis revient à 0 % : la couleur d'origine est restaurée exactement, sans code de nettoyage. Le rappel du chapitre 33 vaut d'être répété : `duration: 0.06, alternate: true` dure **0,12 seconde au total**.

### Le clignotement, sur le joueur

```dart
// Dans Joueur
void _clignoter() {
  // duree totale = duration x 2 x repeatCount   (le x2 vient de alternate)
  final repetitions =
      (Constantes.dureeInvincibilite / (0.075 * 2)).round();   // 1.2 / 0.15 = 8

  corps.add(
    OpacityEffect.to(
      0.25,
      EffectController(duration: 0.075, alternate: true, repeatCount: repetitions),
      onComplete: () => corps.opacity = 1.0,   // filet de sécurité
    ),
  );
}
```

Le calcul du nombre de répétitions est celui de la section 33.36 :

```text
  duree_totale = duration x 2 x repeatCount
  1.2 = 0.075 x 2 x repeatCount
  repeatCount = 1.2 / 0.15 = 8
```

Le `onComplete` qui remet l'opacité à 1 est une assurance : si un jour vous changez `dureeInvincibilite` sans que le calcul tombe juste, le joueur ne restera pas à moitié transparent pour l'éternité.

### La règle d'or des effets

> **Un effet se déclenche sur une transition, jamais sur un état.**

```dart
// INTERDIT : soixante effets créés par seconde
@override
void update(double dt) {
  super.update(dt);
  if (invincible) {
    corps.add(OpacityEffect.to(0.25, EffectController(duration: 0.075)));
  }
}
```

C'est « l'erreur du siècle » de la section 33.18. Nos deux effets sont déclenchés depuis `onDegatsRecus` et depuis `subirDegats`, qui sont des **événements**, appelés une fois chacun.

---

## 37.27 — La mort d'un ennemi : effet, score, retrait

### Les cinq gestes obligatoires

```text
  1. VERROUILLER   etat = mort, pour que rien ne se redéclenche
  2. DÉSARMER      hitbox inactive : un cadavre ne blesse plus, ne bloque plus
  3. RÉCOMPENSER   ajouter les points au score
  4. CÉLÉBRER      particules + effet de disparition
  5. RETIRER       removeFromParent(), une fois l'effet terminé
```

Sauter l'étape 2 produit le bug le plus désagréable de la catégorie : le joueur avance dans le cadavre en train de disparaître et perd 12 PV.

### Le code

```dart
// Dans Ennemi
@override
void onMort() => mourir();

void mourir() {
  if (etat == EtatEnnemi.mort) {
    return;                                   // 1. garde d'idempotence
  }
  etat = EtatEnnemi.mort;
  velocite.setZero();
  hitbox.collisionType = CollisionType.inactive;   // 2. désarmé

  game.ajouterScore(pointsScore);                  // 3. récompense

  game.monde.add(                                  // 4. célébration
    ParticleSystemComponent(
      position: absoluteCenter.clone(),
      priority: 20,
      particle: _gerbeDeMort(),
    ),
  );

  corps.add(
    SequenceEffect(
      [
        ScaleEffect.to(Vector2.all(1.35), EffectController(duration: 0.08)),
        ScaleEffect.to(Vector2.zero(), EffectController(duration: 0.16)),
      ],
      onComplete: removeFromParent,                // 5. retrait
    ),
  );
}

Particle _gerbeDeMort() {
  final rnd = Random();
  return Particle.generate(
    count: 14,
    lifespan: 0.5,
    generator: (i) {
      final angle = rnd.nextDouble() * tau;
      final force = 40 + rnd.nextDouble() * 110;
      return AcceleratedParticle(
        speed: Vector2(cos(angle), sin(angle)) * force,
        acceleration: Vector2(0, 320),
        child: CircleParticle(
          radius: 1.5 + rnd.nextDouble() * 2,
          paint: Paint()..color = couleur,
        ),
      );
    },
  );
}
```

### La garde d'idempotence

```dart
if (etat == EtatEnnemi.mort) {
  return;
}
```

Sans elle, un gobelin touché simultanément par l'épée et par un projectile mourrait deux fois : deux gerbes de particules, **deux fois le score**, deux appels à `removeFromParent`. Toute méthode qui produit un effet définitif doit commencer par vérifier qu'elle n'a pas déjà été appelée. C'est une habitude à prendre pour la vie.

### Pourquoi les particules vont dans le monde, pas dans l'ennemi

```dart
game.monde.add(ParticleSystemComponent(position: absoluteCenter.clone(), ...));
```

Si les particules étaient un enfant de l'ennemi, elles disparaîtraient avec lui à la fin de la séquence — au bout de 0,24 seconde au lieu de 0,5. En les ajoutant au monde, à la position absolue de l'ennemi, elles vivent leur vie indépendamment. Même raison pour `.clone()` : sans lui, elles suivraient un `Vector2` qui appartient à un composant en cours de retrait.

### `onComplete: removeFromParent`

`onComplete` est un `void Function()?` et `removeFromParent` est une méthode sans argument qui renvoie `void` : on peut donc la passer **par référence** (chapitre 07), sans l'envelopper dans `() => removeFromParent()`. Rappel de la section 33.18 : `onComplete` est appelé **avant** que l'effet ne se retire, le composant est donc encore dans l'arbre.

### Le score

`ajouterScore` appartient officiellement au chapitre 38, avec le HUD. Nous en écrivons dès maintenant la version minimale, que le chapitre 38 complétera avec l'affichage et le meilleur score :

```dart
// Dans DonjonGame
void ajouterScore(int points) {
  score += points;
  if (score > meilleurScore) {
    meilleurScore = score;
  }
}
```

**Résultat attendu :**

```text
Coup d'épée : 25 dégâts   -> gobelin pv 30 -> 5,  flash blanc, recul
Coup d'épée : 25 dégâts   -> gobelin pv  5 -> 0
  etat -> mort
  hitbox -> inactive
  score  : 0 -> 50
  14 particules vertes
  le corps gonfle puis se réduit à zéro en 0,24 s
  le gobelin quitte l'arbre de composants
```

---

## 37.28 — `perdreUneVie()` et le respawn du joueur

### La chaîne complète

```text
  Projectile / contact ennemi
        -> Joueur.subirDegats(12)
        -> [mixin Sante] pv passe à 0
        -> [mixin Sante] onMort()
        -> Joueur.mourir()
              etat = mort, hitbox inactive, effet de chute
              attente de 0,9 s
        -> DonjonGame.perdreUneVie()
              vies--
              vies == 0 ?  -> changerEtat(GameState.gameOver)
              sinon        -> joueur.reapparaitre(pointDeReapparition)
```

### `Joueur.mourir()`

```dart
@override
void onMort() => mourir();

void mourir() {
  if (etat == EtatJoueur.mort) {
    return;
  }
  etat = EtatJoueur.mort;
  velocite.setZero();
  hitbox.collisionType = CollisionType.inactive;
  _finirAttaque();
  _tempsMort = 0.9;                       // délai avant la suite
  corps.add(RotateEffect.by(pi / 2, EffectController(duration: 0.35)));
}
```

Le délai de 0,9 seconde est essentiel : sans lui, le joueur réapparaîtrait dans la même image que sa mort, sans que le joueur humain ait le temps de comprendre ce qui s'est passé.

```dart
// Dans Joueur.update
if (etat == EtatJoueur.mort) {
  _tempsMort -= dt;
  if (_tempsMort <= 0) {
    _tempsMort = double.infinity;   // pour ne déclencher qu'UNE fois
    game.perdreUneVie();
  }
  return;                            // un mort ne bouge plus
}
```

L'astuce `_tempsMort = double.infinity` évite d'appeler `perdreUneVie()` soixante fois par seconde en attendant la reconstruction de la scène. On pourrait aussi utiliser un booléen `_mortTraitee` ; le résultat est le même.

### `DonjonGame.perdreUneVie()`

```dart
void perdreUneVie() {
  vies--;

  if (vies <= 0) {
    vies = 0;
    changerEtat(GameState.gameOver);
    return;
  }

  joueur.reapparaitre(pointDeReapparition);
}
```

Quatre lignes, une seule responsabilité : décrémenter et arbitrer. Le jeu ne sait pas **comment** le joueur réapparaît, il sait seulement qu'il doit réapparaître. Le chapitre 40 branchera ici le son et l'écran de Game Over définitif.

### `Joueur.reapparaitre()`

```dart
void reapparaitre(Vector2 point) {
  position = point.clone();
  velocite.setZero();

  pv = pvMax;                         // affectation directe : `soigner`
                                      // refuse de soigner un mort
  etat = EtatJoueur.immobile;
  auSol = false;
  sens = 1;

  corps.angle = 0;                    // on annule la rotation de la mort
  corps.opacity = 1.0;
  corps.scale = Vector2.all(1);
  hitbox.collisionType = CollisionType.active;

  _tempsMort = 0;
  _tempsTouche = 0;
  _cooldownAttaque = 0;

  invincible = true;                  // invincibilité de courtoisie
  _tempsInvincible = Constantes.dureeInvincibilite;
  _clignoter();
}
```

### La liste de contrôle du respawn

C'est la méthode où l'on oublie le plus de choses. Servez-vous de ce tableau comme d'une check-list, il vaut pour n'importe quel jeu :

| À remettre à zéro | Symptôme si on l'oublie |
| --- | --- |
| `position` | le joueur réapparaît là où il est mort, et remeurt |
| `velocite` | il repart avec la vitesse de sa chute mortelle |
| `pv` | il réapparaît avec 0 PV et remeurt immédiatement |
| `etat` | il réapparaît dans l'état `mort` : plus aucun contrôle |
| `hitbox.collisionType` | il traverse le décor pour toujours |
| angle / échelle / opacité du corps | il reste couché, minuscule ou transparent |
| cooldowns et minuteurs | son épée reste bloquée quelques secondes |
| invincibilité | il remeurt instantanément s'il réapparaît près d'un ennemi |

---

## 37.29 — Le point de réapparition

### Le champ

```dart
// Dans DonjonGame
Vector2 pointDeReapparition = Vector2(48, 160);
```

Il est renseigné au moment où la salle est construite :

```dart
Future<void> _construireSalle() async {
  // ... les plateformes ...

  pointDeReapparition = Vector2(48, 160);

  joueur = Joueur(position: pointDeReapparition.clone());
  await monde.add(joueur);
}
```

Le `.clone()` n'est pas décoratif : sans lui, `joueur.position` et `pointDeReapparition` seraient le **même objet**, et le point de réapparition suivrait le joueur. Le joueur réapparaîtrait donc exactement là où il vient de mourir. C'est un bug redoutable, parce qu'il est invisible tant qu'on teste le respawn près du départ.

### La règle du placement

```text
  MAUVAIS point de réapparition        BON point de réapparition

     J                                    J
  ####  g  ####                        ####          ####
        ^                                   le joueur a le temps
        l'ennemi est à 20 px :              de comprendre où il est
        mort instantanée en boucle
```

Un point de réapparition doit être :

1. sur un sol stable (pas au-dessus du vide) ;
2. à plus de 100 pixels de tout ennemi ;
3. avec une vue sur ce qui a tué le joueur, pour qu'il comprenne son erreur.

L'invincibilité de courtoisie de 1,2 seconde à la réapparition (section 37.28) est le filet de sécurité qui couvre les cas restants.

---

## 37.30 — Équilibrer : le tableau des PV et des dégâts

L'équilibrage n'est pas une question de goût : il se calcule, puis se vérifie manette en main.

### Les valeurs du jeu

| Entité | PV | Dégâts infligés | Portée | Cadence |
| --- | --- | --- | --- | --- |
| Joueur | 100 | 25 (épée) | 18 px devant | 1 coup / 0,45 s |
| Gobelin | 30 | 12 (contact) | contact | 1 fois / 1,2 s (invincibilité) |
| Chauve-souris | 15 | 8 (contact) | contact | 1 fois / 1,2 s |
| Projectile | — | 10 | 3 s de vol | 1 tir / 1,8 s |

### Les deux chiffres qui comptent

**Combien de coups pour tuer (TTK, *time to kill*).**

```text
  Gobelin      : 30 PV / 25 dégâts = 1.2 -> 2 coups
  Chauve-souris: 15 PV / 25 dégâts = 0.6 -> 1 coup
  Boss (ch. 39): 300 PV / 25       = 12  -> 12 coups
```

Deux coups pour un gobelin est la bonne valeur : un seul coup rendrait l'ennemi insignifiant, trois coups rendraient chaque rencontre longue.

**Combien de fautes avant de mourir (TTD, *time to die*).**

```text
  100 PV / 12 dégâts = 8.3 -> 9 contacts de gobelin
  100 PV / 10 dégâts = 10  -> 10 projectiles
  Mélange réaliste   : environ 8 fautes
```

Avec trois vies, le joueur dispose donc d'environ vingt-quatre fautes pour terminer un niveau. C'est généreux — c'est voulu pour un premier niveau.

### La formule de dimensionnement

```text
  PV_ennemi = degats_joueur x nombre_de_coups_voulu
  PV_joueur = degats_ennemi x nombre_de_fautes_tolerees
```

Vous choisissez d'abord l'**expérience** (deux coups, huit fautes), et vous en déduisez les chiffres. L'inverse — poser des chiffres et découvrir le ressenti — mène à des semaines de tâtonnement.

### La règle des nombres ronds

Prenez toujours des dégâts qui **divisent** les PV, ou presque :

| Dégâts | PV cible | Coups | Verdict |
| --- | --- | --- | --- |
| 25 | 100 | 4 | lisible |
| 25 | 30 | 2 (avec 20 de gâchis) | acceptable |
| 7 | 100 | 15 | illisible pour le joueur |
| 33 | 100 | 4 (avec 32 de gâchis) | frustrant : le dernier coup semble injuste |

Le joueur compte inconsciemment les coups. Des nombres ronds rendent le combat prévisible, donc maîtrisable.

---

## 37.31 — Ce que le projet fait à la fin de ce chapitre

### La liste de ce qui fonctionne

Lancez `flutter run`, cliquez sur « Jouer » :

1. la salle apparaît avec ses plateformes, deux gobelins et deux chauves-souris ;
2. les gobelins patrouillent chacun autour de leur point d'apparition et **font demi-tour au bord du vide** ;
3. les chauves-souris ondulent, chacune avec sa propre phase ;
4. approchez-vous : le gobelin le plus proche se retourne et vous poursuit ;
5. cachez-vous derrière un pilier : il vous perd de vue et rentre chez lui ;
6. la chauve-souris vous tire dessus toutes les 1,8 seconde et pique quand vous êtes près ;
7. prenez un coup : flash rouge, recul, clignotement d'une seconde et deux dixièmes, aucun dégât pendant ce temps ;
8. frappez : une zone claire apparaît devant vous, le gobelin devient blanc, s'écrase, recule ;
9. frappez encore : il gonfle, éclate en quatorze particules vertes, disparaît, le score augmente de 50 ;
10. tombez à 0 PV : le joueur bascule, une seconde s'écoule, vous réapparaissez au point de départ avec 100 PV et une invincibilité de courtoisie ;
11. mourez trois fois : l'overlay « Game Over » s'affiche.

### L'arborescence à jour

```text
  donjon_de_dart/
  ├── pubspec.yaml
  └── lib/
      ├── main.dart                     [modifié] overlay Game Over provisoire
      ├── donjon_game.dart              [modifié] perdreUneVie, ajouterScore, salle
      ├── config/
      │   ├── constantes.dart           [modifié] constantes de combat
      │   └── palette.dart              [modifié] couleurs des ennemis
      ├── core/
      │   ├── game_state.dart
      │   ├── entite.dart
      │   └── sante.dart                [CRÉÉ]    mixin Sante
      ├── composants/
      │   ├── joueur.dart               [modifié] Sante, attaque, dégâts, mort
      │   ├── plateforme.dart
      │   ├── ennemi.dart               [CRÉÉ]    EtatEnnemi + Ennemi (abstrait)
      │   ├── gobelin.dart              [CRÉÉ]
      │   ├── chauvesouris.dart         [CRÉÉ]
      │   └── projectile.dart           [CRÉÉ]
      └── ecrans/
          └── menu_principal.dart
```

### Ce qui manque encore, et quand ça arrive

| Manque | Chapitre |
| --- | --- |
| voir ses PV, ses vies et son score à l'écran | 38 (HUD, barres de vie) |
| ramasser des pièces, des potions, des clés | 38 |
| plusieurs salles, des portes, un boss | 39 |
| des sons, une vraie pause, un vrai Game Over, la sauvegarde | 40 |

---

## 37.32 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `LateInitializationError: Field 'pv' has not been initialized` | le mixin `Sante` déclare `late double pv;` et personne n'a appelé `initialiserSante` | appeler `initialiserSante(...)` en **première ligne** de `onLoad()` |
| L'ennemi est invisible et traverse tout | `await super.onLoad();` oublié dans la sous-classe : ni corps, ni hitbox | première ligne de tout `onLoad()` de sous-classe |
| L'ennemi vibre sur place au bord de sa zone | demi-tour testé sans tenir compte du sens de marche | `if (sens > 0 && x >= borneDroite)` et non `if (x >= borneDroite)` |
| L'ennemi part tout droit à l'infini | `ancre = position;` : les deux vecteurs sont le **même objet** mutable | `ancre = position.clone();` |
| Le knockback n'a aucun effet visible | `mettreAJourIA` réécrit `velocite` dès la frame suivante | suspendre l'IA pendant `_tempsKnockback` |
| Le joueur perd toutes ses vies en trois images | dégâts appliqués à chaque frame de contact sans invincibilité | filtrer sur `invincible` dans `Joueur.subirDegats` |
| Le joueur perd deux fois les points de vie | les deux côtés de la collision infligent des dégâts | un effet, un seul propriétaire (tableau 37.22) |
| Un seul coup d'épée tue un gobelin à 30 PV en un coup de 25 | l'ennemi est frappé plusieurs fois par la même zone d'attaque | mémoriser les cibles dans un `Set<Ennemi>` |
| Le gobelin marche dans le vide et tombe | aucune sonde de sol | `solDevant()` par raycast, testé **avant** de bouger |
| L'ennemi poursuit le joueur à travers un mur épais | rayon d'aggro sans ligne de vue | `voitLeJoueur()` avec `raycast` filtré sur `Plateforme` |
| Le raycast touche toujours quelque chose à distance 0 | le rayon part de l'intérieur de la hitbox de l'ennemi | `hitboxFilter` ou `ignoreHitboxes: [hitbox]` |
| Le projectile disparaît instantanément ou file à l'infini | direction non normalisée, ou vecteur nul normalisé (`NaN`) | `normalized()` **et** garde `if (vers.length < 1) return;` |
| Le framerate s'effondre après deux minutes | les projectiles sortis de l'écran ne sont jamais retirés | durée de vie obligatoire (`_tempsRestant`, `RemoveEffect`) |
| La chauve-souris tire soixante fois par seconde | aucun cooldown, ou cooldown jamais décrémenté | décrémenter dans `update`, recharger dans `tirer()` |
| Un `Timer` de Flame ne se termine jamais | `timer.update(dt)` oublié dans `update` | appeler `update(dt)` sur chaque `Timer` manuel |
| `ColorEffect` / `OpacityEffect` : « The target does not have a Paint » | l'effet est appliqué au `PositionComponent` conteneur | appliquer l'effet à l'enfant visuel (`corps`) |
| Le joueur reste à moitié transparent pour toujours | `repeatCount` mal calculé, ou effet infini interrompu | `duree = duration x 2 x repeatCount` et `onComplete: () => corps.opacity = 1.0` |
| Le cadavre d'un ennemi blesse encore | hitbox laissée active pendant l'effet de disparition | `hitbox.collisionType = CollisionType.inactive;` dans `mourir()` |
| Le score double à chaque mort d'ennemi | `mourir()` appelée deux fois (épée + projectile) | garde d'idempotence `if (etat == EtatEnnemi.mort) return;` |
| Le joueur réapparaît là où il vient de mourir | `pointDeReapparition` partage l'objet `Vector2` du joueur | `.clone()` au moment de l'affectation |
| Le joueur réapparaît couché, minuscule ou fantomatique | angle, échelle et opacité du corps non réinitialisés | suivre la check-list de respawn (37.28) |
| `perdreUneVie()` appelée soixante fois par seconde | rien ne verrouille le déclenchement dans `update` | `_tempsMort = double.infinity;` après le premier appel |
| Les particules de mort disparaissent aussitôt | elles sont enfants de l'ennemi, retiré juste après | les ajouter à `game.monde` avec `absoluteCenter.clone()` |
| L'ennemi tremble entre poursuite et retour | `rayonAggro == rayonAbandon` | hystérésis : `rayonAbandon` ≈ 1,5 × `rayonAggro` |

---

## 37.33 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| `mixin Sante on PositionComponent` | des PV réutilisables, sans occuper la place de l'héritage |
| `late double pv` | oblige à appeler `initialiserSante(...)` dans `onLoad` |
| Méthode modèle | `subirDegats` fixe le déroulé, `onDegatsRecus` / `onMort` sont les trous |
| Mixin vs classe mère | « sait faire » = mixin, « est un » = classe mère |
| `abstract class Ennemi` | non instanciable, impose `mettreAJourIA` aux sous-classes |
| Inversion de contrôle | `Ennemi.update` appelle l'IA ; l'IA n'écrit que dans `velocite`, `sens`, `etat` |
| `bool get subitGravite` | un getter redéfini remplace un `update` dupliqué |
| Patrouille | bornes calculées autour de `ancre`, demi-tour dépendant du sens |
| `solDevant()` | un raycast vertical : l'absence de sol n'est jamais un événement de collision |
| `Ray2` + `raycast` | direction **normalisée**, `maxDistance`, `hitboxFilter`, `null` = rien touché |
| Rayon d'aggro | distance **et** ligne de vue ; test le moins cher en premier |
| Hystérésis | `rayonAbandon` > `rayonAggro`, zone morte de 2 px sur l'orientation |
| Machine à états | un `switch` sur l'enum, une méthode par état, transitions locales |
| État `mort` | terminal, atteint depuis n'importe où via `Sante.onMort()` |
| Mouvement sinusoïdal | dérivée `A·ω·cos(ω·t)` plus un rappel élastique contre la dérive |
| Piqué | direction **mémorisée** au déclenchement : sinon l'attaque est inesquivable |
| `Projectile` | direction normalisée dans le constructeur, durée de vie obligatoire |
| Durée de vie | `_tempsRestant`, `RemoveEffect(delay:)` ou `TimerComponent` — jamais la sortie d'écran |
| Cooldown | décrémenté dans `update`, rechargé dans l'action |
| `CollisionType` | décor **passif**, entités **actives**, cadavres **inactifs** |
| Un effet, un propriétaire | chaque interaction n'est codée que d'un seul côté |
| `ZoneAttaque` | hitbox temporaire, enfant du joueur, avec un `Set` anti-double-coup |
| Dégâts de contact | dans `onCollision` (pas `onCollisionStart`), régulés par l'invincibilité |
| Knockback | impulsion qui **écrase** la vélocité, IA suspendue pendant le recul |
| Flash et clignotement | effets sur l'enfant visuel `corps` (`HasPaint`), déclenchés sur un événement |
| Mort d'un ennemi | verrouiller, désarmer, récompenser, célébrer, retirer |
| `perdreUneVie()` | décrémente et arbitre ; c'est le joueur qui sait se réinitialiser |
| Respawn | check-list : position, vélocité, PV, état, hitbox, visuel, minuteurs, invincibilité |
| Équilibrage | choisir l'expérience (2 coups, 8 fautes) puis en déduire les chiffres |

---

## 37.34 — Code complet du chapitre

Les dix fichiers ci-dessous forment un projet complet, compilable et jouable. Copiez-les tels quels.

### `lib/config/constantes.dart` (modifié)

```dart
class Constantes {
  // ---- Chapitre 35 ----
  static const double tailleTuile = 32.0;
  static const double gravite = 1200.0;
  static const double vitesseJoueur = 180.0;
  static const double forceSaut = -520.0;
  static const double vitesseMaxChute = 900.0;
  static const int viesDepart = 3;
  static const double pvJoueurMax = 100.0;
  static const double dureeInvincibilite = 1.2;
  static const double zoomCamera = 2.0;
  static const int nombreNiveaux = 3;

  // ---- Chapitre 37 : combat du joueur ----
  static const double degatsAttaqueJoueur = 25.0;
  static const double dureeAttaqueJoueur = 0.16;
  static const double cooldownAttaqueJoueur = 0.45;
  static const double forceKnockback = 220.0;
  static const double dureeKnockback = 0.18;

  // ---- Chapitre 37 : gobelin ----
  static const double pvGobelin = 30.0;
  static const double vitesseGobelin = 60.0;
  static const double degatsContactGobelin = 12.0;
  static const double rayonAggroGobelin = 140.0;
  static const double rayonAbandonGobelin = 220.0;
  static const double rayonAttaqueGobelin = 28.0;
  static const int scoreGobelin = 50;

  // ---- Chapitre 37 : chauve-souris ----
  static const double pvChauvesouris = 15.0;
  static const double vitesseChauvesouris = 90.0;
  static const double degatsContactChauvesouris = 8.0;
  static const double rayonAggroChauvesouris = 180.0;
  static const double rayonAbandonChauvesouris = 260.0;
  static const double rayonAttaqueChauvesouris = 34.0;
  static const double dureePiqueChauvesouris = 0.45;
  static const double cooldownTirChauvesouris = 1.8;
  static const int scoreChauvesouris = 35;

  // ---- Chapitre 37 : commun aux ennemis ----
  static const double dureeAttaqueEnnemi = 0.5;

  // ---- Chapitre 37 : projectiles ----
  static const double vitesseProjectile = 170.0;
  static const double degatsProjectile = 10.0;
  static const double dureeVieProjectile = 3.0;
}
```

### `lib/config/palette.dart` (modifié)

```dart
import 'package:flutter/material.dart';

class Palette {
  // ---- Chapitre 35 ----
  static const Color fond = Color(0xFF12121A);
  static const Color mur = Color(0xFF3A3F58);
  static const Color plateforme = Color(0xFF5A6484);
  static const Color joueur = Color(0xFF4CC3FF);
  static const Color texte = Color(0xFFF2F2F2);
  static const Color accent = Color(0xFFFFC048);

  // ---- Chapitre 37 ----
  static const Color blanc = Color(0xFFFFFFFF);
  static const Color gobelin = Color(0xFF6ABE30);
  static const Color chauvesouris = Color(0xFF9B59B6);
  static const Color projectile = Color(0xFFFF6B6B);
  static const Color attaque = Color(0xFFFFF3B0);
  static const Color oeil = Color(0xFF1B1B24);
}
```

### `lib/core/sante.dart` (créé)

```dart
import 'package:flame/components.dart';

/// Points de vie d'une entité du donjon.
mixin Sante on PositionComponent {
  late double pvMax;
  late double pv;

  bool get estVivant => pv > 0;

  double get ratioPv => pvMax <= 0 ? 0 : (pv / pvMax).clamp(0.0, 1.0);

  /// À appeler dans `onLoad`, AVANT toute lecture de `pv`.
  void initialiserSante(double valeurMax) {
    pvMax = valeurMax;
    pv = valeurMax;
  }

  void subirDegats(double degats) {
    if (degats <= 0 || !estVivant) {
      return;
    }
    pv -= degats;
    if (pv < 0) {
      pv = 0;
    }
    onDegatsRecus(degats);
    if (!estVivant) {
      onMort();
    }
  }

  void soigner(double points) {
    if (points <= 0 || !estVivant) {
      return;
    }
    pv += points;
    if (pv > pvMax) {
      pv = pvMax;
    }
    onSoinRecu(points);
  }

  void onDegatsRecus(double degats) {}

  void onSoinRecu(double points) {}

  void onMort() {}
}
```

### `lib/composants/ennemi.dart` (créé)

```dart
import 'dart:math';

import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/geometry.dart';
import 'package:flame/particles.dart';
import 'package:flutter/material.dart';

import '../config/constantes.dart';
import '../config/palette.dart';
import '../core/sante.dart';
import '../donjon_game.dart';
import 'joueur.dart';
import 'plateforme.dart';

enum EtatEnnemi { patrouille, poursuite, attaque, retour, mort }

abstract class Ennemi extends PositionComponent
    with HasGameReference<DonjonGame>, Sante, CollisionCallbacks {
  Ennemi({
    required Vector2 position,
    required Vector2 taille,
    required this.vitesse,
    required this.degatsContact,
    required this.pointsScore,
    required this.couleur,
    required double pvDepart,
  })  : _pvDepart = pvDepart,
        super(position: position, size: taille, anchor: Anchor.topLeft);

  // Réglages de l'espèce.
  double vitesse;
  double degatsContact;
  int pointsScore;
  final Color couleur;
  final double _pvDepart;

  // État courant.
  EtatEnnemi _etat = EtatEnnemi.patrouille;
  Vector2 velocite = Vector2.zero();
  int sens = 1;
  bool auSol = false;
  late Vector2 ancre;

  // Portées, ajustées par espèce dans onLoad.
  double rayonAggro = 140;
  double rayonAbandon = 220;
  double rayonAttaque = 26;
  double demiPatrouille = 3 * Constantes.tailleTuile;

  double _tempsKnockback = 0;

  late final RectangleComponent corps;
  late final RectangleComponent oeil;
  late final ShapeHitbox hitbox;

  bool get subitGravite => true;

  bool get estRepousse => _tempsKnockback > 0;

  EtatEnnemi get etat => _etat;

  set etat(EtatEnnemi nouveau) {
    if (nouveau == _etat) {
      return;
    }
    if (debugMode) {
      // ignore: avoid_print
      print('$runtimeType : $_etat -> $nouveau');
    }
    _etat = nouveau;
  }

  /// Redéfinissable : la chauve-souris préfère un cercle.
  ShapeHitbox creerHitbox() => RectangleHitbox();

  @override
  Future<void> onLoad() async {
    initialiserSante(_pvDepart);
    ancre = position.clone();

    corps = RectangleComponent(
      size: size.clone(),
      position: size / 2,
      anchor: Anchor.center,
      paint: Paint()..color = couleur,
    );
    oeil = RectangleComponent(
      size: Vector2.all(4),
      position: Vector2(size.x - 7, 5),
      paint: Paint()..color = Palette.oeil,
    );
    corps.add(oeil);
    await add(corps);

    hitbox = creerHitbox();
    await add(hitbox);
  }

  @override
  void update(double dt) {
    super.update(dt);

    if (etat == EtatEnnemi.mort) {
      return;
    }

    if (_tempsKnockback > 0) {
      _tempsKnockback -= dt;
      velocite = velocite * 0.88;
    } else {
      mettreAJourIA(dt);
    }

    if (subitGravite) {
      velocite.y += Constantes.gravite * dt;
      if (velocite.y > Constantes.vitesseMaxChute) {
        velocite.y = Constantes.vitesseMaxChute;
      }
    }

    auSol = false;
    position += velocite * dt;

    oeil.position.x = sens > 0 ? size.x - 7 : 3;
  }

  /// La décision : écrite par chaque espèce.
  void mettreAJourIA(double dt);

  // ---------------------------------------------------------------- capteurs

  double get distanceAuJoueur {
    final cible = game.joueur;
    if (!cible.isMounted) {
      return double.infinity;
    }
    return (cible.absoluteCenter - absoluteCenter).length;
  }

  bool solDevant() {
    final origine =
        absoluteCenter + Vector2(sens * size.x * 0.6, size.y * 0.5 - 1);
    final resultat = game.collisionDetection.raycast(
      Ray2(origin: origine, direction: Vector2(0, 1)),
      maxDistance: Constantes.tailleTuile * 0.75,
      hitboxFilter: (candidate) => candidate.parent is Plateforme,
    );
    return resultat != null;
  }

  bool murDevant() {
    final origine = absoluteCenter + Vector2(0, size.y * 0.25);
    final resultat = game.collisionDetection.raycast(
      Ray2(origin: origine, direction: Vector2(sens.toDouble(), 0)),
      maxDistance: size.x * 0.5 + 4,
      hitboxFilter: (candidate) => candidate.parent is Plateforme,
    );
    return resultat != null;
  }

  bool voitLeJoueur() {
    final cible = game.joueur;
    if (!cible.isMounted || !cible.estVivant) {
      return false;
    }

    final vers = cible.absoluteCenter - absoluteCenter;
    final distance = vers.length;
    if (distance > rayonAggro) {
      return false;
    }
    if (distance < 1) {
      return true;
    }

    final resultat = game.collisionDetection.raycast(
      Ray2(origin: absoluteCenter.clone(), direction: vers.normalized()),
      maxDistance: distance,
      hitboxFilter: (candidate) => candidate.parent is Plateforme,
    );
    return resultat == null;
  }

  // --------------------------------------------------------------- réactions

  void appliquerKnockback(
    Vector2 direction, [
    double force = Constantes.forceKnockback,
  ]) {
    velocite = direction.normalized() * force;
    velocite.y = -force * 0.35;
    _tempsKnockback = Constantes.dureeKnockback;
  }

  @override
  void onDegatsRecus(double degats) {
    corps.add(
      ColorEffect(
        Palette.blanc,
        EffectController(duration: 0.06, alternate: true),
        opacityTo: 1.0,
      ),
    );
    corps.add(
      ScaleEffect.by(
        Vector2(1.18, 0.86),
        EffectController(duration: 0.06, alternate: true),
      ),
    );
  }

  @override
  void onMort() => mourir();

  void mourir() {
    if (etat == EtatEnnemi.mort) {
      return;
    }
    etat = EtatEnnemi.mort;
    velocite.setZero();
    hitbox.collisionType = CollisionType.inactive;

    game.ajouterScore(pointsScore);

    game.monde.add(
      ParticleSystemComponent(
        position: absoluteCenter.clone(),
        priority: 20,
        particle: _gerbeDeMort(),
      ),
    );

    corps.add(
      SequenceEffect(
        [
          ScaleEffect.to(Vector2.all(1.35), EffectController(duration: 0.08)),
          ScaleEffect.to(Vector2.zero(), EffectController(duration: 0.16)),
        ],
        onComplete: removeFromParent,
      ),
    );
  }

  Particle _gerbeDeMort() {
    final rnd = Random();
    return Particle.generate(
      count: 14,
      lifespan: 0.5,
      generator: (i) {
        final angle = rnd.nextDouble() * tau;
        final force = 40 + rnd.nextDouble() * 110;
        return AcceleratedParticle(
          speed: Vector2(cos(angle), sin(angle)) * force,
          acceleration: Vector2(0, 320),
          child: CircleParticle(
            radius: 1.5 + rnd.nextDouble() * 2,
            paint: Paint()..color = couleur,
          ),
        );
      },
    );
  }

  // -------------------------------------------------------------- collisions

  @override
  void onCollision(
    Set<Vector2> intersectionPoints,
    PositionComponent other,
  ) {
    super.onCollision(intersectionPoints, other);

    if (etat == EtatEnnemi.mort) {
      return;
    }

    if (other is Plateforme) {
      _resoudreCollision(other);
      return;
    }

    if (other is Joueur) {
      final versJoueur = other.absoluteCenter - absoluteCenter;
      versJoueur.y = 0;
      if (versJoueur.length2 < 0.01) {
        versJoueur.setValues(sens.toDouble(), 0);
      }
      other.subirDegats(degatsContact, direction: versJoueur.normalized());
    }
  }

  void _resoudreCollision(Plateforme plateforme) {
    final moi = toAbsoluteRect();
    final lui = plateforme.toAbsoluteRect();
    final inter = moi.intersect(lui);
    if (inter.width <= 0 || inter.height <= 0) {
      return;
    }

    if (inter.width < inter.height) {
      position.x += moi.center.dx < lui.center.dx ? -inter.width : inter.width;
      velocite.x = 0;
    } else {
      if (moi.center.dy < lui.center.dy) {
        position.y -= inter.height;
        auSol = true;
      } else {
        position.y += inter.height;
      }
      velocite.y = 0;
    }
  }
}
```

### `lib/composants/gobelin.dart` (créé)

```dart
import 'package:flame/components.dart';

import '../config/constantes.dart';
import '../config/palette.dart';
import 'ennemi.dart';

class Gobelin extends Ennemi {
  Gobelin({required Vector2 position})
      : super(
          position: position,
          taille: Vector2(24, 30),
          vitesse: Constantes.vitesseGobelin,
          degatsContact: Constantes.degatsContactGobelin,
          pointsScore: Constantes.scoreGobelin,
          couleur: Palette.gobelin,
          pvDepart: Constantes.pvGobelin,
        );

  double _tempsAttaque = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    rayonAggro = Constantes.rayonAggroGobelin;
    rayonAbandon = Constantes.rayonAbandonGobelin;
    rayonAttaque = Constantes.rayonAttaqueGobelin;
    demiPatrouille = 3 * Constantes.tailleTuile;
  }

  @override
  void mettreAJourIA(double dt) {
    switch (etat) {
      case EtatEnnemi.patrouille:
        _patrouiller(dt);
        if (voitLeJoueur()) {
          etat = EtatEnnemi.poursuite;
        }
        break;
      case EtatEnnemi.poursuite:
        _poursuivre(dt);
        break;
      case EtatEnnemi.attaque:
        _attaquer(dt);
        break;
      case EtatEnnemi.retour:
        _revenir(dt);
        break;
      case EtatEnnemi.mort:
        velocite.x = 0;
        break;
    }
  }

  void _patrouiller(double dt) {
    velocite.x = sens * vitesse;

    final borneGauche = ancre.x - demiPatrouille;
    final borneDroite = ancre.x + demiPatrouille;

    final doitSeRetourner = (sens > 0 && position.x >= borneDroite) ||
        (sens < 0 && position.x <= borneGauche) ||
        murDevant() ||
        !solDevant();

    if (doitSeRetourner) {
      sens = -sens;
      velocite.x = sens * vitesse;
    }
  }

  void _poursuivre(double dt) {
    final ecartX = game.joueur.absoluteCenter.x - absoluteCenter.x;
    if (ecartX.abs() > 2) {
      sens = ecartX > 0 ? 1 : -1;
    }

    velocite.x = (solDevant() && !murDevant()) ? sens * vitesse : 0;

    final distance = distanceAuJoueur;
    if (distance <= rayonAttaque) {
      etat = EtatEnnemi.attaque;
      _tempsAttaque = Constantes.dureeAttaqueEnnemi;
    } else if (distance > rayonAbandon || !voitLeJoueur()) {
      etat = EtatEnnemi.retour;
    }
  }

  void _attaquer(double dt) {
    velocite.x = 0;
    _tempsAttaque -= dt;
    if (_tempsAttaque <= 0) {
      etat = EtatEnnemi.poursuite;
    }
  }

  void _revenir(double dt) {
    final ecartX = ancre.x - position.x;

    if (ecartX.abs() < 4) {
      velocite.x = 0;
      etat = EtatEnnemi.patrouille;
      return;
    }

    sens = ecartX > 0 ? 1 : -1;
    velocite.x = (solDevant() && !murDevant()) ? sens * vitesse * 0.8 : 0;

    if (voitLeJoueur()) {
      etat = EtatEnnemi.poursuite;
    }
  }
}
```

### `lib/composants/chauvesouris.dart` (créé)

```dart
import 'dart:math';

import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/geometry.dart';

import '../config/constantes.dart';
import '../config/palette.dart';
import 'ennemi.dart';
import 'projectile.dart';

class Chauvesouris extends Ennemi {
  Chauvesouris({required Vector2 position})
      : super(
          position: position,
          taille: Vector2(22, 16),
          vitesse: Constantes.vitesseChauvesouris,
          degatsContact: Constantes.degatsContactChauvesouris,
          pointsScore: Constantes.scoreChauvesouris,
          couleur: Palette.chauvesouris,
          pvDepart: Constantes.pvChauvesouris,
        );

  @override
  bool get subitGravite => false;

  double _phase = 0;
  double _cooldownTir = 0;
  double _tempsPique = 0;

  double amplitude = 18;
  double frequence = 2.4;

  @override
  ShapeHitbox creerHitbox() => CircleHitbox();

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    rayonAggro = Constantes.rayonAggroChauvesouris;
    rayonAbandon = Constantes.rayonAbandonChauvesouris;
    rayonAttaque = Constantes.rayonAttaqueChauvesouris;
    demiPatrouille = 4 * Constantes.tailleTuile;

    // Une phase de départ différente pour chaque individu.
    _phase = Random().nextDouble() * tau;
  }

  @override
  void mettreAJourIA(double dt) {
    _phase += frequence * dt;
    if (_cooldownTir > 0) {
      _cooldownTir -= dt;
    }

    switch (etat) {
      case EtatEnnemi.patrouille:
        _voler(dt);
        if (voitLeJoueur()) {
          etat = EtatEnnemi.poursuite;
        }
        break;
      case EtatEnnemi.poursuite:
        _poursuivreEnVol(dt);
        break;
      case EtatEnnemi.attaque:
        _piquer(dt);
        break;
      case EtatEnnemi.retour:
        _remonter(dt);
        break;
      case EtatEnnemi.mort:
        velocite.setZero();
        break;
    }
  }

  void _ondulation() {
    final cibleY = ancre.y + sin(_phase) * amplitude;
    velocite.y =
        cos(_phase) * amplitude * frequence + (cibleY - position.y) * 4.0;
  }

  void _voler(double dt) {
    velocite.x = sens * vitesse;

    final borneGauche = ancre.x - demiPatrouille;
    final borneDroite = ancre.x + demiPatrouille;

    if ((sens > 0 && position.x >= borneDroite) ||
        (sens < 0 && position.x <= borneGauche) ||
        murDevant()) {
      sens = -sens;
      velocite.x = sens * vitesse;
    }

    _ondulation();
  }

  void _poursuivreEnVol(double dt) {
    final vers = game.joueur.absoluteCenter - absoluteCenter;
    final distance = vers.length;

    if (distance > 1) {
      velocite = vers.normalized() * (vitesse * 0.55);
      sens = vers.x >= 0 ? 1 : -1;
    }
    _ondulation();

    if (distance <= rayonAttaque && _cooldownTir <= 0) {
      _declencherPique();
      _cooldownTir = Constantes.cooldownTirChauvesouris;
      return;
    }

    if (_cooldownTir <= 0 && voitLeJoueur()) {
      tirer();
    }

    if (distance > rayonAbandon || !voitLeJoueur()) {
      etat = EtatEnnemi.retour;
    }
  }

  void _declencherPique() {
    final vers = game.joueur.absoluteCenter - absoluteCenter;
    if (vers.length < 1) {
      return;
    }
    final direction = vers.normalized();

    velocite = direction * (vitesse * 2.4);
    sens = direction.x >= 0 ? 1 : -1;

    _tempsPique = Constantes.dureePiqueChauvesouris;
    etat = EtatEnnemi.attaque;
  }

  void _piquer(double dt) {
    _tempsPique -= dt;
    if (_tempsPique <= 0 || murDevant()) {
      etat = EtatEnnemi.retour;
    }
    // Pendant le piqué, la vélocité fixée au déclenchement n'est pas retouchée.
  }

  void _remonter(double dt) {
    final vers = ancre - position;

    if (vers.length < 6) {
      velocite.setZero();
      etat = EtatEnnemi.patrouille;
      return;
    }

    velocite = vers.normalized() * (vitesse * 0.8);
    sens = vers.x >= 0 ? 1 : -1;

    if (voitLeJoueur() && distanceAuJoueur < rayonAggro * 0.7) {
      etat = EtatEnnemi.poursuite;
    }
  }

  void tirer() {
    final vers = game.joueur.absoluteCenter - absoluteCenter;
    if (vers.length < 1) {
      return;
    }

    game.monde.add(
      Projectile(
        position: absoluteCenter.clone(),
        direction: vers.normalized(),
      ),
    );

    _cooldownTir = Constantes.cooldownTirChauvesouris;
  }
}
```

### `lib/composants/projectile.dart` (créé)

```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flutter/material.dart';

import '../config/constantes.dart';
import '../config/palette.dart';
import '../donjon_game.dart';
import 'joueur.dart';
import 'plateforme.dart';

class Projectile extends CircleComponent
    with HasGameReference<DonjonGame>, CollisionCallbacks {
  Projectile({
    required Vector2 position,
    required Vector2 direction,
    this.vitesse = Constantes.vitesseProjectile,
    this.degats = Constantes.degatsProjectile,
    this.dureeVie = Constantes.dureeVieProjectile,
  })  : direction = direction.normalized(),
        super(
          position: position,
          radius: 4,
          anchor: Anchor.center,
          priority: 5,
        );

  final Vector2 direction;
  final double vitesse;
  final double degats;
  final double dureeVie;

  double _tempsRestant = 0;

  @override
  Future<void> onLoad() async {
    _tempsRestant = dureeVie;
    paint = Paint()..color = Palette.projectile;
    add(CircleHitbox());
  }

  @override
  void update(double dt) {
    super.update(dt);

    _tempsRestant -= dt;
    if (_tempsRestant <= 0) {
      removeFromParent();
      return;
    }

    position += direction * (vitesse * dt);

    paint.color = Palette.projectile.withValues(
      alpha: (_tempsRestant / dureeVie).clamp(0.25, 1.0),
    );
  }

  @override
  void onCollisionStart(
    Set<Vector2> intersectionPoints,
    PositionComponent other,
  ) {
    super.onCollisionStart(intersectionPoints, other);

    if (other is Joueur) {
      other.subirDegats(degats, direction: direction.clone());
      removeFromParent();
    } else if (other is Plateforme) {
      removeFromParent();
    }
  }
}
```

### `lib/composants/joueur.dart` (modifié)

```dart
import 'dart:math';

import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

import '../config/constantes.dart';
import '../config/palette.dart';
import '../core/sante.dart';
import '../donjon_game.dart';
import 'ennemi.dart';
import 'plateforme.dart';

enum EtatJoueur { immobile, marche, saut, chute, attaque, touche, mort }

class Joueur extends PositionComponent
    with HasGameReference<DonjonGame>,
        Sante,
        KeyboardHandler,
        CollisionCallbacks {
  Joueur({required Vector2 position})
      : super(position: position, size: Vector2(24, 32), anchor: Anchor.topLeft);

  Vector2 velocite = Vector2.zero();
  EtatJoueur etat = EtatJoueur.immobile;
  bool auSol = false;
  bool invincible = false;
  int cles = 0;
  int sens = 1;

  double _entreeX = 0;
  double _tempsInvincible = 0;
  double _tempsAttaque = 0;
  double _cooldownAttaque = 0;
  double _tempsTouche = 0;
  double _tempsMort = 0;

  late final RectangleComponent corps;
  late final RectangleComponent oeil;
  late final ShapeHitbox hitbox;
  ZoneAttaque? _zone;

  @override
  Future<void> onLoad() async {
    initialiserSante(Constantes.pvJoueurMax);

    corps = RectangleComponent(
      size: size.clone(),
      position: size / 2,
      anchor: Anchor.center,
      paint: Paint()..color = Palette.joueur,
    );
    oeil = RectangleComponent(
      size: Vector2.all(4),
      position: Vector2(size.x - 7, 7),
      paint: Paint()..color = Palette.oeil,
    );
    corps.add(oeil);
    await add(corps);

    hitbox = RectangleHitbox();
    await add(hitbox);
  }

  // ------------------------------------------------------------------ entrée

  @override
  bool onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
    final gauche = keysPressed.contains(LogicalKeyboardKey.arrowLeft) ||
        keysPressed.contains(LogicalKeyboardKey.keyA) ||
        keysPressed.contains(LogicalKeyboardKey.keyQ);
    final droite = keysPressed.contains(LogicalKeyboardKey.arrowRight) ||
        keysPressed.contains(LogicalKeyboardKey.keyD);

    _entreeX = (droite ? 1.0 : 0.0) - (gauche ? 1.0 : 0.0);

    if (event is KeyDownEvent) {
      final touche = event.logicalKey;
      if (touche == LogicalKeyboardKey.space ||
          touche == LogicalKeyboardKey.arrowUp ||
          touche == LogicalKeyboardKey.keyW ||
          touche == LogicalKeyboardKey.keyZ) {
        sauter();
      }
      if (touche == LogicalKeyboardKey.keyE ||
          touche == LogicalKeyboardKey.keyJ) {
        attaquer();
      }
    }
    return true;
  }

  // ------------------------------------------------------------------ boucle

  @override
  void update(double dt) {
    super.update(dt);

    if (etat == EtatJoueur.mort) {
      _tempsMort -= dt;
      if (_tempsMort <= 0) {
        _tempsMort = double.infinity;   // ne déclencher qu'une seule fois
        game.perdreUneVie();
      }
      return;
    }

    if (_tempsInvincible > 0) {
      _tempsInvincible -= dt;
      if (_tempsInvincible <= 0) {
        invincible = false;
      }
    }
    if (_cooldownAttaque > 0) {
      _cooldownAttaque -= dt;
    }
    if (_tempsTouche > 0) {
      _tempsTouche -= dt;
    }
    if (_tempsAttaque > 0) {
      _tempsAttaque -= dt;
      if (_tempsAttaque <= 0) {
        _finirAttaque();
      }
    }

    if (_tempsTouche <= 0) {
      velocite.x = _entreeX * Constantes.vitesseJoueur;
      if (_entreeX != 0) {
        sens = _entreeX > 0 ? 1 : -1;
      }
    }

    velocite.y += Constantes.gravite * dt;
    if (velocite.y > Constantes.vitesseMaxChute) {
      velocite.y = Constantes.vitesseMaxChute;
    }

    auSol = false;
    position += velocite * dt;

    _majEtat();
    oeil.position.x = sens > 0 ? size.x - 7 : 3;
  }

  void _majEtat() {
    if (etat == EtatJoueur.attaque || etat == EtatJoueur.touche) {
      return;
    }
    if (!auSol && velocite.y < 0) {
      etat = EtatJoueur.saut;
    } else if (!auSol && velocite.y > 0) {
      etat = EtatJoueur.chute;
    } else if (velocite.x.abs() > 1) {
      etat = EtatJoueur.marche;
    } else {
      etat = EtatJoueur.immobile;
    }
  }

  // ------------------------------------------------------------------ action

  void sauter() {
    if (!auSol || etat == EtatJoueur.mort) {
      return;
    }
    velocite.y = Constantes.forceSaut;
    auSol = false;
    etat = EtatJoueur.saut;
  }

  void attaquer() {
    if (_cooldownAttaque > 0 || etat == EtatJoueur.mort) {
      return;
    }

    _cooldownAttaque = Constantes.cooldownAttaqueJoueur;
    _tempsAttaque = Constantes.dureeAttaqueJoueur;
    etat = EtatJoueur.attaque;

    _zone = ZoneAttaque(
      degats: Constantes.degatsAttaqueJoueur,
      position: Vector2(sens > 0 ? size.x : -18, 4),
      size: Vector2(18, 22),
    );
    add(_zone!);
  }

  void _finirAttaque() {
    _zone?.removeFromParent();
    _zone = null;
    if (etat == EtatJoueur.attaque) {
      etat = EtatJoueur.immobile;
    }
  }

  // ------------------------------------------------------------------ dégâts

  @override
  void subirDegats(double degats, {Vector2? direction}) {
    if (invincible || !estVivant || etat == EtatJoueur.mort) {
      return;
    }

    super.subirDegats(degats);

    if (!estVivant) {
      return;
    }

    invincible = true;
    _tempsInvincible = Constantes.dureeInvincibilite;
    etat = EtatJoueur.touche;
    _tempsTouche = 0.25;
    _reculer(direction);
    _clignoter();
  }

  void _reculer(Vector2? direction) {
    final horizontal = (direction == null || direction.x.abs() < 0.01)
        ? -sens.toDouble()
        : (direction.x > 0 ? 1.0 : -1.0);

    velocite.x = horizontal * Constantes.forceKnockback;
    velocite.y = -Constantes.forceKnockback * 0.5;
  }

  void _clignoter() {
    final repetitions =
        (Constantes.dureeInvincibilite / (0.075 * 2)).round();
    corps.add(
      OpacityEffect.to(
        0.25,
        EffectController(
          duration: 0.075,
          alternate: true,
          repeatCount: repetitions,
        ),
        onComplete: () => corps.opacity = 1.0,
      ),
    );
  }

  @override
  void onDegatsRecus(double degats) {
    corps.add(
      ColorEffect(
        const Color(0xFFFF5252),
        EffectController(duration: 0.07, alternate: true),
        opacityTo: 1.0,
      ),
    );
  }

  @override
  void onMort() => mourir();

  void mourir() {
    if (etat == EtatJoueur.mort) {
      return;
    }
    etat = EtatJoueur.mort;
    velocite.setZero();
    hitbox.collisionType = CollisionType.inactive;
    _finirAttaque();
    _tempsMort = 0.9;
    corps.add(RotateEffect.by(pi / 2, EffectController(duration: 0.35)));
  }

  void reapparaitre(Vector2 point) {
    position = point.clone();
    velocite.setZero();

    pv = pvMax;
    etat = EtatJoueur.immobile;
    auSol = false;
    sens = 1;

    corps.angle = 0;
    corps.opacity = 1.0;
    corps.scale = Vector2.all(1);
    hitbox.collisionType = CollisionType.active;

    _tempsMort = 0;
    _tempsTouche = 0;
    _cooldownAttaque = 0;

    invincible = true;
    _tempsInvincible = Constantes.dureeInvincibilite;
    _clignoter();
  }

  // -------------------------------------------------------------- collisions

  @override
  void onCollision(
    Set<Vector2> intersectionPoints,
    PositionComponent other,
  ) {
    super.onCollision(intersectionPoints, other);
    if (other is Plateforme) {
      _resoudreCollision(other);
    }
  }

  void _resoudreCollision(Plateforme plateforme) {
    final moi = toAbsoluteRect();
    final lui = plateforme.toAbsoluteRect();
    final inter = moi.intersect(lui);
    if (inter.width <= 0 || inter.height <= 0) {
      return;
    }

    if (inter.width < inter.height) {
      position.x += moi.center.dx < lui.center.dx ? -inter.width : inter.width;
      velocite.x = 0;
    } else {
      if (moi.center.dy < lui.center.dy) {
        position.y -= inter.height;
        auSol = true;
      } else {
        position.y += inter.height;
      }
      velocite.y = 0;
    }
  }
}

/// Hitbox temporaire créée devant le joueur pendant l'attaque.
class ZoneAttaque extends PositionComponent with CollisionCallbacks {
  ZoneAttaque({
    required this.degats,
    required Vector2 position,
    required Vector2 size,
  }) : super(position: position, size: size, anchor: Anchor.topLeft);

  final double degats;
  final Set<Ennemi> _dejaTouches = {};

  @override
  Future<void> onLoad() async {
    add(RectangleHitbox());
    add(
      RectangleComponent(
        size: size.clone(),
        paint: Paint()..color = Palette.attaque.withValues(alpha: 0.35),
      ),
    );
  }

  @override
  void onCollisionStart(
    Set<Vector2> intersectionPoints,
    PositionComponent other,
  ) {
    super.onCollisionStart(intersectionPoints, other);

    if (other is! Ennemi || !other.estVivant) {
      return;
    }
    if (!_dejaTouches.add(other)) {
      return;
    }

    other.subirDegats(degats);

    if (other.estVivant) {
      final versEnnemi = other.absoluteCenter.x - absoluteCenter.x;
      other.appliquerKnockback(Vector2(versEnnemi >= 0 ? 1 : -1, 0));
    }
  }
}
```

### `lib/donjon_game.dart` (modifié)

```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flame/input.dart';
import 'package:flutter/material.dart';

import 'composants/chauvesouris.dart';
import 'composants/gobelin.dart';
import 'composants/joueur.dart';
import 'composants/plateforme.dart';
import 'config/constantes.dart';
import 'config/palette.dart';
import 'core/game_state.dart';

class Overlays {
  static const String menuPrincipal = 'menu_principal';
  static const String hud = 'hud';
  static const String pause = 'pause';
  static const String gameOver = 'game_over';
  static const String victoire = 'victoire';
  static const String chargement = 'chargement';
}

class DonjonGame extends FlameGame
    with HasCollisionDetection, HasKeyboardHandlerComponents {
  /// Alias du `world` fourni par `FlameGame`.
  late final World monde = world;

  GameState etat = GameState.chargement;

  int score = 0;
  int vies = Constantes.viesDepart;
  int niveauCourant = 0;
  int meilleurScore = 0;

  late Joueur joueur;
  Vector2 pointDeReapparition = Vector2(48, 150);

  @override
  Color backgroundColor() => Palette.fond;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.zoom = Constantes.zoomCamera;
    changerEtat(GameState.menu);
  }

  // ------------------------------------------------------------------- états

  void changerEtat(GameState nouvelEtat) {
    etat = nouvelEtat;
    overlays.clear();

    switch (nouvelEtat) {
      case GameState.chargement:
        overlays.add(Overlays.chargement);
        break;
      case GameState.menu:
        pauseEngine();
        overlays.add(Overlays.menuPrincipal);
        break;
      case GameState.enJeu:
        resumeEngine();
        break;
      case GameState.pause:
        pauseEngine();
        overlays.add(Overlays.pause);
        break;
      case GameState.gameOver:
        pauseEngine();
        overlays.add(Overlays.gameOver);
        break;
      case GameState.victoire:
        pauseEngine();
        overlays.add(Overlays.victoire);
        break;
    }
  }

  // ------------------------------------------------------------------ partie

  Future<void> demarrerPartie() async {
    monde.removeAll(monde.children.toList());

    score = 0;
    vies = Constantes.viesDepart;
    niveauCourant = 0;

    await _construireSalle();
    changerEtat(GameState.enJeu);
  }

  Future<void> _construireSalle() async {
    const t = Constantes.tailleTuile;

    // Sol, murs et plateformes. Le chapitre 39 remplacera ce bloc par une carte.
    await monde.addAll([
      Plateforme(position: Vector2(0, 6 * t), size: Vector2(16 * t, t)),
      Plateforme(position: Vector2(0, 0), size: Vector2(t, 6 * t)),
      Plateforme(position: Vector2(15 * t, 0), size: Vector2(t, 6 * t)),
      Plateforme(position: Vector2(3 * t, 4 * t), size: Vector2(3 * t, 12)),
      Plateforme(position: Vector2(9 * t, 3 * t), size: Vector2(4 * t, 12)),
      Plateforme(position: Vector2(7 * t, 5 * t), size: Vector2(t, t)),
    ]);

    // Le joueur EN PREMIER : les ennemis lisent `game.joueur` dès leur montage.
    pointDeReapparition = Vector2(1.5 * t, 5 * t + 2);
    joueur = Joueur(position: pointDeReapparition.clone());
    await monde.add(joueur);

    // Les ennemis.
    await monde.addAll([
      Gobelin(position: Vector2(6 * t, 5 * t + 2)),
      Gobelin(position: Vector2(12 * t, 5 * t + 2)),
      Chauvesouris(position: Vector2(5 * t, 2 * t)),
      Chauvesouris(position: Vector2(11 * t, 1.5 * t)),
    ]);

    camera.follow(joueur);
  }

  // ------------------------------------------------------------- score, vies

  void ajouterScore(int points) {
    score += points;
    if (score > meilleurScore) {
      meilleurScore = score;
    }
  }

  void perdreUneVie() {
    vies--;

    if (vies <= 0) {
      vies = 0;
      changerEtat(GameState.gameOver);
      return;
    }

    joueur.reapparaitre(pointDeReapparition);
  }
}
```

### `lib/main.dart` (modifié)

```dart
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

import 'config/palette.dart';
import 'donjon_game.dart';
import 'ecrans/menu_principal.dart';

void main() {
  runApp(const DonjonApp());
}

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    final jeu = DonjonGame();

    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: GameWidget<DonjonGame>(
          game: jeu,
          overlayBuilderMap: {
            Overlays.menuPrincipal: (context, game) => MenuPrincipal(game: game),
            Overlays.chargement: (context, game) =>
                const Center(child: CircularProgressIndicator()),
            // Provisoire : le chapitre 40 fournira le vrai écran.
            Overlays.gameOver: (context, game) => _GameOverProvisoire(game: game),
          },
        ),
      ),
    );
  }
}

class _GameOverProvisoire extends StatelessWidget {
  const _GameOverProvisoire({required this.game});

  final DonjonGame game;

  @override
  Widget build(BuildContext context) {
    return Container(
      color: Colors.black.withValues(alpha: 0.7),
      child: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            const Text(
              'GAME OVER',
              style: TextStyle(
                color: Palette.texte,
                fontSize: 40,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 8),
            Text(
              'Score : ${game.score}',
              style: const TextStyle(color: Palette.texte, fontSize: 20),
            ),
            const SizedBox(height: 20),
            ElevatedButton(
              onPressed: game.demarrerPartie,
              child: const Text('Rejouer'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Rappel — les fichiers du chapitre 35-36 non modifiés

Pour que le projet compile, ces trois fichiers doivent être présents tels que les chapitres 35 et 36 les ont laissés.

```dart
// lib/core/game_state.dart
enum GameState { chargement, menu, enJeu, pause, gameOver, victoire }
```

```dart
// lib/ecrans/menu_principal.dart
import 'package:flutter/material.dart';

import '../config/palette.dart';
import '../donjon_game.dart';

class MenuPrincipal extends StatelessWidget {
  const MenuPrincipal({super.key, required this.game});

  final DonjonGame game;

  @override
  Widget build(BuildContext context) {
    return Container(
      color: Palette.fond,
      child: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            const Text(
              'DONJON DE DART',
              style: TextStyle(
                color: Palette.accent,
                fontSize: 34,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: game.demarrerPartie,
              child: const Text('Jouer'),
            ),
          ],
        ),
      ),
    );
  }
}
```

`lib/composants/plateforme.dart` figure en section 37.1. `lib/core/entite.dart` n'est pas utilisé par ce chapitre.

**Commandes de vérification :**

```text
flutter pub get
dart analyze
flutter run -d chrome
```

**Contrôles clavier :**

```text
Q / A / flèche gauche    aller à gauche
D / flèche droite        aller à droite
Z / W / espace / haut    sauter
E / J                    attaquer
```

---

## 37.35 — Exercices

### Exercice 1 — Le soin (facile)
Ajoutez au mixin `Sante` une méthode `soignerTout()` qui remet `pv` à `pvMax`, même si l'entité est morte, et qui renvoie le nombre de points effectivement rendus.

### Exercice 2 — Le rat des cavernes (facile)
Créez `lib/composants/rat.dart` : un ennemi terrestre de 16 × 12 pixels, 10 PV, 5 dégâts de contact, vitesse 110, 15 points de score, qui **patrouille sans jamais poursuivre** le joueur.

### Exercice 3 — Le gobelin qui saute au bord (moyen)
Modifiez `Gobelin` pour qu'en état `poursuite`, au lieu de s'arrêter quand `solDevant()` est faux, il saute (`velocite.y = Constantes.forceSaut * 0.8`) s'il est au sol et si le joueur est plus bas que lui.

### Exercice 4 — Le champ de vision (moyen)
Ajoutez à `Ennemi` un champ `bool vueDeDos = false`. Quand il vaut `false`, `voitLeJoueur()` doit renvoyer `false` si le joueur se trouve derrière l'ennemi (du côté opposé à `sens`).

### Exercice 5 — La barre de vie flottante (moyen)
Ajoutez à `Ennemi` une petite barre de vie de 20 × 3 pixels au-dessus de sa tête, visible uniquement lorsque `pv < pvMax`, dont la largeur suit `ratioPv`.

### Exercice 6 — Le gobelin lanceur (moyen)
Créez `GobelinLanceur`, sous-classe de `Gobelin`, qui garde ses distances : en `poursuite`, il s'arrête à 90 pixels du joueur et lance un `Projectile` toutes les 2 secondes.

### Exercice 7 — Le projectile rebondissant (difficile)
Ajoutez au `Projectile` un champ `int rebonds = 0`. Au contact d'une plateforme, s'il lui reste des rebonds, il inverse sa direction en utilisant `normal` du résultat de collision au lieu de disparaître.

### Exercice 8 — Le saut sur la tête (difficile)
Implémentez la mécanique « Mario » décrite en 37.24 : si le joueur touche l'ennemi en tombant et par le dessus, l'ennemi meurt et le joueur rebondit.

### Exercice 9 — L'alerte de groupe (difficile)
Quand un ennemi passe en `poursuite`, tous les ennemis situés à moins de 120 pixels de lui passent également en `poursuite`, à condition qu'ils soient en `patrouille`.

### Exercice 10 — Le multiplicateur de difficulté (difficile)
Ajoutez à `DonjonGame` un champ `double difficulte = 1.0` et faites en sorte que tout ennemi créé multiplie ses PV par `difficulte` et ses dégâts de contact par `(1 + (difficulte - 1) * 0.5)`. Vérifiez avec `difficulte = 1.5`.

---

## 37.36 — Corrections des exercices

### Correction 1

```dart
// Dans lib/core/sante.dart
double soignerTout() {
  final rendus = pvMax - pv;
  pv = pvMax;
  if (rendus > 0) {
    onSoinRecu(rendus);
  }
  return rendus;
}
```

**Explication :** on calcule l'écart **avant** de modifier `pv`, sinon le résultat vaudrait toujours zéro. Contrairement à `soigner`, la méthode ne teste pas `estVivant` : c'est justement son rôle, ressusciter. Le `if (rendus > 0)` évite de notifier un soin de zéro point, ce qui déclencherait un effet visuel inutile.

### Correction 2

```dart
// lib/composants/rat.dart
import 'package:flame/components.dart';

import '../config/constantes.dart';
import 'ennemi.dart';

class Rat extends Ennemi {
  Rat({required Vector2 position})
      : super(
          position: position,
          taille: Vector2(16, 12),
          vitesse: 110,
          degatsContact: 5,
          pointsScore: 15,
          couleur: const Color(0xFF8D6E63),
          pvDepart: 10,
        );

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    demiPatrouille = 5 * Constantes.tailleTuile;
  }

  @override
  void mettreAJourIA(double dt) {
    if (etat != EtatEnnemi.patrouille) {
      return;
    }
    velocite.x = sens * vitesse;

    final trop = (sens > 0 && position.x >= ancre.x + demiPatrouille) ||
        (sens < 0 && position.x <= ancre.x - demiPatrouille);

    if (trop || murDevant() || !solDevant()) {
      sens = -sens;
      velocite.x = sens * vitesse;
    }
  }
}
```

**Explication :** un ennemi entier en trente lignes, parce que la classe abstraite fournit déjà la santé, le corps, la hitbox, la gravité, la résolution de collision, le knockback, le flash et la mort. `mettreAJourIA` n'implémente qu'un seul état : le rat n'appelle jamais `voitLeJoueur()`, il ne peut donc jamais quitter `patrouille`. Il reste dangereux au contact, puisque les dégâts sont gérés par `Ennemi.onCollision`. N'oubliez pas `import 'package:flutter/material.dart';` pour `Color`.

### Correction 3

```dart
// Dans Gobelin._poursuivre, en remplacement de la ligne velocite.x = ...
final joueurPlusBas =
    game.joueur.absoluteCenter.y > absoluteCenter.y + size.y * 0.5;

if (murDevant()) {
  velocite.x = 0;
} else if (solDevant()) {
  velocite.x = sens * vitesse;
} else if (auSol && joueurPlusBas) {
  velocite.x = sens * vitesse;
  velocite.y = Constantes.forceSaut * 0.8;   // il saute dans le vide
} else {
  velocite.x = 0;
}
```

**Explication :** l'ordre des tests est le point délicat. Un mur bloque toujours ; sinon, s'il y a du sol on avance normalement ; sinon, on n'ose le saut que si le joueur est **plus bas** — sauter vers le haut dans le vide n'aurait aucun sens. La condition `auSol` empêche le double saut : sans elle, le gobelin s'envolerait en enchaînant les impulsions à chaque frame passée en l'air.

### Correction 4

```dart
// Dans Ennemi
bool vueDeDos = false;

bool voitLeJoueur() {
  final cible = game.joueur;
  if (!cible.isMounted || !cible.estVivant) {
    return false;
  }

  final vers = cible.absoluteCenter - absoluteCenter;

  if (!vueDeDos && vers.x * sens < -4) {
    return false;                 // le joueur est derrière : on ne voit rien
  }

  final distance = vers.length;
  // ... la suite est inchangée
}
```

**Explication :** `vers.x * sens` est un produit scalaire à une dimension (chapitre 23.9). Il est positif quand le joueur est devant, négatif quand il est derrière. La marge de `-4` pixels évite une bascule nerveuse quand le joueur est exactement à la verticale de l'ennemi. Le test est placé **avant** le raycast : c'est encore l'ordre « du moins cher au plus cher » de la section 37.10.

### Correction 5

```dart
// Dans Ennemi
late final RectangleComponent _jaugeFond;
late final RectangleComponent _jauge;

// à la fin de onLoad()
_jaugeFond = RectangleComponent(
  size: Vector2(20, 3),
  position: Vector2(size.x / 2 - 10, -7),
  paint: Paint()..color = const Color(0xAA000000),
);
_jauge = RectangleComponent(
  size: Vector2(20, 3),
  paint: Paint()..color = const Color(0xFFE53935),
);
_jaugeFond.add(_jauge);
await add(_jaugeFond);
_jaugeFond.opacity = 0;

// à la fin de update(dt)
final blesse = pv < pvMax && estVivant;
_jaugeFond.opacity = blesse ? 1 : 0;
if (blesse) {
  _jauge.size.x = 20 * ratioPv;
}
```

**Explication :** la jauge est ajoutée à l'ennemi (et non à `corps`) pour qu'elle ne subisse ni le flash blanc ni l'écrasement. `-7` en Y la place au-dessus de la tête, puisque les coordonnées d'un enfant sont relatives au coin supérieur gauche du parent. On ne modifie que `size.x` : la barre se vide donc **vers la droite**, ce qui est la convention attendue. Le chapitre 38 généralisera ce composant sous le nom `BarreDeVie`.

### Correction 6

```dart
// lib/composants/gobelin_lanceur.dart
class GobelinLanceur extends Gobelin {
  GobelinLanceur({required super.position});

  double _cooldown = 0;
  static const double _distanceIdeale = 90;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    rayonAggro = 220;
    rayonAbandon = 300;
  }

  @override
  void mettreAJourIA(double dt) {
    if (_cooldown > 0) {
      _cooldown -= dt;
    }

    if (etat != EtatEnnemi.poursuite) {
      super.mettreAJourIA(dt);
      return;
    }

    final vers = game.joueur.absoluteCenter - absoluteCenter;
    sens = vers.x >= 0 ? 1 : -1;

    // Il garde ses distances : il avance, recule ou reste immobile.
    if (vers.length > _distanceIdeale + 20 && solDevant() && !murDevant()) {
      velocite.x = sens * vitesse;
    } else if (vers.length < _distanceIdeale - 20) {
      velocite.x = -sens * vitesse * 0.7;
    } else {
      velocite.x = 0;
    }

    if (_cooldown <= 0 && voitLeJoueur()) {
      game.monde.add(
        Projectile(
          position: absoluteCenter.clone(),
          direction: vers.normalized(),
        ),
      );
      _cooldown = 2.0;
    }

    if (vers.length > rayonAbandon || !voitLeJoueur()) {
      etat = EtatEnnemi.retour;
    }
  }
}
```

**Explication :** la sous-classe ne redéfinit que l'état `poursuite` et délègue tous les autres à `super.mettreAJourIA(dt)` : c'est la façon la plus économique d'ajouter un comportement. La zone morte de 40 pixels (`± 20` autour de la distance idéale) est indispensable : sans elle, le gobelin avancerait et reculerait alternativement à chaque frame. Le cooldown est décrémenté **hors** du `switch`, sinon il se figerait dès que l'ennemi change d'état.

### Correction 7

```dart
// Dans Projectile
int rebonds = 0;

@override
void onCollisionStart(
  Set<Vector2> intersectionPoints,
  PositionComponent other,
) {
  super.onCollisionStart(intersectionPoints, other);

  if (other is Joueur) {
    other.subirDegats(degats, direction: direction.clone());
    removeFromParent();
    return;
  }

  if (other is Plateforme) {
    if (rebonds <= 0) {
      removeFromParent();
      return;
    }
    rebonds--;

    // Réflexion : d' = d - 2 (d . n) n
    final n = _normaleApprochee(other);
    final produit = direction.dot(n);
    direction.setFrom(direction - n * (2 * produit));

    // On s'écarte du mur pour ne pas re-collisionner immédiatement.
    position += direction * 3;
  }
}

Vector2 _normaleApprochee(PositionComponent mur) {
  final r = mur.toAbsoluteRect();
  final ecart = absoluteCenter - Vector2(r.center.dx, r.center.dy);
  return ecart.x.abs() * r.height > ecart.y.abs() * r.width
      ? Vector2(ecart.x.sign, 0)
      : Vector2(0, ecart.y.sign);
}
```

**Explication :** la formule de réflexion d'un vecteur sur une surface de normale unitaire `n` est `d' = d - 2·(d·n)·n` — le produit scalaire du chapitre 23.9. La normale est ici déduite de la position relative des centres, ce qui suffit pour des rectangles alignés sur les axes. Le décalage de 3 pixels après le rebond est obligatoire : sans lui, le projectile reste dans la hitbox du mur et rebondit à nouveau à la frame suivante, ce qui le fait vibrer sur place. Notez que `RaycastResult.normal` existe mais n'est pas disponible dans un callback de collision : `onCollisionStart` ne fournit que des points d'intersection.

### Correction 8

```dart
// Dans Ennemi.onCollision, avant l'application des dégâts de contact
if (other is Joueur) {
  final vientDuDessus =
      other.absoluteCenter.y < absoluteCenter.y - size.y * 0.25;
  final tombe = other.velocite.y > 0;

  if (vientDuDessus && tombe && !other.invincible) {
    subirDegats(pvMax);                                // l'ennemi meurt
    other.velocite.y = Constantes.forceSaut * 0.7;     // le joueur rebondit
    return;                                            // pas de dégâts au joueur
  }

  // ... l'application normale des dégâts de contact
}
```

**Explication :** deux conditions et pas une seule. `vientDuDessus` seule ne suffit pas : un joueur qui monte le long du flanc de l'ennemi remplirait la condition et le tuerait injustement. La combinaison avec `velocite.y > 0` (il descend) garantit le vrai saut sur la tête. Le `return` est capital : sans lui, l'ennemi meurt **et** le joueur perd des points de vie dans la même frame. Le rebond à 70 % de la force de saut est le réglage classique : plus haut que rien, plus bas qu'un vrai saut, pour ne pas donner un moyen d'ascension gratuit.

### Correction 9

```dart
// Dans Ennemi
void alerterVoisins() {
  for (final autre in game.monde.children.whereType<Ennemi>()) {
    if (identical(autre, this) || autre.etat != EtatEnnemi.patrouille) {
      continue;
    }
    if ((autre.absoluteCenter - absoluteCenter).length <= 120) {
      autre.etat = EtatEnnemi.poursuite;
    }
  }
}

// Dans le setter de `etat`
set etat(EtatEnnemi nouveau) {
  if (nouveau == _etat) {
    return;
  }
  final ancien = _etat;
  _etat = nouveau;
  if (nouveau == EtatEnnemi.poursuite && ancien == EtatEnnemi.patrouille) {
    alerterVoisins();
  }
}
```

**Explication :** le point délicat est la récursion. `alerterVoisins` met des voisins en `poursuite`, ce qui déclenche à nouveau leur propre setter, donc leur propre alerte. La condition `autre.etat != EtatEnnemi.patrouille` sert de garde d'arrêt : un ennemi déjà en poursuite n'est jamais réalerté, la propagation s'éteint donc après avoir couvert la grappe. `_etat = nouveau;` est affecté **avant** l'appel, sinon l'ennemi s'alerterait lui-même en boucle. `identical` compare les références, ce qui est exactement ce que l'on veut ici.

### Correction 10

```dart
// Dans DonjonGame
double difficulte = 1.0;

// Dans Ennemi.onLoad, en remplacement de initialiserSante(_pvDepart)
initialiserSante(_pvDepart * game.difficulte);
degatsContact *= 1 + (game.difficulte - 1) * 0.5;
```

**Explication :** `onLoad` est le seul endroit correct : `game` n'est pas fiable dans le constructeur (rappel de la fiche Flame), et `initialiserSante` doit rester la première instruction utile. Le fait que `degatsContact` ne soit pas `final` autorise la multiplication sur place. La formule dissymétrique — PV × 1,5 mais dégâts × 1,25 — est un choix d'équilibrage délibéré (section 37.30) : augmenter la résistance allonge les combats, augmenter les dégâts raccourcit la vie du joueur ; la seconde punit beaucoup plus vite que la première, on la fait donc croître deux fois moins.

**Vérification à `difficulte = 1.5` :**

```text
Gobelin      : 30 x 1.5 = 45 PV   -> 2 coups d'épée (45 / 25 = 1.8)
               12 x 1.25 = 15 dégâts -> 7 fautes tolérées au lieu de 9
Chauve-souris: 15 x 1.5 = 22.5 PV -> 1 coup d'épée
```

---

## Et maintenant ?

Le donjon est enfin dangereux : les gobelins patrouillent et vous poursuivent, les chauves-souris ondulent et vous tirent dessus, votre épée frappe, les ennemis meurent, vous perdez des vies et vous réapparaissez. Le jeu est complet sur le plan des règles.

Il lui manque pourtant une chose élémentaire : **le joueur ne voit rien de tout cela**. Ses points de vie, ses trois vies, son score qui grimpe de 50 en 50 n'existent que dans la mémoire du programme. Un joueur qui ne sait pas combien il lui reste de PV ne peut pas prendre de décision.

Le chapitre 38 corrige cela et ajoute les récompenses : les pièces à ramasser, les potions qui rendent des points de vie grâce à `soigner()` écrit aujourd'hui, les clés qui ouvriront les portes du chapitre 39, et surtout le HUD — barre de vie, barre d'énergie, compteur de score, cœurs de vies — construit avec les composants du viewport et les overlays Flutter.

Rendez-vous au chapitre 38 : [38-PARTIE-2C—OBJETS-COLLECTIBLES-SCORE-ET-HUD.md](./38-PARTIE-2C—OBJETS-COLLECTIBLES-SCORE-ET-HUD.md)
