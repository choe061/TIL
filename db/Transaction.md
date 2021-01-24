## 목차
[1. 트랜잭션이란](#트랜잭션이란)

[2. ACID](#ACID)

[3. Propagation](#Propagation)

[4. Isolation](#Isolation)

## 트랜잭션이란
* 하나의 논리적인 기능을 수행하기 위한 최소한의 작업 단위이다. 데이터의 일관성과 무결성을 보장하기 위해선 트랜잭션은 필수이다.
* Spring Framework 에서는 애플리케이션 전반에 걸친 관심사로 AOP 를 이용하여 선언적인 트랜잭션 관리 방법을 지원한다.

## ACID
* 원자성 (Atomicity)
  * 하나의 트랜잭션 범위에서 수행된 작업들은 전부 commit 되던지 전부 rollback 되어야 한다.
* 일관성 (Consistency)
  * 모든 데이터는 일관성있는 규칙으로 처리되어야 한다.
* 독립성 (Isolation)
  * 각 트랜잭션은 서로 영향없이 독립적으로 수행되어야 한다.
* 지속성 (Durability)
  * 한 번 Commit 된 데이터는 영구히 보관되어야 한다.

## Propagation

## Isolation
#### MySQL InnoDB Lock
* MySQL InnoDB 의 Isolation Level 은 REPEATABLE READ 가 디폴트이다. InnoDB 는 기본적으로 row level 의 Lock 을 지원한다. 
* S Lock, X Lock
  * S(`S`hare) Lock 은 Read 작업을 위한 잠금
  * X(E`x`clusive) Lock 은 Write 작업을 위한 잠금
