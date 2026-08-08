# PARTIE 1A — DART
# CHAPITRE 02 — VARIABLES, CONSTANTES ET TYPES

> **Niveau :** débutant
> **Durée estimée :** 3 h
> **Pré-requis :** chapitre 01 — Apprendre Dart de zéro
> **Ce que vous saurez faire à la fin :** déclarer, typer, modifier et afficher des variables Dart (`String`, `int`, `double`, `bool`, `num`), choisir entre `var`, `dynamic`, `final`, `const` et `late`, utiliser l'interpolation de chaînes et convertir entre types.

---

## 02.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- créer une variable ;
- stocker du texte ;
- stocker des nombres entiers ;
- stocker des nombres décimaux ;
- utiliser des valeurs booléennes ;
- comprendre la différence entre `var` et un type explicite ;
- comprendre `dynamic` ;
- utiliser `final` ;
- utiliser `const` ;
- comprendre `late` ;
- modifier la valeur d'une variable ;
- utiliser l'interpolation de chaînes ;
- effectuer des conversions simples entre les types ;
- utiliser correctement les conventions de nommage Dart.

---

## 02.1 — Qu'est-ce qu'une variable ?

Une variable est un emplacement permettant de stocker une information.

Imaginez une boîte :

```text
+-------------------+
| nom               |
|-------------------|
| "Alex"            |
+-------------------+
```

Le nom de la boîte est :

```text
nom
```

La valeur stockée est :

```text
Alex
```

En Dart :

```dart
String nom = 'Alex';
```

On peut lire cette instruction ainsi :

> Créer une variable appelée `nom`, capable de stocker une chaîne de caractères, et lui donner la valeur `Alex`.

---

## 02.2 — Premier exemple

```dart
void main() {
  String nom = 'Alex';

  print(nom);
}
```

Résultat :

```text
Alex
```

Attention :

```dart
print('nom');
```

affiche :

```text
nom
```

alors que :

```dart
print(nom);
```

affiche le contenu de la variable.

Donc :

```text
'nom' = texte littéral

nom   = variable
```

---

## 02.3 — Pourquoi utiliser des variables ?

Considérons :

```dart
void main() {
  print('Alex');
  print('Bonjour Alex');
  print('Alex apprend Dart');
  print('Alex va apprendre Flutter');
}
```

Si le nom change, il faut modifier plusieurs lignes.

Avec une variable :

```dart
void main() {
  String nom = 'Alex';

  print(nom);
  print('Bonjour $nom');
  print('$nom apprend Dart');
  print('$nom va apprendre Flutter');
}
```

Maintenant, il suffit de modifier :

```dart
String nom = 'Sophie';
```

Le reste du programme s'adapte automatiquement.

---

## 02.4 — Les principaux types de base

Nous allons commencer avec quatre types essentiels :

```text
String
int
double
bool
```

---

## 02.5 — Le type `String`

`String` permet de stocker du texte.

Exemple :

```dart
void main() {
  String nom = 'Alex';
  String ville = 'Montréal';
  String langage = 'Dart';

  print(nom);
  print(ville);
  print(langage);
}
```

On peut utiliser :

```dart
String message = 'Bonjour';
```

ou :

```dart
String message = "Bonjour";
```

Les deux syntaxes sont valides.

---

## 02.6 — Le type `int`

`int` signifie integer.

Il permet de stocker des nombres entiers.

Exemples :

```dart
int age = 25;
int quantite = 10;
int score = 1500;
int temperature = -5;
```

Programme :

```dart
void main() {
  int age = 25;

  print(age);
}
```

Résultat :

```text
25
```

---

## 02.7 — Un entier peut être négatif

```dart
void main() {
  int temperature = -15;

  print(temperature);
}
```

Résultat :

```text
-15
```

---

## 02.8 — Le type `double`

`double` permet de stocker des nombres décimaux.

Exemple :

```dart
double prix = 19.99;
```

Autres exemples :

```dart
double taille = 1.75;
double moyenne = 87.5;
double temperature = -3.5;
```

Programme :

```dart
void main() {
  double prix = 24.99;

  print(prix);
}
```

Résultat :

```text
24.99
```

---

## 02.9 — Différence entre `int` et `double`

```text
int
--------------------------------
10
25
100
-5
0
```

```text
double
--------------------------------
10.5
25.99
3.14
-7.2
```

Donc :

```dart
int age = 25;
```

mais :

```dart
double prix = 25.99;
```

---

## 02.10 — Attention au séparateur décimal

En programmation, on utilise :

```text
.
```

et non :

```text
,
```

Correct :

```dart
double prix = 19.99;
```

Incorrect :

```dart
double prix = 19,99;
```

---

## 02.11 — Le type `bool`

`bool` signifie booléen.

Une variable booléenne ne peut avoir que deux valeurs :

```text
true
false
```

Exemple :

```dart
bool disponible = true;
```

ou :

```dart
bool connecte = false;
```

Programme :

```dart
void main() {
  bool disponible = true;

  print(disponible);
}
```

Résultat :

```text
true
```

---

## 02.12 — Pourquoi les booléens sont importants ?

Plus tard, dans Flutter, nous pourrons avoir :

```dart
bool modeSombre = true;
bool utilisateurConnecte = false;
bool chargement = true;
bool jeuTermine = false;
```

Ces variables permettent de représenter l'état d'une application.

Par exemple :

```text
utilisateurConnecte = true
        |
        v
Afficher l'espace utilisateur
```

Sinon :

```text
utilisateurConnecte = false
        |
        v
Afficher l'écran de connexion
```

---

## 02.13 — Exemple complet avec plusieurs types

```dart
void main() {
  String nom = 'Alex';
  int age = 25;
  double taille = 1.78;
  bool developpeur = true;

  print(nom);
  print(age);
  print(taille);
  print(developpeur);
}
```

Résultat :

```text
Alex
25
1.78
true
```

---

## 02.14 — Modifier une variable

Une variable peut généralement changer de valeur.

```dart
void main() {
  int score = 10;

  print(score);

  score = 20;

  print(score);
}
```

Résultat :

```text
10
20
```

---

## 02.15 — Attention à ne pas redéclarer la variable

Correct :

```dart
int score = 10;

score = 20;
```

La première instruction crée la variable.

La deuxième change sa valeur.

Il ne faut pas écrire inutilement :

```dart
int score = 10;

int score = 20;
```

Dans le même bloc, cela provoquera une erreur car `score` existe déjà.

---

## 02.16 — Une variable doit respecter son type

Considérons :

```dart
int age = 25;
```

La variable `age` est un entier.

On peut ensuite écrire :

```dart
age = 30;
```

Mais on ne peut pas écrire :

```dart
age = 'trente';
```

Pourquoi ?

Parce que :

```text
age
 |
 +--> type int
```

alors que :

```text
'trente'
 |
 +--> type String
```

Les deux types sont incompatibles.

---

## 02.17 — Dart est un langage typé

Lorsqu'on écrit :

```dart
String nom = 'Alex';
```

Dart connaît le type de la variable.

Il peut donc détecter certaines erreurs avant même l'exécution.

Exemple :

```dart
String nom = 'Alex';

nom = 123;
```

Cela n'est pas valide.

---

## 02.18 — Le mot-clé `var`

Dart peut souvent déterminer automatiquement le type.

Au lieu d'écrire :

```dart
String nom = 'Alex';
```

on peut écrire :

```dart
var nom = 'Alex';
```

Dart comprend que :

```text
'Alex'
```

est une chaîne de caractères.

Il déduit donc :

```text
nom → String
```

---

## 02.19 — Autres exemples avec `var`

```dart
void main() {
  var nom = 'Alex';
  var age = 25;
  var prix = 19.99;
  var disponible = true;

  print(nom);
  print(age);
  print(prix);
  print(disponible);
}
```

Dart déduit :

```text
nom        → String
age        → int
prix       → double
disponible → bool
```

---

## 02.20 — `var` ne signifie pas "n'importe quel type"

C'est une erreur fréquente.

Considérons :

```dart
var age = 25;
```

Dart déduit que `age` est un `int`.

Donc ceci est valide :

```dart
age = 30;
```

Mais ceci ne l'est pas :

```dart
age = 'trente';
```

Une fois le type déduit, il reste fixé.

---

## 02.21 — Type explicite ou `var` ?

Ces deux lignes sont correctes :

```dart
String nom = 'Alex';
```

et :

```dart
var nom = 'Alex';
```

Pour l'apprentissage, nous utiliserons souvent le type explicite afin de voir clairement les types.

Exemple :

```dart
String nom = 'Alex';
int age = 25;
double prix = 19.99;
bool actif = true;
```

Puis nous utiliserons progressivement davantage `var`.

---

## 02.22 — Le type `num`

Dart possède également le type :

```dart
num
```

`num` peut représenter un entier ou un nombre décimal.

Exemple :

```dart
void main() {
  num valeur = 10;

  print(valeur);

  valeur = 12.5;

  print(valeur);
}
```

Résultat :

```text
10
12.5
```

Pourquoi ?

Parce que :

```text
num
 |
 +--> int
 |
 +--> double
```

---

## 02.23 — Quand utiliser `num` ?

On peut utiliser `num` lorsqu'une valeur peut être entière ou décimale.

Exemple :

```dart
num temperature = 20;

temperature = 20.5;
```

Cependant, dans la majorité des situations, il est préférable d'être plus précis avec :

```dart
int
```

ou :

```dart
double
```

---

## 02.24 — Le type `dynamic`

Dart propose également :

```dart
dynamic
```

Exemple :

```dart
void main() {
  dynamic valeur = 'Bonjour';

  print(valeur);

  valeur = 100;

  print(valeur);

  valeur = true;

  print(valeur);
}
```

Résultat :

```text
Bonjour
100
true
```

---

## 02.25 — Différence entre `var` et `dynamic`

C'est très important.

### Avec `var`

```dart
var valeur = 10;
```

Dart déduit :

```text
int
```

Donc :

```dart
valeur = 20;
```

est valide.

Mais :

```dart
valeur = 'Bonjour';
```

ne l'est pas.

---

### Avec `dynamic`

```dart
dynamic valeur = 10;
```

Puis :

```dart
valeur = 'Bonjour';
```

est autorisé.

Puis :

```dart
valeur = true;
```

est également autorisé.

---

## 02.26 — Pourquoi éviter `dynamic` lorsque possible ?

`dynamic` retire une partie de la sécurité offerte par le typage.

Par exemple :

```dart
dynamic valeur = 'Bonjour';
```

Puis :

```dart
valeur = 100;
```

Puis :

```dart
valeur = true;
```

Le programme accepte ces changements.

Cela peut rendre les erreurs plus difficiles à détecter.

Donc :

```text
Type explicite → recommandé
var            → recommandé lorsque le type est évident
dynamic        → seulement lorsque nécessaire
```

---

## 02.27 — Interpolation de chaînes

Supposons :

```dart
String nom = 'Alex';
```

Nous voulons afficher :

```text
Bonjour Alex
```

On pourrait écrire :

```dart
print('Bonjour ' + nom);
```

Mais Dart propose une syntaxe plus propre :

```dart
print('Bonjour $nom');
```

C'est ce qu'on appelle :

> l'interpolation de chaînes.

---

## 02.28 — Premier exemple d'interpolation

```dart
void main() {
  String nom = 'Alex';

  print('Bonjour $nom');
}
```

Résultat :

```text
Bonjour Alex
```

---

## 02.29 — Plusieurs variables dans une chaîne

```dart
void main() {
  String nom = 'Alex';
  int age = 25;
  String ville = 'Montréal';

  print('Je m’appelle $nom.');
  print('J’ai $age ans.');
  print('J’habite à $ville.');
}
```

---

## 02.30 — Utiliser `${ }`

Lorsque l'expression est plus complexe, on utilise :

```dart
${}
```

Exemple :

```dart
void main() {
  int age = 25;

  print('L’année prochaine, vous aurez ${age + 1} ans.');
}
```

Résultat :

```text
L’année prochaine, vous aurez 26 ans.
```

---

## 02.31 — `$variable` ou `${expression}` ?

Pour une variable simple :

```dart
print('Bonjour $nom');
```

Pour une expression :

```dart
print('Total : ${prix * quantite}');
```

---

## 02.32 — Exemple avec calcul intégré

```dart
void main() {
  double prix = 15.0;
  int quantite = 3;

  print('Prix : $prix');
  print('Quantité : $quantite');
  print('Total : ${prix * quantite}');
}
```

Résultat :

```text
Prix : 15.0
Quantité : 3
Total : 45.0
```

---

## 02.33 — Concaténation avec `+`

Il est également possible d'assembler des chaînes avec :

```text
+
```

Exemple :

```dart
void main() {
  String prenom = 'Alex';
  String nom = 'Martin';

  print(prenom + ' ' + nom);
}
```

Résultat :

```text
Alex Martin
```

Cependant, l'interpolation est souvent plus lisible :

```dart
print('$prenom $nom');
```

---

## 02.34 — Le mot-clé `final`

`final` permet de créer une valeur qui ne pourra être assignée qu'une seule fois.

Exemple :

```dart
final String nom = 'Alex';
```

Puis :

```dart
nom = 'Sophie';
```

n'est pas autorisé.

---

## 02.35 — Exemple avec `final`

```dart
void main() {
  final String nom = 'Alex';

  print(nom);
}
```

Après initialisation :

```text
nom
 |
 +--> Alex
      valeur non réassignable
```

---

## 02.36 — Dart peut aussi déduire le type avec `final`

On peut écrire :

```dart
final nom = 'Alex';
```

Dart comprend que `nom` est une chaîne de caractères.

Autre exemple :

```dart
final age = 25;
```

---

## 02.37 — Pourquoi `final` est important en Flutter ?

Vous verrez extrêmement souvent :

```dart
final String title;
```

ou :

```dart
final int score;
```

dans les classes Flutter.

Exemple futur :

```dart
class Produit {
  final String nom;
  final double prix;
}
```

Cela signifie qu'une fois la propriété initialisée, elle ne doit pas être réassignée.

---

## 02.38 — Le mot-clé `const`

`const` permet également de créer une constante.

Exemple :

```dart
const double pi = 3.14159;
```

Il est impossible ensuite de faire :

```dart
pi = 4;
```

---

## 02.39 — Différence fondamentale entre `final` et `const`

Les deux empêchent la réassignation.

Mais il existe une différence importante.

`const` doit être connu au moment de la compilation.

Exemple :

```dart
const double pi = 3.14159;
```

La valeur est connue immédiatement.

---

Avec `final`, la valeur peut être déterminée pendant l'exécution.

Exemple :

```dart
final date = DateTime.now();
```

La date actuelle n'est pas connue au moment où le programme est compilé.

Elle est obtenue au moment de l'exécution.

---

## 02.40 — Exemple incorrect avec `const`

Ceci pose problème :

```dart
const date = DateTime.now();
```

Pourquoi ?

Parce que :

```text
DateTime.now()
```

dépend du moment où le programme s'exécute.

---

## 02.41 — Exemple correct avec `final`

```dart
void main() {
  final date = DateTime.now();

  print(date);
}
```

---

## 02.42 — Résumé `final` vs `const`

| Élément | `final` | `const` |
|---|---|---|
| Réassignable | Non | Non |
| Valeur connue à la compilation | Pas obligatoire | Oui |
| Peut utiliser `DateTime.now()` | Oui | Non |
| Très utilisé avec Flutter | Oui | Oui |

---

## 02.43 — Exemple pratique

```dart
void main() {
  const String application = 'MonApp';

  final DateTime lancement = DateTime.now();

  print(application);
  print(lancement);
}
```

---

## 02.44 — Pourquoi Flutter utilise beaucoup `const`

Plus tard, vous verrez :

```dart
const Text('Bonjour')
```

ou :

```dart
const Icon(Icons.home)
```

ou encore :

```dart
const SizedBox(height: 20)
```

Lorsque Flutter sait qu'un objet ne changera jamais, `const` peut éviter certaines créations inutiles d'objets.

Nous approfondirons cela dans la Partie Flutter.

---

## 02.45 — Le mot-clé `late`

`late` signifie qu'une variable sera initialisée plus tard.

Exemple :

```dart
late String nom;
```

La variable existe, mais elle n'a pas encore reçu de valeur.

Plus tard :

```dart
nom = 'Alex';
```

---

## 02.46 — Exemple complet avec `late`

```dart
void main() {
  late String message;

  message = 'Bonjour Dart';

  print(message);
}
```

Résultat :

```text
Bonjour Dart
```

---

## 02.47 — Pourquoi utiliser `late` ?

Parfois, une variable doit être déclarée avant que sa valeur soit disponible.

Exemple conceptuel :

```dart
late String utilisateur;
```

Puis plus tard :

```dart
utilisateur = 'Alex';
```

Vous rencontrerez parfois `late` avec Flutter, notamment avec des contrôleurs ou certaines initialisations.

---

## 02.48 — Attention avec `late`

Ceci est dangereux :

```dart
void main() {
  late String nom;

  print(nom);
}
```

La variable n'a jamais été initialisée.

Le programme provoquera une erreur au moment de son utilisation.

Donc :

```text
late = je promets que je donnerai une valeur avant utilisation
```

---

## 02.49 — Plusieurs variables du même type

On peut écrire :

```dart
String prenom = 'Alex';
String nom = 'Martin';
String ville = 'Montréal';
```

C'est souvent plus lisible.

Dart autorise également :

```dart
String prenom = 'Alex', nom = 'Martin', ville = 'Montréal';
```

Mais pour l'apprentissage et la lisibilité, nous privilégierons une variable par ligne.

---

## 02.50 — Les noms de variables

Un bon nom de variable doit expliquer ce qu'elle contient.

Mauvais :

```dart
int x = 25;
```

Mieux :

```dart
int age = 25;
```

Mauvais :

```dart
double p = 19.99;
```

Mieux :

```dart
double prix = 19.99;
```

---

## 02.51 — Convention `lowerCamelCase`

En Dart, les variables utilisent généralement :

```text
lowerCamelCase
```

Exemples :

```dart
String nomUtilisateur = 'Alex';

double prixTotal = 99.99;

bool utilisateurConnecte = true;

int nombreDeVies = 3;
```

Le premier mot commence par une minuscule.

Chaque mot suivant commence par une majuscule.

---

## 02.52 — Exemples à éviter

Évitez :

```dart
String NomUtilisateur = 'Alex';
```

Évitez :

```dart
String nom_utilisateur = 'Alex';
```

Évitez :

```dart
String NOMUTILISATEUR = 'Alex';
```

Pour une variable classique, préférez :

```dart
String nomUtilisateur = 'Alex';
```

---

## 02.53 — Les noms ne peuvent pas contenir d'espaces

Incorrect :

```dart
String nom utilisateur = 'Alex';
```

Correct :

```dart
String nomUtilisateur = 'Alex';
```

---

## 02.54 — Le nom ne doit pas commencer par un chiffre

Incorrect :

```dart
String 1nom = 'Alex';
```

Correct :

```dart
String nom1 = 'Alex';
```

Mais dans un vrai projet, choisissez un nom plus descriptif.

---

## 02.55 — Faire des calculs avec des variables

```dart
void main() {
  int a = 10;
  int b = 5;

  print(a + b);
}
```

Résultat :

```text
15
```

---

## 02.56 — Stocker le résultat d'un calcul

```dart
void main() {
  int a = 10;
  int b = 5;

  int resultat = a + b;

  print(resultat);
}
```

---

## 02.57 — Exemple de calcul commercial

```dart
void main() {
  double prix = 20.0;
  int quantite = 4;

  double total = prix * quantite;

  print('Prix unitaire : $prix');
  print('Quantité : $quantite');
  print('Total : $total');
}
```

Résultat :

```text
Prix unitaire : 20.0
Quantité : 4
Total : 80.0
```

---

## 02.58 — Utiliser une variable pour modifier une autre

```dart
void main() {
  int score = 100;

  score = score + 50;

  print(score);
}
```

Résultat :

```text
150
```

Nous verrons bientôt une syntaxe plus courte :

```dart
score += 50;
```

dans le chapitre consacré aux opérateurs.

---

## 02.59 — Conversion d'un entier en texte

Supposons :

```dart
int age = 25;
```

On peut obtenir une chaîne avec :

```dart
String texte = age.toString();
```

Exemple :

```dart
void main() {
  int age = 25;

  String texte = age.toString();

  print(texte);
}
```

---

## 02.60 — Conversion de texte en entier

Supposons que nous recevions :

```dart
String ageTexte = '25';
```

Pour obtenir un entier :

```dart
int age = int.parse(ageTexte);
```

Programme :

```dart
void main() {
  String ageTexte = '25';

  int age = int.parse(ageTexte);

  print(age + 5);
}
```

Résultat :

```text
30
```

---

## 02.61 — Conversion vers `double`

```dart
void main() {
  String prixTexte = '19.99';

  double prix = double.parse(prixTexte);

  print(prix);
}
```

---

## 02.62 — Pourquoi les conversions seront importantes avec Flutter ?

Imaginez un champ de texte Flutter.

L'utilisateur saisit :

```text
25
```

Pour Flutter, ce contenu sera généralement récupéré sous forme de texte :

```dart
String
```

Mais pour effectuer :

```text
25 + 10
```

il faudra convertir cette chaîne en nombre.

Nous pourrons donc utiliser :

```dart
int.parse()
```

ou :

```dart
double.parse()
```

---

## 02.63 — Exemple complet

```dart
void main() {
  String produit = 'Ordinateur';
  double prix = 999.99;
  int quantite = 2;
  bool disponible = true;

  double total = prix * quantite;

  print('Produit : $produit');
  print('Prix : $prix \$');
  print('Quantité : $quantite');
  print('Disponible : $disponible');
  print('Total : $total \$');
}
```

---

## 02.64 — Premier mini-programme : profil utilisateur

```dart
void main() {
  String nom = 'Alex';
  int age = 28;
  String ville = 'Montréal';
  bool actif = true;

  print('========================');
  print('PROFIL UTILISATEUR');
  print('========================');

  print('Nom : $nom');
  print('Âge : $age');
  print('Ville : $ville');
  print('Compte actif : $actif');
}
```

---

## 02.65 — Deuxième mini-programme : boutique

```dart
void main() {
  String produit = 'Clavier';
  double prix = 79.99;
  int quantite = 3;

  double total = prix * quantite;

  print('========================');
  print('COMMANDE');
  print('========================');

  print('Produit : $produit');
  print('Prix unitaire : $prix');
  print('Quantité : $quantite');
  print('Total : $total');
}
```

---

## 02.66 — Fiche mémo : les types Dart

| Type | Utilisation | Exemple |
|---|---|---|
| `String` | Texte | `'Bonjour'` |
| `int` | Entier | `25` |
| `double` | Décimal | `19.99` |
| `num` | Entier ou décimal | `10`, `10.5` |
| `bool` | Vrai/Faux | `true` |
| `var` | Type déduit automatiquement | `var age = 25` |
| `dynamic` | Type pouvant changer | `dynamic valeur` |

---

## 02.67 — Fiche mémo : variables et constantes

Variable modifiable :

```dart
int score = 100;

score = 200;
```

Type déduit :

```dart
var score = 100;
```

Valeur assignée une seule fois :

```dart
final date = DateTime.now();
```

Constante connue à la compilation :

```dart
const double pi = 3.14159;
```

Initialisation différée :

```dart
late String nom;

nom = 'Alex';
```

---

## 02.68 — Fiche mémo : interpolation et conversions

Variable simple :

```dart
print('Bonjour $nom');
```

Expression :

```dart
print('Total : ${prix * quantite}');
```

---

`String` vers `int` :

```dart
int valeur = int.parse('25');
```

`String` vers `double` :

```dart
double valeur = double.parse('19.99');
```

Nombre vers `String` :

```dart
String texte = 25.toString();
```

---

## 02.69 — À retenir absolument

```text
String → texte

int → entier

double → nombre décimal

bool → true ou false
```

```text
var
→ Dart déduit le type.
```

```text
dynamic
→ le type peut changer.
→ à utiliser avec prudence.
```

```text
final
→ valeur assignée une seule fois.
→ valeur possiblement connue pendant l'exécution.
```

```text
const
→ constante connue à la compilation.
```

```text
late
→ la valeur sera initialisée plus tard.
```

Et surtout :

```dart
String nom = 'Alex';

print('Bonjour $nom');
```

Cette syntaxe sera omniprésente lorsque nous commencerons Flutter.

---

## 02.70 — Erreurs fréquentes

| Erreur | Cause | Correction |
|---|---|---|
| Redéclarer une variable existante (`int score = 10; int score = 20;`) | Le mot-clé de type est répété alors que la variable existe déjà dans le même bloc | Réassigner sans redéclarer : `score = 20;` |
| Changer le type d'une variable typée (`age = 'trente';` sur un `int age`) | Dart est un langage statiquement typé : une fois le type fixé, il ne change plus | Convertir explicitement la valeur ou déclarer une nouvelle variable du bon type |
| Confondre `var` et `dynamic` | `var` fige le type déduit dès la première valeur ; `dynamic` l'autorise à changer ensuite | N'utiliser `dynamic` que si la variable doit vraiment changer de type au fil du programme |
| Utiliser une virgule comme séparateur décimal (`double prix = 19,99;`) | Dart, comme la majorité des langages, utilise le point `.` comme séparateur décimal | Écrire `double prix = 19.99;` |
| Déclarer `const` avec une valeur connue seulement à l'exécution (`const date = DateTime.now();`) | `const` exige une valeur connue au moment de la compilation | Utiliser `final date = DateTime.now();` |
| Utiliser une variable `late` avant de l'initialiser | `late` promet une initialisation future, pas immédiate | Toujours assigner une valeur à la variable avant son premier usage |
| Nom de variable avec espace, majuscule initiale ou `snake_case` | Ne respecte pas la convention `lowerCamelCase` de Dart | Écrire `nomUtilisateur`, jamais `Nom Utilisateur` ni `nom_utilisateur` |
| Additionner un `String` et un nombre sans conversion (`'25' + 10`) | Un texte saisi (formulaire Flutter, entrée utilisateur) reste un `String` tant qu'il n'est pas converti | Utiliser `int.parse()` ou `double.parse()` avant tout calcul |

---

## 02.71 — Résumé du chapitre

| Notion | À retenir |
|---|---|
| Variable | Emplacement nommé qui stocke une valeur, généralement modifiable |
| `String` | Stocke du texte, entre guillemets simples ou doubles |
| `int` | Stocke un nombre entier, positif ou négatif |
| `double` | Stocke un nombre décimal, avec un point comme séparateur |
| `bool` | Stocke uniquement `true` ou `false` |
| `num` | Superclasse commune à `int` et `double` : accepte les deux |
| `var` | Le type est déduit une seule fois à partir de la valeur, puis reste fixé |
| `dynamic` | Le type peut changer à chaque réassignation ; à utiliser avec prudence |
| `final` | Valeur assignée une seule fois ; peut être connue seulement à l'exécution |
| `const` | Constante obligatoirement connue à la compilation |
| `late` | Initialisation différée : la valeur doit être fournie avant tout usage |
| Interpolation `$variable` | Insère la valeur d'une variable simple dans une chaîne |
| Interpolation `${expression}` | Insère le résultat d'un calcul ou d'une expression dans une chaîne |
| Conversion en nombre | `int.parse()` et `double.parse()` transforment un `String` en nombre |
| Conversion en texte | `.toString()` transforme un nombre en `String` |
| `lowerCamelCase` | Convention de nommage : première lettre minuscule, puis une majuscule par mot |

---

## 02.72 — Exercices

### Exercice 1 — Créer des variables (facile)

Créez les variables suivantes :

```text
nom = Alex
age = 25
taille = 1.80
actif = true
```

Utilisez les types explicites appropriés.

---

### Exercice 2 — Afficher un profil (facile)

Avec les variables :

```dart
String nom = 'Sophie';
int age = 30;
String ville = 'Québec';
```

affichez :

```text
Nom : Sophie
Âge : 30
Ville : Québec
```

Utilisez l'interpolation.

---

### Exercice 3 — Modifier une variable (facile)

Créez :

```dart
int score = 100;
```

Affichez le score.

Modifiez ensuite sa valeur à :

```text
200
```

puis affichez le nouveau score.

---

### Exercice 4 — Identifier les types (facile)

Indiquez le type le plus approprié :

```text
"Bonjour"
125
12.5
true
-15
"Montréal"
false
3.14159
```

Choisissez parmi :

```text
String
int
double
bool
```

---

### Exercice 5 — `var` (facile)

Réécrivez :

```dart
String nom = 'Alex';
int age = 25;
double prix = 99.99;
bool actif = true;
```

en utilisant `var`.

---

### Exercice 6 — Comprendre l'inférence (moyen)

Que se passe-t-il ici ?

```dart
void main() {
  var score = 100;

  score = 'cent';

  print(score);
}
```

Expliquez pourquoi.

---

### Exercice 7 — `dynamic` (moyen)

Créez une variable :

```dart
dynamic valeur
```

Donnez-lui successivement :

```text
Bonjour
100
true
19.99
```

Affichez sa valeur après chaque changement.

---

### Exercice 8 — `final` (moyen)

Créez :

```dart
final String application = 'GameApp';
```

Essayez ensuite de modifier sa valeur.

Expliquez le résultat.

---

### Exercice 9 — `const` (facile)

Créez les constantes :

```text
pi = 3.14159
nombreDeVies = 3
nomDuJeu = "Dart Adventure"
```

Utilisez `const`.

---

### Exercice 10 — `final` ou `const` (moyen)

Choisissez la meilleure solution pour chacune des valeurs :

```text
3.14159
DateTime.now()
nom fixe d'une application
date actuelle
nombre maximum de vies fixé à 5
```

---

### Exercice 11 — Calcul simple (facile)

Créez :

```dart
double prix = 25.0;
int quantite = 4;
```

Calculez et affichez le total.

Résultat attendu :

```text
Total : 100.0
```

---

### Exercice 12 — Interpolation (facile)

Avec :

```dart
String nom = 'Alex';
int age = 25;
```

produisez :

```text
Bonjour Alex, vous avez 25 ans.
```

---

### Exercice 13 — Expression dans une interpolation (moyen)

Avec :

```dart
int age = 25;
```

produisez :

```text
L'année prochaine, vous aurez 26 ans.
```

Utilisez :

```dart
${}
```

---

### Exercice 14 — Conversion (moyen)

Créez :

```dart
String nombre = '50';
```

Convertissez la chaîne en entier.

Ajoutez ensuite :

```text
25
```

Résultat attendu :

```text
75
```

---

### Exercice 15 — Conversion décimale (moyen)

Avec :

```dart
String prixTexte = '19.99';
```

convertissez la valeur en `double`.

Multipliez-la par `2`.

---

### Exercice 16 — Mini-facture (moyen)

Créez les variables :

```text
produit : Souris
prix : 39.99
quantité : 2
```

Affichez :

```text
============================
FACTURE
============================
Produit : Souris
Prix unitaire : 39.99
Quantité : 2
Total : 79.98
============================
```

Le total doit être calculé par Dart.

---

### Exercice 17 — Jeu vidéo (moyen)

Créez les variables :

```text
joueur
score
nombreDeVies
niveau
jeuTermine
```

avec des types appropriés.

Affichez ensuite une fiche :

```text
JOUEUR
Nom : ...
Score : ...
Vies : ...
Niveau : ...
Jeu terminé : ...
```

Cet exercice prépare directement la Partie 2 consacrée au développement de jeux.

---

### Exercice 18 — Trouver les erreurs (difficile)

Corrigez :

```dart
void main() {
  String nom = 25;
  int age = '30';
  double prix = 19,99;
  bool disponible = 'true';

  print(nom)
}
```

---

### Exercice 19 — Challenge boutique (difficile)

Créez un programme avec :

```text
nom du produit
prix unitaire
quantité
taxe
```

Calculez :

```text
sous-total
montant de la taxe
total final
```

Pour cet exercice, utilisez une taxe de :

```text
15 %
```

Formule :

```text
taxe = sousTotal * 0.15
```

---

### Exercice 20 — Challenge profil de jeu (difficile)

Créez un programme contenant au minimum :

```dart
String nomJoueur;
int niveau;
int score;
int vies;
double energie;
bool vivant;
final DateTime dateDebut;
const int scoreMaximum;
```

Affichez toutes les informations proprement.

---

## 02.73 — Corrections des exercices

### Correction 1

```dart
void main() {
  String nom = 'Alex';
  int age = 25;
  double taille = 1.80;
  bool actif = true;

  print(nom);
  print(age);
  print(taille);
  print(actif);
}
```

**Explication :** on déclare quatre variables avec leur type explicite (`String`, `int`, `double`, `bool`), puis on les affiche une à une avec `print`.

---

### Correction 2

```dart
void main() {
  String nom = 'Sophie';
  int age = 30;
  String ville = 'Québec';

  print('Nom : $nom');
  print('Âge : $age');
  print('Ville : $ville');
}
```

**Explication :** l'interpolation `$nom` et `$age` insère directement la valeur des variables dans la chaîne, sans concaténation manuelle.

---

### Correction 3

```dart
void main() {
  int score = 100;

  print(score);

  score = 200;

  print(score);
}
```

**Explication :** `score` est déclaré une seule fois avec `int score = 100;` ; pour changer sa valeur on écrit simplement `score = 200;`, sans redéclarer le type.

---

### Correction 4

```text
"Bonjour"  → String
125        → int
12.5       → double
true       → bool
-15        → int
"Montréal" → String
false      → bool
3.14159    → double
```

**Explication :** chaque valeur littérale est associée à son type Dart : le texte entre guillemets est un `String`, un nombre sans virgule est un `int`, un nombre avec point décimal est un `double`, et `true`/`false` sont des `bool`.

---

### Correction 5

```dart
void main() {
  var nom = 'Alex';
  var age = 25;
  var prix = 99.99;
  var actif = true;

  print(nom);
  print(age);
  print(prix);
  print(actif);
}
```

**Explication :** `var` déduit automatiquement le type à partir de la valeur d'initialisation : `nom` devient `String`, `age` devient `int`, `prix` devient `double`, `actif` devient `bool`.

---

### Correction 6

Le code n'est pas valide.

```dart
var score = 100;
```

fait comprendre à Dart que :

```text
score → int
```

On ne peut donc pas lui assigner ensuite :

```dart
'cent'
```

qui est un `String`.

**Explication :** `var score = 100;` fixe le type de `score` à `int` dès la première initialisation. Assigner ensuite `'cent'`, qui est un `String`, provoque une erreur car le type ne peut plus changer.

---

### Correction 7

```dart
void main() {
  dynamic valeur = 'Bonjour';
  print(valeur);

  valeur = 100;
  print(valeur);

  valeur = true;
  print(valeur);

  valeur = 19.99;
  print(valeur);
}
```

**Explication :** `dynamic` autorise `valeur` à changer de type à chaque réassignation : `String`, puis `int`, puis `bool`, puis `double` sont tous acceptés.

---

### Correction 8

```dart
void main() {
  final String application = 'GameApp';

  print(application);
}
```

Cette instruction serait interdite :

```dart
application = 'NouvelleApp';
```

Une variable `final` ne peut pas être réassignée.

**Explication :** `final String application = 'GameApp';` fige la valeur après son initialisation. Toute tentative de réassignation, comme `application = 'NouvelleApp';`, est refusée par le compilateur.

---

### Correction 9

```dart
void main() {
  const double pi = 3.14159;
  const int nombreDeVies = 3;
  const String nomDuJeu = 'Dart Adventure';

  print(pi);
  print(nombreDeVies);
  print(nomDuJeu);
}
```

**Explication :** `const` déclare des constantes dont la valeur est connue dès la compilation : un nombre, un entier et une chaîne littérale conviennent parfaitement.

---

### Correction 10

```text
3.14159
→ const

DateTime.now()
→ final

nom fixe d'une application
→ const

date actuelle
→ final

nombre maximum de vies fixé à 5
→ const
```

**Explication :** `const` s'utilise pour des valeurs connues à la compilation (un nombre fixe, un nom d'application, une limite figée) ; `final` s'utilise dès que la valeur dépend de l'exécution, comme `DateTime.now()`.

---

### Correction 11

```dart
void main() {
  double prix = 25.0;
  int quantite = 4;

  double total = prix * quantite;

  print('Total : $total');
}
```

**Explication :** le total est obtenu par une simple multiplication `prix * quantite`, stockée dans une variable `double total`, puis affichée par interpolation.

---

### Correction 12

```dart
void main() {
  String nom = 'Alex';
  int age = 25;

  print('Bonjour $nom, vous avez $age ans.');
}
```

**Explication :** `$nom` et `$age` sont insérés directement dans la phrase grâce à l'interpolation simple, sans concaténation avec `+`.

---

### Correction 13

```dart
void main() {
  int age = 25;

  print("L'année prochaine, vous aurez ${age + 1} ans.");
}
```

**Explication :** l'expression `${age + 1}` est évaluée avant d'être insérée dans la chaîne : les accolades sont obligatoires dès qu'on manipule autre chose qu'une simple variable.

---

### Correction 14

```dart
void main() {
  String nombre = '50';

  int valeur = int.parse(nombre);

  print(valeur + 25);
}
```

**Explication :** `int.parse(nombre)` convertit le `String '50'` en `int 50`, ce qui permet ensuite d'additionner `25` normalement.

---

### Correction 15

```dart
void main() {
  String prixTexte = '19.99';

  double prix = double.parse(prixTexte);

  print(prix * 2);
}
```

**Explication :** `double.parse(prixTexte)` convertit le `String '19.99'` en `double 19.99`, qui peut ensuite être multiplié par `2`.

---

### Correction 16

```dart
void main() {
  String produit = 'Souris';
  double prix = 39.99;
  int quantite = 2;

  double total = prix * quantite;

  print('============================');
  print('FACTURE');
  print('============================');
  print('Produit : $produit');
  print('Prix unitaire : $prix');
  print('Quantité : $quantite');
  print('Total : $total');
  print('============================');
}
```

**Explication :** chaque information (produit, prix, quantité) est stockée dans une variable typée, le total est calculé par Dart (`prix * quantite`), puis toute la facture est affichée avec des séparateurs visuels.

---

### Correction 17

```dart
void main() {
  String joueur = 'Alex';
  int score = 1250;
  int nombreDeVies = 3;
  int niveau = 4;
  bool jeuTermine = false;

  print('JOUEUR');
  print('Nom : $joueur');
  print('Score : $score');
  print('Vies : $nombreDeVies');
  print('Niveau : $niveau');
  print('Jeu terminé : $jeuTermine');
}
```

**Explication :** chaque propriété du joueur reçoit le type le plus adapté (`String` pour le nom, `int` pour le score et les vies, `bool` pour l'état de la partie), puis une fiche complète est affichée par interpolation.

---

### Correction 18

```dart
void main() {
  String nom = 'Alex';
  int age = 30;
  double prix = 19.99;
  bool disponible = true;

  print(nom);
}
```

**Explication :** quatre erreurs à corriger : `String nom` ne peut pas recevoir `25` (un `int`) ; `int age` ne peut pas recevoir `'30'` (un `String`) ; `19,99` doit s'écrire `19.99` avec un point ; et `print(nom)` doit se terminer par un point-virgule.

---

### Correction 19

```dart
void main() {
  String produit = 'Clavier';
  double prix = 100.0;
  int quantite = 2;
  double tauxTaxe = 0.15;

  double sousTotal = prix * quantite;
  double taxe = sousTotal * tauxTaxe;
  double total = sousTotal + taxe;

  print('Produit : $produit');
  print('Sous-total : $sousTotal');
  print('Taxe : $taxe');
  print('Total : $total');
}
```

**Explication :** le calcul se fait en trois étapes : le sous-total (`prix * quantite`), la taxe (`sousTotal * tauxTaxe`), puis le total final (`sousTotal + taxe`) — chaque résultat intermédiaire est stocké dans sa propre variable.

---

### Correction 20

```dart
void main() {
  String nomJoueur = 'Alex';
  int niveau = 5;
  int score = 2500;
  int vies = 3;
  double energie = 87.5;
  bool vivant = true;

  final DateTime dateDebut = DateTime.now();

  const int scoreMaximum = 10000;

  print('============================');
  print('PROFIL DU JOUEUR');
  print('============================');

  print('Nom : $nomJoueur');
  print('Niveau : $niveau');
  print('Score : $score');
  print('Vies : $vies');
  print('Énergie : $energie');
  print('Vivant : $vivant');
  print('Date de début : $dateDebut');
  print('Score maximum : $scoreMaximum');
}
```

**Explication :** le profil combine des types variés (`String`, `int`, `double`, `bool`), une valeur `final` connue seulement à l'exécution (`DateTime.now()`) et une constante `const` connue à la compilation (`scoreMaximum`), le tout affiché dans une fiche formatée.

---

## Et maintenant ?

Vous savez désormais créer, typer, modifier et afficher des variables, choisir entre `var`, `dynamic`, `final`, `const` et `late`, et convertir entre les types. Le prochain chapitre s'appuie directement sur ces variables pour leur faire subir des calculs et des comparaisons : opérateurs arithmétiques, opérateurs d'affectation raccourcis, opérateurs de comparaison et opérateurs logiques.

Rendez-vous au chapitre suivant : [03-PARTIE-1A—OPÉRATEURS-ET-EXPRESSIONS.md](./03-PARTIE-1A—OPÉRATEURS-ET-EXPRESSIONS.md)
