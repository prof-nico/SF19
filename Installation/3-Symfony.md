# Installation

## Pré-requis

### PHP
Pour utiliser `Symfony 6`, vous devrez avoir installé `PHP 8.x`

Faites attention que vous aurez besoin de **deux versions** de PHP :
- PHP en tant que module Apache
- PHP en ligne de commande

Ces deux versions ne sont pas nécessairement en phase. De fait, vous pourrez par exemple utiliser PHP 8 via XAMPP pour afficher des pages, mais vous retrouver en PHP 7 en exécutant les commandes de la console.

Vous pouvez tout à fait installer plusieurs version différentes de PHP en parallèle, l'important est de bien configurer votre variable d'environnement `PATH` dans les variables d'environnement systèmes.

### Composer

Vous devez aussi installer `Composer` qui est le gestionnaire de paquetages de PHP et que vous pouvez télécharger depuis le site [https://getcomposer.org](https://getcomposer.org).

Des explications plus détaillées dans le fichier [Composer.md](Composer.md).

`Composer` permet d'exécuter un certain nombre de commandes dont :

* `create-project` : initialise un noveau projet, en ajoutant notamment des fichiers comme `composer.json` et les ressources d'auto-chargement des classes ;

### Symfony

En parallèle à Composer, Symfony propose son propre outil en ligne de commande `symfony` (écrit en `Go`) qui n'a toutefois pas la même vocation.

D'une part, il est centré sur Symfony, là où Composer est généraliste. Et il vise plutôt à la création et au déploiement d'environnements d'applications. Par exemple

* `new` : crée une nouvelle application (comme `composer create-project` mais avec une syntaxe un peu différentes)
* `init` : crée un projet d'après un squelette défini sur un dépôt distant
* `deploy` : déploie une application sur un serveur
* `console` : exécute une commande dans la console de Symfony

En dehors de ces commandes de base, `symfony` peut servir à configurer une infrastructure locale ou délocalisée (cloud) et à gérer un serveur HTTP.

```bash
# Lancement du serveur HTTP
symfony serve
# Arrêt du serveur HTTP
symfony server:stop
```

La commande `symfony` avait été dépréciée par Sensio Labs pendant un temps, mais elle est revenue et est maintenant [documentée](https://symfony.com/download).

L'installation recommandée se fait via les gestionnaires de paquetages des systèmes d'exploitation, comme `apt` (Linux), `brew` (macos) ou `scoop` (Windows). 
Un executable est aussi disponible pour Windows.

Ressource https://symfony.com/download

### Outils système

Pour écrire une application Symfony, il vous sera recommandé d'utiliser `git` (et éventuellement Github ou un autre système de partage).

En termes d'éditeur de code, les deux outils de prédilection sont bien sûr `PHPStorm` et `VSCode`. Mais `Atom` peut éventuellement faire l'affaire (mais pas `Notepad++`). Et `vim` ou `emacs` restent des possibilités intéressantes :wink: 

Enfin, avoir une connaissance de base des commandes du shell UNIX est naturellement un gain de temps ( cd, ls, ... ).

## Installation

Symfony peut s'installer sous deux modes différents :

* En tant que plate-forme de développement de site web ;
* En tant que noyau pour le développement d'applications spécifiques.

Dans le premier cas,le code global du « _framework_ » est installé avec ses quelque 30+ composants. La plupart sont utiles lorsque l'on veut développer des applications web courantes. Mais, évidemment, cela a un coût en termes d'espace disque et de temps de calcul lors de l'initialisation de l'application. L'avantage est d'avoir un outil prêt à l'emploi.

Pour installer cette version, on utilisera :
```bash
# Composer
composer create-project symfony/website-skeleton <directory>
# Symfony
symfony new --webapp <directory>
```

Dans le second cas, Symfony n'installe que le strict minimum. Ce sera au développeur d'installer un par un les paquetages dont il a besoin... ou d'écrire lui-même le code manquant.

Pour installer cette version, on utilisera :
```bash
# Composer
composer create-project symfony/skeleton <directory>
# Symfony
symfony new <directory>
```

Une fois la commande d'installation exécutée, Symfony peut vous demander quelques précisions. Celles-ci ont peu d'importances et peuvent être facilement modifiées par la suite dans les fichiers de configuraton.

### Flex

La deuxième procédure est encouragée par Symfony depuis le passage à Symfony Flex, lors de la transition en Symfony 3 et Symfony 4. Flex apporte des améliorations essentielles dans la gestion des paquetages, lors de leur installation... et de leur suppression. Dans les versions antérieures, les développeurs étaient souvent amenés à faire des modifications manuelles dans le code ou les fichiers de configuration, et on courait toujours le risque de ne pas avoir complètement fait les modifications nécessaires.

```bash
# Autre modalité d'installation
symfony new <directory>
composer require webapp
```

Flex introduit la notion de `recipe` (recette) qui est associée au paquetage et donne toutes les indications pour l'installation et la suppression de celui-ci. 

Les recettes, garantissant la facilité d'emploi des paquetages, permettent donc de partir du « _bare metal_ » et d'ajouter progressivment toutes les ressouces requises sans avoir à craindre de ne pas pouvoir les désinstaller. Le mode d'installation est donc une question de choix pratique pour l'équipe de développement.

Un petit avantage des recettes est aussi de fournir des alias pour certains des paquetages. Par exemple :
```bash
composer require mail
# ===
composer require symfony/mailer
```

Vous pouvez consulter un répertoire de nombreuses recettes sur le [dépôt Github](https://github.com/symfony/recipes/tree/flex/main) de Symfony. Et il est maintenant plus facile de développer ses prores recettes car Symfony a abandonné le contrôle centralisé de celles-ci (cf. [documentation](https://symfony.com/blog/symfony-flex-is-going-serverless)).

Ceci nous amène à considérer la structure même de la plate-forme Symfony, qui est en fait l'agrégation de composants indépendants les uns de autres, mais reliés entre eux par le **conteneur de services**, qui fera l'objet de modules dans ce cours. Cela fait que c'est un outil très souple, dans lequel chaque brique peut être remplacée par une autre, **sous la seule condition que cette dernière respecte les mêmes spécifications**.

#### PSR

Un certain nombre de spécifications sont maintenues par le collectif **PHP-FIG** sous l'appellation **PSR** (pour PHP Standards Recommendations).
 
 Elles se présentent sous la forme d'interfaces PHP qui indiquent comment implémenter une classe remplissant telle fonctionnalité et compatible avec tout un grand nombre d'applications, notamment développées par les acteurs de PHP-FIG. À titre d'exemple, voici l'interface correspondant à PSR-3, qui spécifie les méthodes de « _logging_ » (traçage) :

 ```php
namespace Psr\Log;

interface LoggerInterface
{
    /**
     * System is unusable.
     */
    public function emergency($message, array $context = array());

    /**
     * Action must be taken immediately.
     *
     * Example: Entire website down, database unavailable, etc. This should
     * trigger the SMS alerts and wake you up.
     */
    public function alert($message, array $context = array());

    /**
     * Critical conditions.
     *
     * Example: Application component unavailable, unexpected exception.
     */
    public function critical($message, array $context = array());

    /**
     * Runtime errors that do not require immediate action but should typically
     * be logged and monitored.
     */
    public function error($message, array $context = array());

    /**
     * Exceptional occurrences that are not errors.
     *
     * Example: Use of deprecated APIs, poor use of an API, undesirable things
     * that are not necessarily wrong.
     */
    public function warning($message, array $context = array());

    /**
     * Normal but significant events.
     */
    public function notice($message, array $context = array());

    /**
     * Interesting events.
     *
     * Example: User logs in, SQL logs.
     */
    public function info($message, array $context = array());

    /**
     * Detailed debug information.
     */
    public function debug($message, array $context = array());

    /**
     * Logs with an arbitrary level.
     */
    public function log($level, $message, array $context = array());
}
```
 
### Structure des applications.

Quelle est la structure d'une application Symfony une fois installée ?

1. `vendor`

Le dossier `vendor` contient la base de code nécesaire au fonctionnement du « _framework_ ». C'est-à-dire :
* le noyau de Symfony
* les composants de Symfony
* les bibliothèques tierces (i.e. d'autres éditeurs que Symfony ou vous-mêmes)

C'est dans ce dossier que sont installés les paquetages requis via `Composer`.

Il est évidemment fortement déconseillé de modifier le code présent à l'intérieur de ce dossier.

2. `var`

Ce dossier sert à maintenir les *caches* et les *logs* de l'application. En soi, ce n'est pas essentiel et son contenu peut être régulièrement effacé pour gagner de la place ou éviter les erreurs de cache (qui arrivent régulièrement). Il sera recréé automatiquement.

3. `bin`

Le dossier `bin` contient des scripts PHP reconnus par la commande `symfony`, mais exécutables indépendamment.

Par défaut, il contient deux scripts :
* `console` : la console de Symfony
* `phpunit` : l'outil de tests unitaires de PHP

Il est possible d'écrire ses propres scripts, bien entendu.

4. `config`

C'est le dossier de configuration de l'application, notamment des différentes bibliothèques installées dont le fonctionnement a besoin d'être paramétré.

5. `src`

Dans ce dossier, on mettra le code *de* l'application.

Dans les versions récentes, Symfony a abandonné la logique de décomposition des applications en « **bundles** », qui sont des unités applicatives. Ces derniers n'ont pas disparu — ils sont au fondement de l'architecture de Symfony, mais la logique est maintenant de considérer qu'une application est *un seul*  « bundle » et que les autres, si besoin il y a, sont des bibliothèques indépendantes qui doivent, en tant que telles, être déposées dans le dossier `vendor`. Cette philosophie n'est toutefois pas absolue. C'est plutôt une recommandation très forte.

En tant que « bundle », le dossier `src` a lui-même une structure très codifiée. On y trouvera (entre autres) les dossiers suivants :
* Controller
* Entity
* Repository
* DepencyInjection
* Form
* ainsi qu'un fichier Kernel.php

Tous les éléments de cette structure seront approfondis plus tard.

6. `templates`

Le dossier `templates` contient tous les squelettes de mise en page qui seront utilisés par Twig pour construire le code HTML des vues. L'organisation de ce dossier est arbitraire, ainsi que les noms des fichiers. La seule règle est dans l'utilisation des suffixes :
* `.html.twig` pour les fichiers au format Twig
* `.html.php` pour les fichiers PHP
* etc.

Par défaut, il existe dans ce dossier un fichier `base.html.twig` qui est utilié par Symfony lorsque rien n'a encore été développé.

7. `public`

Le dossier `public` est le seul qui doit être ouvert sur l'extérieur. C'est la racine de l'espace public du site.

Par défaut il contiendra le **contrôleur principal** de l'application, c'est-à-dire une [Façade](https://www.wikiwand.com/fr/Fa%C3%A7ade_(patron_de_conception)) qui est le point d'entrée unique de l'application. Dans certains cas, on peut envisager plusieurs points d'accès. Le dossier ne devrait, en tout état de cause, contenir aucun autre code.

Ce qu'on y trouvera par ailleurs seront toutes les ressources web de l'application : feuilles de style, scripts, images, etc. 

1. `tests`

Le dossier `tests` est destiné à contenir toutes les classes de tests automatisés concernant l'application.

Dans la mesure du possible, la convention veut que sa structure soit le reflet de celle du dossier `src` contenant le code de l'application.

9. Fichiers spéciaux

En plus de ces dossiers, on trouvera au moins un fichier `.env` qui contient les constantes d'environnement de l'application, qui font en fait partie des “super-globales” de PHP et qui sont accessibles comme telles.

Et pour finir les fichiers `composer.json` et `composer.lock` qui recensent les bibliothèques de l'application.


## Post-installation

Une fois la plate-forme installée, elle est immédiatement opérationnelle. On peut d'ailleurs consulter la page d'accueil dans un navigateur en exécutant le `index.php` du dossier `public`.

Cette page est spécifique (c'est indiqué en entête de la page) et nous ne la voyons *que* parce que rien n'a été défini pour le moment dans l'application. Il est donc inutile de chercher le code source de la page dans les dossier `src` ou `templates`, par exemple.

Au bas de la page, s'affiche un bandeau d'informations que l'on appelle le “profileur”. Il donne énormément d'informations sur l'exécution des requêtes. Nous le voyons parce que nous travaillons dans un « environnement » spécifique appelé `dev` (à savoir développement).

Un “environnement” constitue le cadre (ou contexte) dans lequel l'application s'exécute. Il en existe trois par défaut :
* `dev` active toutes les fonctionnalités “développeur”, notamment les bibliothèques installés pour cet environnement (cf. `require-dev`du fichier `composer.json`) ;
* `prod` active les fonctionnalités du site lorsqu'il est déployé en “production”. Dans ce cas, les fonctionnalités `dev` sont inhibées ;
* `test` qui est un environnement activé automatiquement lors de l'exécution de procédures des tests automatisés.

### Affichage des pages

Les URL de l'application varient en fonction du serveur HTTP que vous utilisez.

#### Serveur HTTP local

Si vous utilisez un serveur HTTP local ou intégré à une plateforme de type `*AMP`, celui-ci prendra comme référence le fichier `index.php` de l'application. Il faut donc écrire :
```bash
curl -X GET localhost/<chemin-du-dossier-racine>/public/index.php
```
Et pour toute route de l'application :
```bash
# Exemple pour la route /book/1
curl -X GET localhost/<chemin-du-dossier-racine>/public/index.php/book/1
```
Si on le souhaite, il est possible de retrouver des URL plus « intuitives » :
- soit en activant la réécriture d'URL dans un fichier `.htaccess` (pour les serveurs Apache)
  - [documentation](https://httpd.apache.org/docs/2.4/fr/mod/mod_rewrite.html)
  - [tutoriel](https://www.benmccann.com/url-rewriting-intro-with-apache-htaccess/)
- soit en configurant un « **hôte virtuel** », qui est un nom de domaine limité à votre propre machine.
  - [documentation](https://httpd.apache.org/docs/2.2/vhosts/)
  - [tutoriel pour Ubuntu mais transposable à d'autres contextes](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-20-04)

#### Serveur Symfony

Si vous utilisez le serveur intégré de Symfony (en fait celui de PHP), les URL sont immédiatement simplifiées, ce qui est sans doute l'un des rares avantages d'utilisation de ce serveur.
```bash
http://localhost/<chemin-du-dossier-racine>
# et
http -X GET localhost/<chemin-du-dossier-racine>/book/1
```


## Introduction à la ligne de commande

La console est un outil essentiel pour le développeur. Beaucoup d'opérations se font via le terminal, qui accélèrent et facilitent le travail d'écriture. Le commandes sont accessibles depuis la racine de l'application via les instructions :
```sh
# symfony
symfony console
# ou shell (si la commande symfony n'est pas installée
bin/console
```
Comme on peut le constater, les commandes sont *groupées* et dépendent généralement d'une bibliothèque particulière (comme `Doctrine`, par exemple, ou le composant `Maker`).

Le  nom de chaque commande est divisé en deux ou trois segments divisés par le caractère ':'. Le premier segment représente le groupe ; le dernier représente l'action.

Chaque commande dispose d'une aide que l'on peut consulter avec l'option `--help`. Exemple :
```sh
symfony console make:entity --help
```
## Ressources

* [Composer](https://get-composer.org)
* [La commande `symfony`](https://symfony.com/download)
* [Documentation Setup Symfony](https://symfony.com/doc/current/setup.html)
* [L'architecture en `bundle`](https://symfony.com/doc/current/bundles.html)
* [Les composants de Symfony](https://symfony.com/doc/current/components/index.html)
