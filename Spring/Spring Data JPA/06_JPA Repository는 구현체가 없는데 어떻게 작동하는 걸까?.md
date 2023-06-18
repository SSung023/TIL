## JPA Repository는 구현체가 없는데 어떻게 작동하는걸까?

> Spring Data JPA를 사용하다보면 드는 의문점이 하나 있다.  
> 바로 구현체에 관한 부분이다. 
> 인터페이스만 선언하는데 어떻게 작동하는 걸까?
 
<br>

Spring Data JPA를 사용하면 Repository 클래스를 생성할 때에 밑과 같이 "Interface"를 생성한다.
```
public interface MemberRepository extends JpaRepository<Member, Long> { }
```
JpaRepository라는 인터페이스를 상속받고, 찾고자하는 클래스 타입과 PK의 타입을 전달하면 
기본적인 메서드(ex: save, findAll, findById...)를 따로 작성하지 않아도 바로 사용할 수 있다.

> 인터페이스는 구현체를 필요로 하는데, JPA Repository의 경우에는 인터페이스의 구현체를 따로 생성하지 않는데 어떻게 작동하는걸까?

JPA 리포지토리는 @Autowired를 통해 의존성을 주입받는데, 우리는 이 때 JpaRepository를 상속받은 인터페이스를 넣어준다.  
그렇다면 의존성을 주입받은 객체에는 어떤 것이 들어가있을까?

테스트 코드에서 .getClass() 메서드를 통해 어떤 것이 들어가있는지 확인해보자.
``` java
@SpringBootTest
@Transactional
@Slf4j
class MemberRepositoryTest {
    @Autowired MemberRepository memberRepository;

    @Test
    @DisplayName("JPA 인터페이스 테스트")
    public void interfaceTest(){
        log.info(String.valueOf(memberRepository.getClass()));
    }
}
```
```
2023-06-18 10:28:29.682  INFO 84446 --- [           main] c.d.repository.MemberRepositoryTest      : class jdk.proxy2.$Proxy119
```
테스트 코드를 돌려본 결과, Spring Data JPA가 Proxy 객체를 만들어 의존성 주입했다는 것을 알 수 있다.  
인터페이스 구현체를 개발자가 만든 것이 아니라, Spring Data JPA가 만든 것이다.  

어플리케이션 로딩 시점에 Spring Data JPA와 관련된 인터페이스가 있으면, 개발자가 구현체를 만들지 않아도 Proxy 객체를 만들어서 Injection을 한다.




