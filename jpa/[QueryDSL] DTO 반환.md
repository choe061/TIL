## 목차
[1. DTO 반환](#DTO-반환)

[2. Projections](#Projections)

[3. QueryProjection Annotation](#QueryProjection-Annotation)

## DTO 반환
* Entity 대신 DTO 를 반환하는 방법이다.
* 아래는 Querydsl 을 사용하지 않고 DTO 를 반환하는 방법인데, join 에 제약이 있기 때문에 그냥 Querydsl 을 사용하는 것이 좋다.
* interface 기반
  1. interface 생성
  2. field 에 대한 getter 작성
  3. return type 을 interface 로 지정
  4. `Proxy 객체`가 반환되어 원하는 값만 받아올 수 있다.
    * (직접 정의한 interface 를 상속받는 Proxy 객체를 생성해서 반환하나보다.)
  ```java
  public interface UsernameOnly {
    String getUsername();
  }
  ```
* class 기반
  * 생성자의 파라미터 명을 보고 해당 필드만 쿼리하여 가져온다.
  ```java
  public class UsernameOnlyDTO {
    private final String username;

    public UsernameOnlyDTO(String username) {
      this.username = username;
    }

    // getter
  }
  ```
* 단점
  * 아래 예시는 Username 과 유저가 속한 Team name 을 조회
    * user 엔티티에서는 username 만 가져오는데,
    * team 은 엔티티 전부를 쿼리하여 가져온 후 name 만 리턴해준다.
  ```java
  public interface NestedClosedProjections {
    String getUsername();
    TeamInfo getTeam();

    interface TeamInfo {
      String getName();
    }
  }
  ```

## Projections
* Projections 은 세 가지 방법이 있다.
  1. [setter](#setter)
  2. [fields](#fields)
  3. [constructor](#constructor)
  4. [(추가) select 절의 sub query](#(추가)-select-절의-sub-query)

#### setter
* 기본 생성자 필수. NoArgsConstructor
* field 명이 동일해야함.
  * 이름이 다르다면, as() 를 사용

```java
public void findDTOBySetter() {
	queryFactory.select(Projections.bean(MemberDTO.class,
				member.username.as("name"),
				member.age))
			.from(member)
			.fetch();
}
```

#### fields
* Getter, Setter 가 없어도 된다.
```java
public void findDTOByField() {
	queryFactory.select(Projections.fields(MemberDTO.class,
					member.username,
					member.age))
			.from(member)
			.fetch();
}
```

#### constructor
* 생성자 parameter 의 순서가 동일해야 한다.
* 이름이 아닌 type 을 보고 들어간다.
```java
public void findDTOBySetter() {
	queryFactory.select(Projections.constructor(MemberDTO.class,
					// constructor 기반에서는 아래 member.username, .age 의 순서가 동일해야함.
					member.username,
					member.age))
			.from(member)
			.fetch();
}
```

#### (추가) select 절의 sub query
* select 절에 sub query 를 아래처럼 사용할 수 있다.
```java
public void findDTOBySetter() {
	queryFactory.select(Projections.fields(MemberDTO.class,
						member.username,
						ExpressionUtils.as(JPAExpressions.select(memberSub.age.max())
														 .from(memberSub), "age")))
			.from(member)
			.fetch();
}
```

## QueryProjection Annotation
* DTO 생성자에 `@QueryProjection` 을 붙이면 컴파일시 DTO 도 QClass 가 생성되어 해당 QClass 를 직접 사용하는 방법
```java
public class MemberDTO {
  private String name;
  
  @QueryProjection
  public MemberDTO(final String name) {
    this.name = name;
  }
}
```
```java
public void findMemberDTO() {
	queryFactory.select(new QMemberDTO(member.username))
			.from(member)
			.fetch();
}
```
#### 장점
* QueryProjection 으로 생성된 QClass 를 사용하면 컴파일 타임에 오류를 잡을 수 있다. 하지만 constructor 기반은 생성자 파라미터의 순서를 동일하게 지켜야하는데 컴파일 타임에 알 수 없다.
* 생성자 기반에서는 member.username, member.age 에서 member. 다른 데이터가 잘못 추가될 때 런타임에 알 수 있지만, new QMemberDTO() 로 작성하면 컴파일 타임에 오류를 알 수 있다.

#### 단점
* DTO 가 QueryDSL 기술에 의존적이게 된다.

#### 생각
* 그래도 컴파일 타임에 오류를 잡을 수 있다는 것에 큰 장점이 있기 때문에 trade-off 하여 사용하면 좋을 것 같다.
