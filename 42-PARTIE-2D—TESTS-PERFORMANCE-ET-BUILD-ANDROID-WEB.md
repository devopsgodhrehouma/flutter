# PARTIE 2D — MÉTHODE, QUALITÉ ET LIVRAISON
# CHAPITRE 42 — TESTS, PERFORMANCE ET PUBLICATION

> **Niveau :** intermédiaire / avancé
> **Durée estimée :** 12 h
> **Pré-requis :** chapitres 35 à 40 (le jeu « Donjon de Dart » complet), chapitre 41 (cahier des charges et GDD), chapitre 16 (projet Dart, `pubspec.yaml`, `dart test`), chapitre 20 (boucle de jeu, FPS, delta time)
> **Version de Flutter utilisée :** **Flutter 3.44 / Dart 3.12**
> **Version de Flame utilisée :** **Flame 1.38.0**, **flame_test 2.2.4**
> **Commandes et procédures vérifiées le :** **8 août 2026** sur `docs.flutter.dev` (`/testing/overview`, `/perf`, `/perf/ui-performance`, `/tools/devtools/performance`, `/deployment/android`, `/deployment/web`, `/deployment/windows`, `/platform-integration/web/renderers`), sur `docs.flame-engine.org/latest/flame/other/debug.html` et sur `pub.dev/packages/flame_test`.
> **Ce que vous saurez faire à la fin :** écrire une suite de tests automatiques pour un jeu Flame, mesurer et corriger une chute de FPS avec le DevTools, puis produire et publier un APK Android signé et une version Web hébergée.

---

## 42.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- distinguer ce qui se teste automatiquement dans un jeu de ce qui ne se teste qu'à la main ;
- installer et configurer `flutter_test` et `flame_test` dans le `pubspec.yaml` ;
- écrire un test unitaire sur une fonction de logique pure (dégâts, score, économie) ;
- écrire un test de composant Flame avec `testWithGame` et `FlameTester` ;
- simuler le passage du temps en appelant `game.update(dt)` dans un test ;
- écrire un test qui vérifie qu'une collision déclenche bien `onCollisionStart` ;
- écrire un test de widget sur un menu Flutter avec `pumpWidget` et les *finders* ;
- expliquer ce qu'apporte un golden test et pourquoi il est fragile pour un jeu ;
- organiser un dossier `test/` qui reflète l'arborescence de `lib/` ;
- lancer `flutter test`, lire son rapport, et produire une couverture avec `--coverage` ;
- écrire un workflow GitHub Actions minimal qui analyse, teste et compile le projet ;
- activer le `debugMode` de Flame et lire les informations qu'il affiche ;
- afficher les hitboxes et un compteur de FPS avec `FpsTextComponent` ;
- remplacer les `print` sauvages par un logger discipliné ;
- poser un point d'arrêt conditionnel dans VS Code et inspecter l'état d'un composant ;
- refuser d'optimiser avant d'avoir mesuré, et savoir avec quoi mesurer ;
- lancer le jeu en mode profil, ouvrir le DevTools et lire la vue *Performance* ;
- interpréter le *Flutter Frames Chart*, distinguer le fil UI du fil raster ;
- expliquer le budget de 16 ms et ce qui se passe quand on le dépasse ;
- identifier les cinq causes classiques de chute de FPS dans un jeu Flame ;
- implémenter un *culling* qui n'affiche pas ce que la caméra ne voit pas ;
- implémenter une *pool* d'objets pour les projectiles et les particules ;
- traquer les allocations de `Vector2` dans `update()` et les supprimer ;
- réduire le coût des images et comprendre l'intérêt d'un atlas ;
- comprendre pourquoi un jeu qui rame en debug peut être parfaitement fluide en release ;
- limiter la consommation de batterie d'un jeu mobile ;
- dérouler une liste de vérification complète avant publication ;
- changer l'icône, le nom et l'écran de démarrage de l'application ;
- produire un APK et un App Bundle avec `flutter build apk` et `flutter build appbundle` ;
- créer un keystore avec `keytool` et configurer la signature dans Gradle ;
- modifier `applicationId`, `versionCode` et `versionName` ;
- décrire les étapes de publication sur le Google Play Store ;
- produire une version Web avec `flutter build web` et choisir son moteur de rendu ;
- héberger le résultat sur GitHub Pages ou sur itch.io, en réglant `--base-href` ;
- produire un exécutable Windows, macOS ou Linux ;
- versionner proprement vos mises à jour ;
- présenter votre projet devant un jury et comprendre la grille qui vous évalue ;
- situer chacun des 42 chapitres de cette formation dans un tableau d'ensemble ;
- savoir où continuer d'apprendre après ce chapitre.

---

## 42.1 — Que teste-t-on dans un jeu, et que ne teste-t-on pas

Il faut commencer par une vérité que peu de tutoriels assument : **on ne teste pas un jeu comme on teste une application de gestion.**

Une application bancaire a un comportement déterministe et vérifiable. « Si je vire 100 euros du compte A vers le compte B, le compte A perd 100 euros. » Cela s'écrit en trois lignes de test et cela se vérifie en une milliseconde.

Un jeu, lui, contient une part énorme de **ressenti**. « Le saut est agréable. » « Le boss est difficile mais juste. » « L'écran de victoire donne envie de recommencer. » Aucune de ces phrases n'est un test automatique. Elles se vérifient en jouant, avec des joueurs réels, ce qu'on appelle le *playtest*.

La bonne question n'est donc pas « comment tester mon jeu ? » mais « **quelle partie de mon jeu est déterministe ?** ». Cette partie-là, on la teste automatiquement. Le reste, on le teste à la main, et on l'assume.

Voici la ligne de partage, appliquée à « Donjon de Dart ».

```text
  ┌──────────────────────────────────────────────────────────────────────┐
  │  SE TESTE AUTOMATIQUEMENT (déterministe, sans écran)                 │
  ├──────────────────────────────────────────────────────────────────────┤
  │  • Le calcul des dégâts : 100 PV − 25 dégâts = 75 PV                 │
  │  • Le clamp de la vie : on ne soigne jamais au-dessus de pvJoueurMax │
  │  • Le score : 3 pièces à 10 points avec un combo ×2 = 60 points      │
  │  • L'économie : acheter une potion à 50 pièces quand on en a 40      │
  │    doit échouer                                                      │
  │  • Le parsing d'un niveau : la carte 'List<String>' produit-elle     │
  │    le bon nombre de gobelins ?                                       │
  │  • La sérialisation de la sauvegarde : sauver puis charger doit      │
  │    redonner exactement le même état                                  │
  │  • La machine à états : de 'enJeu' on peut aller en 'pause',         │
  │    pas en 'chargement'                                               │
  │  • Le déplacement : après 1 seconde à 180 px/s, x a augmenté de 180  │
  │  • La collision : joueur + gobelin superposés → subirDegats appelé   │
  │  • Le HUD en texte : score 1240 → chaîne '01240'                     │
  └──────────────────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────────────────┐
  │  NE SE TESTE PAS AUTOMATIQUEMENT (subjectif ou visuel)               │
  ├──────────────────────────────────────────────────────────────────────┤
  │  • « Le saut est-il satisfaisant ? »                                 │
  │  • « La courbe de difficulté est-elle bonne ? »                      │
  │  • « Le niveau 2 est-il lisible ? »                                  │
  │  • « La musique colle-t-elle à l'ambiance ? »                        │
  │  • « Le joueur comprend-il qu'il faut la clé ? »                     │
  │  • « Le jeu est-il beau ? »                                          │
  │  → Ces questions se répondent en donnant la manette à quelqu'un      │
  │    d'autre et en le regardant jouer sans rien dire.                  │
  └──────────────────────────────────────────────────────────────────────┘
```

Regardez la première colonne. Ce n'est pas un hasard si elle contient presque exclusivement des choses que vous avez écrites dans des **fonctions pures** ou dans des **méthodes sans rendu**. C'est la récompense directe de la séparation logique / rendu défendue au chapitre 26 et appliquée depuis le chapitre 35 : ce qui est séparé du rendu est testable, ce qui est mélangé au rendu ne l'est pas.

Autrement dit : **la testabilité n'est pas une phase de fin de projet, c'est une conséquence de votre architecture.** Si vous découvrez au chapitre 42 que rien n'est testable, ce n'est pas un problème de tests, c'est un problème de conception.

> **À retenir :** on teste automatiquement ce qui est déterministe et sans écran. On teste à la main ce qui relève du ressenti. Un jeu bien architecturé rend la première catégorie beaucoup plus grande qu'on ne le croit.

---

## 42.2 — `flutter_test` et `flame_test`

Deux packages, deux rôles distincts.

### `flutter_test` — fourni avec le SDK

`flutter_test` fait partie du SDK Flutter. Vous ne le téléchargez pas depuis pub.dev : vous le déclarez avec `sdk: flutter`. Il apporte :

- la fonction `test()` et la fonction `expect()` (héritées du package `test` vu au chapitre 16) ;
- la fonction `testWidgets()` qui construit une vraie interface Flutter en mémoire ;
- l'objet `WidgetTester` avec `pumpWidget()`, `pump()`, `tap()`, `drag()` ;
- les *finders* : `find.text()`, `find.byType()`, `find.byKey()` ;
- `matchesGoldenFile()` pour les tests d'image.

### `flame_test` — publié par l'équipe Flame

`flame_test` est publié sur pub.dev par l'éditeur vérifié `flame-engine.org`. Version au 8 août 2026 : **2.2.4**. Il apporte ce que `flutter_test` ne sait pas faire : **instancier un jeu Flame, attendre qu'il soit prêt, et le nettoyer**.

Ses éléments principaux :

| Élément | Rôle |
| --- | --- |
| `testWithGame<T>` | Crée un jeu de votre type `T`, l'initialise, le passe au test, le nettoie. |
| `testWithFlameGame` | Même chose avec un `FlameGame` générique, sans sous-classe. |
| `FlameTester<T>` | Classe configurable (taille du jeu, widget englobant) réutilisable dans tout un fichier. |
| `GameTester<T>` | Version pour un `Game` quelconque, dont `FlameTester` hérite. |
| `testGolden` | Rend le jeu dans une image et la compare à une image de référence. |
| `closeToVector` | *Matcher* qui compare deux `Vector2` à une tolérance près. |
| `closeToAabb`, `closeToMatrix4` | Mêmes comparaisons pour des boîtes englobantes et des matrices. |
| `createTapDownEvents`, `createDragStartEvents` | Fabriquent des événements d'entrée synthétiques. |
| `expectDouble` | Compare deux `double` avec un `epsilon`. |

### La déclaration dans `pubspec.yaml`

Les deux vont dans `dev_dependencies`, jamais dans `dependencies` : ce sont des outils de développement, ils ne doivent pas grossir l'application livrée.

```yaml
name: donjon_de_dart
description: Un jeu de plateforme 2D en Flutter et Flame.
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: ^3.12.0

dependencies:
  flutter:
    sdk: flutter
  flame: ^1.38.0
  flame_audio: ^2.12.2
  shared_preferences: ^2.3.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flame_test: ^2.2.4
  flutter_lints: ^5.0.0

flutter:
  uses-material-design: true
  assets:
    - assets/images/
    - assets/audio/
```

Puis :

```bash
flutter pub get
```

**Résultat :**

```text
Resolving dependencies...
+ flame_test 2.2.4
Changed 1 dependency!
```

> **Piège classique :** mettre `flame_test` dans `dependencies`. Le code compilera, les tests passeront, et votre APK contiendra une bibliothèque de test inutile. Le compilateur ne vous dira rien. Relisez toujours votre `pubspec.yaml` avant publication.

---

## 42.3 — Tester une fonction de logique pure (dégâts, score, économie)

C'est le test le plus simple, le plus rapide et le plus rentable. Aucun moteur, aucun écran, aucune image : juste des fonctions Dart et des `expect`.

### La logique à tester

Reprenons trois règles du jeu et isolons-les dans un fichier de règles pures. Si ce fichier n'existe pas encore dans votre projet, créez-le : c'est un excellent réflexe d'architecture.

```dart
// lib/core/regles.dart

/// Règles de calcul du jeu, sans aucune dépendance à Flame ni à Flutter.
/// Tout est statique, tout est pur : mêmes entrées, mêmes sorties, toujours.
class Regles {
  /// Applique des dégâts à une valeur de PV, sans jamais descendre sous zéro.
  static double appliquerDegats(double pv, double degats) {
    if (degats < 0) {
      throw ArgumentError.value(degats, 'degats', 'Les dégâts ne peuvent pas être négatifs');
    }
    final double resultat = pv - degats;
    return resultat < 0 ? 0 : resultat;
  }

  /// Soigne sans jamais dépasser le maximum.
  static double soigner(double pv, double points, double pvMax) {
    final double resultat = pv + points;
    return resultat > pvMax ? pvMax : resultat;
  }

  /// Points gagnés pour un objet, en tenant compte du multiplicateur de combo.
  static int pointsAvecCombo(int valeurDeBase, int multiplicateur) {
    return valeurDeBase * multiplicateur;
  }

  /// Peut-on acheter un objet ? Retourne le solde restant, ou null si trop cher.
  static int? acheter(int solde, int prix) {
    if (prix > solde) {
      return null;
    }
    return solde - prix;
  }
}
```

### Le test

```dart
// test/core/regles_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:donjon_de_dart/core/regles.dart';

void main() {
  group('Regles.appliquerDegats', () {
    test('retire simplement les dégâts quand il reste assez de PV', () {
      expect(Regles.appliquerDegats(100, 25), 75);
    });

    test('ne descend jamais sous zéro', () {
      expect(Regles.appliquerDegats(10, 999), 0);
    });

    test('zéro dégât ne change rien', () {
      expect(Regles.appliquerDegats(42, 0), 42);
    });

    test('refuse des dégâts négatifs', () {
      expect(() => Regles.appliquerDegats(100, -5), throwsArgumentError);
    });
  });

  group('Regles.soigner', () {
    test('soigne normalement sous le plafond', () {
      expect(Regles.soigner(50, 20, 100), 70);
    });

    test('ne dépasse jamais le plafond', () {
      expect(Regles.soigner(90, 50, 100), 100);
    });

    test('soigner un personnage déjà au maximum ne change rien', () {
      expect(Regles.soigner(100, 30, 100), 100);
    });
  });

  group('Regles.pointsAvecCombo', () {
    test('multiplicateur 1 : la valeur de base', () {
      expect(Regles.pointsAvecCombo(10, 1), 10);
    });

    test('multiplicateur 3 : trois fois la valeur', () {
      expect(Regles.pointsAvecCombo(10, 3), 30);
    });
  });

  group('Regles.acheter', () {
    test('achat possible : le solde diminue', () {
      expect(Regles.acheter(100, 40), 60);
    });

    test('achat impossible : null', () {
      expect(Regles.acheter(30, 50), isNull);
    });

    test('achat au prix exact : solde à zéro', () {
      expect(Regles.acheter(50, 50), 0);
    });
  });
}
```

### Le lancement

```bash
flutter test test/core/regles_test.dart
```

**Résultat :**

```text
00:02 +12: All tests passed!
```

Douze tests, deux secondes. Voilà le rapport coût / bénéfice qui rend les tests unitaires irrésistibles.

### Anatomie d'un test

Trois éléments, toujours les mêmes :

```text
  group('nom du groupe', () {          ← regroupe des tests liés
    test('description du cas', () {    ← un cas = un test
      expect(valeurObtenue, valeurAttendue);   ← l'assertion
    });
  });
```

La **description** compte plus qu'on ne croit. Quand un test échoue six mois plus tard, la seule chose que vous lirez, c'est sa description. Écrivez-la comme une phrase de spécification : « ne descend jamais sous zéro » est utile ; « test 3 » ne l'est pas.

### Les matchers utiles

| Matcher | Signification |
| --- | --- |
| `equals(x)` ou simplement `x` | Égalité. |
| `isNull` / `isNotNull` | Nullité. |
| `isTrue` / `isFalse` | Booléens. |
| `isZero` / `isPositive` / `isNegative` | Signe d'un nombre. |
| `closeTo(x, epsilon)` | Égalité de `double` à une tolérance près. |
| `greaterThan(x)` / `lessThan(x)` | Comparaisons. |
| `contains(x)` | Un élément dans une collection ou une sous-chaîne. |
| `hasLength(n)` | Taille d'une collection. |
| `throwsArgumentError`, `throwsStateError` | Exceptions attendues (chapitre 13). |
| `isA<MonType>()` | Vérification de type. |

> **Le piège des `double` :** n'écrivez jamais `expect(0.1 + 0.2, 0.3)`. En virgule flottante, cela vaut `0.30000000000000004` et le test échoue. Écrivez `expect(0.1 + 0.2, closeTo(0.3, 0.0001))`. Dans un jeu où tout est en `double` (positions, vitesses, PV), cette règle s'applique en permanence.

---

## 42.4 — Tester un composant Flame avec `FlameTester`

Un composant Flame n'est pas une fonction pure. Il a un cycle de vie : il est créé, ajouté à un parent, monté (`onMount`), chargé (`onLoad`), mis à jour, puis retiré. Un test doit **respecter ce cycle**, sinon il teste un objet dans un état qui n'existe jamais en vrai.

C'est exactement le rôle de `flame_test`.

### `testWithGame` : la forme la plus directe

Signature exacte, telle que documentée sur pub.dev :

```dart
void testWithGame<T extends FlameGame<World>>(
  String testName,
  CreateFunction<T> create,
  AsyncGameFunction<T> testBody, {
  Timeout? timeout,
  dynamic tags,
  dynamic skip,
  Map<String, dynamic>? onPlatform,
  int? retry,
})
```

En clair : un nom, une fabrique de jeu, un corps de test asynchrone qui reçoit le jeu.

L'exemple officiel :

```dart
testWithGame<MyGame>(
  'MyComponent can be added to MyGame',
  () => MyGame(mySecret: 3781),
  (MyGame game) async {
    final component = MyComponent()..addToParent(game);
    await game.ready();
    expect(component.isMounted, true);
  },
);
```

Retenez la ligne centrale : **`await game.ready();`**. Sans elle, le composant n'est pas encore monté et `isMounted` vaut `false`. C'est la première cause d'échec incompréhensible dans les tests Flame.

### Appliqué au joueur du donjon

```dart
// test/composants/joueur_test.dart
import 'package:flame/components.dart';
import 'package:flame_test/flame_test.dart';
import 'package:flutter_test/flutter_test.dart';

import 'package:donjon_de_dart/config/constantes.dart';
import 'package:donjon_de_dart/composants/joueur.dart';
import 'package:donjon_de_dart/donjon_game.dart';

void main() {
  group('Joueur', () {
    testWithGame<DonjonGame>(
      'le joueur se monte avec ses PV au maximum',
      DonjonGame.new,
      (DonjonGame game) async {
        final Joueur joueur = Joueur(position: Vector2(100, 100));
        await game.world.add(joueur);
        await game.ready();

        expect(joueur.isMounted, isTrue);
        expect(joueur.pv, Constantes.pvJoueurMax);
        expect(joueur.etat, EtatJoueur.immobile);
        expect(joueur.cles, 0);
      },
    );

    testWithGame<DonjonGame>(
      'subirDegats retire les PV et rend invincible',
      DonjonGame.new,
      (DonjonGame game) async {
        final Joueur joueur = Joueur(position: Vector2.zero());
        await game.world.add(joueur);
        await game.ready();

        joueur.subirDegats(30);

        expect(joueur.pv, Constantes.pvJoueurMax - 30);
        expect(joueur.invincible, isTrue);
      },
    );

    testWithGame<DonjonGame>(
      'un joueur invincible ne subit pas deux fois les dégâts',
      DonjonGame.new,
      (DonjonGame game) async {
        final Joueur joueur = Joueur(position: Vector2.zero());
        await game.world.add(joueur);
        await game.ready();

        joueur.subirDegats(30);
        joueur.subirDegats(30); // ignoré : invincible

        expect(joueur.pv, Constantes.pvJoueurMax - 30);
      },
    );
  });
}
```

Remarquez `DonjonGame.new` : c'est la référence au constructeur, syntaxe Dart vue au chapitre 9. C'est strictement équivalent à `() => DonjonGame()`, en plus court.

### `FlameTester` : quand tous les tests d'un fichier partagent la même configuration

Si les dix tests d'un fichier veulent le même jeu et la même taille de fenêtre, répéter la fabrique dix fois est inutile. `FlameTester` factorise cela.

Constructeur exact :

```dart
FlameTester(
  GameCreateFunction<T> createGame, {
  Vector2? gameSize,
  GameWidgetCreateFunction<T>? createGameWidget,
  PumpWidgetFunction<T>? pumpWidget,
})
```

Usage :

```dart
// test/composants/gobelin_test.dart
import 'package:flame/components.dart';
import 'package:flame_test/flame_test.dart';
import 'package:flutter_test/flutter_test.dart';

import 'package:donjon_de_dart/composants/gobelin.dart';
import 'package:donjon_de_dart/donjon_game.dart';

void main() {
  // Une seule configuration, partagée par tout le fichier.
  final FlameTester<DonjonGame> jeu = FlameTester<DonjonGame>(
    DonjonGame.new,
    gameSize: Vector2(800, 600),
  );

  group('Gobelin', () {
    jeu.testGameWidget(
      'le gobelin apparaît avec ses PV pleins',
      setUp: (DonjonGame game, WidgetTester tester) async {
        await game.world.add(Gobelin(position: Vector2(200, 200)));
        await game.ready();
      },
      verify: (DonjonGame game, WidgetTester tester) async {
        final Gobelin g = game.world.children.whereType<Gobelin>().first;
        expect(g.pv, g.pvMax);
        expect(g.estVivant, isTrue);
      },
    );
  });
}
```

`testGameWidget` prend un `setUp` (préparer la scène) et un `verify` (vérifier). Signature exacte, telle que documentée :

```dart
void testGameWidget(
  String description, {
  WidgetSetupFunction<T>? setUp,
  WidgetVerifyFunction<T>? verify,
  bool? skip,
  Timeout? timeout,
  bool? semanticsEnabled,
  dynamic tags,
})
```

Le paramètre `gameSize` remplace la taille par défaut du jeu de test, qui est de **500 × 500** si vous ne dites rien. Cela compte : si votre HUD se positionne par rapport aux bords, un test à 500 × 500 ne donnera pas les mêmes coordonnées que votre téléphone.

### Quand utiliser lequel

| Situation | Outil |
| --- | --- |
| Un test isolé, un jeu par test | `testWithGame<T>` |
| Un test qui n'a pas besoin d'une sous-classe de `FlameGame` | `testWithFlameGame` |
| Dix tests dans un fichier, même configuration | `FlameTester<T>` |
| Test qui a besoin d'un vrai arbre de widgets Flutter autour du jeu | `FlameTester.testGameWidget` |
| Test de rendu comparé à une image | `testGolden` |

---

## 42.5 — Simuler des frames (`game.update(dt)`)

Voilà le point qui change tout dans le test d'un jeu.

Dans un test, **il n'y a pas de boucle de jeu**. Personne n'appelle `update()` soixante fois par seconde. Le temps ne passe pas tout seul. Si vous ajoutez un joueur avec une vitesse de 180 px/s et que vous vérifiez immédiatement sa position, elle n'aura pas bougé d'un pixel — et c'est normal.

**C'est vous qui faites passer le temps**, en appelant `game.update(dt)` avec la valeur de `dt` de votre choix.

```text
  DANS LE JEU RÉEL                     DANS UN TEST
  ────────────────                     ────────────
  Le moteur appelle update(dt)         Vous appelez game.update(dt)
  ~60 fois par seconde                 autant de fois que vous voulez
  dt ≈ 0.0167 s, variable              dt exactement ce que vous décidez
  Vous subissez le temps               Vous contrôlez le temps
```

C'est un pouvoir considérable : vous pouvez simuler dix minutes de jeu en une milliseconde, ou avancer image par image pour observer un comportement précis.

### Un déplacement sur une seconde

```dart
testWithGame<DonjonGame>(
  'le joueur avance de 180 pixels en une seconde à pleine vitesse',
  DonjonGame.new,
  (DonjonGame game) async {
    final Joueur joueur = Joueur(position: Vector2(0, 0));
    await game.world.add(joueur);
    await game.ready();

    joueur.velocite.setValues(Constantes.vitesseJoueur, 0);

    // Une seconde simulée en 60 frames de 1/60 s.
    for (int i = 0; i < 60; i++) {
      game.update(1 / 60);
    }

    expect(joueur.position.x, closeTo(180, 1.0));
  },
);
```

Notez `closeTo(180, 1.0)` : soixante additions de `180 * (1/60)` ne redonnent pas exactement `180` en virgule flottante. Une tolérance d'un pixel est raisonnable pour un jeu.

### Un raccourci : une seule grande frame

Pour un mouvement linéaire, une frame d'une seconde donne le même résultat que soixante frames d'un soixantième :

```dart
game.update(1.0);
expect(joueur.position.x, closeTo(180, 0.001));
```

**Attention :** cela n'est vrai que pour un mouvement **linéaire**. Dès qu'il y a accélération (la gravité), une grosse frame et soixante petites ne donnent pas le même résultat, exactement pour la raison expliquée au chapitre 23 sur l'intégration d'Euler. Pour tout ce qui accélère, simulez de vraies petites frames.

### Une fonction utilitaire à s'écrire une fois pour toutes

```dart
// test/_helpers/simulation.dart
import 'package:flame/game.dart';

/// Fait avancer [game] de [secondes] au pas fixe [pas].
/// Reproduit le comportement d'une boucle de jeu à cadence régulière.
void simuler(FlameGame game, double secondes, {double pas = 1 / 60}) {
  double restant = secondes;
  while (restant > 0) {
    final double dt = restant < pas ? restant : pas;
    game.update(dt);
    restant -= dt;
  }
}
```

Usage :

```dart
simuler(game, 2.5);            // 2,5 secondes de jeu simulé
simuler(game, 1.0, pas: 1/30); // 1 seconde à 30 FPS simulés
```

### Tester une fin d'invincibilité

Cette fonction rend testable des comportements liés au temps qui seraient sinon impossibles à vérifier.

```dart
testWithGame<DonjonGame>(
  "l'invincibilité s'éteint après dureeInvincibilite",
  DonjonGame.new,
  (DonjonGame game) async {
    final Joueur joueur = Joueur(position: Vector2.zero());
    await game.world.add(joueur);
    await game.ready();

    joueur.subirDegats(10);
    expect(joueur.invincible, isTrue);

    // Juste avant la fin : toujours invincible.
    simuler(game, Constantes.dureeInvincibilite - 0.1);
    expect(joueur.invincible, isTrue);

    // Juste après : plus invincible.
    simuler(game, 0.2);
    expect(joueur.invincible, isFalse);
  },
);
```

Ce test vaut de l'or. Il vérifie une règle de gameplay (1,2 seconde d'invincibilité) que vous ne pourriez pas mesurer au chronomètre en jouant, et il la vérifiera automatiquement à chaque modification du code pendant des années.

### Tester la gravité

```dart
testWithGame<DonjonGame>(
  'un joueur sans sol tombe et sa vitesse plafonne',
  DonjonGame.new,
  (DonjonGame game) async {
    final Joueur joueur = Joueur(position: Vector2(0, 0));
    await game.world.add(joueur);
    await game.ready();

    joueur.auSol = false;
    simuler(game, 5.0); // cinq secondes de chute libre

    expect(joueur.position.y, greaterThan(0));
    expect(joueur.velocite.y, lessThanOrEqualTo(Constantes.vitesseMaxChute));
  },
);
```

Ce test protège une règle discrète mais critique : la vitesse de chute est plafonnée. Sans plafond, un joueur qui tombe longtemps traverse le sol (le *tunneling* du chapitre 24). Le jour où quelqu'un supprimera le `clamp` par inadvertance, ce test le rattrapera.

> **À retenir :** dans un test, le temps ne passe pas tout seul. `game.update(dt)` est votre machine à remonter le temps. Utilisez de petits pas dès qu'il y a une accélération.

---

## 42.6 — Tester une collision

Une collision Flame n'est pas détectée au moment où vous déplacez un composant. Elle est détectée **pendant `update()`**, par le système fourni par le mixin `HasCollisionDetection` (chapitre 32).

La conséquence pratique est capitale :

```text
  1. Placer les deux composants de manière à ce qu'ils se chevauchent
  2. await game.ready()          ← les hitboxes sont montées
  3. game.update(0)              ← LA détection a lieu ICI
  4. expect(...)                 ← seulement maintenant on vérifie
```

Un `dt` de `0` suffit : on ne veut pas faire avancer le monde, seulement déclencher la passe de détection.

### Le test

```dart
// test/composants/collision_test.dart
import 'package:flame/components.dart';
import 'package:flame_test/flame_test.dart';
import 'package:flutter_test/flutter_test.dart';

import 'package:donjon_de_dart/config/constantes.dart';
import 'package:donjon_de_dart/composants/gobelin.dart';
import 'package:donjon_de_dart/composants/piece.dart';
import 'package:donjon_de_dart/composants/joueur.dart';
import 'package:donjon_de_dart/donjon_game.dart';

void main() {
  group('Collisions', () {
    testWithGame<DonjonGame>(
      'toucher un gobelin fait perdre des PV au joueur',
      DonjonGame.new,
      (DonjonGame game) async {
        final Joueur joueur = Joueur(position: Vector2(100, 100));
        final Gobelin gobelin = Gobelin(position: Vector2(100, 100));

        await game.world.addAll(<Component>[joueur, gobelin]);
        await game.ready();

        final double pvAvant = joueur.pv;

        game.update(0); // déclenche la détection de collision

        expect(joueur.pv, lessThan(pvAvant));
        expect(joueur.invincible, isTrue);
      },
    );

    testWithGame<DonjonGame>(
      'un gobelin éloigné ne touche pas le joueur',
      DonjonGame.new,
      (DonjonGame game) async {
        final Joueur joueur = Joueur(position: Vector2(0, 0));
        final Gobelin gobelin = Gobelin(position: Vector2(1000, 1000));

        await game.world.addAll(<Component>[joueur, gobelin]);
        await game.ready();

        final double pvAvant = joueur.pv;
        game.update(0);

        expect(joueur.pv, pvAvant);
        expect(joueur.invincible, isFalse);
      },
    );

    testWithGame<DonjonGame>(
      'ramasser une pièce augmente le score et retire la pièce',
      DonjonGame.new,
      (DonjonGame game) async {
        final Joueur joueur = Joueur(position: Vector2(50, 50));
        final Piece piece = Piece(position: Vector2(50, 50));

        await game.world.addAll(<Component>[joueur, piece]);
        await game.ready();

        expect(game.score, 0);

        game.update(0);        // collision détectée, ramasser() appelé
        await game.ready();    // laisse le retrait du composant se finaliser

        expect(game.score, greaterThan(0));
        expect(game.world.children.whereType<Piece>(), isEmpty);
      },
    );
  });
}
```

### Le deuxième test est le plus important

Le premier test vérifie que la collision **arrive** quand elle doit arriver. Le deuxième vérifie qu'elle **n'arrive pas** quand elle ne doit pas.

Les débutants n'écrivent que le premier. C'est une erreur. Un bug fréquent consiste à donner à une hitbox une taille bien trop grande : le joueur prend des dégâts alors qu'il est visiblement à trois mètres du gobelin. Le premier test passe malgré ce bug. Seul le second le détecte.

**Règle générale : pour chaque test « cela se produit », écrivez le test « cela ne se produit pas ».**

### Le troisième test et le `await game.ready()` de la fin

Dans Flame, `removeFromParent()` n'agit pas instantanément : le retrait est mis en file d'attente et traité au cycle suivant. Dans le jeu réel, personne ne le remarque. Dans un test, si vous vérifiez immédiatement, la pièce est encore là et le test échoue.

`await game.ready()` vide cette file. C'est le complément indispensable de tout test qui vérifie une **disparition**.

> **Piège classique :** oublier `game.update(0)` et se demander pourquoi `onCollisionStart` n'est jamais appelé. Les hitboxes se chevauchent visuellement dans votre tête, mais le moteur n'a pas encore eu l'occasion de le constater.

---

## 42.7 — Les tests de widgets pour les menus

Le jeu tourne dans un `GameWidget`, mais tout ce qui l'entoure — menu principal, écran de pause, Game Over, victoire — est en **widgets Flutter**, dans des overlays (chapitres 35 et 40). Cela se teste avec `testWidgets`.

### Le principe

```text
  testWidgets('description', (WidgetTester tester) async {
    await tester.pumpWidget(...);   ← construit l'interface en mémoire
    expect(find.text('Jouer'), findsOneWidget);   ← cherche dedans
    await tester.tap(find.text('Jouer'));         ← simule un appui
    await tester.pump();                          ← reconstruit
    expect(...);                                  ← vérifie l'effet
  });
```

Il n'y a pas d'écran : Flutter construit l'arbre de widgets en mémoire, le met en page et permet de l'interroger. C'est rapide (quelques millisecondes) et parfaitement déterministe.

### Le test du menu principal

```dart
// test/ecrans/menu_principal_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

import 'package:donjon_de_dart/ecrans/menu_principal.dart';

void main() {
  group('MenuPrincipal', () {
    testWidgets('affiche le titre et les trois boutons', (WidgetTester tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: MenuPrincipal(
            onJouer: () {},
            onOptions: () {},
            onQuitter: () {},
            meilleurScore: 0,
          ),
        ),
      );

      expect(find.text('DONJON DE DART'), findsOneWidget);
      expect(find.text('JOUER'), findsOneWidget);
      expect(find.text('OPTIONS'), findsOneWidget);
      expect(find.text('QUITTER'), findsOneWidget);
    });

    testWidgets('le bouton JOUER déclenche le rappel', (WidgetTester tester) async {
      int appels = 0;

      await tester.pumpWidget(
        MaterialApp(
          home: MenuPrincipal(
            onJouer: () => appels++,
            onOptions: () {},
            onQuitter: () {},
            meilleurScore: 0,
          ),
        ),
      );

      await tester.tap(find.text('JOUER'));
      await tester.pump();

      expect(appels, 1);
    });

    testWidgets('le meilleur score est masqué quand il vaut zéro',
        (WidgetTester tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: MenuPrincipal(
            onJouer: () {},
            onOptions: () {},
            onQuitter: () {},
            meilleurScore: 0,
          ),
        ),
      );

      expect(find.textContaining('Meilleur score'), findsNothing);
    });

    testWidgets('le meilleur score est affiché quand il est positif',
        (WidgetTester tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: MenuPrincipal(
            onJouer: () {},
            onOptions: () {},
            onQuitter: () {},
            meilleurScore: 4520,
          ),
        ),
      );

      expect(find.textContaining('4520'), findsOneWidget);
    });
  });
}
```

### Les finders

| Finder | Ce qu'il cherche |
| --- | --- |
| `find.text('JOUER')` | Un widget texte dont le contenu vaut exactement `JOUER`. |
| `find.textContaining('score')` | Un texte qui contient la sous-chaîne. |
| `find.byType(ElevatedButton)` | Tous les widgets d'un type donné. |
| `find.byKey(const Key('bouton_jouer'))` | Le widget portant cette clé. |
| `find.byIcon(Icons.pause)` | Une icône précise. |
| `find.byTooltip('Pause')` | Un widget par son infobulle. |

Et les *matchers* associés :

| Matcher | Signification |
| --- | --- |
| `findsOneWidget` | Exactement un. |
| `findsNothing` | Aucun. |
| `findsWidgets` | Au moins un. |
| `findsNWidgets(3)` | Exactement trois. |

### `pump` contre `pumpAndSettle`

| Méthode | Effet |
| --- | --- |
| `await tester.pump()` | Reconstruit une seule frame. |
| `await tester.pump(const Duration(milliseconds: 300))` | Avance le temps de 300 ms et reconstruit. |
| `await tester.pumpAndSettle()` | Reconstruit en boucle jusqu'à ce que plus aucune animation ne soit en cours. |

> **Piège classique :** `pumpAndSettle()` sur un écran contenant une animation **infinie** (un titre qui pulse en boucle, par exemple) ne se termine jamais et le test expire au bout de dix minutes. Dans ce cas, utilisez `pump(Duration(...))` avec une durée fixe. C'est une des raisons pour lesquelles les animations infinies dans les menus doivent rester identifiables et, idéalement, désactivables.

### Utiliser des `Key` pour rendre les tests robustes

Chercher un bouton par son texte est fragile : le jour où vous traduisez le menu en anglais, tous les tests cassent. Une `Key` est stable.

```dart
ElevatedButton(
  key: const Key('bouton_jouer'),
  onPressed: onJouer,
  child: const Text('JOUER'),
)
```

```dart
await tester.tap(find.byKey(const Key('bouton_jouer')));
```

Prenez l'habitude de poser une `Key` sur tout élément interactif d'un menu. Cela coûte une ligne et cela vous épargne une réécriture complète de vos tests le jour de la traduction.

---

## 42.8 — Les golden tests : intérêt et limites pour un jeu

Un **golden test** (test « d'image de référence ») rend un widget ou un jeu dans une image, puis compare **pixel par pixel** cette image à un fichier PNG de référence stocké dans le dépôt.

### En Flutter pur

```dart
testWidgets('le HUD ressemble à sa référence', (WidgetTester tester) async {
  await tester.pumpWidget(const MaterialApp(home: HudDemo()));
  await expectLater(
    find.byType(HudDemo),
    matchesGoldenFile('goldens/hud.png'),
  );
});
```

### Avec `flame_test`

`flame_test` fournit `testGolden`, dont voici la signature exacte :

```dart
@isTest
void testGolden(
  String testName,
  PrepareFunction testBody, {
  required String goldenFile,
  Vector2? size,
  Color? backgroundColor,
  FlameGame<World>? game,
  bool skip = false,
})
```

La fonction prépare la scène, rend le jeu dans une image, la compare au fichier de référence. Elle appelle `await game.ready()` automatiquement avant de rendre. La taille par défaut, si vous ne passez pas `size`, est **2400 × 1800**.

```dart
// test/golden/piece_golden_test.dart
import 'package:flame/components.dart';
import 'package:flame_test/flame_test.dart';
import 'package:flutter/material.dart';

import 'package:donjon_de_dart/composants/piece.dart';

void main() {
  testGolden(
    'une pièce dessinée sans image',
    (game, tester) async {
      await game.world.add(Piece(position: Vector2(50, 50)));
    },
    goldenFile: 'goldens/piece.png',
    size: Vector2(200, 200),
    backgroundColor: const Color(0xFF101018),
  );
}
```

### Générer les images de référence

La première exécution échoue forcément : le fichier de référence n'existe pas. On le crée avec :

```bash
flutter test --update-goldens
```

**Résultat :**

```text
00:04 +1: All tests passed!
```

Le fichier `test/golden/goldens/piece.png` est créé. Vous devez le **committer** dans Git : c'est lui, la référence.

### Pourquoi c'est séduisant

Un golden test attrape des régressions qu'aucun `expect` ne verrait : une couleur modifiée par erreur, un décalage de deux pixels, une barre de vie qui déborde de son cadre, une police qui change. Sur une interface stable, c'est puissant.

### Pourquoi c'est dangereux dans un jeu

Il faut être franc : **dans un jeu, les golden tests cassent tout le temps, souvent sans que rien ne soit cassé.**

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │  Causes d'échec d'un golden test qui ne sont PAS des bugs        │
  ├──────────────────────────────────────────────────────────────────┤
  │  • L'animation flottante des pièces n'était pas à la même phase  │
  │  • Une particule aléatoire est partie à gauche au lieu de droite │
  │  • Le rendu du texte diffère entre Linux, macOS et Windows       │
  │  • Une version de Flutter a changé l'anticrénelage d'un cercle   │
  │  • Le lissage des polices diffère entre la CI et votre machine   │
  │  • Vous avez ajusté une couleur de 2 unités volontairement       │
  └──────────────────────────────────────────────────────────────────┘
```

Un test qui échoue souvent pour de mauvaises raisons finit ignoré. Un test ignoré est pire qu'un test absent, parce qu'il donne une fausse impression de sécurité.

### La position raisonnable

| Recommandation | Détail |
| --- | --- |
| N'en faites pas beaucoup | Cinq à dix golden tests suffisent pour un jeu de cette taille. |
| Choisissez des scènes **statiques** | Une pièce immobile, une barre de vie à 50 %, l'icône du HUD. Jamais une explosion. |
| Neutralisez l'aléatoire | Passez une graine fixe à vos `Random` (voir `testRandom` de `flame_test`). |
| Figez l'animation | Rendez la scène à `t = 0`, sans appeler `update`. |
| Fixez la police | Le rendu de texte varie d'une plateforme à l'autre. `flame_test` fournit `DebugTextRenderer`, qui dessine les mots comme des rectangles précisément pour éviter ce problème. |
| Regénérez consciemment | `--update-goldens` doit être une décision, jamais un réflexe pour faire taire un test rouge. |

> **À retenir :** le golden test est un excellent gardien pour une interface figée, et un mauvais gardien pour un jeu animé. Utilisez-le avec parcimonie, sur des scènes que vous avez volontairement rendues immobiles.

---

## 42.9 — Organiser le dossier `test/`

La règle est simple et ne souffre aucune exception : **le dossier `test/` reflète le dossier `lib/`**.

```text
  donjon_de_dart/
  ├── lib/
  │   ├── donjon_game.dart
  │   ├── config/
  │   │   └── constantes.dart
  │   ├── core/
  │   │   ├── game_state.dart
  │   │   ├── regles.dart
  │   │   └── sante.dart
  │   ├── composants/
  │   │   ├── joueur.dart
  │   │   ├── gobelin.dart
  │   │   └── piece.dart
  │   ├── niveaux/
  │   │   ├── niveau.dart
  │   │   └── niveaux_data.dart
  │   ├── services/
  │   │   └── sauvegarde_service.dart
  │   └── ecrans/
  │       └── menu_principal.dart
  │
  └── test/
      ├── _helpers/
      │   ├── simulation.dart          ← utilitaires partagés, pas des tests
      │   └── fixtures.dart
      ├── core/
      │   ├── regles_test.dart
      │   ├── game_state_test.dart
      │   └── sante_test.dart
      ├── composants/
      │   ├── joueur_test.dart
      │   ├── gobelin_test.dart
      │   ├── piece_test.dart
      │   └── collision_test.dart
      ├── niveaux/
      │   └── niveau_test.dart
      ├── services/
      │   └── sauvegarde_service_test.dart
      ├── ecrans/
      │   └── menu_principal_test.dart
      ├── golden/
      │   ├── goldens/
      │   │   ├── piece.png
      │   │   └── hud.png
      │   └── piece_golden_test.dart
      └── donjon_game_test.dart
```

Quatre règles à respecter :

1. **Un fichier de `lib/` → un fichier de `test/` du même nom suffixé `_test.dart`.** Le suffixe n'est pas décoratif : `flutter test` ne ramasse que les fichiers qui finissent par `_test.dart`.
2. **Les utilitaires partagés vont dans un dossier qui n'est pas ramassé**, par exemple `_helpers/`. Un fichier nommé `simulation.dart` (sans `_test`) ne sera pas exécuté comme un test, c'est ce qu'on veut.
3. **Un fichier de test ne dépend jamais d'un autre fichier de test.** Chaque fichier doit pouvoir se lancer seul.
4. **Les tests d'intégration vont dans `integration_test/`**, à la racine du projet, pas dans `test/`. Ils tournent sur un vrai appareil et se lancent avec `flutter test integration_test`.

### Les fixtures : arrêter de répéter la préparation

```dart
// test/_helpers/fixtures.dart
import 'package:flame/components.dart';

import 'package:donjon_de_dart/composants/gobelin.dart';
import 'package:donjon_de_dart/composants/joueur.dart';
import 'package:donjon_de_dart/donjon_game.dart';

/// Ajoute un joueur au monde et le renvoie, prêt à être testé.
Future<Joueur> ajouterJoueur(DonjonGame game, {Vector2? position}) async {
  final Joueur joueur = Joueur(position: position ?? Vector2.zero());
  await game.world.add(joueur);
  await game.ready();
  return joueur;
}

/// Ajoute un gobelin au monde et le renvoie.
Future<Gobelin> ajouterGobelin(DonjonGame game, {Vector2? position}) async {
  final Gobelin gobelin = Gobelin(position: position ?? Vector2(200, 0));
  await game.world.add(gobelin);
  await game.ready();
  return gobelin;
}

/// Une carte minimale valide, utile à de nombreux tests de niveau.
const List<String> carteDeTest = <String>[
  '##########',
  '#........#',
  '#..o..g..#',
  '#.J.....D#',
  '##########',
];
```

Un test devient alors court et lisible :

```dart
testWithGame<DonjonGame>(
  'le gobelin patrouille vers la gauche puis revient',
  DonjonGame.new,
  (DonjonGame game) async {
    final Gobelin gobelin = await ajouterGobelin(game, position: Vector2(300, 0));
    final double departX = gobelin.position.x;

    simuler(game, 1.0);
    expect(gobelin.position.x, isNot(closeTo(departX, 0.1)));
  },
);
```

> **À retenir :** un test doit se lire comme une phrase. Si sa préparation fait quinze lignes, sortez-la dans un helper. Un test illisible ne sera jamais maintenu.

---

## 42.10 — `flutter test` et la couverture

### Lancer tous les tests

```bash
flutter test
```

**Résultat :**

```text
00:06 +47: All tests passed!
```

Le `+47` compte les tests réussis. En cas d'échec, la sortie est autrement plus bavarde :

```text
00:05 +31 -1: Joueur subirDegats retire les PV [E]
  Expected: <70.0>
    Actual: <75.0>

  package:matcher                          expect
  test/composants/joueur_test.dart 42:9    main.<fn>.<fn>

00:06 +46 -1: Some tests failed.
```

Trois informations : ce qui était attendu, ce qui a été obtenu, et le fichier avec le numéro de ligne. C'est presque toujours suffisant pour comprendre.

### Cibler l'exécution

```bash
# Un seul fichier
flutter test test/composants/joueur_test.dart

# Un seul dossier
flutter test test/composants/

# Tous les tests dont le nom contient « collision »
flutter test --name collision

# Tous les tests portant un tag donné
flutter test --tags lent

# Sortie détaillée, test par test
flutter test --reporter expanded
```

### La couverture de code

La **couverture** mesure le pourcentage de lignes de `lib/` exécutées au moins une fois pendant les tests.

```bash
flutter test --coverage
```

Cela produit un fichier :

```text
coverage/lcov.info
```

C'est un format texte standard, exploitable par de nombreux outils. Sur macOS ou Linux, avec `lcov` installé, on en tire un rapport HTML :

```bash
genhtml coverage/lcov.info -o coverage/html
```

Puis on ouvre `coverage/html/index.html` dans un navigateur. Chaque ligne de code apparaît en vert (exécutée) ou en rouge (jamais exécutée).

> **Remarque :** `genhtml` appartient au paquet `lcov`, qui n'est pas fourni par Flutter. Sur Ubuntu : `sudo apt install lcov`. Sur macOS avec Homebrew : `brew install lcov`. Si vous ne voulez rien installer, de nombreuses extensions VS Code lisent directement `lcov.info` et colorent votre code dans l'éditeur.

### Quel taux viser

C'est là que beaucoup d'équipes se trompent. **La couverture n'est pas un objectif, c'est un révélateur.**

| Taux | Interprétation raisonnable |
| --- | --- |
| Moins de 20 % | Le projet n'est pas testé. Commencez par les règles pures. |
| 40 % à 60 % | Correct pour un jeu : la logique est couverte, le rendu ne l'est pas. |
| 70 % à 80 % | Très bon, si c'est atteint honnêtement. |
| 100 % | Presque toujours le signe de tests écrits pour la statistique, pas pour la qualité. |

Dans un jeu, une grande partie de `lib/` est du **rendu** (`render()`, `Canvas`, couleurs). Ce code est difficile à couvrir et son test apporte peu. Une couverture de 50 % avec 100 % des règles métier testées vaut infiniment mieux qu'une couverture de 90 % obtenue en instanciant tous les composants sans rien vérifier.

Exclure le rendu du calcul, si votre outil le permet, donne une image plus honnête de la situation.

---

## 42.11 — L'intégration continue (GitHub Actions) : un workflow minimal

L'**intégration continue** (CI) consiste à faire exécuter automatiquement vos vérifications par un serveur, à chaque `push`. Vous ne pouvez plus oublier de lancer les tests : la machine le fait pour vous, et elle ne se fatigue pas.

GitHub Actions lit les fichiers YAML placés dans `.github/workflows/`.

### Le workflow

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  verifier:
    name: Analyse et tests
    runs-on: ubuntu-latest

    steps:
      - name: Récupérer le dépôt
        uses: actions/checkout@v4

      - name: Installer Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: stable
          flutter-version: 3.44.0
          cache: true

      - name: Afficher la version
        run: flutter --version

      - name: Installer les dépendances
        run: flutter pub get

      - name: Vérifier le formatage
        run: dart format --output=none --set-exit-if-changed .

      - name: Analyser le code
        run: flutter analyze

      - name: Lancer les tests avec couverture
        run: flutter test --coverage

      - name: Publier le rapport de couverture
        uses: actions/upload-artifact@v4
        with:
          name: couverture
          path: coverage/lcov.info
```

### Ce que fait chaque étape

| Étape | Rôle | Échec si… |
| --- | --- | --- |
| `actions/checkout` | Copie le code du dépôt sur la machine du serveur. | Jamais, en principe. |
| `subosito/flutter-action@v2` | Installe le SDK Flutter dans la version demandée. Action de référence de l'écosystème. | Version inexistante. |
| `flutter pub get` | Télécharge les dépendances. | Une dépendance est introuvable ou en conflit. |
| `dart format --set-exit-if-changed` | Vérifie le formatage sans modifier les fichiers. | Un fichier n'est pas formaté. |
| `flutter analyze` | Analyse statique. | Une erreur ou un avertissement configuré comme bloquant. |
| `flutter test --coverage` | Lance la suite de tests. | Un test échoue. |
| `actions/upload-artifact` | Attache `lcov.info` au résultat du workflow, téléchargeable. | Fichier absent. |

### Ajouter la compilation

Une fois la vérification en place, on ajoute un second travail qui produit les binaires. Il ne démarre que si le premier a réussi, grâce à `needs`.

```yaml
  construire:
    name: Compiler APK et Web
    needs: verifier
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: subosito/flutter-action@v2
        with:
          channel: stable
          flutter-version: 3.44.0
          cache: true

      - run: flutter pub get

      - name: Compiler l'APK de release
        run: flutter build apk --release

      - name: Compiler la version Web
        run: flutter build web --release

      - name: Publier l'APK
        uses: actions/upload-artifact@v4
        with:
          name: apk-release
          path: build/app/outputs/flutter-apk/app-release.apk

      - name: Publier le site
        uses: actions/upload-artifact@v4
        with:
          name: web-release
          path: build/web
```

Attention : l'APK produit ici est signé avec la **clé de debug** générée automatiquement, parce que le serveur n'a pas accès à votre keystore. Un tel APK s'installe sur un téléphone pour tester, mais **ne peut pas être publié sur le Play Store**. La section 42.33 explique pourquoi et comment faire autrement.

### La discipline que cela impose

Un workflow rouge doit être réparé **avant** toute autre chose. Le jour où l'équipe s'habitue à voir du rouge en permanence, la CI ne sert plus à rien. C'est une règle sociale autant que technique.

> **À retenir :** la CI ne rend pas votre code meilleur. Elle vous empêche seulement de publier du code cassé sans le savoir. C'est déjà énorme.

---

## 42.12 — `debugMode` de Flame

Passons du test au débogage. Flame embarque un mode de débogage visuel qui s'active en une ligne.

D'après la documentation officielle : quand `debugMode` vaut `true` sur un `FlameGame`, « chaque `PositionComponent` est rendu avec sa taille englobante, et ses positions sont écrites à l'écran ».

### Activation globale

```dart
class DonjonGame extends FlameGame
    with HasCollisionDetection, HasKeyboardHandlerComponents {

  DonjonGame() {
    debugMode = true; // TOUT l'arbre passe en mode debug
  }
}
```

### Activation ciblée

Mettre tout le jeu en `debugMode` produit un écran illisible dès qu'il y a trente composants. La plupart du temps, on n'en active qu'un seul :

```dart
class Joueur extends PositionComponent
    with HasGameReference<DonjonGame>, KeyboardHandler, CollisionCallbacks {

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    debugMode = true;                  // uniquement le joueur
    debugColor = const Color(0xFF00FF00); // et en vert
  }
}
```

`debugMode` se propage aux enfants : mettre le joueur en `debugMode` affiche aussi le cadre de ses hitboxes enfants. C'est exactement ce qu'on veut.

### Ce que vous voyez à l'écran

```text
  ┌─────────────────────────────────────────────────────┐
  │                                                     │
  │        ┌────────────┐                               │
  │        │            │  ← rectangle englobant        │
  │        │   JOUEUR   │                               │
  │        │            │                               │
  │        └────────────┘                               │
  │       (312.0, 480.0)  ← position écrite sous le     │
  │                          composant                  │
  └─────────────────────────────────────────────────────┘
```

### Le réflexe à prendre

Un composant invisible à l'écran, trois causes possibles, et `debugMode` les départage instantanément :

| Ce que montre `debugMode` | Diagnostic |
| --- | --- |
| Rien du tout | Le composant n'est pas monté. Vérifiez le `add()` et l'`await`. |
| Un cadre au bon endroit, mais rien dedans | Le composant est monté et placé ; c'est le **rendu** qui est en cause (couleur transparente, sprite non chargé, `size` nulle). |
| Un cadre hors de l'écran | Problème de **position** ou de **caméra** (chapitre 31). |
| Un cadre de taille `(0, 0)` | Vous avez oublié de fixer `size`. |

Ce petit tableau vous fera gagner des heures. Avant de relire cent lignes de code, activez `debugMode` et regardez.

> **Avertissement, à retenir dès maintenant :** `debugMode` a un coût de rendu. Il ne doit **jamais** rester actif dans une version publiée. Voir la section 42.26 pour la manière propre de s'en assurer.

---

## 42.13 — Afficher les hitboxes et les FPS

### Les hitboxes

Les hitboxes du chapitre 32 (`RectangleHitbox`, `CircleHitbox`) sont des composants comme les autres. Leur `debugMode` s'active de la même façon.

```dart
@override
Future<void> onLoad() async {
  await super.onLoad();

  final RectangleHitbox hitbox = RectangleHitbox(
    size: Vector2(24, 40),
    position: Vector2(4, 8),
  );
  hitbox.debugMode = true;
  hitbox.debugColor = const Color(0xFFFF00FF);
  await add(hitbox);
}
```

Un interrupteur global évite d'avoir à modifier vingt fichiers :

```dart
// lib/config/constantes.dart
class Constantes {
  // ... les autres constantes du contrat ...

  /// Interrupteur unique du mode debug visuel.
  /// Doit valoir false dans toute version publiée.
  static const bool afficherDebug = false;
}
```

```dart
hitbox.debugMode = Constantes.afficherDebug;
```

Encore mieux, Dart fournit une constante qui vaut `true` uniquement en compilation de debug :

```dart
import 'package:flutter/foundation.dart';

hitbox.debugMode = kDebugMode;
```

`kDebugMode` vient de `package:flutter/foundation.dart`. En mode release, elle vaut `false` **à la compilation**, ce qui permet au compilateur de supprimer purement et simplement le code correspondant. C'est la solution la plus sûre : on ne peut pas oublier de la désactiver.

### Les FPS

Flame fournit deux composants dédiés, documentés sur la page « Debug features » :

- `FpsComponent` : mesure le nombre d'images par seconde, sans rien afficher ;
- `FpsTextComponent` : enveloppe le précédent et affiche la valeur sous forme de texte.

```dart
import 'package:flame/components.dart';
import 'package:flutter/foundation.dart';

class DonjonGame extends FlameGame
    with HasCollisionDetection, HasKeyboardHandlerComponents {

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    if (kDebugMode) {
      await camera.viewport.add(
        FpsTextComponent(position: Vector2(8, 8)),
      );
    }
  }
}
```

Notez `camera.viewport.add` et non `world.add` : le compteur doit rester collé au coin de l'écran, pas se promener dans le donjon quand la caméra bouge. C'est exactement la logique du HUD du chapitre 38.

La documentation de Flame est catégorique sur un point : le FPS rapporté par Flame « est la source de vérité sur le nombre de FPS auquel tourne votre jeu ». Préférez-le aux estimations que vous pourriez calculer vous-même.

### Compter les composants

`ChildCounterComponent` affiche, chaque seconde, le nombre d'enfants d'un type donné dans un composant cible.

```dart
if (kDebugMode) {
  await camera.viewport.add(
    ChildCounterComponent<Projectile>(
      target: world,
      position: Vector2(8, 28),
    ),
  );
}
```

Ce composant a une vertu diagnostique remarquable. Si ce nombre **monte sans jamais redescendre**, vous avez une fuite : des projectiles qui ne sont jamais retirés. C'est la cause numéro un des chutes de FPS progressives dans un jeu de tir. Vous verrez le problème en dix secondes au lieu de le chercher pendant deux jours.

### Mesurer un morceau de code

`TimeTrackComponent` chronomètre une portion de code et affiche la durée écoulée en microsecondes.

```dart
TimeTrackComponent.start('MyComponent.update');
// le code à mesurer
TimeTrackComponent.end('MyComponent.update');
```

Appliqué à une IA suspecte :

```dart
@override
void update(double dt) {
  super.update(dt);
  TimeTrackComponent.start('Boss.ia');
  mettreAJourIA(dt);
  TimeTrackComponent.end('Boss.ia');
}
```

### La barre d'outils Flame du DevTools

Le DevTools de Flutter propose, quand un jeu Flame tourne, un **onglet « Flame »** dédié. Il donne l'arbre des composants, des contrôles de lecture (mettre le jeu en pause, avancer) et l'inspection d'un composant sélectionné. C'est l'équivalent de l'inspecteur de widgets, mais pour l'arbre Flame. Ouvrez-le au moins une fois : voir son propre arbre de composants en vrai est très instructif.

---

## 42.14 — Les `print` et le logger

### Pourquoi `print` finit par poser problème

`print` est parfait pour cinq minutes de diagnostic. Il devient nuisible dès qu'il reste.

```text
  ┌────────────────────────────────────────────────────────────────┐
  │  Les quatre problèmes de print dans un jeu                     │
  ├────────────────────────────────────────────────────────────────┤
  │  1. Un print dans update() s'exécute 60 fois par seconde.      │
  │     À 200 composants, cela fait 12 000 lignes par seconde.     │
  │     La console sature et le jeu ralentit RÉELLEMENT.           │
  │                                                                │
  │  2. On ne peut pas l'éteindre sans le supprimer.               │
  │                                                                │
  │  3. Il reste dans la version publiée et écrit dans les logs    │
  │     système du téléphone, visibles par n'importe qui.          │
  │                                                                │
  │  4. Le linter officiel l'interdit (règle avoid_print).         │
  └────────────────────────────────────────────────────────────────┘
```

### Un logger minimal, sans dépendance

Vingt lignes suffisent pour résoudre les quatre problèmes.

```dart
// lib/core/journal.dart
import 'package:flutter/foundation.dart';

enum NiveauLog { debug, info, avertissement, erreur }

/// Journal minimal : filtrable, silencieux en release, sans dépendance.
class Journal {
  /// En dessous de ce niveau, rien n'est écrit.
  static NiveauLog seuil = NiveauLog.info;

  /// Coupe totalement le journal en release, sans avoir à y penser.
  static bool get actif => kDebugMode;

  static void _ecrire(NiveauLog niveau, String tag, String message) {
    if (!actif || niveau.index < seuil.index) return;
    final String etiquette = niveau.name.toUpperCase().padRight(13);
    debugPrint('[$etiquette] [$tag] $message');
  }

  static void debug(String tag, String message) =>
      _ecrire(NiveauLog.debug, tag, message);

  static void info(String tag, String message) =>
      _ecrire(NiveauLog.info, tag, message);

  static void avertissement(String tag, String message) =>
      _ecrire(NiveauLog.avertissement, tag, message);

  static void erreur(String tag, String message) =>
      _ecrire(NiveauLog.erreur, tag, message);
}
```

Usage :

```dart
Journal.info('Joueur', 'Saut déclenché, vitesse ${velocite.y}');
Journal.avertissement('Niveau', 'Caractère inconnu dans la carte : "z"');
Journal.erreur('Audio', 'Fichier introuvable : saut.wav');
```

**Résultat en debug :**

```text
[INFO         ] [Joueur] Saut déclenché, vitesse -520.0
[AVERTISSEMENT] [Niveau] Caractère inconnu dans la carte : "z"
[ERREUR       ] [Audio] Fichier introuvable : saut.wav
```

**Résultat en release :** rien du tout. Le compilateur voit `kDebugMode == false`, sait que `actif` vaut `false`, et supprime les appels.

### `debugPrint` plutôt que `print`

`debugPrint` vient de `package:flutter/foundation.dart`. Il **étrangle** la sortie : quand vous écrivez trop vite, il met en file d'attente au lieu de saturer le canal. Sur Android, `print` en rafale provoque des lignes tronquées ; `debugPrint` non.

### La règle absolue

**Jamais de log dans `update()` ou `render()` sans condition.**

Si vous devez absolument tracer quelque chose dans `update()`, espacez :

```dart
double _accumulateur = 0;

@override
void update(double dt) {
  super.update(dt);
  _accumulateur += dt;
  if (_accumulateur >= 1.0) {          // une ligne par seconde, pas 60
    _accumulateur = 0;
    Journal.debug('Joueur', 'position=$position velocite=$velocite');
  }
}
```

### Activer la règle du linter

```yaml
# analysis_options.yaml
include: package:flutter_lints/flutter.yaml

linter:
  rules:
    avoid_print: true
    prefer_const_constructors: true
    prefer_final_locals: true
```

Désormais, `flutter analyze` refuse tout `print` oublié, et votre CI aussi.

---

## 42.15 — Le débogueur de VS Code

Le `print` répond à la question « que vaut cette variable ? ». Le débogueur répond à « que se passe-t-il exactement, ligne par ligne ? ». C'est un outil différent, et beaucoup plus puissant.

### Poser un point d'arrêt

Cliquez dans la marge, à gauche du numéro de ligne. Un point rouge apparaît. Lancez le jeu avec **F5** (mode debug). Quand l'exécution atteint cette ligne, tout se fige.

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │  L'interface de débogage de VS Code                              │
  ├──────────────────────────────────────────────────────────────────┤
  │  VARIABLES     ← toutes les variables locales et this            │
  │  WATCH         ← des expressions que vous surveillez             │
  │  CALL STACK    ← qui a appelé qui pour arriver ici               │
  │  BREAKPOINTS   ← la liste de vos points d'arrêt                  │
  └──────────────────────────────────────────────────────────────────┘
```

### Les commandes de navigation

| Touche | Commande | Effet |
| --- | --- | --- |
| **F5** | Continue | Repart jusqu'au prochain point d'arrêt. |
| **F10** | Step Over | Exécute la ligne courante sans entrer dans les fonctions appelées. |
| **F11** | Step Into | Entre dans la fonction appelée. |
| **Maj + F11** | Step Out | Termine la fonction courante et remonte à l'appelant. |
| **Maj + F5** | Stop | Arrête la session. |

### Le point d'arrêt conditionnel : l'arme décisive dans un jeu

Un point d'arrêt normal dans `update()` est inutilisable : il se déclenche soixante fois par seconde et vous ne pouvez plus rien faire.

Faites un **clic droit** dans la marge, choisissez **Add Conditional Breakpoint**, et saisissez une expression Dart. L'exécution ne s'arrêtera que si elle est vraie.

| Condition | Ce qu'elle attrape |
| --- | --- |
| `pv <= 0` | L'instant précis de la mort. |
| `position.y > 2000` | Le moment où le joueur passe à travers le sol. |
| `velocite.y.abs() > 5000` | Une vitesse aberrante, symptôme d'un bug numérique. |
| `game.score > 1000 && !invincible` | Une combinaison d'état rare. |
| `dt > 0.1` | Une frame anormalement longue. |

Le dernier exemple est un bijou : il vous téléporte exactement dans la frame qui a produit le à-coup. C'est infiniment plus efficace que de deviner.

### Le logpoint : un `print` qui ne modifie pas le code

Clic droit dans la marge, **Add Logpoint**, puis un message contenant des expressions entre accolades :

```text
Joueur en {position.x}, {position.y} — PV {pv}
```

L'exécution ne s'arrête pas ; le message s'affiche dans la console de debug. Vous obtenez la trace d'un `print` sans jamais toucher au fichier source — donc sans risque d'en oublier un avant publication.

### Le hot reload et ses limites

**Ctrl + S** en mode debug recharge le code sans redémarrer le jeu. C'est ce qui rend le développement Flutter si agréable. Mais il faut connaître ses frontières :

| Situation | Hot reload suffit ? |
| --- | --- |
| Modifier le corps d'une méthode | Oui. |
| Changer une couleur, une constante `static const` | Non, il faut un hot **restart** (Maj + R). |
| Modifier `onLoad()` d'un composant déjà monté | Non : `onLoad` ne sera pas rappelé. |
| Ajouter un champ à une classe | Souvent non. |
| Modifier `main()` | Non. |
| Changer une dépendance dans `pubspec.yaml` | Non : arrêt complet et relance. |

Dans un jeu, l'état est vivant (position du joueur, ennemis en cours, score). Le hot reload le **conserve**, ce qui est parfois exactement ce qu'on veut — ajuster la couleur d'une explosion pendant qu'elle joue — et parfois trompeur : vous croyez tester un nouveau comportement, mais l'ancien état persiste. Au moindre doute, hot restart.

---

## 42.16 — Les erreurs les plus fréquentes en fin de projet

Voici les pannes qui surviennent précisément au moment où l'on croit avoir fini. Elles ne relèvent pas de l'algorithmique, mais de l'outillage et de la configuration — ce qui les rend d'autant plus irritantes.

| Symptôme | Cause probable | Correction |
| --- | --- | --- |
| `Unable to load asset: assets/images/joueur.png` | Le dossier n'est pas déclaré dans `pubspec.yaml`, ou l'indentation YAML est fausse. | Déclarer `assets:` sous `flutter:` avec exactement deux espaces d'indentation, puis `flutter pub get`. |
| L'asset est déclaré mais toujours introuvable | Flame préfixe automatiquement par `assets/images/`. Vous avez écrit le chemin complet. | Écrire `images.load('joueur.png')`, pas `images.load('assets/images/joueur.png')`. |
| Le jeu marche en debug, écran noir en release | Un asset chargé conditionnellement, ou une exception silencieusement avalée. | Tester en `flutter run --release` **avant** de compiler, et lire `flutter logs`. |
| `Bad state: No element` au lancement d'un niveau | `world.children.whereType<Joueur>().first` alors que le joueur n'est pas encore monté. | Utiliser `firstOrNull` et gérer le `null`, ou attendre `await game.ready()`. |
| `LateInitializationError: Field 'monde' has not been initialized` | Un `late final` lu avant `onLoad()`. | Initialiser dans `onLoad()` et ne jamais y accéder depuis un constructeur. |
| L'application plante au démarrage sur téléphone seulement | Chemin d'asset avec une majuscule différente. Android est sensible à la casse, pas Windows. | Tout nommer en minuscules, sans espace ni accent. |
| Le son ne se joue qu'une fois sur deux | Fichier non préchargé, ou format non supporté. | Précharger dans `onLoad()` avec `FlameAudio.audioCache.loadAll([...])` ; préférer le `.wav` court ou l'`.ogg`. |
| `flutter build apk` échoue : version de Gradle | Chaîne d'outils Android désynchronisée. | `flutter doctor -v` puis suivre les indications ; mettre à jour Android Studio. |
| `Execution failed for task ':app:lintVitalRelease'` | Une ressource Android manquante ou un manifeste invalide. | Lire l'erreur complète ; le plus souvent, une icône absente d'un dossier `mipmap-*`. |
| `flutter build web` réussit, page blanche sur le serveur | `--base-href` incorrect pour un sous-répertoire. | Voir la section 42.39. |
| Le jeu tourne à 20 FPS sur un téléphone récent | Compilé en debug. | `flutter run --release`, puis mesurer à nouveau. |
| `RangeError (index): Invalid value: Not in inclusive range` au chargement d'un niveau | Une ligne de la carte est plus courte que les autres. | Valider la carte au chargement : toutes les lignes doivent avoir la même longueur. |
| `setState() called after dispose()` | Un `Timer` ou un `Future` qui se termine après la fermeture d'un overlay. | Annuler les timers dans `dispose()` et tester `mounted` avant `setState`. |
| Le meilleur score revient à zéro après réinstallation | `SharedPreferences` est effacé avec l'application. C'est normal. | Documenter le comportement ; une sauvegarde cloud est un autre sujet. |

---

## 42.17 — Mesurer avant d'optimiser

Une règle, une seule, et elle est non négociable : **on ne modifie jamais du code au nom de la performance sans avoir mesuré avant.**

La raison est brutale : l'intuition d'un développeur sur la performance est mauvaise. Presque toujours. On soupçonne la boucle de collisions, et c'est un `TextPainter` reconstruit à chaque frame. On soupçonne le rendu, et c'est une image de 4096 pixels de côté chargée pour dessiner une pièce de 16 pixels.

Optimiser à l'aveugle produit trois dégâts :

```text
  1. Vous rendez le code plus compliqué                → plus de bugs
  2. Vous perdez des heures sur la mauvaise fonction   → le vrai problème reste
  3. Vous cassez un comportement qui marchait          → régression silencieuse
```

Le protocole correct tient en six étapes :

```text
  1. CONSTATER   « Le jeu saccade quand il y a beaucoup d'ennemis. »
  2. REPRODUIRE  Trouver la situation exacte, reproductible à volonté.
  3. MESURER     Mode profil + DevTools. Noter un chiffre : 34 ms par frame.
  4. LOCALISER   Lire la timeline : d'où viennent ces 34 ms ?
  5. CORRIGER    Une seule modification à la fois.
  6. REMESURER   34 ms → 11 ms ? On garde. Pas d'amélioration ? On annule.
```

L'étape 6 est celle qu'on saute, et c'est celle qui compte. Une « optimisation » sans gain mesuré est simplement du code moins lisible.

> **À retenir :** « Ça a l'air plus rapide » n'est pas une mesure. « 34 ms puis 11 ms sur la même scène » en est une.

---

## 42.18 — Le DevTools Flutter et le Performance Overlay

### Toujours en mode profil

La documentation officielle est explicite : profilez **sur un appareil physique**, **en mode profil**.

```bash
flutter run --profile
```

Pourquoi pas en debug ? Parce qu'en debug, le code Dart est interprété par la machine virtuelle en JIT, les assertions sont actives, et Flutter ajoute des vérifications partout. Un jeu peut être trois à cinq fois plus lent en debug. Mesurer en debug, c'est mesurer un autre programme que celui que jouera votre utilisateur.

Pourquoi pas en release ? Parce que le mode release supprime les informations de traçage dont le DevTools a besoin. Le mode **profil** est le compromis : les optimisations du release, plus l'instrumentation nécessaire à la mesure.

Dans VS Code, on configure cela dans `.vscode/launch.json` :

```text
"configurations": [
  {
    "name": "Flutter",
    "request": "launch",
    "type": "dart",
    "flutterMode": "profile"
  }
]
```

### Le Performance Overlay

C'est la mesure la plus rapide à obtenir : deux graphiques superposés au jeu, sans quitter l'application. Trois moyens de l'activer, tous documentés :

1. dans le DevTools, vue *Performance*, bouton **Performance Overlay** ;
2. depuis le terminal où tourne `flutter run`, en appuyant sur la touche **P** ;
3. par le code, avec `MaterialApp(showPerformanceOverlay: true, ...)`.

```dart
void main() {
  runApp(
    MaterialApp(
      debugShowCheckedModeBanner: false,
      showPerformanceOverlay: kDebugMode, // jamais en dur à true
      home: GameWidget<DonjonGame>.controlled(
        gameFactory: DonjonGame.new,
      ),
    ),
  );
}
```

### Lire les deux graphiques

L'overlay affiche les **300 dernières frames**.

```text
  ┌──────────────────────────────────────────────────────────┐
  │  ▁▂▁▁▂▁█▁▁▂▁▁▁▂▁▁█▁▁▁▂▁   GPU / RASTER  (graphe du HAUT) │
  │  ────────────────────────  ← ligne blanche = 16 ms       │
  ├──────────────────────────────────────────────────────────┤
  │  ▁▁▂▁▁▁▁▂▁▁▁▁▁▁▂▁▁▁▁▁▁▁▁   UI  (graphe du BAS)           │
  │  ────────────────────────  ← ligne blanche = 16 ms       │
  └──────────────────────────────────────────────────────────┘
```

| Graphe | Fil d'exécution | Ce qu'il mesure |
| --- | --- | --- |
| Haut | Raster (GPU) | Le temps de dessin de l'arbre de couches sur le GPU, via Skia ou Impeller. |
| Bas | UI | Le temps d'exécution de votre code Dart : `update()`, la logique, la construction de l'arbre de couches. |

Les **lignes blanches horizontales marquent des paliers de 16 ms**. La documentation le dit sans détour : « si le graphe dépasse une de ces lignes, vous tournez à moins de 60 Hz ».

Une **barre rouge** signale une frame qui a dépassé son budget et qui sera perdue, donc un à-coup visible.

### Le diagnostic en une question

| Où est le rouge ? | Ce que cela signifie | Où chercher |
| --- | --- | --- |
| Graphe du bas (UI) | Votre code Dart coûte trop cher. | `update()`, IA, collisions, allocations. |
| Graphe du haut (raster) | L'arbre de couches est simple à construire mais coûteux à dessiner. | `saveLayer`, opacités imbriquées, découpes, ombres, images non mises en cache. |
| Les deux | Souvent : vous mesurez en mode debug. | Repasser en `--profile`. |

C'est une question, une réponse, et elle divise par deux l'espace de recherche. Ne commencez jamais une session d'optimisation sans y avoir répondu.

---

## 42.19 — Lire un timeline de frames

L'overlay dit **qu'il y a** un problème. Le DevTools dit **où**.

Lancez le jeu en mode profil, ouvrez le DevTools (l'URL s'affiche dans le terminal, ou passez par VS Code), puis l'onglet **Performance**.

### Le Flutter Frames Chart

En haut, une barre par frame, en deux couleurs superposées : une part pour le fil UI, une part pour le fil raster.

```text
  ms
  40 ┤                      █
  32 ┤                      █
  24 ┤            █         █          █
  16 ┼────────────█─────────█──────────█─────────  budget 60 FPS
   8 ┤ ▄ ▄ ▄ ▄ ▄  █ ▄ ▄ ▄ ▄ █ ▄ ▄ ▄ ▄  █ ▄ ▄ ▄ ▄
   0 └─────────────────────────────────────────→ frames
          normal   pic     pic        pic
```

Ce que le DevTools signale automatiquement :

- une frame dont le temps dépasse le budget est surlignée en **rouge** : c'est du *jank* ;
- une frame qui a compilé un **shader** pour la première fois est marquée en **rouge sombre**. Ce cas mérite d'être connu : le premier passage d'un effet graphique peut coûter très cher, une seule fois, et donne l'impression d'un bug intermittent.

### Cliquer sur une frame

Cliquer sur une barre ouvre les onglets d'analyse :

| Onglet | Contenu |
| --- | --- |
| **Frame Analysis** | Des indications automatiques : opérations coûteuses détectées dans cette frame. |
| **Timeline Events** | Tous les événements tracés : construction de frame, dessin de scène, requêtes HTTP, ramasse-miettes, plus vos propres événements. |

Le *flame chart* des événements se lit de haut en bas : chaque barre est une fonction, les barres du dessous sont ses appels. La **largeur** est le temps. On cherche donc la barre large, puis on descend dedans jusqu'à trouver le coupable.

### Enhance Tracing

Un menu déroulant **Enhance Tracing** ajoute des événements plus fins :

| Option | Ce qu'elle ajoute |
| --- | --- |
| **Track Widget Builds** | Les appels à `build()`, avec le nom du widget. |
| **Track Layouts** | Les mises en page des objets de rendu. |
| **Track Paints** | Les peintures des objets de rendu. |

Ces options ralentissent l'application : elles servent à comprendre, pas à mesurer. Désactivez-les avant de relever un chiffre.

Trois interrupteurs complémentaires permettent de tester une hypothèse en une seconde : **Render Clip layers**, **Render Opacity layers**, **Render Physical Shape layers**. En les désactivant, on mesure exactement combien coûtent respectivement les découpes, les opacités et les ombres. Si le FPS remonte en désactivant les opacités, vous savez quoi corriger.

### Vos propres marqueurs

`dart:developer` permet d'ajouter vos événements dans la timeline :

```dart
import 'dart:developer' as developer;

@override
void update(double dt) {
  developer.Timeline.startSync('IA des ennemis');
  for (final Ennemi e in _ennemis) {
    e.mettreAJourIA(dt);
  }
  developer.Timeline.finishSync();

  super.update(dt);
}
```

Vous verrez désormais une barre nommée « IA des ennemis » dans le flame chart, avec sa largeur exacte. C'est la manière la plus directe de répondre à « combien coûte vraiment mon IA ? ».

---

## 42.20 — Le budget de 16 ms

Flutter vise **60 images par seconde**, et **120 sur les appareils qui en sont capables**. Une seconde contient 1000 millisecondes.

```text
  1000 ms / 60 images  = 16,67 ms par image
  1000 ms / 120 images =  8,33 ms par image
```

Ces 16,67 ms constituent votre **budget total** pour une frame. Et il faut être précis sur ce que ce budget contient :

```text
  ┌──────────────── UNE FRAME À 60 FPS : 16,67 ms ────────────────┐
  │                                                               │
  │  FIL UI (votre code Dart)                                     │
  │  ├─ update() de tous les composants                           │
  │  ├─ détection des collisions                                  │
  │  ├─ IA, timers, effets                                        │
  │  └─ construction de l'arbre de couches                        │
  │                                                               │
  │  FIL RASTER (le GPU)                                          │
  │  └─ dessin effectif de l'arbre de couches                     │
  │                                                               │
  │  Les deux fils travaillent en parallèle, mais CHACUN doit     │
  │  tenir dans 16,67 ms. Le plus lent des deux impose le FPS.    │
  └───────────────────────────────────────────────────────────────┘
```

Une conséquence importante : optimiser le fil UI ne sert à rien si c'est le fil raster qui déborde. D'où la question de la section 42.18.

### Ce que « dépasser le budget » veut dire

Le vocabulaire compte, parce qu'il oriente le diagnostic.

| Temps de frame | FPS effectif | Perception |
| --- | --- | --- |
| 8 ms | 120 | Très fluide. |
| 16 ms | 60 | Fluide. Objectif normal. |
| 20 ms | 50 | Légère irrégularité, perceptible sur un défilement. |
| 33 ms | 30 | Nettement moins fluide, mais jouable. |
| 50 ms | 20 | Désagréable. |
| 100 ms | 10 | Injouable. |

Et surtout : **une frame à 100 ms au milieu de frames à 8 ms est plus désagréable que 60 frames constantes à 20 ms.** L'œil humain déteste l'irrégularité plus que la lenteur. C'est pourquoi le DevTools met en évidence les pics et non la moyenne.

Le corollaire pratique est contre-intuitif : si vous ne pouvez pas tenir 60 FPS de manière stable, **il vaut mieux viser 30 FPS stables**. Un jeu régulier à 30 est plus agréable qu'un jeu qui oscille entre 60 et 25.

### Le budget dans votre code

Il devient utile de raisonner en microsecondes :

```text
  Budget total ................................ 16 670 µs
  ├─ Rendu (raster, hors de votre contrôle direct)
  └─ Votre code Dart : visez moins de 8 000 µs
      ├─ 200 composants × update() ....  ~2 000 µs
      ├─ détection de collisions ......  ~2 000 µs
      ├─ IA des ennemis ...............  ~1 000 µs
      ├─ effets et particules .........  ~1 500 µs
      └─ marge de sécurité ............  ~1 500 µs
```

Avec `TimeTrackComponent` (section 42.13), vous pouvez vérifier chacune de ces lignes réellement, sur votre jeu, sur votre appareil.

---

## 42.21 — Les causes classiques de chute de FPS dans un jeu Flame

Avant d'entrer dans les corrections, voici la carte du territoire. Dans un jeu Flame amateur, les chutes de FPS ont presque toujours l'une de ces sept causes.

| # | Cause | Signe caractéristique | Section |
| --- | --- | --- | --- |
| 1 | Trop de composants dans l'arbre | Le FPS baisse **progressivement** au fil de la partie. | 42.22 |
| 2 | Composants jamais retirés (fuite) | `ChildCounterComponent` monte sans jamais redescendre. | 42.23 |
| 3 | Allocations dans `update()` | Pics périodiques réguliers : c'est le ramasse-miettes. | 42.24 |
| 4 | Images trop grandes | Chute dès l'affichage d'un décor précis ; mémoire élevée. | 42.25 |
| 5 | `debugMode` laissé actif | Chute globale et constante, dès le lancement. | 42.26 |
| 6 | Mesure faite en mode debug | Le problème disparaît en `--release`. | 42.27 |
| 7 | Opacités, découpes et ombres empilées | Rouge sur le graphe **du haut** uniquement. | 42.18 |

Notez la colonne « signe caractéristique ». Elle vaut mieux qu'un long discours : la **forme** de la courbe de FPS désigne presque toujours la cause.

```text
  FPS
  60 ┤████████▇▇▇▆▆▆▅▅▅▄▄▄        cause 1 ou 2 : ça se dégrade
     └──────────────────────→     avec le temps

  FPS
  60 ┤███▁███▁███▁███▁███▁        cause 3 : pics réguliers
     └──────────────────────→     (ramasse-miettes)

  FPS
  30 ┤██████████████████████      cause 5 ou 6 : bas et plat
     └──────────────────────→     dès la première seconde

  FPS
  60 ┤██████▁▁▁▁▁▁██████████      cause 4 ou 7 : lié à une
     └──────────────────────→     zone ou un effet précis
```

---

## 42.22 — Trop de composants : le culling et le pooling

Chaque composant de l'arbre coûte, à chaque frame, un appel à `update()` et un appel à `render()`. Ce coût unitaire est faible ; multiplié par 3 000, il ne l'est plus.

Deux stratégies complémentaires :

- le **culling** : ne pas dessiner ce qui n'est pas visible ;
- le **pooling** : ne pas créer ni détruire, mais réutiliser (section 42.23).

### Le culling

Un niveau du donjon fait 100 × 30 tuiles, soit 3 000 tuiles. La caméra, au zoom 2, en montre environ 20 × 12, soit 240. **On dessine donc 3 000 objets pour en montrer 240.**

Flame fournit exactement l'outil nécessaire sur `CameraComponent` :

```dart
Rect get visibleWorldRect;
bool canSee(PositionComponent component, {World? componentWorld});
```

La correction la plus simple consiste à ne pas dessiner un composant que la caméra ne voit pas :

```dart
class Plateforme extends PositionComponent with HasGameReference<DonjonGame> {
  @override
  void renderTree(Canvas canvas) {
    if (!game.camera.canSee(this)) return; // hors champ : on ne dessine rien
    super.renderTree(canvas);
  }
}
```

Une variante sans appel par composant, pour les décors très nombreux, consiste à comparer directement avec le rectangle visible :

```dart
class Decor extends PositionComponent with HasGameReference<DonjonGame> {
  @override
  void renderTree(Canvas canvas) {
    final Rect vue = game.camera.visibleWorldRect;
    final Rect moi = toAbsoluteRect();
    if (!vue.overlaps(moi)) return;
    super.renderTree(canvas);
  }
}
```

### Ce que le culling ne fait pas

Il évite le **rendu**, pas la **mise à jour**. Les 3 000 tuiles continuent de recevoir `update()`. Pour une plateforme statique, c'est sans importance : son `update()` ne fait rien. Pour un ennemi, ce serait différent — et il ne faut surtout pas suspendre l'IA d'un ennemi hors champ sans y réfléchir, sinon le monde se fige dès qu'on tourne le dos.

Le compromis raisonnable :

| Type d'objet | Culling du rendu | Suspension de la logique |
| --- | --- | --- |
| Décor, plateformes, murs | Oui | Sans objet (pas de logique). |
| Collectibles | Oui | Non (l'animation doit continuer). |
| Ennemis proches | Oui | Non. |
| Ennemis très éloignés (plus de deux écrans) | Oui | Oui, avec prudence. |
| Particules | Oui | Non (elles doivent finir leur vie et se retirer). |

### Mesurer le gain

```dart
if (kDebugMode) {
  await camera.viewport.addAll(<Component>[
    FpsTextComponent(position: Vector2(8, 8)),
    ChildCounterComponent<PositionComponent>(target: world, position: Vector2(8, 28)),
  ]);
}
```

Avant / après, sur la même scène, même appareil, mode profil. Si le FPS ne bouge pas, le culling n'était pas votre problème : annulez la modification et retournez mesurer.

---

## 42.23 — L'object pooling des projectiles et des particules

### Le problème

```dart
// À chaque tir, dans un jeu qui tire vite :
final Projectile p = Projectile(position: position.clone());
world.add(p);
// ... puis, à l'impact :
p.removeFromParent();
```

À dix tirs par seconde, cela fait dix objets créés et dix objets abandonnés chaque seconde. Le ramasse-miettes de Dart doit les collecter, et cette collecte se produit **au milieu d'une frame**, produisant un pic. C'est exactement la signature « pics réguliers » de la section 42.21.

### Le principe de la pool

On ne crée jamais, on ne détruit jamais. On **emprunte** et on **rend**.

```text
  ┌─────────────── LA POOL ───────────────┐
  │  [libre][libre][libre][libre][libre]  │  5 projectiles préalloués
  └───────────────────────────────────────┘
            │ obtenir()
            ▼
  ┌─────────────── LA POOL ───────────────┐
  │  [ACTIF][libre][libre][libre][libre]  │
  └───────────────────────────────────────┘
            │ rendre()  (à l'impact)
            ▼
  ┌─────────────── LA POOL ───────────────┐
  │  [libre][libre][libre][libre][libre]  │  aucune allocation
  └───────────────────────────────────────┘
```

### L'implémentation

```dart
// lib/core/pool.dart
import 'package:flame/components.dart';

/// Réserve d'objets réutilisables.
/// [creer] fabrique un nouvel élément, [reinitialiser] le remet à neuf.
class Pool<T> {
  Pool({
    required T Function() creer,
    required void Function(T) reinitialiser,
    int tailleInitiale = 0,
  })  : _creer = creer,
        _reinitialiser = reinitialiser {
    for (int i = 0; i < tailleInitiale; i++) {
      _libres.add(_creer());
    }
  }

  final T Function() _creer;
  final void Function(T) _reinitialiser;
  final List<T> _libres = <T>[];

  int get disponibles => _libres.length;

  /// Emprunte un objet : on le reprend dans la réserve, ou on en crée un.
  T obtenir() => _libres.isEmpty ? _creer() : _libres.removeLast();

  /// Rend un objet à la réserve après l'avoir remis à neuf.
  void rendre(T objet) {
    _reinitialiser(objet);
    _libres.add(objet);
  }
}
```

### Appliquée aux projectiles du donjon

```dart
// dans DonjonGame
late final Pool<Projectile> poolProjectiles;

@override
Future<void> onLoad() async {
  await super.onLoad();

  poolProjectiles = Pool<Projectile>(
    creer: Projectile.new,
    reinitialiser: (Projectile p) {
      p.velocite.setZero();
      p.position.setZero();
      p.actif = false;
    },
    tailleInitiale: 32,
  );
}

/// Tire un projectile depuis [depart] dans la direction [direction].
Future<void> tirer(Vector2 depart, Vector2 direction) async {
  final Projectile p = poolProjectiles.obtenir();
  p.position.setFrom(depart);
  p.velocite.setFrom(direction)..scale(400);
  p.actif = true;

  if (p.parent == null) {
    await world.add(p);
  }
}
```

Côté projectile, on ne se retire plus de l'arbre : on se désactive et on retourne dans la réserve.

```dart
class Projectile extends PositionComponent
    with HasGameReference<DonjonGame>, CollisionCallbacks {
  final Vector2 velocite = Vector2.zero();
  bool actif = false;

  @override
  void update(double dt) {
    if (!actif) return;                       // dormant : coût quasi nul
    super.update(dt);
    position.addScaled(velocite, dt);
    if (!game.camera.canSee(this)) {
      _recycler();
    }
  }

  @override
  void render(Canvas canvas) {
    if (!actif) return;                       // dormant : on ne dessine rien
    super.render(canvas);
    canvas.drawCircle(Offset(size.x / 2, size.y / 2), 4, _peinture);
  }

  void _recycler() {
    actif = false;
    game.poolProjectiles.rendre(this);
  }
}
```

### Le tableau de décision

Le pooling ajoute de la complexité. Il ne se justifie pas partout.

| Objet | Pooling utile ? | Pourquoi |
| --- | --- | --- |
| Projectiles | Oui | Créés et détruits en rafale. |
| Particules | Oui | Le pire cas : des dizaines par explosion. |
| Textes de dégâts flottants | Oui | Un par coup porté. |
| Ennemis | Rarement | Une dizaine par niveau, créés une fois. |
| Plateformes | Non | Créées au chargement du niveau, jamais détruites. |
| Le joueur | Non | Il y en a un. |

> **Piège classique du pooling :** oublier de tout réinitialiser. Un projectile recyclé qui garde son ancienne vélocité part dans la mauvaise direction, et le bug est incompréhensible parce qu'il ne se produit qu'après plusieurs tirs. Écrivez la fonction `reinitialiser` en listant **tous** les champs mutables de la classe, sans exception.

---

## 42.24 — Les allocations dans `update()` : le piège du `Vector2` créé à chaque frame

C'est l'erreur de performance la plus répandue chez ceux qui découvrent Flame, et elle est invisible à la lecture.

```dart
// LENT — trois objets créés à chaque frame, pour chaque composant
@override
void update(double dt) {
  super.update(dt);
  position = position + velocite * dt;   // 2 Vector2 créés ici
  final Vector2 gravite = Vector2(0, Constantes.gravite * dt); // 1 de plus
  velocite = velocite + gravite;         // encore 1
}
```

Faisons le calcul. Quatre `Vector2` par frame, 200 composants, 60 frames par seconde :

```text
  4 × 200 × 60 = 48 000 objets créés PAR SECONDE
```

Chacun est abandonné aussitôt. Le ramasse-miettes finit par se déclencher, prend quelques millisecondes, et vous obtenez un pic. Régulier. Inexplicable si on ne connaît pas la cause.

### La version rapide

`vector_math`, la bibliothèque derrière `Vector2`, fournit des opérations **en place** qui ne créent rien.

```dart
// RAPIDE — zéro allocation
@override
void update(double dt) {
  super.update(dt);
  velocite.y += Constantes.gravite * dt;          // scalaire, pas d'objet
  velocite.y = velocite.y.clamp(-9999, Constantes.vitesseMaxChute);
  position.addScaled(velocite, dt);               // en place
}
```

### Le vocabulaire à connaître

| À éviter dans `update()` | Équivalent en place |
| --- | --- |
| `a = a + b` | `a.add(b)` |
| `a = a - b` | `a.sub(b)` |
| `a = a * k` | `a.scale(k)` |
| `a = a + b * k` | `a.addScaled(b, k)` |
| `a = b.clone()` | `a.setFrom(b)` |
| `a = Vector2(x, y)` | `a.setValues(x, y)` |
| `a = Vector2.zero()` | `a.setZero()` |
| `v.normalized()` | `v.normalize()` |

### Les autres allocations cachées

Le `Vector2` n'est pas seul en cause. Tout ce qui suit crée un objet à chaque appel :

```dart
// LENT
@override
void render(Canvas canvas) {
  final Paint p = Paint()..color = const Color(0xFFFFD700); // objet par frame
  canvas.drawCircle(Offset(8, 8), 8, p);
}

// RAPIDE : la peinture est créée une fois pour toutes
static final Paint _peinture = Paint()..color = const Color(0xFFFFD700);

@override
void render(Canvas canvas) {
  canvas.drawCircle(const Offset(8, 8), 8, _peinture);
}
```

| Coupable | Correction |
| --- | --- |
| `Paint()` dans `render()` | Champ `static final`. |
| `TextPainter` reconstruit dans `render()` | Ne le reconstruire que si le texte change (chapitre 38). |
| `'Score : $score'` dans `render()` | Ne recalculer la chaîne que quand le score change. |
| `children.whereType<Ennemi>().toList()` dans `update()` | Garder une liste mise à jour à l'ajout et au retrait. |
| `List<...>[]` littéral dans `update()` | Réutiliser une liste champ de la classe, avec `clear()`. |
| `Rect.fromLTWH(...)` dans `render()` | Le mettre en cache si les dimensions ne changent pas. |

### Repérer le problème dans le DevTools

L'onglet **Memory** du DevTools montre la courbe d'occupation mémoire. La signature est caractéristique :

```text
  Mémoire
    │       ╱│      ╱│      ╱│      ╱│
    │     ╱  │    ╱  │    ╱  │    ╱  │     ← dents de scie
    │   ╱    │  ╱    │  ╱    │  ╱    │
    │ ╱      │╱      │╱      │╱      │
    └──────────────────────────────────→ temps
       chaque chute verticale = un passage du ramasse-miettes
```

Des dents de scie serrées signifient beaucoup d'allocations éphémères. C'est exactement ce que la présente section vous apprend à supprimer.

> **Nuance importante :** hors de `update()` et de `render()`, allouer est parfaitement normal et lisible. `Vector2(100, 200)` dans `onLoad()` s'exécute une fois : personne ne s'en plaindra. La règle ne concerne que le code appelé soixante fois par seconde.

---

## 42.25 — Les images trop grandes et les atlas

### Le coût réel d'une image

Une image en mémoire n'occupe pas la taille de son fichier PNG. Elle occupe **4 octets par pixel** une fois décodée.

```text
  Image 4096 × 4096  →  4096 × 4096 × 4  = 67 108 864 octets ≈ 64 Mo
  Image 1024 × 1024  →  1024 × 1024 × 4  =  4 194 304 octets ≈  4 Mo
  Image  256 ×  256  →   256 ×  256 × 4  =    262 144 octets ≈ 256 Ko
```

Un PNG de 800 Ko sur le disque peut donc peser 64 Mo en mémoire. Sur un téléphone d'entrée de gamme, quelques images de ce genre suffisent à faire fermer l'application par le système.

**Règle simple : une image ne doit jamais être plus grande que ce qui est affiché.** Un sprite de joueur affiché en 32 × 32 pixels sur un écran ne gagne rien à être stocké en 512 × 512. Sur un écran à haute densité, prévoyez au maximum deux ou trois fois la taille affichée.

### Le problème des appels de dessin

Chaque `drawImage` avec une texture différente oblige le GPU à changer de texture. Ce changement est le vrai coût.

```text
  SANS ATLAS : 8 fichiers = 8 textures = 8 changements par frame
  joueur.png  gobelin.png  piece.png  potion.png  cle.png  ...

  AVEC ATLAS : 1 fichier = 1 texture = 1 seul changement
  ┌──────────────────────────────────┐
  │ [joueur] [gobelin] [piece] [cle] │
  │ [potion] [porte]   [boss]  [mur] │   atlas.png
  └──────────────────────────────────┘
```

Un **atlas** (ou *sprite sheet*, chapitre 22) regroupe tous les sprites dans une seule image. On découpe ensuite avec `Sprite(image, srcPosition:, srcSize:)`, exactement comme au chapitre 29.

### Précharger, toujours

```dart
@override
Future<void> onLoad() async {
  await super.onLoad();
  await images.loadAll(<String>[
    'atlas.png',
    'tuiles.png',
  ]);
}
```

Charger une image **pendant** le jeu provoque une frame très longue, au pire moment : l'apparition du boss. Tout ce dont un niveau a besoin doit être chargé avant qu'il ne commence, pendant l'écran de chargement prévu à cet effet (`Overlays.chargement`, chapitre 35).

### La liste de contrôle des images

| Vérification | Cible |
| --- | --- |
| Taille des images | Jamais plus de 2 à 3 fois la taille affichée. |
| Format | PNG pour les sprites avec transparence. |
| Nombre de fichiers | Regroupés en un ou deux atlas. |
| Dimensions de l'atlas | Rester en dessous de 2048 × 2048 pour la compatibilité. |
| Chargement | Tout dans `onLoad()`, jamais pendant le jeu. |
| Assets inutilisés | Supprimés du dossier `assets/` avant publication. |

Cette dernière ligne n'est pas anodine : Flutter embarque **tout** ce que déclare le `pubspec.yaml`. Trente images de test oubliées grossissent votre APK de plusieurs mégaoctets, pour rien.

---

## 42.26 — `debugMode` laissé actif en production

C'est l'erreur la plus bête de tout ce chapitre, et l'une des plus fréquentes.

`debugMode = true` sur un `FlameGame` fait dessiner, pour **chaque** `PositionComponent` de l'arbre, un rectangle englobant et un texte de position. Le texte est le point critique : rendre du texte est une opération coûteuse, et Flame en produit un par composant, à chaque frame.

```text
  200 composants × (1 rectangle + 1 rendu de texte) × 60 frames/s
  = 24 000 rendus de texte par seconde
```

Un jeu parfaitement optimisé peut ainsi tomber à 15 FPS pour cette seule raison.

### La protection en trois niveaux

**Niveau 1 — `kDebugMode`, la meilleure solution.**

```dart
import 'package:flutter/foundation.dart';

class DonjonGame extends FlameGame {
  DonjonGame() {
    debugMode = kDebugMode; // faux en release, garanti par le compilateur
  }
}
```

`kDebugMode` est une constante de compilation. En release, la condition est éliminée avec le code qu'elle protège. On ne peut pas l'oublier.

**Niveau 2 — un interrupteur unique et explicite.**

```dart
// lib/config/constantes.dart
static const bool afficherDebug = false;
```

Une seule ligne à vérifier avant publication, au lieu de vingt fichiers.

**Niveau 3 — un test automatique.**

```dart
// test/config/constantes_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:donjon_de_dart/config/constantes.dart';

void main() {
  test('le mode debug visuel est désactivé', () {
    expect(Constantes.afficherDebug, isFalse,
        reason: 'afficherDebug doit valoir false avant toute publication');
  });
}
```

Votre CI refusera désormais une version où le debug est resté allumé. C'est le genre de test qui semble ridicule, jusqu'au jour où il vous sauve.

### Le même raisonnement s'applique à

| Élément | À désactiver avant publication |
| --- | --- |
| `debugMode` de Flame | Oui. |
| `showPerformanceOverlay` | Oui. |
| `debugShowCheckedModeBanner` | Le mettre à `false` (c'est une bannière, pas un coût). |
| `FpsTextComponent`, `ChildCounterComponent` | Oui, sous `kDebugMode`. |
| Menus de triche, sauts de niveau | Oui. |
| Journaux (`Journal`) | Automatique avec `kDebugMode`. |

---

## 42.27 — Le mode release change tout : `flutter run --release`

Il faut le dire clairement, parce que beaucoup d'élèves perdent des journées entières sur un problème qui n'existe pas : **les mesures faites en mode debug ne veulent rien dire.**

### Les trois modes

| Mode | Compilation | Assertions | Hot reload | Usage |
| --- | --- | --- | --- | --- |
| **debug** | JIT | Actives | Oui | Développement quotidien. |
| **profile** | AOT | Inactives | Non | Mesure de performance. |
| **release** | AOT | Inactives | Non | Publication. |

En debug, votre code Dart est compilé à la volée, les assertions s'exécutent, le framework ajoute des vérifications de cohérence, les images ne sont pas optimisées. Un jeu peut être **trois à cinq fois plus lent** qu'en release. Ce n'est pas une anomalie : c'est le prix du confort de développement.

### Les commandes

```bash
flutter run --debug     # par défaut
flutter run --profile   # pour mesurer
flutter run --release   # ce que verra le joueur
```

### Le réflexe à acquérir

Avant de vous inquiéter d'un FPS bas, faites toujours ceci :

```bash
flutter run --release -d <votre_appareil>
```

Dans un très grand nombre de cas, le problème disparaît. Vous venez d'économiser une journée d'optimisation inutile.

### Ce que le mode release change aussi

Le tree shaking retire le code jamais appelé. Les `assert` disparaissent — ce qui veut dire qu'un comportement reposant sur un effet de bord dans un `assert` cassera en release. Les icônes non utilisées sont retirées des polices. Le code est compilé en avance (AOT), donc plus rapide à démarrer et à exécuter.

> **À retenir :** développez en debug, mesurez en profil, publiez en release. Ne mélangez jamais.

---

## 42.28 — La consommation batterie et la limitation du framerate

Sur un téléphone, la performance n'est pas seulement une question de fluidité : c'est une question d'**autonomie** et de **chaleur**. Un jeu qui vide la batterie en quarante minutes ou qui fait chauffer l'appareil sera désinstallé, même s'il est bon.

### D'où vient la consommation

```text
  ┌────────────────────────────────────────────────────┐
  │  Ce qui consomme dans un jeu mobile                │
  ├────────────────────────────────────────────────────┤
  │  L'écran ........................  souvent le 1er  │
  │  Le GPU (rendu par frame) ......   proportionnel   │
  │                                    au FPS visé     │
  │  Le CPU (votre code Dart) ......   proportionnel   │
  │                                    à la charge     │
  │  L'audio .......................   modeste         │
  │  Le réseau (si présent) ........   très coûteux    │
  └────────────────────────────────────────────────────┘
```

Deux leviers vous appartiennent : le nombre de frames calculées, et le travail fait dans chacune.

### Ne rien calculer quand rien ne bouge

C'est la mesure la plus efficace, et la plus simple. Un jeu en pause, un menu affiché, un écran de Game Over : rien ne doit tourner.

```dart
void changerEtat(GameState nouvelEtat) {
  etat = nouvelEtat;
  switch (nouvelEtat) {
    case GameState.pause:
    case GameState.gameOver:
    case GameState.victoire:
    case GameState.menu:
      pauseEngine();   // la boucle s'arrête : consommation quasi nulle
    case GameState.enJeu:
      resumeEngine();
    case GameState.chargement:
      break;
  }
}
```

`pauseEngine()` arrête la boucle de jeu. Un menu affiché consomme alors autant qu'une page statique.

### Réagir au passage en arrière-plan

Un joueur qui reçoit un appel doit retrouver son jeu tel quel, et pendant ce temps le jeu ne doit rien consommer. Flutter signale ces transitions par le cycle de vie de l'application.

```dart
import 'package:flutter/widgets.dart';

class _EcranJeuState extends State<EcranJeu> with WidgetsBindingObserver {
  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance.addObserver(this);
  }

  @override
  void dispose() {
    WidgetsBinding.instance.removeObserver(this);
    super.dispose();
  }

  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    if (state != AppLifecycleState.resumed) {
      widget.game.changerEtat(GameState.pause);
    }
  }
}
```

C'est une exigence de qualité autant qu'une économie : un jeu qui continue de tourner en arrière-plan est un jeu qui vide la batterie sans que le joueur comprenne pourquoi.

### Faut-il limiter à 30 FPS ?

Certains jeux mobiles proposent une option « 30 FPS » pour économiser la batterie. C'est un choix défendable pour un jeu lent, contestable pour un jeu d'action où la réactivité compte.

Trois points de méthode :

1. si vous limitez, faites-en une **option**, jamais un choix imposé ;
2. votre code doit déjà être **indépendant du framerate** grâce au `dt` (chapitre 20) : passer de 60 à 30 FPS ne doit rien changer à la vitesse du joueur ;
3. le rendu à taux réduit se règle plus proprement au niveau de la plateforme qu'en bricolant dans la boucle. Avant d'écrire du code, vérifiez ce que propose la version de Flutter que vous utilisez.

### La liste de contrôle « batterie »

| Point | Objectif |
| --- | --- |
| `pauseEngine()` en pause et dans les menus | Aucun calcul quand rien ne bouge. |
| Pause automatique en arrière-plan | Aucun calcul écran éteint. |
| Musique arrêtée en pause | Le décodage audio consomme. |
| Zéro allocation dans `update()` | Moins de ramasse-miettes, moins de CPU. |
| Culling actif | Moins de travail GPU. |
| Aucune animation dans les menus statiques | Un menu ne doit pas redessiner en continu. |
| Aucun réseau si le jeu est hors ligne | Le plus gros consommateur. |

---

## 42.29 — Avant de publier : la liste de vérification

Publier n'est pas une commande, c'est une procédure. Voici celle à dérouler intégralement, dans l'ordre, avant chaque livraison.

```text
  ┌─── CODE ──────────────────────────────────────────────────────┐
  │  □ flutter analyze          → aucune erreur, aucun warning    │
  │  □ dart format .            → tout est formaté                │
  │  □ flutter test             → tous les tests passent          │
  │  □ Aucun print              → règle avoid_print active        │
  │  □ Aucun TODO bloquant      → recherche « TODO » dans lib/    │
  ├─── CONFIGURATION ─────────────────────────────────────────────┤
  │  □ debugMode = kDebugMode   → pas de true en dur              │
  │  □ showPerformanceOverlay   → false ou kDebugMode             │
  │  □ debugShowCheckedModeBanner: false                          │
  │  □ Menus de triche retirés                                    │
  │  □ version: dans pubspec.yaml incrémentée                     │
  ├─── IDENTITÉ ──────────────────────────────────────────────────┤
  │  □ applicationId défini (pas com.example.*)                   │
  │  □ Nom de l'application correct                               │
  │  □ Icône en place, toutes densités                            │
  │  □ Écran de démarrage cohérent                                │
  ├─── ASSETS ────────────────────────────────────────────────────┤
  │  □ Assets inutilisés supprimés                                │
  │  □ Licences des assets vérifiées et créditées                 │
  │  □ Images redimensionnées                                     │
  ├─── TESTS RÉELS ───────────────────────────────────────────────┤
  │  □ Testé en --release sur un appareil physique                │
  │  □ Testé sur un petit écran et un grand écran                 │
  │  □ Testé le retour arrière, l'appel entrant, la rotation      │
  │  □ Partie complète terminée sans plantage                     │
  │  □ Sauvegarde vérifiée après fermeture/réouverture            │
  └───────────────────────────────────────────────────────────────┘
```

La ligne la plus souvent négligée est celle des **licences d'assets**. Un sprite trouvé sur Internet n'est pas libre de droits parce qu'il était téléchargeable. Kenney.nl, OpenGameArt et itch.io indiquent la licence de chaque lot : lisez-la, respectez l'attribution demandée, et gardez la trace de la source dans un fichier `CREDITS.md`.

---

## 42.30 — L'icône et le nom de l'application

### Le nom affiché

**Android** — dans `android/app/src/main/AndroidManifest.xml` :

```text
<application
    android:label="Donjon de Dart"
    android:icon="@mipmap/ic_launcher">
```

**Web** — dans `web/index.html` (balise `<title>`) et dans `web/manifest.json` (champs `name` et `short_name`).

Attention : `name:` dans `pubspec.yaml` est le nom **technique** du paquet Dart (minuscules, sans espace ni accent). Il n'a rien à voir avec le nom affiché sur le téléphone.

### L'icône

Android attend l'icône dans les dossiers `android/app/src/main/res/mipmap-*`, un par densité d'écran :

```text
  android/app/src/main/res/
  ├── mipmap-mdpi/ic_launcher.png       48 × 48
  ├── mipmap-hdpi/ic_launcher.png       72 × 72
  ├── mipmap-xhdpi/ic_launcher.png      96 × 96
  ├── mipmap-xxhdpi/ic_launcher.png    144 × 144
  └── mipmap-xxxhdpi/ic_launcher.png   192 × 192
```

Les produire à la main est fastidieux. Le package `flutter_launcher_icons` le fait à partir d'une seule image de 1024 × 1024 :

```yaml
dev_dependencies:
  flutter_launcher_icons: ^0.14.1

flutter_launcher_icons:
  android: true
  ios: true
  image_path: "assets/icone/icone.png"
  web:
    generate: true
  windows:
    generate: true
```

```bash
dart run flutter_launcher_icons
```

Tous les dossiers `mipmap-*` sont remplis, ainsi que les icônes Web et Windows.

Conseils de conception : un dessin **simple et lisible en 48 pixels**, pas de texte, un fort contraste, et une marge de sécurité car Android peut rogner les coins selon le lanceur utilisé.

---

## 42.31 — L'écran de démarrage

Entre le moment où l'utilisateur touche l'icône et celui où Flutter est prêt, l'écran est vide. L'**écran de démarrage** (*splash screen*) remplit cet intervalle. Il ne se code pas en Flutter : il est affiché par Android avant que Flutter n'existe.

Le package `flutter_native_splash` le génère pour toutes les plateformes :

```yaml
dev_dependencies:
  flutter_native_splash: ^2.4.1

flutter_native_splash:
  color: "#101018"
  image: assets/icone/splash.png
  android_12:
    color: "#101018"
    image: assets/icone/splash_android12.png
  web: true
```

```bash
dart run flutter_native_splash:create
```

Trois règles : il doit être **court** (moins d'une seconde perçue), **cohérent** avec la première image du jeu — même couleur de fond que votre menu, pour éviter un clignotement — et il ne doit **rien annoncer d'utile**, car il peut être invisible sur un appareil rapide.

Android 12 et suivants imposent leur propre format d'écran de démarrage, d'où la section `android_12` séparée.

---

## 42.32 — Le build Android : `flutter build apk` et `flutter build appbundle`

Deux formats, deux usages. C'est la première chose à comprendre.

| Format | Commande | Pour qui |
| --- | --- | --- |
| **APK** | `flutter build apk` | Installation directe, tests, distribution hors magasin, itch.io. |
| **App Bundle (.aab)** | `flutter build appbundle` | Le Google Play Store. C'est le format **recommandé** et exigé. |

### L'App Bundle

```bash
flutter build appbundle
```

**Résultat :**

```text
Running Gradle task 'bundleRelease'...                             62,3s
✓ Built build/app/outputs/bundle/release/app.aab (18.4MB)
```

Un `.aab` n'est pas installable directement sur un téléphone. C'est un paquet que Google Play recompile en APK adaptés à chaque appareil : le joueur ne télécharge que ce qui le concerne (son architecture processeur, sa densité d'écran, sa langue). L'installation est donc plus légère.

### L'APK

```bash
flutter build apk
```

**Résultat :**

```text
✓ Built build/app/outputs/flutter-apk/app-release.apk (22.1MB)
```

Cet APK contient toutes les architectures : il est lourd, mais il s'installe partout.

### L'APK découpé par architecture

```bash
flutter build apk --split-per-abi
```

**Résultat :**

```text
✓ Built build/app/outputs/flutter-apk/app-armeabi-v7a-release.apk (8.2MB)
✓ Built build/app/outputs/flutter-apk/app-arm64-v8a-release.apk (8.7MB)
✓ Built build/app/outputs/flutter-apk/app-x86_64-release.apk (8.9MB)
```

Trois fichiers, trois fois plus légers. `arm64-v8a` couvre la quasi-totalité des téléphones actuels ; `armeabi-v7a` sert aux appareils plus anciens ; `x86_64` sert surtout aux émulateurs.

### Installer sur un appareil branché

```bash
flutter install
```

### Les options utiles

```bash
# Fixer la version sans toucher au pubspec.yaml
flutter build appbundle --build-name=1.2.0 --build-number=7

# Rendre le code Dart difficile à lire, en conservant les symboles à part
flutter build appbundle --obfuscate --split-debug-info=build/app/outputs/symbols
```

L'obfuscation renomme les classes et les méthodes dans le binaire. Les traces d'erreur deviennent illisibles : c'est précisément pourquoi `--split-debug-info` est obligatoire avec `--obfuscate`. Le dossier de symboles produit permet de reconstituer une trace lisible avec `flutter symbolize`. **Conservez ce dossier pour chaque version publiée**, sinon vous ne pourrez plus interpréter les rapports de plantage de vos joueurs.

### Vérifier le contenu d'un `.aab` avant publication

L'outil `bundletool`, fourni par Google, permet de transformer un `.aab` en APK installables et de les poser sur un appareil :

```bash
bundletool build-apks --bundle=app.aab --output=app.apks
bundletool install-apks --apks=app.apks
```

C'est la seule manière de tester exactement ce que vous allez envoyer au magasin.

---

## 42.33 — La signature d'une application Android (keystore)

Android exige que toute application soit **signée** cryptographiquement. La signature prouve que la mise à jour vient bien du même auteur que la version précédente.

En debug, Flutter signe automatiquement avec une clé de développement. Cette clé n'est acceptée par aucun magasin. Pour publier, il faut la vôtre.

### Créer le keystore

```bash
keytool -genkey -v -keystore ~/upload-keystore.jks -keyalg RSA \
        -storetype JKS -keysize 2048 -validity 10000 -alias upload
```

Sous Windows, en PowerShell :

```bash
keytool -genkey -v -keystore $env:USERPROFILE\upload-keystore.jks `
        -storetype JKS -keyalg RSA -keysize 2048 -validity 10000 `
        -alias upload
```

`keytool` est fourni avec le JDK, lui-même installé avec Android Studio. La commande pose des questions (nom, organisation, pays) et demande deux mots de passe.

> **Avertissement, à lire deux fois :** ce fichier et ces mots de passe sont **irremplaçables**. Si vous les perdez, vous ne pourrez plus jamais publier de mise à jour de votre application sous la même identité. Sauvegardez-les hors du dépôt Git, dans un gestionnaire de mots de passe ou un stockage chiffré.

### Déclarer le keystore

Créez `android/key.properties` :

```text
storePassword=<mot de passe du magasin>
keyPassword=<mot de passe de la clé>
keyAlias=upload
storeFile=/Users/moi/upload-keystore.jks
```

Sous Windows, doublez les antislashs : `C:\\Users\\moi\\upload-keystore.jks`.

**Puis, immédiatement, ajoutez ce fichier à `.gitignore` :**

```text
# .gitignore
android/key.properties
*.jks
*.keystore
```

Committer un keystore et ses mots de passe dans un dépôt public est un incident de sécurité définitif : même supprimé plus tard, le fichier reste dans l'historique Git.

### Configurer Gradle

Dans `android/app/build.gradle` (variante Groovy) :

```groovy
import java.util.Properties
import java.io.FileInputStream

plugins {
    // ...
}

def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
    // ...

    signingConfigs {
        release {
            keyAlias = keystoreProperties['keyAlias']
            keyPassword = keystoreProperties['keyPassword']
            storeFile = keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword = keystoreProperties['storePassword']
        }
    }
    buildTypes {
        release {
            signingConfig = signingConfigs.release
        }
    }
}
```

Le test `if (keystorePropertiesFile.exists())` est important : il permet à un collaborateur qui n'a pas le keystore de compiler quand même une version de debug.

Les projets Flutter récents utilisent la variante Kotlin `build.gradle.kts`. La logique est identique ; seule la syntaxe change (`val keystoreProperties = Properties()`, `signingConfigs { create("release") { ... } }`).

### Vérifier

```bash
flutter build appbundle
```

Si la sortie ne mentionne aucun avertissement de signature, c'est bon. Un message du type « signing with debug key » signifie que Gradle n'a pas trouvé votre configuration.

---

## 42.34 — Le `build.gradle` et l'`applicationId`

L'**`applicationId`** est l'identifiant unique de votre application dans tout l'écosystème Android. Deux applications ne peuvent pas partager le même.

Dans `android/app/build.gradle.kts` (ou `build.gradle`) :

```groovy
android {
    namespace = "com.exemple.donjondedart"
    compileSdk = flutter.compileSdkVersion
    ndkVersion = flutter.ndkVersion

    defaultConfig {
        applicationId = "com.exemple.donjondedart"
        minSdk = flutter.minSdkVersion
        targetSdk = flutter.targetSdkVersion
        versionCode = flutter.versionCode
        versionName = flutter.versionName
    }
}
```

### Les règles de l'`applicationId`

| Règle | Détail |
| --- | --- |
| Format « domaine inversé » | `com.monsite.monjeu`. |
| Jamais `com.example.*` | Le Play Store refuse cet identifiant réservé aux exemples. |
| Minuscules, sans tiret ni accent | Les points séparent les segments. |
| **Définitif** | Une fois publié, il ne peut plus changer. Le modifier revient à publier une autre application. |

C'est la première chose à corriger dans un projet créé avec `flutter create`, et c'est celle qu'on oublie le plus souvent.

### Les champs de version

`versionCode` et `versionName` ne sont pas renseignés à la main : ils viennent du `pubspec.yaml`.

```yaml
version: 1.2.0+7
```

```text
  1.2.0  →  versionName  : ce que voit l'utilisateur
      7  →  versionCode  : ce que compare le Play Store
```

Le `versionCode` doit **strictement augmenter** à chaque envoi. Le Play Store refuse un envoi dont le `versionCode` est inférieur ou égal au précédent.

### `minSdk` : jusqu'où descendre

`minSdk` fixe la version d'Android minimale. Les valeurs héritées de `flutter.minSdkVersion` conviennent dans l'immense majorité des cas. Ne la baissez pas « pour toucher plus de monde » sans mesurer : vous vous condamneriez à tester sur des appareils que vous n'avez pas.

---

## 42.35 — Publier sur le Google Play Store : les étapes

La partie technique est finie ; la partie administrative commence. Voici la procédure, dans l'ordre.

```text
  1. COMPTE DÉVELOPPEUR
     Créer un compte sur la Google Play Console.
     Frais d'inscription uniques, à régler une fois.
     Vérification d'identité obligatoire : prévoir plusieurs jours.

  2. CRÉER L'APPLICATION
     Nom, langue par défaut, application ou jeu, gratuite ou payante.
     Le choix gratuit/payant est DÉFINITIF pour une application gratuite.

  3. REMPLIR LA FICHE
     • Description courte (80 caractères)
     • Description complète (4000 caractères)
     • Icône 512 × 512
     • Image de présentation 1024 × 500
     • Au moins 2 captures d'écran par type d'appareil
     • Catégorie et coordonnées de contact

  4. RÉPONDRE AUX QUESTIONNAIRES
     • Classification du contenu (violence, achats, etc.)
     • Public cible et enfants
     • Sécurité des données : ce que l'application collecte
     • Politique de confidentialité (URL publique obligatoire)

  5. ENVOYER LE BUILD
     Production, ou test interne / fermé / ouvert.
     Déposer le fichier app.aab signé.
     Rédiger les notes de version.

  6. EXAMEN
     Google vérifie l'application. Comptez de quelques heures
     à plusieurs jours pour une première publication.

  7. PUBLICATION
     Déploiement progressif possible (5 %, 20 %, 100 %).
```

### La signature d'application Play

Google propose de gérer lui-même la clé finale de signature : vous signez avec votre clé d'*upload*, Google resigne avec la clé d'application. C'est la raison pour laquelle la documentation Flutter nomme le keystore `upload-keystore.jks`.

L'avantage est décisif : si vous perdez votre clé d'upload, Google peut la réinitialiser. Sans ce service, une clé perdue signifie une application définitivement bloquée.

### Les pièges classiques d'une première publication

| Piège | Conséquence |
| --- | --- |
| `applicationId` resté en `com.example.*` | Envoi refusé. |
| Pas d'URL de politique de confidentialité | Publication bloquée. |
| Captures d'écran au mauvais format | Fiche incomplète, publication impossible. |
| APK au lieu d'AAB | Envoi refusé pour une nouvelle application. |
| `versionCode` non incrémenté | Envoi refusé. |
| Assets sous licence non respectée | Retrait de l'application, éventuellement du compte. |

> **Conseil :** utilisez d'abord la piste de **test interne**. Elle accepte jusqu'à cent testeurs, se publie en quelques minutes au lieu de plusieurs jours, et permet de dérouler toute la chaîne sans risque.

---

## 42.36 — Le build Web : `flutter build web`

C'est de loin le chemin le plus court entre votre jeu et un joueur : un lien, aucune installation.

```bash
flutter build web
```

**Résultat :**

```text
Compiling lib/main.dart for the Web...                             28,4s
✓ Built build/web
```

Le contenu de `build/web` :

```text
  build/web/
  ├── index.html
  ├── main.dart.js
  ├── flutter.js
  ├── flutter_bootstrap.js
  ├── flutter_service_worker.js
  ├── manifest.json
  ├── favicon.png
  ├── icons/
  ├── assets/
  │   ├── AssetManifest.json
  │   ├── FontManifest.json
  │   └── assets/images/...
  └── canvaskit/
```

Ce dossier est un site statique complet. N'importe quel hébergement de fichiers suffit ; aucun serveur applicatif n'est nécessaire.

### Les options vérifiées

```bash
flutter build web --release        # par défaut : minifié, tree shaking
flutter build web --profile        # pour profiler dans Chrome DevTools
flutter build web --debug          # assertions actives, non minifié
flutter build web --wasm           # compilation WebAssembly
flutter build web --source-maps    # génère les .js.map / .wasm.map
flutter build web --base-href /mon-jeu/
flutter build web --csp            # pas de génération dynamique de code
flutter build web -O 4             # niveau d'optimisation de la compilation
```

| Mode | Code minifié | Tree shaking |
| --- | --- | --- |
| debug | Non | Non |
| profile | Non | Oui |
| release | Oui | Oui |

> **Sur `--source-maps` :** les fichiers `.map` permettent de retrouver votre code Dart d'origine à partir d'une erreur JavaScript. La documentation est formelle : **ne les publiez pas sur un hébergement public**. Envoyez-les à votre service de suivi d'erreurs et excluez-les du site.

### Tester localement

```bash
flutter run -d chrome --release
```

Ou, pour servir le dossier produit comme le ferait un vrai hébergeur :

```bash
cd build/web
python3 -m http.server 8000
```

Puis ouvrez `http://localhost:8000`. Ouvrir `index.html` directement par un double clic ne fonctionne pas : le protocole `file://` interdit les chargements dont Flutter a besoin.

---

## 42.37 — Les renderers Web et lequel choisir pour un jeu

Au 8 août 2026, Flutter dispose de **deux moteurs de rendu Web actifs** :

| Moteur | Taille de téléchargement | Particularité |
| --- | --- | --- |
| **canvaskit** | environ 1,5 Mo | Skia compilé en WebAssembly. Compatible avec tous les navigateurs modernes. |
| **skwasm** | environ 1,1 Mo | Version plus compacte de Skia. Sait rendre sur un fil séparé. Nécessite la compilation WebAssembly. |

L'ancien moteur HTML n'existe plus dans les versions actuelles de Flutter : la question « HTML ou CanvasKit ? », que l'on trouve encore dans de vieux tutoriels, n'a plus lieu d'être.

### Comment le moteur est choisi

| Compilation | Commande | Moteur utilisé |
| --- | --- | --- |
| JavaScript (par défaut) | `flutter build web` | `canvaskit` |
| WebAssembly | `flutter build web --wasm` | `skwasm` si le navigateur gère WasmGC, sinon repli sur `canvaskit` |

### Forcer un moteur au chargement

Quand on compile avec `--wasm`, on peut imposer le moteur dans `web/index.html` :

```text
<script>
  const config = {
    renderer: "canvaskit",
  };
  _flutter.loader.load({
    config: config,
  });
</script>
```

Le moteur ne peut plus changer après l'appel à `_flutter.loader.load()`.

### Que choisir pour un jeu

Un jeu 2D dessine énormément par frame. Les deux moteurs reposent sur Skia et conviennent ; `skwasm`, qui sait rendre sur un fil séparé, a l'avantage théorique sur les scènes chargées.

La recommandation pratique tient en trois points :

1. commencez par `flutter build web` sans option : c'est le chemin le plus compatible ;
2. essayez `flutter build web --wasm` et **mesurez** sur la même scène, avec le compteur FPS de Flame ;
3. gardez à l'esprit que le Web impose de toute façon un temps de premier chargement de l'ordre de 1 à 2 Mo. Pour un jeu de navigateur, un écran de chargement soigné est indispensable.

---

## 42.38 — Héberger sur GitHub Pages ou itch.io

### GitHub Pages

Gratuit, adossé à votre dépôt, idéal pour un projet de formation.

```bash
# 1. Compiler avec le bon chemin de base (voir 42.39)
flutter build web --release --base-href /donjon-de-dart/

# 2. Publier le contenu de build/web sur la branche gh-pages
git checkout --orphan gh-pages
git rm -rf .
cp -r build/web/* .
git add .
git commit -m "Déploiement de la version Web"
git push origin gh-pages
git checkout main
```

Puis, dans les réglages du dépôt, section *Pages*, choisissez la branche `gh-pages`. Le jeu sera à l'adresse :

```text
https://<utilisateur>.github.io/<nom-du-depot>/
```

Automatiser le déploiement est préférable :

```yaml
# .github/workflows/deploy.yml
name: Déployer sur GitHub Pages

on:
  push:
    branches: [main]

permissions:
  contents: write

jobs:
  deployer:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: subosito/flutter-action@v2
        with:
          channel: stable
          cache: true

      - run: flutter pub get

      - run: flutter build web --release --base-href /donjon-de-dart/

      - name: Publier
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./build/web
```

À chaque `push` sur `main`, le jeu en ligne est mis à jour. C'est le circuit court dont rêvent tous les projets étudiants.

### itch.io

itch.io est la plateforme de référence pour les jeux indépendants, et elle accepte les jeux HTML5.

```bash
# 1. Compiler à la racine : itch sert le jeu depuis sa propre racine
flutter build web --release

# 2. Créer une archive du contenu de build/web
cd build/web
zip -r ../../donjon-de-dart-web.zip .
```

Sur itch.io : créer un projet, choisir le type **HTML**, envoyer le ZIP, cocher « This file will be played in the browser », et régler la taille de la fenêtre (par exemple 960 × 640).

**Le point à ne pas manquer :** l'archive doit contenir `index.html` **à sa racine**, et non dans un sous-dossier. C'est la première cause d'échec.

itch.io permet aussi de déposer l'APK Android sur la même page. Un seul lien, deux plateformes : c'est une excellente manière de présenter un projet de fin de formation.

### Comparatif

| Plateforme | Coût | Public | Mise à jour |
| --- | --- | --- | --- |
| GitHub Pages | Gratuit | Technique, recruteurs | Automatisable en CI |
| itch.io | Gratuit | Joueurs, game jams | Manuelle ou via `butler` |
| Firebase Hosting | Gratuit jusqu'à un quota | Quelconque | `firebase deploy` |
| Google Play | Frais uniques | Grand public mobile | Via la Console |

Pour Firebase, la documentation précise que `firebase deploy` lance lui-même `flutter build web --release` : il est inutile de compiler avant.

---

## 42.39 — Le chemin de base (`--base-href`)

Voici la panne la plus fréquente d'un premier déploiement Web : **le jeu marche en local, et affiche une page blanche une fois en ligne.**

### La cause

`index.html` contient une balise qui indique à partir d'où charger les fichiers :

```text
<base href="/">
```

Avec `/`, le navigateur cherche `main.dart.js` à la racine du domaine.

```text
  Hébergé sur https://moi.github.io/donjon-de-dart/
  Le navigateur cherche https://moi.github.io/main.dart.js   → 404
  Il faudrait     https://moi.github.io/donjon-de-dart/main.dart.js
```

La page se charge, mais aucun script n'est trouvé : écran blanc, et des erreurs 404 dans la console du navigateur.

### La correction

```bash
flutter build web --base-href /donjon-de-dart/
```

L'option est documentée dans l'outil Flutter : elle définit la valeur de `base href` pour l'application Web, et **la valeur doit commencer et finir par `/`**.

### Le tableau de correspondance

| Adresse finale | Option à utiliser |
| --- | --- |
| `https://monsite.com/` | `--base-href /` (valeur par défaut) |
| `https://moi.github.io/donjon-de-dart/` | `--base-href /donjon-de-dart/` |
| `https://monsite.com/jeux/donjon/` | `--base-href /jeux/donjon/` |
| itch.io (sert depuis sa propre racine) | ne rien préciser |

### Le diagnostic en trois secondes

Page blanche en ligne ? Ouvrez la console du navigateur (F12), onglet **Network**. Si vous voyez des `404` sur `main.dart.js` ou `flutter.js`, le problème est le `base href`, à 99 %. Comparez le chemin demandé et le chemin réel : la différence saute aux yeux.

> **Piège associé :** oublier le `/` final. `--base-href /donjon-de-dart` (sans slash) est refusé. Les deux slashs sont obligatoires.

---

## 42.40 — Le build Windows, macOS et Linux

Flutter compile aussi vers le bureau. Le même code, sans modification, donne un exécutable natif.

```bash
flutter build windows
flutter build macos
flutter build linux
```

**Contrainte incontournable :** on ne compile pour une plateforme que **depuis cette plateforme**. Pas de Windows depuis Linux, pas de macOS depuis Windows. Une machine virtuelle ou un runner de CI adapté est nécessaire — c'est un usage classique de GitHub Actions, qui propose des runners Windows, macOS et Linux.

Vérifiez d'abord que la plateforme est activée :

```bash
flutter devices
flutter config --enable-windows-desktop
```

### Spécificités par plateforme

| Plateforme | Icône | Distribution |
| --- | --- | --- |
| **Windows** | `windows\runner\resources\app_icon.ico` | Dossier à zipper, ou paquet `.msix` pour le Microsoft Store (package `msix` sur pub.dev). |
| **macOS** | `macos/Runner/Assets.xcassets` | `.app` à signer et notariser pour une distribution hors App Store. |
| **Linux** | Fichier `.desktop` et icône | Dossier, Snap, Flatpak ou AppImage. |

Le versionnage accepte les mêmes options qu'ailleurs :

```bash
flutter build windows --build-name=1.0.0 --build-number=1
```

Une particularité documentée pour le Microsoft Store : le numéro de révision (le quatrième nombre) doit rester à zéro, au format `X.Y.Z.0`.

### Ce qu'il faut adapter dans un jeu

| Point | Adaptation |
| --- | --- |
| Contrôles | Clavier et souris, pas de tactile. Le joystick du chapitre 30 doit être masqué. |
| Taille de fenêtre | Redimensionnable : la caméra du chapitre 31 doit suivre. |
| Plein écran | À prévoir explicitement (touche F11, par exemple). |
| Sortie du jeu | Un bouton « Quitter » a du sens sur bureau, pas sur Android. |

Le bureau est une excellente cible pour un projet de formation : aucune signature obligatoire, aucun magasin, aucun compte payant. Un ZIP suffit à faire jouer un jury.

---

## 42.41 — Les mises à jour et le versionnage

### Le format

```yaml
version: 1.2.3+45
```

```text
  1  .  2  .  3  +  45
  │     │     │     └── numéro de build (versionCode Android)
  │     │     └──────── correctif  : corrections de bugs
  │     └────────────── mineure    : nouvelles fonctionnalités compatibles
  └──────────────────── majeure    : changement de fond
```

C'est le **versionnage sémantique**, la convention adoptée par l'écosystème Dart (chapitre 16, contraintes de version et caret `^`).

### La règle pour un jeu

| Changement | Version passe de… à |
| --- | --- |
| Correction d'un bug de collision | `1.2.3` → `1.2.4` |
| Ajout d'un niveau, d'un ennemi | `1.2.4` → `1.3.0` |
| Refonte des contrôles, sauvegarde incompatible | `1.3.0` → `2.0.0` |

Et dans tous les cas, **le numéro après `+` augmente**. Toujours. Même pour un correctif minuscule.

### Le journal des modifications

Un fichier `CHANGELOG.md` à la racine, tenu à chaque version :

```text
# Journal des modifications

## 1.3.0 — 2026-09-14
### Ajouté
- Niveau 4 « Les catacombes »
- Nouvel ennemi : le squelette archer
### Modifié
- Le boss du niveau 3 a 20 % de PV en moins
### Corrigé
- Le joueur pouvait traverser le sol après un double saut
- Le meilleur score n'était pas sauvegardé après une victoire

## 1.2.4 — 2026-08-30
### Corrigé
- Plantage au chargement du niveau 2 sur Android 10
```

Ce fichier alimente directement les notes de version du Play Store, et il vous servira de mémoire quand un joueur signalera un bug « apparu récemment ».

### La compatibilité des sauvegardes

C'est le piège des mises à jour d'un jeu. Un joueur qui met à jour ne doit pas perdre sa progression. Si le format de sauvegarde change, prévoyez une **migration** et versionnez la sauvegarde elle-même :

```dart
class SauvegardeService {
  static const int versionFormat = 2;

  Future<Map<String, dynamic>> migrer(Map<String, dynamic> donnees) async {
    final int version = donnees['version'] as int? ?? 1;
    if (version == versionFormat) return donnees;

    if (version == 1) {
      // La version 1 n'avait pas de champ « clés ».
      donnees['cles'] = 0;
      donnees['version'] = 2;
    }
    return donnees;
  }
}
```

Sans ce mécanisme, la mise à jour 1.3.0 fera planter tous les joueurs qui avaient une partie en cours — et vous ne le découvrirez qu'à travers les avis une étoile.

---

## 42.42 — La présentation finale du projet : plan et grille d'évaluation

Un projet non présenté est un projet à moitié terminé. Que ce soit devant un jury, une classe ou un recruteur, l'exercice obéit à des règles simples.

### Le plan d'une présentation de vingt minutes

```text
  ┌── 1. LE JEU EN MARCHE ─────────────────────────── 2 min ──┐
  │   On lance le jeu et on joue. Pas de diapositive.         │
  │   Le jury doit voir le résultat AVANT les explications.   │
  ├── 2. LE PITCH ─────────────────────────────────── 1 min ──┤
  │   Une phrase : genre, plateforme, boucle de jeu.          │
  │   « Un jeu de plateforme 2D mobile et web où l'on         │
  │     explore trois donjons pour vaincre un boss. »         │
  ├── 3. LE CAHIER DES CHARGES ────────────────────── 3 min ──┤
  │   Ce qui était prévu (chapitre 41), ce qui a été fait,    │
  │   ce qui a été abandonné et POURQUOI. L'honnêteté sur     │
  │   les coupes est valorisée, jamais sanctionnée.           │
  ├── 4. L'ARCHITECTURE ───────────────────────────── 4 min ──┤
  │   Un schéma de l'arborescence, l'arbre de composants,     │
  │   la machine à états. Un seul schéma clair vaut mieux     │
  │   que cinq écrans de code.                                │
  ├── 5. UNE DIFFICULTÉ TECHNIQUE ─────────────────── 4 min ──┤
  │   Un problème réel, votre démarche, la solution.          │
  │   C'est ici que se joue l'essentiel de la note.           │
  ├── 6. QUALITÉ ET MESURES ───────────────────────── 3 min ──┤
  │   Nombre de tests, couverture, FPS mesuré en profil,      │
  │   taille de l'APK. Des chiffres, pas des adjectifs.       │
  ├── 7. LIMITES ET SUITE ─────────────────────────── 2 min ──┤
  │   Ce que vous feriez avec deux semaines de plus.          │
  └── 8. QUESTIONS ────────────────────────────────── 5 min ──┘
```

### Les erreurs de présentation

| Erreur | Pourquoi c'est coûteux |
| --- | --- |
| Commencer par une diapositive de plan | Le jury s'endort avant d'avoir vu le jeu. |
| Faire défiler du code pendant dix minutes | Personne ne lit du code projeté. Montrez un schéma. |
| Cacher ce qui ne marche pas | Le jury le trouvera. Mieux vaut l'annoncer soi-même. |
| Dire « je n'ai pas eu le temps » sans plus | Dire plutôt : « j'ai priorisé X plutôt que Y, voici pourquoi ». |
| Faire la démonstration en mode debug | Le jeu rame, et vous passez pour un développeur qui ne mesure pas. |
| Ne pas avoir de plan B | Prévoyez une vidéo de secours si l'appareil refuse de coopérer. |

### La grille d'évaluation type

Voici une grille de 20 points telle qu'on en rencontre en fin de formation. Lisez-la comme une liste d'exigences, pas comme une menace.

| Critère | Points | Ce qui est attendu |
| --- | --- | --- |
| **Le jeu fonctionne** | 4 | Se lance, se joue du début à la fin sans plantage, sur au moins une plateforme. |
| **Complétude fonctionnelle** | 3 | Menu, jeu, pause, Game Over, victoire, sauvegarde, plusieurs niveaux. |
| **Architecture du code** | 3 | Découpage clair, séparation logique/rendu, noms cohérents, aucun fichier fourre-tout. |
| **Qualité et tests** | 3 | `flutter analyze` propre, suite de tests présente et verte, couverture raisonnable. |
| **Performance** | 2 | Mesure faite en mode profil, FPS annoncé, au moins une optimisation justifiée. |
| **Livraison** | 2 | APK signé et/ou version Web en ligne, procédure de build documentée. |
| **Documentation** | 1 | `README.md` avec installation, contrôles, crédits d'assets, licence. |
| **Présentation orale** | 2 | Claire, dans le temps, honnête sur les limites, réponses précises aux questions. |

### Ce qui fait la différence

Un jury voit beaucoup de projets qui « marchent ». Ce qui distingue un très bon projet d'un projet correct tient en trois choses : des **chiffres** (« 47 tests, 58 % de couverture, 60 FPS stables mesurés en profil sur un Pixel 6a, APK de 8,2 Mo »), une **difficulté réelle racontée honnêtement**, et un **lien qui fonctionne** pour que le jury puisse jouer après la soutenance.

---

## 42.43 — Bilan des 42 chapitres : le tableau récapitulatif de toute la formation

Vous venez de parcourir 42 chapitres. Il est temps de regarder le chemin depuis le sommet.

### PARTIE 1A — Le langage Dart

| # | Chapitre | Ce que vous en avez retiré |
| --- | --- | --- |
| 01 | Apprendre Dart de zéro | `main()`, `print`, la première exécution dans DartPad. |
| 02 | Variables, constantes et types | `var`, `final`, `const`, `int`, `double`, `String`, `bool`. |
| 03 | Opérateurs et expressions | Arithmétique, comparaison, logique, opérateurs d'affectation. |
| 04 | Conditions | `if`, `else`, `switch`, expressions conditionnelles. |
| 05 | Boucles | `for`, `while`, `do while`, `break`, `continue`. |
| 06 | Collections | `List`, `Set`, `Map` et leurs opérations de base. |
| 07 | Les fonctions | Paramètres nommés, optionnels, fonctions fléchées, fermetures. |
| 08 | Introduction à la POO | Classes, objets, champs, méthodes, `this`. |
| 09 | Constructeurs et modélisation | Constructeurs nommés, initialisation, `factory`. |
| 10 | Encapsulation, héritage, polymorphisme | `_privé`, `extends`, `@override`, `super`. |
| 11 | POO avancée | `abstract`, `mixin`, `enum`, interfaces implicites. |
| 12 | Le null safety | `?`, `!`, `??`, `?.`, `late`, la fin des erreurs de nullité. |
| 13 | Exceptions | `throw`, `try`, `catch`, `finally`, exceptions personnalisées. |
| 14 | Programmation fonctionnelle | `map`, `where`, `fold`, `reduce`, `any`, `every`. |
| 15 | Programmation asynchrone | `Future`, `async`, `await`, `Stream`. |
| 16 | Organisation d'un projet Dart | `dart create`, `pubspec.yaml`, packages, `import`, tests. |
| 17 | JSON et modélisation | `jsonEncode`, `jsonDecode`, `fromJson`, `toJson`. |
| 18 | Mini-projet final Dart | Le « Donjon de Dart » en version console. |

### PARTIE 2A — Les fondamentaux du jeu 2D

| # | Chapitre | Ce que vous en avez retiré |
| --- | --- | --- |
| 19 | Flutter en accéléré pour le jeu | `runApp`, widgets, `setState`, `Ticker`, `CustomPainter`. |
| 20 | Game loop, FPS et delta time | La boucle, le `dt`, l'indépendance au framerate. |
| 21 | Coordonnées, Canvas et dessin 2D | Le repère écran, `Canvas`, `Paint`, transformations. |
| 22 | Sprites, sprite sheets et animation | Découpage d'une planche, cadence, index de frame. |
| 23 | Mouvement, vélocité et physique simple | Vecteurs, accélération, gravité, saut, frottement. |
| 24 | Collisions et hitboxes | AABB, cercles, résolution, tunneling. |
| 25 | Caméra, monde et niveaux | Monde contre écran, caméra suiveuse, parallaxe, tilemap. |
| 26 | Architecture et états d'un jeu | Entités, machine à états, séparation logique/rendu. |

### PARTIE 2B — Le moteur Flame

| # | Chapitre | Ce que vous en avez retiré |
| --- | --- | --- |
| 27 | Installer Flame, premier `FlameGame` | `GameWidget`, `onLoad`, `update`, `render`. |
| 28 | Components et cycle de vie | Arbre de composants, `add`, `removeFromParent`, `priority`. |
| 29 | Sprites, animations et assets | `SpriteComponent`, `SpriteAnimationComponent`, `images.load`. |
| 30 | Entrées clavier, tactile et joystick | `KeyboardHandler`, `TapCallbacks`, `JoystickComponent`. |
| 31 | Caméra, viewport et monde | `CameraComponent`, `World`, `follow`, zoom, HUD. |
| 32 | Collisions et `CollisionCallbacks` | `HasCollisionDetection`, hitboxes, `onCollisionStart`. |
| 33 | Effets, particules et timers | `MoveEffect`, `ParticleSystemComponent`, `TimerComponent`. |
| 34 | Audio, Tiled et physique avancée | `flame_audio`, `flame_tiled`, aperçu de `flame_forge2d`. |

### PARTIE 2C — Le jeu complet

| # | Chapitre | Ce que vous en avez retiré |
| --- | --- | --- |
| 35 | Architecture du jeu et menu principal | Squelette du projet, `GameState`, overlays, navigation. |
| 36 | Le joueur : mouvement et animations | Gravité, saut, machine à états d'animation. |
| 37 | Ennemis, IA et combat | Patrouille, poursuite, dégâts, invincibilité. |
| 38 | Objets, collectibles, score et HUD | Pièces, potions, clés, combo, barres de vie et d'énergie. |
| 39 | Niveaux, boss et progression | Cartes en `List<String>`, portes, transitions, boss. |
| 40 | Sons, pause, Game Over et sauvegarde | Effets sonores, musique, écrans finaux, meilleur score. |

### PARTIE 2D — Méthode, qualité et livraison

| # | Chapitre | Ce que vous en avez retiré |
| --- | --- | --- |
| 41 | Cahier des charges et GDD | Spécifier avant de coder, planifier, gérer les assets. |
| 42 | Tests, performance et publication | Tester, mesurer, optimiser, signer, publier, présenter. |

### En une phrase

Vous êtes parti de `print('Bonjour')` et vous arrivez à un jeu de plateforme testé, profilé, signé et publié sur deux plateformes. Ce n'est pas une progression linéaire : c'est un changement de métier.

---

## 42.44 — Aller plus loin : ressources, communautés, prochains sujets

### La documentation, d'abord

Un réflexe à conserver toute votre carrière : la documentation officielle avant les tutoriels.

| Ressource | Ce qu'on y trouve |
| --- | --- |
| `dart.dev` | Le langage, la bibliothèque standard, les guides d'écriture. |
| `docs.flutter.dev` | Le framework, le déploiement, la performance, les tests. |
| `api.flutter.dev` | La référence exhaustive de chaque classe. |
| `docs.flame-engine.org` | Le moteur Flame, à jour, avec des exemples exécutables. |
| `pub.dev` | Les packages, leur score, leur documentation, leur code source. |
| `examples.flame-engine.org` | Une galerie d'exemples Flame interactifs. |

Les tutoriels vieillissent mal, particulièrement pour Flame dont l'API a beaucoup changé entre les versions 0.x et 1.x. Un article de 2021 vous fera perdre plus de temps qu'il ne vous en fera gagner.

### Les communautés

| Lieu | Usage |
| --- | --- |
| Discord officiel de Flame | Aide rapide, très active, contributeurs présents. |
| GitHub `flame-engine/flame` | Signaler un bug, lire le code source, suivre les évolutions. |
| Stack Overflow, étiquettes `flutter` et `flame` | Problèmes précis déjà résolus. |
| Reddit `r/FlutterDev` | Veille et retours d'expérience. |
| itch.io et les game jams | Publier, jouer, recevoir des retours réels. |

### Les prochains sujets techniques

| Sujet | Pourquoi c'est la suite logique |
| --- | --- |
| **`flame_forge2d`** | Une vraie physique : corps rigides, articulations, ressorts. |
| **`flame_tiled` en profondeur** | Concevoir vos niveaux dans l'éditeur Tiled plutôt qu'en `List<String>`. |
| **La gestion d'état Flutter** | Provider, Riverpod, Bloc : indispensables pour les menus complexes. |
| **Les shaders** | Effets visuels avancés avec `FragmentProgram`. |
| **Le multijoueur** | WebSocket, Firebase, ou `flame_network` selon les besoins. |
| **Le portage console** | Un autre monde : certification, manettes, contraintes de plateforme. |
| **Le game design** | Level design, courbe de difficulté, économie de jeu, ressenti. |
| **L'audio de jeu** | Musique adaptative, mixage, spatialisation. |

### La meilleure chose à faire maintenant

Participer à une **game jam**. Une jam impose un thème et un délai court (souvent 48 heures), et interdit le perfectionnisme. Vous y apprendrez, en un week-end, plus sur la finition d'un jeu que dans un mois de tutoriels : couper une fonctionnalité, livrer quelque chose de jouable, et surtout regarder quelqu'un jouer à votre jeu sans lui expliquer comment faire.

Ce dernier moment est le plus instructif de tout le développement de jeu. Il est aussi le plus inconfortable. C'est exactement pour cela qu'il faut le provoquer.

---

## 42.45 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| Le test échoue : `expect(component.isMounted, true)` donne `false` | `await game.ready()` a été oublié après l'ajout du composant. | Toujours `await game.ready()` avant d'assertion sur l'arbre de composants. |
| Un test de collision ne déclenche jamais `onCollisionStart` | La détection a lieu pendant `update`, jamais à l'ajout. | Appeler `game.update(0)` avant de vérifier. |
| Un test de déplacement montre une position inchangée | Aucune boucle de jeu ne tourne dans un test. | Appeler `game.update(dt)` autant de fois que nécessaire. |
| `expect(joueur.pv, 75.0)` échoue avec `74.99999999` | Comparaison exacte de `double`. | Utiliser `closeTo(75, 0.001)`. |
| `pumpAndSettle()` expire au bout de dix minutes | Une animation infinie tourne dans le widget testé. | Utiliser `pump(Duration(...))` avec une durée fixe. |
| Le golden test échoue à chaque exécution | Animation, aléatoire ou rendu de texte non déterministes. | Figer la scène, fixer la graine aléatoire, utiliser `DebugTextRenderer`. |
| `flutter test` ne trouve aucun test | Les fichiers ne finissent pas par `_test.dart`. | Renommer, ou déplacer les helpers hors du motif. |
| Le jeu tourne à 20 FPS et rien ne semble anormal | Mesure faite en mode debug. | Mesurer avec `flutter run --profile` ou `--release`. |
| Le FPS baisse régulièrement pendant la partie | Composants jamais retirés, ou allocations dans `update()`. | `ChildCounterComponent` pour la fuite, opérations en place pour les allocations. |
| Le jeu rame dès la première seconde, de manière constante | `debugMode` laissé actif. | `debugMode = kDebugMode` et un test qui le vérifie. |
| Des pics de FPS réguliers, à intervalle fixe | Le ramasse-miettes collecte les objets créés dans `update()`. | Supprimer les `Vector2`, `Paint` et listes créés à chaque frame. |
| L'application se ferme seule sur un téléphone d'entrée de gamme | Images trop grandes en mémoire. | Redimensionner les images et regrouper en atlas. |
| `flutter build appbundle` produit un fichier signé en debug | Gradle n'a pas trouvé `key.properties`. | Vérifier le chemin, le nom exact du fichier et la configuration `signingConfigs`. |
| Le Play Store refuse l'envoi | `applicationId` en `com.example.*`, ou `versionCode` non incrémenté. | Corriger l'`applicationId` et augmenter le nombre après `+` dans `pubspec.yaml`. |
| Page blanche sur GitHub Pages, tout marche en local | `base href` incorrect pour un sous-répertoire. | Compiler avec `--base-href /nom-du-depot/`, slashs compris. |
| itch.io affiche « index.html not found » | L'archive contient un dossier au lieu des fichiers. | Zipper le **contenu** de `build/web`, pas le dossier. |
| Les joueurs perdent leur sauvegarde après mise à jour | Le format de sauvegarde a changé sans migration. | Versionner la sauvegarde et écrire une fonction de migration. |
| Le keystore est perdu | Sauvegardé nulle part, ou committé puis supprimé. | Sauvegarde hors dépôt dès sa création ; activer la signature d'application Play. |

---

## 42.46 — Résumé du chapitre

| Commande | Rôle |
| --- | --- |
| `flutter pub get` | Installe les dépendances déclarées dans `pubspec.yaml`. |
| `flutter analyze` | Analyse statique : erreurs et avertissements sans exécution. |
| `dart format .` | Reformate tout le projet selon le style officiel. |
| `dart format --output=none --set-exit-if-changed .` | Vérifie le formatage sans modifier les fichiers (usage CI). |
| `flutter test` | Exécute tous les fichiers `*_test.dart` de `test/`. |
| `flutter test test/composants/joueur_test.dart` | Exécute un seul fichier de test. |
| `flutter test --name collision` | Exécute les tests dont le nom contient « collision ». |
| `flutter test --coverage` | Produit `coverage/lcov.info`. |
| `flutter test --update-goldens` | Régénère les images de référence des golden tests. |
| `flutter test integration_test` | Exécute les tests d'intégration sur un appareil. |
| `genhtml coverage/lcov.info -o coverage/html` | Convertit la couverture en rapport HTML (outil `lcov`). |
| `flutter run --debug` | Lance en mode développement, avec hot reload. |
| `flutter run --profile` | Lance en mode profil : le mode de mesure de performance. |
| `flutter run --release` | Lance la version optimisée, telle que la verra le joueur. |
| `flutter devices` | Liste les appareils et navigateurs disponibles. |
| `flutter doctor -v` | Diagnostique la chaîne d'outils installée. |
| `flutter clean` | Supprime `build/` et `.dart_tool/` : à faire en cas de build incohérent. |
| `flutter build apk` | Produit un APK contenant toutes les architectures. |
| `flutter build apk --split-per-abi` | Produit un APK par architecture, bien plus léger. |
| `flutter build appbundle` | Produit `app.aab`, le format exigé par le Play Store. |
| `flutter build appbundle --build-name=1.2.0 --build-number=7` | Fixe la version sans modifier `pubspec.yaml`. |
| `flutter build appbundle --obfuscate --split-debug-info=<dossier>` | Obfusque le code Dart et met les symboles à part. |
| `flutter install` | Installe l'application sur l'appareil branché. |
| `flutter symbolize` | Rend lisible une trace d'erreur issue d'un binaire obfusqué. |
| `keytool -genkey -v -keystore ~/upload-keystore.jks -keyalg RSA -storetype JKS -keysize 2048 -validity 10000 -alias upload` | Crée le keystore de signature Android. |
| `bundletool build-apks --bundle=app.aab --output=app.apks` | Convertit un `.aab` en APK testables. |
| `flutter build web` | Produit le site statique dans `build/web`. |
| `flutter build web --wasm` | Compile en WebAssembly (moteur `skwasm` si disponible). |
| `flutter build web --base-href /mon-jeu/` | Règle le chemin de base pour un hébergement en sous-répertoire. |
| `flutter build web --source-maps` | Génère les fichiers `.map` (à ne jamais publier). |
| `flutter build windows` / `macos` / `linux` | Produit l'exécutable de bureau, depuis la plateforme visée. |
| `dart run flutter_launcher_icons` | Génère toutes les icônes à partir d'une seule image. |
| `dart run flutter_native_splash:create` | Génère les écrans de démarrage natifs. |

| Notion | À retenir |
| --- | --- |
| Ce qu'on teste | Le déterministe et le sans-écran. Le ressenti se teste en jouant. |
| `await game.ready()` | Indispensable avant toute assertion sur l'arbre de composants. |
| `game.update(dt)` | La machine à remonter le temps des tests de jeu. |
| `game.update(0)` | Déclenche la détection de collision sans faire avancer le monde. |
| Golden tests | Puissants sur une scène figée, ingérables sur une scène animée. |
| Couverture | Un révélateur, jamais un objectif. 50 % honnêtes valent mieux que 90 % vides. |
| `debugMode` | Utile en développement, désastreux en production. `kDebugMode` protège. |
| Budget de frame | 16,67 ms à 60 FPS. Les deux fils, UI et raster, doivent tenir dedans. |
| Mesurer avant d'optimiser | Une optimisation sans mesure est du code moins lisible, rien de plus. |
| Allocations dans `update()` | La première cause de pics de FPS. Opérations en place obligatoires. |
| Pooling | Pour les projectiles et les particules. Réinitialiser **tous** les champs. |
| Culling | Ne pas dessiner hors champ. `camera.canSee` et `visibleWorldRect`. |
| Modes de compilation | Développer en debug, mesurer en profil, publier en release. |
| `applicationId` | Définitif après publication. Jamais `com.example.*`. |
| Keystore | Irremplaçable. Sauvegardé hors du dépôt, jamais dans Git. |
| `versionCode` | Doit strictement augmenter à chaque envoi au Play Store. |
| Moteurs Web | `canvaskit` par défaut, `skwasm` avec `--wasm`. Le moteur HTML n'existe plus. |
| `--base-href` | La cause numéro un des pages blanches en ligne. Slashs obligatoires. |
| Versionnage | `MAJEUR.MINEUR.CORRECTIF+BUILD`, avec un `CHANGELOG.md` tenu à jour. |

---

## 42.47 — Exercices

### Exercice 1 — Tester les règles pures (facile)

Créez `lib/core/economie.dart` avec une classe `Economie` exposant :

- `static int prixPotion(int niveau)` : 50 pièces au niveau 1, plus 25 par niveau supplémentaire ;
- `static int? acheterPotion(int solde, int niveau)` : retourne le solde restant, ou `null` si le solde est insuffisant.

Écrivez `test/core/economie_test.dart` avec **au moins six tests**, dont un cas limite (achat au prix exact) et un cas d'échec.

### Exercice 2 — Tester un composant Flame (facile)

Écrivez `test/composants/potion_test.dart` qui vérifie, avec `testWithGame` :

1. qu'une `Potion` ajoutée au monde est bien montée ;
2. qu'elle possède une hitbox enfant ;
3. qu'elle a la taille attendue.

### Exercice 3 — Simuler le temps (moyen)

Écrivez le helper `test/_helpers/simulation.dart` contenant la fonction `simuler(FlameGame game, double secondes, {double pas = 1/60})`, puis un test qui vérifie qu'un joueur en chute libre pendant 2 secondes a une vitesse verticale plafonnée à `Constantes.vitesseMaxChute`.

### Exercice 4 — Tester une collision et sa négation (moyen)

Écrivez deux tests dans `test/composants/collision_test.dart` :

1. un joueur superposé à une `Piece` fait augmenter `game.score` et la pièce disparaît du monde ;
2. un joueur situé à 500 pixels de la pièce ne change ni le score ni le nombre de pièces.

### Exercice 5 — Tester un menu (moyen)

Écrivez `test/ecrans/ecran_game_over_test.dart` qui vérifie que l'écran de Game Over :

1. affiche le texte « GAME OVER » ;
2. affiche le score final passé en paramètre ;
3. déclenche le rappel `onRejouer` quand on appuie sur le bouton portant la clé `Key('bouton_rejouer')`.

### Exercice 6 — Mettre en place la CI (moyen)

Écrivez `.github/workflows/ci.yml` qui, à chaque `push` sur `main` et à chaque *pull request* :

1. installe Flutter en version stable avec le cache activé ;
2. vérifie le formatage ;
3. lance `flutter analyze` ;
4. lance `flutter test --coverage` ;
5. publie `coverage/lcov.info` en artefact.

### Exercice 7 — Traquer les allocations (difficile)

Voici une méthode `update` fautive. Réécrivez-la sans aucune allocation, et listez les objets que la version initiale créait à chaque frame.

```dart
@override
void update(double dt) {
  super.update(dt);
  final Vector2 gravite = Vector2(0, Constantes.gravite);
  velocite = velocite + gravite * dt;
  if (velocite.y > Constantes.vitesseMaxChute) {
    velocite = Vector2(velocite.x, Constantes.vitesseMaxChute);
  }
  position = position + velocite * dt;
}
```

### Exercice 8 — Une pool de particules (difficile)

En vous appuyant sur la classe `Pool<T>` de la section 42.23, écrivez une `PoolParticules` capable de fournir 64 `Etincelle` préallouées, avec :

- `Etincelle` qui se recycle automatiquement quand sa durée de vie est écoulée ;
- une méthode `exploser(Vector2 position, int nombre)` sur `DonjonGame` ;
- un test qui vérifie qu'après deux explosions de 20 étincelles, le nombre total d'`Etincelle` créées ne dépasse pas 64.

### Exercice 9 — Produire un APK signé (difficile)

Déroulez la procédure complète :

1. créez un keystore avec `keytool` ;
2. écrivez `android/key.properties` et ajoutez-le au `.gitignore` ;
3. configurez `signingConfigs` dans `android/app/build.gradle` ;
4. réglez `applicationId` sur `com.votrenom.donjondedart` ;
5. passez la version à `1.0.0+1` ;
6. produisez un App Bundle **et** des APK découpés par architecture ;
7. installez l'APK `arm64-v8a` sur un appareil et vérifiez qu'une partie se termine.

Indiquez les commandes exactes et la taille des fichiers obtenus.

### Exercice 10 — Déployer la version Web (difficile)

Déployez « Donjon de Dart » sur GitHub Pages :

1. compilez avec le bon `--base-href` pour un dépôt nommé `donjon-de-dart` ;
2. publiez `build/web` sur la branche `gh-pages` ;
3. écrivez le workflow `.github/workflows/deploy.yml` qui refait ce déploiement à chaque `push` sur `main` ;
4. vérifiez la console du navigateur : aucune erreur 404 ne doit apparaître ;
5. mesurez le FPS en jeu avec `FpsTextComponent` et notez le chiffre.

---

## 42.48 — Corrections des exercices

### Correction 1

```dart
// lib/core/economie.dart
class Economie {
  static const int prixDeBase = 50;
  static const int supplementParNiveau = 25;

  static int prixPotion(int niveau) {
    if (niveau < 1) {
      throw ArgumentError.value(niveau, 'niveau', 'Le niveau commence à 1');
    }
    return prixDeBase + (niveau - 1) * supplementParNiveau;
  }

  static int? acheterPotion(int solde, int niveau) {
    final int prix = prixPotion(niveau);
    if (prix > solde) return null;
    return solde - prix;
  }
}
```

```dart
// test/core/economie_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:donjon_de_dart/core/economie.dart';

void main() {
  group('Economie.prixPotion', () {
    test('niveau 1 : prix de base', () {
      expect(Economie.prixPotion(1), 50);
    });

    test('niveau 3 : deux suppléments', () {
      expect(Economie.prixPotion(3), 100);
    });

    test('niveau invalide : ArgumentError', () {
      expect(() => Economie.prixPotion(0), throwsArgumentError);
    });
  });

  group('Economie.acheterPotion', () {
    test('solde suffisant : le solde diminue du prix', () {
      expect(Economie.acheterPotion(120, 1), 70);
    });

    test('solde insuffisant : null', () {
      expect(Economie.acheterPotion(30, 1), isNull);
    });

    test('solde exactement égal au prix : zéro, pas null', () {
      expect(Economie.acheterPotion(50, 1), 0);
    });

    test('le prix augmente bien avec le niveau', () {
      expect(Economie.acheterPotion(100, 3), 0);
      expect(Economie.acheterPotion(99, 3), isNull);
    });
  });
}
```

**Explication :** trois choses méritent d'être relevées. D'abord, la logique est **hors de tout composant Flame** : elle se teste sans moteur, sans écran, en quelques millisecondes. Ensuite, le cas limite « solde exactement égal au prix » distingue `0` de `null`, deux valeurs que la logique de `if (prix > solde)` sépare et que `if (prix >= solde)` confondrait — c'est exactement le genre de bug d'un caractère qu'un test attrape et qu'une relecture manque. Enfin, le dernier test vérifie la **frontière** au niveau 3 en testant les deux côtés (`100` passe, `99` échoue) : c'est la technique de l'encadrement, la plus rentable de toutes pour tester une condition numérique.

---

### Correction 2

```dart
// test/composants/potion_test.dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame_test/flame_test.dart';
import 'package:flutter_test/flutter_test.dart';

import 'package:donjon_de_dart/composants/potion.dart';
import 'package:donjon_de_dart/donjon_game.dart';

void main() {
  group('Potion', () {
    testWithGame<DonjonGame>(
      'la potion est montée après ajout au monde',
      DonjonGame.new,
      (DonjonGame game) async {
        final Potion potion = Potion(position: Vector2(100, 100));
        await game.world.add(potion);
        await game.ready();

        expect(potion.isMounted, isTrue);
        expect(potion.parent, game.world);
      },
    );

    testWithGame<DonjonGame>(
      'la potion possède une hitbox enfant',
      DonjonGame.new,
      (DonjonGame game) async {
        final Potion potion = Potion(position: Vector2.zero());
        await game.world.add(potion);
        await game.ready();

        final Iterable<ShapeHitbox> hitboxes =
            potion.children.whereType<ShapeHitbox>();

        expect(hitboxes, isNotEmpty);
      },
    );

    testWithGame<DonjonGame>(
      'la potion a la taille attendue',
      DonjonGame.new,
      (DonjonGame game) async {
        final Potion potion = Potion(position: Vector2.zero());
        await game.world.add(potion);
        await game.ready();

        expect(potion.size, closeToVector(Vector2(16, 16), 0.01));
      },
    );
  });
}
```

**Explication :** le point central est `await game.ready()`, présent dans les trois tests. Sans lui, `isMounted` vaut `false` et `children` est vide, parce que Flame traite les ajouts en file d'attente et non immédiatement. Le deuxième test interroge `potion.children.whereType<ShapeHitbox>()` plutôt qu'un type concret comme `RectangleHitbox` : le test reste valide si vous passez plus tard à une `CircleHitbox`, ce qui est exactement le bon niveau de couplage. Le troisième utilise `closeToVector`, le *matcher* de `flame_test` prévu pour les `Vector2` : comparer deux vecteurs de `double` avec `equals` échouerait sur des différences de l'ordre de 10⁻¹⁵.

---

### Correction 3

```dart
// test/_helpers/simulation.dart
import 'package:flame/game.dart';

/// Fait avancer [game] de [secondes], par pas de [pas].
/// La dernière itération utilise le reste exact pour ne pas dépasser.
void simuler(FlameGame game, double secondes, {double pas = 1 / 60}) {
  double restant = secondes;
  while (restant > 0) {
    final double dt = restant < pas ? restant : pas;
    game.update(dt);
    restant -= dt;
  }
}
```

```dart
// test/composants/chute_test.dart
import 'package:flame/components.dart';
import 'package:flame_test/flame_test.dart';
import 'package:flutter_test/flutter_test.dart';

import 'package:donjon_de_dart/config/constantes.dart';
import 'package:donjon_de_dart/composants/joueur.dart';
import 'package:donjon_de_dart/donjon_game.dart';
import '../_helpers/simulation.dart';

void main() {
  testWithGame<DonjonGame>(
    'la vitesse de chute est plafonnée à vitesseMaxChute',
    DonjonGame.new,
    (DonjonGame game) async {
      final Joueur joueur = Joueur(position: Vector2.zero());
      await game.world.add(joueur);
      await game.ready();

      joueur.auSol = false;
      simuler(game, 2.0);

      // Sans plafond, 2 s à 1200 px/s² donneraient 2400 px/s.
      expect(joueur.velocite.y, lessThanOrEqualTo(Constantes.vitesseMaxChute));
      expect(joueur.velocite.y, greaterThan(0));
      expect(joueur.position.y, greaterThan(0));
    },
  );
}
```

**Explication :** `simuler` reproduit ce que fait la boucle de jeu réelle, à pas fixe. Ce détail n'est pas cosmétique : appeler `game.update(2.0)` une seule fois donnerait une position différente, parce que l'intégration d'Euler d'un mouvement **accéléré** dépend du pas de temps (chapitre 23). La dernière itération utilise `restant` plutôt que `pas`, ce qui garantit qu'on simule exactement la durée demandée, ni plus ni moins. Le commentaire du test explique le chiffre attendu : sans plafond, la vitesse atteindrait 2400 px/s après deux secondes, et le joueur traverserait le sol à la frame suivante. Enfin, le test vérifie aussi `greaterThan(0)` : un plafond mal écrit qui mettrait la vitesse à zéro passerait le premier `expect` tout en cassant complètement la gravité.

---

### Correction 4

```dart
// test/composants/collision_test.dart
import 'package:flame/components.dart';
import 'package:flame_test/flame_test.dart';
import 'package:flutter_test/flutter_test.dart';

import 'package:donjon_de_dart/composants/joueur.dart';
import 'package:donjon_de_dart/composants/piece.dart';
import 'package:donjon_de_dart/donjon_game.dart';

void main() {
  group('Ramassage d\'une pièce', () {
    testWithGame<DonjonGame>(
      'joueur superposé : score augmenté et pièce retirée',
      DonjonGame.new,
      (DonjonGame game) async {
        final Joueur joueur = Joueur(position: Vector2(100, 100));
        final Piece piece = Piece(position: Vector2(100, 100));
        await game.world.addAll(<Component>[joueur, piece]);
        await game.ready();

        expect(game.score, 0);
        expect(game.world.children.whereType<Piece>().length, 1);

        game.update(0);       // la détection de collision a lieu ici
        await game.ready();   // le retrait en file d'attente est appliqué

        expect(game.score, greaterThan(0));
        expect(game.world.children.whereType<Piece>(), isEmpty);
      },
    );

    testWithGame<DonjonGame>(
      'joueur éloigné : rien ne change',
      DonjonGame.new,
      (DonjonGame game) async {
        final Joueur joueur = Joueur(position: Vector2(0, 0));
        final Piece piece = Piece(position: Vector2(500, 0));
        await game.world.addAll(<Component>[joueur, piece]);
        await game.ready();

        game.update(0);
        await game.ready();

        expect(game.score, 0);
        expect(game.world.children.whereType<Piece>().length, 1);
      },
    );
  });
}
```

**Explication :** ces deux tests forment un couple indissociable. Le premier prouve que la collision fonctionne ; le second prouve qu'elle ne se déclenche pas à tort. Sans le second, une hitbox accidentellement dimensionnée à 500 pixels de large passerait inaperçue, puisque le premier test réussirait encore. Trois points techniques : `game.update(0)` déclenche la passe de détection sans faire avancer le monde ; le second `await game.ready()` vide la file de retrait, sans quoi la pièce serait encore présente au moment de l'assertion ; et l'état initial est explicitement vérifié avant l'action (`expect(game.score, 0)`), ce qui évite le test qui « réussit » simplement parce que rien ne s'est jamais passé.

---

### Correction 5

```dart
// test/ecrans/ecran_game_over_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

import 'package:donjon_de_dart/ecrans/ecran_game_over.dart';

void main() {
  Widget construire({
    required int score,
    VoidCallback? onRejouer,
    VoidCallback? onMenu,
  }) {
    return MaterialApp(
      home: EcranGameOver(
        scoreFinal: score,
        meilleurScore: 0,
        onRejouer: onRejouer ?? () {},
        onMenu: onMenu ?? () {},
      ),
    );
  }

  group('EcranGameOver', () {
    testWidgets('affiche le titre GAME OVER', (WidgetTester tester) async {
      await tester.pumpWidget(construire(score: 0));
      expect(find.text('GAME OVER'), findsOneWidget);
    });

    testWidgets('affiche le score final', (WidgetTester tester) async {
      await tester.pumpWidget(construire(score: 1240));
      expect(find.textContaining('1240'), findsOneWidget);
    });

    testWidgets('le bouton Rejouer déclenche le rappel',
        (WidgetTester tester) async {
      int appels = 0;
      await tester.pumpWidget(construire(score: 0, onRejouer: () => appels++));

      await tester.tap(find.byKey(const Key('bouton_rejouer')));
      await tester.pump();

      expect(appels, 1);
    });
  });
}
```

**Explication :** la fonction locale `construire` est le vrai enseignement de cette correction. Elle centralise la construction du widget testé : le jour où `EcranGameOver` gagne un paramètre obligatoire, une seule ligne change au lieu de trois. Le `MaterialApp` englobant n'est pas décoratif : sans lui, les widgets Material comme `ElevatedButton` ne trouvent ni thème ni `Directionality` et le test échoue avec une erreur déroutante. Enfin, l'appui se fait via `find.byKey` et non `find.text('REJOUER')` : le test survivra à une traduction ou à un changement de libellé, ce qui est précisément le but d'une `Key`.

---

### Correction 6

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  verifier:
    name: Analyse et tests
    runs-on: ubuntu-latest

    steps:
      - name: Récupérer le dépôt
        uses: actions/checkout@v4

      - name: Installer Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: stable
          cache: true

      - name: Version installée
        run: flutter --version

      - name: Dépendances
        run: flutter pub get

      - name: Formatage
        run: dart format --output=none --set-exit-if-changed .

      - name: Analyse statique
        run: flutter analyze

      - name: Tests et couverture
        run: flutter test --coverage

      - name: Publier la couverture
        uses: actions/upload-artifact@v4
        with:
          name: couverture
          path: coverage/lcov.info
```

**Explication :** l'ordre des étapes n'est pas indifférent : on place les vérifications les plus rapides et les moins coûteuses en premier (formatage, puis analyse, puis tests), afin d'obtenir un retour rouge en trente secondes plutôt qu'en cinq minutes. L'option `--set-exit-if-changed` est ce qui transforme `dart format` en **vérification** : sans elle, la commande reformaterait les fichiers sur le serveur et sortirait avec un code de succès, ce qui ne servirait à rien. `cache: true` mémorise le SDK Flutter et les dépendances entre deux exécutions et divise généralement le temps du workflow par deux ou trois. Enfin, l'artefact de couverture rend le rapport téléchargeable depuis l'interface GitHub, ce qui permet de suivre l'évolution du taux sans installer quoi que ce soit localement.

---

### Correction 7

Les objets créés à chaque frame par la version fautive :

```text
  1. Vector2(0, Constantes.gravite)   → l'objet « gravite »
  2. gravite * dt                     → un Vector2 temporaire
  3. velocite + (...)                 → un Vector2 pour le résultat
  4. Vector2(velocite.x, ...)         → dans la branche du plafond
  5. velocite * dt                    → un Vector2 temporaire
  6. position + (...)                 → un Vector2 pour le résultat

  Soit 5 à 6 objets par frame et par composant.
```

La réécriture sans allocation :

```dart
@override
void update(double dt) {
  super.update(dt);

  // Gravité : opération scalaire, aucun objet créé.
  velocite.y += Constantes.gravite * dt;

  // Plafond de chute : on modifie la composante en place.
  if (velocite.y > Constantes.vitesseMaxChute) {
    velocite.y = Constantes.vitesseMaxChute;
  }

  // Déplacement : addScaled fait position += velocite * dt, en place.
  position.addScaled(velocite, dt);
}
```

**Explication :** trois techniques se combinent ici. La gravité devient une opération sur un `double` (`velocite.y += ...`) au lieu d'une addition de vecteurs : un `Vector2` est un objet, un `double` ne l'est pas, et cette seule ligne supprime trois allocations. Le plafond se règle en assignant directement `velocite.y`, sans reconstruire le vecteur. Le déplacement utilise `addScaled(b, k)`, qui calcule `a += b * k` en une passe et sans temporaire. À 200 composants et 60 frames par seconde, la version initiale créait environ 60 000 objets par seconde ; la version corrigée en crée zéro. Le gain se voit immédiatement dans l'onglet Memory du DevTools : les dents de scie disparaissent, et avec elles les pics périodiques de temps de frame.

---

### Correction 8

```dart
// lib/composants/etincelle.dart
import 'package:flame/components.dart';
import 'package:flutter/material.dart';
import 'package:donjon_de_dart/donjon_game.dart';

class Etincelle extends PositionComponent with HasGameReference<DonjonGame> {
  Etincelle() : super(size: Vector2.all(3), anchor: Anchor.center);

  static final Paint _peinture = Paint()..color = const Color(0xFFFFC857);

  final Vector2 velocite = Vector2.zero();
  double vie = 0;
  bool actif = false;

  void lancer(Vector2 depart, Vector2 direction, double duree) {
    position.setFrom(depart);
    velocite.setFrom(direction);
    vie = duree;
    actif = true;
  }

  @override
  void update(double dt) {
    if (!actif) return;
    super.update(dt);
    position.addScaled(velocite, dt);
    velocite.y += 400 * dt;            // légère gravité
    vie -= dt;
    if (vie <= 0) {
      actif = false;
      game.poolEtincelles.rendre(this);
    }
  }

  @override
  void render(Canvas canvas) {
    if (!actif) return;
    canvas.drawCircle(Offset(size.x / 2, size.y / 2), size.x / 2, _peinture);
  }
}
```

```dart
// dans DonjonGame
late final Pool<Etincelle> poolEtincelles;
final math.Random _alea = math.Random();

@override
Future<void> onLoad() async {
  await super.onLoad();
  poolEtincelles = Pool<Etincelle>(
    creer: Etincelle.new,
    reinitialiser: (Etincelle e) {
      e.velocite.setZero();
      e.position.setZero();
      e.vie = 0;
      e.actif = false;
    },
    tailleInitiale: 64,
  );
}

Future<void> exploser(Vector2 position, int nombre) async {
  for (int i = 0; i < nombre; i++) {
    final Etincelle e = poolEtincelles.obtenir();
    final double angle = _alea.nextDouble() * math.pi * 2;
    final double force = 60 + _alea.nextDouble() * 90;
    e.lancer(
      position,
      Vector2(math.cos(angle), math.sin(angle))..scale(force),
      0.4 + _alea.nextDouble() * 0.3,
    );
    if (e.parent == null) {
      await world.add(e);
    }
  }
}
```

```dart
// test/composants/pool_etincelles_test.dart
testWithGame<DonjonGame>(
  'deux explosions de 20 étincelles ne créent pas plus de 64 objets',
  DonjonGame.new,
  (DonjonGame game) async {
    await game.ready();

    await game.exploser(Vector2(100, 100), 20);
    simuler(game, 1.0);          // les étincelles meurent et sont rendues
    await game.exploser(Vector2(200, 200), 20);
    await game.ready();

    final int total = game.world.children.whereType<Etincelle>().length;
    expect(total, lessThanOrEqualTo(64));
  },
);
```

**Explication :** l'étincelle ne se retire jamais de l'arbre. Elle bascule `actif` à `false`, se rend à la pool, et ses `update`/`render` sortent immédiatement par le `if (!actif) return;` placé **avant** l'appel à `super`. Un objet dormant coûte donc un test booléen par frame, ce qui est négligeable. La fonction `reinitialiser` liste tous les champs mutables — `velocite`, `position`, `vie`, `actif` — et c'est la condition de correction du dispositif : une étincelle recyclée qui conserverait son ancienne `velocite` repartirait dans la mauvaise direction, produisant un bug qui n'apparaît qu'à partir de la deuxième explosion. Le test, enfin, simule une seconde entre les deux explosions pour laisser aux étincelles le temps de mourir : c'est cette simulation qui prouve que le recyclage fonctionne, puisque sans lui le total atteindrait 40 objets créés de toutes pièces et croîtrait à chaque explosion.

---

### Correction 9

```bash
# 1. Créer le keystore (macOS / Linux)
keytool -genkey -v -keystore ~/upload-keystore.jks -keyalg RSA \
        -storetype JKS -keysize 2048 -validity 10000 -alias upload
```

```text
# 2. android/key.properties
storePassword=MotDePasseDuMagasin
keyPassword=MotDePasseDeLaCle
keyAlias=upload
storeFile=/Users/moi/upload-keystore.jks
```

```text
# 2 bis. .gitignore — À FAIRE IMMÉDIATEMENT
android/key.properties
*.jks
*.keystore
```

```groovy
// 3. android/app/build.gradle
import java.util.Properties
import java.io.FileInputStream

def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
    namespace = "com.votrenom.donjondedart"

    defaultConfig {
        applicationId = "com.votrenom.donjondedart"   // 4.
        versionCode = flutter.versionCode
        versionName = flutter.versionName
    }

    signingConfigs {
        release {
            keyAlias = keystoreProperties['keyAlias']
            keyPassword = keystoreProperties['keyPassword']
            storeFile = keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword = keystoreProperties['storePassword']
        }
    }
    buildTypes {
        release {
            signingConfig = signingConfigs.release
        }
    }
}
```

```yaml
# 5. pubspec.yaml
version: 1.0.0+1
```

```bash
# 6. Les deux formats
flutter clean
flutter pub get
flutter build appbundle
flutter build apk --split-per-abi

# 7. Installation sur l'appareil branché
flutter install
```

**Résultat attendu :**

```text
✓ Built build/app/outputs/bundle/release/app.aab (18.4MB)
✓ Built build/app/outputs/flutter-apk/app-armeabi-v7a-release.apk (8.2MB)
✓ Built build/app/outputs/flutter-apk/app-arm64-v8a-release.apk (8.7MB)
✓ Built build/app/outputs/flutter-apk/app-x86_64-release.apk (8.9MB)
```

**Explication :** l'ordre des opérations est celui du risque décroissant. Le `.gitignore` est modifié **avant** même que `key.properties` ne contienne des mots de passe, parce qu'un secret committé reste dans l'historique Git même après suppression : le seul remède serait alors de révoquer la clé. Le test `if (keystorePropertiesFile.exists())` permet à un collaborateur sans keystore de compiler quand même en debug, ce qui évite de bloquer toute l'équipe. `flutter clean` avant le premier build de release évite les incohérences dues à des artefacts de debug résiduels — c'est la première chose à essayer devant une erreur Gradle incompréhensible. Enfin, le `.aab` est destiné au Play Store et n'est pas installable directement : c'est l'APK `arm64-v8a` que l'on pose sur l'appareil pour la vérification finale, et il faut réellement terminer une partie, pas seulement voir le menu s'afficher.

---

### Correction 10

```bash
# 1. Compilation avec le chemin de base du dépôt
flutter build web --release --base-href /donjon-de-dart/
```

```bash
# 2. Publication manuelle sur la branche gh-pages
git checkout --orphan gh-pages
git rm -rf .
cp -r build/web/* .
touch .nojekyll
git add .
git commit -m "Déploiement de la version Web 1.0.0"
git push origin gh-pages --force
git checkout main
```

```yaml
# 3. .github/workflows/deploy.yml
name: Déployer sur GitHub Pages

on:
  push:
    branches: [main]

permissions:
  contents: write

jobs:
  deployer:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: subosito/flutter-action@v2
        with:
          channel: stable
          cache: true

      - run: flutter pub get

      - run: flutter build web --release --base-href /donjon-de-dart/

      - name: Publier
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./build/web
```

```dart
// 5. Compteur de FPS, uniquement en debug
import 'package:flutter/foundation.dart';

@override
Future<void> onLoad() async {
  await super.onLoad();
  if (kDebugMode) {
    await camera.viewport.add(FpsTextComponent(position: Vector2(8, 8)));
  }
}
```

**Explication :** le `--base-href /donjon-de-dart/` est la clé de tout l'exercice, et ses deux slashs sont obligatoires : l'outil Flutter refuse une valeur qui ne commence et ne finit pas par `/`. Sans cette option, le navigateur chercherait `main.dart.js` à la racine du domaine `github.io` au lieu du sous-répertoire du dépôt, et vous obtiendriez une page blanche avec des erreurs 404 dans l'onglet Network — d'où l'étape 4, qui est une vérification et non une formalité. Le fichier `.nojekyll` empêche GitHub Pages de traiter le site avec Jekyll, lequel ignore les dossiers commençant par un underscore et peut faire disparaître certains fichiers générés. Le workflow rend le déploiement idempotent : chaque `push` sur `main` régénère et republie, ce qui supprime toute possibilité d'oublier une étape. Enfin, le compteur de FPS est protégé par `kDebugMode` : mesurer est indispensable, mais le compteur ne doit pas s'afficher chez vos joueurs.

---

## Et maintenant ?

Vous venez de terminer le dernier chapitre de cette formation.

Regardez d'où vous partez. Au chapitre 1, vous écriviez `print('Bonjour')` dans un onglet de navigateur. Vous venez d'écrire une suite de tests automatiques, de mesurer un temps de frame en microsecondes, de supprimer des allocations dans une boucle appelée soixante fois par seconde, de signer une application avec une clé cryptographique et de déployer un jeu sur deux plateformes.

Entre les deux, il y a eu quarante-deux chapitres, un langage complet, un framework, un moteur de jeu, et un donjon.

**Ce que vous savez faire, et qui ne s'oublie pas.** Vous savez modéliser un problème en classes, gérer l'absence de valeur sans plantage, traiter une collection sans écrire une boucle, attendre un résultat asynchrone sans bloquer l'interface. Vous savez ce qu'est une boucle de jeu, pourquoi le `dt` existe, comment une collision se détecte et se résout, ce que fait une caméra. Vous savez organiser un projet, spécifier avant de coder, tester ce qui est testable, mesurer avant d'optimiser, et livrer. Ces compétences ne sont pas propres à Flutter ni à Flame : elles se transposent à n'importe quel moteur et à n'importe quel langage.

**Ce qu'il vous reste à parcourir.** Cette formation comporte deux volets que vous n'avez pas encore ouverts, et ils ne sont pas accessoires.

La **PARTIE 1B — Flutter complet** traite du framework pour lui-même, en dehors du jeu : la bibliothèque de widgets dans son ensemble, la mise en page, la navigation entre écrans, les formulaires, la gestion d'état, les thèmes, l'accès au réseau, la persistance, l'internationalisation, l'accessibilité. Tout ce que le chapitre 19 a survolé en accéléré parce qu'il fallait arriver vite au jeu. Si votre objectif est de travailler comme développeur Flutter, cette partie n'est pas optionnelle : c'est même le cœur du métier.

La **PARTIE 1C — Mini-projets** propose des projets courts et complets, chacun centré sur un besoin réel : une application de liste, un client d'API, un formulaire validé, un outil hors ligne. Ce sont ces projets, plus qu'un jeu, qui remplissent un portfolio et alimentent une conversation d'embauche.

**Le conseil final.** Ne relisez pas ce chapitre. Ouvrez votre projet et faites-en quelque chose de réel : ajoutez un niveau, corrigez le bug que vous connaissez et que vous repoussez, écrivez les cinq tests qui manquent, mettez-le en ligne, envoyez le lien à trois personnes, et regardez-les jouer sans rien dire.

C'est en regardant quelqu'un jouer à votre jeu que vous apprendrez ce qu'aucun chapitre ne peut vous enseigner.

Bonne route.

Pour continuer :

- [00-PARTIE-1A—PLAN-COMPLET-DE-LA-FORMATION.md](./00-PARTIE-1A—PLAN-COMPLET-DE-LA-FORMATION.md) — le plan général de la formation
- [41-PARTIE-2D—CAHIER-DES-CHARGES-ET-GAME-DESIGN-DOCUMENT.md](./41-PARTIE-2D—CAHIER-DES-CHARGES-ET-GAME-DESIGN-DOCUMENT.md) — le chapitre précédent
