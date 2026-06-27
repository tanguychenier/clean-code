[← Fondations et vocabulaire](01-fondations-et-vocabulaire.md) · [↑ Sommaire](../README.md#table-des-matières) · [Fonctions et duplication →](03-fonctions-et-duplication.md)

# 2. Nommer, commenter, mettre en forme

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

### Heuristiques tirées des bases de code PHP réelles

> **Que veut dire « heuristique » ?** Une *heuristique* est une règle approximative, tirée de l'expérience, qui marche dans la plupart des cas sans être une loi absolue. « Si une fonction dépasse l'écran, elle fait sans doute trop de choses » est une heuristique : un bon indice à vérifier, pas une vérité mécanique. Pensez au dicton « ciel rouge le soir, beau temps en perspective » : souvent vrai, pas garanti.

> **Que veut dire « PSR » ?** *PSR* signifie *PHP Standard Recommendation* (« recommandation standard PHP »). Ce sont des règles communes, publiées par un groupe de mainteneurs de projets PHP (le PHP-FIG), pour que tout le monde écrive le code de la même façon : nommage, indentation, organisation des fichiers. Numérotées (PSR-1, PSR-4, PSR-12...), elles jouent le rôle d'un code de la route partagé : chacun roule du même côté, donc tout le monde se comprend.

> **Que veut dire « Symfony », « Laravel », « Doctrine » ?** Ce sont des bibliothèques PHP très répandues. *Symfony* et *Laravel* sont des **frameworks** (des squelettes prêts à l'emploi qui fournissent l'ossature d'une application web, pour éviter de tout réécrire). *Doctrine* est un **ORM**, un outil qui fait le pont entre les objets du code et les tables d'une base de données.

Les conventions ci-dessous ne sont écrites nulle part dans les PSR mais s'imposent par usage dans Symfony, Laravel, Doctrine, et la plupart des projets PHP modernes.

| Heuristique | Énoncé | Exemple |
|-------------|--------|---------|
| **Verbe en tête pour les méthodes** | Une méthode *fait* quelque chose : son nom commence par un verbe d'action. | `calculerTotal()`, `valider()`, `enregistrer()`. |
| **Substantif pour les variables et propriétés** | Une variable *est* une chose : son nom est un nom commun. | `$client`, `$lignesCommande`, `$dateLivraison`. |
| **Préfixe booléen** | Un booléen pose une question : `est`, `a`, `peut`, `doit`. | `$estActif`, `aDroitsAdmin`, `peutPublier()`. |
| **Pluriel = collection** | Un nom au pluriel signale une collection (tableau, `Collection`, `iterable`). | `$utilisateurs`, `$factures`. |
| **Suffixe métier pour les classes techniques** | Le rôle architectural se devine au suffixe. | `UserRepository`, `OrderController`, `EmailValidator`, `ShippingFactory`. |
| **`make` / `create` / `new` pour les fabriques** | Distingue les méthodes *fabriques* du reste. | `make()`, `createFromArray()`, `newFromRequest()`. |
| **`from` / `to` pour les conversions** | Met en valeur la transformation entre formats. | `Email::fromString()`, `$invoice->toArray()`. |

> **Que veut dire « notation hongroise » ?** C'est une vieille habitude qui consiste à coller au nom d'une variable une abréviation de son type : `strNom` pour une chaîne (*string*), `iCount` pour un entier (*integer*), `arrLignes` pour un tableau (*array*). Elle vient d'une époque où les éditeurs n'affichaient pas le type. Aujourd'hui l'éditeur le montre tout seul, donc cette notation ne fait qu'ajouter du bruit : si le type change, le nom ment.

**À éviter en PHP moderne :**

* **Notation hongroise** (`strNom`, `iCount`, `arrLignes`). PHP est typé statiquement (depuis 7.0 sur les arguments, depuis 7.4 sur les propriétés) ; les types sont déjà dans la signature et dans l'IDE. Préfixer le type au nom est du bruit.

> **Que veut dire « IDE » et « typé statiquement » ?** Un *IDE* (*Integrated Development Environment*, « environnement de développement intégré ») est l'éditeur évolué dans lequel on écrit le code : il complète les noms, signale les erreurs au survol, et affiche le type des variables. *Typé statiquement* signifie que l'on déclare à l'avance la nature de chaque donnée (texte, entier, etc.) et que des outils vérifient la cohérence avant même l'exécution, ce qui attrape beaucoup d'erreurs tôt. Puisque l'IDE montre déjà le type, le répéter dans le nom est inutile.
* **Préfixe `m_`** pour les attributs (héritage C++/MFC). Les attributs de classe sont déjà signalés par `$this->` à l'usage.
* **Suffixe `_obj`** ou `_var`. Tautologie : `$client_obj` est aussi vide d'information que `$la_chose`.
* **Anglais bricolé** mêlé au français. Choisir une langue par projet, pas par fichier. Le domaine métier en français + termes techniques en anglais (`Repository`, `Controller`) est un compromis fréquent et acceptable.

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

> **Que veut dire « RGPD » ?** *RGPD* signifie *Règlement général sur la protection des données* (en anglais *GDPR*). C'est une loi européenne qui encadre la manière dont les entreprises collectent et conservent les données personnelles (nom, adresse, e-mail). Concrètement, elle impose par exemple de ne pas garder ces données plus longtemps que nécessaire. Quand une telle règle vient d'une loi extérieure au code, un commentaire est utile : le code seul ne peut pas expliquer *pourquoi* le délai vaut 30 jours et pas 7.

| Type de commentaire | Utile ? | Raison |
|---------------------|---------|--------|
| Justification d'un choix non évident | Oui | Le code montre le *quoi*, pas le *pourquoi*. |
| Référence à une norme, un ticket, une RFC | Oui | Aide le futur lecteur à retrouver le contexte. |
| Avertissement (effet de bord, perf, ordre d'appel) | Oui | Évite des bugs subtils. |
| Paraphrase du code | Non | Bruit ; sera désynchronisé tôt ou tard. |
| TODO sans responsable ni date | Non | Reste *ad vitam æternam* ; préférer un ticket. |

> **Que veut dire « RFC » ?** *RFC* signifie *Request For Comments* (« appel à commentaires »). Ce sont des documents techniques officiels qui définissent les standards d'Internet : comment fonctionne l'e-mail, le web, etc. Chacun porte un numéro (par exemple la RFC 7519 décrit le format de jetons *JWT*). Renvoyer vers une RFC dans un commentaire, c'est citer la source de référence, comme on cite un article de loi.

> **Que veut dire « TODO » ?** *TODO* (« à faire », de l'anglais *to do*) est un commentaire qui marque un travail à terminer plus tard, par exemple `// TODO: gérer le cas du paiement refusé`. Le problème : un TODO sans personne responsable ni date reste là pour toujours. Mieux vaut créer un ticket dans l'outil de suivi de l'équipe, où il sera priorisé et n'oublié.

### Quand commenter abondamment

> **Que veut dire « API » ?** *API* signifie *Application Programming Interface* (« interface de programmation d'application »). C'est la liste des fonctions qu'un morceau de code offre aux autres morceaux de code pour s'en servir, sans avoir à connaître son fonctionnement interne. Analogie : le tableau de bord d'une voiture est une API. Vous tournez le volant et appuyez sur la pédale (les fonctions offertes) sans savoir comment le moteur marche à l'intérieur. Une *API publique*, c'est la partie de votre code que d'autres équipes ou projets vont appeler.

> **Que veut dire « PHPDoc », « JSDoc », « docstring » ?** Ce sont trois formes du même outil selon le langage : un bloc de texte structuré placé juste au-dessus d'une fonction pour décrire ce qu'elle fait, ce qu'elle attend et ce qu'elle renvoie. *PHPDoc* en PHP, *JSDoc* en JavaScript, *docstring* en Python. Les éditeurs de code les affichent automatiquement quand on survole la fonction, comme la notice d'un appareil.

Les API publiques (PHPDoc, JSDoc, docstrings) gagnent à être documentées : leurs utilisateurs ne lisent pas leur implémentation (le code interne), seulement la notice.

### Heuristique générale

> *« Un commentaire est l'aveu qu'on n'a pas su s'exprimer en code. »* (Robert C. Martin)

Avant d'écrire un commentaire, essayez : (1) renommer une variable, (2) extraire une fonction au nom signifiant, (3) introduire une constante. Si le besoin de commenter persiste, alors le commentaire est probablement justifié.

### Nuance : la position de Robert C. Martin est contestée

La formule ci-dessus est l'un des conseils les plus repris (et les plus mal appliqués) de *Clean Code*. Il faut la lire dans son contexte : Martin combat les commentaires **redondants** (« incrémente i »), pas les commentaires en général. Des bases de code reconnues pour leur qualité, comme le **noyau Linux**, **FreeBSD**, **SQLite** ou **PostgreSQL**, sont copieusement commentées, et leurs commentaires sont précisément ce qui les rend abordables à un nouvel arrivant.

Un bon commentaire répond à une question que **ni le code ni les tests ne peuvent répondre** :

| Question à laquelle le code ne répond pas | Exemple de commentaire utile |
|-------------------------------------------|------------------------------|
| Pourquoi cette valeur ? | `// 511 : taille max d'un nom UTF-8 en NTFS, source MSDN.` |
| Pourquoi ce contournement ? | `// Workaround : libxml < 2.9 perd l'encodage en cas d'entité.` |
| Pourquoi cet ordre d'opération ? | `// On purge AVANT d'écrire pour éviter une race avec le worker.` |
| Pourquoi ne pas utiliser X ? | `// array_unique() est O(n²) ici ; map associative manuelle.` |
| Quel papier / RFC / ticket ? | `// Cf. RFC 7519 §4.1.4 (exp claim).` |

Ces commentaires **survivent au refactoring** parce qu'ils décrivent une décision, pas une mécanique. Le test à passer : *« si je supprime ce commentaire, est-ce que le prochain mainteneur perdra une demi-journée à reconstruire le contexte ? »* Si oui, gardez-le.

### Commentaires *de section* dans les fonctions longues : un signal, pas une solution

```php
// À éviter : des commentaires-titres qui découpent une fonction trop longue
public function traiter(): void
{
    // -- Validation --
    // ... 15 lignes
    // -- Calcul --
    // ... 20 lignes
    // -- Persistance --
    // ... 10 lignes
}
```

Ici les commentaires soulignent un découpage qui mérite des **fonctions** (`valider()`, `calculer()`, `enregistrer()`). Le commentaire est alors le symptôme, pas le remède.

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

---

[← Fondations et vocabulaire](01-fondations-et-vocabulaire.md) · [↑ Sommaire](../README.md#table-des-matières) · [Fonctions et duplication →](03-fonctions-et-duplication.md)
