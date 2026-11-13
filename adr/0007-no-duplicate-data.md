# 0007. 중복 데이터는 어느 단계에도 남기지 않는다

- 상태: 승인
- 날짜: 2026-11-14
- 범위: 전체

## 맥락

Kafka 소비는 at-least-once라서 같은 메시지가 다시 올 수 있다. 커넥터를 다시 만들면 스냅샷이 다시 흐르고, 서비스가 재시작되면 마지막 커밋 이후 메시지를 다시 읽는다. 같은 근거가 두 번 보이거나 같은 알림이 두 번 오면 서비스를 믿을 수 없다.

## 결정

메시지 전달은 중복될 수 있다고 보고, **모든 저장 단계를 자연키 기준 멱등으로** 만든다.

| 단계 | 자연키 | 방법 |
| --- | --- | --- |
| 콜렉터 DB | `document.source_key`, `article(source, source_article_id)`, `price_daily(stock_code, bas_dt)` | 유일 제약 + `ON CONFLICT DO NOTHING` |
| Kafka 기준 데이터 | `ref.stock-profile`의 종목코드, `ref.daily-price`의 `code:date` | compacted 토픽 |
| 근거 문서(ES) | `docId` (`DART-`, `POLICY-`, `HK-` + 원천 키) | `_id`를 고정하고 `create`. 409면 버리고 다음 단계로 넘기지 않는다 |
| 급변 기록(ES) | `moveKey` | upsert |
| 알림(Django) | `(code, trading_date, direction)` | 유일 제약. 이미 처리한 `explain_version` 이하가 다시 오면 무시한다 |
| 푸시 | 알림당 첫 발송 | 성공 기록이 있으면 다시 보내지 않는다. 첫 발송이 실패한 경우에만 재전달 때 다시 보낸다 |

검증할 때는 위 단계별로 중복 집계를 돌려 0건임을 확인한다.

## 검토한 대안

- **exactly-once(Kafka 트랜잭션)**: Kafka 안에서는 가능하지만 DB, ES, FCM 같은 외부 저장소까지는 보장하지 못한다. 결국 소비자 쪽 멱등이 필요하다.

## 결과

- 2026-11-14 점검에서 콜렉터 DB, 근거 문서, 알림 DB, 푸시 모두 중복 0건이었다.
- 새 저장 경로를 만들 때는 자연키와 멱등 쓰기를 함께 넣어야 한다. 이 표에 행을 추가한다.
