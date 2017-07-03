## MariaDB Dockerfile


This repository contains **Dockerfile** of [MariaDB](https://mariadb.org/) for [Docker](https://www.docker.com/)'s [automated build](https://registry.hub.docker.com/u/dockerfile/mariadb/) published to the public [Docker Hub Registry](https://registry.hub.docker.com/).


### Base Docker Image

* [luckfellow/ubuntu](http://luckfellow.github.io/#/ubuntu)


### Installation

1. Install [Docker](https://www.docker.com/).

2. Download [automated build](https://registry.hub.docker.com/u/luckfellow/mariadb/) from public [Docker Hub Registry](https://registry.hub.docker.com/): `docker pull dockerfile/mariadb`

   (alternatively, you can build an image from Dockerfile: `docker build -t="luckfellow/mariadb" github.com/luckfellow/mariadb`)


### Usage

#### Run `mysqld-safe`

    docker run -d --name mysql -p 3306:3306 luckfellow/mariadb

#### Run `mysql`

    docker run -it --rm --link mysql:mysql luckfellow/mariadb bash -c 'mysql -h $MYSQL_PORT_3306_TCP_ADDR'