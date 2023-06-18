## 쿼리 메서드 기능 - @Query

> Spring Data JPA에는 "쿼리 메서드"라는 기능을 제공하고 있다.  
> DB로 날리는 쿼리를 손쉽게 작성할 수 있는 기능인데, 3가지 방법으로 제공하고 있다.
> 1. 메서드 이름으로 쿼리 생성
> 2. 메서드 이름으로 JPA NamedQuery 호출
> 3. @Query 어노테이션을 이용하여 Repository 인터페이스에서 쿼리 정의
> 

<br>

Spring Data JPA에서 제공하는 쿼리 메서드 기능들 중 3번째 방법인 @Query 어노테이션을 이용하는 방법을 가장 많이 사용한다.  

1번의 경우에는 조건이 얼마 되지 않는 기능인 경우 매우 편하게 사용할 수 있지만,  
조건이 길어지는 경우 메서드의 이름도 자연스럽게 길어져 가독성/유지보수성이 떨어지게 된다.  
밑과 같이 조건이 세 개가 되는 경우에는 메서드의 이름이 어마무지하게 불어나는 것(...)을 볼 수 있다.
``` java
Member findByNameAndAddressAndAgeGreaterThan(String name, String address, int age);
```

2번의 경우에는 jpql을 직접 작성하고, 메서드 이름도 직접 설정하기 때문에 1번의 단점을 어느정도 커버하지만 3번에 비해 코드가 길어지고 복잡해진다.

마지막으로 3번의 경우에는 리포지토리의 메서드의 바로 위에 jpql을 작성하기 때문에 가독성과 유지보수성 모두 좋다.  


### @Query?
``` java
public interface MemberRepository extends JpaRepository<Member, Long> {
    @Query("select m from Member m where m.username= :username and m.age = :age")
    List<Member> findUserByInfo(@Param("username") String username, @Param("age") int age);
}
```
위와 같이 리포지토리 메서드 위에 @Query 어노테이션 안에 jpql 정적 쿼리를 직접 작성하면 된다.  
즉, 이름이 없는 NamedQuery라고 보면 되는 것이다.

> jpql Parameter를 통해 jpql 쿼리 안에 들어갈 값 전달하기!    
>  jpql 쿼리 안에 ***:{Param name}*** 을 통해 파라미터의 이름을 정의할 수 있고,  
>  메서드 Parameter의 자리에 ***@Param***을 이용하여 값을 매핑할 수 있다.

실무에서는 메소드 이름으로 쿼리 생성 기능은 파라미터가 증가하면 메서드 이름이 매우 지저분해지기 때문에 @Query 기능을 자주 사용한다고 한다.



