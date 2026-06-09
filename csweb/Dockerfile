FROM php:8.2-apache-bullseye

# Use production php settings
RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"

# php-zip support
RUN set -eux; \
	apt-get update; \
	apt-get install -y zlib1g-dev libzip-dev unzip; \
	docker-php-ext-install zip; \
	docker-php-ext-install pdo_mysql


# enable mod_rewrite and allow override from .htacess files
RUN set -eux; \
	a2enmod rewrite; \
	sed -i '/<Directory \/var\/www\/>/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/apache2/apache2.conf

# copy and unzip local csweb source
COPY Data/csweb-8.0.1.zip /tmp/csweb.zip
RUN set -eux; \
	unzip /tmp/csweb.zip -d /var/www/html/; \
	chown -R www-data:www-data /var/www/html; \
	rm /tmp/csweb.zip

# generate configuration file at runtime via entrypoint
RUN echo '#!/bin/bash\n\
echo "<?php define(\"DBHOST\", \"${DB_HOST:-localhost}\"); define(\"DBUSER\", \"${DB_USER}\"); define(\"DBPASS\", \"${DB_PASSWORD}\"); define(\"DBNAME\", \"${DB_NAME}\"); define(\"ENABLE_OAUTH\", true); define(\"FILES_FOLDER\", \"/var/www/html/files\"); define(\"DEFAULT_TIMEZONE\", \"${DEFAULT_TIMEZONE:-UTC}\"); define(\"MAX_EXECUTION_TIME\", \"300\"); define(\"API_URL\", \"${API_URL:-http://localhost/api/}\"); define(\"CSWEB_LOG_LEVEL\" , \"error\"); define(\"CSWEB_PROCESS_CASES_LOG_LEVEL\", \"error\"); ?>" > /var/www/html/src/AppBundle/config.php\n\
exec docker-php-entrypoint "$@"' > /usr/local/bin/start.sh && chmod +x /usr/local/bin/start.sh

ENTRYPOINT ["/usr/local/bin/start.sh"]
CMD ["apache2-foreground"]
