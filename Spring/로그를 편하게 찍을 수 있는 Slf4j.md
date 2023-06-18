## 로그를 편하게 찍을 수 있는 Slf4j

> 개발을 하다보면 로그를 찍어서 확인해야 할 때가 간간히 있다.  
> Java에서의 System.out.println도 있다.  
> 하지만 해당 기능은 상황별, 에러별에 따라서 출력을 제어하지 못하고 무조건 출력이 되기 때문에 권장하지 않는다.

<br>

### Slf4j란?
Simgle Log Facade For Java의 약자로써, Logging framework의 추상화 라이브러리이다.
Lombok와 결합하여 더욱 편리하게 사용할 수 있다. 


<br>

### Gradle 의존성 추가
Lombok(롬복)과 같이 사용하려면 build.gradle 파일에 Lombok 의존성을 추가해야 한다.  
``` java
dependencies {
    ...
    compileOnly 'org.projectlombok:lombok'
    
    annotationProcessor 'org.projectlombok:lombok'
    
    testCompileOnly 'org.projectlombok:lombok'
    testAnnotationProcessor 'org.projectlombok:lombok'
    ...
}
```

기본적으로 롬복 의존성은 compileOnly와 annotationProcessor만 추가해주어도 되지만, 테스트 코드를 작성하면서 로그를 찍어야 할 일이 종종 있기 때문에,  
testCompileOnly와 testAnnotationProcessor도 같이 넣어주면 테스트 코드에서도 로그를 간편하게 찍을 수 있다.

<br>

### Slf4j를 이용한 로그 찍기
Lombok을 사용하지 않고 Slf4j를 통해 로그를 찍으려면 객체를 생성해주어야 한다.
```
Logger logger = LoggerFactory.getLogger(SpringApplication.class);
```
와 같이 logger 객체를 생성해야, logger.info()와 같은 로거 메서드를 작성할 수 있다.  
하지만 매 번 객체를 생성하는 것도 번거로운 일인데, 이러한 것을 Lombok을 통해 극복할 수 있다.

``` java
...
@Slf4j
public class HomeService {
    public void home(){
        log.info("info 로그!");
    }
}
```
위와 같이 클래스 상위에 @Slf4j 어노테이션을 작성하면 알아서 logger 객체를 생성해주기 때문에  
우리는 log.info()와 같은 메서드를 호출하기만 하면 된다!

<br>

### Log 레벨
찍을 수 있는 Log 레벨에는 총 5가지가 있다.
- trace
- debug
- info
- warn
- error

밑으로 갈수록 더욱 심각한 정보이며, Level을 설정하여 해당 레벨 이상의 로그만 찍히게 할 수도 있다.
> 예를 들어 logging level이 info로 설정되어 있다고 한다면, info 이상의 레벨인 info, warn, error가 출력된다.

<br>

### Logging 간단한 설정
Logging에 대한 설정은 logback.xml을 통해 세부적으로 설정하는 것이 좋지만,  
프로젝트의 규모가 작은 경우에는 간단한 설정을 application.yml에서 할 수 있다.
```
logging:
  level:
    root: info
  file:
    path: /{file path}
```
간단하게 logging level과 log 파일이 저장될 곳을 설정해보자.  
logging.level.root의 값을 trace/debug/info/warn/error 중 하나를 선택하여 레벨을 설정할 수 있다.  
logging.file.path의 값에 log 파일이 저장될 경로를 설정하면, 해당 위치에 log가 찍힌 파일이 있는 것을 확인할 수 있다.


