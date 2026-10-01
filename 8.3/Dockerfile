FROM docker.io/library/php:8.3-apache AS production
USER root
RUN set -eux; \
    if command -v a2enmod; then \
    a2enmod rewrite expires; \
    fi; \
    savedAptMark="$(apt-mark showmanual)"; \
    apt-get update; \
    apt-get upgrade -y; \
    # Installing Just Build dependencies
    apt-get install -y --no-install-recommends \
    git \
    libavif-dev \
    libfreetype6-dev \
    libicu-dev \
    libjpeg-dev \
    libmagickwand-dev \
    libmemcached-dev \
    libpng-dev \
    libpq-dev \
    libtidy-dev \
    libwebp-dev \
    libxpm-dev \
    libzip-dev \
    ; \
    yes ' ' | pecl install \
    imagick-3.8.1 \
    memcached-3.4.0 \
    ; \
    docker-php-ext-configure gd \
    --with-avif \
    --with-freetype \
    --with-jpeg \
    --with-webp \
    --with-xpm \
    ; \
    docker-php-ext-enable \
    imagick \
    memcached \
    ; \
    docker-php-ext-install -j "$(nproc)" \
    exif \
    gd \
    intl \
    pdo_mysql \
    pdo_pgsql \
    tidy \
    zip \
    ; \
    apt-mark auto '.*' > /dev/null; \
    apt-mark manual $savedAptMark; \
    # Installing packages we want to keep
    apt-get install -y --no-install-recommends \
    git \
    imagemagick \
    mariadb-client \
    unzip \
    vim \
    ; \
    ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
    | awk '/=>/ { so = $(NF-1); if (index(so, "/usr/local/") == 1) { next }; gsub("^/(usr/)?", "", so); print so }' \
    | sort -u \
    | xargs -r dpkg-query -S \
    | cut -d: -f1 \
    | sort -u \
    | xargs -rt apt-mark manual; \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
    rm -rf /var/lib/apt/lists/*
COPY conf.d/opcache-recommended-production.ini /usr/local/etc/php/conf.d/opcache-recommended.ini
COPY conf.d/docker-php.ini /usr/local/etc/php/conf.d/docker-php.ini
RUN sed -i "s/\/var\/www\/html/\/var\/www\/html\/docroot/" /etc/apache2/sites-available/000-default.conf; \
    chown www-data:www-data /var/www;
COPY --from=docker.io/library/composer /usr/bin/composer /usr/bin/composer
ADD --chown=www-data:www-data https://github.com/gruvbox-community/gruvbox.git /var/www/.vim/pack/default/start/gruvbox
COPY --chown=www-data:www-data vimrc /var/www/.vimrc
COPY --chown=www-data:www-data my.cnf /var/www/.my.cnf
WORKDIR /var/www/html
CMD [ "apache2-foreground" ]

FROM production AS development
USER root
RUN set -eux; \
    apt-get update; \
    apt-get upgrade -y; \
    apt-get install -y \
    bash-completion \
    iproute2 \
    iputils-ping \
    nano \
    net-tools \
    wget \
    ; \
    yes ' ' | pecl install xdebug; \
    docker-php-ext-enable xdebug; \
    apt-get clean;
COPY conf.d/opcache-recommended-development.ini /usr/local/etc/php/conf.d/opcache-recommended.ini
COPY conf.d/xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini
ADD --chmod=+rx https://github.com/mailhog/mhsendmail/releases/download/v0.2.0/mhsendmail_linux_amd64 /usr/local/bin/mhsendmail
USER www-data
RUN set -eux; \
    cp /etc/skel/.bashrc ${HOME}/;
WORKDIR /var/www/html
CMD [ "apache2-foreground" ]
