### 플러시 flush()란?

> *플러시 flush()는 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영한다.*

<br>

### flush 과정

✅ **영속성 컨텍스트를 플러시하는 방법**

1. em.flush()를 직접 호출

   메서드를 직접 호출해서 영속성 컨텍스트를 강제로 flush하는 방법.

   Test나 다른 프레임워크와 JPA를 사용할 때를 제외하고 거의 사용 X

2. 트랜잭션 커밋 시, flush 자동 호출

   JPA는 변경 내용이 DB에 반영되게 하기 위해, 트랜잭션을 commit할 때 flush를 자동으로 호출한다.

3. JPQL 쿼리 실행 시, flush 자동 호출

   객체지향 쿼리를 호출 할 때에도 flush가 실행된다.

```java
query = em.createQuery("select m from Member m", Member.class);
List<Member> members = query.getResultList();
```

<br>

### flush 모드 옵션

✅ **FlushModeType.AUTO**

commit이나 query를 실행할 때 flush가 일어남.  
플러시 모드의 디폴트 값 → 트랜잭션 커밋이나 쿼리 실행 시에 플러시를 자동 호출

✅ **FlushModeType.COMMIT**

성능 최적화를 위해 사용할 수 있음


>💡 ***영속성 컨텍스트의 변경내용을 데이터베이스에 동기화하는 것이 flush이다.***