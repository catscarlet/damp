FROM debian:stretch-slim

COPY html/ /var/www/html/
ADD phpMyAdmin-4.9.0.1-all-languages.tar.xz /var/www/html
#ADD https://www.phpmyadmin.net/downloads/phpMyAdmin-latest-all-languages.tar.xz /var/www/html
COPY config.inc.php /var/www/html/phpMyAdmin-4.9.0.1-all-languages/
RUN chown www-data:www-data -R /var/www/html
VOLUME '/var/www/html/'
