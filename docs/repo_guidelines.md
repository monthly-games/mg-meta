
# Monthly Games 저장소 가이드라인 모음

> 목적: Monthly Games 조직의 **모든 GitHub 저장소**에 공통으로 적용할 운영 규칙과,  
> 각 저장소별 역할/사용법/금지사항을 정리한 기준 문서.

---

## 0. 공통 운영 원칙

### 0.1 공통 용어

- **플랫폼 저장소(공통)**  
  - `mg-meta`  
  - `mg-common-game`  
  - `mg-common-backend`  
  - `mg-common-analytics`  
  - `mg-common-infra`  
  - `mg-common-marketing`
- **게임 저장소**  
  - `mg-game-0001` ~ `mg-game-0012`

### 0.2 공통 규칙 요약

1. **GitHub = 단일 진실의 원천(SSOT)**  
   - 모든 결정/설계/변경 이력은 GitHub 이슈, PR, 문서에 남긴다.
2. **브랜치 구조 통일**
   - `main` : 항상 프로덕션 기준
   - `develop` : 통합/스테이징 기준
   - `feature/*` : 기능 단위 브랜치
   - `hotfix/*` : 긴급 수정
3. **이슈 → 브랜치 → PR 순서 강제**
   - 이슈 없이 직접 main/develop 수정 금지.
4. **작은 PR, 한 가지 목적**
   - 하나의 PR에 기능 추가/리팩터링/포맷 변경을 섞지 않는다.
5. **테스트 없는 핵심 변경 금지**
   - 코어 시스템/공통 모듈 변경 시 최소 1개 이상의 테스트 추가/수정 필수.
6. **AI는 “명세 우선, 코드 후”**
   - 구조화된 spec(예: `spec_json`)을 먼저 만들고, 그다음에 코드/리뷰를 맡긴다.

---

## 1. `mg-meta` 저장소 가이드라인

### 1.1 역할

- Monthly Games 사업의 **비전, 전략, 프로세스, 템플릿**을 관리하는 저장소.
- 코드 X, 문서/템플릿 중심.

### 1.2 디렉터리 구조 규칙

- `docs/business/`
  - `vision.md` : 사업 비전, 1~3년 목표, 핵심 KPI.
  - `portfolio_strategy.md` : 장르/타깃/지역 전략, 연간 라인업 개요.
- `docs/process/`
  - `development.md` : 개발 프로세스, 브랜치 전략, 코드 리뷰 정책.
  - `ai_methodology.md` : AI 개발 방법론(원칙, 안티패턴, 역할 분담).
  - `release_playbook.md` : 릴리즈/롤백/핫픽스 절차.
- `docs/templates/`
  - `issue_spec_template.json` : 이슈 `spec_json` 형식.
  - `gdd_base.json` : 공통 GDD 스켈레톤 템플릿.
  - `marketing_brief.json` : 마케팅 브리프 템플릿.

### 1.3 이슈 사용 규칙

- **타입**
  - `idea` : 사업/제품 아이디어.
  - `process` : 프로세스 개선/변경.
  - `doc` : 문서 추가/수정.
- **제목 규칙 예시**
  - `[idea] Year 1 KR 타깃 장르 전략`
  - `[process] 코드 리뷰 규칙 개정`
- **본문**
  - 가능하면 `spec_json` 블록 포함:

```json
{
  "goal": "",
  "scope": [],
  "non_goals": [],
  "deadline": "",
  "related_docs": []
}
```

### 1.4 PR 규칙

- PR은 항상 관련 이슈 1개 이상과 연결.
- 문서 변경 시:
  - 변경 이유, 적용 범위, 더 읽을 문서 링크를 PR 설명에 명시.

---

## 2. `mg-common-game` 저장소 가이드라인

### 2.1 역할

- 모든 게임이 공유하는 **Flutter/Flame 기반 공통 게임 엔진**.
- 코어 루프, 전투, 퍼즐, 방치, UI, 공통 시스템(경제/인벤토리/퀘스트 등)을 제공.

### 2.2 설계 원칙

1. **하위 호환성 우선**
   - 이미 출시/개발 중인 게임이 사용하는 API는 함부로 깨지지 않게 설계.
2. **공용 API vs 내부 구현 분리**
   - `lib/core/public/**` : 게임에서 직접 사용할 공용 API.
   - `lib/core/internal/**` : 게임에서 직접 참조하면 안 되는 내부 구현.
3. **모듈화**
   - `core/` : 엔진, UI, 시스템
   - `features/` : 전투, 퍼즐, 방치 등 기능 모듈

### 2.3 버전 정책

- 태그 예시: `v1.0.0`, `v1.1.0`
- **Major 변경** (v1 → v2) 기준:
  - 공용 API 시그니처 삭제/변경.
  - 데이터 포맷 호환 불가 변경.
- **Minor 변경** (v1.0 → v1.1) 기준:
  - 기능 추가, 버그 수정, backwards-compatible 변경.

### 2.4 이슈/PR 규칙

- 이슈 제목 예시:
  - `[engine][feature] 씬 전환 트랜지션 추가`
  - `[system][bug] 경험치 계산 오류`
- PR 시 필수:
  - 변경 모듈 목록.
  - 영향 받는 게임 예측(`GamesAffected`: `all` 또는 게임 ID 리스트).
  - 테스트 결과 요약.

### 2.5 테스트

- 변경 시 최소:
  - 관련 모듈 유닛 테스트 1개 이상 추가/수정.
  - `example/` 샘플 빌드가 깨지지 않도록 확인.

---

## 3. `mg-common-backend` 저장소 가이드라인

### 3.1 역할

- 공통 **Firebase Functions / Cloud Run** 백엔드.
- 인증, 경제, 가챠, 텔레메트리, 간단한 배치 작업 담당.

### 3.2 설계 원칙

1. **환경 분리**
   - 각 프로젝트(dev/staging/prod)는 환경 변수/설정 파일로 구분.
2. **게임별 분기**
   - Firestore 컬렉션/문서 구조에서 `game_id` 필드로 분기.
3. **보안/검증**
   - 모든 외부 입력은 검증 후 처리.
   - 민감한 값은 Secret Manager 사용.

### 3.3 이슈/PR 규칙

- 이슈 제목 예시:
  - `[functions][feature] 공통 결제 검증 로직`
  - `[cloud_run][task] 밸런스 시뮬레이터 배포`
- PR 시 필수:
  - API 스펙 변경이 있으면 `docs/api/openapi.yaml` 업데이트.
  - Firestore 스키마 변경 시 `shared/firestore_schemas/` 업데이트.

### 3.4 테스트

- Functions:
  - `npm test` 필수.
  - 주요 엔드포인트별 통합 테스트 권장.
- Cloud Run:
  - 단위 테스트 + 최소 한 개의 e2e 시나리오 (예: HTTP 호출 확인).

---

## 4. `mg-common-analytics` 저장소 가이드라인

### 4.1 역할

- GA4/BigQuery 기반 **공통 이벤트/지표 시스템** 관리.
- 이벤트 스키마, ETL 쿼리, KPI 뷰, 대시보드 정의.

### 4.2 설계 원칙

1. **이벤트 최소 핵심 세트 유지**
   - `login`, `session_start`, `stage_*`, `purchase`, `ad_*`, `tutorial_*` 등.
2. **버전 관리**
   - `events_v1`, `events_v2`처럼 스키마 버전을 명확히 구분.
3. **파생 테이블 분리**
   - `mg_raw` (원시) vs `mg_mart` (마트) 구분.

### 4.3 이슈/PR 규칙

- 이슈 제목 예시:
  - `[schema][feature] 신규 이벤트 stage_retry 추가`
  - `[query][task] D7 Retention 쿼리 최적화`
- PR 시 필수:
  - 스키마 변경 시 영향 받는 게임/쿼리 목록 명시.
  - 쿼리 변경 시 dry-run 결과/코스트 참고 정보 첨부.

### 4.4 테스트

- BigQuery 쿼리:
  - CI에서 `--dry_run` 검사.
- 스키마:
  - JSON Schema로 유효성 검증.

---

## 5. `mg-common-infra` 저장소 가이드라인

### 5.1 역할

- GCP/Firebase/BigQuery 인프라를 **Terraform**으로 관리.

### 5.2 설계 원칙

1. **모듈화**
   - 공통 리소스는 `modules/` 아래 모듈 단위로 정의.
2. **환경별 디렉터리**
   - `env/dev`, `env/staging`, `env/prod`에 환경별 설정 분리.
3. **수정 프로세스**
   - 항상 `terraform plan` → 리뷰 → `apply` 순서.

### 5.3 이슈/PR 규칙

- 이슈 제목 예시:
  - `[infra][feature] BigQuery mart 데이터셋 추가`
  - `[infra][task] dev/staging 분리`
- PR 시 필수:
  - `plan` 결과를 PR 코멘트 또는 첨부 파일로 남김.

---

## 6. `mg-common-marketing` 저장소 가이드라인

### 6.1 역할

- UA 캠페인/크리에이티브 템플릿, 훅/카피 패턴, 런칭 플랜 관리.

### 6.2 설계 원칙

1. **JSON 기반 정의**
   - 캠페인, 크리에이티브는 모두 JSON으로 구조화.
2. **공통 훅 세트 유지**
   - 실패 모음, 도전, 심리 자극 등 패턴은 `hooks_kr.json`에서 관리.

### 6.3 이슈/PR 규칙

- 이슈 제목 예시:
  - `[campaign][template] 퍼즐RPG KR 런칭 기본안`
  - `[creative][template] 실패 모음 20초 영상 스크립트`
- PR 시 필수:
  - JSON Schema 기준 유효성 검사 통과.

---

## 7. 게임 저장소(`mg-game-000X`) 가이드라인

### 7.1 역할

- 각 게임의 **실제 구현/운영 단위**.
- 공통 모듈(submodule)을 활용해 빠르게 게임을 조립.

### 7.2 디렉터리 구조 규칙

- `common/` : submodule로 포함된 공통 코드.
- `game/` : 이 게임 전용 코드/리소스.
- `backend/` : 게임 전용 백엔드 (필요시).
- `analytics/` : 게임 특화 이벤트/쿼리 (필요시).
- `marketing/` : 캠페인/크리에이티브 정의.
- `docs/design/` : GDD, 경제, 레벨 디자인 등.

### 7.3 브랜치 & 릴리즈

- 브랜치:
  - `main` : 프로덕션/라이브 빌드 기준.
  - `develop` : dev/staging 빌드 기준.
  - `feature/*`, `hotfix/*` 공통 규칙 동일.
- 태그:
  - `game_000X-v1.0.0` 형식으로 릴리즈 태그.

### 7.4 이슈 규칙

- 제목 예시:
  - `[game_0001][game][feature] 챕터1 보스 스테이지`
  - `[game_0001][analytics][task] 튜토리얼 퍼널 이벤트 추가`
- 본문:
  - `spec_json` 필수 (템플릿은 `mg-meta/docs/templates/issue_spec_template.json` 참고).

### 7.5 PR 규칙

- PR 설명에 필수 항목:
  - 관련 이슈 ID.
  - 변경 범위(파일/모듈 리스트).
  - Acceptance Criteria 체크.
  - 테스트 결과(명령/결과).

### 7.6 submodule 사용 규칙

1. **버전 고정**
   - 공통 모듈은 항상 태그 또는 특정 커밋으로 고정.
2. **업데이트 절차**
   - 공통 레포에서 새 버전 태그 → 게임 레포에서 submodule checkout → PR 생성 → CI 통과 후 머지.

---

## 8. GitHub Projects 사용 가이드 (요약)

### 8.1 Org Project – `Monthly Games – Roadmap Y1`

- 사용 대상:
  - 각 게임의 Epic 이슈.
  - 플랫폼 Epic 이슈.
- 필드:
  - `GameID`, `Quarter`, `Lifecycle`, `Impact`, `Effort`, `Owner`.
- 컬럼:
  - `Lifecycle` 기준: `Idea → Design → Dev → QA → Live → Retire`.

### 8.2 Org Project – `Platform & Common`

- 사용 대상:
  - `mg-meta`, `mg-common-*` 이슈.
- 필드:
  - `Area`, `Status`, `Priority`, `GamesAffected`.

### 8.3 게임별 Project – `MG-000X – Delivery`

- 사용 대상:
  - 해당 게임 레포의 이슈/PR.
- 필드:
  - `Stage`, `Type`, `Area`, `Sprint`, `Complexity`.
- 컬럼:
  - `Todo → Doing → Review → QA → Done`.

---

## 9. AI 협업 가이드라인 (저장소 공통)

### 9.1 AI가 해도 되는 일

- GDD 스켈레톤 생성 (`gdd_base.json` → `gdd_game_000X.json`).
- 설계 문서 초안 / 리뷰 (RFC 구조).
- 보일러플레이트 코드/테스트 코드 생성.
- 이슈 `spec_json` 기반 Task 쪼개기/PR 템플릿 자동 작성.
- 버그 리포트/로그 요약, 테스트 케이스 제안.
- 캠페인/크리에이티브 템플릿 초안 생성.

### 9.2 AI가 하면 안 되는 일 (사람 확인 필수)

- 공통 모듈의 **breaking change** 결정/머지.
- 데이터 삭제/마이그레이션 스크립트 최종 실행.
- 보안/결제/개인정보에 직접 영향을 주는 변경의 최종 승인.
- 프로덕션 인프라에 직접 apply (`terraform apply`, prod 배포 등).

### 9.3 기본 워크플로우

1. 사람이 이슈 생성 (`spec_json` 작성).
2. AI가 설계/코드/테스트 초안 또는 리뷰 수행.
3. 사람이 최종 검토 및 머지/배포 결정.

---

이 문서는 `mg-meta/docs/process/repo_guidelines.md` 또는  
비슷한 경로에 그대로 복사해 기준 문서로 사용하면 된다.
