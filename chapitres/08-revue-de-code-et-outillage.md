[← Détecter et refactorer](07-detecter-et-refactorer.md) · [↑ Sommaire](../README.md#table-des-matières) · [Pragmatisme et limites →](09-pragmatisme-et-limites.md)

# 8. Revue de code et outillage

## Revue de code : hygiène et fond

> **Que veut dire « revue de code » et « linter » ?** Une *revue de code* (*code review*) est l'examen d'une modification par un ou plusieurs collègues avant de l'intégrer : on relit le travail d'un autre comme on relit le brouillon d'un texte, pour repérer erreurs et améliorations. Un *linter* est un outil qui contrôle automatiquement le respect des règles de style (indentation, espaces, conventions) ; il prend en charge la forme pour que les humains se concentrent sur le fond.

Une revue de code (*code review*) sert à **diffuser la connaissance** et à **détecter les défauts d'intention**. Trop souvent elle dégénère en débat de ponctuation. Quelques règles tirées des écrits de Michael Lynch, Gergely Orosz et de la culture Google.

### Ce qu'une revue doit chercher (par ordre de priorité)

1. **L'intention est-elle juste ?** Le changement résout-il bien le problème énoncé ? Cas-limites couverts ?
2. **L'architecture est-elle saine ?** Le code se trouve-t-il à la bonne couche ? La dépendance va-t-elle dans le bon sens ?
3. **Les tests prouvent-ils ce qu'ils prétendent prouver ?** Une suite verte qui n'assertionne rien ne vaut rien.
4. **La lisibilité est-elle suffisante ?** Un nouveau venu comprendra-t-il dans six mois ?
5. **Les détails de style.** En dernier, et idéalement délégués au linter.

Inverser cet ordre, en discutant d'abord de l'emplacement d'une accolade, est l'erreur la plus fréquente. Le linter règle la forme ; l'humain règle le fond.

### Hygiène : ce qui rend une revue supportable

> **Que veut dire « nitpick », « bikeshedding », « merge » ?** Un *nitpick* (« pinaillage ») est une remarque sur un détail mineur, sans réel enjeu. Le *bikeshedding* (« effet garage à vélos ») désigne la tendance à débattre sans fin de questions triviales (la couleur du garage) tout en survolant les questions importantes (le réacteur nucléaire à côté), parce que chacun a un avis facile sur le détail. *Merger* (« fusionner »), c'est intégrer enfin une modification validée dans la base de code commune.

| Mauvaise pratique | Symptôme | Remède |
|--------------------|----------|--------|
| **Pile de nitpicks** | 30 commentaires sur des renommages, un seul sur l'archi. | Préfixer les remarques mineures par `nit:` (les indiquer comme optionnelles). Automatiser le reste. |
| **Commentaire ambigu** | « Ce n'est pas terrible. » | Toujours dire **pourquoi** et **proposer** : « Ici, deux responsabilités ; extraire un `Validator` ? » |
| **Aller-retour sans fin** | La PR vit deux semaines, le contexte s'efface. | Mettre une **borne** : 2 cycles de revue, puis appel ou *pair review* en synchrone. |
| **Bikeshedding** | Discussion infinie sur le nom d'une variable triviale. | Auteur tranche, revue lève la main si elle change d'avis sous 24 h. |
| **Revue tardive** | La PR fait 2000 lignes ; la revue devient symbolique. | Limiter à **400 lignes** par PR (étude SmartBear). Au-delà, scinder. |
| **Revue par autorité** | Le senior impose son avis sans justification. | Toute remarque doit s'argumenter. *Strong opinions, weakly held* : des avis tranchés, mais qu'on lâche dès qu'un meilleur argument apparaît. |

### Le langage des commentaires de revue

Le ton compte autant que le contenu. Quelques préfixes utilisés chez Google, Microsoft, Stripe :

| Préfixe | Sens |
|---------|------|
| `nit:` | Détail mineur, l'auteur peut ignorer sans justifier. |
| `question:` | Demande de clarification, pas une demande de changement. |
| `suggestion:` | Idée alternative, à débattre. |
| `blocking:` | Doit être traité avant merge (correction de bug, faille, contrat cassé). |
| `praise:` | Compliment explicite. **Important :** une revue qui ne sait que critiquer démotive. |

### Ce qui n'est pas de la revue de code

* Un **chantier d'architecture** se discute ailleurs (ADR, RFC interne, design doc).
* Un **différend de style** se règle dans la configuration du linter, pas dans la PR.
* Un **désaccord profond** se traite en synchrone (5 minutes de visio valent 50 messages).

[🔝 Retour en haut de page](#table-des-matières)

## Outillage PHP : du linter au mutation testing

> **Que veut dire « mutation testing » ?** Le *mutation testing* (« test par mutation ») vérifie la **qualité des tests** eux-mêmes. L'outil introduit volontairement de petites erreurs dans le code (par exemple remplacer un `+` par un `-`) et regarde si au moins un test échoue. Si rien ne casse, c'est que les tests ne couvraient pas vraiment ce comportement. Analogie : un inspecteur qui débranche discrètement un détecteur de fumée pour vérifier que l'alarme se déclenche.

Aucun de ces outils ne remplace la rigueur humaine. Tous l'amplifient en transformant des règles écrites en vérifications automatiques et reproductibles.

### Pyramide d'outillage

> **Que veut dire « analyse statique » et « couverture de tests » ?** L'*analyse statique* examine le code **sans l'exécuter** pour repérer des erreurs probables (un type incohérent, une variable jamais utilisée, un cas non géré), comme un correcteur orthographique lit un texte sans le jouer. La *couverture de tests* (en anglais *coverage*) mesure le pourcentage de lignes du code effectivement parcourues par les tests. Attention : une ligne parcourue n'est pas une ligne **vérifiée** ; on peut traverser du code sans rien affirmer dessus.

| Niveau | Outil | Ce qu'il garantit | Coût d'intégration |
|--------|-------|-------------------|--------------------|
| 1. Style | [PHP-CS-Fixer](https://github.com/PHP-CS-Fixer/PHP-CS-Fixer), [PHP_CodeSniffer](https://github.com/PHPCSStandards/PHP_CodeSniffer) | Conformité PSR-12, cohérence stylistique. | Faible. À brancher en pré-commit. |
| 2. Statique de typage | [PHPStan](https://phpstan.org/), [Psalm](https://psalm.dev/) | Types corrects, branches mortes, *null* non géré, tableaux mal indexés. | Moyen. Démarrer au niveau 0, monter palier par palier. |
| 3. Refactoring automatisé | [Rector](https://getrector.com/) | Migration de versions PHP, application en masse de patterns (constructeurs promus, `readonly`, *final*, etc.). | Moyen. Indispensable pour les vieux projets. |
| 4. Complexité et architecture | [PHPMD](https://phpmd.org/), [Deptrac](https://github.com/qossmic/deptrac), [PhpMetrics](https://phpmetrics.org/) | Complexité cyclomatique, dépendances inter-modules, cohésion. | Moyen. Surtout pour cadrer une équipe. |
| 5. Couverture | [PHPUnit](https://phpunit.de/) `--coverage` | Lignes / branches exécutées par les tests. | Faible. Ne dit **pas** si les assertions sont pertinentes. |
| 6. Mutation testing | [Infection](https://infection.github.io/) | Modifie volontairement le code et vérifie que les tests détectent. Mesure la *qualité* des tests, pas leur quantité. | Élevé. Lent, mais révélateur sur les zones critiques. |
| 7. Sécurité | [composer audit](https://getcomposer.org/), [Roave Security Advisories](https://github.com/Roave/SecurityAdvisories) | Détection des CVE dans les dépendances. | Très faible. Aucun prétexte. |

### Exemple de pipeline minimal en CI

> **Que veut dire « pipeline » ?** Un *pipeline* (« chaîne de traitement ») est une suite d'étapes automatiques exécutées dans l'ordre, chacune ne démarrant que si la précédente a réussi : installer les dépendances, vérifier le style, lancer les tests, contrôler la sécurité. Comme une chaîne de montage, si une étape échoue, la chaîne s'arrête et signale le problème.

```yaml
# .github/workflows/ci.yml : squelette représentatif
jobs:
  qualite:
    steps:
      - run: composer install --no-progress
      - run: vendor/bin/php-cs-fixer fix --dry-run --diff
      - run: vendor/bin/phpstan analyse --level=8
      - run: vendor/bin/phpunit --coverage-text
      - run: composer audit
      # Mutation testing : seulement sur la branche principale (lent)
      # - run: vendor/bin/infection --min-msi=70
```

### Lecture critique : les pièges classiques

* **PHPStan niveau 8 du jour 1 sur une base de code héritée** = mur infranchissable. Mieux : commencer au niveau 0, ne jamais régresser, monter d'un cran par sprint.

> **Que veut dire « code hérité » (legacy) et « sprint » ?** Le *code hérité* (en anglais *legacy*) est le code ancien dont on a hérité : il fonctionne et tourne en production, mais il est souvent peu testé et difficile à modifier sans risque. Un *sprint* est une période de travail courte et fixe (souvent une à deux semaines) au bout de laquelle l'équipe livre un incrément ; c'est un rythme issu des méthodes agiles.
* **100 % de couverture** ne signifie pas 100 % de comportements vérifiés. Une assertion faible sur du code couvert masque un bug. *Coverage* est un *minimum nécessaire*, jamais un objectif suffisant.
* **Rector appliqué sans tests** est un terrain de mines. C'est l'outil qui justifie le plus d'écrire des tests **avant** de l'utiliser.
* **Linter trop strict** (200 règles activées) finit ignoré. Démarrer minimal, ajouter une règle quand l'équipe la juge utile.

### Value Objects : un cas pratique d'application

L'**obsession des primitifs** (smell `Primitive Obsession`) se règle outillée : un Value Object par concept métier qui mérite une invariante.

| Concept | Type primitif | Value Object | Invariants encapsulés |
|---------|---------------|--------------|------------------------|
| Adresse mail | `string` | `Email` | Format RFC, casse normalisée. |
| IBAN | `string` | `Iban` | Longueur par pays, *checksum* MOD-97. |
| Montant | `float` | `Money` | Précision, devise, arithmétique sans erreur d'arrondi. |
| Identifiant produit | `string` | `Sku` | Pattern domaine (`AAA-000-X`). |
| Coordonnée | `float` × 2 | `LatLng` | Latitude ∈ [−90, 90], longitude ∈ [−180, 180]. |
| Date métier | `DateTimeImmutable` | `JourOuvre`, `Echeance` | Pas de week-end, pas de jour férié. |

Bénéfice : la connaissance se concentre dans le type, **toute fonction qui en reçoit un peut le présupposer valide**. Les contrôleurs maigrissent, le domaine s'enrichit.

[🔝 Retour en haut de page](#table-des-matières)

---

[← Détecter et refactorer](07-detecter-et-refactorer.md) · [↑ Sommaire](../README.md#table-des-matières) · [Pragmatisme et limites →](09-pragmatisme-et-limites.md)
