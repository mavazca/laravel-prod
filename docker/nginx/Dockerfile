# Argumento para versão do PHP
ARG PHP_VERSION=8.4

FROM php:${PHP_VERSION}-fpm

# Argumentos para o usuário
ARG user=code
ARG uid=1000

# Instalar dependências do sistema e ferramentas necessárias
RUN apt-get update && apt-get install -y \
    git \
    curl \
    zip \
    unzip \
    supervisor \
    ca-certificates \
    openssl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    libzip-dev \
    libbrotli-dev \
    gnupg \
    lsb-release \
    nginx \

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Instalar extensões PHP necessárias para Laravel
RUN docker-php-ext-install pdo pdo_mysql mysqli mbstring xml exif pcntl bcmath gd sockets intl zip

# Instalar redis
RUN pecl install -o -f redis \
    &&  rm -rf /tmp/pear \
    &&  docker-php-ext-enable redis

# Instalar Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Criar usuário para rodar a aplicação
RUN useradd -u $uid -G www-data,root -m -d /home/$user $user && \
    mkdir -p /home/$user/.composer && \
    chown -R $user:$user /home/$user

# Copiar código do Laravel para a pasta do servidor
COPY . /var/www

# Definir permissões corretas
RUN chown -R $user:www-data /var/www

# Copia o arquivo start-nginx
COPY start-nginx /usr/local/bin/start-nginx
RUN chmod +x /usr/local/bin/start-nginx

# Copia o arquivo do supervisord e php
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY php.ini /usr/local/etc/php/conf.d/custom.ini

# Copiar configurações do Nginx
COPY ./docker/nginx/nginx.conf /etc/nginx/nginx.conf
COPY ./docker/nginx/default.conf /etc/nginx/conf.d/default.conf

# Instalar Node.js e NPM
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && \
    apt-get install -y nodejs && \
    npm install -g npm@latest

# Expor portas HTTP e HTTPS
EXPOSE 80 443

# Definir diretório de trabalho
WORKDIR /var/www

# Trocar para usuário não-root para segurança
USER $user

# Comando de entrada
ENTRYPOINT ["start-nginx"]
