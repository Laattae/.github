# 0001. 실시간 경로의 단계 사이를 Kafka 토픽으로 끊는다

- 상태: 승인
- 날짜: 2026-11-13
- 범위: lattae-alert

## 맥락

알림은 체결 수신 → 급변 감지 → 근거 검색 → 문장 생성 → 발송을 60초 안에 끝내야 한다. 단계마다 속도가 다르다. 감지는 밀리초 단위지만 근거 검색과 문장 생성은 초 단위로 흔들리고, 개장 직후에는 수백 종목이 한꺼번에 움직인다. 한 프로세스에서 순서대로 처리하면 가장 느린 단계가 감지까지 막는다.

## 결정

- 단계 사이를 Kafka 3.9(KRaft, ZooKeeper 없음) 토픽으로 끊는다.
  - 원천: `raw.ticks`(6파티션, 종목코드 키), `raw.index`, `raw.documents`
  - 기준 데이터: `ref.stock-profile`, `ref.daily-price`(둘 다 compacted, 키당 최신값만)
  - 단계: `doc.tagged`, `move.detected`, `move.explained`
- 토픽은 자동 생성하지 않고 `kafka/create-topics.sh`에서 보존 기간과 함께 만든다.
- 원천 토픽은 2~7일 보존해 하루치를 통째로 다시 흘려 볼 수 있게 한다.
- 소비자는 처리 후 커밋한다(at-least-once). 중복 전달은 [0007](0007-no-duplicate-data.md)의 멱등 처리로 막는다.

## 검토한 대안

- **서비스 간 HTTP 호출**: 느린 단계가 앞 단계를 막고, 장애 때 메시지가 사라진다.
- **Redis Streams / RabbitMQ**: 가능하지만 Flink·Debezium·Spark와의 연결과 리플레이 보존을 Kafka만큼 쉽게 얻지 못한다.

## 결과

- 각 단계를 따로 늘리거나 재시작할 수 있다. 로컬 측정에서 체결부터 발송까지 p95가 2.0초였다.
- 운영할 구성 요소가 늘었다(Kafka, Kafka UI). 메모리 8GB 로컬 기준으로 힙을 작게 잡았다.
