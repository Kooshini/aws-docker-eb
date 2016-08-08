FROM phusion/baseimage:0.9.18

MAINTAINER Chris Cooke <kooshini@gmail.com>

# Default baseimage settings
ENV HOME /root
RUN /etc/my_init.d/00_regen_ssh_host_keys.sh
CMD ["/sbin/my_init"]
ENV DEBIAN_FRONTEND noninteractive

# Add the PPA for PHP 5.6
RUN add-apt-repository ppa:ondrej/php5-5.6 -y

# Update software list and install php + nginx
RUN apt-get update \
    && apt-get install -y --force-yes \
    nginx \
    php5 \
    php5-fpm \
    php5-cli \
    php5-mysqlnd \
    php5-mcrypt \
    php5-curl \
    php5-gd \
    php5-intl \
    php5-json

# Clear cache
RUN apt-get clean \
	&& rm -rf /var/lib/apt/lists/* \
              /tmp/* \
              /var/tmp/*

# Configure nginx core
RUN rm -Rf /etc/nginx/nginx.conf
ADD build/nginx/nginx.conf /etc/nginx/nginx.conf

# Configure nginx site
RUN mkdir -p /etc/nginx/sites-available/ && \
mkdir -p /etc/nginx/sites-enabled/ && \
mkdir -p /etc/nginx/ssl/ && \
rm -Rf /var/www/* && \
mkdir /var/www/ && \
rm -Rf /etc/nginx/sites-enabled/* && \
rm -Rf /etc/nginx/sites-available/*
ADD build/nginx/nginx-site-laravel5.conf /etc/nginx/sites-available/default.conf
RUN ln -s /etc/nginx/sites-available/default.conf /etc/nginx/sites-enabled/default.conf

# Configure PHP
ADD build/php/php.ini /etc/php5/fpm/php.ini
ADD build/php/php.ini /etc/php5/cli/php.ini
ADD build/php/www.conf /etc/php5/fpm/pool.d/www.conf
ADD build/php/php-fpm.conf /etc/php5/fpm/php-fpm.conf
# Enable Mcrypt because of bug
RUN php5enmod mcrypt

# Add nginx service
RUN mkdir /etc/service/nginx
ADD build/nginx/run.sh /etc/service/nginx/run
RUN chmod +x /etc/service/nginx/run

# Add PHP service
RUN mkdir /etc/service/phpfpm
ADD build/php/run.sh /etc/service/phpfpm/run
RUN chmod +x /etc/service/phpfpm/run

# Add the application code into the container
ADD . /var/www

# Set appropriate permissions for Laravel 5x
RUN chown -R www-data /var/www/storage && \
chmod -R 0700 /var/www/storage && \
chown -R www-data /var/www/bootstrap/cache && \
chmod -R 0700 /var/www/bootstrap/cache && \
chmod 0444 /var/www/config/*.php

# Ensure that Laravel debug is not on for security reasons
RUN sed -i "s/'debug' => .*/'debug' => false,/" /var/www/config/app.php

# Add nginx volumes
VOLUME ["/var/www", "/etc/nginx/sites-available", "/etc/nginx/sites-enabled"]

# Set the work directory
WORKDIR /var/www

# Expose ports
EXPOSE 80 443
