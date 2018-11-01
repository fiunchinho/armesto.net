---
author: fiunchinho
categories:
- Code
date: 2018-10-30T23:47:55Z
tags:
- development
- docker
title: I didn't know you could do that with a Dockerfile!
url: /i-didnt-know-you-could-do-that-with-a-dockerfile/
thumbnail: "images/docker.png"
---

Docker released the [multi-stage builds feature](https://docs.docker.com/develop/develop-images/multistage-build/) in the version 17.05. This allowed developers to build smaller Docker images by using a final stage containing the minimum required for the application to work. Even though this is being used more and more over time, there are still multi-stage patterns that are not that widely used.

In this post I'll show you how to use multi-stage builds to:

* avoid having different Dockerfiles for every environment
* copy files from remote images
* use parameters in the `FROM` image

<!--more-->
Multi-stage builds allow us to use the `FROM` keyword in several places of the same Dockerfile.
Every time we use the `FROM` keyword a new stage will be created.
One important thing is that we can name these stages to refer to them later on in the Dockerfile.

## You don't need a Dockerfile for each environment
Imagine the following scenario: you need certain tools installed on the Docker image that you will use for development, but you don't want those tools on the final image that will be deployed in production.
For example, when developing in PHP it's useful to have `xdebug` installed, but you normally don't need it in production.


```bash
# Use this image as the base image for dev and prod.
FROM php:7.2-apache as common

# The pdo_mysql extension is required for both dev and prod.
RUN a2enmod rewrite; \
    chown -R www-data:www-data /var/www/html; \
    docker-php-ext-install pdo_mysql;

# Here we configure PHP, but this configuration will be overwritten for prod.
COPY php.ini /usr/local/etc/php/php.ini


# In this image we will download the dependencies, but without the development dependencies.
# The dependencies are installed in the vendor folder that will later be copied.
FROM composer as builder-dev

WORKDIR /app

# Copy only the files needed to download dependencies to avoid redownloading them when our code changes.
# This will download development/testing dependencies.
COPY composer.json composer.lock /app/
RUN composer install  \
    --ignore-platform-reqs \
    --no-ansi \
    --no-autoloader \
    --no-interaction \
    --no-scripts

# We need to copy our whole application so that we can generate the autoload file inside the vendor folder.
COPY . /app
RUN composer dump-autoload --optimize --classmap-authoritative


# This is the image using in development.
FROM common as dev

# We only install xdebug in development.
RUN pecl install xdebug-2.6.0; \
    docker-php-ext-enable xdebug

WORKDIR /var/www/html

# We enable the errors only in development.
ENV DISPLAY_ERRORS="On"

# Copy our application.
COPY . /var/www/html/
# Copy the downloaded dependencies from the previous stage.
COPY --from=builder-dev /app/vendor /var/www/html/vendor
# Setup PHP for development.
COPY php-dev.ini /usr/local/etc/php/php.ini
RUN composer dump-autoload --optimize --classmap-authoritative


# In this image we will download the dependencies, but without the development dependencies.
# The dependencies are installed in the vendor folder that will be copied later into the prod image.
FROM composer as builder-prod

WORKDIR /app

COPY composer.json composer.lock /app/
RUN composer install  \
    --ignore-platform-reqs \
    --no-ansi \
    --no-dev \
    --no-autoloader \
    --no-interaction \
    --no-scripts

# We need to copy our whole application so that we can generate the autoload file inside the vendor folder.
COPY . /app
RUN composer dump-autoload --optimize --no-dev --classmap-authoritative

# This is the image that will be deployed on production.
FROM common as prod

# Add label and exposed port for documentation.
LABEL maintainer="Jose Armesto <jose@armesto.net>"
EXPOSE 80

# No display errors to users in production.
ENV DISPLAY_ERRORS="Off"

# Copy our application
COPY . /var/www/html/
# Copy the downloaded dependencies from the builder-prod stage.
COPY --from=builder-prod /app/vendor /var/www/html/vendor
```

### Building our application image
When building the Docker image now we can choose which stage to build. The stages that will be executed depend on which stage we choose to build and the order in which these stages are defined in the Dockerfile.

If we are developing and need our `dev` version of the application, we can build the Docker image like

```
$ docker build --tag "my-awesome-app" --target "dev" .
```

This will only execute the Dockerfile lines of the stage named `dev`, and the stages that appear right before it: `common` and `builder-dev`.
If we would've placed the `builder-prod` stage before the definition of the `dev` stage (right after the `builder-dev` stage), the `builder-prod` stage lines would've got executed when building the `dev` stage, even though we don't need them.
So make sure to plan ahead and put the stages in the right order: every stage related to `dev` will be placed before any stage related to `prod`.

If we need the `prod` version that will be deployed in production, we can pass the `prod` target or just don't pass any target at all

```
$ docker build --tag "my-awesome-app" .
```

This will execute all the instructions in the Dockerfile, because the `prod` stage is the last one to appear on the Dockerfile.

### Can we improve the building speed for development?
It seems that building our docker image for development is kind of slow. Can we make it faster?
Our approach is copying our application files several times from our laptop to the image layer, making the process slow. And it will be slower as our application grows.

We can merge the `builder-dev` and the `dev` stages into one big stage to reduce the number of times we copy our application.
Let's remove the `builder-dev` stage and change our `dev` stage to this

```bash
# In the development image we download dependencies and copy our code to the image.
FROM common as dev

# We only want xdebug in development.
# We need to install some tools required by Composer, which will run in this stage.
RUN apt-get update; apt-get install -y wget zip unzip git; \
    pecl install xdebug-2.6.0; \
    docker-php-ext-enable xdebug; \
    wget https://raw.githubusercontent.com/composer/getcomposer.org/1b137f8bf6db3e79a38a5bc45324414a6b1f9df2/web/installer -O - -q | php -- --install-dir=/usr/bin --filename=composer --quiet

WORKDIR /var/www/html

# Copy only the files needed to download dependencies to avoid redownloading them when our code changes
COPY composer.json composer.lock /var/www/html/
RUN composer install  \
    --ignore-platform-reqs \
    --no-ansi \
    --no-autoloader \
    --no-interaction \
    --no-scripts

# We enable the errors only in development
ENV DISPLAY_ERRORS="On"

# Copy our application. Now the dependencies are already there.
COPY . /var/www/html/
# Setup PHP for development.
COPY php-dev.ini /usr/local/etc/php/php.ini
RUN composer dump-autoload --optimize --classmap-authoritative
```

Our development image will contain Composer (and wget, zip...) too, but I think that's not a big deal in this scenario.

## Parametrized image tags
Did you know that you can use parameters for the base image to use when building your image? We can define a parameter that sets the PHP version to use

```bash
ARG PHP_VERSION=7.2

FROM php:${PHP_VERSION}-apache as common
# Here would go the rest of the Dockerfile
# ...
```

Then just pass the right PHP version to use when building the image. If we want to use PHP v7.1 instead

```
$ docker build --tag "my-awesome-app" --build-arg "PHP_VERSION=7.1" .
```

### Combine targets with docker-compose
The reality is that many people use docker-compose while developing locally because it makes it really easy to start other containers along with your application, like a database that your application needs to store information.
In this scenario you can tell docker-compose to build your image and which target to use.

```yaml
version: '2.4'
services:
  web:
    build:
      context: .
      target: dev
      args:
        PHP_VERSION: ${PHP_VERSION}
    ports:
      - 8000:80
    volumes:
      - .:/var/www/html
    depends_on:
      - postgres
  postgres:
    image: postgres:11.0-alpine
    environment:
      POSTGRES_USER: dbuser
      POSTGRES_DB: my-db
```

You can now fire up your application and its dependencies as usual, but you can pass the desired PHP version as an environment variable

```
$ PHP_VERSION=7.1 docker-compose up --build
```

## Copying from remote images
We have seen how to copy files from previously generated layers.
But did you know that you can copy files from remote images?

For example, instead of installing Composer in our `dev` stage, we could just copy it from the official Composer image

```bash
# In the development image we download dependencies and copy our code to the image.
FROM common as dev

# We only want xdebug in development.
# We need to install some tools required by Composer, which will run in this stage.
RUN apt-get update; apt-get install -y zip unzip git; \
    pecl install xdebug-2.6.0; \
    docker-php-ext-enable xdebug;

# Copy composer binary from official Composer image.
COPY --from=composer /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/html

# Copy only the files needed to download dependencies to avoid redownloading them when our code changes
COPY composer.json composer.lock /var/www/html/
RUN composer install  \
    --ignore-platform-reqs \
    --no-ansi \
    --no-autoloader \
    --no-interaction \
    --no-scripts

# We enable the errors only in development
ENV DISPLAY_ERRORS="On"

# Copy our application. Now the dependencies are already there.
COPY . /var/www/html/
# Setup PHP for development.
COPY php-dev.ini /usr/local/etc/php/php.ini
RUN composer dump-autoload --optimize --classmap-authoritative
```
