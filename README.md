# [Tansoftware](https://www.tansoftware.com) - Clean Code [![fr](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/France.png)](README.md)

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE) [![Lang](https://img.shields.io/badge/Lang-Français-005EB8.svg)](#) [![Topic](https://img.shields.io/badge/Topic-Clean%20Code-brightgreen.svg)](#) [![Examples](https://img.shields.io/badge/Examples-PHP-777BB4.svg)](#)

## Table des matières

* [Introduction](#introduction)
* [Glossaire](#glossaire)
* [Nommage des variables](#nommage-des-variables)
* [Commentaires](#commentaires)
* [Mise en forme du code (formatage)](#mise-en-forme-du-code-formatage)
* [Fonctions courtes et spécifiques](#fonctions-courtes-et-spécifiques)
* [Respect des conventions de codage](#respect-des-conventions-de-codage)
* [Duplication](#duplication)
* [Objets vs structures de données](#objets-vs-structures-de-données)
* [Loi de Déméter et Tell, Don't Ask](#loi-de-déméter-et-tell-dont-ask)
* [Fonctions pures et effets de bord](#fonctions-pures-et-effets-de-bord)
* [Utilisation de tests unitaires](#utilisation-de-tests-unitaires)
* [Tests : principes F.I.R.S.T. et TDD](#tests--principes-first-et-tdd)
* [Documentation de code](#documentation-de-code)
* [Gestion des erreurs et des exceptions](#gestion-des-erreurs-et-des-exceptions)
* [Structure du code claire et organisée](#structure-du-code-claire-et-organisée)
* [Gestion des dépendances](#gestion-des-dépendances)
* [Gestion de la complexité du code](#gestion-de-la-complexité-du-code)
* [Les fonctions doivent faire une seule chose](#les-fonctions-doivent-faire-une-seule-chose)
* [Code smells (mauvaises odeurs de code)](#code-smells-mauvaises-odeurs-de-code)
* [Catalogue de refactorings essentiels](#catalogue-de-refactorings-essentiels)
* [KISS, YAGNI et optimisation prématurée](#kiss-yagni-et-optimisation-prématurée)
* [Dette technique](#dette-technique)
* [Pour aller plus loin](#pour-aller-plus-loin)

## Introduction

Le terme *Clean Code* (« code propre ») a été popularisé par [Robert C. Martin](https://fr.wikipedia.org/wiki/Robert_C._Martin) dans son livre *Clean Code: A Handbook of Agile Software Craftsmanship* (2008). Il regroupe un ensemble de pratiques qui rendent le code plus lisible, plus facile à modifier et moins coûteux à maintenir.

Un code est dit *propre* lorsqu'un développeur autre que son auteur peut le lire, le comprendre et le faire évoluer sans avoir à mener l'enquête. Ce dépôt présente les pratiques qui contribuent à cet objectif, illustrées par des exemples en PHP.

> *« La règle du boy scout : laissez le campement plus propre que vous ne l'avez trouvé. »* — Robert C. Martin
>
> Appliquée au code, cette règle pousse à améliorer un peu chaque fichier que l'on touche, plutôt que d'attendre une refonte hypothétique.

[🔝 Retour en haut de page](#table-des-matières)

## Glossaire

Ce vocabulaire revient dans tout le document. Le lire en premier évite d'y revenir à chaque chapitre.

| Terme | Définition courte | Pourquoi c'est utile |
|-------|-------------------|----------------------|
| **SRP** (*Single Responsibility Principle*, principe de responsabilité unique) | Une classe ou une fonction ne doit avoir qu'**une seule raison de changer**. | Un module qui répond à plusieurs raisons de changer concentre des risques de régression à chaque évolution. |
| **Cohésion** | Mesure du degré auquel les éléments d'un module (méthodes, attributs) participent à une **même finalité**. Forte cohésion = bon signe. | Une classe cohésive est facile à nommer, à tester, à comprendre. |
| **Couplage** | Mesure de la dépendance entre deux modules. Faible couplage = bon signe. | Moins deux modules se connaissent, plus on peut les modifier indépendamment. |
| **Magic number** (nombre magique) | Valeur littérale (`30`, `0.2`, `0.07`) inscrite dans le code sans nom ni explication. | Les remplacer par une constante nommée (`DELAI_PURGE_JOURS`, `TAUX_TVA`) rend l'intention explicite. |
| **Code smell** (mauvaise odeur de code) | Indice visuel qu'une portion de code mériterait un refactoring. Une *odeur* n'est pas un bug : c'est un signal. | Catalogue popularisé par Martin Fowler et Kent Beck dans *Refactoring*. |
| **Refactoring** (refactorisation) | Transformation **sans changement de comportement** observable, qui améliore la structure interne. | Permet d'améliorer la lisibilité ou la conception sans risque fonctionnel — à condition d'avoir des tests. |
| **DRY** (*Don't Repeat Yourself*) | Une règle ou une connaissance ne doit avoir qu'une seule source faisant autorité dans le système. | Évite la dérive entre copies et la duplication de la maintenance. |
| **KISS** (*Keep It Simple, Stupid*) | Préférer la solution la plus simple qui répond au besoin **actuel**. | La complexité gratuite est de la dette qui ne rend rien. |
| **YAGNI** (*You Aren't Gonna Need It*) | Ne pas implémenter ce dont on n'a **pas encore** besoin. | Évite le code mort, les options non testées, les chemins jamais empruntés. |
| **Optimisation prématurée** | Performances cherchées avant qu'un profilage ait identifié un goulot réel. Donald Knuth la qualifie de « racine de tout mal ». | Tordre la lisibilité pour un gain non mesuré coûte plus qu'il ne rapporte. |
| **Dette technique** | Coût futur d'un raccourci pris aujourd'hui (mauvais nom, copier-coller, test absent). Métaphore de Ward Cunningham. | Comme la dette financière, elle se rembourse avec des intérêts. |
| **Encapsulation** | Cacher l'état interne d'un objet derrière des méthodes qui contrôlent son accès et ses transitions. | Empêche les modules extérieurs de dépendre de détails internes susceptibles de changer. |
| **Polymorphisme** | Capacité, pour différentes classes partageant la même interface, de répondre à un même appel par leur propre comportement. | Remplace les longues cascades `if/else` ou `switch` par une dispatch implicite. |
| **Effet de bord** (*side effect*) | Toute action d'une fonction au-delà du retour de sa valeur : écrire un fichier, modifier un attribut, journaliser, envoyer un mail. | Les effets de bord rendent les fonctions difficiles à tester et à raisonner ; à isoler. |
| **Fonction pure** | Fonction sans effet de bord dont le résultat ne dépend que des arguments. Mêmes entrées = mêmes sorties. | Trivialement testable, mémoïsable, parallélisable. |
| **Loi de Déméter** | « Ne parle qu'à tes amis directs. » Un objet ne doit pas naviguer dans la structure interne d'un autre via des chaînes `a.b.c.d`. | Réduit le couplage et la fragilité aux changements internes. |
| **Tell, Don't Ask** | Plutôt qu'**interroger** un objet pour décider à sa place, **lui dire** quoi faire. | Concentre le comportement dans l'objet qui détient les données. |
| **CQS** (*Command-Query Separation*) | Une méthode est soit une **commande** (effet, retour `void`) soit une **requête** (lecture sans effet). Pas les deux. | Évite les surprises (« lire change l'état »). |
| **Clause de garde** (*guard clause*) | `if (...) return;` ou `throw` placé en début de fonction pour éliminer un cas particulier. | Aplanit l'imbrication, met le cas nominal au premier niveau. |

[🔝 Retour en haut de page](#table-des-matières)

## Nommage des variables

Un nom doit révéler l'intention : pourquoi cette variable existe-t-elle, et que représente-t-elle ? Un nom mal choisi force le lecteur à reconstruire le contexte ; un nom précis le lui sert directement.

| Catégorie | À éviter | À préférer | Pourquoi |
|-----------|----------|------------|----------|
| Variable simple | `$a`, `$d` | `$age`, `$dureeEnSecondes` | Le nom doit dire *quoi* et *dans quelle unité*. |
| Fonction | `foo()`, `process()` | `calculerTotal()`, `validerEmail()` | Un verbe d'action décrit ce que la fonction fait. |
| Classe | `Manager`, `MyClass` | `CommandeClient`, `FactureRepository` | Un nom (substantif) décrit la responsabilité. |
| Constante | `X = 3` | `MAX_TENTATIVES = 3` | Majuscules et nom décrivant la limite, pas la valeur. |
| Booléen | `$flag`, `$check` | `$estConnecte`, `$peutModifier` | Préfixer par `est`, `a`, `peut` pour signaler un booléen. |
| Tableau | `$arr`, `$data` | `$utilisateurs`, `$lignesFacture` | Un pluriel signale une collection. |
| Argument | `$arg1`, `$x` | `$prenom`, `$montantTtc` | Le nom dans la signature documente l'API. |

### Les sept règles de nommage de *Clean Code* (chapitre 2)

| Règle | Idée | Exemple à éviter | Exemple à préférer |
|-------|------|------------------|---------------------|
| Révéler l'intention | Le nom répond à *pourquoi*, *quoi*, *comment l'utiliser*. | `$d` (jours ? distance ?) | `$delaiAvantPurgeEnJours` |
| Éviter la désinformation | Pas de nom qui ment sur le type ou la nature. | `$accountList` (qui est en fait un `array` non ordonné) | `$comptes` |
| Distinguer significativement | Pas de variantes vides (`data1`, `data2`, `info`, `infoData`). | `getActiveAccount()`, `getActiveAccounts()`, `getActiveAccountInfo()` | `getCompteActif()`, `getComptesActifs()` |
| Prononçable | On doit pouvoir en parler à voix haute. | `$genymdhms` | `$dateDeGeneration` |
| Recherchable | Les noms courts ne se cherchent pas dans un éditeur (`grep e` retourne tout). | `7` (taux de TVA dispersé) | `const TAUX_TVA = 0.20;` |
| Sans encodage | Pas de notation hongroise (`strNom`, `iCount`), pas de préfixe `m_`. PHP est typé : laissez les types au compilateur. | `string $strNom` | `string $nom` |
| Sans mapping mental | Le lecteur ne doit pas avoir à traduire `r` en *« le résultat retourné »*. | `for ($i = 0; …) { $r += $a[$i]; }` | `foreach ($valeurs as $valeur) { $somme += $valeur; }` |

### Quand assouplir

Les noms courts (`i`, `j`, `n`) restent acceptables pour des indices de boucle locale. Les abréviations consacrées dans le domaine (`url`, `id`, `http`) sont également admises ; elles trompent moins qu'une expansion artificielle (`uniformResourceLocator`).

### Magic numbers : nommer les littéraux

```php
// À éviter
if ($commande->total() > 100 && $client->ancienneteEnAnnees() >= 2) {
    $remise = $commande->total() * 0.07;
}

// À préférer
const SEUIL_REMISE_FIDELITE = 100;
const ANCIENNETE_MINIMALE_EN_ANNEES = 2;
const TAUX_REMISE_FIDELITE = 0.07;

if ($commande->total() > SEUIL_REMISE_FIDELITE
    && $client->ancienneteEnAnnees() >= ANCIENNETE_MINIMALE_EN_ANNEES) {
    $remise = $commande->total() * TAUX_REMISE_FIDELITE;
}
```

Le second extrait est aussi rapide et révèle la **règle métier**, pas seulement le calcul.

[🔝 Retour en haut de page](#table-des-matières)

## Commentaires

Un bon commentaire explique *pourquoi* le code fait ce qu'il fait, jamais *ce qu'il fait* (le code le dit déjà). Avant d'écrire un commentaire, demandez-vous si un meilleur nom de fonction ou de variable ne le rendrait pas inutile.

### À éviter

```php
// Incrémenter i
$i++;

// Vérifier si l'utilisateur est admin
if ($user->role === 'admin') { ... }
```

### À préférer

```php
$i++;

if ($user->estAdministrateur()) { ... }

// La règlementation RGPD impose un délai de purge de 30 jours minimum.
const DELAI_PURGE_JOURS = 30;
```

| Type de commentaire | Utile ? | Raison |
|---------------------|---------|--------|
| Justification d'un choix non évident | Oui | Le code montre le *quoi*, pas le *pourquoi*. |
| Référence à une norme, un ticket, une RFC | Oui | Aide le futur lecteur à retrouver le contexte. |
| Avertissement (effet de bord, perf, ordre d'appel) | Oui | Évite des bugs subtils. |
| Paraphrase du code | Non | Bruit ; sera désynchronisé tôt ou tard. |
| TODO sans owner ni date | Non | Reste *ad vitam æternam* ; préférer un ticket. |

### Quand commenter abondamment

Les API publiques (PHPDoc, JSDoc, docstrings) gagnent à être documentées : leurs utilisateurs ne lisent pas leur implémentation.

### Heuristique générale

> *« Un commentaire est l'aveu qu'on n'a pas su s'exprimer en code. »* — Robert C. Martin

Avant d'écrire un commentaire, essayez : (1) renommer une variable, (2) extraire une fonction au nom signifiant, (3) introduire une constante. Si le besoin de commenter persiste, alors le commentaire est probablement justifié.

[🔝 Retour en haut de page](#table-des-matières)

## Mise en forme du code (formatage)

Le formatage n'est pas cosmétique : il guide le regard du lecteur. Un fichier bien mis en forme se parcourt comme un texte, du plus général (en haut) au plus détaillé (en bas).

### Densité verticale

| Règle | Idée |
|-------|------|
| Lignes vides comme paragraphes | Une ligne vide sépare deux idées ; à l'intérieur d'un bloc, restez denses. |
| Concepts liés rapprochés | Une variable est déclarée près de son premier usage, pas en haut de fonction « comme en C ». |
| Du général au détail | Une classe expose d'abord ses méthodes publiques (le *quoi*), puis ses méthodes privées (le *comment*). |
| Fonctions appelées **après** l'appelant | On lit le fichier comme un journal : titre, paragraphes, sous-paragraphes. |

### Densité horizontale

| Règle | Idée |
|-------|------|
| Lignes courtes (~120 caractères) | Au-delà, l'œil saute des lignes en lisant. |
| Espaces autour des opérateurs | `a + b` se lit mieux que `a+b`. |
| Indentation cohérente | 4 espaces en PHP (PSR-12). Un éditeur correctement configuré rend la règle invisible. |

### Exemple

```php
final class CalculateurRemise
{
    private const SEUIL_FIDELITE = 100;
    private const TAUX_FIDELITE = 0.07;

    public function calculer(Commande $commande, Client $client): Montant
    {
        if (! $this->estEligible($commande, $client)) {
            return Montant::zero();
        }

        return $commande->total()->multiplier(self::TAUX_FIDELITE);
    }

    private function estEligible(Commande $commande, Client $client): bool
    {
        return $commande->total()->superieurA(self::SEUIL_FIDELITE)
            && $client->estFidele();
    }
}
```

L'ordre raconte : « voici la règle (constantes), voici l'API (`calculer`), voici le détail (`estEligible`) ».

### Règles d'équipe avant règles personnelles

Le pire formatage cohérent vaut mieux que le meilleur formatage inconstant. Outils recommandés en PHP : [PHP-CS-Fixer](https://github.com/PHP-CS-Fixer/PHP-CS-Fixer), [PHP_CodeSniffer](https://github.com/PHPCSStandards/PHP_CodeSniffer). Branchez-les en pré-commit pour rendre le débat impossible.

[🔝 Retour en haut de page](#table-des-matières)

## Fonctions courtes et spécifiques

Une fonction doit faire **une seule chose** et la faire à un seul niveau d'abstraction. Robert C. Martin propose comme heuristique : « si vous pouvez extraire une autre fonction avec un nom signifiant, faites-le ».

### À éviter

```php
function traiterCommande(array $commande): void {
    // validation
    if (empty($commande['lignes'])) { throw new RuntimeException('vide'); }
    foreach ($commande['lignes'] as $l) {
        if ($l['quantite'] <= 0) { throw new RuntimeException('quantité'); }
    }
    // calcul
    $total = 0;
    foreach ($commande['lignes'] as $l) {
        $total += $l['prix'] * $l['quantite'];
    }
    // persistance
    $pdo = new PDO(...);
    $pdo->prepare('INSERT ...')->execute([...]);
    // notification
    mail($commande['email'], 'Confirmation', "Total : $total");
}
```

### À préférer

```php
function traiterCommande(Commande $commande): void {
    valider($commande);
    $total = calculerTotal($commande);
    enregistrer($commande, $total);
    notifier($commande, $total);
}
```

Chaque sous-fonction est testable isolément et son nom documente l'étape.

### Niveaux d'abstraction descendants

Une fonction mélange souvent du **quoi** (politique métier) et du **comment** (mécanique technique). Robert C. Martin appelle cela la **règle des niveaux descendants** : à chaque niveau, on doit pouvoir lire la fonction comme une suite de phrases d'**un même niveau d'abstraction**.

```php
// À éviter : trois niveaux mélangés en six lignes
public function expedier(Commande $commande): void
{
    $client = $this->pdo->query('SELECT * FROM clients WHERE id = ' . $commande->clientId())->fetch();
    $this->mailer->envoyer($client['email'], 'Expédiée', 'Votre commande #' . $commande->id() . ' part demain.');
    $commande->marquerCommeExpediee();
    $this->pdo->prepare('UPDATE commandes SET statut = ? WHERE id = ?')->execute(['EXPEDIEE', $commande->id()]);
}
```

```php
// À préférer : chaque ligne raconte une étape, les détails descendent d'un cran
public function expedier(Commande $commande): void
{
    $client = $this->clients->trouverParId($commande->clientId());
    $this->notifierExpedition($client, $commande);
    $this->commandes->marquerExpediee($commande);
}
```

### Préférer peu d'arguments

Plus une fonction prend d'arguments, plus elle est difficile à comprendre, à appeler correctement, à tester. Robert C. Martin propose une échelle :

| Nombre d'arguments | Nom | Recommandation |
|--------------------|-----|----------------|
| 0 | *niladique* | Idéal. |
| 1 | *monadique* | Excellent. |
| 2 | *dyadique* | Acceptable, surtout si l'ordre est naturel (`new Point($x, $y)`). |
| 3 | *triadique* | À justifier ; envisager un objet paramètre. |
| 4 et + | *polyadique* | Presque toujours une *odeur* (« *long parameter list* »). |

#### Introduire un objet paramètre

```php
// Long parameter list : on doit retenir l'ordre, le type, le sens des null
public function creerCommande(
    int $clientId,
    string $deviseIso,
    bool $expressShipping,
    ?string $codePromo,
    ?DateTimeImmutable $dateLivraisonSouhaitee,
): Commande { /* ... */ }

// Mieux : un objet paramètre nommé, immuable, validable
public function creerCommande(NouvelleCommande $payload): Commande { /* ... */ }
```

### Booléens en argument : un drapeau rouge

Un argument booléen indique souvent que la fonction fait **deux choses**. Préférez deux fonctions au nom explicite.

```php
// À éviter
$utilisateur->save(true); // que fait `true` ?

// À préférer
$utilisateur->saveAndNotify();
$utilisateur->saveSilently();
```

### Command-Query Separation (CQS)

Une **commande** modifie l'état et ne renvoie rien d'utile (`void`). Une **requête** lit l'état et n'a aucun effet de bord. Bertrand Meyer a formalisé la règle dans *Object-Oriented Software Construction*.

```php
// À éviter : modifie ET renvoie. L'appelant ne peut pas relire sans risquer de modifier.
public function compteurEtIncremente(): int
{
    return ++$this->compteur;
}

// À préférer
public function incrementer(): void { $this->compteur++; }
public function compteur(): int     { return $this->compteur; }
```

Exception traditionnelle : `pop()` sur une pile. Les exceptions doivent rester rares et nommées explicitement.

### Quand ne pas découper

Découper une fonction de cinq lignes triviales en cinq fonctions d'une ligne nuit à la lisibilité. La règle est *un niveau d'abstraction*, pas *un nombre maximal de lignes*.

[🔝 Retour en haut de page](#table-des-matières)

## Respect des conventions de codage

Les conventions sont des choix arbitraires (placement des accolades, casse des noms…) qui n'ont d'intérêt que par leur **uniformité**. Pour PHP, le standard de référence est [PSR-12](https://www.php-fig.org/psr/psr-12/).

### À éviter

```php
class user_service{
function GetUser($Id){
if($Id==null)return null;
    return $this->repo->find( $Id );
}}
```

### À préférer

```php
class UserService
{
    public function getUser(int $id): ?User
    {
        if ($id === 0) {
            return null;
        }

        return $this->repo->find($id);
    }
}
```

### Quand assouplir

Un projet hérité avec sa propre convention doit conserver sa cohérence interne ; mélanger PSR-12 et l'ancien style introduit plus de friction qu'il n'en résout. Mieux vaut migrer en bloc à l'aide d'un outil ([PHP-CS-Fixer](https://github.com/PHP-CS-Fixer/PHP-CS-Fixer)).

[🔝 Retour en haut de page](#table-des-matières)

## Duplication

La règle DRY (*Don't Repeat Yourself*, Hunt & Thomas, 1999) impose qu'une connaissance n'ait qu'une représentation autoritaire dans le système. Dupliquer du code, c'est dupliquer la maintenance et risquer la dérive entre les copies.

### À éviter

```php
// Dans le contrôleur d'inscription
if (!isset($data['email']) || $data['email'] === '' || !filter_var($data['email'], FILTER_VALIDATE_EMAIL)) {
    return ['erreur' => 'email invalide'];
}

// Dans le contrôleur de mise à jour de profil
if (!isset($data['email']) || $data['email'] === '' || !filter_var($data['email'], FILTER_VALIDATE_EMAIL)) {
    return ['erreur' => 'email invalide'];
}
```

### À préférer

```php
function emailValide(?string $email): bool
{
    return $email !== null
        && $email !== ''
        && filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
}
```

### Attention à la fausse duplication

Deux blocs qui se ressemblent aujourd'hui mais évoluent pour des raisons différentes ne sont pas une duplication — les fusionner crée un couplage accidentel. Avant d'extraire, vérifiez que les deux occurrences décrivent bien la **même règle métier**.

[🔝 Retour en haut de page](#table-des-matières)

## Objets vs structures de données

Robert C. Martin oppose deux styles de modélisation que beaucoup confondent :

| | Structure de données | Objet |
|--|----------------------|-------|
| Expose | ses **données** ; l'extérieur écrit la logique. | son **comportement** ; les données sont cachées. |
| Idéal pour | DTO, transport, persistance, sérialisation. | Domaine métier, règles, invariants. |
| Test du *bon style* | Ajouter un nouveau **type** est facile (une nouvelle structure suffit). | Ajouter une nouvelle **opération** est facile (une nouvelle méthode sur l'interface suffit). |

```php
// Structure de données : transparente, sans logique
final class CoordonneesDto
{
    public function __construct(
        public readonly float $latitude,
        public readonly float $longitude,
    ) {}
}

// Objet : logique encapsulée, données cachées
final class Position
{
    public function __construct(private float $latitude, private float $longitude) {}

    public function distanceA(self $autre): float { /* ... */ }
    public function estDansRayon(self $centre, float $rayonKm): bool { /* ... */ }
}
```

Le piège classique : un *Active Record* qui est mi-DTO, mi-objet métier — il cumule les inconvénients des deux. Mieux vaut séparer la couche persistance (DTO/Repository) du domaine (objet riche).

[🔝 Retour en haut de page](#table-des-matières)

## Loi de Déméter et Tell, Don't Ask

### Loi de Déméter (LoD)

Énoncée à la Northeastern University en 1987, la loi tient en une phrase : **un objet ne devrait parler qu'à ses amis directs**, pas aux amis de ses amis.

Concrètement, à l'intérieur d'une méthode de la classe `A`, on ne peut appeler que :

1. les méthodes de `A` lui-même,
2. les méthodes des objets passés en argument,
3. les méthodes des attributs directs de `A`,
4. les méthodes des objets que `A` crée elle-même.

```php
// À éviter : "train wreck", chaîne d'appels qui révèle la structure interne
$prix = $commande->getClient()->getAdresse()->getPays()->getTauxTva();

// À préférer : on demande directement ce dont on a besoin
$prix = $commande->tauxTvaApplicable();
```

Le second extrait permet de changer la structure interne (par exemple : la TVA dépend désormais de la catégorie du produit) sans casser l'appelant.

### Tell, Don't Ask

Plutôt que d'**interroger** un objet pour prendre une décision à sa place, on lui **dit** quoi faire.

```php
// Ask : on extrait l'état, on décide à l'extérieur
if ($compte->getSolde() >= $montant) {
    $compte->setSolde($compte->getSolde() - $montant);
} else {
    throw new SoldeInsuffisant();
}

// Tell : la règle vit là où vivent les données
$compte->debiter($montant); // lance SoldeInsuffisant si nécessaire
```

Le second style respecte l'**encapsulation** : l'invariant « solde non négatif » ne peut plus être violé par un appelant distrait.

[🔝 Retour en haut de page](#table-des-matières)

## Fonctions pures et effets de bord

Une **fonction pure** :

1. **renvoie toujours le même résultat** pour les mêmes arguments ;
2. **n'a aucun effet de bord** : pas d'écriture en base, en fichier, en réseau, dans une variable globale, dans un attribut.

Les fonctions pures sont triviales à tester : pas de mock, pas de fixture, pas de nettoyage.

```php
// Fonction pure : entrées => sortie, point.
function appliquerRemise(Montant $total, float $taux): Montant
{
    return $total->multiplier(1 - $taux);
}

// Impure : journalise, lit l'horloge, écrit en BDD
function appliquerRemiseEtJournaliser(Commande $c, float $taux): Montant
{
    $remise = $c->total()->multiplier(1 - $taux);
    $this->logger->info('Remise appliquée', ['date' => new DateTime()]);
    $this->pdo->prepare('UPDATE ...')->execute([...]);
    return $remise;
}
```

### Stratégie : *functional core, imperative shell*

L'idéal pratique est de :

* **concentrer la logique métier dans des fonctions pures** (testables, raisonnables) ;
* **reléguer les effets de bord à une fine couche extérieure** (contrôleur, *use case*) qui orchestre.

```php
// Cœur pur : décide
final class CalculateurFacture
{
    public function calculer(Panier $panier, Client $client): Facture { /* aucun I/O */ }
}

// Coquille imperative : agit
final class ValiderCommandeUseCase
{
    public function executer(NouvelleCommande $payload): void
    {
        $panier   = $this->paniers->charger($payload->panierId);   // I/O
        $client   = $this->clients->charger($payload->clientId);   // I/O
        $facture  = $this->calculateur->calculer($panier, $client); // pur
        $this->factures->enregistrer($facture);                    // I/O
        $this->mailer->envoyer($client->email, $facture);          // I/O
    }
}
```

[🔝 Retour en haut de page](#table-des-matières)

## Utilisation de tests unitaires

Un test unitaire vérifie le comportement d'une unité de code (typiquement une classe ou une fonction) isolée de ses dépendances. Il sert de filet de sécurité pour le refactoring et de spécification exécutable.

### Bonnes pratiques

| Pratique | Description |
|----------|-------------|
| Un test = un comportement | Le nom du test décrit ce qui est vérifié (`it_renvoie_null_quand_id_inconnu`). |
| Pattern AAA | *Arrange* (préparer), *Act* (exécuter), *Assert* (vérifier). |
| Indépendance | Les tests s'exécutent dans n'importe quel ordre, sans état partagé. |
| Rapidité | Un test unitaire dure quelques millisecondes ; les tests lents découragent leur exécution. |
| Test d'erreurs | Vérifier les chemins d'échec autant que les chemins nominaux. |

### Exemple

```php
use PHPUnit\Framework\TestCase;

final class CalculatriceTest extends TestCase
{
    public function test_addition_de_deux_entiers(): void
    {
        $calc = new Calculatrice();

        $resultat = $calc->ajouter(2, 3);

        $this->assertSame(5, $resultat);
    }
}
```

### Frameworks usuels

[PHPUnit](https://phpunit.de/) en PHP, [JUnit](https://junit.org/) en Java, [pytest](https://pytest.org/) en Python, [Jest](https://jestjs.io/) en JavaScript, [xUnit](https://xunit.net/) en .NET.

### Quand ne pas écrire de tests unitaires

Le code purement déclaratif (configuration, mapping ORM) gagne peu à être unitairement testé. À l'inverse, du code algorithmique simple n'a pas toujours besoin d'une couverture exhaustive ; les tests d'intégration peuvent suffire.

[🔝 Retour en haut de page](#table-des-matières)

## Tests : principes F.I.R.S.T. et TDD

### F.I.R.S.T.

Acronyme proposé par Robert C. Martin pour caractériser un bon test unitaire :

| Lettre | Mot | Signification | Conséquence pratique |
|--------|-----|---------------|----------------------|
| **F** | **Fast** (rapide) | Le test s'exécute en quelques millisecondes. | Pas de réseau, pas de base réelle, pas de `sleep()`. Doubles de tests pour les I/O. |
| **I** | **Independent** (indépendant) | Aucun test ne dépend du résultat d'un autre, ni de leur ordre. | Pas d'état partagé, pas de fixture globale mutable. |
| **R** | **Repeatable** (reproductible) | Le test donne le même résultat à chaque exécution, sur n'importe quelle machine. | Geler l'horloge (*clock injection*), figer les générateurs aléatoires. |
| **S** | **Self-validating** (auto-validant) | Le test renvoie *vert* ou *rouge* sans interprétation humaine. | Pas de `var_dump`, pas d'inspection visuelle. Des assertions, point. |
| **T** | **Timely** (à temps) | Le test est écrit **juste avant** ou **avec** le code de production, pas après coup. | Sinon le code se fige en formes difficiles à tester (statiques, singletons, longues fonctions). |

#### Exemple commenté

```php
final class CalculateurRemiseTest extends TestCase
{
    public function test_pas_de_remise_sous_le_seuil(): void
    {
        // Arrange : données figées, pas d'I/O => Fast, Repeatable
        $calc = new CalculateurRemise(seuil: 100, taux: 0.10);

        // Act
        $remise = $calc->pour(montant: 50);

        // Assert : auto-validant, sans var_dump
        $this->assertSame(0.0, $remise);
    }

    public function test_remise_de_dix_pourcent_au_dessus_du_seuil(): void
    {
        $calc = new CalculateurRemise(seuil: 100, taux: 0.10);

        $remise = $calc->pour(montant: 200);

        $this->assertSame(20.0, $remise);
    }
}
```

Ces deux tests sont **indépendants** (chacun crée son propre objet), **rapides** (aucune I/O), **reproductibles** (aucune horloge ni aléa), **auto-validants** (assertions strictes).

### TDD : Red, Green, Refactor

Le *Test-Driven Development* (Kent Beck, 2003) est une discipline en boucle courte :

1. **Red** — Écrire un test qui échoue parce que le comportement n'existe pas encore. Lancer le test : il doit être rouge.
2. **Green** — Écrire **le code minimum** qui fait passer le test au vert. Pas plus. Même si c'est moche.
3. **Refactor** — Le test étant vert, améliorer la structure (extraire, renommer, dédupliquer) **sans changer le comportement**. Relancer les tests : ils restent verts.
4. Recommencer pour le comportement suivant.

Bénéfices : on n'écrit que du code couvert, la conception émerge progressivement, et chaque refactoring est protégé par un filet.

#### Mini-cycle TDD en PHP

```php
// 1. RED — j'écris le test avant que la classe n'existe.
public function test_total_panier_vide_vaut_zero(): void
{
    $panier = new Panier();
    $this->assertSame(0, $panier->total());
}
// (Le test échoue : la classe Panier n'existe pas encore.)

// 2. GREEN — je fais passer, même grossièrement.
final class Panier
{
    public function total(): int { return 0; }
}

// 3. RED suivant
public function test_total_avec_un_article(): void
{
    $panier = new Panier();
    $panier->ajouter(new Article(prixCentimes: 250));

    $this->assertSame(250, $panier->total());
}

// 4. GREEN minimal
final class Panier
{
    /** @var Article[] */
    private array $articles = [];

    public function ajouter(Article $a): void { $this->articles[] = $a; }
    public function total(): int
    {
        $somme = 0;
        foreach ($this->articles as $a) { $somme += $a->prixCentimes(); }
        return $somme;
    }
}

// 5. REFACTOR sans changer le comportement
public function total(): int
{
    return array_sum(array_map(fn(Article $a) => $a->prixCentimes(), $this->articles));
}
```

### Test smells (mauvaises odeurs côté tests)

| Smell | Symptôme | Remède |
|-------|----------|--------|
| Test fragile | Un changement interne casse le test sans changer le comportement public. | Tester via l'API publique, pas l'implémentation. |
| Test obscur | Beaucoup de mise en place avant la moindre assertion. | Extraire des *builders* ou *factories* de tests. |
| Tests interdépendants | Un test ne passe que si un autre est exécuté avant. | Repartir d'un état neuf à chaque test. |
| Assertion molle | Le test passe pour un trop large éventail de comportements. | Assertions précises (`assertSame` plutôt qu'`assertNotNull`). |
| Mocking excessif | Le test est presque entièrement constitué de doubles. | Souvent le signe d'un couplage trop fort à corriger côté production. |

[🔝 Retour en haut de page](#table-des-matières)

## Documentation de code

La documentation utile est celle qui survit aux refactorings : elle décrit *l'intention*, pas l'implémentation. Trois niveaux complémentaires :

| Niveau | Public | Exemples |
|--------|--------|----------|
| README | Nouveaux contributeurs | But du projet, prérequis, démarrage. |
| Architecture (ADR) | Mainteneurs | Décisions structurantes et leurs justifications. |
| API (PHPDoc, OpenAPI) | Consommateurs du code | Signatures, contrats, codes d'erreur. |

### À éviter

```php
/**
 * Cette méthode prend un id et retourne un utilisateur.
 *
 * @param int $id l'id
 * @return User l'utilisateur
 */
public function find(int $id): User { ... }
```

Le commentaire paraphrase la signature sans rien ajouter.

### À préférer

```php
/**
 * Récupère un utilisateur par son identifiant interne.
 *
 * @throws UtilisateurIntrouvable si aucun utilisateur ne porte cet identifiant.
 */
public function find(int $id): User { ... }
```

[🔝 Retour en haut de page](#table-des-matières)

## Gestion des erreurs et des exceptions

Une erreur est un événement exceptionnel qui empêche une opération d'aboutir. Le code doit la signaler clairement, l'attraper là où on peut décider quoi en faire, et fournir au journal de quoi diagnostiquer.

### À éviter

```php
$resultat = mysqli_connect(...);
if (!$resultat) {
    die('Erreur : connexion impossible');
}
```

`die()` et `exit()` interrompent l'exécution sans laisser à l'appelant la moindre chance de réagir, et ne produisent aucune trace exploitable.

### À préférer

```php
try {
    $connexion = new PDO($dsn, $user, $password);
} catch (PDOException $e) {
    $logger->error('Connexion BDD impossible', ['exception' => $e]);
    throw new ServiceIndisponible('Base de données injoignable', previous: $e);
}
```

| Pratique | Pourquoi |
|----------|----------|
| Exceptions plutôt que codes de retour | L'oubli d'un code d'erreur est silencieux ; une exception non gérée explose. |
| Types d'exceptions métier | `UtilisateurIntrouvable` est plus clair que `Exception('not found')`. |
| Capturer le plus tard possible | Là où l'on peut vraiment décider : afficher un message, retenter, basculer. |
| Conserver la cause (`previous:`) | Préserve la chaîne complète pour le débogage. |
| Journaliser avec contexte | Identifiant utilisateur, identifiant de requête, et non l'objet brut. |

### Quand ne pas lancer d'exception

Une absence de résultat *attendue* (recherche qui ne trouve rien) n'est pas une erreur ; renvoyer `null` ou un type optionnel est plus honnête.

[🔝 Retour en haut de page](#table-des-matières)

## Structure du code claire et organisée

Un projet bien structuré laisse deviner où ajouter une fonctionnalité avant même de l'avoir lue. Cela suppose une organisation **par domaine** plutôt que par couche technique.

### À éviter

```
src/
├── controllers/
├── models/
├── services/
└── helpers/
```

Cette organisation par couche force à parcourir tout le projet pour comprendre une fonctionnalité.

### À préférer

```
src/
├── Facturation/
│   ├── Controleur/
│   ├── Domaine/
│   └── Persistance/
├── Catalogue/
│   ├── Controleur/
│   ├── Domaine/
│   └── Persistance/
└── Utilisateur/
    ├── Controleur/
    ├── Domaine/
    └── Persistance/
```

Chaque module reste autonome : on peut le lire sans connaître les autres, et le déplacer ou l'extraire en service à part sans démêler des dépendances cachées.

[🔝 Retour en haut de page](#table-des-matières)

## Gestion des dépendances

Une dépendance externe (bibliothèque, framework) est du code que vous ne maintenez pas mais que vous embarquez. Le coût se paie à la mise à jour, à la sécurité et à la compatibilité.

### Bonnes pratiques

| Pratique | Pourquoi |
|----------|----------|
| Utiliser un gestionnaire ([Composer](https://getcomposer.org/)) | Versions reproductibles, résolution transitive, autoload. |
| Verrouiller les versions (`composer.lock`) | Garantit que CI, devs et prod installent le même graphe. |
| Suivre [SemVer](https://semver.org/) | `^1.2.3` accepte les correctifs et fonctionnalités, pas les ruptures. |
| Auditer régulièrement (`composer audit`) | Détecte les CVE connues. |
| Limiter les dépendances optionnelles | Chaque dépendance ajoute une surface d'attaque et un risque de conflit. |

### À éviter

```json
{
  "require": {
    "vendor/lib": "*"
  }
}
```

`*` accepte la prochaine version majeure et son lot de ruptures.

### À préférer

```json
{
  "require": {
    "vendor/lib": "^2.3"
  }
}
```

[🔝 Retour en haut de page](#table-des-matières)

## Gestion de la complexité du code

La complexité cyclomatique mesure le nombre de chemins d'exécution distincts dans une fonction. Au-delà de 10, la fonction devient difficile à tester exhaustivement et à comprendre.

### À éviter — imbrication excessive

```php
function inscrire(array $data) {
    if (!empty($data['email'])) {
        if (filter_var($data['email'], FILTER_VALIDATE_EMAIL)) {
            if (!empty($data['mdp'])) {
                if (strlen($data['mdp']) >= 8) {
                    // ... la vraie logique, perdue à 4 niveaux d'indentation
                }
            }
        }
    }
}
```

### À préférer — clauses de garde

```php
function inscrire(array $data) {
    if (empty($data['email']) || !filter_var($data['email'], FILTER_VALIDATE_EMAIL)) {
        throw new DonneesInvalides('email');
    }
    if (empty($data['mdp']) || strlen($data['mdp']) < 8) {
        throw new DonneesInvalides('mdp');
    }

    // logique réelle au premier niveau d'indentation
}
```

| Symptôme | Remède |
|----------|--------|
| `if`/`else` profondément imbriqués | Clauses de garde (`return`/`throw` tôt). |
| Longue chaîne `else if` | Table de dispatch, polymorphisme, ou `match`. |
| Conditions booléennes longues | Extraire dans une fonction au nom signifiant : `estEligible(...)`. |

[🔝 Retour en haut de page](#table-des-matières)

## Les fonctions doivent faire une seule chose

C'est le principe de responsabilité unique appliqué au niveau d'une fonction. Si vous pouvez décrire le rôle d'une fonction sans utiliser « et » ou « puis », elle est probablement bien découpée.

### À éviter

```php
function getUtilisateur(int $id): ?array {
    $cnx = mysqli_connect('localhost', 'user', 'pwd', 'db');
    $sql = "SELECT * FROM users WHERE id = $id";
    $res = mysqli_query($cnx, $sql);
    $row = mysqli_fetch_assoc($res);
    mysqli_close($cnx);
    return $row;
}
```

Cette fonction connecte, exécute, hydrate et nettoie ; quatre raisons de changer (driver, schéma, format de retour, gestion de connexion).

### À préférer

```php
final class UtilisateurRepository
{
    public function __construct(private PDO $pdo) {}

    public function trouverParId(int $id): ?Utilisateur
    {
        $stmt = $this->pdo->prepare('SELECT * FROM users WHERE id = :id');
        $stmt->execute(['id' => $id]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);

        return $row ? Utilisateur::depuisLigne($row) : null;
    }
}
```

La connexion est injectée (responsabilité d'un autre), la requête est paramétrée (sécurité), l'hydratation est déléguée à `Utilisateur::depuisLigne` (responsabilité du domaine).

### Quand assouplir

Une fonction utilitaire d'une dizaine de lignes qui orchestre deux étapes très liées (« lire un fichier puis le parser ») peut rester d'un seul tenant si l'extraction n'apporte aucune réutilisabilité.

[🔝 Retour en haut de page](#table-des-matières)

## Code smells (mauvaises odeurs de code)

Un *code smell* est un signal visuel qu'un morceau de code mérite probablement un refactoring. Ce catalogue, popularisé par Martin Fowler et Kent Beck dans *Refactoring*, est un vocabulaire commun pour nommer ce que l'on sent sans toujours savoir formuler.

| Smell (anglais) | Nom français | Symptôme | Refactoring typique |
|------------------|--------------|----------|---------------------|
| **Long method** | Méthode trop longue | La fonction dépasse l'écran ; on perd le fil. | *Extract Method* (extraire des sous-fonctions au nom signifiant). |
| **Large class** | Classe trop grosse | Trop d'attributs, trop de méthodes, plusieurs responsabilités. | *Extract Class* / *Extract Subclass* en respectant la SRP. |
| **Long parameter list** | Liste de paramètres trop longue | Quatre arguments et plus, certains corrélés. | *Introduce Parameter Object*. |
| **Primitive obsession** | Obsession des primitifs | Tout est `string`, `int`, `array` — y compris les concepts métier (email, montant, IBAN). | *Introduce Value Object* (`Email`, `Iban`, `Montant`). |
| **Feature envy** | Envie de fonctionnalité | Une méthode de `A` manipule plus les attributs de `B` que les siens. | *Move Method* vers `B`. |
| **Data clump** | Grumeau de données | Un même groupe de variables apparaît dans plusieurs signatures (`$rue`, `$cp`, `$ville`). | Regrouper dans une classe `Adresse`. |
| **Shotgun surgery** | Modification éparpillée | Un seul changement métier oblige à toucher dix fichiers. | Regrouper la connaissance (*Move Method*, *Inline Class*, module dédié). |
| **Divergent change** | Changement divergent | Le contraire : une seule classe est modifiée pour des raisons indépendantes. | *Extract Class* selon les axes de changement (SRP). |
| **Switch / chain of if** | Cascade de `switch` sur le type | Le même `switch` apparaît à plusieurs endroits. | *Replace Conditional with Polymorphism*. |
| **Speculative generality** | Généralité spéculative | Abstractions ajoutées « au cas où », jamais utilisées. | *Inline Class* / *Inline Method* (YAGNI). |
| **Comments** | Commentaires palliatifs | Les commentaires expliquent ce qu'un meilleur nom dirait. | *Rename* / *Extract Method*. |
| **Dead code** | Code mort | Branches jamais exécutées, méthodes jamais appelées. | Suppression. Git garde l'historique. |
| **Magic number** | Nombre magique | Littéral inscrit dans le code sans nom. | *Replace Magic Number with Symbolic Constant*. |
| **God object** | Objet-Dieu | Une classe « sait tout, fait tout ». | Découper en collaborateurs cohésifs. |
| **Train wreck** | Train d'appels | `$a->b()->c()->d()->e()`. | Demander à l'objet ce qu'on veut (Tell, Don't Ask), respecter la loi de Déméter. |

### Exemple : *Primitive Obsession* en PHP

```php
// À éviter : un email est une string... jusqu'à la prochaine validation oubliée.
function inscrire(string $email, string $motDePasse): void { /* ... */ }

// À préférer : le type rend l'invariant impossible à oublier.
final class Email
{
    public function __construct(private string $valeur)
    {
        if (! filter_var($valeur, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException("Email invalide : $valeur");
        }
    }

    public function __toString(): string { return $this->valeur; }
}

function inscrire(Email $email, MotDePasse $motDePasse): void { /* ... */ }
```

Toute fonction qui reçoit un `Email` peut **présupposer** sa validité : la connaissance n'est plus dispersée.

[🔝 Retour en haut de page](#table-des-matières)

## Catalogue de refactorings essentiels

Un refactoring est une transformation **qui ne change pas le comportement**. Tests verts avant, tests verts après. Voici les transformations à connaître par cœur ; le catalogue complet est dans le livre de Martin Fowler.

### Extract Method (extraire une méthode)

```php
// Avant
public function imprimerFacture(Facture $f): void
{
    echo "Client : {$f->client()->nom()}\n";
    echo "Date   : {$f->date()->format('Y-m-d')}\n";
    $total = 0;
    foreach ($f->lignes() as $l) {
        $total += $l->prix() * $l->quantite();
        echo "- {$l->libelle()} : {$l->prix()} x {$l->quantite()}\n";
    }
    echo "Total : $total\n";
}

// Après
public function imprimerFacture(Facture $f): void
{
    $this->imprimerEntete($f);
    $total = $this->imprimerLignes($f);
    $this->imprimerTotal($total);
}
```

### Rename (renommer)

Le refactoring le plus rentable et le plus sous-estimé. Un IDE moderne le rend mécanique. Renommer `process()` en `validerCommande()` économise dix commentaires.

### Replace Magic Number with Symbolic Constant

Voir le chapitre [Nommage](#nommage-des-variables). On nomme `0.20` en `TAUX_TVA`, `30` en `DELAI_PURGE_JOURS`.

### Replace Conditional with Polymorphism

Quand un même `switch` apparaît à plusieurs endroits, on le remplace par un appel polymorphe.

```php
// Avant : switch dispersé dans plusieurs services
function calculerCommission(Employe $e): float
{
    switch ($e->type()) {
        case 'CDI':       return $e->salaire() * 0.05;
        case 'CDD':       return $e->salaire() * 0.03;
        case 'STAGIAIRE': return 0.0;
    }
    throw new LogicException('type inconnu');
}

// Après : la connaissance vit dans chaque sous-classe
abstract class Employe { abstract public function commission(): float; }

final class Cdi       extends Employe { public function commission(): float { return $this->salaire * 0.05; } }
final class Cdd       extends Employe { public function commission(): float { return $this->salaire * 0.03; } }
final class Stagiaire extends Employe { public function commission(): float { return 0.0; } }

function calculerCommission(Employe $e): float { return $e->commission(); }
```

Ajouter un nouveau type d'employé (alternant, freelance) ne touche **plus aucun appelant**.

### Introduce Parameter Object

Voir [Préférer peu d'arguments](#fonctions-courtes-et-spécifiques). Un objet paramètre regroupe ce qui voyage ensemble.

### Replace Conditional with Guard Clauses

Voir [Gestion de la complexité](#gestion-de-la-complexité-du-code). Sortir tôt aplatit l'imbrication.

### Encapsulate Field

```php
// Avant : champ public, invariants impossibles à garantir
final class Compte { public float $solde = 0.0; }

// Après : champ privé, méthodes d'accès qui imposent la règle
final class Compte
{
    private float $solde = 0.0;

    public function solde(): float { return $this->solde; }

    public function crediter(float $montant): void
    {
        if ($montant <= 0) { throw new InvalidArgumentException('montant > 0'); }
        $this->solde += $montant;
    }

    public function debiter(float $montant): void
    {
        if ($montant <= 0)         { throw new InvalidArgumentException('montant > 0'); }
        if ($montant > $this->solde) { throw new SoldeInsuffisant(); }
        $this->solde -= $montant;
    }
}
```

### Inline Method / Inline Class

Le refactoring inverse de l'extraction. Si une fonction n'apporte plus rien (un seul appelant, nom redondant), on la « remet » en ligne. Refactoriser, c'est aussi savoir **dé-factoriser**.

### Move Method

Si une méthode de `A` accède plus aux attributs de `B` que de `A` (*feature envy*), on la déplace dans `B`. La cohésion remonte, le couplage baisse.

### Discipline du refactoring

1. **Filet de sécurité** : aucun refactoring sans tests automatisés sur la zone touchée.
2. **Petits pas** : commit après chaque transformation atomique. Si un commit casse, le retour en arrière est trivial.
3. **Une chose à la fois** : ne pas refactoriser et ajouter une fonctionnalité dans le même commit. *Refactor first, then add feature.* (Kent Beck.)
4. **Outillage** : laissez l'IDE faire les refactorings mécaniques (renommage, extraction). L'erreur humaine est plus probable que l'erreur de l'outil.

[🔝 Retour en haut de page](#table-des-matières)

## KISS, YAGNI et optimisation prématurée

Trois maximes qui poussent dans la même direction : **ne pas écrire ce qui n'est pas nécessaire**.

### KISS — *Keep It Simple, Stupid*

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

L'extraction sera triviale **le jour où** un troisième cas apparaît, et probablement mieux conçue car informée par trois cas réels au lieu de deux imaginaires.

### YAGNI — *You Aren't Gonna Need It*

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

> *« L'optimisation prématurée est la racine de tous les maux. »* — Donald Knuth

L'optimisation prématurée n'est pas « faire attention aux performances » ; c'est **tordre le code pour un gain non mesuré**. La discipline :

1. **Mesurer d'abord** (profiling : Xdebug, Blackfire, `microtime` ciblé).
2. **Optimiser ensuite** uniquement le goulot identifié.
3. **Garder la version lisible** en commentaire ou en branche, pour comprendre demain.

Le bon moment pour optimiser, c'est quand un test de performance, un SLA ou un utilisateur dit clairement : *ici, c'est trop lent*.

[🔝 Retour en haut de page](#table-des-matières)

## Dette technique

Métaphore de Ward Cunningham (1992) : un raccourci de conception est un emprunt — il rend service maintenant et coûte plus cher demain. La dette n'est pas honteuse : c'est un outil, à condition de la connaître et de la rembourser.

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

### Stratégies de remboursement

| Stratégie | Quand l'employer |
|-----------|------------------|
| **Boy scout rule** | Refactoriser à la marge chaque fichier touché par une feature. Indolore, continu. |
| **Refactoring planifié** | Bloquer 10–20 % du temps de sprint pour des refactorings ciblés. |
| **Strangler Fig pattern** | Remplacer un module hérité morceau par morceau, sans big bang. |
| **Réécriture totale** | Dernier recours. Coûteuse, risquée, longtemps non livrable. À justifier. |

### Tracer la dette

Une dette invisible n'est pas remboursée. Outils utiles :

* Tickets « TECH-DEBT » étiquetés dans le suivi.
* Fichier `ARCHITECTURE.md` ou *Architecture Decision Records* (ADR) qui documentent les compromis.
* Indicateurs : couverture de tests, complexité cyclomatique, durée moyenne d'une PR — tous suivis dans le temps.

[🔝 Retour en haut de page](#table-des-matières)

## Pour aller plus loin

- *Clean Code: A Handbook of Agile Software Craftsmanship* — Robert C. Martin
- *The Clean Coder: A Code of Conduct for Professional Programmers* — Robert C. Martin
- *Refactoring: Improving the Design of Existing Code* (2e éd.) — Martin Fowler
- *Clean Architecture* — Robert C. Martin
- [Refactoring Guru — Code smells](https://refactoring.guru/refactoring/smells)
- [PHP The Right Way](https://phptherightway.com/)
- [PSR coding standards (PHP-FIG)](https://www.php-fig.org/psr/)

## Licence

Distribué sous licence [MIT](LICENSE).

## Auteur

**Tansoftware - Tanguy Chénier** · [LinkedIn](https://www.linkedin.com/in/tanguy-chenier) · [Tan-Software](https://github.com/Tan-Software) · [Compte personnel (derniers outils)](https://github.com/tanguychenier) · [tansoftware.com](https://www.tansoftware.com)
