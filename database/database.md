# Database

> Database에 필요한 지식 정리



## 1. Transaction

> Transaction이란, 데이터베이스를 조작하는 작업의 단위이다. 조회 와 수정을 하나의 행위로 묶는 등 한가지 이상의 행위를 묶어 하나의 단위로 표현한다. Transaction은 흔히 ACID원칙을 보장해야 한다고 한다. ACID는 각각 Atomicity(원자성), Consistency(일관성), Isolation(독립성), Durability(영구성)을 뜻한다.




- Atomicity: transaction의 작업이 부분적으로 성공하는 일이 없도록 하는 성질이다.
- Consistency: transaction이 끝나고 나면 DB의 제약조건이 맞도록 보장하는 성질이다.
- Isolation: Transaction이 진행되는 중에 다른 Transaction이 작업중인 작업을 간섭할 수 없도록 보장한다.
- Durability: Transaction이 완료되면 결과가 영구적으로 적용되는 것을 보장한다.

```
ACID원칙은 완벽히 지켜지지 않는다. 왜냐하면 ACID원칙을 지키면 동시성이 떨어지게 되기 때문이다. 그렇기 때문에 Transaction은 Isolation level을 제공한다. Isolation level은 `READ UNCOMMITED`, `READ COMMITED`, `REPEATABLE READ`, `SERIALIZABLE` 등이 있다. ACID 원칙을 덜 지킬수록 문제가 발생할 부분이 증하지만 동시성을 높일 수있다.
```







## 1. InnoDB

> InnoDB는 MYSQL을 위한 데이터베이스 엔진이다. ACID 호환 트렌젝션에 대응하고, 외래 키도 지원하고 있다. 



