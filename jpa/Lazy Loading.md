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
  * 위의 예제에서 Member 를 조회했을때 Team 객체를 디버깅해보면 **HibernateProxy** 객체로 보여진다. Team 객체를 감싼 가짜 HibernateProxy 객체이다. 정확히 Team 을 상속받아 구현된 Proxy 객체.
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
      * 아래 케이스는 전부 통과하는 코드
      ```java
      @Test
      void typeCheck() {
          Member member = repository.findById(1L);
          assertThat(member.getClass() == Member.class).isTrue();
          assertThat(member instanceof Member).isTrue();

          assertThat(member.getTeam().getClass() == Team.class).isFalse();
          assertThat(member.getTeam() instanceof Team).isTrue();
      }
      ```
      * 세 번째 검증에서 member.getTeam().getClass() 과 Team.class 는 다른 타입으로 확인되어 isFalse() 가 통과된다. member.getTeam() 을 확인해보면 진짜 Team 객체가 아니라 Team 클래스를 상속받은 HibernateProxy 객체이므로 == 연산자로 비교할 수 없다. 그래서 타입 체크가 필요하다면 instanceof 를 사용해야한다.
2. 영속성 컨텍스트와 강결합 - org.hibernate.LazyInitializationException
  * HibernateProxy 객체도 영속성 컨텍스트에서 관리하는 상태인 **영속 상태**에서만 프록시 초기화가 가능하다. 아직 초기화되지 않은 프록시 객체를 트랜잭션이 종료되고 Hibernate Session 이 종료된 상태에서 getter 등으로 접근한다면 LazyInitializationException 이 발생한다.
    * (참고로 이 예외도 Hibernate 에서 발생시키는 예외이므로 JPA 표준이 아니고 다른 구현체는 다르게 동작할 수 도 있다.)

## N + 1
* N + 1 문제는 Eager 타입인 경우도 발생할 수 있다. 다만 발생하는 이유는 약간 다르다.

#### Eager loading N + 1
* 기본적으로 Fetch type 을 Eager 로 설정하면 어떤식으로 Query 가 생성될지 예측하기 힘들기 때문에 Lazy 를 사용하고 즉시 로딩이 필요한 경우는 fetch join 으로 해결한다.
* 그래도 Eager 을 사용하는 경우가 있다면, 이때도 N + 1 이 발생할 수 있으니 주의가 필요하다.
* 기본적으로 한번에 모든 데이터를 조회할 것 같지만, JPQL 을 사용하는 경우는 Eager 이 적용되지 않고 N + 1 문제가 발생한다.
* 결국 Lazy 타입으로 지정하고 fetch join 을 사용하는 것이 좋다.

#### Lazy loading N + 1
* 위에 작성한 예제에서 Member 리스트를 조회한다면 N + 1 문제가 발생할 것이다. Member 리스트는 한 번에 조회했어도 Member 엔티티의 필드로 존재하는 Lazy 필드인 Team 객체는 아직 조회되지 않은 상태이기 때문이다.
  ```java
  List<Member> members = repository.findAll();
  for (Member member : members) {
    member.getTeam(); // Team 을 조회하는 순간 쿼리가 매번 호출된다.
  }
  ```
  * 위 예제에서 쿼리는 총 N + 1 개가 호출된다.
    * 1개 쿼리 - Member 리스트를 조회, repository.findAll()
    * N개 쿼리 - 각 Member 별 Team 을 조회
  * 이 문제를 해결하기 위해서는 fetch join 을 사용해야한다.

