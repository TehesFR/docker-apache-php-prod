# Pull base image.
FROM php:7.1.8-apache
COPY config/php.ini /usr/local/etc/php/

RUN apt-get clean && apt-get update && apt-get install --fix-missing wget apt-transport-https lsb-release ca-certificates -y
RUN echo "deb http://packages.dotdeb.org jessie all" >> /etc/apt/sources.list
RUN echo "deb-src http://packages.dotdeb.org jessie all" >> /etc/apt/sources.list
RUN cd /tmp && wget https://www.dotdeb.org/dotdeb.gpg && apt-key add dotdeb.gpg
RUN wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
RUN echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list

RUN apt-get clean && apt-get update && apt-get install --fix-missing -y \
  ruby-dev \
  rubygems \
  imagemagick \
  graphviz \
  sudo \
  git \
  vim \
  php7.1-dev \
  memcached \
  libmemcached-tools \
  libmemcached-dev \
  libpng12-dev \
  libjpeg62-turbo-dev \
  libmcrypt-dev \
  libxml2-dev \
  libxslt1-dev \
  mysql-client \
  php7.1-mysql \
  zip \
  wget \
  linux-libc-dev \
  libyaml-dev \
  zlib1g-dev \
  libicu-dev \
  libpq-dev \
  bash-completion \
  htop

# postgresql-client-9.5
RUN wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && echo "deb http://apt.postgresql.org/pub/repos/apt/ jessie-pgdg main" >> /etc/apt/sources.list && apt-get update && apt-get install -y postgresql-client-9.5

# Install memcached for PHP 7
RUN cd /tmp && git clone https://github.com/php-memcached-dev/php-memcached.git
RUN cd /tmp/php-memcached && sudo git checkout php7 && phpize && ./configure --disable-memcached-sasl && make && make install
RUN touch /usr/local/etc/php/conf.d/memcached.ini &&\
 echo "extension=/usr/local/lib/php/extensions/no-debug-non-zts-20160303/memcached.so" >> /usr/local/etc/php/conf.d/memcached.ini

COPY docker-php-ext-install /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-php-ext-install
RUN docker-php-ext-configure gd --with-jpeg-dir=/usr/include/ \
&& docker-php-ext-install \
  gd \
  mbstring \
  mcrypt \
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

# Create new web user for apache and grant sudo without password
RUN useradd web -d /var/www -g www-data -s /bin/bash
RUN usermod -aG sudo web
RUN echo 'web ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# Install YAML extension
RUN pecl install yaml-2.0.2 && echo "extension=yaml.so" > /usr/local/etc/php/conf.d/ext-yaml.ini

# Install APCu extension
RUN pecl install apcu-5.1.8

COPY core/memcached.conf /etc/memcached.conf

# SASS and Compass installation
#RUN gem install compass

# Installation node.js
RUN curl -sL https://deb.nodesource.com/setup_8.x | bash - && \
	apt-get update && apt-get install -y nodejs && \
	npm install npm@latest -g

# Installation of LESS
#RUN npm install -g less && npm install -g less-plugin-clean-css

# Installation of Grunt
#RUN npm install -g grunt-cli

# Installation of Gulp
#RUN npm install -g gulp

# Installation of Bower
RUN npm install -g bower

# Installation of Composer
RUN cd /usr/src && curl -sS http://getcomposer.org/installer | php
RUN cd /usr/src && mv composer.phar /usr/bin/composer

# Installation of drush
RUN git clone https://github.com/drush-ops/drush.git /usr/local/src/drush
RUN cd /usr/local/src/drush && git checkout 8.1.13
RUN ln -s /usr/local/src/drush/drush /usr/bin/drush
RUN cd /usr/local/src/drush && composer update && composer install

# Apache2 config
COPY config/apache2.conf /etc/apache2
COPY core/envvars /etc/apache2
COPY core/other-vhosts-access-log.conf /etc/apache2/conf-enabled/
RUN rm /etc/apache2/sites-enabled/000-default.conf

# Installation of APCu cache
RUN ( \
  echo "extension=apcu.so"; \
  echo "apc.enabled=1"; \
  ) > /usr/local/etc/php/conf.d/ext-apcu.ini

# Installation of Opcode cache
RUN ( \
  echo "opcache.memory_consumption=128"; \
  echo "opcache.interned_strings_buffer=8"; \
  echo "opcache.max_accelerated_files=20000"; \
  echo "opcache.revalidate_freq=5"; \
  echo "opcache.fast_shutdown=1"; \
  echo "opcache.enable_cli=1"; \
  ) > /usr/local/etc/php/conf.d/opcache-recommended.ini

RUN rm -rf /var/www/html && \
  mkdir -p /var/lock/apache2 /var/run/apache2 /var/log/apache2 /var/www/html && \
  chown -R web:www-data /var/lock/apache2 /var/run/apache2 /var/log/apache2 /var/www/html
RUN a2enmod rewrite expires ssl && service apache2 restart

# installation of ssmtp
RUN DEBIAN_FRONTEND=noninteractive apt-get install --fix-missing -y ssmtp && rm -r /var/lib/apt/lists/*
ADD core/ssmtp.conf /etc/ssmtp/ssmtp.conf
ADD core/php-smtp.ini /usr/local/etc/php/conf.d/php-smtp.ini

# Install Drupal Console for Drupal 8
RUN curl https://drupalconsole.com/installer -L -o drupal.phar && mv drupal.phar /usr/local/bin/drupal && chmod +x /usr/local/bin/drupal

# Install WKHTMLTOPDF
RUN apt-get update && apt-get remove -y libqt4-dev qt4-dev-tools wkhtmltopdf && apt-get autoremove -y
RUN apt-get install openssl build-essential libssl-dev libxrender-dev git-core libx11-dev libxext-dev libfontconfig1-dev libfreetype6-dev fontconfig -y
RUN mkdir /var/wkhtmltopdf
RUN cd /var/wkhtmltopdf && wget https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.4/wkhtmltox-0.12.4_linux-generic-amd64.tar.xz && tar xf wkhtmltox-0.12.4_linux-generic-amd64.tar.xz
RUN cp /var/wkhtmltopdf/wkhtmltox/bin/wkhtmltopdf /bin/wkhtmltopdf && cp /var/wkhtmltopdf/wkhtmltox/bin/wkhtmltoimage /bin/wkhtmltoimage
RUN chown -R web:www-data /var/wkhtmltopdf
RUN chmod +x /bin/wkhtmltopdf && chmod +x /bin/wkhtmltoimage

# Our apache volume
VOLUME /var/www/html

# create directory for ssh keys
RUN mkdir /var/www/.ssh/
RUN chown -R web:www-data /var/www/

# Set timezone to Europe/Paris
RUN echo "Europe/Paris" > /etc/timezone && dpkg-reconfigure -f noninteractive tzdata

# Expose 80,443 for apache
EXPOSE 80 443

# Add web and root .bashrc config
# When you "docker exec -it" into the container, you will be switched as web user and placed in /var/www/html
RUN echo "exec su - web" > /root/.bashrc && \
    echo ". .profile" > /var/www/.bashrc && \
    echo "alias ll='ls -al'" > /var/www/.profile && \
    echo "cd /var/www/html" >> /var/www/.profile

# Install symfony autocomplete for web user
RUN sudo -u web composer global require bamarni/symfony-console-autocomplete
RUN echo "export PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/var/www/.composer/vendor/bamarni/symfony-console-autocomplete/" >> /var/www/.profile
RUN echo 'eval "$(symfony-autocomplete)"' >> /var/www/.profile

# Set and run a custom entrypoint
COPY core/docker-entrypoint.sh /
RUN chmod 777 /docker-entrypoint.sh && chmod +x /docker-entrypoint.sh
ENTRYPOINT ["/docker-entrypoint.sh"]
