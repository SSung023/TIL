### 직접 할당 전략 : @Id

기본 키를 직접 할당하려면 @Id를 통해 매핑하면 된다.

```java
@Id
@Column(name = "id")
private String userid;
```

<br>

### IDENTITY 전략

>IDENTITY는 *PK 생성을 데이터베이스에 위임*하는 전략이다.

MySQL, SQL Server, DB2, PostgreSQL에서 주로 사용한다.

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

✅ ***@Id*** 와 ***@GeneratedValue*** 어노테이션 이용  
✅ 이 전략에서는 트랜잭션을 지원하는 ‘쓰기 지연’이 동작하지 않는다.



<br>

### AUTO 전략 ✨

> 선택한 데이터베이스 방언에 따라 IDNETITY, SEQUENCE, TABLE 전략 중 하나 선택해준다.
>

```java
@Id
@GeneratedValue
private Long id;
```

✅ ***@GeneratedValue***의 기본 값이 *GeneratedType.AUTO*이다.  
✅ 장점 - 데이터베이스를 변경해도 코드를 수정할 필요 없다.