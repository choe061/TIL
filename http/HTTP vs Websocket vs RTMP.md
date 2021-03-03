## HTTP vs Websocket vs RTMP

[1. HTTP](#HTTP)

[2. Websocket](#Websocket)

[3. RTMP](#RTMP)


## HTTP


## Websocket
* HTTP 는 클라이언트에서 하나의 HTTP Reqeust 를 전송하면 서버에서 하나의 HTTP Response 로 응답하는 구조로서 클라이언트에서 일방적으로 요청하는 단방향 통신이다. 반면 websocket 은 서버와 클라이언트 양측에서 데이터를 전송할 수 있어 양방향 통신이 가능한 프로토콜이다.
* 그리고 HTTP 메시지에는 여러 Headers 도 포함되어 있어 websocket 에 비해 오버헤드가 조금 더 있다.
* 그런데 websocket 도 초기 연결은 HTTP 를 기반으로 이루어진다.

#### Websocket 연결 과정
* 웹소켓은 초기 연결시 HTTP 를 기반으로 연결을 맺는다. 그렇기 때문에 HTTP 와 동일하게 TCP Handshake Connection 을 맺는다. 그 후 서버에서 protocol switching 이 이루어지고 그 때부터 ws(websocket protocol)을 사용하여 통신한다.
* 과정
	1. TCP Connection
	2. HTTP Request
		* HTTP Method : GET
		* Request Header
			* `Upgrade: websocket`
				* 클라이언트는 스위칭하려는 프로토콜을 Upgrade Header 에 명시하여 요청한다.
				* hop-by-hop header
			* `Connection: Upgrade`
	3. HTTP Response
		* HTTP Status : 101 Switching protocols
		* Response Header
			* `Upgrade: websocket`
				* 서버는 어떤 프로토콜로 스위칭했는지 Upgrade Header 에 명시하여 응답한다.
			* `Connection: Upgrade`
	4. 이때부터 ws connection 이 맺어지고, 클라이언트와 서버는 ws protocol 을 사용하여 통신한다.

###### hop-by-hop header
* [hop 이란? (출처. wikipedia)](https://ko.wikipedia.org/wiki/%ED%99%89_(%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC))
	* network 에서 출발지와 목적지 사이에 위치한 경로의 한 부분이다.
	* Proxy header 의 경우 첫 번째 Proxy 에서 해당 헤더를 소비해야하고, 뒤로 포워딩해주면 안된다.
* hop-by-hop header 란?
	* 최초 전송에 대해서만 유효하고, 캐싱되거나 프록시되지 않아야한다.
	* websocket 연결 요청시 `Upgrade`, `Connection` header 도 hop-by-hop header 이다. 그래서 최초 ws 연결 요청시만 유효하고, 실제로 연결 요청 이후에는 실제 ws 프로토콜로 스위칭되어 사용되지 않는다.
* [hop-by-hop 참고](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Connection)

###### [Upgrade Header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Upgrade)
* 클라이언트-서버 간 이미 맺어진 Connection 을 통해 다른 프로토콜로 업그레이드할 수 있다.
	* Example
		* HTTP 1.1 → HTTP 2.0
		* HTTP(s) → WebSocket(s)
* [Protocol Upgrade Mechanism](https://developer.mozilla.org/en-US/docs/Web/HTTP/Protocol_upgrade_mechanism)
	* 물론 Request Header 에서 websocket 으로 업그레이드를 요청하여도 서버에서 무조건 프로토콜 스위칭해야하는 것은 아니다. 선택사항이고 만약 서버에서 스위칭한다면 HTTP status 를 101 Switching protocols 으로 응답해주면 된다.

## RTMP
* Real Time Messaging Protocol, 실시간 통신 프로토콜
  * 비디오, 오디오를 실시간으로 전송할 수 있는 프로토콜
  * RTMPS 는 TLS/SSL 을 통한 RTMP 연결
* 디폴트로 1935 port 를 사용
  * 1935 port 연결 실패시 443(RTMPS), 80(RTMP) 를 사용하여 통신
* TCP 기반의 RTMP 는 접속을 지속적으로 유지
* 스트림을 부드럽게(끊김없이) 전달하기 위해 데이터를 여러 Fragment 로 나누어 전달
  * Video stream fragment
    * 기본 크기는 128 bytes
    * 하지만 클라이언트 서버 상황에 따라 유동적으로 결정될 수 있다.
  * Audio stream fragment
    * 기본 크기는 64 bytes

#### packet structure

<img src="https://www.globaldots.com/hubfs/Imported_Blog_Media/RTMPPacket.png">

* 패킷에는 header 와 body 가 포함되어 있다.

<img src="https://www.globaldots.com/hubfs/Imported_Blog_Media/rtmp_adobe-1.png">

* Header 는 메시지 크기, 메시지 타입, timestamp 정보가 있다.

* 참고 : https://www.globaldots.com/blog/rtmp-real-time-messaging-protocol-explained-2

#### AWS IVS

* IVS(Iteractive Video Service) 는 AWS 의 라이브 스트림 솔루션이다. 사용자에게 짧은 지연 시간으로 라이브 스트림을 제공하고, Aamzon IVS player SDK 와 Timed metadata API 를 제공해준다.
  * (Timed metadata API 는 아마도 위에서 정리한 RTMP 패킷 구조 중 header 에 있는 timestamp 를 이용하는게 아닐까?)
* [Amazon IVS Timed Metadata 를 이용하여 Quiz 만들기](https://aws.amazon.com/ko/blogs/media/part-2-using-amazon-interactive-video-service-timed-metadata-2/)
  * timed metadata 를 이용하면 정확한 시간에 사용자에게 payload 를 제공해줄 수 있다.
    *  `Timed metadata` payload 를 삽입하면 늦지도 빠르지도 않게 모든 유저에게 동시에 payload(여기서는 퀴즈 정보) 를 보여줄 수 있다.
  * example) workshop - https://ivs-streaming.workshop.aws/en/timedmetadata.html

