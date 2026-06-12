FROM php:8.4-cli

WORKDIR /var/www/html

# Install only the missing extensions for Laravel
RUN apt-get update && apt-get install -y --no-install-recommends \
    libzip-dev \
    libonig-dev \
    && docker-php-ext-install \
    pdo_mysql \
    zip \
    bcmath \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Install Node.js
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    gnupg \
    && curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && apt-get install -y --no-install-recommends nodejs \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Create sail user (matching typical WSL UID 1000)
RUN groupadd -g 1000 sail && \
    useradd -u 1000 -g sail -m -s /bin/bash sail && \
    usermod -aG sudo sail && \
    echo 'sail ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

USER sail

EXPOSE 8000

CMD ["php", "artisan", "serve", "--host=0.0.0.0", "--port=8000"]
