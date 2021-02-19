## 목차

[1. Cache 특징 및 목적](#Cache-특징-및-목적)

[2. Spring Cache 추상화](#Spring-Cache-추상화)

[3. Spring Cache Annotation](#Spring-Cache-Annotation)

  * [3-1. @CacheConfig](#@CacheConfig)
	
  * [3-2. @Cacheable](#@Cacheable)
	
  * [3-3. @CacheEvict](#@CacheEvict)
	
  * [3-4. @CachePut](#@CachePut)
  
  * [3-5. @Caching](#@Caching)

[4. Caffeine Cache](#Caffeine-Cache)

[5. Spring Actuator 로 Caffeine Cache statistics 모니터링](#Spring-Actuator-로-Caffeine-Cache-statistics-모니터링)


## Cache 특징 및 목적
* 일반적으로 시스템 향상을 위해 Cache 를 사용한다. Spring 환경에서는 캐시 추상화를 통해 메서드에 대해 쉽게 캐시를 적용하여 메서드 성능을 향상시킬 수 있고, 캐시 구현체를 쉽게 변경할 수 있다.
* 캐시 추상화는 `spring-context` module 에 있다.
	* `spring-context-support` module 도 있는데, spring-context 를 기반으로 Caffeine, EhCache, JCache 같은 CacheManager 를 제공한다.
	* spring-context-support 를 사용하면 이미 spring-context 모듈의 의존성을 가지고 있다. 그런데 spring-boot-starter-cache 의존성을 가지면 다 포함되어있다.
    	* `spring-boot-starter-cache` → `spring-context-support` → `spring-context`
* Cache Annotation 도 Spring 환경에서 AOP Proxy 방식으로 동작하기 때문에 아래 룰을 지켜야 한다.
	1. Proxy class 가 호출할 수 있도록 public method 만 캐싱이 적용된다.
	2. 동일 클래스에서 내부 호출은 캐싱이 적용되지 않는다.

#### JCache ? Caffeine ? EhCache ?
* spring-context-support 모듈에는 세 가지 CacheManager 가 있다. 하지만, IDE 에서 클래스를 열어보면 화면 가득 빨간색이 보이고 참조되지 않는 것으로 나오는데, 필요한 Cache 구현체 디펜던시를 추가해야 사용이 가능하다.
* JCache
	* JSR-107 표준을 따르는 자바에서 제공하는 interface
* Caffeine, EhCache 등등
	* JCache 표준 interface 를 구현하는 구현체

## Spring Cache 추상화

## Spring Cache Annotation
* 캐시 활성화 방법 ①
	1. Configuration class 에 `@EnableCaching` 을 class level 에 선언하면 캐싱 기능이 활성화된다.
	2. CacheManager 를 Bean 으로 등록
        ```java
        @Configuration
        @EnableCaching
        public class CachingConfig {
            @Bean
            public CacheManager cacheManager() {
            	return new ConcurrentMapCacheManager("members");
            }
        }
        ```
* 캐시 활성화 방법 ② - Spring Boot 환경
	* Spring Boot 를 사용한다면, CacheManager 를 Bean 으로 직접 등록하지 않아도 된다. EnableCaching 만 선언하면 ConcurrentMapCacheManager Bean 이 자동으로 등록된다.
  * Spring Boot 에 의해 자동 설정된 CacheManager 를 Customize 할 수도 있다. CacheAutoConfiguration 이 customizer 를 골라서 CacheManager 를 생성하는데 적용한다.
      ```java
      @Component
      public class SimpleCacheCustomizer 
          implements CacheManagerCustomizer<ConcurrentMapCacheManager> {

          @Override
          public void customize(ConcurrentMapCacheManager cacheManager) {
          cacheManager.setCacheNames(asList("users", "transactions"));
          }
      }
      ```

#### @Cacheable
* class 나 method level 에 선언하여 캐싱을 활성화시킬 수 있다.

	* interface 에도 적용할 수 있지만, Spring Documentation 에서 추천하지 않는다. interface 에 @Cache* 를 적용하면 interface 기반의 proxy 를 사용할때만 동작한다. 
	> [Spring documentation Tip]
Spring recommends that you only annotate concrete classes (and methods of concrete classes) with the @Cache* annotation, as opposed to annotating interfaces. You certainly can place the @Cache* annotation on an interface (or an interface method), but this works only as you would expect it to if you are using interface-based proxies. The fact that Java annotations are not inherited from interfaces means that if you are using class-based proxies ( proxy-target-class="true") or the weaving-based aspect ( mode="aspectj"), then the caching settings are not recognized by the proxying and weaving infrastructure, and the object will not be wrapped in a caching proxy, which would be decidedly bad.
	
	* [Stack Overflow](https://stackoverflow.com/a/28193945/10958783) 에 다른 추가 의견도 있다. interface 는 설계 및 뼈대 역할인 반면, Cache 적용은 구체적인 세부 사항이므로 concrete class 에 적용하는게 좋아보인다는 의견이다.

	* IntelliJ 에서도 interface 에 @Cache* 어노테이션을 적용하면 warning 표시를 띄우고 인터페이스에 선언하는 방식은 추천하지 않는다고 메시지가 나온다.

* Serializable
	* JSR-107 표준인 JCache 에서는 캐싱하려는 객체는 반드시 직렬화가 필요하여 Serializable 를 상속받아야 한다.
	* 하지만 구현체에 따라 아닌 경우도 있는데 Caffeine cache 의 경우는 직렬화하지 않아도 잘 동작한다. (캐시 구현체는 중간에 변경될 수 있으니 직렬화하는 것도 좋을 것 같다.)
	
* Cacheable options
	* value, cacheNames
		* 캐시 이름
	* key
		* 해당 key 를 기준으로 저장/조회된다.
		* SpEL expression 을 사용하여 동적으로 key 를 생성할 수 있다.
	* condition
		* SpEL expression 사용
		* 메서드가 실행되기전 parameter 를 보고 조건에 맞는 경우만 캐싱하고 싶을때 사용
	* unless
		* SpEL expression 사용
		* condition 과 조금 다른데, 메서드가 호출된 후 return 값을 보고 조건에 맞는 경우만 캐싱하고 싶을때 사용
	* [sync](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/cache/annotation/Cacheable.html#sync--)
		* multi-thread 가 동시에 하나의 key 에 대해 값을 load 하려고할때 동기화 작업이 일어난다. sync 를 true 로 주는 경우 여러 쓰레드가 접근해도 하나의 쓰레드만 메서드로 진입할 수 있고, 나머지 쓰레드는 블로킹 상태에 빠진다. 결국 한 쓰레드가 값을 가져와야 나머지 쓰레드도 값을 가져갈 수 있다.
		* sync 옵션은 hint 일뿐 실제 cache provider 마다 지원할 수도 있고, 하지 않을 수도 있다.
    * sync 옵션은 unless 와 함께 사용할 수 없다.

#### @CacheEvict

* caffeine, ehcache 는 결국 in-memory cache 이다. 캐시의 크기가 증가하여 메모리를 계속 점유하지 않도록 적당히 삭제하는 기능도 구현이 필요하다.

#### @CachePut

* @CachePut 이 달린 메서드는 항상 실행되고 key 에 해당하는 캐시 값을 업데이트한다.

#### @CacheConfig

* Cacheable, CacheEvict, CachePut 을 method level 에서 사용할때 공통적으로 사용하는 설정 값을 class level 에 @CacheConfig 로 선언하여 사용할 수 있다.

#### @Caching

* 여러 어노테이션을 같이 사용하고 싶은 경우
* example
	```java
    @Caching(evict = { 
    	@CacheEvict("addresses"), 
    	@CacheEvict(value="directory", key="#customer.name") 
    })
    public String getAddress(Customer customer) {...}
	```

## [Caffeine Cache](https://github.com/ben-manes/caffeine)

* caffeine github readme 에서는 Java 8 을 기반으로 만든 고성능 캐싱 라이브러리라고 소개한다.
* Caffeine cache 의 경우 size-based eviction 과 time-based expiration 을 지원한다.
  * size-based eviction 은 빈번하게 접근했는지 최근에 생성되었는지를 기반으로 사이즈 초과분에 대해 eviction 한다. (하지만 완전한 LRU 방식은 아니라고 한다. [자세한 설명...](https://github.com/ben-manes/caffeine/wiki/Efficiency))
  * time-based expiration 은 마지막으로 접근한 시간 또는 쓰여진 시간으로부터 캐시를 만료한다.

## Spring Actuator 로 Caffeine Cache statistics 모니터링

* caffeine cache 에서는 `.recordStats()` 를 붙여주면 Actuator Metric 에서 cache 관련 통계 정보를 확인할 수 있다. 통계에는 캐시 수, 히트율도 포함되어 캐시를 튜닝하는데 유용한 정보가 된다.
```java
public CaffeineCache toCaffeineCache() {
	var cache = Caffeine.newBuilder()
						.recordStats()
						.expireAfterWrite(getExpiredAfterWrite())
						.maximumSize(getMaximumSize())
						.build();
	return new CaffeineCache(getCacheName(), );
    }
```

