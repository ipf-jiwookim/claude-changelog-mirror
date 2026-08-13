# claude-changelog-mirror

Claude 데스크톱 changelog를 한글로 정리해 Slack에 보내는 파이프라인. 구조와 근거는 README.md.

## 고치기 전에

- **`routine/prompt.md`는 사본이다.** 편집·커밋해도 배포되지 않는다. routine은 자체 저장 설정으로
  실행되므로 https://claude.ai/code/routines 에서 직접 붙여넣어야 한다. 순서: 레포 먼저 → routine.
- **`feed.xml`은 Actions 생성물이다.** 직접 편집하지 말 것 — 다음 실행이 덮어쓴다.

## 지우면 안 되는 것 (전부 load-bearing)

- **GitHub 미러 계층** — routine 샌드박스는 `claude.com`에 접근할 수 없다(egress allowlist,
  프록시 CONNECT 403). "미러 없이 원본을 직접 받으면 간단하다"는 제안은 파이프라인을 죽인다.
- **Slack 전송은 MCP 커넥터로만** — `hooks.slack.com`도 차단되어 webhook은 반드시 실패한다.
- **저장소는 public 유지** — `raw.githubusercontent.com`이 비공개 저장소에는 인증을 요구한다.
  미러 내용은 이미 공개된 릴리스 노트이므로 공개로 두는 것이 맞다.
- `sed '/<lastBuildDate>/d'` — 이 필드는 요청마다 바뀐다. 지우면 매시간 무의미한 커밋이 쌓인다.
- `grep -q '<item>'` — 에러 페이지·빈 응답을 커밋하는 것을 막는다.

## 금지

- routine 프롬프트에 시크릿을 넣지 말 것. RemoteTrigger API 응답에 프롬프트 전문이 실려
  대화·로그에 노출된다. 자격증명은 MCP 커넥터에 둔다.
