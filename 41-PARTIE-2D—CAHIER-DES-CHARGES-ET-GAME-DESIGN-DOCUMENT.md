# PARTIE 2D — VOTRE PROPRE JEU
# CHAPITRE 41 — CAHIER DES CHARGES, GAME DESIGN DOCUMENT ET ARCHITECTURE

> **Version de Flame utilisée dans ce chapitre :** `flame` **1.38.0** (publiée le 19 juillet 2026).
> **Date de vérification des API :** 8 août 2026, sur `https://docs.flame-engine.org/latest/`
> et `https://pub.dev/packages/flame`.
> **Contraintes SDK :** `sdk: ">=3.12.0 <4.0.0"`, `flutter: ">=3.44.0"`.
>
> **Niveau :** intermédiaire
> **Durée estimée :** 5 h (dont 3 h de rédaction de vos propres documents)
> **Pré-requis :** toute la PARTIE 2C (chapitres 35 à 40), c'est-à-dire un jeu complet livré. Côté Dart : chapitre 16 (organisation d'un projet, `pubspec.yaml`, Git), chapitre 18 (mini-projet final). Côté Flame : chapitre 26 (architecture et états), chapitre 29 (assets).
> **Ce que vous saurez faire à la fin :** transformer une idée de jeu vague en trois documents exploitables — un pitch, un cahier des charges et un Game Design Document — puis en une arborescence de projet, un planning découpé en jalons et une grille de playtest, de façon à ce que votre jeu personnel soit **fini** et non abandonné.

---

## 41.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer pourquoi la grande majorité des projets de jeu amateurs ne sont jamais terminés ;
- reconnaître, chez vous, les signes du projet trop ambitieux ;
- définir un périmètre minimal jouable et le défendre ;
- passer d'une envie floue à une idée testable grâce à trois questions ;
- écrire un pitch de jeu en une seule phrase, sans adjectif inutile ;
- distinguer un cahier des charges d'un Game Design Document et savoir lequel écrire en premier ;
- remplir un gabarit complet de cahier des charges ;
- séparer exigences fonctionnelles et exigences non fonctionnelles ;
- écrire un périmètre en deux colonnes : dans le lot / hors du lot ;
- remplir un gabarit complet de Game Design Document, section par section ;
- décrire les mécaniques principales de votre jeu sans les confondre avec ses fonctionnalités ;
- dessiner la boucle de gameplay de votre jeu sous forme de schéma ;
- tracer une courbe de difficulté et la traduire en chiffres ;
- concevoir une économie de jeu cohérente (points, ressources, récompenses) ;
- rédiger une direction artistique tenable pour une personne seule ;
- planifier le son et la musique sans dépendre d'un musicien ;
- lister les écrans de votre jeu et les transitions entre eux ;
- chiffrer le contenu : niveaux, ennemis, objets ;
- lire un GDD complet et rempli, celui du « Donjon de Dart » ;
- dessiner le schéma en couches de votre architecture technique ;
- choisir une structure de dossiers et la justifier ;
- appliquer des conventions de nommage stables ;
- organiser vos assets : nommage, tailles, formats ;
- tenir un tableau de suivi des licences, obligatoire avant toute publication ;
- découper votre projet en jalons livrables ;
- comprendre pourquoi le jalon « ça tourne » précède toujours le jalon « c'est beau » ;
- configurer Git pour un projet de jeu, `.gitignore` compris ;
- prototyper une mécanique sur papier avant de la coder ;
- organiser un playtest et savoir quoi observer ;
- remplir une grille de playtest ;
- éviter les douze pièges classiques du débutant ;
- choisir l'un des trois sujets de projet final proposés, avec son périmètre chiffré.

---

## 41.1 — Pourquoi 90 % des projets de jeu ne sont jamais finis

Vous venez de terminer un jeu complet. Vous ne l'avez pas conçu : il vous a été donné, chapitre après chapitre, avec sa spécification, ses noms de classes et ses valeurs chiffrées. Ce chapitre vous retire ce filet.

Il faut donc commencer par un constat désagréable. Sur les forums de développement de jeux, sur itch.io, sur GitHub, on trouve des dizaines de milliers de dépôts de jeux abandonnés. Le schéma est presque toujours le même.

```text
   enthousiasme
        │
        │  ██████████
        │  ██████████
        │  ██████████        semaine 1 : "je vais faire un RPG open world"
        │  ██████████
        │  ██████████ ████
        │  ██████████ ████   semaine 2 : le personnage bouge
        │  ██████████ ████ ██
        │  ██████████ ████ ██  semaine 3 : refonte de l'architecture
        │  ██████████ ████ ██ █
        │  ██████████ ████ ██ █  semaine 4 : "il faudrait de vrais graphismes"
        │  ██████████ ████ ██ █ ▏
        └──────────────────────────────────► temps
                                        abandon
```

Les causes réelles sont peu nombreuses et parfaitement identifiables.

**Cause 1 — le périmètre n'a jamais été écrit.** Tant que le projet n'existe que dans votre tête, il grossit tout seul. Chaque bonne idée s'ajoute aux précédentes sans jamais en remplacer aucune. Un projet non écrit n'a pas de fin, donc il ne peut pas être fini.

**Cause 2 — la première tâche codée n'était pas la bonne.** Beaucoup commencent par le menu principal, le système de sauvegarde ou l'écran d'options. Ce sont des tâches confortables : elles ressemblent à du développement d'application classique. Mais elles ne répondent pas à la seule question qui compte : *est-ce que ce jeu est amusant ?*

**Cause 3 — la boucle de gameplay n'a jamais été jouable.** Si, au bout de trois semaines, personne n'a encore pu jouer trente secondes d'affilée, le projet ne survivra pas. On perd la motivation quand on ne voit pas le jeu.

**Cause 4 — la production de contenu a été sous-estimée.** Coder un ennemi prend deux heures. Concevoir, équilibrer et placer vingt ennemis dans quinze niveaux prend deux mois. Le code n'est pas le goulot d'étranglement ; le contenu l'est.

**Cause 5 — les assets sont devenus un blocage.** « Je reprendrai quand j'aurai de vrais graphismes » est la phrase d'adieu la plus fréquente du développement de jeu amateur.

**Cause 6 — aucune date, aucun jalon.** Un projet sans jalon n'a pas de progression mesurable. Sans progression mesurable, la sensation d'avancer disparaît, et avec elle la motivation.

| Cause d'abandon | Contre-mesure de ce chapitre |
| --- | --- |
| Périmètre non écrit | Cahier des charges, section 41.7 |
| Mauvaise première tâche | Périmètre minimal jouable, section 41.3 |
| Boucle jamais jouable | Jalon « ça tourne », section 41.29 |
| Contenu sous-estimé | Chiffrage du contenu, section 41.21 |
| Blocage sur les assets | Gestion des assets, section 41.26 |
| Aucun jalon | Planning, section 41.28 |

> **À retenir.** Un jeu n'est pas abandonné par manque de compétence technique, mais par absence de décisions écrites. Ce chapitre ne vous apprend aucune API : il vous apprend à décider avant de coder.

---

## 41.2 — Le piège du projet trop ambitieux

Le projet trop ambitieux ne se reconnaît pas à sa taille absolue, mais au **rapport entre ce qu'il exige et ce que vous pouvez produire par semaine**.

Voici un test simple. Prenez votre idée de jeu et remplissez le tableau suivant honnêtement.

| Question | Votre réponse | Seuil d'alerte |
| --- | --- | --- |
| Combien d'heures par semaine y consacrerez-vous ? | ... h | moins de 5 h |
| Combien de types d'ennemis différents ? | ... | plus de 4 |
| Combien de niveaux ? | ... | plus de 6 |
| Combien de sprites à dessiner ou trouver ? | ... | plus de 30 |
| Combien de systèmes distincts (inventaire, dialogue, craft, quêtes) ? | ... | plus de 2 |
| Le jeu a-t-il besoin d'un réseau ou d'un serveur ? | oui / non | oui |
| Le jeu a-t-il besoin d'une sauvegarde de partie complète ? | oui / non | oui, si c'est votre premier jeu |
| Combien de personnes travaillent dessus ? | ... | 1, avec plus de 2 seuils déjà dépassés |

Chaque seuil franchi multiplie la durée du projet. Ce n'est pas une addition : c'est un produit, parce que les systèmes interagissent. Un inventaire seul est simple. Un inventaire **plus** une sauvegarde **plus** des objets équipables qui modifient les statistiques du joueur, ce n'est pas trois fois plus de travail, c'est dix fois plus de cas de test.

### Les trois idées qui tuent un premier projet

**Le monde ouvert.** Il exige une carte grande, donc du contenu partout, donc du chargement par morceaux, donc de la sauvegarde d'état de monde. Écartez-le.

**Le multijoueur.** Il exige un serveur, une synchronisation, une gestion de la latence, une sécurité minimale. Chacun de ces points est un projet à lui seul. Écartez-le.

**Le jeu à histoire.** Il exige des dialogues, des personnages, des embranchements, de la localisation, et surtout de l'écriture — beaucoup d'écriture. Écartez-le, ou réduisez-le à trois lignes de texte au total.

### Le bon réflexe : diviser par cinq

Prenez votre idée. Divisez toutes les quantités par cinq. Si l'idée reste intéressante, elle est bonne. Si elle devient insipide, c'est que l'intérêt reposait sur la quantité, pas sur la mécanique — et c'est un mauvais signe.

```text
   IDÉE INITIALE                      IDÉE DIVISÉE PAR CINQ
   ─────────────────────────          ─────────────────────────
   20 niveaux                    →    4 niveaux
   10 types d'ennemis            →    2 types d'ennemis
   5 armes                       →    1 arme
   un boss par monde             →    1 boss, à la fin
   inventaire + craft + magie    →    ramasser des pièces
   histoire en 12 chapitres      →    une phrase à l'écran-titre
```

> **À retenir.** L'ambition ne se mesure pas en idées, mais en heures. Divisez par cinq, et vous obtiendrez un projet finissable.

---

## 41.3 — La règle du périmètre minimal jouable

Le **périmètre minimal jouable** (souvent appelé *vertical slice* ou *tranche verticale*) est la plus petite version de votre jeu qui soit réellement jouable du début à la fin.

Définition précise, que vous pouvez recopier :

> Le périmètre minimal jouable est la version du jeu dans laquelle un joueur peut lancer l'application, jouer une partie complète, gagner ou perdre, et recommencer, sans qu'aucun message d'erreur ni écran vide n'apparaisse.

Ce qui compte dans cette définition, c'est le mot **complète**. Un joueur doit pouvoir aller jusqu'au bout. Peu importe que le bout arrive au bout de quarante secondes.

### Ce que le périmètre minimal jouable contient toujours

| Élément | Pourquoi il est obligatoire |
| --- | --- |
| Un écran de démarrage | Sinon le joueur ne sait pas que le jeu a commencé |
| Une entrée de commande | Sans commande, ce n'est pas un jeu |
| Une boucle de gameplay | Le cœur de l'expérience |
| Une condition de victoire | Sinon le joueur ne peut pas finir |
| Une condition de défaite | Sinon il n'y a pas d'enjeu |
| Un retour au menu ou un rejeu | Sinon le joueur est coincé |

### Ce qu'il ne contient jamais au premier tour

- les options (volume, langue, difficulté) ;
- les succès et les statistiques ;
- les animations de transition entre les écrans ;
- la sauvegarde ;
- les particules et les effets visuels ;
- plus d'un type d'ennemi ;
- plus d'un niveau.

### Application au « Donjon de Dart »

Le jeu de la PARTIE 2C, dans son périmètre minimal jouable, tenait en ceci :

```text
menu ──► une salle ──► un gobelin ──► une porte ──► écran de victoire ──► menu
                  └──► perdre 3 vies ──► écran Game Over ──► menu
```

Tout le reste — trois niveaux, deux types d'ennemis, le boss, les potions, les clés, le HUD, l'audio, la sauvegarde — a été ajouté **après** que cette boucle a été jouable.

### Le test des trente secondes

Posez-vous cette question à chaque fin de séance de travail :

> Si je donnais mon téléphone à quelqu'un maintenant, aurait-il trente secondes de jeu compréhensible ?

Tant que la réponse est non, ne travaillez sur rien d'autre que sur cette question.

> **À retenir.** Le périmètre minimal jouable n'est pas un brouillon : c'est la colonne vertébrale du projet. Tout le reste s'y accroche.

---

## 41.4 — Trouver une idée : les trois questions

Chercher « une bonne idée de jeu » ne mène nulle part, parce que la formule est trop vague. On remplace donc la recherche d'idée par trois questions concrètes, à répondre dans l'ordre.

### Question 1 — Que fait le joueur, physiquement, avec ses doigts ?

Répondez par un verbe d'action et un objet. Pas de généralité.

| Mauvaise réponse | Bonne réponse |
| --- | --- |
| « Il explore un monde fantastique » | « Il appuie à gauche et à droite pour esquiver des blocs qui tombent » |
| « Il vit une aventure » | « Il trace un trait au doigt pour dévier une bille » |
| « Il gère sa base » | « Il pose une tour toutes les dix secondes sur une grille » |

Si vous ne savez pas répondre en une phrase avec un verbe concret, vous n'avez pas encore d'idée de jeu : vous avez une envie d'ambiance.

### Question 2 — Qu'est-ce qui rend cette action difficile ?

Un jeu, c'est une action simple rendue difficile par une contrainte. Si l'action n'est pas difficile, il n'y a pas de jeu.

| Action | Contrainte qui crée le jeu |
| --- | --- |
| Sauter | La gravité, des trous, un temps limité |
| Ramasser des pièces | Des ennemis qui patrouillent, un chronomètre |
| Trier des objets | La vitesse d'arrivée qui augmente |
| Viser | La cible bouge, les munitions sont limitées |
| Poser des tuiles | La place manque, les tuiles arrivent au hasard |

### Question 3 — Pourquoi le joueur recommencerait-il après avoir perdu ?

C'est la question la plus souvent oubliée. Elle décide de la durée de vie du jeu.

| Raison de recommencer | Exemple |
| --- | --- |
| Battre son score | Un jeu d'arcade avec un meilleur score affiché |
| Aller plus loin | Un niveau jamais atteint |
| Essayer une autre stratégie | Plusieurs armes, plusieurs chemins |
| Débloquer quelque chose | Un personnage, un niveau bonus |
| La partie est très courte | « Encore une », en dix secondes |

### La grille des trois questions

Recopiez cette grille pour chacune de vos idées :

```markdown
### Idée : <nom provisoire>

1. Que fait le joueur, physiquement ?
   →

2. Qu'est-ce qui rend cette action difficile ?
   →

3. Pourquoi recommencerait-il après avoir perdu ?
   →

Verdict : idée retenue / idée écartée
Raison :
```

Remplissez-la pour trois idées différentes. Celle dont les trois réponses sont les plus courtes et les plus précises est presque toujours la meilleure.

> **À retenir.** Une idée de jeu se valide par un verbe, une contrainte et une raison de rejouer. Tout le reste — univers, graphismes, histoire — vient après.

---

## 41.5 — Le pitch en une phrase

Le pitch est la version compressée du jeu. Il tient en une phrase, se dit à voix haute en dix secondes, et sert trois fois par jour pendant tout le projet : quand vous hésitez sur une fonctionnalité, quand vous présentez votre travail, quand vous écrivez la description sur la boutique.

### La formule

```text
<Titre> est un <genre> dans lequel le joueur <action principale>
pour <objectif>, tout en <contrainte principale>.
```

### Exemples corrects

> **Donjon de Dart** est un jeu de plateformes 2D dans lequel le joueur traverse trois salles de donjon pour trouver la clé et atteindre la sortie, tout en évitant des gobelins et des chauves-souris avec trois vies seulement.

> **Chute Libre** est un jeu d'arcade vertical dans lequel le joueur déplace une plateforme pour rattraper des œufs qui tombent, tout en évitant des pierres qui accélèrent avec le temps.

> **Racine** est un jeu de réflexion sur grille dans lequel le joueur fait pousser une racine case par case pour atteindre une source d'eau, tout en gérant un nombre de segments limité.

### Exemples incorrects, et pourquoi

| Pitch raté | Défaut |
| --- | --- |
| « Un jeu de plateformes fun et immersif avec de superbes graphismes » | Aucun verbe d'action, que des adjectifs |
| « Un RPG où tout est possible » | Pas de périmètre, donc pas de projet |
| « Comme Zelda mais en mieux » | Comparaison sans contenu propre |
| « Un jeu où vous incarnez un héros qui doit sauver le monde » | Objectif générique, aucune mécanique |
| « Un puzzle-platformer-roguelike-deckbuilder » | Empilement de genres, signe de projet non décidé |

### Le test du pitch

Un bon pitch passe ces quatre tests :

1. **Test du verbe** : il contient au moins un verbe d'action concret.
2. **Test de l'adjectif** : on peut supprimer tous les adjectifs sans perdre de sens.
3. **Test de la contrainte** : il dit ce qui rend le jeu difficile.
4. **Test de la longueur** : il tient en une phrase, moins de quarante mots.

### Gabarit à recopier

```markdown
## Pitch

<Titre> est un <genre> dans lequel le joueur <action>
pour <objectif>, tout en <contrainte>.

Vérification :
- [ ] contient un verbe d'action
- [ ] ne repose sur aucun adjectif
- [ ] énonce la contrainte
- [ ] moins de 40 mots
```

> **À retenir.** Si vous ne savez pas pitcher votre jeu en une phrase, vous ne saurez pas non plus décider quelles fonctionnalités couper.

---

## 41.6 — Le cahier des charges : à quoi il sert

Beaucoup de développeurs débutants confondent le cahier des charges et le Game Design Document. Ce sont deux documents différents, écrits pour des raisons différentes.

| | Cahier des charges | Game Design Document |
| --- | --- | --- |
| Question à laquelle il répond | Que doit livrer le projet ? | Comment le jeu se joue-t-il ? |
| Public | Vous, votre client, votre enseignant, votre équipe | Vous, vos collaborateurs, vos testeurs |
| Nature | Contractuelle, vérifiable | Descriptive, évolutive |
| Contient des chiffres ? | Oui, des seuils à respecter | Oui, des valeurs d'équilibrage |
| Change souvent ? | Non, il est figé au début | Oui, à chaque playtest |
| Longueur typique | 2 à 4 pages | 5 à 20 pages |

Le cahier des charges répond à : **qu'est-ce qui sera livré, et comment saura-t-on que c'est fini ?**

### Les quatre fonctions du cahier des charges

**Fonction 1 — arrêter le périmètre.** Le document dit ce qui est dans le lot et ce qui n'y est pas. Toute idée nouvelle se compare à ce texte.

**Fonction 2 — définir « fini ».** Sans critères d'acceptation écrits, un projet n'est jamais fini : il est seulement abandonné plus tard.

**Fonction 3 — révéler les décisions manquantes.** L'exercice de rédaction fait apparaître les trous. « Sur quelles plateformes ? » « Avec quelle orientation d'écran ? » « Combien de temps dure une partie ? » Ces questions coûtent cinq minutes sur le papier et cinq jours dans le code.

**Fonction 4 — servir de référence en cas de désaccord.** En équipe, ou avec un enseignant qui note votre projet, le cahier des charges est le texte auquel tout le monde revient.

### Quand l'écrire

Avant la première ligne de code, et après le pitch. L'ordre est toujours le même :

```text
   idée ──► trois questions ──► pitch ──► cahier des charges ──► GDD ──► architecture ──► code
   (41.4)      (41.4)          (41.5)        (41.7)            (41.11)     (41.23)
```

Comptez une heure et demie pour un premier cahier des charges. Ce n'est pas du temps perdu : c'est du temps déplacé.

> **À retenir.** Le cahier des charges répond à « quoi » et « quand est-ce fini ». Le GDD répond à « comment ça se joue ». Écrivez le premier avant le second.

---

## 41.7 — Gabarit complet de cahier des charges (à recopier)

Voici le gabarit complet. Recopiez-le dans un fichier `CAHIER-DES-CHARGES.md` à la racine de votre projet, et remplissez chaque champ. **Ne supprimez aucune rubrique** : si une rubrique ne s'applique pas, écrivez « sans objet » — c'est une décision, et elle doit être visible.

```markdown
# CAHIER DES CHARGES — <Titre du jeu>

Version : 1.0
Date : JJ/MM/AAAA
Auteur :

---

## 1. Présentation

### 1.1 Pitch
<Titre> est un <genre> dans lequel le joueur <action> pour <objectif>,
tout en <contrainte>.

### 1.2 Contexte
En une phrase : pourquoi ce projet existe (projet de formation, jeu de
game jam, portfolio, commande).

### 1.3 Objectif du projet
Ce que le projet doit atteindre pour être considéré comme réussi.

---

## 2. Identité du produit

| Rubrique | Décision |
| --- | --- |
| Titre | |
| Genre | |
| Nombre de joueurs | |
| Public cible | |
| Plateformes visées | |
| Orientation de l'écran | |
| Résolution de référence | |
| Durée d'une partie | |
| Durée de vie totale visée | |
| Modèle de distribution | gratuit / payant / démonstration |
| Langue(s) | |

---

## 3. Exigences fonctionnelles

Chaque exigence a un identifiant, une priorité et une formulation vérifiable.
Priorités : O = obligatoire, S = souhaitable, F = facultatif.

| ID | Exigence | Priorité |
| --- | --- | --- |
| EF-01 | | O |
| EF-02 | | O |
| EF-03 | | S |
| ... | | |

---

## 4. Exigences non fonctionnelles

| ID | Exigence | Seuil mesurable |
| --- | --- | --- |
| ENF-01 | Performance | |
| ENF-02 | Taille du binaire | |
| ENF-03 | Temps de démarrage | |
| ENF-04 | Compatibilité | |
| ENF-05 | Accessibilité | |
| ENF-06 | Robustesse | |

---

## 5. Périmètre

### 5.1 Dans le lot
-
-

### 5.2 Hors du lot (explicitement refusé)
-
-

### 5.3 Reporté à une version ultérieure
-
-

---

## 6. Contraintes techniques

| Contrainte | Valeur |
| --- | --- |
| Langage | Dart |
| Framework | Flutter <version> |
| Moteur | Flame <version> |
| Paquets autorisés | |
| Version minimale d'Android | |
| Navigateurs supportés | |
| Taille maximale des assets | |

---

## 7. Livrables

| Livrable | Format | Échéance |
| --- | --- | --- |
| Code source | dépôt Git | |
| Build Android | .apk | |
| Build Web | dossier build/web | |
| Documentation | README.md | |
| GDD | GDD.md | |
| Tableau des licences | LICENCES.md | |

---

## 8. Planning et jalons

| Jalon | Contenu | Date cible |
| --- | --- | --- |
| J1 | | |
| J2 | | |
| J3 | | |
| J4 | | |

---

## 9. Critères d'acceptation

Le projet est considéré comme terminé si et seulement si :

1.
2.
3.
4.
5.

---

## 10. Risques identifiés

| Risque | Probabilité | Impact | Parade |
| --- | --- | --- | --- |
| | | | |
```

### Comment remplir sans se mentir

Trois règles de rédaction :

1. **Une exigence se vérifie.** « Le jeu doit être agréable » ne se vérifie pas. « Le joueur doit pouvoir terminer une partie en moins de cinq minutes » se vérifie au chronomètre.
2. **Une exigence facultative est vraiment facultative.** Si tout est marqué obligatoire, la colonne priorité ne sert à rien, et vous n'aurez rien à couper quand le temps manquera.
3. **La section « hors du lot » est la plus importante.** Si elle est vide, votre cahier des charges n'a rien tranché.

> **À retenir.** Un cahier des charges est un contrat avec vous-même. Il se lit en cinq minutes et se relit à chaque hésitation.

---

## 41.8 — Exigences fonctionnelles et non fonctionnelles

C'est la distinction la plus utile du chapitre, et celle que les débutants confondent le plus souvent.

**Une exigence fonctionnelle** décrit ce que le logiciel *fait*. Elle se formule toujours avec un sujet et un verbe d'action.

**Une exigence non fonctionnelle** décrit *comment* il le fait : à quelle vitesse, avec quelle fiabilité, sur quel matériel, dans quelle taille.

### Le test de reformulation

Posez la question : « Est-ce que je peux imaginer un utilisateur en train de faire cela ? »

- Oui → exigence fonctionnelle.
- Non, c'est une qualité du logiciel → exigence non fonctionnelle.

| Énoncé | Type | Pourquoi |
| --- | --- | --- |
| Le joueur peut sauter | Fonctionnelle | Une action de l'utilisateur |
| Le jeu tourne à 60 images par seconde | Non fonctionnelle | Une qualité, pas une action |
| Le joueur peut mettre le jeu en pause | Fonctionnelle | Une action |
| Le jeu démarre en moins de 3 secondes | Non fonctionnelle | Une performance |
| Le score le plus élevé est conservé | Fonctionnelle | Un comportement observable |
| L'APK pèse moins de 40 Mo | Non fonctionnelle | Une contrainte de livraison |
| Le jeu se joue au clavier et au tactile | Fonctionnelle | Deux modes d'entrée |
| Le jeu ne plante jamais sur perte de focus | Non fonctionnelle | Une robustesse |

### Comment écrire une exigence fonctionnelle

Utilisez le format suivant, systématiquement :

```text
EF-NN : <acteur> peut <action> [dans <contexte>] [afin de <but>].
```

Exemples pour un jeu de plateformes :

```markdown
| ID | Exigence | Priorité |
| --- | --- | --- |
| EF-01 | Le joueur peut déplacer le héros à gauche et à droite. | O |
| EF-02 | Le joueur peut faire sauter le héros depuis le sol. | O |
| EF-03 | Le joueur perd un point de vie au contact d'un ennemi. | O |
| EF-04 | Le joueur peut ramasser une pièce, qui augmente le score de 10. | O |
| EF-05 | Le joueur atteint la porte de sortie et passe au niveau suivant. | O |
| EF-06 | Le joueur peut mettre la partie en pause et la reprendre. | O |
| EF-07 | Le jeu affiche un écran de fin quand les trois vies sont perdues. | O |
| EF-08 | Le jeu conserve le meilleur score entre deux lancements. | S |
| EF-09 | Le joueur peut couper la musique depuis le menu des options. | S |
| EF-10 | Le joueur peut rejouer un niveau déjà terminé. | F |
```

### Comment écrire une exigence non fonctionnelle

Elle doit **toujours** comporter un nombre. Sans nombre, elle n'est pas vérifiable.

```markdown
| ID | Exigence | Seuil mesurable |
| --- | --- | --- |
| ENF-01 | Fluidité | 60 images/s sur un appareil Android de milieu de gamme |
| ENF-02 | Démarrage | écran de menu affiché en moins de 3 s |
| ENF-03 | Taille | APK inférieur à 40 Mo |
| ENF-04 | Compatibilité | Android 8.0 (API 26) minimum |
| ENF-05 | Web | fonctionne sur Chrome et Firefox à jour |
| ENF-06 | Robustesse | aucune exception non capturée en 10 parties consécutives |
| ENF-07 | Entrées | latence entre l'appui et la réaction inférieure à 100 ms |
| ENF-08 | Accessibilité | aucune information transmise par la seule couleur |
| ENF-09 | Mémoire | consommation stable après 10 minutes de jeu |
| ENF-10 | Hors-ligne | le jeu fonctionne sans connexion réseau |
```

### L'erreur classique

L'erreur la plus fréquente est d'écrire des exigences fonctionnelles qui décrivent l'implémentation :

| À ne pas écrire | À écrire |
| --- | --- |
| « Utiliser un `RectangleHitbox` pour le joueur » | « Le joueur perd de la vie au contact d'un ennemi » |
| « Créer une classe `AudioService` » | « Le joueur entend un son quand il ramasse une pièce » |
| « Stocker le score avec `SharedPreferences` » | « Le meilleur score est conservé après fermeture » |

Le cahier des charges décrit le **comportement observable**, jamais la solution technique. La solution technique appartient à l'architecture (section 41.23).

> **À retenir.** Fonctionnelle = ce que le joueur peut faire. Non fonctionnelle = avec quelle qualité mesurable. Une exigence sans verbe ou sans nombre est inutilisable.

---

## 41.9 — Le périmètre : dans le lot / hors du lot

Le périmètre est la partie du cahier des charges qui vous protégera le plus. Il s'écrit en trois colonnes ou en trois listes.

### Le tableau à trois colonnes

```markdown
| Dans le lot (v1.0) | Hors du lot (jamais) | Reporté (v1.1 ou plus) |
| --- | --- | --- |
| 3 niveaux | Multijoueur | 6 niveaux |
| 2 types d'ennemis | Monde ouvert | Un troisième ennemi |
| 1 boss | Génération procédurale | Un second boss |
| Score et meilleur score | Succès en ligne | Tableau des scores en ligne |
| Sons de base | Doublage vocal | Musique originale |
| Français | Localisation multilingue | Anglais |
| Android et Web | iOS, consoles | Bureau |
```

### Pourquoi trois colonnes et pas deux

La colonne « reporté » est un exutoire. Elle vous permet de dire oui à une idée sans la faire maintenant. Sans elle, les bonnes idées reviennent en boucle et finissent par entrer dans le projet par la porte de derrière.

### La règle de l'échange

Une fois le cahier des charges signé — même si c'est avec vous-même — le périmètre ne grossit plus. Il ne peut que **s'échanger** :

```text
   Vous voulez ajouter X ?
        │
        ├── Le planning a-t-il du mou ?
        │      │
        │      ├── OUI ──► ajouter X dans le lot, et documenter la date
        │      │
        │      └── NON ──► que retirez-vous du lot en échange ?
        │                        │
        │                        ├── Vous savez répondre ──► échange, et on note
        │                        │
        │                        └── Vous ne savez pas ──► X va en "reporté"
        │
        └── Fin. Aucun ajout sans contrepartie.
```

### Formuler un « hors du lot » utile

Un « hors du lot » vague ne protège de rien. Comparez :

| Formulation faible | Formulation forte |
| --- | --- |
| « Pas trop de contenu » | « Exactement 3 niveaux, pas 4 » |
| « Pas de gros système » | « Aucun inventaire : les objets sont consommés immédiatement » |
| « Graphismes simples » | « Aucun sprite : uniquement des formes géométriques colorées » |
| « Pas de sauvegarde compliquée » | « On sauvegarde uniquement le meilleur score, un entier » |
| « Peu de sons » | « 5 effets sonores maximum, 1 musique de fond » |

### Le périmètre du « Donjon de Dart »

| Dans le lot | Hors du lot | Reporté |
| --- | --- | --- |
| 3 niveaux écrits à la main | Multijoueur | Éditeur de niveaux |
| Joueur : marche, saut, attaque | Génération procédurale | Double saut |
| Gobelin et chauve-souris | Physique rigide (`flame_forge2d`) | Un ennemi volant à distance |
| 1 boss au niveau 3 | Dialogues et scénario | Plusieurs boss |
| Pièces, potions, clés | Inventaire complexe | Objets équipables |
| Score et meilleur score | Sauvegarde de position exacte | Reprise en cours de niveau |
| HUD : vie, score, vies | Localisation | Traduction anglaise |
| Formes géométriques colorées | Sprites achetés | Vrais sprites Kenney |

> **À retenir.** Le périmètre s'écrit une fois et ne grossit plus. Il s'échange. La colonne « reporté » est votre soupape.

---

## 41.10 — Qu'est-ce qu'un Game Design Document

Le **Game Design Document**, abrégé GDD, est le document qui décrit *comment le jeu se joue*. Il ne parle ni de classes, ni de widgets, ni de moteur. Il parle du joueur.

### Ce qu'un GDD contient

Un GDD répond, dans l'ordre, à ces questions :

1. Quel est le concept ? (une page)
2. Pour qui ? (un paragraphe)
3. Que fait le joueur en boucle ? (la boucle de gameplay)
4. Quelles sont les mécaniques ? (une liste chiffrée)
5. Comment la difficulté évolue-t-elle ?
6. Comment fonctionne l'économie du jeu ?
7. À quoi ça ressemble ?
8. À quoi ça ressemble sonorement ?
9. Quels écrans, quelles transitions ?
10. Quel contenu exactement ? (niveaux, ennemis, objets)

### Ce qu'un GDD ne contient pas

- du code ;
- des noms de classes ;
- l'arborescence des fichiers ;
- des considérations de performance ;
- le planning.

Tout cela appartient soit au cahier des charges, soit au document d'architecture. Si votre GDD contient le mot `PositionComponent`, vous avez mélangé deux documents.

### Le GDD est vivant

C'est la grande différence avec le cahier des charges. Le GDD **change**, et il doit changer, parce que le playtest révèle des choses. Une bonne pratique consiste à horodater chaque modification :

```markdown
## Journal des modifications

| Date | Section | Modification | Raison |
| --- | --- | --- | --- |
| 12/03 | 4. Mécaniques | Saut passé de 480 à 520 px/s | Les testeurs ratent la plateforme 3 |
| 15/03 | 6. Économie | Potion : +30 PV au lieu de +50 | Le jeu devenait trop facile |
| 19/03 | 8. Contenu | Suppression du niveau 4 | Manque de temps, jalon J3 en retard |
```

### La taille raisonnable

Pour un projet solo de quatre à huit semaines, visez **cinq à dix pages**. Un GDD de cinquante pages pour un jeu de quatre semaines est le signe le plus fiable d'un projet qui ne sortira jamais : le temps passé à écrire n'a pas été passé à jouer.

| Taille du projet | Taille du GDD | Temps de rédaction |
| --- | --- | --- |
| Game jam de 48 h | 1 page | 30 min |
| Projet de formation, 4 à 8 semaines | 5 à 10 pages | 2 à 3 h |
| Jeu commercial solo, 1 an | 20 à 40 pages | 2 semaines |
| Studio, 3 ans | Wiki d'équipe | continu |

> **À retenir.** Le GDD décrit l'expérience du joueur, pas l'implémentation. Il est court, chiffré, et il évolue à chaque playtest.

---

## 41.11 — Gabarit complet de GDD (à recopier), section par section

Voici le gabarit complet. Recopiez-le dans `GDD.md`, à la racine de votre projet, à côté du `CAHIER-DES-CHARGES.md`. Les sections 41.12 à 41.21 détaillent ensuite chaque rubrique.

```markdown
# GAME DESIGN DOCUMENT — <Titre du jeu>

Version : 0.1
Date : JJ/MM/AAAA
Auteur :

---

## 1. Concept

### 1.1 Pitch
<une phrase>

### 1.2 Description longue
<un paragraphe de 5 à 10 lignes>

### 1.3 Références
| Jeu de référence | Ce qu'on lui emprunte | Ce qu'on ne lui emprunte pas |
| --- | --- | --- |
| | | |

### 1.4 Ce qui rend ce jeu différent
<une phrase>

---

## 2. Public cible

| Rubrique | Réponse |
| --- | --- |
| Tranche d'âge | |
| Expérience du jeu vidéo | débutant / occasionnel / confirmé |
| Contexte de jeu | transports / bureau / canapé |
| Durée de session type | |
| Support principal | téléphone / navigateur / ordinateur |
| Ce que ce joueur déteste | |

---

## 3. Mécaniques

### 3.1 Mécanique principale
<une seule, décrite en trois lignes>

### 3.2 Mécaniques secondaires
| Nom | Description | Entrée du joueur | Effet |
| --- | --- | --- | --- |
| | | | |

### 3.3 Verbes du joueur
Liste exhaustive de ce que le joueur peut faire :
-

### 3.4 Ce que le joueur ne peut pas faire
-

---

## 4. Boucle de gameplay

### 4.1 Boucle courte (quelques secondes)
<schéma>

### 4.2 Boucle moyenne (une partie)
<schéma>

### 4.3 Boucle longue (plusieurs parties)
<schéma>

---

## 5. Courbe de difficulté

| Étape | Durée | Ce qui augmente | Valeur |
| --- | --- | --- | --- |
| | | | |

<schéma de la courbe>

---

## 6. Économie du jeu

### 6.1 Ressources
| Ressource | Comment on en gagne | Comment on en perd | Plafond |
| --- | --- | --- | --- |
| | | | |

### 6.2 Barème de points
| Action | Points |
| --- | --- |
| | |

### 6.3 Équilibrage
<calcul du score maximum théorique>

---

## 7. Direction artistique

| Rubrique | Décision |
| --- | --- |
| Style | |
| Palette | |
| Taille des sprites | |
| Ambiance | |
| Références visuelles | |

---

## 8. Son et musique

| Événement | Son | Durée | Source |
| --- | --- | --- | --- |
| | | | |

---

## 9. Interface et expérience utilisateur

### 9.1 Liste des écrans
| Écran | Contenu | Sorties possibles |
| --- | --- | --- |
| | | |

### 9.2 Schéma de navigation
<schéma>

### 9.3 HUD
| Information | Position | Format |
| --- | --- | --- |

### 9.4 Contrôles
| Plateforme | Action | Commande |
| --- | --- | --- |

---

## 10. Contenu

### 10.1 Niveaux
| N° | Nom | Thème | Nouveauté introduite | Durée visée |
| --- | --- | --- | --- | --- |

### 10.2 Ennemis
| Nom | PV | Dégâts | Vitesse | Comportement | Apparaît au niveau |
| --- | --- | --- | --- | --- | --- |

### 10.3 Objets
| Nom | Effet | Fréquence | Apparaît au niveau |
| --- | --- | --- | --- |

---

## 11. Journal des modifications

| Date | Section | Modification | Raison |
| --- | --- | --- | --- |
```

### Ordre de remplissage recommandé

N'écrivez pas ce document du haut vers le bas. Remplissez-le dans cet ordre :

```text
   1. Concept (1)            ──► la phrase qui fixe tout
   2. Mécaniques (3)         ──► ce que le joueur fait
   3. Boucle (4)             ──► comment ça s'enchaîne
   4. Contenu (10)           ──► combien de tout
   5. Économie (6)           ──► les chiffres
   6. Difficulté (5)         ──► la progression
   7. Interface (9)          ──► les écrans
   8. Public (2)             ──► la vérification
   9. Art (7) et Son (8)     ──► en dernier, toujours
```

L'art et le son viennent en dernier parce qu'ils sont les plus faciles à faire dériver, et parce qu'ils dépendent de tout le reste.

> **À retenir.** Un GDD se remplit du cœur vers la périphérie : mécanique, boucle, contenu, chiffres, puis seulement l'habillage.

---

## 41.12 — Le concept

Le concept est la première section du GDD. Elle contient le pitch (déjà écrit en 41.5), une description longue, des références et un facteur de différenciation.

### La description longue

Cinq à dix lignes, au présent, du point de vue du joueur. Elle raconte une partie type.

Exemple pour le « Donjon de Dart » :

```markdown
### 1.2 Description longue

Le joueur incarne un aventurier lâché dans la première salle d'un donjon.
Il court, saute de plateforme en plateforme et ramasse les pièces qu'il
croise. Des gobelins patrouillent au sol et des chauves-souris fondent
sur lui dès qu'il approche. Chaque contact lui coûte des points de vie ;
à zéro, il perd une vie sur les trois dont il dispose. Une clé est cachée
quelque part dans la salle : sans elle, la porte de sortie reste
verrouillée. Une fois la clé récupérée et la porte franchie, la salle
suivante s'ouvre, plus grande et plus peuplée. La troisième salle contient
un boss qui alterne charges et projectiles. Le joueur qui le vainc voit
son score final et son meilleur score.
```

### Les références

Les références servent à deux choses : gagner du temps de conception, et clarifier ce que vous ne voulez **pas** copier.

```markdown
### 1.3 Références

| Jeu de référence | Ce qu'on lui emprunte | Ce qu'on ne lui emprunte pas |
| --- | --- | --- |
| Super Mario Bros. | La sensation de saut, l'inertie légère | Les power-ups, les mondes |
| Spelunky | La salle unique et lisible d'un écran | La génération procédurale |
| Celeste | Le retour immédiat après la mort | La difficulté extrême, le dash |
```

La deuxième colonne est celle qui vous fait gagner du temps. La troisième est celle qui vous empêche de dériver.

### Le facteur de différenciation

Une phrase, qui commence par « Contrairement à… » ou « La particularité est… ».

```markdown
### 1.4 Ce qui rend ce jeu différent

Contrairement à un jeu de plateformes classique, chaque salle est un
espace fermé d'un seul écran : il n'y a pas de défilement, donc le joueur
voit toujours la totalité du problème à résoudre.
```

### L'erreur du concept

L'erreur la plus commune est de décrire l'univers au lieu du jeu.

| Ce qui est décrit | Verdict |
| --- | --- |
| « Un royaume déchiré par une guerre millénaire… » | Univers, pas concept |
| « Le joueur incarne un chevalier au passé tragique… » | Personnage, pas concept |
| « Le joueur saute de plateforme en plateforme pour… » | Concept |

Testez ainsi : si votre description peut s'appliquer à un roman, ce n'est pas un concept de jeu.

> **À retenir.** Le concept décrit une partie, pas un univers. Les références disent aussi ce que vous refusez de copier.

---

## 41.13 — Le public cible

Définir le public cible n'est pas un exercice marketing. C'est un outil de décision technique : il tranche les questions de contrôles, de durée, de difficulté et de plateforme.

### Le tableau du public cible

```markdown
| Rubrique | Réponse |
| --- | --- |
| Tranche d'âge | 12 à 40 ans |
| Expérience du jeu vidéo | occasionnel : a déjà joué à un jeu de plateformes |
| Contexte de jeu | transports en commun, pauses courtes |
| Durée de session type | 3 à 8 minutes |
| Support principal | téléphone Android, en paysage |
| Ce que ce joueur déteste | les tutoriels longs, la publicité, les chargements |
```

### Comment ce tableau décide à votre place

C'est là que se trouve son intérêt. Chaque ligne impose des conséquences.

| Ligne du tableau | Conséquence de conception |
| --- | --- |
| Session de 3 à 8 minutes | Une partie complète doit tenir en 5 minutes → 3 niveaux, pas 10 |
| Téléphone en paysage | Contrôles tactiles à deux pouces → boutons dans les coins bas |
| Joueur occasionnel | Difficulté douce au niveau 1, aucune mécanique cachée |
| Déteste les tutoriels | Apprentissage par le niveau 1 lui-même, pas par du texte |
| Transports en commun | Le jeu doit se mettre en pause quand l'application perd le focus |
| Déteste les chargements | Niveaux écrits en dur, pas de téléchargement |

### Le contre-exemple utile

Décrivez aussi le joueur pour qui votre jeu n'est **pas** fait. Cela protège de la dérive.

```markdown
### 2.2 Public non visé

Ce jeu n'est pas fait pour un joueur qui cherche :
- une expérience de plusieurs dizaines d'heures ;
- un scénario ou des personnages écrits ;
- un défi extrême de type "die and retry" ;
- une compétition en ligne.
```

### Le persona en trois lignes

Certains préfèrent la forme narrative. Elle fonctionne bien si elle reste courte.

```markdown
### 2.3 Persona

Léa, 22 ans, étudiante. Elle joue vingt minutes par jour, dans le bus,
sur un téléphone Android de milieu de gamme. Elle a fini Celeste sur
Switch mais joue surtout à des jeux courts sur mobile. Elle abandonne
un jeu qui lui demande de créer un compte ou qui met plus de cinq
secondes à démarrer.
```

> **À retenir.** Le public cible n'est pas une case à cocher : c'est une machine à trancher les décisions de contrôles, de durée et de difficulté.

---

## 41.14 — Les mécaniques principales

Une **mécanique** est une règle du jeu qui relie une entrée du joueur à un effet dans le monde. Ce n'est pas une fonctionnalité logicielle.

| Énoncé | Mécanique ? |
| --- | --- |
| « Le joueur saute plus haut s'il maintient le bouton » | Oui |
| « Le jeu a un menu principal » | Non, c'est une fonctionnalité |
| « Les ennemis touchés reculent » | Oui |
| « Le score est sauvegardé » | Non, c'est une fonctionnalité |
| « Ramasser une potion redonne 30 PV » | Oui |
| « Le jeu a un mode plein écran » | Non |

### La règle de la mécanique unique

Votre jeu a **une** mécanique principale. Une seule. Toutes les autres la servent.

```text
        ┌──────────────────────────────┐
        │   MÉCANIQUE PRINCIPALE       │
        │   Sauter avec précision      │
        └──────────────┬───────────────┘
                       │
        ┌──────────────┼──────────────┬───────────────┐
        ▼              ▼              ▼               ▼
   Plateformes    Gravité et      Ennemis qui    Trous mortels
   de tailles     inertie         bougent        qui punissent
   variées                                        l'erreur
```

Si vous ne savez pas laquelle est la principale, faites ce test : supprimez-en une. Celle dont la suppression détruit le jeu est la principale.

### La fiche de mécanique

Chaque mécanique secondaire mérite une fiche. Voici le gabarit :

```markdown
#### Mécanique : <nom>

- **Entrée du joueur :**
- **Condition d'activation :**
- **Effet immédiat :**
- **Effet différé :**
- **Retour au joueur (visuel / sonore) :**
- **Valeurs chiffrées :**
- **Ce que le joueur doit comprendre en la voyant une fois :**
```

Exemple rempli :

```markdown
#### Mécanique : le saut

- **Entrée du joueur :** touche Espace, ou bouton A tactile.
- **Condition d'activation :** le héros touche le sol (`auSol == true`).
- **Effet immédiat :** vitesse verticale mise à -520 px/s.
- **Effet différé :** la gravité (1200 px/s²) ramène le héros au sol
  en environ 0,87 s, pour une hauteur maximale de 112 px, soit 3,5 tuiles.
- **Retour au joueur :** animation de saut, son court, léger nuage de
  poussière à l'impulsion.
- **Valeurs chiffrées :** forceSaut = -520, gravite = 1200,
  vitesseMaxChute = 900.
- **Ce que le joueur doit comprendre :** je peux franchir un trou de
  trois tuiles, pas de cinq.
```

### La table des mécaniques secondaires

```markdown
| Nom | Description | Entrée du joueur | Effet |
| --- | --- | --- | --- |
| Marche | Déplacement horizontal constant | Flèches gauche/droite | 180 px/s |
| Saut | Impulsion verticale depuis le sol | Espace | -520 px/s |
| Attaque | Zone de dégâts devant le héros | Touche E | 25 dégâts, portée 24 px |
| Invincibilité | Immunité brève après un coup | automatique | 1,2 s, clignotement |
| Ramassage | Contact avec un collectible | automatique | score ou soin |
| Déverrouillage | Ouvre la porte si clé possédée | contact avec la porte | passage au niveau suivant |
```

### Les verbes du joueur

Listez de façon exhaustive ce que le joueur peut faire. Cette liste doit être **courte**. Si elle dépasse sept verbes, votre jeu est trop gros.

```markdown
### 3.3 Verbes du joueur
- courir
- sauter
- attaquer
- ramasser
- ouvrir

### 3.4 Ce que le joueur ne peut pas faire
- s'accroupir
- grimper
- lancer un projectile
- se soigner volontairement
- sauvegarder en cours de niveau
```

La seconde liste vaut la première. Elle est la version « game design » du hors-lot.

> **À retenir.** Une mécanique relie une entrée à un effet. Une seule est principale. Sept verbes maximum.

---

## 41.15 — La boucle de gameplay (core loop), avec schéma

La **boucle de gameplay**, ou *core loop*, est la séquence d'actions que le joueur répète. Un jeu tient tant que cette boucle est satisfaisante.

Attention à ne pas la confondre avec la boucle de jeu technique du chapitre 20 (`update` / `render` à 60 images par seconde). Ce sont deux notions différentes qui portent malheureusement le même nom.

| | Boucle technique (ch. 20) | Boucle de gameplay (ce chapitre) |
| --- | --- | --- |
| Fréquence | 60 fois par seconde | quelques secondes à quelques minutes |
| Acteur | le moteur | le joueur |
| Contenu | update, collisions, render | agir, obtenir, progresser |

### Les trois échelles de boucle

Un bon jeu a trois boucles imbriquées.

**La boucle courte** dure quelques secondes. C'est l'action de base.

```text
   ┌───────────────────────────────────────────────┐
   │                                               │
   │   observer l'obstacle                         │
   │           │                                   │
   │           ▼                                   │
   │   décider (sauter / attaquer / attendre)      │
   │           │                                   │
   │           ▼                                   │
   │   exécuter                                    │
   │           │                                   │
   │           ▼                                   │
   │   voir le résultat (réussite ou dégâts)  ─────┘
   │
   └── durée : 2 à 5 secondes
```

**La boucle moyenne** dure une partie. C'est la progression.

```text
   ┌────────────────────────────────────────────────────────────┐
   │                                                            │
   │   entrer dans la salle                                     │
   │        │                                                   │
   │        ▼                                                   │
   │   explorer et ramasser des pièces  ──► le score monte      │
   │        │                                                   │
   │        ▼                                                   │
   │   éviter ou tuer les ennemis  ──► perte de PV possible     │
   │        │                                                   │
   │        ▼                                                   │
   │   trouver la clé                                           │
   │        │                                                   │
   │        ▼                                                   │
   │   ouvrir la porte  ──► salle suivante, plus difficile  ────┘
   │        │
   │        └──► 3e salle : boss ──► victoire
   │
   │   (PV à 0 ──► perte d'une vie ──► redémarrage de la salle)
   │   (0 vie   ──► Game Over ──► menu)
   └── durée : 4 à 6 minutes
```

**La boucle longue** dépasse la partie. C'est la raison de revenir.

```text
   ┌──────────────────────────────────────────────────┐
   │                                                  │
   │   terminer ou perdre une partie                  │
   │        │                                         │
   │        ▼                                         │
   │   voir son score comparé au meilleur score       │
   │        │                                         │
   │        ▼                                         │
   │   se dire "je peux faire mieux"                  │
   │        │                                         │
   │        ▼                                         │
   │   relancer une partie  ──────────────────────────┘
   │
   └── durée : plusieurs sessions
```

### Le test de la boucle

Une boucle de gameplay est valide si elle passe ces quatre tests :

1. **Test de la fermeture** : la dernière étape ramène bien à la première.
2. **Test de la récompense** : au moins une étape donne quelque chose au joueur.
3. **Test du risque** : au moins une étape peut mal tourner.
4. **Test de la variation** : deux tours de boucle ne sont pas identiques.

Une boucle qui échoue au test 3 produit un jeu ennuyeux. Une boucle qui échoue au test 4 produit un jeu répétitif.

### Gabarit de boucle vierge

```text
   ┌─────────────────────────────────────────┐
   │                                         │
   │   1. <le joueur observe / reçoit>       │
   │            │                            │
   │            ▼                            │
   │   2. <le joueur décide>                 │
   │            │                            │
   │            ▼                            │
   │   3. <le joueur agit>                   │
   │            │                            │
   │            ▼                            │
   │   4. <le jeu répond : gain ou perte>    │
   │            │                            │
   │            ▼                            │
   │   5. <l'état change, la boucle reprend> ┘
   │
   └── durée d'un tour : ... s
       récompense : ...
       risque : ...
       variation : ...
```

> **À retenir.** Trois boucles, trois échelles : quelques secondes, une partie, plusieurs parties. Si l'une des trois manque, le jeu s'arrête là.

---

## 41.16 — La courbe de difficulté

La difficulté ne doit ni rester plate ni monter en ligne droite. Elle monte par paliers, avec des respirations.

### La forme cible

```text
   difficulté
      ▲
      │                                            ┌── boss
      │                                        ┌───┘
      │                              ┌─────────┘
      │                          ┌───┘   respiration
      │                ┌─────────┘
      │            ┌───┘   respiration
      │      ┌─────┘
      │  ┌───┘
      │──┘  apprentissage
      └──────────────────────────────────────────────► temps
         N1        N2            N3
```

Trois principes structurent cette courbe.

**Principe 1 — chaque palier introduit une seule nouveauté.** Un nouvel ennemi, ou une nouvelle disposition, ou une contrainte de temps. Jamais les trois en même temps.

**Principe 2 — la nouveauté est d'abord présentée sans danger.** Le joueur voit le nouvel élément dans une situation où il ne peut pas perdre, puis dans une situation où il peut perdre.

**Principe 3 — après un pic, on redescend.** Après un boss ou un passage difficile, offrez trente secondes faciles. Sans respiration, le joueur s'épuise.

### La table de progression

C'est le format le plus utile, parce qu'il est chiffré et directement traduisible en code.

```markdown
| Étape | Durée | Ce qui augmente | Valeurs |
| --- | --- | --- | --- |
| N1 début | 0-30 s | rien, apprentissage | 0 ennemi, plateformes larges (5 tuiles) |
| N1 milieu | 30-60 s | 1er ennemi au sol | 1 gobelin, vitesse 40 px/s |
| N1 fin | 60-90 s | trous | 2 trous de 3 tuiles |
| N2 début | 0-45 s | ennemi volant | 2 gobelins, 1 chauve-souris |
| N2 milieu | 45-90 s | plateformes étroites | largeur 2 tuiles |
| N2 fin | 90-120 s | densité | 3 gobelins, 2 chauves-souris |
| N3 début | 0-60 s | tout combiné | 3 gobelins, 3 chauves-souris, trous |
| N3 boss | 60-150 s | pic | boss 300 PV, 2 attaques |
| Fin | - | respiration | écran de victoire, score |
```

### Comment doser un palier

Trois leviers, à utiliser un seul à la fois :

| Levier | Exemple concret | Effet ressenti |
| --- | --- | --- |
| Quantité | 1 gobelin → 3 gobelins | Plus dense, peu de nouveauté |
| Vitesse | gobelin à 40 → 60 px/s | Moins de temps pour réagir |
| Variété | ajout de la chauve-souris | Nouvelle compétence exigée |
| Espace | plateforme de 5 → 2 tuiles | Précision exigée |
| Ressource | 3 vies → 2 vies | Enjeu augmenté, à manier avec prudence |

Le dernier levier, la réduction des ressources, est le plus brutal et le plus frustrant. Réservez-le au boss.

### Le piège de la difficulté du concepteur

Vous jouez à votre jeu depuis trois semaines. Vous connaissez chaque plateforme. Votre perception de la difficulté n'a plus aucune valeur. C'est pour cela que la section 41.32 sur le playtest est obligatoire, et non facultative.

Règle empirique : **votre niveau 1 est deux fois trop difficile**. Adoucissez-le avant même le premier test.

### Formaliser dans le code

La courbe de difficulté se traduit très bien en données, jamais en `if` éparpillés :

```dart
class ReglageNiveau {
  const ReglageNiveau({
    required this.nombreGobelins,
    required this.nombreChauvesouris,
    required this.vitesseEnnemis,
    required this.largeurPlateformeMin,
  });

  final int nombreGobelins;
  final int nombreChauvesouris;
  final double vitesseEnnemis;
  final int largeurPlateformeMin;
}

const List<ReglageNiveau> reglages = [
  ReglageNiveau(
    nombreGobelins: 1,
    nombreChauvesouris: 0,
    vitesseEnnemis: 40,
    largeurPlateformeMin: 5,
  ),
  ReglageNiveau(
    nombreGobelins: 2,
    nombreChauvesouris: 1,
    vitesseEnnemis: 50,
    largeurPlateformeMin: 3,
  ),
  ReglageNiveau(
    nombreGobelins: 3,
    nombreChauvesouris: 3,
    vitesseEnnemis: 60,
    largeurPlateformeMin: 2,
  ),
];
```

**Résultat :** régler la difficulté après un playtest devient une modification de trois nombres dans une liste, et non une chasse dans tout le code.

> **À retenir.** Une nouveauté par palier, une présentation sans danger, une respiration après chaque pic. Et votre niveau 1 est trop difficile.

---

## 41.17 — L'économie du jeu (points, ressources, récompenses)

L'économie d'un jeu, ce sont toutes les quantités qui entrent et sortent : points de vie, score, pièces, munitions, temps, clés. Même un jeu très simple en a une, et une économie mal réglée rend un bon jeu injouable.

### Les trois flux à décrire

Pour chaque ressource, décrivez trois choses : comment on en gagne, comment on en perd, et quelle est la limite.

```markdown
| Ressource | Gain | Perte | Plafond |
| --- | --- | --- | --- |
| Points de vie | potion +30 | ennemi -20, boss -35 | 100 |
| Vies | aucune | PV à 0 | 3 au départ, pas de gain |
| Score | pièce +10, gobelin +25, boss +500 | jamais | aucun |
| Clés | 1 par niveau | consommée par la porte | 1 |
| Temps | aucun | s'écoule | sans objet |
```

Deux règles s'imposent immédiatement à la lecture de ce tableau :

- **Une ressource sans plafond doit être une ressource sans usage stratégique.** Le score peut être illimité parce qu'il ne sert à rien dans la partie. Les points de vie ont un plafond parce qu'ils sont utiles.
- **Une ressource qu'on ne peut pas regagner crée une tension mais interdit la récupération.** Les vies, ici, ne se regagnent pas : c'est un choix, et il faut donc que trois vies suffisent.

### Le calcul du score maximum théorique

C'est un exercice que la plupart des débutants sautent, et c'est une erreur : il révèle immédiatement les déséquilibres.

```text
   Niveau 1 : 12 pièces × 10 =  120
              1 gobelin × 25 =   25
                            -------
                                145

   Niveau 2 : 18 pièces × 10 =  180
              2 gobelins × 25 =  50
              1 chauve-souris × 15 = 15
                            -------
                                245

   Niveau 3 : 24 pièces × 10 =  240
              3 gobelins × 25 =  75
              3 chauves-souris × 15 = 45
              1 boss × 500     =  500
                            -------
                                860

   TOTAL MAXIMUM THÉORIQUE : 1250
```

Ce calcul montre tout de suite un problème : le boss vaut 500 points, soit 40 % du total. Un joueur qui joue mal les trois niveaux mais bat le boss aura un meilleur score qu'un joueur méticuleux qui échoue au boss. Est-ce voulu ? Peut-être. Mais la décision est maintenant consciente.

### Les quatre types de récompense

| Type | Exemple | Effet sur le joueur |
| --- | --- | --- |
| Récompense de score | +10 points | Faible, mais cumulable |
| Récompense de ressource | +30 PV | Forte, immédiatement utile |
| Récompense de progression | une clé, une porte | Structurante, obligatoire |
| Récompense de statut | meilleur score battu | Durable, motive à rejouer |

Un jeu qui n'a qu'un seul type de récompense s'essouffle. Le « Donjon de Dart » utilise les quatre.

### Le rythme de récompense

Chiffrez l'intervalle entre deux récompenses. Une bonne règle empirique pour un jeu d'action court :

| Intervalle | Type de récompense |
| --- | --- |
| Toutes les 2 à 5 s | Une pièce, un petit son, un effet visuel |
| Toutes les 20 à 30 s | Une potion, un ennemi vaincu |
| Toutes les 90 à 150 s | Une clé, une porte, un niveau terminé |
| Une fois par partie | La victoire, le score final |

Si votre jeu passe plus de dix secondes sans donner quoi que ce soit au joueur, il paraîtra vide.

### Le tableau d'équilibrage

Ce gabarit vous servira après chaque playtest :

```markdown
| Valeur | Réglage initial | Après test 1 | Après test 2 | Justification |
| --- | --- | --- | --- | --- |
| PV du joueur | 100 | 100 | 120 | Trop de morts au niveau 3 |
| Dégâts du gobelin | 20 | 15 | 15 | 5 contacts au lieu de 4 |
| Soin de la potion | 50 | 30 | 30 | Le jeu devenait trivial |
| Vitesse du gobelin | 40 | 40 | 45 | Trop facile à éviter |
| Points par pièce | 10 | 10 | 10 | Inchangé |
| PV du boss | 200 | 300 | 250 | Combat trop long à 300 |
```

Notez la colonne « justification ». Sans elle, vous oublierez pourquoi vous avez changé une valeur, et vous la rechangerez dans l'autre sens un mois plus tard.

> **À retenir.** Toute ressource se décrit par gain, perte et plafond. Le score maximum théorique se calcule avant d'écrire le code. Chaque changement d'équilibrage se justifie par écrit.

---

## 41.18 — La direction artistique

La direction artistique d'un projet solo se juge à un seul critère : **est-elle tenable jusqu'au bout par la personne qui la porte ?**

### Les quatre options réalistes

| Option | Coût | Avantage | Inconvénient |
| --- | --- | --- | --- |
| Formes géométriques | Nul | Toujours cohérent, aucun blocage | Peu vendeur |
| Pack d'assets gratuit (Kenney) | Nul | Cohérent, professionnel, immédiat | Reconnaissable par d'autres développeurs |
| Pack payant sur itch.io | 5 à 30 € | Moins vu, souvent complet | Coût, licence à vérifier |
| Dessins maison | Très élevé | Unique | Lent, incohérent si vous débutez |

Pour un premier projet personnel, l'ordre de préférence est : formes géométriques, puis pack gratuit. Le dessin maison n'entre en jeu que si vous savez déjà dessiner et que vous acceptez d'y consacrer la moitié du temps du projet.

### La fiche de direction artistique

```markdown
## 7. Direction artistique

| Rubrique | Décision |
| --- | --- |
| Style | Pixel art 16×16, contours noirs |
| Origine des assets | Kenney "Pixel Platformer" (CC0) |
| Palette | 8 couleurs, voir ci-dessous |
| Taille de la tuile | 16 px, affichée en ×2 (zoom caméra 2.0) |
| Fond | Aplat sombre + 2 plans de parallaxe |
| Ambiance | Souterrain, éclairage froid, contrastes forts |
| Contraintes | Aucun élément de gameplay dans le décor |
```

### La palette, écrite en dur

Une palette limitée est le moyen le plus rapide d'obtenir une image cohérente sans savoir dessiner. Huit couleurs suffisent.

```dart
class Palette {
  // Fond
  static const Color fondSombre = Color(0xFF1A1423);
  static const Color fondMur = Color(0xFF3E3546);

  // Entités
  static const Color joueur = Color(0xFF4EA5D9);
  static const Color ennemi = Color(0xFFD94E4E);
  static const Color boss = Color(0xFF8B1E3F);

  // Objets
  static const Color piece = Color(0xFFF2C14E);
  static const Color potion = Color(0xFF63C132);
  static const Color cle = Color(0xFFE8E8E8);
}
```

**Résultat :** toutes les couleurs du jeu sont dans un seul fichier. Changer l'ambiance générale prend deux minutes, pas deux heures.

### La règle de lisibilité

C'est la seule règle artistique non négociable dans un jeu d'action :

> Le joueur doit pouvoir distinguer, en un dixième de seconde, ce qui le tue de ce qui l'aide et de ce qui ne fait rien.

Traduction concrète :

| Catégorie | Signal visuel obligatoire |
| --- | --- |
| Dangereux | Une teinte réservée (ici, le rouge) et rien d'autre |
| Utile | Une teinte réservée (ici, le vert et le jaune) |
| Décor | Faible contraste, désaturé, jamais rouge ni vert |
| Joueur | La couleur la plus lumineuse de l'écran |

Erreur classique : un décor rouge vif « parce que c'est joli ». Le joueur croit qu'il est dangereux, il l'évite, et le niveau ne se joue plus comme prévu.

### La contrainte d'accessibilité

Environ 8 % des hommes ont une forme de daltonisme. Ne transmettez jamais une information par la seule couleur.

| Information | Couleur seule | Couleur + forme |
| --- | --- | --- |
| Ennemi | carré rouge | carré rouge avec des pointes |
| Potion | carré vert | carré vert en forme de flacon |
| Plateforme mortelle | bande rouge | bande rouge avec des piques dessinés |

C'est aussi ce qu'exige l'exigence non fonctionnelle ENF-08 du gabarit de la section 41.8.

> **À retenir.** Choisissez la direction artistique la moins coûteuse qui reste lisible. Réservez une teinte au danger et une teinte à l'aide, et ne les employez jamais ailleurs.

---

## 41.19 — Le son et la musique

Le son est ce que les débutants coupent en premier, et c'est un tort : c'est la couche qui donne le plus de sensation de qualité pour le moins d'effort.

### Le tableau des sons

Listez chaque événement du jeu qui mérite un son. Cinq à huit suffisent pour un jeu court.

```markdown
| Événement | Son | Durée | Volume | Source |
| --- | --- | --- | --- | --- |
| Saut | jump.wav | 0,2 s | 0,6 | jsfxr, généré |
| Pièce ramassée | coin.wav | 0,3 s | 0,7 | Kenney Audio (CC0) |
| Potion bue | heal.wav | 0,5 s | 0,7 | jsfxr, généré |
| Dégâts subis | hurt.wav | 0,4 s | 0,8 | jsfxr, généré |
| Ennemi vaincu | kill.wav | 0,4 s | 0,7 | Kenney Audio (CC0) |
| Porte ouverte | door.wav | 0,8 s | 0,6 | freesound.org, CC0 |
| Game Over | gameover.wav | 1,5 s | 0,8 | jsfxr, généré |
| Musique de fond | donjon.ogg | boucle 90 s | 0,35 | incompetech.com, CC-BY |
```

### Les quatre règles du son de jeu

**Règle 1 — un son par action du joueur.** Chaque appui de touche qui produit un effet doit produire un son. C'est ce qui donne la sensation de contrôle.

**Règle 2 — jamais deux sons identiques simultanés.** Dix pièces ramassées en une seconde produisent une bouillie. Limitez à un son du même type toutes les 80 ms.

**Règle 3 — la musique est deux à trois fois moins forte que les effets.** Un volume de 0,3 à 0,4 pour la musique et 0,6 à 0,8 pour les effets est un bon point de départ.

**Règle 4 — le joueur doit pouvoir tout couper.** Un bouton « son » dans le menu, et le réglage est conservé. C'est une exigence, pas une option.

### Où trouver des sons sans savoir en faire

| Source | Type | Licence | Remarque |
| --- | --- | --- | --- |
| jsfxr / sfxr (web) | Effets rétro générés | Vous êtes l'auteur | Idéal, aucune licence à gérer |
| Kenney Audio | Packs d'effets | CC0 | Qualité constante, très large |
| freesound.org | Enregistrements | Variable (CC0, CC-BY) | Vérifier chaque fichier |
| incompetech.com | Musique | CC-BY | Attribution obligatoire |
| OpenGameArt | Effets et musique | Variable | Vérifier chaque fichier |

L'option **jsfxr** mérite une mention particulière : vous cliquez sur « Pickup/Coin », vous obtenez un son, vous le téléchargez. Vous en êtes l'auteur, donc aucune licence à documenter. Pour un premier projet, c'est la solution la plus économique en temps.

### Le format

| Usage | Format | Raison |
| --- | --- | --- |
| Effets courts | `.wav` ou `.ogg` | Latence minimale |
| Musique en boucle | `.ogg` | Compression correcte, boucle propre |
| À éviter | `.mp3` | Silence d'amorce qui casse les boucles |

### Le repli sans aucun son

Si vous n'avez aucun fichier audio, votre jeu doit quand même tourner. La bonne pratique consiste à encapsuler l'audio dans un service qui absorbe l'absence de fichier :

```dart
class AudioService {
  bool actif = true;

  Future<void> jouer(String nomFichier) async {
    if (!actif) return;
    try {
      await FlameAudio.play(nomFichier);
    } catch (_) {
      // Fichier absent : le jeu continue sans son.
      // Voir le chapitre 13 pour la gestion des exceptions.
    }
  }
}
```

**Résultat :** le jeu se lance et se joue même si le dossier `assets/audio/` est vide. Aucun blocage possible sur les assets.

> **À retenir.** Cinq à huit sons suffisent. Générez-les vous-même avec jsfxr pour éviter toute question de licence, et rendez le jeu jouable sans aucun fichier audio.

---

## 41.20 — L'interface et l'expérience utilisateur

Cette section du GDD liste les écrans et les transitions. Elle est courte mais elle évite des refontes coûteuses.

### La liste des écrans

```markdown
| Écran | Contenu | Sorties possibles |
| --- | --- | --- |
| Menu principal | Titre, Jouer, Options, Quitter | Jeu, Options, fermeture |
| Chargement | Barre ou texte | Jeu |
| Jeu | Le monde, le HUD | Pause, Game Over, Victoire |
| Pause | Reprendre, Options, Menu | Jeu, Options, Menu |
| Options | Son, musique, debug | écran appelant |
| Game Over | Score, Rejouer, Menu | Jeu, Menu |
| Victoire | Score, meilleur score, Menu | Menu |
```

### Le schéma de navigation

```text
                   ┌──────────────────┐
                   │  MENU PRINCIPAL  │◄──────────────────┐
                   └────┬──────┬──────┘                   │
                        │      │                          │
              "Jouer"   │      │  "Options"                │
                        ▼      ▼                          │
                 ┌───────────┐ ┌─────────┐                 │
                 │CHARGEMENT │ │ OPTIONS │                 │
                 └─────┬─────┘ └────┬────┘                 │
                       │            │ retour               │
                       ▼            └──────────────────────┤
                 ┌───────────┐                             │
          ┌─────►│    JEU    │                             │
          │      └──┬────┬───┘                             │
   reprendre        │    │                                 │
          │  échap  │    │  PV = 0 et vies = 0             │
          │         ▼    │                                 │
      ┌───┴────┐         ▼                                 │
      │ PAUSE  │   ┌───────────┐    "Menu"                 │
      └───┬────┘   │ GAME OVER ├───────────────────────────┤
          │        └─────┬─────┘                           │
          │ "Menu"       │ "Rejouer"                       │
          └──────────────┼─────────────► JEU               │
                         │                                 │
                 boss vaincu                               │
                         ▼                                 │
                   ┌───────────┐        "Menu"             │
                   │ VICTOIRE  ├───────────────────────────┘
                   └───────────┘
```

Dessinez ce schéma **avant** de coder vos overlays. Chaque flèche est un bouton à écrire, et chaque écran sans flèche sortante est un piège dans lequel le joueur restera coincé.

### Le HUD

```markdown
| Information | Position | Format | Mise à jour |
| --- | --- | --- | --- |
| Points de vie | haut gauche | barre verte, 120×12 px | à chaque dégât |
| Vies restantes | haut gauche, sous la barre | 3 carrés | à chaque mort |
| Score | haut droite | "SCORE 0240", chiffres alignés | à chaque gain |
| Clé possédée | haut droite, sous le score | icône grisée ou allumée | au ramassage |
| Niveau | haut centre | "SALLE 2/3" | au changement |
```

Deux règles de HUD :

1. **Le HUD ne bouge jamais.** Un compteur qui change de largeur quand le score passe de 99 à 100 attire l'œil au mauvais moment. Formatez sur un nombre fixe de chiffres.
2. **Le HUD n'occupe pas la zone de jeu.** Réservez les coins. Le centre appartient à l'action.

### Les contrôles

```markdown
| Plateforme | Action | Commande |
| --- | --- | --- |
| Bureau / Web | Gauche | Flèche gauche ou Q |
| Bureau / Web | Droite | Flèche droite ou D |
| Bureau / Web | Saut | Espace ou flèche haut |
| Bureau / Web | Attaque | E |
| Bureau / Web | Pause | Échap |
| Android | Déplacement | Joystick virtuel, bas gauche |
| Android | Saut | Bouton A, bas droite |
| Android | Attaque | Bouton B, bas droite |
| Android | Pause | Bouton en haut à droite |
```

Prévoyez les deux jeux de commandes dès le GDD. Ajouter le tactile après coup oblige à réorganiser l'écran, donc à refaire le HUD.

### Les trois secondes de la première partie

Ce que le joueur doit comprendre sans lire une ligne de texte :

| Temps | Ce qu'il doit comprendre | Comment |
| --- | --- | --- |
| 0 s | Quel élément est moi | La couleur la plus vive, au centre-gauche |
| 1 s | Je peux bouger | Les commandes répondent immédiatement |
| 3 s | Où je dois aller | La porte est visible dès le départ |
| 10 s | Ce qui me tue | Un premier ennemi, dans une situation sûre |
| 20 s | Ce qui me récompense | Une pièce facile à ramasser |

> **À retenir.** Dessinez le graphe des écrans avant de coder. Tout écran sans sortie est un bug. Le HUD reste dans les coins et ne change jamais de taille.

---

## 41.21 — Le contenu : niveaux, ennemis, objets

C'est la section la plus utile du GDD, parce qu'elle chiffre le travail restant. Un GDD sans table de contenu ne permet aucune estimation.

### La table des niveaux

```markdown
| N° | Nom | Thème | Nouveauté introduite | Durée visée | Statut |
| --- | --- | --- | --- | --- | --- |
| 1 | Le vestibule | Pierre grise | Marche, saut, pièces | 90 s | fait |
| 2 | Les oubliettes | Pierre humide | Chauve-souris, plateformes étroites | 120 s | fait |
| 3 | La salle du trône | Rouge sombre | Boss | 150 s | en cours |
```

Deux colonnes méritent votre attention. **« Nouveauté introduite »** garantit qu'aucun niveau n'est un simple remplissage. **« Statut »** transforme le GDD en outil de suivi.

### La table des ennemis

```markdown
| Nom | PV | Dégâts | Vitesse | Comportement | Niveau | Points |
| --- | --- | --- | --- | --- | --- | --- |
| Gobelin | 30 | 20 | 45 px/s | Patrouille entre deux bords | 1, 2, 3 | 25 |
| Chauve-souris | 15 | 15 | 70 px/s | Poursuite si distance < 150 px | 2, 3 | 15 |
| Boss | 250 | 35 | 60 px/s | Charge, puis 3 projectiles | 3 | 500 |
```

Chaque ligne de ce tableau représente environ deux à quatre heures de travail : classe, comportement, hitbox, réglage, test. Trois lignes, c'est déjà une à deux journées. Dix lignes, c'est une semaine et demie. Le tableau chiffre donc directement le planning.

### La table des objets

```markdown
| Nom | Effet | Fréquence | Niveau | Points |
| --- | --- | --- | --- | --- |
| Pièce | +10 au score | 12 à 24 par niveau | 1, 2, 3 | 10 |
| Potion | +30 PV, plafonné à 100 | 1 à 2 par niveau | 1, 2, 3 | 0 |
| Clé | Déverrouille la porte | 1 par niveau | 1, 2, 3 | 0 |
```

### La règle du contenu chiffré

Ne rédigez jamais « quelques ennemis », « plusieurs niveaux » ou « des objets à ramasser ». Écrivez des nombres. Un GDD non chiffré ne permet ni de planifier, ni de savoir quand s'arrêter.

| Formulation à bannir | Formulation à écrire |
| --- | --- |
| « plusieurs types d'ennemis » | « 2 types d'ennemis + 1 boss » |
| « des niveaux variés » | « 3 niveaux, 90 s / 120 s / 150 s » |
| « beaucoup de pièces » | « 12 pièces au N1, 18 au N2, 24 au N3 » |
| « un boss costaud » | « boss 250 PV, 2 attaques, 35 dégâts » |

### Le budget en heures

Une fois le contenu chiffré, vous pouvez estimer. Voici des ordres de grandeur réalistes pour quelqu'un qui vient de terminer la PARTIE 2C.

| Élément | Temps estimé |
| --- | --- |
| Un type d'ennemi simple (patrouille) | 2 à 3 h |
| Un type d'ennemi avec poursuite | 3 à 4 h |
| Un boss avec 2 attaques | 6 à 10 h |
| Un niveau conçu, placé et testé | 2 à 4 h |
| Un objet collectible | 1 h |
| Le HUD complet | 3 à 5 h |
| Le menu et les écrans de fin | 4 à 6 h |
| L'audio complet | 2 à 4 h |
| La sauvegarde du meilleur score | 1 à 2 h |
| Le build Android et Web | 3 à 6 h (première fois) |

Additionnez pour votre propre contenu. Si le total dépasse le nombre d'heures dont vous disposez, retirez du contenu **maintenant**, pas dans trois semaines.

> **À retenir.** Le contenu se chiffre. Chaque ligne de vos tables se traduit en heures, et la somme se compare au temps réellement disponible.

---

## 41.22 — Exemple : le GDD rempli du « Donjon de Dart »

Voici un GDD complet, celui du jeu que vous venez de construire. Il sert de modèle de longueur, de ton et de niveau de précision. Recopiez sa forme, pas son contenu.

```markdown
# GAME DESIGN DOCUMENT — Donjon de Dart

Version : 1.3
Date : 08/08/2026
Auteur : équipe pédagogique

---

## 1. Concept

### 1.1 Pitch
Donjon de Dart est un jeu de plateformes 2D dans lequel le joueur traverse
trois salles de donjon pour trouver la clé et atteindre la sortie, tout en
évitant des gobelins et des chauves-souris avec trois vies seulement.

### 1.2 Description longue
Le joueur incarne un aventurier lâché dans la première salle d'un donjon.
Il court, saute de plateforme en plateforme et ramasse les pièces qu'il
croise. Des gobelins patrouillent au sol et des chauves-souris fondent sur
lui dès qu'il approche. Chaque contact lui coûte des points de vie ; à zéro,
il perd une vie sur les trois dont il dispose. Une clé est cachée quelque
part dans la salle : sans elle, la porte de sortie reste verrouillée. Une
fois la clé récupérée et la porte franchie, la salle suivante s'ouvre, plus
grande et plus peuplée. La troisième salle contient un boss qui alterne
charges et projectiles. Le joueur qui le vainc voit son score final comparé
à son meilleur score, conservé d'une session à l'autre.

### 1.3 Références
| Jeu de référence | Ce qu'on lui emprunte | Ce qu'on ne lui emprunte pas |
| --- | --- | --- |
| Super Mario Bros. | La sensation de saut, l'inertie légère | Power-ups, mondes multiples |
| Spelunky | La salle lisible d'un seul écran | La génération procédurale |
| Celeste | Le redémarrage instantané après la mort | La difficulté extrême |

### 1.4 Ce qui rend ce jeu différent
Chaque salle tient sur un espace fermé de taille modeste : le joueur voit
presque toujours la totalité du problème à résoudre.

---

## 2. Public cible

| Rubrique | Réponse |
| --- | --- |
| Tranche d'âge | 12 à 40 ans |
| Expérience | occasionnel, a déjà joué à un jeu de plateformes |
| Contexte | apprentissage, démonstration, sessions courtes |
| Durée de session | 5 à 10 minutes |
| Support principal | navigateur et Android, en paysage |
| Ce que ce joueur déteste | les tutoriels longs, les chargements |

### 2.2 Public non visé
Ce jeu n'est pas fait pour un joueur qui cherche un scénario, une
expérience longue, une difficulté extrême ou une compétition en ligne.

---

## 3. Mécaniques

### 3.1 Mécanique principale
Sauter avec précision entre des plateformes tout en gérant la position des
ennemis. Tout le reste — pièces, clé, porte, boss — sert à donner une raison
et un rythme à ce saut.

### 3.2 Mécaniques secondaires
| Nom | Description | Entrée | Effet |
| --- | --- | --- | --- |
| Marche | Déplacement horizontal | Flèches / joystick | 180 px/s |
| Saut | Impulsion depuis le sol | Espace / bouton A | -520 px/s |
| Attaque | Zone de dégâts devant le héros | E / bouton B | 25 dégâts, portée 24 px |
| Invincibilité | Immunité brève après un coup | automatique | 1,2 s |
| Ramassage | Contact avec un collectible | automatique | score ou soin |
| Déverrouillage | Ouvre la porte si la clé est possédée | contact | niveau suivant |

### 3.3 Verbes du joueur
courir, sauter, attaquer, ramasser, ouvrir

### 3.4 Ce que le joueur ne peut pas faire
s'accroupir, grimper, double-sauter, tirer, se soigner volontairement,
sauvegarder en cours de niveau

---

## 4. Boucle de gameplay

### 4.1 Boucle courte (2 à 5 s)
observer → décider → sauter ou attaquer → constater le résultat → observer

### 4.2 Boucle moyenne (une partie, 5 à 8 min)
entrer dans la salle → ramasser les pièces → éviter ou tuer les ennemis →
trouver la clé → ouvrir la porte → salle suivante → boss → victoire
(PV à 0 → perte d'une vie → redémarrage de la salle ; 0 vie → Game Over)

### 4.3 Boucle longue (plusieurs parties)
finir ou perdre → comparer au meilleur score → relancer pour faire mieux

---

## 5. Courbe de difficulté

| Étape | Durée | Ce qui augmente | Valeurs |
| --- | --- | --- | --- |
| N1 début | 0-30 s | rien | 0 ennemi, plateformes de 5 tuiles |
| N1 milieu | 30-60 s | 1er ennemi | 1 gobelin à 45 px/s |
| N1 fin | 60-90 s | trous | 2 trous de 3 tuiles |
| N2 début | 0-45 s | ennemi volant | 2 gobelins, 1 chauve-souris |
| N2 milieu | 45-90 s | précision | plateformes de 3 tuiles |
| N2 fin | 90-120 s | densité | 3 gobelins, 2 chauves-souris |
| N3 | 0-60 s | tout combiné | 3 gobelins, 3 chauves-souris |
| N3 boss | 60-150 s | pic | boss 250 PV, 2 attaques |

---

## 6. Économie du jeu

### 6.1 Ressources
| Ressource | Gain | Perte | Plafond |
| --- | --- | --- | --- |
| Points de vie | potion +30 | ennemi -20, boss -35 | 100 |
| Vies | aucun | PV à 0 | 3, pas de gain |
| Score | pièce +10, gobelin +25, chauve-souris +15, boss +500 | jamais | aucun |
| Clé | 1 par salle | consommée par la porte | 1 |

### 6.2 Barème de points
| Action | Points |
| --- | --- |
| Pièce ramassée | 10 |
| Gobelin vaincu | 25 |
| Chauve-souris vaincue | 15 |
| Boss vaincu | 500 |
| Salle terminée | 100 |

### 6.3 Équilibrage
Score maximum théorique : 1550.
- N1 : 12 pièces (120) + 1 gobelin (25) + salle (100) = 245
- N2 : 18 pièces (180) + 2 gobelins (50) + 1 chauve-souris (15) + salle (100) = 345
- N3 : 24 pièces (240) + 3 gobelins (75) + 3 chauves-souris (45) + boss (500) + salle (100) = 960

Le boss représente 32 % du score total : c'est assumé, il doit rester
l'objectif principal.

---

## 7. Direction artistique

| Rubrique | Décision |
| --- | --- |
| Style | Formes géométriques colorées, aucun sprite |
| Palette | 8 couleurs, `lib/config/palette.dart` |
| Taille de la tuile | 32 px, zoom caméra 2.0 |
| Fond | Aplat sombre uni |
| Ambiance | Souterrain, contrastes forts |
| Règle | Rouge = danger, vert et jaune = utile, gris = décor |

---

## 8. Son et musique

| Événement | Son | Durée | Volume | Source |
| --- | --- | --- | --- | --- |
| Saut | jump.wav | 0,2 s | 0,6 | jsfxr |
| Pièce | coin.wav | 0,3 s | 0,7 | jsfxr |
| Potion | heal.wav | 0,5 s | 0,7 | jsfxr |
| Dégâts | hurt.wav | 0,4 s | 0,8 | jsfxr |
| Ennemi vaincu | kill.wav | 0,4 s | 0,7 | jsfxr |
| Porte | door.wav | 0,8 s | 0,6 | jsfxr |
| Game Over | gameover.wav | 1,5 s | 0,8 | jsfxr |
| Musique | donjon.ogg | boucle | 0,35 | à définir |

Le jeu doit rester jouable si le dossier audio est vide.

---

## 9. Interface et expérience utilisateur

### 9.1 Écrans
menu principal, chargement, jeu, pause, game over, victoire

### 9.2 HUD
| Information | Position | Format |
| --- | --- | --- |
| PV | haut gauche | barre 120×12 px |
| Vies | haut gauche | 3 carrés |
| Score | haut droite | "SCORE 0000" |
| Clé | haut droite | icône grisée ou allumée |
| Salle | haut centre | "SALLE 1/3" |

### 9.3 Contrôles
| Plateforme | Gauche/Droite | Saut | Attaque | Pause |
| --- | --- | --- | --- | --- |
| Bureau/Web | flèches ou Q/D | Espace | E | Échap |
| Android | joystick | bouton A | bouton B | bouton haut droite |

---

## 10. Contenu

### 10.1 Niveaux
| N° | Nom | Nouveauté | Durée |
| --- | --- | --- | --- |
| 1 | Le vestibule | marche, saut, pièces, gobelin | 90 s |
| 2 | Les oubliettes | chauve-souris, plateformes étroites | 120 s |
| 3 | La salle du trône | boss | 150 s |

### 10.2 Ennemis
| Nom | PV | Dégâts | Vitesse | Comportement | Niveaux |
| --- | --- | --- | --- | --- | --- |
| Gobelin | 30 | 20 | 45 px/s | patrouille | 1, 2, 3 |
| Chauve-souris | 15 | 15 | 70 px/s | poursuite < 150 px | 2, 3 |
| Boss | 250 | 35 | 60 px/s | charge + 3 projectiles | 3 |

### 10.3 Objets
| Nom | Effet | Fréquence |
| --- | --- | --- |
| Pièce | +10 score | 12 / 18 / 24 |
| Potion | +30 PV | 1 à 2 par salle |
| Clé | ouvre la porte | 1 par salle |

---

## 11. Journal des modifications

| Date | Section | Modification | Raison |
| --- | --- | --- | --- |
| 02/08 | 6 | Potion : +50 → +30 PV | Jeu trop facile |
| 04/08 | 10.2 | Boss : 300 → 250 PV | Combat trop long |
| 06/08 | 5 | Gobelin : 40 → 45 px/s | Trop facile à éviter |
| 08/08 | 6.2 | Ajout "salle terminée : 100" | Récompenser la progression |
```

### Ce qu'il faut observer dans cet exemple

| Observation | Pourquoi c'est important |
| --- | --- |
| Aucun nom de classe Dart | Le GDD décrit le jeu, pas le code |
| Toutes les quantités sont des nombres | Le contenu est chiffrable donc planifiable |
| La section 3.4 dit ce qu'on ne fait pas | Protège du glissement de périmètre |
| Le journal des modifications est daté | Chaque décision est traçable |
| Le calcul du score maximum est explicite | Le déséquilibre du boss a été vu et assumé |
| Le document tient en 6 pages | Il se relit en dix minutes |

> **À retenir.** Un GDD complet pour un projet de six semaines tient en six pages, ne cite aucune classe, et chiffre absolument tout.

---

## 41.23 — L'architecture technique : le schéma des couches

Le GDD est terminé. On passe au troisième document : l'architecture. Il tient en une page et en un schéma.

Le principe est celui du chapitre 26 : **une couche ne connaît que la couche du dessous**. Une violation de cette règle est la cause principale du code de jeu impossible à modifier.

```text
   ┌─────────────────────────────────────────────────────────────┐
   │  COUCHE 4 — PRÉSENTATION (Flutter)                          │
   │  menus, pause, game over, options, HUD en widgets           │
   │  Connaît : la couche 3 (lit l'état, appelle des méthodes)   │
   └───────────────────────────┬─────────────────────────────────┘
                               │
   ┌───────────────────────────▼─────────────────────────────────┐
   │  COUCHE 3 — JEU (FlameGame)                                 │
   │  la classe de jeu, la machine à états, le score, les vies    │
   │  le chargement des niveaux, l'orchestration                  │
   │  Connaît : les couches 2 et 1                                │
   └───────────────────────────┬─────────────────────────────────┘
                               │
   ┌───────────────────────────▼─────────────────────────────────┐
   │  COUCHE 2 — COMPOSANTS (Flame)                              │
   │  joueur, ennemis, collectibles, plateformes, projectiles     │
   │  Connaît : la couche 1 et le jeu par HasGameReference        │
   └───────────────────────────┬─────────────────────────────────┘
                               │
   ┌───────────────────────────▼─────────────────────────────────┐
   │  COUCHE 1 — NOYAU ET DONNÉES (Dart pur)                      │
   │  constantes, palette, enums, mixins, cartes de niveaux       │
   │  Connaît : rien. Testable sans Flutter.                      │
   └─────────────────────────────────────────────────────────────┘

   Services transverses (audio, sauvegarde) : appelés depuis la couche 3
   uniquement, jamais depuis un composant.
```

### Les quatre règles de l'architecture

| Règle | Conséquence pratique |
| --- | --- |
| Une couche ne connaît que celles du dessous | Un `import` qui remonte est une erreur de conception |
| La couche 1 ne dépend pas de Flutter | Elle se teste avec `dart test`, sans émulateur |
| Un widget n'appelle jamais un composant | Il passe par la classe de jeu |
| Un composant n'appelle jamais un service | Il passe par `game`, qui possède les services |

### Le test de l'import

Ouvrez n'importe quel fichier de la couche 1 ou 2. Si vous y lisez `import 'package:flutter/material.dart';`, il y a de fortes chances que la séparation soit rompue. Un composant Flame n'a besoin que de `package:flame` et de `dart:ui`.

### Le fichier `ARCHITECTURE.md`

Écrivez ce document, court, à la racine du projet :

```markdown
# ARCHITECTURE — <Titre>

## Couches
<le schéma ci-dessus, adapté>

## Règles de dépendance
1.
2.

## Points d'entrée
| Fichier | Rôle |
| --- | --- |
| lib/main.dart | runApp, crée l'instance de jeu |
| lib/<nom>_game.dart | la classe FlameGame |

## Décisions techniques
| Décision | Alternative écartée | Raison |
| --- | --- | --- |
| Physique écrite à la main | flame_forge2d | Pas de corps rigide nécessaire |
| Niveaux en List<String> | Tiled | Aucun outil externe à installer |
| SharedPreferences | fichier JSON | Un seul entier à stocker |
```

La table des décisions techniques est celle que vous relirez le plus. Elle vous évite de refaire trois fois le même arbitrage.

> **À retenir.** Quatre couches, une seule direction de dépendance. La couche 1 doit se tester sans Flutter.

---

## 41.24 — Choisir sa structure de dossiers

Il n'existe pas une structure universelle, mais trois structures raisonnables. Choisissez-en une et n'en changez plus.

### Structure A — par type technique (recommandée pour un premier jeu)

```text
lib/
├── main.dart
├── mon_jeu.dart
├── config/          constantes, palette
├── core/            enums, mixins, classes de base
├── composants/      joueur, ennemis, objets
├── niveaux/         données et chargement
├── hud/             éléments d'affichage dans le jeu
├── services/        audio, sauvegarde
└── ecrans/          widgets Flutter
```

C'est celle du « Donjon de Dart ». Avantage : on sait toujours où chercher. Inconvénient : au-delà de trente composants, le dossier `composants/` devient long.

### Structure B — par fonctionnalité

```text
lib/
├── main.dart
├── mon_jeu.dart
├── commun/
├── joueur/          joueur.dart, joueur_animations.dart, joueur_etats.dart
├── ennemis/         ennemi.dart, gobelin.dart, chauvesouris.dart, boss.dart
├── collecte/        collectible.dart, piece.dart, potion.dart, cle.dart
├── monde/           niveau.dart, plateforme.dart, porte.dart
└── ui/              hud/, ecrans/
```

Avantage : tout ce qui concerne une fonctionnalité est au même endroit. Inconvénient : les fichiers communs sont plus difficiles à placer.

### Structure C — plate (jeux de game jam)

```text
lib/
├── main.dart
├── mon_jeu.dart
├── joueur.dart
├── ennemi.dart
├── niveau.dart
└── hud.dart
```

Acceptable jusqu'à une dizaine de fichiers, au-delà elle devient ingérable. Utilisez-la pour un projet de 48 heures, jamais pour un projet de six semaines.

### Comment choisir

| Situation | Structure |
| --- | --- |
| Premier jeu personnel, 4 à 8 semaines | A |
| Plus de 30 composants prévus | B |
| Game jam de moins de 3 jours | C |
| Travail à plusieurs, une personne par fonctionnalité | B |

### Les dossiers hors de `lib/`

```text
mon_jeu/
├── assets/
│   ├── images/
│   └── audio/
├── docs/
│   ├── CAHIER-DES-CHARGES.md
│   ├── GDD.md
│   ├── ARCHITECTURE.md
│   ├── LICENCES.md
│   └── playtests/
│       ├── 2026-08-12-test-01.md
│       └── 2026-08-19-test-02.md
├── test/
└── lib/
```

Le dossier `docs/` n'est pas décoratif : c'est là que vivent les quatre documents de ce chapitre, versionnés avec le code.

> **À retenir.** Choisissez A pour un premier jeu, et ne réorganisez jamais l'arborescence en cours de projet : c'est du temps qui ne produit rien de jouable.

---

## 41.25 — Les conventions de nommage

Les conventions de nommage sont vues au chapitre 16. Voici le rappel appliqué au jeu, plus les règles spécifiques.

| Élément | Convention | Exemple |
| --- | --- | --- |
| Fichier Dart | `snake_case.dart` | `chauve_souris.dart` |
| Classe | `PascalCase` | `ChauveSouris` |
| Variable, méthode | `lowerCamelCase` | `vitesseMax`, `subirDegats()` |
| Constante | `lowerCamelCase` avec `static const` | `Constantes.gravite` |
| Enum | `PascalCase`, valeurs en `lowerCamelCase` | `EtatJoueur.marche` |
| Membre privé | préfixe `_` | `_tempsInvincible` |
| Dossier | `snake_case` | `composants/` |

### Les règles propres au jeu

**Règle 1 — une langue, une seule.** Le « Donjon de Dart » est écrit en français : `Joueur`, `subirDegats`, `pointsScore`. Un mélange (`Player.subirDegats()`) est la pire option. Décidez, notez la décision dans `ARCHITECTURE.md`, et tenez-la.

**Règle 2 — les noms d'états au participe ou à l'infinitif, jamais les deux.**

```dart
// Cohérent
enum EtatJoueur { immobile, marche, saut, chute, attaque, touche, mort }

// Incohérent : mélange de noms et de verbes conjugués
enum EtatJoueur { immobile, ilMarche, sauter, enTrainDeTomber }
```

**Règle 3 — le nom dit ce que c'est, pas comment c'est fait.**

| À éviter | À préférer |
| --- | --- |
| `RectangleRouge` | `Gobelin` |
| `TimerInvincibilite` | `dureeInvincibilite` |
| `ListeDeTrucs` | `collectibles` |
| `gererTout()` | `mettreAJourIA()` |

**Règle 4 — les booléens commencent par `est`, `a` ou `peut`.**

```dart
bool estVivant;
bool auSol;
bool aLaCle;
bool peutSauter;
```

**Règle 5 — les méthodes commencent par un verbe à l'infinitif.**

```dart
void sauter();
void attaquer();
void subirDegats(double degats);
void chargerNiveau(int index);
```

### Le nom du projet lui-même

Le nom du paquet, dans `pubspec.yaml`, suit les règles de Dart : minuscules, chiffres et tirets bas uniquement.

```yaml
name: donjon_de_dart
description: Jeu de plateformes 2D réalisé avec Flutter et Flame.
version: 1.0.0
```

Un nom invalide (`Donjon-de-Dart`, `DonjonDeDart`) fait échouer `flutter create` ou `pub get`. C'est la première erreur de la journée pour beaucoup.

> **À retenir.** Une langue, des verbes à l'infinitif pour les méthodes, `est`/`a`/`peut` pour les booléens, et des noms qui décrivent le rôle et non la forme.

---

## 41.26 — La gestion des assets : nommage, tailles, licences

Les assets sont la première cause de blocage d'un projet amateur. Trois décisions les neutralisent.

### Décision 1 — le nommage

Adoptez un schéma unique, appliqué sans exception :

```text
<categorie>_<nom>_<variante>_<index>.<ext>
```

| Fichier | Lecture |
| --- | --- |
| `perso_joueur_marche_01.png` | personnage, joueur, animation de marche, image 1 |
| `perso_gobelin_idle_01.png` | personnage, gobelin, immobile, image 1 |
| `objet_piece_01.png` | objet, pièce, image 1 |
| `tuile_mur_pierre.png` | tuile, mur, variante pierre |
| `ui_bouton_jouer.png` | interface, bouton jouer |
| `sfx_saut.wav` | effet sonore, saut |
| `bgm_donjon.ogg` | musique de fond, donjon |

Trois interdits absolus, qui cassent le build Android ou Web :

- pas d'espace : `mon sprite.png` est un piège ;
- pas de majuscule : Windows ne distingue pas la casse, Android si ;
- pas d'accent ni de caractère spécial : `épée.png` échouera un jour.

### Décision 2 — les tailles

Choisissez **une** taille de tuile pour tout le projet, et n'en dérogez plus.

| Taille de tuile | Convient à | Zoom caméra typique |
| --- | --- | --- |
| 8 px | Pixel art minimaliste | 4,0 à 6,0 |
| 16 px | Pixel art classique | 2,0 à 4,0 |
| 32 px | Pixel art lisible sur mobile | 1,5 à 2,0 |
| 64 px | Style illustré | 1,0 |

Règle complémentaire : **toutes les images sont des puissances de deux** (16, 32, 64, 128, 256). Les cartes graphiques anciennes et certains navigateurs le préfèrent, et cela évite les artefacts de mise à l'échelle.

Autre règle : pour du pixel art, désactivez le lissage. Sans cela, tout paraît flou.

```dart
// Dans onLoad de votre FlameGame
@override
Future<void> onLoad() async {
  // Rendu net pour le pixel art
  images.prefix = 'assets/images/';
  await super.onLoad();
}
```

### Décision 3 — la source, et donc la licence

| Source | Licence habituelle | Attribution | Usage commercial |
| --- | --- | --- | --- |
| Kenney.nl | CC0 | non requise | autorisé |
| OpenGameArt | variable | selon le fichier | selon le fichier |
| itch.io (packs gratuits) | variable | souvent requise | souvent restreint |
| freesound.org | CC0 ou CC-BY | selon le fichier | selon le fichier |
| Généré par vous (jsfxr) | vous êtes l'auteur | sans objet | autorisé |
| Trouvé par recherche d'images | inconnue | interdit d'usage | interdit |

La dernière ligne est la plus importante : **une image trouvée sur un moteur de recherche n'a pas de licence utilisable**. Elle est protégée par défaut. L'utiliser dans un jeu publié vous expose à un retrait, voire à une réclamation.

### Le budget de poids

| Type d'asset | Poids cible unitaire | Total cible |
| --- | --- | --- |
| Sprite 32×32 en PNG | < 3 Ko | — |
| Atlas 512×512 | < 120 Ko | — |
| Effet sonore | < 40 Ko | 5 à 8 fichiers |
| Musique en `.ogg` | < 1,5 Mo | 1 fichier |
| **Total du dossier `assets/`** | | **< 8 Mo** |

Au-delà de 8 Mo d'assets, le build Web devient lent à charger et l'APK dépasse rapidement les seuils confortables.

### La déclaration dans `pubspec.yaml`

```yaml
flutter:
  assets:
    - assets/images/
    - assets/audio/
```

Attention au piège classique : déclarer un dossier ne prend **pas** ses sous-dossiers. `assets/images/` n'inclut pas `assets/images/ennemis/`. Il faut déclarer chaque sous-dossier explicitement.

> **À retenir.** Un schéma de nommage sans espace, sans majuscule et sans accent ; une seule taille de tuile ; une source dont vous connaissez la licence. Et jamais d'image trouvée par recherche.

---

## 41.27 — Le tableau de suivi des licences (obligatoire pour publier)

Ce tableau n'est pas une formalité. C'est le document qui vous permet de publier sans risque, et de répondre en trente secondes si on vous demande d'où vient tel élément.

### Le gabarit

Créez `docs/LICENCES.md` et remplissez-le **au moment où vous téléchargez le fichier**, pas à la fin. À la fin, vous ne vous souviendrez plus.

```markdown
# LICENCES ET ATTRIBUTIONS — <Titre du jeu>

Dernière mise à jour : JJ/MM/AAAA

## Code

| Élément | Auteur | Licence | Lien |
| --- | --- | --- | --- |
| Flutter | Google | BSD-3-Clause | https://flutter.dev |
| Flame | Blue Fire | MIT | https://pub.dev/packages/flame |
| flame_audio | Blue Fire | MIT | https://pub.dev/packages/flame_audio |
| shared_preferences | Flutter team | BSD-3-Clause | https://pub.dev/packages/shared_preferences |
| Code du jeu | <votre nom> | <votre licence> | — |

## Images

| Fichier | Auteur | Licence | Attribution requise | Lien | Date |
| --- | --- | --- | --- | --- | --- |
| perso_joueur_*.png | Kenney | CC0 | non | https://kenney.nl/assets/pixel-platformer | 12/08/2026 |
| tuile_*.png | Kenney | CC0 | non | https://kenney.nl/assets/pixel-platformer | 12/08/2026 |
| ui_bouton_*.png | <votre nom> | — | — | création personnelle | 14/08/2026 |

## Sons

| Fichier | Auteur | Licence | Attribution requise | Lien | Date |
| --- | --- | --- | --- | --- | --- |
| sfx_*.wav | <votre nom> | — | — | généré avec jsfxr | 13/08/2026 |
| bgm_donjon.ogg | Kevin MacLeod | CC-BY 4.0 | OUI | https://incompetech.com | 15/08/2026 |

## Polices

| Fichier | Auteur | Licence | Lien |
| --- | --- | --- | --- |
| PressStart2P.ttf | CodeMan38 | SIL OFL 1.1 | https://fonts.google.com |

## Mentions à afficher dans le jeu

Écran « Crédits », texte exact à afficher :

    Musique : "Dungeon Theme" par Kevin MacLeod (incompetech.com)
    Licence Creative Commons BY 4.0
    Police : Press Start 2P par CodeMan38, SIL Open Font License 1.1
    Graphismes : Kenney.nl (CC0)
```

### Les licences que vous rencontrerez

| Licence | Attribution | Usage commercial | Modification | Piège |
| --- | --- | --- | --- | --- |
| CC0 | non | oui | oui | aucun, la plus simple |
| CC-BY | **oui** | oui | oui | l'attribution doit être visible |
| CC-BY-SA | oui | oui | oui | votre œuvre dérivée doit être sous la même licence |
| CC-BY-NC | oui | **non** | oui | interdit dès que le jeu est payant ou monétisé |
| CC-BY-ND | oui | oui | **non** | interdit de recolorer ou redécouper |
| MIT | oui (dans le code) | oui | oui | il faut inclure le texte de la licence |
| Licence propriétaire | selon le contrat | selon | selon | à lire intégralement |

Deux licences méritent une vigilance particulière : **CC-BY-NC** interdit toute monétisation, même une publicité ; **CC-BY-ND** interdit toute modification, y compris un redimensionnement.

### Le réflexe à prendre

Trois lignes ajoutées à `LICENCES.md` **au moment du téléchargement**. Cela prend vingt secondes. Reconstituer l'origine de quarante fichiers trois mois plus tard prend une journée, et échoue souvent.

### Le cas de la publication

Avant de mettre le jeu sur itch.io, le Play Store ou un portfolio, vérifiez :

1. tous les fichiers de `assets/` figurent dans `LICENCES.md` ;
2. aucune ligne n'a « inconnue » en licence ;
3. toutes les attributions requises apparaissent dans l'écran des crédits du jeu ;
4. aucune licence NC si le jeu est monétisé ;
5. les licences des paquets Dart sont mentionnées (`flutter pub deps` vous les liste).

> **À retenir.** Le tableau des licences se remplit au téléchargement, pas à la livraison. Une ligne « licence inconnue » interdit la publication.

---

## 41.28 — Le planning : découper en jalons

Un jalon n'est pas une date : c'est **un état vérifiable du projet**. La différence est capitale. « Semaine 3 » n'est pas un jalon. « Le joueur peut terminer un niveau complet » en est un.

### Les propriétés d'un bon jalon

| Propriété | Explication |
| --- | --- |
| Vérifiable | On peut répondre oui ou non, sans discussion |
| Jouable | À chaque jalon, le jeu se lance et se joue |
| Autonome | Il ne dépend pas d'un jalon futur |
| Petit | Une à deux semaines maximum |
| Ordonné | Chaque jalon rend le suivant possible |

### Le planning type d'un projet de six semaines

```markdown
| Jalon | Nom | Contenu vérifiable | Durée |
| --- | --- | --- | --- |
| J0 | Documents | Pitch, cahier des charges, GDD, architecture écrits | 3 j |
| J1 | Ça bouge | Une fenêtre, un carré qui se déplace au clavier | 3 j |
| J2 | Ça joue | Boucle complète : jouer, gagner, perdre, recommencer | 7 j |
| J3 | Ça a du contenu | Tous les niveaux, tous les ennemis, tous les objets | 10 j |
| J4 | Ça s'équilibre | 3 playtests menés, valeurs corrigées | 5 j |
| J5 | C'est beau | Assets, sons, effets, polish | 7 j |
| J6 | C'est livré | Build Android et Web, README, licences, page de jeu | 5 j |
```

Total : 40 jours de travail. À dix heures par semaine, cela fait environ dix semaines calendaires. Ce calcul est le vrai résultat du planning : il vous dit si votre périmètre tient dans votre temps disponible.

### Le schéma des jalons

```text
   J0        J1        J2         J3            J4        J5        J6
   ●─────────●─────────●──────────●─────────────●─────────●─────────●
   docs   ça bouge  ÇA JOUE    contenu     équilibrage  polish   livraison
                       ▲
                       │
             Point de non-retour :
             si J2 n'est pas atteint, le projet
             doit être réduit, pas poursuivi.
```

### La règle du J2

Le jalon J2, « ça joue », est le point de contrôle du projet. S'il n'est pas atteint dans le tiers du temps prévu, **le projet est trop gros**. La réaction correcte est de couper du contenu, pas de travailler plus vite.

| Temps écoulé | J2 atteint ? | Action |
| --- | --- | --- |
| 1/3 du temps | oui | continuer |
| 1/3 du temps | non | supprimer un niveau et un type d'ennemi |
| 1/2 du temps | non | réduire au périmètre minimal jouable |
| 2/3 du temps | non | livrer ce qui existe, arrêter les ajouts |

### La marge

Réservez **20 %** du temps total pour les imprévus. Ils arriveront : un bug de build Android, un problème de licence, une mécanique qui ne fonctionne pas au playtest. Un planning sans marge est un planning faux.

### Le suivi hebdomadaire

Trois lignes par semaine suffisent, dans `docs/JOURNAL.md` :

```markdown
## Semaine 3 (17-23/08)

- Fait : chauve-souris, poursuite, niveau 2 placé.
- Pas fait : le boss (reporté S4), les sons (reportés J5).
- Décision : le niveau 4 est supprimé du périmètre, J3 était en retard de 2 j.
```

> **À retenir.** Un jalon est un état vérifiable et jouable, pas une date. Si J2 « ça joue » n'est pas atteint au tiers du temps, coupez du contenu immédiatement.

---

## 41.29 — Le jalon « ça tourne » avant le jalon « c'est beau »

C'est le principe le plus important du chapitre, et le plus souvent violé.

### L'ordre correct

```text
   1. ÇA TOURNE      le jeu se lance, ne plante pas
   2. ÇA JOUE        la boucle complète fonctionne
   3. ÇA S'ÉQUILIBRE les chiffres sont réglés par le playtest
   4. C'EST BEAU     assets, sons, effets, transitions
   5. C'EST LIVRÉ    builds, documentation, publication
```

### Pourquoi cet ordre

**Raison 1 — l'habillage d'un jeu qui n'est pas amusant ne le rend pas amusant.** Si la boucle ennuie, aucun sprite ne la sauvera. Il vaut mieux le découvrir en semaine 2 qu'en semaine 8.

**Raison 2 — l'habillage se refait à chaque changement de mécanique.** Si vous animez le héros avant d'avoir figé ses états, chaque nouvel état coûte une animation supplémentaire. En attendant l'étape 3, vous animez une fois.

**Raison 3 — l'habillage est du travail à volume élastique.** On peut mettre trois jours ou trois mois dans le polish. Le placer en dernier permet de s'arrêter quand le temps est écoulé, sans que le jeu soit incomplet.

### Ce qui appartient à « c'est beau » et doit attendre

| Élément | Étape correcte |
| --- | --- |
| Sprites et animations | 4 |
| Particules | 4 |
| Transitions entre écrans | 4 |
| Musique | 4 |
| Tremblement d'écran | 4 |
| Parallaxe de fond | 4 |
| Police personnalisée | 4 |
| Effets sonores | 3 ou 4 |

### L'exception à connaître

Il y a une exception, et une seule : **le retour immédiat aux actions du joueur**. Un son de saut, un léger recul de l'ennemi touché, un clignotement à la prise de dégâts. Ce n'est pas du polish, c'est de la lisibilité — sans cela, le playtest de l'étape 3 ne donne aucune information exploitable, parce que le testeur ne comprend pas ce qui se passe.

| Effet | Polish ou lisibilité ? |
| --- | --- |
| Clignotement d'invincibilité | Lisibilité, à faire tôt |
| Recul de l'ennemi touché | Lisibilité, à faire tôt |
| Explosion de particules à la mort | Polish, à faire tard |
| Son de dégâts | Lisibilité, à faire tôt |
| Musique d'ambiance | Polish, à faire tard |
| Tremblement d'écran au coup de boss | Polish, à faire tard |

### La question de contrôle

À chaque fois que vous hésitez à travailler sur quelque chose, posez cette question :

> Si je retire ceci, le joueur peut-il toujours finir une partie et comprendre ce qu'il fait ?

- **Oui** → c'est du polish, notez-le et passez à autre chose.
- **Non** → c'est du gameplay ou de la lisibilité, faites-le maintenant.

> **À retenir.** L'habillage est élastique, la boucle ne l'est pas. Faites tourner, faites jouer, équilibrez, puis seulement embellissez.

---

## 41.30 — Git et les jeux : `.gitignore`, gros fichiers, branches

Le chapitre 16 a présenté Git et le `.gitignore`. Un projet de jeu ajoute trois difficultés : les fichiers générés par Flutter, les assets binaires, et les fichiers de build volumineux.

### Le `.gitignore` d'un projet de jeu Flutter

```text
# Dart / Flutter
.dart_tool/
.packages
.pub-cache/
.pub/
build/
pubspec.lock          # à garder pour une application, à ignorer pour un paquet

# IDE
.idea/
.vscode/
*.iml
*.iws

# Système
.DS_Store
Thumbs.db

# Android
android/.gradle/
android/local.properties
android/key.properties
*.jks
*.keystore

# iOS
ios/Pods/
ios/.symlinks/

# Web
web/.dart_tool/

# Fichiers de travail des assets (sources, pas les exports)
assets_sources/*.aseprite
assets_sources/*.psd
assets_sources/*.xcf

# Sorties de build
*.apk
*.aab
*.ipa
```

Deux lignes méritent une explication.

**`build/`** est le dossier le plus lourd du projet. Ne le versionnez jamais : il se régénère et il pèse des centaines de mégaoctets.

**`android/key.properties` et `*.jks`** contiennent la clé de signature de votre application Android. Les versionner sur un dépôt public revient à publier votre mot de passe de publication. C'est une erreur irréversible : la clé doit alors être considérée comme compromise.

### Les gros fichiers

Git n'est pas fait pour les fichiers binaires volumineux. Chaque version d'une image de 3 Mo est stockée intégralement, pas en différentiel. Un dépôt de jeu peut ainsi atteindre plusieurs gigaoctets en quelques mois.

| Situation | Solution |
| --- | --- |
| Assets < 8 Mo au total | Versionner normalement, aucun problème |
| Assets de 8 à 100 Mo | Versionner, mais ne pas modifier les fichiers souvent |
| Fichiers sources (.psd, .aseprite) | Les garder hors du dépôt, ou dans un dépôt séparé |
| Assets > 100 Mo | Git LFS, ou un stockage externe documenté dans le README |

Règle simple pour un projet de formation : **gardez les assets exportés sous 8 Mo, et laissez les fichiers sources en dehors du dépôt.**

### Les branches

Pour un projet solo, deux branches suffisent :

```text
   main ──●──────●──────────●──────────●──────► version qui compile toujours
           \    /            \        /
            ●──●              ●──●──●          branches de fonctionnalité
          feat/joueur       feat/boss
```

| Branche | Règle |
| --- | --- |
| `main` | Compile et se joue à tout moment. On n'y pousse jamais de code cassé. |
| `feat/<nom>` | Une fonctionnalité en cours. Fusionnée dans `main` quand elle marche. |

### Les messages de commit

Utilisez un préfixe. Cela rend l'historique lisible et permet de générer les notes de version.

```text
feat: ajout de la chauve-souris et de sa poursuite
fix: le joueur traversait la plateforme à grande vitesse
balance: dégâts du gobelin passés de 20 à 15
art: remplacement des rectangles par les sprites Kenney
docs: mise à jour du GDD après le playtest 2
chore: mise à jour de flame vers 1.38.0
```

### Le tag de jalon

À chaque jalon atteint, posez une étiquette. Cela vous permet de revenir à une version jouable si une refonte tourne mal.

```text
git tag -a j2-ca-joue -m "Boucle complete jouable"
git tag -a j3-contenu -m "3 niveaux, 2 ennemis, boss"
git push --tags
```

> **À retenir.** Ignorez `build/` et les clés de signature, gardez les assets sous 8 Mo, une branche par fonctionnalité, un tag par jalon.

---

## 41.31 — Le prototypage papier

Le prototypage papier consiste à tester une mécanique avec un crayon, du papier quadrillé et éventuellement des dés, avant d'écrire la moindre ligne de code. Cela paraît naïf. C'est pourtant la technique qui fait gagner le plus de temps sur un projet.

### Ce que le papier permet de tester

| Testable sur papier | Non testable sur papier |
| --- | --- |
| La disposition d'un niveau | La sensation du saut |
| L'ordre d'apparition des ennemis | La réactivité des commandes |
| L'économie : combien de pièces, combien de PV | Le rendu visuel en mouvement |
| Les règles de progression | Les performances |
| La courbe de difficulté | Le confort tactile |

Retenez la ligne clé : **le papier teste les règles, pas les sensations**. Le saut ne se teste qu'au clavier.

### La méthode, en quatre étapes

**Étape 1 — dessinez la grille.** Une feuille quadrillée, un carreau par tuile. Un niveau de 40 × 20 tuiles tient sur une page A4.

```text
   ################################
   #..............................#
   #..o..o................o..o....#
   #....====.........====.........#
   #..........g...............k...#
   #......====....====............#
   #..J......................g..D.#
   ################################
```

**Étape 2 — placez le contenu au crayon.** Murs, plateformes, pièces, ennemis, clé, porte. Le crayon se gomme, le code non.

**Étape 3 — simulez une partie avec le doigt.** Suivez le chemin du joueur case par case. Comptez : combien de sauts, combien de pièces sur le chemin, combien de rencontres d'ennemis, en combien de secondes ?

**Étape 4 — notez les problèmes.** Ils apparaissent immédiatement :

```markdown
| Problème constaté sur papier | Correction |
| --- | --- |
| La clé est sur le chemin direct vers la porte | La déplacer en haut à gauche |
| 6 pièces seulement sur le trajet, 18 placées | Réduire à 12 et les mettre sur le chemin |
| Le gobelin de droite n'est jamais rencontré | Le déplacer près de la porte |
| Le trou du milieu fait 5 tuiles, le saut en franchit 3 | Le ramener à 3 tuiles |
```

Ce dernier point mérite qu'on s'y arrête. Un trou infranchissable dans un niveau codé coûte : écrire le niveau, lancer le jeu, mourir dix fois, comprendre, modifier, relancer. Vingt minutes. Sur papier, il se voit en dix secondes, parce que vous savez que le saut couvre trois tuiles.

### La fiche de portée du joueur

Avant tout prototypage papier, calculez une fois pour toutes les capacités du héros, et écrivez-les sur un post-it collé à l'écran.

```text
   PORTÉE DU HÉROS (Donjon de Dart)
   ────────────────────────────────
   hauteur de saut ......... 112 px = 3,5 tuiles de 32
   distance de saut ........ 158 px = 4,9 tuiles
   trou franchissable max .. 4 tuiles (marge de sécurité : 3)
   portée d'attaque ........ 24 px  = 0,75 tuile
   champ de vision caméra .. 20 × 12 tuiles au zoom 2,0
```

Ces cinq nombres suffisent à concevoir tous vos niveaux sans jamais lancer le jeu.

### Le prototypage papier pour un jeu de réflexion

Pour un jeu de puzzle ou de gestion, le papier n'est pas un raccourci : c'est le vrai prototype. Découpez des jetons, jouez dix parties à la main. Si le jeu est ennuyeux sur table, il le sera à l'écran, avec quatre semaines de travail en plus.

> **À retenir.** Le papier teste les règles et les niveaux, jamais les sensations. Cinq nombres de portée suffisent à concevoir vos niveaux avant de coder.

---

## 41.32 — Le playtest : comment faire tester et quoi observer

Le playtest est l'unique source d'information fiable sur votre jeu. Votre propre avis ne vaut rien : vous connaissez toutes les réponses.

### Les cinq règles absolues

**Règle 1 — ne dites rien.** Aucune explication, aucun conseil, aucune commande annoncée. Vous dites « voilà, joue » et vous vous taisez. Si le joueur ne comprend pas comment sauter, c'est une information capitale — et vous venez de la détruire en la lui disant.

**Règle 2 — ne vous justifiez pas.** « Oui, mais normalement... », « ça, c'est parce que... » : ces phrases vous protègent et vous privent de l'information. Notez, remerciez, corrigez plus tard.

**Règle 3 — observez les mains et le visage, pas l'écran.** Vous connaissez l'écran. Ce que vous ne connaissez pas, c'est le moment où le testeur fronce les sourcils, soupire ou repose l'appareil.

**Règle 4 — chronométrez.** Notez les temps. « Il a mis 40 secondes à comprendre qu'il fallait ramasser la clé » est exploitable ; « il a eu du mal » ne l'est pas.

**Règle 5 — testez avec au moins trois personnes différentes.** Un seul testeur peut être atypique. Trois font apparaître les vrais problèmes : ceux que tout le monde rencontre.

### Ce qu'il faut observer, dans l'ordre d'importance

| Ordre | Observation | Question qu'elle résout |
| --- | --- | --- |
| 1 | Combien de secondes avant la première action ? | Le jeu est-il compréhensible ? |
| 2 | Où le joueur s'arrête-t-il, perplexe ? | Où manque une information ? |
| 3 | À quel endroit meurt-il le plus ? | Où la difficulté est-elle mal dosée ? |
| 4 | Quand repose-t-il l'appareil ? | Où le jeu devient-il ennuyeux ? |
| 5 | Que fait-il que vous n'aviez pas prévu ? | Quelle mécanique émergente existe ? |
| 6 | Demande-t-il à rejouer ? | La boucle longue fonctionne-t-elle ? |
| 7 | Que dit-il spontanément ? | Rien de fiable, mais des pistes |

Notez bien la ligne 7. Ce que les testeurs **disent** est la donnée la moins fiable du playtest. Ce qu'ils **font** est la plus fiable. Un joueur qui dit « c'était bien » après avoir soupiré trois fois et abandonné au niveau 2 vous donne deux informations contradictoires ; croyez la seconde.

### Le déroulé d'une séance de 20 minutes

```text
   00:00  Vous : "Voici un jeu. Joue. Je ne peux pas répondre pendant que tu joues."
   00:00  Vous démarrez le chronomètre et vous vous taisez.
   00:00  ─────────── 10 à 12 minutes de jeu, en silence ───────────
   12:00  Vous : "Tu peux t'arrêter. Merci."
   12:00  Questions ouvertes, dans cet ordre :
            1. "Raconte-moi ce que tu as fait."
            2. "À quel moment tu ne savais pas quoi faire ?"
            3. "Qu'est-ce qui t'a agacé ?"
            4. "Tu rejouerais ?"
            5. "Si tu pouvais changer une seule chose, ce serait quoi ?"
   20:00  Fin. Vous remplissez la grille pendant que c'est frais.
```

Ne posez jamais de question fermée (« C'était bien ? »), ni de question orientée (« Le boss était trop dur, non ? »). Vous obtiendriez la réponse que vous voulez entendre.

### La règle de la répétition

Une remarque isolée est un avis. **La même remarque chez trois testeurs sur trois est un fait.** Ne corrigez que les faits ; notez les avis dans une liste d'attente.

| Fréquence | Statut | Action |
| --- | --- | --- |
| 1 testeur sur 5 | Avis individuel | Noter, ne rien faire |
| 2 testeurs sur 5 | Signal faible | Surveiller au prochain test |
| 3 testeurs sur 5 ou plus | Fait | Corriger |

### Le playtest à distance

Si vous ne pouvez pas être présent, demandez un enregistrement d'écran plus la voix. À défaut, une capture vidéo du jeu suffit : le chemin parcouru et les hésitations restent visibles.

> **À retenir.** Vous ne parlez pas, vous notez. Ce que les testeurs font compte plus que ce qu'ils disent. Trois testeurs qui butent au même endroit, c'est un fait, pas un avis.

---

## 41.33 — La grille de playtest (gabarit)

Voici la grille à remplir pour chaque séance. Créez un fichier par test dans `docs/playtests/`.

```markdown
# PLAYTEST — <Titre du jeu>

Test n° : 01
Date : JJ/MM/AAAA
Version testée : commit <hash> / tag <jalon>
Testeur : prénom ou initiales, âge, expérience du jeu vidéo
Support : Android / navigateur / bureau
Durée de la séance : ... min

---

## 1. Chronologie observée

| Temps | Événement observé | Interprétation |
| --- | --- | --- |
| 00:00 | | |
| 00:00 | | |

---

## 2. Indicateurs chiffrés

| Indicateur | Valeur |
| --- | --- |
| Temps avant la première action | ... s |
| Temps avant le premier saut | ... s |
| Temps pour comprendre l'objectif | ... s |
| Nombre de morts au niveau 1 | ... |
| Nombre de morts au niveau 2 | ... |
| Nombre de morts au niveau 3 | ... |
| Niveau le plus loin atteint | ... |
| Score final | ... |
| Durée totale de jeu | ... min |
| A demandé à rejouer | oui / non |

---

## 3. Points de blocage

| Lieu / moment | Ce qui s'est passé | Hypothèse de cause |
| --- | --- | --- |
| | | |

---

## 4. Comportements non prévus

| Ce que le testeur a fait | Était-ce prévu ? | À exploiter ? |
| --- | --- | --- |
| | | |

---

## 5. Verbatim (citations exactes)

- "..."
- "..."

---

## 6. Réponses aux questions finales

1. Ce que tu as fait :
2. Moment sans savoir quoi faire :
3. Ce qui t'a agacé :
4. Rejouerais-tu :
5. La seule chose à changer :

---

## 7. Décisions prises

| Problème | Fréquence (tests concernés) | Décision | Statut |
| --- | --- | --- | --- |
| | | corriger / surveiller / ignorer | à faire / fait |

---

## 8. Modifications d'équilibrage

| Valeur | Avant | Après | Raison |
| --- | --- | --- | --- |
| | | | |
```

### La synthèse multi-tests

Après trois séances, remplissez une synthèse. C'est elle qui décide des corrections.

```markdown
# SYNTHÈSE DES PLAYTESTS 01 à 03

| Problème | T01 | T02 | T03 | Fréquence | Décision |
| --- | --- | --- | --- | --- | --- |
| N'a pas vu la clé | oui | oui | oui | 3/3 | Corriger : clé plus lumineuse + son |
| Trouve le boss trop long | oui | non | oui | 2/3 | Surveiller au T04 |
| Voudrait un double saut | non | oui | non | 1/3 | Ignorer, hors du lot |
| Meurt 5+ fois au niveau 2 | oui | oui | oui | 3/3 | Corriger : élargir les plateformes |
```

> **À retenir.** Une grille par séance, une synthèse tous les trois tests. La colonne « fréquence » décide ; votre goût personnel ne décide pas.

---

## 41.34 — Les pièges classiques du débutant (tableau)

| # | Piège | Ce qui se passe | Comment l'éviter |
| --- | --- | --- | --- |
| 1 | Coder avant d'écrire | Le périmètre grossit sans limite, rien n'est jamais fini | Pitch, cahier des charges et GDD avant la première ligne |
| 2 | Commencer par le menu | Trois jours sur des boutons, zéro information sur le jeu | Commencer par la mécanique principale, menu au jalon J5 |
| 3 | Refondre l'architecture en cours de route | Deux semaines sans rien de jouable en plus | Figer l'architecture à J0, ne la changer qu'entre deux jalons |
| 4 | Attendre les « vrais » graphismes | Le projet s'arrête sur un blocage non technique | Formes géométriques jusqu'au jalon J5 |
| 5 | Ajouter une idée par jour | Le périmètre double en un mois | Règle de l'échange : rien n'entre sans que quelque chose sorte |
| 6 | Ne jamais faire tester | Le jeu est réglé pour une seule personne : vous | Trois playtests minimum avant le polish |
| 7 | Équilibrer d'après son propre ressenti | Le niveau 1 est deux fois trop difficile | Chiffrer, tester, corriger, journaliser |
| 8 | Dupliquer le code d'un ennemi | Cinq classes presque identiques, cinq bugs à corriger | Une classe abstraite `Ennemi`, comme au chapitre 37 |
| 9 | Semer des valeurs en dur dans le code | Modifier la gravité oblige à fouiller dix fichiers | Une classe `Constantes` unique |
| 10 | Négliger la sauvegarde des licences | Publication impossible, ou retrait après coup | `LICENCES.md` rempli au téléchargement |
| 11 | Versionner `build/` et la clé de signature | Dépôt de plusieurs gigaoctets, clé compromise | `.gitignore` de la section 41.30 |
| 12 | Tester uniquement sur ordinateur | Sur téléphone, les commandes sont inutilisables | Tester sur l'appareil cible dès le jalon J2 |
| 13 | Faire dépendre un widget d'un composant | Une modification de composant casse l'interface | Respecter les couches de la section 41.23 |
| 14 | Prévoir 100 % du temps disponible | Le moindre imprévu fait exploser le planning | Réserver 20 % de marge |
| 15 | Confondre polish et lisibilité | Le playtest ne donne rien d'exploitable | Retour immédiat tôt, particules tard |

> **À retenir.** Ces quinze pièges expliquent la quasi-totalité des projets abandonnés. Relisez ce tableau à chaque jalon.

---

## 41.35 — Trois sujets de projet final proposés, avec leur périmètre

Vous devez maintenant choisir un sujet. Les trois propositions ci-dessous sont calibrées pour **30 à 40 heures de travail**, avec ce que vous savez faire à la fin de la PARTIE 2C. Vous pouvez aussi proposer votre propre sujet, à condition de le calibrer de la même manière.

### Sujet A — « Chute Libre » (arcade, le plus simple)

**Pitch.** Chute Libre est un jeu d'arcade vertical dans lequel le joueur déplace un panier de gauche à droite pour rattraper des œufs qui tombent, tout en évitant des pierres dont la fréquence augmente avec le temps.

| Rubrique | Valeur |
| --- | --- |
| Genre | Arcade, score, une seule scène |
| Durée d'une partie | 60 à 180 s |
| Niveaux | 0 (difficulté continue) |
| Types d'objets | 3 : œuf (+10), œuf doré (+50), pierre (-1 vie) |
| Ennemis | 0 |
| Écrans | menu, jeu, game over |
| Mécaniques | déplacement horizontal, collision, accélération de la difficulté |
| Réutilisation de la 2C | collisions (ch. 32), HUD (ch. 38), sauvegarde du meilleur score (ch. 40) |
| Difficulté technique | faible |
| Estimation | 20 à 25 h |

**Hors du lot :** aucun niveau, aucun ennemi mobile, aucun boss, aucune sauvegarde autre que le meilleur score.

**Le vrai défi :** la courbe de difficulté. La fréquence et la vitesse de chute doivent augmenter assez pour que la partie se termine, mais assez lentement pour que le joueur ait le sentiment de progresser.

### Sujet B — « Les Gardiens du Pont » (défense, difficulté moyenne)

**Pitch.** Les Gardiens du Pont est un jeu de défense dans lequel le joueur place des tours sur les cases libres d'un pont pour arrêter des vagues d'ennemis, tout en gérant un budget d'or limité.

| Rubrique | Valeur |
| --- | --- |
| Genre | Défense de position, vagues |
| Durée d'une partie | 5 à 8 min |
| Vagues | 8, avec 3 paliers de difficulté |
| Types de tours | 2 : tour rapide (peu de dégâts), tour lourde (lente, forts dégâts) |
| Ennemis | 2 : marcheur (60 PV) et coureur (30 PV, vitesse ×2) |
| Écrans | menu, jeu, entre-vagues, game over, victoire |
| Mécaniques | placement au clic, budget, tir automatique, vagues chronométrées |
| Réutilisation de la 2C | composants (ch. 28), entrées tactiles (ch. 30), collisions (ch. 32), timers (ch. 33), HUD (ch. 38) |
| Difficulté technique | moyenne |
| Estimation | 30 à 35 h |

**Hors du lot :** amélioration des tours, chemins multiples, tours à effets (ralentissement, zone), génération de vagues aléatoire.

**Le vrai défi :** l'économie. Combien coûte une tour, combien rapporte un ennemi, combien d'or au départ. Le calcul du budget théorique de la section 41.17 est ici obligatoire, sinon le jeu est soit impossible, soit trivial.

### Sujet C — « Racine » (réflexion, le plus original)

**Pitch.** Racine est un jeu de réflexion sur grille dans lequel le joueur fait pousser une racine case par case pour atteindre une source d'eau, tout en gérant un nombre de segments limité et en contournant des rochers.

| Rubrique | Valeur |
| --- | --- |
| Genre | Puzzle, tour par tour, grille |
| Durée d'une partie | 10 à 20 min pour 12 niveaux |
| Niveaux | 12, courts, chacun avec une solution unique |
| Éléments de grille | 4 : terre, rocher, source, engrais (+3 segments) |
| Ennemis | 0 |
| Écrans | menu, sélection de niveau, jeu, victoire de niveau, victoire finale |
| Mécaniques | pousser dans 4 directions, compteur de segments, annulation du dernier coup |
| Réutilisation de la 2C | grille de tuiles (ch. 25), composants (ch. 28), entrées (ch. 30), niveaux en `List<String>` (ch. 39) |
| Difficulté technique | moyenne (la conception des niveaux est le vrai travail) |
| Estimation | 30 à 40 h |

**Hors du lot :** génération procédurale de puzzles, indices automatiques, minuteur, score.

**Le vrai défi :** la conception des douze niveaux. Chacun doit introduire ou combiner une idée, et être résoluble. Le prototypage papier de la section 41.31 n'est pas optionnel ici : il constitue l'essentiel du travail de design.

### Comparaison

| Critère | A — Chute Libre | B — Gardiens du Pont | C — Racine |
| --- | --- | --- | --- |
| Difficulté technique | Faible | Moyenne | Moyenne |
| Difficulté de game design | Moyenne | Élevée | Élevée |
| Quantité de contenu à produire | Très faible | Faible | Élevée |
| Risque de dépassement | Faible | Moyen | Élevé |
| Recommandé si | Vous voulez livrer sûrement | Vous aimez l'équilibrage | Vous aimez concevoir |

### Si vous proposez votre propre sujet

Il doit satisfaire ces sept conditions :

1. le pitch tient en une phrase de moins de quarante mots ;
2. la partie complète dure moins de dix minutes ;
3. il y a au maximum deux types d'ennemis ou d'obstacles mobiles ;
4. il y a au maximum quatre niveaux, ou aucun ;
5. il n'exige ni réseau, ni compte utilisateur, ni serveur ;
6. il réutilise au moins quatre chapitres de la PARTIE 2B ou 2C ;
7. l'estimation totale, calculée avec la table de la section 41.21, tient en 40 heures.

> **À retenir.** Choisissez le sujet dont vous saurez livrer la version complète, pas celui qui vous impressionne le plus. Un jeu simple et fini vaut infiniment mieux qu'un jeu ambitieux abandonné.

---

## 41.36 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| Le GDD contient `PositionComponent`, `overlays`, `onLoad` | Confusion entre GDD et document d'architecture | Le GDD décrit l'expérience du joueur ; déplacez le technique dans `ARCHITECTURE.md` |
| Le cahier des charges dit « le jeu doit être amusant » | Exigence non vérifiable | Reformuler en critère mesurable : « 3 testeurs sur 3 demandent à rejouer » |
| Toutes les exigences sont marquées « obligatoire » | Aucune priorisation réelle | Répartir en O / S / F ; s'il n'y a rien en F, la liste est fausse |
| La section « hors du lot » est vide | Aucune décision n'a été prise | Écrire au moins cinq refus explicites et chiffrés |
| Le pitch contient « fun », « immersif », « innovant » | Adjectifs à la place de mécaniques | Supprimer tous les adjectifs et vérifier qu'il reste un verbe d'action |
| Le GDD écrit « plusieurs niveaux » et « beaucoup d'ennemis » | Contenu non chiffré | Remplacer par des nombres : 3 niveaux, 2 ennemis, 1 boss |
| La boucle de gameplay ne revient pas à son point de départ | Le schéma n'est pas une boucle | Vérifier que la dernière étape ramène à la première |
| Un écran du graphe de navigation n'a aucune flèche sortante | Écran-piège pour le joueur | Ajouter au moins un bouton de sortie à chaque écran |
| Le score maximum théorique n'a jamais été calculé | Étape jugée inutile | Le calculer : il révèle presque toujours un déséquilibre |
| Le niveau 1 fait mourir tous les testeurs | Difficulté réglée sur la compétence du concepteur | Diviser la difficulté du niveau 1 par deux avant même le premier test |
| Les assets s'appellent `Sprite Final (2).png` | Aucune convention de nommage | Schéma `categorie_nom_variante_index.ext`, sans espace ni majuscule ni accent |
| `LICENCES.md` contient des lignes « licence : inconnue » | Fichiers téléchargés sans noter la source | Retirer ces fichiers ou retrouver la source ; publication interdite sinon |
| Le dépôt Git pèse 2 Go | `build/` et les fichiers sources versionnés | Appliquer le `.gitignore` de la section 41.30 et purger l'historique |
| Le planning n'a aucune marge | 100 % du temps affecté à des tâches | Réserver 20 % pour les imprévus |
| Le jalon « c'est beau » a été fait avant « ça joue » | Envie de voir un résultat visuel | Revenir à l'ordre : tourner, jouer, équilibrer, embellir, livrer |
| Le développeur explique le jeu pendant le playtest | Réflexe de protection | Se taire intégralement pendant la partie |
| Une remarque d'un seul testeur a déclenché une refonte | Confusion entre avis et fait | Ne corriger qu'à partir de 3 testeurs sur 5 |
| L'architecture a été refondue trois fois | Structure choisie sans être écrite | Figer `ARCHITECTURE.md` à J0, ne changer qu'entre deux jalons |

---

## 41.37 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| Cause d'abandon | Absence de décisions écrites, jamais un manque de compétence technique |
| Projet trop ambitieux | Divisez toutes les quantités par cinq ; si l'idée survit, elle est bonne |
| Périmètre minimal jouable | Lancer, jouer, gagner ou perdre, recommencer — sans écran vide |
| Trois questions | Que fait le joueur ? Qu'est-ce qui rend cela difficile ? Pourquoi rejouer ? |
| Pitch | Une phrase, un verbe d'action, une contrainte, moins de 40 mots |
| Cahier des charges | Répond à « quoi » et à « quand est-ce fini » ; figé au départ |
| GDD | Répond à « comment ça se joue » ; vivant, chiffré, sans code |
| Exigence fonctionnelle | Ce que le joueur peut faire ; contient un verbe |
| Exigence non fonctionnelle | Avec quelle qualité ; contient obligatoirement un nombre |
| Périmètre | Trois colonnes : dans le lot, hors du lot, reporté ; il s'échange, il ne grossit pas |
| Mécanique | Relie une entrée du joueur à un effet ; une seule est principale |
| Boucle de gameplay | Trois échelles : quelques secondes, une partie, plusieurs parties |
| Courbe de difficulté | Une nouveauté par palier, présentée sans danger, puis une respiration |
| Économie | Chaque ressource : gain, perte, plafond ; score maximum calculé à l'avance |
| Direction artistique | La moins coûteuse qui reste lisible ; une teinte pour le danger, une pour l'aide |
| Son | 5 à 8 effets suffisent ; jsfxr évite toute question de licence |
| Interface | Graphe des écrans dessiné avant de coder ; aucun écran sans sortie |
| Contenu | Tout est chiffré, donc tout est estimable en heures |
| Architecture | Quatre couches, une seule direction de dépendance |
| Structure de dossiers | Par type technique pour un premier jeu ; on n'en change plus |
| Nommage | Une langue, verbes à l'infinitif, booléens en `est`/`a`/`peut` |
| Assets | Pas d'espace, pas de majuscule, pas d'accent ; une seule taille de tuile ; moins de 8 Mo |
| Licences | `LICENCES.md` rempli au téléchargement ; aucune ligne « inconnue » |
| Planning | Un jalon est un état vérifiable et jouable, pas une date ; 20 % de marge |
| Ordre des jalons | Ça tourne, ça joue, ça s'équilibre, c'est beau, c'est livré |
| Git | Ignorer `build/` et les clés ; une branche par fonctionnalité, un tag par jalon |
| Prototypage papier | Teste les règles et les niveaux, jamais les sensations |
| Playtest | Se taire, chronométrer, observer ; 3 testeurs sur 5 font un fait |
| Pièges du débutant | Quinze, listés en 41.34 ; à relire à chaque jalon |
| Sujet final | Choisir celui qu'on saura livrer, pas celui qui impressionne |

---

## 41.38 — Exercices

Ces dix exercices ne sont pas des exercices d'entraînement : ce sont **les dix premières étapes réelles de votre projet final**. Faites-les dans l'ordre, sur le sujet que vous avez choisi en 41.35. À la fin, votre dossier `docs/` sera complet et vous pourrez commencer à coder.

### Exercice 1 — Le tri des idées (facile)

Remplissez la grille des trois questions de la section 41.4 pour **trois** idées de jeu différentes : les trois sujets proposés en 41.35, ou vos propres idées. Concluez par un verdict argumenté pour chacune, puis désignez celle que vous retenez.

### Exercice 2 — Le pitch (facile)

Écrivez le pitch de l'idée retenue, en respectant la formule de la section 41.5. Faites-le passer par les quatre tests (verbe, adjectif, contrainte, longueur) et montrez explicitement le résultat de chaque test. Écrivez ensuite deux variantes rejetées et dites pourquoi elles échouent.

### Exercice 3 — Les exigences (moyen)

Rédigez **dix exigences fonctionnelles** (EF-01 à EF-10) et **huit exigences non fonctionnelles** (ENF-01 à ENF-08) pour votre jeu. Chaque exigence fonctionnelle commence par « Le joueur peut… » ou « Le jeu… ». Chaque exigence non fonctionnelle contient un nombre. Attribuez une priorité O, S ou F à chaque exigence fonctionnelle, avec au moins deux F.

### Exercice 4 — Le périmètre (moyen)

Écrivez le tableau à trois colonnes de la section 41.9 pour votre jeu : au moins six lignes dans « dans le lot », six dans « hors du lot » et quatre dans « reporté ». Chaque ligne de « hors du lot » doit être chiffrée ou nommée précisément, jamais vague.

### Exercice 5 — Le cahier des charges complet (difficile)

À partir des exercices 2, 3 et 4, produisez le fichier `docs/CAHIER-DES-CHARGES.md` complet, en suivant le gabarit de la section 41.7. Toutes les rubriques doivent être remplies, y compris les critères d'acceptation (au moins cinq) et les risques (au moins trois, avec parade).

### Exercice 6 — La boucle de gameplay et la courbe de difficulté (moyen)

Dessinez, dans un bloc `text`, les trois boucles de votre jeu (courte, moyenne, longue), en indiquant pour chacune sa durée, sa récompense, son risque et sa source de variation. Ajoutez ensuite la table de progression de la difficulté de la section 41.16, avec au moins six étapes chiffrées.

### Exercice 7 — L'économie et le contenu (difficile)

Écrivez les sections 6 (économie) et 10 (contenu) du GDD pour votre jeu : tableau des ressources, barème de points, calcul du score ou du budget maximum théorique, table des niveaux, table des ennemis ou obstacles, table des objets. Terminez par l'estimation en heures, à l'aide de la table de la section 41.21, et comparez-la au temps dont vous disposez réellement.

### Exercice 8 — L'architecture et l'arborescence (moyen)

Produisez `docs/ARCHITECTURE.md` : le schéma en couches adapté à votre jeu, les règles de dépendance, l'arborescence complète de `lib/` avec un commentaire par fichier indiquant le jalon qui le crée, et une table d'au moins quatre décisions techniques avec l'alternative écartée et la raison.

### Exercice 9 — Le planning et le tableau des licences (moyen)

Produisez deux livrables. D'abord le planning en jalons de la section 41.28, adapté à votre projet et à votre temps disponible réel, avec la marge de 20 % visible. Ensuite `docs/LICENCES.md` prérempli : les paquets Dart que vous utiliserez, et les sources d'assets que vous comptez employer, avec leur licence et l'attribution requise.

### Exercice 10 — La grille de playtest et le prototype papier (difficile)

Produisez deux livrables. D'abord `docs/playtests/GRILLE-VIERGE.md`, adapté à votre jeu : reprenez le gabarit de la section 41.33 et remplacez les indicateurs génériques par ceux qui ont un sens pour votre jeu. Ensuite, réalisez le prototypage papier de votre premier niveau ou de votre première partie : la grille ASCII, la fiche de portée du joueur, et le tableau des problèmes constatés avec leur correction.

---

## 41.39 — Corrections des exercices

Les corrections ci-dessous sont des **livrables complets**, réalisés sur le sujet A, « Chute Libre », de la section 41.35. Elles vous montrent le niveau de précision attendu. Votre propre livrable portera sur votre sujet, mais devra atteindre le même niveau de détail.

### Correction 1 — Le tri des idées

```markdown
### Idée 1 : Chute Libre

1. Que fait le joueur, physiquement ?
   → Il fait glisser son doigt à gauche et à droite pour déplacer un panier
     au bas de l'écran.

2. Qu'est-ce qui rend cette action difficile ?
   → Plusieurs objets tombent en même temps, à des vitesses différentes, et
     certains (les pierres) doivent être évités et non rattrapés.

3. Pourquoi recommencerait-il après avoir perdu ?
   → Pour battre son meilleur score, affiché en permanence sous le score
     courant. Une partie dure 60 à 180 s : recommencer coûte peu.

Verdict : idée retenue.
Raison : les trois réponses sont courtes et précises. La mécanique tient en
un geste, la difficulté est intrinsèque, et la boucle longue existe.

---

### Idée 2 : Les Gardiens du Pont

1. Que fait le joueur, physiquement ?
   → Il touche une case libre du pont pour y poser une tour, s'il a assez d'or.

2. Qu'est-ce qui rend cette action difficile ?
   → L'or est limité, les vagues arrivent à heure fixe, et le mauvais type de
     tour au mauvais endroit se paie deux vagues plus tard.

3. Pourquoi recommencerait-il après avoir perdu ?
   → Pour essayer une autre disposition de tours. La rejouabilité vient de la
     stratégie, pas du score.

Verdict : idée retenue en seconde position.
Raison : les réponses sont bonnes, mais l'équilibrage or / coût / dégâts
demande beaucoup de tests, donc du temps que je n'ai pas.

---

### Idée 3 : Un jeu d'exploration sous-marine

1. Que fait le joueur, physiquement ?
   → Il... explore les fonds marins et découvre des espèces.

2. Qu'est-ce qui rend cette action difficile ?
   → L'oxygène, peut-être. Ou la pression.

3. Pourquoi recommencerait-il après avoir perdu ?
   → Pour voir le reste de la carte.

Verdict : idée écartée.
Raison : la première réponse n'a pas de verbe concret, la deuxième contient
"peut-être" et "ou", la troisième suppose une grande carte donc beaucoup de
contenu. Trois signaux d'alerte sur trois : c'est une envie d'ambiance, pas
une idée de jeu.
```

**Explication :** l'intérêt de l'exercice n'est pas de trouver la bonne idée, mais de rendre visible ce qui distingue une idée exploitable d'une envie d'ambiance. Observez la troisième fiche : les hésitations (« peut-être », « ou », « ... ») sont le vrai verdict. Une idée qu'on ne sait pas décrire en trois phrases courtes ne se codera pas non plus. Notez aussi que l'idée 2 n'est pas mauvaise : elle est écartée pour une raison de temps disponible, ce qui est une décision de projet et non de conception.

### Correction 2 — Le pitch

```markdown
## Pitch

Chute Libre est un jeu d'arcade vertical dans lequel le joueur déplace un
panier pour rattraper des œufs qui tombent, tout en évitant des pierres dont
la fréquence augmente avec le temps.

Vérification :
- [x] contient un verbe d'action : "déplace", "rattraper", "évitant"
- [x] ne repose sur aucun adjectif : aucun adjectif qualificatif présent
- [x] énonce la contrainte : "des pierres dont la fréquence augmente"
- [x] moins de 40 mots : 34 mots

---

## Variantes rejetées

### Variante A
"Chute Libre est un jeu d'arcade fun et addictif où il faut attraper plein
de choses qui tombent du ciel dans une ambiance colorée."

Rejetée : "fun", "addictif", "colorée" sont des adjectifs qui portent tout le
sens. En les supprimant, il ne reste que "attraper des choses qui tombent",
sans aucune contrainte. Le test de l'adjectif et le test de la contrainte
échouent tous les deux.

### Variante B
"Chute Libre est un jeu où vous incarnez un fermier qui doit sauver sa récolte
d'œufs menacée par un éboulement, dans un univers rural attachant."

Rejetée : c'est un synopsis, pas un pitch. Il décrit une situation narrative
et ne dit ni ce que le joueur fait avec ses doigts, ni ce qui rend cela
difficile. Le test du verbe échoue : "incarner" et "devoir sauver" ne sont
pas des actions de joueur.
```

**Explication :** les deux variantes rejetées sont les deux formes d'échec les plus fréquentes. La variante A remplace la mécanique par des adjectifs ; le test de l'adjectif la détruit en une seconde. La variante B remplace la mécanique par une fiction ; le test du verbe la détruit tout aussi vite. Écrire les variantes rejetées n'est pas une perte de temps : cela vous entraîne à reconnaître ces deux formes chez vous.

### Correction 3 — Les exigences

```markdown
## 3. Exigences fonctionnelles

| ID | Exigence | Priorité |
| --- | --- | --- |
| EF-01 | Le joueur peut déplacer le panier horizontalement sur toute la largeur de l'écran. | O |
| EF-02 | Le jeu fait tomber des œufs depuis le haut de l'écran, à des positions horizontales aléatoires. | O |
| EF-03 | Le joueur gagne 10 points lorsqu'un œuf touche le panier. | O |
| EF-04 | Le joueur perd une vie sur trois lorsqu'un œuf touche le sol. | O |
| EF-05 | Le joueur perd une vie sur trois lorsqu'une pierre touche le panier. | O |
| EF-06 | Le jeu affiche en permanence le score, le meilleur score et les vies restantes. | O |
| EF-07 | Le jeu augmente la vitesse de chute et la fréquence d'apparition toutes les 20 secondes. | O |
| EF-08 | Le jeu affiche un écran de fin quand les trois vies sont perdues, avec le score final. | O |
| EF-09 | Le jeu conserve le meilleur score entre deux lancements de l'application. | S |
| EF-10 | Le joueur peut couper les effets sonores depuis le menu principal. | S |
| EF-11 | Le jeu fait apparaître un œuf doré valant 50 points environ toutes les 30 secondes. | F |
| EF-12 | Le joueur peut mettre la partie en pause et la reprendre. | F |

## 4. Exigences non fonctionnelles

| ID | Exigence | Seuil mesurable |
| --- | --- | --- |
| ENF-01 | Fluidité | 60 images par seconde avec 15 objets simultanés à l'écran |
| ENF-02 | Démarrage | menu principal affiché en moins de 3 secondes |
| ENF-03 | Taille | APK inférieur à 25 Mo, build Web inférieur à 6 Mo |
| ENF-04 | Compatibilité Android | Android 8.0 (API 26) minimum |
| ENF-05 | Compatibilité Web | Chrome et Firefox en version à jour, en 1280×720 minimum |
| ENF-06 | Latence d'entrée | moins de 80 ms entre le déplacement du doigt et celui du panier |
| ENF-07 | Robustesse | aucune exception non capturée sur 10 parties complètes consécutives |
| ENF-08 | Accessibilité | œuf et pierre distinguables par la forme seule, sans la couleur |
| ENF-09 | Hors-ligne | le jeu fonctionne intégralement sans connexion réseau |
| ENF-10 | Mémoire | consommation stable après 10 minutes de jeu continu |
```

**Explication :** trois points sont à observer. D'abord, chaque exigence fonctionnelle décrit un comportement **observable** : aucune ne mentionne `SpriteComponent`, `CollisionCallbacks` ni `SharedPreferences`. Ensuite, la colonne de priorité est réellement utilisée : EF-11 et EF-12 sont en F, ce qui signifie que si le planning dérape, l'œuf doré et la pause sautent sans que le jeu soit incomplet. Enfin, chaque exigence non fonctionnelle contient un nombre : « fluide », « rapide » ou « léger » ne se vérifient pas, mais « 60 images par seconde », « moins de 3 secondes » et « moins de 25 Mo » se vérifient au chapitre 42.

### Correction 4 — Le périmètre

```markdown
## 5. Périmètre

| Dans le lot (v1.0) | Hors du lot (jamais) | Reporté (v1.1) |
| --- | --- | --- |
| 1 scène unique, sans défilement | Niveaux ou mondes | Un mode "nuit" avec fond différent |
| 3 objets : œuf, œuf doré, pierre | Ennemis mobiles ou intelligents | Un 4e objet : le panier bonus |
| 3 vies, pas de gain de vie | Système de vies achetables | Une vie bonus tous les 500 points |
| Difficulté continue par paliers de 20 s | Sélection de difficulté | Un mode "difficile" au démarrage |
| Score et meilleur score local | Classement en ligne | Partage du score par capture d'écran |
| 5 effets sonores, 1 musique | Doublage, voix | Deux musiques alternées |
| Interface en français | Localisation multilingue | Traduction anglaise |
| Android et navigateur | iOS, Windows, consoles | Build Windows |
| Formes géométriques colorées | Sprites achetés, animations complexes | Sprites Kenney (CC0) |
```

**Explication :** comparez la colonne du milieu à ce qu'écrit un débutant. « Pas de niveaux » est vague ; « Niveaux ou mondes » associé à « 1 scène unique, sans défilement » dans la colonne de gauche est une décision technique nette : il n'y aura pas de caméra mobile, donc pas de gestion de monde, donc plusieurs jours économisés. Observez aussi la colonne de droite : elle contient uniquement des idées **auxquelles on a dit oui, plus tard**. C'est ce qui permet de ne pas les refuser en bloc, et donc de ne pas y revenir toutes les semaines.

### Correction 5 — Le cahier des charges complet

```markdown
# CAHIER DES CHARGES — Chute Libre

Version : 1.0
Date : 08/08/2026
Auteur : <votre nom>

---

## 1. Présentation

### 1.1 Pitch
Chute Libre est un jeu d'arcade vertical dans lequel le joueur déplace un
panier pour rattraper des œufs qui tombent, tout en évitant des pierres dont
la fréquence augmente avec le temps.

### 1.2 Contexte
Projet final de la PARTIE 2D de la formation Flutter + jeu 2D. Il doit
démontrer la maîtrise des chapitres 27 à 40 sur un jeu conçu de bout en bout.

### 1.3 Objectif du projet
Livrer un jeu complet, jouable, buildé pour Android et pour le Web, en
40 heures de travail maximum, documents de conception compris.

---

## 2. Identité du produit

| Rubrique | Décision |
| --- | --- |
| Titre | Chute Libre |
| Genre | Arcade, score, scène unique |
| Nombre de joueurs | 1 |
| Public cible | 10 à 50 ans, joueur occasionnel |
| Plateformes visées | Android 8.0+, navigateur Chrome et Firefox |
| Orientation de l'écran | Portrait |
| Résolution de référence | 720 × 1280 logique |
| Durée d'une partie | 60 à 180 secondes |
| Durée de vie totale visée | 20 à 30 minutes cumulées |
| Modèle de distribution | Gratuit, sans publicité |
| Langue | Français |

---

## 3. Exigences fonctionnelles
<voir la correction 3, EF-01 à EF-12>

## 4. Exigences non fonctionnelles
<voir la correction 3, ENF-01 à ENF-10>

## 5. Périmètre
<voir la correction 4>

---

## 6. Contraintes techniques

| Contrainte | Valeur |
| --- | --- |
| Langage | Dart 3.12 |
| Framework | Flutter 3.44 |
| Moteur | Flame 1.38.0 |
| Paquets autorisés | flame, flame_audio, shared_preferences |
| Version minimale d'Android | 8.0 (API 26) |
| Navigateurs supportés | Chrome, Firefox, versions à jour |
| Taille maximale des assets | 4 Mo |

---

## 7. Livrables

| Livrable | Format | Échéance |
| --- | --- | --- |
| Code source | dépôt Git, branche `main` | J6 |
| Build Android | `chute-libre-1.0.0.apk` | J6 |
| Build Web | dossier `build/web` publié | J6 |
| Documentation | `README.md` | J6 |
| Cahier des charges | `docs/CAHIER-DES-CHARGES.md` | J0 |
| GDD | `docs/GDD.md` | J0 |
| Architecture | `docs/ARCHITECTURE.md` | J0 |
| Licences | `docs/LICENCES.md` | J6, tenu à jour en continu |
| Rapports de playtest | `docs/playtests/*.md` | J4 |

---

## 8. Planning et jalons

| Jalon | Contenu | Date cible |
| --- | --- | --- |
| J0 | Les quatre documents de conception écrits | 10/08 |
| J1 | Une fenêtre, un panier qui suit le doigt | 13/08 |
| J2 | Boucle complète : objets qui tombent, score, 3 vies, game over, rejeu | 20/08 |
| J3 | Œuf doré, paliers de difficulté, meilleur score sauvegardé | 27/08 |
| J4 | 3 playtests menés, valeurs d'équilibrage corrigées | 31/08 |
| J5 | Sons, effets visuels, menu et écrans finalisés | 06/09 |
| J6 | Builds Android et Web, documentation, publication | 10/09 |

---

## 9. Critères d'acceptation

Le projet est terminé si et seulement si :

1. l'application se lance sur le menu principal en moins de 3 secondes, sans
   écran noir intermédiaire ;
2. une partie complète peut être jouée du début au Game Over sans exception
   ni blocage, sur Android et sur navigateur ;
3. le meilleur score survit à la fermeture complète de l'application ;
4. le jeu maintient 60 images par seconde avec 15 objets simultanés à l'écran ;
5. trois personnes extérieures ont joué au moins une partie complète, et deux
   sur trois ont relancé une seconde partie sans qu'on le leur demande ;
6. `docs/LICENCES.md` ne contient aucune ligne de licence inconnue ;
7. l'APK pèse moins de 25 Mo et le build Web moins de 6 Mo.

---

## 10. Risques identifiés

| Risque | Probabilité | Impact | Parade |
| --- | --- | --- | --- |
| L'équilibrage de la difficulté demande plus de tests que prévu | forte | moyen | Mettre les paliers dans une liste de constantes modifiable en une ligne (section 41.16) |
| Le build Android échoue la première fois (SDK, signature) | forte | fort | Faire un build de test dès le jalon J1, pas à J6 |
| Le contrôle tactile est imprécis sur téléphone | moyenne | fort | Tester sur l'appareil cible dès J2, prévoir un suivi direct du doigt |
| Manque de temps en fin de projet | forte | moyen | EF-11 et EF-12 sont en priorité F et sautent en premier |
| Aucun testeur disponible | moyenne | fort | Prévoir les playtests dès J3, avec deux dates de repli |
```

**Explication :** ce document tient en trois pages et se relit en cinq minutes. Trois éléments sont structurants. Le critère d'acceptation n° 5 transforme une intuition (« le jeu est amusant ») en un fait observable (« deux personnes sur trois relancent une partie sans qu'on le leur demande ») : c'est ainsi qu'on rend vérifiable une exigence qui semblait subjective. Le risque « le build Android échoue » a une parade qui déplace le travail plus tôt, ce qui est la seule parade efficace contre les risques techniques. Enfin, la parade du risque de manque de temps ne dit pas « travailler plus » mais désigne nommément ce qui sera coupé : la décision est déjà prise, à froid.

### Correction 6 — La boucle de gameplay et la courbe de difficulté

```text
   BOUCLE COURTE — durée d'un tour : 1 à 3 s
   ┌──────────────────────────────────────────────┐
   │                                              │
   │   1. un objet apparaît en haut de l'écran    │
   │            │                                 │
   │            ▼                                 │
   │   2. le joueur identifie sa nature           │
   │      (œuf = aller vers ; pierre = fuir)      │
   │            │                                 │
   │            ▼                                 │
   │   3. il déplace le panier                    │
   │            │                                 │
   │            ▼                                 │
   │   4. contact : +10 points, ou -1 vie         │
   │            │                                 │
   │            ▼                                 │
   │   5. l'objet disparaît, un autre arrive  ────┘
   │
   └── récompense : +10 points et un son
       risque     : perte d'une vie sur trois
       variation  : position horizontale aléatoire, nature de l'objet,
                    et surtout objets simultanés qui se contredisent
```

```text
   BOUCLE MOYENNE — durée : 60 à 180 s
   ┌──────────────────────────────────────────────────────────┐
   │                                                          │
   │   lancer une partie (3 vies, score à 0)                  │
   │            │                                             │
   │            ▼                                             │
   │   rattraper des œufs  ──► le score monte                 │
   │            │                                             │
   │            ▼                                             │
   │   palier de 20 s franchi ──► vitesse et fréquence + 12 % │
   │            │                                             │
   │            ▼                                             │
   │   les erreurs deviennent inévitables ──► vies perdues    │
   │            │                                             │
   │            ▼                                             │
   │   0 vie ──► écran de fin, score comparé au meilleur  ────┘
   │
   └── récompense : le score final, et le meilleur score battu
       risque     : la partie se termine toujours, c'est un jeu de survie
       variation  : chaque partie a une séquence d'objets différente
```

```text
   BOUCLE LONGUE — durée : plusieurs sessions
   ┌───────────────────────────────────────────────┐
   │                                               │
   │   voir "MEILLEUR 0430" et "SCORE 0410"        │
   │            │                                  │
   │            ▼                                  │
   │   estimer que 20 points étaient à portée      │
   │            │                                  │
   │            ▼                                  │
   │   relancer immédiatement (bouton "Rejouer")   │
   │            │                                  │
   │            ▼                                  │
   │   battre le record ──► nouveau seuil à battre ┘
   │
   └── récompense : le statut, le nombre affiché
       risque     : aucun, c'est ce qui rend le rejeu peu coûteux
       variation  : la marge à combler change à chaque partie
```

```markdown
## 5. Courbe de difficulté

| Étape | Durée | Ce qui augmente | Valeurs |
| --- | --- | --- | --- |
| Palier 0 | 0-20 s | rien, apprentissage | 1 objet toutes les 1,4 s, chute à 180 px/s, 0 % de pierres |
| Palier 1 | 20-40 s | apparition des pierres | 1 objet / 1,3 s, 200 px/s, 15 % de pierres |
| Palier 2 | 40-60 s | fréquence | 1 objet / 1,15 s, 225 px/s, 20 % de pierres |
| Palier 3 | 60-80 s | vitesse | 1 objet / 1,05 s, 255 px/s, 22 % de pierres |
| Palier 4 | 80-100 s | proportion de pierres | 1 objet / 0,95 s, 285 px/s, 28 % de pierres |
| Palier 5 | 100-120 s | fréquence | 1 objet / 0,85 s, 315 px/s, 30 % de pierres |
| Palier 6+ | 120 s et plus | plafond atteint | 1 objet / 0,75 s, 350 px/s, 33 % de pierres |

Règle appliquée : un seul levier augmente par palier (section 41.16). Le
palier 0 ne contient aucune pierre : la nouveauté "pierre" est donc présentée
au palier 1, à faible fréquence, avant de devenir dangereuse.

Plafond : au-delà du palier 6, plus rien n'augmente. Sans plafond, le jeu
deviendrait mathématiquement impossible et le score cesserait de dépendre de
l'habileté.
```

**Explication :** trois choses méritent l'attention. D'abord, chaque boucle indique explicitement sa récompense, son risque et sa variation : c'est le test de la section 41.15, appliqué. Ensuite, la table de difficulté ne fait varier **qu'un seul levier par palier**, ce qui permettra, après le playtest, de savoir lequel est en cause. Enfin, le plafond au palier 6 est une décision de conception essentielle pour un jeu de score : sans lui, tous les joueurs finiraient par mourir au même moment, et le score ne récompenserait plus l'habileté.

### Correction 7 — L'économie et le contenu

```markdown
## 6. Économie du jeu

### 6.1 Ressources

| Ressource | Gain | Perte | Plafond |
| --- | --- | --- | --- |
| Score | œuf +10, œuf doré +50 | jamais | aucun |
| Vies | aucun gain | œuf tombé au sol -1, pierre attrapée -1 | 3 au départ |
| Temps | s'écoule | sans objet | sans objet |

### 6.2 Barème de points

| Action | Points |
| --- | --- |
| Œuf rattrapé | 10 |
| Œuf doré rattrapé | 50 |
| Pierre évitée | 0 (aucune récompense : éviter est la norme) |
| Survie de 20 s (palier franchi) | 0 (le score vient des œufs, pas du temps) |

### 6.3 Équilibrage : score théorique par durée de survie

Hypothèse : un joueur parfait rattrape 100 % des œufs et évite 100 % des
pierres. On calcule le nombre d'objets par palier, dont on retire les pierres.

| Palier | Durée | Objets | Dont pierres | Œufs | Points |
| --- | --- | --- | --- | --- | --- |
| 0 | 20 s | 14 | 0 | 14 | 140 |
| 1 | 20 s | 15 | 2 | 13 | 130 |
| 2 | 20 s | 17 | 3 | 14 | 140 |
| 3 | 20 s | 19 | 4 | 15 | 150 |
| 4 | 20 s | 21 | 6 | 15 | 150 |
| 5 | 20 s | 23 | 7 | 16 | 160 |
| **Total à 120 s** | | **109** | **22** | **87** | **870** |

Ajout des œufs dorés : environ 1 toutes les 30 s, soit 4 sur 120 s, ce qui
remplace 4 œufs ordinaires : +40 × 4 = +160.

**Score théorique maximal à 120 secondes de survie : environ 1030 points.**

Conséquences retenues :
- un score de 400 correspond à une bonne partie ordinaire (environ 60 s) ;
- un score au-dessus de 800 signale un joueur qui dépasse les 100 s ;
- l'œuf doré représente 15 % du score total : il est significatif sans
  écraser le reste, contrairement au boss du Donjon de Dart qui pesait 32 %.

---

## 10. Contenu

### 10.1 Niveaux

| N° | Nom | Thème | Nouveauté | Durée |
| --- | --- | --- | --- | --- |
| — | scène unique | ciel dégradé, sol en bas | difficulté continue | illimitée |

Décision : aucun niveau. Le contenu est généré par les paliers de difficulté,
ce qui supprime entièrement le travail de level design. C'est la raison
principale du choix de ce sujet à 40 heures.

### 10.2 Obstacles

| Nom | Comportement | Vitesse | Effet au contact | Fréquence |
| --- | --- | --- | --- | --- |
| Pierre | chute verticale | selon le palier | -1 vie | 0 à 33 % des objets |

### 10.3 Objets

| Nom | Effet | Fréquence | Forme | Couleur |
| --- | --- | --- | --- | --- |
| Œuf | +10 points | 67 à 100 % des objets | ovale | blanc cassé |
| Œuf doré | +50 points | 1 toutes les 30 s environ | ovale avec liseré | jaune |
| Pierre | -1 vie | 0 à 33 % des objets | hexagone irrégulier | gris foncé |

Note d'accessibilité : les trois objets se distinguent par la forme (ovale,
ovale liséré, hexagone) et pas seulement par la couleur. Exigence ENF-08.

---

## Estimation en heures

| Élément | Heures |
| --- | --- |
| Documents de conception (déjà faits) | 4 |
| Squelette du projet, FlameGame, machine à états | 3 |
| Panier et contrôle tactile + clavier | 3 |
| Générateur d'objets et chute | 3 |
| Collisions et effets (score, vies) | 3 |
| Paliers de difficulté | 2 |
| HUD (score, meilleur score, vies) | 3 |
| Menu, écran de game over, rejeu | 4 |
| Sauvegarde du meilleur score | 2 |
| Audio (5 sons + 1 musique) | 3 |
| Effets visuels et polish | 4 |
| Playtests et corrections d'équilibrage | 4 |
| Builds Android et Web, documentation | 5 |
| **Sous-total** | **43** |
| Marge de 20 % | 9 |
| **Total** | **52 h** |

Temps disponible réel : 10 h par semaine pendant 5 semaines = 50 h.

Décision : 52 > 50. On coupe EF-11 (œuf doré, -2 h) et EF-12 (pause, -1 h),
qui étaient en priorité F. Nouveau total : 49 h. Le projet tient.
```

**Explication :** la dernière ligne est l'aboutissement de tout le chapitre. Le chiffrage du contenu (section 41.21) donne 43 heures ; la marge de 20 % (section 41.28) porte le total à 52 ; le temps réellement disponible est de 50. L'écart apparaît **avant** le début du développement, et la priorisation faite à l'exercice 3 fournit immédiatement la réponse : on coupe les deux exigences F. Sans cette chaîne — chiffrage, marge, priorités —, l'écart de deux heures serait apparu en semaine 5, sous forme de projet non livré. Observez aussi que la décision « aucun niveau » est justifiée explicitement par le budget, et non par un manque d'idées.

### Correction 8 — L'architecture et l'arborescence

```markdown
# ARCHITECTURE — Chute Libre

## Couches

┌─────────────────────────────────────────────────────────┐
│  COUCHE 4 — PRÉSENTATION (Flutter)                      │
│  menu_principal.dart, ecran_game_over.dart              │
│  Connaît : ChuteLibreGame                                │
└──────────────────────────┬──────────────────────────────┘
┌──────────────────────────▼──────────────────────────────┐
│  COUCHE 3 — JEU (FlameGame)                             │
│  chute_libre_game.dart, generateur.dart, difficulte.dart │
│  Connaît : composants, core, services                    │
└──────────────────────────┬──────────────────────────────┘
┌──────────────────────────▼──────────────────────────────┐
│  COUCHE 2 — COMPOSANTS (Flame)                          │
│  panier.dart, objet_tombant.dart, oeuf.dart, pierre.dart │
│  Connaît : core, et le jeu par HasGameReference          │
└──────────────────────────┬──────────────────────────────┘
┌──────────────────────────▼──────────────────────────────┐
│  COUCHE 1 — NOYAU (Dart pur, testable sans Flutter)     │
│  constantes.dart, palette.dart, game_state.dart,         │
│  paliers.dart                                            │
│  Connaît : rien                                          │
└─────────────────────────────────────────────────────────┘

## Règles de dépendance

1. Aucun fichier de `config/` ou `core/` n'importe `package:flutter/*`.
2. Aucun fichier de `ecrans/` n'importe un fichier de `composants/`.
3. Un composant n'appelle jamais `SauvegardeService` : il passe par `game`.
4. `paliers.dart` ne contient que des données constantes, aucune logique.

## Arborescence

chute_libre/
├── pubspec.yaml                        # J1
├── docs/
│   ├── CAHIER-DES-CHARGES.md           # J0
│   ├── GDD.md                          # J0
│   ├── ARCHITECTURE.md                 # J0
│   ├── LICENCES.md                     # J0, tenu à jour
│   ├── JOURNAL.md                      # hebdomadaire
│   └── playtests/                      # J4
├── assets/
│   ├── images/                         # J5 (vide jusque-là)
│   └── audio/                          # J5
├── test/
│   ├── paliers_test.dart               # J3 — teste la couche 1
│   └── score_test.dart                 # J3
└── lib/
    ├── main.dart                       # J1 — runApp
    ├── chute_libre_game.dart           # J1 — la classe FlameGame
    ├── config/
    │   ├── constantes.dart             # J1 — toutes les valeurs chiffrées
    │   └── palette.dart                # J1 — les 6 couleurs du jeu
    ├── core/
    │   ├── game_state.dart             # J1 — enum GameState
    │   └── paliers.dart                # J3 — la table de difficulté
    ├── composants/
    │   ├── panier.dart                 # J1 — le panier du joueur
    │   ├── objet_tombant.dart          # J2 — classe abstraite
    │   ├── oeuf.dart                   # J2
    │   ├── oeuf_dore.dart              # J3 (priorité F, peut sauter)
    │   ├── pierre.dart                 # J2
    │   └── sol.dart                    # J2 — détecte les œufs perdus
    ├── systemes/
    │   ├── generateur.dart             # J2 — fait apparaître les objets
    │   └── difficulte.dart             # J3 — applique les paliers
    ├── hud/
    │   ├── hud.dart                    # J2
    │   └── compteur_vies.dart          # J2
    ├── services/
    │   ├── audio_service.dart          # J5
    │   └── sauvegarde_service.dart     # J3
    └── ecrans/
        ├── menu_principal.dart         # J5
        └── ecran_game_over.dart        # J2 (version minimale), J5 (finale)

## Décisions techniques

| Décision | Alternative écartée | Raison |
| --- | --- | --- |
| Scène unique sans caméra mobile | `CameraComponent` qui suit le joueur | Le jeu tient sur un écran : aucune caméra nécessaire (ch. 31 non utilisé) |
| Objets en chute par vélocité constante | `flame_forge2d` | Aucun rebond ni contact entre objets : la physique du ch. 23 suffit |
| Table de paliers en `List<Palier>` const | Une formule mathématique | Une table se règle après playtest sans recompiler la logique |
| `shared_preferences` | Fichier JSON dans le dossier de l'application | Un seul entier à stocker (ch. 40) |
| Détection de perte par un composant `Sol` | Test de position dans `update` | Réutilise la détection de collision du ch. 32, déjà en place |
| Formes dessinées avec `CustomPainter` puis `RectangleComponent` | Sprites dès le départ | Jalon "ça tourne" avant jalon "c'est beau" (section 41.29) |
```

**Explication :** deux détails font la qualité de ce document. Le premier est la colonne « jalon » dans l'arborescence : chaque fichier a une date d'apparition, ce qui rend le planning concret et empêche d'écrire `audio_service.dart` en semaine 1. Le second est la table des décisions techniques, et particulièrement sa première ligne : décider explicitement de **ne pas** utiliser la caméra du chapitre 31 est une économie de plusieurs heures, et cette décision, une fois écrite, ne sera pas remise en question tous les lundis. Notez enfin que `oeuf_dore.dart` porte la mention « priorité F, peut sauter » : la priorisation du cahier des charges se propage jusque dans l'arborescence.

### Correction 9 — Le planning et le tableau des licences

```markdown
# PLANNING — Chute Libre

Temps disponible : 10 h par semaine, 5 semaines, soit 50 h.
Budget après coupe des exigences F : 49 h (voir correction 7).

| Jalon | Nom | Contenu vérifiable | Heures | Cumul | Date cible |
| --- | --- | --- | --- | --- | --- |
| J0 | Documents | Les 4 documents de `docs/` sont écrits | 4 | 4 | 10/08 |
| J1 | Ça bouge | L'app se lance ; le panier suit le doigt et les flèches | 6 | 10 | 13/08 |
| J2 | ÇA JOUE | Objets qui tombent, score, 3 vies, game over, rejeu | 11 | 21 | 20/08 |
| J3 | Contenu | Paliers de difficulté, meilleur score sauvegardé, tests unitaires | 7 | 28 | 27/08 |
| J4 | Équilibrage | 3 playtests menés, table des paliers corrigée | 4 | 32 | 31/08 |
| J5 | C'est beau | 5 sons, effets visuels, menu et écran de fin finalisés | 7 | 39 | 06/09 |
| J6 | C'est livré | APK, build Web, README, licences vérifiées | 5 | 44 | 10/09 |
| — | Marge | Imprévus (20 % du total) | 5 | 49 | — |

## Point de contrôle J2

J2 est prévu à 21 h cumulées, soit 43 % du budget. Le seuil de la section
41.28 fixe le tiers du temps, soit 16 h. Le dépassement est assumé parce que
J0 (4 h de documents) ne produit rien de jouable.

Règle de réaction :
- si J2 n'est pas atteint à 25 h → suppression de l'œuf doré et du menu
  d'options, retour au périmètre minimal jouable ;
- si J2 n'est pas atteint à 32 h → livraison de ce qui existe à J6, aucun
  ajout de fonctionnalité.

## Suivi hebdomadaire

Trois lignes par semaine dans `docs/JOURNAL.md` : fait / pas fait / décision.
```

```markdown
# LICENCES ET ATTRIBUTIONS — Chute Libre

Dernière mise à jour : 08/08/2026

## Code

| Élément | Auteur | Licence | Attribution | Lien |
| --- | --- | --- | --- | --- |
| Flutter 3.44 | Google | BSD-3-Clause | dans les crédits | https://flutter.dev |
| flame 1.38.0 | Blue Fire | MIT | dans les crédits | https://pub.dev/packages/flame |
| flame_audio 2.12.2 | Blue Fire | MIT | dans les crédits | https://pub.dev/packages/flame_audio |
| shared_preferences | Flutter team | BSD-3-Clause | dans les crédits | https://pub.dev/packages/shared_preferences |
| Code de Chute Libre | <votre nom> | MIT | — | dépôt du projet |

## Images

| Fichier | Auteur | Licence | Attribution | Lien | Date |
| --- | --- | --- | --- | --- | --- |
| aucune (formes dessinées par code) | <votre nom> | — | — | — | — |

Décision : le jeu n'utilise aucun fichier image. Toutes les formes sont
dessinées avec `Canvas` (chapitre 21). Cette ligne est conservée pour
attester que la question a été traitée, et non oubliée.

## Sons

| Fichier | Auteur | Licence | Attribution | Lien | Date |
| --- | --- | --- | --- | --- | --- |
| sfx_attrape.wav | <votre nom> | — | non | généré avec jsfxr | 04/09/2026 |
| sfx_dore.wav | <votre nom> | — | non | généré avec jsfxr | 04/09/2026 |
| sfx_casse.wav | <votre nom> | — | non | généré avec jsfxr | 04/09/2026 |
| sfx_pierre.wav | <votre nom> | — | non | généré avec jsfxr | 04/09/2026 |
| sfx_gameover.wav | <votre nom> | — | non | généré avec jsfxr | 04/09/2026 |
| bgm_ciel.ogg | Kevin MacLeod | CC-BY 4.0 | OUI | https://incompetech.com | 05/09/2026 |

## Polices

| Fichier | Auteur | Licence | Attribution | Lien |
| --- | --- | --- | --- | --- |
| PressStart2P-Regular.ttf | CodeMan38 | SIL OFL 1.1 | non requise | https://fonts.google.com/specimen/Press+Start+2P |

## Mentions à afficher dans l'écran "Crédits"

    Chute Libre — <votre nom>, 2026

    Musique : "Sunny Sky" par Kevin MacLeod (incompetech.com)
    Sous licence Creative Commons Attribution 4.0

    Police : Press Start 2P par CodeMan38
    SIL Open Font License 1.1

    Réalisé avec Flutter (Google) et Flame (Blue Fire)

## Vérification avant publication

- [x] Tous les fichiers de `assets/` figurent dans ce document
- [x] Aucune licence "inconnue"
- [x] Aucune licence NC (le jeu est gratuit, mais la règle est appliquée)
- [x] Les attributions requises sont dans l'écran Crédits
- [x] Le texte des licences MIT et BSD est inclus dans le dépôt
```

**Explication :** deux points sont à relever. Dans le planning, le point de contrôle J2 est chiffré en deux seuils, avec la réaction déjà décidée pour chacun : le jour où vous serez à 26 heures sans J2, vous n'aurez pas à arbitrer sous pression, la décision aura été prise à froid. Dans le tableau des licences, observez la ligne « aucune image » : documenter une absence est utile, parce qu'elle prouve que la question a été traitée. Observez enfin la date en face de chaque son : elle est celle du téléchargement ou de la génération, conformément à la règle de la section 41.27, et non celle de la fin du projet.

### Correction 10 — La grille de playtest et le prototype papier

```markdown
# PLAYTEST — Chute Libre

Test n° : 01
Date : 29/08/2026
Version testée : tag `j3-contenu`, commit a4f19c2
Testeur : M., 34 ans, joue occasionnellement sur téléphone
Support : Android 13, téléphone 6,1 pouces, en portrait
Durée de la séance : 18 min

---

## 1. Chronologie observée

| Temps | Événement observé | Interprétation |
| --- | --- | --- |
| 00:00 | Regarde le menu 4 s, cherche un bouton | Le bouton "Jouer" manque de contraste |
| 00:06 | Lance la partie, ne bouge pas pendant 2 s | N'a pas compris que le panier suit le doigt |
| 00:09 | Touche l'écran au hasard, le panier bouge | Découvre la commande par accident |
| 00:31 | Rattrape 8 œufs d'affilée, sourit | La boucle courte fonctionne |
| 00:44 | Attrape une pierre, perd une vie, dit "ah ?" | N'a pas identifié la pierre comme dangereuse |
| 01:02 | Attrape une seconde pierre | Le problème se répète : signal visuel insuffisant |
| 01:20 | Regarde le compteur de vies pour la 1re fois | Le HUD n'attire pas l'œil lors de la perte |
| 01:38 | Game over, score 320 | Partie plus courte que la cible de 60-180 s |
| 01:41 | Appuie immédiatement sur "Rejouer" | La boucle longue fonctionne |
| 03:55 | 2e partie, score 510, aucune pierre attrapée | Apprentissage rapide une fois le danger compris |
| 06:10 | 3e partie, score 470 | Plateau atteint |
| 10:30 | Repose le téléphone de lui-même | Durée de session naturelle : environ 10 min |

---

## 2. Indicateurs chiffrés

| Indicateur | Valeur |
| --- | --- |
| Temps avant la première action | 9 s |
| Temps pour comprendre la commande | 9 s |
| Temps pour comprendre l'objectif | 12 s |
| Temps pour identifier la pierre comme dangereuse | 62 s |
| Nombre de parties jouées | 4 |
| Meilleur score atteint | 510 |
| Durée de la meilleure partie | 118 s |
| A demandé à rejouer sans qu'on le lui dise | oui, 3 fois |
| Durée totale de jeu | 10 min 30 |

---

## 3. Points de blocage

| Lieu / moment | Ce qui s'est passé | Hypothèse de cause |
| --- | --- | --- |
| Menu | 4 s pour trouver le bouton | Bouton gris sur fond gris, contraste insuffisant |
| Début de partie | 9 s sans bouger | Aucune indication de la commande |
| 1re pierre | Attrapée sans hésiter | La pierre est grise, l'œuf blanc : trop proches |

---

## 4. Comportements non prévus

| Ce que le testeur a fait | Prévu ? | À exploiter ? |
| --- | --- | --- |
| A gardé le doigt posé en permanence, sans le lever | non | Oui : optimiser le suivi continu plutôt que le tap |
| A tenu le téléphone à deux mains, pouce droit uniquement | non | Oui : la zone utile est la moitié droite de l'écran |
| A essayé d'incliner le téléphone au début | non | Non : hors du lot, pas d'accéléromètre |

---

## 5. Verbatim

- "Ah, ça suit le doigt en fait."
- "J'ai pas vu que c'était un caillou."
- "Encore une."

---

## 6. Réponses aux questions finales

1. Ce que tu as fait : "J'attrapais les trucs blancs, faut éviter les gris."
2. Moment sans savoir quoi faire : "Au tout début, je savais pas comment bouger."
3. Ce qui t'a agacé : "Les cailloux, on les voit pas assez."
4. Rejouerais-tu : "Ouais, dans le bus."
5. Une seule chose à changer : "Mettre les cailloux d'une autre couleur."

---

## 7. Décisions prises

| Problème | Fréquence | Décision | Statut |
| --- | --- | --- | --- |
| Pierre non identifiée comme dangereuse | T01 (à confirmer) | Corriger : pierre en rouge foncé + forme hexagonale marquée + son grave | à faire |
| Commande non comprise au début | T01 (à confirmer) | Corriger : main animée pendant les 3 premières secondes | à faire |
| Bouton "Jouer" peu visible | T01 (à confirmer) | Corriger : fond jaune, texte sombre | à faire |
| Perte de vie peu perceptible | T01 (à confirmer) | Corriger : flash rouge de l'écran + vibration courte | à faire |
| Souhaite l'accéléromètre | T01 | Ignorer : hors du lot | clos |

## 8. Modifications d'équilibrage

| Valeur | Avant | Après | Raison |
| --- | --- | --- | --- |
| Aucune | | | Attendre T02 et T03 : la difficulté n'a pas posé problème |
```

```text
   PROTOTYPE PAPIER — Chute Libre, première partie
   Grille : 9 colonnes × 16 lignes. 1 case = 80 px logiques.

   col   1  2  3  4  5  6  7  8  9
       ┌──┬──┬──┬──┬──┬──┬──┬──┬──┐
   L1  │  │  │O │  │  │  │  │P │  │   apparition, t = 0,0 s
   L2  │  │  │  │  │  │  │  │  │  │
   L3  │  │  │  │  │O │  │  │  │  │   apparition, t = 1,4 s
   L4  │  │  │  │  │  │  │  │  │  │
   L5  │  │  │  │  │  │  │  │  │  │
   L6  │  │  │  │  │  │  │  │  │  │
   L7  │  │  │  │  │  │  │  │  │  │
   L8  │  │  │  │  │  │  │  │  │  │
   L9  │  │  │  │  │  │  │  │  │  │
   L10 │  │  │  │  │  │  │  │  │  │
   L11 │  │  │  │  │  │  │  │  │  │
   L12 │  │  │  │  │  │  │  │  │  │
   L13 │  │  │  │  │  │  │  │  │  │
   L14 │  │  │[==PANIER==]  │  │  │   le panier occupe 3 colonnes
   L15 │  │  │  │  │  │  │  │  │  │
   L16 │▓▓│▓▓│▓▓│▓▓│▓▓│▓▓│▓▓│▓▓│▓▓│   sol : un œuf touché ici = -1 vie
       └──┴──┴──┴──┴──┴──┴──┴──┴──┘

   O = œuf     P = pierre     [====] = panier (3 colonnes)

   FICHE DE PORTÉE DU JOUEUR
   ─────────────────────────────────────────────
   largeur du panier ......... 3 colonnes = 240 px
   vitesse de déplacement .... instantanée (suit le doigt)
   temps de chute (palier 0) . 16 lignes × 80 px / 180 px/s = 7,1 s
   temps de chute (palier 6) . 16 lignes × 80 px / 350 px/s = 3,7 s
   écart maximal à couvrir ... 8 colonnes = 640 px
   temps disponible minimal .. 3,7 s au palier 6

   SIMULATION À LA MAIN, 30 PREMIÈRES SECONDES
   ─────────────────────────────────────────────
   t=0,0  un œuf col.3 et une pierre col.8 partent ensemble
   t=1,4  un œuf col.5 part
   t=2,8  un œuf col.1 part
   t=7,1  l'œuf col.3 arrive : le panier est en col.3-5, il l'attrape (+10)
   t=8,5  l'œuf col.5 arrive : panier déjà en place, il l'attrape (+10)
   t=9,9  l'œuf col.1 arrive : le panier doit couvrir 2 colonnes, faisable

   PROBLÈMES CONSTATÉS SUR PAPIER
   ─────────────────────────────────────────────
```

```markdown
| Problème constaté | Correction décidée |
| --- | --- |
| Un œuf et une pierre partant ensemble aux colonnes 3 et 8 ne se contredisent jamais : le panier n'a aucun dilemme | Interdire deux apparitions simultanées écartées de plus de 4 colonnes : les objets simultanés doivent créer un choix |
| Au palier 0, 7,1 s de chute pour un objet toutes les 1,4 s donne 5 objets à l'écran en permanence | Réduire la hauteur utile : faire apparaître les objets à la ligne 4, pas à la ligne 1. Temps de chute ramené à 5,3 s |
| Le panier de 3 colonnes couvre un tiers de la largeur : trop généreux | Réduire à 2 colonnes (160 px) au-dessus du palier 3 |
| Un œuf en colonne 1 et un œuf en colonne 9 simultanés sont tous deux rattrapables au palier 0, mais impossibles au palier 6 | Comportement voulu : c'est la source naturelle de la fin de partie. Aucune correction |
| Aucune récompense visuelle pour une série d'œufs consécutifs | Reporté en v1.1 : un multiplicateur de série serait une nouvelle mécanique, hors du lot |
```

**Explication :** ce dernier livrable montre les deux outils à l'œuvre sur le même jeu, et leur complémentarité. La grille de playtest capte ce que le papier ne peut pas voir : le testeur n'a pas compris la commande en 9 secondes, et n'a pas identifié la pierre comme dangereuse avant 62 secondes. Ces deux problèmes sont des problèmes de **sensation et de lisibilité**, invisibles sur papier. Inversement, le prototype papier a révélé en dix minutes, et sans une ligne de code, que deux objets simultanés très écartés ne créent aucun dilemme, et que le panier de trois colonnes est trop généreux — deux problèmes de **règles**, qui auraient demandé plusieurs playtests pour être diagnostiqués. Notez enfin la colonne « fréquence » de la section 7 : toutes les décisions portent la mention « à confirmer », parce qu'un seul test ne fait pas un fait (section 41.32). Seule la demande d'accéléromètre est close immédiatement, non parce qu'elle est isolée, mais parce qu'elle est hors du lot.

---

## Et maintenant ?

Vos documents sont écrits. Vous avez un pitch, un cahier des charges, un Game Design Document, une architecture, un planning, un tableau de licences et une grille de playtest. Vous savez exactement ce que vous allez construire, en combien de temps, et à quoi vous reconnaîtrez que c'est fini.

Il reste à savoir **livrer**. Un jeu qui tourne sur votre machine n'est pas un jeu publié : il faut le tester automatiquement pour ne pas casser ce qui marchait, mesurer ses performances réelles sur un appareil, réduire sa taille, puis produire un APK Android et un build Web que d'autres personnes peuvent réellement lancer.

C'est l'objet du dernier chapitre de la formation.

[42-PARTIE-2D—TESTS-PERFORMANCE-ET-BUILD-ANDROID-WEB.md](./42-PARTIE-2D—TESTS-PERFORMANCE-ET-BUILD-ANDROID-WEB.md)
