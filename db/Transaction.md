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
  * S(`S`hare) Lock
    * Read 작업을 위한 잠금
    * SELECT 문에 `FOR SHARE` 를 붙여서 명시적으로 잠금
    * S Lock 을 사용하는 쿼리끼리는 같은 row 에 동시에 접근 가능
  * X(E`x`clusive) Lock
    * Write 작업을 위한 잠금
    * X Lock 이 걸린 row 는 다른 쿼리가 접근 불가능
* MySQL InnoDB 는 모든 Read Operation 에 Lock 을 걸지 않고, FOR SHARE 로 선언한 Read Operation 에 대해서만 S Lock 을 건다. 그럼 MySQL InnoDB 는 Read Operation 에 S Lock 을 걸어서 REPEATABLE READ 를 지킬까? InnoDB 에서는 S Lock 없이 아래 Example 에서 첫번째 읽은 row1 과 두번째 읽은 row1 이 동일한 값을 읽도록 보장해준다.
  * Example (이하 Transaction 을 T 로 표시)
    1. T1 에서 row1 을 읽음
    2. T2 에서 row1 의 내용을 변경 
    3. T1 에서 다시 row1 을 읽음
  * Snapshot 활용
    * 모든 Read 에 Lock 을 걸면 동시성 성능이 떨어질 것이다. 그래서 Lock 을 걸지 않고 Snapshot 을 활용하여 REPEATABLE READ 를 보장한다.
    * InnoDB 가 snapshot 파일을 이용하는 방법
      1. 실행되는 쿼리를 Log 파일에 작성한다.
      2. 다시 consistent read 를 할 때 Log 파일을 읽어 특정 시점의 snapshot 을 복구한다.
      3. 이후 동일한 row 의 read operation 은 snapshot 데이터를 바라본다.
    * 이 방법은 snapshot 을 생성하기 위한 비용이 많이 들긴하지만, S Lock 방식보다는 동시성 수준을 높일 수 있다.
* InnoDB snapshot 의 허점
  * UPDATE, DELETE 문의 WHERE 절에서는 consistent read 가 적용되지 않는다.
    * 즉, 동일한 WHERE 절 조건을 적용해도 SELECT 로 읽은 row 와 UPDATE, DELETE 로 조건으로 걸리는 row 가 다를 수 있다.
    * 그런데, UPDATE 와 DELETE 로 영향을 준 경우 그 시점으로 부터 해당 row 가 같은 Transaction 내에서 보이기 시작한다.
