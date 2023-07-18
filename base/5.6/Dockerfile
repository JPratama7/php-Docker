FROM debian:bookworm-slim

LABEL maintainer="Colin Wilson colin@wyveo.com"

# Let the container know that there is no tty
ENV DEBIAN_FRONTEND noninteractive
ENV PHP_VER 5.6
ENV php_conf /etc/php/${PHP_VER}/fpm/php.ini
ENV fpm_conf /etc/php/${PHP_VER}/fpm/pool.d/www.conf
ENV COMPOSER_VERSION 2.5.8

ARG BUILD_DEPS='curl gcc make autoconf libc-dev zlib1g-dev pkg-config gnupg2 dirmngr wget'

COPY docker-php-ext-* docker-php-entrypoint /usr/local/bin/

# Install Basic Requirements
RUN set -x \
    && apt-get update \
    && apt-get install --no-install-recommends $BUILD_DEPS --no-install-suggests -q -y apt-transport-https lsb-release ca-certificates \
    && wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg \
    && echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list \
    && apt-get update \
    && apt-get install --no-install-recommends --no-install-suggests -q -y \
            apt-utils \
            bash \
            nano \
            zip \
            unzip \
            git \
            libmemcached-dev \
            libmemcached11 \
            libmagickwand-dev \
            php${PHP_VER}-fpm \
            php${PHP_VER}-cli \
            php${PHP_VER}-bcmath \
            php${PHP_VER}-dev \
            php${PHP_VER}-common \
            php${PHP_VER}-opcache \
            php${PHP_VER}-readline \
            php${PHP_VER}-mbstring \
            php${PHP_VER}-curl \
            php${PHP_VER}-gd \
            php${PHP_VER}-imagick \
            php${PHP_VER}-mysql \
            php${PHP_VER}-zip \
            php${PHP_VER}-pgsql \
            php${PHP_VER}-intl \
            php${PHP_VER}-xml \
            php-pear \
    && pecl -d php_suffix=${PHP_VER} install -o -f redis-4.3.0 memcached-2.2.0 \
    && mkdir -p /run/php \
    && echo "#!/bin/sh\nexit 0" > /usr/sbin/policy-rc.d \
    && sed -i -e "s/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/g" ${php_conf} \
    && sed -i -e "s/memory_limit\s*=\s*.*/memory_limit = 256M/g" ${php_conf} \
    && sed -i -e "s/upload_max_filesize\s*=\s*2M/upload_max_filesize = 100M/g" ${php_conf} \
    && sed -i -e "s/post_max_size\s*=\s*8M/post_max_size = 100M/g" ${php_conf} \
    && sed -i -e "s/variables_order = \"GPCS\"/variables_order = \"EGPCS\"/g" ${php_conf} \
    && sed -i -e "s/;daemonize\s*=\s*yes/daemonize = no/g" /etc/php/${PHP_VER}/fpm/php-fpm.conf \
    && sed -i -e "s/;catch_workers_output\s*=\s*yes/catch_workers_output = yes/g" ${fpm_conf} \
    && sed -i -e "s/pm.max_children = 5/pm.max_children = 4/g" ${fpm_conf} \
    && sed -i -e "s/pm.start_servers = 2/pm.start_servers = 3/g" ${fpm_conf} \
    && sed -i -e "s/pm.min_spare_servers = 1/pm.min_spare_servers = 2/g" ${fpm_conf} \
    && sed -i -e "s/pm.max_spare_servers = 3/pm.max_spare_servers = 4/g" ${fpm_conf} \
    && sed -i -e "s/pm.max_requests = 500/pm.max_requests = 200/g" ${fpm_conf} \
    && sed -i -e "s/^;clear_env = no$/clear_env = no/" ${fpm_conf} \
    && echo "extension=redis.so" > /etc/php/${PHP_VER}/mods-available/redis.ini \
    && echo "extension=memcached.so" > /etc/php/${PHP_VER}/mods-available/memcached.ini \
    && echo "extension=imagick.so" > /etc/php/${PHP_VER}/mods-available/imagick.ini \
    && ln -sf /etc/php/${PHP_VER}/mods-available/redis.ini /etc/php/${PHP_VER}/fpm/conf.d/20-redis.ini \
    && ln -sf /etc/php/${PHP_VER}/mods-available/redis.ini /etc/php/${PHP_VER}/cli/conf.d/20-redis.ini \
    && ln -sf /etc/php/${PHP_VER}/mods-available/memcached.ini /etc/php/${PHP_VER}/fpm/conf.d/20-memcached.ini \
    && ln -sf /etc/php/${PHP_VER}/mods-available/memcached.ini /etc/php/${PHP_VER}/cli/conf.d/20-memcached.ini \
    && ln -sf /etc/php/${PHP_VER}/mods-available/imagick.ini /etc/php/${PHP_VER}/fpm/conf.d/20-imagick.ini \
    && ln -sf /etc/php/${PHP_VER}/mods-available/imagick.ini /etc/php/${PHP_VER}/cli/conf.d/20-imagick.ini \
    # Install Composer
    && curl -o /tmp/composer-setup.php https://getcomposer.org/installer \
    && curl -o /tmp/composer-setup.sig https://composer.github.io/installer.sig \
    && php -r "if (hash('SHA384', file_get_contents('/tmp/composer-setup.php')) !== trim(file_get_contents('/tmp/composer-setup.sig'))) { unlink('/tmp/composer-setup.php'); echo 'Invalid installer' . PHP_EOL; exit(1); }" \
    && php /tmp/composer-setup.php --no-ansi --install-dir=/usr/local/bin --filename=composer --version=${COMPOSER_VERSION} \
    && rm -rf /tmp/composer-setup.php \
    && ln -s /usr/sbin/php-fpm /usr/sbin/php-fpm${PHP_BASE} \
    # Clean up
    && rm -rf /tmp/pear \
    && apt-get purge -y --auto-remove $BUILD_DEPS \
    && apt-get clean \
    && apt-get autoremove \
    && rm -rf /var/lib/apt/lists/*


# Override nginx's default config
EXPOSE 9000

CMD ["php-fpm"]
