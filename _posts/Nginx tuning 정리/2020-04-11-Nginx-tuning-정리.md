## The Backlog Queue

### net.core.somaxconn

NGINX가 수락하도록 대기열에 넣을 수 있는 최대 연결 수 기본값은 종종 매우 낮으며, NGINX는 연결을 매우 빠르게 받아들이기 때문에 일반적으로 허용되지만, 웹 사이트가 많은 트래픽을 경험할 경우 이를 늘려야 한다. listen()으로 바인딩 된 서버 소켓에서 accept()를 기다리는 소켓 개수에 관련된 커널 파라미터

```bash
sysctl net.core.somaxconn
sudo sysctl -w net.core.somaxconn="1024"
```

이 값을 512보다 큰 값으로 설정하려면 백 로그 매개 변수를 NGINX listen 지시문으로 변경

```bash
listen *:8080 backlog=16384;
```

### net.ipv4.tcp_max_syn_backlog

net.core.somaxconn'이 accept()을 기다리는 ESTABLISHED 상태의 소켓(즉, connection completed)을 위한 queue라면, 'net.ipv4.tcp_max_syn_backlog'는 SYN_RECEIVED 상태의 소켓(즉, connection incompleted)을 위한 queue

```bash
sysctl net.ipv4.tcp_max_syn_backlog
sudo sysctl -w net.ipv4.tcp_max_syn_backlog="1024"
```

### net.core.netdev_max_backlog

CPU에 전달되기 전에 패킷이 네트워크 카드로 버퍼링되는 속도. 값을 늘리면 대역폭이 많은 컴퓨터의 성능이 향상될 수 있다.

각 네트워크 장치 별로 커널이 처리하도록 쌓아두는 queue의 크기를 설정한다. 커널의 패킷 처리 속도가 이 queue에 추가되는 패킷의 인입 속도보다 떨어진다면 미처 queue에 추가되지 못한 패킷들은 버려진다.

이 커널 파라미터도 trade-off 관계가 메모리 사용량 밖에 없으므로, 적당히 증가시켜두는 것도 괜찮다.

```bash
sysctl net.core.netdev_max_backlog
sudo sysctl -w net.core.netdev_max_backlog="30000"
```

## File Descriptors

파일 설명자는 무엇보다도 연결과 파일을 여는 데 사용되는 운영 체제 자원이다. NGINX는 연결당 최대 2개의 파일 설명자를 사용할 수 있다. 예를 들어, NGINX가 프록시를 사용하는 경우 일반적으로 클라이언트 연결에는 하나의 파일 설명자를 사용하고 프록시 서버에 대한 연결에는 다른 파일 설명자를 사용하지만 HTTP keepalive를 사용하는 경우에는 이 비율이 훨씬 낮다.

리눅스를 비롯한 일반적인 유닉스에서 소켓은 마치 파일과 같은 취급을 받는다. 전체 시스템에서 가질 수 있는 파일 개수가 제한이 있다면, 당연히 소켓의 전체 개수에 영향을 끼친다.

리눅스에서 전체 시스템이 가질 수 있는 최대 파일 개수 제한은 '`fs.file-max`' 커널 파라미터에서 설정 된다. 이 값은 일반적으로 적당히 큰 값이 설정되어 있으므로, 웬만하면 손 볼 일이 없을 것입니다.

```bash
# 최대 파일 개수 제한
sysctl fs.file-max

# 현재 열려있는 파일 현황
sysctl fs.file-nr
```

프로세스별 제한 설정인 user limit 값을 살펴봐야 한다.

```bash
ulimit -a
```

```bash
sudo vi /etc/security/limits.conf

* soft    nofile  128000
* hard    nofile  128000
```

## Ephemeral Ports

NGINX가 프록시로 작동하는 경우 업스트림 서버에 대한 각 연결은 임시 포트 또는 임시 포트를 사용

```bash
cat /proc/sys/net/ipv4/ip_local_port_range
sysctl -w net.ipv4.ip_local_port_range="1024 65535"
```

# NGINX Configuration

## Worker Processes

최적의 값은 CPU 코어 수, 데이터를 저장하는 하드 디스크 드라이브의 수, 로드 패턴을 포함한 많은 요인에 따라 결정된다. 의심스러울 때, 사용 가능한 CPU 코어 수로 설정하는 것이 좋은 시작일 것이다

```bash
worker_processes number | auto;
worker_processes 1;
```

## Worker Connections

작업자 프로세스에서 열 수 있는 최대 동시 연결 수를 설정한다. 이 숫자는 클라이언트와의 연결뿐만 아니라 모든 연결(예: 프록시 서버와의 연결 등)을 포함한다는 점을 유념해야 한다. 또 다른 고려사항은 동시 접속의 실제 횟수가 worker_rlimit_nofile이 변경할 수 있는 열린 파일의 최대 수에 대한 현재 제한을 초과할 수 없다는 것이다.

```bash
worker_connections number;
worker_connections 512;
```

작업자 프로세스에 대한 최대 열린 파일 수 (RLIMIT_NOFILE)에 대한 제한을 변경합니다. 기본 프로세스를 다시 시작하지 않고 한계를 늘리는 데 사용됩니다.

```bash
worker_rlimit_nofile number;
```

## Keepalive Requests

단일 keep-alive 연결을 통해 서비스할 수 있는 요청의 최대 수를 설정한다. 최대 요청 횟수가 이루어진 후에는 연결이 종료된다. 연결당 메모리 할당을 해제하려면 정기적으로 연결을 닫아야 한다. 따라서, 너무 많은 최대 요청 수를 사용하면 메모리 사용량이 과도하게 증가하여 권장되지 않을 수 있다.

기본값은 100이지만 일반적으로 단일 클라이언트에서 많은 요청을 보내는 부하 발생 도구를 사용하여 테스트하는 데 훨씬 더 높은 값이 특히 유용할 수 있다.

```bash
keepalive_requests number;
keepalive_requests 100;
```

## Keepalive Timeout

첫 번째 매개 변수는 유지 관리 클라이언트 연결이 서버 측에서 열려 있는 동안 타임아웃을 설정한다. 0 값은 유지 관리 클라이언트 연결을 비활성화한다. 선택적 두 번째 매개 변수는 "Keep-Alive : timeout=time"응답 헤더 필드의 값을 설정한다. 두 가지 매개변수가 다를 수 있다. "Keep-Alive : timeout=time"헤더 필드는 Mozilla와 Konqueror에 의해 인식된다. MSIE는 약 60초 안에 킵-얼라이브 연결을 스스로 닫는다.

```bash
keepalive_timeout timeout [header_timeout];
keepalive_timeout 75s;
```

## Keepalive

업스트림 서버에 연결할 캐시를 활성화한다. 연결 매개변수는 각 작업자 프로세스의 캐시에 보존되는 업스트림 서버에 대한 최대 유휴 유지 접속 수를 설정한다. 이 숫자를 초과하면 가장 최근에 사용한 연결이 닫힌다.

```bash
keepalive connections;

upstream http_backend {
    server 127.0.0.1:8080;
    keepalive 16;
}

server {
    ...

    location /http/ {
        proxy_pass http://http_backend;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        ...
    }
}
```

## Configuring Socket Sharding

```bash
http {
     server {
          listen 80 reuseport;
          server_name  localhost;
          # ...
     }
}

stream {
     server {
          listen 12345 reuseport;
          # ...
     }
}
```

## epoll

Nginx는 non-blocking I/O 방식을 사용하므로 자신에게 요청해온 connection file에 read 가능한지를 계속해서 확인해 주어야 한다.

multi-thread 방식에서는 context-switching을 계속 해야 하므로 대량 호출이 발생할 경우, 자원을 많이 사용하게 된다면 non-blocking I/O를 사용하는 single-thread 방식의 Nginx에서는 자신에게 연결되어 있는 socket에 읽을 데이터가 있는지 계속 확인하는데 자원을 낭비할 수 있다고 한다.

Nginx에 들어오는 연결이 적을 경우 낭비되는 자원이 크지 않지만, 사용자가 많아질 경우 idle 상태로 연결된 connection이 많다면 이 connection들을 스캔하는 데 많은 자원이 필요하게 되어 그러한 시스템에서는 아래와 같이 설정하도록 권장하고 있다.

```bash
events {
    use epoll;
    worker_connections  1024;
}
```

위에서 지정한 epoll은 Linux에서 socket을 관리하는 데 사용하는 방식 중 하나로, epoll 외에도 poll, select 방식이 있다.

이 중 poll과 select는 해당 프로세스에 연결된 모든 connection file을 스캔하지만, epoll은 수천개의 file descriptor를 처리할 수 있도록 보다 효율적인 알고리즘을 사용하고 있어 대량 요청이 발생하는 시스템에 적합하다고 한다.