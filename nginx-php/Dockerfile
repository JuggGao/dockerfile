FROM alpine:3.11
LABEL maintainer="Gao Peng <jugg.gao@qq.com>"

# ARG HOST_USER
# ENV HOST_USER ${HOST_USER:-"www"}

# ARG HOST_UID
# ENV HOST_UID ${HOST_UID:-1000}

ARG INSTALL_PACKAGES

ARG APK_REPOSITORIES
ENV APK_REPOSITORIES ${APK_REPOSITORIES:-""}

ENV TERM=xterm
# Install packages
RUN echo -e "${APK_REPOSITORIES}" >> /etc/apk/repositories && \
    cat /etc/apk/repositories && \
    apk --update add \
      curl \
      gnu-libiconv \
      php7 \
      php7-curl \
    #   php7-pecl \
      php7-fpm \
      php7-openssl \
      php7-phar \
      php7-zlib \
      php7-mbstring \
      php7-json \
      php7-pdo \
      php7-pdo_dblib \
      php7-pdo_mysql \
      php7-session \
      nginx \
      supervisor $INSTALL_PACKAGES

# addgroup info
#         -g Group id

# adduser info
#         -s SHELL    Login shell
#         -G GRP      Add user to existing group
#         -D      Don't assign a password
#         -u UID      User id

RUN echo "Creating user '${HOST_USER}' with id '${HOST_UID}'"

# RUN addgroup -g $HOST_UID $HOST_USER && adduser -s /bin/sh -D -u $HOST_UID -G $HOST_USER $HOST_USER && \
RUN ln -snf /usr/bin/php7 /usr/bin/php && \
    rm -rf /var/cache/apk/* && \
    mkdir -p /data/app /autostart/ && \
    sed -i "s/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/" /etc/php7/php.ini && \
    sed -i 's/; sys_temp_dir = .*/sys_temp_dir = "\/tmp"/' /etc/php7/php.ini && \
    curl -sS https://getcomposer.org/installer | /usr/bin/php -- --install-dir=/usr/local/bin --filename=composer

# support for parallel installs. Faster composer.
# RUN composer global require hirak/prestissimo

# Configure nginx
COPY ./files/nginx/nginx.conf /etc/nginx/nginx.conf
COPY ./files/nginx/default_server.conf /etc/nginx/conf.d/default.conf

# Configure PHP-FPM
COPY ./files/php/php-fpm.conf /etc/php7/php-fpm.conf
COPY ./files/php/php-fpm.d/www.conf /etc/php7/php-fpm.d/www.conf

# Configure supervisord
COPY ./files/supervisor/supervisord.conf /etc/supervisor/conf.d/
COPY ./files/supervisor/init.d/* /autostart/

# Add application
WORKDIR /data/app

VOLUME /data/app

EXPOSE 80 443 9000
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]