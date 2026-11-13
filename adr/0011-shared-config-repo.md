# 0011. 팀 공유 설정과 키는 비공개 environment 레포에 둔다

- 상태: 승인
- 날짜: 2026-11-14
- 범위: 전체

## 맥락

`.env`, Firebase 서비스 계정, API 키는 각 레포에서 git이 무시하도록 돼 있다. 그래서 팀원이 새로 클론하면 실행할 수 없고, 키를 메신저로 주고받게 된다.

## 결정

- 팀원이 공유해야 하는 설정 파일은 모두 비공개 레포 `Laattae/environment`에 둔다.
- 폴더 구조는 각 레포 경로를 그대로 따른다. 클론한 뒤 복사만 하면 된다.

```bash
# ssafy_final/ 에서
cp -r environment/lattae-app/. lattae-app/
cp -r environment/lattae-alert/. lattae-alert/
cp -r environment/lattae-collector/. lattae-collector/
```

- 지금 들어 있는 것: `lattae-app/.env.local`(Firebase 웹 설정, VAPID 키), `lattae-alert/secrets/firebase-service-account.json`, `lattae-alert/realtime/.env`, `lattae-collector/.env`(OpenDART 키, 공공데이터포털 키).
- 각 레포에는 값이 빈 `.env.example`만 둔다.

## 검토한 대안

- **비밀 관리 서비스(Vault, 클라우드 Secret Manager)**: 더 안전하지만 학생 프로젝트 규모에 비해 설정 부담이 크다.

## 결과

- 레포 접근 권한이 곧 키 접근 권한이다. 비공개를 유지하고, 팀원이 빠지면 키를 재발급한다.
- Firebase 서비스 계정은 프로젝트 관리자 권한이라 레포 밖으로 내보내지 않는다.
