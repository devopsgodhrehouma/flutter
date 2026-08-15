# PARTIE 1B — FLUTTER
# CHAPITRE 54 — STOCKAGE LOCAL ET PERSISTANCE

> **Niveau :** intermédiaire
> **Durée estimée :** 10 h
> **Pré-requis :** chapitres 43 à 53 (toute la PARTIE 1B) ; chapitres 08 à 17 de la PARTIE 1A (POO, null safety, exceptions, asynchrone, JSON)
> **Ce que vous saurez faire à la fin :** choisir la bonne famille de stockage pour chaque donnée, écrire et relire des préférences, des fichiers et une vraie base de données SQL sur l'appareil, et faire survivre l'état de votre application à sa fermeture.

---

## 54.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer pourquoi une variable Dart disparaît quand l'application se ferme ;
- distinguer la mémoire vive, le disque de l'appareil et le serveur distant ;
- choisir entre les quatre familles de stockage local à l'aide d'un tableau de décision ;
- installer et utiliser `shared_preferences` pour des réglages simples ;
- expliquer pourquoi `SharedPreferences.getInstance()` renvoie un `Future` ;
- citer les cinq types supportés par `shared_preferences` et contourner les autres ;
- écrire, lire, supprimer une clé, et tout effacer ;
- appliquer une valeur par défaut correcte avec l'opérateur `??` du chapitre 12 ;
- dire où sont physiquement stockées ces données sur Android, iOS et le Web ;
- énumérer ce qu'il ne faut jamais mettre dans `shared_preferences` ;
- écrire un service de préférences testable et typé ;
- persister le choix de thème du chapitre 51 entre deux lancements ;
- sérialiser un objet Dart en JSON pour le stocker (rappel du chapitre 17) ;
- obtenir les dossiers de l'application avec `path_provider` ;
- choisir entre documents, support, cache et temporaire en connaissance de cause ;
- écrire et relire un fichier avec `dart:io` ;
- gérer proprement l'absence d'un fichier avec les exceptions du chapitre 13 ;
- sauvegarder une liste d'objets dans un fichier JSON ;
- expliquer les trois limites du fichier unique ;
- ouvrir une base SQLite avec `sqflite` ;
- écrire un `onCreate` et une migration `onUpgrade` versionnée ;
- créer une table, insérer, interroger, mettre à jour et supprimer ;
- écrire des conditions paramétrées et éviter l'injection SQL ;
- convertir une ligne SQL en objet Dart et réciproquement ;
- structurer l'accès aux données avec le patron DAO ;
- garantir l'atomicité d'un groupe d'écritures avec une transaction ;
- adapter votre stratégie de stockage au Web ;
- ranger un jeton d'authentification dans `flutter_secure_storage` ;
- situer `hive`, `isar` et `drift` par rapport à `sqflite` ;
- implémenter une stratégie de cache « réseau d'abord, cache ensuite » ;
- afficher des données périmées plutôt qu'un écran vide ;
- tester une couche de persistance sans lancer d'émulateur ;
- dresser le bilan complet de la PARTIE 1B ;
- construire une liste de tâches persistante, d'abord en `shared_preferences`, puis en `sqflite`.

---

## 54.1 — Pourquoi les données disparaissent quand on ferme l'application

Reprenons un compteur du chapitre 45.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const ApplicationScore());

class ApplicationScore extends StatelessWidget {
  const ApplicationScore({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Score',
      theme: ThemeData(
        colorSchemeSeed: Colors.indigo,
        useMaterial3: true,
      ),
      home: const PageScore(),
    );
  }
}

class PageScore extends StatefulWidget {
  const PageScore({super.key});

  @override
  State<PageScore> createState() => _PageScoreState();
}

class _PageScoreState extends State<PageScore> {
  int _score = 0;

  void _marquer() {
    setState(() {
      _score += 10;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Score du joueur')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
              '$_score',
              style: Theme.of(context).textTheme.displayLarge,
            ),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: _marquer,
              child: const Text('Marquer 10 points'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :** vous appuyez cinq fois, l'écran affiche `50`. Vous fermez l'application, vous la relancez : l'écran affiche `0`.

Ce n'est pas un bug. C'est le fonctionnement normal d'un programme.

La variable `_score` est une **case en mémoire vive**. La mémoire vive appartient au processus. Quand le système d'exploitation termine le processus, il récupère toute cette mémoire pour la donner à une autre application. Vos 50 points n'ont jamais existé ailleurs que dans cette case.

```text
    LANCEMENT                  UTILISATION                 FERMETURE
    ─────────                  ───────────                 ─────────

  ┌───────────────┐          ┌───────────────┐          ┌───────────────┐
  │ Mémoire vive  │          │ Mémoire vive  │          │ Mémoire vive  │
  │               │          │               │          │               │
  │  _score = 0   │  ──────► │  _score = 50  │  ──────► │   (libérée)   │
  │               │          │               │          │               │
  └───────────────┘          └───────────────┘          └───────────────┘
                                                                │
                                                                v
                                                        rien n'a été écrit
                                                          sur le disque
```

**Persister** une donnée, c'est l'écrire quelque part qui survit à la fin du processus. Il y a exactement trois endroits possibles :

| Endroit | Survit à la fermeture ? | Survit à la désinstallation ? | Accessible hors ligne ? |
| --- | --- | --- | --- |
| Mémoire vive (variables Dart) | Non | Non | — |
| Disque de l'appareil (stockage local) | Oui | Non | Oui |
| Serveur distant (API du chapitre 53) | Oui | Oui | Non |

Le chapitre 53 vous a appris le troisième. Ce chapitre vous apprend le deuxième.

> Retenez la phrase suivante : **tant que vous n'avez pas écrit sur le disque, vous n'avez rien sauvegardé.**

---

## 54.1.1 — Le cas particulier du hot reload

Une confusion classique du débutant : « mon score se conserve pendant le développement, donc c'est persisté ».

Non. Le **hot reload** (chapitre 43) réinjecte votre code modifié dans un processus **qui tourne toujours**. La mémoire vive n'est pas vidée, donc `_score` garde sa valeur. Ce n'est pas de la persistance, c'est simplement le même processus qui continue.

| Action | La mémoire est-elle conservée ? |
| --- | --- |
| Hot reload (`r`) | Oui, l'état survit |
| Hot restart (`R`) | Non, l'application repart de `main()` |
| Fermeture puis relancement | Non |
| Le système tue l'application en arrière-plan | Non |

Pour tester une persistance, faites toujours un **hot restart**, jamais un hot reload. Sinon vous testez une illusion.

---

## 54.2 — Les quatre familles de stockage local (tableau de décision)

Sur l'appareil, quatre familles de solutions coexistent. Elles ne sont pas concurrentes : elles répondent à des besoins différents.

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │                     STOCKAGE LOCAL                               │
  ├──────────────────────────────────────────────────────────────────┤
  │                                                                  │
  │  1. CLÉ-VALEUR          « thème = sombre »                       │
  │     shared_preferences   petites données de réglage              │
  │                                                                  │
  │  2. FICHIERS            un .json, un .txt, une image téléchargée │
  │     path_provider        données volumineuses, format libre      │
  │        + dart:io                                                 │
  │                                                                  │
  │  3. BASE DE DONNÉES     1 200 objets, recherches, tris, filtres  │
  │     sqflite / drift      données structurées et nombreuses       │
  │     hive / isar                                                  │
  │                                                                  │
  │  4. STOCKAGE SÉCURISÉ   jeton JWT, mot de passe, clé d'API       │
  │     flutter_secure_      chiffré par le système                  │
  │     storage                                                      │
  │                                                                  │
  └──────────────────────────────────────────────────────────────────┘
```

Voici le tableau de décision. Lisez-le de haut en bas et arrêtez-vous à la première ligne qui décrit votre besoin.

| Votre besoin | Volume | Famille | Paquet |
| --- | --- | --- | --- |
| Un secret (jeton, mot de passe, clé d'API) | Petit | Sécurisé | `flutter_secure_storage` |
| Un réglage simple (thème, langue, volume, pseudo) | Petit | Clé-valeur | `shared_preferences` |
| Un booléen « l'utilisateur a vu le tutoriel » | Minuscule | Clé-valeur | `shared_preferences` |
| Une réponse d'API mise en cache, quelques kilo-octets | Moyen | Fichier | `path_provider` + `dart:io` |
| Un export ou un import de sauvegarde par l'utilisateur | Moyen | Fichier | `path_provider` + `dart:io` |
| Une liste de 50 objets qu'on relit toujours en entier | Moyen | Fichier JSON | `path_provider` + `dart:io` |
| Une liste de 5 000 objets qu'on filtre et qu'on trie | Grand | Base de données | `sqflite` |
| Des relations entre entités (joueur → inventaire → objets) | Grand | Base de données | `sqflite` ou `drift` |
| Des écritures très fréquentes, position par position | Grand | Base de données | `sqflite`, `hive`, `isar` |

Trois règles simples résument tout cela :

1. **Un réglage n'est pas une donnée métier.** Le thème va dans `shared_preferences`. Le catalogue de potions n'y va pas.
2. **Un secret n'est jamais en clair.** Un jeton va dans `flutter_secure_storage`, point final.
3. **Dès que vous voulez chercher, trier ou filtrer, prenez une base de données.** Relire 5 000 objets depuis un fichier pour en afficher 20 est un gâchis.

---

## 54.2.1 — Ce que le stockage local ne remplace pas

Le stockage local ne remplace pas un serveur. Il est **local à un appareil**, et il disparaît dans quatre cas :

| Événement | Les données locales survivent ? |
| --- | --- |
| Fermeture de l'application | Oui |
| Redémarrage du téléphone | Oui |
| Mise à jour de l'application | Oui |
| Désinstallation de l'application | **Non** |
| Changement de téléphone | **Non** |
| « Effacer les données » dans les réglages Android | **Non** |
| Nettoyage du cache par le système | Non, pour le dossier cache uniquement |

Une donnée qui doit survivre à un changement de téléphone doit être sur un serveur. Le stockage local sert alors de **cache**, pas de source de vérité. Nous y reviendrons en 54.36.

---

## 54.3 — `shared_preferences` : le stockage clé-valeur

`shared_preferences` est le paquet officiel de l'équipe Flutter pour ranger de **petites valeurs nommées**.

Le modèle mental est celui d'une `Map<String, Object>` du chapitre 06, mais qui survit à la fermeture de l'application.

```text
   Une Map en mémoire                shared_preferences
   ──────────────────                ──────────────────

   {                                 {
     'pseudo': 'Aria',                 'pseudo': 'Aria',
     'volume': 0.8,                    'volume': 0.8,
     'sombre': true                    'sombre': true
   }                                 }
        │                                    │
        v                                    v
   perdue à la fermeture             écrite sur le disque
```

Derrière ce paquet, chaque plateforme utilise son propre mécanisme natif :

| Plateforme | Mécanisme réel |
| --- | --- |
| Android | `SharedPreferences` (un fichier XML privé) ou DataStore |
| iOS / macOS | `NSUserDefaults` (un fichier `.plist` privé) |
| Linux / Windows | Un fichier JSON dans le dossier de l'application |
| Web | `window.localStorage` du navigateur |

Vous n'avez pas à connaître ces détails pour écrire du code, mais ils expliquent les limites : ce sont tous des mécanismes conçus pour de **petites préférences**, pas pour des bases de données.

---

## 54.4 — Installation

Depuis la racine de votre projet :

```text
flutter pub add shared_preferences
```

La commande ajoute la dépendance dans `pubspec.yaml` et lance `flutter pub get` :

```yaml
dependencies:
  flutter:
    sdk: flutter
  shared_preferences: ^2.5.5
```

> Le numéro `2.5.5` est la version publiée au moment de la rédaction. Ne le recopiez pas à la main : laissez `flutter pub add` inscrire la version courante. C'est la règle générale de cette formation pour tous les paquets.

Comme `shared_preferences` contient du code natif (Android, iOS, ...), il faut **arrêter puis relancer complètement l'application** après l'installation. Un hot reload ne suffit pas : le code natif n'est chargé qu'au démarrage.

```text
  flutter pub add shared_preferences
            │
            v
  Arrêt complet de l'application  (pas un hot reload)
            │
            v
  flutter run
```

Si vous oubliez cette étape, vous obtiendrez au premier appel :

```text
MissingPluginException(No implementation found for method getAll
on channel plugins.flutter.io/shared_preferences)
```

C'est l'erreur la plus fréquente du chapitre. Elle ne vient pas de votre code : elle vient du fait que la partie native n'a pas été embarquée.

---

## 54.5 — `SharedPreferences.getInstance()`

Voici le premier programme complet.

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final SharedPreferences prefs = await SharedPreferences.getInstance();
  final int score = prefs.getInt('score') ?? 0;
  runApp(ApplicationScore(scoreInitial: score));
}

class ApplicationScore extends StatelessWidget {
  const ApplicationScore({super.key, required this.scoreInitial});

  final int scoreInitial;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Score persistant',
      theme: ThemeData(
        colorSchemeSeed: Colors.indigo,
        useMaterial3: true,
      ),
      home: PageScore(scoreInitial: scoreInitial),
    );
  }
}

class PageScore extends StatefulWidget {
  const PageScore({super.key, required this.scoreInitial});

  final int scoreInitial;

  @override
  State<PageScore> createState() => _PageScoreState();
}

class _PageScoreState extends State<PageScore> {
  late int _score = widget.scoreInitial;

  Future<void> _marquer() async {
    setState(() {
      _score += 10;
    });
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    await prefs.setInt('score', _score);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Score persistant')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
              '$_score',
              style: Theme.of(context).textTheme.displayLarge,
            ),
            const SizedBox(height: 8),
            const Text('Fermez l\'application et relancez-la.'),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: _marquer,
              child: const Text('Marquer 10 points'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :** vous appuyez cinq fois, l'écran affiche `50`. Vous faites un **hot restart**, l'écran affiche toujours `50`. Vous fermez l'application, vous la relancez : `50`.

Trois points méritent une explication.

**Point 1 — `main` est devenu `Future<void> main() async`.**
Nous avons besoin d'`await`. Le chapitre 15 vous a appris qu'`await` n'est légal que dans une fonction `async`. `main` peut être `async` sans problème.

**Point 2 — `WidgetsFlutterBinding.ensureInitialized()`.**
Cette ligne est **obligatoire** dès que vous appelez du code de plugin (donc du code natif) **avant** `runApp`. Elle initialise le pont entre Dart et le code natif de la plateforme.

Sans elle, vous obtenez :

```text
Unhandled Exception: ServicesBinding.defaultBinaryMessenger was accessed
before the binding was initialized.
```

**Point 3 — `getInstance()` renvoie un `Future`.**
Pourquoi ? Parce qu'au premier appel, le paquet doit demander au système natif l'ensemble des préférences déjà enregistrées, et cet aller-retour Dart ↔ natif est asynchrone. Il n'y a aucun moyen de le rendre synchrone.

Une fois cette lecture faite, l'instance garde une **copie en mémoire**. C'est pourquoi les lectures (`getInt`, `getString`, ...) sont **synchrones** : elles lisent la copie, pas le disque.

```text
   1er appel getInstance()
   ───────────────────────
   Dart  ──── « donne-moi tout » ────►  Natif
   Dart  ◄─── {score: 50, ...}    ────  Natif
   Dart  garde la copie en mémoire  →  lectures synchrones ensuite

   Appels suivants getInstance()
   ─────────────────────────────
   Dart  renvoie immédiatement la même instance (singleton)
```

Appeler `getInstance()` plusieurs fois ne coûte donc presque rien : la deuxième fois, l'instance est déjà là.

---

## 54.5.1 — Les deux API modernes : `SharedPreferencesAsync` et `SharedPreferencesWithCache`

Le paquet expose aujourd'hui trois API. Il faut savoir les distinguer.

| API | Lecture | Écriture | Cache en mémoire |
| --- | --- | --- | --- |
| `SharedPreferences` (historique) | synchrone | asynchrone | oui |
| `SharedPreferencesAsync` | asynchrone | asynchrone | non |
| `SharedPreferencesWithCache` | synchrone | asynchrone | oui, avec liste blanche |

La documentation officielle indique que `SharedPreferences` est une **API historique, destinée à être dépréciée à terme**. Les deux autres sont les remplaçantes recommandées.

`SharedPreferencesAsync` ne conserve rien en mémoire : chaque lecture interroge la plateforme.

```dart
import 'package:shared_preferences/shared_preferences.dart';

Future<void> demonstration() async {
  final SharedPreferencesAsync prefs = SharedPreferencesAsync();

  await prefs.setInt('score', 120);
  final int? score = await prefs.getInt('score');
  print(score); // 120

  await prefs.setString('pseudo', 'Aria');
  final String? pseudo = await prefs.getString('pseudo');
  print(pseudo); // Aria

  await prefs.remove('score');
  await prefs.clear();
}
```

Notez la différence essentielle : **`getInt` est ici asynchrone**, il faut l'`await`.

`SharedPreferencesWithCache` reprend le confort des lectures synchrones, mais vous oblige à déclarer les clés autorisées :

```dart
import 'package:shared_preferences/shared_preferences.dart';

Future<void> demonstration() async {
  final SharedPreferencesWithCache prefs =
      await SharedPreferencesWithCache.create(
    cacheOptions: const SharedPreferencesWithCacheOptions(
      allowList: <String>{'score', 'pseudo'},
    ),
  );

  await prefs.setInt('score', 120);
  final int? score = prefs.getInt('score'); // synchrone
  print(score); // 120
}
```

Quelle API choisir pour ce chapitre ? Nous utilisons `SharedPreferences.getInstance()` dans la majorité des exemples, parce que c'est celle que vous rencontrerez dans 95 % du code existant, des tutoriels et des projets d'entreprise. Mais la section 54.12 vous montrera comment isoler cette API derrière un service, précisément pour pouvoir en changer sans toucher au reste de l'application.

---

## 54.6 — Les types supportés

`shared_preferences` ne stocke que **cinq types**.

| Type Dart | Écriture | Lecture | Valeur de retour |
| --- | --- | --- | --- |
| `int` | `setInt(clé, valeur)` | `getInt(clé)` | `int?` |
| `double` | `setDouble(clé, valeur)` | `getDouble(clé)` | `double?` |
| `bool` | `setBool(clé, valeur)` | `getBool(clé)` | `bool?` |
| `String` | `setString(clé, valeur)` | `getString(clé)` | `String?` |
| `List<String>` | `setStringList(clé, valeur)` | `getStringList(clé)` | `List<String>?` |

C'est tout. Il n'y a pas de `setDateTime`, pas de `setMap`, pas de `setObject`, pas de `setList<int>`.

Ce n'est pas une limite arbitraire : ce sont les seuls types que les cinq plateformes savent stocker nativement de la même façon.

Pour les autres types, on **convertit**.

| Type à stocker | Conversion à l'écriture | Conversion à la lecture |
| --- | --- | --- |
| `DateTime` | `.toIso8601String()` puis `setString` | `getString` puis `DateTime.parse` |
| `DateTime` (variante) | `.millisecondsSinceEpoch` puis `setInt` | `getInt` puis `DateTime.fromMillisecondsSinceEpoch` |
| `enum` | `.name` puis `setString` | `getString` puis comparaison sur `.name` |
| `Color` | `.toARGB32()` puis `setInt` | `getInt` puis `Color(valeur)` |
| Objet Dart | `jsonEncode(objet.toJson())` puis `setString` | `getString` puis `jsonDecode` |
| `List<int>` | `.map((e) => e.toString()).toList()` | `.map(int.parse).toList()` |

Voici un programme complet qui exerce les cinq types et deux conversions.

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const ApplicationTypes());
}

class ApplicationTypes extends StatelessWidget {
  const ApplicationTypes({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Types supportés',
      theme: ThemeData(
        colorSchemeSeed: Colors.teal,
        useMaterial3: true,
      ),
      home: const PageTypes(),
    );
  }
}

class PageTypes extends StatefulWidget {
  const PageTypes({super.key});

  @override
  State<PageTypes> createState() => _PageTypesState();
}

class _PageTypesState extends State<PageTypes> {
  String _rapport = 'Appuyez sur le bouton.';

  Future<void> _executer() async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();

    // Les cinq types natifs.
    await prefs.setInt('niveau', 7);
    await prefs.setDouble('volume', 0.65);
    await prefs.setBool('sombre', true);
    await prefs.setString('pseudo', 'Aria');
    await prefs.setStringList(
      'inventaire',
      <String>['Potion', 'Épée', 'Bouclier'],
    );

    // Deux conversions.
    final DateTime derniereConnexion = DateTime(2026, 3, 14, 9, 30);
    await prefs.setString(
      'derniereConnexion',
      derniereConnexion.toIso8601String(),
    );
    await prefs.setInt('classe', Classe.magicienne.index);

    // Relecture.
    final int niveau = prefs.getInt('niveau') ?? 1;
    final double volume = prefs.getDouble('volume') ?? 1.0;
    final bool sombre = prefs.getBool('sombre') ?? false;
    final String pseudo = prefs.getString('pseudo') ?? 'Anonyme';
    final List<String> inventaire =
        prefs.getStringList('inventaire') ?? <String>[];
    final String? brut = prefs.getString('derniereConnexion');
    final DateTime? date = brut == null ? null : DateTime.parse(brut);
    final int indexClasse = prefs.getInt('classe') ?? 0;
    final Classe classe = Classe.values[indexClasse];

    setState(() {
      _rapport = <String>[
        'niveau            : $niveau',
        'volume            : $volume',
        'sombre            : $sombre',
        'pseudo            : $pseudo',
        'inventaire        : ${inventaire.join(', ')}',
        'derniereConnexion : $date',
        'classe            : ${classe.name}',
      ].join('\n');
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Types supportés')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            FilledButton(
              onPressed: _executer,
              child: const Text('Écrire puis relire'),
            ),
            const SizedBox(height: 24),
            Text(
              _rapport,
              style: const TextStyle(fontFamily: 'monospace'),
            ),
          ],
        ),
      ),
    );
  }
}

enum Classe { guerriere, magicienne, voleuse }
```

**Résultat :**

```text
niveau            : 7
volume            : 0.65
sombre            : true
pseudo            : Aria
inventaire        : Potion, Épée, Bouclier
derniereConnexion : 2026-03-14 09:30:00.000
classe            : magicienne
```

> Une remarque importante sur les `enum`. Ici nous stockons `index`. C'est court, mais **fragile** : si vous insérez une valeur au milieu de l'énumération dans une future version, tous les index enregistrés désignent autre chose. Préférez `.name` (une `String`) dès que l'énumération risque d'évoluer. L'exercice 3 revient sur ce point.

---

## 54.7 — Écrire et lire

Les méthodes d'écriture renvoient toutes un `Future<bool>` : `true` si l'écriture a réussi.

```dart
final SharedPreferences prefs = await SharedPreferences.getInstance();
final bool ok = await prefs.setInt('score', 120);
```

En pratique, on ignore presque toujours ce booléen. Mais **il faut `await` l'écriture** si la suite du code dépend du fait que la donnée soit bien posée (par exemple avant de fermer l'application, ou dans un test).

Les méthodes de lecture, elles, sont **synchrones** et renvoient un type **nullable** :

```dart
final int? score = prefs.getInt('score');
```

`null` signifie « cette clé n'existe pas ». Nous traitons ce cas en 54.8.

Trois méthodes utilitaires complètent l'API :

| Méthode | Rôle | Retour |
| --- | --- | --- |
| `containsKey(clé)` | La clé existe-t-elle ? | `bool` |
| `getKeys()` | Toutes les clés enregistrées | `Set<String>` |
| `reload()` | Recharge la copie mémoire depuis le disque | `Future<void>` |

`reload()` sert dans un cas précis : si un autre composant (une extension, un widget d'écran d'accueil, du code natif) a modifié les préférences pendant que votre application tournait, la copie en mémoire est périmée. `reload()` la resynchronise.

Voici un programme complet qui montre l'écriture, la lecture et l'inspection.

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const ApplicationProfil());
}

class ApplicationProfil extends StatelessWidget {
  const ApplicationProfil({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Profil du joueur',
      theme: ThemeData(
        colorSchemeSeed: Colors.deepPurple,
        useMaterial3: true,
      ),
      home: const PageProfil(),
    );
  }
}

class PageProfil extends StatefulWidget {
  const PageProfil({super.key});

  @override
  State<PageProfil> createState() => _PageProfilState();
}

class _PageProfilState extends State<PageProfil> {
  final TextEditingController _controleur = TextEditingController();
  String _pseudoEnregistre = '';
  Set<String> _cles = <String>{};

  @override
  void initState() {
    super.initState();
    _charger();
  }

  @override
  void dispose() {
    _controleur.dispose();
    super.dispose();
  }

  Future<void> _charger() async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    if (!mounted) return;
    setState(() {
      _pseudoEnregistre = prefs.getString('pseudo') ?? '(aucun)';
      _cles = prefs.getKeys();
    });
  }

  Future<void> _enregistrer() async {
    final String saisie = _controleur.text.trim();
    if (saisie.isEmpty) return;
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    await prefs.setString('pseudo', saisie);
    await _charger();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Profil du joueur')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            Text(
              'Pseudo enregistré : $_pseudoEnregistre',
              style: Theme.of(context).textTheme.titleMedium,
            ),
            const SizedBox(height: 24),
            TextField(
              controller: _controleur,
              decoration: const InputDecoration(
                labelText: 'Nouveau pseudo',
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 16),
            FilledButton(
              onPressed: _enregistrer,
              child: const Text('Enregistrer'),
            ),
            const SizedBox(height: 32),
            const Text('Clés présentes dans les préférences :'),
            const SizedBox(height: 8),
            Expanded(
              child: ListView(
                children: _cles
                    .map((String cle) => ListTile(
                          dense: true,
                          leading: const Icon(Icons.key),
                          title: Text(cle),
                        ))
                    .toList(),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :** vous saisissez `Aria`, vous appuyez sur « Enregistrer », la ligne du haut affiche `Pseudo enregistré : Aria` et la liste du bas affiche une clé `pseudo`. Après relancement de l'application, le pseudo est toujours là.

> Notez le `if (!mounted) return;` après l'`await`. C'est la règle vue au chapitre 45 : entre le début et la fin d'un `await`, l'utilisateur a pu quitter l'écran. Appeler `setState` sur un `State` démonté lève une exception.

---

## 54.8 — Les valeurs par défaut (rappel chapitre 12)

Toutes les méthodes `get...` renvoient un type nullable. Au premier lancement, **aucune clé n'existe** : tout est `null`.

Le chapitre 12 vous a donné l'outil exact pour ce cas : l'opérateur `??`.

```dart
final int score = prefs.getInt('score') ?? 0;
final String pseudo = prefs.getString('pseudo') ?? 'Anonyme';
final bool sombre = prefs.getBool('sombre') ?? false;
final double volume = prefs.getDouble('volume') ?? 1.0;
final List<String> sac = prefs.getStringList('sac') ?? <String>[];
```

À gauche du `??`, un `int?`. À droite, un `int`. Le résultat est un `int` non nullable : le reste du code n'a plus jamais à tester `null`.

Trois pièges classiques.

**Piège 1 — `!` au lieu de `??`.**

```dart
// À NE PAS FAIRE
final int score = prefs.getInt('score')!;
```

Cela fonctionne sur votre téléphone où la clé existe déjà, et plante chez tous vos utilisateurs au premier lancement :

```text
Null check operator used on a null value
```

**Piège 2 — un défaut différent selon les endroits.**

```dart
// Fichier A
final double volume = prefs.getDouble('volume') ?? 1.0;
// Fichier B
final double volume = prefs.getDouble('volume') ?? 0.5;
```

Deux écrans, deux volumes par défaut différents. C'est exactement ce que le service de la section 54.12 va empêcher : la valeur par défaut est déclarée **une seule fois**.

**Piège 3 — confondre « absent » et « valeur fausse ».**

```dart
final bool tutorielVu = prefs.getBool('tutorielVu') ?? false;
```

Ici `null` et `false` mènent au même comportement : on affiche le tutoriel. C'est correct.

Mais imaginez un réglage « notifications activées », dont le défaut souhaité est `true` :

```dart
final bool notifications = prefs.getBool('notifications') ?? true;
```

Si vous écrivez `?? false` par réflexe, vous désactivez les notifications de tout le monde au premier lancement. **La valeur par défaut est une décision de conception, pas une formalité.**

Voici un tableau de défauts raisonnables pour une application de jeu.

| Clé | Type | Défaut | Justification |
| --- | --- | --- | --- |
| `pseudo` | `String` | `'Aventurier'` | Un nom neutre vaut mieux qu'un champ vide |
| `score` | `int` | `0` | On commence à zéro |
| `meilleurScore` | `int` | `0` | Idem |
| `volume` | `double` | `0.8` | Audible sans être agressif |
| `sombre` | `bool` | `false` | On suit le thème clair par défaut |
| `suivreSysteme` | `bool` | `true` | On respecte le réglage du système |
| `tutorielVu` | `bool` | `false` | Un nouveau joueur n'a rien vu |
| `inventaire` | `List<String>` | `<String>[]` | Un sac vide, jamais `null` |

---

## 54.9 — Supprimer une clé, tout effacer

Deux méthodes, à ne surtout pas confondre.

```dart
await prefs.remove('score'); // supprime UNE clé
await prefs.clear();         // supprime TOUTES les clés de l'application
```

| Méthode | Portée | Cas d'usage |
| --- | --- | --- |
| `remove(clé)` | Une seule clé | Réinitialiser un réglage, oublier un brouillon |
| `clear()` | Toutes les clés | Déconnexion complète, « réinitialiser l'application » |

`clear()` est brutal. Après un `clear()`, le thème redevient clair, la langue redevient celle du système, le tutoriel se réaffiche. Ce n'est presque jamais ce que l'on veut lors d'une simple déconnexion.

La bonne pratique pour une déconnexion : supprimer **explicitement** les clés liées à l'utilisateur, et conserver les réglages d'interface.

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const ApplicationNettoyage());
}

class ApplicationNettoyage extends StatelessWidget {
  const ApplicationNettoyage({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Nettoyage',
      theme: ThemeData(
        colorSchemeSeed: Colors.orange,
        useMaterial3: true,
      ),
      home: const PageNettoyage(),
    );
  }
}

class PageNettoyage extends StatefulWidget {
  const PageNettoyage({super.key});

  @override
  State<PageNettoyage> createState() => _PageNettoyageState();
}

class _PageNettoyageState extends State<PageNettoyage> {
  Map<String, Object?> _contenu = <String, Object?>{};

  static const List<String> _clesUtilisateur = <String>[
    'pseudo',
    'score',
    'inventaire',
  ];

  @override
  void initState() {
    super.initState();
    _preparerEtLire();
  }

  Future<void> _preparerEtLire() async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    if (!prefs.containsKey('pseudo')) {
      await prefs.setString('pseudo', 'Aria');
      await prefs.setInt('score', 480);
      await prefs.setStringList('inventaire', <String>['Potion', 'Torche']);
      await prefs.setBool('sombre', true);
      await prefs.setDouble('volume', 0.7);
    }
    await _lire();
  }

  Future<void> _lire() async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    final Map<String, Object?> contenu = <String, Object?>{};
    for (final String cle in prefs.getKeys()) {
      contenu[cle] = prefs.get(cle);
    }
    if (!mounted) return;
    setState(() => _contenu = contenu);
  }

  Future<void> _deconnexion() async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    for (final String cle in _clesUtilisateur) {
      await prefs.remove(cle);
    }
    await _lire();
  }

  Future<void> _reinitialisationTotale() async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    await prefs.clear();
    await _lire();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Nettoyage des préférences')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            Expanded(
              child: _contenu.isEmpty
                  ? const Center(child: Text('Aucune préférence.'))
                  : ListView(
                      children: _contenu.entries
                          .map((MapEntry<String, Object?> e) => ListTile(
                                dense: true,
                                title: Text(e.key),
                                trailing: Text('${e.value}'),
                              ))
                          .toList(),
                    ),
            ),
            const SizedBox(height: 16),
            OutlinedButton(
              onPressed: _deconnexion,
              child: const Text('Déconnexion (remove ciblés)'),
            ),
            const SizedBox(height: 8),
            FilledButton(
              onPressed: _reinitialisationTotale,
              style: FilledButton.styleFrom(
                backgroundColor: Theme.of(context).colorScheme.error,
              ),
              child: const Text('Tout effacer (clear)'),
            ),
            const SizedBox(height: 8),
            TextButton(
              onPressed: _preparerEtLire,
              child: const Text('Recréer des données de test'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :** au lancement, cinq clés. Après « Déconnexion », il reste `sombre` et `volume`. Après « Tout effacer », la liste est vide.

Notez `prefs.get(cle)` : cette méthode générique renvoie un `Object?` sans que vous ayez à connaître le type. Elle est très pratique pour du débogage, beaucoup moins pour du code de production, où l'on veut un type précis.

---

## 54.10 — Où sont réellement stockées ces données

Savoir où vont les octets vous évitera des heures de perplexité.

| Plateforme | Emplacement |
| --- | --- |
| Android | `/data/data/<applicationId>/shared_prefs/FlutterSharedPreferences.xml` |
| iOS | Le domaine `NSUserDefaults` de l'application, dans `Library/Preferences/<bundleId>.plist` |
| macOS | Idem iOS |
| Linux | `~/.local/share/<applicationId>/shared_preferences.json` |
| Windows | `%APPDATA%\<companyName>\<productName>\shared_preferences.json` |
| Web | `window.localStorage` du navigateur, préfixé par `flutter.` |

Sur Android, le fichier XML ressemble à ceci :

```text
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <string name="flutter.pseudo">Aria</string>
    <int name="flutter.score" value="480" />
    <boolean name="flutter.sombre" value="true" />
</map>
```

Trois conséquences pratiques.

**Conséquence 1 — le préfixe `flutter.`.**
Le paquet préfixe automatiquement toutes vos clés par `flutter.`. Vous n'avez jamais à l'écrire vous-même, mais vous le verrez si vous inspectez le fichier, ou si vous devez lire ces valeurs depuis du code natif.

**Conséquence 2 — c'est un fichier en clair.**
Il n'est pas chiffré. Il est simplement **privé** : sur un appareil non rooté, seule votre application peut le lire. Sur un appareil rooté, sur un émulateur, ou dans une sauvegarde, il est parfaitement lisible. C'est le fondement de la section 54.11.

**Conséquence 3 — sur le Web, c'est visible par l'utilisateur.**
Ouvrez les outils de développement du navigateur, onglet « Application » puis « Local Storage » : vos préférences y sont, en clair, modifiables à la main. Nous y reviendrons en 54.33.

Sur Android, vous pouvez inspecter le fichier depuis un terminal, sur un émulateur :

```text
adb shell run-as com.exemple.mon_jeu cat shared_prefs/FlutterSharedPreferences.xml
```

---

## 54.11 — Ce qu'il ne faut PAS y mettre

Voici la liste noire. Elle est courte et absolue.

| À ne pas mettre | Pourquoi | Où le mettre |
| --- | --- | --- |
| Un mot de passe | Stocké en clair | `flutter_secure_storage` (ou nulle part) |
| Un jeton JWT, un jeton de rafraîchissement | Stocké en clair | `flutter_secure_storage` |
| Une clé d'API secrète | Stockée en clair, et lisible dans le binaire | Sur le serveur, jamais sur le client |
| Un numéro de carte bancaire | Stocké en clair, et non conforme | Nulle part, jamais |
| Une liste de 5 000 objets | Le fichier entier est relu à chaque démarrage | `sqflite` |
| Une image, un PDF, un binaire | Ce n'est pas fait pour | Un fichier (`path_provider`) |
| Un cache d'API de plusieurs mégaoctets | Ralentit le démarrage | Un fichier ou `sqflite` |
| Des données médicales ou personnelles sensibles | Non chiffré | Stockage sécurisé ou serveur |

La raison technique est toujours la même pour les gros volumes : **au premier `getInstance()`, le paquet lit et désérialise l'intégralité du contenu**. Un fichier de préférences de 4 Mo, c'est 4 Mo à lire et à parser avant que le premier pixel s'affiche.

Une règle de pouce simple :

> `shared_preferences` est fait pour quelques dizaines de clés et quelques kilo-octets. Au-delà, changez de famille.

Et une règle de sécurité :

> Si la fuite d'une donnée pose un problème, elle n'a rien à faire dans `shared_preferences`.

---

## 54.12 — Un service de préférences propre (thème, volume, pseudo)

Le code des sections précédentes fonctionne, mais il présente quatre défauts que tout projet réel finit par payer :

1. les clés sont écrites en dur, à plusieurs endroits, sous forme de chaînes (`'volume'`, `'volumme'`...) ;
2. les valeurs par défaut sont dupliquées, et divergent ;
3. `SharedPreferences` est appelé directement depuis les widgets, donc l'interface dépend d'un paquet ;
4. on ne peut pas tester la logique sans plugin.

La solution est un **service** : une classe qui centralise tout cela. C'est l'application directe du chapitre 16 (organisation d'un projet) et du chapitre 10 (encapsulation).

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

// ─────────────────────────────────────────────────────────────
//  lib/services/preferences_service.dart
// ─────────────────────────────────────────────────────────────

class PreferencesService {
  PreferencesService(this._prefs);

  final SharedPreferences _prefs;

  static Future<PreferencesService> creer() async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    return PreferencesService(prefs);
  }

  // Les clés : déclarées une seule fois, privées, constantes.
  static const String _clePseudo = 'pseudo';
  static const String _cleVolume = 'volume';
  static const String _cleTheme = 'theme';

  // Les valeurs par défaut : déclarées une seule fois, publiques.
  static const String pseudoParDefaut = 'Aventurier';
  static const double volumeParDefaut = 0.8;
  static const ThemeMode themeParDefaut = ThemeMode.system;

  // Pseudo -------------------------------------------------------
  String get pseudo => _prefs.getString(_clePseudo) ?? pseudoParDefaut;

  Future<void> setPseudo(String valeur) async {
    final String propre = valeur.trim();
    if (propre.isEmpty) {
      await _prefs.remove(_clePseudo);
      return;
    }
    await _prefs.setString(_clePseudo, propre);
  }

  // Volume -------------------------------------------------------
  double get volume => _prefs.getDouble(_cleVolume) ?? volumeParDefaut;

  Future<void> setVolume(double valeur) async {
    final double borne = valeur.clamp(0.0, 1.0);
    await _prefs.setDouble(_cleVolume, borne);
  }

  // Thème --------------------------------------------------------
  ThemeMode get themeMode {
    final String? nom = _prefs.getString(_cleTheme);
    return switch (nom) {
      'light' => ThemeMode.light,
      'dark' => ThemeMode.dark,
      'system' => ThemeMode.system,
      _ => themeParDefaut,
    };
  }

  Future<void> setThemeMode(ThemeMode mode) async {
    await _prefs.setString(_cleTheme, mode.name);
  }

  // Divers -------------------------------------------------------
  Future<void> reinitialiser() async {
    await _prefs.remove(_clePseudo);
    await _prefs.remove(_cleVolume);
    await _prefs.remove(_cleTheme);
  }
}

// ─────────────────────────────────────────────────────────────
//  lib/main.dart
// ─────────────────────────────────────────────────────────────

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final PreferencesService service = await PreferencesService.creer();
  runApp(ApplicationReglages(service: service));
}

class ApplicationReglages extends StatefulWidget {
  const ApplicationReglages({super.key, required this.service});

  final PreferencesService service;

  @override
  State<ApplicationReglages> createState() => _ApplicationReglagesState();
}

class _ApplicationReglagesState extends State<ApplicationReglages> {
  late ThemeMode _mode = widget.service.themeMode;

  Future<void> _changerTheme(ThemeMode mode) async {
    await widget.service.setThemeMode(mode);
    if (!mounted) return;
    setState(() => _mode = mode);
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Réglages',
      themeMode: _mode,
      theme: ThemeData(
        colorSchemeSeed: Colors.green,
        brightness: Brightness.light,
        useMaterial3: true,
      ),
      darkTheme: ThemeData(
        colorSchemeSeed: Colors.green,
        brightness: Brightness.dark,
        useMaterial3: true,
      ),
      home: PageReglages(
        service: widget.service,
        modeCourant: _mode,
        onChangerTheme: _changerTheme,
      ),
    );
  }
}

class PageReglages extends StatefulWidget {
  const PageReglages({
    super.key,
    required this.service,
    required this.modeCourant,
    required this.onChangerTheme,
  });

  final PreferencesService service;
  final ThemeMode modeCourant;
  final ValueChanged<ThemeMode> onChangerTheme;

  @override
  State<PageReglages> createState() => _PageReglagesState();
}

class _PageReglagesState extends State<PageReglages> {
  late final TextEditingController _pseudo =
      TextEditingController(text: widget.service.pseudo);
  late double _volume = widget.service.volume;

  @override
  void dispose() {
    _pseudo.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Réglages')),
      body: ListView(
        padding: const EdgeInsets.all(24),
        children: <Widget>[
          Text('Pseudo', style: Theme.of(context).textTheme.titleMedium),
          const SizedBox(height: 8),
          TextField(
            controller: _pseudo,
            decoration: const InputDecoration(border: OutlineInputBorder()),
            onSubmitted: (String valeur) async {
              await widget.service.setPseudo(valeur);
              if (!context.mounted) return;
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(content: Text('Pseudo : ${widget.service.pseudo}')),
              );
            },
          ),
          const SizedBox(height: 32),
          Text(
            'Volume : ${(_volume * 100).round()} %',
            style: Theme.of(context).textTheme.titleMedium,
          ),
          Slider(
            value: _volume,
            onChanged: (double v) => setState(() => _volume = v),
            onChangeEnd: (double v) => widget.service.setVolume(v),
          ),
          const SizedBox(height: 32),
          Text('Thème', style: Theme.of(context).textTheme.titleMedium),
          const SizedBox(height: 8),
          SegmentedButton<ThemeMode>(
            segments: const <ButtonSegment<ThemeMode>>[
              ButtonSegment<ThemeMode>(
                value: ThemeMode.light,
                label: Text('Clair'),
                icon: Icon(Icons.light_mode),
              ),
              ButtonSegment<ThemeMode>(
                value: ThemeMode.system,
                label: Text('Système'),
                icon: Icon(Icons.settings),
              ),
              ButtonSegment<ThemeMode>(
                value: ThemeMode.dark,
                label: Text('Sombre'),
                icon: Icon(Icons.dark_mode),
              ),
            ],
            selected: <ThemeMode>{widget.modeCourant},
            onSelectionChanged: (Set<ThemeMode> selection) {
              widget.onChangerTheme(selection.first);
            },
          ),
          const SizedBox(height: 48),
          OutlinedButton(
            onPressed: () async {
              await widget.service.reinitialiser();
              if (!context.mounted) return;
              setState(() {
                _pseudo.text = widget.service.pseudo;
                _volume = widget.service.volume;
              });
              widget.onChangerTheme(widget.service.themeMode);
            },
            child: const Text('Réinitialiser les réglages'),
          ),
        ],
      ),
    );
  }
}
```

**Résultat :** un écran de réglages complet. Le pseudo, le volume et le thème survivent au relancement de l'application.

Ce que le service apporte, ligne par ligne :

| Problème initial | Réponse du service |
| --- | --- |
| Clés en dur, fautes de frappe | `static const String _clePseudo` : le compilateur vérifie |
| Défauts divergents | Un seul `?? pseudoParDefaut` dans tout le projet |
| L'interface dépend de `shared_preferences` | Les widgets ne connaissent que `PreferencesService` |
| Validation absente | `trim()`, `clamp(0.0, 1.0)` faits une fois pour toutes |
| Non testable | Le constructeur reçoit un `SharedPreferences` : on peut en injecter un faux (54.38) |

> Notez `switch (nom) { 'light' => ..., _ => ... }`. C'est l'expression `switch` de Dart 3 : elle renvoie une valeur, elle est exhaustive grâce au cas `_`, et elle évite un `if/else` en cascade.

---

## 54.13 — Persister le choix de thème du chapitre 51

Au chapitre 51, vous avez construit un sélecteur de thème avec `ThemeMode`. Il avait un défaut : au relancement, on revenait au thème système.

La section précédente a déjà résolu ce point, mais avec `setState` remonté jusqu'à la racine. Voici la version propre, avec le `ChangeNotifier` du chapitre 52. C'est le patron que vous réutiliserez dans tous vos projets.

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

// ─────────────────────────────────────────────────────────────
//  lib/state/theme_controller.dart
// ─────────────────────────────────────────────────────────────

class ThemeController extends ChangeNotifier {
  ThemeController(this._prefs) {
    _mode = _lire();
  }

  static const String _cle = 'themeMode';

  final SharedPreferences _prefs;
  ThemeMode _mode = ThemeMode.system;

  ThemeMode get mode => _mode;

  ThemeMode _lire() {
    final String? nom = _prefs.getString(_cle);
    return switch (nom) {
      'light' => ThemeMode.light,
      'dark' => ThemeMode.dark,
      _ => ThemeMode.system,
    };
  }

  Future<void> changer(ThemeMode nouveau) async {
    if (nouveau == _mode) return;
    _mode = nouveau;
    notifyListeners();               // 1. l'interface réagit tout de suite
    await _prefs.setString(_cle, nouveau.name); // 2. le disque suit
  }

  Future<void> basculer() async {
    final ThemeMode cible =
        _mode == ThemeMode.dark ? ThemeMode.light : ThemeMode.dark;
    await changer(cible);
  }
}

// ─────────────────────────────────────────────────────────────
//  lib/main.dart
// ─────────────────────────────────────────────────────────────

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final SharedPreferences prefs = await SharedPreferences.getInstance();
  runApp(ApplicationTheme(controleur: ThemeController(prefs)));
}

class ApplicationTheme extends StatelessWidget {
  const ApplicationTheme({super.key, required this.controleur});

  final ThemeController controleur;

  @override
  Widget build(BuildContext context) {
    return ListenableBuilder(
      listenable: controleur,
      builder: (BuildContext context, Widget? child) {
        return MaterialApp(
          title: 'Thème persistant',
          themeMode: controleur.mode,
          theme: ThemeData(
            colorSchemeSeed: Colors.indigo,
            brightness: Brightness.light,
            useMaterial3: true,
          ),
          darkTheme: ThemeData(
            colorSchemeSeed: Colors.indigo,
            brightness: Brightness.dark,
            useMaterial3: true,
          ),
          home: PageAccueil(controleur: controleur),
        );
      },
    );
  }
}

class PageAccueil extends StatelessWidget {
  const PageAccueil({super.key, required this.controleur});

  final ThemeController controleur;

  String _libelle(ThemeMode mode) => switch (mode) {
        ThemeMode.light => 'Clair',
        ThemeMode.dark => 'Sombre',
        ThemeMode.system => 'Système',
      };

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Thème persistant'),
        actions: <Widget>[
          IconButton(
            onPressed: controleur.basculer,
            icon: const Icon(Icons.brightness_6),
            tooltip: 'Basculer clair / sombre',
          ),
        ],
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Icon(
              Icons.palette,
              size: 96,
              color: Theme.of(context).colorScheme.primary,
            ),
            const SizedBox(height: 16),
            Text(
              'Mode actuel : ${_libelle(controleur.mode)}',
              style: Theme.of(context).textTheme.titleLarge,
            ),
            const SizedBox(height: 32),
            Wrap(
              spacing: 12,
              children: ThemeMode.values.map((ThemeMode mode) {
                final bool actif = mode == controleur.mode;
                return ChoiceChip(
                  label: Text(_libelle(mode)),
                  selected: actif,
                  onSelected: (_) => controleur.changer(mode),
                );
              }).toList(),
            ),
            const SizedBox(height: 32),
            const Text('Relancez l\'application : le choix est conservé.'),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :** vous choisissez « Sombre », l'écran devient sombre immédiatement. Vous fermez l'application, vous la relancez : elle démarre en sombre, sans flash blanc.

Deux détails de conception valent d'être soulignés.

**Détail 1 — l'ordre dans `changer()`.**

```dart
_mode = nouveau;
notifyListeners();                          // interface d'abord
await _prefs.setString(_cle, nouveau.name); // disque ensuite
```

L'interface est mise à jour **avant** l'écriture sur disque. L'écriture prend quelques millisecondes ; il n'y a aucune raison de faire attendre l'utilisateur. Si l'écriture échouait, le pire scénario est que le réglage ne soit pas retenu au prochain lancement. C'est acceptable pour un thème.

Pour une donnée critique (une commande, un paiement), on ferait exactement l'inverse : écrire d'abord, notifier ensuite, et afficher une erreur si l'écriture échoue.

**Détail 2 — pas de flash blanc.**
`main()` lit les préférences **avant** `runApp`. Le premier `build` connaît donc déjà le bon thème. Si vous chargiez le thème dans `initState` d'un widget, l'utilisateur verrait un écran clair pendant une frame, puis un écran sombre. C'est visible, et désagréable.

```text
  MAUVAIS                                BON
  ───────                                ───

  runApp()                               await prefs
     │                                      │
     v                                      v
  build() → thème clair                  runApp()
     │                                      │
     v                                      v
  initState → lecture async              build() → thème sombre
     │                                   (correct dès la 1re frame)
     v
  setState → thème sombre
  (l'utilisateur a vu le flash)
```

---

## 54.14 — Sérialiser un objet en JSON pour le stocker (rappel chapitre 17)

`shared_preferences` ne sait pas stocker un objet Dart. Mais il sait stocker une `String`. Et le chapitre 17 vous a appris à transformer un objet en `String` : le JSON.

La chaîne complète est la suivante :

```text
  Objet Dart          Map<String, dynamic>        String JSON        Disque
  ──────────          ────────────────────        ───────────        ──────

  Joueur(...)  ──►    joueur.toJson()     ──►    jsonEncode(...)  ──►  setString
                                                                          │
  Joueur(...)  ◄──   Joueur.fromJson(...) ◄──    jsonDecode(...)  ◄──  getString
```

Voici un programme complet.

```dart
import 'dart:convert';

import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

// ─────────────────────────────────────────────────────────────
//  lib/models/joueur.dart
// ─────────────────────────────────────────────────────────────

class Joueur {
  const Joueur({
    required this.pseudo,
    required this.niveau,
    required this.pointsDeVie,
    required this.inventaire,
    required this.derniereConnexion,
  });

  final String pseudo;
  final int niveau;
  final double pointsDeVie;
  final List<String> inventaire;
  final DateTime derniereConnexion;

  Map<String, dynamic> toJson() => <String, dynamic>{
        'pseudo': pseudo,
        'niveau': niveau,
        'pointsDeVie': pointsDeVie,
        'inventaire': inventaire,
        'derniereConnexion': derniereConnexion.toIso8601String(),
      };

  factory Joueur.fromJson(Map<String, dynamic> json) {
    return Joueur(
      pseudo: json['pseudo'] as String? ?? 'Anonyme',
      niveau: json['niveau'] as int? ?? 1,
      pointsDeVie: (json['pointsDeVie'] as num?)?.toDouble() ?? 100.0,
      inventaire: (json['inventaire'] as List<dynamic>? ?? <dynamic>[])
          .map((dynamic e) => e as String)
          .toList(),
      derniereConnexion: DateTime.tryParse(
            json['derniereConnexion'] as String? ?? '',
          ) ??
          DateTime.fromMillisecondsSinceEpoch(0),
    );
  }

  Joueur copyWith({
    String? pseudo,
    int? niveau,
    double? pointsDeVie,
    List<String>? inventaire,
    DateTime? derniereConnexion,
  }) {
    return Joueur(
      pseudo: pseudo ?? this.pseudo,
      niveau: niveau ?? this.niveau,
      pointsDeVie: pointsDeVie ?? this.pointsDeVie,
      inventaire: inventaire ?? this.inventaire,
      derniereConnexion: derniereConnexion ?? this.derniereConnexion,
    );
  }
}

// ─────────────────────────────────────────────────────────────
//  lib/services/joueur_store.dart
// ─────────────────────────────────────────────────────────────

class JoueurStore {
  JoueurStore(this._prefs);

  static const String _cle = 'joueur';

  final SharedPreferences _prefs;

  Joueur? lire() {
    final String? brut = _prefs.getString(_cle);
    if (brut == null) return null;
    try {
      final Map<String, dynamic> json =
          jsonDecode(brut) as Map<String, dynamic>;
      return Joueur.fromJson(json);
    } on FormatException {
      // Donnée corrompue : on l'efface plutôt que de planter au démarrage.
      _prefs.remove(_cle);
      return null;
    }
  }

  Future<void> ecrire(Joueur joueur) async {
    await _prefs.setString(_cle, jsonEncode(joueur.toJson()));
  }

  Future<void> effacer() async {
    await _prefs.remove(_cle);
  }
}

// ─────────────────────────────────────────────────────────────
//  lib/main.dart
// ─────────────────────────────────────────────────────────────

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final SharedPreferences prefs = await SharedPreferences.getInstance();
  runApp(ApplicationJoueur(store: JoueurStore(prefs)));
}

class ApplicationJoueur extends StatelessWidget {
  const ApplicationJoueur({super.key, required this.store});

  final JoueurStore store;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Joueur persistant',
      theme: ThemeData(
        colorSchemeSeed: Colors.deepOrange,
        useMaterial3: true,
      ),
      home: PageJoueur(store: store),
    );
  }
}

class PageJoueur extends StatefulWidget {
  const PageJoueur({super.key, required this.store});

  final JoueurStore store;

  @override
  State<PageJoueur> createState() => _PageJoueurState();
}

class _PageJoueurState extends State<PageJoueur> {
  late Joueur _joueur = widget.store.lire() ??
      Joueur(
        pseudo: 'Aria',
        niveau: 1,
        pointsDeVie: 100,
        inventaire: const <String>['Potion'],
        derniereConnexion: DateTime.now(),
      );

  Future<void> _monterDeNiveau() async {
    setState(() {
      _joueur = _joueur.copyWith(
        niveau: _joueur.niveau + 1,
        pointsDeVie: _joueur.pointsDeVie + 20,
        derniereConnexion: DateTime.now(),
      );
    });
    await widget.store.ecrire(_joueur);
  }

  Future<void> _ramasser(String objet) async {
    setState(() {
      _joueur = _joueur.copyWith(
        inventaire: <String>[..._joueur.inventaire, objet],
      );
    });
    await widget.store.ecrire(_joueur);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(_joueur.pseudo)),
      body: ListView(
        padding: const EdgeInsets.all(24),
        children: <Widget>[
          ListTile(
            leading: const Icon(Icons.military_tech),
            title: const Text('Niveau'),
            trailing: Text('${_joueur.niveau}'),
          ),
          ListTile(
            leading: const Icon(Icons.favorite),
            title: const Text('Points de vie'),
            trailing: Text(_joueur.pointsDeVie.toStringAsFixed(0)),
          ),
          ListTile(
            leading: const Icon(Icons.schedule),
            title: const Text('Dernière connexion'),
            subtitle: Text('${_joueur.derniereConnexion}'),
          ),
          const Divider(),
          const Padding(
            padding: EdgeInsets.symmetric(vertical: 8),
            child: Text('Inventaire'),
          ),
          Wrap(
            spacing: 8,
            children: _joueur.inventaire
                .map((String o) => Chip(label: Text(o)))
                .toList(),
          ),
          const SizedBox(height: 32),
          FilledButton(
            onPressed: _monterDeNiveau,
            child: const Text('Monter de niveau'),
          ),
          const SizedBox(height: 8),
          OutlinedButton(
            onPressed: () => _ramasser('Torche'),
            child: const Text('Ramasser une torche'),
          ),
          const SizedBox(height: 8),
          TextButton(
            onPressed: () async {
              await widget.store.effacer();
              if (!context.mounted) return;
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(
                  content: Text('Sauvegarde effacée. Relancez l\'application.'),
                ),
              );
            },
            child: const Text('Effacer la sauvegarde'),
          ),
        ],
      ),
    );
  }
}
```

**Résultat :** le joueur monte de niveau, ramasse des objets, et retrouve tout après un relancement. La chaîne réellement écrite sur le disque ressemble à ceci :

```json
{
  "pseudo": "Aria",
  "niveau": 3,
  "pointsDeVie": 140.0,
  "inventaire": ["Potion", "Torche", "Torche"],
  "derniereConnexion": "2026-03-14T09:30:00.000"
}
```

Trois points de vigilance, tous hérités du chapitre 17.

**Point 1 — `fromJson` doit être défensif.**
Le JSON vient de **votre propre disque**, mais il a pu être écrit par une **version précédente** de votre application, où le champ `pointsDeVie` n'existait pas encore. D'où les `?? valeurDefaut` partout. Un `fromJson` qui suppose que tous les champs sont présents provoquera un plantage au démarrage lors de votre première mise à jour.

**Point 2 — attraper `FormatException`.**
Si l'écriture précédente a été interrompue (batterie vide en plein `setString`), la chaîne stockée peut être tronquée. `jsonDecode` lèvera alors une `FormatException` (chapitre 13). Une donnée locale corrompue ne doit jamais empêcher l'application de démarrer : on l'efface et on repart des valeurs par défaut.

**Point 3 — `(json['pointsDeVie'] as num?)?.toDouble()`.**
Le JSON ne distingue pas `int` et `double`. Si vous écrivez `100.0` et que le décodeur relit `100`, un `as double` direct lèverait une erreur de type. Passer par `num` puis `.toDouble()` couvre les deux cas. C'est le piège numéro un du JSON en Dart.

---

## 54.15 — `path_provider` : les dossiers de l'application

`shared_preferences` s'arrête vite. Dès que vous voulez écrire un fichier — un export, un cache d'API, une image téléchargée — il vous faut un **chemin sur le disque**.

Or vous ne pouvez pas écrire ce chemin en dur. Il diffère à chaque plateforme, à chaque appareil, et même à chaque installation :

```text
Android : /data/user/0/com.exemple.mon_jeu/app_flutter
iOS     : /var/mobile/Containers/Data/Application/8A3F.../Documents
macOS   : /Users/aria/Library/Containers/com.exemple.monJeu/Data/Documents
Linux   : /home/aria/.local/share/mon_jeu
Windows : C:\Users\aria\AppData\Roaming\exemple\mon_jeu
```

`path_provider` est le paquet officiel qui demande ces chemins au système.

```text
flutter pub add path_provider
```

```yaml
dependencies:
  flutter:
    sdk: flutter
  path_provider: ^2.1.6
```

Il expose six fonctions, toutes asynchrones.

| Fonction | Retour | Android | iOS | Linux | macOS | Windows |
| --- | --- | --- | --- | --- | --- | --- |
| `getTemporaryDirectory()` | `Future<Directory>` | oui | oui | oui | oui | oui |
| `getApplicationSupportDirectory()` | `Future<Directory>` | oui | oui | oui | oui | oui |
| `getApplicationDocumentsDirectory()` | `Future<Directory>` | oui | oui | oui | oui | oui |
| `getApplicationCacheDirectory()` | `Future<Directory>` | oui | oui | oui | oui | oui |
| `getDownloadsDirectory()` | `Future<Directory?>` | oui | oui | oui | oui | oui |
| `getExternalStorageDirectory()` | `Future<Directory?>` | oui | non | non | non | non |

Deux remarques.

- `getDownloadsDirectory()` et `getExternalStorageDirectory()` renvoient un `Directory?` **nullable** : la notion n'existe pas partout. Il faut donc tester le `null` (chapitre 12).
- **Aucune de ces fonctions ne marche sur le Web.** Un navigateur n'a pas de système de fichiers accessible. Nous traiterons ce cas en 54.33.

Voici un programme qui affiche tous les chemins de votre appareil.

```dart
import 'dart:io';

import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';
import 'package:path_provider/path_provider.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const ApplicationChemins());
}

class ApplicationChemins extends StatelessWidget {
  const ApplicationChemins({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Dossiers de l\'application',
      theme: ThemeData(
        colorSchemeSeed: Colors.blueGrey,
        useMaterial3: true,
      ),
      home: const PageChemins(),
    );
  }
}

class PageChemins extends StatefulWidget {
  const PageChemins({super.key});

  @override
  State<PageChemins> createState() => _PageCheminsState();
}

class _PageCheminsState extends State<PageChemins> {
  Map<String, String> _chemins = <String, String>{};
  bool _charge = false;

  @override
  void initState() {
    super.initState();
    _lireChemins();
  }

  Future<void> _lireChemins() async {
    if (kIsWeb) {
      setState(() {
        _chemins = <String, String>{
          'Web': 'path_provider n\'est pas disponible sur le Web.',
        };
        _charge = true;
      });
      return;
    }

    final Map<String, String> resultat = <String, String>{};

    Future<void> essayer(
      String nom,
      Future<Directory?> Function() fonction,
    ) async {
      try {
        final Directory? dossier = await fonction();
        resultat[nom] = dossier?.path ?? '(non disponible)';
      } on Object catch (e) {
        resultat[nom] = 'Erreur : $e';
      }
    }

    await essayer('Temporaire', getTemporaryDirectory);
    await essayer('Support', getApplicationSupportDirectory);
    await essayer('Documents', getApplicationDocumentsDirectory);
    await essayer('Cache', getApplicationCacheDirectory);
    await essayer('Téléchargements', getDownloadsDirectory);
    if (Platform.isAndroid) {
      await essayer('Stockage externe', getExternalStorageDirectory);
    }

    if (!mounted) return;
    setState(() {
      _chemins = resultat;
      _charge = true;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Dossiers de l\'application')),
      body: !_charge
          ? const Center(child: CircularProgressIndicator())
          : ListView(
              padding: const EdgeInsets.all(16),
              children: _chemins.entries.map((MapEntry<String, String> e) {
                return Card(
                  child: Padding(
                    padding: const EdgeInsets.all(16),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: <Widget>[
                        Text(
                          e.key,
                          style: Theme.of(context).textTheme.titleMedium,
                        ),
                        const SizedBox(height: 8),
                        SelectableText(
                          e.value,
                          style: const TextStyle(
                            fontFamily: 'monospace',
                            fontSize: 12,
                          ),
                        ),
                      ],
                    ),
                  ),
                );
              }).toList(),
            ),
    );
  }
}
```

**Résultat (exemple sur un émulateur Android) :**

```text
Temporaire       /data/user/0/com.exemple.mon_jeu/cache
Support          /data/user/0/com.exemple.mon_jeu/files
Documents        /data/user/0/com.exemple.mon_jeu/app_flutter
Cache            /data/user/0/com.exemple.mon_jeu/cache
Téléchargements  /storage/emulated/0/Download
Stockage externe /storage/emulated/0/Android/data/com.exemple.mon_jeu/files
```

> Le `import 'dart:io'` est nécessaire pour `Directory` et `Platform`. Attention : `dart:io` **n'existe pas sur le Web**. Le garde `if (kIsWeb)` évite l'appel, mais l'import lui-même empêchera la compilation web. Pour un projet réellement multi-plateforme, on isole le code `dart:io` dans un fichier séparé et on utilise l'import conditionnel. C'est traité en 54.33.

---

## 54.16 — Documents, support, cache, temporaire : lequel choisir

C'est la décision la plus mal comprise du chapitre. Quatre dossiers, quatre politiques de survie très différentes.

| Dossier | Sauvegardé par le système ? | Effaçable par le système ? | Visible par l'utilisateur ? |
| --- | --- | --- | --- |
| Documents | Oui (iCloud, Google Backup) | Non | Sur iOS, si l'option est activée |
| Support | Oui | Non | Non |
| Cache | Non | **Oui, à tout moment** | Non |
| Temporaire | Non | **Oui, à tout moment** | Non |

Le mot important est **effaçable**. Le système d'exploitation se réserve le droit de vider le dossier cache et le dossier temporaire **quand il veut**, sans prévenir, y compris pendant que votre application tourne. C'est ainsi que le téléphone libère de la place quand la mémoire est pleine.

```text
  ┌──────────────────────────────────────────────────────────┐
  │  DOCUMENTS                                               │
  │  Ce que l'utilisateur perdrait avec colère.              │
  │  → sauvegardes de partie, notes, exports, dessins        │
  ├──────────────────────────────────────────────────────────┤
  │  SUPPORT                                                 │
  │  Ce dont l'application a besoin, mais que l'utilisateur  │
  │  n'a pas produit.                                        │
  │  → base de données, index, fichier de configuration      │
  ├──────────────────────────────────────────────────────────┤
  │  CACHE                                                   │
  │  Ce qui est reconstructible depuis le réseau.            │
  │  → réponses d'API, images téléchargées, vignettes        │
  ├──────────────────────────────────────────────────────────┤
  │  TEMPORAIRE                                              │
  │  Ce qui ne sert que pendant quelques secondes.           │
  │  → fichier en cours de téléchargement, image redimen-    │
  │    sionnée avant envoi, décompression intermédiaire      │
  └──────────────────────────────────────────────────────────┘
```

L'arbre de décision tient en trois questions.

```text
  Puis-je reconstruire ce fichier depuis le réseau ou le recalculer ?
        │
        ├── OUI ──► Est-ce que j'en ai besoin plus de quelques minutes ?
        │              ├── NON  ──► Temporaire
        │              └── OUI  ──► Cache
        │
        └── NON ──► L'utilisateur l'a-t-il créé lui-même ?
                       ├── OUI  ──► Documents
                       └── NON  ──► Support
```

Trois erreurs très fréquentes, et leurs conséquences :

| Erreur | Conséquence réelle |
| --- | --- |
| Sauvegarde de partie dans le cache | Le joueur perd sa progression sans comprendre pourquoi |
| Base de données dans le cache | La base disparaît, l'application plante ou repart à vide |
| Cache d'images dans Documents | La sauvegarde iCloud de l'utilisateur gonfle de 500 Mo |
| Fichier temporaire jamais supprimé du dossier Documents | L'espace occupé grossit indéfiniment |

> Retenez la formulation d'Apple, qui est la plus claire : **« Documents, c'est ce que vous ne pouvez pas régénérer. Cache, c'est ce que vous pouvez régénérer. »**

---

## 54.17 — Écrire un fichier

Avec un dossier et le paquet `path` (déjà présent dans toute application Flutter), on construit un chemin, puis on écrit.

```dart
import 'dart:io';
import 'package:path/path.dart' as p;
import 'package:path_provider/path_provider.dart';

Future<File> _fichierNotes() async {
  final Directory dossier = await getApplicationDocumentsDirectory();
  return File(p.join(dossier.path, 'notes.txt'));
}

Future<void> ecrire(String contenu) async {
  final File fichier = await _fichierNotes();
  await fichier.writeAsString(contenu);
}
```

Pourquoi `p.join` plutôt que `'${dossier.path}/notes.txt'` ? Parce que le séparateur de chemin est `/` sur Android, iOS, Linux et macOS, mais `\` sur Windows. `p.join` choisit le bon. C'est une habitude à prendre dès maintenant.

L'API d'écriture de `dart:io` :

| Méthode | Effet |
| --- | --- |
| `writeAsString(contenu)` | Écrit du texte, **écrase** le fichier existant |
| `writeAsString(contenu, mode: FileMode.append)` | Ajoute à la fin |
| `writeAsBytes(octets)` | Écrit des octets bruts (`List<int>`) |
| `writeAsStringSync(contenu)` | Version bloquante : à éviter dans l'interface |
| `create(recursive: true)` | Crée le fichier et les dossiers parents |
| `delete()` | Supprime le fichier |

Voici un programme complet : un journal de bord de partie.

```dart
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:path_provider/path_provider.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const ApplicationJournal());
}

class ApplicationJournal extends StatelessWidget {
  const ApplicationJournal({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Journal de partie',
      theme: ThemeData(
        colorSchemeSeed: Colors.brown,
        useMaterial3: true,
      ),
      home: const PageJournal(),
    );
  }
}

class PageJournal extends StatefulWidget {
  const PageJournal({super.key});

  @override
  State<PageJournal> createState() => _PageJournalState();
}

class _PageJournalState extends State<PageJournal> {
  final TextEditingController _saisie = TextEditingController();
  String _contenu = '';
  String _chemin = '';

  @override
  void initState() {
    super.initState();
    _relire();
  }

  @override
  void dispose() {
    _saisie.dispose();
    super.dispose();
  }

  Future<File> _fichier() async {
    final Directory dossier = await getApplicationDocumentsDirectory();
    return File(p.join(dossier.path, 'journal.txt'));
  }

  Future<void> _relire() async {
    final File fichier = await _fichier();
    final bool existe = await fichier.exists();
    final String contenu = existe ? await fichier.readAsString() : '';
    if (!mounted) return;
    setState(() {
      _contenu = contenu;
      _chemin = fichier.path;
    });
  }

  Future<void> _ajouterLigne() async {
    final String texte = _saisie.text.trim();
    if (texte.isEmpty) return;
    final File fichier = await _fichier();
    final String horodatage = DateTime.now().toIso8601String().substring(11, 19);
    await fichier.writeAsString(
      '[$horodatage] $texte\n',
      mode: FileMode.append,
    );
    _saisie.clear();
    await _relire();
  }

  Future<void> _remplacerTout() async {
    final File fichier = await _fichier();
    await fichier.writeAsString('--- Nouvelle partie ---\n');
    await _relire();
  }

  Future<void> _supprimer() async {
    final File fichier = await _fichier();
    if (await fichier.exists()) {
      await fichier.delete();
    }
    await _relire();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Journal de partie')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            SelectableText(
              _chemin,
              style: const TextStyle(fontSize: 11, fontFamily: 'monospace'),
            ),
            const SizedBox(height: 12),
            Expanded(
              child: Container(
                width: double.infinity,
                padding: const EdgeInsets.all(12),
                decoration: BoxDecoration(
                  border: Border.all(
                    color: Theme.of(context).colorScheme.outlineVariant,
                  ),
                  borderRadius: BorderRadius.circular(8),
                ),
                child: SingleChildScrollView(
                  child: Text(
                    _contenu.isEmpty ? '(fichier vide ou absent)' : _contenu,
                    style: const TextStyle(fontFamily: 'monospace'),
                  ),
                ),
              ),
            ),
            const SizedBox(height: 12),
            TextField(
              controller: _saisie,
              decoration: const InputDecoration(
                labelText: 'Événement de la partie',
                border: OutlineInputBorder(),
              ),
              onSubmitted: (_) => _ajouterLigne(),
            ),
            const SizedBox(height: 12),
            Row(
              children: <Widget>[
                Expanded(
                  child: FilledButton(
                    onPressed: _ajouterLigne,
                    child: const Text('Ajouter'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: OutlinedButton(
                    onPressed: _remplacerTout,
                    child: const Text('Remplacer'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: TextButton(
                    onPressed: _supprimer,
                    child: const Text('Supprimer'),
                  ),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
[09:31:02] Entrée dans la caverne
[09:31:18] Rencontre avec un gobelin
[09:31:44] Potion de soin utilisée
```

Le bouton « Remplacer » écrase tout : c'est le comportement de `writeAsString` sans `mode`. Le bouton « Ajouter » utilise `FileMode.append` et conserve l'historique. C'est la distinction à mémoriser.

---

## 54.18 — Lire un fichier

La lecture est symétrique.

| Méthode | Retour | Usage |
| --- | --- | --- |
| `readAsString()` | `Future<String>` | Fichier texte entier |
| `readAsLines()` | `Future<List<String>>` | Fichier texte ligne par ligne |
| `readAsBytes()` | `Future<Uint8List>` | Fichier binaire |
| `openRead()` | `Stream<List<int>>` | Gros fichier, lecture par morceaux |
| `exists()` | `Future<bool>` | Le fichier est-il là ? |
| `length()` | `Future<int>` | Taille en octets |
| `stat()` | `Future<FileStat>` | Date de modification, taille, type |

`stat()` est particulièrement utile pour un cache : il donne la date de dernière modification, donc l'âge de la donnée.

```dart
final FileStat infos = await fichier.stat();
final Duration age = DateTime.now().difference(infos.modified);
if (age > const Duration(hours: 1)) {
  // Le cache est périmé.
}
```

Pour un fichier volumineux, ne chargez pas tout en mémoire d'un coup. `openRead()` renvoie un `Stream` (chapitre 15), que vous pouvez consommer morceau par morceau :

```dart
import 'dart:convert';
import 'dart:io';

Future<int> compterLignes(File fichier) async {
  int total = 0;
  final Stream<String> lignes = fichier
      .openRead()
      .transform(utf8.decoder)
      .transform(const LineSplitter());
  await for (final String _ in lignes) {
    total++;
  }
  return total;
}
```

Cette forme lit un fichier de 200 Mo sans jamais dépasser quelques kilo-octets de mémoire vive.

---

## 54.19 — Gérer l'absence du fichier (rappel chapitre 13)

Un fichier peut ne pas exister. Trois raisons, toutes normales :

1. c'est le **premier lancement** de l'application ;
2. l'utilisateur a effacé les données depuis les réglages du système ;
3. le fichier était dans le **cache**, et le système l'a supprimé (54.16).

Il existe deux façons de traiter ce cas. Elles ne sont pas équivalentes.

**Approche 1 — tester avant de lire.**

```dart
Future<String> lireOuDefaut() async {
  final File fichier = await _fichier();
  if (await fichier.exists()) {
    return fichier.readAsString();
  }
  return '';
}
```

Lisible, mais techniquement sujette à une course : entre le `exists()` et le `readAsString()`, le fichier peut disparaître. En pratique, c'est rarissime sur mobile.

**Approche 2 — tenter et attraper.**

```dart
Future<String> lireOuDefaut() async {
  final File fichier = await _fichier();
  try {
    return await fichier.readAsString();
  } on PathNotFoundException {
    return '';
  } on FileSystemException catch (e) {
    debugPrint('Lecture impossible : ${e.message}');
    return '';
  }
}
```

C'est l'approche robuste, et celle du chapitre 13. Notez la hiérarchie :

```text
  Exception
      │
      └── IOException
              │
              └── FileSystemException      « quelque chose a échoué sur le disque »
                      │
                      └── PathNotFoundException   « le fichier n'existe pas »
```

`PathNotFoundException` **hérite** de `FileSystemException`. Il faut donc l'attraper **en premier**, sinon le `catch` plus général l'intercepte et le cas particulier n'est jamais atteint. C'est exactement la règle d'ordre des `on ... catch` du chapitre 13.

Voici un programme complet qui illustre les trois situations : fichier présent, fichier absent, fichier corrompu.

```dart
import 'dart:convert';
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:path_provider/path_provider.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const ApplicationRobuste());
}

class ApplicationRobuste extends StatelessWidget {
  const ApplicationRobuste({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Lecture robuste',
      theme: ThemeData(
        colorSchemeSeed: Colors.cyan,
        useMaterial3: true,
      ),
      home: const PageRobuste(),
    );
  }
}

class ResultatLecture {
  const ResultatLecture(this.etat, this.donnees);

  final String etat;
  final Map<String, dynamic> donnees;
}

class PageRobuste extends StatefulWidget {
  const PageRobuste({super.key});

  @override
  State<PageRobuste> createState() => _PageRobusteState();
}

class _PageRobusteState extends State<PageRobuste> {
  ResultatLecture? _resultat;

  static const Map<String, dynamic> _defaut = <String, dynamic>{
    'pseudo': 'Aventurier',
    'niveau': 1,
  };

  Future<File> _fichier() async {
    final Directory dossier = await getApplicationDocumentsDirectory();
    return File(p.join(dossier.path, 'sauvegarde.json'));
  }

  Future<void> _lire() async {
    final File fichier = await _fichier();
    ResultatLecture resultat;
    try {
      final String brut = await fichier.readAsString();
      final Object? decode = jsonDecode(brut);
      if (decode is! Map<String, dynamic>) {
        throw const FormatException('La racine n\'est pas un objet JSON.');
      }
      resultat = ResultatLecture('Fichier lu correctement.', decode);
    } on PathNotFoundException {
      resultat = const ResultatLecture(
        'Fichier absent : valeurs par défaut utilisées.',
        _defaut,
      );
    } on FormatException catch (e) {
      await fichier.delete();
      resultat = ResultatLecture(
        'Fichier corrompu (${e.message}) : effacé, défauts utilisés.',
        _defaut,
      );
    } on FileSystemException catch (e) {
      resultat = ResultatLecture(
        'Erreur disque : ${e.message}. Défauts utilisés.',
        _defaut,
      );
    }
    if (!mounted) return;
    setState(() => _resultat = resultat);
  }

  Future<void> _ecrireValide() async {
    final File fichier = await _fichier();
    await fichier.writeAsString(
      jsonEncode(<String, dynamic>{'pseudo': 'Aria', 'niveau': 7}),
    );
    await _lire();
  }

  Future<void> _ecrireCorrompu() async {
    final File fichier = await _fichier();
    await fichier.writeAsString('{"pseudo": "Aria", "niv');
    await _lire();
  }

  Future<void> _supprimer() async {
    final File fichier = await _fichier();
    if (await fichier.exists()) await fichier.delete();
    await _lire();
  }

  @override
  Widget build(BuildContext context) {
    final ResultatLecture? r = _resultat;
    return Scaffold(
      appBar: AppBar(title: const Text('Lecture robuste')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: <Widget>[
                    Text(r?.etat ?? 'Appuyez sur « Lire ».'),
                    const SizedBox(height: 8),
                    Text(
                      '${r?.donnees ?? {}}',
                      style: const TextStyle(fontFamily: 'monospace'),
                    ),
                  ],
                ),
              ),
            ),
            const SizedBox(height: 24),
            FilledButton(onPressed: _lire, child: const Text('Lire')),
            const SizedBox(height: 8),
            OutlinedButton(
              onPressed: _ecrireValide,
              child: const Text('Écrire un fichier valide'),
            ),
            const SizedBox(height: 8),
            OutlinedButton(
              onPressed: _ecrireCorrompu,
              child: const Text('Écrire un fichier corrompu'),
            ),
            const SizedBox(height: 8),
            TextButton(
              onPressed: _supprimer,
              child: const Text('Supprimer le fichier'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :** trois messages différents selon le bouton pressé.

```text
Fichier lu correctement.
{pseudo: Aria, niveau: 7}

Fichier absent : valeurs par défaut utilisées.
{pseudo: Aventurier, niveau: 1}

Fichier corrompu (Unexpected end of input) : effacé, défauts utilisés.
{pseudo: Aventurier, niveau: 1}
```

> La règle de conception à retenir : **une donnée locale illisible ne doit jamais empêcher l'application de démarrer.** Vous effacez, vous repartez des défauts, et éventuellement vous prévenez l'utilisateur. Vous ne plantez pas.

---

## 54.20 — Sauvegarder une liste d'objets dans un fichier JSON

Nous combinons maintenant tout ce qui précède : les modèles du chapitre 17, `path_provider`, `dart:io` et la gestion d'erreurs.

L'objectif : un catalogue de potions, persisté dans un fichier `potions.json`.

```dart
import 'dart:convert';
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:path_provider/path_provider.dart';

// ─────────────────────────────────────────────────────────────
//  lib/models/potion.dart
// ─────────────────────────────────────────────────────────────

class Potion {
  const Potion({
    required this.id,
    required this.nom,
    required this.soin,
    required this.prix,
  });

  final String id;
  final String nom;
  final int soin;
  final double prix;

  Map<String, dynamic> toJson() => <String, dynamic>{
        'id': id,
        'nom': nom,
        'soin': soin,
        'prix': prix,
      };

  factory Potion.fromJson(Map<String, dynamic> json) => Potion(
        id: json['id'] as String,
        nom: json['nom'] as String? ?? 'Sans nom',
        soin: json['soin'] as int? ?? 0,
        prix: (json['prix'] as num?)?.toDouble() ?? 0,
      );
}

// ─────────────────────────────────────────────────────────────
//  lib/services/potion_file_store.dart
// ─────────────────────────────────────────────────────────────

class PotionFileStore {
  static const String _nomFichier = 'potions.json';

  Future<File> _fichier() async {
    final Directory dossier = await getApplicationDocumentsDirectory();
    return File(p.join(dossier.path, _nomFichier));
  }

  Future<List<Potion>> lireTout() async {
    final File fichier = await _fichier();
    try {
      final String brut = await fichier.readAsString();
      if (brut.trim().isEmpty) return <Potion>[];
      final Object? decode = jsonDecode(brut);
      if (decode is! List<dynamic>) return <Potion>[];
      return decode
          .whereType<Map<String, dynamic>>()
          .map(Potion.fromJson)
          .toList();
    } on PathNotFoundException {
      return <Potion>[];
    } on FormatException {
      await fichier.delete();
      return <Potion>[];
    }
  }

  Future<void> ecrireTout(List<Potion> potions) async {
    final File fichier = await _fichier();
    final String contenu = jsonEncode(
      potions.map((Potion p) => p.toJson()).toList(),
    );
    // Écriture atomique : on écrit à côté, puis on renomme.
    final File temporaire = File('${fichier.path}.tmp');
    await temporaire.writeAsString(contenu, flush: true);
    await temporaire.rename(fichier.path);
  }

  Future<String> chemin() async => (await _fichier()).path;
}

// ─────────────────────────────────────────────────────────────
//  lib/main.dart
// ─────────────────────────────────────────────────────────────

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const ApplicationPotions());
}

class ApplicationPotions extends StatelessWidget {
  const ApplicationPotions({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Potions',
      theme: ThemeData(
        colorSchemeSeed: Colors.purple,
        useMaterial3: true,
      ),
      home: const PagePotions(),
    );
  }
}

class PagePotions extends StatefulWidget {
  const PagePotions({super.key});

  @override
  State<PagePotions> createState() => _PagePotionsState();
}

class _PagePotionsState extends State<PagePotions> {
  final PotionFileStore _store = PotionFileStore();
  List<Potion> _potions = <Potion>[];
  bool _charge = false;
  int _compteur = 0;

  @override
  void initState() {
    super.initState();
    _charger();
  }

  Future<void> _charger() async {
    final List<Potion> potions = await _store.lireTout();
    if (!mounted) return;
    setState(() {
      _potions = potions;
      _compteur = potions.length;
      _charge = true;
    });
  }

  Future<void> _ajouter() async {
    _compteur++;
    final Potion nouvelle = Potion(
      id: DateTime.now().microsecondsSinceEpoch.toString(),
      nom: 'Potion n°$_compteur',
      soin: 10 * _compteur,
      prix: 4.5 * _compteur,
    );
    final List<Potion> mises = <Potion>[..._potions, nouvelle];
    await _store.ecrireTout(mises);
    if (!mounted) return;
    setState(() => _potions = mises);
  }

  Future<void> _supprimer(String id) async {
    final List<Potion> mises =
        _potions.where((Potion p) => p.id != id).toList();
    await _store.ecrireTout(mises);
    if (!mounted) return;
    setState(() => _potions = mises);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Potions (${_potions.length})'),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _ajouter,
        child: const Icon(Icons.add),
      ),
      body: !_charge
          ? const Center(child: CircularProgressIndicator())
          : _potions.isEmpty
              ? const Center(child: Text('Aucune potion. Appuyez sur +.'))
              : ListView.separated(
                  itemCount: _potions.length,
                  separatorBuilder: (_, __) => const Divider(height: 1),
                  itemBuilder: (BuildContext context, int index) {
                    final Potion potion = _potions[index];
                    return ListTile(
                      leading: const Icon(Icons.local_drink),
                      title: Text(potion.nom),
                      subtitle: Text(
                        'Soin ${potion.soin} — ${potion.prix.toStringAsFixed(2)} pièces',
                      ),
                      trailing: IconButton(
                        icon: const Icon(Icons.delete_outline),
                        onPressed: () => _supprimer(potion.id),
                      ),
                    );
                  },
                ),
    );
  }
}
```

**Résultat :** vous ajoutez trois potions, vous relancez l'application, les trois potions sont là. Le fichier contient :

```json
[
  {"id": "1773487261000123", "nom": "Potion n°1", "soin": 10, "prix": 4.5},
  {"id": "1773487263000456", "nom": "Potion n°2", "soin": 20, "prix": 9.0},
  {"id": "1773487265000789", "nom": "Potion n°3", "soin": 30, "prix": 13.5}
]
```

Deux techniques importantes apparaissent ici.

**Technique 1 — l'écriture atomique.**

```dart
final File temporaire = File('${fichier.path}.tmp');
await temporaire.writeAsString(contenu, flush: true);
await temporaire.rename(fichier.path);
```

Si l'application est tuée **pendant** un `writeAsString` direct, le fichier final se retrouve tronqué : vous avez perdu vos données. En écrivant dans un fichier temporaire puis en le renommant, le renommage est une opération atomique du système de fichiers : soit l'ancien fichier est intact, soit le nouveau est complet. Jamais un mélange des deux.

Le `flush: true` force l'écriture physique avant que le `Future` ne se termine.

**Technique 2 — `whereType<Map<String, dynamic>>()`.**
`jsonDecode` d'un tableau renvoie une `List<dynamic>`. `whereType` (chapitre 14) filtre et type en une seule opération : les éléments qui ne sont pas des objets JSON sont simplement ignorés au lieu de faire planter la conversion.

---

## 54.21 — Les limites du fichier unique

Le code de la section précédente marche très bien... jusqu'à un certain point. Voici les trois murs que vous rencontrerez.

**Limite 1 — tout ou rien.**

Pour modifier **une seule** potion sur 5 000, il faut :

```text
  1. lire le fichier entier          (5 000 objets → mémoire)
  2. décoder tout le JSON            (coûteux en CPU)
  3. modifier un élément
  4. ré-encoder tout le JSON
  5. réécrire le fichier entier      (5 000 objets → disque)
```

Cinq étapes, dont quatre inutiles. Une base de données ferait :

```sql
UPDATE potion SET soin = 40 WHERE id = 'p-123';
```

Une seule ligne touchée.

**Limite 2 — pas de requêtes.**

« Donne-moi les 20 potions les moins chères qui soignent plus de 30 points. » Avec un fichier, vous devez charger les 5 000 objets en mémoire puis filtrer en Dart. Avec SQL :

```sql
SELECT * FROM potion WHERE soin > 30 ORDER BY prix ASC LIMIT 20;
```

La base ne charge que 20 objets, et s'appuie sur un index pour trouver les bons.

**Limite 3 — écritures concurrentes.**

Si deux parties de votre application écrivent le fichier « en même temps » (deux `Future` non attendus), la dernière écriture écrase la première. Il n'y a aucun verrou. Une base de données gère cela nativement, avec des transactions.

Voici le tableau de décision final entre fichier et base.

| Critère | Fichier JSON | Base SQLite |
| --- | --- | --- |
| Nombre d'objets | Jusqu'à ~1 000 | Des millions |
| Modification unitaire | Réécrit tout | Touche une ligne |
| Recherche / tri / filtre | En mémoire, en Dart | En SQL, avec index |
| Écritures concurrentes | Dangereux | Transactions |
| Format lisible à la main | Oui | Non |
| Facilité d'export | Excellente | Moyenne |
| Complexité de mise en place | Très faible | Moyenne |

> La bascule se fait généralement autour du millier d'objets, ou dès que vous écrivez votre première boucle `where` sur une liste chargée depuis un fichier.

---

## 54.22 — `sqflite` : une vraie base de données

`sqflite` est le paquet qui expose **SQLite** à Flutter. SQLite est un moteur de base de données relationnel embarqué : il n'y a pas de serveur, pas de configuration réseau, pas de mot de passe. Toute la base tient dans **un seul fichier** sur l'appareil.

C'est le moteur de base de données le plus déployé au monde : il est présent dans tous les Android, tous les iPhone, tous les navigateurs, la plupart des avions et des voitures.

```text
   ┌──────────────────────────────────────────────────────┐
   │  Votre application Flutter                           │
   │                                                      │
   │    Dart  ──►  sqflite  ──►  code natif  ──►  SQLite  │
   │                                                      │
   └──────────────────────────────────────────────────────┘
                                                    │
                                                    v
                                            un fichier .db
                                          sur le disque local
```

Le vocabulaire relationnel, en trois mots :

| Mot | Équivalent Dart |
| --- | --- |
| **Table** | Une `List<Potion>` |
| **Ligne** (*row*) | Un objet `Potion` |
| **Colonne** | Un champ de la classe (`nom`, `soin`, `prix`) |

Une différence essentielle avec un fichier JSON : dans une base, **vous décrivez la structure avant de stocker les données**. C'est ce qu'on appelle le *schéma*. Cette contrainte apparaît d'abord comme une lourdeur ; elle est en réalité ce qui rend les requêtes rapides et les données fiables.

---

## 54.23 — Installation et ouverture d'une base

```text
flutter pub add sqflite
flutter pub add path
```

```yaml
dependencies:
  flutter:
    sdk: flutter
  sqflite: ^2.4.3
  path: ^1.9.1
```

`path` fournit `join`, indispensable pour construire le chemin du fichier de base.

L'ouverture se fait avec `openDatabase` :

```dart
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> ouvrir() async {
  final String dossier = await getDatabasesPath();
  final String chemin = p.join(dossier, 'jeu.db');

  return openDatabase(
    chemin,
    version: 1,
    onCreate: (Database db, int version) async {
      await db.execute('''
        CREATE TABLE potion (
          id      TEXT PRIMARY KEY,
          nom     TEXT NOT NULL,
          soin    INTEGER NOT NULL,
          prix    REAL NOT NULL
        )
      ''');
    },
  );
}
```

Trois éléments à comprendre.

**`getDatabasesPath()`** renvoie le dossier réservé aux bases de données par la plateforme. Sur Android, c'est `/data/data/<applicationId>/databases`. Sur iOS et macOS, c'est le dossier Documents. N'écrivez jamais ce chemin à la main.

**`version: 1`** est le numéro de schéma. Il pilote `onCreate` et `onUpgrade` (54.24).

**`onCreate`** n'est appelé **qu'une seule fois** : lors de la toute première ouverture, quand le fichier n'existe pas encore. Aux ouvertures suivantes, le fichier est là, `onCreate` est ignoré.

C'est le piège numéro un de `sqflite` :

> Vous modifiez le `CREATE TABLE` dans `onCreate`, vous relancez, et rien ne change. Normal : la base existe déjà. Il faut soit désinstaller l'application, soit incrémenter `version` et écrire un `onUpgrade`.

`openDatabase` accepte d'autres rappels, tous utiles :

| Paramètre | Appelé quand | Usage typique |
| --- | --- | --- |
| `onCreate` | Le fichier n'existe pas | Créer les tables |
| `onUpgrade` | `version` > version du fichier | Migrer le schéma |
| `onDowngrade` | `version` < version du fichier | Rare ; `onDowngradeDeleteAndCreate` |
| `onOpen` | À chaque ouverture | Journalisation, `PRAGMA` |
| `onConfigure` | Avant tout le reste | Activer les clés étrangères |

L'activation des clés étrangères mérite une mention, car elle est **désactivée par défaut** dans SQLite :

```dart
onConfigure: (Database db) async {
  await db.execute('PRAGMA foreign_keys = ON');
},
```

---

## 54.24 — `onCreate` et les migrations (`version`, `onUpgrade`)

Votre application vit. En version 1, une potion a un `nom`, un `soin`, un `prix`. En version 2, vous voulez ajouter une `rarete`. En version 3, une table `arme`.

Le problème : vos utilisateurs ont déjà des bases en version 1, remplies de leurs données. Vous ne pouvez pas les effacer.

C'est le rôle du couple `version` / `onUpgrade`.

```text
   Fichier .db sur l'appareil          Code de l'application
   ──────────────────────────          ─────────────────────
        user_version = 1        <────       version: 3

              │
              v
   onUpgrade(db, oldVersion: 1, newVersion: 3)
              │
              v
   if (oldVersion < 2) → ALTER TABLE potion ADD COLUMN rarete ...
   if (oldVersion < 3) → CREATE TABLE arme ...
              │
              v
        user_version = 3
```

Le patron correct est une **suite de `if` sans `else`**, chacun traitant une marche de l'escalier :

```dart
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> ouvrir() async {
  final String chemin = p.join(await getDatabasesPath(), 'jeu.db');

  return openDatabase(
    chemin,
    version: 3,
    onCreate: (Database db, int version) async {
      // Un nouvel utilisateur : on crée directement le schéma final.
      await db.execute('''
        CREATE TABLE potion (
          id      TEXT PRIMARY KEY,
          nom     TEXT NOT NULL,
          soin    INTEGER NOT NULL,
          prix    REAL NOT NULL,
          rarete  TEXT NOT NULL DEFAULT 'commune'
        )
      ''');
      await db.execute('''
        CREATE TABLE arme (
          id      TEXT PRIMARY KEY,
          nom     TEXT NOT NULL,
          degats  INTEGER NOT NULL
        )
      ''');
    },
    onUpgrade: (Database db, int oldVersion, int newVersion) async {
      if (oldVersion < 2) {
        await db.execute(
          "ALTER TABLE potion ADD COLUMN rarete TEXT NOT NULL DEFAULT 'commune'",
        );
      }
      if (oldVersion < 3) {
        await db.execute('''
          CREATE TABLE arme (
            id      TEXT PRIMARY KEY,
            nom     TEXT NOT NULL,
            degats  INTEGER NOT NULL
          )
        ''');
      }
    },
  );
}
```

Pourquoi des `if` séparés et pas un `switch` ? Parce qu'un utilisateur peut sauter des versions. Quelqu'un qui n'a pas ouvert l'application depuis un an passe directement de 1 à 3 : les deux `if` doivent s'exécuter, dans l'ordre.

Quatre règles de migration à ne jamais transgresser.

| Règle | Raison |
| --- | --- |
| Toute modification du schéma incrémente `version` | Sinon `onUpgrade` n'est jamais appelé |
| `onCreate` produit le schéma **final**, pas le schéma v1 | Un nouvel utilisateur ne passe pas par les migrations |
| Une colonne ajoutée a une valeur par défaut ou est nullable | Les lignes existantes n'ont pas cette donnée |
| On ne modifie jamais une migration déjà livrée | Des utilisateurs l'ont déjà exécutée |

> SQLite ne sait pas supprimer ni renommer une colonne aussi librement que d'autres moteurs. Pour un remaniement lourd, le patron est : créer une nouvelle table, `INSERT INTO nouvelle SELECT ... FROM ancienne`, supprimer l'ancienne, renommer la nouvelle.

Pendant le développement, quand votre schéma bouge toutes les dix minutes, deux raccourcis sont acceptables :

```dart
// 1. Repartir de zéro à chaque ouverture (DÉVELOPPEMENT UNIQUEMENT)
await deleteDatabase(chemin);

// 2. Ou déléguer à sqflite
onDowngrade: onDatabaseDowngradeDelete,
```

Ne livrez jamais la première ligne en production : elle efface les données de vos utilisateurs à chaque lancement.

---

## 54.25 — Créer une table

Le `CREATE TABLE` mérite qu'on s'y arrête, parce que c'est là que se joue la qualité de vos données.

```sql
CREATE TABLE potion (
  id      TEXT    PRIMARY KEY,
  nom     TEXT    NOT NULL,
  soin    INTEGER NOT NULL DEFAULT 0,
  prix    REAL    NOT NULL,
  rarete  TEXT    NOT NULL DEFAULT 'commune',
  cree_le INTEGER NOT NULL
)
```

SQLite ne connaît que **cinq types de stockage**. C'est peu, et il faut savoir à quoi cela correspond en Dart.

| Type SQLite | Type Dart | Remarque |
| --- | --- | --- |
| `INTEGER` | `int` | Sert aussi aux booléens (0 / 1) et aux dates (millisecondes) |
| `REAL` | `double` | Nombre à virgule flottante |
| `TEXT` | `String` | UTF-8 |
| `BLOB` | `Uint8List` | Octets bruts : image, fichier |
| `NULL` | `null` | Absence de valeur |

**Il n'y a ni type booléen ni type date.** Les conventions universelles sont :

| Donnée Dart | Colonne | Conversion |
| --- | --- | --- |
| `bool` | `INTEGER` | `valeur ? 1 : 0` puis `entier == 1` |
| `DateTime` | `INTEGER` | `.millisecondsSinceEpoch` puis `DateTime.fromMillisecondsSinceEpoch` |
| `DateTime` (variante lisible) | `TEXT` | `.toIso8601String()` puis `DateTime.parse` |
| `enum` | `TEXT` | `.name` puis recherche par nom |
| `List<String>` | `TEXT` | `jsonEncode` puis `jsonDecode`, ou une table dédiée |

Les contraintes de colonne les plus utiles :

| Contrainte | Effet |
| --- | --- |
| `PRIMARY KEY` | Identifiant unique de la ligne |
| `INTEGER PRIMARY KEY AUTOINCREMENT` | Identifiant entier généré automatiquement |
| `NOT NULL` | Interdit `NULL` |
| `UNIQUE` | Interdit les doublons sur cette colonne |
| `DEFAULT valeur` | Valeur utilisée si la colonne n'est pas fournie |
| `CHECK (condition)` | Refuse les lignes qui violent la condition |
| `REFERENCES autre(id)` | Clé étrangère |

Un exemple avec deux tables liées :

```sql
CREATE TABLE joueur (
  id     INTEGER PRIMARY KEY AUTOINCREMENT,
  pseudo TEXT    NOT NULL UNIQUE,
  niveau INTEGER NOT NULL DEFAULT 1 CHECK (niveau >= 1)
);

CREATE TABLE objet (
  id        INTEGER PRIMARY KEY AUTOINCREMENT,
  joueur_id INTEGER NOT NULL,
  nom       TEXT    NOT NULL,
  quantite  INTEGER NOT NULL DEFAULT 1,
  FOREIGN KEY (joueur_id) REFERENCES joueur (id) ON DELETE CASCADE
);

CREATE INDEX idx_objet_joueur ON objet (joueur_id);
```

Le `ON DELETE CASCADE` signifie : si on supprime un joueur, ses objets disparaissent avec lui. Il ne fonctionne que si vous avez activé `PRAGMA foreign_keys = ON` (54.23).

L'`INDEX` est l'équivalent d'une table des matières : sans lui, chercher tous les objets d'un joueur oblige SQLite à parcourir toute la table. Avec lui, c'est immédiat. **Indexez toute colonne sur laquelle vous filtrez souvent.**

---

## 54.26 — `insert()`

```dart
final int idInsere = await db.insert(
  'potion',
  <String, Object?>{
    'id': 'p-001',
    'nom': 'Potion de soin',
    'soin': 30,
    'prix': 12.5,
  },
);
```

`insert` prend le nom de la table et une `Map<String, Object?>` dont les clés sont les noms des colonnes. Il renvoie l'identifiant de la ligne insérée (le `rowid` interne de SQLite).

Le paramètre `conflictAlgorithm` décide de ce qui se passe si la ligne existe déjà (violation de `PRIMARY KEY` ou de `UNIQUE`) :

| Valeur | Comportement |
| --- | --- |
| `ConflictAlgorithm.abort` | Défaut : lève une exception |
| `ConflictAlgorithm.replace` | Supprime l'ancienne ligne et insère la nouvelle |
| `ConflictAlgorithm.ignore` | N'insère rien, ne lève rien |
| `ConflictAlgorithm.fail` | Échoue sans annuler les opérations précédentes |
| `ConflictAlgorithm.rollback` | Annule toute la transaction |

`replace` est le plus utilisé, car il transforme `insert` en « insère ou met à jour » :

```dart
await db.insert(
  'potion',
  potion.toMap(),
  conflictAlgorithm: ConflictAlgorithm.replace,
);
```

Pour insérer beaucoup de lignes, n'enchaînez pas les `insert` : utilisez un `batch`, qui n'effectue qu'un seul aller-retour vers le code natif.

```dart
final Batch lot = db.batch();
for (final Potion potion in potions) {
  lot.insert(
    'potion',
    potion.toMap(),
    conflictAlgorithm: ConflictAlgorithm.replace,
  );
}
await lot.commit(noResult: true);
```

Sur 1 000 lignes, la différence de performance est d'un ordre de grandeur. Le `noResult: true` indique que vous ne voulez pas récupérer les 1 000 identifiants, ce qui économise encore du temps.

---

## 54.27 — `query()`

```dart
final List<Map<String, Object?>> lignes = await db.query('potion');
```

`query` renvoie une **liste de `Map`**, une par ligne, dont les clés sont les noms des colonnes.

```text
[
  {id: p-001, nom: Potion de soin, soin: 30, prix: 12.5},
  {id: p-002, nom: Élixir, soin: 80, prix: 45.0}
]
```

Tous les paramètres de `query` correspondent à une clause SQL :

| Paramètre Dart | Clause SQL | Exemple |
| --- | --- | --- |
| `columns` | `SELECT` | `<String>['id', 'nom']` |
| `where` | `WHERE` | `'soin > ?'` |
| `whereArgs` | valeurs des `?` | `<Object?>[30]` |
| `groupBy` | `GROUP BY` | `'rarete'` |
| `having` | `HAVING` | `'COUNT(*) > 2'` |
| `orderBy` | `ORDER BY` | `'prix ASC'` |
| `limit` | `LIMIT` | `20` |
| `offset` | `OFFSET` | `40` |
| `distinct` | `SELECT DISTINCT` | `true` |

Exemple complet :

```dart
final List<Map<String, Object?>> lignes = await db.query(
  'potion',
  columns: <String>['id', 'nom', 'prix'],
  where: 'soin > ? AND rarete = ?',
  whereArgs: <Object?>[30, 'rare'],
  orderBy: 'prix ASC',
  limit: 20,
);
```

équivaut exactement à :

```sql
SELECT id, nom, prix
FROM potion
WHERE soin > 30 AND rarete = 'rare'
ORDER BY prix ASC
LIMIT 20;
```

Quand la requête devient trop complexe pour ces paramètres (jointures, sous-requêtes, fonctions d'agrégat), passez à `rawQuery` :

```dart
final List<Map<String, Object?>> lignes = await db.rawQuery('''
  SELECT j.pseudo, COUNT(o.id) AS nb_objets
  FROM joueur j
  LEFT JOIN objet o ON o.joueur_id = j.id
  GROUP BY j.id
  ORDER BY nb_objets DESC
''');
```

Pour un simple comptage, `sqflite` fournit un utilitaire :

```dart
final int? total = Sqflite.firstIntValue(
  await db.rawQuery('SELECT COUNT(*) FROM potion'),
);
```

> Attention : les `Map` renvoyées par `query` sont **en lecture seule**. Tenter d'en modifier une lève une `UnsupportedError`. Si vous devez la modifier, faites-en une copie : `Map<String, Object?>.from(ligne)`.

---

## 54.28 — `update()` et `delete()`

```dart
final int nbModifiees = await db.update(
  'potion',
  <String, Object?>{'prix': 9.99, 'rarete': 'commune'},
  where: 'id = ?',
  whereArgs: <Object?>['p-001'],
);

final int nbSupprimees = await db.delete(
  'potion',
  where: 'soin < ?',
  whereArgs: <Object?>[10],
);
```

Les deux renvoient le **nombre de lignes affectées**. C'est très utile :

```dart
if (nbModifiees == 0) {
  throw StateError('Aucune potion avec cet identifiant.');
}
```

Un avertissement, en majuscules parce qu'il coûte cher :

> **`update` et `delete` SANS clause `where` s'appliquent à TOUTE la table.**

```dart
await db.delete('potion');           // vide la table entière
await db.update('potion', {'prix': 0}); // met tout à zéro
```

Ce n'est pas un bug, c'est la sémantique de SQL. Prenez l'habitude d'écrire le `where` **avant** le reste de la ligne. Et pour les opérations destructrices d'un écran de réglages, exigez une confirmation.

Voici le tableau complet des quatre opérations, dites CRUD.

| Opération | Méthode `sqflite` | SQL | Retour |
| --- | --- | --- | --- |
| **C**reate | `insert()` | `INSERT INTO` | `int` (rowid) |
| **R**ead | `query()` | `SELECT` | `List<Map<String, Object?>>` |
| **U**pdate | `update()` | `UPDATE` | `int` (lignes modifiées) |
| **D**elete | `delete()` | `DELETE FROM` | `int` (lignes supprimées) |

---

## 54.29 — Les requêtes avec conditions et arguments

Voici la section la plus importante de toute la partie SQL. Elle tient en une règle :

> **Ne construisez JAMAIS une requête SQL par concaténation de chaînes. Utilisez toujours des `?` et `whereArgs`.**

Le code fautif :

```dart
// À NE JAMAIS FAIRE
final String saisie = champRecherche.text;
final lignes = await db.rawQuery(
  "SELECT * FROM potion WHERE nom = '$saisie'",
);
```

Si l'utilisateur saisit `Potion`, la requête envoyée est :

```sql
SELECT * FROM potion WHERE nom = 'Potion';
```

Correct. Mais s'il saisit `x'; DROP TABLE potion; --`, la requête devient :

```sql
SELECT * FROM potion WHERE nom = 'x'; DROP TABLE potion; --';
```

La table est détruite. C'est une **injection SQL**. Sur une application locale, l'attaquant est l'utilisateur lui-même, ce qui limite les dégâts ; mais le même réflexe appliqué côté serveur est l'une des failles les plus exploitées au monde.

Le code correct :

```dart
final lignes = await db.query(
  'potion',
  where: 'nom = ?',
  whereArgs: <Object?>[champRecherche.text],
);
```

Le `?` n'est pas un remplacement de texte. La valeur est transmise **séparément** au moteur, qui la traite comme une donnée, jamais comme du code. Il devient littéralement impossible d'injecter du SQL.

Deuxième avantage, souvent oublié : les `?` gèrent les apostrophes pour vous. Une potion nommée `Élixir d'Aria` casserait la requête concaténée. Avec `?`, aucun problème.

Quelques formes de conditions utiles :

```dart
// Égalité multiple
where: 'rarete = ? AND soin >= ?', whereArgs: <Object?>['rare', 50]

// Intervalle
where: 'prix BETWEEN ? AND ?', whereArgs: <Object?>[10.0, 50.0]

// Recherche partielle (insensible à la casse pour l'ASCII)
where: 'nom LIKE ?', whereArgs: <Object?>['%soin%']

// Valeur nulle : PAS de ?, car « = NULL » est toujours faux en SQL
where: 'description IS NULL'

// Liste de valeurs : autant de ? que d'éléments
final ids = <String>['p-001', 'p-002', 'p-003'];
final marques = List<String>.filled(ids.length, '?').join(', ');
final lignes = await db.query(
  'potion',
  where: 'id IN ($marques)',
  whereArgs: ids,
);
```

La dernière forme mérite une explication : le nombre de `?` doit correspondre au nombre de valeurs. Comme il est variable, on le construit. Ici la chaîne interpolée `$marques` ne contient que des `?` produits par votre code, jamais une saisie utilisateur : c'est sûr.

Trois pièges de débutant sur le `where` :

| Écriture | Problème |
| --- | --- |
| `where: 'nom = $nom'` | Injection SQL |
| `where: "nom = '?'"` | Le `?` entre apostrophes est un texte littéral, pas un paramètre |
| `where: 'age = ?'` avec `whereArgs: [null]` | `= NULL` est toujours faux : utilisez `IS NULL` |

---

## 54.30 — De la ligne à l'objet Dart et retour

`query` renvoie des `Map<String, Object?>`. Votre application, elle, veut des objets typés. La conversion se fait dans le modèle, exactement comme le JSON du chapitre 17, avec deux méthodes symétriques.

```dart
class Potion {
  const Potion({
    required this.id,
    required this.nom,
    required this.soin,
    required this.prix,
    required this.favorite,
    required this.creeLe,
    this.rarete = Rarete.commune,
  });

  final String id;
  final String nom;
  final int soin;
  final double prix;
  final bool favorite;
  final DateTime creeLe;
  final Rarete rarete;

  // Objet Dart  ──►  ligne SQL
  Map<String, Object?> toMap() => <String, Object?>{
        'id': id,
        'nom': nom,
        'soin': soin,
        'prix': prix,
        'favorite': favorite ? 1 : 0,
        'cree_le': creeLe.millisecondsSinceEpoch,
        'rarete': rarete.name,
      };

  // Ligne SQL  ──►  objet Dart
  factory Potion.fromMap(Map<String, Object?> map) => Potion(
        id: map['id']! as String,
        nom: map['nom']! as String,
        soin: map['soin']! as int,
        prix: (map['prix']! as num).toDouble(),
        favorite: (map['favorite'] as int? ?? 0) == 1,
        creeLe: DateTime.fromMillisecondsSinceEpoch(
          map['cree_le'] as int? ?? 0,
        ),
        rarete: Rarete.values.firstWhere(
          (Rarete r) => r.name == map['rarete'],
          orElse: () => Rarete.commune,
        ),
      );
}

enum Rarete { commune, rare, epique, legendaire }
```

Quatre conversions apparaissent, et ce sont toujours les mêmes.

| Champ Dart | Colonne | Aller | Retour |
| --- | --- | --- | --- |
| `bool favorite` | `INTEGER` | `favorite ? 1 : 0` | `entier == 1` |
| `DateTime creeLe` | `INTEGER` | `.millisecondsSinceEpoch` | `DateTime.fromMillisecondsSinceEpoch` |
| `Rarete rarete` | `TEXT` | `.name` | `firstWhere(... orElse: ...)` |
| `double prix` | `REAL` | direct | `(... as num).toDouble()` |

Le `orElse` de `firstWhere` est **indispensable** : si une future version de votre application supprime une valeur de l'énumération, les lignes existantes contiennent encore l'ancien nom. Sans `orElse`, `firstWhere` lève une `StateError` et l'application plante au démarrage.

La conversion d'une liste entière tient alors en une ligne (chapitre 14) :

```dart
final List<Map<String, Object?>> lignes = await db.query('potion');
final List<Potion> potions = lignes.map(Potion.fromMap).toList();
```

---

## 54.31 — Le patron DAO

Si vous laissez du SQL dans vos widgets, votre projet devient rapidement illisible. Le remède est le **DAO** (*Data Access Object*) : une classe dont l'unique rôle est de traduire entre le monde SQL et le monde Dart.

```text
   ┌───────────────┐    List<Potion>    ┌───────────────┐   SQL    ┌────────┐
   │   Widgets     │ ◄────────────────► │  PotionDao    │ ◄──────► │ SQLite │
   │  (interface)  │                    │  (traduction) │          │        │
   └───────────────┘                    └───────────────┘          └────────┘

      ne connaît                          seul endroit du projet
      aucun SQL                            où l'on écrit du SQL
```

Trois règles :

1. **Le DAO ne renvoie jamais de `Map`.** Il renvoie des objets Dart, ou des types simples.
2. **Le DAO ne connaît pas les widgets.** Aucun `import 'package:flutter/material.dart'`.
3. **Le SQL n'existe nulle part ailleurs.** Si vous cherchez `SELECT` dans le projet, vous ne devez le trouver que dans les DAO.

Voici l'implémentation complète, avec le gestionnaire d'ouverture de base.

```dart
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

// ─────────────────────────────────────────────────────────────
//  lib/data/app_database.dart
// ─────────────────────────────────────────────────────────────

class AppDatabase {
  AppDatabase._();

  static final AppDatabase instance = AppDatabase._();

  Database? _db;

  Future<Database> get database async {
    return _db ??= await _ouvrir();
  }

  Future<Database> _ouvrir() async {
    final String chemin = p.join(await getDatabasesPath(), 'jeu.db');
    return openDatabase(
      chemin,
      version: 1,
      onConfigure: (Database db) async {
        await db.execute('PRAGMA foreign_keys = ON');
      },
      onCreate: (Database db, int version) async {
        await db.execute('''
          CREATE TABLE potion (
            id       TEXT    PRIMARY KEY,
            nom      TEXT    NOT NULL,
            soin     INTEGER NOT NULL,
            prix     REAL    NOT NULL,
            favorite INTEGER NOT NULL DEFAULT 0,
            cree_le  INTEGER NOT NULL,
            rarete   TEXT    NOT NULL DEFAULT 'commune'
          )
        ''');
        await db.execute('CREATE INDEX idx_potion_soin ON potion (soin)');
      },
    );
  }

  Future<void> fermer() async {
    await _db?.close();
    _db = null;
  }
}

// ─────────────────────────────────────────────────────────────
//  lib/data/potion_dao.dart
// ─────────────────────────────────────────────────────────────

class PotionDao {
  PotionDao(this._db);

  final Database _db;

  static const String _table = 'potion';

  Future<List<Potion>> toutes({String? tri}) async {
    final List<Map<String, Object?>> lignes = await _db.query(
      _table,
      orderBy: tri ?? 'nom ASC',
    );
    return lignes.map(Potion.fromMap).toList();
  }

  Future<Potion?> parId(String id) async {
    final List<Map<String, Object?>> lignes = await _db.query(
      _table,
      where: 'id = ?',
      whereArgs: <Object?>[id],
      limit: 1,
    );
    if (lignes.isEmpty) return null;
    return Potion.fromMap(lignes.first);
  }

  Future<List<Potion>> rechercher(String terme) async {
    final List<Map<String, Object?>> lignes = await _db.query(
      _table,
      where: 'nom LIKE ?',
      whereArgs: <Object?>['%$terme%'],
      orderBy: 'nom ASC',
    );
    return lignes.map(Potion.fromMap).toList();
  }

  Future<List<Potion>> favorites() async {
    final List<Map<String, Object?>> lignes = await _db.query(
      _table,
      where: 'favorite = ?',
      whereArgs: <Object?>[1],
    );
    return lignes.map(Potion.fromMap).toList();
  }

  Future<int> compter() async {
    return Sqflite.firstIntValue(
          await _db.rawQuery('SELECT COUNT(*) FROM $_table'),
        ) ??
        0;
  }

  Future<void> enregistrer(Potion potion) async {
    await _db.insert(
      _table,
      potion.toMap(),
      conflictAlgorithm: ConflictAlgorithm.replace,
    );
  }

  Future<void> enregistrerToutes(List<Potion> potions) async {
    final Batch lot = _db.batch();
    for (final Potion potion in potions) {
      lot.insert(
        _table,
        potion.toMap(),
        conflictAlgorithm: ConflictAlgorithm.replace,
      );
    }
    await lot.commit(noResult: true);
  }

  Future<bool> basculerFavorite(String id) async {
    final Potion? potion = await parId(id);
    if (potion == null) return false;
    final int nouvelle = potion.favorite ? 0 : 1;
    await _db.update(
      _table,
      <String, Object?>{'favorite': nouvelle},
      where: 'id = ?',
      whereArgs: <Object?>[id],
    );
    return nouvelle == 1;
  }

  Future<int> supprimer(String id) async {
    return _db.delete(_table, where: 'id = ?', whereArgs: <Object?>[id]);
  }

  Future<int> viderTout() async {
    return _db.delete(_table);
  }
}
```

Le widget qui l'utilise n'écrit plus une seule ligne de SQL :

```dart
final PotionDao dao = PotionDao(await AppDatabase.instance.database);
final List<Potion> potions = await dao.toutes(tri: 'prix DESC');
```

Le bénéfice le plus concret : si vous migrez un jour de `sqflite` vers `drift` ou `isar`, seul le DAO change. Vos écrans ne bougent pas.

---

## 54.32 — Les transactions

Considérez une opération d'achat : le joueur perd de l'or, et gagne une potion. Deux écritures, dans deux tables.

```dart
await db.update('joueur', <String, Object?>{'or': orRestant},
    where: 'id = ?', whereArgs: <Object?>[1]);
// ← si l'application est tuée ICI
await db.insert('objet', potion.toMap());
```

Si l'application meurt entre les deux lignes, le joueur a payé et n'a rien reçu. La base est dans un état **incohérent**, et rien ne permet de le détecter après coup.

Une **transaction** rend un groupe d'écritures atomique : soit tout réussit, soit rien n'est appliqué.

```dart
await db.transaction((Transaction txn) async {
  await txn.update(
    'joueur',
    <String, Object?>{'or': orRestant},
    where: 'id = ?',
    whereArgs: <Object?>[1],
  );
  await txn.insert('objet', potion.toMap());
});
```

Le mécanisme est simple à retenir :

```text
   db.transaction((txn) async {
     ...          ─────► si TOUT se termine sans exception : COMMIT
     ...
   });             ─────► si une exception est levée      : ROLLBACK
```

Il n'y a ni `commit()` ni `rollback()` à appeler : `sqflite` s'en charge selon que le bloc se termine normalement ou par une exception.

Une règle absolue, et c'est la cause d'interblocage la plus fréquente :

> **À l'intérieur d'une transaction, utilisez `txn`, jamais `db`.**

```dart
// FAUX : interblocage garanti
await db.transaction((Transaction txn) async {
  await db.insert('objet', potion.toMap()); // <-- db, pas txn
});
```

`db` attend que la transaction se termine, la transaction attend que `db` réponde : l'application se fige sans message d'erreur.

Corollaire : une méthode de DAO qui utilise `_db` ne peut pas être appelée depuis une transaction. Le patron correct est de faire circuler un `DatabaseExecutor`, l'interface commune à `Database` et `Transaction` :

```dart
class PotionDao {
  PotionDao(this._db);

  final Database _db;

  Future<void> enregistrer(Potion potion, {DatabaseExecutor? executeur}) async {
    final DatabaseExecutor cible = executeur ?? _db;
    await cible.insert(
      'potion',
      potion.toMap(),
      conflictAlgorithm: ConflictAlgorithm.replace,
    );
  }
}

// Usage hors transaction
await dao.enregistrer(potion);

// Usage dans une transaction
await db.transaction((Transaction txn) async {
  await dao.enregistrer(potion, executeur: txn);
  await dao.enregistrer(autre, executeur: txn);
});
```

Enfin, ne confondez pas transaction et lot (`batch`) :

| Outil | Garantit l'atomicité ? | Optimise les performances ? |
| --- | --- | --- |
| `transaction` | Oui | Un peu |
| `batch` | Non par défaut | Beaucoup |
| `batch` dans une `transaction` | Oui | Beaucoup |

Pour importer 5 000 lignes de façon sûre et rapide, combinez les deux :

```dart
await db.transaction((Transaction txn) async {
  final Batch lot = txn.batch();
  for (final Potion potion in potions) {
    lot.insert('potion', potion.toMap());
  }
  await lot.commit(noResult: true);
});
```

---

## 54.33 — Le stockage sur le Web : ce qui change

Une application Flutter Web tourne dans un navigateur. Le navigateur ne donne pas accès au système de fichiers de l'utilisateur. Tout ce chapitre doit donc être relu à cette lumière.

| Outil | Sur le Web | Mécanisme réel |
| --- | --- | --- |
| `shared_preferences` | Fonctionne | `window.localStorage` |
| `path_provider` | **Ne fonctionne pas** | Aucun système de fichiers |
| `dart:io` (`File`, `Directory`) | **Ne compile pas** | Bibliothèque absente |
| `sqflite` | **Ne fonctionne pas** | `sqflite_common_ffi_web` (expérimental) |
| `flutter_secure_storage` | Fonctionne partiellement | WebCrypto + `localStorage`, HTTPS requis |
| `hive` / `hive_ce` | Fonctionne | IndexedDB |
| `drift` | Fonctionne | WebAssembly + IndexedDB / OPFS |

Trois limites de `localStorage` que vous devez connaître :

1. **Environ 5 Mo** au total par origine. Au-delà, une exception de quota.
2. **Tout est en clair**, et l'utilisateur peut modifier les valeurs à la main depuis les outils de développement.
3. **Effaçable** par un nettoyage de navigateur, ou en navigation privée à la fermeture de l'onglet.

Le problème de compilation le plus courant est celui-ci :

```dart
import 'dart:io'; // Erreur de compilation sur le Web
```

Un `if (kIsWeb)` ne suffit pas : l'`import` lui-même échoue avant même l'exécution. Il faut isoler le code par plateforme. Le patron officiel est l'**import conditionnel**.

```dart
// lib/storage/storage.dart — l'interface commune
abstract class Storage {
  Future<void> ecrire(String cle, String valeur);
  Future<String?> lire(String cle);
}
```

```dart
// lib/storage/storage_factory.dart
import 'storage.dart';
import 'storage_io.dart' if (dart.library.js_interop) 'storage_web.dart';

Storage creerStorage() => creerStorageImpl();
```

Le compilateur choisit `storage_web.dart` si `dart:js_interop` est disponible (donc sur le Web), et `storage_io.dart` sinon. Chacun des deux fichiers fournit sa propre `creerStorageImpl()`.

Pour la grande majorité des projets, la stratégie pragmatique est plus simple :

| Besoin | Solution multi-plateforme |
| --- | --- |
| Réglages | `shared_preferences` : marche partout |
| Secrets | `flutter_secure_storage` : marche partout |
| Données structurées | `drift` ou `hive_ce` : marchent partout |
| Fichiers | Accepter que la fonctionnalité soit désactivée sur le Web |

Voici comment désactiver proprement une fonctionnalité indisponible :

```dart
import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';

void main() => runApp(const ApplicationExport());

class ApplicationExport extends StatelessWidget {
  const ApplicationExport({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Export',
      theme: ThemeData(colorSchemeSeed: Colors.blue, useMaterial3: true),
      home: const PageExport(),
    );
  }
}

class PageExport extends StatelessWidget {
  const PageExport({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Export de la sauvegarde')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            FilledButton.icon(
              onPressed: kIsWeb ? null : () {},
              icon: const Icon(Icons.download),
              label: const Text('Exporter dans un fichier'),
            ),
            const SizedBox(height: 12),
            if (kIsWeb)
              const Text(
                'L\'export de fichier n\'est pas disponible dans le navigateur.',
                textAlign: TextAlign.center,
              ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :** sur mobile et bureau, le bouton est actif. Sur le Web, il est grisé et une explication apparaît. L'utilisateur n'a jamais l'impression d'un bug.

> Pour utiliser `sqflite` sur un **bureau** (Windows, Linux, macOS en développement), ajoutez `sqflite_common_ffi`, appelez `sqfliteFfiInit()` puis `databaseFactory = databaseFactoryFfi;` au début de `main`. Sans cela, `openDatabase` échoue sur ces plateformes.

---

## 54.34 — `flutter_secure_storage` pour les jetons et les secrets

Le chapitre 53 vous a fait appeler une API. Une vraie API demande une authentification, et vous rend un **jeton**. Ce jeton donne accès au compte de l'utilisateur : il ne doit pas traîner en clair.

`flutter_secure_storage` s'appuie sur les mécanismes de sécurité du système.

| Plateforme | Mécanisme |
| --- | --- |
| iOS / macOS | Keychain |
| Android | Chiffrement RSA-OAEP + AES-GCM, clé protégée par le Keystore |
| Windows | Credential Locker |
| Linux | libsecret (Gnome Keyring, KWallet) |
| Web | WebCrypto, sur HTTPS ou localhost uniquement |

```text
flutter pub add flutter_secure_storage
```

```yaml
dependencies:
  flutter_secure_storage: ^11.0.0
```

L'API est volontairement minimale, et **entièrement asynchrone** :

| Méthode | Signature |
| --- | --- |
| Écrire | `write({required String key, required String? value})` |
| Lire | `read({required String key}) → Future<String?>` |
| Tester | `containsKey({required String key}) → Future<bool>` |
| Supprimer | `delete({required String key})` |
| Tout lire | `readAll() → Future<Map<String, String>>` |
| Tout supprimer | `deleteAll()` |

```dart
import 'package:flutter/material.dart';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

// ─────────────────────────────────────────────────────────────
//  lib/services/session_service.dart
// ─────────────────────────────────────────────────────────────

class SessionService {
  SessionService([FlutterSecureStorage? storage])
      : _storage = storage ?? const FlutterSecureStorage();

  final FlutterSecureStorage _storage;

  static const String _cleJeton = 'jeton_acces';
  static const String _cleRafraichissement = 'jeton_rafraichissement';

  Future<void> ouvrirSession({
    required String jeton,
    required String rafraichissement,
  }) async {
    await _storage.write(key: _cleJeton, value: jeton);
    await _storage.write(key: _cleRafraichissement, value: rafraichissement);
  }

  Future<String?> jeton() => _storage.read(key: _cleJeton);

  Future<bool> estConnecte() async => _storage.containsKey(key: _cleJeton);

  Future<void> fermerSession() async {
    await _storage.delete(key: _cleJeton);
    await _storage.delete(key: _cleRafraichissement);
  }
}

// ─────────────────────────────────────────────────────────────
//  lib/main.dart
// ─────────────────────────────────────────────────────────────

void main() => runApp(const ApplicationSession());

class ApplicationSession extends StatelessWidget {
  const ApplicationSession({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Session sécurisée',
      theme: ThemeData(colorSchemeSeed: Colors.red, useMaterial3: true),
      home: const PageSession(),
    );
  }
}

class PageSession extends StatefulWidget {
  const PageSession({super.key});

  @override
  State<PageSession> createState() => _PageSessionState();
}

class _PageSessionState extends State<PageSession> {
  final SessionService _session = SessionService();
  String _etat = 'Inconnu';

  @override
  void initState() {
    super.initState();
    _rafraichir();
  }

  Future<void> _rafraichir() async {
    final String? jeton = await _session.jeton();
    if (!mounted) return;
    setState(() {
      _etat = jeton == null
          ? 'Aucune session.'
          : 'Connecté. Jeton : ${jeton.substring(0, 12)}...';
    });
  }

  Future<void> _connexion() async {
    await _session.ouvrirSession(
      jeton: 'eyJhbGciOiJIUzI1NiJ9.faux-jeton-de-demonstration',
      rafraichissement: 'r-0000-1111-2222',
    );
    await _rafraichir();
  }

  Future<void> _deconnexion() async {
    await _session.fermerSession();
    await _rafraichir();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Session sécurisée')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(_etat, textAlign: TextAlign.center),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: _connexion,
              child: const Text('Se connecter'),
            ),
            const SizedBox(height: 8),
            OutlinedButton(
              onPressed: _deconnexion,
              child: const Text('Se déconnecter'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :** le jeton survit au relancement de l'application, mais il n'apparaît nulle part en clair dans les fichiers de l'application.

Quatre points de vigilance.

**Point 1 — c'est lent.** Chaque lecture passe par un mécanisme de chiffrement système. N'y stockez que des secrets, et lisez-les une fois au démarrage, pas à chaque `build`.

**Point 2 — les valeurs sont des `String`.** Pour stocker autre chose, encodez en JSON.

**Point 3 — Android sans configuration.** Le paquet désactive la sauvegarde automatique Android par défaut, pour éviter qu'une restauration de sauvegarde ne ramène des données chiffrées avec une clé disparue (`InvalidKeyException`).

**Point 4 — ce n'est pas magique.** Sur un appareil rooté ou débridé, un attaquant déterminé peut toujours accéder à la mémoire du processus. `flutter_secure_storage` élève fortement la barre ; il ne la rend pas infranchissable. **Un secret qui ne doit jamais fuir n'a rien à faire sur un client.**

---

## 54.35 — Panorama : `hive`, `isar`, `drift`

`sqflite` n'est pas la seule base locale. Voici le paysage, honnêtement.

**`drift`** (version 2.34.3 au moment de la rédaction) est une couche typée au-dessus de SQLite. Vous décrivez vos tables en Dart, un générateur de code produit les requêtes, et le compilateur vérifie votre SQL. Les requêtes peuvent renvoyer des `Stream` qui se mettent à jour tout seuls quand la table change.

```dart
// Extrait illustratif de drift : le SQL est vérifié à la compilation
Stream<List<Potion>> potionsPuissantes() {
  return (select(potions)..where((t) => t.soin.isBiggerThanValue(50))).watch();
}
```

| Avantage | Inconvénient |
| --- | --- |
| SQL vérifié à la compilation | Génération de code (`build_runner`) |
| `Stream` réactifs natifs | Courbe d'apprentissage plus raide |
| Fonctionne sur le Web | Plus de fichiers dans le projet |
| Migrations outillées | |

**`hive`** est une base clé-valeur en Dart pur, très rapide, sans SQL. On y range des objets dans des « boîtes » (*boxes*).

```dart
final box = await Hive.openBox<Potion>('potions');
await box.put('p-001', potion);
final Potion? p = box.get('p-001');
```

Attention à l'état du projet : le paquet `hive` d'origine en est resté à la version 2.2.3, publiée en 2022, et n'est plus activement maintenu. La communauté a pris le relais avec **`hive_ce`** (*community edition*, version 2.19.3 au moment de la rédaction), qui est le choix raisonnable aujourd'hui si vous voulez Hive.

| Avantage | Inconvénient |
| --- | --- |
| Très rapide, Dart pur | Pas de requêtes SQL |
| Fonctionne partout, Web compris | Filtres et tris faits en mémoire |
| API très simple | Le paquet d'origine n'est plus maintenu |

**`isar`** est une base NoSQL orientée objets, avec des index et un langage de requêtes fluide en Dart. Sa version 3 est éprouvée ; la version 4 a connu un développement heurté et son avenir a été discuté publiquement. Vérifiez son état sur pub.dev avant de l'adopter pour un projet à long terme.

Le tableau de choix :

| Situation | Choix conseillé |
| --- | --- |
| Vous apprenez, ou le projet est simple | `sqflite` |
| Vous connaissez SQL et voulez de la sûreté de types | `drift` |
| Vous voulez du réactif sans écrire de SQL | `drift` |
| Vous stockez des objets sans les interroger finement | `hive_ce` |
| Vous ciblez le Web en priorité | `drift` ou `hive_ce` |
| Vous voulez le plus large écosystème de tutoriels | `sqflite` |

> Le conseil pédagogique est net : **apprenez `sqflite` d'abord.** SQL est une compétence transférable qui vous servira côté serveur, en analyse de données et dans tous les langages. Les autres paquets s'apprennent en une journée quand on comprend déjà les concepts.

---

## 54.36 — Le cache hors-ligne : stratégie « réseau d'abord, cache ensuite » (rappel chapitre 53)

Au chapitre 53, votre `FutureBuilder` affichait une erreur dès que le réseau tombait. Une application sérieuse fait mieux : elle affiche les dernières données connues.

Il existe quatre stratégies de cache. Il faut savoir les nommer.

| Stratégie | Comportement | Convient à |
| --- | --- | --- |
| **Réseau seul** | Aucune persistance | Données de sécurité, temps réel strict |
| **Cache d'abord** | Cache si présent, réseau sinon | Contenu quasi statique (conditions d'utilisation) |
| **Réseau d'abord** | Réseau, et repli sur le cache en cas d'échec | Cas général |
| **Périmé pendant revalidation** | Cache immédiatement, puis réseau en arrière-plan | Listes, flux, catalogues |

La plus utile au quotidien est « réseau d'abord ».

```text
   Demande de données
          │
          v
   ┌──────────────────┐
   │ Appel réseau     │
   └──────────────────┘
          │
    ┌─────┴─────┐
    │           │
  succès      échec
    │           │
    v           v
 écrire      lire le
 le cache     cache
    │           │
    v      ┌────┴────┐
 renvoyer  vide   présent
 (frais)    │        │
            v        v
        propager  renvoyer
        l'erreur  (périmé)
```

Voici l'implémentation complète, avec `shared_preferences` comme support de cache.

```dart
import 'dart:convert';

import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

// ─────────────────────────────────────────────────────────────
//  lib/models/monstre.dart
// ─────────────────────────────────────────────────────────────

class Monstre {
  const Monstre({required this.nom, required this.pv});

  final String nom;
  final int pv;

  Map<String, dynamic> toJson() => <String, dynamic>{'nom': nom, 'pv': pv};

  factory Monstre.fromJson(Map<String, dynamic> j) => Monstre(
        nom: j['nom'] as String? ?? '?',
        pv: j['pv'] as int? ?? 0,
      );
}

// ─────────────────────────────────────────────────────────────
//  lib/data/monstre_repository.dart
// ─────────────────────────────────────────────────────────────

class ResultatDonnees {
  const ResultatDonnees({
    required this.monstres,
    required this.provenance,
    this.dateCache,
  });

  final List<Monstre> monstres;
  final String provenance; // 'réseau' ou 'cache'
  final DateTime? dateCache;

  bool get estPerime => provenance == 'cache';
}

class MonstreRepository {
  MonstreRepository(this._prefs, {this.simulerPanne = false});

  static const String _cleDonnees = 'cache_monstres';
  static const String _cleDate = 'cache_monstres_date';

  final SharedPreferences _prefs;
  final bool simulerPanne;

  /// Simule un appel HTTP (chapitre 53).
  Future<List<Monstre>> _appelReseau() async {
    await Future<void>.delayed(const Duration(milliseconds: 900));
    if (simulerPanne) {
      throw Exception('Pas de connexion réseau');
    }
    return const <Monstre>[
      Monstre(nom: 'Gobelin', pv: 30),
      Monstre(nom: 'Squelette', pv: 45),
      Monstre(nom: 'Dragon', pv: 400),
    ];
  }

  Future<void> _ecrireCache(List<Monstre> monstres) async {
    await _prefs.setString(
      _cleDonnees,
      jsonEncode(monstres.map((Monstre m) => m.toJson()).toList()),
    );
    await _prefs.setInt(_cleDate, DateTime.now().millisecondsSinceEpoch);
  }

  (List<Monstre>, DateTime)? _lireCache() {
    final String? brut = _prefs.getString(_cleDonnees);
    if (brut == null) return null;
    try {
      final List<dynamic> liste = jsonDecode(brut) as List<dynamic>;
      final List<Monstre> monstres = liste
          .whereType<Map<String, dynamic>>()
          .map(Monstre.fromJson)
          .toList();
      final DateTime date = DateTime.fromMillisecondsSinceEpoch(
        _prefs.getInt(_cleDate) ?? 0,
      );
      return (monstres, date);
    } on FormatException {
      _prefs.remove(_cleDonnees);
      return null;
    }
  }

  /// Réseau d'abord, cache ensuite.
  Future<ResultatDonnees> charger() async {
    try {
      final List<Monstre> frais = await _appelReseau();
      await _ecrireCache(frais);
      return ResultatDonnees(monstres: frais, provenance: 'réseau');
    } on Object {
      final (List<Monstre>, DateTime)? cache = _lireCache();
      if (cache == null) {
        rethrow; // Rien à afficher : l'erreur est légitime.
      }
      return ResultatDonnees(
        monstres: cache.$1,
        provenance: 'cache',
        dateCache: cache.$2,
      );
    }
  }
}
```

Le point clé est le `rethrow` : on ne masque l'erreur que si l'on a **quelque chose à montrer à la place**. Si le cache est vide, l'utilisateur doit voir l'erreur réelle, pas une liste vide inexplicable.

Le `(List<Monstre>, DateTime)?` est un **enregistrement** (*record*) de Dart 3 : une manière légère de renvoyer deux valeurs sans créer de classe. On accède aux champs par `.$1` et `.$2`.

---

## 54.37 — Afficher des données périmées plutôt qu'un écran vide

Le dépôt de la section précédente sait d'où viennent les données. L'interface doit le dire honnêtement à l'utilisateur.

Voici l'écran complet.

```dart
import 'dart:convert';

import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final SharedPreferences prefs = await SharedPreferences.getInstance();
  runApp(ApplicationBestiaire(prefs: prefs));
}

class Monstre {
  const Monstre({required this.nom, required this.pv});

  final String nom;
  final int pv;

  Map<String, dynamic> toJson() => <String, dynamic>{'nom': nom, 'pv': pv};

  factory Monstre.fromJson(Map<String, dynamic> j) => Monstre(
        nom: j['nom'] as String? ?? '?',
        pv: j['pv'] as int? ?? 0,
      );
}

class ResultatDonnees {
  const ResultatDonnees({
    required this.monstres,
    required this.provenance,
    this.dateCache,
  });

  final List<Monstre> monstres;
  final String provenance;
  final DateTime? dateCache;

  bool get estPerime => provenance == 'cache';
}

class MonstreRepository {
  MonstreRepository(this._prefs);

  static const String _cleDonnees = 'cache_monstres';
  static const String _cleDate = 'cache_monstres_date';

  final SharedPreferences _prefs;
  bool panne = false;

  Future<List<Monstre>> _appelReseau() async {
    await Future<void>.delayed(const Duration(milliseconds: 900));
    if (panne) throw Exception('Pas de connexion réseau');
    return const <Monstre>[
      Monstre(nom: 'Gobelin', pv: 30),
      Monstre(nom: 'Squelette', pv: 45),
      Monstre(nom: 'Dragon', pv: 400),
    ];
  }

  Future<void> _ecrireCache(List<Monstre> monstres) async {
    await _prefs.setString(
      _cleDonnees,
      jsonEncode(monstres.map((Monstre m) => m.toJson()).toList()),
    );
    await _prefs.setInt(_cleDate, DateTime.now().millisecondsSinceEpoch);
  }

  (List<Monstre>, DateTime)? _lireCache() {
    final String? brut = _prefs.getString(_cleDonnees);
    if (brut == null) return null;
    try {
      final List<dynamic> liste = jsonDecode(brut) as List<dynamic>;
      return (
        liste
            .whereType<Map<String, dynamic>>()
            .map(Monstre.fromJson)
            .toList(),
        DateTime.fromMillisecondsSinceEpoch(_prefs.getInt(_cleDate) ?? 0),
      );
    } on FormatException {
      _prefs.remove(_cleDonnees);
      return null;
    }
  }

  Future<ResultatDonnees> charger() async {
    try {
      final List<Monstre> frais = await _appelReseau();
      await _ecrireCache(frais);
      return ResultatDonnees(monstres: frais, provenance: 'réseau');
    } on Object {
      final (List<Monstre>, DateTime)? cache = _lireCache();
      if (cache == null) rethrow;
      return ResultatDonnees(
        monstres: cache.$1,
        provenance: 'cache',
        dateCache: cache.$2,
      );
    }
  }
}

class ApplicationBestiaire extends StatelessWidget {
  const ApplicationBestiaire({super.key, required this.prefs});

  final SharedPreferences prefs;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Bestiaire hors-ligne',
      theme: ThemeData(colorSchemeSeed: Colors.green, useMaterial3: true),
      home: PageBestiaire(depot: MonstreRepository(prefs)),
    );
  }
}

class PageBestiaire extends StatefulWidget {
  const PageBestiaire({super.key, required this.depot});

  final MonstreRepository depot;

  @override
  State<PageBestiaire> createState() => _PageBestiaireState();
}

class _PageBestiaireState extends State<PageBestiaire> {
  late Future<ResultatDonnees> _futur = widget.depot.charger();

  Future<void> _recharger() async {
    setState(() {
      _futur = widget.depot.charger();
    });
    await _futur.catchError((Object _) => const ResultatDonnees(
          monstres: <Monstre>[],
          provenance: 'cache',
        ));
  }

  String _age(DateTime? date) {
    if (date == null) return '';
    final Duration d = DateTime.now().difference(date);
    if (d.inMinutes < 1) return 'il y a quelques secondes';
    if (d.inHours < 1) return 'il y a ${d.inMinutes} min';
    if (d.inDays < 1) return 'il y a ${d.inHours} h';
    return 'il y a ${d.inDays} j';
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Bestiaire'),
        actions: <Widget>[
          IconButton(
            icon: Icon(
              widget.depot.panne ? Icons.wifi_off : Icons.wifi,
            ),
            tooltip: 'Simuler une panne réseau',
            onPressed: () {
              setState(() => widget.depot.panne = !widget.depot.panne);
              _recharger();
            },
          ),
        ],
      ),
      body: RefreshIndicator(
        onRefresh: _recharger,
        child: FutureBuilder<ResultatDonnees>(
          future: _futur,
          builder: (BuildContext context,
              AsyncSnapshot<ResultatDonnees> snapshot) {
            if (snapshot.connectionState == ConnectionState.waiting) {
              return const Center(child: CircularProgressIndicator());
            }

            if (snapshot.hasError) {
              return ListView(
                children: <Widget>[
                  const SizedBox(height: 120),
                  const Icon(Icons.cloud_off, size: 64),
                  const SizedBox(height: 16),
                  Center(
                    child: Text(
                      'Impossible de charger le bestiaire.\n'
                      'Aucune donnée en cache.',
                      textAlign: TextAlign.center,
                    ),
                  ),
                  const SizedBox(height: 16),
                  Center(
                    child: FilledButton(
                      onPressed: _recharger,
                      child: const Text('Réessayer'),
                    ),
                  ),
                ],
              );
            }

            final ResultatDonnees resultat = snapshot.data!;
            return Column(
              children: <Widget>[
                if (resultat.estPerime)
                  Material(
                    color: Theme.of(context).colorScheme.tertiaryContainer,
                    child: Padding(
                      padding: const EdgeInsets.all(12),
                      child: Row(
                        children: <Widget>[
                          const Icon(Icons.cloud_off, size: 20),
                          const SizedBox(width: 12),
                          Expanded(
                            child: Text(
                              'Mode hors-ligne — données mises à jour '
                              '${_age(resultat.dateCache)}.',
                            ),
                          ),
                        ],
                      ),
                    ),
                  ),
                Expanded(
                  child: ListView.builder(
                    itemCount: resultat.monstres.length,
                    itemBuilder: (BuildContext context, int index) {
                      final Monstre m = resultat.monstres[index];
                      return ListTile(
                        leading: const Icon(Icons.pest_control),
                        title: Text(m.nom),
                        trailing: Text('${m.pv} PV'),
                      );
                    },
                  ),
                ),
              ],
            );
          },
        ),
      ),
    );
  }
}
```

**Résultat :** en ligne, la liste s'affiche normalement. Vous appuyez sur l'icône Wi-Fi pour simuler une panne, puis vous tirez pour rafraîchir : la liste reste affichée, surmontée d'un bandeau « Mode hors-ligne — données mises à jour il y a 2 min ».

La règle de conception à retenir tient en une phrase :

> **Une donnée périmée et signalée vaut mieux qu'un écran vide.**

Trois conditions pour que cela soit honnête :

1. l'utilisateur voit clairement que les données ne sont pas fraîches ;
2. il connaît l'âge des données ;
3. il dispose d'un moyen évident de réessayer.

Le contre-exemple absolu, à ne jamais faire : afficher silencieusement des données de la semaine dernière comme si elles étaient à jour.

---

## 54.38 — Tester une couche de persistance

Une couche de persistance se teste **sans émulateur**, avec `flutter test`. C'est possible parce que les paquets fournissent des implémentations en mémoire.

Ajoutez les dépendances de test :

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  sqflite_common_ffi: ^2.3.7
```

**Tester `shared_preferences`.** Le paquet fournit `setMockInitialValues`, qui remplace la plateforme native par une simple `Map`.

```dart
// test/preferences_service_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:shared_preferences/shared_preferences.dart';

import 'package:mon_jeu/services/preferences_service.dart';

void main() {
  TestWidgetsFlutterBinding.ensureInitialized();

  group('PreferencesService', () {
    test('renvoie les valeurs par défaut quand rien n\'est stocké', () async {
      SharedPreferences.setMockInitialValues(<String, Object>{});
      final PreferencesService service = await PreferencesService.creer();

      expect(service.pseudo, PreferencesService.pseudoParDefaut);
      expect(service.volume, PreferencesService.volumeParDefaut);
    });

    test('relit ce qui a été écrit', () async {
      SharedPreferences.setMockInitialValues(<String, Object>{});
      final PreferencesService service = await PreferencesService.creer();

      await service.setPseudo('  Aria  ');
      expect(service.pseudo, 'Aria'); // le trim a bien eu lieu
    });

    test('borne le volume entre 0 et 1', () async {
      SharedPreferences.setMockInitialValues(<String, Object>{});
      final PreferencesService service = await PreferencesService.creer();

      await service.setVolume(3.7);
      expect(service.volume, 1.0);

      await service.setVolume(-2);
      expect(service.volume, 0.0);
    });

    test('lit une valeur préexistante', () async {
      SharedPreferences.setMockInitialValues(<String, Object>{
        'pseudo': 'Kael',
      });
      final PreferencesService service = await PreferencesService.creer();

      expect(service.pseudo, 'Kael');
    });
  });
}
```

**Résultat :**

```text
00:01 +4: All tests passed!
```

Notez `setMockInitialValues(<String, Object>{})` **au début de chaque test** : sinon les tests se contaminent entre eux.

**Tester `sqflite`.** Le paquet `sqflite_common_ffi` fournit une implémentation qui tourne sur votre machine de développement, y compris **en mémoire** grâce au chemin spécial `inMemoryDatabasePath`.

```dart
// test/potion_dao_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:sqflite_common_ffi/sqflite_ffi.dart';

import 'package:mon_jeu/data/potion_dao.dart';
import 'package:mon_jeu/models/potion.dart';

void main() {
  setUpAll(() {
    sqfliteFfiInit();
    databaseFactory = databaseFactoryFfi;
  });

  late Database db;
  late PotionDao dao;

  setUp(() async {
    db = await databaseFactory.openDatabase(
      inMemoryDatabasePath,
      options: OpenDatabaseOptions(
        version: 1,
        onCreate: (Database db, int version) async {
          await db.execute('''
            CREATE TABLE potion (
              id       TEXT    PRIMARY KEY,
              nom      TEXT    NOT NULL,
              soin     INTEGER NOT NULL,
              prix     REAL    NOT NULL,
              favorite INTEGER NOT NULL DEFAULT 0,
              cree_le  INTEGER NOT NULL,
              rarete   TEXT    NOT NULL DEFAULT 'commune'
            )
          ''');
        },
      ),
    );
    dao = PotionDao(db);
  });

  tearDown(() async {
    await db.close();
  });

  Potion faire(String id, {int soin = 10, String nom = 'Potion'}) => Potion(
        id: id,
        nom: nom,
        soin: soin,
        prix: 5,
        favorite: false,
        creeLe: DateTime(2026),
      );

  test('la base est vide au départ', () async {
    expect(await dao.compter(), 0);
  });

  test('enregistrer puis relire', () async {
    await dao.enregistrer(faire('p-1', nom: 'Élixir'));
    final Potion? lu = await dao.parId('p-1');

    expect(lu, isNotNull);
    expect(lu!.nom, 'Élixir');
  });

  test('enregistrer deux fois le même id met à jour', () async {
    await dao.enregistrer(faire('p-1', soin: 10));
    await dao.enregistrer(faire('p-1', soin: 99));

    expect(await dao.compter(), 1);
    expect((await dao.parId('p-1'))!.soin, 99);
  });

  test('recherche partielle', () async {
    await dao.enregistrer(faire('p-1', nom: 'Potion de soin'));
    await dao.enregistrer(faire('p-2', nom: 'Élixir de force'));

    final List<Potion> trouvees = await dao.rechercher('soin');
    expect(trouvees.length, 1);
    expect(trouvees.single.id, 'p-1');
  });

  test('supprimer renvoie le nombre de lignes', () async {
    await dao.enregistrer(faire('p-1'));

    expect(await dao.supprimer('p-1'), 1);
    expect(await dao.supprimer('inexistant'), 0);
  });

  test('basculer favorite', () async {
    await dao.enregistrer(faire('p-1'));

    expect(await dao.basculerFavorite('p-1'), true);
    expect(await dao.basculerFavorite('p-1'), false);
    expect(await dao.basculerFavorite('inconnu'), false);
  });
}
```

**Résultat :**

```text
00:02 +6: All tests passed!
```

Trois principes rendent ces tests possibles, et vous devez les appliquer dès la conception :

| Principe | Traduction en code |
| --- | --- |
| Injecter la dépendance | `PotionDao(this._db)` plutôt qu'un singleton interne |
| Isoler chaque test | `setUp` crée une base neuve, `tearDown` la ferme |
| Utiliser une base en mémoire | `inMemoryDatabasePath` : rapide, jamais de résidus |

> Une couche de persistance non testée est une bombe à retardement : ses bugs n'apparaissent qu'après une mise à jour, chez les utilisateurs, sur des données que vous ne pouvez pas reproduire.

---

## 54.39 — Bilan de la PARTIE 1B : ce que vous savez faire maintenant

Ce chapitre clôt la PARTIE 1B. Faisons le compte, chapitre par chapitre.

| Ch. | Titre | Ce que vous savez faire |
| --- | --- | --- |
| 43 | Installer Flutter | Installer le SDK, lire `flutter doctor`, créer un projet, comprendre l'arborescence, utiliser le hot reload |
| 44 | Widgets et arbre | Composer une interface, lire un `BuildContext`, poser un `MaterialApp` et un `Scaffold`, utiliser `const` à bon escient |
| 45 | Stateless / Stateful | Choisir entre les deux, appeler `setState`, utiliser `initState` et `dispose`, remonter l'état |
| 46 | Layouts | Placer n'importe quoi n'importe où, lire les contraintes, corriger un `RenderFlex overflowed` |
| 47 | Texte, images, assets | Styler du texte, déclarer des assets, charger une image réseau, poser une icône |
| 48 | Listes | `ListView.builder`, `.separated`, `GridView`, `Card`, `ListTile`, `Dismissible` |
| 49 | Formulaires | `TextField`, `TextEditingController`, `Form`, validation, gestion du clavier |
| 50 | Navigation | `push` / `pop`, routes nommées, passage et retour de données, onglets, `Drawer` |
| 51 | Thèmes | `ThemeData`, Material 3, mode sombre, `MediaQuery`, `LayoutBuilder`, responsive |
| 52 | Gestion d'état | `ChangeNotifier`, `ListenableBuilder`, `provider`, choix d'une architecture |
| 53 | API REST | `http`, JSON, modèles, `FutureBuilder`, chargement, erreurs, `StreamBuilder` |
| 54 | Persistance | Préférences, fichiers, SQLite, stockage sécurisé, cache hors-ligne |

Autrement dit, vous savez construire une application Flutter **complète** :

```text
   ┌────────────────────────────────────────────────────────────┐
   │  INTERFACE          widgets, layouts, thèmes, navigation   │  ch. 44-51
   ├────────────────────────────────────────────────────────────┤
   │  ÉTAT               ChangeNotifier, provider               │  ch. 52
   ├────────────────────────────────────────────────────────────┤
   │  DONNÉES DISTANTES  http, JSON, FutureBuilder              │  ch. 53
   ├────────────────────────────────────────────────────────────┤
   │  DONNÉES LOCALES    préférences, fichiers, SQLite, cache   │  ch. 54
   └────────────────────────────────────────────────────────────┘
```

Les quatre couches sont là. Il ne manque plus que la pratique : c'est exactement l'objet de la PARTIE 1C.

Une petite liste de contrôle avant de passer à la suite. Si vous répondez « non » à l'une de ces questions, relisez le chapitre indiqué.

| Question | Chapitre |
| --- | --- |
| Sauriez-vous expliquer la différence entre `StatelessWidget` et `StatefulWidget` sans regarder ? | 45 |
| Sauriez-vous corriger un `RenderFlex overflowed` de trois façons ? | 46 |
| Sauriez-vous valider un formulaire et afficher un message d'erreur par champ ? | 49 |
| Sauriez-vous passer une donnée à un écran, et en récupérer une au retour ? | 50 |
| Sauriez-vous rendre le thème sombre persistant ? | 51 + 54 |
| Sauriez-vous partager un compteur entre trois écrans sans `setState` en cascade ? | 52 |
| Sauriez-vous afficher une liste venue d'une API, avec chargement et erreur ? | 53 |
| Sauriez-vous choisir entre `shared_preferences`, un fichier et `sqflite` ? | 54 |

---

## 54.40 — Mini-projet : une liste de tâches persistante, en `shared_preferences` puis en `sqflite`

Le meilleur moyen de comprendre quand changer de solution est d'écrire **la même application deux fois**.

### Cahier des charges

| Exigence | Détail |
| --- | --- |
| Ajouter une tâche | Titre obligatoire, non vide |
| Cocher / décocher | Un appui sur la ligne bascule l'état |
| Supprimer | Balayage latéral (`Dismissible`, chapitre 48) |
| Filtrer | Toutes / En cours / Terminées |
| Persister | Les tâches survivent à la fermeture |
| Compter | Le titre affiche « n restantes » |

### Le modèle, commun aux deux versions

```dart
class Tache {
  const Tache({
    required this.id,
    required this.titre,
    required this.faite,
    required this.creeLe,
  });

  final String id;
  final String titre;
  final bool faite;
  final DateTime creeLe;

  Tache copyWith({String? titre, bool? faite}) => Tache(
        id: id,
        titre: titre ?? this.titre,
        faite: faite ?? this.faite,
        creeLe: creeLe,
      );

  Map<String, dynamic> toJson() => <String, dynamic>{
        'id': id,
        'titre': titre,
        'faite': faite,
        'creeLe': creeLe.millisecondsSinceEpoch,
      };

  factory Tache.fromJson(Map<String, dynamic> j) => Tache(
        id: j['id'] as String,
        titre: j['titre'] as String? ?? '',
        faite: j['faite'] as bool? ?? false,
        creeLe: DateTime.fromMillisecondsSinceEpoch(j['creeLe'] as int? ?? 0),
      );

  Map<String, Object?> toMap() => <String, Object?>{
        'id': id,
        'titre': titre,
        'faite': faite ? 1 : 0,
        'cree_le': creeLe.millisecondsSinceEpoch,
      };

  factory Tache.fromMap(Map<String, Object?> m) => Tache(
        id: m['id']! as String,
        titre: m['titre']! as String,
        faite: (m['faite']! as int) == 1,
        creeLe: DateTime.fromMillisecondsSinceEpoch(m['cree_le']! as int),
      );
}
```

Notez que le modèle porte **quatre** méthodes de conversion : deux pour le JSON, deux pour SQL. C'est volontaire, pour la démonstration. Dans un vrai projet, on ne garde que celles dont on a besoin.

### L'interface d'accès, commune aux deux versions

C'est la clé du mini-projet : les deux implémentations respectent **le même contrat**, donc l'interface graphique ne change pas d'une ligne.

```dart
abstract class TacheStore {
  Future<List<Tache>> toutes();
  Future<void> ajouter(Tache tache);
  Future<void> mettreAJour(Tache tache); // remplace la tâche de même id
  Future<void> supprimer(String id);
}
```

### Version 1 — `shared_preferences`

```dart
import 'dart:convert';

import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

// ── Modèle (voir ci-dessus, repris ici pour un fichier autonome) ──
class Tache {
  const Tache({
    required this.id,
    required this.titre,
    required this.faite,
    required this.creeLe,
  });

  final String id;
  final String titre;
  final bool faite;
  final DateTime creeLe;

  Tache copyWith({String? titre, bool? faite}) => Tache(
        id: id,
        titre: titre ?? this.titre,
        faite: faite ?? this.faite,
        creeLe: creeLe,
      );

  Map<String, dynamic> toJson() => <String, dynamic>{
        'id': id,
        'titre': titre,
        'faite': faite,
        'creeLe': creeLe.millisecondsSinceEpoch,
      };

  factory Tache.fromJson(Map<String, dynamic> j) => Tache(
        id: j['id'] as String,
        titre: j['titre'] as String? ?? '',
        faite: j['faite'] as bool? ?? false,
        creeLe: DateTime.fromMillisecondsSinceEpoch(j['creeLe'] as int? ?? 0),
      );
}

// ── Store en préférences ──
class TachePrefsStore {
  TachePrefsStore(this._prefs);

  static const String _cle = 'taches';

  final SharedPreferences _prefs;

  List<Tache> _cache = <Tache>[];

  Future<List<Tache>> toutes() async {
    final String? brut = _prefs.getString(_cle);
    if (brut == null) {
      _cache = <Tache>[];
      return _cache;
    }
    try {
      final List<dynamic> liste = jsonDecode(brut) as List<dynamic>;
      _cache = liste
          .whereType<Map<String, dynamic>>()
          .map(Tache.fromJson)
          .toList();
    } on FormatException {
      await _prefs.remove(_cle);
      _cache = <Tache>[];
    }
    return _cache;
  }

  Future<void> _sauver() async {
    await _prefs.setString(
      _cle,
      jsonEncode(_cache.map((Tache t) => t.toJson()).toList()),
    );
  }

  Future<void> ajouter(Tache tache) async {
    _cache = <Tache>[tache, ..._cache];
    await _sauver();
  }

  Future<void> mettreAJour(Tache tache) async {
    _cache = _cache.map((Tache t) => t.id == tache.id ? tache : t).toList();
    await _sauver();
  }

  Future<void> supprimer(String id) async {
    _cache = _cache.where((Tache t) => t.id != id).toList();
    await _sauver();
  }
}

// ── Application ──
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final SharedPreferences prefs = await SharedPreferences.getInstance();
  runApp(ApplicationTaches(store: TachePrefsStore(prefs)));
}

enum Filtre { toutes, enCours, terminees }

class ApplicationTaches extends StatelessWidget {
  const ApplicationTaches({super.key, required this.store});

  final TachePrefsStore store;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Tâches',
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: PageTaches(store: store),
    );
  }
}

class PageTaches extends StatefulWidget {
  const PageTaches({super.key, required this.store});

  final TachePrefsStore store;

  @override
  State<PageTaches> createState() => _PageTachesState();
}

class _PageTachesState extends State<PageTaches> {
  final TextEditingController _saisie = TextEditingController();
  List<Tache> _taches = <Tache>[];
  Filtre _filtre = Filtre.toutes;
  bool _charge = false;

  @override
  void initState() {
    super.initState();
    _charger();
  }

  @override
  void dispose() {
    _saisie.dispose();
    super.dispose();
  }

  Future<void> _charger() async {
    final List<Tache> taches = await widget.store.toutes();
    if (!mounted) return;
    setState(() {
      _taches = taches;
      _charge = true;
    });
  }

  Future<void> _ajouter() async {
    final String titre = _saisie.text.trim();
    if (titre.isEmpty) return;
    final Tache tache = Tache(
      id: DateTime.now().microsecondsSinceEpoch.toString(),
      titre: titre,
      faite: false,
      creeLe: DateTime.now(),
    );
    await widget.store.ajouter(tache);
    _saisie.clear();
    await _charger();
  }

  Future<void> _basculer(Tache tache) async {
    await widget.store.mettreAJour(tache.copyWith(faite: !tache.faite));
    await _charger();
  }

  Future<void> _supprimer(Tache tache) async {
    await widget.store.supprimer(tache.id);
    await _charger();
  }

  List<Tache> get _visibles => switch (_filtre) {
        Filtre.toutes => _taches,
        Filtre.enCours => _taches.where((Tache t) => !t.faite).toList(),
        Filtre.terminees => _taches.where((Tache t) => t.faite).toList(),
      };

  @override
  Widget build(BuildContext context) {
    final int restantes = _taches.where((Tache t) => !t.faite).length;

    return Scaffold(
      appBar: AppBar(title: Text('Tâches — $restantes restantes')),
      body: !_charge
          ? const Center(child: CircularProgressIndicator())
          : Column(
              children: <Widget>[
                Padding(
                  padding: const EdgeInsets.all(16),
                  child: Row(
                    children: <Widget>[
                      Expanded(
                        child: TextField(
                          controller: _saisie,
                          decoration: const InputDecoration(
                            labelText: 'Nouvelle tâche',
                            border: OutlineInputBorder(),
                          ),
                          onSubmitted: (_) => _ajouter(),
                        ),
                      ),
                      const SizedBox(width: 12),
                      FilledButton(
                        onPressed: _ajouter,
                        child: const Text('Ajouter'),
                      ),
                    ],
                  ),
                ),
                SegmentedButton<Filtre>(
                  segments: const <ButtonSegment<Filtre>>[
                    ButtonSegment<Filtre>(
                      value: Filtre.toutes,
                      label: Text('Toutes'),
                    ),
                    ButtonSegment<Filtre>(
                      value: Filtre.enCours,
                      label: Text('En cours'),
                    ),
                    ButtonSegment<Filtre>(
                      value: Filtre.terminees,
                      label: Text('Terminées'),
                    ),
                  ],
                  selected: <Filtre>{_filtre},
                  onSelectionChanged: (Set<Filtre> s) =>
                      setState(() => _filtre = s.first),
                ),
                const SizedBox(height: 8),
                Expanded(
                  child: _visibles.isEmpty
                      ? const Center(child: Text('Rien à afficher.'))
                      : ListView.builder(
                          itemCount: _visibles.length,
                          itemBuilder: (BuildContext context, int index) {
                            final Tache t = _visibles[index];
                            return Dismissible(
                              key: ValueKey<String>(t.id),
                              direction: DismissDirection.endToStart,
                              background: Container(
                                alignment: Alignment.centerRight,
                                padding: const EdgeInsets.only(right: 24),
                                color: Theme.of(context).colorScheme.error,
                                child: const Icon(Icons.delete,
                                    color: Colors.white),
                              ),
                              onDismissed: (_) => _supprimer(t),
                              child: CheckboxListTile(
                                value: t.faite,
                                onChanged: (_) => _basculer(t),
                                title: Text(
                                  t.titre,
                                  style: TextStyle(
                                    decoration: t.faite
                                        ? TextDecoration.lineThrough
                                        : null,
                                  ),
                                ),
                              ),
                            );
                          },
                        ),
                ),
              ],
            ),
    );
  }
}
```

**Résultat :** une liste de tâches complète et persistante, en environ 200 lignes. Vous ajoutez « Acheter des potions », vous cochez, vous relancez : tout est là.

### Ce qui casse quand la liste grandit

Ajoutez mentalement 3 000 tâches à cette application. Trois choses se dégradent :

| Opération | Coût réel |
| --- | --- |
| Démarrage | Décoder 3 000 objets JSON avant le premier affichage |
| Cocher une case | Ré-encoder et réécrire **les 3 000** tâches |
| Filtrer « en cours » | Parcourir 3 000 objets en mémoire à chaque `build` |

C'est exactement le moment de passer à SQLite.

### Version 2 — `sqflite`

L'interface graphique est identique. Seul le store change.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

// ── Modèle ──
class Tache {
  const Tache({
    required this.id,
    required this.titre,
    required this.faite,
    required this.creeLe,
  });

  final String id;
  final String titre;
  final bool faite;
  final DateTime creeLe;

  Tache copyWith({String? titre, bool? faite}) => Tache(
        id: id,
        titre: titre ?? this.titre,
        faite: faite ?? this.faite,
        creeLe: creeLe,
      );

  Map<String, Object?> toMap() => <String, Object?>{
        'id': id,
        'titre': titre,
        'faite': faite ? 1 : 0,
        'cree_le': creeLe.millisecondsSinceEpoch,
      };

  factory Tache.fromMap(Map<String, Object?> m) => Tache(
        id: m['id']! as String,
        titre: m['titre']! as String,
        faite: (m['faite']! as int) == 1,
        creeLe: DateTime.fromMillisecondsSinceEpoch(m['cree_le']! as int),
      );
}

// ── Store SQLite ──
class TacheSqlStore {
  TacheSqlStore(this._db);

  final Database _db;

  static const String _table = 'tache';

  static Future<TacheSqlStore> ouvrir() async {
    final String chemin = p.join(await getDatabasesPath(), 'taches.db');
    final Database db = await openDatabase(
      chemin,
      version: 1,
      onCreate: (Database db, int version) async {
        await db.execute('''
          CREATE TABLE tache (
            id      TEXT    PRIMARY KEY,
            titre   TEXT    NOT NULL,
            faite   INTEGER NOT NULL DEFAULT 0,
            cree_le INTEGER NOT NULL
          )
        ''');
        await db.execute('CREATE INDEX idx_tache_faite ON tache (faite)');
      },
    );
    return TacheSqlStore(db);
  }

  Future<List<Tache>> toutes({bool? faite}) async {
    final List<Map<String, Object?>> lignes = await _db.query(
      _table,
      where: faite == null ? null : 'faite = ?',
      whereArgs: faite == null ? null : <Object?>[faite ? 1 : 0],
      orderBy: 'cree_le DESC',
    );
    return lignes.map(Tache.fromMap).toList();
  }

  Future<int> compterRestantes() async {
    return Sqflite.firstIntValue(
          await _db.rawQuery('SELECT COUNT(*) FROM $_table WHERE faite = 0'),
        ) ??
        0;
  }

  Future<void> ajouter(Tache tache) async {
    await _db.insert(
      _table,
      tache.toMap(),
      conflictAlgorithm: ConflictAlgorithm.replace,
    );
  }

  Future<void> basculer(String id, bool faite) async {
    await _db.update(
      _table,
      <String, Object?>{'faite': faite ? 1 : 0},
      where: 'id = ?',
      whereArgs: <Object?>[id],
    );
  }

  Future<void> supprimer(String id) async {
    await _db.delete(_table, where: 'id = ?', whereArgs: <Object?>[id]);
  }

  Future<void> fermer() => _db.close();
}

// ── Application ──
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final TacheSqlStore store = await TacheSqlStore.ouvrir();
  runApp(ApplicationTachesSql(store: store));
}

enum Filtre { toutes, enCours, terminees }

class ApplicationTachesSql extends StatelessWidget {
  const ApplicationTachesSql({super.key, required this.store});

  final TacheSqlStore store;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Tâches SQL',
      theme: ThemeData(colorSchemeSeed: Colors.teal, useMaterial3: true),
      home: PageTachesSql(store: store),
    );
  }
}

class PageTachesSql extends StatefulWidget {
  const PageTachesSql({super.key, required this.store});

  final TacheSqlStore store;

  @override
  State<PageTachesSql> createState() => _PageTachesSqlState();
}

class _PageTachesSqlState extends State<PageTachesSql> {
  final TextEditingController _saisie = TextEditingController();
  List<Tache> _visibles = <Tache>[];
  int _restantes = 0;
  Filtre _filtre = Filtre.toutes;
  bool _charge = false;

  @override
  void initState() {
    super.initState();
    _charger();
  }

  @override
  void dispose() {
    _saisie.dispose();
    super.dispose();
  }

  Future<void> _charger() async {
    final bool? filtreSql = switch (_filtre) {
      Filtre.toutes => null,
      Filtre.enCours => false,
      Filtre.terminees => true,
    };
    final List<Tache> taches = await widget.store.toutes(faite: filtreSql);
    final int restantes = await widget.store.compterRestantes();
    if (!mounted) return;
    setState(() {
      _visibles = taches;
      _restantes = restantes;
      _charge = true;
    });
  }

  Future<void> _ajouter() async {
    final String titre = _saisie.text.trim();
    if (titre.isEmpty) return;
    await widget.store.ajouter(
      Tache(
        id: DateTime.now().microsecondsSinceEpoch.toString(),
        titre: titre,
        faite: false,
        creeLe: DateTime.now(),
      ),
    );
    _saisie.clear();
    await _charger();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Tâches SQL — $_restantes restantes')),
      body: !_charge
          ? const Center(child: CircularProgressIndicator())
          : Column(
              children: <Widget>[
                Padding(
                  padding: const EdgeInsets.all(16),
                  child: Row(
                    children: <Widget>[
                      Expanded(
                        child: TextField(
                          controller: _saisie,
                          decoration: const InputDecoration(
                            labelText: 'Nouvelle tâche',
                            border: OutlineInputBorder(),
                          ),
                          onSubmitted: (_) => _ajouter(),
                        ),
                      ),
                      const SizedBox(width: 12),
                      FilledButton(
                        onPressed: _ajouter,
                        child: const Text('Ajouter'),
                      ),
                    ],
                  ),
                ),
                SegmentedButton<Filtre>(
                  segments: const <ButtonSegment<Filtre>>[
                    ButtonSegment<Filtre>(
                      value: Filtre.toutes,
                      label: Text('Toutes'),
                    ),
                    ButtonSegment<Filtre>(
                      value: Filtre.enCours,
                      label: Text('En cours'),
                    ),
                    ButtonSegment<Filtre>(
                      value: Filtre.terminees,
                      label: Text('Terminées'),
                    ),
                  ],
                  selected: <Filtre>{_filtre},
                  onSelectionChanged: (Set<Filtre> s) async {
                    setState(() => _filtre = s.first);
                    await _charger();
                  },
                ),
                const SizedBox(height: 8),
                Expanded(
                  child: _visibles.isEmpty
                      ? const Center(child: Text('Rien à afficher.'))
                      : ListView.builder(
                          itemCount: _visibles.length,
                          itemBuilder: (BuildContext context, int index) {
                            final Tache t = _visibles[index];
                            return Dismissible(
                              key: ValueKey<String>(t.id),
                              direction: DismissDirection.endToStart,
                              background: Container(
                                alignment: Alignment.centerRight,
                                padding: const EdgeInsets.only(right: 24),
                                color: Theme.of(context).colorScheme.error,
                                child: const Icon(Icons.delete,
                                    color: Colors.white),
                              ),
                              onDismissed: (_) async {
                                await widget.store.supprimer(t.id);
                                await _charger();
                              },
                              child: CheckboxListTile(
                                value: t.faite,
                                onChanged: (bool? v) async {
                                  await widget.store
                                      .basculer(t.id, v ?? false);
                                  await _charger();
                                },
                                title: Text(
                                  t.titre,
                                  style: TextStyle(
                                    decoration: t.faite
                                        ? TextDecoration.lineThrough
                                        : null,
                                  ),
                                ),
                              ),
                            );
                          },
                        ),
                ),
              ],
            ),
    );
  }
}
```

**Résultat :** exactement la même application, à l'œil nu. Mais le comportement interne est très différent.

### Comparaison des deux versions

| Opération | Version `shared_preferences` | Version `sqflite` |
| --- | --- | --- |
| Démarrage avec 3 000 tâches | Décode 3 000 objets | Charge la page affichée |
| Cocher une tâche | Réécrit 3 000 objets | `UPDATE` d'une ligne |
| Filtrer | Parcourt tout en Dart | `WHERE faite = 0`, avec index |
| Compter les restantes | Parcourt tout en Dart | `SELECT COUNT(*)` |
| Lignes de code du store | ~45 | ~60 |
| Lisibilité du fichier stocké | Excellente | Nulle sans outil |

### Extensions à réaliser seul

1. Ajouter une échéance (`DateTime?`) et trier par échéance croissante.
2. Ajouter une recherche par titre (`LIKE ?`) sur la version SQL.
3. Ajouter des catégories, avec une seconde table et une clé étrangère.
4. Exporter toutes les tâches dans un fichier JSON via `path_provider`.
5. Écrire les tests de `TacheSqlStore` avec `sqflite_common_ffi` (54.38).
6. Remplacer le `setState` par un `ChangeNotifier` et `provider` (chapitre 52).

---

## 54.41 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `ServicesBinding.defaultBinaryMessenger was accessed before the binding was initialized` | Appel d'un plugin avant `runApp` sans initialiser le pont natif | Ajouter `WidgetsFlutterBinding.ensureInitialized();` en première ligne de `main` |
| `MissingPluginException(No implementation found for method getAll...)` | Le paquet a été ajouté mais l'application n'a pas été relancée complètement | Arrêter l'application et relancer `flutter run` ; un hot reload ne suffit pas |
| `type 'Future<int?>' is not a subtype of type 'int?'` | `getInstance()` ou une méthode `SharedPreferencesAsync` non attendue | Ajouter `await` et rendre la fonction `async` |
| `Null check operator used on a null value` au premier lancement | `prefs.getInt('score')!` sur une clé inexistante | Remplacer `!` par `?? valeurParDefaut` |
| Les données disparaissent au bout de quelques jours | Fichier écrit dans le dossier cache ou temporaire | Utiliser `getApplicationDocumentsDirectory()` ou `getApplicationSupportDirectory()` |
| `DatabaseException(no such table: potion)` | `onCreate` modifié sans changer `version` : la base existe déjà | Incrémenter `version` et écrire `onUpgrade`, ou désinstaller l'application en développement |
| Une colonne ajoutée n'existe pas chez les anciens utilisateurs | Migration sans changement de version | Toute modification de schéma incrémente `version` et ajoute un `if (oldVersion < n)` dans `onUpgrade` |
| L'application se fige sans message pendant une transaction | Utilisation de `db` au lieu de `txn` à l'intérieur du bloc `transaction` | N'utiliser que le `txn` fourni ; faire circuler un `DatabaseExecutor` dans les DAO |
| `DatabaseException(database_closed)` | La base a été fermée puis réutilisée, ou fermée dans un `dispose` alors qu'elle est partagée | Ouvrir la base une seule fois (singleton) et ne la fermer qu'en fin de vie de l'application |
| La base reste verrouillée après un hot restart | Base ouverte plusieurs fois sans être fermée | Mémoriser l'instance (`_db ??= await _ouvrir()`) au lieu d'appeler `openDatabase` à chaque écran |
| `UnsupportedError` en modifiant une ligne renvoyée par `query()` | Les `Map` renvoyées sont en lecture seule | Copier : `Map<String, Object?>.from(ligne)` |
| `type 'int' is not a subtype of type 'double'` à la relecture JSON | JSON ne distingue pas `1` de `1.0` | Écrire `(json['prix'] as num?)?.toDouble() ?? 0` |
| `FormatException: Unexpected end of input` au démarrage | Fichier ou préférence tronqué par une écriture interrompue | Attraper `FormatException`, effacer la donnée, repartir des défauts ; écrire de façon atomique (fichier `.tmp` puis `rename`) |
| La table entière est vidée par erreur | `delete` ou `update` sans clause `where` | Toujours écrire `where:` et `whereArgs:` ; vérifier le nombre de lignes affectées |
| Un utilisateur peut détruire la base en tapant une apostrophe | Requête construite par concaténation de chaînes | Utiliser `?` et `whereArgs`, jamais l'interpolation |
| `setState() called after dispose()` après une écriture disque | `await` puis `setState` alors que l'écran a été quitté | Tester `if (!mounted) return;` après chaque `await` |
| L'application affiche un flash clair avant de passer en sombre | Thème lu dans `initState` au lieu de `main` | Lire les préférences avant `runApp` et passer la valeur en paramètre |
| `MissingPluginException` uniquement sur Windows ou Linux avec `sqflite` | `sqflite` n'a pas d'implémentation bureau native | Ajouter `sqflite_common_ffi`, appeler `sqfliteFfiInit()` et affecter `databaseFactory = databaseFactoryFfi` |
| Erreur de compilation `dart:io` sur le Web | `import 'dart:io'` est interdit dans un navigateur | Isoler le code fichier derrière un import conditionnel, ou désactiver la fonctionnalité si `kIsWeb` |
| Un jeton d'authentification lisible dans le fichier XML de l'application | Secret rangé dans `shared_preferences` | Utiliser `flutter_secure_storage` |
| Les tests de préférences échouent en cascade | `setMockInitialValues` appelé une seule fois pour tous les tests | Appeler `SharedPreferences.setMockInitialValues({})` au début de chaque test |
| L'insertion de 5 000 lignes prend plusieurs secondes | Un `await db.insert` par ligne | Regrouper dans un `batch`, lui-même dans une `transaction` |
| Un enregistrement relu ne correspond plus après une mise à jour | `enum` stocké par `index`, et l'énumération a changé | Stocker `.name` et relire avec `firstWhere(..., orElse: ...)` |

---

## 54.42 — Résumé du chapitre

| Besoin | Solution | Paquet |
| --- | --- | --- |
| Retenir un réglage simple (thème, langue, volume) | Clé-valeur, cinq types | `shared_preferences` |
| Retenir un objet unique (le profil du joueur) | JSON encodé dans une `String` | `shared_preferences` + `dart:convert` |
| Savoir si l'utilisateur a déjà vu le tutoriel | `setBool` / `getBool ?? false` | `shared_preferences` |
| Stocker un jeton, un mot de passe, une clé | Stockage chiffré par le système | `flutter_secure_storage` |
| Connaître le dossier de l'application | Six fonctions de dossier | `path_provider` |
| Construire un chemin multi-plateforme | `p.join(dossier, 'fichier.json')` | `path` |
| Écrire ou lire un fichier texte | `writeAsString` / `readAsString` | `dart:io` |
| Écrire un fichier sans risque de corruption | Fichier `.tmp` puis `rename` | `dart:io` |
| Sauvegarder une liste de moins de 1 000 objets | Un fichier JSON | `path_provider` + `dart:convert` |
| Gérer un fichier absent ou corrompu | `on PathNotFoundException` / `on FormatException` | `dart:io`, `dart:convert` |
| Stocker des milliers d'objets | Base relationnelle SQLite | `sqflite` |
| Créer le schéma la première fois | `onCreate` | `sqflite` |
| Faire évoluer le schéma | `version` + `onUpgrade` avec des `if (oldVersion < n)` | `sqflite` |
| Ajouter, lire, modifier, supprimer | `insert`, `query`, `update`, `delete` | `sqflite` |
| Filtrer, trier, limiter | `where`, `whereArgs`, `orderBy`, `limit` | `sqflite` |
| Empêcher l'injection SQL | `?` et `whereArgs`, jamais l'interpolation | `sqflite` |
| Convertir une ligne en objet | `toMap()` / `fromMap()` | — |
| Isoler le SQL du reste du projet | Patron DAO | — |
| Rendre un groupe d'écritures atomique | `db.transaction((txn) async { ... })` | `sqflite` |
| Insérer massivement | `batch()` dans une `transaction` | `sqflite` |
| Faire fonctionner SQLite sur le bureau ou en test | `sqfliteFfiInit()` + `databaseFactoryFfi` | `sqflite_common_ffi` |
| Persister sur le Web | Préférences, IndexedDB | `shared_preferences`, `drift`, `hive_ce` |
| Base typée, réactive, multi-plateforme | Génération de code au-dessus de SQLite | `drift` |
| Stockage d'objets sans SQL | Boîtes clé-valeur | `hive_ce` |
| Survivre à une coupure réseau | Réseau d'abord, repli sur le cache | `shared_preferences` ou fichier |
| Ne pas mentir à l'utilisateur hors-ligne | Bandeau « données du ... » + bouton Réessayer | — |
| Tester les préférences sans émulateur | `SharedPreferences.setMockInitialValues({})` | `shared_preferences` |
| Tester une base sans émulateur | `inMemoryDatabasePath` | `sqflite_common_ffi` |

---

## 54.43 — Exercices

### Exercice 1 — Le compteur de lancements (facile)

Écrivez une application qui affiche « C'est votre N-ième lancement ». Le compteur est incrémenté à chaque démarrage et stocké dans `shared_preferences`. Au tout premier lancement, l'application affiche « Bienvenue ! » à la place.

Contraintes : la lecture et l'incrémentation se font dans `main`, avant `runApp`.

### Exercice 2 — Le dernier onglet visité (facile)

Construisez une application à trois onglets (Carte, Inventaire, Réglages) avec une `BottomNavigationBar`. L'onglet actif au démarrage doit être celui que l'utilisateur consultait la dernière fois.

Contraintes : aucun flash sur le mauvais onglet au démarrage.

### Exercice 3 — Le niveau de difficulté (facile)

Une énumération `Difficulte { facile, normal, difficile, cauchemar }`. L'utilisateur choisit une valeur dans une liste où l'élément actif est marqué d'une puce. Le choix est persistant.

Contraintes : stockez le **nom** de la valeur, pas son index, et gérez le cas où le nom stocké ne correspond à aucune valeur connue.

### Exercice 4 — Le service de préférences (moyen)

Écrivez une classe `ReglagesService` qui encapsule `SharedPreferences` et expose : `pseudo` (défaut `'Aventurier'`), `volume` (défaut `0.8`, borné entre 0 et 1), `notifications` (défaut `true`), `sombre` (défaut `false`). Ajoutez une méthode `reinitialiser()`.

Contraintes : aucune chaîne de clé ne doit apparaître ailleurs que dans le service ; aucune valeur par défaut ne doit être dupliquée.

### Exercice 5 — La fiche de personnage en JSON (moyen)

Un objet `Personnage { nom, classe, niveau, pointsDeVie, competences }` où `competences` est une `List<String>`. L'utilisateur peut modifier le nom, monter de niveau et ajouter une compétence. Tout est persisté sous forme d'une seule chaîne JSON.

Contraintes : `fromJson` doit tolérer un champ absent, et une chaîne corrompue ne doit pas empêcher l'application de démarrer.

### Exercice 6 — Le journal de bord en fichier (moyen)

Écrivez un journal d'événements stocké dans un fichier texte du dossier Documents. Chaque ligne est horodatée. L'application affiche le contenu, le nombre de lignes et la taille du fichier en octets.

Contraintes : gérez l'absence du fichier avec `on PathNotFoundException`, et proposez un bouton « Vider le journal ».

### Exercice 7 — Le cache avec péremption (moyen)

Écrivez une classe `CacheFichier` qui stocke une chaîne dans le dossier cache avec une durée de vie. La méthode `lire()` renvoie `null` si le fichier n'existe pas **ou** s'il est plus vieux que la durée de vie. Démontrez son fonctionnement avec une durée de vie de 10 secondes.

Contraintes : utilisez `stat()` pour connaître l'âge du fichier ; affichez l'âge en secondes à l'écran.

### Exercice 8 — Le bestiaire SQL (difficile)

Créez une base `bestiaire.db` avec une table `monstre (id, nom, pv, niveau, capture)`. L'application permet d'ajouter un monstre, de les lister triés par niveau décroissant, de basculer l'état « capturé », de supprimer, et de rechercher par nom.

Contraintes : tout le SQL est dans un DAO ; la recherche utilise `LIKE ?` ; la base est ouverte une seule fois.

### Exercice 9 — La migration de schéma (difficile)

Partez d'une base en version 1 avec `monstre (id, nom, pv)`. Écrivez la version 2 qui ajoute une colonne `element TEXT NOT NULL DEFAULT 'neutre'` et une table `zone (id, nom)`.

Contraintes : l'application doit proposer deux boutons, « Ouvrir en v1 » et « Ouvrir en v2 », et afficher le schéma réel de la table après chaque ouverture, sans perdre les lignes existantes.

### Exercice 10 — Le catalogue hors-ligne (difficile)

Un catalogue d'armes chargé depuis une source réseau simulée, mis en cache dans `sqflite`. Stratégie « réseau d'abord, cache ensuite ». L'écran affiche un bandeau quand les données viennent du cache, avec leur date.

Contraintes : l'écriture du cache se fait dans une transaction contenant un `batch` ; un bouton simule la panne réseau ; si le cache est vide et le réseau absent, l'erreur est affichée avec un bouton « Réessayer ».

---

## 54.44 — Corrections des exercices

### Correction 1

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final SharedPreferences prefs = await SharedPreferences.getInstance();

  final int precedent = prefs.getInt('lancements') ?? 0;
  final int courant = precedent + 1;
  await prefs.setInt('lancements', courant);

  runApp(ApplicationLancements(lancement: courant));
}

class ApplicationLancements extends StatelessWidget {
  const ApplicationLancements({super.key, required this.lancement});

  final int lancement;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Lancements',
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: PageLancements(lancement: lancement),
    );
  }
}

class PageLancements extends StatelessWidget {
  const PageLancements({super.key, required this.lancement});

  final int lancement;

  Future<void> _reinitialiser(BuildContext context) async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    await prefs.remove('lancements');
    if (!context.mounted) return;
    ScaffoldMessenger.of(context).showSnackBar(
      const SnackBar(content: Text('Compteur remis à zéro. Relancez.')),
    );
  }

  @override
  Widget build(BuildContext context) {
    final bool premier = lancement == 1;
    return Scaffold(
      appBar: AppBar(title: const Text('Compteur de lancements')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Icon(
              premier ? Icons.celebration : Icons.replay,
              size: 96,
              color: Theme.of(context).colorScheme.primary,
            ),
            const SizedBox(height: 24),
            Text(
              premier
                  ? 'Bienvenue !'
                  : 'C\'est votre $lancement-ième lancement.',
              style: Theme.of(context).textTheme.headlineSmall,
              textAlign: TextAlign.center,
            ),
            const SizedBox(height: 32),
            OutlinedButton(
              onPressed: () => _reinitialiser(context),
              child: const Text('Remettre le compteur à zéro'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Explication :** tout se joue dans `main`. `WidgetsFlutterBinding.ensureInitialized()` autorise l'appel au plugin avant `runApp`. `prefs.getInt('lancements') ?? 0` donne `0` au premier lancement, donc `courant` vaut `1` et l'application affiche « Bienvenue ! ». La valeur est réécrite immédiatement, puis passée au widget racine par constructeur : l'interface n'a aucune lecture asynchrone à faire, donc aucun état de chargement à gérer. Le bouton de réinitialisation utilise `remove` et non `clear`, pour ne toucher qu'à cette clé.

---

### Correction 2

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final SharedPreferences prefs = await SharedPreferences.getInstance();
  final int index = prefs.getInt('ongletActif') ?? 0;
  // On borne l'index : le nombre d'onglets peut avoir changé entre deux versions.
  final int sur = index.clamp(0, 2);
  runApp(ApplicationOnglets(indexInitial: sur, prefs: prefs));
}

class ApplicationOnglets extends StatelessWidget {
  const ApplicationOnglets({
    super.key,
    required this.indexInitial,
    required this.prefs,
  });

  final int indexInitial;
  final SharedPreferences prefs;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Onglets',
      theme: ThemeData(colorSchemeSeed: Colors.green, useMaterial3: true),
      home: PageOnglets(indexInitial: indexInitial, prefs: prefs),
    );
  }
}

class PageOnglets extends StatefulWidget {
  const PageOnglets({
    super.key,
    required this.indexInitial,
    required this.prefs,
  });

  final int indexInitial;
  final SharedPreferences prefs;

  @override
  State<PageOnglets> createState() => _PageOngletsState();
}

class _PageOngletsState extends State<PageOnglets> {
  late int _index = widget.indexInitial;

  static const List<String> _titres = <String>[
    'Carte',
    'Inventaire',
    'Réglages',
  ];

  static const List<IconData> _icones = <IconData>[
    Icons.map,
    Icons.backpack,
    Icons.settings,
  ];

  Future<void> _changer(int nouveau) async {
    setState(() => _index = nouveau);
    await widget.prefs.setInt('ongletActif', nouveau);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(_titres[_index])),
      body: IndexedStack(
        index: _index,
        children: List<Widget>.generate(3, (int i) {
          return Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: <Widget>[
                Icon(_icones[i], size: 96),
                const SizedBox(height: 16),
                Text(
                  'Onglet ${_titres[i]}',
                  style: Theme.of(context).textTheme.titleLarge,
                ),
              ],
            ),
          );
        }),
      ),
      bottomNavigationBar: NavigationBar(
        selectedIndex: _index,
        onDestinationSelected: _changer,
        destinations: const <NavigationDestination>[
          NavigationDestination(icon: Icon(Icons.map), label: 'Carte'),
          NavigationDestination(
            icon: Icon(Icons.backpack),
            label: 'Inventaire',
          ),
          NavigationDestination(
            icon: Icon(Icons.settings),
            label: 'Réglages',
          ),
        ],
      ),
    );
  }
}
```

**Explication :** l'index est lu dans `main`, donc le premier `build` affiche déjà le bon onglet : il n'y a aucun flash. Le `clamp(0, 2)` est une précaution importante : si une future version supprime un onglet, un index enregistré à `3` provoquerait une `RangeError` au démarrage, chez les seuls utilisateurs concernés. L'instance de `SharedPreferences` est passée en paramètre plutôt que redemandée : les écritures suivantes sont immédiates. `IndexedStack` conserve l'état de chaque onglet, conformément au chapitre 46.

---

### Correction 3

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

enum Difficulte {
  facile('Facile', 'Les monstres vous ménagent.'),
  normal('Normal', 'L\'équilibre voulu par les concepteurs.'),
  difficile('Difficile', 'Chaque erreur se paie.'),
  cauchemar('Cauchemar', 'Une seule vie. Bonne chance.');

  const Difficulte(this.libelle, this.description);

  final String libelle;
  final String description;

  static Difficulte parNom(String? nom) {
    return Difficulte.values.firstWhere(
      (Difficulte d) => d.name == nom,
      orElse: () => Difficulte.normal,
    );
  }
}

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final SharedPreferences prefs = await SharedPreferences.getInstance();
  runApp(ApplicationDifficulte(prefs: prefs));
}

class ApplicationDifficulte extends StatelessWidget {
  const ApplicationDifficulte({super.key, required this.prefs});

  final SharedPreferences prefs;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Difficulté',
      theme: ThemeData(colorSchemeSeed: Colors.deepOrange, useMaterial3: true),
      home: PageDifficulte(prefs: prefs),
    );
  }
}

class PageDifficulte extends StatefulWidget {
  const PageDifficulte({super.key, required this.prefs});

  final SharedPreferences prefs;

  @override
  State<PageDifficulte> createState() => _PageDifficulteState();
}

class _PageDifficulteState extends State<PageDifficulte> {
  static const String _cle = 'difficulte';

  late Difficulte _choix = Difficulte.parNom(widget.prefs.getString(_cle));

  Future<void> _choisir(Difficulte? valeur) async {
    if (valeur == null) return;
    setState(() => _choix = valeur);
    await widget.prefs.setString(_cle, valeur.name);
  }

  Future<void> _corrompre() async {
    await widget.prefs.setString(_cle, 'valeur-inconnue');
    setState(() => _choix = Difficulte.parNom(widget.prefs.getString(_cle)));
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Niveau de difficulté')),
      body: Column(
        children: <Widget>[
          ...Difficulte.values.map((Difficulte d) {
            final bool actif = d == _choix;
            return ListTile(
              onTap: () => _choisir(d),
              leading: Icon(
                actif
                    ? Icons.radio_button_checked
                    : Icons.radio_button_unchecked,
                color: actif ? Theme.of(context).colorScheme.primary : null,
              ),
              title: Text(d.libelle),
              subtitle: Text(d.description),
            );
          }),
          const Divider(),
          Padding(
            padding: const EdgeInsets.all(16),
            child: Text(
              'Valeur stockée : ${widget.prefs.getString(_cle) ?? '(aucune)'}',
              style: const TextStyle(fontFamily: 'monospace'),
            ),
          ),
          OutlinedButton(
            onPressed: _corrompre,
            child: const Text('Écrire une valeur inconnue'),
          ),
        ],
      ),
    );
  }
}
```

**Explication :** l'énumération est **enrichie** : elle porte un libellé et une description, ce que Dart autorise depuis la version 3. La méthode statique `parNom` est le point central de la correction : `firstWhere` avec `orElse` renvoie `Difficulte.normal` si le nom stocké ne correspond à rien. Le bouton « Écrire une valeur inconnue » simule ce qui arrivera réellement le jour où vous renommerez ou supprimerez une valeur : l'application retombe sur le défaut au lieu de lever une `StateError`. Stocker `.name` plutôt que `.index` rend le fichier de préférences lisible et robuste au réordonnancement.

---

### Correction 4

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

// ── lib/services/reglages_service.dart ──
class ReglagesService {
  ReglagesService(this._prefs);

  static Future<ReglagesService> creer() async =>
      ReglagesService(await SharedPreferences.getInstance());

  final SharedPreferences _prefs;

  static const String _clePseudo = 'pseudo';
  static const String _cleVolume = 'volume';
  static const String _cleNotifications = 'notifications';
  static const String _cleSombre = 'sombre';

  static const String pseudoDefaut = 'Aventurier';
  static const double volumeDefaut = 0.8;
  static const bool notificationsDefaut = true;
  static const bool sombreDefaut = false;

  String get pseudo => _prefs.getString(_clePseudo) ?? pseudoDefaut;

  Future<void> setPseudo(String v) async {
    final String propre = v.trim();
    if (propre.isEmpty) {
      await _prefs.remove(_clePseudo);
    } else {
      await _prefs.setString(_clePseudo, propre);
    }
  }

  double get volume => _prefs.getDouble(_cleVolume) ?? volumeDefaut;

  Future<void> setVolume(double v) async =>
      _prefs.setDouble(_cleVolume, v.clamp(0.0, 1.0));

  bool get notifications =>
      _prefs.getBool(_cleNotifications) ?? notificationsDefaut;

  Future<void> setNotifications(bool v) async =>
      _prefs.setBool(_cleNotifications, v);

  bool get sombre => _prefs.getBool(_cleSombre) ?? sombreDefaut;

  Future<void> setSombre(bool v) async => _prefs.setBool(_cleSombre, v);

  Future<void> reinitialiser() async {
    for (final String cle in <String>[
      _clePseudo,
      _cleVolume,
      _cleNotifications,
      _cleSombre,
    ]) {
      await _prefs.remove(cle);
    }
  }
}

// ── lib/main.dart ──
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(ApplicationReglages(service: await ReglagesService.creer()));
}

class ApplicationReglages extends StatefulWidget {
  const ApplicationReglages({super.key, required this.service});

  final ReglagesService service;

  @override
  State<ApplicationReglages> createState() => _ApplicationReglagesState();
}

class _ApplicationReglagesState extends State<ApplicationReglages> {
  late bool _sombre = widget.service.sombre;

  void _rafraichir() => setState(() => _sombre = widget.service.sombre);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Réglages',
      themeMode: _sombre ? ThemeMode.dark : ThemeMode.light,
      theme: ThemeData(
        colorSchemeSeed: Colors.blue,
        brightness: Brightness.light,
        useMaterial3: true,
      ),
      darkTheme: ThemeData(
        colorSchemeSeed: Colors.blue,
        brightness: Brightness.dark,
        useMaterial3: true,
      ),
      home: PageReglages(
        service: widget.service,
        onChange: _rafraichir,
      ),
    );
  }
}

class PageReglages extends StatefulWidget {
  const PageReglages({
    super.key,
    required this.service,
    required this.onChange,
  });

  final ReglagesService service;
  final VoidCallback onChange;

  @override
  State<PageReglages> createState() => _PageReglagesState();
}

class _PageReglagesState extends State<PageReglages> {
  late final TextEditingController _pseudo =
      TextEditingController(text: widget.service.pseudo);
  late double _volume = widget.service.volume;
  late bool _notifications = widget.service.notifications;

  @override
  void dispose() {
    _pseudo.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final ReglagesService s = widget.service;
    return Scaffold(
      appBar: AppBar(title: const Text('Réglages')),
      body: ListView(
        padding: const EdgeInsets.all(24),
        children: <Widget>[
          TextField(
            controller: _pseudo,
            decoration: const InputDecoration(
              labelText: 'Pseudo',
              border: OutlineInputBorder(),
            ),
            onSubmitted: (String v) async {
              await s.setPseudo(v);
              if (!mounted) return;
              setState(() => _pseudo.text = s.pseudo);
            },
          ),
          const SizedBox(height: 24),
          Text('Volume : ${(_volume * 100).round()} %'),
          Slider(
            value: _volume,
            onChanged: (double v) => setState(() => _volume = v),
            onChangeEnd: s.setVolume,
          ),
          SwitchListTile(
            title: const Text('Notifications'),
            value: _notifications,
            onChanged: (bool v) async {
              await s.setNotifications(v);
              if (!mounted) return;
              setState(() => _notifications = v);
            },
          ),
          SwitchListTile(
            title: const Text('Thème sombre'),
            value: s.sombre,
            onChanged: (bool v) async {
              await s.setSombre(v);
              widget.onChange();
            },
          ),
          const SizedBox(height: 32),
          OutlinedButton(
            onPressed: () async {
              await s.reinitialiser();
              if (!mounted) return;
              setState(() {
                _pseudo.text = s.pseudo;
                _volume = s.volume;
                _notifications = s.notifications;
              });
              widget.onChange();
            },
            child: const Text('Réinitialiser'),
          ),
        ],
      ),
    );
  }
}
```

**Explication :** les quatre clés et les quatre valeurs par défaut sont déclarées **une seule fois**, en `static const`. Aucun widget ne manipule de chaîne littérale : une faute de frappe devient une erreur de compilation. La validation est faite dans le service (`trim`, `clamp`), donc elle s'applique quel que soit l'appelant. Le service est également testable : son constructeur reçoit un `SharedPreferences`, ce qui permet d'injecter les valeurs de test avec `setMockInitialValues` (section 54.38). Enfin, `setPseudo('')` supprime la clé plutôt que d'écrire une chaîne vide : on retombe naturellement sur le défaut.

---

### Correction 5

```dart
import 'dart:convert';

import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

// ── lib/models/personnage.dart ──
class Personnage {
  const Personnage({
    required this.nom,
    required this.classe,
    required this.niveau,
    required this.pointsDeVie,
    required this.competences,
  });

  final String nom;
  final String classe;
  final int niveau;
  final double pointsDeVie;
  final List<String> competences;

  static const Personnage defaut = Personnage(
    nom: 'Aria',
    classe: 'Magicienne',
    niveau: 1,
    pointsDeVie: 100,
    competences: <String>['Boule de feu'],
  );

  Personnage copyWith({
    String? nom,
    int? niveau,
    double? pointsDeVie,
    List<String>? competences,
  }) {
    return Personnage(
      nom: nom ?? this.nom,
      classe: classe,
      niveau: niveau ?? this.niveau,
      pointsDeVie: pointsDeVie ?? this.pointsDeVie,
      competences: competences ?? this.competences,
    );
  }

  Map<String, dynamic> toJson() => <String, dynamic>{
        'nom': nom,
        'classe': classe,
        'niveau': niveau,
        'pointsDeVie': pointsDeVie,
        'competences': competences,
      };

  factory Personnage.fromJson(Map<String, dynamic> j) => Personnage(
        nom: j['nom'] as String? ?? defaut.nom,
        classe: j['classe'] as String? ?? defaut.classe,
        niveau: j['niveau'] as int? ?? defaut.niveau,
        pointsDeVie:
            (j['pointsDeVie'] as num?)?.toDouble() ?? defaut.pointsDeVie,
        competences: (j['competences'] as List<dynamic>? ?? <dynamic>[])
            .whereType<String>()
            .toList(),
      );
}

// ── lib/services/personnage_store.dart ──
class PersonnageStore {
  PersonnageStore(this._prefs);

  static const String _cle = 'personnage';

  final SharedPreferences _prefs;

  Personnage lire() {
    final String? brut = _prefs.getString(_cle);
    if (brut == null) return Personnage.defaut;
    try {
      return Personnage.fromJson(jsonDecode(brut) as Map<String, dynamic>);
    } on FormatException {
      _prefs.remove(_cle);
      return Personnage.defaut;
    } on TypeError {
      _prefs.remove(_cle);
      return Personnage.defaut;
    }
  }

  Future<void> ecrire(Personnage p) async =>
      _prefs.setString(_cle, jsonEncode(p.toJson()));

  Future<void> corrompre() async => _prefs.setString(_cle, '{"nom": "Ari');
}

// ── lib/main.dart ──
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final SharedPreferences prefs = await SharedPreferences.getInstance();
  runApp(ApplicationFiche(store: PersonnageStore(prefs)));
}

class ApplicationFiche extends StatelessWidget {
  const ApplicationFiche({super.key, required this.store});

  final PersonnageStore store;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Fiche de personnage',
      theme: ThemeData(colorSchemeSeed: Colors.purple, useMaterial3: true),
      home: PageFiche(store: store),
    );
  }
}

class PageFiche extends StatefulWidget {
  const PageFiche({super.key, required this.store});

  final PersonnageStore store;

  @override
  State<PageFiche> createState() => _PageFicheState();
}

class _PageFicheState extends State<PageFiche> {
  late Personnage _p = widget.store.lire();
  final TextEditingController _competence = TextEditingController();

  @override
  void dispose() {
    _competence.dispose();
    super.dispose();
  }

  Future<void> _appliquer(Personnage nouveau) async {
    setState(() => _p = nouveau);
    await widget.store.ecrire(nouveau);
  }

  Future<void> _renommer() async {
    final TextEditingController champ = TextEditingController(text: _p.nom);
    final String? nom = await showDialog<String>(
      context: context,
      builder: (BuildContext context) => AlertDialog(
        title: const Text('Nom du personnage'),
        content: TextField(controller: champ, autofocus: true),
        actions: <Widget>[
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('Annuler'),
          ),
          FilledButton(
            onPressed: () => Navigator.pop(context, champ.text.trim()),
            child: const Text('Valider'),
          ),
        ],
      ),
    );
    champ.dispose();
    if (nom == null || nom.isEmpty) return;
    await _appliquer(_p.copyWith(nom: nom));
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(_p.nom),
        actions: <Widget>[
          IconButton(onPressed: _renommer, icon: const Icon(Icons.edit)),
        ],
      ),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: <Widget>[
          ListTile(
            leading: const Icon(Icons.auto_awesome),
            title: const Text('Classe'),
            trailing: Text(_p.classe),
          ),
          ListTile(
            leading: const Icon(Icons.military_tech),
            title: const Text('Niveau'),
            trailing: Text('${_p.niveau}'),
          ),
          ListTile(
            leading: const Icon(Icons.favorite),
            title: const Text('Points de vie'),
            trailing: Text(_p.pointsDeVie.toStringAsFixed(0)),
          ),
          const Divider(),
          const Padding(
            padding: EdgeInsets.symmetric(vertical: 8),
            child: Text('Compétences'),
          ),
          Wrap(
            spacing: 8,
            children: _p.competences
                .map((String c) => Chip(label: Text(c)))
                .toList(),
          ),
          const SizedBox(height: 16),
          Row(
            children: <Widget>[
              Expanded(
                child: TextField(
                  controller: _competence,
                  decoration: const InputDecoration(
                    labelText: 'Nouvelle compétence',
                    border: OutlineInputBorder(),
                  ),
                ),
              ),
              const SizedBox(width: 12),
              FilledButton(
                onPressed: () async {
                  final String c = _competence.text.trim();
                  if (c.isEmpty) return;
                  _competence.clear();
                  await _appliquer(
                    _p.copyWith(
                      competences: <String>[..._p.competences, c],
                    ),
                  );
                },
                child: const Text('Ajouter'),
              ),
            ],
          ),
          const SizedBox(height: 24),
          FilledButton.icon(
            onPressed: () => _appliquer(
              _p.copyWith(
                niveau: _p.niveau + 1,
                pointsDeVie: _p.pointsDeVie + 15,
              ),
            ),
            icon: const Icon(Icons.arrow_upward),
            label: const Text('Monter de niveau'),
          ),
          const SizedBox(height: 8),
          OutlinedButton(
            onPressed: () async {
              await widget.store.corrompre();
              if (!mounted) return;
              setState(() => _p = widget.store.lire());
              if (!mounted) return;
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(
                  content: Text('Donnée corrompue détectée et effacée.'),
                ),
              );
            },
            child: const Text('Simuler une sauvegarde corrompue'),
          ),
        ],
      ),
    );
  }
}
```

**Explication :** le personnage entier tient dans une seule clé, sous forme de chaîne JSON. `fromJson` applique un `??` sur **chaque** champ, en s'appuyant sur la constante `Personnage.defaut` : une sauvegarde écrite par une version antérieure, où `pointsDeVie` n'existait pas, se relit sans erreur. Le `whereType<String>()` protège la liste de compétences contre un élément d'un autre type. Le store attrape `FormatException` (JSON tronqué) **et** `TypeError` (JSON valide mais de la mauvaise forme, par exemple un tableau à la racine), efface la clé et repart des défauts. Le bouton de simulation prouve que l'application survit à une corruption au lieu de refuser de démarrer.

---

### Correction 6

```dart
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:path_provider/path_provider.dart';

// ── lib/services/journal_service.dart ──
class JournalService {
  static const String _nom = 'journal.txt';

  Future<File> _fichier() async {
    final Directory dossier = await getApplicationDocumentsDirectory();
    return File(p.join(dossier.path, _nom));
  }

  Future<List<String>> lignes() async {
    final File f = await _fichier();
    try {
      return await f.readAsLines();
    } on PathNotFoundException {
      return <String>[];
    } on FileSystemException {
      return <String>[];
    }
  }

  Future<int> taille() async {
    final File f = await _fichier();
    try {
      return await f.length();
    } on FileSystemException {
      return 0;
    }
  }

  Future<String> chemin() async => (await _fichier()).path;

  Future<void> ajouter(String evenement) async {
    final File f = await _fichier();
    final String heure = DateTime.now().toIso8601String().substring(0, 19);
    await f.writeAsString('[$heure] $evenement\n', mode: FileMode.append);
  }

  Future<void> vider() async {
    final File f = await _fichier();
    if (await f.exists()) await f.delete();
  }
}

// ── lib/main.dart ──
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const ApplicationJournal());
}

class ApplicationJournal extends StatelessWidget {
  const ApplicationJournal({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Journal de bord',
      theme: ThemeData(colorSchemeSeed: Colors.brown, useMaterial3: true),
      home: const PageJournal(),
    );
  }
}

class PageJournal extends StatefulWidget {
  const PageJournal({super.key});

  @override
  State<PageJournal> createState() => _PageJournalState();
}

class _PageJournalState extends State<PageJournal> {
  final JournalService _service = JournalService();
  final TextEditingController _saisie = TextEditingController();

  List<String> _lignes = <String>[];
  int _octets = 0;
  String _chemin = '';

  @override
  void initState() {
    super.initState();
    _rafraichir();
  }

  @override
  void dispose() {
    _saisie.dispose();
    super.dispose();
  }

  Future<void> _rafraichir() async {
    final List<String> lignes = await _service.lignes();
    final int octets = await _service.taille();
    final String chemin = await _service.chemin();
    if (!mounted) return;
    setState(() {
      _lignes = lignes;
      _octets = octets;
      _chemin = chemin;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Journal — ${_lignes.length} lignes')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            SelectableText(
              _chemin,
              style: const TextStyle(fontSize: 11, fontFamily: 'monospace'),
            ),
            const SizedBox(height: 4),
            Text('Taille : $_octets octets'),
            const SizedBox(height: 12),
            Expanded(
              child: _lignes.isEmpty
                  ? const Center(child: Text('Journal vide ou inexistant.'))
                  : ListView.builder(
                      itemCount: _lignes.length,
                      itemBuilder: (BuildContext context, int i) => Text(
                        _lignes[i],
                        style: const TextStyle(
                          fontFamily: 'monospace',
                          fontSize: 12,
                        ),
                      ),
                    ),
            ),
            const SizedBox(height: 12),
            TextField(
              controller: _saisie,
              decoration: const InputDecoration(
                labelText: 'Événement',
                border: OutlineInputBorder(),
              ),
              onSubmitted: (_) => _ajouter(),
            ),
            const SizedBox(height: 12),
            Row(
              children: <Widget>[
                Expanded(
                  child: FilledButton(
                    onPressed: _ajouter,
                    child: const Text('Ajouter'),
                  ),
                ),
                const SizedBox(width: 12),
                Expanded(
                  child: OutlinedButton(
                    onPressed: () async {
                      await _service.vider();
                      await _rafraichir();
                    },
                    child: const Text('Vider le journal'),
                  ),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Future<void> _ajouter() async {
    final String texte = _saisie.text.trim();
    if (texte.isEmpty) return;
    await _service.ajouter(texte);
    _saisie.clear();
    await _rafraichir();
  }
}
```

**Explication :** le service ne connaît ni widget ni `BuildContext` : il expose quatre opérations métier. `readAsLines()` évite de découper la chaîne à la main, et `length()` donne la taille en octets sans lire le contenu. Les trois cas d'absence sont traités : `PathNotFoundException` au premier lancement (liste vide, taille zéro), `FileSystemException` pour tout autre problème de disque, et le bouton « Vider » qui supprime le fichier après avoir vérifié son existence. `FileMode.append` est ce qui distingue un journal d'un simple fichier de sauvegarde : chaque écriture s'ajoute au lieu d'écraser. Le dossier choisi est Documents, car un journal n'est pas régénérable.

---

### Correction 7

```dart
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:path_provider/path_provider.dart';

// ── lib/services/cache_fichier.dart ──
class CacheFichier {
  CacheFichier({required this.nom, required this.dureeDeVie});

  final String nom;
  final Duration dureeDeVie;

  Future<File> _fichier() async {
    final Directory dossier = await getApplicationCacheDirectory();
    return File(p.join(dossier.path, nom));
  }

  /// Renvoie l'âge du cache, ou null si le fichier n'existe pas.
  Future<Duration?> age() async {
    final File f = await _fichier();
    try {
      final FileStat infos = await f.stat();
      if (infos.type == FileSystemEntityType.notFound) return null;
      return DateTime.now().difference(infos.modified);
    } on FileSystemException {
      return null;
    }
  }

  /// Renvoie le contenu, ou null si absent ou périmé.
  Future<String?> lire() async {
    final Duration? actuel = await age();
    if (actuel == null) return null;
    if (actuel > dureeDeVie) return null;
    final File f = await _fichier();
    try {
      return await f.readAsString();
    } on FileSystemException {
      return null;
    }
  }

  Future<void> ecrire(String contenu) async {
    final File f = await _fichier();
    await f.create(recursive: true);
    await f.writeAsString(contenu, flush: true);
  }

  Future<void> invalider() async {
    final File f = await _fichier();
    if (await f.exists()) await f.delete();
  }
}

// ── lib/main.dart ──
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const ApplicationCache());
}

class ApplicationCache extends StatelessWidget {
  const ApplicationCache({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Cache avec péremption',
      theme: ThemeData(colorSchemeSeed: Colors.cyan, useMaterial3: true),
      home: const PageCache(),
    );
  }
}

class PageCache extends StatefulWidget {
  const PageCache({super.key});

  @override
  State<PageCache> createState() => _PageCacheState();
}

class _PageCacheState extends State<PageCache> {
  final CacheFichier _cache = CacheFichier(
    nom: 'bestiaire.json',
    dureeDeVie: const Duration(seconds: 10),
  );

  String _contenu = '(non lu)';
  String _ageTexte = '(inconnu)';
  int _generation = 0;

  @override
  void initState() {
    super.initState();
    _rafraichir();
  }

  Future<void> _rafraichir() async {
    final String? valeur = await _cache.lire();
    final Duration? age = await _cache.age();
    if (!mounted) return;
    setState(() {
      _contenu = valeur ?? '(absent ou périmé)';
      _ageTexte =
          age == null ? '(pas de fichier)' : '${age.inSeconds} s';
    });
  }

  Future<void> _remplir() async {
    _generation++;
    await _cache.ecrire('{"generation": $_generation}');
    await _rafraichir();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Cache — durée de vie 10 s')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: <Widget>[
                    Text('Âge du fichier : $_ageTexte'),
                    const SizedBox(height: 8),
                    Text(
                      'lire() renvoie : $_contenu',
                      style: const TextStyle(fontFamily: 'monospace'),
                    ),
                  ],
                ),
              ),
            ),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: _remplir,
              child: const Text('Écrire dans le cache'),
            ),
            const SizedBox(height: 8),
            FilledButton.tonal(
              onPressed: _rafraichir,
              child: const Text('Relire (observez après 10 s)'),
            ),
            const SizedBox(height: 8),
            OutlinedButton(
              onPressed: () async {
                await _cache.invalider();
                await _rafraichir();
              },
              child: const Text('Invalider le cache'),
            ),
            const SizedBox(height: 24),
            const Text(
              'Écrivez, relisez immédiatement : le contenu apparaît.\n'
              'Attendez 11 secondes, relisez : le cache est périmé.',
              textAlign: TextAlign.center,
            ),
          ],
        ),
      ),
    );
  }
}
```

**Explication :** la péremption ne se stocke pas, elle se **calcule**. `stat()` fournit `modified`, la date de dernière écriture ; la différence avec `DateTime.now()` donne l'âge. `lire()` combine trois cas dans un seul type de retour : fichier absent, fichier périmé, fichier valide — les deux premiers renvoyant `null`, ce qui simplifie l'appelant (`final donnees = await cache.lire() ?? await reseau()`). Le dossier choisi est le dossier **cache** : c'est précisément l'usage prévu, et sa suppression par le système est sans conséquence puisque la donnée est régénérable. `stat()` sur un fichier absent ne lève pas d'exception : il renvoie un `FileStat` dont le `type` vaut `notFound`, d'où le test explicite.

---

### Correction 8

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

// ── lib/models/monstre.dart ──
class Monstre {
  const Monstre({
    required this.id,
    required this.nom,
    required this.pv,
    required this.niveau,
    required this.capture,
  });

  final String id;
  final String nom;
  final int pv;
  final int niveau;
  final bool capture;

  Map<String, Object?> toMap() => <String, Object?>{
        'id': id,
        'nom': nom,
        'pv': pv,
        'niveau': niveau,
        'capture': capture ? 1 : 0,
      };

  factory Monstre.fromMap(Map<String, Object?> m) => Monstre(
        id: m['id']! as String,
        nom: m['nom']! as String,
        pv: m['pv']! as int,
        niveau: m['niveau']! as int,
        capture: (m['capture']! as int) == 1,
      );
}

// ── lib/data/bestiaire_database.dart ──
class BestiaireDatabase {
  BestiaireDatabase._();

  static final BestiaireDatabase instance = BestiaireDatabase._();

  Database? _db;

  Future<Database> get database async => _db ??= await _ouvrir();

  Future<Database> _ouvrir() async {
    final String chemin = p.join(await getDatabasesPath(), 'bestiaire.db');
    return openDatabase(
      chemin,
      version: 1,
      onCreate: (Database db, int version) async {
        await db.execute('''
          CREATE TABLE monstre (
            id      TEXT    PRIMARY KEY,
            nom     TEXT    NOT NULL,
            pv      INTEGER NOT NULL,
            niveau  INTEGER NOT NULL,
            capture INTEGER NOT NULL DEFAULT 0
          )
        ''');
        await db.execute('CREATE INDEX idx_monstre_niveau ON monstre (niveau)');
      },
    );
  }
}

// ── lib/data/monstre_dao.dart ──
class MonstreDao {
  MonstreDao(this._db);

  final Database _db;

  static const String _table = 'monstre';

  Future<List<Monstre>> toutes({String recherche = ''}) async {
    final String terme = recherche.trim();
    final List<Map<String, Object?>> lignes = await _db.query(
      _table,
      where: terme.isEmpty ? null : 'nom LIKE ?',
      whereArgs: terme.isEmpty ? null : <Object?>['%$terme%'],
      orderBy: 'niveau DESC, nom ASC',
    );
    return lignes.map(Monstre.fromMap).toList();
  }

  Future<int> compterCaptures() async {
    return Sqflite.firstIntValue(
          await _db.rawQuery(
            'SELECT COUNT(*) FROM $_table WHERE capture = 1',
          ),
        ) ??
        0;
  }

  Future<void> enregistrer(Monstre m) async {
    await _db.insert(
      _table,
      m.toMap(),
      conflictAlgorithm: ConflictAlgorithm.replace,
    );
  }

  Future<void> basculerCapture(String id, bool capture) async {
    await _db.update(
      _table,
      <String, Object?>{'capture': capture ? 1 : 0},
      where: 'id = ?',
      whereArgs: <Object?>[id],
    );
  }

  Future<int> supprimer(String id) async =>
      _db.delete(_table, where: 'id = ?', whereArgs: <Object?>[id]);
}

// ── lib/main.dart ──
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final Database db = await BestiaireDatabase.instance.database;
  runApp(ApplicationBestiaire(dao: MonstreDao(db)));
}

class ApplicationBestiaire extends StatelessWidget {
  const ApplicationBestiaire({super.key, required this.dao});

  final MonstreDao dao;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Bestiaire',
      theme: ThemeData(colorSchemeSeed: Colors.green, useMaterial3: true),
      home: PageBestiaire(dao: dao),
    );
  }
}

class PageBestiaire extends StatefulWidget {
  const PageBestiaire({super.key, required this.dao});

  final MonstreDao dao;

  @override
  State<PageBestiaire> createState() => _PageBestiaireState();
}

class _PageBestiaireState extends State<PageBestiaire> {
  final TextEditingController _recherche = TextEditingController();
  List<Monstre> _monstres = <Monstre>[];
  int _captures = 0;
  bool _charge = false;
  int _compteur = 0;

  static const List<String> _noms = <String>[
    'Gobelin',
    'Squelette',
    'Loup des cavernes',
    'Golem de pierre',
    'Dragon rouge',
  ];

  @override
  void initState() {
    super.initState();
    _charger();
  }

  @override
  void dispose() {
    _recherche.dispose();
    super.dispose();
  }

  Future<void> _charger() async {
    final List<Monstre> liste =
        await widget.dao.toutes(recherche: _recherche.text);
    final int captures = await widget.dao.compterCaptures();
    if (!mounted) return;
    setState(() {
      _monstres = liste;
      _captures = captures;
      _charge = true;
    });
  }

  Future<void> _ajouter() async {
    _compteur++;
    final String nom = _noms[_compteur % _noms.length];
    await widget.dao.enregistrer(
      Monstre(
        id: DateTime.now().microsecondsSinceEpoch.toString(),
        nom: nom,
        pv: 20 + (_compteur * 17) % 380,
        niveau: 1 + (_compteur * 7) % 40,
        capture: false,
      ),
    );
    await _charger();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Bestiaire — $_captures capturés'),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _ajouter,
        child: const Icon(Icons.add),
      ),
      body: Column(
        children: <Widget>[
          Padding(
            padding: const EdgeInsets.all(16),
            child: TextField(
              controller: _recherche,
              onChanged: (_) => _charger(),
              decoration: const InputDecoration(
                labelText: 'Rechercher un monstre',
                prefixIcon: Icon(Icons.search),
                border: OutlineInputBorder(),
              ),
            ),
          ),
          Expanded(
            child: !_charge
                ? const Center(child: CircularProgressIndicator())
                : _monstres.isEmpty
                    ? const Center(child: Text('Aucun monstre.'))
                    : ListView.builder(
                        itemCount: _monstres.length,
                        itemBuilder: (BuildContext context, int i) {
                          final Monstre m = _monstres[i];
                          return Dismissible(
                            key: ValueKey<String>(m.id),
                            direction: DismissDirection.endToStart,
                            background: Container(
                              alignment: Alignment.centerRight,
                              padding: const EdgeInsets.only(right: 24),
                              color: Theme.of(context).colorScheme.error,
                              child: const Icon(
                                Icons.delete,
                                color: Colors.white,
                              ),
                            ),
                            onDismissed: (_) async {
                              await widget.dao.supprimer(m.id);
                              await _charger();
                            },
                            child: CheckboxListTile(
                              value: m.capture,
                              onChanged: (bool? v) async {
                                await widget.dao
                                    .basculerCapture(m.id, v ?? false);
                                await _charger();
                              },
                              secondary: CircleAvatar(
                                child: Text('${m.niveau}'),
                              ),
                              title: Text(m.nom),
                              subtitle: Text('${m.pv} PV'),
                            ),
                          );
                        },
                      ),
          ),
        ],
      ),
    );
  }
}
```

**Explication :** trois couches nettement séparées. `BestiaireDatabase` ouvre la base **une seule fois** grâce à `_db ??= await _ouvrir()` : rouvrir la base à chaque écran provoquerait des verrous et des fuites. `MonstreDao` contient tout le SQL et ne renvoie que des `Monstre` ; il expose même le comptage sous forme d'`int`, jamais de `Map`. Enfin, l'interface n'importe pas `sqflite`. La recherche utilise `LIKE ?` avec `whereArgs`, donc une apostrophe saisie par l'utilisateur ne casse rien. L'index sur `niveau` accélère le `ORDER BY niveau DESC`. Le `where` est mis à `null` quand la recherche est vide, ce qui produit un `SELECT` sans clause plutôt qu'un `LIKE '%%'` inutile.

---

### Correction 9

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const ApplicationMigration());
}

class ApplicationMigration extends StatelessWidget {
  const ApplicationMigration({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Migration de schéma',
      theme: ThemeData(colorSchemeSeed: Colors.deepPurple, useMaterial3: true),
      home: const PageMigration(),
    );
  }
}

class PageMigration extends StatefulWidget {
  const PageMigration({super.key});

  @override
  State<PageMigration> createState() => _PageMigrationState();
}

class _PageMigrationState extends State<PageMigration> {
  Database? _db;
  String _rapport = 'Aucune base ouverte.';
  int _versionOuverte = 0;

  Future<String> _chemin() async =>
      p.join(await getDatabasesPath(), 'migration.db');

  Future<void> _fermer() async {
    await _db?.close();
    _db = null;
  }

  // ── Schéma v1 ─────────────────────────────────────────────
  Future<void> _creerV1(Database db) async {
    await db.execute('''
      CREATE TABLE monstre (
        id  TEXT    PRIMARY KEY,
        nom TEXT    NOT NULL,
        pv  INTEGER NOT NULL
      )
    ''');
  }

  // ── Schéma v2 complet, pour un nouvel utilisateur ─────────
  Future<void> _creerV2(Database db) async {
    await db.execute('''
      CREATE TABLE monstre (
        id      TEXT    PRIMARY KEY,
        nom     TEXT    NOT NULL,
        pv      INTEGER NOT NULL,
        element TEXT    NOT NULL DEFAULT 'neutre'
      )
    ''');
    await db.execute('''
      CREATE TABLE zone (
        id  TEXT PRIMARY KEY,
        nom TEXT NOT NULL
      )
    ''');
  }

  // ── Migration v1 → v2, pour un utilisateur existant ───────
  Future<void> _migrer(Database db, int oldVersion, int newVersion) async {
    if (oldVersion < 2) {
      await db.execute(
        "ALTER TABLE monstre ADD COLUMN element TEXT NOT NULL DEFAULT 'neutre'",
      );
      await db.execute('''
        CREATE TABLE zone (
          id  TEXT PRIMARY KEY,
          nom TEXT NOT NULL
        )
      ''');
    }
  }

  Future<void> _ouvrir(int version) async {
    await _fermer();
    final String chemin = await _chemin();
    final Database db = await openDatabase(
      chemin,
      version: version,
      onCreate: (Database db, int v) async {
        if (v == 1) {
          await _creerV1(db);
        } else {
          await _creerV2(db);
        }
      },
      onUpgrade: _migrer,
    );
    _db = db;
    _versionOuverte = version;
    await _decrire();
  }

  Future<void> _decrire() async {
    final Database? db = _db;
    if (db == null) return;

    final List<Map<String, Object?>> colonnes =
        await db.rawQuery('PRAGMA table_info(monstre)');
    final List<Map<String, Object?>> tables = await db.rawQuery(
      "SELECT name FROM sqlite_master WHERE type = 'table' "
      "AND name NOT LIKE 'sqlite_%' ORDER BY name",
    );
    final List<Map<String, Object?>> lignes = await db.query('monstre');

    final StringBuffer texte = StringBuffer()
      ..writeln('Version ouverte : $_versionOuverte')
      ..writeln('Version réelle du fichier : ${await db.getVersion()}')
      ..writeln('')
      ..writeln('Tables : '
          '${tables.map((Map<String, Object?> t) => t['name']).join(', ')}')
      ..writeln('')
      ..writeln('Colonnes de « monstre » :');
    for (final Map<String, Object?> c in colonnes) {
      texte.writeln('  - ${c['name']} (${c['type']})');
    }
    texte
      ..writeln('')
      ..writeln('Lignes (${lignes.length}) :');
    for (final Map<String, Object?> l in lignes) {
      texte.writeln('  $l');
    }

    if (!mounted) return;
    setState(() => _rapport = texte.toString());
  }

  Future<void> _insererMonstre() async {
    final Database? db = _db;
    if (db == null) return;
    await db.insert(
      'monstre',
      <String, Object?>{
        'id': DateTime.now().microsecondsSinceEpoch.toString(),
        'nom': 'Gobelin',
        'pv': 30,
      },
      conflictAlgorithm: ConflictAlgorithm.replace,
    );
    await _decrire();
  }

  Future<void> _repartirDeZero() async {
    await _fermer();
    await deleteDatabase(await _chemin());
    if (!mounted) return;
    setState(() {
      _rapport = 'Fichier supprimé. Ouvrez en v1 pour recommencer.';
      _versionOuverte = 0;
    });
  }

  @override
  void dispose() {
    _db?.close();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Migration de schéma')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            Wrap(
              spacing: 8,
              runSpacing: 8,
              children: <Widget>[
                FilledButton(
                  onPressed: () => _ouvrir(1),
                  child: const Text('Ouvrir en v1'),
                ),
                FilledButton(
                  onPressed: () => _ouvrir(2),
                  child: const Text('Ouvrir en v2'),
                ),
                OutlinedButton(
                  onPressed: _insererMonstre,
                  child: const Text('Insérer un gobelin'),
                ),
                TextButton(
                  onPressed: _repartirDeZero,
                  child: const Text('Supprimer la base'),
                ),
              ],
            ),
            const SizedBox(height: 16),
            Expanded(
              child: Container(
                width: double.infinity,
                padding: const EdgeInsets.all(12),
                decoration: BoxDecoration(
                  border: Border.all(
                    color: Theme.of(context).colorScheme.outlineVariant,
                  ),
                  borderRadius: BorderRadius.circular(8),
                ),
                child: SingleChildScrollView(
                  child: Text(
                    _rapport,
                    style: const TextStyle(
                      fontFamily: 'monospace',
                      fontSize: 12,
                    ),
                  ),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat attendu, dans l'ordre :** ouvrez en v1, insérez deux gobelins, puis ouvrez en v2.

```text
Version ouverte : 2
Version réelle du fichier : 2

Tables : monstre, zone

Colonnes de « monstre » :
  - id (TEXT)
  - nom (TEXT)
  - pv (INTEGER)
  - element (TEXT)

Lignes (2) :
  {id: 1773487261000123, nom: Gobelin, pv: 30, element: neutre}
  {id: 1773487263000456, nom: Gobelin, pv: 30, element: neutre}
```

**Explication :** les deux gobelins insérés en v1 sont toujours là après la migration, et ils ont automatiquement reçu `element = 'neutre'` grâce au `DEFAULT` du `ALTER TABLE`. C'est le point central : **une colonne ajoutée doit avoir une valeur par défaut ou être nullable**, sinon SQLite refuse la migration puisqu'il ne saurait pas quoi mettre dans les lignes existantes. `onCreate` produit le schéma **final** (`_creerV2`), pour qu'un nouvel utilisateur n'ait jamais à exécuter les migrations ; le branchement sur `v == 1` n'existe ici que pour la démonstration. `PRAGMA table_info` et `sqlite_master` permettent d'inspecter le schéma réellement présent sur le disque, ce qui est l'outil de diagnostic à connaître quand une migration ne se comporte pas comme prévu. Enfin, l'ancienne base est fermée avant toute réouverture : deux `Database` ouverts sur le même fichier provoquent des verrous.

---

### Correction 10

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

// ── lib/models/arme.dart ──
class Arme {
  const Arme({required this.id, required this.nom, required this.degats});

  final String id;
  final String nom;
  final int degats;

  Map<String, Object?> toMap() =>
      <String, Object?>{'id': id, 'nom': nom, 'degats': degats};

  factory Arme.fromMap(Map<String, Object?> m) => Arme(
        id: m['id']! as String,
        nom: m['nom']! as String,
        degats: m['degats']! as int,
      );
}

class Resultat {
  const Resultat({
    required this.armes,
    required this.provenance,
    this.dateCache,
  });

  final List<Arme> armes;
  final String provenance; // 'réseau' ou 'cache'
  final DateTime? dateCache;

  bool get estPerime => provenance == 'cache';
}

// ── lib/data/arme_repository.dart ──
class ArmeRepository {
  ArmeRepository(this._db);

  final Database _db;

  bool panne = false;

  static Future<ArmeRepository> ouvrir() async {
    final String chemin = p.join(await getDatabasesPath(), 'catalogue.db');
    final Database db = await openDatabase(
      chemin,
      version: 1,
      onCreate: (Database db, int version) async {
        await db.execute('''
          CREATE TABLE arme (
            id     TEXT    PRIMARY KEY,
            nom    TEXT    NOT NULL,
            degats INTEGER NOT NULL
          )
        ''');
        await db.execute('''
          CREATE TABLE meta (
            cle    TEXT PRIMARY KEY,
            valeur INTEGER NOT NULL
          )
        ''');
      },
    );
    return ArmeRepository(db);
  }

  Future<List<Arme>> _appelReseau() async {
    await Future<void>.delayed(const Duration(milliseconds: 800));
    if (panne) throw Exception('Réseau indisponible');
    return const <Arme>[
      Arme(id: 'a-1', nom: 'Épée courte', degats: 12),
      Arme(id: 'a-2', nom: 'Hache de guerre', degats: 28),
      Arme(id: 'a-3', nom: 'Arc long', degats: 19),
      Arme(id: 'a-4', nom: 'Lame de l\'aube', degats: 44),
    ];
  }

  Future<void> _remplacerCache(List<Arme> armes) async {
    await _db.transaction((Transaction txn) async {
      await txn.delete('arme');
      final Batch lot = txn.batch();
      for (final Arme a in armes) {
        lot.insert('arme', a.toMap());
      }
      lot.insert(
        'meta',
        <String, Object?>{
          'cle': 'date_cache',
          'valeur': DateTime.now().millisecondsSinceEpoch,
        },
        conflictAlgorithm: ConflictAlgorithm.replace,
      );
      await lot.commit(noResult: true);
    });
  }

  Future<List<Arme>> _lireCache() async {
    final List<Map<String, Object?>> lignes =
        await _db.query('arme', orderBy: 'degats DESC');
    return lignes.map(Arme.fromMap).toList();
  }

  Future<DateTime?> _dateCache() async {
    final List<Map<String, Object?>> lignes = await _db.query(
      'meta',
      where: 'cle = ?',
      whereArgs: <Object?>['date_cache'],
      limit: 1,
    );
    if (lignes.isEmpty) return null;
    return DateTime.fromMillisecondsSinceEpoch(lignes.first['valeur']! as int);
  }

  Future<Resultat> charger() async {
    try {
      final List<Arme> frais = await _appelReseau();
      await _remplacerCache(frais);
      frais.sort((Arme a, Arme b) => b.degats.compareTo(a.degats));
      return Resultat(armes: frais, provenance: 'réseau');
    } on Object {
      final List<Arme> cache = await _lireCache();
      if (cache.isEmpty) rethrow;
      return Resultat(
        armes: cache,
        provenance: 'cache',
        dateCache: await _dateCache(),
      );
    }
  }
}

// ── lib/main.dart ──
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(ApplicationCatalogue(depot: await ArmeRepository.ouvrir()));
}

class ApplicationCatalogue extends StatelessWidget {
  const ApplicationCatalogue({super.key, required this.depot});

  final ArmeRepository depot;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Catalogue hors-ligne',
      theme: ThemeData(colorSchemeSeed: Colors.amber, useMaterial3: true),
      home: PageCatalogue(depot: depot),
    );
  }
}

class PageCatalogue extends StatefulWidget {
  const PageCatalogue({super.key, required this.depot});

  final ArmeRepository depot;

  @override
  State<PageCatalogue> createState() => _PageCatalogueState();
}

class _PageCatalogueState extends State<PageCatalogue> {
  late Future<Resultat> _futur = widget.depot.charger();

  void _recharger() {
    setState(() => _futur = widget.depot.charger());
  }

  String _age(DateTime? d) {
    if (d == null) return 'date inconnue';
    final Duration ecart = DateTime.now().difference(d);
    if (ecart.inMinutes < 1) return 'il y a quelques secondes';
    if (ecart.inHours < 1) return 'il y a ${ecart.inMinutes} min';
    return 'il y a ${ecart.inHours} h';
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Catalogue d\'armes'),
        actions: <Widget>[
          IconButton(
            tooltip: 'Simuler une panne réseau',
            icon: Icon(widget.depot.panne ? Icons.wifi_off : Icons.wifi),
            onPressed: () {
              widget.depot.panne = !widget.depot.panne;
              _recharger();
            },
          ),
        ],
      ),
      body: FutureBuilder<Resultat>(
        future: _futur,
        builder: (BuildContext context, AsyncSnapshot<Resultat> snap) {
          if (snap.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }
          if (snap.hasError) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: <Widget>[
                  const Icon(Icons.cloud_off, size: 64),
                  const SizedBox(height: 16),
                  const Text(
                    'Catalogue indisponible et cache vide.',
                    textAlign: TextAlign.center,
                  ),
                  const SizedBox(height: 16),
                  FilledButton(
                    onPressed: _recharger,
                    child: const Text('Réessayer'),
                  ),
                ],
              ),
            );
          }

          final Resultat r = snap.data!;
          return Column(
            children: <Widget>[
              if (r.estPerime)
                Material(
                  color: Theme.of(context).colorScheme.tertiaryContainer,
                  child: ListTile(
                    leading: const Icon(Icons.cloud_off),
                    title: const Text('Mode hors-ligne'),
                    subtitle: Text('Données ${_age(r.dateCache)}'),
                    trailing: TextButton(
                      onPressed: _recharger,
                      child: const Text('Réessayer'),
                    ),
                  ),
                ),
              Expanded(
                child: ListView.separated(
                  itemCount: r.armes.length,
                  separatorBuilder: (_, __) => const Divider(height: 1),
                  itemBuilder: (BuildContext context, int i) {
                    final Arme a = r.armes[i];
                    return ListTile(
                      leading: CircleAvatar(child: Text('${a.degats}')),
                      title: Text(a.nom),
                      subtitle: Text('Source : ${r.provenance}'),
                    );
                  },
                ),
              ),
            ],
          );
        },
      ),
    );
  }
}
```

**Explication :** le dépôt applique la stratégie « réseau d'abord » de la section 54.36, mais avec SQLite comme support de cache. `_remplacerCache` est le cœur de la correction : le `DELETE` puis les `INSERT` sont dans **une seule transaction**, donc le catalogue n'est jamais vu à moitié vide, même si l'application est tuée en plein remplacement. Le `batch` à l'intérieur de la transaction limite les allers-retours vers le code natif : on gagne l'atomicité **et** la vitesse. La date de fraîcheur est rangée dans une table `meta`, technique classique pour associer des métadonnées à un cache SQL. Le `rethrow` quand le cache est vide est essentiel : sans lui, l'utilisateur verrait une liste vide sans explication au tout premier lancement hors ligne. Enfin, le bandeau indique clairement la provenance et l'âge des données, et propose une action : les trois conditions de l'honnêteté définies en 54.37.

---

## Et maintenant ?

Vous venez de terminer la **PARTIE 1B**.

Reprenez un instant le chemin parcouru depuis le chapitre 43. Vous avez installé Flutter, compris l'arbre de widgets, apprivoisé `setState`, dompté les contraintes de mise en page, affiché du texte et des images, construit des listes, validé des formulaires, navigué entre des écrans, thémé une application entière, partagé un état entre plusieurs pages, appelé une API, et enfin — dans ce chapitre — fait survivre vos données à la fermeture de l'application.

Ce sont les quatre couches d'une application réelle : **interface, état, données distantes, données locales**. Vous les avez toutes.

Il vous manque une seule chose, et ce n'est pas une notion : c'est du **kilométrage**. Savoir ce qu'est un `ListView.builder` et savoir construire une application de bout en bout sont deux compétences différentes. La seconde ne s'acquiert qu'en écrivant des applications complètes, du cahier des charges à la dernière ligne, avec les hésitations, les impasses et les remaniements que cela suppose.

C'est exactement ce que propose la **PARTIE 1C**. Huit projets, chacun étant une application entière, présentée comme en entreprise : un cahier des charges, une découpe en étapes numérotées, le code de chaque fichier, les pièges rencontrés, et des extensions à réaliser seul.

| # | Projet | Ce que vous y consoliderez |
| --- | --- | --- |
| 55 | Carte de profil | Widgets, layouts, thème, assets |
| 56 | Calculatrice | État, grille de boutons, gestion d'erreurs |
| 57 | Convertisseur d'unités | Formulaires, listes déroulantes, validation |
| 58 | Liste de tâches | Listes, `Dismissible`, **persistance du chapitre 54** |
| 59 | Quiz | Modèles, navigation, score |
| 60 | Catalogue et panier | `GridView`, détail, `provider` |
| 61 | Application météo | API REST, JSON, `FutureBuilder`, **cache du chapitre 54** |
| 62 | Lanceur de jeu | Synthèse, et passerelle vers la PARTIE 2 |

Le premier est volontairement modeste : une carte de profil. L'objectif n'est pas la difficulté, mais la **méthode** — partir d'une maquette, la découper, la construire de l'extérieur vers l'intérieur, et obtenir un résultat propre du premier coup.

Rendez-vous au chapitre 55 : [55-PARTIE-1C—PROJET-1-CARTE-DE-PROFIL.md](./55-PARTIE-1C—PROJET-1-CARTE-DE-PROFIL.md)
