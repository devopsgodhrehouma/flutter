# PARTIE 1C — MINI-PROJETS FLUTTER
# CHAPITRE 61 — PROJET 7 : L'APPLICATION MÉTÉO

> **Niveau :** intermédiaire / avancé
> **Durée estimée :** 14 h
> **Pré-requis :** PARTIE 1A (chapitres 01 à 18), PARTIE 1B (chapitres 43 à 54), et les projets 55 à 60
> **Ce que vous saurez faire à la fin :** brancher une application Flutter sur une vraie API publique, en modéliser la réponse JSON, gérer proprement les trois états d'un écran de données, survivre à la perte du réseau grâce à un cache local, et livrer une interface qui change d'apparence selon la météo et selon l'heure.

---

## 61.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- lire la documentation d'une API réelle et en construire l'URL exacte ;
- utiliser une API **sans clé d'inscription**, donc exécutable immédiatement ;
- transformer une saisie de l'utilisateur en coordonnées géographiques via une API de géocodage ;
- modéliser une réponse JSON complexe en quatre classes Dart (rappel du chapitre 17) ;
- décoder une structure JSON « en colonnes », très différente d'une simple liste d'objets ;
- écrire une couche service isolée du reste de l'application (rappel du chapitre 53) ;
- injecter un `http.Client` pour rendre cette couche testable ;
- déclarer une exception métier avec un `enum` de cause (rappel du chapitre 13) ;
- distinguer une panne réseau, un dépassement de délai, une erreur serveur et une donnée corrompue ;
- déclarer la permission Internet sur Android et sur macOS ;
- afficher les trois états d'un écran de données avec `FutureBuilder` ;
- traduire un code météo WMO en libellé français et en icône ;
- peindre un fond en dégradé qui dépend de la condition et du moment de la journée ;
- construire une bande de prévisions horaires avec une `ListView` horizontale (rappel du chapitre 48) ;
- construire une liste de prévisions sur sept jours avec des barres de températures ;
- formater des dates en français avec le paquet `intl` ;
- écrire une barre de recherche avec **débounce**, pour ne pas interroger l'API à chaque frappe ;
- mémoriser un historique de recherche et une liste de villes favorites ;
- persister ces favoris avec `shared_preferences` (rappel du chapitre 54) ;
- mettre en cache la dernière réponse pour afficher quelque chose hors-ligne ;
- rafraîchir par tirage vers le bas avec `RefreshIndicator` ;
- basculer entre degrés Celsius et degrés Fahrenheit ;
- centraliser tout cet état dans un `ChangeNotifier` exposé par `provider` (rappel du chapitre 52) ;
- tester la couche service avec une réponse JSON factice, sans jamais toucher au réseau.

---

## 61.0.1 — Aperçu du résultat final

Voici l'application terminée. Écran principal, données chargées :

```text
┌────────────────────────────────────────────────┐
│  ░░░ dégradé bleu clair → bleu profond ░░░     │
│                                                │
│   ⌕  Paris, Île-de-France          ♥   °C/°F   │
│                                                │
│                    ☀                           │
│                                                │
│                  26 °C                         │
│            Partiellement nuageux               │
│              Ressenti 27 °C                    │
│                                                │
│         ↑ 29 °C          ↓ 17 °C               │
│                                                │
│   Humidité 48 %   Vent 12 km/h   Pluie 0 mm    │
│                                                │
│   Mis à jour à 14:05                           │
├────────────────────────────────────────────────┤
│  PROCHAINES HEURES                             │
│  ┌────┐┌────┐┌────┐┌────┐┌────┐┌────┐┌────┐    │
│  │15 h││16 h││17 h││18 h││19 h││20 h││21 h│ →  │
│  │ ☀  ││ ☀  ││ ☁  ││ ☁  ││ ☂  ││ ☂  ││ ☁  │    │
│  │27° ││28° ││27° ││25° ││22° ││20° ││19° │    │
│  │ 0% ││ 0% ││10% ││20% ││60% ││55% ││20% │    │
│  └────┘└────┘└────┘└────┘└────┘└────┘└────┘    │
├────────────────────────────────────────────────┤
│  7 PROCHAINS JOURS                             │
│  Aujourd'hui   ☀   0%   17° ▓▓▓▓▓▓▓▓░░  29°    │
│  demain        ☁  20%   16° ▓▓▓▓▓▓░░░░  26°    │
│  lundi         ☂  70%   14° ▓▓▓▓░░░░░░  22°    │
│  mardi         ☂  80%   13° ▓▓▓░░░░░░░  20°    │
│  mercredi      ☁  30%   14° ▓▓▓▓▓░░░░░  23°    │
│  jeudi         ☀  10%   16° ▓▓▓▓▓▓▓░░░  27°    │
│  vendredi      ☀   0%   18° ▓▓▓▓▓▓▓▓▓░  30°    │
└────────────────────────────────────────────────┘
```

L'écran de recherche, avec ses favoris et son historique :

```text
┌────────────────────────────────────────────────┐
│  ←  ┌──────────────────────────────────────┐   │
│     │ ⌕  bord                          ✕   │   │
│     └──────────────────────────────────────┘   │
├────────────────────────────────────────────────┤
│  Bordeaux                                      │
│  Nouvelle-Aquitaine · France                   │
├────────────────────────────────────────────────┤
│  Bordeaux                                      │
│  Ohio · États-Unis                             │
├────────────────────────────────────────────────┤
│  Bordeauxville                                 │
│  Bruxelles · Belgique                          │
└────────────────────────────────────────────────┘

Champ vide → on affiche plutôt ceci :

┌────────────────────────────────────────────────┐
│  FAVORIS                                       │
│  ♥  Paris, France                          ✕   │
│  ♥  Reykjavík, Islande                     ✕   │
│                                                │
│  RECHERCHES RÉCENTES                    Effacer│
│  ⟲  Lyon, France                               │
│  ⟲  Tokyo, Japon                               │
└────────────────────────────────────────────────┘
```

Les trois états d'un écran de données, que ce projet met en scène en grand :

```text
CHARGEMENT                ERREUR                    SUCCÈS
┌──────────────┐          ┌──────────────┐          ┌──────────────┐
│              │          │      ⚠       │          │      ☀       │
│      ◌       │          │              │          │    26 °C     │
│  Chargement  │          │  Pas de      │          │  Partiel...  │
│  de la météo │          │  connexion   │          │              │
│              │          │              │          │  ...         │
│              │          │  [RÉESSAYER] │          │              │
└──────────────┘          └──────────────┘          └──────────────┘
```

Et le mode hors-ligne, où l'on affiche des données périmées plutôt qu'un écran vide :

```text
┌────────────────────────────────────────────────┐
│  ⚠  Hors-ligne — données du 15 août à 14:05    │
├────────────────────────────────────────────────┤
│                    ☀                           │
│                  26 °C                         │
│            Partiellement nuageux               │
└────────────────────────────────────────────────┘
```

---

## 61.0.2 — Cahier des charges

### Fonctionnalités obligatoires

| # | Exigence | Vérification |
| --- | --- | --- |
| O1 | L'application fonctionne **sans aucune clé d'API** et sans compte. | Un `flutter run` après `flutter pub get` suffit. |
| O2 | L'utilisateur cherche une ville par son nom et choisit dans une liste de résultats. | Taper « bord » propose Bordeaux. |
| O3 | La réponse de l'API est décodée dans des classes Dart typées. | Aucun `dynamic` ne sort de la couche service. |
| O4 | L'écran affiche la météo actuelle : température, ressenti, condition, humidité, vent, précipitations. | Comparer avec le site open-meteo.com. |
| O5 | L'écran affiche les 24 prochaines heures dans une liste horizontale. | Faire défiler latéralement. |
| O6 | L'écran affiche 7 jours avec minimum, maximum et probabilité de pluie. | Sept lignes. |
| O7 | Le code météo numérique est traduit en français et en icône. | Le code 61 affiche « Pluie faible ». |
| O8 | Le fond change selon la condition et selon qu'il fait jour ou nuit. | Comparer Reykjavík et Sydney. |
| O9 | Les trois états chargement / erreur / succès sont tous traités. | Couper le Wi-Fi. |
| O10 | Une erreur affiche un message compréhensible et un bouton « Réessayer ». | Message en français, pas de trace technique. |
| O11 | La recherche est débouncée : au plus une requête par pause de saisie. | Taper « bordeaux » ne déclenche pas 8 requêtes. |
| O12 | Les dates et les heures sont en français. | « dimanche 16 août », « 14:05 ». |
| O13 | L'utilisateur peut mettre des villes en favori ; les favoris survivent au redémarrage. | Tuer l'application, la relancer. |
| O14 | La dernière réponse est mise en cache et réaffichée hors-ligne, avec un bandeau d'avertissement. | Mode avion. |
| O15 | Un tirage vers le bas recharge les données. | `RefreshIndicator`. |
| O16 | L'unité de température est commutable entre °C et °F, et le choix est persisté. | La bascule change toutes les valeurs. |
| O17 | Tout l'état applicatif est centralisé dans un `ChangeNotifier` fourni par `provider`. | Aucun `setState` de donnée métier. |
| O18 | La couche service est couverte par des tests utilisant un JSON factice. | `flutter test` passe sans réseau. |

### Fonctionnalités bonus

| # | Exigence |
| --- | --- |
| B1 | Un indicateur de qualité : afficher l'heure exacte de la donnée et son âge. |
| B2 | Le lever et le coucher du soleil affichés dans la carte du jour. |
| B3 | Un mode sombre automatique la nuit. |
| B4 | Le vent affiché avec une flèche orientée selon `wind_direction_10m`. |
| B5 | Plusieurs villes consultables par onglets glissants. |

---

## 61.0.3 — Notions mobilisées

Aucune notion nouvelle dans ce projet. Tout vient des chapitres précédents. Si une ligne du tableau vous surprend, relisez le chapitre indiqué **avant** de commencer.

| Notion | Chapitre | Usage exact dans ce projet |
| --- | --- | --- |
| `List`, `Map`, indexation | 06 | Les colonnes parallèles de la réponse JSON. |
| Fonctions, paramètres nommés | 07 | Les constructeurs des modèles. |
| Classes et méthodes | 08 | `Ville`, `MeteoActuelle`, `PrevisionJour`, `PrevisionHeure`. |
| Constructeurs nommés, `required` | 09 | `Ville.fromJson`, `BulletinMeteo.fromJson`. |
| `enum` enrichi | 11 | `CauseErreur`, `UniteTemperature`, `FamilleMeteo`. |
| Null safety, `?`, `??` | 12 | Les champs facultatifs de la réponse (`precipitation_probability` peut être nul). |
| Exceptions, `try`/`catch`/`on` | 13 | `ErreurMeteo`, `SocketException`, `TimeoutException`. |
| `map`, `where`, `reduce`, `generate` | 14 | Reconstruire des objets à partir des colonnes. |
| `Future`, `async`/`await`, `timeout` | 15 | Toute la couche réseau. |
| `pubspec.yaml`, paquets | 16 | `http`, `provider`, `shared_preferences`, `intl`. |
| `jsonDecode`, `fromJson` | 17 | La désérialisation complète. |
| `MaterialApp`, `Scaffold` | 44 | La structure des écrans. |
| `StatefulWidget`, `initState`, `dispose` | 45 | Le champ de recherche et son minuteur de débounce. |
| `Column`, `Row`, `Expanded`, `Stack` | 46 | La mise en page de la carte et du fond. |
| `Text`, `TextStyle`, `Icon` | 47 | Les températures et les icônes météo. |
| `ListView` horizontale, `ListView.builder` | 48 | La bande horaire et la liste des 7 jours. |
| `TextField`, `TextEditingController` | 49 | La barre de recherche. |
| `Navigator.push` et retour de données | 50 | L'écran de recherche renvoie une `Ville`. |
| `ThemeData`, dégradés, `MediaQuery` | 51 | Le fond dégradé, l'espace sous la barre d'état. |
| `ChangeNotifier`, `provider` | 52 | `EtatMeteo`. |
| `http`, `FutureBuilder`, `RefreshIndicator` | 53 | La couche service et l'écran principal. |
| `shared_preferences`, cache hors-ligne | 54 | Favoris, historique, unité, dernière réponse. |

---

## 61.0.4 — Arborescence du projet

Voici l'arborescence finale. Elle est donnée dès maintenant pour que vous sachiez où va chaque fichier ; nous la construirons progressivement.

```text
appli_meteo/
├── pubspec.yaml
├── lib/
│   ├── main.dart                          point d'entrée, thème, provider
│   ├── modeles/
│   │   ├── ville.dart                     résultat de l'API de géocodage
│   │   ├── meteo_actuelle.dart            le bloc "current"
│   │   ├── prevision_heure.dart           une case de la bande horaire
│   │   ├── prevision_jour.dart            une ligne des 7 jours
│   │   └── bulletin_meteo.dart            l'assemblage des trois précédents
│   ├── logique/
│   │   ├── codes_wmo.dart                 code → libellé français (pur, testable)
│   │   └── unites.dart                    enum UniteTemperature
│   ├── services/
│   │   ├── erreur_meteo.dart              exception métier + enum de cause
│   │   ├── geocodage_service.dart         recherche de villes
│   │   ├── meteo_service.dart             appel de l'API de prévision
│   │   ├── cache_meteo.dart               dernière réponse sur disque
│   │   └── depot_preferences.dart         favoris, historique, unité
│   ├── etat/
│   │   └── etat_meteo.dart                ChangeNotifier principal
│   ├── ecrans/
│   │   ├── ecran_meteo.dart               écran principal
│   │   └── ecran_recherche.dart           recherche, favoris, historique
│   ├── widgets/
│   │   ├── fond_meteo.dart                dégradé selon météo et heure
│   │   ├── icone_meteo.dart               code → IconData
│   │   ├── carte_actuelle.dart            la grande carte du haut
│   │   ├── bande_horaire.dart             ListView horizontale
│   │   ├── liste_jours.dart               les 7 jours
│   │   ├── bandeau_hors_ligne.dart        l'avertissement de données périmées
│   │   └── etats_ecran.dart               chargement et erreur
│   └── utilitaires/
│       ├── dates.dart                     formatage intl en français
│       └── temps.dart                     heure murale d'une ville
└── test/
    ├── codes_wmo_test.dart
    └── meteo_service_test.dart
```

**Pourquoi cette découpe ?** Elle sépare quatre responsabilités que le débutant a tendance à mélanger dans un seul fichier :

```text
modeles/      QUOI ?        des données typées, aucune dépendance à Flutter ni à http
logique/      QUELLE RÈGLE ? des fonctions pures, testables sans réseau ni écran
services/     D'OÙ ?        le réseau et le disque, seuls endroits qui peuvent échouer
etat/         QUAND ?       ce qui change et qui prévient l'interface
ecrans/       QUOI VOIR ?   les pages complètes
widgets/      AVEC QUOI ?   les morceaux d'interface réutilisables
```

Règle stricte, la même qu'au chapitre 58 : un fichier de `modeles/` ou de `logique/` n'importe **jamais** `package:flutter/material.dart`. C'est cette règle qui rend les tests de la section 61.23 possibles et rapides.

---

## 61.1 — L'API Open-Meteo, et pourquoi celle-ci

### Le problème des clés d'API

La plupart des services météo demandent une inscription, puis fournissent une clé à insérer dans chaque requête. Cela pose trois problèmes pour un projet d'apprentissage :

1. vous ne pouvez pas exécuter le code avant d'avoir créé un compte ;
2. la clé finit presque toujours écrite en dur dans le dépôt, ce que le chapitre 53 (section 53.32) vous a formellement déconseillé ;
3. les quotas gratuits expirent, et le chapitre devient injouable six mois plus tard.

**Open-Meteo** ne demande rien. Aucune inscription, aucune clé, aucun en-tête d'autorisation. L'usage non commercial est libre. C'est pour cette raison que nous l'utilisons ici : vous copiez le code, vous lancez, ça marche.

### Deux API distinctes

Le projet en utilise deux, servies par deux domaines différents :

```text
1. GÉOCODAGE   « Bordeaux »  →  latitude 44.84044, longitude -0.5805
   https://geocoding-api.open-meteo.com/v1/search

2. PRÉVISION   latitude/longitude  →  température, codes météo, 7 jours
   https://api.open-meteo.com/v1/forecast
```

C'est un enchaînement classique, que vous retrouverez partout : l'utilisateur saisit du texte, une première API le convertit en identifiant technique, une seconde API renvoie les données. Le mot exact est **géocodage** : passer d'un nom de lieu à des coordonnées.

```text
      saisie utilisateur
             │
             ▼
   ┌───────────────────┐
   │ GeocodageService  │  →  List<Ville>
   └───────────────────┘
             │  l'utilisateur choisit une Ville
             ▼
   ┌───────────────────┐
   │   MeteoService    │  →  BulletinMeteo
   └───────────────────┘
             │
             ▼
        l'interface
```

### L'URL de géocodage

Testez-la tout de suite dans votre navigateur, avant d'écrire la moindre ligne de Dart :

```text
https://geocoding-api.open-meteo.com/v1/search?name=Bordeaux&count=5&language=fr&format=json
```

| Paramètre | Rôle |
| --- | --- |
| `name` | le texte cherché (nom de ville ou code postal) |
| `count` | nombre de résultats, 10 par défaut, 100 au maximum |
| `language` | langue des libellés renvoyés ; `fr` traduit les pays et les régions |
| `format` | `json` (par défaut) ou `protobuf` |
| `countryCode` | filtre facultatif sur un code pays ISO-3166-1 alpha-2, par exemple `FR` |

La réponse, réduite ici à un seul résultat, ressemble exactement à ceci :

```json
{
  "results": [
    {
      "id": 2988507,
      "name": "Paris",
      "latitude": 48.85341,
      "longitude": 2.3488,
      "elevation": 42.0,
      "feature_code": "PPLC",
      "country_code": "FR",
      "admin1_id": 3012874,
      "admin2_id": 2968815,
      "admin3_id": 2988506,
      "admin4_id": 6455259,
      "timezone": "Europe/Paris",
      "population": 2138551,
      "postcodes": ["75001", "75002", "75003"],
      "country_id": 3017382,
      "country": "France",
      "admin1": "Île-de-France",
      "admin2": "Département de Paris",
      "admin3": "Paris",
      "admin4": "Paris"
    }
  ],
  "generationtime_ms": 1.0780096
}
```

Trois remarques que vous devez retenir avant de coder :

1. **Quand aucune ville ne correspond, la clé `results` est tout simplement absente.** Il n'y a pas de tableau vide. Un `donnees['results']` renverra `null`, et un `as List` direct plantera. C'est le premier piège du projet.
2. Les champs `admin1` à `admin4` ne sont pas toujours présents. Bordeaux (Ohio) n'a pas d'`admin3`. Tout doit être lu en nullable.
3. `admin1` est la subdivision la plus large (région, État). C'est celle qui distingue le mieux deux villes homonymes : « Paris, Île-de-France » contre « Paris, Texas ».

### L'URL de prévision

Toujours dans le navigateur :

```text
https://api.open-meteo.com/v1/forecast
  ?latitude=48.85341
  &longitude=2.3488
  &current=temperature_2m,relative_humidity_2m,apparent_temperature,is_day,precipitation,weather_code,wind_speed_10m
  &hourly=temperature_2m,weather_code,precipitation_probability
  &daily=weather_code,temperature_2m_max,temperature_2m_min,sunrise,sunset,precipitation_probability_max
  &timezone=auto
  &forecast_days=7
  &temperature_unit=celsius
  &wind_speed_unit=kmh
```

(Les retours à la ligne sont là pour la lisibilité. Une URL n'en contient jamais.)

| Paramètre | Rôle |
| --- | --- |
| `latitude`, `longitude` | obligatoires, en degrés décimaux |
| `current` | liste des variables **instantanées** demandées |
| `hourly` | liste des variables **horaires** demandées |
| `daily` | liste des variables **journalières** demandées |
| `timezone` | `auto` demande à l'API de renvoyer les heures dans le fuseau du lieu |
| `forecast_days` | de 0 à 16, 7 par défaut |
| `past_days` | de 0 à 92, pour remonter dans le passé |
| `temperature_unit` | `celsius` (défaut) ou `fahrenheit` |
| `wind_speed_unit` | `kmh` (défaut), `ms`, `mph`, `kn` |
| `precipitation_unit` | `mm` (défaut) ou `inch` |
| `timeformat` | `iso8601` (défaut) ou `unixtime` |

Les noms de variables sont normalisés et **ne s'inventent pas**. Voici celles utilisées dans ce projet, et leur signification exacte :

| Variable | Bloc | Signification |
| --- | --- | --- |
| `temperature_2m` | current, hourly | température de l'air à 2 mètres du sol |
| `relative_humidity_2m` | current, hourly | humidité relative en pourcentage |
| `apparent_temperature` | current, hourly | température ressentie |
| `is_day` | current, hourly | 1 s'il fait jour, 0 s'il fait nuit |
| `precipitation` | current, hourly | précipitations cumulées de l'heure, en mm |
| `weather_code` | current, hourly, daily | code de condition WMO (voir 61.10) |
| `wind_speed_10m` | current, hourly | vent à 10 mètres |
| `precipitation_probability` | hourly | probabilité de précipitations en pourcentage |
| `temperature_2m_max` / `_min` | daily | extrêmes de la journée |
| `sunrise` / `sunset` | daily | lever et coucher du soleil, en iso8601 |
| `precipitation_probability_max` | daily | probabilité maximale de la journée |

### La forme de la réponse : des colonnes, pas des objets

C'est **le** point qui surprend tout le monde. Une API classique renverrait une liste d'objets :

```json
[
  { "heure": "14:00", "temperature": 26.4, "code": 2 },
  { "heure": "15:00", "temperature": 27.1, "code": 2 }
]
```

Open-Meteo renvoie au contraire des **colonnes parallèles** : un tableau par variable, tous de la même longueur, alignés par index.

```json
{
  "hourly": {
    "time": ["2026-08-15T14:00", "2026-08-15T15:00"],
    "temperature_2m": [26.4, 27.1],
    "weather_code": [2, 2]
  }
}
```

```text
index :        0                    1
              ─┴──────────────────  ─┴──────────────────
time        : "2026-08-15T14:00"    "2026-08-15T15:00"
temperature : 26.4                  27.1
weather_code: 2                     2
              ▼                     ▼
        PrevisionHeure          PrevisionHeure
```

Ce format est bien plus compact sur le réseau : le nom `"temperature_2m"` n'est écrit qu'une fois au lieu de 168 fois. En échange, **c'est à vous de recomposer les objets**, en parcourant les index. Vous le ferez en une ligne avec `List.generate` à la section 61.7.

Voici la réponse complète, tronquée à trois heures et deux jours pour tenir sur la page. Étudiez-la : c'est la carte du territoire que vous allez modéliser.

```json
{
  "latitude": 48.86,
  "longitude": 2.3399997,
  "generationtime_ms": 0.3020763397216797,
  "utc_offset_seconds": 7200,
  "timezone": "Europe/Paris",
  "timezone_abbreviation": "GMT+2",
  "elevation": 43.0,
  "current_units": {
    "time": "iso8601",
    "interval": "seconds",
    "temperature_2m": "°C",
    "relative_humidity_2m": "%",
    "apparent_temperature": "°C",
    "is_day": "",
    "precipitation": "mm",
    "weather_code": "wmo code",
    "wind_speed_10m": "km/h"
  },
  "current": {
    "time": "2026-08-15T14:00",
    "interval": 900,
    "temperature_2m": 26.4,
    "relative_humidity_2m": 48,
    "apparent_temperature": 26.9,
    "is_day": 1,
    "precipitation": 0.0,
    "weather_code": 2,
    "wind_speed_10m": 11.9
  },
  "hourly_units": {
    "time": "iso8601",
    "temperature_2m": "°C",
    "weather_code": "wmo code",
    "precipitation_probability": "%"
  },
  "hourly": {
    "time": ["2026-08-15T00:00", "2026-08-15T01:00", "2026-08-15T02:00"],
    "temperature_2m": [18.2, 17.8, 17.5],
    "weather_code": [1, 1, 2],
    "precipitation_probability": [0, 0, 3]
  },
  "daily_units": {
    "time": "iso8601",
    "weather_code": "wmo code",
    "temperature_2m_max": "°C",
    "temperature_2m_min": "°C",
    "sunrise": "iso8601",
    "sunset": "iso8601",
    "precipitation_probability_max": "%"
  },
  "daily": {
    "time": ["2026-08-15", "2026-08-16"],
    "weather_code": [2, 61],
    "temperature_2m_max": [29.1, 26.3],
    "temperature_2m_min": [17.4, 16.2],
    "sunrise": ["2026-08-15T06:47", "2026-08-16T06:48"],
    "sunset": ["2026-08-15T21:03", "2026-08-16T21:01"],
    "precipitation_probability_max": [0, 70]
  }
}
```

Notez `utc_offset_seconds: 7200`. Avec `timezone=auto`, toutes les heures renvoyées sont **les heures locales du lieu**, sans indicateur de fuseau à la fin de la chaîne. `"2026-08-15T14:00"` signifie « 14 h à Paris », pas « 14 h chez vous ». Nous traiterons ce point délicat à la section 61.6.

---

## 61.2 — Créer le projet, les dépendances, les permissions

### Créer le projet

```text
flutter create appli_meteo
cd appli_meteo
```

### Ajouter les dépendances

Comme dans tous les chapitres de cette formation, on ajoute les paquets par la commande, jamais en écrivant un numéro de version à la main. La commande résout la version compatible avec votre SDK et met à jour `pubspec.yaml` toute seule.

```text
flutter pub add http
flutter pub add provider
flutter pub add shared_preferences
flutter pub add intl
```

| Paquet | Rôle dans ce projet | Chapitre de référence |
| --- | --- | --- |
| `http` | les deux appels réseau | 53 |
| `provider` | la diffusion de l'état à l'arbre de widgets | 52 |
| `shared_preferences` | favoris, historique, unité, cache | 54 |
| `intl` | dates et heures en français | 58 (section 58.10) |

Votre `pubspec.yaml` doit ressembler à ceci (les numéros de version chez vous seront ceux du jour) :

```yaml
name: appli_meteo
description: "Une application météo branchée sur l'API Open-Meteo."
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: ^3.12.0

dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.8
  http: ^1.5.0
  intl: ^0.20.2
  provider: ^6.1.5
  shared_preferences: ^2.5.3

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^6.0.0

flutter:
  uses-material-design: true
```

Aucune section `assets:` : ce projet ne contient **aucune image**. Toutes les icônes viennent de la police Material, tous les fonds sont des dégradés calculés. C'est volontaire : une application météo n'a pas besoin de fichiers pour être belle.

### Les permissions Internet

C'est l'oubli numéro un des débutants, et il produit une erreur trompeuse : tout fonctionne en mode debug, et l'application échoue une fois compilée en release, avec un `SocketException` sans explication.

**Android.** Le fichier `android/app/src/debug/AndroidManifest.xml` généré par Flutter contient déjà la permission, uniquement pour le debug. Il faut l'ajouter au manifeste principal pour la version de production. Ouvrez `android/app/src/main/AndroidManifest.xml` et insérez la ligne avant la balise `<application>` :

```text
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-permission android:name="android.permission.INTERNET"/>

    <application
        android:label="appli_meteo"
        ...
```

**macOS.** Le bac à sable de macOS interdit les connexions sortantes par défaut. Il faut cocher la capacité « client réseau » dans **les deux** fichiers d'entitlements, sinon la version release échouera :

```text
macos/Runner/DebugProfile.entitlements
macos/Runner/Release.entitlements
```

Dans chacun, à l'intérieur de la balise `<dict>` :

```text
<key>com.apple.security.network.client</key>
<true/>
```

**iOS, Windows, Linux, Web.** Rien à faire : l'accès réseau sortant y est autorisé par défaut. Sur le Web, en revanche, le navigateur applique la politique CORS ; Open-Meteo renvoie les en-têtes nécessaires, l'application fonctionne donc aussi dans un onglet.

### Le squelette

Remplacez intégralement `lib/main.dart` :

```dart
// lib/main.dart
import 'package:flutter/material.dart';

void main() {
  runApp(const AppliMeteo());
}

/// Racine de l'application.
///
/// Pour l'instant elle n'affiche qu'un écran vide : nous validons
/// simplement que le projet compile et démarre.
class AppliMeteo extends StatelessWidget {
  const AppliMeteo({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Météo',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        // Une graine bleue : tout le nuancier Material 3 en découle.
        colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF1E88E5)),
        useMaterial3: true,
      ),
      home: const Scaffold(
        body: Center(child: Text('Météo')),
      ),
    );
  }
}
```

```text
flutter run
```

**État exécutable n° 1 :** un écran blanc affichant « Météo ». Le projet compile, les paquets sont résolus, les permissions sont posées. Nous pouvons construire.

---

## 61.3 — Le modèle `Ville`

Premier modèle, et premier `fromJson` (rappel du chapitre 17). Une `Ville` est ce que renvoie l'API de géocodage, allégé de tout ce qui ne nous sert pas : nous ignorons `feature_code`, `postcodes`, `admin2_id` et compagnie.

**Règle de modélisation :** on ne recopie jamais toute la réponse. On garde ce dont l'interface a besoin, plus ce qui identifie l'objet. Ici : un identifiant, un nom, des coordonnées, un pays, une région et un fuseau.

```dart
// lib/modeles/ville.dart

/// Un lieu retourné par l'API de géocodage d'Open-Meteo.
///
/// Cette classe ne dépend ni de Flutter ni de http : elle est testable
/// dans un simple `dart test`.
class Ville {
  /// Identifiant GeoNames, unique et stable. Sert de clé pour les favoris.
  final int id;

  /// Nom court : « Paris », « Bordeaux ».
  final String nom;

  final double latitude;
  final double longitude;

  /// Nom du pays traduit (paramètre `language=fr`) : « France ».
  /// Absent pour certains lieux exotiques, donc nullable.
  final String? pays;

  /// Subdivision de premier niveau : « Île-de-France », « Texas ».
  /// C'est le champ `admin1` du JSON.
  final String? region;

  /// Fuseau horaire IANA : « Europe/Paris ». Informatif ici.
  final String? fuseau;

  const Ville({
    required this.id,
    required this.nom,
    required this.latitude,
    required this.longitude,
    this.pays,
    this.region,
    this.fuseau,
  });

  /// Construit une [Ville] depuis un élément du tableau `results`.
  ///
  /// Chaque lecture est défensive : une clé absente ne doit jamais faire
  /// planter l'application (rappel du chapitre 12 sur le null safety).
  factory Ville.fromJson(Map<String, dynamic> json) {
    return Ville(
      // `as num` puis `.toInt()` : l'API peut renvoyer 2988507 ou 2988507.0
      // selon les décodeurs. On ne présume jamais du type exact d'un nombre.
      id: (json['id'] as num?)?.toInt() ?? 0,
      nom: json['name'] as String? ?? 'Lieu inconnu',
      latitude: (json['latitude'] as num?)?.toDouble() ?? 0,
      longitude: (json['longitude'] as num?)?.toDouble() ?? 0,
      pays: json['country'] as String?,
      region: json['admin1'] as String?,
      fuseau: json['timezone'] as String?,
    );
  }

  /// Sérialisation, utilisée pour persister les favoris et l'historique
  /// (section 61.18). Les noms de clés reprennent ceux de l'API : ainsi
  /// [fromJson] relit sans effort ce que [toJson] a écrit.
  Map<String, dynamic> toJson() => {
        'id': id,
        'name': nom,
        'latitude': latitude,
        'longitude': longitude,
        'country': pays,
        'admin1': region,
        'timezone': fuseau,
      };

  /// « Paris, Île-de-France » — pour le titre de l'écran.
  String get libelleCourt {
    if (region == null || region!.isEmpty) return nom;
    return '$nom, $region';
  }

  /// « Île-de-France · France » — pour le sous-titre d'un résultat de recherche.
  String get sousTitre {
    final morceaux = <String>[
      if (region != null && region!.isNotEmpty) region!,
      if (pays != null && pays!.isNotEmpty) pays!,
    ];
    return morceaux.join(' · ');
  }

  /// Deux villes sont la même si elles ont le même identifiant GeoNames.
  /// Sans cette redéfinition, `favoris.contains(ville)` serait toujours faux,
  /// car Dart comparerait les références d'objets et non leur contenu.
  @override
  bool operator ==(Object other) => other is Ville && other.id == id;

  @override
  int get hashCode => id.hashCode;

  @override
  String toString() => 'Ville($id, $nom, $latitude, $longitude)';
}
```

### Le détail à ne pas manquer

```dart
id: (json['id'] as num?)?.toInt() ?? 0,
```

Trois opérateurs enchaînés, et chacun a une raison :

| Morceau | Raison |
| --- | --- |
| `as num?` | JSON ne distingue pas `int` et `double`. Un `as int` planterait sur `42.0`. |
| `?.toInt()` | si la clé est absente, on ne tente pas la conversion. |
| `?? 0` | valeur de repli, pour que le champ reste non nullable. |

C'est exactement le motif du chapitre 17, appliqué à une vraie API. Vous allez le réécrire une trentaine de fois dans ce projet ; apprenez-le une fois pour toutes.

Et la redéfinition de `==` : sans elle, la section 61.18 (favoris) ne fonctionnerait pas. Testez-le mentalement — `Ville(id: 2988507, ...) == Ville(id: 2988507, ...)` vaut `false` par défaut en Dart, car l'égalité par défaut est l'identité de référence.

---

## 61.4 — L'exception métier `ErreurMeteo`

Avant d'écrire le moindre appel réseau, décidons **comment il échoue**. C'est l'ordre correct : on conçoit les pannes avant le chemin heureux, sinon on les traite à la va-vite à la fin.

Le chapitre 53 (sections 53.18 et 53.19) a recensé les pannes possibles. Les voici, avec ce que l'utilisateur doit lire :

```text
CAUSE TECHNIQUE                       CE QUE L'UTILISATEUR DOIT LIRE
──────────────────────────────────    ──────────────────────────────────────
SocketException                       « Pas de connexion Internet. »
TimeoutException                      « Le serveur met trop de temps. »
statusCode 500, 503                   « Le service météo est indisponible. »
statusCode 400                        « Requête incorrecte. »
FormatException sur jsonDecode        « Réponse illisible du serveur. »
results absent du géocodage           « Aucune ville ne correspond. »
```

Plutôt que de laisser fuiter six types d'exceptions différents vers l'interface, la couche service les convertit toutes en **une seule** exception métier, porteuse d'une cause. L'interface n'a alors qu'un seul `catch` à écrire.

```dart
// lib/services/erreur_meteo.dart

/// Les causes possibles d'échec, du point de vue de l'utilisateur.
///
/// Chaque valeur porte le message affichable et l'icône associée
/// (rappel du chapitre 11 sur les `enum` enrichis).
enum CauseErreur {
  reseau('Pas de connexion Internet.',
      'Vérifiez le Wi-Fi ou les données mobiles, puis réessayez.'),
  delai('Le serveur met trop de temps à répondre.',
      'La connexion est peut-être très lente. Réessayez dans un instant.'),
  serveur('Le service météo est momentanément indisponible.',
      'Ce n\'est pas de votre faute. Réessayez dans quelques minutes.'),
  requete('La requête est incorrecte.',
      'Les coordonnées demandées ne sont pas valides.'),
  donnees('Réponse illisible du serveur.',
      'Le format des données a changé ou la réponse est incomplète.'),
  aucuneVille('Aucune ville ne correspond.',
      'Vérifiez l\'orthographe, ou essayez un nom plus court.');

  const CauseErreur(this.titre, this.detail);

  /// Phrase courte, affichée en gras.
  final String titre;

  /// Phrase d'explication, affichée en dessous.
  final String detail;

  /// Une erreur réseau ou de délai vaut la peine d'être retentée telle quelle ;
  /// une erreur de requête, non.
  bool get peutReessayer => this != CauseErreur.requete;
}

/// L'unique exception que la couche service laisse remonter.
///
/// Toutes les exceptions techniques (`SocketException`, `TimeoutException`,
/// `FormatException`, code HTTP non 200) sont converties en une [ErreurMeteo]
/// avant de quitter le service. L'interface n'a donc qu'un seul type à
/// attraper (rappel du chapitre 13).
class ErreurMeteo implements Exception {
  final CauseErreur cause;

  /// Détail technique, utile en journalisation, jamais affiché à l'utilisateur.
  final String? technique;

  const ErreurMeteo(this.cause, [this.technique]);

  String get titre => cause.titre;
  String get detail => cause.detail;

  @override
  String toString() =>
      'ErreurMeteo(${cause.name})${technique == null ? '' : ' : $technique'}';
}
```

**Pourquoi un `enum` et pas six classes filles ?** Parce que l'interface va faire exactement une chose de la cause : afficher deux phrases et, éventuellement, un bouton. Un `enum` enrichi suffit et tient en trente lignes. Si un jour chaque cause devait porter des données différentes — un code HTTP, une durée, un champ manquant — alors une hiérarchie de classes deviendrait justifiée.

---

## 61.5 — Le service de géocodage

Nous appliquons le patron de la section 53.17 : une classe qui ne connaît que le réseau, qui reçoit son `http.Client` de l'extérieur, et qui renvoie des objets Dart.

```dart
// lib/services/geocodage_service.dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';

import 'package:http/http.dart' as http;

import '../modeles/ville.dart';
import 'erreur_meteo.dart';

/// Recherche de villes par leur nom, via l'API de géocodage d'Open-Meteo.
///
/// Aucune clé d'API n'est nécessaire.
class GeocodageService {
  /// Le client est injecté pour pouvoir le remplacer par un faux dans
  /// les tests (rappel de la section 53.39). En production, on ne passe
  /// rien et un client par défaut est créé.
  GeocodageService({http.Client? client}) : _client = client ?? http.Client();

  final http.Client _client;

  static const String _hote = 'geocoding-api.open-meteo.com';
  static const String _chemin = '/v1/search';
  static const Duration _delaiMax = Duration(seconds: 10);

  /// Construit l'URL de recherche. Méthode publique : elle est testable
  /// sans le moindre accès au réseau.
  Uri construireUri(String requete, {int nombre = 8}) {
    return Uri.https(_hote, _chemin, {
      'name': requete,
      'count': '$nombre',
      'language': 'fr',
      'format': 'json',
    });
  }

  /// Renvoie les villes correspondant à [requete].
  ///
  /// Lève une [ErreurMeteo] en cas de problème, jamais autre chose.
  /// Renvoie une liste vide si [requete] fait moins de deux caractères :
  /// l'API rejette les recherches trop courtes, autant ne pas l'appeler.
  Future<List<Ville>> chercherVilles(String requete, {int nombre = 8}) async {
    final propre = requete.trim();
    if (propre.length < 2) return const <Ville>[];

    final uri = construireUri(propre, nombre: nombre);

    late final http.Response reponse;
    try {
      reponse = await _client.get(uri).timeout(_delaiMax);
    } on SocketException catch (e) {
      // Pas de route vers l'hôte : Wi-Fi coupé, mode avion, DNS en panne.
      throw ErreurMeteo(CauseErreur.reseau, e.message);
    } on TimeoutException {
      throw const ErreurMeteo(CauseErreur.delai, 'timeout de 10 s dépassé');
    } on http.ClientException catch (e) {
      // Sur le Web, une coupure réseau se manifeste par ClientException
      // et non par SocketException. Les deux doivent être attrapées.
      throw ErreurMeteo(CauseErreur.reseau, e.message);
    }

    if (reponse.statusCode != 200) {
      throw ErreurMeteo(
        reponse.statusCode >= 500 ? CauseErreur.serveur : CauseErreur.requete,
        'HTTP ${reponse.statusCode}',
      );
    }

    late final Map<String, dynamic> donnees;
    try {
      // `utf8.decode(reponse.bodyBytes)` et non `reponse.body` :
      // sans cela, « Île-de-France » revient en « Ãle-de-France ».
      donnees = jsonDecode(utf8.decode(reponse.bodyBytes))
          as Map<String, dynamic>;
    } on FormatException catch (e) {
      throw ErreurMeteo(CauseErreur.donnees, e.message);
    } on TypeError {
      throw const ErreurMeteo(CauseErreur.donnees, 'racine JSON inattendue');
    }

    // PIÈGE : quand rien ne correspond, la clé `results` est ABSENTE.
    // Ce n'est pas un tableau vide. Un `as List` direct planterait ici.
    final resultats = donnees['results'];
    if (resultats is! List || resultats.isEmpty) {
      return const <Ville>[];
    }

    return resultats
        .whereType<Map<String, dynamic>>()
        .map(Ville.fromJson)
        .toList();
  }

  /// À appeler quand le service n'est plus utilisé, pour libérer les
  /// connexions persistantes du client.
  void fermer() => _client.close();
}
```

### Quatre décisions expliquées

| Décision | Pourquoi |
| --- | --- |
| `Uri.https(hote, chemin, parametres)` | encode automatiquement les espaces et les accents. `Uri.parse('...?name=Saint-Étienne')` échouerait sur certains serveurs. |
| `utf8.decode(reponse.bodyBytes)` | `reponse.body` suppose du latin-1 quand l'en-tête ne précise pas le jeu de caractères. Tous les accents seraient abîmés. |
| Liste vide plutôt qu'exception si `results` est absent | « aucun résultat » n'est pas une erreur : c'est un résultat. L'écran affichera « Aucune ville ne correspond ». |
| `whereType<Map<String, dynamic>>()` | filtre les éléments inattendus au lieu de planter sur un `cast` (rappel du chapitre 14). |

### Vérification en console

Avant de dessiner quoi que ce soit, prouvez que le service fonctionne. Remplacez temporairement `main.dart` par ceci :

```dart
// lib/main.dart — VERSION TEMPORAIRE DE VÉRIFICATION
import 'package:flutter/material.dart';

import 'services/erreur_meteo.dart';
import 'services/geocodage_service.dart';

Future<void> main() async {
  // Obligatoire dès qu'on fait quelque chose avant runApp.
  WidgetsFlutterBinding.ensureInitialized();

  final service = GeocodageService();
  try {
    final villes = await service.chercherVilles('bordeaux');
    for (final v in villes) {
      debugPrint('${v.nom} — ${v.sousTitre} — ${v.latitude}, ${v.longitude}');
    }
  } on ErreurMeteo catch (e) {
    debugPrint('Échec : ${e.titre} (${e.technique})');
  }

  runApp(const MaterialApp(home: Scaffold(body: Center(child: Text('OK')))));
}
```

```text
flutter run
```

**Résultat attendu dans la console :**

```text
Bordeaux — Nouvelle-Aquitaine · France — 44.84044, -0.5805
Bordeaux — Gauteng · Afrique du Sud — -26.11504, 27.98995
Bordeaux — Ohio · États-Unis — 40.36034, -81.11566
...
```

Les valeurs exactes varient : la base GeoNames évolue. Ce qui compte, c'est que les accents soient corrects et que les coordonnées soient plausibles.

**État exécutable n° 2 :** le géocodage fonctionne. Coupez le Wi-Fi et relancez : vous devez lire « Échec : Pas de connexion Internet. » et non une trace d'exception.

---

## 61.6 — Les modèles météo

Quatre classes, une par bloc de la réponse. Toutes dans `lib/modeles/`, toutes sans aucun import Flutter.

```text
BulletinMeteo
├── Ville            ville               (vient du géocodage, pas de l'API météo)
├── MeteoActuelle    actuelle            ← bloc "current"
├── List<PrevisionHeure> heures          ← bloc "hourly", recomposé par index
└── List<PrevisionJour>  jours           ← bloc "daily",  recomposé par index
```

### L'heure murale : un piège à désamorcer tout de suite

Avec `timezone=auto`, l'API renvoie `"2026-08-15T14:00"` sans indication de fuseau. Cette chaîne signifie « 14 h à l'heure de la ville consultée ». Or `DateTime.parse("2026-08-15T14:00")` produit en Dart un `DateTime` marqué « heure locale de l'appareil ». Si votre téléphone est à Montréal et que vous consultez Tokyo, la comparaison avec `DateTime.now()` sera fausse de treize heures.

La solution est simple et tient en deux fonctions : on manipule uniquement des **heures murales**, c'est-à-dire des `DateTime` dont on ne regarde que les chiffres affichés, jamais l'instant absolu.

```dart
// lib/utilitaires/temps.dart

/// Ramène un [DateTime] à ses chiffres affichés, en oubliant tout fuseau.
///
/// L'API renvoie « 14:00 » pour dire « 14 h dans la ville consultée ».
/// On ne veut jamais convertir cette valeur : on veut juste la lire.
DateTime heureMurale(DateTime instant) => DateTime(
      instant.year,
      instant.month,
      instant.day,
      instant.hour,
      instant.minute,
    );

/// L'heure qu'il est *dans la ville consultée*, exprimée elle aussi
/// en heure murale, donc comparable aux valeurs de l'API.
///
/// [decalageUtcSecondes] est le champ `utc_offset_seconds` de la réponse.
DateTime maintenantDansLaVille(int decalageUtcSecondes) => heureMurale(
      DateTime.now().toUtc().add(Duration(seconds: decalageUtcSecondes)),
    );
```

```text
Appareil à Montréal (UTC-4), ville consultée : Tokyo (utc_offset_seconds = 32400)

DateTime.now()                       →  2026-08-15 01:00 (Montréal)
DateTime.now().toUtc()               →  2026-08-15 05:00 (UTC)
   .add(9 h)                         →  2026-08-15 14:00 (chiffres de Tokyo)
heureMurale(...)                     →  2026-08-15 14:00 comparable à l'API
```

Retenez la règle : **toute date issue de l'API passe par `heureMurale`, et l'instant présent passe par `maintenantDansLaVille`.** Elles sont alors comparables entre elles, et seulement entre elles.

### `MeteoActuelle`

```dart
// lib/modeles/meteo_actuelle.dart
import '../utilitaires/temps.dart';

/// Le bloc `current` de la réponse : un instantané.
class MeteoActuelle {
  /// Heure murale de la mesure, dans le fuseau de la ville.
  final DateTime instant;

  final double temperature;
  final double ressenti;

  /// Humidité relative en pourcentage.
  final int humidite;

  /// Précipitations du dernier quart d'heure, en millimètres.
  final double precipitation;

  /// Code de condition WMO. Voir `logique/codes_wmo.dart`.
  final int codeMeteo;

  final double vitesseVent;

  /// Vrai s'il fait jour au lieu consulté (champ `is_day` valant 1).
  final bool estJour;

  const MeteoActuelle({
    required this.instant,
    required this.temperature,
    required this.ressenti,
    required this.humidite,
    required this.precipitation,
    required this.codeMeteo,
    required this.vitesseVent,
    required this.estJour,
  });

  /// Décode le bloc `current`.
  ///
  /// Les noms de clés sont EXACTEMENT ceux demandés dans l'URL, à la
  /// lettre près. Une faute de frappe ici ne produit pas d'erreur de
  /// compilation : elle produit un `null`, donc une valeur de repli
  /// silencieuse. C'est la raison des tests de la section 61.23.
  factory MeteoActuelle.fromJson(Map<String, dynamic> json) {
    return MeteoActuelle(
      instant: heureMurale(
        DateTime.tryParse(json['time'] as String? ?? '') ?? DateTime.now(),
      ),
      temperature: (json['temperature_2m'] as num?)?.toDouble() ?? 0,
      ressenti: (json['apparent_temperature'] as num?)?.toDouble() ?? 0,
      humidite: (json['relative_humidity_2m'] as num?)?.toInt() ?? 0,
      precipitation: (json['precipitation'] as num?)?.toDouble() ?? 0,
      codeMeteo: (json['weather_code'] as num?)?.toInt() ?? -1,
      vitesseVent: (json['wind_speed_10m'] as num?)?.toDouble() ?? 0,
      // `is_day` est un entier 0/1, pas un booléen JSON.
      estJour: ((json['is_day'] as num?)?.toInt() ?? 1) == 1,
    );
  }
}
```

Notez `codeMeteo: ... ?? -1`. La valeur de repli d'un code météo ne doit **pas** être `0`, car 0 signifie « ciel dégagé » : une donnée manquante afficherait un grand soleil. On choisit une valeur impossible, `-1`, que la table des codes traduira par « Condition inconnue ».

### `PrevisionHeure`

```dart
// lib/modeles/prevision_heure.dart

/// Une case de la bande horaire.
class PrevisionHeure {
  /// Heure murale du créneau, dans le fuseau de la ville.
  final DateTime heure;

  final double temperature;
  final int codeMeteo;

  /// Probabilité de précipitations en pourcentage.
  /// L'API renvoie parfois `null` sur les horizons lointains : le champ
  /// est donc nullable, et l'interface affichera un tiret.
  final int? probabilitePluie;

  const PrevisionHeure({
    required this.heure,
    required this.temperature,
    required this.codeMeteo,
    this.probabilitePluie,
  });
}
```

### `PrevisionJour`

```dart
// lib/modeles/prevision_jour.dart

/// Une ligne de la liste des sept jours.
class PrevisionJour {
  /// Date murale du jour (heure à 00:00).
  final DateTime jour;

  final double temperatureMin;
  final double temperatureMax;
  final int codeMeteo;

  /// Probabilité maximale de précipitations sur la journée.
  final int? probabilitePluieMax;

  /// Lever et coucher du soleil, en heure murale. Nullables : au-delà des
  /// cercles polaires, il n'y a certains jours ni l'un ni l'autre.
  final DateTime? leverSoleil;
  final DateTime? coucherSoleil;

  const PrevisionJour({
    required this.jour,
    required this.temperatureMin,
    required this.temperatureMax,
    required this.codeMeteo,
    this.probabilitePluieMax,
    this.leverSoleil,
    this.coucherSoleil,
  });
}
```

Le commentaire sur les cercles polaires n'est pas décoratif : à Longyearbyen, `sunrise` et `sunset` valent `null` la moitié de l'année. Une application qui suppose ces champs présents plante pour tout un pan de la planète. Modéliser correctement, c'est modéliser le cas rare.

---

## 61.7 — `BulletinMeteo` : décoder des colonnes

Il reste à assembler les trois blocs. C'est ici que se trouve la seule vraie difficulté technique du projet : recomposer des objets à partir de colonnes parallèles.

### La version naïve

Si l'on suppose que toutes les colonnes ont la même longueur et ne contiennent jamais `null`, une seule ligne suffit (rappel du chapitre 14) :

```dart
final heures = List.generate(
  (hourly['time'] as List).length,
  (i) => PrevisionHeure(
    heure: DateTime.parse(hourly['time'][i] as String),
    temperature: (hourly['temperature_2m'][i] as num).toDouble(),
    codeMeteo: (hourly['weather_code'][i] as num).toInt(),
    probabilitePluie: (hourly['precipitation_probability'][i] as num).toInt(),
  ),
);
```

C'est court, c'est élégant, et **cela plantera** le jour où `precipitation_probability` contient un `null` sur l'horizon lointain — ce qui arrive régulièrement. Le `as num` sur un `null` lève une `TypeError`, et votre application affiche un écran rouge à l'utilisateur.

### La version défensive

On isole donc la lecture d'une colonne dans deux petites fonctions, puis on boucle en tolérant les trous et les longueurs inégales.

```dart
// lib/modeles/bulletin_meteo.dart
import '../utilitaires/temps.dart';
import 'meteo_actuelle.dart';
import 'prevision_heure.dart';
import 'prevision_jour.dart';
import 'ville.dart';

/// Lit une colonne numérique d'un bloc `hourly` ou `daily`.
///
/// Renvoie toujours une liste, éventuellement vide, dont chaque case peut
/// valoir `null` : l'API laisse des trous sur les horizons lointains.
List<num?> _colonneNombres(Map<String, dynamic> bloc, String cle) {
  final brut = bloc[cle];
  if (brut is! List) return const <num?>[];
  return brut.map((e) => e is num ? e : null).toList();
}

/// Lit une colonne de dates iso8601 et la ramène en heures murales.
List<DateTime?> _colonneDates(Map<String, dynamic> bloc, String cle) {
  final brut = bloc[cle];
  if (brut is! List) return const <DateTime?>[];
  return brut.map((e) {
    if (e is! String) return null;
    final d = DateTime.tryParse(e);
    return d == null ? null : heureMurale(d);
  }).toList();
}

/// Accès à l'index [i] d'une colonne, sans jamais dépasser sa longueur.
///
/// Sans cette précaution, une colonne plus courte que `time` — cela
/// arrive — provoquerait un `RangeError`.
T? _a<T>(List<T?> colonne, int i) => i < colonne.length ? colonne[i] : null;

/// L'ensemble des données météo d'une ville, prêtes pour l'affichage.
class BulletinMeteo {
  final Ville ville;

  /// Décalage du lieu par rapport à UTC, en secondes (`utc_offset_seconds`).
  /// Indispensable pour savoir quelle heure il est *là-bas*.
  final int decalageUtcSecondes;

  final MeteoActuelle actuelle;
  final List<PrevisionHeure> heures;
  final List<PrevisionJour> jours;

  /// Symbole d'unité lu dans la réponse : « °C » ou « °F ».
  /// On ne le devine pas, on le lit : c'est l'API qui fait foi.
  final String symboleTemperature;

  /// « km/h », « m/s », « mp/h »… selon `wind_speed_unit`.
  final String symboleVent;

  /// Instant réel (horloge de l'appareil) de la récupération.
  /// Sert à afficher l'âge de la donnée en mode hors-ligne.
  final DateTime recupereLe;

  const BulletinMeteo({
    required this.ville,
    required this.decalageUtcSecondes,
    required this.actuelle,
    required this.heures,
    required this.jours,
    required this.symboleTemperature,
    required this.symboleVent,
    required this.recupereLe,
  });

  /// Décode la réponse complète de `/v1/forecast`.
  ///
  /// [ville] ne vient pas de cette réponse : l'API de prévision ne connaît
  /// que des coordonnées. C'est le géocodage qui a fourni le nom.
  ///
  /// Lève une [FormatException] si la structure attendue est absente.
  /// La conversion en [ErreurMeteo] est faite par la couche service : un
  /// modèle ne doit rien savoir du réseau.
  factory BulletinMeteo.fromJson(
    Map<String, dynamic> json, {
    required Ville ville,
    DateTime? recupereLe,
  }) {
    final current = json['current'];
    if (current is! Map<String, dynamic>) {
      throw const FormatException('bloc "current" absent ou malformé');
    }

    final unites = json['current_units'];
    final unitesMap =
        unites is Map<String, dynamic> ? unites : const <String, dynamic>{};

    // ---- Bloc horaire ------------------------------------------------
    final hourly = json['hourly'];
    final heures = <PrevisionHeure>[];
    if (hourly is Map<String, dynamic>) {
      final temps = _colonneDates(hourly, 'time');
      final temperatures = _colonneNombres(hourly, 'temperature_2m');
      final codes = _colonneNombres(hourly, 'weather_code');
      final probabilites = _colonneNombres(hourly, 'precipitation_probability');

      for (var i = 0; i < temps.length; i++) {
        final quand = temps[i];
        // Une case de temps illisible rend toute la ligne inutilisable :
        // on la saute au lieu d'inventer une heure.
        if (quand == null) continue;
        heures.add(PrevisionHeure(
          heure: quand,
          temperature: _a(temperatures, i)?.toDouble() ?? 0,
          codeMeteo: _a(codes, i)?.toInt() ?? -1,
          probabilitePluie: _a(probabilites, i)?.toInt(),
        ));
      }
    }

    // ---- Bloc journalier ---------------------------------------------
    final daily = json['daily'];
    final jours = <PrevisionJour>[];
    if (daily is Map<String, dynamic>) {
      final dates = _colonneDates(daily, 'time');
      final maxi = _colonneNombres(daily, 'temperature_2m_max');
      final mini = _colonneNombres(daily, 'temperature_2m_min');
      final codes = _colonneNombres(daily, 'weather_code');
      final probabilites =
          _colonneNombres(daily, 'precipitation_probability_max');
      final levers = _colonneDates(daily, 'sunrise');
      final couchers = _colonneDates(daily, 'sunset');

      for (var i = 0; i < dates.length; i++) {
        final jour = dates[i];
        if (jour == null) continue;
        jours.add(PrevisionJour(
          jour: jour,
          temperatureMax: _a(maxi, i)?.toDouble() ?? 0,
          temperatureMin: _a(mini, i)?.toDouble() ?? 0,
          codeMeteo: _a(codes, i)?.toInt() ?? -1,
          probabilitePluieMax: _a(probabilites, i)?.toInt(),
          leverSoleil: _a(levers, i),
          coucherSoleil: _a(couchers, i),
        ));
      }
    }

    return BulletinMeteo(
      ville: ville,
      decalageUtcSecondes: (json['utc_offset_seconds'] as num?)?.toInt() ?? 0,
      actuelle: MeteoActuelle.fromJson(current),
      heures: heures,
      jours: jours,
      symboleTemperature: unitesMap['temperature_2m'] as String? ?? '°C',
      symboleVent: unitesMap['wind_speed_10m'] as String? ?? 'km/h',
      recupereLe: recupereLe ?? DateTime.now(),
    );
  }

  /// L'heure qu'il est dans la ville consultée, en heure murale.
  DateTime get maintenantLaBas => maintenantDansLaVille(decalageUtcSecondes);

  /// Les 24 créneaux à venir, en partant de l'heure en cours.
  ///
  /// Le bloc `hourly` commence toujours à minuit du premier jour : sans
  /// ce filtrage, la bande horaire afficherait des heures déjà passées.
  List<PrevisionHeure> get prochainesHeures {
    final debut = maintenantLaBas.subtract(const Duration(minutes: 59));
    return heures.where((h) => h.heure.isAfter(debut)).take(24).toList();
  }

  /// La prévision du jour même, ou `null` si le bloc `daily` est vide.
  PrevisionJour? get aujourdHui => jours.isEmpty ? null : jours.first;

  /// Vrai si la donnée date de plus d'une heure : l'interface le signale.
  bool get estPerimee =>
      DateTime.now().difference(recupereLe) > const Duration(hours: 1);
}
```

### Ce que ce code vous apprend

| Précaution | Ce qu'elle évite |
| --- | --- |
| `bloc[cle] is! List` → liste vide | un `TypeError` si une variable n'a pas été demandée dans l'URL |
| `e is num ? e : null` | un `TypeError` sur les trous de l'API |
| `_a(colonne, i)` au lieu de `colonne[i]` | un `RangeError` sur une colonne plus courte |
| `if (quand == null) continue;` | une ligne d'affichage avec une date inventée |
| lire `current_units` au lieu de coder « °C » en dur | un affichage faux quand on bascule en Fahrenheit |
| `FormatException` et non `ErreurMeteo` | une dépendance de `modeles/` vers `services/` |

Quinze lignes de plus que la version naïve, et l'application ne plantera pas devant l'utilisateur. C'est le prix normal du code de production.

---

## 61.8 — Le service météo

### L'unité de température

Un tout petit fichier, mais qui sera lu par le service, par l'interface et par la persistance. Il mérite d'exister seul.

```dart
// lib/logique/unites.dart

/// Unité de température choisie par l'utilisateur.
///
/// [parametreApi] est la valeur exacte attendue par le paramètre
/// `temperature_unit` d'Open-Meteo : « celsius » ou « fahrenheit ».
enum UniteTemperature {
  celsius('celsius', '°C', 'Celsius'),
  fahrenheit('fahrenheit', '°F', 'Fahrenheit');

  const UniteTemperature(this.parametreApi, this.symbole, this.libelle);

  final String parametreApi;
  final String symbole;
  final String libelle;

  /// L'autre unité, pour un bouton de bascule.
  UniteTemperature get inverse =>
      this == UniteTemperature.celsius ? fahrenheit : celsius;

  /// Relit une valeur persistée. On stocke `.name`, jamais `.index`
  /// (rappel de la règle du chapitre 58).
  static UniteTemperature depuisNom(String? nom) =>
      UniteTemperature.values.firstWhere(
        (u) => u.name == nom,
        orElse: () => UniteTemperature.celsius,
      );
}
```

**Pourquoi demander la conversion à l'API plutôt que de la calculer ?** Parce que `F = C × 9/5 + 32` n'est pas la seule conversion en jeu : le vent doit passer en `mp/h`, les précipitations en `inch`, et les arrondis doivent rester cohérents. Un paramètre d'URL fait tout cela sans une ligne de calcul chez vous, et sans risque d'erreur.

### Le service

```dart
// lib/services/meteo_service.dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';

import 'package:http/http.dart' as http;

import '../logique/unites.dart';
import '../modeles/bulletin_meteo.dart';
import '../modeles/ville.dart';
import 'erreur_meteo.dart';

/// Appelle l'API de prévision d'Open-Meteo et renvoie un [BulletinMeteo].
///
/// Aucune clé d'API. Aucun en-tête d'authentification. Une simple requête
/// GET publique.
class MeteoService {
  MeteoService({http.Client? client}) : _client = client ?? http.Client();

  final http.Client _client;

  static const String _hote = 'api.open-meteo.com';
  static const String _chemin = '/v1/forecast';
  static const Duration _delaiMax = Duration(seconds: 12);

  /// Les variables demandées, écrites une fois pour toutes.
  ///
  /// Ces noms viennent de la documentation officielle. Une faute de frappe
  /// n'est PAS détectée à la compilation : l'API ignore silencieusement une
  /// variable inconnue, et le bloc correspondant sera absent de la réponse.
  static const List<String> _variablesActuelles = [
    'temperature_2m',
    'relative_humidity_2m',
    'apparent_temperature',
    'is_day',
    'precipitation',
    'weather_code',
    'wind_speed_10m',
  ];

  static const List<String> _variablesHoraires = [
    'temperature_2m',
    'weather_code',
    'precipitation_probability',
  ];

  static const List<String> _variablesJournalieres = [
    'weather_code',
    'temperature_2m_max',
    'temperature_2m_min',
    'sunrise',
    'sunset',
    'precipitation_probability_max',
  ];

  /// Construit l'URL complète. Publique, donc testable sans réseau.
  Uri construireUri(
    Ville ville, {
    UniteTemperature unite = UniteTemperature.celsius,
    int nombreJours = 7,
  }) {
    return Uri.https(_hote, _chemin, {
      'latitude': '${ville.latitude}',
      'longitude': '${ville.longitude}',
      'current': _variablesActuelles.join(','),
      'hourly': _variablesHoraires.join(','),
      'daily': _variablesJournalieres.join(','),
      // `auto` demande les heures dans le fuseau du lieu interrogé.
      'timezone': 'auto',
      'forecast_days': '$nombreJours',
      'temperature_unit': unite.parametreApi,
      'wind_speed_unit': unite == UniteTemperature.fahrenheit ? 'mph' : 'kmh',
      'precipitation_unit':
          unite == UniteTemperature.fahrenheit ? 'inch' : 'mm',
    });
  }

  /// Récupère le bulletin complet de [ville].
  ///
  /// Ne lève jamais autre chose qu'une [ErreurMeteo].
  Future<BulletinMeteo> obtenirBulletin(
    Ville ville, {
    UniteTemperature unite = UniteTemperature.celsius,
  }) async {
    final corps = await telechargerJson(ville, unite: unite);
    try {
      return BulletinMeteo.fromJson(corps, ville: ville);
    } on FormatException catch (e) {
      throw ErreurMeteo(CauseErreur.donnees, e.message);
    } on TypeError catch (e) {
      throw ErreurMeteo(CauseErreur.donnees, '$e');
    }
  }

  /// Étape réseau isolée : renvoie le JSON brut décodé.
  ///
  /// Séparée d'[obtenirBulletin] pour deux raisons : le cache (section
  /// 61.19) veut enregistrer ce JSON brut, et les tests veulent pouvoir
  /// vérifier le décodage sans rejouer la partie réseau.
  Future<Map<String, dynamic>> telechargerJson(
    Ville ville, {
    UniteTemperature unite = UniteTemperature.celsius,
  }) async {
    final uri = construireUri(ville, unite: unite);

    late final http.Response reponse;
    try {
      reponse = await _client.get(uri).timeout(_delaiMax);
    } on SocketException catch (e) {
      throw ErreurMeteo(CauseErreur.reseau, e.message);
    } on TimeoutException {
      throw const ErreurMeteo(CauseErreur.delai, 'timeout de 12 s dépassé');
    } on http.ClientException catch (e) {
      throw ErreurMeteo(CauseErreur.reseau, e.message);
    }

    if (reponse.statusCode != 200) {
      // Open-Meteo renvoie 400 avec un corps {"error":true,"reason":"..."}
      // lorsqu'un paramètre est invalide. On récupère la raison pour la
      // journalisation, sans jamais l'afficher telle quelle.
      String? raison;
      try {
        final erreur = jsonDecode(reponse.body);
        if (erreur is Map && erreur['reason'] is String) {
          raison = erreur['reason'] as String;
        }
      } on FormatException {
        raison = null;
      }
      throw ErreurMeteo(
        reponse.statusCode >= 500 ? CauseErreur.serveur : CauseErreur.requete,
        'HTTP ${reponse.statusCode}${raison == null ? '' : ' : $raison'}',
      );
    }

    try {
      return jsonDecode(utf8.decode(reponse.bodyBytes))
          as Map<String, dynamic>;
    } on FormatException catch (e) {
      throw ErreurMeteo(CauseErreur.donnees, e.message);
    } on TypeError {
      throw const ErreurMeteo(CauseErreur.donnees, 'racine JSON inattendue');
    }
  }

  void fermer() => _client.close();
}
```

### Vérification en console

```dart
// lib/main.dart — VERSION TEMPORAIRE DE VÉRIFICATION
import 'package:flutter/material.dart';

import 'services/erreur_meteo.dart';
import 'services/geocodage_service.dart';
import 'services/meteo_service.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  final geo = GeocodageService();
  final meteo = MeteoService();

  try {
    final villes = await geo.chercherVilles('Paris');
    final paris = villes.first;
    debugPrint('URL appelée : ${meteo.construireUri(paris)}');

    final bulletin = await meteo.obtenirBulletin(paris);
    debugPrint('Ville      : ${bulletin.ville.libelleCourt}');
    debugPrint('Il est     : ${bulletin.maintenantLaBas}');
    debugPrint('Température: ${bulletin.actuelle.temperature}'
        '${bulletin.symboleTemperature}');
    debugPrint('Code météo : ${bulletin.actuelle.codeMeteo}');
    debugPrint('Jour ?     : ${bulletin.actuelle.estJour}');
    debugPrint('Heures     : ${bulletin.heures.length} créneaux, dont '
        '${bulletin.prochainesHeures.length} à venir');
    debugPrint('Jours      : ${bulletin.jours.length}');
    debugPrint('Demain max : ${bulletin.jours[1].temperatureMax}');
  } on ErreurMeteo catch (e) {
    debugPrint('Échec : ${e.titre} — ${e.technique}');
  }

  runApp(const MaterialApp(home: Scaffold(body: Center(child: Text('OK')))));
}
```

**Résultat attendu** (les valeurs changent, la forme non) :

```text
URL appelée : https://api.open-meteo.com/v1/forecast?latitude=48.85341&...
Ville      : Paris, Île-de-France
Il est     : 2026-08-15 14:07:00.000
Température: 26.4°C
Code météo : 2
Jour ?     : true
Heures     : 168 créneaux, dont 24 à venir
Jours      : 7
Demain max : 26.3
```

168 créneaux, c'est 7 jours × 24 heures : la preuve que la recomposition par colonnes a fonctionné.

**État exécutable n° 3 :** la chaîne complète géocodage → prévision → objets Dart tourne. Tout le reste du chapitre n'est plus que de l'interface.

---

## 61.9 — Les codes WMO : du nombre au français

L'API ne renvoie pas « Partiellement nuageux ». Elle renvoie `2`. C'est un **code WMO**, une norme de l'Organisation météorologique mondiale, et c'est à l'application de le traduire.

Voici la table officielle publiée par Open-Meteo, avec les libellés d'origine et notre traduction :

| Code | Libellé Open-Meteo | Traduction retenue |
| --- | --- | --- |
| 0 | Clear sky | Ciel dégagé |
| 1 | Mainly clear | Plutôt dégagé |
| 2 | Partly cloudy | Partiellement nuageux |
| 3 | Overcast | Couvert |
| 45 | Fog | Brouillard |
| 48 | Depositing rime fog | Brouillard givrant |
| 51 | Drizzle: light | Bruine légère |
| 53 | Drizzle: moderate | Bruine modérée |
| 55 | Drizzle: dense | Bruine dense |
| 56 | Freezing drizzle: light | Bruine verglaçante légère |
| 57 | Freezing drizzle: dense | Bruine verglaçante dense |
| 61 | Rain: slight | Pluie faible |
| 63 | Rain: moderate | Pluie modérée |
| 65 | Rain: heavy | Pluie forte |
| 66 | Freezing rain: light | Pluie verglaçante faible |
| 67 | Freezing rain: heavy | Pluie verglaçante forte |
| 71 | Snow fall: slight | Neige faible |
| 73 | Snow fall: moderate | Neige modérée |
| 75 | Snow fall: heavy | Neige forte |
| 77 | Snow grains | Grains de neige |
| 80 | Rain showers: slight | Averses faibles |
| 81 | Rain showers: moderate | Averses modérées |
| 82 | Rain showers: violent | Averses violentes |
| 85 | Snow showers: slight | Averses de neige faibles |
| 86 | Snow showers: heavy | Averses de neige fortes |
| 95 | Thunderstorm: slight or moderate | Orage |
| 96 | Thunderstorm with slight hail | Orage avec grêle |
| 99 | Thunderstorm with heavy hail | Orage avec forte grêle |

Les codes de grêle 96 et 99 ne sont fournis que pour l'Europe centrale : ailleurs, un orage grêleux sortira en 95. Ce n'est pas un défaut de votre code.

Notez surtout que la numérotation est **trouée** : il n'existe pas de code 4, ni 20, ni 62. Une structure `switch` sur des valeurs isolées est donc le bon outil ; un tableau indexé serait absurde.

```dart
// lib/logique/codes_wmo.dart

/// Grandes familles de conditions, utiles pour choisir une icône,
/// une couleur de fond ou un dégradé.
enum FamilleMeteo {
  degage,
  peuNuageux,
  nuageux,
  brouillard,
  bruine,
  pluie,
  neige,
  averse,
  orage,
  inconnu,
}

/// Traduit un code WMO en libellé français affichable.
///
/// Renvoie « Condition inconnue » pour tout code hors table, y compris
/// la valeur de repli -1 posée par les modèles.
String libelleWmo(int code) {
  switch (code) {
    case 0:
      return 'Ciel dégagé';
    case 1:
      return 'Plutôt dégagé';
    case 2:
      return 'Partiellement nuageux';
    case 3:
      return 'Couvert';
    case 45:
      return 'Brouillard';
    case 48:
      return 'Brouillard givrant';
    case 51:
      return 'Bruine légère';
    case 53:
      return 'Bruine modérée';
    case 55:
      return 'Bruine dense';
    case 56:
      return 'Bruine verglaçante légère';
    case 57:
      return 'Bruine verglaçante dense';
    case 61:
      return 'Pluie faible';
    case 63:
      return 'Pluie modérée';
    case 65:
      return 'Pluie forte';
    case 66:
      return 'Pluie verglaçante faible';
    case 67:
      return 'Pluie verglaçante forte';
    case 71:
      return 'Neige faible';
    case 73:
      return 'Neige modérée';
    case 75:
      return 'Neige forte';
    case 77:
      return 'Grains de neige';
    case 80:
      return 'Averses faibles';
    case 81:
      return 'Averses modérées';
    case 82:
      return 'Averses violentes';
    case 85:
      return 'Averses de neige faibles';
    case 86:
      return 'Averses de neige fortes';
    case 95:
      return 'Orage';
    case 96:
      return 'Orage avec grêle';
    case 99:
      return 'Orage avec forte grêle';
    default:
      return 'Condition inconnue';
  }
}

/// Regroupe un code WMO dans une famille.
///
/// La syntaxe `case 51 || 53 || 55:` est celle des motifs de Dart 3 :
/// elle remplace avantageusement une cascade de `case` vides.
FamilleMeteo familleWmo(int code) {
  switch (code) {
    case 0:
      return FamilleMeteo.degage;
    case 1 || 2:
      return FamilleMeteo.peuNuageux;
    case 3:
      return FamilleMeteo.nuageux;
    case 45 || 48:
      return FamilleMeteo.brouillard;
    case 51 || 53 || 55 || 56 || 57:
      return FamilleMeteo.bruine;
    case 61 || 63 || 65 || 66 || 67:
      return FamilleMeteo.pluie;
    case 71 || 73 || 75 || 77 || 85 || 86:
      return FamilleMeteo.neige;
    case 80 || 81 || 82:
      return FamilleMeteo.averse;
    case 95 || 96 || 99:
      return FamilleMeteo.orage;
    default:
      return FamilleMeteo.inconnu;
  }
}

/// Vrai si la condition justifie de sortir couvert. Utilisé pour le
/// bandeau d'alerte de la carte du jour.
bool estPluvieux(int code) {
  final f = familleWmo(code);
  return f == FamilleMeteo.bruine ||
      f == FamilleMeteo.pluie ||
      f == FamilleMeteo.averse ||
      f == FamilleMeteo.orage;
}
```

Ce fichier n'importe rien du tout. Ni Flutter, ni `http`, ni même `dart:io`. C'est du Dart pur, donc testable en quelques millisecondes (section 61.23).

---

## 61.10 — De la condition à l'icône

L'icône, elle, dépend de Flutter : elle vit donc dans `widgets/`, pas dans `logique/`. Et elle dépend d'un second paramètre que l'on oublie souvent : **il fait jour ou nuit**. Un ciel dégagé se dessine avec un soleil le jour, avec une lune la nuit.

```dart
// lib/widgets/icone_meteo.dart
import 'package:flutter/material.dart';

import '../logique/codes_wmo.dart';

/// Choisit l'icône Material correspondant à un code WMO.
///
/// [estJour] fait toute la différence pour les conditions dégagées :
/// personne n'attend un soleil à deux heures du matin.
IconData iconeMeteo(int code, {required bool estJour}) {
  switch (familleWmo(code)) {
    case FamilleMeteo.degage:
      return estJour ? Icons.wb_sunny_rounded : Icons.nightlight_round;
    case FamilleMeteo.peuNuageux:
      return estJour ? Icons.wb_cloudy_outlined : Icons.nights_stay_rounded;
    case FamilleMeteo.nuageux:
      return Icons.cloud_rounded;
    case FamilleMeteo.brouillard:
      return Icons.foggy;
    case FamilleMeteo.bruine:
      return Icons.grain_rounded;
    case FamilleMeteo.pluie:
      return Icons.water_drop_rounded;
    case FamilleMeteo.averse:
      return Icons.umbrella_rounded;
    case FamilleMeteo.neige:
      return Icons.ac_unit_rounded;
    case FamilleMeteo.orage:
      return Icons.thunderstorm_rounded;
    case FamilleMeteo.inconnu:
      return Icons.help_outline_rounded;
  }
}

/// Une icône météo prête à poser, avec sa taille et sa couleur.
class IconeMeteo extends StatelessWidget {
  const IconeMeteo({
    super.key,
    required this.code,
    required this.estJour,
    this.taille = 32,
    this.couleur,
  });

  final int code;
  final bool estJour;
  final double taille;
  final Color? couleur;

  @override
  Widget build(BuildContext context) {
    return Icon(
      iconeMeteo(code, estJour: estJour),
      size: taille,
      color: couleur ?? Colors.white,
      // Lu par les lecteurs d'écran : une icône sans étiquette est
      // invisible pour une personne malvoyante.
      semanticLabel: libelleWmo(code),
    );
  }
}
```

Remarquez le `switch` **sans `default`** sur un `enum` : Dart vérifie alors que tous les cas sont couverts. Si vous ajoutez demain une famille `grele`, le compilateur signalera ce fichier. C'est l'un des grands avantages des `enum` sur les chaînes de caractères, déjà signalé au chapitre 11.

---

## 61.11 — L'écran principal avec `FutureBuilder`

### Les trois états, et pourquoi on les traite tous

Un écran branché sur le réseau n'a jamais un seul visage, mais trois. Le débutant n'en dessine qu'un — celui du succès — et découvre les deux autres en production.

```text
                      ┌──────────────┐
       initState  ──▶ │  CHARGEMENT  │
                      └──────┬───────┘
                             │
                ┌────────────┴────────────┐
                ▼                         ▼
        ┌──────────────┐          ┌──────────────┐
        │   ERREUR     │          │    SUCCÈS    │
        │  + Réessayer │          │   données    │
        └──────┬───────┘          └──────────────┘
               │ appui
               └──────────▶ retour à CHARGEMENT
```

`FutureBuilder` (section 53.22) matérialise exactement ces trois états dans son `AsyncSnapshot`. Écrivons d'abord les widgets de chargement et d'erreur, puis l'écran.

```dart
// lib/widgets/etats_ecran.dart
import 'package:flutter/material.dart';

import '../services/erreur_meteo.dart';

/// L'état « en cours de chargement ».
///
/// Un simple `CircularProgressIndicator` centré suffit, mais on lui ajoute
/// un texte : un rond qui tourne sans explication inquiète l'utilisateur.
class EtatChargement extends StatelessWidget {
  const EtatChargement({super.key, this.message = 'Chargement de la météo…'});

  final String message;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          const CircularProgressIndicator(color: Colors.white),
          const SizedBox(height: 20),
          Text(
            message,
            style: const TextStyle(color: Colors.white, fontSize: 16),
          ),
        ],
      ),
    );
  }
}

/// L'état « quelque chose a échoué ».
///
/// Trois éléments obligatoires : dire QUOI, dire POURQUOI, proposer QUOI FAIRE.
/// Un écran d'erreur sans bouton est une impasse.
class EtatErreur extends StatelessWidget {
  const EtatErreur({super.key, required this.erreur, this.onReessayer});

  final ErreurMeteo erreur;
  final VoidCallback? onReessayer;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Padding(
        padding: const EdgeInsets.all(32),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Icon(
              erreur.cause == CauseErreur.reseau
                  ? Icons.wifi_off_rounded
                  : Icons.error_outline_rounded,
              size: 64,
              color: Colors.white,
            ),
            const SizedBox(height: 20),
            Text(
              erreur.titre,
              textAlign: TextAlign.center,
              style: const TextStyle(
                color: Colors.white,
                fontSize: 20,
                fontWeight: FontWeight.w600,
              ),
            ),
            const SizedBox(height: 10),
            Text(
              erreur.detail,
              textAlign: TextAlign.center,
              style: const TextStyle(color: Colors.white70, fontSize: 15),
            ),
            const SizedBox(height: 28),
            if (onReessayer != null && erreur.cause.peutReessayer)
              FilledButton.tonalIcon(
                onPressed: onReessayer,
                icon: const Icon(Icons.refresh_rounded),
                label: const Text('Réessayer'),
              ),
          ],
        ),
      ),
    );
  }
}

/// Le cas particulier « tout va bien, mais il n'y a rien à montrer ».
/// Ici : aucune ville n'a encore été choisie.
class EtatAucuneVille extends StatelessWidget {
  const EtatAucuneVille({super.key, required this.onChercher});

  final VoidCallback onChercher;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          const Icon(Icons.travel_explore_rounded,
              size: 64, color: Colors.white),
          const SizedBox(height: 20),
          const Text(
            'Aucune ville sélectionnée',
            style: TextStyle(
                color: Colors.white, fontSize: 20, fontWeight: FontWeight.w600),
          ),
          const SizedBox(height: 10),
          const Text(
            'Cherchez une ville pour afficher sa météo.',
            style: TextStyle(color: Colors.white70, fontSize: 15),
          ),
          const SizedBox(height: 28),
          FilledButton.tonalIcon(
            onPressed: onChercher,
            icon: const Icon(Icons.search_rounded),
            label: const Text('Chercher une ville'),
          ),
        ],
      ),
    );
  }
}
```

### La première version de l'écran

Cette version travaille sur une ville figée, sans recherche ni favoris. Elle sert à valider les trois états. Nous l'enrichirons ensuite.

```dart
// lib/ecrans/ecran_meteo.dart — PREMIÈRE VERSION
import 'package:flutter/material.dart';

import '../modeles/bulletin_meteo.dart';
import '../modeles/ville.dart';
import '../services/erreur_meteo.dart';
import '../services/meteo_service.dart';
import '../widgets/etats_ecran.dart';

/// Ville de départ, en dur pour l'instant. Elle disparaîtra à la section
/// 61.17, quand la recherche sera en place.
const Ville villeParDefaut = Ville(
  id: 2988507,
  nom: 'Paris',
  latitude: 48.85341,
  longitude: 2.3488,
  pays: 'France',
  region: 'Île-de-France',
  fuseau: 'Europe/Paris',
);

class EcranMeteo extends StatefulWidget {
  const EcranMeteo({super.key});

  @override
  State<EcranMeteo> createState() => _EcranMeteoState();
}

class _EcranMeteoState extends State<EcranMeteo> {
  final MeteoService _service = MeteoService();

  /// LE point crucial (section 53.24) : le Future est créé UNE FOIS,
  /// dans initState, et jamais dans build. Un `Future` créé dans `build`
  /// serait relancé à chaque reconstruction, donc en boucle infinie.
  late Future<BulletinMeteo> _futur;

  @override
  void initState() {
    super.initState();
    _futur = _service.obtenirBulletin(villeParDefaut);
  }

  @override
  void dispose() {
    _service.fermer();
    super.dispose();
  }

  /// Relance la requête. Le `setState` sert ici à remplacer l'objet
  /// `Future` : c'est ce changement de référence qui fait repartir le
  /// `FutureBuilder` à l'état « en attente ».
  void _recharger() {
    setState(() {
      _futur = _service.obtenirBulletin(villeParDefaut);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF1B4B8F),
      appBar: AppBar(
        backgroundColor: Colors.transparent,
        foregroundColor: Colors.white,
        title: const Text('Météo'),
      ),
      body: FutureBuilder<BulletinMeteo>(
        future: _futur,
        builder: (context, instantane) {
          // ÉTAT 1 — la requête est en cours.
          if (instantane.connectionState == ConnectionState.waiting) {
            return const EtatChargement();
          }

          // ÉTAT 2 — la requête a échoué.
          if (instantane.hasError) {
            final erreur = instantane.error;
            return EtatErreur(
              // Le service ne laisse remonter que des ErreurMeteo, mais
              // une programmation défensive coûte deux lignes.
              erreur: erreur is ErreurMeteo
                  ? erreur
                  : const ErreurMeteo(CauseErreur.donnees),
              onReessayer: _recharger,
            );
          }

          // ÉTAT 3 — succès. `hasData` est vrai, `data` est non nul.
          final bulletin = instantane.data!;
          return Center(
            child: Column(
              mainAxisSize: MainAxisSize.min,
              children: [
                Text(
                  bulletin.ville.libelleCourt,
                  style: const TextStyle(color: Colors.white70, fontSize: 18),
                ),
                Text(
                  '${bulletin.actuelle.temperature.round()}'
                  '${bulletin.symboleTemperature}',
                  style: const TextStyle(color: Colors.white, fontSize: 64),
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

Et le `main.dart`, qui redevient propre :

```dart
// lib/main.dart
import 'package:flutter/material.dart';

import 'ecrans/ecran_meteo.dart';

void main() {
  runApp(const AppliMeteo());
}

class AppliMeteo extends StatelessWidget {
  const AppliMeteo({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Météo',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF1E88E5)),
        useMaterial3: true,
      ),
      home: const EcranMeteo(),
    );
  }
}
```

**État exécutable n° 4 :** l'application affiche « Paris, Île-de-France » et la température actuelle. Passez en mode avion et relancez : vous obtenez l'écran d'erreur et son bouton « Réessayer », qui refonctionne dès le retour du réseau.

### Les deux erreurs classiques de `FutureBuilder`

| Erreur | Symptôme | Correction |
| --- | --- | --- |
| `future: _service.obtenirBulletin(...)` écrit directement dans `build` | requêtes en boucle, compteur d'appels qui explose | créer le `Future` dans `initState` |
| `instantane.data!` sans vérifier l'état | `Null check operator used on a null value` | tester `waiting` et `hasError` **avant** |

---

## 61.12 — Le fond dégradé selon la météo et l'heure

Une application météo se reconnaît à son fond. Il ne coûte presque rien et change complètement la perception du produit.

Le principe : deux entrées — la **famille** de condition et le **moment** (jour ou nuit) — donnent deux couleurs, du haut vers le bas.

```text
              JOUR                          NUIT
dégagé      #3A8FE0 → #82C4F8          #0B1E3D → #1E3A63
nuageux     #6B8CA8 → #A8BECF          #1A2735 → #2E3E4F
pluie       #4A6478 → #7C93A3          #101C26 → #24333F
neige       #8FA8C0 → #D6E4F0          #1B2838 → #3A4A5E
orage       #33404D → #5B6875          #0A0E14 → #232B36
```

```dart
// lib/widgets/fond_meteo.dart
import 'package:flutter/material.dart';

import '../logique/codes_wmo.dart';

/// Peint un dégradé vertical dépendant de la condition et du moment.
///
/// Ce widget enveloppe le contenu de l'écran ; il ne dessine rien d'autre
/// qu'un fond, et n'impose aucune contrainte à son enfant.
class FondMeteo extends StatelessWidget {
  const FondMeteo({
    super.key,
    required this.code,
    required this.estJour,
    required this.child,
  });

  final int code;
  final bool estJour;
  final Widget child;

  /// Les deux couleurs du dégradé, du haut vers le bas.
  List<Color> get _couleurs {
    switch (familleWmo(code)) {
      case FamilleMeteo.degage:
      case FamilleMeteo.peuNuageux:
        return estJour
            ? const [Color(0xFF3A8FE0), Color(0xFF82C4F8)]
            : const [Color(0xFF0B1E3D), Color(0xFF1E3A63)];
      case FamilleMeteo.nuageux:
      case FamilleMeteo.brouillard:
        return estJour
            ? const [Color(0xFF6B8CA8), Color(0xFFA8BECF)]
            : const [Color(0xFF1A2735), Color(0xFF2E3E4F)];
      case FamilleMeteo.bruine:
      case FamilleMeteo.pluie:
      case FamilleMeteo.averse:
        return estJour
            ? const [Color(0xFF4A6478), Color(0xFF7C93A3)]
            : const [Color(0xFF101C26), Color(0xFF24333F)];
      case FamilleMeteo.neige:
        return estJour
            ? const [Color(0xFF8FA8C0), Color(0xFFD6E4F0)]
            : const [Color(0xFF1B2838), Color(0xFF3A4A5E)];
      case FamilleMeteo.orage:
        return estJour
            ? const [Color(0xFF33404D), Color(0xFF5B6875)]
            : const [Color(0xFF0A0E14), Color(0xFF232B36)];
      case FamilleMeteo.inconnu:
        return const [Color(0xFF37474F), Color(0xFF62757F)];
    }
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedContainer(
      // Le changement de ville modifie souvent le dégradé : une transition
      // d'une demi-seconde évite un clignotement brutal.
      duration: const Duration(milliseconds: 500),
      decoration: BoxDecoration(
        gradient: LinearGradient(
          begin: Alignment.topCenter,
          end: Alignment.bottomCenter,
          colors: _couleurs,
        ),
      ),
      child: child,
    );
  }
}
```

**Détail d'ergonomie important :** sur les dégradés de neige, le texte blanc devient illisible. Ce projet garde volontairement les fonds clairs assez soutenus pour rester compatibles avec du blanc. Si vous éclaircissez la palette, il faudra calculer la couleur du texte à partir de la luminance du fond — c'est le défi 6 de la section 61.27.

---

## 61.13 — La carte de la météo du jour

C'est le bloc principal : température géante, condition, ressenti, extrêmes du jour et trois indicateurs.

```text
┌────────────────────────────────────────┐
│                  ☀                     │  icône 96 px
│                26 °C                   │  température 72 px, fine
│         Partiellement nuageux          │  libellé WMO
│            Ressenti 27 °C              │
│                                        │
│       ↑ 29 °C            ↓ 17 °C       │  extrêmes du jour
│  ────────────────────────────────────  │
│  Humidité      Vent         Pluie      │
│    48 %      12 km/h        0 mm       │
└────────────────────────────────────────┘
```

```dart
// lib/widgets/carte_actuelle.dart
import 'package:flutter/material.dart';

import '../logique/codes_wmo.dart';
import '../modeles/bulletin_meteo.dart';
import 'icone_meteo.dart';

/// La grande carte du haut de l'écran.
class CarteActuelle extends StatelessWidget {
  const CarteActuelle({super.key, required this.bulletin});

  final BulletinMeteo bulletin;

  @override
  Widget build(BuildContext context) {
    final actuelle = bulletin.actuelle;
    final jour = bulletin.aujourdHui;
    final unite = bulletin.symboleTemperature;

    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 8),
      child: Column(
        children: [
          IconeMeteo(
            code: actuelle.codeMeteo,
            estJour: actuelle.estJour,
            taille: 96,
          ),
          const SizedBox(height: 12),
          Text(
            // `.round()` et non `.toStringAsFixed(1)` : personne ne veut
            // lire « 26.4 °C » sur l'écran principal d'une météo.
            '${actuelle.temperature.round()} $unite',
            style: const TextStyle(
              color: Colors.white,
              fontSize: 72,
              fontWeight: FontWeight.w200,
              height: 1,
            ),
          ),
          const SizedBox(height: 8),
          Text(
            libelleWmo(actuelle.codeMeteo),
            style: const TextStyle(
              color: Colors.white,
              fontSize: 20,
              fontWeight: FontWeight.w500,
            ),
          ),
          const SizedBox(height: 4),
          Text(
            'Ressenti ${actuelle.ressenti.round()} $unite',
            style: const TextStyle(color: Colors.white70, fontSize: 15),
          ),
          if (jour != null) ...[
            const SizedBox(height: 16),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                _Extreme(
                  icone: Icons.arrow_upward_rounded,
                  valeur: '${jour.temperatureMax.round()} $unite',
                ),
                const SizedBox(width: 28),
                _Extreme(
                  icone: Icons.arrow_downward_rounded,
                  valeur: '${jour.temperatureMin.round()} $unite',
                ),
              ],
            ),
          ],
          const SizedBox(height: 20),
          const Divider(color: Colors.white24, height: 1),
          const SizedBox(height: 16),
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceEvenly,
            children: [
              _Indicateur(
                icone: Icons.water_drop_outlined,
                titre: 'Humidité',
                valeur: '${actuelle.humidite} %',
              ),
              _Indicateur(
                icone: Icons.air_rounded,
                titre: 'Vent',
                valeur: '${actuelle.vitesseVent.round()} '
                    '${bulletin.symboleVent}',
              ),
              _Indicateur(
                icone: Icons.umbrella_outlined,
                titre: 'Précipitations',
                // Une décimale ici : 0.2 mm et 0 mm ne veulent pas dire
                // la même chose.
                valeur: '${actuelle.precipitation.toStringAsFixed(1)} mm',
              ),
            ],
          ),
        ],
      ),
    );
  }
}

/// Un extrême de la journée : une flèche et une valeur.
class _Extreme extends StatelessWidget {
  const _Extreme({required this.icone, required this.valeur});

  final IconData icone;
  final String valeur;

  @override
  Widget build(BuildContext context) {
    return Row(
      mainAxisSize: MainAxisSize.min,
      children: [
        Icon(icone, color: Colors.white70, size: 18),
        const SizedBox(width: 4),
        Text(valeur,
            style: const TextStyle(color: Colors.white, fontSize: 17)),
      ],
    );
  }
}

/// Une colonne icône / valeur / libellé.
class _Indicateur extends StatelessWidget {
  const _Indicateur({
    required this.icone,
    required this.titre,
    required this.valeur,
  });

  final IconData icone;
  final String titre;
  final String valeur;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Icon(icone, color: Colors.white70, size: 22),
        const SizedBox(height: 6),
        Text(
          valeur,
          style: const TextStyle(
              color: Colors.white, fontSize: 16, fontWeight: FontWeight.w600),
        ),
        const SizedBox(height: 2),
        Text(titre,
            style: const TextStyle(color: Colors.white60, fontSize: 12)),
      ],
    );
  }
}
```

Notez la syntaxe `if (jour != null) ...[ ... ]` : c'est l'élément conditionnel du chapitre 06, combiné à l'opérateur d'étalement. Il insère zéro ou deux enfants dans la `Column` selon la condition. Sans lui, on écrirait un ternaire renvoyant un `SizedBox.shrink()`, bien moins lisible.

---

## 61.14 — Les dates en français avec `intl`

Par défaut, Flutter formate les dates en anglais : « Sunday 16 August ». Le paquet `intl` corrige cela, mais il exige **deux initialisations distinctes** que l'on confond souvent :

| Initialisation | Ce qu'elle règle | Où l'appeler |
| --- | --- | --- |
| `initializeDateFormatting('fr_FR', null)` | les données de langue de `DateFormat` | dans `main`, avant `runApp` |
| `localizationsDelegates` + `supportedLocales` | la langue des **widgets** Material | dans `MaterialApp` |

Oublier la première produit `LocaleDataException: Locale data has not been initialized`. Oublier la seconde laisse les boîtes de dialogue en anglais.

```dart
// lib/utilitaires/dates.dart
import 'package:intl/intl.dart';

const String _locale = 'fr_FR';

/// « 14:05 » — heure d'une mesure.
String heureCourte(DateTime d) => DateFormat('HH:mm', _locale).format(d);

/// « 15 h » — libellé d'une case de la bande horaire.
String heureBande(DateTime d) => '${DateFormat('HH', _locale).format(d)} h';

/// « dimanche » — nom du jour, en minuscules comme le veut le français.
String jourSemaine(DateTime d) => DateFormat('EEEE', _locale).format(d);

/// « 16 août » — jour et mois, sans l'année.
String jourEtMois(DateTime d) => DateFormat('d MMMM', _locale).format(d);

/// « samedi 15 août à 14:05 » — pour le bandeau hors-ligne.
String dateEtHeure(DateTime d) =>
    '${DateFormat('EEEE d MMMM', _locale).format(d)} à ${heureCourte(d)}';

/// Libellé d'une ligne de prévision journalière.
///
/// [reference] est la date du jour DANS LA VILLE consultée, pas sur
/// l'appareil : à Tokyo, « aujourd'hui » n'est pas le même jour qu'à Paris.
String libelleJour(DateTime jour, DateTime reference) {
  final ecart = DateTime(jour.year, jour.month, jour.day)
      .difference(DateTime(reference.year, reference.month, reference.day))
      .inDays;
  if (ecart == 0) return "Aujourd'hui";
  if (ecart == 1) return 'Demain';
  return jourSemaine(jour);
}

/// Met la première lettre en capitale : « dimanche » → « Dimanche ».
/// `intl` renvoie les jours en minuscules ; c'est correct au milieu d'une
/// phrase, mais laid en début de ligne.
String capitaliser(String texte) =>
    texte.isEmpty ? texte : texte[0].toUpperCase() + texte.substring(1);
```

Et le `main.dart` mis à jour :

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter_localizations/flutter_localizations.dart';
import 'package:intl/date_symbol_data_local.dart';
import 'package:intl/intl.dart';

import 'ecrans/ecran_meteo.dart';

Future<void> main() async {
  // Obligatoire : on effectue un travail asynchrone avant runApp.
  WidgetsFlutterBinding.ensureInitialized();

  // 1) Les données de langue de DateFormat.
  await initializeDateFormatting('fr_FR', null);
  // 2) La locale par défaut, pour ne pas la répéter partout.
  Intl.defaultLocale = 'fr_FR';

  runApp(const AppliMeteo());
}

class AppliMeteo extends StatelessWidget {
  const AppliMeteo({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Météo',
      debugShowCheckedModeBanner: false,
      // 3) La langue des widgets Material eux-mêmes.
      locale: const Locale('fr', 'FR'),
      localizationsDelegates: const [
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate,
        GlobalCupertinoLocalizations.delegate,
      ],
      supportedLocales: const [Locale('fr', 'FR'), Locale('en', 'US')],
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF1E88E5)),
        useMaterial3: true,
      ),
      home: const EcranMeteo(),
    );
  }
}
```

`flutter_localizations` fait partie du SDK Flutter ; il ne s'installe pas avec `pub add`, il se déclare dans `pubspec.yaml` :

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter
```

Puis :

```text
flutter pub get
```

---

## 61.15 — Les prévisions horaires en `ListView` horizontale

Une `ListView` horizontale est une `ListView` ordinaire à laquelle on donne `scrollDirection: Axis.horizontal`. Deux règles s'imposent alors (chapitre 48) :

1. il faut lui donner une **hauteur** explicite, sinon elle tente de prendre une hauteur infinie ;
2. chaque enfant doit avoir une **largeur** fixe, sinon il tente de prendre une largeur infinie.

```dart
// lib/widgets/bande_horaire.dart
import 'package:flutter/material.dart';

import '../modeles/bulletin_meteo.dart';
import '../modeles/prevision_heure.dart';
import '../utilitaires/dates.dart';
import 'icone_meteo.dart';

/// La bande défilante des 24 prochaines heures.
class BandeHoraire extends StatelessWidget {
  const BandeHoraire({super.key, required this.bulletin});

  final BulletinMeteo bulletin;

  @override
  Widget build(BuildContext context) {
    final heures = bulletin.prochainesHeures;
    if (heures.isEmpty) return const SizedBox.shrink();

    // L'heure en cours dans la ville : sert à mettre la première case
    // en évidence.
    final maintenant = bulletin.maintenantLaBas;

    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        const Padding(
          padding: EdgeInsets.fromLTRB(20, 8, 20, 10),
          child: Text(
            'PROCHAINES HEURES',
            style: TextStyle(
              color: Colors.white70,
              fontSize: 12,
              letterSpacing: 1.2,
              fontWeight: FontWeight.w600,
            ),
          ),
        ),
        SizedBox(
          // Hauteur imposée : sans elle, « Vertical viewport was given
          // unbounded height » ou son équivalent horizontal.
          height: 128,
          child: ListView.builder(
            scrollDirection: Axis.horizontal,
            padding: const EdgeInsets.symmetric(horizontal: 14),
            itemCount: heures.length,
            itemBuilder: (context, i) {
              final h = heures[i];
              final estHeureCourante = h.heure.hour == maintenant.hour &&
                  h.heure.day == maintenant.day;
              return _CaseHoraire(
                heure: h,
                estJour: _faitJour(h),
                unite: bulletin.symboleTemperature,
                enEvidence: estHeureCourante,
              );
            },
          ),
        ),
      ],
    );
  }

  /// Détermine s'il fait jour à ce créneau, en comparant au lever et au
  /// coucher du soleil du jour correspondant. À défaut d'information, on
  /// retombe sur une règle grossière : entre 7 h et 20 h.
  bool _faitJour(PrevisionHeure h) {
    for (final j in bulletin.jours) {
      if (j.jour.day == h.heure.day && j.jour.month == h.heure.month) {
        final lever = j.leverSoleil;
        final coucher = j.coucherSoleil;
        if (lever == null || coucher == null) break;
        return h.heure.isAfter(lever) && h.heure.isBefore(coucher);
      }
    }
    return h.heure.hour >= 7 && h.heure.hour < 20;
  }
}

/// Une case de la bande : heure, icône, température, probabilité de pluie.
class _CaseHoraire extends StatelessWidget {
  const _CaseHoraire({
    required this.heure,
    required this.estJour,
    required this.unite,
    required this.enEvidence,
  });

  final PrevisionHeure heure;
  final bool estJour;
  final String unite;
  final bool enEvidence;

  @override
  Widget build(BuildContext context) {
    return Container(
      // Largeur imposée : obligatoire dans une liste horizontale.
      width: 68,
      margin: const EdgeInsets.symmetric(horizontal: 4, vertical: 6),
      decoration: BoxDecoration(
        color: enEvidence ? Colors.white24 : Colors.white10,
        borderRadius: BorderRadius.circular(18),
        border: enEvidence
            ? Border.all(color: Colors.white54, width: 1)
            : null,
      ),
      child: Column(
        mainAxisAlignment: MainAxisAlignment.spaceEvenly,
        children: [
          Text(
            enEvidence ? 'Mtnt' : heureBande(heure.heure),
            style: const TextStyle(color: Colors.white70, fontSize: 13),
          ),
          IconeMeteo(
            code: heure.codeMeteo,
            estJour: estJour,
            taille: 26,
          ),
          Text(
            '${heure.temperature.round()}$unite',
            style: const TextStyle(
              color: Colors.white,
              fontSize: 16,
              fontWeight: FontWeight.w600,
            ),
          ),
          Text(
            // Le champ peut être nul : on affiche un tiret cadratin
            // plutôt qu'un « 0 % » mensonger.
            heure.probabilitePluie == null
                ? '—'
                : '${heure.probabilitePluie} %',
            style: TextStyle(
              color: (heure.probabilitePluie ?? 0) >= 40
                  ? const Color(0xFF8ED0FF)
                  : Colors.white54,
              fontSize: 12,
            ),
          ),
        ],
      ),
    );
  }
}
```

**Pourquoi filtrer les heures passées ?** Parce que le bloc `hourly` commence toujours à `00:00` du premier jour. À 14 h, les quatorze premières cases seraient du passé. C'est exactement le rôle du getter `prochainesHeures` écrit à la section 61.7 — allez le relire, il est plus subtil qu'il n'en a l'air.

---

## 61.16 — Les prévisions sur sept jours

Chaque ligne comporte un libellé de jour, une icône, une probabilité de pluie, et une **barre de températures** qui situe le minimum et le maximum du jour dans l'amplitude de la semaine.

```text
                     min de la semaine        max de la semaine
                            │                        │
Aujourd'hui  ☀   0%    17° ─┼──────▓▓▓▓▓▓▓▓▓▓▓───────┼─ 29°
lundi        ☂  70%    14° ─┼──▓▓▓▓▓▓▓▓───────────────┼─ 22°
```

```dart
// lib/widgets/liste_jours.dart
import 'package:flutter/material.dart';

import '../modeles/bulletin_meteo.dart';
import '../modeles/prevision_jour.dart';
import '../utilitaires/dates.dart';
import 'icone_meteo.dart';

/// La liste des sept prochains jours.
class ListeJours extends StatelessWidget {
  const ListeJours({super.key, required this.bulletin});

  final BulletinMeteo bulletin;

  @override
  Widget build(BuildContext context) {
    final jours = bulletin.jours;
    if (jours.isEmpty) return const SizedBox.shrink();

    // Amplitude de toute la semaine : c'est l'échelle commune des barres.
    // `fold` (chapitre 14) évite d'écrire deux boucles.
    final minSemaine =
        jours.fold<double>(double.infinity, (m, j) => j.temperatureMin < m ? j.temperatureMin : m);
    final maxSemaine =
        jours.fold<double>(double.negativeInfinity, (m, j) => j.temperatureMax > m ? j.temperatureMax : m);
    // Garde-fou : une amplitude nulle produirait une division par zéro.
    final amplitude = (maxSemaine - minSemaine).abs() < 0.5
        ? 1.0
        : maxSemaine - minSemaine;

    final reference = bulletin.maintenantLaBas;

    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        const Padding(
          padding: EdgeInsets.fromLTRB(20, 16, 20, 10),
          child: Text(
            'PROCHAINS JOURS',
            style: TextStyle(
              color: Colors.white70,
              fontSize: 12,
              letterSpacing: 1.2,
              fontWeight: FontWeight.w600,
            ),
          ),
        ),
        // `shrinkWrap` + `NeverScrollableScrollPhysics` : cette liste est
        // à l'intérieur d'un défilement vertical parent. Sans ces deux
        // réglages, on obtiendrait deux défilements imbriqués et une
        // erreur de contrainte.
        ListView.separated(
          shrinkWrap: true,
          physics: const NeverScrollableScrollPhysics(),
          padding: const EdgeInsets.symmetric(horizontal: 12),
          itemCount: jours.length,
          separatorBuilder: (_, __) =>
              const Divider(color: Colors.white12, height: 1, indent: 16),
          itemBuilder: (context, i) => _LigneJour(
            jour: jours[i],
            reference: reference,
            minSemaine: minSemaine,
            amplitude: amplitude,
            unite: bulletin.symboleTemperature,
          ),
        ),
      ],
    );
  }
}

class _LigneJour extends StatelessWidget {
  const _LigneJour({
    required this.jour,
    required this.reference,
    required this.minSemaine,
    required this.amplitude,
    required this.unite,
  });

  final PrevisionJour jour;
  final DateTime reference;
  final double minSemaine;
  final double amplitude;
  final String unite;

  @override
  Widget build(BuildContext context) {
    // Position et longueur de la barre, exprimées en fractions de 0 à 1.
    final debut = (jour.temperatureMin - minSemaine) / amplitude;
    final fin = (jour.temperatureMax - minSemaine) / amplitude;

    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 10),
      child: Row(
        children: [
          SizedBox(
            width: 96,
            child: Text(
              capitaliser(libelleJour(jour.jour, reference)),
              style: const TextStyle(color: Colors.white, fontSize: 15),
              overflow: TextOverflow.ellipsis,
            ),
          ),
          IconeMeteo(code: jour.codeMeteo, estJour: true, taille: 22),
          SizedBox(
            width: 42,
            child: Text(
              jour.probabilitePluieMax == null
                  ? ''
                  : '${jour.probabilitePluieMax} %',
              textAlign: TextAlign.right,
              style: const TextStyle(color: Color(0xFF8ED0FF), fontSize: 12),
            ),
          ),
          const SizedBox(width: 10),
          SizedBox(
            width: 34,
            child: Text(
              '${jour.temperatureMin.round()}$unite',
              textAlign: TextAlign.right,
              style: const TextStyle(color: Colors.white60, fontSize: 14),
            ),
          ),
          const SizedBox(width: 8),
          Expanded(
            child: _BarreTemperature(debut: debut, fin: fin),
          ),
          const SizedBox(width: 8),
          SizedBox(
            width: 34,
            child: Text(
              '${jour.temperatureMax.round()}$unite',
              style: const TextStyle(
                color: Colors.white,
                fontSize: 14,
                fontWeight: FontWeight.w600,
              ),
            ),
          ),
        ],
      ),
    );
  }
}

/// La barre colorée d'une ligne de jour.
///
/// [debut] et [fin] sont des fractions de 0 à 1 dans l'amplitude de la
/// semaine. `LayoutBuilder` (chapitre 51) donne la largeur réellement
/// disponible, seule façon de convertir une fraction en pixels.
class _BarreTemperature extends StatelessWidget {
  const _BarreTemperature({required this.debut, required this.fin});

  final double debut;
  final double fin;

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, contraintes) {
        final largeur = contraintes.maxWidth;
        final gauche = (debut.clamp(0.0, 1.0)) * largeur;
        // Largeur minimale de 6 px : une journée sans amplitude doit
        // rester visible.
        final longueur =
            ((fin - debut).clamp(0.0, 1.0) * largeur).clamp(6.0, largeur);

        return SizedBox(
          height: 6,
          child: Stack(
            children: [
              // La piste grise, sur toute la largeur.
              Container(
                decoration: BoxDecoration(
                  color: Colors.white24,
                  borderRadius: BorderRadius.circular(3),
                ),
              ),
              // Le segment coloré, positionné à sa place.
              Positioned(
                left: gauche,
                width: longueur,
                top: 0,
                bottom: 0,
                child: Container(
                  decoration: BoxDecoration(
                    borderRadius: BorderRadius.circular(3),
                    gradient: const LinearGradient(
                      colors: [Color(0xFF7FD4FF), Color(0xFFFFC46B)],
                    ),
                  ),
                ),
              ),
            ],
          ),
        );
      },
    );
  }
}
```

---

## 61.17 — La barre de recherche avec débounce

### Le problème

Un `TextField` déclenche `onChanged` à **chaque frappe**. Taper « bordeaux » lancerait huit requêtes réseau, dont sept inutiles, et les réponses pourraient revenir dans le désordre — un grand classique : la réponse de « bord » arrive après celle de « bordeaux » et écrase le bon résultat.

Le **débounce** consiste à attendre une pause dans la saisie avant d'agir.

```text
frappe  b---o---r---d---e---a---u---x-------------------→
                                          │
                             pause de 350 ms écoulée
                                          │
requête                                   └──▶ une seule
```

L'outil est le `Timer` de `dart:async` : à chaque frappe, on annule le minuteur précédent et on en arme un nouveau. Seul le dernier survit.

```dart
// lib/ecrans/ecran_recherche.dart
import 'dart:async';

import 'package:flutter/material.dart';

import '../modeles/ville.dart';
import '../services/erreur_meteo.dart';
import '../services/geocodage_service.dart';

/// Écran de recherche de ville.
///
/// Il ne connaît ni `provider` ni la persistance : tout lui est fourni par
/// son constructeur. Il renvoie la [Ville] choisie via `Navigator.pop`.
class EcranRecherche extends StatefulWidget {
  const EcranRecherche({
    super.key,
    this.favoris = const <Ville>[],
    this.historique = const <Ville>[],
    this.onBasculerFavori,
    this.onEffacerHistorique,
  });

  final List<Ville> favoris;
  final List<Ville> historique;
  final void Function(Ville)? onBasculerFavori;
  final VoidCallback? onEffacerHistorique;

  @override
  State<EcranRecherche> createState() => _EcranRechercheState();
}

class _EcranRechercheState extends State<EcranRecherche> {
  final TextEditingController _controleur = TextEditingController();
  final GeocodageService _service = GeocodageService();

  /// Le minuteur de débounce. Nullable : il n'existe qu'entre deux frappes.
  Timer? _minuteur;

  /// Compteur de requêtes : il permet d'ignorer une réponse tardive.
  int _numeroRequete = 0;

  List<Ville> _resultats = const [];
  bool _enCours = false;
  ErreurMeteo? _erreur;

  static const Duration _pause = Duration(milliseconds: 350);

  @override
  void dispose() {
    // Ordre important : on annule le minuteur AVANT de libérer le reste,
    // sinon il pourrait se déclencher sur un State détruit.
    _minuteur?.cancel();
    _controleur.dispose();
    _service.fermer();
    super.dispose();
  }

  void _surSaisie(String texte) {
    // On annule la frappe précédente : c'est tout le débounce.
    _minuteur?.cancel();

    if (texte.trim().length < 2) {
      setState(() {
        _resultats = const [];
        _erreur = null;
        _enCours = false;
      });
      return;
    }

    setState(() => _enCours = true);
    _minuteur = Timer(_pause, () => _chercher(texte));
  }

  Future<void> _chercher(String texte) async {
    final mien = ++_numeroRequete;
    try {
      final villes = await _service.chercherVilles(texte);
      // Une réponse plus ancienne que la dernière requête lancée est
      // périmée : on la jette. Sans ce test, « bord » écraserait
      // « bordeaux » s'il revenait en second.
      if (!mounted || mien != _numeroRequete) return;
      setState(() {
        _resultats = villes;
        _erreur = null;
        _enCours = false;
      });
    } on ErreurMeteo catch (e) {
      if (!mounted || mien != _numeroRequete) return;
      setState(() {
        _erreur = e;
        _resultats = const [];
        _enCours = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    final saisieVide = _controleur.text.trim().isEmpty;

    return Scaffold(
      appBar: AppBar(
        title: TextField(
          controller: _controleur,
          autofocus: true,
          textInputAction: TextInputAction.search,
          decoration: InputDecoration(
            hintText: 'Nom de ville ou code postal',
            border: InputBorder.none,
            prefixIcon: const Icon(Icons.search_rounded),
            suffixIcon: saisieVide
                ? null
                : IconButton(
                    icon: const Icon(Icons.close_rounded),
                    onPressed: () {
                      _controleur.clear();
                      _surSaisie('');
                      setState(() {});
                    },
                  ),
          ),
          onChanged: (t) {
            _surSaisie(t);
            // Un second setState pour rafraîchir la croix d'effacement.
            setState(() {});
          },
        ),
      ),
      body: _corps(saisieVide),
    );
  }

  Widget _corps(bool saisieVide) {
    if (saisieVide) return _listesMemorisees();

    if (_enCours) {
      return const Center(child: CircularProgressIndicator());
    }

    if (_erreur != null) {
      return Center(
        child: Padding(
          padding: const EdgeInsets.all(28),
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              const Icon(Icons.cloud_off_rounded, size: 48),
              const SizedBox(height: 12),
              Text(_erreur!.titre, textAlign: TextAlign.center),
              const SizedBox(height: 6),
              Text(
                _erreur!.detail,
                textAlign: TextAlign.center,
                style: Theme.of(context).textTheme.bodySmall,
              ),
            ],
          ),
        ),
      );
    }

    if (_resultats.isEmpty) {
      return const Center(child: Text('Aucune ville ne correspond.'));
    }

    return ListView.separated(
      itemCount: _resultats.length,
      separatorBuilder: (_, __) => const Divider(height: 1),
      itemBuilder: (context, i) {
        final ville = _resultats[i];
        return ListTile(
          leading: const Icon(Icons.location_city_rounded),
          title: Text(ville.nom),
          subtitle: Text(ville.sousTitre),
          // On renvoie la ville choisie à l'écran appelant (chapitre 50).
          onTap: () => Navigator.pop<Ville>(context, ville),
        );
      },
    );
  }

  /// Ce qu'on affiche quand le champ est vide : favoris et historique.
  Widget _listesMemorisees() {
    if (widget.favoris.isEmpty && widget.historique.isEmpty) {
      return const Center(
        child: Padding(
          padding: EdgeInsets.all(32),
          child: Text(
            'Tapez au moins deux lettres pour chercher une ville.',
            textAlign: TextAlign.center,
          ),
        ),
      );
    }

    return ListView(
      children: [
        if (widget.favoris.isNotEmpty) ...[
          const _EnTete(titre: 'FAVORIS'),
          for (final ville in widget.favoris)
            ListTile(
              leading: const Icon(Icons.favorite, color: Colors.redAccent),
              title: Text(ville.nom),
              subtitle: Text(ville.sousTitre),
              trailing: widget.onBasculerFavori == null
                  ? null
                  : IconButton(
                      icon: const Icon(Icons.close_rounded),
                      tooltip: 'Retirer des favoris',
                      onPressed: () => widget.onBasculerFavori!(ville),
                    ),
              onTap: () => Navigator.pop<Ville>(context, ville),
            ),
        ],
        if (widget.historique.isNotEmpty) ...[
          _EnTete(
            titre: 'RECHERCHES RÉCENTES',
            action: widget.onEffacerHistorique == null
                ? null
                : TextButton(
                    onPressed: widget.onEffacerHistorique,
                    child: const Text('Effacer'),
                  ),
          ),
          for (final ville in widget.historique)
            ListTile(
              leading: const Icon(Icons.history_rounded),
              title: Text(ville.nom),
              subtitle: Text(ville.sousTitre),
              onTap: () => Navigator.pop<Ville>(context, ville),
            ),
        ],
      ],
    );
  }
}

class _EnTete extends StatelessWidget {
  const _EnTete({required this.titre, this.action});

  final String titre;
  final Widget? action;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.fromLTRB(16, 18, 8, 4),
      child: Row(
        children: [
          Expanded(
            child: Text(
              titre,
              style: TextStyle(
                fontSize: 12,
                letterSpacing: 1.1,
                fontWeight: FontWeight.w600,
                color: Theme.of(context).colorScheme.primary,
              ),
            ),
          ),
          if (action != null) action!,
        ],
      ),
    );
  }
}
```

### Les deux protections, et pourquoi les deux sont nécessaires

| Protection | Ce qu'elle empêche |
| --- | --- |
| `Timer` de 350 ms | huit requêtes pour un mot de huit lettres |
| `_numeroRequete` | qu'une réponse lente écrase une réponse plus récente |

Le débounce seul ne suffit pas : si l'utilisateur marque une pause après « bord », puis complète en « bordeaux », deux requêtes partent bel et bien. Sur un réseau capricieux, la première peut revenir en dernier. Le compteur règle définitivement ce désordre.

### Brancher la recherche

Dans `ecran_meteo.dart`, on ajoute un bouton et on stocke la ville courante :

```dart
  Ville _ville = villeParDefaut;

  Future<void> _ouvrirRecherche() async {
    final choisie = await Navigator.push<Ville>(
      context,
      MaterialPageRoute(builder: (_) => const EcranRecherche()),
    );
    // `null` signifie « l'utilisateur a fait retour sans choisir ».
    if (choisie == null || !mounted) return;
    setState(() {
      _ville = choisie;
      _futur = _service.obtenirBulletin(_ville);
    });
  }
```

et, dans l'`AppBar` :

```dart
        title: Text(_ville.libelleCourt),
        actions: [
          IconButton(
            icon: const Icon(Icons.search_rounded),
            onPressed: _ouvrirRecherche,
          ),
        ],
```

**État exécutable n° 5 :** vous cherchez une ville, vous la choisissez, l'écran affiche sa météo. Tapez « bordeaux » lettre par lettre en observant les journaux : une seule requête part.

---

## 61.18 — Favoris, historique et persistance

Trois choses doivent survivre à la fermeture de l'application : les villes favorites, les recherches récentes, et l'unité de température. Toutes trois sont petites, plates et peu nombreuses : `shared_preferences` est le bon outil (chapitre 54, section 54.2).

```text
CLÉ                      TYPE STOCKÉ                CONTENU
meteo.favoris            String (JSON)              [ {ville}, {ville} ]
meteo.historique         String (JSON)              [ {ville}, … ] max 8
meteo.unite              String                     "celsius" | "fahrenheit"
meteo.derniere_ville     String (JSON)              { ville }
```

```dart
// lib/services/depot_preferences.dart
import 'dart:convert';

import 'package:shared_preferences/shared_preferences.dart';

import '../logique/unites.dart';
import '../modeles/ville.dart';

/// Lecture et écriture des préférences durables.
///
/// Toutes les méthodes sont asynchrones et ne lèvent jamais : une
/// préférence illisible est traitée comme une préférence absente. On ne
/// bloque jamais le démarrage de l'application pour un favori corrompu.
class DepotPreferences {
  static const String _cleFavoris = 'meteo.favoris';
  static const String _cleHistorique = 'meteo.historique';
  static const String _cleUnite = 'meteo.unite';
  static const String _cleDerniereVille = 'meteo.derniere_ville';

  /// Nombre maximal d'entrées conservées dans l'historique.
  static const int tailleHistorique = 8;

  Future<SharedPreferences> get _prefs => SharedPreferences.getInstance();

  // ---- Outils communs ------------------------------------------------

  Future<List<Ville>> _lireListe(String cle) async {
    final prefs = await _prefs;
    final brut = prefs.getString(cle);
    if (brut == null || brut.isEmpty) return <Ville>[];
    try {
      final decode = jsonDecode(brut);
      if (decode is! List) return <Ville>[];
      return decode
          .whereType<Map<String, dynamic>>()
          .map(Ville.fromJson)
          .toList();
    } on FormatException {
      // Donnée corrompue : on la jette et on repart de zéro.
      await prefs.remove(cle);
      return <Ville>[];
    }
  }

  Future<void> _ecrireListe(String cle, List<Ville> villes) async {
    final prefs = await _prefs;
    await prefs.setString(
      cle,
      jsonEncode(villes.map((v) => v.toJson()).toList()),
    );
  }

  // ---- Favoris -------------------------------------------------------

  Future<List<Ville>> lireFavoris() => _lireListe(_cleFavoris);

  Future<void> ecrireFavoris(List<Ville> favoris) =>
      _ecrireListe(_cleFavoris, favoris);

  // ---- Historique ----------------------------------------------------

  Future<List<Ville>> lireHistorique() => _lireListe(_cleHistorique);

  /// Ajoute [ville] en tête de l'historique, sans doublon, et tronque.
  ///
  /// La comparaison repose sur `Ville.operator ==`, redéfini à la
  /// section 61.3 : sans lui, `removeWhere` ne trouverait jamais rien.
  Future<List<Ville>> ajouterHistorique(Ville ville) async {
    final liste = await lireHistorique();
    liste.removeWhere((v) => v == ville);
    liste.insert(0, ville);
    final tronquee = liste.take(tailleHistorique).toList();
    await _ecrireListe(_cleHistorique, tronquee);
    return tronquee;
  }

  Future<void> effacerHistorique() async {
    final prefs = await _prefs;
    await prefs.remove(_cleHistorique);
  }

  // ---- Unité ---------------------------------------------------------

  Future<UniteTemperature> lireUnite() async {
    final prefs = await _prefs;
    // On persiste `.name` et jamais `.index` : ajouter une valeur à
    // l'enum décalerait tous les index déjà écrits sur le disque.
    return UniteTemperature.depuisNom(prefs.getString(_cleUnite));
  }

  Future<void> ecrireUnite(UniteTemperature unite) async {
    final prefs = await _prefs;
    await prefs.setString(_cleUnite, unite.name);
  }

  // ---- Dernière ville consultée --------------------------------------

  Future<Ville?> lireDerniereVille() async {
    final prefs = await _prefs;
    final brut = prefs.getString(_cleDerniereVille);
    if (brut == null || brut.isEmpty) return null;
    try {
      final decode = jsonDecode(brut);
      if (decode is! Map<String, dynamic>) return null;
      return Ville.fromJson(decode);
    } on FormatException {
      return null;
    }
  }

  Future<void> ecrireDerniereVille(Ville ville) async {
    final prefs = await _prefs;
    await prefs.setString(_cleDerniereVille, jsonEncode(ville.toJson()));
  }
}
```

**Pourquoi mémoriser la dernière ville consultée ?** Pour que l'application rouvre là où on l'a laissée. C'est un détail d'ergonomie à cinq lignes qui change complètement l'usage quotidien : sans lui, l'utilisateur retape sa ville chaque matin.

---

## 61.19 — Le cache hors-ligne

La stratégie retenue est celle de la section 54.36 : **réseau d'abord, cache ensuite**.

```text
        demande de bulletin
                │
                ▼
       ┌────────────────┐    succès    ┌───────────────────────┐
       │ appel réseau   ├─────────────▶│ afficher + mettre en  │
       └───────┬────────┘              │ cache le JSON brut    │
               │ échec                 └───────────────────────┘
               ▼
       ┌────────────────┐   présent    ┌───────────────────────┐
       │ lire le cache  ├─────────────▶│ afficher + bandeau    │
       └───────┬────────┘              │ « hors-ligne »        │
               │ absent                └───────────────────────┘
               ▼
        écran d'erreur
```

**Ce que l'on met en cache est le JSON brut, pas les objets Dart.** Deux raisons :

1. on n'a alors aucun `toJson` à écrire pour `BulletinMeteo`, `MeteoActuelle`, `PrevisionHeure` et `PrevisionJour` — quatre classes économisées ;
2. la relecture emprunte exactement le même chemin de décodage que le réseau, donc un seul code à tester et à maintenir.

```dart
// lib/services/cache_meteo.dart
import 'dart:convert';

import 'package:shared_preferences/shared_preferences.dart';

import '../modeles/bulletin_meteo.dart';
import '../modeles/ville.dart';

/// Conserve la dernière réponse brute de l'API pour l'affichage hors-ligne.
class CacheMeteo {
  static const String _cle = 'meteo.cache';

  /// Enregistre la réponse brute [corps] associée à [ville].
  ///
  /// On stocke aussi l'instant d'enregistrement : sans lui, impossible de
  /// dire à l'utilisateur de quand datent les données affichées.
  Future<void> enregistrer(Ville ville, Map<String, dynamic> corps) async {
    final prefs = await SharedPreferences.getInstance();
    final paquet = <String, dynamic>{
      'ville': ville.toJson(),
      'reponse': corps,
      'enregistre_le': DateTime.now().toIso8601String(),
    };
    await prefs.setString(_cle, jsonEncode(paquet));
  }

  /// Relit le dernier bulletin enregistré, ou `null`.
  ///
  /// Si [ville] est fourni, on ne renvoie le cache que s'il concerne
  /// cette ville : afficher la météo de Paris sous le titre « Tokyo »
  /// serait pire que ne rien afficher.
  Future<BulletinMeteo?> lire({Ville? ville}) async {
    final prefs = await SharedPreferences.getInstance();
    final brut = prefs.getString(_cle);
    if (brut == null || brut.isEmpty) return null;

    try {
      final paquet = jsonDecode(brut);
      if (paquet is! Map<String, dynamic>) return null;

      final villeJson = paquet['ville'];
      final reponse = paquet['reponse'];
      if (villeJson is! Map<String, dynamic> ||
          reponse is! Map<String, dynamic>) {
        return null;
      }

      final villeCache = Ville.fromJson(villeJson);
      if (ville != null && villeCache != ville) return null;

      final quand = DateTime.tryParse(paquet['enregistre_le'] as String? ?? '');

      return BulletinMeteo.fromJson(
        reponse,
        ville: villeCache,
        recupereLe: quand,
      );
    } on FormatException {
      await prefs.remove(_cle);
      return null;
    } on TypeError {
      await prefs.remove(_cle);
      return null;
    }
  }

  Future<void> vider() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.remove(_cle);
  }
}
```

Et le bandeau d'avertissement, sans lequel l'utilisateur croirait consulter des données fraîches :

```dart
// lib/widgets/bandeau_hors_ligne.dart
import 'package:flutter/material.dart';

import '../utilitaires/dates.dart';

/// Bandeau affiché au-dessus du contenu quand les données viennent du cache.
class BandeauHorsLigne extends StatelessWidget {
  const BandeauHorsLigne({super.key, required this.recupereLe});

  final DateTime recupereLe;

  @override
  Widget build(BuildContext context) {
    return Container(
      width: double.infinity,
      color: const Color(0xFFB26A00),
      padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 10),
      child: Row(
        children: [
          const Icon(Icons.cloud_off_rounded, color: Colors.white, size: 18),
          const SizedBox(width: 10),
          Expanded(
            child: Text(
              'Hors-ligne — données du ${dateEtHeure(recupereLe)}',
              style: const TextStyle(color: Colors.white, fontSize: 13),
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## 61.20 — Le changement d'unité °C / °F

Le fichier `logique/unites.dart` existe depuis la section 61.8, et `MeteoService.construireUri` sait déjà passer `temperature_unit`. Il ne reste que deux choses à décider.

**Première décision : convertir ou redemander ?** On redemande. Changer d'unité relance une requête, et l'API renvoie des valeurs déjà arrondies, cohérentes entre température, ressenti, extrêmes et prévisions. Convertir localement obligerait à traiter aussi le vent (`km/h` → `mp/h`) et les précipitations (`mm` → `inch`), avec un risque d'incohérence à chaque oubli. Une requête coûte quelques dizaines de kilo-octets ; le bogue coûte plus cher.

**Seconde décision : où afficher la bascule ?** Dans la barre d'application, sous forme d'un bouton texte qui affiche l'unité **cible**, pas l'unité courante.

```dart
// Extrait à placer dans les `actions` de l'AppBar (voir 61.22).
TextButton(
  onPressed: () => etat.changerUnite(etat.unite.inverse),
  child: Text(
    // On affiche « °F » quand on est en Celsius : le bouton annonce
    // ce qu'il va faire, pas ce qui est déjà affiché.
    etat.unite.inverse.symbole,
    style: const TextStyle(
      color: Colors.white,
      fontSize: 16,
      fontWeight: FontWeight.w600,
    ),
  ),
)
```

Le choix est persisté par `DepotPreferences.ecrireUnite` : au prochain démarrage, l'application se souvient. C'est un réglage, pas un caprice de session.

---

## 61.21 — Centraliser l'état avec `provider`

### Pourquoi maintenant

L'écran principal détient aujourd'hui la ville, le `Future`, le service. L'écran de recherche doit connaître les favoris et l'historique. Le bouton d'unité doit relancer la requête. Sans centralisation, il faudrait faire remonter et redescendre cinq valeurs à travers deux écrans : c'est exactement le problème décrit au chapitre 52.

Un seul `ChangeNotifier` détient tout, et les deux écrans le lisent.

```text
                    ┌──────────────────────────┐
                    │        EtatMeteo         │
                    │  ville, bulletin, statut │
                    │  favoris, historique     │
                    │  unite, horsLigne        │
                    └────────┬─────────────────┘
             context.watch   │   context.read
          ┌──────────────────┴───────────────────┐
          ▼                                      ▼
    EcranMeteo                            EcranRecherche
```

### Une méthode à ajouter à `MeteoService`

L'état va télécharger le JSON, le mettre en cache, **puis** le décoder. Il faut donc exposer le décodage séparément :

```dart
  /// Décode un corps JSON déjà obtenu, qu'il vienne du réseau ou du cache.
  ///
  /// Convertit les échecs de structure en [ErreurMeteo] : c'est le rôle de
  /// la couche service, pas celui des modèles.
  BulletinMeteo decoder(
    Map<String, dynamic> corps,
    Ville ville, {
    DateTime? recupereLe,
  }) {
    try {
      return BulletinMeteo.fromJson(
        corps,
        ville: ville,
        recupereLe: recupereLe,
      );
    } on FormatException catch (e) {
      throw ErreurMeteo(CauseErreur.donnees, e.message);
    } on TypeError catch (e) {
      throw ErreurMeteo(CauseErreur.donnees, '$e');
    }
  }
```

`obtenirBulletin` peut alors se réduire à une ligne :

```dart
  Future<BulletinMeteo> obtenirBulletin(
    Ville ville, {
    UniteTemperature unite = UniteTemperature.celsius,
  }) async =>
      decoder(await telechargerJson(ville, unite: unite), ville);
```

### Le `ChangeNotifier`

```dart
// lib/etat/etat_meteo.dart
import 'package:flutter/foundation.dart';

import '../logique/unites.dart';
import '../modeles/bulletin_meteo.dart';
import '../modeles/ville.dart';
import '../services/cache_meteo.dart';
import '../services/depot_preferences.dart';
import '../services/erreur_meteo.dart';
import '../services/meteo_service.dart';

/// Les états possibles de l'écran principal.
enum StatutMeteo { initial, chargement, succes, erreur }

/// Tout l'état applicatif, en un seul objet observable.
class EtatMeteo extends ChangeNotifier {
  EtatMeteo({
    MeteoService? meteo,
    DepotPreferences? preferences,
    CacheMeteo? cache,
  })  : _meteo = meteo ?? MeteoService(),
        _preferences = preferences ?? DepotPreferences(),
        _cache = cache ?? CacheMeteo();

  final MeteoService _meteo;
  final DepotPreferences _preferences;
  final CacheMeteo _cache;

  // ---- État, exposé en lecture seule ---------------------------------

  StatutMeteo _statut = StatutMeteo.initial;
  StatutMeteo get statut => _statut;

  Ville? _ville;
  Ville? get ville => _ville;

  BulletinMeteo? _bulletin;
  BulletinMeteo? get bulletin => _bulletin;

  ErreurMeteo? _erreur;
  ErreurMeteo? get erreur => _erreur;

  /// Vrai quand le bulletin affiché vient du cache et non du réseau.
  bool _horsLigne = false;
  bool get horsLigne => _horsLigne;

  UniteTemperature _unite = UniteTemperature.celsius;
  UniteTemperature get unite => _unite;

  List<Ville> _favoris = const [];
  /// Copie non modifiable : l'interface ne doit pas pouvoir muter l'état
  /// sans passer par une méthode qui notifie.
  List<Ville> get favoris => List.unmodifiable(_favoris);

  List<Ville> _historique = const [];
  List<Ville> get historique => List.unmodifiable(_historique);

  bool estFavorite(Ville v) => _favoris.contains(v);

  // ---- Démarrage -----------------------------------------------------

  /// Charge les préférences puis le bulletin de la dernière ville
  /// consultée. À appeler une seule fois, depuis `main`.
  Future<void> demarrer() async {
    _unite = await _preferences.lireUnite();
    _favoris = await _preferences.lireFavoris();
    _historique = await _preferences.lireHistorique();

    final derniere = await _preferences.lireDerniereVille() ??
        (_favoris.isNotEmpty ? _favoris.first : null);

    if (derniere == null) {
      // Aucune ville connue : l'écran proposera d'en chercher une.
      _statut = StatutMeteo.initial;
      notifyListeners();
      return;
    }

    _ville = derniere;
    notifyListeners();
    await _charger();
  }

  // ---- Actions de l'utilisateur --------------------------------------

  Future<void> choisirVille(Ville nouvelle) async {
    _ville = nouvelle;
    _bulletin = null;
    _erreur = null;
    notifyListeners();

    await _preferences.ecrireDerniereVille(nouvelle);
    _historique = await _preferences.ajouterHistorique(nouvelle);

    await _charger();
  }

  /// Rechargement explicite, déclenché par le tirage vers le bas ou par
  /// le bouton « Réessayer ».
  Future<void> rafraichir() => _charger();

  Future<void> changerUnite(UniteTemperature nouvelle) async {
    if (nouvelle == _unite) return;
    _unite = nouvelle;
    notifyListeners();
    await _preferences.ecrireUnite(nouvelle);
    // On redemande les données à l'API dans la nouvelle unité.
    if (_ville != null) await _charger();
  }

  Future<void> basculerFavori(Ville v) async {
    final liste = List<Ville>.from(_favoris);
    if (liste.contains(v)) {
      liste.removeWhere((e) => e == v);
    } else {
      liste.add(v);
    }
    _favoris = liste;
    // Notifier AVANT d'écrire : l'interface répond instantanément,
    // l'écriture disque se fait ensuite (mise à jour optimiste).
    notifyListeners();
    await _preferences.ecrireFavoris(liste);
  }

  Future<void> effacerHistorique() async {
    _historique = const [];
    notifyListeners();
    await _preferences.effacerHistorique();
  }

  // ---- Le cœur : réseau d'abord, cache ensuite -----------------------

  Future<void> _charger() async {
    final ville = _ville;
    if (ville == null) return;

    _statut = StatutMeteo.chargement;
    _erreur = null;
    notifyListeners();

    try {
      final corps = await _meteo.telechargerJson(ville, unite: _unite);
      // On enregistre le JSON BRUT : la relecture empruntera exactement
      // le même décodeur.
      await _cache.enregistrer(ville, corps);
      _bulletin = _meteo.decoder(corps, ville);
      _horsLigne = false;
      _statut = StatutMeteo.succes;
    } on ErreurMeteo catch (e) {
      // Le réseau a échoué : dernière chance, le cache.
      final secours = await _cache.lire(ville: ville);
      if (secours != null) {
        _bulletin = secours;
        _horsLigne = true;
        _erreur = null;
        _statut = StatutMeteo.succes;
      } else {
        _erreur = e;
        _statut = StatutMeteo.erreur;
      }
    }
    notifyListeners();
  }

  @override
  void dispose() {
    _meteo.fermer();
    super.dispose();
  }
}
```

### Trois points à ne pas manquer

| Point | Explication |
| --- | --- |
| `List.unmodifiable` sur les accesseurs | un widget ne peut pas faire `etat.favoris.add(...)` en douce ; toute modification passe par une méthode qui notifie. |
| Le cache est lu **dans le `catch`** | l'utilisateur en ligne ne voit jamais de donnée périmée ; l'utilisateur hors-ligne voit quelque chose. |
| `notifyListeners()` avant l'écriture disque | l'interface répond en moins d'une image ; c'est la mise à jour optimiste du chapitre 58. |

### Le `main.dart` définitif

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter_localizations/flutter_localizations.dart';
import 'package:intl/date_symbol_data_local.dart';
import 'package:intl/intl.dart';
import 'package:provider/provider.dart';

import 'ecrans/ecran_meteo.dart';
import 'etat/etat_meteo.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await initializeDateFormatting('fr_FR', null);
  Intl.defaultLocale = 'fr_FR';

  runApp(
    // Le provider est AU-DESSUS de MaterialApp : ainsi tous les écrans
    // poussés par le Navigator y ont accès (rappel du chapitre 52).
    ChangeNotifierProvider<EtatMeteo>(
      // `..demarrer()` lance le chargement initial sans attendre :
      // l'interface s'affiche immédiatement en état « chargement ».
      create: (_) => EtatMeteo()..demarrer(),
      child: const AppliMeteo(),
    ),
  );
}

class AppliMeteo extends StatelessWidget {
  const AppliMeteo({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Météo',
      debugShowCheckedModeBanner: false,
      locale: const Locale('fr', 'FR'),
      localizationsDelegates: const [
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate,
        GlobalCupertinoLocalizations.delegate,
      ],
      supportedLocales: const [Locale('fr', 'FR'), Locale('en', 'US')],
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF1E88E5)),
        useMaterial3: true,
      ),
      home: const EcranMeteo(),
    );
  }
}
```

---

## 61.22 — L'écran final, avec `RefreshIndicator`

`RefreshIndicator` impose deux conditions, souvent oubliées :

1. son enfant doit être **défilant** (`ListView`, `CustomScrollView`…) ;
2. ce défilement doit avoir `physics: AlwaysScrollableScrollPhysics()`, sinon le geste ne part pas quand le contenu tient dans l'écran.

Voici l'écran principal dans sa version définitive. Il remplace intégralement celui de la section 61.11.

```dart
// lib/ecrans/ecran_meteo.dart — VERSION DÉFINITIVE
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../etat/etat_meteo.dart';
import '../modeles/bulletin_meteo.dart';
import '../modeles/ville.dart';
import '../utilitaires/dates.dart';
import '../widgets/bandeau_hors_ligne.dart';
import '../widgets/bande_horaire.dart';
import '../widgets/carte_actuelle.dart';
import '../widgets/etats_ecran.dart';
import '../widgets/fond_meteo.dart';
import '../widgets/liste_jours.dart';
import 'ecran_recherche.dart';

class EcranMeteo extends StatelessWidget {
  const EcranMeteo({super.key});

  /// Ouvre l'écran de recherche et applique le choix de l'utilisateur.
  Future<void> _ouvrirRecherche(BuildContext context) async {
    // `read` et non `watch` : nous sommes dans un rappel, pas dans build.
    final etat = context.read<EtatMeteo>();

    final choisie = await Navigator.push<Ville>(
      context,
      MaterialPageRoute(
        builder: (_) => EcranRecherche(
          favoris: etat.favoris,
          historique: etat.historique,
          onBasculerFavori: etat.basculerFavori,
          onEffacerHistorique: etat.effacerHistorique,
        ),
      ),
    );
    if (choisie == null) return;
    await etat.choisirVille(choisie);
  }

  @override
  Widget build(BuildContext context) {
    // `watch` : cet écran se reconstruit à chaque notifyListeners.
    final etat = context.watch<EtatMeteo>();
    final bulletin = etat.bulletin;

    // Le fond suit la météo affichée ; à défaut, un bleu neutre de nuit.
    final code = bulletin?.actuelle.codeMeteo ?? 0;
    final estJour = bulletin?.actuelle.estJour ?? true;
    final ville = etat.ville;

    return Scaffold(
      // Le dégradé passe derrière la barre d'application.
      extendBodyBehindAppBar: true,
      appBar: AppBar(
        backgroundColor: Colors.transparent,
        elevation: 0,
        foregroundColor: Colors.white,
        title: Text(
          ville?.libelleCourt ?? 'Météo',
          style: const TextStyle(fontSize: 18),
          overflow: TextOverflow.ellipsis,
        ),
        actions: [
          IconButton(
            icon: const Icon(Icons.search_rounded),
            tooltip: 'Chercher une ville',
            onPressed: () => _ouvrirRecherche(context),
          ),
          if (ville != null)
            IconButton(
              icon: Icon(
                etat.estFavorite(ville)
                    ? Icons.favorite
                    : Icons.favorite_border,
              ),
              tooltip: etat.estFavorite(ville)
                  ? 'Retirer des favoris'
                  : 'Ajouter aux favoris',
              onPressed: () => etat.basculerFavori(ville),
            ),
          TextButton(
            onPressed: () => etat.changerUnite(etat.unite.inverse),
            child: Text(
              etat.unite.inverse.symbole,
              style: const TextStyle(
                color: Colors.white,
                fontSize: 16,
                fontWeight: FontWeight.w600,
              ),
            ),
          ),
        ],
      ),
      body: FondMeteo(
        code: code,
        estJour: estJour,
        child: SafeArea(child: _corps(context, etat)),
      ),
    );
  }

  Widget _corps(BuildContext context, EtatMeteo etat) {
    switch (etat.statut) {
      case StatutMeteo.initial:
        return EtatAucuneVille(onChercher: () => _ouvrirRecherche(context));

      case StatutMeteo.chargement:
        // Si un bulletin est déjà affiché, on ne le remplace pas par un
        // rond qui tourne : on laisse l'ancien contenu et le
        // RefreshIndicator montre l'activité en cours.
        final ancien = etat.bulletin;
        if (ancien != null) return _contenu(context, etat, ancien);
        return const EtatChargement();

      case StatutMeteo.erreur:
        return EtatErreur(
          erreur: etat.erreur!,
          onReessayer: etat.rafraichir,
        );

      case StatutMeteo.succes:
        return _contenu(context, etat, etat.bulletin!);
    }
  }

  Widget _contenu(
    BuildContext context,
    EtatMeteo etat,
    BulletinMeteo bulletin,
  ) {
    return RefreshIndicator(
      onRefresh: etat.rafraichir,
      color: Colors.white,
      backgroundColor: Colors.black26,
      child: ListView(
        // Indispensable : sans cela, le tirage ne fonctionne pas quand
        // le contenu tient entièrement dans l'écran.
        physics: const AlwaysScrollableScrollPhysics(),
        padding: const EdgeInsets.only(bottom: 32),
        children: [
          if (etat.horsLigne)
            BandeauHorsLigne(recupereLe: bulletin.recupereLe),
          const SizedBox(height: 12),
          CarteActuelle(bulletin: bulletin),
          const SizedBox(height: 12),
          BandeHoraire(bulletin: bulletin),
          ListeJours(bulletin: bulletin),
          const SizedBox(height: 16),
          Center(
            child: Text(
              'Mis à jour à ${heureCourte(bulletin.recupereLe)}'
              '${bulletin.estPerimee ? ' (données anciennes)' : ''}',
              style: const TextStyle(color: Colors.white54, fontSize: 12),
            ),
          ),
        ],
      ),
    );
  }
}
```

**État exécutable n° 6 — l'application complète.** Vérifiez dans l'ordre :

1. Premier lancement : « Aucune ville sélectionnée » et le bouton de recherche.
2. Choisir Bordeaux : le fond, la carte, la bande horaire et les sept jours s'affichent.
3. Appuyer sur le cœur, tuer l'application, la relancer : Bordeaux revient tout seul, en favori.
4. Basculer sur °F : toutes les valeurs changent, y compris le vent.
5. Passer en mode avion, tirer vers le bas : le bandeau orange « Hors-ligne — données du … » apparaît, le contenu reste.
6. Effacer les données de l'application, repasser en mode avion, relancer : cette fois l'écran d'erreur s'affiche, car il n'y a plus de cache.

Le point 6 est le plus important : il prouve que les deux chemins de repli sont bien distincts.

---

## 61.23 — Tester le service avec un JSON factice

Un test qui appelle vraiment l'API serait lent, dépendant du réseau, et **non reproductible** : la température change toutes les quinze minutes. On remplace donc le client HTTP par un faux, qui renvoie une réponse figée.

Le paquet `http` fournit exactement l'outil : `MockClient`, dans `package:http/testing.dart`. Il est déjà installé avec `http` ; rien à ajouter au `pubspec.yaml`.

```dart
// test/meteo_service_test.dart
import 'dart:convert';

import 'package:appli_meteo/logique/unites.dart';
import 'package:appli_meteo/modeles/ville.dart';
import 'package:appli_meteo/services/erreur_meteo.dart';
import 'package:appli_meteo/services/meteo_service.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:http/http.dart' as http;
import 'package:http/testing.dart';

/// Une ville de test. Aucune requête ne partira réellement.
const villeTest = Ville(
  id: 2988507,
  nom: 'Paris',
  latitude: 48.85341,
  longitude: 2.3488,
  pays: 'France',
  region: 'Île-de-France',
);

/// Réponse factice, copiée sur la structure réelle de l'API.
/// Deux heures et deux jours suffisent à tout vérifier.
const String reponseFactice = '''
{
  "latitude": 48.86,
  "longitude": 2.3399997,
  "generationtime_ms": 0.3,
  "utc_offset_seconds": 7200,
  "timezone": "Europe/Paris",
  "timezone_abbreviation": "GMT+2",
  "elevation": 43.0,
  "current_units": {
    "time": "iso8601", "interval": "seconds", "temperature_2m": "°C",
    "relative_humidity_2m": "%", "apparent_temperature": "°C",
    "is_day": "", "precipitation": "mm", "weather_code": "wmo code",
    "wind_speed_10m": "km/h"
  },
  "current": {
    "time": "2026-08-15T14:00", "interval": 900, "temperature_2m": 26.4,
    "relative_humidity_2m": 48, "apparent_temperature": 26.9, "is_day": 1,
    "precipitation": 0.0, "weather_code": 2, "wind_speed_10m": 11.9
  },
  "hourly_units": {
    "time": "iso8601", "temperature_2m": "°C", "weather_code": "wmo code",
    "precipitation_probability": "%"
  },
  "hourly": {
    "time": ["2026-08-15T00:00", "2026-08-15T01:00", "2026-08-15T02:00"],
    "temperature_2m": [18.2, 17.8, 17.5],
    "weather_code": [1, 1, 2],
    "precipitation_probability": [0, null, 3]
  },
  "daily_units": {
    "time": "iso8601", "weather_code": "wmo code",
    "temperature_2m_max": "°C", "temperature_2m_min": "°C",
    "sunrise": "iso8601", "sunset": "iso8601",
    "precipitation_probability_max": "%"
  },
  "daily": {
    "time": ["2026-08-15", "2026-08-16"],
    "weather_code": [2, 61],
    "temperature_2m_max": [29.1, 26.3],
    "temperature_2m_min": [17.4, 16.2],
    "sunrise": ["2026-08-15T06:47", "2026-08-16T06:48"],
    "sunset": ["2026-08-15T21:03", "2026-08-16T21:01"],
    "precipitation_probability_max": [0, 70]
  }
}
''';

void main() {
  group('construireUri', () {
    final service = MeteoService();

    test('place les coordonnées et le fuseau automatique', () {
      final uri = service.construireUri(villeTest);
      expect(uri.host, 'api.open-meteo.com');
      expect(uri.path, '/v1/forecast');
      expect(uri.queryParameters['latitude'], '48.85341');
      expect(uri.queryParameters['longitude'], '2.3488');
      expect(uri.queryParameters['timezone'], 'auto');
      expect(uri.queryParameters['forecast_days'], '7');
    });

    test('demande les variables attendues', () {
      final uri = service.construireUri(villeTest);
      expect(uri.queryParameters['current'], contains('weather_code'));
      expect(uri.queryParameters['hourly'],
          contains('precipitation_probability'));
      expect(uri.queryParameters['daily'], contains('temperature_2m_max'));
    });

    test('traduit le choix d unité en paramètres cohérents', () {
      final uri = service.construireUri(
        villeTest,
        unite: UniteTemperature.fahrenheit,
      );
      expect(uri.queryParameters['temperature_unit'], 'fahrenheit');
      expect(uri.queryParameters['wind_speed_unit'], 'mph');
      expect(uri.queryParameters['precipitation_unit'], 'inch');
    });
  });

  group('obtenirBulletin sur une réponse valide', () {
    late MeteoService service;

    setUp(() {
      // Le faux client répond toujours la même chose, en UTF-8.
      final faux = MockClient((requete) async {
        return http.Response.bytes(
          utf8.encode(reponseFactice),
          200,
          headers: {'content-type': 'application/json; charset=utf-8'},
        );
      });
      service = MeteoService(client: faux);
    });

    test('décode le bloc current', () async {
      final b = await service.obtenirBulletin(villeTest);
      expect(b.actuelle.temperature, 26.4);
      expect(b.actuelle.ressenti, 26.9);
      expect(b.actuelle.humidite, 48);
      expect(b.actuelle.codeMeteo, 2);
      expect(b.actuelle.estJour, isTrue);
      expect(b.symboleTemperature, '°C');
      expect(b.symboleVent, 'km/h');
      expect(b.decalageUtcSecondes, 7200);
    });

    test('recompose les colonnes horaires', () async {
      final b = await service.obtenirBulletin(villeTest);
      expect(b.heures.length, 3);
      expect(b.heures[0].heure.hour, 0);
      expect(b.heures[1].temperature, 17.8);
      // La case nulle de precipitation_probability doit rester nulle,
      // surtout pas devenir 0 : « inconnu » n'est pas « aucun risque ».
      expect(b.heures[1].probabilitePluie, isNull);
      expect(b.heures[2].probabilitePluie, 3);
    });

    test('recompose les colonnes journalières', () async {
      final b = await service.obtenirBulletin(villeTest);
      expect(b.jours.length, 2);
      expect(b.jours[0].temperatureMax, 29.1);
      expect(b.jours[1].codeMeteo, 61);
      expect(b.jours[1].probabilitePluieMax, 70);
      expect(b.jours[0].leverSoleil?.hour, 6);
      expect(b.jours[0].coucherSoleil?.hour, 21);
    });
  });

  group('gestion des échecs', () {
    test('un code 500 devient une erreur serveur', () async {
      final faux = MockClient((_) async => http.Response('', 503));
      final service = MeteoService(client: faux);
      expect(
        () => service.obtenirBulletin(villeTest),
        throwsA(isA<ErreurMeteo>()
            .having((e) => e.cause, 'cause', CauseErreur.serveur)),
      );
    });

    test('un code 400 devient une erreur de requête', () async {
      final faux = MockClient(
        (_) async => http.Response('{"error":true,"reason":"latitude"}', 400),
      );
      final service = MeteoService(client: faux);
      expect(
        () => service.obtenirBulletin(villeTest),
        throwsA(isA<ErreurMeteo>()
            .having((e) => e.cause, 'cause', CauseErreur.requete)),
      );
    });

    test('un corps illisible devient une erreur de données', () async {
      final faux = MockClient((_) async => http.Response('ceci nest pas', 200));
      final service = MeteoService(client: faux);
      expect(
        () => service.obtenirBulletin(villeTest),
        throwsA(isA<ErreurMeteo>()
            .having((e) => e.cause, 'cause', CauseErreur.donnees)),
      );
    });

    test('un bloc current absent devient une erreur de données', () async {
      final faux = MockClient((_) async => http.Response('{"daily":{}}', 200));
      final service = MeteoService(client: faux);
      expect(
        () => service.obtenirBulletin(villeTest),
        throwsA(isA<ErreurMeteo>()
            .having((e) => e.cause, 'cause', CauseErreur.donnees)),
      );
    });
  });
}
```

Et le test de la table WMO, qui ne demande ni réseau ni Flutter :

```dart
// test/codes_wmo_test.dart
import 'package:appli_meteo/logique/codes_wmo.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  test('les codes documentés ont tous un libellé', () {
    const codes = [
      0, 1, 2, 3, 45, 48, 51, 53, 55, 56, 57, 61, 63, 65, 66, 67,
      71, 73, 75, 77, 80, 81, 82, 85, 86, 95, 96, 99,
    ];
    for (final c in codes) {
      expect(libelleWmo(c), isNot('Condition inconnue'),
          reason: 'le code $c doit être traduit');
      expect(familleWmo(c), isNot(FamilleMeteo.inconnu),
          reason: 'le code $c doit avoir une famille');
    }
  });

  test('un code hors table reste inconnu, sans planter', () {
    expect(libelleWmo(-1), 'Condition inconnue');
    expect(libelleWmo(4), 'Condition inconnue');
    expect(familleWmo(9999), FamilleMeteo.inconnu);
  });

  test('la pluie est correctement identifiée', () {
    expect(estPluvieux(0), isFalse);
    expect(estPluvieux(3), isFalse);
    expect(estPluvieux(63), isTrue);
    expect(estPluvieux(82), isTrue);
    expect(estPluvieux(95), isTrue);
    expect(estPluvieux(75), isFalse); // la neige n'est pas de la pluie
  });
}
```

```text
flutter test
```

**Résultat attendu :**

```text
00:02 +13: All tests passed!
```

Treize tests, zéro octet échangé sur le réseau, une exécution en deux secondes. C'est précisément ce que permet l'injection du `http.Client` décidée à la section 61.5 : sans elle, aucun de ces tests ne serait écrivable.

---

## 61.24 — Récapitulatif de l'arborescence finale

```text
appli_meteo/
├── pubspec.yaml                        http, provider, shared_preferences,
│                                       intl, flutter_localizations
├── android/app/src/main/AndroidManifest.xml   ← permission INTERNET
├── macos/Runner/DebugProfile.entitlements     ← network.client
├── macos/Runner/Release.entitlements          ← network.client
├── lib/
│   ├── main.dart                       intl, provider, MaterialApp
│   ├── modeles/
│   │   ├── ville.dart                  fromJson, toJson, ==, hashCode
│   │   ├── meteo_actuelle.dart         bloc "current"
│   │   ├── prevision_heure.dart        une case horaire
│   │   ├── prevision_jour.dart         une ligne journalière
│   │   └── bulletin_meteo.dart         décodage des colonnes + getters
│   ├── logique/
│   │   ├── codes_wmo.dart              28 codes → français, familles
│   │   └── unites.dart                 enum UniteTemperature
│   ├── services/
│   │   ├── erreur_meteo.dart           CauseErreur + ErreurMeteo
│   │   ├── geocodage_service.dart      /v1/search
│   │   ├── meteo_service.dart          /v1/forecast, decoder()
│   │   ├── cache_meteo.dart            JSON brut de la dernière réponse
│   │   └── depot_preferences.dart      favoris, historique, unité, ville
│   ├── etat/
│   │   └── etat_meteo.dart             ChangeNotifier unique
│   ├── ecrans/
│   │   ├── ecran_meteo.dart            FutureBuilder → provider, Refresh
│   │   └── ecran_recherche.dart        débounce, favoris, historique
│   ├── widgets/
│   │   ├── fond_meteo.dart             dégradés par famille et par moment
│   │   ├── icone_meteo.dart            code → IconData
│   │   ├── carte_actuelle.dart         la grande carte
│   │   ├── bande_horaire.dart          ListView horizontale
│   │   ├── liste_jours.dart            7 jours + barres de température
│   │   ├── bandeau_hors_ligne.dart     avertissement de données périmées
│   │   └── etats_ecran.dart            chargement, erreur, aucune ville
│   └── utilitaires/
│       ├── dates.dart                  formatage intl en français
│       └── temps.dart                  heure murale
└── test/
    ├── codes_wmo_test.dart
    └── meteo_service_test.dart
```

Vingt-deux fichiers Dart, pas un de plus. Vérifiez le sens des dépendances : il ne doit jamais y avoir de flèche remontante.

```text
ecrans  ──▶  etat  ──▶  services  ──▶  modeles  ──▶  logique
   │           │                          ▲             ▲
   └─▶ widgets ┴──────────────────────────┴─────────────┘

modeles  n'importe NI flutter NI http
logique  n'importe rien du tout
```

---

## 61.25 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `SocketException: Failed host lookup` en release Android | permission `INTERNET` absente du manifeste principal | ajouter `<uses-permission android:name="android.permission.INTERNET"/>` |
| Tout marche en debug, rien en release sur macOS | `com.apple.security.network.client` absent de `Release.entitlements` | l'ajouter dans **les deux** fichiers d'entitlements |
| « Ãle-de-France » au lieu de « Île-de-France » | `reponse.body` au lieu de `utf8.decode(reponse.bodyBytes)` | toujours décoder les octets explicitement |
| `type 'Null' is not a subtype of type 'List<dynamic>'` sur le géocodage | la clé `results` est **absente** quand rien ne correspond | tester `if (resultats is! List)` avant tout `cast` |
| `type 'int' is not a subtype of type 'double'` | JSON renvoie `26` et non `26.0` | lire en `as num?` puis `.toDouble()` |
| `TypeError` aléatoire sur les prévisions horaires | `precipitation_probability` contient des `null` | modéliser le champ en `int?` |
| `RangeError (index)` dans le décodage | une colonne plus courte que `time` | passer par un accesseur borné (`_a`) |
| Le bloc `hourly` est vide et personne ne comprend pourquoi | faute de frappe dans un nom de variable de l'URL | l'API ignore silencieusement les variables inconnues : relire la documentation |
| La bande horaire commence à minuit | `hourly` couvre la journée entière depuis 00:00 | filtrer avec `prochainesHeures` |
| L'heure affichée est fausse de plusieurs heures | comparaison entre un `DateTime` local de l'appareil et une heure locale du lieu | passer par `heureMurale` et `maintenantDansLaVille` |
| Requêtes en boucle infinie, application qui chauffe | `future:` construit dans `build()` | créer le `Future` dans `initState` ou dans le `ChangeNotifier` |
| `Null check operator used on a null value` dans `FutureBuilder` | `snapshot.data!` avant de tester `waiting` et `hasError` | tester les trois états dans l'ordre |
| Huit requêtes pour un mot de huit lettres | pas de débounce | un `Timer` annulé à chaque frappe |
| Un résultat de recherche périmé écrase le bon | réponses revenues dans le désordre | numéroter les requêtes et jeter les réponses obsolètes |
| `setState() called after dispose()` | réponse arrivée après la fermeture de l'écran | `if (!mounted) return;` après chaque `await` |
| `Timer` qui se déclenche sur un `State` détruit | `cancel()` oublié | annuler le minuteur dans `dispose()` |
| `LocaleDataException: Locale data has not been initialized` | `initializeDateFormatting` non appelé | l'appeler dans `main`, avant `runApp` |
| Les jours restent en anglais malgré `intl` | `Intl.defaultLocale` non renseigné, ou locale absente de `DateFormat` | `Intl.defaultLocale = 'fr_FR';` |
| Les boîtes de dialogue Material sont en anglais | `localizationsDelegates` absent | ajouter les trois délégués globaux |
| `Horizontal viewport was given unbounded height` | `ListView` horizontale sans `SizedBox(height: …)` | imposer une hauteur au conteneur et une largeur aux cases |
| Deux défilements imbriqués, contraintes en erreur | `ListView` dans une `ListView` | `shrinkWrap: true` + `NeverScrollableScrollPhysics` sur l'interne |
| Le tirage vers le bas ne déclenche rien | contenu plus court que l'écran | `physics: AlwaysScrollableScrollPhysics()` |
| `ProviderNotFoundException` | le provider est sous `MaterialApp` | le placer au-dessus, dans `runApp` |
| L'écran ne se met plus à jour | `context.read` dans un `build` | `watch` dans `build`, `read` dans les rappels |
| Les favoris ne se retirent jamais | `Ville` sans `operator ==` | redéfinir `==` et `hashCode` sur l'identifiant |
| L'unité relue est fausse après une mise à jour | `enum.index` persisté au lieu de `.name` | toujours `.name` |
| Le cache affiche Paris sous le titre « Tokyo » | cache non vérifié contre la ville demandée | comparer la ville du cache à la ville courante |
| L'utilisateur croit voir des données fraîches hors-ligne | pas de bandeau d'avertissement | afficher l'âge de la donnée, toujours |
| `flutter test` échoue avec des types incompatibles | imports relatifs de `lib/` depuis `test/` | importer en `package:appli_meteo/...` |

---

## 61.26 — Ce que vous avez appris

| Notion | À retenir |
| --- | --- |
| API sans clé | Open-Meteo se consomme en GET public : aucun secret à cacher, aucun quota à gérer, un projet reproductible dans le temps. |
| Enchaînement de deux API | Géocodage puis prévision. Le motif « texte → identifiant → données » est universel. |
| Lire une documentation | Les noms de paramètres et de variables ne s'inventent pas. Une variable inconnue est ignorée en silence par le serveur. |
| Format en colonnes | Un tableau par variable, aligné par index. Compact sur le réseau, à recomposer côté client. |
| `fromJson` défensif | `as num?` + `?.toDouble()` + `??`. Une colonne trouée ou plus courte ne doit jamais faire planter l'écran. |
| Valeur de repli signifiante | Un code météo manquant vaut `-1`, jamais `0` : `0` signifie « ciel dégagé ». |
| Heure murale | Avec `timezone=auto`, l'API renvoie l'heure du lieu. On la compare à `maintenantDansLaVille`, jamais à `DateTime.now()`. |
| Unités lues, non devinées | `current_units` dit « °C » ou « °F ». On l'affiche tel quel plutôt que de le coder en dur. |
| Couche service | Elle seule connaît `http` et les URL. Elle ne laisse remonter qu'un seul type d'exception. |
| Exception métier | Un `enum CauseErreur` porte le message affichable ; l'interface n'a qu'un `catch` à écrire. |
| Injection du client | `MeteoService({http.Client? client})` : c'est cette ligne qui rend tout le service testable. |
| Trois états | Chargement, erreur, succès. Un écran d'erreur sans bouton d'action est une impasse. |
| Permissions réseau | Android : le manifeste principal, pas seulement celui de debug. macOS : les deux fichiers d'entitlements. |
| Codes WMO | Numérotation trouée, `switch` plutôt que tableau ; `switch` sans `default` sur un `enum` pour que le compilateur surveille l'exhaustivité. |
| Dégradé contextuel | Deux entrées — famille de condition et jour/nuit — suffisent à donner une identité forte à l'application. |
| `ListView` horizontale | Hauteur imposée au conteneur, largeur imposée aux enfants. |
| Liste dans une liste | `shrinkWrap: true` et `NeverScrollableScrollPhysics` sur l'interne. |
| `intl` | Deux initialisations distinctes : les données de langue et les délégués de localisation. |
| Débounce | Un `Timer` annulé à chaque frappe, plus un numéro de requête pour ignorer les réponses tardives. |
| `shared_preferences` | Parfait pour des réglages et quelques dizaines d'objets. Écrire `.name`, jamais `.index`. |
| Cache du JSON brut | Zéro `toJson` à écrire pour les modèles, et un seul chemin de décodage à maintenir. |
| Réseau d'abord, cache ensuite | Le cache est lu dans le `catch`. En ligne, on ne montre jamais de donnée périmée. |
| Honnêteté de l'affichage | Toute donnée hors-ligne s'accompagne de son âge. Mentir à l'utilisateur est un défaut fonctionnel. |
| `RefreshIndicator` | Enfant défilant obligatoire, et `AlwaysScrollableScrollPhysics` pour que le geste parte toujours. |
| `ChangeNotifier` + `provider` | Un objet détient l'état, expose des lectures non modifiables, notifie après chaque changement. |
| `watch` / `read` | `watch` dans un `build`, `read` dans un rappel. L'erreur la plus fréquente avec `provider`. |
| `MockClient` | Tester une couche réseau sans réseau : réponse figée, exécution en millisecondes, résultat reproductible. |
| Tester les échecs | Un test par cause d'erreur. C'est là que les bogues se cachent, pas dans le chemin heureux. |

---

## 61.27 — Extensions : dix défis

Chaque défi est réalisable avec ce que vous savez déjà. L'indication donne la direction, pas la solution.

### Défi 1 — Lever et coucher du soleil (facile)

Affichez les heures de lever et de coucher sous les indicateurs de la carte du jour.

*Indication :* les champs existent déjà dans `PrevisionJour` et sont demandés dans l'URL. Il n'y a rien à changer côté service : ajoutez deux `_Indicateur` dans `carte_actuelle.dart`, avec `heureCourte(jour.leverSoleil!)`. Attention au cas nul des régions polaires : conditionnez l'affichage.

### Défi 2 — La flèche du vent (facile)

Orientez une icône de flèche selon la direction du vent.

*Indication :* ajoutez `wind_direction_10m` à `_variablesActuelles`, un champ `directionVent` à `MeteoActuelle`, puis enveloppez l'icône dans un `Transform.rotate(angle: direction * pi / 180)`. La direction météorologique indique d'**où** vient le vent : il faut ajouter 180 degrés pour dessiner la flèche dans le sens du déplacement.

### Défi 3 — L'indice UV (facile)

Affichez l'indice UV maximal du jour, avec une pastille de couleur selon le seuil.

*Indication :* la variable journalière s'appelle `uv_index_max`. Les seuils officiels sont 0-2 faible, 3-5 modéré, 6-7 fort, 8-10 très fort, 11+ extrême. Écrivez la conversion seuil → couleur dans `logique/`, avec son test.

### Défi 4 — Le graphique des températures horaires (moyen)

Remplacez la bande horaire par une courbe.

*Indication :* `CustomPainter` (que vous retrouverez au chapitre 21 de la PARTIE 2). Normalisez les températures dans l'amplitude des 24 heures, puis tracez un `Path` avec `lineTo`. Le plus dur n'est pas la courbe, ce sont les graduations.

### Défi 5 — Plusieurs villes en onglets (moyen)

Faites glisser horizontalement d'une ville favorite à l'autre.

*Indication :* un `PageView.builder` alimenté par `etat.favoris`, avec un `SmoothPageIndicator` maison en points. Attention : il faut alors un bulletin **par ville**, donc une `Map<int, BulletinMeteo>` dans `EtatMeteo` au lieu d'un seul champ.

### Défi 6 — Le texte lisible sur tout fond (moyen)

Les fonds clairs de neige rendent le texte blanc illisible. Calculez automatiquement la couleur du texte.

*Indication :* `ThemeData.estimateBrightnessForColor(couleur)` renvoie `Brightness.light` ou `Brightness.dark`. Calculez-la sur la couleur médiane du dégradé, et exposez la couleur de texte via un `InheritedWidget` ou un simple paramètre de `FondMeteo`. Vérifiez le résultat sur le code 71 en journée.

### Défi 7 — La position de l'appareil (moyen)

Un bouton « Ma position » affiche la météo du lieu où l'on se trouve.

*Indication :* le paquet `geolocator`, plus les permissions de localisation dans le manifeste Android et dans l'`Info.plist` iOS. Vous obtiendrez des coordonnées sans nom : appelez alors l'API de géocodage **inverse** d'Open-Meteo, ou construisez une `Ville` au nom « Ma position ». Traitez les trois refus possibles : permission refusée, refusée définitivement, service désactivé.

### Défi 8 — Le cache multi-villes (difficile)

Aujourd'hui, une seule réponse est mise en cache. Conservez la dernière réponse de **chaque** ville favorite.

*Indication :* remplacez la clé unique par une clé calculée `meteo.cache.<id>`, et ajoutez une purge des entrées de plus de 24 heures au démarrage. Quand le nombre d'entrées dépasse la dizaine, `shared_preferences` n'est plus le bon outil : c'est le moment de passer à `sqflite` (chapitre 54).

### Défi 9 — L'alerte de pluie dans l'heure (difficile)

Prévenez l'utilisateur si la pluie commence dans les soixante minutes.

*Indication :* Open-Meteo propose un bloc `minutely_15` avec `precipitation` au quart d'heure. Ajoutez `minutely_15=precipitation` à l'URL, un cinquième modèle, et une règle métier dans `logique/` : « la pluie commence à HH:MM » dès qu'un créneau dépasse 0,1 mm. Écrivez d'abord le test, avec une colonne factice.

### Défi 10 — Comparer avec les années passées (difficile)

Affichez « 4 °C de plus que la moyenne du 15 août ».

*Indication :* Open-Meteo publie une API d'archive, sur `archive-api.open-meteo.com/v1/archive`, avec `start_date` et `end_date`. Interrogez les dix dernières années au même jour, calculez la moyenne avec `fold` (chapitre 14), et mettez le résultat en cache pendant un an : cette donnée ne change pas. C'est le défi le plus long des dix ; comptez une journée.

---

## Et maintenant ?

Vous venez d'écrire l'application la plus complète de cette partie. Reprenez la liste : deux API réelles enchaînées, cinq modèles, une couche service testée, un cache hors-ligne, une recherche débouncée, un état centralisé, une interface qui change de visage selon le ciel. C'est, à la virgule près, l'ossature d'une application professionnelle qui consomme un service distant. Changez `open-meteo.com` en n'importe quelle autre API, changez les noms des modèles, et l'architecture reste identique.

Retenez surtout la leçon la moins spectaculaire du chapitre : **la moitié du code sert à traiter ce qui ne se passe pas comme prévu.** Une clé absente, une colonne trouée, un réseau coupé, une réponse revenue dans le désordre. Un développeur débutant écrit le chemin heureux ; un développeur confirmé écrit les autres.

Le chapitre suivant clôt la PARTIE 1C, et il a une double fonction. Vous y construirez un **lanceur de jeu** — un écran d'accueil, un menu, un choix de personnage, un tableau de scores persisté, un écran de réglages — en réutilisant absolument tout ce que vous savez faire. Mais il sert aussi de pont : ce lanceur est exactement l'enveloppe Flutter dans laquelle viendra se loger le jeu 2D de la PARTIE 2. Vous n'écrirez pas ce code deux fois.

Rendez-vous au chapitre 62 : [62-PARTIE-1C—PROJET-8-APPLICATION-PRÉPARATOIRE-AU-JEU.md](./62-PARTIE-1C—PROJET-8-APPLICATION-PRÉPARATOIRE-AU-JEU.md)
