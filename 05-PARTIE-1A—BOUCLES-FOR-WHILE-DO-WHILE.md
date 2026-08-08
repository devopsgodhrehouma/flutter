# PARTIE 1A — DART
# CHAPITRE 05 — LES BOUCLES : `for`, `while`, `do while`, `break` ET `continue`

> **Niveau :** débutant
> **Durée estimée :** 4 h
> **Pré-requis :** chapitre 04 — Conditions (`if`, `else`, `switch`)
> **Ce que vous saurez faire à la fin :** écrire des boucles `for`, `while` et `do while`, contrôler leur déroulement avec `break` et `continue`, et imbriquer des boucles pour parcourir des grilles 2D.

---

## 05.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- comprendre pourquoi les boucles sont nécessaires ;
- répéter une instruction plusieurs fois ;
- utiliser une boucle `for` ;
- comprendre l'initialisation, la condition et l'incrémentation ;
- utiliser une boucle `while` ;
- utiliser une boucle `do while` ;
- utiliser `break` ;
- utiliser `continue` ;
- créer des compteurs ;
- parcourir des valeurs croissantes ou décroissantes ;
- utiliser une boucle avec des conditions ;
- créer des boucles imbriquées ;
- éviter les boucles infinies ;
- choisir entre `for`, `while` et `do while` ;
- préparer la manipulation des listes du chapitre suivant.

---

## 05.1 — Pourquoi avons-nous besoin de boucles ?

Supposons que nous voulions afficher :

```text
Niveau 1
Niveau 2
Niveau 3
Niveau 4
Niveau 5
```

Nous pourrions écrire :

```dart
void main() {
  print('Niveau 1');
  print('Niveau 2');
  print('Niveau 3');
  print('Niveau 4');
  print('Niveau 5');
}
```

Cela fonctionne.

Mais imaginez maintenant que nous voulions afficher :

```text
100 niveaux
```

Écrire 100 instructions `print()` serait inutilement long.

Une boucle permet de dire :

> Répète cette instruction plusieurs fois.

---

## 05.2 — Première boucle `for`

Exemple :

```dart
void main() {
  for (int i = 1; i <= 5; i++) {
    print('Niveau $i');
  }
}
```

**Résultat :**

```text
Niveau 1
Niveau 2
Niveau 3
Niveau 4
Niveau 5
```

Quelques lignes permettent donc de remplacer de nombreuses instructions répétitives.

---

## 05.3 — Structure générale de `for`

Une boucle `for` classique possède cette structure :

```dart
for (initialisation; condition; modification) {
  // instructions
}
```

Exemple :

```dart
for (int i = 1; i <= 5; i++) {
  print(i);
}
```

Les trois parties sont :

```text
int i = 1
i <= 5
i++
```

---

## 05.4 — Comprendre l'initialisation `int i = 1`

Dans :

```dart
for (int i = 1; i <= 5; i++)
```

la première partie :

```dart
int i = 1
```

crée une variable appelée :

```text
i
```

avec la valeur :

```text
1
```

Cette variable est souvent appelée :

> compteur.

---

## 05.5 — Comprendre la condition `i <= 5`

La deuxième partie :

```dart
i <= 5
```

est la condition permettant à la boucle de continuer.

Tant que :

```text
i <= 5
```

est vrai, la boucle continue.

---

## 05.6 — Comprendre l'incrémentation `i++`

La troisième partie :

```dart
i++
```

signifie :

```dart
i = i + 1;
```

Après chaque tour de boucle, la valeur de `i` augmente de 1.

---

## 05.7 — Déroulement complet d'une boucle `for`

Considérons :

```dart
for (int i = 1; i <= 3; i++) {
  print(i);
}
```

Déroulement :

```text
i = 1

1 <= 3 ?
oui
→ afficher 1
→ i devient 2

2 <= 3 ?
oui
→ afficher 2
→ i devient 3

3 <= 3 ?
oui
→ afficher 3
→ i devient 4

4 <= 3 ?
non
→ fin de la boucle
```

**Résultat :**

```text
1
2
3
```

---

## 05.8 — Visualisation du déroulement

```text
Initialisation
     |
     v
   i = 1
     |
     v
Condition vraie ?
     |
 +---+---+
 |       |
oui     non
 |       |
 v       v
Bloc    Fin
 |
 v
Modification
 |
 v
Retour à la condition
```

---

## 05.9 — Compter de 1 à 10

```dart
void main() {
  for (int i = 1; i <= 10; i++) {
    print(i);
  }
}
```

**Résultat :**

```text
1
2
3
4
5
6
7
8
9
10
```

---

## 05.10 — Compter de 0 à 9

```dart
void main() {
  for (int i = 0; i < 10; i++) {
    print(i);
  }
}
```

**Résultat :**

```text
0
1
2
3
4
5
6
7
8
9
```

Cette forme :

```dart
i = 0;
i < 10;
```

sera extrêmement fréquente lorsque nous manipulerons des listes.

---

## 05.11 — Pourquoi beaucoup de programmeurs commencent à zéro ?

Dans de nombreux langages, y compris Dart, les collections utilisent des positions appelées :

> index.

Le premier élément possède généralement l'index :

```text
0
```

Exemple futur :

```text
liste[0] → premier élément
liste[1] → deuxième élément
liste[2] → troisième élément
```

C'est pour cette raison que vous verrez souvent :

```dart
for (int i = 0; i < 10; i++) {
}
```

---

## 05.12 — Compter de deux en deux

Nous ne sommes pas obligés d'utiliser :

```dart
i++
```

On peut écrire :

```dart
i += 2
```

Exemple :

```dart
void main() {
  for (int i = 0; i <= 10; i += 2) {
    print(i);
  }
}
```

**Résultat :**

```text
0
2
4
6
8
10
```

---

## 05.13 — Compter de cinq en cinq

```dart
void main() {
  for (int i = 0; i <= 50; i += 5) {
    print(i);
  }
}
```

**Résultat :**

```text
0
5
10
15
20
25
30
35
40
45
50
```

---

## 05.14 — Compter à rebours

Nous pouvons également diminuer la valeur.

```dart
void main() {
  for (int i = 5; i >= 1; i--) {
    print(i);
  }
}
```

**Résultat :**

```text
5
4
3
2
1
```

---

## 05.15 — Compte à rebours de jeu

```dart
void main() {
  for (int seconde = 5; seconde >= 1; seconde--) {
    print('$seconde...');
  }

  print('GO !');
}
```

**Résultat :**

```text
5...
4...
3...
2...
1...
GO !
```

---

## 05.16 — Répéter un message

```dart
void main() {
  for (int i = 1; i <= 3; i++) {
    print('Bienvenue dans Dart');
  }
}
```

**Résultat :**

```text
Bienvenue dans Dart
Bienvenue dans Dart
Bienvenue dans Dart
```

---

## 05.17 — Utiliser le compteur dans le message

```dart
void main() {
  for (int i = 1; i <= 5; i++) {
    print('Tour numéro $i');
  }
}
```

**Résultat :**

```text
Tour numéro 1
Tour numéro 2
Tour numéro 3
Tour numéro 4
Tour numéro 5
```

---

## 05.18 — Générer plusieurs ennemis

Pour le moment, nous simulons simplement leur création avec `print()`.

```dart
void main() {
  for (int i = 1; i <= 5; i++) {
    print('Création de l’ennemi $i');
  }
}
```

**Résultat :**

```text
Création de l’ennemi 1
Création de l’ennemi 2
Création de l’ennemi 3
Création de l’ennemi 4
Création de l’ennemi 5
```

Plus tard avec Flame, cette logique pourra servir à réellement créer plusieurs objets.

---

## 05.19 — Calculer une somme avec une boucle

Nous pouvons accumuler des valeurs.

Exemple :

```dart
void main() {
  int total = 0;

  for (int i = 1; i <= 5; i++) {
    total += i;
  }

  print(total);
}
```

Calcul :

```text
0 + 1 = 1
1 + 2 = 3
3 + 3 = 6
6 + 4 = 10
10 + 5 = 15
```

**Résultat :**

```text
15
```

---

## 05.20 — Variable accumulatrice

Dans :

```dart
int total = 0;
```

puis :

```dart
total += i;
```

`total` est souvent appelée :

> variable accumulatrice.

Elle permet de conserver un résultat au fil des répétitions.

---

## 05.21 — Exemple : score total

```dart
void main() {
  int score = 0;

  for (int tour = 1; tour <= 5; tour++) {
    score += 100;

    print('Tour $tour : score = $score');
  }
}
```

**Résultat :**

```text
Tour 1 : score = 100
Tour 2 : score = 200
Tour 3 : score = 300
Tour 4 : score = 400
Tour 5 : score = 500
```

---

## 05.22 — Combiner `for` et `if`

Nous pouvons combiner :

```text
for
+
if
```

Exemple :

```dart
void main() {
  for (int i = 1; i <= 10; i++) {
    if (i % 2 == 0) {
      print('$i est pair');
    }
  }
}
```

**Résultat :**

```text
2 est pair
4 est pair
6 est pair
8 est pair
10 est pair
```

---

## 05.23 — Afficher les nombres pairs et impairs

```dart
void main() {
  for (int i = 1; i <= 10; i++) {
    if (i % 2 == 0) {
      print('$i → pair');
    } else {
      print('$i → impair');
    }
  }
}
```

---

## 05.24 — Exemple : boss tous les cinq niveaux

```dart
void main() {
  for (int niveau = 1; niveau <= 20; niveau++) {
    if (niveau % 5 == 0) {
      print('Niveau $niveau → BOSS');
    } else {
      print('Niveau $niveau → normal');
    }
  }
}
```

**Résultat partiel :**

```text
Niveau 1 → normal
Niveau 2 → normal
Niveau 3 → normal
Niveau 4 → normal
Niveau 5 → BOSS
...
Niveau 10 → BOSS
...
Niveau 15 → BOSS
...
Niveau 20 → BOSS
```

---

## 05.25 — La boucle `while`

`while` signifie :

> tant que

Structure :

```dart
while (condition) {
  // instructions
}
```

La boucle continue tant que la condition reste vraie.

---

## 05.26 — Premier exemple avec `while`

```dart
void main() {
  int i = 1;

  while (i <= 5) {
    print(i);

    i++;
  }
}
```

**Résultat :**

```text
1
2
3
4
5
```

---

## 05.27 — Déroulement de `while`

```text
i = 1

condition : i <= 5 ?
        |
      true
        |
        v
   exécuter bloc
        |
        v
      i++
        |
        v
retour condition
```

Lorsque la condition devient fausse, la boucle se termine.

---

## 05.28 — Attention : modifier la condition

Considérons :

```dart
int i = 1;

while (i <= 5) {
  print(i);
}
```

Ici :

```text
i
```

reste toujours égal à :

```text
1
```

Donc :

```text
1 <= 5
```

reste toujours vrai.

La boucle ne se termine jamais.

C'est une :

> boucle infinie.

---

## 05.29 — Correction de la boucle infinie

Il faut faire évoluer la variable :

```dart
void main() {
  int i = 1;

  while (i <= 5) {
    print(i);
    i++;
  }
}
```

---

## 05.30 — Exemple de jeu avec `while`

Imaginons un joueur avec :

```text
3 vies
```

```dart
void main() {
  int vies = 3;

  while (vies > 0) {
    print('Le joueur joue. Vies : $vies');

    vies--;
  }

  print('Game Over');
}
```

**Résultat :**

```text
Le joueur joue. Vies : 3
Le joueur joue. Vies : 2
Le joueur joue. Vies : 1
Game Over
```

---

## 05.31 — Exemple : énergie

```dart
void main() {
  int energie = 100;

  while (energie > 0) {
    print('Énergie : $energie');

    energie -= 20;
  }

  print('Énergie épuisée');
}
```

**Résultat :**

```text
Énergie : 100
Énergie : 80
Énergie : 60
Énergie : 40
Énergie : 20
Énergie épuisée
```

---

## 05.32 — Quand utiliser `while` ?

`while` est pratique lorsque nous ne savons pas exactement à l'avance combien de répétitions seront nécessaires.

Par exemple :

```text
tant que le joueur possède des vies
tant que l'énergie est supérieure à zéro
tant qu'une connexion n'est pas disponible
tant qu'un objectif n'est pas atteint
```

---

## 05.33 — `for` ou `while` ?

Utilisez volontiers `for` lorsque vous connaissez le nombre de répétitions.

Exemple :

```dart
for (int i = 1; i <= 10; i++) {
}
```

Vous savez que vous voulez faire :

```text
10 répétitions
```

Utilisez volontiers `while` lorsque la répétition dépend surtout d'une condition.

Exemple :

```dart
while (vies > 0) {
}
```

---

## 05.34 — La boucle `do while`

Dart possède aussi :

```dart
do {
  // instructions
} while (condition);
```

La différence importante est que le bloc est exécuté :

> au moins une fois.

---

## 05.35 — Premier exemple avec `do while`

```dart
void main() {
  int i = 1;

  do {
    print(i);

    i++;
  } while (i <= 5);
}
```

**Résultat :**

```text
1
2
3
4
5
```

---

## 05.36 — Différence entre `while` et `do while`

Avec `while`, la condition est vérifiée avant d'exécuter le bloc.

```text
condition
   |
   v
bloc
```

Avec `do while`, le bloc est exécuté avant la première vérification.

```text
bloc
 |
 v
condition
```

---

## 05.37 — Exemple important : condition fausse dès le départ

```dart
void main() {
  int nombre = 10;

  while (nombre < 5) {
    print('while');
  }
}
```

**Résultat :**

```text
aucun affichage
```

Pourquoi ?

Car :

```text
10 < 5
```

est faux dès le départ.

---

## 05.38 — Même exemple avec `do while`

```dart
void main() {
  int nombre = 10;

  do {
    print('do while');
  } while (nombre < 5);
}
```

**Résultat :**

```text
do while
```

Même si la condition est fausse, le bloc est exécuté une première fois.

---

## 05.39 — Quand utiliser `do while` ?

`do while` est utile lorsqu'une action doit obligatoirement se produire au moins une fois.

Exemples conceptuels :

```text
afficher un menu une première fois
demander une valeur
lancer une première tentative
effectuer un premier tour avant validation
```

---

## 05.40 — Le mot-clé `break`

`break` permet d'arrêter immédiatement une boucle.

Exemple :

```dart
void main() {
  for (int i = 1; i <= 10; i++) {
    if (i == 5) {
      break;
    }

    print(i);
  }
}
```

**Résultat :**

```text
1
2
3
4
```

Lorsque :

```text
i == 5
```

la boucle s'arrête complètement.

---

## 05.41 — Visualiser `break`

```text
boucle
 |
 v
i == 5 ?
 |
 +--- non ---> continuer
 |
 +--- oui ---> BREAK
                |
                v
           quitter boucle
```

---

## 05.42 — Exemple : trouver un niveau

```dart
void main() {
  for (int niveau = 1; niveau <= 20; niveau++) {
    print('Analyse du niveau $niveau');

    if (niveau == 7) {
      print('Niveau recherché trouvé');
      break;
    }
  }
}
```

La boucle ne continue pas jusqu'au niveau 20.

---

## 05.43 — Exemple avec score et `break`

```dart
void main() {
  int score = 0;

  for (int tour = 1; tour <= 20; tour++) {
    score += 200;

    print('Tour $tour : $score points');

    if (score >= 1000) {
      print('Objectif atteint');
      break;
    }
  }
}
```

---

## 05.44 — Le mot-clé `continue`

`continue` ne termine pas complètement la boucle.

Il signifie :

> passer directement à l'itération suivante.

Exemple :

```dart
void main() {
  for (int i = 1; i <= 5; i++) {
    if (i == 3) {
      continue;
    }

    print(i);
  }
}
```

**Résultat :**

```text
1
2
4
5
```

La valeur :

```text
3
```

a été ignorée.

---

## 05.45 — Différence entre `break` et `continue`

```text
break
→ arrêter complètement la boucle
```

```text
continue
→ ignorer le tour actuel
→ continuer avec le suivant
```

---

## 05.46 — Exemple avec nombres pairs et `continue`

Nous voulons ignorer les nombres impairs.

```dart
void main() {
  for (int i = 1; i <= 10; i++) {
    if (i % 2 != 0) {
      continue;
    }

    print(i);
  }
}
```

**Résultat :**

```text
2
4
6
8
10
```

---

## 05.47 — Exemple avec niveaux interdits

```dart
void main() {
  for (int niveau = 1; niveau <= 10; niveau++) {
    if (niveau == 4 || niveau == 7) {
      print('Niveau $niveau ignoré');
      continue;
    }

    print('Chargement du niveau $niveau');
  }
}
```

---

## 05.48 — Compter les répétitions

```dart
void main() {
  int compteur = 0;

  for (int i = 1; i <= 10; i++) {
    compteur++;
  }

  print('Nombre de répétitions : $compteur');
}
```

**Résultat :**

```text
Nombre de répétitions : 10
```

---

## 05.49 — Compter uniquement certaines valeurs

```dart
void main() {
  int compteurPairs = 0;

  for (int i = 1; i <= 10; i++) {
    if (i % 2 == 0) {
      compteurPairs++;
    }
  }

  print('Nombres pairs : $compteurPairs');
}
```

**Résultat :**

```text
Nombres pairs : 5
```

---

## 05.50 — Calculer une moyenne avec une boucle

Supposons que nous simulions plusieurs scores.

```dart
void main() {
  int total = 0;

  for (int i = 1; i <= 5; i++) {
    total += 100;
  }

  double moyenne = total / 5;

  print('Total : $total');
  print('Moyenne : $moyenne');
}
```

**Résultat :**

```text
Total : 500
Moyenne : 100.0
```

---

## 05.51 — Table de multiplication

```dart
void main() {
  int nombre = 7;

  for (int i = 1; i <= 10; i++) {
    print('$nombre × $i = ${nombre * i}');
  }
}
```

**Résultat :**

```text
7 × 1 = 7
7 × 2 = 14
7 × 3 = 21
...
7 × 10 = 70
```

---

## 05.52 — Boucles imbriquées

Une boucle peut être placée à l'intérieur d'une autre boucle.

Exemple :

```dart
void main() {
  for (int ligne = 1; ligne <= 3; ligne++) {
    for (int colonne = 1; colonne <= 3; colonne++) {
      print('Ligne $ligne - Colonne $colonne');
    }
  }
}
```

---

## 05.53 — Déroulement des boucles imbriquées

Pour :

```text
ligne = 1
```

la boucle intérieure exécute :

```text
colonne 1
colonne 2
colonne 3
```

Puis :

```text
ligne = 2
```

et ainsi de suite.

---

## 05.54 — Résultat des boucles imbriquées

```text
Ligne 1 - Colonne 1
Ligne 1 - Colonne 2
Ligne 1 - Colonne 3

Ligne 2 - Colonne 1
Ligne 2 - Colonne 2
Ligne 2 - Colonne 3

Ligne 3 - Colonne 1
Ligne 3 - Colonne 2
Ligne 3 - Colonne 3
```

---

## 05.55 — Pourquoi les boucles imbriquées seront utiles ?

Elles sont particulièrement utiles pour manipuler :

```text
grilles
tableaux
plateaux de jeu
coordonnées X/Y
matrices
cartes
```

Par exemple, un jeu comportant une grille :

```text
4 lignes × 4 colonnes
```

pourrait utiliser deux boucles.

---

## 05.56 — Exemple de grille 4 × 4

```dart
void main() {
  for (int ligne = 0; ligne < 4; ligne++) {
    for (int colonne = 0; colonne < 4; colonne++) {
      print('Case [$ligne][$colonne]');
    }
  }
}
```

Cela produit :

```text
16 cases
```

car :

```text
4 × 4 = 16
```

---

## 05.57 — Dessiner avec une boucle

Nous pouvons également afficher des motifs simples.

```dart
void main() {
  String ligne = '';

  for (int i = 1; i <= 5; i++) {
    ligne += '*';
    print(ligne);
  }
}
```

**Résultat :**

```text
*
**
***
****
*****
```

---

## 05.58 — Exemple avec des niveaux et des mondes

```dart
void main() {
  for (int monde = 1; monde <= 3; monde++) {
    print('MONDE $monde');

    for (int niveau = 1; niveau <= 4; niveau++) {
      print('  Niveau $monde-$niveau');
    }
  }
}
```

**Résultat :**

```text
MONDE 1
  Niveau 1-1
  Niveau 1-2
  Niveau 1-3
  Niveau 1-4
MONDE 2
  Niveau 2-1
...
```

---

## 05.59 — Une boucle infinie volontaire

Il est techniquement possible d'écrire :

```dart
while (true) {
  // ...
}
```

Cela signifie :

```text
répéter indéfiniment
```

Mais la boucle doit alors normalement contenir une manière d'en sortir. Par exemple :

```dart
void main() {
  int compteur = 0;

  while (true) {
    compteur++;

    print(compteur);

    if (compteur == 5) {
      break;
    }
  }
}
```

**Résultat :**

```text
1
2
3
4
5
```

---

## 05.60 — Attention aux boucles infinies

Une erreur fréquente :

```dart
int i = 1;

while (i <= 10) {
  print(i);
}
```

Le problème est que :

```dart
i
```

ne change jamais.

Une boucle infinie peut :

```text
bloquer un programme
consommer beaucoup de CPU
rendre une application non réactive
```

Il faut toujours vérifier :

```text
Comment ma condition va-t-elle éventuellement devenir fausse ?
```

---

## 05.61 — Exemple complet : simulation de combat

```dart
void main() {
  int vieEnnemi = 100;
  int attaque = 25;
  int tour = 0;

  while (vieEnnemi > 0) {
    tour++;

    vieEnnemi -= attaque;

    if (vieEnnemi < 0) {
      vieEnnemi = 0;
    }

    print('Tour $tour');
    print('Vie de l’ennemi : $vieEnnemi');
  }

  print('Ennemi vaincu');
}
```

**Résultat :**

```text
Tour 1
Vie de l’ennemi : 75

Tour 2
Vie de l’ennemi : 50

Tour 3
Vie de l’ennemi : 25

Tour 4
Vie de l’ennemi : 0

Ennemi vaincu
```

---

## 05.62 — Exemple complet : progression du joueur

```dart
void main() {
  int score = 0;

  for (int niveau = 1; niveau <= 10; niveau++) {
    score += 250;

    print('====================');
    print('Niveau : $niveau');
    print('Score : $score');

    if (niveau % 5 == 0) {
      print('BOSS');
    }

    if (score >= 1500) {
      print('Récompense spéciale débloquée');
    }
  }
}
```

---

## 05.63 — Exemple complet : vies et Game Over

```dart
void main() {
  int vies = 5;
  int tour = 1;

  while (vies > 0) {
    print('Tour $tour');
    print('Vies avant collision : $vies');

    vies--;

    print('Vies après collision : $vies');

    if (vies == 1) {
      print('Attention : dernière vie !');
    }

    tour++;
  }

  print('Game Over');
}
```

---

## 05.64 — Choisir la bonne boucle

**Utilisez `for`** lorsque vous savez combien de répétitions vous souhaitez.

Exemple :

```dart
for (int i = 0; i < 10; i++) {
}
```

**Utilisez `while`** lorsque vous souhaitez répéter tant qu'une condition est vraie.

Exemple :

```dart
while (vies > 0) {
}
```

**Utilisez `do while`** lorsque le bloc doit être exécuté au moins une fois.

Exemple :

```dart
do {
  // afficher un menu
} while (condition);
```

---

## 05.65 — Fiche mémo des boucles

Structure de `for` :

```dart
for (initialisation; condition; modification) {
  // instructions
}
```

Exemple :

```dart
for (int i = 1; i <= 5; i++) {
  print(i);
}
```

Compter à rebours :

```dart
for (int i = 10; i >= 1; i--) {
  print(i);
}
```

Structure de `while` :

```dart
while (condition) {
  // instructions
}
```

Exemple :

```dart
while (vies > 0) {
  vies--;
}
```

Structure de `do while` :

```dart
do {
  // instructions
} while (condition);
```

Le bloc s'exécute au minimum une fois.

`break` arrête immédiatement la boucle :

```dart
if (condition) {
  break;
}
```

```text
sortir immédiatement de la boucle
```

`continue` ignore uniquement le tour en cours :

```dart
if (condition) {
  continue;
}
```

```text
ignorer l'itération actuelle
et passer à la suivante
```

Pour choisir une boucle :

```text
for
→ nombre de répétitions connu

while
→ répétition contrôlée par une condition

do while
→ exécution obligatoire au moins une fois
```

---

## 05.66 — Lien avec Flutter et notre futur jeu

Ces notions seront directement utiles.

Aujourd'hui :

```dart
for (int i = 1; i <= 5; i++) {
  print('Ennemi $i');
}
```

Plus tard avec Flame, le même principe pourra servir à :

```text
créer plusieurs ennemis
créer plusieurs pièces
générer des plateformes
charger plusieurs niveaux
parcourir des objets
analyser les collisions
```

Les boucles imbriquées pourront notamment être utiles pour :

```text
grilles 2D
maps
plateaux
cases
coordonnées X/Y
```

---

## 05.67 — Aperçu du chapitre suivant : les collections

Nous allons maintenant passer d'une seule valeur :

```dart
String ennemi = 'Zombie';
```

à plusieurs valeurs :

```dart
List<String> ennemis = [
  'Zombie',
  'Robot',
  'Dragon',
];
```

Nous apprendrons à :

```text
créer une liste
ajouter un élément
supprimer un élément
modifier un élément
accéder à un élément avec son index
obtenir la taille d'une liste
parcourir une liste
utiliser for-in
utiliser List
utiliser Set
utiliser Map
```

Exemple :

```dart
List<String> joueurs = [
  'Alex',
  'Sophie',
  'Samir',
];

for (String joueur in joueurs) {
  print(joueur);
}
```

Puis :

```dart
Map<String, int> scores = {
  'Alex': 1200,
  'Sophie': 1800,
  'Samir': 950,
};
```

Les collections sont parmi les notions les plus importantes de Dart et seront omniprésentes dans Flutter.

---

## 05.68 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| Oublier de faire évoluer la variable dans un `while` | La condition reste toujours vraie car la variable ne change jamais | Toujours modifier la variable testée à l'intérieur du bloc (`i++`, `vies--`, etc.) |
| Confondre `break` et `continue` | `break` arrête toute la boucle, `continue` ignore seulement le tour actuel | Se demander : « je veux arrêter complètement » (`break`) ou « je veux juste sauter ce tour » (`continue`) |
| Utiliser `do while` alors qu'un `while` suffirait | Le bloc de `do while` s'exécute toujours au moins une fois, même si la condition est fausse au départ | Réserver `do while` aux cas où une exécution minimale d'une fois est réellement nécessaire |
| Se tromper entre `<` et `<=` dans la condition | La borne finale n'est pas parcourue, ou est parcourue une fois de trop | Vérifier avec un petit déroulement manuel (par exemple `i = 1 à 5`) que la dernière valeur voulue est bien incluse |
| Oublier que `for` peut aussi décompter | On pense que `for` ne sert qu'à compter vers le haut | Utiliser `i--` dans la modification pour compter à rebours |
| Imbriquer des boucles sans renommer les compteurs | Réutiliser `i` dans les deux boucles imbriquées provoque des conflits de logique | Nommer clairement chaque compteur (`ligne`, `colonne`, `monde`, `niveau`, etc.) |
| Écrire une boucle infinie volontaire sans `break` | `while (true)` sans condition de sortie bloque le programme | Toujours prévoir un `break` avec une condition claire à l'intérieur d'une boucle `while (true)` |
| Laisser une valeur négative après une boucle de combat | Soustraire un montant fixe peut faire passer une vie ou une énergie sous zéro | Vérifier et ramener la valeur à `0` après la soustraction, comme dans les exemples de combat |

---

## 05.69 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| Boucle | Permet de répéter automatiquement des instructions sans les recopier |
| `for` | Structure `for (initialisation; condition; modification)`, idéale quand le nombre de répétitions est connu |
| Initialisation | Crée et donne une valeur de départ au compteur (`int i = 1`) |
| Condition | Tant qu'elle est vraie, la boucle continue (`i <= 5`) |
| Modification | Fait évoluer le compteur à chaque tour (`i++`, `i--`, `i += 2`) |
| Compter à rebours | `for (int i = 10; i >= 1; i--)` |
| Pas de 2 ou de 5 | `i += 2` ou `i += 5` dans la partie modification |
| Variable accumulatrice | Variable initialisée avant la boucle puis modifiée à chaque tour pour cumuler un résultat (`total += i`) |
| `for` + `if` | Permet de filtrer ou d'agir seulement sur certaines valeurs pendant la répétition |
| `while` | Structure `while (condition)`, idéale quand le nombre de répétitions dépend d'une condition |
| Boucle infinie | Se produit quand la condition d'un `while` ne devient jamais fausse ; toujours faire évoluer la variable testée |
| `do while` | Exécute le bloc au moins une fois, avant même de vérifier la condition |
| `break` | Arrête complètement la boucle |
| `continue` | Ignore uniquement le tour actuel et passe au suivant |
| Boucles imbriquées | Une boucle à l'intérieur d'une autre, utile pour les grilles, plateaux et coordonnées X/Y |
| Choisir sa boucle | `for` (nombre connu), `while` (condition), `do while` (au moins une exécution) |

---

## 05.70 — Exercices

### Exercice 1 — Compter jusqu'à 10 (facile)

Utilisez une boucle `for` pour afficher :

```text
1
2
3
4
5
6
7
8
9
10
```

---

### Exercice 2 — Compter de 0 à 20 (facile)

Utilisez :

```dart
for
```

pour afficher les nombres de :

```text
0 à 20
```

---

### Exercice 3 — Compte à rebours (facile)

Affichez :

```text
10
9
8
7
6
5
4
3
2
1
GO !
```

avec une boucle.

---

### Exercice 4 — Nombres pairs (facile)

Affichez tous les nombres pairs entre :

```text
0 et 20
```

Deux solutions sont possibles :

```text
i += 2
```

ou :

```text
i % 2 == 0
```

---

### Exercice 5 — Nombres impairs (facile)

Affichez tous les nombres impairs de :

```text
1 à 19
```

---

### Exercice 6 — Table de multiplication (moyen)

Créez :

```dart
int nombre = 5;
```

Affichez :

```text
5 × 1 = 5
5 × 2 = 10
...
5 × 10 = 50
```

---

### Exercice 7 — Addition de 1 à 100 (moyen)

Calculez :

```text
1 + 2 + 3 + ... + 100
```

à l'aide d'une boucle.

---

### Exercice 8 — Score (moyen)

Créez :

```dart
int score = 0;
```

Pendant 10 tours :

```text
ajoutez 100 points
```

Affichez le score après chaque tour.

---

### Exercice 9 — Boss (moyen)

Affichez les niveaux :

```text
1 à 30
```

et affichez :

```text
BOSS
```

pour chaque niveau multiple de :

```text
5
```

---

### Exercice 10 — Boucle `while` (facile)

Avec :

```dart
int compteur = 1;
```

utilisez `while` pour afficher :

```text
1 à 10
```

---

### Exercice 11 — Énergie (moyen)

Créez :

```dart
int energie = 100;
```

Tant que l'énergie est supérieure à zéro :

```text
affichez l'énergie
retirez 10
```

Puis affichez :

```text
Énergie épuisée
```

---

### Exercice 12 — Vies (moyen)

Créez :

```dart
int vies = 3;
```

Utilisez `while` pour simuler une perte de vie jusqu'au `Game Over`.

Résultat possible :

```text
Vies : 3
Vies : 2
Vies : 1
Game Over
```

---

### Exercice 13 — `do while` (facile)

Créez :

```dart
int nombre = 10;
```

Utilisez `do while` avec la condition :

```dart
nombre < 5
```

Observez pourquoi le message est tout de même affiché une fois.

---

### Exercice 14 — `break` (moyen)

Affichez les nombres de :

```text
1 à 100
```

mais arrêtez la boucle lorsque :

```text
i == 8
```

---

### Exercice 15 — `continue` (moyen)

Affichez les nombres de :

```text
1 à 10
```

mais ignorez :

```text
5
```

---

### Exercice 16 — Ignorer les nombres pairs (moyen)

Utilisez `continue` pour afficher uniquement les nombres impairs entre :

```text
1 et 20
```

---

### Exercice 17 — Chercher un objectif (moyen)

Simulez une progression :

```text
+250 points par tour
```

Arrêtez la boucle avec `break` lorsque le score atteint ou dépasse :

```text
2000
```

Affichez le nombre de tours nécessaires.

---

### Exercice 18 — Grille (moyen)

Utilisez deux boucles imbriquées pour afficher toutes les coordonnées d'une grille :

```text
3 × 3
```

Format attendu :

```text
[0,0]
[0,1]
[0,2]
[1,0]
...
```

---

### Exercice 19 — Mondes et niveaux (moyen)

Créez :

```text
3 mondes
```

avec :

```text
5 niveaux par monde
```

Affichez :

```text
Monde 1 - Niveau 1
Monde 1 - Niveau 2
...
Monde 3 - Niveau 5
```

---

### Exercice 20 — Challenge combat (difficile)

Créez :

```dart
int vieEnnemi = 150;
int degats = 30;
```

À chaque tour :

```text
retirer 30 points de vie
afficher le numéro du tour
afficher la vie restante
```

Lorsque l'ennemi atteint :

```text
0 vie
```

affichez :

```text
Ennemi vaincu
```

La vie affichée ne doit jamais devenir négative.

---

## 05.71 — Corrections des exercices

### Correction 1

```dart
void main() {
  for (int i = 1; i <= 10; i++) {
    print(i);
  }
}
```

**Explication :** la boucle part de `1`, continue tant que `i <= 10`, et incrémente `i` de 1 à chaque tour.

---

### Correction 2

```dart
void main() {
  for (int i = 0; i <= 20; i++) {
    print(i);
  }
}
```

**Explication :** en partant de `0` et en utilisant `<=`, la valeur `20` est bien affichée.

---

### Correction 3

```dart
void main() {
  for (int i = 10; i >= 1; i--) {
    print(i);
  }

  print('GO !');
}
```

**Explication :** le compteur décroît avec `i--` jusqu'à ce que la condition `i >= 1` devienne fausse, puis `GO !` est affiché après la boucle.

---

### Correction 4

Première solution :

```dart
void main() {
  for (int i = 0; i <= 20; i += 2) {
    print(i);
  }
}
```

Deuxième solution :

```dart
void main() {
  for (int i = 0; i <= 20; i++) {
    if (i % 2 == 0) {
      print(i);
    }
  }
}
```

**Explication :** la première solution avance directement de 2 en 2, la seconde parcourt tous les nombres et filtre les pairs avec `%`.

---

### Correction 5

```dart
void main() {
  for (int i = 1; i <= 19; i += 2) {
    print(i);
  }
}
```

**Explication :** en partant d'un nombre impair (`1`) et en avançant de 2 en 2, seuls des nombres impairs sont produits.

---

### Correction 6

```dart
void main() {
  int nombre = 5;

  for (int i = 1; i <= 10; i++) {
    print('$nombre × $i = ${nombre * i}');
  }
}
```

**Explication :** le compteur `i` sert à la fois à afficher le multiplicateur et à calculer le résultat avec `nombre * i`.

---

### Correction 7

```dart
void main() {
  int total = 0;

  for (int i = 1; i <= 100; i++) {
    total += i;
  }

  print('Total : $total');
}
```

**Résultat :**

```text
Total : 5050
```

**Explication :** `total` est une variable accumulatrice qui additionne chaque valeur de `i` de 1 à 100.

---

### Correction 8

```dart
void main() {
  int score = 0;

  for (int tour = 1; tour <= 10; tour++) {
    score += 100;

    print('Tour $tour : $score points');
  }
}
```

**Explication :** `score` augmente de 100 à chaque tour, et le résultat est affiché immédiatement dans la boucle.

---

### Correction 9

```dart
void main() {
  for (int niveau = 1; niveau <= 30; niveau++) {
    if (niveau % 5 == 0) {
      print('Niveau $niveau → BOSS');
    } else {
      print('Niveau $niveau');
    }
  }
}
```

**Explication :** `niveau % 5 == 0` est vrai uniquement pour les multiples de 5.

---

### Correction 10

```dart
void main() {
  int compteur = 1;

  while (compteur <= 10) {
    print(compteur);

    compteur++;
  }
}
```

**Explication :** `compteur` est incrémenté manuellement dans le bloc `while`, contrairement au `for` où l'incrémentation est intégrée à la structure.

---

### Correction 11

```dart
void main() {
  int energie = 100;

  while (energie > 0) {
    print('Énergie : $energie');

    energie -= 10;
  }

  print('Énergie épuisée');
}
```

**Explication :** la boucle continue tant que `energie > 0`, et s'arrête naturellement quand l'énergie atteint 0.

---

### Correction 12

```dart
void main() {
  int vies = 3;

  while (vies > 0) {
    print('Vies : $vies');

    vies--;
  }

  print('Game Over');
}
```

**Explication :** chaque tour de boucle retire une vie ; `Game Over` s'affiche seulement après la fin de la boucle.

---

### Correction 13

```dart
void main() {
  int nombre = 10;

  do {
    print('Le bloc est exécuté.');
  } while (nombre < 5);
}
```

**Explication :** `do while` exécute le bloc avant de vérifier la condition. Même si `10 < 5` est faux, le message s'affiche une fois.

---

### Correction 14

```dart
void main() {
  for (int i = 1; i <= 100; i++) {
    if (i == 8) {
      break;
    }

    print(i);
  }
}
```

**Résultat :**

```text
1
2
3
4
5
6
7
```

**Explication :** `break` interrompt la boucle dès que `i` atteint 8, avant même d'afficher cette valeur.

---

### Correction 15

```dart
void main() {
  for (int i = 1; i <= 10; i++) {
    if (i == 5) {
      continue;
    }

    print(i);
  }
}
```

**Explication :** `continue` saute uniquement l'itération où `i == 5` ; la boucle reprend normalement ensuite.

---

### Correction 16

```dart
void main() {
  for (int i = 1; i <= 20; i++) {
    if (i % 2 == 0) {
      continue;
    }

    print(i);
  }
}
```

**Explication :** chaque nombre pair déclenche `continue`, donc seuls les nombres impairs sont affichés.

---

### Correction 17

```dart
void main() {
  int score = 0;

  for (int tour = 1; tour <= 100; tour++) {
    score += 250;

    print('Tour $tour : $score points');

    if (score >= 2000) {
      print('Objectif atteint au tour $tour');
      break;
    }
  }
}
```

**Explication :** la boucle `for` sert de filet de sécurité (100 tours maximum), tandis que `break` arrête réellement la progression dès que l'objectif est atteint.

---

### Correction 18

```dart
void main() {
  for (int ligne = 0; ligne < 3; ligne++) {
    for (int colonne = 0; colonne < 3; colonne++) {
      print('[$ligne,$colonne]');
    }
  }
}
```

**Explication :** la boucle extérieure parcourt les lignes, et pour chaque ligne, la boucle intérieure parcourt toutes les colonnes.

---

### Correction 19

```dart
void main() {
  for (int monde = 1; monde <= 3; monde++) {
    for (int niveau = 1; niveau <= 5; niveau++) {
      print('Monde $monde - Niveau $niveau');
    }
  }
}
```

**Explication :** deux boucles imbriquées permettent de générer toutes les combinaisons monde/niveau sans les écrire à la main.

---

### Correction 20

```dart
void main() {
  int vieEnnemi = 150;
  int degats = 30;
  int tour = 0;

  while (vieEnnemi > 0) {
    tour++;

    vieEnnemi -= degats;

    if (vieEnnemi < 0) {
      vieEnnemi = 0;
    }

    print('Tour $tour');
    print('Vie restante : $vieEnnemi');
  }

  print('Ennemi vaincu');
}
```

**Explication :** la vérification `if (vieEnnemi < 0) { vieEnnemi = 0; }` empêche l'affichage d'une vie négative, même si les dégâts dépassent la vie restante lors du dernier tour.

---

## Et maintenant ?

Vous savez désormais répéter des instructions avec `for`, `while` et `do while`, contrôler leur déroulement avec `break` et `continue`, et imbriquer des boucles pour parcourir des grilles. Le chapitre suivant vous apprend à manipuler plusieurs valeurs à la fois grâce aux collections `List`, `Set` et `Map`, que vous parcourrez justement à l'aide des boucles vues ici.

Direction le chapitre suivant : [06-PARTIE-1A—COLLECTIONS-LIST-SET-MAP.md](./06-PARTIE-1A—COLLECTIONS-LIST-SET-MAP.md)
