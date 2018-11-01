---
author: fiunchinho
categories:
- Code
date: 2018-10-31T21:00:31Z
tags:
- development
- docker
title: You don't need different Dockerfiles
url: /you-dont-need-different-dockerfiles/
thumbnail: "images/docker.png"
---

Docker released the [multi-stage builds feature](https://docs.docker.com/develop/develop-images/multistage-build/) in the version 17.05. This allowed developers to build smaller Docker images by using a final stage containing the minimum required for the application to work. Even though this is being used more and more over time, there are still multi-stage patterns that are not that widely used.

In this post I'll show you how to use multi-stage builds to:
* avoid having different Dockerfiles for every environment
* copy files from remote images
* use parameters in the `FROM` image

<!--more-->
Imagine the following scenario: you need certain tools installed on the Docker image that you will use for development, but you don't want those tools on the final image that will be deployed in production.
For example, when developing in PHP it's useful to have `xdebug` installed, but you normally don't need it in production.


```docker
# Use this image as the base image for dev and prod
FROM php:7.2-apache as common

# The pdo_mysql extension is required for both dev and prod
RUN a2enmod rewrite; \
    chown -R www-data:www-data /var/www/html; \
    docker-php-ext-install pdo_mysql;

# Here we configure PHP, but this configuration will be overwritten for prod
COPY php.ini /usr/local/etc/php/php.ini


# In the development image we download dependencies and copy our code to the image
FROM common as dev

# We only want xdebug in development
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

# Copy our application and setup PHP for development
COPY . /var/www/html/
COPY php-dev.ini /usr/local/etc/php/php.ini
RUN composer dump-autoload --optimize --classmap-authoritative


# In this image we will download the dependencies, but without the development dependencies
# The dependencies are installed in the vendor folder that will later be copied
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

COPY . /app
RUN composer dump-autoload --optimize --no-dev --classmap-authoritative

# This is the image that will be deployed on production
FROM common as prod

# Add label and exposed port for documentation
LABEL maintainer="Jose Armesto <jose@armesto.net>"

EXPOSE 80

# No display errors to users in production
ENV DISPLAY_ERRORS="Off"

# Copy our application
COPY . /var/www/html/
# Copy the downloaded dependencies from the previous stage
COPY --from=builder-prod /app/vendor /var/www/html/vendor
```

### Building our application image
Now if you are developing and need your `dev` version of the application, you can build the Docker image like

```
$ docker build --tag "my-awesome-app" --target dev .
```

And if you need the `prod` version that will be deployed in production, you can pass the `prod` target or just don't pass any target at all

```
$ docker build --tag "my-awesome-app" .
```

### Can we improve the building speed for development?
It seems that building our docker image for development is kind of slow. Can we do it faster?
It seems that our approach is copying our application several times from our laptop to the image layer, making the process slow. And it will be slower as our application grows.
We can merge the `builder-dev` and the `dev` stages into one big stage to reduce the number of times we copy our application.

```docker
# Use this image as the base image for dev and prod
FROM php:7.2-apache as common

# The pdo_mysql extension is required for both dev and prod
RUN a2enmod rewrite; \
    chown -R www-data:www-data /var/www/html; \
    docker-php-ext-install pdo_mysql;

# Here we configure PHP, but this configuration will be overwritten for prod
COPY php.ini /usr/local/etc/php/php.ini


# In the development image we download dependencies and copy our code to the image
FROM common as dev

# We only want xdebug in development
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

# Copy our application and setup PHP for development
COPY . /var/www/html/
COPY php-dev.ini /usr/local/etc/php/php.ini
RUN composer dump-autoload --optimize --classmap-authoritative


# In this image we will download the dependencies, but without the development dependencies
# The dependencies are installed in the vendor folder that will later be copied
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

COPY . /app
RUN composer dump-autoload --optimize --no-dev --classmap-authoritative

# This is the image that will be deployed on production
FROM common as prod

# Add label and exposed port for documentation
LABEL maintainer="Jose Armesto <jose@armesto.net>"

EXPOSE 80

# No display errors to users in production
ENV DISPLAY_ERRORS="Off"

# Copy our application
COPY . /var/www/html/
# Copy the downloaded dependencies from the previous stage
COPY --from=builder-prod /app/vendor /var/www/html/vendor
```

Our development image will contain composer too, but I think that's not a big deal in this scenario.

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

## Parametrized image tags
Did you know that you can use parameters for the base image to use when building your image? We can define a parameter that sets the PHP version to use

```docker
ARG PHP_VERSION=7.2

FROM php:${PHP_VERSION}-apache as common
# Here would go the rest of the Dockerfile
# ...
# ..
# .
```

Then just pass the right PHP version to use when building the image

```
$ docker build --tag "my-awesome-app" --build-arg PHP_VERSION=7.1 .
```

## Copying from remote images
We have seen how to copy files from previously generated layers.
But did you know that you can copy files from remote images? For example, instead of installing Composer, we could just copy it from the official Composer image

```docker
# Use this image as the base image for dev and prod
FROM php:7.2-apache as common

# The pdo_mysql extension is required for both dev and prod
RUN a2enmod rewrite; \
    chown -R www-data:www-data /var/www/html; \
    docker-php-ext-install pdo_mysql;

# Here we configure PHP, but this configuration will be overwritten for prod
COPY php.ini /usr/local/etc/php/php.ini


# In the development image we download dependencies and copy our code to the image
FROM common as dev

# We only want xdebug in development
RUN apt-get update; apt-get install -y zip unzip git; \
    pecl install xdebug-2.6.0; \
    docker-php-ext-enable xdebug;

WORKDIR /var/www/html

# Copy composer binary from official Composer image
COPY --from=composer /usr/bin/composer /usr/bin/composer

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

# Copy our application and setup PHP for development
COPY . /var/www/html/
COPY php-dev.ini /usr/local/etc/php/php.ini
RUN composer dump-autoload --optimize --classmap-authoritative


# In this image we will download the dependencies, but without the development dependencies
# The dependencies are installed in the vendor folder that will later be copied
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

COPY . /app
RUN composer dump-autoload --optimize --no-dev --classmap-authoritative

# This is the image that will be deployed on production
FROM common as prod

# Add label and exposed port for documentation
LABEL maintainer="Jose Armesto <jose@armesto.net>"

EXPOSE 80

# No display errors to users in production
ENV DISPLAY_ERRORS="Off"

# Copy our application
COPY . /var/www/html/
# Copy the downloaded dependencies from the previous stage
COPY --from=builder-prod /app/vendor /var/www/html/vendor
```
