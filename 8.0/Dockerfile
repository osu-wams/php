FROM docker.io/library/php:8.0-apache AS production
USER root
RUN set -eux; \
  if command -v a2enmod; then \
    a2enmod rewrite; \
  fi; \
  savedAptMark="$(apt-mark showmanual)"; \
  apt-get update; \
  apt-get upgrade -y; \
  # Installing Just Build dependencies
  apt-get install -y --no-install-recommends \
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
    git \
  ; \
  yes ' ' | pecl install \
    imagick-3.8.0 \
    memcached-3.3.0 \
  ; \
  docker-php-ext-configure gd \
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
    intl \
    gd \
    opcache \
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
RUN { \
      echo 'opcache.memory_consumption=128'; \
      echo 'opcache.save_comments=1'; \
      echo 'opcache.interned_strings_buffer=8'; \
      echo 'opcache.max_accelerated_files=4000'; \
      echo 'opcache.revalidate_freq=60'; \
      echo 'opcache.fast_shutdown=1'; \
    } > /usr/local/etc/php/conf.d/opcache-recommended.ini; \
    { \
      echo 'max_execution_time = 300'; \
      echo 'max_input_vars = 1500'; \
      echo 'memory_limit = 256M';\
      echo 'output_buffering=4096'; \
      echo 'post_max_size = 256M'; \
      echo 'upload_max_filesize = 256M'; \
    } > /usr/local/etc/php/conf.d/docker-php-drupal.ini;
RUN sed -i "s/\/var\/www\/html/\/var\/www\/html\/docroot/" /etc/apache2/sites-available/000-default.conf; \
  chown www-data:www-data /var/www;
COPY --from=docker.io/library/composer /usr/bin/composer /usr/bin/composer
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
    vim \
    wget \
  ; \
  yes ' ' | pecl install xdebug; \
  docker-php-ext-enable xdebug; \
  apt-get clean;
RUN { \
    echo 'xdebug.mode=${PHP_XDEBUG_MODE}'; \
    echo 'xdebug.idekey=${PHP_XDEBUG_IDEKEY}'; \
    echo 'xdebug.discover_client_host=true'; \
    echo 'xdebug.client_host=host.docker.internal'; \
  } > /usr/local/etc/php/conf.d/xdebug.ini;
RUN \
    wget https://github.com/mailhog/mhsendmail/releases/download/v0.2.0/mhsendmail_linux_amd64 -O /usr/local/bin/mhsendmail; \
    chmod a+x /usr/local/bin/mhsendmail;
USER www-data
RUN set -eux; \
  git clone https://github.com/gruvbox-community/gruvbox.git ~/.vim/pack/default/start/gruvbox ; \
  { \
    echo 'syntax on'; \
    echo 'set background=dark'; \
    echo 'set colorcolumn=80'; \
    echo 'set expandtab'; \
    echo 'set incsearch'; \
    echo 'set noerrorbells'; \
    echo 'set number'; \
    echo 'set relativenumber'; \
    echo 'set shiftwidth=4'; \
    echo 'set showmatch'; \
    echo 'set smartcase'; \
    echo 'set smartindent'; \
    echo 'set t_Co=256'; \
    echo 'set tabstop=4 softtabstop=4'; \
    echo 'set termguicolors'; \
    echo 'set wildmenu'; \
    echo 'set wildmode=longest,full'; \
    echo 'highlight ColorColumn ctermbg=0 guibg=lightgrey'; \
    echo 'colorscheme gruvbox'; \
    echo 'let g:skip_defaults_vim = 1'; \
  } > ${HOME}/.vimrc; \
  { \
    echo '# MySQL/Mariadb Client configuration'; \
    echo '[client]' ;\
    echo 'skip-ssl = true' ;\
  } > ${HOME}/.my.cnf; \
  cp /etc/skel/.bashrc ${HOME}/;
WORKDIR /var/www/html
CMD [ "apache2-foreground" ]
