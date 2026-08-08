# PARTIE 1A — DART
# CHAPITRE 17 — JSON ET MODÉLISATION DE DONNÉES

> **Niveau :** intermédiaire
> **Durée estimée :** 6 h
> **Pré-requis :** chapitre 16 — Organisation d'un projet Dart (et les chapitres 9, 12, 13, 14)
> **Ce que vous saurez faire à la fin :** transformer du texte JSON en objets Dart typés et vos objets Dart en texte JSON, en gérant proprement les valeurs manquantes, les types incohérents et les erreurs de format.

---

## 17.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer à quoi sert JSON et pourquoi tout le monde l'utilise ;
- reconnaître les six types de valeurs JSON ;
- lire et écrire un objet JSON et un tableau JSON ;
- citer les différences entre JSON et une `Map` Dart ;
- respecter les règles de syntaxe JSON (guillemets doubles, pas de virgule finale) ;
- décoder du texte avec `jsonDecode()` et encoder avec `jsonEncode()` ;
- expliquer pourquoi `jsonDecode()` renvoie `dynamic` ;
- caster un résultat en `Map<String, dynamic>` et en `List<dynamic>` ;
- écrire une classe modèle avec `fromJson()` et `toJson()` ;
- faire un aller-retour objet → JSON → objet sans perte ;
- gérer les objets imbriqués et les listes d'objets imbriqués ;
- écrire du code défensif face aux valeurs manquantes et aux types incohérents ;
- rattraper une `FormatException` sur un JSON invalide ;
- convertir des `enum` et des `DateTime` vers JSON et retour ;
- sauvegarder et recharger une partie de jeu dans un fichier JSON ;
- comprendre à quoi sert `json_serializable` ;
- faire le lien avec les API REST que vous consommerez en Flutter.

---

## 17.1 — Pourquoi JSON ?

Jusqu'ici, toutes vos données vivaient **dans la mémoire du programme**. Vous créiez un joueur, vous le modifiiez, puis le programme se terminait et tout disparaissait.

Le joueur lance le jeu, gagne 3000 points, quitte, relance : son score est revenu à zéro. Il a tout perdu.

Il vous faut donc un moyen de **sortir vos objets du programme** : un fichier de sauvegarde, une base de données, un serveur distant.

Or un fichier ne sait pas stocker un objet Dart. Un fichier ne stocke que du texte. Un réseau non plus : il ne transporte que des octets. Il faut donc une **représentation textuelle** de vos objets.

> **Sérialisation** : transformer un objet en texte.
> **Désérialisation** : transformer ce texte en objet.

JSON est le format texte le plus utilisé au monde pour cela.

```text
  Objet Dart                Texte JSON                 Objet Dart
  ┌───────────┐  encode    ┌──────────────────┐ decode ┌───────────┐
  │ Player    │ ────────>  │ {"nom":"Alex",   │ ─────> │ Player    │
  │ nom=Alex  │            │  "score":3000}   │        │ nom=Alex  │
  │ score=3000│            └──────────────────┘        │ score=3000│
  └───────────┘                     │                  └───────────┘
                                    ▼
                          fichier / réseau / base
```

| Avantage de JSON | Explication |
| --- | --- |
| Lisible par un humain | Vous ouvrez le fichier et vous comprenez son contenu |
| Universel | Dart, Python, Java, JavaScript, C#, Go... savent le lire |
| Léger | Peu de caractères parasites, contrairement à XML |
| Standard | Il n'y a qu'une seule façon de l'écrire correctement |
| Natif en Dart | La bibliothèque `dart:convert` est fournie avec le langage |

Et surtout : la quasi-totalité des API que vous consommerez en Flutter répondent en JSON. Ce chapitre n'est pas un détour, c'est la porte d'entrée de la suite de la formation.

---

## 17.2 — Qu'est-ce que JSON ?

JSON signifie **JavaScript Object Notation**. Malgré son nom, JSON n'a plus rien à voir avec JavaScript : c'est un **format de texte**, indépendant de tout langage.

```json
{
  "nom": "Alex",
  "niveau": 7,
  "energie": 87.5,
  "estVivant": true,
  "arme": null,
  "inventaire": ["potion", "clé", "torche"]
}
```

Ce n'est pas du code Dart. Ce n'est pas du code du tout. C'est une suite de caractères que vous pourriez taper dans un bloc-notes.

```text
  {                       <- une ACCOLADE ouvre un OBJET
    "nom": "Alex",        <- "nom" est une CLÉ, "Alex" est une VALEUR
    "inventaire": [ ... ] <- des CROCHETS ouvrent un TABLEAU
  }
```

| Terme JSON | Équivalent mental en Dart |
| --- | --- |
| objet `{...}` | une `Map<String, dynamic>` |
| tableau `[...]` | une `List<dynamic>` |
| clé | la clé d'une `Map`, toujours du texte |
| valeur | ce que l'on range derrière la clé |

> Un document JSON n'est **jamais** un objet Dart. C'est **toujours** une chaîne de caractères. Tant qu'on ne l'a pas décodé, on ne peut rien en faire.

---

## 17.3 — Les types JSON

JSON ne connaît que **six** types de valeurs. Six, pas un de plus.

| Type JSON | Exemple | Type Dart après décodage |
| --- | --- | --- |
| string | `"Alex"` | `String` |
| number | `7` ou `87.5` | `int` ou `double` |
| boolean | `true` / `false` | `bool` |
| null | `null` | `Null` |
| object | `{"nom": "Alex"}` | `Map<String, dynamic>` |
| array | `[1, 2, 3]` | `List<dynamic>` |

L'exemple de la section 17.2 contient déjà cinq de ces six types ; il ne lui manque qu'un objet imbriqué, par exemple `"position": { "x": 12, "y": 40 }`.

Trois points d'attention.

**1. Il n'existe pas de type date.** Une date en JSON est un texte (`"2026-08-08"`) ou un nombre. Voir 17.30.

**2. JSON ne distingue pas `int` et `double`** : il n'a qu'un type « number ». C'est Dart qui décidera, à la lecture, si `7` devient un `int` et `87.5` un `double`.

**3. `null` est une valeur JSON à part entière.** Écrire `"arme": null` n'est pas la même chose qu'omettre la clé `arme`. Cette nuance revient en 17.26.

---

## 17.4 — Un objet JSON

Un **objet JSON** est une collection de paires `"clé": valeur`, entre accolades.

```json
{
  "nom": "Gobelin",
  "pointsDeVie": 30,
  "estBoss": false
}
```

```text
  {  "nom"  :  "Gobelin"  ,  "pointsDeVie"  :  30  }
  │    │     │     │       │                        │
  │    │     │     │       └─ virgule entre paires  │
  │    │     │     └─ la valeur                     │
  │    │     └─ deux-points obligatoire             │
  │    └─ la clé, TOUJOURS entre guillemets doubles │
  └─ accolade ouvrante            accolade fermante ┘
```

- une clé est toujours une chaîne entre guillemets doubles ;
- une clé ne doit apparaître qu'une seule fois ;
- l'ordre des clés n'a aucune importance ;
- un objet peut être vide : `{}` est un JSON valide.

Un objet peut contenir un autre objet : on dit alors qu'il est **imbriqué**, comme `{"nom":"Alex","position":{"x":12,"y":40}}`. Il n'y a pas de limite de profondeur, mais au-delà de trois ou quatre niveaux un JSON devient pénible à manipuler.

---

## 17.5 — Un tableau JSON

Un **tableau JSON** est une suite ordonnée de valeurs, entre crochets.

```json
["potion", "clé", "torche"]
```

Contrairement à l'objet, ici **l'ordre compte** : `["a", "b"]` et `["b", "a"]` sont deux tableaux différents.

Un tableau peut contenir des objets. C'est la forme la plus fréquente dans le monde réel :

```json
[
  { "nom": "Gobelin", "pv": 30 },
  { "nom": "Orc",     "pv": 55 },
  { "nom": "Dragon",  "pv": 400 }
]
```

Un tableau peut être vide (`[]`) ou mélanger des types (`[1, "deux", true]`). Le mélange est autorisé par la norme mais **fortement déconseillé** : côté Dart vous obtiendrez une `List<dynamic>` impossible à typer.

Enfin, un document JSON peut avoir un tableau comme racine : `["potion", "clé"]` est un JSON valide à lui seul.

---

## 17.6 — JSON vs Map Dart : les différences

Ces deux écritures se ressemblent à l'écran mais n'ont rien à voir.

```dart
// Une Map Dart : du CODE, en mémoire, avec des types.
Map<String, dynamic> joueur = {'nom': 'Alex', 'niveau': 7};
```

```json
{ "nom": "Alex", "niveau": 7 }
```

| Critère | `Map` Dart | JSON |
| --- | --- | --- |
| Nature | objet en mémoire | texte |
| Clés | n'importe quel type | uniquement des `String` |
| Guillemets | simples `'` ou doubles `"` | doubles `"` uniquement |
| Valeurs | n'importe quel objet Dart | 6 types seulement |
| Virgule finale | autorisée | **interdite** |
| Commentaires | autorisés (`//`) | **interdits** |
| Méthodes | `.length`, `.keys`, `[]`... | aucune |

```dart
void main() {
  Map<String, dynamic> joueur = {'nom': 'Alex', 'niveau': 7};
  String texteJson = '{"nom": "Alex", "niveau": 7}';

  print(joueur.runtimeType);
  print(texteJson.runtimeType);
  print(texteJson.length);
}
```

**Résultat :**

```text
_Map<String, dynamic>
String
28
```

`texteJson.length` vaut 28 : c'est le nombre de **caractères**. Dart ne voit là qu'une chaîne, pas une structure.

> On ne « lit » pas un JSON directement. On le **décode** en `Map` (ou en `List`), puis on lit la `Map`.

---

## 17.7 — Règles de syntaxe

Une seule erreur de syntaxe et **tout** le document est rejeté.

**Règle 1 — Guillemets doubles obligatoires.** `{ 'nom': 'Alex' }` est invalide.

**Règle 2 — Pas de virgule après le dernier élément.** `{ "nom": "Alex", }` est invalide. C'est l'erreur numéro un, parce que Dart, lui, **autorise** la virgule finale dans une `Map` : l'habitude prise en Dart se retourne contre vous.

**Règle 3 — Les clés sont toujours des chaînes.** `{ 1: "Alex" }` est invalide.

**Règle 4 — Pas de commentaires.** Aucun `//`, aucun `/* */`.

**Règle 5 — `true`, `false` et `null` en minuscules, sans guillemets.** `"true"` est un texte ; `true` est un booléen.

**Règle 6 — Les nombres s'écrivent sans guillemets.** `07` est invalide, `7` et `7.5` sont valides. `"7"` est valide mais c'est un **texte** : c'est la source du problème traité en 17.27.

| Écriture | Valide ? | Pourquoi |
| --- | --- | --- |
| `{"a": 1}` | oui | — |
| `{'a': 1}` | non | guillemets simples |
| `{a: 1}` | non | clé sans guillemets |
| `{"a": 1,}` | non | virgule finale |
| `{"a": True}` | non | majuscule |
| `{"a": undefined}` | non | `undefined` n'existe pas en JSON |
| `[]` et `{}` | oui | tableau vide, objet vide |

---

## 17.8 — La bibliothèque `dart:convert`

Tout ce dont vous avez besoin est **déjà installé** avec Dart. Aucun package à ajouter dans `pubspec.yaml`. Il suffit d'écrire, en haut du fichier :

```dart
import 'dart:convert';
```

Pour ce chapitre, deux fonctions suffisent :

```text
  ┌────────────────────────────────────────────────────────────┐
  │  jsonDecode(String texte)  ->  dynamic  (Map ou List...)   │
  │      « texte JSON »  devient  « structure Dart »           │
  ├────────────────────────────────────────────────────────────┤
  │  jsonEncode(Object? valeur)  ->  String                    │
  │      « structure Dart »  devient  « texte JSON »           │
  └────────────────────────────────────────────────────────────┘
```

> **Erreur classique :** oublier l'import. Le message est alors `Error: Undefined name 'jsonDecode'`.

`jsonDecode` et `jsonEncode` sont des raccourcis : `json.decode(...)` et `json.encode(...)` sont strictement équivalents.

---

## 17.9 — `jsonDecode()`

`jsonDecode()` prend un `String` contenant du JSON et renvoie la structure Dart correspondante.

```dart
import 'dart:convert';

void main() {
  String texte = '{"nom": "Alex", "niveau": 7}';

  var donnees = jsonDecode(texte);

  print(donnees);
  print(donnees.runtimeType);
  print(donnees['nom']);
}
```

**Résultat :**

```text
{nom: Alex, niveau: 7}
_Map<String, dynamic>
Alex
```

Sur la première ligne de sortie, les guillemets ont disparu : ce n'est plus du JSON, c'est l'affichage d'une `Map` Dart.

Point de syntaxe. Le JSON contient des guillemets doubles ; entourez-le donc de guillemets **simples** en Dart, et utilisez une chaîne multiligne `'''` pour un JSON long :

```dart
  String texte = '{"nom": "Alex"}';     // simple et lisible
  String autre = "{\"nom\": \"Alex\"}";  // valide mais illisible

  String long = '''
  {
    "nom": "Alex",
    "inventaire": ["potion", "clé"]
  }
  ''';
```

Les espaces et retours à la ligne autour du JSON sont ignorés par `jsonDecode`.

---

## 17.10 — Le type de retour `dynamic` et pourquoi

La signature réelle est :

```dart
dynamic jsonDecode(String source, {Object? Function(Object?, Object?)? reviver});
```

Pourquoi `dynamic` et pas `Map<String, dynamic>` ? Parce que Dart **ne peut pas savoir à l'avance** ce que contient le texte.

```dart
import 'dart:convert';

void main() {
  print(jsonDecode('{"a": 1}').runtimeType);
  print(jsonDecode('[1, 2, 3]').runtimeType);
  print(jsonDecode('42').runtimeType);
  print(jsonDecode('"bonjour"').runtimeType);
}
```

**Résultat :**

```text
_Map<String, dynamic>
List<dynamic>
int
String
```

Quatre appels, quatre types différents. La seule signature possible est donc `dynamic` : « ce sera décidé à l'exécution ».

Conséquence désagréable : sur une valeur `dynamic`, le compilateur **ne vous protège plus**. Le code `jsonDecode('{"nom":"Alex"}').nomDuJoueur` compile sans la moindre alerte, et n'échoue qu'à l'exécution :

```text
Unhandled exception:
NoSuchMethodError: Class '_Map<String, dynamic>' has no instance getter 'nomDuJoueur'.
```

C'est exactement le problème du chapitre 12 : une erreur repoussée à l'exécution, chez l'utilisateur.

> **Règle :** ne laissez jamais une valeur `dynamic` circuler. Convertissez-la en type précis dès la ligne qui suit `jsonDecode`.

---

## 17.11 — Caster en `Map<String, dynamic>`

« Caster » signifie annoncer au compilateur le type réel d'une valeur. On utilise le mot-clé `as`.

```dart
import 'dart:convert';

void main() {
  String texte = '{"nom": "Alex", "niveau": 7}';

  Map<String, dynamic> joueur = jsonDecode(texte) as Map<String, dynamic>;

  print(joueur['nom']);
  print(joueur.keys);
  print(joueur.length);
}
```

**Résultat :**

```text
Alex
(nom, niveau)
2
```

Trois bénéfices immédiats : l'auto-complétion revient, une faute de frappe sur une méthode devient une erreur de compilation, et le lecteur du code sait à quoi il a affaire.

Pourquoi `Map<String, dynamic>` et pas `Map<String, String>` ? Parce que les valeurs sont hétérogènes : `"Alex"` est un texte, `7` un nombre. Seul `dynamic` couvre les deux.

Attention, `as` n'est pas magique. Si le contenu ne correspond pas, l'erreur tombe à l'exécution. Ainsi `jsonDecode('[1, 2, 3]') as Map<String, dynamic>` donne :

```text
Unhandled exception:
type 'List<dynamic>' is not a subtype of type 'Map<String, dynamic>' in type cast
```

> `as X` se lit « je certifie que cette valeur est un `X` ». Si vous vous trompez, c'est vous qui payez, à l'exécution.

---

## 17.12 — Lire une valeur, gérer l'absence

Rappel du chapitre 12 : lire une clé absente d'une `Map` ne plante pas, cela renvoie `null`. Sur `{"nom": "Alex"}`, `joueur['niveau']` vaut `null` et `joueur.containsKey('niveau')` vaut `false`.

C'est une bonne nouvelle (pas de crash) et une mauvaise (un `null` circule sans prévenir). Si vous écrivez `int niveau = joueur['niveau'];`, le programme plante avec `type 'Null' is not a subtype of type 'int'`.

La parade est l'opérateur `??` :

```dart
import 'dart:convert';

void main() {
  final joueur = jsonDecode('{"nom": "Alex"}') as Map<String, dynamic>;

  String nom = joueur['nom'] as String? ?? 'Inconnu';
  int niveau = joueur['niveau'] as int? ?? 1;
  bool vivant = joueur['estVivant'] as bool? ?? true;

  print('$nom, niveau $niveau, vivant : $vivant');
}
```

**Résultat :**

```text
Alex, niveau 1, vivant : true
```

Décomposons `joueur['niveau'] as int? ?? 1` :

```text
  joueur['niveau']   ->  dynamic  (ici : null)
  as int?            ->  int?     (on autorise explicitement null)
  ?? 1               ->  int      (si null, on prend 1)
```

Le `?` de `as int?` est indispensable : avec `as int`, un `null` provoquerait une erreur de cast avant même d'arriver au `??`.

> **Bonne pratique :** dès qu'une clé peut manquer, écrivez `as T? ?? valeurParDefaut`.

---

## 17.13 — `jsonEncode()`

`jsonEncode()` fait le trajet inverse : il prend une structure Dart et renvoie un `String` JSON.

```dart
import 'dart:convert';

void main() {
  Map<String, dynamic> joueur = {
    'nom': 'Alex',
    'niveau': 7,
    'energie': 87.5,
    'estVivant': true,
    'arme': null,
  };

  String texte = jsonEncode(joueur);

  print(texte);
  print(texte.runtimeType);
}
```

**Résultat :**

```text
{"nom":"Alex","niveau":7,"energie":87.5,"estVivant":true,"arme":null}
String
```

Deux observations : les guillemets simples sont devenus doubles (c'est `jsonEncode` qui applique la norme), et il n'y a aucun espace (forme compacte, pour économiser des octets sur le réseau).

Pour une sortie lisible (un fichier de sauvegarde que l'on doit pouvoir inspecter), utilisez `JsonEncoder.withIndent` :

```dart
  const encodeur = JsonEncoder.withIndent('  ');
  print(encodeur.convert({'nom': 'Alex', 'inventaire': ['potion']}));
```

**Résultat :**

```text
{
  "nom": "Alex",
  "inventaire": [
    "potion"
  ]
}
```

`jsonEncode` ne sait encoder que les six types JSON. Si vous lui passez un objet de votre propre classe, il échoue :

```text
Unhandled exception:
Converting object to an encodable object failed: Instance of 'Player'
```

Ce message est le point de départ de la seconde moitié du chapitre : il faut apprendre à `jsonEncode` comment transformer un `Player` en `Map`. C'est le rôle de `toJson()` (section 17.21).

---

## 17.14 — Encoder une Map

Une `Map` devient un objet JSON. Les structures imbriquées sont encodées automatiquement, aussi profond qu'il le faut.

```dart
import 'dart:convert';

void main() {
  final ennemi = {
    'nom': 'Gobelin',
    'pointsDeVie': 30,
    'estBoss': false,
    'butin': ['piece', 'dague'],
    'position': {'x': 12, 'y': 40},
  };

  print(jsonEncode(ennemi));
}
```

**Résultat :**

```text
{"nom":"Gobelin","pointsDeVie":30,"estBoss":false,"butin":["piece","dague"],"position":{"x":12,"y":40}}
```

Contrainte à connaître : **les clés doivent être des `String`**. Une `Map<int, String>` échoue avec `Converting object to an encodable object failed`. Il faut convertir les clés :

```dart
  final scores = {1: 'Alex', 2: 'Sam'};
  final pret = scores.map((cle, valeur) => MapEntry(cle.toString(), valeur));
  print(jsonEncode(pret));   // {"1":"Alex","2":"Sam"}
```

Remarquez qu'au retour, `"1"` sera une chaîne, pas un entier. C'est une perte d'information imposée par le format.

---

## 17.15 — Encoder une List

Une `List` devient un tableau JSON.

```dart
import 'dart:convert';

void main() {
  print(jsonEncode(['potion', 'clé', 'torche']));
  print(jsonEncode([12, 30, 7]));
  print(jsonEncode([
    {'nom': 'Gobelin', 'pv': 30},
    {'nom': 'Orc', 'pv': 55},
  ]));
}
```

**Résultat :**

```text
["potion","clé","torche"]
[12,30,7]
[{"nom":"Gobelin","pv":30},{"nom":"Orc","pv":55}]
```

Un `Set` en revanche n'est **pas** encodable directement : `jsonEncode({'feu', 'glace'})` échoue avec `Converting object to an encodable object failed`. Convertissez-le d'abord avec `.toList()`, ce qui donne `["feu","glace"]`.

> **Règle simple :** avant d'appeler `jsonEncode`, vérifiez que votre structure ne contient **que** des `Map<String, ...>`, des `List`, des `String`, des `num`, des `bool` et des `null`.

---

## 17.16 — Décoder un tableau JSON en `List<dynamic>`

Quand la racine du JSON est un tableau, `jsonDecode` renvoie une `List<dynamic>`.

```dart
import 'dart:convert';

void main() {
  String texte = '[{"nom":"Gobelin","pv":30},{"nom":"Orc","pv":55}]';
  List<dynamic> brut = jsonDecode(texte) as List<dynamic>;

  print(brut.length);
  print(brut.first);
  print(brut.first.runtimeType);

  for (final element in brut) {
    print(element['nom']);
  }
}
```

**Résultat :**

```text
2
{nom: Gobelin, pv: 30}
_Map<String, dynamic>
Gobelin
Orc
```

Le type est `List<dynamic>` et non `List<Map<String, dynamic>>` : chaque élément est bien une `Map`, mais Dart ne le sait pas statiquement. La boucle fonctionne, mais `element['nom']` n'est vérifié par personne. Une faute de frappe (`element['non']`) passerait la compilation et renverrait `null` silencieusement.

---

## 17.17 — `.cast<Map<String, dynamic>>()`

La méthode `.cast<T>()` renvoie une vue typée de la liste.

```dart
import 'dart:convert';

void main() {
  String texte = '[{"nom":"Gobelin","pv":30},{"nom":"Orc","pv":55}]';

  final brut = jsonDecode(texte) as List<dynamic>;
  final ennemis = brut.cast<Map<String, dynamic>>();

  print(ennemis.runtimeType);
  for (final e in ennemis) {
    print('${e['nom']} a ${e['pv']} PV');
  }
}
```

**Résultat :**

```text
CastList<dynamic, Map<String, dynamic>>
Gobelin a 30 PV
Orc a 55 PV
```

Maintenant `e` est de type `Map<String, dynamic>` : l'éditeur propose `keys` et `containsKey`, et une méthode inexistante devient une erreur de compilation.

Alternative très courante, avec `map` (chapitre 14), qui produit une vraie `List<Map<String, dynamic>>` :

```dart
  final ennemis = (jsonDecode(texte) as List<dynamic>)
      .map((e) => e as Map<String, dynamic>)
      .toList();
```

| Approche | Type obtenu | Remarque |
| --- | --- | --- |
| `.cast<T>()` | `CastList` (vue) | pas de copie, vérification à chaque accès |
| `.map(...).toList()` | vraie `List<T>` | copie, vérification immédiate |

Nous utiliserons la seconde dans la suite du chapitre.

---

## 17.18 — Le problème : manipuler des Maps partout est fragile

Techniquement, vous savez déjà tout faire. Vous pourriez écrire votre jeu entier avec des `Map<String, dynamic>`. Ce serait une très mauvaise idée.

```dart
import 'dart:convert';

void main() {
  final joueur = jsonDecode('{"nom":"Alex","niveau":7}') as Map<String, dynamic>;

  print(joueur['non']);       // faute de frappe
  print(joueur['NIVEAU']);    // mauvaise casse
}
```

**Résultat :**

```text
null
null
```

**Aucune erreur signalée.** Le programme continue avec des `null` qui contamineront la suite et provoqueront un plantage bien plus loin, dans une fonction qui n'a rien à voir.

| Défaut | Conséquence concrète |
| --- | --- |
| Clés écrites à la main | une faute de frappe donne `null` sans alerte |
| Aucune auto-complétion | il faut mémoriser les noms de clés exacts |
| Aucun type | `joueur['niveau']` est `dynamic`, donc non vérifié |
| Aucune logique métier | impossible d'écrire `joueur.subitDegats(10)` |

Et si le format change (`niveau` devient `level`), il faut retrouver **toutes** les occurrences dispersées dans le projet.

```text
  Architecture fragile                Architecture solide
  ─────────────────────               ────────────────────
  Map<String, dynamic>                Map<String, dynamic>
      │                                   │
      ├──> écran 1 lit ['nom']            └──> Player.fromJson()
      ├──> écran 2 lit ['nom']                       │
      ├──> écran 3 lit ['non']  <- bug               ▼
      └──> service lit ['nom']              objet Player typé
                                                     │
     4 endroits à corriger                  ├──> écran 1 lit .nom
     si la clé change                       ├──> écran 2 lit .nom
                                            └──> écran 3 lit .nom

                                         1 seul endroit à corriger
```

---

## 17.19 — La solution : une classe modèle

Une **classe modèle** est une classe Dart ordinaire dont le seul rôle est de représenter une donnée de votre domaine.

```dart
class Player {
  final String nom;
  final int niveau;
  final int score;

  Player({required this.nom, required this.niveau, required this.score});

  @override
  String toString() => 'Player($nom, niv $niveau, $score pts)';
}

void main() {
  final joueur = Player(nom: 'Alex', niveau: 7, score: 3000);
  print(joueur);
  print(joueur.nom);
}
```

**Résultat :**

```text
Player(Alex, niv 7, 3000 pts)
Alex
```

| Avec une `Map` | Avec un modèle |
| --- | --- |
| `joueur['nom']` → `dynamic` | `joueur.nom` → `String` |
| `joueur['non']` → `null` | `joueur.non` → **erreur de compilation** |
| aucune méthode métier | `joueur.subitDegats(10)` possible |

Testez la faute de frappe : `print(joueur.non);` donne

```text
Error: The getter 'non' isn't defined for the class 'Player'.
```

L'erreur arrive **avant** l'exécution. C'est la philosophie du chapitre 12 appliquée aux données.

Il reste à relier les deux mondes, avec deux passerelles : `fromJson()` et `toJson()`.

---

## 17.20 — Le constructeur nommé `fromJson()`

Rappel du chapitre 9 : un constructeur nommé permet d'offrir plusieurs façons de créer un objet. Par convention universelle en Dart, celui qui construit un objet depuis une `Map` JSON s'appelle `fromJson`.

```dart
import 'dart:convert';

class Player {
  final String nom;
  final int niveau;
  final int score;

  Player({required this.nom, required this.niveau, required this.score});

  Player.fromJson(Map<String, dynamic> json)
      : nom = json['nom'] as String,
        niveau = json['niveau'] as int,
        score = json['score'] as int;

  @override
  String toString() => 'Player($nom, niv $niveau, $score pts)';
}

void main() {
  final map = jsonDecode('{"nom":"Alex","niveau":7,"score":3000}')
      as Map<String, dynamic>;
  final joueur = Player.fromJson(map);

  print(joueur);
  print(joueur.niveau + 1);
}
```

**Résultat :**

```text
Player(Alex, niv 7, 3000 pts)
8
```

La syntaxe `: nom = ..., niveau = ...` est la **liste d'initialisation** du chapitre 9. Elle est obligatoire ici parce que les champs sont `final`.

Beaucoup de développeurs préfèrent un constructeur `factory`, plus souple car il autorise du code avant la construction :

```dart
  factory Player.fromJson(Map<String, dynamic> json) {
    final niveauBrut = json['niveau'] as int;
    return Player(
      nom: json['nom'] as String,
      niveau: niveauBrut < 1 ? 1 : niveauBrut,   // on corrige une donnée aberrante
      score: json['score'] as int,
    );
  }
```

Avec ce `factory`, un JSON contenant `"niveau": 0` produit un joueur de niveau 1. C'est la forme que nous utiliserons dans la suite du chapitre.

---

## 17.21 — La méthode `toJson()`

`toJson()` transforme l'objet en `Map<String, dynamic>`.

Attention au piège de vocabulaire : malgré son nom, `toJson()` ne renvoie **pas** du texte JSON. Elle renvoie une `Map`. C'est `jsonEncode` qui produit le texte.

```dart
import 'dart:convert';

class Player {
  final String nom;
  final int niveau;
  final int score;

  Player({required this.nom, required this.niveau, required this.score});

  Map<String, dynamic> toJson() {
    return {'nom': nom, 'niveau': niveau, 'score': score};
  }
}

void main() {
  final joueur = Player(nom: 'Alex', niveau: 7, score: 3000);

  final map = joueur.toJson();
  print(map);
  print(map.runtimeType);
  print(jsonEncode(map));
}
```

**Résultat :**

```text
{nom: Alex, niveau: 7, score: 3000}
_Map<String, dynamic>
{"nom":"Alex","niveau":7,"score":3000}
```

Bonus très pratique : `jsonEncode` appelle **automatiquement** `toJson()` sur un objet qu'il ne sait pas encoder. Avec la classe ci-dessus :

```dart
  print(jsonEncode(joueur));                       // pas besoin de .toJson()
  print(jsonEncode([joueur, joueur]));             // fonctionne aussi sur une liste
```

**Résultat :**

```text
{"nom":"Alex","niveau":7,"score":3000}
[{"nom":"Alex","niveau":7,"score":3000},{"nom":"Alex","niveau":7,"score":3000}]
```

C'est pour cela que la méthode doit s'appeler **exactement** `toJson`, sans paramètre obligatoire. Si vous l'appelez `versJson()`, l'automatisme disparaît et vous retombez sur `Converting object to an encodable object failed`.

---

## 17.22 — Aller-retour objet → JSON → objet

Un modèle correct doit satisfaire une propriété simple :

> Encoder un objet puis le décoder doit redonner un objet **équivalent**.

C'est l'**aller-retour** (*round-trip*), le test numéro un de tout modèle.

```dart
import 'dart:convert';

class Player {
  final String nom;
  final int niveau;
  final int score;

  Player({required this.nom, required this.niveau, required this.score});

  factory Player.fromJson(Map<String, dynamic> json) => Player(
        nom: json['nom'] as String,
        niveau: json['niveau'] as int,
        score: json['score'] as int,
      );

  Map<String, dynamic> toJson() =>
      {'nom': nom, 'niveau': niveau, 'score': score};

  @override
  String toString() => 'Player($nom, niv $niveau, $score pts)';
}

void main() {
  final original = Player(nom: 'Alex', niveau: 7, score: 3000);
  final texte = jsonEncode(original.toJson());
  final copie = Player.fromJson(jsonDecode(texte) as Map<String, dynamic>);

  print('1. objet de départ   : $original');
  print('2. texte JSON        : $texte');
  print('3. objet reconstruit : $copie');

  final identique = original.nom == copie.nom &&
      original.niveau == copie.niveau &&
      original.score == copie.score;
  print('Aller-retour réussi ? $identique');
}
```

**Résultat :**

```text
1. objet de départ   : Player(Alex, niv 7, 3000 pts)
2. texte JSON        : {"nom":"Alex","niveau":7,"score":3000}
3. objet reconstruit : Player(Alex, niv 7, 3000 pts)
Aller-retour réussi ? true
```

Le schéma complet, à retenir pour tout le reste de la formation :

```text
   ┌────────┐  toJson()   ┌─────┐  jsonEncode()  ┌────────┐
   │ Player │ ──────────> │ Map │ ─────────────> │ String │
   └────────┘             └─────┘                └────────┘
        ▲                    ▲                        │
        │  fromJson()        │  jsonDecode()          │
        └────────────────────┴────────────────────────┘
```

> **Attention :** `original == copie` renverrait `false`, car sans redéfinition de `==` Dart compare les identités (chapitre 10), pas les contenus. D'où la comparaison champ par champ.

---

## 17.23 — Une liste d'objets depuis un tableau JSON

C'est le cas le plus fréquent : une API renvoie un tableau, vous voulez une `List<Enemy>`.

```dart
import 'dart:convert';

class Enemy {
  final String nom;
  final int pointsDeVie;

  Enemy({required this.nom, required this.pointsDeVie});

  factory Enemy.fromJson(Map<String, dynamic> json) => Enemy(
        nom: json['nom'] as String,
        pointsDeVie: json['pv'] as int,
      );

  Map<String, dynamic> toJson() => {'nom': nom, 'pv': pointsDeVie};

  static List<Enemy> listFromJson(List<dynamic> liste) =>
      liste.map((e) => Enemy.fromJson(e as Map<String, dynamic>)).toList();

  @override
  String toString() => '$nom($pointsDeVie PV)';
}

void main() {
  const texte = '[{"nom":"Gobelin","pv":30},{"nom":"Orc","pv":55},'
      '{"nom":"Dragon","pv":400}]';

  final ennemis = Enemy.listFromJson(jsonDecode(texte) as List<dynamic>);

  print(ennemis);
  print('PV cumulés : ${ennemis.fold<int>(0, (s, e) => s + e.pointsDeVie)}');
  print('Costauds : ${ennemis.where((e) => e.pointsDeVie > 50).toList()}');
  print(jsonEncode(ennemis));
}
```

**Résultat :**

```text
[Gobelin(30 PV), Orc(55 PV), Dragon(400 PV)]
PV cumulés : 485
Costauds : [Orc(55 PV), Dragon(400 PV)]
[{"nom":"Gobelin","pv":30},{"nom":"Orc","pv":55},{"nom":"Dragon","pv":400}]
```

Une fois la liste typée obtenue, toute la programmation fonctionnelle du chapitre 14 redevient disponible. La méthode statique `listFromJson` évite de répéter la même transformation à dix endroits du projet.

---

## 17.24 — Objets imbriqués

Un joueur possède une arme, et l'arme est elle-même un objet.

```json
{
  "nom": "Alex",
  "niveau": 7,
  "arme": { "nom": "Épée de feu", "degats": 25 }
}
```

La règle est simple et se répète à l'infini :

> Dans `fromJson`, chaque sous-objet est construit par **son propre** `fromJson`.
> Dans `toJson`, chaque sous-objet est converti par **son propre** `toJson`.

```dart
import 'dart:convert';

class Weapon {
  final String nom;
  final int degats;

  Weapon({required this.nom, required this.degats});

  factory Weapon.fromJson(Map<String, dynamic> json) => Weapon(
        nom: json['nom'] as String,
        degats: json['degats'] as int,
      );

  Map<String, dynamic> toJson() => {'nom': nom, 'degats': degats};

  @override
  String toString() => '$nom (+$degats)';
}

class Player {
  final String nom;
  final int niveau;
  final Weapon arme;

  Player({required this.nom, required this.niveau, required this.arme});

  factory Player.fromJson(Map<String, dynamic> json) => Player(
        nom: json['nom'] as String,
        niveau: json['niveau'] as int,
        arme: Weapon.fromJson(json['arme'] as Map<String, dynamic>),
      );

  Map<String, dynamic> toJson() =>
      {'nom': nom, 'niveau': niveau, 'arme': arme.toJson()};

  @override
  String toString() => 'Player($nom, niv $niveau, arme : $arme)';
}

void main() {
  const texte =
      '{"nom":"Alex","niveau":7,"arme":{"nom":"Épée de feu","degats":25}}';

  final joueur = Player.fromJson(jsonDecode(texte) as Map<String, dynamic>);
  print(joueur);
  print(joueur.arme.degats);
  print(jsonEncode(joueur));
}
```

**Résultat :**

```text
Player(Alex, niv 7, arme : Épée de feu (+25))
25
{"nom":"Alex","niveau":7,"arme":{"nom":"Épée de feu","degats":25}}
```

Si l'arme est facultative, le champ devient `Weapon?` et la lecture devient conditionnelle :

```dart
  factory Player.fromJson(Map<String, dynamic> json) {
    final brut = json['arme'];
    return Player(
      nom: json['nom'] as String,
      niveau: json['niveau'] as int,
      arme: brut == null ? null : Weapon.fromJson(brut as Map<String, dynamic>),
    );
  }

  Map<String, dynamic> toJson() =>
      {'nom': nom, 'niveau': niveau, 'arme': arme?.toJson()};
```

Un joueur sans arme produit alors `{"nom":"Sam","niveau":1,"arme":null}`, et `joueur.arme?.nom ?? 'mains nues'` affiche `mains nues`. On retrouve tous les outils du chapitre 12 : `?`, `?.` et `??`.

---

## 17.25 — Listes d'objets imbriqués

Un joueur possède un inventaire, c'est-à-dire une **liste d'objets** `Item`.

```json
{
  "nom": "Alex",
  "inventaire": [
    { "nom": "Potion", "quantite": 3 },
    { "nom": "Clé",    "quantite": 1 }
  ]
}
```

Le principe reste identique, avec un `.map(...)` en plus, dans les deux sens.

```dart
import 'dart:convert';

class Item {
  final String nom;
  final int quantite;

  Item({required this.nom, required this.quantite});

  factory Item.fromJson(Map<String, dynamic> json) => Item(
        nom: json['nom'] as String,
        quantite: json['quantite'] as int,
      );

  Map<String, dynamic> toJson() => {'nom': nom, 'quantite': quantite};

  @override
  String toString() => '$nom x$quantite';
}

class Player {
  final String nom;
  final List<Item> inventaire;

  Player({required this.nom, required this.inventaire});

  factory Player.fromJson(Map<String, dynamic> json) {
    final brut = json['inventaire'] as List<dynamic>? ?? [];
    return Player(
      nom: json['nom'] as String,
      inventaire:
          brut.map((e) => Item.fromJson(e as Map<String, dynamic>)).toList(),
    );
  }

  Map<String, dynamic> toJson() => {
        'nom': nom,
        'inventaire': inventaire.map((i) => i.toJson()).toList(),
      };

  @override
  String toString() => 'Player($nom, sac : $inventaire)';
}

void main() {
  const texte = '{"nom":"Alex","inventaire":['
      '{"nom":"Potion","quantite":3},{"nom":"Clé","quantite":1}]}';

  final joueur = Player.fromJson(jsonDecode(texte) as Map<String, dynamic>);
  print(joueur);
  print(jsonEncode(joueur));

  final vide =
      Player.fromJson(jsonDecode('{"nom":"Sam"}') as Map<String, dynamic>);
  print(vide);
}
```

**Résultat :**

```text
Player(Alex, sac : [Potion x3, Clé x1])
{"nom":"Alex","inventaire":[{"nom":"Potion","quantite":3},{"nom":"Clé","quantite":1}]}
Player(Sam, sac : [])
```

Deux détails importants.

**1.** `json['inventaire'] as List<dynamic>? ?? []` : si la clé manque, on obtient une liste vide plutôt qu'un plantage.

**2.** `inventaire.map((i) => i.toJson()).toList()` : écrire ce `.map` explicitement rend `toJson()` autonome, utilisable même en dehors de `jsonEncode`.

---

## 17.26 — Valeurs manquantes et valeurs par défaut

Dans le monde réel, un JSON est rarement parfait. Une clé peut manquer, un serveur peut envoyer `null`, une sauvegarde peut dater d'une version antérieure du jeu.

| JSON reçu | `json['arme']` vaut | `containsKey('arme')` |
| --- | --- | --- |
| `{"arme": "Épée"}` | `'Épée'` | `true` |
| `{"arme": null}` | `null` | `true` |
| `{}` | `null` | `false` |

Dans la grande majorité des cas, on traite les deux derniers de la même façon : « pas d'arme ».

```dart
import 'dart:convert';

class Player {
  final String nom;
  final int niveau;
  final int score;
  final List<String> inventaire;

  Player({
    required this.nom,
    required this.niveau,
    required this.score,
    required this.inventaire,
  });

  factory Player.fromJson(Map<String, dynamic> json) => Player(
        nom: json['nom'] as String? ?? 'Anonyme',
        niveau: json['niveau'] as int? ?? 1,
        score: json['score'] as int? ?? 0,
        inventaire: (json['inventaire'] as List<dynamic>? ?? []).cast<String>(),
      );

  @override
  String toString() => 'Player($nom, niv $niveau, $score pts, sac $inventaire)';
}

void main() {
  print(Player.fromJson(jsonDecode('{}') as Map<String, dynamic>));
  print(Player.fromJson(
      jsonDecode('{"nom":"Alex","niveau":null}') as Map<String, dynamic>));
  print(Player.fromJson(jsonDecode(
          '{"nom":"Sam","niveau":3,"score":900,"inventaire":["potion"]}')
      as Map<String, dynamic>));
}
```

**Résultat :**

```text
Player(Anonyme, niv 1, 0 pts, sac [])
Player(Alex, niv 1, 0 pts, sac [])
Player(Sam, niv 3, 900 pts, sac [potion])
```

Le modèle n'a pas planté une seule fois, même sur `{}`.

| Situation | Décision recommandée |
| --- | --- |
| `score` absent | valeur par défaut `0` |
| `inventaire` absent | liste vide |
| `niveau` absent | valeur par défaut `1` |
| `id` absent | **lever une exception** : la donnée est inexploitable |

Pour ce dernier cas :

```dart
  final id = json['id'] as String?;
  if (id == null) {
    throw FormatException('Champ "id" manquant dans le JSON du joueur');
  }
```

C'est une application directe du chapitre 13 : mieux vaut une exception claire à l'endroit du problème qu'un `null` qui se propage.

---

## 17.27 — Types incohérents et conversion défensive

Voici le bug qui fait perdre le plus de temps face à une vraie API.

```dart
import 'dart:convert';

void main() {
  final json = jsonDecode('{"score": "3000"}') as Map<String, dynamic>;
  final int score = json['score'] as int;
  print(score);
}
```

**Résultat :**

```text
Unhandled exception:
type 'String' is not a subtype of type 'int' in type cast
```

Le serveur a envoyé `"3000"` (un texte) au lieu de `3000` (un nombre). Certaines API et certaines bases renvoient tous les nombres sous forme de chaînes.

Variante plus sournoise : le JSON `100` est décodé en `int`, mais votre modèle attend un `double`. `json['energie'] as double` échoue alors que la valeur est correcte.

La solution est d'écrire des **fonctions de conversion défensives**, une fois pour toutes.

```dart
int lireInt(dynamic valeur, {int defaut = 0}) {
  if (valeur is int) return valeur;
  if (valeur is double) return valeur.toInt();
  if (valeur is String) return int.tryParse(valeur) ?? defaut;
  return defaut;
}

double lireDouble(dynamic valeur, {double defaut = 0.0}) {
  if (valeur is num) return valeur.toDouble();
  if (valeur is String) return double.tryParse(valeur) ?? defaut;
  return defaut;
}

String lireString(dynamic valeur, {String defaut = ''}) {
  if (valeur is String) return valeur;
  if (valeur == null) return defaut;
  return valeur.toString();
}

bool lireBool(dynamic valeur, {bool defaut = false}) {
  if (valeur is bool) return valeur;
  if (valeur is String) return valeur.toLowerCase() == 'true';
  if (valeur is num) return valeur != 0;
  return defaut;
}

void main() {
  print(lireInt('3000'));
  print(lireInt('abc'));
  print(lireInt(null, defaut: 1));
  print(lireDouble(100));
  print(lireBool('true'));
}
```

**Résultat :**

```text
3000
0
1
100.0
true
```

Le modèle devient alors à la fois plus court et incassable :

```dart
  factory Player.fromJson(Map<String, dynamic> json) => Player(
        nom: lireString(json['nom'], defaut: 'Anonyme'),
        niveau: lireInt(json['niveau'], defaut: 1),
        energie: lireDouble(json['energie'], defaut: 100.0),
      );
```

Notez le rôle central de `int.tryParse` et `double.tryParse` (chapitre 13) : contrairement à `int.parse`, ils renvoient `null` au lieu de lever une exception.

---

## 17.28 — `FormatException` sur un JSON invalide

Que se passe-t-il quand le texte n'est pas du JSON valide ?

```dart
import 'dart:convert';

void main() {
  print(jsonDecode('{"nom": "Alex", }'));
}
```

**Résultat :**

```text
Unhandled exception:
FormatException: Unexpected character (at character 17)
{"nom": "Alex", }
                ^
```

`jsonDecode` lève une `FormatException`, exactement comme `int.parse` au chapitre 13. Le message indique la position du caractère fautif.

En production, ce texte vient d'un fichier ou d'un serveur : vous ne pouvez pas garantir qu'il sera valide. Il faut donc l'attraper.

```dart
import 'dart:convert';

Map<String, dynamic>? decoderJoueur(String texte) {
  try {
    final resultat = jsonDecode(texte);
    if (resultat is! Map<String, dynamic>) {
      print('Erreur : la racine du JSON n\'est pas un objet.');
      return null;
    }
    return resultat;
  } on FormatException catch (e) {
    print('JSON invalide : ${e.message}');
    return null;
  }
}

void main() {
  print(decoderJoueur('{"nom":"Alex"}'));
  print(decoderJoueur('{"nom": "Alex", }'));
  print(decoderJoueur('[1,2,3]'));
}
```

**Résultat :**

```text
{nom: Alex}
JSON invalide : Unexpected character
null
Erreur : la racine du JSON n'est pas un objet.
null
```

Deux techniques cohabitent, à bien distinguer :

| Problème | Outil |
| --- | --- |
| Le texte n'est pas du JSON | `try / on FormatException` |
| Le JSON est valide mais pas de la forme attendue | test `is` / `is!` avant le cast |

Le test `is!` est préférable au `as` quand vous voulez traiter le cas d'erreur proprement : `as` lève un `TypeError` que l'on n'attrape presque jamais, alors que `is!` vous laisse décider.

---

## 17.29 — Les enums et JSON

Au chapitre 11, vous avez appris à remplacer les « chaînes magiques » par des `enum`. Un objet n'a plus une rareté écrite `'epique'` au hasard : il a une valeur `Rarete.epique`, contrôlée par le compilateur.

Problème : **JSON ne connaît pas les enums.** Les six types JSON de la section 17.3 sont `string`, `number`, `boolean`, `null`, `object` et `array`. Il n'y a pas de septième case pour vos énumérations.

Il faut donc choisir une **représentation** de l'enum dans le JSON. Deux options existent.

| Représentation | JSON produit | Verdict |
| --- | --- | --- |
| Le **nom** de la valeur | `"rarete": "epique"` | recommandé |
| L'**index** de la valeur | `"rarete": 2` | à éviter |

Pourquoi éviter l'index ? Parce qu'il dépend de l'ordre de déclaration. Le jour où vous insérez une valeur au milieu de l'enum, toutes les sauvegardes déjà écrites sur le disque des joueurs deviennent fausses, sans le moindre message d'erreur.

```text
  Version 1 du jeu                     Version 2 du jeu
  enum Rarete {                        enum Rarete {
    commun,      // 0                    commun,      // 0
    rare,        // 1                    inhabituel,  // 1   <-- inséré
    epique,      // 2                    rare,        // 2
    legendaire   // 3                    epique,      // 3
  }                                      legendaire   // 4
                                       }

  Sauvegarde écrite en v1 : {"rarete": 2}
      relue en v1 ......... epique
      relue en v2 ......... rare     <-- l'objet a changé de rareté
```

Le nom, lui, ne bouge pas. `"epique"` restera `"epique"` quelle que soit la position de la valeur dans l'enum.

Dart fournit deux outils pour cela, disponibles sur tout enum :

```dart
enum Rarete { commun, rare, epique, legendaire }

void main() {
  const r = Rarete.epique;

  print(r);
  print(r.name);
  print(r.index);
  print(Rarete.values);
  print(Rarete.values.byName('rare'));
}
```

**Résultat :**

```text
Rarete.epique
epique
2
[Rarete.commun, Rarete.rare, Rarete.epique, Rarete.legendaire]
Rarete.rare
```

- `.name` donne le nom de la valeur sous forme de `String` : c'est ce que l'on écrit dans le JSON.
- `.values.byName(...)` fait le chemin inverse : d'un `String` vers la valeur d'enum.

Attention cependant : `byName` **lève une exception** si le nom est inconnu.

```dart
enum Rarete { commun, rare, epique, legendaire }

void main() {
  print(Rarete.values.byName('epique'));
  print(Rarete.values.byName('mythique'));
}
```

**Résultat :**

```text
Rarete.epique
Unhandled exception:
Invalid argument (name): No enum value with that name: "mythique"
```

Or un JSON vient souvent d'ailleurs : d'un serveur, d'un fichier de configuration écrit à la main, d'une ancienne version de votre jeu. Un nom inconnu est donc une situation **normale**, pas un bug. On écrit donc une fonction de lecture tolérante, exactement dans l'esprit défensif de la section 17.27.

```dart
enum Rarete { commun, rare, epique, legendaire }

Rarete rareteDepuisTexte(dynamic valeur, {Rarete defaut = Rarete.commun}) {
  if (valeur is String) {
    for (final r in Rarete.values) {
      if (r.name == valeur) return r;
    }
  }
  return defaut;
}

void main() {
  print(rareteDepuisTexte('legendaire'));
  print(rareteDepuisTexte('mythique'));
  print(rareteDepuisTexte(null));
  print(rareteDepuisTexte(2));
}
```

**Résultat :**

```text
Rarete.legendaire
Rarete.commun
Rarete.commun
Rarete.commun
```

Quatre entrées, aucun plantage. Un nom valide donne la bonne valeur, tout le reste retombe sur la valeur par défaut.

Voici maintenant le modèle complet, avec l'enum en aller-retour.

```dart
import 'dart:convert';

enum Rarete { commun, rare, epique, legendaire }

Rarete rareteDepuisTexte(dynamic valeur, {Rarete defaut = Rarete.commun}) {
  if (valeur is String) {
    for (final r in Rarete.values) {
      if (r.name == valeur) return r;
    }
  }
  return defaut;
}

class Item {
  final String nom;
  final Rarete rarete;

  Item({required this.nom, required this.rarete});

  factory Item.fromJson(Map<String, dynamic> json) => Item(
        nom: json['nom'] as String? ?? 'Objet inconnu',
        rarete: rareteDepuisTexte(json['rarete']),
      );

  Map<String, dynamic> toJson() => {'nom': nom, 'rarete': rarete.name};

  @override
  String toString() => '$nom [${rarete.name}]';
}

void main() {
  const texte = '[{"nom":"Potion","rarete":"commun"},'
      '{"nom":"Épée de feu","rarete":"epique"},'
      '{"nom":"Relique","rarete":"mythique"},'
      '{"nom":"Torche"}]';

  final objets = (jsonDecode(texte) as List<dynamic>)
      .map((e) => Item.fromJson(e as Map<String, dynamic>))
      .toList();

  for (final o in objets) {
    print(o);
  }

  print(jsonEncode(objets));
}
```

**Résultat :**

```text
Potion [commun]
Épée de feu [epique]
Relique [commun]
Torche [commun]
[{"nom":"Potion","rarete":"commun"},{"nom":"Épée de feu","rarete":"epique"},{"nom":"Relique","rarete":"commun"},{"nom":"Torche","rarete":"commun"}]
```

Observez la troisième ligne. `"mythique"` n'existe pas dans l'enum : l'objet devient `commun` au lieu de faire planter le chargement de la partie. Le joueur perd une nuance de rareté, il ne perd pas sa sauvegarde.

> **Remarque :** ce comportement est un **choix**. Si un nom inconnu signale une sauvegarde corrompue que vous refusez de charger, remplacez le `return defaut;` par un `throw FormatException('Rareté inconnue : $valeur');`. L'important est de décider, pas de subir.

Si votre enum utilise des noms Dart qui ne correspondent pas au JSON du serveur (par exemple `epique` côté Dart et `EPIC` côté serveur), ajoutez une `Map` de correspondance :

```dart
enum Rarete { commun, rare, epique, legendaire }

const Map<String, Rarete> _depuisApi = {
  'COMMON': Rarete.commun,
  'RARE': Rarete.rare,
  'EPIC': Rarete.epique,
  'LEGENDARY': Rarete.legendaire,
};

const Map<Rarete, String> _versApi = {
  Rarete.commun: 'COMMON',
  Rarete.rare: 'RARE',
  Rarete.epique: 'EPIC',
  Rarete.legendaire: 'LEGENDARY',
};

void main() {
  print(_depuisApi['EPIC'] ?? Rarete.commun);
  print(_depuisApi['???'] ?? Rarete.commun);
  print(_versApi[Rarete.legendaire]);
}
```

**Résultat :**

```text
Rarete.epique
Rarete.commun
LEGENDARY
```

Les deux `Map` sont `const` : elles ne coûtent rien à l'exécution, et elles isolent en un seul endroit du projet le vocabulaire imposé par le serveur.

---

## 17.30 — `DateTime` et JSON

Même constat que pour les enums, et pour la même raison : **JSON ne connaît pas les dates.** Il n'y a ni type `date`, ni type `datetime` dans le format.

Une date doit donc voyager sous la forme d'un des six types disponibles. Deux conventions dominent.

| Convention | JSON produit | Lisible ? | Usage |
| --- | --- | --- | --- |
| Texte ISO 8601 | `"2026-08-08T14:30:00.000Z"` | oui | recommandé |
| Nombre d'millisecondes | `1786199400000` | non | interne, compact |

ISO 8601 est un standard international : année, mois, jour, `T`, heure, minutes, secondes. Tous les langages savent le lire. Dart le produit et le relit avec deux méthodes.

```dart
void main() {
  final creation = DateTime.utc(2026, 8, 8, 14, 30);

  print(creation);
  print(creation.toIso8601String());
  print(creation.millisecondsSinceEpoch);

  final relue = DateTime.parse('2026-08-08T14:30:00.000Z');
  print(relue);
  print(relue == creation);
}
```

**Résultat :**

```text
2026-08-08 14:30:00.000Z
2026-08-08T14:30:00.000Z
1786199400000
2026-08-08 14:30:00.000Z
true
```

Le test final `relue == creation` renvoie `true` : l'aller-retour est **sans perte**. C'est exactement la propriété que l'on recherche, comme pour les objets à la section 17.22.

Trois méthodes à retenir :

| Méthode | Sens | Type |
| --- | --- | --- |
| `date.toIso8601String()` | objet → texte | `String` |
| `DateTime.parse(texte)` | texte → objet | lève une `FormatException` si invalide |
| `DateTime.tryParse(texte)` | texte → objet | renvoie `null` si invalide |

Comme partout dans ce chapitre, la version `try...` est celle qui convient à des données venues de l'extérieur.

```dart
void main() {
  print(DateTime.tryParse('2026-08-08T14:30:00.000Z'));
  print(DateTime.tryParse('hier'));
  print(DateTime.tryParse(''));
}
```

**Résultat :**

```text
2026-08-08 14:30:00.000Z
null
null
```

Un piège classique mérite d'être nommé tout de suite : **le fuseau horaire**.

```text
  DateTime.now()        ──> heure LOCALE de la machine
  DateTime.now().toUtc() ──> même instant, exprimé en UTC

  Sauvegarde écrite à Paris   : "2026-08-08T16:30:00.000"   (pas de Z)
  Relue à Tokyo               : 16h30 heure de Tokyo -> faux instant
```

La règle professionnelle tient en une ligne :

> **Stockez toujours en UTC. Convertissez en heure locale uniquement au moment de l'affichage.**

Un texte ISO qui se termine par `Z` est en UTC ; sans `Z`, il est interprété comme une heure locale. Le `Z` n'est donc pas un détail cosmétique.

```dart
void main() {
  final avecZ = DateTime.parse('2026-08-08T14:30:00.000Z');
  final sansZ = DateTime.parse('2026-08-08T14:30:00.000');

  print(avecZ.isUtc);
  print(sansZ.isUtc);
  print(avecZ.toIso8601String());
}
```

**Résultat :**

```text
true
false
2026-08-08T14:30:00.000Z
```

Voici le modèle complet, avec une date obligatoire et une date facultative.

```dart
import 'dart:convert';

class Partie {
  final String joueur;
  final DateTime creation;
  final DateTime? dernierEnregistrement;

  Partie({
    required this.joueur,
    required this.creation,
    this.dernierEnregistrement,
  });

  factory Partie.fromJson(Map<String, dynamic> json) {
    final brutCreation = json['creation'];
    final brutDernier = json['dernierEnregistrement'];

    return Partie(
      joueur: json['joueur'] as String? ?? 'Anonyme',
      creation: (brutCreation is String ? DateTime.tryParse(brutCreation) : null)
              ?.toUtc() ??
          DateTime.utc(1970),
      dernierEnregistrement:
          (brutDernier is String ? DateTime.tryParse(brutDernier) : null)
              ?.toUtc(),
    );
  }

  Map<String, dynamic> toJson() => {
        'joueur': joueur,
        'creation': creation.toIso8601String(),
        'dernierEnregistrement': dernierEnregistrement?.toIso8601String(),
      };

  @override
  String toString() =>
      'Partie($joueur, créée le ${creation.toIso8601String()}, '
      'dernier enregistrement : ${dernierEnregistrement?.toIso8601String() ?? "jamais"})';
}

void main() {
  final p = Partie(
    joueur: 'Alex',
    creation: DateTime.utc(2026, 8, 1, 9, 0),
    dernierEnregistrement: DateTime.utc(2026, 8, 8, 14, 30),
  );

  final texte = jsonEncode(p);
  print(texte);

  final relue = Partie.fromJson(jsonDecode(texte) as Map<String, dynamic>);
  print(relue);
  print(relue.creation == p.creation);

  final abimee = Partie.fromJson(
      jsonDecode('{"joueur":"Sam","creation":"avant-hier"}')
          as Map<String, dynamic>);
  print(abimee);
}
```

**Résultat :**

```text
{"joueur":"Alex","creation":"2026-08-01T09:00:00.000Z","dernierEnregistrement":"2026-08-08T14:30:00.000Z"}
Partie(Alex, créée le 2026-08-01T09:00:00.000Z, dernier enregistrement : 2026-08-08T14:30:00.000Z)
true
Partie(Sam, créée le 1970-01-01T00:00:00.000Z, dernier enregistrement : jamais)
```

Quatre points à relever dans ce code.

**1.** `dernierEnregistrement?.toIso8601String()` produit `null` dans le JSON si la date est absente. C'est valide : `null` est un type JSON à part entière (section 17.3).

**2.** `DateTime.tryParse` évite la `FormatException` sur `"avant-hier"`.

**3.** `.toUtc()` normalise systématiquement : quelle que soit la façon dont la date a été écrite, l'objet en mémoire est en UTC.

**4.** `DateTime.utc(1970)` sert de date de repli. Ce n'est pas la plus élégante des solutions, mais elle est explicite : une partie affichée comme créée en 1970 signale immédiatement une date illisible.

Si vous préférez la représentation numérique, l'aller-retour se fait avec deux autres méthodes :

```dart
void main() {
  final d = DateTime.utc(2026, 8, 8, 14, 30);

  final ms = d.millisecondsSinceEpoch;
  print(ms);

  final relue = DateTime.fromMillisecondsSinceEpoch(ms, isUtc: true);
  print(relue);
  print(relue == d);
}
```

**Résultat :**

```text
1786199400000
2026-08-08 14:30:00.000Z
true
```

N'oubliez jamais `isUtc: true` en relecture : sans ce paramètre, Dart reconstruit une date locale, et vous réintroduisez précisément le bug de fuseau que vous cherchiez à éviter.

| Besoin | Choix |
| --- | --- |
| Échange avec une API ou un fichier lisible | `toIso8601String()` |
| Sauvegarde interne compacte | `millisecondsSinceEpoch` |
| Date facultative | `DateTime?` + `?.toIso8601String()` |
| Donnée venue de l'extérieur | `DateTime.tryParse` |

---

## 17.31 — Sauvegarder une partie en JSON

Nous avons maintenant tout ce qu'il faut pour répondre à la question posée à la section 17.1 : **comment un joueur retrouve-t-il sa progression après avoir fermé le jeu ?**

La réponse tient en quatre étapes.

```text
  ┌──────────────┐   toJson()    ┌──────────────┐  jsonEncode  ┌──────────┐
  │ objets Dart  │ ────────────> │ Map / List   │ ───────────> │  texte   │
  └──────────────┘               └──────────────┘              └────┬─────┘
        ▲                                                           │ écriture
        │                                                           ▼
        │                                                    ┌─────────────┐
        │                                                    │ sauvegarde  │
        │                                                    │   .json     │
        │                                                    └─────┬───────┘
        │  fromJson()            ┌──────────────┐ jsonDecode       │ lecture
        └───────────────────────  │ Map / List  │ <────────────────┘
                                 └──────────────┘
```

Pour écrire un fichier, il faut la bibliothèque `dart:io`. Attention : **`dart:io` ne fonctionne pas dans DartPad**, qui s'exécute dans un navigateur et n'a donc pas accès au disque. Ce code se lance dans un vrai projet Dart, comme vous avez appris à en créer au chapitre 16 :

```text
dart create -t console jeu_sauvegarde
cd jeu_sauvegarde
dart run
```

Voici le programme complet.

```dart
import 'dart:convert';
import 'dart:io';

class Sauvegarde {
  final int version;
  final String joueur;
  final int niveau;
  final int score;
  final List<String> inventaire;

  Sauvegarde({
    this.version = 1,
    required this.joueur,
    required this.niveau,
    required this.score,
    required this.inventaire,
  });

  factory Sauvegarde.fromJson(Map<String, dynamic> json) => Sauvegarde(
        version: json['version'] as int? ?? 1,
        joueur: json['joueur'] as String? ?? 'Anonyme',
        niveau: json['niveau'] as int? ?? 1,
        score: json['score'] as int? ?? 0,
        inventaire: (json['inventaire'] as List<dynamic>? ?? []).cast<String>(),
      );

  Map<String, dynamic> toJson() => {
        'version': version,
        'joueur': joueur,
        'niveau': niveau,
        'score': score,
        'inventaire': inventaire,
      };

  @override
  String toString() =>
      'Sauvegarde(v$version, $joueur, niv $niveau, $score pts, $inventaire)';
}

const JsonEncoder encodeurLisible = JsonEncoder.withIndent('  ');

Future<void> ecrireSauvegarde(String chemin, Sauvegarde s) async {
  final fichier = File(chemin);
  await fichier.writeAsString(encodeurLisible.convert(s.toJson()));
}

Future<Sauvegarde?> lireSauvegarde(String chemin) async {
  final fichier = File(chemin);

  if (!await fichier.exists()) {
    return null;
  }

  try {
    final texte = await fichier.readAsString();
    final brut = jsonDecode(texte);

    if (brut is! Map<String, dynamic>) {
      print('Fichier de sauvegarde inattendu : la racine n\'est pas un objet.');
      return null;
    }

    return Sauvegarde.fromJson(brut);
  } on FormatException catch (e) {
    print('Sauvegarde illisible : ${e.message}');
    return null;
  }
}

Future<void> main() async {
  const chemin = 'sauvegarde.json';

  final avant = await lireSauvegarde(chemin);
  print('Chargement au démarrage : $avant');

  final partie = Sauvegarde(
    joueur: 'Alex',
    niveau: 7,
    score: 3000,
    inventaire: ['potion', 'clé rouillée', 'épée de feu'],
  );

  await ecrireSauvegarde(chemin, partie);
  print('Partie enregistrée dans $chemin');

  final apres = await lireSauvegarde(chemin);
  print('Rechargement : $apres');
  print('Score retrouvé : ${apres?.score ?? 0}');
}
```

**Résultat (premier lancement) :**

```text
Chargement au démarrage : null
Partie enregistrée dans sauvegarde.json
Rechargement : Sauvegarde(v1, Alex, niv 7, 3000 pts, [potion, clé rouillée, épée de feu])
Score retrouvé : 3000
```

**Résultat (deuxième lancement) :**

```text
Chargement au démarrage : Sauvegarde(v1, Alex, niv 7, 3000 pts, [potion, clé rouillée, épée de feu])
Partie enregistrée dans sauvegarde.json
Rechargement : Sauvegarde(v1, Alex, niv 7, 3000 pts, [potion, clé rouillée, épée de feu])
Score retrouvé : 3000
```

La différence entre les deux lancements est **tout l'intérêt du chapitre** : au second démarrage, le programme retrouve une partie qu'il n'a pas créée. La donnée a survécu à la fin du processus.

Et voici le contenu du fichier écrit sur le disque :

```json
{
  "version": 1,
  "joueur": "Alex",
  "niveau": 7,
  "score": 3000,
  "inventaire": [
    "potion",
    "clé rouillée",
    "épée de feu"
  ]
}
```

Détaillons les cinq décisions de conception présentes dans ce code.

**1. `JsonEncoder.withIndent('  ')`.** `jsonEncode` produit tout sur une seule ligne, ce qui est parfait pour le réseau mais illisible pour un humain. `JsonEncoder.withIndent` ajoute des retours à la ligne et une indentation. Comparez :

```dart
import 'dart:convert';

void main() {
  final donnees = {
    'joueur': 'Alex',
    'inventaire': ['potion', 'clé'],
  };

  print(jsonEncode(donnees));
  print('---');
  print(const JsonEncoder.withIndent('  ').convert(donnees));
}
```

**Résultat :**

```text
{"joueur":"Alex","inventaire":["potion","clé"]}
---
{
  "joueur": "Alex",
  "inventaire": [
    "potion",
    "clé"
  ]
}
```

Pour un fichier de sauvegarde que vous ouvrirez vous-même pendant le développement, l'indentation vaut largement les quelques octets supplémentaires.

**2. Le champ `version`.** Il ne sert à rien aujourd'hui. Il vous sauvera dans six mois, le jour où la structure de la sauvegarde changera :

```dart
  factory Sauvegarde.fromJson(Map<String, dynamic> json) {
    final version = json['version'] as int? ?? 1;

    if (version == 1) {
      // ancienne structure : "score" était un texte
      return Sauvegarde(
        version: 2,
        joueur: json['joueur'] as String? ?? 'Anonyme',
        niveau: json['niveau'] as int? ?? 1,
        score: int.tryParse('${json['score']}') ?? 0,
        inventaire: (json['inventaire'] as List<dynamic>? ?? []).cast<String>(),
      );
    }

    return Sauvegarde(
      version: version,
      joueur: json['joueur'] as String? ?? 'Anonyme',
      niveau: json['niveau'] as int? ?? 1,
      score: json['score'] as int? ?? 0,
      inventaire: (json['inventaire'] as List<dynamic>? ?? []).cast<String>(),
    );
  }
```

Un numéro de version coûte une ligne à l'écriture, et rend la migration possible. Sans lui, vous devrez deviner la structure d'un fichier écrit par une version du jeu que vous ne connaissez plus.

**3. Le retour `Sauvegarde?`.** `lireSauvegarde` renvoie `null` dans trois cas : fichier absent, JSON invalide, racine inattendue. L'appelant décide quoi en faire — le plus souvent, démarrer une nouvelle partie :

```dart
  final partie = await lireSauvegarde(chemin) ??
      Sauvegarde(joueur: 'Nouveau', niveau: 1, score: 0, inventaire: []);
```

L'opérateur `??` du chapitre 12 exprime en une ligne « charge la partie, ou commence-en une neuve ».

**4. Le `try` / `on FormatException`.** Un fichier peut être corrompu : coupure de courant pendant l'écriture, édition manuelle malheureuse, disque plein. Sans le `try`, le jeu refuse de démarrer. Avec lui, il repart sur une partie neuve en expliquant pourquoi.

**5. L'écriture atomique.** Le point faible du programme ci-dessus est le moment de l'écriture. Si le processus est tué au milieu du `writeAsString`, le fichier est à moitié écrit, donc invalide, et l'ancienne sauvegarde est perdue. La parade tient en trois lignes :

```dart
Future<void> ecrireSauvegardeSure(String chemin, Sauvegarde s) async {
  final temporaire = File('$chemin.tmp');
  await temporaire.writeAsString(encodeurLisible.convert(s.toJson()));
  await temporaire.rename(chemin);
}
```

On écrit d'abord dans un fichier temporaire, puis on le renomme. Le renommage est une opération très rapide au niveau du système de fichiers : soit l'ancienne sauvegarde est en place, soit la nouvelle l'est, jamais un mélange des deux.

| Risque | Parade |
| --- | --- |
| Fichier inexistant au premier lancement | `await fichier.exists()` |
| JSON corrompu | `try` / `on FormatException` |
| Structure inattendue | test `is!` avant le cast |
| Écriture interrompue | fichier temporaire + `rename` |
| Format qui évolue | champ `version` |

> **Remarque Flutter :** en Flutter, vous n'écrirez pas dans le dossier courant. Les systèmes mobiles imposent des répertoires précis, obtenus via le package `path_provider`. Le reste — `toJson`, `jsonEncode`, `jsonDecode`, `fromJson` — est **rigoureusement identique** à ce que vous venez d'écrire. Vous ne réapprendrez que le chemin du fichier.

---

## 17.32 — Aperçu de `json_serializable`

Regardez honnêtement le code de la section 17.31. Le modèle a cinq champs, et chacun apparaît **trois fois** :

```text
  final int score;                          <- déclaration
  score: json['score'] as int? ?? 0,        <- fromJson
  'score': score,                           <- toJson
```

Sur cinq champs, c'est supportable. Sur une vraie application avec quarante champs répartis sur douze modèles, cela devient un travail de copie, et le travail de copie produit des bugs :

- une faute de frappe sur `'niveau'` écrit `'nivau'` dans `toJson` mais pas dans `fromJson` ;
- un champ ajouté à la classe et oublié dans `toJson` disparaît silencieusement des sauvegardes ;
- un champ renommé n'est corrigé qu'à deux endroits sur trois.

Aucune de ces erreurs n'est détectée par le compilateur. Elles se manifestent bien plus tard, chez le joueur, sous la forme d'une donnée perdue.

D'où l'idée de la **génération de code** : vous décrivez la classe une seule fois, un outil écrit `fromJson` et `toJson` à votre place. En Dart, l'outil standard s'appelle `json_serializable`.

Il s'installe comme vous l'avez appris au chapitre 16, dans le `pubspec.yaml` :

```yaml
name: mon_jeu
environment:
  sdk: ^3.5.0

dependencies:
  json_annotation: ^4.9.0

dev_dependencies:
  build_runner: ^2.4.0
  json_serializable: ^6.8.0
  lints: ^4.0.0
```

Notez la répartition, elle est logique :

| Dépendance | Où | Pourquoi |
| --- | --- | --- |
| `json_annotation` | `dependencies` | fournit l'annotation `@JsonSerializable`, présente dans votre code final |
| `json_serializable` | `dev_dependencies` | ne sert qu'à générer, jamais à l'exécution |
| `build_runner` | `dev_dependencies` | lance les générateurs |

Vous écrivez ensuite `lib/src/player.dart` ainsi :

```dart
import 'package:json_annotation/json_annotation.dart';

part 'player.g.dart';

@JsonSerializable()
class Player {
  final String nom;
  final int niveau;
  final int score;

  @JsonKey(name: 'inventory', defaultValue: <String>[])
  final List<String> inventaire;

  Player({
    required this.nom,
    required this.niveau,
    required this.score,
    required this.inventaire,
  });

  factory Player.fromJson(Map<String, dynamic> json) => _$PlayerFromJson(json);

  Map<String, dynamic> toJson() => _$PlayerToJson(this);
}
```

Quatre éléments nouveaux :

| Élément | Rôle |
| --- | --- |
| `part 'player.g.dart';` | rattache le fichier généré au vôtre |
| `@JsonSerializable()` | marque la classe à traiter |
| `@JsonKey(...)` | ajuste un champ (nom JSON différent, valeur par défaut) |
| `_$PlayerFromJson` / `_$PlayerToJson` | fonctions écrites par le générateur |

Tant que le fichier n'est pas généré, votre projet **ne compile pas** : `_$PlayerFromJson` n'existe pas encore. C'est normal. On lance alors la génération :

```text
dart run build_runner build --delete-conflicting-outputs
```

**Résultat :**

```text
[INFO] Generating build script completed, took 312ms
[INFO] Running build...
[INFO] Running build completed, took 1.9s
[INFO] Succeeded after 2.3s with 2 outputs
```

Un fichier `lib/src/player.g.dart` apparaît, contenant exactement ce que vous auriez écrit à la main :

```dart
part of 'player.dart';

Player _$PlayerFromJson(Map<String, dynamic> json) => Player(
      nom: json['nom'] as String,
      niveau: (json['niveau'] as num).toInt(),
      score: (json['score'] as num).toInt(),
      inventaire: (json['inventory'] as List<dynamic>?)
              ?.map((e) => e as String)
              .toList() ??
          [],
    );

Map<String, dynamic> _$PlayerToJson(Player instance) => <String, dynamic>{
      'nom': instance.nom,
      'niveau': instance.niveau,
      'score': instance.score,
      'inventory': instance.inventaire,
    };
```

Vous **ne modifiez jamais** un fichier `.g.dart` : il est réécrit à chaque génération. Pendant le développement, on laisse d'ailleurs le générateur tourner en continu :

```text
dart run build_runner watch --delete-conflicting-outputs
```

Chaque fois que vous ajoutez un champ à la classe, le `.g.dart` est refait en une seconde. L'oubli dans `toJson` devient structurellement impossible.

Faut-il l'utiliser tout de suite ? Voici une règle de décision honnête :

| Situation | Recommandation |
| --- | --- |
| Vous apprenez le JSON | écrivez à la main : c'est ce chapitre |
| 1 à 3 modèles simples | à la main, c'est plus rapide que d'installer l'outil |
| Plus de 5 modèles, ou des champs qui changent souvent | `json_serializable` |
| API externe avec des noms `snake_case` | `json_serializable` + `@JsonKey(name: ...)` |
| Modèles très particuliers (formats exotiques) | à la main, ou `@JsonKey` avec `fromJson:` / `toJson:` |

Une chose ne change pas, et c'est pour cela que la section vient **après** les vingt-neuf précédentes :

> Le générateur écrit le code que vous savez déjà écrire. Il ne vous dispense pas de le comprendre.

Le jour où le fichier généré produit une erreur — un champ nullable mal déclaré, un `DateTime` mal converti, une liste imbriquée refusée — vous devrez lire ce `.g.dart` et comprendre ce qu'il fait. Sans les sections 17.19 à 17.30, ce fichier resterait de la magie.

---

## 17.33 — Préparation aux API REST de Flutter

Dernière section, et elle est tournée vers la suite de la formation.

En Flutter, la quasi-totalité des données affichées vient d'un **serveur**. On parle d'API REST : votre application envoie une requête HTTP, le serveur répond avec du texte, et ce texte est presque toujours du JSON.

```text
  Application Flutter                          Serveur
  ┌──────────────────┐                    ┌──────────────────┐
  │                  │  GET /joueurs/12   │                  │
  │                  │ ─────────────────> │   base de        │
  │                  │                    │   données        │
  │                  │  200 OK            │                  │
  │  Player(...)     │ <───────────────── │                  │
  │                  │  {"nom":"Alex",... │                  │
  └──────────────────┘                    └──────────────────┘
           ▲                    │
           │  fromJson          │  jsonDecode
           └────────────────────┘
```

Regardez bien la partie basse du schéma : `jsonDecode` puis `fromJson`. **C'est exactement ce que vous avez fait pendant tout ce chapitre.** Le réseau ne change que la provenance du texte.

Le code Flutter typique, avec le package `http`, ressemble à ceci :

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

Future<Player> chargerJoueur(int id) async {
  final reponse =
      await http.get(Uri.parse('https://api.mon-jeu.com/joueurs/$id'));

  if (reponse.statusCode != 200) {
    throw Exception('Erreur serveur : ${reponse.statusCode}');
  }

  final brut = jsonDecode(reponse.body);

  if (brut is! Map<String, dynamic>) {
    throw const FormatException('Réponse inattendue du serveur');
  }

  return Player.fromJson(brut);
}
```

Neuf lignes, et pas une seule notion réellement nouvelle :

| Ligne | Chapitre où vous l'avez apprise |
| --- | --- |
| `Future<Player>` et `await` | chapitre 15 |
| `throw Exception(...)` | chapitre 13 |
| `jsonDecode(reponse.body)` | section 17.9 |
| `is!` avant le cast | section 17.28 |
| `Player.fromJson(brut)` | section 17.20 |

Seuls `http.get`, `Uri.parse` et `statusCode` sont nouveaux, et ils seront traités en détail dans la partie Flutter.

Comme ce chapitre doit rester exécutable sans réseau, voici la **même architecture**, avec un faux serveur écrit en Dart pur. Le `Future.delayed` du chapitre 15 simule la latence.

```dart
import 'dart:convert';

class Player {
  final String nom;
  final int niveau;
  final int score;

  Player({required this.nom, required this.niveau, required this.score});

  factory Player.fromJson(Map<String, dynamic> json) => Player(
        nom: json['nom'] as String? ?? 'Anonyme',
        niveau: json['niveau'] as int? ?? 1,
        score: json['score'] as int? ?? 0,
      );

  @override
  String toString() => 'Player($nom, niv $niveau, $score pts)';
}

class ReponseHttp {
  final int statusCode;
  final String body;

  ReponseHttp(this.statusCode, this.body);
}

Future<ReponseHttp> fausseRequete(String url) async {
  await Future<void>.delayed(const Duration(milliseconds: 300));

  if (url.endsWith('/joueurs/12')) {
    return ReponseHttp(200, '{"nom":"Alex","niveau":7,"score":3000}');
  }
  if (url.endsWith('/joueurs')) {
    return ReponseHttp(
      200,
      '[{"nom":"Alex","niveau":7,"score":3000},'
          '{"nom":"Sam","niveau":3,"score":900}]',
    );
  }
  if (url.endsWith('/casse')) {
    return ReponseHttp(200, '{"nom":"Alex",');
  }
  return ReponseHttp(404, 'Not Found');
}

Future<Player> chargerJoueur(int id) async {
  final reponse = await fausseRequete('https://api.mon-jeu.com/joueurs/$id');

  if (reponse.statusCode != 200) {
    throw Exception('Erreur serveur : ${reponse.statusCode}');
  }

  final brut = jsonDecode(reponse.body);

  if (brut is! Map<String, dynamic>) {
    throw const FormatException('Réponse inattendue du serveur');
  }

  return Player.fromJson(brut);
}

Future<List<Player>> chargerClassement() async {
  final reponse = await fausseRequete('https://api.mon-jeu.com/joueurs');

  if (reponse.statusCode != 200) {
    throw Exception('Erreur serveur : ${reponse.statusCode}');
  }

  final brut = jsonDecode(reponse.body) as List<dynamic>;
  return brut.map((e) => Player.fromJson(e as Map<String, dynamic>)).toList();
}

Future<void> main() async {
  print(await chargerJoueur(12));

  final classement = await chargerClassement();
  print(classement);
  print('Meilleur score : ${classement.first.score}');

  try {
    await chargerJoueur(99);
  } catch (e) {
    print('Attrapé : $e');
  }

  try {
    final r = await fausseRequete('https://api.mon-jeu.com/casse');
    jsonDecode(r.body);
  } on FormatException catch (e) {
    print('JSON du serveur invalide : ${e.message}');
  }
}
```

**Résultat :**

```text
Player(Alex, niv 7, 3000 pts)
[Player(Alex, niv 7, 3000 pts), Player(Sam, niv 3, 900 pts)]
Meilleur score : 3000
Attrapé : Exception: Erreur serveur : 404
JSON du serveur invalide : Unexpected end of input (at character 15)
```

Ce programme contient les quatre situations que vous rencontrerez sur toute vraie API :

| Situation | Traitement |
| --- | --- |
| Réponse correcte, objet | `jsonDecode` + `fromJson` |
| Réponse correcte, tableau | `jsonDecode` + `.map(...).toList()` |
| Code HTTP d'erreur | `throw Exception` avant tout décodage |
| Corps illisible | `on FormatException` |

Trois conseils pour la suite, que vous pouvez appliquer dès aujourd'hui.

**1. Vérifiez le code HTTP avant de décoder.** Un serveur en erreur renvoie souvent une page HTML. `jsonDecode('<html>...')` lève une `FormatException` dont le message ne dit rien d'utile, alors que `statusCode == 500` est parfaitement clair.

**2. Ne laissez jamais une `Map` remonter dans l'interface.** La règle de la section 17.18 est encore plus vraie en Flutter : un widget qui écrit `donnees['scroe']` compile sans broncher et affiche du vide. Un widget qui écrit `joueur.scroe` ne compile pas.

**3. Isolez le réseau dans une classe dédiée.** Au chapitre 16, vous avez appris à découper un projet. Le schéma standard en Flutter est le suivant :

```text
  lib/
   ├── models/
   │    ├── player.dart          <- Player, fromJson, toJson
   │    └── item.dart
   ├── services/
   │    └── api_client.dart      <- http.get, statusCode, jsonDecode
   └── ui/
        └── ecran_classement.dart <- n'utilise que des List<Player>
```

Les modèles ne connaissent pas le réseau. L'interface ne connaît pas le JSON. Le service est le seul endroit du projet où le mot `jsonDecode` apparaît.

Ainsi, le jour où vous remplacez le serveur par un fichier local, ou l'inverse, vous ne touchez qu'un seul fichier. C'est le but recherché depuis la section 17.19 : **une frontière nette entre les données brutes et les objets de votre jeu**.

---

## 17.34 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `FormatException: Unexpected character (at character 2)` | Guillemets simples dans le JSON : `{'nom': 'Alex'}` | JSON n'accepte que les guillemets doubles : `{"nom": "Alex"}` |
| `FormatException: Unexpected character` en fin d'objet | Virgule finale : `{"a": 1, }` | Supprimer la virgule qui précède `}` ou `]` |
| `FormatException: Unexpected end of input` | Texte JSON tronqué (accolade non fermée, téléchargement interrompu) | Vérifier que le texte est complet ; entourer d'un `try` / `on FormatException` |
| `FormatException: Unexpected character (at character 1)` sur une réponse serveur | Le serveur a renvoyé du HTML (page d'erreur) et non du JSON | Tester `reponse.statusCode == 200` **avant** d'appeler `jsonDecode` |
| `type 'String' is not a subtype of type 'int' in type cast` | `json['score'] as int` alors que le JSON contient `"3000"` | Passer par une fonction défensive : `int.tryParse('${json['score']}') ?? 0` |
| `type 'int' is not a subtype of type 'double' in type cast` | `json['energie'] as double` alors que le JSON contient `100` | Utiliser `(json['energie'] as num).toDouble()` |
| `type '_Map<String, dynamic>' is not a subtype of type 'List<dynamic>'` | La racine du JSON est un objet, pas un tableau | Vérifier la forme avec `is` avant le cast |
| `type 'List<dynamic>' is not a subtype of type 'List<Map<String, dynamic>>'` | Oubli du `.cast<Map<String, dynamic>>()` après `jsonDecode` | Ajouter le `.cast<...>()` ou passer par `.map(...).toList()` |
| `Converting object to an encodable object failed: Instance of 'Player'` | `jsonEncode` reçoit un objet sans méthode `toJson()` | Ajouter `Map<String, dynamic> toJson()` à la classe |
| `NoSuchMethodError: Class 'int' has no instance method 'toUpperCase'` | Méthode appelée sur une valeur `dynamic` du mauvais type | Caster et typer immédiatement après `jsonDecode` |
| Un champ vaut toujours `null` sans aucune erreur | Faute de frappe sur la clé : `json['scroe']` | Vérifier l'orthographe ; centraliser les clés dans des constantes ; utiliser une classe modèle |
| Un champ ajouté à la classe n'apparaît pas dans la sauvegarde | Oubli de la ligne correspondante dans `toJson()` | Tester l'aller-retour objet → JSON → objet, ou passer à `json_serializable` |
| La date relue est décalée de plusieurs heures | Date écrite en heure locale (sans `Z`) et relue sur une autre machine | Stocker en UTC : `date.toUtc().toIso8601String()` |
| Après une mise à jour, tous les objets ont changé de rareté | Enum sérialisé par son `index`, et l'ordre de l'enum a changé | Sérialiser par `.name`, jamais par `.index` |
| `ArgumentError: Invalid argument (name): No enum value with that name` | `Rarete.values.byName(...)` sur une valeur inconnue | Écrire une fonction de lecture tolérante avec une valeur par défaut |
| `Undefined name '_$PlayerFromJson'` | `json_serializable` utilisé sans avoir lancé la génération | `dart run build_runner build --delete-conflicting-outputs` |
| Le fichier de sauvegarde est vide ou corrompu après un plantage | Écriture directe interrompue en cours de route | Écrire dans un fichier `.tmp` puis le renommer |
| `Unsupported operation: Cannot add to an unmodifiable list` | Liste issue d'un `const` ou d'un `.cast()` modifiée ensuite | Recréer une liste modifiable avec `List.of(...)` ou `.toList()` |

---

## 17.35 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| Sérialisation | Transformer un objet en texte pour le stocker ou l'envoyer |
| Désérialisation | Transformer un texte en objet utilisable |
| JSON | Format texte universel, six types seulement |
| Types JSON | `string`, `number`, `boolean`, `null`, `object`, `array` |
| Objet JSON | Entre `{ }`, paires clé/valeur, clés toujours entre guillemets doubles |
| Tableau JSON | Entre `[ ]`, valeurs ordonnées |
| Guillemets | Doubles obligatoires, jamais de simples |
| Virgule finale | Interdite en JSON, contrairement à Dart |
| Commentaires | Interdits en JSON |
| `dart:convert` | Bibliothèque standard, aucun package à installer |
| `jsonDecode(texte)` | Texte → structure Dart, type de retour `dynamic` |
| Pourquoi `dynamic` | La fonction ne peut pas deviner la forme du JSON avant de l'avoir lu |
| Cast | `as Map<String, dynamic>` pour un objet, `as List<dynamic>` pour un tableau |
| `jsonEncode(valeur)` | Structure Dart → texte |
| `toJson()` | Méthode appelée automatiquement par `jsonEncode` sur vos objets |
| `.cast<T>()` | Retype une liste `dynamic` en liste typée |
| Danger des `Map` | Clés non vérifiées par le compilateur, fautes de frappe silencieuses |
| Classe modèle | Champs typés, vérifiés à la compilation, autocomplétés |
| `fromJson()` | Constructeur nommé (souvent `factory`) : `Map<String, dynamic>` → objet |
| `toJson()` | Méthode d'instance : objet → `Map<String, dynamic>` |
| Aller-retour | `jsonDecode(jsonEncode(obj))` doit redonner un objet identique |
| Liste d'objets | `liste.map((e) => T.fromJson(e as Map<String, dynamic>)).toList()` |
| Objet imbriqué | Chaque sous-objet a son propre `fromJson` / `toJson` |
| Liste imbriquée | `.map(...)` dans les deux sens |
| Valeur absente | `json['x'] as int? ?? valeurParDefaut` |
| Champ vital absent | Lever une `FormatException` explicite |
| Conversion défensive | `lireInt`, `lireDouble`, `lireString`, `lireBool` écrites une fois pour toutes |
| JSON invalide | `try` / `on FormatException` |
| Forme inattendue | Test `is!` avant le cast |
| Enum | Sérialiser par `.name`, jamais par `.index` |
| `DateTime` | `toIso8601String()` / `DateTime.tryParse(...)`, toujours en UTC |
| Sauvegarde fichier | `dart:io`, `writeAsString` / `readAsString`, écriture atomique |
| Champ `version` | Rend les migrations de format possibles |
| `JsonEncoder.withIndent` | JSON indenté et lisible pour un fichier |
| `json_serializable` | Génère `fromJson` / `toJson` quand les modèles sont nombreux |
| API REST | Vérifier `statusCode`, puis `jsonDecode`, puis `fromJson` |
| Architecture | Le JSON ne sort jamais de la couche « service » |

---

## 17.36 — Exercices

Sauf mention contraire, chaque exercice se réalise dans DartPad avec `import 'dart:convert';`.

### Exercice 1 — Réparer un JSON invalide (facile)

Le fichier de configuration suivant est refusé par `jsonDecode` :

```text
{
  'nom': "Alex",
  "niveau": 7,
  "score": 3000,
  "vivant": True,
  "inventaire": ["potion", "clé",],
}
```

1. Trouvez les **quatre** erreurs de syntaxe.
2. Réécrivez le JSON correct.
3. Écrivez un programme Dart qui le décode et affiche le nom, le niveau et le nombre d'objets de l'inventaire.

### Exercice 2 — Décoder un objet JSON (facile)

À partir du texte `'{"nom":"Alex","niveau":7,"score":3000,"vivant":true}'` :

1. Décodez-le avec `jsonDecode`.
2. Castez le résultat en `Map<String, dynamic>`.
3. Affichez chaque valeur avec son type Dart réel (`runtimeType`).

### Exercice 3 — Encoder une fiche d'ennemi (facile)

Créez une `Map<String, dynamic>` décrivant un ennemi :

- `nom` : `'Dragon'`
- `pointsDeVie` : `400`
- `boss` : `true`
- `butin` : la liste `['écaille', 'or', 'gemme']`
- `zone` : `null`

Affichez-la en JSON compact, puis en JSON indenté de deux espaces.

### Exercice 4 — Décoder un tableau JSON (facile)

Soit le texte :

```text
[{"nom":"Gobelin","pv":30},{"nom":"Orc","pv":55},{"nom":"Dragon","pv":400}]
```

1. Décodez-le en `List<dynamic>`.
2. Convertissez-le en `List<Map<String, dynamic>>` avec `.cast<...>()`.
3. Affichez le nombre d'ennemis, la somme de leurs points de vie et le nom du plus résistant.

### Exercice 5 — Valeurs manquantes (moyen)

Écrivez une fonction `String resume(Map<String, dynamic> json)` qui produit une ligne de la forme :

```text
Alex — niveau 7 — 3000 pts — arme : Épée de feu
```

Règles de repli : nom absent → `Anonyme`, niveau absent → `1`, score absent → `0`, arme absente ou `null` → `mains nues`.

Testez-la sur `{}`, sur `{"nom":"Sam"}` et sur un JSON complet.

### Exercice 6 — Première classe modèle (moyen)

Écrivez une classe `Enemy` avec trois champs `final` : `nom` (`String`), `pointsDeVie` (`int`), `boss` (`bool`).

Ajoutez :

- un constructeur avec paramètres nommés `required` ;
- un constructeur `factory Enemy.fromJson(Map<String, dynamic> json)` ;
- une méthode `Map<String, dynamic> toJson()` ;
- un `toString()` lisible.

Décodez `'{"nom":"Dragon","pv":400,"boss":true}'` (attention : la clé JSON est `pv`, pas `pointsDeVie`) et réencodez l'objet obtenu.

### Exercice 7 — Aller-retour sur une liste (moyen)

En reprenant la classe `Enemy` de l'exercice 6 :

1. Décodez le tableau JSON de l'exercice 4 en `List<Enemy>`.
2. Filtrez les ennemis ayant plus de 50 points de vie.
3. Réencodez la liste filtrée avec `jsonEncode`.
4. Vérifiez que `jsonEncode(liste)` puis re-décodage redonne des objets identiques champ par champ.

### Exercice 8 — Objet imbriqué (moyen)

Modélisez ce JSON avec deux classes `Weapon` et `Hero` :

```text
{"nom":"Alex","niveau":7,"arme":{"nom":"Épée de feu","degats":25}}
```

L'arme doit être **facultative** : le champ Dart est `Weapon?`. Testez avec un héros sans arme (`{"nom":"Sam","niveau":1}`) et affichez `mains nues` dans ce cas. Vérifiez l'aller-retour dans les deux situations.

### Exercice 9 — Conversion défensive (moyen)

Un serveur mal configuré renvoie ceci :

```text
{"nom":"Alex","niveau":"7","energie":100,"vivant":"true","score":null}
```

Écrivez les fonctions `lireInt`, `lireDouble`, `lireBool` et `lireString`, puis une classe `Stats` qui lit ce JSON **sans jamais planter** et affiche :

```text
Stats(Alex, niv 7, énergie 100.0, vivant true, score 0)
```

### Exercice 10 — Décodage sûr (moyen)

Écrivez une fonction `Map<String, dynamic>? decoderObjet(String texte)` qui :

- renvoie la `Map` si le texte est un objet JSON valide ;
- renvoie `null` et affiche un message clair si le texte n'est pas du JSON ;
- renvoie `null` et affiche un autre message si le JSON est valide mais n'est pas un objet.

Testez-la sur `'{"nom":"Alex"}'`, `'{nom:Alex}'`, `'[1,2,3]'` et `'42'`.

### Exercice 11 — Enum et date (difficile)

Créez `enum Difficulte { facile, normal, difficile, cauchemar }` et une classe `Session` avec :

- `joueur` (`String`) ;
- `difficulte` (`Difficulte`) ;
- `debut` (`DateTime`, en UTC) ;
- `dureeSecondes` (`int`).

`fromJson` doit tolérer une difficulté inconnue (repli sur `normal`) et une date illisible (repli sur `DateTime.utc(1970)`). `toJson` doit écrire la difficulté par son nom et la date au format ISO 8601.

Testez sur un JSON correct, puis sur `'{"joueur":"Sam","difficulte":"impossible","debut":"hier"}'`.

### Exercice 12 — Mini-projet : sauvegarde de partie (difficile)

Construisez le système de sauvegarde complet de votre jeu, avec trois classes.

**`Item`**

| Champ | Type | Repli si absent |
| --- | --- | --- |
| `nom` | `String` | `'Objet inconnu'` |
| `quantite` | `int` | `1` |
| `rarete` | `Rarete` (enum) | `Rarete.commun` |

**`Player`**

| Champ | Type | Repli si absent |
| --- | --- | --- |
| `nom` | `String` | `'Anonyme'` |
| `niveau` | `int` | `1` |
| `pointsDeVie` | `int` | `100` |
| `inventaire` | `List<Item>` | liste vide |
| `armeEquipee` | `String?` | `null` |

**`SaveGame`**

| Champ | Type | Repli si absent |
| --- | --- | --- |
| `version` | `int` | `1` |
| `joueur` | `Player` | joueur par défaut |
| `dateSauvegarde` | `DateTime` (UTC) | `DateTime.utc(1970)` |
| `tempsDeJeuSecondes` | `int` | `0` |

Contraintes :

1. Les trois classes possèdent `fromJson` **et** `toJson`.
2. Aucune donnée d'entrée, même absurde, ne doit faire planter le chargement.
3. Les entrées non conformes de l'inventaire (par exemple un texte au lieu d'un objet) sont ignorées.
4. `SaveGame` expose un getter `dureeLisible` affichant `2h 3min 5s`.
5. Le `main` doit : créer une partie, l'encoder en JSON indenté, la recharger, vérifier que l'aller-retour est parfait, puis charger une sauvegarde volontairement abîmée.

---

## 17.37 — Corrections des exercices

### Correction 1

Les quatre erreurs :

1. `'nom'` : clé entre guillemets **simples** — interdit en JSON ;
2. `True` : booléen à la mode Python — JSON veut `true` en minuscules ;
3. virgule finale après `"clé"` dans le tableau ;
4. virgule finale après le tableau, avant l'accolade fermante.

JSON corrigé :

```json
{
  "nom": "Alex",
  "niveau": 7,
  "score": 3000,
  "vivant": true,
  "inventaire": ["potion", "clé"]
}
```

Programme de vérification :

```dart
import 'dart:convert';

void main() {
  const texte = '''
{
  "nom": "Alex",
  "niveau": 7,
  "score": 3000,
  "vivant": true,
  "inventaire": ["potion", "clé"]
}
''';

  final config = jsonDecode(texte) as Map<String, dynamic>;
  final inventaire = config['inventaire'] as List<dynamic>;

  print('Nom      : ${config['nom']}');
  print('Niveau   : ${config['niveau']}');
  print('Vivant   : ${config['vivant']}');
  print('Objets   : ${inventaire.length}');
  print('Contenu  : $inventaire');
}
```

**Résultat :**

```text
Nom      : Alex
Niveau   : 7
Vivant   : true
Objets   : 2
Contenu  : [potion, clé]
```

**Explication :** les quatre fautes correspondent aux quatre réflexes que l'on garde de Dart ou d'un autre langage. Dart accepte les guillemets simples et la virgule finale ; JSON, non. Python écrit `True` ; JSON écrit `true`. Le triple guillemet `'''` de Dart permet ici d'écrire le JSON sur plusieurs lignes sans échapper les guillemets doubles, ce qui rend le texte de test lisible. Notez que `config['inventaire']` doit être casté en `List<dynamic>` avant qu'on puisse lui demander `.length` de manière typée.

---

### Correction 2

```dart
import 'dart:convert';

void main() {
  const texte = '{"nom":"Alex","niveau":7,"score":3000,"vivant":true}';

  final brut = jsonDecode(texte);
  print('Est-ce une Map<String, dynamic> ? ${brut is Map<String, dynamic>}');

  final joueur = brut as Map<String, dynamic>;

  joueur.forEach((cle, valeur) {
    print('$cle -> $valeur (${valeur.runtimeType})');
  });

  final String nom = joueur['nom'] as String;
  final int niveau = joueur['niveau'] as int;
  print('Typé : $nom, niveau $niveau');
}
```

**Résultat :**

```text
Est-ce une Map<String, dynamic> ? true
nom -> Alex (String)
niveau -> 7 (int)
score -> 3000 (int)
vivant -> true (bool)
Typé : Alex, niveau 7
```

**Explication :** `jsonDecode` a un type statique `dynamic`, mais l'objet réellement créé est bien une `Map<String, dynamic>` — c'est ce que confirme le test `is`. Chaque valeur, elle, a le type Dart correspondant à son type JSON : `"Alex"` devient un `String`, `7` un `int`, `true` un `bool`. Les deux dernières lignes montrent l'étape essentielle : dès qu'on sort une valeur de la `Map`, on la caste pour retrouver un type vérifié par le compilateur.

---

### Correction 3

```dart
import 'dart:convert';

void main() {
  final dragon = <String, dynamic>{
    'nom': 'Dragon',
    'pointsDeVie': 400,
    'boss': true,
    'butin': ['écaille', 'or', 'gemme'],
    'zone': null,
  };

  print(jsonEncode(dragon));
  print('---');
  print(const JsonEncoder.withIndent('  ').convert(dragon));
}
```

**Résultat :**

```text
{"nom":"Dragon","pointsDeVie":400,"boss":true,"butin":["écaille","or","gemme"],"zone":null}
---
{
  "nom": "Dragon",
  "pointsDeVie": 400,
  "boss": true,
  "butin": [
    "écaille",
    "or",
    "gemme"
  ],
  "zone": null
}
```

**Explication :** `jsonEncode` traduit chaque type Dart vers son équivalent JSON, y compris `null`, qui est un type JSON à part entière et n'est donc pas supprimé de la sortie. Le type `Map<String, dynamic>` est indispensable : avec un `Map<String, Object>`, la valeur `null` serait refusée par le compilateur. La version indentée n'est pas un autre format — c'est le **même JSON**, avec des espaces sans signification ajoutés pour la lecture humaine ; les deux textes sont décodés à l'identique.

---

### Correction 4

```dart
import 'dart:convert';

void main() {
  const texte =
      '[{"nom":"Gobelin","pv":30},{"nom":"Orc","pv":55},{"nom":"Dragon","pv":400}]';

  final brut = jsonDecode(texte) as List<dynamic>;
  final ennemis = brut.cast<Map<String, dynamic>>();

  print('Nombre d\'ennemis : ${ennemis.length}');

  final total = ennemis.fold<int>(0, (somme, e) => somme + (e['pv'] as int));
  print('PV cumulés : $total');

  var plusResistant = ennemis.first;
  for (final e in ennemis) {
    if ((e['pv'] as int) > (plusResistant['pv'] as int)) {
      plusResistant = e;
    }
  }
  print('Plus résistant : ${plusResistant['nom']} (${plusResistant['pv']} PV)');

  final noms = ennemis.map((e) => e['nom'] as String).toList();
  print('Noms : $noms');
}
```

**Résultat :**

```text
Nombre d'ennemis : 3
PV cumulés : 485
Plus résistant : Dragon (400 PV)
Noms : [Gobelin, Orc, Dragon]
```

**Explication :** `jsonDecode` renvoie une `List<dynamic>` car il ne peut pas savoir ce que contient le tableau. Le `.cast<Map<String, dynamic>>()` change le type **statique** de la liste sans recopier les éléments : à partir de là, `e['pv']` est accepté par le compilateur, ce qui ne serait pas le cas sur un élément resté `dynamic` dans un `fold` typé. Chaque `e['pv']` doit malgré tout être casté en `int`, puisque les valeurs de la `Map` sont `dynamic`. Cette accumulation de casts est précisément le problème que la classe modèle de la correction 6 va supprimer.

---

### Correction 5

```dart
import 'dart:convert';

String resume(Map<String, dynamic> json) {
  final nom = json['nom'] as String? ?? 'Anonyme';
  final niveau = json['niveau'] as int? ?? 1;
  final score = json['score'] as int? ?? 0;
  final arme = json['arme'] as String? ?? 'mains nues';

  return '$nom — niveau $niveau — $score pts — arme : $arme';
}

void main() {
  print(resume(jsonDecode('{}') as Map<String, dynamic>));
  print(resume(jsonDecode('{"nom":"Sam"}') as Map<String, dynamic>));
  print(resume(jsonDecode('{"nom":"Lia","arme":null}') as Map<String, dynamic>));
  print(resume(jsonDecode(
          '{"nom":"Alex","niveau":7,"score":3000,"arme":"Épée de feu"}')
      as Map<String, dynamic>));
}
```

**Résultat :**

```text
Anonyme — niveau 1 — 0 pts — arme : mains nues
Sam — niveau 1 — 0 pts — arme : mains nues
Lia — niveau 1 — 0 pts — arme : mains nues
Alex — niveau 7 — 3000 pts — arme : Épée de feu
```

**Explication :** le motif `json['cle'] as Type? ?? defaut` est le plus important du chapitre. Le `?` sur le type est obligatoire : sans lui, `as String` sur une clé absente lèverait immédiatement une erreur de cast, avant même que `??` puisse intervenir. Ce motif traite d'un seul coup les deux cas de la section 17.26 : clé absente et clé présente valant `null` — la troisième ligne du résultat le prouve. Remarquez enfin que `resume` ne connaît rien du format : elle reçoit une `Map` déjà décodée, ce qui la rend testable sans passer par du texte JSON.

---

### Correction 6

```dart
import 'dart:convert';

class Enemy {
  final String nom;
  final int pointsDeVie;
  final bool boss;

  Enemy({
    required this.nom,
    required this.pointsDeVie,
    required this.boss,
  });

  factory Enemy.fromJson(Map<String, dynamic> json) => Enemy(
        nom: json['nom'] as String? ?? 'Ennemi',
        pointsDeVie: json['pv'] as int? ?? 1,
        boss: json['boss'] as bool? ?? false,
      );

  Map<String, dynamic> toJson() => {
        'nom': nom,
        'pv': pointsDeVie,
        'boss': boss,
      };

  @override
  String toString() => 'Enemy($nom, $pointsDeVie PV${boss ? ", BOSS" : ""})';
}

void main() {
  final dragon = Enemy.fromJson(
      jsonDecode('{"nom":"Dragon","pv":400,"boss":true}')
          as Map<String, dynamic>);

  print(dragon);
  print(dragon.pointsDeVie + 100);
  print(jsonEncode(dragon));

  final gobelin = Enemy.fromJson(
      jsonDecode('{"nom":"Gobelin","pv":30}') as Map<String, dynamic>);

  print(gobelin);
  print(jsonEncode(gobelin));
}
```

**Résultat :**

```text
Enemy(Dragon, 400 PV, BOSS)
500
{"nom":"Dragon","pv":400,"boss":true}
Enemy(Gobelin, 30 PV)
{"nom":"Gobelin","pv":30,"boss":false}
```

**Explication :** la classe fait la traduction entre deux vocabulaires. Côté JSON la clé s'appelle `pv` ; côté Dart le champ s'appelle `pointsDeVie`. Cette correspondance est écrite **à deux endroits seulement** — dans `fromJson` et dans `toJson` — au lieu d'être disséminée dans tout le programme. Une fois l'objet construit, `dragon.pointsDeVie + 100` est une addition d'entiers vérifiée par le compilateur, sans cast : comparez avec la correction 4, où chaque lecture exigeait un `as int`. Enfin, `jsonEncode(dragon)` fonctionne sans appeler explicitement `toJson()` : `jsonEncode` cherche cette méthode sur tout objet qu'il ne sait pas convertir.

---

### Correction 7

```dart
import 'dart:convert';

class Enemy {
  final String nom;
  final int pointsDeVie;
  final bool boss;

  Enemy({
    required this.nom,
    required this.pointsDeVie,
    required this.boss,
  });

  factory Enemy.fromJson(Map<String, dynamic> json) => Enemy(
        nom: json['nom'] as String? ?? 'Ennemi',
        pointsDeVie: json['pv'] as int? ?? 1,
        boss: json['boss'] as bool? ?? false,
      );

  Map<String, dynamic> toJson() => {
        'nom': nom,
        'pv': pointsDeVie,
        'boss': boss,
      };

  static List<Enemy> listFromJson(List<dynamic> liste) =>
      liste.map((e) => Enemy.fromJson(e as Map<String, dynamic>)).toList();

  @override
  String toString() => 'Enemy($nom, $pointsDeVie PV)';
}

void main() {
  const texte =
      '[{"nom":"Gobelin","pv":30},{"nom":"Orc","pv":55},{"nom":"Dragon","pv":400}]';

  final ennemis = Enemy.listFromJson(jsonDecode(texte) as List<dynamic>);
  print(ennemis);

  final costauds = ennemis.where((e) => e.pointsDeVie > 50).toList();
  print(costauds);

  final texteFiltre = jsonEncode(costauds);
  print(texteFiltre);

  final relus = Enemy.listFromJson(jsonDecode(texteFiltre) as List<dynamic>);

  var identiques = relus.length == costauds.length;
  for (var i = 0; i < relus.length; i++) {
    identiques = identiques &&
        relus[i].nom == costauds[i].nom &&
        relus[i].pointsDeVie == costauds[i].pointsDeVie &&
        relus[i].boss == costauds[i].boss;
  }
  print('Aller-retour fidèle ? $identiques');
}
```

**Résultat :**

```text
[Enemy(Gobelin, 30 PV), Enemy(Orc, 55 PV), Enemy(Dragon, 400 PV)]
[Enemy(Orc, 55 PV), Enemy(Dragon, 400 PV)]
[{"nom":"Orc","pv":55,"boss":false},{"nom":"Dragon","pv":400,"boss":false}]
Aller-retour fidèle ? true
```

**Explication :** trois points méritent l'attention. D'abord, `listFromJson` est une méthode `static` : elle appartient à la classe et non à une instance, ce qui est logique puisqu'elle sert justement à créer des instances. Ensuite, `jsonEncode(costauds)` reçoit une `List<Enemy>` et non une `Map` : `jsonEncode` parcourt la liste et appelle `toJson()` sur chaque élément, ce qui donne un tableau JSON. Enfin, la comparaison finale se fait champ par champ. Écrire `relus == costauds` renverrait `false` : sans redéfinition de `==` (chapitre 10), Dart compare l'identité des objets, et les objets relus sont bien de nouveaux objets en mémoire, même s'ils portent les mêmes valeurs.

---

### Correction 8

```dart
import 'dart:convert';

class Weapon {
  final String nom;
  final int degats;

  Weapon({required this.nom, required this.degats});

  factory Weapon.fromJson(Map<String, dynamic> json) => Weapon(
        nom: json['nom'] as String? ?? 'Arme',
        degats: json['degats'] as int? ?? 0,
      );

  Map<String, dynamic> toJson() => {'nom': nom, 'degats': degats};

  @override
  String toString() => '$nom (+$degats)';
}

class Hero {
  final String nom;
  final int niveau;
  final Weapon? arme;

  Hero({required this.nom, required this.niveau, this.arme});

  factory Hero.fromJson(Map<String, dynamic> json) {
    final brut = json['arme'];
    return Hero(
      nom: json['nom'] as String? ?? 'Anonyme',
      niveau: json['niveau'] as int? ?? 1,
      arme: brut is Map<String, dynamic> ? Weapon.fromJson(brut) : null,
    );
  }

  Map<String, dynamic> toJson() => {
        'nom': nom,
        'niveau': niveau,
        'arme': arme?.toJson(),
      };

  @override
  String toString() =>
      'Hero($nom, niv $niveau, arme : ${arme ?? "mains nues"})';
}

void main() {
  const avecArme =
      '{"nom":"Alex","niveau":7,"arme":{"nom":"Épée de feu","degats":25}}';
  const sansArme = '{"nom":"Sam","niveau":1}';

  final alex = Hero.fromJson(jsonDecode(avecArme) as Map<String, dynamic>);
  final sam = Hero.fromJson(jsonDecode(sansArme) as Map<String, dynamic>);

  print(alex);
  print(sam);
  print('Dégâts d\'Alex : ${alex.arme?.degats ?? 0}');
  print('Dégâts de Sam  : ${sam.arme?.degats ?? 0}');

  print(jsonEncode(alex));
  print(jsonEncode(sam));

  final alexRelu =
      Hero.fromJson(jsonDecode(jsonEncode(alex)) as Map<String, dynamic>);
  final samRelu =
      Hero.fromJson(jsonDecode(jsonEncode(sam)) as Map<String, dynamic>);

  print(alexRelu);
  print(samRelu);
}
```

**Résultat :**

```text
Hero(Alex, niv 7, arme : Épée de feu (+25))
Hero(Sam, niv 1, arme : mains nues)
Dégâts d'Alex : 25
Dégâts de Sam  : 0
{"nom":"Alex","niveau":7,"arme":{"nom":"Épée de feu","degats":25}}
{"nom":"Sam","niveau":1,"arme":null}
Hero(Alex, niv 7, arme : Épée de feu (+25))
Hero(Sam, niv 1, arme : mains nues)
```

**Explication :** le test `brut is Map<String, dynamic>` est plus robuste qu'un simple `brut == null`. Il couvre trois cas d'un coup : clé absente, clé valant `null`, et clé contenant autre chose qu'un objet (un texte, par exemple). Côté écriture, `arme?.toJson()` produit `null` si l'arme est absente ; le champ apparaît alors dans le JSON avec la valeur `null`, ce qui est parfaitement valide et se relit sans problème, comme le montrent les deux dernières lignes. La chaîne d'accès `alex.arme?.degats ?? 0` combine deux outils du chapitre 12 : `?.` pour ne pas planter si l'arme est absente, `??` pour fournir une valeur de repli à la fin.

---

### Correction 9

```dart
import 'dart:convert';

int lireInt(dynamic valeur, {int defaut = 0}) {
  if (valeur is int) return valeur;
  if (valeur is double) return valeur.toInt();
  if (valeur is String) return int.tryParse(valeur) ?? defaut;
  return defaut;
}

double lireDouble(dynamic valeur, {double defaut = 0.0}) {
  if (valeur is num) return valeur.toDouble();
  if (valeur is String) return double.tryParse(valeur) ?? defaut;
  return defaut;
}

bool lireBool(dynamic valeur, {bool defaut = false}) {
  if (valeur is bool) return valeur;
  if (valeur is String) return valeur.toLowerCase() == 'true';
  if (valeur is num) return valeur != 0;
  return defaut;
}

String lireString(dynamic valeur, {String defaut = ''}) {
  if (valeur is String) return valeur;
  if (valeur == null) return defaut;
  return valeur.toString();
}

class Stats {
  final String nom;
  final int niveau;
  final double energie;
  final bool vivant;
  final int score;

  Stats({
    required this.nom,
    required this.niveau,
    required this.energie,
    required this.vivant,
    required this.score,
  });

  factory Stats.fromJson(Map<String, dynamic> json) => Stats(
        nom: lireString(json['nom'], defaut: 'Anonyme'),
        niveau: lireInt(json['niveau'], defaut: 1),
        energie: lireDouble(json['energie'], defaut: 100.0),
        vivant: lireBool(json['vivant'], defaut: true),
        score: lireInt(json['score']),
      );

  @override
  String toString() =>
      'Stats($nom, niv $niveau, énergie $energie, vivant $vivant, score $score)';
}

void main() {
  const texte =
      '{"nom":"Alex","niveau":"7","energie":100,"vivant":"true","score":null}';

  print(Stats.fromJson(jsonDecode(texte) as Map<String, dynamic>));
  print(Stats.fromJson(jsonDecode('{}') as Map<String, dynamic>));
  print(Stats.fromJson(jsonDecode(
          '{"nom":42,"niveau":"abc","energie":"87.5","vivant":0}')
      as Map<String, dynamic>));
}
```

**Résultat :**

```text
Stats(Alex, niv 7, énergie 100.0, vivant true, score 0)
Stats(Anonyme, niv 1, énergie 100.0, vivant true, score 0)
Stats(42, niv 1, énergie 87.5, vivant false, score 0)
```

**Explication :** ces quatre fonctions valent tout un fichier utilitaire dans un vrai projet. Chacune suit le même schéma : on teste les types plausibles du plus probable au moins probable, et on termine par un repli. `lireDouble` mérite une mention particulière : son premier test porte sur `num`, le type parent commun à `int` et `double`. C'est ce qui permet au JSON `100` — décodé en `int` — d'alimenter un champ `double` sans le moindre cast raté. La troisième ligne du résultat montre le comportement sur des données franchement mauvaises : `42` devient le texte `"42"`, `"abc"` retombe sur `1`, `"87.5"` est bien converti, et `0` est interprété comme `false` selon la convention historique des langages C. Aucun appel n'a levé d'exception.

---

### Correction 10

```dart
import 'dart:convert';

Map<String, dynamic>? decoderObjet(String texte) {
  try {
    final brut = jsonDecode(texte);

    if (brut is! Map<String, dynamic>) {
      print('JSON valide, mais ce n\'est pas un objet : ${brut.runtimeType}');
      return null;
    }

    return brut;
  } on FormatException catch (e) {
    print('Texte JSON invalide : ${e.message}');
    return null;
  }
}

void main() {
  print(decoderObjet('{"nom":"Alex"}'));
  print('---');
  print(decoderObjet('{nom:Alex}'));
  print('---');
  print(decoderObjet('[1,2,3]'));
  print('---');
  print(decoderObjet('42'));
}
```

**Résultat :**

```text
{nom: Alex}
---
Texte JSON invalide : Unexpected character
null
---
JSON valide, mais ce n'est pas un objet : List<dynamic>
null
---
JSON valide, mais ce n'est pas un objet : int
null
```

**Explication :** cette fonction distingue deux échecs de nature différente, comme expliqué à la section 17.28. Un texte qui n'est pas du JSON provoque une **exception** : il faut un `try` / `on FormatException`. Un texte qui est du JSON valide mais d'une autre forme ne provoque **aucune** exception : `'[1,2,3]'` et `'42'` sont des documents JSON tout à fait légaux, seulement ce ne sont pas des objets. Seul un test `is!` permet de les repérer. Notez le choix du type de retour `Map<String, dynamic>?` : il oblige l'appelant à traiter le cas d'échec, alors qu'un `throw` l'aurait laissé libre de l'ignorer. À vous de choisir selon que l'échec est prévisible — ici, oui — ou anormal.

---

### Correction 11

```dart
import 'dart:convert';

enum Difficulte { facile, normal, difficile, cauchemar }

Difficulte difficulteDepuisTexte(dynamic valeur,
    {Difficulte defaut = Difficulte.normal}) {
  if (valeur is String) {
    for (final d in Difficulte.values) {
      if (d.name == valeur) return d;
    }
  }
  return defaut;
}

DateTime dateDepuisTexte(dynamic valeur) {
  if (valeur is String) {
    final d = DateTime.tryParse(valeur);
    if (d != null) return d.toUtc();
  }
  return DateTime.utc(1970);
}

class Session {
  final String joueur;
  final Difficulte difficulte;
  final DateTime debut;
  final int dureeSecondes;

  Session({
    required this.joueur,
    required this.difficulte,
    required this.debut,
    required this.dureeSecondes,
  });

  factory Session.fromJson(Map<String, dynamic> json) => Session(
        joueur: json['joueur'] as String? ?? 'Anonyme',
        difficulte: difficulteDepuisTexte(json['difficulte']),
        debut: dateDepuisTexte(json['debut']),
        dureeSecondes: json['dureeSecondes'] as int? ?? 0,
      );

  Map<String, dynamic> toJson() => {
        'joueur': joueur,
        'difficulte': difficulte.name,
        'debut': debut.toIso8601String(),
        'dureeSecondes': dureeSecondes,
      };

  @override
  String toString() => 'Session($joueur, ${difficulte.name}, '
      'début ${debut.toIso8601String()}, ${dureeSecondes}s)';
}

void main() {
  const correct = '{"joueur":"Alex","difficulte":"cauchemar",'
      '"debut":"2026-08-08T14:30:00.000Z","dureeSecondes":3600}';

  final s = Session.fromJson(jsonDecode(correct) as Map<String, dynamic>);
  print(s);
  print(jsonEncode(s));

  final relue =
      Session.fromJson(jsonDecode(jsonEncode(s)) as Map<String, dynamic>);

  final fidele = relue.joueur == s.joueur &&
      relue.difficulte == s.difficulte &&
      relue.debut == s.debut &&
      relue.dureeSecondes == s.dureeSecondes;
  print('Aller-retour fidèle ? $fidele');

  const abimee = '{"joueur":"Sam","difficulte":"impossible","debut":"hier"}';
  print(Session.fromJson(jsonDecode(abimee) as Map<String, dynamic>));
}
```

**Résultat :**

```text
Session(Alex, cauchemar, début 2026-08-08T14:30:00.000Z, 3600s)
{"joueur":"Alex","difficulte":"cauchemar","debut":"2026-08-08T14:30:00.000Z","dureeSecondes":3600}
Aller-retour fidèle ? true
Session(Sam, normal, début 1970-01-01T00:00:00.000Z, 0s)
```

**Explication :** les deux fonctions de lecture appliquent la même philosophie à deux types que JSON ne connaît pas. `difficulteDepuisTexte` parcourt `Difficulte.values` au lieu d'appeler `byName`, précisément pour ne pas lever d'`ArgumentError` sur une valeur inconnue. `dateDepuisTexte` utilise `tryParse` plutôt que `parse`, pour la même raison. Le `.toUtc()` garantit que toute date en mémoire est en temps universel, quelle que soit la façon dont elle a été écrite dans le fichier — sans lui, une sauvegarde faite à Paris et relue à Tokyo décalerait toutes les sessions. Enfin, `relue.debut == s.debut` renvoie `true` parce que `DateTime` redéfinit `==` pour comparer l'instant, contrairement à vos propres classes.

---

### Correction 12

```dart
import 'dart:convert';

enum Rarete { commun, rare, epique, legendaire }

Rarete rareteDepuisTexte(dynamic valeur, {Rarete defaut = Rarete.commun}) {
  if (valeur is String) {
    for (final r in Rarete.values) {
      if (r.name == valeur) return r;
    }
  }
  return defaut;
}

int lireInt(dynamic valeur, {int defaut = 0}) {
  if (valeur is int) return valeur;
  if (valeur is double) return valeur.toInt();
  if (valeur is String) return int.tryParse(valeur) ?? defaut;
  return defaut;
}

String lireString(dynamic valeur, {String defaut = ''}) {
  if (valeur is String) return valeur;
  if (valeur == null) return defaut;
  return valeur.toString();
}

DateTime lireDate(dynamic valeur) {
  if (valeur is String) {
    final d = DateTime.tryParse(valeur);
    if (d != null) return d.toUtc();
  }
  return DateTime.utc(1970);
}

class Item {
  final String nom;
  final int quantite;
  final Rarete rarete;

  Item({required this.nom, required this.quantite, required this.rarete});

  factory Item.fromJson(Map<String, dynamic> json) => Item(
        nom: lireString(json['nom'], defaut: 'Objet inconnu'),
        quantite: lireInt(json['quantite'], defaut: 1),
        rarete: rareteDepuisTexte(json['rarete']),
      );

  Map<String, dynamic> toJson() => {
        'nom': nom,
        'quantite': quantite,
        'rarete': rarete.name,
      };

  @override
  String toString() => '$nom x$quantite [${rarete.name}]';
}

class Player {
  final String nom;
  final int niveau;
  final int pointsDeVie;
  final List<Item> inventaire;
  final String? armeEquipee;

  Player({
    required this.nom,
    required this.niveau,
    required this.pointsDeVie,
    required this.inventaire,
    this.armeEquipee,
  });

  factory Player.fromJson(Map<String, dynamic> json) {
    final brut = json['inventaire'] as List<dynamic>? ?? const [];

    return Player(
      nom: lireString(json['nom'], defaut: 'Anonyme'),
      niveau: lireInt(json['niveau'], defaut: 1),
      pointsDeVie: lireInt(json['pointsDeVie'], defaut: 100),
      inventaire: brut
          .whereType<Map<String, dynamic>>()
          .map((e) => Item.fromJson(e))
          .toList(),
      armeEquipee: json['armeEquipee'] as String?,
    );
  }

  Map<String, dynamic> toJson() => {
        'nom': nom,
        'niveau': niveau,
        'pointsDeVie': pointsDeVie,
        'armeEquipee': armeEquipee,
        'inventaire': inventaire.map((i) => i.toJson()).toList(),
      };

  @override
  String toString() => 'Player($nom, niv $niveau, $pointsDeVie PV, '
      'arme : ${armeEquipee ?? "mains nues"}, sac : $inventaire)';
}

class SaveGame {
  final int version;
  final Player joueur;
  final DateTime dateSauvegarde;
  final int tempsDeJeuSecondes;

  SaveGame({
    required this.version,
    required this.joueur,
    required this.dateSauvegarde,
    required this.tempsDeJeuSecondes,
  });

  factory SaveGame.fromJson(Map<String, dynamic> json) => SaveGame(
        version: lireInt(json['version'], defaut: 1),
        joueur: Player.fromJson(
            json['joueur'] as Map<String, dynamic>? ?? <String, dynamic>{}),
        dateSauvegarde: lireDate(json['dateSauvegarde']),
        tempsDeJeuSecondes: lireInt(json['tempsDeJeuSecondes']),
      );

  Map<String, dynamic> toJson() => {
        'version': version,
        'joueur': joueur.toJson(),
        'dateSauvegarde': dateSauvegarde.toIso8601String(),
        'tempsDeJeuSecondes': tempsDeJeuSecondes,
      };

  String get dureeLisible {
    final h = tempsDeJeuSecondes ~/ 3600;
    final m = (tempsDeJeuSecondes % 3600) ~/ 60;
    final s = tempsDeJeuSecondes % 60;
    return '${h}h ${m}min ${s}s';
  }

  @override
  String toString() =>
      'SaveGame(v$version, ${dateSauvegarde.toIso8601String()}, '
      'temps $dureeLisible)\n  $joueur';
}

void main() {
  final partie = SaveGame(
    version: 1,
    joueur: Player(
      nom: 'Alex',
      niveau: 7,
      pointsDeVie: 85,
      armeEquipee: 'Épée de feu',
      inventaire: [
        Item(nom: 'Potion', quantite: 3, rarete: Rarete.commun),
        Item(nom: 'Clé rouillée', quantite: 1, rarete: Rarete.rare),
        Item(nom: 'Relique', quantite: 1, rarete: Rarete.legendaire),
      ],
    ),
    dateSauvegarde: DateTime.utc(2026, 8, 8, 14, 30),
    tempsDeJeuSecondes: 7385,
  );

  final texte = const JsonEncoder.withIndent('  ').convert(partie.toJson());
  print(texte);

  final rechargee =
      SaveGame.fromJson(jsonDecode(texte) as Map<String, dynamic>);

  print('');
  print(rechargee);
  print('Aller-retour parfait ? ${jsonEncode(rechargee) == jsonEncode(partie)}');

  print('');
  const abimee = '{"version":"2","joueur":{"nom":"Sam","niveau":"3",'
      '"inventaire":[{"nom":"Torche"},"pas un objet",'
      '{"nom":"Gemme","rarete":"mythique"}]},"tempsDeJeuSecondes":null}';

  print(SaveGame.fromJson(jsonDecode(abimee) as Map<String, dynamic>));

  print('');
  print(SaveGame.fromJson(jsonDecode('{}') as Map<String, dynamic>));
}
```

**Résultat :**

```text
{
  "version": 1,
  "joueur": {
    "nom": "Alex",
    "niveau": 7,
    "pointsDeVie": 85,
    "armeEquipee": "Épée de feu",
    "inventaire": [
      {
        "nom": "Potion",
        "quantite": 3,
        "rarete": "commun"
      },
      {
        "nom": "Clé rouillée",
        "quantite": 1,
        "rarete": "rare"
      },
      {
        "nom": "Relique",
        "quantite": 1,
        "rarete": "legendaire"
      }
    ]
  },
  "dateSauvegarde": "2026-08-08T14:30:00.000Z",
  "tempsDeJeuSecondes": 7385
}

SaveGame(v1, 2026-08-08T14:30:00.000Z, temps 2h 3min 5s)
  Player(Alex, niv 7, 85 PV, arme : Épée de feu, sac : [Potion x3 [commun], Clé rouillée x1 [rare], Relique x1 [legendaire]])
Aller-retour parfait ? true

SaveGame(v2, 1970-01-01T00:00:00.000Z, temps 0h 0min 0s)
  Player(Sam, niv 3, 100 PV, arme : mains nues, sac : [Torche x1 [commun], Gemme x1 [commun]])

SaveGame(v1, 1970-01-01T00:00:00.000Z, temps 0h 0min 0s)
  Player(Anonyme, niv 1, 100 PV, arme : mains nues, sac : [])
```

**Explication :** ce programme réunit tout le chapitre. Sept mécanismes y travaillent ensemble.

**1. Trois niveaux d'imbrication.** `SaveGame` contient un `Player`, qui contient une `List<Item>`. Chaque classe ne s'occupe que d'elle-même : `SaveGame.fromJson` appelle `Player.fromJson`, qui appelle `Item.fromJson`. Le même enchaînement se produit en sens inverse dans les `toJson`. C'est la règle de la section 17.24, appliquée deux fois de suite.

**2. Les fonctions de lecture partagées.** `lireInt`, `lireString`, `lireDate` et `rareteDepuisTexte` sont écrites une fois et utilisées par les trois classes. C'est ce qui permet à `"version":"2"` et `"niveau":"3"` — des nombres envoyés sous forme de texte — d'être lus correctement, comme le montre la deuxième sauvegarde du résultat.

**3. `whereType<Map<String, dynamic>>()`.** C'est la réponse à la contrainte numéro 3. L'inventaire abîmé contient trois entrées, dont `"pas un objet"` qui est un simple texte. `whereType` ne conserve que les éléments du type demandé : la chaîne est ignorée, et les deux vrais objets sont chargés. Sans cette ligne, le `.map((e) => Item.fromJson(e as Map<String, dynamic>))` aurait levé un `TypeError` et fait perdre toute la sauvegarde pour un seul élément fautif.

**4. `"rarete":"mythique"`.** Cette valeur n'existe pas dans l'enum. `rareteDepuisTexte` la ramène à `Rarete.commun` au lieu de lever une `ArgumentError`. La Gemme perd sa rareté, mais le joueur garde sa partie.

**5. La date en UTC.** `toIso8601String()` à l'écriture, `DateTime.tryParse(...).toUtc()` à la lecture. Le repli `DateTime.utc(1970)` rend visible une date absente ou illisible : une partie affichée comme sauvegardée en 1970 signale immédiatement le problème, alors qu'un `DateTime.now()` de repli l'aurait masqué.

**6. La vérification de l'aller-retour.** `jsonEncode(rechargee) == jsonEncode(partie)` compare deux textes JSON. C'est le test le plus simple et le plus efficace pour vérifier qu'aucun champ n'a été oublié dans `toJson` : si vous ajoutez un champ à `Player` sans l'ajouter à `toJson`, ce test passe à `false` immédiatement. Placez-le dans un test unitaire, comme vous avez appris à en écrire au chapitre 16.

**7. Le cas `{}`.** La troisième sortie montre un chargement à partir d'un JSON entièrement vide. Aucun champ n'est présent, aucune exception n'est levée, et l'on obtient une partie neuve parfaitement cohérente. C'est la contrainte numéro 2 : **une sauvegarde abîmée ne doit jamais empêcher le jeu de démarrer.**

Pour transformer ce programme en véritable système de sauvegarde, il ne reste qu'à y brancher les deux fonctions de la section 17.31 :

```dart
Future<void> sauvegarder(String chemin, SaveGame partie) async {
  final temporaire = File('$chemin.tmp');
  await temporaire
      .writeAsString(const JsonEncoder.withIndent('  ').convert(partie.toJson()));
  await temporaire.rename(chemin);
}

Future<SaveGame?> charger(String chemin) async {
  final fichier = File(chemin);
  if (!await fichier.exists()) return null;
  try {
    final brut = jsonDecode(await fichier.readAsString());
    if (brut is! Map<String, dynamic>) return null;
    return SaveGame.fromJson(brut);
  } on FormatException catch (e) {
    print('Sauvegarde illisible : ${e.message}');
    return null;
  }
}
```

Avec `import 'dart:io';` en plus, et dans un vrai projet Dart plutôt que dans DartPad, votre jeu sait désormais survivre à sa propre fermeture.

---

## Et maintenant ?

Vous venez de franchir la dernière étape technique de la partie Dart.

Faites le compte de ce que vous savez faire. Vous savez déclarer des variables et des types (chapitres 2 et 3), écrire des conditions et des boucles (4 et 5), manipuler des collections (6), découper votre code en fonctions (7), modéliser un monde en objets (8 à 11), écrire du code qui ne plante pas sur une valeur absente (12), attraper les erreurs (13), transformer des collections avec `map`, `where` et `fold` (14), attendre une opération longue sans bloquer le programme (15), organiser un vrai projet avec ses packages et ses tests (16), et maintenant faire entrer et sortir vos données du programme (17).

Autrement dit : vous avez tout ce qu'il faut pour écrire une application complète.

C'est exactement ce que propose le chapitre suivant. Pas de nouvelle notion, pas de nouvelle syntaxe : un **mini-projet final** qui rassemble les dix-sept chapitres précédents en un seul programme Dart cohérent — un jeu console avec ses personnages, son inventaire, ses combats, sa gestion d'erreurs, ses tests et sa sauvegarde JSON.

C'est le dernier chapitre avant Flutter, et le meilleur moyen de vérifier que tout est en place.

Rendez-vous au chapitre 18 : [18-PARTIE-1A—MINI-PROJET-FINAL-DART.md](18-PARTIE-1A—MINI-PROJET-FINAL-DART.md)
