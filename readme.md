Full Dev Stack
===================
What does this dev stack can help you with?
* Starts Symfony project in docker as easy as possible in dev mode
* Starts MySQL with PhpMyAdmin and Nginx also in docker containers
* Easy simulates prod mode on your local dev computer if needed
* Setups MySQL, PhpMyAdmin and Symfony right from the box
* Php dockerfile is highly customisable but still as small as possible
* Includes APCu, OPcache, Redis and Xdebug (only for dev)
* Includes optional MySQLi, Memcached and other
* Includes gitlab-CI for full CI/CD

## Installation
##### 1. Start new git project and pull all files
```bash
git init
git remote add dockerimage https://github.com/toteaIT/docker-dev-stack-symfony.git
git pull dockerimage master
```
##### 2. Build docker images (this will take few minutes during first run, next run is in seconds)  
```bash
docker-compose build
```
##### 3. Run docker containers with your project name as variable
```bash
COMPOSE_PROJECT_NAME=myproject docker-compose up -d
```
##### 4. Get Symfony
Copy your Symfony existing project into actual directory (where docker-compose.yml is) or create new project with composer.

## Requirements
* Docker installed
* git installed
* no need for PHP, Composer, DB server, web server

Server Dev
===================
Example of docker-compose.yml on development server. This file is ran by gitlab-ci.yml via SSH. 