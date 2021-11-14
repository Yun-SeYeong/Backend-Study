# Database

> Database에 필요한 지식 정리



## 1. Transaction

> Transaction이란, 데이터베이스를 조작하는 작업의 단위이다. 조회 와 수정을 하나의 행위로 묶는 등 한가지 이상의 행위를 묶어 하나의 단위로 표현한다. Transaction은 흔히 ACID원칙을 보장해야 한다고 한다. ACID는 각각 Atomicity(원자성), Consistency(일관성), Isolation(독립성), Durability(영구성)을 뜻한다.




- Atomicity: transaction의 작업이 부분적으로 성공하는 일이 없도록 하는 성질이다.
- Consistency: transaction이 끝나고 나면 DB의 제약조건이 맞도록 보장하는 성질이다.
- Isolation: Transaction이 진행되는 중에 다른 Transaction이 작업중인 작업을 간섭할 수 없도록 보장한다.
- Durability: Transaction이 완료되면 결과가 영구적으로 적용되는 것을 보장한다.

```
ACID원칙은 완벽히 지켜지지 않는다. 왜냐하면 ACID원칙을 지키면 동시성이 떨어지게 되기 때문이다. 
그렇기 때문에 Transaction은 Isolation level을 제공한다. Isolation level은 
`READ UNCOMMITED`, `READ COMMITED`, `REPEATABLE READ`, `SERIALIZABLE` 등이 있다. 
ACID 원칙을 덜 지킬수록 문제가 발생할 부분이 증하지만 동시성을 높일 수있다.
```







## 1. InnoDB

> InnoDB는 MYSQL을 위한 데이터베이스 엔진이다. ACID 호환 트렌젝션에 대응하고, 외래 키도 지원하고 있다. 



#### Row-level lock

InnoDB는 두개의 row-level locking을 사용한는데 shared(s) locks 와 exclusive(x) locks가 있다.

- Shared(s) lock은 transaction이 row를 읽을 때 락을 걸어준다.
- Exclusive(x) lock은 transaction이 데이터를 수정 또는 삭제할때 락을 걸어준다.

만약 `t1`이라는 transaction이 있고 `r`이라는 row에 s lock을 걸고, 또 다른 `t2`라는 transaction이 `r`에 lock을 걸어야 하는 상황일 때 다음과 같다.

- `t2`는 `r`에 바로 s lock을 걸 수 있다. 결과적으로 `t1`과  `t2` 둘다 `r`에 s lock을 건 상태가 된다.
- `t2`는 r에 x lock을 바로 걸 수 없다.

만약 `t1`에 row `r`에 x lock을 걸고 있다면,  `t2`에서는 s lock과 x lock 모두 걸 수 없다.



#### Intension Locks

InnoDB는 row locks와 table locks가 공존하여 사용하기 위해 multiple granularity locking을 지원한다.



#### Record Locks
Record lock은 index records에 걸리게 된다. 만약 `select c1 from t where c1 = 10 for update;` 일때 다른 transaction에서 해당 row에 삽입, 수정, 삭제르 할 수 없도록 막는다. 만약 index가 없는 테이블도 무조건 index records에 record lock을 걸게 되는데 InnoDB 같은 경우에는 clustered index가 자동으로 생성되는데 이걸 통해 record lock을 걸게 된다.



#### Gap Locks

Gap lock은 조건 사이에 있는 index records에 걸게된다.  만약 `SELECT c1 FROM t WHERE c1 BETWEEN 10 and 20 FOR UPDATE;` 를 실행하면 다른 트렌젝션에서 t.c1이 15에 값을 넣는 것을 막아준다. 이는 해당 하는 부분에 값이 없어도 막악준다. 왜냐하면 gap lock은 해당범위에 lock을 걸기때문이다. gap은 single index value, multiple index value에 적용되고, 만약 index가 없더라도 적용된다.







#### Transaction Isolation Level

InnoDB는 SQL:1992의 모든 transaction isolation level을 지원한다. `READ UNCOMMITED`, `READ COMMITED`, `REPEATABLE READ`, `SERIALIZABLE` 이 있다. 기본적으로 InnoDB는 `REPEATABLE READ`가 설정되어있다.



- `REPEATABLE READ`

  InnoDB의 Default 설정이다. 만약 같은 trasaction에서 여러 consistent read를 하게되면 첫번째로 완료된 snapshot을 읽게된다. 

  Locking reads (`SELECT` with `FOR UPDATE` or `FOR SHARE`), UPDATE, DELETE 상태일때 unique index에 대해 찾거나 range-type (범위로) 찾는냐에 따라 달라지게 된다.

  - Unique index를 통해 찾게되면, InnoDB는 해당 index record에만 lock을 걸게된다.
  - 외에 다른 상황에서 InnoDB는 모두 range로 찾게 된다. 이때 걸리는 락은 gap lock또는 next-key lock을 사용하게 된다.

- `READ COMMITED`

  













reference

- https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html
- https://suhwan.dev/2019/06/09/transaction-isolation-level-and-lock/
