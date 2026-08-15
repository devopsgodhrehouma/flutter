# PARTIE 1B — FLUTTER
# CHAPITRE 49 — BOUTONS, FORMULAIRES ET VALIDATION

> **Niveau :** intermédiaire
> **Durée estimée :** 10 h
> **Pré-requis :** chapitres 43 à 48 (installation, widgets, `setState`, layouts, texte, listes) ; chapitres 07, 12 et 13 de la PARTIE 1A (fonctions, null safety, exceptions et `int.tryParse`)
> **Ce que vous saurez faire à la fin :** construire un formulaire complet, validé champ par champ, avec clavier adapté, messages d'erreur, retours utilisateur et soumission propre.

---

## 49.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer ce qui rend une interface Flutter « interactive » ;
- choisir le bon bouton Material 3 parmi les cinq principaux ;
- utiliser `ElevatedButton`, `FilledButton`, `OutlinedButton`, `TextButton` et leurs variantes `.icon` ;
- utiliser `IconButton` et `FloatingActionButton` ;
- désactiver un bouton avec `onPressed: null` et expliquer pourquoi ce n'est pas une astuce mais l'API officielle ;
- réagir à un appui long avec `onLongPress` ;
- styliser un bouton avec `styleFrom` puis avec un `ButtonStyle` complet et `WidgetStateProperty` ;
- rendre n'importe quel widget cliquable avec `GestureDetector` ;
- obtenir l'effet d'ondulation Material avec `InkWell` et comprendre pourquoi il faut un `Material` au-dessus ;
- afficher un champ de saisie avec `TextField` ;
- habiller ce champ avec `InputDecoration` : label, hint, icônes, bordures, compteur ;
- lire et écrire le contenu d'un champ avec un `TextEditingController` ;
- libérer ce contrôleur dans `dispose()` et expliquer la fuite mémoire évitée ;
- réagir à chaque frappe avec `onChanged` ;
- réagir à la validation clavier avec `onSubmitted` ;
- choisir le bon clavier avec `keyboardType` ;
- masquer un mot de passe avec `obscureText` et ajouter un œil de visibilité ;
- limiter la saisie avec `maxLength` et `inputFormatters` ;
- gérer le focus avec `FocusNode` et passer d'un champ au suivant ;
- fermer le clavier de trois façons différentes ;
- regrouper des champs dans un `Form` piloté par une `GlobalKey<FormState>` ;
- remplacer `TextField` par `TextFormField` et savoir pourquoi ;
- écrire un `validator` qui retourne `null` ou un message ;
- déclencher la validation avec `formKey.currentState!.validate()` ;
- choisir un `autovalidateMode` adapté à l'expérience utilisateur ;
- collecter les valeurs avec `onSaved` et `save()` ;
- écrire une bibliothèque de validateurs réutilisables ;
- valider un nombre avec `int.tryParse` sans jamais lever d'exception ;
- utiliser `Checkbox`, `CheckboxListTile`, `Switch`, `Radio` + `RadioGroup`, `RadioListTile` et `Slider` ;
- utiliser `DropdownButton` et `DropdownButtonFormField` ;
- ouvrir un sélecteur de date avec `showDatePicker` et d'heure avec `showTimePicker` ;
- afficher un `SnackBar` avec le bon `context` ;
- ouvrir une boîte de dialogue avec `showDialog` et `AlertDialog` ;
- ouvrir une feuille modale avec `showModalBottomSheet` ;
- assembler un formulaire de création de personnage complet, validé et soumis.

---

## 49.0.1 — Ce que le chapitre suppose déjà connu

Ce chapitre s'appuie directement sur des notions déjà vues. Un rappel rapide vous évitera de bloquer.

```text
PARTIE 1A
  chapitre 07 : fonctions, fonctions anonymes, paramètres nommés
  chapitre 12 : null safety, String?, !, ??, ?.
  chapitre 13 : int.tryParse, gestion des erreurs sans exception
  chapitre 14 : where, map, every, any

PARTIE 1B
  chapitre 44 : widget, arbre de widgets, BuildContext
  chapitre 45 : StatefulWidget, State, setState, initState, dispose
  chapitre 46 : Column, Row, Padding, SizedBox, Expanded
  chapitre 47 : Text, TextStyle, Icon
  chapitre 48 : ListView, ListTile, SnackBar (aperçu)
```

Le point le plus important est le chapitre 45. Tout ce chapitre-ci repose sur une idée unique :

> **La saisie de l'utilisateur est un état. Un état vit dans un `State`. Il se modifie avec `setState` et se libère dans `dispose`.**

Si cette phrase ne vous parle pas encore, relisez la section 45.6 avant de continuer.

---

## 49.1 — Rendre une interface interactive

Jusqu'ici, vos écrans étaient **muets**. Ils affichaient des données. L'utilisateur pouvait regarder, faire défiler, et c'est tout.

Une interface interactive ajoute deux directions de circulation :

```text
        INTERFACE MUETTE (chapitres 44 à 48)

        Données  ────────────────>  Écran
                    build()


        INTERFACE INTERACTIVE (ce chapitre)

        Données  ────────────────>  Écran
           ^        build()            │
           │                           │
           └───────────────────────────┘
                 callback + setState
```

Concrètement, Flutter propose deux familles de widgets pour la flèche du retour :

| Famille | Rôle | Exemples |
| --- | --- | --- |
| Les **actions** | l'utilisateur déclenche quelque chose | `ElevatedButton`, `IconButton`, `GestureDetector` |
| Les **saisies** | l'utilisateur fournit une valeur | `TextField`, `Checkbox`, `Slider`, `DropdownButton` |

Toutes ces widgets fonctionnent sur le **même principe** : vous leur passez une **fonction de rappel** (un *callback*), et Flutter l'appelle quand l'événement se produit.

Voici le plus petit programme interactif possible.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Interactif',
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

  void _ajouterDixPoints() {
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
              '$_score points',
              style: const TextStyle(fontSize: 40, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: _ajouterDixPoints,
              child: const Text('Gagner 10 points'),
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
┌──────────────────────────────────┐
│  Score du joueur                 │
├──────────────────────────────────┤
│                                  │
│                                  │
│            0 points              │
│                                  │
│      ┌────────────────────┐      │
│      │ Gagner 10 points   │      │
│      └────────────────────┘      │
│                                  │
└──────────────────────────────────┘

Après trois appuis : « 30 points »
```

Trois choses à remarquer, et elles reviendront dans **tout** le chapitre :

1. `onPressed:` reçoit **le nom de la fonction**, sans parenthèses. `onPressed: _ajouterDixPoints` et non `onPressed: _ajouterDixPoints()`. Avec les parenthèses, vous appelleriez la fonction pendant le `build`, ce qui est une erreur classique décrite plus bas.
2. La fonction appelle `setState`. Sans `setState`, `_score` changerait en mémoire mais l'écran ne se redessinerait pas.
3. La page est un `StatefulWidget`. Une interface interactive est presque toujours un `StatefulWidget` (ou un widget piloté par une solution d'état, sujet du chapitre 52).

---

## 49.1.1 — La forme d'un callback

Un callback est une fonction que **vous écrivez** et que **Flutter appelle**. Il y a trois écritures possibles, strictement équivalentes.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageCallbacks(),
    );
  }
}

class PageCallbacks extends StatefulWidget {
  const PageCallbacks({super.key});

  @override
  State<PageCallbacks> createState() => _PageCallbacksState();
}

class _PageCallbacksState extends State<PageCallbacks> {
  String _dernierAppui = 'aucun';

  void _noter(String origine) {
    setState(() {
      _dernierAppui = origine;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Trois écritures')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text('Dernier appui : $_dernierAppui'),
            const SizedBox(height: 24),

            // 1. Référence à une méthode existante.
            ElevatedButton(
              onPressed: _appuiUn,
              child: const Text('Écriture 1 : référence'),
            ),
            const SizedBox(height: 12),

            // 2. Fonction anonyme avec corps.
            ElevatedButton(
              onPressed: () {
                _noter('bouton 2');
              },
              child: const Text('Écriture 2 : fonction anonyme'),
            ),
            const SizedBox(height: 12),

            // 3. Fonction fléchée, quand le corps tient en une expression.
            ElevatedButton(
              onPressed: () => _noter('bouton 3'),
              child: const Text('Écriture 3 : flèche'),
            ),
          ],
        ),
      ),
    );
  }

  void _appuiUn() => _noter('bouton 1');
}
```

**Résultat :**

```text
Dernier appui : aucun
   -> appui sur le bouton 2 ->
Dernier appui : bouton 2
```

**Laquelle choisir ?**

| Situation | Écriture recommandée |
| --- | --- |
| La logique tient en une ligne et n'est utilisée qu'ici | flèche `() => ...` |
| La logique fait plusieurs lignes | fonction anonyme `() { ... }` |
| La logique est réutilisée ailleurs, ou fait plus de dix lignes | méthode nommée `_maMethode` |

L'erreur à ne jamais commettre :

```dart
// FAUX : la fonction est exécutée immédiatement, pendant le build.
ElevatedButton(
  onPressed: _ajouterDixPoints(),   // <-- parenthèses fatales
  child: const Text('Gagner'),
)
```

Le compilateur refuse ce code parce que `_ajouterDixPoints()` retourne `void`, et `void` n'est pas une `VoidCallback`. Mais si la méthode retournait quelque chose, le code compilerait et provoquerait un `setState` pendant le `build`, donc une exception `setState() or markNeedsBuild() called during build`.

---

## 49.2 — `ElevatedButton`

`ElevatedButton` est un bouton **surélevé** : il a un fond coloré et une légère ombre. Dans Material 3, il sert aux actions importantes mais pas primaires.

Ses deux paramètres essentiels sont :

| Paramètre | Type | Rôle |
| --- | --- | --- |
| `onPressed` | `VoidCallback?` | ce qui se passe à l'appui ; `null` désactive le bouton |
| `child` | `Widget` | le contenu, presque toujours un `Text` |

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageElevated(),
    );
  }
}

class PageElevated extends StatefulWidget {
  const PageElevated({super.key});

  @override
  State<PageElevated> createState() => _PageElevatedState();
}

class _PageElevatedState extends State<PageElevated> {
  int _potions = 3;
  int _energie = 40;

  void _boirePotion() {
    if (_potions == 0) {
      return;
    }
    setState(() {
      _potions -= 1;
      _energie += 25;
      if (_energie > 100) {
        _energie = 100;
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ElevatedButton')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text('Énergie : $_energie / 100',
                style: const TextStyle(fontSize: 22)),
            Text('Potions restantes : $_potions'),
            const SizedBox(height: 24),

            // Forme simple.
            ElevatedButton(
              onPressed: _boirePotion,
              child: const Text('Boire une potion'),
            ),
            const SizedBox(height: 16),

            // Variante avec icône.
            ElevatedButton.icon(
              onPressed: _boirePotion,
              icon: const Icon(Icons.local_drink),
              label: const Text('Boire une potion'),
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
Énergie : 40 / 100
Potions restantes : 3

  ┌──────────────────────┐
  │  Boire une potion    │      <- fond coloré + ombre
  └──────────────────────┘

  ┌──────────────────────────┐
  │  ▪  Boire une potion    │   <- variante .icon
  └──────────────────────────┘

Après un appui : Énergie 65 / 100, Potions restantes : 2
```

**Attention à la confusion la plus fréquente.** `ElevatedButton` a un paramètre `child`, mais `ElevatedButton.icon` n'en a **pas** : il a `icon` **et** `label`. Écrire `ElevatedButton.icon(icon: ..., child: ...)` ne compile pas.

---

## 49.3 — `FilledButton`

`FilledButton` est le bouton **primaire** de Material 3. C'est celui qui porte l'action principale d'un écran : « Valider », « Créer », « Payer ». Il est plein, sans ombre, et plus visible que `ElevatedButton`.

Il possède quatre constructeurs :

| Constructeur | Aspect |
| --- | --- |
| `FilledButton(...)` | plein, couleur primaire |
| `FilledButton.icon(...)` | idem avec une icône |
| `FilledButton.tonal(...)` | plein, couleur secondaire plus douce |
| `FilledButton.tonalIcon(...)` | idem avec une icône |

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageFilled(),
    );
  }
}

class PageFilled extends StatefulWidget {
  const PageFilled({super.key});

  @override
  State<PageFilled> createState() => _PageFilledState();
}

class _PageFilledState extends State<PageFilled> {
  String _journal = 'Aucune action.';

  void _agir(String action) {
    setState(() {
      _journal = action;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('FilledButton')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            Text(_journal, textAlign: TextAlign.center),
            const SizedBox(height: 32),

            FilledButton(
              onPressed: () => _agir('Partie lancée.'),
              child: const Text('Lancer la partie'),
            ),
            const SizedBox(height: 12),

            FilledButton.icon(
              onPressed: () => _agir('Partie sauvegardée.'),
              icon: const Icon(Icons.save),
              label: const Text('Sauvegarder'),
            ),
            const SizedBox(height: 12),

            FilledButton.tonal(
              onPressed: () => _agir('Options ouvertes.'),
              child: const Text('Options'),
            ),
            const SizedBox(height: 12),

            FilledButton.tonalIcon(
              onPressed: () => _agir('Crédits affichés.'),
              icon: const Icon(Icons.info_outline),
              label: const Text('Crédits'),
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
                Aucune action.

  ████████████ Lancer la partie ████████████    <- primaire, plein
  ████████████ ▪ Sauvegarder  ████████████
  ▒▒▒▒▒▒▒▒▒▒▒▒    Options      ▒▒▒▒▒▒▒▒▒▒▒▒    <- tonal, plus doux
  ▒▒▒▒▒▒▒▒▒▒▒▒  i Crédits      ▒▒▒▒▒▒▒▒▒▒▒▒
```

**Règle de hiérarchie Material 3.** Sur un écran donné, il ne devrait y avoir **qu'un seul** `FilledButton` : l'action principale. Le reste est en `FilledButton.tonal`, `OutlinedButton` ou `TextButton`.

Le `CrossAxisAlignment.stretch` de la `Column` fait que les boutons occupent toute la largeur. C'est le réglage habituel pour un formulaire : voir la section 46.9.

---

## 49.4 — `OutlinedButton`

`OutlinedButton` a un contour et un fond transparent. Il porte les actions **secondaires** qui restent importantes : « Annuler », « Retour », « Voir plus ».

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageOutlined(),
    );
  }
}

class PageOutlined extends StatefulWidget {
  const PageOutlined({super.key});

  @override
  State<PageOutlined> createState() => _PageOutlinedState();
}

class _PageOutlinedState extends State<PageOutlined> {
  String _choix = 'en attente';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('OutlinedButton')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text('Quitter la partie ? ($_choix)'),
            const SizedBox(height: 24),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: <Widget>[
                OutlinedButton(
                  onPressed: () => setState(() => _choix = 'annulé'),
                  child: const Text('Annuler'),
                ),
                const SizedBox(width: 16),
                FilledButton(
                  onPressed: () => setState(() => _choix = 'confirmé'),
                  child: const Text('Quitter'),
                ),
              ],
            ),
            const SizedBox(height: 24),
            OutlinedButton.icon(
              onPressed: () => setState(() => _choix = 'aide'),
              icon: const Icon(Icons.help_outline),
              label: const Text('Aide'),
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
        Quitter la partie ? (en attente)

    ┌───────────┐   ████████████████
    │  Annuler  │   ███  Quitter  ██     <- l'action destructrice
    └───────────┘   ████████████████        est le bouton plein
             ┌────────────┐
             │  ? Aide    │
             └────────────┘
```

**Convention d'ordre.** Sur Android et dans Material Design, l'action de confirmation est **à droite**. Sur iOS c'est souvent l'inverse. Si votre application vise les deux plateformes, respectez Material : Flutter vous donne Material par défaut.

---

## 49.5 — `TextButton`

`TextButton` n'a ni fond ni contour : c'est du texte cliquable. Il porte les actions **les moins importantes**, ou celles qui vivent dans une boîte de dialogue, une `AppBar`, une `Card`.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageTextButton(),
    );
  }
}

class PageTextButton extends StatefulWidget {
  const PageTextButton({super.key});

  @override
  State<PageTextButton> createState() => _PageTextButtonState();
}

class _PageTextButtonState extends State<PageTextButton> {
  bool _detailsVisibles = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('TextButton'),
        actions: <Widget>[
          TextButton(
            onPressed: () {},
            child: const Text('Passer'),
          ),
        ],
      ),
      body: Center(
        child: Card(
          margin: const EdgeInsets.all(24),
          child: Padding(
            padding: const EdgeInsets.all(16),
            child: Column(
              mainAxisSize: MainAxisSize.min,
              crossAxisAlignment: CrossAxisAlignment.start,
              children: <Widget>[
                const Text(
                  'Gobelin des cavernes',
                  style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
                ),
                const Text('Niveau 3 — 45 points de vie'),
                if (_detailsVisibles) ...<Widget>[
                  const SizedBox(height: 8),
                  const Text(
                    'Faiblesse : feu. Résistance : poison. '
                    'Attaque de base : 12 dégâts.',
                  ),
                ],
                const SizedBox(height: 8),
                Row(
                  mainAxisAlignment: MainAxisAlignment.end,
                  children: <Widget>[
                    TextButton(
                      onPressed: () {
                        setState(() {
                          _detailsVisibles = !_detailsVisibles;
                        });
                      },
                      child: Text(_detailsVisibles ? 'Masquer' : 'Détails'),
                    ),
                    TextButton.icon(
                      onPressed: () {},
                      icon: const Icon(Icons.sports_martial_arts),
                      label: const Text('Attaquer'),
                    ),
                  ],
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────┐
│ TextButton                   Passer  │
├──────────────────────────────────────┤
│   ┌──────────────────────────────┐   │
│   │ Gobelin des cavernes         │   │
│   │ Niveau 3 — 45 points de vie  │   │
│   │                              │   │
│   │            Détails  ✕Attaquer│   │
│   └──────────────────────────────┘   │
└──────────────────────────────────────┘

Après appui sur « Détails » :
   la carte s'agrandit, le texte de faiblesse apparaît,
   et le bouton devient « Masquer ».
```

Remarquez la syntaxe `if (_detailsVisibles) ...<Widget>[ ... ]` : c'est le `if` de collection du chapitre 06, combiné à l'opérateur *spread* `...`. C'est la manière idiomatique d'insérer conditionnellement plusieurs widgets dans une liste.

---

## 49.5.1 — Choisir son bouton : le tableau de décision

```text
     Importance de l'action
            ^
            │
    HAUTE   │   FilledButton          « Créer le personnage »
            │
    MOYENNE │   FilledButton.tonal    « Sauvegarder »
            │   ElevatedButton        « Boire une potion »
            │
    BASSE   │   OutlinedButton        « Annuler »
            │
    TRÈS    │   TextButton            « Détails », « Passer »
    BASSE   │
            └──────────────────────────────────────────>
```

| Bouton | Fond | Contour | Ombre | Usage |
| --- | --- | --- | --- | --- |
| `FilledButton` | plein primaire | non | non | action principale, une seule par écran |
| `FilledButton.tonal` | plein secondaire | non | non | action importante mais pas principale |
| `ElevatedButton` | plein | non | oui | action à détacher d'un fond chargé |
| `OutlinedButton` | transparent | oui | non | action secondaire |
| `TextButton` | transparent | non | non | action tertiaire, dialogues, `AppBar` |

---

## 49.6 — `IconButton`

`IconButton` est un bouton **rond** qui ne contient qu'une icône. On le trouve dans les `AppBar`, les `ListTile`, les barres d'outils.

Ses paramètres importants :

| Paramètre | Rôle |
| --- | --- |
| `icon` | le widget icône, généralement `Icon(Icons.xxx)` |
| `onPressed` | l'action ; `null` désactive |
| `tooltip` | l'info-bulle affichée à l'appui long ; **fortement recommandée** |
| `iconSize` | la taille en pixels logiques |
| `color` | la couleur de l'icône |

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageIconButton(),
    );
  }
}

class PageIconButton extends StatefulWidget {
  const PageIconButton({super.key});

  @override
  State<PageIconButton> createState() => _PageIconButtonState();
}

class _PageIconButtonState extends State<PageIconButton> {
  int _vies = 3;
  bool _favori = false;
  bool _sonActif = true;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('IconButton'),
        actions: <Widget>[
          IconButton(
            icon: Icon(_sonActif ? Icons.volume_up : Icons.volume_off),
            tooltip: _sonActif ? 'Couper le son' : 'Activer le son',
            onPressed: () => setState(() => _sonActif = !_sonActif),
          ),
        ],
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text('Vies : $_vies', style: const TextStyle(fontSize: 24)),
            const SizedBox(height: 16),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: <Widget>[
                IconButton(
                  icon: const Icon(Icons.remove_circle_outline),
                  tooltip: 'Perdre une vie',
                  iconSize: 40,
                  color: Colors.red,
                  onPressed: _vies > 0
                      ? () => setState(() => _vies -= 1)
                      : null,
                ),
                const SizedBox(width: 24),
                IconButton(
                  icon: const Icon(Icons.add_circle_outline),
                  tooltip: 'Gagner une vie',
                  iconSize: 40,
                  color: Colors.green,
                  onPressed: _vies < 5
                      ? () => setState(() => _vies += 1)
                      : null,
                ),
              ],
            ),
            const SizedBox(height: 24),

            // Variante « pleine », Material 3.
            IconButton.filled(
              icon: Icon(_favori ? Icons.star : Icons.star_border),
              tooltip: 'Marquer comme favori',
              onPressed: () => setState(() => _favori = !_favori),
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
┌────────────────────────────────────┐
│ IconButton                    ♪   │
├────────────────────────────────────┤
│                                    │
│              Vies : 3              │
│                                    │
│         ⊖              ⊕           │
│       (rouge)       (vert)         │
│                                    │
│              (★)                   │
│                                    │
└────────────────────────────────────┘

Quand _vies vaut 0, le ⊖ devient gris et ne réagit plus.
Quand _vies vaut 5, le ⊕ devient gris et ne réagit plus.
```

Deux points essentiels :

1. **`tooltip` n'est pas décoratif.** Une icône seule est ambiguë. Le `tooltip` sert à l'accessibilité : les lecteurs d'écran l'énoncent.
2. `IconButton` existe aussi en variantes Material 3 : `IconButton.filled`, `IconButton.filledTonal`, `IconButton.outlined`.

---

## 49.7 — `FloatingActionButton`

Le `FloatingActionButton` (FAB) est le bouton rond qui flotte en bas à droite. Il porte **l'action la plus importante de l'écran entier**, celle que l'utilisateur fera le plus souvent : ajouter, créer, composer.

Il ne se met pas dans le `body`. Il se met dans le paramètre `floatingActionButton:` du `Scaffold`.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageFab(),
    );
  }
}

class PageFab extends StatefulWidget {
  const PageFab({super.key});

  @override
  State<PageFab> createState() => _PageFabState();
}

class _PageFabState extends State<PageFab> {
  final List<String> _inventaire = <String>['Épée courte', 'Bouclier'];
  int _compteur = 0;

  void _ajouterObjet() {
    _compteur += 1;
    setState(() {
      _inventaire.add('Potion n°$_compteur');
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Inventaire')),
      body: ListView.builder(
        itemCount: _inventaire.length,
        itemBuilder: (BuildContext context, int index) {
          return ListTile(
            leading: const Icon(Icons.inventory_2),
            title: Text(_inventaire[index]),
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _ajouterObjet,
        tooltip: 'Ajouter un objet',
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────┐
│  Inventaire                      │
├──────────────────────────────────┤
│  ▣  Épée courte                 │
│  ▣  Bouclier                    │
│  ▣  Potion n°1                  │
│                                  │
│                          ╭─────╮ │
│                          │  +  │ │
│                          ╰─────╯ │
└──────────────────────────────────┘
```

**Les trois formes de FAB :**

```dart
// Petit, pour une action secondaire.
FloatingActionButton.small(
  onPressed: () {},
  child: const Icon(Icons.add),
)

// Grand, pour un écran où l'action est vraiment centrale.
FloatingActionButton.large(
  onPressed: () {},
  child: const Icon(Icons.add),
)

// Étendu, avec un texte : le plus clair pour l'utilisateur.
FloatingActionButton.extended(
  onPressed: () {},
  icon: const Icon(Icons.add),
  label: const Text('Nouvel objet'),
)
```

**Position.** Le paramètre `floatingActionButtonLocation:` du `Scaffold` change l'emplacement : `FloatingActionButtonLocation.centerFloat`, `.endDocked`, `.centerDocked`, etc.

**Règle.** Un seul FAB par écran. Si vous en voulez deux, c'est que l'un des deux n'est pas l'action principale : mettez-le dans l'`AppBar`.

---

## 49.8 — `onPressed: null` : le bouton désactivé

Voici l'un des points d'API les plus élégants de Flutter, et pourtant l'un des plus mal compris.

> **Pour désactiver un bouton, on passe `null` à `onPressed`.**

Il n'existe **pas** de paramètre `enabled:` sur les boutons Material. Le type de `onPressed` est `VoidCallback?` — le `?` du chapitre 12. Quand la valeur est `null`, Flutter :

- grise le bouton automatiquement (couleurs `disabledForegroundColor` et `disabledBackgroundColor`) ;
- supprime l'effet d'ondulation ;
- ignore les appuis ;
- signale l'état « désactivé » aux lecteurs d'écran.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageDesactive(),
    );
  }
}

class PageDesactive extends StatefulWidget {
  const PageDesactive({super.key});

  @override
  State<PageDesactive> createState() => _PageDesactiveState();
}

class _PageDesactiveState extends State<PageDesactive> {
  bool _conditionsAcceptees = false;
  bool _envoiEnCours = false;

  Future<void> _creerLePersonnage() async {
    setState(() => _envoiEnCours = true);
    // Simulation d'un appel réseau (asynchrone, chapitre 15).
    await Future<void>.delayed(const Duration(seconds: 2));
    if (!mounted) {
      return;
    }
    setState(() => _envoiEnCours = false);
  }

  @override
  Widget build(BuildContext context) {
    // Le bouton n'est actif que si les conditions sont acceptées
    // ET qu'aucun envoi n'est en cours.
    final bool actif = _conditionsAcceptees && !_envoiEnCours;

    return Scaffold(
      appBar: AppBar(title: const Text('Bouton désactivé')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            CheckboxListTile(
              value: _conditionsAcceptees,
              title: const Text("J'accepte les règles du jeu"),
              onChanged: (bool? valeur) {
                setState(() => _conditionsAcceptees = valeur ?? false);
              },
            ),
            const SizedBox(height: 24),
            SizedBox(
              width: double.infinity,
              child: FilledButton(
                // Le cœur de la section : null ou une fonction.
                onPressed: actif ? _creerLePersonnage : null,
                child: _envoiEnCours
                    ? const SizedBox(
                        width: 20,
                        height: 20,
                        child: CircularProgressIndicator(strokeWidth: 2),
                      )
                    : const Text('Créer le personnage'),
              ),
            ),
            const SizedBox(height: 16),
            Text(
              actif
                  ? 'Le bouton est actif.'
                  : 'Le bouton est désactivé (onPressed vaut null).',
              style: const TextStyle(fontStyle: FontStyle.italic),
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
Case décochée :
  [ ] J'accepte les règles du jeu
  ▒▒▒▒▒▒ Créer le personnage ▒▒▒▒▒▒   <- gris, insensible
  Le bouton est désactivé (onPressed vaut null).

Case cochée :
  [x] J'accepte les règles du jeu
  ██████ Créer le personnage ██████   <- coloré, actif
  Le bouton est actif.

Pendant l'envoi (2 secondes) :
  ▒▒▒▒▒▒▒▒▒▒  ◌  ▒▒▒▒▒▒▒▒▒▒          <- désactivé + indicateur
```

**Le piège à éviter absolument :**

```dart
// FAUX : une fonction vide n'est PAS un bouton désactivé.
ElevatedButton(
  onPressed: () {},          // le bouton reste coloré et cliquable
  child: const Text('Créer'),
)
```

Visuellement, l'utilisateur croit pouvoir appuyer. Il appuie. Rien ne se passe. Il appuie encore. C'est la pire expérience possible. Utilisez `null`.

**Le double-envoi.** Le drapeau `_envoiEnCours` a un rôle précis : empêcher l'utilisateur d'appuyer deux fois pendant un appel réseau. Sans ce drapeau, deux personnages seraient créés. C'est un réflexe à prendre dès maintenant.

Notez aussi `if (!mounted) return;` après le `await` : sans lui, vous risquez un `setState() called after dispose()` si l'utilisateur quitte l'écran pendant les deux secondes. Ce point a été expliqué en 45.15.

---

## 49.9 — `onLongPress`

Tous les boutons Material acceptent un second callback : `onLongPress`, déclenché après environ 500 millisecondes d'appui maintenu.

C'est utile pour une action « avancée » sur le même bouton : incrémenter de 10 au lieu de 1, ouvrir un menu, afficher une aide.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageLongPress(),
    );
  }
}

class PageLongPress extends StatefulWidget {
  const PageLongPress({super.key});

  @override
  State<PageLongPress> createState() => _PageLongPressState();
}

class _PageLongPressState extends State<PageLongPress> {
  int _or = 0;
  String _dernier = '—';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('onLongPress')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text('Or : $_or', style: const TextStyle(fontSize: 32)),
            Text('Dernière action : $_dernier'),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: () {
                setState(() {
                  _or += 1;
                  _dernier = 'appui court (+1)';
                });
              },
              onLongPress: () {
                setState(() {
                  _or += 10;
                  _dernier = 'appui long (+10)';
                });
              },
              child: const Text('Miner de l\'or'),
            ),
            const SizedBox(height: 12),
            const Text(
              'Appui court : +1 or\nAppui long : +10 or',
              textAlign: TextAlign.center,
              style: TextStyle(fontSize: 12, color: Colors.grey),
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
Or : 0
Dernière action : —

   -> appui court ->  Or : 1   (appui court (+1))
   -> appui long  ->  Or : 11  (appui long (+10))
```

**Trois remarques importantes.**

1. `onLongPress` est **invisible**. L'utilisateur ne devine jamais qu'un appui long existe. N'y mettez jamais une action indispensable : c'est toujours un **raccourci** pour une action accessible autrement.
2. Un appui long ne déclenche **pas** `onPressed`. Les deux sont exclusifs.
3. Si `onPressed` vaut `null` **et** `onLongPress` vaut `null`, le bouton est désactivé. Si l'un des deux est non nul, le bouton reste actif.

---

## 49.10 — Styliser un bouton (`ButtonStyle`, `styleFrom`)

Un bouton se style par son paramètre `style:`, qui attend un objet `ButtonStyle`. Il y a deux façons d'en fabriquer un.

### Méthode 1 — `styleFrom` (à utiliser dans 95 % des cas)

Chaque classe de bouton possède une méthode statique `styleFrom` qui prend des valeurs simples et fabrique le `ButtonStyle` pour vous.

```dart
ElevatedButton.styleFrom(
  backgroundColor: Colors.deepOrange,
  foregroundColor: Colors.white,
  padding: const EdgeInsets.symmetric(horizontal: 32, vertical: 18),
  shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(30)),
  textStyle: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
  elevation: 6,
)
```

Les paramètres les plus utiles :

| Paramètre | Effet |
| --- | --- |
| `backgroundColor` | couleur de fond |
| `foregroundColor` | couleur du texte et de l'icône |
| `disabledBackgroundColor` | fond quand `onPressed` vaut `null` |
| `disabledForegroundColor` | texte quand `onPressed` vaut `null` |
| `padding` | marge intérieure |
| `shape` | forme et arrondi |
| `side` | bordure (`BorderSide`) |
| `textStyle` | style de police |
| `elevation` | hauteur de l'ombre |
| `minimumSize` | taille minimale, ex. `Size(double.infinity, 52)` |
| `shadowColor` | couleur de l'ombre |

### Méthode 2 — `ButtonStyle` avec `WidgetStateProperty`

Quand la couleur doit **dépendre de l'état** du bouton (survolé, pressé, désactivé), `styleFrom` ne suffit plus. Il faut un `ButtonStyle` dont chaque propriété est une `WidgetStateProperty`, c'est-à-dire une fonction « état → valeur ».

Les états sont dans l'énumération `WidgetState` : `hovered`, `focused`, `pressed`, `disabled`, `selected`, `dragged`, `error`.

Voici les deux méthodes côte à côte.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageStyle(),
    );
  }
}

class PageStyle extends StatefulWidget {
  const PageStyle({super.key});

  @override
  State<PageStyle> createState() => _PageStyleState();
}

class _PageStyleState extends State<PageStyle> {
  bool _actif = true;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Styliser un bouton')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            // 1. Bouton par défaut, aucun style.
            ElevatedButton(
              onPressed: _actif ? () {} : null,
              child: const Text('Par défaut'),
            ),
            const SizedBox(height: 20),

            // 2. Style via styleFrom.
            ElevatedButton(
              onPressed: _actif ? () {} : null,
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.deepOrange,
                foregroundColor: Colors.white,
                disabledBackgroundColor: Colors.grey.shade300,
                disabledForegroundColor: Colors.grey.shade600,
                padding: const EdgeInsets.symmetric(
                  horizontal: 32,
                  vertical: 18,
                ),
                shape: RoundedRectangleBorder(
                  borderRadius: BorderRadius.circular(30),
                ),
                textStyle: const TextStyle(
                  fontSize: 18,
                  fontWeight: FontWeight.bold,
                ),
                elevation: 6,
              ),
              child: const Text('styleFrom'),
            ),
            const SizedBox(height: 20),

            // 3. Style via ButtonStyle + WidgetStateProperty.
            ElevatedButton(
              onPressed: _actif ? () {} : null,
              style: ButtonStyle(
                backgroundColor: WidgetStateProperty.resolveWith<Color>(
                  (Set<WidgetState> etats) {
                    if (etats.contains(WidgetState.disabled)) {
                      return Colors.grey.shade300;
                    }
                    if (etats.contains(WidgetState.pressed)) {
                      return Colors.green.shade900;
                    }
                    if (etats.contains(WidgetState.hovered)) {
                      return Colors.green.shade600;
                    }
                    return Colors.green;
                  },
                ),
                foregroundColor: const WidgetStatePropertyAll<Color>(
                  Colors.white,
                ),
                padding: const WidgetStatePropertyAll<EdgeInsetsGeometry>(
                  EdgeInsets.symmetric(horizontal: 28, vertical: 16),
                ),
              ),
              child: const Text('ButtonStyle'),
            ),
            const SizedBox(height: 32),

            SwitchListTile(
              value: _actif,
              title: const Text('Boutons actifs'),
              onChanged: (bool valeur) => setState(() => _actif = valeur),
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
Interrupteur sur ON :
      ┌──────────────┐
      │  Par défaut  │             couleurs du thème
      └──────────────┘
   ╭────────────────────╮
   │     styleFrom      │          orange, arrondi 30, gras
   ╰────────────────────╯
   ┌────────────────────┐
   │    ButtonStyle     │          vert ; plus foncé si pressé
   └────────────────────┘

Interrupteur sur OFF : les trois deviennent gris.
```

### Styliser tous les boutons d'un coup

Répéter un `style:` sur chaque bouton est une mauvaise pratique. Le bon endroit est le **thème**, dans `ThemeData`. Le chapitre 51 y est consacré, mais voici le principe :

```dart
ThemeData(
  colorSchemeSeed: Colors.indigo,
  useMaterial3: true,
  filledButtonTheme: FilledButtonThemeData(
    style: FilledButton.styleFrom(
      minimumSize: const Size(double.infinity, 52),
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(12),
      ),
    ),
  ),
)
```

Tous les `FilledButton` de l'application prennent alors cette allure sans une seule ligne supplémentaire.

---

## 49.11 — `GestureDetector`

Les boutons sont pratiques, mais parfois vous voulez rendre cliquable **autre chose** : une carte, une image, un `Container` coloré, une zone de l'écran.

`GestureDetector` est un widget invisible qui enveloppe un enfant et écoute les gestes. Il ne dessine **rien** : ni fond, ni ondulation, ni changement visuel.

Ses callbacks les plus utiles :

| Callback | Geste |
| --- | --- |
| `onTap` | appui simple |
| `onDoubleTap` | double appui |
| `onLongPress` | appui maintenu |
| `onTapDown` | le doigt se pose |
| `onTapUp` | le doigt se lève |
| `onTapCancel` | le doigt glisse hors de la zone |
| `onPanUpdate` | le doigt se déplace en glissant |
| `onHorizontalDragEnd` | fin d'un glissement horizontal |
| `onVerticalDragEnd` | fin d'un glissement vertical |

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageGeste(),
    );
  }
}

class PageGeste extends StatefulWidget {
  const PageGeste({super.key});

  @override
  State<PageGeste> createState() => _PageGesteState();
}

class _PageGesteState extends State<PageGeste> {
  String _dernierGeste = 'aucun';
  bool _presse = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('GestureDetector')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text('Dernier geste : $_dernierGeste'),
            const SizedBox(height: 24),
            GestureDetector(
              onTap: () => setState(() => _dernierGeste = 'appui simple'),
              onDoubleTap: () => setState(() => _dernierGeste = 'double appui'),
              onLongPress: () => setState(() => _dernierGeste = 'appui long'),
              onTapDown: (TapDownDetails details) {
                setState(() => _presse = true);
              },
              onTapUp: (TapUpDetails details) {
                setState(() => _presse = false);
              },
              onTapCancel: () => setState(() => _presse = false),
              child: AnimatedContainer(
                duration: const Duration(milliseconds: 120),
                width: _presse ? 180 : 200,
                height: _presse ? 108 : 120,
                decoration: BoxDecoration(
                  color: _presse ? Colors.indigo.shade800 : Colors.indigo,
                  borderRadius: BorderRadius.circular(16),
                ),
                alignment: Alignment.center,
                child: const Text(
                  'Coffre au trésor',
                  style: TextStyle(color: Colors.white, fontSize: 18),
                ),
              ),
            ),
            const SizedBox(height: 24),
            const Text(
              'Essayez : appui, double appui, appui long.',
              style: TextStyle(fontSize: 12, color: Colors.grey),
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
Dernier geste : aucun

     ┌──────────────────────┐
     │   Coffre au trésor   │      <- se rétrécit quand on appuie
     └──────────────────────┘

  -> appui simple   -> Dernier geste : appui simple
  -> double appui   -> Dernier geste : double appui
  -> appui maintenu -> Dernier geste : appui long
```

**Le piège de la zone morte.** Un `GestureDetector` n'écoute que sur la surface **peinte** de son enfant. Si l'enfant est un `Container` sans `color` ni `decoration`, la zone vide n'est pas touchable :

```dart
// Zone morte : le Container est transparent, les appuis passent au travers.
GestureDetector(
  onTap: () {},
  child: Container(
    width: 200,
    height: 100,
    child: const Text('Cliquez ici'),
  ),
)
```

Deux corrections possibles :

```dart
// Correction 1 : donner une couleur (même transparente).
child: Container(color: Colors.transparent, ...)

// Correction 2 : forcer le comportement d'interception.
GestureDetector(
  behavior: HitTestBehavior.opaque,
  onTap: () {},
  child: ...,
)
```

`HitTestBehavior.opaque` signifie « toute ma boîte capte les appuis, même les pixels transparents ».

---

## 49.12 — `InkWell` et l'effet d'ondulation

`GestureDetector` fonctionne mais ne donne **aucun retour visuel**. L'utilisateur appuie et rien ne bouge : l'interface semble cassée.

`InkWell` fait la même chose **et** dessine l'ondulation Material (le *ripple*) : un cercle d'encre qui part du doigt et se propage.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageInkWell(),
    );
  }
}

class PageInkWell extends StatefulWidget {
  const PageInkWell({super.key});

  @override
  State<PageInkWell> createState() => _PageInkWellState();
}

class _PageInkWellState extends State<PageInkWell> {
  String _selection = 'aucune';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('InkWell')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text('Classe choisie : $_selection'),
            const SizedBox(height: 24),

            // Carte cliquable avec ondulation correcte.
            _CarteClasse(
              nom: 'Guerrier',
              icone: Icons.shield,
              couleur: Colors.red,
              onTap: () => setState(() => _selection = 'Guerrier'),
            ),
            const SizedBox(height: 16),
            _CarteClasse(
              nom: 'Mage',
              icone: Icons.auto_fix_high,
              couleur: Colors.purple,
              onTap: () => setState(() => _selection = 'Mage'),
            ),
            const SizedBox(height: 16),
            _CarteClasse(
              nom: 'Voleur',
              icone: Icons.visibility_off,
              couleur: Colors.teal,
              onTap: () => setState(() => _selection = 'Voleur'),
            ),
          ],
        ),
      ),
    );
  }
}

class _CarteClasse extends StatelessWidget {
  const _CarteClasse({
    required this.nom,
    required this.icone,
    required this.couleur,
    required this.onTap,
  });

  final String nom;
  final IconData icone;
  final Color couleur;
  final VoidCallback onTap;

  @override
  Widget build(BuildContext context) {
    // 1. Material fournit la surface sur laquelle l'encre se répand.
    return Material(
      color: couleur.withValues(alpha: 0.12),
      borderRadius: BorderRadius.circular(16),
      clipBehavior: Clip.antiAlias,
      child: InkWell(
        // 2. Le même rayon que le Material, sinon l'encre déborde des coins.
        borderRadius: BorderRadius.circular(16),
        onTap: onTap,
        child: Padding(
          padding: const EdgeInsets.all(20),
          child: Row(
            children: <Widget>[
              Icon(icone, color: couleur, size: 32),
              const SizedBox(width: 16),
              Text(nom, style: const TextStyle(fontSize: 20)),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
Classe choisie : aucune

  ┌────────────────────────────────┐
  │  ◈   Guerrier                 │   <- au doigt : cercle d'encre
  └────────────────────────────────┘
  ┌────────────────────────────────┐
  │  ✦   Mage                     │
  └────────────────────────────────┘
  ┌────────────────────────────────┐
  │  ◉   Voleur                   │
  └────────────────────────────────┘

Après un appui sur « Mage » : Classe choisie : Mage
```

### L'ordre des widgets est critique

L'ondulation est de l'encre. L'encre a besoin d'une **surface**. Cette surface est le widget `Material`. Et l'encre se dessine **au-dessus** du `Material` mais **en dessous** de l'enfant de l'`InkWell`.

```text
        ORDRE CORRECT                    ORDRE FAUX

        Material         <- surface      Container(color: ...)
          └── InkWell    <- encre          └── InkWell
                └── Padding                      └── Padding
                      └── contenu                      └── contenu

        L'encre est visible.             L'encre est PEINTE SOUS
                                         le fond du Container,
                                         donc invisible.
```

**Erreur classique numéro un :** mettre l'`InkWell` **dans** un `Container` coloré.

```dart
// FAUX : l'ondulation existe mais est cachée par la couleur du Container.
Container(
  color: Colors.blue.shade50,
  child: InkWell(
    onTap: () {},
    child: const Text('Invisible'),
  ),
)

// CORRECT : Material porte la couleur, InkWell est au-dessus.
Material(
  color: Colors.blue.shade50,
  child: InkWell(
    onTap: () {},
    child: const Text('Visible'),
  ),
)
```

**Erreur classique numéro deux :** oublier `borderRadius` sur l'`InkWell`. L'encre est alors rectangulaire et dépasse des coins arrondis. Il faut le **même** rayon des deux côtés, ou bien `clipBehavior: Clip.antiAlias` sur le `Material`, comme ci-dessus.

**Bonne nouvelle.** `Card`, `ListTile`, `Scaffold` et `AppBar` contiennent déjà un `Material`. Dans un `ListTile`, `onTap:` suffit ; l'ondulation fonctionne d'elle-même.

### `InkWell` ou `GestureDetector` ?

| Besoin | Widget |
| --- | --- |
| Zone cliquable avec retour visuel Material | `InkWell` |
| Zone cliquable **sans** retour (ex. fermer le clavier) | `GestureDetector` |
| Glissement, déplacement, zoom | `GestureDetector` |
| Double appui + ondulation | `InkWell` (il a `onDoubleTap`) |

---

## 49.13 — `TextField`

`TextField` est le champ de saisie de base. Sa forme minimale tient en une ligne :

```dart
TextField()
```

Cela suffit à afficher un champ fonctionnel : l'utilisateur tape, le clavier apparaît, le texte s'affiche. Mais rien n'est encore récupéré côté code.

Il y a **deux façons** de lire ce que l'utilisateur tape :

```text
   MÉTHODE A : onChanged                MÉTHODE B : TextEditingController

   TextField(                            final c = TextEditingController();
     onChanged: (String texte) {         TextField(controller: c)
       setState(() => _nom = texte);     ...
     },                                  print(c.text);
   )
   
   Simple, sans dispose.                 Permet aussi d'ÉCRIRE dans le champ,
   Suffit pour lire.                     de vider, de pré-remplir.
```

Voici les deux, dans un même programme.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageTextField(),
    );
  }
}

class PageTextField extends StatefulWidget {
  const PageTextField({super.key});

  @override
  State<PageTextField> createState() => _PageTextFieldState();
}

class _PageTextFieldState extends State<PageTextField> {
  String _nomHeros = '';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('TextField')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: <Widget>[
            const Text('Nom de votre héros :'),
            const SizedBox(height: 8),

            TextField(
              onChanged: (String valeur) {
                setState(() {
                  _nomHeros = valeur;
                });
              },
            ),
            const SizedBox(height: 24),

            Text(
              _nomHeros.isEmpty
                  ? 'Aucun nom saisi.'
                  : 'Bienvenue, $_nomHeros !',
              style: const TextStyle(fontSize: 20),
            ),
            const SizedBox(height: 8),
            Text('Longueur : ${_nomHeros.length} caractères'),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
Nom de votre héros :
─────────────────────────────

Aucun nom saisi.
Longueur : 0 caractères


Après avoir tapé « Kael » :

Nom de votre héros :
Kael
─────────────────────────────

Bienvenue, Kael !
Longueur : 4 caractères
```

Le texte se met à jour **à chaque frappe** car `onChanged` est appelé à chaque caractère et déclenche un `setState`.

Les paramètres de `TextField` que nous verrons dans les sections suivantes :

| Paramètre | Section |
| --- | --- |
| `decoration` | 49.14 |
| `controller` | 49.15 |
| `onChanged` | 49.17 |
| `onSubmitted` | 49.18 |
| `keyboardType` | 49.19 |
| `obscureText` | 49.20 |
| `maxLength`, `inputFormatters` | 49.21 |
| `focusNode`, `textInputAction` | 49.22 |
| `enabled`, `readOnly`, `maxLines` | ci-dessous |

Trois paramètres simples à connaître tout de suite :

```dart
// Champ désactivé : grisé, non modifiable, pas de clavier.
TextField(enabled: false)

// Champ en lecture seule : apparence normale, sélection possible,
// mais aucune modification. Utile pour un champ rempli par un sélecteur.
TextField(readOnly: true)

// Champ multiligne : 3 lignes visibles, agrandissement jusqu'à 6.
TextField(minLines: 3, maxLines: 6)

// Champ qui grandit indéfiniment avec le texte.
TextField(maxLines: null, keyboardType: TextInputType.multiline)
```

---

## 49.14 — `InputDecoration` : label, hint, icônes, bordures

Un `TextField` nu est une simple ligne. Tout son habillage passe par le paramètre `decoration:`, qui reçoit un objet `InputDecoration`.

Les paramètres essentiels :

| Paramètre | Rôle |
| --- | --- |
| `labelText` | l'étiquette ; flotte au-dessus quand le champ est actif |
| `hintText` | l'exemple grisé ; disparaît dès la première frappe |
| `helperText` | l'aide sous le champ, en permanence |
| `errorText` | le message d'erreur, en rouge ; remplace `helperText` |
| `prefixIcon` | l'icône à gauche, **dans** le champ |
| `suffixIcon` | l'icône à droite, **dans** le champ ; souvent un `IconButton` |
| `prefixText` / `suffixText` | un texte fixe à gauche / droite |
| `border` | la bordure par défaut |
| `enabledBorder` | la bordure au repos |
| `focusedBorder` | la bordure quand le champ a le focus |
| `errorBorder` | la bordure en cas d'erreur |
| `filled` / `fillColor` | un fond plein derrière le champ |
| `counterText` | le compteur en bas à droite ; `''` le masque |
| `contentPadding` | la marge intérieure |

**La différence `labelText` / `hintText` en une image :**

```text
   AU REPOS                       AVEC LE FOCUS / DU TEXTE

  ┌────────────────────────┐     ┌─ Nom du héros ─────────┐
  │ Nom du héros           │     │ Kael                   │
  └────────────────────────┘     └────────────────────────┘
    ^ labelText, à l'intérieur     ^ labelText remonté et rétréci

  ┌────────────────────────┐
  │ ex. Kael le Vaillant   │       hintText : disparaît
  └────────────────────────┘       dès la première frappe
```

**Règle :** utilisez toujours un `labelText` (il reste visible une fois le champ rempli, donc l'utilisateur sait ce qu'il a saisi). Le `hintText` est un **complément**, jamais un remplacement.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageDecoration(),
    );
  }
}

class PageDecoration extends StatelessWidget {
  const PageDecoration({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('InputDecoration')),
      body: ListView(
        padding: const EdgeInsets.all(20),
        children: <Widget>[
          // 1. Décoration minimale : juste un label.
          const TextField(
            decoration: InputDecoration(labelText: 'Nom du héros'),
          ),
          const SizedBox(height: 24),

          // 2. Label + hint + helper.
          const TextField(
            decoration: InputDecoration(
              labelText: 'Nom du héros',
              hintText: 'ex. Kael le Vaillant',
              helperText: 'Entre 3 et 20 caractères.',
            ),
          ),
          const SizedBox(height: 24),

          // 3. Bordure encadrée + icône de préfixe.
          const TextField(
            decoration: InputDecoration(
              labelText: 'Royaume',
              prefixIcon: Icon(Icons.castle),
              border: OutlineInputBorder(),
            ),
          ),
          const SizedBox(height: 24),

          // 4. Champ « rempli », style Material 3.
          const TextField(
            decoration: InputDecoration(
              labelText: 'Guilde',
              prefixIcon: Icon(Icons.groups),
              filled: true,
              border: OutlineInputBorder(
                borderRadius: BorderRadius.all(Radius.circular(16)),
                borderSide: BorderSide.none,
              ),
            ),
          ),
          const SizedBox(height: 24),

          // 5. Bordures personnalisées selon l'état.
          TextField(
            decoration: InputDecoration(
              labelText: 'Or de départ',
              prefixText: 'Or ',
              suffixText: 'pièces',
              enabledBorder: OutlineInputBorder(
                borderRadius: BorderRadius.circular(12),
                borderSide: const BorderSide(color: Colors.grey, width: 1),
              ),
              focusedBorder: OutlineInputBorder(
                borderRadius: BorderRadius.circular(12),
                borderSide: const BorderSide(color: Colors.indigo, width: 2.5),
              ),
            ),
          ),
          const SizedBox(height: 24),

          // 6. Erreur affichée en dur (pour la démonstration).
          const TextField(
            decoration: InputDecoration(
              labelText: 'Niveau',
              border: OutlineInputBorder(),
              errorText: 'Le niveau doit être un nombre entre 1 et 99.',
            ),
          ),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
1.  Nom du héros
    ──────────────────────────────

2.  Nom du héros
    ──────────────────────────────
    Entre 3 et 20 caractères.

3.  ┌─ Royaume ───────────────────┐
    │ ▲                          │
    └─────────────────────────────┘

4.  ╭─────────────────────────────╮
    │ ◉  Guilde                  │   fond gris clair, sans bordure
    ╰─────────────────────────────╯

5.  ┌─ Or de départ ──────────────┐
    │ Or           pièces         │   bordure indigo épaisse au focus
    └─────────────────────────────┘

6.  ┌─ Niveau ────────────────────┐   bordure ROUGE
    │                             │
    └─────────────────────────────┘
    Le niveau doit être un nombre entre 1 et 99.
```

**Attention :** dès que `errorText` n'est pas `null`, la bordure devient rouge, le label devient rouge, et `helperText` est masqué. Vous n'avez rien d'autre à faire. À partir de la section 49.26, ce n'est même plus vous qui remplirez `errorText` : le `validator` le fera.

### Décorer tous les champs d'un coup

Comme pour les boutons, la répétition se traite dans le thème :

```dart
ThemeData(
  colorSchemeSeed: Colors.indigo,
  useMaterial3: true,
  inputDecorationTheme: InputDecorationTheme(
    filled: true,
    border: OutlineInputBorder(
      borderRadius: BorderRadius.circular(12),
    ),
    contentPadding: const EdgeInsets.symmetric(
      horizontal: 16,
      vertical: 14,
    ),
  ),
)
```

---

## 49.15 — `TextEditingController`

Un `TextEditingController` est un objet qui **détient** le texte d'un champ. Il permet trois choses que `onChanged` ne permet pas :

1. **lire** la valeur à n'importe quel moment (`controller.text`), sans stocker de copie ;
2. **écrire** dans le champ depuis le code (`controller.text = 'Kael'`) ;
3. **vider** le champ (`controller.clear()`).

```text
   ┌──────────────────────┐
   │ TextEditingController│   <- détient la chaîne + la position du curseur
   │   text : "Kael"      │
   │   selection : 4..4   │
   └──────────┬───────────┘
              │ controller:
              v
   ┌──────────────────────┐
   │      TextField       │   <- affiche ce que le contrôleur contient
   └──────────────────────┘

   Écrire dans le contrôleur met le champ à jour.
   Taper dans le champ met le contrôleur à jour.
```

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageControleur(),
    );
  }
}

class PageControleur extends StatefulWidget {
  const PageControleur({super.key});

  @override
  State<PageControleur> createState() => _PageControleurState();
}

class _PageControleurState extends State<PageControleur> {
  // 1. Le contrôleur est un champ final du State, créé une seule fois.
  final TextEditingController _nomControleur = TextEditingController();
  final TextEditingController _titreControleur =
      TextEditingController(text: 'Apprenti'); // valeur initiale

  String _resultat = '';

  // 2. Libération obligatoire (détail en 49.16).
  @override
  void dispose() {
    _nomControleur.dispose();
    _titreControleur.dispose();
    super.dispose();
  }

  void _valider() {
    setState(() {
      _resultat = '${_titreControleur.text} ${_nomControleur.text}'.trim();
    });
  }

  void _remplirAuHasard() {
    // 3. Écrire dans le champ depuis le code.
    _nomControleur.text = 'Kael';
    _titreControleur.text = 'Chevalier';
    _valider();
  }

  void _toutEffacer() {
    _nomControleur.clear();
    _titreControleur.clear();
    setState(() => _resultat = '');
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('TextEditingController')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            TextField(
              controller: _titreControleur,
              decoration: const InputDecoration(
                labelText: 'Titre',
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 16),
            TextField(
              controller: _nomControleur,
              decoration: const InputDecoration(
                labelText: 'Nom',
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 24),
            Row(
              children: <Widget>[
                Expanded(
                  child: FilledButton(
                    onPressed: _valider,
                    child: const Text('Valider'),
                  ),
                ),
                const SizedBox(width: 12),
                Expanded(
                  child: OutlinedButton(
                    onPressed: _remplirAuHasard,
                    child: const Text('Exemple'),
                  ),
                ),
                const SizedBox(width: 12),
                Expanded(
                  child: OutlinedButton(
                    onPressed: _toutEffacer,
                    child: const Text('Effacer'),
                  ),
                ),
              ],
            ),
            const SizedBox(height: 24),
            Text(
              _resultat.isEmpty ? 'Aucun héros.' : 'Héros : $_resultat',
              style: const TextStyle(fontSize: 20),
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
┌─ Titre ─────────────────────┐
│ Apprenti                    │      <- valeur initiale du contrôleur
└─────────────────────────────┘
┌─ Nom ───────────────────────┐
│                             │
└─────────────────────────────┘

  [Valider]  [Exemple]  [Effacer]

Aucun héros.

  -> appui sur « Exemple » ->

┌─ Titre ─────────────────────┐
│ Chevalier                   │
└─────────────────────────────┘
┌─ Nom ───────────────────────┐
│ Kael                        │
└─────────────────────────────┘

Héros : Chevalier Kael
```

**Trois règles absolues.**

1. **Le contrôleur se crée dans le `State`, pas dans le `build`.** S'il est créé dans `build`, un nouveau contrôleur naît à chaque reconstruction, le champ se vide, et vous fuyez de la mémoire.

   ```dart
   // FAUX
   @override
   Widget build(BuildContext context) {
     final TextEditingController c = TextEditingController(); // catastrophe
     return TextField(controller: c);
   }
   ```

2. **On ne combine pas `controller:` et un `initialValue:`.** Pour une valeur de départ, passez-la au constructeur : `TextEditingController(text: 'Apprenti')`. Sur un `TextFormField` (section 49.25), donner à la fois `controller:` et `initialValue:` lève une assertion.

3. **`controller.text = ...` ne déclenche pas `onChanged`.** C'est logique : `onChanged` signale une action de l'**utilisateur**. Si vous voulez réagir à une écriture programmatique, appelez vous-même votre logique, comme `_valider()` ci-dessus.

### Écouter le contrôleur

Un contrôleur est un `ChangeNotifier` (vous reverrez ce type au chapitre 52). On peut donc s'y abonner :

```dart
@override
void initState() {
  super.initState();
  _nomControleur.addListener(_surChangement);
}

void _surChangement() {
  setState(() {});   // reconstruit à chaque frappe
}

@override
void dispose() {
  _nomControleur.removeListener(_surChangement);
  _nomControleur.dispose();
  super.dispose();
}
```

C'est utile quand plusieurs widgets dépendent du contenu du champ (par exemple activer un bouton dès que le champ n'est pas vide).

---

## 49.16 — Libérer le contrôleur dans `dispose()`

Cette section est courte, mais c'est peut-être la plus importante du chapitre.

> **Tout objet que vous créez dans un `State` et qui possède une méthode `dispose()` doit être libéré dans le `dispose()` du `State`.**

Cela concerne : `TextEditingController`, `FocusNode`, `ScrollController` (chapitre 48), `AnimationController`, `PageController`, `TabController`, `StreamSubscription`.

### Pourquoi

Un `TextEditingController` maintient une liste d'auditeurs (les widgets qui l'écoutent) et des ressources natives liées au clavier. Quand l'écran disparaît, si personne ne libère le contrôleur, ces ressources restent en mémoire. Sur un écran ouvert et fermé cent fois, vous accumulez cent contrôleurs vivants.

```text
   SANS dispose()                        AVEC dispose()

   Ouverture 1  -> contrôleur A          Ouverture 1  -> contrôleur A
   Fermeture 1  -> A reste en mémoire    Fermeture 1  -> A libéré
   Ouverture 2  -> contrôleur B          Ouverture 2  -> contrôleur B
   Fermeture 2  -> A et B en mémoire     Fermeture 2  -> B libéré
   ...                                    ...
   Ouverture 100 -> 100 contrôleurs      Ouverture 100 -> 1 contrôleur
                    vivants
```

### Le squelette exact

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageDispose(),
    );
  }
}

class PageDispose extends StatefulWidget {
  const PageDispose({super.key});

  @override
  State<PageDispose> createState() => _PageDisposeState();
}

class _PageDisposeState extends State<PageDispose> {
  // 1. DÉCLARATION : final, dans le State.
  final TextEditingController _nom = TextEditingController();
  final TextEditingController _classe = TextEditingController();
  final FocusNode _focusNom = FocusNode();

  @override
  void initState() {
    super.initState();
    debugPrint('initState : les contrôleurs sont prêts.');
  }

  // 2. LIBÉRATION : dans dispose, AVANT super.dispose().
  @override
  void dispose() {
    debugPrint('dispose : libération des contrôleurs.');
    _nom.dispose();
    _classe.dispose();
    _focusNom.dispose();
    super.dispose(); // toujours en DERNIER
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('dispose()')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          children: <Widget>[
            TextField(
              controller: _nom,
              focusNode: _focusNom,
              decoration: const InputDecoration(labelText: 'Nom'),
            ),
            const SizedBox(height: 16),
            TextField(
              controller: _classe,
              decoration: const InputDecoration(labelText: 'Classe'),
            ),
            const SizedBox(height: 24),
            const Text(
              'Regardez la console : initState au premier affichage, '
              'dispose quand vous quittez la page.',
              textAlign: TextAlign.center,
              style: TextStyle(fontSize: 12, color: Colors.grey),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat (console) :**

```text
flutter: initState : les contrôleurs sont prêts.
   ... l'utilisateur quitte l'écran ...
flutter: dispose : libération des contrôleurs.
```

### Les trois erreurs à connaître

**Erreur 1 — `super.dispose()` en premier.**

```dart
// FAUX
@override
void dispose() {
  super.dispose();     // le State est démonté ici
  _nom.dispose();      // trop tard
}
```

L'ordre correct est : **mes objets d'abord, `super.dispose()` en dernier.** C'est l'inverse de `initState`, où `super.initState()` vient en premier.

**Erreur 2 — utiliser le contrôleur après `dispose`.**

```dart
Future<void> _envoyer() async {
  await Future<void>.delayed(const Duration(seconds: 3));
  print(_nom.text);    // si l'écran est fermé : « A TextEditingController
                       // was used after being disposed. »
}
```

Correction : `if (!mounted) return;` juste après chaque `await`.

**Erreur 3 — oublier un contrôleur sur cinq.** Dans un formulaire à six champs, on oublie toujours le dernier. Deux parades :

```dart
// Parade 1 : une liste, et une boucle.
final List<TextEditingController> _controleurs = <TextEditingController>[
  TextEditingController(),
  TextEditingController(),
  TextEditingController(),
];

@override
void dispose() {
  for (final TextEditingController c in _controleurs) {
    c.dispose();
  }
  super.dispose();
}
```

```text
Parade 2 : activez la règle d'analyse dans analysis_options.yaml

linter:
  rules:
    - close_sinks
    - cancel_subscriptions
```

Et surtout : Flutter lui-même vous prévient. En mode debug, un contrôleur non libéré déclenche à terme le message :

```text
A TextEditingController was used after being disposed.
Once you have called dispose() on a TextEditingController,
it can no longer be used.
```

---

## 49.17 — `onChanged`

`onChanged` est appelé **à chaque modification du texte** : une frappe, une suppression, un collage.

Sa signature est `void Function(String)`. Le paramètre est la **nouvelle valeur complète** du champ, pas le caractère tapé.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageOnChanged(),
    );
  }
}

class PageOnChanged extends StatefulWidget {
  const PageOnChanged({super.key});

  @override
  State<PageOnChanged> createState() => _PageOnChangedState();
}

class _PageOnChangedState extends State<PageOnChanged> {
  static const List<String> _tousLesEnnemis = <String>[
    'Gobelin', 'Gargouille', 'Golem', 'Dragon', 'Drake',
    'Squelette', 'Spectre', 'Troll', 'Sorcier', 'Vampire',
  ];

  List<String> _resultats = _tousLesEnnemis;
  int _frappes = 0;

  void _filtrer(String recherche) {
    setState(() {
      _frappes += 1;
      final String r = recherche.trim().toLowerCase();
      if (r.isEmpty) {
        _resultats = _tousLesEnnemis;
      } else {
        _resultats = _tousLesEnnemis
            .where((String e) => e.toLowerCase().contains(r))
            .toList();
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('onChanged')),
      body: Column(
        children: <Widget>[
          Padding(
            padding: const EdgeInsets.all(16),
            child: TextField(
              onChanged: _filtrer,
              decoration: const InputDecoration(
                labelText: 'Rechercher un ennemi',
                prefixIcon: Icon(Icons.search),
                border: OutlineInputBorder(),
              ),
            ),
          ),
          Text('onChanged appelé $_frappes fois'),
          const Divider(),
          Expanded(
            child: _resultats.isEmpty
                ? const Center(child: Text('Aucun ennemi trouvé.'))
                : ListView.builder(
                    itemCount: _resultats.length,
                    itemBuilder: (BuildContext context, int index) {
                      return ListTile(
                        leading: const Icon(Icons.pest_control),
                        title: Text(_resultats[index]),
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

**Résultat :**

```text
┌─ Rechercher un ennemi ──────────┐
│ ⌕ go                           │
└─────────────────────────────────┘
onChanged appelé 2 fois
─────────────────────────────────
  ◆  Gobelin
  ◆  Golem

Après avoir tapé « gob » :
onChanged appelé 3 fois
  ◆  Gobelin
```

**Attention à la fréquence.** `onChanged` se déclenche à chaque caractère. Pour une recherche locale sur dix éléments, aucun problème. Pour une **requête réseau**, c'est inacceptable : taper « dragon » enverrait six requêtes.

La solution s'appelle le *debounce* : attendre que l'utilisateur s'arrête de taper. Elle utilise un `Timer` :

```dart
import 'dart:async';

Timer? _minuteur;

void _rechercher(String texte) {
  _minuteur?.cancel();
  _minuteur = Timer(const Duration(milliseconds: 400), () {
    // Appelée seulement 400 ms après la DERNIÈRE frappe.
    _lancerLaRequete(texte);
  });
}

@override
void dispose() {
  _minuteur?.cancel();
  super.dispose();
}
```

Nous réutiliserons ce motif au chapitre 53, avec de vraies requêtes HTTP.

---

## 49.18 — `onSubmitted`

`onSubmitted` est appelé quand l'utilisateur **valide** le champ : appui sur la touche Entrée d'un clavier physique, ou sur le bouton d'action du clavier virtuel (la loupe, la flèche, « OK »...).

Ce bouton d'action se choisit avec `textInputAction`.

| `TextInputAction` | Aspect du bouton |
| --- | --- |
| `TextInputAction.done` | « OK » ou coche |
| `TextInputAction.next` | flèche vers le champ suivant |
| `TextInputAction.search` | loupe |
| `TextInputAction.send` | avion en papier |
| `TextInputAction.go` | « Aller » |
| `TextInputAction.newline` | retour à la ligne (multiligne) |

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageOnSubmitted(),
    );
  }
}

class PageOnSubmitted extends StatefulWidget {
  const PageOnSubmitted({super.key});

  @override
  State<PageOnSubmitted> createState() => _PageOnSubmittedState();
}

class _PageOnSubmittedState extends State<PageOnSubmitted> {
  final TextEditingController _controleur = TextEditingController();
  final List<String> _journalDeQuete = <String>[];

  @override
  void dispose() {
    _controleur.dispose();
    super.dispose();
  }

  void _ajouterEntree(String texte) {
    final String propre = texte.trim();
    if (propre.isEmpty) {
      return;
    }
    setState(() {
      _journalDeQuete.insert(0, propre);
    });
    _controleur.clear();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Journal de quête')),
      body: Column(
        children: <Widget>[
          Padding(
            padding: const EdgeInsets.all(16),
            child: TextField(
              controller: _controleur,
              // Le bouton du clavier devient un avion en papier.
              textInputAction: TextInputAction.send,
              // Déclenché à l'appui sur ce bouton.
              onSubmitted: _ajouterEntree,
              decoration: InputDecoration(
                labelText: 'Nouvelle entrée',
                hintText: 'ex. Trouvé la clé du donjon',
                border: const OutlineInputBorder(),
                suffixIcon: IconButton(
                  icon: const Icon(Icons.send),
                  tooltip: 'Ajouter',
                  onPressed: () => _ajouterEntree(_controleur.text),
                ),
              ),
            ),
          ),
          Expanded(
            child: _journalDeQuete.isEmpty
                ? const Center(child: Text('Journal vide.'))
                : ListView.builder(
                    itemCount: _journalDeQuete.length,
                    itemBuilder: (BuildContext context, int index) {
                      return ListTile(
                        leading: CircleAvatar(
                          child: Text('${_journalDeQuete.length - index}'),
                        ),
                        title: Text(_journalDeQuete[index]),
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

**Résultat :**

```text
┌─ Nouvelle entrée ───────────────┐
│ Trouvé la clé du donjon      ▶  │
└─────────────────────────────────┘

  -> validation clavier ->

┌─ Nouvelle entrée ───────────────┐
│                              ▶  │   <- champ vidé
└─────────────────────────────────┘
 (1)  Trouvé la clé du donjon

  -> deuxième entrée ->
 (2)  Vaincu le gobelin
 (1)  Trouvé la clé du donjon
```

**Deux remarques.**

1. **`onSubmitted` reçoit le texte**, exactement comme `onChanged`. Sa signature est aussi `void Function(String)`.
2. **Proposez toujours une alternative visuelle.** Ici, le `suffixIcon` fait la même chose que la touche du clavier. Certains utilisateurs ne pensent jamais à valider au clavier.

**Attention à `maxLines: null`.** Sur un champ multiligne, la touche Entrée insère un saut de ligne au lieu de déclencher `onSubmitted`. Si vous avez besoin des deux, gardez `maxLines: 1` et ajoutez un bouton d'envoi.

---

## 49.19 — `keyboardType`

Le paramètre `keyboardType` choisit **quel clavier** s'ouvre. C'est un détail invisible dans le code, mais l'un des plus visibles pour l'utilisateur.

| Valeur | Clavier affiché |
| --- | --- |
| `TextInputType.text` | clavier alphabétique standard (défaut) |
| `TextInputType.number` | pavé numérique |
| `TextInputType.numberWithOptions(decimal: true)` | pavé numérique avec virgule |
| `TextInputType.numberWithOptions(signed: true)` | pavé numérique avec signe moins |
| `TextInputType.phone` | pavé téléphonique avec `*`, `#`, `+` |
| `TextInputType.emailAddress` | clavier avec `@` et `.` visibles |
| `TextInputType.url` | clavier avec `/` et `.com` |
| `TextInputType.multiline` | clavier avec touche Entrée |
| `TextInputType.name` | clavier optimisé pour les noms propres |
| `TextInputType.datetime` | clavier avec chiffres et séparateurs |
| `TextInputType.visiblePassword` | clavier de mot de passe visible |

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageClaviers(),
    );
  }
}

class PageClaviers extends StatelessWidget {
  const PageClaviers({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('keyboardType')),
      body: ListView(
        padding: const EdgeInsets.all(20),
        children: const <Widget>[
          TextField(
            keyboardType: TextInputType.name,
            textCapitalization: TextCapitalization.words,
            decoration: InputDecoration(
              labelText: 'Nom du personnage',
              prefixIcon: Icon(Icons.person),
              border: OutlineInputBorder(),
            ),
          ),
          SizedBox(height: 16),
          TextField(
            keyboardType: TextInputType.number,
            decoration: InputDecoration(
              labelText: 'Niveau',
              prefixIcon: Icon(Icons.trending_up),
              border: OutlineInputBorder(),
            ),
          ),
          SizedBox(height: 16),
          TextField(
            keyboardType: TextInputType.numberWithOptions(decimal: true),
            decoration: InputDecoration(
              labelText: 'Poids de l\'équipement (kg)',
              prefixIcon: Icon(Icons.fitness_center),
              border: OutlineInputBorder(),
            ),
          ),
          SizedBox(height: 16),
          TextField(
            keyboardType: TextInputType.emailAddress,
            decoration: InputDecoration(
              labelText: 'Adresse e-mail',
              prefixIcon: Icon(Icons.email),
              border: OutlineInputBorder(),
            ),
          ),
          SizedBox(height: 16),
          TextField(
            keyboardType: TextInputType.phone,
            decoration: InputDecoration(
              labelText: 'Téléphone',
              prefixIcon: Icon(Icons.phone),
              border: OutlineInputBorder(),
            ),
          ),
          SizedBox(height: 16),
          TextField(
            keyboardType: TextInputType.multiline,
            minLines: 3,
            maxLines: 6,
            decoration: InputDecoration(
              labelText: 'Histoire du personnage',
              alignLabelWithHint: true,
              border: OutlineInputBorder(),
            ),
          ),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
Champ « Niveau » sélectionné :

  ┌───────────────────┐
  │  1   2   3        │
  │  4   5   6        │      <- pavé numérique seul
  │  7   8   9        │
  │      0    ←       │
  └───────────────────┘

Champ « Adresse e-mail » sélectionné :

  ┌───────────────────────────────┐
  │ a z e r t y u i o p           │
  │  q s d f g h j k l m          │
  │   w x c v b n  @  .  ←        │   <- @ et . directement accessibles
  └───────────────────────────────┘
```

**Trois avertissements essentiels.**

1. **`keyboardType` ne valide rien.** C'est une **suggestion** au système. Sur un clavier physique, ou par copier-coller, l'utilisateur peut mettre n'importe quoi dans un champ `TextInputType.number`. Vous devez **toujours** valider dans le code (sections 49.26 et 49.31).
2. **`keyboardType` n'empêche pas la saisie.** Pour restreindre réellement les caractères, il faut `inputFormatters` (section 49.21).
3. `textCapitalization` complète bien `TextInputType.name` : `TextCapitalization.words` met une majuscule au début de chaque mot, `.sentences` au début de chaque phrase, `.characters` met tout en majuscules.

---

## 49.20 — `obscureText` pour un mot de passe

`obscureText: true` remplace chaque caractère par un point. C'est tout ce qu'il faut pour un champ mot de passe.

Deux compléments indispensables en pratique :

- `obscuringCharacter` pour choisir le symbole (par défaut `•`) ;
- un `suffixIcon` en forme d'œil pour basculer la visibilité.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageMotDePasse(),
    );
  }
}

class PageMotDePasse extends StatefulWidget {
  const PageMotDePasse({super.key});

  @override
  State<PageMotDePasse> createState() => _PageMotDePasseState();
}

class _PageMotDePasseState extends State<PageMotDePasse> {
  final TextEditingController _motDePasse = TextEditingController();
  bool _masque = true;

  @override
  void initState() {
    super.initState();
    _motDePasse.addListener(() => setState(() {}));
  }

  @override
  void dispose() {
    _motDePasse.dispose();
    super.dispose();
  }

  // Force du mot de passe : 0 à 4.
  int get _force {
    final String m = _motDePasse.text;
    int points = 0;
    if (m.length >= 8) {
      points += 1;
    }
    if (m.contains(RegExp(r'[A-Z]'))) {
      points += 1;
    }
    if (m.contains(RegExp(r'[0-9]'))) {
      points += 1;
    }
    if (m.contains(RegExp(r'[!@#\$%^&*(),.?":{}|<>_-]'))) {
      points += 1;
    }
    return points;
  }

  String get _libelleForce {
    switch (_force) {
      case 0:
      case 1:
        return 'Très faible';
      case 2:
        return 'Faible';
      case 3:
        return 'Correct';
      default:
        return 'Solide';
    }
  }

  Color get _couleurForce {
    switch (_force) {
      case 0:
      case 1:
        return Colors.red;
      case 2:
        return Colors.orange;
      case 3:
        return Colors.lightGreen;
      default:
        return Colors.green;
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('obscureText')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: <Widget>[
            TextField(
              controller: _motDePasse,
              obscureText: _masque,
              obscuringCharacter: '•',
              // Aide les gestionnaires de mots de passe du système.
              autofillHints: const <String>[AutofillHints.newPassword],
              decoration: InputDecoration(
                labelText: 'Mot de passe',
                prefixIcon: const Icon(Icons.lock),
                border: const OutlineInputBorder(),
                suffixIcon: IconButton(
                  icon: Icon(
                    _masque ? Icons.visibility : Icons.visibility_off,
                  ),
                  tooltip: _masque ? 'Afficher' : 'Masquer',
                  onPressed: () => setState(() => _masque = !_masque),
                ),
              ),
            ),
            const SizedBox(height: 16),
            LinearProgressIndicator(
              value: _force / 4,
              minHeight: 8,
              color: _couleurForce,
              backgroundColor: Colors.grey.shade300,
            ),
            const SizedBox(height: 8),
            Text(
              'Force : $_libelleForce',
              style: TextStyle(
                color: _couleurForce,
                fontWeight: FontWeight.bold,
              ),
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
┌─ Mot de passe ──────────────────┐
│ ▪ ••••••••                 ◉  │
└─────────────────────────────────┘
██████░░░░░░░░░░░░░░░░░░░░░░░░░░░   Force : Faible

  -> appui sur l'œil ->

┌─ Mot de passe ──────────────────┐
│ ▪ dragon12                 ◉̶  │   <- texte en clair
└─────────────────────────────────┘

  -> l'utilisateur ajoute « ! » et une majuscule ->
████████████████████████████████░   Force : Solide
```

**Quatre règles de sécurité et d'ergonomie.**

1. **`obscureText: true` impose `maxLines: 1`.** Un champ masqué multiligne lève une assertion.
2. **Toujours offrir l'œil.** Sans lui, les utilisateurs se trompent et abandonnent. C'est aujourd'hui une pratique recommandée, y compris sur les sites bancaires.
3. **Renseignez `autofillHints`.** `AutofillHints.password` pour une connexion, `AutofillHints.newPassword` pour une inscription. Les gestionnaires de mots de passe du système s'en servent.
4. **`obscureText` ne chiffre rien.** Le texte est en clair dans le contrôleur et dans la mémoire. Le masquage protège uniquement contre le regard par-dessus l'épaule.

---

## 49.21 — `maxLength` et `inputFormatters`

Deux outils différents limitent la saisie.

- **`maxLength`** limite le **nombre de caractères** et affiche un compteur `3/20`.
- **`inputFormatters`** filtre ou transforme le texte **avant** qu'il n'arrive dans le champ.

Les formateurs fournis par Flutter, dans `package:flutter/services.dart` :

| Formateur | Effet |
| --- | --- |
| `FilteringTextInputFormatter.digitsOnly` | n'accepte que les chiffres 0-9 |
| `FilteringTextInputFormatter.allow(RegExp(...))` | n'accepte que ce qui correspond |
| `FilteringTextInputFormatter.deny(RegExp(...))` | refuse ce qui correspond |
| `FilteringTextInputFormatter.singleLineFormatter` | supprime les sauts de ligne |
| `LengthLimitingTextInputFormatter(n)` | limite à `n` caractères, sans compteur |

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageFormateurs(),
    );
  }
}

/// Formateur maison : met tout en majuscules.
class MajusculesFormatter extends TextInputFormatter {
  const MajusculesFormatter();

  @override
  TextEditingValue formatEditUpdate(
    TextEditingValue ancien,
    TextEditingValue nouveau,
  ) {
    return TextEditingValue(
      text: nouveau.text.toUpperCase(),
      selection: nouveau.selection,
    );
  }
}

class PageFormateurs extends StatelessWidget {
  const PageFormateurs({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('maxLength / inputFormatters')),
      body: ListView(
        padding: const EdgeInsets.all(20),
        children: <Widget>[
          // 1. maxLength avec compteur visible.
          const TextField(
            maxLength: 20,
            decoration: InputDecoration(
              labelText: 'Nom du héros (20 max)',
              border: OutlineInputBorder(),
            ),
          ),
          const SizedBox(height: 12),

          // 2. maxLength sans compteur : counterText vide.
          const TextField(
            maxLength: 20,
            decoration: InputDecoration(
              labelText: 'Sans compteur',
              counterText: '',
              border: OutlineInputBorder(),
            ),
          ),
          const SizedBox(height: 12),

          // 3. Chiffres uniquement, 2 caractères maximum.
          TextField(
            keyboardType: TextInputType.number,
            inputFormatters: <TextInputFormatter>[
              FilteringTextInputFormatter.digitsOnly,
              LengthLimitingTextInputFormatter(2),
            ],
            decoration: const InputDecoration(
              labelText: 'Niveau (1-99)',
              border: OutlineInputBorder(),
            ),
          ),
          const SizedBox(height: 12),

          // 4. Lettres et espaces uniquement.
          TextField(
            inputFormatters: <TextInputFormatter>[
              FilteringTextInputFormatter.allow(
                RegExp(r"[a-zA-ZÀ-ÿ' -]"),
              ),
            ],
            decoration: const InputDecoration(
              labelText: 'Royaume (lettres uniquement)',
              border: OutlineInputBorder(),
            ),
          ),
          const SizedBox(height: 12),

          // 5. Formateur personnalisé.
          const TextField(
            inputFormatters: <TextInputFormatter>[
              MajusculesFormatter(),
              LengthLimitingTextInputFormatter(6),
            ],
            decoration: InputDecoration(
              labelText: 'Code de triche',
              hintText: 'ex. IDDQD',
              border: OutlineInputBorder(),
            ),
          ),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
1. ┌─ Nom du héros (20 max) ──────┐
   │ Kael                         │
   └──────────────────────────────┘
                              4/20   <- compteur automatique

3. Tape « a1b2c3 »  ->  le champ contient « 12 »  (lettres refusées,
                        puis coupé à 2 caractères)

5. Tape « iddqd »   ->  le champ contient « IDDQD »
```

**Trois précisions.**

1. **L'ordre des formateurs compte.** Ils s'appliquent dans l'ordre de la liste. `digitsOnly` puis `LengthLimitingTextInputFormatter(2)` : on filtre puis on coupe. Inversé, on couperait « a1 » à deux caractères avant de supprimer le « a », et il ne resterait que « 1 ».
2. **`maxLength` ajoute un compteur, pas `LengthLimitingTextInputFormatter`.** Choisissez selon que vous voulez ce compteur ou non.
3. **Un formateur ne remplace pas un validateur.** Il empêche la saisie, mais un champ vide reste possible. La validation reste indispensable.

---

## 49.22 — `FocusNode` et le passage d'un champ au suivant

Le **focus** désigne le champ qui reçoit les frappes. Un seul champ a le focus à la fois.

Un `FocusNode` est l'objet qui représente ce focus pour un champ. Comme un contrôleur, il se crée dans le `State` et se libère dans `dispose`.

Trois usages :

```dart
_monFocus.requestFocus();          // donner le focus à ce champ
_monFocus.unfocus();               // le lui retirer (ferme le clavier)
_monFocus.hasFocus;                // savoir s'il l'a
```

Le cas le plus fréquent est le passage d'un champ au suivant à la validation clavier.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageFocus(),
    );
  }
}

class PageFocus extends StatefulWidget {
  const PageFocus({super.key});

  @override
  State<PageFocus> createState() => _PageFocusState();
}

class _PageFocusState extends State<PageFocus> {
  final FocusNode _focusNom = FocusNode();
  final FocusNode _focusClasse = FocusNode();
  final FocusNode _focusNiveau = FocusNode();

  @override
  void initState() {
    super.initState();
    // Reconstruit quand le focus change, pour colorer les libellés.
    for (final FocusNode n in <FocusNode>[
      _focusNom,
      _focusClasse,
      _focusNiveau,
    ]) {
      n.addListener(() => setState(() {}));
    }
  }

  @override
  void dispose() {
    _focusNom.dispose();
    _focusClasse.dispose();
    _focusNiveau.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('FocusNode')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            TextField(
              focusNode: _focusNom,
              autofocus: true, // focus dès l'ouverture de l'écran
              textInputAction: TextInputAction.next,
              onSubmitted: (String _) => _focusClasse.requestFocus(),
              decoration: const InputDecoration(
                labelText: 'Nom',
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 16),
            TextField(
              focusNode: _focusClasse,
              textInputAction: TextInputAction.next,
              onSubmitted: (String _) => _focusNiveau.requestFocus(),
              decoration: const InputDecoration(
                labelText: 'Classe',
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 16),
            TextField(
              focusNode: _focusNiveau,
              keyboardType: TextInputType.number,
              textInputAction: TextInputAction.done,
              onSubmitted: (String _) => _focusNiveau.unfocus(),
              decoration: const InputDecoration(
                labelText: 'Niveau',
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 24),
            Text(
              'Focus actuel : '
              '${_focusNom.hasFocus ? "Nom" : _focusClasse.hasFocus ? "Classe" : _focusNiveau.hasFocus ? "Niveau" : "aucun"}',
            ),
            const SizedBox(height: 16),
            FilledButton(
              onPressed: () => _focusNom.requestFocus(),
              child: const Text('Revenir au premier champ'),
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
┌─ Nom ─────────────┐  <- bordure épaisse : ce champ a le focus
│ Kael              │
└───────────────────┘
┌─ Classe ──────────┐
└───────────────────┘
┌─ Niveau ──────────┐
└───────────────────┘
Focus actuel : Nom

  -> touche « suivant » du clavier ->

Focus actuel : Classe   (le curseur a sauté au deuxième champ)
```

### La version sans `FocusNode`

Pour le simple enchaînement « champ suivant », Flutter propose un raccourci qui utilise l'ordre d'apparition dans l'arbre :

```dart
TextField(
  textInputAction: TextInputAction.next,
  onSubmitted: (String _) => FocusScope.of(context).nextFocus(),
)
```

`FocusScope.of(context)` expose aussi `previousFocus()` et `unfocus()`.

**Quand créer des `FocusNode` explicites ?** Quand l'ordre n'est pas linéaire (sauter un champ selon une case cochée), quand vous voulez connaître `hasFocus`, ou quand vous devez donner le focus à un champ précis après une erreur de validation.

---

## 49.23 — Fermer le clavier

Le clavier virtuel masque la moitié de l'écran. Savoir le fermer est indispensable.

Il y a trois manières, à utiliser dans des situations différentes.

### Méthode 1 — retirer le focus

```dart
FocusScope.of(context).unfocus();
```

C'est la méthode générale : plus aucun champ n'a le focus, donc le clavier se ferme. À appeler après une soumission de formulaire.

### Méthode 2 — appui hors des champs

On enveloppe le corps de la page dans un `GestureDetector` (section 49.11) qui retire le focus.

### Méthode 3 — `onTapOutside`

Depuis Flutter 3.7, chaque `TextField` accepte `onTapOutside`, déclenché quand l'utilisateur touche ailleurs :

```dart
TextField(
  onTapOutside: (PointerDownEvent event) =>
      FocusManager.instance.primaryFocus?.unfocus(),
)
```

Voici les trois réunies.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageClavier(),
    );
  }
}

class PageClavier extends StatelessWidget {
  const PageClavier({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Fermer le clavier')),
      // MÉTHODE 2 : tout appui hors d'un champ ferme le clavier.
      body: GestureDetector(
        behavior: HitTestBehavior.opaque,
        onTap: () => FocusScope.of(context).unfocus(),
        child: Padding(
          padding: const EdgeInsets.all(24),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: <Widget>[
              TextField(
                decoration: const InputDecoration(
                  labelText: 'Nom du héros',
                  border: OutlineInputBorder(),
                ),
                // MÉTHODE 3 : ce champ se déselectionne tout seul.
                onTapOutside: (PointerDownEvent event) =>
                    FocusManager.instance.primaryFocus?.unfocus(),
              ),
              const SizedBox(height: 24),
              FilledButton(
                // MÉTHODE 1 : fermeture explicite à la soumission.
                onPressed: () {
                  FocusScope.of(context).unfocus();
                  ScaffoldMessenger.of(context).showSnackBar(
                    const SnackBar(content: Text('Formulaire envoyé.')),
                  );
                },
                child: const Text('Envoyer'),
              ),
              const SizedBox(height: 24),
              const Text(
                'Touchez le champ pour ouvrir le clavier, '
                'puis touchez la zone vide pour le fermer.',
                textAlign: TextAlign.center,
                style: TextStyle(fontSize: 12, color: Colors.grey),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
Clavier ouvert :                    Après appui sur la zone vide :

┌─ Nom du héros ──────┐             ┌─ Nom du héros ──────┐
│ Kael                │             │ Kael                │
└─────────────────────┘             └─────────────────────┘
     [ Envoyer ]                         [ Envoyer ]
┌─────────────────────┐
│ a z e r t y u i o p │             (tout l'écran est visible)
│  q s d f g h j k l  │
└─────────────────────┘
```

**Le paramètre `resizeToAvoidBottomInset`.** Par défaut, le `Scaffold` **redimensionne** son corps quand le clavier apparaît. C'est ce qui provoque parfois un `RenderFlex overflowed` (chapitre 46) : la `Column` n'a plus la place. Deux corrections :

```dart
// Correction recommandée : rendre le corps défilant.
body: SingleChildScrollView(child: Column(...))

// Correction de dernier recours : ne pas redimensionner.
// Le clavier recouvre alors le contenu.
Scaffold(resizeToAvoidBottomInset: false, ...)
```

---

## 49.24 — `Form` et `GlobalKey<FormState>`

Jusqu'ici, chaque champ vivait seul. Pour un vrai formulaire, on veut :

- valider **tous** les champs d'un seul appel ;
- afficher les erreurs de chacun au bon endroit ;
- récupérer toutes les valeurs ensemble ;
- réinitialiser l'ensemble.

C'est le rôle du widget `Form`. Il ne dessine rien : c'est un **conteneur logique** qui coordonne ses champs.

```text
   Form  (+ GlobalKey<FormState>)
     │
     ├── TextFormField  « Nom »       validator: ...
     ├── TextFormField  « Classe »    validator: ...
     └── TextFormField  « Niveau »    validator: ...

   formKey.currentState!.validate()  -> appelle les 3 validators
                                        et retourne true si tous
                                        renvoient null
```

Pour piloter le `Form` depuis l'extérieur, il faut une poignée : une `GlobalKey<FormState>`.

```dart
final GlobalKey<FormState> _formKey = GlobalKey<FormState>();
```

Une `GlobalKey` est une clé unique dans toute l'application. Attachée à un `Form`, elle donne accès à son état via `_formKey.currentState`, de type `FormState?`, qui expose :

| Méthode | Rôle |
| --- | --- |
| `validate()` | exécute tous les `validator` ; retourne `true` si tout est valide |
| `save()` | exécute tous les `onSaved` |
| `reset()` | remet tous les champs à leur valeur initiale et efface les erreurs |

**Règle absolue :** la `GlobalKey` se déclare **comme champ `final` du `State`**, jamais dans `build`.

```dart
// FAUX : une nouvelle clé à chaque reconstruction.
@override
Widget build(BuildContext context) {
  final GlobalKey<FormState> formKey = GlobalKey<FormState>();
  ...
}
```

Créée dans `build`, la clé est neuve à chaque frame. `currentState` vaut alors `null` juste après, les erreurs affichées disparaissent, et `validate()` plante avec un `Null check operator used on a null value`.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageForm(),
    );
  }
}

class PageForm extends StatefulWidget {
  const PageForm({super.key});

  @override
  State<PageForm> createState() => _PageFormState();
}

class _PageFormState extends State<PageForm> {
  // La clé : final, dans le State, une seule fois.
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();
  String _message = 'Formulaire non soumis.';

  void _soumettre() {
    final bool valide = _formKey.currentState!.validate();
    setState(() {
      _message = valide
          ? 'Formulaire valide, envoi en cours...'
          : 'Formulaire invalide, corrigez les champs.';
    });
  }

  void _reinitialiser() {
    _formKey.currentState!.reset();
    setState(() => _message = 'Formulaire réinitialisé.');
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Form')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Form(
          key: _formKey, // <-- rattachement de la clé
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: <Widget>[
              TextFormField(
                decoration: const InputDecoration(
                  labelText: 'Nom du héros',
                  border: OutlineInputBorder(),
                ),
                validator: (String? valeur) {
                  if (valeur == null || valeur.trim().isEmpty) {
                    return 'Le nom est obligatoire.';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 16),
              TextFormField(
                decoration: const InputDecoration(
                  labelText: 'Classe',
                  border: OutlineInputBorder(),
                ),
                validator: (String? valeur) {
                  if (valeur == null || valeur.trim().isEmpty) {
                    return 'La classe est obligatoire.';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 24),
              FilledButton(
                onPressed: _soumettre,
                child: const Text('Créer le personnage'),
              ),
              const SizedBox(height: 8),
              OutlinedButton(
                onPressed: _reinitialiser,
                child: const Text('Réinitialiser'),
              ),
              const SizedBox(height: 16),
              Text(_message, textAlign: TextAlign.center),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
Appui sur « Créer le personnage » avec les deux champs vides :

┌─ Nom du héros ──────────────┐   (bordure rouge)
└─────────────────────────────┘
Le nom est obligatoire.
┌─ Classe ────────────────────┐   (bordure rouge)
└─────────────────────────────┘
La classe est obligatoire.

Formulaire invalide, corrigez les champs.

Après avoir rempli les deux champs et réappuyé :

Formulaire valide, envoi en cours...
```

---

## 49.25 — `TextFormField`

`TextFormField` est un `TextField` qui **sait dialoguer avec un `Form`**. Il accepte tous les paramètres de `TextField` (`controller`, `decoration`, `keyboardType`, `obscureText`...) plus quatre paramètres propres à la validation :

| Paramètre | Rôle |
| --- | --- |
| `validator` | fonction qui retourne `null` (valide) ou un message d'erreur |
| `onSaved` | appelé par `formKey.currentState!.save()` |
| `initialValue` | valeur de départ, **si et seulement si** aucun `controller` |
| `autovalidateMode` | quand valider ce champ précisément |

```text
   TextField                    TextFormField
      │                              │
      │  saisie                      │  saisie
      │                              │
      └── vous gérez tout            ├── validator     -> errorText automatique
                                     ├── onSaved       -> collecte des valeurs
                                     └── s'inscrit auprès du Form parent
```

**Règle :** dès qu'il y a un `Form`, utilisez `TextFormField`. Un `TextField` placé dans un `Form` fonctionne, mais il est **ignoré** par `validate()` et `save()` : c'est une source de bugs silencieux.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageTextFormField(),
    );
  }
}

class PageTextFormField extends StatefulWidget {
  const PageTextFormField({super.key});

  @override
  State<PageTextFormField> createState() => _PageTextFormFieldState();
}

class _PageTextFormFieldState extends State<PageTextFormField> {
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();
  final TextEditingController _nom = TextEditingController();

  @override
  void dispose() {
    _nom.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('TextFormField')),
      body: Form(
        key: _formKey,
        child: ListView(
          padding: const EdgeInsets.all(20),
          children: <Widget>[
            // Version avec contrôleur : on lit _nom.text à tout moment.
            TextFormField(
              controller: _nom,
              decoration: const InputDecoration(
                labelText: 'Nom (avec contrôleur)',
                border: OutlineInputBorder(),
              ),
              validator: (String? v) =>
                  (v == null || v.isEmpty) ? 'Obligatoire.' : null,
            ),
            const SizedBox(height: 16),

            // Version avec initialValue : PAS de contrôleur.
            TextFormField(
              initialValue: 'Guerrier',
              decoration: const InputDecoration(
                labelText: 'Classe (avec initialValue)',
                border: OutlineInputBorder(),
              ),
              validator: (String? v) =>
                  (v == null || v.isEmpty) ? 'Obligatoire.' : null,
            ),
            const SizedBox(height: 16),

            // Un TextField ordinaire dans un Form : il ne sera JAMAIS validé.
            const TextField(
              decoration: InputDecoration(
                labelText: 'TextField ordinaire (non validé)',
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 24),

            FilledButton(
              onPressed: () {
                if (_formKey.currentState!.validate()) {
                  ScaffoldMessenger.of(context).showSnackBar(
                    SnackBar(content: Text('Nom saisi : ${_nom.text}')),
                  );
                }
              },
              child: const Text('Valider'),
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
Appui sur « Valider » avec le premier champ vide :

┌─ Nom (avec contrôleur) ──────────────┐   rouge
└──────────────────────────────────────┘
Obligatoire.
┌─ Classe (avec initialValue) ─────────┐
│ Guerrier                             │   valide
└──────────────────────────────────────┘
┌─ TextField ordinaire (non validé) ───┐
└──────────────────────────────────────┘   jamais d'erreur

Après avoir rempli le nom : SnackBar « Nom saisi : Kael »
```

**Erreur fréquente :** donner à la fois `controller:` et `initialValue:` déclenche l'assertion suivante.

```text
'package:flutter/src/material/text_form_field.dart':
Failed assertion: initialValue == null || controller == null
```

Pour pré-remplir un champ **avec** contrôleur, passez la valeur au contrôleur : `TextEditingController(text: 'Guerrier')`.

---

## 49.26 — `validator`

Le `validator` est le cœur de la validation. Sa signature est :

```dart
String? Function(String? valeur)
```

- Le paramètre est le contenu du champ, **nullable** (d'où le `?` du chapitre 12).
- Le retour est `null` **si tout va bien**, ou une `String` contenant le message d'erreur.

> **La règle mnémotechnique : `null` = « rien à signaler » = valide.**

C'est contre-intuitif au début : on s'attend à retourner `true`. Retenez que la valeur retournée est **le message d'erreur** ; s'il n'y a pas de message, il n'y a pas d'erreur.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageValidator(),
    );
  }
}

class PageValidator extends StatefulWidget {
  const PageValidator({super.key});

  @override
  State<PageValidator> createState() => _PageValidatorState();
}

class _PageValidatorState extends State<PageValidator> {
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('validator')),
      body: Form(
        key: _formKey,
        child: ListView(
          padding: const EdgeInsets.all(20),
          children: <Widget>[
            // Validateur à une seule condition.
            TextFormField(
              decoration: const InputDecoration(
                labelText: 'Nom du héros',
                border: OutlineInputBorder(),
              ),
              validator: (String? valeur) {
                if (valeur == null || valeur.trim().isEmpty) {
                  return 'Le nom est obligatoire.';
                }
                return null; // valide
              },
            ),
            const SizedBox(height: 16),

            // Validateur à conditions multiples, du plus général au plus fin.
            TextFormField(
              decoration: const InputDecoration(
                labelText: 'Cri de guerre',
                border: OutlineInputBorder(),
              ),
              validator: (String? valeur) {
                final String v = (valeur ?? '').trim();
                if (v.isEmpty) {
                  return 'Un héros doit avoir un cri de guerre.';
                }
                if (v.length < 4) {
                  return 'Trop court : 4 caractères minimum '
                      '(actuellement ${v.length}).';
                }
                if (v.length > 30) {
                  return 'Trop long : 30 caractères maximum.';
                }
                if (v == v.toLowerCase()) {
                  return 'Un cri de guerre contient au moins une majuscule.';
                }
                return null;
              },
            ),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: () => _formKey.currentState!.validate(),
              child: const Text('Vérifier'),
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
Saisie « ah » :
  Trop court : 4 caractères minimum (actuellement 2).

Saisie « pour la gloire » :
  Un cri de guerre contient au moins une majuscule.

Saisie « Pour la gloire » :
  (aucune erreur, bordure normale)
```

**Les cinq règles d'un bon validateur.**

1. **Tester `null` en premier.** Le paramètre est `String?`. L'idiome le plus court est `final String v = (valeur ?? '').trim();`.
2. **Appliquer `.trim()`.** Sinon un champ contenant trois espaces passe pour rempli.
3. **Ordonner du plus général au plus précis.** Vide, puis longueur, puis format. Un seul message s'affiche : le premier retourné.
4. **Écrire un message actionnable.** « Invalide » n'aide personne. « Trop court : 4 caractères minimum » indique quoi faire.
5. **Ne jamais appeler `setState` dans un validateur.** Un validateur s'exécute pendant la construction de l'arbre ; un `setState` y provoque l'exception `setState() or markNeedsBuild() called during build`.

---

## 49.27 — `formKey.currentState!.validate()`

`validate()` fait exactement trois choses :

```text
   formKey.currentState!.validate()
        │
        ├── 1. appelle le validator de CHAQUE TextFormField du Form
        ├── 2. affiche sous chaque champ le message retourné (ou l'efface)
        └── 3. retourne true si TOUS ont retourné null, false sinon
```

Le motif de soumission standard est donc :

```dart
void _soumettre() {
  if (!_formKey.currentState!.validate()) {
    return;              // au moins un champ est invalide : on s'arrête
  }
  // Ici, et seulement ici, toutes les données sont valides.
  _envoyerAuServeur();
}
```

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageValidate(),
    );
  }
}

class PageValidate extends StatefulWidget {
  const PageValidate({super.key});

  @override
  State<PageValidate> createState() => _PageValidateState();
}

class _PageValidateState extends State<PageValidate> {
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();
  final TextEditingController _nom = TextEditingController();
  final TextEditingController _niveau = TextEditingController();

  int _tentatives = 0;
  int _succes = 0;

  @override
  void dispose() {
    _nom.dispose();
    _niveau.dispose();
    super.dispose();
  }

  void _soumettre() {
    setState(() => _tentatives += 1);

    // 1. Validation.
    final FormState? etat = _formKey.currentState;
    if (etat == null) {
      return; // sécurité : le Form n'est pas monté
    }
    if (!etat.validate()) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(
          content: Text('Corrigez les erreurs avant de continuer.'),
          backgroundColor: Colors.red,
        ),
      );
      return;
    }

    // 2. Traitement, uniquement si tout est valide.
    setState(() => _succes += 1);
    FocusScope.of(context).unfocus();
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text('${_nom.text} (niveau ${_niveau.text}) créé.'),
        backgroundColor: Colors.green,
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('validate()')),
      body: Form(
        key: _formKey,
        child: ListView(
          padding: const EdgeInsets.all(20),
          children: <Widget>[
            TextFormField(
              controller: _nom,
              decoration: const InputDecoration(
                labelText: 'Nom',
                border: OutlineInputBorder(),
              ),
              validator: (String? v) =>
                  (v == null || v.trim().isEmpty) ? 'Nom obligatoire.' : null,
            ),
            const SizedBox(height: 16),
            TextFormField(
              controller: _niveau,
              keyboardType: TextInputType.number,
              decoration: const InputDecoration(
                labelText: 'Niveau (1 à 99)',
                border: OutlineInputBorder(),
              ),
              validator: (String? v) {
                final int? n = int.tryParse((v ?? '').trim());
                if (n == null) {
                  return 'Entrez un nombre entier.';
                }
                if (n < 1 || n > 99) {
                  return 'Le niveau doit être entre 1 et 99.';
                }
                return null;
              },
            ),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: _soumettre,
              child: const Text('Créer'),
            ),
            const SizedBox(height: 16),
            Text('Tentatives : $_tentatives — Succès : $_succes'),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
Champs vides, appui sur « Créer » :
  Nom obligatoire.
  Entrez un nombre entier.
  SnackBar rouge : « Corrigez les erreurs avant de continuer. »
  Tentatives : 1 — Succès : 0

Nom = Kael, Niveau = 150 :
  Le niveau doit être entre 1 et 99.
  Tentatives : 2 — Succès : 0

Nom = Kael, Niveau = 12 :
  SnackBar verte : « Kael (niveau 12) créé. »
  Tentatives : 3 — Succès : 1
```

**À propos du `!`.** `currentState` est de type `FormState?`. Le `!` affirme « je sais qu'il n'est pas nul ». C'est vrai **tant que la clé est bien attachée à un `Form` monté**. Si vous appelez `validate()` depuis un `initState`, avant le premier `build`, `currentState` vaut `null` et le `!` fait planter l'application. La version défensive (`final FormState? etat = ...; if (etat == null) return;`) évite ce risque.

**Erreur fréquente :** appeler `validate()` sans qu'aucun `Form` n'existe. Vous obtenez :

```text
Null check operator used on a null value
```

Vérifiez alors trois points : le `Form` existe-t-il, la clé lui est-elle passée via `key:`, et la clé est-elle bien un champ du `State` et non une variable locale de `build` ?

---

## 49.28 — `autovalidateMode`

Par défaut, les erreurs n'apparaissent qu'après un appel à `validate()`. Ce n'est pas toujours l'expérience souhaitée.

`autovalidateMode` existe sur le `Form` (pour tous les champs) et sur chaque `TextFormField` (pour lui seul). Ses valeurs :

| Valeur | Comportement |
| --- | --- |
| `AutovalidateMode.disabled` | valeur par défaut : validation uniquement sur appel explicite |
| `AutovalidateMode.always` | valide en permanence, y compris avant toute saisie |
| `AutovalidateMode.onUserInteraction` | valide dès que l'utilisateur a modifié le champ |
| `AutovalidateMode.onUnfocus` | valide quand le champ perd le focus |
| `AutovalidateMode.onUserInteractionIfError` | valide à chaque frappe, mais seulement si le champ est déjà en erreur |

```text
   always                 « Le nom est obligatoire » s'affiche
                          AVANT que l'utilisateur ait touché à quoi que ce soit.
                          -> agressif, à éviter sur un formulaire de création.

   onUserInteraction      Rien au départ. Dès la première frappe, le champ
                          se valide à chaque caractère.
                          -> le meilleur compromis dans la plupart des cas.

   onUnfocus              Rien pendant la frappe. L'erreur apparaît quand
                          l'utilisateur passe au champ suivant.
                          -> excellent pour les e-mails et les mots de passe.

   disabled               Rien tant que l'utilisateur n'a pas appuyé
                          sur « Valider ».
                          -> correct, mais l'utilisateur découvre tout d'un coup.
```

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageAutovalidate(),
    );
  }
}

class PageAutovalidate extends StatefulWidget {
  const PageAutovalidate({super.key});

  @override
  State<PageAutovalidate> createState() => _PageAutovalidateState();
}

class _PageAutovalidateState extends State<PageAutovalidate> {
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();

  // On passe de disabled à onUserInteraction après la première tentative.
  AutovalidateMode _mode = AutovalidateMode.disabled;

  String? _valider(String? v) {
    final String t = (v ?? '').trim();
    if (t.isEmpty) {
      return 'Champ obligatoire.';
    }
    if (t.length < 3) {
      return 'Au moins 3 caractères.';
    }
    return null;
  }

  void _soumettre() {
    if (!_formKey.currentState!.validate()) {
      // Après un échec, on valide en direct pour guider l'utilisateur.
      setState(() => _mode = AutovalidateMode.onUserInteraction);
      return;
    }
    ScaffoldMessenger.of(context).showSnackBar(
      const SnackBar(content: Text('Tout est valide.')),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('autovalidateMode')),
      body: Form(
        key: _formKey,
        autovalidateMode: _mode,
        child: ListView(
          padding: const EdgeInsets.all(20),
          children: <Widget>[
            Text('Mode courant : ${_mode.name}'),
            const SizedBox(height: 16),
            TextFormField(
              decoration: const InputDecoration(
                labelText: 'Nom (mode du Form)',
                border: OutlineInputBorder(),
              ),
              validator: _valider,
            ),
            const SizedBox(height: 16),
            TextFormField(
              // Ce champ impose son propre mode, indépendant du Form.
              autovalidateMode: AutovalidateMode.onUserInteraction,
              decoration: const InputDecoration(
                labelText: 'Surnom (toujours onUserInteraction)',
                border: OutlineInputBorder(),
              ),
              validator: _valider,
            ),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: _soumettre,
              child: const Text('Valider'),
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
État initial (mode : disabled) :
  Les deux champs sont vides, aucune erreur affichée.

Premier caractère tapé dans « Surnom » :
  Surnom -> « Au moins 3 caractères. »   (mode propre au champ)
  Nom    -> rien

Appui sur « Valider » avec le nom vide :
  Nom -> « Champ obligatoire. »
  Mode courant : onUserInteraction
  Désormais, le champ Nom se valide aussi à chaque frappe.
```

**Recommandation.** Le motif ci-dessus — `disabled` au départ, `onUserInteraction` après le premier échec — est celui qui donne la meilleure expérience. L'utilisateur n'est pas agressé de rouge en arrivant, puis il est guidé en direct dès qu'il a fait une erreur.

---

## 49.29 — `onSaved` et `save()`

`validate()` vérifie. `save()` **collecte**.

Quand vous appelez `_formKey.currentState!.save()`, Flutter parcourt tous les `TextFormField` du `Form` et appelle leur `onSaved` avec la valeur courante. C'est la manière de remplir un objet sans déclarer un contrôleur par champ.

```text
   MÉTHODE CONTRÔLEURS              MÉTHODE onSaved

   3 contrôleurs à créer            0 contrôleur
   3 dispose() à écrire             0 dispose()
   lecture : _nom.text              lecture : dans onSaved
   écriture possible                écriture impossible
```

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageOnSaved(),
    );
  }
}

/// Modèle du chapitre 09 : constructeur à paramètres nommés.
class Personnage {
  Personnage({required this.nom, required this.classe, required this.niveau});

  final String nom;
  final String classe;
  final int niveau;

  @override
  String toString() => '$nom, $classe de niveau $niveau';
}

class PageOnSaved extends StatefulWidget {
  const PageOnSaved({super.key});

  @override
  State<PageOnSaved> createState() => _PageOnSavedState();
}

class _PageOnSavedState extends State<PageOnSaved> {
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();

  // Tampons remplis par les onSaved.
  String _nom = '';
  String _classe = '';
  int _niveau = 1;

  Personnage? _cree;

  void _soumettre() {
    final FormState etat = _formKey.currentState!;
    if (!etat.validate()) {
      return;
    }
    etat.save(); // <-- déclenche TOUS les onSaved
    setState(() {
      _cree = Personnage(nom: _nom, classe: _classe, niveau: _niveau);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('onSaved et save()')),
      body: Form(
        key: _formKey,
        child: ListView(
          padding: const EdgeInsets.all(20),
          children: <Widget>[
            TextFormField(
              decoration: const InputDecoration(
                labelText: 'Nom',
                border: OutlineInputBorder(),
              ),
              validator: (String? v) =>
                  (v == null || v.trim().isEmpty) ? 'Obligatoire.' : null,
              onSaved: (String? v) => _nom = (v ?? '').trim(),
            ),
            const SizedBox(height: 16),
            TextFormField(
              initialValue: 'Guerrier',
              decoration: const InputDecoration(
                labelText: 'Classe',
                border: OutlineInputBorder(),
              ),
              validator: (String? v) =>
                  (v == null || v.trim().isEmpty) ? 'Obligatoire.' : null,
              onSaved: (String? v) => _classe = (v ?? '').trim(),
            ),
            const SizedBox(height: 16),
            TextFormField(
              initialValue: '1',
              keyboardType: TextInputType.number,
              decoration: const InputDecoration(
                labelText: 'Niveau',
                border: OutlineInputBorder(),
              ),
              validator: (String? v) =>
                  int.tryParse((v ?? '').trim()) == null
                      ? 'Nombre entier attendu.'
                      : null,
              onSaved: (String? v) => _niveau = int.parse((v ?? '1').trim()),
            ),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: _soumettre,
              child: const Text('Créer'),
            ),
            const SizedBox(height: 24),
            Text(
              _cree == null
                  ? 'Aucun personnage créé.'
                  : 'Créé : $_cree',
              style: const TextStyle(fontSize: 18),
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
Nom = Kael, Classe = Mage, Niveau = 12, appui sur « Créer » :

Créé : Kael, Mage de niveau 12
```

**Trois points de vigilance.**

1. **`save()` n'appelle pas `validate()`.** Toujours dans cet ordre : `validate()` d'abord, `save()` ensuite. Sinon vous sauvegardez des données invalides.
2. **Dans `onSaved`, `int.parse` est sans danger** : le `validator` a déjà garanti que la conversion est possible. Ailleurs, utilisez `int.tryParse` (section 49.31).
3. **`onSaved` ne doit pas contenir de `setState`.** Faites le `setState` **après** `save()`, comme ci-dessus.

---

## 49.30 — Écrire des validateurs réutilisables

Recopier « si vide, retourner Obligatoire » dans dix champs est une erreur de conception. La bonne pratique est une **classe utilitaire de validateurs**.

Un validateur réutilisable est une fonction qui **retourne un validateur** : c'est un usage direct des fonctions d'ordre supérieur du chapitre 07.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

/// Bibliothèque de validateurs réutilisables.
class Valideurs {
  const Valideurs._(); // classe purement statique

  static String? Function(String?) requis([String message = 'Champ obligatoire.']) {
    return (String? v) => (v == null || v.trim().isEmpty) ? message : null;
  }

  static String? Function(String?) longueurMin(int n) {
    return (String? v) {
      final String t = (v ?? '').trim();
      if (t.isEmpty) {
        return null; // le contrôle « vide » appartient à `requis`
      }
      return t.length < n ? 'Au moins $n caractères.' : null;
    };
  }

  static String? Function(String?) longueurMax(int n) {
    return (String? v) =>
        (v ?? '').trim().length > n ? 'Au plus $n caractères.' : null;
  }

  static String? email(String? v) {
    final String t = (v ?? '').trim();
    if (t.isEmpty) {
      return null;
    }
    final RegExp motif = RegExp(r'^[\w.+-]+@[\w-]+\.[\w.-]{2,}$');
    return motif.hasMatch(t) ? null : 'Adresse e-mail invalide.';
  }

  static String? Function(String?) entierEntre(int min, int max) {
    return (String? v) {
      final String t = (v ?? '').trim();
      if (t.isEmpty) {
        return null;
      }
      final int? n = int.tryParse(t);
      if (n == null) {
        return 'Entrez un nombre entier.';
      }
      return (n < min || n > max) ? 'Valeur entre $min et $max.' : null;
    };
  }

  /// Combine plusieurs validateurs : retourne la PREMIÈRE erreur trouvée.
  static String? Function(String?) tous(
    List<String? Function(String?)> validateurs,
  ) {
    return (String? v) {
      for (final String? Function(String?) validateur in validateurs) {
        final String? erreur = validateur(v);
        if (erreur != null) {
          return erreur;
        }
      }
      return null;
    };
  }
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageValideurs(),
    );
  }
}

class PageValideurs extends StatefulWidget {
  const PageValideurs({super.key});

  @override
  State<PageValideurs> createState() => _PageValideursState();
}

class _PageValideursState extends State<PageValideurs> {
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Validateurs réutilisables')),
      body: Form(
        key: _formKey,
        autovalidateMode: AutovalidateMode.onUserInteraction,
        child: ListView(
          padding: const EdgeInsets.all(20),
          children: <Widget>[
            TextFormField(
              decoration: const InputDecoration(
                labelText: 'Nom du héros',
                border: OutlineInputBorder(),
              ),
              validator: Valideurs.tous(<String? Function(String?)>[
                Valideurs.requis('Le nom est obligatoire.'),
                Valideurs.longueurMin(3),
                Valideurs.longueurMax(20),
              ]),
            ),
            const SizedBox(height: 16),
            TextFormField(
              keyboardType: TextInputType.emailAddress,
              decoration: const InputDecoration(
                labelText: 'E-mail du joueur',
                border: OutlineInputBorder(),
              ),
              validator: Valideurs.tous(<String? Function(String?)>[
                Valideurs.requis(),
                Valideurs.email,
              ]),
            ),
            const SizedBox(height: 16),
            TextFormField(
              keyboardType: TextInputType.number,
              decoration: const InputDecoration(
                labelText: 'Niveau (1 à 99)',
                border: OutlineInputBorder(),
              ),
              validator: Valideurs.tous(<String? Function(String?)>[
                Valideurs.requis(),
                Valideurs.entierEntre(1, 99),
              ]),
            ),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: () {
                if (_formKey.currentState!.validate()) {
                  ScaffoldMessenger.of(context).showSnackBar(
                    const SnackBar(content: Text('Tout est valide.')),
                  );
                }
              },
              child: const Text('Valider'),
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
Nom = « Ka »              -> Au moins 3 caractères.
E-mail = « kael@ »        -> Adresse e-mail invalide.
Niveau = « abc »          -> Entrez un nombre entier.
Niveau = « 150 »          -> Valeur entre 1 et 99.

Nom = Kael, e-mail = kael@jeu.fr, niveau = 12
                          -> SnackBar « Tout est valide. »
```

**Le principe de séparation.** Chaque validateur teste **une seule chose** et ignore le cas « vide » (sauf `requis`). C'est ce qui permet de composer : un champ facultatif utilise `Valideurs.email` seul, un champ obligatoire utilise `tous([requis(), email])`.

Dans un vrai projet, ce fichier s'appelle `lib/utils/valideurs.dart` (organisation du chapitre 16).

**Note sur l'expression régulière.** Aucune expression régulière ne valide parfaitement une adresse e-mail. Celle-ci attrape 99 % des fautes de frappe, ce qui est le but. La seule validation certaine reste l'envoi d'un e-mail de confirmation.

---

## 49.31 — Valider avec `int.tryParse`

Au chapitre 13, vous avez appris la différence entre `int.parse` et `int.tryParse` :

```dart
int.parse('12')      // 12
int.parse('abc')     // lève FormatException  <- plante l'application
int.tryParse('12')   // 12
int.tryParse('abc')  // null                  <- ne plante jamais
```

Dans un validateur, **`int.parse` est interdit**. Le champ contient ce que l'utilisateur a tapé, donc potentiellement n'importe quoi. Une exception non rattrapée pendant la validation casse l'écran.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageTryParse(),
    );
  }
}

class PageTryParse extends StatefulWidget {
  const PageTryParse({super.key});

  @override
  State<PageTryParse> createState() => _PageTryParseState();
}

class _PageTryParseState extends State<PageTryParse> {
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();
  int _pointsDeVie = 0;
  double _multiplicateur = 1.0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('int.tryParse')),
      body: Form(
        key: _formKey,
        autovalidateMode: AutovalidateMode.onUserInteraction,
        child: ListView(
          padding: const EdgeInsets.all(20),
          children: <Widget>[
            TextFormField(
              keyboardType: TextInputType.number,
              decoration: const InputDecoration(
                labelText: 'Points de vie (10 à 999)',
                border: OutlineInputBorder(),
              ),
              validator: (String? v) {
                final String t = (v ?? '').trim();
                if (t.isEmpty) {
                  return 'Champ obligatoire.';
                }
                final int? pv = int.tryParse(t);
                if (pv == null) {
                  return 'Ce n\'est pas un nombre entier.';
                }
                if (pv < 10) {
                  return 'Minimum 10 points de vie.';
                }
                if (pv > 999) {
                  return 'Maximum 999 points de vie.';
                }
                return null;
              },
              onSaved: (String? v) => _pointsDeVie = int.parse(v!.trim()),
            ),
            const SizedBox(height: 16),
            TextFormField(
              keyboardType:
                  const TextInputType.numberWithOptions(decimal: true),
              decoration: const InputDecoration(
                labelText: 'Multiplicateur de dégâts (0.5 à 3.0)',
                helperText: 'Utilisez un point, pas une virgule.',
                border: OutlineInputBorder(),
              ),
              validator: (String? v) {
                // La virgule française est acceptée puis convertie.
                final String t = (v ?? '').trim().replaceAll(',', '.');
                if (t.isEmpty) {
                  return 'Champ obligatoire.';
                }
                final double? m = double.tryParse(t);
                if (m == null) {
                  return 'Ce n\'est pas un nombre décimal.';
                }
                if (m < 0.5 || m > 3.0) {
                  return 'Entre 0.5 et 3.0.';
                }
                return null;
              },
              onSaved: (String? v) => _multiplicateur =
                  double.parse(v!.trim().replaceAll(',', '.')),
            ),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: () {
                final FormState etat = _formKey.currentState!;
                if (!etat.validate()) {
                  return;
                }
                etat.save();
                final int degats = (_pointsDeVie * _multiplicateur).round();
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(content: Text('Dégâts maximaux : $degats')),
                );
              },
              child: const Text('Calculer'),
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
PV = « cent »        -> Ce n'est pas un nombre entier.
PV = « 5 »           -> Minimum 10 points de vie.
Multiplicateur = « 5 » -> Entre 0.5 et 3.0.

PV = 200, multiplicateur = 1,5  -> SnackBar « Dégâts maximaux : 300 »
```

**Quatre remarques.**

1. **`int.tryParse` refuse les décimaux.** `int.tryParse('1.5')` retourne `null`. Pour un décimal, utilisez `double.tryParse`.
2. **La virgule française.** `double.tryParse('1,5')` retourne `null`. Le `replaceAll(',', '.')` ci-dessus règle le problème pour vos utilisateurs francophones.
3. **`int.tryParse` accepte les espaces de tête et de fin** mais pas les espaces internes, ni les séparateurs de milliers.
4. **Dans `onSaved`, `int.parse` est acceptable** puisque le validateur a déjà écarté les cas impossibles. En cas de doute, restez sur `int.tryParse(...) ?? 0`.

---

## 49.32 — `Checkbox` et `CheckboxListTile`

Une case à cocher représente un booléen. Le widget **ne mémorise rien** : c'est vous qui détenez la valeur dans le `State` et qui la mettez à jour dans `onChanged`.

```text
   ┌───────────────────────────────────────────┐
   │  Checkbox                                 │
   │    value:     ce que J'AFFICHE            │  <- vient de votre State
   │    onChanged: ce qu'on me DEMANDE         │  <- vous appelez setState
   └───────────────────────────────────────────┘
```

`Checkbox` a une particularité : `onChanged` reçoit un `bool?` et non un `bool`, car le widget gère un troisième état (`tristate: true`) où la valeur peut être `null`.

`CheckboxListTile` combine la case et un `ListTile` : titre, sous-titre, et **toute la ligne devient cliquable**. C'est presque toujours le bon choix dans un formulaire.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageCheckbox(),
    );
  }
}

class PageCheckbox extends StatefulWidget {
  const PageCheckbox({super.key});

  @override
  State<PageCheckbox> createState() => _PageCheckboxState();
}

class _PageCheckboxState extends State<PageCheckbox> {
  bool _conditions = false;

  // Une Map<String, bool> gère élégamment un groupe de cases.
  final Map<String, bool> _competences = <String, bool>{
    'Force': true,
    'Agilité': false,
    'Intelligence': false,
    'Endurance': false,
  };

  int get _nbChoisies =>
      _competences.values.where((bool coche) => coche).length;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Checkbox')),
      body: ListView(
        children: <Widget>[
          // 1. Checkbox nue, dans une Row construite à la main.
          Padding(
            padding: const EdgeInsets.all(16),
            child: Row(
              children: <Widget>[
                Checkbox(
                  value: _conditions,
                  onChanged: (bool? v) {
                    setState(() => _conditions = v ?? false);
                  },
                ),
                const Expanded(child: Text("J'accepte les règles du jeu")),
              ],
            ),
          ),
          const Divider(),

          // 2. CheckboxListTile : titre, sous-titre, ligne entière cliquable.
          ..._competences.keys.map((String nom) {
            return CheckboxListTile(
              value: _competences[nom],
              title: Text(nom),
              subtitle: Text('Spécialisation en $nom'),
              secondary: const Icon(Icons.star_outline),
              controlAffinity: ListTileControlAffinity.leading,
              onChanged: (bool? v) {
                setState(() => _competences[nom] = v ?? false);
              },
            );
          }),

          const Divider(),
          Padding(
            padding: const EdgeInsets.all(16),
            child: Text(
              'Compétences choisies : $_nbChoisies / 4\n'
              'Règles acceptées : ${_conditions ? "oui" : "non"}',
            ),
          ),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
  [ ]  J'accepte les règles du jeu
  ─────────────────────────────────────────
  [x]  ★ Force
        Spécialisation en Force
  [ ]  ★ Agilité
        Spécialisation en Agilité
  [ ]  ★ Intelligence
        Spécialisation en Intelligence
  [ ]  ★ Endurance
        Spécialisation en Endurance
  ─────────────────────────────────────────
  Compétences choisies : 1 / 4
  Règles acceptées : non
```

**Les deux erreurs classiques.**

1. **Oublier `setState`.** Vous cochez, rien ne bouge. La valeur a changé en mémoire mais l'écran affiche toujours l'ancienne. C'est l'erreur numéro un des débutants sur ce widget.

   ```dart
   // FAUX : la case ne se coche jamais visuellement.
   onChanged: (bool? v) => _conditions = v ?? false,
   ```

2. **Passer `onChanged: null`.** Cela **désactive** la case (grisée). Pour une case en lecture seule mais lisible, c'est correct ; sinon c'est un bug.

**`controlAffinity`** place la case : `.leading` (à gauche), `.trailing` (à droite, par défaut), `.platform`.

---

## 49.33 — `Switch`

Le `Switch` est un interrupteur. Techniquement identique à la `Checkbox` (un booléen), il s'en distingue par l'**usage** :

| Widget | Usage | Effet |
| --- | --- | --- |
| `Checkbox` | choix dans un formulaire | prend effet à la validation |
| `Switch` | réglage, préférence | prend effet **immédiatement** |

Différence importante : `Switch.onChanged` reçoit un `bool` (non nullable), contrairement à `Checkbox`.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageSwitch(),
    );
  }
}

class PageSwitch extends StatefulWidget {
  const PageSwitch({super.key});

  @override
  State<PageSwitch> createState() => _PageSwitchState();
}

class _PageSwitchState extends State<PageSwitch> {
  bool _musique = true;
  bool _effets = true;
  bool _vibrations = false;
  bool _modeDifficile = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Options du jeu')),
      body: ListView(
        children: <Widget>[
          SwitchListTile(
            value: _musique,
            title: const Text('Musique'),
            subtitle: const Text('Bande sonore d\'ambiance'),
            secondary: Icon(_musique ? Icons.music_note : Icons.music_off),
            onChanged: (bool v) => setState(() => _musique = v),
          ),
          SwitchListTile(
            value: _effets,
            title: const Text('Effets sonores'),
            secondary: const Icon(Icons.graphic_eq),
            onChanged: (bool v) => setState(() => _effets = v),
          ),
          SwitchListTile(
            value: _vibrations,
            title: const Text('Vibrations'),
            secondary: const Icon(Icons.vibration),
            // Désactivé tant que les effets sonores sont coupés.
            onChanged: _effets
                ? (bool v) => setState(() => _vibrations = v)
                : null,
          ),
          const Divider(),
          SwitchListTile(
            value: _modeDifficile,
            title: const Text('Mode difficile'),
            subtitle: const Text('Les ennemis infligent le double de dégâts'),
            secondary: const Icon(Icons.local_fire_department),
            onChanged: (bool v) => setState(() => _modeDifficile = v),
          ),
          const Divider(),

          // Switch nu, hors d'un ListTile.
          Padding(
            padding: const EdgeInsets.all(16),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: <Widget>[
                const Text('Interrupteur simple'),
                Switch(
                  value: _modeDifficile,
                  onChanged: (bool v) => setState(() => _modeDifficile = v),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
  ♪  Musique                                  [ ●──]
       Bande sonore d'ambiance
  ≈  Effets sonores                           [ ●──]
  ≈  Vibrations                               [──● ]
  ───────────────────────────────────────────────────
  ▲  Mode difficile                           [──● ]
       Les ennemis infligent le double de dégâts

Si « Effets sonores » passe à OFF :
  ≈  Vibrations                     (grisé, non cliquable)
```

**`Switch.adaptive`** affiche l'interrupteur natif iOS sur iOS et macOS, et l'interrupteur Material ailleurs. Utile pour une application multiplateforme.

---

## 49.34 — `Radio` et `RadioListTile`

Les boutons radio servent à choisir **une seule** option parmi plusieurs. Contrairement aux cases à cocher, ils sont **exclusifs**.

Depuis Flutter 3.32, l'API a changé. La bonne pratique actuelle est d'envelopper le groupe dans un widget **`RadioGroup<T>`** qui porte `groupValue` et `onChanged`. Les paramètres `groupValue` et `onChanged` de `Radio` et `RadioListTile` sont **dépréciés**.

```text
   RadioGroup<String>            <- porte la valeur choisie + le callback
     groupValue: _classe
     onChanged: ...
       │
       ├── RadioListTile<String>(value: 'Guerrier')
       ├── RadioListTile<String>(value: 'Mage')
       └── RadioListTile<String>(value: 'Voleur')
              ^ chaque tuile ne porte QUE sa propre valeur
```

Le type générique `<T>` est le type des valeurs. `String` fonctionne, mais une `enum` (chapitre 11) est bien plus sûre.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

enum ClasseHeros {
  guerrier('Guerrier', Icons.shield),
  mage('Mage', Icons.auto_fix_high),
  voleur('Voleur', Icons.visibility_off);

  const ClasseHeros(this.libelle, this.icone);

  final String libelle;
  final IconData icone;
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageRadio(),
    );
  }
}

class PageRadio extends StatefulWidget {
  const PageRadio({super.key});

  @override
  State<PageRadio> createState() => _PageRadioState();
}

class _PageRadioState extends State<PageRadio> {
  ClasseHeros? _classe;
  String _difficulte = 'Normal';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Radio')),
      body: ListView(
        children: <Widget>[
          const Padding(
            padding: EdgeInsets.all(16),
            child: Text(
              'Classe du héros',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
          ),

          // 1. RadioListTile, la forme recommandée.
          RadioGroup<ClasseHeros>(
            groupValue: _classe,
            onChanged: (ClasseHeros? v) => setState(() => _classe = v),
            child: Column(
              children: ClasseHeros.values.map((ClasseHeros c) {
                return RadioListTile<ClasseHeros>(
                  value: c,
                  title: Text(c.libelle),
                  secondary: Icon(c.icone),
                );
              }).toList(),
            ),
          ),

          const Divider(),
          const Padding(
            padding: EdgeInsets.all(16),
            child: Text(
              'Difficulté',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
          ),

          // 2. Radio nu, disposé à la main dans une Row.
          RadioGroup<String>(
            groupValue: _difficulte,
            onChanged: (String? v) =>
                setState(() => _difficulte = v ?? 'Normal'),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: <String>['Facile', 'Normal', 'Cauchemar']
                  .map((String d) {
                return Column(
                  mainAxisSize: MainAxisSize.min,
                  children: <Widget>[
                    Radio<String>(value: d),
                    Text(d),
                  ],
                );
              }).toList(),
            ),
          ),

          const Divider(),
          Padding(
            padding: const EdgeInsets.all(16),
            child: Text(
              'Classe : ${_classe?.libelle ?? "non choisie"}\n'
              'Difficulté : $_difficulte',
              style: const TextStyle(fontSize: 16),
            ),
          ),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
Classe du héros
  ○  ◈  Guerrier
  ●  ✦  Mage
  ○  ◉  Voleur
  ─────────────────────────────────────
Difficulté
     ○           ●            ○
   Facile      Normal     Cauchemar
  ─────────────────────────────────────
Classe : Mage
Difficulté : Normal
```

**Trois points.**

1. **`_classe` est `ClasseHeros?`**, nullable, parce qu'au départ aucun choix n'est fait. Si vous voulez un choix par défaut, initialisez-le : `ClasseHeros _classe = ClasseHeros.guerrier;`.
2. **Le type générique doit être identique** partout : `RadioGroup<ClasseHeros>` et `RadioListTile<ClasseHeros>`. Un mélange ne compile pas.
3. **Combien d'options ?** Jusqu'à 5, des boutons radio. Au-delà, un `DropdownButton` (section 49.36) est plus lisible.

---

## 49.35 — `Slider`

Le `Slider` sert à choisir une valeur numérique **continue ou par paliers**. Il travaille en `double`.

| Paramètre | Rôle |
| --- | --- |
| `value` | la valeur affichée (obligatoire) |
| `onChanged` | appelé en continu pendant le glissement |
| `min` / `max` | les bornes ; par défaut 0.0 et 1.0 |
| `divisions` | nombre de paliers ; sans lui, le glissement est continu |
| `label` | l'étiquette affichée pendant le glissement (nécessite `divisions`) |
| `onChangeEnd` | appelé une seule fois, au relâchement |

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageSlider(),
    );
  }
}

class PageSlider extends StatefulWidget {
  const PageSlider({super.key});

  @override
  State<PageSlider> createState() => _PageSliderState();
}

class _PageSliderState extends State<PageSlider> {
  double _volume = 0.7;
  double _pointsDeVie = 100;
  double _difficulte = 2;
  String _dernierRelachement = '—';

  static const List<String> _libellesDifficulte = <String>[
    'Très facile', 'Facile', 'Normal', 'Difficile', 'Cauchemar',
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Slider')),
      body: ListView(
        padding: const EdgeInsets.all(20),
        children: <Widget>[
          // 1. Glissement continu, de 0.0 à 1.0.
          Text('Volume : ${(_volume * 100).round()} %'),
          Slider(
            value: _volume,
            onChanged: (double v) => setState(() => _volume = v),
            onChangeEnd: (double v) => setState(
              () => _dernierRelachement = 'volume ${(v * 100).round()} %',
            ),
          ),
          const SizedBox(height: 16),

          // 2. Bornes personnalisées et paliers.
          Text('Points de vie : ${_pointsDeVie.round()}'),
          Slider(
            value: _pointsDeVie,
            min: 50,
            max: 300,
            divisions: 25, // paliers de 10
            label: '${_pointsDeVie.round()} PV',
            onChanged: (double v) => setState(() => _pointsDeVie = v),
          ),
          const SizedBox(height: 16),

          // 3. Un Slider utilisé comme sélecteur d'index.
          Text('Difficulté : ${_libellesDifficulte[_difficulte.round()]}'),
          Slider(
            value: _difficulte,
            min: 0,
            max: 4,
            divisions: 4,
            label: _libellesDifficulte[_difficulte.round()],
            onChanged: (double v) => setState(() => _difficulte = v),
          ),
          const Divider(height: 40),
          Text('Dernier relâchement : $_dernierRelachement'),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
Volume : 70 %
──────────────●───────

Points de vie : 150
─────●────────────────      étiquette « 150 PV » pendant le glissement
     (paliers visibles de 10 en 10)

Difficulté : Normal
──────●───────────────

Dernier relâchement : volume 70 %
```

**Cinq pièges.**

1. **`value` doit toujours rester entre `min` et `max`**, sinon une assertion échoue. Attention si vous changez `max` dynamiquement.
2. **`divisions` doit être un entier strictement positif.** Pour des paliers de 10 entre 50 et 300, il y a `(300-50)/10 = 25` divisions.
3. **`label` ne s'affiche que si `divisions` est renseigné.**
4. **`onChanged` se déclenche des dizaines de fois par seconde.** N'y mettez jamais une écriture disque ou une requête réseau : utilisez `onChangeEnd`.
5. **Le `Slider` travaille en `double`.** Pour un entier, convertissez avec `.round()` à l'affichage et à l'usage.

**Variante :** `RangeSlider` permet de choisir un intervalle (deux poignées) avec un objet `RangeValues`.

---

## 49.36 — `DropdownButton` et `DropdownButtonFormField`

Le menu déroulant sert à choisir une valeur parmi **beaucoup** d'options, sans occuper de place à l'écran.

Il existe en deux versions :

| Widget | Usage |
| --- | --- |
| `DropdownButton<T>` | hors d'un formulaire ; vous gérez `value` et `onChanged` |
| `DropdownButtonFormField<T>` | dans un `Form` ; accepte `validator`, `onSaved`, `decoration` |

Chaque option est un `DropdownMenuItem<T>` qui associe une **valeur** (`value`) et un **affichage** (`child`).

> **Attention :** sur `DropdownButtonFormField`, le paramètre `value` est déprécié au profit de `initialValue`.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageDropdown(),
    );
  }
}

class PageDropdown extends StatefulWidget {
  const PageDropdown({super.key});

  @override
  State<PageDropdown> createState() => _PageDropdownState();
}

class _PageDropdownState extends State<PageDropdown> {
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();

  static const List<String> _royaumes = <String>[
    'Val-Sombre', 'Haute-Roche', 'Terres Brûlées', 'Cité des Brumes',
  ];
  static const List<String> _armes = <String>[
    'Épée longue', 'Arc court', 'Bâton runique', 'Dague empoisonnée',
  ];

  String _royaume = _royaumes.first;
  String? _arme; // null = rien de choisi

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('DropdownButton')),
      body: Form(
        key: _formKey,
        child: ListView(
          padding: const EdgeInsets.all(20),
          children: <Widget>[
            const Text('1. DropdownButton simple'),
            const SizedBox(height: 8),
            DropdownButton<String>(
              value: _royaume,
              isExpanded: true, // occupe toute la largeur
              icon: const Icon(Icons.expand_more),
              items: _royaumes.map((String r) {
                return DropdownMenuItem<String>(
                  value: r,
                  child: Text(r),
                );
              }).toList(),
              onChanged: (String? v) {
                setState(() => _royaume = v ?? _royaumes.first);
              },
            ),
            const SizedBox(height: 32),

            const Text('2. DropdownButtonFormField, validé'),
            const SizedBox(height: 8),
            DropdownButtonFormField<String>(
              initialValue: _arme, // et non « value: » (déprécié)
              decoration: const InputDecoration(
                labelText: 'Arme de départ',
                prefixIcon: Icon(Icons.hardware),
                border: OutlineInputBorder(),
              ),
              hint: const Text('Choisissez une arme'),
              items: _armes.map((String a) {
                return DropdownMenuItem<String>(
                  value: a,
                  child: Text(a),
                );
              }).toList(),
              onChanged: (String? v) => setState(() => _arme = v),
              validator: (String? v) =>
                  v == null ? 'Une arme est obligatoire.' : null,
            ),
            const SizedBox(height: 24),

            FilledButton(
              onPressed: () {
                if (_formKey.currentState!.validate()) {
                  ScaffoldMessenger.of(context).showSnackBar(
                    SnackBar(content: Text('$_royaume — $_arme')),
                  );
                }
              },
              child: const Text('Valider'),
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
1. DropdownButton simple
   Val-Sombre                                    ⌄

2. DropdownButtonFormField, validé
   ┌─ Arme de départ ────────────────────────┐
   │ ◈ Choisissez une arme               ⌄  │
   └─────────────────────────────────────────┘

Appui sur « Valider » sans arme choisie :
   Une arme est obligatoire.

Après avoir choisi « Arc court » :
   SnackBar « Val-Sombre — Arc court »
```

**Les trois erreurs qui font planter un `DropdownButton`.**

1. **`value` absent de la liste des `items`.** Assertion immédiate :

   ```text
   There should be exactly one item with [DropdownButton]'s value
   ```
   Cela arrive typiquement quand la liste d'options change et que la valeur choisie n'existe plus. Corrigez en remettant la valeur à `null` en même temps que la liste.

2. **Deux `DropdownMenuItem` avec la même `value`.** Même assertion : « exactly one item ».

3. **Oublier `isExpanded: true`** sur un texte long : le libellé déborde et provoque un `RenderFlex overflowed`.

**Alternative Material 3 :** `DropdownMenu<T>` combine un champ de saisie et un menu, avec filtrage au clavier. Il utilise une liste de `DropdownMenuEntry` plutôt que des `DropdownMenuItem`.

---

## 49.37 — `showDatePicker`

`showDatePicker` ouvre le calendrier Material. Sa signature l'essentiel :

```dart
Future<DateTime?> showDatePicker({
  required BuildContext context,
  required DateTime firstDate,
  required DateTime lastDate,
  DateTime? initialDate,
  String? helpText,
  String? cancelText,
  String? confirmText,
  DatePickerEntryMode initialEntryMode = DatePickerEntryMode.calendar,
  ...
})
```

C'est une fonction **asynchrone** (chapitre 15) : elle retourne un `Future<DateTime?>`. La valeur est `null` si l'utilisateur annule.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageDate(),
    );
  }
}

class PageDate extends StatefulWidget {
  const PageDate({super.key});

  @override
  State<PageDate> createState() => _PageDateState();
}

class _PageDateState extends State<PageDate> {
  DateTime? _dateNaissance;

  // Le contrôleur est un champ du State, jamais créé dans build (49.15).
  final TextEditingController _champDate = TextEditingController();

  @override
  void dispose() {
    _champDate.dispose();
    super.dispose();
  }

  String _formater(DateTime d) {
    final String jour = d.day.toString().padLeft(2, '0');
    final String mois = d.month.toString().padLeft(2, '0');
    return '$jour/$mois/${d.year}';
  }

  Future<void> _choisirDate() async {
    final DateTime maintenant = DateTime.now();
    final DateTime? choisie = await showDatePicker(
      context: context,
      initialDate: _dateNaissance ?? DateTime(maintenant.year - 20),
      firstDate: DateTime(1900),
      lastDate: maintenant,
      helpText: 'Date de naissance du joueur',
      cancelText: 'Annuler',
      confirmText: 'Valider',
    );

    // Après un await, le contexte peut ne plus être valide.
    if (!mounted || choisie == null) {
      return;
    }
    setState(() {
      _dateNaissance = choisie;
      _champDate.text = _formater(choisie);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('showDatePicker')),
      body: Padding(
        padding: const EdgeInsets.all(20),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            // Motif classique : un champ en lecture seule qui ouvre le
            // sélecteur. L'utilisateur ne peut pas saisir n'importe quoi.
            TextField(
              readOnly: true,
              controller: _champDate,
              onTap: _choisirDate,
              decoration: const InputDecoration(
                labelText: 'Date de naissance',
                hintText: 'jj/mm/aaaa',
                prefixIcon: Icon(Icons.cake),
                suffixIcon: Icon(Icons.calendar_month),
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 24),
            OutlinedButton.icon(
              onPressed: _choisirDate,
              icon: const Icon(Icons.event),
              label: const Text('Ouvrir le calendrier'),
            ),
            const SizedBox(height: 24),
            Text(
              _dateNaissance == null
                  ? 'Aucune date choisie.'
                  : 'Âge : ${DateTime.now().year - _dateNaissance!.year} ans',
              style: const TextStyle(fontSize: 18),
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
┌─ Date de naissance ──────────────┐
│ ◍ 14/03/2001                ▤  │
└──────────────────────────────────┘
     [ ▤ Ouvrir le calendrier ]

Âge : 25 ans

Boîte ouverte :
┌──────────────────────────────────┐
│  Date de naissance du joueur     │
│  mars 2001                    ⌄  │
│  L  M  M  J  V  S  D             │
│           1  2  3  4             │
│  5  6  7  8  9 10 11             │
│ 12 13 (14) 15 16 17 18           │
│  ...                             │
│           Annuler     Valider    │
└──────────────────────────────────┘
```

**Quatre points d'attention.**

1. **`if (!mounted) return;` après le `await`** est obligatoire. Sans lui, l'analyseur signale `use_build_context_synchronously` et l'application peut planter si l'écran est fermé pendant que la boîte est ouverte.
2. **`initialDate` doit être entre `firstDate` et `lastDate`**, sinon assertion.
3. **Le calendrier est en anglais par défaut.** Pour le français, ajoutez `flutter_localizations` et déclarez `locale: const Locale('fr', 'FR')` sur le `MaterialApp` (voir chapitre 51).
4. **Le formatage manuel** utilisé ici évite une dépendance. En vrai projet, préférez le paquet `intl` : `DateFormat('dd/MM/yyyy', 'fr_FR').format(date)`.

**Variantes :** `showDateRangePicker` pour un intervalle, et `selectableDayPredicate` pour interdire certains jours.

---

## 49.38 — `showTimePicker`

Même principe pour l'heure. La signature est :

```dart
Future<TimeOfDay?> showTimePicker({
  required BuildContext context,
  required TimeOfDay initialTime,
  TimePickerEntryMode initialEntryMode = TimePickerEntryMode.dial,
  String? helpText,
  ...
})
```

Le type retourné est `TimeOfDay`, qui contient seulement `hour` (0-23) et `minute` (0-59).

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageHeure(),
    );
  }
}

class PageHeure extends StatefulWidget {
  const PageHeure({super.key});

  @override
  State<PageHeure> createState() => _PageHeureState();
}

class _PageHeureState extends State<PageHeure> {
  TimeOfDay? _heureRaid;

  String _formater(TimeOfDay h) =>
      '${h.hour.toString().padLeft(2, '0')}h'
      '${h.minute.toString().padLeft(2, '0')}';

  Future<void> _choisirHeure() async {
    final TimeOfDay? choisie = await showTimePicker(
      context: context,
      initialTime: _heureRaid ?? const TimeOfDay(hour: 21, minute: 0),
      helpText: 'Heure de lancement du raid',
      cancelText: 'Annuler',
      confirmText: 'Valider',
      // Force le format 24 h, quel que soit le réglage du téléphone.
      builder: (BuildContext context, Widget? enfant) {
        return MediaQuery(
          data: MediaQuery.of(context).copyWith(alwaysUse24HourFormat: true),
          child: enfant!,
        );
      },
    );
    if (!mounted || choisie == null) {
      return;
    }
    setState(() => _heureRaid = choisie);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('showTimePicker')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
              _heureRaid == null
                  ? 'Aucune heure définie.'
                  : 'Raid prévu à ${_formater(_heureRaid!)}',
              style: const TextStyle(fontSize: 22),
            ),
            const SizedBox(height: 24),
            FilledButton.icon(
              onPressed: _choisirHeure,
              icon: const Icon(Icons.schedule),
              label: const Text('Choisir l\'heure'),
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
Aucune heure définie.
        [ ◔ Choisir l'heure ]

Boîte ouverte :
┌────────────────────────────┐
│  Heure de lancement du raid│
│        21 : 00             │
│         ( cadran )         │
│      Annuler    Valider    │
└────────────────────────────┘

Après validation :
Raid prévu à 21h00
```

**Trois remarques.**

1. **`TimeOfDay` n'est pas un `DateTime`.** Pour combiner une date et une heure :

   ```dart
   final DateTime complet = DateTime(
     date.year, date.month, date.day, heure.hour, heure.minute,
   );
   ```

2. **Le format 12 h / 24 h** suit le réglage du téléphone. Le `builder` ci-dessus le force en 24 h.
3. **`TimePickerEntryMode.input`** ouvre directement la saisie clavier plutôt que le cadran.

---

## 49.39 — `SnackBar` pour le retour utilisateur

Un `SnackBar` est un bandeau temporaire en bas de l'écran. C'est le retour utilisateur standard après une action : « Personnage créé », « Objet supprimé ».

On l'affiche par le `ScaffoldMessenger` :

```dart
ScaffoldMessenger.of(context).showSnackBar(
  const SnackBar(content: Text('Personnage créé.')),
);
```

| Paramètre du `SnackBar` | Rôle |
| --- | --- |
| `content` | le contenu, généralement un `Text` |
| `duration` | la durée d'affichage (4 s par défaut) |
| `action` | un `SnackBarAction` : bouton « Annuler » |
| `backgroundColor` | la couleur du bandeau |
| `behavior` | `SnackBarBehavior.floating` le décolle du bord |
| `showCloseIcon` | ajoute une croix de fermeture |

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageSnackBar(),
    );
  }
}

class PageSnackBar extends StatefulWidget {
  const PageSnackBar({super.key});

  @override
  State<PageSnackBar> createState() => _PageSnackBarState();
}

class _PageSnackBarState extends State<PageSnackBar> {
  final List<String> _inventaire = <String>[
    'Épée longue', 'Potion de soin', 'Clé rouillée',
  ];

  void _supprimer(int index) {
    final String objet = _inventaire[index];
    setState(() => _inventaire.removeAt(index));

    ScaffoldMessenger.of(context)
      // Retire le message précédent : évite l'empilement.
      ..hideCurrentSnackBar()
      ..showSnackBar(
        SnackBar(
          content: Text('$objet supprimé.'),
          duration: const Duration(seconds: 4),
          behavior: SnackBarBehavior.floating,
          action: SnackBarAction(
            label: 'Annuler',
            onPressed: () {
              setState(() => _inventaire.insert(index, objet));
            },
          ),
        ),
      );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('SnackBar')),
      body: Column(
        children: <Widget>[
          Expanded(
            child: ListView.builder(
              itemCount: _inventaire.length,
              itemBuilder: (BuildContext context, int index) {
                return ListTile(
                  leading: const Icon(Icons.inventory_2),
                  title: Text(_inventaire[index]),
                  trailing: IconButton(
                    icon: const Icon(Icons.delete_outline),
                    tooltip: 'Supprimer',
                    onPressed: () => _supprimer(index),
                  ),
                );
              },
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(16),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: <Widget>[
                OutlinedButton(
                  onPressed: () {
                    ScaffoldMessenger.of(context).showSnackBar(
                      const SnackBar(
                        content: Text('Sauvegarde effectuée.'),
                        backgroundColor: Colors.green,
                        showCloseIcon: true,
                      ),
                    );
                  },
                  child: const Text('Succès'),
                ),
                OutlinedButton(
                  onPressed: () {
                    ScaffoldMessenger.of(context).showSnackBar(
                      const SnackBar(
                        content: Text('Connexion au serveur impossible.'),
                        backgroundColor: Colors.red,
                      ),
                    );
                  },
                  child: const Text('Erreur'),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
  ▣  Épée longue                        ✕
  ▣  Potion de soin                     ✕
  ▣  Clé rouillée                       ✕

     [ Succès ]        [ Erreur ]

Après suppression de « Potion de soin » :

  ▣  Épée longue                        ✕
  ▣  Clé rouillée                       ✕

  ┌───────────────────────────────────────┐
  │ Potion de soin supprimé.     ANNULER  │
  └───────────────────────────────────────┘
```

### Le piège du `context`

C'est l'erreur la plus fréquente de tout ce chapitre :

```text
Scaffold does not contain a ScaffoldMessenger.
```

ou plus souvent, rien ne s'affiche du tout.

La cause : `ScaffoldMessenger.of(context)` remonte l'arbre depuis le `context` fourni. Si ce `context` est celui du widget qui **crée** le `Scaffold`, il est **au-dessus** de lui, donc `of` ne le trouve pas.

```dart
// FAUX : le context du build de MonApp est au-dessus du Scaffold.
class MonApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: ElevatedButton(
          onPressed: () => ScaffoldMessenger.of(context).showSnackBar(...),
          child: const Text('Afficher'),
        ),
      ),
    );
  }
}
```

**Trois corrections possibles :**

```dart
// 1. Extraire la page dans son propre widget (la meilleure).
home: const MaPage(),   // le build de MaPage a un context sous le Scaffold

// 2. Utiliser un Builder pour créer un context descendant.
body: Builder(
  builder: (BuildContext contexteInterne) {
    return ElevatedButton(
      onPressed: () =>
          ScaffoldMessenger.of(contexteInterne).showSnackBar(...),
      child: const Text('Afficher'),
    );
  },
),

// 3. Utiliser une GlobalKey<ScaffoldMessengerState> sur le MaterialApp
//    (utile depuis une couche sans context, ex. un service).
```

**Autre piège : le `context` après un `await`.**

```dart
Future<void> _envoyer() async {
  await _api.creer();
  if (!mounted) return;                     // indispensable
  ScaffoldMessenger.of(context).showSnackBar(...);
}
```

---

## 49.40 — `showDialog` et `AlertDialog`

Un dialogue **interrompt** l'utilisateur et exige une réponse. À réserver aux actions destructrices ou irréversibles.

`showDialog<T>` retourne un `Future<T?>`. La valeur est celle passée à `Navigator.pop(context, valeur)`, ou `null` si l'utilisateur ferme le dialogue en touchant à côté.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageDialog(),
    );
  }
}

class PageDialog extends StatefulWidget {
  const PageDialog({super.key});

  @override
  State<PageDialog> createState() => _PageDialogState();
}

class _PageDialogState extends State<PageDialog> {
  String _etat = 'Partie en cours.';

  Future<void> _confirmerAbandon() async {
    // Le type générique <bool> fixe le type de la valeur retournée.
    final bool? confirme = await showDialog<bool>(
      context: context,
      barrierDismissible: false, // oblige à choisir un bouton
      builder: (BuildContext contexteDialogue) {
        return AlertDialog(
          icon: const Icon(Icons.warning_amber, color: Colors.orange),
          title: const Text('Abandonner la partie ?'),
          content: const Text(
            'Toute progression non sauvegardée sera perdue. '
            'Cette action est irréversible.',
          ),
          actions: <Widget>[
            TextButton(
              onPressed: () => Navigator.pop(contexteDialogue, false),
              child: const Text('Continuer à jouer'),
            ),
            FilledButton(
              style: FilledButton.styleFrom(backgroundColor: Colors.red),
              onPressed: () => Navigator.pop(contexteDialogue, true),
              child: const Text('Abandonner'),
            ),
          ],
        );
      },
    );

    if (!mounted) {
      return;
    }
    setState(() {
      _etat = switch (confirme) {
        true => 'Partie abandonnée.',
        false => 'Partie en cours.',
        null => 'Dialogue fermé sans réponse.',
      };
    });
  }

  Future<void> _choisirRecompense() async {
    final String? choix = await showDialog<String>(
      context: context,
      builder: (BuildContext c) {
        return SimpleDialog(
          title: const Text('Choisissez votre récompense'),
          children: <Widget>[
            for (final String r in <String>['100 pièces d\'or', 'Potion rare',
                'Parchemin de feu'])
              SimpleDialogOption(
                onPressed: () => Navigator.pop(c, r),
                child: Text(r),
              ),
          ],
        );
      },
    );
    if (!mounted || choix == null) {
      return;
    }
    setState(() => _etat = 'Récompense : $choix');
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('showDialog')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(_etat, style: const TextStyle(fontSize: 18)),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: _confirmerAbandon,
              child: const Text('Abandonner la partie'),
            ),
            const SizedBox(height: 12),
            OutlinedButton(
              onPressed: _choisirRecompense,
              child: const Text('Ouvrir le coffre'),
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
┌────────────────────────────────────────┐
│               !                        │
│      Abandonner la partie ?            │
│                                        │
│  Toute progression non sauvegardée     │
│  sera perdue. Cette action est         │
│  irréversible.                         │
│                                        │
│   Continuer à jouer    [ Abandonner ]  │
└────────────────────────────────────────┘

Choix « Abandonner »  ->  Partie abandonnée.
Choix « Continuer »   ->  Partie en cours.
```

**Quatre règles.**

1. **Nommez le contexte du dialogue.** Dans le `builder`, le paramètre est un **nouveau** `BuildContext`. C'est lui qu'il faut passer à `Navigator.pop` pour fermer le dialogue, pas celui de la page.
2. **`showDialog<bool>` et `Navigator.pop(context, true)` vont ensemble.** Le type générique décide du type du `Future`.
3. **Le résultat est nullable.** `barrierDismissible: true` (défaut) permet de fermer en touchant à côté : le résultat est alors `null`. Traitez toujours ce cas.
4. **L'action destructrice se distingue visuellement** (rouge ici), et l'action sûre est toujours à gauche.

Le `switch` en expression utilisé plus haut est la syntaxe Dart 3 vue au chapitre 04.

---

## 49.41 — `showModalBottomSheet`

La feuille modale glisse depuis le bas. Elle convient mieux qu'un dialogue quand il faut afficher **une liste d'options** ou **un petit formulaire**.

```dart
Future<T?> showModalBottomSheet<T>({
  required BuildContext context,
  required WidgetBuilder builder,
  bool isScrollControlled = false,
  bool isDismissible = true,
  bool enableDrag = true,
  bool useSafeArea = false,
  Color? backgroundColor,
  ShapeBorder? shape,
})
```

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageBottomSheet(),
    );
  }
}

class PageBottomSheet extends StatefulWidget {
  const PageBottomSheet({super.key});

  @override
  State<PageBottomSheet> createState() => _PageBottomSheetState();
}

class _PageBottomSheetState extends State<PageBottomSheet> {
  String _action = 'aucune';
  String _nouvelObjet = '';

  Future<void> _ouvrirMenu() async {
    final String? choix = await showModalBottomSheet<String>(
      context: context,
      showDragHandle: true,
      builder: (BuildContext c) {
        return SafeArea(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: <Widget>[
              ListTile(
                leading: const Icon(Icons.visibility),
                title: const Text('Examiner'),
                onTap: () => Navigator.pop(c, 'examiner'),
              ),
              ListTile(
                leading: const Icon(Icons.backpack),
                title: const Text('Ranger dans le sac'),
                onTap: () => Navigator.pop(c, 'ranger'),
              ),
              ListTile(
                leading: const Icon(Icons.delete, color: Colors.red),
                title: const Text('Jeter',
                    style: TextStyle(color: Colors.red)),
                onTap: () => Navigator.pop(c, 'jeter'),
              ),
            ],
          ),
        );
      },
    );
    if (!mounted || choix == null) {
      return;
    }
    setState(() => _action = choix);
  }

  Future<void> _ouvrirFormulaire() async {
    final TextEditingController controleur = TextEditingController();
    final String? nom = await showModalBottomSheet<String>(
      context: context,
      // Indispensable : sans cela le clavier recouvre le champ.
      isScrollControlled: true,
      builder: (BuildContext c) {
        return Padding(
          // Décale la feuille de la hauteur exacte du clavier.
          padding: EdgeInsets.only(
            left: 20,
            right: 20,
            top: 20,
            bottom: MediaQuery.of(c).viewInsets.bottom + 20,
          ),
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: <Widget>[
              const Text('Nouvel objet',
                  style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
              const SizedBox(height: 16),
              TextField(
                controller: controleur,
                autofocus: true,
                decoration: const InputDecoration(
                  labelText: 'Nom de l\'objet',
                  border: OutlineInputBorder(),
                ),
                onSubmitted: (String v) => Navigator.pop(c, v),
              ),
              const SizedBox(height: 16),
              SizedBox(
                width: double.infinity,
                child: FilledButton(
                  onPressed: () => Navigator.pop(c, controleur.text),
                  child: const Text('Ajouter'),
                ),
              ),
            ],
          ),
        );
      },
    );
    controleur.dispose();
    if (!mounted || nom == null || nom.trim().isEmpty) {
      return;
    }
    setState(() => _nouvelObjet = nom.trim());
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('showModalBottomSheet')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text('Action : $_action'),
            Text('Dernier objet ajouté : '
                '${_nouvelObjet.isEmpty ? "—" : _nouvelObjet}'),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: _ouvrirMenu,
              child: const Text('Menu de l\'objet'),
            ),
            const SizedBox(height: 12),
            OutlinedButton(
              onPressed: _ouvrirFormulaire,
              child: const Text('Ajouter un objet'),
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
Menu ouvert :
┌──────────────────────────────────┐
│              ▬▬▬                 │   <- poignée de glissement
│  ◉  Examiner                    │
│  ▣  Ranger dans le sac          │
│  ✕  Jeter                       │
└──────────────────────────────────┘

Choix « Ranger »  ->  Action : ranger

Formulaire ouvert, clavier visible :
┌──────────────────────────────────┐
│         Nouvel objet             │
│  ┌─ Nom de l'objet ───────────┐  │
│  │ Amulette                   │  │
│  └────────────────────────────┘  │
│  ████████ Ajouter ████████       │
└──────────────────────────────────┘
   (la feuille est remontée au-dessus du clavier)
```

**Trois points clés.**

1. **`isScrollControlled: true` + `MediaQuery.of(c).viewInsets.bottom`** est le duo obligatoire dès qu'il y a un champ de saisie dans la feuille. Sans lui, le clavier recouvre le champ.
2. **`mainAxisSize: MainAxisSize.min`** sur la `Column` : sinon elle tente de remplir une hauteur infinie et lève une erreur de contraintes (chapitre 46).
3. **Le contrôleur créé pour la feuille doit être libéré.** Ici, `controleur.dispose()` est appelé après le `await`, quand la feuille est fermée.

**Variante :** `showBottomSheet` (sans « Modal ») affiche une feuille **persistante**, sans voile sombre, qui laisse le reste de l'écran utilisable.

---

## 49.42 — Mini-projet : un formulaire de création de personnage complet et validé

Ce projet réunit tout le chapitre : validateurs réutilisables, `Form`, `TextFormField`, sélecteurs, dialogue de confirmation, feuille de résumé et `SnackBar`.

### Cahier des charges

```text
CHAMPS                          CONTRAINTES
  Nom du héros                  obligatoire, 3 à 20 caractères
  E-mail du joueur              obligatoire, format valide
  Mot de passe                  obligatoire, 8 caractères minimum,
                                au moins un chiffre
  Classe (menu déroulant)       obligatoire
  Niveau de départ              obligatoire, entier de 1 à 99
  Alignement (radio)            obligatoire
  Points de vie (slider)        de 50 à 300, par paliers de 10
  Compétences (cases)           au moins une
  Notifications (switch)        facultatif
  Date de naissance             obligatoire, dans le passé
  Règles acceptées (case)       obligatoire

COMPORTEMENT
  Le bouton « Créer » est désactivé tant que les règles ne sont pas acceptées.
  À la soumission : validation, dialogue de confirmation, résumé, SnackBar.
  Les erreurs s'affichent en direct après le premier échec.
```

### Le code complet

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(const MonApplication());
}

// ---------------------------------------------------------------------------
// MODÈLE
// ---------------------------------------------------------------------------

enum Alignement { bon('Bon'), neutre('Neutre'), mauvais('Mauvais');

  const Alignement(this.libelle);
  final String libelle;
}

class Personnage {
  Personnage({
    required this.nom,
    required this.email,
    required this.classe,
    required this.niveau,
    required this.alignement,
    required this.pointsDeVie,
    required this.competences,
    required this.notifications,
    required this.naissance,
  });

  final String nom;
  final String email;
  final String classe;
  final int niveau;
  final Alignement alignement;
  final int pointsDeVie;
  final List<String> competences;
  final bool notifications;
  final DateTime naissance;

  String get resume => '''
Nom          : $nom
E-mail       : $email
Classe       : $classe
Niveau       : $niveau
Alignement   : ${alignement.libelle}
Points de vie: $pointsDeVie
Compétences  : ${competences.join(', ')}
Notifications: ${notifications ? 'activées' : 'désactivées'}
Naissance    : ${naissance.day}/${naissance.month}/${naissance.year}''';
}

// ---------------------------------------------------------------------------
// VALIDATEURS RÉUTILISABLES
// ---------------------------------------------------------------------------

class Valideurs {
  const Valideurs._();

  static String? Function(String?) requis([String m = 'Champ obligatoire.']) =>
      (String? v) => (v == null || v.trim().isEmpty) ? m : null;

  static String? Function(String?) longueur(int min, int max) => (String? v) {
        final String t = (v ?? '').trim();
        if (t.isEmpty) {
          return null;
        }
        if (t.length < min) {
          return 'Au moins $min caractères.';
        }
        if (t.length > max) {
          return 'Au plus $max caractères.';
        }
        return null;
      };

  static String? email(String? v) {
    final String t = (v ?? '').trim();
    if (t.isEmpty) {
      return null;
    }
    return RegExp(r'^[\w.+-]+@[\w-]+\.[\w.-]{2,}$').hasMatch(t)
        ? null
        : 'Adresse e-mail invalide.';
  }

  static String? motDePasse(String? v) {
    final String t = v ?? '';
    if (t.isEmpty) {
      return null;
    }
    if (t.length < 8) {
      return 'Au moins 8 caractères.';
    }
    if (!t.contains(RegExp(r'[0-9]'))) {
      return 'Doit contenir au moins un chiffre.';
    }
    return null;
  }

  static String? Function(String?) entierEntre(int min, int max) =>
      (String? v) {
        final String t = (v ?? '').trim();
        if (t.isEmpty) {
          return null;
        }
        final int? n = int.tryParse(t);
        if (n == null) {
          return 'Entrez un nombre entier.';
        }
        return (n < min || n > max) ? 'Valeur entre $min et $max.' : null;
      };

  static String? Function(String?) tous(
    List<String? Function(String?)> liste,
  ) =>
      (String? v) {
        for (final String? Function(String?) f in liste) {
          final String? e = f(v);
          if (e != null) {
            return e;
          }
        }
        return null;
      };
}

// ---------------------------------------------------------------------------
// APPLICATION
// ---------------------------------------------------------------------------

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Création de personnage',
      theme: ThemeData(
        colorSchemeSeed: Colors.deepPurple,
        useMaterial3: true,
        inputDecorationTheme: const InputDecorationTheme(
          border: OutlineInputBorder(),
          filled: true,
        ),
      ),
      home: const PageCreation(),
    );
  }
}

class PageCreation extends StatefulWidget {
  const PageCreation({super.key});

  @override
  State<PageCreation> createState() => _PageCreationState();
}

class _PageCreationState extends State<PageCreation> {
  // --- Clé du formulaire, champ final du State ---------------------------
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();

  // --- Contrôleurs et focus ---------------------------------------------
  final TextEditingController _nom = TextEditingController();
  final TextEditingController _email = TextEditingController();
  final TextEditingController _motDePasse = TextEditingController();
  final TextEditingController _niveau = TextEditingController(text: '1');
  final TextEditingController _champNaissance = TextEditingController();

  final FocusNode _focusEmail = FocusNode();
  final FocusNode _focusMotDePasse = FocusNode();
  final FocusNode _focusNiveau = FocusNode();

  // --- État des widgets non textuels -------------------------------------
  static const List<String> _classes = <String>[
    'Guerrier', 'Mage', 'Voleur', 'Clerc', 'Rôdeur',
  ];
  String? _classe;
  Alignement? _alignement;
  double _pointsDeVie = 100;
  bool _notifications = true;
  bool _reglesAcceptees = false;
  bool _masquerMotDePasse = true;
  DateTime? _naissance;
  bool _envoiEnCours = false;
  AutovalidateMode _mode = AutovalidateMode.disabled;

  final Map<String, bool> _competences = <String, bool>{
    'Escrime': false,
    'Magie': false,
    'Discrétion': false,
    'Alchimie': false,
  };

  List<String> get _competencesChoisies => _competences.entries
      .where((MapEntry<String, bool> e) => e.value)
      .map((MapEntry<String, bool> e) => e.key)
      .toList();

  @override
  void dispose() {
    _nom.dispose();
    _email.dispose();
    _motDePasse.dispose();
    _niveau.dispose();
    _champNaissance.dispose();
    _focusEmail.dispose();
    _focusMotDePasse.dispose();
    _focusNiveau.dispose();
    super.dispose();
  }

  // --- Sélecteur de date --------------------------------------------------
  Future<void> _choisirNaissance() async {
    final DateTime maintenant = DateTime.now();
    final DateTime? d = await showDatePicker(
      context: context,
      initialDate: _naissance ?? DateTime(maintenant.year - 20),
      firstDate: DateTime(1900),
      lastDate: maintenant,
      helpText: 'Date de naissance du joueur',
      cancelText: 'Annuler',
      confirmText: 'Valider',
    );
    if (!mounted || d == null) {
      return;
    }
    setState(() {
      _naissance = d;
      _champNaissance.text = '${d.day.toString().padLeft(2, '0')}/'
          '${d.month.toString().padLeft(2, '0')}/${d.year}';
    });
  }

  // --- Soumission ---------------------------------------------------------
  Future<void> _soumettre() async {
    FocusScope.of(context).unfocus();

    // 1. Validation des champs du Form.
    final bool champsValides = _formKey.currentState!.validate();

    // 2. Validation des widgets hors Form (cases et radio).
    final bool competencesOk = _competencesChoisies.isNotEmpty;
    final bool alignementOk = _alignement != null;
    final bool dateOk = _naissance != null;

    setState(() => _mode = AutovalidateMode.onUserInteraction);

    if (!champsValides || !competencesOk || !alignementOk || !dateOk) {
      _messageErreur(competencesOk, alignementOk, dateOk);
      return;
    }

    // 3. Confirmation.
    final bool? confirme = await showDialog<bool>(
      context: context,
      builder: (BuildContext c) => AlertDialog(
        icon: const Icon(Icons.person_add),
        title: const Text('Créer ce personnage ?'),
        content: Text('${_nom.text} — $_classe de niveau ${_niveau.text}'),
        actions: <Widget>[
          TextButton(
            onPressed: () => Navigator.pop(c, false),
            child: const Text('Modifier'),
          ),
          FilledButton(
            onPressed: () => Navigator.pop(c, true),
            child: const Text('Créer'),
          ),
        ],
      ),
    );
    if (!mounted || confirme != true) {
      return;
    }

    // 4. Envoi simulé, bouton désactivé pendant l'opération.
    setState(() => _envoiEnCours = true);
    await Future<void>.delayed(const Duration(milliseconds: 1200));
    if (!mounted) {
      return;
    }
    setState(() => _envoiEnCours = false);

    final Personnage p = Personnage(
      nom: _nom.text.trim(),
      email: _email.text.trim(),
      classe: _classe!,
      niveau: int.parse(_niveau.text.trim()),
      alignement: _alignement!,
      pointsDeVie: _pointsDeVie.round(),
      competences: _competencesChoisies,
      notifications: _notifications,
      naissance: _naissance!,
    );

    // 5. Résumé + retour utilisateur.
    await _afficherResume(p);
    if (!mounted) {
      return;
    }
    ScaffoldMessenger.of(context)
      ..hideCurrentSnackBar()
      ..showSnackBar(
        SnackBar(
          content: Text('${p.nom} rejoint votre équipe.'),
          backgroundColor: Colors.green,
          behavior: SnackBarBehavior.floating,
        ),
      );
  }

  void _messageErreur(bool competencesOk, bool alignementOk, bool dateOk) {
    final List<String> manques = <String>[
      if (!competencesOk) 'une compétence',
      if (!alignementOk) 'un alignement',
      if (!dateOk) 'une date de naissance',
    ];
    final String texte = manques.isEmpty
        ? 'Corrigez les champs en rouge.'
        : 'Il manque : ${manques.join(', ')}.';
    ScaffoldMessenger.of(context)
      ..hideCurrentSnackBar()
      ..showSnackBar(
        SnackBar(content: Text(texte), backgroundColor: Colors.red),
      );
  }

  Future<void> _afficherResume(Personnage p) {
    return showModalBottomSheet<void>(
      context: context,
      showDragHandle: true,
      isScrollControlled: true,
      builder: (BuildContext c) => SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(24),
          child: Column(
            mainAxisSize: MainAxisSize.min,
            crossAxisAlignment: CrossAxisAlignment.start,
            children: <Widget>[
              Text('Personnage créé',
                  style: Theme.of(c).textTheme.headlineSmall),
              const SizedBox(height: 16),
              Text(p.resume,
                  style: const TextStyle(fontFamily: 'monospace')),
              const SizedBox(height: 24),
              SizedBox(
                width: double.infinity,
                child: FilledButton(
                  onPressed: () => Navigator.pop(c),
                  child: const Text('Fermer'),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }

  // --- Interface ----------------------------------------------------------
  @override
  Widget build(BuildContext context) {
    final bool boutonActif = _reglesAcceptees && !_envoiEnCours;

    return Scaffold(
      appBar: AppBar(title: const Text('Créer un personnage')),
      body: Form(
        key: _formKey,
        autovalidateMode: _mode,
        child: ListView(
          padding: const EdgeInsets.all(20),
          children: <Widget>[
            const _Titre('Identité'),
            TextFormField(
              controller: _nom,
              textInputAction: TextInputAction.next,
              textCapitalization: TextCapitalization.words,
              onFieldSubmitted: (String _) => _focusEmail.requestFocus(),
              decoration: const InputDecoration(
                labelText: 'Nom du héros',
                hintText: 'ex. Kael le Vaillant',
                prefixIcon: Icon(Icons.person),
              ),
              validator: Valideurs.tous(<String? Function(String?)>[
                Valideurs.requis('Le nom est obligatoire.'),
                Valideurs.longueur(3, 20),
              ]),
            ),
            const SizedBox(height: 16),
            TextFormField(
              controller: _email,
              focusNode: _focusEmail,
              keyboardType: TextInputType.emailAddress,
              textInputAction: TextInputAction.next,
              autofillHints: const <String>[AutofillHints.email],
              onFieldSubmitted: (String _) => _focusMotDePasse.requestFocus(),
              decoration: const InputDecoration(
                labelText: 'E-mail du joueur',
                prefixIcon: Icon(Icons.email),
              ),
              validator: Valideurs.tous(<String? Function(String?)>[
                Valideurs.requis(),
                Valideurs.email,
              ]),
            ),
            const SizedBox(height: 16),
            TextFormField(
              controller: _motDePasse,
              focusNode: _focusMotDePasse,
              obscureText: _masquerMotDePasse,
              textInputAction: TextInputAction.next,
              autofillHints: const <String>[AutofillHints.newPassword],
              onFieldSubmitted: (String _) => _focusNiveau.requestFocus(),
              decoration: InputDecoration(
                labelText: 'Mot de passe',
                prefixIcon: const Icon(Icons.lock),
                helperText: '8 caractères minimum, dont un chiffre',
                suffixIcon: IconButton(
                  icon: Icon(_masquerMotDePasse
                      ? Icons.visibility
                      : Icons.visibility_off),
                  tooltip: _masquerMotDePasse ? 'Afficher' : 'Masquer',
                  onPressed: () => setState(
                    () => _masquerMotDePasse = !_masquerMotDePasse,
                  ),
                ),
              ),
              validator: Valideurs.tous(<String? Function(String?)>[
                Valideurs.requis(),
                Valideurs.motDePasse,
              ]),
            ),
            const SizedBox(height: 16),
            TextFormField(
              readOnly: true,
              controller: _champNaissance,
              onTap: _choisirNaissance,
              decoration: InputDecoration(
                labelText: 'Date de naissance',
                prefixIcon: const Icon(Icons.cake),
                suffixIcon: const Icon(Icons.calendar_month),
                errorText: _mode != AutovalidateMode.disabled &&
                        _naissance == null
                    ? 'Choisissez une date.'
                    : null,
              ),
            ),

            const _Titre('Caractéristiques'),
            DropdownButtonFormField<String>(
              initialValue: _classe,
              decoration: const InputDecoration(
                labelText: 'Classe',
                prefixIcon: Icon(Icons.shield),
              ),
              hint: const Text('Choisissez une classe'),
              items: _classes
                  .map((String c) =>
                      DropdownMenuItem<String>(value: c, child: Text(c)))
                  .toList(),
              onChanged: (String? v) => setState(() => _classe = v),
              validator: (String? v) =>
                  v == null ? 'La classe est obligatoire.' : null,
            ),
            const SizedBox(height: 16),
            TextFormField(
              controller: _niveau,
              focusNode: _focusNiveau,
              keyboardType: TextInputType.number,
              textInputAction: TextInputAction.done,
              inputFormatters: <TextInputFormatter>[
                FilteringTextInputFormatter.digitsOnly,
                LengthLimitingTextInputFormatter(2),
              ],
              decoration: const InputDecoration(
                labelText: 'Niveau de départ (1 à 99)',
                prefixIcon: Icon(Icons.trending_up),
              ),
              validator: Valideurs.tous(<String? Function(String?)>[
                Valideurs.requis(),
                Valideurs.entierEntre(1, 99),
              ]),
            ),
            const SizedBox(height: 16),
            Text('Points de vie : ${_pointsDeVie.round()}'),
            Slider(
              value: _pointsDeVie,
              min: 50,
              max: 300,
              divisions: 25,
              label: '${_pointsDeVie.round()} PV',
              onChanged: (double v) => setState(() => _pointsDeVie = v),
            ),

            const _Titre('Alignement'),
            RadioGroup<Alignement>(
              groupValue: _alignement,
              onChanged: (Alignement? v) => setState(() => _alignement = v),
              child: Column(
                children: Alignement.values.map((Alignement a) {
                  return RadioListTile<Alignement>(
                    value: a,
                    title: Text(a.libelle),
                    dense: true,
                  );
                }).toList(),
              ),
            ),
            if (_mode != AutovalidateMode.disabled && _alignement == null)
              const Padding(
                padding: EdgeInsets.only(left: 16),
                child: Text(
                  'Choisissez un alignement.',
                  style: TextStyle(color: Colors.red, fontSize: 12),
                ),
              ),

            const _Titre('Compétences (au moins une)'),
            ..._competences.keys.map((String nom) {
              return CheckboxListTile(
                value: _competences[nom],
                title: Text(nom),
                dense: true,
                controlAffinity: ListTileControlAffinity.leading,
                onChanged: (bool? v) =>
                    setState(() => _competences[nom] = v ?? false),
              );
            }),
            if (_mode != AutovalidateMode.disabled &&
                _competencesChoisies.isEmpty)
              const Padding(
                padding: EdgeInsets.only(left: 16),
                child: Text(
                  'Choisissez au moins une compétence.',
                  style: TextStyle(color: Colors.red, fontSize: 12),
                ),
              ),

            const _Titre('Préférences'),
            SwitchListTile(
              value: _notifications,
              title: const Text('Notifications de guilde'),
              secondary: const Icon(Icons.notifications),
              onChanged: (bool v) => setState(() => _notifications = v),
            ),
            CheckboxListTile(
              value: _reglesAcceptees,
              title: const Text("J'accepte les règles du jeu"),
              controlAffinity: ListTileControlAffinity.leading,
              onChanged: (bool? v) =>
                  setState(() => _reglesAcceptees = v ?? false),
            ),

            const SizedBox(height: 24),
            SizedBox(
              height: 52,
              child: FilledButton(
                onPressed: boutonActif ? _soumettre : null,
                child: _envoiEnCours
                    ? const SizedBox(
                        width: 22,
                        height: 22,
                        child: CircularProgressIndicator(strokeWidth: 2),
                      )
                    : const Text('Créer le personnage'),
              ),
            ),
            const SizedBox(height: 8),
            TextButton(
              onPressed: _envoiEnCours
                  ? null
                  : () {
                      _formKey.currentState!.reset();
                      setState(() {
                        _classe = null;
                        _alignement = null;
                        _naissance = null;
                        _champNaissance.clear();
                        _pointsDeVie = 100;
                        _reglesAcceptees = false;
                        _mode = AutovalidateMode.disabled;
                        for (final String k in _competences.keys) {
                          _competences[k] = false;
                        }
                      });
                    },
              child: const Text('Tout réinitialiser'),
            ),
            const SizedBox(height: 24),
          ],
        ),
      ),
    );
  }
}

class _Titre extends StatelessWidget {
  const _Titre(this.texte);

  final String texte;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.only(top: 24, bottom: 8),
      child: Text(
        texte.toUpperCase(),
        style: TextStyle(
          fontSize: 12,
          letterSpacing: 1.2,
          fontWeight: FontWeight.bold,
          color: Theme.of(context).colorScheme.primary,
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  Créer un personnage                     │
├──────────────────────────────────────────┤
│  IDENTITÉ                                │
│  ┌─ Nom du héros ──────────────────────┐ │
│  │ ◉ Kael le Vaillant                 │ │
│  └─────────────────────────────────────┘ │
│  ┌─ E-mail du joueur ──────────────────┐ │
│  │ ▫ kael@jeu.fr                       │ │
│  └─────────────────────────────────────┘ │
│  ┌─ Mot de passe ──────────────────────┐ │
│  │ ▪ ••••••••                     ◉  │ │
│  └─────────────────────────────────────┘ │
│  ┌─ Date de naissance ─────────────────┐ │
│  │ ◍ 14/03/2001                   ▤  │ │
│  └─────────────────────────────────────┘ │
│  CARACTÉRISTIQUES                        │
│  ┌─ Classe ────────────────────────┐     │
│  │ ◈ Mage                       ⌄ │     │
│  └─────────────────────────────────┘     │
│  Points de vie : 180                     │
│  ────────────●───────────────            │
│  ALIGNEMENT                              │
│   ○ Bon    ● Neutre    ○ Mauvais         │
│  COMPÉTENCES (AU MOINS UNE)              │
│   [x] Escrime   [x] Magie                    │
│  [x] J'accepte les règles du jeu           │
│  ███████ Créer le personnage ███████     │
└──────────────────────────────────────────┘

Appui sur « Créer » :
  1. Dialogue « Créer ce personnage ? Kael le Vaillant — Mage de niveau 12 »
  2. Indicateur de chargement pendant 1,2 s
  3. Feuille de résumé :

     Personnage créé
     Nom          : Kael le Vaillant
     E-mail       : kael@jeu.fr
     Classe       : Mage
     Niveau       : 12
     Alignement   : Neutre
     Points de vie: 180
     Compétences  : Escrime, Magie
     Notifications: activées
     Naissance    : 14/3/2001

  4. SnackBar verte « Kael le Vaillant rejoint votre équipe. »
```

### Les huit décisions de conception à retenir

| Décision | Raison |
| --- | --- |
| `GlobalKey<FormState>` en champ `final` du `State` | sinon `currentState` est perdu à chaque build |
| Tous les contrôleurs et `FocusNode` libérés dans `dispose` | pas de fuite mémoire |
| `_mode` passe à `onUserInteraction` après le premier échec | l'utilisateur n'est pas agressé de rouge dès l'arrivée |
| Erreurs manuelles pour radio et cases | ces widgets ne sont pas des `FormField`, `validate()` les ignore |
| `onPressed: null` tant que les règles ne sont pas acceptées | l'API officielle de désactivation |
| `_envoiEnCours` bloque le double-envoi | évite deux créations |
| `if (!mounted) return;` après chaque `await` | pas de `setState` après `dispose` |
| Validateurs regroupés dans `Valideurs` | zéro duplication, testables unitairement |

### Extensions à réaliser seul

1. Ajouter un champ « Confirmer le mot de passe » dont le validateur compare avec le premier.
2. Remplacer les cases par des `FilterChip` (widget `Chip` sélectionnable).
3. Empêcher plus de deux compétences choisies.
4. Faire dépendre le maximum de points de vie de la classe choisie.
5. Transformer le groupe de compétences en un vrai `FormField<List<String>>` pour qu'il soit pris en charge par `validate()`.

---

## 49.43 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| Le bouton s'exécute au chargement de l'écran | `onPressed: maFonction()` avec des parenthèses | écrire `onPressed: maFonction` ou `onPressed: () => maFonction()` |
| `A TextEditingController was used after being disposed.` | le contrôleur est utilisé après `dispose`, souvent après un `await` | ajouter `if (!mounted) return;` après chaque `await` |
| Le champ se vide à chaque frappe et la mémoire grimpe | `TextEditingController` créé dans `build` | le déclarer `final` dans le `State` |
| Fuite mémoire, écran de plus en plus lent | contrôleur ou `FocusNode` non libéré | tout libérer dans `dispose()`, avec `super.dispose()` en dernier |
| `Null check operator used on a null value` sur `currentState!` | `GlobalKey<FormState>` recréée à chaque `build`, ou aucun `Form` attaché | déclarer la clé en champ `final` du `State` et la passer à `Form(key: ...)` |
| `validate()` retourne toujours `true` alors que les champs sont vides | les champs sont des `TextField` et non des `TextFormField` | remplacer par `TextFormField` avec un `validator` |
| Le validateur ne s'exécute jamais | le `TextFormField` n'est pas descendant du `Form` porteur de la clé | vérifier l'arbre : le `Form` doit être un ancêtre |
| La case à cocher ne se coche pas visuellement | `onChanged` modifie la variable sans `setState` | envelopper l'affectation dans `setState(() { ... })` |
| Le `SnackBar` ne s'affiche pas | `context` situé au-dessus du `Scaffold` | extraire la page dans son propre widget, ou utiliser un `Builder` |
| `setState() or markNeedsBuild() called during build` | `setState` appelé dans un `validator` ou pendant `build` | déplacer la logique hors de la phase de construction |
| `initialValue == null || controller == null` | `TextFormField` reçoit à la fois `controller` et `initialValue` | passer la valeur au contrôleur : `TextEditingController(text: ...)` |
| `There should be exactly one item with [DropdownButton]'s value` | la valeur courante n'est pas dans `items`, ou deux items ont la même valeur | remettre la valeur à `null` en même temps que la liste ; vérifier l'unicité |
| L'ondulation de l'`InkWell` est invisible | l'`InkWell` est enfant d'un `Container` coloré | remplacer le `Container` par un `Material` porteur de la couleur |
| L'ondulation dépasse les coins arrondis | pas de `borderRadius` sur l'`InkWell` | donner le même `borderRadius` qu'au `Material`, ou `clipBehavior: Clip.antiAlias` |
| Un `GestureDetector` ne réagit pas dans les zones vides | l'enfant est transparent | ajouter `behavior: HitTestBehavior.opaque` |
| `RenderFlex overflowed` dès que le clavier s'ouvre | la `Column` ne tient plus dans la hauteur réduite | rendre le corps défilant avec `ListView` ou `SingleChildScrollView` |
| Le clavier recouvre le champ d'une feuille modale | `isScrollControlled` absent | `isScrollControlled: true` + padding `MediaQuery.of(c).viewInsets.bottom` |
| Le formulaire s'envoie deux fois | double appui pendant l'appel réseau | drapeau `_envoiEnCours` et `onPressed: null` pendant l'envoi |
| Exception `FormatException` à la validation d'un nombre | usage de `int.parse` sur une saisie libre | utiliser `int.tryParse` et tester `null` |
| Un champ « obligatoire » accepte trois espaces | validateur sans `.trim()` | tester `valeur.trim().isEmpty` |
| L'utilisateur voit du rouge partout dès l'ouverture | `AutovalidateMode.always` | commencer en `disabled`, passer à `onUserInteraction` après le premier échec |
| Le bouton semble cliquable mais ne fait rien | `onPressed: () {}` au lieu de `onPressed: null` | passer `null` pour désactiver réellement |
| `use_build_context_synchronously` signalé par l'analyseur | `context` utilisé après un `await` | `if (!mounted) return;` juste après le `await` |
| Le sélecteur de date plante à l'ouverture | `initialDate` hors de `[firstDate, lastDate]` | recalculer `initialDate` en le bornant |

---

## 49.44 — Résumé du chapitre

| Widget / notion | Usage |
| --- | --- |
| `ElevatedButton` | action importante avec ombre ; variante `.icon` |
| `FilledButton` | action principale Material 3 ; variantes `.tonal`, `.icon` |
| `OutlinedButton` | action secondaire, contour sans fond |
| `TextButton` | action tertiaire, dialogues et `AppBar` |
| `IconButton` | bouton rond à icône seule ; toujours avec `tooltip` |
| `FloatingActionButton` | action la plus fréquente de l'écran ; un seul par écran |
| `onPressed: null` | désactive réellement le bouton (grisé, inerte) |
| `onLongPress` | raccourci sur appui maintenu ; jamais indispensable |
| `styleFrom` | style simple d'un bouton (couleurs, forme, marges) |
| `ButtonStyle` + `WidgetStateProperty` | style dépendant de l'état (pressé, survolé, désactivé) |
| `GestureDetector` | rend n'importe quel widget cliquable, sans retour visuel |
| `InkWell` | idem avec l'ondulation Material ; exige un `Material` parent |
| `TextField` | champ de saisie de base, hors formulaire |
| `InputDecoration` | habillage du champ : label, hint, icônes, bordures, compteur |
| `TextEditingController` | lit et écrit le contenu d'un champ |
| `dispose()` | libère contrôleurs et `FocusNode` ; `super.dispose()` en dernier |
| `onChanged` | appelé à chaque frappe |
| `onSubmitted` | appelé à la validation clavier |
| `keyboardType` | choisit le clavier affiché ; ne valide rien |
| `obscureText` | masque un mot de passe ; à compléter par un œil |
| `maxLength` | limite la longueur et affiche un compteur |
| `inputFormatters` | filtre ou transforme la saisie en temps réel |
| `FocusNode` | contrôle le focus ; enchaînement de champs |
| `FocusScope.of(context).unfocus()` | ferme le clavier |
| `Form` | conteneur logique qui coordonne des champs |
| `GlobalKey<FormState>` | poignée vers l'état du formulaire ; champ `final` du `State` |
| `TextFormField` | champ intégré au `Form`, avec `validator` et `onSaved` |
| `validator` | retourne `null` si valide, sinon le message d'erreur |
| `validate()` | exécute tous les validateurs, affiche les erreurs, retourne un `bool` |
| `save()` | exécute tous les `onSaved` ; à appeler après `validate()` |
| `autovalidateMode` | quand valider : `disabled`, `onUserInteraction`, `onUnfocus`… |
| `int.tryParse` | conversion sûre d'un texte en entier ; retourne `null` si impossible |
| `Checkbox` / `CheckboxListTile` | booléen dans un formulaire ; `onChanged` reçoit un `bool?` |
| `Switch` / `SwitchListTile` | réglage à effet immédiat ; `onChanged` reçoit un `bool` |
| `RadioGroup` + `RadioListTile` | choix exclusif parmi quelques options |
| `Slider` | valeur numérique continue ou par paliers ; travaille en `double` |
| `DropdownButton` | choix parmi de nombreuses options |
| `DropdownButtonFormField` | idem, validé dans un `Form` ; `initialValue` et non `value` |
| `showDatePicker` | calendrier ; retourne `Future<DateTime?>` |
| `showTimePicker` | horloge ; retourne `Future<TimeOfDay?>` |
| `SnackBar` | retour utilisateur bref, via `ScaffoldMessenger.of(context)` |
| `showDialog` + `AlertDialog` | confirmation bloquante ; retourne `Future<T?>` |
| `showModalBottomSheet` | menu ou petit formulaire glissant depuis le bas |

---

## 49.45 — Exercices

### Exercice 1 — Compteur de score (facile)

Écrivez un écran affichant un score initialisé à 0, avec trois boutons : un `FilledButton` « +10 », un `OutlinedButton` « -5 » et un `TextButton` « Remettre à zéro ». Le bouton « -5 » doit être désactivé quand le score vaut 0.

### Exercice 2 — Bienvenue en direct (facile)

Affichez un `TextField` avec le label « Votre pseudo ». Sous le champ, affichez « Bonjour, X ! » qui se met à jour à chaque frappe. Si le champ est vide, affichez « Entrez votre pseudo ».

### Exercice 3 — Champ mot de passe (facile)

Créez un champ mot de passe masqué, avec un `suffixIcon` qui bascule la visibilité, un `maxLength` de 20 et un `helperText`. Affichez sous le champ le nombre de caractères saisis.

### Exercice 4 — Sélecteur de difficulté (facile)

Avec un `RadioGroup` et trois `RadioListTile`, laissez choisir « Facile », « Normal » ou « Cauchemar ». Affichez sous les options le multiplicateur de dégâts correspondant : 0.5, 1.0 ou 2.0.

### Exercice 5 — Formulaire de connexion (intermédiaire)

Construisez un `Form` avec deux champs : e-mail (format valide obligatoire) et mot de passe (6 caractères minimum). Le bouton « Se connecter » valide le formulaire et affiche un `SnackBar` vert en cas de succès, rouge en cas d'échec.

### Exercice 6 — Panneau d'options (intermédiaire)

Écrivez un écran d'options avec : un `Switch` « Musique », un `Switch` « Effets » désactivé si la musique est coupée, un `Slider` de volume de 0 à 100 par paliers de 5, et une `Checkbox` « Mode plein écran ». Affichez un résumé de l'état en bas.

### Exercice 7 — Calculateur de dégâts (intermédiaire)

Créez un `Form` avec deux champs numériques : « Attaque » (1 à 999) et « Défense » (0 à 999). Utilisez `int.tryParse` dans les validateurs et `inputFormatters` pour n'accepter que des chiffres. Le bouton calcule et affiche `max(1, attaque - défense)`.

### Exercice 8 — Suppression avec annulation (intermédiaire)

Affichez une `ListView` de cinq objets d'inventaire. Chaque ligne a un `IconButton` de suppression qui retire l'objet et affiche un `SnackBar` avec une action « Annuler » qui le remet à sa position d'origine.

### Exercice 9 — Rendez-vous de raid (avancé)

Écrivez un formulaire avec un champ « Nom du raid » (obligatoire), un sélecteur de date (`showDatePicker`, à partir d'aujourd'hui) et un sélecteur d'heure (`showTimePicker`). Le bouton « Planifier » n'est actif que si les trois valeurs sont renseignées, et ouvre un `AlertDialog` de confirmation.

### Exercice 10 — Inscription complète (avancé)

Assemblez un formulaire d'inscription avec : pseudo (3 à 15 caractères), e-mail, mot de passe (8 caractères et un chiffre), confirmation du mot de passe (identique au premier), une `DropdownButtonFormField` de pays et une case « J'accepte les conditions » qui conditionne l'activation du bouton. Utilisez `AutovalidateMode.disabled` puis `onUserInteraction` après le premier échec, et affichez le résultat dans une feuille modale.

---

## 49.46 — Corrections des exercices

### Correction 1

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
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

  void _ajouter() => setState(() => _score += 10);

  void _retirer() => setState(() {
        _score -= 5;
        if (_score < 0) {
          _score = 0;
        }
      });

  void _remettreAZero() => setState(() => _score = 0);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Score')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
              '$_score',
              style: const TextStyle(fontSize: 64, fontWeight: FontWeight.bold),
            ),
            const Text('points'),
            const SizedBox(height: 32),
            FilledButton(
              onPressed: _ajouter,
              child: const Text('+10'),
            ),
            const SizedBox(height: 12),
            OutlinedButton(
              // Désactivation par null : c'est l'API officielle.
              onPressed: _score == 0 ? null : _retirer,
              child: const Text('-5'),
            ),
            const SizedBox(height: 12),
            TextButton(
              onPressed: _score == 0 ? null : _remettreAZero,
              child: const Text('Remettre à zéro'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Explication :** l'état `_score` vit dans le `State`, et chaque bouton le modifie dans un `setState`. La désactivation utilise l'expression conditionnelle `_score == 0 ? null : _retirer` : quand le score vaut 0, `onPressed` reçoit `null`, donc Flutter grise le bouton et ignore les appuis. La méthode `_retirer` borne le résultat à 0 pour qu'un score négatif soit impossible même si la logique évoluait.

---

### Correction 2

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PagePseudo(),
    );
  }
}

class PagePseudo extends StatefulWidget {
  const PagePseudo({super.key});

  @override
  State<PagePseudo> createState() => _PagePseudoState();
}

class _PagePseudoState extends State<PagePseudo> {
  String _pseudo = '';

  @override
  Widget build(BuildContext context) {
    final String propre = _pseudo.trim();

    return Scaffold(
      appBar: AppBar(title: const Text('Bienvenue')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            TextField(
              autofocus: true,
              textCapitalization: TextCapitalization.words,
              decoration: const InputDecoration(
                labelText: 'Votre pseudo',
                prefixIcon: Icon(Icons.person),
                border: OutlineInputBorder(),
              ),
              onChanged: (String v) => setState(() => _pseudo = v),
            ),
            const SizedBox(height: 32),
            Text(
              propre.isEmpty ? 'Entrez votre pseudo' : 'Bonjour, $propre !',
              textAlign: TextAlign.center,
              style: TextStyle(
                fontSize: 24,
                color: propre.isEmpty ? Colors.grey : Colors.indigo,
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Explication :** `onChanged` reçoit la valeur complète du champ à chaque frappe et la stocke dans `_pseudo` via `setState`, ce qui provoque une reconstruction et met le message à jour instantanément. Le `.trim()` est appliqué **à l'affichage** et non au stockage : l'utilisateur peut encore taper un espace au milieu d'un pseudo composé, mais un champ ne contenant que des espaces est traité comme vide.

---

### Correction 3

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageMdp(),
    );
  }
}

class PageMdp extends StatefulWidget {
  const PageMdp({super.key});

  @override
  State<PageMdp> createState() => _PageMdpState();
}

class _PageMdpState extends State<PageMdp> {
  final TextEditingController _controleur = TextEditingController();
  bool _masque = true;

  @override
  void initState() {
    super.initState();
    _controleur.addListener(_surChangement);
  }

  void _surChangement() => setState(() {});

  @override
  void dispose() {
    _controleur.removeListener(_surChangement);
    _controleur.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Mot de passe')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: <Widget>[
            TextField(
              controller: _controleur,
              obscureText: _masque,
              maxLength: 20,
              autofillHints: const <String>[AutofillHints.newPassword],
              decoration: InputDecoration(
                labelText: 'Mot de passe',
                helperText: 'Entre 8 et 20 caractères.',
                prefixIcon: const Icon(Icons.lock),
                border: const OutlineInputBorder(),
                suffixIcon: IconButton(
                  icon: Icon(
                    _masque ? Icons.visibility : Icons.visibility_off,
                  ),
                  tooltip: _masque ? 'Afficher' : 'Masquer',
                  onPressed: () => setState(() => _masque = !_masque),
                ),
              ),
            ),
            const SizedBox(height: 16),
            Text('Caractères saisis : ${_controleur.text.length}'),
          ],
        ),
      ),
    );
  }
}
```

**Explication :** le contrôleur est nécessaire ici parce qu'on veut lire la longueur du texte **en dehors** du `onChanged`. Comme un contrôleur ne provoque pas de reconstruction à lui seul, on lui ajoute un auditeur (`addListener`) qui appelle un `setState` vide : cela suffit à redessiner le compteur. L'auditeur est retiré et le contrôleur libéré dans `dispose`, dans cet ordre, avant `super.dispose()`. Le `maxLength` affiche automatiquement le compteur `n/20` de Flutter, en plus de notre propre affichage.

---

### Correction 4

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

enum Difficulte {
  facile('Facile', 0.5),
  normal('Normal', 1.0),
  cauchemar('Cauchemar', 2.0);

  const Difficulte(this.libelle, this.multiplicateur);

  final String libelle;
  final double multiplicateur;
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageDifficulte(),
    );
  }
}

class PageDifficulte extends StatefulWidget {
  const PageDifficulte({super.key});

  @override
  State<PageDifficulte> createState() => _PageDifficulteState();
}

class _PageDifficulteState extends State<PageDifficulte> {
  Difficulte _choix = Difficulte.normal;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Difficulté')),
      body: Column(
        children: <Widget>[
          RadioGroup<Difficulte>(
            groupValue: _choix,
            onChanged: (Difficulte? v) =>
                setState(() => _choix = v ?? Difficulte.normal),
            child: Column(
              children: Difficulte.values.map((Difficulte d) {
                return RadioListTile<Difficulte>(
                  value: d,
                  title: Text(d.libelle),
                  subtitle: Text('Dégâts x${d.multiplicateur}'),
                );
              }).toList(),
            ),
          ),
          const Divider(),
          Padding(
            padding: const EdgeInsets.all(24),
            child: Text(
              'Multiplicateur de dégâts : x${_choix.multiplicateur}',
              style: const TextStyle(fontSize: 20),
            ),
          ),
        ],
      ),
    );
  }
}
```

**Explication :** l'énumération enrichie du chapitre 11 porte à la fois le libellé et le multiplicateur, ce qui supprime tout `switch` ou table de correspondance. Le `RadioGroup<Difficulte>` détient la valeur choisie et le callback ; chaque `RadioListTile<Difficulte>` ne porte que sa propre `value`. Comme `_choix` est initialisé à `Difficulte.normal`, un choix est déjà coché à l'ouverture et le type n'a pas besoin d'être nullable.

---

### Correction 5

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageConnexion(),
    );
  }
}

class PageConnexion extends StatefulWidget {
  const PageConnexion({super.key});

  @override
  State<PageConnexion> createState() => _PageConnexionState();
}

class _PageConnexionState extends State<PageConnexion> {
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();
  final TextEditingController _email = TextEditingController();
  final TextEditingController _mdp = TextEditingController();
  final FocusNode _focusMdp = FocusNode();
  bool _masque = true;

  @override
  void dispose() {
    _email.dispose();
    _mdp.dispose();
    _focusMdp.dispose();
    super.dispose();
  }

  void _seConnecter() {
    FocusScope.of(context).unfocus();
    final bool valide = _formKey.currentState!.validate();
    ScaffoldMessenger.of(context)
      ..hideCurrentSnackBar()
      ..showSnackBar(
        SnackBar(
          content: Text(
            valide
                ? 'Connexion réussie : ${_email.text.trim()}'
                : 'Corrigez les erreurs avant de continuer.',
          ),
          backgroundColor: valide ? Colors.green : Colors.red,
        ),
      );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Connexion')),
      body: Form(
        key: _formKey,
        child: ListView(
          padding: const EdgeInsets.all(24),
          children: <Widget>[
            TextFormField(
              controller: _email,
              keyboardType: TextInputType.emailAddress,
              textInputAction: TextInputAction.next,
              onFieldSubmitted: (String _) => _focusMdp.requestFocus(),
              decoration: const InputDecoration(
                labelText: 'Adresse e-mail',
                prefixIcon: Icon(Icons.email),
                border: OutlineInputBorder(),
              ),
              validator: (String? v) {
                final String t = (v ?? '').trim();
                if (t.isEmpty) {
                  return "L'e-mail est obligatoire.";
                }
                if (!RegExp(r'^[\w.+-]+@[\w-]+\.[\w.-]{2,}$').hasMatch(t)) {
                  return 'Adresse e-mail invalide.';
                }
                return null;
              },
            ),
            const SizedBox(height: 16),
            TextFormField(
              controller: _mdp,
              focusNode: _focusMdp,
              obscureText: _masque,
              textInputAction: TextInputAction.done,
              onFieldSubmitted: (String _) => _seConnecter(),
              decoration: InputDecoration(
                labelText: 'Mot de passe',
                prefixIcon: const Icon(Icons.lock),
                border: const OutlineInputBorder(),
                suffixIcon: IconButton(
                  icon: Icon(
                    _masque ? Icons.visibility : Icons.visibility_off,
                  ),
                  onPressed: () => setState(() => _masque = !_masque),
                ),
              ),
              validator: (String? v) {
                final String t = v ?? '';
                if (t.isEmpty) {
                  return 'Le mot de passe est obligatoire.';
                }
                if (t.length < 6) {
                  return 'Au moins 6 caractères.';
                }
                return null;
              },
            ),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: _seConnecter,
              child: const Text('Se connecter'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Explication :** la clé `_formKey` est un champ `final` du `State`, condition pour que `currentState` reste valide entre deux reconstructions. Les deux champs sont des `TextFormField`, seuls widgets pris en compte par `validate()`. L'enchaînement clavier est assuré par `textInputAction: TextInputAction.next` et `onFieldSubmitted` qui donne le focus au champ suivant ; sur le dernier champ, `TextInputAction.done` déclenche directement la connexion. Le clavier est fermé au début de `_seConnecter` pour que le `SnackBar` soit visible.

---

### Correction 6

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageOptions(),
    );
  }
}

class PageOptions extends StatefulWidget {
  const PageOptions({super.key});

  @override
  State<PageOptions> createState() => _PageOptionsState();
}

class _PageOptionsState extends State<PageOptions> {
  bool _musique = true;
  bool _effets = true;
  bool _pleinEcran = false;
  double _volume = 60;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Options')),
      body: ListView(
        children: <Widget>[
          SwitchListTile(
            value: _musique,
            title: const Text('Musique'),
            secondary: Icon(_musique ? Icons.music_note : Icons.music_off),
            onChanged: (bool v) {
              setState(() {
                _musique = v;
                if (!v) {
                  _effets = false; // dépendance descendante
                }
              });
            },
          ),
          SwitchListTile(
            value: _effets,
            title: const Text('Effets sonores'),
            subtitle: _musique
                ? null
                : const Text('Indisponible : la musique est coupée'),
            secondary: const Icon(Icons.graphic_eq),
            // Désactivé si la musique est coupée.
            onChanged:
                _musique ? (bool v) => setState(() => _effets = v) : null,
          ),
          const Divider(),
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 16),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: <Widget>[
                Text('Volume : ${_volume.round()} %'),
                Slider(
                  value: _volume,
                  min: 0,
                  max: 100,
                  divisions: 20, // paliers de 5
                  label: '${_volume.round()} %',
                  onChanged: _musique
                      ? (double v) => setState(() => _volume = v)
                      : null,
                ),
              ],
            ),
          ),
          const Divider(),
          CheckboxListTile(
            value: _pleinEcran,
            title: const Text('Mode plein écran'),
            controlAffinity: ListTileControlAffinity.leading,
            onChanged: (bool? v) => setState(() => _pleinEcran = v ?? false),
          ),
          const Divider(),
          Padding(
            padding: const EdgeInsets.all(16),
            child: Text(
              'Résumé\n'
              'Musique      : ${_musique ? "activée" : "coupée"}\n'
              'Effets       : ${_effets ? "activés" : "coupés"}\n'
              'Volume       : ${_volume.round()} %\n'
              'Plein écran  : ${_pleinEcran ? "oui" : "non"}',
              style: const TextStyle(fontFamily: 'monospace'),
            ),
          ),
        ],
      ),
    );
  }
}
```

**Explication :** la dépendance entre réglages est gérée à deux endroits. D'une part, couper la musique force `_effets` à `false` dans le même `setState`, ce qui évite un état incohérent. D'autre part, `onChanged: _musique ? ... : null` désactive visuellement les widgets dépendants. Le `Slider` utilise `divisions: 20` pour obtenir des paliers de 5 sur une plage de 0 à 100 : le nombre de divisions est `(max - min) / pas`. Notez que `Slider.onChanged` accepte aussi `null` pour désactiver.

---

### Correction 7

```dart
import 'dart:math' as math;

import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageDegats(),
    );
  }
}

class PageDegats extends StatefulWidget {
  const PageDegats({super.key});

  @override
  State<PageDegats> createState() => _PageDegatsState();
}

class _PageDegatsState extends State<PageDegats> {
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();
  int _attaque = 0;
  int _defense = 0;
  int? _resultat;

  String? _valider(String? v, int min, int max) {
    final String t = (v ?? '').trim();
    if (t.isEmpty) {
      return 'Champ obligatoire.';
    }
    final int? n = int.tryParse(t);
    if (n == null) {
      return 'Entrez un nombre entier.';
    }
    if (n < min || n > max) {
      return 'Valeur entre $min et $max.';
    }
    return null;
  }

  void _calculer() {
    final FormState etat = _formKey.currentState!;
    if (!etat.validate()) {
      setState(() => _resultat = null);
      return;
    }
    etat.save();
    setState(() => _resultat = math.max(1, _attaque - _defense));
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Calcul de dégâts')),
      body: Form(
        key: _formKey,
        autovalidateMode: AutovalidateMode.onUserInteraction,
        child: ListView(
          padding: const EdgeInsets.all(24),
          children: <Widget>[
            TextFormField(
              keyboardType: TextInputType.number,
              inputFormatters: <TextInputFormatter>[
                FilteringTextInputFormatter.digitsOnly,
                LengthLimitingTextInputFormatter(3),
              ],
              decoration: const InputDecoration(
                labelText: 'Attaque (1 à 999)',
                prefixIcon: Icon(Icons.sports_martial_arts),
                border: OutlineInputBorder(),
              ),
              validator: (String? v) => _valider(v, 1, 999),
              onSaved: (String? v) => _attaque = int.parse(v!.trim()),
            ),
            const SizedBox(height: 16),
            TextFormField(
              keyboardType: TextInputType.number,
              inputFormatters: <TextInputFormatter>[
                FilteringTextInputFormatter.digitsOnly,
                LengthLimitingTextInputFormatter(3),
              ],
              decoration: const InputDecoration(
                labelText: 'Défense (0 à 999)',
                prefixIcon: Icon(Icons.shield),
                border: OutlineInputBorder(),
              ),
              validator: (String? v) => _valider(v, 0, 999),
              onSaved: (String? v) => _defense = int.parse(v!.trim()),
            ),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: _calculer,
              child: const Text('Calculer les dégâts'),
            ),
            const SizedBox(height: 24),
            Text(
              _resultat == null
                  ? 'Aucun calcul effectué.'
                  : 'Dégâts infligés : $_resultat',
              textAlign: TextAlign.center,
              style: const TextStyle(fontSize: 22),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Explication :** le validateur `_valider` est factorisé et paramétré par les bornes, ce qui évite d'écrire deux fois la même logique. Il utilise `int.tryParse`, jamais `int.parse` : une saisie comme « abc » retourne `null` et produit un message au lieu d'une `FormatException`. Les `inputFormatters` empêchent physiquement la saisie de lettres et limitent à trois chiffres, mais ils ne remplacent pas le validateur, qui reste seul capable de refuser un champ vide ou un `0` pour l'attaque. Dans `onSaved`, `int.parse` est sûr car la validation a déjà réussi.

---

### Correction 8

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageInventaire(),
    );
  }
}

class PageInventaire extends StatefulWidget {
  const PageInventaire({super.key});

  @override
  State<PageInventaire> createState() => _PageInventaireState();
}

class _PageInventaireState extends State<PageInventaire> {
  final List<String> _objets = <String>[
    'Épée longue',
    'Potion de soin',
    'Clé rouillée',
    'Parchemin de feu',
    'Bouclier de chêne',
  ];

  void _supprimer(int index) {
    final String objet = _objets[index];
    setState(() => _objets.removeAt(index));

    ScaffoldMessenger.of(context)
      ..hideCurrentSnackBar()
      ..showSnackBar(
        SnackBar(
          content: Text('« $objet » supprimé.'),
          duration: const Duration(seconds: 5),
          behavior: SnackBarBehavior.floating,
          action: SnackBarAction(
            label: 'Annuler',
            onPressed: () {
              // On borne l'index : la liste a pu changer entre-temps.
              final int position =
                  index <= _objets.length ? index : _objets.length;
              setState(() => _objets.insert(position, objet));
            },
          ),
        ),
      );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Inventaire (${_objets.length})')),
      body: _objets.isEmpty
          ? const Center(child: Text('Inventaire vide.'))
          : ListView.separated(
              itemCount: _objets.length,
              separatorBuilder: (BuildContext c, int i) =>
                  const Divider(height: 1),
              itemBuilder: (BuildContext context, int index) {
                return ListTile(
                  leading: CircleAvatar(child: Text('${index + 1}')),
                  title: Text(_objets[index]),
                  trailing: IconButton(
                    icon: const Icon(Icons.delete_outline),
                    tooltip: 'Supprimer',
                    color: Colors.red,
                    onPressed: () => _supprimer(index),
                  ),
                );
              },
            ),
    );
  }
}
```

**Explication :** la suppression mémorise l'objet **et** son index avant de le retirer, ce qui permet à l'action « Annuler » de le réinsérer à sa position d'origine avec `insert`. L'appel à `hideCurrentSnackBar()` avant `showSnackBar` évite l'empilement de bandeaux quand l'utilisateur supprime plusieurs objets rapidement. Le calcul `index <= _objets.length ? index : _objets.length` protège contre un `RangeError` si d'autres suppressions ont raccourci la liste entre-temps.

---

### Correction 9

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const PageRaid(),
    );
  }
}

class PageRaid extends StatefulWidget {
  const PageRaid({super.key});

  @override
  State<PageRaid> createState() => _PageRaidState();
}

class _PageRaidState extends State<PageRaid> {
  final TextEditingController _nom = TextEditingController();
  DateTime? _date;
  TimeOfDay? _heure;

  @override
  void initState() {
    super.initState();
    // Le bouton dépend du contenu du champ : on écoute le contrôleur.
    _nom.addListener(_rafraichir);
  }

  void _rafraichir() => setState(() {});

  @override
  void dispose() {
    _nom.removeListener(_rafraichir);
    _nom.dispose();
    super.dispose();
  }

  String get _dateTexte => _date == null
      ? 'Aucune date'
      : '${_date!.day.toString().padLeft(2, '0')}/'
          '${_date!.month.toString().padLeft(2, '0')}/${_date!.year}';

  String get _heureTexte => _heure == null
      ? 'Aucune heure'
      : '${_heure!.hour.toString().padLeft(2, '0')}h'
          '${_heure!.minute.toString().padLeft(2, '0')}';

  bool get _pretAPlanifier =>
      _nom.text.trim().isNotEmpty && _date != null && _heure != null;

  Future<void> _choisirDate() async {
    final DateTime aujourdHui = DateTime.now();
    final DateTime? d = await showDatePicker(
      context: context,
      initialDate: _date ?? aujourdHui,
      firstDate: aujourdHui,
      lastDate: aujourdHui.add(const Duration(days: 365)),
      helpText: 'Date du raid',
    );
    if (!mounted || d == null) {
      return;
    }
    setState(() => _date = d);
  }

  Future<void> _choisirHeure() async {
    final TimeOfDay? h = await showTimePicker(
      context: context,
      initialTime: _heure ?? const TimeOfDay(hour: 21, minute: 0),
      helpText: 'Heure du raid',
    );
    if (!mounted || h == null) {
      return;
    }
    setState(() => _heure = h);
  }

  Future<void> _planifier() async {
    final bool? ok = await showDialog<bool>(
      context: context,
      builder: (BuildContext c) => AlertDialog(
        icon: const Icon(Icons.event_available),
        title: const Text('Planifier ce raid ?'),
        content: Text('${_nom.text.trim()}\n$_dateTexte à $_heureTexte'),
        actions: <Widget>[
          TextButton(
            onPressed: () => Navigator.pop(c, false),
            child: const Text('Annuler'),
          ),
          FilledButton(
            onPressed: () => Navigator.pop(c, true),
            child: const Text('Planifier'),
          ),
        ],
      ),
    );
    if (!mounted || ok != true) {
      return;
    }
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text('Raid « ${_nom.text.trim()} » planifié.'),
        backgroundColor: Colors.green,
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Planifier un raid')),
      body: ListView(
        padding: const EdgeInsets.all(24),
        children: <Widget>[
          TextField(
            controller: _nom,
            textCapitalization: TextCapitalization.sentences,
            decoration: const InputDecoration(
              labelText: 'Nom du raid',
              prefixIcon: Icon(Icons.flag),
              border: OutlineInputBorder(),
            ),
          ),
          const SizedBox(height: 16),
          ListTile(
            leading: const Icon(Icons.calendar_month),
            title: const Text('Date'),
            subtitle: Text(_dateTexte),
            trailing: const Icon(Icons.chevron_right),
            onTap: _choisirDate,
          ),
          ListTile(
            leading: const Icon(Icons.schedule),
            title: const Text('Heure'),
            subtitle: Text(_heureTexte),
            trailing: const Icon(Icons.chevron_right),
            onTap: _choisirHeure,
          ),
          const SizedBox(height: 24),
          FilledButton(
            onPressed: _pretAPlanifier ? _planifier : null,
            child: const Text('Planifier'),
          ),
        ],
      ),
    );
  }
}
```

**Explication :** le bouton dépend de trois valeurs, dont le contenu d'un champ texte ; comme un `TextEditingController` ne redessine rien à lui seul, on lui ajoute un auditeur qui déclenche un `setState` vide à chaque frappe, sinon le bouton resterait grisé après la saisie du nom. Le getter `_pretAPlanifier` centralise la condition d'activation. Chaque `await` est suivi de `if (!mounted) return;` avant tout usage de `context` ou `setState`, ce qui évite l'avertissement `use_build_context_synchronously` et les plantages si l'écran est quitté pendant qu'une boîte est ouverte. `firstDate: aujourdHui` empêche de planifier un raid dans le passé.

---

### Correction 10

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class Valideurs {
  const Valideurs._();

  static String? Function(String?) requis([String m = 'Champ obligatoire.']) =>
      (String? v) => (v == null || v.trim().isEmpty) ? m : null;

  static String? Function(String?) longueur(int min, int max) => (String? v) {
        final String t = (v ?? '').trim();
        if (t.isEmpty) {
          return null;
        }
        if (t.length < min) {
          return 'Au moins $min caractères.';
        }
        if (t.length > max) {
          return 'Au plus $max caractères.';
        }
        return null;
      };

  static String? email(String? v) {
    final String t = (v ?? '').trim();
    if (t.isEmpty) {
      return null;
    }
    return RegExp(r'^[\w.+-]+@[\w-]+\.[\w.-]{2,}$').hasMatch(t)
        ? null
        : 'Adresse e-mail invalide.';
  }

  static String? motDePasse(String? v) {
    final String t = v ?? '';
    if (t.isEmpty) {
      return null;
    }
    if (t.length < 8) {
      return 'Au moins 8 caractères.';
    }
    if (!t.contains(RegExp(r'[0-9]'))) {
      return 'Doit contenir au moins un chiffre.';
    }
    return null;
  }

  static String? Function(String?) tous(
    List<String? Function(String?)> liste,
  ) =>
      (String? v) {
        for (final String? Function(String?) f in liste) {
          final String? e = f(v);
          if (e != null) {
            return e;
          }
        }
        return null;
      };
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorSchemeSeed: Colors.teal,
        useMaterial3: true,
        inputDecorationTheme: const InputDecorationTheme(
          border: OutlineInputBorder(),
          filled: true,
        ),
      ),
      home: const PageInscription(),
    );
  }
}

class PageInscription extends StatefulWidget {
  const PageInscription({super.key});

  @override
  State<PageInscription> createState() => _PageInscriptionState();
}

class _PageInscriptionState extends State<PageInscription> {
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();
  final TextEditingController _pseudo = TextEditingController();
  final TextEditingController _email = TextEditingController();
  final TextEditingController _mdp = TextEditingController();
  final TextEditingController _confirmation = TextEditingController();

  static const List<String> _pays = <String>[
    'France', 'Belgique', 'Suisse', 'Canada', 'Maroc', 'Sénégal',
  ];
  String? _paysChoisi;
  bool _conditions = false;
  bool _masque = true;
  AutovalidateMode _mode = AutovalidateMode.disabled;

  @override
  void dispose() {
    _pseudo.dispose();
    _email.dispose();
    _mdp.dispose();
    _confirmation.dispose();
    super.dispose();
  }

  Future<void> _sInscrire() async {
    FocusScope.of(context).unfocus();
    if (!_formKey.currentState!.validate()) {
      setState(() => _mode = AutovalidateMode.onUserInteraction);
      return;
    }
    await showModalBottomSheet<void>(
      context: context,
      showDragHandle: true,
      builder: (BuildContext c) => SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(24),
          child: Column(
            mainAxisSize: MainAxisSize.min,
            crossAxisAlignment: CrossAxisAlignment.start,
            children: <Widget>[
              Text('Inscription confirmée',
                  style: Theme.of(c).textTheme.headlineSmall),
              const SizedBox(height: 16),
              Text(
                'Pseudo : ${_pseudo.text.trim()}\n'
                'E-mail : ${_email.text.trim()}\n'
                'Pays   : $_paysChoisi',
                style: const TextStyle(fontFamily: 'monospace'),
              ),
              const SizedBox(height: 24),
              SizedBox(
                width: double.infinity,
                child: FilledButton(
                  onPressed: () => Navigator.pop(c),
                  child: const Text('Fermer'),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Inscription')),
      body: Form(
        key: _formKey,
        autovalidateMode: _mode,
        child: ListView(
          padding: const EdgeInsets.all(24),
          children: <Widget>[
            TextFormField(
              controller: _pseudo,
              textInputAction: TextInputAction.next,
              decoration: const InputDecoration(
                labelText: 'Pseudo',
                prefixIcon: Icon(Icons.person),
              ),
              validator: Valideurs.tous(<String? Function(String?)>[
                Valideurs.requis('Le pseudo est obligatoire.'),
                Valideurs.longueur(3, 15),
              ]),
            ),
            const SizedBox(height: 16),
            TextFormField(
              controller: _email,
              keyboardType: TextInputType.emailAddress,
              textInputAction: TextInputAction.next,
              decoration: const InputDecoration(
                labelText: 'Adresse e-mail',
                prefixIcon: Icon(Icons.email),
              ),
              validator: Valideurs.tous(<String? Function(String?)>[
                Valideurs.requis(),
                Valideurs.email,
              ]),
            ),
            const SizedBox(height: 16),
            TextFormField(
              controller: _mdp,
              obscureText: _masque,
              textInputAction: TextInputAction.next,
              decoration: InputDecoration(
                labelText: 'Mot de passe',
                prefixIcon: const Icon(Icons.lock),
                helperText: '8 caractères minimum, dont un chiffre',
                suffixIcon: IconButton(
                  icon: Icon(
                    _masque ? Icons.visibility : Icons.visibility_off,
                  ),
                  onPressed: () => setState(() => _masque = !_masque),
                ),
              ),
              validator: Valideurs.tous(<String? Function(String?)>[
                Valideurs.requis(),
                Valideurs.motDePasse,
              ]),
            ),
            const SizedBox(height: 16),
            TextFormField(
              controller: _confirmation,
              obscureText: _masque,
              textInputAction: TextInputAction.done,
              decoration: const InputDecoration(
                labelText: 'Confirmer le mot de passe',
                prefixIcon: Icon(Icons.lock_outline),
              ),
              validator: (String? v) {
                if (v == null || v.isEmpty) {
                  return 'Confirmation obligatoire.';
                }
                if (v != _mdp.text) {
                  return 'Les mots de passe ne correspondent pas.';
                }
                return null;
              },
            ),
            const SizedBox(height: 16),
            DropdownButtonFormField<String>(
              initialValue: _paysChoisi,
              decoration: const InputDecoration(
                labelText: 'Pays',
                prefixIcon: Icon(Icons.public),
              ),
              hint: const Text('Choisissez un pays'),
              items: _pays
                  .map((String p) =>
                      DropdownMenuItem<String>(value: p, child: Text(p)))
                  .toList(),
              onChanged: (String? v) => setState(() => _paysChoisi = v),
              validator: (String? v) =>
                  v == null ? 'Le pays est obligatoire.' : null,
            ),
            const SizedBox(height: 8),
            CheckboxListTile(
              value: _conditions,
              title: const Text("J'accepte les conditions d'utilisation"),
              controlAffinity: ListTileControlAffinity.leading,
              onChanged: (bool? v) =>
                  setState(() => _conditions = v ?? false),
            ),
            const SizedBox(height: 16),
            SizedBox(
              height: 52,
              child: FilledButton(
                onPressed: _conditions ? _sInscrire : null,
                child: const Text("S'inscrire"),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Explication :** la classe `Valideurs` supprime toute duplication : chaque règle est testée une seule fois et les règles se combinent avec `tous`, qui retourne la première erreur rencontrée. Le validateur de confirmation est le seul à ne pas être réutilisable, car il compare avec un **autre** champ : il lit directement `_mdp.text`, ce qui est possible parce que les deux champs ont un contrôleur. Le mode de validation démarre à `disabled` pour ne pas afficher de rouge à l'ouverture, puis bascule sur `onUserInteraction` dès le premier échec afin de guider l'utilisateur en direct. Enfin, la case des conditions n'est pas un `FormField` : elle n'est donc pas vérifiée par `validate()`, mais elle conditionne l'activation du bouton via `onPressed: _conditions ? _sInscrire : null`.

---

## Et maintenant ?

Vos écrans savent désormais **recevoir** des données autant qu'en afficher. Vous connaissez les cinq boutons Material 3, la désactivation par `onPressed: null`, `TextField` et son contrôleur, le cycle `initState` / `dispose`, le trio `Form` + `TextFormField` + `validator`, tous les widgets de choix, et les trois manières de rendre la main à l'utilisateur : `SnackBar`, `AlertDialog`, `showModalBottomSheet`.

Il manque une chose, et elle est énorme : votre application n'a toujours **qu'un seul écran**. Le formulaire de création de personnage devrait mener à une fiche de personnage, puis à la liste de l'équipe, avec un bouton de retour et une transition animée.

Le chapitre 50 traite exactement cela : `Navigator.push` et `Navigator.pop`, le passage de données vers l'écran suivant, la **récupération** d'un résultat au retour (ce qui remplacera avantageusement la feuille modale du mini-projet), les routes nommées, les onglets avec `TabBar`, le menu latéral `Drawer`, la barre de navigation inférieure, et le routage moderne avec `go_router`.

À la fin du chapitre 50, votre formulaire de création rendra un vrai `Personnage` à l'écran précédent, qui l'ajoutera à sa liste.

Rendez-vous au chapitre 50 : [50-PARTIE-1B—NAVIGATION-ET-ROUTES.md](./50-PARTIE-1B—NAVIGATION-ET-ROUTES.md)
