# PARTIE 1A — DART
# CHAPITRE 15 — LA PROGRAMMATION ASYNCHRONE : FUTURE, ASYNC, AWAIT ET STREAM

> **Niveau :** intermédiaire
> **Durée estimée :** 6 h
> **Pré-requis :** chapitre 13 — Exceptions et gestion des erreurs, chapitre 14 — Programmation fonctionnelle sur les collections
> **Ce que vous saurez faire à la fin :** écrire un programme qui attend un résultat long (chargement, réseau, fichier) sans jamais bloquer le reste de l'application, en maîtrisant `Future`, `async`, `await`, `Future.wait()` et les `Stream`.

---

## 15.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer pourquoi une opération longue ne doit jamais bloquer le programme ;
- distinguer un traitement synchrone d'un traitement asynchrone ;
- expliquer ce qu'est l'event loop de Dart et pourquoi Dart est mono-thread ;
- prédire **l'ordre réel d'affichage** d'un programme asynchrone ;
- définir un `Future` et nommer ses trois états ;
- créer un `Future` avec `Future.delayed()`, `Future.value()` et `Future.error()` ;
- utiliser `.then()`, `.catchError()` et `.whenComplete()` ;
- écrire une fonction `async` et utiliser `await` ;
- comprendre pourquoi une fonction `async` retourne toujours un `Future` ;
- attraper une erreur asynchrone avec `try` / `catch` / `finally` ;
- reconnaître des `await` inutilement séquentiels et les paralléliser avec `Future.wait()` ;
- utiliser `unawaited` pour un future volontairement non attendu ;
- écrire un `main()` asynchrone ;
- expliquer la différence entre un `Future` et un `Stream` ;
- produire un `Stream` avec `async*` et `yield` ;
- consommer un `Stream` avec `await for` et avec `listen()` ;
- transformer un `Stream` avec `map` et `where` ;
- expliquer pourquoi Flutter repose entièrement sur l'asynchrone.

---

## 15.1 — Le problème : une opération qui prend du temps

Jusqu'ici, tout votre code allait **vite**. Additionner deux nombres, parcourir une liste de dix éléments, afficher un texte : quelques microsecondes. Le programme exécutait une ligne, puis la suivante, et personne ne remarquait la différence.

Certaines opérations, elles, sont **lentes** :

| Opération | Durée typique |
| --- | --- |
| Une addition | 0,000001 s |
| Lire un fichier de sauvegarde sur le disque | 0,01 à 0,5 s |
| Demander le profil du joueur à un serveur | 0,2 à 3 s |
| Télécharger la texture d'un niveau | 1 à 10 s |
| Attendre que le joueur appuie sur un bouton | indéfini |

Ces durées ont un point commun : pendant tout ce temps, **votre programme n'a rien à calculer**. Il attend. Il attend le disque, il attend le réseau, il attend l'utilisateur.

La question du chapitre est donc :

> Que fait le reste du programme pendant cette attente ?

Regardons ce qui se passe quand on attend « bêtement ». La fonction suivante occupe le processeur pendant un nombre de secondes donné, sans rien faire d'utile. C'est exactement ce qu'il **ne faut pas** faire, mais c'est très pédagogique.

```dart
void attendreEnBloquant(int secondes) {
  final DateTime fin = DateTime.now().add(Duration(seconds: secondes));
  while (DateTime.now().isBefore(fin)) {
    // On tourne en rond. Le processeur est occupé, mais il ne produit rien.
  }
}

void main() {
  print('1. Le joueur clique sur "Charger la partie"');
  print('2. Chargement du profil...');
  attendreEnBloquant(3);
  print('3. Profil chargé');
  print('4. Affichage du menu');
}
```

**Résultat :**

```text
1. Le joueur clique sur "Charger la partie"
2. Chargement du profil...
(rien pendant 3 secondes)
3. Profil chargé
4. Affichage du menu
```

Dans une console, ce n'est pas dramatique. Dans un jeu, c'est une catastrophe :

```text
  ┌────────────────────────────────────────────────────────────┐
  │  Pendant les 3 secondes de "attendreEnBloquant"            │
  ├────────────────────────────────────────────────────────────┤
  │  L'image ne se redessine plus ......... écran figé         │
  │  L'animation du logo s'arrête ......... saccade            │
  │  Le bouton "Annuler" ne répond plus ... clic ignoré        │
  │  Le système peut afficher .............. "ne répond pas"   │
  └────────────────────────────────────────────────────────────┘
```

C'est ce qu'on appelle **bloquer le thread principal**. L'utilisateur, lui, appelle ça « le jeu a planté ».

La programmation asynchrone est la réponse à ce problème. Elle permet d'écrire :

> « Lance le chargement, préviens-moi quand c'est fini, et en attendant continue à faire tourner le jeu. »

---

## 15.2 — Synchrone contre asynchrone : l'analogie du restaurant

Le vocabulaire fait peur, l'idée est très simple. Imaginez un restaurant.

**Version synchrone (bloquante) :**

```text
  Le serveur prend la commande de la table 1.
  Il va en cuisine.
  Il ATTEND devant le four que le plat cuise (20 minutes).
  Il apporte le plat à la table 1.
  Il prend enfin la commande de la table 2.
```

Pendant vingt minutes, le serveur est immobilisé. Les tables 2, 3 et 4 ne sont pas servies. Le restaurant tourne au ralenti alors que le serveur ne fait rien d'utile.

**Version asynchrone (non bloquante) :**

```text
  Le serveur prend la commande de la table 1.
  Il la transmet à la cuisine.
  Il prend la commande de la table 2.
  Il apporte les boissons de la table 3.
  La cuisine sonne : "plat de la table 1 prêt".
  Le serveur apporte le plat à la table 1.
```

Le serveur n'a jamais attendu. Il a **délégué** l'opération longue (la cuisson) et il est resté disponible.

Traduisons le vocabulaire :

| Restaurant | Programmation |
| --- | --- |
| Le serveur | Le thread principal de votre programme |
| Une commande | Un appel de fonction |
| La cuisson du plat | Une opération longue (réseau, disque) |
| Le ticket de commande | Un `Future` (la promesse d'un résultat) |
| La sonnette de la cuisine | L'event loop qui signale « c'est prêt » |
| Servir le plat quand il est prêt | Le code qui suit un `await` ou un `.then()` |

Retenez surtout ceci, car c'est le contresens numéro un des débutants :

> **Asynchrone ne veut pas dire « en même temps ».** Il n'y a toujours qu'un seul serveur. Asynchrone veut dire « sans rester bloqué à attendre ».

---

## 15.3 — Dart est mono-thread : l'event loop

Un **thread** est un fil d'exécution : une suite d'instructions exécutées l'une après l'autre. Certains langages permettent de lancer plusieurs threads qui travaillent réellement en parallèle, sur plusieurs cœurs du processeur. Cela apporte de la puissance, mais aussi des bugs redoutables (deux threads qui modifient la même variable en même temps).

Dart a fait un autre choix :

> **Votre code Dart s'exécute sur un seul thread.** Il n'y a jamais deux lignes de votre programme exécutées au même instant.

C'est une excellente nouvelle pour vous : vous n'aurez jamais à protéger une variable contre un accès concurrent. Le score du joueur ne peut pas être modifié « pendant » que vous le lisez.

Mais alors, comment fait Dart pour ne pas bloquer pendant une attente ? Grâce à l'**event loop** (la « boucle d'événements »).

L'event loop est une boucle infinie, gérée par Dart, qui fonctionne ainsi :

```text
  tant que le programme vit :
      s'il y a une microtâche en attente  -> l'exécuter
      sinon s'il y a un événement en attente -> l'exécuter
      sinon -> ne rien faire (le processeur se repose)
```

Deux files d'attente existent donc :

| File | Contenu | Priorité |
| --- | --- | --- |
| File des microtâches | Petits travaux internes créés par Dart (suites de `Future` déjà terminés) | Haute |
| File des événements | Timers, entrées/sorties, clics, résultats réseau | Normale |

Le point essentiel, celui qui explique tous les ordres d'affichage surprenants de ce chapitre :

> Une tâche prise dans une file est exécutée **jusqu'au bout**, sans interruption. L'event loop ne repasse à la tâche suivante que lorsque la précédente est terminée.

C'est pour cette raison que `attendreEnBloquant()` de la section 15.1 fige tout : tant que cette fonction tourne, l'event loop ne peut rien faire d'autre, ni redessiner l'écran, ni traiter un clic.

---

## 15.4 — Schéma de l'event loop

Voici la vue complète. Gardez ce schéma en tête pendant tout le chapitre.

```text
                    ┌──────────────────────────────┐
                    │        VOTRE PROGRAMME       │
                    │   (un seul fil d'exécution)  │
                    └──────────────┬───────────────┘
                                   │
                                   v
   ┌───────────────────────────────────────────────────────────┐
   │                        EVENT LOOP                         │
   │                                                           │
   │   1) File des MICROTÂCHES        (priorité haute)         │
   │      ┌──────┬──────┬──────┐                               │
   │      │ m1   │ m2   │ m3   │  <- suites de Future          │
   │      └──────┴──────┴──────┘                               │
   │              vidée ENTIÈREMENT avant de passer en 2       │
   │                                                           │
   │   2) File des ÉVÉNEMENTS         (priorité normale)       │
   │      ┌──────┬──────┬──────┐                               │
   │      │ e1   │ e2   │ e3   │  <- timers, réseau, clics     │
   │      └──────┴──────┴──────┘                               │
   │              on en prend UN, puis on retourne en 1        │
   └───────────────────────────────────────────────────────────┘
                                   ^
                                   │  "le plat est prêt"
                    ┌──────────────┴───────────────┐
                    │   MONDE EXTÉRIEUR / SYSTÈME  │
                    │  disque, réseau, horloge,    │
                    │  clavier, souris             │
                    └──────────────────────────────┘
```

Déroulons un exemple concret, celui que vous écrirez à la section 15.8 :

```text
  main() démarre
    |
    |-- print('A')                       -> affiché tout de suite
    |
    |-- Future.delayed(2s, () => print('B'))
    |      -> Dart demande au système : "réveille-moi dans 2 s"
    |      -> la fonction ne bloque PAS, elle rend la main
    |
    |-- print('C')                       -> affiché tout de suite
    |
  main() est terminé, mais le programme ne s'arrête pas :
  l'event loop attend qu'il ne reste plus rien à faire.

    ... 2 secondes plus tard ...

  Le système pousse l'événement dans la file
    -> l'event loop le prend
    -> print('B')                        -> affiché maintenant
```

Ordre réel d'affichage : **A, C, B**.

Cet ordre n'est pas un hasard ni un bug. C'est la conséquence directe du schéma ci-dessus, et c'est le cœur pédagogique de ce chapitre : dans un programme asynchrone, **l'ordre du code n'est pas l'ordre d'exécution**.

---

## 15.5 — Qu'est-ce qu'un `Future` ?

Un `Future` est un objet Dart qui représente :

> **une valeur qui n'existe pas encore, mais qui existera plus tard** (ou une erreur).

C'est exactement le ticket de commande du restaurant. Le ticket n'est pas le plat. C'est la promesse qu'un plat arrivera, et le moyen de savoir quand.

```text
  ┌──────────────────────────────────────────────────────────┐
  │  String nom = 'Alex';                                    │
  │      -> la valeur est là, MAINTENANT                     │
  ├──────────────────────────────────────────────────────────┤
  │  Future<String> nom = chargerProfil();                   │
  │      -> la valeur n'est pas là                           │
  │      -> vous tenez un TICKET qui la livrera plus tard    │
  └──────────────────────────────────────────────────────────┘
```

Regardons ce que contient réellement un `Future` avant qu'il ne soit terminé.

```dart
Future<String> chargerProfil() {
  return Future.delayed(Duration(seconds: 1), () => 'Alex');
}

void main() {
  Future<String> ticket = chargerProfil();
  print('Ce que je tiens en main : $ticket');
  print('Son type : ${ticket.runtimeType}');
}
```

**Résultat :**

```text
Ce que je tiens en main : Instance of 'Future<String>'
Son type : Future<String>
```

Notez bien cette sortie : `Instance of 'Future<String>'`. Vous la reverrez souvent, et elle signifie toujours la même chose :

> Vous avez affiché **le ticket** au lieu d'attendre **le plat**.

C'est l'erreur la plus fréquente du chapitre. Nous verrons à la section 15.14 comment obtenir la valeur.

---

## 15.6 — Les trois états d'un `Future`

Un `Future` passe par exactement trois états, et il ne revient jamais en arrière.

```text
                    ┌────────────────────┐
                    │    UNCOMPLETED     │
                    │   (non terminé)    │
                    │  le plat est en    │
                    │     cuisine        │
                    └─────────┬──────────┘
                              │
                 ┌────────────┴────────────┐
                 │                         │
                 v                         v
      ┌────────────────────┐   ┌────────────────────────┐
      │  COMPLETED WITH    │   │   COMPLETED WITH       │
      │      VALUE         │   │       ERROR            │
      │ terminé avec une   │   │  terminé avec une      │
      │      valeur        │   │      erreur            │
      │  "voici le plat"   │   │ "le four est en panne" │
      └────────────────────┘   └────────────────────────┘
```

| État | Nom anglais | Ce que cela signifie |
| --- | --- | --- |
| Non terminé | *uncompleted* | L'opération est en cours. Aucune valeur, aucune erreur. |
| Terminé avec une valeur | *completed with a value* | L'opération a réussi. La valeur est disponible. |
| Terminé avec une erreur | *completed with an error* | L'opération a échoué. Une exception est disponible. |

Deux règles à retenir :

1. Un `Future` **ne se termine qu'une seule fois**. Une fois terminé, son résultat ne change plus.
2. Un `Future` terminé avec une erreur **doit** être traité, sinon l'erreur remonte comme une exception non attrapée (voir la section 15.10).

Vérifions le premier état avec un compte à rebours :

```dart
void main() {
  print('t=0 s : je commande');

  final Future<String> plat = Future.delayed(
    Duration(seconds: 2),
    () => 'Steak-frites',
  );

  print('t=0 s : le ticket existe, le plat non');

  plat.then((String valeur) {
    print('t=2 s : le plat est arrivé -> $valeur');
  });

  print('t=0 s : je continue à travailler');
}
```

**Résultat (dans cet ordre exact) :**

```text
t=0 s : je commande
t=0 s : le ticket existe, le plat non
t=0 s : je continue à travailler
t=2 s : le plat est arrivé -> Steak-frites
```

Les trois premières lignes sortent immédiatement. La quatrième sort deux secondes plus tard.

---

## 15.7 — `Future<T>` et `Future<void>`

`Future` est un type générique, comme `List`. Ce qui est entre chevrons est **le type de la valeur livrée plus tard**.

| Écriture | Signification |
| --- | --- |
| `Future<String>` | livrera un texte |
| `Future<int>` | livrera un entier |
| `Future<List<String>>` | livrera une liste de textes |
| `Future<Player>` | livrera un objet `Player` |
| `Future<void>` | ne livrera **aucune valeur**, seulement la fin de l'opération |

`Future<void>` est très courant : il correspond à une action dont on veut savoir qu'elle est **terminée**, sans qu'elle produise de résultat. Sauvegarder une partie, par exemple.

```dart
Future<String> chargerNomJoueur() {
  return Future.delayed(Duration(seconds: 1), () => 'Alex');
}

Future<int> chargerNiveau() {
  return Future.delayed(Duration(seconds: 1), () => 7);
}

Future<List<String>> chargerInventaire() {
  return Future.delayed(
    Duration(seconds: 1),
    () => ['Potion', 'Épée de feu', 'Clé rouillée'],
  );
}

Future<void> sauvegarderPartie() {
  return Future.delayed(Duration(seconds: 1), () {
    print('Partie sauvegardée sur le disque.');
  });
}

void main() {
  print(chargerNomJoueur().runtimeType);
  print(chargerNiveau().runtimeType);
  print(chargerInventaire().runtimeType);
  print(sauvegarderPartie().runtimeType);
}
```

**Résultat :**

```text
Future<String>
Future<int>
Future<List<String>>
Future<void>
Partie sauvegardée sur le disque.
```

Remarquez la dernière ligne : elle apparaît **après** les quatre types, une seconde plus tard, parce que la sauvegarde était différée. Le programme ne s'arrête pas tant qu'il reste un timer en attente.

> **Règle de nommage :** une fonction qui retourne un `Future` décrit une action qui prend du temps. Nommez-la avec un verbe : `chargerProfil`, `sauvegarderPartie`, `telechargerNiveau`.

---

## 15.8 — `Future.delayed()`

`Future.delayed()` est votre outil d'apprentissage numéro un. Il crée un `Future` qui se termine après une durée choisie. Il simule parfaitement une opération longue, sans avoir besoin d'un vrai serveur.

Sa forme générale :

```text
  Future.delayed(Duration, [fonction à exécuter à la fin])
```

Rappel sur `Duration` :

| Écriture | Durée |
| --- | --- |
| `Duration(seconds: 2)` | 2 secondes |
| `Duration(milliseconds: 500)` | une demi-seconde |
| `Duration(minutes: 1)` | 1 minute |
| `Duration.zero` | zéro seconde (mais toujours asynchrone) |

L'exemple fondateur du chapitre :

```dart
void main() {
  print('A — début de main');

  Future.delayed(Duration(seconds: 2), () {
    print('B — 2 secondes plus tard');
  });

  print('C — fin de main');
}
```

**Résultat (ordre réel) :**

```text
A — début de main
C — fin de main
B — 2 secondes plus tard
```

Lisez bien : `B` est écrit **au milieu** du code, mais s'affiche **en dernier**. C'est normal. `Future.delayed()` ne bloque pas ; elle enregistre un rendez-vous auprès de l'event loop et rend immédiatement la main.

Poussons la démonstration à l'extrême avec `Duration.zero` :

```dart
void main() {
  print('A');

  Future.delayed(Duration.zero, () {
    print('B — pourtant zéro seconde d\'attente');
  });

  print('C');
}
```

**Résultat :**

```text
A
C
B — pourtant zéro seconde d'attente
```

Même avec une durée nulle, `B` passe en dernier. La raison est dans le schéma de la section 15.4 : le travail est placé dans la **file des événements**, et cette file n'est consultée qu'une fois le code synchrone en cours entièrement terminé.

> **À retenir :** ce n'est pas la durée qui décale l'exécution, c'est le passage par l'event loop.

---

## 15.9 — `.then()` : réagir quand le `Future` se termine

`.then()` est la méthode qui dit : « quand ce `Future` sera terminé avec une valeur, exécute cette fonction en lui passant la valeur ».

```dart
Future<String> chargerProfil() {
  return Future.delayed(Duration(seconds: 2), () => 'Alex');
}

void main() {
  print('1. Début du chargement');

  chargerProfil().then((String nom) {
    print('3. Profil reçu : $nom');
  });

  print('2. Le jeu continue de tourner');
}
```

**Résultat (ordre réel) :**

```text
1. Début du chargement
2. Le jeu continue de tourner
3. Profil reçu : Alex
```

Les numéros sont volontairement dans le désordre dans le code source, et dans l'ordre dans la sortie. C'est le meilleur moyen de fixer l'idée.

`.then()` retourne lui-même un `Future`, ce qui permet de **chaîner** les opérations :

```dart
Future<String> chargerProfil() {
  return Future.delayed(Duration(seconds: 1), () => 'Alex');
}

Future<int> chargerNiveau(String nom) {
  return Future.delayed(Duration(seconds: 1), () => nom.length * 2);
}

void main() {
  print('Début');

  chargerProfil()
      .then((String nom) {
        print('Profil : $nom');
        return chargerNiveau(nom);
      })
      .then((int niveau) {
        print('Niveau : $niveau');
      });

  print('Fin de main');
}
```

**Résultat :**

```text
Début
Fin de main
Profil : Alex
Niveau : 8
```

Le second `.then()` attend automatiquement le `Future` retourné par le premier. C'est la règle d'aplatissement : si la fonction passée à `.then()` retourne un `Future`, la chaîne attend ce `Future` avant de continuer.

---

## 15.10 — `.catchError()` : attraper une erreur asynchrone

Une opération longue peut échouer : serveur injoignable, fichier corrompu, sauvegarde absente. Le `Future` se termine alors **avec une erreur**.

Un `try` / `catch` classique **ne suffit pas** ici. Regardons pourquoi :

```dart
Future<String> chargerProfil() {
  return Future.delayed(Duration(seconds: 1), () {
    throw Exception('Serveur injoignable');
  });
}

void main() {
  try {
    chargerProfil().then((String nom) => print('Profil : $nom'));
  } catch (e) {
    print('Attrapé ? $e');
  }
  print('Fin de main');
}
```

**Résultat :**

```text
Fin de main
Unhandled exception: Exception: Serveur injoignable
```

Le `catch` n'a rien attrapé. La raison est chronologique : au moment où le `try` se termine, l'erreur n'a pas encore eu lieu. Elle surviendra une seconde plus tard, alors que le bloc `try` est de l'histoire ancienne.

La solution, dans le style `.then()`, est `.catchError()` :

```dart
Future<String> chargerProfil() {
  return Future.delayed(Duration(seconds: 1), () {
    throw Exception('Serveur injoignable');
  });
}

void main() {
  print('Début');

  chargerProfil()
      .then((String nom) => print('Profil : $nom'))
      .catchError((Object erreur) => print('Erreur attrapée : $erreur'));

  print('Fin de main');
}
```

**Résultat :**

```text
Début
Fin de main
Erreur attrapée : Exception: Serveur injoignable
```

Le programme ne plante plus. Le `.then()` est simplement sauté : quand un `Future` se termine avec une erreur, les `.then()` de la chaîne sont ignorés jusqu'au premier `.catchError()`.

> **Piège de typage :** si vous placez `.catchError()` **directement** sur un `Future<String>`, le gestionnaire doit renvoyer une `String` de repli, car la chaîne doit toujours produire une valeur du bon type. Sinon Dart lève une erreur de type à l'exécution.

```dart
Future<String> chargerProfil() {
  return Future.delayed(Duration(seconds: 1), () {
    throw Exception('Serveur injoignable');
  });
}

void main() {
  chargerProfil()
      .catchError((Object erreur) {
        print('Erreur : $erreur');
        return 'Invité'; // valeur de repli, du même type que le Future
      })
      .then((String nom) => print('Joueur chargé : $nom'));
}
```

**Résultat :**

```text
Erreur : Exception: Serveur injoignable
Joueur chargé : Invité
```

---

## 15.11 — `.whenComplete()` : le « quoi qu'il arrive »

`.whenComplete()` exécute une fonction **dans tous les cas** : succès comme erreur. C'est l'équivalent asynchrone du `finally` du chapitre 13. On y met le nettoyage : masquer un écran de chargement, fermer un fichier, arrêter une animation.

```dart
Future<String> chargerNiveau(bool doitEchouer) {
  return Future.delayed(Duration(seconds: 1), () {
    if (doitEchouer) {
      throw Exception('Fichier de niveau corrompu');
    }
    return 'Caverne de glace';
  });
}

void main() {
  print('--- Cas 1 : succès ---');
  chargerNiveau(false)
      .then((String niveau) => print('Niveau chargé : $niveau'))
      .catchError((Object e) => print('Erreur : $e'))
      .whenComplete(() => print('Écran de chargement masqué.'));
}
```

**Résultat :**

```text
--- Cas 1 : succès ---
Niveau chargé : Caverne de glace
Écran de chargement masqué.
```

Le même code avec `chargerNiveau(true)` :

```dart
Future<String> chargerNiveau(bool doitEchouer) {
  return Future.delayed(Duration(seconds: 1), () {
    if (doitEchouer) {
      throw Exception('Fichier de niveau corrompu');
    }
    return 'Caverne de glace';
  });
}

void main() {
  print('--- Cas 2 : échec ---');
  chargerNiveau(true)
      .then((String niveau) => print('Niveau chargé : $niveau'))
      .catchError((Object e) => print('Erreur : $e'))
      .whenComplete(() => print('Écran de chargement masqué.'));
}
```

**Résultat :**

```text
--- Cas 2 : échec ---
Erreur : Exception: Fichier de niveau corrompu
Écran de chargement masqué.
```

Dans les deux cas, la dernière ligne s'affiche. C'est toute l'utilité de `.whenComplete()`.

| Méthode | Appelée quand ? | Reçoit |
| --- | --- | --- |
| `.then()` | succès uniquement | la valeur |
| `.catchError()` | erreur uniquement | l'erreur |
| `.whenComplete()` | toujours | rien |

---

## 15.12 — Pourquoi `async` / `await` est plus lisible que `.then()`

`.then()` fonctionne, mais il devient vite illisible dès que les étapes s'enchaînent. Voici un chargement de partie en quatre étapes, écrit avec `.then()` imbriqués. C'est le style que les développeurs appellent la « pyramide de l'enfer ».

```dart
Future<String> chargerProfil() =>
    Future.delayed(Duration(milliseconds: 300), () => 'Alex');

Future<int> chargerNiveau(String nom) =>
    Future.delayed(Duration(milliseconds: 300), () => 7);

Future<List<String>> chargerInventaire(int niveau) =>
    Future.delayed(Duration(milliseconds: 300), () => ['Potion', 'Épée']);

Future<String> chargerCarte(int niveau) =>
    Future.delayed(Duration(milliseconds: 300), () => 'Caverne de glace');

void main() {
  chargerProfil().then((String nom) {
    chargerNiveau(nom).then((int niveau) {
      chargerInventaire(niveau).then((List<String> sac) {
        chargerCarte(niveau).then((String carte) {
          print('$nom (niv. $niveau) sur "$carte" avec $sac');
        });
      });
    });
  });
}
```

**Résultat :**

```text
Alex (niv. 7) sur "Caverne de glace" avec [Potion, Épée]
```

Le résultat est bon, mais le code est mauvais :

- l'indentation part vers la droite et n'en revient pas ;
- gérer une erreur à chaque niveau demanderait quatre `.catchError()` ;
- ajouter une cinquième étape ajoute un niveau d'imbrication.

Voici **exactement le même programme** avec `async` / `await` :

```dart
Future<String> chargerProfil() =>
    Future.delayed(Duration(milliseconds: 300), () => 'Alex');

Future<int> chargerNiveau(String nom) =>
    Future.delayed(Duration(milliseconds: 300), () => 7);

Future<List<String>> chargerInventaire(int niveau) =>
    Future.delayed(Duration(milliseconds: 300), () => ['Potion', 'Épée']);

Future<String> chargerCarte(int niveau) =>
    Future.delayed(Duration(milliseconds: 300), () => 'Caverne de glace');

Future<void> main() async {
  final String nom = await chargerProfil();
  final int niveau = await chargerNiveau(nom);
  final List<String> sac = await chargerInventaire(niveau);
  final String carte = await chargerCarte(niveau);

  print('$nom (niv. $niveau) sur "$carte" avec $sac');
}
```

**Résultat :**

```text
Alex (niv. 7) sur "Caverne de glace" avec [Potion, Épée]
```

Quatre lignes plates, lues de haut en bas, comme du code synchrone. C'est le même comportement asynchrone, avec une lisibilité de code ordinaire.

> **Règle du chapitre :** écrivez vos programmes avec `async` / `await`. Réservez `.then()` aux cas où vous ne pouvez pas rendre la fonction appelante `async`.

---

## 15.13 — Le mot-clé `async`

`async` se place **entre la parenthèse fermante et l'accolade ouvrante** d'une fonction :

```text
  Future<String> chargerProfil() async {
                                 ^^^^^
  }
```

`async` fait deux choses, et deux seulement :

1. il **autorise** l'usage de `await` à l'intérieur de la fonction ;
2. il **transforme** le type de retour : la fonction retourne désormais un `Future`.

```dart
Future<void> direBonjour() async {
  print('Bonjour, joueur.');
}

void main() {
  final Future<void> resultat = direBonjour();
  print('Type retourné : ${resultat.runtimeType}');
}
```

**Résultat :**

```text
Bonjour, joueur.
Type retourné : Future<void>
```

Observez la première ligne : `Bonjour, joueur.` s'affiche **avant** le `print` de `main`. C'est un point capital, souvent mal compris :

> Le corps d'une fonction `async` démarre **immédiatement et de façon synchrone**. Il ne devient asynchrone qu'au premier `await` rencontré.

Démontrons-le proprement :

```dart
Future<void> preparerLeNiveau() async {
  print('B — début de preparerLeNiveau');
  await Future.delayed(Duration(seconds: 1));
  print('D — fin de preparerLeNiveau');
}

void main() {
  print('A — début de main');
  preparerLeNiveau();
  print('C — fin de main');
}
```

**Résultat (ordre réel) :**

```text
A — début de main
B — début de preparerLeNiveau
C — fin de main
D — fin de preparerLeNiveau
```

Lecture du déroulé :

```text
  A  main démarre
  B  on entre dans preparerLeNiveau : le code AVANT le premier await
     s'exécute tout de suite
     -> au await, la fonction rend la main à main()
  C  main termine
     -> l'event loop attend le timer d'une seconde
  D  la suite de preparerLeNiveau reprend là où elle s'était arrêtée
```

---

## 15.14 — Le mot-clé `await`

`await` se place devant une expression qui produit un `Future`. Il signifie :

> « suspends cette fonction ici, laisse l'event loop travailler, et reprends avec la valeur quand le `Future` sera terminé ».

`await` **transforme un `Future<T>` en `T`** :

```text
  Future<String>            await          String
  (le ticket)        ─────────────────>   (le plat)
```

C'est la réponse au problème de la section 15.5.

```dart
Future<String> chargerProfil() {
  return Future.delayed(Duration(seconds: 1), () => 'Alex');
}

Future<void> main() async {
  // SANS await : on affiche le ticket
  final Future<String> ticket = chargerProfil();
  print('Sans await : $ticket');

  // AVEC await : on affiche la valeur
  final String nom = await chargerProfil();
  print('Avec await : $nom');
}
```

**Résultat :**

```text
Sans await : Instance of 'Future<String>'
Avec await : Alex
```

La première ligne est **le** symptôme à reconnaître. Chaque fois que vous voyez `Instance of 'Future<...>'` dans votre console, la correction est toujours la même : il manque un `await`.

Deux règles absolues :

| Règle | Conséquence si on l'oublie |
| --- | --- |
| `await` ne peut s'utiliser que dans une fonction marquée `async` | Erreur de compilation |
| `await` ne suspend que **la fonction courante**, jamais le programme entier | L'event loop continue de tourner : c'est tout l'intérêt |

Vérifions le message d'erreur exact du premier cas :

```dart
void main() {
  final String nom = await Future.value('Alex');
  print(nom);
}
```

**Résultat :**

```text
Error: 'await' can only be used in 'async' or 'async*' methods.
```

La correction tient en deux mots : ajouter `async` et changer le type de retour en `Future<void>`.

---

## 15.15 — Une fonction `async` retourne toujours un `Future`

C'est une règle sans exception :

> Toute fonction marquée `async` retourne un `Future`, **même si vous n'écrivez aucun `return`**.

| Ce que vous écrivez | Ce que la fonction retourne vraiment |
| --- | --- |
| `Future<void> f() async { }` | `Future<void>` |
| `Future<String> f() async { return 'Alex'; }` | `Future<String>` |
| `Future<int> f() async { return 7; }` | `Future<int>` |

Conséquence pratique très importante : **`void f() async` est presque toujours une erreur de conception**, parce que l'appelant n'a alors aucun moyen d'attendre la fin de l'opération ni d'attraper ses erreurs.

```dart
// À ÉVITER : personne ne peut attendre cette fonction
void sauvegarderMauvais() async {
  await Future.delayed(Duration(seconds: 1));
  print('Sauvegarde terminée (mauvaise version)');
}

// CORRECT : l'appelant peut attendre et attraper les erreurs
Future<void> sauvegarderBon() async {
  await Future.delayed(Duration(seconds: 1));
  print('Sauvegarde terminée (bonne version)');
}

Future<void> main() async {
  sauvegarderMauvais(); // on ne peut PAS écrire "await" ici de façon utile
  await sauvegarderBon();
  print('Fin de main');
}
```

**Résultat :**

```text
Sauvegarde terminée (mauvaise version)
Sauvegarde terminée (bonne version)
Fin de main
```

Les deux sauvegardes durent une seconde et démarrent quasiment en même temps, donc elles se terminent quasiment ensemble. Mais seule la seconde était réellement **attendue** : `Fin de main` est garanti après `sauvegarderBon`, pas après `sauvegarderMauvais`.

> **Exception à la règle :** en Flutter, les gestionnaires d'événements (`onPressed: () async { ... }`) sont bien de type `void`. C'est le seul cas où `void ... async` est normal.

---

## 15.16 — `Future<String>` : retourner une valeur depuis une fonction `async`

Dans une fonction `async`, vous écrivez `return` comme d'habitude. Dart emballe automatiquement la valeur dans un `Future`.

```text
  Future<String> f() async {
      return 'Alex';        <- vous rendez une String
  }                         <- Dart livre un Future<String>
```

```dart
Future<String> chargerNomJoueur() async {
  await Future.delayed(Duration(seconds: 1));
  return 'Alex'; // une String, pas un Future<String>
}

Future<int> chargerScore() async {
  await Future.delayed(Duration(seconds: 1));
  return 4200;
}

Future<List<String>> chargerInventaire() async {
  await Future.delayed(Duration(seconds: 1));
  return ['Potion de vie', 'Épée de feu', 'Clé rouillée'];
}

Future<void> main() async {
  final String nom = await chargerNomJoueur();
  final int score = await chargerScore();
  final List<String> sac = await chargerInventaire();

  print('Joueur    : $nom');
  print('Score     : $score');
  print('Inventaire: ${sac.join(', ')}');
  print('Objets    : ${sac.length}');
}
```

**Résultat (après environ 3 secondes) :**

```text
Joueur    : Alex
Score     : 4200
Inventaire: Potion de vie, Épée de feu, Clé rouillée
Objets    : 3
```

Erreur classique du débutant : écrire le type de retour sans `Future`.

```dart
String chargerNomJoueur() async {
  return 'Alex';
}

void main() {
  print(chargerNomJoueur());
}
```

**Résultat :**

```text
Error: Functions marked 'async' must have a return type assignable to 'Future'.
```

La correction est mécanique : `String` devient `Future<String>`.

---

## 15.17 — `try` / `catch` avec `await` (rappel du chapitre 13)

Voici la meilleure nouvelle du chapitre. Avec `await`, **le `try` / `catch` ordinaire fonctionne à nouveau**.

Souvenez-vous de la section 15.10 : sans `await`, le `try` se terminait avant l'erreur. Avec `await`, la fonction est suspendue **à l'intérieur** du `try`, donc l'erreur survient bien pendant le bloc protégé.

```dart
Future<String> chargerSauvegarde(String fichier) async {
  await Future.delayed(Duration(seconds: 1));
  if (fichier.isEmpty) {
    throw FormatException('Nom de fichier vide');
  }
  if (fichier != 'partie1.sav') {
    throw Exception('Fichier introuvable : $fichier');
  }
  return 'Alex — niveau 7 — 4200 points';
}

Future<void> essayer(String fichier) async {
  try {
    final String donnees = await chargerSauvegarde(fichier);
    print('Chargé : $donnees');
  } on FormatException catch (e) {
    print('Sauvegarde illisible : ${e.message}');
  } catch (e) {
    print('Erreur : $e');
  } finally {
    print('-> écran de chargement masqué');
  }
}

Future<void> main() async {
  await essayer('partie1.sav');
  await essayer('partie9.sav');
  await essayer('');
}
```

**Résultat :**

```text
Chargé : Alex — niveau 7 — 4200 points
-> écran de chargement masqué
Erreur : Exception: Fichier introuvable : partie9.sav
-> écran de chargement masqué
Sauvegarde illisible : Nom de fichier vide
-> écran de chargement masqué
```

Tout ce que vous avez appris au chapitre 13 s'applique sans changement : `on Type catch (e)`, `catch (e)`, `finally`, `rethrow`, vos propres classes d'exception.

| Style `.then()` | Style `async` / `await` |
| --- | --- |
| `.catchError((e) { ... })` | `catch (e) { ... }` |
| pas d'équivalent direct de `on Type` | `on FormatException catch (e)` |
| `.whenComplete(() { ... })` | `finally { ... }` |

> **À retenir :** `await` ramène les erreurs asynchrones dans le monde familier du `try` / `catch`. C'est la deuxième grande raison de préférer `async` / `await` à `.then()`.

---

## 15.18 — Enchaîner plusieurs `await`

Quand une étape a besoin du résultat de la précédente, on enchaîne les `await` de haut en bas. C'est le cas le plus courant et le plus simple.

```dart
Future<String> connecterJoueur(String pseudo) async {
  await Future.delayed(Duration(seconds: 1));
  print('   -> connexion réussie');
  return 'TOKEN-$pseudo';
}

Future<int> chargerNiveau(String token) async {
  await Future.delayed(Duration(seconds: 1));
  print('   -> niveau récupéré avec $token');
  return 7;
}

Future<String> chargerCarte(int niveau) async {
  await Future.delayed(Duration(seconds: 1));
  print('   -> carte du niveau $niveau récupérée');
  return 'Caverne de glace';
}

Future<void> main() async {
  print('1. Connexion...');
  final String token = await connecterJoueur('Alex');

  print('2. Chargement du niveau...');
  final int niveau = await chargerNiveau(token);

  print('3. Chargement de la carte...');
  final String carte = await chargerCarte(niveau);

  print('4. Prêt : Alex, niveau $niveau, sur "$carte".');
}
```

**Résultat (ordre réel, une étape par seconde) :**

```text
1. Connexion...
   -> connexion réussie
2. Chargement du niveau...
   -> niveau récupéré avec TOKEN-Alex
3. Chargement de la carte...
   -> carte du niveau 7 récupérée
4. Prêt : Alex, niveau 7, sur "Caverne de glace".
```

Ici l'enchaînement est **obligatoire** : impossible de charger le niveau sans le token, ni la carte sans le numéro de niveau. On parle de dépendance de données.

Le code se lit exactement comme du code synchrone. C'est voulu : `await` a été conçu pour cela.

---

## 15.19 — Le piège : des `await` en série alors qu'ils pourraient être parallèles

Voici maintenant l'erreur de performance la plus fréquente de tout le chapitre.

Reprenons trois chargements qui, cette fois, **ne dépendent pas les uns des autres** : le profil, l'inventaire et la carte peuvent être demandés en même temps.

```dart
Future<String> chargerProfil() async {
  await Future.delayed(Duration(seconds: 2));
  return 'Alex';
}

Future<List<String>> chargerInventaire() async {
  await Future.delayed(Duration(seconds: 3));
  return ['Potion', 'Épée de feu'];
}

Future<String> chargerCarte() async {
  await Future.delayed(Duration(seconds: 1));
  return 'Caverne de glace';
}

Future<void> main() async {
  final Stopwatch chrono = Stopwatch()..start();

  final String profil = await chargerProfil();
  final List<String> sac = await chargerInventaire();
  final String carte = await chargerCarte();

  chrono.stop();

  print('Profil     : $profil');
  print('Inventaire : $sac');
  print('Carte      : $carte');
  print('Durée      : ${chrono.elapsed.inSeconds} s');
}
```

**Résultat :**

```text
Profil     : Alex
Inventaire : [Potion, Épée de feu]
Carte      : Caverne de glace
Durée      : 6 s
```

Six secondes. Pourtant, l'opération la plus longue ne dure que trois secondes. Où sont passées les trois autres ?

Le schéma répond :

```text
  SÉRIE (trois await à la suite) — total 6 s

  0s        1s        2s        3s        4s        5s        6s
  |---------|---------|---------|---------|---------|---------|
  [==== profil (2s) ==]
                      [======= inventaire (3s) =======]
                                                      [carte 1s]
                                                               ^
                                                          terminé à 6 s


  PARALLÈLE (les trois lancés ensemble) — total 3 s

  0s        1s        2s        3s
  |---------|---------|---------|
  [==== profil (2s) ==]
  [======= inventaire (3s) =====]
  [carte 1s]
                                ^
                           terminé à 3 s
```

En série, chaque `await` **suspend la fonction** avant même que la requête suivante ne soit lancée. Le second chargement ne démarre qu'une fois le premier terminé.

En parallèle, les trois opérations sont lancées d'abord, puis on attend. La durée totale devient celle de la **plus longue**, et non la **somme**.

> **Le test à faire mentalement :** « l'étape B a-t-elle besoin du résultat de l'étape A ? »
> Si oui, série obligatoire. Si non, vous perdez du temps pour rien.

Notez qu'il est déjà possible de paralléliser à la main, sans nouvel outil : il suffit de lancer les trois futures avant d'attendre.

```dart
Future<String> chargerProfil() async {
  await Future.delayed(Duration(seconds: 2));
  return 'Alex';
}

Future<List<String>> chargerInventaire() async {
  await Future.delayed(Duration(seconds: 3));
  return ['Potion', 'Épée de feu'];
}

Future<String> chargerCarte() async {
  await Future.delayed(Duration(seconds: 1));
  return 'Caverne de glace';
}

Future<void> main() async {
  final Stopwatch chrono = Stopwatch()..start();

  // 1) On lance les trois opérations SANS await : elles démarrent toutes.
  final Future<String> futProfil = chargerProfil();
  final Future<List<String>> futSac = chargerInventaire();
  final Future<String> futCarte = chargerCarte();

  // 2) On attend ensuite les résultats.
  final String profil = await futProfil;
  final List<String> sac = await futSac;
  final String carte = await futCarte;

  chrono.stop();

  print('Profil     : $profil');
  print('Inventaire : $sac');
  print('Carte      : $carte');
  print('Durée      : ${chrono.elapsed.inSeconds} s');
}
```

**Résultat :**

```text
Profil     : Alex
Inventaire : [Potion, Épée de feu]
Carte      : Caverne de glace
Durée      : 3 s
```

Deux fois plus rapide, pour un jeu identique. La différence tient uniquement à l'endroit où l'on place les `await`.

---

## 15.20 — `Future.wait()` : attendre plusieurs futures proprement

Écrire soi-même les variables `futProfil`, `futSac`, `futCarte` fonctionne, mais devient lourd. `Future.wait()` fait la même chose en une ligne.

`Future.wait()` prend une **liste de futures** et retourne un `Future` qui livre la **liste des résultats**, dans le même ordre.

```text
  Future.wait([ futureA, futureB, futureC ])
        |
        v
  Future<List<...>>  qui se termine quand LES TROIS sont terminés
```

```dart
Future<String> chargerProfil() async {
  await Future.delayed(Duration(seconds: 2));
  return 'Alex';
}

Future<String> chargerInventaire() async {
  await Future.delayed(Duration(seconds: 3));
  return 'Potion, Épée de feu';
}

Future<String> chargerCarte() async {
  await Future.delayed(Duration(seconds: 1));
  return 'Caverne de glace';
}

Future<void> main() async {
  final Stopwatch chrono = Stopwatch()..start();

  final List<String> resultats = await Future.wait([
    chargerProfil(),
    chargerInventaire(),
    chargerCarte(),
  ]);

  chrono.stop();

  print('Profil     : ${resultats[0]}');
  print('Inventaire : ${resultats[1]}');
  print('Carte      : ${resultats[2]}');
  print('Durée      : ${chrono.elapsed.inSeconds} s');
}
```

**Résultat :**

```text
Profil     : Alex
Inventaire : Potion, Épée de feu
Carte      : Caverne de glace
Durée      : 3 s
```

Trois points à connaître sur `Future.wait()` :

1. **L'ordre des résultats est celui de la liste**, pas celui d'arrivée. Même si la carte arrive en premier, elle reste à l'indice 2.
2. **Tous les futures doivent avoir le même type**, sinon la liste devient `List<dynamic>` et vous devrez convertir manuellement.
3. **Si un seul future échoue, `Future.wait()` échoue.** Les autres continuent en arrière-plan, mais l'erreur remonte immédiatement.

Illustration du troisième point :

```dart
Future<String> chargerProfil() async {
  await Future.delayed(Duration(seconds: 1));
  return 'Alex';
}

Future<String> chargerInventaire() async {
  await Future.delayed(Duration(seconds: 2));
  throw Exception('Inventaire corrompu');
}

Future<void> main() async {
  try {
    final List<String> r = await Future.wait([
      chargerProfil(),
      chargerInventaire(),
    ]);
    print('Tout est chargé : $r');
  } catch (e) {
    print('Chargement interrompu : $e');
  }
}
```

**Résultat :**

```text
Chargement interrompu : Exception: Inventaire corrompu
```

`Future.wait()` accepte aussi le paramètre `eagerError: true`, qui fait remonter l'erreur dès qu'elle survient au lieu d'attendre que tous les futures soient terminés.

`Future.wait()` s'associe très bien avec `map` du chapitre 14 pour lancer N opérations d'un coup :

```dart
Future<String> chargerTexture(String nom) async {
  await Future.delayed(Duration(milliseconds: 500));
  return 'texture_$nom.png';
}

Future<void> main() async {
  final List<String> aCharger = ['heros', 'gobelin', 'boss', 'potion'];

  final List<String> textures = await Future.wait(
    aCharger.map((String nom) => chargerTexture(nom)),
  );

  for (final String t in textures) {
    print('Chargée : $t');
  }
}
```

**Résultat (après environ 0,5 s au total, pas 2 s) :**

```text
Chargée : texture_heros.png
Chargée : texture_gobelin.png
Chargée : texture_boss.png
Chargée : texture_potion.png
```

---

## 15.21 — `Future.value()` et `Future.error()`

Ces deux constructeurs créent un `Future` **déjà terminé**. Ils sont indispensables pour les tests, les caches et les valeurs de repli.

| Constructeur | Crée un `Future`... |
| --- | --- |
| `Future.value(x)` | déjà terminé **avec la valeur** `x` |
| `Future.error(e)` | déjà terminé **avec l'erreur** `e` |

Cas typique : un cache. Si la donnée est déjà en mémoire, inutile d'aller sur le réseau, mais la signature doit rester un `Future`.

```dart
final Map<String, String> cache = <String, String>{};

Future<String> chargerProfil(String pseudo) {
  if (cache.containsKey(pseudo)) {
    print('   (cache)');
    return Future.value(cache[pseudo]!); // immédiat
  }
  print('   (réseau)');
  return Future.delayed(Duration(seconds: 2), () {
    final String profil = '$pseudo — niveau 7';
    cache[pseudo] = profil;
    return profil;
  });
}

Future<void> main() async {
  print('Premier appel :');
  print(await chargerProfil('Alex'));

  print('Second appel :');
  print(await chargerProfil('Alex'));
}
```

**Résultat (le premier appel prend 2 s, le second est instantané) :**

```text
Premier appel :
   (réseau)
Alex — niveau 7
Second appel :
   (cache)
Alex — niveau 7
```

`Future.error()` sert à signaler un échec sans passer par `throw` :

```dart
Future<String> chargerSauvegarde(String fichier) {
  if (fichier.isEmpty) {
    return Future.error(ArgumentError('Nom de fichier vide'));
  }
  return Future.delayed(Duration(seconds: 1), () => 'Alex — niveau 7');
}

Future<void> main() async {
  try {
    print(await chargerSauvegarde('partie1.sav'));
    print(await chargerSauvegarde(''));
  } catch (e) {
    print('Erreur : $e');
  }
}
```

**Résultat :**

```text
Alex — niveau 7
Erreur : Invalid argument(s): Nom de fichier vide
```

> **Attention :** un `Future.error()` que personne n'attend ni n'attrape provoque une erreur non gérée qui peut arrêter le programme. Un `Future` en erreur doit **toujours** trouver un `await` dans un `try` ou un `.catchError()`.

Notez aussi que `Future.value()` reste **asynchrone**, même s'il est déjà terminé :

```dart
void main() {
  print('A');
  Future.value('B').then((String v) => print(v));
  print('C');
}
```

**Résultat :**

```text
A
C
B
```

La suite d'un `Future` passe toujours par la file des microtâches de l'event loop. Elle ne peut donc jamais s'exécuter avant la fin du code synchrone en cours.

---

## 15.22 — `unawaited` et les futures oubliés

Un **future oublié** (*fire and forget*) est un `Future` que l'on lance sans jamais l'attendre. Ce n'est pas toujours une erreur : envoyer une statistique de jeu à un serveur d'analyse ne doit pas retarder le démarrage de la partie.

Le problème est que l'analyseur ne peut pas deviner votre intention. Il déclenche l'avertissement `unawaited_futures` :

```text
Info: Missing an 'await' for the 'Future' computed by this expression.
```

Deux situations, deux lectures :

| Situation | Signification |
| --- | --- |
| Vous avez **oublié** un `await` | Bug réel : la suite s'exécute trop tôt, les erreurs sont perdues |
| Vous ne voulez **volontairement pas** attendre | Il faut le dire explicitement au lecteur et à l'outil |

`unawaited()`, fourni par `dart:async`, sert exactement à cela : il documente « je sais, et c'est voulu ».

```dart
import 'dart:async';

Future<void> envoyerStatistiques(String evenement) async {
  await Future.delayed(Duration(seconds: 2));
  print('   [stats] "$evenement" envoyé au serveur');
}

Future<void> demarrerPartie() async {
  await Future.delayed(Duration(milliseconds: 500));
  print('Partie démarrée.');
}

Future<void> main() async {
  // Volontairement non attendu : ne doit pas retarder le joueur.
  unawaited(envoyerStatistiques('partie_lancee'));

  await demarrerPartie();
  print('Le joueur peut jouer.');

  // On laisse le programme vivre assez longtemps pour voir la statistique.
  await Future.delayed(Duration(seconds: 2));
}
```

**Résultat (ordre réel) :**

```text
Partie démarrée.
Le joueur peut jouer.
   [stats] "partie_lancee" envoyé au serveur
```

Danger à connaître : dans un future oublié, une exception **n'est attrapée par personne**.

```dart
import 'dart:async';

Future<void> envoyerStatistiques() async {
  await Future.delayed(Duration(milliseconds: 300));
  throw Exception('Serveur de stats injoignable');
}

Future<void> main() async {
  // Mauvais : l'erreur remontera comme exception non gérée.
  // unawaited(envoyerStatistiques());

  // Bon : on avale l'erreur volontairement, en la traçant.
  unawaited(
    envoyerStatistiques().catchError((Object e) {
      print('[stats] ignorée : $e');
    }),
  );

  print('Le jeu continue normalement.');
  await Future.delayed(Duration(seconds: 1));
}
```

**Résultat :**

```text
Le jeu continue normalement.
[stats] ignorée : Exception: Serveur de stats injoignable
```

> **Règle :** un future oublié doit **toujours** porter son propre `.catchError()`. Sinon, une panne du serveur de statistiques peut faire tomber tout votre jeu.

---

## 15.23 — `main()` asynchrone

`main()` est une fonction comme les autres : elle peut être `async`.

```text
  void main() { ... }                 <- version synchrone
  Future<void> main() async { ... }   <- version asynchrone
```

Dès que vous avez besoin d'un `await` au premier niveau de votre programme, la version asynchrone est obligatoire.

```dart
Future<String> chargerProfil() async {
  await Future.delayed(Duration(seconds: 1));
  return 'Alex';
}

Future<void> main() async {
  print('Démarrage du jeu...');
  final String nom = await chargerProfil();
  print('Bienvenue, $nom.');
}
```

**Résultat :**

```text
Démarrage du jeu...
Bienvenue, Alex.
```

Point important sur la **fin du programme** : Dart ne s'arrête pas à la dernière ligne de `main()`. Il s'arrête quand l'event loop n'a plus rien à faire.

```dart
void main() {
  print('Début de main');

  Future.delayed(Duration(seconds: 2), () {
    print('Le timer a fini, bien après main');
  });

  print('Fin de main');
}
```

**Résultat :**

```text
Début de main
Fin de main
Le timer a fini, bien après main
```

`main()` est terminé depuis deux secondes quand la troisième ligne s'affiche. Le programme est resté en vie parce qu'un timer était encore enregistré.

> **Conséquence pratique :** un test qui « se termine trop vite » et n'affiche rien vient presque toujours d'un `await` manquant dans `main()`.

---

## 15.24 — Introduction aux `Stream`

Un `Future` livre **une** valeur, **une seule fois**. Or beaucoup de choses, dans un jeu, arrivent **plusieurs fois** :

- les positions successives d'un ennemi qui se déplace ;
- les touches appuyées par le joueur ;
- les messages du tchat d'une partie en ligne ;
- les points de vie du héros, qui changent à chaque coup reçu ;
- la progression d'un téléchargement : 10 %, 25 %, 60 %, 100 %.

Pour cela, Dart fournit le `Stream` (« flux »).

> Un `Stream<T>` est une **suite de valeurs de type `T` qui arrivent au fil du temps**.

L'image la plus juste est celle du tapis roulant :

```text
   FUTURE : un colis, livré une fois

        ┌──────┐
   ─────│ Alex │─────>   fin
        └──────┘


   STREAM : un tapis roulant qui apporte des colis

        ┌───┐   ┌───┐   ┌───┐   ┌───┐
   ─────│ 3 │───│ 2 │───│ 1 │───│ 0 │────>  fin du flux
        └───┘   └───┘   └───┘   └───┘
         t=1s    t=2s    t=3s    t=4s
```

Un `Stream` peut se terminer (le tapis s'arrête) ou ne jamais se terminer (les clics du joueur, tant que le jeu tourne). Il peut aussi émettre une **erreur**, comme un `Future`.

---

## 15.25 — `Future` contre `Stream`

| | `Future<T>` | `Stream<T>` |
| --- | --- | --- |
| Nombre de valeurs | exactement une | zéro, une, plusieurs, une infinité |
| Analogie | un colis | un tapis roulant |
| Création simple | `Future.delayed()`, `async` | `async*` avec `yield` |
| Consommation | `await`, `.then()` | `await for`, `.listen()` |
| Erreur | une seule, qui termine tout | possible à chaque valeur |
| Exemple de jeu | charger le profil du joueur | les points de vie au fil du combat |
| Widget Flutter associé | `FutureBuilder` | `StreamBuilder` |

Le critère de choix tient en une question :

> **« Combien de fois cette information va-t-elle m'être livrée ? »**
> Une fois : `Future`. Plusieurs fois : `Stream`.

Charger la carte du niveau : une fois, donc `Future`. Suivre la position du boss pendant le combat : en continu, donc `Stream`.

---

## 15.26 — Créer un `Stream` avec `async*` et `yield`

Pour écrire une fonction qui produit un flux, on utilise deux mots-clés nouveaux :

| Mot-clé | Rôle |
| --- | --- |
| `async*` | marque la fonction comme **génératrice asynchrone** ; elle retourne un `Stream` |
| `yield` | **émet une valeur** dans le flux, puis la fonction continue |

Comparez avec ce que vous connaissez :

```text
  async  + return  ->  Future<T>  : je livre UNE valeur et je m'arrête
  async* + yield   ->  Stream<T>  : je livre UNE valeur et je CONTINUE
```

Un compte à rebours avant le début de la partie :

```dart
Stream<int> compteARebours(int depart) async* {
  for (int i = depart; i >= 1; i--) {
    await Future.delayed(Duration(seconds: 1));
    yield i;
  }
}

Future<void> main() async {
  print('Préparez-vous...');
  await for (final int n in compteARebours(3)) {
    print('$n...');
  }
  print('Partie lancée !');
}
```

**Résultat (une ligne par seconde) :**

```text
Préparez-vous...
3...
2...
1...
Partie lancée !
```

Un flux de points de vie pendant un combat :

```dart
Stream<int> combat(int pvDepart) async* {
  int pv = pvDepart;
  final List<int> degats = [12, 8, 25, 5, 30, 40];

  for (final int d in degats) {
    await Future.delayed(Duration(milliseconds: 400));
    pv = pv - d;
    if (pv < 0) {
      pv = 0;
    }
    yield pv;
    if (pv == 0) {
      return; // on arrête le flux : le héros est mort
    }
  }
}

Future<void> main() async {
  await for (final int pv in combat(100)) {
    print('Points de vie : $pv');
  }
  print('Combat terminé.');
}
```

**Résultat :**

```text
Points de vie : 88
Points de vie : 80
Points de vie : 55
Points de vie : 50
Points de vie : 20
Points de vie : 0
Combat terminé.
```

> **Remarque :** dans une fonction `async*`, `return` ne renvoie aucune valeur. Il sert uniquement à **fermer le flux**.

Point important : un `Stream` créé par `async*` est **paresseux**. Tant que personne ne l'écoute, aucune ligne de son corps ne s'exécute.

```dart
Stream<int> compteARebours(int depart) async* {
  print('   (le corps du stream démarre)');
  for (int i = depart; i >= 1; i--) {
    yield i;
  }
}

Future<void> main() async {
  final Stream<int> flux = compteARebours(3);
  print('Le stream est créé, mais rien ne s\'est passé.');
  await for (final int n in flux) {
    print('valeur : $n');
  }
}
```

**Résultat :**

```text
Le stream est créé, mais rien ne s'est passé.
   (le corps du stream démarre)
valeur : 3
valeur : 2
valeur : 1
```

---

## 15.27 — `await for` : consommer un flux dans une boucle

`await for` est la boucle `for` des `Stream`. Elle attend chaque valeur, exécute le corps, puis attend la suivante, jusqu'à la fin du flux.

```text
  for (final x in liste)      -> les valeurs sont DÉJÀ toutes là
  await for (final x in flux) -> les valeurs ARRIVENT une par une
```

Deux contraintes, identiques à celles de `await` :

1. `await for` ne s'utilise que dans une fonction `async` ;
2. il **suspend la fonction** jusqu'à la fermeture du flux. Sur un flux infini, le code qui suit ne s'exécutera jamais.

```dart
Stream<String> messagesDuTchat() async* {
  yield 'Maria : bien joué !';
  await Future.delayed(Duration(milliseconds: 500));
  yield 'Samir : attention au boss';
  await Future.delayed(Duration(milliseconds: 500));
  yield 'Maria : j\'arrive';
}

Future<void> main() async {
  print('--- Tchat ---');
  await for (final String message in messagesDuTchat()) {
    print(message);
  }
  print('--- Fin du tchat ---');
}
```

**Résultat :**

```text
--- Tchat ---
Maria : bien joué !
Samir : attention au boss
Maria : j'arrive
--- Fin du tchat ---
```

`break` fonctionne comme dans une boucle normale et ferme l'abonnement au flux :

```dart
Stream<int> vagueDEnnemis() async* {
  for (int i = 1; i <= 100; i++) {
    await Future.delayed(Duration(milliseconds: 100));
    yield i;
  }
}

Future<void> main() async {
  await for (final int ennemi in vagueDEnnemis()) {
    print('Ennemi $ennemi éliminé');
    if (ennemi == 3) {
      print('Le joueur fuit le combat.');
      break;
    }
  }
  print('Retour au village.');
}
```

**Résultat :**

```text
Ennemi 1 éliminé
Ennemi 2 éliminé
Ennemi 3 éliminé
Le joueur fuit le combat.
Retour au village.
```

Les 97 ennemis restants ne sont jamais produits : le flux est paresseux et il a été fermé.

Les erreurs d'un flux s'attrapent avec un `try` / `catch` autour du `await for` :

```dart
Stream<int> capteurDeVie() async* {
  yield 100;
  yield 75;
  throw Exception('Capteur de vie déconnecté');
}

Future<void> main() async {
  try {
    await for (final int pv in capteurDeVie()) {
      print('PV : $pv');
    }
  } catch (e) {
    print('Flux interrompu : $e');
  }
}
```

**Résultat :**

```text
PV : 100
PV : 75
Flux interrompu : Exception: Capteur de vie déconnecté
```

---

## 15.28 — `listen()` : s'abonner sans bloquer

`listen()` est l'autre façon de consommer un `Stream`. La différence est capitale :

| | `await for` | `.listen()` |
| --- | --- | --- |
| Bloque la fonction courante | oui, jusqu'à la fin du flux | non, jamais |
| Utilisable hors d'une fonction `async` | non | oui |
| Convient à un flux infini | non | oui |
| Peut être annulé | par `break` | par `.cancel()` |

```dart
Stream<int> compteARebours(int depart) async* {
  for (int i = depart; i >= 1; i--) {
    await Future.delayed(Duration(milliseconds: 500));
    yield i;
  }
}

void main() {
  print('A — abonnement');

  compteARebours(3).listen(
    (int n) => print('C — reçu : $n'),
    onError: (Object e) => print('erreur : $e'),
    onDone: () => print('D — flux terminé'),
  );

  print('B — la suite du programme continue tout de suite');
}
```

**Résultat (ordre réel) :**

```text
A — abonnement
B — la suite du programme continue tout de suite
C — reçu : 3
C — reçu : 2
C — reçu : 1
D — flux terminé
```

`listen()` retourne un objet `StreamSubscription` (« abonnement ») que l'on peut annuler. C'est indispensable dans un jeu : quand le joueur quitte l'écran de combat, il faut cesser d'écouter les points de vie du boss.

```dart
import 'dart:async';

Stream<int> positionDuBoss() async* {
  int x = 0;
  while (true) {
    await Future.delayed(Duration(milliseconds: 300));
    x = x + 10;
    yield x;
  }
}

Future<void> main() async {
  final StreamSubscription<int> abonnement =
      positionDuBoss().listen((int x) => print('Boss en x = $x'));

  await Future.delayed(Duration(seconds: 1));
  await abonnement.cancel();

  print('Écran de combat quitté, plus aucune écoute.');
}
```

**Résultat :**

```text
Boss en x = 10
Boss en x = 20
Boss en x = 30
Écran de combat quitté, plus aucune écoute.
```

Sans le `cancel()`, le flux serait infini et le programme ne s'arrêterait jamais.

---

## 15.29 — Opérations sur un `Stream` : `map`, `where` et les autres

Un `Stream` propose les mêmes transformations que les collections du chapitre 14, à une différence près : elles sont **paresseuses** et s'appliquent à chaque valeur au moment où elle arrive.

| Méthode | Rôle |
| --- | --- |
| `map()` | transforme chaque valeur |
| `where()` | ne laisse passer que les valeurs qui vérifient une condition |
| `take(n)` | ne garde que les `n` premières valeurs, puis ferme le flux |
| `skip(n)` | ignore les `n` premières valeurs |
| `length` | `Future<int>` : le nombre total de valeurs |
| `toList()` | `Future<List<T>>` : toutes les valeurs, une fois le flux fermé |
| `first` / `last` | `Future<T>` : la première / la dernière valeur |

```dart
Stream<int> scoresDeLaPartie() async* {
  final List<int> scores = [120, 45, 800, 30, 650, 90];
  for (final int s in scores) {
    await Future.delayed(Duration(milliseconds: 200));
    yield s;
  }
}

Future<void> main() async {
  print('--- Tous les scores ---');
  await for (final int s in scoresDeLaPartie()) {
    print(s);
  }

  print('--- Seulement les scores > 100 ---');
  await for (final int s in scoresDeLaPartie().where((int s) => s > 100)) {
    print(s);
  }

  print('--- Scores doublés (bonus x2) ---');
  await for (final int s in scoresDeLaPartie().map((int s) => s * 2)) {
    print(s);
  }
}
```

**Résultat :**

```text
--- Tous les scores ---
120
45
800
30
650
90
--- Seulement les scores > 100 ---
120
800
650
--- Scores doublés (bonus x2) ---
240
90
1600
60
1300
180
```

Les transformations se chaînent, exactement comme au chapitre 14 :

```dart
Stream<int> degatsInfliges() async* {
  final List<int> coups = [5, 40, 12, 90, 3, 70, 25];
  for (final int d in coups) {
    await Future.delayed(Duration(milliseconds: 100));
    yield d;
  }
}

Future<void> main() async {
  final List<String> critiques = await degatsInfliges()
      .where((int d) => d >= 40)
      .map((int d) => 'COUP CRITIQUE : $d dégâts')
      .toList();

  for (final String c in critiques) {
    print(c);
  }

  final int nombreDeCoups = await degatsInfliges().length;
  print('Nombre total de coups : $nombreDeCoups');

  final int premierCoup = await degatsInfliges().first;
  print('Premier coup : $premierCoup');
}
```

**Résultat :**

```text
COUP CRITIQUE : 40 dégâts
COUP CRITIQUE : 90 dégâts
COUP CRITIQUE : 70 dégâts
Nombre total de coups : 7
Premier coup : 5
```

> **Remarque :** `length`, `toList()`, `first` et `last` retournent des `Future`, car il faut attendre le flux. Elles s'utilisent donc toujours avec `await`.

---

## 15.30 — Pourquoi Flutter repose entièrement sur l'asynchrone

Tout ce chapitre prépare la partie Flutter de la formation. Voici pourquoi.

Une application Flutter redessine son interface jusqu'à 60 fois par seconde. Chaque image dispose donc d'environ **16 millisecondes**. Si votre code bloque le thread principal plus longtemps, l'utilisateur voit une saccade. S'il le bloque une seconde, l'application semble morte.

```text
  ┌────────────────────────────────────────────────────────────┐
  │  Budget d'une image Flutter : 16 ms                        │
  ├────────────────────────────────────────────────────────────┤
  │  Un appel réseau bloquant de 500 ms                        │
  │      = 30 images perdues                                   │
  │      = l'interface se fige un demi-seconde                 │
  └────────────────────────────────────────────────────────────┘
```

Conséquence : dans Flutter, **tout ce qui est lent est asynchrone**. Les trois familles que vous rencontrerez :

**1. Les appels réseau.** Le package `http` ne retourne que des `Future`.

```dart
// Illustration : ce code nécessite le package http, il ne tourne pas dans DartPad.
// Future<Player> chargerJoueur(int id) async {
//   final reponse = await http.get(Uri.parse('https://api.jeu.com/joueurs/$id'));
//   if (reponse.statusCode != 200) {
//     throw Exception('Serveur indisponible (${reponse.statusCode})');
//   }
//   return Player.fromJson(jsonDecode(reponse.body));
// }
```

**2. `FutureBuilder` : afficher un widget qui dépend d'un `Future`.**

Un widget ne peut pas « attendre ». Il doit être capable de se dessiner **maintenant**, même si la donnée n'est pas encore arrivée. `FutureBuilder` sert exactement à cela : il redessine automatiquement l'écran quand le `Future` se termine.

```dart
// Illustration Flutter (nécessite le SDK Flutter) :
// FutureBuilder<String>(
//   future: chargerProfil(),
//   builder: (context, snapshot) {
//     if (snapshot.connectionState != ConnectionState.done) {
//       return CircularProgressIndicator();   // pendant le chargement
//     }
//     if (snapshot.hasError) {
//       return Text('Erreur : ${snapshot.error}');  // Future en erreur
//     }
//     return Text('Bienvenue, ${snapshot.data}');   // Future terminé
//   },
// )
```

Reconnaissez-vous les trois branches ? Ce sont **les trois états d'un `Future`** de la section 15.6 : non terminé, terminé avec une valeur, terminé avec une erreur. Vous connaissez déjà `FutureBuilder` sans l'avoir jamais utilisé.

**3. `StreamBuilder` : afficher un widget qui dépend d'un `Stream`.**

Même principe, mais le widget se redessine **à chaque nouvelle valeur** : points de vie du héros, chronomètre de la partie, messages du tchat, position des ennemis.

```dart
// Illustration Flutter :
// StreamBuilder<int>(
//   stream: pointsDeVieDuHeros(),
//   builder: (context, snapshot) {
//     final pv = snapshot.data ?? 100;
//     return Text('PV : $pv');
//   },
// )
```

Récapitulatif de la correspondance :

| Notion Dart de ce chapitre | Usage en Flutter |
| --- | --- |
| `Future<T>` | résultat d'un appel réseau, lecture de fichier, base de données |
| trois états d'un `Future` | les trois branches d'un `FutureBuilder` |
| `async` / `await` | corps des méthodes de chargement d'un écran |
| `try` / `catch` sur `await` | affichage d'un message d'erreur à l'utilisateur |
| `Future.wait()` | charger plusieurs ressources d'un écran en parallèle |
| `Stream<T>` | données qui évoluent : capteurs, tchat, état du jeu |
| `.listen()` et `.cancel()` | abonnement dans `initState`, annulation dans `dispose` |
| `StreamBuilder` | widget qui se redessine à chaque valeur |

> **À retenir :** vous n'apprenez pas l'asynchrone « en plus » de Flutter. Vous apprenez la **fondation** sur laquelle Flutter est construit.

---

## 15.31 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| La console affiche `Instance of 'Future<String>'` | Vous avez affiché le `Future` lui-même au lieu de sa valeur : il manque un `await`. | `print(await chargerProfil());` |
| `Error: 'await' can only be used in 'async' or 'async*' methods.` | Vous utilisez `await` dans une fonction non marquée `async`. | Ajoutez `async` et changez le type de retour en `Future<...>`. |
| `Error: Functions marked 'async' must have a return type assignable to 'Future'.` | Vous avez écrit `String f() async` au lieu de `Future<String> f() async`. | `Future<String> f() async { ... }` |
| `Error: A value of type 'Future<int>' can't be assigned to a variable of type 'int'.` | `await` manquant lors de l'affectation. | `final int score = await chargerScore();` |
| `Error: The getter 'length' isn't defined for the type 'Future<List<String>>'.` | Vous appelez une méthode sur le `Future` et non sur la valeur. | `(await chargerInventaire()).length` |
| Le `try` / `catch` ne rattrape rien, l'erreur reste `Unhandled exception` | Le bloc `try` s'est terminé avant que le `Future` n'échoue, car il n'y avait pas de `await`. | Mettez `await` **à l'intérieur** du `try` : `try { await f(); } catch (e) { ... }` |
| `Unhandled exception: Exception: ...` alors que vous avez un `.catchError()` | Le `.catchError()` est placé sur un autre future que celui qui échoue, ou l'échec vient d'un future oublié. | Chaînez `.catchError()` directement sur le future qui échoue, ou attendez-le dans un `try`. |
| `type 'Null' is not a subtype of type 'FutureOr<String>'` | Le gestionnaire de `.catchError()` sur un `Future<String>` ne retourne aucune valeur. | Retournez une valeur de repli du bon type : `return 'Invité';` |
| Le chargement prend 6 s au lieu de 3 s | Des `await` en série alors que les opérations sont indépendantes. | Lancez les futures d'abord, puis `await Future.wait([...])`. |
| Une boucle `for` avec `await` à l'intérieur est très lente | Chaque tour attend le précédent alors que rien ne le justifie. | `await Future.wait(liste.map((e) => traiter(e)));` |
| `Info: Missing an 'await' for the 'Future' computed by this expression.` | Un `Future` lancé sans être attendu (future oublié). | Ajoutez `await`, ou `unawaited(...)` avec un `.catchError()` si c'est volontaire. |
| Le programme se termine sans rien afficher | `main()` s'est terminé sans attendre les opérations lancées. | `Future<void> main() async { await ... }` |
| `Error: The type 'Stream<int>' used in the 'for' loop must implement 'Iterable'.` | Vous parcourez un `Stream` avec `for` au lieu de `await for`. | `await for (final n in flux) { ... }` |
| Le code placé après un `await for` ne s'exécute jamais | Le flux est infini : `await for` ne rend jamais la main. | Utilisez `.listen()`, ou `take(n)`, ou un `break`. |
| Le programme ne s'arrête jamais | Un `Stream` infini est encore écouté, ou un timer périodique tourne toujours. | Appelez `.cancel()` sur l'abonnement quand vous n'en avez plus besoin. |
| Un `Stream` créé avec `async*` n'affiche rien | Personne ne l'écoute : un `Stream` est paresseux. | Consommez-le avec `await for` ou `.listen()`. |
| Une exception dans un future oublié fait tomber le programme | Aucun gestionnaire n'est attaché à ce future. | `unawaited(f().catchError((e) { ... }));` |

---

## 15.32 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| Opération longue | Elle ne doit jamais bloquer le thread principal. |
| Asynchrone | « sans attendre bloqué », et non « en même temps ». |
| Mono-thread | Une seule ligne de votre code s'exécute à la fois. |
| Event loop | Deux files : microtâches (priorité haute) puis événements. |
| `Future<T>` | Promesse d'une valeur de type `T`, livrée plus tard. |
| Trois états | non terminé, terminé avec valeur, terminé avec erreur. |
| `Future<void>` | Une action dont on attend la fin, sans résultat. |
| `Future.delayed(d, f)` | Crée un `Future` qui se termine après la durée `d`. |
| `.then((v) { })` | Réagit au succès et retourne un nouveau `Future`. |
| `.catchError((e) { })` | Réagit à l'erreur ; doit retourner une valeur du bon type. |
| `.whenComplete(() { })` | S'exécute dans tous les cas (équivalent de `finally`). |
| `async` | Autorise `await` et fait retourner un `Future`. |
| `await` | Transforme `Future<T>` en `T` et suspend **la fonction seule**. |
| Corps d'une fonction `async` | Il démarre de façon **synchrone** jusqu'au premier `await`. |
| `Instance of 'Future<...>'` | Symptôme d'un `await` manquant. |
| `try` / `catch` + `await` | La façon normale de gérer les erreurs asynchrones. |
| `await` en série | À réserver aux étapes qui dépendent l'une de l'autre. |
| `Future.wait([...])` | Lance tout en parallèle ; durée = la plus longue, pas la somme. |
| `Future.value(x)` | `Future` déjà terminé avec une valeur (cache, test). |
| `Future.error(e)` | `Future` déjà terminé avec une erreur. |
| `unawaited(f())` | Future volontairement non attendu ; toujours avec `.catchError()`. |
| `Future<void> main() async` | Obligatoire dès qu'il y a un `await` au premier niveau. |
| `Stream<T>` | Plusieurs valeurs livrées au fil du temps. |
| `async*` + `yield` | Produit un `Stream`, valeur après valeur. |
| `await for` | Boucle sur un flux ; bloque la fonction jusqu'à la fin du flux. |
| `.listen()` | S'abonne sans bloquer ; renvoie un `StreamSubscription`. |
| `.cancel()` | Met fin à l'abonnement ; indispensable sur un flux infini. |
| `map` / `where` sur un `Stream` | Mêmes idées qu'au chapitre 14, appliquées à chaque valeur. |
| Flutter | `FutureBuilder` = trois états d'un `Future` ; `StreamBuilder` = flux. |

Les quatre phrases à retenir :

1. Dans un programme asynchrone, **l'ordre du code n'est pas l'ordre d'exécution**.
2. `await` ne bloque pas le programme : il suspend seulement la fonction courante.
3. Deux `await` à la suite ne se justifient que si le second dépend du premier ; sinon, `Future.wait()`.
4. Un `Future` peut échouer : un `Future` sans gestion d'erreur est un plantage en attente.

---

## 15.33 — Exercices

### Exercice 1 — Prédire l'ordre d'affichage (facile)

Sans exécuter le code, écrivez l'ordre exact des quatre lignes affichées, puis vérifiez.

```dart
void main() {
  print('A');
  Future.delayed(Duration(seconds: 2), () => print('B'));
  Future.delayed(Duration.zero, () => print('C'));
  print('D');
}
```

### Exercice 2 — Un premier `Future` avec `.then()` (facile)

Écrivez une fonction `Future<String> chargerPseudo()` qui retourne `'Alex'` après 2 secondes. Dans `main()`, affichez le pseudo avec `.then()`, et affichez `'Chargement en cours...'` avant, sans bloquer.

Sortie attendue :

```text
Chargement en cours...
Pseudo : Alex
```

### Exercice 3 — La même chose avec `async` / `await` (facile)

Réécrivez l'exercice 2 en utilisant `Future<void> main() async` et `await`. La sortie doit être identique.

### Exercice 4 — Corriger un `await` manquant (facile)

Ce programme affiche `Instance of 'Future<int>'`. Corrigez-le pour qu'il affiche le score, puis le score doublé.

```dart
Future<int> chargerScore() async {
  await Future.delayed(Duration(seconds: 1));
  return 4200;
}

void main() {
  final score = chargerScore();
  print('Score : $score');
  print('Score doublé : ${score * 2}');
}
```

### Exercice 5 — `try` / `catch` sur un `await` (moyen)

Écrivez `Future<String> chargerSauvegarde(String fichier)` qui, après 1 seconde :

- lève `FormatException('Nom de fichier vide')` si `fichier` est vide ;
- lève `Exception('Fichier introuvable')` si le fichier n'est pas `'partie1.sav'` ;
- retourne `'Alex — niveau 7'` sinon.

Dans `main()`, appelez-la trois fois (`'partie1.sav'`, `'partie9.sav'`, `''`) en attrapant chaque cas séparément.

### Exercice 6 — Écran de chargement avec `finally` (moyen)

Reprenez l'exercice 5 et ajoutez un `finally` qui affiche `-> écran de chargement masqué` après chaque tentative, réussie ou non.

### Exercice 7 — Trois étapes dépendantes (moyen)

Écrivez trois fonctions, chacune durant 1 seconde :

- `Future<String> connecter(String pseudo)` retourne `'TOKEN-<pseudo>'` ;
- `Future<int> chargerNiveau(String token)` retourne `7` ;
- `Future<String> chargerCarte(int niveau)` retourne `'Caverne de glace'`.

Enchaînez-les dans un `main()` asynchrone et affichez une phrase finale récapitulative.

### Exercice 8 — Série contre parallèle (moyen)

Écrivez trois fonctions `chargerA()` (2 s), `chargerB()` (3 s) et `chargerC()` (1 s), qui retournent chacune un texte.

Mesurez avec un `Stopwatch` la durée totale :

1. en les attendant l'une après l'autre ;
2. en les lançant toutes avant d'attendre.

Affichez les deux durées en secondes.

### Exercice 9 — Charger N textures en parallèle (moyen)

Écrivez `Future<String> chargerTexture(String nom)` qui retourne `'texture_<nom>.png'` après 500 ms.

Chargez en parallèle les textures `heros`, `gobelin`, `boss`, `potion`, `coffre` avec `Future.wait()` et `map`, puis affichez-les et la durée totale (elle doit être proche de 0,5 s).

### Exercice 10 — Compte à rebours en `Stream` (moyen)

Écrivez `Stream<int> compteARebours(int depart)` avec `async*` et `yield`, qui émet `depart`, `depart - 1`, ... jusqu'à `1`, une valeur par demi-seconde.

Consommez-le avec `await for` en affichant `3...`, `2...`, `1...`, puis `Partie lancée !`.

### Exercice 11 — Filtrer un flux de dégâts (difficile)

Écrivez `Stream<int> degats()` qui émet `[5, 40, 12, 90, 3, 70, 25]`, une valeur toutes les 100 ms.

1. Avec `where` et `map`, construisez la liste des coups critiques (`>= 40`) sous la forme `'CRITIQUE : 90'`, en utilisant `toList()`.
2. Affichez le nombre total de coups.
3. Réécoutez le flux avec `.listen()` et annulez l'abonnement après 350 ms.

### Exercice 12 — Mini-projet : chargement d'une partie (difficile)

Simulez le chargement complet d'une partie.

Trois ressources, indépendantes les unes des autres :

| Fonction | Durée | Résultat |
| --- | --- | --- |
| `chargerProfil()` | 2 s | `'Alex — niveau 7'` |
| `chargerInventaire()` | 3 s | `'Potion, Épée de feu, Clé rouillée'` |
| `chargerCarte()` | 1 s | `'Caverne de glace'` |

Écrivez un programme qui :

1. affiche `=== Chargement en série ===`, charge les trois ressources l'une après l'autre avec `await`, affiche chaque résultat et la durée totale ;
2. affiche `=== Chargement en parallèle ===`, recharge les trois ressources avec `Future.wait()`, affiche les résultats et la durée totale ;
3. affiche `=== Chargement avec panne ===`, relance le chargement parallèle avec un inventaire qui échoue (`Exception('Inventaire corrompu')`), attrape l'erreur et affiche un message clair ;
4. affiche dans tous les cas `-> écran de chargement masqué` grâce à un `finally`.

Chaque fonction doit accepter un paramètre `bool panne` permettant de forcer l'échec.

---

## 15.34 — Corrections des exercices

### Correction 1

```dart
void main() {
  print('A');
  Future.delayed(Duration(seconds: 2), () => print('B'));
  Future.delayed(Duration.zero, () => print('C'));
  print('D');
}
```

**Résultat :**

```text
A
D
C
B
```

**Explication :** le code synchrone passe toujours en premier, donc `A` puis `D` sortent immédiatement. Les deux `Future.delayed` n'exécutent rien tout de suite : ils déposent un rendez-vous dans la file des événements de l'event loop. `C` a une durée nulle, il est donc traité dès que `main()` a rendu la main. `B` attend deux secondes de plus. Retenez que l'ordre d'écriture (`B` avant `C`) n'a aucune influence : seule compte la date de fin de chaque `Future`.

---

### Correction 2

```dart
Future<String> chargerPseudo() {
  return Future.delayed(Duration(seconds: 2), () => 'Alex');
}

void main() {
  print('Chargement en cours...');
  chargerPseudo().then((String pseudo) {
    print('Pseudo : $pseudo');
  });
}
```

**Résultat :**

```text
Chargement en cours...
Pseudo : Alex
```

**Explication :** `chargerPseudo()` retourne immédiatement un `Future<String>` non terminé. `main()` continue donc sans attendre et affiche la première ligne. Deux secondes plus tard, le `Future` se termine avec la valeur `'Alex'` ; l'event loop exécute alors la fonction passée à `.then()`, qui reçoit cette valeur dans le paramètre `pseudo`. Le programme ne s'arrête pas à la fin de `main()` : il reste en vie tant qu'un timer est enregistré.

---

### Correction 3

```dart
Future<String> chargerPseudo() async {
  await Future.delayed(Duration(seconds: 2));
  return 'Alex';
}

Future<void> main() async {
  print('Chargement en cours...');
  final String pseudo = await chargerPseudo();
  print('Pseudo : $pseudo');
}
```

**Résultat :**

```text
Chargement en cours...
Pseudo : Alex
```

**Explication :** trois changements par rapport à la correction 2. La fonction est marquée `async`, ce qui permet d'écrire `await Future.delayed(...)` puis un `return 'Alex'` ordinaire : Dart emballe automatiquement la valeur dans un `Future<String>`. `main()` devient `Future<void> main() async`, sans quoi le `await` serait refusé à la compilation. Enfin, `await chargerPseudo()` transforme le `Future<String>` en `String`. Le comportement est identique, mais le code se lit de haut en bas.

---

### Correction 4

```dart
Future<int> chargerScore() async {
  await Future.delayed(Duration(seconds: 1));
  return 4200;
}

Future<void> main() async {
  final int score = await chargerScore();
  print('Score : $score');
  print('Score doublé : ${score * 2}');
}
```

**Résultat :**

```text
Score : 4200
Score doublé : 8400
```

**Explication :** le programme d'origine rangeait le `Future<int>` lui-même dans `score`. `print` affichait alors le ticket (`Instance of 'Future<int>'`), et `score * 2` ne compilait même pas, car la multiplication n'existe pas sur un `Future<int>`. Il fallait deux corrections liées : `await` devant l'appel, et `async` sur `main()` (avec le type de retour `Future<void>`) pour que ce `await` soit autorisé. Le type explicite `final int score` est une bonne habitude : il fait apparaître l'erreur dès l'écriture si vous oubliez le `await`.

---

### Correction 5

```dart
Future<String> chargerSauvegarde(String fichier) async {
  await Future.delayed(Duration(seconds: 1));
  if (fichier.isEmpty) {
    throw FormatException('Nom de fichier vide');
  }
  if (fichier != 'partie1.sav') {
    throw Exception('Fichier introuvable');
  }
  return 'Alex — niveau 7';
}

Future<void> essayer(String fichier) async {
  try {
    final String donnees = await chargerSauvegarde(fichier);
    print('Chargé : $donnees');
  } on FormatException catch (e) {
    print('Sauvegarde illisible : ${e.message}');
  } catch (e) {
    print('Erreur : $e');
  }
}

Future<void> main() async {
  await essayer('partie1.sav');
  await essayer('partie9.sav');
  await essayer('');
}
```

**Résultat :**

```text
Chargé : Alex — niveau 7
Erreur : Exception: Fichier introuvable
Sauvegarde illisible : Nom de fichier vide
```

**Explication :** le point clé est que `await` se trouve **à l'intérieur** du bloc `try`. La fonction est suspendue dans le `try` ; quand le `Future` se termine avec une erreur, celle-ci est relancée exactement à l'endroit du `await`, donc dans le bloc protégé. Sans `await`, le `try` se terminerait avant l'échec et n'attraperait rien. L'ordre des clauses compte aussi : `on FormatException` doit précéder le `catch` général, sinon le cas particulier ne serait jamais atteint. Enfin, les trois `await essayer(...)` dans `main()` garantissent que les tentatives s'affichent dans l'ordre.

---

### Correction 6

```dart
Future<String> chargerSauvegarde(String fichier) async {
  await Future.delayed(Duration(seconds: 1));
  if (fichier.isEmpty) {
    throw FormatException('Nom de fichier vide');
  }
  if (fichier != 'partie1.sav') {
    throw Exception('Fichier introuvable');
  }
  return 'Alex — niveau 7';
}

Future<void> essayer(String fichier) async {
  print('Écran de chargement affiché pour "$fichier"');
  try {
    final String donnees = await chargerSauvegarde(fichier);
    print('Chargé : $donnees');
  } on FormatException catch (e) {
    print('Sauvegarde illisible : ${e.message}');
  } catch (e) {
    print('Erreur : $e');
  } finally {
    print('-> écran de chargement masqué');
  }
}

Future<void> main() async {
  await essayer('partie1.sav');
  await essayer('partie9.sav');
  await essayer('');
}
```

**Résultat :**

```text
Écran de chargement affiché pour "partie1.sav"
Chargé : Alex — niveau 7
-> écran de chargement masqué
Écran de chargement affiché pour "partie9.sav"
Erreur : Exception: Fichier introuvable
-> écran de chargement masqué
Écran de chargement affiché pour ""
Sauvegarde illisible : Nom de fichier vide
-> écran de chargement masqué
```

**Explication :** `finally` s'exécute dans les trois cas, succès comme échec. C'est exactement le rôle du masquage d'un écran de chargement : quoi qu'il arrive, l'interface ne doit pas rester bloquée sur un indicateur qui tourne indéfiniment. Dans le style `.then()`, l'équivalent aurait été `.whenComplete(() => print('-> écran de chargement masqué'))`. Notez que `finally` s'exécute aussi si le `catch` relance l'exception avec `rethrow`.

---

### Correction 7

```dart
Future<String> connecter(String pseudo) async {
  await Future.delayed(Duration(seconds: 1));
  return 'TOKEN-$pseudo';
}

Future<int> chargerNiveau(String token) async {
  await Future.delayed(Duration(seconds: 1));
  return 7;
}

Future<String> chargerCarte(int niveau) async {
  await Future.delayed(Duration(seconds: 1));
  return 'Caverne de glace';
}

Future<void> main() async {
  print('1. Connexion...');
  final String token = await connecter('Alex');
  print('   token obtenu : $token');

  print('2. Chargement du niveau...');
  final int niveau = await chargerNiveau(token);
  print('   niveau : $niveau');

  print('3. Chargement de la carte...');
  final String carte = await chargerCarte(niveau);

  print('4. Alex est prêt : niveau $niveau, carte "$carte".');
}
```

**Résultat (une étape par seconde, environ 3 s au total) :**

```text
1. Connexion...
   token obtenu : TOKEN-Alex
2. Chargement du niveau...
   niveau : 7
3. Chargement de la carte...
4. Alex est prêt : niveau 7, carte "Caverne de glace".
```

**Explication :** ici la série est **obligatoire**, car chaque étape consomme le résultat de la précédente : `chargerNiveau` a besoin du token, `chargerCarte` a besoin du numéro de niveau. Il serait impossible de paralléliser avec `Future.wait()`. C'est le cas où trois `await` à la suite sont le bon choix, et le code se lit alors comme une procédure synchrone ordinaire.

---

### Correction 8

```dart
Future<String> chargerA() async {
  await Future.delayed(Duration(seconds: 2));
  return 'A chargé';
}

Future<String> chargerB() async {
  await Future.delayed(Duration(seconds: 3));
  return 'B chargé';
}

Future<String> chargerC() async {
  await Future.delayed(Duration(seconds: 1));
  return 'C chargé';
}

Future<void> main() async {
  // 1) En série
  final Stopwatch chrono1 = Stopwatch()..start();
  final String a1 = await chargerA();
  final String b1 = await chargerB();
  final String c1 = await chargerC();
  chrono1.stop();
  print('Série     : $a1, $b1, $c1 -> ${chrono1.elapsed.inSeconds} s');

  // 2) En parallèle
  final Stopwatch chrono2 = Stopwatch()..start();
  final Future<String> fa = chargerA();
  final Future<String> fb = chargerB();
  final Future<String> fc = chargerC();
  final String a2 = await fa;
  final String b2 = await fb;
  final String c2 = await fc;
  chrono2.stop();
  print('Parallèle : $a2, $b2, $c2 -> ${chrono2.elapsed.inSeconds} s');
}
```

**Résultat :**

```text
Série     : A chargé, B chargé, C chargé -> 6 s
Parallèle : A chargé, B chargé, C chargé -> 3 s
```

**Explication :** dans le premier bloc, chaque `await` suspend `main()` avant même que l'appel suivant ne soit écrit : les trois attentes s'additionnent (2 + 3 + 1 = 6 s). Dans le second bloc, les trois appels sont exécutés d'affilée, sans `await` : les trois chronomètres internes démarrent donc au même instant. Les `await` qui suivent ne font que récupérer des résultats déjà en cours de production. La durée totale devient celle de la plus longue opération (3 s). Le déplacement du mot `await` de trois lignes suffit à diviser le temps de chargement par deux.

---

### Correction 9

```dart
Future<String> chargerTexture(String nom) async {
  await Future.delayed(Duration(milliseconds: 500));
  return 'texture_$nom.png';
}

Future<void> main() async {
  final List<String> aCharger = ['heros', 'gobelin', 'boss', 'potion', 'coffre'];

  final Stopwatch chrono = Stopwatch()..start();

  final List<String> textures = await Future.wait(
    aCharger.map((String nom) => chargerTexture(nom)),
  );

  chrono.stop();

  for (final String t in textures) {
    print('Chargée : $t');
  }
  print('Durée totale : ${chrono.elapsed.inMilliseconds} ms');
}
```

**Résultat (les millisecondes varient légèrement) :**

```text
Chargée : texture_heros.png
Chargée : texture_gobelin.png
Chargée : texture_boss.png
Chargée : texture_potion.png
Chargée : texture_coffre.png
Durée totale : 505 ms
```

**Explication :** `aCharger.map(...)` du chapitre 14 produit ici un `Iterable<Future<String>>` : cinq futures créés d'un coup, donc cinq chargements démarrés simultanément. `Future.wait()` les rassemble en un seul `Future<List<String>>` qui se termine quand les cinq sont finis. La durée totale reste d'environ 500 ms, et non 2 500 ms. Point important : l'ordre de la liste résultat suit l'ordre de la liste d'entrée, pas l'ordre d'arrivée réel. Une boucle `for` avec `await` à l'intérieur aurait donné le même affichage, mais cinq fois plus lentement.

---

### Correction 10

```dart
Stream<int> compteARebours(int depart) async* {
  for (int i = depart; i >= 1; i--) {
    await Future.delayed(Duration(milliseconds: 500));
    yield i;
  }
}

Future<void> main() async {
  print('Préparez-vous...');
  await for (final int n in compteARebours(3)) {
    print('$n...');
  }
  print('Partie lancée !');
}
```

**Résultat (une ligne par demi-seconde) :**

```text
Préparez-vous...
3...
2...
1...
Partie lancée !
```

**Explication :** `async*` transforme la fonction en génératrice asynchrone : elle retourne un `Stream<int>` au lieu d'un `Future<int>`. Chaque `yield` émet une valeur dans le flux **sans arrêter la fonction**, contrairement à `return`. Côté consommation, `await for` attend chaque valeur, exécute le corps de la boucle, puis attend la suivante. La ligne `Partie lancée !` ne s'affiche qu'après la fermeture du flux, c'est-à-dire à la sortie de la boucle `for` interne. Notez que rien ne s'exécute tant que personne n'écoute : un `Stream` produit par `async*` est paresseux.

---

### Correction 11

```dart
import 'dart:async';

Stream<int> degats() async* {
  final List<int> coups = [5, 40, 12, 90, 3, 70, 25];
  for (final int d in coups) {
    await Future.delayed(Duration(milliseconds: 100));
    yield d;
  }
}

Future<void> main() async {
  // 1) Coups critiques
  final List<String> critiques = await degats()
      .where((int d) => d >= 40)
      .map((int d) => 'CRITIQUE : $d')
      .toList();
  for (final String c in critiques) {
    print(c);
  }

  // 2) Nombre total de coups
  final int total = await degats().length;
  print('Nombre total de coups : $total');

  // 3) Écoute puis annulation
  print('--- Écoute partielle ---');
  final StreamSubscription<int> abonnement =
      degats().listen((int d) => print('reçu : $d'));

  await Future.delayed(Duration(milliseconds: 350));
  await abonnement.cancel();
  print('Abonnement annulé.');
}
```

**Résultat :**

```text
CRITIQUE : 40
CRITIQUE : 90
CRITIQUE : 70
Nombre total de coups : 7
--- Écoute partielle ---
reçu : 5
reçu : 40
reçu : 12
Abonnement annulé.
```

**Explication :** trois idées se combinent ici. D'abord, `where` et `map` sur un `Stream` fonctionnent comme au chapitre 14, mais s'appliquent à chaque valeur au moment où elle arrive ; `toList()` retourne un `Future<List<String>>`, d'où le `await`. Ensuite, `length` est également un `Future`, car il faut attendre la fermeture du flux pour compter. Enfin, `listen()` ne bloque pas : `main()` continue immédiatement vers le `await Future.delayed(...)`. Pendant ces 350 ms, trois valeurs sont émises (à 100, 200 et 300 ms). L'appel à `cancel()` interrompt la production : les quatre coups restants ne sont jamais générés. Sans ce `cancel()`, l'abonnement resterait actif jusqu'à la fin du flux.

---

### Correction 12

```dart
Future<String> chargerProfil({bool panne = false}) async {
  await Future.delayed(Duration(seconds: 2));
  if (panne) {
    throw Exception('Profil corrompu');
  }
  return 'Alex — niveau 7';
}

Future<String> chargerInventaire({bool panne = false}) async {
  await Future.delayed(Duration(seconds: 3));
  if (panne) {
    throw Exception('Inventaire corrompu');
  }
  return 'Potion, Épée de feu, Clé rouillée';
}

Future<String> chargerCarte({bool panne = false}) async {
  await Future.delayed(Duration(seconds: 1));
  if (panne) {
    throw Exception('Carte introuvable');
  }
  return 'Caverne de glace';
}

Future<void> chargerEnSerie() async {
  print('=== Chargement en série ===');
  final Stopwatch chrono = Stopwatch()..start();
  try {
    final String profil = await chargerProfil();
    final String sac = await chargerInventaire();
    final String carte = await chargerCarte();

    print('Profil     : $profil');
    print('Inventaire : $sac');
    print('Carte      : $carte');
  } catch (e) {
    print('Chargement interrompu : $e');
  } finally {
    chrono.stop();
    print('Durée      : ${chrono.elapsed.inSeconds} s');
    print('-> écran de chargement masqué');
  }
}

Future<void> chargerEnParallele({bool panneInventaire = false}) async {
  final Stopwatch chrono = Stopwatch()..start();
  try {
    final List<String> r = await Future.wait([
      chargerProfil(),
      chargerInventaire(panne: panneInventaire),
      chargerCarte(),
    ]);

    print('Profil     : ${r[0]}');
    print('Inventaire : ${r[1]}');
    print('Carte      : ${r[2]}');
  } catch (e) {
    print('Chargement interrompu : $e');
    print('La partie ne peut pas démarrer. Réessayez.');
  } finally {
    chrono.stop();
    print('Durée      : ${chrono.elapsed.inSeconds} s');
    print('-> écran de chargement masqué');
  }
}

Future<void> main() async {
  await chargerEnSerie();

  print('');
  print('=== Chargement en parallèle ===');
  await chargerEnParallele();

  print('');
  print('=== Chargement avec panne ===');
  await chargerEnParallele(panneInventaire: true);
}
```

**Résultat :**

```text
=== Chargement en série ===
Profil     : Alex — niveau 7
Inventaire : Potion, Épée de feu, Clé rouillée
Carte      : Caverne de glace
Durée      : 6 s
-> écran de chargement masqué

=== Chargement en parallèle ===
Profil     : Alex — niveau 7
Inventaire : Potion, Épée de feu, Clé rouillée
Carte      : Caverne de glace
Durée      : 3 s
-> écran de chargement masqué

=== Chargement avec panne ===
Chargement interrompu : Exception: Inventaire corrompu
La partie ne peut pas démarrer. Réessayez.
Durée      : 3 s
-> écran de chargement masqué
```

**Explication :** ce mini-projet réunit tout le chapitre.

- Les trois fonctions de chargement sont des `Future<String>` simulés par `Future.delayed`, avec un paramètre nommé `panne` qui permet de forcer l'échec sans modifier le code appelant.
- `chargerEnSerie()` enchaîne trois `await`. Comme les trois ressources sont **indépendantes**, cet enchaînement n'est justifié par rien : les durées s'additionnent (2 + 3 + 1 = 6 s). C'est le piège de la section 15.19.
- `chargerEnParallele()` passe les trois appels à `Future.wait()`. Les trois futures sont créés au moment de la construction de la liste, donc les trois opérations démarrent ensemble. La durée totale tombe à 3 s, celle de la plus longue.
- Dans le troisième cas, l'inventaire lève une exception au bout de 3 s. `Future.wait()` propage cette erreur, et le `await` la relance à l'intérieur du `try`, où le `catch` la récupère. Aucun résultat partiel n'est utilisable : c'est le comportement voulu ici, puisqu'une partie sans inventaire n'a pas de sens.
- Le bloc `finally` s'exécute dans les trois scénarios. Il joue le rôle du `.whenComplete()` : masquer l'indicateur de chargement, quoi qu'il advienne. C'est exactement ce que vous ferez dans un écran Flutter avec un `FutureBuilder`.

Pour aller plus loin, vous pouvez remplacer `Future.wait` par une version tolérante aux pannes : chaque chargement porte alors son propre `.catchError()` qui retourne une valeur de repli (`'inventaire vide'`), et la partie démarre malgré l'incident.

---

## Et maintenant ?

Vous savez maintenant écrire du code qui attend sans bloquer. Vous savez qu'un `Future` est une promesse de valeur, qu'il possède trois états, et que `async` / `await` est la façon lisible de le manipuler. Vous savez surtout prédire **l'ordre réel d'exécution** d'un programme asynchrone, repérer des `await` inutilement séquentiels et les remplacer par un `Future.wait()`, et attraper une erreur asynchrone avec un `try` / `catch` ordinaire. Avec les `Stream`, vous savez enfin traiter des valeurs qui arrivent au fil du temps, et vous avez vu à quoi correspondront `FutureBuilder` et `StreamBuilder` en Flutter.

Vos programmes commencent cependant à être longs. Tout tient encore dans un seul fichier, avec des fonctions et des classes empilées les unes sur les autres. Un vrai projet ne s'écrit pas ainsi : il se découpe en fichiers, en dossiers et en bibliothèques, il déclare ses dépendances, et il s'installe avec une simple commande.

Le chapitre suivant quitte donc DartPad pour la ligne de commande : structure d'un projet Dart, fichier `pubspec.yaml`, dossiers `bin/`, `lib/` et `test/`, mots-clés `import`, `export` et `part`, installation de paquets depuis pub.dev.

Rendez-vous au chapitre 16 : [16-PARTIE-1A—ORGANISATION-DUN-PROJET-DART.md](16-PARTIE-1A—ORGANISATION-DUN-PROJET-DART.md)
