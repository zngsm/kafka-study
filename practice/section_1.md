## 1. Kafka topic, producer, consumer 이해
```buildoutcfg
producer -> send (message) -> topic[(partition1), (partition2), ...] <- consume(message) <- consumer
```
### Topic
> partition 으로 구성된 일련의 로그 파일이다.

- 하나의 토픽은 여러 개의 partition 을 가질 수 있다.
- key, value 기반의 메시지 구조이며, value 는 java에서 serializable 한 모든 타입 가능
- 시간에 흐름에 따라 메시지가 appendehlsek.


### Partition
> kafka 병렬 성능과 가용성 기능의 핵심요소이다.

- 개별 파티션은 정렬되고, 변경할 수 없는 일련의 레코드로 구성된 로그 메시지
- 개별 레코드는 offset 으로 불리는 일련번호를 할당받는다.
- 개별 파티션은 다른 파티션과 완전히 독립적이며, offset 은 개별 파티션에서 정렬되고 할당


### 토픽과 파티션의 병렬 분산 처리
- 메시지는 병렬 성능과 가용성을 고려한 방식으로 토픽 내의 개별 파티션에 분산 저장된다.
- replication 으로 인해 실제 topic 의 저장공간보다 더 많은 용량이 소모된다.
```markdown
# replication-factor=2 인 경우

문제 케이스: broker_2 가 작동하지 않는 경우, Topic A 의 50% 에 해당하는 메시지가 유실될 수 있음
======================================
[카프카 클러스트]
- broker_1
    - Topic A
        - partition #0
- broker_2
    - Topic A
        - partition #1
=======================================

해결 방법: 각 broker 간 partition 을 복제한다.
======================================
[카프카 클러스트]
- broker_1
    - Topic A
        - partition #0 (leader)
        - partition #1 (follower)
- broker_2
    - Topic A
        - partition #1 (leader)
        - partition #0 (follower)
=======================================
-> broker_2 에 이상이 생긴다면,
broker_2 topic A 의 leader 인 partition #1의 follower 가 leader로 동작
```