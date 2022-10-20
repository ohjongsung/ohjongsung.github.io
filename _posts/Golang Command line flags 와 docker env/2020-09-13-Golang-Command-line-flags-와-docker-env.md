go 로 api server 를 개발하는데, spring 의 [spring.profiles.active](http://spring.profiles.active) 처럼 환경에 맞게 deploy 처럼 할 수 있는 방법 정리

[https://mingrammer.com/gobyexample/command-line-flags/](https://mingrammer.com/gobyexample/command-line-flags/)

### command line flsgs

말 그대로 커맨드 라인에 플래그 값을 주면 코드에서 받아서 사용하는 방법이다.

```bash
go build command-line-flags.go

./command-line-flags -word=opt -numb=7 -fork -svar=flag
word: opt
numb: 7
fork: true
svar: flag
tail: []

./command-line-flags -word=opt
word: opt
numb: 42
fork: false
svar: bar
tail: []

./command-line-flags -word=opt a1 a2 a3
word: opt
...
tail: [a1 a2 a3]

./command-line-flags -word=opt a1 a2 a3 -numb=7
word: opt
numb: 42
fork: false
svar: bar
tail: [a1 a2 a3 -numb=7]

./command-line-flags -h
Usage of ./command-line-flags:
  -fork=false: a bool
  -numb=42: an int
  -svar="bar": a string var
  -word="foo": a string
```

### example code

```go
package main

import "flag"
import "fmt"

func main() {
	wordPtr := flag.String("word", "foo", "a string")
	numbPtr := flag.Int("numb", 42, "an int")
	boolPtr := flag.Bool("fork", false, "a bool")

	var svar string
	flag.StringVar(&svar, "svar", "bar", "a string var")
	flag.Parse()
	fmt.Println("word:", *wordPtr)
	fmt.Println("numb:", *numbPtr)
	fmt.Println("fork:", *boolPtr)
	fmt.Println("svar:", svar)
	fmt.Println("tail:", flag.Args())
}
```

### Dockerfile

도커파일에서는 아래와 같은 방법으로 command line flsgs 를 전달할 수 있다.

```docker
FROM golang:onbuild
ENTRYPOINT ["/go/bin/app", "-name=foo", "-title=bar"]
```
## 실제 활용

위의 두가지 방법을 조합해서 실제로 사용해보자.

### Deployment.yaml

container 의 env 로 profile 값을 정한다.

```yaml
spec:
      containers:
          env:
            - name: profile
              value: dev
```

### Dockerfile

dockerfile 에서는 위에 정한 변수를 받아서 사용하게 설정하면 된다.

```docker
FROM golang:onbuild
ENTRYPOINT ["/go/bin/app", "-profile=${profile}"]
```