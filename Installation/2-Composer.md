# Composer

## Introduction

`Composer` est un gestionnaire de paquetages pour PHP.

Comme tous les outils de ce type, il repose sur des dépôts partagés de bibliothèques, dont le plus connu est [packagist.org](https://packagist.org).

## Installation

Les modes d'installation diffèrent selon les systèmes d'exploitation.

### Windows

La manière la plus simple est d'exécuter l'installeur `Composer-Setup.exe`. Celui-ci s'occupe de tout, notamment d'enregistrer dans la variable système PATH le chemin d'accès à l'exécutable de `Composer`.
**https://getcomposer.org/Composer-Setup.exe**


#### Vérification de l'installation
```bash 
composer -v 
```

Si cette commande ne renvoie rien, l'installation ne s'est pas bien passée.

La solution :
* copier le composer-setup.exe dans un dossier sur votre disque qui ne sera pas effacé,
* allez dans vos variables d'environnement systèmes,
* ajoutez dans **path** le chemin vers le fichier composer-setup.exe.
  
Fermez vos terminaux (et votre IDE ), et réessayez la commande ci-dessus.

### Linux/macOS

La manœuvre est un tout petit peu plus longue puisque vous devez télécharger l'installeur et l'exécuter manuellement.
```bash
# Téléchargement de l'installeur
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

# Vérification de l'intégrité du fichier
# Etape optionnelle mais sécurisante
php -r "if (hash_file('sha384', 'composer-setup.php') === '55ce33d7678c5a611085589f1f3ddf8b3c52d662cd01d4ba75c0ee0459970c2200a51f492d557530c71c15d8dba01eae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"

# Exécution de l'installeur
php composer-setup.php
# Déplacement de l'archive PHP (`.phar`) dans un répertoire partagé.
# Renommer composer.phar --> composer permet de transformer l'archive en commande système.
mv composer.phar /usr/local/bin/composer
# ou plus compact pour installer composer globalement : 
# php composer-setup.php --install-dir=/usr/local/bin --filename=composer

# Suppression de l'installeur
php -r "unlink('composer-setup.php');"
```

## Utilisation de Composer

### init

Pour initialiser un environnement de développement supportant `Composer`, vous pouvez commencer par :
```bash
composer init
```
Cette commande fait deux choses essentielles :

1. Elle crée le fichier `composer.json` qui décrit la configuration de l'application, en particulier toutes les dépendances qui seront nécessaires.
2. Elle installe toute la logique qui permettra à Composer de gérer l'autochargement des classes, piloté par le script `vendor/autoload.php`.

Le processus demande interactivement à l'utilisateur certaines informations de configuration de l'application, souvent facultatives et qui peuvent ensuite être facilement modifiées.

Dès lors, il est possible d'utiliser l'autochargement des classes dans n'importe quel programme PHP — indépendamment de Symfony. Il suffit d'injecter le script avec l'instruction :
```php
// $root désigne le répertoire racine de l'application
require_once("{$root}/vendor/autoload.php");
```

:warning: Symfony présente déjà un _autoloader_ intégré, qui contient cette ligne.

### require

C'est la commande la plus utilisée de Composer. Elle permet de déclarer une bibliothèque de code comme dépendance de l'application, en tenant compte récursivement de ses propres dépendances.

La commande `require` examine le contenu du fichier `composer.json` et résout toutes les dépendances pour déterminer — si cela est possible — quelles versions installer afin qu'elles soient l'ensmeble des bibliothèques inter-compatibles. Si aucune solution n'est trouvée, le fichier est restauré dans son état préalable.

Une fois cet écheveau de dépendances résolu, `require` fixe la cartographie des bibliothèques dans le fichier `composer.lock`. Celui-ci devrait faire partie des fichiers partagés dans le cadre d'un travail d'équipe.

Implicitment, `require` exécute en séquence la commande `install`. Il est donc inutile d'exécuter cette dernière. 

#### Nommage

Les noms de paquetages suivent une règle précise. Ils se composent de deux segments séparés par le caractère “`/`” , comme `symfony/routing` par exemple. Le premier segment désigne généralement le nom du « _vendor_ », c'est-à-dire le nom de l'auteur du paquetage, alors que le second segment est le nom du paquetage lui-même.

#### Dépôts

Composer extrait les paquetages de depôts, publics ou privés. Par défaut, il va chercher sur le site **packagist.org**, mais il est possible de configurer d'autres dépôts (comme des dépôts Github) comme nous le verrons plus tard.

#### Déclaration

Déclarer un paquetage se fait par la commande :
```bash
composer require nikic/php-parser
```
Chaque paquetage est référencé dans le fichier `composer.json` dans un bloc intitulé `require`. Certains paquetages ne sont utiles que dans la phase de développement, comme un générateur de données factices, par exemple. Dans ce cas, on peut préciser l'option :
```bash
composer require fakerphp/faker --dev
```
Le paquetage sera aors référencé dans un bloc `require-dev`.

#### Versions

Il est possible de préciser la version souhaitée du paquetage.

> **N.B.** Si vous publiez vous-mêmes des paquetages, vous devrez alors gérer les versions en utilisant des tags git afin d'aider les dépôts à choisir entre différentes versions.

Choisir une version nécessite de la préciser dans la commande :
```bash
composer require fakerphp/faker --dev 1.19.*
```

Le numéros de versions respectent la quadri-partition :
```<majeure>.<mineure>.<patch>-<stabilité>```

Si vous ne savez pas exactement quelle version installer, vous pouvez laisser la décision à Composer, qui choisira en fonction de la compatibilité avec d'autres paquetages déjà installés. La syntaxe est la suivante :

| Syntaxe      | Installe                 |
|--------------|--------------------------|
| 1.2.3        | 1.2.3-stable             |
| \>1.2        | \>1.2.0-stable           |
| \>=1.2       | \>=1.2.0-dev             |
| >=1.2-stable | \>=1.2.0-stable          |
| <1.3         | <1.3.0-dev               |
| <=1.3        | <=1.3.0-stable           |
| 1 - 2        | 	\>=1.0.0-dev <3.0.0-dev |
| ~1.3         | \>=1.3.0-dev <2.0.0-dev  |
| ~1.3.2       | \>=1.3.2-dev <1.3.0-dev  |
| ^1.3.2       | \>=1.3.2-dev <2.0.0-dev  |
| 1.4.*        | \>=1.4.0-dev <1.5.0-dev  |


### install

L'installation proprement dite est effectuée par la commande :
```bash
composer install
```
Composer lit alors le contenu du fichier `composer.json` et, s'il existe,  le fichier `composer.lock`. La stratégie peut s'avérer complexe car `install` tentera de suivre les contraintes du premier fichier, tout en respectant la situation décrite dans le second. Rappelons que celui-ci contient l'état _réel_ de l'application, telle qu'elle est vue par les différents acteurs du projet. Ne pas en tenir compte signifierait introduire des différences entre les diverses versions locales, ce qui n'est pas souhaitable.

Lors de ce processus, toutes les bibiothèques téléchargées sont déposées dans le dossier `vendor`, qui est par convention le dossier racine des bibliothèques tierces dans les applications PHP.

> **N.B.** Dans le cadre de Symfony, des traitements additionnels peuvent être effectués du fait du mécanisme des « _recipes_ » associées à Symfony Flex.

Il y a souvent une confusion entre `require`et `install`, mais la documentation officielle précise que la première commande (`require`) devrait toujours être privilégiée par rapport à la seconde. 

### update

`update`— ou `upgrade` qui est un alias — est la commande qui permet soit de créer, soit de mettre à jour le fichier `composer.lock`. En temps normal, il n'est pas nécessaire de l'exécuter puisque `require` la met à jour automatiquement à chaque déclaration de dépendance.

```bash
composer update
```
La commande ci-dessus met à jour tous les paquetages à leur version la plus récente possible, compte tenu des contraintes énoncées et des relations d'inter-compatibilité.

Vous pouvez mettre à jour quelques paquetages isolément :
```bash
# Tous les paquetages du vendor `symfony`
composer update symfony/*
# Un paquetage spécifique
composer update fakerphp/faker
```
Si vous souhaitez revenir à des versions plus anciennes, vous avez deux solutions pour cela :
1. changer manuellement le fichier `composer.json`
2. utiliser l'option `--with`
```bash
composer update --with easycorp/easyadmin-bundle:^3.5.0
```
L'option `--with` permet de revenir à une version antérieure et de résoudre en cascade toutes les dépendances des paquetages installés.

### remove

Dernière commande importante de `Composer`, `remove` permet de désinstaller des paquetages.
```bash
composer remove fakerphp/faker
```

> **N.B.** `remove` ne s'occupe pas de l'application, c'est pourquoi désinstaller des paquetages dans une application Symfony était souvent problématique. Depuis Symfony Flex et l'introduction des « _recipes_ », ces problèmes ont été résolus et tout se fait de manière transparente.


## Configuration

Toute la configuration de `Composer` est contenue dans le fichier `composer.json`, au-delà de la liste des paquetages installés. Le schéma complet de ce fichier est expliqué dans la [documentation officielle](https://getcomposer.org/doc/04-schema.md).

## Autres commandes

Composer sait exécuter bien d'autres commandes, comme `show` pour afficher les caractéristiques d'un paquetage, `browse` pour afficher la page web d'un paquetage, ou `suggests` pour lister une liste de paquetages suggérés pas les paquetages déjà installés, etc. 

Toute la documentation est disponible [sur cette page](https://getcomposer.org/doc/03-cli.md).