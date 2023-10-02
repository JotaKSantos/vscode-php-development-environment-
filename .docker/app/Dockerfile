FROM php:8.2.8-fpm

# Arguments defined in docker-compose.yml
ARG user
ARG uid

COPY --from=mlocati/php-extension-installer:latest /usr/bin/install-php-extensions /usr/bin/

RUN apt upgrade -y

RUN apt update && apt install git unzip -y && \
    apt-get clean autoclean; apt-get autoremove --yes; rm -rf /var/lib/{apt,dpkg,cache,log}/

# Install system dependencies
RUN apt-get install -y \
    git \
    curl \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    libzip-dev \
    libssl-dev \
    libxslt-dev \
    zip \
    unzip \
    vim \
    zsh \
    default-mysql-client \
    wget

RUN curl -sL https://deb.nodesource.com/setup_14.x -o nodesource_setup.sh
RUN bash nodesource_setup.sh

RUN apt-get update && apt-get install -y \
    nodejs

RUN curl https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt update && apt install yarn

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# configure, install and enable all php packages
RUN docker-php-ext-configure gd --enable-gd --with-freetype --with-jpeg

# Install PHP extensions
RUN docker-php-ext-install pdo_mysql pdo mbstring exif pcntl bcmath gd zip

RUN cd /usr/local/etc/php/conf.d/ && \
  echo 'upload_max_filesize = 100M' >> /usr/local/etc/php/conf.d/docker-php-limits.ini && \
  echo 'post_max_size = 120M' >> /usr/local/etc/php/conf.d/docker-php-limits.ini && \
  echo 'memory_limit = 1024M' >> /usr/local/etc/php/conf.d/docker-php-limits.ini && \
  echo 'variables_order = "EGPCS"' >> /usr/local/etc/php/conf.d/docker-php-limits.ini && \
  echo 'date.timezone = "America/Sao_Paulo"' >> /usr/local/etc/php/conf.d/docker-php-limits.ini

#ativando o xdebug
RUN install-php-extensions gd xdebug

RUN apt-get update && apt-get install -y libmagickwand-dev --no-install-recommends
RUN pecl install imagick
RUN docker-php-ext-enable imagick

ENV DOCKERIZE_VERSION v0.6.1
RUN wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && tar -C /usr/local/bin -xzvf dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz
    
# Get latest Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Create system user to run Composer and Artisan Commands
RUN useradd -G www-data,root -u $uid -d /home/$user $user
RUN mkdir -p /home/$user/.composer && \
    chown -R $user:$user /home/$user

# ativa o bash como terminal padrão
RUN chsh -s $(which zsh) $user

# Set working directory
WORKDIR /var/www

USER $user

RUN composer global require friendsofphp/php-cs-fixer
RUN composer global require squizlabs/php_codesniffer

RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
RUN git clone https://github.com/denysdovhan/spaceship-prompt.git "/home/$user/.oh-my-zsh/custom/themes/spaceship-prompt" 
RUN ln -s "/home/$user/.oh-my-zsh/custom/themes/spaceship-prompt/spaceship.zsh-theme" "/home/$user/.oh-my-zsh/custom/themes/spaceship.zsh-theme"