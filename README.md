# [Tansoftware](https://www.tansoftware.com) - Clean Code [![fr](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/France.png)](README.md)

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE) [![Lang](https://img.shields.io/badge/Lang-Français-005EB8.svg)](#) [![Topic](https://img.shields.io/badge/Topic-Clean%20Code-brightgreen.svg)](#) [![Examples](https://img.shields.io/badge/Examples-PHP-777BB4.svg)](#)

## Table des matières

* [Introduction](#introduction)
* [Nommage des variables](#nommage-des-variables)
* [Commentaires](#commentaires)
* [Fonctions courtes et spécifiques](#fonctions-courtes-et-spécifiques)
* [Respect des conventions de codage](#respect-des-conventions-de-codage)
* [Duplication](#duplication)
* [Utilisation de tests unitaires](#utilisation-de-tests-unitaires)
* [Documentation de code](#documentation-de-code)
* [Gestion des erreurs et des exceptions](#gestion-des-erreurs-et-des-exceptions)
* [Structure du code claire et organisée](#structure-du-code-claire-et-organisée)
* [Gestion des dépendances](#gestion-des-dépendances)
* [Gestion de la complexité du code](#gestion-de-la-complexité-du-code)
* [Les fonctions doivent faire une seule chose](#les-fonctions-doivent-faire-une-seule-chose)
* [Pour aller plus loin](#pour-aller-plus-loin)

## Introduction

Le terme *Clean Code* (« code propre ») a été popularisé par [Robert C. Martin](https://fr.wikipedia.org/wiki/Robert_C._Martin) dans son livre *Clean Code: A Handbook of Agile Software Craftsmanship* (2008). Il regroupe un ensemble de pratiques qui rendent le code plus lisible, plus facile à modifier et moins coûteux à maintenir.

Un code est dit *propre* lorsqu'un développeur autre que son auteur peut le lire, le comprendre et le faire évoluer sans avoir à mener l'enquête. Ce dépôt présente les pratiques qui contribuent à cet objectif, illustrées par des exemples en PHP.

> *« La règle du boy scout : laissez le campement plus propre que vous ne l'avez trouvé. »* — Robert C. Martin
>
> Appliquée au code, cette règle pousse à améliorer un peu chaque fichier que l'on touche, plutôt que d'attendre une refonte hypothétique.

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

### Quand assouplir

Les noms courts (`i`, `j`, `n`) restent acceptables pour des indices de boucle locale. Les abréviations consacrées dans le domaine (`url`, `id`, `http`) sont également admises ; elles trompent moins qu'une expansion artificielle (`uniformResourceLocator`).

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
