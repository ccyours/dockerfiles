FROM ubuntu:18.04

LABEL MAINTAINER='https://github.com/ccyours'  \
email='c.hang.chun@163.com' \
description='php env'

ADD sources.list /etc/apt/sources.list
RUN rm -rf /etc/apt/sources.list.d/*

ENV TZ Asia/Shanghai
ENV TERM linux
ENV LANG en_US.utf8

# Update SourceList
RUN apt-get update && \
    apt-get -y install software-properties-common && \
    add-apt-repository ppa:ondrej/php && \
    apt-get update

# Install Base Env
RUN apt-get -y install wget curl telnet supervisor vim && \
    apt-get -y --allow-unauthenticated install php7.1-fpm php7.1-cli php7.1-gd php7.1-mbstring php7.1-mysql php7.1-sqlite3 php7.1-curl php7.1-ldap php7.1-zip php7.1-soap && \
    apt-get -y --allow-unauthenticated install build-essential php-msgpack php-zmq php-redis php7.1-dev php7.1-bcmath php7.1-mcrypt php7.1-xml && \
    echo "error_log = /var/log/php7/cgi.log" >> /etc/php/7.1/fpm/php.ini && \
    echo "post_max_size = 16M" >> /etc/php/7.1/fpm/php.ini && \
    echo "upload_max_filesize = 16M" >> /etc/php/7.1/fpm/php.ini && \
    echo "error_log = /var/log/php7/cli.log" >> /etc/php/7.1/cli/php.ini

# Install PHP7-oci8
ADD oracle-instantclient12.2-basiclite-12.2.0.1.0-1.x86_64.rpm /tmp/basiclite.rpm
ADD oracle-instantclient12.2-devel-12.2.0.1.0-1.x86_64.rpm /tmp/devel.rpm
RUN apt-get install -yq libaio1 alien libaio-dev php7.1-dev php-pear make build-essential && \
    alien -i /tmp/basiclite.rpm /tmp/devel.rpm && \
    pecl install oci8  && \
    PHP_VERSION_ID=$(echo "<?= PHP_MAJOR_VERSION.'.'.PHP_MINOR_VERSION ?>"|php) && \
    echo 'extension=oci8.so' > /etc/php/$PHP_VERSION_ID/mods-available/oci8.ini && \
    phpenmod oci8 && \
    echo '/usr/lib/oracle/12.2/client64/lib' > /etc/ld.so.conf.d/oracle.conf && \
    dpkg -r oracle-instantclient12.2-devel

# Install PHP7 yaml &&
RUN apt-get -y install libyaml-dev && \
    printf '\n' | pecl install yaml-2.0.0 && \
    echo "extension=yaml.so" > /etc/php/7.1/mods-available/yaml.ini && \
    phpenmod yaml && \
    rm -rf /tmp/yaml-2.0.0.tgz && \
    apt-get -y install liblua5.2-dev && \
    ln -s /usr/include/lua5.2 /usr/include/lua && \
    cp /usr/lib/x86_64-linux-gnu/liblua5.2.a /usr/lib/liblua.a && \
    cp /usr/lib/x86_64-linux-gnu/liblua5.2.so /usr/lib/liblua.so && \
    printf '\n' | pecl install lua && \
    echo "extension=lua.so" > /etc/php/7.1/mods-available/lua.ini && \
    phpenmod lua

# Swoole
RUN pecl install swoole-4.3.6 && \
    echo "extension=swoole.so" > /etc/php/7.1/mods-available/swoole.ini && \
    phpenmod swoole

#Composer
ADD composer.phar /tmp/composer.phar
RUN mkdir -p /usr/local/bin && mv /tmp/composer.phar /usr/local/bin/composer && \
    chmod +x /usr/local/bin/composer && \
    echo 'export COMPOSER_HOME="/usr/local/share/composer"' > /etc/profile.d/composer.sh && \
    echo 'export PATH="/usr/local/share/composer/vendor/bin:$PATH"' >> /etc/profile.d/composer.sh && \
    rm -rf /var/lib/apt/lists/*

# Install Composer
ENV COMPOSER_PROCESS_TIMEOUT 40000
ENV COMPOSER_HOME /usr/local/share/composer
ADD config.json $COMPOSER_HOME/config.json

RUN mkdir -p /var/log/php7 && \
    mkdir -p /run/php && \
    touch /var/log/php7/cgi.log && \
    touch /var/log/php7/cli.log && \
    chown -R www-data:www-data /var/log/php7 && \
    rm -rf /tmp/*.rpm && \
    rm -rf /tmp/pear

ADD www.conf /etc/php/7.1/fpm/pool.d/www.conf
ADD supervisor.php7-fpm.conf /etc/supervisor/conf.d/php7-fpm.conf

EXPOSE 80

CMD ["/usr/bin/supervisord", "--nodaemon", "-c", "/etc/supervisor/supervisord.conf"]
