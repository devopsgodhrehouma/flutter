# PARTIE 1A — DART
# CHAPITRE 04 — CONDITIONS : `if`, `else if`, `else` ET `switch`

> **Niveau :** débutant
> **Durée estimée :** 3 h
> **Pré-requis :** chapitre 03 — Opérateurs et expressions
> **Ce que vous saurez faire à la fin :** faire réagir un programme aux données grâce à `if`, `else if`, `else` et `switch`, y compris le `switch` expression moderne de Dart.

---

## 04.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- comprendre ce qu'est une condition ;
- utiliser `if` ;
- utiliser `else` ;
- utiliser `else if` ;
- combiner plusieurs conditions ;
- créer des conditions imbriquées ;
- utiliser les opérateurs logiques dans une condition ;
- utiliser `switch` ;
- comprendre quand utiliser `if` ou `switch` ;
- créer des systèmes de décision ;
- construire une logique simple de jeu ;
- afficher un `Game Over` ;
- gérer différents niveaux ;
- attribuer des récompenses ;
- contrôler l'accès à certaines fonctionnalités.

---

## 04.1 — Pourquoi avons-nous besoin de conditions ?

Jusqu'à maintenant, nos programmes exécutaient les instructions dans l'ordre.

Exemple :

```dart
void main() {
  print('Début');
  print('Joueur');
  print('Fin');
}
```

Le programme exécute toujours :

```text
Début
Joueur
Fin
```

Mais dans une vraie application, nous avons besoin de décisions.

Par exemple :

```text
SI le joueur possède encore des vies
    continuer la partie

SINON
    afficher Game Over
```

Ou :

```text
SI l'utilisateur est connecté
    afficher son profil

SINON
    afficher l'écran de connexion
```

Ou encore :

```text
SI le score dépasse 1000
    débloquer le niveau suivant
```

C'est le rôle des conditions.

---

## 04.2 — Première condition avec `if`

La syntaxe générale est :

```dart
if (condition) {
  // instructions
}
```

Exemple :

```dart
void main() {
  int age = 20;

  if (age >= 18) {
    print('Accès autorisé');
  }
}
```

Résultat :

```text
Accès autorisé
```

Pourquoi ?

Parce que :

```text
20 >= 18
```

donne :

```text
true
```

---

## 04.3 — Comment fonctionne `if` ?

Considérons :

```dart
if (age >= 18) {
  print('Accès autorisé');
}
```

Dart évalue :

```dart
age >= 18
```

Deux possibilités existent.

```text
condition
   |
   +------ true ------> exécuter le bloc
   |
   +------ false -----> ignorer le bloc
```

---

## 04.4 — Exemple avec une condition fausse

```dart
void main() {
  int age = 15;

  if (age >= 18) {
    print('Accès autorisé');
  }

  print('Fin du programme');
}
```

Résultat :

```text
Fin du programme
```

Le message :

```text
Accès autorisé
```

n'est pas affiché.

La condition était fausse.

---

## 04.5 — Utiliser un booléen directement

Supposons :

```dart
bool compteActif = true;
```

Nous pouvons écrire :

```dart
if (compteActif) {
  print('Compte actif');
}
```

Il n'est pas nécessaire d'écrire :

```dart
if (compteActif == true)
```

La version :

```dart
if (compteActif)
```

est plus naturelle.

---

## 04.6 — Exemple

```dart
void main() {
  bool utilisateurConnecte = true;

  if (utilisateurConnecte) {
    print('Bienvenue dans votre compte.');
  }
}
```

---

## 04.7 — Utiliser `!` dans une condition

Supposons :

```dart
bool connecte = false;
```

On peut écrire :

```dart
if (!connecte) {
  print('Veuillez vous connecter.');
}
```

`!connecte` signifie :

```text
NON connecté
```

---

## 04.8 — Premier exemple de jeu

```dart
void main() {
  int vies = 3;

  if (vies > 0) {
    print('Le joueur peut continuer.');
  }
}
```

---

## 04.9 — Condition Game Over

```dart
void main() {
  int vies = 0;

  if (vies <= 0) {
    print('Game Over');
  }
}
```

Résultat :

```text
Game Over
```

---

## 04.10 — Le mot-clé `else`

`else` signifie :

> sinon

Structure :

```dart
if (condition) {
  // si vrai
} else {
  // sinon
}
```

---

## 04.11 — Exemple simple

```dart
void main() {
  int age = 16;

  if (age >= 18) {
    print('Majeur');
  } else {
    print('Mineur');
  }
}
```

Résultat :

```text
Mineur
```

---

## 04.12 — Visualiser `if / else`

```text
               condition
                   |
          +--------+--------+
          |                 |
        true              false
          |                 |
          v                 v
      bloc if           bloc else
```

Un seul des deux blocs est exécuté.

---

## 04.13 — Exemple Game Over complet

```dart
void main() {
  int vies = 2;

  if (vies > 0) {
    print('Continuez à jouer.');
  } else {
    print('Game Over');
  }
}
```

Résultat :

```text
Continuez à jouer.
```

---

## 04.14 — Autre exemple

```dart
void main() {
  int score = 1200;

  if (score >= 1000) {
    print('Niveau débloqué.');
  } else {
    print('Score insuffisant.');
  }
}
```

---

## 04.15 — `else if`

Parfois, deux possibilités ne suffisent pas.

Exemple :

```text
score >= 1000
→ excellent

score >= 500
→ bon

sinon
→ à améliorer
```

Nous pouvons utiliser :

```dart
else if
```

---

## 04.16 — Premier exemple avec `else if`

```dart
void main() {
  int score = 750;

  if (score >= 1000) {
    print('Excellent');
  } else if (score >= 500) {
    print('Bon');
  } else {
    print('Continuez');
  }
}
```

Résultat :

```text
Bon
```

---

## 04.17 — L'ordre des conditions est important

Considérons :

```dart
int score = 1200;
```

Puis :

```dart
if (score >= 500) {
  print('Bon');
} else if (score >= 1000) {
  print('Excellent');
}
```

Résultat :

```text
Bon
```

C'est un problème.

Pourquoi ?

Parce que :

```text
1200 >= 500
```

est déjà vrai.

Dart exécute ce premier bloc puis ignore les autres.

---

## 04.18 — Bonne organisation

Il faut tester la condition la plus restrictive en premier.

Correct :

```dart
if (score >= 1000) {
  print('Excellent');
} else if (score >= 500) {
  print('Bon');
} else {
  print('Continuez');
}
```

---

## 04.19 — Exemple de système de notes

```dart
void main() {
  double note = 82;

  if (note >= 90) {
    print('A');
  } else if (note >= 80) {
    print('B');
  } else if (note >= 70) {
    print('C');
  } else if (note >= 60) {
    print('D');
  } else {
    print('Échec');
  }
}
```

Résultat :

```text
B
```

---

## 04.20 — Comprendre le cheminement

Avec :

```text
note = 82
```

Dart vérifie :

```text
82 >= 90 ?
false

82 >= 80 ?
true
```

Dart affiche :

```text
B
```

Puis s'arrête dans cette chaîne `if / else if / else`.

---

## 04.21 — Conditions avec `&&`

Nous pouvons utiliser :

```text
&&
```

pour vérifier plusieurs critères.

Exemple :

```dart
void main() {
  int age = 25;
  bool compteActif = true;

  if (age >= 18 && compteActif) {
    print('Accès autorisé');
  } else {
    print('Accès refusé');
  }
}
```

---

## 04.22 — Exemple de jeu avec `&&`

Un joueur peut entrer dans une zone si :

```text
niveau >= 5
ET
possède une clé
```

```dart
void main() {
  int niveau = 7;
  bool possedeCle = true;

  if (niveau >= 5 && possedeCle) {
    print('Zone débloquée');
  } else {
    print('Zone verrouillée');
  }
}
```

---

## 04.23 — Conditions avec `||`

`||` signifie :

> OU

Exemple :

```dart
void main() {
  bool administrateur = false;
  bool moderateur = true;

  if (administrateur || moderateur) {
    print('Accès au panneau de gestion');
  }
}
```

---

## 04.24 — Exemple de victoire

Un joueur gagne s'il :

```text
atteint 10 000 points
OU
bat le boss final
```

```dart
void main() {
  int score = 8500;
  bool bossFinalBattu = true;

  if (score >= 10000 || bossFinalBattu) {
    print('Victoire !');
  } else {
    print('Continuez la partie.');
  }
}
```

---

## 04.25 — Combiner `&&`, `||` et `!`

Exemple :

```dart
void main() {
  int vies = 2;
  double energie = 50;
  bool pause = false;

  if (vies > 0 && energie > 0 && !pause) {
    print('La partie continue.');
  } else {
    print('Le joueur ne peut pas continuer.');
  }
}
```

---

## 04.26 — Utiliser des parenthèses

Pour améliorer la lisibilité :

```dart
if ((vies > 0) && (energie > 0) && !pause) {
  print('Continuer');
}
```

---

## 04.27 — Condition plus complexe

Supposons que le joueur puisse entrer si :

```text
(niveau >= 10 ET possède la clé)
OU
est administrateur
```

Code :

```dart
void main() {
  int niveau = 7;
  bool possedeCle = false;
  bool administrateur = true;

  if ((niveau >= 10 && possedeCle) || administrateur) {
    print('Accès autorisé');
  } else {
    print('Accès refusé');
  }
}
```

---

## 04.28 — Conditions imbriquées

Une condition peut se trouver à l'intérieur d'une autre condition.

Exemple :

```dart
void main() {
  bool connecte = true;
  bool administrateur = true;

  if (connecte) {
    print('Utilisateur connecté');

    if (administrateur) {
      print('Accès administrateur');
    }
  }
}
```

---

## 04.29 — Visualiser une condition imbriquée

```text
Utilisateur connecté ?
        |
       oui
        |
        v
Administrateur ?
        |
       oui
        |
        v
Afficher panneau administrateur
```

---

## 04.30 — Exemple de jeu imbriqué

```dart
void main() {
  int vies = 3;
  int score = 1500;

  if (vies > 0) {
    print('Joueur vivant');

    if (score >= 1000) {
      print('Niveau suivant débloqué');
    }
  } else {
    print('Game Over');
  }
}
```

---

## 04.31 — Ne pas abuser des conditions imbriquées

Ceci devient rapidement difficile à lire :

```dart
if (condition1) {
  if (condition2) {
    if (condition3) {
      if (condition4) {
        // ...
      }
    }
  }
}
```

Nous verrons progressivement comment rendre le code plus lisible grâce aux fonctions.

---

## 04.32 — Exemple : énergie du joueur

```dart
void main() {
  double energie = 65;

  if (energie >= 80) {
    print('Énergie élevée');
  } else if (energie >= 50) {
    print('Énergie moyenne');
  } else if (energie > 0) {
    print('Énergie faible');
  } else {
    print('Aucune énergie');
  }
}
```

---

## 04.33 — Exemple : nombre de vies

```dart
void main() {
  int vies = 3;

  if (vies >= 3) {
    print('Très bon état');
  } else if (vies == 2) {
    print('Attention');
  } else if (vies == 1) {
    print('Danger');
  } else {
    print('Game Over');
  }
}
```

---

## 04.34 — Exemple : niveau du joueur

```dart
void main() {
  int niveau = 12;

  if (niveau >= 20) {
    print('Maître');
  } else if (niveau >= 10) {
    print('Avancé');
  } else if (niveau >= 5) {
    print('Intermédiaire');
  } else {
    print('Débutant');
  }
}
```

---

## 04.35 — Exemple : récompense

```dart
void main() {
  int score = 2500;

  if (score >= 5000) {
    print('Récompense légendaire');
  } else if (score >= 2500) {
    print('Récompense épique');
  } else if (score >= 1000) {
    print('Récompense rare');
  } else {
    print('Récompense standard');
  }
}
```

---

## 04.36 — Vérifier si un nombre est pair

Nous avons appris :

```dart
nombre % 2
```

Nous pouvons maintenant écrire :

```dart
void main() {
  int nombre = 14;

  if (nombre % 2 == 0) {
    print('Nombre pair');
  } else {
    print('Nombre impair');
  }
}
```

---

## 04.37 — Détection d'un niveau boss

```dart
void main() {
  int niveau = 15;

  if (niveau % 5 == 0) {
    print('Niveau Boss');
  } else {
    print('Niveau normal');
  }
}
```

---

## 04.38 — Vérifier une plage de valeurs

Supposons que nous voulions vérifier si une note est entre 0 et 100.

```dart
void main() {
  double note = 85;

  if (note >= 0 && note <= 100) {
    print('Note valide');
  } else {
    print('Note invalide');
  }
}
```

---

## 04.39 — Vérifier une valeur hors plage

```dart
void main() {
  double note = 150;

  if (note < 0 || note > 100) {
    print('Note invalide');
  } else {
    print('Note valide');
  }
}
```

---

## 04.40 — Exemple de sécurité

```dart
void main() {
  String motDePasse = 'dart123';

  if (motDePasse == 'dart123') {
    print('Connexion réussie');
  } else {
    print('Mot de passe incorrect');
  }
}
```

---

## 04.41 — Comparaison de chaînes

On peut comparer des chaînes avec :

```text
==
```

Exemple :

```dart
void main() {
  String role = 'admin';

  if (role == 'admin') {
    print('Administrateur');
  }
}
```

---

## 04.42 — Les majuscules comptent

Dart est sensible à la casse.

Donc :

```dart
'admin'
```

et :

```dart
'Admin'
```

ne sont pas identiques.

Exemple :

```dart
void main() {
  String role = 'Admin';

  print(role == 'admin');
}
```

Résultat :

```text
false
```

---

## 04.43 — Convertir une chaîne en minuscules

Une solution possible :

```dart
void main() {
  String role = 'ADMIN';

  if (role.toLowerCase() == 'admin') {
    print('Administrateur');
  }
}
```

---

## 04.44 — Introduction à `switch`

Lorsque nous voulons comparer une même variable avec plusieurs valeurs précises, `switch` peut être plus lisible.

Exemple :

```text
niveau = 1
→ Débutant

niveau = 2
→ Intermédiaire

niveau = 3
→ Avancé
```

---

## 04.45 — Syntaxe de base de `switch`

```dart
switch (variable) {
  case valeur1:
    // instructions
    break;

  case valeur2:
    // instructions
    break;

  default:
    // instructions
}
```

---

## 04.46 — Premier exemple

```dart
void main() {
  int choix = 2;

  switch (choix) {
    case 1:
      print('Nouveau jeu');
      break;

    case 2:
      print('Continuer');
      break;

    case 3:
      print('Options');
      break;

    default:
      print('Choix invalide');
  }
}
```

Résultat :

```text
Continuer
```

---

## 04.47 — Comprendre `case`

Chaque `case` représente une valeur possible.

Exemple :

```dart
case 1:
```

signifie :

```text
si la valeur est 1
```

---

## 04.48 — Le rôle de `default`

`default` est exécuté lorsqu'aucun `case` ne correspond.

Exemple :

```dart
void main() {
  int choix = 8;

  switch (choix) {
    case 1:
      print('Jouer');
      break;

    case 2:
      print('Options');
      break;

    default:
      print('Choix inconnu');
  }
}
```

Résultat :

```text
Choix inconnu
```

---

## 04.49 — Exemple avec une chaîne

```dart
void main() {
  String difficulte = 'difficile';

  switch (difficulte) {
    case 'facile':
      print('Mode facile activé');
      break;

    case 'normal':
      print('Mode normal activé');
      break;

    case 'difficile':
      print('Mode difficile activé');
      break;

    default:
      print('Difficulté inconnue');
  }
}
```

---

## 04.50 — Menu de jeu avec `switch`

```dart
void main() {
  int menu = 1;

  switch (menu) {
    case 1:
      print('Commencer une nouvelle partie');
      break;

    case 2:
      print('Continuer la partie');
      break;

    case 3:
      print('Afficher les paramètres');
      break;

    case 4:
      print('Quitter');
      break;

    default:
      print('Option invalide');
  }
}
```

---

## 04.51 — `if` ou `switch` ?

Utilisez généralement `if` lorsque vous testez :

```text
>
<
>=
<=
&&
||
```

Exemple :

```dart
if (score >= 1000) {
  // ...
}
```

Utilisez volontiers `switch` lorsqu'une variable peut prendre plusieurs valeurs précises.

Exemple :

```dart
switch (difficulte) {
  case 'facile':
  case 'normal':
  case 'difficile':
}
```

---

## 04.52 — Comparaison

### Plus adapté à `if`

```dart
if (age >= 18) {
  print('Majeur');
}
```

### Plus adapté à `switch`

```dart
switch (role) {
  case 'admin':
    print('Administrateur');
    break;

  case 'user':
    print('Utilisateur');
    break;
}
```

---

## 04.53 — `switch` moderne sous forme d'expression

Dart moderne permet également d'utiliser `switch` pour produire directement une valeur.

Exemple :

```dart
void main() {
  String difficulte = 'normal';

  String message = switch (difficulte) {
    'facile' => 'Mode facile',
    'normal' => 'Mode normal',
    'difficile' => 'Mode difficile',
    _ => 'Mode inconnu',
  };

  print(message);
}
```

---

## 04.54 — Comprendre `_`

Dans ce type de `switch` :

```dart
_ => 'Mode inconnu'
```

le symbole :

```text
_
```

représente ici :

> toutes les autres valeurs.

C'est comparable à `default`.

---

## 04.55 — Exemple moderne avec un nombre

```dart
void main() {
  int choix = 2;

  String action = switch (choix) {
    1 => 'Jouer',
    2 => 'Options',
    3 => 'Quitter',
    _ => 'Choix invalide',
  };

  print(action);
}
```

---

## 04.56 — Pourquoi cette syntaxe est intéressante ?

Parce que nous pouvons produire directement une valeur.

Au lieu de :

```dart
String action;

switch (choix) {
  case 1:
    action = 'Jouer';
    break;

  case 2:
    action = 'Options';
    break;

  default:
    action = 'Inconnu';
}
```

nous pouvons écrire :

```dart
String action = switch (choix) {
  1 => 'Jouer',
  2 => 'Options',
  _ => 'Inconnu',
};
```

---

## 04.57 — Exemple complet : état du joueur

```dart
void main() {
  int vies = 2;
  double energie = 45;
  int score = 1250;
  bool pause = false;

  if (vies <= 0) {
    print('Game Over');
  } else {
    print('Joueur vivant');

    if (energie <= 0) {
      print('Énergie épuisée');
    } else if (energie < 25) {
      print('Énergie critique');
    } else if (energie < 60) {
      print('Énergie moyenne');
    } else {
      print('Bonne énergie');
    }

    if (score >= 1000 && !pause) {
      print('Le niveau suivant est disponible.');
    }
  }
}
```

---

## 04.58 — Exemple complet : niveau de difficulté

```dart
void main() {
  String difficulte = 'difficile';

  switch (difficulte) {
    case 'facile':
      print('Vies : 5');
      print('Ennemis : faibles');
      break;

    case 'normal':
      print('Vies : 3');
      print('Ennemis : normaux');
      break;

    case 'difficile':
      print('Vies : 2');
      print('Ennemis : puissants');
      break;

    default:
      print('Difficulté invalide');
  }
}
```

---

## 04.59 — Exemple complet : récompense selon le score

```dart
void main() {
  int score = 4200;

  if (score >= 5000) {
    print('Coffre légendaire');
  } else if (score >= 3000) {
    print('Coffre épique');
  } else if (score >= 1500) {
    print('Coffre rare');
  } else if (score >= 500) {
    print('Coffre commun');
  } else {
    print('Aucune récompense');
  }
}
```

---

## 04.60 — Fiche mémo et points à retenir

### FICHE MÉMO — `if`

```dart
if (condition) {
  // code exécuté si la condition est vraie
}
```

Exemple :

```dart
if (score >= 1000) {
  print('Niveau débloqué');
}
```

### FICHE MÉMO — `if / else`

```dart
if (condition) {
  // vrai
} else {
  // faux
}
```

Exemple :

```dart
if (vies > 0) {
  print('Continue');
} else {
  print('Game Over');
}
```

### FICHE MÉMO — `else if`

```dart
if (condition1) {
  // ...
} else if (condition2) {
  // ...
} else {
  // ...
}
```

Exemple :

```dart
if (score >= 1000) {
  print('Excellent');
} else if (score >= 500) {
  print('Bon');
} else {
  print('Continuez');
}
```

### FICHE MÉMO — CONDITIONS LOGIQUES

ET :

```dart
if (age >= 18 && compteActif) {
}
```

OU :

```dart
if (admin || moderateur) {
}
```

NON :

```dart
if (!jeuEnPause) {
}
```

### FICHE MÉMO — `switch`

```dart
switch (valeur) {
  case 1:
    print('Un');
    break;

  case 2:
    print('Deux');
    break;

  default:
    print('Autre');
}
```

### FICHE MÉMO — SWITCH EXPRESSION

```dart
String message = switch (valeur) {
  1 => 'Un',
  2 => 'Deux',
  _ => 'Autre',
};
```

### À RETENIR ABSOLUMENT

Une condition doit produire :

```text
true
```

ou :

```text
false
```

Exemple :

```dart
score >= 1000
```

---

`if` permet de dire :

```text
SI une condition est vraie
ALORS exécuter quelque chose
```

---

`else` signifie :

```text
SINON
```

---

`else if` permet d'ajouter d'autres possibilités.

---

Utilisez :

```text
&& → ET
|| → OU
!  → NON
```

---

Utilisez généralement `if` pour :

```text
score >= 1000
energie > 0
age >= 18
vies <= 0
condition1 && condition2
```

Utilisez volontiers `switch` pour :

```text
menu = 1, 2, 3, 4

difficulté = facile, normal, difficile

rôle = admin, moderateur, user
```

### Lien avec notre futur jeu Flutter

Nous avons déjà suffisamment de connaissances pour écrire une partie importante de la logique d'un jeu.

Exemple :

```dart
if (vies <= 0) {
  print('Game Over');
}
```

Ou :

```dart
if (score >= scoreRequis) {
  print('Niveau suivant');
}
```

Ou :

```dart
if (joueurX >= tresorX && joueurX <= tresorX + 20) {
  print('Trésor récupéré');
}
```

Plus tard, avec Flame, les mêmes principes permettront de gérer :

```text
collisions
vies
score
bonus
ennemis
niveaux
boss
victoire
défaite
menus
états du jeu
```

Nous construisons donc déjà les fondations de la Partie 2.

---

## 04.61 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| Confondre `=` et `==` dans une condition | `=` affecte une valeur, `==` compare deux valeurs | Utiliser `==` dans un `if` ou un `switch`, jamais `=` |
| Oublier les accolades `{ }` d'un bloc à une seule instruction | Dart accepte `if (condition) print('...')` sans accolades, ce qui piège en cas d'ajout d'une ligne | Toujours entourer le bloc de `{ }`, même pour une seule instruction |
| Tester la condition la plus large avant la plus restrictive | Dans une chaîne `if / else if`, Dart exécute le premier bloc vrai puis ignore les suivants | Classer les conditions de la plus restrictive à la plus large (`score >= 1000` avant `score >= 500`) |
| Utiliser `&&` alors qu'une seule condition suffirait, ou l'inverse avec `||` | `&&` exige que toutes les conditions soient vraies, `||` qu'une seule le soit | Relire la règle métier avant de choisir l'opérateur logique |
| Comparer deux `String` sans tenir compte de la casse | `'admin' == 'Admin'` renvoie `false`, Dart est sensible à la casse | Normaliser avec `.toLowerCase()` avant de comparer si la casse ne doit pas compter |
| Empiler des conditions imbriquées inutilement | Un enchaînement de `if` imbriqués devient vite illisible alors qu'un seul `&&` suffit souvent | Remplacer une imbrication par une condition composée avec `&&` quand c'est possible |
| Oublier `default` dans un `switch` classique | Sans `default`, une valeur imprévue ne produit aucun affichage et l'erreur passe inaperçue | Toujours prévoir un `default`, même pour signaler un cas invalide |
| Oublier `break;` à la fin de chaque `case` | Contrairement à d'autres langages, Dart interdit le fall-through implicite et refuse de compiler un `case` sans `break`, `return` ou instruction terminale | Terminer chaque `case` par `break;` (ou `return`, ou regrouper plusieurs valeurs sur le même `case`) |

---

## 04.62 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| `if` | Exécute un bloc uniquement si la condition vaut `true` |
| `else` | Exécute un bloc de repli lorsque la condition du `if` est fausse |
| `else if` | Ajoute d'autres possibilités entre le `if` et le `else` final |
| Ordre des conditions | Dans une chaîne `if / else if`, il faut tester la condition la plus restrictive en premier |
| `&&`, `\|\|`, `!` | Combinent (ET, OU) ou inversent (NON) des conditions, comme vu au chapitre 03 |
| Conditions imbriquées | Un `if` peut contenir un autre `if` ; à utiliser avec modération pour rester lisible |
| Validation de plage | `note >= 0 && note <= 100` vérifie qu'une valeur reste dans un intervalle |
| Comparaison de chaînes | `==` compare deux `String` en tenant compte de la casse ; `.toLowerCase()` permet d'ignorer la casse |
| `switch` classique | Compare une variable à plusieurs valeurs précises grâce à des `case`, chacun terminé par `break` |
| `case` / `default` | `case` teste une valeur possible, `default` couvre tous les cas restants |
| `switch` avec `String` | Fonctionne comme un `switch` sur `int`, en comparant des chaînes de caractères |
| `if` vs `switch` | `if` convient aux comparaisons (`>`, `<`, `&&`...), `switch` convient à une variable qui prend des valeurs précises |
| `switch` expression | Forme moderne de `switch` qui produit directement une valeur avec `=>`, sans `break` |
| Le motif `_` | Dans un `switch` expression, `_` capture toutes les valeurs non listées, comme `default` |

---

## 04.63 — Exercices

### Exercice 1 — Majeur ou mineur (facile)

Créez :

```dart
int age = 17;
```

Affichez :

```text
Majeur
```

si `age >= 18`.

Sinon :

```text
Mineur
```

---

### Exercice 2 — Nombre positif (facile)

Créez :

```dart
int nombre = 10;
```

Affichez :

```text
Nombre positif
```

si le nombre est supérieur à zéro.

---

### Exercice 3 — Positif, négatif ou zéro (facile)

Créez un programme capable d'afficher :

```text
Positif
```

ou :

```text
Négatif
```

ou :

```text
Zéro
```

selon la valeur d'un entier.

---

### Exercice 4 — Pair ou impair (facile)

Avec :

```dart
int nombre = 17;
```

utilisez :

```dart
%
```

et une condition pour afficher :

```text
Pair
```

ou :

```text
Impair
```

---

### Exercice 5 — Game Over (facile)

Avec :

```dart
int vies = 0;
```

affichez :

```text
Game Over
```

si le joueur n'a plus de vies.

Sinon :

```text
Continuez à jouer
```

---

### Exercice 6 — Niveau suivant (facile)

Données :

```dart
int score = 1250;
```

Le niveau suivant est débloqué si :

```text
score >= 1000
```

Affichez le résultat approprié.

---

### Exercice 7 — Contrôle d'accès (moyen)

Données :

```dart
int age = 25;
bool compteActif = true;
```

L'accès est autorisé seulement si :

```text
âge >= 18
ET
compte actif
```

---

### Exercice 8 — Porte secrète (moyen)

Données :

```dart
bool possedeCle = true;
int niveau = 4;
```

La porte s'ouvre uniquement si :

```text
possède la clé
ET
niveau >= 5
```

Affichez :

```text
Porte ouverte
```

ou :

```text
Porte verrouillée
```

---

### Exercice 9 — Victoire (moyen)

Données :

```dart
int score = 9000;
bool bossBattu = false;
```

Le joueur gagne si :

```text
score >= 10000
OU
boss battu
```

Affichez le résultat.

---

### Exercice 10 — État de l'énergie (moyen)

Données :

```dart
double energie = 42;
```

Affichez :

```text
Énergie élevée
```

si énergie >= 75.

```text
Énergie moyenne
```

si énergie >= 40.

```text
Énergie faible
```

si énergie > 0.

Sinon :

```text
Énergie épuisée
```

---

### Exercice 11 — Système de note (moyen)

Avec :

```dart
double note = 76;
```

Affichez :

```text
A → 90 et plus
B → 80 à 89
C → 70 à 79
D → 60 à 69
F → moins de 60
```

---

### Exercice 12 — Validation de note (moyen)

Avec :

```dart
double note = 110;
```

Vérifiez d'abord que la note se trouve entre :

```text
0 et 100
```

Sinon affichez :

```text
Note invalide
```

---

### Exercice 13 — Niveau du joueur (moyen)

Avec :

```dart
int niveau = 14;
```

Affichez :

```text
Maître
```

si niveau >= 20.

```text
Avancé
```

si niveau >= 10.

```text
Intermédiaire
```

si niveau >= 5.

Sinon :

```text
Débutant
```

---

### Exercice 14 — Boss (moyen)

Un boss apparaît à tous les niveaux multiples de 5.

Avec :

```dart
int niveau = 20;
```

affichez :

```text
Boss
```

ou :

```text
Niveau normal
```

---

### Exercice 15 — Switch menu (moyen)

Créez :

```dart
int choix = 3;
```

Utilisez `switch` pour afficher :

```text
1 → Jouer
2 → Continuer
3 → Options
4 → Quitter
autre → Choix invalide
```

---

### Exercice 16 — Difficulté (moyen)

Créez :

```dart
String difficulte = 'normal';
```

Utilisez `switch` pour afficher :

```text
facile → 5 vies
normal → 3 vies
difficile → 1 vie
autre → difficulté inconnue
```

---

### Exercice 17 — Rôle utilisateur (moyen)

Créez :

```dart
String role = 'moderateur';
```

Utilisez `switch` pour afficher :

```text
admin → Accès complet
moderateur → Accès modération
user → Accès standard
autre → Rôle inconnu
```

---

### Exercice 18 — Switch expression (moyen)

Transformez :

```dart
int choix = 2;
```

en une variable :

```dart
String action;
```

avec un `switch` moderne :

```text
1 → Jouer
2 → Paramètres
3 → Quitter
autre → Inconnu
```

---

### Exercice 19 — Conditions imbriquées (difficile)

Données :

```dart
bool connecte = true;
bool abonnementActif = true;
```

Si l'utilisateur est connecté :

```text
Utilisateur connecté
```

Puis vérifiez son abonnement.

Si actif :

```text
Contenu premium disponible
```

Sinon :

```text
Abonnement requis
```

Si l'utilisateur n'est pas connecté :

```text
Connexion requise
```

---

### Exercice 20 — Challenge jeu complet (difficile)

Données :

```dart
String joueur = 'Alex';
int niveau = 10;
int score = 3200;
int vies = 2;
double energie = 35;
bool possedeCle = true;
bool jeuEnPause = false;
```

Le programme doit déterminer :

```text
si le joueur est vivant
si le joueur peut continuer
si le niveau est un niveau boss
si une porte peut être ouverte
le niveau de récompense
l'état de l'énergie
```

Règles :

```text
vivant
→ vies > 0

peut continuer
→ vivant ET énergie > 0 ET jeu non en pause

boss
→ niveau multiple de 5

porte
→ possède clé ET niveau >= 5

récompense légendaire
→ score >= 5000

récompense épique
→ score >= 3000

récompense rare
→ score >= 1000

sinon
→ récompense normale
```

---

## 04.64 — Corrections des exercices

### Correction 1

```dart
void main() {
  int age = 17;

  if (age >= 18) {
    print('Majeur');
  } else {
    print('Mineur');
  }
}
```

**Explication :** Le premier `if / else` du chapitre : une seule condition suffit à choisir entre deux issues.

---

### Correction 2

```dart
void main() {
  int nombre = 10;

  if (nombre > 0) {
    print('Nombre positif');
  }
}
```

**Explication :** Un `if` seul, sans `else`, laisse simplement passer le programme si la condition est fausse.

---

### Correction 3

```dart
void main() {
  int nombre = -5;

  if (nombre > 0) {
    print('Positif');
  } else if (nombre < 0) {
    print('Négatif');
  } else {
    print('Zéro');
  }
}
```

**Explication :** Trois issues possibles imposent une chaîne `if / else if / else`, testée dans l'ordre.

---

### Correction 4

```dart
void main() {
  int nombre = 17;

  if (nombre % 2 == 0) {
    print('Pair');
  } else {
    print('Impair');
  }
}
```

**Explication :** Le modulo `%` (vu au chapitre 03) donne le reste de la division par 2, utilisé ici dans la condition.

---

### Correction 5

```dart
void main() {
  int vies = 0;

  if (vies <= 0) {
    print('Game Over');
  } else {
    print('Continuez à jouer');
  }
}
```

**Explication :** `vies <= 0` couvre aussi bien `0` que d'éventuelles valeurs négatives, plus prudent qu'un simple `== 0`.

---

### Correction 6

```dart
void main() {
  int score = 1250;

  if (score >= 1000) {
    print('Niveau suivant débloqué');
  } else {
    print('Score insuffisant');
  }
}
```

**Explication :** Une seule condition (`score >= 1000`) suffit à décider si le niveau suivant est débloqué.

---

### Correction 7

```dart
void main() {
  int age = 25;
  bool compteActif = true;

  if (age >= 18 && compteActif) {
    print('Accès autorisé');
  } else {
    print('Accès refusé');
  }
}
```

**Explication :** `&&` exige que les deux conditions (âge suffisant ET compte actif) soient vraies en même temps.

---

### Correction 8

```dart
void main() {
  bool possedeCle = true;
  int niveau = 4;

  if (possedeCle && niveau >= 5) {
    print('Porte ouverte');
  } else {
    print('Porte verrouillée');
  }
}
```

**Explication :** Même logique que l'exercice précédent : `&&` combine la possession de la clé et le niveau minimum.

---

### Correction 9

```dart
void main() {
  int score = 9000;
  bool bossBattu = false;

  if (score >= 10000 || bossBattu) {
    print('Victoire');
  } else {
    print('Continuez à jouer');
  }
}
```

**Explication :** `||` ne demande qu'une seule des deux conditions (score élevé OU boss battu) pour valider la victoire.

---

### Correction 10

```dart
void main() {
  double energie = 42;

  if (energie >= 75) {
    print('Énergie élevée');
  } else if (energie >= 40) {
    print('Énergie moyenne');
  } else if (energie > 0) {
    print('Énergie faible');
  } else {
    print('Énergie épuisée');
  }
}
```

**Explication :** Une chaîne `if / else if / else` classe les seuils d'énergie du plus élevé au plus bas.

---

### Correction 11

```dart
void main() {
  double note = 76;

  if (note >= 90) {
    print('A');
  } else if (note >= 80) {
    print('B');
  } else if (note >= 70) {
    print('C');
  } else if (note >= 60) {
    print('D');
  } else {
    print('F');
  }
}
```

**Explication :** Même structure que l'exercice précédent, appliquée à des tranches de notes plutôt qu'à l'énergie.

---

### Correction 12

```dart
void main() {
  double note = 110;

  if (note < 0 || note > 100) {
    print('Note invalide');
  } else if (note >= 90) {
    print('A');
  } else if (note >= 80) {
    print('B');
  } else if (note >= 70) {
    print('C');
  } else if (note >= 60) {
    print('D');
  } else {
    print('F');
  }
}
```

**Explication :** La validation de plage (`note < 0 || note > 100`) est testée en premier, avant même de chercher la lettre correspondante.

---

### Correction 13

```dart
void main() {
  int niveau = 14;

  if (niveau >= 20) {
    print('Maître');
  } else if (niveau >= 10) {
    print('Avancé');
  } else if (niveau >= 5) {
    print('Intermédiaire');
  } else {
    print('Débutant');
  }
}
```

**Explication :** Les seuils de niveau sont à nouveau testés du plus élevé au plus bas, comme pour l'énergie ou les notes.

---

### Correction 14

```dart
void main() {
  int niveau = 20;

  if (niveau % 5 == 0) {
    print('Boss');
  } else {
    print('Niveau normal');
  }
}
```

**Explication :** Le modulo `%` détecte les niveaux multiples de 5, réutilisé du chapitre 03.

---

### Correction 15

```dart
void main() {
  int choix = 3;

  switch (choix) {
    case 1:
      print('Jouer');
      break;

    case 2:
      print('Continuer');
      break;

    case 3:
      print('Options');
      break;

    case 4:
      print('Quitter');
      break;

    default:
      print('Choix invalide');
  }
}
```

**Explication :** Un `switch` classique convient bien ici : la variable `choix` ne prend que quelques valeurs précises.

---

### Correction 16

```dart
void main() {
  String difficulte = 'normal';

  switch (difficulte) {
    case 'facile':
      print('5 vies');
      break;

    case 'normal':
      print('3 vies');
      break;

    case 'difficile':
      print('1 vie');
      break;

    default:
      print('Difficulté inconnue');
  }
}
```

**Explication :** Le `switch` compare directement des `String`, chaque `case` correspondant à une difficulté.

---

### Correction 17

```dart
void main() {
  String role = 'moderateur';

  switch (role) {
    case 'admin':
      print('Accès complet');
      break;

    case 'moderateur':
      print('Accès modération');
      break;

    case 'user':
      print('Accès standard');
      break;

    default:
      print('Rôle inconnu');
  }
}
```

**Explication :** Même principe que l'exercice précédent, appliqué à un rôle utilisateur plutôt qu'à une difficulté.

---

### Correction 18

```dart
void main() {
  int choix = 2;

  String action = switch (choix) {
    1 => 'Jouer',
    2 => 'Paramètres',
    3 => 'Quitter',
    _ => 'Inconnu',
  };

  print(action);
}
```

**Explication :** La forme `switch` expression remplace les `case` classiques par des `=>`, sans `break`, et produit directement une valeur.

---

### Correction 19

```dart
void main() {
  bool connecte = true;
  bool abonnementActif = true;

  if (connecte) {
    print('Utilisateur connecté');

    if (abonnementActif) {
      print('Contenu premium disponible');
    } else {
      print('Abonnement requis');
    }
  } else {
    print('Connexion requise');
  }
}
```

**Explication :** Deux niveaux de décision s'imbriquent : la connexion d'abord, puis l'abonnement seulement si l'utilisateur est connecté.

---

### Correction 20

```dart
void main() {
  String joueur = 'Alex';
  int niveau = 10;
  int score = 3200;
  int vies = 2;
  double energie = 35;
  bool possedeCle = true;
  bool jeuEnPause = false;

  bool vivant = vies > 0;

  print('============================');
  print('ÉTAT DU JOUEUR');
  print('============================');

  print('Joueur : $joueur');
  print('Niveau : $niveau');
  print('Score : $score');
  print('Vies : $vies');
  print('Énergie : $energie');

  if (vivant) {
    print('Statut : vivant');
  } else {
    print('Statut : Game Over');
  }

  if (vivant && energie > 0 && !jeuEnPause) {
    print('Le joueur peut continuer.');
  } else {
    print('Le joueur ne peut pas continuer.');
  }

  if (niveau % 5 == 0) {
    print('Type de niveau : BOSS');
  } else {
    print('Type de niveau : normal');
  }

  if (possedeCle && niveau >= 5) {
    print('Porte : ouverte');
  } else {
    print('Porte : verrouillée');
  }

  if (score >= 5000) {
    print('Récompense : légendaire');
  } else if (score >= 3000) {
    print('Récompense : épique');
  } else if (score >= 1000) {
    print('Récompense : rare');
  } else {
    print('Récompense : normale');
  }

  if (energie >= 75) {
    print('Énergie : élevée');
  } else if (energie >= 40) {
    print('Énergie : moyenne');
  } else if (energie > 0) {
    print('Énergie : faible');
  } else {
    print('Énergie : épuisée');
  }
}
```

**Explication :** Cette correction combine conditions imbriquées, `&&`, `||`, `%` et un `switch` implicite via plusieurs `if` : elle résume tout le chapitre.

---

## Et maintenant ?

Vous savez désormais faire réagir votre programme grâce à `if`, `else if`, `else` et `switch`, y compris la forme moderne du `switch` expression. Le chapitre suivant vous apprend à répéter automatiquement des instructions grâce aux boucles `for`, `while` et `do while`, que vous combinerez très vite avec les conditions vues ici.

Direction le chapitre suivant : [05-PARTIE-1A—BOUCLES-FOR-WHILE-DO-WHILE.md](./05-PARTIE-1A—BOUCLES-FOR-WHILE-DO-WHILE.md)