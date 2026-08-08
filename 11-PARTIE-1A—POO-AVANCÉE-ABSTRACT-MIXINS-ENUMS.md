# PARTIE 1A — DART
# CHAPITRE 11 — POO AVANCÉE : CLASSES ABSTRAITES, INTERFACES, MIXINS, ENUMS ET EXTENSIONS

> **Niveau :** intermédiaire
> **Durée estimée :** 7 h
> **Pré-requis :** chapitre 10 — Encapsulation, héritage, polymorphisme
> **Ce que vous saurez faire à la fin :** concevoir une hiérarchie de classes propre avec des classes abstraites, des interfaces, des mixins, des enums et des extensions, et choisir le bon outil pour chaque besoin.

---

## 11.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer pourquoi l'héritage simple ne suffit pas toujours ;
- comprendre ce qu'est une classe abstraite ;
- utiliser le mot-clé `abstract` ;
- écrire une méthode abstraite, c'est-à-dire une méthode sans corps ;
- comprendre pourquoi une classe abstraite ne s'instancie pas ;
- mélanger dans une même classe des méthodes concrètes et des méthodes abstraites ;
- comprendre la notion d'interface implicite propre à Dart ;
- utiliser le mot-clé `implements` ;
- distinguer clairement `extends` et `implements` ;
- implémenter plusieurs interfaces sur une même classe ;
- expliquer le problème que résolvent les mixins ;
- déclarer un `mixin` et l'appliquer avec `with` ;
- appliquer plusieurs mixins et comprendre l'ordre de résolution ;
- restreindre un mixin avec le mot-clé `on` ;
- déclarer et utiliser un `enum` ;
- écrire un `switch` sur un enum ;
- utiliser `values`, `index` et `name` ;
- écrire un enhanced enum avec des propriétés et des méthodes ;
- ajouter des fonctionnalités à un type existant avec `extension ... on` ;
- choisir entre composition et héritage ;
- savoir quel outil utiliser pour quel besoin.

---

## 11.1 — Pourquoi aller plus loin que l'héritage simple ?

Au chapitre 10, nous avons appris l'héritage. Une classe `Ennemi` peut hériter d'une classe `Personnage` :

```dart
class Personnage {
  String nom;
  int pv;

  Personnage(this.nom, this.pv);

  void seDecrire() {
    print('$nom possède $pv PV.');
  }
}

class Ennemi extends Personnage {
  Ennemi(String nom, int pv) : super(nom, pv);
}
```

C'est très pratique. Mais l'héritage a une limite très stricte en Dart :

```text
Une classe ne peut hériter que d'UNE SEULE classe parente.
```

Prenons un cas concret de jeu vidéo. Nous avons trois créatures :

```text
Dragon        -> vole
Requin        -> nage
DragonMarin   -> vole ET nage
```

Avec l'héritage simple, nous sommes bloqués :

```text
        Creature
        /      \
    Volante   Nageuse
        \      /
         ??????        <- impossible en Dart
       DragonMarin
```

`DragonMarin` ne peut pas écrire `extends Volante, Nageuse`. Ce n'est pas autorisé.

Nous avons donc besoin d'autres outils :

| Besoin | Outil |
| --- | --- |
| Décrire un modèle incomplet, à compléter | classe abstraite |
| Imposer un contrat (« tu dois savoir faire X ») | interface / `implements` |
| Greffer un bloc de comportement réutilisable | mixin / `with` |
| Représenter une liste fermée de valeurs | `enum` |
| Ajouter une méthode à un type existant | `extension` |

Ce chapitre présente ces cinq outils, un par un.

---

## 11.2 — Classes abstraites : l'idée

Posons-nous une question simple.

Dans notre jeu, nous avons des `Guerrier`, des `Mage`, des `Archer`. Tous sont des `Personnage`.

Mais est-ce que le joueur peut jouer un « Personnage » tout court ?

```text
Non.
Un Personnage générique n'existe pas dans le jeu.
C'est seulement un modèle commun.
```

`Personnage` sert uniquement à :

1. rassembler ce que toutes les classes ont en commun (nom, PV) ;
2. imposer ce que toutes les classes doivent savoir faire (attaquer).

Une classe qui sert de modèle mais qu'on ne veut jamais créer directement s'appelle :

```text
une classe abstraite
```

Retenez l'image suivante :

```text
Classe abstraite = un moule
Classe concrète  = un objet sorti du moule

On peut fabriquer une pièce avec le moule.
On ne peut pas jouer avec le moule lui-même.
```

---

## 11.3 — Le mot-clé `abstract`

Pour déclarer une classe abstraite, on place le mot-clé `abstract` devant `class` :

```dart
abstract class Personnage {
  String nom;
  int pv;

  Personnage(this.nom, this.pv);

  void seDecrire() {
    print('$nom possède $pv PV.');
  }
}

class Guerrier extends Personnage {
  Guerrier(String nom, int pv) : super(nom, pv);
}

void main() {
  Guerrier g = Guerrier('Kael', 120);
  g.seDecrire();
}
```

**Résultat :**

```text
Kael possède 120 PV.
```

Remarquez bien :

- `Personnage` est abstraite ;
- `Guerrier` est une classe normale, dite **concrète** ;
- `Guerrier` hérite de `Personnage` avec `extends`, exactement comme au chapitre 10.

Une classe abstraite peut donc contenir :

- des attributs ;
- un constructeur ;
- des méthodes complètes.

La seule différence pour l'instant : on ne peut pas écrire `Personnage('X', 10)`. Nous verrons cela en 11.5.

---

## 11.4 — Méthodes abstraites (sans corps)

C'est ici que les classes abstraites deviennent réellement utiles.

Une **méthode abstraite** est une méthode déclarée sans corps. Elle se termine par un point-virgule, sans accolades :

```dart
void attaquer();
```

Elle signifie :

```text
« Toute classe qui hérite de moi DOIT écrire cette méthode. »
```

Exemple complet :

```dart
abstract class Personnage {
  String nom;
  int pv;

  Personnage(this.nom, this.pv);

  void attaquer();
}

class Guerrier extends Personnage {
  Guerrier(String nom, int pv) : super(nom, pv);

  @override
  void attaquer() {
    print('$nom donne un coup d\'épée.');
  }
}

class Mage extends Personnage {
  Mage(String nom, int pv) : super(nom, pv);

  @override
  void attaquer() {
    print('$nom lance une boule de feu.');
  }
}

void main() {
  List<Personnage> equipe = [
    Guerrier('Kael', 120),
    Mage('Lyra', 80),
  ];

  for (Personnage p in equipe) {
    p.attaquer();
  }
}
```

**Résultat :**

```text
Kael donne un coup d'épée.
Lyra lance une boule de feu.
```

Ce que nous venons de gagner est énorme :

```text
Le compilateur GARANTIT que chaque personnage sait attaquer.
```

Si vous créez une classe `Archer extends Personnage` en oubliant `attaquer()`, le programme refuse de compiler :

```text
Missing concrete implementation of 'Personnage.attaquer'.
Try implementing the missing method.
```

L'erreur apparaît **avant** l'exécution. C'est exactement le but.

---

## 11.5 — Une classe abstraite ne s'instancie pas

Essayons volontairement d'instancier une classe abstraite :

```dart
abstract class Personnage {
  void attaquer();
}

void main() {
  Personnage p = Personnage(); // interdit
}
```

**Résultat :**

```text
Error: The class 'Personnage' is abstract and can't be instantiated.
```

C'est logique. Dart ne saurait pas quoi faire si l'on appelait `p.attaquer()` : la méthode n'a pas de corps.

En revanche, une chose est parfaitement autorisée et très utile :

```dart
abstract class Personnage {
  String nom;
  Personnage(this.nom);
  void attaquer();
}

class Mage extends Personnage {
  Mage(String nom) : super(nom);

  @override
  void attaquer() {
    print('$nom lance un éclair.');
  }
}

void main() {
  Personnage p = Mage('Lyra'); // autorisé
  p.attaquer();
}
```

**Résultat :**

```text
Lyra lance un éclair.
```

Retenez la distinction :

```text
Personnage p = Personnage();  -> INTERDIT (on crée un objet abstrait)
Personnage p = Mage('Lyra');  -> AUTORISÉ (le TYPE est abstrait, l'OBJET est concret)
```

Une classe abstraite est donc un excellent **type de variable**. C'est ce qui permet d'écrire `List<Personnage>` en y rangeant des `Guerrier` et des `Mage`.

---

## 11.6 — Mélanger méthodes concrètes et abstraites

Une classe abstraite n'est pas obligée d'être vide. Elle peut très bien fournir du code déjà écrit.

C'est même son plus grand intérêt : **factoriser ce qui est commun, imposer ce qui est spécifique.**

```dart
abstract class Personnage {
  String nom;
  int pv;

  Personnage(this.nom, this.pv);

  // Méthode CONCRÈTE : identique pour tout le monde.
  void subirDegats(int montant) {
    pv = pv - montant;
    if (pv < 0) {
      pv = 0;
    }
    print('$nom subit $montant dégâts. PV restants : $pv');
  }

  // Méthode CONCRÈTE : identique pour tout le monde.
  bool estVivant() {
    return pv > 0;
  }

  // Méthode ABSTRAITE : chacun sa version.
  void attaquer();
}

class Guerrier extends Personnage {
  Guerrier(String nom, int pv) : super(nom, pv);

  @override
  void attaquer() {
    print('$nom frappe avec sa hache.');
  }
}

class Mage extends Personnage {
  Mage(String nom, int pv) : super(nom, pv);

  @override
  void attaquer() {
    print('$nom invoque une tempête de glace.');
  }
}

void main() {
  Guerrier kael = Guerrier('Kael', 120);
  Mage lyra = Mage('Lyra', 80);

  kael.attaquer();
  lyra.attaquer();

  kael.subirDegats(30);
  lyra.subirDegats(100);

  print('Kael vivant ? ${kael.estVivant()}');
  print('Lyra vivante ? ${lyra.estVivant()}');
}
```

**Résultat :**

```text
Kael frappe avec sa hache.
Lyra invoque une tempête de glace.
Kael subit 30 dégâts. PV restants : 90
Lyra subit 100 dégâts. PV restants : 0
Kael vivant ? true
Lyra vivante ? false
```

Le code de `subirDegats()` est écrit **une seule fois**. Il est partagé par toutes les sous-classes.

Résumons la règle :

| Type de méthode | Écrite dans la classe abstraite | Obligatoire dans l'enfant |
| --- | --- | --- |
| Concrète | Oui, avec un corps | Non (héritée telle quelle) |
| Abstraite | Oui, sans corps | Oui, sinon erreur de compilation |

---

## 11.7 — Interfaces implicites en Dart

Voici une particularité de Dart qui surprend souvent les débutants venus d'autres langages.

```text
En Dart, il n'existe pas de mot-clé `interface` pour déclarer une interface.
Toute classe définit AUTOMATIQUEMENT une interface implicite.
```

Qu'est-ce qu'une interface ? C'est **la liste des membres publics** d'une classe : ses attributs, ses getters, ses setters et ses méthodes, sans leur code.

Prenons cette classe :

```dart
class Arme {
  String nom = 'Épée rouillée';
  int degats = 5;

  void frapper() {
    print('$nom inflige $degats dégâts.');
  }
}
```

Dart en déduit tout seul l'interface suivante :

```text
Interface implicite de Arme
├── String nom       (lecture + écriture)
├── int degats       (lecture + écriture)
└── void frapper()
```

Cette interface est un **contrat**. Une autre classe peut promettre de le respecter, sans reprendre le moindre bout de code de `Arme`.

C'est exactement ce que fait `implements`, que nous voyons maintenant.

---

## 11.8 — `implements`

`implements` signifie :

```text
« Je promets de fournir TOUS les membres de cette interface,
  mais je ne récupère AUCUN code. »
```

Exemple :

```dart
class Arme {
  String nom = 'Épée rouillée';
  int degats = 5;

  void frapper() {
    print('$nom inflige $degats dégâts.');
  }
}

class Sortilege implements Arme {
  @override
  String nom = 'Boule de feu';

  @override
  int degats = 25;

  @override
  void frapper() {
    print('$nom brûle la cible pour $degats dégâts.');
  }
}

void main() {
  Arme a = Arme();
  Arme s = Sortilege();

  a.frapper();
  s.frapper();
}
```

**Résultat :**

```text
Épée rouillée inflige 5 dégâts.
Boule de feu brûle la cible pour 25 dégâts.
```

Trois points importants :

1. `Sortilege` n'hérite de rien. Le code de `frapper()` de `Arme` n'a pas été réutilisé.
2. `Sortilege` a dû **tout** redéfinir : `nom`, `degats` et `frapper()`. Oublier un seul membre provoque une erreur de compilation.
3. Malgré tout, `Sortilege` est bien **de type** `Arme`. On peut écrire `Arme s = Sortilege();`.

Si l'on oublie un membre :

```dart
class Sortilege implements Arme {
  @override
  void frapper() {
    print('Boum.');
  }
}
```

**Résultat :**

```text
Error: Missing concrete implementations of 'Arme.degats',
'Arme.degats=', 'Arme.nom' and 'Arme.nom='.
```

En pratique, on n'implémente presque jamais une classe concrète. On implémente plutôt une classe abstraite qui ne contient que des méthodes abstraites, ce qui rend le contrat évident :

```dart
abstract class Sauvegardable {
  Map<String, Object> versJson();
}

class Joueur implements Sauvegardable {
  String pseudo;
  int niveau;

  Joueur(this.pseudo, this.niveau);

  @override
  Map<String, Object> versJson() {
    return {'pseudo': pseudo, 'niveau': niveau};
  }
}

void main() {
  Sauvegardable s = Joueur('Kael', 7);
  print(s.versJson());
}
```

**Résultat :**

```text
{pseudo: Kael, niveau: 7}
```

---

## 11.9 — `extends` vs `implements`

C'est la confusion numéro un des débutants. Voici le schéma à retenir.

```text
                    extends                        implements
                 (héritage)                        (contrat)

   ┌──────────────────────────┐        ┌──────────────────────────┐
   │        Parent            │        │        Parent            │
   │  nom = 'Épée'            │        │  nom = 'Épée'            │
   │  frapper() { ...code...} │        │  frapper() { ...code...} │
   └────────────┬─────────────┘        └────────────┬─────────────┘
                │                                   │
      LE CODE DESCEND                     SEULE LA FORME DESCEND
      (attributs + corps                  (juste la liste des
       des méthodes)                       membres à fournir)
                │                                   │
                v                                   v
   ┌──────────────────────────┐        ┌──────────────────────────┐
   │        Enfant            │        │        Enfant            │
   │  nom hérité              │        │  nom : À RÉÉCRIRE        │
   │  frapper() hérité        │        │  frapper() : À RÉÉCRIRE  │
   │  (on peut redéfinir      │        │  (tout est obligatoire)  │
   │   si on veut)            │        │                          │
   └──────────────────────────┘        └──────────────────────────┘

   super.frapper() disponible          super.frapper() INDISPONIBLE
   1 seul parent maximum               autant d'interfaces qu'on veut
```

Sous forme de tableau :

| Critère | `extends` | `implements` |
| --- | --- | --- |
| Nombre autorisé | 1 seul | illimité |
| Le code est récupéré | Oui | Non |
| Redéfinir est obligatoire | Non | Oui, pour tout |
| Accès à `super` | Oui | Non |
| Question à se poser | « est-ce une sorte de… ? » | « sait-il faire… ? » |

Petit test mental :

```text
Un Mage EST UNE SORTE DE Personnage       -> extends
Un Coffre SAIT SE SAUVEGARDER              -> implements Sauvegardable
Un Boss EST UNE SORTE D'Ennemi             -> extends
Un Boss SAIT AFFICHER UNE BARRE DE VIE     -> implements Affichable
```

Petit rappel utile : on peut combiner les deux dans la même déclaration, dans cet ordre exact :

```dart
class Boss extends Ennemi implements Sauvegardable {
  // ...
}
```

---

## 11.10 — Implémenter plusieurs interfaces

Contrairement à `extends`, `implements` accepte plusieurs types, séparés par des virgules.

```dart
abstract class Sauvegardable {
  String versTexte();
}

abstract class Affichable {
  void afficher();
}

class Joueur implements Sauvegardable, Affichable {
  String pseudo;
  int niveau;

  Joueur(this.pseudo, this.niveau);

  @override
  String versTexte() {
    return '$pseudo;$niveau';
  }

  @override
  void afficher() {
    print('[$pseudo] niveau $niveau');
  }
}

void main() {
  Joueur j = Joueur('Kael', 7);

  j.afficher();
  print(j.versTexte());

  print(j is Sauvegardable);
  print(j is Affichable);
}
```

**Résultat :**

```text
[Kael] niveau 7
Kael;7
true
true
```

Un objet peut donc porter **plusieurs identités** en même temps. C'est très pratique pour écrire des fonctions génériques :

```dart
abstract class Affichable {
  void afficher();
}

class Joueur implements Affichable {
  String pseudo;
  Joueur(this.pseudo);

  @override
  void afficher() {
    print('Joueur : $pseudo');
  }
}

class Potion implements Affichable {
  int soin;
  Potion(this.soin);

  @override
  void afficher() {
    print('Potion : +$soin PV');
  }
}

void afficherTout(List<Affichable> elements) {
  for (Affichable e in elements) {
    e.afficher();
  }
}

void main() {
  afficherTout([
    Joueur('Kael'),
    Potion(50),
    Joueur('Lyra'),
  ]);
}
```

**Résultat :**

```text
Joueur : Kael
Potion : +50 PV
Joueur : Lyra
```

La fonction `afficherTout()` ne connaît ni `Joueur` ni `Potion`. Elle connaît seulement le contrat `Affichable`. C'est cela, programmer contre une interface.

---

## 11.11 — Les mixins : le problème qu'ils résolvent

Revenons au problème du 11.1.

Nous avons trois créatures et deux capacités :

```text
Dragon        -> voler
Requin        -> nager
DragonMarin   -> voler ET nager
```

Solution 1 : l'héritage. Impossible, une seule classe parente.

Solution 2 : les interfaces. Techniquement possible, mais catastrophique :

```dart
abstract class Volant {
  void voler();
}

abstract class Nageur {
  void nager();
}

class Dragon implements Volant {
  @override
  void voler() {
    print('Le dragon s\'envole.');
  }
}

class DragonMarin implements Volant, Nageur {
  @override
  void voler() {
    print('Le dragon marin s\'envole.'); // code DUPLIQUÉ
  }

  @override
  void nager() {
    print('Le dragon marin plonge.');
  }
}
```

Le problème saute aux yeux :

```text
implements ne transporte AUCUN code.
Chaque classe doit RÉÉCRIRE voler() et nager().
Avec 10 créatures volantes, le code de voler() est copié 10 fois.
```

Solution 3 : le mixin. Un mixin est exactement l'inverse d'une interface :

```text
interface = du contrat SANS code
mixin     = du code SANS obligation de parenté
```

Un mixin est un **bloc de comportement réutilisable** que l'on greffe sur autant de classes qu'on veut.

Voici le schéma à retenir :

```text
      LES BRIQUES DE COMPORTEMENT (mixins)

      ┌──────────┐  ┌──────────┐  ┌──────────┐
      │  Volant  │  │  Nageur  │  │ Invisible│
      │ voler()  │  │ nager()  │  │ cacher() │
      └────┬─────┘  └────┬─────┘  └────┬─────┘
           │             │             │
           │  on greffe la brique avec `with`
           v             v             v
   ┌───────────────────────────────────────────┐
   │  class DragonMarin extends Creature       │
   │        with Volant, Nageur                │
   │                                           │
   │   [ socle Creature : nom, pv, ... ]       │
   │   [ + brique Volant   : voler()   ]       │
   │   [ + brique Nageur   : nager()   ]       │
   └───────────────────────────────────────────┘

   Le code de voler() est écrit UNE SEULE FOIS,
   dans le mixin, et réutilisé partout.
```

---

## 11.12 — `mixin` et `with`

Un mixin se déclare avec le mot-clé `mixin`, et s'applique avec le mot-clé `with`.

```dart
mixin Volant {
  int altitude = 0;

  void voler() {
    altitude = 100;
    print('Décollage. Altitude : $altitude m.');
  }

  void atterrir() {
    altitude = 0;
    print('Atterrissage.');
  }
}

mixin Nageur {
  int profondeur = 0;

  void nager() {
    profondeur = 20;
    print('Plongée. Profondeur : $profondeur m.');
  }
}

class Creature {
  String nom;
  int pv;

  Creature(this.nom, this.pv);

  void seDecrire() {
    print('$nom ($pv PV)');
  }
}

class Dragon extends Creature with Volant {
  Dragon(String nom, int pv) : super(nom, pv);
}

class Requin extends Creature with Nageur {
  Requin(String nom, int pv) : super(nom, pv);
}

class DragonMarin extends Creature with Volant, Nageur {
  DragonMarin(String nom, int pv) : super(nom, pv);
}

void main() {
  Dragon d = Dragon('Ignis', 300);
  d.seDecrire();
  d.voler();

  Requin r = Requin('Croc', 150);
  r.seDecrire();
  r.nager();

  DragonMarin dm = DragonMarin('Leviathan', 500);
  dm.seDecrire();
  dm.voler();
  dm.nager();
  dm.atterrir();
}
```

**Résultat :**

```text
Ignis (300 PV)
Décollage. Altitude : 100 m.
Croc (150 PV)
Plongée. Profondeur : 20 m.
Leviathan (500 PV)
Décollage. Altitude : 100 m.
Plongée. Profondeur : 20 m.
Atterrissage.
```

Points essentiels :

- un mixin peut contenir des **attributs** et des **méthodes avec leur code** ;
- un mixin **n'a pas de constructeur** ; c'est la règle qui le distingue d'une classe ;
- une classe peut recevoir **autant de mixins qu'elle veut** ;
- l'ordre est toujours : `class X extends Parent with M1, M2 implements I1, I2`.

Un mixin crée aussi un type utilisable :

```dart
mixin Volant {
  void voler() => print('Je vole.');
}

class Creature {}

class Dragon extends Creature with Volant {}

class Rat extends Creature {}

void main() {
  List<Creature> zoo = [Dragon(), Rat()];

  for (Creature c in zoo) {
    if (c is Volant) {
      c.voler();
    } else {
      print('Cette créature reste au sol.');
    }
  }
}
```

**Résultat :**

```text
Je vole.
Cette créature reste au sol.
```

---

## 11.13 — Plusieurs mixins et ordre de résolution

Que se passe-t-il si deux mixins définissent la **même** méthode ?

```dart
mixin Feu {
  String element() => 'Feu';
}

mixin Glace {
  String element() => 'Glace';
}

class Sort {}

class SortA extends Sort with Feu, Glace {}

class SortB extends Sort with Glace, Feu {}

void main() {
  print(SortA().element());
  print(SortB().element());
}
```

**Résultat :**

```text
Glace
Feu
```

La règle est simple et absolue :

```text
Le DERNIER mixin déclaré gagne.
On lit `with A, B, C` de gauche à droite : C écrase B, qui écrase A.
```

Visuellement, Dart empile les mixins :

```text
   class SortA extends Sort with Feu, Glace

   ┌───────────────┐
   │    Glace      │  <- appliqué en DERNIER, donc au-dessus
   ├───────────────┤
   │     Feu       │
   ├───────────────┤
   │    Sort       │  <- la classe parente, tout en bas
   └───────────────┘

   Un appel part du HAUT et descend jusqu'à trouver la méthode.
   element() est trouvé dans Glace  ->  'Glace'
```

Grâce à cet empilement, un mixin peut appeler `super` pour atteindre la couche du dessous. C'est la technique dite de « décoration » :

```dart
class Attaque {
  int degats() => 10;
}

mixin BonusFeu on Attaque {
  @override
  int degats() => super.degats() + 5;
}

mixin BonusCritique on Attaque {
  @override
  int degats() => super.degats() * 2;
}

void main() {
  print(AttaqueA().degats());
  print(AttaqueB().degats());
}

class AttaqueA extends Attaque with BonusFeu, BonusCritique {}

class AttaqueB extends Attaque with BonusCritique, BonusFeu {}
```

**Résultat :**

```text
30
25
```

Détaillons le calcul, c'est très instructif :

```text
AttaqueA : Attaque -> BonusFeu -> BonusCritique
  BonusCritique.degats() = super.degats() * 2
                         = BonusFeu.degats() * 2
                         = (Attaque.degats() + 5) * 2
                         = (10 + 5) * 2
                         = 30

AttaqueB : Attaque -> BonusCritique -> BonusFeu
  BonusFeu.degats() = super.degats() + 5
                    = BonusCritique.degats() + 5
                    = (Attaque.degats() * 2) + 5
                    = (10 * 2) + 5
                    = 25
```

Même code, ordre différent, résultat différent. L'ordre des mixins compte réellement.

---

## 11.14 — `on` pour restreindre un mixin

Dans l'exemple précédent, `BonusFeu` a écrit `super.degats()`. Comment Dart peut-il savoir que `super` possède une méthode `degats()` ?

Grâce au mot-clé `on` :

```dart
mixin BonusFeu on Attaque {
  // ici, super est forcément une Attaque
}
```

`on Attaque` signifie :

```text
« Ce mixin ne peut être greffé QUE sur une classe
  qui est (ou hérite de) Attaque. »
```

En échange de cette restriction, le mixin obtient deux droits :

1. accéder aux membres de `Attaque` ;
2. appeler `super`.

Exemple complet dans notre univers de jeu :

```dart
class Personnage {
  String nom;
  int pv;
  int pvMax;

  Personnage(this.nom, this.pv, this.pvMax);
}

mixin Regeneration on Personnage {
  void regenerer(int montant) {
    pv = pv + montant;
    if (pv > pvMax) {
      pv = pvMax;
    }
    print('$nom se régénère : $pv/$pvMax PV.');
  }
}

class Troll extends Personnage with Regeneration {
  Troll(String nom, int pv, int pvMax) : super(nom, pv, pvMax);
}

void main() {
  Troll t = Troll('Grosh', 40, 100);
  t.regenerer(30);
  t.regenerer(50);
}
```

**Résultat :**

```text
Grosh se régénère : 70/100 PV.
Grosh se régénère : 100/100 PV.
```

Le mixin `Regeneration` utilise `nom`, `pv` et `pvMax` sans les déclarer. Il sait qu'ils existent, parce que `on Personnage` le garantit.

Maintenant, tentons de greffer ce mixin sur une classe qui n'est pas un `Personnage` :

```dart
class Coffre {}

class CoffreVivant extends Coffre with Regeneration {} // interdit
```

**Résultat :**

```text
Error: 'Regeneration' can't be mixed onto 'Coffre'
because 'Coffre' doesn't implement 'Personnage'.
```

Retenez la comparaison :

| Déclaration | Signification |
| --- | --- |
| `mixin Volant { }` | greffable sur n'importe quelle classe |
| `mixin Volant on Creature { }` | greffable uniquement sur une `Creature` |

Utilisez `on` dès que votre mixin a besoin des données de la classe cible. C'est un filet de sécurité, pas une contrainte gênante.

---

## 11.15 — Les `enum`

Changeons de sujet. Nous allons résoudre un autre problème classique.

Dans un jeu, un sort possède un élément : feu, eau, terre ou air. Comment stocker cette information ?

Première idée, avec une `String` :

```dart
String element = 'feu';
```

C'est fragile :

```text
'feu'   'Feu'   'FEU'   'feuu'   'fue'
```

Toutes ces valeurs compilent sans erreur, et pourtant quatre sur cinq sont fausses. L'erreur n'apparaîtra qu'à l'exécution, peut-être des mois plus tard.

Deuxième idée, avec un `int` :

```dart
int element = 0; // 0 = feu, 1 = eau...
```

C'est illisible, et rien n'empêche d'écrire `element = 47;`.

La bonne solution est l'`enum` (énumération). Un enum définit une **liste fermée de valeurs possibles**, connues du compilateur :

```dart
enum TypeElement { feu, eau, terre, air }

void main() {
  TypeElement e = TypeElement.feu;
  print(e);
}
```

**Résultat :**

```text
TypeElement.feu
```

Trois règles :

- l'enum se déclare **en dehors** de `main()`, au même niveau qu'une classe ;
- on accède toujours à une valeur par `NomDeLEnum.valeur` ;
- écrire `TypeElement.metal` provoque une erreur immédiate, car cette valeur n'existe pas.

Comparaison :

```dart
enum TypeElement { feu, eau, terre, air }

void main() {
  TypeElement a = TypeElement.feu;
  TypeElement b = TypeElement.feu;
  TypeElement c = TypeElement.eau;

  print(a == b);
  print(a == c);
}
```

**Résultat :**

```text
true
false
```

---

## 11.16 — `switch` sur un enum

L'enum et le `switch` forment un couple parfait.

```dart
enum TypeElement { feu, eau, terre, air }

void decrireElement(TypeElement element) {
  switch (element) {
    case TypeElement.feu:
      print('Le feu inflige des brûlures.');
      break;
    case TypeElement.eau:
      print('L\'eau éteint les flammes.');
      break;
    case TypeElement.terre:
      print('La terre protège et ralentit.');
      break;
    case TypeElement.air:
      print('L\'air augmente la vitesse.');
      break;
  }
}

void main() {
  decrireElement(TypeElement.feu);
  decrireElement(TypeElement.air);
}
```

**Résultat :**

```text
Le feu inflige des brûlures.
L'air augmente la vitesse.
```

L'avantage décisif est ailleurs. Supposons que vous ajoutiez un cinquième élément :

```dart
enum TypeElement { feu, eau, terre, air, foudre }
```

Le compilateur vous signale immédiatement le `switch` incomplet :

```text
Error: The type 'TypeElement' is not exhaustively matched
by the switch cases since it doesn't match 'TypeElement.foudre'.
```

Avec des `String`, personne ne vous aurait prévenu. C'est la raison principale d'utiliser des enums.

Notez que si vous couvrez tous les cas, le `default` devient inutile. Et si vous en oubliez un, Dart le voit. Ne mettez donc pas de `default` par réflexe sur un enum : vous perdriez cette vérification.

---

## 11.17 — `values`, `index`, `name`

Tout enum fournit gratuitement trois outils.

**`values`** est la liste de toutes les valeurs, dans l'ordre de déclaration :

```dart
enum TypeElement { feu, eau, terre, air }

void main() {
  print(TypeElement.values);
  print(TypeElement.values.length);

  for (TypeElement t in TypeElement.values) {
    print(t);
  }
}
```

**Résultat :**

```text
[TypeElement.feu, TypeElement.eau, TypeElement.terre, TypeElement.air]
4
TypeElement.feu
TypeElement.eau
TypeElement.terre
TypeElement.air
```

**`index`** est la position de la valeur, en partant de 0 :

```dart
enum TypeElement { feu, eau, terre, air }

void main() {
  print(TypeElement.feu.index);
  print(TypeElement.terre.index);
  print(TypeElement.values[2]);
}
```

**Résultat :**

```text
0
2
TypeElement.terre
```

**`name`** est le nom écrit dans le code, sous forme de `String` :

```dart
enum TypeElement { feu, eau, terre, air }

void main() {
  print(TypeElement.feu.name);
  print(TypeElement.feu.toString());
  print('Élément : ${TypeElement.eau.name}');
}
```

**Résultat :**

```text
feu
TypeElement.feu
Élément : eau
```

Retenez la différence :

```text
.name       -> 'feu'                (à afficher au joueur)
.toString() -> 'TypeElement.feu'    (pour le débogage)
.index      -> 0                    (position technique)
```

Un cas très fréquent : afficher un menu numéroté.

```dart
enum TypeElement { feu, eau, terre, air }

void main() {
  for (TypeElement t in TypeElement.values) {
    print('${t.index + 1}. ${t.name}');
  }
}
```

**Résultat :**

```text
1. feu
2. eau
3. terre
4. air
```

---

## 11.18 — Enhanced enums (enums avec propriétés et méthodes)

Un enum simple ne transporte qu'un nom. Or, dans un vrai jeu, chaque élément a des caractéristiques : un libellé affichable, des dégâts de base, une faiblesse.

Depuis Dart 2.17, un enum peut posséder des attributs, un constructeur et des méthodes. On parle d'**enhanced enum** (enum enrichi).

```dart
enum TypeElement {
  feu('Feu', 12),
  eau('Eau', 10),
  terre('Terre', 8),
  air('Air', 9);

  const TypeElement(this.libelle, this.degatsBase);

  final String libelle;
  final int degatsBase;

  TypeElement get faiblesse {
    switch (this) {
      case TypeElement.feu:
        return TypeElement.eau;
      case TypeElement.eau:
        return TypeElement.terre;
      case TypeElement.terre:
        return TypeElement.air;
      case TypeElement.air:
        return TypeElement.feu;
    }
  }

  bool estFortContre(TypeElement autre) {
    return autre.faiblesse == this;
  }
}

void main() {
  for (TypeElement t in TypeElement.values) {
    print('${t.libelle} : ${t.degatsBase} dégâts, faible face à ${t.faiblesse.libelle}');
  }

  print(TypeElement.eau.estFortContre(TypeElement.feu));
  print(TypeElement.feu.estFortContre(TypeElement.eau));
}
```

**Résultat :**

```text
Feu : 12 dégâts, faible face à Eau
Eau : 10 dégâts, faible face à Terre
Terre : 8 dégâts, faible face à Air
Air : 9 dégâts, faible face à Feu
true
false
```

Les règles de syntaxe sont strictes. Mémorisez-les :

```text
1. Les valeurs sont listées EN PREMIER, avec leurs arguments.
2. La dernière valeur se termine par un POINT-VIRGULE ( ; ), pas une virgule.
3. Le constructeur doit être `const`.
4. Tous les attributs doivent être `final`.
```

Schéma de la structure :

```text
enum TypeElement {
  feu('Feu', 12),      <- valeurs, séparées par des virgules
  air('Air', 9);       <- POINT-VIRGULE ici

  const TypeElement(this.libelle, this.degatsBase);   <- constructeur const

  final String libelle;      <- attributs final
  final int degatsBase;

  bool estFortContre(...) { ... }    <- méthodes normales
}
```

Un enhanced enum remplace avantageusement une longue série de `if` dispersés dans le code : toute la connaissance sur les éléments tient dans un seul bloc.

---

## 11.19 — Les extensions (`extension ... on`)

Dernier outil du chapitre.

Problème : vous voulez ajouter une méthode à un type que vous n'avez pas écrit, par exemple `int` ou `String`. Vous ne pouvez pas modifier le code source de Dart.

La solution est l'**extension** :

```dart
extension BarreDeVie on int {
  String barre(int max) {
    int pleines = (this * 10) ~/ max;
    int vides = 10 - pleines;
    String pleine = '#' * pleines;
    String vide = '.' * vides;
    return '[$pleine$vide] $this/$max';
  }
}

void main() {
  print(70.barre(100));
  print(35.barre(100));
  print(100.barre(100));
}
```

**Résultat :**

```text
[#######...] 70/100
[###.......] 35/100
[##########] 100/100
```

Lisez la déclaration ainsi :

```text
extension  BarreDeVie  on  int  { ... }
   |            |       |    |
   |            |       |    +-- le type que l'on enrichit
   |            |       +------- mot-clé obligatoire
   |            +--------------- nom libre de l'extension
   +---------------------------- mot-clé
```

À l'intérieur d'une extension, `this` désigne la valeur sur laquelle la méthode est appelée. Dans `70.barre(100)`, `this` vaut `70`.

Une extension peut aussi ajouter des **getters** :

```dart
extension FormatPseudo on String {
  String get enTitre {
    if (isEmpty) {
      return this;
    }
    return this[0].toUpperCase() + substring(1).toLowerCase();
  }

  bool get estPseudoValide => length >= 3 && length <= 12;
}

void main() {
  print('kAEL'.enTitre);
  print('lyra'.enTitre);
  print('ab'.estPseudoValide);
  print('Kael'.estPseudoValide);
}
```

**Résultat :**

```text
Kael
Lyra
false
true
```

On peut également étendre ses propres types, y compris un enum :

```dart
enum TypeElement { feu, eau, terre, air }

extension SymboleElement on TypeElement {
  String get symbole {
    switch (this) {
      case TypeElement.feu:
        return '(F)';
      case TypeElement.eau:
        return '(E)';
      case TypeElement.terre:
        return '(T)';
      case TypeElement.air:
        return '(A)';
    }
  }
}

void main() {
  for (TypeElement t in TypeElement.values) {
    print('${t.symbole} ${t.name}');
  }
}
```

**Résultat :**

```text
(F) feu
(E) eau
(T) terre
(A) air
```

Deux limites à connaître :

1. une extension **ne peut pas ajouter d'attribut** ; elle n'ajoute que des méthodes, des getters et des setters ;
2. si le type possède déjà une méthode du même nom, c'est **la méthode d'origine qui gagne**. L'extension est ignorée.

---

## 11.20 — Composition vs héritage : le bon réflexe

Nous disposons maintenant de beaucoup d'outils. Il reste une question de conception à trancher.

Imaginons qu'un `Personnage` doive posséder un inventaire. Réflexe de débutant :

```dart
class Inventaire {
  List<String> objets = [];
  void ajouter(String o) => objets.add(o);
}

class Personnage extends Inventaire { // MAUVAIS
  String nom;
  Personnage(this.nom);
}
```

Testons la phrase magique :

```text
« Un Personnage EST UN Inventaire » ?
Non. Un Personnage A UN Inventaire.
```

L'héritage est donc le mauvais outil. La bonne approche est la **composition** : mettre l'objet en attribut.

```dart
class Inventaire {
  List<String> objets = [];

  void ajouter(String objet) {
    objets.add(objet);
    print('Ajouté : $objet');
  }

  int get taille => objets.length;
}

class Personnage {
  String nom;
  Inventaire sac = Inventaire(); // composition

  Personnage(this.nom);

  void resumer() {
    print('$nom transporte ${sac.taille} objet(s) : ${sac.objets}');
  }
}

void main() {
  Personnage kael = Personnage('Kael');

  kael.sac.ajouter('Potion');
  kael.sac.ajouter('Épée courte');

  kael.resumer();
}
```

**Résultat :**

```text
Ajouté : Potion
Ajouté : Épée courte
Kael transporte 2 objet(s) : [Potion, Épée courte]
```

Le tableau de décision à garder sous les yeux :

| Question | Réponse | Outil |
| --- | --- | --- |
| « X EST UN Y » | oui | `extends` |
| « X A UN Y » | oui | composition (attribut) |
| « X SAIT FAIRE Y » | oui | `implements` |
| « X SAIT FAIRE Y, et le code est le même partout » | oui | `mixin` + `with` |

Règle de métier, très largement partagée :

```text
Préférez la composition à l'héritage.
L'héritage crée un lien très fort et définitif entre deux classes.
La composition reste souple : on peut remplacer la pièce à tout moment.
```

Un signe qui doit vous alerter : si votre hiérarchie dépasse trois niveaux (`A extends B extends C extends D`), il y a presque toujours une composition ou un mixin qui aurait été plus simple.

---

## 11.21 — Récapitulatif : quel outil pour quel besoin ?

Voici l'arbre de décision complet du chapitre.

```text
                Je veux réutiliser ou imposer quelque chose
                                |
        +-----------------------+------------------------+
        |                       |                        |
   Je veux du CODE         Je veux un CONTRAT      Je veux une LISTE
   partagé                 sans code                FERMÉE de valeurs
        |                       |                        |
        |                       |                     enum / enhanced enum
        |                       |
   +----+----------+       implements
   |               |       (autant qu'on veut)
« EST UN »   « comportement
   |          transversal »
extends           |
(1 seul)        mixin + with
                (autant qu'on veut)


                Je veux ajouter une méthode à un type existant
                que je ne peux pas modifier
                                |
                          extension ... on
```

Exemple final qui combine tout, sans rien inventer de nouveau :

```dart
enum Rarete { commun, rare, epique }

abstract class Objet {
  String nom;
  Rarete rarete;

  Objet(this.nom, this.rarete);

  void utiliser();

  void decrire() {
    print('$nom [${rarete.name}]');
  }
}

mixin Consommable {
  int utilisations = 1;

  bool consommer() {
    if (utilisations <= 0) {
      return false;
    }
    utilisations = utilisations - 1;
    return true;
  }
}

class Potion extends Objet with Consommable {
  int soin;

  Potion(String nom, Rarete rarete, this.soin) : super(nom, rarete);

  @override
  void utiliser() {
    if (consommer()) {
      print('$nom rend $soin PV.');
    } else {
      print('$nom est vide.');
    }
  }
}

extension AffichageRarete on Rarete {
  String get etoiles => '*' * (index + 1);
}

void main() {
  Potion p = Potion('Élixir', Rarete.epique, 120);

  p.decrire();
  print(p.rarete.etoiles);
  p.utiliser();
  p.utiliser();
}
```

**Résultat :**

```text
Élixir [epique]
***
Élixir rend 120 PV.
Élixir est vide.
```

Dans ce seul programme :

- `Objet` est une classe **abstraite** avec une méthode abstraite et une méthode concrète ;
- `Consommable` est un **mixin** greffé avec `with` ;
- `Rarete` est un **enum** ;
- `AffichageRarete` est une **extension** sur cet enum.

---

## 11.22 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `The class 'Personnage' is abstract and can't be instantiated.` | On appelle le constructeur d'une classe abstraite. | Instancier une sous-classe concrète : `Personnage p = Mage('Lyra');`. |
| `Missing concrete implementation of 'Personnage.attaquer'.` | Une sous-classe concrète n'a pas écrit une méthode abstraite héritée. | Écrire la méthode dans la sous-classe, avec `@override`. |
| `Missing concrete implementations of 'Arme.nom' and 'Arme.nom='.` | On a utilisé `implements` et oublié un attribut du contrat. | Redéclarer **tous** les membres, attributs compris. |
| `Superclass has no method 'frapper'.` après un `implements` | On croit hériter du code alors que `implements` ne transporte rien. | Utiliser `extends` si l'on veut réellement récupérer le code. |
| `Classes can only extend one other class.` | On a écrit `extends A, B`. | Garder un seul `extends` et transformer le reste en mixins ou en interfaces. |
| Une méthode d'un mixin est ignorée sans erreur | Deux mixins définissent la même méthode ; le dernier écrase le précédent. | Renommer la méthode, ou ordonner volontairement `with A, B` en sachant que B gagne. |
| `The class 'Coffre' can't be used as a mixin because it declares a constructor.` | On tente d'utiliser comme mixin une classe qui a un constructeur. | Déclarer un vrai `mixin` (sans constructeur) plutôt qu'une `class`. |
| `'Regeneration' can't be mixed onto 'Coffre'` | Le mixin est déclaré `on Personnage` et la classe cible n'en est pas une. | Faire hériter la classe de `Personnage`, ou retirer la clause `on`. |
| `Undefined name 'feu'.` | On écrit `feu` au lieu de `TypeElement.feu`. | Toujours préfixer une valeur d'enum par le nom de l'enum. |
| `The type 'TypeElement' is not exhaustively matched by the switch cases.` | Une valeur a été ajoutée à l'enum sans mettre le `switch` à jour. | Ajouter le `case` manquant (et non un `default` qui masquerait le problème). |
| Enhanced enum : `Expected ';' after this.` | La liste des valeurs se termine par une virgule au lieu d'un point-virgule. | Terminer la dernière valeur par `;` avant le constructeur. |
| Enhanced enum : `Constructor must be const.` | Le constructeur de l'enum n'est pas `const`. | Écrire `const TypeElement(this.libelle);` et rendre les attributs `final`. |
| Une extension semble sans effet | Le type possède déjà une méthode du même nom, qui a la priorité. | Renommer la méthode de l'extension. |
| `Extensions can't declare instance fields.` | On a déclaré un attribut dans une `extension`. | N'y mettre que des méthodes, getters et setters. |
| Hiérarchie de classes ingérable | On a utilisé `extends` là où « A UN » était la bonne relation. | Passer à la composition : mettre l'objet en attribut. |

---

## 11.23 — Résumé du chapitre

| Outil | Mot-clé | À utiliser quand |
| --- | --- | --- |
| Classe abstraite | `abstract class` | Vous voulez un modèle commun que personne ne doit instancier directement. |
| Méthode abstraite | `void f();` | Vous voulez obliger chaque sous-classe à fournir sa propre version. |
| Héritage | `extends` | La relation est « X EST UN Y » et vous voulez récupérer le code du parent. |
| Interface implicite | `implements` | Vous voulez imposer un contrat « X SAIT FAIRE Y », sans partager de code, et sur plusieurs types à la fois. |
| Mixin | `mixin` + `with` | Vous voulez greffer le même bloc de code sur des classes qui n'ont pas de parent commun. |
| Mixin restreint | `mixin M on C` | Votre mixin a besoin des attributs ou des méthodes de la classe cible. |
| Enum | `enum` | La donnée ne peut prendre qu'un petit nombre de valeurs connues à l'avance. |
| Enhanced enum | `enum` + `const` + `final` | Chaque valeur de l'enum doit transporter des données ou du comportement. |
| Parcours d'enum | `values`, `index`, `name` | Vous devez lister, numéroter ou afficher les valeurs d'un enum. |
| Extension | `extension ... on` | Vous voulez ajouter une méthode à un type que vous ne pouvez pas modifier. |
| Composition | attribut simple | La relation est « X A UN Y ». C'est le cas le plus fréquent. |

---

## 11.24 — Exercices

Faites chaque exercice dans DartPad, sans regarder la correction. Les énoncés sont progressifs : l'exercice 12 réutilise presque tout le chapitre.

---

### Exercice 1 — La classe abstraite `Personnage` (facile)

Écrivez une classe abstraite `Personnage` qui possède :

- deux attributs : `nom` (`String`) et `pv` (`int`) ;
- un constructeur qui reçoit ces deux valeurs ;
- une méthode **concrète** `seDecrire()` qui affiche `nom` et `pv` ;
- une méthode **abstraite** `attaquer()`.

Créez ensuite deux classes concrètes `Guerrier` et `Mage` qui héritent de `Personnage` et qui écrivent chacune leur propre `attaquer()`.

Dans `main()`, rangez un `Guerrier` et un `Mage` dans une `List<Personnage>`, puis parcourez la liste en appelant `seDecrire()` et `attaquer()` sur chaque élément.

**Deuxième partie :** essayez d'écrire `Personnage p = Personnage('Anonyme', 50);`. Notez le message d'erreur, puis mettez la ligne en commentaire.

---

### Exercice 2 — Méthodes concrètes et abstraites mélangées (facile)

Écrivez une classe abstraite `Arme` avec :

- les attributs `nom` (`String`) et `degats` (`int`) ;
- une méthode concrète `decrire()` qui affiche `nom` et `degats` ;
- une méthode concrète `frapper()` qui affiche le nom de l'arme puis appelle la méthode abstraite `bruit()` ;
- une méthode abstraite `bruit()`.

Créez `Epee` et `Arc`. Chacune définit son propre `bruit()`.

Dans `main()`, créez une arme de chaque type et appelez `decrire()` puis `frapper()`.

L'objectif est de bien voir qu'une méthode concrète du parent peut appeler une méthode que le parent ne connaît pas encore.

---

### Exercice 3 — Le contrat `Sauvegardable` (facile)

Écrivez une classe abstraite `Sauvegardable` qui déclare une seule méthode : `String versTexte();`.

Créez ensuite deux classes **qui n'ont aucun lien de parenté entre elles** :

- `Joueur`, avec `nom` et `niveau` ;
- `Coffre`, avec `or`.

Les deux `implements Sauvegardable` et retournent une ligne de texte de leur choix.

Dans `main()`, rangez-les dans une `List<Sauvegardable>` et affichez le résultat de `versTexte()` pour chacune.

---

### Exercice 4 — `extends` contre `implements` (moyen)

Écrivez une classe **concrète** `Arme` avec `nom`, `degats` et une méthode `frapper()` qui affiche une phrase.

Créez ensuite :

- `Epee` qui **hérite** de `Arme` (`extends`) et ne réécrit pas `frapper()` ;
- `Sortilege` qui **implémente** `Arme` (`implements`) et doit donc tout redéclarer, attributs compris.

Dans `main()`, appelez `frapper()` sur les deux objets, puis expliquez en commentaire pourquoi `Epee` n'a rien eu à écrire alors que `Sortilege` a tout dû réécrire.

---

### Exercice 5 — Plusieurs interfaces sur une même classe (moyen)

Déclarez deux contrats :

- `Sauvegardable` avec `String versTexte();` ;
- `Affichable` avec `void afficher();`.

Écrivez une classe `Ennemi` avec `nom` et `pv`, puis une classe `Boss` qui **hérite** de `Ennemi` **et** implémente les deux contrats à la fois.

Dans `main()`, créez un `Boss` et rangez-le successivement dans une variable de type `Ennemi`, puis `Sauvegardable`, puis `Affichable`, pour vérifier qu'il est bien les trois à la fois.

---

### Exercice 6 — Votre premier mixin (moyen)

Écrivez un mixin `Regenerant` qui possède :

- un attribut `pv` initialisé à `100` ;
- un attribut `regeneration` initialisé à `10` ;
- une méthode `regenerer()` qui ajoute `regeneration` à `pv` et affiche le nouveau total.

Greffez ce mixin sur deux classes qui n'ont **aucun parent commun** : `Troll` et `ArbreAncien`.

Dans `main()`, faites régénérer chacune deux fois.

---

### Exercice 7 — Ordre de résolution des mixins (moyen)

Déclarez deux mixins qui possèdent **la même méthode** `crier()` :

- `Bruyant`, qui affiche un cri fort ;
- `Discret`, qui affiche un cri faible.

Créez deux classes :

- `Ogre` déclarée `with Bruyant, Discret` ;
- `Voleur` déclarée `with Discret, Bruyant`.

Appelez `crier()` sur les deux, observez la différence et expliquez-la en commentaire.

---

### Exercice 8 — Restreindre un mixin avec `on` (moyen)

Écrivez une classe `Personnage` avec `nom`, `pv` et `pvMax`.

Écrivez un mixin `Soignable on Personnage` avec une méthode `soigner(int montant)` qui :

- augmente `pv` de `montant` ;
- plafonne `pv` à `pvMax` ;
- affiche `nom` et les PV au format `pv/pvMax`.

Créez `Paladin extends Personnage with Soignable` démarrant à 40 PV sur 100.

Dans `main()`, soignez-le de 30 puis de 50 et vérifiez que le plafond fonctionne.

Ajoutez ensuite, **en commentaire**, une classe `Coffre with Soignable` et notez l'erreur obtenue.

---

### Exercice 9 — Enum et `switch` (moyen)

Déclarez un enum `Difficulte` avec quatre valeurs : `facile`, `normal`, `difficile`, `cauchemar`.

Écrivez une fonction `int multiplicateurDeDegats(Difficulte d)` qui retourne respectivement `1`, `2`, `3` et `5` grâce à un `switch` **sans `default`**.

Écrivez aussi une fonction `String messageAccueil(Difficulte d)` qui retourne une phrase différente pour chaque valeur.

Dans `main()`, parcourez `Difficulte.values` et affichez pour chaque valeur son nom, son multiplicateur et son message.

---

### Exercice 10 — `values`, `index` et `name` (moyen)

Déclarez un enum `TypeElement` avec cinq valeurs : `feu`, `eau`, `terre`, `air`, `foudre`.

Dans `main()` :

1. affichez le nombre total d'éléments ;
2. affichez chaque élément sous la forme `index - name` ;
3. récupérez l'élément d'index `2` et affichez son nom ;
4. affichez le nom du dernier élément de la liste ;
5. construisez et affichez la liste de tous les noms séparés par des virgules.

---

### Exercice 11 — Enhanced enum et extension (difficile)

Déclarez un enhanced enum `TypeArme` avec trois valeurs : `epee`, `arc`, `baton`.

Chaque valeur transporte :

- un `libelle` (`String`) ;
- des `degats` (`int`).

L'enum possède aussi une méthode `degatsCritiques()` qui retourne le double des dégâts.

Ajoutez ensuite une `extension DescriptionArme on TypeArme` qui fournit :

- un getter `fiche` retournant une ligne de description complète ;
- un getter `estPuissante` retournant `true` si les dégâts dépassent 15.

Dans `main()`, parcourez `TypeArme.values` et affichez la fiche de chaque arme ainsi que sa puissance.

---

### Exercice 12 — Mini-projet : le système de compétences (projet)

C'est l'exercice de synthèse du chapitre. Vous allez construire un bestiaire complet.

**Ce qu'il faut écrire :**

1. Un **enhanced enum** `TypeElement` avec quatre valeurs — `feu`, `eau`, `air`, `terre` — chacune transportant un `libelle` (`String`) et un `bonusDegats` (`int`).
2. Trois **mixins**, chacun représentant une brique de comportement :
   - `Volant`, avec un attribut `altitude` et une méthode `voler()` ;
   - `Nageur`, avec un attribut `profondeur` et une méthode `nager()` ;
   - `Invisible`, avec un attribut `visible` et une méthode `disparaitre()`.
3. Une **classe abstraite** `Creature` avec :
   - les attributs `nom`, `pv` et `element` ;
   - une méthode **abstraite** `degatsDeBase()` ;
   - une méthode concrète `degats()` qui additionne `degatsDeBase()` et le bonus de l'élément ;
   - une méthode concrète `seDecrire()` qui affiche la fiche **et la liste des compétences détectées** avec `is` ;
   - une méthode concrète `utiliserCompetences()` qui déclenche chaque compétence présente.
4. Quatre **créatures concrètes** :
   - `Dragon extends Creature with Volant` (élément feu) ;
   - `Requin extends Creature with Nageur` (élément eau) ;
   - `DragonMarin extends Creature with Volant, Nageur, Invisible` (élément air) ;
   - `Golem extends Creature`, sans aucune compétence (élément terre).
5. Une **extension** `FicheElement on TypeElement` qui fournit un getter `etoiles` retournant autant d'astérisques que `index + 1`.

**Ce que doit faire `main()` :**

- ranger les quatre créatures dans une `List<Creature>` ;
- pour chacune : afficher sa fiche, puis déclencher ses compétences ;
- afficher enfin le tableau des éléments avec leur index, leur libellé, leurs étoiles et leur bonus.

**Contrainte :** aucune créature ne doit contenir de code dupliqué. Le code de `voler()` doit être écrit **une seule fois**, dans le mixin.

---

## 11.25 — Corrections des exercices

Chaque correction est un programme complet, à copier tel quel dans DartPad.

---

### Correction 1

```dart
abstract class Personnage {
  String nom;
  int pv;

  Personnage(this.nom, this.pv);

  void seDecrire() {
    print('$nom possède $pv PV.');
  }

  void attaquer();
}

class Guerrier extends Personnage {
  Guerrier(String nom, int pv) : super(nom, pv);

  @override
  void attaquer() {
    print('$nom donne un coup d\'épée.');
  }
}

class Mage extends Personnage {
  Mage(String nom, int pv) : super(nom, pv);

  @override
  void attaquer() {
    print('$nom lance une boule de feu.');
  }
}

void main() {
  List<Personnage> equipe = [
    Guerrier('Bran', 120),
    Mage('Lyra', 80),
  ];

  for (Personnage p in equipe) {
    p.seDecrire();
    p.attaquer();
  }

  // Personnage p = Personnage('Anonyme', 50);
  // Erreur : The class 'Personnage' is abstract and can't be instantiated.
}
```

**Résultat :**

```text
Bran possède 120 PV.
Bran donne un coup d'épée.
Lyra possède 80 PV.
Lyra lance une boule de feu.
```

**Explication :** `Personnage` est déclarée `abstract` parce qu'un « personnage tout court » n'existe pas dans le jeu. Elle contient une méthode concrète (`seDecrire()`, écrite une seule fois et partagée) et une méthode abstraite (`attaquer()`, terminée par un point-virgule, sans accolades). Chaque sous-classe est obligée d'écrire `attaquer()` : si vous supprimez la méthode dans `Mage`, Dart refuse de compiler avec `Missing concrete implementation of 'Personnage.attaquer'`. La ligne commentée à la fin rappelle qu'une classe abstraite ne s'instancie jamais directement. Enfin, la `List<Personnage>` fonctionne parce qu'un `Guerrier` **est un** `Personnage` : c'est le polymorphisme du chapitre 10, appliqué à un type abstrait.

---

### Correction 2

```dart
abstract class Arme {
  String nom;
  int degats;

  Arme(this.nom, this.degats);

  void decrire() {
    print('$nom : $degats dégâts.');
  }

  void frapper() {
    print('$nom frappe.');
    bruit();
  }

  void bruit();
}

class Epee extends Arme {
  Epee() : super('Épée longue', 18);

  @override
  void bruit() {
    print('  CLING !');
  }
}

class Arc extends Arme {
  Arc() : super('Arc court', 12);

  @override
  void bruit() {
    print('  TWANG !');
  }
}

void main() {
  List<Arme> armes = [Epee(), Arc()];

  for (Arme a in armes) {
    a.decrire();
    a.frapper();
  }
}
```

**Résultat :**

```text
Épée longue : 18 dégâts.
Épée longue frappe.
  CLING !
Arc court : 12 dégâts.
Arc court frappe.
  TWANG !
```

**Explication :** le point important est la méthode `frapper()`. Elle est **concrète** : son code est écrit dans la classe abstraite. Pourtant elle appelle `bruit()`, une méthode dont le corps n'existe nulle part dans `Arme`. C'est légal parce que Dart sait qu'aucun objet `Arme` ne sera jamais créé directement : toute instance réelle sera une `Epee` ou un `Arc`, donc `bruit()` existera forcément au moment de l'exécution. C'est le mécanisme le plus utile des classes abstraites : le parent écrit le **squelette** de l'algorithme, l'enfant remplit les **trous**.

---

### Correction 3

```dart
abstract class Sauvegardable {
  String versTexte();
}

class Joueur implements Sauvegardable {
  String nom;
  int niveau;

  Joueur(this.nom, this.niveau);

  @override
  String versTexte() {
    return 'JOUEUR;$nom;$niveau';
  }
}

class Coffre implements Sauvegardable {
  int or;

  Coffre(this.or);

  @override
  String versTexte() {
    return 'COFFRE;$or';
  }
}

void main() {
  List<Sauvegardable> aSauver = [
    Joueur('Alex', 7),
    Coffre(250),
  ];

  print('--- Fichier de sauvegarde ---');
  for (Sauvegardable s in aSauver) {
    print(s.versTexte());
  }
  print('${aSauver.length} élément(s) sauvegardé(s).');
}
```

**Résultat :**

```text
--- Fichier de sauvegarde ---
JOUEUR;Alex;7
COFFRE;250
2 élément(s) sauvegardé(s).
```

**Explication :** `Joueur` et `Coffre` n'ont strictement rien en commun : l'un a un nom et un niveau, l'autre une quantité d'or. Aucun héritage n'aurait de sens ici. En revanche, les deux **savent faire** la même chose : se transformer en texte. C'est exactement le rôle d'un contrat. Grâce à `implements Sauvegardable`, les deux objets peuvent cohabiter dans une même `List<Sauvegardable>`, et la boucle appelle `versTexte()` sans savoir sur quel type elle travaille. Notez que `Sauvegardable` ne fournit aucun code : chaque classe écrit sa propre version.

---

### Correction 4

```dart
class Arme {
  String nom = 'Arme générique';
  int degats = 5;

  void frapper() {
    print('$nom inflige $degats dégâts.');
  }
}

class Epee extends Arme {
  Epee() {
    nom = 'Épée longue';
    degats = 18;
  }
}

class Sortilege implements Arme {
  @override
  String nom = 'Boule de feu';

  @override
  int degats = 30;

  @override
  void frapper() {
    print('$nom brûle la cible pour $degats dégâts.');
  }
}

void main() {
  List<Arme> armes = [Epee(), Sortilege()];

  for (Arme a in armes) {
    a.frapper();
  }

  // Epee     : extends -> a HÉRITÉ de nom, degats et frapper().
  //            Elle n'a réécrit que les valeurs, pas le code.
  // Sortilege: implements -> n'a RIEN hérité.
  //            Elle a dû redéclarer nom, degats ET frapper().
}
```

**Résultat :**

```text
Épée longue inflige 18 dégâts.
Boule de feu brûle la cible pour 30 dégâts.
```

**Explication :** les deux classes sont utilisables comme des `Arme`, mais pour deux raisons différentes. `Epee` **est une** `Arme` : elle récupère le code du parent, y compris le corps de `frapper()`, et se contente de changer deux valeurs dans son constructeur. `Sortilege` n'est pas une `Arme`, elle **respecte la forme** d'une `Arme` : Dart lui impose de fournir tous les membres publics du type, attributs compris. Si vous retirez la ligne `int degats = 30;`, l'erreur est immédiate : `Missing concrete implementations of 'Arme.degats' and 'Arme.degats='`. Retenez la règle : `extends` transporte le code, `implements` ne transporte que la liste des obligations.

---

### Correction 5

```dart
abstract class Sauvegardable {
  String versTexte();
}

abstract class Affichable {
  void afficher();
}

class Ennemi {
  String nom;
  int pv;

  Ennemi(this.nom, this.pv);

  void rugir() {
    print('$nom rugit.');
  }
}

class Boss extends Ennemi implements Sauvegardable, Affichable {
  int phase;

  Boss(String nom, int pv, this.phase) : super(nom, pv);

  @override
  String versTexte() {
    return 'BOSS;$nom;$pv;phase=$phase';
  }

  @override
  void afficher() {
    print('=== $nom === $pv PV (phase $phase)');
  }
}

void main() {
  Boss boss = Boss('Malakar', 900, 2);

  Ennemi commeEnnemi = boss;
  Sauvegardable commeSauvegardable = boss;
  Affichable commeAffichable = boss;

  commeEnnemi.rugir();
  print(commeSauvegardable.versTexte());
  commeAffichable.afficher();
}
```

**Résultat :**

```text
Malakar rugit.
BOSS;Malakar;900;phase=2
=== Malakar === 900 PV (phase 2)
```

**Explication :** un même objet possède ici **trois identités** simultanées. Il est un `Ennemi` par héritage, donc il récupère `nom`, `pv` et le code de `rugir()`. Il est un `Sauvegardable` et un `Affichable` par contrat, donc il a dû écrire lui-même `versTexte()` et `afficher()`. L'ordre d'écriture est imposé par Dart : `extends` d'abord, `implements` ensuite, et un seul `extends` est autorisé alors que `implements` accepte autant de types que vous voulez, séparés par des virgules. Les trois variables `commeEnnemi`, `commeSauvegardable` et `commeAffichable` pointent vers **le même objet en mémoire** : seul le type déclaré change, et donc seule la liste des méthodes accessibles change.

---

### Correction 6

```dart
mixin Regenerant {
  int pv = 100;
  int regeneration = 10;

  void regenerer() {
    pv = pv + regeneration;
    print('Régénération : +$regeneration PV (total $pv).');
  }
}

class Troll with Regenerant {
  String nom;

  Troll(this.nom);
}

class ArbreAncien with Regenerant {
  int age;

  ArbreAncien(this.age);
}

void main() {
  Troll troll = Troll('Grumf');
  ArbreAncien arbre = ArbreAncien(800);

  print('Troll ${troll.nom} :');
  troll.regenerer();
  troll.regenerer();

  print('Arbre de ${arbre.age} ans :');
  arbre.regenerer();
  arbre.regenerer();
}
```

**Résultat :**

```text
Troll Grumf :
Régénération : +10 PV (total 110).
Régénération : +10 PV (total 120).
Arbre de 800 ans :
Régénération : +10 PV (total 110).
Régénération : +10 PV (total 120).
```

**Explication :** un troll et un arbre n'ont aucun parent commun, et il serait absurde d'en inventer un juste pour partager la régénération. Le mixin résout ce problème : il contient du **vrai code** (l'attribut `regeneration`, l'attribut `pv` et le corps de `regenerer()`), et ce code est greffé sur n'importe quelle classe avec `with`. Remarquez que `class Troll with Regenerant` s'écrit sans `extends` : la classe hérite alors implicitement d'`Object`, ce qui est parfaitement valide. Remarquez aussi que chaque objet possède **sa propre copie** des attributs du mixin : les deux compteurs de PV évoluent indépendamment.

---

### Correction 7

```dart
mixin Bruyant {
  void crier() {
    print('AAAAAH ! (cri puissant)');
  }
}

mixin Discret {
  void crier() {
    print('... (murmure)');
  }
}

class Ogre with Bruyant, Discret {
  String nom;

  Ogre(this.nom);
}

class Voleur with Discret, Bruyant {
  String nom;

  Voleur(this.nom);
}

void main() {
  Ogre ogre = Ogre('Krag');
  Voleur voleur = Voleur('Nyx');

  print('Ogre ${ogre.nom} :');
  ogre.crier();

  print('Voleur ${voleur.nom} :');
  voleur.crier();

  // Règle : le DERNIER mixin écrit gagne.
  // Ogre  = with Bruyant, Discret -> Discret l'emporte.
  // Voleur = with Discret, Bruyant -> Bruyant l'emporte.
}
```

**Résultat :**

```text
Ogre Krag :
... (murmure)
Voleur Nyx :
AAAAAH ! (cri puissant)
```

**Explication :** les deux classes utilisent exactement les mêmes mixins, et pourtant elles ne se comportent pas de la même façon. Quand plusieurs mixins définissent la même méthode, Dart les empile de gauche à droite : chaque mixin recouvre le précédent, donc **le dernier écrit gagne**. C'est le résultat le plus déroutant du chapitre, car aucune erreur n'est signalée : le programme compile et affiche silencieusement le mauvais comportement. Dans un vrai projet, évitez les collisions de noms entre mixins ; si vous en provoquez une volontairement, écrivez un commentaire pour justifier l'ordre.

---

### Correction 8

```dart
class Personnage {
  String nom;
  int pv;
  int pvMax;

  Personnage(this.nom, this.pv, this.pvMax);
}

mixin Soignable on Personnage {
  void soigner(int montant) {
    pv = pv + montant;
    if (pv > pvMax) {
      pv = pvMax;
    }
    print('$nom est soigné : $pv/$pvMax PV.');
  }
}

class Paladin extends Personnage with Soignable {
  Paladin(String nom) : super(nom, 40, 100);
}

// class Coffre with Soignable {}
// Erreur : 'Soignable' can't be mixed onto 'Object' because 'Object'
// doesn't implement 'Personnage'.

void main() {
  Paladin paladin = Paladin('Ael');

  paladin.soigner(30);
  paladin.soigner(50);
}
```

**Résultat :**

```text
Ael est soigné : 70/100 PV.
Ael est soigné : 100/100 PV.
```

**Explication :** ce mixin ne déclare aucun attribut. Il utilise `nom`, `pv` et `pvMax`, qui appartiennent à la classe cible. C'est précisément ce que rend possible la clause `on Personnage` : elle dit à Dart « ce mixin ne pourra être greffé que sur un `Personnage` ou une de ses sous-classes, donc je peux compter sur ses membres ». Sans cette clause, les trois lignes du mixin refuseraient de compiler avec `Undefined name 'pv'`. Le deuxième appel démontre le plafonnement : 70 + 50 = 120, ramené à `pvMax`, soit 100. La classe `Coffre` en commentaire montre la protection en action : `on` est aussi une garantie de sécurité.

---

### Correction 9

```dart
enum Difficulte { facile, normal, difficile, cauchemar }

int multiplicateurDeDegats(Difficulte d) {
  switch (d) {
    case Difficulte.facile:
      return 1;
    case Difficulte.normal:
      return 2;
    case Difficulte.difficile:
      return 3;
    case Difficulte.cauchemar:
      return 5;
  }
}

String messageAccueil(Difficulte d) {
  switch (d) {
    case Difficulte.facile:
      return 'Promenade de santé.';
    case Difficulte.normal:
      return 'Aventure équilibrée.';
    case Difficulte.difficile:
      return 'Préparez vos potions.';
    case Difficulte.cauchemar:
      return 'Bonne chance.';
  }
}

void main() {
  for (Difficulte d in Difficulte.values) {
    print('${d.name} -> x${multiplicateurDeDegats(d)} | ${messageAccueil(d)}');
  }
}
```

**Résultat :**

```text
facile -> x1 | Promenade de santé.
normal -> x2 | Aventure équilibrée.
difficile -> x3 | Préparez vos potions.
cauchemar -> x5 | Bonne chance.
```

**Explication :** l'absence de `default` n'est pas un oubli, c'est le but de l'exercice. Comme un enum contient une liste **fermée** de valeurs, Dart vérifie que le `switch` les traite toutes. Si vous ajoutez plus tard une cinquième difficulté, les deux fonctions cesseront de compiler et vous serez averti de l'endroit exact à mettre à jour. Avec un `default`, le compilateur se tairait et la nouvelle difficulté hériterait silencieusement d'une valeur au hasard. Le `default` sur un enum est donc, la plupart du temps, une mauvaise idée.

---

### Correction 10

```dart
enum TypeElement { feu, eau, terre, air, foudre }

void main() {
  print('Nombre d\'éléments : ${TypeElement.values.length}');

  for (TypeElement e in TypeElement.values) {
    print('${e.index} - ${e.name}');
  }

  TypeElement choisi = TypeElement.values[2];
  print('Élément d\'index 2 : ${choisi.name}');

  print('Dernier élément : ${TypeElement.values.last.name}');

  String liste = '';
  for (TypeElement e in TypeElement.values) {
    if (liste != '') {
      liste = liste + ', ';
    }
    liste = liste + e.name;
  }
  print('Tous : $liste');
}
```

**Résultat :**

```text
Nombre d'éléments : 5
0 - feu
1 - eau
2 - terre
3 - air
4 - foudre
Élément d'index 2 : terre
Dernier élément : foudre
Tous : feu, eau, terre, air, foudre
```

**Explication :** trois outils sont utilisés ici. `values` est une liste constante contenant toutes les valeurs, dans l'ordre exact de la déclaration : elle se parcourt comme n'importe quelle `List`, et accepte `length`, `last` ou l'accès par index. `index` donne la position, en commençant à `0` ; c'est pratique pour du calcul, mais dangereux à sauvegarder, car réordonner les valeurs de l'enum changerait tous les index. `name` retourne le nom écrit dans le code, sous forme de `String` : c'est ce que vous afficherez à l'utilisateur.

---

### Correction 11

```dart
enum TypeArme {
  epee('Épée longue', 18),
  arc('Arc court', 12),
  baton('Bâton de mage', 9);

  final String libelle;
  final int degats;

  const TypeArme(this.libelle, this.degats);

  int degatsCritiques() {
    return degats * 2;
  }
}

extension DescriptionArme on TypeArme {
  String get fiche {
    return '$libelle : $degats dégâts (critique ${degatsCritiques()})';
  }

  bool get estPuissante {
    return degats > 15;
  }
}

void main() {
  for (TypeArme a in TypeArme.values) {
    print(a.fiche);
    if (a.estPuissante) {
      print('  -> arme puissante');
    } else {
      print('  -> arme légère');
    }
  }
}
```

**Résultat :**

```text
Épée longue : 18 dégâts (critique 36)
  -> arme puissante
Arc court : 12 dégâts (critique 24)
  -> arme légère
Bâton de mage : 9 dégâts (critique 18)
  -> arme légère
```

**Explication :** trois détails de syntaxe sont obligatoires dans un enhanced enum, et ce sont les trois erreurs classiques. Premièrement, la liste des valeurs se termine par un **point-virgule** (`baton(...);`) et non par une virgule : c'est lui qui sépare les valeurs du reste du corps. Deuxièmement, les attributs sont `final` et le constructeur est `const`, car les valeurs d'un enum sont créées une fois pour toutes à la compilation. Troisièmement, l'`extension` ne peut contenir ni attribut ni constructeur : uniquement des méthodes, des getters et des setters. Ici, `fiche` et `estPuissante` s'utilisent comme s'ils appartenaient à l'enum, alors qu'ils sont écrits à l'extérieur.

---

### Correction 12

```dart
enum TypeElement {
  feu('Feu', 15),
  eau('Eau', 12),
  air('Air', 10),
  terre('Terre', 8);

  final String libelle;
  final int bonusDegats;

  const TypeElement(this.libelle, this.bonusDegats);
}

mixin Volant {
  int altitude = 0;

  void voler() {
    altitude = 120;
    print('  Vol : altitude $altitude m.');
  }
}

mixin Nageur {
  int profondeur = 0;

  void nager() {
    profondeur = 30;
    print('  Nage : profondeur $profondeur m.');
  }
}

mixin Invisible {
  bool visible = true;

  void disparaitre() {
    visible = false;
    print('  Invisibilité activée.');
  }
}

abstract class Creature {
  String nom;
  int pv;
  TypeElement element;

  Creature(this.nom, this.pv, this.element);

  int degatsDeBase();

  int degats() {
    return degatsDeBase() + element.bonusDegats;
  }

  void seDecrire() {
    print('$nom [${element.libelle}] - $pv PV - ${degats()} dégâts');
    if (this is Volant) {
      print('  Compétence : vol');
    }
    if (this is Nageur) {
      print('  Compétence : nage');
    }
    if (this is Invisible) {
      print('  Compétence : invisibilité');
    }
  }

  void utiliserCompetences() {
    if (this is Volant) {
      (this as Volant).voler();
    }
    if (this is Nageur) {
      (this as Nageur).nager();
    }
    if (this is Invisible) {
      (this as Invisible).disparaitre();
    }
  }
}

class Dragon extends Creature with Volant {
  Dragon(String nom) : super(nom, 300, TypeElement.feu);

  @override
  int degatsDeBase() {
    return 40;
  }
}

class Requin extends Creature with Nageur {
  Requin(String nom) : super(nom, 150, TypeElement.eau);

  @override
  int degatsDeBase() {
    return 25;
  }
}

class DragonMarin extends Creature with Volant, Nageur, Invisible {
  DragonMarin(String nom) : super(nom, 400, TypeElement.air);

  @override
  int degatsDeBase() {
    return 55;
  }
}

class Golem extends Creature {
  Golem(String nom) : super(nom, 500, TypeElement.terre);

  @override
  int degatsDeBase() {
    return 30;
  }
}

extension FicheElement on TypeElement {
  String get etoiles {
    return '*' * (index + 1);
  }
}

void main() {
  List<Creature> bestiaire = [
    Dragon('Ignis'),
    Requin('Abyss'),
    DragonMarin('Léviathan'),
    Golem('Roc'),
  ];

  print('=== BESTIAIRE ===');
  for (Creature c in bestiaire) {
    c.seDecrire();
    c.utiliserCompetences();
    print('');
  }

  print('=== ELEMENTS ===');
  for (TypeElement e in TypeElement.values) {
    print('${e.index} - ${e.libelle} ${e.etoiles} (+${e.bonusDegats})');
  }
}
```

**Résultat :**

```text
=== BESTIAIRE ===
Ignis [Feu] - 300 PV - 55 dégâts
  Compétence : vol
  Vol : altitude 120 m.

Abyss [Eau] - 150 PV - 37 dégâts
  Compétence : nage
  Nage : profondeur 30 m.

Léviathan [Air] - 400 PV - 65 dégâts
  Compétence : vol
  Compétence : nage
  Compétence : invisibilité
  Vol : altitude 120 m.
  Nage : profondeur 30 m.
  Invisibilité activée.

Roc [Terre] - 500 PV - 38 dégâts

=== ELEMENTS ===
0 - Feu * (+15)
1 - Eau ** (+12)
2 - Air *** (+10)
3 - Terre **** (+8)
```

**Explication :** ce programme réunit les cinq outils du chapitre, et chacun y joue le rôle pour lequel il est fait.

- `TypeElement` est un **enhanced enum**. Chaque élément transporte son libellé d'affichage et son bonus de dégâts. Le calcul `degatsDeBase() + element.bonusDegats` donne 40 + 15 = 55 pour le dragon, 25 + 12 = 37 pour le requin, 55 + 10 = 65 pour le dragon marin, 30 + 8 = 38 pour le golem.
- `Creature` est une **classe abstraite**. Elle est le socle commun : nom, PV, élément, plus tout le code d'affichage. Sa méthode `degatsDeBase()` est abstraite, donc chaque créature est obligée de fixer sa propre puissance, et personne ne peut écrire `Creature('X', 10, TypeElement.feu)`.
- `Volant`, `Nageur` et `Invisible` sont des **mixins**. C'est ici que se joue la contrainte de l'énoncé : le code de `voler()` est écrit **une seule fois**, et pourtant `Dragon` et `DragonMarin` en profitent tous les deux. Avec `implements`, il aurait fallu le recopier dans chaque classe. Avec `extends`, `DragonMarin` n'aurait pas pu cumuler trois comportements.
- Le test `this is Volant` fonctionne parce qu'un mixin définit aussi un **type**. Une créature greffée avec `with Volant` est bien un `Volant`. Le `Golem`, greffé avec rien du tout, ne déclenche aucune compétence : c'est pour cela que sa fiche n'affiche que deux lignes.
- `FicheElement` est une **extension** sur l'enum. Elle ajoute le getter `etoiles` sans modifier une seule ligne de `TypeElement`. L'expression `'*' * (index + 1)` répète le caractère autant de fois que demandé : `feu` a l'index 0, donc une étoile ; `terre` a l'index 3, donc quatre étoiles.

Reprenez ce programme et ajoutez-y une créature de votre choix, par exemple un `Fantome extends Creature with Invisible`. Vous constaterez que vous n'avez qu'une chose à écrire : sa méthode `degatsDeBase()`. Tout le reste est déjà en place. C'est exactement le bénéfice recherché par la POO avancée.

---

## Et maintenant ?

Vous disposez à présent de la boîte à outils complète de la conception orientée objet en Dart.

Vous savez décrire un modèle incomplet avec `abstract`, imposer un contrat avec `implements`, partager du code transversal avec un `mixin`, fermer un ensemble de valeurs avec un `enum`, enrichir un type existant avec une `extension`, et surtout choisir entre héritage et composition.

Il reste pourtant une question que nous avons soigneusement contournée depuis le début de la formation : que se passe-t-il lorsqu'une valeur **n'existe pas** ?

Un joueur qui n'a pas encore d'arme équipée. Un coffre vide. Un boss qui n'a pas de deuxième phase. Un nom qui n'a jamais été saisi.

En Dart, cette absence porte un nom : `null`. Et Dart possède un système, unique parmi les langages courants par sa rigueur, qui vous empêche d'oublier ce cas : le **null safety**. C'est lui qui explique le point d'interrogation que vous avez peut-être déjà aperçu dans des exemples de code Flutter :

```dart
String? armeEquipee;
```

Le prochain chapitre lui est entièrement consacré. Vous y apprendrez les types nullables, les opérateurs `?`, `!`, `??`, `?.` et `??=`, le mot-clé `late`, et la manière de ne plus jamais rencontrer l'erreur la plus célèbre de la programmation : la fameuse « null pointer exception ».

Chapitre suivant : [12-PARTIE-1A—LE-NULL-SAFETY.md](12-PARTIE-1A—LE-NULL-SAFETY.md)
