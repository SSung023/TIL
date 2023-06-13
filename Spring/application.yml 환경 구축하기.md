## application.yml(properties) 환경 구축하기

> 스프링 이니셜라이져(https://start.spring.io) 를 통해서 스프링 부트 프로젝트를 생성하면  
> application.properties(혹은 application.yml)이 자동으로 같이 생기는 것을 확인할 수 있다.  
> 
> 스프링 부트 어플리케이션을 실행하면 application.yml 파일에서 필요한 설정을 자동으로 읽어들이는데,  
> 실행 환경에 따라 어떤 application 파일을 읽고 싶은지 설정하는 방법을 알아보자. 

<br>

### application.properties vs application.yml
스프링 부트를 처음 공부할 때에 들었던 의문점이 있었다.  
스프링 부트 프로젝트를 처음 생성하면 application 설정 파일의 확장자가 기본적으로 .properties로 설정이 되어 있는데,  
왜 대부분의 사람들은 application.yml 형태를 사용하는지가 너무 궁금했다.

결과적으로는 표현 방식의 차이인데, .yml 형식이 가독성이 더 좋고, 설정들을 관리하기가 더 편하다.

#### application.properties 예시
``` 
spring.profiles.include = dev
spring.datasource.url = jdbc:mysql://localhost:3306/DBName?serverTimezone=UTC&characterEncoding=UTF-8
spring.datasource.driverClassName = com.mysql.cj.jdbc.Driver
spring.datasource.username = guest
spring.datasource.password = a

```

#### applcation.yml 예시
``` 
spring:
  profiles:
    include: dev
  datasource:
    url: jdbc:mysql://localhost:3306/DBName?serverTimezone=UTC&characterEncoding=UTF-8
    driverClassName: com.mysql.cj.jdbc.Driver
    username: guest
    password: a

```

위 두 예시는 같은 환경을 설정하는 코드이지만 가독성에서 차이가 나는 것을 바로 볼 수 있다.  
.properties 방식은 key-value 형식을 띄고 있고, .yml 방식은 엔터와 들여쓰기를 통해 계층적 구조를 나타낸다.  

프로젝트 몇몇개를 진행하면서 .yml 방식이 더 편리하다고 느꼈는데,
1. 같은 종류의 설정을 작성할 때에 편리하다.  
.properties에서는 특정 환경에 대한 설정 값을 여러 번 작성할 때에 무조건 spring.부터 시작하여 key를 작성해주어야 한다.  
반면 .yml 에서는 줄바꿈과 들여쓰기를 통해 계층을 구분하기 때문에 위와 같이 중복 작성을 피할 수 있다.
2. 가독성이 좋다.  
   1번의 작성의 편리함은 가독성으로 이어진다. properties와 yml 예시를 비교해보면 느껴질 것이다.

<br>

***

### application.yml의 환경 구축 필요성

> 프로젝트의 실행 환경에 따라 이 application.yml의 환경을 따로 구축이 필요할 때가 있다.  
> 제일 최근에 진행했던 프로젝트에서 있었던 일이다.  
> ChatPrompt 프로젝트 때, 배포가 되어 있는 상태에서 다른 기능을 개발해야 하는 상황이었다.  
> 개발 코드를 실행해서 어떻게 동작하는지 확인해보고 싶은데 같은 환경(대표적으로 DB)를 사용하면 사용자들이 사용하고 있는 환경에 영향을 미칠 것이다.
> 
> 따라서 상황에 따라 다른 환경을 사용할 수 있도록 application.yml 파일을 설정해야 한다.  

<br>

***

### application.yml 환경 구축하기

application.yml의 기본 위치는 ***src/main/resources*** 에 위치하고, 스프링이 가동되면서 해당 위치에서 application.yml을 읽어들인다.  

여러 개의 Profile(프로파일)을 만들어놓고 상황에 따라 원하는 프로파일을 적용하게 할 수도 있다.  
파일의 이름을 밑과 같이 적용하면 된다.  *ex) application-dev.yml*
```
application-{profile}.yml
```
spring에서 config 파일을 읽어들이는 순서가 application.yml → application-{profile}.yml 이기 때문에  
모든 환경에서 공통으로 들어가야 할 설정들은 application.yml에 적용하면 된다.

application-{profile}.yml 파일을 만들고 내용까지 작성했다면 이 파일을 어떻게 적용할까?  
application.yml 파일에 밑과 같이 spring.profiles.active 설정을 통해 적용할 수 있다.

``` 
spring:
  profiles:
    active: dev
```

<br>

***예시 전체***
``` 
# application.yml
spring:
  profiles:
    default: dev
  jpa:
    properties:
      hibernate:
        format_sql: true
        show_sql: true
  main:
    allow-bean-definition-overriding: true
```
``` 
# application-dev.yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/DBname?serverTimezone=UTC&characterEncoding=UTF-8 # 로컬 환경
    username: guest
    password: a
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    defer-datasource-initialization: true
    ddl-auto: create
```
``` 
# application-prod.yml
spring:
  datasource:
    url: jdbc:mysql://ip 주소:3306/DBname?allowPublicKeyRetrieval=true&useSSL=false&serverTimezone=Asia/Seoul&characterEncoding=UTF-8 # AWS EC2 환경
    username: guest
    password: a
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    defer-datasource-initialization: true
    hibernate:
      ddl-auto: none
```




