# 자바와 JUnit을 활용한 실용주의 단위 테스트
* 좋았던 부분만 간단하게 요약

## 목차
[1. 단위 테스트의 기초](#단위-테스트의-기초)

[2. 빠른 암기법](#빠른-암기법)

[3. 더 큰 설계 그림과 리펙토링](#더-큰-설계-그림과-리펙토링)
  * [테스트 리펙토링](#테스트-리펙토링)
    * (이 책에서 가장 알고싶었던 부분)

## 단위 테스트의 기초
* AAA
  * 준비 : Arrange, Given
  * 실행 : Act, When
  * 단언 : Assert, Then
* Tip
  * 의도한 바로, 정상적으로 동작하는 테스트 코드인지 확인하기 위해 의도적으로 테스트에 실패해보는 것도 중요하다.
  * 어떤 테스트를 작성할 것인가? checklist
    * 조건문
    * 데이터가 null 또는 empty 로 들어올 여지가 있는지 확인
  * `메서드 → 테스트` 가 아니라 `동작 → 테스트` 를 통해 테스트 코드의 유지 보수성을 높이는 방법
* JUnit 은 각 테스트를 위해 각각 별도의 Test class 의 인스턴스를 생성한다.
* 예외 무시
  * 간혹 checked exception 을 처리해야 하는 경우가 있다. 하지만 긍정적인 테스트를 설계한다면 정말 예외적인 상황을 제외하고 예외가 발생하지 않을 것임을 알고 있다. 
  * 이런 경우 테스트 코드에 직접 try ~ catch 블록을 넣을 필요가 없다. 다시 method 밖으로 throws 하면 JUnit 이 알아서 처리해 준다.
  * 아주 드문 경우로 그 exception 이 발생한다고 해도 JUnit 은 예외를 잡아 테스트 실패가 아니라 테스트 오류로 보고한다.

## 빠른 암기법
* FIRST 속성
  * Fast
    * 테스트가 느리면 돌리기 싫어지고, 점차 테스트에 무관심해진다.
    * 외부 연동이 있다면, mock 으로 대체하여 테스트 수행시간을 빠르게 처리하자.
  * Isolation
    * 각 테스트는 독립적이여야 한다. 서로 영향을 받는다면 안된다.
    * JUnit 은 기본적으로 method 를 실행하기 위해 매번 class 를 생성한다.
  * Repeatable
    * 테스트는 매번 동일한 결과로 동일하게 성공해야 한다.
      * 시간의 경우 가변하는 값이기 때문에 반복 가능한 테스트를 힘들게 하는 요소이다. 아래 처럼 Clock 객체로 시간과 관련된 테스트를 쉽게 처리할 수 있다.
  * Self validating
    * 프로덕션 코드와 얽히면 안된다. Java 의 JUnit 처럼 별도의 테스트 코드를 위한 패키지 클래스 를 만들고 테스트 코드를 작성해야 한다.
  * Timely
    * 테스트 코드로 검증을 미룰수록 미래에 힘들어진다.
    * 리뷰 프로세스, tool 을 이용하여 테스트가 없을때 코드를 거부하는 자동화 도구를 사용하는 것도 매우 좋다. Java, Gradle 환경 에서는 jacoco 를 사용하여 테스트 커버리지를 확인하고 커버리지가 일정 % 이하인 경우 빌드 에러를 발생시킬 수 있다.
    
#### 무엇을 테스트할 것인가?
* `경계 조건`, 역 관계, 오류 조건, 성능에 대한 테스트가 필요하다.
* 경계 조건은 알기 힘든 작은 결함을 만들어낸다. 그래서 모든 경계 조건을 파악하고 테스트가 필요하다.
    
## 더 큰 설계 그림과 리펙토링

#### 리펙토링하기
* 리펙토링이란
  * 기존 기능을 유지하고 코드의 정상 동작을 보장하면서 코드 구조를 깔끔하게 변경하는 것
  * 단! 마음대로 코드 구조를 변경하는 것은 위험!
    * 적절한 보호 장치가 있는지! -> 보호 장치는 바로 `테스트 코드`!
  * private 메서드를 테스트하려는 충동 -> 클래스가 커졌다는 힌트 -> 리펙토링이 필요
  * 단위 테스트가 어려운 것 같다? -> 리펙토링이 필요하다는 힌트
* 단위 테스트의 유지 보수 비용
  * 물론 단위 테스트를 유지하기 위한 비용은 있다. 보통 설계나 코드 품질이 낮을수록 단위 테스트의 유지 보수 비용도 증가한다.
  * 그럼에도 불구하고 테스트 코드가 존재함으로 인해 돌아오는 가치가 훨씬 크기 때문에 단위 테스트를 유지 보수하는 비용을 받아들이자.
    * 돌아오는 가치
      * 다른 코드가 깨질 것을 걱정하지 않으면서 코드를 변경할 수 있는 점
      * 코드가 어떻게 동작하는지 파악하는데 도움을 주는 점
      * 다양한 예외 케이스에 대해 파악하기 쉬워지는 점

#### Mock 객체 사용
* HTTP 통신이 필요하거나 Database 연동이 필요한 경우 -> Application 의 통제 밖 대상 -> 다른 코드와 분리
* 번거로운 동작을 stub 으로 대체
  * 테스트 용도로 하드 코딩한 값을 반환하는 구현체를 stub 이라고 한다.
  * 테스트 코드를 작성할 때 프로덕션 구현 대신 stub 을 사용할 수 있게 하는 방법은 의존성 주입!
    * 아래 코드처럼 Http 를 주입받아서 사용한다면, 테스트 코드에서는 하드 코딩된 객체를 반환하는 stub 을 주입하여 AddressRetriever class 의 단위 테스트를 작성할 수 있다.
  ```java
  public class AddressRetriever {
    private Http http;
    
    public AddressRetriever(Http http) {
      this.http = http;
    }
    
    public Address retrieve(double latitude, double longtitude) {
      // ...
      String response = http.get("http://open.api/maps?lat=" + latitude + "&long=" + longtitude);
      // return ...
    }
  }
  ```
* Mock
  * Mock 객체는 의도한 가짜 객체, 가짜 동작을 제공하고 인자가 기대되는 값인지 검증하는 일을 한다.
  * 이런 범용적인 목적으로 Mock 도구가 Mockito

#### 테스트 리펙토링
* 테스트 코드도 잘못 작성하거나 불필요한 테스트 코드를 만들면 유지 보수 비용만 증가하고 피곤할뿐 도움이 되지 않는다.
* 의무적으로 대충 만드는 코드가 아니라 프로덕션 코드에 도움이 되는 가치있고 깔끔한 테스트 코드를 작성해야 한다.

###### 불필요한 테스트 코드
* 시스템 로깅 출력
* null 체크
  * 테스트 코드에서 null 체크는 필요 없는 코드이다.
    * 프로덕션 코드에서 null 체크는 NullPointerException 을 막기 위해 필요한 코드이지만, 테스트에서는 not-null assert 는 이점이 없는 악취나는 테스트 코드이다.
    * 만약 null 을 참조한다면 JUnit 은 그 예외를 잡아 테스트 오류로 처리한다.
    * not-null assert 는 유용한 정보를 담고 있지 않다.

###### 추상화 누락
* Given, When, Then - 데이터 준비, 시스템과 동작, 결과 단언 - 세 가지 관점으로 테스트 코드가 이루어짐
* 각 단계를 위해 자세한 코드가 필요하지만, 세부 사항을 추상화하여 이해하기 쉽게해야 한다.
  🌟 `좋은 테스트는 클라이언트가 시스템과 어떻게 상호작용하는지 추상화한다.`
* 적절한 단언을 사용하여 개념을 잘 표현해보자. 아래 1번과 2번 중 무엇이 더 읽기 쉬울까? 두 번째처럼 isEmpty() 단언을 사용하여 **비어있다**는 추상적인 의미를 표시해줄 수 있다. 큰 차이는 없겠지만, 그래도 두 번째가 더 읽기가 쉽다. 이렇게 미미한 차이가 쌓여 읽기 좋은 테스트 코드가 된다.
  * Example ①
    ```java
    assertThat(result.size()).isEqualTo(0);
    ```
  * Example ②
    ```java
    assertThat(result).isEmpty();
    ```

###### 부적절한 정보 (상수 이용하기)
* 테스트 메서드에는 필요없지만 단지 컴파일이 되도록 하기 위한 의미 없는 값을 넣는 경우가 있다.
* 만약 아래와 같은 코드가 있다. 그런데 이 테스트 메소드에는 name 과 관련된 테스트를 진행하기 위해 name 을 제외한 값은 아무 값이 들어가도 상관없다고 해보자.
  ```java
  @Test
  void checkName() {
    var name = "bk";
    var people = new People(name, 10, "M");
    // assert
  }
  ```
  * 2, 3 번째 인자 10, "M" 은 어떤걸 의미하는지 테스트 코드만 보고 파악할 수 없다. 테스트 코드를 파악하는데 불필요한 의문을 남기게되고, 직접 생성자로 들어가봐야 알 수 있다.
    * 이를 해결하기 위한 가장 쉬운 방법은 `의미 있는 이름을 가진 상수`로 추출하는 것이다.
      ```java
      @Test
      void checkName() {
        var name = "bk";
        var age = 10;
        var gender = "M";
        var people = new People(name, age, gender);
        // assert
      }
      ```
      * 의미 있는 상수로 추출하는 방법도 좋다.
      * 그런데 name 과 관련된 테스트이고 그 외 필드는 중요하지 않기 때문에 아래와 같은 방법도 좋다.
    * 의미 있는 메서드로 추출하는 방법
      ```java
      @Test
      void checkName() {
        var name = "bk";
        var people = createPeopleOfName(name);
      }
      
      private People createPeopleOfName(String name) {
        var age = 10;
        var gender = "M";
        return new People(name, age, gender);
      }
      ```

###### 부푼 생성
* 큰 데이터를 검증해야 하는 경우 테스트 코드에서 큰 데이터를 생성해야 한다. 테스트 메서드에서 직접 생성하지 말고, 도우미 메서드를 활용해 분리해보자.

###### 많은 단언
* 한 테스트에 다른 의미의 단언이 들어가면, 테스트를 분리해야 한다는 의미다.
* 테스트마다 한 개의 단언으로 처리되면 테스트 목적도 분명해지고, 테스트 메소드 이름도 명확하게 만들 수 있다.

###### 테스트와 무관한 세부 사항들
* 테스트와 무관하지만, 필요한 것들은 `@BeforeEach` 또는 `@AfterEach` 로 빼보자.
* 예를들어 stream 을 열었으면 close 해야하는데 테스트 메서드에서 처리하지 말고, `@AfterEach` 를 만들어 넣어주면 테스트 메소드가 더욱 깔끔해진다.

###### 암시적 의미
* Given 절에서 사용하는 테스트를 위한 데이터도 보기 쉽고 이해하기 쉽게 만들자.
* Example) content 에 특정 단어가 포함되어 있는지 확인하는 테스트
  * `String content = "weiewjtextasfa";
  * `String content = "xxxxx_text_xxxxx";
  * 분명 위의 데이터보다 아래 데이터가 찾기 쉬울 것이다. 테스트 데이터셋도 신경써서 만들면 좋다.
