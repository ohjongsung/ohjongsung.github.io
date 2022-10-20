윈도우 10 Pro에 도커를 설치하고 mysql 컨테이너를 뛰워보자. 먼저 알아둬야 하는 점은 윈도우 10 Home 이하 버전과 윈도우 10 Pro  이상, 즉 윈도우 서버와 같은 환경은 설치 방법이 다르다는 것이다. 그래서 자신의 윈도우 환경이 10 Home 이하인 경우, 이 글을 더 이상 읽지 말고, Docker Tool Box 설치 방법을 찾아 보길 바란다.

## 윈도우 10 Pro에 Docker 설치

#### 가상화 여부 확인

윈도우 작업 관리자 성능 탭에서 CPU 정보의 가상화 사용 여부를 확인하자. 사용안함인 경우, 컴퓨터 부팅 시, Bios 세팅에 접근해서 가상화 기능을 활성화 시켜야 한다.

#### Hyper-V 활성화

제어판 - Windows 기능 화면에서 Hyper-V 기능을 키고, 재시작을 하자.

#### Docker installer Download

[Link](https://docs.docker.com/) 에서 상단 메뉴 Get Docker -> Windows -> Download for Windows 로 가면 인스톨 파일을 다운로드 할 수 있다. 하지만, 아직 가입을 하지 않았다면, 다운로드 링크가 보이지 않는다. 먼저 가입을 하고 다운로드 하던가 아니면 [Docker for Windows Installer.exe](https://download.docker.com/win/stable/Docker%20for%20Windows%20Installer.exe) 로 다운받자. 물론, 어찌하든 회원가입은 해야한다.

다운로드한 파일을 설치하면, 윈도우 로그아웃 -> 재 로그인을 진행하게 되고 작업표시줄 트레이 아이콘을 보면 도커가 실행되고 있음이 확인된다. 설치 중 선택 항목에 Windows container 또는 Linux container 를 고르는 부분이 있는데, 나는 Linux로 진행했다. 설치 후에도 변경이 가능하다.

#### Kitematic 설치

도커가 설치되면 Kitematic를 설치해야 한다. Kitematic은 도커를 관리 할수 있는 GUI툴이다. 작업 표시줄 트레이 아이콘에서 도커 아이콘을 우클릭해서 kitematic 메뉴를 클리하자. 그러면 Zip 파일을 다운받게 되는데, 요것을 도커가 설치된 폴더에 압축 풀어주자. 이제 Kitematic 을 통해서 도커를 관리하거나, Kitematic 화면 좌측 아래의 Docker CLI 를 클릭해서 윈도우 파워쉘을 사용해서 관리할 수 있다.

#### Disk Image 위치 변경

도커 Settings 메뉴의 Advadnced 항목을 보면 Disk image location 정보가 있다. 기본 세팅은 아래와 같을 것이다.


> C:\Users\Public\Documents\Hyper-V\Virtual Hard Disks\MobyLinuxVM.vhdx


이렇게 기본 세팅을 두고 작업을 하면 C 드라이브의 용량이 부족해지는 순간이 올 것이다.  그래서 좀 더 여유가 있는 보조 드라이브로 옮겨보자. 참고로 인터넷 검색을 통해 찾은 여러 방법들을 시도 해보았지만, 다 실패했다. 도커의 세팅 변경으로는 제대로 먹혀들지 않았다. 그러다 Hyper-V 자체의 기본 가상 하드 디스크 경로를 변경하니 원하는데로 작동했다. 방법은 간단하다.

* 실행 중인 도커 종료
* Hyper-V 관리자 -> 내컴퓨터 선택 -> 우클릭 -> Hyper-V 설정(s) 클릭 후, 가상 하드 디스크, 가상 컴퓨터 항목을 원하는 위치로 변경
* 도커 실행
* 변경한 위치에 MobyLinuxVM.vhdx 생성됨을 확인

## Docker에 Mysql 설치

Docker 의 Kitematic 을 열고 좌측 하단의 Docker CLI 를 클릭해서 윈도우 파워쉘로 진행하겠다.

#### Mysql Docker Image 조회

```bash
docker search mysql
```

#### Mysql Docker Image Download

```bash
docker pull mysql
```

#### Mysql Docker Image 확인

```bash
docker images
```

#### Mysql Docker container 생성

```bash
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password --name mysql_sample mysql
```

* -p 3306:3306 : 호스트의 3306포트와 컨테이너의 3306포트를 연결한다. 즉 호스트에서 3306포트에 접근하면 해당 컨테이너에 접속 된다.
* -e MYSQL_ROOT_PASSWORD=password : 컨테이너를 생성하면서 환경변수를 지정한다. root계정의 비밀번호를 설정한다.
* -name mysql_sample : 컨테이너의 이름은 mysql_sample 로 지정한다.

#### Mysql Docker container 동작 확인

```bash
docker ps -a
```

#### 윈도우 파워쉘에서 Mysql Container 접속

```bash
docker exec -i -t mysql_sample bash
```

#### Mysql 접속

```bash
mysql -u root -p
```

#### Mysql 클라이언트로 접속

Mysql workbench 나 HeidiSQL 과 같은 Mysql 클라이언트로 접속을 시도하면, 다음과 같은 에러를 보게 된다.


> Authentication plugin ‘caching_sha2_password’ cannot be loaded:... 또는 Unable to load authentication plugin 'caching_sha2_password'.


이는 Mysql8 버전에서 사용하는 계정 비밀번호 암호화를 클라이언트가 지원하지 않아서 생기는 문제다. 다시 앞서 설명한 방법으로 Mysql에 접속해서 root 계정의 비밀번호 암호를 다음과 같이 바꿔주자.

```sql
ALTER USER 'root' IDENTIFIED WITH mysql_native_password BY 'password';
```

다시 접속을 시도하면 잘 될 것이다.

## Reference
* https://blog.hanumoka.net/2018/04/28/docker-20180428-windows10pro-install-docker/
* https://blog.hanumoka.net/2018/04/29/docker-20180429-docker-install-mysql/
* https://blog.frankfu.com.au/2018/03/09/change-dockers-default-storage-directory-for-images/
* https://github.com/docker/for-win/issues/2063
* https://stackoverflow.com/questions/49019652/not-able-to-connect-to-mysql-docker-from-local

