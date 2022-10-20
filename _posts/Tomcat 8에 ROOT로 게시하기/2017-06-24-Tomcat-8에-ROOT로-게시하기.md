Tomcat 8에 지금 이 블로그를 ROOT로 게시하기 위해 여기저기 정보를 검색했었다. 계속해서 에러가 나고 원하는데로 게시가 되지 않았다. 지금은 잘 운영되고 있지만, 관련 내용을 정리해 둔다. 시일이 좀 지난터라 정확한 삽질 내용과 순서가 흐릿하다.

## 삽질기

[Tomcat 설치디렉토리]/webapps/ROOT 폴더를 context path="/"로 판단하는 것을 몰랐었다. 이유를 알고 blog 폴더와 root 폴더에 해당하는 context path를 아래처럼 수정했다.

```xml
<Host name="localhost" appBase="webapps" >
    <Context path="/" docBase="blog.war" />
    <Context path="/ROOT" docBase="ROOT" />
</Host>
```

그러나 http://localhost:port/와 http://localhost:port/blog 두 가지 경로로 접근이 가능했다. 이유는 [Tomcat 설치디렉토리]/webapps에 blog.war 파일을 업로드 했기 때문이었다.

blog.war를 ROOT.war로 올리면 된다는 글을 봤고 원하는데로 구동될거 같았지만, WAR 파일 명을 변경하고 싶지 않았다. 그러던 중에 톰캣 설치 시, 딸려오는 ROOT 폴더는 나한테 전혀 필요가 없으니 다른 곳에 백업해도 문제 없다는 것을 알았다. 그리고 Host element 아래에 context를 설정하는 것은 톰캣 4이전의 방식이라는 것을 알았다.

마지막으로 [$CATALINA_BASE]/conf/[enginename]/[hostname]/ROOT.xml로 설정하면 [Tomcat 설치디렉토리]/webapps에 ROOT 폴더로 unpack됨을 알았다.

#### [$CATALINA_BASE]/conf/serverl.xml
```xml
<Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true"/>
```
#### [$CATALINA_BASE]/conf/Catalina/localhost/ROOT.xml
```xml
<Context path="" docBase="/업로드폴더/blog.war"
		 reloadable="true" crossContext="true" />
```

위와 같이 설정하니 원하는데로 게시되어 잘 돌아갔다. hostname은 보통 IP 주소나 도메인 명으로 한다.

## 톰캣 특성

삽질하면서 알게된 톰캣 특성을 정리하면 다음과 같다.

```xml
<Host name="localhost" appBase="webapps" />
```

* 기본적으로 servel.xml을 보면 host의 appbase가 [Tomcat 설치디렉토리]/webapps이다. 이렇게 되면, URL 상의 기본 web root는 [appBase]/ROOT/ 그러니까 webapps/ROOT가 된다. 즉,  http://localhost:port/test.jsp를 호출하면 [appBase]/ROOT/test.jsp를 출력하게 된다.

```xml
<Host name="localhost" appBase="webapps">
    <Context path="/" docBase="/web"/>
</Host>
```

* Host element 아래에 Context를 설정할 수 있다. path는 URL 상의 주소가 되며 docBase는 어플리케이션 서버상의 위치가 된다. docBase 값이 상대경로면 Host의 appBase부터의 상대경로가 되며, 절대경로면 서버상의 절대경로가 된다.

* 톰캣 5.5 버전 이후부터 Context 설정 위치가 [$CATALINA_BASE]/conf/[enginename]/[hostname]/로 변경되었다. 이 덕분에 톰캣을 재시작할 필요없이 Context 설정 변경이 가능해 졌다. 기본적으로 [$CATALINA_BASE]/conf/[enginename]/[hostname]/ROOT.xml 파일을 만들어서 설정한다. xml 파일명이 Context path가 되는 구조며 2단계 이상의 폴더 구조 Context도 지원한다. 경로가 /blog/category와 같은 경우, blog#category.xml로 파일명을 작성하면 된다.

* 톰캣은 context 경로가 명시적으로 설정되어 있지 않으면 다음과 같이 움직인다.  [Tomcat 설치디렉토리]/webapps의 ROOT 폴더를 context path="/"로 판단한다. [Tomcat 설치디렉토리]/webapps의 서브 폴더들은 web root의 서브 경로로 취급한다.

* [Tomcat 설치디렉토리]/webapps에 blog.war를 업로드하면 톰캣은 context 설정한 것과 context path="/blog"로 두 가지 경로가 혼용된다.

## Reference
* <http://bigzero37.tistory.com/16>
* <http://egloos.zum.com/playgame/v/287965>
* <http://seeit.kr/24>
* <http://pshcode.tistory.com/109>


