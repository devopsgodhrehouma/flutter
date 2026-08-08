# PARTIE 1A — DART
# CHAPITRE 07 — LES FONCTIONS

> **Niveau :** débutant
> **Durée estimée :** 6 h
> **Pré-requis :** chapitre 06 — Collections : `List`, `Set` et `Map`
> **Ce que vous saurez faire à la fin :** découper un programme en fonctions claires, avec des paramètres, des valeurs de retour, des paramètres optionnels et nommés, et écrire un petit moteur de combat entièrement construit avec des fonctions.

---

## 07.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- comprendre pourquoi une fonction évite de répéter du code ;
- déclarer une fonction ;
- appeler une fonction ;
- comprendre le mot-clé `void` ;
- renvoyer une valeur avec `return` ;
- choisir un type de retour ;
- définir des paramètres ;
- passer des arguments ;
- utiliser plusieurs paramètres ;
- distinguer paramètres positionnels et paramètres nommés ;
- utiliser les paramètres optionnels `[ ]` ;
- utiliser les paramètres nommés `{ }` ;
- utiliser `required` ;
- donner des valeurs par défaut ;
- écrire des fonctions fléchées `=>` ;
- écrire des fonctions anonymes ;
- comprendre la portée des variables ;
- distinguer variables locales et variables globales ;
- passer une fonction en paramètre d'une autre fonction ;
- écrire une fonction récursive simple ;
- décomposer un gros programme en petites fonctions.

---

## 07.1 — Pourquoi créer des fonctions ?

Observons un programme sans fonction.

```dart
void main() {
  print('=== ÉTAT DU JOUEUR ===');
  print('Nom : Alex');
  print('Vies : 3');
  print('======================');

  print('=== ÉTAT DU JOUEUR ===');
  print('Nom : Sophie');
  print('Vies : 2');
  print('======================');

  print('=== ÉTAT DU JOUEUR ===');
  print('Nom : Samir');
  print('Vies : 5');
  print('======================');
}
```

**Résultat :**

```text
=== ÉTAT DU JOUEUR ===
Nom : Alex
Vies : 3
======================
=== ÉTAT DU JOUEUR ===
Nom : Sophie
Vies : 2
======================
=== ÉTAT DU JOUEUR ===
Nom : Samir
Vies : 5
======================
```

Ce programme fonctionne, mais il pose trois problèmes.

```text
1. Le même bloc est recopié trois fois.
2. Si nous voulons changer le cadre, nous devons le changer partout.
3. Le programme devient long et difficile à lire.
```

Une fonction résout ces trois problèmes.

Une fonction est un bloc de code :

```text
- que l'on écrit UNE seule fois ;
- auquel on donne un NOM ;
- que l'on peut exécuter AUTANT DE FOIS que nécessaire.
```

---

## 07.2 — Déclaration d'une fonction

Déclarer une fonction, c'est la créer.

La forme minimale est la suivante :

```text
typeDeRetour nomDeLaFonction() {
  // instructions
}
```

Exemple concret :

```dart
void afficherTitre() {
  print('=== ÉTAT DU JOUEUR ===');
}
```

Décomposons :

```text
void              -> la fonction ne renvoie aucune valeur
afficherTitre     -> le nom de la fonction
()                -> la liste des paramètres (ici : aucun)
{ ... }           -> le corps de la fonction
```

Important : déclarer une fonction ne l'exécute pas.

```dart
void afficherTitre() {
  print('=== ÉTAT DU JOUEUR ===');
}

void main() {
  print('Programme terminé');
}
```

**Résultat :**

```text
Programme terminé
```

La fonction `afficherTitre` existe, mais elle n'a jamais été appelée.

---

## 07.3 — Appel d'une fonction

Appeler une fonction, c'est demander son exécution.

On écrit son nom suivi de parenthèses.

```dart
void afficherTitre() {
  print('=== ÉTAT DU JOUEUR ===');
}

void main() {
  afficherTitre();
  print('Nom : Alex');
  afficherTitre();
}
```

**Résultat :**

```text
=== ÉTAT DU JOUEUR ===
Nom : Alex
=== ÉTAT DU JOUEUR ===
```

Voici le déroulement de l'exécution :

```text
main() démarre
   │
   ├── appel de afficherTitre()  ──► exécution du corps
   │                             ◄── retour dans main()
   ├── print('Nom : Alex')
   │
   ├── appel de afficherTitre()  ──► exécution du corps
   │                             ◄── retour dans main()
   │
main() se termine
```

Remarque : les parenthèses sont obligatoires. `afficherTitre;` sans parenthèses n'exécute rien.

---

## 07.4 — `void`

`void` signifie : cette fonction ne renvoie aucune valeur.

Elle fait quelque chose (souvent afficher), mais elle ne rend rien à celui qui l'appelle.

```dart
void afficherGameOver() {
  print('----------------');
  print('   GAME OVER    ');
  print('----------------');
}

void main() {
  afficherGameOver();
}
```

**Résultat :**

```text
----------------
   GAME OVER    
----------------
```

C'est pour cette raison que `main` s'écrit :

```dart
void main() {
}
```

`main` est la fonction de départ du programme, et elle ne renvoie rien.

Conséquence importante : on ne peut pas récupérer le résultat d'une fonction `void`.

```text
var x = afficherGameOver();   // interdit : il n'y a rien à récupérer
```

---

## 07.5 — `return`

`return` renvoie une valeur à celui qui a appelé la fonction.

```dart
int donnerScoreDepart() {
  return 100;
}

void main() {
  int score = donnerScoreDepart();
  print(score);
}
```

**Résultat :**

```text
100
```

Ce qui se passe :

```text
main demande la valeur  ──►  donnerScoreDepart()
                             return 100
main reçoit 100         ◄──
score vaut 100
```

`return` termine immédiatement la fonction.

```dart
int donnerScoreDepart() {
  return 100;
  print('Cette ligne ne sera jamais exécutée');
}

void main() {
  print(donnerScoreDepart());
}
```

**Résultat :**

```text
100
```

Dart signale d'ailleurs que le `print` placé après le `return` est du code mort.

---

## 07.6 — Type de retour

Le type écrit devant le nom de la fonction indique la nature de la valeur renvoyée.

```text
int    -> renvoie un nombre entier
double -> renvoie un nombre décimal
String -> renvoie du texte
bool   -> renvoie true ou false
List   -> renvoie une liste
void   -> ne renvoie rien
```

Renvoyer un entier :

```dart
int calculerDegats(int force) {
  return force * 2;
}

void main() {
  print(calculerDegats(15));
}
```

**Résultat :**

```text
30
```

Renvoyer du texte :

```dart
String nomDuNiveau(int numero) {
  return 'Niveau $numero';
}

void main() {
  print(nomDuNiveau(3));
}
```

**Résultat :**

```text
Niveau 3
```

Renvoyer un booléen :

```dart
bool estVivant(int vies) {
  return vies > 0;
}

void main() {
  print(estVivant(3));
  print(estVivant(0));
}
```

**Résultat :**

```text
true
false
```

Renvoyer un nombre décimal :

```dart
double moitieEnergie(double energie) {
  return energie / 2;
}

void main() {
  print(moitieEnergie(80.0));
}
```

**Résultat :**

```text
40.0
```

Règle absolue : le type annoncé et la valeur renvoyée doivent correspondre.

```text
int calculerDegats(int force) {
  return 'beaucoup';   // ERREUR : String renvoyé alors que int est annoncé
}
```

---

## 07.7 — Paramètres

Un paramètre est une variable déclarée entre les parenthèses de la fonction.

Il permet à la fonction de recevoir une information de l'extérieur.

```dart
void afficherJoueur(String nom) {
  print('Joueur : $nom');
}

void main() {
  afficherJoueur('Alex');
  afficherJoueur('Sophie');
  afficherJoueur('Samir');
}
```

**Résultat :**

```text
Joueur : Alex
Joueur : Sophie
Joueur : Samir
```

Une seule fonction, trois affichages différents.

Le paramètre `nom` :

```text
- n'existe QUE à l'intérieur de la fonction ;
- prend une valeur différente à chaque appel.
```

---

## 07.8 — Arguments

Il faut distinguer deux mots proches.

```text
Paramètre -> le nom écrit dans la DÉCLARATION de la fonction
Argument  -> la valeur réelle passée lors de l'APPEL
```

Illustration :

```dart
void afficherVies(int vies) {   // vies est le PARAMÈTRE
  print('Vies restantes : $vies');
}

void main() {
  afficherVies(3);              // 3 est l'ARGUMENT
  int reste = 1;
  afficherVies(reste);          // reste est l'ARGUMENT
}
```

**Résultat :**

```text
Vies restantes : 3
Vies restantes : 1
```

Schéma du passage de valeur :

```text
main                     afficherVies
----                     ------------
reste = 1  ──copie──►    vies = 1
```

La fonction reçoit une copie de la valeur. Modifier `vies` dans la fonction ne change pas `reste` dans `main`.

```dart
void consommer(int energie) {
  energie = energie - 50;
  print('Dans la fonction : $energie');
}

void main() {
  int energieJoueur = 100;
  consommer(energieJoueur);
  print('Dans main : $energieJoueur');
}
```

**Résultat :**

```text
Dans la fonction : 50
Dans main : 100
```

---

## 07.9 — Plusieurs paramètres

Une fonction peut recevoir plusieurs paramètres, séparés par des virgules.

```dart
void afficherFiche(String nom, int vies, int score) {
  print('$nom | vies : $vies | score : $score');
}

void main() {
  afficherFiche('Alex', 3, 1200);
  afficherFiche('Sophie', 2, 950);
}
```

**Résultat :**

```text
Alex | vies : 3 | score : 1200
Sophie | vies : 2 | score : 950
```

Chaque paramètre a son propre type.

```dart
int degatsTotaux(int force, int arme, double multiplicateur) {
  double resultat = (force + arme) * multiplicateur;
  return resultat.round();
}

void main() {
  print(degatsTotaux(10, 5, 1.5));
}
```

**Résultat :**

```text
23
```

Remarque : `(10 + 5) * 1.5` vaut `22.5`, et `round()` arrondit à `23`.

---

## 07.10 — Paramètres positionnels

Les paramètres écrits simplement entre parenthèses sont dits **positionnels**.

Cela signifie que l'ordre des arguments compte.

```dart
void afficherCombat(String attaquant, String defenseur) {
  print('$attaquant attaque $defenseur');
}

void main() {
  afficherCombat('Alex', 'Gobelin');
  afficherCombat('Gobelin', 'Alex');
}
```

**Résultat :**

```text
Alex attaque Gobelin
Gobelin attaque Alex
```

Le même appel avec les arguments inversés produit un sens totalement différent.

Schéma :

```text
Déclaration :  afficherCombat(attaquant, defenseur)
                                 │           │
Appel :        afficherCombat('Alex',   'Gobelin')
```

Danger classique : deux paramètres de même type.

```dart
void afficherStats(int vies, int score) {
  print('Vies : $vies');
  print('Score : $score');
}

void main() {
  afficherStats(1200, 3);
}
```

**Résultat :**

```text
Vies : 1200
Score : 3
```

Dart ne signale aucune erreur : les deux valeurs sont des `int`. L'erreur est purement humaine.

Nous verrons en section 07.12 comment les paramètres nommés suppriment ce risque.

---

## 07.11 — Paramètres optionnels `[ ]`

Un paramètre optionnel peut être omis lors de l'appel.

On l'écrit entre crochets, et on lui donne une valeur par défaut.

```dart
void attaquer(String cible, [int degats = 10]) {
  print('$cible subit $degats dégâts');
}

void main() {
  attaquer('Gobelin');
  attaquer('Dragon', 75);
}
```

**Résultat :**

```text
Gobelin subit 10 dégâts
Dragon subit 75 dégâts
```

Règles à connaître :

```text
1. Les paramètres optionnels se placent TOUJOURS après les obligatoires.
2. Ils restent positionnels : l'ordre compte.
3. Ils doivent avoir une valeur par défaut.
```

Plusieurs paramètres optionnels :

```dart
void lancerNiveau(String monde, [int numero = 1, bool difficile = false]) {
  print('Monde : $monde | Niveau : $numero | Difficile : $difficile');
}

void main() {
  lancerNiveau('Forêt');
  lancerNiveau('Volcan', 3);
  lancerNiveau('Château', 5, true);
}
```

**Résultat :**

```text
Monde : Forêt | Niveau : 1 | Difficile : false
Monde : Volcan | Niveau : 3 | Difficile : false
Monde : Château | Niveau : 5 | Difficile : true
```

Limite importante : on ne peut pas sauter un paramètre optionnel.

```text
lancerNiveau('Volcan', true);   // ERREUR : true n'est pas un int
```

Pour fournir `difficile`, il faut obligatoirement fournir `numero` avant.

---

## 07.12 — Paramètres nommés `{ }`

Un paramètre nommé se déclare entre accolades et s'utilise avec son nom lors de l'appel.

```dart
void creerJoueur({String nom = 'Héros', int vies = 3, int score = 0}) {
  print('$nom | vies : $vies | score : $score');
}

void main() {
  creerJoueur(nom: 'Alex', vies: 5, score: 1200);
  creerJoueur(score: 900, nom: 'Sophie');
  creerJoueur();
}
```

**Résultat :**

```text
Alex | vies : 5 | score : 1200
Sophie | vies : 3 | score : 900
Héros | vies : 3 | score : 0
```

Trois avantages majeurs :

```text
1. L'ordre n'a plus aucune importance.
2. L'appel est lisible : on voit ce que chaque valeur signifie.
3. On peut ne fournir que les paramètres utiles.
```

Comparaison directe :

```text
Positionnel :  afficherStats(1200, 3);
Nommé :        afficherStats(score: 1200, vies: 3);
```

La seconde version se relit sans consulter la déclaration.

Remarque : Flutter utilise massivement les paramètres nommés. Vous les retrouverez dans presque tous les widgets.

---

## 07.13 — `required`

Par défaut, un paramètre nommé est facultatif.

Si nous voulons le rendre obligatoire, nous écrivons `required` devant lui.

```dart
void infligerDegats({required String cible, required int degats}) {
  print('$cible perd $degats points de vie');
}

void main() {
  infligerDegats(cible: 'Gobelin', degats: 25);
}
```

**Résultat :**

```text
Gobelin perd 25 points de vie
```

Si un paramètre `required` est oublié, le programme refuse de compiler :

```text
infligerDegats(cible: 'Gobelin');   // ERREUR : degats est manquant
```

C'est une protection utile : elle empêche de créer un objet du jeu incomplet.

Mélange de `required` et de valeurs par défaut :

```dart
void creerEnnemi({
  required String nom,
  required int vies,
  int degats = 5,
  bool boss = false,
}) {
  print('$nom | vies : $vies | dégâts : $degats | boss : $boss');
}

void main() {
  creerEnnemi(nom: 'Gobelin', vies: 30);
  creerEnnemi(nom: 'Dragon', vies: 500, degats: 60, boss: true);
}
```

**Résultat :**

```text
Gobelin | vies : 30 | dégâts : 5 | boss : false
Dragon | vies : 500 | dégâts : 60 | boss : true
```

---

## 07.14 — Valeurs par défaut

Une valeur par défaut est utilisée quand l'argument n'est pas fourni.

On l'écrit avec `=` dans la déclaration.

```dart
void soigner({int soin = 20}) {
  print('Le joueur récupère $soin points de vie');
}

void main() {
  soigner();
  soigner(soin: 50);
}
```

**Résultat :**

```text
Le joueur récupère 20 points de vie
Le joueur récupère 50 points de vie
```

Une valeur par défaut doit être une constante connue à l'écriture du code.

```text
Autorisé   : int degats = 10
Autorisé   : String nom = 'Héros'
Autorisé   : bool boss = false
Interdit   : int degats = calculerDegats(5)
```

La dernière ligne est interdite car la valeur ne serait connue qu'à l'exécution.

Exemple complet avec les deux familles de paramètres :

```dart
String decrireArme(String nom, {int degats = 10, bool magique = false}) {
  String texte = '$nom : $degats dégâts';
  if (magique) {
    texte = '$texte (magique)';
  }
  return texte;
}

void main() {
  print(decrireArme('Épée'));
  print(decrireArme('Bâton', degats: 25, magique: true));
}
```

**Résultat :**

```text
Épée : 10 dégâts
Bâton : 25 dégâts (magique)
```

---

## 07.15 — Fonctions fléchées `=>`

Quand une fonction contient une seule instruction, on peut l'écrire sur une ligne.

Forme longue :

```dart
int doubler(int valeur) {
  return valeur * 2;
}
```

Forme fléchée équivalente :

```dart
int doubler(int valeur) => valeur * 2;
```

La flèche `=>` remplace à la fois les accolades et le mot `return`.

Programme complet :

```dart
int doubler(int valeur) => valeur * 2;
bool estVivant(int vies) => vies > 0;
String niveauTexte(int niveau) => 'Niveau $niveau';

void main() {
  print(doubler(21));
  print(estVivant(0));
  print(niveauTexte(7));
}
```

**Résultat :**

```text
42
false
Niveau 7
```

La flèche fonctionne aussi avec `void` :

```dart
void afficherScore(int score) => print('Score : $score');

void main() {
  afficherScore(1500);
}
```

**Résultat :**

```text
Score : 1500
```

Attention : la flèche n'accepte qu'une seule expression.

```text
int total(int a, int b) => {   // INTERDIT
  int s = a + b;
  return s;
};
```

Dès qu'il y a deux instructions, il faut revenir aux accolades.

---

## 07.16 — Fonctions anonymes

Une fonction anonyme est une fonction sans nom.

On la stocke généralement dans une variable, puis on l'appelle par le nom de cette variable.

```dart
void main() {
  var doubler = (int valeur) {
    return valeur * 2;
  };

  print(doubler(15));
}
```

**Résultat :**

```text
30
```

La même chose en version fléchée :

```dart
void main() {
  var tripler = (int valeur) => valeur * 3;
  print(tripler(15));
}
```

**Résultat :**

```text
45
```

On peut aussi annoncer précisément le type de la variable.

```dart
void main() {
  int Function(int) quadrupler = (int valeur) => valeur * 4;
  print(quadrupler(15));
}
```

**Résultat :**

```text
60
```

Lecture du type :

```text
int Function(int)
 │        │    │
 │        │    └── prend un int en paramètre
 │        └─────── c'est une fonction
 └──────────────── elle renvoie un int
```

Les fonctions anonymes prendront tout leur sens en section 07.19, et surtout au chapitre 14 lorsque nous manipulerons les collections.

---

## 07.17 — Portée des variables

La portée d'une variable est la zone du programme où cette variable existe.

En Dart, une variable existe à l'intérieur du bloc `{ }` où elle a été déclarée.

```dart
void preparerCombat() {
  int degats = 30;
  print('Dégâts préparés : $degats');
}

void main() {
  preparerCombat();
  print('Fin du programme');
}
```

**Résultat :**

```text
Dégâts préparés : 30
Fin du programme
```

La variable `degats` disparaît dès que la fonction se termine.

```text
void main() {
  preparerCombat();
  print(degats);   // ERREUR : degats n'existe pas ici
}
```

Schéma de la portée :

```text
main { ................................ }
       │
       └── preparerCombat {
              degats existe ici
           }
           degats n'existe plus après cette accolade
```

La portée s'applique aussi aux blocs `if` et `for`.

```dart
void main() {
  int score = 0;

  for (int i = 1; i <= 3; i++) {
    int bonus = i * 10;
    score = score + bonus;
  }

  print(score);
}
```

**Résultat :**

```text
60
```

Ici, `bonus` n'existe que dans la boucle. `score`, déclaré avant, existe partout dans `main`.

---

## 07.18 — Variables locales / globales

Une variable **locale** est déclarée à l'intérieur d'une fonction.

Une variable **globale** est déclarée en dehors de toute fonction, au niveau du fichier.

```dart
int scoreGlobal = 0;

void ajouterPoints(int points) {
  scoreGlobal = scoreGlobal + points;
}

void main() {
  ajouterPoints(100);
  ajouterPoints(250);
  print('Score total : $scoreGlobal');
}
```

**Résultat :**

```text
Score total : 350
```

Comparaison :

| Type | Déclarée où | Visible où | Durée de vie |
| --- | --- | --- | --- |
| Locale | dans une fonction ou un bloc | dans ce bloc uniquement | le temps de l'exécution du bloc |
| Globale | en dehors de toute fonction | dans tout le fichier | toute la durée du programme |

Attention au piège de l'ombrage : une variable locale peut porter le même nom qu'une globale.

```dart
int vies = 3;

void perdreUneVie() {
  int vies = 99;
  vies = vies - 1;
  print('Dans la fonction : $vies');
}

void main() {
  perdreUneVie();
  print('Globale : $vies');
}
```

**Résultat :**

```text
Dans la fonction : 98
Globale : 3
```

La variable locale masque la globale. La globale n'a pas bougé.

Conseil : utilisez les variables globales avec parcimonie. Préférez les paramètres et les valeurs de retour, plus faciles à suivre.

---

## 07.19 — Fonctions comme paramètres (callbacks, `Function`)

En Dart, une fonction est une valeur. On peut donc la passer en paramètre d'une autre fonction.

La fonction ainsi passée est appelée un **callback** : elle sera rappelée plus tard par la fonction qui la reçoit.

Version la plus simple avec le type `Function` :

```dart
void attaquer() {
  print('Le joueur attaque');
}

void soigner() {
  print('Le joueur se soigne');
}

void jouerTour(Function action) {
  print('--- Nouveau tour ---');
  action();
}

void main() {
  jouerTour(attaquer);
  jouerTour(soigner);
}
```

**Résultat :**

```text
--- Nouveau tour ---
Le joueur attaque
--- Nouveau tour ---
Le joueur se soigne
```

Remarque essentielle :

```text
jouerTour(attaquer)    -> on PASSE la fonction
jouerTour(attaquer())  -> on passe le RÉSULTAT de la fonction
```

Sans parenthèses, on transmet la fonction elle-même.

Version typée, plus sûre :

```dart
int appliquerBonus(int score, int Function(int) bonus) {
  return bonus(score);
}

int doubleScore(int valeur) => valeur * 2;
int scorePlusCent(int valeur) => valeur + 100;

void main() {
  print(appliquerBonus(500, doubleScore));
  print(appliquerBonus(500, scorePlusCent));
}
```

**Résultat :**

```text
1000
600
```

On peut aussi passer directement une fonction anonyme :

```dart
int appliquerBonus(int score, int Function(int) bonus) {
  return bonus(score);
}

void main() {
  print(appliquerBonus(500, (int valeur) => valeur ~/ 2));
}
```

**Résultat :**

```text
250
```

Ce mécanisme est partout dans Flutter : le paramètre `onPressed` d'un bouton est exactement un callback.

---

## 07.20 — Récursivité (bases)

Une fonction récursive est une fonction qui s'appelle elle-même.

Toute fonction récursive a besoin de deux éléments :

```text
1. Un cas d'arrêt  -> la condition qui stoppe les appels.
2. Un cas récursif -> l'appel à elle-même, sur un problème plus petit.
```

Exemple : un compte à rebours avant le début du combat.

```dart
void compteARebours(int n) {
  if (n == 0) {
    print('COMBAT !');
    return;
  }

  print(n);
  compteARebours(n - 1);
}

void main() {
  compteARebours(3);
}
```

**Résultat :**

```text
3
2
1
COMBAT !
```

Déroulement des appels :

```text
compteARebours(3)  affiche 3
  compteARebours(2)  affiche 2
    compteARebours(1)  affiche 1
      compteARebours(0)  affiche COMBAT ! puis return
```

Exemple classique : la factorielle, utile pour calculer un nombre de combinaisons d'équipement.

```dart
int factorielle(int n) {
  if (n <= 1) {
    return 1;
  }

  return n * factorielle(n - 1);
}

void main() {
  print(factorielle(5));
}
```

**Résultat :**

```text
120
```

Détail du calcul :

```text
factorielle(5) = 5 * factorielle(4)
factorielle(4) = 4 * factorielle(3)
factorielle(3) = 3 * factorielle(2)
factorielle(2) = 2 * factorielle(1)
factorielle(1) = 1

5 * 4 * 3 * 2 * 1 = 120
```

Danger : sans cas d'arrêt, les appels ne s'arrêtent jamais.

```text
int factorielle(int n) {
  return n * factorielle(n - 1);   // appels infinis : le programme plante
}
```

Le programme finit par saturer la mémoire réservée aux appels.

Remarque : une boucle `for` peut souvent remplacer la récursivité, et reste plus simple pour un débutant.

---

## 07.21 — Décomposer un gros programme en fonctions

Voici la méthode à appliquer.

```text
1. Écrire ce que le programme doit faire, en français, étape par étape.
2. Créer une fonction par étape.
3. Donner à chaque fonction un nom qui décrit son rôle.
4. Faire en sorte que chaque fonction fasse UNE seule chose.
5. Assembler les fonctions dans main().
```

Appliquons cela à un combat.

Étapes en français :

```text
- savoir si un combattant est vivant ;
- appliquer des dégâts sans descendre sous zéro ;
- afficher l'état d'un combattant ;
- enchaîner les tours jusqu'à la fin du combat.
```

Programme complet :

```dart
bool estVivant(int vies) => vies > 0;

int appliquerDegats(int vies, int degats) {
  int resultat = vies - degats;
  if (resultat < 0) {
    resultat = 0;
  }
  return resultat;
}

void afficherEtat(String nom, int vies) {
  print('$nom : $vies PV');
}

void main() {
  String joueur = 'Alex';
  String ennemi = 'Gobelin';
  int viesJoueur = 100;
  int viesEnnemi = 60;
  int tour = 1;

  while (estVivant(viesJoueur) && estVivant(viesEnnemi)) {
    print('--- Tour $tour ---');

    viesEnnemi = appliquerDegats(viesEnnemi, 25);
    afficherEtat(ennemi, viesEnnemi);

    if (!estVivant(viesEnnemi)) {
      break;
    }

    viesJoueur = appliquerDegats(viesJoueur, 15);
    afficherEtat(joueur, viesJoueur);

    tour++;
  }

  if (estVivant(viesJoueur)) {
    print('$joueur remporte le combat');
  } else {
    print('$ennemi remporte le combat');
  }
}
```

**Résultat :**

```text
--- Tour 1 ---
Gobelin : 35 PV
Alex : 85 PV
--- Tour 2 ---
Gobelin : 10 PV
Alex : 70 PV
--- Tour 3 ---
Gobelin : 0 PV
Alex remporte le combat
```

Le `main` se lit maintenant comme un scénario. Les détails sont rangés dans les fonctions.

---

## 07.22 — Une fonction peut en appeler une autre

Les fonctions se combinent entre elles.

```dart
int degatsDeBase(int force) => force * 2;

int degatsAvecArme(int force, int bonusArme) {
  return degatsDeBase(force) + bonusArme;
}

int degatsCritiques(int force, int bonusArme) {
  return degatsAvecArme(force, bonusArme) * 3;
}

void main() {
  print(degatsDeBase(10));
  print(degatsAvecArme(10, 5));
  print(degatsCritiques(10, 5));
}
```

**Résultat :**

```text
20
25
75
```

Chaque fonction reste courte et compréhensible seule.

---

## 07.23 — Une fonction peut renvoyer une collection

Le type de retour peut être une `List`.

```dart
List<String> inventaireDeDepart() {
  return ['Épée', 'Potion', 'Bouclier'];
}

void main() {
  List<String> sac = inventaireDeDepart();
  print(sac);
  print(sac.length);
}
```

**Résultat :**

```text
[Épée, Potion, Bouclier]
3
```

Autre exemple : construire une liste de scores.

```dart
List<int> scoresDesNiveaux(int nombreDeNiveaux) {
  List<int> scores = [];

  for (int i = 1; i <= nombreDeNiveaux; i++) {
    scores.add(i * 100);
  }

  return scores;
}

void main() {
  print(scoresDesNiveaux(5));
}
```

**Résultat :**

```text
[100, 200, 300, 400, 500]
```

---

## 07.24 — Le retour anticipé

Une fonction peut contenir plusieurs `return`. Le premier exécuté arrête la fonction.

```dart
String rangDuJoueur(int score) {
  if (score >= 1000) {
    return 'Or';
  }

  if (score >= 500) {
    return 'Argent';
  }

  return 'Bronze';
}

void main() {
  print(rangDuJoueur(1500));
  print(rangDuJoueur(700));
  print(rangDuJoueur(100));
}
```

**Résultat :**

```text
Or
Argent
Bronze
```

Cette technique évite les `if / else` imbriqués et rend le code plus plat.

Elle sert aussi à sortir tout de suite en cas de valeur invalide.

```dart
void boirePotion(int potions) {
  if (potions <= 0) {
    print('Aucune potion disponible');
    return;
  }

  print('Potion bue, énergie restaurée');
}

void main() {
  boirePotion(0);
  boirePotion(2);
}
```

**Résultat :**

```text
Aucune potion disponible
Potion bue, énergie restaurée
```

Dans une fonction `void`, `return` s'écrit seul, sans valeur.

---

## 07.25 — Bien nommer ses fonctions

Un bon nom de fonction décrit une action.

| Convention | Exemple |
| --- | --- |
| Écriture en `lowerCamelCase` | `calculerDegats` |
| Commence par un verbe | `afficherEtat`, `appliquerDegats` |
| Question pour un `bool` | `estVivant`, `aAssezEnergie` |
| Nom explicite plutôt que court | `soignerJoueur` plutôt que `sj` |

Mauvais noms et bons noms :

```text
f1(int a)               ->  calculerDegats(int force)
truc(String s)          ->  afficherJoueur(String nom)
check(int v)            ->  estVivant(int vies)
```

Un nom clair supprime le besoin de commentaire.

---

## 07.26 — Récapitulatif des types de paramètres

| Écriture | Nom | Obligatoire ? | À l'appel |
| --- | --- | --- | --- |
| `f(int a)` | positionnel obligatoire | oui | `f(10)` |
| `f(int a, [int b = 0])` | positionnel optionnel | non | `f(10)` ou `f(10, 5)` |
| `f({int b = 0})` | nommé avec défaut | non | `f()` ou `f(b: 5)` |
| `f({required int b})` | nommé obligatoire | oui | `f(b: 5)` |

Exemple qui réunit les trois familles :

```dart
void lancerPartie(
  String joueur, [
  int niveau = 1,
]) {
  print('$joueur commence au niveau $niveau');
}

void configurer({required String difficulte, bool sonActif = true}) {
  print('Difficulté : $difficulte | Son : $sonActif');
}

void main() {
  lancerPartie('Alex');
  lancerPartie('Sophie', 4);
  configurer(difficulte: 'Difficile');
  configurer(difficulte: 'Facile', sonActif: false);
}
```

**Résultat :**

```text
Alex commence au niveau 1
Sophie commence au niveau 4
Difficulté : Difficile | Son : true
Difficulté : Facile | Son : false
```

Remarque : une même fonction ne peut pas mélanger paramètres optionnels `[ ]` et paramètres nommés `{ }`. Il faut choisir l'un ou l'autre.

---

## 07.27 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| La fonction ne fait rien | Elle est déclarée mais jamais appelée | Ajouter l'appel dans `main()`, avec les parenthèses : `afficherTitre();` |
| Oublier les parenthèses à l'appel | `afficherTitre;` désigne la fonction, il ne l'exécute pas | Toujours écrire `nomDeLaFonction()` pour exécuter |
| `The body might complete normally` | Une fonction annonce un type de retour mais un chemin d'exécution n'a pas de `return` | Vérifier que TOUS les cas (`if`, `else`) se terminent par un `return` |
| Déclarer `void` puis utiliser le résultat | `void` ne renvoie rien, il n'y a rien à stocker | Changer le type de retour (`int`, `String`, `bool`) et ajouter un `return` |
| Inverser deux arguments positionnels | Les deux paramètres ont le même type, Dart ne détecte rien | Utiliser des paramètres nommés `{ }` pour rendre l'appel explicite |
| Paramètre optionnel placé avant un obligatoire | `void f([int a = 0], String b)` est interdit | Placer les paramètres optionnels `[ ]` toujours en dernier |
| Oublier `nom:` avec un paramètre nommé | Les paramètres `{ }` s'appellent obligatoirement par leur nom | Écrire `creerJoueur(nom: 'Alex')` et non `creerJoueur('Alex')` |
| Paramètre `required` non fourni | Un paramètre marqué `required` ne peut pas être omis | Fournir la valeur, ou lui donner une valeur par défaut et retirer `required` |
| Valeur par défaut calculée | Une valeur par défaut doit être connue à l'écriture du code | Utiliser une constante (`10`, `'Héros'`, `false`) et calculer dans le corps |
| Utiliser une variable locale hors de sa fonction | La variable est détruite à la fin du bloc | La déclarer avant, ou la renvoyer avec `return` |
| Croire qu'un paramètre modifie la variable d'origine | La fonction reçoit une copie de la valeur | Renvoyer la nouvelle valeur et réaffecter : `vies = appliquerDegats(vies, 20);` |
| Fonction récursive sans cas d'arrêt | La fonction s'appelle sans fin et sature la mémoire | Écrire d'abord la condition d'arrêt, avant l'appel récursif |
| Utiliser `=>` avec plusieurs instructions | La flèche n'accepte qu'une seule expression | Revenir aux accolades `{ }` et à `return` |
| Passer `action()` au lieu de `action` en callback | Les parenthèses exécutent la fonction et passent son résultat | Passer le nom seul, sans parenthèses |
| Deux fonctions avec le même nom | Dart n'autorise pas la surcharge de fonctions | Donner des noms différents : `soignerJoueur`, `soignerEnnemi` |

---

## 07.28 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| Fonction | Bloc de code nommé, écrit une fois, exécutable autant de fois que voulu |
| Déclaration | `typeDeRetour nom(paramètres) { ... }` |
| Appel | `nom();` — les parenthèses sont obligatoires |
| `void` | La fonction ne renvoie aucune valeur |
| `return` | Renvoie une valeur et termine immédiatement la fonction |
| Type de retour | Doit correspondre exactement à la valeur renvoyée |
| Paramètre | Variable déclarée dans la parenthèse de la fonction |
| Argument | Valeur réellement passée lors de l'appel |
| Passage par valeur | La fonction reçoit une copie ; l'original n'est pas modifié |
| Positionnel | L'ordre des arguments compte |
| Optionnel `[ ]` | Peut être omis, doit avoir une valeur par défaut, reste positionnel |
| Nommé `{ }` | Appelé par son nom, ordre libre, très lisible |
| `required` | Rend un paramètre nommé obligatoire |
| Valeur par défaut | Constante utilisée quand l'argument n'est pas fourni |
| `=>` | Écriture courte pour une fonction d'une seule expression |
| Fonction anonyme | Fonction sans nom, stockée dans une variable ou passée en argument |
| `Function` | Type permettant de passer une fonction en paramètre (callback) |
| Portée | Une variable n'existe que dans le bloc `{ }` où elle est déclarée |
| Locale / globale | Locale : dans une fonction. Globale : au niveau du fichier |
| Récursivité | Fonction qui s'appelle elle-même ; exige toujours un cas d'arrêt |
| Décomposition | Une fonction = une seule responsabilité, avec un nom qui la décrit |

---

## 07.29 — Exercices

### Exercice 1 — Message de bienvenue (facile)

Créez une fonction `direBienvenue()` sans paramètre et sans valeur de retour.

Elle doit afficher :

```text
Bienvenue dans le jeu !
```

Appelez-la trois fois depuis `main()`.

---

### Exercice 2 — Afficher un joueur (facile)

Créez une fonction `afficherJoueur` qui reçoit un `String nom`.

Elle doit afficher :

```text
Joueur : Alex
Joueur : Sophie
Joueur : Samir
```

pour les trois appels correspondants.

---

### Exercice 3 — Additionner deux scores (facile)

Créez une fonction `additionnerScores` qui reçoit deux `int` et renvoie leur somme.

Affichez le résultat pour :

```text
1200 et 350
```

---

### Exercice 4 — Le joueur est-il vivant ? (facile)

Créez une fonction `estVivant` qui reçoit un `int vies` et renvoie un `bool`.

Elle renvoie `true` si `vies` est supérieur à `0`.

Testez avec `3`, `1` et `0`.

---

### Exercice 5 — Fonction fléchée (facile)

Réécrivez la fonction suivante avec la syntaxe `=>` :

```dart
int doublerScore(int score) {
  return score * 2;
}
```

Testez avec `750`.

---

### Exercice 6 — Appliquer des dégâts (moyen)

Créez une fonction `appliquerDegats(int vies, int degats)` qui renvoie les vies restantes.

Le résultat ne doit jamais être négatif : il s'arrête à `0`.

Testez avec :

```text
100 vies et 30 dégâts
20 vies et 50 dégâts
```

---

### Exercice 7 — Score avec bonus optionnel (moyen)

Créez une fonction `calculerScore(int base, [int bonus = 0])` qui renvoie `base + bonus`.

Testez avec :

```text
calculerScore(500)
calculerScore(500, 250)
```

---

### Exercice 8 — Créer un ennemi (moyen)

Créez une fonction avec des paramètres nommés :

```text
nom    -> String, required
vies   -> int, required
degats -> int, valeur par défaut 5
boss   -> bool, valeur par défaut false
```

Elle affiche une ligne récapitulative.

Testez avec un `Gobelin` (30 vies) et un `Dragon` (500 vies, 60 dégâts, boss).

---

### Exercice 9 — Total de l'inventaire (moyen)

Créez une fonction `totalInventaire(List<int> valeurs)` qui renvoie la somme des valeurs.

Testez avec :

```dart
[50, 120, 30, 200]
```

---

### Exercice 10 — Effet appliqué au score (moyen)

Créez une fonction `appliquerEffet(int score, int Function(int) effet)` qui renvoie `effet(score)`.

Passez-lui deux fonctions anonymes :

```text
- une qui double le score ;
- une qui retire 100 points.
```

Testez avec un score de `800`.

---

### Exercice 11 — Récursivité (difficile)

Créez deux fonctions récursives.

```text
sommeDegats(int n) -> renvoie n + (n-1) + ... + 1
puissance(int base, int exposant) -> renvoie base élevé à la puissance exposant
```

Testez avec `sommeDegats(5)` et `puissance(2, 10)`.

---

### Exercice 12 — Mini-projet : moteur de combat (difficile)

Écrivez un programme complet de combat au tour par tour, construit uniquement avec des fonctions.

Fonctions imposées :

```text
attaquer(int viesCible, int degats)      -> renvoie les vies restantes, minimum 0
soigner(int vies, int soin, int maximum) -> renvoie les vies soignées, plafonnées au maximum
estVivant(int vies)                      -> renvoie true si vies > 0
afficherEtat(String nom, int vies)       -> affiche "nom : X PV"
```

Règles du combat :

```text
- Le joueur Alex a 100 PV et 2 potions.
- L'ennemi Gobelin a 80 PV.
- À chaque tour, le joueur attaque en premier pour 30 dégâts.
- Si l'ennemi est mort, le combat s'arrête immédiatement.
- Sinon l'ennemi riposte pour 25 dégâts.
- Si le joueur tombe sous 60 PV et qu'il lui reste une potion,
  il boit une potion et récupère 40 PV, sans dépasser 100 PV.
- Le combat continue tant que les deux sont vivants.
- À la fin, on affiche le vainqueur.
```

---

## 07.30 — Corrections des exercices

### Correction 1

```dart
void direBienvenue() {
  print('Bienvenue dans le jeu !');
}

void main() {
  direBienvenue();
  direBienvenue();
  direBienvenue();
}
```

**Résultat :**

```text
Bienvenue dans le jeu !
Bienvenue dans le jeu !
Bienvenue dans le jeu !
```

**Explication :** la fonction est déclarée une seule fois, puis exécutée trois fois. `void` indique qu'elle ne renvoie rien, et les parenthèses vides indiquent qu'elle n'attend aucun paramètre.

---

### Correction 2

```dart
void afficherJoueur(String nom) {
  print('Joueur : $nom');
}

void main() {
  afficherJoueur('Alex');
  afficherJoueur('Sophie');
  afficherJoueur('Samir');
}
```

**Résultat :**

```text
Joueur : Alex
Joueur : Sophie
Joueur : Samir
```

**Explication :** `nom` est le paramètre. À chaque appel, il prend la valeur de l'argument fourni. Une seule fonction suffit pour trois affichages différents.

---

### Correction 3

```dart
int additionnerScores(int scoreA, int scoreB) {
  return scoreA + scoreB;
}

void main() {
  int total = additionnerScores(1200, 350);
  print('Score total : $total');
}
```

**Résultat :**

```text
Score total : 1550
```

**Explication :** le type de retour est `int`, donc `return` doit renvoyer un entier. La valeur renvoyée est stockée dans la variable `total` de `main`.

---

### Correction 4

```dart
bool estVivant(int vies) {
  return vies > 0;
}

void main() {
  print(estVivant(3));
  print(estVivant(1));
  print(estVivant(0));
}
```

**Résultat :**

```text
true
true
false
```

**Explication :** la comparaison `vies > 0` produit déjà un `bool`. Il est donc inutile d'écrire un `if / else` : on renvoie directement le résultat de la comparaison.

---

### Correction 5

```dart
int doublerScore(int score) => score * 2;

void main() {
  print(doublerScore(750));
}
```

**Résultat :**

```text
1500
```

**Explication :** la fonction ne contenait qu'une seule instruction `return`. La flèche `=>` remplace les accolades et le mot `return` à elle seule.

---

### Correction 6

```dart
int appliquerDegats(int vies, int degats) {
  int resultat = vies - degats;

  if (resultat < 0) {
    resultat = 0;
  }

  return resultat;
}

void main() {
  print(appliquerDegats(100, 30));
  print(appliquerDegats(20, 50));
}
```

**Résultat :**

```text
70
0
```

**Explication :** la soustraction peut produire une valeur négative. Le `if` ramène le résultat à `0` avant le `return`. La fonction ne modifie pas la variable d'origine : elle renvoie une nouvelle valeur, qu'il faudra réaffecter.

---

### Correction 7

```dart
int calculerScore(int base, [int bonus = 0]) {
  return base + bonus;
}

void main() {
  print(calculerScore(500));
  print(calculerScore(500, 250));
}
```

**Résultat :**

```text
500
750
```

**Explication :** `bonus` est un paramètre positionnel optionnel. Sans argument, il vaut `0` grâce à sa valeur par défaut. Il est placé après le paramètre obligatoire, comme l'exige Dart.

---

### Correction 8

```dart
void creerEnnemi({
  required String nom,
  required int vies,
  int degats = 5,
  bool boss = false,
}) {
  print('$nom | vies : $vies | dégâts : $degats | boss : $boss');
}

void main() {
  creerEnnemi(nom: 'Gobelin', vies: 30);
  creerEnnemi(nom: 'Dragon', vies: 500, degats: 60, boss: true);
}
```

**Résultat :**

```text
Gobelin | vies : 30 | dégâts : 5 | boss : false
Dragon | vies : 500 | dégâts : 60 | boss : true
```

**Explication :** `required` rend `nom` et `vies` obligatoires : les oublier empêche la compilation. `degats` et `boss` ont une valeur par défaut, donc ils sont facultatifs. Comme les paramètres sont nommés, leur ordre à l'appel est libre.

---

### Correction 9

```dart
int totalInventaire(List<int> valeurs) {
  int total = 0;

  for (int valeur in valeurs) {
    total = total + valeur;
  }

  return total;
}

void main() {
  List<int> objets = [50, 120, 30, 200];
  print(totalInventaire(objets));
}
```

**Résultat :**

```text
400
```

**Explication :** la fonction combine une boucle `for-in` et une `List`, toutes deux vues au chapitre 06. L'accumulateur `total` est une variable locale : il est recréé à zéro à chaque appel.

---

### Correction 10

```dart
int appliquerEffet(int score, int Function(int) effet) {
  return effet(score);
}

void main() {
  int Function(int) doubler = (int valeur) => valeur * 2;
  int Function(int) malus = (int valeur) => valeur - 100;

  print(appliquerEffet(800, doubler));
  print(appliquerEffet(800, malus));
  print(appliquerEffet(800, (int valeur) => valeur + 50));
}
```

**Résultat :**

```text
1600
700
850
```

**Explication :** `int Function(int)` décrit une fonction qui reçoit un `int` et renvoie un `int`. Les deux premières fonctions anonymes sont stockées dans des variables, la troisième est écrite directement à l'appel. Dans les trois cas, `appliquerEffet` ne connaît pas le calcul : elle se contente de rappeler la fonction reçue.

---

### Correction 11

```dart
int sommeDegats(int n) {
  if (n <= 0) {
    return 0;
  }

  return n + sommeDegats(n - 1);
}

int puissance(int base, int exposant) {
  if (exposant == 0) {
    return 1;
  }

  return base * puissance(base, exposant - 1);
}

void main() {
  print(sommeDegats(5));
  print(puissance(2, 10));
}
```

**Résultat :**

```text
15
1024
```

**Explication :** chaque fonction commence par son cas d'arrêt, puis s'appelle elle-même sur un problème plus petit. `sommeDegats(5)` calcule `5 + 4 + 3 + 2 + 1`, soit `15`. `puissance(2, 10)` multiplie `2` par lui-même dix fois, soit `1024`. Sans le premier `if`, les appels ne s'arrêteraient jamais.

---

### Correction 12

```dart
String nomJoueur = 'Alex';
int viesJoueur = 100;
int potions = 2;

String nomEnnemi = 'Gobelin';
int viesEnnemi = 80;

bool estVivant(int vies) => vies > 0;

int attaquer(int viesCible, int degats) {
  int resultat = viesCible - degats;

  if (resultat < 0) {
    resultat = 0;
  }

  return resultat;
}

int soigner(int vies, int soin, int maximum) {
  int resultat = vies + soin;

  if (resultat > maximum) {
    resultat = maximum;
  }

  return resultat;
}

void afficherEtat(String nom, int vies) {
  print('$nom : $vies PV');
}

void main() {
  int tour = 1;

  while (estVivant(viesJoueur) && estVivant(viesEnnemi)) {
    print('--- Tour $tour ---');

    viesEnnemi = attaquer(viesEnnemi, 30);
    afficherEtat(nomEnnemi, viesEnnemi);

    if (!estVivant(viesEnnemi)) {
      break;
    }

    viesJoueur = attaquer(viesJoueur, 25);
    afficherEtat(nomJoueur, viesJoueur);

    if (viesJoueur < 60 && potions > 0) {
      viesJoueur = soigner(viesJoueur, 40, 100);
      potions = potions - 1;
      print('$nomJoueur boit une potion');
      afficherEtat(nomJoueur, viesJoueur);
    }

    tour = tour + 1;
  }

  if (estVivant(viesJoueur)) {
    print('$nomJoueur remporte le combat');
  } else {
    print('$nomEnnemi remporte le combat');
  }
}
```

**Résultat :**

```text
--- Tour 1 ---
Gobelin : 50 PV
Alex : 75 PV
--- Tour 2 ---
Gobelin : 20 PV
Alex : 50 PV
Alex boit une potion
Alex : 90 PV
--- Tour 3 ---
Gobelin : 0 PV
Alex remporte le combat
```

**Explication :** l'état du combat est stocké dans des variables globales, ce qui permet à `main` de le faire évoluer tour après tour. Chaque fonction a une responsabilité unique : `attaquer` calcule des vies après dégâts, `soigner` calcule des vies après soin, `estVivant` répond à une question, `afficherEtat` affiche. Comme les paramètres sont passés par copie, les fonctions renvoient toujours la nouvelle valeur, que `main` réaffecte avec `viesEnnemi = attaquer(...)`. Le `break` évite que l'ennemi riposte alors qu'il est déjà à `0` PV.

---

## Et maintenant ?

Vous savez désormais découper un programme en fonctions, leur passer des paramètres positionnels, optionnels ou nommés, renvoyer des valeurs, écrire des fonctions fléchées et anonymes, passer une fonction en callback et raisonner sur la portée des variables. Votre moteur de combat repose pourtant encore sur des variables globales éparpillées : le nom, les vies et les potions du joueur ne sont reliés par rien d'autre que leur nom.

Le chapitre suivant règle exactement ce problème. Il introduit la programmation orientée objet : regrouper des données et des fonctions dans une même entité, appelée classe, pour créer autant de joueurs et d'ennemis que nécessaire.

Direction le chapitre suivant : [08-PARTIE-1A—INTRODUCTION-À-LA-POO.md](08-PARTIE-1A—INTRODUCTION-À-LA-POO.md)
