FROM php:7.4-fpm-alpine3.16

LABEL maintainer="JPratama7 <josepratama080@gmail.com>"


COPY --from=mlocati/php-extension-installer:latest /usr/bin/install-php-extensions /usr/local/bin/install-php-ext

ENV php_conf=/usr/local/etc/php-fpm.conf \
    fpm_conf=/usr/local/etc/php-fpm.d/www.conf \
    php_vars=/usr/local/etc/php/conf.d/docker-vars.ini \
    LUAJIT_LIB=/usr/local/lib \
    LUAJIT_INC=/usr/local/include/luajit-2.1 \
    LUA_RESTY_PATH=/usr/local/share/lua/5.1 \
    COMPILE_FLAGS="-O3 -pipe -fmodulo-sched -fmodulo-sched-allow-regmoves -flto -fipa-ra -ftree-loop-linear -floop-block -floop-strip-mine -fsplit-paths -floop-unroll-and-jam -fipa-pta -fgcse-lm -fgcse-sm -flive-range-shrinkage -fsched-pressure -fuse-linker-plugin -ffat-lto-objects -ftree-vectorize -fdata-sections -ffunction-sections -mfpmath=sse -fgraphite-identity -ftree-loop-ivcanon -fivopts  -floop-interchange -floop-nest-optimize -floop-parallelize-all -ftree-loop-distribution -fira-hoist-pressure -fira-loop-pressure -ftree-loop-im"

ARG NGINX_VERSION=1.24.0
ARG LUA_MODULE_VERSION=0.10.25
ARG DEVEL_KIT_MODULE_VERSION=0.3.2
ARG VERSION_PCRE=8.45
ARG PCRE_FOLDER=pcre-${VERSION_PCRE}
ARG VERSION_ZLIB=1.2.13
ARG VERSION_LUA_RUSTY_CORE=0.1.27
ARG VERSION_RUSTY_LRU=0.13
ARG VERSION_LUAJIT=2.1-20230410
ARG SOURCE_FOLDER="/usr/src"

ARG CONFIG="\
    --prefix=/etc/nginx \
    --sbin-path=/usr/sbin/nginx \
    --modules-path=/usr/lib/nginx/modules \
    --conf-path=/etc/nginx/nginx.conf \
    --error-log-path=/var/log/nginx/error.log \
    --http-log-path=/var/log/nginx/access.log \
    --pid-path=/var/run/nginx.pid \
    --lock-path=/var/run/nginx.lock \
    --http-client-body-temp-path=/var/cache/nginx/client_temp \
    --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
    --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
    --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
    --http-scgi-temp-path=/var/cache/nginx/scgi_temp \
    --user=nginx \
    --group=nginx \
    --with-http_ssl_module \
    --with-http_realip_module \
    --with-http_addition_module \
    --with-http_sub_module \
    --with-http_dav_module \
    --with-http_flv_module \
    --with-http_mp4_module \
    --with-http_gunzip_module \
    --with-http_gzip_static_module \
    --with-http_random_index_module \
    --with-http_secure_link_module \
    --with-http_stub_status_module \
    --with-http_auth_request_module \
    --with-http_xslt_module=dynamic \
    --with-http_image_filter_module=dynamic \
#    --with-http_geoip_module=dynamic \
    --with-http_perl_module=dynamic \
    --with-threads \
    --with-stream \
    --with-stream_ssl_module \
    --with-stream_ssl_preread_module \
    --with-stream_realip_module \
#    --with-stream_geoip_module=dynamic \
    --with-http_slice_module \
    --with-mail \
    --with-mail_ssl_module \
    --with-compat \
    --with-file-aio \
    --with-http_v2_module \
    --add-module=$SOURCE_FOLDER/ngx_devel_kit-$DEVEL_KIT_MODULE_VERSION \
    --add-module=/usr/src/lua-nginx-module-$LUA_MODULE_VERSION \
    --with-zlib=/usr/src/zlib-$VERSION_ZLIB \
    --with-pcre=/usr/src/$PCRE_FOLDER \
    --with-pcre-jit \
  "

# resolves #166
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php

RUN apk add --no-cache --virtual .libressl-deps libressl tzdata \ 
  && apk add --no-cache --repository https://dl-cdn.alpinelinux.org/alpine/edge/community gnu-libiconv \ 
  && GPG_KEYS=573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62 \
  && addgroup -S nginx \
  && adduser -D -S -h /var/cache/nginx -s /sbin/nologin -G nginx nginx \ 
  && apk add --no-cache --virtual .build-deps \
      autoconf \
      g++ \
      libc-dev \
      make \
      libtool \
      libressl-dev \
      file \
      automake \
      linux-headers \
      curl \
      gnupg \
      libxslt-dev \
      gd-dev \
      libmaxminddb-dev \
      perl-dev \
      xclip \
  && mkdir -p /usr/src \
  && cd /usr/src \
  && echo "Downloading......" \
  && echo "Download nginx" \
  && curl -fSL http://nginx.org/download/nginx-$NGINX_VERSION.tar.gz -o nginx.tar.gz \
  && curl -fSL http://nginx.org/download/nginx-$NGINX_VERSION.tar.gz.asc  -o nginx.tar.gz.asc \
  && echo "Download nginx devel kit" \
  && curl -fSL https://github.com/simpl/ngx_devel_kit/archive/v$DEVEL_KIT_MODULE_VERSION.tar.gz -o ndk.tar.gz \
  && echo "Download lua nginx module" \
  && curl -fSL https://github.com/openresty/lua-nginx-module/archive/v$LUA_MODULE_VERSION.tar.gz -o lua.tar.gz \
  && echo "Download zlib" \
  && curl -fSL https://github.com/madler/zlib/archive/v$VERSION_ZLIB.tar.gz -o zlib.tar.gz \
  && echo "Download lua resty core" \
  && curl -fSL https://github.com/openresty/lua-resty-core/archive/v$VERSION_LUA_RUSTY_CORE.tar.gz -o rustycore.tar.gz \
  && echo "Download lua resty lrucache" \
  && curl -fSL https://github.com/openresty/lua-resty-lrucache/archive/v$VERSION_RUSTY_LRU.tar.gz -o rustylru.tar.gz \
  && echo "Download pcre" \
  && curl -fSL https://sourceforge.net/projects/pcre/files/pcre/${VERSION_PCRE}/$PCRE_FOLDER.tar.gz/download -o pcre.tar.gz \
  && echo "Download luajit2" \
  && curl -fSL https://github.com/openresty/luajit2/archive/v$VERSION_LUAJIT.tar.gz -o luajit.tar.gz \
#  && curl -fSL https://github.com/leev/ngx_http_geoip2_module/archive/$GEOIP2_MODULE_VERSION.tar.gz -o ngx_http_geoip2_module.tar.gz \
#  && export GNUPGHOME="$(mktemp -d)" \
#  && found=''; \
#  for server in \
#    ha.pool.sks-keyservers.net \
#    hkp://keyserver.ubuntu.com:80 \
#    hkp://p80.pool.sks-keyservers.net:80 \
#    pgp.mit.edu \
#  ; do \
#    echo "Fetching GPG key $GPG_KEYS from $server"; \
#    gpg --keyserver "$server" --keyserver-options timeout=10 --recv-keys "$GPG_KEYS" && found=yes && break; \
#  done; \
#  test -z "$found" && echo >&2 "error: failed to fetch GPG key $GPG_KEYS" && exit 1; \
#  gpg --batch --verify nginx.tar.gz.asc nginx.tar.gz \
  #&& rm -r "$GNUPGHOME" nginx.tar.gz.asc \
  && tar -zxf nginx.tar.gz \
  && tar -zxf ndk.tar.gz \
  && tar -zxf lua.tar.gz \
  && tar -zxf pcre.tar.gz \
  && tar -zxf zlib.tar.gz \
  && tar -zxf rustycore.tar.gz \
  && tar -zxf rustylru.tar.gz \
  && tar -zxf luajit.tar.gz \
  && cd luajit2-$VERSION_LUAJIT \
  && CFLAGS="$COMPILE_FLAGS" make all && make install \
  && cd /usr/src/lua-resty-core-$VERSION_LUA_RUSTY_CORE \
  && LUA_LIB_DIR=$LUA_RESTY_PATH CFLAGS="$COMPILE_FLAGS" make install \
  && cd /usr/src/lua-resty-lrucache-$VERSION_RUSTY_LRU \
  && LUA_LIB_DIR=$LUA_RESTY_PATH CFLAGS="$COMPILE_FLAGS" make install \
  # && cd /usr/src/pcre2-$VERSION_PCRE \
  # && (./autogen.sh || ./configure) \
  && cd /usr/src \
#  && tar -zxC /usr/src -f ngx_http_geoip2_module.tar.gz \
#  && rm nginx.tar.gz ndk.tar.gz lua.tar.gz ngx_http_geoip2_module.tar.gz \ 
  && cd nginx-$NGINX_VERSION \
  && ./configure $CONFIG --with-debug \
            --with-cc-opt="${COMPILE_FLAGS/-O3/}" \
            --with-pcre-opt="$COMPILE_FLAGS" \
            --with-ld-opt="-Wl,-O4,--sort-common,--gc-sections,--as-needed,-z,relro,-z,now" \
  && make -j$(getconf _NPROCESSORS_ONLN) \
  && mv objs/nginx objs/nginx-debug \
  && mv objs/ngx_http_xslt_filter_module.so objs/ngx_http_xslt_filter_module-debug.so \
  && mv objs/ngx_http_image_filter_module.so objs/ngx_http_image_filter_module-debug.so \
#  && mv objs/ngx_http_geoip_module.so objs/ngx_http_geoip_module-debug.so \
  && mv objs/ngx_http_perl_module.so objs/ngx_http_perl_module-debug.so \
#  && mv objs/ngx_stream_geoip_module.so objs/ngx_stream_geoip_module-debug.so \
  && ./configure $CONFIG \
            --with-cc-opt="$COMPILE_FLAGS"\
            --with-ld-opt="-Wl,-O4,--sort-common,--gc-sections,--as-needed,-z,relro,-z,now" \
            --with-pcre-opt="$COMPILE_FLAGS"\
  && make -j$(getconf _NPROCESSORS_ONLN) \
  && make install \
  && rm -rf /etc/nginx/html/ \
  && mkdir /etc/nginx/conf.d/ \
  && mkdir -p /usr/share/nginx/html/ \
  && install -m644 html/index.html /usr/share/nginx/html/ \
  && install -m644 html/50x.html /usr/share/nginx/html/ \
  && install -m755 objs/nginx-debug /usr/sbin/nginx-debug \
  && install -m755 objs/ngx_http_xslt_filter_module-debug.so /usr/lib/nginx/modules/ngx_http_xslt_filter_module-debug.so \
  && install -m755 objs/ngx_http_image_filter_module-debug.so /usr/lib/nginx/modules/ngx_http_image_filter_module-debug.so \
#  && install -m755 objs/ngx_http_geoip_module-debug.so /usr/lib/nginx/modules/ngx_http_geoip_module-debug.so \
  && install -m755 objs/ngx_http_perl_module-debug.so /usr/lib/nginx/modules/ngx_http_perl_module-debug.so \
#  && install -m755 objs/ngx_stream_geoip_module-debug.so /usr/lib/nginx/modules/ngx_stream_geoip_module-debug.so \
  && ln -s ../../usr/lib/nginx/modules /etc/nginx/modules \
  && strip /usr/sbin/nginx* \
  && strip /usr/lib/nginx/modules/*.so \
  && rm -rf /usr/src/nginx-$NGINX_VERSION \
#  && rm -rf /usr/src/ngx_http_geoip2_module-$GEOIP2_MODULE_VERSION \
  \
  # Bring in gettext so we can get `envsubst`, then throw
  # the rest away. To do this, we need to install `gettext`
  # then move `envsubst` out of the way so `gettext` can
  # be deleted completely, then move `envsubst` back.
  && apk add --no-cache --virtual .gettext gettext \
  && mv /usr/bin/envsubst /tmp/ \
  \
  && runDeps="$( \
    scanelf --needed --nobanner /usr/sbin/nginx /usr/lib/nginx/modules/*.so /tmp/envsubst \
      | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
      | sort -u \
      | xargs -r apk info --installed \
      | sort -u \
  )" \
  && apk add --no-cache --virtual .nginx-rundeps $runDeps \
  && apk del .build-deps \
  && apk del .gettext \
  && mv /tmp/envsubst /usr/local/bin/ \
  \
  # forward request and error logs to docker log collector
  && ln -sf /dev/stdout /var/log/nginx/access.log \
  && ln -sf /dev/stderr /var/log/nginx/error.log \
  && mkdir /var/log/php/ \
  && ln -sf /dev/stderr /var/log/php/error.log

RUN echo @testing https://dl-cdn.alpinelinux.org/alpine/edge/testing >> /etc/apk/repositories && \
    echo /etc/apk/respositories && \
    apk update && apk upgrade &&\
    apk add --no-cache \
    bash \
    openssh-client \
    wget \
    supervisor \
    curl \
    libcurl \
    libpq \
    git \
    python3 \
    py3-pip \
    ca-certificates \
    dialog \
    autoconf \
    make \
    libressl-dev \
    libzip-dev \
    bzip2-dev \
    icu-dev \
    g++ && \
    apk add --no-cache --virtual .sys-deps \
    musl-dev \
    linux-headers \
    augeas-dev \
    libmcrypt-dev \
    libpng-dev \
    libxslt-dev \
    python3-dev \
    libffi-dev \
    freetype-dev \
    sqlite-dev \
    imap-dev \
    libjpeg-turbo-dev \
    postgresql-dev && \
    docker-php-ext-configure gd \
      --enable-gd \
      --with-freetype \
      --with-jpeg && \
    docker-php-ext-install gd && \
     pip install --upgrade pip && \
    #curl iconv session
    #docker-php-ext-install pdo_mysql pdo_sqlite mysqli mcrypt gd exif intl xsl json soap dom zip opcache && \
    # docker-php-ext-install iconv pdo_mysql pdo_sqlite pgsql pdo_pgsql mysqli gd exif intl xsl json soap dom zip opcache && \
    docker-php-ext-install pdo_mysql mysqli pdo_sqlite pgsql pdo_pgsql exif intl xsl soap zip && \
    pecl install xdebug-3.1.4 && \
    pecl install -o -f redis && \
    echo "extension=redis.so" > /usr/local/etc/php/conf.d/redis.ini && \
    docker-php-source delete && \
    mkdir -p /etc/nginx && \
    mkdir -p /var/www/app && \
    mkdir -p /run/nginx && \
    mkdir -p /var/log/supervisor && \
    php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
    php composer-setup.php --quiet --install-dir=/usr/bin --filename=composer && \
    rm composer-setup.php &&\
  #  pip3 install -U pip && \
    pip3 install -U certbot && \
    mkdir -p /etc/letsencrypt/webrootauth && \
    apk del g++ musl-dev linux-headers libffi-dev augeas-dev python3-dev make autoconf && \
    apk del .sys-deps
#    ln -s /usr/bin/php7 /usr/bin/php

ADD 7.4/conf/supervisord.conf /etc/supervisord.conf

# Copy our nginx config
RUN rm -Rf /etc/nginx/nginx.conf
ADD 7.4/conf/nginx.conf /etc/nginx/nginx.conf

# nginx site conf
RUN mkdir -p /etc/nginx/sites-available/ \
    && mkdir -p /etc/nginx/sites-enabled/ \
    && mkdir -p /etc/nginx/ssl/ \
    && rm -Rf /var/www/* \
    && mkdir /var/www/html/

ADD 7.4/conf/nginx-site.conf /etc/nginx/sites-available/default.conf
ADD 7.4/conf/nginx-site-ssl.conf /etc/nginx/sites-available/default-ssl.conf
RUN ln -s /etc/nginx/sites-available/default.conf /etc/nginx/sites-enabled/default.conf

# tweak php-fpm config
RUN echo "cgi.fix_pathinfo=0" > ${php_vars} &&\
    echo "upload_max_filesize = 100M"  >> ${php_vars} &&\
    echo "post_max_size = 100M"  >> ${php_vars} &&\
    echo "variables_order = \"EGPCS\""  >> ${php_vars} && \
    echo "memory_limit = 128M"  >> ${php_vars} && \
    echo "error_log = /var/log/php/error.log" && \
    sed -i \
        -e "s/;catch_workers_output\s*=\s*yes/catch_workers_output = yes/g" \
        -e "s/pm.max_children = 5/pm.max_children = 4/g" \
        -e "s/pm.start_servers = 2/pm.start_servers = 3/g" \
        -e "s/pm.min_spare_servers = 1/pm.min_spare_servers = 2/g" \
        -e "s/pm.max_spare_servers = 3/pm.max_spare_servers = 4/g" \
        -e "s/;pm.max_requests = 500/pm.max_requests = 200/g" \
        -e "s/user = www-data/user = nginx/g" \
        -e "s/group = www-data/group = nginx/g" \
        -e "s/;listen.mode = 0660/listen.mode = 0666/g" \
        -e "s/;listen.owner = www-data/listen.owner = nginx/g" \
        -e "s/;listen.group = www-data/listen.group = nginx/g" \
        -e "s/listen = 127.0.0.1:9000/listen = \/var\/run\/php-fpm.sock/g" \
        -e "s/^;clear_env = no$/clear_env = no/" \
        ${fpm_conf}
#    ln -s /etc/php7/php.ini /etc/php7/conf.d/php.ini && \

RUN cp /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini \
    && sed -i \
	    -e "s/;opcache/opcache/g" \
	    -e "s/;zend_extension=opcache/zend_extension=opcache/g" \
            /usr/local/etc/php/php.ini \
#    && find /usr/src -type f -not \( -name 'php.tar.xz' \) -delete \
    && cd /usr/src && rm -r `ls | grep -Ev 'php'` \
    && rm -rf /var/cache/apk/*


# Add Scripts
ADD 7.4/scripts/start.sh /start.sh
ADD 7.4/scripts/pull /usr/bin/pull
ADD 7.4/scripts/push /usr/bin/push
ADD 7.4/scripts/letsencrypt-setup /usr/bin/letsencrypt-setup
ADD 7.4/scripts/letsencrypt-renew /usr/bin/letsencrypt-renew
RUN chmod 755 /usr/bin/pull && chmod 755 /usr/bin/push && chmod 755 /usr/bin/letsencrypt-setup && chmod 755 /usr/bin/letsencrypt-renew && chmod 755 /start.sh

# copy in code
ADD 7.4/src/ /var/www/html/
ADD 7.4/errors/ /var/www/errors


EXPOSE 443 80

WORKDIR "/var/www/html"
CMD ["/start.sh"]
