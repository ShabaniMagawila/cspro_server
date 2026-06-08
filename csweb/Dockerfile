FROM php:8.2-apache-bullseye

# Use production php settings
RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"

# php-zip support
RUN set -eux; \
	sed -i "s#deb http://deb.debian.org/debian bullseye main#deb http://deb.debian.org/debian bullseye main contrib non-free#g" /etc/apt/sources.list.d/debian.sources; \
	apt-get update; \
	apt-get install -y zlib1g-dev libzip-dev unzip; \
	docker-php-ext-install zip; \
	docker-php-ext-install pdo_mysql


# enable mod_rewrite and allow override from .htacess files
RUN set -eux; \
	a2enmod rewrite; \
	sed -i '/<Directory \/var\/www\/>/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/apache2/apache2.conf

# download and unzip csweb source
RUN set -eux; \
	curl -o csweb.zip -fSL https://www.csprousers.org/downloads/cspro/csweb8.0.zip; \
	unzip csweb.zip -d /var/www/html/; \
	chown -R www-data:www-data /var/www/html; \
	rm csweb.zip

# configuration file
RUN set -eux; \
echo "<?php define('DBHOST', getenv('DB_HOST') ?: 'localhost'); define('DBUSER', getenv('DB_USER')); define('DBPASS', getenv('DB_PASSWORD')); define('DBNAME', getenv('DB_NAME')); define('ENABLE_OAUTH', true); define('FILES_FOLDER', '/var/www/html/files'); define('DEFAULT_TIMEZONE', getenv('DEFAULT_TIMEZONE') ?: 'UTC'); define('MAX_EXECUTION_TIME', '300'); define('API_URL', 'http://localhost/api/'); define('CSWEB_LOG_LEVEL' , 'error'); define('CSWEB_PROCESS_CASES_LOG_LEVEL', 'error'); ?>" > /var/www/html/src/AppBundle/config.php
