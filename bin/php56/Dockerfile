FROM php:5.6-apache

RUN apt-get -y update && apt-get upgrade -y

# Install tools && libraries
RUN apt-get -y install --fix-missing apt-utils nano wget dialog \
    build-essential git curl libcurl3 libcurl3-dev zip \
    libmcrypt-dev libsqlite3-dev libsqlite3-0 mysql-client \
    zlib1g-dev libicu-dev libfreetype6-dev libjpeg62-turbo-dev libpng-dev \
    libldap2-dev libxml2-dev \
    && rm -rf /var/lib/apt/lists/*

# Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# PHP5 Extensions
RUN docker-php-ext-install curl \
    && docker-php-ext-install tokenizer \
    && docker-php-ext-install json \
    && docker-php-ext-install mcrypt \
    && docker-php-ext-install pdo_mysql \
    && docker-php-ext-install pdo_sqlite \
    && docker-php-ext-install mysqli \
    && docker-php-ext-install zip \
    && docker-php-ext-install -j$(nproc) intl \
    && docker-php-ext-install mbstring \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ \
    && docker-php-ext-install ldap \
    && docker-php-ext-configure soap --enable-soap \
    && docker-php-ext-install soap

# Oracle (oci8 / pdo_oci)
# credits https://github.com/tassoevan/pdo-oci-extension
#  1) Oracle InstantClient
RUN curl -L -o /tmp/instantclient-sdk-12.2.zip http://bit.ly/2Bab3NM \
    && curl -L -o /tmp/instantclient-basic-12.2.zip http://bit.ly/2mBFHdA \
    && ln -s /usr/include/php5 /usr/include/php \
    && mkdir -p /opt/oracle/instantclient \
    && unzip -q /tmp/instantclient-basic-12.2.zip -d /opt/oracle \
    && mv /opt/oracle/instantclient_12_2 /opt/oracle/instantclient/lib \
    && unzip -q /tmp/instantclient-sdk-12.2.zip -d /opt/oracle \
    && mv /opt/oracle/instantclient_12_2/sdk/include /opt/oracle/instantclient/include \
    && ln -s /opt/oracle/instantclient/lib/libclntsh.so.12.1 /opt/oracle/instantclient/lib/libclntsh.so \
    && ln -s /opt/oracle/instantclient/lib/libocci.so.12.1 /opt/oracle/instantclient/lib/libocci.so \
    && echo /opt/oracle/instantclient/lib >> /etc/ld.so.conf \
    && ldconfig
#  2) OCI8
RUN echo 'instantclient,/opt/oracle/instantclient/lib' | pecl install oci8-2.0.12
RUN echo 'extension=oci8.so' > /usr/local/etc/php/conf.d/20-oci8.ini
#  2) PDO_OCI
RUN pecl channel-update pear.php.net \
    && cd /tmp \
    && pecl download pdo_oci \
    && tar xvf /tmp/PDO_OCI-1.0.tgz -C /tmp \
    && sed 's/function_entry/zend_function_entry/' -i /tmp/PDO_OCI-1.0/pdo_oci.c \
    && sed 's/10.1/12.1/' -i /tmp/PDO_OCI-1.0/config.m4 \
    && cd /tmp/PDO_OCI-1.0 \
    && phpize \
    && ./configure --with-pdo-oci=/opt/oracle/instantclient \
    && make install
RUN echo 'extension=pdo_oci.so' > /usr/local/etc/php/conf.d/20-pdo_oci.ini
RUN rm -rf /tmp/*

# Enable apache modules
RUN a2enmod rewrite headers

EXPOSE 80

ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
