ARG PHP_VERSION
FROM php:${PHP_VERSION}

## Versão da Lib do Redis para PHP
ARG REDIS_LIB_VERSION=6.0.2

## Versão do NODE
ARG NODE_VERSION=21

## Diretório da aplicação
ARG APP_DIR=/var/www

### apt-utils é um extensão de recursos do gerenciador de pacotes APT
RUN apt-get update -y && apt-get install -y --no-install-recommends \
    apt-utils \
    supervisor

# dependências recomendadas de desenvolvido para ambiente linux
RUN apt-get update && apt-get install -y \
    zlib1g-dev \
    libzip-dev \
    unzip \
    libpng-dev \
    libpq-dev \
    libxml2-dev

# Instalando dependências do PHP
RUN docker-php-ext-install mysqli pdo pdo_mysql session xml intl zip simplexml pcntl gd

# habilita instalação do Redis
RUN pecl install redis-${REDIS_LIB_VERSION} \
    && docker-php-ext-enable redis

### Instalar e Habilitar o Swoole
RUN pecl install swoole
RUN docker-php-ext-enable swoole

# Instalação do Node NPM
RUN curl -sLS https://deb.nodesource.com/setup_$NODE_VERSION.x | bash - \
    && apt-get install -y nodejs \
    && npm install -g npm

# Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Instalando mysql-client para que o pacite spatie laravel-backup tenha acesso ao mysqldump
RUN apt update && apt-get install -y default-mysql-client

### Supervisor permite monitorar e controlar vários processos (LINUX)
### Bastante utilizado para manter processos em Daemon, ou seja, executando em segundo plano
COPY ./docker/supervisord/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

COPY ./docker/php/extra-php.ini "$PHP_INI_DIR/conf.d/99_extra.ini"
# COPY ./docker/php/extra-php-fpm.conf /usr/local/etc/php-fpm.d/www.conf

WORKDIR $APP_DIR
RUN cd $APP_DIR
RUN chown www-data:www-data $APP_DIR

COPY --chown=www-data:www-data . .
RUN rm -rf vendor
RUN composer install --no-interaction

### OCTANE
# RUN composer require laravel/octane

### NGINX
RUN apt-get install nginx -y
RUN rm -rf /etc/nginx/sites-enabled/* && rm -rf /etc/nginx/sites-available/*
COPY ./docker/nginx/server.conf /etc/nginx/sites-enabled/default.conf
COPY ./docker/nginx/error.html /var/www/html/error.html

RUN apt-get clean && rm -rf /var/lib/apt/lists/*
RUN apt update -y && apt install nano git -y

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]