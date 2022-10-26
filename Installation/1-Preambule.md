# Préambule

## **Introduction**

Le cours se base sur l'état de l'art du développement, c'est-à-dire que nous utiliserons principalement :
- PHP 8.x
- Symfony 6.x

Prévoyez aussi un IDE moderne comme PHPStorm (payant) ou VSCode (gratuit) et installez toutes les extensions qui vous faciliteront le travail. A priori, ces IDE intègrent pas défaut un analyseur statique qui vous signalera beaucoup d'erreurs PHP.

### Les extensions indispensables sur VSCode :
* PHP Intelephense
* PHP Namespace Resolver
* Symfony
* Twig
* Twig Language 2
* PHP Debug


## **Infrastructure système**

Il existe plusieurs stratégies pour profiter de l'ensemble des outils nécessaires au développement web.

### **Environnements ad hoc**

La manière la plus simple — surtout pour ceux qui travaillent sous Windows — est d'utiliser un environnement de type *AMP (MAMP, WAMP, XAMPP).

Dans ce cas, vous n'avez généralement pas grand-chose à faire. Une fois l'environnement installé, vous aurez un dossier `htdocs` ou `www` où vous déposerez le code source de vos applications.

Ces environnements ne sont pas toujours à jour et il est quelquefois nécessaire d'installer des versions plus récentes, notamment pour PHP ou PhpMyAdmin.

:nerd_face: Attention, Symfony embarque son propre serveur HTTP Apache. Le serveur Apache présent dans votre *AMP vous servira seulement au développement pour faire fonctionner PhpMyAdmin par exemple. 

### **Installation manuelle**

Pour les utilisateurs de macOS ou Linux, vous pouvez éventuellement installer les outils sur votre propre machine.

Sous macOS, vous aurez normalement déjà Apache et PHP. Néanmoins, vous devrez vérifier que vous avez les bonnes versions.

Pour installer des logiciels sur ces machines, vous pouvez utiliser `apt` sous Linux et `Homebrew` sous macOS. Il sera peut-être nécessaire d'installer ces gestionnaires de paquetages.

Il existe également un gestionnaire des paquetages pour Windows qui dont `scoop` et `Chocolatey`.

Tous ces gestionnaires permettent de procéder à des installations très facilement, en éxécutant des scripts souvent écrits par la communauté.

### Docker


### Louer un serveur

Une solution possible est aussi de louer temporairement un VPS (Virtual Private Server) chez un hébergeur quelconque. Les avantages sont nombreux :
- Vous avez une machine qui fonctionne comme un bac à sable
- Vous résiliez votre abonnement à peu près quand vous voulez
- Vous vous exercez à l'administration système
- Votre application est directement disponible en ligne
- Vous installez à peu près ce que vous voulez

L'inconvénient est bien sûr que cela demande un peu de temps au démarrage si vous n'avez pas l'habitude et pour les cours, nous préfererons travailler avec un environnement local.

## Rappels sur PHP 8

PHP 8 a amené énormément de changements dans la manière d'écrire le code. Certaines nouveautés ont été adoptés par Symfony, d'autres, qui auraient sans doute obligé à modifier le code en profondeur n'ont pour l'instant pas été reprises. Quelles sont les fonctionnaités les plus essentielles ?

### Le typage ( typehint )

Rappelons que le typage du code n'est pas une option. C'est une obligation (morale :o) qui permet plusieurs choses très importantes :
- le partage non ambigu du code (en lisant le code, je sais quel est son périmètre d'utilisation) ;
- la détection précoce des erreurs, l'analyse statique de votre IDE permet de détecter une grande partie des bugs.

Il existe das cas de figure où l'on a besoin de typage dynamique, mais cela reste une minorité des applications. On peut donc considérer que les langages comme Python ou JavaScript (et PHP < 5) souffrent de défaults de conception majeurs.

> **N.B.** JS a hérité de Scheme/Lisp et des fondations du lambda-calcul non typé. 

Le typage implique donc également une bonne utilisation des concepts de la programmation objet et de SOLID, notamment la déclaration d'interfaces pour les différentes classes de l'application.

> **N.B.** Les types respectent maintenant les contraintes de _substitution de Liskov_ :
> 1. si la méthode d'une classe Fille surcharge celle d'une classe Mère, ses paramètres doivent être d'un type égal ou plus générique.
> 
> 2. son type de retour doit être d'un type égal ou plus spécifique.

Où en est-on ?

A partir de PHP 8.1, nous avons un typage (presque) complet du code :
- typage des paramètres de fonction
- typage des valeurs de retour des fonctions
- typage des propriétés des classes
- typage algébrique :
  - Union de types
  - Intersection de types

Les types natifs sont également plus complets, avec `object` ou `mixed`, voire `never` qui signale qu'un fonction produisant **toujours** une exception ou un arrêt du programme. `null`, `true`, `false`sont également désormais des types.
```php
function trace(object $o): true
{
    var_dump($o::class);
}
```

> Deux choses manquent :
> - On ne peut pas typer les variables (qui ne sont jamais déclarées) ; en conséquence, on limitera au minimum l'usage de variables locales ;
> - Plus grave, on ne dispose pas en PHP de types génériques (e.g. ArrayCollection<User>) qui permettraient de déclarer des contraintes étendues ; ce manque a été compensé en partie par la communauté des analyseurs statiques de PHPStan ou Psalm en utilisant des annotations dans les DocBlocks.

### Attributs

Les attributs PHP 8 replacent maintenant dans bon nombre de cas les annotations DocBlock et Symfony ainsi que les principales bibliothèques s'y sont maintenant converties.

### Paramètres nommés

Une nouveauté très pratique venue de Python (notamment) consiste à pouvoir utiliser des paramètres nommés, ce qui permet deux choses importantes :
- À la lecture, les valeurs des arguments sont signifiantes (sachant que les IDE modernes le font également, en général) ;
- Nous pouvons nous abstraire de l'ordre des paramètres dans la signature de la fonction (dans une certaine mesure), ce qui est très pratique quand certains paramètres admettent des valeurs par défaut.

### Enums

Il est possible de créer des types énumérés, qui sont beaucoup plus efficaces pour gérer des contraintes sur des valeurs que les types scalaires ou les constantes.

### Paramètres de constructeur

Il est maintenant possible de mentionner la visibilité des propriétés directement dans la signature du constructeur des classes, ce qui évite de les déclarer explicitement (avec l'inconvénient d'être moins complet dans la documentation technique).

```php
<?php
class Point {
    public function __construct(protected int $x, protected int $y = 0) {
    }
}
```

Par ailleurs, les propriétés (et les classes) peuvent être déclarées `readonly` ce qui, de fait, permet de les rendre publiques.


> **N.B.** Il pourrait y avoir une longue discussion sur la question de savoir si c'est une bonne pratique en termes de conception de l'interface publique des classes :o)

Corollairement, il est maintenant interdit d'uiliser des propriétés qui n'ont pas été préalablement explicitement déclarées. Il est toutefois possible de passer outre, mais cela doit être considéré comme une mauvaise pratique. 

### Ajouts divers

#### match

Un switch qui est une expression

#### throw

throw peut être maintenant une expression

#### Opérateur « Null-safe »

détecte si un objet n'existe pas,  de manière récursive

```php
$x = $user?->getAddress()?->getCity;
```

#### ::class

fonctionne désormais sur les instances

#### Callables

Onpeut créer un objet `Callable` avec la syntaxe `...`.
```php
$f = $user->getName(...);
/*
 * Identique à :
 * $f = Closure::fromCallable([$user, 'getName'])
 * 
 * Crée un objet `Closure` qui peut ensuite être lié à différents contextes
 */
$name = $f();
```

