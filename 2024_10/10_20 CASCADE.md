## **CASCADE 설정**

보통 일반적으로 각 엔티티 간 연관관계를 설정할 경우 영속성 컨텍스트에 해당 엔티티를 등록해주는 작업을 

해줘야한다. 

```java
EntityManager em = new EntityManger();
em.persist();
```

스프링 프레임워크 환경에서는 위와 같이 해당 **엔티티를 사용하기 전에 영속성 컨텍스트에 등록을 시켜줘야한다**.

하지만 스프링 부트 환경에서도 사용해도 되지만 **CRUD 작업을 수행하면서 자동으로 등록을 시켜준다**.

```java
Member member = Member.builder(
	.name("미키");
	.age(24);
)
member.save();
// 이외에도 다음과 같은 작업에도 해당
member.deleteByID(1L);
member.findById(1L);
```

예시로 위 코드처럼 해당 객체에 대한 값을 처리할 경우 자동으로 영속성 컨텍스트에 등록 및 작업을 처리해준다.

스프링 부트 환경에서 자동으로 영속성 컨텍스트에 등록을 시켜주는 또 다른 방법으로 **CASCADE 설정**이 있다.

```java
public class Member(
	
	@OneToMany(mappedBy="member", cascade=CascadeType.ALL)
	private List<OrderItems> orderItems = new ArrayList<OrderItems>();
)

public class OrderItems(
 @ManyToOne
 private Member member;
)
```

이와 같은 연관관계일 때 연관관계의 **주체**가 되는 엔티티에 **CascadeType.ALL을 설정해주게 되면 해당 엔티티들을 자동으로 영속성 전이를 설정해주게 된다**.

해당 연관된 엔티티의 객체를 생성할 경우 자동으로 영속성에 등록 및 동작 처리를 해준다.

 → 즉, 스프링에서 **em.persist()**를 **cascade=CascadeType.ALL**로 처리할 수 있다.