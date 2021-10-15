FROM drupal:9.2.5-php7.4-apache-buster as base

## Templating binary
COPY --from=hairyhenderson/gomplate:v3.9.0-slim /gomplate /bin/gomplate

## Aditional packages for day-2 operations
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y \
    unzip \
    git \
    vim \
    rsync \
    default-mysql-client \
	libfcgi-bin \
    openssh-client && \
    apt-get clean -y && \
    apt-get autoclean -y && \
    apt-get autoremove -y && \
    rm -rf /var/lib/{apt,dpkg,cache,log}/ && \
    rm -rf /var/lib/apt/lists/*

## Project requirements (civi-crm)
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y libcurl4-openssl-dev libonig-dev libzip-dev libicu-dev libxml2-dev &&\
    docker-php-ext-install bcmath curl mbstring zip intl dom soap mysqli

## Install drush
RUN composer global require drush/drush:10.6 && \
    mv $HOME/.composer /usr/local/share/composer && \
    ln -s /usr/local/share/composer/vendor/drush/drush/drush /usr/local/bin/drush

RUN curl -LsS https://download.civicrm.org/cv/cv.phar -o /usr/local/bin/cv && chmod +x /usr/local/bin/cv

## Set operator user
RUN useradd -ms /bin/bash emergya && mkdir -p /home/emergya/.ssh && chmod 0700 /home/emergya/.ssh

## Set assets
RUN mkdir -p /etc/drush/sites/
COPY assets/drush/drush.yml /etc/drush/drush.yml

COPY assets/etc/apache2/sites-enabled/000-default.conf /etc/apache2/sites-enabled/000-default.conf

COPY assets/ /assets

WORKDIR /var/www/html

## Required apache2 modules
RUN a2enmod headers proxy ssl

EXPOSE 9000

USER emergya

ENTRYPOINT ["/assets/bin/docker-entrypoint.sh"]

CMD ["apache2-foreground"]