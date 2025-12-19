# 문서 동기화 프로세스

## 목적
- 게임 콘셉트 변경 시 문서 불일치 방지
- GDD 또는 Manifest를 기준으로 모든 문서 메타데이터를 일치
- 문서 업데이트 흐름을 표준화하여 누락을 줄임

## 적용 범위
- 대상: `mg-game-0001` ~ `mg-game-0036`
- Year 1/2 문서
  - `docs/fun_design.md`
  - `docs/bm_design.md`
  - `docs/monetization_design.md`
  - `docs/ops_design.md`
- Level A 추가 문서
  - `docs/character_bible.md`
  - `docs/world_bible.md`
  - `docs/art_style.md`
  - `docs/enemy_design.md`
  - `docs/scene_structure.md`
- `README.md`는 선택 적용(권장)

## SSOT(단일 진실 소스) 우선순위
1. `docs/design/gdd_game_XXXX.json` (Year 1/2 기본)
2. `config/game_manifest.json` (존재 시 우선)
3. `mg-meta/docs/business/levelA_lineup.md` (Level A 메타 참고)
4. 위 파일이 없으면 `README.md`를 임시 SSOT로 사용하고, 반드시 SSOT 파일을 생성

## 헤더 표준
- 모든 문서는 제목 바로 아래에 메타 헤더를 둔다.
- 템플릿: `mg-meta/docs/templates/doc_header.md`

필수 필드:
- `game_id`
- `repo`
- `title_kr`
- `title_en`
- `genre_tags`
- `doc_type`
- `source_of_truth`
- `last_updated`

## 변경 체크리스트
1. SSOT 업데이트 (GDD 또는 Manifest 또는 Lineup)
2. 관련 문서 내용 수정
3. 모든 문서 헤더 갱신 (`last_updated` 포함)
4. 자동 검증 스크립트 실행
5. 경고/오류 해결 또는 사유 기록

## 자동 검증 스크립트
경로: `scripts/verify_doc_consistency.py`

기본 실행:
```
python scripts/verify_doc_consistency.py
```

보고서 저장:
```
python scripts/verify_doc_consistency.py --report DOC_SYNC_REPORT.md
```

옵션:
- `--root`: 게임 저장소 루트 경로 지정
- `--include-readme`: README 포함 검사
- `--strict`: 경고를 오류로 승격

## 검사 규칙 요약
- `game_id`, `repo` 불일치: ERROR
- `title_kr`, `title_en`, `genre_tags` 불일치: ERROR
- `doc_type`, `source_of_truth`, `last_updated` 누락: WARN (strict 시 ERROR)
