nginx 1.18.0 version 을 설치하는 기준으로 작성했다. 따라할 때, 참고해야 한다.

# 설치 파일 다운로드

[https://nginx.org/en/download.html](https://nginx.org/en/download.html)

### pcre

[https://sourceforge.net/projects/pcre/files/pcre/](https://sourceforge.net/projects/pcre/files/pcre/)

### zlib

[https://zlib.net/](https://zlib.net/)

### OpenSSL 1.1.1g

[https://www.openssl.org/source/](https://www.openssl.org/source/)

```bash
cd /usr/local/src
wget http://nginx.org/download/nginx-1.18.0.tar.gz
wget http://zlib.net/zlib-1.2.11.tar.gz
wget http://downloads.sourceforge.net/project/pcre/pcre/8.44/pcre-8.44.tar.gz
wget http://www.openssl.org/source/openssl-1.1.1g.tar.gz
```

# 설치 폴더에 압축 해제

```bash
tar zxf nginx-1.18.0.tar.gz
tar zxf openssl-1.1.1g.tar.gz
tar zxf pcre-8.44.tar.gz
tar zxf zlib-1.2.11.tar.gz
```

# Nginx Compile

```bash
cd nginx-1.18.0

./configure \
--prefix=/usr/local/nginx-1.18.0 \
--conf-path=/usr/local/nginx-1.18.0/conf/nginx.conf \
--error-log-path=/usr/local/nginx-1.18.0/logs/error.log \
--http-log-path=/usr/local/nginx-1.18.0/logs/access.log \
--user=nginx \
--group=nginx \
--lock-path=/usr/local/nginx-1.18.0/system/nginx.lock \
--pid-path=/usr/local/nginx-1.18.0/system/nginx.pid \
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

# Configuration summary 확인

```bash
Configuration summary
  + using threads
  + using PCRE library: ../pcre-8.44
  + using OpenSSL library: ../openssl-1.1.1g
  + using zlib library: ../zlib-1.2.11

  nginx path prefix: "/usr/local/nginx-1.18.0"
  nginx binary file: "/usr/local/nginx-1.18.0/sbin/nginx"
  nginx modules path: "/usr/local/nginx-1.18.0/modules"
  nginx configuration prefix: "/usr/local/nginx-1.18.0/conf"
  nginx configuration file: "/usr/local/nginx-1.18.0/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx-1.18.0/system/nginx.pid"
  nginx error log file: "/usr/local/nginx-1.18.0/logs/error.log"
  nginx http access log file: "/usr/local/nginx-1.18.0/logs/access.log"
  nginx http client request body temporary files: "/var/cache/nginx/client_body_temp"
  nginx http proxy temporary files: "/var/cache/nginx/proxy_temp"
  nginx http fastcgi temporary files: "/var/cache/nginx/fastcgi_temp"
  nginx http uwsgi temporary files: "/var/cache/nginx/uwsgi_temp"
  nginx http scgi temporary files: "/var/cache/nginx/scgi_temp"
```

# nignx 설치

```bash
make
make install
```

# nginx user & group 생성

```bash
useradd --system --shell /sbin/nologin --comment "nginx user" --user-group nginx
```

# symbolic link & cache directory 생성

심볼릭 링크를 만들어 nginx 업그레이드에 유연하게 대처할 수 있게 한다.

compile 할 때 설정한 캐시 디렉토리를 생성해야 한다.

```bash
ln -s /usr/local/nginx-1.18.0/ /usr/local/nginx
mkdir /var/cache/nginx/
```

# nginx version 확인

```bash
/usr/local/nginx/sbin/nginx -V
```

# Systemd 등록

Configuration summary 값을 확인해서 수정해야 한다.

실행 경로가 symbolic link 를 바라보게 해서 최신 버전 nginx 를 설치하고 symblic link 만 교체하면 되게끔 한다.

```bash
rm -rf /usr/lib/systemd/system/nginx.service
vi /usr/lib/systemd/system/nginx.service

[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/usr/local/nginx/system/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

# Nginx Service 실행

```bash
systemctl daemon-reload
systemctl enable nginx.service
systemctl start nginx.service
```
