## 목차

[1. DLQ 생성을 위한 Headers](#DLQ-생성을-위한-Headers)

[2. DLQ 생성 Example](#DLQ-생성-Example)

[3. RabbitMQ management 의 shovel plugin](#RabbitMQ-management-의-shovel-plugin)


## DLQ 생성을 위한 Headers

### x-dead-letter-exchange

* Queue 에서 일정 시간 이후 메시지가 re-publish 될 exchange
* 이 Queue 에 publish 된 데이터들은 x-message-ttl 또는 x-expires 시간 이후에 exchange 값을 사용하여 새로운 큐로 바인딩된다

### x-dead-letter-routing-key

* x-dead-letter-exchange 로 exchange 를 지정한 후 더 디테일하게 routing 하고 싶은 경우 routing-key 추가
  * exchange 가 fanout 방식인 경우 필요 없음

### x-message-ttl

* message 의 time-to-live
* Queue 생성 시 x-message-ttl 로 지정한 ms 만큼 보존된 후 x-dead-letter-exchange, x-dead-letter-routing-key 로 지정한 Queue 로 re-publish 된다.

### x-expires

* x-message-ttl 은 queue 생성 시 기본 값
* x-expires 는 그 기본 값을 변경할 때 사용

## DLQ 생성 Example

### event exchange 생성

* event queue 와 event_retry queue 를 binding 할 exchange
* event
  * routing key : event
* event_retry
  * routing key : event.retry

### event.retry queue 생성

* event exchange 에 binding 하기

### event queue 생성

* x-dead-letter-exchange 만 추가하거나 또는 x-dead-letter-exchange 와 -routing-key 를 함께 넣어야한다.
* x-message-ttl 은 큐에 데이터가 만료될 시간, 이 시간 이후 retry 큐로 이동하게 된다.
  * 큐 생성 시 x-message-ttl 로 디폴트 값을 넣을 수 있고, x-expires 로 케이스별로 시간 값을 다르게 줄 수 있다.
  
## [RabbitMQ management 의 shovel plugin](https://www.rabbitmq.com/shovel.html)

* 무한 루프를 방지하기 위해 재시도 횟수를 정해야한다. 재시도 횟수가 모두 지난 메시지는 임시 queue 에 넣거나 어떤 처리가 필요하다. 만약 error queue 라는 임시 큐에 넣는다고 할때, 이후에 error queue 에 있는 메시지를 수동으로 전송하고 싶다면, rabbitmq plugin 을 이용하여 메시지를 다른 queue 로 이동시킬 수 있다.
* event queue 에서 발송이 실패되면 event_retry queue(DLQ) 를 통해 N 회 재시도한다. N 회의 재시도를 모두 실패하면 더 이상 재시도하지 않고 error queue 로 임시 보관한다. 임시 보관 중인 error queue 의 메시지들을 재발송하고자할 때, shovel plugin 을 사용하여 error → event 로 이동시킬 수 있다.

