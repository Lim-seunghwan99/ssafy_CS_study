# Transaction
- 데이터베이스 내에서 수행되는 작업의 최소 단위
- 무결성을 유지하고 DB의 상태를 변화시키는 기능을 수행
- 하나 이상의 query를 포함해야 하고 ACID라고 하는 원자성, 일관성, 고립성, 지속성의 4가지 규칙을 만족해야 한다

<br>
<br>

```
예를 들어 은행에서 A가 100만원을 출금하여 B에게 입금을 하는 상황이라고 생각해보자
A의 잔고에서는 100만원을 출금하였는데 전산오류가 생겨서 B의 계좌에는 100만원이 입금되지 않았다!!!
이렇게 예상치 못한 오류가 발생하여 데이터의 부정합이 발생하는 경우 다시 원상복귀해야 한다
따라서 모든 입출금은 하나의 "묶음" 단위로 작동해야 한다
출금을 했으면 입금을 마치던지 아니면 아예 없던 일로 하던지..
두 행위는 분리될 수 없는 하나의 거래로 처리되어야 하는 단일 업무이다
이런 업무 처리으ㅣ 최소 단위를 데이터베이스에서는 transaction이라고 한다

데이터를 다룰 때 장애가 일어나는 경우 transaction은 장애 발생 시 데이터를 복구하는 작업의 단위가 되기 때문이다
또한 데이터베이스에서 여러 작업이 동시에 같은 데이터를 다룰 때가 있는데
transaction을 통해 이 작업을 서로 분리하고 이를 통해 오류가 발생하지 않게 한다
```

<br>
<br>

# ACID
1. `A: Atomicity(원자성)`
transaction에 포함된 작업은 전부 수행되거나 아니면 전부 수행되지 않아야 한다(all or nothing)
2. `C: Consistency(일관성)`
transaction이 실행을 성공적으로 완료하면 언제나 일관성 잇는 데이터 베이스 상태로 유지하는 것을 의미<br>
ex) 송금 전후 모두 잔액의 data type은 integer이여야 한다
3. `I: Isolation(고립성)`
여러 transaction은 동시에 수행된다<br>
따라서 동시에 수행되는 transaction이 동일한 data를 가지고 충돌하지 않도록 제어해줘야 한다<br>
이를 동시성 제어(concurrency control)라고 한다
4. `D: Durability(지속성)`
성공적으로 수행된 transaction은 데이터베이스에 영원히 반영되어야 함을 의미br>
transaction이 완료되어 저장이 된 데이터 베이스는 저장 후에 생기는 정전, 장애, 오류 등에 영향을 받지 않아야 한다


```
Concurrency control
여러 개의 transaction이 한 개의 데이터를 동시에 갱신(update)할 때 한 transaction의 갱신이 무효화 될 수도 있다-> 이를 갱신손실이라고 함
그리고 이걸 동시성제어로 막을 수 있다

그러니까, transaction이 동시에 수행될 때 일관성을 해치지 않도록 transaction의 데이터 접근을 제어하는 DBMS의 기능을 동시성 제어라고 한다!!

데이터의 수정 중에 있는 transaction은 해당 데이터를 Lock으로 잠금장치를 하여 다른 Transaction이 접근하지 못하게 하는 방법이 있다
Lock이 걸린 데이터는 Unlock이 될 때까지 다른 transaction들은 접근하지 못하고 기다려야 한다
```

![image](https://github.com/ChaesongYun/ssafy_CS_study/assets/139418987/07935e01-7900-4ac4-a685-486a7e2e8e0e)

transaction2가 transaction3의 데이터가 저장(write)되기 전에 데이터에 접근하여 작업을 수행하고 그 다음 transaction3의 데이터가 저장된다면 <br>
A는 900이 되고 그 다음 transaction2의 데이터가 저장된다면 A는 1100이 된다 <br>
이처럼 잘못된 계산결과가 나오지 않도록 Read부터 Write까지 Lock을 거는 방법을 Concurrency control이라고 한다 <br>

<br>

```
Q. commit과 rollback에 대해서 설명해보세요

A. 데이터베이스는 commit과 rollback이라는 명령어를 통해 데이터 무결성을 보장합니다.
commit이란 transaction작업을 완료했다고 확정하는 명령어입니다.
transaction 작업 내용을 실제 db에 저장하고 db가 변경됩니다.
rollback은 작업 중 문제가 발생했을 때 transaction 처리 과정에서 발생한 변경 사항을 취소하고 이전 commit의 상태로 되돌립니다.
```
