## 목차

[1. Collection 상속 구조](#Collection-상속-구조)

[2. Map Set](#Map-Set)
  
  * [2-1. Set](#Set)
  
  * [2-2. Map](#Map)
  
  * [2-3. HashMap](#HashMap)
  
  * [2-4. TreeMap](#TreeMap)

[4. Concurrent Collection](#Concurrent-Collection)


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

## Concurrent Collection




