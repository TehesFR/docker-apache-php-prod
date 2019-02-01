# Pull base image.
FROM php:7.3.1-apache

# Install tools
RUN apt-get clean && apt-get update && apt-get install --fix-missing -y \
  sudo \
  git \
  vim \
  zip \
  wget \
  htop \
  iputils-ping \
  dnsutils \
  telnet \
  mysql-client

# Create new web user for apache and grant sudo without password
RUN useradd web -d /var/www -g www-data -s /bin/bash
RUN usermod -aG sudo web
RUN echo 'web ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# Apache2 config
COPY config/apache2.conf /etc/apache2
COPY core/envvars /etc/apache2
COPY core/other-vhosts-access-log.conf /etc/apache2/conf-enabled/
RUN rm /etc/apache2/sites-enabled/000-default.conf

# Install php extensions
RUN docker-php-ext-install opcache pdo_mysql

# Custom Opcache
RUN ( \
  echo "opcache.memory_consumption=128"; \
  echo "opcache.interned_strings_buffer=8"; \
  echo "opcache.max_accelerated_files=20000"; \
  echo "opcache.revalidate_freq=5"; \
  echo "opcache.fast_shutdown=1"; \
  echo "opcache.enable_cli=1"; \
  ) >> /usr/local/etc/php/conf.d/docker-php-ext-opcache.ini

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

# Custom php.ini
COPY config/php.ini /usr/local/etc/php/

# Set and run a custom entrypoint
COPY core/docker-entrypoint.sh /
RUN chmod 777 /docker-entrypoint.sh && chmod +x /docker-entrypoint.sh
ENTRYPOINT ["/docker-entrypoint.sh"]
