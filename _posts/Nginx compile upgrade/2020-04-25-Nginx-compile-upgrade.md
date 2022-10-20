[nginx compile install](https://ohjongsung.github.io/Nginx-compile-install) 과 연계된다. 1.8.0 을 1.19.1 로 업그레이드한다.

# 최신 버전 다운로드

[https://nginx.org/download/nginx-1.19.1.tar.gz](https://nginx.org/download/nginx-1.19.1.tar.gz)

```bash
cd /usr/local/src
wget https://nginx.org/download/nginx-1.19.1.tar.gz
```

# 설치 폴더에 압축 해제

```bash
tar zxf nginx-1.19.1.tar.gz
```

# Nginx Compile

버전만 1.18.0 에서 1.19.1 로 변경하고 나머지는 똑같다.

```bash
cd nginx-1.19.1

./configure \
--prefix=/usr/local/nginx-1.19.1 \
--conf-path=/usr/local/nginx-1.19.1/conf/nginx.conf \
--error-log-path=/usr/local/nginx-1.19.1/logs/error.log \
--http-log-path=/usr/local/nginx-1.19.1/logs/access.log \
--user=nginx \
--group=nginx \
--lock-path=/usr/local/nginx-1.19.1/system/nginx.lock \
--pid-path=/usr/local/nginx-1.19.1/system/nginx.pid \
--without-http_autoindex_module \
--without-http_ssi_module \
--with-file-aio \
--with-debug \
--with-pcre=../pcre-8.44 \
--with-zlib=../zlib-1.2.11 \
--with-http_ssl_module \
--with-openssl=../openssl-1.1.1g \
--with-openssl-opt=enable-weak-ssl-ciphers \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_random_index_module \
--with-http_secure_link_module \
--with-http_stub_status_module \
--with-http_auth_request_module \
--with-http_addition_module \
--with-http_realip_module \
--with-http_stub_status_module \
--with-threads --with-stream \
--with-stream_ssl_module \
--with-http_v2_module \
--http-client-body-temp-path=/var/cache/nginx/client_body_temp \
--http-proxy-temp-path=/var/cache/nginx/proxy_temp \
--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
--http-scgi-temp-path=/var/cache/nginx/scgi_temp
```

# Nginx Service 중지

```bash
systemctl stop nginx.service
```

# symbloic link 변경

```bash
cd /usr/local
rm -rf nginx
ln -s nginx-1.19.1 nginx
rm -rf nginx-1.19.1/conf/*
# mkdir -p nginx-1.19.1/conf/conf.d
cp -rf nginx-1.18.0/conf/* nginx-1.19.1/conf
```

### Nginx Service 시작

```bash
systemctl restart nginx.service
```