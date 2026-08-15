# PARTIE 1C — MINI-PROJETS FLUTTER
# CHAPITRE 56 — PROJET 2 : LA CALCULATRICE

> **Niveau :** intermédiaire
> **Durée estimée :** 12 h
> **Pré-requis :** chapitres 08, 09, 13, 16 (Dart) et 45, 46, 48, 49, 51 (Flutter), plus le chapitre 55 (projet 1)
> **Ce que vous saurez faire à la fin :** écrire une application Flutter dont le cœur métier est une classe Dart pure, testée unitairement, et dont l'interface n'est qu'une couche d'affichage branchée dessus.

---

## 56.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- transformer un cahier des charges en automate d'états avant d'écrire la moindre ligne ;
- isoler la logique métier dans une classe Dart **sans aucun `import 'package:flutter/...'`** ;
- modéliser la saisie chiffre par chiffre d'un nombre, séparateur décimal compris ;
- enchaîner des opérations comme une vraie calculatrice de poche, gérer la répétition de `=`, `C`, `CE`, le signe et le pourcentage ;
- lever et rattraper une exception métier (`DivisionParZeroException`) ;
- formater un `double` pour un affichage humain : séparateur de milliers, virgule décimale, notation scientifique ;
- empêcher un texte trop long de faire déborder l'écran avec `FittedBox` ;
- construire une grille de boutons avec `Row` et `Expanded`, sans `GridView` ;
- écrire un widget de bouton réutilisable, paramétré par un `enum` et coloré par le `ColorScheme` ;
- produire un effet d'appui correct avec `Material` + `InkWell` ;
- brancher la logique sur l'interface avec un unique `setState`, et afficher un historique dans un `ListView` ;
- accepter le clavier physique avec `KeyboardListener` et `LogicalKeyboardKey` ;
- écrire les tests unitaires de la logique, exécutables sans émulateur.

---


## 56.1 — Aperçu du résultat final

Voici ce que vous aurez à la fin du chapitre. Tout ce que nous allons construire figure dans ces deux maquettes.

**Écran principal**

```text
┌────────────────────────────────────────┐
│  Calculatrice                      [⟲] │  <- AppBar + bouton « historique »
├────────────────────────────────────────┤
│                                        │
│                              12,5 +    │  <- ligne d'expression (petite)
│                                 16,5   │  <- écran principal (grand)
│                                        │
├────────────────────────────────────────┤
│  (  C  ) (  CE ) (  %  ) (  ÷  )       │
│  (  7  ) (  8  ) (  9  ) (  ×  )       │
│  (  4  ) (  5  ) (  6  ) (  -  )       │
│  (  1  ) (  2  ) (  3  ) (  +  )       │
│  (  ±  ) (  0  ) (  ,  ) (  =  )       │
└────────────────────────────────────────┘
```

**Panneau d'historique** (ouvert par le bouton de l'`AppBar`)

```text
┌────────────────────────────────────────┐
│  Historique                    [vider] │
├────────────────────────────────────────┤
│  200 + 20                          220 │
│  45 ÷ 100                         0,45 │
│  0,45 × 20                           9 │
│  1 234 567 + 7 654 321       8 888 888 │
└────────────────────────────────────────┘
```

Trois détails de ces maquettes sont déjà des décisions techniques :

1. **Le séparateur de milliers est une espace** (`1 234 567`), pas une virgule. C'est la convention française.
2. **Le séparateur décimal est une virgule** (`0,45`). Or, en Dart, un `double` s'écrit avec un point. Il faudra donc traduire.
3. **La ligne d'expression est distincte de l'écran principal.** Elle montre le calcul en attente, ce qui évite à l'utilisateur de se demander « qu'est-ce que j'ai tapé, déjà ? ».

---


## 56.2 — Cahier des charges

### 56.2.1 — Fonctionnalités obligatoires

| # | Exigence | Critère d'acceptation |
| --- | --- | --- |
| O1 | Saisir un nombre chiffre par chiffre | Taper `1`, `2`, `3` affiche `123` |
| O2 | Limiter la saisie | Au-delà de 12 chiffres, les appuis suivants sont ignorés |
| O3 | Saisir un nombre décimal | Taper `1`, `,`, `5` affiche `1,5` |
| O4 | Un seul séparateur décimal | Un deuxième appui sur `,` ne fait rien |
| O5 | Les quatre opérations | `+`, `-`, `×`, `÷` |
| O6 | Refuser la division par zéro | `5 ÷ 0 =` affiche `Division par zéro` |
| O7 | Enchaîner les opérations | `2 + 3 × 4 =` affiche `20` (calcul à plat, de gauche à droite) |
| O8 | Répéter avec `=` | `2 + 3 = = =` affiche `5`, puis `8`, puis `11` |
| O9 | Tout effacer (`C`) | Remet la calculatrice à son état initial, historique compris |
| O10 | Effacer l'entrée (`CE`) | Remet l'écran à `0` sans perdre le calcul en attente |
| O11 | Changer de signe (`±`) | `9`, `±` affiche `-9` |
| O12 | Pourcentage (`%`) | `200 + 10 %` affiche `20`, puis `=` affiche `220` |
| O13 | Formater l'affichage | `1234567` s'affiche `1 234 567` |
| O14 | Ne jamais déborder de l'écran | Un nombre de 12 chiffres reste lisible, réduit s'il le faut |
| O15 | Grille de boutons responsive | La grille occupe toute la largeur, quelle que soit la taille de l'écran |
| O16 | Retour visuel à l'appui | Chaque bouton produit une onde Material |
| O17 | Historique | Les 50 derniers calculs sont consultables |
| O18 | Clavier physique | Sur bureau et web, les touches du pavé numérique fonctionnent |
| O19 | Logique testée | La classe `Calculatrice` est couverte par des tests `flutter test` |
| O20 | Zéro Flutter dans la logique | Le fichier `calculatrice.dart` ne contient aucun `import` de Flutter |

### 56.2.2 — Fonctionnalités bonus

| # | Bonus |
| --- | --- |
| B1 | Retour arrière (touche `Backspace`) |
| B2 | Mode sombre suivant le réglage du système |
| B3 | Réutilisation d'une ligne d'historique par simple appui |
| B4 | Copie du résultat dans le presse-papiers |

### 56.2.3 — Hors périmètre

Nous ne faisons **pas** :

- la priorité des opérateurs (`2 + 3 × 4` vaudra `20`, pas `14`) ;
- les parenthèses ;
- les fonctions scientifiques (`sin`, `log`, racine carrée) ;
- la mémoire (`M+`, `M-`, `MR`).

Ces quatre points sont proposés comme extensions en 56.26. Une calculatrice de poche à quatre opérations calcule bel et bien **à plat**, de gauche à droite : ce n'est pas un bug, c'est le comportement attendu d'un tel appareil.

---

## 56.3 — Notions mobilisées

| Notion | Chapitre d'origine | Usage dans ce projet |
| --- | --- | --- |
| Classes, champs privés, getters | 08 | La classe `Calculatrice` et son état interne |
| Constructeurs, modélisation | 09 | `LigneHistorique`, constructeurs `const` |
| Conditions, `switch` | 04 | Le dispatch des touches |
| Collections `List` | 06 | La liste d'historique |
| Exceptions personnalisées | 13 | `DivisionParZeroException` |
| Enums enrichis | 11 | `Operation` et `TypeBouton` |
| Null safety, `?`, `??`, `!` | 12 | `double? _accumulateur`, `Operation? _operationEnAttente` |
| Organisation d'un projet, tests | 16 | Dossiers `lib/logique`, `lib/ui`, `test/` |
| `StatefulWidget`, `setState` | 45 | La page qui pilote la calculatrice |
| `Row`, `Column`, `Expanded`, `FittedBox` | 46 | La grille de boutons et l'écran |
| `ListView.separated` | 48 | Le panneau d'historique |
| `InkWell`, boutons | 49 | Le retour visuel à l'appui |
| `ThemeData`, `ColorScheme`, Material 3 | 51 | Les couleurs par type de bouton, le mode sombre |

> Ce projet ne demande **aucun package externe**. Tout tient dans le SDK Flutter et dans `dart:core`.

---

## 56.4 — Arborescence du projet

Voici la cible. Nous la construirons fichier par fichier.

```text
calculatrice/
├── pubspec.yaml
├── lib/
│   ├── main.dart                        <- point d'entrée, thème
│   ├── logique/                          <- DART PUR : aucun import Flutter
│   │   ├── operation.dart                <- enum des 4 opérations
│   │   ├── erreurs.dart                  <- DivisionParZeroException
│   │   ├── formatage.dart                <- double -> texte affichable
│   │   └── calculatrice.dart             <- LE cœur du projet
│   ├── modeles/
│   │   └── ligne_historique.dart         <- une ligne « expression = résultat »
│   └── ui/                               <- FLUTTER : aucune règle de calcul
│       ├── page_calculatrice.dart        <- StatefulWidget, setState, clavier
│       ├── ecran_calculatrice.dart       <- la zone d'affichage
│       ├── clavier_calculatrice.dart     <- la grille 5 x 4
│       ├── bouton_calc.dart              <- un bouton réutilisable
│       └── panneau_historique.dart       <- la feuille d'historique
└── test/
    └── calculatrice_test.dart            <- les tests unitaires
```

La règle d'or de ce projet tient en une phrase :

> **Rien dans `lib/logique/` ne doit connaître Flutter. Rien dans `lib/ui/` ne doit calculer.**

Respectez cette frontière et vous pourrez remplacer entièrement l'interface (ligne de commande, montre connectée, site web) sans toucher une virgule au calcul. C'est ce que le chapitre 16 appelait « séparer le domaine de la présentation ».

```text
   lib/ui/                                lib/logique/
   ────────────────────────               ──────────────────────────
   PageCalculatrice   ── appelle ──>      chiffre('7'), operateur(),
   (StatefulWidget)                       egal(), effacerTout()...
                      <── lit ──          affichage, expressionEnCours,
                                          historique, enErreur
```

La flèche ne va que dans un sens : l'interface dépend de la logique, jamais l'inverse.

---


## 56.5 — Analyse du besoin : la calculatrice est un automate

Avant tout code, répondons à une question : **quel est l'état d'une calculatrice ?**

Regardez une calculatrice de poche. À un instant donné, elle « se souvient » de quatre choses :

| Champ | Rôle | Exemple |
| --- | --- | --- |
| `_saisie` | ce qui est écrit à l'écran, sous forme de **texte** | `"12.5"` |
| `_accumulateur` | l'opérande de gauche du calcul en attente | `7.0` |
| `_operationEnAttente` | l'opération choisie, pas encore effectuée | `+` |
| `_nouvelleSaisie` | « le prochain chiffre commence-t-il un nouveau nombre ? » | `true` |

Le quatrième point est le plus subtil, et c'est celui que ratent 90 % des débutants. Déroulons `7 + 8` :

```text
appui   _saisie   _accumulateur   _operationEnAttente   _nouvelleSaisie
─────   ───────   ─────────────   ───────────────────   ───────────────
(init)  "0"       null            null                  true
  7     "7"       null            null                  false
  +     "7"       7.0             addition              true
  8     "8"       7.0             addition              false
  =     "15"      15.0            null                  true
```

Observez la ligne du `+` : l'écran continue d'afficher `7`, mais `_nouvelleSaisie` est repassé à `true`. Résultat : quand l'utilisateur tape `8`, le `8` **remplace** le `7` au lieu de s'y coller. Sans ce drapeau, on afficherait `78`.

C'est pour cela que la saisie est stockée en **texte** et non en `double` : `"1.50"` et `"1.5"` valent le même `double`, mais ne s'affichent pas pareil pendant que l'utilisateur tape. Le texte est la vérité de l'écran, le `double` est la vérité du calcul.

Nous ajouterons ensuite deux champs pour la répétition de `=`, et un pour le message d'erreur. Cela fera sept champs en tout. C'est peu, et c'est suffisant.

---


## 56.6 — Étape 1 : créer le projet et vérifier qu'il démarre

Nous commençons par un projet vide qui compile. Chaque étape se terminera de la même façon : par un état exécutable.

```bash
flutter create calculatrice
cd calculatrice
mkdir -p lib/logique lib/modeles lib/ui bin
flutter run
```

Vous n'avez **rien** à ajouter au `pubspec.yaml` : ce projet n'utilise aucun package externe.

**Fichier : `pubspec.yaml`**

```yaml
name: calculatrice
description: "Projet 2 de la PARTIE 1C : une calculatrice à quatre opérations."
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: ^3.12.0

dependencies:
  flutter:
    sdk: flutter

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^6.0.0

flutter:
  uses-material-design: true
```

> `flutter_lints` est déjà présent dans le projet généré par `flutter create`. Ne modifiez pas son numéro de version : celui que `flutter create` a écrit est le bon pour votre SDK.

**État exécutable :** `flutter run` affiche le compteur d'exemple de Flutter. Nous allons le remplacer, mais le projet est prêt.

---


## 56.7 — Étape 2 : l'enum `Operation` et l'exception métier

Deux petits fichiers, mais ils fixent le vocabulaire de tout le projet.

Pourquoi un `enum` plutôt qu'une `String` `'+'` ? Parce qu'avec une `String`, rien n'empêche d'écrire `operateur('plus')` et de s'en apercevoir à l'exécution. Avec un `enum`, le compilateur refuse : **un ensemble fini de valeurs se modélise par un `enum`** (chapitre 11).

**Fichier : `lib/logique/operation.dart`**

```dart
/// Les quatre opérations que la calculatrice sait faire.
///
/// C'est un « enum enrichi » (chapitre 11) : chaque valeur transporte
/// le symbole que l'on affichera à l'écran.
enum Operation {
  addition('+'),
  soustraction('-'),
  multiplication('×'), // × (signe multiplication, PAS la lettre x)
  division('÷'); // ÷ (signe division)

  /// Constructeur const de l'enum : il reçoit le symbole d'affichage.
  const Operation(this.symbole);

  /// Le caractère montré à l'utilisateur, sur le bouton et dans l'historique.
  final String symbole;
}
```

**Fichier : `lib/logique/erreurs.dart`**

```dart
/// Levée quand l'utilisateur demande une division par zéro.
///
/// Rappel du chapitre 13 : on crée une exception métier dédiée plutôt que
/// de renvoyer une valeur spéciale (null, -1, NaN...). L'appelant est ainsi
/// *obligé* de traiter le cas, et le message est écrit une seule fois.
class DivisionParZeroException implements Exception {
  const DivisionParZeroException();

  /// Le texte affiché à l'utilisateur.
  String get message => 'Division par zéro';

  @override
  String toString() => 'DivisionParZeroException: $message';
}
```

Pourquoi lever une exception alors qu'en Dart `5.0 / 0.0` ne plante pas ? C'est justement le problème. Testez dans DartPad :

```dart
void main() {
  print(5.0 / 0.0);
  print(-5.0 / 0.0);
  print(0.0 / 0.0);
}
```

**Résultat :**

```text
Infinity
-Infinity
NaN
```

Les `double` suivent la norme IEEE 754 : la division par zéro ne lève **aucune** erreur, elle produit `Infinity` ou `NaN`. Ces valeurs contaminent ensuite tous les calculs suivants (`Infinity + 1` vaut `Infinity`). Si nous ne testons pas nous-mêmes le diviseur, l'utilisateur verra `Infinity` sans comprendre pourquoi. Nous décidons donc explicitement d'interdire l'opération.

**État exécutable :** créez un dossier `bin/` et un fichier de brouillon `bin/essai.dart` (temporaire, supprimé à la fin du chapitre) qui affiche `Operation.values` et `const DivisionParZeroException()`.

```bash
dart run bin/essai.dart
```

**Résultat :**

```text
addition -> +
soustraction -> -
multiplication -> ×
division -> ÷
DivisionParZeroException: Division par zéro
```

Notez que nous lançons `dart run`, pas `flutter run` : ces fichiers sont du **Dart pur**. C'est la première preuve que la séparation tient.

---


## 56.8 — Étape 3 : le formatage, ou comment un `double` devient lisible

Avant d'écrire la calculatrice, réglons le problème de l'affichage. C'est un problème isolé, donc un fichier isolé.

### 56.8.1 — Trois pièges à éviter

**Piège 1 — `toString()` colle un `.0` partout.** `(7.0 * 8.0).toString()` vaut `"56.0"` et `(100.0 / 4.0).toString()` vaut `"25.0"`. Personne n'écrit `56.0` sur une calculatrice.

**Piège 2 — les flottants traînent des poussières binaires.**

```dart
void main() {
  print(0.1 + 0.2);
  print(1.0 - 0.9);
}
```

**Résultat :**

```text
0.30000000000000004
0.09999999999999998
```

Ce n'est pas un bug de Dart : `0,1` et `0,2` n'ont pas d'écriture binaire exacte, exactement comme `1/3` n'a pas d'écriture décimale exacte. Une calculatrice doit afficher `0,3`, donc il faut **arrondir à un nombre fixe de chiffres significatifs**.

**Piège 3 — les très grands nombres.** `123456789 × 987654321` vaut environ 1,2 × 10^17. Aucun écran de téléphone ne peut afficher 18 chiffres lisiblement : il faut basculer en notation scientifique.


### 56.8.2 — La solution : douze chiffres significatifs

Nous fixons une règle unique : **la calculatrice travaille à 12 chiffres significatifs**. C'est la précision d'une calculatrice de poche classique, et c'est bien en dessous des ~15,9 chiffres qu'un `double` garantit. Arrondir à 12 fait donc disparaître les poussières binaires du piège 2.

L'outil est `toStringAsPrecision(12)`, méthode de `num` en Dart :

| Appel | Résultat |
| --- | --- |
| `(0.30000000000000004).toStringAsPrecision(12)` | `0.300000000000` |
| `(1 / 3).toStringAsPrecision(12)` | `0.333333333333` |
| `(2 / 3).toStringAsPrecision(12)` | `0.666666666667` |
| `(1e20).toStringAsPrecision(12)` | `1.00000000000e+20` |

Il reste ensuite à **retirer les zéros inutiles** à droite : `0.300000000000` doit devenir `0.3`.

### 56.8.3 — Le fichier de formatage

**Fichier : `lib/logique/formatage.dart`**

```dart
/// Conversion d'un nombre en texte affichable, et rien d'autre.
///
/// Aucune dépendance : ni Flutter, ni le package `intl`. Tout est fait
/// à la main pour que vous compreniez chaque étape.
class Formatage {
  // Constructeur privé : cette classe n'est qu'une boîte à fonctions,
  // on ne doit jamais en créer une instance.
  const Formatage._();

  /// Nombre de chiffres significatifs conservés dans les calculs.
  static const int chiffresSignificatifs = 12;

  /// Le séparateur décimal montré à l'utilisateur (convention française).
  static const String separateurDecimal = ',';

  /// Le séparateur de milliers : une espace insécable (U+00A0), pour que
  /// le nombre ne soit jamais coupé en deux en fin de ligne.
  static const String separateurMilliers = ' ';

  /// Convertit un `double` en texte « technique » : point décimal, pas de
  /// séparateur de milliers. C'est la forme stockée par la calculatrice.
  ///
  ///   56.0                 -> "56"
  ///   0.30000000000000004  -> "0.3"
  ///   1 / 3                -> "0.333333333333"
  ///   1e20                 -> "1e+20"
  static String versTexteBrut(double valeur) {
    // Gardes : ces cas ne doivent jamais arriver puisque nous refusons la
    // division par zéro, mais elles ne coûtent rien.
    if (valeur.isNaN || valeur.isInfinite) {
      return 'Erreur';
    }
    // -0.0 existe en IEEE 754 et s'afficherait « -0 ». On le neutralise.
    if (valeur == 0) {
      return '0';
    }

    // Cas 1 : un entier assez petit pour être écrit en entier. Si la
    // valeur arrondie est égale à la valeur d'origine, il n'y a pas de
    // partie décimale.
    if (valeur == valeur.roundToDouble() && valeur.abs() < 1e15) {
      return valeur.toStringAsFixed(0);
    }

    // Cas 2 : on arrondit à 12 chiffres significatifs.
    String texte = valeur.toStringAsPrecision(chiffresSignificatifs);

    // Cas 2a : Dart a choisi la notation scientifique (nombre très grand
    // ou très petit). On nettoie seulement la mantisse.
    if (texte.contains('e')) {
      final List<String> parties = texte.split('e');
      String mantisse = parties[0];
      if (mantisse.contains('.')) {
        mantisse = _sansZerosFinaux(mantisse);
      }
      return '${mantisse}e${parties[1]}';
    }

    // Cas 2b : notation décimale normale, on retire les zéros inutiles.
    if (texte.contains('.')) {
      texte = _sansZerosFinaux(texte);
    }
    return texte;
  }

  /// Retire les zéros de droite, puis le point s'il ne reste rien après.
  ///   "0.300000000000" -> "0.3"   |   "56.0000000000" -> "56"
  static String _sansZerosFinaux(String texte) {
    String resultat = texte.replaceFirst(RegExp(r'0+$'), '');
    if (resultat.endsWith('.')) {
      resultat = resultat.substring(0, resultat.length - 1);
    }
    return resultat;
  }

  /// Convertit le texte technique en texte affichable : point décimal ->
  /// virgule, et espaces tous les trois chiffres.
  ///
  ///   "1234567" -> "1 234 567"   |   "0.45" -> "0,45"
  ///   "12."     -> "12,"         (la virgule vient d'être tapée)
  static String versAffichage(String brut) {
    if (brut.isEmpty) {
      return '0';
    }

    // On met le signe de côté pour ne pas le compter comme un chiffre.
    final bool negatif = brut.startsWith('-');
    final String corps = negatif ? brut.substring(1) : brut;

    // La notation scientifique n'est pas groupée par milliers.
    if (corps.contains('e')) {
      return brut;
    }

    // Séparation partie entière / partie décimale.
    final int position = corps.indexOf('.');
    final String entiere = position < 0 ? corps : corps.substring(0, position);
    final String? decimale =
        position < 0 ? null : corps.substring(position + 1);

    // Groupement par tranches de trois, en partant de la droite.
    final StringBuffer tampon = StringBuffer();
    for (int i = 0; i < entiere.length; i++) {
      if (i > 0 && (entiere.length - i) % 3 == 0) {
        tampon.write(separateurMilliers);
      }
      tampon.write(entiere[i]);
    }

    // Recomposition.
    final StringBuffer sortie = StringBuffer();
    if (negatif) {
      sortie.write('-');
    }
    sortie.write(tampon);
    if (decimale != null) {
      sortie.write(separateurDecimal);
      sortie.write(decimale);
    }
    return sortie.toString();
  }
}
```


### 56.8.4 — Vérifier le formatage

Dans `bin/essai.dart`, affichez `Formatage.versTexteBrut(v)` puis `Formatage.versAffichage(...)` pour une liste de valeurs. Les résultats ci-dessous sont exacts.

| `double` d'entrée | `versTexteBrut` | `versAffichage` |
| --- | --- | --- |
| `56.0` | `56` | `56` |
| `0.1 + 0.2` | `0.3` | `0,3` |
| `1.0 / 3.0` | `0.333333333333` | `0,333333333333` |
| `2.0 / 3.0` | `0.666666666667` | `0,666666666667` |
| `100.0 / 8.0` | `12.5` | `12,5` |
| `8888888.0` | `8888888` | `8 888 888` |
| `123456789.0 * 987654321.0` | `1.21932631113e+17` | `1.21932631113e+17` |
| `-9.0` | `-9` | `-9` |

Les trois pièges sont réglés : plus de `.0`, plus de `0.30000000000000004`, et les très grands nombres basculent tout seuls en notation scientifique.

**État exécutable :** `dart run bin/essai.dart` reproduit exactement ce tableau.

---


## 56.9 — Étape 4 : la classe `Calculatrice`, saisie chiffre par chiffre

Nous attaquons le cœur du projet. Première version : elle sait seulement saisir un nombre.

### 56.9.1 — Pourquoi stocker la saisie en `String`

Un débutant écrirait `_valeur = _valeur * 10 + c;` avec un `double`. Cela marche pour `123`. Cela ne marche plus du tout pour `1,05` : après `1`, `,`, `0`, il faudrait afficher `1,0`, mais `1,0` et `1` valent le même `double`. L'information « l'utilisateur a déjà tapé un zéro après la virgule » n'existe nulle part.

> Pendant la saisie, la vérité est un **texte**. Le `double` n'apparaît qu'au moment du calcul.

### 56.9.2 — Les règles de la saisie

| Situation | Appui | Résultat |
| --- | --- | --- |
| `_nouvelleSaisie == true` | `5` | La saisie devient `5` |
| Saisie vaut `0` | `5` | La saisie devient `5` (pas `05`) |
| Saisie vaut `-0` | `5` | La saisie devient `-5` |
| Saisie vaut `12` | `5` | La saisie devient `125` |
| Saisie a déjà 12 chiffres | `5` | Rien |
| `_nouvelleSaisie == true` | `,` | La saisie devient `0.` |
| Saisie vaut `12` | `,` | La saisie devient `12.` |
| Saisie contient déjà un `.` | `,` | Rien |


### 56.9.3 — Le code

**Fichier : `lib/logique/calculatrice.dart`** (première version)

```dart
import 'package:calculatrice/logique/formatage.dart';

/// Le moteur de la calculatrice.
///
/// AUCUN import de Flutter ici, et il n'y en aura jamais : cette classe
/// se teste sans émulateur, en quelques millisecondes.
class Calculatrice {
  /// Nombre maximal de chiffres saisissables.
  static const int maxChiffres = 12;

  /// La saisie en cours, au format technique (point décimal).
  String _saisie = '0';

  /// true tant que le prochain chiffre doit REMPLACER l'affichage.
  bool _nouvelleSaisie = true;

  /// Le texte à afficher à l'écran, prêt à l'emploi.
  String get affichage => Formatage.versAffichage(_saisie);

  /// La valeur numérique de la saisie. Pendant la frappe, le texte peut
  /// valoir "12." ou "-" : ce ne sont pas des nombres valides, on les
  /// nettoie avant de convertir.
  double get valeurCourante {
    String texte = _saisie;
    if (texte.endsWith('.')) {
      texte = texte.substring(0, texte.length - 1);
    }
    if (texte.isEmpty || texte == '-') {
      return 0;
    }
    return double.tryParse(texte) ?? 0;
  }

  /// Le nombre de chiffres saisis (hors signe et hors point).
  int get _nombreDeChiffres => _saisie.replaceAll(RegExp(r'[^0-9]'), '').length;

  /// Appui sur une touche chiffre, de "0" à "9".
  void chiffre(String c) {
    assert(c.length == 1 && '0123456789'.contains(c),
        'chiffre() attend un caractère de "0" à "9", reçu : "$c"');
    if (_nouvelleSaisie) {
      _saisie = c;
      _nouvelleSaisie = false;
      return;
    }
    if (_saisie == '0') {
      _saisie = c; // "0" puis "5" donne "5", surtout pas "05"
      return;
    }
    if (_saisie == '-0') {
      _saisie = '-$c';
      return;
    }
    if (_nombreDeChiffres >= maxChiffres) {
      return; // saisie saturée : on ignore l'appui
    }
    _saisie += c;
  }

  /// Appui sur la touche virgule.
  void separateurDecimal() {
    if (_nouvelleSaisie) {
      _saisie = '0.'; // une calculatrice affiche "0," et non "," seule
      _nouvelleSaisie = false;
      return;
    }
    if (_saisie.contains('.')) {
      return; // un seul séparateur, c'est la règle O4
    }
    _saisie += '.';
  }

  /// Remet la calculatrice à zéro.
  void effacerTout() {
    _saisie = '0';
    _nouvelleSaisie = true;
  }
}
```


### 56.9.4 — Vérifier la saisie

**Fichier : `bin/essai.dart`**

```dart
import 'package:calculatrice/logique/calculatrice.dart';

void main() {
  final Calculatrice calc = Calculatrice();
  for (final String t in <String>['1', '2', ',', '5', ',', '7']) {
    if (t == ',') {
      calc.separateurDecimal();
    } else {
      calc.chiffre(t);
    }
    print('$t  ->  ${calc.affichage}');
  }
}
```

**Résultat :**

```text
1  ->  1
2  ->  12
,  ->  12,
5  ->  12,5
,  ->  12,5
7  ->  12,57
```

Trois observations :

1. Après `1`, `2`, `,` l'écran montre `12,` : la virgule est visible **avant** qu'un chiffre décimal soit tapé. C'est ce que fait une vraie calculatrice, et c'est possible uniquement parce que la saisie est un texte.
2. La deuxième virgule ne change rien.
3. Rejouez la suite `1 2 3 4 5 6 7 8 9 0 1 2 3 4` après un `effacerTout()` : l'écran se fige sur `123 456 789 012`, les deux derniers appuis sont ignorés.

**État exécutable :** `dart run bin/essai.dart` produit la sortie ci-dessus.

---


## 56.10 — Étape 5 : les quatre opérations et la division par zéro

### 56.10.1 — Ce qui se passe quand on appuie sur `+`

Trois cas, et un seul est piégeux.

| Cas | Situation | Appui sur `×` | Effet |
| --- | --- | --- | --- |
| A | aucune opération en attente, saisie `7` | `×` | on met `7` de côté dans l'accumulateur, on retient `×`, l'écran ne bouge pas |
| B | accumulateur `2`, opération `+`, saisie `3` | `×` | on **calcule** `2 + 3 = 5`, on affiche `5`, on retient `×` |
| C | accumulateur `12`, opération `+`, rien de saisi | `×` | l'utilisateur change d'avis : on remplace `+` par `×`, aucun calcul |

Le cas B est ce qui permet d'enchaîner sans jamais appuyer sur `=`. C'est le drapeau `_nouvelleSaisie` qui le distingue du cas C : dans le cas C, aucun nouvel opérande n'a été tapé depuis le choix de l'opération.


### 56.10.2 — Le code

Ajoutez trois champs, `double? _accumulateur`, `Operation? _operationEnAttente` et `String? _messageErreur`, puis trois getters de lecture :

```dart
  /// true si la calculatrice est bloquée sur une erreur.
  bool get enErreur => _messageErreur != null;

  /// Le texte de l'écran principal (le message d'erreur a priorité).
  String get affichage => _messageErreur ?? Formatage.versAffichage(_saisie);

  /// La petite ligne du dessus : "12,5 +", ou "" s'il n'y a rien en attente.
  String get expressionEnCours {
    final double? gauche = _accumulateur;
    final Operation? op = _operationEnAttente;
    if (enErreur || gauche == null || op == null) {
      return '';
    }
    final String texte =
        Formatage.versAffichage(Formatage.versTexteBrut(gauche));
    return '$texte ${op.symbole}';
  }
```

Puis les trois méthodes qui font le travail. Le fichier complet et définitif est donné en 56.14.3.

```dart
  /// Appui sur +, -, × ou ÷.
  void operateur(Operation op) {
    if (enErreur) {
      return; // bloqué tant que l'utilisateur n'a pas appuyé sur C
    }
    if (_operationEnAttente != null && !_nouvelleSaisie) {
      _calculerEnAttente(); // cas B : on solde le calcul précédent
      if (enErreur) {
        return; // une division par zéro vient d'être détectée
      }
    } else {
      _accumulateur = valeurCourante; // cas A et cas C
    }
    _operationEnAttente = op;
    _nouvelleSaisie = true;
  }

  /// Effectue l'opération en attente et place le résultat à l'écran.
  void _calculerEnAttente() {
    final double gauche = _accumulateur!;
    final double droite = valeurCourante;
    final Operation op = _operationEnAttente!;

    final double resultat;
    try {
      resultat = _appliquer(gauche, droite, op);
    } on DivisionParZeroException catch (e) {
      // Rappel du chapitre 13 : on rattrape l'exception là où l'on sait
      // quoi en faire.
      _messageErreur = e.message;
      _accumulateur = null;
      _operationEnAttente = null;
      _nouvelleSaisie = true;
      return;
    }
    _accumulateur = resultat;
    _saisie = Formatage.versTexteBrut(resultat);
    _operationEnAttente = null;
    _nouvelleSaisie = true;
  }

  /// Le calcul pur : deux nombres, une opération, un résultat.
  /// C'est la SEULE méthode du projet qui fait de l'arithmétique.
  double _appliquer(double a, double b, Operation op) {
    switch (op) {
      case Operation.addition:
        return a + b;
      case Operation.soustraction:
        return a - b;
      case Operation.multiplication:
        return a * b;
      case Operation.division:
        if (b == 0) {
          // En IEEE 754, a / 0 vaut Infinity et ne lève rien.
          // C'est nous qui décidons que c'est une erreur.
          throw const DivisionParZeroException();
        }
        return a / b;
    }
  }
```

Trois retouches complètent l'étape. D'abord, un appui sur un chiffre ou sur la virgule pendant une erreur doit repartir de zéro : ajoutez `if (enErreur) { effacerTout(); }` en tête de `chiffre()` et de `separateurDecimal()`. Ensuite, `effacerTout()` doit remettre `_accumulateur`, `_operationEnAttente` et `_messageErreur` à `null`. Enfin, complétez les imports :

```dart
import 'package:calculatrice/logique/erreurs.dart';
import 'package:calculatrice/logique/formatage.dart';
import 'package:calculatrice/logique/operation.dart';
```

> Le `switch` sur un `enum` n'a pas besoin de `default` : le compilateur sait que les quatre valeurs sont couvertes. Si vous ajoutez un jour `Operation.puissance`, il vous signalera immédiatement ce `switch` comme incomplet.


### 56.10.3 — Vérifier l'enchaînement et l'erreur

**Fichier : `bin/essai.dart`**

```dart
import 'package:calculatrice/logique/calculatrice.dart';
import 'package:calculatrice/logique/operation.dart';

/// Joue une suite de touches sur une calculatrice neuve et trace l'écran.
void jouer(String titre, List<String> touches) {
  print('--- $titre ---');
  final Calculatrice calc = Calculatrice();
  for (final String t in touches) {
    switch (t) {
      case '+':
        calc.operateur(Operation.addition);
      case '-':
        calc.operateur(Operation.soustraction);
      case '×':
        calc.operateur(Operation.multiplication);
      case '÷':
        calc.operateur(Operation.division);
      case ',':
        calc.separateurDecimal();
      case 'C':
        calc.effacerTout();
      default:
        calc.chiffre(t);
    }
    final String expr = calc.expressionEnCours;
    print('${t.padRight(3)} -> ${calc.affichage.padRight(20)} '
        '${expr.isEmpty ? '' : '[$expr]'}');
  }
}

void main() {
  jouer('2 + 3 + (le second + calcule)', <String>['2', '+', '3', '+']);
  jouer('5 ÷ 0 ÷ (division par zéro)', <String>['5', '÷', '0', '÷']);
}
```

**Résultat :**

```text
--- 2 + 3 + (le second + calcule) ---
2   -> 2
+   -> 2                    [2 +]
3   -> 3                    [2 +]
+   -> 5                    [5 +]
--- 5 ÷ 0 ÷ (division par zéro) ---
5   -> 5
÷   -> 5                    [5 ÷]
0   -> 0                    [5 ÷]
÷   -> Division par zéro
```

> Le `switch` de `jouer()` n'a pas de `break` : depuis Dart 3, chaque `case` d'un `switch` d'instructions s'arrête tout seul dès que son corps n'est pas vide.

**État exécutable :** `dart run bin/essai.dart` produit la sortie ci-dessus.

---


## 56.11 — Étape 6 : la touche `=` et sa répétition

### 56.11.1 — Le comportement attendu

Tapez `2 + 3 =` sur n'importe quelle calculatrice, puis appuyez encore sur `=` : vous obtenez `5`, puis `8`, puis `11`, puis `14`. La calculatrice **rejoue la dernière opération avec le dernier opérande**. Il faut donc mémoriser deux choses de plus : `_derniereOperation` (ici `Operation.addition`) et `_dernierOperande` (ici `3.0`).

Attention au sens : à la répétition, l'opérande de **gauche** est le résultat affiché, et l'opérande de **droite** est le `3` mémorisé. `5 + 3`, pas `3 + 5`. Pour l'addition cela ne se voit pas ; pour `7 × 8 = =` en revanche, on attend `56` puis `448` (`56 × 8`).

### 56.11.2 — Ce que `=` fait exactement

```text
appui sur =
  │
  ├── erreur en cours ? ──── oui ──> ne rien faire
  │
  ├── une opération est en attente ?
  │      oui ──> mémoriser (opération, opérande droit), puis calculer
  │
  └── non, mais une opération a déjà été mémorisée ?
         oui ──> rejouer : affichage OP dernierOperande
         non ──> ne rien faire (appuyer sur = après "5" laisse 5)
```


### 56.11.3 — Le code ajouté

Ajoutez les deux champs de mémoire et la méthode `egal()`. Le fichier complet est donné en 56.14.3.

```dart
  /// Mémoire de répétition : la dernière opération validée par =,
  /// et son opérande de droite.
  Operation? _derniereOperation;
  double? _dernierOperande;

  /// Appui sur =.
  void egal() {
    if (enErreur) {
      return;
    }

    if (_operationEnAttente != null) {
      // Cas normal : on solde le calcul en attente, et on mémorise
      // de quoi le rejouer.
      _derniereOperation = _operationEnAttente;
      _dernierOperande = valeurCourante;
      _calculerEnAttente();
      _nouvelleSaisie = true;
      return;
    }

    final Operation? op = _derniereOperation;
    final double? droite = _dernierOperande;
    if (op == null || droite == null) {
      _nouvelleSaisie = true;
      return; // rien à rejouer : = ne fait rien
    }

    // Répétition : affichage OP dernierOperande, dans cet ordre.
    final double gauche = valeurCourante;
    final double resultat;
    try {
      resultat = _appliquer(gauche, droite, op);
    } on DivisionParZeroException catch (e) {
      _messageErreur = e.message;
      _nouvelleSaisie = true;
      return;
    }
    _accumulateur = resultat;
    _saisie = Formatage.versTexteBrut(resultat);
    _nouvelleSaisie = true;
  }
```

Il faut aussi **oublier la répétition** dès que l'utilisateur choisit une nouvelle opération. Ajoutez ces deux lignes à la fin de `operateur()`, et les mêmes dans `effacerTout()` :

```dart
    _derniereOperation = null;
    _dernierOperande = null;
```

Sans elles, la séquence `2 + 3 = × 4 = =` répéterait l'addition au lieu de la multiplication.


### 56.11.4 — Vérifier

Ajoutez `case '=': calc.egal();` au `switch` de `jouer()` dans `bin/essai.dart`, puis contrôlez ces séquences.

| Séquence de touches | Écran final | Pourquoi |
| --- | --- | --- |
| `2 + 3 =` | `5` | calcul normal |
| `2 + 3 = =` | `8` | répétition : `5 + 3` |
| `2 + 3 = = =` | `11` | répétition : `8 + 3` |
| `7 × 8 = =` | `448` | répétition : `56 × 8` |
| `2 + 3 × 4 =` | `20` | calcul à plat : `(2 + 3) × 4` |
| `1 ÷ 3 =` | `0,333333333333` | coupé à 12 chiffres significatifs |
| `=` seul | `0` | rien à rejouer |
| `2 + 3 = × 4 = =` | `80` | le `×` a annulé la répétition du `+` |

La ligne `2 + 3 × 4 = -> 20` mérite un mot : la calculatrice a effectué `2 + 3 = 5` au moment de l'appui sur `×`, puis `5 × 4 = 20`. C'est le comportement d'une calculatrice de poche, pas celui d'un tableur (rappel de 56.2.3).

**État exécutable :** `dart run bin/essai.dart` reproduit exactement ce tableau.

---


## 56.12 — Étape 7 : `C` et `CE`, deux touches qui ne font pas la même chose

C'est une confusion classique, y compris chez les utilisateurs.

| Touche | Nom complet | Effet |
| --- | --- | --- |
| `C` | *Clear* | Tout est effacé : écran, accumulateur, opération en attente, mémoire de répétition, erreur |
| `CE` | *Clear Entry* | Seul l'écran repart à `0`. Le calcul en attente est **conservé** |

Le scénario qui justifie `CE` : l'utilisateur tape `12 + 9`, puis se rend compte qu'il voulait `5`. Avec `CE`, la séquence `12 + 9 CE 5 =` donne `17`, car le « 12 + » est préservé. Avec `C`, elle donne `5` : tout est perdu.

```dart
  /// Touche C : tout effacer.
  void effacerTout() {
    _saisie = '0';
    _nouvelleSaisie = true;
    _accumulateur = null;
    _operationEnAttente = null;
    _derniereOperation = null;
    _dernierOperande = null;
    _messageErreur = null;
  }

  /// Touche CE : effacer seulement l'entrée courante.
  void effacerEntree() {
    if (enErreur) {
      // Après une erreur, plus rien de cohérent n'est à conserver :
      // CE se comporte comme C.
      effacerTout();
      return;
    }
    _saisie = '0';
    _nouvelleSaisie = true;
  }
```

Notez le détail : `effacerEntree()` remet `_nouvelleSaisie` à `true`. Sinon, taper `5` après `CE` donnerait `05`.

**Vérifier :** ajoutez `case 'CE': calc.effacerEntree();` au `switch` de `jouer()`.

| Séquence de touches | Écran final |
| --- | --- |
| `1 2 + 9 CE 5 =` | `17` |
| `1 2 + 9 C 5 =` | `5` |
| `8 ÷ 0 =` | `Division par zéro` |
| `8 ÷ 0 = C 3 + 3 =` | `6` |

Sur la troisième ligne, la calculatrice est bloquée : tant que `C` n'est pas appuyé, les opérateurs et `=` sont ignorés. Seuls un chiffre, une virgule, `C` ou `CE` la débloquent.

**État exécutable :** `dart run bin/essai.dart` reproduit exactement ce tableau.

---


## 56.13 — Étape 8 : le signe et le pourcentage

### 56.13.1 — La touche `±`

Elle inverse le signe de la saisie **en cours**, pas du résultat d'un calcul futur. C'est une opération sur le texte :

```dart
  /// Touche ± : change le signe de la saisie en cours.
  void inverserSigne() {
    if (enErreur) {
      return;
    }
    if (_saisie.startsWith('-')) {
      _saisie = _saisie.substring(1);
      return;
    }
    if (_saisie == '0') {
      return; // "-0" n'a pas de sens à l'écran
    }
    _saisie = '-$_saisie';
  }
```

Pourquoi travailler sur le texte plutôt que sur `valeurCourante` ? Parce que l'utilisateur peut appuyer sur `±` au milieu d'une saisie : `1`, `2`, `±`, `5` doit donner `-125`. Si l'on repassait par un `double`, on perdrait la trace d'une éventuelle virgule en cours (`12.` deviendrait `12`).

### 56.13.2 — La touche `%`

Le pourcentage est la touche la plus mal comprise de toutes les calculatrices, parce qu'elle ne fait pas la même chose selon l'opération en attente.

| Contexte | Appui sur `%` | Justification |
| --- | --- | --- |
| `200 + 10 %` | `10` devient `20` | 10 % **de 200** |
| `200 - 10 %` | `10` devient `20` | idem, pour retirer 10 % |
| `6 × 5 %` | `5` devient `0,05` | ici `%` veut dire « divisé par cent » |
| `50 %` (rien en attente) | `50` devient `0,5` | idem |

Autrement dit : avec `+` ou `-` en attente, `%` renvoie `accumulateur × saisie / 100` ; sinon, `saisie / 100`.

```dart
  /// Touche % : transforme la saisie en cours en pourcentage.
  void pourcentage() {
    if (enErreur) {
      return;
    }
    final double valeur = valeurCourante;
    final Operation? op = _operationEnAttente;
    final double? gauche = _accumulateur;

    final double resultat;
    if (gauche != null &&
        (op == Operation.addition || op == Operation.soustraction)) {
      resultat = gauche * valeur / 100; // « 10 % de l'accumulateur »
    } else {
      resultat = valeur / 100; // « divisé par cent »
    }
    _saisie = Formatage.versTexteBrut(resultat);
    // On NE remet PAS _nouvelleSaisie à true : le résultat du pourcentage
    // est bien l'opérande de droite du calcul en attente.
    _nouvelleSaisie = false;
  }
```

> Ce `_nouvelleSaisie = false` est indispensable. S'il valait `true`, l'appui suivant sur `=` ou sur un opérateur croirait qu'aucun opérande n'a été saisi et jetterait le résultat du pourcentage. Contrepartie assumée : si l'utilisateur tape un chiffre juste après `%`, ce chiffre s'ajoute au résultat au lieu de le remplacer. C'est le défi 3 de la section 56.26.

**Vérifier :** ajoutez `case '±': calc.inverserSigne();` et `case '%': calc.pourcentage();` au `switch` de `jouer()`. Les valeurs ci-dessous sont exactes.

| Séquence de touches | Écran final |
| --- | --- |
| `9 ± + 4 =` | `-5` |
| `1 2 ± 5` | `-125` |
| `2 0 0 + 1 0 %` | `20` |
| `2 0 0 + 1 0 % =` | `220` |
| `2 0 0 + 1 0 % + 5 =` | `225` |
| `6 × 5 %` | `0,05` |
| `6 × 5 % =` | `0,3` |
| `5 0 %` | `0,5` |

**État exécutable :** `dart run bin/essai.dart` reproduit exactement ce tableau.

---


## 56.14 — Étape 9 : l'historique et le fichier de logique complet

### 56.14.1 — Le modèle d'une ligne d'historique

Une ligne porte trois informations : le calcul affiché, le résultat affiché, et la valeur numérique du résultat (pour réutiliser la ligne d'un simple appui, bonus B3).

**Fichier : `lib/modeles/ligne_historique.dart`**

```dart
/// Une entrée de l'historique : « 200 + 20 » qui donne « 220 ».
///
/// Classe immuable (chapitre 09) : tous les champs sont `final`, le
/// constructeur est `const`. On ne modifie jamais une ligne d'historique,
/// on en crée une nouvelle.
class LigneHistorique {
  const LigneHistorique({
    required this.expression,
    required this.resultat,
    required this.valeur,
  });

  /// Le calcul, déjà formaté pour l'affichage : "200 + 20".
  final String expression;

  /// Le résultat, déjà formaté : "220".
  final String resultat;

  /// Le résultat sous forme numérique, pour le réinjecter dans un calcul.
  final double valeur;

  @override
  String toString() => '$expression = $resultat';
}
```


### 56.14.2 — Enregistrer un calcul

Chaque calcul effectué produit une ligne, insérée **en tête** de liste (le plus récent en premier) par une méthode privée `_enregistrer(gauche, op, droite, resultat)`, et la liste est plafonnée à `maxHistorique` entrées :

```dart
    while (_historique.length > maxHistorique) {
      _historique.removeLast(); // on jette la plus ancienne
    }
```

Les trois textes de la ligne sont produits par un raccourci statique, `_formater(double v)`, qui enchaîne `Formatage.versTexteBrut` puis `Formatage.versAffichage`. Le code complet figure juste en dessous.


### 56.14.3 — Le fichier complet

Voici la version définitive. Elle rassemble les étapes 4 à 9, plus le retour arrière (bonus B1) et l'historique.

**Fichier : `lib/logique/calculatrice.dart`** (version finale)

```dart
import 'package:calculatrice/logique/erreurs.dart';
import 'package:calculatrice/logique/formatage.dart';
import 'package:calculatrice/logique/operation.dart';
import 'package:calculatrice/modeles/ligne_historique.dart';

/// Le moteur d'une calculatrice à quatre opérations.
///
/// Cette classe est du DART PUR : ni widgets, ni BuildContext, ni setState.
/// Elle expose des méthodes d'action (chiffre, operateur, egal...) et des
/// getters de lecture (affichage, expressionEnCours, historique...).
/// L'interface appelle une action, puis relit les getters. C'est tout.
class Calculatrice {
  /// Nombre maximal de chiffres saisissables.
  static const int maxChiffres = 12;

  /// Nombre maximal de lignes conservées dans l'historique.
  static const int maxHistorique = 50;

  /// La saisie en cours, au format technique (point décimal, pas d'espaces).
  String _saisie = '0';

  /// true si le prochain chiffre doit REMPLACER la saisie.
  bool _nouvelleSaisie = true;

  /// L'opérande de gauche mis de côté.
  double? _accumulateur;

  /// L'opération choisie mais pas encore effectuée.
  Operation? _operationEnAttente;

  /// Mémoire de répétition : dernière opération validée par =, et son
  /// opérande de droite.
  Operation? _derniereOperation;
  double? _dernierOperande;

  /// Message d'erreur courant, ou null.
  String? _messageErreur;

  /// Historique, le plus récent en premier.
  final List<LigneHistorique> _historique = <LigneHistorique>[];

  /// true si la calculatrice est bloquée sur une erreur.
  bool get enErreur => _messageErreur != null;

  /// Le texte de l'écran principal, prêt à être affiché.
  String get affichage => _messageErreur ?? Formatage.versAffichage(_saisie);

  /// La petite ligne du dessus : "12,5 +", ou "" s'il n'y a rien en attente.
  String get expressionEnCours {
    final double? gauche = _accumulateur;
    final Operation? op = _operationEnAttente;
    if (enErreur || gauche == null || op == null) {
      return '';
    }
    return '${_formater(gauche)} ${op.symbole}';
  }

  /// L'historique, en lecture seule : l'interface ne doit pas le modifier.
  List<LigneHistorique> get historique =>
      List<LigneHistorique>.unmodifiable(_historique);

  /// La valeur numérique de la saisie en cours.
  /// "12." et "-" ne sont pas des nombres valides : on les nettoie.
  double get valeurCourante {
    String texte = _saisie;
    if (texte.endsWith('.')) {
      texte = texte.substring(0, texte.length - 1);
    }
    if (texte.isEmpty || texte == '-') {
      return 0;
    }
    return double.tryParse(texte) ?? 0;
  }

  /// Le nombre de chiffres saisis (hors signe et hors point).
  int get _nombreDeChiffres => _saisie.replaceAll(RegExp(r'[^0-9]'), '').length;

  /// Appui sur une touche chiffre, de "0" à "9".
  void chiffre(String c) {
    assert(c.length == 1 && '0123456789'.contains(c),
        'chiffre() attend un caractère de "0" à "9", reçu : "$c"');

    if (enErreur) {
      effacerTout();
    }
    if (_nouvelleSaisie) {
      _saisie = c;
      _nouvelleSaisie = false;
      return;
    }
    if (_saisie == '0') {
      _saisie = c; // "0" + "5" donne "5", surtout pas "05"
      return;
    }
    if (_saisie == '-0') {
      _saisie = '-$c';
      return;
    }
    if (_nombreDeChiffres >= maxChiffres) {
      return; // saisie saturée
    }
    _saisie += c;
  }

  /// Appui sur la touche virgule.
  void separateurDecimal() {
    if (enErreur) {
      effacerTout();
    }
    if (_nouvelleSaisie) {
      _saisie = '0.'; // une calculatrice affiche "0," et non "," seule
      _nouvelleSaisie = false;
      return;
    }
    if (_saisie.contains('.')) {
      return; // un seul séparateur
    }
    _saisie += '.';
  }

  /// Retour arrière : efface le dernier caractère saisi (bonus B1).
  void retourArriere() {
    if (enErreur) {
      effacerTout();
      return;
    }
    if (_nouvelleSaisie) {
      return; // rien n'est en cours de saisie
    }
    if (_saisie.length <= 1 ||
        (_saisie.length == 2 && _saisie.startsWith('-'))) {
      _saisie = '0';
      _nouvelleSaisie = true;
      return;
    }
    _saisie = _saisie.substring(0, _saisie.length - 1);
    if (_saisie == '-') {
      _saisie = '0';
      _nouvelleSaisie = true;
    }
  }

  /// Appui sur +, -, × ou ÷.
  void operateur(Operation op) {
    if (enErreur) {
      return;
    }
    if (_operationEnAttente != null && !_nouvelleSaisie) {
      _calculerEnAttente();
      if (enErreur) {
        return;
      }
    } else {
      _accumulateur = valeurCourante;
    }
    _operationEnAttente = op;
    _nouvelleSaisie = true;
    // Une nouvelle opération annule la mémoire de répétition.
    _derniereOperation = null;
    _dernierOperande = null;
  }

  /// Appui sur =.
  void egal() {
    if (enErreur) {
      return;
    }

    if (_operationEnAttente != null) {
      _derniereOperation = _operationEnAttente;
      _dernierOperande = valeurCourante;
      _calculerEnAttente();
      _nouvelleSaisie = true;
      return;
    }

    final Operation? op = _derniereOperation;
    final double? droite = _dernierOperande;
    if (op == null || droite == null) {
      _nouvelleSaisie = true;
      return; // rien à rejouer
    }

    // Répétition : affichage OP dernierOperande, dans cet ordre.
    final double gauche = valeurCourante;
    final double resultat;
    try {
      resultat = _appliquer(gauche, droite, op);
    } on DivisionParZeroException catch (e) {
      _messageErreur = e.message;
      _nouvelleSaisie = true;
      return;
    }
    _enregistrer(gauche, op, droite, resultat);
    _accumulateur = resultat;
    _saisie = Formatage.versTexteBrut(resultat);
    _nouvelleSaisie = true;
  }

  /// Touche ± : change le signe de la saisie en cours.
  void inverserSigne() {
    if (enErreur) {
      return;
    }
    if (_saisie.startsWith('-')) {
      _saisie = _saisie.substring(1);
      return;
    }
    if (_saisie == '0') {
      return; // "-0" n'a pas de sens à l'écran
    }
    _saisie = '-$_saisie';
  }

  /// Touche % : transforme la saisie en cours en pourcentage.
  void pourcentage() {
    if (enErreur) {
      return;
    }
    final double valeur = valeurCourante;
    final Operation? op = _operationEnAttente;
    final double? gauche = _accumulateur;

    final double resultat;
    if (gauche != null &&
        (op == Operation.addition || op == Operation.soustraction)) {
      resultat = gauche * valeur / 100; // « 10 % de l'accumulateur »
    } else {
      resultat = valeur / 100; // « divisé par cent »
    }
    _saisie = Formatage.versTexteBrut(resultat);
    _nouvelleSaisie = false;
  }

  /// Touche C : tout effacer, sauf l'historique.
  void effacerTout() {
    _saisie = '0';
    _nouvelleSaisie = true;
    _accumulateur = null;
    _operationEnAttente = null;
    _derniereOperation = null;
    _dernierOperande = null;
    _messageErreur = null;
  }

  /// Touche CE : effacer seulement l'entrée courante.
  void effacerEntree() {
    if (enErreur) {
      effacerTout(); // plus rien de cohérent à conserver
      return;
    }
    _saisie = '0';
    _nouvelleSaisie = true;
  }

  /// Vide l'historique (bouton « vider » du panneau).
  void viderHistorique() => _historique.clear();

  /// Recharge une valeur venue de l'historique dans l'écran (bonus B3).
  void reprendre(double valeur) {
    if (enErreur) {
      effacerTout();
    }
    _saisie = Formatage.versTexteBrut(valeur);
    _nouvelleSaisie = false;
  }

  /// Effectue l'opération en attente et place le résultat à l'écran.
  void _calculerEnAttente() {
    final double gauche = _accumulateur!;
    final double droite = valeurCourante;
    final Operation op = _operationEnAttente!;

    final double resultat;
    try {
      resultat = _appliquer(gauche, droite, op);
    } on DivisionParZeroException catch (e) {
      _messageErreur = e.message;
      _accumulateur = null;
      _operationEnAttente = null;
      _nouvelleSaisie = true;
      return;
    }
    _enregistrer(gauche, op, droite, resultat);
    _accumulateur = resultat;
    _saisie = Formatage.versTexteBrut(resultat);
    _operationEnAttente = null;
    _nouvelleSaisie = true;
  }

  /// Le calcul pur. Seule méthode arithmétique du projet.
  double _appliquer(double a, double b, Operation op) {
    switch (op) {
      case Operation.addition:
        return a + b;
      case Operation.soustraction:
        return a - b;
      case Operation.multiplication:
        return a * b;
      case Operation.division:
        if (b == 0) {
          throw const DivisionParZeroException();
        }
        return a / b;
    }
  }

  /// Ajoute une ligne en tête de l'historique et plafonne la liste.
  void _enregistrer(
      double gauche, Operation op, double droite, double resultat) {
    _historique.insert(
      0,
      LigneHistorique(
        expression: '${_formater(gauche)} ${op.symbole} ${_formater(droite)}',
        resultat: _formater(resultat),
        valeur: resultat,
      ),
    );
    while (_historique.length > maxHistorique) {
      _historique.removeLast(); // on jette la plus ancienne
    }
  }

  /// Raccourci : double -> texte affichable.
  static String _formater(double v) =>
      Formatage.versAffichage(Formatage.versTexteBrut(v));
}
```


### 56.14.4 — Vérifier l'historique

Dans `bin/essai.dart`, ajoutez `case '%': calc.pourcentage();` au `switch` de `jouer()`, réutilisez la même instance de `Calculatrice` pour les trois séquences suivantes, puis affichez `calc.historique` :

```dart
  jouer(<String>['2', '0', '0', '+', '1', '0', '%', '=']);
  jouer(<String>['4', '5', '÷', '1', '0', '0', '×', '2', '0', '=']);
  jouer('1234567+7654321='.split(''));

  print('--- historique (le plus récent en premier) ---');
  for (final LigneHistorique ligne in calc.historique) {
    print(ligne);
  }
```

**Résultat :**

```text
--- historique (le plus récent en premier) ---
1 234 567 + 7 654 321 = 8 888 888
0,45 × 20 = 9
45 ÷ 100 = 0,45
200 + 20 = 220
```

Remarquez la dernière ligne : l'historique enregistre l'opération **réellement effectuée**, avec le `10 %` déjà converti en `20`. C'est plus honnête que d'écrire `200 + 10 %`.

**État exécutable :** `dart run bin/essai.dart` produit la sortie ci-dessus. La logique est terminée, et pas une seule ligne de Flutter n'a été écrite.

---


## 56.15 — Étape 10 : les tests unitaires

C'est le moment de récolter les fruits de la séparation. Comme `Calculatrice` n'importe pas Flutter, ses tests s'exécutent en une fraction de seconde, sans émulateur, sans navigateur.

Rappel du chapitre 16 : un test est un fichier `*_test.dart` placé dans `test/`, contenant des `test('description', () { ... })` groupés par `group(...)`.

**Fichier : `test/calculatrice_test.dart`**

```dart
import 'package:calculatrice/logique/calculatrice.dart';
import 'package:calculatrice/logique/operation.dart';
import 'package:flutter_test/flutter_test.dart';

/// Joue une suite de touches sur une calculatrice neuve et la renvoie.
/// Les touches s'écrivent comme sur l'appareil, ce qui rend chaque test
/// lisible d'un coup d'oeil : jouer(['2', '+', '3', '=']).
Calculatrice jouer(List<String> touches) {
  final Calculatrice calc = Calculatrice();
  for (final String t in touches) {
    switch (t) {
      case '+':
        calc.operateur(Operation.addition);
      case '-':
        calc.operateur(Operation.soustraction);
      case '×':
        calc.operateur(Operation.multiplication);
      case '÷':
        calc.operateur(Operation.division);
      case '=':
        calc.egal();
      case ',':
        calc.separateurDecimal();
      case 'C':
        calc.effacerTout();
      case 'CE':
        calc.effacerEntree();
      case '±':
        calc.inverserSigne();
      case '%':
        calc.pourcentage();
      case '<':
        calc.retourArriere();
      default:
        calc.chiffre(t);
    }
  }
  return calc;
}

/// Raccourci : joue les touches et renvoie l'écran.
String ecran(List<String> touches) => jouer(touches).affichage;

void main() {
  group('Saisie', () {
    test('écran à 0, chiffres collés, zéro initial remplacé', () {
      expect(Calculatrice().affichage, '0');
      expect(ecran(<String>['1', '2', '3']), '123');
      expect(ecran(<String>['0', '5']), '5');
    });

    test('la saisie est limitée à 12 chiffres', () {
      expect(ecran('12345678901234'.split('')), '123 456 789 012');
    });

    test('séparateur décimal : visible aussitôt, et unique', () {
      expect(ecran(<String>['1', '2', ',']), '12,');
      expect(ecran(<String>['1', '2', ',', '5', ',', '7']), '12,57');
      expect(ecran(<String>[',', '5']), '0,5');
    });

    test('le retour arrière efface le dernier caractère', () {
      expect(ecran(<String>['1', '2', '3', '<']), '12');
      expect(ecran(<String>['7', '<']), '0');
    });
  });

  group('Les quatre opérations', () {
    test('addition, soustraction, multiplication, division', () {
      expect(ecran(<String>['7', '+', '8', '=']), '15');
      expect(ecran(<String>['1', '0', '-', '3', '=']), '7');
      expect(ecran(<String>['7', '×', '8', '=']), '56');
      expect(ecran(<String>['1', '0', '0', '÷', '8', '=']), '12,5');
    });
  });

  group('Division par zéro', () {
    test('affiche un message et bloque les opérateurs', () {
      final Calculatrice calc = jouer(<String>['5', '÷', '0', '=']);
      expect(calc.enErreur, isTrue);
      expect(calc.affichage, 'Division par zéro');
      calc.operateur(Operation.addition); // ignoré
      expect(calc.affichage, 'Division par zéro');
    });

    test('un chiffre ou C débloquent la calculatrice', () {
      expect(ecran(<String>['5', '÷', '0', '=', '+', '2']), '2');
      expect(ecran(<String>['8', '÷', '0', '=', 'C', '3', '+', '3', '=']), '6');
    });
  });

  group('Enchaînement', () {
    test('un opérateur solde le calcul, le calcul est à plat', () {
      expect(ecran(<String>['2', '+', '3', '+']), '5');
      expect(ecran(<String>['2', '+', '3', '×', '4', '=']), '20');
      // Deux opérateurs de suite : le second remplace le premier.
      expect(ecran(<String>['1', '2', '+', '×', '3', '=']), '36');
    });
  });

  group('Touche =', () {
    test('répète la dernière opération, dans le bon sens', () {
      expect(ecran(<String>['2', '+', '3', '=', '=', '=']), '11');
      expect(ecran(<String>['7', '×', '8', '=', '=']), '448');
    });

    test('rien à rejouer, et annulation par un nouvel opérateur', () {
      expect(ecran(<String>['=']), '0');
      // 2+3=5, puis ×4=20, puis = doit refaire ×4 -> 80, pas +3.
      expect(ecran(<String>['2', '+', '3', '=', '×', '4', '=', '=']), '80');
    });
  });

  group('C et CE', () {
    test('CE conserve le calcul en attente, C le perd', () {
      expect(ecran(<String>['1', '2', '+', '9', 'CE', '5', '=']), '17');
      expect(ecran(<String>['1', '2', '+', '9', 'C', '5', '=']), '5');
    });
  });

  group('Signe et pourcentage', () {
    test('± inverse le signe sans jamais produire -0', () {
      expect(ecran(<String>['9', '±']), '-9');
      expect(ecran(<String>['9', '±', '±']), '9');
      expect(ecran(<String>['0', '±']), '0');
      expect(ecran(<String>['1', '2', '±', '5']), '-125');
    });

    test('% additif : 10 % de 200 vaut 20', () {
      final Calculatrice calc =
          jouer(<String>['2', '0', '0', '+', '1', '0', '%']);
      expect(calc.affichage, '20');
      calc.egal();
      expect(calc.affichage, '220');
    });

    test('% multiplicatif ou seul : divise par cent', () {
      expect(ecran(<String>['6', '×', '5', '%']), '0,05');
      expect(ecran(<String>['6', '×', '5', '%', '=']), '0,3');
      expect(ecran(<String>['5', '0', '%']), '0,5');
    });
  });

  group('Formatage', () {
    test('les milliers sont groupés', () {
      expect(ecran('1234567+7654321='.split('')), '8 888 888');
    });

    test('poussières binaires gommées, décimales coupées à 12 chiffres', () {
      expect(ecran(<String>[',', '1', '+', ',', '2', '=']), '0,3');
      expect(ecran(<String>['1', '-', ',', '9', '=']), '0,1');
      expect(ecran(<String>['1', '÷', '3', '=']), '0,333333333333');
      expect(ecran(<String>['2', '÷', '3', '=']), '0,666666666667');
    });
  });

  group('Historique et expression', () {
    test('l\'expression montre le calcul en attente', () {
      expect(jouer(<String>['1', '2', ',', '5', '+']).expressionEnCours,
          '12,5 +');
      expect(jouer(<String>['1', '2']).expressionEnCours, '');
    });

    test('chaque calcul ajoute une ligne, la plus récente en tête', () {
      final Calculatrice calc =
          jouer(<String>['2', '+', '3', '=', '1', '0', '×', '2', '=']);
      expect(calc.historique.length, 2);
      expect(calc.historique.first.expression, '10 × 2');
      expect(calc.historique.first.resultat, '20');
      expect(calc.historique.last.expression, '2 + 3');
    });

    test('une erreur n\'entre pas dans l\'historique, qui est en lecture seule',
        () {
      expect(jouer(<String>['5', '÷', '0', '=']).historique, isEmpty);
      final Calculatrice calc = jouer(<String>['2', '+', '3', '=']);
      expect(() => calc.historique.clear(), throwsUnsupportedError);
    });
  });
}
```

> Le test des milliers utilise `'1234567+7654321='.split('')` : chaque caractère de la chaîne devient une touche, `+` et `=` compris.

Lancez :

```bash
flutter test
```

**Résultat :**

```text
00:02 +19: All tests passed!
```

Dix-neuf tests couvrant les vingt exigences du cahier des charges, en deux secondes, sans lancer la moindre application. Voilà pourquoi on isole la logique.

> Si ces règles de calcul vivaient dans le `State` d'un `StatefulWidget`, il faudrait écrire des *widget tests* : construire un `MaterialApp`, chercher les boutons avec `find.text('7')`, appeler `tester.tap(...)` puis `tester.pump()`. C'est bien plus long à écrire, plus lent à exécuter, et cela casse au moindre changement de maquette.

**État exécutable :** `flutter test` affiche `All tests passed!`.

---


## 56.16 — Étape 11 : `main.dart`, le thème et une page vide

La logique est finie et testée. On passe à Flutter.

### 56.16.1 — Une seule couleur source pour tout le projet

Rappel du chapitre 51 : en Material 3, on ne choisit pas vingt couleurs à la main. On choisit **une** couleur source, et `ColorScheme.fromSeed` en dérive un jeu complet et cohérent, en clair comme en sombre. C'est ce qui nous permettra, en 56.18, de colorer quatre familles de boutons sans écrire un seul code hexadécimal.

**Fichier : `lib/main.dart`**

```dart
import 'package:flutter/material.dart';

import 'package:calculatrice/ui/page_calculatrice.dart';

void main() {
  runApp(const ApplicationCalculatrice());
}

class ApplicationCalculatrice extends StatelessWidget {
  const ApplicationCalculatrice({super.key});

  /// L'unique couleur choisie « à la main » du projet.
  /// Tout le reste en est dérivé par Material 3.
  static const Color couleurSource = Color(0xFF00695C);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Calculatrice',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(seedColor: couleurSource),
      ),
      // Même graine, luminosité inversée (bonus B2).
      darkTheme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(
          seedColor: couleurSource,
          brightness: Brightness.dark,
        ),
      ),
      // Flutter choisit l'un ou l'autre selon le réglage du système.
      themeMode: ThemeMode.system,
      home: const PageCalculatrice(),
    );
  }
}
```

**Fichier : `lib/ui/page_calculatrice.dart`** (première version)

```dart
import 'package:flutter/material.dart';

/// La page unique de l'application. Elle est `Stateful` parce qu'elle
/// possède une Calculatrice dont l'état change à chaque appui (ch. 45).
class PageCalculatrice extends StatefulWidget {
  const PageCalculatrice({super.key});

  @override
  State<PageCalculatrice> createState() => _EtatPageCalculatrice();
}

class _EtatPageCalculatrice extends State<PageCalculatrice> {
  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Calculatrice')),
        body: const Center(child: Text('Interface à construire')),
      );
}
```

Le compteur d'exemple disparaît : `lib/main.dart` est intégralement remplacé, et le fichier `test/widget_test.dart` généré par `flutter create` doit être supprimé, car il teste ce compteur et ne compile plus.

```bash
rm test/widget_test.dart
```

**État exécutable :** `flutter run` affiche une page vide avec le titre « Calculatrice », dans les couleurs du thème et en mode sombre si votre système l'est.

---


## 56.17 — Étape 12 : l'écran d'affichage et le débordement

### 56.17.1 — Le problème

L'écran doit afficher jusqu'à `123 456 789 012` en très gros. Sur un téléphone étroit, ce texte ne rentre pas. Trois mauvaises solutions, une bonne.

| Solution | Effet | Verdict |
| --- | --- | --- |
| Ne rien faire | Bande jaune et noire « RenderFlex overflowed » (ch. 46) | Non |
| `overflow: TextOverflow.ellipsis` | `123 456...` : le résultat devient faux à l'œil | Non |
| Baisser la taille de police en dur | Illisible pour les petits nombres | Non |
| `FittedBox(fit: BoxFit.scaleDown)` | Le texte se réduit **juste ce qu'il faut** | Oui |

### 56.17.2 — Comment `FittedBox` fonctionne

Rappel du modèle du chapitre 46 : les contraintes descendent, les tailles remontent.

```text
  Le parent donne au FittedBox : largeur max = 360, hauteur = 76 (imposée)
                     │
  FittedBox mesure son enfant SANS contrainte de largeur.
  Le Text répond : « je fais 520 x 70 »
                     │
  Facteur d'échelle : 360/520 = 0,69 et 76/70 = 1,08
  scaleDown -> min(1 ; 0,69 ; 1,08) = 0,69
                     │
  Il dessine le Text à 69 % de sa taille. Rien ne déborde.
```

`BoxFit.scaleDown` se comporte comme `BoxFit.contain`, avec une différence essentielle : **il ne grossit jamais**. Un `0` seul reste à 64 points.

Deux précautions dans le code :

1. **Fixer la hauteur** avec un `SizedBox`. Sans cela, la hauteur de l'écran changerait à chaque chiffre tapé, et toute la page sauterait.
2. **Empêcher le retour à la ligne** avec `maxLines: 1` et `softWrap: false`. Un `FittedBox` mesure son enfant sans contrainte de largeur, donc le `Text` ne se couperait pas de lui-même — mais ces réglages rendent l'intention explicite.


### 56.17.3 — Le code

**Fichier : `lib/ui/ecran_calculatrice.dart`**

```dart
import 'package:flutter/material.dart';

/// La zone d'affichage : la ligne d'expression et l'écran principal.
///
/// Widget PUREMENT décoratif : il reçoit deux textes déjà formatés et ne
/// calcule rien. Il n'a donc besoin d'aucun état -> StatelessWidget.
class EcranCalculatrice extends StatelessWidget {
  const EcranCalculatrice({
    super.key,
    required this.affichage,
    required this.expression,
    this.enErreur = false,
  });

  /// Le grand texte : "12,5" ou "Division par zéro".
  final String affichage;

  /// La petite ligne du dessus : "200 +", ou "" s'il n'y a rien en attente.
  final String expression;

  /// true pour afficher le grand texte en rouge d'erreur.
  final bool enErreur;

  @override
  Widget build(BuildContext context) {
    final ColorScheme schema = Theme.of(context).colorScheme;

    return Container(
      width: double.infinity,
      padding: const EdgeInsets.fromLTRB(24, 12, 24, 12),
      child: Column(
        // Le contenu est collé en bas à droite, comme sur une vraie
        // calculatrice.
        mainAxisAlignment: MainAxisAlignment.end,
        crossAxisAlignment: CrossAxisAlignment.end,
        children: <Widget>[
          // ---------- ligne d'expression ----------
          SizedBox(
            width: double.infinity,
            height: 28,
            child: FittedBox(
              fit: BoxFit.scaleDown,
              alignment: Alignment.centerRight,
              child: Text(
                expression,
                maxLines: 1,
                softWrap: false,
                style: TextStyle(fontSize: 22, color: schema.onSurfaceVariant),
              ),
            ),
          ),
          const SizedBox(height: 8),
          // ---------- écran principal ----------
          SizedBox(
            width: double.infinity,
            height: 76,
            child: FittedBox(
              fit: BoxFit.scaleDown,
              alignment: Alignment.centerRight,
              child: Text(
                affichage,
                maxLines: 1,
                softWrap: false,
                style: TextStyle(
                  fontSize: 64,
                  fontWeight: FontWeight.w300,
                  height: 1.1,
                  // La couleur d'erreur vient du thème, pas de Colors.red.
                  color: enErreur ? schema.error : schema.onSurface,
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```


### 56.17.4 — L'essayer

Dans `page_calculatrice.dart`, importez `ecran_calculatrice.dart`, ajoutez un champ `int _exemple = 0` et une constante `_exemples` contenant quatre couples `[affichage, expression]` : `['0', '']`, `['12,5', '200 +']`, `['123 456 789 012', '999 999 999 999 ×']` et `['Division par zéro', '']`. Placez ensuite un `EcranCalculatrice` en haut du corps et un `FilledButton` qui fait avancer `_exemple` dans un `setState`.

**État exécutable :** `flutter run`, puis appuyez plusieurs fois sur le bouton. Vérifiez que `0` s'affiche en très gros aligné à droite, que `123 456 789 012` se réduit pour tenir dans la largeur, que la hauteur de la zone d'affichage ne bouge **jamais**, et que `Division par zéro` s'affiche en rouge d'erreur. Rétrécissez la fenêtre (bureau ou web) : le texte se réduit encore.

---


## 56.18 — Étape 13 : le widget `BoutonCalc`

### 56.18.1 — Pourquoi un widget dédié

Notre clavier compte vingt boutons. Sans widget dédié, il faudrait écrire vingt fois la même dizaine de lignes : forme, couleur, ombre, effet d'appui, taille de police. Toute modification de style demanderait vingt corrections. Un widget réutilisable, c'est le principe de composition du chapitre 44 appliqué à la lettre.

### 56.18.2 — Les quatre familles de boutons

| Famille | Touches | Fond | Texte |
| --- | --- | --- | --- |
| `chiffre` | `0` à `9`, `,` | `surfaceContainerHighest` | `onSurface` |
| `fonction` | `C`, `CE`, `%`, `±` | `secondaryContainer` | `onSecondaryContainer` |
| `operateur` | `+`, `-`, `×`, `÷` | `primaryContainer` | `onPrimaryContainer` |
| `egal` | `=` | `primary` | `onPrimary` |

Quatre familles, donc un `enum` de quatre valeurs (chapitre 11). Et pour chaque famille, un couple de couleurs pris **dans le `ColorScheme`** : aucune couleur en dur. Conséquence directe, le mode sombre fonctionne sans une ligne de code supplémentaire, et les contrastes restent lisibles parce que Material 3 garantit que `onX` se lit sur `X`.

### 56.18.3 — L'effet d'appui : `Material` puis `InkWell`

Rappel du chapitre 49 : l'onde d'appui est dessinée **par le widget `Material`**, pas par `InkWell`. `InkWell` ne fait que détecter le geste et demander à son `Material` ancêtre de peindre l'onde. D'où deux erreurs classiques :

```text
ERREUR 1 : InkWell sans Material au-dessus
  Container(color: bleu, child: InkWell(...))
  -> le clic fonctionne, mais AUCUNE onde n'apparaît.

ERREUR 2 : la couleur mise sur le Container et non sur le Material
  Material(child: Container(color: bleu, child: InkWell(...)))
  -> l'onde est peinte par le Material, donc SOUS le Container opaque.
```

> La couleur de fond se met **sur le `Material`**, jamais sur un `Container` placé entre le `Material` et l'`InkWell`. Et pour que l'onde reste dans les coins arrondis, il faut donner au `Material` à la fois `borderRadius` et `clipBehavior: Clip.antiAlias`.


### 56.18.4 — Le code

**Fichier : `lib/ui/bouton_calc.dart`**

```dart
import 'package:flutter/material.dart';

/// Les quatre familles visuelles de boutons.
enum TypeBouton {
  /// 0 à 9 et la virgule.
  chiffre,

  /// C, CE, %, ±.
  fonction,

  /// +, -, × et ÷.
  operateur,

  /// La touche =, mise en avant.
  egal,
}

/// Un bouton de la calculatrice.
///
/// Ce widget renvoie un `Expanded` : il est donc conçu pour être placé
/// DIRECTEMENT dans un `Row` ou un `Column`. C'est légal parce que
/// `BoutonCalc` est un `StatelessWidget` : il ne crée aucun objet de
/// rendu entre le `Row` et l'`Expanded` (rappel du chapitre 46).
class BoutonCalc extends StatelessWidget {
  const BoutonCalc({
    super.key,
    required this.libelle,
    required this.onPressed,
    this.type = TypeBouton.chiffre,
    this.flex = 1,
  });

  /// Le texte écrit sur le bouton : "7", "÷", "CE"...
  final String libelle;

  /// Ce qu'il faut faire à l'appui.
  final VoidCallback onPressed;

  /// La famille visuelle du bouton.
  final TypeBouton type;

  /// La part de largeur prise dans le Row. 2 pour un bouton double.
  final int flex;

  /// La couleur de fond, prise dans le thème (chapitre 51).
  Color _couleurFond(ColorScheme schema) {
    switch (type) {
      case TypeBouton.chiffre:
        return schema.surfaceContainerHighest;
      case TypeBouton.fonction:
        return schema.secondaryContainer;
      case TypeBouton.operateur:
        return schema.primaryContainer;
      case TypeBouton.egal:
        return schema.primary;
    }
  }

  /// La couleur du texte. Material 3 garantit qu'un `onX` se lit sur `X`.
  Color _couleurTexte(ColorScheme schema) {
    switch (type) {
      case TypeBouton.chiffre:
        return schema.onSurface;
      case TypeBouton.fonction:
        return schema.onSecondaryContainer;
      case TypeBouton.operateur:
        return schema.onPrimaryContainer;
      case TypeBouton.egal:
        return schema.onPrimary;
    }
  }

  @override
  Widget build(BuildContext context) {
    final ColorScheme schema = Theme.of(context).colorScheme;
    final Color encre = _couleurTexte(schema);

    return Expanded(
      flex: flex,
      child: Padding(
        padding: const EdgeInsets.all(5),
        child: Material(
          // La couleur va ICI, sur le Material, pour que l'onde d'appui
          // soit peinte AU-DESSUS.
          color: _couleurFond(schema),
          borderRadius: BorderRadius.circular(20),
          // Sans ce clip, l'onde déborderait des coins arrondis.
          clipBehavior: Clip.antiAlias,
          child: InkWell(
            onTap: onPressed,
            // L'onde et le voile d'appui sont faits de la couleur du
            // texte, très transparente : ils restent visibles sur
            // n'importe quel fond, clair comme sombre.
            splashColor: encre.withValues(alpha: 0.18),
            highlightColor: encre.withValues(alpha: 0.08),
            child: Center(
              child: Text(
                libelle,
                style: TextStyle(
                  fontSize: 26,
                  fontWeight: FontWeight.w500,
                  color: encre,
                ),
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

> `withValues(alpha: 0.18)` remplace l'ancien `withOpacity(0.18)`, qui perdait de la précision de couleur. La version de référence de cette formation est Flutter 3.44.


### 56.18.5 — L'essayer

Placez temporairement quatre boutons, un par famille, dans le corps de la page :

```dart
      body: SafeArea(
        child: Row(
          children: <Widget>[
            BoutonCalc(libelle: '7', onPressed: () {}),
            BoutonCalc(
                libelle: 'CE', type: TypeBouton.fonction, onPressed: () {}),
            BoutonCalc(
                libelle: '÷', type: TypeBouton.operateur, onPressed: () {}),
            // flex: 2 -> ce bouton est deux fois plus large.
            BoutonCalc(
                libelle: '=',
                type: TypeBouton.egal,
                flex: 2,
                onPressed: () {}),
          ],
        ),
      ),
```

**État exécutable :** `flutter run`. Quatre boutons de quatre couleurs, le dernier deux fois plus large. Maintenez le doigt (ou le clic) appuyé : l'onde part du point de contact et reste **à l'intérieur** des coins arrondis. Basculez votre système en mode sombre : les couleurs s'adaptent sans que vous ayez touché au code.

---


## 56.19 — Étape 14 : la grille de boutons avec `Row` et `Expanded`

### 56.19.1 — Pourquoi pas `GridView`

Le chapitre 48 présente `GridView`, qui semble tout indiqué pour une grille. Ce serait pourtant un mauvais choix ici :

| Critère | `GridView` | `Column` + `Row` + `Expanded` |
| --- | --- | --- |
| Remplit la hauteur disponible | Non : il défile | Oui : `Expanded` partage la hauteur |
| Bouton de largeur double | Difficile | Trivial : `flex: 2` |
| Recycle les éléments | Oui, inutile pour 20 boutons | Non, sans importance |

`GridView` sert aux listes **longues et défilantes**. Un clavier a vingt touches et ne défile pas. On utilise donc le layout du chapitre 46.

### 56.19.2 — La structure

```text
Column                        <- 5 lignes qui se partagent la hauteur
├── Expanded                  <- ligne 1, 1/5 de la hauteur
│   └── Row
│       ├── BoutonCalc "C"    <- chacun renvoie un Expanded : 1/4 de largeur
│       ├── BoutonCalc "CE"
│       ├── BoutonCalc "%"
│       └── BoutonCalc "÷"
└── Expanded × 4              <- lignes 2 à 5, construites de même
```

Chaque `Expanded` de la `Column` prend un cinquième de la hauteur, chaque `BoutonCalc` un quart de la largeur. La grille s'adapte donc à n'importe quel écran, sans une seule dimension écrite en dur.


### 56.19.3 — Le code

Nous décrivons la disposition par une table de chaînes, puis nous la parcourons avec une boucle `for` dans la liste des enfants (collection-for du chapitre 06). Ajouter une touche demandera de modifier une seule ligne.

**Fichier : `lib/ui/clavier_calculatrice.dart`**

```dart
import 'package:flutter/material.dart';

import 'package:calculatrice/ui/bouton_calc.dart';

/// La grille 5 x 4 des touches.
///
/// Ce widget ne connaît PAS la calculatrice. Il se contente de dire
/// « on a appuyé sur tel jeton » via [onTouche] : c'est la remontée
/// d'état du chapitre 45.
class ClavierCalculatrice extends StatelessWidget {
  const ClavierCalculatrice({super.key, required this.onTouche});

  /// Appelé avec le jeton du bouton appuyé : "7", "÷", "CE"...
  final void Function(String jeton) onTouche;

  /// La disposition, lue de haut en bas et de gauche à droite.
  /// Modifier le clavier = modifier cette table, et rien d'autre.
  static const List<List<String>> disposition = <List<String>>[
    <String>['C', 'CE', '%', '÷'],
    <String>['7', '8', '9', '×'],
    <String>['4', '5', '6', '-'],
    <String>['1', '2', '3', '+'],
    <String>['±', '0', ',', '='],
  ];

  /// À quelle famille visuelle appartient un jeton.
  static TypeBouton typeDe(String jeton) {
    if (jeton == '=') {
      return TypeBouton.egal;
    }
    if (jeton == '+' || jeton == '-' || jeton == '×' || jeton == '÷') {
      return TypeBouton.operateur;
    }
    if (jeton == 'C' || jeton == 'CE' || jeton == '%' || jeton == '±') {
      return TypeBouton.fonction;
    }
    return TypeBouton.chiffre;
  }

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.fromLTRB(7, 0, 7, 7),
      child: Column(
        children: <Widget>[
          for (final List<String> ligne in disposition)
            Expanded(
              child: Row(
                children: <Widget>[
                  for (final String jeton in ligne)
                    BoutonCalc(
                      libelle: jeton,
                      type: typeDe(jeton),
                      onPressed: () => onTouche(jeton),
                    ),
                ],
              ),
            ),
        ],
      ),
    );
  }
}
```

> Notez `onPressed: () => onTouche(jeton)`. On ne peut pas écrire `onPressed: onTouche` : `onTouche` attend un argument, alors que `onPressed` est un `VoidCallback` sans argument. La lambda sert d'adaptateur et capture le `jeton` de la boucle.


### 56.19.4 — L'essayer

Ajoutez un champ `String _dernier = '';` au `State`, importez `clavier_calculatrice.dart` et remplacez le corps par un `Column` contenant un `Text('Dernier jeton : $_dernier')` puis `Expanded(child: ClavierCalculatrice(onTouche: (String jeton) => setState(() => _dernier = jeton)))`.

**État exécutable :** `flutter run`. Le clavier complet remplit toute la place disponible et chaque appui met à jour la ligne du haut. Tournez l'appareil en paysage : les boutons deviennent larges et plats, mais rien ne déborde.

---


## 56.20 — Étape 15 : brancher la logique sur l'interface

Nous avons d'un côté une `Calculatrice` testée, de l'autre un clavier et un écran. Il ne reste qu'à les relier. Ce sera étonnamment court.

### 56.20.1 — Le principe

```text
   appui utilisateur ──> onTouche('7') ──> setState(() {
                                             _calculatrice.chiffre('7');
                                           })
                                              │
                                              v
                                           build() est rappelée
                                              │
                                              v
                    EcranCalculatrice(affichage: _calculatrice.affichage, ...)
```

Rappel du chapitre 45 : `setState` ne fait pas la modification, il **prévient** Flutter qu'une modification a eu lieu et qu'il faut reconstruire. On met donc l'appel métier à l'intérieur, et on relit les getters dans `build()`.

Toute la traduction « jeton d'interface » vers « méthode métier » tient dans un seul `switch`. C'est le seul endroit du projet où les deux mondes se rencontrent.


### 56.20.2 — Le code

**Fichier : `lib/ui/page_calculatrice.dart`** (version complète, hors historique et clavier physique)

```dart
import 'package:flutter/material.dart';

import 'package:calculatrice/logique/calculatrice.dart';
import 'package:calculatrice/logique/operation.dart';
import 'package:calculatrice/ui/clavier_calculatrice.dart';
import 'package:calculatrice/ui/ecran_calculatrice.dart';

class PageCalculatrice extends StatefulWidget {
  const PageCalculatrice({super.key});

  @override
  State<PageCalculatrice> createState() => _EtatPageCalculatrice();
}

class _EtatPageCalculatrice extends State<PageCalculatrice> {
  /// L'objet métier. `final` : on ne le remplace jamais, on le modifie.
  final Calculatrice _calculatrice = Calculatrice();

  /// Traduit un jeton d'interface en appel de méthode métier.
  ///
  /// C'est le SEUL pont entre lib/ui et lib/logique.
  void _appuyer(String jeton) {
    setState(() {
      switch (jeton) {
        case 'C':
          _calculatrice.effacerTout();
        case 'CE':
          _calculatrice.effacerEntree();
        case '%':
          _calculatrice.pourcentage();
        case '±':
          _calculatrice.inverserSigne();
        case ',':
          _calculatrice.separateurDecimal();
        case '=':
          _calculatrice.egal();
        case '<':
          _calculatrice.retourArriere();
        case '+':
          _calculatrice.operateur(Operation.addition);
        case '-':
          _calculatrice.operateur(Operation.soustraction);
        case '×':
          _calculatrice.operateur(Operation.multiplication);
        case '÷':
          _calculatrice.operateur(Operation.division);
        default:
          _calculatrice.chiffre(jeton);
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Calculatrice')),
      body: SafeArea(
        child: Column(
          children: <Widget>[
            // L'écran occupe 2 parts de hauteur...
            Expanded(
              flex: 2,
              child: EcranCalculatrice(
                affichage: _calculatrice.affichage,
                expression: _calculatrice.expressionEnCours,
                enErreur: _calculatrice.enErreur,
              ),
            ),
            // ... et le clavier 5 parts.
            Expanded(
              flex: 5,
              child: ClavierCalculatrice(onTouche: _appuyer),
            ),
          ],
        ),
      ),
    );
  }
}
```

C'est tout. Quarante lignes utiles pour une calculatrice complète, parce que les règles métier sont ailleurs.

### 56.20.3 — Le scénario de recette

**État exécutable :** `flutter run`, puis déroulez ce scénario. Chaque ligne doit correspondre exactement.

```text
appui        écran affiché         ligne d'expression
──────       ─────────────────     ──────────────────
1 2 , 5      12,5                  (vide)
  +          12,5                  12,5 +
  4          4                     12,5 +
  =          16,5                  (vide)
  C          0                     (vide)
2 + 3        3                     2 +
  ×          5                     5 ×
  4          4                     5 ×
  =          20                    (vide)
  C          0                     (vide)
5 ÷ 0        0                     5 ÷
  =          Division par zéro     (vide)      <- en rouge
  C          0                     (vide)
```

Si tout correspond, l'application est fonctionnelle. Les deux dernières étapes ajoutent du confort.

---


## 56.21 — Étape 16 : le panneau d'historique

### 56.21.1 — Le widget

L'historique est une liste courte (50 lignes au maximum) mais potentiellement défilante : c'est le cas d'usage de `ListView.separated` (chapitre 48). Chaque ligne est un `ListTile` : le calcul à gauche, le résultat à droite.

**Fichier : `lib/ui/panneau_historique.dart`**

```dart
import 'package:flutter/material.dart';

import 'package:calculatrice/modeles/ligne_historique.dart';

/// La feuille d'historique, affichée en bas de l'écran.
///
/// Comme le reste de lib/ui, ce widget ne calcule rien : il reçoit une
/// liste déjà formatée et remonte les intentions de l'utilisateur.
class PanneauHistorique extends StatelessWidget {
  const PanneauHistorique({
    super.key,
    required this.lignes,
    required this.onVider,
    required this.onReprendre,
  });

  /// Les lignes à afficher, la plus récente en premier.
  final List<LigneHistorique> lignes;

  /// Appelé quand l'utilisateur veut tout effacer.
  final VoidCallback onVider;

  /// Appelé quand l'utilisateur touche une ligne pour la réutiliser.
  final void Function(LigneHistorique ligne) onReprendre;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return Column(
      children: <Widget>[
        Padding(
          padding: const EdgeInsets.fromLTRB(20, 4, 8, 4),
          child: Row(
            children: <Widget>[
              Text('Historique', style: theme.textTheme.titleLarge),
              const Spacer(),
              TextButton.icon(
                // null désactive le bouton : Flutter le grise tout seul.
                onPressed: lignes.isEmpty ? null : onVider,
                icon: const Icon(Icons.delete_outline),
                label: const Text('Vider'),
              ),
            ],
          ),
        ),
        const Divider(height: 1),
        Expanded(
          child: lignes.isEmpty
              ? const Center(child: Text('Aucun calcul pour le moment.'))
              : ListView.separated(
                  itemCount: lignes.length,
                  separatorBuilder: (BuildContext context, int index) =>
                      const Divider(height: 1),
                  itemBuilder: (BuildContext context, int index) {
                    final LigneHistorique ligne = lignes[index];
                    return ListTile(
                      title: Text(
                        ligne.expression,
                        style: theme.textTheme.bodyMedium?.copyWith(
                          color: theme.colorScheme.onSurfaceVariant,
                        ),
                      ),
                      trailing: Text(
                        ligne.resultat,
                        style: theme.textTheme.titleMedium
                            ?.copyWith(fontWeight: FontWeight.w600),
                      ),
                      onTap: () => onReprendre(ligne),
                    );
                  },
                ),
        ),
      ],
    );
  }
}
```


### 56.21.2 — L'ouvrir depuis l'`AppBar`

Ajoutez à `_EtatPageCalculatrice` la méthode ci-dessous, plus les imports de `ligne_historique.dart` et `panneau_historique.dart` :

```dart
  /// Ouvre l'historique dans une feuille qui monte du bas de l'écran.
  void _ouvrirHistorique() {
    showModalBottomSheet<void>(
      context: context,
      showDragHandle: true,
      builder: (BuildContext contexteFeuille) {
        return SizedBox(
          // La feuille occupe 60 % de la hauteur de l'écran.
          height: MediaQuery.sizeOf(context).height * 0.6,
          child: PanneauHistorique(
            lignes: _calculatrice.historique,
            onVider: () {
              // On ferme AVANT de modifier : la feuille a été construite
              // avec la liste d'alors, elle ne se reconstruirait pas.
              Navigator.pop(contexteFeuille);
              setState(_calculatrice.viderHistorique);
            },
            onReprendre: (LigneHistorique ligne) {
              Navigator.pop(contexteFeuille);
              setState(() => _calculatrice.reprendre(ligne.valeur));
            },
          ),
        );
      },
    );
  }
```

puis l'action dans l'`AppBar` :

```dart
        actions: <Widget>[
          IconButton(
            onPressed: _ouvrirHistorique,
            icon: const Icon(Icons.history),
            tooltip: 'Historique',
          ),
        ],
```

> Pourquoi fermer la feuille avant de vider ? Parce que `showModalBottomSheet` construit son contenu **une seule fois**, avec la liste passée à ce moment-là. Un `setState` de la page ne reconstruit pas la feuille : elle vit dans une autre route (chapitre 50). Fermer puis rouvrir est la solution la plus simple ; la solution avancée consiste à envelopper le contenu dans un `StatefulBuilder`.

**État exécutable :** `flutter run`. Faites trois calculs, ouvrez l'historique : les trois lignes sont là, la plus récente en haut. Touchez une ligne : son résultat revient sur l'écran principal. Appuyez sur « Vider » : la feuille se ferme et l'historique est vide au prochain passage.

---


## 56.22 — Étape 17 : le clavier physique

Sur ordinateur et sur le web, taper `1`, `2`, `+`, `3`, `Entrée` doit fonctionner. C'est l'exigence O18.

### 56.22.1 — Les trois outils

| Outil | Rôle |
| --- | --- |
| `FocusNode` | Le jeton qui dit « c'est moi qui reçois les touches » |
| `KeyboardListener` | Le widget qui écoute et appelle un callback |
| `KeyEvent` | L'événement reçu : `KeyDownEvent`, `KeyUpEvent`, `KeyRepeatEvent` |

Deux façons d'identifier une touche, et il faut les deux : `evenement.character` donne le **caractère produit** (`'7'`, `'+'`, `'*'`), idéal pour les chiffres et les opérateurs quelle que soit la disposition du clavier ; `evenement.logicalKey` donne la **touche logique** (`LogicalKeyboardKey.enter`, `.escape`, `.backspace`), indispensable pour les touches qui ne produisent aucun caractère.

### 56.22.2 — La table de correspondance

| Touche physique | Jeton envoyé | Touche physique | Jeton envoyé |
| --- | --- | --- | --- |
| `0` à `9` | `"0"` à `"9"` | `=` | `"="` |
| `.` ou `,` | `","` | `%` | `"%"` |
| `+` | `"+"` | Entrée / Entrée du pavé | `"="` |
| `-` | `"-"` | Échap | `"C"` |
| `*`, `x` ou `X` | `"×"` | Suppr | `"CE"` |
| `/` ou `:` | `"÷"` | Retour arrière | `"<"` |


### 56.22.3 — Le code

Ajoutez à `_EtatPageCalculatrice`, ainsi que l'import `package:flutter/services.dart` qui contient `KeyEvent` et `LogicalKeyboardKey` :

```dart
  /// Le noeud de focus qui capte les touches du clavier physique.
  final FocusNode _noeudClavier = FocusNode(debugLabel: 'clavier calculatrice');

  @override
  void dispose() {
    // Rappel du chapitre 45 : tout ce qui est créé dans un State et qui
    // possède des ressources doit être libéré ici.
    _noeudClavier.dispose();
    super.dispose();
  }

  /// Caractère tapé -> jeton de la calculatrice.
  static const Map<String, String> correspondances = <String, String>{
    '0': '0', '1': '1', '2': '2', '3': '3', '4': '4',
    '5': '5', '6': '6', '7': '7', '8': '8', '9': '9',
    '.': ',', ',': ',',
    '+': '+', '-': '-',
    '*': '×', 'x': '×', 'X': '×',
    '/': '÷', ':': '÷',
    '=': '=', '%': '%',
  };

  /// Appelée à chaque événement clavier.
  void _surToucheClavier(KeyEvent evenement) {
    // On ne réagit qu'à l'enfoncement. Sans ce filtre, chaque touche
    // compterait deux fois (enfoncement + relâchement), et une touche
    // maintenue produirait une rafale de KeyRepeatEvent.
    if (evenement is! KeyDownEvent) {
      return;
    }

    // 1) Les touches qui ne produisent aucun caractère.
    final LogicalKeyboardKey touche = evenement.logicalKey;
    if (touche == LogicalKeyboardKey.enter ||
        touche == LogicalKeyboardKey.numpadEnter) {
      _appuyer('=');
      return;
    }
    if (touche == LogicalKeyboardKey.escape) {
      _appuyer('C');
      return;
    }
    if (touche == LogicalKeyboardKey.delete) {
      _appuyer('CE');
      return;
    }
    if (touche == LogicalKeyboardKey.backspace) {
      _appuyer('<');
      return;
    }

    // 2) Les touches qui produisent un caractère.
    final String? caractere = evenement.character;
    if (caractere == null) {
      return;
    }
    final String? jeton = correspondances[caractere];
    if (jeton != null) {
      _appuyer(jeton);
    }
  }
```

Enfin, enveloppez le corps de la page. `autofocus: true` fait que la page reçoit les touches dès son ouverture, sans que l'utilisateur ait à cliquer dessus.

```dart
      body: KeyboardListener(
        focusNode: _noeudClavier,
        autofocus: true,
        onKeyEvent: _surToucheClavier,
        child: SafeArea(
          child: Column(
            children: <Widget>[
              // ... le contenu inchangé de la version 56.20.2 ...
            ],
          ),
        ),
      ),
```

> Attention à un piège : après avoir ouvert puis fermé la feuille d'historique, le focus est perdu et le clavier physique ne répond plus. Le correctif tient en trois lignes, à la fin de `_ouvrirHistorique()` :

```dart
  Future<void> _ouvrirHistorique() async {
    await showModalBottomSheet<void>(/* ... identique ... */);
    if (mounted) {
      _noeudClavier.requestFocus();
    }
  }
```

Le test `if (mounted)` est le réflexe du chapitre 15 : après un `await`, le widget a pu être retiré de l'arbre.

**État exécutable :** `flutter run -d chrome` (ou `-d windows`, `-d macos`, `-d linux`). Tapez `1`, `2`, `*`, `4`, `Entrée` : l'écran affiche `48`. Appuyez sur `Retour arrière` pendant une saisie : le dernier chiffre disparaît. Appuyez sur `Échap` : tout s'efface.

---


## 56.23 — Le projet terminé : où trouver chaque fichier

| Fichier | Version définitive donnée en |
| --- | --- |
| `pubspec.yaml` | 56.6 |
| `lib/main.dart` | 56.16.1 |
| `lib/logique/operation.dart`, `erreurs.dart` | 56.7 |
| `lib/logique/formatage.dart` | 56.8.3 |
| `lib/logique/calculatrice.dart` | 56.14.3 |
| `lib/modeles/ligne_historique.dart` | 56.14.1 |
| `lib/ui/ecran_calculatrice.dart` | 56.17.3 |
| `lib/ui/bouton_calc.dart` | 56.18.4 |
| `lib/ui/clavier_calculatrice.dart` | 56.19.3 |
| `lib/ui/panneau_historique.dart` | 56.21.1 |
| `lib/ui/page_calculatrice.dart` | 56.20.2, complété par 56.21.2 et 56.22.3 |
| `test/calculatrice_test.dart` | 56.15 |

Supprimez le brouillon, puis contrôlez le tout :

```bash
rm -rf bin/
flutter analyze
flutter test
```

`flutter analyze` doit répondre `No issues found!` et `flutter test`, `All tests passed!`. Dernière vérification, la plus importante du chapitre :

```bash
grep -r "package:flutter" lib/logique/ lib/modeles/
```

Cette commande ne doit **rien** afficher. Si elle affiche quelque chose, la frontière a été franchie : de la logique s'est mélangée à de l'interface, et le projet a perdu sa testabilité.

---


## 56.24 — Erreurs fréquentes

| Erreur | Symptôme | Correction |
| --- | --- | --- |
| Oublier `_nouvelleSaisie` | `7 + 8` affiche `78` | Repasser le drapeau à `true` dans `operateur()` et `egal()` |
| Stocker la saisie en `double` | Impossible d'afficher `1,0` pendant la frappe | Stocker un `String`, convertir seulement au calcul |
| Se fier à `a / 0` | L'écran affiche `Infinity` | Tester `b == 0` et lever `DivisionParZeroException` |
| Afficher `toString()` | `56.0` au lieu de `56` | Passer par `Formatage.versTexteBrut` |
| Ne pas arrondir | `0,1 + 0,2` affiche `0,30000000000000004` | `toStringAsPrecision(12)` puis retirer les zéros finaux |
| `double.parse('12.')` | `FormatException` | Retirer le point final avant de convertir |
| Répétition à l'envers | `7 × 8 = =` donne un résultat faux | L'ordre est `affichage OP dernierOperande` |
| Répétition qui survit | `2 + 3 = × 4 = =` refait `+3` | Mettre `_derniereOperation = null` dans `operateur()` |
| `CE` sans `_nouvelleSaisie` | Après `CE`, taper `5` affiche `05` | Remettre `_nouvelleSaisie = true` dans `effacerEntree()` |
| `%` qui remet `_nouvelleSaisie` à `true` | `200 + 10 %` puis `=` affiche `400` | Laisser `_nouvelleSaisie = false` après `pourcentage()` |
| `InkWell` sans `Material` | Le clic marche, aucune onde | Envelopper l'`InkWell` dans un `Material` |
| Couleur sur le `Container` | L'onde est invisible | Mettre la couleur sur le `Material` |
| Pas de `clipBehavior` | L'onde déborde des coins arrondis | `clipBehavior: Clip.antiAlias` |
| `Expanded` hors d'un flex | `Incorrect use of ParentDataWidget` | Placer `BoutonCalc` directement dans un `Row` |
| `FittedBox` sans `SizedBox` | La page saute à chaque chiffre | Fixer la hauteur avec un `SizedBox` |
| `GridView` pour le clavier | La grille défile et ne remplit pas la hauteur | `Column` + `Row` + `Expanded` |
| `setState` autour de rien | L'écran ne se met pas à jour | Mettre l'appel métier **dans** le `setState` |
| `FocusNode` non libéré | Fuite mémoire signalée en debug | Appeler `_noeudClavier.dispose()` |
| Réagir à tous les `KeyEvent` | Chaque touche compte double | `if (evenement is! KeyDownEvent) return;` |
| Feuille d'historique figée | « Vider » ne change rien à l'écran | Fermer la feuille avant de modifier |

---


## 56.25 — Ce que vous avez appris

| Notion | À retenir |
| --- | --- |
| Séparation logique / interface | La logique ne connaît pas Flutter ; l'interface ne calcule pas. Une seule flèche de dépendance, dans un seul sens. |
| Modéliser avant de coder | Lister les champs d'état et tracer le tableau de transitions évite les trois quarts des bugs. |
| Saisie textuelle | Pendant la frappe, la vérité est un `String` ; le `double` n'apparaît qu'au moment du calcul. |
| Le drapeau `_nouvelleSaisie` | Un seul booléen distingue « je continue à taper » de « je commence un nouveau nombre ». |
| Exception métier | `DivisionParZeroException` documente l'erreur, force l'appelant à la traiter et centralise son message. |
| IEEE 754 | `a / 0` ne plante pas, il donne `Infinity`. C'est au code métier de refuser l'opération. |
| Arrondi d'affichage | `toStringAsPrecision(12)` puis suppression des zéros finaux fait disparaître les poussières binaires. |
| Formatage local | Point décimal en interne, virgule et espaces insécables à l'affichage. Deux mondes, deux écritures. |
| `FittedBox` | `BoxFit.scaleDown` réduit sans jamais agrandir ; à combiner avec un `SizedBox` pour figer la hauteur. |
| `Row` + `Expanded` | Une grille fixe se construit avec des `Expanded`, pas avec un `GridView`. |
| `flex` | Deux parts au lieu d'une suffisent à faire un bouton double largeur. |
| Widget paramétré par un `enum` | Un `BoutonCalc` et un `TypeBouton` remplacent quatre widgets presque identiques. |
| `ColorScheme` | Une couleur source, un jeu complet de couleurs, le mode sombre gratuit. |
| `Material` + `InkWell` | La couleur sur le `Material`, le geste sur l'`InkWell`, `Clip.antiAlias` pour les coins. |
| `setState` | Il ne modifie rien : il signale qu'une modification a eu lieu. L'appel métier va à l'intérieur. |
| Remontée d'état | Le clavier ne connaît pas la calculatrice : il remonte un jeton, la page décide. |
| `ListView.separated` | La bonne liste pour un historique court mais défilant, avec des séparateurs gratuits. |
| `KeyboardListener` | `FocusNode` + filtrage `KeyDownEvent` + `character` pour les caractères, `logicalKey` pour le reste. |
| Tests unitaires | Une logique pure se teste en millisecondes ; une logique noyée dans un `State` demande des widget tests lents et fragiles. |

---

## 56.26 — Extensions : dix défis

Les dix défis sont classés du plus simple au plus exigeant. Chacun est réalisable sans nouveau package.

**Défi 1 — La racine carrée (facile).** Ajoutez une touche `√` qui remplace la saisie par sa racine carrée. *Indication :* importez `dart:math`, ajoutez `racineCarree()` dans `Calculatrice`. Une racine de nombre négatif doit lever une exception métier, sur le modèle de `DivisionParZeroException` : créez `RacineNegativeException`. Ajoutez la touche à la table `disposition`, en passant à six colonnes ou en ajoutant une ligne.

**Défi 2 — La mémoire `M+`, `M-`, `MR`, `MC` (facile).** Quatre touches et un champ `double _memoire = 0`. *Indication :* `M+` fait `_memoire += valeurCourante`, `MR` fait `_saisie = Formatage.versTexteBrut(_memoire)` avec `_nouvelleSaisie = false`. Ajoutez un getter `bool get memoireOccupee => _memoire != 0` pour afficher un `M` dans la ligne d'expression.

**Défi 3 — Corriger le pourcentage (facile).** Après un `%`, taper un chiffre l'ajoute au résultat au lieu de le remplacer (voir la remarque de 56.13.2). *Indication :* remplacez le booléen `_nouvelleSaisie` par un `enum EtatSaisie { neuve, enCours, resultat }`. L'état `resultat` se comporte comme `enCours` pour `=` et les opérateurs, et comme `neuve` pour les chiffres. Vos dix-neuf tests doivent continuer à passer : c'est là tout leur intérêt.

**Défi 4 — Copier le résultat (facile).** Un appui long sur l'écran copie le nombre affiché. *Indication :* `Clipboard.setData(ClipboardData(text: ...))` de `package:flutter/services.dart`, dans un `GestureDetector` avec `onLongPress`, confirmé par un `SnackBar`.

**Défi 5 — Persister l'historique (intermédiaire).** L'historique doit survivre à la fermeture de l'application. *Indication :* chapitre 54, `shared_preferences`. Sérialisez chaque `LigneHistorique` en JSON (chapitre 17) : ajoutez `Map<String, dynamic> versJson()` et un constructeur nommé `LigneHistorique.depuisJson(...)`. Chargez la liste dans `initState()`, sauvegardez après chaque calcul.

**Défi 6 — Un thème dédié (intermédiaire).** Fond très sombre, chiffres gris anthracite, opérateurs orange. *Indication :* ne touchez surtout pas à `BoutonCalc`. Construisez le `ColorScheme` à la main dans `main.dart`, ou partez de `ColorScheme.fromSeed(...)` et surchargez quelques rôles avec `copyWith`. Le reste suivra tout seul, ce qui prouve que la couleur n'était pas écrite en dur.

**Défi 7 — Une mise en page paysage enrichie (intermédiaire).** En paysage, affichez une sixième colonne de touches scientifiques. *Indication :* chapitre 51, `LayoutBuilder` ou `MediaQuery.orientationOf(context)`. Faites de `disposition` un paramètre de `ClavierCalculatrice` et fournissez deux tables.

**Défi 8 — La priorité des opérateurs (difficile).** `2 + 3 × 4` doit valoir `14`. *Indication :* remplacez l'accumulateur unique par deux piles, une de `double` et une d'`Operation`. À chaque nouvel opérateur, dépilez tant que l'opérateur au sommet est de priorité supérieure ou égale (algorithme « shunting-yard » simplifié). Écrivez d'abord les tests, ensuite le code. Ne modifiez que `calculatrice.dart` : l'interface ne doit pas bouger d'une ligne.

**Défi 9 — Les parenthèses (difficile).** Ajoutez les touches `(` et `)`. *Indication :* suppose le défi 8 fait. Une parenthèse ouvrante empile un marqueur, une fermante dépile jusqu'au marqueur. Affichez le nombre de parenthèses encore ouvertes et refusez `=` tant qu'il en reste.

**Défi 10 — Un ruban d'historique en direct (difficile).** Bandeau latéral permanent sur écran large, feuille modale sur téléphone. *Indication :* `LayoutBuilder` avec un seuil vers 700 pixels (chapitre 51). Au-delà, un `Row` : la calculatrice dans un `Expanded`, le `PanneauHistorique` dans un `SizedBox(width: 320)`. Vous réutilisez `PanneauHistorique` sans le modifier, signe qu'il était bien découpé, et le rafraîchissement en direct devient gratuit puisque le panneau est reconstruit par le même `setState`.

---


## Et maintenant ?

Vous venez de construire une application dont le cœur est une classe Dart de trois cents lignes, testée par dix-neuf tests, et dont l'interface complète tient en cinq widgets. C'est le modèle de toutes les applications sérieuses : **la logique d'abord, l'écran ensuite**.

Le projet 3 poursuit dans cette direction, avec un métier différent : convertir des unités. Vous y découvrirez la saisie libre au clavier avec `TextField` et `TextEditingController`, le choix dans une liste avec `DropdownButton`, et surtout la validation d'un formulaire — car cette fois, l'utilisateur pourra taper n'importe quoi, y compris `12,,5` ou `abc`.

Rendez-vous au chapitre 57 : [57-PARTIE-1C—PROJET-3-CONVERTISSEUR.md](./57-PARTIE-1C—PROJET-3-CONVERTISSEUR.md)
