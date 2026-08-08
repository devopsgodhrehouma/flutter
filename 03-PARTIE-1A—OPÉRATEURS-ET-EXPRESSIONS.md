# PARTIE 1A — DART
# CHAPITRE 03 — OPÉRATEURS ET EXPRESSIONS

> **Niveau :** débutant
> **Durée estimée :** 3 h
> **Pré-requis :** chapitre 02 — Variables, constantes et types
> **Ce que vous saurez faire à la fin :** écrire des calculs, des comparaisons et des expressions booléennes complètes, prêtes à être réutilisées dans des conditions.

---

## 03.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- effectuer des calculs avec Dart ;
- utiliser les opérateurs arithmétiques ;
- comprendre la division entière ;
- calculer un reste avec `%` ;
- modifier rapidement une variable avec `+=`, `-=`, `*=`, `/=` ;
- utiliser `++` et `--` ;
- comparer des valeurs ;
- créer des expressions booléennes ;
- utiliser les opérateurs logiques `&&`, `||` et `!` ;
- comprendre les priorités entre opérateurs ;
- utiliser l'opérateur ternaire ;
- construire des expressions qui seront utilisées ensuite avec `if`, `else` et `switch`.

---

## 03.1 — Qu'est-ce qu'un opérateur ?

Un opérateur est un symbole permettant d'effectuer une opération.

Exemple :

```dart
int resultat = 10 + 5;
```

Dans cette instruction :

```text
10 + 5
```

le symbole :

```text
+
```

est un opérateur.

Les valeurs :

```text
10
5
```

sont les opérandes.

On peut donc représenter :

```text
10 + 5
│ │ │
│ │ └── opérande
│ └──────── opérateur
└─────────────── opérande
```

---

## 03.2 — Les opérateurs arithmétiques

Les principaux opérateurs mathématiques sont :

| Opérateur | Signification | Exemple |
|---|---|---|
| `+` | Addition | `10 + 5` |
| `-` | Soustraction | `10 - 5` |
| `*` | Multiplication | `10 * 5` |
| `/` | Division | `10 / 5` |
| `~/` | Division entière | `10 ~/ 3` |
| `%` | Reste d'une division | `10 % 3` |

---

## 03.3 — Addition

Exemple simple :

```dart
void main() {
  int a = 10;
  int b = 5;

  int resultat = a + b;

  print(resultat);
}
```

Résultat :

```text
15
```

---

## 03.4 — Addition dans un contexte de jeu

```dart
void main() {
  int score = 100;
  int bonus = 50;

  int scoreFinal = score + bonus;

  print('Score final : $scoreFinal');
}
```

Résultat :

```text
Score final : 150
```

---

## 03.5 — Soustraction

```dart
void main() {
  int vies = 3;

  int nouvellesVies = vies - 1;

  print(nouvellesVies);
}
```

Résultat :

```text
2
```

---

## 03.6 — Multiplication

```dart
void main() {
  double prix = 25.0;
  int quantite = 4;

  double total = prix * quantite;

  print(total);
}
```

Résultat :

```text
100.0
```

---

## 03.7 — Division classique `/`

Exemple :

```dart
void main() {
  print(10 / 2);
}
```

Résultat :

```text
5.0
```

Même lorsque le résultat mathématique est entier, l'opérateur `/` retourne une valeur de type `double`.

Exemple :

```dart
double resultat = 10 / 2;
```

---

## 03.8 — Division produisant un nombre décimal

```dart
void main() {
  print(10 / 4);
}
```

Résultat :

```text
2.5
```

---

## 03.9 — Division entière avec `~/`

Dart possède un opérateur très pratique :

```text
~/
```

Il effectue une division entière.

Exemple :

```dart
void main() {
  print(10 ~/ 3);
}
```

Résultat :

```text
3
```

La partie décimale est supprimée.

---

## 03.10 — Comparaison entre `/` et `~/`

```dart
void main() {
  print(10 / 3);
  print(10 ~/ 3);
}
```

Résultat approximatif :

```text
3.3333333333333335
3
```

Donc :

```text
/
→ division réelle

~/
→ division entière
```

---

## 03.11 — Exemple pratique avec `~/`

Imaginons :

```text
10 pièces
```

et :

```text
3 joueurs
```

Combien de pièces chaque joueur reçoit-il entièrement ?

```dart
void main() {
  int pieces = 10;
  int joueurs = 3;

  int piecesParJoueur = pieces ~/ joueurs;

  print(piecesParJoueur);
}
```

Résultat :

```text
3
```

---

## 03.12 — Le modulo `%`

L'opérateur `%` donne le reste d'une division.

Exemple :

```dart
void main() {
  print(10 % 3);
}
```

Calcul :

```text
10 / 3
```

On peut faire :

```text
3 × 3 = 9
```

Il reste :

```text
1
```

Donc :

```text
10 % 3 = 1
```

---

## 03.13 — Pourquoi `%` est très utile ?

Le modulo est extrêmement utile en programmation.

Par exemple, pour vérifier si un nombre est pair.

Un nombre pair divisé par 2 produit un reste de :

```text
0
```

Exemple :

```dart
void main() {
  int nombre = 10;

  print(nombre % 2);
}
```

Résultat :

```text
0
```

Avec :

```dart
int nombre = 11;
```

Résultat :

```text
1
```

Nous utiliserons bientôt cela avec :

```dart
if (nombre % 2 == 0)
```

---

## 03.14 — Exemple modulo dans un jeu

Supposons qu'une récompense spéciale apparaisse tous les 5 niveaux.

On pourrait tester :

```dart
niveau % 5 == 0
```

Exemples :

```text
niveau 5 → 5 % 5 = 0
niveau 10 → 10 % 5 = 0
niveau 15 → 15 % 5 = 0
```

Cela permettra plus tard d'écrire :

```dart
if (niveau % 5 == 0) {
  print('Boss !');
}
```

---

## 03.15 — Les opérateurs d'affectation

Vous connaissez déjà :

```dart
int score = 100;
```

Le symbole :

```text
=
```

sert à affecter une valeur.

Attention :

```text
=
```

ne signifie pas :

> est égal à

dans une comparaison.

Il signifie :

> affecter une valeur.

---

## 03.16 — Modifier une variable

Supposons :

```dart
int score = 100;
```

Pour ajouter 50 :

```dart
score = score + 50;
```

Après cette instruction :

```text
score = 150
```

---

## 03.17 — Syntaxe raccourcie `+=`

Dart propose :

```dart
score += 50;
```

C'est équivalent à :

```dart
score = score + 50;
```

Exemple :

```dart
void main() {
  int score = 100;

  score += 50;

  print(score);
}
```

Résultat :

```text
150
```

---

## 03.18 — Opérateur `-=`

```dart
void main() {
  int energie = 100;

  energie -= 25;

  print(energie);
}
```

Résultat :

```text
75
```

Cela équivaut à :

```dart
energie = energie - 25;
```

---

## 03.19 — Opérateur `*=`

```dart
void main() {
  int score = 100;

  score *= 2;

  print(score);
}
```

Résultat :

```text
200
```

---

## 03.20 — Opérateur `/=`

```dart
void main() {
  double energie = 100;

  energie /= 2;

  print(energie);
}
```

Résultat :

```text
50.0
```

---

## 03.21 — Tableau récapitulatif

| Syntaxe courte | Équivalent |
|---|---|
| `x += 5` | `x = x + 5` |
| `x -= 5` | `x = x - 5` |
| `x *= 5` | `x = x * 5` |
| `x /= 5` | `x = x / 5` |

---

## 03.22 — Incrémentation avec `++`

Supposons :

```dart
int score = 10;
```

Pour ajouter 1 :

```dart
score = score + 1;
```

On peut également écrire :

```dart
score += 1;
```

Mais Dart propose encore plus court :

```dart
score++;
```

---

## 03.23 — Exemple avec `++`

```dart
void main() {
  int niveau = 1;

  niveau++;

  print(niveau);
}
```

Résultat :

```text
2
```

---

## 03.24 — Décrémentation avec `--`

`--` diminue une valeur de 1.

```dart
void main() {
  int vies = 3;

  vies--;

  print(vies);
}
```

Résultat :

```text
2
```

Dans un jeu, cette instruction sera extrêmement fréquente :

```dart
vies--;
```

---

## 03.25 — Pré-incrémentation et post-incrémentation

Dart permet :

```dart
score++;
```

et :

```dart
++score;
```

Si l'instruction est seule, le résultat final est généralement identique.

Mais une différence apparaît lorsqu'on utilise la valeur dans une expression.

---

## 03.26 — Post-incrémentation

```dart
void main() {
  int score = 10;

  print(score++);
  print(score);
}
```

Résultat :

```text
10
11
```

Pourquoi ?

```text
score++
```

signifie :

```text
utiliser la valeur actuelle
puis ajouter 1
```

---

## 03.27 — Pré-incrémentation

```dart
void main() {
  int score = 10;

  print(++score);
  print(score);
}
```

Résultat :

```text
11
11
```

Cette fois :

```text
++score
```

signifie :

```text
ajouter 1
puis utiliser la nouvelle valeur
```

Pour les débutants, il est préférable d'éviter de mettre `++` directement dans une expression compliquée.

Utilisez plutôt :

```dart
score++;
print(score);
```

---

## 03.28 — Les opérateurs de comparaison

Les comparaisons produisent toujours :

```text
true
```

ou :

```text
false
```

Les principaux opérateurs sont :

| Opérateur | Signification |
|---|---|
| `==` | égal à |
| `!=` | différent de |
| `>` | supérieur à |
| `<` | inférieur à |
| `>=` | supérieur ou égal |
| `<=` | inférieur ou égal |

---

## 03.29 — Égalité `==`

Exemple :

```dart
void main() {
  int age = 18;

  print(age == 18);
}
```

Résultat :

```text
true
```

---

## 03.30 — Attention à `=` et `==`

C'est une différence fondamentale.

```dart
age = 18;
```

signifie :

> donner la valeur 18 à `age`.

Alors que :

```dart
age == 18
```

signifie :

> vérifier si `age` est égal à 18.

Donc :

```text
= → affectation

== → comparaison
```

---

## 03.31 — Différent de `!=`

```dart
void main() {
  int niveau = 5;

  print(niveau != 10);
}
```

Résultat :

```text
true
```

Car :

```text
5 n'est pas égal à 10
```

---

## 03.32 — Supérieur à `>`

```dart
void main() {
  int score = 1000;

  print(score > 500);
}
```

Résultat :

```text
true
```

---

## 03.33 — Inférieur à `<`

```dart
void main() {
  int vies = 2;

  print(vies < 3);
}
```

Résultat :

```text
true
```

---

## 03.34 — Supérieur ou égal `>=`

```dart
void main() {
  int age = 18;

  print(age >= 18);
}
```

Résultat :

```text
true
```

Cela sera très utile pour écrire plus tard :

```dart
if (age >= 18) {
  print('Accès autorisé');
}
```

---

## 03.35 — Inférieur ou égal `<=`

```dart
void main() {
  int energie = 20;

  print(energie <= 25);
}
```

Résultat :

```text
true
```

---

## 03.36 — Stocker le résultat d'une comparaison

Une comparaison produit un booléen.

Nous pouvons donc faire :

```dart
void main() {
  int age = 25;

  bool majeur = age >= 18;

  print(majeur);
}
```

Résultat :

```text
true
```

---

## 03.37 — Exemple dans un jeu

```dart
void main() {
  int score = 1500;

  bool niveauDebloque = score >= 1000;

  print(niveauDebloque);
}
```

Résultat :

```text
true
```

---

## 03.38 — Les opérateurs logiques

Les opérateurs logiques permettent de combiner plusieurs conditions.

Les trois principaux sont :

```text
&&
||
!
```

---

## 03.39 — Opérateur `&&`

`&&` signifie :

> ET

Les deux conditions doivent être vraies.

Exemple :

```dart
void main() {
  int age = 20;
  bool compteActif = true;

  bool accesAutorise = age >= 18 && compteActif;

  print(accesAutorise);
}
```

Résultat :

```text
true
```

---

## 03.40 — Comprendre `&&`

Considérons :

```dart
age >= 18 && compteActif
```

Il faut :

```text
âge >= 18
ET
compte actif
```

pour obtenir :

```text
true
```

Table simplifiée :

| A | B | A && B |
|---|---|---|
| true | true | true |
| true | false | false |
| false | true | false |
| false | false | false |

---

## 03.41 — Exemple avec un jeu

Un joueur peut ouvrir une porte si :

```text
il possède la clé
ET
son niveau est supérieur ou égal à 5
```

Code :

```dart
void main() {
  bool possedeCle = true;
  int niveau = 7;

  bool peutOuvrir = possedeCle && niveau >= 5;

  print(peutOuvrir);
}
```

Résultat :

```text
true
```

---

## 03.42 — Opérateur `||`

`||` signifie :

> OU

Il suffit qu'au moins une condition soit vraie.

Exemple :

```dart
void main() {
  bool administrateur = false;
  bool moderateur = true;

  bool acces = administrateur || moderateur;

  print(acces);
}
```

Résultat :

```text
true
```

---

## 03.43 — Table de vérité de `||`

| A | B | A \|\| B |
|---|---|---|
| true | true | true |
| true | false | true |
| false | true | true |
| false | false | false |

---

## 03.44 — Exemple de jeu avec `||`

Un joueur peut gagner si :

```text
il atteint 10 000 points
OU
il récupère l'objet légendaire
```

```dart
void main() {
  int score = 8000;
  bool objetLegendaire = true;

  bool victoire = score >= 10000 || objetLegendaire;

  print(victoire);
}
```

Résultat :

```text
true
```

---

## 03.45 — Opérateur logique `!`

`!` signifie :

> NON

Il inverse une valeur booléenne.

Exemple :

```dart
void main() {
  bool jeuTermine = false;

  print(!jeuTermine);
}
```

Résultat :

```text
true
```

---

## 03.46 — Exemple concret

```dart
bool connecte = false;
```

Alors :

```dart
!connecte
```

donne :

```text
true
```

On pourra bientôt écrire :

```dart
if (!connecte) {
  print('Veuillez vous connecter.');
}
```

---

## 03.47 — Combiner plusieurs opérateurs logiques

Exemple :

```dart
void main() {
  int age = 20;
  bool compteActif = true;
  bool compteBloque = false;

  bool acces =
    age >= 18 &&
    compteActif &&
    !compteBloque;

  print(acces);
}
```

Pour obtenir `true`, il faut :

```text
âge >= 18
ET
compte actif
ET
compte non bloqué
```

---

## 03.48 — Exemple plus proche d'un jeu

```dart
void main() {
  int vies = 2;
  double energie = 30;
  bool pause = false;

  bool peutContinuer =
    vies > 0 &&
    energie > 0 &&
    !pause;

  print(peutContinuer);
}
```

---

## 03.49 — Les expressions

Une expression produit une valeur.

Exemples :

```dart
10 + 5
```

produit :

```text
15
```

---

```dart
age >= 18
```

produit :

```text
true
```

ou :

```text
false
```

---

```dart
score >= 1000 && vies > 0
```

produit également :

```text
true
```

ou :

```text
false
```

---

## 03.50 — Expression arithmétique

```dart
double total = prix * quantite;
```

La partie :

```dart
prix * quantite
```

est une expression.

---

## 03.51 — Expression booléenne

```dart
bool vivant = vies > 0;
```

La partie :

```dart
vies > 0
```

est une expression booléenne.

---

## 03.52 — Priorité des opérateurs

Considérons :

```dart
int resultat = 2 + 3 * 4;
```

Dart respecte les règles mathématiques.

La multiplication est effectuée avant l'addition.

Donc :

```text
3 × 4 = 12
2 + 12 = 14
```

Résultat :

```text
14
```

---

## 03.53 — Utiliser des parenthèses

Si nous voulons effectuer l'addition d'abord :

```dart
int resultat = (2 + 3) * 4;
```

Calcul :

```text
2 + 3 = 5
5 × 4 = 20
```

Résultat :

```text
20
```

---

## 03.54 — Pourquoi utiliser les parenthèses même lorsqu'elles ne sont pas obligatoires ?

Parce qu'elles améliorent souvent la lisibilité.

Exemple :

```dart
bool acces =
  (age >= 18) &&
  compteActif;
```

Même si certaines parenthèses ne sont pas toujours nécessaires, elles peuvent rendre l'intention plus claire.

---

## 03.55 — Ordre général simplifié

Pour ce cours, retenez approximativement :

```text
1. Parenthèses
2. Multiplication / division / modulo
3. Addition / soustraction
4. Comparaisons
5. &&
6. ||
7. Affectation
```

En cas de doute :

> utilisez des parenthèses.

---

## 03.56 — L'opérateur ternaire

Dart possède un opérateur très pratique :

```text
condition ? valeurSiVrai : valeurSiFaux
```

Exemple :

```dart
void main() {
  int age = 20;

  String resultat = age >= 18
    ? 'Majeur'
    : 'Mineur';

  print(resultat);
}
```

Résultat :

```text
Majeur
```

---

## 03.57 — Comprendre le ternaire

Structure :

```dart
condition
  ? valeurSiVrai
  : valeurSiFaux;
```

On peut le lire ainsi :

```text
Si condition vraie
→ première valeur

Sinon
→ deuxième valeur
```

---

## 03.58 — Exemple de jeu

```dart
void main() {
  int vies = 0;

  String statut = vies > 0
    ? 'En vie'
    : 'Game Over';

  print(statut);
}
```

Résultat :

```text
Game Over
```

---

## 03.59 — Pourquoi le ternaire est important avec Flutter ?

Vous verrez souvent des expressions du type :

```dart
Text(
  connecte ? 'Déconnexion' : 'Connexion',
  )
```

ou :

```dart
icon: favori
  ? Icons.favorite
  : Icons.favorite_border,
```

Nous n'utilisons pas encore Flutter, mais cette syntaxe sera très fréquente.

---

## 03.60 — Ne pas abuser du ternaire

Ceci reste lisible :

```dart
String message =
  score >= 1000 ? 'Bravo' : 'Continue';
```

Mais évitez des ternaires trop compliqués et imbriqués.

Dans ces cas, `if` sera souvent plus lisible.

---

## 03.61 — Exemple complet : statistiques d'un joueur

```dart
void main() {
  String joueur = 'Alex';

  int score = 1200;
  int vies = 3;
  double energie = 75.5;
  bool jeuEnPause = false;

  score += 500;
  vies--;
  energie -= 20;

  bool vivant = vies > 0;
  bool assezEnergie = energie > 0;
  bool peutJouer =
    vivant &&
    assezEnergie &&
    !jeuEnPause;

  String statut =
    peutJouer ? 'Peut continuer' : 'Partie terminée';

  print('========================');
  print('JOUEUR');
  print('========================');

  print('Nom : $joueur');
  print('Score : $score');
  print('Vies : $vies');
  print('Énergie : $energie');
  print('Peut jouer : $peutJouer');
  print('Statut : $statut');
}
```

---

## 03.62 — Exemple : système de points

```dart
void main() {
  int score = 500;

  print('Score initial : $score');

  score += 100;
  print('Bonus : $score');

  score += 250;
  print('Trésor récupéré : $score');

  score -= 50;
  print('Pénalité : $score');

  score *= 2;
  print('Multiplicateur x2 : $score');
}
```

---

## 03.63 — Exemple : système de vies

```dart
void main() {
  int vies = 3;

  print('Vies initiales : $vies');

  vies--;
  print('Après une collision : $vies');

  vies--;
  print('Après une deuxième collision : $vies');

  bool gameOver = vies <= 0;

  print('Game Over : $gameOver');
}
```

---

## 03.64 — Exemple : progression de niveau

```dart
void main() {
  int niveau = 4;
  int score = 1200;

  bool niveauSuivantDisponible =
    score >= 1000 && niveau == 4;

  print(niveauSuivantDisponible);
}
```

---

## 03.65 — Exemple : boss tous les cinq niveaux

```dart
void main() {
  int niveau = 10;

  bool niveauBoss = niveau % 5 == 0;

  print('Boss : $niveauBoss');
}
```

Résultat :

```text
Boss : true
```

---

## 03.66 — Fiche mémo et points à retenir

### FICHE MÉMO — OPÉRATEURS ARITHMÉTIQUES

```text
+ addition
- soustraction
* multiplication
/ division décimale
~/ division entière
% reste de division
```

Exemple :

```dart
int reste = 10 % 3;
```

---

### FICHE MÉMO — MODIFICATION DES VARIABLES

```dart
score += 100;
score -= 50;
score *= 2;
```

Pour ajouter ou retirer 1 :

```dart
score++;
vies--;
```

---

### FICHE MÉMO — COMPARAISONS

```text
== égal à

!= différent de

> supérieur à

< inférieur à

>= supérieur ou égal

<= inférieur ou égal
```

Exemple :

```dart
bool majeur = age >= 18;
```

---

### FICHE MÉMO — LOGIQUE

ET :

```dart
&&
```

Exemple :

```dart
bool acces =
  age >= 18 && compteActif;
```

OU :

```dart
||
```

Exemple :

```dart
bool victoire =
  score >= 10000 || bossBattu;
```

NON :

```dart
!
```

Exemple :

```dart
bool actif = !compteBloque;
```

---

### FICHE MÉMO — TERNAIRE

Structure :

```dart
condition
  ? valeurSiVrai
  : valeurSiFaux;
```

Exemple :

```dart
String statut =
  vies > 0
  ? 'En vie'
  : 'Game Over';
```

---

### À RETENIR ABSOLUMENT

Ne confondez jamais :

```text
=
```

et :

```text
==
```

`=` signifie :

```text
affecter
```

Exemple :

```dart
score = 100;
```

`==` signifie :

```text
comparer
```

Exemple :

```dart
score == 100
```

---

Les trois opérateurs logiques fondamentaux sont :

```text
&& → ET

|| → OU

! → NON
```

---

Pour un futur jeu Flutter, nous pourrons utiliser des expressions comme :

```dart
bool joueurVivant = vies > 0;
```

```dart
bool bossDisponible =
  niveau % 5 == 0;
```

```dart
bool peutJouer =
  vies > 0 &&
  energie > 0 &&
  !jeuEnPause;
```

Ces expressions constituent déjà une grande partie de la logique métier d'un jeu.

---

## 03.67 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| Confondre `=` et `==` | `=` affecte une valeur, `==` compare deux valeurs | Utiliser `==` dans une condition ou une expression booléenne, jamais `=` |
| Croire que `/` renvoie toujours un `int` | `/` renvoie systématiquement un `double`, même sur deux `int` | Utiliser `~/` si un résultat entier est attendu |
| Oublier que `~/` tronque au lieu d'arrondir | `~/` supprime la partie décimale, il n'arrondit pas au plus proche | Vérifier le comportement attendu avant de choisir `~/` |
| Confondre `%` avec une division | `%` renvoie le reste, pas le quotient | Utiliser `~/` pour le quotient et `%` pour le reste, souvent ensemble |
| Mettre `score++` dans une expression complexe | La différence entre pré- et post-incrémentation devient difficile à lire | Isoler `score++;` sur sa propre ligne, puis utiliser `score` séparément |
| Oublier des parenthèses dans une expression mixte | `&&` est évalué après les comparaisons, qui sont évaluées après les opérateurs arithmétiques | Ajouter des parenthèses pour clarifier l'intention, même quand ce n'est pas obligatoire |
| Utiliser `&&` alors qu'une seule condition suffit | `&&` exige que toutes les conditions soient vraies, alors que `||` n'en exige qu'une seule | Relire la règle métier : « ET » impose tout, « OU » impose au moins une condition |
| Imbriquer plusieurs opérateurs ternaires | Le code devient illisible et source d'erreurs | Préférer un `if` / `else` classique dès que le ternaire n'est plus trivial |

---

## 03.68 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| Opérateurs arithmétiques | `+ - * / ~/ %` effectuent les calculs de base ; `/` renvoie toujours un `double` |
| Division entière `~/` | Tronque la partie décimale, utile pour répartir des quantités entières |
| Modulo `%` | Donne le reste d'une division, utile pour tester la parité ou une périodicité (`niveau % 5 == 0`) |
| Affectation `=` | Donne une valeur à une variable ; ne compare jamais deux valeurs |
| Opérateurs raccourcis `+= -= *= /=` | Modifient une variable à partir de sa propre valeur, en une seule instruction |
| `++` et `--` | Ajoutent ou retirent 1 ; la position (avant ou après la variable) change le moment où la nouvelle valeur est utilisée |
| Comparaisons `== != > < >= <=` | Produisent toujours un booléen (`true` ou `false`) |
| Opérateurs logiques `&& \|\| !` | Combinent ou inversent des expressions booléennes (ET, OU, NON) |
| Expression | Tout morceau de code qui produit une valeur, arithmétique ou booléenne |
| Priorité des opérateurs | Parenthèses, puis `* / ~/ %`, puis `+ -`, puis comparaisons, puis `&&`, puis `\|\|` |
| Opérateur ternaire `? :` | Remplace un `if` / `else` simple par une seule expression |

---

## 03.69 — Exercices

### Exercice 1 — Calculs de base (facile)

Créez :

```dart
int a = 20;
int b = 6;
```

Affichez :

```text
a + b
a - b
a * b
a / b
a ~/ b
a % b
```

---

### Exercice 2 — Comprendre `/` et `~/` (facile)

Sans exécuter le programme, prévoyez les résultats :

```dart
print(15 / 4);
print(15 ~/ 4);
print(15 % 4);
```

Puis vérifiez avec Dart.

---

### Exercice 3 — Score (facile)

Créez :

```dart
int score = 100;
```

Puis :

- ajoutez 50 ;
- ajoutez 200 ;
- retirez 25 ;
- affichez le résultat final.

Utilisez uniquement :

```text
+=
-=
```

---

### Exercice 4 — Vies (facile)

Créez :

```dart
int vies = 5;
```

Utilisez `--` trois fois.

Affichez le nombre de vies restantes.

---

### Exercice 5 — Niveau (facile)

Créez :

```dart
int niveau = 1;
```

Utilisez `++` quatre fois.

Quel est le niveau final ?

---

### Exercice 6 — Comparaisons (facile)

Avec :

```dart
int score = 1500;
```

affichez les résultats de :

```text
score == 1500
score != 1500
score > 1000
score < 1000
score >= 1500
score <= 1499
```

---

### Exercice 7 — Autorisation (moyen)

Créez :

```dart
int age = 21;
bool compteActif = true;
```

Créez une variable :

```dart
bool accesAutorise;
```

La valeur doit être vraie uniquement si :

```text
âge >= 18
ET
compte actif
```

---

### Exercice 8 — Porte magique (moyen)

Un joueur possède :

```dart
bool possedeCle = true;
int niveau = 6;
```

La porte peut être ouverte uniquement si :

```text
le joueur possède la clé
ET
son niveau est supérieur ou égal à 5
```

Créez :

```dart
bool peutOuvrir;
```

---

### Exercice 9 — Condition OU (moyen)

Un joueur gagne si :

```text
score >= 10000
OU
bossBattu == true
```

Données :

```dart
int score = 8500;
bool bossBattu = true;
```

Créez :

```dart
bool victoire;
```

---

### Exercice 10 — Négation (facile)

Créez :

```dart
bool jeuTermine = false;
```

Créez :

```dart
bool jeuEnCours;
```

en utilisant uniquement :

```text
!
```

---

### Exercice 11 — Joueur actif (moyen)

Un joueur peut continuer si :

```text
vies > 0
ET
energie > 0
ET
jeuEnPause == false
```

Données :

```dart
int vies = 2;
double energie = 45;
bool jeuEnPause = false;
```

Créez :

```dart
bool peutContinuer;
```

---

### Exercice 12 — Nombre pair (facile)

Créez :

```dart
int nombre = 24;
```

Puis :

```dart
bool estPair;
```

Utilisez `%`.

Résultat attendu :

```text
true
```

---

### Exercice 13 — Nombre impair (facile)

Avec :

```dart
int nombre = 17;
```

déterminez si le nombre est impair.

Indice :

```dart
nombre % 2 != 0
```

---

### Exercice 14 — Boss (moyen)

Un boss apparaît tous les 5 niveaux.

Avec :

```dart
int niveau = 15;
```

créez :

```dart
bool bossDisponible;
```

---

### Exercice 15 — Ternaire simple (facile)

Avec :

```dart
int age = 16;
```

créez :

```dart
String statut;
```

qui doit contenir :

```text
Majeur
```

ou :

```text
Mineur
```

à l'aide d'un opérateur ternaire.

---

### Exercice 16 — Game Over (facile)

Avec :

```dart
int vies = 0;
```

utilisez un ternaire pour créer :

```dart
String message;
```

Résultat :

```text
Game Over
```

Si le joueur a encore des vies :

```text
Continue
```

---

### Exercice 17 — Énergie (moyen)

Créez :

```dart
double energie = 100;
```

Effectuez successivement :

```text
- 25
- 30
+ 10
```

en utilisant les opérateurs raccourcis.

Affichez l'énergie finale.

---

### Exercice 18 — Facture (moyen)

Données :

```dart
double prix = 49.99;
int quantite = 3;
double tauxTaxe = 0.15;
```

Calculez :

```text
sousTotal
taxe
total
```

---

### Exercice 19 — Jeu de pièces (difficile)

Un joueur possède :

```dart
int pieces = 47;
```

Chaque potion coûte :

```dart
int prixPotion = 10;
```

Calculez :

1. combien de potions complètes il peut acheter ;
2. combien de pièces restent.

Utilisez :

```text
~/
%
```

---

### Exercice 20 — Challenge complet (difficile)

Créez :

```dart
String joueur = 'Alex';
int score = 900;
int vies = 3;
double energie = 80;
bool possedeCle = true;
bool jeuEnPause = false;
```

Puis effectuez :

```text
+ 250 points
- 1 vie
- 35 énergie
```

Calculez ensuite :

```text
scoreSuffisant
joueurVivant
energieDisponible
peutJouer
peutOuvrirPorte
```

Règles :

```text
scoreSuffisant
→ score >= 1000

joueurVivant
→ vies > 0

energieDisponible
→ energie > 0

peutJouer
→ vivant ET énergie disponible ET jeu non en pause

peutOuvrirPorte
→ possède la clé ET score suffisant
```

Affichez toutes les informations.

---

## 03.70 — Corrections des exercices

### Correction 1

```dart
void main() {
  int a = 20;
  int b = 6;

  print(a + b);
  print(a - b);
  print(a * b);
  print(a / b);
  print(a ~/ b);
  print(a % b);
}
```

**Explication :** Cette correction applique les six opérateurs arithmétiques sur les mêmes deux opérandes, ce qui permet de comparer directement leurs résultats.

---

### Correction 2

```text
15 / 4
→ 3.75

15 ~/ 4
→ 3

15 % 4
→ 3
```

**Explication :** `/` retourne toujours un `double`, même quand le résultat est un nombre entier. `~/` tronque la partie décimale et `%` renvoie le reste de la division.

---

### Correction 3

```dart
void main() {
  int score = 100;

  score += 50;
  score += 200;
  score -= 25;

  print(score);
}
```

Résultat :

```text
325
```

**Explication :** Chaque `+=` et `-=` modifie directement `score`, sans avoir à réécrire `score = score + ...`.

---

### Correction 4

```dart
void main() {
  int vies = 5;

  vies--;
  vies--;
  vies--;

  print(vies);
}
```

Résultat :

```text
2
```

**Explication :** `vies--` retire 1 à chaque appel ; utilisé seul sur sa propre ligne, il n'y a pas de piège de pré/post-incrémentation.

---

### Correction 5

```dart
void main() {
  int niveau = 1;

  niveau++;
  niveau++;
  niveau++;
  niveau++;

  print(niveau);
}
```

Résultat :

```text
5
```

**Explication :** `niveau++` ajoute 1 à chaque appel, quatre fois de suite.

---

### Correction 6

```dart
void main() {
  int score = 1500;

  print(score == 1500);
  print(score != 1500);
  print(score > 1000);
  print(score < 1000);
  print(score >= 1500);
  print(score <= 1499);
}
```

Résultats :

```text
true
false
true
false
true
false
```

**Explication :** Chaque comparaison produit un booléen indépendant ; `score` lui-même n'est jamais modifié par ces opérateurs.

---

### Correction 7

```dart
void main() {
  int age = 21;
  bool compteActif = true;

  bool accesAutorise =
    age >= 18 && compteActif;

  print(accesAutorise);
}
```

**Explication :** `&&` exige que les deux conditions soient vraies : l'âge suffisant ET le compte actif.

---

### Correction 8

```dart
void main() {
  bool possedeCle = true;
  int niveau = 6;

  bool peutOuvrir =
    possedeCle && niveau >= 5;

  print(peutOuvrir);
}
```

**Explication :** Même logique que l'exercice précédent avec `&&`, appliquée cette fois à une clé possédée et un niveau minimum.

---

### Correction 9

```dart
void main() {
  int score = 8500;
  bool bossBattu = true;

  bool victoire =
    score >= 10000 || bossBattu;

  print(victoire);
}
```

**Explication :** `||` ne demande qu'une seule des deux conditions vraies pour valider la victoire.

---

### Correction 10

```dart
void main() {
  bool jeuTermine = false;

  bool jeuEnCours = !jeuTermine;

  print(jeuEnCours);
}
```

**Explication :** `!` inverse simplement la valeur booléenne de `jeuTermine`.

---

### Correction 11

```dart
void main() {
  int vies = 2;
  double energie = 45;
  bool jeuEnPause = false;

  bool peutContinuer =
    vies > 0 &&
    energie > 0 &&
    !jeuEnPause;

  print(peutContinuer);
}
```

**Explication :** Trois conditions sont combinées avec `&&`, dont une négation `!jeuEnPause`.

---

### Correction 12

```dart
void main() {
  int nombre = 24;

  bool estPair =
    nombre % 2 == 0;

  print(estPair);
}
```

**Explication :** Un nombre est pair lorsque le reste de sa division par 2 vaut 0.

---

### Correction 13

```dart
void main() {
  int nombre = 17;

  bool estImpair =
    nombre % 2 != 0;

  print(estImpair);
}
```

**Explication :** Un nombre est impair lorsque ce reste est différent de 0.

---

### Correction 14

```dart
void main() {
  int niveau = 15;

  bool bossDisponible =
    niveau % 5 == 0;

  print(bossDisponible);
}
```

**Explication :** Le même principe de modulo est réutilisé pour détecter les niveaux multiples de 5.

---

### Correction 15

```dart
void main() {
  int age = 16;

  String statut =
    age >= 18 ? 'Majeur' : 'Mineur';

  print(statut);
}
```

**Explication :** L'opérateur ternaire remplace un `if` / `else` par une seule expression.

---

### Correction 16

```dart
void main() {
  int vies = 0;

  String message =
    vies <= 0 ? 'Game Over' : 'Continue';

  print(message);
}
```

**Explication :** Le ternaire choisit le message affiché en fonction de la comparaison `vies <= 0`.

---

### Correction 17

```dart
void main() {
  double energie = 100;

  energie -= 25;
  energie -= 30;
  energie += 10;

  print(energie);
}
```

Résultat :

```text
55.0
```

**Explication :** Les opérateurs `-=` et `+=` s'enchaînent pour faire évoluer l'énergie étape par étape.

---

### Correction 18

```dart
void main() {
  double prix = 49.99;
  int quantite = 3;
  double tauxTaxe = 0.15;

  double sousTotal =
    prix * quantite;

  double taxe =
    sousTotal * tauxTaxe;

  double total =
    sousTotal + taxe;

  print('Sous-total : $sousTotal');
  print('Taxe : $taxe');
  print('Total : $total');
}
```

**Explication :** `sousTotal`, `taxe` et `total` sont calculés successivement, chacun réutilisant le résultat précédent.

---

### Correction 19

```dart
void main() {
  int pieces = 47;
  int prixPotion = 10;

  int nombrePotions =
    pieces ~/ prixPotion;

  int piecesRestantes =
    pieces % prixPotion;

  print('Potions : $nombrePotions');
  print('Pièces restantes : $piecesRestantes');
}
```

Résultat :

```text
Potions : 4
Pièces restantes : 7
```

**Explication :** `~/` donne le nombre de potions achetables, `%` donne les pièces restantes : les deux résultats sont complémentaires.

---

### Correction 20

```dart
void main() {
  String joueur = 'Alex';
  int score = 900;
  int vies = 3;
  double energie = 80;
  bool possedeCle = true;
  bool jeuEnPause = false;

  score += 250;
  vies--;
  energie -= 35;

  bool scoreSuffisant =
    score >= 1000;

  bool joueurVivant =
    vies > 0;

  bool energieDisponible =
    energie > 0;

  bool peutJouer =
    joueurVivant &&
    energieDisponible &&
    !jeuEnPause;

  bool peutOuvrirPorte =
    possedeCle &&
    scoreSuffisant;

  print('============================');
  print('ÉTAT DU JOUEUR');
  print('============================');

  print('Joueur : $joueur');
  print('Score : $score');
  print('Vies : $vies');
  print('Énergie : $energie');

  print('Score suffisant : $scoreSuffisant');
  print('Joueur vivant : $joueurVivant');
  print('Énergie disponible : $energieDisponible');
  print('Peut jouer : $peutJouer');
  print('Peut ouvrir la porte : $peutOuvrirPorte');
}
```

**Explication :** Cette correction combine `+=`, `--`, `-=`, plusieurs comparaisons, `&&` et l'affichage de ces booléens : elle résume tout le chapitre.

---

## Et maintenant ?

Vous savez désormais calculer, comparer et combiner des expressions booléennes. Le chapitre suivant vous apprend à faire réagir votre programme à ces expressions grâce aux structures de décision `if`, `else if`, `else` et `switch`.

Direction le chapitre suivant : [04-PARTIE-1A—CONDITIONS-IF-ELSE-SWITCH.md](./04-PARTIE-1A—CONDITIONS-IF-ELSE-SWITCH.md)
