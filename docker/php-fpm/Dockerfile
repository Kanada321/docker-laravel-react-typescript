FROM php:8.2.17-fpm

# Composerのインストール & パス通す
COPY --from=composer:2.7.2 /usr/bin/composer /usr/bin/composer

COPY --from=node:20.12.2 /usr/local/bin /usr/local/bin
COPY --from=node:20.12.2 /usr/local/lib /usr/local/lib

RUN apt-get update && \
    apt-get -y install \
    git \
    zip \
    unzip \
    vim \
    libpq-dev \
    && docker-php-ext-install pdo_pgsql bcmath && \
    pecl install xdebug && \
    docker-php-ext-enable xdebug

ENV PATH="/var/www/application/node_modules/.bin:${PATH}"

WORKDIR /var/www/application

# Viteのサーバーにローカルからアクセスするために開ける
EXPOSE 5173
# Exxpose port 9000 and start php-fpm server
EXPOSE 9000
CMD ["php-fpm"]
