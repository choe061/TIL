## 목차
[1. Tip. IntelliJ 에서 코틀린 코드 디컴파일](#Tip.-IntelliJ-에서-코틀린-코드-디컴파일)

[2. inline fun 디컴파일된 코드](#inline-fun-디컴파일된-코드)

  * [2-1. inline fun & lambda parameter](#inline-fun-&-lambda-parameter)
  * [2-2. no inline fun & lambda 가 아닌 parameter](#no-inline-fun-&-lambda-가-아닌-parameter)
  * [2-3. no inline fun & lambda param & 정적인 데이터](#no-inline-fun-&-lambda-param-&-정적인-데이터)

[3. 정리](#정리)

## Tip. IntelliJ 에서 코틀린 코드 디컴파일
* 코틀린 코드가 어떻게 자바로 변환되는지 보면 inline 함수를 쉽게 이해할 수 있다.
* 방법
  * IntelliJ 상단 메뉴 > Tools > Kotlin > `Show Kotlin bytecode`
    * 위 명령을 실행하면 아래 컴파일된 bytecode 를 확인할 수 있다.
  * Kotlin bytecode 탭 > `Decompile` 버튼을 누르면 Java code 로 디컴파일된 소스 코드를 볼 수 있다.

## inline fun 디컴파일된 코드

#### inline fun & lambda parameter
* 첫번째 Kotlin 코드 예시
  * testInlineFun, testNotInlineFun 함수 2개가 있고, test 메서드에서 호출하는 코드.
    * testInlineFun : inline 함수이고, lambda 파라미터를 받는 함수
    * testNotInlineFun : 일반 함수이고, lambda 파라미터를 받는 함수
```Kotlin
class InlineExample {
    fun test(num: Int) {
        testInlineFun { println("call inline fun. input num : $num") }
        testNotInlineFun { println("call not inline fun. input num : $num") }
    }
}

inline fun testInlineFun(lambda: () -> Unit) {
    lambda()
}

fun testNotInlineFun(lambda: () -> Unit) {
    lambda()
}
```

* 위 코드는 아래와 같이 변환된다.
  * testInlineFun : test 함수 내부에 코드가 삽입되어 있다.
  * testNotInlineFun : 함수를 직접 호출하며, Function 객체를 생성하여 파라미터로 넘긴다.
```Java
public final class InlineExample {
   public final void test(final int num) {
      // inline 키워드를 붙인 함수 호출
      int $i$f$testInlineFun = false;
      int var3 = false;
      String var4 = "call inline fun. input num : " + num;
      System.out.println(var4);

      // inline 키워드를 붙이지 않은 함수 호출
      InlineExampleKt.testNotInlineFun((Function0)(new Function0() {
         // $FF: synthetic method
         // $FF: bridge method
         public Object invoke() {
            this.invoke();
            return Unit.INSTANCE;
         }

         public final void invoke() {
            String var1 = "call not inline fun. input num : " + num;
            System.out.println(var1);
         }
      }));
   }
}
```

#### no inline fun & lambda 가 아닌 parameter
* 두번째 Kotlin 코드 예시
  * inline 함수도 아니고, parameter 가 lambda 도 아니다.
```Kotlin
class InlineExample {
    fun test2(num: Int) {
        testNotInlineFun(num)
    }
}

fun testNotInlineFun(num: Int) {
    println(num)
}
```

* 위 코드는 아래와 같이 변환된다.
  * 당연하지만 lambda 가 아닌 일반 변수를 파라미터로 전달한다면, 굳이 inline 함수를 사용할 필요는 없어보인다.
```Java
public final class InlineExample {
    public final void test2(int num) {
        int $i$f$testInlineFun = false;
        System.out.println(num);
        InlineExampleKt.testNotInlineFun(num);
    }
}
```

* 만약 아래처럼 코틀린 코드를 작성하면, IntelliJ 에서 warning 표시를 와 함께 아래 메시지를 보여준다.
  > Expected performance impact from inlining is insignificant. Inlining works best for functions with parameters of functional types
    * 함수의 parameter 가 functional type 인 경우에 inline 함수로 성능 향상을 기대할 수 있지만, parameter 가 다른 타입인 경우는 성능 향상은 굉장히 미미하다. 즉 functional type 이 아닌 경우 굳이 inline 을 사용할 필요가 없다.
```Kotlin
inline fun testInlineFun(num: Int) {
    println(num)
}
```

#### no inline fun & lambda param & 정적인 데이터
* 세번째 Kotlin 코드 예시
  * lambda 의 body 코드가 하드 코딩된 값을 사용하는 경우 컴파일러가 코드를 최적화하는지 어디선가 생성된 Function 객체를 재사용한다.
```Kotlin
class InlineExample {
    fun test() {
        testNotInlineFun { println("call not inline fun.") }
    }
}

fun testNotInlineFun(lambda: () -> Unit) {
    lambda()
}
```

* 위 코드는 아래와 같이 변환된다.
  * 첫번째 케이스 코드처럼 `new Function()` 으로 객체를 생성하지 않고, `(Function0)null.INSTANCE` 미리 생성된 INSTANCE 를 사용하는 것 같다.
  * 컴파일러가 알아서 최적화하는 것 같다.
```Java
public final class InlineExample {
   public final void test() {
      // inline 키워드를 붙이지 않은 함수 호출
      InlineExampleKt.testNotInlineFun((Function0)null.INSTANCE);
   }
}
```

## 정리
* parameter 가 functional type 인 경우 inline 키워드를 사용하면 `메모리 절약`과 `성능 향상`을 꾀할 수 있다.
  * 런타임에 Function 객체를 생성하지 않아 메모리를 절약할 수 있다.
    * 함수의 parameter 가 functional type 인 경우에 inline 함수로 성능 향상을 기대할 수 있지만, parameter 가 다른 타입인 경우는 성능 향상은 굉장히 미미하다.
  * inline 함수의 코드 자체가 함수를 호출하는 곳으로 삽입되기 때문에 `method call stack` 이 줄어든다.
