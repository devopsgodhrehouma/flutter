# PARTIE 1A — DART
# CHAPITRE 12 — LE NULL SAFETY

> **Niveau :** débutant / intermédiaire
> **Durée estimée :** 5 h
> **Pré-requis :** chapitre 11 — POO avancée (`abstract`, `mixins`, `enums`)
> **Ce que vous saurez faire à la fin :** écrire du code Dart dans lequel une valeur absente ne peut plus faire planter le programme, en distinguant les types nullables des types non nullables et en maniant `?.`, `??`, `??=`, `!` et `late` à bon escient.

---

## 12.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer ce qu'est `null` et pourquoi il a coûté si cher à l'industrie ;
- distinguer un type non nullable (`String`) d'un type nullable (`String?`) ;
- lire et comprendre les messages d'erreur du compilateur liés au null safety ;
- utiliser l'accès sûr `?.` ;
- utiliser la valeur de repli `??` ;
- utiliser l'affectation conditionnelle `??=` ;
- expliquer pourquoi l'opérateur `!` est un dernier recours ;
- utiliser `late` pour une initialisation différée ;
- utiliser `late final` pour une valeur écrite une seule fois ;
- reconnaître et corriger une `LateInitializationError` ;
- comprendre la promotion de type (flow analysis) ;
- expliquer pourquoi la promotion échoue sur un champ de classe ;
- corriger ce problème avec une variable locale intermédiaire ;
- distinguer `List<int>?` de `List<int?>` ;
- lire une `Map` sans faire planter le programme ;
- écrire des paramètres nullables, des valeurs par défaut et des paramètres `required` ;
- appliquer une checklist de bonnes pratiques sur un vrai projet.

---

## 12.1 — Le problème du « milliard de dollars »

En 1965, l'informaticien Tony Hoare invente la référence `null` pour le langage ALGOL W. Quarante ans plus tard, il monte sur scène et présente ses excuses. Il appelle son invention « my billion-dollar mistake » : **mon erreur à un milliard de dollars**.

Pourquoi une telle somme ?

Parce que `null` a produit, pendant quarante ans, la même famille de bugs dans presque tous les langages : un programme demande une information à une valeur qui n'existe pas, et il s'arrête net.

Prenons notre fil rouge. Un joueur peut avoir une arme équipée, ou pas.

```text
  joueur.arme  ──>  "Épée de feu"       le joueur a une arme
  joueur.arme  ──>  null                le joueur n'a PAS d'arme
```

Le programme affiche la puissance de l'arme :

```text
  puissance = joueur.arme.puissance
```

Si l'arme existe, tout va bien. Si l'arme vaut `null`, il n'y a **rien** dont on puisse demander la puissance. Le programme s'arrête.

Le vrai problème n'est pas le plantage. Le vrai problème est **le moment** du plantage :

```text
  ┌──────────────────────────────────────────────────────────┐
  │  Sans null safety                                        │
  ├──────────────────────────────────────────────────────────┤
  │  Vous écrivez le code ..................... aucune alerte │
  │  Vous compilez ............................ aucune alerte │
  │  Vous testez 30 minutes ................... ça marche     │
  │  Vous publiez le jeu ...................... ça marche     │
  │  Un joueur enlève son arme et ouvre l'inventaire          │
  │      -> CRASH chez le client                             │
  └──────────────────────────────────────────────────────────┘
```

Le bug était présent dès la première minute. Il ne s'est manifesté que six mois plus tard, chez un utilisateur, dans une situation que personne n'avait testée.

Depuis Dart 2.12, Dart possède le **null safety** (« sûreté vis-à-vis de `null` »). L'idée tient en une phrase :

> Déplacer l'erreur du moment de l'exécution vers le moment de l'écriture du code.

Autrement dit :

```text
  ┌──────────────────────────────────────────────────────────┐
  │  Avec null safety                                        │
  ├──────────────────────────────────────────────────────────┤
  │  Vous écrivez le code ..... le compilateur REFUSE tout   │
  │                             de suite et vous explique     │
  │                             pourquoi                      │
  │  -> le bug n'atteint jamais le joueur                    │
  └──────────────────────────────────────────────────────────┘
```

C'est pour cela qu'il faut voir ce chapitre comme une **bonne nouvelle**, et non comme une série d'obstacles. Chaque refus du compilateur est un crash évité chez l'utilisateur.

---

## 12.2 — Qu'est-ce que `null` ?

`null` est une valeur particulière du langage Dart. Elle signifie :

> « il n'y a pas de valeur ici »

Attention, c'est le point que les débutants confondent le plus souvent. `null` n'est ni `0`, ni `''`, ni `false`, ni une liste vide.

| Valeur | Signification |
| --- | --- |
| `0` | un nombre, qui vaut zéro |
| `''` | un texte, qui est vide |
| `false` | un booléen, qui est faux |
| `[]` | une liste, qui ne contient rien |
| `null` | **rien du tout : la boîte est vide** |

Un schéma mémoire rend la différence évidente :

```text
  int score = 0;                String nom = '';

  score ──> ┌─────┐             nom ──> ┌─────────┐
            │  0  │                     │  (vide) │
            └─────┘                     └─────────┘
   il y a bien un nombre         il y a bien un texte


  String? arme = null;

  arme ───> (ne pointe vers rien)

   il n'y a AUCUN objet au bout
```

Voici du code qui montre concrètement la différence :

```dart
void main() {
  int score = 0;
  String nom = '';
  List<String> inventaire = [];
  String? arme = null;

  print('score  : $score');
  print('nom    : "$nom"');
  print('sac    : $inventaire');
  print('arme   : $arme');

  print('nom est vide ? ${nom.isEmpty}');
  print('sac est vide ? ${inventaire.isEmpty}');
  print('arme est nulle ? ${arme == null}');
}
```

**Résultat :**

```text
score  : 0
nom    : ""
sac    : []
arme   : null
nom est vide ? true
sac est vide ? true
arme est nulle ? true
```

Remarquez la ligne `nom.isEmpty`. Elle fonctionne parce que `nom` **est** un texte, même vide : un objet `String` existe, et il sait répondre à `isEmpty`.

Sur `arme`, on ne peut rien demander de tel, puisqu'il n'y a aucun objet. C'est toute la difficulté que le null safety va régler.

> **Remarque de vocabulaire :** on dit « une variable vaut `null` », ou « une variable est nulle ». On ne dit pas « une variable est vide » : « vide » se réserve aux textes et aux collections.

---

## 12.3 — Types non nullables : le défaut en Dart moderne

En Dart moderne, **tous les types sont non nullables par défaut**.

Cela veut dire que si vous écrivez :

```dart
String nomJoueur = 'Alex';
```

vous obtenez une garantie, offerte par le compilateur : `nomJoueur` **contiendra toujours un texte**. Jamais `null`. À aucun moment de la vie du programme.

Cette garantie n'est pas une promesse morale. Elle est vérifiée mécaniquement. Essayez de la violer :

```dart
void main() {
  String nomJoueur = null;
  print(nomJoueur);
}
```

**Résultat :**

```text
Error: The value 'null' can't be assigned to a variable of type 'String'
because 'String' is not nullable.
```

Le code ne compile même pas. Il n'y a pas d'exécution, donc pas de crash possible.

Le même principe s'applique à tous les types :

```dart
void main() {
  int vies = 3;
  double energie = 87.5;
  bool estVivant = true;
  List<String> inventaire = ['potion', 'clé'];

  print('vies      : $vies');
  print('energie   : $energie');
  print('vivant    : $estVivant');
  print('inventaire: $inventaire');
}
```

**Résultat :**

```text
vies      : 3
energie   : 87.5
vivant    : true
inventaire: [potion, clé]
```

Aucune de ces quatre variables ne pourra jamais valoir `null`. Vous pouvez donc appeler leurs méthodes en toute confiance :

```dart
void main() {
  String nomJoueur = 'Alex';
  print(nomJoueur.toUpperCase());
  print(nomJoueur.length);
}
```

**Résultat :**

```text
ALEX
4
```

Retenez la conséquence pratique, elle est énorme :

> Sur une variable non nullable, **aucune vérification n'est nécessaire**. Pas de `if (x != null)`, pas de `?.`, pas de `!`. Le code reste court et lisible.

C'est la raison pour laquelle la règle numéro un du chapitre est :

> **Préférez toujours un type non nullable.** Ne rendez une variable nullable que si l'absence de valeur a un sens réel dans votre jeu.

---

## 12.4 — Types nullables avec `?`

Parfois, l'absence de valeur a un vrai sens métier.

Dans notre jeu :

- un joueur peut n'avoir **aucune arme équipée** ;
- un joueur peut n'avoir **aucune guilde** ;
- un ennemi peut n'avoir **aucun butin** à lâcher ;
- un score en ligne peut être **inconnu** tant que le serveur n'a pas répondu.

Dans ces cas-là, `null` n'est pas un bug : c'est une information. Il faut alors le dire explicitement au compilateur en ajoutant un point d'interrogation après le type.

```dart
void main() {
  String? armeEquipee = null;
  String? guilde;

  print('arme   : $armeEquipee');
  print('guilde : $guilde');
}
```

**Résultat :**

```text
arme   : null
guilde : null
```

Deux points importants dans ce code.

**Premier point :** `String?` se lit « un texte, **ou** rien ». Ce n'est pas le même type que `String`.

```text
  String   ─── contient obligatoirement un texte
  String?  ─── contient un texte OU null
```

**Deuxième point :** la variable `guilde` n'a reçu aucune valeur, et pourtant le programme fonctionne. C'est une particularité des types nullables : une variable nullable non initialisée vaut automatiquement `null`.

Comparez avec un type non nullable :

```dart
void main() {
  String nom;
  print(nom);
}
```

**Résultat :**

```text
Error: Non-nullable variable 'nom' must be assigned before it can be used.
```

Le compilateur refuse : une variable non nullable doit recevoir une valeur avant toute lecture.

Voici un exemple complet, dans le contexte du jeu :

```dart
class Player {
  String name;
  int health;
  String? weapon;
  String? guild;

  Player({
    required this.name,
    this.health = 100,
    this.weapon,
    this.guild,
  });
}

void main() {
  Player alex = Player(name: 'Alex', weapon: 'Épée de feu', guild: 'Les Loups');
  Player novice = Player(name: 'Novice');

  print('${alex.name} | arme: ${alex.weapon} | guilde: ${alex.guild}');
  print('${novice.name} | arme: ${novice.weapon} | guilde: ${novice.guild}');
}
```

**Résultat :**

```text
Alex | arme: Épée de feu | guilde: Les Loups
Novice | arme: null | guilde: null
```

Le modèle est honnête : `name` et `health` existent toujours, `weapon` et `guild` peuvent manquer. Le type raconte la règle du jeu.

> **Remarque :** écrire `String? arme = null;` ou `String? arme;` revient exactement au même. La seconde forme est plus courte, et l'analyseur Dart préfère celle-ci.

---

## 12.5 — Ce que le compilateur refuse, et pourquoi c'est une bonne nouvelle

Nous arrivons au cœur du chapitre.

Dès qu'une variable est nullable, le compilateur **interdit d'utiliser directement ses membres**. Regardez :

```dart
void main() {
  String? arme;
  print(arme.length);
}
```

**Résultat :**

```text
Error: Property 'length' cannot be unconditionally accessed because the
receiver can be 'null'.
Try making the access conditional (using '?.') or adding a null check
to the target ('!').
```

Lisez ce message attentivement, vous allez le revoir souvent. Il dit trois choses :

```text
  "cannot be unconditionally accessed"
       -> vous demandez .length SANS condition

  "because the receiver can be 'null'"
       -> l'objet à gauche du point peut être null

  "Try making the access conditional (using '?.')
   or adding a null check to the target ('!')"
       -> voici deux pistes de correction
```

Le compilateur ne dit pas « votre variable est nulle ». Il dit « votre variable **peut** être nulle, et vous n'avez rien prévu pour ce cas ».

Le même refus se produit sur une méthode :

```dart
void main() {
  String? arme;
  print(arme.toUpperCase());
}
```

**Résultat :**

```text
Error: Method 'toUpperCase' cannot be unconditionally invoked because the
receiver can be 'null'.
```

Et lors d'une affectation d'un nullable vers un non nullable :

```dart
void main() {
  String? arme = 'Épée';
  String armeSure = arme;
  print(armeSure);
}
```

**Résultat :**

```text
Error: A value of type 'String?' can't be assigned to a variable of type
'String'.
```

Ce dernier cas surprend beaucoup : « mais la variable contient bien `'Épée'` ! ». Oui, à cette ligne précise. Mais le compilateur raisonne sur le **type déclaré**, pas sur la valeur du moment. Le type déclaré autorise `null`, donc l'affectation est refusée.

Voyons ce que ces refus nous font gagner. Voici le même programme écrit dans un langage sans null safety, en pseudo-code :

```text
  arme = null
  affiche(arme.longueur)

  -> le code compile
  -> le code démarre
  -> CRASH : NullPointerException
```

Et en Dart moderne :

```text
  String? arme;
  print(arme.length);

  -> le code NE COMPILE PAS
  -> aucun utilisateur n'a jamais vu ce bug
```

Voilà pourquoi il faut accueillir ces messages avec soulagement, et non avec agacement. Le compilateur ne vous embête pas : il fait le travail de test que vous n'auriez pas fait.

Les sections suivantes présentent les outils pour répondre proprement à ces refus. Il y en a quatre principaux :

```text
  ?.    accéder seulement si ce n'est pas null
  ??    donner une valeur de remplacement
  ??=   remplir la case seulement si elle est vide
  !     affirmer "ce n'est pas null" -- DANGEREUX, dernier recours
```

Nous les voyons un par un.

---

## 12.6 — L'accès sûr `?.`

L'opérateur `?.` s'appelle l'**accès conditionnel**, ou **accès sûr**.

Sa règle tient en une phrase :

> `objet?.membre` : si `objet` vaut `null`, l'expression entière vaut `null` et le membre n'est jamais évalué. Sinon, on accède normalement au membre.

Schéma de fonctionnement :

```text
        arme?.length

        arme vaut null ?
          │
          ├── OUI ──> le résultat vaut null
          │           (on ne touche PAS à .length)
          │
          └── NON ──> le résultat vaut arme.length
```

Reprenons le code qui était refusé :

```dart
void main() {
  String? arme;
  print(arme?.length);
}
```

**Résultat :**

```text
null
```

Plus d'erreur de compilation, et aucun crash. Le programme affiche `null`, ce qui est l'information exacte : il n'y a pas d'arme, donc pas de longueur.

Avec une valeur présente :

```dart
void main() {
  String? arme = 'Épée de feu';
  print(arme?.length);
  print(arme?.toUpperCase());
}
```

**Résultat :**

```text
11
ÉPÉE DE FEU
```

Point capital à comprendre : **le résultat de `?.` est toujours nullable**.

```dart
void main() {
  String? arme = 'Épée';
  int? longueur = arme?.length;
  print(longueur);
}
```

**Résultat :**

```text
4
```

Notez le type : `int?`, et non `int`. Le compilateur raisonne ainsi : « si `arme` avait été `null`, `arme?.length` aurait valu `null` ; donc le résultat peut être `null` ». C'est logique, et c'est la source d'une erreur fréquente :

```dart
void main() {
  String? arme = 'Épée';
  int longueur = arme?.length;
  print(longueur);
}
```

**Résultat :**

```text
Error: A value of type 'int?' can't be assigned to a variable of type 'int'.
```

Nous verrons en 12.7 comment régler ce cas proprement avec `??`.

### 12.6.1 — Le chaînage

`?.` peut s'enchaîner. C'est là qu'il devient vraiment utile.

```dart
class Weapon {
  String name;
  int power;
  Weapon(this.name, this.power);
}

class Player {
  String name;
  Weapon? weapon;
  Player(this.name, {this.weapon});
}

void main() {
  Player alex = Player('Alex', weapon: Weapon('Épée de feu', 45));
  Player novice = Player('Novice');

  print(alex.weapon?.name);
  print(alex.weapon?.power);
  print(novice.weapon?.name);
  print(novice.weapon?.power);
}
```

**Résultat :**

```text
Épée de feu
45
null
null
```

Sur `novice`, l'évaluation s'arrête au premier `?.` : dès que `weapon` vaut `null`, tout le reste de la chaîne est abandonné et l'expression vaut `null`. C'est ce qu'on appelle le **court-circuit**.

```text
  novice.weapon?.name
         │        │
         │        └── jamais évalué
         └── null -> court-circuit, résultat = null
```

Une chaîne plus longue fonctionne pareil :

```dart
class Enchantment {
  String label;
  Enchantment(this.label);
}

class Weapon {
  String name;
  Enchantment? enchantment;
  Weapon(this.name, {this.enchantment});
}

class Player {
  String name;
  Weapon? weapon;
  Player(this.name, {this.weapon});
}

void main() {
  Player alex = Player('Alex', weapon: Weapon('Épée'));
  Player maria = Player(
    'Maria',
    weapon: Weapon('Bâton', enchantment: Enchantment('Givre')),
  );
  Player novice = Player('Novice');

  print(alex.weapon?.enchantment?.label);
  print(maria.weapon?.enchantment?.label);
  print(novice.weapon?.enchantment?.label);
}
```

**Résultat :**

```text
null
Givre
null
```

Une seule ligne remplace trois `if` imbriqués. C'est exactement l'objectif.

### 12.6.2 — `?.` sur un appel de méthode qui ne retourne rien

`?.` fonctionne aussi quand la méthode ne retourne rien :

```dart
class Weapon {
  String name;
  int power;
  Weapon(this.name, this.power);

  void describe() {
    print('$name (puissance $power)');
  }
}

void main() {
  Weapon? arme = Weapon('Arc long', 30);
  Weapon? rien;

  arme?.describe();
  rien?.describe();
  print('Fin du programme');
}
```

**Résultat :**

```text
Arc long (puissance 30)
Fin du programme
```

La ligne `rien?.describe();` ne fait tout simplement rien. Pas d'erreur, pas d'affichage.

> **Remarque :** il existe aussi `?[ ]` pour l'indexation conditionnelle (`maListe?[0]`) et `...?` pour l'étalement conditionnel dans une collection. Ce sont les mêmes idées appliquées à d'autres syntaxes.

---

## 12.7 — La valeur de repli `??`

L'opérateur `??` s'appelle **opérateur de coalescence nulle**, ou plus simplement **valeur de repli** (« si c'est nul, prends plutôt ça »).

Sa règle :

> `a ?? b` : si `a` n'est pas `null`, le résultat est `a`. Sinon, le résultat est `b`.

```text
        arme ?? 'Poings nus'

        arme vaut null ?
          │
          ├── OUI ──> résultat = 'Poings nus'
          │
          └── NON ──> résultat = arme
```

Exemple :

```dart
void main() {
  String? armeAlex = 'Épée de feu';
  String? armeNovice;

  print(armeAlex ?? 'Poings nus');
  print(armeNovice ?? 'Poings nus');
}
```

**Résultat :**

```text
Épée de feu
Poings nus
```

### 12.7.1 — `??` fabrique une valeur non nullable

C'est son immense intérêt, et la solution au problème rencontré en 12.6.

```dart
void main() {
  String? arme;
  String armeAffichee = arme ?? 'Poings nus';

  print(armeAffichee.toUpperCase());
  print('Longueur : ${armeAffichee.length}');
}
```

**Résultat :**

```text
POINGS NUS
Longueur : 10
```

Regardez la deuxième ligne : `armeAffichee` est déclarée `String`, sans `?`. Le compilateur l'accepte, parce que `arme ?? 'Poings nus'` ne peut jamais valoir `null` : si le côté gauche est nul, on prend le côté droit, qui est un vrai texte.

```text
  String?  ?? String   ────>  String     (non nullable !)
  String?  ?? String?  ────>  String?    (encore nullable)
```

C'est le mécanisme le plus utile du chapitre. Retenez la formule :

> **`??` est la façon propre de transformer un nullable en non nullable.**

### 12.7.2 — Combinaison `?.` et `??`

Les deux opérateurs se combinent naturellement. C'est le duo que vous écrirez le plus souvent :

```dart
class Weapon {
  String name;
  int power;
  Weapon(this.name, this.power);
}

class Player {
  String name;
  Weapon? weapon;
  Player(this.name, {this.weapon});

  int get attaque => weapon?.power ?? 5;

  String get nomArme => weapon?.name ?? 'Poings nus';
}

void main() {
  Player alex = Player('Alex', weapon: Weapon('Épée de feu', 45));
  Player novice = Player('Novice');

  print('${alex.name}   : ${alex.nomArme} -> ${alex.attaque} dégâts');
  print('${novice.name} : ${novice.nomArme} -> ${novice.attaque} dégâts');
}
```

**Résultat :**

```text
Alex   : Épée de feu -> 45 dégâts
Novice : Poings nus -> 5 dégâts
```

Décomposons `weapon?.power ?? 5` :

```text
  weapon?.power        ->  int?   (45  ou  null)
  weapon?.power ?? 5   ->  int    (45  ou  5)
```

Le getter `attaque` retourne bien un `int`. Le reste du programme peut faire des additions sans jamais se poser de question.

### 12.7.3 — Attention : `??` ne réagit qu'à `null`

Erreur classique du débutant : croire que `??` remplace aussi `0` ou `''`.

```dart
void main() {
  int? score = 0;
  String? surnom = '';

  print(score ?? 100);
  print('surnom : "${surnom ?? 'Anonyme'}"');
}
```

**Résultat :**

```text
0
surnom : ""
```

`0` n'est pas `null`. `''` n'est pas `null`. `??` les laisse passer tels quels.

Si vous voulez aussi traiter le texte vide, il faut l'écrire explicitement :

```dart
void main() {
  String? surnom = '';
  String affichage = (surnom == null || surnom.isEmpty) ? 'Anonyme' : surnom;
  print('surnom : $affichage');
}
```

**Résultat :**

```text
surnom : Anonyme
```

---

## 12.8 — L'affectation conditionnelle `??=`

L'opérateur `??=` signifie :

> « affecte cette valeur **seulement si** la variable vaut actuellement `null` ».

```text
  pseudo ??= 'Invité';

  équivaut à :

  if (pseudo == null) {
    pseudo = 'Invité';
  }
```

Exemple :

```dart
void main() {
  String? pseudo;

  print('avant : $pseudo');
  pseudo ??= 'Invité';
  print('après : $pseudo');

  pseudo ??= 'Anonyme';
  print('encore : $pseudo');
}
```

**Résultat :**

```text
avant : null
après : Invité
encore : Invité
```

La troisième affectation n'a rien fait : `pseudo` ne valait plus `null`, donc `??=` s'est abstenu. C'est exactement le comportement voulu.

### 12.8.1 — Cas d'usage typique : les valeurs par défaut d'une configuration

```dart
class GameSettings {
  String? pseudo;
  String? difficulte;
  int? volume;

  void appliquerDefauts() {
    pseudo ??= 'Joueur1';
    difficulte ??= 'normal';
    volume ??= 70;
  }

  void afficher() {
    print('pseudo=$pseudo | difficulté=$difficulte | volume=$volume');
  }
}

void main() {
  GameSettings vierge = GameSettings();
  vierge.appliquerDefauts();
  vierge.afficher();

  GameSettings perso = GameSettings();
  perso.pseudo = 'Alex';
  perso.volume = 20;
  perso.appliquerDefauts();
  perso.afficher();
}
```

**Résultat :**

```text
pseudo=Joueur1 | difficulté=normal | volume=70
pseudo=Alex | difficulté=normal | volume=20
```

Les choix de l'utilisateur sont préservés, les trous sont bouchés. C'est le modèle « remplir les cases manquantes ».

### 12.8.2 — Cas d'usage : le cache paresseux

```dart
class Boss {
  String name;
  int health;
  String? _ficheEnCache;

  Boss(this.name, this.health);

  String _construireFiche() {
    print('  (construction de la fiche...)');
    return 'BOSS $name — $health PV';
  }

  String get fiche {
    _ficheEnCache ??= _construireFiche();
    return _ficheEnCache ?? '';
  }
}

void main() {
  Boss dragon = Boss('Dragon Noir', 5000);

  print(dragon.fiche);
  print(dragon.fiche);
}
```

**Résultat :**

```text
  (construction de la fiche...)
BOSS Dragon Noir — 5000 PV
BOSS Dragon Noir — 5000 PV
```

Le message de construction n'apparaît qu'une seule fois : au second appel, `_ficheEnCache` n'est plus `null`, donc `??=` ne recalcule rien.

> **Remarque :** la ligne `return _ficheEnCache ?? '';` peut sembler inutile, puisque `??=` vient de remplir le champ. Elle est pourtant obligatoire : `_ficheEnCache` est un **champ de classe**, et le compilateur refuse de le considérer comme non nullable. Nous expliquons précisément ce phénomène en section 12.14.

---

## 12.9 — L'opérateur d'assertion `!` et son danger

L'opérateur `!` s'appelle **null assertion operator** (opérateur d'assertion de non-nullité). Il se place **après** l'expression :

```dart
String? arme = 'Épée';
String sure = arme!;
```

Sa signification exacte est la suivante :

> « Compilateur, je te certifie que cette valeur n'est pas `null`. Laisse-moi passer, et si je me trompe, fais planter le programme. »

Ce n'est pas une vérification. Ce n'est pas une conversion. C'est **une promesse de votre part**, que le compilateur accepte sans la contrôler.

Voyez les deux issues :

```text
      arme!

      arme vaut null ?
        │
        ├── NON ──> tout va bien, on obtient la valeur
        │
        └── OUI ──> CRASH IMMÉDIAT à l'exécution
                    Null check operator used on a null value
```

Le cas qui fonctionne :

```dart
void main() {
  String? arme = 'Épée de feu';
  String armeSure = arme!;
  print(armeSure.toUpperCase());
}
```

**Résultat :**

```text
ÉPÉE DE FEU
```

Le cas qui plante :

```dart
void main() {
  String? arme;
  String armeSure = arme!;
  print(armeSure);
}
```

**Résultat :**

```text
Unhandled exception:
Null check operator used on a null value
```

Retenez ce message. `Null check operator used on a null value` veut toujours dire la même chose : **vous avez écrit `!` sur quelque chose qui valait `null`**.

### 12.9.1 — Pourquoi `!` est dangereux

Le danger de `!` n'est pas qu'il fasse planter. Le danger est qu'il **rétablit exactement le problème que le null safety avait supprimé**.

```text
  Sans null safety   : crash à l'exécution, chez l'utilisateur
  Avec null safety   : erreur à la compilation, chez vous
  Avec null safety
  et des ! partout   : crash à l'exécution, chez l'utilisateur
                       (on est revenu au point de départ)
```

Chaque `!` que vous écrivez est une petite zone du programme où le compilateur ne vous protège plus. Il vous fait confiance. Si votre raisonnement était juste aujourd'hui et qu'un collègue modifie le code demain, le `!` reste, mais la garantie a disparu.

C'est pour cette raison que la règle du chapitre est stricte :

> **`!` est un dernier recours.** Avant d'écrire un `!`, essayez dans l'ordre : `??`, `?.`, un `if` de vérification, une variable locale, ou un changement du modèle de données. Dans plus de neuf cas sur dix, l'une de ces solutions convient.

### 12.9.2 — Les alternatives, cas par cas

Voici les mauvaises habitudes les plus courantes, et leur remplacement.

**Cas 1 — Afficher une valeur qui peut manquer.**

```dart
void main() {
  String? arme;

  // À éviter : plante si arme est null
  // print(arme!);

  // Bien
  print(arme ?? 'Aucune arme');
}
```

**Résultat :**

```text
Aucune arme
```

**Cas 2 — Faire un calcul avec un nombre qui peut manquer.**

```dart
void main() {
  int? bonus;

  // À éviter
  // int total = 100 + bonus!;

  // Bien
  int total = 100 + (bonus ?? 0);
  print('Score total : $total');
}
```

**Résultat :**

```text
Score total : 100
```

**Cas 3 — Exécuter du code seulement si la valeur existe.**

```dart
void main() {
  String? arme;

  // À éviter
  // print(arme!.toUpperCase());

  // Bien
  if (arme != null) {
    print(arme.toUpperCase());
  } else {
    print('Le joueur se bat à mains nues.');
  }
}
```

**Résultat :**

```text
Le joueur se bat à mains nues.
```

Nous verrons en 12.13 pourquoi, dans ce dernier exemple, `arme.toUpperCase()` est accepté sans `!` à l'intérieur du `if`.

### 12.9.3 — Les rares cas où `!` se défend

`!` n'est pas interdit. Il est acceptable quand vous venez, sur les lignes immédiatement précédentes, de garantir vous-même la non-nullité, et que le compilateur ne peut pas le déduire.

Exemple acceptable :

```dart
class Player {
  String name;
  String? weapon;
  Player(this.name, {this.weapon});
}

void main() {
  List<Player> equipe = [
    Player('Alex', weapon: 'Épée'),
    Player('Novice'),
    Player('Maria', weapon: 'Bâton'),
  ];

  List<Player> armes = [];
  for (Player p in equipe) {
    if (p.weapon != null) {
      armes.add(p);
    }
  }

  for (Player p in armes) {
    print('${p.name} porte ${p.weapon!.toUpperCase()}');
  }
}
```

**Résultat :**

```text
Alex porte ÉPÉE
Maria porte BÂTON
```

Ici, le filtre garantit que tous les joueurs de la liste `armes` ont une arme. Le compilateur, lui, ne suit pas ce raisonnement : il voit seulement un champ `String?`. Le `!` comble l'écart.

Même dans ce cas, une version sans `!` reste plus sûre :

```dart
class Player {
  String name;
  String? weapon;
  Player(this.name, {this.weapon});
}

void main() {
  List<Player> equipe = [
    Player('Alex', weapon: 'Épée'),
    Player('Novice'),
    Player('Maria', weapon: 'Bâton'),
  ];

  for (Player p in equipe) {
    String? arme = p.weapon;
    if (arme != null) {
      print('${p.name} porte ${arme.toUpperCase()}');
    }
  }
}
```

**Résultat :**

```text
Alex porte ÉPÉE
Maria porte BÂTON
```

Aucun `!`, aucun crash possible. Préférez toujours cette forme quand elle est disponible.

---

## 12.10 — `late` : promettre une valeur pour plus tard

Il existe une situation gênante : une variable qui n'est **jamais** `null` pendant l'utilisation, mais qui ne peut pas recevoir sa valeur au moment de la déclaration.

Exemple typique : un niveau de jeu dont la carte est chargée juste après la création de l'objet.

```dart
class GameLevel {
  String name;
  List<String> map = [];

  GameLevel(this.name);
}
```

Ici on triche : on met une liste vide en attendant. Le type dit « il y a une carte », alors qu'il n'y en a pas encore.

L'autre triche consiste à rendre le champ nullable :

```dart
class GameLevel {
  String name;
  List<String>? map;

  GameLevel(this.name);
}
```

Mais alors, **tout le reste du programme** devra écrire `map?.` ou `map!` , alors que la carte existe toujours en pratique. Le type ment dans l'autre sens.

Le mot-clé `late` règle ce dilemme :

> `late` signifie : « cette variable n'est pas nullable, elle recevra sa valeur plus tard, avant toute lecture. Je m'y engage. »

```dart
class GameLevel {
  String name;
  late List<String> map;

  GameLevel(this.name);

  void charger() {
    map = ['....#', '..P..', '#...E'];
  }

  void afficher() {
    print('Niveau $name');
    for (String ligne in map) {
      print(ligne);
    }
  }
}

void main() {
  GameLevel niveau = GameLevel('Caverne');
  niveau.charger();
  niveau.afficher();
}
```

**Résultat :**

```text
Niveau Caverne
....#
..P..
#...E
```

Le champ `map` est de type `List<String>`, **non nullable**. Aucun `?` ni `!` n'est nécessaire dans `afficher()`.

### 12.10.1 — Ce que `late` change vraiment

`late` déplace la vérification :

```text
  sans late  :  vérification à la COMPILATION
                "must be assigned before it can be used"

  avec late  :  vérification à l'EXÉCUTION
                "LateInitializationError" si vous oubliez
```

C'est donc un **contrat** que vous signez. Le compilateur vous fait confiance, mais le programme, lui, vérifie au moment de la lecture.

### 12.10.2 — `late` et l'initialisation paresseuse

`late` a un second effet, moins connu et très utile : si vous donnez une valeur initiale à une variable `late`, ce calcul n'est effectué **qu'au premier accès**, et pas avant.

```dart
int calculerCarte() {
  print('  (génération coûteuse de la carte...)');
  return 4096;
}

class GameLevel {
  String name;
  late int tailleCarte = calculerCarte();

  GameLevel(this.name);
}

void main() {
  print('Création du niveau');
  GameLevel niveau = GameLevel('Caverne');
  print('Niveau créé');
  print('Taille : ${niveau.tailleCarte}');
  print('Taille : ${niveau.tailleCarte}');
}
```

**Résultat :**

```text
Création du niveau
Niveau créé
  (génération coûteuse de la carte...)
Taille : 4096
Taille : 4096
```

La génération n'a lieu ni à la création de l'objet, ni deux fois : exactement une fois, au premier accès. Si le programme n'avait jamais lu `tailleCarte`, `calculerCarte()` n'aurait jamais été appelée.

> **Attention :** `late` n'est pas une baguette magique. Si vous vous surprenez à mettre `late` partout pour faire taire le compilateur, c'est le signe que votre modèle de données est mal conçu. Demandez-vous d'abord si un constructeur ne pourrait pas remplir le champ tout de suite.

---

## 12.11 — `late final` : une valeur écrite une seule fois

`final` interdit toute modification après affectation. `late final` combine les deux idées :

> « Cette valeur sera écrite plus tard, et **une seule fois**. »

```dart
class Player {
  final String name;
  late final String id;

  Player(this.name);

  void enregistrer(int numero) {
    id = 'PL-$numero-${name.toUpperCase()}';
  }
}

void main() {
  Player alex = Player('Alex');
  alex.enregistrer(7);
  print('${alex.name} -> ${alex.id}');
}
```

**Résultat :**

```text
Alex -> PL-7-ALEX
```

Essayons maintenant d'écrire deux fois :

```dart
class Player {
  final String name;
  late final String id;

  Player(this.name);

  void enregistrer(int numero) {
    id = 'PL-$numero';
  }
}

void main() {
  Player alex = Player('Alex');
  alex.enregistrer(7);
  alex.enregistrer(9);
  print(alex.id);
}
```

**Résultat :**

```text
Unhandled exception:
LateInitializationError: Field 'id' has already been initialized.
```

Le programme s'arrête volontairement. C'est exactement ce qu'on veut : un identifiant ne doit jamais changer, et l'erreur apparaît à la seconde tentative, pas trois heures plus tard sous forme de données incohérentes.

Comparons les trois formes :

| Déclaration | Valeur au départ | Modifiable ensuite |
| --- | --- | --- |
| `final String id = 'PL-1';` | obligatoire, tout de suite | non |
| `late final String id;` | plus tard, une seule fois | non, après la première écriture |
| `String id = 'PL-1';` | obligatoire, tout de suite | oui, autant de fois qu'on veut |

> **Bonne pratique :** pour un identifiant, une configuration ou une dépendance injectée après construction, `late final` est le meilleur choix. Il exprime « cette valeur arrive plus tard, mais elle est ensuite gravée dans le marbre ».

---

## 12.12 — `LateInitializationError`

C'est l'erreur qui accompagne `late`. Elle survient quand vous **lisez** une variable `late` qui n'a pas encore reçu de valeur.

```dart
class GameLevel {
  String name;
  late List<String> map;

  GameLevel(this.name);

  void afficher() {
    print('Niveau $name, ${map.length} lignes');
  }
}

void main() {
  GameLevel niveau = GameLevel('Caverne');
  niveau.afficher();
}
```

**Résultat :**

```text
Unhandled exception:
LateInitializationError: Field 'map' has not been initialized.
```

Sur une variable locale, le message est légèrement différent :

```dart
void main() {
  late String arme;
  print(arme);
}
```

**Résultat :**

```text
Unhandled exception:
LateInitializationError: Local 'arme' has not been initialized.
```

Récapitulons les trois messages de la famille `late` :

| Message | Signification |
| --- | --- |
| `LateInitializationError: Field 'x' has not been initialized.` | vous lisez un champ `late` jamais rempli |
| `LateInitializationError: Local 'x' has not been initialized.` | vous lisez une variable locale `late` jamais remplie |
| `LateInitializationError: Field 'x' has already been initialized.` | vous écrivez une seconde fois dans un `late final` |

Comment corriger ? Trois pistes, dans l'ordre de préférence.

**Piste 1 — Donner la valeur dans le constructeur.** C'est presque toujours la bonne réponse.

```dart
class GameLevel {
  String name;
  List<String> map;

  GameLevel(this.name, this.map);

  void afficher() {
    print('Niveau $name, ${map.length} lignes');
  }
}

void main() {
  GameLevel niveau = GameLevel('Caverne', ['....#', '..P..']);
  niveau.afficher();
}
```

**Résultat :**

```text
Niveau Caverne, 2 lignes
```

**Piste 2 — Donner une valeur d'initialisation paresseuse.**

```dart
class GameLevel {
  String name;
  late List<String> map = <String>[];

  GameLevel(this.name);

  void afficher() {
    print('Niveau $name, ${map.length} lignes');
  }
}

void main() {
  GameLevel niveau = GameLevel('Caverne');
  niveau.afficher();
}
```

**Résultat :**

```text
Niveau Caverne, 0 lignes
```

**Piste 3 — Renoncer à `late` et assumer le nullable.** Si l'absence est un cas normal, c'est `?` qu'il faut, pas `late`.

> **Règle de décision :**
> - la valeur peut légitimement manquer -> `?`
> - la valeur existe toujours mais arrive un peu plus tard -> `late`
> - la valeur existe dès la création -> ni l'un ni l'autre

---

## 12.13 — La promotion de type (flow analysis)

Reprenons ce code, qui semble contradictoire avec la section 12.5 :

```dart
void main() {
  String? arme = 'Épée de feu';

  if (arme != null) {
    print(arme.toUpperCase());
    print(arme.length);
  }
}
```

**Résultat :**

```text
ÉPÉE DE FEU
11
```

Aucun `?.`, aucun `!`, et pourtant le compilateur accepte. Pourquoi ?

Parce que Dart analyse le **chemin d'exécution**. Ce mécanisme s'appelle la **flow analysis**, et son résultat s'appelle la **promotion de type** :

> À l'intérieur d'un bloc où Dart a la certitude que la variable n'est pas `null`, il change temporairement son type de `String?` en `String`.

Schéma :

```text
  String? arme = 'Épée de feu';        type ici : String?

  if (arme != null) {
      ┌──────────────────────────┐
      │  type ici : String       │   <- PROMU
      │  arme.toUpperCase() OK   │
      └──────────────────────────┘
  }

  type de nouveau : String?
```

### 12.13.1 — Les formes qui déclenchent la promotion

La promotion fonctionne avec plusieurs écritures.

**Le `if` classique :**

```dart
void main() {
  String? arme = 'Arc long';
  if (arme != null) {
    print('Arme : ${arme.toUpperCase()}');
  }
}
```

**Résultat :**

```text
Arme : ARC LONG
```

**Le retour anticipé (early return) :**

```dart
void decrire(String? arme) {
  if (arme == null) {
    print('Aucune arme équipée.');
    return;
  }
  print('Arme : ${arme.toUpperCase()} (${arme.length} caractères)');
}

void main() {
  decrire('Épée');
  decrire(null);
}
```

**Résultat :**

```text
Arme : ÉPÉE (4 caractères)
Aucune arme équipée.
```

Après le `return`, Dart sait que le seul chemin restant est celui où `arme` n'est pas `null`. Toute la suite de la fonction est promue. C'est la forme la plus lisible, et celle à privilégier.

**Le `&&` dans une condition :**

```dart
void main() {
  String? arme = 'Bâton de givre';
  if (arme != null && arme.length > 5) {
    print('Arme longue : $arme');
  }
}
```

**Résultat :**

```text
Arme longue : Bâton de givre
```

Le côté droit du `&&` n'est évalué que si le côté gauche est vrai : la promotion est donc valide dès la deuxième partie de la condition.

**Le `||` avec la négation :**

```dart
void main() {
  String? arme;
  if (arme == null || arme.isEmpty) {
    print('Le joueur se bat à mains nues.');
  }
}
```

**Résultat :**

```text
Le joueur se bat à mains nues.
```

### 12.13.2 — La promotion se perd sur une réaffectation

```dart
String? trouverArme() => null;

void main() {
  String? arme = 'Épée';

  if (arme != null) {
    arme = trouverArme();
    print(arme.length);
  }
}
```

**Résultat :**

```text
Error: Property 'length' cannot be unconditionally accessed because the
receiver can be 'null'.
```

L'affectation intermédiaire a détruit la garantie : la promotion est annulée dès qu'une nouvelle valeur potentiellement nulle est écrite dans la variable. Là encore, le compilateur a raison.

---

## 12.14 — Pourquoi la promotion échoue sur un champ de classe

Voici l'obstacle qui fait perdre le plus de temps aux débutants.

```dart
class Player {
  String name;
  String? weapon;

  Player(this.name, {this.weapon});

  void attaquer() {
    if (weapon != null) {
      print('$name attaque avec ${weapon.toUpperCase()}');
    }
  }
}

void main() {
  Player('Alex', weapon: 'Épée').attaquer();
}
```

**Résultat :**

```text
Error: Method 'toUpperCase' cannot be unconditionally invoked because the
receiver can be 'null'.
```

Le `if` est pourtant identique à celui de la section précédente. Pourquoi le compilateur refuse-t-il ici ?

Parce que `weapon` n'est pas une variable locale : c'est un **champ d'objet**, modifiable de l'extérieur. Dart ne peut pas garantir qu'il gardera sa valeur entre la ligne du `if` et la ligne suivante.

```text
  if (weapon != null) {
        │
        │   entre ces deux lignes, weapon peut changer :
        │     - un autre bout de code peut écrire joueur.weapon = null
        │     - un getter peut renvoyer une valeur différente
        │     - une sous-classe peut redéfinir weapon
        ▼
      weapon.toUpperCase()      <- plus aucune garantie
  }
```

Le cas du **getter** rend la chose évidente :

```dart
import 'dart:math';

class Player {
  final Random _hasard = Random();

  String? get weapon => _hasard.nextBool() ? 'Épée' : null;
}
```

Ici, deux lectures successives de `weapon` peuvent donner deux résultats différents. Le compilateur, incapable de distinguer un champ simple d'un getter capricieux, applique la même règle prudente à tous.

> **Nuance utile :** depuis Dart 3.2, la promotion fonctionne sur les champs **privés et `final`** (par exemple `final String? _weapon;`), car ceux-là ne peuvent ni être réécrits ni être redéfinis hors de la bibliothèque. Pour tous les autres champs, la promotion reste refusée. La solution universelle, valable dans tous les cas, est présentée dans la section suivante.

---

## 12.15 — La variable locale intermédiaire

La solution est simple, universelle et lisible :

> Copiez le champ dans une **variable locale**, puis testez cette variable locale.

```dart
class Player {
  String name;
  String? weapon;

  Player(this.name, {this.weapon});

  void attaquer() {
    final String? arme = weapon;

    if (arme == null) {
      print('$name frappe à mains nues.');
      return;
    }

    print('$name attaque avec ${arme.toUpperCase()} (${arme.length} lettres)');
  }
}

void main() {
  Player('Alex', weapon: 'Épée de feu').attaquer();
  Player('Novice').attaquer();
}
```

**Résultat :**

```text
Alex attaque avec ÉPÉE DE FEU (11 lettres)
Novice frappe à mains nues.
```

Pourquoi cela fonctionne-t-il ? Parce que `arme` est une variable **locale** et **`final`** : personne d'autre ne peut la modifier, et elle ne peut pas être réaffectée. Dart peut donc la promouvoir sans risque.

```text
  final String? arme = weapon;    on prend une PHOTO de la valeur
                                  au moment T

  if (arme == null) return;       test sur la photo

  arme.toUpperCase()              la photo ne change jamais
                                  -> promotion autorisée
```

Le même motif s'applique au chaînage d'objets :

```dart
class Weapon {
  String name;
  int power;
  Weapon(this.name, this.power);
}

class Player {
  String name;
  Weapon? weapon;
  Player(this.name, {this.weapon});

  void fiche() {
    final Weapon? arme = weapon;

    if (arme != null) {
      print('$name : ${arme.name}, puissance ${arme.power}');
    } else {
      print('$name : aucune arme');
    }
  }
}

void main() {
  Player('Alex', weapon: Weapon('Épée de feu', 45)).fiche();
  Player('Novice').fiche();
}
```

**Résultat :**

```text
Alex : Épée de feu, puissance 45
Novice : aucune arme
```

> **Le réflexe à acquérir :** dès que vous voyez `cannot be unconditionally accessed` sur un champ que vous venez de tester, écrivez `final X? local = leChamp;` en haut de la méthode et travaillez sur `local`. C'est plus court que de discuter avec le compilateur, et infiniment plus sûr qu'un `!`.

---

## 12.16 — `List<int>?` contre `List<int?>`

Le `?` ne se place pas au hasard. Selon sa position, il porte sur la **liste** ou sur ses **éléments**.

```text
  List<int>?     la LISTE peut être null,
                 mais si elle existe, ses éléments sont tous des int

  List<int?>     la LISTE existe toujours,
                 mais chaque élément peut être null

  List<int?>?    la liste peut être null
                 ET ses éléments peuvent être null
```

Schéma mémoire :

```text
  List<int>? scores = null;

      scores ──> (rien)


  List<int?> scores = [10, null, 30];

      scores ──> ┌────┬──────┬────┐
                 │ 10 │ null │ 30 │
                 └────┴──────┴────┘
```

**Cas 1 : `List<int>?` — la liste elle-même peut manquer.**

```dart
void main() {
  List<int>? scores;

  print(scores?.length);
  print(scores?.first);

  scores = [120, 90, 45];
  print(scores.length);
  print(scores[0] + 10);
}
```

**Résultat :**

```text
null
null
3
130
```

Notez `scores[0] + 10` : une fois la liste présente, ses éléments sont des `int` normaux, et l'addition ne pose aucun problème.

**Cas 2 : `List<int?>` — la liste existe, ses cases peuvent être vides.**

```dart
void main() {
  List<int?> scoresDeManche = [120, null, 45, null, 30];

  print('Nombre de manches : ${scoresDeManche.length}');

  int total = 0;
  for (int? s in scoresDeManche) {
    total += s ?? 0;
  }
  print('Total : $total');
}
```

**Résultat :**

```text
Nombre de manches : 5
Total : 195
```

Ici `scoresDeManche.length` fonctionne sans `?.` (la liste existe), mais chaque élément doit être traité avec `?? 0`.

Sans le `??`, le compilateur refuse :

```dart
void main() {
  List<int?> scores = [120, null, 45];
  int total = 0;
  for (int? s in scores) {
    total += s;
  }
  print(total);
}
```

**Résultat :**

```text
Error: A value of type 'int?' can't be assigned to a variable of type 'int'.
```

**Cas 3 : nettoyer une `List<int?>` en `List<int>`.**

```dart
void main() {
  List<int?> brut = [120, null, 45, null, 30];

  List<int> propre = brut.whereType<int>().toList();

  int somme = 0;
  for (int s in propre) {
    somme += s;
  }

  print('Brut   : $brut');
  print('Propre : $propre');
  print('Somme  : $somme');
}
```

**Résultat :**

```text
Brut   : [120, null, 45, null, 30]
Propre : [120, 45, 30]
Somme  : 195
```

`whereType<int>()` garde uniquement les éléments réellement de type `int` et écarte les `null`. Le résultat est une `List<int>` pleinement non nullable, exploitable sans précaution.

> **Bonne pratique :** dans un jeu, préférez presque toujours `List<Item>` à `List<Item?>`. Une liste vide (`[]`) exprime déjà « rien à l'intérieur ». Les trous dans une liste ne se justifient que pour une grille de jeu où une case peut être réellement vide.

---

## 12.17 — Map et valeurs nullables

Voici un point que **tous** les débutants découvrent au même endroit.

> Lire une clé dans une `Map` renvoie **toujours** un type nullable, même si la `Map` est déclarée avec des valeurs non nullables.

```dart
void main() {
  Map<String, int> scores = {'Alex': 120, 'Maria': 340};

  print(scores['Alex']);
  print(scores['Inconnu']);
}
```

**Résultat :**

```text
120
null
```

La raison est évidente une fois énoncée : la clé demandée peut ne pas exister. Dart n'a pas d'autre choix que de renvoyer `null` dans ce cas, donc le type de retour est `int?`.

Conséquence directe :

```dart
void main() {
  Map<String, int> scores = {'Alex': 120};
  int score = scores['Alex'];
  print(score);
}
```

**Résultat :**

```text
Error: A value of type 'int?' can't be assigned to a variable of type 'int'.
```

La bonne écriture utilise `??` :

```dart
void main() {
  Map<String, int> scores = {'Alex': 120, 'Maria': 340};

  int scoreAlex = scores['Alex'] ?? 0;
  int scoreInconnu = scores['Inconnu'] ?? 0;

  print('Alex    : $scoreAlex');
  print('Inconnu : $scoreInconnu');
}
```

**Résultat :**

```text
Alex    : 120
Inconnu : 0
```

### 12.17.1 — `containsKey` ne promeut pas

```dart
void main() {
  Map<String, int> scores = {'Alex': 120};

  if (scores.containsKey('Alex')) {
    int score = scores['Alex'];
    print(score);
  }
}
```

**Résultat :**

```text
Error: A value of type 'int?' can't be assigned to a variable of type 'int'.
```

`containsKey` est un simple appel de méthode : le compilateur ne fait aucun lien entre son résultat et le type de `scores['Alex']`. La solution est, une fois de plus, la variable locale :

```dart
void main() {
  Map<String, int> scores = {'Alex': 120, 'Maria': 340};

  final int? score = scores['Alex'];
  if (score != null) {
    print('Score trouvé : ${score + 10}');
  } else {
    print('Joueur inconnu');
  }
}
```

**Résultat :**

```text
Score trouvé : 130
```

### 12.17.2 — Clé absente contre valeur nulle

Attention au cas particulier d'une `Map` dont les valeurs sont elles-mêmes nullables.

```dart
void main() {
  Map<String, String?> armes = {
    'Alex': 'Épée de feu',
    'Novice': null,
  };

  print(armes['Alex']);
  print(armes['Novice']);
  print(armes['Fantôme']);

  print('Novice est dans la map ? ${armes.containsKey('Novice')}');
  print('Fantôme est dans la map ? ${armes.containsKey('Fantôme')}');
}
```

**Résultat :**

```text
Épée de feu
null
null
Novice est dans la map ? true
Fantôme est dans la map ? false
```

`armes['Novice']` et `armes['Fantôme']` donnent tous deux `null`, mais les deux situations sont différentes : le premier joueur existe sans arme, le second n'existe pas. Seul `containsKey` permet de les distinguer.

> **Bonne pratique :** évitez `Map<K, V?>`. Une valeur nulle dans une `Map` rend impossible la distinction entre « absent » et « présent mais vide ». Utilisez plutôt `Map<K, V>` et l'absence de clé.

---

## 12.18 — Paramètres nullables et valeurs par défaut

Les paramètres optionnels et le null safety se croisent sans arrêt. Voici les trois cas à connaître.

**Cas 1 — Paramètre optionnel sans valeur par défaut : le type doit être nullable.**

```dart
void creerJoueur(String nom, {String? arme}) {
  print('$nom porte ${arme ?? 'rien'}');
}

void main() {
  creerJoueur('Alex', arme: 'Épée');
  creerJoueur('Novice');
}
```

**Résultat :**

```text
Alex porte Épée
Novice porte rien
```

Si vous oubliez le `?` :

```dart
void creerJoueur(String nom, {String arme}) {
  print('$nom porte $arme');
}

void main() {
  creerJoueur('Alex');
}
```

**Résultat :**

```text
Error: The parameter 'arme' can't have a value of 'null' because of its
type 'String', but the implicit default value is 'null'.
```

Le message est explicite : un paramètre nommé non fourni vaut `null` par défaut, ce qui est incompatible avec un type non nullable.

**Cas 2 — Paramètre optionnel avec valeur par défaut : le type reste non nullable.**

```dart
void creerJoueur(String nom, {String arme = 'Poings nus', int vies = 3}) {
  print('$nom | arme: $arme | vies: $vies');
}

void main() {
  creerJoueur('Alex', arme: 'Épée de feu', vies: 5);
  creerJoueur('Novice');
}
```

**Résultat :**

```text
Alex | arme: Épée de feu | vies: 5
Novice | arme: Poings nus | vies: 3
```

C'est la forme à privilégier : le corps de la fonction n'a plus aucune vérification à faire.

**Cas 3 — Paramètres positionnels optionnels, entre crochets.**

```dart
void degats(int base, [int? bonus, String? critique]) {
  int total = base + (bonus ?? 0);
  if (critique != null) {
    total = total * 2;
    print('COUP CRITIQUE ($critique) : $total dégâts');
  } else {
    print('$total dégâts');
  }
}

void main() {
  degats(20);
  degats(20, 15);
  degats(20, 15, 'dans le dos');
}
```

**Résultat :**

```text
20 dégâts
35 dégâts
COUP CRITIQUE (dans le dos) : 70 dégâts
```

> **Règle de choix :** si une valeur de repli raisonnable existe (`0`, `''`, `'Poings nus'`), utilisez une valeur par défaut et gardez un type non nullable. Ne rendez le paramètre nullable que si « non fourni » et « fourni avec la valeur par défaut » sont deux situations réellement différentes.

---

## 12.19 — `required` et null safety

`required` marque un paramètre nommé comme **obligatoire**. Le null safety l'a rendu indispensable.

Sans null safety, tous les paramètres nommés étaient optionnels. Avec le null safety, un paramètre nommé non fourni vaudrait `null` — impossible pour un type non nullable. Il fallait donc un moyen de dire « celui-ci doit obligatoirement être fourni ».

```dart
class Player {
  String name;
  int health;
  String? weapon;
  String? guild;

  Player({
    required this.name,
    this.health = 100,
    this.weapon,
    this.guild,
  });

  void fiche() {
    print('$name | $health PV | arme: ${weapon ?? 'aucune'} | '
        'guilde: ${guild ?? 'aucune'}');
  }
}

void main() {
  Player(name: 'Alex', weapon: 'Épée de feu', guild: 'Les Loups').fiche();
  Player(name: 'Novice').fiche();
  Player(name: 'Boss', health: 5000).fiche();
}
```

**Résultat :**

```text
Alex | 100 PV | arme: Épée de feu | guilde: Les Loups
Novice | 100 PV | arme: aucune | guilde: aucune
Boss | 5000 PV | arme: aucune | guilde: aucune
```

Si l'on oublie le paramètre requis :

```dart
class Player {
  String name;
  Player({required this.name});
}

void main() {
  Player joueur = Player();
  print(joueur.name);
}
```

**Résultat :**

```text
Error: Required named parameter 'name' must be provided.
```

Voici le tableau de décision complet pour un paramètre nommé :

| Écriture | Obligatoire | Type | Valeur si non fourni |
| --- | --- | --- | --- |
| `required String name` | oui | non nullable | (impossible) |
| `String name = 'X'` | non | non nullable | `'X'` |
| `String? name` | non | nullable | `null` |
| `required String? name` | oui | nullable | (impossible, mais peut valoir `null`) |

La dernière ligne surprend : `required String? weapon` signifie « vous **devez** passer ce paramètre, mais vous avez le droit d'y mettre `null` ». C'est utile quand vous voulez forcer l'appelant à prendre position explicitement.

```dart
class Player {
  String name;
  String? weapon;
  Player({required this.name, required this.weapon});
}

void main() {
  Player alex = Player(name: 'Alex', weapon: 'Épée');
  Player novice = Player(name: 'Novice', weapon: null);
  print('${alex.name}: ${alex.weapon ?? 'aucune'}');
  print('${novice.name}: ${novice.weapon ?? 'aucune'}');
}
```

**Résultat :**

```text
Alex: Épée
Novice: aucune
```

---

## 12.20 — Bonnes pratiques (checklist)

Voici la checklist à relire avant chaque commit.

**1. Le non nullable est le défaut.**
Ne mettez un `?` que si l'absence de valeur a un sens dans votre jeu. « Je ne sais pas encore quoi mettre » n'est pas un sens : c'est un modèle incomplet.

**2. Poussez le `null` vers les bords du programme.**
Traitez l'absence au plus tôt (lecture d'un fichier, réponse réseau, saisie utilisateur), et faites circuler des types non nullables dans le reste du code.

```text
  ENTRÉE            CŒUR DU PROGRAMME              SORTIE
  (nullable)   ->   (tout est non nullable)   ->   (affichage)
      ?? et ?.            aucun ? aucun !
```

**3. `!` est un dernier recours.**
Avant chaque `!`, essayez dans l'ordre : `??`, `?.`, un `if` avec retour anticipé, une variable locale, une refonte du modèle. Si vous écrivez quand même un `!`, ajoutez un commentaire expliquant pourquoi la valeur ne peut pas être nulle.

**4. Sur un champ de classe, passez par une variable locale.**
`final X? local = leChamp;` puis testez `local`. C'est la réponse universelle à `cannot be unconditionally accessed`.

**5. Préférez le retour anticipé au `if` imbriqué.**

```dart
void decrire(String? arme) {
  if (arme == null) return;
  print(arme.toUpperCase());
}

void main() {
  decrire('Épée');
  decrire(null);
  print('Fin');
}
```

**Résultat :**

```text
ÉPÉE
Fin
```

**6. Une valeur par défaut vaut mieux qu'un nullable.**
`int vies = 3` est préférable à `int? vies` suivi de `vies ?? 3` répété dix fois.

**7. Pas de collection nullable.**
Utilisez `List<Item> inventaire = []` plutôt que `List<Item>? inventaire`. Une liste vide dit déjà « rien dedans ».

**8. Pas de `Map<K, V?>`.**
L'absence de clé exprime déjà l'absence de valeur.

**9. `late` uniquement pour une valeur qui arrive vraiment plus tard.**
Si le constructeur peut remplir le champ, faites-le. `late` mis partout est une manière déguisée de désactiver le null safety.

**10. `late final` pour ce qui s'écrit une seule fois.**
Identifiants, configuration, dépendances injectées après construction.

**11. Nommez le repli, ne le devinez pas.**
`weapon?.name ?? 'Poings nus'` est plus clair que `weapon?.name ?? ''`.

**12. Lisez les messages du compilateur en entier.**
Dart indique presque toujours la correction à appliquer dans la ligne `Try ...`.

Résumé visuel de la décision :

```text
  La valeur peut-elle légitimement manquer ?
    │
    ├── NON ─── existe-t-elle dès la construction ?
    │             ├── OUI ──> type normal      : String nom
    │             └── NON ──> late             : late String nom
    │                          (ou late final)
    │
    └── OUI ─── type nullable                  : String? arme
                  puis, à l'usage :
                    - lecture simple  ->  ?? valeurDeRepli
                    - appel de membre ->  ?.
                    - logique         ->  if (x == null) return;
                    - champ de classe ->  final X? local = champ;
                    - !               ->  seulement en dernier recours
```

---

## 12.21 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `Error: The value 'null' can't be assigned to a variable of type 'String' because 'String' is not nullable.` | Vous affectez `null` à un type non nullable. | Déclarez `String? arme;` si l'absence a un sens, sinon donnez une vraie valeur. |
| `Error: Non-nullable variable 'nom' must be assigned before it can be used.` | Vous lisez une variable non nullable jamais initialisée. | Initialisez-la à la déclaration, ou déclarez-la `late`, ou rendez-la nullable. |
| `Error: Field 'name' should be initialized because its type 'String' doesn't allow null.` | Un champ de classe non nullable n'a ni valeur par défaut ni initialisation par le constructeur. | Ajoutez `= 'valeur'`, ou passez par le constructeur (`Player(this.name)`), ou utilisez `late`. |
| `Error: Property 'length' cannot be unconditionally accessed because the receiver can be 'null'.` | Vous lisez une propriété sur une variable nullable sans précaution. | Utilisez `arme?.length`, ou `(arme ?? '').length`, ou testez avec `if (arme != null)`. |
| `Error: Method 'toUpperCase' cannot be unconditionally invoked because the receiver can be 'null'.` | Même cause, sur un appel de méthode. | `arme?.toUpperCase()` ou un `if` avec retour anticipé. |
| `Error: A value of type 'String?' can't be assigned to a variable of type 'String'.` | Vous rangez un nullable dans un non nullable. | `String sure = arme ?? 'Poings nus';` |
| `Error: A value of type 'int?' can't be assigned to a variable of type 'int'.` | Cas très fréquent avec `maMap['cle']` ou `liste?.length`. | `int score = scores['Alex'] ?? 0;` |
| `Error: The argument type 'String?' can't be assigned to the parameter type 'String'.` | Vous passez une valeur nullable à un paramètre non nullable. | `afficher(arme ?? 'Aucune');` |
| `Error: The operator '+' isn't defined for the type 'int?'.` | Vous faites un calcul directement sur un nombre nullable. | `int total = base + (bonus ?? 0);` |
| `Error: The parameter 'arme' can't have a value of 'null' because of its type 'String', but the implicit default value is 'null'.` | Paramètre nommé optionnel non nullable et sans valeur par défaut. | `{String arme = 'Poings nus'}` ou `{String? arme}`. |
| `Error: Required named parameter 'name' must be provided.` | Vous appelez un constructeur sans fournir un paramètre `required`. | Passez le paramètre : `Player(name: 'Alex')`. |
| `Unhandled exception: Null check operator used on a null value` | Vous avez écrit `!` sur une valeur qui valait `null`. | Supprimez le `!` et utilisez `??`, `?.` ou un `if`. C'est presque toujours possible. |
| `Unhandled exception: LateInitializationError: Field 'map' has not been initialized.` | Vous lisez un champ `late` avant de lui donner une valeur. | Remplissez-le dans le constructeur, ou donnez-lui une initialisation paresseuse. |
| `Unhandled exception: LateInitializationError: Local 'arme' has not been initialized.` | Même cause, sur une variable locale `late`. | Affectez la variable avant toute lecture. |
| `Unhandled exception: LateInitializationError: Field 'id' has already been initialized.` | Vous écrivez deux fois dans un `late final`. | N'écrivez qu'une fois, ou retirez `final` si la valeur doit changer. |
| `Warning: The '!' will have no effect because the receiver can't be null.` | Vous mettez un `!` sur une valeur déjà non nullable. | Supprimez le `!`. |
| `Warning: The receiver can't be null, so the null-aware operator '?.' is unnecessary.` | Vous mettez un `?.` sur une valeur déjà non nullable. | Remplacez `?.` par `.`. |

---

## 12.22 — Résumé du chapitre

| Opérateur / mot-clé | Signification | Exemple |
| --- | --- | --- |
| `Type` | non nullable : contient toujours une valeur | `String nom = 'Alex';` |
| `Type?` | nullable : contient une valeur **ou** `null` | `String? arme;` |
| `?.` | accès sûr : renvoie `null` au lieu de planter | `arme?.length` |
| `??` | valeur de repli si l'expression est `null` | `arme ?? 'Poings nus'` |
| `??=` | affecte seulement si la variable vaut `null` | `pseudo ??= 'Invité';` |
| `!` | affirme « ce n'est pas null » — dernier recours | `arme!.length` |
| `?[]` | indexation sûre sur une collection nullable | `sac?[0]` |
| `...?` | étalement sûr d'une collection nullable | `[...?bonus]` |
| `late` | valeur non nullable fournie plus tard | `late List<String> map;` |
| `late final` | valeur fournie plus tard, écrite une seule fois | `late final String id;` |
| `required` | paramètre nommé obligatoire | `Player({required this.name})` |
| promotion de type | Dart considère la variable comme non nullable dans un bloc sûr | `if (a != null) { a.length }` |
| variable locale | copie d'un champ pour autoriser la promotion | `final String? x = weapon;` |
| `whereType<T>()` | retire les `null` d'une `List<T?>` | `brut.whereType<int>().toList()` |

Les trois phrases à retenir :

1. Le non nullable est le défaut ; le `?` est une décision de conception, pas un dépannage.
2. `??` est l'outil qui transforme proprement un nullable en non nullable.
3. `!` remet le crash à l'exécution : c'est un dernier recours, jamais un réflexe.

---

## 12.23 — Exercices

### Exercice 1 — Déclarations correctes (facile)

Le code suivant ne compile pas. Corrigez les déclarations pour qu'il fonctionne, en respectant les intentions écrites en commentaire.

```dart
void main() {
  String nom = null;        // le joueur a toujours un nom
  int vies;                 // le joueur a toujours 3 vies au départ
  String arme = null;       // le joueur peut ne pas avoir d'arme
  String guilde;            // le joueur peut ne pas avoir de guilde

  print('$nom | $vies vies | $arme | $guilde');
}
```

Sortie attendue :

```text
Alex | 3 vies | null | null
```

### Exercice 2 — Valeur de repli (facile)

Écrivez une fonction `String nomArme(String? arme)` qui retourne l'arme si elle existe, et `'Poings nus'` sinon. Testez-la avec `'Épée de feu'` et avec `null`.

### Exercice 3 — Accès sûr (facile)

Une variable `String? guilde` peut valoir `null`. Affichez le nom de la guilde en majuscules et sa longueur, sans jamais faire planter le programme, pour les deux cas : `'Les Loups'` et `null`.

### Exercice 4 — Affectation conditionnelle (facile)

Écrivez une classe `GameSettings` avec trois champs nullables : `pseudo`, `difficulte`, `volume`. Ajoutez une méthode `appliquerDefauts()` qui utilise `??=` pour poser `'Joueur1'`, `'normal'` et `70` uniquement sur les champs vides.

### Exercice 5 — Combiner `?.` et `??` (moyen)

Vous disposez d'une classe `Weapon(String name, int power)` et d'une classe `Player` avec un champ `Weapon? weapon`. Ajoutez au joueur deux getters :

- `int get attaque` qui vaut la puissance de l'arme, ou `5` si aucune arme ;
- `String get nomArme` qui vaut le nom de l'arme, ou `'Poings nus'`.

### Exercice 6 — Supprimer un `!` dangereux (moyen)

Réécrivez ce code sans aucun `!`, en conservant le même comportement pour un joueur armé et en affichant `'Novice ne fait aucun dégât'` pour un joueur sans arme.

```dart
void main() {
  String? arme;
  print('Dégâts : ${arme!.length * 10}');
}
```

### Exercice 7 — Retour anticipé (moyen)

Écrivez une fonction `void decrire(String? arme)` qui :

- affiche `'Aucune arme équipée.'` puis s'arrête si `arme` vaut `null` ;
- sinon affiche `'ÉPÉE (4 lettres)'` par exemple, en majuscules et avec la longueur.

Utilisez la promotion de type : votre code ne doit contenir ni `?.` ni `!`.

### Exercice 8 — Champ de classe et promotion (moyen)

Le code suivant est refusé par le compilateur. Corrigez-le avec une variable locale intermédiaire.

```dart
class Player {
  String name;
  String? weapon;
  Player(this.name, {this.weapon});

  void attaquer() {
    if (weapon != null) {
      print('$name attaque avec ${weapon.toUpperCase()}');
    }
  }
}
```

### Exercice 9 — `late` et `late final` (moyen)

Écrivez une classe `GameLevel` avec :

- un champ `final String name` fourni au constructeur ;
- un champ `late final String id` rempli par une méthode `enregistrer(int numero)` sous la forme `'LVL-7-CAVERNE'` ;
- un champ `late List<String> map` rempli par une méthode `charger()` ;
- une méthode `afficher()` qui montre l'identifiant et le nombre de lignes de la carte.

### Exercice 10 — `List<int?>` (difficile)

Vous recevez les scores de cinq manches : `[120, null, 45, null, 30]`. Une manche non jouée vaut `null`.

Affichez :

- le total des scores (les manches non jouées comptent pour 0) ;
- le nombre de manches réellement jouées ;
- la moyenne sur les manches jouées, avec deux décimales.

Utilisez `whereType<int>()`.

### Exercice 11 — Map de scores (difficile)

Vous disposez de `Map<String, int> scores = {'Alex': 120, 'Maria': 340, 'Samir': 95};`.

Écrivez une fonction `void afficherScore(Map<String, int> scores, String nom)` qui affiche :

- `'Maria : 340 points'` si le joueur existe ;
- `'Fantôme : joueur inconnu'` sinon.

Écrivez ensuite une fonction `String? meilleurJoueur(Map<String, int> scores)` qui retourne le nom du joueur au plus haut score, ou `null` si la map est vide. Testez avec une map pleine et une map vide.

### Exercice 12 — Mini-projet : profil de joueur avec champs optionnels (difficile)

Créez une classe `PlayerProfile` modélisant le profil d'un joueur en ligne.

Champs :

| Champ | Type | Règle |
| --- | --- | --- |
| `name` | `String` | obligatoire, jamais `null` |
| `level` | `int` | valeur par défaut `1` |
| `weapon` | `String?` | facultatif |
| `guild` | `String?` | facultatif |
| `bio` | `String?` | facultatif |
| `friends` | `List<String>` | jamais `null`, vide par défaut |
| `bestScore` | `int?` | inconnu tant qu'aucune partie n'est terminée |
| `id` | `late final String` | attribué une seule fois par `enregistrer(int numero)` |

Méthodes demandées :

- `void enregistrer(int numero)` : donne à `id` la valeur `'PL-<numero>-<NOM EN MAJUSCULES>'` ;
- `void terminerPartie(int score)` : met à jour `bestScore` uniquement si le nouveau score est meilleur (ou si aucun score n'existe encore) ;
- `String get resume` : retourne une ligne du type
  `'Alex (niv. 5) | arme: Épée de feu | guilde: Les Loups | amis: 2 | record: 340'`,
  avec `'aucune'`, `'aucun'` et `'aucun record'` pour les champs absents ;
- `void afficherBio()` : affiche la bio, ou `'Ce joueur n'a pas encore écrit de bio.'`.

Contraintes : aucun `!` dans tout le programme.

Dans `main()`, créez deux profils (un complet, un minimal), enregistrez-les, jouez quelques parties, et affichez les résumés.

---

## 12.24 — Corrections des exercices

### Correction 1

```dart
void main() {
  String nom = 'Alex';
  int vies = 3;
  String? arme;
  String? guilde;

  print('$nom | $vies vies | $arme | $guilde');
}
```

**Résultat :**

```text
Alex | 3 vies | null | null
```

**Explication :** deux corrections de nature différente. `nom` et `vies` existent toujours : ils restent non nullables, mais doivent recevoir une valeur à la déclaration, sinon le compilateur affiche `Non-nullable variable 'vies' must be assigned before it can be used.`. `arme` et `guilde` peuvent légitimement manquer : on ajoute le `?`. Notez qu'il est inutile d'écrire `= null` : une variable nullable non initialisée vaut déjà `null`.

---

### Correction 2

```dart
String nomArme(String? arme) {
  return arme ?? 'Poings nus';
}

void main() {
  print(nomArme('Épée de feu'));
  print(nomArme(null));
}
```

**Résultat :**

```text
Épée de feu
Poings nus
```

**Explication :** le paramètre est déclaré `String?` puisqu'il accepte `null`, mais le type de retour est `String` **non nullable**. C'est possible grâce à `??` : l'expression `arme ?? 'Poings nus'` ne peut jamais valoir `null`, donc son type est `String`. Le code appelant peut ensuite utiliser le résultat sans aucune précaution.

---

### Correction 3

```dart
void afficherGuilde(String? guilde) {
  print('Guilde   : ${guilde?.toUpperCase() ?? 'AUCUNE'}');
  print('Longueur : ${guilde?.length ?? 0}');
}

void main() {
  afficherGuilde('Les Loups');
  afficherGuilde(null);
}
```

**Résultat :**

```text
Guilde   : LES LOUPS
Longueur : 9
Guilde   : AUCUNE
Longueur : 0
```

**Explication :** `guilde?.toUpperCase()` vaut `null` quand la guilde est absente, sans jamais planter. On enchaîne ensuite `?? 'AUCUNE'` pour obtenir un texte affichable. Le duo `?.` suivi de `??` est le motif le plus courant du null safety : le premier protège l'accès, le second fournit le repli.

---

### Correction 4

```dart
class GameSettings {
  String? pseudo;
  String? difficulte;
  int? volume;

  void appliquerDefauts() {
    pseudo ??= 'Joueur1';
    difficulte ??= 'normal';
    volume ??= 70;
  }

  void afficher() {
    print('pseudo=$pseudo | difficulté=$difficulte | volume=$volume');
  }
}

void main() {
  GameSettings vierge = GameSettings();
  vierge.appliquerDefauts();
  vierge.afficher();

  GameSettings perso = GameSettings();
  perso.pseudo = 'Alex';
  perso.volume = 20;
  perso.appliquerDefauts();
  perso.afficher();
}
```

**Résultat :**

```text
pseudo=Joueur1 | difficulté=normal | volume=70
pseudo=Alex | difficulté=normal | volume=20
```

**Explication :** `??=` n'écrit que si la case est vide. Sur le second objet, `pseudo` et `volume` avaient déjà été choisis par l'utilisateur : ils sont conservés. Seul `difficulte`, resté `null`, reçoit sa valeur par défaut. Écrire `pseudo = pseudo ?? 'Joueur1';` produirait le même résultat, mais `??=` est plus court et exprime mieux l'intention.

---

### Correction 5

```dart
class Weapon {
  final String name;
  final int power;

  Weapon(this.name, this.power);
}

class Player {
  final String name;
  Weapon? weapon;

  Player(this.name, {this.weapon});

  int get attaque => weapon?.power ?? 5;

  String get nomArme => weapon?.name ?? 'Poings nus';

  void fiche() {
    print('$name : $nomArme -> $attaque dégâts');
  }
}

void main() {
  Player('Alex', weapon: Weapon('Épée de feu', 45)).fiche();
  Player('Maria', weapon: Weapon('Bâton de givre', 30)).fiche();
  Player('Novice').fiche();
}
```

**Résultat :**

```text
Alex : Épée de feu -> 45 dégâts
Maria : Bâton de givre -> 30 dégâts
Novice : Poings nus -> 5 dégâts
```

**Explication :** l'intérêt de ces deux getters est de **confiner la nullité à un seul endroit**. Le champ `weapon` reste nullable, parce qu'un joueur peut réellement être désarmé. Mais `attaque` retourne un `int` et `nomArme` retourne une `String` : tout le reste du jeu (calcul de dégâts, affichage) manipule des types non nullables et n'a plus jamais à écrire `?.` ou `??`.

---

### Correction 6

```dart
void afficherDegats(String nom, String? arme) {
  if (arme == null) {
    print('$nom ne fait aucun dégât');
    return;
  }
  print('$nom inflige ${arme.length * 10} dégâts');
}

void main() {
  afficherDegats('Alex', 'Épée');
  afficherDegats('Novice', null);
}
```

**Résultat :**

```text
Alex inflige 40 dégâts
Novice ne fait aucun dégât
```

**Explication :** le code initial contenait `arme!.length`, qui produisait `Null check operator used on a null value` dès que l'arme manquait. Ici, le `if` traite explicitement le cas de l'absence, puis `return`. Après ce `return`, Dart sait qu'`arme` ne peut plus être `null` : c'est la promotion de type. On écrit donc `arme.length` sans `!`, et le programme ne peut plus planter. Un paramètre est une variable locale, donc la promotion s'applique directement.

---

### Correction 7

```dart
void decrire(String? arme) {
  if (arme == null) {
    print('Aucune arme équipée.');
    return;
  }
  print('${arme.toUpperCase()} (${arme.length} lettres)');
}

void main() {
  decrire('Épée');
  decrire('Arc long');
  decrire(null);
}
```

**Résultat :**

```text
ÉPÉE (4 lettres)
ARC LONG (8 lettres)
Aucune arme équipée.
```

**Explication :** le retour anticipé est la forme la plus lisible du null safety. On traite le cas anormal en premier, on sort, et tout le reste de la fonction travaille sur une valeur garantie. Comparez avec la version imbriquée : le corps principal aurait été décalé d'un niveau d'indentation à l'intérieur d'un `if (arme != null) { ... }`. Sur une fonction de trente lignes, la différence de lisibilité devient considérable.

---

### Correction 8

```dart
class Player {
  final String name;
  String? weapon;

  Player(this.name, {this.weapon});

  void attaquer() {
    final String? arme = weapon;

    if (arme == null) {
      print('$name frappe à mains nues.');
      return;
    }

    print('$name attaque avec ${arme.toUpperCase()}');
  }
}

void main() {
  Player('Alex', weapon: 'Épée de feu').attaquer();
  Player('Novice').attaquer();
}
```

**Résultat :**

```text
Alex attaque avec ÉPÉE DE FEU
Novice frappe à mains nues.
```

**Explication :** le code d'origine échouait avec `Method 'toUpperCase' cannot be unconditionally invoked because the receiver can be 'null'.`, alors même que le `if` semblait correct. La raison est que `weapon` est un **champ**, susceptible d'être modifié entre les deux lignes par un autre bout de code. La copie dans la variable locale `arme`, déclarée `final`, fige la valeur : Dart peut alors la promouvoir en toute sécurité. Retenez ce réflexe, il vous servira dans tous vos projets Flutter.

---

### Correction 9

```dart
class GameLevel {
  final String name;
  late final String id;
  late List<String> map;

  GameLevel(this.name);

  void enregistrer(int numero) {
    id = 'LVL-$numero-${name.toUpperCase()}';
  }

  void charger() {
    map = ['....#', '..P..', '#...E'];
  }

  void afficher() {
    print('$id : ${map.length} lignes');
    for (String ligne in map) {
      print('  $ligne');
    }
  }
}

void main() {
  GameLevel niveau = GameLevel('Caverne');
  niveau.enregistrer(7);
  niveau.charger();
  niveau.afficher();
}
```

**Résultat :**

```text
LVL-7-CAVERNE : 3 lignes
  ....#
  ..P..
  #...E
```

**Explication :** `id` est déclaré `late final` : il n'existe pas à la construction, il est écrit une seule fois par `enregistrer`, et toute seconde écriture provoquerait `LateInitializationError: Field 'id' has already been initialized.`. `map` est simplement `late` : la carte peut être rechargée plusieurs fois, par exemple lors d'un redémarrage du niveau. Dans les deux cas, les champs restent **non nullables** : `afficher()` s'écrit sans `?.` ni `!`. En revanche, appeler `afficher()` avant `charger()` déclencherait `LateInitializationError: Field 'map' has not been initialized.` : c'est le prix du contrat `late`.

---

### Correction 10

```dart
void main() {
  List<int?> manches = [120, null, 45, null, 30];

  int total = 0;
  for (int? score in manches) {
    total += score ?? 0;
  }

  List<int> jouees = manches.whereType<int>().toList();
  int nbJouees = jouees.length;
  double moyenne = nbJouees == 0 ? 0.0 : total / nbJouees;

  print('Manches proposées : ${manches.length}');
  print('Manches jouées    : $nbJouees');
  print('Total             : $total');
  print('Moyenne           : ${moyenne.toStringAsFixed(2)}');
}
```

**Résultat :**

```text
Manches proposées : 5
Manches jouées    : 3
Total             : 195
Moyenne           : 65.00
```

**Explication :** trois points importants. Premièrement, `manches.length` s'écrit sans `?.` : la liste est de type `List<int?>`, donc **la liste existe toujours**, seuls ses éléments peuvent manquer. Deuxièmement, `total += score ?? 0` est obligatoire : sans le `??`, le compilateur refuserait avec `A value of type 'int?' can't be assigned to a variable of type 'int'.`. Troisièmement, `whereType<int>()` produit une `List<int>` totalement non nullable, ce qui permet un calcul de moyenne propre. Notez le `0.0` du ternaire : avec `0`, le type de l'expression serait `num` et l'affectation à un `double` serait refusée.

---

### Correction 11

```dart
void afficherScore(Map<String, int> scores, String nom) {
  final int? score = scores[nom];

  if (score == null) {
    print('$nom : joueur inconnu');
    return;
  }

  print('$nom : $score points');
}

String? meilleurJoueur(Map<String, int> scores) {
  String? meilleur;
  int record = -1;

  for (MapEntry<String, int> entree in scores.entries) {
    if (entree.value > record) {
      record = entree.value;
      meilleur = entree.key;
    }
  }

  return meilleur;
}

void main() {
  Map<String, int> scores = {'Alex': 120, 'Maria': 340, 'Samir': 95};
  Map<String, int> vide = {};

  afficherScore(scores, 'Maria');
  afficherScore(scores, 'Fantôme');

  print('Meilleur joueur : ${meilleurJoueur(scores) ?? 'aucun joueur'}');
  print('Map vide        : ${meilleurJoueur(vide) ?? 'aucun joueur'}');
}
```

**Résultat :**

```text
Maria : 340 points
Fantôme : joueur inconnu
Meilleur joueur : Maria
Map vide        : aucun joueur
```

**Explication :** `scores[nom]` retourne toujours un `int?`, même si la `Map` est déclarée `Map<String, int>` : la clé demandée peut ne pas exister. On range donc le résultat dans une variable locale `final int? score`, on teste, et la promotion fait le reste. Pour `meilleurJoueur`, le type de retour `String?` est ici pleinement justifié : une map vide n'a pas de meilleur joueur, et `null` est la réponse honnête. L'appelant est ensuite forcé par le compilateur de traiter ce cas, ce qu'il fait avec `?? 'aucun joueur'`.

---

### Correction 12 — Mini-projet : profil de joueur avec champs optionnels

```dart
class PlayerProfile {
  final String name;
  int level;
  String? weapon;
  String? guild;
  String? bio;
  List<String> friends;
  int? bestScore;
  late final String id;

  PlayerProfile({
    required this.name,
    this.level = 1,
    this.weapon,
    this.guild,
    this.bio,
    List<String>? friends,
  }) : friends = friends ?? <String>[];

  void enregistrer(int numero) {
    id = 'PL-$numero-${name.toUpperCase()}';
  }

  void terminerPartie(int score) {
    final int? record = bestScore;

    if (record == null || score > record) {
      bestScore = score;
    }
  }

  String get resume {
    return '$name (niv. $level) '
        '| arme: ${weapon ?? 'aucune'} '
        '| guilde: ${guild ?? 'aucune'} '
        '| amis: ${friends.length} '
        '| record: ${bestScore ?? 'aucun record'}';
  }

  void afficherBio() {
    final String? texte = bio;

    if (texte == null || texte.isEmpty) {
      print("Ce joueur n'a pas encore écrit de bio.");
      return;
    }

    print(texte);
  }
}

void main() {
  PlayerProfile alex = PlayerProfile(
    name: 'Alex',
    level: 5,
    weapon: 'Épée de feu',
    guild: 'Les Loups',
    bio: 'Explorateur de cavernes depuis 2019.',
    friends: ['Maria', 'Samir'],
  );

  PlayerProfile novice = PlayerProfile(name: 'Novice');

  alex.enregistrer(7);
  novice.enregistrer(8);

  alex.terminerPartie(120);
  alex.terminerPartie(340);
  alex.terminerPartie(90);

  print('--- Profil complet ---');
  print(alex.id);
  print(alex.resume);
  alex.afficherBio();

  print('');
  print('--- Profil minimal ---');
  print(novice.id);
  print(novice.resume);
  novice.afficherBio();

  print('');
  print('--- Après une première partie du novice ---');
  novice.terminerPartie(15);
  print(novice.resume);
}
```

**Résultat :**

```text
--- Profil complet ---
PL-7-ALEX
Alex (niv. 5) | arme: Épée de feu | guilde: Les Loups | amis: 2 | record: 340
Explorateur de cavernes depuis 2019.

--- Profil minimal ---
PL-8-NOVICE
Novice (niv. 1) | arme: aucune | guilde: aucune | amis: 0 | record: aucun record
Ce joueur n'a pas encore écrit de bio.

--- Après une première partie du novice ---
Novice (niv. 1) | arme: aucune | guilde: aucune | amis: 0 | record: 15
```

**Explication :** ce mini-projet applique toute la checklist de la section 12.20.

- `name` est `required` et non nullable : un profil sans nom n'a aucun sens.
- `level` a une valeur par défaut (`1`) plutôt qu'un type nullable : c'est la règle « une valeur par défaut vaut mieux qu'un nullable ».
- `weapon`, `guild` et `bio` sont nullables, car leur absence est une information réelle sur le joueur.
- `friends` n'est **jamais** `null` : le paramètre du constructeur est nullable (`List<String>? friends`), mais la liste d'initialisation le transforme immédiatement en liste vide avec `friends ?? <String>[]`. Tout le reste du programme peut alors écrire `friends.length` sans précaution. C'est le principe « pousser le `null` vers les bords ».
- `bestScore` est `int?` parce que « aucune partie terminée » et « score de zéro » sont deux situations différentes. C'est le cas rare où le nullable est vraiment le bon choix.
- `id` est `late final` : il est attribué une seule fois par `enregistrer`, et le compilateur n'oblige jamais le reste du code à le tester.
- Dans `terminerPartie`, `bestScore` est un champ : on le copie dans `final int? record` avant de le comparer, sinon `score > record` serait refusé. Notez aussi l'ordre de la condition : `record == null || score > record`. Le `||` court-circuite, donc `score > record` n'est évalué que si `record` n'est pas `null`, et la promotion s'applique.
- `afficherBio` combine le test de nullité et le test de texte vide, car `??` seul ne remplacerait pas une bio vide.
- Le programme entier ne contient **aucun `!`**. C'est l'objectif à viser dans vos propres projets.

---

## Et maintenant ?

Vous savez désormais empêcher une valeur absente de faire planter votre programme. Le compilateur travaille pour vous : il refuse le code dangereux avant même son exécution, et vous disposez de `?.`, `??`, `??=`, `late` et `late final` pour répondre proprement à chacun de ses refus. Vous savez surtout que `!` est un dernier recours, et non un moyen de faire taire l'analyseur.

Mais toutes les défaillances ne se ramènent pas à `null`. Un fichier de sauvegarde peut être corrompu, une division peut se faire par zéro, une conversion de texte en nombre peut échouer, un serveur peut ne pas répondre. Ces situations ne se règlent ni par un `?` ni par un `??` : il faut pouvoir **signaler** un problème, puis le **rattraper** au bon endroit.

C'est l'objet du chapitre suivant, consacré aux exceptions : `throw`, `try`, `catch`, `on`, `finally`, et l'écriture de vos propres classes d'exception pour votre jeu.

Rendez-vous au chapitre 13 : [13-PARTIE-1A—EXCEPTIONS-ET-GESTION-DES-ERREURS.md](13-PARTIE-1A—EXCEPTIONS-ET-GESTION-DES-ERREURS.md)
