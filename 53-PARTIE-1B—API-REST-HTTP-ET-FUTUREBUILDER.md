# PARTIE 1B — FLUTTER
# CHAPITRE 53 — CONSOMMER UNE API REST : HTTP, JSON ET FUTUREBUILDER

> **Niveau :** intermédiaire
> **Durée estimée :** 12 h
> **Pré-requis :** chapitre 52 — Gestion d'état : `setState` et `provider` (et, dans la PARTIE 1A : chapitre 13 — Exceptions, chapitre 15 — Asynchrone, chapitre 16 — Organisation d'un projet, chapitre 17 — JSON)
> **Ce que vous saurez faire à la fin :** aller chercher des données sur un serveur, les transformer en objets Dart, les afficher dans une liste, et gérer proprement le chargement, les erreurs, le réessai, la recherche et la pagination.

---

## 53.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer d'où viennent les données d'une application mobile ;
- décrire le cycle requête / réponse entre un client et un serveur ;
- définir ce qu'est une API REST et ce qu'est une ressource ;
- choisir le bon verbe HTTP parmi `GET`, `POST`, `PUT`, `PATCH` et `DELETE` ;
- lire un code de statut HTTP et réagir correctement ;
- lire et écrire des en-têtes HTTP, en particulier `Content-Type` et `Accept` ;
- installer le package `http` et déclarer les permissions réseau sur Android et macOS ;
- écrire un appel `http.get()` et récupérer `statusCode` et `body` ;
- décoder du JSON avec `jsonDecode` et construire un modèle Dart avec `fromJson()` ;
- transformer un tableau JSON en `List<T>` d'objets typés ;
- isoler le réseau dans une **couche service**, séparée de l'interface ;
- attraper `SocketException`, `TimeoutException`, `FormatException` et `ClientException` ;
- poser un délai d'attente avec `.timeout()` ;
- énoncer les trois états d'un écran de données : chargement, erreur, succès ;
- utiliser `FutureBuilder` et lire un `AsyncSnapshot` ;
- expliquer pourquoi il ne faut jamais créer un `Future` dans `build()` ;
- créer le `Future` dans `initState()` et le recréer avec `setState()` ;
- ajouter un bouton « réessayer » et un `RefreshIndicator` ;
- comparer `FutureBuilder` et une gestion d'état manuelle, et choisir ;
- envoyer du JSON avec `http.post()`, puis modifier et supprimer ;
- authentifier une requête avec l'en-tête `Authorization` ;
- expliquer pourquoi une clé d'API n'a rien à faire dans le code source ;
- afficher un flux continu avec `StreamBuilder` ;
- paginer une liste et charger la suite au défilement ;
- appliquer un **débounce** à un champ de recherche ;
- annuler une requête devenue inutile ;
- citer ce que `dio` apporte de plus que `http` ;
- simuler une API pour développer sans réseau ;
- tester une couche service avec un client factice ;
- construire un écran de liste complet : chargement, erreur, réessai, recherche.

---

## 53.0.1 — Pourquoi ce chapitre change tout

Depuis le chapitre 43, toutes vos applications ont un point commun gênant : **les données sont écrites dans le code**.

```dart
final joueurs = ['Alia', 'Baltus', 'Cendre'];
```

Cette liste est figée. Elle ne changera jamais, sauf si vous recompilez l'application et la republiez sur les magasins. Autrement dit : vos applications sont de jolies vitrines vides.

Une vraie application affiche des données **qui vivent ailleurs** : un classement de joueurs mis à jour toutes les secondes, un catalogue de produits, une météo, des messages, un inventaire partagé entre plusieurs appareils.

Ces données sont sur un **serveur**. Pour les obtenir, votre application doit :

1. envoyer une requête sur le réseau ;
2. attendre (c'est long, et ça peut échouer) ;
3. recevoir du texte, en général du JSON ;
4. transformer ce texte en objets Dart ;
5. afficher ces objets ;
6. et gérer tout ce qui peut mal tourner.

Chacune de ces six étapes a ses pièges. Ce chapitre les traite dans l'ordre.

> Ce chapitre réutilise massivement la PARTIE 1A. L'asynchrone du chapitre 15, les exceptions du chapitre 13 et le JSON du chapitre 17 ne sont plus des exercices théoriques : ils deviennent votre outil de travail quotidien.

---

## 53.1 — D'où viennent les données d'une application

Une application Flutter n'a que quatre sources de données possibles. Les voici, de la plus simple à la plus complexe.

**Source 1 — Le code lui-même.**
Les données sont écrites en dur dans les fichiers `.dart`. C'est ce que vous faites depuis le chapitre 43. Avantage : instantané, jamais en panne. Inconvénient : figé.

**Source 2 — Les assets.**
Un fichier embarqué dans l'application (`assets/niveaux.json`), lu au démarrage. C'est vu au chapitre 47 pour les images. Avantage : pas de réseau. Inconvénient : toujours figé au moment de la publication.

**Source 3 — Le stockage local.**
Les données sont écrites sur l'appareil de l'utilisateur : préférences, base de données locale, fichiers. C'est le sujet du chapitre 54. Avantage : rapide, hors-ligne. Inconvénient : propre à un seul appareil.

**Source 4 — Le réseau.**
Les données sont sur un serveur distant, partagées par tous les utilisateurs. C'est le sujet de **ce** chapitre. Avantage : à jour, partagé, illimité. Inconvénient : lent, faillible, et l'utilisateur peut être dans le métro.

Voici le tableau de décision.

| Source | Modifiable après publication | Partagée entre appareils | Fonctionne hors-ligne | Peut échouer |
| --- | --- | --- | --- | --- |
| Code Dart | Non | Non | Oui | Non |
| Asset embarqué | Non | Non | Oui | Non |
| Stockage local | Oui | Non | Oui | Rarement |
| Réseau (API) | Oui | Oui | Non | **Souvent** |

Retenez la dernière colonne. **Une requête réseau échoue régulièrement.** Ce n'est pas un cas exceptionnel à traiter à la fin : c'est un cas normal à traiter dès la première ligne de code.

---

## 53.2 — Client, serveur, requête, réponse

Le modèle est toujours le même, et il n'a que quatre pièces.

- Le **client** : votre application Flutter. C'est lui qui demande.
- Le **serveur** : une machine allumée quelque part, qui écoute. C'est lui qui répond.
- La **requête** (*request*) : le message que le client envoie.
- La **réponse** (*response*) : le message que le serveur renvoie.

Point capital : **le serveur ne parle jamais en premier.** Il ne peut que répondre. Si votre application veut des données fraîches, c'est à elle de redemander.

Voici le cycle complet.

```text
   CLIENT (votre application Flutter)              SERVEUR (api.exemple.com)
   ┌────────────────────────────────┐              ┌────────────────────────┐
   │                                │              │                        │
   │  1. On construit la requête    │              │                        │
   │     GET /joueurs               │              │                        │
   │     Host: api.exemple.com      │              │                        │
   │     Accept: application/json   │              │                        │
   │                                │              │                        │
   │            ────────────────────┼─────────────>│  2. Le serveur reçoit  │
   │                  REQUÊTE       │              │     et cherche dans    │
   │                                │              │     sa base            │
   │  3. L'application ATTEND       │              │                        │
   │     (100 ms ? 5 s ? jamais ?)  │              │  4. Il fabrique une    │
   │                                │              │     réponse JSON       │
   │                                │              │                        │
   │            <───────────────────┼──────────────│                        │
   │                  RÉPONSE       │              │                        │
   │     HTTP/1.1 200 OK            │              │                        │
   │     Content-Type: application/ │              │                        │
   │                   json         │              │                        │
   │                                │              │                        │
   │     [{"id":1,"nom":"Alia"}]    │              │                        │
   │                                │              │                        │
   │  5. On décode le JSON          │              │                        │
   │  6. On construit des objets    │              │                        │
   │  7. On affiche                 │              │                        │
   └────────────────────────────────┘              └────────────────────────┘
```

Regardez bien l'étape 3. **L'application attend.** Pendant ce temps, l'interface doit rester vivante : l'utilisateur peut faire défiler, appuyer sur un bouton, quitter l'écran. C'est exactement pourquoi tout le réseau est **asynchrone** en Dart, et pourquoi vous avez appris `Future`, `async` et `await` au chapitre 15.

---

## 53.2.1 — L'anatomie d'une requête

Une requête HTTP contient quatre choses, toujours les mêmes.

```text
┌─────────────────────────────────────────────────────────────┐
│  1. LE VERBE   +   2. L'URL                                 │
│     GET            https://api.exemple.com/joueurs?tri=score│
├─────────────────────────────────────────────────────────────┤
│  3. LES EN-TÊTES                                            │
│     Accept: application/json                                │
│     Authorization: Bearer eyJhbGciOi...                     │
│     Content-Type: application/json                          │
├─────────────────────────────────────────────────────────────┤
│  4. LE CORPS (facultatif, absent en GET)                    │
│     {"nom": "Alia", "score": 4200}                          │
└─────────────────────────────────────────────────────────────┘
```

Et une réponse en contient trois.

```text
┌─────────────────────────────────────────────────────────────┐
│  1. LE CODE DE STATUT                                       │
│     200 OK                                                  │
├─────────────────────────────────────────────────────────────┤
│  2. LES EN-TÊTES                                            │
│     Content-Type: application/json; charset=utf-8           │
│     Content-Length: 217                                     │
├─────────────────────────────────────────────────────────────┤
│  3. LE CORPS                                                │
│     [{"id":1,"nom":"Alia","score":4200}]                    │
└─────────────────────────────────────────────────────────────┘
```

Ces sept éléments sont tout ce qu'il y a à connaître. Les sections 53.4 à 53.7 les détaillent un par un.

---

## 53.2.2 — Décomposer une URL

Une URL n'est pas une chaîne magique. Elle a des morceaux nommés, et Dart sait les manipuler avec la classe `Uri`.

```text
  https://api.exemple.com:443/v1/joueurs?tri=score&limite=20#haut
  └─┬─┘   └──────┬──────┘└┬┘└────┬─────┘└────────┬─────────┘└─┬┘
 schéma        hôte      port   chemin      paramètres     fragment
                                            de requête
```

| Morceau | Nom Dart | Rôle |
| --- | --- | --- |
| `https` | `scheme` | Le protocole. Toujours `https` en production. |
| `api.exemple.com` | `host` | La machine à contacter. |
| `443` | `port` | Le port. Implicite : 80 en HTTP, 443 en HTTPS. |
| `/v1/joueurs` | `path` | La ressource demandée. |
| `tri=score&limite=20` | `queryParameters` | Des options de la requête. |
| `haut` | `fragment` | Jamais envoyé au serveur. Inutile en API. |

En Dart, on ne colle jamais une URL à la main quand elle contient des paramètres. On utilise `Uri.https` ou `Uri.parse`.

```dart
void main() {
  // Méthode 1 — analyser une chaîne complète.
  final a = Uri.parse('https://api.exemple.com/v1/joueurs?tri=score');
  print(a.host);
  print(a.path);
  print(a.queryParameters);

  // Méthode 2 — construire proprement (recommandé).
  final b = Uri.https(
    'api.exemple.com',
    '/v1/joueurs',
    {'tri': 'score', 'limite': '20'},
  );
  print(b);

  // Méthode 3 — le piège que Uri.https évite : les caractères spéciaux.
  final c = Uri.https('api.exemple.com', '/recherche', {'q': 'épée à deux mains'});
  print(c);
}
```

**Résultat :**

```text
api.exemple.com
/v1/joueurs
{tri: score}
https://api.exemple.com/v1/joueurs?tri=score&limite=20
https://api.exemple.com/recherche?q=%C3%A9p%C3%A9e+%C3%A0+deux+mains
```

Regardez la dernière ligne. `Uri.https` a **encodé** les espaces et les accents. Si vous aviez écrit l'URL à la main, le serveur aurait reçu une requête invalide.

> Retenez : **les valeurs de `queryParameters` sont toujours des `String`.** `{'limite': 20}` ne compile pas. Écrivez `{'limite': '20'}` ou `{'limite': 20.toString()}`.

---

## 53.3 — Qu'est-ce qu'une API REST

**API** signifie *Application Programming Interface* : une porte d'entrée pensée pour être utilisée par un programme, pas par un humain. Une page web est faite pour un œil ; une API est faite pour du code.

**REST** (*REpresentational State Transfer*) est un **style** de conception d'API. Ce n'est pas une technologie, pas une bibliothèque, pas une norme obligatoire : c'est un ensemble de conventions que la plupart des serveurs suivent.

Les conventions REST tiennent en cinq idées.

**Idée 1 — Tout est une ressource, désignée par une URL.**

Une ressource est une chose : un joueur, une arme, un score. Chaque ressource a une adresse.

```text
/joueurs          -> la collection de tous les joueurs
/joueurs/42       -> le joueur numéro 42
/joueurs/42/armes -> les armes du joueur 42
```

**Idée 2 — Le verbe HTTP dit ce qu'on veut faire.**

L'URL dit *sur quoi*, le verbe dit *quoi faire*. On ne met jamais l'action dans l'URL.

```text
BIEN                         MAL
GET    /joueurs/42           GET /obtenirJoueur?id=42
DELETE /joueurs/42           GET /supprimerJoueur?id=42
POST   /joueurs              GET /creerJoueur?nom=Alia
```

**Idée 3 — Les URL de collection sont au pluriel.**

`/joueurs`, pas `/joueur`. `/armes`, pas `/arme`. C'est une convention, mais elle est quasi universelle.

**Idée 4 — Le serveur est sans mémoire de session (*stateless*).**

Chaque requête est indépendante. Le serveur ne se souvient pas de la requête précédente. Si une requête a besoin d'une identité, elle transporte elle-même son jeton d'authentification (section 53.31).

**Idée 5 — L'échange se fait en JSON.**

Historiquement c'était du XML. Aujourd'hui, dans plus de 95 % des cas, c'est du JSON — celui du chapitre 17.

Voici à quoi ressemble une API REST complète pour des joueurs.

```text
┌────────┬────────────────┬──────────────────────────────────────┐
│ VERBE  │ URL            │ SIGNIFICATION                        │
├────────┼────────────────┼──────────────────────────────────────┤
│ GET    │ /joueurs       │ Lire la liste de tous les joueurs    │
│ GET    │ /joueurs/42    │ Lire le joueur 42                    │
│ POST   │ /joueurs       │ Créer un nouveau joueur              │
│ PUT    │ /joueurs/42    │ Remplacer entièrement le joueur 42   │
│ PATCH  │ /joueurs/42    │ Modifier un champ du joueur 42       │
│ DELETE │ /joueurs/42    │ Supprimer le joueur 42               │
└────────┴────────────────┴──────────────────────────────────────┘
```

Six lignes. Une fois que vous savez lire ce tableau, vous savez lire la documentation de n'importe quelle API REST du monde.

---

## 53.4 — Les verbes HTTP : GET, POST, PUT, PATCH, DELETE

Le verbe est le premier mot de la requête. Il annonce l'intention.

### GET — lire

`GET` demande des données. Il ne modifie **rien** sur le serveur. On dit qu'il est **sûr** (*safe*).

```text
GET /joueurs/42 HTTP/1.1
Host: api.exemple.com
Accept: application/json
```

Un `GET` n'a **pas de corps**. Toutes ses options passent par les paramètres de requête (`?tri=score`).

Conséquence pratique : vous pouvez rejouer un `GET` autant de fois que vous voulez, sans risque. C'est pourquoi le bouton « réessayer » de la section 53.26 est sans danger.

### POST — créer

`POST` envoie des données pour créer une nouvelle ressource. Le corps contient le nouvel objet.

```text
POST /joueurs HTTP/1.1
Host: api.exemple.com
Content-Type: application/json

{"nom": "Alia", "score": 0}
```

Le serveur répond en général `201 Created` et renvoie l'objet créé, **avec l'identifiant qu'il a attribué**.

Attention : `POST` n'est **pas** idempotent. Deux `POST` identiques créent **deux** joueurs. C'est la raison pour laquelle on désactive un bouton d'envoi pendant la requête.

### PUT — remplacer

`PUT` remplace **entièrement** une ressource existante. Le corps doit contenir **tous** les champs.

```text
PUT /joueurs/42 HTTP/1.1
Content-Type: application/json

{"id": 42, "nom": "Alia", "score": 5100, "niveau": 7}
```

Si vous oubliez `niveau` dans le corps d'un `PUT`, un serveur strict effacera le niveau.

`PUT` **est** idempotent : envoyer deux fois la même requête donne le même résultat final.

### PATCH — modifier partiellement

`PATCH` ne modifie que les champs fournis. Le reste est laissé tel quel.

```text
PATCH /joueurs/42 HTTP/1.1
Content-Type: application/json

{"score": 5100}
```

C'est le verbe le plus économique : on n'envoie que ce qui change.

### DELETE — supprimer

`DELETE` supprime la ressource. Il n'a en général pas de corps.

```text
DELETE /joueurs/42 HTTP/1.1
```

Le serveur répond `204 No Content` (succès sans corps) ou `200 OK`.

`DELETE` est idempotent au sens où l'état final est le même : après deux `DELETE`, le joueur 42 n'existe pas. Le second renverra probablement `404`.

### Le tableau de synthèse

| Verbe | Rôle | Corps | Sûr (ne modifie rien) | Idempotent |
| --- | --- | --- | --- | --- |
| `GET` | Lire | Non | Oui | Oui |
| `POST` | Créer | Oui | Non | **Non** |
| `PUT` | Remplacer en entier | Oui | Non | Oui |
| `PATCH` | Modifier en partie | Oui | Non | En général |
| `DELETE` | Supprimer | Rarement | Non | Oui |

**Idempotent** signifie : répéter la requête ne change pas le résultat final. C'est la propriété qui décide si vous avez le droit de réessayer automatiquement après un échec réseau. On réessaie un `GET` sans réfléchir ; on réessaie un `POST` avec prudence.

---

## 53.5 — Les codes de statut

Le serveur répond toujours par un nombre à trois chiffres. Le premier chiffre donne la famille.

```text
1xx  Information       « J'ai reçu, je continue »        (rare en API)
2xx  Succès            « C'est fait »
3xx  Redirection       « Ce n'est plus ici »
4xx  Erreur du CLIENT  « C'est VOTRE requête qui est fautive »
5xx  Erreur du SERVEUR « C'est MOI qui suis en panne »
```

Cette distinction 4xx / 5xx est la plus importante du chapitre pour vos messages d'erreur.

- **4xx** : réessayer à l'identique ne servira à rien. La requête est mauvaise. Il faut corriger quelque chose (l'URL, le jeton, les données envoyées).
- **5xx** : la requête était correcte, le serveur a un problème. Réessayer plus tard a du sens.

### Les codes que vous rencontrerez vraiment

| Code | Nom | Signification | Que faire dans l'application |
| --- | --- | --- | --- |
| `200` | OK | Succès, le corps contient le résultat | Décoder le JSON |
| `201` | Created | Ressource créée (réponse à un `POST`) | Lire l'objet créé, souvent son `id` |
| `204` | No Content | Succès, **corps vide** | Ne surtout pas décoder le corps |
| `301` / `302` | Moved / Found | La ressource a déménagé | Le package `http` suit la redirection tout seul |
| `304` | Not Modified | Rien n'a changé depuis votre dernière requête | Garder le cache local |
| `400` | Bad Request | Requête malformée (JSON invalide, champ manquant) | Corriger l'envoi ; afficher le message du serveur |
| `401` | Unauthorized | Non authentifié : jeton absent, expiré ou faux | Renvoyer vers l'écran de connexion |
| `403` | Forbidden | Authentifié mais pas le droit | Message « accès refusé », pas de reconnexion |
| `404` | Not Found | La ressource n'existe pas | Afficher « introuvable », pas une erreur réseau |
| `408` | Request Timeout | Le serveur a attendu trop longtemps | Réessayer |
| `409` | Conflict | Conflit (doublon, version concurrente) | Recharger puis proposer une fusion |
| `422` | Unprocessable Entity | JSON valide mais données refusées | Afficher les erreurs de validation champ par champ |
| `429` | Too Many Requests | Vous appelez trop souvent | Attendre ; lire l'en-tête `Retry-After` |
| `500` | Internal Server Error | Bug côté serveur | « Réessayez plus tard » |
| `502` | Bad Gateway | Passerelle en panne | Réessayer |
| `503` | Service Unavailable | Serveur en maintenance ou saturé | Réessayer plus tard |
| `504` | Gateway Timeout | Le serveur amont n'a pas répondu | Réessayer |

### Le test à écrire, et celui à ne pas écrire

Beaucoup de débutants écrivent :

```dart
if (response.statusCode == 200) {
  // succès
}
```

C'est correct pour un `GET` simple, mais faux pour un `POST` qui renvoie `201`, et faux pour un `DELETE` qui renvoie `204`.

Le test robuste porte sur la **famille** :

```dart
void main() {
  bool estSucces(int code) => code >= 200 && code < 300;

  for (final code in [200, 201, 204, 301, 400, 404, 500]) {
    print('$code -> ${estSucces(code) ? "succès" : "échec"}');
  }
}
```

**Résultat :**

```text
200 -> succès
201 -> succès
204 -> succès
301 -> échec
400 -> échec
404 -> échec
500 -> échec
```

> Retenez la règle : **`statusCode >= 200 && statusCode < 300` est le test du succès.**

---

## 53.6 — Les en-têtes

Les en-têtes (*headers*) sont des couples `clé: valeur` qui accompagnent la requête et la réponse. Ils ne contiennent jamais les données elles-mêmes, mais des **informations sur** les données.

En Dart, c'est simplement une `Map<String, String>`.

### Les en-têtes de requête utiles

| En-tête | Exemple | Rôle |
| --- | --- | --- |
| `Accept` | `application/json` | « Je veux du JSON en retour » |
| `Content-Type` | `application/json; charset=utf-8` | « Ce que j'envoie dans le corps est du JSON » |
| `Authorization` | `Bearer eyJhbGciOi...` | Prouve qui vous êtes (section 53.31) |
| `User-Agent` | `MonJeu/1.4 (Android)` | Identifie l'application appelante |
| `Accept-Language` | `fr-FR` | Demande une réponse en français |
| `If-None-Match` | `"abc123"` | « Ne renvoie rien si ça n'a pas changé » (cache) |

### Les en-têtes de réponse utiles

| En-tête | Exemple | Rôle |
| --- | --- | --- |
| `Content-Type` | `application/json; charset=utf-8` | Le format du corps renvoyé |
| `Content-Length` | `1274` | La taille du corps en octets |
| `ETag` | `"abc123"` | Une empreinte de la ressource, pour le cache |
| `Retry-After` | `30` | « Réessayez dans 30 secondes » (avec un `429`) |
| `X-Total-Count` | `240` | Nombre total d'éléments (pagination, non standard) |
| `Link` | `<...?page=3>; rel="next"` | Liens vers les pages suivantes |

### Écrire des en-têtes en Dart

```dart
void main() {
  final enTetes = <String, String>{
    'Accept': 'application/json',
    'Content-Type': 'application/json; charset=utf-8',
    'Accept-Language': 'fr-FR',
  };

  enTetes.forEach((cle, valeur) => print('$cle: $valeur'));
}
```

**Résultat :**

```text
Accept: application/json
Content-Type: application/json; charset=utf-8
Accept-Language: fr-FR
```

### Lire les en-têtes de la réponse

Les en-têtes reçus sont dans `response.headers`. Deux pièges :

1. **Les clés sont toujours en minuscules** dans le package `http`, quelle que soit la casse envoyée par le serveur. Écrivez `response.headers['content-type']`, jamais `'Content-Type'`.
2. La valeur est une `String?`. Elle peut être absente.

```dart
void main() {
  // Simulation de ce que renvoie le package http.
  final headers = <String, String>{
    'content-type': 'application/json; charset=utf-8',
    'content-length': '1274',
  };

  final type = headers['content-type'];
  print('Type : $type');
  print('Est du JSON ? ${type?.contains('application/json') ?? false}');
  print('Absent : ${headers['x-total-count']}');
}
```

**Résultat :**

```text
Type : application/json; charset=utf-8
Est du JSON ? true
Absent : null
```

---

## 53.7 — Le corps de la requête

Le **corps** (*body*) est le contenu envoyé au serveur. Il n'existe que pour `POST`, `PUT` et `PATCH`.

Dans le package `http`, le paramètre `body` accepte trois types :

| Type passé à `body` | Ce que `http` en fait |
| --- | --- |
| `String` | Envoyé tel quel, encodé selon `encoding` (UTF-8 par défaut) |
| `List<int>` | Envoyé octet par octet |
| `Map<String, String>` | Encodé automatiquement en `application/x-www-form-urlencoded` |

Cette troisième ligne est un piège majeur. Si vous passez une `Map` en pensant envoyer du JSON, vous envoyez en réalité un formulaire.

```dart
import 'dart:convert';

void main() {
  final joueur = {'nom': 'Alia', 'score': 4200};

  // FAUX : ce n'est pas du JSON, et « score » n'est même pas une String.
  // http.post(url, body: joueur);  // ne compile pas : Map<String, Object>

  // CORRECT : on encode nous-mêmes en JSON.
  final corps = jsonEncode(joueur);
  print(corps);
  print(corps.runtimeType);
}
```

**Résultat :**

```text
{"nom":"Alia","score":4200}
String
```

> **La règle à retenir :** pour envoyer du JSON, on encode soi-même avec `jsonEncode()` **et** on déclare `Content-Type: application/json`. Les deux, toujours ensemble. Un corps JSON sans le bon `Content-Type` sera rejeté par la plupart des serveurs avec un `400` ou un `415`.

### Les trois formats de corps que vous croiserez

```text
1. JSON  (99 % des API modernes)
   Content-Type: application/json
   {"nom":"Alia","score":4200}

2. FORMULAIRE  (anciens serveurs, connexions simples)
   Content-Type: application/x-www-form-urlencoded
   nom=Alia&score=4200

3. MULTIPART  (envoi de fichiers : photo de profil, capture d'écran)
   Content-Type: multipart/form-data; boundary=----xyz
   ------xyz
   Content-Disposition: form-data; name="avatar"; filename="a.png"
   <octets du fichier>
   ------xyz--
```

Pour le troisième, `http` fournit `MultipartRequest`. Il sort du cadre de ce chapitre, mais sachez qu'il existe.

---

## 53.8 — Une API publique de test pour s'entraîner

Pour apprendre, il vous faut un serveur qui répond, gratuitement, sans inscription et sans clé. Il en existe plusieurs. Ce chapitre en utilise deux, choisies pour leur stabilité.

### JSONPlaceholder

Adresse : `https://jsonplaceholder.typicode.com`

C'est une fausse API qui simule un mini-réseau social. Elle accepte les cinq verbes. Les écritures sont **simulées** : le serveur répond comme si l'opération avait réussi, mais rien n'est réellement enregistré. C'est parfait pour apprendre sans rien casser.

Les ressources disponibles :

| Ressource | Nombre d'éléments |
| --- | --- |
| `/posts` | 100 |
| `/comments` | 500 |
| `/albums` | 100 |
| `/photos` | 5000 |
| `/todos` | 200 |
| `/users` | 10 |

Un élément de `/posts` a exactement quatre champs :

```json
{
  "userId": 1,
  "id": 1,
  "title": "sunt aut facere repellat provident",
  "body": "quia et suscipit\nsuscipit recusandae"
}
```

Un élément de `/users` est plus riche, avec des objets imbriqués — pratique pour s'exercer au JSON du chapitre 17 :

```json
{
  "id": 1,
  "name": "Leanne Graham",
  "username": "Bret",
  "email": "Sincere@april.biz",
  "address": {
    "street": "Kulas Light",
    "suite": "Apt. 556",
    "city": "Gwenborough",
    "zipcode": "92998-3874",
    "geo": { "lat": "-37.3159", "lng": "81.1496" }
  },
  "phone": "1-770-736-8031 x56442",
  "website": "hildegard.org",
  "company": {
    "name": "Romaguera-Crona",
    "catchPhrase": "Multi-layered client-server neural-net",
    "bs": "harness real-time e-markets"
  }
}
```

Les routes imbriquées existent aussi, sur un seul niveau :

```text
GET /posts/1/comments
GET /users/1/posts
GET /users/1/todos
GET /albums/1/photos
```

Et le filtrage par paramètre de requête :

```text
GET /posts?userId=1
GET /comments?postId=1
```

### DummyJSON

Adresse : `https://dummyjson.com`

Gratuite elle aussi, sans clé. Son intérêt : elle **pagine** et elle **cherche**, ce qui en fait le terrain d'entraînement idéal pour les sections 53.34 et 53.35.

```text
GET https://dummyjson.com/products?limit=10&skip=0
GET https://dummyjson.com/products/search?q=phone
```

La réponse a une **enveloppe** : les éléments ne sont pas à la racine, ils sont dans une clé `products`.

```json
{
  "products": [
    {
      "id": 1,
      "title": "Essence Mascara Lash Princess",
      "description": "The Essence Mascara Lash Princess is a popular mascara...",
      "category": "beauty",
      "price": 9.99,
      "rating": 4.94,
      "stock": 5,
      "thumbnail": "https://cdn.dummyjson.com/product-images/..."
    }
  ],
  "total": 194,
  "skip": 0,
  "limit": 10
}
```

Ce format « enveloppe + total + skip + limit » est extrêmement répandu dans les vraies API. Apprenez à le reconnaître.

### Le tableau de comparaison

| | JSONPlaceholder | DummyJSON |
| --- | --- | --- |
| Clé d'API | Aucune | Aucune |
| Forme d'une liste | Tableau JSON à la racine | Objet avec clé `products` |
| Pagination | Non documentée | `?limit=&skip=` |
| Recherche | Non | `/search?q=` |
| Écritures | Simulées | Simulées |
| Usage dans ce chapitre | Sections 53.11 à 53.32 | Sections 53.34 à 53.40 |

JSONPlaceholder est bâtie sur `json-server` et accepte en pratique le paramètre
`_limit` (ainsi que `_start`), pratique pour ne charger que vingt éléments au lieu
de deux cents. Ce paramètre n'est pas documenté officiellement : s'il était ignoré,
vous recevriez simplement la collection entière, sans erreur. Les exemples de ce
chapitre l'utilisent pour alléger les réponses.

> **Prudence en production :** ces deux services sont des bacs à sable publics. On ne construit jamais une application réelle dessus. Ils peuvent être lents, saturés, ou indisponibles. Si un exemple de ce chapitre échoue, ce n'est pas forcément votre code : vérifiez d'abord dans un navigateur que l'URL répond.

---

## 53.9 — Installer `http`

Flutter ne sait pas parler HTTP tout seul de manière portable. Le paquet officiel, maintenu par l'équipe Dart, s'appelle **`http`**.

Depuis la racine de votre projet :

```text
flutter pub add http
```

Cette commande fait deux choses : elle ajoute la ligne dans `pubspec.yaml` **et** elle lance `flutter pub get`.

Votre `pubspec.yaml` contient maintenant :

```yaml
name: mon_application
description: Une application Flutter.
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: ^3.12.0

dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.8
  http: ^1.6.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^6.0.0

flutter:
  uses-material-design: true
```

> La version `^1.6.0` est celle disponible au moment de la rédaction. **Ne recopiez pas ce numéro à l'aveugle.** Lancez `flutter pub add http` et laissez l'outil écrire la version courante. C'est la règle générale de la formation : jamais de version figée à la main.

Rappel du chapitre 16 sur la notation `^1.6.0` : le curseur `^` signifie « toute version compatible », c'est-à-dire `>= 1.6.0 < 2.0.0`. Une version `1.7.0` sera acceptée automatiquement ; une version `2.0.0`, qui pourrait casser votre code, non.

Ensuite, dans votre code Dart, on importe **avec un préfixe** :

```dart
import 'package:http/http.dart' as http;
```

Le préfixe `as http` n'est pas décoratif. Sans lui, le package exposerait des noms très courants (`get`, `post`, `delete`, `Response`, `Client`) directement dans votre fichier, avec un risque élevé de collision. Avec le préfixe, tout est explicite : `http.get`, `http.Response`, `http.Client`. **Tous les exemples de ce chapitre utilisent ce préfixe.**

---

## 53.10 — Les permissions Internet (Android, macOS)

Voici l'un des pièges les plus décourageants pour un débutant : **le code est correct, il marche sur l'émulateur en mode debug, et il échoue une fois publié.** Ou bien il marche sur Android et échoue sur macOS. La cause est presque toujours une permission manquante.

### Android

Android exige une déclaration explicite d'accès à Internet.

Ouvrez `android/app/src/main/AndroidManifest.xml` et ajoutez la ligne **avant** la balise `<application>` :

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:label="mon_application"
        android:name="${applicationName}"
        android:icon="@mipmap/ic_launcher">
        <!-- ... -->
    </application>
</manifest>
```

> **Le piège exact :** Flutter crée un fichier séparé `android/app/src/debug/AndroidManifest.xml` qui contient déjà cette permission, pour que le rechargement à chaud fonctionne. Votre application en mode debug a donc Internet **sans que vous ayez rien fait**. Mais le manifeste de release, lui, ne l'a pas. Résultat : tout fonctionne pendant des semaines de développement, et la version publiée sur le magasin ne charge plus rien. Ajoutez la ligne dans le manifeste `main` dès le premier jour.

### iOS

Rien à faire pour du HTTPS classique. iOS autorise les connexions sécurisées sans déclaration.

En revanche, iOS **bloque le HTTP en clair** par défaut (App Transport Security). Si une API de test n'est accessible qu'en `http://`, il faut une exception dans `ios/Runner/Info.plist` — et c'est fortement déconseillé en production.

### macOS

macOS applique un bac à sable strict. Il faut activer le droit « client réseau » dans **deux** fichiers.

`macos/Runner/DebugProfile.entitlements` :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>com.apple.security.app-sandbox</key>
    <true/>
    <key>com.apple.security.cs.allow-jit</key>
    <true/>
    <key>com.apple.security.network.server</key>
    <true/>
    <key>com.apple.security.network.client</key>
    <true/>
</dict>
</plist>
```

`macos/Runner/Release.entitlements` :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>com.apple.security.app-sandbox</key>
    <true/>
    <key>com.apple.security.network.client</key>
    <true/>
</dict>
</plist>
```

Le fichier `DebugProfile` sert au développement, `Release` à la version publiée. **Oublier le second donne exactement le même symptôme qu'sur Android : ça marche en développement, ça échoue une fois publié.**

### Web, Windows, Linux

Aucune permission à déclarer. En revanche, sur le **web**, une contrainte différente apparaît : le **CORS**. Le navigateur refuse qu'une page servie depuis `localhost:port` appelle `api.exemple.com`, sauf si le serveur renvoie l'en-tête `Access-Control-Allow-Origin`. Ce n'est pas un problème de Flutter et vous ne pouvez pas le régler côté client : c'est au serveur de l'autoriser. JSONPlaceholder et DummyJSON l'autorisent, ce qui permet de tester ce chapitre dans un navigateur.

### Le tableau de contrôle

| Plateforme | Fichier à modifier | Ligne à ajouter |
| --- | --- | --- |
| Android | `android/app/src/main/AndroidManifest.xml` | `<uses-permission android:name="android.permission.INTERNET" />` |
| iOS | Rien (HTTPS) | — |
| macOS | `macos/Runner/DebugProfile.entitlements` **et** `Release.entitlements` | `com.apple.security.network.client` = `true` |
| Web | Rien côté client | Le serveur doit autoriser le CORS |
| Windows / Linux | Rien | — |

---

## 53.11 — `http.get()`

Passons enfin au code. La fonction la plus simple du package :

```dart
Future<http.Response> get(Uri url, {Map<String, String>? headers})
```

Trois observations avant d'écrire une ligne.

1. Le premier paramètre est un **`Uri`**, pas une `String`. Il faut passer par `Uri.parse` ou `Uri.https`.
2. La fonction renvoie un **`Future<http.Response>`**. Elle ne renvoie pas la réponse : elle renvoie la **promesse** d'une réponse. C'est l'asynchrone du chapitre 15.
3. `headers` est facultatif.

Voici le tout premier programme complet. Il n'affiche rien de beau, mais il prouve que le réseau fonctionne.

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() {
  runApp(const ApplicationPremierAppel());
}

class ApplicationPremierAppel extends StatelessWidget {
  const ApplicationPremierAppel({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Premier appel réseau',
      theme: ThemeData(
        colorSchemeSeed: Colors.indigo,
        useMaterial3: true,
      ),
      home: const PagePremierAppel(),
    );
  }
}

class PagePremierAppel extends StatefulWidget {
  const PagePremierAppel({super.key});

  @override
  State<PagePremierAppel> createState() => _PagePremierAppelState();
}

class _PagePremierAppelState extends State<PagePremierAppel> {
  String _resultat = 'Appuyez sur le bouton.';
  bool _enCours = false;

  Future<void> _appeler() async {
    setState(() {
      _enCours = true;
      _resultat = 'Requête en cours...';
    });

    final url = Uri.parse('https://jsonplaceholder.typicode.com/posts/1');
    final reponse = await http.get(url);

    if (!mounted) return;
    setState(() {
      _enCours = false;
      _resultat = 'Code : ${reponse.statusCode}\n\n${reponse.body}';
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('53.11 — http.get()')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            FilledButton.icon(
              onPressed: _enCours ? null : _appeler,
              icon: const Icon(Icons.cloud_download),
              label: const Text('Appeler l\'API'),
            ),
            const SizedBox(height: 24),
            Expanded(
              child: SingleChildScrollView(
                child: Text(
                  _resultat,
                  style: const TextStyle(fontFamily: 'monospace'),
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

**Résultat affiché après appui :**

```text
Code : 200

{
  "userId": 1,
  "id": 1,
  "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
  "body": "quia et suscipit\nsuscipit recusandae consequuntur..."
}
```

Trois détails de ce code méritent qu'on s'y arrête, et ils reviendront dans tout le chapitre.

- `_enCours ? null : _appeler` : passer `null` à `onPressed` **désactive** le bouton. C'est ainsi qu'on empêche l'utilisateur de lancer dix requêtes en tapotant.
- `await http.get(url)` est dans une fonction `async` qui renvoie `Future<void>`. On n'`await` jamais dans `build()`.
- `if (!mounted) return;` : ligne capitale, expliquée en 53.12.1.

---

## 53.12 — La réponse est asynchrone (rappel chapitre 15)

Rappelons le mécanisme du chapitre 15, appliqué au réseau.

Quand vous écrivez :

```dart
final reponse = await http.get(url);
```

il ne se passe **pas** ceci :

```text
MAUVAIS MODÈLE MENTAL

  ligne 1 : http.get(url)   [l'application est gelée 800 ms]
  ligne 2 : suite du code
```

Il se passe ceci :

```text
BON MODÈLE MENTAL

  t=0 ms     http.get(url) part sur le réseau.
             La fonction _appeler() rend la main IMMÉDIATEMENT.
             Flutter continue de dessiner 60 images par seconde.
             L'utilisateur peut faire défiler, appuyer, quitter l'écran.

  t=0→800ms  L'interface est parfaitement fluide.
             Rien n'est bloqué.

  t=800 ms   La réponse arrive. Le Future se complète.
             Le code APRÈS le await reprend, là où il s'était arrêté.
             setState() est appelé, l'écran se reconstruit.
```

Le mot-clé `await` ne bloque pas l'application : il **met en pause la fonction courante** et rend la main à la boucle d'événements. C'est toute la différence.

### Ce qui arrive si vous oubliez `await`

```dart
void main() async {
  Future<String> chercher() async {
    await Future<void>.delayed(const Duration(milliseconds: 300));
    return 'données du serveur';
  }

  // Sans await : on récupère le Future, pas la valeur.
  final a = chercher();
  print('Sans await : $a');

  // Avec await : on récupère la valeur.
  final b = await chercher();
  print('Avec await : $b');
}
```

**Résultat :**

```text
Sans await : Instance of 'Future<String>'
Avec await : données du serveur
```

C'est l'erreur numéro un du débutant en réseau : afficher `Instance of 'Future<...>'` à l'écran. Si vous voyez ce texte, il vous manque un `await`.

---

## 53.12.1 — `mounted` : pourquoi cette ligne est obligatoire

Voici un scénario parfaitement banal :

```text
t=0 ms    L'utilisateur ouvre l'écran « Classement ».
          initState() lance http.get().

t=200 ms  L'utilisateur trouve ça long, appuie sur « retour ».
          L'écran est retiré de l'arbre. dispose() est appelé.
          L'objet State existe encore en mémoire, mais il est MORT.

t=800 ms  La réponse arrive enfin.
          Le code après le await reprend.
          setState() est appelé... sur un State démonté.
```

Flutter lève alors une erreur en console :

```text
setState() called after dispose(): _PageClassementState#a1b2c(lifecycle state: defunct, not mounted)
```

La parade tient en une ligne, à placer **après chaque `await`** et **avant chaque `setState`** :

```dart
if (!mounted) return;
setState(() { /* ... */ });
```

`mounted` est un booléen fourni par la classe `State` (chapitre 45). Il vaut `true` entre `initState()` et `dispose()`, `false` après.

> **Règle absolue :** dans un `State`, tout `setState()` qui suit un `await` doit être précédé d'un test `mounted`. Sans exception. Le même principe vaut pour l'usage d'un `BuildContext` après un `await` (afficher une `SnackBar`, faire un `Navigator.pop`).

---

## 53.13 — `response.statusCode` et `response.body`

L'objet `http.Response` a plusieurs propriétés. Voici celles qui comptent.

| Propriété | Type | Contenu |
| --- | --- | --- |
| `statusCode` | `int` | Le code de statut : 200, 404, 500... |
| `body` | `String` | Le corps décodé en texte |
| `bodyBytes` | `Uint8List` | Le corps brut, en octets |
| `headers` | `Map<String, String>` | Les en-têtes, **clés en minuscules** |
| `reasonPhrase` | `String?` | Le libellé du code : `"OK"`, `"Not Found"` |
| `contentLength` | `int?` | La taille annoncée du corps |
| `isRedirect` | `bool` | Vrai si la réponse était une redirection |
| `request` | `BaseRequest?` | La requête d'origine |

Le programme suivant les affiche toutes.

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() {
  runApp(const ApplicationInspection());
}

class ApplicationInspection extends StatelessWidget {
  const ApplicationInspection({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.teal, useMaterial3: true),
      home: const PageInspection(),
    );
  }
}

class PageInspection extends StatefulWidget {
  const PageInspection({super.key});

  @override
  State<PageInspection> createState() => _PageInspectionState();
}

class _PageInspectionState extends State<PageInspection> {
  String _rapport = 'Choisissez une URL.';

  Future<void> _inspecter(String chemin) async {
    setState(() => _rapport = 'Requête sur $chemin ...');

    final reponse =
        await http.get(Uri.parse('https://jsonplaceholder.typicode.com$chemin'));

    if (!mounted) return;

    final tampon = StringBuffer()
      ..writeln('URL          : $chemin')
      ..writeln('statusCode   : ${reponse.statusCode}')
      ..writeln('reasonPhrase : ${reponse.reasonPhrase}')
      ..writeln('succès       : ${reponse.statusCode >= 200 && reponse.statusCode < 300}')
      ..writeln('content-type : ${reponse.headers['content-type']}')
      ..writeln('contentLength: ${reponse.contentLength}')
      ..writeln('isRedirect   : ${reponse.isRedirect}')
      ..writeln('body (200 c.):')
      ..writeln(reponse.body.length > 200
          ? '${reponse.body.substring(0, 200)}...'
          : reponse.body);

    setState(() => _rapport = tampon.toString());
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('53.13 — Inspecter la réponse')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            Wrap(
              spacing: 8,
              children: [
                OutlinedButton(
                  onPressed: () => _inspecter('/posts/1'),
                  child: const Text('200'),
                ),
                OutlinedButton(
                  onPressed: () => _inspecter('/posts/9999'),
                  child: const Text('404'),
                ),
                OutlinedButton(
                  onPressed: () => _inspecter('/users'),
                  child: const Text('Liste'),
                ),
              ],
            ),
            const SizedBox(height: 16),
            Expanded(
              child: SingleChildScrollView(
                child: Text(
                  _rapport,
                  style: const TextStyle(fontFamily: 'monospace', fontSize: 12),
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

**Résultat pour `/posts/9999` :**

```text
URL          : /posts/9999
statusCode   : 404
reasonPhrase : Not Found
succès       : false
content-type : application/json; charset=utf-8
contentLength: 2
isRedirect   : false
body (200 c.):
{}
```

Notez le point essentiel : **avec un `404`, il y a quand même un corps.** Ici, `{}`. Sur d'autres API, ce sera `{"error":"Not found"}` ou une page HTML complète.

C'est la raison pour laquelle on ne décode **jamais** le corps avant d'avoir vérifié le code de statut. Un `jsonDecode` sur une page HTML d'erreur lève une `FormatException` incompréhensible, et vous perdrez vingt minutes à chercher un bug qui n'existe pas dans votre code.

```text
ORDRE OBLIGATOIRE DE TRAITEMENT D'UNE RÉPONSE

   1. La requête a-t-elle abouti ?        (try / catch réseau)
             │
             v
   2. Le code de statut est-il 2xx ?      (statusCode)
             │
             v
   3. Le corps est-il du JSON valide ?    (try / catch FormatException)
             │
             v
   4. Le JSON a-t-il la forme attendue ?  (vérification des types)
             │
             v
   5. On construit les objets Dart.
```

Sauter une étape, c'est planter en production.

---

## 53.14 — Décoder le JSON (rappel chapitre 17)

Le corps d'une réponse est une **`String`**. Pas une `Map`, pas une `List` : du texte.

```dart
final reponse = await http.get(url);
print(reponse.body.runtimeType);   // String
```

Pour en faire une structure Dart, on utilise `jsonDecode`, de la bibliothèque `dart:convert` (chapitre 17).

```dart
import 'dart:convert';
```

La signature est déroutante au premier abord :

```dart
dynamic jsonDecode(String source)
```

Elle renvoie **`dynamic`**, parce que le contenu dépend du JSON. Deux cas seulement en pratique :

| Le JSON commence par | `jsonDecode` renvoie |
| --- | --- |
| `{` | une `Map<String, dynamic>` |
| `[` | une `List<dynamic>` |

Voici les deux cas, avec les autres types possibles.

```dart
import 'dart:convert';

void main() {
  // Cas 1 : un objet JSON.
  const texteObjet = '{"id": 42, "nom": "Alia", "score": 4200, "actif": true}';
  final objet = jsonDecode(texteObjet);
  print('Type   : ${objet.runtimeType}');
  print('nom    : ${objet['nom']}');
  print('score  : ${objet['score']} (${objet['score'].runtimeType})');

  print('');

  // Cas 2 : un tableau JSON.
  const texteTableau = '[{"nom":"Alia"},{"nom":"Baltus"}]';
  final tableau = jsonDecode(texteTableau);
  print('Type   : ${tableau.runtimeType}');
  print('Taille : ${tableau.length}');
  print('1er    : ${tableau[0]['nom']}');

  print('');

  // Cas 3 : les correspondances de types.
  const varie = '{"e":1,"d":1.5,"s":"a","b":true,"n":null,"l":[1,2],"o":{"x":1}}';
  final m = jsonDecode(varie) as Map<String, dynamic>;
  m.forEach((cle, valeur) {
    print('$cle -> $valeur  (${valeur.runtimeType})');
  });
}
```

**Résultat :**

```text
Type   : _Map<String, dynamic>
nom    : Alia
score  : 4200 (int)

Type   : List<dynamic>
Taille : 2
1er    : Alia

e -> 1  (int)
d -> 1.5  (double)
s -> a  (String)
b -> true  (bool)
n -> null  (Null)
l -> [1, 2]  (List<dynamic>)
o -> {x: 1}  (_Map<String, dynamic>)
```

### La table de correspondance JSON → Dart

| JSON | Dart après `jsonDecode` |
| --- | --- |
| `{ ... }` | `Map<String, dynamic>` |
| `[ ... ]` | `List<dynamic>` |
| `"texte"` | `String` |
| `42` | `int` |
| `1.5` | `double` |
| `true` / `false` | `bool` |
| `null` | `Null` |

### Le piège des nombres

C'est le piège JSON le plus fréquent en Flutter. Observez :

```dart
import 'dart:convert';

void main() {
  final a = jsonDecode('{"prix": 10}') as Map<String, dynamic>;
  final b = jsonDecode('{"prix": 10.0}') as Map<String, dynamic>;

  print('${a['prix']} est un ${a['prix'].runtimeType}');
  print('${b['prix']} est un ${b['prix'].runtimeType}');

  // Ce cast plante si le serveur renvoie 10 au lieu de 10.0.
  try {
    final prix = a['prix'] as double;
    print(prix);
  } catch (e) {
    print('ERREUR : $e');
  }

  // La parade universelle.
  final sur = (a['prix'] as num).toDouble();
  print('Robuste : $sur (${sur.runtimeType})');
}
```

**Résultat :**

```text
10 est un int
10.0 est un double
ERREUR : type 'int' is not a subtype of type 'double' in type cast
Robuste : 10.0 (double)
```

Un serveur qui renvoie `9.99` puis, un autre jour, `10` (parce que la décimale est nulle) fera planter votre application chez certains utilisateurs seulement. Impossible à reproduire, très long à diagnostiquer.

> **Règle :** pour tout champ numérique décimal, écrivez `(json['prix'] as num).toDouble()`. Jamais `as double`.

### Le piège de l'encodage

`response.body` décode les octets en **latin-1** par défaut si le serveur n'annonce pas de charset. Résultat : les accents s'affichent en `Ã©` au lieu de `é`.

La parade, quand vous savez que le serveur renvoie de l'UTF-8 :

```dart
final donnees = jsonDecode(utf8.decode(reponse.bodyBytes));
```

`bodyBytes` donne les octets bruts, `utf8.decode` les interprète correctement. Si vous voyez `MystÃ¨re` à l'écran, c'est ce correctif qu'il vous faut.

---

## 53.15 — Du JSON au modèle Dart : `fromJson()`

On pourrait s'arrêter à `jsonDecode` et manipuler des `Map` partout :

```dart
Text(donnees['title'])
```

C'est une très mauvaise idée, pour cinq raisons.

1. **Aucune autocomplétion.** L'éditeur ne peut rien vous proposer sur une `Map<String, dynamic>`.
2. **Aucune vérification à la compilation.** `donnees['titel']` compile parfaitement et plante à l'exécution.
3. **Le type est `dynamic`.** Vous pouvez additionner un titre et une date sans que rien ne proteste avant le crash.
4. **La structure du JSON se répand partout.** Si le serveur renomme `title` en `heading`, vous devez chercher dans tout le projet.
5. **Impossible de documenter.** Personne ne sait quels champs existent.

La solution est celle du chapitre 17 : une **classe Dart** avec un constructeur nommé `fromJson`.

```dart
import 'dart:convert';

class Article {
  final int id;
  final int idUtilisateur;
  final String titre;
  final String corps;

  const Article({
    required this.id,
    required this.idUtilisateur,
    required this.titre,
    required this.corps,
  });

  factory Article.fromJson(Map<String, dynamic> json) {
    return Article(
      id: json['id'] as int,
      idUtilisateur: json['userId'] as int,
      titre: json['title'] as String,
      corps: json['body'] as String,
    );
  }

  Map<String, dynamic> toJson() => {
        'id': id,
        'userId': idUtilisateur,
        'title': titre,
        'body': corps,
      };

  @override
  String toString() => 'Article($id, "$titre")';
}

void main() {
  const texte = '''
  {
    "userId": 1,
    "id": 1,
    "title": "L'épée de Cendre",
    "body": "Une lame forgée dans les cendres du volcan."
  }
  ''';

  final json = jsonDecode(texte) as Map<String, dynamic>;
  final article = Article.fromJson(json);

  print(article);
  print('Titre : ${article.titre}');
  print('Auteur : ${article.idUtilisateur}');
  print('Re-encodé : ${jsonEncode(article.toJson())}');
}
```

**Résultat :**

```text
Article(1, "L'épée de Cendre")
Titre : L'épée de Cendre
Auteur : 1
Re-encodé : {"id":1,"userId":1,"title":"L'épée de Cendre","body":"Une lame forgée dans les cendres du volcan."}
```

Remarquez trois choix de conception.

- Le constructeur `fromJson` est un **`factory`**. Ce n'est pas obligatoire, mais c'est la convention : cela signale qu'il peut faire des calculs, choisir une sous-classe ou renvoyer une instance mise en cache.
- Les **noms Dart diffèrent des noms JSON** : `userId` devient `idUtilisateur`, `title` devient `titre`. C'est justement l'intérêt : le vocabulaire de l'API reste enfermé dans `fromJson`.
- `toJson()` existe pour le chemin inverse, indispensable en 53.29 pour envoyer des données.

### Un `fromJson` défensif

Le code ci-dessus suppose que le serveur est parfait. Il ne l'est jamais. Voici la version blindée, à utiliser quand vous ne maîtrisez pas l'API.

```dart
import 'dart:convert';

class Produit {
  final int id;
  final String titre;
  final double prix;
  final String? vignette;
  final List<String> etiquettes;

  const Produit({
    required this.id,
    required this.titre,
    required this.prix,
    this.vignette,
    this.etiquettes = const [],
  });

  factory Produit.fromJson(Map<String, dynamic> json) {
    return Produit(
      // Un entier absent devient 0 au lieu de faire planter.
      id: (json['id'] as num?)?.toInt() ?? 0,
      // Une chaîne absente devient un libellé de secours.
      titre: json['title'] as String? ?? 'Sans titre',
      // num -> double : accepte 10 comme 10.0.
      prix: (json['price'] as num?)?.toDouble() ?? 0.0,
      // Champ facultatif : on le garde nullable.
      vignette: json['thumbnail'] as String?,
      // Liste absente -> liste vide, jamais null.
      etiquettes: (json['tags'] as List<dynamic>?)
              ?.map((e) => e.toString())
              .toList() ??
          const [],
    );
  }

  @override
  String toString() =>
      'Produit(#$id, "$titre", $prix EUR, ${etiquettes.length} étiquette(s))';
}

void main() {
  // Cas 1 : JSON complet.
  const complet = '''
  {"id": 1, "title": "Potion de soin", "price": 9.99,
   "thumbnail": "https://example.com/p.png", "tags": ["soin", "consommable"]}
  ''';
  print(Produit.fromJson(jsonDecode(complet) as Map<String, dynamic>));

  // Cas 2 : JSON incomplet — l'application ne plante pas.
  const partiel = '{"id": 2, "price": 15}';
  print(Produit.fromJson(jsonDecode(partiel) as Map<String, dynamic>));

  // Cas 3 : JSON vide.
  print(Produit.fromJson(jsonDecode('{}') as Map<String, dynamic>));
}
```

**Résultat :**

```text
Produit(#1, "Potion de soin", 9.99 EUR, 2 étiquette(s))
Produit(#2, "Sans titre", 15.0 EUR, 0 étiquette(s))
Produit(#0, "Sans titre", 0.0 EUR, 0 étiquette(s))
```

Aucun plantage sur les trois cas. Les techniques employées :

| Technique | Effet |
| --- | --- |
| `as num?` puis `?.toInt()` | Accepte `int` comme `double`, tolère l'absence |
| `as String? ?? 'défaut'` | Valeur de secours si le champ manque |
| `as List<dynamic>?` puis `?? const []` | Une liste vide plutôt que `null` |
| `.map((e) => e.toString())` | Convertit sans supposer le type des éléments |

> **Le principe :** un `fromJson` ne doit **jamais** faire planter l'application. Il doit produire un objet dégradé mais utilisable, ou lever une exception claire que vous attraperez. Un `type 'Null' is not a subtype of type 'String'` remonté jusqu'à l'écran est un échec de conception.

---

## 53.15.1 — Les champs imbriqués

Le JSON de `/users` de JSONPlaceholder contient des objets dans l'objet. On crée alors **une classe par niveau**.

```dart
import 'dart:convert';

class Geo {
  final String latitude;
  final String longitude;

  const Geo({required this.latitude, required this.longitude});

  factory Geo.fromJson(Map<String, dynamic> json) => Geo(
        latitude: json['lat'] as String? ?? '0',
        longitude: json['lng'] as String? ?? '0',
      );

  @override
  String toString() => '($latitude, $longitude)';
}

class Adresse {
  final String rue;
  final String ville;
  final String codePostal;
  final Geo geo;

  const Adresse({
    required this.rue,
    required this.ville,
    required this.codePostal,
    required this.geo,
  });

  factory Adresse.fromJson(Map<String, dynamic> json) => Adresse(
        rue: json['street'] as String? ?? '',
        ville: json['city'] as String? ?? '',
        codePostal: json['zipcode'] as String? ?? '',
        // On descend d'un niveau : la valeur est elle-même une Map.
        geo: Geo.fromJson((json['geo'] as Map<String, dynamic>?) ?? const {}),
      );

  @override
  String toString() => '$rue, $codePostal $ville $geo';
}

class Utilisateur {
  final int id;
  final String nom;
  final String courriel;
  final Adresse adresse;

  const Utilisateur({
    required this.id,
    required this.nom,
    required this.courriel,
    required this.adresse,
  });

  factory Utilisateur.fromJson(Map<String, dynamic> json) => Utilisateur(
        id: (json['id'] as num?)?.toInt() ?? 0,
        nom: json['name'] as String? ?? 'Inconnu',
        courriel: json['email'] as String? ?? '',
        adresse:
            Adresse.fromJson((json['address'] as Map<String, dynamic>?) ?? const {}),
      );

  @override
  String toString() => '#$id $nom <$courriel>\n   $adresse';
}

void main() {
  const texte = '''
  {
    "id": 1,
    "name": "Leanne Graham",
    "email": "Sincere@april.biz",
    "address": {
      "street": "Kulas Light",
      "city": "Gwenborough",
      "zipcode": "92998-3874",
      "geo": { "lat": "-37.3159", "lng": "81.1496" }
    }
  }
  ''';

  final u = Utilisateur.fromJson(jsonDecode(texte) as Map<String, dynamic>);
  print(u);

  // Et si « address » manque complètement ?
  final v = Utilisateur.fromJson(jsonDecode('{"id":2,"name":"X"}') as Map<String, dynamic>);
  print(v);
}
```

**Résultat :**

```text
#1 Leanne Graham <Sincere@april.biz>
   Kulas Light, 92998-3874 Gwenborough (-37.3159, 81.1496)
#2 X <>
   ,  (0, 0)
```

La règle est mécanique : **un objet JSON imbriqué = une classe Dart de plus, avec son propre `fromJson`.** On ne va jamais chercher `json['address']['geo']['lat']` directement : trois occasions de plantage sur une seule ligne.

---

## 53.16 — Une liste d'objets depuis un tableau JSON

C'est le cas le plus fréquent : le serveur renvoie un tableau, vous voulez une `List<Article>`.

L'opération se fait en trois temps.

```text
   "[{...},{...},{...}]"          String  (response.body)
              │
              │  jsonDecode
              v
   [ {..}, {..}, {..} ]           List<dynamic>
              │
              │  .map((e) => Article.fromJson(e as Map<String, dynamic>))
              v
   ( Article, Article, Article )  Iterable<Article>   (paresseux !)
              │
              │  .toList()
              v
   [ Article, Article, Article ]  List<Article>
```

Attention à la dernière étape : `.map()` renvoie un `Iterable` **paresseux** (chapitre 14). Sans `.toList()`, la conversion n'est pas encore faite, et un widget de liste la referait à chaque construction.

```dart
import 'dart:convert';

class Article {
  final int id;
  final String titre;

  const Article({required this.id, required this.titre});

  factory Article.fromJson(Map<String, dynamic> json) => Article(
        id: (json['id'] as num?)?.toInt() ?? 0,
        titre: json['title'] as String? ?? '',
      );

  @override
  String toString() => '#$id ${titre.length > 30 ? '${titre.substring(0, 30)}...' : titre}';
}

void main() {
  const texte = '''
  [
    {"userId":1,"id":1,"title":"sunt aut facere repellat provident"},
    {"userId":1,"id":2,"title":"qui est esse"},
    {"userId":1,"id":3,"title":"ea molestias quasi exercitationem"}
  ]
  ''';

  // Étape 1 : décoder.
  final brut = jsonDecode(texte);
  print('Après jsonDecode : ${brut.runtimeType}');

  // Étape 2 : caster en List<dynamic>.
  final liste = brut as List<dynamic>;
  print('Taille : ${liste.length}');

  // Étape 3 : convertir chaque élément.
  final articles = liste
      .map((e) => Article.fromJson(e as Map<String, dynamic>))
      .toList();

  print('Type final : ${articles.runtimeType}');
  for (final a in articles) {
    print('  $a');
  }
}
```

**Résultat :**

```text
Après jsonDecode : List<dynamic>
Taille : 3
Type final : List<Article>
  #1 sunt aut facere repellat provi...
  #2 qui est esse
  #3 ea molestias quasi exercitatio...
```

### La forme condensée

Une fois le mécanisme compris, on écrit la ligne unique. C'est la formule à mémoriser :

```dart
final articles = (jsonDecode(reponse.body) as List<dynamic>)
    .map((e) => Article.fromJson(e as Map<String, dynamic>))
    .toList();
```

### Quand la liste est enveloppée

DummyJSON ne renvoie pas un tableau à la racine, mais un objet contenant le tableau. Il faut alors descendre d'un cran.

```dart
import 'dart:convert';

class Produit {
  final int id;
  final String titre;
  final double prix;

  const Produit({required this.id, required this.titre, required this.prix});

  factory Produit.fromJson(Map<String, dynamic> json) => Produit(
        id: (json['id'] as num?)?.toInt() ?? 0,
        titre: json['title'] as String? ?? '',
        prix: (json['price'] as num?)?.toDouble() ?? 0,
      );

  @override
  String toString() => '#$id $titre — $prix EUR';
}

/// Une page de résultats : les éléments et les compteurs de pagination.
class PageProduits {
  final List<Produit> produits;
  final int total;
  final int saut;
  final int limite;

  const PageProduits({
    required this.produits,
    required this.total,
    required this.saut,
    required this.limite,
  });

  factory PageProduits.fromJson(Map<String, dynamic> json) => PageProduits(
        produits: (json['products'] as List<dynamic>? ?? const [])
            .map((e) => Produit.fromJson(e as Map<String, dynamic>))
            .toList(),
        total: (json['total'] as num?)?.toInt() ?? 0,
        saut: (json['skip'] as num?)?.toInt() ?? 0,
        limite: (json['limit'] as num?)?.toInt() ?? 0,
      );
}

void main() {
  const texte = '''
  {
    "products": [
      {"id": 1, "title": "Mascara", "price": 9.99},
      {"id": 2, "title": "Crème",   "price": 19}
    ],
    "total": 194,
    "skip": 0,
    "limit": 2
  }
  ''';

  final page = PageProduits.fromJson(jsonDecode(texte) as Map<String, dynamic>);
  print('Total serveur : ${page.total}');
  print('Reçus         : ${page.produits.length}');
  print('Saut / limite : ${page.saut} / ${page.limite}');
  for (final p in page.produits) {
    print('  $p');
  }
}
```

**Résultat :**

```text
Total serveur : 194
Reçus         : 2
Saut / limite : 0 / 2
  #1 Mascara — 9.99 EUR
  #2 Crème — 19.0 EUR
```

Notez le prix `19` du JSON, reçu comme `int`, converti en `19.0` grâce à `as num?)?.toDouble()`. C'est exactement le piège de la section 53.14 — et il est neutralisé.

---

## 53.17 — La couche service : séparer le réseau de l'interface

Voici le code que tout débutant écrit :

```dart
// À NE PAS FAIRE
class _MaPageState extends State<MaPage> {
  List<Article> _articles = [];

  Future<void> _charger() async {
    final r = await http.get(Uri.parse('https://jsonplaceholder.typicode.com/posts'));
    setState(() {
      _articles = (jsonDecode(r.body) as List)
          .map((e) => Article.fromJson(e))
          .toList();
    });
  }
  // ... 200 lignes de widgets
}
```

Ça fonctionne. Et c'est un problème pour six raisons.

1. **L'URL est perdue au milieu des widgets.** Quand l'API changera d'adresse, il faudra fouiller chaque écran.
2. **Impossible à tester.** Pour tester ce décodage, il faudrait démarrer une application Flutter complète.
3. **Impossible à réutiliser.** Un deuxième écran qui a besoin des mêmes articles recopiera tout.
4. **Les erreurs ne sont pas gérées.** Une coupure réseau fait planter l'écran.
5. **Le widget mélange trois métiers :** réseau, transformation de données, affichage.
6. **Le code est illisible.** On ne voit plus la structure de l'écran.

La solution est le découpage en **couches**, vu au chapitre 16 pour l'organisation d'un projet Dart.

```text
┌───────────────────────────────────────────────────────────┐
│  COUCHE PRÉSENTATION      lib/ecrans/ , lib/widgets/       │
│  Widgets, mise en page, boutons, animations.               │
│  Ne connaît ni URL, ni JSON, ni http.                      │
└──────────────────────────┬────────────────────────────────┘
                           │  appelle
                           v
┌───────────────────────────────────────────────────────────┐
│  COUCHE SERVICE           lib/services/                    │
│  Construit les URL, appelle http, vérifie le statut,       │
│  décode le JSON, transforme les erreurs techniques         │
│  en exceptions métier.                                     │
│  Renvoie : Future<List<Article>>. Aucun Widget.            │
└──────────────────────────┬────────────────────────────────┘
                           │  produit
                           v
┌───────────────────────────────────────────────────────────┐
│  COUCHE MODÈLE            lib/modeles/                     │
│  Classes pures : Article, Produit, Utilisateur.            │
│  fromJson / toJson. Aucun import de Flutter.               │
└───────────────────────────────────────────────────────────┘
```

Une seule règle, mais impérative : **les flèches ne remontent jamais.** Un modèle n'importe pas un service. Un service n'importe pas `material.dart`.

L'arborescence correspondante :

```text
lib/
├── main.dart
├── modeles/
│   └── article.dart
├── services/
│   └── service_articles.dart
└── ecrans/
    └── ecran_articles.dart
```

Écrivons ce service. Une exception métier d'abord, pour ne jamais laisser fuiter les détails techniques vers l'interface.

```dart
/// Exception métier : tout ce qui empêche d'obtenir des articles.
/// L'interface n'a pas à savoir si c'était un SocketException,
/// un 500 ou un JSON malformé. Elle a besoin d'un message affichable.
class ErreurApi implements Exception {
  final String message;
  final int? codeStatut;

  const ErreurApi(this.message, {this.codeStatut});

  bool get estReseau => codeStatut == null;
  bool get estAuthentification => codeStatut == 401 || codeStatut == 403;
  bool get estIntrouvable => codeStatut == 404;
  bool get estServeur => codeStatut != null && codeStatut! >= 500;

  @override
  String toString() => message;
}
```

Et le service :

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class ServiceArticles {
  /// Le client est injecté : c'est ce qui rendra le service testable (53.39).
  final http.Client _client;
  final Uri _base;

  ServiceArticles({http.Client? client, Uri? base})
      : _client = client ?? http.Client(),
        _base = base ?? Uri.parse('https://jsonplaceholder.typicode.com');

  Future<List<Article>> listerArticles({int? idUtilisateur}) async {
    final url = Uri.https(
      _base.host,
      '/posts',
      idUtilisateur == null ? null : {'userId': '$idUtilisateur'},
    );

    final reponse = await _client.get(url, headers: const {
      'Accept': 'application/json',
    });

    if (reponse.statusCode < 200 || reponse.statusCode >= 300) {
      throw ErreurApi(
        'Le serveur a répondu ${reponse.statusCode}.',
        codeStatut: reponse.statusCode,
      );
    }

    final brut = jsonDecode(utf8.decode(reponse.bodyBytes));
    if (brut is! List) {
      throw const ErreurApi('Réponse inattendue : un tableau était attendu.');
    }

    return brut
        .map((e) => Article.fromJson(e as Map<String, dynamic>))
        .toList();
  }

  void fermer() => _client.close();
}
```

Regardez la signature de `listerArticles` : `Future<List<Article>>`. Elle ne dit **rien** de HTTP, rien de JSON, rien du serveur. L'écran qui l'appelle n'a aucune raison de savoir d'où viennent les données. Demain, vous remplacez l'API par une base locale : l'écran ne change pas d'une ligne.

> **`http.Client` plutôt que `http.get`.** Les fonctions de haut niveau `http.get()`, `http.post()` ouvrent une connexion, l'utilisent, la ferment. Pour plusieurs requêtes vers le même serveur, un `http.Client` réutilise la connexion : c'est notablement plus rapide. Et surtout, un `Client` peut être remplacé par un faux client en test (section 53.39). N'oubliez pas `close()` quand vous avez fini.

---

## 53.18 — Gérer les erreurs réseau (rappel chapitre 13)

Reprenons le chapitre 13 et appliquons-le au réseau.

Une requête HTTP peut échouer de **quatre** manières différentes, et elles ne se traitent pas pareil.

```text
   ÉCHEC 1 : LA REQUÊTE NE PART PAS OU N'ARRIVE PAS
   ─────────────────────────────────────────────────
   Avion, tunnel, wifi coupé, DNS injoignable, serveur éteint.
   -> Une EXCEPTION est levée. Il n'y a AUCUNE réponse.
   -> SocketException, ClientException.

   ÉCHEC 2 : LA RÉPONSE MET TROP DE TEMPS
   ─────────────────────────────────────────────────
   Réseau 2G, serveur surchargé.
   -> TimeoutException, si et seulement si VOUS avez posé un .timeout().
      Sans timeout, l'application attend indéfiniment.

   ÉCHEC 3 : LE SERVEUR RÉPOND, MAIS AVEC UNE ERREUR
   ─────────────────────────────────────────────────
   404, 401, 500...
   -> AUCUNE exception n'est levée ! http considère avoir fait son travail.
   -> C'est à VOUS de tester statusCode.

   ÉCHEC 4 : LA RÉPONSE EST INEXPLOITABLE
   ─────────────────────────────────────────────────
   HTML au lieu de JSON, champ manquant, type inattendu.
   -> FormatException, TypeError, NoSuchMethodError.
```

**L'échec 3 est celui qu'on oublie systématiquement.** Répétons-le : un code `500` ne lève **pas** d'exception. `http.get` a rempli son contrat : il a obtenu une réponse. Que cette réponse annonce une catastrophe est votre affaire.

Le squelette complet de traitement :

```dart
Future<List<Article>> listerArticles() async {
  try {
    final reponse = await _client
        .get(url)
        .timeout(const Duration(seconds: 10));          // échecs 1 et 2

    if (reponse.statusCode < 200 || reponse.statusCode >= 300) {
      throw ErreurApi('Erreur ${reponse.statusCode}',    // échec 3
          codeStatut: reponse.statusCode);
    }

    final brut = jsonDecode(reponse.body);               // échec 4
    if (brut is! List) {
      throw const ErreurApi('Format inattendu.');
    }
    return brut.map((e) => Article.fromJson(e as Map<String, dynamic>)).toList();

  } on SocketException {
    throw const ErreurApi('Pas de connexion Internet.');
  } on TimeoutException {
    throw const ErreurApi('Le serveur met trop de temps à répondre.');
  } on FormatException {
    throw const ErreurApi('Réponse illisible du serveur.');
  } on http.ClientException {
    throw const ErreurApi('La connexion a été interrompue.');
  }
}
```

Deux principes de conception, à retenir.

**Principe 1 — Le service traduit, il n'avale pas.**
Chaque `catch` relance une `ErreurApi` avec un message compréhensible. Il ne renvoie **jamais** une liste vide en silence : une liste vide veut dire « le serveur n'a rien », pas « le réseau est coupé ». Confondre les deux est un bug classique.

**Principe 2 — L'interface ne connaît qu'un seul type d'erreur.**
L'écran attrape `ErreurApi` et affiche `erreur.message`. Il n'a jamais à importer `dart:io` ni à connaître le nom `SocketException`.

### Ce qu'il ne faut jamais écrire

```dart
// ANTI-PATRON 1 : avaler l'erreur.
try {
  return await charger();
} catch (e) {
  return [];   // l'utilisateur voit une liste vide et croit qu'il n'y a rien
}

// ANTI-PATRON 2 : afficher l'exception brute.
Text('$e')
// -> « SocketException: Failed host lookup: 'api.exemple.com'
//      (OS Error: nodename nor servname provided, errno = 8) »
// Incompréhensible pour un utilisateur.

// ANTI-PATRON 3 : catch vide.
try {
  await envoyer();
} catch (_) {}   // le bug devient invisible pour toujours
```

---

## 53.19 — `SocketException`, `TimeoutException`, `FormatException`

Détaillons les exceptions que vous allez réellement rencontrer, avec leur provenance et leur message typique.

| Exception | Import | Quand | Message typique |
| --- | --- | --- | --- |
| `SocketException` | `dart:io` | Hôte introuvable, réseau coupé, connexion refusée | `Failed host lookup: 'api.exemple.com'` |
| `TimeoutException` | `dart:async` | Le `.timeout()` a expiré | `TimeoutException after 0:00:10.000000` |
| `FormatException` | `dart:core` | `jsonDecode` sur du non-JSON | `Unexpected character (at character 1)` |
| `http.ClientException` | `package:http` | Connexion interrompue en cours, erreur du client | `Connection closed before full header was received` |
| `HandshakeException` | `dart:io` | Certificat TLS invalide ou expiré | `CERTIFICATE_VERIFY_FAILED` |
| `TypeError` | `dart:core` | Un cast `as` a échoué | `type 'Null' is not a subtype of type 'String'` |
| `_TypeError` sur `[]` | — | Accès à une clé sur `null` | `NoSuchMethodError: '[]' called on null` |

### Le piège de `SocketException` sur le web

`SocketException` vient de `dart:io`. Or **`dart:io` n'existe pas sur Flutter web.** Un fichier qui écrit `import 'dart:io';` ne compilera pas pour le navigateur.

Deux stratégies :

**Stratégie A — attraper `ClientException`, qui est portable.**

Le package `http` enveloppe les erreurs de bas niveau dans `ClientException` sur toutes les plateformes.

```dart
} on http.ClientException catch (e) {
  throw ErreurApi('Connexion impossible : ${e.message}');
}
```

**Stratégie B — attraper largement et filtrer sur le nom du type.**

```dart
} catch (e) {
  final nom = e.runtimeType.toString();
  if (nom.contains('SocketException') || e is http.ClientException) {
    throw const ErreurApi('Pas de connexion Internet.');
  }
  rethrow;
}
```

Pour une application uniquement mobile, l'import de `dart:io` est parfaitement acceptable et plus lisible. **Tous les exemples de ce chapitre ciblent le mobile et importent `dart:io`.** Si vous visez aussi le web, remplacez `on SocketException` par `on http.ClientException`.

### Les voir toutes en action

Ce programme provoque volontairement chacune des erreurs.

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() {
  runApp(const ApplicationErreurs());
}

class ApplicationErreurs extends StatelessWidget {
  const ApplicationErreurs({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.red, useMaterial3: true),
      home: const PageErreurs(),
    );
  }
}

class PageErreurs extends StatefulWidget {
  const PageErreurs({super.key});

  @override
  State<PageErreurs> createState() => _PageErreursState();
}

class _PageErreursState extends State<PageErreurs> {
  final List<String> _journal = [];

  void _noter(String ligne) {
    if (!mounted) return;
    setState(() => _journal.insert(0, ligne));
  }

  /// Cas 1 : un nom de domaine qui n'existe pas.
  Future<void> _hoteInexistant() async {
    _noter('--- Hôte inexistant ---');
    try {
      await http.get(Uri.parse('https://ce-domaine-nexiste-vraiment-pas-42.dev'));
      _noter('Aucune erreur (inattendu).');
    } on SocketException catch (e) {
      _noter('SocketException : ${e.message}');
    } on http.ClientException catch (e) {
      _noter('ClientException : ${e.message}');
    } catch (e) {
      _noter('${e.runtimeType} : $e');
    }
  }

  /// Cas 2 : un délai volontairement trop court.
  Future<void> _delaiDepasse() async {
    _noter('--- Délai dépassé ---');
    try {
      await http
          .get(Uri.parse('https://jsonplaceholder.typicode.com/photos'))
          .timeout(const Duration(milliseconds: 1));
      _noter('Arrivé à temps (réseau très rapide).');
    } on TimeoutException catch (e) {
      _noter('TimeoutException : ${e.duration}');
    } catch (e) {
      _noter('${e.runtimeType} : $e');
    }
  }

  /// Cas 3 : décoder du HTML comme si c'était du JSON.
  Future<void> _jsonInvalide() async {
    _noter('--- JSON invalide ---');
    const html = '<!DOCTYPE html><html><body>503 Service Unavailable</body></html>';
    try {
      jsonDecode(html);
      _noter('Décodé (inattendu).');
    } on FormatException catch (e) {
      _noter('FormatException : ${e.message}');
    }
  }

  /// Cas 4 : un statut d'erreur qui ne lève AUCUNE exception.
  Future<void> _statutErreur() async {
    _noter('--- Statut 404 ---');
    final r = await http.get(
      Uri.parse('https://jsonplaceholder.typicode.com/posts/999999'),
    );
    _noter('Aucune exception levée. statusCode = ${r.statusCode}');
    _noter('Corps reçu : "${r.body}"');
  }

  /// Cas 5 : un cast qui échoue sur un champ absent.
  Future<void> _castRate() async {
    _noter('--- Cast raté ---');
    final json = jsonDecode('{"id":1}') as Map<String, dynamic>;
    try {
      final titre = json['title'] as String;
      _noter('Titre : $titre');
    } catch (e) {
      _noter('${e.runtimeType} : $e');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('53.19 — Les erreurs réseau')),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(12),
            child: Wrap(
              spacing: 8,
              runSpacing: 8,
              children: [
                OutlinedButton(onPressed: _hoteInexistant, child: const Text('Hôte')),
                OutlinedButton(onPressed: _delaiDepasse, child: const Text('Délai')),
                OutlinedButton(onPressed: _jsonInvalide, child: const Text('JSON')),
                OutlinedButton(onPressed: _statutErreur, child: const Text('404')),
                OutlinedButton(onPressed: _castRate, child: const Text('Cast')),
                TextButton(
                  onPressed: () => setState(_journal.clear),
                  child: const Text('Effacer'),
                ),
              ],
            ),
          ),
          const Divider(height: 1),
          Expanded(
            child: ListView.builder(
              itemCount: _journal.length,
              itemBuilder: (context, i) => Padding(
                padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 4),
                child: Text(
                  _journal[i],
                  style: const TextStyle(fontFamily: 'monospace', fontSize: 12),
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

**Résultat (journal, du plus récent au plus ancien) :**

```text
Corps reçu : "{}"
Aucune exception levée. statusCode = 404
--- Statut 404 ---
FormatException : Unexpected character
--- JSON invalide ---
TimeoutException : 0:00:00.001000
--- Délai dépassé ---
SocketException : Failed host lookup: 'ce-domaine-nexiste-vraiment-pas-42.dev'
--- Hôte inexistant ---
```

La ligne à retenir de tout ce programme :

```text
Aucune exception levée. statusCode = 404
```

---

## 53.20 — Le délai d'attente (`timeout`)

Par défaut, une requête HTTP **n'a pas de limite de temps**. Si le serveur accepte la connexion puis ne répond jamais, votre `Future` reste en attente pour toujours. L'utilisateur voit tourner un indicateur de chargement, indéfiniment. C'est l'un des pires bugs d'expérience utilisateur, parce qu'il ne produit aucune erreur en console.

La parade est la méthode `.timeout()`, disponible sur **tous** les `Future` (chapitre 15).

```dart
Future<T> timeout(Duration timeLimit, {FutureOr<T> Function()? onTimeout})
```

Deux usages.

**Usage 1 — sans `onTimeout` : une `TimeoutException` est levée.**

```dart
final reponse = await http
    .get(url)
    .timeout(const Duration(seconds: 10));
```

**Usage 2 — avec `onTimeout` : on fournit une valeur de repli.**

```dart
final reponse = await http.get(url).timeout(
      const Duration(seconds: 10),
      onTimeout: () => http.Response('[]', 408),
    );
```

Le second usage est à manier avec précaution : il masque le problème. Préférez le premier, et affichez un message à l'utilisateur.

### Quelle durée choisir ?

| Type d'appel | Durée conseillée | Raison |
| --- | --- | --- |
| Suggestion de recherche (autocomplétion) | 3 s | Au-delà, l'utilisateur a fini de taper |
| Chargement de liste classique | 10 s | Compromis usuel |
| Envoi de formulaire | 15 s | On ne veut pas perdre la saisie |
| Envoi d'un fichier | 60 s et plus | Dépend de la taille |
| Synchronisation en arrière-plan | 30 s | Personne n'attend devant l'écran |

> **Attention :** `.timeout()` interrompt **votre attente**, pas la requête. La connexion réseau continue en arrière-plan jusqu'à ce que le système d'exploitation la ferme. Pour vraiment annuler une requête, il faut fermer le `Client` (section 53.36) ou utiliser un `CancelToken` de `dio` (section 53.37).

### Un timeout global sur toutes vos requêtes

Plutôt que de répéter `.timeout(...)` partout, on centralise dans le service.

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';
import 'package:http/http.dart' as http;

class ErreurApi implements Exception {
  final String message;
  final int? codeStatut;
  const ErreurApi(this.message, {this.codeStatut});
  @override
  String toString() => message;
}

/// Un service de base : toutes les méthodes passent par _obtenir(),
/// qui applique le même timeout et la même traduction d'erreurs.
class ServiceHttp {
  final http.Client _client;
  final String _hote;
  final Duration _delai;

  ServiceHttp({
    http.Client? client,
    String hote = 'jsonplaceholder.typicode.com',
    Duration delai = const Duration(seconds: 10),
  })  : _client = client ?? http.Client(),
        _hote = hote,
        _delai = delai;

  Future<dynamic> obtenirJson(
    String chemin, {
    Map<String, String>? parametres,
  }) async {
    final url = Uri.https(_hote, chemin, parametres);

    try {
      final reponse = await _client.get(
        url,
        headers: const {'Accept': 'application/json'},
      ).timeout(_delai);

      if (reponse.statusCode < 200 || reponse.statusCode >= 300) {
        throw ErreurApi(
          _messagePourStatut(reponse.statusCode),
          codeStatut: reponse.statusCode,
        );
      }

      return jsonDecode(utf8.decode(reponse.bodyBytes));
    } on TimeoutException {
      throw ErreurApi('Le serveur met plus de ${_delai.inSeconds} s à répondre.');
    } on SocketException {
      throw const ErreurApi('Aucune connexion Internet.');
    } on http.ClientException {
      throw const ErreurApi('La connexion a été interrompue.');
    } on FormatException {
      throw const ErreurApi('Le serveur a renvoyé une réponse illisible.');
    }
  }

  static String _messagePourStatut(int code) {
    switch (code) {
      case 400:
        return 'Requête incorrecte.';
      case 401:
        return 'Vous devez vous reconnecter.';
      case 403:
        return 'Accès refusé.';
      case 404:
        return 'Ressource introuvable.';
      case 429:
        return 'Trop de requêtes. Patientez un instant.';
      case 500:
      case 502:
      case 503:
      case 504:
        return 'Le serveur rencontre un problème. Réessayez plus tard.';
      default:
        return 'Erreur inattendue ($code).';
    }
  }

  void fermer() => _client.close();
}
```

Une seule fonction, `obtenirJson`, concentre tout : URL, timeout, statut, décodage, traduction des erreurs. Toutes les autres méthodes du service l'appelleront. **Écrivez cette fonction une fois dans votre projet, et ne la réécrivez plus jamais.**

---

## 53.21 — Les trois états d'un écran de données

Voici le concept central de la seconde moitié du chapitre. Prenez le temps de l'intégrer, il structure tout ce qui suit.

Un écran qui affiche des données réseau n'a pas un état, ni deux, mais **trois** — et souvent quatre.

```text
            ┌──────────────────────────────┐
            │  L'utilisateur ouvre l'écran │
            └───────────────┬──────────────┘
                            v
            ┌──────────────────────────────┐
            │   ÉTAT 1 : CHARGEMENT        │
            │   ────────────────────       │
            │   On a lancé la requête,     │
            │   on n'a pas encore reçu.    │
            │                              │
            │   Affichage :                │
            │   CircularProgressIndicator  │
            └───────────┬─────────┬────────┘
                        │         │
          la requête    │         │   la requête
          échoue        │         │   réussit
                        v         v
      ┌───────────────────┐   ┌──────────────────────┐
      │  ÉTAT 2 : ERREUR  │   │  ÉTAT 3 : SUCCÈS     │
      │  ───────────────  │   │  ───────────────     │
      │  Icône + message  │   │  Les données !       │
      │  + bouton         │   │                      │
      │    « Réessayer »  │   │  Mais attention...   │
      └─────────┬─────────┘   └──────────┬───────────┘
                │                        │
                │                        v
                │             ┌──────────────────────┐
                │             │  ÉTAT 3bis : VIDE    │
                │             │  ─────────────────   │
                │             │  Succès, mais la     │
                │             │  liste est vide.     │
                │             │  « Aucun résultat »  │
                │             └──────────────────────┘
                │
                └──────> retour à l'ÉTAT 1
```

### Pourquoi l'état 3bis mérite son nom

Un débutant traite « liste vide » comme « succès » et affiche une page blanche. L'utilisateur, lui, croit que l'application est cassée.

Comparez :

```text
MAUVAIS                        BON
┌───────────────┐              ┌──────────────────────────┐
│               │              │                          │
│               │              │        [icône]           │
│   (rien)      │              │                          │
│               │              │   Aucun article trouvé   │
│               │              │                          │
│               │              │   Essayez un autre mot.  │
└───────────────┘              └──────────────────────────┘
```

Le second coûte dix lignes de code et évite des courriels de support.

### Les quatre états, et rien d'autre

| État | Condition | Ce qu'on affiche |
| --- | --- | --- |
| Chargement | Requête en cours, aucune donnée | Indicateur circulaire, ou squelette |
| Erreur | Exception attrapée | Icône, message clair, bouton « Réessayer » |
| Vide | Succès, mais `liste.isEmpty` | Illustration, explication, action suggérée |
| Succès | Succès, `liste.isNotEmpty` | Les données |

> **La faute la plus courante de tout ce chapitre :** ne coder que l'état « succès », et découvrir les trois autres en production, chez les utilisateurs.

Il y a même un cinquième cas, plus subtil, qui n'apparaît qu'au rechargement : **anciennes données + rechargement en cours**. On veut alors garder la liste visible et n'afficher qu'un fin bandeau de progression, pas un écran blanc. `FutureBuilder` gère ce cas via `snapshot.hasData` combiné à `connectionState` (section 53.23).

---

## 53.22 — `FutureBuilder`

Coder les quatre états à la main avec `setState` fonctionne, mais c'est répétitif. Flutter fournit un widget qui fait exactement ce travail : **`FutureBuilder`**.

L'idée : vous lui donnez un `Future`, il vous rappelle **à chaque changement d'état** de ce `Future`, et vous décidez quoi dessiner.

Voici la signature exacte :

```dart
FutureBuilder({
  Key? key,
  required Future<T>? future,
  T? initialData,
  required AsyncWidgetBuilder<T> builder,
})
```

| Paramètre | Type | Rôle |
| --- | --- | --- |
| `future` | `Future<T>?` | Le calcul asynchrone à surveiller. Peut être `null`. |
| `builder` | `AsyncWidgetBuilder<T>` | Appelé à chaque changement : `(context, snapshot) => Widget` |
| `initialData` | `T?` | Une donnée affichée avant que le `Future` ne se termine |

Et le type du `builder` :

```dart
typedef AsyncWidgetBuilder<T> = Widget Function(
  BuildContext context,
  AsyncSnapshot<T> snapshot,
);
```

`FutureBuilder` **appelle `setState` pour vous**. Vous n'avez plus de variable `_enCours`, plus de variable `_erreur`, plus de variable `_donnees`. Tout est dans le `snapshot`.

Voici l'application complète. C'est le modèle de référence : relisez-la plusieurs fois.

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

// ─────────────────────────── MODÈLE ───────────────────────────

class Article {
  final int id;
  final int idUtilisateur;
  final String titre;
  final String corps;

  const Article({
    required this.id,
    required this.idUtilisateur,
    required this.titre,
    required this.corps,
  });

  factory Article.fromJson(Map<String, dynamic> json) => Article(
        id: (json['id'] as num?)?.toInt() ?? 0,
        idUtilisateur: (json['userId'] as num?)?.toInt() ?? 0,
        titre: json['title'] as String? ?? 'Sans titre',
        corps: json['body'] as String? ?? '',
      );
}

// ────────────────────────── EXCEPTION ─────────────────────────

class ErreurApi implements Exception {
  final String message;
  final int? codeStatut;
  const ErreurApi(this.message, {this.codeStatut});
  @override
  String toString() => message;
}

// ─────────────────────────── SERVICE ──────────────────────────

class ServiceArticles {
  final http.Client _client;
  ServiceArticles({http.Client? client}) : _client = client ?? http.Client();

  Future<List<Article>> lister() async {
    final url = Uri.https('jsonplaceholder.typicode.com', '/posts');
    try {
      final reponse = await _client
          .get(url, headers: const {'Accept': 'application/json'})
          .timeout(const Duration(seconds: 10));

      if (reponse.statusCode < 200 || reponse.statusCode >= 300) {
        throw ErreurApi('Erreur ${reponse.statusCode}.',
            codeStatut: reponse.statusCode);
      }

      final brut = jsonDecode(utf8.decode(reponse.bodyBytes));
      if (brut is! List) {
        throw const ErreurApi('Format inattendu.');
      }
      return brut
          .map((e) => Article.fromJson(e as Map<String, dynamic>))
          .toList();
    } on TimeoutException {
      throw const ErreurApi('Le serveur met trop de temps à répondre.');
    } on SocketException {
      throw const ErreurApi('Aucune connexion Internet.');
    } on http.ClientException {
      throw const ErreurApi('La connexion a été interrompue.');
    } on FormatException {
      throw const ErreurApi('Réponse illisible du serveur.');
    }
  }

  void fermer() => _client.close();
}

// ────────────────────────── APPLICATION ───────────────────────

void main() {
  runApp(const ApplicationFutureBuilder());
}

class ApplicationFutureBuilder extends StatelessWidget {
  const ApplicationFutureBuilder({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'FutureBuilder',
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const EcranArticles(),
    );
  }
}

class EcranArticles extends StatefulWidget {
  const EcranArticles({super.key});

  @override
  State<EcranArticles> createState() => _EcranArticlesState();
}

class _EcranArticlesState extends State<EcranArticles> {
  final ServiceArticles _service = ServiceArticles();

  // Le Future est un CHAMP du State, pas une expression dans build().
  late Future<List<Article>> _futur;

  @override
  void initState() {
    super.initState();
    _futur = _service.lister();
  }

  @override
  void dispose() {
    _service.fermer();
    super.dispose();
  }

  void _recharger() {
    setState(() {
      _futur = _service.lister();
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('53.22 — FutureBuilder'),
        actions: [
          IconButton(
            onPressed: _recharger,
            icon: const Icon(Icons.refresh),
            tooltip: 'Recharger',
          ),
        ],
      ),
      body: FutureBuilder<List<Article>>(
        future: _futur,
        builder: (context, snapshot) {
          // ÉTAT 1 — chargement
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }

          // ÉTAT 2 — erreur
          if (snapshot.hasError) {
            return _VueErreur(
              message: '${snapshot.error}',
              onReessayer: _recharger,
            );
          }

          final articles = snapshot.data ?? const <Article>[];

          // ÉTAT 3bis — vide
          if (articles.isEmpty) {
            return const _VueVide();
          }

          // ÉTAT 3 — succès
          return ListView.separated(
            itemCount: articles.length,
            separatorBuilder: (_, __) => const Divider(height: 1),
            itemBuilder: (context, i) {
              final a = articles[i];
              return ListTile(
                leading: CircleAvatar(child: Text('${a.id}')),
                title: Text(a.titre, maxLines: 1, overflow: TextOverflow.ellipsis),
                subtitle: Text(a.corps, maxLines: 2, overflow: TextOverflow.ellipsis),
              );
            },
          );
        },
      ),
    );
  }
}

class _VueErreur extends StatelessWidget {
  final String message;
  final VoidCallback onReessayer;

  const _VueErreur({required this.message, required this.onReessayer});

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Padding(
        padding: const EdgeInsets.all(32),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Icon(Icons.cloud_off,
                size: 64, color: Theme.of(context).colorScheme.error),
            const SizedBox(height: 16),
            Text(
              message,
              textAlign: TextAlign.center,
              style: Theme.of(context).textTheme.bodyLarge,
            ),
            const SizedBox(height: 24),
            FilledButton.icon(
              onPressed: onReessayer,
              icon: const Icon(Icons.refresh),
              label: const Text('Réessayer'),
            ),
          ],
        ),
      ),
    );
  }
}

class _VueVide extends StatelessWidget {
  const _VueVide();

  @override
  Widget build(BuildContext context) {
    return const Center(
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          Icon(Icons.inbox_outlined, size: 64),
          SizedBox(height: 16),
          Text('Aucun article pour le moment.'),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
Pendant ~500 ms : un indicateur circulaire au centre.
Puis : une liste de 100 articles.
Sans réseau (mode avion) : l'icône « nuage barré »,
  le message « Aucune connexion Internet. »,
  et un bouton « Réessayer ».
```

Cette application contient à peu près tout ce que ce chapitre a enseigné jusqu'ici : modèle, service, timeout, traduction d'erreurs, quatre états, `initState`, `dispose`. C'est la structure que vous reproduirez sur chaque écran de données.

---

## 53.23 — `AsyncSnapshot` : `connectionState`, `hasData`, `hasError`

Le `snapshot` est le seul argument utile du `builder`. C'est une **photographie instantanée** de l'état du `Future`.

| Propriété | Type | Signification |
| --- | --- | --- |
| `connectionState` | `ConnectionState` | Où en est la connexion au `Future` |
| `data` | `T?` | La donnée, si elle est arrivée |
| `error` | `Object?` | L'erreur, si elle s'est produite |
| `stackTrace` | `StackTrace?` | La pile d'appels de l'erreur |
| `hasData` | `bool` | Vrai si `data != null` |
| `hasError` | `bool` | Vrai si `error != null` |
| `requireData` | `T` | `data`, mais lève une exception si absente |

### Les quatre valeurs de `ConnectionState`

| Valeur | Signification | Avec un `Future` |
| --- | --- | --- |
| `none` | Aucun `Future` n'est branché | `future` vaut `null` |
| `waiting` | Branché, en attente du premier résultat | La requête est en cours |
| `active` | Des données arrivent, ce n'est pas fini | **Jamais** avec un `Future` (seulement `Stream`) |
| `done` | Terminé, en succès ou en erreur | La requête est finie |

Retenez : avec un `Future`, seuls **`none`**, **`waiting`** et **`done`** apparaissent.

### La chronologie exacte

```text
   t=0      Le widget est construit pour la première fois.
            connectionState : waiting
            hasData         : false
            hasError        : false
            data            : null

   t=600ms  La requête réussit.
            connectionState : done
            hasData         : true
            hasError        : false
            data            : [Article, Article, ...]

   OU (autre exécution)

   t=600ms  La requête échoue.
            connectionState : done
            hasData         : false
            hasError        : true
            error           : ErreurApi("Aucune connexion Internet.")
```

Le programme suivant affiche le `snapshot` en direct, à chaque appel du `builder`.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationSnapshot());
}

class ApplicationSnapshot extends StatelessWidget {
  const ApplicationSnapshot({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.orange, useMaterial3: true),
      home: const PageSnapshot(),
    );
  }
}

class PageSnapshot extends StatefulWidget {
  const PageSnapshot({super.key});

  @override
  State<PageSnapshot> createState() => _PageSnapshotState();
}

class _PageSnapshotState extends State<PageSnapshot> {
  Future<String>? _futur;
  final List<String> _traces = [];

  Future<String> _reussir() async {
    await Future<void>.delayed(const Duration(seconds: 2));
    return 'Trois potions et une épée.';
  }

  Future<String> _echouer() async {
    await Future<void>.delayed(const Duration(seconds: 2));
    throw Exception('Le coffre est verrouillé.');
  }

  void _lancer(Future<String> Function() fabrique) {
    setState(() {
      _traces.clear();
      _futur = fabrique();
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('53.23 — AsyncSnapshot')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            Row(
              children: [
                Expanded(
                  child: FilledButton(
                    onPressed: () => _lancer(_reussir),
                    child: const Text('Réussir'),
                  ),
                ),
                const SizedBox(width: 12),
                Expanded(
                  child: FilledButton.tonal(
                    onPressed: () => _lancer(_echouer),
                    child: const Text('Échouer'),
                  ),
                ),
              ],
            ),
            const SizedBox(height: 12),
            OutlinedButton(
              onPressed: () => setState(() {
                _traces.clear();
                _futur = null;
              }),
              child: const Text('future = null'),
            ),
            const SizedBox(height: 16),
            Expanded(
              child: FutureBuilder<String>(
                future: _futur,
                builder: (context, snapshot) {
                  // On enregistre chaque appel du builder.
                  final trace = 'appel #${_traces.length + 1}  '
                      'état=${snapshot.connectionState.name}  '
                      'hasData=${snapshot.hasData}  '
                      'hasError=${snapshot.hasError}';
                  _traces.add(trace);

                  return Column(
                    crossAxisAlignment: CrossAxisAlignment.stretch,
                    children: [
                      Card(
                        child: Padding(
                          padding: const EdgeInsets.all(12),
                          child: Column(
                            crossAxisAlignment: CrossAxisAlignment.start,
                            children: [
                              Text('connectionState : '
                                  '${snapshot.connectionState.name}'),
                              Text('hasData         : ${snapshot.hasData}'),
                              Text('hasError        : ${snapshot.hasError}'),
                              Text('data            : ${snapshot.data}'),
                              Text('error           : ${snapshot.error}'),
                            ],
                          ),
                        ),
                      ),
                      const SizedBox(height: 8),
                      const Text('Journal des appels du builder :'),
                      Expanded(
                        child: ListView(
                          children: _traces
                              .map((t) => Text(
                                    t,
                                    style: const TextStyle(
                                        fontFamily: 'monospace', fontSize: 11),
                                  ))
                              .toList(),
                        ),
                      ),
                    ],
                  );
                },
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat après appui sur « Réussir » :**

```text
connectionState : done
hasData         : true
hasError        : false
data            : Trois potions et une épée.
error           : null

Journal des appels du builder :
appel #1  état=waiting  hasData=false  hasError=false
appel #2  état=done     hasData=true   hasError=false
```

**Résultat après appui sur « Échouer » :**

```text
connectionState : done
hasData         : false
hasError        : true
data            : null
error           : Exception: Le coffre est verrouillé.

appel #1  état=waiting  hasData=false  hasError=false
appel #2  état=done     hasData=false  hasError=true
```

Deux appels du `builder`, pas plus. C'est exactement le comportement attendu.

### L'ordre des tests dans le `builder`

L'ordre n'est pas arbitraire. Il y a un ordre correct et un seul.

```dart
builder: (context, snapshot) {
  // 1. Le chargement d'abord.
  if (snapshot.connectionState == ConnectionState.waiting) {
    return const Center(child: CircularProgressIndicator());
  }

  // 2. L'erreur ensuite. TOUJOURS avant les données :
  //    en cas d'erreur, data vaut null.
  if (snapshot.hasError) {
    return VueErreur(message: '${snapshot.error}');
  }

  // 3. L'absence de données (future == null, ou T? nul).
  if (!snapshot.hasData) {
    return const Center(child: Text('Aucune donnée.'));
  }

  // 4. Le succès. Ici, snapshot.data est garanti non nul.
  final donnees = snapshot.data!;
  return VueSucces(donnees: donnees);
}
```

> **L'erreur avant les données.** Si vous testez `hasData` en premier et affichez sinon un indicateur, une requête en échec laisse tourner l'indicateur pour toujours. C'est un bug fréquent et très déroutant.

### Le raffinement : garder les anciennes données pendant le rechargement

Le test `connectionState == waiting` en premier a un défaut : lors d'un rechargement, il efface la liste déjà affichée et remet un indicateur plein écran. Ça clignote.

La version soignée :

```dart
builder: (context, snapshot) {
  final enChargement = snapshot.connectionState == ConnectionState.waiting;

  // On a déjà des données : on les garde, avec un fin bandeau en haut.
  if (snapshot.hasData) {
    return Column(
      children: [
        if (enChargement) const LinearProgressIndicator(minHeight: 2),
        Expanded(child: VueListe(donnees: snapshot.data!)),
      ],
    );
  }

  if (enChargement) {
    return const Center(child: CircularProgressIndicator());
  }
  if (snapshot.hasError) {
    return VueErreur(message: '${snapshot.error}');
  }
  return const Center(child: Text('Aucune donnée.'));
}
```

Attention : ce raffinement ne fonctionne que si le `FutureBuilder` conserve son `State` entre les deux `Future`, ce qui est le cas quand on remplace simplement le champ `_futur` par `setState`.

---

## 53.24 — Le piège : appeler le `Future` dans `build()`

Voici le bug le plus fréquent de tout Flutter. Il ne provoque **aucun message d'erreur**. L'application semble fonctionner. Et pourtant elle appelle l'API en boucle.

Le code fautif :

```dart
// NE JAMAIS ÉCRIRE CECI
@override
Widget build(BuildContext context) {
  return FutureBuilder<List<Article>>(
    future: _service.lister(),        // <-- APPEL DANS build()
    builder: (context, snapshot) { /* ... */ },
  );
}
```

### Pourquoi c'est grave

Rappelez-vous le chapitre 45 : **`build()` peut être appelé à chaque image, soit 60 fois par seconde.** Il est appelé quand :

- `setState()` est appelé ;
- le parent se reconstruit ;
- le thème change ;
- l'orientation change ;
- le clavier s'ouvre ou se ferme ;
- une animation quelconque tourne dans un ancêtre ;
- `MediaQuery` change (taille de fenêtre, mode sombre).

À chacun de ces événements, `_service.lister()` est rappelé, une **nouvelle** requête HTTP part, le `FutureBuilder` reçoit un **nouveau** `Future`, repasse en `waiting`, et l'indicateur de chargement réapparaît.

```text
        AVEC LE FUTURE DANS build()

  build() #1  --> lister() --> requête HTTP #1 --> waiting
  build() #2  --> lister() --> requête HTTP #2 --> waiting
  build() #3  --> lister() --> requête HTTP #3 --> waiting
  build() #4  --> lister() --> requête HTTP #4 --> waiting
                        ...
  L'écran clignote. Le serveur vous bannit (429).
  La batterie se vide. Le forfait data fond.


        AVEC LE FUTURE DANS initState()

  initState() --> lister() --> requête HTTP #1
  build() #1  --> réutilise le MÊME Future --> waiting
  build() #2  --> réutilise le MÊME Future --> waiting
  build() #3  --> réutilise le MÊME Future --> done, données
  build() #4  --> réutilise le MÊME Future --> done, données
```

La documentation officielle de `FutureBuilder` est explicite : le `Future` « ne doit pas être créé pendant l'appel à `State.build` ou `StatelessWidget.build` », car « chaque fois que le parent du `FutureBuilder` est reconstruit, la tâche asynchrone est redémarrée ».

### La démonstration

Cette application compte les requêtes. Faites tourner l'appareil, ouvrez le clavier, changez le curseur : le compteur de gauche explose, celui de droite non.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationPiege());
}

class ApplicationPiege extends StatelessWidget {
  const ApplicationPiege({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.deepPurple, useMaterial3: true),
      home: const PagePiege(),
    );
  }
}

/// Compteurs globaux d'appels simulés à l'API.
int compteurMauvais = 0;
int compteurBon = 0;

Future<String> appelMauvais() async {
  compteurMauvais++;
  await Future<void>.delayed(const Duration(milliseconds: 300));
  return 'requêtes : $compteurMauvais';
}

Future<String> appelBon() async {
  compteurBon++;
  await Future<void>.delayed(const Duration(milliseconds: 300));
  return 'requêtes : $compteurBon';
}

class PagePiege extends StatefulWidget {
  const PagePiege({super.key});

  @override
  State<PagePiege> createState() => _PagePiegeState();
}

class _PagePiegeState extends State<PagePiege> {
  double _curseur = 0;

  // Le BON : créé une seule fois.
  late final Future<String> _futurBon = appelBon();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('53.24 — Le piège du build()')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            const Text(
              'Faites glisser le curseur. Chaque mouvement '
              'déclenche un build().',
              textAlign: TextAlign.center,
            ),
            Slider(
              value: _curseur,
              onChanged: (v) => setState(() => _curseur = v),
            ),
            const SizedBox(height: 24),
            Card(
              color: Theme.of(context).colorScheme.errorContainer,
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Text('MAUVAIS : future créé dans build()'),
                    const SizedBox(height: 8),
                    FutureBuilder<String>(
                      // Le Future est recréé à CHAQUE build.
                      future: appelMauvais(),
                      builder: (context, snap) => Text(
                        snap.data ?? 'chargement...',
                        style: const TextStyle(
                            fontSize: 20, fontWeight: FontWeight.bold),
                      ),
                    ),
                  ],
                ),
              ),
            ),
            const SizedBox(height: 16),
            Card(
              color: Theme.of(context).colorScheme.primaryContainer,
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Text('BON : future créé une seule fois'),
                    const SizedBox(height: 8),
                    FutureBuilder<String>(
                      future: _futurBon,
                      builder: (context, snap) => Text(
                        snap.data ?? 'chargement...',
                        style: const TextStyle(
                            fontSize: 20, fontWeight: FontWeight.bold),
                      ),
                    ),
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

**Résultat après avoir bougé le curseur quelques secondes :**

```text
MAUVAIS : future créé dans build()
requêtes : 187

BON : future créé une seule fois
requêtes : 1
```

187 requêtes contre 1. Sur une vraie API, c'est un `429 Too Many Requests` garanti, et une facture serveur multipliée par cent.

### Comment le repérer dans votre code

Cherchez `future:` dans tout le projet. Si la valeur qui suit contient une **paire de parenthèses d'appel**, c'est un bug.

```text
future: _futur                      OK  (une variable)
future: widget.futur                OK  (une variable)
future: _service.lister()           BUG (un appel)
future: chargerDonnees()            BUG (un appel)
future: Future.delayed(...)         BUG (une construction)
```

---

## 53.25 — Créer le `Future` dans `initState()`

La solution canonique : le `Future` est un **champ** de la classe `State`, créé une seule fois.

```dart
class _EcranArticlesState extends State<EcranArticles> {
  final ServiceArticles _service = ServiceArticles();
  late Future<List<Article>> _futur;

  @override
  void initState() {
    super.initState();
    _futur = _service.lister();     // une seule fois, à la création du State
  }

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<List<Article>>(
      future: _futur,               // on lit le champ, on n'appelle rien
      builder: (context, snapshot) { /* ... */ },
    );
  }
}
```

Pourquoi ça marche : `initState()` est appelé **une seule fois** dans la vie d'un `State` (chapitre 45), juste après sa création et avant le premier `build()`.

### Les trois écritures possibles

| Écriture | Quand l'utiliser |
| --- | --- |
| `late Future<T> _futur;` + affectation dans `initState()` | Le cas général. Permet de réaffecter plus tard (rechargement). |
| `late final Future<T> _futur = charger();` | Quand le `Future` ne sera jamais rechargé. Concis, initialisation paresseuse. |
| `Future<T>? _futur;` | Quand le chargement ne démarre pas à l'ouverture (déclenché par un bouton). |

Attention au deuxième : `late final` interdit la réaffectation. Vous ne pourrez pas implémenter le bouton « réessayer ».

### Quand les paramètres viennent du widget

Si l'URL dépend d'une propriété du widget (`widget.idUtilisateur`, par exemple), il faut aussi gérer le cas où le parent change ce paramètre **sans** détruire le `State`. C'est le rôle de `didUpdateWidget`.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationParametre());
}

class ApplicationParametre extends StatelessWidget {
  const ApplicationParametre({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.green, useMaterial3: true),
      home: const PageParametre(),
    );
  }
}

/// Simulation d'un service.
Future<String> chargerProfil(int id) async {
  await Future<void>.delayed(const Duration(milliseconds: 700));
  const noms = {1: 'Alia', 2: 'Baltus', 3: 'Cendre'};
  return noms[id] ?? 'Inconnu';
}

class PageParametre extends StatefulWidget {
  const PageParametre({super.key});

  @override
  State<PageParametre> createState() => _PageParametreState();
}

class _PageParametreState extends State<PageParametre> {
  int _id = 1;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('53.25 — didUpdateWidget')),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16),
            child: SegmentedButton<int>(
              segments: const [
                ButtonSegment(value: 1, label: Text('Joueur 1')),
                ButtonSegment(value: 2, label: Text('Joueur 2')),
                ButtonSegment(value: 3, label: Text('Joueur 3')),
              ],
              selected: {_id},
              onSelectionChanged: (s) => setState(() => _id = s.first),
            ),
          ),
          // Le MÊME widget VueProfil est réutilisé, seul idJoueur change.
          Expanded(child: VueProfil(idJoueur: _id)),
        ],
      ),
    );
  }
}

class VueProfil extends StatefulWidget {
  final int idJoueur;
  const VueProfil({super.key, required this.idJoueur});

  @override
  State<VueProfil> createState() => _VueProfilState();
}

class _VueProfilState extends State<VueProfil> {
  late Future<String> _futur;

  @override
  void initState() {
    super.initState();
    _futur = chargerProfil(widget.idJoueur);
  }

  @override
  void didUpdateWidget(covariant VueProfil ancien) {
    super.didUpdateWidget(ancien);
    // Le parent a changé le paramètre : on relance la requête.
    // SANS cette méthode, l'écran resterait figé sur le joueur 1.
    if (ancien.idJoueur != widget.idJoueur) {
      setState(() {
        _futur = chargerProfil(widget.idJoueur);
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<String>(
      future: _futur,
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const Center(child: CircularProgressIndicator());
        }
        if (snapshot.hasError) {
          return Center(child: Text('Erreur : ${snapshot.error}'));
        }
        return Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              CircleAvatar(
                radius: 40,
                child: Text(
                  snapshot.data!.substring(0, 1),
                  style: const TextStyle(fontSize: 32),
                ),
              ),
              const SizedBox(height: 16),
              Text(
                snapshot.data!,
                style: Theme.of(context).textTheme.headlineSmall,
              ),
              Text('identifiant ${widget.idJoueur}'),
            ],
          ),
        );
      },
    );
  }
}
```

**Résultat :**

```text
Sélection « Joueur 1 » : indicateur, puis « A / Alia / identifiant 1 »
Sélection « Joueur 2 » : indicateur, puis « B / Baltus / identifiant 2 »
Sélection « Joueur 3 » : indicateur, puis « C / Cendre / identifiant 3 »
```

Sans `didUpdateWidget`, l'écran afficherait toujours « Alia » avec le libellé « identifiant 3 » : un mélange incohérent, et un bug très difficile à comprendre.

### Le tableau de décision

| Où créer le `Future` ? | Verdict |
| --- | --- |
| Dans `build()` | **Jamais.** Relancé à chaque image. |
| Dans `initState()` | Le cas normal. |
| Dans `didUpdateWidget()` | Quand un paramètre du widget change. |
| Dans `didChangeDependencies()` | Quand la donnée dépend d'un `InheritedWidget` (chapitre 52). |
| Dans un gestionnaire d'événement (`onPressed`), avec `setState` | Rechargement manuel, réessai. |

---

## 53.26 — Le bouton « réessayer »

Un écran d'erreur sans moyen de réessayer est une impasse. L'utilisateur doit fermer l'application et la rouvrir. C'est inacceptable.

Le mécanisme est d'une simplicité trompeuse :

```dart
void _reessayer() {
  setState(() {
    _futur = _service.lister();   // un NOUVEAU Future
  });
}
```

Trois choses se produisent, dans cet ordre :

1. `_service.lister()` lance une nouvelle requête et renvoie un nouveau `Future` ;
2. `setState` marque le widget comme devant se reconstruire ;
3. `build()` passe ce **nouveau** `Future` au `FutureBuilder`, qui repasse en `waiting`.

`FutureBuilder` détecte le changement de `Future` par **identité d'objet**. C'est pour cela qu'il faut bien créer un nouvel objet, et non réutiliser l'ancien.

> **Attention :** `setState(() { _futur = _service.lister(); })` et `setState(() {})` ne font pas la même chose. Le second reconstruit avec le **même** `Future`, déjà terminé : rien ne change. Il faut réaffecter.

Voici un écran d'erreur complet et réutilisable, avec adaptation du message au type d'échec.

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';
import 'dart:math';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

class ErreurApi implements Exception {
  final String message;
  final int? codeStatut;
  const ErreurApi(this.message, {this.codeStatut});

  bool get estReseau => codeStatut == null;
  bool get estAuthentification => codeStatut == 401 || codeStatut == 403;
  bool get estIntrouvable => codeStatut == 404;
  bool get estServeur => codeStatut != null && codeStatut! >= 500;

  /// Réessayer a-t-il une chance d'aboutir ?
  bool get reessayable => estReseau || estServeur || codeStatut == 429;

  IconData get icone {
    if (estReseau) return Icons.wifi_off;
    if (estAuthentification) return Icons.lock_outline;
    if (estIntrouvable) return Icons.search_off;
    if (estServeur) return Icons.dns_outlined;
    return Icons.error_outline;
  }

  @override
  String toString() => message;
}

void main() => runApp(const ApplicationReessai());

class ApplicationReessai extends StatelessWidget {
  const ApplicationReessai({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.blueGrey, useMaterial3: true),
      home: const PageReessai(),
    );
  }
}

class PageReessai extends StatefulWidget {
  const PageReessai({super.key});

  @override
  State<PageReessai> createState() => _PageReessaiState();
}

class _PageReessaiState extends State<PageReessai> {
  final _alea = Random();
  late Future<List<String>> _futur;
  int _tentatives = 0;

  @override
  void initState() {
    super.initState();
    _futur = _charger();
  }

  /// Simulation : deux fois sur trois, l'appel échoue.
  Future<List<String>> _charger() async {
    _tentatives++;
    await Future<void>.delayed(const Duration(milliseconds: 900));
    final tirage = _alea.nextInt(3);
    if (tirage == 0) {
      return ['Épée de fer', 'Potion de soin', 'Bouclier rond'];
    }
    if (tirage == 1) {
      throw const ErreurApi('Aucune connexion Internet.');
    }
    throw const ErreurApi(
      'Le serveur rencontre un problème. Réessayez plus tard.',
      codeStatut: 503,
    );
  }

  void _reessayer() {
    setState(() {
      _futur = _charger();
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('53.26 — Réessayer (essai $_tentatives)')),
      body: FutureBuilder<List<String>>(
        future: _futur,
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }
          if (snapshot.hasError) {
            final erreur = snapshot.error;
            return VueErreur(
              erreur: erreur is ErreurApi
                  ? erreur
                  : ErreurApi('Erreur inattendue : $erreur'),
              onReessayer: _reessayer,
            );
          }
          final objets = snapshot.data!;
          return ListView(
            children: [
              for (final o in objets)
                ListTile(leading: const Icon(Icons.inventory_2), title: Text(o)),
            ],
          );
        },
      ),
    );
  }
}

class VueErreur extends StatelessWidget {
  final ErreurApi erreur;
  final VoidCallback onReessayer;

  const VueErreur({super.key, required this.erreur, required this.onReessayer});

  @override
  Widget build(BuildContext context) {
    final couleurs = Theme.of(context).colorScheme;
    return Center(
      child: Padding(
        padding: const EdgeInsets.all(32),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Icon(erreur.icone, size: 72, color: couleurs.error),
            const SizedBox(height: 20),
            Text(
              erreur.message,
              textAlign: TextAlign.center,
              style: Theme.of(context).textTheme.titleMedium,
            ),
            if (erreur.codeStatut != null) ...[
              const SizedBox(height: 8),
              Text(
                'Code ${erreur.codeStatut}',
                style: Theme.of(context).textTheme.bodySmall,
              ),
            ],
            const SizedBox(height: 28),
            if (erreur.reessayable)
              FilledButton.icon(
                onPressed: onReessayer,
                icon: const Icon(Icons.refresh),
                label: const Text('Réessayer'),
              )
            else
              OutlinedButton.icon(
                onPressed: onReessayer,
                icon: const Icon(Icons.arrow_back),
                label: const Text('Recharger la page'),
              ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat (une exécution possible) :**

```text
essai 1 : indicateur, puis icône wifi barré + « Aucune connexion Internet. »
          + bouton « Réessayer »
essai 2 : indicateur, puis icône serveur + « Le serveur rencontre un problème... »
          + « Code 503 » + bouton « Réessayer »
essai 3 : indicateur, puis la liste : Épée de fer / Potion de soin / Bouclier rond
```

Remarquez le détail soigné : quand l'erreur n'est **pas** réessayable (un `404`, un `401`), on n'affiche pas un bouton « Réessayer » qui échouerait à coup sûr. On propose autre chose.

---

## 53.27 — `RefreshIndicator` (rappel chapitre 48)

Le geste « tirer vers le bas pour rafraîchir » est un standard universel sur mobile. Flutter le fournit clé en main avec `RefreshIndicator`, déjà croisé au chapitre 48.

```dart
RefreshIndicator({
  Key? key,
  required Widget child,
  required Future<void> Function() onRefresh,
  double displacement = 40.0,
  Color? color,
  Color? backgroundColor,
})
```

Trois exigences, et elles se cumulent :

1. `child` doit être **défilable** : `ListView`, `GridView`, `CustomScrollView`, ou un `SingleChildScrollView`.
2. `onRefresh` doit renvoyer un `Future<void>`. L'animation tourne **tant que ce `Future` n'est pas terminé**.
3. Le contenu doit pouvoir défiler **au-delà du haut**. Si la liste est plus courte que l'écran, il faut `physics: const AlwaysScrollableScrollPhysics()`.

Le point 2 est celui qu'on rate. Comparez :

```dart
// MAUVAIS : le Future se termine instantanément,
// l'animation disparaît avant même d'avoir commencé.
onRefresh: () async {
  setState(() => _futur = _service.lister());
}

// BON : on attend réellement la fin de la requête.
onRefresh: () async {
  final futur = _service.lister();
  setState(() => _futur = futur);
  await futur;
}
```

Dans la seconde version, on garde une référence locale au `Future` **avant** le `setState`, et on l'`await` ensuite. L'animation tourne pendant toute la durée de la requête.

Attention aussi : si `await futur` lève une exception, `onRefresh` propage l'erreur. On l'absorbe, car le `FutureBuilder` l'affichera déjà.

```dart
onRefresh: () async {
  final futur = _service.lister();
  setState(() => _futur = futur);
  try {
    await futur;
  } catch (_) {
    // Le FutureBuilder affiche déjà l'erreur ; on évite juste
    // une exception non attrapée dans le gestionnaire de rafraîchissement.
  }
}
```

Voici l'application complète.

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

class Article {
  final int id;
  final String titre;
  const Article({required this.id, required this.titre});

  factory Article.fromJson(Map<String, dynamic> j) => Article(
        id: (j['id'] as num?)?.toInt() ?? 0,
        titre: j['title'] as String? ?? '',
      );
}

class ErreurApi implements Exception {
  final String message;
  const ErreurApi(this.message);
  @override
  String toString() => message;
}

class ServiceArticles {
  final http.Client _client = http.Client();

  Future<List<Article>> lister() async {
    try {
      final r = await _client
          .get(Uri.https('jsonplaceholder.typicode.com', '/posts',
              {'_limit': '20'}))
          .timeout(const Duration(seconds: 10));
      if (r.statusCode < 200 || r.statusCode >= 300) {
        throw ErreurApi('Erreur ${r.statusCode}.');
      }
      final brut = jsonDecode(utf8.decode(r.bodyBytes));
      if (brut is! List) throw const ErreurApi('Format inattendu.');
      return brut.map((e) => Article.fromJson(e as Map<String, dynamic>)).toList();
    } on TimeoutException {
      throw const ErreurApi('Délai dépassé.');
    } on SocketException {
      throw const ErreurApi('Aucune connexion Internet.');
    } on http.ClientException {
      throw const ErreurApi('Connexion interrompue.');
    } on FormatException {
      throw const ErreurApi('Réponse illisible.');
    }
  }

  void fermer() => _client.close();
}

void main() => runApp(const ApplicationRafraichir());

class ApplicationRafraichir extends StatelessWidget {
  const ApplicationRafraichir({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.cyan, useMaterial3: true),
      home: const PageRafraichir(),
    );
  }
}

class PageRafraichir extends StatefulWidget {
  const PageRafraichir({super.key});

  @override
  State<PageRafraichir> createState() => _PageRafraichirState();
}

class _PageRafraichirState extends State<PageRafraichir> {
  final ServiceArticles _service = ServiceArticles();
  late Future<List<Article>> _futur;
  DateTime? _dernierChargement;

  @override
  void initState() {
    super.initState();
    _futur = _lancer();
  }

  @override
  void dispose() {
    _service.fermer();
    super.dispose();
  }

  Future<List<Article>> _lancer() async {
    final resultat = await _service.lister();
    _dernierChargement = DateTime.now();
    return resultat;
  }

  Future<void> _rafraichir() async {
    final futur = _lancer();
    setState(() => _futur = futur);
    try {
      await futur;
    } catch (_) {
      // Le FutureBuilder affichera l'erreur.
    }
  }

  String get _horodatage {
    final d = _dernierChargement;
    if (d == null) return 'jamais';
    return '${d.hour.toString().padLeft(2, '0')}:'
        '${d.minute.toString().padLeft(2, '0')}:'
        '${d.second.toString().padLeft(2, '0')}';
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('53.27 — RefreshIndicator'),
        bottom: PreferredSize(
          preferredSize: const Size.fromHeight(24),
          child: Padding(
            padding: const EdgeInsets.only(bottom: 6),
            child: Text('Dernier chargement : $_horodatage'),
          ),
        ),
      ),
      body: RefreshIndicator(
        onRefresh: _rafraichir,
        child: FutureBuilder<List<Article>>(
          future: _futur,
          builder: (context, snapshot) {
            if (snapshot.connectionState == ConnectionState.waiting &&
                !snapshot.hasData) {
              return const Center(child: CircularProgressIndicator());
            }

            if (snapshot.hasError) {
              // IMPORTANT : même l'écran d'erreur doit être défilable,
              // sinon le geste de rafraîchissement ne fonctionne plus.
              return ListView(
                physics: const AlwaysScrollableScrollPhysics(),
                children: [
                  SizedBox(
                    height: MediaQuery.of(context).size.height * 0.7,
                    child: Center(
                      child: Column(
                        mainAxisSize: MainAxisSize.min,
                        children: [
                          const Icon(Icons.cloud_off, size: 64),
                          const SizedBox(height: 16),
                          Text('${snapshot.error}'),
                          const SizedBox(height: 8),
                          const Text('Tirez vers le bas pour réessayer.'),
                        ],
                      ),
                    ),
                  ),
                ],
              );
            }

            final articles = snapshot.data ?? const <Article>[];
            if (articles.isEmpty) {
              return ListView(
                physics: const AlwaysScrollableScrollPhysics(),
                children: const [
                  SizedBox(height: 200),
                  Center(child: Text('Aucun article.')),
                ],
              );
            }

            return ListView.separated(
              physics: const AlwaysScrollableScrollPhysics(),
              itemCount: articles.length,
              separatorBuilder: (_, __) => const Divider(height: 1),
              itemBuilder: (context, i) => ListTile(
                leading: CircleAvatar(child: Text('${articles[i].id}')),
                title: Text(
                  articles[i].titre,
                  maxLines: 2,
                  overflow: TextOverflow.ellipsis,
                ),
              ),
            );
          },
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
Barre : « Dernier chargement : 14:32:07 »
Liste de 20 articles.
Geste : tirer vers le bas -> l'indicateur tourne pendant la requête,
        puis l'horodatage se met à jour.
Sans réseau : « Aucune connexion Internet. » — et le geste fonctionne encore,
        parce que l'écran d'erreur est lui aussi un ListView.
```

> Le détail qui fait la différence : **l'écran d'erreur et l'écran vide doivent être défilables.** Sinon l'utilisateur se retrouve avec un message d'erreur et aucun moyen de recharger.

---

## 53.28 — `FutureBuilder` ou gestion d'état manuelle : comparaison

`FutureBuilder` n'est pas la seule façon d'afficher des données réseau. L'alternative est celle du chapitre 52 : gérer soi-même l'état.

Voici les deux versions du même écran, côte à côte.

### Version A — `FutureBuilder`

```dart
class _EcranAState extends State<EcranA> {
  late Future<List<Article>> _futur;

  @override
  void initState() {
    super.initState();
    _futur = ServiceArticles().lister();
  }

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<List<Article>>(
      future: _futur,
      builder: (context, snap) {
        if (snap.connectionState == ConnectionState.waiting) {
          return const Center(child: CircularProgressIndicator());
        }
        if (snap.hasError) return Center(child: Text('${snap.error}'));
        return VueListe(articles: snap.data!);
      },
    );
  }
}
```

### Version B — état manuel

```dart
class _EcranBState extends State<EcranB> {
  List<Article>? _articles;
  String? _erreur;
  bool _enCours = false;

  @override
  void initState() {
    super.initState();
    _charger();
  }

  Future<void> _charger() async {
    setState(() {
      _enCours = true;
      _erreur = null;
    });
    try {
      final resultat = await ServiceArticles().lister();
      if (!mounted) return;
      setState(() {
        _articles = resultat;
        _enCours = false;
      });
    } catch (e) {
      if (!mounted) return;
      setState(() {
        _erreur = '$e';
        _enCours = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    if (_enCours) return const Center(child: CircularProgressIndicator());
    if (_erreur != null) return Center(child: Text(_erreur!));
    return VueListe(articles: _articles ?? const []);
  }
}
```

### La comparaison honnête

| Critère | `FutureBuilder` | État manuel |
| --- | --- | --- |
| Lignes de code | Moins | Plus |
| Risque d'oublier `mounted` | Nul (géré par le widget) | Élevé |
| Risque de `Future` relancé | Élevé (le piège 53.24) | Faible |
| Modifier une donnée déjà chargée | Difficile | Naturel |
| Ajouter un élément à la liste sans tout recharger | Très difficile | Facile |
| Pagination (ajouter à la suite) | Inadapté | Adapté |
| Recherche avec débounce | Inadapté | Adapté |
| Partager l'état entre plusieurs écrans | Impossible | Possible via `provider` |
| Garder l'ancienne liste pendant le rechargement | Possible mais subtil | Naturel |
| Lisibilité pour un débutant | Très bonne | Moyenne |

### Le vrai critère de choix

Posez-vous une seule question :

> **Les données seront-elles modifiées après leur chargement ?**

- **Non** → `FutureBuilder`. Écran de détail, page « à propos », liste consultée seulement.
- **Oui** → état manuel, éventuellement avec `ChangeNotifier` et `provider` (chapitre 52). Panier, liste modifiable, pagination, recherche.

### La version « état manuel » propre : une classe scellée

Les trois variables `_enCours`, `_erreur`, `_articles` peuvent prendre des combinaisons absurdes : `_enCours == true` **et** `_erreur != null` en même temps. Dart 3 permet de rendre ces états impossibles à représenter, avec une hiérarchie **scellée** (`sealed`) et un `switch` exhaustif.

```dart
import 'package:flutter/material.dart';

/// Les états possibles, et RIEN d'autre.
sealed class EtatDonnees<T> {
  const EtatDonnees();
}

class EtatChargement<T> extends EtatDonnees<T> {
  const EtatChargement();
}

class EtatSucces<T> extends EtatDonnees<T> {
  final T donnees;
  const EtatSucces(this.donnees);
}

class EtatErreur<T> extends EtatDonnees<T> {
  final String message;
  const EtatErreur(this.message);
}

void main() => runApp(const ApplicationEtatScelle());

class ApplicationEtatScelle extends StatelessWidget {
  const ApplicationEtatScelle({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.purple, useMaterial3: true),
      home: const PageEtatScelle(),
    );
  }
}

class PageEtatScelle extends StatefulWidget {
  const PageEtatScelle({super.key});

  @override
  State<PageEtatScelle> createState() => _PageEtatScelleState();
}

class _PageEtatScelleState extends State<PageEtatScelle> {
  EtatDonnees<List<String>> _etat = const EtatChargement();
  bool _doitEchouer = false;

  @override
  void initState() {
    super.initState();
    _charger();
  }

  Future<void> _charger() async {
    setState(() => _etat = const EtatChargement());
    await Future<void>.delayed(const Duration(milliseconds: 800));
    if (!mounted) return;
    setState(() {
      _etat = _doitEchouer
          ? const EtatErreur('Le serveur ne répond pas.')
          : const EtatSucces(['Alia', 'Baltus', 'Cendre']);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('53.28 — État scellé')),
      body: Column(
        children: [
          SwitchListTile(
            title: const Text('Simuler une panne'),
            value: _doitEchouer,
            onChanged: (v) {
              setState(() => _doitEchouer = v);
              _charger();
            },
          ),
          const Divider(height: 1),
          Expanded(
            // Le switch est EXHAUSTIF : si vous ajoutez un état,
            // le compilateur vous signale ce switch.
            child: switch (_etat) {
              EtatChargement() =>
                const Center(child: CircularProgressIndicator()),
              EtatErreur(message: final m) => Center(
                  child: Column(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      const Icon(Icons.error_outline, size: 56),
                      const SizedBox(height: 12),
                      Text(m),
                      const SizedBox(height: 16),
                      FilledButton(
                        onPressed: _charger,
                        child: const Text('Réessayer'),
                      ),
                    ],
                  ),
                ),
              EtatSucces(donnees: final liste) => ListView(
                  children: [
                    for (final nom in liste)
                      ListTile(
                        leading: CircleAvatar(child: Text(nom[0])),
                        title: Text(nom),
                      ),
                  ],
                ),
            },
          ),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
Interrupteur éteint : indicateur, puis Alia / Baltus / Cendre.
Interrupteur allumé : indicateur, puis icône d'erreur
                      + « Le serveur ne répond pas. » + « Réessayer ».
```

Le gain est réel : **il devient impossible d'être en chargement et en erreur en même temps**, et le compilateur refuse un `switch` incomplet. C'est le patron employé dans la plupart des applications professionnelles.

---

## 53.29 — `http.post()` et l'envoi de JSON

Jusqu'ici, on n'a fait que lire. Passons à l'écriture.

```dart
Future<http.Response> post(
  Uri url, {
  Map<String, String>? headers,
  Object? body,
  Encoding? encoding,
})
```

Trois obligations, à respecter ensemble :

1. encoder le corps avec `jsonEncode()` — le paramètre `body` attend une `String` ;
2. déclarer `Content-Type: application/json; charset=utf-8` ;
3. accepter que le succès soit `201`, pas `200`.

```dart
final reponse = await http.post(
  Uri.https('jsonplaceholder.typicode.com', '/posts'),
  headers: const {
    'Content-Type': 'application/json; charset=utf-8',
    'Accept': 'application/json',
  },
  body: jsonEncode({
    'title': 'L\'épée de Cendre',
    'body': 'Forgée dans les cendres du volcan.',
    'userId': 1,
  }),
);
// reponse.statusCode == 201
// reponse.body        == {"title":"...","body":"...","userId":1,"id":101}
```

Le serveur renvoie l'objet créé, **avec l'identifiant qu'il a attribué**. On ne devine jamais cet identifiant côté client : on lit celui de la réponse.

Voici l'application complète : un formulaire qui envoie un article.

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

class Article {
  final int id;
  final String titre;
  final String corps;
  final int idUtilisateur;

  const Article({
    required this.id,
    required this.titre,
    required this.corps,
    required this.idUtilisateur,
  });

  factory Article.fromJson(Map<String, dynamic> j) => Article(
        id: (j['id'] as num?)?.toInt() ?? 0,
        titre: j['title'] as String? ?? '',
        corps: j['body'] as String? ?? '',
        idUtilisateur: (j['userId'] as num?)?.toInt() ?? 0,
      );

  Map<String, dynamic> toJson() => {
        'title': titre,
        'body': corps,
        'userId': idUtilisateur,
      };
}

class ErreurApi implements Exception {
  final String message;
  final int? codeStatut;
  const ErreurApi(this.message, {this.codeStatut});
  @override
  String toString() => message;
}

class ServiceArticles {
  final http.Client _client = http.Client();
  static const _hote = 'jsonplaceholder.typicode.com';

  Future<Article> creer({
    required String titre,
    required String corps,
    int idUtilisateur = 1,
  }) async {
    try {
      final reponse = await _client
          .post(
            Uri.https(_hote, '/posts'),
            headers: const {
              'Content-Type': 'application/json; charset=utf-8',
              'Accept': 'application/json',
            },
            body: jsonEncode({
              'title': titre,
              'body': corps,
              'userId': idUtilisateur,
            }),
          )
          .timeout(const Duration(seconds: 15));

      if (reponse.statusCode != 201 && reponse.statusCode != 200) {
        throw ErreurApi(
          'Création refusée (${reponse.statusCode}).',
          codeStatut: reponse.statusCode,
        );
      }

      return Article.fromJson(
        jsonDecode(utf8.decode(reponse.bodyBytes)) as Map<String, dynamic>,
      );
    } on TimeoutException {
      throw const ErreurApi('Le serveur met trop de temps à répondre.');
    } on SocketException {
      throw const ErreurApi('Aucune connexion Internet.');
    } on http.ClientException {
      throw const ErreurApi('Connexion interrompue.');
    } on FormatException {
      throw const ErreurApi('Réponse illisible du serveur.');
    }
  }

  void fermer() => _client.close();
}

void main() => runApp(const ApplicationPost());

class ApplicationPost extends StatelessWidget {
  const ApplicationPost({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.pink, useMaterial3: true),
      home: const PagePost(),
    );
  }
}

class PagePost extends StatefulWidget {
  const PagePost({super.key});

  @override
  State<PagePost> createState() => _PagePostState();
}

class _PagePostState extends State<PagePost> {
  final _cleFormulaire = GlobalKey<FormState>();
  final _ctrlTitre = TextEditingController();
  final _ctrlCorps = TextEditingController();
  final ServiceArticles _service = ServiceArticles();

  bool _envoiEnCours = false;
  Article? _dernierCree;

  @override
  void dispose() {
    _ctrlTitre.dispose();
    _ctrlCorps.dispose();
    _service.fermer();
    super.dispose();
  }

  Future<void> _envoyer() async {
    if (!_cleFormulaire.currentState!.validate()) return;

    setState(() => _envoiEnCours = true);
    try {
      final cree = await _service.creer(
        titre: _ctrlTitre.text.trim(),
        corps: _ctrlCorps.text.trim(),
      );
      if (!mounted) return;
      setState(() {
        _dernierCree = cree;
        _envoiEnCours = false;
      });
      _ctrlTitre.clear();
      _ctrlCorps.clear();
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Article créé, identifiant ${cree.id}.')),
      );
    } catch (e) {
      if (!mounted) return;
      setState(() => _envoiEnCours = false);
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('$e'),
          backgroundColor: Theme.of(context).colorScheme.error,
        ),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('53.29 — http.post()')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: _cleFormulaire,
          child: ListView(
            children: [
              TextFormField(
                controller: _ctrlTitre,
                decoration: const InputDecoration(
                  labelText: 'Titre',
                  border: OutlineInputBorder(),
                ),
                validator: (v) => (v == null || v.trim().length < 3)
                    ? 'Trois caractères minimum.'
                    : null,
              ),
              const SizedBox(height: 16),
              TextFormField(
                controller: _ctrlCorps,
                maxLines: 4,
                decoration: const InputDecoration(
                  labelText: 'Contenu',
                  border: OutlineInputBorder(),
                  alignLabelWithHint: true,
                ),
                validator: (v) => (v == null || v.trim().isEmpty)
                    ? 'Le contenu est obligatoire.'
                    : null,
              ),
              const SizedBox(height: 24),
              FilledButton.icon(
                // Bouton désactivé pendant l'envoi : sinon deux appuis
                // rapides créent DEUX articles (POST non idempotent).
                onPressed: _envoiEnCours ? null : _envoyer,
                icon: _envoiEnCours
                    ? const SizedBox(
                        width: 18,
                        height: 18,
                        child: CircularProgressIndicator(strokeWidth: 2),
                      )
                    : const Icon(Icons.send),
                label: Text(_envoiEnCours ? 'Envoi en cours...' : 'Envoyer'),
              ),
              if (_dernierCree != null) ...[
                const SizedBox(height: 32),
                Card(
                  child: ListTile(
                    leading: const Icon(Icons.check_circle, color: Colors.green),
                    title: Text('Créé : ${_dernierCree!.titre}'),
                    subtitle: Text(
                      'Identifiant attribué par le serveur : '
                      '${_dernierCree!.id}',
                    ),
                  ),
                ),
              ],
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
Saisie « L'épée de Cendre » / « Forgée dans les cendres. » puis « Envoyer » :
  le bouton se désactive et affiche « Envoi en cours... »
  puis une SnackBar : « Article créé, identifiant 101. »
  et une carte verte : « Créé : L'épée de Cendre »
```

> **Le détail non négociable :** le bouton est désactivé pendant l'envoi. `POST` n'est pas idempotent (section 53.4). Deux appuis rapides sur un réseau lent créent deux ressources, et l'utilisateur ne comprendra pas pourquoi il a deux commandes.

---

## 53.30 — `PUT`, `PATCH`, `DELETE`

Les trois autres verbes s'écrivent exactement comme `post`, avec la même signature.

```dart
// PUT — remplacement complet : TOUS les champs.
final r1 = await _client.put(
  Uri.https(_hote, '/posts/1'),
  headers: const {'Content-Type': 'application/json; charset=utf-8'},
  body: jsonEncode({
    'id': 1,
    'title': 'Titre entièrement remplacé',
    'body': 'Contenu entièrement remplacé',
    'userId': 1,
  }),
);

// PATCH — modification partielle : seulement ce qui change.
final r2 = await _client.patch(
  Uri.https(_hote, '/posts/1'),
  headers: const {'Content-Type': 'application/json; charset=utf-8'},
  body: jsonEncode({'title': 'Nouveau titre seulement'}),
);

// DELETE — pas de corps.
final r3 = await _client.delete(Uri.https(_hote, '/posts/1'));
```

Le service complet, méthode par méthode :

```dart
class ServiceArticlesEcriture {
  final http.Client _client = http.Client();
  static const _hote = 'jsonplaceholder.typicode.com';
  static const _enTetesJson = {
    'Content-Type': 'application/json; charset=utf-8',
    'Accept': 'application/json',
  };

  Future<Article> remplacer(Article article) async {
    final r = await _client
        .put(
          Uri.https(_hote, '/posts/${article.id}'),
          headers: _enTetesJson,
          body: jsonEncode({
            'id': article.id,
            'title': article.titre,
            'body': article.corps,
            'userId': article.idUtilisateur,
          }),
        )
        .timeout(const Duration(seconds: 15));

    if (r.statusCode != 200) {
      throw ErreurApi('Mise à jour refusée (${r.statusCode}).',
          codeStatut: r.statusCode);
    }
    return Article.fromJson(
        jsonDecode(utf8.decode(r.bodyBytes)) as Map<String, dynamic>);
  }

  Future<Article> modifierTitre(int id, String nouveauTitre) async {
    final r = await _client
        .patch(
          Uri.https(_hote, '/posts/$id'),
          headers: _enTetesJson,
          body: jsonEncode({'title': nouveauTitre}),
        )
        .timeout(const Duration(seconds: 15));

    if (r.statusCode != 200) {
      throw ErreurApi('Modification refusée (${r.statusCode}).',
          codeStatut: r.statusCode);
    }
    return Article.fromJson(
        jsonDecode(utf8.decode(r.bodyBytes)) as Map<String, dynamic>);
  }

  Future<void> supprimer(int id) async {
    final r = await _client
        .delete(Uri.https(_hote, '/posts/$id'), headers: _enTetesJson)
        .timeout(const Duration(seconds: 15));

    // 200 et 204 sont deux succès valides. On ne décode RIEN sur un 204 :
    // le corps est vide, et jsonDecode('') lève une FormatException.
    if (r.statusCode != 200 && r.statusCode != 204) {
      throw ErreurApi('Suppression refusée (${r.statusCode}).',
          codeStatut: r.statusCode);
    }
  }

  void fermer() => _client.close();
}
```

### Le tableau des statuts attendus

| Verbe | Statut de succès | Corps de la réponse | À décoder ? |
| --- | --- | --- | --- |
| `GET` | `200` | La ressource | Oui |
| `POST` | `201` (parfois `200`) | La ressource créée, avec son `id` | Oui |
| `PUT` | `200` (parfois `204`) | La ressource mise à jour | Si `200` |
| `PATCH` | `200` | La ressource mise à jour | Oui |
| `DELETE` | `204` (parfois `200`) | Vide sur `204` | **Non** sur `204` |

> **Le piège du `204` :** `jsonDecode('')` lève `FormatException: Unexpected end of input`. Testez toujours `if (reponse.body.isEmpty) return;` avant de décoder après une suppression.

### La mise à jour optimiste

Un raffinement d'expérience utilisateur très employé. Sur une suppression, deux stratégies :

```text
STRATÉGIE PESSIMISTE (sûre, mais lente à l'œil)
  1. L'utilisateur appuie sur « Supprimer ».
  2. Indicateur de chargement.
  3. Requête DELETE.  ← 800 ms d'attente visible
  4. Succès : on retire l'élément de la liste.

STRATÉGIE OPTIMISTE (instantanée, mais il faut savoir revenir en arrière)
  1. L'utilisateur appuie sur « Supprimer ».
  2. On retire IMMÉDIATEMENT l'élément de la liste. L'écran répond au doigt.
  3. Requête DELETE en arrière-plan.
  4a. Succès : rien à faire, c'est déjà fait.
  4b. Échec : on REMET l'élément et on affiche « La suppression a échoué. »
```

En code :

```dart
Future<void> _supprimer(Article article) async {
  final index = _articles.indexOf(article);
  setState(() => _articles.remove(article));      // optimiste

  try {
    await _service.supprimer(article.id);
  } catch (e) {
    if (!mounted) return;
    setState(() => _articles.insert(index, article));   // annulation
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('Suppression impossible : $e')),
    );
  }
}
```

Notez qu'on mémorise l'`index` **avant** de retirer, pour remettre l'élément exactement à sa place.

---

## 53.31 — L'authentification par en-tête `Authorization`

La plupart des vraies API exigent de savoir qui vous êtes. Le mécanisme standard est l'en-tête `Authorization`.

### Les trois schémas courants

```text
1. BEARER (jeton, le plus répandu — OAuth 2, JWT)
   Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

2. BASIC (identifiant:motdepasse encodés en base64 — obsolète, à éviter)
   Authorization: Basic YWxpYTpzZWNyZXQ=

3. CLÉ D'API (en-tête propre au fournisseur, pas de norme)
   X-API-Key: 3f8a9b2c...
   ou en paramètre : ?apikey=3f8a9b2c...
```

En Dart, dans les trois cas, c'est une simple entrée de la `Map` d'en-têtes.

```dart
import 'dart:convert';

void main() {
  // Bearer.
  const jeton = 'eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxIn0.abc';
  final enTetesBearer = {'Authorization': 'Bearer $jeton'};
  print(enTetesBearer);

  // Basic : identifiant:motdepasse en base64.
  const identifiant = 'alia';
  const motDePasse = 'secret';
  final code = base64Encode(utf8.encode('$identifiant:$motDePasse'));
  final enTetesBasic = {'Authorization': 'Basic $code'};
  print(enTetesBasic);

  // On peut décoder — d'où le fait que Basic n'est PAS sécurisé
  // sans HTTPS : le mot de passe circule presque en clair.
  print(utf8.decode(base64Decode(code)));
}
```

**Résultat :**

```text
{Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxIn0.abc}
{Authorization: Basic YWxpYTpzZWNyZXQ=}
alia:secret
```

### Centraliser l'authentification

Répéter l'en-tête dans chaque méthode est source d'oubli. On l'injecte une fois, au niveau du service.

```dart
class ServiceAuthentifie {
  final http.Client _client;
  final String _hote;
  String? _jeton;

  ServiceAuthentifie({http.Client? client, required String hote})
      : _client = client ?? http.Client(),
        _hote = hote;

  /// Appelé après la connexion de l'utilisateur.
  void definirJeton(String jeton) => _jeton = jeton;

  /// Appelé à la déconnexion.
  void effacerJeton() => _jeton = null;

  bool get estConnecte => _jeton != null;

  Map<String, String> get _enTetes => {
        'Accept': 'application/json',
        'Content-Type': 'application/json; charset=utf-8',
        if (_jeton != null) 'Authorization': 'Bearer $_jeton',
      };

  Future<dynamic> obtenir(String chemin,
      {Map<String, String>? parametres}) async {
    final r = await _client
        .get(Uri.https(_hote, chemin, parametres), headers: _enTetes)
        .timeout(const Duration(seconds: 10));

    // Le jeton a expiré : on le jette et on prévient l'appelant,
    // qui renverra vers l'écran de connexion.
    if (r.statusCode == 401) {
      effacerJeton();
      throw const ErreurApi('Session expirée. Reconnectez-vous.',
          codeStatut: 401);
    }
    if (r.statusCode == 403) {
      throw const ErreurApi('Vous n\'avez pas les droits nécessaires.',
          codeStatut: 403);
    }
    if (r.statusCode < 200 || r.statusCode >= 300) {
      throw ErreurApi('Erreur ${r.statusCode}.', codeStatut: r.statusCode);
    }
    return jsonDecode(utf8.decode(r.bodyBytes));
  }

  void fermer() => _client.close();
}
```

La syntaxe clé est la **collection `if`** dans le littéral de `Map` :

```dart
if (_jeton != null) 'Authorization': 'Bearer $_jeton',
```

L'en-tête n'est ajouté que si le jeton existe. Pas de `null` envoyé au serveur, pas de `Map` construite en deux temps.

### Distinguer 401 et 403

C'est une confusion très fréquente, et elle produit de mauvais parcours utilisateur.

| Code | Sens | Ce que fait l'application |
| --- | --- | --- |
| `401 Unauthorized` | « Je ne sais pas qui vous êtes » | Effacer le jeton, renvoyer vers la connexion |
| `403 Forbidden` | « Je sais qui vous êtes, et vous n'avez pas le droit » | Message d'explication. **Ne pas** déconnecter. |

Déconnecter l'utilisateur sur un `403` est un bug pénible : il essaie d'accéder à une page réservée aux administrateurs, et se retrouve éjecté de l'application entière.

---

## 53.32 — Ne jamais mettre une clé d'API dans le code source

C'est une règle, pas un conseil.

```dart
// À NE JAMAIS ÉCRIRE
const cleApi = 'sk_live_51H8xQ2eZvKYlo2C...';
```

### Pourquoi c'est une faille, même dans une application compilée

Un débutant pense : « le code est compilé, personne ne peut le lire ». C'est faux, pour trois raisons.

1. **Le dépôt Git.** Si le fichier part sur un dépôt public, la clé est indexée en quelques minutes par des robots qui scrutent GitHub en continu. Et même après suppression, elle reste dans l'historique.
2. **L'application publiée.** Un fichier `.apk` ou `.ipa` se décompresse comme une archive. Les chaînes de caractères en dur sont visibles avec un simple `strings`. C'est l'affaire de deux minutes.
3. **Le trafic réseau.** Un proxy d'inspection sur un appareil rooté montre chaque en-tête envoyé.

Une clé fuitée, ce sont des factures d'API à quatre chiffres, ou des données de vos utilisateurs aspirées.

### Ce qu'il faut faire

**Solution 1 — `--dart-define` (bonne, pour une clé publique à faible risque).**

La valeur est fournie à la compilation, pas écrite dans le code.

```dart
const cleApi = String.fromEnvironment('CLE_API', defaultValue: '');
```

Puis à l'exécution ou à la compilation :

```text
flutter run --dart-define=CLE_API=abc123
flutter build apk --dart-define=CLE_API=abc123
```

En pratique, on regroupe les définitions dans un fichier, exclu de Git :

```json
{
  "CLE_API": "abc123",
  "URL_BASE": "https://api.exemple.com"
}
```

```text
flutter run --dart-define-from-file=config/dev.json
```

Et dans `.gitignore` :

```text
config/dev.json
config/prod.json
*.env
```

> **Ce que `--dart-define` résout et ne résout pas :** il empêche la clé d'arriver dans Git. Il **n'empêche pas** de l'extraire du binaire publié. C'est déjà beaucoup, mais ce n'est pas de la sécurité.

**Solution 2 — le serveur intermédiaire (la seule vraie solution).**

Le principe : votre application n'a **jamais** la clé.

```text
   SANS INTERMÉDIAIRE (dangereux)

   Application  ──[clé secrète]──>  API payante
       ▲
       └── la clé est dans le binaire, extractible


   AVEC INTERMÉDIAIRE (sûr)

   Application  ──[jeton utilisateur]──>  VOTRE serveur
                                              │
                                              │ [clé secrète, jamais
                                              │  sortie du serveur]
                                              v
                                          API payante
```

Votre serveur détient la clé, applique vos quotas, et journalise les abus. Si le jeton d'un utilisateur fuite, vous le révoquez sans toucher à la clé maîtresse.

### Le tableau de décision

| Type de secret | Peut-il être dans l'application ? |
| --- | --- |
| URL de base de l'API | Oui, ce n'est pas un secret |
| Clé publique d'un service (Firebase, Maps avec restriction) | Oui, si le fournisseur la restreint par empreinte d'application |
| Clé d'API facturée à l'usage | **Non.** Serveur intermédiaire obligatoire |
| Secret client OAuth | **Non**, jamais |
| Mot de passe de base de données | **Non**, jamais |
| Jeton de l'utilisateur connecté | Oui, mais en stockage sécurisé (chapitre 54) |

---

## 53.33 — `StreamBuilder` et les données qui arrivent en continu

Un `Future` livre **une** valeur, puis c'est fini. Un `Stream` (chapitre 15) livre **plusieurs** valeurs, étalées dans le temps.

```text
   FUTURE                          STREAM
   ──────                          ──────
   ○········>●                     ○···●···●···●···●···>
   lancé     une valeur            lancé  v1  v2  v3  v4 ... (peut ne
             puis terminé                                     jamais finir)
```

Quand utilise-t-on un `Stream` en réseau ?

| Cas | Technologie |
| --- | --- |
| Messagerie instantanée, notifications | WebSocket |
| Cours de bourse, score en direct | WebSocket, SSE |
| Position GPS, capteurs | Flux du système |
| Base de données temps réel (Firestore) | Flux du SDK |
| Interrogation périodique d'une API REST | `Stream.periodic` |
| Progression d'un téléchargement | `StreamedResponse` de `http` |

Le widget est `StreamBuilder`, avec **exactement** la même forme que `FutureBuilder` :

```dart
StreamBuilder({
  Key? key,
  required Stream<T>? stream,
  T? initialData,
  required AsyncWidgetBuilder<T> builder,
})
```

Une seule différence de comportement : `connectionState` passe par **`active`**, et le `builder` est appelé **à chaque valeur émise**.

```text
   FUTURE :  waiting  ->  done
   STREAM  :  waiting  ->  active  ->  active  ->  active  ->  done
```

Voici une application qui interroge une API toutes les cinq secondes et affiche la valeur reçue en continu.

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const ApplicationStream());

class ApplicationStream extends StatelessWidget {
  const ApplicationStream({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.amber, useMaterial3: true),
      home: const PageStream(),
    );
  }
}

class ReleveScore {
  final int idArticle;
  final String titre;
  final DateTime instant;

  const ReleveScore({
    required this.idArticle,
    required this.titre,
    required this.instant,
  });
}

class ServiceReleve {
  final http.Client _client = http.Client();
  int _compteur = 1;

  /// Interroge l'API toutes les [periode] et émet chaque résultat.
  /// async* : cette fonction produit un Stream (chapitre 15).
  Stream<ReleveScore> flux({
    Duration periode = const Duration(seconds: 5),
  }) async* {
    while (true) {
      try {
        final r = await _client
            .get(Uri.https('jsonplaceholder.typicode.com', '/posts/$_compteur'))
            .timeout(const Duration(seconds: 8));

        if (r.statusCode == 200) {
          final j = jsonDecode(utf8.decode(r.bodyBytes)) as Map<String, dynamic>;
          yield ReleveScore(
            idArticle: (j['id'] as num).toInt(),
            titre: j['title'] as String,
            instant: DateTime.now(),
          );
        }
      } on TimeoutException {
        // On saute ce relevé et on réessaiera au prochain tour.
      } on SocketException {
        // Idem : un flux ne doit pas mourir sur une coupure passagère.
      } on http.ClientException {
        // Idem.
      }

      _compteur = _compteur % 100 + 1;
      await Future<void>.delayed(periode);
    }
  }

  void fermer() => _client.close();
}

class PageStream extends StatefulWidget {
  const PageStream({super.key});

  @override
  State<PageStream> createState() => _PageStreamState();
}

class _PageStreamState extends State<PageStream> {
  final ServiceReleve _service = ServiceReleve();

  // Le Stream, comme le Future, est créé UNE SEULE FOIS.
  // Le créer dans build() relancerait une interrogation infinie
  // à chaque image : encore pire que pour un Future.
  late final Stream<ReleveScore> _flux =
      _service.flux(periode: const Duration(seconds: 5)).asBroadcastStream();

  final List<ReleveScore> _historique = [];

  @override
  void dispose() {
    _service.fermer();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('53.33 — StreamBuilder')),
      body: StreamBuilder<ReleveScore>(
        stream: _flux,
        builder: (context, snapshot) {
          if (snapshot.hasData &&
              (_historique.isEmpty ||
                  _historique.first.instant != snapshot.data!.instant)) {
            _historique.insert(0, snapshot.data!);
            if (_historique.length > 20) _historique.removeLast();
          }

          return Column(
            children: [
              Card(
                margin: const EdgeInsets.all(12),
                child: Padding(
                  padding: const EdgeInsets.all(16),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text('connectionState : '
                          '${snapshot.connectionState.name}'),
                      Text('relevés reçus   : ${_historique.length}'),
                      const SizedBox(height: 8),
                      if (snapshot.connectionState == ConnectionState.waiting)
                        const Row(
                          children: [
                            SizedBox(
                              width: 16,
                              height: 16,
                              child: CircularProgressIndicator(strokeWidth: 2),
                            ),
                            SizedBox(width: 12),
                            Text('Premier relevé en cours...'),
                          ],
                        )
                      else if (snapshot.hasData)
                        Text(
                          snapshot.data!.titre,
                          style: Theme.of(context).textTheme.titleMedium,
                        ),
                    ],
                  ),
                ),
              ),
              const Divider(height: 1),
              Expanded(
                child: ListView.builder(
                  itemCount: _historique.length,
                  itemBuilder: (context, i) {
                    final r = _historique[i];
                    final h = r.instant;
                    return ListTile(
                      dense: true,
                      leading: Text('#${r.idArticle}'),
                      title: Text(r.titre, maxLines: 1,
                          overflow: TextOverflow.ellipsis),
                      trailing: Text(
                        '${h.hour.toString().padLeft(2, '0')}:'
                        '${h.minute.toString().padLeft(2, '0')}:'
                        '${h.second.toString().padLeft(2, '0')}',
                        style: const TextStyle(fontSize: 11),
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

**Résultat :**

```text
t=0 s   connectionState : waiting      « Premier relevé en cours... »
t=1 s   connectionState : active       « sunt aut facere repellat provident »
        relevés reçus   : 1
t=6 s   connectionState : active       « qui est esse »
        relevés reçus   : 2
t=11 s  connectionState : active       « ea molestias quasi exercitationem »
        relevés reçus   : 3
... et ainsi de suite, indéfiniment
```

### `FutureBuilder` ou `StreamBuilder` ?

| Question | `FutureBuilder` | `StreamBuilder` |
| --- | --- | --- |
| Combien de valeurs ? | Une | Plusieurs |
| Se termine ? | Toujours | Parfois jamais |
| `connectionState.active` | Jamais | Oui |
| Faut-il annuler ? | Non | **Oui**, sinon fuite |
| Cas typique | Charger une liste | Messagerie, capteurs |

> **Attention aux fuites.** Un `Stream` infini créé dans un `State` continue de tourner après la fermeture de l'écran si personne ne l'arrête. `StreamBuilder` se désabonne bien, mais la boucle `while (true)` de la source, elle, continue. En production, on gère explicitement un `StreamController` et on le ferme dans `dispose()`.

---

## 53.34 — La pagination

Une API ne renvoie jamais dix mille éléments d'un coup. Elle les découpe en **pages**.

### Les trois conventions

```text
1. OFFSET / LIMIT  (la plus fréquente)
   GET /produits?limit=20&skip=0     -> éléments 1 à 20
   GET /produits?limit=20&skip=20    -> éléments 21 à 40
   Réponse : { "products": [...], "total": 194, "skip": 20, "limit": 20 }

2. PAGE / PER_PAGE
   GET /produits?page=1&per_page=20
   GET /produits?page=2&per_page=20

3. CURSEUR  (la plus robuste pour des données qui bougent)
   GET /produits?limit=20
   Réponse : { "items": [...], "next_cursor": "eyJpZCI6MjB9" }
   GET /produits?limit=20&cursor=eyJpZCI6MjB9
```

Les deux premières ont un défaut : si un élément est inséré entre deux requêtes, tout décale et un élément apparaît deux fois. La troisième l'évite. En apprentissage, on utilise la première : c'est celle de DummyJSON.

### Le mécanisme du chargement infini

```text
   L'utilisateur fait défiler
             │
             v
   A-t-on dépassé 80 % de la hauteur ?
             │  oui
             v
   Une requête est-elle DÉJÀ en cours ?  ─── oui ──> ne rien faire
             │  non
             v
   Reste-t-il des éléments ?  ─── non ──> afficher « Fin de la liste »
             │  oui
             v
   charger la page suivante (skip += limit)
             │
             v
   AJOUTER les résultats à la liste existante (jamais remplacer)
```

Deux garde-fous sont indispensables :

- **`_chargementEnCours`** : sans lui, le défilement déclenche vingt requêtes identiques par seconde.
- **`_toutCharge`** : sans lui, on interroge le serveur à l'infini une fois arrivé au bout.

Voici l'application complète.

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

class Produit {
  final int id;
  final String titre;
  final double prix;
  final String? vignette;

  const Produit({
    required this.id,
    required this.titre,
    required this.prix,
    this.vignette,
  });

  factory Produit.fromJson(Map<String, dynamic> j) => Produit(
        id: (j['id'] as num?)?.toInt() ?? 0,
        titre: j['title'] as String? ?? 'Sans titre',
        prix: (j['price'] as num?)?.toDouble() ?? 0,
        vignette: j['thumbnail'] as String?,
      );
}

class PageProduits {
  final List<Produit> produits;
  final int total;

  const PageProduits({required this.produits, required this.total});

  factory PageProduits.fromJson(Map<String, dynamic> j) => PageProduits(
        produits: (j['products'] as List<dynamic>? ?? const [])
            .map((e) => Produit.fromJson(e as Map<String, dynamic>))
            .toList(),
        total: (j['total'] as num?)?.toInt() ?? 0,
      );
}

class ErreurApi implements Exception {
  final String message;
  const ErreurApi(this.message);
  @override
  String toString() => message;
}

class ServiceProduits {
  final http.Client _client = http.Client();

  Future<PageProduits> page({required int saut, int limite = 20}) async {
    try {
      final r = await _client
          .get(Uri.https('dummyjson.com', '/products', {
            'limit': '$limite',
            'skip': '$saut',
            // On ne demande que les champs utiles : réponse plus légère.
            'select': 'title,price,thumbnail',
          }))
          .timeout(const Duration(seconds: 10));

      if (r.statusCode != 200) {
        throw ErreurApi('Erreur ${r.statusCode}.');
      }
      return PageProduits.fromJson(
          jsonDecode(utf8.decode(r.bodyBytes)) as Map<String, dynamic>);
    } on TimeoutException {
      throw const ErreurApi('Délai dépassé.');
    } on SocketException {
      throw const ErreurApi('Aucune connexion Internet.');
    } on http.ClientException {
      throw const ErreurApi('Connexion interrompue.');
    } on FormatException {
      throw const ErreurApi('Réponse illisible.');
    }
  }

  void fermer() => _client.close();
}

void main() => runApp(const ApplicationPagination());

class ApplicationPagination extends StatelessWidget {
  const ApplicationPagination({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.brown, useMaterial3: true),
      home: const PagePagination(),
    );
  }
}

class PagePagination extends StatefulWidget {
  const PagePagination({super.key});

  @override
  State<PagePagination> createState() => _PagePaginationState();
}

class _PagePaginationState extends State<PagePagination> {
  static const int _taillePage = 20;

  final ServiceProduits _service = ServiceProduits();
  final ScrollController _defilement = ScrollController();

  final List<Produit> _produits = [];
  bool _chargementEnCours = false;
  bool _toutCharge = false;
  String? _erreur;
  int _total = 0;

  @override
  void initState() {
    super.initState();
    _defilement.addListener(_surDefilement);
    _chargerPageSuivante();
  }

  @override
  void dispose() {
    _defilement.removeListener(_surDefilement);
    _defilement.dispose();
    _service.fermer();
    super.dispose();
  }

  void _surDefilement() {
    if (!_defilement.hasClients) return;
    final seuil = _defilement.position.maxScrollExtent * 0.8;
    if (_defilement.position.pixels >= seuil) {
      _chargerPageSuivante();
    }
  }

  Future<void> _chargerPageSuivante() async {
    // GARDE-FOU 1 : une seule requête à la fois.
    if (_chargementEnCours) return;
    // GARDE-FOU 2 : on s'arrête quand tout est chargé.
    if (_toutCharge) return;

    setState(() {
      _chargementEnCours = true;
      _erreur = null;
    });

    try {
      final page = await _service.page(
        saut: _produits.length,
        limite: _taillePage,
      );
      if (!mounted) return;
      setState(() {
        // ON AJOUTE, on ne remplace jamais.
        _produits.addAll(page.produits);
        _total = page.total;
        _toutCharge =
            page.produits.isEmpty || _produits.length >= page.total;
        _chargementEnCours = false;
      });
    } catch (e) {
      if (!mounted) return;
      setState(() {
        _erreur = '$e';
        _chargementEnCours = false;
      });
    }
  }

  Future<void> _repartirDeZero() async {
    setState(() {
      _produits.clear();
      _toutCharge = false;
      _erreur = null;
    });
    await _chargerPageSuivante();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('53.34 — ${_produits.length} / $_total'),
      ),
      body: RefreshIndicator(
        onRefresh: _repartirDeZero,
        child: ListView.builder(
          controller: _defilement,
          physics: const AlwaysScrollableScrollPhysics(),
          // +1 pour la ligne de pied de liste.
          itemCount: _produits.length + 1,
          itemBuilder: (context, i) {
            if (i < _produits.length) {
              final p = _produits[i];
              return ListTile(
                leading: p.vignette == null
                    ? const CircleAvatar(child: Icon(Icons.inventory_2))
                    : CircleAvatar(backgroundImage: NetworkImage(p.vignette!)),
                title: Text(p.titre),
                subtitle: Text('${p.prix.toStringAsFixed(2)} EUR'),
                trailing: Text('#${i + 1}'),
              );
            }
            return _PiedDeListe(
              chargement: _chargementEnCours,
              erreur: _erreur,
              termine: _toutCharge,
              onReessayer: _chargerPageSuivante,
            );
          },
        ),
      ),
    );
  }
}

class _PiedDeListe extends StatelessWidget {
  final bool chargement;
  final String? erreur;
  final bool termine;
  final VoidCallback onReessayer;

  const _PiedDeListe({
    required this.chargement,
    required this.erreur,
    required this.termine,
    required this.onReessayer,
  });

  @override
  Widget build(BuildContext context) {
    if (erreur != null) {
      return Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          children: [
            Text(erreur!, textAlign: TextAlign.center),
            const SizedBox(height: 12),
            FilledButton.tonal(
              onPressed: onReessayer,
              child: const Text('Réessayer'),
            ),
          ],
        ),
      );
    }
    if (chargement) {
      return const Padding(
        padding: EdgeInsets.all(24),
        child: Center(child: CircularProgressIndicator()),
      );
    }
    if (termine) {
      return const Padding(
        padding: EdgeInsets.all(24),
        child: Center(child: Text('Fin de la liste.')),
      );
    }
    return const SizedBox(height: 48);
  }
}
```

**Résultat :**

```text
Ouverture      : « 0 / 0 » puis « 20 / 194 », 20 produits affichés.
Défilement     : à 80 % de la liste, l'indicateur apparaît en bas,
                 puis « 40 / 194 », 40 produits.
Encore         : « 60 / 194 », « 80 / 194 »...
Arrivé au bout : « 194 / 194 » et « Fin de la liste. »
Tirer vers le bas : la liste repart de zéro.
```

> **Le détail à ne pas manquer :** `_produits.addAll(...)`, jamais `_produits = ...`. Remplacer la liste ferait perdre les pages déjà chargées et remonterait le défilement en haut.

---

## 53.35 — Le débounce d'une recherche

Un champ de recherche qui appelle l'API à chaque frappe est une catastrophe.

```text
   L'utilisateur tape « épée » (4 caractères)

   SANS DÉBOUNCE                    AVEC DÉBOUNCE (400 ms)
   ─────────────                    ──────────────────────
   'é'    -> requête 1              'é'    -> minuteur armé
   'ép'   -> requête 2              'ép'   -> minuteur RÉarmé
   'épé'  -> requête 3              'épé'  -> minuteur RÉarmé
   'épée' -> requête 4              'épée' -> minuteur RÉarmé
                                    ... 400 ms de silence ...
   4 requêtes                       -> requête 1
   Les réponses arrivent dans
   le désordre : la réponse 2       1 requête
   peut arriver APRÈS la 4,         Pas de désordre possible.
   et écraser le bon résultat.
```

Le **débounce** consiste à attendre que l'utilisateur ait arrêté de taper pendant un court instant avant de lancer la requête. On l'implémente avec un `Timer` de `dart:async`.

```dart
Timer? _minuteur;

void _surChangementTexte(String texte) {
  _minuteur?.cancel();                 // on annule le précédent
  _minuteur = Timer(
    const Duration(milliseconds: 400),
    () => _rechercher(texte),          // exécuté seulement si 400 ms de calme
  );
}

@override
void dispose() {
  _minuteur?.cancel();                 // OBLIGATOIRE : sinon fuite
  super.dispose();
}
```

### Le second problème : les réponses dans le désordre

Même avec un débounce, deux requêtes peuvent se chevaucher si l'utilisateur reprend sa saisie. La réponse de « ép » peut arriver après celle de « épée » et écraser le bon résultat.

La parade : un **jeton de séquence**. Chaque recherche reçoit un numéro ; on n'affiche que la réponse du numéro le plus récent.

```dart
int _numeroRecherche = 0;

Future<void> _rechercher(String requete) async {
  final monNumero = ++_numeroRecherche;
  final resultats = await _service.chercher(requete);

  // Une recherche plus récente est partie entre-temps : on jette celle-ci.
  if (monNumero != _numeroRecherche) return;
  if (!mounted) return;

  setState(() => _resultats = resultats);
}
```

L'application complète, avec débounce, jeton de séquence, quatre états et annulation :

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

class Produit {
  final int id;
  final String titre;
  final double prix;
  final String categorie;

  const Produit({
    required this.id,
    required this.titre,
    required this.prix,
    required this.categorie,
  });

  factory Produit.fromJson(Map<String, dynamic> j) => Produit(
        id: (j['id'] as num?)?.toInt() ?? 0,
        titre: j['title'] as String? ?? '',
        prix: (j['price'] as num?)?.toDouble() ?? 0,
        categorie: j['category'] as String? ?? '',
      );
}

class ErreurApi implements Exception {
  final String message;
  const ErreurApi(this.message);
  @override
  String toString() => message;
}

class ServiceRecherche {
  final http.Client _client = http.Client();

  Future<List<Produit>> chercher(String requete) async {
    try {
      final r = await _client
          .get(Uri.https('dummyjson.com', '/products/search', {
            'q': requete,
            'limit': '30',
            'select': 'title,price,category',
          }))
          .timeout(const Duration(seconds: 8));

      if (r.statusCode != 200) throw ErreurApi('Erreur ${r.statusCode}.');

      final j = jsonDecode(utf8.decode(r.bodyBytes)) as Map<String, dynamic>;
      return (j['products'] as List<dynamic>? ?? const [])
          .map((e) => Produit.fromJson(e as Map<String, dynamic>))
          .toList();
    } on TimeoutException {
      throw const ErreurApi('La recherche met trop de temps.');
    } on SocketException {
      throw const ErreurApi('Aucune connexion Internet.');
    } on http.ClientException {
      throw const ErreurApi('Connexion interrompue.');
    } on FormatException {
      throw const ErreurApi('Réponse illisible.');
    }
  }

  void fermer() => _client.close();
}

void main() => runApp(const ApplicationRecherche());

class ApplicationRecherche extends StatelessWidget {
  const ApplicationRecherche({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.lightBlue, useMaterial3: true),
      home: const PageRecherche(),
    );
  }
}

class PageRecherche extends StatefulWidget {
  const PageRecherche({super.key});

  @override
  State<PageRecherche> createState() => _PageRechercheState();
}

class _PageRechercheState extends State<PageRecherche> {
  final ServiceRecherche _service = ServiceRecherche();
  final TextEditingController _ctrl = TextEditingController();

  Timer? _minuteur;
  int _numeroRecherche = 0;

  List<Produit> _resultats = const [];
  bool _enCours = false;
  String? _erreur;
  String _requeteAffichee = '';
  int _requetesEnvoyees = 0;

  @override
  void dispose() {
    _minuteur?.cancel();
    _ctrl.dispose();
    _service.fermer();
    super.dispose();
  }

  void _surTexte(String texte) {
    _minuteur?.cancel();

    final propre = texte.trim();
    if (propre.length < 2) {
      setState(() {
        _resultats = const [];
        _erreur = null;
        _enCours = false;
        _requeteAffichee = '';
      });
      return;
    }

    setState(() => _enCours = true);
    _minuteur = Timer(const Duration(milliseconds: 400), () {
      _lancer(propre);
    });
  }

  Future<void> _lancer(String requete) async {
    final monNumero = ++_numeroRecherche;
    _requetesEnvoyees++;

    try {
      final resultats = await _service.chercher(requete);
      if (monNumero != _numeroRecherche || !mounted) return;
      setState(() {
        _resultats = resultats;
        _erreur = null;
        _enCours = false;
        _requeteAffichee = requete;
      });
    } catch (e) {
      if (monNumero != _numeroRecherche || !mounted) return;
      setState(() {
        _erreur = '$e';
        _enCours = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('53.35 — Recherche ($_requetesEnvoyees requêtes)'),
      ),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(12),
            child: TextField(
              controller: _ctrl,
              onChanged: _surTexte,
              autofocus: true,
              decoration: InputDecoration(
                labelText: 'Rechercher un produit',
                hintText: 'phone, laptop, watch...',
                prefixIcon: const Icon(Icons.search),
                border: const OutlineInputBorder(),
                suffixIcon: _ctrl.text.isEmpty
                    ? null
                    : IconButton(
                        icon: const Icon(Icons.clear),
                        onPressed: () {
                          _ctrl.clear();
                          _surTexte('');
                        },
                      ),
              ),
            ),
          ),
          if (_enCours) const LinearProgressIndicator(minHeight: 2),
          Expanded(child: _corps()),
        ],
      ),
    );
  }

  Widget _corps() {
    if (_erreur != null) {
      return Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            const Icon(Icons.error_outline, size: 56),
            const SizedBox(height: 12),
            Text(_erreur!),
            const SizedBox(height: 16),
            FilledButton(
              onPressed: () => _lancer(_ctrl.text.trim()),
              child: const Text('Réessayer'),
            ),
          ],
        ),
      );
    }

    if (_ctrl.text.trim().length < 2) {
      return const Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Icon(Icons.search, size: 56),
            SizedBox(height: 12),
            Text('Saisissez au moins deux caractères.'),
          ],
        ),
      );
    }

    if (_resultats.isEmpty && !_enCours) {
      return Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            const Icon(Icons.search_off, size: 56),
            const SizedBox(height: 12),
            Text('Aucun résultat pour « $_requeteAffichee ».'),
          ],
        ),
      );
    }

    return ListView.separated(
      itemCount: _resultats.length,
      separatorBuilder: (_, __) => const Divider(height: 1),
      itemBuilder: (context, i) {
        final p = _resultats[i];
        return ListTile(
          title: Text(p.titre),
          subtitle: Text(p.categorie),
          trailing: Text('${p.prix.toStringAsFixed(2)} EUR'),
        );
      },
    );
  }
}
```

**Résultat :**

```text
Saisie rapide de « phone » (5 caractères) :
  la barre de progression apparaît dès la 1re lettre,
  UNE SEULE requête est envoyée 400 ms après la dernière frappe,
  le titre affiche « Recherche (1 requêtes) »,
  puis la liste des produits contenant « phone ».

Sans débounce, le titre afficherait « Recherche (5 requêtes) ».
```

### Le tableau des durées de débounce

| Contexte | Durée | Raison |
| --- | --- | --- |
| Recherche locale (sur une liste en mémoire) | 0 ms | Pas de réseau, inutile d'attendre |
| Suggestion d'autocomplétion | 200–300 ms | Doit sembler instantané |
| Recherche réseau classique | 400–500 ms | Le meilleur compromis |
| Requête coûteuse ou payante | 800–1000 ms | On économise les appels |
| Sauvegarde automatique d'un brouillon | 2000 ms | Personne n'attend |

---

## 53.36 — Annuler une requête devenue inutile

Le débounce évite de **lancer** des requêtes inutiles. Mais que faire d'une requête **déjà partie** dont on ne veut plus ?

Trois situations :

- l'utilisateur quitte l'écran pendant le chargement ;
- il lance une nouvelle recherche alors que la précédente tourne encore ;
- il annule volontairement un envoi.

### Ce que le package `http` permet, et ce qu'il ne permet pas

Le package `http` n'a **pas** de mécanisme d'annulation par requête. Vos options :

**Option 1 — le jeton de séquence (déjà vu en 53.35).**
La requête continue en arrière-plan, mais son résultat est ignoré. Simple, suffisant dans la majorité des cas. Coût : la bande passante est consommée pour rien.

**Option 2 — fermer le `Client`.**
`client.close()` interrompt **toutes** les requêtes en cours de ce client. Les `Future` en attente lèvent alors une `ClientException`. C'est brutal mais réel : la connexion est coupée.

```dart
@override
void dispose() {
  _service.fermer();   // ferme le http.Client -> coupe les requêtes en cours
  super.dispose();
}
```

**Option 3 — passer à `dio` et utiliser un `CancelToken` (section 53.37).**
La seule solution propre pour annuler une requête précise sans toucher aux autres.

### Le tableau comparatif

| Technique | Annule vraiment le trafic | Granularité | Complexité |
| --- | --- | --- | --- |
| Jeton de séquence | Non | Par requête | Faible |
| `client.close()` | Oui | Toutes les requêtes du client | Faible |
| `CancelToken` de `dio` | Oui | Par requête | Moyenne |

### Le patron complet avec `http`

```dart
class _EcranAnnulableState extends State<EcranAnnulable> {
  http.Client? _client;
  int _numero = 0;
  String _etat = 'inactif';

  Future<void> _charger() async {
    // On coupe la requête précédente, s'il y en a une.
    _client?.close();
    _client = http.Client();

    final monNumero = ++_numero;
    final client = _client!;

    setState(() => _etat = 'chargement');
    try {
      final r = await client
          .get(Uri.https('jsonplaceholder.typicode.com', '/photos'))
          .timeout(const Duration(seconds: 20));
      if (monNumero != _numero || !mounted) return;
      setState(() => _etat = 'reçu ${r.body.length} octets');
    } on http.ClientException {
      if (monNumero != _numero || !mounted) return;
      setState(() => _etat = 'annulé ou interrompu');
    } catch (e) {
      if (monNumero != _numero || !mounted) return;
      setState(() => _etat = 'erreur : $e');
    }
  }

  void _annuler() {
    _numero++;              // invalide la réponse à venir
    _client?.close();       // coupe la connexion
    _client = null;
    setState(() => _etat = 'annulé par l\'utilisateur');
  }

  @override
  void dispose() {
    _client?.close();       // capital : on ne laisse rien tourner
    super.dispose();
  }

  // ... build()
}
```

> **La règle simple à retenir :** tout `http.Client` créé dans un `State` doit être fermé dans `dispose()`. Toujours. Sans exception.

---

## 53.37 — `dio` : ce qu'il apporte de plus que `http`

`http` est le paquet officiel, minimal et suffisant pour la majorité des applications. **`dio`** est l'alternative la plus populaire, plus riche.

```text
flutter pub add dio
```

Au moment de la rédaction, la version courante est `5.11.0`. Comme toujours, laissez `flutter pub add` écrire le numéro.

### Le tableau de comparaison

| Fonctionnalité | `http` | `dio` |
| --- | --- | --- |
| Mainteneur | Équipe Dart officielle | Communauté (éditeur vérifié) |
| Taille du paquet | Petit | Plus gros |
| `GET` / `POST` / etc. | Oui | Oui |
| Décodage JSON automatique | **Non** (`jsonDecode` à la main) | **Oui** (`response.data` est déjà décodé) |
| URL de base configurable | Non | Oui (`options.baseUrl`) |
| Timeouts intégrés | Non (`.timeout()` à la main) | Oui (`connectTimeout`, `receiveTimeout`) |
| Intercepteurs | Non | **Oui** |
| Annulation par requête | Non | **Oui** (`CancelToken`) |
| Progression d'envoi / réception | Difficile | **Oui** (`onSendProgress`) |
| `FormData` / envoi de fichiers | `MultipartRequest`, verbeux | Simple |
| Erreur sur statut 4xx / 5xx | Non, il faut tester | **Oui** (`DioException` levée) |

### Les intercepteurs : l'argument décisif

Un intercepteur est un code exécuté **automatiquement** avant chaque requête et après chaque réponse. C'est l'endroit idéal pour : ajouter le jeton, journaliser, réessayer, rafraîchir un jeton expiré.

```dart
import 'package:dio/dio.dart';

Dio construireClient({required String Function() lireJeton}) {
  final dio = Dio(
    BaseOptions(
      baseUrl: 'https://api.exemple.com/v1',
      connectTimeout: const Duration(seconds: 10),
      receiveTimeout: const Duration(seconds: 10),
      headers: const {'Accept': 'application/json'},
    ),
  );

  dio.interceptors.add(
    InterceptorsWrapper(
      onRequest: (options, handler) {
        // Le jeton est ajouté à TOUTES les requêtes, sans rien écrire ailleurs.
        final jeton = lireJeton();
        if (jeton.isNotEmpty) {
          options.headers['Authorization'] = 'Bearer $jeton';
        }
        handler.next(options);
      },
      onResponse: (reponse, handler) {
        handler.next(reponse);
      },
      onError: (erreur, handler) {
        // Traduction centralisée des erreurs.
        handler.next(erreur);
      },
    ),
  );

  return dio;
}
```

Avec `http`, il faudrait ajouter l'en-tête dans chaque méthode du service, et un oubli passe inaperçu jusqu'à un `401` en production.

### Un appel `dio` complet

```dart
import 'package:dio/dio.dart';

class Produit {
  final int id;
  final String titre;
  final double prix;

  const Produit({required this.id, required this.titre, required this.prix});

  factory Produit.fromJson(Map<String, dynamic> j) => Produit(
        id: (j['id'] as num?)?.toInt() ?? 0,
        titre: j['title'] as String? ?? '',
        prix: (j['price'] as num?)?.toDouble() ?? 0,
      );
}

class ErreurApi implements Exception {
  final String message;
  const ErreurApi(this.message);
  @override
  String toString() => message;
}

class ServiceDio {
  final Dio _dio = Dio(
    BaseOptions(
      baseUrl: 'https://dummyjson.com',
      connectTimeout: const Duration(seconds: 10),
      receiveTimeout: const Duration(seconds: 10),
    ),
  );

  /// Le CancelToken permet d'annuler CETTE requête précisément.
  Future<List<Produit>> chercher(String q, {CancelToken? annulation}) async {
    try {
      // Pas de jsonDecode : dio l'a déjà fait. response.data est une Map.
      final reponse = await _dio.get<Map<String, dynamic>>(
        '/products/search',
        queryParameters: {'q': q, 'limit': 30},
        cancelToken: annulation,
      );

      final donnees = reponse.data ?? const {};
      return (donnees['products'] as List<dynamic>? ?? const [])
          .map((e) => Produit.fromJson(e as Map<String, dynamic>))
          .toList();
    } on DioException catch (e) {
      // Un 404 ou un 500 lève une DioException : pas besoin
      // de tester statusCode comme avec http.
      switch (e.type) {
        case DioExceptionType.cancel:
          throw const ErreurApi('Requête annulée.');
        case DioExceptionType.connectionTimeout:
        case DioExceptionType.receiveTimeout:
        case DioExceptionType.sendTimeout:
          throw const ErreurApi('Le serveur met trop de temps à répondre.');
        case DioExceptionType.connectionError:
          throw const ErreurApi('Aucune connexion Internet.');
        case DioExceptionType.badResponse:
          throw ErreurApi(
            'Le serveur a répondu ${e.response?.statusCode}.',
          );
        default:
          throw ErreurApi('Erreur réseau : ${e.message}');
      }
    }
  }

  void fermer() => _dio.close(force: true);
}
```

L'annulation devient triviale :

```dart
CancelToken? _annulation;

Future<void> _chercher(String q) async {
  _annulation?.cancel('Nouvelle recherche.');   // on coupe la précédente
  _annulation = CancelToken();
  final resultats = await _service.chercher(q, annulation: _annulation);
  // ...
}

@override
void dispose() {
  _annulation?.cancel('Écran fermé.');
  super.dispose();
}
```

### Lequel choisir ?

| Situation | Choix |
| --- | --- |
| Première application, quelques appels `GET` | `http` |
| Vous apprenez (ce chapitre, la PARTIE 1C) | `http` |
| Une application sans authentification | `http` |
| Authentification par jeton sur toutes les requêtes | `dio` |
| Rafraîchissement automatique de jeton | `dio` |
| Envoi de fichiers avec barre de progression | `dio` |
| Annulation fine indispensable | `dio` |
| Journalisation centralisée des appels | `dio` |

> **Le conseil :** commencez avec `http`. Vous comprendrez ce que `dio` automatise, et vous saurez le déboguer. Migrer vers `dio` plus tard ne prend qu'une journée si votre couche service est bien isolée (section 53.17) — c'est précisément l'intérêt de cette isolation.

---

## 53.38 — Simuler une API pour développer hors-ligne

Trois situations où vous voudrez travailler sans le vrai serveur :

- l'API n'existe pas encore, l'équipe serveur est en retard ;
- vous êtes dans un train, sans réseau ;
- vous voulez démontrer les états d'erreur, qu'un serveur en bonne santé ne produit jamais.

La solution : une **implémentation factice** du service, interchangeable avec la vraie. C'est ici que l'architecture en couches paie.

### L'interface abstraite

```dart
abstract class DepotArticles {
  Future<List<Article>> lister();
  Future<Article> obtenir(int id);
}
```

### Les deux implémentations

```dart
/// La vraie : appelle le réseau.
class DepotArticlesHttp implements DepotArticles {
  final http.Client _client;
  DepotArticlesHttp({http.Client? client}) : _client = client ?? http.Client();

  @override
  Future<List<Article>> lister() async {
    final r = await _client
        .get(Uri.https('jsonplaceholder.typicode.com', '/posts'))
        .timeout(const Duration(seconds: 10));
    if (r.statusCode != 200) throw ErreurApi('Erreur ${r.statusCode}.');
    return (jsonDecode(utf8.decode(r.bodyBytes)) as List<dynamic>)
        .map((e) => Article.fromJson(e as Map<String, dynamic>))
        .toList();
  }

  @override
  Future<Article> obtenir(int id) async {
    final r = await _client
        .get(Uri.https('jsonplaceholder.typicode.com', '/posts/$id'))
        .timeout(const Duration(seconds: 10));
    if (r.statusCode != 200) throw ErreurApi('Erreur ${r.statusCode}.');
    return Article.fromJson(
        jsonDecode(utf8.decode(r.bodyBytes)) as Map<String, dynamic>);
  }
}

/// La fausse : aucune connexion, données en dur, délais simulés.
class DepotArticlesFactice implements DepotArticles {
  final Duration delai;
  final bool doitEchouer;

  const DepotArticlesFactice({
    this.delai = const Duration(milliseconds: 600),
    this.doitEchouer = false,
  });

  static const _donnees = <Article>[
    Article(id: 1, idUtilisateur: 1, titre: 'L\'épée de Cendre',
        corps: 'Forgée dans les cendres du volcan.'),
    Article(id: 2, idUtilisateur: 1, titre: 'Le bouclier de Baltus',
        corps: 'Il a paré mille coups.'),
    Article(id: 3, idUtilisateur: 2, titre: 'La potion d\'Alia',
        corps: 'Rend vingt points de vie.'),
  ];

  @override
  Future<List<Article>> lister() async {
    await Future<void>.delayed(delai);   // on SIMULE la latence réseau
    if (doitEchouer) throw const ErreurApi('Panne simulée du serveur.');
    return _donnees;
  }

  @override
  Future<Article> obtenir(int id) async {
    await Future<void>.delayed(delai);
    if (doitEchouer) throw const ErreurApi('Panne simulée du serveur.');
    final trouve = _donnees.where((a) => a.id == id);
    if (trouve.isEmpty) throw const ErreurApi('Article introuvable.', );
    return trouve.first;
  }
}
```

### Basculer de l'une à l'autre

```dart
// Dans main.dart, un seul endroit à changer.
const bool modeHorsLigne =
    bool.fromEnvironment('HORS_LIGNE', defaultValue: false);

final DepotArticles depot = modeHorsLigne
    ? const DepotArticlesFactice()
    : DepotArticlesHttp();
```

```text
flutter run                                  # vraie API
flutter run --dart-define=HORS_LIGNE=true    # données factices
```

**Aucun écran ne change.** Ils dépendent de `DepotArticles`, pas de son implémentation. C'est la définition même d'une bonne architecture.

### Toujours simuler la latence

Un dépôt factice qui répond instantanément vous cache tous vos bugs de chargement. Vous ne verrez jamais l'indicateur, jamais le clignotement, jamais la course entre deux requêtes.

```dart
await Future<void>.delayed(const Duration(milliseconds: 600));
```

Cette ligne est **obligatoire** dans un faux dépôt. Mieux : rendez le délai variable, pour reproduire un réseau capricieux.

```dart
final _alea = Random();

Future<void> _latenceRealiste() async {
  // Entre 200 ms et 2 s, comme un vrai réseau mobile.
  await Future<void>.delayed(
    Duration(milliseconds: 200 + _alea.nextInt(1800)),
  );
  // Une requête sur dix échoue, comme dans la vraie vie.
  if (_alea.nextInt(10) == 0) {
    throw const ErreurApi('Connexion perdue.');
  }
}
```

Un faux dépôt qui échoue une fois sur dix vous force à écrire les quatre états dès le premier jour. C'est un excellent investissement.

---

## 53.39 — Tester une couche service avec un client factice (rappel chapitre 16)

Le chapitre 16 a introduit `test` et le dossier `test/`. Appliquons-le au réseau.

Un test ne doit **jamais** appeler la vraie API. Un test qui dépend d'Internet est lent, capricieux, échoue dans le train, et interdit de tester les erreurs.

La solution : remplacer le `http.Client` par un faux. Le paquet `http` fournit exactement cela dans `package:http/testing.dart` : la classe **`MockClient`**.

```text
flutter pub add dev:test
```

`MockClient` prend une fonction qui reçoit la `Request` et renvoie une `Response` de votre choix.

```dart
// test/service_articles_test.dart
import 'dart:convert';

import 'package:flutter_test/flutter_test.dart';
import 'package:http/http.dart' as http;
import 'package:http/testing.dart';

import 'package:mon_application/modeles/article.dart';
import 'package:mon_application/services/service_articles.dart';

void main() {
  group('ServiceArticles.lister', () {
    test('renvoie une liste d\'articles quand le serveur répond 200', () async {
      final client = MockClient((requete) async {
        // On peut vérifier la requête elle-même.
        expect(requete.method, 'GET');
        expect(requete.url.path, '/posts');

        return http.Response(
          jsonEncode([
            {'id': 1, 'userId': 1, 'title': 'Épée', 'body': 'Tranchante'},
            {'id': 2, 'userId': 2, 'title': 'Potion', 'body': 'Rouge'},
          ]),
          200,
          headers: {'content-type': 'application/json; charset=utf-8'},
        );
      });

      final service = ServiceArticles(client: client);
      final articles = await service.lister();

      expect(articles, hasLength(2));
      expect(articles.first.titre, 'Épée');
      expect(articles.last.idUtilisateur, 2);
    });

    test('lève une ErreurApi quand le serveur répond 500', () async {
      final client = MockClient((_) async => http.Response('Erreur', 500));
      final service = ServiceArticles(client: client);

      expect(
        () => service.lister(),
        throwsA(isA<ErreurApi>()
            .having((e) => e.codeStatut, 'codeStatut', 500)),
      );
    });

    test('lève une ErreurApi quand la réponse n\'est pas du JSON', () async {
      final client = MockClient(
        (_) async => http.Response('<html>503</html>', 200),
      );
      final service = ServiceArticles(client: client);

      expect(() => service.lister(), throwsA(isA<ErreurApi>()));
    });

    test('renvoie une liste vide quand le serveur renvoie []', () async {
      final client = MockClient((_) async => http.Response('[]', 200));
      final service = ServiceArticles(client: client);

      expect(await service.lister(), isEmpty);
    });

    test('tolère un champ manquant sans planter', () async {
      final client = MockClient(
        (_) async => http.Response(jsonEncode([
              {'id': 1}    // ni title, ni body, ni userId
            ]), 200),
      );
      final service = ServiceArticles(client: client);

      final articles = await service.lister();
      expect(articles.first.titre, 'Sans titre');
      expect(articles.first.corps, '');
    });
  });
}
```

Exécution :

```text
flutter test
```

**Résultat :**

```text
00:02 +5: All tests passed!
```

### Ce que ces tests garantissent

| Test | Ce qu'il protège |
| --- | --- |
| Réponse `200` valide | Le décodage fonctionne, les champs sont bien mappés |
| Réponse `500` | Le service lève bien une `ErreurApi`, pas une liste vide |
| Réponse non-JSON | Une `FormatException` ne remonte pas brute jusqu'à l'écran |
| Réponse `[]` | Le cas « vide » n'est pas confondu avec une erreur |
| Champ manquant | Le `fromJson` défensif fait son travail |

Ces cinq tests s'exécutent en deux secondes, sans réseau, et détecteront la plupart des régressions.

> **Pourquoi ces tests sont possibles :** parce que `ServiceArticles` accepte un `http.Client` en paramètre de constructeur (section 53.17). Si le service faisait `http.get(...)` directement, aucun test ne serait possible. **L'injection du client est ce qui rend le code testable.** C'est le principe le plus important de cette section.

### Ce qu'on ne teste pas ainsi

`MockClient` teste **votre** code, pas l'API. Il ne vous dira jamais que le serveur a renommé un champ. Pour cela, il faut soit des tests d'intégration réels, exécutés séparément, soit un contrat de schéma partagé avec l'équipe serveur.

---

## 53.40 — Mini-projet : un écran de liste chargée depuis une API

Réunissons tout : modèle défensif, couche service, timeout, traduction des erreurs, quatre états, recherche avec débounce, réessai et rafraîchissement.

### Le cahier des charges

1. Afficher la liste des produits de DummyJSON.
2. Indicateur pendant le chargement.
3. Message clair et bouton « Réessayer » en cas d'échec.
4. Message spécifique si la liste est vide.
5. Champ de recherche avec débounce de 400 ms.
6. Rafraîchissement par geste vers le bas.
7. Écran de détail au toucher d'un élément.
8. Aucune requête relancée inutilement.

### L'organisation des fichiers

En vrai projet, on découpe. Ici, tout est dans un seul `main.dart` pour rester copiable, mais les sections sont séparées par des bandeaux qui correspondent aux fichiers.

```text
lib/
├── main.dart
├── modeles/produit.dart
├── services/erreur_api.dart
├── services/service_produits.dart
├── ecrans/ecran_catalogue.dart
└── ecrans/ecran_detail.dart
```

### Le code complet

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

// ══════════════════════════ modeles/produit.dart ══════════════════════════

class Produit {
  final int id;
  final String titre;
  final String description;
  final String categorie;
  final double prix;
  final double note;
  final int stock;
  final String? vignette;

  const Produit({
    required this.id,
    required this.titre,
    required this.description,
    required this.categorie,
    required this.prix,
    required this.note,
    required this.stock,
    this.vignette,
  });

  factory Produit.fromJson(Map<String, dynamic> json) => Produit(
        id: (json['id'] as num?)?.toInt() ?? 0,
        titre: json['title'] as String? ?? 'Sans titre',
        description: json['description'] as String? ?? '',
        categorie: json['category'] as String? ?? 'divers',
        prix: (json['price'] as num?)?.toDouble() ?? 0,
        note: (json['rating'] as num?)?.toDouble() ?? 0,
        stock: (json['stock'] as num?)?.toInt() ?? 0,
        vignette: json['thumbnail'] as String?,
      );

  bool get enStock => stock > 0;
}

// ═════════════════════ services/erreur_api.dart ═══════════════════════════

class ErreurApi implements Exception {
  final String message;
  final int? codeStatut;

  const ErreurApi(this.message, {this.codeStatut});

  bool get estReseau => codeStatut == null;
  bool get estServeur => codeStatut != null && codeStatut! >= 500;
  bool get reessayable => estReseau || estServeur || codeStatut == 429;

  IconData get icone {
    if (estReseau) return Icons.wifi_off;
    if (codeStatut == 404) return Icons.search_off;
    if (estServeur) return Icons.dns_outlined;
    return Icons.error_outline;
  }

  @override
  String toString() => message;
}

// ══════════════════ services/service_produits.dart ════════════════════════

class ServiceProduits {
  final http.Client _client;
  static const String _hote = 'dummyjson.com';
  static const Duration _delai = Duration(seconds: 10);

  ServiceProduits({http.Client? client}) : _client = client ?? http.Client();

  Future<List<Produit>> lister({int limite = 30}) {
    return _appeler('/products', {'limit': '$limite'});
  }

  Future<List<Produit>> chercher(String requete, {int limite = 30}) {
    return _appeler('/products/search', {'q': requete, 'limit': '$limite'});
  }

  /// Point d'entrée unique : un seul endroit pour le timeout,
  /// la vérification du statut, le décodage et la traduction des erreurs.
  Future<List<Produit>> _appeler(
    String chemin,
    Map<String, String> parametres,
  ) async {
    final url = Uri.https(_hote, chemin, parametres);
    try {
      final reponse = await _client
          .get(url, headers: const {'Accept': 'application/json'})
          .timeout(_delai);

      if (reponse.statusCode < 200 || reponse.statusCode >= 300) {
        throw ErreurApi(
          _messagePourStatut(reponse.statusCode),
          codeStatut: reponse.statusCode,
        );
      }

      final brut = jsonDecode(utf8.decode(reponse.bodyBytes));
      if (brut is! Map<String, dynamic>) {
        throw const ErreurApi('Le serveur a renvoyé un format inattendu.');
      }

      final elements = brut['products'];
      if (elements is! List) {
        throw const ErreurApi('Le serveur n\'a pas renvoyé de produits.');
      }

      return elements
          .map((e) => Produit.fromJson(e as Map<String, dynamic>))
          .toList();
    } on TimeoutException {
      throw ErreurApi(
        'Le serveur met plus de ${_delai.inSeconds} secondes à répondre.',
      );
    } on SocketException {
      throw const ErreurApi('Aucune connexion Internet.');
    } on http.ClientException {
      throw const ErreurApi('La connexion a été interrompue.');
    } on FormatException {
      throw const ErreurApi('Le serveur a renvoyé une réponse illisible.');
    }
  }

  static String _messagePourStatut(int code) => switch (code) {
        400 => 'Requête incorrecte.',
        401 => 'Vous devez vous reconnecter.',
        403 => 'Accès refusé.',
        404 => 'Ressource introuvable.',
        429 => 'Trop de requêtes. Patientez un instant.',
        >= 500 => 'Le serveur rencontre un problème. Réessayez plus tard.',
        _ => 'Erreur inattendue ($code).',
      };

  void fermer() => _client.close();
}

// ═══════════════════════════ main.dart ════════════════════════════════════

void main() => runApp(const ApplicationCatalogue());

class ApplicationCatalogue extends StatelessWidget {
  const ApplicationCatalogue({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Catalogue',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorSchemeSeed: Colors.deepOrange,
        useMaterial3: true,
      ),
      darkTheme: ThemeData(
        colorSchemeSeed: Colors.deepOrange,
        brightness: Brightness.dark,
        useMaterial3: true,
      ),
      home: const EcranCatalogue(),
    );
  }
}

// ═════════════════════ ecrans/ecran_catalogue.dart ════════════════════════

class EcranCatalogue extends StatefulWidget {
  const EcranCatalogue({super.key});

  @override
  State<EcranCatalogue> createState() => _EcranCatalogueState();
}

class _EcranCatalogueState extends State<EcranCatalogue> {
  final ServiceProduits _service = ServiceProduits();
  final TextEditingController _ctrlRecherche = TextEditingController();

  Timer? _minuteur;
  int _numeroRequete = 0;

  List<Produit> _produits = const [];
  bool _enCours = true;
  ErreurApi? _erreur;
  String _requeteCourante = '';

  @override
  void initState() {
    super.initState();
    _charger();
  }

  @override
  void dispose() {
    _minuteur?.cancel();
    _ctrlRecherche.dispose();
    _service.fermer();
    super.dispose();
  }

  /// Charge la liste, ou lance une recherche si _requeteCourante est renseignée.
  Future<void> _charger() async {
    final monNumero = ++_numeroRequete;
    final requete = _requeteCourante;

    setState(() {
      _enCours = true;
      _erreur = null;
    });

    try {
      final resultats = requete.isEmpty
          ? await _service.lister()
          : await _service.chercher(requete);

      // Une requête plus récente est partie : on jette ce résultat.
      if (monNumero != _numeroRequete || !mounted) return;

      setState(() {
        _produits = resultats;
        _enCours = false;
      });
    } on ErreurApi catch (e) {
      if (monNumero != _numeroRequete || !mounted) return;
      setState(() {
        _erreur = e;
        _enCours = false;
      });
    } catch (e) {
      if (monNumero != _numeroRequete || !mounted) return;
      setState(() {
        _erreur = ErreurApi('Erreur inattendue : $e');
        _enCours = false;
      });
    }
  }

  void _surTexteRecherche(String texte) {
    _minuteur?.cancel();
    _minuteur = Timer(const Duration(milliseconds: 400), () {
      _requeteCourante = texte.trim();
      _charger();
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Catalogue'),
        actions: [
          IconButton(
            tooltip: 'Recharger',
            onPressed: _enCours ? null : _charger,
            icon: const Icon(Icons.refresh),
          ),
        ],
        bottom: PreferredSize(
          preferredSize: const Size.fromHeight(64),
          child: Padding(
            padding: const EdgeInsets.fromLTRB(12, 0, 12, 12),
            child: TextField(
              controller: _ctrlRecherche,
              onChanged: _surTexteRecherche,
              textInputAction: TextInputAction.search,
              decoration: InputDecoration(
                hintText: 'Rechercher un produit...',
                prefixIcon: const Icon(Icons.search),
                filled: true,
                fillColor: Theme.of(context).colorScheme.surface,
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(28),
                  borderSide: BorderSide.none,
                ),
                suffixIcon: _ctrlRecherche.text.isEmpty
                    ? null
                    : IconButton(
                        icon: const Icon(Icons.clear),
                        onPressed: () {
                          _ctrlRecherche.clear();
                          _minuteur?.cancel();
                          _requeteCourante = '';
                          _charger();
                        },
                      ),
              ),
            ),
          ),
        ),
      ),
      body: RefreshIndicator(
        onRefresh: _charger,
        child: _corps(),
      ),
    );
  }

  Widget _corps() {
    // ÉTAT 1 — chargement (uniquement si on n'a rien à montrer).
    if (_enCours && _produits.isEmpty) {
      return const Center(child: CircularProgressIndicator());
    }

    // ÉTAT 2 — erreur.
    if (_erreur != null) {
      return _VueMessage(
        icone: _erreur!.icone,
        titre: _erreur!.message,
        sousTitre: _erreur!.codeStatut == null
            ? 'Vérifiez votre connexion, puis réessayez.'
            : 'Code ${_erreur!.codeStatut}',
        libelleBouton: _erreur!.reessayable ? 'Réessayer' : 'Recharger',
        onBouton: _charger,
      );
    }

    // ÉTAT 3bis — vide.
    if (_produits.isEmpty) {
      return _VueMessage(
        icone: Icons.search_off,
        titre: _requeteCourante.isEmpty
            ? 'Le catalogue est vide.'
            : 'Aucun résultat pour « $_requeteCourante ».',
        sousTitre: 'Essayez un autre terme de recherche.',
        libelleBouton: 'Tout afficher',
        onBouton: () {
          _ctrlRecherche.clear();
          _requeteCourante = '';
          _charger();
        },
      );
    }

    // ÉTAT 3 — succès.
    return Column(
      children: [
        if (_enCours) const LinearProgressIndicator(minHeight: 2),
        Expanded(
          child: ListView.separated(
            physics: const AlwaysScrollableScrollPhysics(),
            itemCount: _produits.length,
            separatorBuilder: (_, __) => const Divider(height: 1),
            itemBuilder: (context, i) {
              final p = _produits[i];
              return ListTile(
                leading: _Vignette(url: p.vignette),
                title: Text(p.titre, maxLines: 1,
                    overflow: TextOverflow.ellipsis),
                subtitle: Text('${p.categorie} — note ${p.note}'),
                trailing: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  crossAxisAlignment: CrossAxisAlignment.end,
                  children: [
                    Text(
                      '${p.prix.toStringAsFixed(2)} EUR',
                      style: const TextStyle(fontWeight: FontWeight.bold),
                    ),
                    Text(
                      p.enStock ? 'en stock' : 'épuisé',
                      style: TextStyle(
                        fontSize: 11,
                        color: p.enStock ? Colors.green : Colors.red,
                      ),
                    ),
                  ],
                ),
                onTap: () => Navigator.of(context).push(
                  MaterialPageRoute<void>(
                    builder: (_) => EcranDetail(produit: p),
                  ),
                ),
              );
            },
          ),
        ),
      ],
    );
  }
}

class _Vignette extends StatelessWidget {
  final String? url;
  const _Vignette({required this.url});

  @override
  Widget build(BuildContext context) {
    if (url == null || url!.isEmpty) {
      return const CircleAvatar(child: Icon(Icons.inventory_2));
    }
    return CircleAvatar(
      backgroundColor: Theme.of(context).colorScheme.surfaceContainerHighest,
      child: ClipOval(
        child: Image.network(
          url!,
          width: 40,
          height: 40,
          fit: BoxFit.cover,
          // Une image réseau échoue aussi : on prévoit le repli.
          errorBuilder: (_, __, ___) => const Icon(Icons.broken_image),
          loadingBuilder: (context, enfant, progression) =>
              progression == null
                  ? enfant
                  : const SizedBox(
                      width: 20,
                      height: 20,
                      child: CircularProgressIndicator(strokeWidth: 2),
                    ),
        ),
      ),
    );
  }
}

class _VueMessage extends StatelessWidget {
  final IconData icone;
  final String titre;
  final String sousTitre;
  final String libelleBouton;
  final VoidCallback onBouton;

  const _VueMessage({
    required this.icone,
    required this.titre,
    required this.sousTitre,
    required this.libelleBouton,
    required this.onBouton,
  });

  @override
  Widget build(BuildContext context) {
    // ListView et non Column : le geste de rafraîchissement doit
    // fonctionner même sur les écrans d'erreur et de vide.
    return ListView(
      physics: const AlwaysScrollableScrollPhysics(),
      children: [
        SizedBox(
          height: MediaQuery.of(context).size.height * 0.6,
          child: Center(
            child: Padding(
              padding: const EdgeInsets.all(32),
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  Icon(icone, size: 72,
                      color: Theme.of(context).colorScheme.outline),
                  const SizedBox(height: 20),
                  Text(
                    titre,
                    textAlign: TextAlign.center,
                    style: Theme.of(context).textTheme.titleMedium,
                  ),
                  const SizedBox(height: 8),
                  Text(
                    sousTitre,
                    textAlign: TextAlign.center,
                    style: Theme.of(context).textTheme.bodySmall,
                  ),
                  const SizedBox(height: 24),
                  FilledButton.icon(
                    onPressed: onBouton,
                    icon: const Icon(Icons.refresh),
                    label: Text(libelleBouton),
                  ),
                ],
              ),
            ),
          ),
        ),
      ],
    );
  }
}

// ═══════════════════════ ecrans/ecran_detail.dart ═════════════════════════

class EcranDetail extends StatelessWidget {
  final Produit produit;
  const EcranDetail({super.key, required this.produit});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(produit.titre)),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          if (produit.vignette != null)
            ClipRRect(
              borderRadius: BorderRadius.circular(16),
              child: Image.network(
                produit.vignette!,
                height: 220,
                width: double.infinity,
                fit: BoxFit.cover,
                errorBuilder: (_, __, ___) => Container(
                  height: 220,
                  color: Theme.of(context).colorScheme.surfaceContainerHighest,
                  child: const Icon(Icons.broken_image, size: 64),
                ),
              ),
            ),
          const SizedBox(height: 20),
          Text(produit.titre,
              style: Theme.of(context).textTheme.headlineSmall),
          const SizedBox(height: 8),
          Wrap(
            spacing: 8,
            children: [
              Chip(label: Text(produit.categorie)),
              Chip(label: Text('note ${produit.note}')),
              Chip(
                label: Text(produit.enStock
                    ? '${produit.stock} en stock'
                    : 'épuisé'),
              ),
            ],
          ),
          const SizedBox(height: 16),
          Text(
            '${produit.prix.toStringAsFixed(2)} EUR',
            style: Theme.of(context).textTheme.headlineMedium?.copyWith(
                  color: Theme.of(context).colorScheme.primary,
                ),
          ),
          const SizedBox(height: 20),
          Text(produit.description,
              style: Theme.of(context).textTheme.bodyLarge),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
Ouverture      : indicateur circulaire, puis 30 produits.
Saisie « phone » : une seule requête après 400 ms de calme,
                 la liste se remplace par les produits correspondants.
Aucun résultat : icône « recherche barrée », message
                 « Aucun résultat pour « xyz ». » et bouton « Tout afficher ».
Mode avion     : icône wifi barré, « Aucune connexion Internet. »,
                 bouton « Réessayer » — qui fonctionne dès que le réseau revient.
Geste vers le bas : rechargement, avec l'animation de RefreshIndicator.
Toucher un élément : écran de détail avec image, prix et description.
```

### Ce que ce projet démontre

| Exigence | Où elle est traitée |
| --- | --- |
| Modèle défensif | `Produit.fromJson` avec `as num?` et `??` |
| Couche service isolée | `ServiceProduits`, aucun `import 'material.dart'` |
| Client injectable | `ServiceProduits({http.Client? client})` |
| Timeout | `.timeout(_delai)` dans `_appeler` |
| Traduction des erreurs | Les cinq `on ... catch` et `_messagePourStatut` |
| Quatre états | La méthode `_corps()` |
| Débounce | `_minuteur` et `_surTexteRecherche` |
| Réponses désordonnées | `_numeroRequete` |
| Fuites mémoire | `dispose()` ferme le minuteur, le contrôleur et le client |
| `setState` après démontage | Les tests `!mounted` |
| Rafraîchissement | `RefreshIndicator` + `ListView` partout |
| Image réseau défaillante | `errorBuilder` |

---

## 53.41 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| L'écran clignote et l'API est appelée en boucle | Le `Future` est créé dans `build()` : `future: _service.lister()` | Créer le `Future` dans `initState()` et passer la variable : `future: _futur` |
| `SocketException: Failed host lookup` uniquement en version publiée sur Android | La permission `INTERNET` n'est que dans le manifeste de debug | Ajouter `<uses-permission android:name="android.permission.INTERNET" />` dans `android/app/src/main/AndroidManifest.xml` |
| Rien ne se charge sur macOS | Le droit `com.apple.security.network.client` manque | L'ajouter dans `DebugProfile.entitlements` **et** `Release.entitlements` |
| `FormatException: Unexpected character (at character 1)` | `jsonDecode` appelé sur une page HTML d'erreur ou une réponse `404` | Vérifier `statusCode` **avant** de décoder ; entourer le décodage d'un `try / catch` |
| `FormatException: Unexpected end of input` | `jsonDecode` sur un corps vide, typiquement après un `DELETE` renvoyant `204` | Tester `if (reponse.body.isEmpty) return;` avant de décoder |
| `setState() called after dispose()` | Un `setState` s'exécute après un `await`, alors que l'écran est fermé | Écrire `if (!mounted) return;` juste après chaque `await`, avant le `setState` |
| `type 'int' is not a subtype of type 'double' in type cast` | Le serveur a renvoyé `10` là où on attendait `10.0` | Écrire `(json['prix'] as num).toDouble()`, jamais `as double` |
| `type 'Null' is not a subtype of type 'String'` | Un champ attendu est absent du JSON | Écrire `json['titre'] as String? ?? 'valeur par défaut'` |
| `NoSuchMethodError: '[]' called on null` | Accès en cascade `json['a']['b']` alors que `a` est absent | Créer une classe par niveau, avec `(json['a'] as Map<String, dynamic>?) ?? const {}` |
| L'écran affiche `Instance of 'Future<List<Article>>'` | Un `await` manque devant l'appel | Ajouter `await`, et déclarer la fonction `async` |
| Une erreur `500` ne déclenche aucun `catch` | `http` ne lève pas d'exception sur un statut d'erreur | Tester `statusCode >= 200 && statusCode < 300` et lever soi-même |
| La liste reste vide sans message alors que le réseau est coupé | Le `catch` renvoie `[]` au lieu de relancer une exception | Ne jamais avaler l'erreur : relancer une `ErreurApi` avec un message |
| Le chargement tourne indéfiniment | Aucun `.timeout()` : le serveur ne répond jamais et le `Future` reste en attente | Ajouter `.timeout(const Duration(seconds: 10))` sur chaque requête |
| Vingt requêtes partent pendant qu'on tape dans le champ de recherche | Pas de débounce sur `onChanged` | Utiliser un `Timer` de 400 ms, annulé à chaque frappe |
| Un ancien résultat de recherche écrase le nouveau | Les réponses arrivent dans le désordre | Utiliser un jeton de séquence : `if (monNumero != _numeroRequete) return;` |
| Deux commandes sont créées d'un seul appui | `POST` n'est pas idempotent, le bouton est resté actif | Passer `onPressed: null` pendant l'envoi |
| Le geste « tirer pour rafraîchir » ne fonctionne pas sur l'écran d'erreur | L'écran d'erreur est un `Column`, non défilable | Utiliser un `ListView` avec `physics: const AlwaysScrollableScrollPhysics()` |
| L'animation de `RefreshIndicator` disparaît aussitôt | `onRefresh` renvoie un `Future` déjà terminé | Garder la référence au `Future` et l'`await` dans `onRefresh` |
| Les accents s'affichent en `Ã©`, `Ã¨` | `response.body` décode en latin-1 | Utiliser `utf8.decode(reponse.bodyBytes)` |
| `Uri.https` refuse de compiler avec `{'limite': 20}` | Les `queryParameters` doivent être des `String` | Écrire `{'limite': '20'}` ou `'$limite'` |
| Le serveur répond `400` alors que le JSON envoyé est correct | L'en-tête `Content-Type: application/json` manque | Toujours joindre `Content-Type` à un corps JSON |
| L'application est bannie avec un `429` | Trop de requêtes : `Future` dans `build()`, ou pas de débounce | Corriger les deux causes ; respecter l'en-tête `Retry-After` |
| Le service est impossible à tester | Le service appelle `http.get()` directement | Injecter un `http.Client` dans le constructeur et utiliser `MockClient` en test |
| Les requêtes continuent après la fermeture de l'écran | Le `http.Client` et le `Timer` ne sont pas libérés | Appeler `client.close()` et `minuteur.cancel()` dans `dispose()` |
| La clé d'API apparaît dans l'historique Git | Elle a été écrite en dur dans un fichier `.dart` | Utiliser `--dart-define`, ajouter les fichiers de configuration au `.gitignore`, et **révoquer la clé** |
| L'utilisateur est déconnecté quand il ouvre une page réservée | Un `403` est traité comme un `401` | Ne déconnecter que sur `401` ; sur `403`, afficher « accès refusé » |
| La pagination remonte en haut de la liste à chaque page | La liste est remplacée au lieu d'être complétée | Utiliser `_produits.addAll(nouveaux)`, jamais `_produits = nouveaux` |
| La pagination boucle à l'infini au bas de la liste | Aucun drapeau « tout chargé » | Tenir `_toutCharge` à jour, et sortir immédiatement s'il est vrai |

---

## 53.42 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| Client / serveur | Le serveur ne parle jamais en premier ; c'est le client qui demande |
| `Uri` | Toujours construire les URL avec `Uri.https` ou `Uri.parse`, jamais par concaténation |
| API REST | L'URL désigne la ressource, le verbe désigne l'action |
| `GET` | Lire. Sûr, idempotent, sans corps. Réessayable sans risque |
| `POST` | Créer. **Non idempotent** : désactiver le bouton pendant l'envoi |
| `PUT` / `PATCH` | Remplacer entièrement / modifier partiellement |
| `DELETE` | Supprimer. Répond souvent `204`, sans corps : ne rien décoder |
| Succès HTTP | `statusCode >= 200 && statusCode < 300`, pas seulement `== 200` |
| `4xx` / `5xx` | `4xx` = votre requête est fautive ; `5xx` = le serveur est en panne |
| En-têtes | `Accept` pour recevoir du JSON, `Content-Type` pour en envoyer |
| Corps JSON | `jsonEncode()` **et** `Content-Type: application/json`, toujours ensemble |
| Package `http` | `flutter pub add http`, puis `import 'package:http/http.dart' as http;` |
| Permission Android | `INTERNET` dans le manifeste `main`, pas seulement celui de debug |
| Entitlements macOS | `com.apple.security.network.client` dans `DebugProfile` **et** `Release` |
| `http.get()` | Prend un `Uri`, renvoie un `Future<http.Response>` |
| `response.body` | C'est une `String`. Il faut la décoder |
| `utf8.decode(bodyBytes)` | La parade aux accents cassés |
| `jsonDecode` | Renvoie `Map<String, dynamic>` si le JSON commence par `{`, `List<dynamic>` s'il commence par `[` |
| Nombres JSON | `(json['x'] as num).toDouble()`, jamais `as double` |
| `fromJson()` | Une classe par niveau d'imbrication ; défensif : `as T? ?? défaut` |
| Liste d'objets | `(jsonDecode(body) as List).map((e) => T.fromJson(e as Map<String, dynamic>)).toList()` |
| Couche service | Le réseau et le JSON sont enfermés dans `lib/services/`. L'interface n'en sait rien |
| Client injecté | `Service({http.Client? client})` : c'est ce qui rend le code testable |
| Statut d'erreur | `http` ne lève **aucune** exception sur un `404` ou un `500` |
| `SocketException` | Réseau coupé, hôte introuvable. Vient de `dart:io`, absent du web |
| `ClientException` | L'équivalent portable, y compris sur le web |
| `TimeoutException` | N'arrive que si vous avez posé un `.timeout()` |
| `FormatException` | JSON invalide ou corps vide |
| `.timeout()` | Obligatoire sur chaque requête. Sans lui, l'attente est infinie |
| Quatre états | Chargement, erreur, vide, succès. Les quatre, toujours |
| `FutureBuilder` | Surveille un `Future` et reconstruit à chaque changement |
| `AsyncSnapshot` | `connectionState`, `hasData`, `hasError`, `data`, `error` |
| `ConnectionState` | Avec un `Future` : `none`, `waiting`, `done`. Jamais `active` |
| Ordre dans le `builder` | Chargement, **puis erreur**, puis absence de données, puis succès |
| Le piège de `build()` | `future: service.appel()` relance la requête à chaque image |
| `initState()` | L'endroit où créer le `Future`, une seule fois |
| `didUpdateWidget()` | Recréer le `Future` quand un paramètre du widget change |
| Bouton « réessayer » | `setState(() => _futur = service.appel())` : un **nouveau** `Future` |
| `mounted` | Test obligatoire après chaque `await`, avant chaque `setState` |
| `RefreshIndicator` | `onRefresh` doit renvoyer un `Future` qui dure la requête ; l'enfant doit défiler |
| `FutureBuilder` ou état manuel | Données figées → `FutureBuilder`. Données modifiables → état manuel |
| État scellé | `sealed class` + `switch` exhaustif : les états incohérents deviennent impossibles |
| `Authorization` | `'Bearer $jeton'`, ajouté par une collection `if` dans la `Map` d'en-têtes |
| `401` / `403` | `401` = se reconnecter. `403` = pas les droits, **ne pas** déconnecter |
| Clé d'API | Jamais dans le code source. `--dart-define`, ou serveur intermédiaire |
| `StreamBuilder` | Même forme que `FutureBuilder`, mais pour plusieurs valeurs. Passe par `active` |
| Pagination | `addAll`, jamais de remplacement ; drapeaux `_enCours` et `_toutCharge` |
| Débounce | `Timer` de 400 ms annulé à chaque frappe |
| Jeton de séquence | La parade aux réponses qui arrivent dans le désordre |
| Annulation | `client.close()` avec `http`, `CancelToken` avec `dio` |
| `dio` | Décodage automatique, intercepteurs, `CancelToken`, timeouts intégrés |
| Dépôt factice | Une interface, deux implémentations. Simuler la latence est obligatoire |
| `MockClient` | `package:http/testing.dart` : tester le service sans réseau |
| `dispose()` | Fermer le `Client`, annuler les `Timer`, libérer les contrôleurs |

---

## 53.43 — Exercices

Toutes les API utilisées sont publiques et gratuites, sans inscription :
`https://jsonplaceholder.typicode.com` et `https://dummyjson.com`.
Pensez à `flutter pub add http` et aux permissions de la section 53.10.

### Exercice 1 — Le premier appel (facile)

Écrivez une application avec un bouton « Charger » et une zone de texte.
Au toucher, appelez `https://jsonplaceholder.typicode.com/todos/1` et affichez
le code de statut ainsi que le corps brut de la réponse.
Le bouton doit être désactivé pendant la requête.

### Exercice 2 — Un modèle et un objet (facile)

Créez une classe `Tache` avec `id`, `titre` (`title`), `terminee` (`completed`)
et un `factory Tache.fromJson`.
Chargez `https://jsonplaceholder.typicode.com/todos/1` et affichez le titre
dans un `Text` et l'état dans un `Icon` (coche verte ou croix rouge).
Le `fromJson` doit tolérer l'absence de chaque champ.

### Exercice 3 — Une liste avec `FutureBuilder` (facile)

Chargez les 20 premières tâches (`/todos?_limit=20`) et affichez-les dans un
`ListView.builder` avec un `FutureBuilder`.
Traitez les trois états : indicateur, message d'erreur, liste.
Le `Future` doit être créé dans `initState()`.

### Exercice 4 — Les quatre états et le réessai (moyen)

Reprenez l'exercice 3. Ajoutez :
- une gestion d'erreur avec `.timeout(Duration(seconds: 10))` et des messages
  distincts pour l'absence de réseau, le délai dépassé et un statut d'erreur ;
- un bouton « Réessayer » sur l'écran d'erreur ;
- un écran « Aucune tâche » si la liste est vide.
Testez en activant le mode avion.

### Exercice 5 — Une couche service testable (moyen)

Extrayez le réseau de l'exercice 4 dans une classe `ServiceTaches` qui :
- accepte un `http.Client` optionnel dans son constructeur ;
- expose `Future<List<Tache>> lister({int limite})` ;
- lève une `ErreurApi` avec un `message` et un `codeStatut` facultatif ;
- possède une méthode `fermer()` appelée dans `dispose()`.
L'écran ne doit contenir **aucun** `import 'package:http/http.dart'`.

### Exercice 6 — Recherche filtrée côté serveur (moyen)

Ajoutez un `DropdownButton` listant les utilisateurs 1 à 10.
Au changement, rechargez les tâches de l'utilisateur choisi avec
`/todos?userId=N`.
Utilisez un jeton de séquence pour ignorer les réponses obsolètes,
et affichez le nombre de tâches terminées sur le total.

### Exercice 7 — Rafraîchissement et détail (moyen)

Ajoutez à l'exercice 6 :
- un `RefreshIndicator` fonctionnel, y compris sur l'écran d'erreur ;
- au toucher d'une tâche, un écran de détail qui charge
  `/users/{userId}` et affiche le nom, le courriel et la ville du propriétaire.
L'écran de détail doit lui aussi gérer ses trois états.

### Exercice 8 — Création par `POST` (difficile)

Ajoutez un bouton flottant qui ouvre un formulaire (titre + utilisateur).
À la validation, envoyez un `POST` sur `/todos` avec un corps JSON et
l'en-tête `Content-Type` correct.
- Le bouton d'envoi est désactivé pendant la requête.
- Un `201` affiche une `SnackBar` avec l'identifiant attribué.
- La nouvelle tâche est ajoutée en tête de la liste locale, sans tout recharger.

### Exercice 9 — Recherche avec débounce et pagination (difficile)

Sur `https://dummyjson.com/products` :
- un champ de recherche avec débounce de 400 ms appelant `/products/search?q=` ;
- une pagination par `limit=20&skip=N` déclenchée à 80 % du défilement ;
- un pied de liste qui affiche l'indicateur, l'erreur avec réessai, ou
  « Fin de la liste » ;
- le compteur « chargés / total » dans la barre de titre.
La recherche et la pagination doivent cohabiter : chercher réinitialise la pagination.

### Exercice 10 — Dépôt factice et tests (difficile)

Reprenez l'exercice 5 et :
1. définissez `abstract class DepotTaches` avec `lister()` et `obtenir(int id)` ;
2. écrivez `DepotTachesHttp` et `DepotTachesFactice` (latence aléatoire de
   200 à 1500 ms, échec une fois sur cinq) ;
3. choisissez l'implémentation avec
   `bool.fromEnvironment('HORS_LIGNE')` ;
4. écrivez cinq tests avec `MockClient` : réponse `200` valide, réponse `500`,
   corps non-JSON, tableau vide, champ manquant.

---

## 53.44 — Corrections des exercices

### Correction 1 — Le premier appel

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const App1());

class App1 extends StatelessWidget {
  const App1({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
      home: const Page1(),
    );
  }
}

class Page1 extends StatefulWidget {
  const Page1({super.key});

  @override
  State<Page1> createState() => _Page1State();
}

class _Page1State extends State<Page1> {
  bool _enCours = false;
  String _sortie = 'Appuyez sur « Charger ».';

  Future<void> _charger() async {
    setState(() {
      _enCours = true;
      _sortie = 'Requête en cours...';
    });

    try {
      final reponse = await http
          .get(Uri.https('jsonplaceholder.typicode.com', '/todos/1'))
          .timeout(const Duration(seconds: 10));

      if (!mounted) return;
      setState(() {
        _sortie = 'statusCode : ${reponse.statusCode}\n'
            'reasonPhrase : ${reponse.reasonPhrase}\n\n'
            '${reponse.body}';
        _enCours = false;
      });
    } catch (e) {
      if (!mounted) return;
      setState(() {
        _sortie = 'Échec : $e';
        _enCours = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Exercice 1')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            FilledButton.icon(
              onPressed: _enCours ? null : _charger,
              icon: const Icon(Icons.cloud_download),
              label: Text(_enCours ? 'Chargement...' : 'Charger'),
            ),
            const SizedBox(height: 24),
            Expanded(
              child: SingleChildScrollView(
                child: Text(
                  _sortie,
                  style: const TextStyle(fontFamily: 'monospace'),
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

**Résultat :**

```text
statusCode : 200
reasonPhrase : OK

{
  "userId": 1,
  "id": 1,
  "title": "delectus aut autem",
  "completed": false
}
```

**Explication :** on construit l'URL avec `Uri.https` plutôt qu'en concaténant une chaîne. Le `.timeout(...)` empêche une attente infinie. Le `try / catch` traite les échecs de la couche 1 (réseau) ; le `statusCode` est simplement affiché ici, puisque l'exercice demande de l'inspecter. Le `if (!mounted) return;` après chaque `await` empêche le `setState() called after dispose()` si l'utilisateur quitte l'écran pendant la requête. Enfin, `onPressed: _enCours ? null : _charger` désactive le bouton : `null` sur `onPressed` grise le bouton et bloque les appuis multiples.

---

### Correction 2 — Un modèle et un objet

```dart
import 'dart:convert';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

class Tache {
  final int id;
  final int idUtilisateur;
  final String titre;
  final bool terminee;

  const Tache({
    required this.id,
    required this.idUtilisateur,
    required this.titre,
    required this.terminee,
  });

  /// Défensif : chaque champ a une valeur de repli.
  factory Tache.fromJson(Map<String, dynamic> json) => Tache(
        id: (json['id'] as num?)?.toInt() ?? 0,
        idUtilisateur: (json['userId'] as num?)?.toInt() ?? 0,
        titre: json['title'] as String? ?? 'Sans titre',
        terminee: json['completed'] as bool? ?? false,
      );
}

void main() => runApp(const App2());

class App2 extends StatelessWidget {
  const App2({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.teal, useMaterial3: true),
      home: const Page2(),
    );
  }
}

class Page2 extends StatefulWidget {
  const Page2({super.key});

  @override
  State<Page2> createState() => _Page2State();
}

class _Page2State extends State<Page2> {
  Tache? _tache;
  String? _erreur;
  bool _enCours = false;

  Future<void> _charger() async {
    setState(() {
      _enCours = true;
      _erreur = null;
    });

    try {
      final r = await http
          .get(Uri.https('jsonplaceholder.typicode.com', '/todos/1'))
          .timeout(const Duration(seconds: 10));

      if (r.statusCode < 200 || r.statusCode >= 300) {
        throw Exception('Le serveur a répondu ${r.statusCode}.');
      }

      final json = jsonDecode(utf8.decode(r.bodyBytes));
      if (json is! Map<String, dynamic>) {
        throw Exception('Format inattendu.');
      }

      if (!mounted) return;
      setState(() {
        _tache = Tache.fromJson(json);
        _enCours = false;
      });
    } catch (e) {
      if (!mounted) return;
      setState(() {
        _erreur = '$e';
        _enCours = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Exercice 2')),
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(24),
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              if (_enCours)
                const CircularProgressIndicator()
              else if (_erreur != null)
                Text(_erreur!, textAlign: TextAlign.center)
              else if (_tache != null) ...[
                Icon(
                  _tache!.terminee ? Icons.check_circle : Icons.cancel,
                  size: 72,
                  color: _tache!.terminee ? Colors.green : Colors.red,
                ),
                const SizedBox(height: 16),
                Text(
                  _tache!.titre,
                  textAlign: TextAlign.center,
                  style: Theme.of(context).textTheme.titleLarge,
                ),
                const SizedBox(height: 8),
                Text('Tâche ${_tache!.id} '
                    '— utilisateur ${_tache!.idUtilisateur}'),
              ] else
                const Text('Aucune donnée. Appuyez sur « Charger ».'),
              const SizedBox(height: 32),
              FilledButton(
                onPressed: _enCours ? null : _charger,
                child: const Text('Charger'),
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
Icône : croix rouge (la tâche 1 n'est pas terminée)
Titre : delectus aut autem
Sous-titre : Tâche 1 — utilisateur 1
```

**Explication :** le `fromJson` n'utilise que des casts nullables suivis de `??`. Un JSON incomplet produit une `Tache` dégradée plutôt qu'un plantage. `(json['id'] as num?)?.toInt()` accepte aussi bien `1` que `1.0`. On vérifie le `statusCode` **avant** de décoder, sinon un `404` renverrait `{}` et le titre vaudrait « Sans titre » sans qu'on comprenne pourquoi. `utf8.decode(r.bodyBytes)` garantit que les accents s'affichent correctement. Enfin, le `is! Map<String, dynamic>` protège contre une réponse dont la forme aurait changé.

---

### Correction 3 — Une liste avec `FutureBuilder`

```dart
import 'dart:convert';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

class Tache {
  final int id;
  final String titre;
  final bool terminee;

  const Tache({required this.id, required this.titre, required this.terminee});

  factory Tache.fromJson(Map<String, dynamic> j) => Tache(
        id: (j['id'] as num?)?.toInt() ?? 0,
        titre: j['title'] as String? ?? 'Sans titre',
        terminee: j['completed'] as bool? ?? false,
      );
}

Future<List<Tache>> chargerTaches() async {
  final r = await http
      .get(Uri.https('jsonplaceholder.typicode.com', '/todos', {'_limit': '20'}))
      .timeout(const Duration(seconds: 10));

  if (r.statusCode < 200 || r.statusCode >= 300) {
    throw Exception('Le serveur a répondu ${r.statusCode}.');
  }

  final brut = jsonDecode(utf8.decode(r.bodyBytes));
  if (brut is! List) throw Exception('Un tableau était attendu.');

  return brut.map((e) => Tache.fromJson(e as Map<String, dynamic>)).toList();
}

void main() => runApp(const App3());

class App3 extends StatelessWidget {
  const App3({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.blue, useMaterial3: true),
      home: const Page3(),
    );
  }
}

class Page3 extends StatefulWidget {
  const Page3({super.key});

  @override
  State<Page3> createState() => _Page3State();
}

class _Page3State extends State<Page3> {
  // Le Future est un CHAMP, créé une seule fois dans initState().
  late Future<List<Tache>> _futur;

  @override
  void initState() {
    super.initState();
    _futur = chargerTaches();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Exercice 3')),
      body: FutureBuilder<List<Tache>>(
        future: _futur,
        builder: (context, snapshot) {
          // 1. Chargement.
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }
          // 2. Erreur — TOUJOURS avant les données.
          if (snapshot.hasError) {
            return Center(
              child: Padding(
                padding: const EdgeInsets.all(32),
                child: Text(
                  'Erreur : ${snapshot.error}',
                  textAlign: TextAlign.center,
                ),
              ),
            );
          }
          // 3. Succès.
          final taches = snapshot.data ?? const <Tache>[];
          return ListView.builder(
            itemCount: taches.length,
            itemBuilder: (context, i) {
              final t = taches[i];
              return ListTile(
                leading: Icon(
                  t.terminee ? Icons.check_circle : Icons.radio_button_unchecked,
                  color: t.terminee ? Colors.green : null,
                ),
                title: Text(
                  t.titre,
                  style: TextStyle(
                    decoration: t.terminee ? TextDecoration.lineThrough : null,
                  ),
                ),
                trailing: Text('#${t.id}'),
              );
            },
          );
        },
      ),
    );
  }
}
```

**Résultat :**

```text
Indicateur circulaire pendant environ 500 ms,
puis 20 lignes :
  ○  delectus aut autem                              #1
  ○  quis ut nam facilis et officia qui              #2
  ...
  ✓  et porro tempora                                #4   (texte barré)
```

**Explication :** le point capital est `late Future<List<Tache>> _futur;` affecté dans `initState()`. Si l'on avait écrit `future: chargerTaches()` dans `build()`, une nouvelle requête partirait à chaque reconstruction (rotation de l'écran, ouverture du clavier, changement de thème). L'ordre des tests dans le `builder` est également imposé : `waiting` d'abord, puis `hasError`, puis les données. Tester `hasData` avant `hasError` laisserait tourner l'indicateur indéfiniment en cas d'échec, car `data` reste `null` sur erreur.

---

### Correction 4 — Les quatre états et le réessai

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

class Tache {
  final int id;
  final String titre;
  final bool terminee;

  const Tache({required this.id, required this.titre, required this.terminee});

  factory Tache.fromJson(Map<String, dynamic> j) => Tache(
        id: (j['id'] as num?)?.toInt() ?? 0,
        titre: j['title'] as String? ?? 'Sans titre',
        terminee: j['completed'] as bool? ?? false,
      );
}

class ErreurApi implements Exception {
  final String message;
  final int? codeStatut;
  const ErreurApi(this.message, {this.codeStatut});

  bool get reessayable =>
      codeStatut == null || codeStatut! >= 500 || codeStatut == 429;

  IconData get icone {
    if (codeStatut == null) return Icons.wifi_off;
    if (codeStatut == 404) return Icons.search_off;
    if (codeStatut! >= 500) return Icons.dns_outlined;
    return Icons.error_outline;
  }

  @override
  String toString() => message;
}

Future<List<Tache>> chargerTaches({int limite = 20}) async {
  try {
    final r = await http
        .get(Uri.https(
            'jsonplaceholder.typicode.com', '/todos', {'_limit': '$limite'}))
        .timeout(const Duration(seconds: 10));

    if (r.statusCode < 200 || r.statusCode >= 300) {
      throw ErreurApi(
        r.statusCode >= 500
            ? 'Le serveur rencontre un problème. Réessayez plus tard.'
            : 'Le serveur a refusé la requête (${r.statusCode}).',
        codeStatut: r.statusCode,
      );
    }

    final brut = jsonDecode(utf8.decode(r.bodyBytes));
    if (brut is! List) throw const ErreurApi('Format inattendu.');
    return brut.map((e) => Tache.fromJson(e as Map<String, dynamic>)).toList();
  } on TimeoutException {
    throw const ErreurApi('Le serveur met plus de 10 secondes à répondre.');
  } on SocketException {
    throw const ErreurApi('Aucune connexion Internet.');
  } on http.ClientException {
    throw const ErreurApi('La connexion a été interrompue.');
  } on FormatException {
    throw const ErreurApi('Le serveur a renvoyé une réponse illisible.');
  }
}

void main() => runApp(const App4());

class App4 extends StatelessWidget {
  const App4({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.green, useMaterial3: true),
      home: const Page4(),
    );
  }
}

class Page4 extends StatefulWidget {
  const Page4({super.key});

  @override
  State<Page4> createState() => _Page4State();
}

class _Page4State extends State<Page4> {
  late Future<List<Tache>> _futur;

  @override
  void initState() {
    super.initState();
    _futur = chargerTaches();
  }

  void _reessayer() {
    // Un NOUVEAU Future : c'est ce que FutureBuilder détecte.
    setState(() => _futur = chargerTaches());
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Exercice 4')),
      body: FutureBuilder<List<Tache>>(
        future: _futur,
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }

          if (snapshot.hasError) {
            final e = snapshot.error;
            final erreur =
                e is ErreurApi ? e : ErreurApi('Erreur inattendue : $e');
            return _Message(
              icone: erreur.icone,
              titre: erreur.message,
              libelle: erreur.reessayable ? 'Réessayer' : 'Recharger',
              onAction: _reessayer,
            );
          }

          final taches = snapshot.data ?? const <Tache>[];

          if (taches.isEmpty) {
            return _Message(
              icone: Icons.inbox_outlined,
              titre: 'Aucune tâche pour le moment.',
              libelle: 'Recharger',
              onAction: _reessayer,
            );
          }

          return ListView.separated(
            itemCount: taches.length,
            separatorBuilder: (_, __) => const Divider(height: 1),
            itemBuilder: (context, i) => ListTile(
              leading: Icon(
                taches[i].terminee
                    ? Icons.check_circle
                    : Icons.radio_button_unchecked,
                color: taches[i].terminee ? Colors.green : null,
              ),
              title: Text(taches[i].titre),
              trailing: Text('#${taches[i].id}'),
            ),
          );
        },
      ),
    );
  }
}

class _Message extends StatelessWidget {
  final IconData icone;
  final String titre;
  final String libelle;
  final VoidCallback onAction;

  const _Message({
    required this.icone,
    required this.titre,
    required this.libelle,
    required this.onAction,
  });

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Padding(
        padding: const EdgeInsets.all(32),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Icon(icone, size: 72, color: Theme.of(context).colorScheme.outline),
            const SizedBox(height: 20),
            Text(
              titre,
              textAlign: TextAlign.center,
              style: Theme.of(context).textTheme.titleMedium,
            ),
            const SizedBox(height: 24),
            FilledButton.icon(
              onPressed: onAction,
              icon: const Icon(Icons.refresh),
              label: Text(libelle),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat (mode avion) :**

```text
Indicateur, puis :
      [icône wifi barré]
   Aucune connexion Internet.
      [ Réessayer ]

Après désactivation du mode avion et appui sur « Réessayer » :
   indicateur, puis les 20 tâches.
```

**Explication :** les quatre `on ... catch` traduisent chaque exception technique en un message compréhensible. Aucune trace de `SocketException` n'arrive à l'écran. La classe `ErreurApi` porte un `codeStatut` facultatif, ce qui permet de savoir si un réessai a une chance d'aboutir : sur un `404`, le bouton change de libellé plutôt que de promettre l'impossible. Le réessai fonctionne parce que `_reessayer()` **réaffecte** `_futur` à l'intérieur d'un `setState` : `FutureBuilder` compare les `Future` par identité et repart en `waiting` en voyant un objet différent.

---

### Correction 5 — Une couche service testable

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

// ─────────────────────────── modele ───────────────────────────

class Tache {
  final int id;
  final int idUtilisateur;
  final String titre;
  final bool terminee;

  const Tache({
    required this.id,
    required this.idUtilisateur,
    required this.titre,
    required this.terminee,
  });

  factory Tache.fromJson(Map<String, dynamic> j) => Tache(
        id: (j['id'] as num?)?.toInt() ?? 0,
        idUtilisateur: (j['userId'] as num?)?.toInt() ?? 0,
        titre: j['title'] as String? ?? 'Sans titre',
        terminee: j['completed'] as bool? ?? false,
      );
}

// ────────────────────────── exception ─────────────────────────

class ErreurApi implements Exception {
  final String message;
  final int? codeStatut;
  const ErreurApi(this.message, {this.codeStatut});

  bool get reessayable =>
      codeStatut == null || codeStatut! >= 500 || codeStatut == 429;

  @override
  String toString() => message;
}

// ─────────────────────────── service ──────────────────────────

class ServiceTaches {
  final http.Client _client;
  final String _hote;
  final Duration _delai;

  /// Le client est INJECTÉ : c'est ce qui rend la classe testable.
  ServiceTaches({
    http.Client? client,
    String hote = 'jsonplaceholder.typicode.com',
    Duration delai = const Duration(seconds: 10),
  })  : _client = client ?? http.Client(),
        _hote = hote,
        _delai = delai;

  Future<List<Tache>> lister({int limite = 20, int? idUtilisateur}) async {
    final parametres = <String, String>{
      '_limit': '$limite',
      if (idUtilisateur != null) 'userId': '$idUtilisateur',
    };

    try {
      final r = await _client
          .get(Uri.https(_hote, '/todos', parametres),
              headers: const {'Accept': 'application/json'})
          .timeout(_delai);

      if (r.statusCode < 200 || r.statusCode >= 300) {
        throw ErreurApi(_messagePourStatut(r.statusCode),
            codeStatut: r.statusCode);
      }

      final brut = jsonDecode(utf8.decode(r.bodyBytes));
      if (brut is! List) {
        throw const ErreurApi('Le serveur n\'a pas renvoyé de tableau.');
      }
      return brut.map((e) => Tache.fromJson(e as Map<String, dynamic>)).toList();
    } on TimeoutException {
      throw ErreurApi('Le serveur met plus de ${_delai.inSeconds} s à répondre.');
    } on SocketException {
      throw const ErreurApi('Aucune connexion Internet.');
    } on http.ClientException {
      throw const ErreurApi('La connexion a été interrompue.');
    } on FormatException {
      throw const ErreurApi('Le serveur a renvoyé une réponse illisible.');
    }
  }

  static String _messagePourStatut(int c) => switch (c) {
        404 => 'Ressource introuvable.',
        429 => 'Trop de requêtes. Patientez.',
        >= 500 => 'Le serveur rencontre un problème.',
        _ => 'Erreur inattendue ($c).',
      };

  void fermer() => _client.close();
}

// ──────────────────────────── écran ───────────────────────────
// Aucun import de package:http ici : l'interface ignore le réseau.

void main() => runApp(const App5());

class App5 extends StatelessWidget {
  const App5({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.purple, useMaterial3: true),
      home: const Page5(),
    );
  }
}

class Page5 extends StatefulWidget {
  const Page5({super.key});

  @override
  State<Page5> createState() => _Page5State();
}

class _Page5State extends State<Page5> {
  final ServiceTaches _service = ServiceTaches();
  late Future<List<Tache>> _futur;

  @override
  void initState() {
    super.initState();
    _futur = _service.lister();
  }

  @override
  void dispose() {
    _service.fermer();
    super.dispose();
  }

  void _recharger() => setState(() => _futur = _service.lister());

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Exercice 5'),
        actions: [
          IconButton(onPressed: _recharger, icon: const Icon(Icons.refresh)),
        ],
      ),
      body: FutureBuilder<List<Tache>>(
        future: _futur,
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }
          if (snapshot.hasError) {
            final e = snapshot.error;
            final reessayable = e is ErreurApi ? e.reessayable : true;
            return Center(
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  const Icon(Icons.cloud_off, size: 64),
                  const SizedBox(height: 16),
                  Text('$e', textAlign: TextAlign.center),
                  const SizedBox(height: 20),
                  FilledButton(
                    onPressed: _recharger,
                    child: Text(reessayable ? 'Réessayer' : 'Recharger'),
                  ),
                ],
              ),
            );
          }
          final taches = snapshot.data ?? const <Tache>[];
          if (taches.isEmpty) {
            return const Center(child: Text('Aucune tâche.'));
          }
          return ListView.builder(
            itemCount: taches.length,
            itemBuilder: (context, i) => ListTile(
              leading: Icon(
                taches[i].terminee
                    ? Icons.check_circle
                    : Icons.radio_button_unchecked,
                color: taches[i].terminee ? Colors.green : null,
              ),
              title: Text(taches[i].titre),
              subtitle: Text('utilisateur ${taches[i].idUtilisateur}'),
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
Indicateur, puis 20 tâches avec leur propriétaire.
Le bouton de la barre recharge la liste.
```

**Explication :** trois éléments rendent ce service correct. **Premièrement**, le client est un paramètre optionnel du constructeur : en production il vaut `http.Client()`, en test on injecte un `MockClient` (correction 10). **Deuxièmement**, le service ne renvoie que des types métier (`List<Tache>`) et ne lève que des `ErreurApi` : rien de HTTP ne fuit vers l'écran. **Troisièmement**, `fermer()` est appelé dans `dispose()`, ce qui coupe les requêtes en cours et libère les connexions persistantes. Notez aussi la collection `if` dans la construction des paramètres : `userId` n'est ajouté que s'il est fourni, sans construire la `Map` en deux temps.

---

### Correction 6 — Recherche filtrée côté serveur

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

class Tache {
  final int id;
  final String titre;
  final bool terminee;

  const Tache({required this.id, required this.titre, required this.terminee});

  factory Tache.fromJson(Map<String, dynamic> j) => Tache(
        id: (j['id'] as num?)?.toInt() ?? 0,
        titre: j['title'] as String? ?? 'Sans titre',
        terminee: j['completed'] as bool? ?? false,
      );
}

class ErreurApi implements Exception {
  final String message;
  final int? codeStatut;
  const ErreurApi(this.message, {this.codeStatut});
  @override
  String toString() => message;
}

class ServiceTaches {
  final http.Client _client;
  ServiceTaches({http.Client? client}) : _client = client ?? http.Client();

  Future<List<Tache>> listerPourUtilisateur(int idUtilisateur) async {
    try {
      final r = await _client
          .get(Uri.https('jsonplaceholder.typicode.com', '/todos',
              {'userId': '$idUtilisateur'}))
          .timeout(const Duration(seconds: 10));

      if (r.statusCode < 200 || r.statusCode >= 300) {
        throw ErreurApi('Erreur ${r.statusCode}.', codeStatut: r.statusCode);
      }
      final brut = jsonDecode(utf8.decode(r.bodyBytes));
      if (brut is! List) throw const ErreurApi('Format inattendu.');
      return brut.map((e) => Tache.fromJson(e as Map<String, dynamic>)).toList();
    } on TimeoutException {
      throw const ErreurApi('Délai dépassé.');
    } on SocketException {
      throw const ErreurApi('Aucune connexion Internet.');
    } on http.ClientException {
      throw const ErreurApi('Connexion interrompue.');
    } on FormatException {
      throw const ErreurApi('Réponse illisible.');
    }
  }

  void fermer() => _client.close();
}

void main() => runApp(const App6());

class App6 extends StatelessWidget {
  const App6({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.orange, useMaterial3: true),
      home: const Page6(),
    );
  }
}

class Page6 extends StatefulWidget {
  const Page6({super.key});

  @override
  State<Page6> createState() => _Page6State();
}

class _Page6State extends State<Page6> {
  final ServiceTaches _service = ServiceTaches();

  int _idUtilisateur = 1;
  int _numeroRequete = 0;

  List<Tache> _taches = const [];
  bool _enCours = true;
  String? _erreur;

  @override
  void initState() {
    super.initState();
    _charger();
  }

  @override
  void dispose() {
    _service.fermer();
    super.dispose();
  }

  Future<void> _charger() async {
    // Jeton de séquence : chaque appel reçoit un numéro croissant.
    final monNumero = ++_numeroRequete;
    final idDemande = _idUtilisateur;

    setState(() {
      _enCours = true;
      _erreur = null;
    });

    try {
      final resultats = await _service.listerPourUtilisateur(idDemande);
      // Une requête plus récente est partie : ce résultat est périmé.
      if (monNumero != _numeroRequete || !mounted) return;
      setState(() {
        _taches = resultats;
        _enCours = false;
      });
    } catch (e) {
      if (monNumero != _numeroRequete || !mounted) return;
      setState(() {
        _erreur = '$e';
        _enCours = false;
      });
    }
  }

  int get _terminees => _taches.where((t) => t.terminee).length;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Exercice 6')),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16),
            child: Row(
              children: [
                const Text('Utilisateur : '),
                const SizedBox(width: 12),
                DropdownButton<int>(
                  value: _idUtilisateur,
                  items: [
                    for (int i = 1; i <= 10; i++)
                      DropdownMenuItem(value: i, child: Text('$i')),
                  ],
                  onChanged: _enCours
                      ? null
                      : (v) {
                          if (v == null) return;
                          setState(() => _idUtilisateur = v);
                          _charger();
                        },
                ),
                const Spacer(),
                if (!_enCours && _erreur == null)
                  Text('$_terminees / ${_taches.length} terminées'),
              ],
            ),
          ),
          if (_enCours) const LinearProgressIndicator(minHeight: 2),
          const Divider(height: 1),
          Expanded(child: _corps()),
        ],
      ),
    );
  }

  Widget _corps() {
    if (_erreur != null) {
      return Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            const Icon(Icons.cloud_off, size: 56),
            const SizedBox(height: 12),
            Text(_erreur!),
            const SizedBox(height: 16),
            FilledButton(onPressed: _charger, child: const Text('Réessayer')),
          ],
        ),
      );
    }
    if (_enCours && _taches.isEmpty) {
      return const Center(child: CircularProgressIndicator());
    }
    if (_taches.isEmpty) {
      return const Center(child: Text('Aucune tâche pour cet utilisateur.'));
    }
    return ListView.separated(
      itemCount: _taches.length,
      separatorBuilder: (_, __) => const Divider(height: 1),
      itemBuilder: (context, i) => ListTile(
        leading: Icon(
          _taches[i].terminee
              ? Icons.check_circle
              : Icons.radio_button_unchecked,
          color: _taches[i].terminee ? Colors.green : null,
        ),
        title: Text(_taches[i].titre),
        trailing: Text('#${_taches[i].id}'),
      ),
    );
  }
}
```

**Résultat :**

```text
Utilisateur : 1     11 / 20 terminées
  ○  delectus aut autem                    #1
  ✓  et porro tempora                      #4
  ...

Changement vers l'utilisateur 5 :
  barre de progression, puis « 12 / 20 terminées » et la nouvelle liste.
```

**Explication :** on est passé du `FutureBuilder` à un état manuel, parce que le rechargement doit être déclenché par un menu déroulant et qu'on veut garder la liste précédente visible pendant le chargement (`LinearProgressIndicator` plutôt qu'écran blanc). Le **jeton de séquence** `_numeroRequete` est le point important : si l'utilisateur change rapidement de sélection, plusieurs requêtes sont en vol simultanément et rien ne garantit qu'elles reviennent dans l'ordre. Le test `if (monNumero != _numeroRequete) return;` jette silencieusement toute réponse qui n'est pas la plus récente. Sans lui, la liste de l'utilisateur 3 pourrait s'afficher alors que le menu indique « 5 ».

---

### Correction 7 — Rafraîchissement et détail

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

class Tache {
  final int id;
  final int idUtilisateur;
  final String titre;
  final bool terminee;

  const Tache({
    required this.id,
    required this.idUtilisateur,
    required this.titre,
    required this.terminee,
  });

  factory Tache.fromJson(Map<String, dynamic> j) => Tache(
        id: (j['id'] as num?)?.toInt() ?? 0,
        idUtilisateur: (j['userId'] as num?)?.toInt() ?? 0,
        titre: j['title'] as String? ?? 'Sans titre',
        terminee: j['completed'] as bool? ?? false,
      );
}

class Utilisateur {
  final int id;
  final String nom;
  final String courriel;
  final String ville;

  const Utilisateur({
    required this.id,
    required this.nom,
    required this.courriel,
    required this.ville,
  });

  factory Utilisateur.fromJson(Map<String, dynamic> j) {
    final adresse = (j['address'] as Map<String, dynamic>?) ?? const {};
    return Utilisateur(
      id: (j['id'] as num?)?.toInt() ?? 0,
      nom: j['name'] as String? ?? 'Inconnu',
      courriel: j['email'] as String? ?? '',
      ville: adresse['city'] as String? ?? 'ville inconnue',
    );
  }
}

class ErreurApi implements Exception {
  final String message;
  const ErreurApi(this.message);
  @override
  String toString() => message;
}

class ServiceApi {
  final http.Client _client = http.Client();
  static const _hote = 'jsonplaceholder.typicode.com';

  Future<dynamic> _obtenir(String chemin, [Map<String, String>? p]) async {
    try {
      final r = await _client
          .get(Uri.https(_hote, chemin, p),
              headers: const {'Accept': 'application/json'})
          .timeout(const Duration(seconds: 10));
      if (r.statusCode < 200 || r.statusCode >= 300) {
        throw ErreurApi('Le serveur a répondu ${r.statusCode}.');
      }
      return jsonDecode(utf8.decode(r.bodyBytes));
    } on TimeoutException {
      throw const ErreurApi('Délai dépassé.');
    } on SocketException {
      throw const ErreurApi('Aucune connexion Internet.');
    } on http.ClientException {
      throw const ErreurApi('Connexion interrompue.');
    } on FormatException {
      throw const ErreurApi('Réponse illisible.');
    }
  }

  Future<List<Tache>> taches(int idUtilisateur) async {
    final brut = await _obtenir('/todos', {'userId': '$idUtilisateur'});
    if (brut is! List) throw const ErreurApi('Format inattendu.');
    return brut.map((e) => Tache.fromJson(e as Map<String, dynamic>)).toList();
  }

  Future<Utilisateur> utilisateur(int id) async {
    final brut = await _obtenir('/users/$id');
    if (brut is! Map<String, dynamic>) {
      throw const ErreurApi('Format inattendu.');
    }
    return Utilisateur.fromJson(brut);
  }

  void fermer() => _client.close();
}

void main() => runApp(const App7());

class App7 extends StatelessWidget {
  const App7({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.cyan, useMaterial3: true),
      home: const Page7(),
    );
  }
}

class Page7 extends StatefulWidget {
  const Page7({super.key});

  @override
  State<Page7> createState() => _Page7State();
}

class _Page7State extends State<Page7> {
  final ServiceApi _service = ServiceApi();
  int _idUtilisateur = 1;
  int _numero = 0;

  List<Tache> _taches = const [];
  bool _enCours = true;
  String? _erreur;

  @override
  void initState() {
    super.initState();
    _charger();
  }

  @override
  void dispose() {
    _service.fermer();
    super.dispose();
  }

  Future<void> _charger() async {
    final monNumero = ++_numero;
    setState(() {
      _enCours = true;
      _erreur = null;
    });
    try {
      final r = await _service.taches(_idUtilisateur);
      if (monNumero != _numero || !mounted) return;
      setState(() {
        _taches = r;
        _enCours = false;
      });
    } catch (e) {
      if (monNumero != _numero || !mounted) return;
      setState(() {
        _erreur = '$e';
        _enCours = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Exercice 7 — utilisateur $_idUtilisateur'),
        actions: [
          DropdownButton<int>(
            value: _idUtilisateur,
            underline: const SizedBox.shrink(),
            items: [
              for (int i = 1; i <= 10; i++)
                DropdownMenuItem(value: i, child: Text('  $i  ')),
            ],
            onChanged: (v) {
              if (v == null) return;
              setState(() => _idUtilisateur = v);
              _charger();
            },
          ),
        ],
      ),
      body: RefreshIndicator(onRefresh: _charger, child: _corps()),
    );
  }

  Widget _corps() {
    if (_enCours && _taches.isEmpty) {
      return const Center(child: CircularProgressIndicator());
    }

    if (_erreur != null) {
      // ListView, et non Column : le geste de rafraîchissement
      // doit rester disponible sur l'écran d'erreur.
      return ListView(
        physics: const AlwaysScrollableScrollPhysics(),
        children: [
          SizedBox(
            height: MediaQuery.of(context).size.height * 0.6,
            child: Center(
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  const Icon(Icons.cloud_off, size: 64),
                  const SizedBox(height: 16),
                  Text(_erreur!),
                  const SizedBox(height: 8),
                  const Text('Tirez vers le bas pour réessayer.'),
                ],
              ),
            ),
          ),
        ],
      );
    }

    return ListView.separated(
      physics: const AlwaysScrollableScrollPhysics(),
      itemCount: _taches.length,
      separatorBuilder: (_, __) => const Divider(height: 1),
      itemBuilder: (context, i) {
        final t = _taches[i];
        return ListTile(
          leading: Icon(
            t.terminee ? Icons.check_circle : Icons.radio_button_unchecked,
            color: t.terminee ? Colors.green : null,
          ),
          title: Text(t.titre),
          trailing: const Icon(Icons.chevron_right),
          onTap: () => Navigator.of(context).push(
            MaterialPageRoute<void>(
              builder: (_) => PageDetail(tache: t, service: _service),
            ),
          ),
        );
      },
    );
  }
}

class PageDetail extends StatefulWidget {
  final Tache tache;
  final ServiceApi service;

  const PageDetail({super.key, required this.tache, required this.service});

  @override
  State<PageDetail> createState() => _PageDetailState();
}

class _PageDetailState extends State<PageDetail> {
  late Future<Utilisateur> _futur;

  @override
  void initState() {
    super.initState();
    _futur = widget.service.utilisateur(widget.tache.idUtilisateur);
  }

  void _reessayer() => setState(
      () => _futur = widget.service.utilisateur(widget.tache.idUtilisateur));

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Tâche #${widget.tache.id}')),
      body: ListView(
        padding: const EdgeInsets.all(20),
        children: [
          Text(widget.tache.titre,
              style: Theme.of(context).textTheme.headlineSmall),
          const SizedBox(height: 12),
          Chip(
            avatar: Icon(
              widget.tache.terminee ? Icons.check : Icons.close,
              size: 18,
            ),
            label: Text(widget.tache.terminee ? 'Terminée' : 'À faire'),
          ),
          const Divider(height: 40),
          Text('Propriétaire',
              style: Theme.of(context).textTheme.titleMedium),
          const SizedBox(height: 12),
          FutureBuilder<Utilisateur>(
            future: _futur,
            builder: (context, snapshot) {
              if (snapshot.connectionState == ConnectionState.waiting) {
                return const Padding(
                  padding: EdgeInsets.all(24),
                  child: Center(child: CircularProgressIndicator()),
                );
              }
              if (snapshot.hasError) {
                return Column(
                  children: [
                    Text('${snapshot.error}'),
                    const SizedBox(height: 12),
                    FilledButton.tonal(
                      onPressed: _reessayer,
                      child: const Text('Réessayer'),
                    ),
                  ],
                );
              }
              final u = snapshot.data!;
              return Card(
                child: Column(
                  children: [
                    ListTile(
                      leading: CircleAvatar(child: Text(u.nom[0])),
                      title: Text(u.nom),
                      subtitle: Text('identifiant ${u.id}'),
                    ),
                    ListTile(
                      leading: const Icon(Icons.email_outlined),
                      title: Text(u.courriel),
                    ),
                    ListTile(
                      leading: const Icon(Icons.location_city),
                      title: Text(u.ville),
                    ),
                  ],
                ),
              );
            },
          ),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
Liste des 20 tâches de l'utilisateur 1, rafraîchissable par geste.
Toucher « delectus aut autem » :
  Titre en grand, puce « À faire »,
  puis un indicateur, puis la carte :
     L  Leanne Graham — identifiant 1
     [courriel]  Sincere@april.biz
     [ville]     Gwenborough
```

**Explication :** trois points. **Un**, l'écran d'erreur est un `ListView` avec `AlwaysScrollableScrollPhysics` : sans cela, `RefreshIndicator` cesse de fonctionner dès qu'une erreur survient, ce qui piège l'utilisateur. **Deux**, `_charger` renvoie un `Future<void>` et est passé directement à `onRefresh` : l'animation tourne exactement le temps de la requête. **Trois**, l'écran de détail a son propre `FutureBuilder` avec ses trois états, et reçoit le service par constructeur plutôt que d'en créer un nouveau : la connexion persistante est réutilisée. Notez `Utilisateur.fromJson` qui descend dans `address` via une variable locale protégée par `?? const {}` : accéder à `j['address']['city']` directement planterait si le champ manquait.

---

### Correction 8 — Création par `POST`

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

class Tache {
  final int id;
  final int idUtilisateur;
  final String titre;
  final bool terminee;

  const Tache({
    required this.id,
    required this.idUtilisateur,
    required this.titre,
    required this.terminee,
  });

  factory Tache.fromJson(Map<String, dynamic> j) => Tache(
        id: (j['id'] as num?)?.toInt() ?? 0,
        idUtilisateur: (j['userId'] as num?)?.toInt() ?? 0,
        titre: j['title'] as String? ?? 'Sans titre',
        terminee: j['completed'] as bool? ?? false,
      );
}

class ErreurApi implements Exception {
  final String message;
  const ErreurApi(this.message);
  @override
  String toString() => message;
}

class ServiceTaches {
  final http.Client _client = http.Client();
  static const _hote = 'jsonplaceholder.typicode.com';
  static const _enTetesJson = {
    'Content-Type': 'application/json; charset=utf-8',
    'Accept': 'application/json',
  };

  Future<List<Tache>> lister() async {
    try {
      final r = await _client
          .get(Uri.https(_hote, '/todos', {'_limit': '15'}))
          .timeout(const Duration(seconds: 10));
      if (r.statusCode < 200 || r.statusCode >= 300) {
        throw ErreurApi('Erreur ${r.statusCode}.');
      }
      final brut = jsonDecode(utf8.decode(r.bodyBytes));
      if (brut is! List) throw const ErreurApi('Format inattendu.');
      return brut.map((e) => Tache.fromJson(e as Map<String, dynamic>)).toList();
    } on TimeoutException {
      throw const ErreurApi('Délai dépassé.');
    } on SocketException {
      throw const ErreurApi('Aucune connexion Internet.');
    } on http.ClientException {
      throw const ErreurApi('Connexion interrompue.');
    } on FormatException {
      throw const ErreurApi('Réponse illisible.');
    }
  }

  Future<Tache> creer({required String titre, required int idUtilisateur}) async {
    try {
      final r = await _client
          .post(
            Uri.https(_hote, '/todos'),
            headers: _enTetesJson,
            // jsonEncode ET Content-Type : les deux, toujours ensemble.
            body: jsonEncode({
              'title': titre,
              'completed': false,
              'userId': idUtilisateur,
            }),
          )
          .timeout(const Duration(seconds: 15));

      // Un POST réussi renvoie 201, parfois 200.
      if (r.statusCode != 201 && r.statusCode != 200) {
        throw ErreurApi('Création refusée (${r.statusCode}).');
      }
      return Tache.fromJson(
          jsonDecode(utf8.decode(r.bodyBytes)) as Map<String, dynamic>);
    } on TimeoutException {
      throw const ErreurApi('Le serveur met trop de temps à répondre.');
    } on SocketException {
      throw const ErreurApi('Aucune connexion Internet.');
    } on http.ClientException {
      throw const ErreurApi('Connexion interrompue.');
    } on FormatException {
      throw const ErreurApi('Réponse illisible.');
    }
  }

  void fermer() => _client.close();
}

void main() => runApp(const App8());

class App8 extends StatelessWidget {
  const App8({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.pink, useMaterial3: true),
      home: const Page8(),
    );
  }
}

class Page8 extends StatefulWidget {
  const Page8({super.key});

  @override
  State<Page8> createState() => _Page8State();
}

class _Page8State extends State<Page8> {
  final ServiceTaches _service = ServiceTaches();

  List<Tache> _taches = const [];
  bool _enCours = true;
  String? _erreur;

  @override
  void initState() {
    super.initState();
    _charger();
  }

  @override
  void dispose() {
    _service.fermer();
    super.dispose();
  }

  Future<void> _charger() async {
    setState(() {
      _enCours = true;
      _erreur = null;
    });
    try {
      final r = await _service.lister();
      if (!mounted) return;
      setState(() {
        _taches = r;
        _enCours = false;
      });
    } catch (e) {
      if (!mounted) return;
      setState(() {
        _erreur = '$e';
        _enCours = false;
      });
    }
  }

  Future<void> _ouvrirFormulaire() async {
    final creee = await showModalBottomSheet<Tache>(
      context: context,
      isScrollControlled: true,
      builder: (_) => FormulaireTache(service: _service),
    );

    if (creee == null || !mounted) return;

    // On insère localement : pas de rechargement complet.
    setState(() => _taches = [creee, ..._taches]);
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('Tâche créée, identifiant ${creee.id}.')),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Exercice 8')),
      floatingActionButton: FloatingActionButton(
        onPressed: _ouvrirFormulaire,
        child: const Icon(Icons.add),
      ),
      body: RefreshIndicator(
        onRefresh: _charger,
        child: _erreur != null
            ? ListView(
                physics: const AlwaysScrollableScrollPhysics(),
                children: [
                  const SizedBox(height: 160),
                  Center(child: Text(_erreur!)),
                  const SizedBox(height: 16),
                  Center(
                    child: FilledButton(
                      onPressed: _charger,
                      child: const Text('Réessayer'),
                    ),
                  ),
                ],
              )
            : _enCours && _taches.isEmpty
                ? const Center(child: CircularProgressIndicator())
                : ListView.separated(
                    physics: const AlwaysScrollableScrollPhysics(),
                    itemCount: _taches.length,
                    separatorBuilder: (_, __) => const Divider(height: 1),
                    itemBuilder: (context, i) => ListTile(
                      leading: Icon(
                        _taches[i].terminee
                            ? Icons.check_circle
                            : Icons.radio_button_unchecked,
                        color: _taches[i].terminee ? Colors.green : null,
                      ),
                      title: Text(_taches[i].titre),
                      trailing: Text('#${_taches[i].id}'),
                    ),
                  ),
      ),
    );
  }
}

class FormulaireTache extends StatefulWidget {
  final ServiceTaches service;
  const FormulaireTache({super.key, required this.service});

  @override
  State<FormulaireTache> createState() => _FormulaireTacheState();
}

class _FormulaireTacheState extends State<FormulaireTache> {
  final _cle = GlobalKey<FormState>();
  final _ctrl = TextEditingController();
  int _idUtilisateur = 1;
  bool _envoi = false;

  @override
  void dispose() {
    _ctrl.dispose();
    super.dispose();
  }

  Future<void> _envoyer() async {
    if (!_cle.currentState!.validate()) return;
    setState(() => _envoi = true);
    try {
      final creee = await widget.service.creer(
        titre: _ctrl.text.trim(),
        idUtilisateur: _idUtilisateur,
      );
      if (!mounted) return;
      Navigator.of(context).pop(creee);
    } catch (e) {
      if (!mounted) return;
      setState(() => _envoi = false);
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('$e'),
          backgroundColor: Theme.of(context).colorScheme.error,
        ),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: EdgeInsets.only(
        left: 20,
        right: 20,
        top: 24,
        bottom: MediaQuery.of(context).viewInsets.bottom + 24,
      ),
      child: Form(
        key: _cle,
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Text('Nouvelle tâche',
                style: Theme.of(context).textTheme.titleLarge),
            const SizedBox(height: 20),
            TextFormField(
              controller: _ctrl,
              autofocus: true,
              decoration: const InputDecoration(
                labelText: 'Titre',
                border: OutlineInputBorder(),
              ),
              validator: (v) => (v == null || v.trim().length < 3)
                  ? 'Trois caractères minimum.'
                  : null,
            ),
            const SizedBox(height: 16),
            Row(
              children: [
                const Text('Utilisateur : '),
                const SizedBox(width: 12),
                DropdownButton<int>(
                  value: _idUtilisateur,
                  items: [
                    for (int i = 1; i <= 10; i++)
                      DropdownMenuItem(value: i, child: Text('$i')),
                  ],
                  onChanged: _envoi
                      ? null
                      : (v) => setState(() => _idUtilisateur = v ?? 1),
                ),
              ],
            ),
            const SizedBox(height: 24),
            SizedBox(
              width: double.infinity,
              child: FilledButton.icon(
                // POST n'est pas idempotent : le bouton se désactive.
                onPressed: _envoi ? null : _envoyer,
                icon: _envoi
                    ? const SizedBox(
                        width: 18,
                        height: 18,
                        child: CircularProgressIndicator(strokeWidth: 2),
                      )
                    : const Icon(Icons.send),
                label: Text(_envoi ? 'Envoi...' : 'Créer'),
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
Appui sur « + » : une feuille s'ouvre.
Saisie « Forger une épée », utilisateur 3, appui sur « Créer » :
  le bouton se désactive et affiche « Envoi... »
  la feuille se referme
  SnackBar : « Tâche créée, identifiant 201. »
  la tâche apparaît EN TÊTE de la liste, avec « #201 »
```

**Explication :** trois points. **Un**, `jsonEncode` construit le corps **et** l'en-tête `Content-Type: application/json; charset=utf-8` l'annonce ; sans cet en-tête, la plupart des serveurs répondent `400`. **Deux**, le bouton est désactivé pendant l'envoi via `onPressed: _envoi ? null : _envoyer` — `POST` n'étant pas idempotent, deux appuis rapides créeraient deux tâches. **Trois**, on n'appelle pas `_charger()` après la création : le serveur a renvoyé l'objet créé avec son identifiant, on l'insère directement avec `[creee, ..._taches]`. C'est instantané et cela économise une requête. Le `Navigator.pop(creee)` renvoie l'objet à l'écran appelant, patron classique de retour de données vu au chapitre 50.

---

### Correction 9 — Recherche avec débounce et pagination

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

class Produit {
  final int id;
  final String titre;
  final String categorie;
  final double prix;

  const Produit({
    required this.id,
    required this.titre,
    required this.categorie,
    required this.prix,
  });

  factory Produit.fromJson(Map<String, dynamic> j) => Produit(
        id: (j['id'] as num?)?.toInt() ?? 0,
        titre: j['title'] as String? ?? 'Sans titre',
        categorie: j['category'] as String? ?? 'divers',
        prix: (j['price'] as num?)?.toDouble() ?? 0,
      );
}

class PageProduits {
  final List<Produit> produits;
  final int total;
  const PageProduits({required this.produits, required this.total});
}

class ErreurApi implements Exception {
  final String message;
  const ErreurApi(this.message);
  @override
  String toString() => message;
}

class ServiceProduits {
  final http.Client _client = http.Client();

  Future<PageProduits> charger({
    required int saut,
    required int limite,
    String requete = '',
  }) async {
    final chemin = requete.isEmpty ? '/products' : '/products/search';
    final parametres = <String, String>{
      'limit': '$limite',
      'skip': '$saut',
      'select': 'title,price,category',
      if (requete.isNotEmpty) 'q': requete,
    };

    try {
      final r = await _client
          .get(Uri.https('dummyjson.com', chemin, parametres),
              headers: const {'Accept': 'application/json'})
          .timeout(const Duration(seconds: 10));

      if (r.statusCode < 200 || r.statusCode >= 300) {
        throw ErreurApi('Le serveur a répondu ${r.statusCode}.');
      }

      final j = jsonDecode(utf8.decode(r.bodyBytes));
      if (j is! Map<String, dynamic>) {
        throw const ErreurApi('Format inattendu.');
      }
      return PageProduits(
        produits: (j['products'] as List<dynamic>? ?? const [])
            .map((e) => Produit.fromJson(e as Map<String, dynamic>))
            .toList(),
        total: (j['total'] as num?)?.toInt() ?? 0,
      );
    } on TimeoutException {
      throw const ErreurApi('Délai dépassé.');
    } on SocketException {
      throw const ErreurApi('Aucune connexion Internet.');
    } on http.ClientException {
      throw const ErreurApi('Connexion interrompue.');
    } on FormatException {
      throw const ErreurApi('Réponse illisible.');
    }
  }

  void fermer() => _client.close();
}

void main() => runApp(const App9());

class App9 extends StatelessWidget {
  const App9({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.brown, useMaterial3: true),
      home: const Page9(),
    );
  }
}

class Page9 extends StatefulWidget {
  const Page9({super.key});

  @override
  State<Page9> createState() => _Page9State();
}

class _Page9State extends State<Page9> {
  static const int _taillePage = 20;

  final ServiceProduits _service = ServiceProduits();
  final TextEditingController _ctrl = TextEditingController();
  final ScrollController _defilement = ScrollController();

  Timer? _minuteur;
  int _numeroRecherche = 0;

  final List<Produit> _produits = [];
  String _requete = '';
  int _total = 0;
  bool _enCours = false;
  bool _toutCharge = false;
  String? _erreur;

  @override
  void initState() {
    super.initState();
    _defilement.addListener(_surDefilement);
    _chargerSuite();
  }

  @override
  void dispose() {
    _minuteur?.cancel();
    _defilement.removeListener(_surDefilement);
    _defilement.dispose();
    _ctrl.dispose();
    _service.fermer();
    super.dispose();
  }

  void _surDefilement() {
    if (!_defilement.hasClients) return;
    if (_defilement.position.pixels >=
        _defilement.position.maxScrollExtent * 0.8) {
      _chargerSuite();
    }
  }

  /// Charge la page suivante et l'AJOUTE à la liste existante.
  Future<void> _chargerSuite() async {
    if (_enCours || _toutCharge) return;

    final monNumero = _numeroRecherche;
    setState(() {
      _enCours = true;
      _erreur = null;
    });

    try {
      final page = await _service.charger(
        saut: _produits.length,
        limite: _taillePage,
        requete: _requete,
      );
      // Une nouvelle recherche est partie : ce résultat est périmé.
      if (monNumero != _numeroRecherche || !mounted) return;
      setState(() {
        _produits.addAll(page.produits);
        _total = page.total;
        _toutCharge = page.produits.isEmpty || _produits.length >= page.total;
        _enCours = false;
      });
    } catch (e) {
      if (monNumero != _numeroRecherche || !mounted) return;
      setState(() {
        _erreur = '$e';
        _enCours = false;
      });
    }
  }

  /// Une nouvelle recherche remet la pagination à zéro.
  void _relancer(String requete) {
    _numeroRecherche++;
    setState(() {
      _requete = requete;
      _produits.clear();
      _total = 0;
      _toutCharge = false;
      _erreur = null;
      _enCours = false;
    });
    _chargerSuite();
  }

  void _surTexte(String texte) {
    _minuteur?.cancel();
    _minuteur = Timer(
      const Duration(milliseconds: 400),
      () => _relancer(texte.trim()),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('${_produits.length} / $_total'),
        bottom: PreferredSize(
          preferredSize: const Size.fromHeight(64),
          child: Padding(
            padding: const EdgeInsets.fromLTRB(12, 0, 12, 12),
            child: TextField(
              controller: _ctrl,
              onChanged: _surTexte,
              decoration: InputDecoration(
                hintText: 'Rechercher...',
                prefixIcon: const Icon(Icons.search),
                filled: true,
                fillColor: Theme.of(context).colorScheme.surface,
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(28),
                  borderSide: BorderSide.none,
                ),
                suffixIcon: _ctrl.text.isEmpty
                    ? null
                    : IconButton(
                        icon: const Icon(Icons.clear),
                        onPressed: () {
                          _ctrl.clear();
                          _minuteur?.cancel();
                          _relancer('');
                        },
                      ),
              ),
            ),
          ),
        ),
      ),
      body: RefreshIndicator(
        onRefresh: () async => _relancer(_requete),
        child: ListView.builder(
          controller: _defilement,
          physics: const AlwaysScrollableScrollPhysics(),
          itemCount: _produits.length + 1,
          itemBuilder: (context, i) {
            if (i < _produits.length) {
              final p = _produits[i];
              return ListTile(
                leading: CircleAvatar(child: Text('${i + 1}')),
                title: Text(p.titre),
                subtitle: Text(p.categorie),
                trailing: Text('${p.prix.toStringAsFixed(2)} EUR'),
              );
            }
            return _Pied(
              enCours: _enCours,
              erreur: _erreur,
              termine: _toutCharge,
              vide: _produits.isEmpty && !_enCours && _erreur == null,
              requete: _requete,
              onReessayer: _chargerSuite,
            );
          },
        ),
      ),
    );
  }
}

class _Pied extends StatelessWidget {
  final bool enCours;
  final String? erreur;
  final bool termine;
  final bool vide;
  final String requete;
  final VoidCallback onReessayer;

  const _Pied({
    required this.enCours,
    required this.erreur,
    required this.termine,
    required this.vide,
    required this.requete,
    required this.onReessayer,
  });

  @override
  Widget build(BuildContext context) {
    if (erreur != null) {
      return Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          children: [
            const Icon(Icons.cloud_off, size: 48),
            const SizedBox(height: 12),
            Text(erreur!, textAlign: TextAlign.center),
            const SizedBox(height: 12),
            FilledButton.tonal(
              onPressed: onReessayer,
              child: const Text('Réessayer'),
            ),
          ],
        ),
      );
    }
    if (enCours) {
      return const Padding(
        padding: EdgeInsets.all(24),
        child: Center(child: CircularProgressIndicator()),
      );
    }
    if (vide) {
      return Padding(
        padding: const EdgeInsets.all(48),
        child: Column(
          children: [
            const Icon(Icons.search_off, size: 56),
            const SizedBox(height: 12),
            Text(
              requete.isEmpty
                  ? 'Le catalogue est vide.'
                  : 'Aucun résultat pour « $requete ».',
              textAlign: TextAlign.center,
            ),
          ],
        ),
      );
    }
    if (termine) {
      return const Padding(
        padding: EdgeInsets.all(24),
        child: Center(child: Text('Fin de la liste.')),
      );
    }
    return const SizedBox(height: 48);
  }
}
```

**Résultat :**

```text
Ouverture         : « 20 / 194 », 20 produits.
Défilement        : « 40 / 194 », « 60 / 194 »... jusqu'à
                    « 194 / 194 » et « Fin de la liste. »
Saisie « phone »  : une seule requête 400 ms après la dernière frappe,
                    la liste repart de zéro : « 23 / 23 ».
Saisie « zzzz »   : « 0 / 0 » et « Aucun résultat pour « zzzz ». »
Effacement        : retour au catalogue complet.
```

**Explication :** quatre mécanismes cohabitent. Le **débounce** (`Timer` de 400 ms annulé à chaque frappe) évite une requête par caractère. Le **jeton de séquence** `_numeroRecherche`, incrémenté par `_relancer`, invalide les pages en vol quand la requête change : sans lui, la page 3 de « phone » pourrait arriver après la page 1 de « laptop » et polluer la liste. Le drapeau `_enCours` interdit deux requêtes de pagination simultanées, sinon un défilement rapide en lancerait des dizaines. Le drapeau `_toutCharge` arrête la pagination une fois le total atteint. Enfin, `_produits.addAll(...)` complète la liste au lieu de la remplacer, ce qui préserve la position de défilement ; c'est `_relancer` — et lui seul — qui appelle `clear()`.

---

### Correction 10 — Dépôt factice et tests

Le fichier `lib/main.dart` :

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';
import 'dart:math';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

// ───────────────────────── modeles/tache.dart ─────────────────────────

class Tache {
  final int id;
  final int idUtilisateur;
  final String titre;
  final bool terminee;

  const Tache({
    required this.id,
    required this.idUtilisateur,
    required this.titre,
    required this.terminee,
  });

  factory Tache.fromJson(Map<String, dynamic> j) => Tache(
        id: (j['id'] as num?)?.toInt() ?? 0,
        idUtilisateur: (j['userId'] as num?)?.toInt() ?? 0,
        titre: j['title'] as String? ?? 'Sans titre',
        terminee: j['completed'] as bool? ?? false,
      );
}

class ErreurApi implements Exception {
  final String message;
  final int? codeStatut;
  const ErreurApi(this.message, {this.codeStatut});
  @override
  String toString() => message;
}

// ─────────────────── services/depot_taches.dart ───────────────────────

/// Le CONTRAT. Les écrans ne dépendent que de cette abstraction.
abstract class DepotTaches {
  Future<List<Tache>> lister({int limite});
  Future<Tache> obtenir(int id);
  void fermer();
}

/// Implémentation réseau.
class DepotTachesHttp implements DepotTaches {
  final http.Client _client;
  static const _hote = 'jsonplaceholder.typicode.com';

  DepotTachesHttp({http.Client? client}) : _client = client ?? http.Client();

  @override
  Future<List<Tache>> lister({int limite = 20}) async {
    final brut = await _obtenirJson('/todos', {'_limit': '$limite'});
    if (brut is! List) {
      throw const ErreurApi('Le serveur n\'a pas renvoyé de tableau.');
    }
    return brut.map((e) => Tache.fromJson(e as Map<String, dynamic>)).toList();
  }

  @override
  Future<Tache> obtenir(int id) async {
    final brut = await _obtenirJson('/todos/$id');
    if (brut is! Map<String, dynamic>) {
      throw const ErreurApi('Format inattendu.');
    }
    return Tache.fromJson(brut);
  }

  Future<dynamic> _obtenirJson(String chemin, [Map<String, String>? p]) async {
    try {
      final r = await _client
          .get(Uri.https(_hote, chemin, p),
              headers: const {'Accept': 'application/json'})
          .timeout(const Duration(seconds: 10));

      if (r.statusCode < 200 || r.statusCode >= 300) {
        throw ErreurApi('Le serveur a répondu ${r.statusCode}.',
            codeStatut: r.statusCode);
      }
      return jsonDecode(utf8.decode(r.bodyBytes));
    } on TimeoutException {
      throw const ErreurApi('Délai dépassé.');
    } on SocketException {
      throw const ErreurApi('Aucune connexion Internet.');
    } on http.ClientException {
      throw const ErreurApi('Connexion interrompue.');
    } on FormatException {
      throw const ErreurApi('Réponse illisible.');
    }
  }

  @override
  void fermer() => _client.close();
}

/// Implémentation factice : aucun réseau, latence et pannes simulées.
class DepotTachesFactice implements DepotTaches {
  final Random _alea = Random();

  static const List<Tache> _donnees = [
    Tache(id: 1, idUtilisateur: 1, titre: 'Forger une épée', terminee: true),
    Tache(id: 2, idUtilisateur: 1, titre: 'Récolter dix herbes', terminee: false),
    Tache(id: 3, idUtilisateur: 2, titre: 'Vaincre le boss', terminee: false),
    Tache(id: 4, idUtilisateur: 2, titre: 'Parler au forgeron', terminee: true),
    Tache(id: 5, idUtilisateur: 3, titre: 'Explorer la grotte', terminee: false),
  ];

  /// Latence réaliste et échec une fois sur cinq.
  Future<void> _simuler() async {
    await Future<void>.delayed(
      Duration(milliseconds: 200 + _alea.nextInt(1300)),
    );
    if (_alea.nextInt(5) == 0) {
      throw const ErreurApi('Panne simulée du serveur.', codeStatut: 503);
    }
  }

  @override
  Future<List<Tache>> lister({int limite = 20}) async {
    await _simuler();
    return _donnees.take(limite).toList();
  }

  @override
  Future<Tache> obtenir(int id) async {
    await _simuler();
    final trouve = _donnees.where((t) => t.id == id);
    if (trouve.isEmpty) {
      throw const ErreurApi('Tâche introuvable.', codeStatut: 404);
    }
    return trouve.first;
  }

  @override
  void fermer() {}
}

// ─────────────────────────── main.dart ────────────────────────────────

/// Basculé par : flutter run --dart-define=HORS_LIGNE=true
const bool modeHorsLigne = bool.fromEnvironment('HORS_LIGNE');

void main() => runApp(const App10());

class App10 extends StatelessWidget {
  const App10({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(colorSchemeSeed: Colors.lime, useMaterial3: true),
      home: const Page10(),
    );
  }
}

class Page10 extends StatefulWidget {
  const Page10({super.key});

  @override
  State<Page10> createState() => _Page10State();
}

class _Page10State extends State<Page10> {
  // UN SEUL endroit décide de l'implémentation.
  final DepotTaches _depot =
      modeHorsLigne ? DepotTachesFactice() : DepotTachesHttp();

  late Future<List<Tache>> _futur;

  @override
  void initState() {
    super.initState();
    _futur = _depot.lister();
  }

  @override
  void dispose() {
    _depot.fermer();
    super.dispose();
  }

  void _recharger() => setState(() => _futur = _depot.lister());

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(modeHorsLigne ? 'Exercice 10 (factice)' : 'Exercice 10'),
        actions: [
          IconButton(onPressed: _recharger, icon: const Icon(Icons.refresh)),
        ],
      ),
      body: FutureBuilder<List<Tache>>(
        future: _futur,
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }
          if (snapshot.hasError) {
            return Center(
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  const Icon(Icons.cloud_off, size: 64),
                  const SizedBox(height: 16),
                  Text('${snapshot.error}'),
                  const SizedBox(height: 20),
                  FilledButton(
                    onPressed: _recharger,
                    child: const Text('Réessayer'),
                  ),
                ],
              ),
            );
          }
          final taches = snapshot.data ?? const <Tache>[];
          if (taches.isEmpty) {
            return const Center(child: Text('Aucune tâche.'));
          }
          return ListView.separated(
            itemCount: taches.length,
            separatorBuilder: (_, __) => const Divider(height: 1),
            itemBuilder: (context, i) => ListTile(
              leading: Icon(
                taches[i].terminee
                    ? Icons.check_circle
                    : Icons.radio_button_unchecked,
                color: taches[i].terminee ? Colors.green : null,
              ),
              title: Text(taches[i].titre),
              subtitle: Text('utilisateur ${taches[i].idUtilisateur}'),
              trailing: Text('#${taches[i].id}'),
            ),
          );
        },
      ),
    );
  }
}
```

Le fichier `test/depot_taches_test.dart` :

```dart
import 'dart:convert';

import 'package:flutter_test/flutter_test.dart';
import 'package:http/http.dart' as http;
import 'package:http/testing.dart';

import 'package:mon_application/main.dart';

void main() {
  group('DepotTachesHttp.lister', () {
    test('décode correctement une réponse 200', () async {
      final client = MockClient((requete) async {
        expect(requete.method, 'GET');
        expect(requete.url.host, 'jsonplaceholder.typicode.com');
        expect(requete.url.path, '/todos');
        expect(requete.url.queryParameters['_limit'], '20');

        return http.Response(
          jsonEncode([
            {'id': 1, 'userId': 1, 'title': 'Forger', 'completed': true},
            {'id': 2, 'userId': 2, 'title': 'Récolter', 'completed': false},
          ]),
          200,
          headers: {'content-type': 'application/json; charset=utf-8'},
        );
      });

      final depot = DepotTachesHttp(client: client);
      final taches = await depot.lister();

      expect(taches, hasLength(2));
      expect(taches.first.titre, 'Forger');
      expect(taches.first.terminee, isTrue);
      expect(taches.last.idUtilisateur, 2);
    });

    test('lève une ErreurApi portant le code sur une réponse 500', () async {
      final client =
          MockClient((_) async => http.Response('Erreur interne', 500));
      final depot = DepotTachesHttp(client: client);

      await expectLater(
        depot.lister(),
        throwsA(isA<ErreurApi>()
            .having((e) => e.codeStatut, 'codeStatut', 500)),
      );
    });

    test('lève une ErreurApi quand le corps n\'est pas du JSON', () async {
      final client = MockClient(
        (_) async => http.Response('<html>503</html>', 200),
      );
      final depot = DepotTachesHttp(client: client);

      await expectLater(depot.lister(), throwsA(isA<ErreurApi>()));
    });

    test('renvoie une liste vide sur un tableau vide, sans erreur', () async {
      final client = MockClient((_) async => http.Response('[]', 200));
      final depot = DepotTachesHttp(client: client);

      expect(await depot.lister(), isEmpty);
    });

    test('tolère des champs manquants sans planter', () async {
      final client = MockClient(
        (_) async => http.Response(jsonEncode([
              {'id': 7}
            ]), 200),
      );
      final depot = DepotTachesHttp(client: client);

      final taches = await depot.lister();
      expect(taches.single.id, 7);
      expect(taches.single.titre, 'Sans titre');
      expect(taches.single.terminee, isFalse);
      expect(taches.single.idUtilisateur, 0);
    });
  });
}
```

Exécution :

```text
flutter test
```

**Résultat :**

```text
00:02 +5: All tests passed!
```

Et pour lancer l'application sans réseau :

```text
flutter run --dart-define=HORS_LIGNE=true
```

**Explication :** l'abstraction `DepotTaches` est ce qui rend tout le reste possible. L'écran ne connaît que ce contrat : il ne sait pas — et n'a pas à savoir — si les données viennent du réseau ou d'une liste en dur. Le basculement tient en une expression conditionnelle, pilotée par `bool.fromEnvironment`, donc sans modifier une seule ligne de code entre les deux modes. Le dépôt factice **simule une latence de 200 à 1500 ms et échoue une fois sur cinq** : cette imperfection délibérée vous force à écrire correctement les états de chargement et d'erreur dès le premier jour, au lieu de les découvrir en production. Côté tests, `MockClient` remplace le vrai `http.Client` : les cinq tests s'exécutent en deux secondes, sans réseau, et couvrent précisément les cas qu'un serveur en bonne santé ne produit jamais — le `500`, le HTML au lieu du JSON, le tableau vide, le champ manquant. Rien de tout cela ne serait testable si `DepotTachesHttp` appelait `http.get()` directement au lieu d'accepter un client en paramètre.

---

## Et maintenant ?

Vos applications savent désormais aller chercher des données sur un serveur, les transformer en objets Dart typés, les afficher, et surtout **échouer proprement** : chargement, erreur, vide, succès, réessai, rafraîchissement, recherche et pagination. C'est le socle de toute application connectée.

Il reste un problème, et il est immédiat : **dès que le réseau disparaît, votre application n'a plus rien à montrer.** Elle repart de zéro à chaque démarrage. Le mode sombre choisi par l'utilisateur est oublié, sa liste de favoris s'évapore, et la liste de produits qu'il vient de consulter doit être rechargée intégralement dans le métro.

Le chapitre suivant y répond. Vous y apprendrez à écrire des données **sur l'appareil** : `shared_preferences` pour les réglages simples, les fichiers via `path_provider` pour les données volumineuses, `sqflite` pour une vraie base de données locale interrogeable. Et surtout, vous combinerez les deux chapitres pour construire un **cache hors-ligne** : afficher immédiatement les dernières données connues, puis les rafraîchir en arrière-plan dès que le réseau revient.

[54-PARTIE-1B—STOCKAGE-LOCAL-ET-PERSISTANCE.md](./54-PARTIE-1B—STOCKAGE-LOCAL-ET-PERSISTANCE.md)
