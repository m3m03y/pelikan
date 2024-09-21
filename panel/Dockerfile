FROM ubuntu:24.04

# Install PHP, NGINX, and Redis
RUN apt update && apt install -y \
    php8.3 php8.3-gd php8.3-mysql php8.3-mbstring \
    php8.3-bcmath php8.3-xml php8.3-curl php8.3-zip php8.3-intl \
    php8.3-sqlite3 php8.3-fpm nginx redis-server cron unzip curl tar \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Create directories and downloading files
RUN mkdir -p /var/www/pelican
WORKDIR /var/www/pelican

# Download panel files
RUN curl -L https://github.com/pelican-dev/panel/releases/latest/download/panel.tar.gz | tar -xzv -C /var/www/pelican

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
RUN composer install --no-dev --optimize-autoloader

# Remove default configuration
RUN rm /etc/nginx/sites-enabled/default

# Copy and set up NGINX configuration
COPY ./data/pelican.conf /etc/nginx/sites-available/pelican.conf

# Enabling configuration
RUN ln -s /etc/nginx/sites-available/pelican.conf /etc/nginx/sites-enabled/pelican.conf 

RUN service nginx restart

# Panel setup, generate and copy api key
#COPY ./data/generate_api_key.sh /usr/local/bin/generate_api_key.sh
#RUN chmod +x /usr/local/bin/generate_api_key.sh
RUN php artisan p:environment:setup

# Setting permissions
RUN chmod -R 755 storage/* bootstrap/cache/
RUN chown -R www-data:www-data /var/www/pelican

# Crontab Configuration
RUN echo "* * * * * php /var/www/pelican/artisan schedule:run >> /dev/null 2>&1" > /etc/cron.d/pelican-cron \
    && chmod 0644 /etc/cron.d/pelican-cron \
    && crontab -u www-data /etc/cron.d/pelican-cron

RUN service cron start

# Setting up Queue Service
# CMD php artisan p:environment:queue-service 

CMD service cron start && service nginx start && php-fpm8.3 -F \
 & php artisan queue:work --timeout=90 --tries=3
