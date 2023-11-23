# Lock
- 동시성 이슈를 해결하는 방법 

### DB Lock
------------------------------------------------------------------------------------------
### 1. 낙관적 락(Optimistic Lock) 
![image](https://github.com/eunseo2/TILL/assets/70589857/af9b0247-1980-46c7-8a02-eb8fdc879e36)

- 버전으로 관리합니다.
- 트랜잭션이 커밋될 때, 애플리케이션 격리가 위반되었는지 체크합니다. 만약, 위반하였다면 해당 트랜잭션은 Rollback 합니다. 
- Race condition 발생이 심하지 않은 상황이라면 낙관적락이 비관적락보다 비교적 성능이 좋습니다. 단, 경쟁이 심하다면 Rollback 비율이 높아지기 때문에 성능이 떨어집니다.
- 어플리케이션에서 관리되므로, db의 cpu, performace 부담이 없습니다. 
### 테스트   

```java
  @Version //javax.persistence
  private Long version;
```
```java
@Lock(value = LockModeType.OPTIMISTIC)
```

** 결제 동시 요청

동시성 문제에 대한 테스트로는 curl을 이용하여 재사용 할 수 있도록 파일로 저장하면 테스트 하기 용이합니다. (스크립트)

![image](https://github.com/eunseo2/TILL/assets/70589857/2aef1cd1-cd2d-4385-914d-37a242686ec4)


- 최초 한번만 실행되고 두번째 요청에 대해서는 에러가 발생합니다.
- 다른 요청도 처리하기 위해서는 `ObjectOptimisticLockingFailureException`  예외를 잡아서 retry를 시켜야 합니다. 



### 2. 비관적 락 (Pessimistic Lock)
- 각 트랜잭션이 실행이 되는 동안 전체 데이터베이스에 독점 잠금을 획득합니다.
- 즉, 락이 걸린 상태에서 다른 트랜잭션은 락이 끝날때까지 대기 상태가 됩니다.

![image](https://github.com/eunseo2/TILL/assets/70589857/332bcfe0-e516-4860-baf1-cd2805b15d26)


### **s Lock**

- 다른 사용자가 동시에 읽을 수는 있지만, Update Delete 를 방지합니다. 
- JPA: PESSIMISTIC.READ

```java
@Transactional(readOnly = true)
@Lock(value = LockModeType.PESSIMISTIC_READ)
```

### **x Lock**

- 다른 사용자가 읽기, 수정, 삭제 모두를 불가능합니다. 
- JPA: PESSIMISTIC.WRITE

```java
@Transactional
@Lock(value = LockModeType.PESSIMISTIC_WRITE)
```
### 3. 분산 락
- 여러 서버에서 공유된 데이터를 제어하기 위해 사용합니다.

(1) Lettuce
- setnx(set not exist) 명령어를 활용하여 분산락 구현
- Spin lock 방식 -> Retry 필요
- srping data redis를 이용하면 lettuce가 기본이므로, 별도의 라이브러리가 필요하지 않습니다. 

![image](https://github.com/eunseo2/TILL/assets/70589857/8d2bb1b4-5fd3-446d-9c45-4a6969b4551f)


(2) Redisson
- Pub-sub 기반으로 Lock 구현 제공
- 락 획득 재시도를 기본으로 제공합니다. 
- Lettuce는 계속 락 획득을 시도하는 반면에 Redisson은 구독하고 있으므로 락해제 메세지를 받으면 락 획득을 시도합니다. -> 레디스 부하 감소



참고자료
https://sabarada.tistory.com/175
