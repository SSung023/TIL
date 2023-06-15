## application.yml에서 원하는 값 불러오기

> 개발하다보면 특정 값을 불러와야할 때가 있다.  
> private final 변수로 선언하고 불러오면 되지 않나 싶을 수도 있지만, 해당 내용이 외부로 공개되면 안되는 경우 *(ex REST API KEY, JWT secret key)* 에는 곤란할 수 있다.  
> (Github에 코드가 올라가는 경우 코드를 다운로드 받으면 노출이 되기 때문)  
> 
> 그럼 이런 값을 어떻게 불러올까?  
> 바로 application.yml 설정 파일을 이용하는 것이다.  지금부터 방법을 알아보자!


### application 프로파일 만들기
예를 들어 JWT에 관련된 정보를 저장하고 불러오고 싶다고 가정해보자.  
application.yml 이라는 설정 파일에 한 번에 넣고 값을 불러오는 방법도 있겠지만, 만약 application.yml 파일에 저장해놓은 정보가 많아지면 찾기 힘들어질 것이다.

따라서 application-{profile}.yml 형태의 JWT 관련 정보만 넣고 관리할 수 있는 프로파일을 만들면 관리하기가 더 쉬울 것이다!  


application 프로파일을 만드는 방법은 간단하다.    
>***application-{profile}.yml*** 

형태로 파일을 생성하면 된다. JWT 관련 정보를 넣고 싶으니 application-jwt.yml라는 이름으로 작성하자.

***

### application-{profile}.yml에서 값 읽어들이기
```
jwt:
    header: Authorization
    secret: secretkeysecretkeysecretkeysecretkeysecretkeysecretkey
    refresh-secret: refreshsecretkeyrefreshsecretkeyrefreshsecretkeyrefreshsecretkey
    access-token-validity-in-seconds: 600000  # 10 min: 10min * 60 sec * 1000 millisecond
    refresh-token-validity-in-seconds: 10800000  # 10800000 3 hours: 3hours * 60min * 60sec * 1000 millisecond
```
application-jwt.yml 파일의 구성이 위와 같다고 가정해보자.  
JWT를 구현하는 Service 단에서 위에서 정의한 값을 읽어와야 할 것이다.

<br>

#### application.yml에서 include를 통해 프로파일 포함시키기
우선 application.yml에서 spring.profiles.include를 통해 해당 프로파일(application-jwt.yml)을 포함시켜준다. 
``` java
spring:
    profiles:
        include: jwt
```

<br>

#### 프로덕션 코드에서 값 불러오기
프로덕션 코드 *(ex 서비스 클래스)* 에서 값을 가져오는 것은 간단하다.  
바로 ***@Value*** 어노테이션을 사용하면 된다.
``` java
...
public class JwtTokenProvider {

    @Value("${jwt.secret}")
    private String secretKey;
    @Value("${jwt.refresh-secret}")
    private String refreshSecretKey;
    @Value("${jwt.access-token-validity-in-seconds}")
    private Long accessTokenValidityInMilliseconds;
    @Value("${jwt.refresh-token-validity-in-seconds}")
    private Long refreshTokenValidityInMilliseconds;

...
```
위와 같이 @Value 어노테이션 안에 어떤 값을 가져오고 싶은지 작성하면 되는데, **${profile 이름.key 값}** 을 통해 가져올 수 있다.  
이렇게 값을 가져오면 외부에서 코드를 받았다고 하더라도, application.yml 파일이 없는 이상 실제로 어떤 값이 들어가는지 알 수 없다.


만약 변수로 선언하는 것이 아니라, 값으로 바로 넣고 싶다면 값이 들어가야 할 자리에 @Value()를 통해 값을 집어넣어주면 된다.

```java
public TokenProvider(
    @Value("${jwt.secret}") String secret,
    @Value("${jwt.token-validity-in-seconds}") long tokenValidityInSeconds) {
        this.secret = secret;
        this.tokenValidityInMilliseconds = tokenValidityInSeconds * 1000;
}
```


<br>


#### Test 코드에서 값 불러오기
테스트 코드에서 값을 가져오는 것도 비슷하나, 어노테이션 하나를 추가적으로 작성해야 한다.

``` java
...
@ActiveProfiles({"jwt"})
public class JwtTokenProviderServiceTest {
    @Value("${jwt.secret}")
    String secretKey;
    @Value("${jwt.refresh-secret}")
    String refreshSecretKey;
    @Value("${jwt.refresh-token-validity-in-seconds}")
    Long refreshTokenValidityInMilliseconds;
    ...
```

***@ActiveProfiles({"profile 이름"})*** 어노테이션을 추가해서, 해당 프로파일을 이 테스트 클래스에서 사용할 것임을 알려주어야 한다.  
그 외에는 위에서 했던 방법과 동일하다.

