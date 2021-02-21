## 목차

[1. RESTful](#RESTful)

[2. REST 특징](#REST-특징)

  * [2-1. Uniform interface](#Uniform-interface)

  * [2-2. Stateless](#Stateless)
  
  * [2-3. Self descriptive message](#Self-descriptive-message)

  * [2-4. Cacheable](#Cacheable)

  * [2-5. Client Server 구조](#Client-Server-구조)

[3. REST API 디자인](#REST-API-디자인)


## RESTful
* RESTful API 란?
	* REST 아키텍처의 제약조건을 `모두` 지키는 API
* REST 란?
	* Representational State Transfer
	* 프로토콜이나 표준이 아닌 아키텍처 원칙
* resource(자원), method(행위), representation(표현) 으로 구성된다.
* 리소스를 URI 로 표현하고, 리소스에 대한 행위를 HTTP Method 로 표현하는 방법


## REST 특징

#### Uniform interface
* URI 로 지정한 리소스에 대한 조작은 통일성있게 수행해야한다.
* Client 나 Server 의 플랫폼, 언어에 따라 구현 방법이 변경되거나 종속되어선 안된다.

#### Stateless
* REST API 는 무상태성으로 서로 연결되어있지 않고 각각 독립적으로 수행되어야한다. 세션이나 쿠키같은 상태 정보를 통해 API 가 동작하지 않아야한다.

#### Self descriptive message
* 하나의 REST API 만으로 전송하고자하는 내용을 자체적으로 표현할 수 있어야한다.
* Example) Header 없이 Body 만 전송하는 경우... 등등...
	* Request 는 Request-Line, Header, Body 로 이루어져야하고,
	* Response 는 Response-Line, Header, Body 로 이루어져야한다.

#### Cacheable
* HTTP 가 제공해주는 기능으로 캐싱할 수 있다.
* HTTP 표준 헤더인 `Last-Modified` 와 `E-Tag` 를 사용하여 캐싱을 적용하여 HTTP 통신에 대한 효율성을 향상시킬 수 있다.

###### Last-Modified, Last-Modified-Since
* 두 Header 를 통해 서버에 리소스가 마지막으로 언제 변경되었는지 확인할 수 있고, 그에 따라 리소스를 응답하거나 응답하지 않는 방식으로 캐싱하는 방법이다.
* Last-Modified
	* 서버가 Last-Modified Header 에 서버가 알고있는 가장 마지막에 수정된 시간을 응답 헤더에 담아 내려준다.
* Last-Modified-Since
	* 클라이언트가 이전에 Last-Modified 헤더로 받아 저장하고 있는 시간 값을 추후 다시 요청을 전송할때 Last-Modified-Since Header 에 담아 요청한다.
* 캐싱 적용 과정
	1. 처음 접속하는 경우 클라이언트는 값을 모르기 때문에 그냥 요청한다.
	2. 서버가 Last-Modified 에 리소스가 마지막으로 수정된 시간을 담고 response body 에 리소스를 담아준다.
	3. 클라이언트가 다시 접속하면 Last-Modified-Since 에 이전에 저장한 시간 값을 넣어서 서버에 요청한다.
	4. 서버에서 Last-Modified-Since 값을 확인하면 마지막으로 리소스가 마지막으로 수정된 시간 값을 비교한다.
		* 시간 값이 동일하면 → 리소스를 내려주지 않고,
		* 시간 값이 다르면 → Last-Modified 에 변경된 시간과 리소스를 내려준다.

###### E-Tag (Entity Tag), If-None-Match
* E-Tag 는 특정 버전의 리소스를 식별해주는 식별자
* Last-Modified 는 업데이트 시간을 비교하고, E-Tag 는 식별자 값을 비교하여 리소스의 변경이 일어났는지 확인한다. 캐싱 적용 과정은 Last-Modified 와 유사하다.

#### Client Server 구조
* Client - Server 구조로 이루어져 HTTP 통신


## REST API 디자인
* 규칙
	* URI : 리소스를 표현
		* 동사보다 명사로 표현
		* 긴 단어보다 `/` 로 계층 관계를 표현하도록
		* 단수와 복수를 상황에 맞게 지킨다면 더 좋다.
			* Example) 회원 리스트(컬렉션)을 조회하는 API 라면, /member 보단 /members
		* URI 설계시 추가적으로 주의할 점
			* 긴 단어보다 계층 관계로 표현하되 불가피하게 길어지는 단어는 `_` 보다 `-` 를 사용
				* 링크의 경우 대부분 밑줄을 표시해주기 때문에 `_` 는 가려질 수 있다.
	* HTTP Method : 리소스를 어떻게 처리할지 행위를 표현
		* GET, POST, PUT, PATCH, DELETE
	* HTTP status code 도 의미에 맞게 사용해야한다.

