CAPTCHA(Completely Automated Public Turing test to tell Computers and Humans Apart, 완전 자동화된 사람과 컴퓨터 판별,캡차)는 HIP(Human Interaction Proof) 기술의 일종으로, 어떠한 사용자가 실제 사람인지 컴퓨터 프로그램인지를 구별하기 위해 사용되는 방법이다. 흔히 웹사이트 회원가입을 할 때 뜨는 자동가입방지 프로그램 같은 곳에 쓰인다.

## reCAPTCHA 적용하기

Spring에 reCAPTCHA를 적용해보자.

## Google reCAPTCHA에 사이트 등록

reCAPTCHA를 사용하기 위해서는 먼저 <https://www.google.com/recaptcha/admin>에서 자신의 사이트를 등록 해야한다. 등록을 하면 site-key와 secret-key를 받는데, 이를 가지고 Google  reCAPTCHA API를 사용하게 된다.

## API key-pair 설정

site-key와 secret-key를 properties 파일에 써놓고 사용한다.
```text
google.recaptcha.key.site=6LfaHiITAAAA...
google.recaptcha.key.secret=6LfaHiITAAAA...
```

## reCAPTCHA 관련 script와 html을 view에 작성

reCAPTCHA를 사용하려는 화면 상단에 아래의 스크립트를 등록한다.

```javascript
<script type="text/javascript">
        var onloadCallback = function() {
            grecaptcha.render('g-recaptcha', {
                'sitekey' : '<c:out value="${recaptchaSiteKey}" />',
                'callback' : function(response) {
                    document.getElementById('recaptchaResponse').value = response;
                },
                'theme' : 'light'
            });
        }
</script>
<script src="https://www.google.com/recaptcha/api.js?onload=onloadCallback&render=explicit" async defer></script>
```

화면의 form Tag 안에 아래의 Element를 추가한다.

```html
<div id="g-recaptcha"></div>
<form:hidden path="recaptchaResponse"/>
```

* 클라이언트 API의 로드는 script 태그 안에 정의되어 있다. 로드 될 때, onload 파라미터 덕분에 onloadCallback 메서드를 호출한다. 그리고 &render=explicit는 reCAPTCHA 위젯의 렌더링을 클라이언트(브라우저)에 맡기는 거라고 보면 된다.

* 실제 렌더링은 onloadCallback에서 시작된다. grecaptcha.render를 호출하면 위젯이 id="g-recaptcha"를 가지는 div에 그려진다. 이때, sitekey를 파라미터로 전달한다.

* grecaptcha.reder에는 콜백함수가 정의되어 있다. 이는 사용자가 i'm not a robot 체크박스를 클릭하면, 응답 토큰을 받게 된다. 응답 토큰은 히든 폼 필드인 recaptchaResponse에 값이 저장되고, form submit 에 값이 담겨간다. 이제 recaptchaResponse의 토큰 값을 가지고 서버에서 유효성 검사를 하면 된다.

## 서버에서 유효성 검사

서버단에서 https://www.google.com/recaptcha/api/siteverify URL로 쿼리 매개 변수 secret, response 및 remoteip를 담아 HTTP 요청을 하면, 아래의 형태로 json 응답을 받는다.

```json
{
    "success": true|false,
    "challenge_ts": timestamp,
    "hostname": string,
    "error-codes": [ ... ]
}
```

#### form submit을 처리하는 RegistrationController

```java
public class RegistrationController {
 
    @Autowired
    private ICaptchaService captchaService; 

    @RequestMapping(value = "/user/registration", method = RequestMethod.POST)
    public String registerUserAccount(@Valid UserDto accountDto, HttpServletRequest request) {
        String response = request.getParameter("g-recaptcha-response");
        String result = captchaService.processResponse(response);
 
        // Rest of implementation
    }
}
```

#### 유효성 검사를 위한 CaptchaService

```java
public class CaptchaService implements ICaptchaService {
 
    @Value("${google.recaptcha.key.secret}")
    private String secret;
 
    @Autowired
    private HttpServletRequest request;

    @Autowired
    private RestOperations restTemplate;
 
    private static Pattern RESPONSE_PATTERN = Pattern.compile("[A-Za-z0-9_-]+");
 
    @Override
    public String processResponse(String response) {
        if(!responseSanityCheck(response)) {
            return "Response contains invalid characters";
        }
 
        URI verifyUri = URI.create(String.format(
          "https://www.google.com/recaptcha/api/siteverify?secret=%s&response=%s&remoteip=%s",
          secret, response, getClientIP()));
 
        GoogleResponse googleResponse = restTemplate.getForObject(verifyUri, GoogleResponse.class);
 
        if(!googleResponse.isSuccess()) {
            return "reCaptcha was not successfully validated";
        }
        
        return "success";
    }
 
    private boolean responseSanityCheck(String response) {
        return StringUtils.hasLength(response) && RESPONSE_PATTERN.matcher(response).matches();
    }

    private String getClientIP() {
        final String xfHeader = request.getHeader("X-Forwarded-For");
        if (xfHeader == null) {
            return request.getRemoteAddr();
        }
        return xfHeader.split(",")[0];
    }
}
```
* processResponse 메소드의 return 타입을 String으로 해놨는데, 좋은 방법이 아니다. 참고한 github 소스에는 Exception throw로 되어 있는데, $.post 로 form submit 하는 것을 가정하고 작성된 것이다. 그래서 내 경우와는 맞지 않고 설명하자니 주제에 어긋나는 내용이 너무 길어져서 유효성 검사 후 결과에 대한 처리는 각자 상황에 맞게 알아서 하는 걸로 생략한다.

#### json 응답 클래스 GoogleResponse

```java
@JsonInclude(JsonInclude.Include.NON_NULL)
@JsonIgnoreProperties(ignoreUnknown = true)
@JsonPropertyOrder({
    "success",
    "challenge_ts",
    "hostname",
    "error-codes"
})
public class GoogleResponse {
 
    @JsonProperty("success")
    private boolean success;
     
    @JsonProperty("challenge_ts")
    private String challengeTs;
     
    @JsonProperty("hostname")
    private String hostname;
     
    @JsonProperty("error-codes")
    private ErrorCode[] errorCodes;
 
    @JsonIgnore
    public boolean hasClientError() {
        ErrorCode[] errors = getErrorCodes();
        if(errors == null) {
            return false;
        }
        for(ErrorCode error : errors) {
            switch(error) {
                case InvalidResponse:
                case MissingResponse:
                    return true;
            }
        }
        return false;
    }
 
    static enum ErrorCode {
        MissingSecret,     InvalidSecret,
        MissingResponse,   InvalidResponse;
 
        private static Map<String, ErrorCode> errorsMap = new HashMap<String, ErrorCode>(4);
 
        static {
            errorsMap.put("missing-input-secret",   MissingSecret);
            errorsMap.put("invalid-input-secret",   InvalidSecret);
            errorsMap.put("missing-input-response", MissingResponse);
            errorsMap.put("invalid-input-response", InvalidResponse);
        }
 
        @JsonCreator
        public static ErrorCode forValue(String value) {
            return errorsMap.get(value.toLowerCase());
        }
    }
     
    // standard getters and setters
}
```

* success 값이 true면 사용자가 사람임을 확인한 것이다. false면 errorCodes에 실패한 이유가 담겨오는데, 이것이 사용자의 잘못인지(InvalidResponse, MissingResponse) 서버 검증 로직의 잘못(MissingSecret, MissingResponse)인지 hasClientError 메소드로 판단한다.

## Reference
* <https://github.com/Baeldung/spring-security-registration>
* <https://kielczewski.eu/2015/07/spring-recaptcha-v2-form-validation/>
* <http://www.nomadism.co.kr/19>
* <http://ohgyun.com/590>
* <https://ko.wikipedia.org/wiki/CAPTCHA>




