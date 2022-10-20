'HTTPS는 속도가 느리다', 'HTTPS 인증서 비용이 비싸다', 'HTTPS 인증서 발급 과정이 어렵다' 와 같은 말은 이제 옛말이 된거 같다. 속도 문제는 과거에 비해 CPU와 메모리의 성능이 매우 좋아졌다. 게다가 [Let's Encrypt](https://letsencrypt.org/)라는 무료 HTTPS 인증서 보급 기관의 등장 덕분에 비용과 발급 문제도 해결되어 HTTPS 보급에 큰 공을 세우고 있다. Let's Encrypt를 사용해서 Nginx에 HTTPS를 적용한 경험을 공유한다. 서버는 Ubuntu 16.04.1 LTS 이다.

먼저, 등록하려는 도메인은 mydomain.com이며, HTML은 /var/www/mydomain에서 제공된다고 가정하고 진행하겠다. 해당 부분을 자신의 도메인 명, WEB Root로 수정해서 따라하면 된다.

## Nginx 설정

* /etc/nginx/snippets/letsencrypt.conf 파일을 만들고 아래의 내용을 작성한다.

```bash
location ^~ /.well-known/acme-challenge/ {
	default_type "text/plain";
	root /var/www/letsencrypt;
}
```

* /etc/nginx/snippets/ssl.conf 파일을 만들고 아래의 내용을 작성한다. ([참고](https://cipherli.st/))

```bash
ssl_protocols TLSv1.2;
ssl_prefer_server_ciphers on;
ssl_ciphers "ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384";
ssl_ecdh_curve secp384r1; # Requires nginx >= 1.1.0
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off; # Requires nginx >= 1.5.9
ssl_stapling on; # Requires nginx >= 1.3.7
ssl_stapling_verify on; # Requires nginx => 1.3.7
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
```

* 아래의 폴더를 만든다. (폴더가 만들어졌는지 꼭 확인하자)

```bash
sudo mkdir -p /var/www/letsencrypt/.well-known/acme-challenge
```

## Nginx virtual hosts 설정(HTTP)

아직 HTTPS 인증이 되지 않았으므로 HTTP로 서비스되게 작성한다.

* /etc/nginx/sites-available/mydomain.conf 파일을 만들고 아래의 내용을 작성한다.

```bash
server {
	listen 80 default_server;
	listen [::]:80 default_server ipv6only=on;
	server_name mydomain.com www.mydomain.com;

	include /etc/nginx/snippets/letsencrypt.conf;

	root /var/www/mydomain;
	index index.html;
	location / {
		try_files $uri $uri/ =404;
	}
}
```

* 사이트 활성화

```bash
rm /etc/nginx/sites-enabled/default
ln -s /etc/nginx/sites-available/mydomain.conf /etc/nginx/sites-enabled/mydomain.conf
```

* Nginx reload

```bash
sudo systemctl reload nginx
```

## Certbot 설치

HTTPS 인증서를 쉽게 발급받을 수 있는 이유는 Certbot 덕분이다. Certbot은 클라이언트로 Let's Encrypt 인증을 쉽게 해준다. Let's Encrypt는 Ubuntu 16.04 LTS 에서 기본패키지로 추가되었다. 그래서 쉽게 설치할 수 있다. 하지만, 오래된 버전이기 때문에 최신 버전 설치를 위해 다음과 같이 해야한다.

```bash
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install certbot
```

## 인증서 받기

```bash
certbot certonly --webroot --webroot-path=/var/www/letsencrypt -d www.mydomain.com -d mydomain.com
```

#### 명령어 설명

* -d 는 도메인명을 작성하면 된다. 일반적으로 기본 도메인과 www 도메인 두개를 지정한다.
* -webroot 는 웹인증을 받을 것이라는 것이다. 외부 인증프로그램이 -d 에 지정된 도메인 사이트에 접속한다.
* -webroot-path 는 웹루트의 경로이다. 인증 프로그램이 이 경로에 임시 랜덤 파일을 생성하고, 외부 인증프로그램이 이 파일을 접근할 수 있다면 인증서가 발급되는 것이다.

명령어 실행 뒤, 인증서 관련 메시지를 수신할 메일 주소를 입력하고 별 문제가 없으면 /etc/letsencrypt/live/www.mydomain.com/ 폴더에 인증서가 발급된다.

## Nginx virtual hosts 설정(HTTPS)

인증서를 발급받았으니, /etc/nginx/sites-available/mydomain.conf의 설정을 HTTPS로 변경해야 한다.

```bash
## http://mydomain.com and http://www.mydomain.com redirect to https://www.mydomain.com
server {
	listen 80 default_server;
	listen [::]:80 default_server ipv6only=on;
	server_name mydomain.com www.mydomain.com;

	include /etc/nginx/snippets/letsencrypt.conf;

	location / {
		return 301 https://www.mydomain.com$request_uri;
	}
}

## https://mydomain.com redirects to https://www.mydomain.com
server {
	listen 443 ssl http2;
	listen [::]:443 ssl http2;
	server_name mydomain.com;

	ssl_certificate /etc/letsencrypt/live/www.mydomain.com/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/www.mydomain.com/privkey.pem;
	ssl_trusted_certificate /etc/letsencrypt/live/www.mydomain.com/fullchain.pem;
	include /etc/nginx/snippets/ssl.conf;

	location / {
		return 301 https://www.mydomain.com$request_uri;
	}
}

## Serves https://www.mydomain.com
server {
	server_name www.mydomain.com;
	listen 443 ssl http2 default_server;
	listen [::]:443 ssl http2 default_server ipv6only=on;

	ssl_certificate /etc/letsencrypt/live/www.mydomain.com/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/www.mydomain.com/privkey.pem;
	ssl_trusted_certificate /etc/letsencrypt/live/www.mydomain.com/fullchain.pem;
	include /etc/nginx/snippets/ssl.conf;

	root /var/www/mydomain;
	index index.html;
	location / {
		try_files $uri $uri/ =404;
	}
}
```

* Nginx reload

```bash
sudo systemctl reload nginx
```

HTTPS가 적용됐는지 접속해보자.

## 인증서 갱신

Let's Encrypt에서 발급하는 인증서는 90일에 한번씩 갱신해야 한다. Cerbot은 만료기간 30일 이내의 모든 인증서를 갱신할 수 있다.

* cron 수정

```bash
sudo crontab -e
```

혹시나 에디터 선택문구가 출력된다면 3번을 선택한다.

* 아래의 내용을 추가한다.

```bash
20 3 * * * sudo certbot renew --renew-hook "sudo service nginx restart"
```

* dry run으로 실행 테스트

```bash
sudo certbot renew --dry-run
```

## SSL rating

아래의 사이트에서 SSL Server 수준을 테스트해보자.

* <https://www.ssllabs.com/ssltest/analyze.html>

## Reference
* <https://gist.github.com/cecilemuller/a26737699a7e70a7093d4dc115915de8>
* <https://blog.lael.be/post/5107>
* <http://blog.naver.com/itperson/220853849351>













