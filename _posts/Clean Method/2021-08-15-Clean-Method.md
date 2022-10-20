가능한 충분히 작은 사이즈로 한 가지 역할만 수행하고, 테스트 가능하고 중복이 없어야 한다. 파리미터 개수는 줄이고, 내부 코드를 보거나 API 문서를 보지 않아도 될만큼 이해하기 쉽게 만들어야 한다.

## 파라미터

파라미터 개수가 3개 이상일 때, 메소드가 너무 많은 역할을 하고 있을 가능성이 있다. 메소드 분할을 하거나 파라미터 오브젝트를 사용한다.

### 메소드 분할

```java
public int calculate(1, 2, "add");

// refactoring
public int add(1, 2)
public int divide(1, 2)
```

## 메소드 크기

똑같은 크기(줄 수)라도 품질은 다를 수 있다. 라인 수 보다 코드 품질에 대해서 끊임 없이 고민하고 개선해야 한다. 읽고 이해하기 쉬운가? 메소드 동작을 설명하기 위해 주석을 달아야한다면?! 더 작게 분할 가능한가? 등

```java
public void saveUser(email, password) {
	User user = new User(email, password);
	if(email != null && email.contains("@") && email.contains(".")) {
		if(!isDuplicateEmail(email)) {
			if(password.lenght() > 5 && password.lenght() < 15) {
				user.save();
			}
		}
	}
}
```

```java
public void saveUser(email, password) throws UserException {
	validateEmailFormat(email);
	validateEmailDuplication(email);
	validatePassword(password);
	
	User user = new User(email, password);
	
	user.save();

}
```

## 메소드 추상화 레벨

| High Level | Intermediate Level | Low Level | Lower Level |
| --- | --- | --- | --- |
| 회원가입 |사용자 정보 저장 | 이메일 포멧 검증 | 이메일 @ 체크 |
| | 이메일 발송 | 이메일 중복 검증 | 패스워드 길이 체크 |
| | 쿠폰 발행 | 패스워드 검증 | |


추상화의 수준은 높을 수록 무엇을 하는 건지에 대해 말하고, 낮을 수록 그것을 어떻게 하는지를 말한다.

### 하나의 메소드는 동일한 추상화 수준만 가져야 한다.

아래의 코드를 보면, 하나의 메소드에 추상화 수준이 뒤섞여 있다.

```java
public void saveUser(email, password) {
	User user = new User(email, password);
	if(email != null && email.contains("@") && email.contains(".")) {
		if(!isDuplicateEmail(email)) {
			if(password.lenght() > 5 && password.lenght() < 15) {
				user.save();
			}
		}
	}
}
```

이를 개선하면, (대충 맥락만 이해하자)

클래스 내의 메소드 순서는 추상화 레벨 High 에서 Low 순으로 작성한다.

```java
public String signUp(email, password) {
	saveUser(email, password);
	sendWelcomeEmail(email);
	issueCoupon(user);
	
	return "success";
}

public void saveUser(email, password) throws UserException {
	validateEmailFormat(email);
	validateEmailDuplication(email);
	validatePassword(password);
	
	User user = new User(email, password);
	
	user.save();
}

private validateEmailFormat(email) throws UserException {
	if(!email.contains("@") && !email.contains(".")) {
			throw new UserException(...);
	}
}

private validateEmailDuplication(email) throws UserException {
	if(userService.getUserByEmail(email)) {
			throw new UserException(...);
	}
}

private validatePassword(password) throws UserException {
	if(password.lenght() < 5 || password.lenght() > 15) {
			throw new UserException(...);
	}
}
```