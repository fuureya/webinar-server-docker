# =========================
# Stage 1: Build Vue Assets with Composer Dependencies
# =========================
FROM dunglas/frankenphp:latest-php8.3 AS builder

# Install PHP extensions needed for composer install
RUN install-php-extensions \
    pdo_mysql \
    pdo_pgsql \
    mbstring \
    zip \
    gd \
    opcache \
    intl \
    exif \
    pcntl \
    bcmath

# Install Composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Install Node.js 22.x from NodeSource
RUN apt-get update && apt-get install -y ca-certificates curl gnupg && \
    mkdir -p /etc/apt/keyrings && \
    curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg && \
    echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_22.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list && \
    apt-get update && \
    apt-get install -y nodejs && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /build

# Copy composer files and install PHP dependencies
COPY composer.json composer.lock ./
RUN composer install --prefer-dist --no-scripts --no-autoloader

# Copy all application files
COPY . ./

# Create required directories and set permissions
RUN mkdir -p bootstrap/cache storage/framework/cache storage/framework/sessions storage/framework/views && \
    chmod -R 775 bootstrap/cache storage

# Generate optimized autoloader
RUN composer dump-autoload --optimize

# Install npm dependencies and build assets
RUN npm ci --prefer-offline --no-audit && npm run build


# =========================
# Stage 2: Final Production Image
# =========================
FROM dunglas/frankenphp:latest-php8.3

# Install PHP extensions
RUN install-php-extensions \
    pdo_mysql \
    pdo_pgsql \
    mbstring \
    zip \
    gd \
    opcache \
    intl \
    exif \
    pcntl \
    bcmath

WORKDIR /app

# Copy application files and dependencies from builder
COPY --from=builder /build ./

# Set permissions
RUN chown -R www-data:www-data /app/storage /app/bootstrap/cache && \
    chmod -R 775 /app/storage /app/bootstrap/cache

# Expose port
EXPOSE 8000

# Start FrankenPHP
CMD ["frankenphp", "run", "--config", "/etc/caddy/Caddyfile"]