[← Nommer, commenter, mettre en forme](02-nommer-commenter-mettre-en-forme.md) · [↑ Sommaire](../README.md#table-des-matières) · [Conception orientée objet →](04-conception-orientee-objet.md)

# 3. Fonctions et duplication

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

### Les seuils chiffrés : heuristiques, pas dogmes

Robert C. Martin propose des fonctions « de quelques lignes ». Sandi Metz, dans ses conférences chez RailsConf, propose des seuils plus détaillés, qu'elle qualifie elle-même d'**heuristiques de relecture**, pas de lois :

| Élément | Seuil Sandi Metz | Lecture |
|---------|------------------|---------|
| Lignes par méthode | ≤ 5 | Au-delà, se demander s'il y a deux responsabilités. |
| Lignes par classe | ≤ 100 | Au-delà, chercher une *Extract Class*. |
| Arguments par méthode | ≤ 4 | Au-delà, *Introduce Parameter Object*. |
| Variables d'instance utilisées par action de contrôleur | 1 | Garde le contrôleur fin. |

Ces chiffres sont **des indicateurs de relecture**, à appliquer avec discernement. Une méthode de 8 lignes parfaitement claire est meilleure qu'une méthode de 4 lignes qui appelle deux helpers triviaux. Le critère final reste : *un lecteur étranger comprend-il sans effort ?* Les chiffres aident à se poser la question, pas à y répondre.

> **Que veut dire « use case » ?** Un *use case* (« cas d'utilisation »), c'est une action métier complète vue de l'utilisateur : « valider une commande », « inscrire un client », « publier un article ». Dans le code, c'est souvent une classe dont la seule mission est d'orchestrer les étapes de cette action. Pensez à une recette de cuisine : elle énumère les étapes dans l'ordre, mais délègue le détail (couper, cuire) à d'autres.

> **Que veut dire « Value Object » ?** Un *Value Object* (« objet-valeur ») est un petit objet qui représente une valeur du métier en garantissant qu'elle est toujours valide : un `Email`, un `Montant`, un `Iban`. Sa particularité : deux objets-valeurs identiques en contenu sont considérés comme égaux (comme deux billets de 10 euros : peu importe lequel, ils valent pareil). Il refuse d'exister dans un état invalide, ce qui évite de revérifier la même chose partout.

> **Que veut dire « invariant » ?** Un *invariant* est une règle qui doit **toujours** rester vraie pour qu'un objet ait du sens : « un solde de compte ne descend jamais sous zéro », « un e-mail contient un `@` ». L'objet est responsable de protéger ses propres invariants, comme un videur qui ne laisse entrer que les personnes en règle.

> **Que veut dire « enum » ?** Un *enum* (abréviation d'*énumération*) est un type qui ne peut prendre qu'un nombre fixe et connu de valeurs, par exemple un statut de commande qui vaut forcément `EN_ATTENTE`, `PAYEE` ou `EXPEDIEE`, et rien d'autre. Cela empêche les valeurs aberrantes (une commande au statut `bleu`).

Cas où la règle « courte » s'efface :

* **Boucles principales** d'orchestration (un *use case* qui enchaîne 7 étapes claires peut faire 30 lignes lisibles, mieux qu'éclaté en 7 méthodes privées d'une ligne chacune).
* **Constructeurs de validation** d'un Value Object qui vérifie 5 invariants : séquence linéaire, sans abstraction utile à extraire.
* **`match` exhaustif** sur un enum métier : la longueur traduit la richesse du domaine, pas un défaut.

[🔝 Retour en haut de page](#table-des-matières)

## Duplication

> **Que veut dire « DRY » ?** *DRY* signifie *Don't Repeat Yourself* (« ne vous répétez pas »). Une même règle ou connaissance ne doit exister qu'à **un seul endroit** dans le code, qui fait autorité. Sinon, le jour où la règle change, on risque d'oublier l'une des copies, et les versions divergent. Analogie : noter le numéro de téléphone d'un ami dans cinq carnets différents ; quand il change de numéro, certains carnets resteront faux.

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

Deux blocs qui se ressemblent aujourd'hui mais évoluent pour des raisons différentes ne sont pas une duplication ; les fusionner crée un couplage accidentel. Avant d'extraire, vérifiez que les deux occurrences décrivent bien la **même règle métier**.

### « Duplication coûte moins cher que la mauvaise abstraction »

> **Que veut dire « abstraction » et « couplage » ?** Une *abstraction* est une représentation simplifiée qui cache les détails pour ne garder que l'essentiel : une fonction `envoyerEmail()` est une abstraction qui masque les dizaines de lignes techniques en dessous. Une *mauvaise* abstraction regroupe à tort des choses qui n'avaient pas vocation à l'être. Le *couplage* mesure à quel point deux morceaux de code dépendent l'un de l'autre : plus ils sont couplés, plus toucher l'un risque de casser l'autre. On vise un couplage **faible**, comme des appareils branchés sur prises plutôt que soudés ensemble.

Sandi Metz a popularisé cette idée à RailsConf 2014, sous le nom de **AHA principle** (*Avoid Hasty Abstractions*, « évitez les abstractions hâtives »), repris ensuite par Kent C. Dodds. Le raisonnement :

1. Un développeur voit deux blocs qui se ressemblent et les fusionne dans une fonction commune.
2. Plus tard, l'un des deux contextes change. Le développeur ajoute un paramètre, un `if`, un drapeau booléen.
3. Trois mois après, la fonction « factorisée » contient cinq paramètres, deux modes incompatibles, et personne n'ose la toucher.
4. Le coût total dépasse largement celui de la duplication initiale.

**Règle pratique : la règle des trois (*Rule of Three*).** Attendez d'avoir **trois** exemples avant d'extraire une abstraction. Avec seulement deux occurrences, on ne sait pas encore ce qui varie ni ce qui reste constant. Avec trois, l'axe de variation est généralement clair.

```php
// À l'arrivée du 2e cas : on duplique sans culpabilité.
public function exporterFactures(): string { /* ... 12 lignes propres */ }
public function exporterDevis():    string { /* ... 12 lignes propres, qui ressemblent mais... */ }

// Au 3e cas (avoirs), on voit l'invariant : c'est une exportation CSV de documents commerciaux.
// On peut alors extraire un ExportateurDocumentsCommerciaux avec une bonne abstraction.
```

### Duplication accidentelle vs duplication essentielle

| Type | Exemple | Action |
|------|---------|--------|
| **Accidentelle** (même règle, plusieurs copies) | Validation d'email recopiée dans 4 contrôleurs. | Extraire (DRY justifié). |
| **Essentielle** (règles distinctes qui se ressemblent) | Calcul de TVA et calcul de commission, tous deux `montant * taux`. | Garder séparé : ils évolueront indépendamment. |
| **De convergence** (deux règles qui sont *en train* de se rejoindre) | Deux exports CSV qui supportent peu à peu les mêmes colonnes. | Attendre la stabilisation, puis extraire. |

[🔝 Retour en haut de page](#table-des-matières)

---

[← Nommer, commenter, mettre en forme](02-nommer-commenter-mettre-en-forme.md) · [↑ Sommaire](../README.md#table-des-matières) · [Conception orientée objet →](04-conception-orientee-objet.md)
