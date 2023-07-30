# concurrency-issue

### MySQL을 활용한 다양한 방법

- Pessimistic Lock
  - 실제로 데이터에 Lock을 걸어서 정합성을 맞추는 방법. 
  - exclusive lock을 걸게되면 다른 트랜잭션에서는 lock이 해제되기 전에 데이터를 가져갈 수 없게 됨. 
  - 데드락이 걸릴 수 있기 때문에 주의하여 사용해야 함.
  - select s1_0.id,s1_0.product_id,s1_0.quantity from stock s1_0 where s1_0.id=? **for update** 구문에서 for update가 lock을 거는 부분임.
  - 충돌이 빈번하게 일어난다면 optimistic lock 보다 성능이 좋을 수 있음.
  - lock을 통해 update를 제어하기 때문에 데이터 정합성이 보장됨.
  - 별도의 lock을 잡기 때문에 성능 감소가 있을 수 있음.
- Optimistic Lock
  - 실제로 lock을 이용하지 않고 버전을 이용함으로써 정합성을 맞추는 방법
  - 먼저 데이터를 읽은 후에 update를 수행할 때 내가 읽은 버전이 맞는지 확인하며 업데이트
  - 내가 읽은 버전에서 수정사항이 생겼을 경우에는 application에서 다시 읽은 후에 작업을 수행해야 함.
  - 별도의 lock을 잡지 않으므로 pessimistic lock 보다 성능상 이점이 있음.
  - update가 실패했을 때 재시도 로직을 개발자가 직접 작성해야 하는 번거로움이 있음.
  - 충돌이 빈번하게 일어나지 않을 것이라 예상되면 optimistic lock 사용을 권장.
- Named Lock
  - 이름을 가진 metadata locking
  - 이름을 가진 lock을 획득한 후 해제할 때까지 다른 세션은 이 lock을 획득할 수 없도록 함.
  - transaction이 종료될 때 lock이 자동으로 해제되지 않으므로 주의.
  - 별도의 명령어로 해제를 수행해주거나 선점시간이 끝나야 해제됨.
  - datasource (JDBC)를 분리하여 사용해야 함. 같은 datasource를 사용하면 커넥션풀이 부족해지는 현상으로 인해, 다른 서비스에도 영향을 끼칠 수 있음.
  - 주로 분산락을 구현할 때 사용.
  - pessimistic lock은 timeout을 구현하기 힘들지만, named lock은 손쉽게 구현할 수 있음.
  - 데이터 삽입 시에 정합성을 맞춰야 하는 경우에도 named lock을 사용할 수 있음.
  - 트랜잭션 종료 시에 lock 해제 및 세션 관리를 잘해줘야 하기 때문에, 주의해서 사용해야 함.