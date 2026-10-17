# 보정 사례집

게임마다 사용자가 직접 해 보고 고친 것(칼리브레이션)을 모은 것. **자동 셋업(`tools/autosetup.py`)이 만든 1차본에 "비슷한 상황이면 이렇게 고쳤다"를 제안하는 데 쓴다** — 자동으로 적용하지 않는다(사용자 10-15: "칼리브레이션 된 데이터는 우리 스킬에 녹여서, 비슷한 상황의 예외케이스 제안에 넣는걸로 가자").

한 줄 = 상황 → 한 것 → 출처. 새 보정이 생기면 여기에 상황을 일반화해서 더한다(게임 이름이 아니라 "어떤 소리·어떤 순간"으로).

## 일반화한 사례 (손으로 정리)

- **원음 가운데가 꺼졌다 다시 올라오는 긴 소리**(갈기·연마처럼 쭉 이어지는 동작): 시작과 끝은 원음, 가운데는 사인파로 쭉 민다 — P의 거짓 그라인딩(사용자 10-15, 아직 반영 안 함)
- **늦게 올라오는 휘두름 + 타격 처리의 앞 0.2 초 자르기**: 휘두름이 거의 안 남는다. 단독 예외 대신 한계로 두거나 세부 보정 — 오공 3단 내려치기 "휭휭탁"(사용자 10-15)
- **멀리서 나는 기기 소리가 패드 스피커로 바로 옆처럼 남**: 내 마지막 소리와의 거리로 줄인다(8 m 안 전부, 25 m 밖 없음) — P의 거짓 공중전화(사용자 10-11)
- **패드 스피커에 전투 소리가 계속 남**: 이름 붙은 순간만 허용(`speaker.allow`), "all" 금지 — 니어 비행 유닛 칼(사용자 10-11)
- **원래 게임이 진동하는 순간인데 우리 칸에 없음**(배가 흔들림·지진·포효·순간이동·계속 나는 불이나 분사): 원래 진동 표에서 순간을 찾아 모델 칸(`quake`·`roar`·`warp`·`hold`)에 넣는다. 파형은 그 칸의 우리 것 — 오버쿡드 2 스위치 표 15종(사용자 10-16)
- **모으기(차지)가 단계별로 약해짐**: 단계마다 세기를 옛 클립 최대치에 맞춤 — 오공 기모으기(사용자 10-14, 이후 되돌림: 예외 대신 정규 규칙 선호)

## 프로필에 남은 사용자 메모 (자동 수집 10-15, 원문)

- `dino` `(top)`: states that last: the wave repeats on that player's pad from on until off. User switch: the weather one
- `ffight` `clips`: impacts: the arcade sample itself, band-limited, as the vibration (recipe when the user has not prepared sounds)
- `liesofp` `source`: tools/soundprep.py liesofp: the user's own paks -> cue trees, haptic clips, pad-speaker sounds. Key = read from the user's own game on first use (tools/keyfetch.py), kept in tools/config.json, never in this file
- `liesofp` `speaker.near`: user 10-11: a far phone booth rang on the pad as if it were next to me. Distance between the ring and my own last sound: full within 8 m, nothing beyond 25 m (first guess, to be tuned by ear)
- `liesofp` `rules[^SE_PC_MT_Arm_GrappleGun_Start]`: Puppet String (user 10-09): shot = a soft whoosh, catch = two thuds, on the pull and on the fly-in alike. No buzz. synth = play this wave instead of the vibration made from the sound (that one clicked)
- `liesofp` `triggers`: 10-09 (user: is a buzz right for a channelled arm?): trigger = force, handle = texture. Fulminis: the pull gets stiffer with each charge level the game announces (Start_Normal -> Charge_Lv2_Start -> Lv3), and lets go the moment it fires. Flamberge: a steady stiff pull while the flame is on. No 'when': these FX names only play when the arm is really used. Lv3 name is a guess (only Lv1/Lv2 seen in the log)
- `liesofp` `triggers`: the rest of the arms by the same rule (user: 'do I have to explain every one?'), from the pak's sound names, none felt yet. A hit on the trigger = 'vibrate' at 8-9 Hz for 0.13 s (one kick) or 0.22 s (two). Names that turned out to be shared arm-motion sounds and must not be used alone: MT_Arm_SniperCannon_(End|Recoil|Aim), MT_Arm_FlameThrower_(Start|End) (= any arm starts / ends), FX_Aegis_Counter_01_Stop (plays on perfect guard)
- `liesofp` `triggers.deus_hold`: Deus Ex Machina (user 10-09): heavy from the start, then two thuds as the mine goes into the ground. The heavy start needs the fitted arm to be known (arm-change screen opened once since the engine started)
- `nier` `source`: tools/soundprep.py nier: banks found inside the user's own game archives (no key), events -> sounds read from the banks, names from the public list
- `nier` `speaker`: 2026-10-11 (player, first run to the first boss: 'the pad speaker on the sword in the shooting part is too much'): was all = true, and the flight unit's blade swing shares two sound files with pl0010_just_avoid, so every swing up there rang on the speaker. Now only the named moments
- `sekiro` `native`: 2026-10-11 first real run: the game opens its banks without names (every sound came as (null)), so the hook reports a sound by its place and tools/fmod_ids.py makes this table from the user's own banks
- `wukong` `source`: tools/soundprep.py, all from the user's own paks (Wwise .wem by source id). Each set = a table of this folder and what is made of its sounds: out = the sound, hap = its haptic clip (band 30-500 Hz, one gain for all sets, kept in hap_gain.json), track = a cutscene's small track. thin = a sound with nothing in the haptic band gets its loudness curve on a carrier. Clips made before 2026-10-14 came from the Sound2Hap model and are left as they are until their folder is deleted. The pak is encrypted: tools/keyfetch.py reads the key from the user's own game on first use, it stays in tools/config.json
- `wukong` `(top)`: read by hapticsd.py. Edits apply live within 3 s; clips reload by themselves when a clip-shaping section changes. Clip folders are made on the user's PC from their own game files (tools/), never shipped
