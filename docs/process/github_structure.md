
# Monthly Games GitHub 프로젝트 & 저장소 구조 설계서

> 목적: Monthly Games 사업(“한 달 1게임”)을 위해 **GitHub 프로젝트/저장소/프로젝트 보드 구조**를 표준화하고, 사람/AI 모두가 그대로 따라 쓰도록 하는 기준 문서.

---

## 1. 전체 개요

### 1.1 목표

- GitHub를 단일 진실의 원천(SSOT)으로 사용한다.
- 공통 플랫폼과 개별 게임을 분리하되, 공통 코드는 **submodule**로 재사용한다.
- Org 레벨 Project와 레포별 Project를 함께 활용해  
  **로드맵(전략)**과 **실행(전투)**를 분리해서 관리한다.
- AI는 이 구조를 전제로, 설계/코드/리뷰에 참여한다.

### 1.2 전제

- Organization 이름 예시: `monthly-games`
- 1년 동안 12개 게임 출시를 목표로 한다.
- Flutter/Flame + Firebase + GCP + GA4 + BigQuery 스택을 사용한다.

---

## 2. GitHub Organization & 저장소 목록

### 2.1 공통/플랫폼 저장소

1. `mg-meta`  
   - 비즈니스/프로세스/템플릿/규칙 정리용.
2. `mg-common-game`  
   - Flutter/Flame 기반 공통 게임 엔진, UI, 게임 시스템 모듈.
3. `mg-common-backend`  
   - Firebase Functions, Cloud Run 기반 공통 백엔드 코드.
4. `mg-common-analytics`  
   - GA4/BigQuery 이벤트 스키마, 공통 분석 쿼리, 공통 대시보드 정의.
5. `mg-common-infra`  
   - Terraform 모듈, 공통 GCP/Firebase 인프라 정의.
6. `mg-common-marketing`  
   - UA 캠페인/크리에이티브 스키마, 공통 마케팅 룰/템플릿.

### 2.2 게임별 저장소

- `mg-game-0001`
- `mg-game-0002`
- ...
- `mg-game-0012`

각 게임 레포는 공통 레포들을 **git submodule**로 포함한다.

---

## 3. `mg-meta` – 비즈니스 & 프로세스

```text
mg-meta/
  ├─ docs/
  │   ├─ business/
  │   │   ├─ vision.md              # Monthly Games 비전 / KPI
  │   │   └─ portfolio_strategy.md  # 장르/타깃 전략, Year 1 라인업 개요
  │   ├─ process/
  │   │   ├─ development.md         # 개발 프로세스, 브랜치 전략
  │   │   ├─ ai_methodology.md      # AI 개발 방법론 규칙
  │   │   └─ release_playbook.md    # 릴리즈/롤백 절차
  │   └─ templates/
  │       ├─ issue_spec_template.json
  │       ├─ gdd_base.json
  │       └─ marketing_brief.json
  ├─ .github/
  │   └─ ISSUE_TEMPLATE/
  │       ├─ idea.yml
  │       ├─ epic.yml
  │       └─ task.yml
  └─ README.md
```

---

## 4. 공통 저장소 구조

### 4.1 `mg-common-game` – 공통 게임 엔진

```text
mg-common-game/
  ├─ lib/
  │   ├─ core/
  │   │   ├─ engine/          # 루프, 씬 관리, 입력 처리
  │   │   ├─ ui/              # HUD, 팝업, 버튼, 토스트 등 공통 UI
  │   │   ├─ systems/         # 경제, 경험치, 인벤토리, 퀘스트 등 공통 시스템
  │   │   └─ analytics/       # GA4/Firebase 이벤트 트래킹 래퍼
  │   ├─ features/
  │   │   ├─ battle/          # JRPG 전투 공통 모듈
  │   │   ├─ idle/            # 방치 수익 계산, 오프라인 보상
  │   │   └─ puzzle/          # 퍼즐(매치3/블록 등) 코어 로직
  │   └─ api/
  │       └─ backend_client.dart   # 서버 통신 공통 클라이언트
  ├─ test/
  ├─ example/                 # 공통 모듈 데모용 샘플 게임
  ├─ docs/
  │   └─ design/
  │       ├─ architecture.md
  │       └─ modules.md
  ├─ pubspec.yaml
  └─ .github/workflows/ci.yml
```

### 4.2 `mg-common-backend` – 공통 백엔드

```text
mg-common-backend/
  ├─ firebase_functions/
  │   ├─ src/
  │   │   ├─ auth/
  │   │   ├─ economy/
  │   │   ├─ gacha/
  │   │   └─ telemetry/
  │   ├─ test/
  │   └─ package.json
  ├─ cloud_run/
  │   ├─ balance_simulator/
  │   └─ batch_jobs/
  ├─ shared/
  │   └─ firestore_schemas/    # 공통 Firestore 스키마 JSON
  ├─ docs/
  │   └─ api/
  │       └─ openapi.yaml
  ├─ .github/workflows/ci.yml
  └─ README.md
```

### 4.3 `mg-common-analytics` – 분석

```text
mg-common-analytics/
  ├─ schemas/
  │   ├─ events.json              # 공통 GA4 이벤트 스키마
  │   ├─ bq_events_schema.json
  │   └─ user_daily_schema.json
  ├─ queries/
  │   ├─ mart_fact_user_daily.sql
  │   ├─ mart_fact_revenue.sql
  │   ├─ retention.sql
  │   └─ funnel_tutorial.sql
  ├─ dashboards/
  │   ├─ kpi_core.json            # DAU, Retention, ARPDAU, LTV
  │   └─ marketing_roas.json
  ├─ docs/
  │   └─ metrics_definition.md
  ├─ .github/workflows/ci.yml
  └─ README.md
```

### 4.4 `mg-common-infra` – 인프라

```text
mg-common-infra/
  ├─ terraform/
  │   ├─ modules/
  │   │   ├─ firebase_project/
  │   │   ├─ bigquery_dataset/
  │   │   ├─ cloud_run_service/
  │   │   └─ monitoring_alerts/
  │   └─ env/
  │       ├─ dev/
  │       │   └─ main.tf
  │       ├─ staging/
  │       └─ prod/
  ├─ scripts/
  │   └─ plan_apply.sh
  ├─ docs/
  │   └─ infra_architecture.md
  ├─ .github/workflows/infra-ci.yml
  └─ README.md
```

### 4.5 `mg-common-marketing` – 마케팅

```text
mg-common-marketing/
  ├─ schemas/
  │   ├─ campaign_schema.json
  │   └─ creative_schema.json
  ├─ templates/
  │   ├─ hooks_kr.json             # 훅/카피 패턴 템플릿
  │   ├─ store_description_kr.md
  │   └─ store_description_en.md
  ├─ playbooks/
  │   └─ launch_30days.md          # 출시 전후 30일 마케팅 액션 플랜
  ├─ .github/workflows/marketing-ci.yml
  └─ README.md
```

---

## 5. 게임 저장소 구조 (`mg-game-000X`)

### 5.1 기본 구조

```text
mg-game-000X/
  ├─ common/
  │   ├─ game/        # submodule → mg-common-game
  │   ├─ backend/     # submodule → mg-common-backend (선택)
  │   ├─ analytics/   # submodule → mg-common-analytics (선택)
  │   └─ infra/       # submodule → mg-common-infra (선택)
  │
  ├─ game/
  │   ├─ lib/
  │   │   ├─ features/
  │   │   ├─ theme/
  │   │   └─ main.dart
  │   ├─ assets/
  │   └─ test/
  │
  ├─ backend/         # 게임 전용 백엔드 확장 (필요시)
  ├─ analytics/       # 게임 전용 이벤트/쿼리 (실험용)
  ├─ marketing/
  │   ├─ campaigns/
  │   └─ creatives/
  ├─ docs/
  │   ├─ design/
  │   │   ├─ gdd_game_000X.json
  │   │   ├─ economy.json
  │   │   └─ level_design.json
  │   └─ notes/
  ├─ config/
  │   └─ game_manifest.json
  ├─ .github/workflows/
  │   ├─ game-ci.yml
  │   ├─ backend-ci.yml
  │   └─ analytics-ci.yml
  └─ README.md
```

### 5.2 `game_manifest.json` 예시

```json
{
  "game_id": "game_0001",
  "code_name": "MG-0001",
  "title_kr": "던전 타이쿤 퍼즐RPG",
  "genre_tags": ["puzzle", "idle", "jrpg"],
  "target_region": ["KR", "SEA"],
  "engine_version": "1.0.0",
  "analytics_profile": "default_v1"
}
```

---

## 6. git submodule 운용 규칙

### 6.1 추가 방법

각 게임 레포에서:

```bash
git submodule add git@github.com:monthly-games/mg-common-game.git common/game
git submodule add git@github.com:monthly-games/mg-common-backend.git common/backend
git submodule add git@github.com:monthly-games/mg-common-analytics.git common/analytics
git submodule add git@github.com:monthly-games/mg-common-infra.git common/infra
```

### 6.2 업데이트 방법 (공통 모듈 버전 반영)

1. 공통 레포(`mg-common-*`)에서 변경 → 태그/릴리즈 (`v1.1.0`) 생성.
2. 게임 레포에서:

```bash
cd common/game
git checkout v1.1.0
cd ../..
git add common/game
git commit -m "chore(common): bump mg-common-game to v1.1.0"
```

3. PR 생성 → CI 빌드/테스트 확인 → 머지.

---

## 7. GitHub Projects 구조

### 7.1 Org Project #1 – `Monthly Games – Roadmap Y1`

- 목적:  
  12개 게임 + 공통 플랫폼의 **연간 로드맵/단계** 관리.

#### 7.1.1 대상 아이템

- 각 게임 레포의 **Epic 이슈**
- 공통 레포의 **Platform Epic 이슈**

#### 7.1.2 필드

| 필드 이름   | 타입   | 예시 값                                    |
|------------|--------|--------------------------------------------|
| `GameID`   | Text   | `game_0001`, `platform`                    |
| `Quarter`  | Single | `Q1`, `Q2`, `Q3`, `Q4`                      |
| `Lifecycle`| Single | `Idea`, `Design`, `Dev`, `QA`, `Live`, `Retire` |
| `Impact`   | Single | `High`, `Medium`, `Low`                    |
| `Effort`   | Single | `S`, `M`, `L`, `XL`                         |
| `Owner`    | Text   | 담당자/팀 이름                             |

#### 7.1.3 주요 뷰

- `Roadmap by Quarter` (타임라인)
- `By Game` (칸반 – 칼럼 = Lifecycle)
- `Platform only` (GameID = platform 필터)

---

### 7.2 Org Project #2 – `Platform & Common`

- 목적:  
  `mg-meta`, `mg-common-*` 관련 작업 전체 관리.

#### 7.2.1 필드

| 필드 이름       | 타입   | 예시 값                                   |
|-----------------|--------|-------------------------------------------|
| `Area`          | Single | `engine`, `backend`, `analytics`, `infra`, `marketing`, `meta` |
| `Status`        | Single | `Todo`, `Doing`, `Review`, `Done`        |
| `Priority`      | Single | `P0`, `P1`, `P2`, `P3`                   |
| `GamesAffected` | Text   | `0001,0002` 또는 `all`                   |

#### 7.2.2 주요 뷰

- `By Area` (Status 칼럼, Area 그룹)
- `High Priority` (P0/P1 필터)
- `Game Impact` (특정 GameID 포함 필터)

---

### 7.3 게임 레포별 Project – `MG-000X – Delivery`

#### 7.3.1 목적

- 각 게임의 **실제 개발/QA/릴리즈 실행 보드**.

#### 7.3.2 필드

| 필드 이름    | 타입   | 예시 값                                   |
|--------------|--------|-------------------------------------------|
| `Stage`      | Single | `Spec`, `Implementation`, `QA`, `Release` |
| `Type`       | Single | `feature`, `bug`, `task`, `chore`         |
| `Area`       | Single | `game`, `backend`, `analytics`, `marketing` |
| `Sprint`     | Text   | `2025-W01`, `2025-W02` 등                  |
| `Complexity` | Single | `S`, `M`, `L`                             |

#### 7.3.3 칼럼

- `Todo` → `Doing` → `Review` → `QA` → `Done`

---

## 8. 이슈/PR 네이밍 & 템플릿 규칙

### 8.1 이슈 제목

- 공통 레포:

  - `[platform][engine][epic] 공통 전투 시스템 v1`
  - `[platform][analytics][task] LTV 계산 쿼리 정리`

- 게임 레포:

  - `[game_0001][game][feature] 튜토리얼 스테이지 구성`
  - `[game_0001][backend][bug] 가챠 결과 저장 오류`

### 8.2 이슈 본문 `spec_json` 예시

```json
{
  "task_id": "GAME-0001-123",
  "game_id": "game_0001",
  "area": "game",
  "type": "feature",
  "spec_json": {
    "goal": "튜토리얼 스테이지 3까지 3분 내 완료 가능",
    "acceptance_criteria": [
      "튜토리얼 완료 시 tutorial_complete 이벤트 발생",
      "새 유저 기준 80% 이상 튜토리얼 완료"
    ],
    "target_files": ["game/lib/features/tutorial/**"],
    "non_goals": [
      "튜토리얼 스토리 퀄리티 개선"
    ],
    "related_docs": [
      "docs/design/gdd_game_0001.json#tutorial"
    ],
    "constraints": [
      "공통 이벤트 스키마 default_v1을 사용"
    ]
  }
}
```

### 8.3 PR 템플릿 골격

```markdown
## Summary
- 무엇을, 왜 했는지 한 줄 요약

## Changes
```json
{
  "task_id": "GAME-0001-123",
  "game_id": "game_0001",
  "modules_touched": [
    "game/lib/features/tutorial/**"
  ],
  "breaking_changes": false
}
```

## Acceptance Criteria Check
- [ ] AC1: ...
- [ ] AC2: ...

## Tests
- [x] flutter test
- [ ] custom scenario: ...

## Risks / Assumptions
- Risks:
  - ...
- Assumptions:
  - ...

## Self Review Notes
- ...
```

---

## 9. Year 1 게임 라인업(요약)

Year 1 기준, 12개 게임 레포와 장르/역할은 다음과 같다.

| 코드 | 레포명          | 가제(한글)                 | 코어 장르 조합                          |
|------|-----------------|----------------------------|-----------------------------------------|
| 1    | mg-game-0001    | 던전 타이쿤 퍼즐RPG        | 퍼즐 + 방치형 + 라이트 JRPG             |
| 2    | mg-game-0002    | 고양이 연금술 공방         | 퍼즐 제작/합성 + 방치 생산              |
| 3    | mg-game-0003    | 픽셀 용병단 키우기         | 방치형 + JRPG 전투(자동 전투)          |
| 4    | mg-game-0004    | 카페 매치 타이쿤           | 매치3 + 경영 시뮬 + 가벼운 방치        |
| 5    | mg-game-0005    | 로그라이크 퍼즐 던전       | 턴제 퍼즐 + 로그라이크                  |
| 6    | mg-game-0006    | 히어로 오토 배틀 아레나    | 자동 전투 + 덱 빌딩 + 가벼운 PVP        |
| 7    | mg-game-0007    | 픽셀 농장 방치RPG          | 농장 방치 + 퀘스트 + 라이트 JRPG        |
| 8    | mg-game-0008    | 타임루프 마을 방어전       | 디펜스 + 방치 + 타임루프                |
| 9    | mg-game-0009    | 영웅 수집 카드 퍼즐        | 카드 수집 + 퍼즐 전투                  |
| 10   | mg-game-0010    | 던전 샵 시뮬레이터         | 상점 경영 + 방치 생산 + 간단 전투       |
| 11   | mg-game-0011    | 요정 숲 힐링 아이들        | 힐링 방치 + 꾸미기 + 소셜 퀘스트        |
| 12   | mg-game-0012    | 연말 레이드 이벤트RPG      | 연속 레이드 + 이벤트 중심 JRPG         |

---

## 10. 이 문서를 사용하는 방법

1. `mg-meta` 레포의 `/docs/process/` 아래에  
   이 문서를 `github_structure.md` 이름으로 저장한다.
2. 새 레포를 만들 때마다:
   - 여기 정의된 디렉터리 구조를 그대로 복붙한다.
   - 필요 시 일부 디렉터리는 비워두고 시작해도 된다.
3. AI/에이전트에게 레포를 설명할 때:
   - 이 문서의 링크 또는 핵심 구조만 전달하고,
   - 세부 컨텍스트는 이슈/PR/템플릿에 `spec_json`으로 넘긴다.
