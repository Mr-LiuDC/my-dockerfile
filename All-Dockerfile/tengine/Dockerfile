# Build with: docker build -t tengine .
# Run with: docker run -d -p 80:80 -p 443:443 --name tengine tengine
# nginx: https://git.io/vSIyj

FROM alpine:latest

ENV TENGINE_VERSION 2.3.1

# tengine make configuration
ENV CONFIG "\
        --prefix=/etc/nginx \
        --sbin-path=/usr/sbin/nginx \
        --modules-path=/usr/lib/nginx/modules \
        --conf-path=/etc/nginx/nginx.conf \
        --error-log-path=/var/log/nginx/error.log \
        --http-log-path=/var/log/nginx/access.log \
        --pid-path=/var/run/nginx.pid \
        --lock-path=/var/run/nginx.lock \
        --http-client-body-temp-path=/var/cache/nginx/client_temp \
        --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
        --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
        --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
        --http-scgi-temp-path=/var/cache/nginx/scgi_temp \
        --user=nginx \
        --group=nginx \
        --with-http_ssl_module \
        --with-http_realip_module \
        --with-http_addition_module \
        --with-http_sub_module \
        --with-http_dav_module \
        --with-http_flv_module \
        --with-http_mp4_module \
        --with-http_gunzip_module \
        --with-http_gzip_static_module \
        --with-http_random_index_module \
        --with-http_secure_link_module \
        --with-http_stub_status_module \
        --with-http_auth_request_module \
        --with-http_xslt_module=dynamic \
        --with-http_image_filter_module=dynamic \
        --with-http_geoip_module=dynamic \
        --with-threads \
        --with-stream \
        --with-stream_sni \
        --with-stream_ssl_module \
        --with-stream_ssl_preread_module \
        --with-stream_realip_module \
        --with-stream_geoip_module=dynamic \
        --with-http_slice_module \
        --with-mail \
        --with-mail_ssl_module \
        --with-compat \
        --with-file-aio \
        --with-http_v2_module \
        --add-module=modules/ngx_http_upstream_check_module \
        "

RUN     rm -rf /var/cache/apk/* \
        && rm -rf /tmp/* \
        && addgroup -S nginx \
        && adduser -D -S -h /var/cache/nginx -s /sbin/nologin -G nginx nginx \
# change the apk resource if need
#        && echo "https://mirrors.aliyun.com/alpine/latest-stable/main" > /etc/apk/repositories \
#        && echo "https://mirrors.aliyun.com/alpine/latest-stable/community" >> /etc/apk/repositories \
#
        && apk update upgrade \
# install make dependence
        && apk add --no-cache --virtual .build-deps \
            gcc \
            libc-dev \
            make \
            openssl-dev \
            pcre-dev \
            zlib-dev \
            linux-headers \
            geoip-dev \
            libxslt-dev \
            gd-dev \
# install runtime dependence
#        && apk add --no-cache \
#            openssl \
#            pcre \
#            zlib \
#            gzip \
#            geoip \
#
# install some tools we need
        && apk add --no-cache \
            net-tools \
            procps \
            unzip \
            tar \
            bash \
            tzdata \
            vim \
            curl \
            wget \
        && curl -L "http://tengine.taobao.org/download/tengine-$TENGINE_VERSION.tar.gz" -o tengine.tar.gz \
        && mkdir -p /usr/src \
        && tar -zxC /usr/src -f tengine.tar.gz \
        && rm tengine.tar.gz \
        && cd /usr/src/tengine-$TENGINE_VERSION \
        && ./configure $CONFIG --with-debug \
        && make -j$(getconf _NPROCESSORS_ONLN) \
        && mv objs/nginx objs/nginx-debug \
        && mv objs/ngx_http_xslt_filter_module.so objs/ngx_http_xslt_filter_module-debug.so \
        && mv objs/ngx_http_image_filter_module.so objs/ngx_http_image_filter_module-debug.so \
        && mv objs/ngx_http_geoip_module.so objs/ngx_http_geoip_module-debug.so \
        && mv objs/ngx_stream_geoip_module.so objs/ngx_stream_geoip_module-debug.so \
        && ./configure $CONFIG \
        && make -j$(getconf _NPROCESSORS_ONLN) \
        && make install \
        && rm -rf /etc/nginx/html/ \
        && mkdir /etc/nginx/conf.d/ \
        && mkdir -p /usr/share/nginx/html/ \
        && install -m644 html/index.html /usr/share/nginx/html/ \
        && install -m644 html/50x.html /usr/share/nginx/html/ \
        && install -m755 objs/nginx-debug /usr/sbin/nginx-debug \
        && install -m755 objs/ngx_http_xslt_filter_module-debug.so /usr/lib/nginx/modules/ngx_http_xslt_filter_module-debug.so \
        && install -m755 objs/ngx_http_image_filter_module-debug.so /usr/lib/nginx/modules/ngx_http_image_filter_module-debug.so \
        && install -m755 objs/ngx_http_geoip_module-debug.so /usr/lib/nginx/modules/ngx_http_geoip_module-debug.so \
        && install -m755 objs/ngx_stream_geoip_module-debug.so /usr/lib/nginx/modules/ngx_stream_geoip_module-debug.so \
        && ln -s ../../usr/lib/nginx/modules /etc/nginx/modules \
        && strip /usr/sbin/nginx* \
        && strip /usr/lib/nginx/modules/*.so \
        && rm -rf /usr/src/tengine-$NGINX_VERSION \
        \
        # Bring in gettext so we can get `envsubst`, then throw
        # the rest away. To do this, we need to install `gettext`
        # then move `envsubst` out of the way so `gettext` can
        # be deleted completely, then move `envsubst` back.
        && apk add --no-cache --virtual .gettext gettext \
        && mv /usr/bin/envsubst /tmp/ \
        \
        && runDeps="$( \
        scanelf --needed --nobanner --format '%n#p' /usr/sbin/nginx /usr/lib/nginx/modules/*.so /tmp/envsubst \
        | tr ',' '\n' \
        | sort -u \
        | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
        )" \
        && apk add --no-cache --virtual .nginx-rundeps $runDeps \
        && apk del .build-deps \
        && apk del .gettext \
        && mv /tmp/envsubst /usr/local/bin/ \
        \
        # forward request and error logs to docker log collector
        && ln -sf /dev/stdout /var/log/nginx/access.log \
        && ln -sf /dev/stderr /var/log/nginx/error.log  \
        \
        && ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
        && echo "Asia/Shanghai" > /etc/timezone \
        && rm -rf ~/* ~/.git ~/.gitignore ~/.travis.yml ~/.ash_history \
        && rm -rf /var/cache/apk/* \
        && rm -rf /tmp/* \
        \
        # nginx custom settings
        && mkdir /etc/nginx/ssl/ \
        && mkdir /etc/nginx/logs/ \
        && mkdir /etc/nginx/conf.d/web/ \
        && mkdir /etc/nginx/conf.d/tcp/

COPY config/nginx.conf /etc/nginx/nginx.conf
COPY config/default.conf /etc/nginx/conf.d/default.conf

WORKDIR /var/www

EXPOSE 80 443

STOPSIGNAL SIGTERM

CMD ["nginx", "-g", "daemon off;"]
