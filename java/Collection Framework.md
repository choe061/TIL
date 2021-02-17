## 목차

[1. Collection 상속 구조](#Collection-상속-구조)

[2. Map Set](#Map-Set)
  
  * [2-1. Set](#Set)
  
  * [2-2. Map](#Map)
  
  * [2-3. HashMap](#HashMap)
  
  * [2-4. TreeMap](#TreeMap)

[3. Concurrent Collections](#Concurrent-Collections)

  * [3-1. Synchronized Java Collections](#Synchronized-Java-Collections)

  * [3-2. Concurrent Collections](#Concurrent-Collections)


## Collection 상속 구조
* Java 의 Collection 상속 관계를 처음 보고 몰랐던 점은 Map 은 Collection interface 를 상속받지 않는다는 것이다.
* 상속 구조
  * Iterable <- Collection <- Set, List
    * Set 과 List 는 Collection interface 를 상속받는 반면
  * Map
    * Map 은 Collection interface 를 상속받지 않는다.
    * [stack overflow 에 관련 질문](https://stackoverflow.com/questions/5700135/why-does-map-not-extend-collection-interface)을 찾아볼 수 있었다. 답변은 생각보다 간단한 이유였다.
      * 두 interface 의 메서드 시그니처가 다르기 때문에 Map 은 Collection interface 를 상속받지 않는다. Collection 은 한 값에 대한 여러 항목을 담아서 처리하는데, Map 은 Key Value 에 대한 entry set 을 담고 처리하는 기능이라 메서드 시그니처가 달라질 수 밖에 없다. 예를 들어 remove 메소드를 보면 알 수 있다.
        * Collection interface
        ```java
        // Collection interface 의 remove 는 element 를 메서드 파라미터로 넘겨 삭제한다.
        boolean remove(Object o);
        
        boolean add(E e);
        ```
        * Map interface
        ```java
        // Map interface 의 remove 는 key 를 메서드 파라미터로 넘겨 삭제한다. Collection interface 와 같은 메서드 시그니처를 가지려면 Entry(key/value) 를 메서드 파라미터로 넘겨야 할 것 같다.
        V remove(Object key);
        
        V put(K key, V value);
        ```

## Map Set
* Collection interface 중 많이 사용하는 Map, Set 에 대해 정리.

#### Set
* 대표적으로 HashSet 의 구현 코드를 보면 Map 을 사용하고 있다. Set 은 Map 과 유사하게 동작할 것이라는 것을 짐작할 수 있다.
  ```java
  public class HashSet<E> extends ... {
    // ...
    private transient HashMap<E,Object> map;

    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();
    
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
    // ...
  }
  ```
#### Map
* key/value 쌍으로 이루어진 집합
  * key 중복을 허용하지 않는다.
  * null 을 허용한다.

#### HashMap
* HashMap 의 put method 구현 코드를 보면 key 의 hashcode 로 중복 체크를 하는 부분이 있다.
  ```java
  public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
  }

  static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  }
  ```
* null 을 허용한다.
  * 위의 코드를 보면 key 가 null 인 경우 예외를 던지지 않고 0 을 리턴한다. 즉 key 가 null 인 경우도 허용한다.
* hashcode 충돌 해결 방법
  * 요약하자면 key 충돌 시 Separate Chaning 방식을 사용하여 해결하는데, Java8 부터는 충돌된 Key 에 대해서는 LinkedList 로 저장하고 데이터 개수가 늘어나면 Tree(정확히는 Red-Black Tree) 로 변환하여 저장한다.

#### TreeMap
* [Oracle java docs - TreeMap](https://docs.oracle.com/javase/8/docs/api/java/util/TreeMap.html)
  > A Red-Black tree based NavigableMap implementation. The map is sorted according to the natural ordering of its keys, or by a Comparator provided at map creation time, depending on which constructor is used.
    * NavigableMap 의 구현체
      * SortedMap 을 상속받는다.
      ```java
      public interface NavigableMap<K,V> extends SortedMap<K,V>
      ```
    * sorted map 이며 key 를 기준으로 정렬되고,
    * map 생성시 제공되는 Comparator 를 사용하여 비교한다.
      * 주의할 점! key 를 hashcode 가 아닌 Comparator 로 비교

## [Concurrent Collections](https://docs.oracle.com/javase/tutorial/essential/concurrency/collections.html)

#### Synchronized Java Collections
* 기존 Collection Framework 의 구현체는 single-thread 환경에서 정상적으로 동작하지만, multi-thread 환경에서 동시성 문제를 가진다. Collection 을 사용할때 동시성 문제를 해결하기 위해 Synchronization wrappers 를 지원한다.
  ```java
  List<Integer> syncList = Collections.synchronizedList(new ArrayList<>());
  ```
  * Collections 의 synchronizedList 같은 static method 를 이용하여 thread-safety 한 view 를 반환한다.

#### Synchronized Java Collections 의 문제점과 Concurrent Collections
* 위 예제 속 `List<Integer> syncList = Collections.synchronizedList(new ArrayList<>())` 의 경우는 Collection 전체 단위로 Lock 되기 때문에, 한 번에 한 Thread 만 접근할 수 있다. 그래서 큰 성능 저하가 발생한다. 
* 하지만 아래에서 설명할 Concurrent Collections 는 데이터를 나누어 segments 단위로 thread-safety 를 보장한다. 그래서 multi-thread 환경에서도 한 Collection 에 여러 Thread 가 접근할 수 있기 때문에 성능이 크게 개선된다.

#### Concurrent Collections
* `java.util.concurrent` package 에는 Collection Framework 가 몇 가지 추가되어 있다. 크게 세 가지 interface 로 분류된다.
  1. BlockingQueue
    * producer-consumer queue 로 사용
    * support Collection interface
    * thread-safe
      * 대부분의 queuing method 는 내부적으로 잠금을 걸거나 다른 방법의 동시성 제어를 통해 원자적으로 수행된다.
      * `bulk Collection 연산은 원자적으로 동작하지 않는다.`
        * ex) addAll, containsAll, retainAll and removeAll
    * 참고
      * https://www.baeldung.com/java-blocking-queue
      * https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html
  2. ConcurrentMap
    * Map 의 sub-interface
    * 유용한 atomic operations 지원
    * ConcurrentHashMap 이 대표적인 구현체
    * thread-safety 와 atomicity 를 보장
      * 데이터를 나누어 segments 단위로 thread-safety 를 보장
    * 참고
      * https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentHashMap.html
  3. ConcurrentNavigableMap
    * ConcurrentMap 의 sub-interface
    * NavigableMap 작업을 지원하는 ConcurrentMap
      * NavigableMap 은 SortedMap 을 상속받으니까 정렬을 지원하는 concurrent map 인가?
    * recursive 하게 navigable sub-map 을 가진다.



