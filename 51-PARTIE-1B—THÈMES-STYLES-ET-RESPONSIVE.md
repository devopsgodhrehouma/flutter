# PARTIE 1B — FLUTTER
# CHAPITRE 51 — THÈMES, MATERIAL 3 ET INTERFACES ADAPTATIVES

> **Niveau :** intermédiaire
> **Durée estimée :** 10 h
> **Pré-requis :** chapitre 50 — Navigation et routes
> **Ce que vous saurez faire à la fin :** définir un thème complet pour votre application, la faire basculer en mode sombre, et écrire une interface qui reste lisible et bien proportionnée du petit téléphone à l'écran de bureau.

---

## 51.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer pourquoi les styles écrits directement dans les widgets deviennent ingérables ;
- construire un `ThemeData` et le brancher sur `MaterialApp` ;
- dire ce que change `useMaterial3` et pourquoi c'est la valeur par défaut ;
- générer une palette cohérente avec `ColorScheme.fromSeed()` ;
- nommer les principaux rôles de couleur de Material 3 et savoir lequel utiliser ;
- lire le thème courant avec `Theme.of(context)` et `ColorScheme.of(context)` ;
- utiliser les quinze styles nommés de `textTheme` ;
- personnaliser un style de texte sans casser les autres ;
- configurer les thèmes de composants : boutons, `AppBar`, `Card`, champs de saisie ;
- fournir un `darkTheme` et piloter le tout avec `themeMode` ;
- suivre automatiquement le réglage clair/sombre du système ;
- laisser l'utilisateur choisir son thème depuis l'application ;
- surcharger le thème d'un seul sous-arbre avec le widget `Theme` ;
- ajouter vos propres couleurs au thème avec `ThemeExtension` ;
- centraliser vos espacements et vos rayons dans un fichier de constantes ;
- interroger l'écran avec `MediaQuery` : taille, orientation, `textScaler`, `padding` ;
- respecter le grossissement de police choisi par l'utilisateur ;
- mesurer l'espace réellement disponible avec `LayoutBuilder` ;
- choisir entre `MediaQuery` et `LayoutBuilder` en connaissance de cause ;
- poser des points de rupture mobile / tablette / bureau ;
- faire basculer une mise en page de `Column` à `Row` selon la largeur ;
- réagir à l'orientation avec `OrientationBuilder` ;
- ajuster un texte trop grand avec `FittedBox` ;
- afficher une grille dont le nombre de colonnes s'adapte ;
- afficher un `NavigationRail` sur grand écran ;
- gérer le clavier qui recouvre un champ de saisie ;
- éviter les encoches avec `SafeArea` ;
- respecter les règles de base d'accessibilité : contraste, cibles tactiles, `Semantics` ;
- tester votre interface sur plusieurs tailles d'écran sans changer d'appareil ;
- assembler une application complète qui s'adapte au téléphone, à la tablette et au mode sombre.

---

## 51.0.1 — Où en sommes-nous ?

Depuis le chapitre 43, vous avez appris à construire des écrans.

Vous savez placer (chapitre 46), afficher du texte et des images (chapitre 47), dérouler des listes (chapitre 48), saisir des données (chapitre 49) et naviguer entre les pages (chapitre 50).

Vos applications **fonctionnent**. Mais elles ne sont pas encore **finies**, pour deux raisons.

**Première raison : elles ne sont pas cohérentes.**
Chaque écran a été écrit indépendamment. Les bleus ne sont pas exactement le même bleu. Les titres n'ont pas tous la même taille. Les marges varient de 12 à 20 pixels sans logique. Le résultat sent l'amateur, même si le code est correct.

**Seconde raison : elles ne s'adaptent pas.**
Vous avez travaillé sur un seul émulateur. Sur un téléphone plus petit, le texte déborde. Sur une tablette, votre colonne unique s'étire ridiculement sur 1000 pixels de large. En mode sombre, votre texte noir devient invisible. Si l'utilisateur a grossi la police du système, tout casse.

Ce chapitre traite ces deux problèmes, dans cet ordre :

```text
  PARTIE A (51.1 → 51.16)  : LE THÈME
      « Toute mon application partage la même identité visuelle. »

  PARTIE B (51.17 → 51.32) : L'ADAPTATION
      « Mon application reste lisible sur tous les écrans. »

  PARTIE C (51.33)         : LE MINI-PROJET
      Les deux ensemble.
```

---

## 51.1 — Le problème des styles copiés-collés partout

Commençons par le problème, avant la solution.

Voici une petite application écrite « à la main », comme on le fait naturellement quand on découvre Flutter.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const AppSansTheme());
}

class AppSansTheme extends StatelessWidget {
  const AppSansTheme({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      title: 'Sans thème',
      home: PageInventaire(),
    );
  }
}

class PageInventaire extends StatelessWidget {
  const PageInventaire({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF5F5F5),
      appBar: AppBar(
        backgroundColor: const Color(0xFF3F51B5),
        foregroundColor: Colors.white,
        title: const Text(
          'Inventaire',
          style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
        ),
      ),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          Container(
            padding: const EdgeInsets.all(16),
            margin: const EdgeInsets.only(bottom: 12),
            decoration: BoxDecoration(
              color: Colors.white,
              borderRadius: BorderRadius.circular(12),
            ),
            child: const Text(
              'Épée en fer',
              style: TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.w600,
                color: Color(0xFF212121),
              ),
            ),
          ),
          Container(
            padding: const EdgeInsets.all(16),
            margin: const EdgeInsets.only(bottom: 12),
            decoration: BoxDecoration(
              color: Colors.white,
              borderRadius: BorderRadius.circular(12),
            ),
            child: const Text(
              'Potion de soin',
              style: TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.w600,
                color: Color(0xFF212121),
              ),
            ),
          ),
          Container(
            padding: const EdgeInsets.all(16),
            margin: const EdgeInsets.only(bottom: 12),
            decoration: BoxDecoration(
              color: Colors.white,
              borderRadius: BorderRadius.circular(10),
            ),
            child: const Text(
              'Bouclier de bois',
              style: TextStyle(
                fontSize: 17,
                fontWeight: FontWeight.w600,
                color: Color(0xFF222222),
              ),
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
┌──────────────────────────────────┐
│  Inventaire            (bleu)    │
├──────────────────────────────────┤
│  ┌────────────────────────────┐  │
│  │ Épée en fer                │  │
│  └────────────────────────────┘  │
│  ┌────────────────────────────┐  │
│  │ Potion de soin             │  │
│  └────────────────────────────┘  │
│  ┌────────────────────────────┐  │
│  │ Bouclier de bois           │  │
│  └────────────────────────────┘  │
└──────────────────────────────────┘
```

Visuellement, c'est acceptable. Mais lisez le code de près.

**Problème 1 — la duplication.**
`fontSize: 18`, `fontWeight: FontWeight.w600`, `Color(0xFF212121)` sont écrits trois fois. Sur une vraie application, ce sera trois cents fois.

**Problème 2 — la dérive.**
Regardez la troisième carte : `fontSize: 17` au lieu de 18, `Color(0xFF222222)` au lieu de `0xFF212121`, `borderRadius: 10` au lieu de 12. Ces écarts sont invisibles à l'œil nu mais bien réels. Ils s'accumulent. C'est ce qu'on appelle la **dérive du design**.

**Problème 3 — le changement impossible.**
Votre client demande : « le bleu est trop foncé, mettez du vert ». Vous devez trouver et remplacer `0xFF3F51B5` dans tous les fichiers. Vous en oublierez un.

**Problème 4 — le mode sombre inexistant.**
`color: Colors.white` sur les cartes et `Color(0xFF212121)` sur le texte sont **codés en dur**. En mode sombre, il faudrait l'inverse. Ici, c'est impossible sans réécrire chaque widget.

**Problème 5 — l'accessibilité perdue.**
Un `fontSize: 18` en dur ne dit rien sur le rôle du texte. Est-ce un titre ? Une légende ? Les outils d'accessibilité ne peuvent pas le savoir.

> La règle à retenir : **si une valeur de style apparaît deux fois dans votre code, elle ne doit apparaître nulle part.** Elle doit vivre dans le thème.

---

## 51.2 — `ThemeData` : le catalogue de styles de l'application

`ThemeData` est un objet qui décrit **l'apparence complète** de votre application : ses couleurs, ses polices, la forme de ses boutons, l'aspect de ses cartes, etc.

On le donne une seule fois, à `MaterialApp`, via le paramètre `theme` :

```dart
MaterialApp(
  theme: ThemeData( /* ... */ ),
  home: /* ... */,
)
```

À partir de là, **tous** les widgets Material situés sous ce `MaterialApp` peuvent le consulter avec `Theme.of(context)`.

Voici la même page que ci-dessus, mais pilotée par un thème.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const AppAvecTheme());
}

class AppAvecTheme extends StatelessWidget {
  const AppAvecTheme({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Avec thème',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF3F51B5)),
      ),
      home: const PageInventaire(),
    );
  }
}

class PageInventaire extends StatelessWidget {
  const PageInventaire({super.key});

  static const List<String> objets = <String>[
    'Épée en fer',
    'Potion de soin',
    'Bouclier de bois',
  ];

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Inventaire'),
        backgroundColor: theme.colorScheme.primaryContainer,
      ),
      body: ListView.builder(
        padding: const EdgeInsets.all(16),
        itemCount: objets.length,
        itemBuilder: (BuildContext context, int index) {
          return Card(
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Text(
                objets[index],
                style: theme.textTheme.titleMedium,
              ),
            ),
          );
        },
      ),
    );
  }
}
```

**Résultat :**

```text
Trois cartes identiques, aux coins arrondis, dont la couleur
de fond, la couleur de texte et l'élévation viennent du thème.
Aucune couleur n'est écrite dans la page.
```

Comparez les deux versions :

| Point | Sans thème | Avec thème |
| --- | --- | --- |
| Couleurs écrites dans la page | 6 | 0 |
| Tailles de texte écrites dans la page | 4 | 0 |
| Changer la couleur principale | 6 remplacements | 1 seul mot |
| Mode sombre | à réécrire entièrement | quasi gratuit (51.10) |

---

## 51.2.1 — Ce que contient réellement un `ThemeData`

`ThemeData` a beaucoup de champs. Vous n'avez pas besoin de tous les connaître. Voici les familles.

```text
ThemeData
│
├── colorScheme ............... toutes les couleurs, par rôle       (51.4, 51.5)
├── brightness ............... clair ou sombre (déduit du colorScheme)
├── useMaterial3 ............. version du design system             (51.3)
│
├── textTheme ................ 15 styles de texte nommés            (51.7)
├── primaryTextTheme ......... styles pour un fond de couleur primaire
│
├── appBarTheme .............. apparence de toutes les AppBar       (51.9)
├── cardTheme ................ apparence de toutes les Card         (51.9)
├── elevatedButtonTheme ...... apparence des ElevatedButton         (51.9)
├── filledButtonTheme ........ apparence des FilledButton
├── outlinedButtonTheme ...... apparence des OutlinedButton
├── textButtonTheme .......... apparence des TextButton
├── inputDecorationTheme ..... apparence des champs de saisie       (51.9)
├── floatingActionButtonTheme  apparence des FAB
├── dialogTheme .............. apparence des dialogues
├── snackBarTheme ............ apparence des SnackBar
├── ... (une trentaine d'autres)
│
├── visualDensity ............ compacité générale des composants
├── splashFactory ............ animation au toucher
│
└── extensions ............... VOS propres jeux de valeurs          (51.15)
```

> Retenez la logique : **un champ `xxxTheme` pour chaque famille de composants.** Si vous cherchez comment styler globalement un widget `Truc`, cherchez `trucTheme` dans `ThemeData`.

---

## 51.2.2 — `ThemeData` est immuable : `copyWith`

`ThemeData` est une classe immuable, comme la plupart des objets Flutter. On ne modifie pas un thème existant : on en **fabrique une copie modifiée** avec `copyWith`.

```dart
import 'package:flutter/material.dart';

void main() {
  final ThemeData base = ThemeData(
    colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
  );

  final ThemeData compact = base.copyWith(
    visualDensity: VisualDensity.compact,
  );

  print(base.visualDensity == compact.visualDensity);
  print(base.colorScheme.primary == compact.colorScheme.primary);
}
```

**Résultat :**

```text
false
true
```

La copie ne change que ce qu'on lui demande. Tout le reste est conservé à l'identique.

C'est exactement le mécanisme du `copyWith` que vous avez rencontré sur vos propres classes au chapitre 09.

---

## 51.3 — Material 3 : `useMaterial3`

Material Design est le langage visuel de Google. Il en existe plusieurs versions.

- **Material 2** (2018) : les couleurs vives, les boutons rectangulaires, les ombres marquées.
- **Material 3** (2021, aussi appelé « Material You ») : les coins très arrondis, les couleurs générées à partir d'une couleur de base, les teintes de surface, les composants plus aérés.

Flutter contrôle cela avec un unique booléen :

```dart
ThemeData(
  useMaterial3: true,
)
```

**Depuis Flutter 3.16, `useMaterial3` vaut `true` par défaut.** Vous n'avez donc rien à écrire : vous êtes déjà en Material 3.

La documentation officielle précise que `useMaterial3` et le support de Material 2 finiront par être supprimés. **Écrivez vos nouvelles applications en Material 3 et ne mettez jamais `useMaterial3: false` dans un projet neuf.**

Voici un programme qui affiche les deux côte à côte pour que vous voyiez la différence.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ComparaisonMaterial());
}

class ComparaisonMaterial extends StatelessWidget {
  const ComparaisonMaterial({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'M2 vs M3',
      home: Scaffold(
        appBar: AppBar(title: const Text('Material 2 / Material 3')),
        body: Row(
          children: <Widget>[
            Expanded(
              child: Theme(
                data: ThemeData(
                  useMaterial3: false,
                  colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
                ),
                child: const _Vitrine(titre: 'Material 2'),
              ),
            ),
            const VerticalDivider(width: 1),
            Expanded(
              child: Theme(
                data: ThemeData(
                  useMaterial3: true,
                  colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
                ),
                child: const _Vitrine(titre: 'Material 3'),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class _Vitrine extends StatelessWidget {
  const _Vitrine({required this.titre});

  final String titre;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return Material(
      color: theme.colorScheme.surface,
      child: Padding(
        padding: const EdgeInsets.all(12),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            Text(titre, style: theme.textTheme.titleMedium),
            const SizedBox(height: 16),
            ElevatedButton(onPressed: () {}, child: const Text('Attaquer')),
            const SizedBox(height: 8),
            OutlinedButton(onPressed: () {}, child: const Text('Fuir')),
            const SizedBox(height: 8),
            TextButton(onPressed: () {}, child: const Text('Inventaire')),
            const SizedBox(height: 16),
            const Card(
              child: ListTile(
                leading: Icon(Icons.shield),
                title: Text('Bouclier'),
              ),
            ),
            const SizedBox(height: 16),
            Slider(value: 0.6, onChanged: (double v) {}),
            const SizedBox(height: 8),
            const LinearProgressIndicator(value: 0.4),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
À gauche (M2)                     À droite (M3)
─────────────                     ─────────────
Boutons peu arrondis              Boutons en gélule (stadium)
Ombres franches                   Élévation par teinte de surface
Card blanche et grise             Card teintée par la couleur de base
Slider fin, pastille ronde        Slider épais, pastille allongée
```

> Le widget `Theme` utilisé ici pour surcharger localement le thème est expliqué en détail en 51.14.

---

## 51.3.1 — Ce que Material 3 change concrètement

| Aspect | Material 2 | Material 3 |
| --- | --- | --- |
| Source des couleurs | `primarySwatch`, couleurs choisies à la main | `ColorScheme.fromSeed()` génère tout |
| Nombre de rôles de couleur | environ 12 | environ 45 |
| Élévation | ombre portée | teinte de surface + ombre discrète |
| Coins des boutons | 4 px | forme en gélule |
| Barre de navigation basse | `BottomNavigationBar` | `NavigationBar` |
| Typographie | `headline1`, `bodyText1`… | `displayLarge`, `bodyLarge`… |
| Grands écrans | rien de prévu | `NavigationRail`, `NavigationDrawer` |

Les anciens noms (`primarySwatch`, `headline1`, `accentColor`) sont dépréciés ou supprimés. Si vous trouvez un tutoriel qui les utilise, il date d'avant 2022 : ne le suivez pas.

---

## 51.4 — `ColorScheme.fromSeed()` : une palette entière depuis une seule couleur

C'est l'idée centrale de Material 3.

Vous donnez **une** couleur. Flutter calcule **toute** une palette harmonieuse autour d'elle : la couleur principale ajustée pour être lisible, une couleur secondaire, une tertiaire, les fonds, les textes, les bordures, les erreurs, et les couleurs de texte à poser dessus.

La signature exacte, vérifiée dans la documentation officielle, commence ainsi :

```dart
ColorScheme.fromSeed({
  required Color seedColor,
  Brightness brightness = Brightness.light,
  DynamicSchemeVariant dynamicSchemeVariant = DynamicSchemeVariant.tonalSpot,
  double contrastLevel = 0.0,
  Color? primary,
  Color? onPrimary,
  // ... une quarantaine de couleurs optionnelles pour forcer un rôle précis
})
```

Trois paramètres suffisent dans 95 % des cas :

- `seedColor` : votre couleur de marque. **C'est le seul paramètre obligatoire.**
- `brightness` : `Brightness.light` (défaut) ou `Brightness.dark`.
- `dynamicSchemeVariant` : le style d'harmonie utilisé pour dériver la palette.

Voici un programme qui affiche la palette générée.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DemoSeed());
}

class DemoSeed extends StatelessWidget {
  const DemoSeed({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'ColorScheme.fromSeed',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF6750A4)),
      ),
      home: const PagePalette(),
    );
  }
}

class PagePalette extends StatelessWidget {
  const PagePalette({super.key});

  @override
  Widget build(BuildContext context) {
    final ColorScheme c = Theme.of(context).colorScheme;

    final List<List<Object>> roles = <List<Object>>[
      <Object>['primary', c.primary, c.onPrimary],
      <Object>['primaryContainer', c.primaryContainer, c.onPrimaryContainer],
      <Object>['secondary', c.secondary, c.onSecondary],
      <Object>['secondaryContainer', c.secondaryContainer, c.onSecondaryContainer],
      <Object>['tertiary', c.tertiary, c.onTertiary],
      <Object>['tertiaryContainer', c.tertiaryContainer, c.onTertiaryContainer],
      <Object>['error', c.error, c.onError],
      <Object>['errorContainer', c.errorContainer, c.onErrorContainer],
      <Object>['surface', c.surface, c.onSurface],
      <Object>['surfaceContainerHighest', c.surfaceContainerHighest, c.onSurfaceVariant],
      <Object>['inverseSurface', c.inverseSurface, c.onInverseSurface],
    ];

    return Scaffold(
      appBar: AppBar(title: const Text('Palette générée')),
      body: ListView.builder(
        itemCount: roles.length,
        itemBuilder: (BuildContext context, int index) {
          final String nom = roles[index][0] as String;
          final Color fond = roles[index][1] as Color;
          final Color texte = roles[index][2] as Color;
          return Container(
            height: 56,
            color: fond,
            alignment: Alignment.centerLeft,
            padding: const EdgeInsets.symmetric(horizontal: 16),
            child: Text(nom, style: TextStyle(color: texte, fontSize: 16)),
          );
        },
      ),
    );
  }
}
```

**Résultat :**

```text
Onze bandes empilées. Chaque bande porte son nom de rôle,
écrit dans la couleur « on… » correspondante.
Le texte est toujours parfaitement lisible sur son fond :
c'est la garantie que Material 3 vous offre.
```

Changez `const Color(0xFF6750A4)` en `Colors.red`, `Colors.green`, `Colors.orange`. Toute la palette suit, et le texte reste lisible dans tous les cas.

---

## 51.4.1 — La couleur de base n'est presque jamais la couleur `primary`

Un point qui surprend systématiquement les débutants :

```dart
final ColorScheme c = ColorScheme.fromSeed(seedColor: const Color(0xFFFF0000));
// c.primary N'EST PAS 0xFFFF0000
```

La graine est une **indication de teinte**, pas une couleur imposée. Flutter la convertit dans un espace de couleur perceptuel, en extrait la teinte, puis reconstruit une gamme de tons calibrés pour garantir les contrastes.

Un rouge pur `#FF0000` est trop saturé et trop clair pour porter du texte blanc de façon lisible. Flutter produira donc un rouge plus sombre et plus posé.

Si vous devez absolument imposer une couleur exacte (une charte graphique d'entreprise, par exemple), forcez le rôle :

```dart
ColorScheme.fromSeed(
  seedColor: const Color(0xFFFF0000),
  primary: const Color(0xFFFF0000),   // on impose
  onPrimary: Colors.white,            // à vous de garantir le contraste
)
```

> Attention : dès que vous forcez un rôle, **vous** devenez responsable du contraste. Flutter ne le vérifie plus pour vous.

---

## 51.4.2 — Les variantes de schéma

Le paramètre `dynamicSchemeVariant` change la façon dont la palette est dérivée. Les valeurs de l'énumération `DynamicSchemeVariant` incluent notamment `tonalSpot` (le défaut), `fidelity`, `monochrome`, `neutral`, `vibrant`, `expressive`, `content`, `rainbow` et `fruitSalad`.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DemoVariantes());
}

class DemoVariantes extends StatelessWidget {
  const DemoVariantes({super.key});

  @override
  Widget build(BuildContext context) {
    const List<DynamicSchemeVariant> variantes = <DynamicSchemeVariant>[
      DynamicSchemeVariant.tonalSpot,
      DynamicSchemeVariant.fidelity,
      DynamicSchemeVariant.vibrant,
      DynamicSchemeVariant.expressive,
      DynamicSchemeVariant.neutral,
      DynamicSchemeVariant.monochrome,
    ];

    return MaterialApp(
      title: 'Variantes',
      home: Scaffold(
        appBar: AppBar(title: const Text('DynamicSchemeVariant')),
        body: ListView(
          children: variantes.map((DynamicSchemeVariant v) {
            final ColorScheme c = ColorScheme.fromSeed(
              seedColor: const Color(0xFF2E7D32),
              dynamicSchemeVariant: v,
            );
            return Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 6),
              child: Row(
                children: <Widget>[
                  SizedBox(width: 130, child: Text(v.name)),
                  _Pastille(couleur: c.primary),
                  _Pastille(couleur: c.secondary),
                  _Pastille(couleur: c.tertiary),
                  _Pastille(couleur: c.primaryContainer),
                  _Pastille(couleur: c.surfaceContainerHighest),
                ],
              ),
            );
          }).toList(),
        ),
      ),
    );
  }
}

class _Pastille extends StatelessWidget {
  const _Pastille({required this.couleur});

  final Color couleur;

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 34,
      height: 34,
      margin: const EdgeInsets.only(right: 6),
      decoration: BoxDecoration(
        color: couleur,
        borderRadius: BorderRadius.circular(6),
        border: Border.all(color: Colors.black26),
      ),
    );
  }
}
```

**Résultat :**

```text
tonalSpot     ■ ■ ■ ■ ■   palette équilibrée, secondaire désaturée
fidelity      ■ ■ ■ ■ ■   reste très proche de la graine
vibrant       ■ ■ ■ ■ ■   couleurs plus saturées
expressive    ■ ■ ■ ■ ■   secondaire et tertiaire décalées en teinte
neutral       ■ ■ ■ ■ ■   presque gris
monochrome    ■ ■ ■ ■ ■   noir et blanc uniquement
```

En pratique : gardez `tonalSpot` par défaut, essayez `fidelity` si votre client veut « exactement sa couleur », et `monochrome` pour un thème très sobre.

---

## 51.5 — Les rôles de couleur de Material 3

Material 3 ne raisonne pas en « bleu », « gris clair », « rouge ». Il raisonne en **rôles**.

Un rôle répond à la question : « à quoi sert cette couleur ? », pas « quelle couleur est-ce ? ».

Les rôles vont par paires : une couleur de fond `xxx`, et la couleur de contenu `onXxx` garantie lisible dessus.

```text
        ┌──────────────────────────────┐
        │  fond = primary              │
        │                              │
        │   texte = onPrimary          │   <- contraste garanti
        │                              │
        └──────────────────────────────┘
```

### Tableau des rôles principaux

| Rôle | Usage typique | Couleur de contenu associée |
| --- | --- | --- |
| `primary` | action principale, éléments les plus importants (FAB, `FilledButton`) | `onPrimary` |
| `onPrimary` | texte et icônes posés sur `primary` | — |
| `primaryContainer` | conteneur mis en avant, mais moins fort que `primary` | `onPrimaryContainer` |
| `onPrimaryContainer` | texte posé sur `primaryContainer` | — |
| `secondary` | éléments d'appui, filtres, puces sélectionnées | `onSecondary` |
| `secondaryContainer` | fond des puces actives, sélection dans une liste | `onSecondaryContainer` |
| `tertiary` | accent de contraste, décoration, catégorisation | `onTertiary` |
| `tertiaryContainer` | fond d'accent secondaire | `onTertiaryContainer` |
| `error` | erreurs : bordure de champ invalide, icône d'alerte | `onError` |
| `errorContainer` | fond d'un bandeau d'erreur | `onErrorContainer` |
| `surface` | fond général des pages et des composants | `onSurface` |
| `onSurface` | texte principal | — |
| `onSurfaceVariant` | texte secondaire, icônes moins importantes | — |
| `surfaceContainerLowest` | la surface la plus basse (la plus proche du fond) | `onSurface` |
| `surfaceContainerLow` | surface légèrement élevée | `onSurface` |
| `surfaceContainer` | surface de référence des cartes | `onSurface` |
| `surfaceContainerHigh` | surface élevée (menus, feuilles) | `onSurface` |
| `surfaceContainerHighest` | la surface la plus élevée | `onSurface` |
| `surfaceDim` / `surfaceBright` | variantes assombrie / éclaircie de la surface | `onSurface` |
| `outline` | bordures visibles, séparateurs importants | — |
| `outlineVariant` | séparateurs discrets (`Divider`) | — |
| `inverseSurface` | fond inversé : `SnackBar`, infobulle | `onInverseSurface` |
| `inversePrimary` | action colorée posée sur `inverseSurface` | — |
| `shadow` | couleur des ombres portées | — |
| `scrim` | voile sombre derrière un dialogue ou un `Drawer` | — |
| `surfaceTint` | teinte appliquée aux surfaces selon leur élévation | — |

> Les rôles `background`, `onBackground` et `surfaceVariant` existent encore mais sont **dépréciés**. Utilisez respectivement `surface`, `onSurface` et `surfaceContainerHighest`.

---

## 51.5.1 — Comment choisir le bon rôle : l'arbre de décision

```text
                    Je dois colorer quelque chose
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
   C'est un FOND          C'est du TEXTE          C'est une BORDURE
        │                  ou une ICÔNE                  │
        │                       │                       │
   ┌────┴────┐             ┌────┴────┐             ┌─────┴─────┐
   │         │             │         │             │           │
Action    Zone de       Sur un    Sur une      Visible    Discrète
principale contenu      fond      surface      et forte
   │         │          coloré       │             │           │
   │         │             │      ┌──┴──┐          │           │
primary  surface        onXxx   Important        outline  outlineVariant
   ou       ou          du fond    │  Secondaire
primary-  surface-      utilisé    │      │
Container Container…              onSurface  onSurfaceVariant
```

Deux règles suffisent au quotidien :

1. **Si vous posez du texte sur `xxx`, utilisez `onXxx`.** Jamais `Colors.white` ou `Colors.black`.
2. **Dans le doute, utilisez `surface` et `onSurface`.** C'est le couple neutre par défaut.

---

## 51.5.2 — Un exemple appliqué au fil rouge

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DemoRoles());
}

class DemoRoles extends StatelessWidget {
  const DemoRoles({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Rôles de couleur',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF00695C)),
      ),
      home: const FicheJoueur(),
    );
  }
}

class FicheJoueur extends StatelessWidget {
  const FicheJoueur({super.key});

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final ColorScheme c = theme.colorScheme;

    return Scaffold(
      appBar: AppBar(title: const Text('Fiche du joueur')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: <Widget>[
          // Bandeau principal : primaryContainer + onPrimaryContainer
          Container(
            padding: const EdgeInsets.all(20),
            decoration: BoxDecoration(
              color: c.primaryContainer,
              borderRadius: BorderRadius.circular(16),
            ),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: <Widget>[
                Text(
                  'Aldric le Vaillant',
                  style: theme.textTheme.headlineSmall
                      ?.copyWith(color: c.onPrimaryContainer),
                ),
                const SizedBox(height: 4),
                Text(
                  'Guerrier — niveau 12',
                  style: theme.textTheme.bodyMedium
                      ?.copyWith(color: c.onPrimaryContainer),
                ),
              ],
            ),
          ),
          const SizedBox(height: 16),

          // Texte principal / texte secondaire
          Text('Statistiques', style: theme.textTheme.titleLarge),
          const SizedBox(height: 4),
          Text(
            'Mises à jour après chaque combat.',
            style: theme.textTheme.bodySmall
                ?.copyWith(color: c.onSurfaceVariant),
          ),
          const SizedBox(height: 12),

          // Séparateur discret : outlineVariant (couleur par défaut de Divider)
          const Divider(),

          // Puce de sélection : secondaryContainer
          Wrap(
            spacing: 8,
            children: <Widget>[
              Chip(
                label: const Text('Force 18'),
                backgroundColor: c.secondaryContainer,
                labelStyle: TextStyle(color: c.onSecondaryContainer),
              ),
              Chip(
                label: const Text('Agilité 11'),
                backgroundColor: c.secondaryContainer,
                labelStyle: TextStyle(color: c.onSecondaryContainer),
              ),
              Chip(
                label: const Text('Magie 4'),
                backgroundColor: c.tertiaryContainer,
                labelStyle: TextStyle(color: c.onTertiaryContainer),
              ),
            ],
          ),
          const SizedBox(height: 16),

          // Bandeau d'erreur : errorContainer + onErrorContainer
          Container(
            padding: const EdgeInsets.all(16),
            decoration: BoxDecoration(
              color: c.errorContainer,
              borderRadius: BorderRadius.circular(12),
            ),
            child: Row(
              children: <Widget>[
                Icon(Icons.warning_amber, color: c.onErrorContainer),
                const SizedBox(width: 12),
                Expanded(
                  child: Text(
                    'Vos points de vie sont critiques.',
                    style: TextStyle(color: c.onErrorContainer),
                  ),
                ),
              ],
            ),
          ),
          const SizedBox(height: 24),

          // Action principale : FilledButton utilise primary automatiquement
          FilledButton.icon(
            onPressed: () {
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(content: Text('Potion utilisée.')),
              );
            },
            icon: const Icon(Icons.local_drink),
            label: const Text('Boire une potion'),
          ),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
┌────────────────────────────────────────┐
│  Fiche du joueur                       │
├────────────────────────────────────────┤
│ ╔════════════════════════════════════╗ │  primaryContainer
│ ║ Aldric le Vaillant                 ║ │
│ ║ Guerrier — niveau 12               ║ │
│ ╚════════════════════════════════════╝ │
│ Statistiques                           │  onSurface
│ Mises à jour après chaque combat.      │  onSurfaceVariant
│ ────────────────────────────────────── │  outlineVariant
│ (Force 18) (Agilité 11) (Magie 4)      │  secondary/tertiaryContainer
│ ╔════════════════════════════════════╗ │  errorContainer
│ ║ ! Vos points de vie sont critiques ║ │
│ ╚════════════════════════════════════╝ │
│ [ Boire une potion ]                   │  primary
└────────────────────────────────────────┘
```

Zéro couleur codée en dur. Cette page fonctionnera telle quelle en mode sombre.

---

## 51.6 — `Theme.of(context)` : lire le thème courant

`Theme.of(context)` remonte l'arbre des widgets depuis `context` et renvoie le `ThemeData` le plus proche.

```dart
final ThemeData theme = Theme.of(context);
final ColorScheme couleurs = theme.colorScheme;
final TextTheme textes = theme.textTheme;
```

Depuis Flutter 3.27, deux raccourcis existent :

```dart
final ColorScheme couleurs = ColorScheme.of(context); // == Theme.of(context).colorScheme
```

`ColorScheme.of(context)` est strictement équivalent à `Theme.of(context).colorScheme`, en plus court.

### La bonne pratique : une variable locale en début de `build`

```dart
@override
Widget build(BuildContext context) {
  final ThemeData theme = Theme.of(context);   // une fois
  final ColorScheme c = theme.colorScheme;
  final TextTheme t = theme.textTheme;

  return Column(
    children: <Widget>[
      Text('Titre', style: t.titleLarge),
      Text('Corps', style: t.bodyMedium?.copyWith(color: c.onSurfaceVariant)),
    ],
  );
}
```

C'est plus lisible que de répéter `Theme.of(context)` dix fois, et cela évite dix recherches dans l'arbre.

---

## 51.6.1 — `Theme.of` crée une dépendance : c'est ce qu'on veut

`Theme.of(context)` n'est pas une simple lecture. Il **abonne** le widget appelant aux changements de thème.

Conséquence : quand vous basculez en mode sombre, tous les widgets qui ont appelé `Theme.of(context)` se reconstruisent automatiquement. Vous n'avez rien à faire.

```text
      MaterialApp (theme change)
              │
              │  notifie
              v
      Tous les widgets qui ont appelé Theme.of(context)
              │
              v
      build() rappelé  ->  nouvelles couleurs
```

---

## 51.6.2 — L'erreur classique : appeler `Theme.of` au-dessus du `MaterialApp`

Voici l'erreur que tout le monde commet une fois.

```dart
// NE FONCTIONNE PAS COMME PRÉVU
class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    // Ce `context` est AU-DESSUS du MaterialApp construit juste en dessous.
    final ThemeData theme = Theme.of(context); // renvoie un thème par défaut !

    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.purple),
      ),
      home: Scaffold(
        body: Container(color: theme.colorScheme.primary), // pas du violet
      ),
    );
  }
}
```

Le `context` reçu par `build` décrit l'emplacement du widget `MonApp`, c'est-à-dire **au-dessus** du `MaterialApp` que ce même `build` est en train de créer. Le thème n'existe pas encore à cet endroit. Flutter renvoie alors un `ThemeData` par défaut (bleu Material) au lieu de lever une exception, ce qui rend le bug silencieux et déroutant.

**La correction :** extraire la page dans un widget séparé.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApp());
}

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Contexte correct',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.purple),
      ),
      home: const MaPage(), // widget séparé => son context est SOUS MaterialApp
    );
  }
}

class MaPage extends StatelessWidget {
  const MaPage({super.key});

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context); // le bon thème, cette fois
    return Scaffold(
      appBar: AppBar(title: const Text('Page')),
      body: Container(color: theme.colorScheme.primaryContainer),
    );
  }
}
```

**Résultat :**

```text
Le corps de la page est bien teinté de violet clair.
```

> Règle : **`Theme.of`, `MediaQuery.of`, `Navigator.of` et `ScaffoldMessenger.of` ne fonctionnent que sous le `MaterialApp`.** Si vous avez besoin de l'un d'eux, créez un widget enfant.

---

## 51.7 — `textTheme` et les styles nommés

Material 3 définit **quinze** styles de texte, organisés en cinq familles de trois tailles.

| Style | Taille par défaut | Usage |
| --- | --- | --- |
| `displayLarge` | 57 | très grand chiffre, écran d'accueil |
| `displayMedium` | 45 | grand nombre mis en avant |
| `displaySmall` | 36 | titre de page très visible |
| `headlineLarge` | 32 | titre de section importante |
| `headlineMedium` | 28 | titre de section |
| `headlineSmall` | 24 | titre de carte importante |
| `titleLarge` | 22 | titre d'`AppBar`, titre de dialogue |
| `titleMedium` | 16 | titre de `ListTile`, en-tête de bloc |
| `titleSmall` | 14 | sous-titre |
| `bodyLarge` | 16 | texte courant mis en avant |
| `bodyMedium` | 14 | **texte courant par défaut** |
| `bodySmall` | 12 | légende, mention discrète |
| `labelLarge` | 14 | texte des boutons |
| `labelMedium` | 12 | étiquette |
| `labelSmall` | 11 | très petite étiquette, badge |

Les logiques de nommage :

```text
display   -> énorme, décoratif, un seul par écran
headline  -> titres de haut niveau
title     -> titres de blocs et de composants
body      -> texte à lire
label     -> texte fonctionnel dans un composant (bouton, puce, onglet)
```

> Les anciens noms de Material 2 (`headline1` à `headline6`, `bodyText1`, `bodyText2`, `caption`, `button`, `subtitle1`, `overline`) ont été **supprimés**. Correspondances : `body1`/`body2` sont devenus `bodyLarge`/`bodyMedium`, `caption` est devenu `bodySmall`, `button` est devenu `labelLarge`, `overline` est devenu `labelSmall`.

Voici les quinze styles affichés.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DemoTextTheme());
}

class DemoTextTheme extends StatelessWidget {
  const DemoTextTheme({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'textTheme',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepOrange),
      ),
      home: const PageStyles(),
    );
  }
}

class PageStyles extends StatelessWidget {
  const PageStyles({super.key});

  @override
  Widget build(BuildContext context) {
    final TextTheme t = Theme.of(context).textTheme;

    final List<(String, TextStyle?)> styles = <(String, TextStyle?)>[
      ('displayLarge', t.displayLarge),
      ('displayMedium', t.displayMedium),
      ('displaySmall', t.displaySmall),
      ('headlineLarge', t.headlineLarge),
      ('headlineMedium', t.headlineMedium),
      ('headlineSmall', t.headlineSmall),
      ('titleLarge', t.titleLarge),
      ('titleMedium', t.titleMedium),
      ('titleSmall', t.titleSmall),
      ('bodyLarge', t.bodyLarge),
      ('bodyMedium', t.bodyMedium),
      ('bodySmall', t.bodySmall),
      ('labelLarge', t.labelLarge),
      ('labelMedium', t.labelMedium),
      ('labelSmall', t.labelSmall),
    ];

    return Scaffold(
      appBar: AppBar(title: const Text('Les 15 styles')),
      body: ListView.separated(
        padding: const EdgeInsets.all(16),
        itemCount: styles.length,
        separatorBuilder: (_, __) => const Divider(height: 24),
        itemBuilder: (BuildContext context, int index) {
          final (String nom, TextStyle? style) = styles[index];
          return Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: <Widget>[
              Text(
                '$nom  (${style?.fontSize?.toStringAsFixed(0)} px)',
                style: t.labelSmall,
              ),
              const SizedBox(height: 4),
              Text('Le boss du niveau 3', style: style),
            ],
          );
        },
      ),
    );
  }
}
```

**Résultat :**

```text
displayLarge  (57 px)
Le boss du niveau 3
─────────────────────────
displayMedium  (45 px)
Le boss du niveau 3
─────────────────────────
...
labelSmall  (11 px)
Le boss du niveau 3
```

> Le tuple `(String, TextStyle?)` utilisé ici est un **record** Dart 3, vu au chapitre 07. Vous pouvez le remplacer par une petite classe si vous préférez.

---

## 51.7.1 — Utiliser les styles nommés plutôt que des tailles

Comparez.

```dart
// AVANT — on dit COMMENT
Text('Inventaire', style: TextStyle(fontSize: 22, fontWeight: FontWeight.w500))

// APRÈS — on dit CE QUE C'EST
Text('Inventaire', style: Theme.of(context).textTheme.titleLarge)
```

La seconde version :

- reste cohérente avec le reste de l'application ;
- suit automatiquement le grossissement de police du système ;
- change partout si vous décidez de modifier `titleLarge` dans le thème ;
- prend la bonne couleur de texte en mode sombre.

---

## 51.8 — Personnaliser un style de texte du thème

Trois cas se présentent. Ne les confondez pas.

### Cas 1 — modifier un style pour un seul `Text`

Utilisez `copyWith` **sur le style du thème**, pas un `TextStyle` neuf.

```dart
Text(
  'Score record',
  style: Theme.of(context).textTheme.titleLarge?.copyWith(
    color: Theme.of(context).colorScheme.primary,
    fontWeight: FontWeight.bold,
  ),
)
```

On conserve la taille, la hauteur de ligne et l'espacement définis par le thème ; on ne change que ce qui doit l'être.

### Cas 2 — modifier un style dans toute l'application

On redéfinit le style dans le `textTheme` du thème.

```dart
ThemeData(
  colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
  textTheme: const TextTheme(
    titleLarge: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
  ),
)
```

> Attention : ce `TextTheme` partiel est **fusionné** avec le `TextTheme` par défaut. Les quatorze autres styles restent intacts. Vous n'avez pas besoin de tous les redéclarer.

### Cas 3 — appliquer une transformation à tous les styles

`TextTheme.apply()` applique un changement global.

```dart
final ThemeData base = ThemeData(
  colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
);

final ThemeData theme = base.copyWith(
  textTheme: base.textTheme.apply(
    fontSizeFactor: 1.1,      // tout 10 % plus grand
    fontSizeDelta: 0,         // ou +2 px sur tout
    fontFamily: 'Roboto',
  ),
);
```

Voici les trois cas réunis dans un programme complet.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DemoPersonnalisation());
}

class DemoPersonnalisation extends StatelessWidget {
  const DemoPersonnalisation({super.key});

  @override
  Widget build(BuildContext context) {
    final ThemeData base = ThemeData(
      colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF7B1FA2)),
    );

    return MaterialApp(
      title: 'Personnaliser les textes',
      theme: base.copyWith(
        // Cas 2 : on redéfinit deux styles pour toute l'application
        textTheme: base.textTheme.copyWith(
          headlineSmall: base.textTheme.headlineSmall?.copyWith(
            fontWeight: FontWeight.w800,
            letterSpacing: -0.5,
          ),
          bodyMedium: base.textTheme.bodyMedium?.copyWith(
            height: 1.5, // interligne plus aéré
          ),
        ),
      ),
      home: const PagePerso(),
    );
  }
}

class PagePerso extends StatelessWidget {
  const PagePerso({super.key});

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final TextTheme t = theme.textTheme;

    return Scaffold(
      appBar: AppBar(title: const Text('Styles personnalisés')),
      body: Padding(
        padding: const EdgeInsets.all(20),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: <Widget>[
            // Style global modifié (cas 2)
            Text('Le donjon oublié', style: t.headlineSmall),
            const SizedBox(height: 12),

            // Style global modifié (cas 2) : interligne 1.5
            Text(
              'Trois couloirs mènent à la salle du trône. Le premier est '
              'gardé par un golem de pierre. Le deuxième est inondé. '
              'Le troisième est scellé par une rune ancienne.',
              style: t.bodyMedium,
            ),
            const SizedBox(height: 20),

            // Style local modifié (cas 1)
            Text(
              'Difficulté : extrême',
              style: t.titleMedium?.copyWith(
                color: theme.colorScheme.error,
                fontStyle: FontStyle.italic,
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
┌────────────────────────────────────────┐
│  Styles personnalisés                  │
├────────────────────────────────────────┤
│ Le donjon oublié          (très gras)  │
│                                        │
│ Trois couloirs mènent à la salle du    │
│ trône. Le premier est gardé par un     │
│ golem de pierre. Le deuxième est       │
│ inondé. Le troisième est scellé par    │
│ une rune ancienne.       (aéré)        │
│                                        │
│ Difficulté : extrême   (rouge italique)│
└────────────────────────────────────────┘
```

---

## 51.8.1 — L'erreur à ne pas commettre : écraser au lieu de copier

```dart
// MAUVAIS : on perd tout ce que le thème avait défini
textTheme: const TextTheme(
  bodyMedium: TextStyle(fontSize: 15),
  // fontFamily, couleur, hauteur de ligne, espacement : tout est reparti de zéro
)
```

```dart
// BON : on part du style existant et on ajuste
textTheme: base.textTheme.copyWith(
  bodyMedium: base.textTheme.bodyMedium?.copyWith(fontSize: 15),
)
```

Dans le premier cas, le `TextStyle` construit ne contient qu'une taille ; tout le reste sera `null` et remplacé par des valeurs par défaut du moteur de rendu, souvent différentes de celles du thème. Dans le second, seul `fontSize` est remplacé.

---

## 51.9 — Les thèmes de composants

Le `colorScheme` colore automatiquement les composants standard. Mais parfois vous voulez aller plus loin : « tous mes boutons ont des coins à 8 px », « toutes mes cartes ont une élévation de 0 ».

C'est le rôle des **thèmes de composants**.

---

## 51.9.1 — `elevatedButtonTheme`

```dart
ThemeData(
  elevatedButtonTheme: ElevatedButtonThemeData(
    style: ElevatedButton.styleFrom(
      minimumSize: const Size(0, 52),
      padding: const EdgeInsets.symmetric(horizontal: 24),
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(12),
      ),
      textStyle: const TextStyle(fontSize: 16, fontWeight: FontWeight.w600),
    ),
  ),
)
```

`ElevatedButton.styleFrom(...)` est une fabrique pratique qui construit un `ButtonStyle`. Les autres boutons ont la même : `FilledButton.styleFrom`, `OutlinedButton.styleFrom`, `TextButton.styleFrom`.

---

## 51.9.2 — `appBarTheme`

```dart
ThemeData(
  appBarTheme: const AppBarTheme(
    centerTitle: true,
    elevation: 0,
    scrolledUnderElevation: 3,
    titleTextStyle: TextStyle(fontSize: 20, fontWeight: FontWeight.w600),
  ),
)
```

- `centerTitle` : centre le titre. Utile, car Material 3 aligne le titre à gauche par défaut sur Android.
- `scrolledUnderElevation` : élévation prise par l'`AppBar` quand du contenu passe dessous. Mettez `0` pour supprimer l'effet de teinte au défilement.

---

## 51.9.3 — `cardTheme`

Attention à un détail important : dans les versions récentes de Flutter, le paramètre `cardTheme` de `ThemeData` attend un objet de type **`CardThemeData`**, et non `CardTheme`.

```dart
ThemeData(
  cardTheme: CardThemeData(
    elevation: 0,
    margin: const EdgeInsets.symmetric(vertical: 6),
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(16),
    ),
  ),
)
```

`CardTheme` (sans `Data`) est le **widget** qui applique un thème de carte à un sous-arbre. C'est une source de confusion fréquente : si le compilateur vous dit qu'il attendait `CardThemeData`, ajoutez simplement `Data`.

---

## 51.9.4 — `inputDecorationTheme`

C'est le thème des champs de saisie du chapitre 49.

```dart
ThemeData(
  inputDecorationTheme: InputDecorationTheme(
    filled: true,
    border: OutlineInputBorder(
      borderRadius: BorderRadius.circular(12),
      borderSide: BorderSide.none,
    ),
    contentPadding: const EdgeInsets.symmetric(horizontal: 16, vertical: 14),
  ),
)
```

Avec ce thème, **tous** vos `TextField` et `TextFormField` deviennent des champs remplis à coins arrondis sans bordure, sans que vous ayez à répéter la décoration à chaque champ.

---

## 51.9.5 — Un thème de composants complet

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DemoComposants());
}

ThemeData construireTheme(Brightness brightness) {
  final ColorScheme schema = ColorScheme.fromSeed(
    seedColor: const Color(0xFF00639B),
    brightness: brightness,
  );

  return ThemeData(
    colorScheme: schema,

    appBarTheme: AppBarTheme(
      centerTitle: true,
      elevation: 0,
      scrolledUnderElevation: 2,
      backgroundColor: schema.surface,
      foregroundColor: schema.onSurface,
      titleTextStyle: TextStyle(
        fontSize: 20,
        fontWeight: FontWeight.w600,
        color: schema.onSurface,
      ),
    ),

    cardTheme: CardThemeData(
      elevation: 0,
      color: schema.surfaceContainerHighest,
      margin: const EdgeInsets.symmetric(vertical: 6),
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(16),
      ),
    ),

    elevatedButtonTheme: ElevatedButtonThemeData(
      style: ElevatedButton.styleFrom(
        minimumSize: const Size(0, 52),
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(12),
        ),
      ),
    ),

    filledButtonTheme: FilledButtonThemeData(
      style: FilledButton.styleFrom(
        minimumSize: const Size(0, 52),
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(12),
        ),
      ),
    ),

    outlinedButtonTheme: OutlinedButtonThemeData(
      style: OutlinedButton.styleFrom(
        minimumSize: const Size(0, 52),
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(12),
        ),
      ),
    ),

    inputDecorationTheme: InputDecorationTheme(
      filled: true,
      fillColor: schema.surfaceContainerHighest,
      border: OutlineInputBorder(
        borderRadius: BorderRadius.circular(12),
        borderSide: BorderSide.none,
      ),
      contentPadding: const EdgeInsets.symmetric(horizontal: 16, vertical: 14),
    ),

    dividerTheme: DividerThemeData(
      color: schema.outlineVariant,
      thickness: 1,
      space: 24,
    ),

    chipTheme: ChipThemeData(
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(8),
      ),
    ),

    snackBarTheme: SnackBarThemeData(
      behavior: SnackBarBehavior.floating,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(12),
      ),
    ),
  );
}

class DemoComposants extends StatelessWidget {
  const DemoComposants({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Thèmes de composants',
      theme: construireTheme(Brightness.light),
      darkTheme: construireTheme(Brightness.dark),
      home: const PageComposants(),
    );
  }
}

class PageComposants extends StatelessWidget {
  const PageComposants({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Composants')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: <Widget>[
          const TextField(
            decoration: InputDecoration(
              labelText: 'Nom du héros',
              prefixIcon: Icon(Icons.person),
            ),
          ),
          const SizedBox(height: 12),
          const TextField(
            decoration: InputDecoration(
              labelText: 'Classe',
              prefixIcon: Icon(Icons.shield),
            ),
          ),
          const Divider(),
          const Card(
            child: ListTile(
              leading: Icon(Icons.inventory_2),
              title: Text('Inventaire'),
              subtitle: Text('12 objets'),
              trailing: Icon(Icons.chevron_right),
            ),
          ),
          const Card(
            child: ListTile(
              leading: Icon(Icons.map),
              title: Text('Carte du monde'),
              subtitle: Text('3 régions découvertes'),
              trailing: Icon(Icons.chevron_right),
            ),
          ),
          const Divider(),
          Wrap(
            spacing: 8,
            children: const <Widget>[
              Chip(label: Text('Épée')),
              Chip(label: Text('Bouclier')),
              Chip(label: Text('Arc')),
            ],
          ),
          const SizedBox(height: 16),
          FilledButton(
            onPressed: () {
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(content: Text('Partie sauvegardée.')),
              );
            },
            child: const Text('Sauvegarder'),
          ),
          const SizedBox(height: 8),
          ElevatedButton(onPressed: () {}, child: const Text('Charger')),
          const SizedBox(height: 8),
          OutlinedButton(onPressed: () {}, child: const Text('Quitter')),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
Tous les boutons font 52 px de haut, tous les champs sont
remplis et arrondis à 12, toutes les cartes sont plates et
arrondies à 16. Aucune de ces valeurs n'apparaît dans la page.
La fonction construireTheme est réutilisée pour le mode sombre.
```

Notez le point clé : la fonction `construireTheme(Brightness)` produit le thème clair **et** le thème sombre. Un seul endroit à modifier.

---

## 51.10 — `darkTheme` et `themeMode`

`MaterialApp` accepte **deux** thèmes et un sélecteur :

```dart
MaterialApp(
  theme: /* thème clair */,
  darkTheme: /* thème sombre */,
  themeMode: ThemeMode.system,
  home: /* ... */,
)
```

Le tableau de décision est simple :

| `themeMode` | Thème réellement appliqué |
| --- | --- |
| `ThemeMode.light` | toujours `theme` |
| `ThemeMode.dark` | toujours `darkTheme` (ou `theme` si `darkTheme` est `null`) |
| `ThemeMode.system` | `darkTheme` si le système est en sombre, sinon `theme` |

**`ThemeMode.system` est la valeur par défaut.** Si vous fournissez un `darkTheme`, votre application suit déjà le réglage du téléphone sans une ligne de plus.

---

## 51.10.1 — Construire le thème sombre correctement

L'erreur la plus fréquente est d'oublier `brightness: Brightness.dark` dans le `ColorScheme.fromSeed` du thème sombre.

```dart
// FAUX : le "thème sombre" est en fait clair
darkTheme: ThemeData(
  colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
)

// CORRECT
darkTheme: ThemeData(
  colorScheme: ColorScheme.fromSeed(
    seedColor: Colors.teal,
    brightness: Brightness.dark,
  ),
)
```

`ColorScheme.fromSeed` avec `Brightness.dark` inverse la logique des tons : `surface` devient très sombre, `onSurface` devient très clair, `primary` devient une version claire et désaturée de la teinte.

> **La graine reste la même dans les deux thèmes.** C'est ce qui donne la cohérence : même identité, deux luminosités.

---

## 51.10.2 — Application complète clair / sombre

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const AppDeuxThemes());
}

const Color graine = Color(0xFF386A20);

ThemeData themeClair() => ThemeData(
      colorScheme: ColorScheme.fromSeed(
        seedColor: graine,
        brightness: Brightness.light,
      ),
    );

ThemeData themeSombre() => ThemeData(
      colorScheme: ColorScheme.fromSeed(
        seedColor: graine,
        brightness: Brightness.dark,
      ),
    );

class AppDeuxThemes extends StatelessWidget {
  const AppDeuxThemes({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Clair et sombre',
      theme: themeClair(),
      darkTheme: themeSombre(),
      themeMode: ThemeMode.system,
      home: const PageJournal(),
    );
  }
}

class PageJournal extends StatelessWidget {
  const PageJournal({super.key});

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final bool estSombre = theme.brightness == Brightness.dark;

    return Scaffold(
      appBar: AppBar(title: const Text('Journal de quête')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: <Widget>[
          Card(
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: <Widget>[
                  Text('Mode actuel', style: theme.textTheme.titleMedium),
                  const SizedBox(height: 8),
                  Text(
                    estSombre ? 'sombre' : 'clair',
                    style: theme.textTheme.displaySmall?.copyWith(
                      color: theme.colorScheme.primary,
                    ),
                  ),
                  const SizedBox(height: 8),
                  Text(
                    'Changez le réglage clair/sombre de votre système : '
                    'cette page suit automatiquement.',
                    style: theme.textTheme.bodyMedium?.copyWith(
                      color: theme.colorScheme.onSurfaceVariant,
                    ),
                  ),
                ],
              ),
            ),
          ),
          const SizedBox(height: 12),
          const Card(
            child: ListTile(
              leading: Icon(Icons.flag),
              title: Text('Retrouver la clé du donjon'),
              subtitle: Text('En cours'),
            ),
          ),
          const Card(
            child: ListTile(
              leading: Icon(Icons.check_circle),
              title: Text('Parler au forgeron'),
              subtitle: Text('Terminé'),
            ),
          ),
        ],
      ),
    );
  }
}
```

**Résultat, en mode clair :**

```text
┌────────────────────────────────────────┐
│  Journal de quête       (fond très clair)
│  ┌──────────────────────────────────┐  │
│  │ Mode actuel                      │  │
│  │ clair          (vert foncé)      │  │
│  └──────────────────────────────────┘  │
└────────────────────────────────────────┘
```

**Résultat, en mode sombre :**

```text
┌────────────────────────────────────────┐
│  Journal de quête       (fond très sombre)
│  ┌──────────────────────────────────┐  │
│  │ Mode actuel                      │  │
│  │ sombre         (vert clair)      │  │
│  └──────────────────────────────────┘  │
└────────────────────────────────────────┘
```

Aucun `if (estSombre)` ne sert à choisir une couleur. La variable `estSombre` n'est là que pour afficher un texte. **Le style, lui, vient entièrement du `colorScheme`.**

---

## 51.11 — Suivre le réglage du système

Vous venez de le faire sans le savoir : `ThemeMode.system` est le comportement par défaut.

Techniquement, Flutter lit `MediaQuery.platformBrightnessOf(context)`, qui reflète le réglage du système d'exploitation. Lorsque l'utilisateur bascule son téléphone en mode sombre, la plateforme prévient Flutter, `MaterialApp` recalcule le thème, et tous les widgets qui dépendent de `Theme.of(context)` se reconstruisent.

Vous pouvez lire cette information vous-même :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DemoSysteme());
}

class DemoSysteme extends StatelessWidget {
  const DemoSysteme({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Réglage système',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
      ),
      darkTheme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.blue,
          brightness: Brightness.dark,
        ),
      ),
      home: const PageSysteme(),
    );
  }
}

class PageSysteme extends StatelessWidget {
  const PageSysteme({super.key});

  @override
  Widget build(BuildContext context) {
    final Brightness systeme = MediaQuery.platformBrightnessOf(context);
    final Brightness applique = Theme.of(context).brightness;

    return Scaffold(
      appBar: AppBar(title: const Text('Luminosité')),
      body: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: <Widget>[
            Text('Réglage du système : ${systeme.name}'),
            const SizedBox(height: 8),
            Text('Thème appliqué : ${applique.name}'),
          ],
        ),
      ),
    );
  }
}
```

**Résultat (système en mode sombre) :**

```text
Réglage du système : dark
Thème appliqué : dark
```

> Sur un émulateur Android : Paramètres > Affichage > Thème sombre. Sur iOS : Réglages > Luminosité et affichage. Sur Chrome (Flutter web) : les outils de développement, onglet Rendering, « Emulate CSS prefers-color-scheme ».

---

## 51.11.1 — Ne détectez pas le mode sombre pour choisir des couleurs

Vous verrez parfois ce code :

```dart
// MAUVAISE PRATIQUE
final bool sombre = Theme.of(context).brightness == Brightness.dark;
Container(color: sombre ? Colors.grey.shade900 : Colors.white);
```

C'est un aveu d'échec du thème. Écrivez plutôt :

```dart
// BONNE PRATIQUE
Container(color: Theme.of(context).colorScheme.surface);
```

Le rôle `surface` **est déjà** blanc en clair et presque noir en sombre. Le test est inutile.

Le test sur `brightness` reste légitime dans deux cas seulement :

1. choisir entre deux **images** différentes (un logo clair et un logo sombre) ;
2. afficher une information à l'utilisateur, comme dans l'exemple 51.10.2.

---

## 51.12 — Laisser l'utilisateur choisir son thème

Beaucoup d'applications proposent trois options : « Clair », « Sombre », « Système ». C'est exactement l'énumération `ThemeMode`.

Pour que le choix agisse, il faut que la valeur de `themeMode` soit **un état**, donc que le widget qui construit `MaterialApp` soit un `StatefulWidget` (chapitre 45).

```text
        MonApp (StatefulWidget)
          état : ThemeMode _mode
                 │
                 v
        MaterialApp(themeMode: _mode)
                 │
                 v
              PageReglages
                 │
                 │  appelle le callback
                 └──> setState(() => _mode = nouveau)
```

Voici l'application complète.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const AppChoixTheme());
}

const Color graine = Color(0xFF6750A4);

ThemeData construire(Brightness b) => ThemeData(
      colorScheme: ColorScheme.fromSeed(seedColor: graine, brightness: b),
      cardTheme: CardThemeData(
        elevation: 0,
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
      ),
    );

class AppChoixTheme extends StatefulWidget {
  const AppChoixTheme({super.key});

  @override
  State<AppChoixTheme> createState() => _AppChoixThemeState();
}

class _AppChoixThemeState extends State<AppChoixTheme> {
  ThemeMode _mode = ThemeMode.system;

  void _changerMode(ThemeMode nouveau) {
    setState(() {
      _mode = nouveau;
    });
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Choix du thème',
      theme: construire(Brightness.light),
      darkTheme: construire(Brightness.dark),
      themeMode: _mode,
      home: PageReglages(
        modeCourant: _mode,
        onModeChange: _changerMode,
      ),
    );
  }
}

class PageReglages extends StatelessWidget {
  const PageReglages({
    super.key,
    required this.modeCourant,
    required this.onModeChange,
  });

  final ThemeMode modeCourant;
  final ValueChanged<ThemeMode> onModeChange;

  String _libelle(ThemeMode m) {
    switch (m) {
      case ThemeMode.system:
        return 'Selon le système';
      case ThemeMode.light:
        return 'Toujours clair';
      case ThemeMode.dark:
        return 'Toujours sombre';
    }
  }

  IconData _icone(ThemeMode m) {
    switch (m) {
      case ThemeMode.system:
        return Icons.brightness_auto;
      case ThemeMode.light:
        return Icons.light_mode;
      case ThemeMode.dark:
        return Icons.dark_mode;
    }
  }

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return Scaffold(
      appBar: AppBar(title: const Text('Réglages')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: <Widget>[
          Text('Apparence', style: theme.textTheme.titleLarge),
          const SizedBox(height: 4),
          Text(
            'Choisissez le thème de l\'application.',
            style: theme.textTheme.bodyMedium?.copyWith(
              color: theme.colorScheme.onSurfaceVariant,
            ),
          ),
          const SizedBox(height: 16),

          // Trois choix exclusifs avec RadioListTile
          Card(
            child: Column(
              children: ThemeMode.values.map((ThemeMode m) {
                return RadioListTile<ThemeMode>(
                  value: m,
                  groupValue: modeCourant,
                  onChanged: (ThemeMode? v) {
                    if (v != null) {
                      onModeChange(v);
                    }
                  },
                  title: Text(_libelle(m)),
                  secondary: Icon(_icone(m)),
                );
              }).toList(),
            ),
          ),

          const SizedBox(height: 24),

          // Aperçu qui suit le choix
          Card(
            color: theme.colorScheme.primaryContainer,
            child: Padding(
              padding: const EdgeInsets.all(20),
              child: Row(
                children: <Widget>[
                  Icon(
                    _icone(modeCourant),
                    size: 40,
                    color: theme.colorScheme.onPrimaryContainer,
                  ),
                  const SizedBox(width: 16),
                  Expanded(
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: <Widget>[
                        Text(
                          'Aperçu',
                          style: theme.textTheme.titleMedium?.copyWith(
                            color: theme.colorScheme.onPrimaryContainer,
                          ),
                        ),
                        Text(
                          _libelle(modeCourant),
                          style: theme.textTheme.bodyMedium?.copyWith(
                            color: theme.colorScheme.onPrimaryContainer,
                          ),
                        ),
                      ],
                    ),
                  ),
                ],
              ),
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
┌────────────────────────────────────────┐
│  Réglages                              │
├────────────────────────────────────────┤
│  Apparence                             │
│  Choisissez le thème de l'application. │
│  ┌──────────────────────────────────┐  │
│  │ (A)  Selon le système        (o) │  │
│  │ (S)  Toujours clair          ( ) │  │
│  │ (L)  Toujours sombre         ( ) │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ (A)  Aperçu                      │  │
│  │      Selon le système            │  │
│  └──────────────────────────────────┘  │
└────────────────────────────────────────┘
```

Touchez « Toujours sombre » : toute l'application bascule instantanément, y compris l'`AppBar` et le fond.

---

## 51.12.1 — La limite de cette approche

Cette solution fonctionne, mais elle a un défaut : le callback `onModeChange` doit être **transmis à la main** jusqu'à la page qui l'utilise.

Si la page des réglages est à trois niveaux de profondeur dans la navigation, il faut passer la fonction à travers trois constructeurs. C'est ce qu'on appelle le *prop drilling*.

C'est exactement le problème que résout le **chapitre 52** avec `provider` et `ChangeNotifier`. Retenez le mécanisme d'aujourd'hui ; le chapitre suivant le rendra propre.

---

## 51.13 — Persister ce choix

Il reste un problème avec l'application ci-dessus : **relancez-la, et le choix est perdu.** `_mode` repart à `ThemeMode.system`.

Pour qu'un réglage survive au redémarrage, il faut l'écrire quelque part sur l'appareil. C'est le sujet du **chapitre 54 — Stockage local et persistance**, avec le paquet `shared_preferences`.

Le principe, pour que vous sachiez où l'on va :

```text
  AU DÉMARRAGE
  ────────────
  1. lire la clé 'theme_mode' dans les préférences
  2. si absente        -> ThemeMode.system
  3. si 'light'        -> ThemeMode.light
  4. si 'dark'         -> ThemeMode.dark
  5. construire MaterialApp avec cette valeur

  AU CHANGEMENT
  ─────────────
  1. setState(() => _mode = nouveau)   (effet immédiat à l'écran)
  2. écrire 'theme_mode' dans les préférences (effet au prochain lancement)
```

L'ajout du paquet se fera avec :

```text
flutter pub add shared_preferences
```

Ne l'écrivez pas encore. Terminez d'abord ce chapitre.

---

## 51.14 — Le widget `Theme` : surcharger un sous-arbre

Le thème n'est pas obligatoirement global. Le widget `Theme` applique un `ThemeData` à **une portion** de l'arbre.

```dart
Theme(
  data: /* un ThemeData */,
  child: /* le sous-arbre concerné */,
)
```

Deux usages, très différents.

### Usage 1 — repartir de zéro (rare)

```dart
Theme(
  data: ThemeData(colorScheme: ColorScheme.fromSeed(seedColor: Colors.red)),
  child: /* ... */,
)
```

Tout est remplacé. C'est ce que faisait la démonstration Material 2 / Material 3 en 51.3.

### Usage 2 — ajuster à partir du thème existant (le cas courant)

```dart
Theme(
  data: Theme.of(context).copyWith(
    colorScheme: Theme.of(context).colorScheme.copyWith(
      primary: Colors.red,
    ),
  ),
  child: /* ... */,
)
```

On récupère le thème ambiant, on en fait une copie modifiée, on l'applique localement. **C'est presque toujours la bonne forme.**

---

## 51.14.1 — Exemple : une zone « danger » en rouge

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DemoThemeLocal());
}

class DemoThemeLocal extends StatelessWidget {
  const DemoThemeLocal({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Theme local',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF00695C)),
      ),
      home: const PageDanger(),
    );
  }
}

class PageDanger extends StatelessWidget {
  const PageDanger({super.key});

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return Scaffold(
      appBar: AppBar(title: const Text('Paramètres de la partie')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: <Widget>[
          Text('Général', style: theme.textTheme.titleLarge),
          const SizedBox(height: 8),
          FilledButton(onPressed: () {}, child: const Text('Sauvegarder')),
          const SizedBox(height: 8),
          OutlinedButton(onPressed: () {}, child: const Text('Exporter')),

          const SizedBox(height: 32),

          // ZONE DANGER : thème local dérivé du thème global
          Theme(
            data: theme.copyWith(
              colorScheme: theme.colorScheme.copyWith(
                primary: theme.colorScheme.error,
                onPrimary: theme.colorScheme.onError,
                outline: theme.colorScheme.error,
              ),
            ),
            child: Builder(
              builder: (BuildContext context) {
                final ThemeData local = Theme.of(context);
                return Container(
                  padding: const EdgeInsets.all(16),
                  decoration: BoxDecoration(
                    color: local.colorScheme.errorContainer,
                    borderRadius: BorderRadius.circular(16),
                  ),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.stretch,
                    children: <Widget>[
                      Text(
                        'Zone de danger',
                        style: local.textTheme.titleLarge?.copyWith(
                          color: local.colorScheme.onErrorContainer,
                        ),
                      ),
                      const SizedBox(height: 12),
                      FilledButton(
                        onPressed: () {},
                        child: const Text('Effacer la sauvegarde'),
                      ),
                      const SizedBox(height: 8),
                      OutlinedButton(
                        onPressed: () {},
                        child: const Text('Réinitialiser le personnage'),
                      ),
                    ],
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

**Résultat :**

```text
┌────────────────────────────────────────┐
│  Paramètres de la partie               │
├────────────────────────────────────────┤
│  Général                               │
│  [ Sauvegarder ]        (vert)         │
│  [ Exporter ]           (contour vert) │
│                                        │
│ ╔════════════════════════════════════╗ │
│ ║ Zone de danger                     ║ │
│ ║ [ Effacer la sauvegarde ] (rouge)  ║ │
│ ║ [ Réinitialiser… ] (contour rouge) ║ │
│ ╚════════════════════════════════════╝ │
└────────────────────────────────────────┘
```

Les deux boutons du bas sont **exactement les mêmes widgets** que ceux du haut. Seul le thème sous lequel ils vivent a changé.

---

## 51.14.2 — Pourquoi le `Builder` est indispensable ici

Regardez de près :

```dart
Theme(
  data: /* nouveau thème */,
  child: Builder(
    builder: (BuildContext context) {
      final ThemeData local = Theme.of(context); // le NOUVEAU thème
      ...
    },
  ),
)
```

Sans `Builder`, le `context` utilisé dans le `child` serait toujours celui du `build` englobant, c'est-à-dire **au-dessus** du `Theme`. `Theme.of(context)` renverrait alors l'ancien thème.

C'est exactement le même piège qu'en 51.6.2, appliqué localement.

```text
   context du build de PageDanger
        │
        v
      Theme(data: nouveau)        <-- la frontière
        │
        v
      Builder  ->  son context est SOUS le Theme
        │
        v
      Theme.of(context) = nouveau thème
```

Les widgets qui ne lisent pas explicitement le thème (les `FilledButton`, par exemple) n'ont pas besoin du `Builder` : ils appellent `Theme.of` depuis leur propre `build`, qui est déjà sous le `Theme`.

---

## 51.15 — Les extensions de thème (`ThemeExtension`)

`ColorScheme` couvre les besoins standard. Mais votre application a peut-être des couleurs **métier** :

- vert « succès » pour une quête réussie ;
- orange « avertissement » pour un objet presque cassé ;
- une couleur par rareté d'objet : commun, rare, épique, légendaire.

Aucune de ces couleurs n'a de place dans `ColorScheme`. Les mettre en constantes globales fonctionne… jusqu'au mode sombre, où elles doivent changer.

La solution officielle est `ThemeExtension`.

---

## 51.15.1 — Le contrat à respecter

`ThemeExtension<T>` est une classe abstraite. Vous devez fournir :

| Membre | Rôle |
| --- | --- |
| `copyWith(...)` | crée une copie avec certains champs remplacés |
| `lerp(other, t)` | interpole entre deux instances, pour animer la transition de thème |
| `type` (hérité) | identifie l'extension ; il est fourni automatiquement par le générique |

Le squelette :

```dart
class MesCouleurs extends ThemeExtension<MesCouleurs> {
  const MesCouleurs({required this.succes});

  final Color succes;

  @override
  MesCouleurs copyWith({Color? succes}) {
    return MesCouleurs(succes: succes ?? this.succes);
  }

  @override
  MesCouleurs lerp(ThemeExtension<MesCouleurs>? other, double t) {
    if (other is! MesCouleurs) {
      return this;
    }
    return MesCouleurs(
      succes: Color.lerp(succes, other.succes, t)!,
    );
  }
}
```

---

## 51.15.2 — Exemple complet : les couleurs de rareté

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const AppExtension());
}

/// Couleurs métier du jeu, absentes de ColorScheme.
@immutable
class CouleursJeu extends ThemeExtension<CouleursJeu> {
  const CouleursJeu({
    required this.commun,
    required this.rare,
    required this.epique,
    required this.legendaire,
    required this.surTexte,
  });

  final Color commun;
  final Color rare;
  final Color epique;
  final Color legendaire;
  final Color surTexte;

  /// Jeu de couleurs pour le mode clair.
  static const CouleursJeu clair = CouleursJeu(
    commun: Color(0xFF9E9E9E),
    rare: Color(0xFF1565C0),
    epique: Color(0xFF6A1B9A),
    legendaire: Color(0xFFEF6C00),
    surTexte: Colors.white,
  );

  /// Jeu de couleurs pour le mode sombre : plus clair, moins saturé.
  static const CouleursJeu sombre = CouleursJeu(
    commun: Color(0xFFBDBDBD),
    rare: Color(0xFF64B5F6),
    epique: Color(0xFFCE93D8),
    legendaire: Color(0xFFFFB74D),
    surTexte: Colors.black,
  );

  @override
  CouleursJeu copyWith({
    Color? commun,
    Color? rare,
    Color? epique,
    Color? legendaire,
    Color? surTexte,
  }) {
    return CouleursJeu(
      commun: commun ?? this.commun,
      rare: rare ?? this.rare,
      epique: epique ?? this.epique,
      legendaire: legendaire ?? this.legendaire,
      surTexte: surTexte ?? this.surTexte,
    );
  }

  @override
  CouleursJeu lerp(ThemeExtension<CouleursJeu>? other, double t) {
    if (other is! CouleursJeu) {
      return this;
    }
    return CouleursJeu(
      commun: Color.lerp(commun, other.commun, t)!,
      rare: Color.lerp(rare, other.rare, t)!,
      epique: Color.lerp(epique, other.epique, t)!,
      legendaire: Color.lerp(legendaire, other.legendaire, t)!,
      surTexte: Color.lerp(surTexte, other.surTexte, t)!,
    );
  }
}

/// Raccourci de lecture, pour ne pas écrire l'appel générique partout.
extension LectureCouleursJeu on BuildContext {
  CouleursJeu get jeu => Theme.of(this).extension<CouleursJeu>()!;
}

ThemeData construire(Brightness b) {
  return ThemeData(
    colorScheme: ColorScheme.fromSeed(
      seedColor: const Color(0xFF3949AB),
      brightness: b,
    ),
    extensions: <ThemeExtension<dynamic>>[
      b == Brightness.light ? CouleursJeu.clair : CouleursJeu.sombre,
    ],
  );
}

class AppExtension extends StatelessWidget {
  const AppExtension({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'ThemeExtension',
      theme: construire(Brightness.light),
      darkTheme: construire(Brightness.dark),
      home: const PageButin(),
    );
  }
}

enum Rarete { commun, rare, epique, legendaire }

class Objet {
  const Objet(this.nom, this.rarete);
  final String nom;
  final Rarete rarete;
}

class PageButin extends StatelessWidget {
  const PageButin({super.key});

  static const List<Objet> butin = <Objet>[
    Objet('Dague rouillée', Rarete.commun),
    Objet('Corde de chanvre', Rarete.commun),
    Objet('Arc elfique', Rarete.rare),
    Objet('Amulette de givre', Rarete.epique),
    Objet('Lame du Crépuscule', Rarete.legendaire),
  ];

  Color _couleur(BuildContext context, Rarete r) {
    final CouleursJeu j = context.jeu;
    switch (r) {
      case Rarete.commun:
        return j.commun;
      case Rarete.rare:
        return j.rare;
      case Rarete.epique:
        return j.epique;
      case Rarete.legendaire:
        return j.legendaire;
    }
  }

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return Scaffold(
      appBar: AppBar(title: const Text('Butin du donjon')),
      body: ListView.builder(
        padding: const EdgeInsets.all(16),
        itemCount: butin.length,
        itemBuilder: (BuildContext context, int index) {
          final Objet o = butin[index];
          final Color couleur = _couleur(context, o.rarete);
          return Card(
            child: ListTile(
              leading: CircleAvatar(
                backgroundColor: couleur,
                child: Icon(Icons.star, color: context.jeu.surTexte),
              ),
              title: Text(o.nom, style: theme.textTheme.titleMedium),
              subtitle: Text(
                o.rarete.name,
                style: theme.textTheme.bodySmall?.copyWith(color: couleur),
              ),
            ),
          );
        },
      ),
    );
  }
}
```

**Résultat :**

```text
┌────────────────────────────────────────┐
│  Butin du donjon                       │
├────────────────────────────────────────┤
│ (gris)   Dague rouillée / commun       │
│ (gris)   Corde de chanvre / commun     │
│ (bleu)   Arc elfique / rare            │
│ (violet) Amulette de givre / epique    │
│ (orange) Lame du Crépuscule/legendaire │
└────────────────────────────────────────┘
```

En mode sombre, les cinq couleurs deviennent leurs variantes claires, **sans un seul `if` dans la page**.

---

## 51.15.3 — Les points à ne pas rater

1. `extensions` attend une **liste** : `<ThemeExtension<dynamic>>[ ... ]`.
2. `Theme.of(context).extension<CouleursJeu>()` renvoie `CouleursJeu?`. Si vous oubliez d'enregistrer l'extension dans le thème, c'est `null`, et le `!` lève une exception au premier affichage. C'est un bon défaut : l'erreur est immédiate et évidente.
3. L'extension sur `BuildContext` (`context.jeu`) n'est pas obligatoire, mais elle rend le code des pages beaucoup plus lisible. C'est la technique des *extension methods* du chapitre 11.
4. Implémentez `lerp` sérieusement : c'est lui qui rend fluide l'animation quand on change de thème.

---

## 51.16 — Centraliser ses constantes de design

Les couleurs et les textes sont maintenant dans le thème. Restent les **espacements**, les **rayons** et les **durées**. Eux aussi doivent être centralisés.

Deux techniques cohabitent. Choisissez-en une et tenez-vous-y.

### Technique 1 — un fichier de constantes (le plus simple)

```dart
/// Fichier lib/design/dimensions.dart
abstract final class Espace {
  static const double xs = 4;
  static const double s = 8;
  static const double m = 16;
  static const double l = 24;
  static const double xl = 32;
  static const double xxl = 48;
}

abstract final class Rayon {
  static const double petit = 8;
  static const double moyen = 12;
  static const double grand = 16;
  static const double gelule = 999;
}
```

L'échelle est **volontairement limitée**. Six espacements possibles, pas trente. C'est ce qui produit un rendu régulier.

### Technique 2 — une `ThemeExtension` de dimensions

Utile si vos espacements doivent changer selon la taille de l'écran (plus généreux sur tablette).

Voici les deux techniques appliquées.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const AppConstantes());
}

// ── Technique 1 : constantes pures ─────────────────────────────
abstract final class Espace {
  static const double xs = 4;
  static const double s = 8;
  static const double m = 16;
  static const double l = 24;
  static const double xl = 32;

  static const EdgeInsets pageH = EdgeInsets.symmetric(horizontal: m);
  static const EdgeInsets carte = EdgeInsets.all(m);
  static const SizedBox vS = SizedBox(height: s);
  static const SizedBox vM = SizedBox(height: m);
  static const SizedBox vL = SizedBox(height: l);
}

abstract final class Rayon {
  static const Radius petit = Radius.circular(8);
  static const Radius moyen = Radius.circular(12);
  static const Radius grand = Radius.circular(16);

  static const BorderRadius brMoyen = BorderRadius.all(moyen);
  static const BorderRadius brGrand = BorderRadius.all(grand);
}

// ── Technique 2 : extension de thème pour les dimensions ───────
@immutable
class Dimensions extends ThemeExtension<Dimensions> {
  const Dimensions({required this.margePage, required this.hauteurCarte});

  final double margePage;
  final double hauteurCarte;

  static const Dimensions telephone =
      Dimensions(margePage: 16, hauteurCarte: 96);
  static const Dimensions tablette =
      Dimensions(margePage: 32, hauteurCarte: 128);

  @override
  Dimensions copyWith({double? margePage, double? hauteurCarte}) {
    return Dimensions(
      margePage: margePage ?? this.margePage,
      hauteurCarte: hauteurCarte ?? this.hauteurCarte,
    );
  }

  @override
  Dimensions lerp(ThemeExtension<Dimensions>? other, double t) {
    if (other is! Dimensions) {
      return this;
    }
    return Dimensions(
      margePage: margePage + (other.margePage - margePage) * t,
      hauteurCarte: hauteurCarte + (other.hauteurCarte - hauteurCarte) * t,
    );
  }
}

class AppConstantes extends StatelessWidget {
  const AppConstantes({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Constantes de design',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF00838F)),
        cardTheme: const CardThemeData(
          elevation: 0,
          shape: RoundedRectangleBorder(borderRadius: Rayon.brGrand),
        ),
        extensions: const <ThemeExtension<dynamic>>[Dimensions.telephone],
      ),
      home: const PageConstantes(),
    );
  }
}

class PageConstantes extends StatelessWidget {
  const PageConstantes({super.key});

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final Dimensions d = theme.extension<Dimensions>()!;

    return Scaffold(
      appBar: AppBar(title: const Text('Design cohérent')),
      body: ListView(
        padding: EdgeInsets.symmetric(horizontal: d.margePage),
        children: <Widget>[
          Espace.vM,
          Text('Compétences', style: theme.textTheme.titleLarge),
          Espace.vS,
          SizedBox(
            height: d.hauteurCarte,
            child: Card(
              child: Padding(
                padding: Espace.carte,
                child: Row(
                  children: <Widget>[
                    const Icon(Icons.local_fire_department, size: 32),
                    const SizedBox(width: Espace.m),
                    Expanded(
                      child: Column(
                        mainAxisAlignment: MainAxisAlignment.center,
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: <Widget>[
                          Text('Boule de feu',
                              style: theme.textTheme.titleMedium),
                          Text('Coût : 12 mana',
                              style: theme.textTheme.bodySmall),
                        ],
                      ),
                    ),
                  ],
                ),
              ),
            ),
          ),
          Espace.vL,
          Text('Tous les espacements de cette page viennent de la classe '
              'Espace, tous les rayons de la classe Rayon, et les deux '
              'dimensions variables de l\'extension Dimensions.',
              style: theme.textTheme.bodyMedium),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
┌────────────────────────────────────────┐
│  Design cohérent                       │
├────────────────────────────────────────┤
│  Compétences                           │
│  ┌──────────────────────────────────┐  │
│  │ (i)  Boule de feu                │  │  (hauteur 96)
│  │     Coût : 12 mana               │  │
│  └──────────────────────────────────┘  │
│                                        │
│  Tous les espacements de cette page…   │
└────────────────────────────────────────┘
```

> `abstract final class` est la façon moderne, en Dart 3, de déclarer une classe qui ne sert que de conteneur de constantes : on ne peut ni l'instancier ni en hériter.

---

## 51.16.1 — Une échelle d'espacement, pourquoi ?

```text
  SANS échelle                  AVEC échelle (4, 8, 16, 24, 32)
  ────────────                  ───────────────────────────────
  padding: 13                   padding: Espace.m   (16)
  padding: 15                   padding: Espace.m   (16)
  padding: 18                   padding: Espace.m   (16)
  padding: 20                   padding: Espace.l   (24)
  padding: 22                   padding: Espace.l   (24)

  -> l'œil perçoit du désordre  -> l'œil perçoit un rythme
```

Le cerveau humain repère très bien les régularités. Cinq valeurs bien choisies suffisent à faire paraître une interface « professionnelle ». Vingt-cinq valeurs approximatives la font paraître bâclée, même si chacune prise isolément est correcte.

---

## 51.17 — `MediaQuery` : interroger l'écran

Le thème est réglé. Passons à la seconde moitié du chapitre : **l'adaptation**.

`MediaQuery` est le widget qui expose les caractéristiques de la fenêtre dans laquelle votre application s'affiche. `MaterialApp` en installe un automatiquement à la racine.

On le lit avec :

```dart
final MediaQueryData donnees = MediaQuery.of(context);
final Size taille = donnees.size;
```

Ou, mieux, avec les accesseurs ciblés :

```dart
final Size taille = MediaQuery.sizeOf(context);
```

---

## 51.17.1 — Pourquoi préférer `sizeOf` à `of`

`MediaQuery.of(context)` abonne votre widget à **toutes** les propriétés du `MediaQueryData`. Si le clavier s'ouvre, `viewInsets` change, et votre widget se reconstruit — même s'il ne s'intéresse qu'à la largeur de l'écran.

`MediaQuery.sizeOf(context)` n'abonne votre widget qu'à `size`. Les autres changements ne le réveillent pas.

| Accesseur ciblé | Propriété lue |
| --- | --- |
| `MediaQuery.sizeOf(context)` | `size` |
| `MediaQuery.orientationOf(context)` | `orientation` |
| `MediaQuery.textScalerOf(context)` | `textScaler` |
| `MediaQuery.paddingOf(context)` | `padding` |
| `MediaQuery.viewInsetsOf(context)` | `viewInsets` |
| `MediaQuery.viewPaddingOf(context)` | `viewPadding` |
| `MediaQuery.platformBrightnessOf(context)` | `platformBrightness` |
| `MediaQuery.devicePixelRatioOf(context)` | `devicePixelRatio` |
| `MediaQuery.boldTextOf(context)` | `boldText` |
| `MediaQuery.highContrastOf(context)` | `highContrast` |
| `MediaQuery.disableAnimationsOf(context)` | `disableAnimations` |
| `MediaQuery.accessibleNavigationOf(context)` | `accessibleNavigation` |

> **Règle :** utilisez toujours l'accesseur le plus précis possible. `MediaQuery.of(context)` ne se justifie que si vous lisez trois ou quatre propriétés d'un coup.

---

## 51.18 — Taille, orientation, `textScaler`, `padding`

Voici le tableau de bord de toutes ces valeurs, en un programme.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const AppMediaQuery());
}

class AppMediaQuery extends StatelessWidget {
  const AppMediaQuery({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'MediaQuery',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blueGrey),
      ),
      home: const PageMediaQuery(),
    );
  }
}

class PageMediaQuery extends StatelessWidget {
  const PageMediaQuery({super.key});

  @override
  Widget build(BuildContext context) {
    final MediaQueryData mq = MediaQuery.of(context);
    final ThemeData theme = Theme.of(context);

    final List<(String, String)> lignes = <(String, String)>[
      ('size.width', mq.size.width.toStringAsFixed(1)),
      ('size.height', mq.size.height.toStringAsFixed(1)),
      ('orientation', mq.orientation.name),
      ('devicePixelRatio', mq.devicePixelRatio.toStringAsFixed(2)),
      ('textScaler.scale(16)', mq.textScaler.scale(16).toStringAsFixed(1)),
      ('padding.top', mq.padding.top.toStringAsFixed(1)),
      ('padding.bottom', mq.padding.bottom.toStringAsFixed(1)),
      ('viewInsets.bottom', mq.viewInsets.bottom.toStringAsFixed(1)),
      ('viewPadding.top', mq.viewPadding.top.toStringAsFixed(1)),
      ('platformBrightness', mq.platformBrightness.name),
      ('boldText', mq.boldText.toString()),
      ('highContrast', mq.highContrast.toString()),
      ('disableAnimations', mq.disableAnimations.toString()),
      ('accessibleNavigation', mq.accessibleNavigation.toString()),
    ];

    return Scaffold(
      appBar: AppBar(title: const Text('Tableau de bord MediaQuery')),
      body: Column(
        children: <Widget>[
          Expanded(
            child: ListView.separated(
              itemCount: lignes.length,
              separatorBuilder: (_, __) => const Divider(height: 1),
              itemBuilder: (BuildContext context, int index) {
                final (String cle, String valeur) = lignes[index];
                return ListTile(
                  dense: true,
                  title: Text(cle, style: theme.textTheme.bodyMedium),
                  trailing: Text(
                    valeur,
                    style: theme.textTheme.titleSmall?.copyWith(
                      color: theme.colorScheme.primary,
                    ),
                  ),
                );
              },
            ),
          ),
          const Padding(
            padding: EdgeInsets.all(12),
            child: TextField(
              decoration: InputDecoration(
                labelText: 'Touchez ici et observez viewInsets.bottom',
                border: OutlineInputBorder(),
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

**Résultat (téléphone en portrait, clavier fermé) :**

```text
size.width                          392.7
size.height                         783.3
orientation                      portrait
devicePixelRatio                     2.75
textScaler.scale(16)                 16.0
padding.top                          24.0
padding.bottom                       24.0
viewInsets.bottom                     0.0
viewPadding.top                      24.0
platformBrightness                  light
boldText                            false
highContrast                        false
```

**Résultat (le clavier est ouvert) :**

```text
viewInsets.bottom                   291.0
```

---

## 51.18.1 — `size` est en pixels logiques, pas en pixels physiques

```text
  Écran physique : 1080 x 2400 pixels
  devicePixelRatio : 2.75
  ────────────────────────────────────
  size = 1080 / 2.75  x  2400 / 2.75
       = 392.7  x  872.7  (moins les barres système)
```

Un pixel logique fait toujours environ la même taille **physique** (autour de 1/160 de pouce), quel que soit l'appareil. C'est ce qui permet d'écrire `width: 100` sans se soucier de la densité de l'écran.

> Ne calculez jamais avec `devicePixelRatio`. Il ne sert qu'à choisir la résolution d'une image à télécharger.

---

## 51.18.2 — `padding`, `viewPadding`, `viewInsets` : la différence

C'est un point qui prête à confusion. Le schéma :

```text
   ┌──────────────────────────────────────┐
   │ ░░░░░ encoche / barre d'état ░░░░░░░ │  <-- viewPadding.top ET padding.top
   ├──────────────────────────────────────┤
   │                                      │
   │            contenu utile             │
   │                                      │
   ├──────────────────────────────────────┤
   │ ▓▓▓▓▓▓▓▓ clavier ouvert ▓▓▓▓▓▓▓▓▓▓▓▓ │  <-- viewInsets.bottom
   │ ░░░ barre de gestes système ░░░░░░░░ │  <-- viewPadding.bottom
   └──────────────────────────────────────┘
                                              padding.bottom = 0
                                              (le clavier cache déjà la barre)
```

| Propriété | Définition | Quand ça bouge |
| --- | --- | --- |
| `viewPadding` | zones occupées par l'interface système, **toujours** | rotation, changement d'appareil |
| `viewInsets` | zones **complètement** masquées, typiquement par le clavier | ouverture / fermeture du clavier |
| `padding` | `viewPadding` moins ce que `viewInsets` recouvre déjà | les deux |

En pratique :

- pour éviter l'encoche : `SafeArea`, ou `padding.top` (51.30) ;
- pour savoir si le clavier est ouvert : `MediaQuery.viewInsetsOf(context).bottom > 0` (51.29).

---

## 51.19 — Respecter le grossissement de police de l'utilisateur

Sur Android comme sur iOS, l'utilisateur peut agrandir la taille du texte du système. Certains réglages vont jusqu'à **200 %**.

Flutter respecte ce réglage automatiquement pour tous les `Text` : c'est l'objet `TextScaler`.

```dart
final TextScaler s = MediaQuery.textScalerOf(context);
final double taillePeinte = s.scale(16); // 16 -> 24 si l'utilisateur est à 150 %
```

`TextScaler` fournit :

| Membre | Rôle |
| --- | --- |
| `scale(double fontSize)` | calcule la taille réellement peinte |
| `TextScaler.linear(double facteur)` | un scaler proportionnel simple |
| `TextScaler.noScaling` | aucun grossissement |
| `clamp(minScaleFactor: , maxScaleFactor: )` | borne le grossissement |

> `textScaleFactor` (un simple `double`) est **déprécié** et disparaîtra. Utilisez `textScaler`. La raison : sur certaines plateformes le grossissement n'est pas linéaire, les gros textes grossissant moins que les petits.

---

## 51.19.1 — Ce que le grossissement casse, et comment y survivre

```text
  À 100 %                       À 200 %
  ──────────────────────        ──────────────────────
  ┌──────────────────┐          ┌──────────────────┐
  │ [icone] Épée  12 │          │ [icone] Épée  1▒ │  <-- débordement
  └──────────────────┘          │ ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒ │
                                └──────────────────┘
```

Les quatre règles de survie :

1. **Ne fixez jamais la hauteur d'un widget qui contient du texte.** Utilisez `Padding`, pas `SizedBox(height: 48)`.
2. **Enveloppez le texte d'une `Row` dans `Expanded` ou `Flexible`.** Sinon, `RenderFlex overflowed`.
3. **Préférez `Wrap` à `Row`** quand les éléments peuvent passer à la ligne.
4. **Bornez le grossissement** aux endroits où il est vraiment impossible d'adapter la mise en page — et seulement là.

---

## 51.19.2 — Borner le grossissement avec `MediaQuery.withClampedTextScaling`

La signature exacte :

```dart
static Widget withClampedTextScaling({
  Key? key,
  double minScaleFactor = 0.0,
  double maxScaleFactor = double.infinity,
  required Widget child,
})
```

Elle enveloppe `child` dans un `MediaQuery` dont le `textScaler` est borné.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const AppGrossissement());
}

class AppGrossissement extends StatefulWidget {
  const AppGrossissement({super.key});

  @override
  State<AppGrossissement> createState() => _AppGrossissementState();
}

class _AppGrossissementState extends State<AppGrossissement> {
  double _facteur = 1.0;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Grossissement',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.orange),
      ),
      // On SIMULE le réglage système pour pouvoir tester sans quitter l'app.
      builder: (BuildContext context, Widget? child) {
        return MediaQuery(
          data: MediaQuery.of(context).copyWith(
            textScaler: TextScaler.linear(_facteur),
          ),
          child: child!,
        );
      },
      home: PageGrossissement(
        facteur: _facteur,
        onChange: (double v) => setState(() => _facteur = v),
      ),
    );
  }
}

class PageGrossissement extends StatelessWidget {
  const PageGrossissement({
    super.key,
    required this.facteur,
    required this.onChange,
  });

  final double facteur;
  final ValueChanged<double> onChange;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final TextScaler scaler = MediaQuery.textScalerOf(context);

    return Scaffold(
      appBar: AppBar(title: const Text('Grossissement du texte')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: <Widget>[
          Text('Facteur simulé : ${facteur.toStringAsFixed(2)}',
              style: theme.textTheme.titleMedium),
          Slider(
            value: facteur,
            min: 0.8,
            max: 2.5,
            divisions: 17,
            label: facteur.toStringAsFixed(2),
            onChanged: onChange,
          ),
          Text('bodyMedium (14) est peint à '
              '${scaler.scale(14).toStringAsFixed(1)} px'),
          const Divider(height: 32),

          // MAUVAIS : hauteur fixe -> le texte est coupé
          Text('Hauteur fixe (casse)', style: theme.textTheme.titleSmall),
          const SizedBox(height: 8),
          Container(
            height: 48,
            color: theme.colorScheme.errorContainer,
            alignment: Alignment.centerLeft,
            padding: const EdgeInsets.symmetric(horizontal: 12),
            child: const Text('Potion de soin majeure'),
          ),
          const SizedBox(height: 24),

          // BON : la hauteur s'adapte
          Text('Hauteur libre (résiste)', style: theme.textTheme.titleSmall),
          const SizedBox(height: 8),
          Container(
            color: theme.colorScheme.primaryContainer,
            padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 12),
            child: const Text('Potion de soin majeure'),
          ),
          const SizedBox(height: 24),

          // BON : Row + Expanded
          Text('Row + Expanded', style: theme.textTheme.titleSmall),
          const SizedBox(height: 8),
          Row(
            children: <Widget>[
              const Icon(Icons.local_drink),
              const SizedBox(width: 8),
              Expanded(
                child: Text(
                  'Potion de soin majeure de la guilde des alchimistes',
                  style: theme.textTheme.bodyMedium,
                ),
              ),
            ],
          ),
          const SizedBox(height: 24),

          // Grossissement borné, pour un badge dont la taille est critique
          Text('Badge à grossissement borné', style: theme.textTheme.titleSmall),
          const SizedBox(height: 8),
          MediaQuery.withClampedTextScaling(
            maxScaleFactor: 1.3,
            child: Container(
              padding: const EdgeInsets.symmetric(horizontal: 10, vertical: 6),
              decoration: BoxDecoration(
                color: theme.colorScheme.tertiaryContainer,
                borderRadius: BorderRadius.circular(999),
              ),
              child: Text(
                'NIVEAU 12',
                style: theme.textTheme.labelMedium?.copyWith(
                  color: theme.colorScheme.onTertiaryContainer,
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

**Résultat (facteur 2.0) :**

```text
Facteur simulé : 2.00
bodyMedium (14) est peint à 28.0 px

Hauteur fixe (casse)
┌──────────────────────────────┐
│ Potion de soin majeu▒▒▒▒▒▒▒▒ │   texte coupé
└──────────────────────────────┘

Hauteur libre (résiste)
┌──────────────────────────────┐
│ Potion de soin majeure       │   la boîte a grandi
└──────────────────────────────┘

Row + Expanded
(i) Potion de soin majeure de la
   guilde des alchimistes           passe à la ligne

Badge à grossissement borné
( NIVEAU 12 )                       grossi de 30 % seulement
```

> `MaterialApp.builder` sert ici à insérer un widget **entre** `MaterialApp` et toutes vos pages. C'est le point d'accroche standard pour ce genre de réglage global.

---

## 51.20 — `LayoutBuilder` : mesurer l'espace réellement disponible

`MediaQuery` donne la taille de **la fenêtre entière**. Ce n'est pas toujours ce dont vous avez besoin.

Si votre widget est dans un panneau latéral de 300 px, `MediaQuery.sizeOf(context).width` vous répond « 1400 ». Vous prendrez la mauvaise décision.

`LayoutBuilder` répond à la vraie question : **quelles contraintes mon parent m'impose-t-il ?**

```dart
LayoutBuilder(
  builder: (BuildContext context, BoxConstraints constraints) {
    // constraints.maxWidth  : largeur maximale autorisée
    // constraints.maxHeight : hauteur maximale autorisée
    return /* un widget choisi en fonction */;
  },
)
```

`BoxConstraints` est l'objet que vous connaissez du chapitre 46 : quatre nombres, `minWidth`, `maxWidth`, `minHeight`, `maxHeight`.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const AppLayoutBuilder());
}

class AppLayoutBuilder extends StatelessWidget {
  const AppLayoutBuilder({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'LayoutBuilder',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
      ),
      home: const PageComparaison(),
    );
  }
}

class PageComparaison extends StatelessWidget {
  const PageComparaison({super.key});

  @override
  Widget build(BuildContext context) {
    final double largeurFenetre = MediaQuery.sizeOf(context).width;

    return Scaffold(
      appBar: AppBar(title: const Text('MediaQuery vs LayoutBuilder')),
      body: Column(
        children: <Widget>[
          Padding(
            padding: const EdgeInsets.all(12),
            child: Text(
              'MediaQuery.sizeOf(context).width = '
              '${largeurFenetre.toStringAsFixed(1)}',
              style: Theme.of(context).textTheme.titleMedium,
            ),
          ),
          Expanded(
            child: Row(
              children: <Widget>[
                const SizedBox(width: 120, child: _Mesure(nom: 'Panneau 120')),
                const VerticalDivider(width: 1),
                Expanded(flex: 1, child: const _Mesure(nom: 'Zone flex 1')),
                const VerticalDivider(width: 1),
                Expanded(flex: 2, child: const _Mesure(nom: 'Zone flex 2')),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

class _Mesure extends StatelessWidget {
  const _Mesure({required this.nom});

  final String nom;

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (BuildContext context, BoxConstraints c) {
        return Container(
          color: Theme.of(context).colorScheme.surfaceContainerHighest,
          padding: const EdgeInsets.all(8),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              Text(nom, textAlign: TextAlign.center),
              const SizedBox(height: 8),
              Text('maxWidth\n${c.maxWidth.toStringAsFixed(1)}',
                  textAlign: TextAlign.center),
              const SizedBox(height: 8),
              Text('maxHeight\n${c.maxHeight.toStringAsFixed(1)}',
                  textAlign: TextAlign.center),
            ],
          ),
        );
      },
    );
  }
}
```

**Résultat (fenêtre de 800 px de large) :**

```text
MediaQuery.sizeOf(context).width = 800.0

┌──────────┬────────────────┬────────────────────────────┐
│ Panneau  │ Zone flex 1    │ Zone flex 2                │
│   120    │                │                            │
│ maxWidth │ maxWidth       │ maxWidth                   │
│  120.0   │  225.7         │  451.3                     │
│ maxHeight│ maxHeight      │ maxHeight                  │
│  668.0   │  668.0         │  668.0                     │
└──────────┴────────────────┴────────────────────────────┘
```

Une seule valeur pour `MediaQuery`, trois valeurs différentes pour `LayoutBuilder`. Ce sont ces trois-là qui comptent pour décider d'une mise en page.

---

## 51.20.1 — Le piège des contraintes non bornées

`LayoutBuilder` reçoit les contraintes de son parent. Si ce parent n'en impose pas dans une direction, la valeur est `double.infinity`.

```dart
// Dans un ListView, la hauteur n'est PAS bornée
ListView(
  children: <Widget>[
    LayoutBuilder(
      builder: (BuildContext context, BoxConstraints c) {
        // c.maxHeight vaut double.infinity !
        // if (c.maxHeight > 400) ... est toujours vrai : test inutile
        return Text('maxWidth = ${c.maxWidth}'); // celle-ci est fiable
      },
    ),
  ],
)
```

> **Décidez sur `maxWidth`, jamais sur `maxHeight`**, sauf si vous êtes certain que la hauteur est bornée. C'est aussi la bonne pratique en soi : les points de rupture se raisonnent en largeur.

---

## 51.21 — `MediaQuery` ou `LayoutBuilder` : lequel et quand

| Question posée | Outil |
| --- | --- |
| « Quelle est la taille de la fenêtre ? » | `MediaQuery.sizeOf` |
| « Suis-je sur téléphone, tablette ou bureau ? » | `MediaQuery.sizeOf` (décision globale) |
| « Combien de place ai-je, ici, à cet endroit ? » | `LayoutBuilder` |
| « Combien de colonnes puis-je afficher dans ce panneau ? » | `LayoutBuilder` |
| « L'écran est-il en paysage ? » | `MediaQuery.orientationOf` ou `OrientationBuilder` |
| « Le clavier est-il ouvert ? » | `MediaQuery.viewInsetsOf` |
| « Quelle est la hauteur de l'encoche ? » | `MediaQuery.paddingOf` ou `SafeArea` |
| « De combien l'utilisateur a-t-il grossi la police ? » | `MediaQuery.textScalerOf` |

La règle mémotechnique :

```text
   MediaQuery    ->  décision de HAUT NIVEAU : quelle mise en page globale ?
                     (une seule fois, en haut de l'écran)

   LayoutBuilder ->  décision LOCALE : comment remplir CETTE zone ?
                     (dans un composant réutilisable)
```

**Corollaire important :** un widget réutilisable (une carte, une grille, un panneau) doit utiliser `LayoutBuilder`. S'il utilise `MediaQuery`, il donnera un résultat faux dès qu'on le placera ailleurs que sur toute la largeur.

---

## 51.21.1 — Un widget qui se trompe, et sa correction

```dart
// FRAGILE : dépend de la fenêtre, pas de sa place réelle
class CarteFragile extends StatelessWidget {
  const CarteFragile({super.key});

  @override
  Widget build(BuildContext context) {
    final bool large = MediaQuery.sizeOf(context).width > 600;
    return large ? const _VersionLarge() : const _VersionEtroite();
  }
}

// ROBUSTE : dépend de l'espace qui lui est réellement donné
class CarteRobuste extends StatelessWidget {
  const CarteRobuste({super.key});

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (BuildContext context, BoxConstraints c) {
        return c.maxWidth > 600 ? const _VersionLarge() : const _VersionEtroite();
      },
    );
  }
}
```

Placez `CarteFragile` dans un panneau de 250 px sur un écran de bureau de 1600 px : elle affichera sa version large dans 250 px, et débordera.

---

## 51.22 — Les points de rupture

Un **point de rupture** (*breakpoint*) est une largeur au-delà de laquelle vous changez de mise en page.

Material 3 définit cinq classes de taille de fenêtre, en pixels logiques de largeur :

```text
    0        600       840       1200      1600           largeur (dp)
    ├─────────┼─────────┼─────────┼─────────┼──────────────────>
    │ COMPACT │ MEDIUM  │EXPANDED │  LARGE  │  EXTRA-LARGE
    │         │         │         │         │
  téléphone  tablette  tablette  grande     écran de
  portrait   portrait  paysage   tablette   bureau
             pliable   pliable
             ouvert    ouvert
             portrait  paysage
```

| Classe | Largeur | Représente |
| --- | --- | --- |
| `compact` | < 600 | 99,9 % des téléphones en portrait |
| `medium` | 600 à 839 | la plupart des tablettes en portrait, pliables ouverts |
| `expanded` | 840 à 1199 | tablettes en paysage, pliables en paysage |
| `large` | 1200 à 1599 | grandes tablettes, fenêtres de bureau |
| `extraLarge` | ≥ 1600 | écrans de bureau |

**Attention à un point essentiel :** la classe de taille dépend de la **fenêtre**, pas de l'appareil. Une application en mode multi-fenêtres sur une tablette peut être en `compact`. Une fenêtre de bureau redimensionnée peut passer de `large` à `compact` pendant que l'utilisateur tire la souris.

---

## 51.22.1 — Une énumération de classes de taille

En pratique, trois classes suffisent souvent : téléphone, tablette, bureau. Voici une implémentation propre.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const AppRuptures());
}

/// Classes de taille inspirées de Material 3, simplifiées à trois niveaux.
enum ClasseEcran {
  compact,   // < 600  : téléphone
  medium,    // 600 à 839 : tablette portrait
  expanded;  // >= 840 : tablette paysage et bureau

  static ClasseEcran depuisLargeur(double largeur) {
    if (largeur < 600) {
      return ClasseEcran.compact;
    }
    if (largeur < 840) {
      return ClasseEcran.medium;
    }
    return ClasseEcran.expanded;
  }

  bool get estCompact => this == ClasseEcran.compact;
  bool get estAuMoinsMedium => index >= ClasseEcran.medium.index;
  bool get estExpanded => this == ClasseEcran.expanded;

  /// Nombre de colonnes conseillé pour une grille.
  int get colonnes {
    switch (this) {
      case ClasseEcran.compact:
        return 2;
      case ClasseEcran.medium:
        return 3;
      case ClasseEcran.expanded:
        return 4;
    }
  }

  /// Marge horizontale conseillée.
  double get marge {
    switch (this) {
      case ClasseEcran.compact:
        return 16;
      case ClasseEcran.medium:
        return 24;
      case ClasseEcran.expanded:
        return 32;
    }
  }
}

class AppRuptures extends StatelessWidget {
  const AppRuptures({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Points de rupture',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
      ),
      home: const PageRuptures(),
    );
  }
}

class PageRuptures extends StatelessWidget {
  const PageRuptures({super.key});

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (BuildContext context, BoxConstraints c) {
        final ClasseEcran classe = ClasseEcran.depuisLargeur(c.maxWidth);
        final ThemeData theme = Theme.of(context);

        return Scaffold(
          appBar: AppBar(title: const Text('Classe de taille')),
          body: Padding(
            padding: EdgeInsets.symmetric(horizontal: classe.marge),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.stretch,
              children: <Widget>[
                const SizedBox(height: 16),
                Card(
                  color: theme.colorScheme.primaryContainer,
                  child: Padding(
                    padding: const EdgeInsets.all(20),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: <Widget>[
                        Text('Largeur : ${c.maxWidth.toStringAsFixed(0)} px',
                            style: theme.textTheme.titleMedium?.copyWith(
                              color: theme.colorScheme.onPrimaryContainer,
                            )),
                        Text('Classe : ${classe.name}',
                            style: theme.textTheme.headlineSmall?.copyWith(
                              color: theme.colorScheme.onPrimaryContainer,
                            )),
                        Text('Colonnes : ${classe.colonnes}  •  '
                            'Marge : ${classe.marge.toStringAsFixed(0)} px',
                            style: theme.textTheme.bodyMedium?.copyWith(
                              color: theme.colorScheme.onPrimaryContainer,
                            )),
                      ],
                    ),
                  ),
                ),
                const SizedBox(height: 16),
                Expanded(
                  child: GridView.builder(
                    gridDelegate:
                        SliverGridDelegateWithFixedCrossAxisCount(
                      crossAxisCount: classe.colonnes,
                      crossAxisSpacing: 12,
                      mainAxisSpacing: 12,
                    ),
                    itemCount: 12,
                    itemBuilder: (BuildContext context, int i) {
                      return Card(
                        color: theme.colorScheme.secondaryContainer,
                        child: Center(child: Text('Objet ${i + 1}')),
                      );
                    },
                  ),
                ),
              ],
            ),
          ),
        );
      },
    );
  }
}
```

**Résultat, à 400 px de large :**

```text
Largeur : 400 px
Classe : compact
Colonnes : 2  •  Marge : 16 px
┌────┐ ┌────┐
│ 1  │ │ 2  │
└────┘ └────┘
```

**Résultat, à 1000 px de large :**

```text
Largeur : 1000 px
Classe : expanded
Colonnes : 4  •  Marge : 32 px
┌────┐ ┌────┐ ┌────┐ ┌────┐
│ 1  │ │ 2  │ │ 3  │ │ 4  │
└────┘ └────┘ └────┘ └────┘
```

Redimensionnez la fenêtre (sur bureau ou web) : la bascule est instantanée.

---

## 51.22.2 — Deux erreurs à éviter avec les points de rupture

**Erreur 1 — trop de points de rupture.**
Deux suffisent presque toujours (600 et 840). Six points de rupture, ce sont six mises en page à maintenir et à tester.

**Erreur 2 — raisonner en « téléphone / tablette » plutôt qu'en largeur.**

```dart
// FRAGILE
if (Platform.isAndroid) { /* mise en page téléphone */ }
```

Une tablette Android est sous Android. Un téléphone en paysage n'est pas une tablette. Un ordinateur portable avec écran tactile n'est ni l'un ni l'autre. **La largeur disponible est la seule information fiable.**

---

## 51.23 — Une mise en page qui bascule de `Column` à `Row`

C'est le motif adaptatif le plus utile de tous.

```text
   ÉTROIT (< 700)                LARGE (>= 700)
   ──────────────                ──────────────
   ┌────────────────┐            ┌───────┬────────────┐
   │    IMAGE       │            │       │  TITRE     │
   ├────────────────┤            │ IMAGE │  TEXTE     │
   │    TITRE       │            │       │  BOUTONS   │
   │    TEXTE       │            │       │            │
   │    BOUTONS     │            └───────┴────────────┘
   └────────────────┘
        Column                        Row
```

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const AppBascule());
}

class AppBascule extends StatelessWidget {
  const AppBascule({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Column ou Row',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF00695C)),
      ),
      home: const FicheObjet(),
    );
  }
}

class FicheObjet extends StatelessWidget {
  const FicheObjet({super.key});

  static const double seuil = 700;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Lame du Crépuscule')),
      body: LayoutBuilder(
        builder: (BuildContext context, BoxConstraints c) {
          final bool large = c.maxWidth >= seuil;

          // Les deux morceaux sont écrits UNE SEULE FOIS.
          const Widget visuel = _Visuel();
          const Widget details = _Details();

          if (large) {
            return const Padding(
              padding: EdgeInsets.all(32),
              child: Row(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: <Widget>[
                  SizedBox(width: 280, child: visuel),
                  SizedBox(width: 32),
                  Expanded(child: details),
                ],
              ),
            );
          }

          return const SingleChildScrollView(
            padding: EdgeInsets.all(16),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.stretch,
              children: <Widget>[
                SizedBox(height: 220, child: visuel),
                SizedBox(height: 16),
                details,
              ],
            ),
          );
        },
      ),
    );
  }
}

class _Visuel extends StatelessWidget {
  const _Visuel();

  @override
  Widget build(BuildContext context) {
    final ColorScheme c = Theme.of(context).colorScheme;
    return Container(
      decoration: BoxDecoration(
        gradient: LinearGradient(
          begin: Alignment.topLeft,
          end: Alignment.bottomRight,
          colors: <Color>[c.primaryContainer, c.tertiaryContainer],
        ),
        borderRadius: BorderRadius.circular(20),
      ),
      child: Center(
        child: Icon(Icons.colorize, size: 96, color: c.onPrimaryContainer),
      ),
    );
  }
}

class _Details extends StatelessWidget {
  const _Details();

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      mainAxisSize: MainAxisSize.min,
      children: <Widget>[
        Text('Lame du Crépuscule', style: theme.textTheme.headlineSmall),
        const SizedBox(height: 4),
        Text('Épée longue — légendaire',
            style: theme.textTheme.titleSmall?.copyWith(
              color: theme.colorScheme.onSurfaceVariant,
            )),
        const SizedBox(height: 16),
        Text(
          'Forgée dans les cendres du volcan Kaldar, cette lame absorbe '
          'la lumière environnante. Elle inflige des dégâts d\'ombre '
          'supplémentaires la nuit et durant les éclipses.',
          style: theme.textTheme.bodyMedium,
        ),
        const SizedBox(height: 16),
        Wrap(
          spacing: 8,
          runSpacing: 8,
          children: const <Widget>[
            Chip(label: Text('Dégâts 48-62')),
            Chip(label: Text('Vitesse 1.2')),
            Chip(label: Text('Ombre +15')),
          ],
        ),
        const SizedBox(height: 24),
        Wrap(
          spacing: 12,
          runSpacing: 12,
          children: <Widget>[
            FilledButton.icon(
              onPressed: () {},
              icon: const Icon(Icons.check),
              label: const Text('Équiper'),
            ),
            OutlinedButton.icon(
              onPressed: () {},
              icon: const Icon(Icons.sell),
              label: const Text('Vendre'),
            ),
          ],
        ),
      ],
    );
  }
}
```

**Résultat, écran étroit :**

```text
┌──────────────────────────┐
│   Lame du Crépuscule     │
├──────────────────────────┤
│ ┌──────────────────────┐ │
│ │        (icône)       │ │
│ └──────────────────────┘ │
│ Lame du Crépuscule       │
│ Épée longue — légendaire │
│ Forgée dans les cendres… │
│ (Dégâts) (Vitesse)(Ombre)│
│ [Équiper] [Vendre]       │
└──────────────────────────┘
```

**Résultat, écran large :**

```text
┌────────────────────────────────────────────────────────┐
│   Lame du Crépuscule                                   │
├────────────────────────────────────────────────────────┤
│ ┌──────────┐   Lame du Crépuscule                      │
│ │          │   Épée longue — légendaire                │
│ │ (icône)  │   Forgée dans les cendres du volcan…      │
│ │          │   (Dégâts) (Vitesse) (Ombre)              │
│ └──────────┘   [Équiper] [Vendre]                      │
└────────────────────────────────────────────────────────┘
```

**Le point clé :** `_Visuel` et `_Details` ne sont écrits qu'une fois. Seul leur **arrangement** change. Si vous dupliquez le contenu dans les deux branches, vous devrez le corriger deux fois à chaque évolution.

---

## 51.24 — `OrientationBuilder`

`OrientationBuilder` reconstruit son enfant selon que la zone disponible est plus large que haute (`Orientation.landscape`) ou l'inverse (`Orientation.portrait`).

```dart
OrientationBuilder(
  builder: (BuildContext context, Orientation orientation) {
    return orientation == Orientation.portrait ? /* … */ : /* … */;
  },
)
```

> Nuance importante : `OrientationBuilder` compare la **largeur et la hauteur de son parent**, pas l'orientation physique de l'appareil. Pour l'orientation réelle de l'appareil, utilisez `MediaQuery.orientationOf(context)`.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const AppOrientation());
}

class AppOrientation extends StatelessWidget {
  const AppOrientation({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'OrientationBuilder',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.brown),
      ),
      home: const PageOrientation(),
    );
  }
}

class PageOrientation extends StatelessWidget {
  const PageOrientation({super.key});

  static const List<(IconData, String)> sorts = <(IconData, String)>[
    (Icons.local_fire_department, 'Boule de feu'),
    (Icons.ac_unit, 'Éclat de givre'),
    (Icons.bolt, 'Foudre'),
    (Icons.healing, 'Soin mineur'),
    (Icons.shield_moon, 'Bouclier d\'ombre'),
    (Icons.air, 'Bourrasque'),
  ];

  @override
  Widget build(BuildContext context) {
    final Orientation appareil = MediaQuery.orientationOf(context);

    return Scaffold(
      appBar: AppBar(title: Text('Grimoire (${appareil.name})')),
      body: OrientationBuilder(
        builder: (BuildContext context, Orientation zone) {
          final int colonnes = zone == Orientation.portrait ? 2 : 3;
          return GridView.builder(
            padding: const EdgeInsets.all(16),
            gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: colonnes,
              crossAxisSpacing: 12,
              mainAxisSpacing: 12,
              childAspectRatio: zone == Orientation.portrait ? 1.0 : 1.4,
            ),
            itemCount: sorts.length,
            itemBuilder: (BuildContext context, int i) {
              final (IconData icone, String nom) = sorts[i];
              return Card(
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: <Widget>[
                    Icon(icone, size: 36),
                    const SizedBox(height: 8),
                    Text(nom, textAlign: TextAlign.center),
                  ],
                ),
              );
            },
          );
        },
      ),
    );
  }
}
```

**Résultat, portrait :**

```text
┌───────┐ ┌───────┐
│  (1)  │ │  (2)  │
│ Boule │ │ Éclat │
└───────┘ └───────┘
┌───────┐ ┌───────┐
│  (3)  │ │  (4)  │
└───────┘ └───────┘
```

**Résultat, paysage :**

```text
┌────────┐ ┌────────┐ ┌────────┐
│   (1)  │ │   (2)  │ │   (3)  │
└────────┘ └────────┘ └────────┘
```

---

## 51.24.1 — `OrientationBuilder` est rarement le bon outil

Soyons honnêtes : dans la plupart des cas, **la largeur est une meilleure information que l'orientation**.

```text
  Téléphone en paysage : 800 x 360  -> landscape, mais ÉTROIT en hauteur
  Tablette en portrait : 800 x 1280 -> portrait,  mais LARGE
```

Un téléphone en paysage et une tablette en portrait ont la même largeur de 800 px et méritent souvent la même mise en page horizontale. `OrientationBuilder` les traiterait différemment.

**Utilisez `OrientationBuilder` quand la hauteur est le facteur limitant** : par exemple, réduire la hauteur d'un en-tête en paysage pour laisser voir la liste.

---

## 51.25 — `Flexible`, `Expanded` et les proportions (rappel du chapitre 46)

Beaucoup de problèmes d'adaptation se résolvent **sans aucun point de rupture**, simplement en raisonnant en proportions plutôt qu'en pixels.

Rappel du chapitre 46 :

| Widget | Effet sur l'enfant |
| --- | --- |
| `Expanded(flex: n, child: …)` | l'enfant **doit** occuper sa part de l'espace restant |
| `Flexible(flex: n, child: …)` | l'enfant **peut** occuper jusqu'à sa part, mais peut être plus petit |
| `Spacer(flex: n)` | un espace vide qui prend sa part |

Une `Row` avec `Expanded(flex: 1)` et `Expanded(flex: 2)` donne toujours un tiers / deux tiers, que l'écran fasse 320 ou 1600 px.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const AppProportions());
}

class AppProportions extends StatelessWidget {
  const AppProportions({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Proportions',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.cyan),
      ),
      home: const PageProportions(),
    );
  }
}

class PageProportions extends StatelessWidget {
  const PageProportions({super.key});

  @override
  Widget build(BuildContext context) {
    final ColorScheme c = Theme.of(context).colorScheme;

    return Scaffold(
      appBar: AppBar(title: const Text('Barres de statut')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            const Text('Vie 72 / 100'),
            const SizedBox(height: 4),
            // Une barre en proportions : aucune largeur en pixels
            SizedBox(
              height: 18,
              child: Row(
                children: <Widget>[
                  Expanded(
                    flex: 72,
                    child: Container(color: c.primary),
                  ),
                  Expanded(
                    flex: 28,
                    child: Container(color: c.surfaceContainerHighest),
                  ),
                ],
              ),
            ),
            const SizedBox(height: 20),
            const Text('Mana 35 / 100'),
            const SizedBox(height: 4),
            SizedBox(
              height: 18,
              child: Row(
                children: <Widget>[
                  Expanded(flex: 35, child: Container(color: c.tertiary)),
                  Expanded(
                    flex: 65,
                    child: Container(color: c.surfaceContainerHighest),
                  ),
                ],
              ),
            ),
            const SizedBox(height: 32),
            const Text('Répartition 1 / 2 / 1'),
            const SizedBox(height: 8),
            Expanded(
              child: Row(
                children: <Widget>[
                  Expanded(
                    child: Container(color: c.primaryContainer,
                        child: const Center(child: Text('1'))),
                  ),
                  Expanded(
                    flex: 2,
                    child: Container(color: c.secondaryContainer,
                        child: const Center(child: Text('2'))),
                  ),
                  Expanded(
                    child: Container(color: c.tertiaryContainer,
                        child: const Center(child: Text('1'))),
                  ),
                ],
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
Vie 72 / 100
████████████████████░░░░░░░░       toujours 72 % de la largeur
Mana 35 / 100
█████████░░░░░░░░░░░░░░░░░░░
Répartition 1 / 2 / 1
┌────┬────────┬────┐
│ 1  │   2    │ 1  │              toujours 25 / 50 / 25 %
└────┴────────┴────┘
```

> **Avant d'ajouter un point de rupture, demandez-vous si des proportions ne suffiraient pas.** Elles sont plus simples, plus robustes et ne demandent aucun test supplémentaire.

---

## 51.25.1 — Borner la largeur du contenu sur grand écran

Sur un écran de 1600 px, une ligne de texte qui traverse toute la largeur est illisible : l'œil perd sa ligne.

La solution universelle : `ConstrainedBox` + `Center`.

```dart
Center(
  child: ConstrainedBox(
    constraints: const BoxConstraints(maxWidth: 720),
    child: /* votre contenu */,
  ),
)
```

C'est probablement l'ajustement « responsive » au meilleur rapport effort / résultat de tout ce chapitre. Une ligne de texte confortable fait entre 60 et 80 caractères, soit environ 600 à 760 pixels logiques.

---

## 51.26 — `FittedBox`

`FittedBox` **met son enfant à l'échelle** pour qu'il tienne dans l'espace disponible.

```dart
FittedBox(
  fit: BoxFit.scaleDown,   // ne réduit que si nécessaire
  child: Text('27 480 points'),
)
```

Les valeurs de `fit` les plus utiles :

| Valeur | Comportement |
| --- | --- |
| `BoxFit.contain` | agrandit ou réduit pour tenir entièrement, sans déformer |
| `BoxFit.scaleDown` | comme `contain`, mais **n'agrandit jamais** |
| `BoxFit.fitWidth` | ajuste sur la largeur |
| `BoxFit.fill` | déforme pour remplir exactement |

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const AppFittedBox());
}

class AppFittedBox extends StatefulWidget {
  const AppFittedBox({super.key});

  @override
  State<AppFittedBox> createState() => _AppFittedBoxState();
}

class _AppFittedBoxState extends State<AppFittedBox> {
  int _score = 42;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'FittedBox',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.amber),
      ),
      home: Scaffold(
        appBar: AppBar(title: const Text('Score du joueur')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: <Widget>[
              const Text('SANS FittedBox (déborde) :'),
              const SizedBox(height: 8),
              Container(
                height: 90,
                color: Theme.of(context).colorScheme.errorContainer,
                alignment: Alignment.center,
                child: Text(
                  '$_score points',
                  maxLines: 1,
                  style: Theme.of(context).textTheme.displayMedium,
                ),
              ),
              const SizedBox(height: 24),
              const Text('AVEC FittedBox (s\'ajuste) :'),
              const SizedBox(height: 8),
              Container(
                height: 90,
                color: Theme.of(context).colorScheme.primaryContainer,
                alignment: Alignment.center,
                padding: const EdgeInsets.symmetric(horizontal: 12),
                child: FittedBox(
                  fit: BoxFit.scaleDown,
                  child: Text(
                    '$_score points',
                    maxLines: 1,
                    style: Theme.of(context).textTheme.displayMedium,
                  ),
                ),
              ),
              const SizedBox(height: 32),
              FilledButton(
                onPressed: () => setState(() => _score = _score * 10 + 7),
                child: const Text('Multiplier le score'),
              ),
              const SizedBox(height: 8),
              OutlinedButton(
                onPressed: () => setState(() => _score = 42),
                child: const Text('Réinitialiser'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat, après trois appuis (score = 42 777) :**

```text
SANS FittedBox (déborde) :
┌──────────────────────────────┐
│ 42 777 point▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒ │   coupé
└──────────────────────────────┘

AVEC FittedBox (s'ajuste) :
┌──────────────────────────────┐
│      42 777 points           │   réduit, entier
└──────────────────────────────┘
```

---

## 51.26.1 — Les limites de `FittedBox`

`FittedBox` est un outil de dernier recours, pas une solution générale.

1. Il **contourne** le grossissement de police choisi par l'utilisateur : un texte réduit par `FittedBox` peut redevenir minuscule et illisible. Ne l'appliquez pas à du texte de lecture.
2. Il ne fonctionne bien que sur **une seule ligne**. Sur un paragraphe, préférez `maxLines` et `TextOverflow.ellipsis`.
3. Il a besoin de contraintes bornées dans la direction où il ajuste.

Usages légitimes : un grand nombre (score, prix, compteur), un titre court sur une carte, un logo textuel.

---

## 51.27 — Les grilles adaptatives (rappel du chapitre 48)

Au chapitre 48 vous avez utilisé `SliverGridDelegateWithFixedCrossAxisCount`, qui fixe le **nombre** de colonnes.

Pour une grille adaptative, il existe mieux : `SliverGridDelegateWithMaxCrossAxisExtent`, qui fixe la **largeur maximale d'une tuile** et laisse Flutter calculer le nombre de colonnes.

```dart
GridView.builder(
  gridDelegate: const SliverGridDelegateWithMaxCrossAxisExtent(
    maxCrossAxisExtent: 220,   // aucune tuile ne dépassera 220 px
    crossAxisSpacing: 12,
    mainAxisSpacing: 12,
    childAspectRatio: 0.8,
  ),
  itemCount: 20,
  itemBuilder: (BuildContext context, int i) => /* … */,
)
```

```text
   Largeur 400   ->  400 / 220 = 1,8  -> 2 colonnes de 194 px
   Largeur 800   ->  800 / 220 = 3,6  -> 4 colonnes de 197 px
   Largeur 1400  -> 1400 / 220 = 6,4  -> 7 colonnes de 194 px
```

Aucun point de rupture à écrire. Voici les deux approches comparées.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const AppGrilles());
}

class AppGrilles extends StatelessWidget {
  const AppGrilles({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Grilles adaptatives',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.pink),
      ),
      home: const PageGrilles(),
    );
  }
}

class PageGrilles extends StatelessWidget {
  const PageGrilles({super.key});

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 2,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('Boutique'),
          bottom: const TabBar(
            tabs: <Widget>[
              Tab(text: 'maxCrossAxisExtent'),
              Tab(text: 'fixedCrossAxisCount'),
            ],
          ),
        ),
        body: TabBarView(
          children: <Widget>[
            // 1. Nombre de colonnes calculé automatiquement
            GridView.builder(
              padding: const EdgeInsets.all(12),
              gridDelegate:
                  const SliverGridDelegateWithMaxCrossAxisExtent(
                maxCrossAxisExtent: 200,
                crossAxisSpacing: 12,
                mainAxisSpacing: 12,
                childAspectRatio: 0.85,
              ),
              itemCount: 18,
              itemBuilder: (BuildContext context, int i) => _Tuile(index: i),
            ),

            // 2. Nombre de colonnes décidé par un point de rupture
            LayoutBuilder(
              builder: (BuildContext context, BoxConstraints c) {
                final int colonnes = c.maxWidth < 600
                    ? 2
                    : c.maxWidth < 900
                        ? 3
                        : 5;
                return GridView.builder(
                  padding: const EdgeInsets.all(12),
                  gridDelegate:
                      SliverGridDelegateWithFixedCrossAxisCount(
                    crossAxisCount: colonnes,
                    crossAxisSpacing: 12,
                    mainAxisSpacing: 12,
                    childAspectRatio: 0.85,
                  ),
                  itemCount: 18,
                  itemBuilder: (BuildContext ctx, int i) => _Tuile(index: i),
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}

class _Tuile extends StatelessWidget {
  const _Tuile({required this.index});

  final int index;

  @override
  Widget build(BuildContext context) {
    final ColorScheme c = Theme.of(context).colorScheme;
    return Card(
      clipBehavior: Clip.antiAlias,
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.stretch,
        children: <Widget>[
          Expanded(
            child: Container(
              color: c.secondaryContainer,
              child: Icon(Icons.shopping_bag,
                  size: 36, color: c.onSecondaryContainer),
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(8),
            child: Text('Objet ${index + 1}',
                style: Theme.of(context).textTheme.labelLarge),
          ),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
Onglet 1 : le nombre de colonnes change en continu quand on
           redimensionne, les tuiles gardent une taille stable.
Onglet 2 : le nombre de colonnes saute à 600 et 900 px, les
           tuiles s'étirent entre deux sauts.
```

> **Choisissez `maxCrossAxisExtent` quand la taille des tuiles compte** (une vignette de produit doit rester lisible). **Choisissez `crossAxisCount` quand le nombre de colonnes compte** (une grille de calculatrice fait toujours 4 colonnes).

---

## 51.28 — `NavigationRail` sur grand écran

Material 3 recommande de changer de composant de navigation selon la largeur :

```text
   COMPACT (< 600)         MEDIUM / EXPANDED (>= 600)     LARGE (>= 1200)
   ───────────────         ──────────────────────────     ───────────────
   ┌─────────────┐         ┌──┬────────────────────┐      ┌──────┬───────┐
   │             │         │▤ │                    │      │ ▤ Vie│       │
   │   contenu   │         │▤ │      contenu       │      │ ▤ Sac│contenu│
   │             │         │▤ │                    │      │ ▤ Map│       │
   ├─────────────┤         └──┴────────────────────┘      └──────┴───────┘
   │ ▤   ▤   ▤   │
   └─────────────┘         NavigationRail                 NavigationRail
   NavigationBar                                          extended: true
```

`NavigationRail` est un composant Material dont les paramètres principaux sont :

| Paramètre | Type | Rôle |
| --- | --- | --- |
| `destinations` (requis) | `List<NavigationRailDestination>` | les entrées du rail |
| `selectedIndex` (requis) | `int?` | l'index sélectionné |
| `onDestinationSelected` | `ValueChanged<int>?` | rappel de sélection |
| `labelType` | `NavigationRailLabelType?` | `none`, `selected`, `all` |
| `extended` | `bool` (défaut `false`) | rail large avec libellés à droite des icônes |
| `leading` | `Widget?` | widget au-dessus des destinations (souvent un FAB) |
| `trailing` | `Widget?` | widget en dessous |
| `groupAlignment` | `double?` | alignement vertical du groupe : `-1` haut, `0` centre, `1` bas |

`NavigationRailDestination` demande `icon` et `label` (tous deux des `Widget`), et accepte `selectedIcon`, `padding`, `disabled`.

> `labelType` et `extended: true` sont **incompatibles**. Si `extended` vaut `true`, laissez `labelType` à `null` ou à `NavigationRailLabelType.none`.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const AppRail());
}

class AppRail extends StatefulWidget {
  const AppRail({super.key});

  @override
  State<AppRail> createState() => _AppRailState();
}

class _AppRailState extends State<AppRail> {
  int _index = 0;

  static const List<(IconData, IconData, String)> entrees =
      <(IconData, IconData, String)>[
    (Icons.favorite_border, Icons.favorite, 'Personnage'),
    (Icons.backpack_outlined, Icons.backpack, 'Inventaire'),
    (Icons.map_outlined, Icons.map, 'Carte'),
    (Icons.settings_outlined, Icons.settings, 'Réglages'),
  ];

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'NavigationRail',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF4527A0)),
      ),
      home: LayoutBuilder(
        builder: (BuildContext context, BoxConstraints c) {
          final double w = c.maxWidth;
          final bool railVisible = w >= 600;
          final bool railEtendu = w >= 1200;

          final Widget contenu = _Contenu(
            titre: entrees[_index].$3,
            index: _index,
          );

          if (!railVisible) {
            // COMPACT : barre de navigation en bas
            return Scaffold(
              appBar: AppBar(title: Text(entrees[_index].$3)),
              body: contenu,
              bottomNavigationBar: NavigationBar(
                selectedIndex: _index,
                onDestinationSelected: (int i) => setState(() => _index = i),
                destinations: entrees
                    .map((e) => NavigationDestination(
                          icon: Icon(e.$1),
                          selectedIcon: Icon(e.$2),
                          label: e.$3,
                        ))
                    .toList(),
              ),
            );
          }

          // MEDIUM et plus : rail à gauche
          return Scaffold(
            appBar: AppBar(title: Text(entrees[_index].$3)),
            body: Row(
              children: <Widget>[
                NavigationRail(
                  selectedIndex: _index,
                  onDestinationSelected: (int i) => setState(() => _index = i),
                  extended: railEtendu,
                  labelType: railEtendu
                      ? NavigationRailLabelType.none
                      : NavigationRailLabelType.all,
                  groupAlignment: -0.9,
                  leading: Padding(
                    padding: const EdgeInsets.only(top: 8, bottom: 16),
                    child: FloatingActionButton(
                      onPressed: () {},
                      child: const Icon(Icons.save),
                    ),
                  ),
                  destinations: entrees
                      .map((e) => NavigationRailDestination(
                            icon: Icon(e.$1),
                            selectedIcon: Icon(e.$2),
                            label: Text(e.$3),
                          ))
                      .toList(),
                ),
                const VerticalDivider(width: 1, thickness: 1),
                Expanded(child: contenu),
              ],
            ),
          );
        },
      ),
    );
  }
}

class _Contenu extends StatelessWidget {
  const _Contenu({required this.titre, required this.index});

  final String titre;
  final int index;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    return Center(
      child: ConstrainedBox(
        constraints: const BoxConstraints(maxWidth: 640),
        child: Padding(
          padding: const EdgeInsets.all(24),
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: <Widget>[
              Text(titre, style: theme.textTheme.headlineMedium),
              const SizedBox(height: 8),
              Text('Onglet numéro ${index + 1}',
                  style: theme.textTheme.bodyMedium?.copyWith(
                    color: theme.colorScheme.onSurfaceVariant,
                  )),
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
< 600 px   : barre en bas, 4 icônes
600-1199   : rail vertical étroit à gauche, libellés sous les icônes
>= 1200    : rail étendu, libellés à droite des icônes
```

L'état `_index` est le même dans les trois cas : **la navigation ne se perd pas** quand on redimensionne la fenêtre. C'est le point important de ce motif.

---

## 51.29 — Le clavier qui recouvre un champ

Reprenons un problème vu au chapitre 49, avec les outils d'aujourd'hui.

Quand le clavier s'ouvre, `MediaQuery.viewInsetsOf(context).bottom` passe de `0` à la hauteur du clavier.

Par défaut, `Scaffold.resizeToAvoidBottomInset` vaut `true` : le `Scaffold` **réduit** la hauteur de son `body` de cette valeur. Le champ focalisé reste visible si le corps est défilant.

```text
  resizeToAvoidBottomInset: true (défaut)     : false
  ────────────────────────────────────────     ─────────────────────────
  ┌────────────────┐   ┌────────────────┐     ┌────────────────┐
  │                │   │                │     │                │
  │    contenu     │   │    contenu     │     │    contenu     │
  │                │   │ (comprimé)     │     │                │
  │                │   ├────────────────┤     │▓▓▓▓ clavier ▓▓▓│
  │                │   │▓▓▓ clavier ▓▓▓▓│     │ (par-dessus)   │
  └────────────────┘   └────────────────┘     └────────────────┘
```

Mettez `false` seulement si le contenu doit rester intact derrière le clavier — un fond d'écran plein cadre, par exemple.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const AppClavier());
}

class AppClavier extends StatelessWidget {
  const AppClavier({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Clavier',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.lightGreen),
      ),
      home: const PageClavier(),
    );
  }
}

class PageClavier extends StatelessWidget {
  const PageClavier({super.key});

  @override
  Widget build(BuildContext context) {
    final double clavier = MediaQuery.viewInsetsOf(context).bottom;
    final bool ouvert = clavier > 0;
    final ThemeData theme = Theme.of(context);

    return Scaffold(
      // true par défaut : on l'écrit pour être explicite
      resizeToAvoidBottomInset: true,
      appBar: AppBar(title: const Text('Créer un personnage')),
      body: SingleChildScrollView(
        // On ajoute la hauteur du clavier au bas du contenu défilant :
        // le dernier champ reste atteignable.
        padding: EdgeInsets.fromLTRB(16, 16, 16, 16 + clavier),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            Card(
              color: ouvert
                  ? theme.colorScheme.tertiaryContainer
                  : theme.colorScheme.surfaceContainerHighest,
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Text(ouvert
                    ? 'Clavier ouvert : ${clavier.toStringAsFixed(0)} px'
                    : 'Clavier fermé'),
              ),
            ),
            const SizedBox(height: 16),
            for (int i = 1; i <= 6; i++) ...<Widget>[
              TextField(
                decoration: InputDecoration(
                  labelText: 'Champ $i',
                  border: const OutlineInputBorder(),
                ),
              ),
              const SizedBox(height: 12),
            ],
            FilledButton(onPressed: () {}, child: const Text('Créer')),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
Touchez « Champ 6 » : la page défile, le champ reste au-dessus
du clavier, et la carte du haut passe en couleur tertiaire en
affichant la hauteur exacte du clavier.
```

---

## 51.29.1 — Trois pièges à connaître

**Piège 1 — la `Column` non défilante.**
Si votre `body` est une `Column` sans `SingleChildScrollView`, le rétrécissement provoqué par le clavier cause un `RenderFlex overflowed`. **Un formulaire doit toujours être dans une zone défilante.**

**Piège 2 — le bouton collé en bas.**
Si vous voulez un bouton d'action fixé en bas et qu'il doit remonter avec le clavier, utilisez `Scaffold.bottomSheet` ou ajoutez `viewInsets.bottom` en `padding`, comme ci-dessus.

**Piège 3 — `SafeArea` et clavier.**
`SafeArea` utilise `padding`, pas `viewInsets`. Il n'ajoute donc pas d'espace pour le clavier : c'est normal et voulu.

---

## 51.30 — `SafeArea` et les encoches

Les écrans modernes ont des encoches, des poinçons, des coins arrondis et des barres de gestes. Ces zones sont décrites par `MediaQuery.paddingOf(context)`.

`SafeArea` est un widget qui ajoute automatiquement le `padding` nécessaire pour que son enfant ne se retrouve pas dessous.

```dart
SafeArea(
  top: true,      // défaut
  bottom: true,   // défaut
  left: true,
  right: true,
  minimum: EdgeInsets.zero,
  child: /* … */,
)
```

Points importants :

1. **`Scaffold` avec une `AppBar` gère déjà le haut.** N'ajoutez pas un `SafeArea(top: true)` par-dessus : vous doubleriez la marge.
2. `SafeArea` sert surtout quand vous n'avez **pas** d'`AppBar` : un écran plein cadre, un `Stack` d'arrière-plan, une image de fond.
3. Un fond de couleur doit s'étendre **sous** l'encoche ; seul le contenu doit être décalé. Mettez donc `SafeArea` **à l'intérieur** du conteneur coloré, pas autour.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const AppSafeArea());
}

class AppSafeArea extends StatelessWidget {
  const AppSafeArea({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'SafeArea',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepOrange),
      ),
      home: const PageSafeArea(),
    );
  }
}

class PageSafeArea extends StatelessWidget {
  const PageSafeArea({super.key});

  @override
  Widget build(BuildContext context) {
    final EdgeInsets p = MediaQuery.paddingOf(context);
    final ColorScheme c = Theme.of(context).colorScheme;

    return Scaffold(
      // Pas d'AppBar : c'est ici que SafeArea devient indispensable.
      body: Container(
        // Le dégradé va jusqu'aux bords physiques de l'écran…
        decoration: BoxDecoration(
          gradient: LinearGradient(
            begin: Alignment.topCenter,
            end: Alignment.bottomCenter,
            colors: <Color>[c.primaryContainer, c.surface],
          ),
        ),
        child: SafeArea(
          // … mais le contenu, lui, évite l'encoche.
          child: Padding(
            padding: const EdgeInsets.all(16),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: <Widget>[
                Text('Écran plein cadre',
                    style: Theme.of(context).textTheme.headlineSmall),
                const SizedBox(height: 16),
                Text('padding.top    = ${p.top.toStringAsFixed(1)}'),
                Text('padding.bottom = ${p.bottom.toStringAsFixed(1)}'),
                Text('padding.left   = ${p.left.toStringAsFixed(1)}'),
                Text('padding.right  = ${p.right.toStringAsFixed(1)}'),
                const Spacer(),
                FilledButton(
                  onPressed: () {},
                  child: const Text('Bouton hors de la barre de gestes'),
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

**Résultat (téléphone avec encoche) :**

```text
┌──────────────────────────────┐
│░░░░░░░░  ▁▁▁▁  ░░░░░░░░░░░░░ │  le dégradé passe sous l'encoche
│                              │
│  Écran plein cadre           │  le texte commence sous l'encoche
│  padding.top    = 47.0       │
│  padding.bottom = 34.0       │
│                              │
│  [ Bouton hors de la barre ] │
│░░░░░ barre de gestes ░░░░░░░ │  le bouton est au-dessus
└──────────────────────────────┘
```

> Dans un `ListView` plein cadre, `SafeArea` couperait le défilement de façon disgracieuse. Préférez alors ajouter `MediaQuery.paddingOf(context)` au `padding` de la liste : le contenu passe visuellement sous la barre pendant le défilement, mais le premier et le dernier élément restent atteignables.

---

## 51.31 — L'accessibilité : contraste, cibles tactiles, `Semantics`

Une interface adaptative qui n'est pas accessible n'est pas terminée. Trois points suffisent à couvrir l'essentiel.

### 51.31.1 — Le contraste

La règle internationale (WCAG AA) demande un rapport de contraste d'au moins **4,5 : 1** pour le texte courant et **3 : 1** pour le texte de grande taille.

Bonne nouvelle : `ColorScheme.fromSeed` garantit ces rapports pour les paires `xxx` / `onXxx`. **Si vous n'utilisez que des rôles, votre contraste est correct par construction.**

Vous cassez cette garantie dès que vous écrivez :

```dart
Text('Attention', style: TextStyle(color: Colors.yellow))   // sur quel fond ?
Container(color: Colors.grey.shade300, child: const Text('…', style: TextStyle(color: Colors.grey)))
```

Pour renforcer encore le contraste, `ColorScheme.fromSeed` accepte `contrastLevel` (de `0.0` à `1.0`). Vous pouvez le brancher sur `MediaQuery.highContrastOf(context)`.

### 51.31.2 — La taille des cibles tactiles

Une zone touchable doit faire au moins **48 x 48 pixels logiques**. Un doigt n'est pas un curseur de souris.

```dart
// TROP PETIT : une icône de 16 px n'offre que 16 px de cible
GestureDetector(onTap: () {}, child: const Icon(Icons.close, size: 16))

// CORRECT : IconButton garantit 48x48 par défaut
IconButton(onPressed: () {}, icon: const Icon(Icons.close, size: 16))
```

`IconButton`, `ElevatedButton`, `Checkbox`, `Switch` et `Radio` respectent déjà la règle. Le danger vient des `GestureDetector` et `InkWell` posés sur de petits widgets. Enveloppez-les :

```dart
ConstrainedBox(
  constraints: const BoxConstraints(minWidth: 48, minHeight: 48),
  child: /* … */,
)
```

`ThemeData.materialTapTargetSize` contrôle ce comportement globalement : `MaterialTapTargetSize.padded` (48 px garantis) ou `shrinkWrap` (taille réelle du widget).

### 51.31.3 — `Semantics`

Les lecteurs d'écran (TalkBack sur Android, VoiceOver sur iOS) lisent l'**arbre sémantique**, pas l'arbre de widgets. Un `Text` est décrit automatiquement. Une `Icon` seule, non.

| Widget | Rôle |
| --- | --- |
| `Semantics(label: …, child: …)` | ajoute ou remplace la description |
| `Semantics(button: true, …)` | annonce « bouton » |
| `MergeSemantics(child: …)` | fusionne les descriptions des enfants en une seule |
| `ExcludeSemantics(child: …)` | masque un décor purement visuel |
| `Icon(…, semanticLabel: 'Vie')` | raccourci pour les icônes |
| `Image(…, semanticLabel: …)` | raccourci pour les images |

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const AppAccessibilite());
}

class AppAccessibilite extends StatelessWidget {
  const AppAccessibilite({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Accessibilité',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
      ),
      home: const PageAccessibilite(),
    );
  }
}

class PageAccessibilite extends StatelessWidget {
  const PageAccessibilite({super.key});

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return Scaffold(
      appBar: AppBar(title: const Text('Accessibilité')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: <Widget>[
          Text('Contraste garanti par les rôles',
              style: theme.textTheme.titleMedium),
          const SizedBox(height: 8),
          Container(
            padding: const EdgeInsets.all(16),
            color: theme.colorScheme.primaryContainer,
            child: Text('Lisible en clair comme en sombre',
                style: TextStyle(color: theme.colorScheme.onPrimaryContainer)),
          ),
          const SizedBox(height: 24),

          Text('Cibles tactiles', style: theme.textTheme.titleMedium),
          const SizedBox(height: 8),
          Row(
            children: <Widget>[
              // Cible garantie 48x48 par IconButton
              IconButton(
                onPressed: () {},
                icon: const Icon(Icons.remove, size: 18),
                tooltip: 'Retirer un point de vie',
              ),
              const Text('12'),
              IconButton(
                onPressed: () {},
                icon: const Icon(Icons.add, size: 18),
                tooltip: 'Ajouter un point de vie',
              ),
            ],
          ),
          const SizedBox(height: 24),

          Text('Sémantique', style: theme.textTheme.titleMedium),
          const SizedBox(height: 8),

          // Une icône seule : sans label, le lecteur d'écran ne dit rien
          Row(
            children: <Widget>[
              const Icon(Icons.favorite, semanticLabel: 'Points de vie'),
              const SizedBox(width: 8),
              Text('72 / 100', style: theme.textTheme.bodyLarge),
            ],
          ),
          const SizedBox(height: 12),

          // Fusionner un bloc en une seule annonce
          MergeSemantics(
            child: Card(
              child: ListTile(
                leading: const Icon(Icons.shield),
                title: const Text('Bouclier de chêne'),
                subtitle: const Text('Défense 14, poids 6'),
                onTap: () {},
              ),
            ),
          ),
          const SizedBox(height: 12),

          // Décor pur : on l'exclut de l'arbre sémantique
          ExcludeSemantics(
            child: Container(
              height: 6,
              decoration: BoxDecoration(
                color: theme.colorScheme.outlineVariant,
                borderRadius: BorderRadius.circular(3),
              ),
            ),
          ),
          const SizedBox(height: 12),

          // Une barre de progression décrite par une valeur parlante
          Semantics(
            label: 'Progression de la quête',
            value: '45 pour cent',
            child: const LinearProgressIndicator(value: 0.45),
          ),
          const SizedBox(height: 24),

          // Un widget non standard rendu accessible
          Semantics(
            button: true,
            label: 'Lancer le combat',
            child: InkWell(
              onTap: () {},
              borderRadius: BorderRadius.circular(12),
              child: ConstrainedBox(
                constraints: const BoxConstraints(minHeight: 48),
                child: Container(
                  alignment: Alignment.center,
                  decoration: BoxDecoration(
                    color: theme.colorScheme.secondaryContainer,
                    borderRadius: BorderRadius.circular(12),
                  ),
                  child: Text('COMBATTRE',
                      style: TextStyle(
                          color: theme.colorScheme.onSecondaryContainer)),
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

**Résultat annoncé par un lecteur d'écran :**

```text
« Points de vie, 72 / 100 »
« Bouclier de chêne, Défense 14, poids 6 »          (une seule annonce)
« Progression de la quête, 45 pour cent »
« Lancer le combat, bouton »
```

> Sans `MergeSemantics`, la carte serait annoncée en deux fois : « Bouclier de chêne » puis « Défense 14, poids 6 ». Avec, l'utilisateur entend une phrase cohérente.

---

## 51.32 — Tester plusieurs tailles d'écran

Vous n'avez pas besoin de dix appareils.

### 51.32.1 — Redimensionner la fenêtre

Le plus rapide : lancez votre application sur **le bureau ou le web**.

```text
flutter run -d chrome
flutter run -d macos      (ou -d windows, -d linux)
```

Puis tirez le bord de la fenêtre. Vous voyez vos points de rupture se déclencher en direct.

### 51.32.2 — Les outils du navigateur

Sur Chrome : F12, puis l'icône « Toggle device toolbar ». Vous choisissez un appareil, ou vous saisissez une taille exacte. L'onglet Rendering permet aussi de simuler `prefers-color-scheme: dark`.

### 51.32.3 — Un cadre de test intégré à l'application

La technique la plus pratique en développement : envelopper votre application dans un `MediaQuery` truqué.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const BancDEssai());
}

/// Formats simulés, en pixels logiques.
const Map<String, Size> formats = <String, Size>{
  'iPhone SE': Size(320, 568),
  'Téléphone': Size(392, 783),
  'Téléphone paysage': Size(783, 392),
  'Tablette 7"': Size(600, 960),
  'Tablette 10"': Size(840, 1180),
  'Bureau': Size(1280, 800),
};

class BancDEssai extends StatefulWidget {
  const BancDEssai({super.key});

  @override
  State<BancDEssai> createState() => _BancDEssaiState();
}

class _BancDEssaiState extends State<BancDEssai> {
  String _format = 'Téléphone';
  double _echelleTexte = 1.0;
  bool _sombre = false;

  @override
  Widget build(BuildContext context) {
    final Size taille = formats[_format]!;

    return MaterialApp(
      title: 'Banc d\'essai',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blueGrey),
      ),
      home: Scaffold(
        appBar: AppBar(title: const Text('Banc d\'essai')),
        body: Column(
          children: <Widget>[
            // Barre de contrôle
            Padding(
              padding: const EdgeInsets.all(12),
              child: Wrap(
                spacing: 16,
                runSpacing: 8,
                crossAxisAlignment: WrapCrossAlignment.center,
                children: <Widget>[
                  DropdownButton<String>(
                    value: _format,
                    items: formats.keys
                        .map((String k) => DropdownMenuItem<String>(
                            value: k, child: Text(k)))
                        .toList(),
                    onChanged: (String? v) =>
                        setState(() => _format = v ?? _format),
                  ),
                  SizedBox(
                    width: 220,
                    child: Row(
                      children: <Widget>[
                        const Text('Texte'),
                        Expanded(
                          child: Slider(
                            value: _echelleTexte,
                            min: 0.8,
                            max: 2.0,
                            divisions: 12,
                            label: _echelleTexte.toStringAsFixed(1),
                            onChanged: (double v) =>
                                setState(() => _echelleTexte = v),
                          ),
                        ),
                      ],
                    ),
                  ),
                  Row(
                    mainAxisSize: MainAxisSize.min,
                    children: <Widget>[
                      const Text('Sombre'),
                      Switch(
                        value: _sombre,
                        onChanged: (bool v) => setState(() => _sombre = v),
                      ),
                    ],
                  ),
                ],
              ),
            ),
            const Divider(height: 1),

            // La zone de simulation
            Expanded(
              child: Container(
                color: Colors.black26,
                alignment: Alignment.center,
                child: FittedBox(
                  fit: BoxFit.scaleDown,
                  child: Container(
                    width: taille.width,
                    height: taille.height,
                    decoration: BoxDecoration(
                      border: Border.all(color: Colors.black, width: 6),
                      borderRadius: BorderRadius.circular(16),
                    ),
                    clipBehavior: Clip.antiAlias,
                    child: MediaQuery(
                      data: MediaQueryData(
                        size: taille,
                        textScaler: TextScaler.linear(_echelleTexte),
                        platformBrightness:
                            _sombre ? Brightness.dark : Brightness.light,
                        padding: const EdgeInsets.only(top: 24, bottom: 16),
                      ),
                      child: const _ApplicationTestee(),
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

/// L'application réellement testée : elle vit dans le MediaQuery truqué.
///
/// On n'imbrique PAS un second MaterialApp : il réinstallerait son propre
/// MediaQuery à partir de la vraie fenêtre et annulerait la simulation.
/// On applique donc directement un widget Theme.
class _ApplicationTestee extends StatelessWidget {
  const _ApplicationTestee();

  @override
  Widget build(BuildContext context) {
    final bool sombre =
        MediaQuery.platformBrightnessOf(context) == Brightness.dark;

    return Theme(
      data: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: const Color(0xFF00695C),
          brightness: sombre ? Brightness.dark : Brightness.light,
        ),
      ),
      child: const _PageTestee(),
    );
  }
}

class _PageTestee extends StatelessWidget {
  const _PageTestee();

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return LayoutBuilder(
      builder: (BuildContext context, BoxConstraints c) {
        final int colonnes = c.maxWidth < 600 ? 1 : (c.maxWidth < 900 ? 2 : 3);
        return Scaffold(
          appBar: AppBar(title: Text('${c.maxWidth.toStringAsFixed(0)} px')),
          body: GridView.builder(
            padding: const EdgeInsets.all(12),
            gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: colonnes,
              crossAxisSpacing: 12,
              mainAxisSpacing: 12,
              mainAxisExtent: 120,
            ),
            itemCount: 9,
            itemBuilder: (BuildContext context, int i) => Card(
              child: Padding(
                padding: const EdgeInsets.all(12),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: <Widget>[
                    Text('Quête ${i + 1}',
                        style: theme.textTheme.titleMedium),
                    const SizedBox(height: 4),
                    Expanded(
                      child: Text(
                        'Récupérer trois cristaux dans les ruines du nord.',
                        style: theme.textTheme.bodySmall,
                        overflow: TextOverflow.ellipsis,
                        maxLines: 3,
                      ),
                    ),
                  ],
                ),
              ),
            ),
          ),
        );
      },
    );
  }
}
```

**Résultat :**

```text
┌───────────────────────────────────────────────────────┐
│ Banc d'essai                                          │
│ [Téléphone ▾]  Texte ──●───  Sombre [ ]               │
├───────────────────────────────────────────────────────┤
│                ┌────────────────┐                     │
│                │  392 px        │                     │
│                │ ┌────────────┐ │                     │
│                │ │ Quête 1    │ │                     │
│                │ └────────────┘ │                     │
│                └────────────────┘                     │
└───────────────────────────────────────────────────────┘
```

Changez le format : la grille passe à 2 puis 3 colonnes. Poussez le curseur de texte : vous voyez immédiatement ce qui déborde.

### 51.32.4 — Les tests automatisés

Dans un test de widget, `tester.view` permet de fixer la taille simulée :

```dart
testWidgets('bascule en deux colonnes sur tablette', (WidgetTester tester) async {
  tester.view.physicalSize = const Size(1200, 1600);
  tester.view.devicePixelRatio = 1.0;
  addTearDown(tester.view.reset);

  await tester.pumpWidget(const MonApp());
  expect(find.byType(NavigationRail), findsOneWidget);
});
```

C'est la seule façon de garantir qu'une régression de mise en page sera détectée. Les tests de widgets seront approfondis plus loin dans la formation.

### 51.32.5 — La liste de contrôle avant livraison

```text
[ ] 320 px de large (le plus petit téléphone courant)
[ ] 392 px (téléphone standard)
[ ] 600 px (tablette portrait / pliable ouvert)
[ ] 840 px (tablette paysage)
[ ] 1280 px (bureau)
[ ] mode clair
[ ] mode sombre
[ ] grossissement du texte à 200 %
[ ] clavier ouvert sur chaque formulaire
[ ] rotation en cours de saisie
[ ] appareil avec encoche
```

Onze cases. Trente minutes. C'est la différence entre une application qui marche « chez vous » et une application qui marche.

---

## 51.33 — Mini-projet : une application qui s'adapte à tout

Nous allons réunir tout le chapitre dans une seule application : **le compagnon d'aventure**.

### 51.33.1 — Le cahier des charges

```text
FONCTIONNEL
  - une liste de quêtes ;
  - le détail d'une quête ;
  - un écran de réglages avec le choix du thème.

THÈME
  - une seule graine de couleur, un thème clair et un thème sombre ;
  - thèmes de composants centralisés (Card, boutons, AppBar) ;
  - une ThemeExtension pour les couleurs de difficulté ;
  - une échelle d'espacement unique.

ADAPTATION
  - < 600 px  : NavigationBar en bas, liste seule, détail sur une page poussée ;
  - >= 600 px : NavigationRail à gauche, liste seule ;
  - >= 900 px : NavigationRail + liste et détail côte à côte (vue maîtresse-détail) ;
  - contenu borné à 1100 px de large ;
  - respect du grossissement de police et des encoches.
```

### 51.33.2 — Le schéma des trois dispositions

```text
  COMPACT (< 600)          MEDIUM (600-899)          EXPANDED (>= 900)
  ───────────────          ────────────────          ─────────────────
  ┌───────────────┐        ┌──┬─────────────┐        ┌──┬──────┬──────┐
  │  Quêtes       │        │▤ │  Quêtes     │        │▤ │Quêtes│Détail│
  │  ┌─────────┐  │        │▤ │ ┌─────────┐ │        │▤ │ ┌──┐ │      │
  │  │ Quête 1 │  │        │▤ │ │ Quête 1 │ │        │▤ │ │Q1│ │ Q1   │
  │  │ Quête 2 │  │        │  │ │ Quête 2 │ │        │  │ │Q2│ │      │
  │  └─────────┘  │        │  │ └─────────┘ │        │  │ └──┘ │      │
  ├───────────────┤        └──┴─────────────┘        └──┴──────┴──────┘
  │  ▤     ▤      │
  └───────────────┘        rail étroit               rail + 2 panneaux
  barre en bas             détail = page poussée     détail = panneau
```

### 51.33.3 — Le code complet

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const CompagnonApp());
}

// ═══════════════════════════════════════════════════════════════
// 1. LE SYSTÈME DE DESIGN
// ═══════════════════════════════════════════════════════════════

abstract final class Espace {
  static const double xs = 4;
  static const double s = 8;
  static const double m = 16;
  static const double l = 24;
  static const double xl = 32;
}

abstract final class Rupture {
  static const double medium = 600;
  static const double expanded = 900;
  static const double largeurMaxContenu = 1100;
}

/// Couleurs métier absentes de ColorScheme : les niveaux de difficulté.
@immutable
class CouleursDifficulte extends ThemeExtension<CouleursDifficulte> {
  const CouleursDifficulte({
    required this.facile,
    required this.moyenne,
    required this.difficile,
    required this.surDifficulte,
  });

  final Color facile;
  final Color moyenne;
  final Color difficile;
  final Color surDifficulte;

  static const CouleursDifficulte clair = CouleursDifficulte(
    facile: Color(0xFF2E7D32),
    moyenne: Color(0xFFEF6C00),
    difficile: Color(0xFFC62828),
    surDifficulte: Colors.white,
  );

  static const CouleursDifficulte sombre = CouleursDifficulte(
    facile: Color(0xFF81C784),
    moyenne: Color(0xFFFFB74D),
    difficile: Color(0xFFE57373),
    surDifficulte: Colors.black,
  );

  @override
  CouleursDifficulte copyWith({
    Color? facile,
    Color? moyenne,
    Color? difficile,
    Color? surDifficulte,
  }) {
    return CouleursDifficulte(
      facile: facile ?? this.facile,
      moyenne: moyenne ?? this.moyenne,
      difficile: difficile ?? this.difficile,
      surDifficulte: surDifficulte ?? this.surDifficulte,
    );
  }

  @override
  CouleursDifficulte lerp(ThemeExtension<CouleursDifficulte>? other, double t) {
    if (other is! CouleursDifficulte) {
      return this;
    }
    return CouleursDifficulte(
      facile: Color.lerp(facile, other.facile, t)!,
      moyenne: Color.lerp(moyenne, other.moyenne, t)!,
      difficile: Color.lerp(difficile, other.difficile, t)!,
      surDifficulte: Color.lerp(surDifficulte, other.surDifficulte, t)!,
    );
  }
}

extension LectureDifficulte on BuildContext {
  CouleursDifficulte get diff =>
      Theme.of(this).extension<CouleursDifficulte>()!;
}

/// Construit le thème complet pour une luminosité donnée.
ThemeData construireTheme(Brightness brightness) {
  final ColorScheme schema = ColorScheme.fromSeed(
    seedColor: const Color(0xFF3F5AA6),
    brightness: brightness,
  );

  return ThemeData(
    colorScheme: schema,
    appBarTheme: AppBarTheme(
      centerTitle: false,
      elevation: 0,
      scrolledUnderElevation: 2,
      backgroundColor: schema.surface,
      foregroundColor: schema.onSurface,
    ),
    cardTheme: CardThemeData(
      elevation: 0,
      color: schema.surfaceContainerHighest,
      margin: EdgeInsets.zero,
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
    ),
    filledButtonTheme: FilledButtonThemeData(
      style: FilledButton.styleFrom(minimumSize: const Size(0, 48)),
    ),
    outlinedButtonTheme: OutlinedButtonThemeData(
      style: OutlinedButton.styleFrom(minimumSize: const Size(0, 48)),
    ),
    listTileTheme: const ListTileThemeData(
      contentPadding: EdgeInsets.symmetric(horizontal: Espace.m, vertical: 4),
    ),
    dividerTheme: DividerThemeData(color: schema.outlineVariant, space: 1),
    extensions: <ThemeExtension<dynamic>>[
      brightness == Brightness.light
          ? CouleursDifficulte.clair
          : CouleursDifficulte.sombre,
    ],
  );
}

// ═══════════════════════════════════════════════════════════════
// 2. LES DONNÉES
// ═══════════════════════════════════════════════════════════════

enum Difficulte { facile, moyenne, difficile }

class Quete {
  const Quete({
    required this.titre,
    required this.lieu,
    required this.difficulte,
    required this.recompense,
    required this.description,
  });

  final String titre;
  final String lieu;
  final Difficulte difficulte;
  final int recompense;
  final String description;
}

const List<Quete> toutesLesQuetes = <Quete>[
  Quete(
    titre: 'Les rats de la cave',
    lieu: 'Auberge du Sanglier',
    difficulte: Difficulte.facile,
    recompense: 25,
    description:
        'L\'aubergiste se plaint de bruits sous le plancher. Descendez '
        'dans la cave et débarrassez-le des rongeurs. Attention : ils '
        'sont plus nombreux qu\'annoncé.',
  ),
  Quete(
    titre: 'La caravane disparue',
    lieu: 'Route de l\'Est',
    difficulte: Difficulte.moyenne,
    recompense: 140,
    description:
        'Une caravane marchande n\'est jamais arrivée. Retrouvez sa trace '
        'entre le pont de pierre et le col. Des empreintes de gobelins '
        'ont été signalées.',
  ),
  Quete(
    titre: 'Le sanctuaire scellé',
    lieu: 'Ruines de Kaldar',
    difficulte: Difficulte.difficile,
    recompense: 620,
    description:
        'Une rune ancienne scelle la porte du sanctuaire. Trouvez les trois '
        'fragments dispersés dans les ruines, puis affrontez ce qui dort '
        'derrière la porte.',
  ),
  Quete(
    titre: 'Herbes pour l\'apothicaire',
    lieu: 'Clairière du nord',
    difficulte: Difficulte.facile,
    recompense: 40,
    description:
        'Rapportez douze racines de mandragore. Elles poussent près des '
        'souches humides et crient quand on les arrache.',
  ),
  Quete(
    titre: 'Le loup blanc',
    lieu: 'Forêt de Verdegris',
    difficulte: Difficulte.moyenne,
    recompense: 210,
    description:
        'Une bête énorme rôde autour des troupeaux. Les bergers demandent '
        'qu\'on la chasse. Certains prétendent qu\'elle parle.',
  ),
];

// ═══════════════════════════════════════════════════════════════
// 3. L'APPLICATION
// ═══════════════════════════════════════════════════════════════

class CompagnonApp extends StatefulWidget {
  const CompagnonApp({super.key});

  @override
  State<CompagnonApp> createState() => _CompagnonAppState();
}

class _CompagnonAppState extends State<CompagnonApp> {
  ThemeMode _mode = ThemeMode.system;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Compagnon d\'aventure',
      debugShowCheckedModeBanner: false,
      theme: construireTheme(Brightness.light),
      darkTheme: construireTheme(Brightness.dark),
      themeMode: _mode,
      home: Coquille(
        modeCourant: _mode,
        onModeChange: (ThemeMode m) => setState(() => _mode = m),
      ),
    );
  }
}

// ═══════════════════════════════════════════════════════════════
// 4. LA COQUILLE ADAPTATIVE
// ═══════════════════════════════════════════════════════════════

class Coquille extends StatefulWidget {
  const Coquille({
    super.key,
    required this.modeCourant,
    required this.onModeChange,
  });

  final ThemeMode modeCourant;
  final ValueChanged<ThemeMode> onModeChange;

  @override
  State<Coquille> createState() => _CoquilleState();
}

class _CoquilleState extends State<Coquille> {
  int _onglet = 0;
  int _queteSelectionnee = 0;

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (BuildContext context, BoxConstraints c) {
        final double largeur = c.maxWidth;
        final bool rail = largeur >= Rupture.medium;
        final bool deuxPanneaux = largeur >= Rupture.expanded;

        final Widget corps = _corps(deuxPanneaux);

        if (!rail) {
          return Scaffold(
            appBar: AppBar(title: Text(_titre())),
            body: corps,
            bottomNavigationBar: NavigationBar(
              selectedIndex: _onglet,
              onDestinationSelected: (int i) => setState(() => _onglet = i),
              destinations: const <NavigationDestination>[
                NavigationDestination(
                  icon: Icon(Icons.flag_outlined),
                  selectedIcon: Icon(Icons.flag),
                  label: 'Quêtes',
                ),
                NavigationDestination(
                  icon: Icon(Icons.settings_outlined),
                  selectedIcon: Icon(Icons.settings),
                  label: 'Réglages',
                ),
              ],
            ),
          );
        }

        return Scaffold(
          appBar: AppBar(title: Text(_titre())),
          body: Row(
            children: <Widget>[
              NavigationRail(
                selectedIndex: _onglet,
                onDestinationSelected: (int i) => setState(() => _onglet = i),
                extended: largeur >= 1200,
                labelType: largeur >= 1200
                    ? NavigationRailLabelType.none
                    : NavigationRailLabelType.all,
                groupAlignment: -0.9,
                destinations: const <NavigationRailDestination>[
                  NavigationRailDestination(
                    icon: Icon(Icons.flag_outlined),
                    selectedIcon: Icon(Icons.flag),
                    label: Text('Quêtes'),
                  ),
                  NavigationRailDestination(
                    icon: Icon(Icons.settings_outlined),
                    selectedIcon: Icon(Icons.settings),
                    label: Text('Réglages'),
                  ),
                ],
              ),
              const VerticalDivider(width: 1, thickness: 1),
              Expanded(child: corps),
            ],
          ),
        );
      },
    );
  }

  String _titre() => _onglet == 0 ? 'Quêtes disponibles' : 'Réglages';

  Widget _corps(bool deuxPanneaux) {
    if (_onglet == 1) {
      return PageReglagesTheme(
        modeCourant: widget.modeCourant,
        onModeChange: widget.onModeChange,
      );
    }

    if (deuxPanneaux) {
      return Center(
        child: ConstrainedBox(
          constraints: const BoxConstraints(
            maxWidth: Rupture.largeurMaxContenu,
          ),
          child: Row(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: <Widget>[
              SizedBox(
                width: 340,
                child: ListeQuetes(
                  indexSelectionne: _queteSelectionnee,
                  onSelection: (int i) =>
                      setState(() => _queteSelectionnee = i),
                ),
              ),
              const VerticalDivider(width: 1, thickness: 1),
              Expanded(
                child: DetailQuete(
                  quete: toutesLesQuetes[_queteSelectionnee],
                  dansUnPanneau: true,
                ),
              ),
            ],
          ),
        ),
      );
    }

    return Center(
      child: ConstrainedBox(
        constraints: const BoxConstraints(maxWidth: 720),
        child: ListeQuetes(
          indexSelectionne: null,
          onSelection: (int i) {
            Navigator.of(context).push(
              MaterialPageRoute<void>(
                builder: (_) => Scaffold(
                  appBar: AppBar(title: const Text('Détail de la quête')),
                  body: DetailQuete(
                    quete: toutesLesQuetes[i],
                    dansUnPanneau: false,
                  ),
                ),
              ),
            );
          },
        ),
      ),
    );
  }
}

// ═══════════════════════════════════════════════════════════════
// 5. LES ÉCRANS
// ═══════════════════════════════════════════════════════════════

class ListeQuetes extends StatelessWidget {
  const ListeQuetes({
    super.key,
    required this.indexSelectionne,
    required this.onSelection,
  });

  final int? indexSelectionne;
  final ValueChanged<int> onSelection;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return ListView.separated(
      padding: EdgeInsets.fromLTRB(
        Espace.m,
        Espace.m,
        Espace.m,
        Espace.m + MediaQuery.paddingOf(context).bottom,
      ),
      itemCount: toutesLesQuetes.length,
      separatorBuilder: (_, __) => const SizedBox(height: Espace.s),
      itemBuilder: (BuildContext context, int i) {
        final Quete q = toutesLesQuetes[i];
        final bool actif = indexSelectionne == i;

        return Card(
          color: actif
              ? theme.colorScheme.secondaryContainer
              : theme.colorScheme.surfaceContainerHighest,
          child: InkWell(
            borderRadius: BorderRadius.circular(16),
            onTap: () => onSelection(i),
            child: Padding(
              padding: const EdgeInsets.all(Espace.m),
              child: Row(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: <Widget>[
                  PastilleDifficulte(difficulte: q.difficulte),
                  const SizedBox(width: Espace.m),
                  Expanded(
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: <Widget>[
                        Text(q.titre, style: theme.textTheme.titleMedium),
                        const SizedBox(height: Espace.xs),
                        Text(
                          q.lieu,
                          style: theme.textTheme.bodySmall?.copyWith(
                            color: theme.colorScheme.onSurfaceVariant,
                          ),
                        ),
                        const SizedBox(height: Espace.s),
                        Row(
                          children: <Widget>[
                            Icon(Icons.paid,
                                size: 16,
                                color: theme.colorScheme.onSurfaceVariant),
                            const SizedBox(width: Espace.xs),
                            Text('${q.recompense} pièces',
                                style: theme.textTheme.labelMedium),
                          ],
                        ),
                      ],
                    ),
                  ),
                ],
              ),
            ),
          ),
        );
      },
    );
  }
}

class PastilleDifficulte extends StatelessWidget {
  const PastilleDifficulte({super.key, required this.difficulte});

  final Difficulte difficulte;

  Color _couleur(BuildContext context) {
    final CouleursDifficulte d = context.diff;
    switch (difficulte) {
      case Difficulte.facile:
        return d.facile;
      case Difficulte.moyenne:
        return d.moyenne;
      case Difficulte.difficile:
        return d.difficile;
    }
  }

  IconData get _icone {
    switch (difficulte) {
      case Difficulte.facile:
        return Icons.sentiment_satisfied;
      case Difficulte.moyenne:
        return Icons.sentiment_neutral;
      case Difficulte.difficile:
        return Icons.local_fire_department;
    }
  }

  @override
  Widget build(BuildContext context) {
    final Color couleur = _couleur(context);
    return Semantics(
      label: 'Difficulté ${difficulte.name}',
      child: Container(
        width: 44,
        height: 44,
        decoration: BoxDecoration(
          color: couleur,
          borderRadius: BorderRadius.circular(12),
        ),
        child: Icon(_icone, color: context.diff.surDifficulte),
      ),
    );
  }
}

class DetailQuete extends StatelessWidget {
  const DetailQuete({
    super.key,
    required this.quete,
    required this.dansUnPanneau,
  });

  final Quete quete;
  final bool dansUnPanneau;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return LayoutBuilder(
      builder: (BuildContext context, BoxConstraints c) {
        // Les actions passent en colonne si la largeur est trop faible.
        final bool actionsEnLigne = c.maxWidth >= 380;

        final List<Widget> actions = <Widget>[
          FilledButton.icon(
            onPressed: () {
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(content: Text('Quête acceptée : ${quete.titre}')),
              );
            },
            icon: const Icon(Icons.check),
            label: const Text('Accepter'),
          ),
          OutlinedButton.icon(
            onPressed: () {},
            icon: const Icon(Icons.map),
            label: const Text('Voir sur la carte'),
          ),
        ];

        return ListView(
          padding: EdgeInsets.fromLTRB(
            Espace.l,
            Espace.l,
            Espace.l,
            Espace.l + MediaQuery.paddingOf(context).bottom,
          ),
          children: <Widget>[
            Row(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: <Widget>[
                PastilleDifficulte(difficulte: quete.difficulte),
                const SizedBox(width: Espace.m),
                Expanded(
                  child: Text(quete.titre,
                      style: theme.textTheme.headlineSmall),
                ),
              ],
            ),
            const SizedBox(height: Espace.s),
            Text(
              quete.lieu,
              style: theme.textTheme.titleSmall?.copyWith(
                color: theme.colorScheme.onSurfaceVariant,
              ),
            ),
            const SizedBox(height: Espace.l),
            Text(quete.description, style: theme.textTheme.bodyLarge),
            const SizedBox(height: Espace.l),
            Card(
              color: theme.colorScheme.primaryContainer,
              child: Padding(
                padding: const EdgeInsets.all(Espace.m),
                child: Row(
                  children: <Widget>[
                    Icon(Icons.paid, color: theme.colorScheme.onPrimaryContainer),
                    const SizedBox(width: Espace.m),
                    Expanded(
                      child: Text(
                        'Récompense : ${quete.recompense} pièces d\'or',
                        style: theme.textTheme.titleMedium?.copyWith(
                          color: theme.colorScheme.onPrimaryContainer,
                        ),
                      ),
                    ),
                  ],
                ),
              ),
            ),
            const SizedBox(height: Espace.xl),
            if (actionsEnLigne)
              Row(
                children: <Widget>[
                  Expanded(child: actions[0]),
                  const SizedBox(width: Espace.m),
                  Expanded(child: actions[1]),
                ],
              )
            else
              Column(
                crossAxisAlignment: CrossAxisAlignment.stretch,
                children: <Widget>[
                  actions[0],
                  const SizedBox(height: Espace.s),
                  actions[1],
                ],
              ),
          ],
        );
      },
    );
  }
}

class PageReglagesTheme extends StatelessWidget {
  const PageReglagesTheme({
    super.key,
    required this.modeCourant,
    required this.onModeChange,
  });

  final ThemeMode modeCourant;
  final ValueChanged<ThemeMode> onModeChange;

  String _libelle(ThemeMode m) {
    switch (m) {
      case ThemeMode.system:
        return 'Selon le système';
      case ThemeMode.light:
        return 'Toujours clair';
      case ThemeMode.dark:
        return 'Toujours sombre';
    }
  }

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return Center(
      child: ConstrainedBox(
        constraints: const BoxConstraints(maxWidth: 640),
        child: ListView(
          padding: const EdgeInsets.all(Espace.m),
          children: <Widget>[
            Text('Apparence', style: theme.textTheme.titleLarge),
            const SizedBox(height: Espace.s),
            Card(
              child: Column(
                children: ThemeMode.values.map((ThemeMode m) {
                  return RadioListTile<ThemeMode>(
                    value: m,
                    groupValue: modeCourant,
                    onChanged: (ThemeMode? v) {
                      if (v != null) {
                        onModeChange(v);
                      }
                    },
                    title: Text(_libelle(m)),
                  );
                }).toList(),
              ),
            ),
            const SizedBox(height: Espace.l),
            Text('Diagnostic', style: theme.textTheme.titleLarge),
            const SizedBox(height: Espace.s),
            Card(
              child: Padding(
                padding: const EdgeInsets.all(Espace.m),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: <Widget>[
                    Text('Largeur de fenêtre : '
                        '${MediaQuery.sizeOf(context).width.toStringAsFixed(0)} px'),
                    Text('Orientation : '
                        '${MediaQuery.orientationOf(context).name}'),
                    Text('Texte 14 px peint à '
                        '${MediaQuery.textScalerOf(context).scale(14).toStringAsFixed(1)} px'),
                    Text('Thème appliqué : ${theme.brightness.name}'),
                  ],
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

**Résultat, à 400 px de large :**

```text
┌────────────────────────────────────┐
│  Quêtes disponibles                │
├────────────────────────────────────┤
│  ┌──────────────────────────────┐  │
│  │ (facile)  Les rats de la cave│  │
│  │           Auberge du Sanglier│  │
│  │           25 pieces          │  │
│  └──────────────────────────────┘  │
│  ┌──────────────────────────────┐  │
│  │ (moyen)   La caravane dispar.│  │
│  └──────────────────────────────┘  │
├────────────────────────────────────┤
│      Quetes            Reglages    │
└────────────────────────────────────┘
```

**Résultat, à 1000 px de large :**

```text
┌────────┬──────────────────┬────────────────────────────────┐
│ Quetes │ Les rats de…     │ Les rats de la cave            │
│ Regla. │ La caravane…     │ Auberge du Sanglier            │
│        │ Le sanctuaire…   │                                │
│        │ Herbes pour…     │ L'aubergiste se plaint de…     │
│        │ Le loup blanc    │ ╔════════════════════════════╗ │
│        │                  │ ║ Recompense : 25 pieces     ║ │
│        │                  │ ╚════════════════════════════╝ │
│        │                  │ [ Accepter ] [ Voir la carte ] │
└────────┴──────────────────┴────────────────────────────────┘
```

### 51.33.4 — Ce que ce projet démontre

| Notion | Où elle est utilisée |
| --- | --- |
| `ColorScheme.fromSeed` | `construireTheme`, une seule graine |
| Rôles de couleur | partout, aucune couleur en dur dans les pages |
| Thèmes de composants | `appBarTheme`, `cardTheme`, `filledButtonTheme`, `listTileTheme` |
| `darkTheme` + `themeMode` | `CompagnonApp`, piloté par les réglages |
| `ThemeExtension` | `CouleursDifficulte`, deux variantes clair / sombre |
| Constantes de design | `Espace`, `Rupture` |
| `LayoutBuilder` | `Coquille` (mise en page globale) et `DetailQuete` (actions) |
| `MediaQuery` | `paddingOf`, `sizeOf`, `orientationOf`, `textScalerOf` |
| Points de rupture | 600 et 900, plus 1200 pour le rail étendu |
| `NavigationRail` / `NavigationBar` | selon la largeur, sans perdre l'état |
| Largeur bornée | `ConstrainedBox(maxWidth: 1100)` et `720` |
| Accessibilité | `Semantics` sur les pastilles, boutons de 48 px |

### 51.33.5 — Extensions à réaliser seul

1. Ajouter un troisième onglet « Personnage » avec une fiche adaptative.
2. Filtrer les quêtes par difficulté avec des `FilterChip`.
3. Persister le `ThemeMode` avec `shared_preferences` (après le chapitre 54).
4. Ajouter un sélecteur de couleur de graine dans les réglages : six pastilles, la palette entière suit.
5. Ajouter une `ThemeExtension` `Dimensions` qui rend les marges plus généreuses au-delà de 900 px.
6. Sur écran très large (≥ 1400 px), ajouter un troisième panneau « journal » à droite.

---

## 51.34 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| Les couleurs sont écrites en dur (`Colors.white`, `Color(0xFF212121)`) et le mode sombre est illisible | On peint sans passer par le thème | Utiliser les rôles : `colorScheme.surface`, `onSurface`, `primaryContainer`… |
| `Theme.of(context)` renvoie un thème bleu par défaut | `Theme.of` est appelé dans le `build` qui crée le `MaterialApp`, donc au-dessus de lui | Extraire la page dans un widget enfant, ou utiliser un `Builder` |
| `MediaQuery.of(context)` lève « No MediaQuery widget ancestor found » | Appelé hors de tout `MaterialApp` / `WidgetsApp`, ou dans le `build` qui le crée | Placer l'appel sous le `MaterialApp` (widget enfant ou `Builder`) |
| Le « thème sombre » reste clair | `brightness: Brightness.dark` oublié dans `ColorScheme.fromSeed` du `darkTheme` | Passer `brightness: Brightness.dark` à `fromSeed`, pas seulement à `ThemeData` |
| `The argument type 'CardTheme' can't be assigned to 'CardThemeData'` | `ThemeData.cardTheme` attend `CardThemeData` dans les versions récentes | Écrire `CardThemeData(...)`, avec le suffixe `Data` |
| `ColorScheme.fromSeed(seedColor: rouge)` ne donne pas ce rouge | La graine est une indication de teinte, pas une couleur imposée | Accepter la couleur calculée, ou forcer `primary:` et `onPrimary:` explicitement |
| Redéfinir `textTheme` fait perdre toutes les autres propriétés du style | On a construit un `TextStyle` neuf au lieu de copier celui du thème | `base.textTheme.copyWith(bodyMedium: base.textTheme.bodyMedium?.copyWith(...))` |
| `RenderFlex overflowed` dès que l'utilisateur grossit la police | Hauteur fixe autour d'un texte, ou `Text` non enveloppé dans une `Row` | Supprimer la hauteur fixe, envelopper dans `Expanded` / `Flexible`, ou utiliser `Wrap` |
| Le texte est coupé sur les petits téléphones | On a supposé une largeur minimale de 400 px | Tester à 320 px, borner avec `maxLines` + `TextOverflow.ellipsis` |
| Un widget réutilisable affiche la mauvaise disposition dans un panneau | Il décide avec `MediaQuery.sizeOf` (fenêtre) au lieu de ses contraintes réelles | Utiliser `LayoutBuilder` et `constraints.maxWidth` dans les composants |
| Un test sur `constraints.maxHeight` ne se déclenche jamais | Dans une zone défilante, `maxHeight` vaut `double.infinity` | Décider sur `maxWidth`, ou borner explicitement la hauteur |
| Le clavier provoque un débordement dans un formulaire | Le `body` est une `Column` non défilante et le `Scaffold` la comprime | Envelopper dans `SingleChildScrollView` ou utiliser une `ListView` |
| Le contenu passe sous l'encoche sur un écran plein cadre | Pas de `SafeArea`, ou `SafeArea` posé autour du fond coloré | Placer `SafeArea` **à l'intérieur** du conteneur coloré, autour du seul contenu |
| Double marge en haut de l'écran | `SafeArea(top: true)` ajouté alors que le `Scaffold` a déjà une `AppBar` | Retirer le `SafeArea`, ou mettre `top: false` |
| Le sélecteur de thème se réinitialise à chaque lancement | `ThemeMode` n'est stocké qu'en mémoire | Persister avec `shared_preferences` (chapitre 54) |
| `Theme.of(context).extension<X>()` renvoie `null` | L'extension n'a pas été enregistrée dans `ThemeData.extensions` | Ajouter l'instance à `extensions: <ThemeExtension<dynamic>>[ ... ]`, dans les deux thèmes |
| `Theme(data: …)` n'a aucun effet sur les widgets du même `build` | Le `context` utilisé est au-dessus du nouveau `Theme` | Insérer un `Builder` sous le `Theme` |
| Une ligne de texte traverse 1600 px sur bureau et devient illisible | Aucune largeur maximale | `Center` + `ConstrainedBox(maxWidth: 720)` |
| Sur tablette, la grille garde deux colonnes énormes | `crossAxisCount` fixe | `SliverGridDelegateWithMaxCrossAxisExtent`, ou `crossAxisCount` calculé par `LayoutBuilder` |
| Une petite icône cliquable est impossible à toucher | Cible tactile inférieure à 48 px | `IconButton`, ou `ConstrainedBox(minWidth: 48, minHeight: 48)` |
| Le lecteur d'écran ne dit rien sur une icône informative | Une `Icon` seule n'a pas de description | Renseigner `semanticLabel`, ou envelopper dans `Semantics(label: …)` |
| `textScaleFactor` provoque un avertissement de dépréciation | La propriété est remplacée | Utiliser `MediaQuery.textScalerOf(context)` et `TextScaler` |
| Le rail se plante avec « labelType and extended cannot both be set » | `extended: true` combiné à `labelType: all` ou `selected` | Mettre `labelType` à `null` ou `NavigationRailLabelType.none` quand `extended` est vrai |

---

## 51.35 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| `ThemeData` | le catalogue de styles de toute l'application ; immuable, on le copie avec `copyWith` |
| `MaterialApp.theme` | l'unique endroit où l'on branche le thème clair |
| `useMaterial3` | vaut `true` par défaut depuis Flutter 3.16 ; ne le mettez jamais à `false` dans un projet neuf |
| `ColorScheme.fromSeed()` | une couleur en entrée, une palette complète et contrastée en sortie |
| Rôles de couleur | on choisit `primary`, `surface`, `error`… selon l'**usage**, jamais une couleur littérale |
| Paires `xxx` / `onXxx` | du texte posé sur `xxx` se peint en `onXxx` ; le contraste est garanti |
| `Theme.of(context)` | lit le thème le plus proche et **abonne** le widget à ses changements |
| `ColorScheme.of(context)` | raccourci équivalent à `Theme.of(context).colorScheme` |
| `textTheme` | 15 styles nommés, de `displayLarge` (57 px) à `labelSmall` (11 px) |
| Personnaliser un style | toujours `styleDuTheme?.copyWith(...)`, jamais un `TextStyle` reconstruit de zéro |
| Thèmes de composants | `appBarTheme`, `cardTheme` (type `CardThemeData`), `elevatedButtonTheme`, `inputDecorationTheme`… |
| `darkTheme` | un second `ThemeData` construit avec `brightness: Brightness.dark`, même graine |
| `themeMode` | `system` (défaut), `light` ou `dark` ; le mettre dans un `State` pour le rendre modifiable |
| Persistance du thème | à faire avec `shared_preferences` au chapitre 54 |
| Widget `Theme` | applique un thème à un sous-arbre ; ajouter un `Builder` pour le lire |
| `ThemeExtension` | vos propres jeux de valeurs dans le thème ; exige `copyWith` et `lerp` |
| Constantes de design | une échelle d'espacements (4, 8, 16, 24, 32) et de rayons, dans une `abstract final class` |
| `MediaQuery` | décrit la **fenêtre** : `size`, `orientation`, `textScaler`, `padding`, `viewInsets` |
| `sizeOf`, `paddingOf`… | accesseurs ciblés : moins de reconstructions inutiles que `MediaQuery.of` |
| `textScaler` | remplace `textScaleFactor` (déprécié) ; `scale(16)` donne la taille peinte |
| `MediaQuery.withClampedTextScaling` | borne le grossissement, à réserver aux cas où la mise en page ne peut pas s'adapter |
| `LayoutBuilder` | donne les **contraintes réelles** de la zone : le bon outil dans un composant réutilisable |
| `MediaQuery` vs `LayoutBuilder` | fenêtre = décision globale ; contraintes = décision locale |
| Points de rupture | Material 3 : 600, 840, 1200, 1600 ; deux suffisent souvent |
| `Column` ↔ `Row` | on écrit le contenu **une fois** et on change seulement son arrangement |
| `OrientationBuilder` | utile quand la **hauteur** est limitante ; sinon, préférer la largeur |
| `Expanded` / `Flexible` | des proportions robustes, souvent préférables à un point de rupture |
| `ConstrainedBox(maxWidth: 720)` | l'ajustement grand écran au meilleur rapport effort / résultat |
| `FittedBox` | met à l'échelle un contenu court (un score, un titre) ; jamais du texte de lecture |
| `SliverGridDelegateWithMaxCrossAxisExtent` | grille adaptative sans aucun point de rupture |
| `NavigationRail` | remplace la `NavigationBar` au-delà de 600 px ; `extended: true` au-delà de 1200 |
| `resizeToAvoidBottomInset` | `true` par défaut ; un formulaire doit être dans une zone défilante |
| `viewInsets.bottom` | hauteur du clavier ; `padding` / `viewPadding` concernent les zones système |
| `SafeArea` | à placer **dans** le conteneur coloré, autour du seul contenu |
| Accessibilité | contraste 4,5:1 (garanti par les rôles), cibles de 48 px, `Semantics` et `semanticLabel` |
| Tester | redimensionner une fenêtre desktop/web, un banc d'essai `MediaQuery`, `tester.view` en test |

---

## 51.36 — Exercices

### Exercice 1 — La graine unique (facile)

Écrivez une application dont le thème est entièrement dérivé de la graine `Color(0xFF8E24AA)`. La page affiche un `AppBar`, trois `Card` contenant chacune un `ListTile`, et un `FilledButton`. **Aucune couleur littérale** ne doit apparaître ailleurs que dans la graine.

### Exercice 2 — La palette de rôles (facile)

Affichez une liste de huit bandes de 60 px de haut. Chaque bande est peinte avec un rôle du `colorScheme` (`primary`, `primaryContainer`, `secondary`, `secondaryContainer`, `tertiary`, `error`, `surface`, `inverseSurface`), et porte son nom écrit dans la couleur `onXxx` correspondante.

### Exercice 3 — Le thème typographique (facile)

Créez une application dont le thème redéfinit `headlineMedium` (poids 800) et `bodyLarge` (interligne 1.6), **en partant du thème de base**. Affichez un titre et deux paragraphes qui utilisent ces styles. Ajoutez un troisième texte qui reprend `bodyLarge` mais en couleur `error`, sans toucher au thème.

### Exercice 4 — Le sélecteur de thème (moyen)

Écrivez une application à deux écrans reliés par une `NavigationBar` : « Accueil » et « Réglages ». L'écran des réglages propose les trois `ThemeMode` par `SegmentedButton`. L'application entière doit réagir instantanément. Affichez aussi sur l'accueil la luminosité effectivement appliquée.

### Exercice 5 — Les couleurs métier (moyen)

Créez une `ThemeExtension` nommée `CouleursEtat` contenant `succes`, `avertissement`, `info` et `surEtat`, avec une variante claire et une variante sombre. Affichez trois bandeaux d'état qui l'utilisent. Ajoutez un raccourci `context.etats` par extension sur `BuildContext`.

### Exercice 6 — Le tableau de bord `MediaQuery` (moyen)

Affichez en direct : la largeur, la hauteur, l'orientation, `padding.top`, `viewInsets.bottom`, la luminosité système et la taille peinte d'un texte de 16 px. Ajoutez un `TextField` en bas pour observer `viewInsets.bottom` changer. Utilisez les accesseurs ciblés, pas `MediaQuery.of`.

### Exercice 7 — La carte qui bascule (moyen)

Écrivez un widget `CarteHeros` qui affiche un avatar, un nom, une classe et deux boutons. En dessous de 500 px de largeur **disponible**, il empile tout en colonne ; au-dessus, il place l'avatar à gauche et le reste à droite. Le widget doit donner le bon résultat même placé dans un panneau étroit d'un écran large : prouvez-le en l'affichant deux fois côte à côte, dans un `SizedBox(width: 300)` et dans un `Expanded`.

### Exercice 8 — La grille sans point de rupture (moyen)

Affichez une grille de 24 objets dont les tuiles ne dépassent jamais 180 px de large et gardent un rapport largeur/hauteur de 0,75. Ajoutez en haut de page un texte indiquant le nombre de colonnes réellement obtenu (calculez-le vous-même à partir des contraintes).

### Exercice 9 — La coquille adaptative (difficile)

Écrivez une coquille de navigation à trois destinations qui affiche :
une `NavigationBar` en bas sous 600 px, un `NavigationRail` étroit entre 600 et 1199 px, un `NavigationRail` étendu à partir de 1200 px.
L'index sélectionné doit être conservé lors du redimensionnement. Chaque page borne son contenu à 800 px de large et respecte `MediaQuery.paddingOf`.

### Exercice 10 — Résister au grossissement (difficile)

Écrivez une page contenant :
un en-tête avec un titre et un badge de niveau ; une ligne d'icône + texte long ; une liste de trois lignes « libellé / valeur » ; un pied de page avec deux boutons.
Ajoutez un `Slider` qui simule un `textScaler` de 0,8 à 2,5 via `MaterialApp.builder`.
La page ne doit **jamais** produire de débordement, quel que soit le facteur. Le badge de niveau, lui, ne doit pas grossir au-delà de 1,3.

---

## 51.37 — Corrections des exercices

### Correction 1

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const Exo1());
}

const Color graine = Color(0xFF8E24AA);

class Exo1 extends StatelessWidget {
  const Exo1({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Exercice 1',
      theme: ThemeData(colorScheme: ColorScheme.fromSeed(seedColor: graine)),
      darkTheme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: graine,
          brightness: Brightness.dark,
        ),
      ),
      home: const PageExo1(),
    );
  }
}

class PageExo1 extends StatelessWidget {
  const PageExo1({super.key});

  static const List<(IconData, String, String)> lignes =
      <(IconData, String, String)>[
    (Icons.shield, 'Bouclier de chêne', 'Défense 14'),
    (Icons.colorize, 'Épée courte', 'Dégâts 8-12'),
    (Icons.local_drink, 'Potion de soin', 'Rend 40 PV'),
  ];

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Inventaire'),
        backgroundColor: theme.colorScheme.primaryContainer,
        foregroundColor: theme.colorScheme.onPrimaryContainer,
      ),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: <Widget>[
          for (final (IconData icone, String titre, String sous) in lignes)
            Padding(
              padding: const EdgeInsets.only(bottom: 12),
              child: Card(
                child: ListTile(
                  leading: Icon(icone, color: theme.colorScheme.primary),
                  title: Text(titre),
                  subtitle: Text(sous),
                  trailing: const Icon(Icons.chevron_right),
                ),
              ),
            ),
          const SizedBox(height: 12),
          FilledButton.icon(
            onPressed: () {
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(content: Text('Inventaire trié.')),
              );
            },
            icon: const Icon(Icons.sort),
            label: const Text('Trier l\'inventaire'),
          ),
        ],
      ),
    );
  }
}
```

**Explication :** la constante `graine` est le seul endroit du fichier où une valeur de couleur est écrite. `ColorScheme.fromSeed` en dérive tout le reste, et la même graine sert au thème sombre avec `brightness: Brightness.dark`. Les `Card`, `ListTile` et `FilledButton` prennent automatiquement leurs couleurs du `colorScheme` ; les seules couleurs mentionnées dans la page sont des **rôles** (`primaryContainer`, `onPrimaryContainer`, `primary`). Changez `graine` en `Colors.green` : toute l'application suit.

---

### Correction 2

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const Exo2());
}

class Exo2 extends StatelessWidget {
  const Exo2({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Exercice 2',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF00695C)),
      ),
      darkTheme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: const Color(0xFF00695C),
          brightness: Brightness.dark,
        ),
      ),
      home: const PagePalette(),
    );
  }
}

class PagePalette extends StatelessWidget {
  const PagePalette({super.key});

  @override
  Widget build(BuildContext context) {
    final ColorScheme c = ColorScheme.of(context);

    // Chaque entrée est un triplet : nom, fond, couleur de contenu.
    final List<(String, Color, Color)> bandes = <(String, Color, Color)>[
      ('primary', c.primary, c.onPrimary),
      ('primaryContainer', c.primaryContainer, c.onPrimaryContainer),
      ('secondary', c.secondary, c.onSecondary),
      ('secondaryContainer', c.secondaryContainer, c.onSecondaryContainer),
      ('tertiary', c.tertiary, c.onTertiary),
      ('error', c.error, c.onError),
      ('surface', c.surface, c.onSurface),
      ('inverseSurface', c.inverseSurface, c.onInverseSurface),
    ];

    return Scaffold(
      appBar: AppBar(title: const Text('Rôles de couleur')),
      body: ListView.builder(
        itemCount: bandes.length,
        itemBuilder: (BuildContext context, int i) {
          final (String nom, Color fond, Color contenu) = bandes[i];
          return Container(
            height: 60,
            color: fond,
            alignment: Alignment.centerLeft,
            padding: const EdgeInsets.symmetric(horizontal: 20),
            child: Row(
              children: <Widget>[
                Icon(Icons.circle, size: 14, color: contenu),
                const SizedBox(width: 12),
                Expanded(
                  child: Text(
                    nom,
                    style: TextStyle(
                      color: contenu,
                      fontSize: 16,
                      fontWeight: FontWeight.w600,
                    ),
                  ),
                ),
              ],
            ),
          );
        },
      ),
    );
  }
}
```

**Explication :** on stocke les paires rôle / rôle « on » dans une liste de records, ce qui évite huit blocs de code presque identiques. `ColorScheme.of(context)` est le raccourci de `Theme.of(context).colorScheme`. Le point pédagogique est la garantie de contraste : chaque nom est écrit avec la couleur `onXxx` de sa bande, donc reste lisible, y compris pour `inverseSurface` dont le fond est presque noir en mode clair. Basculez le système en mode sombre : les huit bandes changent, mais toutes restent lisibles.

---

### Correction 3

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const Exo3());
}

class Exo3 extends StatelessWidget {
  const Exo3({super.key});

  @override
  Widget build(BuildContext context) {
    // 1. On construit le thème de base.
    final ThemeData base = ThemeData(
      colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF4E342E)),
    );

    // 2. On en dérive une version dont deux styles sont ajustés.
    final ThemeData theme = base.copyWith(
      textTheme: base.textTheme.copyWith(
        headlineMedium: base.textTheme.headlineMedium?.copyWith(
          fontWeight: FontWeight.w800,
          letterSpacing: -0.5,
        ),
        bodyLarge: base.textTheme.bodyLarge?.copyWith(height: 1.6),
      ),
    );

    return MaterialApp(
      title: 'Exercice 3',
      theme: theme,
      home: const PageTypo(),
    );
  }
}

class PageTypo extends StatelessWidget {
  const PageTypo({super.key});

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final TextTheme t = theme.textTheme;

    return Scaffold(
      appBar: AppBar(title: const Text('Typographie')),
      body: ListView(
        padding: const EdgeInsets.all(20),
        children: <Widget>[
          Text('La chute de Kaldar', style: t.headlineMedium),
          const SizedBox(height: 16),
          Text(
            'La cité de Kaldar régnait sur les trois vallées depuis quatre '
            'siècles. Ses forges ne s\'éteignaient jamais et ses murailles '
            'n\'avaient jamais cédé.',
            style: t.bodyLarge,
          ),
          const SizedBox(height: 16),
          Text(
            'Puis vint l\'hiver sans fin. Les fleuves gelèrent, les mines '
            's\'effondrèrent, et les forges se turent une à une.',
            style: t.bodyLarge,
          ),
          const SizedBox(height: 24),
          // Ajustement LOCAL : on part du style du thème, on ne le remplace pas.
          Text(
            'Aucun survivant n\'a jamais été retrouvé.',
            style: t.bodyLarge?.copyWith(
              color: theme.colorScheme.error,
              fontWeight: FontWeight.w600,
            ),
          ),
        ],
      ),
    );
  }
}
```

**Explication :** trois niveaux d'intervention sont visibles. `base` est le thème brut. `theme` en est une copie où **deux** styles seulement sont modifiés, en partant à chaque fois du style existant avec `copyWith` : l'interligne 1.6 de `bodyLarge` s'ajoute à sa taille de 16 px et à sa couleur, au lieu de les effacer. Enfin, le dernier `Text` fait un ajustement purement local, sans toucher au thème : il hérite de l'interligne 1.6 défini globalement et n'ajoute que la couleur et la graisse. C'est la démarche à suivre systématiquement : global pour ce qui se répète, local pour l'exception.

---

### Correction 4

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const Exo4());
}

const Color graine = Color(0xFF1565C0);

ThemeData faire(Brightness b) => ThemeData(
      colorScheme: ColorScheme.fromSeed(seedColor: graine, brightness: b),
      cardTheme: CardThemeData(
        elevation: 0,
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
      ),
    );

class Exo4 extends StatefulWidget {
  const Exo4({super.key});

  @override
  State<Exo4> createState() => _Exo4State();
}

class _Exo4State extends State<Exo4> {
  ThemeMode _mode = ThemeMode.system;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Exercice 4',
      theme: faire(Brightness.light),
      darkTheme: faire(Brightness.dark),
      themeMode: _mode,
      home: Coquille(
        mode: _mode,
        onMode: (ThemeMode m) => setState(() => _mode = m),
      ),
    );
  }
}

class Coquille extends StatefulWidget {
  const Coquille({super.key, required this.mode, required this.onMode});

  final ThemeMode mode;
  final ValueChanged<ThemeMode> onMode;

  @override
  State<Coquille> createState() => _CoquilleState();
}

class _CoquilleState extends State<Coquille> {
  int _index = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(_index == 0 ? 'Accueil' : 'Réglages')),
      body: _index == 0
          ? const PageAccueil()
          : PageReglages(mode: widget.mode, onMode: widget.onMode),
      bottomNavigationBar: NavigationBar(
        selectedIndex: _index,
        onDestinationSelected: (int i) => setState(() => _index = i),
        destinations: const <NavigationDestination>[
          NavigationDestination(
            icon: Icon(Icons.home_outlined),
            selectedIcon: Icon(Icons.home),
            label: 'Accueil',
          ),
          NavigationDestination(
            icon: Icon(Icons.settings_outlined),
            selectedIcon: Icon(Icons.settings),
            label: 'Réglages',
          ),
        ],
      ),
    );
  }
}

class PageAccueil extends StatelessWidget {
  const PageAccueil({super.key});

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return ListView(
      padding: const EdgeInsets.all(16),
      children: <Widget>[
        Card(
          color: theme.colorScheme.primaryContainer,
          child: Padding(
            padding: const EdgeInsets.all(20),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: <Widget>[
                Text('Luminosité appliquée',
                    style: theme.textTheme.titleMedium?.copyWith(
                      color: theme.colorScheme.onPrimaryContainer,
                    )),
                const SizedBox(height: 4),
                Text(theme.brightness.name,
                    style: theme.textTheme.displaySmall?.copyWith(
                      color: theme.colorScheme.onPrimaryContainer,
                    )),
                const SizedBox(height: 8),
                Text(
                  'Réglage du système : '
                  '${MediaQuery.platformBrightnessOf(context).name}',
                  style: theme.textTheme.bodyMedium?.copyWith(
                    color: theme.colorScheme.onPrimaryContainer,
                  ),
                ),
              ],
            ),
          ),
        ),
        const SizedBox(height: 12),
        const Card(
          child: ListTile(
            leading: Icon(Icons.flag),
            title: Text('Quête en cours'),
            subtitle: Text('Retrouver la clé du donjon'),
          ),
        ),
      ],
    );
  }
}

class PageReglages extends StatelessWidget {
  const PageReglages({super.key, required this.mode, required this.onMode});

  final ThemeMode mode;
  final ValueChanged<ThemeMode> onMode;

  @override
  Widget build(BuildContext context) {
    return ListView(
      padding: const EdgeInsets.all(16),
      children: <Widget>[
        Text('Thème', style: Theme.of(context).textTheme.titleLarge),
        const SizedBox(height: 12),
        SegmentedButton<ThemeMode>(
          segments: const <ButtonSegment<ThemeMode>>[
            ButtonSegment<ThemeMode>(
              value: ThemeMode.light,
              icon: Icon(Icons.light_mode),
              label: Text('Clair'),
            ),
            ButtonSegment<ThemeMode>(
              value: ThemeMode.system,
              icon: Icon(Icons.brightness_auto),
              label: Text('Auto'),
            ),
            ButtonSegment<ThemeMode>(
              value: ThemeMode.dark,
              icon: Icon(Icons.dark_mode),
              label: Text('Sombre'),
            ),
          ],
          selected: <ThemeMode>{mode},
          onSelectionChanged: (Set<ThemeMode> selection) {
            onMode(selection.first);
          },
        ),
      ],
    );
  }
}
```

**Explication :** l'état `_mode` vit dans `_Exo4State`, c'est-à-dire **au-dessus** du `MaterialApp` : c'est indispensable, car c'est `MaterialApp` qui consomme `themeMode`. Le callback `onMode` descend jusqu'à `PageReglages`, qui le déclenche via `SegmentedButton`. `SegmentedButton` travaille avec un `Set` de valeurs sélectionnées ; en sélection simple, on prend `selection.first`. L'accueil affiche deux informations distinctes : `theme.brightness` (ce qui est réellement appliqué) et `MediaQuery.platformBrightnessOf` (ce que demande le système). Quand `_mode` vaut `light` alors que le système est en sombre, les deux diffèrent — c'est la preuve que la surcharge fonctionne.

---

### Correction 5

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const Exo5());
}

@immutable
class CouleursEtat extends ThemeExtension<CouleursEtat> {
  const CouleursEtat({
    required this.succes,
    required this.avertissement,
    required this.info,
    required this.surEtat,
  });

  final Color succes;
  final Color avertissement;
  final Color info;
  final Color surEtat;

  static const CouleursEtat clair = CouleursEtat(
    succes: Color(0xFF2E7D32),
    avertissement: Color(0xFFEF6C00),
    info: Color(0xFF0277BD),
    surEtat: Colors.white,
  );

  static const CouleursEtat sombre = CouleursEtat(
    succes: Color(0xFFA5D6A7),
    avertissement: Color(0xFFFFCC80),
    info: Color(0xFF81D4FA),
    surEtat: Colors.black,
  );

  @override
  CouleursEtat copyWith({
    Color? succes,
    Color? avertissement,
    Color? info,
    Color? surEtat,
  }) {
    return CouleursEtat(
      succes: succes ?? this.succes,
      avertissement: avertissement ?? this.avertissement,
      info: info ?? this.info,
      surEtat: surEtat ?? this.surEtat,
    );
  }

  @override
  CouleursEtat lerp(ThemeExtension<CouleursEtat>? other, double t) {
    if (other is! CouleursEtat) {
      return this;
    }
    return CouleursEtat(
      succes: Color.lerp(succes, other.succes, t)!,
      avertissement: Color.lerp(avertissement, other.avertissement, t)!,
      info: Color.lerp(info, other.info, t)!,
      surEtat: Color.lerp(surEtat, other.surEtat, t)!,
    );
  }
}

extension LectureEtats on BuildContext {
  CouleursEtat get etats => Theme.of(this).extension<CouleursEtat>()!;
}

ThemeData faire(Brightness b) => ThemeData(
      colorScheme: ColorScheme.fromSeed(
        seedColor: const Color(0xFF37474F),
        brightness: b,
      ),
      extensions: <ThemeExtension<dynamic>>[
        b == Brightness.light ? CouleursEtat.clair : CouleursEtat.sombre,
      ],
    );

class Exo5 extends StatefulWidget {
  const Exo5({super.key});

  @override
  State<Exo5> createState() => _Exo5State();
}

class _Exo5State extends State<Exo5> {
  bool _sombre = false;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Exercice 5',
      theme: faire(Brightness.light),
      darkTheme: faire(Brightness.dark),
      themeMode: _sombre ? ThemeMode.dark : ThemeMode.light,
      home: PageEtats(
        sombre: _sombre,
        onBascule: (bool v) => setState(() => _sombre = v),
      ),
    );
  }
}

class PageEtats extends StatelessWidget {
  const PageEtats({super.key, required this.sombre, required this.onBascule});

  final bool sombre;
  final ValueChanged<bool> onBascule;

  @override
  Widget build(BuildContext context) {
    final CouleursEtat e = context.etats;

    return Scaffold(
      appBar: AppBar(
        title: const Text('États'),
        actions: <Widget>[
          Switch(value: sombre, onChanged: onBascule),
          const SizedBox(width: 12),
        ],
      ),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: <Widget>[
          _Bandeau(
            couleur: e.succes,
            icone: Icons.check_circle,
            texte: 'Quête terminée : les rats de la cave.',
          ),
          const SizedBox(height: 12),
          _Bandeau(
            couleur: e.avertissement,
            icone: Icons.warning_amber,
            texte: 'Votre épée est endommagée à 80 %.',
          ),
          const SizedBox(height: 12),
          _Bandeau(
            couleur: e.info,
            icone: Icons.info,
            texte: 'Un marchand ambulant est arrivé au village.',
          ),
        ],
      ),
    );
  }
}

class _Bandeau extends StatelessWidget {
  const _Bandeau({
    required this.couleur,
    required this.icone,
    required this.texte,
  });

  final Color couleur;
  final IconData icone;
  final String texte;

  @override
  Widget build(BuildContext context) {
    final Color surEtat = context.etats.surEtat;
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: couleur,
        borderRadius: BorderRadius.circular(12),
      ),
      child: Row(
        children: <Widget>[
          Icon(icone, color: surEtat),
          const SizedBox(width: 12),
          Expanded(
            child: Text(texte, style: TextStyle(color: surEtat, fontSize: 15)),
          ),
        ],
      ),
    );
  }
}
```

**Explication :** `CouleursEtat` respecte le contrat de `ThemeExtension` : `copyWith` reconstruit une instance champ par champ, et `lerp` interpole chaque couleur avec `Color.lerp` afin que la transition clair → sombre soit animée plutôt que brutale. Les deux jeux de valeurs sont des constantes statiques ; la fonction `faire(Brightness)` choisit le bon et l'enregistre dans `ThemeData.extensions`. L'extension `LectureEtats on BuildContext` transforme l'appel verbeux `Theme.of(context).extension<CouleursEtat>()!` en un simple `context.etats`, ce qui rend `_Bandeau` très lisible. Notez que `_Bandeau` ne connaît aucune couleur : il reçoit la sienne en paramètre et lit `surEtat` dans le thème.

---

### Correction 6

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const Exo6());
}

class Exo6 extends StatelessWidget {
  const Exo6({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Exercice 6',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blueGrey),
      ),
      home: const PageBord(),
    );
  }
}

class PageBord extends StatelessWidget {
  const PageBord({super.key});

  @override
  Widget build(BuildContext context) {
    // Accesseurs ciblés : chaque ligne n'abonne le widget qu'à ce qu'elle lit.
    final Size taille = MediaQuery.sizeOf(context);
    final Orientation orientation = MediaQuery.orientationOf(context);
    final EdgeInsets padding = MediaQuery.paddingOf(context);
    final EdgeInsets insets = MediaQuery.viewInsetsOf(context);
    final Brightness luminosite = MediaQuery.platformBrightnessOf(context);
    final TextScaler scaler = MediaQuery.textScalerOf(context);

    final List<(String, String)> valeurs = <(String, String)>[
      ('Largeur', '${taille.width.toStringAsFixed(1)} px'),
      ('Hauteur', '${taille.height.toStringAsFixed(1)} px'),
      ('Orientation', orientation.name),
      ('padding.top', '${padding.top.toStringAsFixed(1)} px'),
      ('padding.bottom', '${padding.bottom.toStringAsFixed(1)} px'),
      ('viewInsets.bottom', '${insets.bottom.toStringAsFixed(1)} px'),
      ('Luminosité système', luminosite.name),
      ('Texte 16 px peint à', '${scaler.scale(16).toStringAsFixed(1)} px'),
    ];

    final ThemeData theme = Theme.of(context);

    return Scaffold(
      appBar: AppBar(title: const Text('Tableau de bord')),
      body: Column(
        children: <Widget>[
          Expanded(
            child: ListView.separated(
              itemCount: valeurs.length,
              separatorBuilder: (_, __) => const Divider(height: 1),
              itemBuilder: (BuildContext context, int i) {
                final (String cle, String valeur) = valeurs[i];
                return ListTile(
                  title: Text(cle),
                  trailing: Text(
                    valeur,
                    style: theme.textTheme.titleSmall?.copyWith(
                      color: theme.colorScheme.primary,
                    ),
                  ),
                );
              },
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(16),
            child: TextField(
              decoration: InputDecoration(
                labelText: 'Ouvrez le clavier',
                helperText: insets.bottom > 0
                    ? 'Clavier ouvert'
                    : 'Clavier fermé',
                border: const OutlineInputBorder(),
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

**Explication :** chaque information est obtenue par son accesseur ciblé, ce qui limite les reconstructions : l'ouverture du clavier ne modifie que `viewInsets`, donc seul l'abonnement à `viewInsetsOf` justifie le rebuild. La différence entre `padding.bottom` et `viewInsets.bottom` est visible en direct : quand le clavier s'ouvre, `viewInsets.bottom` grimpe à quelques centaines de pixels tandis que `padding.bottom` tombe à zéro, puisque la barre de gestes est désormais recouverte par le clavier. `scaler.scale(16)` montre la taille réellement peinte : elle reste à 16 tant que l'utilisateur n'a pas modifié le réglage de police du système.

---

### Correction 7

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const Exo7());
}

class Exo7 extends StatelessWidget {
  const Exo7({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Exercice 7',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF00695C)),
        cardTheme: CardThemeData(
          elevation: 0,
          margin: EdgeInsets.zero,
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(16),
          ),
        ),
      ),
      home: const PagePreuve(),
    );
  }
}

/// Carte qui décide de sa disposition d'après SES contraintes,
/// pas d'après la taille de la fenêtre.
class CarteHeros extends StatelessWidget {
  const CarteHeros({
    super.key,
    required this.nom,
    required this.classe,
    required this.initiales,
  });

  final String nom;
  final String classe;
  final String initiales;

  static const double seuil = 500;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    final Widget avatar = CircleAvatar(
      radius: 40,
      backgroundColor: theme.colorScheme.primaryContainer,
      child: Text(
        initiales,
        style: theme.textTheme.headlineSmall?.copyWith(
          color: theme.colorScheme.onPrimaryContainer,
        ),
      ),
    );

    final Widget infos = Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      mainAxisSize: MainAxisSize.min,
      children: <Widget>[
        Text(nom, style: theme.textTheme.titleLarge),
        const SizedBox(height: 4),
        Text(classe,
            style: theme.textTheme.bodyMedium?.copyWith(
              color: theme.colorScheme.onSurfaceVariant,
            )),
        const SizedBox(height: 12),
        Wrap(
          spacing: 8,
          runSpacing: 8,
          children: <Widget>[
            FilledButton(onPressed: () {}, child: const Text('Équiper')),
            OutlinedButton(onPressed: () {}, child: const Text('Soigner')),
          ],
        ),
      ],
    );

    return Card(
      child: LayoutBuilder(
        builder: (BuildContext context, BoxConstraints c) {
          final bool large = c.maxWidth >= seuil;
          return Padding(
            padding: const EdgeInsets.all(16),
            child: large
                ? Row(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: <Widget>[
                      avatar,
                      const SizedBox(width: 20),
                      Expanded(child: infos),
                    ],
                  )
                : Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    mainAxisSize: MainAxisSize.min,
                    children: <Widget>[
                      avatar,
                      const SizedBox(height: 16),
                      infos,
                    ],
                  ),
          );
        },
      ),
    );
  }
}

class PagePreuve extends StatelessWidget {
  const PagePreuve({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Preuve par deux panneaux')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: <Widget>[
            // Panneau ÉTROIT : la carte doit s'empiler
            const SizedBox(
              width: 300,
              child: CarteHeros(
                nom: 'Aldric',
                classe: 'Guerrier — niveau 12',
                initiales: 'AL',
              ),
            ),
            const SizedBox(width: 16),
            // Panneau LARGE : la carte doit se mettre en ligne
            const Expanded(
              child: CarteHeros(
                nom: 'Sélène',
                classe: 'Mage — niveau 9',
                initiales: 'SE',
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Explication :** la décision est prise dans un `LayoutBuilder` **interne** à `CarteHeros`, sur `c.maxWidth`. La page de preuve affiche deux instances côte à côte sur le même écran : celle de gauche reçoit 300 px et s'empile, celle de droite reçoit tout le reste et se met en ligne. Si `CarteHeros` avait utilisé `MediaQuery.sizeOf(context).width`, les deux instances auraient lu la même valeur — la largeur de la fenêtre — et adopté la même disposition, ce qui aurait fait déborder la carte de gauche. Notez aussi que `avatar` et `infos` ne sont construits qu'une fois et simplement réagencés : aucune duplication de contenu entre les deux branches.

---

### Correction 8

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const Exo8());
}

class Exo8 extends StatelessWidget {
  const Exo8({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Exercice 8',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFFAD1457)),
        cardTheme: CardThemeData(
          elevation: 0,
          margin: EdgeInsets.zero,
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(14),
          ),
        ),
      ),
      home: const PageGrille(),
    );
  }
}

class PageGrille extends StatelessWidget {
  const PageGrille({super.key});

  static const double extentMax = 180;
  static const double espacement = 12;
  static const double marge = 16;

  /// Reproduit le calcul de SliverGridDelegateWithMaxCrossAxisExtent :
  /// nombre de colonnes = ceil(largeur / (extentMax + espacement)).
  int _colonnes(double largeurDisponible) {
    final int n = (largeurDisponible / (extentMax + espacement)).ceil();
    return n < 1 ? 1 : n;
  }

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return Scaffold(
      appBar: AppBar(title: const Text('Grille adaptative')),
      body: LayoutBuilder(
        builder: (BuildContext context, BoxConstraints c) {
          final double utile = c.maxWidth - marge * 2;
          final int colonnes = _colonnes(utile);

          return Column(
            children: <Widget>[
              Padding(
                padding: const EdgeInsets.all(marge),
                child: Card(
                  color: theme.colorScheme.primaryContainer,
                  child: Padding(
                    padding: const EdgeInsets.all(16),
                    child: Text(
                      'Largeur utile : ${utile.toStringAsFixed(0)} px  →  '
                      '$colonnes colonne${colonnes > 1 ? 's' : ''}',
                      style: theme.textTheme.titleMedium?.copyWith(
                        color: theme.colorScheme.onPrimaryContainer,
                      ),
                    ),
                  ),
                ),
              ),
              Expanded(
                child: GridView.builder(
                  padding: const EdgeInsets.symmetric(horizontal: marge)
                      .copyWith(bottom: marge),
                  gridDelegate:
                      const SliverGridDelegateWithMaxCrossAxisExtent(
                    maxCrossAxisExtent: extentMax,
                    crossAxisSpacing: espacement,
                    mainAxisSpacing: espacement,
                    childAspectRatio: 0.75,
                  ),
                  itemCount: 24,
                  itemBuilder: (BuildContext context, int i) {
                    return Card(
                      clipBehavior: Clip.antiAlias,
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.stretch,
                        children: <Widget>[
                          Expanded(
                            child: Container(
                              color: theme.colorScheme.secondaryContainer,
                              child: Icon(
                                Icons.inventory_2,
                                color: theme.colorScheme.onSecondaryContainer,
                              ),
                            ),
                          ),
                          Padding(
                            padding: const EdgeInsets.all(8),
                            child: Text(
                              'Objet ${i + 1}',
                              style: theme.textTheme.labelLarge,
                              maxLines: 1,
                              overflow: TextOverflow.ellipsis,
                            ),
                          ),
                        ],
                      ),
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

**Explication :** `SliverGridDelegateWithMaxCrossAxisExtent` garantit qu'aucune tuile ne dépassera 180 px de large ; Flutter choisit lui-même le nombre de colonnes, qui augmente en continu quand on élargit la fenêtre. Il n'y a donc **aucun point de rupture** à écrire ni à tester. La méthode `_colonnes` reproduit la formule interne du délégué (arrondi supérieur de la largeur divisée par l'extent augmenté de l'espacement), ce qui permet d'afficher le résultat. Le `childAspectRatio: 0.75` fixe le rapport largeur/hauteur, donc la hauteur de chaque tuile suit sa largeur : les tuiles restent proportionnées à toutes les tailles.

---

### Correction 9

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const Exo9());
}

abstract final class Rupture {
  static const double rail = 600;
  static const double railEtendu = 1200;
  static const double contenu = 800;
}

class Exo9 extends StatelessWidget {
  const Exo9({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Exercice 9',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF283593)),
      ),
      darkTheme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: const Color(0xFF283593),
          brightness: Brightness.dark,
        ),
      ),
      home: const Coquille(),
    );
  }
}

class Coquille extends StatefulWidget {
  const Coquille({super.key});

  @override
  State<Coquille> createState() => _CoquilleState();
}

class _CoquilleState extends State<Coquille> {
  // L'index vit dans le State : il survit au redimensionnement,
  // car la bascule NavigationBar / NavigationRail ne recrée pas ce State.
  int _index = 0;

  static const List<(IconData, IconData, String)> entrees =
      <(IconData, IconData, String)>[
    (Icons.flag_outlined, Icons.flag, 'Quêtes'),
    (Icons.backpack_outlined, Icons.backpack, 'Sac'),
    (Icons.person_outline, Icons.person, 'Héros'),
  ];

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (BuildContext context, BoxConstraints c) {
        final double largeur = c.maxWidth;
        final bool avecRail = largeur >= Rupture.rail;
        final bool railEtendu = largeur >= Rupture.railEtendu;

        final Widget page = PageContenu(
          titre: entrees[_index].$3,
          index: _index,
          largeur: largeur,
        );

        if (!avecRail) {
          return Scaffold(
            appBar: AppBar(title: Text(entrees[_index].$3)),
            body: page,
            bottomNavigationBar: NavigationBar(
              selectedIndex: _index,
              onDestinationSelected: (int i) => setState(() => _index = i),
              destinations: entrees
                  .map((e) => NavigationDestination(
                        icon: Icon(e.$1),
                        selectedIcon: Icon(e.$2),
                        label: e.$3,
                      ))
                  .toList(),
            ),
          );
        }

        return Scaffold(
          appBar: AppBar(title: Text(entrees[_index].$3)),
          body: Row(
            children: <Widget>[
              NavigationRail(
                selectedIndex: _index,
                onDestinationSelected: (int i) => setState(() => _index = i),
                extended: railEtendu,
                // extended et labelType sont incompatibles :
                // on neutralise labelType quand le rail est étendu.
                labelType: railEtendu
                    ? NavigationRailLabelType.none
                    : NavigationRailLabelType.all,
                groupAlignment: -0.9,
                destinations: entrees
                    .map((e) => NavigationRailDestination(
                          icon: Icon(e.$1),
                          selectedIcon: Icon(e.$2),
                          label: Text(e.$3),
                        ))
                    .toList(),
              ),
              const VerticalDivider(width: 1, thickness: 1),
              Expanded(child: page),
            ],
          ),
        );
      },
    );
  }
}

class PageContenu extends StatelessWidget {
  const PageContenu({
    super.key,
    required this.titre,
    required this.index,
    required this.largeur,
  });

  final String titre;
  final int index;
  final double largeur;

  String get _disposition {
    if (largeur >= Rupture.railEtendu) {
      return 'rail étendu';
    }
    if (largeur >= Rupture.rail) {
      return 'rail étroit';
    }
    return 'barre en bas';
  }

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final double basSysteme = MediaQuery.paddingOf(context).bottom;

    return Center(
      child: ConstrainedBox(
        constraints: const BoxConstraints(maxWidth: Rupture.contenu),
        child: ListView(
          padding: EdgeInsets.fromLTRB(16, 16, 16, 16 + basSysteme),
          children: <Widget>[
            Card(
              color: theme.colorScheme.primaryContainer,
              child: Padding(
                padding: const EdgeInsets.all(20),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: <Widget>[
                    Text(titre,
                        style: theme.textTheme.headlineSmall?.copyWith(
                          color: theme.colorScheme.onPrimaryContainer,
                        )),
                    const SizedBox(height: 8),
                    Text(
                      'Largeur : ${largeur.toStringAsFixed(0)} px\n'
                      'Disposition : $_disposition\n'
                      'Marge système en bas : '
                      '${basSysteme.toStringAsFixed(0)} px',
                      style: theme.textTheme.bodyMedium?.copyWith(
                        color: theme.colorScheme.onPrimaryContainer,
                      ),
                    ),
                  ],
                ),
              ),
            ),
            const SizedBox(height: 16),
            for (int i = 0; i < 8; i++)
              Padding(
                padding: const EdgeInsets.only(bottom: 8),
                child: Card(
                  child: ListTile(
                    leading: CircleAvatar(child: Text('${i + 1}')),
                    title: Text('$titre — élément ${i + 1}'),
                    subtitle: const Text('Contenu borné à 800 px de large'),
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

**Explication :** `_index` est stocké dans `_CoquilleState`, qui se trouve **au-dessus** du `LayoutBuilder`. Quand la largeur franchit 600 px, seul l'arbre construit par le `builder` change ; le `State` de `Coquille`, lui, est conservé, donc l'onglet sélectionné ne se perd pas. Le point technique à ne pas manquer est l'incompatibilité entre `extended: true` et un `labelType` autre que `none` : on le règle par une expression conditionnelle. Enfin, `PageContenu` borne son contenu à 800 px avec `Center` + `ConstrainedBox` et ajoute `MediaQuery.paddingOf(context).bottom` à son `padding` bas, pour que le dernier élément reste atteignable au-dessus de la barre de gestes.

---

### Correction 10

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const Exo10());
}

class Exo10 extends StatefulWidget {
  const Exo10({super.key});

  @override
  State<Exo10> createState() => _Exo10State();
}

class _Exo10State extends State<Exo10> {
  double _facteur = 1.0;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Exercice 10',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF00695C)),
        cardTheme: CardThemeData(
          elevation: 0,
          margin: EdgeInsets.zero,
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(16),
          ),
        ),
      ),
      // On injecte un textScaler simulé au-dessus de toutes les pages.
      builder: (BuildContext context, Widget? child) {
        return MediaQuery(
          data: MediaQuery.of(context)
              .copyWith(textScaler: TextScaler.linear(_facteur)),
          child: child!,
        );
      },
      home: PageResistante(
        facteur: _facteur,
        onFacteur: (double v) => setState(() => _facteur = v),
      ),
    );
  }
}

class PageResistante extends StatelessWidget {
  const PageResistante({
    super.key,
    required this.facteur,
    required this.onFacteur,
  });

  final double facteur;
  final ValueChanged<double> onFacteur;

  static const List<(String, String)> stats = <(String, String)>[
    ('Points de vie', '128 / 160'),
    ('Points de mana', '42 / 90'),
    ('Endurance', '77 / 100'),
  ];

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return Scaffold(
      appBar: AppBar(title: const Text('Fiche résistante')),
      // ListView : la page défile, donc aucun débordement vertical possible.
      body: ListView(
        padding: EdgeInsets.fromLTRB(
          16,
          16,
          16,
          16 + MediaQuery.paddingOf(context).bottom,
        ),
        children: <Widget>[
          // Curseur de simulation
          Text('textScaler simulé : ${facteur.toStringAsFixed(2)}',
              style: theme.textTheme.labelLarge),
          Slider(
            value: facteur,
            min: 0.8,
            max: 2.5,
            divisions: 17,
            label: facteur.toStringAsFixed(2),
            onChanged: onFacteur,
          ),
          const Divider(height: 32),

          // 1. En-tête : titre extensible + badge à grossissement borné.
          //    Wrap plutôt que Row : si les deux ne tiennent pas, on passe
          //    à la ligne au lieu de déborder.
          Wrap(
            crossAxisAlignment: WrapCrossAlignment.center,
            spacing: 12,
            runSpacing: 8,
            children: <Widget>[
              Text('Aldric le Vaillant',
                  style: theme.textTheme.headlineSmall),
              MediaQuery.withClampedTextScaling(
                maxScaleFactor: 1.3,
                child: Container(
                  padding:
                      const EdgeInsets.symmetric(horizontal: 12, vertical: 6),
                  decoration: BoxDecoration(
                    color: theme.colorScheme.tertiaryContainer,
                    borderRadius: BorderRadius.circular(999),
                  ),
                  child: Text(
                    'NIVEAU 12',
                    style: theme.textTheme.labelMedium?.copyWith(
                      color: theme.colorScheme.onTertiaryContainer,
                    ),
                  ),
                ),
              ),
            ],
          ),
          const SizedBox(height: 20),

          // 2. Icône + texte long : Expanded pour autoriser le retour à la ligne.
          //    crossAxisAlignment.start pour que l'icône reste en haut.
          Row(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: <Widget>[
              Icon(Icons.info_outline, color: theme.colorScheme.primary),
              const SizedBox(width: 12),
              Expanded(
                child: Text(
                  'Ce personnage a survécu à trois sièges, deux naufrages '
                  'et une invasion de gobelins particulièrement mal '
                  'organisée.',
                  style: theme.textTheme.bodyMedium,
                ),
              ),
            ],
          ),
          const SizedBox(height: 20),

          // 3. Lignes libellé / valeur : pas de hauteur fixe, deux Expanded
          //    pour partager la largeur au lieu de la disputer.
          Card(
            child: Column(
              children: <Widget>[
                for (final (String libelle, String valeur) in stats)
                  Padding(
                    padding: const EdgeInsets.symmetric(
                        horizontal: 16, vertical: 12),
                    child: Row(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: <Widget>[
                        Expanded(
                          flex: 3,
                          child: Text(libelle,
                              style: theme.textTheme.bodyMedium),
                        ),
                        const SizedBox(width: 12),
                        Expanded(
                          flex: 2,
                          child: Text(
                            valeur,
                            textAlign: TextAlign.end,
                            style: theme.textTheme.titleSmall?.copyWith(
                              color: theme.colorScheme.primary,
                            ),
                          ),
                        ),
                      ],
                    ),
                  ),
              ],
            ),
          ),
          const SizedBox(height: 24),

          // 4. Pied de page : Wrap, donc les boutons passent en colonne
          //    dès qu'ils ne tiennent plus côte à côte.
          Wrap(
            spacing: 12,
            runSpacing: 12,
            children: <Widget>[
              FilledButton.icon(
                onPressed: () {},
                icon: const Icon(Icons.save),
                label: const Text('Sauvegarder'),
              ),
              OutlinedButton.icon(
                onPressed: () {},
                icon: const Icon(Icons.restart_alt),
                label: const Text('Réinitialiser'),
              ),
            ],
          ),
        ],
      ),
    );
  }
}
```

**Explication :** cinq techniques sont combinées pour rendre la page indestructible. Le `ListView` supprime tout risque de débordement vertical. Les deux `Wrap` (en-tête et pied de page) remplacent des `Row` : lorsque les enfants ne tiennent plus côte à côte, ils passent à la ligne au lieu de provoquer un `RenderFlex overflowed`. Le `Row` de l'icône enveloppe son texte dans un `Expanded`, ce qui autorise le retour à la ligne. Les lignes de statistiques n'ont **aucune hauteur fixe** et partagent la largeur avec deux `Expanded` pondérés 3 et 2. Enfin, `MediaQuery.withClampedTextScaling(maxScaleFactor: 1.3)` isole le seul élément dont la taille est critique — le badge en gélule — sans toucher au reste de la page. Poussez le curseur à 2,5 : la page s'allonge, le texte passe à la ligne, mais rien n'est jamais coupé.

---

## Et maintenant ?

Votre application a maintenant une identité visuelle cohérente, un mode sombre, et une mise en page qui tient de l'iPhone SE à l'écran de bureau.

Mais vous avez rencontré deux fois la même gêne dans ce chapitre : en 51.12, puis en 51.33, il a fallu **faire descendre un callback à la main** à travers plusieurs constructeurs pour qu'une page enfouie puisse changer le thème. Plus l'application grandit, plus ce câblage devient pénible et fragile.

C'est le problème de la **gestion d'état**. Le chapitre suivant montre pourquoi `setState` seul ne suffit plus, ce qu'est un `InheritedWidget`, comment `ChangeNotifier` et le paquet `provider` résolvent proprement le partage d'état, et comment choisir parmi les solutions existantes.

Chapitre suivant : [52-PARTIE-1B—GESTION-DÉTAT-SETSTATE-PROVIDER.md](./52-PARTIE-1B—GESTION-DÉTAT-SETSTATE-PROVIDER.md)
