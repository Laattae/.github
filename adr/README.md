# 아키텍처 결정 기록 (ADR)

라때의 구조를 바꾸는 결정을 한 건에 한 파일로 남긴다. 왜 그렇게 정했는지, 무엇을 버렸는지가 코드보다 오래 남아야 할 때 쓴다.

## 목록

| 번호 | 제목 | 상태 | 범위 |
| --- | --- | --- | --- |
| [0001](0001-kafka-backbone.md) | 실시간 경로의 단계 사이를 Kafka 토픽으로 끊는다 | 승인 | lattae-alert |
| [0002](0002-flink-move-detection.md) | 급변 감지는 Flink에서 종목별 임계로 한다 | 승인 | lattae-alert |
| [0003](0003-factor-decomposition-and-escalation.md) | 시장·테마·종목 요인 분해와 재알림 규칙 | 승인 | lattae-alert |
| [0004](0004-percolator-tagging.md) | 문서 태깅은 Elasticsearch percolator로 한다 | 승인 | lattae-alert |
| [0005](0005-no-fabrication.md) | 원인을 지어내지 않는다: 원인 미확인 후 갱신 | 승인 | lattae-alert |
| [0006](0006-nightly-profiles-spark-airflow.md) | 종목 기준선은 매일 밤 Spark로 계산하고 Airflow로 돌린다 | 승인 | lattae-alert |
| [0007](0007-no-duplicate-data.md) | 중복 데이터는 어느 단계에도 남기지 않는다 | 승인 | 전체 |
| [0008](0008-collector-single-writer-cdc.md) | 수집은 콜렉터만 하고, 알림은 Debezium CDC로 받는다 | 승인 | lattae-collector, lattae-alert |
| [0009](0009-hankyung-in-collector.md) | 한국경제 RSS는 콜렉터에서 공식 원천과 함께 수집한다 | 승인 | lattae-collector |
| [0010](0010-fcm-topic-push.md) | 푸시는 종목별 FCM 토픽으로 보내고, 키 없이도 끝까지 돌린다 | 승인 | lattae-alert, lattae-app |
| [0011](0011-shared-config-repo.md) | 팀 공유 설정과 키는 비공개 environment 레포에 둔다 | 승인 | 전체 |
| [0012](0012-tick-source.md) | 실시간 체결 원천: 시뮬레이터로 시작하고 KIS를 붙인다 | 제안 | lattae-alert |
| [0013](0013-local-stack-durability.md) | 로컬 스택은 데이터를 볼륨에 두고 디스크 상한을 건다 | 승인 | lattae-alert |

## 새 ADR 쓰는 법

1. 다음 번호로 `NNNN-짧은-제목.md`를 만든다. 아래 형식을 따른다.
2. 위 목록에 한 줄 추가한다.
3. 결정을 뒤집을 때는 기존 파일을 고치지 않고 새 ADR을 쓴 뒤, 옛 ADR의 상태를 `대체됨 (→ NNNN)`으로 바꾼다.

```markdown
# NNNN. 제목

- 상태: 제안 | 승인 | 대체됨 (→ NNNN) | 폐기
- 날짜: YYYY-MM-DD
- 범위: 레포 이름

## 맥락
무엇이 문제였나. 제약은 무엇이었나.

## 결정
무엇을 하기로 했나.

## 검토한 대안
무엇을 버렸고 왜 버렸나.

## 결과
좋아진 것, 감수한 것, 이후 해야 할 일.
```
