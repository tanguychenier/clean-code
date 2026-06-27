[↑ Sommaire](../README.md#table-des-matières) · [Nommer, commenter, mettre en forme →](02-nommer-commenter-mettre-en-forme.md)

# 1. Fondations et vocabulaire

## Introduction

Le terme *Clean Code* (« code propre ») a été popularisé par [Robert C. Martin](https://fr.wikipedia.org/wiki/Robert_C._Martin) dans son livre *Clean Code: A Handbook of Agile Software Craftsmanship* (2008). Il regroupe un ensemble de pratiques qui rendent le code plus lisible, plus facile à modifier et moins coûteux à maintenir.

Un code est dit *propre* lorsqu'un développeur autre que son auteur peut le lire, le comprendre et le faire évoluer sans avoir à mener l'enquête. Les exemples qui suivent sont écrits en PHP, mais les principes valent pour tout langage.

> **Que veut dire « PHP » ?** *PHP* (à l'origine *Personal Home Page*, aujourd'hui *PHP: Hypertext Preprocessor*) est un langage de programmation très répandu pour construire des sites et des applications web côté serveur (la partie qui tourne sur la machine qui héberge le site, par opposition à ce qui tourne dans le navigateur). En PHP, le signe `$` placé devant un mot signale une **variable**, c'est-à-dire une boîte qui contient une valeur. Quand vous lisez `$age` dans un bloc de code, lisez « la variable nommée age » : ce n'est pas une formule de mathématiques.

> **Que veut dire « maintenir » du code ?** *Maintenir* un logiciel, c'est continuer à le faire vivre après sa première écriture : corriger des bugs, ajouter des fonctionnalités, l'adapter à de nouveaux besoins. Pensez à l'entretien d'une maison : on n'arrête pas de s'en occuper une fois construite. La plupart du coût d'un logiciel se trouve dans cette phase d'entretien, pas dans l'écriture initiale ; voilà pourquoi la lisibilité compte autant.

> *« La règle du boy scout : laissez le campement plus propre que vous ne l'avez trouvé. »* (Robert C. Martin)
>
> Appliquée au code, cette règle pousse à améliorer un peu chaque fichier que l'on touche, plutôt que d'attendre une refonte hypothétique.

[🔝 Retour en haut de page](#table-des-matières)

## Glossaire

Le vocabulaire ci-dessous revient à plusieurs endroits. Chaque terme est aussi réexpliqué, avec une analogie, à sa première apparition dans le texte. Le tableau sert d'aide-mémoire auquel revenir au besoin.

| Terme | Définition courte | Pourquoi c'est utile |
|-------|-------------------|----------------------|
| **SRP** (*Single Responsibility Principle*, principe de responsabilité unique) | Une classe ou une fonction ne doit avoir qu'**une seule raison de changer**. | Un module qui répond à plusieurs raisons de changer concentre des risques de régression à chaque évolution. |
| **Cohésion** | Mesure du degré auquel les éléments d'un module (méthodes, attributs) participent à une **même finalité**. Forte cohésion = bon signe. | Une classe cohésive est facile à nommer, à tester, à comprendre. |
| **Couplage** | Mesure de la dépendance entre deux modules. Faible couplage = bon signe. | Moins deux modules se connaissent, plus on peut les modifier indépendamment. |
| **Magic number** (nombre magique) | Valeur littérale (`30`, `0.2`, `0.07`) inscrite dans le code sans nom ni explication. | Les remplacer par une constante nommée (`DELAI_PURGE_JOURS`, `TAUX_TVA`) rend l'intention explicite. |
| **Code smell** (mauvaise odeur de code) | Indice visuel qu'une portion de code mériterait un refactoring. Une *odeur* n'est pas un bug : c'est un signal. | Catalogue popularisé par Martin Fowler et Kent Beck dans *Refactoring*. |
| **Refactoring** (refactorisation) | Transformation **sans changement de comportement** observable, qui améliore la structure interne. | Permet d'améliorer la lisibilité ou la conception sans risque fonctionnel, à condition d'avoir des tests. |
| **DRY** (*Don't Repeat Yourself*) | Une règle ou une connaissance ne doit avoir qu'une seule source faisant autorité dans le système. | Évite la dérive entre copies et la duplication de la maintenance. |
| **KISS** (*Keep It Simple, Stupid*) | Préférer la solution la plus simple qui répond au besoin **actuel**. | La complexité gratuite est de la dette qui ne rend rien. |
| **YAGNI** (*You Aren't Gonna Need It*) | Ne pas implémenter ce dont on n'a **pas encore** besoin. | Évite le code mort, les options non testées, les chemins jamais empruntés. |
| **Optimisation prématurée** | Performances cherchées avant qu'un profilage ait identifié un goulot réel. Donald Knuth la qualifie de « racine de tout mal ». | Tordre la lisibilité pour un gain non mesuré coûte plus qu'il ne rapporte. |
| **Dette technique** | Coût futur d'un raccourci pris aujourd'hui (mauvais nom, copier-coller, test absent). Métaphore de Ward Cunningham. | Comme la dette financière, elle se rembourse avec des intérêts. |
| **Encapsulation** | Cacher l'état interne d'un objet derrière des méthodes qui contrôlent son accès et ses transitions. | Empêche les modules extérieurs de dépendre de détails internes susceptibles de changer. |
| **Polymorphisme** | Capacité, pour différentes classes partageant la même interface, de répondre à un même appel par leur propre comportement. | Remplace les longues cascades `if/else` ou `switch` par une dispatch implicite. |
| **Effet de bord** (*side effect*) | Toute action d'une fonction au-delà du retour de sa valeur : écrire un fichier, modifier un attribut, journaliser, envoyer un mail. | Les effets de bord rendent les fonctions difficiles à tester et à raisonner ; à isoler. |
| **Fonction pure** | Fonction sans effet de bord dont le résultat ne dépend que des arguments. Mêmes entrées = mêmes sorties. | Trivialement testable, mémoïsable, parallélisable. |
| **Loi de Déméter** | « Ne parler qu'à ses amis directs. » Un objet ne doit pas naviguer dans la structure interne d'un autre via des chaînes `a.b.c.d`. | Réduit le couplage et la fragilité aux changements internes. |
| **Tell, Don't Ask** | Plutôt qu'**interroger** un objet pour décider à sa place, **lui dire** quoi faire. | Concentre le comportement dans l'objet qui détient les données. |
| **CQS** (*Command-Query Separation*) | Une méthode est soit une **commande** (effet, retour `void`) soit une **requête** (lecture sans effet). Pas les deux. | Évite les surprises (« lire change l'état »). |
| **Clause de garde** (*guard clause*) | `if (...) return;` ou `throw` placé en début de fonction pour éliminer un cas particulier. | Aplanit l'imbrication, met le cas nominal au premier niveau. |

[🔝 Retour en haut de page](#table-des-matières)

---

[↑ Sommaire](../README.md#table-des-matières) · [Nommer, commenter, mettre en forme →](02-nommer-commenter-mettre-en-forme.md)
