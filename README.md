# codex_pet_make

Codex 앱에서 사용할 커스텀 Pet을 만들고 정리하는 작업 공간입니다.

## 바로 사용하기

이미 생성된 Pet은 Codex 로컬 설정 폴더에 설치되어 있습니다.

```text
~/.codex/pets/kingking/
~/.codex/pets/booriboorimon/
```

Codex 앱에서 적용하려면:

1. Codex 앱을 엽니다.
2. `Settings > Appearance > Pets`로 이동합니다.
3. custom pets를 새로고침합니다.
4. `kingking` 또는 `부리부리몬`을 선택합니다.
5. `/pet`, `Wake Pet`, 또는 `Tuck Away Pet` 명령으로 화면 위 floating overlay를 켜고 끕니다.

## 산출물 구조

Pet별 산출물은 아래에 정리합니다.

```text
artifacts/pets/<pet-id>/
├── install/   # ~/.codex/pets/<pet-id>/ 에 들어가는 최종 설치 파일
├── final/     # hatch-pet 파이프라인이 만든 최종 spritesheet
├── qa/        # contact sheet, gif preview, validation 결과
└── source/    # 생성에 사용한 원본 참고 이미지
```

현재 Pet:

- `artifacts/pets/kingking` — 왕관 쓴 흰색 캐릭터
- `artifacts/pets/booriboorimon` — 부리부리몬 돼지 캐릭터

## Codex Pet 규격

Codex Pet은 고정 atlas 규격을 사용합니다.

- 최종 파일: `spritesheet.webp`
- 크기: `1536x1872`
- 격자: 8 columns x 9 rows
- 셀 크기: `192x208`
- 배경: transparent
- 설치 manifest: `pet.json`

각 row는 Codex 앱의 상태와 연결됩니다.

```text
0 idle
1 running-right
2 running-left
3 waving
4 jumping
5 failed
6 waiting
7 running
8 review
```

프레임 수와 재생 시간은 앱 계약에 고정되어 있으므로 임의로 row나 frame을 더 추가해서 모션을 늘리는 방식은 권장하지 않습니다. 움직임이 너무 빠르거나 어색하면 프레임 사이의 포즈 변화량을 줄여서 체감 속도를 낮춥니다.

## 수정할 때 주의할 점

- 캐릭터 identity를 먼저 고정합니다.
  - 예: `kingking`은 눈썹 금지, 왕관/분홍 귀/큰 눈 유지.
  - 예: `booriboorimon`은 굵은 눈썹/큰 주둥이/보라색 하의 유지.
- 텍스트, 말풍선, 그림자, 먼지, 속도선, 분리된 이펙트는 넣지 않습니다.
- 소품이 이상하게 움직이면 소품을 제거하거나, 모든 row에서 같은 위치/형태로 강하게 고정합니다.
- 문제가 있는 경우 전체를 다시 만들기보다 실패한 row만 재생성합니다.
- 최종 적용 전 `qa/contact-sheet.png`와 `qa/previews/*.gif`를 확인합니다.

## 검증 명령 예시

`hatch-pet` 스킬의 스크립트를 사용해 최종 spritesheet를 검증합니다.

```bash
RUN_DIR=/path/to/run
SKILL_DIR="$HOME/.codex/skills/hatch-pet"

.venv/bin/python "$SKILL_DIR/scripts/extract_strip_frames.py" \
  --decoded-dir "$RUN_DIR/decoded" \
  --output-dir "$RUN_DIR/frames" \
  --states all \
  --method auto

.venv/bin/python "$SKILL_DIR/scripts/inspect_frames.py" \
  --frames-root "$RUN_DIR/frames" \
  --json-out "$RUN_DIR/qa/review.json" \
  --require-components

.venv/bin/python "$SKILL_DIR/scripts/compose_atlas.py" \
  --frames-root "$RUN_DIR/frames" \
  --output "$RUN_DIR/final/spritesheet.png" \
  --webp-output "$RUN_DIR/final/spritesheet.webp"

.venv/bin/python "$SKILL_DIR/scripts/validate_atlas.py" \
  "$RUN_DIR/final/spritesheet.webp" \
  --json-out "$RUN_DIR/final/validation.json"
```

검증 결과의 `errors`와 `warnings`가 비어 있어야 안정적으로 사용할 수 있습니다.

## 재사용 가능한 Skill

커스텀 Codex Pet 제작/수정 베스트 프랙티스는 별도 skill로도 정리했습니다.

```text
artifacts/skills/codex-pet-best-practices.skill
artifacts/skills/codex-pet-best-practices.SKILL.md
```

이 skill은 다음 작업에 사용할 수 있습니다.

- 이미지 기반 커스텀 Codex Pet 만들기
- 기존 Pet의 row 단위 repair
- 움직임이 너무 빠른 Pet의 체감 속도 조정
- contact sheet / GIF preview 기반 QA
- `~/.codex/pets/<pet-id>` 설치 패키징
- `artifacts/pets/<pet-id>` 산출물 정리
