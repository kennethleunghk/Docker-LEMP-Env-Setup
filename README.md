# Native docker-based local environment for Drupal

Use this Docker compose file to spin up local environment for Drupal with a *native Docker app* on Linux, Mac OS X and Windows.

Docker4Drupal is designed to be used for local development, if you're looking for a dev/staging/production solution check out <a href="https://wodby.com" target="_blank">Wodby</a>. Use Wodby to deploy container-based infrastructure consistent with Docker4Drupal to any server.

---

* [Overview](#overview)
* [Instructions](#instructions)
* [Containers](#containers)
* [Multiple projects](#multiple-projects)
* [CI/CD](#cicd)
* [Docroot in subdirectory](#docroot-in-subdirectory)
* [Logs](#logs)
* [Status](#status)

## Overview

The Drupal bundle consist of the following containers:

| Container | Service name | Image | Public Port | Enabled by default |
| --------- | ------------ | ----- | ----------- | ------------------ |
| [Nginx](#nginx) | nginx | <a href="https://hub.docker.com/r/wodby/drupal-nginx/" target="_blank">wodby/drupal-nginx</a> | 8000 | ✓ |
| [PHP 7 / 5.6](#php) | php | <a href="https://hub.docker.com/r/wodby/drupal-php/" target="_blank">wodby/drupal-php</a> |  | ✓ |
| [MariaDB](#mariadb) | mariadb | <a href="https://hub.docker.com/r/wodby/drupal-mariadb/" target="_blank">wodby/drupal-mariadb</a> | | ✓ |
| [phpMyAdmin](#phpmyadmin) | pma | <a href="https://hub.docker.com/r/phpmyadmin/phpmyadmin" target="_blank">phpmyadmin/phpmyadmin</a> | 8001 | ✓ |
| [Mailhog](#mailhog) | mailhog | <a href="https://hub.docker.com/r/mailhog/mailhog" target="_blank">mailhog/mailhog</a> | 8002 | ✓ |
| [Redis](#redis) | redis | <a href="https://hub.docker.com/_/redis/" target="_blank">redis/redis</a> |||
| [Memcached](#memcached) | memcached | <a href="https://hub.docker.com/_/memcached/" target="_blank">_/memcached</a> |||
| [Solr](#solr) | solr | <a href="https://hub.docker.com/_/solr" target="_blank">_/solr</a> | 8003 ||
| [Varnish](#varnish) | varnish | <a href="https://hub.docker.com/r/wodby/drupal-varnish" target="_blank">wodby/drupal-varnish</a> | 8004 ||

PHP, Nginx, MariaDB and Varnish configs are optimized to be used with Drupal. We regularly update this bundle with performance improvements, bug fixes and newer version of Nginx/PHP/MariaDB.

## Instructions

__Feel free to adjust volumes and ports in the compose file for your convenience.__

Supported Drupal versions: 7 and 8

Supported PHP versions: 7.x and 5.6.x.

1\. Install docker for <a href="https://docs.docker.com/engine/installation/" target="_blank">Linux</a>, <a href="https://docs.docker.com/engine/installation/mac" target="_blank">Mac OS X</a> or <a href="https://docs.docker.com/engine/installation/windows" target="_blank">Windows</a>. __For Mac and Windows make sure you're installing native docker app version 1.12, not docker toolbox.__

For Linux additionally install <a href="https://docs.docker.com/compose/install/" target="_blank">docker compose</a>

2\. Download <a href="https://raw.githubusercontent.com/Wodby/drupal-compose/master/docker-compose.yml" target="_blank">the compose file</a> from this repository and put it to your Drupal project codebase (you might want to add docker-compose.yml to .gitignore).

3\. Since containers <a href="https://docs.docker.com/engine/tutorials/dockervolumes/" target="_blank">do not have a permanent storage</a>, directories from the host machine (volumes) should be mounted: one with code of your Drupal project and another with database files.

By default, the directory with the compose file (volume `./`) will be mounted to PHP container (assuming it's your codebase directory). Additionally `docker-runtime` directory will be created to store files for mariadb and, optionally, solr containers. Do not forget to add `docker-runtime` to your .gitignore file.

**Linux only**: fix permissions for your files directory with:
```bash
$ sudo chgrp -R 82 sites/default/files
$ sudo chmod -R 775 sites/default/files
```

4\. Choose Drupal version by modifying the following environment variable (could be 7 or 8) in the compose file:
```yml
DRUPAL_VERSION: 8
```

5\. Choose PHP version by modifying the name of the image:
```yml
image: wodby/drupal-php:7.0 # Allowed: 7.0, 5.6
```

6\. Update database credentials in your settings.php file:
```
database: drupal
username: drupal
password: drupal
host: mariadb
```

7\. If you want to import your database, uncomment the following line in the compose file:
```yml
#      - ./docker-runtime/mariadb-init:/docker-entrypoint-initdb.d # Place init .sql file(s) here
```

Create the volume directory `./docker-runtime/mariadb-init` in the same directory as the compose file and put there your SQL file(s). All SQL files will be automatically imported once MariaDB container has started.

8\. If you need to deploy one of the optional containers ([Redis](#redis), [Memcached](#memcached), [Apache Solr](#apache-solr)) uncomment the corresponding lines in the compose file.

9\. Now, let's run the compose file. It will download the images and run the containers:
```bash
$ docker-compose up -d
```

10\. Make sure all containers are running by executing:

```bash
$ docker-compose ps
```

11\. That's it! You drupal website should be up and running at http://localhost:8000.

## Containers

### Accessing containers

You can connect to any container by executing the following command:
```bash
$ docker-compose exec php sh
```

Replace `php` with the name of your service (e.g. `mariadb`, `nginx`, etc).

### Nginx

Nginx is being used as a web server. Nginx is pre-configured to be used with Drupal 7 and 8.

### PHP

PHP is used with Nginx via PHP-FPM. Currently PHP version 5.6 and 7 are provided. Check out [the instructions (step 5)](#instructions) to learn how to switch the version.

#### Drush

PHP container has installed drush. When running drush make sure to open the shell as user 82 (www-data) to avoid access problems in the web server, which is running as user 82, too:
```bash
$ docker-compose exec --user 82 php drush
```

Also, you can use preconfigured drush alias @dev:
```bash
$ docker-compose exec --user 82 php drush @dev
```

#### Composer

PHP container has installed composer. Example:
```bash
$ docker-compose exec --user 82 php composer update
```

#### Drupal Console

PHP container has installed drupal console. Example:
```bash
$ docker-compose exec --user 82 php drupal list
```

#### Xdebug

If you want to use Xdebug, uncomment this line to enable it in the compose file before starting containers:

```yml
PHP_XDEBUG_ENABLED: 1       # Comment out to disable (default).
```

If you would like to autostart Xdebug, uncomment this line:

```yml
PHP_XDEBUG_AUTOSTART: 1     # Comment out to disable (default).
```

#### Xdebug on Mac OS X

There are two more things that need to be done on Mac OS X in order to have Xdebug working. Uncomment `PHP_XDEBUG_ENABLED` to enable Xdebug and uncomment the following two lines:

```yml
PHP_XDEBUG_REMOTE_CONNECT_BACK: 0         # Disabled for remote.host to work (enabled by default)
PHP_XDEBUG_REMOTE_HOST: "10.254.254.254"  # Setting the host (localhost by default)
```

It is also needed to have localhost loopback alias with IP from above. You need this only once and that settings stays active until logout or restart.

```bash
sudo ifconfig lo0 alias 10.254.254.254
```

For more details see the issue with <a href="https://github.com/Wodby/drupal-php/issues/1" target="_blank">Xdebug in Mac OS</a>.

### MariaDB

#### Configuring

Many configuration options can be passed as flags without adjusting a cnf file. See example in the compose file:
```bash
#    command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

#### Import

Check out [the instructions (step 7)](#instructions) to learn how to import your existing database.

#### Export

Exporting all databases:

```bash
docker-compose exec mariadb sh -c 'exec mysqldump --all-databases -uroot -p"root-password"' > databases.sql
```

Exporting a specific database:

```bash
docker-compose exec mariadb sh -c 'exec mysqldump -uroot -p"root-password" my-db' > my-db.sql
```

### Redis

To spin up a container with Redis cache and use it as a default cache storage follow these steps:

1. Uncomment lines with redis service definition in the compose file.
2. Download and install <a href="https://www.drupal.org/project/redis" target="_blank">redis module</a>
3. Add the following lines to the settings.php file:

```php
$conf['redis_client_host'] = 'redis';
$conf['redis_client_interface'] = 'PhpRedis';
$conf['lock_inc'] = $contrib_path . '/redis/redis.lock.inc';
$conf['path_inc'] = $contrib_path . '/redis/redis.path.inc';
$conf['cache_backends'][] = 'sites/all/modules/redis/redis.autoload.inc';
$conf['cache_default_class'] = 'Redis_Cache';
$conf['cache_class_cache_form'] = 'DrupalDatabaseCache';
```

### Memcached

To spin up a container with Memcached and use it as a default cache storage follow these steps:

1. Uncomment lines with memcached service definition in the compose file.
2. Download and install <a href="https://www.drupal.org/project/memcache" target="_blank">memcache module</a>
3. Add the following lines to the settings.php file:

```php
$conf['cache_backends'][] = 'sites/all/modules/memcache/memcache.inc';
$conf['lock_inc'] = 'sites/all/modules/memcache/memcache-lock.inc';
$conf['memcache_stampede_protection'] = TRUE;
$conf['cache_default_class'] = 'MemCacheDrupal';
$conf['cache_class_cache_form'] = 'DrupalDatabaseCache';
$conf['memcache_servers'] = array('memcached:11211' => 'default');
```

### Memcached Admin

To spin up a container with the Memcached Admin User interface uncomment to memcached-admin service definition.
To get started visit http://localhost:8006/index.php

### Mailhog

By default, container with mailhog included in the bundle. It will catch all email sent from the PHP container. You can view emails by visiting its admin UI on localhost:8002.

### phpMyAdmin

By default, container with phpMyAdmin included in the bundle. You can access it by localhost:8001

### Apache Solr

To spin up a container with Apache Solr search engine uncomment lines with solr service definition in the compose file. Use  volume directory `./docker-runtime/solr` to access configuration files. Solr admin UI can be accessed by localhost:8003

### Varnish

To spin up a container with Varnish uncomment lines with varnish service definition in the compose file. Use the port specified in the compose file to access the website via Varnish.

## Multiple projects

To use D4D with multiple projects simply adjust the ports in the compose file, e.g. instead of ports 8000, 8001, 8002 you can use 7000, 7001, 7002.

## CI/CD

You can use docker4drupal containers for your test environment in your CI/CD workflow:
* <a href="https://blog.wodby.com/continuous-integration-and-delivery-drupal-docker-circleci-192c6ac97087#.gh8sggeze" target="_blank">CircleCI example</a>
* <a href="https://blog.wodby.com/drupal-8-ci-cd-with-jenkins-part-1-integration-eabd0f5c4f75#.obnyfv84i" target="_blank">Jenkins example</a>

## Docroot in subdirectory

If your docroot located in a subdirectory use options `PHP_DOCROOT` and `NGINX_DOCROOT` to specify the path (relative path inside the /var/www/html/ directory) for PHP and Nginx containers.

## Logs

To get logs from a container simply run (skip the last param to get logs form all the containers):
```
$ docker-compose logs [service]
```

Example: real-time logs of the PHP container:
```
$ docker-compose logs -f php
```

## Status

We're actively working on this instructions and containers. More options will be added soon. If you have a feature request or found a bug please submit an issue.

We update containers from time to time, to get the lastest changes simply run again:
```
$ docker-compose up -d
```
