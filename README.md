# sonmat-skills

손맛 햅틱 제작용 Claude 스킬·에이전트. 저장소 뿌리에서 Claude Code 를 열면 `.claude/` 아래가 그대로 읽힌다.

- `skills/sonmat-new-game` — 새 게임 붙이기의 입구. 아래 다섯 단계를 차례로 돌린다
  1. `skills/sonmat-read-files` — 게임 파일 읽기(엔진·이름표·음원·원래 진동 표)
  2. `skills/sonmat-hook` — 훅 붙이고 켜진 게임에서 이벤트 검증
  3. `skills/sonmat-first-draft` — 1차본(완성본의 층 목록 `layers.md` 포함), 소리 준비·PC 반영
  4. `skills/sonmat-rumble-gap` — 원래 진동 빈자리 대조(이미 붙은 게임에도)
  5. `skills/sonmat-tune` — 첫 판 뒤 다듬기, 보정 사례집 `calibrations.md`
- `skills/sonmat-ref-study` — 기준 게임 규칙을 읽어 우리 프로필과 맞대기
- `skills/sonmat-fault-hunt` — 패드 소리·진동이 가끔 끊기는 장치 쪽 원인 찾기
- `agents/` — 위 스킬이 부르는 조사·검증 에이전트

도구는 [sonmat-maker](https://github.com/ErikKim/sonmat-maker), 게임 목록은 [sonmat-games](https://github.com/ErikKim/sonmat-games).
