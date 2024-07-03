# Этап 1: этап сборки
FROM php:8.3-fpm-alpine as build

# Установка системных зависимостей и php расширений
RUN apk add --no-cache \
	zip \
	libzip-dev \
	freetype \
	libjpeg-turbo \
	libpng \
	freetype-dev \
	libjpeg-turbo-dev \
	libpng-dev \
	nodejs \
	npm \
	&& docker-php-ext-configure zip \
	&& docker-php-ext-install zip pdo pdo_mysql \
	&& docker-php-ext-configure gd --with-freetype=/usr/include/ --with-jpeg=/usr/include/ \
	&& docker-php-ext-install -j$(nproc) gd \
	&& docker-php-ext-enable gd

# Установка composer
COPY --from=composer:2.7.6 /usr/bin/composer /usr/bin/composer

# Рабочая директория для проекта Laravel внутри контейнера
WORKDIR /var/www/html

# Копирование composer.json и composer.lock
COPY ../composer.json ./

# Копирование package.json и package-lock.json
COPY ../package.json ./

# Копирование остальных файлов Laravel
COPY .. .

# Установка php и node.js зависимостей
RUN composer install --no-dev --prefer-dist \
	&& npm install \
	&& npm run build

# Создание необходимых директорий и установка прав доступа
RUN mkdir -p /var/www/html/storage /var/www/html/bootstrap/cache \
	&& chown -R www-data:www-data /var/www/html \
	&& chmod -R 775 /var/www/html/storage \
	&& chmod -R 775 /var/www/html/bootstrap/cache

# Этап 2: этап продакшена
FROM php:8.3-fpm-alpine

# Установка nginx и дополнительных расширений PHP
RUN apk add --no-cache \
	zip \
	libzip-dev \
	freetype \
	libjpeg-turbo \
	libpng \
	freetype-dev \
	libjpeg-turbo-dev \
	libpng-dev \
	oniguruma-dev \
	gettext-dev \
	freetype-dev \
	nginx \
	&& docker-php-ext-configure zip \
	&& docker-php-ext-install zip pdo pdo_mysql \
	&& docker-php-ext-configure gd --with-freetype=/usr/include/ --with-jpeg=/usr/include/ \
	&& docker-php-ext-install -j$(nproc) gd \
	&& docker-php-ext-install bcmath \
	&& docker-php-ext-enable bcmath \
	&& docker-php-ext-install exif \
	&& docker-php-ext-enable exif \
	&& docker-php-ext-install gettext \
	&& docker-php-ext-enable gettext \
	&& docker-php-ext-install opcache \
	&& docker-php-ext-enable opcache \
	&& rm -rf /var/cache/apk/*

# Копирование файлов с этапа сборки
COPY --from=build /var/www/html /var/www/html
COPY ./deploy/nginx.conf /etc/nginx/http.d/default.conf
COPY ./deploy/php.ini "$PHP_INI_DIR/conf.d/app.ini"

WORKDIR /var/www/html

# Добавление всех папок, в которых хранятся файлы, требующие постоянного хранения
VOLUME ["/var/www/html/storage/app"]

# Запуск nginx и PHP-FPM
CMD ["sh", "-c", "nginx && php-fpm"]
