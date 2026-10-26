# 엔진별 훅·음원 노하우

SKILL.md 의 A-2(훅)·A-4(음원)에서 그 게임의 엔진에 해당하는 절만 읽는다.

## 훅 표

| 엔진 | 이벤트 줄 | 쓰는 것 | 함수 찾기 |
|---|---|---|---|
| 언리얼 자체 오디오(P의 거짓) | `ue:<큐>\|<소유자>` | `dshaptics_ue4.dll`(`sonmat-hook-ue4`) | `sonmat-maker/tools/find_ue4_audio.py` |
| 소리가 **끝남**(언리얼) | `ue_end:<큐>\|<소유자>` | 같은 DLL, `hook.json` 의 `native.sigs.ActiveSoundDtor` + `native.sigs.EndFilter`(이름 조각) | `execPlaySound2D` 안에서 AddNewActiveSound 바로 다음에 같은 객체로 부르는 함수 |
| Wwise(오공, 니어) | `wid:<이벤트번호>:<오브젝트>`, `wsw:<그룹>:<값>:<오브젝트>` | `dshaptics_wwise.dll`(2022 이후 SDK, `sonmat-hook-wwise`), `dshaptics_wwise_wide.dll`(그 밖, `sonmat-hook-wwise-wide`) | `sonmat-maker/dll/find_ak.py`(파일), `find_ak_mem.py`(메모리) |
| UE4SS Lua(오공 주 흐름) | `lua:<함수>\|<소리>\|<소유자>`, `seq:` | UE4SS 모드 | 없음 |
| FMOD Ex, 별도 DLL(세키로) | `ue:<소리 이름>\|d<거리cm>`, 이름이 없으면 `ue:#<묶음의 소리 수>.<몇 번째>.<길이>\|…` | `dshaptics_fmodex.dll`(`sonmat-hook-fmodex`) + 자리→이름 표(`hook.json` 의 `native.places` = `fmod_ids.json`, 자동 셋업이 사용자의 묶음에서 만든다) | 없음 — DLL 이 내보내는 공개 함수 이름으로 |
| 언리얼 3, 32비트, FMOD Designer 정적 링크(어사일럼) | `ue:<FMOD 프로젝트>.<SoundCue>\|<주인 액터>`, 끝 `ue_end:<같은 이름>\|<주인>`(`EndFilter` 에 든 이름만) | `dshaptics_ue3_x86.dll`(`sonmat-hook-ue3`), `hook.json` 의 `native.sigs` 에 패턴 3개(`AudioComponentPlay`·`AudioComponentCleanup`·`FNameToString`)와 칸 위치(`CueOffset` 등) | `AudioComponentexecPlay` 네이티브 표 + 게임 자체 소리 목록 출력 문자열("FMOD [%s.%s]")이 쓰는 칸 |
| 영상 컷씬(Bink, 언리얼 3) | `ue:BIK.<파일 이름>\|Bink`, `ue_end:BIK.<같은 이름>\|Bink` | 같은 `dshaptics_ue3_x86.dll`(`Bink=0` 이면 끔) | BinkOpen + 바로 전에 연 `.bik` 파일 이름 |
| 유니티 Mono(오버쿡드 2) | `name:<소리>`, 멈춤 `name:Stop_<소리>`, 유니티+Wwise 는 `wid:<번호>:0` | BepInEx 플러그인 `SonmatUnity.dll`(`sonmat-unity`), `hook.json` 의 `bepinex` | 없음 — 유니티 오디오 API 에 Harmony 로 건다 |
| MAME(파이널파이트, 캐딜락) | `<프레임> snd:<번호>`, `ram:<이름>:<변화>` | `sonmat-maker/mods/mame/sonmat` 플러그인 | 소리 명령 주소 + RAM 감시(치트 목록에서 시작) |

## 소리 종료 훅
- 언리얼: `ue_end:`(위 표).
- 언리얼 3: `UAudioComponent::Cleanup` 이 "재생 중이었나" 비트를 스스로 확인하므로 거기서 받는다.
- 영상 컷씬: 영상이 닫힐 때(Bink 줄)가 끝이다. 영상과 같이 나는 큐(영상의 음악)가 영상 트랙을 갈아 끼우지 않는다.
- Wwise·FMOD Ex: 종료 훅이 아직 없다. 게임이 주는 끝 신호(`cutscene.end`)로 대신한다(니어는 컷씬 123개 중 끝 신호가 있는 게 44개).

## 32비트 게임
- 로더도 32비트 판 `dshaptics_loader_x86.dll` 을 `version.dll` 로 둔다(설치가 exe 의 비트를 보고 고른다).
- 64비트 엔진이 32비트 게임에 끼워 넣으면 주소가 틀려 게임이 죽는다. 엔진은 32비트 DLL 이면 끼워 넣기를 건너뛰고 로더만 믿는다.
- exe 가 `VERSION.dll` 을 직접 가져오는지, 다른 모드의 version.dll 이 없는지 먼저 본다.

## 별도 DLL 소리 엔진 (FMOD Ex 등)
- 가장 안전한 경우다. 공개 API 의 내보낸 함수에 걸면 패턴 탐색·인자 추측·포장된 실행 파일 읽기가 없다.
- 게임 없이 미리 시험한다: 그 DLL 을 시험 프로세스에 무음 출력으로 띄우고 훅을 붙여 호출이 통과하고 기록이 남는지 본다. 할 수 있으면 게임에 처음 붙이기 전에 반드시 한다.
- 이름이 비어 오면 자리(묶음의 소리 수·순번·길이)로 알아본다. 디코더(vgmstream)의 순서·표본 수와 런타임 값이 정확히 일치하는지로 표를 검증한다.
- 묶음 안에 저장된 이름과 이벤트 파일의 이름이 다를 수 있다. 런타임에 오는 건 묶음 안 이름이다. 도구가 옆의 이벤트 파일을 읽어 이름을 바꿔 보여 주는 경우가 있으니 묶음만 따로 놓고 한 번 더 본다.

## Wwise 함수 찾기
- 실마리는 이름→번호 계산(FNV-1, 씨앗 `0x811C9DC5`·곱수 `0x01000193`). 이름으로 부르는 함수가 계산 뒤에 부르는 함수가 번호로 받는 함수다.
- 가장 많이 불리는 후보가 답이 아니다(니어의 첫 후보는 실수값을 받는 RTPC 설정이었다). 재생 함수는 "이벤트 표를 조회하고, 대기열에 종류 1 을 넣는다"로 가린다.
- 인자 형식을 먼저 맞춘다. 다른 Wwise 버전의 함수에 기존 훅을 붙이면 포인터가 잘려 게임이 죽는다. 모르면 인자를 그대로 넘기는 `_wide` 를 쓴다.
- 바닥은 발소리와 따로 스위치로 온다. 스위치 훅(`wsw:`)이 있어야 바닥별 발소리가 된다. 암호화된 실행 파일은 게임이 켜져 있을 때만 찾을 수 있다.
- 번호→이름은 FNV-1(소문자) 해시로 대조한다. 공개 이름표에 없는 번호도 많이 온다(SKILL.md A-3).

## 음원 리더별

리더는 `sonmat/tools/soundprep/` 의 모듈이고 `READERS` 에 등록한다(지금: `ue_pak`, `ue_pak_waves`, `ue_pak_bnk`, `wwise_bnk`, `fromsoft_fsb`, `unity_audio`, `ue3_fmod`, `mame_oki`, `mame_qsound`).

### 프로젝트 파일 없는 Wwise (`wwise_bnk`)
- 뱅크는 아카이브 안에 그대로 들어 있는 경우가 많다 → `BKHD` 를 찾아 그 자리에서 읽는다. 파일별 압축(CRI `.cpk` 의 CRILAYLA)이면 파일 표를 읽고 풀어서 다시 찾는다(부품 `sonmat-crilayla`).
- "이벤트 → 음원" 연결은 뱅크 버전별 구조를 몰라도 풀린다: 이벤트는 동작 번호 목록, 동작은 대상 번호, 묶음과 소리는 서로를 가리키는 쌍이 부모·자식이다. 소리의 음원 번호 위치는 "데이터를 가진 번호가 가장 자주 나오는 자리"로 정한다.
- 긴 소리(컷씬, 환경)는 뱅크 밖 스트림 파일에 있다. 색인 파일의 레코드 형식은 실제 파일 이름·`RIFF` 위치와 대조해 **전부 맞을 때까지** 확정한다.
- 컷씬 소리 묶음의 이름 규칙을 찾는다(니어 `ev####_Se`, P의 거짓 `SE_CIN_*`). 끝·건너뛰기 신호(`ev####_End`, `se_movie_out`)는 `cutscene.end` 에 넣는다.

### 프롬 FSB (`fromsoft_fsb`)
- 게임 옆에 풀려 있는 묶음이 전부가 아니다. 옆에는 음악·환경·컷씬용 긴 소리만 있고 효과음 본체(`main.fsb`)는 아카이브 안일 수 있다. 효과음 묶음을 찾을 때까지 "음원을 꺼냈다"고 하지 않는다.
- 아카이브는 우리 코드로 풀지 않는다. 명령줄로 도는 공개 언패커의 릴리스를 라이선스부터 확인하고 사용자 PC 로 받아 쓴다(`sonmat/data/fetch_tools.json`, 주소 + SHA-256). 세키로 = BinderTool 0.6-pre4(이름표가 비어 있어 경로 해시로 고름, `fromsoft_files`), 다크소울2·3 = 0.5.2(이름표 있음).
- 음원 암호는 디코더(vgmstream)가 풀면 그걸 쓴다. 잘라 낸 묶음이 길다고 거절하면 디코더가 말해 주는 크기로 다시 자른다.
- 끝까지 번호뿐인 소리는 모양으로 가른다(`looks_like_impact`, SKILL.md A-3).

### 언리얼 3 + FMOD Designer (`ue3_fmod`)
- 소리는 패키지(`.upk`·`.umap`·`.u`) 안 `RFMODSound` 객체에 FEV1(이벤트·소리 정의) + FSB4 로 들어 있다. 패키지는 UE3 LZO 덩어리 압축 → 부품 `sonmat-lzo1x`. 키 없음.
- SoundCue 이름 = FMOD 이벤트 이름이라 훅 줄이 그대로 이름표가 된다. 프로젝트 접두어가 주인(`PL_` 나, `EN_` 적, `DV_` 내 도구), 바닥은 프로젝트 이름에 들어 온다(`PL_FootSteps_<바닥>SFX`). 이름이 대소문자를 섞어 오므로 표는 대소문자 무시.
- 소리 정의가 가리키는 음원 번호와 FSB 음원 이름을 대조한다(어긋난 것은 리더가 이름으로 다시 찾음). 주인공 소리는 대부분 공용 패키지, 장소·번호 붙은 컷씬은 각 맵 쪽 — 맵까지 읽어야 "음원을 다 꺼냈다".
- 실사 영상(.bik)은 Bink 가 소리를 직접 내서 소리 훅에 안 온다. 영상 소리는 소리 준비가 .bik 에서 따로 만들고(컷씬 `over`), 훅의 Bink 줄로 시작·끝을 잡는다.

### 공통: 음원 앞 무음
디코딩한 음원 앞에 무음이 붙어 오면(FSB4 MP3 는 소리마다 ~25 ms, 다른 형식은 0~3 ms) 패드가 늦게 느껴진다. 프로필 `source.lag_fix: true` 한 줄로 소리 준비가 앞 무음을 자른다. 파일의 성질이라 PC 마다 같다. 게임마다 새 코드를 쓰지 않는다.

### MAME
- 소리 이름이 번호뿐이다. 자동 플레이로 수집하고 소리 명령마다 화면을 저장해 대조한다(`SONMAT_SHOTS`). 캐릭터별로 같은 입력을 돌려 비교하면 공통 번호와 캐릭터별 번호가 갈린다.
- 누가 냈는지 = 공격 버튼을 누른 패드. 소리 없는 동작(잡기, 착지)은 RAM 상태로 잡는다.
- 장소 상태: 스테이지마다 자동 플레이 + 1초마다 메모리 덤프 → 그 구간에만 같은 값인 칸(`sonmat-maker/tools/mame_autoplay_dino.lua`).
