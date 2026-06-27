[← Revue de code et outillage](08-revue-de-code-et-outillage.md) · [↑ Sommaire](../README.md#table-des-matières)

# 9. Pragmatisme et limites

## Quand ne pas appliquer Clean Code

*Clean Code* est un investissement : on paie un coût aujourd'hui (réflexion, abstractions, tests) pour économiser demain (maintenance, évolutions). L'investissement n'a de sens que si **demain existe**. Voici les cas où l'appliquer rigoureusement est un mauvais calcul.

### Code par essence éphémère

| Contexte | Pourquoi assouplir |
|----------|--------------------|
| **Spike technique** (étude de faisabilité) | Le but est de répondre *oui/non* le plus vite possible ; le code finit à la poubelle. |
| **Script de migration unique** | Tournera une fois en prod, sera supprimé. Surinvestir est gaspillé. |
| **Notebook de data analysis** | Le livrable est l'analyse, pas le code. Lisibilité linéaire suffit. |
| **POC démo** pour une présentation client | La démo dure 20 minutes ; ce qui compte est le rendu. |
| **Code de concours / katas** | Optimiser pour la performance ou la concision, pas pour l'évolutivité. |

> **Que veut dire « POC », « notebook », « kata » ?** Un *POC* (*Proof Of Concept*, « preuve de concept ») est une maquette rapide servant à prouver qu'une idée est réalisable, sans viser la qualité finale. Un *notebook* (« carnet ») est un document interactif mêlant code et résultats, très utilisé en analyse de données, où l'on déroule un raisonnement de haut en bas. Un *kata* (terme emprunté aux arts martiaux) est un petit exercice de programmation répété pour s'entraîner ; le résultat n'a pas vocation à durer.

**Piège classique :** un *spike* qu'on garde « juste pour cette fois ». La règle : si un code éphémère survit à sa raison d'être, ou s'il atterrit sur la branche principale, il **redevient** soumis aux exigences Clean Code, et il faut le refactoriser ou le réécrire.

### Code dont la lisibilité serait contre-productive

> **Que veut dire « hot-path », « goulot », « mémoïsation », « bench » ?** Un *hot-path* (« chemin chaud ») est la portion de code exécutée extrêmement souvent, donc déterminante pour la vitesse globale. Un *goulot* (d'étranglement, en anglais *bottleneck*) est l'endroit précis qui ralentit tout le reste, comme le col d'une bouteille qui limite le débit. La *mémoïsation* consiste à retenir le résultat d'un calcul coûteux pour le ressortir au lieu de le refaire. Un *bench* (de *benchmark*) est une mesure chiffrée de performance qui sert de preuve avant d'optimiser.

* **Hot-path optimisé** : une boucle critique mesurée comme goulot peut justifier une forme moins lisible (`for` plutôt que `foreach` + `array_map`, mémoïsation manuelle, accès direct à un attribut). Le commentaire devient indispensable : il explique *pourquoi* on a sacrifié la lisibilité (et idéalement, le bench qui le justifie).
* **Algorithmes mathématiques** : une transformée de Fourier rapide ou un produit matriciel SIMD a une forme contrainte par les performances. Imposer des fonctions de 5 lignes y casse le regroupement logique. La lisibilité passe par les **commentaires** et les **références aux papiers**, pas par l'extraction.
* **Code généré** (parser, ORM, gRPC, OpenAPI) : ne pas le toucher à la main. Les conventions Clean Code ne s'y appliquent pas ; la source de vérité est ailleurs.

> **Que veut dire « SIMD » et « gRPC » ?** *SIMD* (*Single Instruction, Multiple Data*) est une technique du processeur qui applique une même opération à plusieurs données d'un coup, pour aller plus vite (comme tamponner dix enveloppes en un seul geste). *gRPC* est un protocole de communication entre programmes, à partir duquel des outils **génèrent** automatiquement du code : on ne modifie pas ce code à la main, on régénère depuis la source.

### Code « assez bon » dans un contexte de contrainte

Une équipe en sous-effectif, une échéance contractuelle (un *deadline*, la date limite imposée par contrat), une jeune entreprise qui n'a pas encore de chiffre d'affaires (en anglais *pre-revenue*) et doit prouver son idée avant de viser la qualité. Dans ces contextes, **livrer un produit moyennement propre** vaut mieux que **ne pas livrer un produit parfait**. La dette est consciente, datée, et fait l'objet d'un ticket de remboursement. C'est le quadrant **prudent + délibéré** de la matrice de Fowler (voir [Dette technique](#dette-technique)).

### Le test ultime

Avant de plaider l'exception, se poser trois questions :

1. **Combien de fois** ce code sera-t-il lu ? (Pas écrit, *lu*.)
2. **Combien de personnes** différentes le maintiendront-elles ?
3. **Quelle est la durée de vie raisonnable** du logiciel auquel il appartient ?

Si les trois réponses pointent vers « peu », assouplir est légitime. Si l'une seulement pointe vers « beaucoup », appliquer Clean Code reste rentable. Et si on hésite, c'est généralement que l'investissement est rentable : l'incertitude joue contre le code sale.

[🔝 Retour en haut de page](#table-des-matières)

## KISS, YAGNI et optimisation prématurée

Trois maximes qui poussent dans la même direction : **ne pas écrire ce qui n'est pas nécessaire**.

### KISS : *Keep It Simple, Stupid*

> **Que veut dire « KISS » ?** *KISS* signifie *Keep It Simple, Stupid* (« reste simple, idiot »). C'est un rappel à choisir la solution la plus simple qui répond au besoin **d'aujourd'hui**, plutôt qu'une usine à gaz qui anticipe des besoins imaginaires. Plus une mécanique est simple, moins elle tombe en panne et plus elle est facile à comprendre.

Adage attribué à l'ingénieur Kelly Johnson dans les années 1960. La solution la plus simple qui résout le problème **actuel** est presque toujours la bonne. Une abstraction supplémentaire est un pari sur l'avenir, et la plupart des paris perdent.

```php
// Trop : Strategy + Factory + Registry pour une condition à deux cas
$politique = (new PolitiqueRemiseFactory($registry))->depuisType($commande->type());
$remise    = $politique->appliquer($commande);

// Assez : un `match` clair et localisé ; on extraira si la liste grandit.
$remise = match ($commande->type()) {
    'STANDARD' => 0.0,
    'FIDELE'   => $commande->total() * 0.05,
};
```

> **Que veut dire « design pattern » (Strategy, Factory, Registry) ?** Un *design pattern* (« patron de conception ») est une solution type, reconnue et réutilisable, à un problème d'organisation du code. *Strategy* permet d'échanger un algorithme à la volée, *Factory* centralise la création d'objets, *Registry* tient un annuaire d'objets accessibles globalement. Ce sont des outils utiles, mais les empiler pour un cas trivial (ici deux possibilités) est l'exemple même d'une complexité gratuite que KISS condamne.

L'extraction sera triviale **le jour où** un troisième cas apparaît, et probablement mieux conçue car informée par trois cas réels au lieu de deux imaginaires.

### YAGNI : *You Aren't Gonna Need It*

> **Que veut dire « YAGNI » et « XP » ?** *YAGNI* signifie *You Aren't Gonna Need It* (« vous n'en aurez pas besoin »). C'est l'invitation à ne pas coder une fonctionnalité tant qu'un besoin réel ne l'exige pas, car on devine mal l'avenir et toute option ajoutée coûte à entretenir. *XP* abrège *Extreme Programming* (« programmation extrême »), une méthode de développement agile qui pousse à l'extrême quelques bonnes pratiques (tests, binômes, livraisons fréquentes), et d'où vient ce principe.

Formulé par Ron Jeffries dans le contexte XP. Trois raisons de ne pas anticiper :

1. **On se trompe** sur le besoin futur dans la majorité des cas.
2. **On paie tout de suite** la complexité (lecture, tests, documentation).
3. **On contraint** les solutions ultérieures qui doivent désormais cohabiter avec l'ancienne abstraction.

```php
// À éviter : des paramètres "au cas où" qu'aucun appelant ne fournit
public function calculerTotal(
    Commande $commande,
    ?string $devise = null,
    ?float $tauxChange = null,
    ?bool $arrondir = false,
    ?int $precision = 2,
): float { /* ... */ }

// À préférer : ce dont on a besoin aujourd'hui ; on étendra le jour où c'est requis.
public function calculerTotal(Commande $commande): Montant { /* ... */ }
```

### Optimisation prématurée

> **Que veut dire « optimisation prématurée » ?** *Optimiser* du code, c'est le rendre plus rapide ou plus économe. C'est *prématuré* quand on le fait **avant d'avoir mesuré** où se trouve réellement la lenteur : on complique le code pour un gain souvent inexistant. La phrase de Knuth résume le danger : on devine très mal d'où vient la lenteur, et on sacrifie la lisibilité pour rien.

> *« L'optimisation prématurée est la racine de tous les maux. »* (Donald Knuth)

> **Que veut dire « profiling » et « SLA » ?** Le *profiling* (« profilage ») consiste à mesurer où le programme passe réellement son temps, à l'aide d'outils dédiés, afin d'identifier le vrai goulot avant d'optimiser. Un *SLA* (*Service Level Agreement*, « accord de niveau de service ») est un engagement chiffré sur la qualité d'un service, par exemple « 99 % des pages répondent en moins de 300 millisecondes » ; il indique objectivement quand « c'est trop lent ».

L'optimisation prématurée n'est pas « faire attention aux performances » ; c'est **tordre le code pour un gain non mesuré**. La discipline :

1. **Mesurer d'abord** (profiling : Xdebug, Blackfire, `microtime` ciblé).
2. **Optimiser ensuite** uniquement le goulot identifié.
3. **Garder la version lisible** en commentaire ou en branche, pour comprendre demain.

Le bon moment pour optimiser, c'est quand un test de performance, un SLA ou un utilisateur dit clairement : *ici, c'est trop lent*.

[🔝 Retour en haut de page](#table-des-matières)

## Dette technique

> **Que veut dire « dette technique » ?** La *dette technique* est l'image qui compare un raccourci pris dans le code à un emprunt d'argent : il rend service tout de suite (on livre plus vite), mais il se rembourse plus tard avec des « intérêts » (le code devient plus dur à modifier). Comme une dette financière, elle n'est pas mauvaise en soi : elle devient dangereuse quand on l'ignore et qu'elle s'accumule sans plan de remboursement.

Métaphore de Ward Cunningham (1992) : un raccourci de conception est un emprunt qui rend service maintenant et coûte plus cher demain. La dette n'est pas honteuse : c'est un outil, à condition de la connaître et de la rembourser.

### Quatre quadrants de Martin Fowler

| | Imprudent (*reckless*) | Prudent (*prudent*) |
|--|------------------------|---------------------|
| **Délibéré** (*deliberate*) | « On livre, on nettoiera plus tard. » Souvent jamais. | « On sait que ce n'est pas idéal, voici pourquoi, voici quand on rembourse. » |
| **Involontaire** (*inadvertent*) | « On ne savait pas qu'il existait des design patterns. » | « Maintenant qu'on a livré, on voit comment on aurait dû s'y prendre. » |

Seul le quadrant *prudent + délibéré* est sain : la dette est un choix éclairé, tracé, planifié.

### Symptômes de dette qui dérape

* Chaque changement, même mineur, casse une partie inattendue.
* L'estimation des fonctionnalités s'allonge sans raison apparente.
* Les nouveaux développeurs mettent des semaines à devenir productifs.
* Personne n'ose toucher à un module crucial.
* Les tests sont régulièrement désactivés « parce qu'ils sont flaky ».

> **Que veut dire « flaky » ?** Un test *flaky* (« instable ») passe parfois et échoue parfois sans qu'on ait rien changé au code, souvent à cause d'un facteur incontrôlé (l'heure, l'ordre d'exécution, le réseau). Pire que pas de test : il use la confiance, et l'équipe finit par ignorer ses alertes, y compris les vraies.

### Stratégies de remboursement

| Stratégie | Quand l'employer |
|-----------|------------------|
| **Boy scout rule** | Refactoriser à la marge chaque fichier touché par une feature. Indolore, continu. |
| **Refactoring planifié** | Bloquer 10 à 20 % du temps de sprint pour des refactorings ciblés. |
| **Strangler Fig pattern** | Remplacer un module hérité morceau par morceau, sans big bang. |
| **Réécriture totale** | Dernier recours. Coûteuse, risquée, longtemps non livrable. À justifier. |

> **Que veut dire « Strangler Fig » et « big bang » ?** Le *Strangler Fig pattern* (« motif du figuier étrangleur ») tire son nom d'une plante qui pousse autour d'un arbre et finit par le remplacer. En code, on entoure peu à peu l'ancien système d'un nouveau qui reprend ses fonctions une par une, jusqu'à pouvoir retirer l'ancien sans coupure. À l'inverse, un remplacement *big bang* (« grand boum ») bascule tout d'un coup vers le nouveau système : spectaculaire, mais très risqué car on découvre tous les problèmes en même temps.

### Tracer la dette

Une dette invisible n'est pas remboursée. Outils utiles :

* Tickets « TECH-DEBT » étiquetés dans le suivi.
* Fichier `ARCHITECTURE.md` ou *Architecture Decision Records* (ADR) qui documentent les compromis.
* Indicateurs : couverture de tests, complexité cyclomatique, durée moyenne d'une PR, tous suivis dans le temps.

[🔝 Retour en haut de page](#table-des-matières)

---

[← Revue de code et outillage](08-revue-de-code-et-outillage.md) · [↑ Sommaire](../README.md#table-des-matières)
