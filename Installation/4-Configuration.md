# Introduction à la configuration de Symfony

## Les fichiers d'environnement

Symfony a opté depuis la version 5 pour une configuration principalement basée de simples fichiers textes.

Par défaut, il existe un fichier nommé `.env` à la racine de l'application.

Ce fichier contient simplement des constantes d'environnement, comme par exemple :
```text
###> symfony/framework-bundle ###
APP_ENV=dev
APP_SECRET=057f3ed3e63b6fc5058fa34d8fcf30f7
# ADMIN_MAIL=michel.cadennes@sens-commun.fr
###< symfony/framework-bundle ###
```

Comme nous le voyons, le nom de chaque constante est en majucule et une valeur lui est assignée.

La particularité de ce fichier est que les constantes qui sont définies ici sont en fait des constantes PHP. Elles sont intégrées dans les variables super-globales et ont donc un statu qui dépasse le simple code de Symfony.

> **N.B.** Rappelons au passage qu'on ne doit jamais utilier directement dans une application les variables PHP, comme `$_GET` ou `$_SESSION`, pour diverses raisons dont la sécurité de l'application et qu'il existe des équivalents fournis par Symfony.
> On accédera à `$_GET` via la classe `Request` et à `$_SESSION`via la classe `Session`.

On remarque que les constantes sont souvent insérées entre des commentaires qui signalent à quel « bundle » ou blbliothèque est liée la constante en question. Dans l'exemple, il s'agit du `framework-bundle`, qui est le noyau de Symfony.

Naturellement, il est possible d'ajouter autant de constantes que l'on veut.

> **.B.** Le fichier `.env`permet en particulier de définir l'environnement dans lequel nous voulons travailler.
> 
> Il suffit pour cela d'assigner une valeur à la constante `APP_ENV`.
> 
> On peut vérifier que la valeur par défaut est `dev`

### Variantes du fichier par défaut

Le fichier .env peut être décliné pour plusieurs raisons :
1. Nous souhaitons avoir des configurations différentes en fonction des environnements de travail (`dev`, `prod`, `test`)
2. Nous souhaitons différencier les fonctionnements des dépôts locaux de celui du dépôt partagé (éventuellement en production)

#### Différents environnements

Un cas très parlant est : environnement de développement vs. environnement de test.

Nous souhaitons avoir deux bases de données différentes, une qui nous sert pour le développement, avec des données stables et une pour les tests que l'on peut écraser et reconstruire très souvent.

Il suffit alors de définir un fichier `.env.test` dans lequel nous définissons :
````text
###> doctrine/doctrine-bundle ###
DATABASE_URL="mysql://id:pass!@127.0.0.1:3306/app_test?serverVersion=8&charset=utf8mb4"
###< doctrine/doctrine-bundle ###
````

alors que pour la version de développement, nous aurions soit un fichier `env.dev`, soit le fichier par défaut :
````text
###> doctrine/doctrine-bundle ###
DATABASE_URL="mysql://id:pass!@127.0.0.1:3306/app?serverVersion=8&charset=utf8mb4"
###< doctrine/doctrine-bundle ###
````

#### Configurations partagées

De la même manière, il arrive que les membres d'une équipe n'aient pas tout à fait les mêmes configurations serveur locales, par exemple qu'ils n'utilisent pas les mêmes bases de données ou que celles ci aient des noms différents.

DAns ce cas, nous pouvons avoir recours à un fichier `.env.local` qui surchargera la configuration. Par exemple, j'utilise une base PostgreSQL sur machine personnelle :
````text
# .env.local

###> doctrine/doctrine-bundle ###
# DATABASE_URL="postgresql://app:!ChangeMe!@127.0.0.1:5432/app?serverVersion=14&charset=utf8"
###< doctrine/doctrine-bundle ###
````
et une base MySQL est la base de données partagée (qui se retrouvera donc en production). 
````text
# .env

###> doctrine/doctrine-bundle ###
DATABASE_URL="mysql://id:pass!@127.0.0.1:3306/app_test?serverVersion=8&charset=utf8mb4"
###< doctrine/doctrine-bundle ###
````

On comprend bien que les versions locales ne **doivent** pas être poussées sur un dépôt partagé, pour que chacun conserve son autonomie.

> **N.B.** Les deux modes peuvent être cumulés et il est possible d'avoir un fichier `.env.local.test`

## Configuration des éléments de l'application

La configuration d'une application Symfony est un sujet en soi et nous ne faisons ici que l'effleurer.

Nous entrerons davantage dans le détail lors des modules concernés, comme `Doctrine` ou la sécurité.

Symfony a adopté un mode de configuration très déclaratif, basé sur des fichiers `YAML` que l'on peut trouver dans le dossier `/config`.
Ce dossier contient par défaut :

1. un fichier `services.yaml` qui permet deux choses :
   - définir les paramètres de l'application
   - définir des « services », c'est-à-dire tout l'outillage logiciel « métier » (dit très rapidement)
2. un fichier `routes.yaml` qui permet de configurer les **routes** de l'application, ce qui correspond aux URL — soit l'API de l'application. En pratique, ce fichier est rarement utilisé, nous verrons pourquoi en étudiant les **contrôleurs**.
3. un fichier `bundles.php`, géré automatiquement par Symfony et qui contient une liste des « bundles » utilisés dans l'application.
4. un dossier `packages`qui permet de configurer tous les composants tiers de l'application, comme `Doctrine`, `Monolog`, `Security`, etc.
5. un dossier `routes` qui sert globalement à gérer certaines routes par défaut de Symfony (comme le profileur) et la configuration des routes. En pratique, là aussi, les fichiers sont très rarement modifiés. 

Tous ces éléments ne sont indispensables pour commencer à aborder Symfony et seront détaillés un peu plus loin.


### Paramètres calculés

Les paramètres définis dans les fichiers de configuration peuvent être « calculés » d'après d'autres paramètres ou d'après des constantes d'environnement.

Un cas typique est la constante `DATABASE_URL`, utilisée plus haut, et qui sert dans la configuration de Doctrine :
```yaml
doctrine:
    dbal:
        url: '%env(resolve:DATABASE_URL)%'
# ...
```

- les `%` signalent que la valeur est la valeur d'une variable
- la fonction `env(p)` signifie que `p` est définie dans un fichier `.env`
- `resolve` permet de résudre les variables imbriquées