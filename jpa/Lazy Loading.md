## 목차
[1. Lazy Loading](#Lazy-Loading)

[2. Default Constructor & HibernateProxy](#Default-Constructor-&-HibernateProxy)

[3. N + 1](#N-+-1)

## Lazy Loading
* JPA 에서는 연관 관계 객체에 대해 두 가지 로딩 방법을 제공한다.
* Eager
  * 모든 연관 관계 객체를 즉시 조회하는 방법
* Lazy
  * 그 객체를 실제 사용하는 시점에 데이터를 조회하는 방법
  ```java
  @Entity
  @NoArgsConstructor(access = PROTECTED)
  public class Member {
    // ...
    
    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "team_no", referencedColumnName = "no")
    private Team team;
    
    // ...
  }
  
  @Entity
  @NoArgsConstructor(access = PROTECTED)
  public class Team {
    // ...
  }
  ```
  * Example
    * Member 에서 Team 에 대한 ManyToOne 연관 관계를 Lazy fetch type 으로 설정하고 있다.

## Default Constructor & HibernateProxy
* Fetch type 을 Lazy 로 지정하면 Hibernate 에서 해당 객체는 HibernateProxy 객체라는 가짜 객체로 생성된다.  
* Example
  * 위의 예제에서 Member 를 조회했을때 Team 객체를 디버깅해보면 `HibernateProxy` 객체로 보여진다. Team 객체를 감싼 가짜 HibernateProxy 객체이다. 정확히 Team 을 상속받아 구현된 Proxy 객체.
    * Hibernate 가 reflection 을 사용하여 Team 을 상속받는 Proxy 객체를 생성한다. 그래서 JPA 에서 Entity 로 사용하는 class 에 final 을 선언하면 안되고, Arguments 가 없는 디폴트 생성자가 필수이다.
    * Team 객체를 조회해보면 HibernateProxy 타입을 확인할 수 있고 전부 null 값을 가지고 있다.
* 정리해보면, Hibernate 의 Proxy 객체 특징은...
  1. Entity 클래스를 상속받아 생성
  2. reflection 으로 생성되어 Default Constructor 가 필수
  3. HibernateProxy 객체가 실제 target 클래스를 보관
  4. 이상... JPA 스펙은 아니고, Hibernate 구현체의 특징

#### HibernateProxy 객체와 주의점
1. 타입 체크
  * 프록시 객체는 Entity 클래스를 상속받아 생성된다. 그래서 타입 체크시 주의해야 한다.
    * Example
      * (위의 Member, Team 코드 참고)
      ```java
      Member member = findById(1L);
      assertThat(member.getClass() == Member.class).isTrue();
      assertThat(member instanceof Member).isTrue();

      assertThat(member.getTeam().getClass() == Team.class).isFalse();
      assertThat(member.getTeam() instanceof Team).isTrue();
      ```
    * 
2. 영속성 컨텍스트와 강결합 - LazyInitializationException
      
## N + 1
* N + 1 문제는 Eager 타입인 경우도 발생할 수 있다. 다만 발생하는 이유는 약간 다르다.

#### Eager loading N + 1

#### Lazy loading N + 1


