[![CircleCI](https://circleci.com/gh/bitnami/bitnami-docker-opencart/tree/master.svg?style=shield)](https://circleci.com/gh/bitnami/bitnami-docker-opencart/tree/master)

# What is OpenCart?

> OpenCart is a free and open source e-commerce platform for online merchants. It provides a professional and reliable foundation for a successful online store.

http://www.opencart.com/

# TL;DR;

## Docker Compose

```bash
$ curl -sSL https://raw.githubusercontent.com/bitnami/bitnami-docker-opencart/master/docker-compose.yml > docker-compose.yml
$ docker-compose up -d
```

# Why use Bitnami Images?

* Bitnami closely tracks upstream source changes and promptly publishes new versions of this image using our automated systems.
* With Bitnami images the latest bug fixes and features are available as soon as possible.
* Bitnami containers, virtual machines and cloud images use the same components and configuration approach - making it easy to switch between formats based on your project needs.
* Bitnami images are built on CircleCI and automatically pushed to the Docker Hub.
* All our images are based on [minideb](https://github.com/bitnami/minideb) a minimalist Debian based container image which gives you a small base container image and the familiarity of a leading linux distribution.

# Supported tags and respective `Dockerfile` links

> NOTE: Debian 8 images have been deprecated in favor of Debian 9 images. Bitnami will not longer publish new Docker images based on Debian 8.

Learn more about the Bitnami tagging policy and the difference between rolling tags and immutable tags [in our documentation page](https://docs.bitnami.com/containers/how-to/understand-rolling-tags-containers/).


* [`3-ol-7`, `3.0.2-0-ol-7-r81` (3/ol-7/Dockerfile)](https://github.com/bitnami/bitnami-docker-opencart/blob/3.0.2-0-ol-7-r81/3/ol-7/Dockerfile)
* [`3-debian-9`, `3.0.2-0-debian-9-r57`, `3`, `3.0.2-0`, `3.0.2-0-r57`, `latest` (3/debian-9/Dockerfile)](https://github.com/bitnami/bitnami-docker-opencart/blob/3.0.2-0-debian-9-r57/3/debian-9/Dockerfile)

Subscribe to project updates by watching the [bitnami/opencart GitHub repo](https://github.com/bitnami/bitnami-docker-opencart).

# Prerequisites

To run this application you need Docker Engine 1.10.0. Docker Compose is recomended with a version 1.6.0 or later.

## Run OpenCart with a Database Container

Running OpenCart with a database server is the recommended way. You can either use docker-compose or run the containers manually.

### Run the application using Docker Compose

This is the recommended way to run OpenCart. You can use the following docker compose template:

```yaml
version: '2'
services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    environment:
      - MARIADB_USER=bn_opencart
      - MARIADB_DATABASE=bitnami_opencart
      - ALLOW_EMPTY_PASSWORD=yes
    volumes:
      - 'mariadb_data:/bitnami'
  opencart:
    image: 'bitnami/opencart:latest'
    environment:
      - MARIADB_HOST=mariadb
      - MARIADB_PORT_NUMBER=3306
      - OPENCART_DATABASE_USER=bn_opencart
      - OPENCART_DATABASE_NAME=bitnami_opencart
      - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - 'opencart_data:/bitnami'
    depends_on:
      - mariadb
volumes:
  mariadb_data:
    driver: local
  opencart_data:
    driver: local
```

### Run the application manually

If you want to run the application manually instead of using docker-compose, these are the basic steps you need to run:

1. Create a new network for the application and the database:

  ```bash
  $ docker network create opencart-tier
  ```

2. Create a volume for MariaDB persistence and create a MariaDB container

  ```bash
  $ docker volume create --name mariadb_data
  $ docker run -d --name mariadb \
    -e ALLOW_EMPTY_PASSWORD=yes \
    -e MARIADB_USER=bn_opencart \
    -e MARIADB_DATABASE=bitnami_opencart \
    --net opencart-tier \
    --volume mariadb_data:/bitnami \
    bitnami/mariadb:latest
  ```

  *Note:* You need to give the container a name in order to OpenCart to resolve the host

3. Create volumes for Opencart persistence and launch the container

  ```bash
  $ docker volume create --name opencart_data
  $ docker run -d --name opencart -p 80:80 -p 443:443 \
    -e ALLOW_EMPTY_PASSWORD=yes \
    -e OPENCART_DATABASE_USER=bn_opencart \
    -e OPENCART_DATABASE_NAME=bitnami_opencart \
    --net opencart-tier \
    --volume opencart_data:/bitnami \
    bitnami/opencart:latest
  ```

Then you can access the OpenCart storefront at http://your-ip/. To access the administration area, logon to http://your-ip/admin

  *Note:* If you want to access your application from a public IP or hostname you need to configure OpenCart for it. You can handle it adjusting the configuration of the instance by setting the environment variable `OPENCART_HOST` to your public IP or hostname.

## Persisting your application

If you remove the container all your data and configurations will be lost, and the next time you run the image the database will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed.

For persistence you should mount a volume at the `/bitnami` path. Additionally you should mount a volume for [persistence of the MariaDB data](https://github.com/bitnami/bitnami-docker-mariadb#persisting-your-database).

The above examples define docker volumes namely `mariadb_data` and `opencart_data`. The OpenCart application state will persist as long as these volumes are not removed.

To avoid inadvertent removal of these volumes you can [mount host directories as data volumes](https://docs.docker.com/engine/tutorials/dockervolumes/). Alternatively you can make use of volume plugins to host the volume data.

### Mount persistent folders in the host using docker-compose

This requires a minor change to the `docker-compose.yml` template previously shown:

```yaml
version: '2'

services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - MARIADB_USER=bn_opencart
      - MARIADB_DATABASE=bitnami_opencart
    volumes:
      - '/path/to/mariadb-persitence:/bitnami'
  opencart:
    image: 'bitnami/opencart:latest'
    environment:
      - OPENCART_DATABASE_USER=bn_opencart
      - OPENCART_DATABASE_NAME=bitnami_opencart
      - ALLOW_EMPTY_PASSWORD=yes
    depends_on:
      - mariadb
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - '/path/to/opencart-persistence:/bitnami'
```

### Mount host directories as data volumes using the Docker command line

In this case you need to specify the directories to mount on the run command. The process is the same than the one previously shown:

1. Create a network (if it does not exist):

  ```bash
  $ docker network create opencart-tier
  ```

2. Create a MariaDB container with host volume:

  ```bash
  $ docker run -d --name mariadb \
    -e ALLOW_EMPTY_PASSWORD=yes \
    -e MARIADB_USER=bn_opencart \
    -e MARIADB_DATABASE=bitnami_opencart \
    --net opencart-tier \
    --volume /path/to/mariadb-persistence:/bitnami \
    bitnami/mariadb:latest
  ```

  *Note:* You need to give the container a name in order to OpenCart to resolve the host

3. Create the OpenCart container with host volumes:

  ```bash
  $ docker run -d --name opencart -p 80:80 -p 443:443 \
    --net opencart-tier \
    --volume /path/to/opencart-persistence:/bitnami \
    bitnami/opencart:latest
  ```

# Upgrade this application

Bitnami provides up-to-date versions of MariaDB and OpenCart, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container. We will cover here the upgrade of the OpenCart container. For the MariaDB upgrade see https://github.com/bitnami/bitnami-docker-mariadb/blob/master/README.md#upgrade-this-image

1. Get the updated images:

  ```
  $ docker pull bitnami/opencart:latest
  ```

2. Stop your container

 * For docker-compose: `$ docker-compose stop opencart`
 * For manual execution: `$ docker stop opencart`

3. Take a snapshot of the application state

```bash
$ rsync -a /path/to/opencart-persistence /path/to/opencart-persistence.bkp.$(date +%Y%m%d-%H.%M.%S)
```

Additionally, [snapshot the MariaDB data](https://github.com/bitnami/bitnami-docker-mariadb#step-2-stop-and-backup-the-currently-running-container)

You can use these snapshots to restore the application state should the upgrade fail.

4. Remove the currently running container

 * For docker-compose: `$ docker-compose rm -v opencart`
 * For manual execution: `$ docker rm -v opencart`

5. Run the new image

 * For docker-compose: `$ docker-compose up opencart`
 * For manual execution ([mount](#mount-persistent-folders-manually) the directories if needed): `docker run --name opencart bitnami/opencart:latest`

# Configuration

## Environment variables

When you start the opencart image, you can adjust the configuration of the instance by passing one or more environment variables either on the docker-compose file or on the docker run command line.

##### User and Site configuration

 - `OPENCART_USERNAME`: OpenCart application User's First Name. Default: **user**
 - `OPENCART_PASSWORD`: OpenCart application password. Default: **bitnami1**
 - `OPENCART_EMAIL`: OpenCart application email. Default: **user@example.com**
 - `OPENCART_HOST`: OpenCart Host Server.
 - `OPENCART_HTTP_TIMEOUT`: Timeout in seconds used on http requests during wizard installation. Default: **120**

##### Use an existing database

- `MARIADB_HOST`: Hostname for MariaDB server. Default: **mariadb**
- `MARIADB_PORT_NUMBER`: Port used by MariaDB server. Default: **3306**
- `OPENCART_DATABASE_NAME`: Database name that OpenCart will use to connect with the database. Default: **bitnami_opencart**
- `OPENCART_DATABASE_USER`: Database user that OpenCart will use to connect with the database. Default: **bn_opencart**
- `OPENCART_DATABASE_PASSWORD`: Database password that OpenCart will use to connect with the database. No defaults.
- `ALLOW_EMPTY_PASSWORD`: It can be used to allow blank passwords. Default: **no**

##### Create a database for OpenCart using mysql-client

- `MARIADB_HOST`: Hostname for MariaDB server. Default: **mariadb**
- `MARIADB_PORT_NUMBER`: Port used by MariaDB server. Default: **3306**
- `MARIADB_ROOT_USER`: Database admin user. Default: **root**
- `MARIADB_ROOT_PASSWORD`: Database password for the `MARIADB_ROOT_USER` user. No defaults.
- `MYSQL_CLIENT_CREATE_DATABASE_NAME`: New database to be created by the mysql client module. No defaults.
- `MYSQL_CLIENT_CREATE_DATABASE_USER`: New database user to be created by the mysql client module. No defaults.
- `MYSQL_CLIENT_CREATE_DATABASE_PASSWORD`: Database password for the `MYSQL_CLIENT_CREATE_DATABASE_USER` user. No defaults.
- `ALLOW_EMPTY_PASSWORD`: It can be used to allow blank passwords. Default: **no**

If you want to add a new environment variable:

 * For docker-compose add the variable name and value under the application section:

```yaml
opencart:
  image: bitnami/opencart:latest
  ports:
    - 80:80
    - 443:443
  environment:
    - OPENCART_HOST=your_host
  volumes:
      - opencart_data:/bitnami
```

 * For manual execution add a `-e` option with each variable and value:

   ```bash
   $ docker run -d --name opencart -p 80:80 -p 443:443 \
     -e OPENCART_PASSWORD=my_password \
     --net opencart-tier \
     --volume /path/to/opencart-persistence:/bitnami \
     bitnami/opencart:latest
   ```

## SMTP Configuration

To configure OpenCart to send email using SMTP you can set the following environment variables:

 - `SMTP_HOST`: SMTP host.
 - `SMTP_PORT`: SMTP port.
 - `SMTP_USER`: SMTP account user.
 - `SMTP_PASSWORD`: SMTP account password.
 - `SMTP_PROTOCOL`: SMTP protocol.

This would be an example of SMTP configuration using a GMail account:

 * docker-compose:

```yaml
  opencart:
    image: bitnami/opencart:latest
    ports:
      - 80:80
      - 443:443
    environment:
      - MARIADB_HOST=mariadb
      - MARIADB_PORT_NUMBER=3306
      - OPENCART_DATABASE_USER=bn_opencart
      - OPENCART_DATABASE_NAME=bitnami_opencart
      - SMTP_HOST=smtp.gmail.com
      - SMTP_PORT=587
      - SMTP_USER=your_email@gmail.com
      - SMTP_PASSWORD=your_password
    volumes:
      - opencart_data:/bitnami
```

 * For manual execution:

   ```bash
   $ docker run -d --name opencart -p 80:80 -p 443:443 \
     -e MARIADB_HOST=mariadb \
     -e MARIADB_PORT_NUMBER=3306 \
     -e OPENCART_DATABASE_USER=bn_opencart \
     -e OPENCART_DATABASE_NAME=bitnami_opencart \
     -e SMTP_HOST=smtp.gmail.com \
     -e SMTP_PORT=587 \
     -e SMTP_USER=your_email@gmail.com \
     -e SMTP_PASSWORD=your_password \
     --net opencart-tier \
     --volume /path/to/opencart-persistence:/bitnami \
     bitnami/opencart:latest
   ```

# Contributing

We'd love for you to contribute to this container. You can request new features by creating an [issue](https://github.com/bitnami/bitnami-docker-opencart/issues), or submit a [pull request](https://github.com/bitnami/bitnami-docker-opencart/pulls) with your contribution.

# Issues

If you encountered a problem running this container, you can file an [issue](https://github.com/bitnami/bitnami-docker-opencart/issues). For us to provide better support, be sure to include the following information in your issue:

- Host OS and version
- Docker version (`$ docker version`)
- Output of `$ docker info`
- Version of this container (`$ echo $BITNAMI_IMAGE_VERSION` inside the container)
- The command you used to run the container, and any relevant output you saw (masking any sensitive information)

# License

Copyright 2016-2018 Bitnami

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

  <http://www.apache.org/licenses/LICENSE-2.0>

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
