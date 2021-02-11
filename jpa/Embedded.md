# JPA Embedded, Embeddable

## 목차

[1. 특징](#특징)

[2. 장점](#장점)

[3. 사용법](#사용법)

[4. 주의점](#주의점)

## 특징

* 공통적인 속성을 하나의 Value Object 로 분리하여 매핑하는 방법
* 새로운 값 타입을 직접 정의할 수 있음
  * int, String 과 같은 값 타입
* 생명주기가 엔티티에 의존하는 경우

## 장점

* 재사용
* 높은 응집도
  * 관련된 역할끼리 모아놓은 클래스
* 테이블을 객체로 세밀하게 매핑 가능

## 사용법

#### Annotation
* @Embeddable
  * 값 타입을 정의하는 곳에 표시
  * 기본 생성자 필수
  * 내부에 Embedded 타입, Entity 를 포함할 수 있음
* @Embedded
  * 값 타입을 사용하는 곳에 표시
* @AttributeOverride, @AttributeOverrides
  * 컬럼 명 속성을 재정의

### AS-IS

```java
@Entity
public class Member {
	@Id
	@GeneratedValue
	@Column(name = "member_id")
	private Long id;

	private String city;
	private String street;
	private String zipcode;
}
```

### TO-BE

```java
@Entity
public class Member {
	@Id
	@GeneratedValue
	@Column(name = "member_id")
	private Long id;

	@Embedded
	private Address homeAddress;
}

@Embeddable
@NoArgsConstructor(access = PROTECTED)
public class Address {
	private String city;
	private String street;
	private String zipcode;
}
```
  
## 주의점
* 하나의 영속성 컨텍스트 범위에서... 두 엔티티 객체가 동일한 임베디드 값을 가진다고 하여 하나의 임베디드 객체를 공유하여 사용한다면 side-effect 가 발생할 수 있다.
  * 동일한 값의 임베디드 객체가 필요하다면, 객체를 복사해서 새로운 객체로 만들어야 한다.
  * `불변 객체` 로 만든다.
    * 불변 객체로 만들어 side-effect 를 차단한다.
    * JPA 는 field 를 final 로 선언할 수 없지만, 생성 이후 수정이 불가능하도록 setter 를 제공하지 않고, 무분별한 getter 를 제공하지 않도록 개발해야 한다. getter 처럼 객체를 제공해야하는 경우는 객체를 즉시 반환하지 않고 새로운 객체로 값만 복사하여 생성한 후 리턴하면 불변 객체를 구현할 수 있다.
      * cf) `Effective Java 50. 적시에 방어적 복사본을 만들라`
* 값 타입의 동등성 비교
  * a.equals(b) 를 사용해서 동등성 비교
  * equals() 메서드 재정의
    * Embedded 는 자체적인 식별자가 없기 때문에 주로 모든 필드의 값을 비교
