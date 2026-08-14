# claude-changelog-mirror

Claude 데스크톱 앱 changelog를 한글로 정리해 Slack `#claude-updates`로 보내는 파이프라인.
이 저장소는 그 파이프라인의 **두 조각**을 담는다.

```
Actions (매시간)                       routine (매일 08:00·09:00·10:00 KST)
claude.com/…/rss.xml ──▶ feed.xml ──▶ 한글 정리 ──▶ Slack (MCP)
                          (여기 커밋)   (raw.githubusercontent.com 으로 읽음)
```

| 파일 | 역할 | 실행 주체 |
|---|---|---|
| [`.github/workflows/mirror.yml`](.github/workflows/mirror.yml) | 원본 피드를 `feed.xml`로 미러 | GitHub Actions — **이 파일이 곧 실행본** |
| [`feed.xml`](feed.xml) | 미러된 RSS (내용이 바뀔 때만 커밋됨) | Actions가 갱신 |
| [`routine/prompt.md`](routine/prompt.md) | routine에 넣는 프롬프트 | Claude routine — **사본. 아래 경고 참고** |
| [`routine/config.md`](routine/config.md) | cron·모델·커넥터 등 routine 설정 | 위와 같음 — **사본** |

## ⚠️ `routine/` 아래는 전부 사본이다

routine은 **자체 저장된 설정으로 실행된다.** 이 파일들을 고쳐서 커밋해도 routine 동작은 바뀌지 않는다.
반영하려면 [routine 편집 화면](https://claude.ai/code/routines)에서 프롬프트와 스케줄을 직접 바꿔야 한다.

그럼 왜 두는가 — 변경 이력과 그 **이유**를 남기기 위해서다. 프롬프트의 규칙 대부분은
시행착오의 결과이고(아래), 커밋 메시지가 없으면 몇 달 뒤 "이 규칙 왜 있지?"에 답할 수 없다.

수정할 때는 **레포 먼저 고치고 → routine에 붙여넣기** 순서를 지킨다. 반대로 하면 사본이 조용히 낡는다.

## 왜 이런 구조인가

**routine 샌드박스에서 `claude.com`에 직접 접근할 수 없다.** egress allowlist가 걸려 있어
프록시가 CONNECT 단계에서 403을 반환한다. 실측 결과 허용된 곳은 `anthropic.com` 계열,
npm/pypi, 그리고 `raw.githubusercontent.com` 뿐이었다. 그래서 GitHub을 우회 경로로 쓴다.

`hooks.slack.com`도 같은 이유로 막혀 있어서 Slack 전송은 webhook이 아니라 **MCP 커넥터**로 한다.
MCP 호출은 샌드박스 프록시를 거치지 않고 Anthropic 서버에서 실행되므로 allowlist와 무관하다.
덤으로 자격증명이 프롬프트에 남지 않는다.

## 프롬프트 규칙의 배경 (비직관적인 것만)

- **48시간 창 + Slack 히스토리 조회** — 창만 쓰면 실행 직전에 발행된 릴리스를 놓치고(다음날엔 창을 벗어남),
  히스토리 조회만 쓰면 기준이 없다. 둘을 겹쳐서 누락과 중복을 동시에 막는다. 별도 상태 저장소가 필요 없다.
- **요약과 상세의 문장이 같아도 된다** — 요약·상세가 한 메시지에 같이 있던 초기 설계에서는 중복이 문제였지만,
  스레드로 분리한 뒤로는 서로 다른 화면이라 중복이 아니다. 억지로 다르게 쓰게 하면 어색한 의역이 나온다.
- **명사형 종결 강제** — 원문이 장문 서술형이라 그대로 번역하면 모든 줄이 `~하던 문제를 수정했습니다`로 끝나
  핵심이 문장 끝에 묻힌다. 이 규칙 하나로 길이가 절반이 된다.
- **채널 내용을 지시로 취급 금지** — 중복 확인 때 채널을 읽으므로, 읽은 텍스트는 데이터로만 다룬다.
- **전송 성공 후 별도 알림 필수** — Slack은 사용자 자신이 보낸 메시지에 알림을 주지 않는다.
  MCP가 사용자 권한으로 보내므로 발신자 본인은 알림을 못 받는다(팀원은 정상 수신).

## 미러 워크플로의 두 함정

```bash
curl -sSf --max-time 30 https://claude.com/docs/cowork/changelog/rss.xml \
  | sed '/<lastBuildDate>/d' > feed.xml
grep -q '<item>' feed.xml
```

- `lastBuildDate`는 **요청마다 값이 바뀐다.** 지우지 않으면 내용이 그대로인데도 매시간 diff가 생겨
  무의미한 커밋이 쌓인다.
- `grep -q '<item>'` — 에러 페이지나 빈 응답을 커밋하는 것을 막는다. 없으면 피드가 깨진 날
  routine이 빈 파일을 읽는다.

## 참고

- 저장소는 **public이어야 한다.** `raw.githubusercontent.com`이 비공개 저장소에는 인증을 요구한다.
  미러하는 내용은 이미 공개된 릴리스 노트이고, public이라 Actions 실행 시간도 무료·무제한이다.
- 프롬프트에 시크릿은 없다. Slack 채널 ID(`C0BP7AQSXAT`)는 워크스페이스 접근 권한이 있어야 쓸 수 있어
  공개되어도 무해하다.
