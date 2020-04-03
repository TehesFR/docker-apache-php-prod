# Pull base image.
FROM php:7.3.15-apache

# Install tools
RUN apt-get clean && apt-get update && apt-get install --fix-missing wget apt-transport-https lsb-release ca-certificates gnupg2 -y
RUN apt-get clean && apt-get update && apt-get install --fix-missing -y \
  imagemagick \
  graphviz \
  sudo \
  git \
  vim \
  memcached \
  libmemcached-tools \
  libmemcached-dev \
  libpng-dev \
  libjpeg62-turbo-dev \
  libmcrypt-dev \
  libxml2-dev \
  libxslt1-dev \
  default-mysql-client \
  zip \
  wget \
  linux-libc-dev \
  libyaml-dev \
  zlib1g-dev \
  libicu-dev \
  libpq-dev \
  libzip-dev \
  libldap2-dev \
  libssl-dev \
  bash-completion \
  htop \
  telnet \
  dnsutils \
  iputils-ping

RUN pecl install mcrypt-1.0.3 && \
  docker-php-ext-enable mcrypt

RUN chmod +x /usr/local/bin/docker-php-ext-install
RUN docker-php-ext-configure gd --with-jpeg-dir=/usr/include/ \
&& docker-php-ext-install \
  gd \
  mbstring \
  zip \
  soap \
  pdo_mysql \
  mysqli \
  xsl \
  opcache \
  calendar \
  intl \
  exif \
  pgsql \
  pdo_pgsql \
  ftp \
  bcmath

# Install YAML extension
RUN pecl install yaml-2.0.4 && echo "extension=yaml.so" > /usr/local/etc/php/conf.d/ext-yaml.ini

# Install APCu extension
RUN pecl install apcu-5.1.18
# Installation of APCu cache
RUN ( \
  echo "extension=apcu.so"; \
  echo "apc.enabled=1"; \
  ) > /usr/local/etc/php/conf.d/ext-apcu.ini

# Installation node.js
RUN curl -sL https://deb.nodesource.com/setup_12.x | bash - && apt-get update && apt-get install -y nodejs && npm install npm@latest -g

# Installation of Composer
RUN cd /usr/src && curl -sS http://getcomposer.org/installer | php
RUN cd /usr/src && mv composer.phar /usr/bin/composer

# Custom Opcache
RUN ( \
  echo "opcache.memory_consumption=128"; \
  echo "opcache.interned_strings_buffer=8"; \
  echo "opcache.max_accelerated_files=20000"; \
  echo "opcache.revalidate_freq=5"; \
  echo "opcache.fast_shutdown=1"; \
  echo "opcache.enable_cli=1"; \
  ) >> /usr/local/etc/php/conf.d/docker-php-ext-opcache.ini

# Create new web user for apache and grant sudo without password
RUN useradd web -d /var/www -g www-data -s /bin/bash
RUN usermod -aG sudo web
RUN echo 'web ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# Apache bullshit
RUN rm -rf /var/www/html && \
  mkdir -p /var/lock/apache2 /var/run/apache2 /var/log/apache2 /var/www/html && \
  chown -R web:www-data /var/lock/apache2 /var/run/apache2 /var/log/apache2 /var/www/html
RUN a2enmod rewrite expires ssl && service apache2 restart

# install msmtp
RUN set -x \
    && DEBIAN_FRONTEND=noninteractive \
    && apt-get update && apt-get install -y --no-install-recommends msmtp && rm -r /var/lib/apt/lists/*
ADD core/msmtprc.conf /usr/local/etc/msmtprc
ADD core/php-smtp.ini /usr/local/etc/php/conf.d/php-smtp.ini

# Install WKHTMLTOPDF 0.12.5
RUN apt-get update && apt-get remove -y libqt4-dev qt4-dev-tools wkhtmltopdf && apt-get autoremove -y
RUN apt-get update && apt-get install -y libjpeg62-turbo libfontenc1 xfonts-75dpi xfonts-base xfonts-encodings xfonts-utils openssl build-essential libssl-dev libxrender-dev git-core libx11-dev libxext-dev libfontconfig1-dev libfreetype6-dev fontconfig
RUN cd /tmp && wget https://downloads.wkhtmltopdf.org/0.12/0.12.5/wkhtmltox_0.12.5-1.buster_amd64.deb
RUN cd /tmp && dpkg -i wkhtmltox_0.12.5-1.buster_amd64.deb
RUN apt --fix-broken install

# Apache2 config
COPY config/apache2.conf /etc/apache2
COPY core/envvars /etc/apache2
COPY core/other-vhosts-access-log.conf /etc/apache2/conf-enabled/
RUN rm /etc/apache2/sites-enabled/000-default.conf

# Our apache volume
VOLUME /var/www/html

# create directory for ssh keys
RUN mkdir /var/www/.ssh/
RUN chown -R web:www-data /var/www/

# Set timezone to Europe/Paris
RUN ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime

# Expose 80,443 for apache
EXPOSE 80 443

# Add web and root .bashrc config
# When you "docker exec -it" into the container, you will be switched as web user and placed in /var/www/html
RUN echo "exec su - web" > /root/.bashrc && \
    echo ". .profile" > /var/www/.bashrc && \
    echo "alias ll='ls -al'" > /var/www/.profile && \
    echo "cd /var/www/html" >> /var/www/.profile

# Custom php.ini
COPY config/php.ini /usr/local/etc/php/

# Set and run a custom entrypoint
COPY core/docker-entrypoint.sh /
RUN chmod 777 /docker-entrypoint.sh && chmod +x /docker-entrypoint.sh
ENTRYPOINT ["/docker-entrypoint.sh"]









