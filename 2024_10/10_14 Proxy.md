# Proxy란

### **즉시, 지연 로딩**

JPA에서 식별자로 엔티티를 조회할 때는 즉시로딩(EAGER), 지연로딩(LAZY)가 존재한다.

즉시로딩을 하게 될 경우, **연관된 테이블의 정보까지 한꺼번에 가져오게 된다**.

- 비효율적
- 한번에 가져오면서 N+1 문제 발생
- 성능 문제가 다분할 수 있음

지연로딩은 해당 객체를 생성할 때가 아닌 **해당 객체 값에 접근할 경우**에 정보를 가져온다.

이때 해당 객체가 생성되고 **메서드를 실행하기 전까지** **프록시(가짜 객체)가 생성되어 시간을 지연**시킨다.

- 즉시 로딩 예제
    
    ```java
    import javax.persistence.*;
    import lombok.Getter;
    import lombok.Setter;
    
    @Entity
    @Getter
    @Setter
    public class User {
    
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
    
        private String name;
    
        // 즉시 로딩 설정
        @ManyToOne(fetch = FetchType.EAGER)
        private Team team;
    }
    
    @Entity
    @Getter
    @Setter
    public class Team {
    
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
    
        private String name;
    }
    
    ```
    
    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;
    
    @Service
    public class UserService {
    
        @Autowired
        private UserRepository userRepository;
    
        @Transactional
        public void loadUser() {
            // User 엔티티를 ID로 조회
            User user = userRepository.findById(1L).get();
    
            // 즉시 로딩: Team도 함께 로드됨 (추가 쿼리 없이 바로 사용 가능)
            System.out.println("User's Team: " + user.getTeam().getName());
        }
    }
    ```
    
- 지연 로딩 예제
    
    ```java
    @Entity
    @Getter
    @Setter
    public class User {
    
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
    
        private String name;
    
        // 지연 로딩 설정
        @ManyToOne(fetch = FetchType.LAZY)
        private Team team;
    }
    
    ```
    
    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;
    
    @Service
    public class UserService {
    
        @Autowired
        private UserRepository userRepository;
    
        @Transactional
        public void loadUser() {
            // User 엔티티를 ID로 조회
            User user = userRepository.findById(1L).get();
    
            // 지연 로딩: Team을 사용하는 순간에 추가 쿼리가 실행
            System.out.println("User's Team: " + user.getTeam().getName());
        }
    }
    
    ```
    

### **프록시 초기화**

프록시를 통해 엔티티에 실제 메서드를 통해 DB에 접근할 때 실제 엔티티가 초기화(호출)가 

되어있지 않다면, 영속성 컨텍스트에 **실제 엔티티 생성을 요청**한다(이것을 **초기화** 라고 한다.)

- 실제 엔티티 생성이 된다면, 이것을 프록시가 아닌 실제 객체를 통해 사용한다.
    - 프록시 객체는 실제 엔티티를 상속받아서 만들어졌음
    - 실제 초기화가 된다는 건 어떠한 메서드나 값이 할당 될 경우 실제 엔티티를 영속성 컨텍스트에 등록시켜서 사용한다.

### **프록시의 특징**

1. 최초에 메서드를 호출할때만 초기화된다.
    
    → 영속성 컨텍스트를 통해 실제 DB에 접근할 수 있게 된다.
    
2. 원본 엔티티를 상속받아서 만들어졌기 때문에 타입 체크 시 주의해야한다.
3. 프록시를 호출했을 때 실제 엔티티가 존재한다면 반환 값으로 실제 엔티티를 반환한다.
4. 초기화는 영속성 컨텍스트의 도움이 있어야 가능하다.

엔티티를 프록시로 조회할 때 **PK값을 파라미터로 전달**하고 **프록시**는 이 값을 **보관**한다.

JPA 조인 전략

@JoinColumn(nullable = true) : NULL 허용, 외부 조인 사용

@JoinColumn(nullable = false) : NULL 허용 X, 내부 조인 사용

또는 @ManyToOne.optional = false 설정 시 내부 조인 사용