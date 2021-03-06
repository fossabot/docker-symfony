# Docker Symfony (PHP7-FPM - NGINX - MySQL - ELK - PhpMyAdmin)

[![Build Status](https://travis-ci.org/DarwinOnLine/docker-symfony.svg?branch=master)](https://travis-ci.org/DarwinOnLine/docker-symfony)
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2FDarwinOnLine%2Fdocker-symfony.svg?type=shield)](https://app.fossa.io/projects/git%2Bgithub.com%2FDarwinOnLine%2Fdocker-symfony?ref=badge_shield)

![](doc/schema.png)

Docker-symfony gives you everything you need for developing Symfony application. This complete stack run with docker and [docker-compose (1.7 or higher)](https://docs.docker.com/compose/).

## Installation

1. Create a `.env` from the `.env.dist` file. Adapt it according to your symfony application

    ```bash
    cp .env.dist .env
    ```
    
    Customisable vars :
      * `COMPOSE_PROJECT_NAME`: Your project name, the containers will be prefixed by this name. Snake case is advised.
      * `SYMFONY_APP_PATH`: For my personal use, this is set to `..`, because I embed this whole repo in a `.docker` directory in my Symfony projects.
      * `MYSQL_ROOT_PASSWORD`: Your MySQL root password.
      * `MYSQL_DATABASE`: Your MySQL database.
      * `MYSQL_USER`: You database user.
      * `MYSQL_PASSWORD`: Your database user password.
      * `TIMEZONE`: The project timezone
      * `SITE_URL`: Your project url ([symfony.local](http://symfony.local) by default)

2. Build/run containers with (with and without detached mode)

    ```bash
    $ docker-compose build
    $ docker-compose up -d
    ```

3. Update your system host file (add symfony.local)

    ```bash
    # UNIX only: get containers IP address and update host (replace IP according to your configuration) (on Windows, edit C:\Windows\System32\drivers\etc\hosts)
    $ sudo echo $(docker network inspect bridge | grep Gateway | grep -o -E '[0-9\.]+') "symfony.local" >> /etc/hosts
    ```

    **Note:** For **OS X**, please take a look [here](https://docs.docker.com/docker-for-mac/networking/) and for **Windows** read [this](https://docs.docker.com/docker-for-windows/#/step-4-explore-the-application-and-run-examples) (4th step).

4. Prepare Symfony app
    1. Update app/config/parameters.yml

        ```yml
        # path/to/your/symfony-project/app/config/parameters.yml
        parameters:
            database_host: db
        ```

    2. Composer install & create database

        ```bash
        $ docker-compose exec php bash
        $ composer install
        # Symfony2
        $ sf doctrine:database:create
        $ sf doctrine:schema:update --force
        # Only if you have `doctrine/doctrine-fixtures-bundle` installed
        $ sf doctrine:fixtures:load --no-interaction
        # Symfony3
        $ sf3 doctrine:database:create
        $ sf3 doctrine:schema:update --force
        # Only if you have `doctrine/doctrine-fixtures-bundle` installed
        $ sf3 doctrine:fixtures:load --no-interaction
        ```

5. Enjoy :-)

## Usage

Just run `docker-compose up -d`, then:

* Symfony app: visit [symfony.local](http://symfony.local)  
* Symfony dev mode: visit [symfony.local/app_dev.php](http://symfony.local/app_dev.php)  
* Logs (Kibana): [symfony.local:81](http://symfony.local:81)
* Logs (files location): logs/nginx and logs/symfony
* PhpMyAdmin: [symfony.local:82](http://symfony.local:82)

## Customize

If you want to add optionnals containers like Redis... take a look on [doc/custom.md](doc/custom.md).

## How it works?

Have a look at the `docker-compose.yml` file, here are the `docker-compose` built images:

* `db`: This is the MySQL database container,
* `php`: This is the PHP-FPM container in which the application volume is mounted,
* `nginx`: This is the Nginx webserver container in which application volume is mounted too,
* `elk`: This is a ELK stack container which uses Logstash to collect logs, send them into Elasticsearch and visualize them with Kibana.
* `phpmyadmin`: This is the PhpMyAdmin container, to manage MySQL database.

This results in the following running containers:

```bash
$ docker-compose ps
           Name                          Command               State              Ports            
--------------------------------------------------------------------------------------------------
symfony_project_db            /entrypoint.sh mysqld            Up      0.0.0.0:3306->3306/tcp      
symfony_project_elk           /usr/bin/supervisord -n -c ...   Up      0.0.0.0:81->80/tcp          
symfony_project_nginx         nginx                            Up      443/tcp, 0.0.0.0:80->80/tcp
symfony_project_php           php-fpm                          Up      0.0.0.0:9000->9000/tcp
symfony_project_phpmyadmin    /run.sh phpmyadmin               Up      0.0.0.0:82->80/tcp, 9000/tcp      
```

## Useful commands

```bash
# bash commands
$ docker-compose exec php bash

# Composer (e.g. composer update)
$ docker-compose exec php composer update

# SF commands (Tips: there is an alias inside php container)
$ docker-compose exec php php /var/www/symfony/app/console cache:clear # Symfony2
$ docker-compose exec php php /var/www/symfony/bin/console cache:clear # Symfony3
# Same command by using alias
$ docker-compose exec php bash
$ sf cache:clear

# Retrieve an IP Address (here for the nginx container)
$ docker inspect --format '{{ .NetworkSettings.Networks.dockersymfony_default.IPAddress }}' $(docker ps -f name=nginx -q)
$ docker inspect $(docker ps -f name=nginx -q) | grep IPAddress

# MySQL commands
$ docker-compose exec db mysql -uroot -p"root"

# F***ing cache/logs folder
$ sudo chmod -R 777 app/cache app/logs # Symfony2
$ sudo chmod -R 777 var/cache var/logs var/sessions # Symfony3

# Check CPU consumption
$ docker stats $(docker inspect -f "{{ .Name }}" $(docker ps -q))

# Delete all containers
$ docker rm $(docker ps -aq)

# Delete all images
$ docker rmi $(docker images -q)
```

## FAQ

* Got this error: `ERROR: Couldn't connect to Docker daemon at http+docker://localunixsocket - is it running?
If it's at a non-standard location, specify the URL with the DOCKER_HOST environment variable.` ?  
Run `docker-compose up -d` instead.

* Permission problem? See [this doc (Setting up Permission)](http://symfony.com/doc/current/book/installation.html#checking-symfony-application-configuration-and-setup)

* How to config Xdebug?
Xdebug is configured out of the box!
Just config your IDE to connect port  `9001` and id key `PHPSTORM`

## Credits

This repo is first created for my personal use (but feel free to use it !), the most of the original work comes from :
* [Vincent Composieux](https://github.com/eko/docker-symfony)
* [Maxence Poutord](https://github.com/maxpou/docker-symfony)


## License
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2FDarwinOnLine%2Fdocker-symfony.svg?type=large)](https://app.fossa.io/projects/git%2Bgithub.com%2FDarwinOnLine%2Fdocker-symfony?ref=badge_large)