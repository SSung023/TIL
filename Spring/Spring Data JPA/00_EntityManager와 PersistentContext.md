### EntityManager란?
EntityManager는 Entity를 저장, 수정, 삭제, 조회하는 등 Entity와 관련된 모든 일을 처리한다.

>💡 여기서 Entity는 Database에 저장되는 데이터 집합을 이야기하는데, JPA에서의 클래스(객체)가 Entity로 취급된다.

✅ **EntityManager는 어떻게 설정되며 어떤 특징이 있을까?**

1. 엔티티 매니터를 생성하는 비용은 아주 크다.  
   따라서 엔티티 매니저 팩토리는 어플리케이션 전체에서 한 번만 생성하고 공유해서 사용해야 한다!

2. 앤티티 매니저 생성  
   JPA의 기능 대부분은 EntityManager가 제공  
→ EntityManager를 사용하여 Entity를 DB에 등록/수정/삭제/조회할 수 있다.  
   EntityManager는 Thread 간 공유하거나 재사용하면 안된다!!!

3. 종료  
   사용이 끝난 EntityManager와 EntityManagerFactory를 반드시 종료해야 한다.

✅ Transaction 관리  
JPA를 사용하면 항상 트랜잭션 안에서 데이터를 변경해야 한다.  
트랜잭션 API를 사용하여 비즈니스 로직이 정상 동작하면 트랜잭션을 commit, 예외가 발생하면 트랜잭션 rollback을 한다.

<br>

***

### PersistentContext란?
> **PersistentContext(영속성 컨텍스트)란 *엔티티를 영구 저장하는 환경*이라는 뜻이다.**

영속성 컨텍스트는 *EntityManager를 생성할 때 **하나*** 만들어진다.


✅ 특징 3가지
1. **영속성 컨텍스트와 식별자 값**  
   영속성 컨텍스트는 *식별자 값(@Id)를 통해 엔티티를 구분*한다.
   따라서 영속 상태는 식별자 값이 반드시 있어야 한다.

2. **영속성 컨텍스트와 데이터베이스 저장**  
   영속성 컨텍스트에 엔티티를 저장하면, 데이터베이스에 언제 반영될까?

   *트랜잭션을 commit하는 순간* 데이터베이스에 반영되는데, 이것을 *플러시 flush*라고 한다.

3. **영속성 컨텍스트가 엔티티 관리 시의 장점**  
   1차 캐시, 동일성 보장, 트랜잭션을 지원하는 쓰기 지연, 변경 감지, 지연 로딩

<br>

***

### PersistentContext를 통한 Entity의 관리
### Entity 조회
✅ **1차 캐시**  
영속성 컨텍스트의 내부에 캐시를 가지고 있는데, 엔티티는 모두 이곳에 저장된다.  
@Id로 매핑한 식별자를 키로 사용하는 Map이 있다고 보면 된다.  
1차 캐시에 있는 엔티티는 아직 데이터베d이스에 저장되지 않은 상태이다.

✅ **엔티티 조회**  
`em.find(Class<T> entityClass, Object primaryKey);`

find() 호출 시 1차 캐시에서 엔티티를 찾고, 엔티티가 1차 캐시에 없으면 데이터 베이스에서 조회한다.

- **1차 캐시에서 엔티티 조회 : 식별자 값을 통해 엔티티 조회**

    ```java
    Member member = new Member("member1", "회원1");
    em.persist(member); // 1차 캐시에 저장
    
    Member findMember = em.find(Member.class, "member1"); // 1차 캐시에서 조회
    ```

- **데이터베이스에서 엔티티 조회 : 식별자 값을 통해 엔티티 조회**
    1. em.find(Member.class, “member2”);를 실행한 후, 1차 캐시에서 PK를 통해 엔티티를 조회
    2. member2가 1차 캐시에 없으므로 데이터베이스에서 조회
    3. 조회한 데이터로 member2 엔티티 생성 후, 1차 캐시에 저장(영속 상태)
    4. 조회한 엔티티 반환

✅ **영속 엔티티의 동일성 보장**  
`em.find(Member.Class, “member1”)`를 반복 호출해도, 영속성 컨텍스트는 1차 캐시에 있는 같은 엔티티 인스턴스를 반환한다.


### Entity 등록
✅  **쓰기 지연 transactional write-behind**  
트랜잭션을 커밋하기 전까지 *내부 쿼리 저장소에 INSERT SQL 문을 저장*해놨다가,  
트랜잭션을 *commit*할 때 모아둔 *쿼리를 데이터베이스에 보내는 것*을 말한다.

⇒ 1차 캐시에 엔티티를 저장하면서 내부 쿼리 저장소에 SQL문을 저장한다.  
⇒ 트랜잭션을 commit할 때 영속성 컨텍스트를 flush하여 변경 내용을 데이터베이스에 동기화.  
→ 구체적으로는 *쓰기 지연 SQL 저장소에 모인 쿼리를 DB에 전송*.

✅ **트랜잭션을 지원하는 쓰기 지연이 가능한 이유**  
commit 직전에만 데이터베이스에 SQL을 전달하면 되기 때문  
이 기능을 활용하면 성능을 최적화할 수 있다.

### Entity 수정
✅ **변경 감지**

> *엔티티의 변경사항을 데이터베이스에 자동으로 반영하는 기능*
>

**동작 방식**

JPA는 엔티티를 영속성 컨텍스트에 보관할 때, 최초의 상태를 복사하여 저장한다. 이를 *snapshot*이라고 하는데, *flush 시점*에 *snapshot과 엔티티를 비교*하여 변경된 엔티티를 찾는다.

이후 수정 쿼리를 생성하여 쓰기 지연 SQL 저장소에 보내고, 이를 다시 데이터베이스에 보내어 트랜잭션을 commit한다.

>💡 *변경 감지는 영속성 컨텍스트가 관리하는 영속 상태의 엔티티에만 적용된다.*

✅ **JPA의 기본 전략**

>💡 JPA의 기본 전략은 엔티티의 모든 필드를 업데이트하는 것이다

1. 모든 필드를 사용하면 수정쿼리가 항상 같다 → 로딩 시점에 수정 쿼리를 미리 생성해두고 재사용할 수 있다.
2. 데이터베이스에 동일한 쿼리를 보내면 DB는 이전에 한번 parsing된 쿼리를 재사용할 수 있다.


### Entity 삭제
### 

```java
Member member = em.find(Member.class, "memberA");
em.remove(member);
```

엔티티 삭제 쿼리를 쓰기 지연 SQL 저장소에 저장한 후, 트랜잭션을 커밋해서 flush를 호출하면
실제 데이터베이스에 삭제 쿼리를 전달한다.

em.remove를 호출하는 순간 영속성 컨텍스트에서 제거된다.
제거된 엔티티는 garbage collection의 대상이 되도록 두는 것이 좋다.
