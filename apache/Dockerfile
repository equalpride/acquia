FROM heremedia/acquia-base
#FROM heremedia/acquia-base
MAINTAINER Nicholas Alipaz <nicholas.alipaz@heremedia.com>

COPY container /

############ Setup Apache ############
ENV APACHE2_MAJOR_VERSION=2.2\
 APACHE2_MINOR_VERSION=22
ENV APACHE2_VERSION=${APACHE2_MAJOR_VERSION}.${APACHE2_MINOR_VERSION}

RUN printf "$(cat /tmp/APACHE2_PACKAGES)" "${APACHE2_VERSION}" | xargs apt-get install --no-install-recommends -y &&\
 apt-get clean

# Disable modules
RUN cat /tmp/APACHE2_MODULES_TO_DISABLE | xargs a2dismod &&\
 cat /tmp/APACHE2_MODULES_TO_ENABLE | xargs a2enmod

# Tweak conf
# Set ServerName - to avoid the warning message "apache2 could not fully qualified domain name"
# Configure log
RUN readonly APACHE2_CONF_FILE="/etc/apache2/apache2.conf" &&\
 echo "ServerName localhost" >> "${APACHE2_CONF_FILE}" &&\
 sed -i -r\
 -e 's!^(\s*CustomLog)\s+\S+!\1 /proc/self/fd/1!g'\
 -e 's!^(\s*ErrorLog)\s+\S+!\1 /proc/self/fd/2!g'\
 ${APACHE2_CONF_FILE} &&\
 a2dissite default &&\
 a2ensite any-site &&\
 service apache2 start &&\
 service apache2 stop

############ Setup PHP ############
ENV PHP_MAJOR_VERSION=5.6\
 PHP_MINOR_VERSION=24
ENV PHP_VERSION=${PHP_MAJOR_VERSION}.${PHP_MINOR_VERSION}\
 PHP_EXTENSION_GNUPG_VERSION=1.4.0\
 PHP_EXTENSION_HTTP_VERSION=2.5.5\
 PHP_EXTENSION_IGBINARY_VERSION=1.2.1\
 PHP_EXTENSION_IMAGICK_VERSION=3.3.0\
 PHP_EXTENSION_MEMCACHE_VERSION=3.0.8\
 PHP_EXTENSION_MONGO_VERSION=1.6.12\
 PHP_EXTENSION_OAUTH_VERSION=1.2.3\
 PHP_EXTENSION_PROPRO_VERSION=1.0.2\
 PHP_EXTENSION_RAPHF_VERSION=1.1.2\
 PHP_EXTENSION_SSH2_VERSION=BETA\
 PHP_EXTENSION_UPLOADPROGRESS_VERSION=1.0.3.1\
 PHP_EXTENSION_XDEBUG_VERSION=2.4.1\
 PHP_EXTENSION_XHPROF_VERSION=BETA\
 PHP_CONF=/usr/local/php${PHP_MAJOR_VERSION}/etc/cli/php.ini
ENV PECL_STRING="gnupg-${PHP_EXTENSION_GNUPG_VERSION}\
 igbinary-${PHP_EXTENSION_IGBINARY_VERSION}\
 imagick-${PHP_EXTENSION_IMAGICK_VERSION}\
 memcache-${PHP_EXTENSION_MEMCACHE_VERSION}\
 mongo-${PHP_EXTENSION_MONGO_VERSION}\
 OAuth-${PHP_EXTENSION_OAUTH_VERSION}\
 propro-${PHP_EXTENSION_PROPRO_VERSION}\
 raphf-${PHP_EXTENSION_RAPHF_VERSION}\
 ssh2-${PHP_EXTENSION_SSH2_VERSION}\
 uploadprogress-${PHP_EXTENSION_UPLOADPROGRESS_VERSION}\
 xdebug-${PHP_EXTENSION_XDEBUG_VERSION}\
 xhprof-${PHP_EXTENSION_XHPROF_VERSION}"

WORKDIR /tmp/php/src/${PHP_VERSION}
# required packages
RUN cat /tmp/PHP_DEV_PACKAGES | xargs apt-get install --no-install-recommends -y &&\
 cat /tmp/PHP_PACKAGES | xargs apt-get install --no-install-recommends -y &&\
 curl -sSL -o php-${PHP_VERSION}.tar.gz https://php.net/get/php-${PHP_VERSION}.tar.gz/from/this/mirror &&\
 tar -xzf php-${PHP_VERSION}.tar.gz &&\
 ln -sf /usr/lib/libc-client.a /usr/lib/x86_64-linux-gnu/libc-client.a &&\
 cd php-${PHP_VERSION} &&\
 printf -- "$(cat /tmp/PHP_FLAGS)" "${PHP_MAJOR_VERSION}" "${PHP_MAJOR_VERSION}" "${PHP_MAJOR_VERSION}" | xargs ./configure &&\
 make &&\
 make install &&\
 cp php.ini-production ${PHP_CONF}
RUN sed -i -r\
 -e 's!^display_errors = .*$!display_errors = STDOUT!g'\
 -e 's!^enable_dl = .*$!enable_dl = On!g'\
 -e 's!^expose_php = .*$!expose_php = Off!g'\
 -e 's!^ignore_repeated_errors = .*$!ignore_repeated_errors = On!g'\
 -e 's!^mail.add_x_header = .*$!mail.add_x_header = Off!g'\
 -e 's!^short_open_tag = .*$!short_open_tag = On!g'\
 -e 's!^memory_limit = .*$!memory_limit = 512M!g'\
 -e 's!^post_max_size = .*$!post_max_size = 256M!g'\
 -e 's!^request_order = .*$!request_order = GP!g'\
 -e 's!^upload_max_filesize = .*$!upload_max_filesize = 256M!g'\
 -e 's!^variables_order = .*$!variables_order = EGPCS!g' "${PHP_CONF}" &&\
 echo "realpath_cache_size = 900K" >> "${PHP_CONF}" &&\
 echo "realpath_cache_ttl = 300" >> "${PHP_CONF}" &&\
 echo "sendmail_path = /usr/sbin/sendmail -t -i" >> "${PHP_CONF}" &&\
 echo "error_log = syslog" >> "${PHP_CONF}" &&\
 echo "date.timezone = UTC" >> "${PHP_CONF}" &&\
 rm -rf /tmp/php &&\
 ln -sf /usr/local/php${PHP_MAJOR_VERSION}/bin/php /usr/local/bin/php &&\
 ln -sf /usr/local/php${PHP_MAJOR_VERSION}/bin/pecl /usr/local/bin/pecl &&\
 pecl config-set php_ini ${PHP_CONF}
RUN pecl install ${PECL_STRING} &&\
 echo '' | pecl install pecl_http-"${PHP_EXTENSION_HTTP_VERSION}" &&\
 echo "zend_extension=/usr/local/php${PHP_MAJOR_VERSION}/lib/php/extensions/no-debug-non-zts-20131226/xdebug.so" > ${PHP_CONF} &&\
 for i in ${PECL_STRING}; do\
 ln -sf\
 /usr/local/php${PHP_MAJOR_VERSION}/etc/conf.d/${i}.ini\
 /usr/local/php${PHP_MAJOR_VERSION}/etc/cli/conf.d/${i}.ini;\
 done &&\
 cat /tmp/PHP_DEV_PACKAGES | xargs apt-get remove --purge -y &&\
 apt-get clean &&\
 echo '<?php phpinfo(); ?>' > /var/www/test.php

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --version 1.2.1 --install-dir=/usr/local/bin --filename=composer &&\
 echo "export PATH=~/.composer/vendor/bin:\$PATH" > /etc/profile.d/global-paths.sh

# Install Drupal console
RUN composer global require drupal/console-en:1.0.0-rc16 drupal/console:1.0.0-rc16 drupal/console-core:1.0.0-rc16
RUN echo "source \"$HOME/.console/console.rc\" 2>/dev/null" > /etc/profile.d/drupal-console.sh
#RUN curl -fsSL -o /usr/local/bin/drupal "https://drupalconsole.com/installer" &&\
# chmod +x /usr/local/bin/drupal

# Install drush
RUN apt-get install --no-install-recommends -y mysql-client openssh-client rsync &&\
 apt-get clean
RUN composer global require webflo/drush-shim
#USER 1000
#RUN curl -fsSL -o /usr/local/bin/drush "https://github.com/drush-ops/drush/releases/download/$DRUSH_VERSION/drush.phar" &&\
# chmod +x /usr/local/bin/drush &&\
# drush core-status -y &&\
# drush init -y

EXPOSE 80 443
WORKDIR /var/www/html

ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["apache2-foreground"]
