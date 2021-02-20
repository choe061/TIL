## 1장. HTTP & 2장. URL 과 리소스

[1. HTTP 트랜잭션](#HTTP-트랜잭션)

[2. TCP 네트워크 통신](#TCP-네트워크-통신)

[3. URL 과 리소스](#URL-과-리소스)

## HTTP 트랜잭션
* 하나의 Request - 하나의 Response
* 그 상호작용은 <U>HTTP 메시지</U>를 이용
	* HTTP 메시지
		* 시작줄
			* Request : Request-Line
			* Response : Status-Line
		* Headers
		* Body

## TCP 네트워크 통신
* HTTP 프로토콜은 7Layer 에서 동작하고, 4Layer 의 프로토콜 중 하나인 TCP 프로토콜을 기반으로 데이터를 전송한다. (그리고 TCP 는 IP(3Layer)를 기반으로 동작한다.)
* TCP/IP 커넥션(3-way-handshake) 를 맺기 위해서 IP 주소와 Port 번호를 사용한다.
	* 예를들어 IP 주소가 동일해도 Port 번호가 다르다면 새로 커넥션을 맺는다.
* host name 은 DNS 를 통해 IP 로 변환된다.
	* HTTP 의 디폴트 포트 번호는 80, HTTPS 의 디폴트 포트 번호는 433

## URL 과 리소스
* URL(Uniform Resource Locator) 는 리소스의 위치를 표현
* URL 형태
	* `<스킴>://<사용자 이름>:<비밀번호>@<호스트>:<포트>/<경로>;<파라미터>?<질의>#<프래그먼트>`
		* scheme 은 리소스에 접근하는 방법. 프로토콜에 해당
		* host 는 인터넷 상의 호스트 장비
		* port 는 그 서버가 열어놓은 네트워크 포트 번호
		* 질의는 query string
		* fragment 는 웹 페이지 중 일부를 가르키고자 할때 사용한다.
