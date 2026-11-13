# 0008. 수집은 콜렉터만 하고, 알림은 Debezium CDC로 받는다

- 상태: 승인
- 날짜: 2026-11-14
- 범위: lattae-collector, lattae-alert

## 맥락

lattae-collector는 "적재 테이블에 쓰는 서비스는 이 수집기뿐"이라는 원칙을 갖고 있다. 그런데 lattae-alert의 `doc-collector`가 DART와 한경 RSS를 따로 긁고 있었다. 같은 API를 두 곳에서 부르면 호출 한도를 나눠 쓰고, 원천 목록과 이용 조건(license, display_allowed) 관리가 갈라진다.

알림 쪽은 새 공시가 저장되는 즉시 알아야 한다.

## 결정

- 알림 쪽의 직접 수집(`doc-collector`)을 없앤다. 수집은 콜렉터만 한다.
- 콜렉터 DB(PostgreSQL, `wal_level=logical`)의 변경 기록을 **Debezium 2.7**(Kafka Connect)이 읽어 `cdc.collector.public.{document,article,price_daily}` 토픽으로 낸다.
- 알림 쪽 `cdc-bridge`가 이 토픽을 파이프라인 형태로 바꾼다.
  - `document`, `article`의 새 행(`op=c`)과 최초 스냅샷(`op=r`) → `raw.documents`. 수정(`op=u`)은 버린다. `article`은 피드 생성 상태가 계속 바뀌지만, 근거로는 첫 저장만 의미가 있다.
  - `price_daily` → `ref.daily-price`(compacted). 계산용 필드만 넘긴다.
  - 근거 문서의 수집 시각은 콜렉터가 처음 저장한 시각으로 둔다. 스냅샷으로 다시 흘러온 옛 문서가 "방금 들어온 근거"로 잡히지 않게 하기 위해서다.
- 커넥터 설정은 `connect/collector-cdc.json`이고, 띄울 때마다 PUT으로 등록한다. heartbeat로 복제 슬롯을 전진시키고, `max_slot_wal_keep_size=2GB`로 WAL이 끝없이 쌓이지 않게 한다.

## 검토한 대안

| 방법 | 버린 이유 |
| --- | --- |
| 콜렉터가 Kafka에 직접 발행 | 콜렉터가 Kafka를 알아야 해서 책임이 늘고, 다른 팀원 레포를 알림 사정에 맞춰 고쳐야 한다 |
| 알림이 콜렉터 DB를 주기적으로 조회 | 조회 간격만큼 늦고, 알림이 콜렉터 테이블 구조에 직접 묶인다 |

## 결과

- 콜렉터 코드에는 알림 관련 코드가 없다. 콜렉터 DB 저장부터 ES 태깅까지 1.6초였다(로컬 측정).
- 구성 요소가 늘었다(collector-db, Kafka Connect). 콜렉터 DB가 다른 기기에 있게 되면 커넥터 주소만 바꾸고, 네트워크는 Tailscale 같은 사설망으로 잇는다.
