# claude-changelog-mirror

Claude 데스크톱 앱 changelog RSS 피드 자동 미러 (Slack 알림용).

- 원본: https://claude.com/docs/cowork/changelog/rss.xml
- 미러: [`feed.xml`](feed.xml) — GitHub Actions가 매시간 갱신, **내용이 바뀔 때만** 커밋
- 소비자: Claude Code routine이 `raw.githubusercontent.com`으로 읽어 한글 번역 후 Slack 전송

## 왜 미러가 필요한가

routine이 도는 클라우드 샌드박스는 egress allowlist가 걸려 있어 `claude.com`에 직접 접근할 수 없다
(프록시 CONNECT 단계에서 403). 허용된 호스트 중 하나가 `raw.githubusercontent.com`이라,
피드를 이 저장소로 옮겨 두고 routine이 그쪽을 읽는다.

`lastBuildDate`는 요청마다 값이 바뀌므로 미러 시 제거한다 — 그대로 두면 매시간 무의미한 커밋이 쌓인다.
