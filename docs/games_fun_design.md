
# Monthly Games 24종 포트폴리오 개요 & 재미 설계 요약

> 목적: Year 1 (0001~0012), Year 2 (0013~0024) 24개 게임에 대해  
> 공통 형식으로 **기본 정보 + 역할 + 재미 축(Fun Pillars) + 재미 레버(Design Levers)**를 정리한 문서.  
> `mg-meta/docs/business/portfolio_24_games.md` 로 사용 가능.

---

## 공통 필드 정의

각 게임은 다음 필드로 정리한다.

- `game_id` / `repo`
- `title_kr` : 가제(한글)
- `genre_tags` : 코어 장르 조합
- `portfolio_role` : 포트폴리오 내 역할/포지션
- `core_loop` : 핵심 루프 요약
- `fun_pillars` : 재미 축(3개 내외)
- `design_levers` : 재미를 고도화하기 위한 구체 설계 레버

---

# Year 1 – 0001 ~ 0012

---

## game_0001 – 던전 타이쿤 퍼즐RPG

- `game_id` : game_0001  
- `repo` : mg-game-0001  
- `title_kr` : 던전 타이쿤 퍼즐RPG  
- `genre_tags` : ["puzzle", "idle", "jrpg"]  
- `portfolio_role` :  
  - 퍼즐+방치+JRPG 공통 루프 검증용 레퍼런스 타이틀  
  - 공통 엔진 v1의 대표 샘플

### core_loop

1. 퍼즐 스테이지 플레이  
2. 전투 진행 및 보상 획득  
3. 던전/영웅/시설 강화  
4. 방치 수익 수령  
5. 상위 던전/보스 도전

### fun_pillars

- 성장/수집 (Growth)  
- 숙련/판단 (Mastery)  
- 도전/위험 (Challenge)

### design_levers

- 퍼즐 스테이지를 **전략 턴 vs 청소 턴** 구조로 설계해 피로도/쾌감 리듬 제공  
- 퍼즐 결과(남은 턴, 콤보)에 따라 전투 시작 버프/디버프를 부여해 **퍼즐→전투 연결**  
- 던전 층별 테마/보상을 달리해 “어디에 방치 자원을 투자할지” 선택을 전략 포인트로 설계  
- 전투는 풀오토가 아니라 궁극기 타이밍/전략 프리셋 등 **개입 포인트** 제공  
- 보스전은 패턴/카운터 중심으로 설계해 **성장+학습**을 동시에 요구

---

## game_0002 – 고양이 연금술 공방

- `game_id` : game_0002  
- `repo` : mg-game-0002  
- `title_kr` : 고양이 연금술 공방  
- `genre_tags` : ["puzzle", "craft", "idle"]  
- `portfolio_role` :  
  - 제작/합성 루프 실험 타이틀  
  - 레시피/조합 시스템 공통화 기반 마련

### core_loop

1. 재료 수집 (방치/간단 퍼즐)  
2. 레시피 실험/제작  
3. 결과물 판매/사용  
4. 공방 업그레이드 및 인테리어/고양이 배치

### fun_pillars

- 수집/조합 (Discovery)  
- 성장/효율 (Growth)  
- 표현/꾸미기 (Expression)

### design_levers

- 레시피 발견을 **힌트 기반 부분 공개 + 대성공/실패 분산**으로 설계  
- 공방 인테리어 배치에 생산/성공률 영향을 줘 “배치 퍼즐” 재미 제공  
- 고양이마다 성격/전문 분야를 부여해 어떤 작업에 누구를 배치할지 고민 포인트 생성  
- 성공 레시피는 도감에 기록되어 “원클릭 생산” 언락 → 장기적인 진척감 부여

---

## game_0003 – 픽셀 용병단 키우기

- `game_id` : game_0003  
- `repo` : mg-game-0003  
- `title_kr` : 픽셀 용병단 키우기  
- `genre_tags` : ["idle", "jrpg", "auto_battle"]  
- `portfolio_role` :  
  - 자동 전투 JRPG + 방치 구조의 장기 라이브 후보  
  - 히어로/용병 수집·강화 시스템 공통 모델 검증

### core_loop

1. 방치 전투로 자원/장비 획득  
2. 용병단(영웅, 장비, 스킬) 강화  
3. 던전/보스/탐험 진행  
4. 기지/본부 업그레이드

### fun_pillars

- 성장/수집 (Growth)  
- 빌드/시너지 (Build)  
- 파훼/도전 (Challenge)

### design_levers

- 역할(탱/딜/힐/버프) 명확화 + 세트/종족/직업 시너지로 파티 구성 자체를 퍼즐화  
- 오프라인 보상을 단순 숫자가 아니라 **전투 로그/하이라이트 리포트**로 제공  
- 웨이브 중간 목표(노데스, 시간 제한)를 넣어 관전하는 유저에게도 재미 부여  
- 레벨/장비 외에 용병단 본부/연구 트리로 **평행 성장 축** 제공

---

## game_0004 – 카페 매치 타이쿤

- `game_id` : game_0004  
- `repo` : mg-game-0004  
- `title_kr` : 카페 매치 타이쿤  
- `genre_tags` : ["match3", "management", "idle_light"]  
- `portfolio_role` :  
  - 캐주얼 유저 인입용 앵커 타이틀  
  - 퍼즐 UX/UI 공통 기준 및 스토어/마케팅 레퍼런스

### core_loop

1. 매치3 스테이지 플레이  
2. 카페 확장/인테리어/메뉴 개발  
3. 손님 응대/리뷰/평점 관리  
4. 이벤트/단골 손님과 상호작용

### fun_pillars

- 숙련/퍼즐 (Mastery)  
- 꾸미기 (Expression)  
- 스토리/캐릭터 애착 (Narrative Lite)

### design_levers

- 블록/기믹/미션을 카페 맥락(주문, 메뉴, 손님)과 강하게 연결  
- 인테리어 세트 효과 + 손님 반응 시스템으로 “테마 맞추기” 재미 제공  
- 단골 손님/요일 이벤트로 짧은 스토리와 루틴 플레이 유도  
- 카페 경영 지표(평점, 인기)에 따라 특수/하드 퍼즐이 열리는 구조 설계

---

## game_0005 – 로그라이크 퍼즐 던전

- `game_id` : game_0005  
- `repo` : mg-game-0005  
- `title_kr` : 로그라이크 퍼즐 던전  
- `genre_tags` : ["puzzle", "roguelike", "dungeon"]  
- `portfolio_role` :  
  - 깊이 있는 퍼즐+로그라이크 실험  
  - 빌드/시너지 설계와 메타 progression 실험

### core_loop

1. 턴제 퍼즐/전투 진행  
2. 층별 보상 선택(스킬, 유물, 자원)  
3. 사망 시 런 종료 → 메타 보상 획득  
4. 메타 업그레이드/해금 후 재도전

### fun_pillars

- 빌드/조합 (Build)  
- 도전/재도전 (Challenge/Retry)  
- 발견/학습 (Discovery)

### design_levers

- 각 런마다 **강한 테마 빌드**(독, 크리, 소환 등)를 형성하게 선택지 설계  
- 사망 시 “이번 런에서의 중요한 지표/실수”를 리포트해 학습 유도  
- 메타 progression(새 유물/적/스테이지)으로 반복 플레이 동기 제공  
- 퍼즐 정보 노출/패턴 설계를 통해 “후회가 남는 선택”이 생기도록 조정

---

## game_0006 – 히어로 오토 배틀 아레나

- `game_id` : game_0006  
- `repo` : mg-game-0006  
- `title_kr` : 히어로 오토 배틀 아레나  
- `genre_tags` : ["auto_battle", "pvp", "deck_building"]  
- `portfolio_role` :  
  - PVP/리그 구조 첫 실험 타이틀  
  - 덱 빌딩 + 자동 전투 메타의 기반

### core_loop

1. 영웅 수집/강화  
2. 덱 편성/전략 설정  
3. PVP 매치/리그 참여  
4. 시즌 보상 수령 및 리셋

### fun_pillars

- 경쟁/랭킹 (Competition)  
- 빌드/메타 읽기 (Meta)  
- 수집/강화 (Growth)

### design_levers

- 특정 메타가 뜨면 자연스럽게 카운터 메타가 생기도록 스킬/역할 설계  
- 코스트/속성/직업 제한으로 “그냥 좋은 거 다 넣기”를 막고 덱 구성 퍼즐화  
- 승급/강등이 모두 있는 리그 설계로 긴장 유지  
- 상위권 리플레이/덱 공유 기능으로 메타 학습 지원

---

## game_0007 – 픽셀 농장 방치RPG

- `game_id` : game_0007  
- `repo` : mg-game-0007  
- `title_kr` : 픽셀 농장 방치RPG  
- `genre_tags` : ["idle", "farm", "jrpg_light"]  
- `portfolio_role` :  
  - 힐링/루틴 플레이용 타이틀  
  - 농장+모험 루프 공통 프레임 구축

### core_loop

1. 농장 관리(수확, 씨앗 심기, 동물 돌보기)  
2. 자원 수집 후 마을/모험 준비  
3. 간단한 전투/퀘스트 진행  
4. 농장/마을 업그레이드 및 꾸미기

### fun_pillars

- 힐링/루틴 (Healing)  
- 성장/확장 (Growth)  
- 꾸미기 (Expression)

### design_levers

- 하루 5~10분이면 끝나는 “아침/저녁 루틴” 체크리스트 설계  
- 농장 자원 ↔ 모험 보상 간 상호 보강 구조로 루프 연결  
- NPC들의 시간/날씨 기반 루틴으로 생활감 제공  
- 꾸미기에 소소한 기능성 버프 부여 (단, 밸런스는 미세 조정 수준)

---

## game_0008 – 타임루프 마을 방어전

- `game_id` : game_0008  
- `repo` : mg-game-0008  
- `title_kr` : 타임루프 마을 방어전  
- `genre_tags` : ["defense", "idle", "loop"]  
- `portfolio_role` :  
  - 타임루프/회귀 메타 실험  
  - 디펜스 구조 공통 모듈 기반

### core_loop

1. 마을 방어 디펜스 세팅/플레이  
2. 웨이브 진행 및 보상 획득  
3. 패배/시간 도달 시 루프 종료 → 메타 자원 획득  
4. 루프 업그레이드 후 재도전

### fun_pillars

- 재도전/학습 (Retry)  
- 빌드/배치 (Build)  
- 점점 빨라지는 성장 (Acceleration)

### design_levers

- 루프마다 일부 구조/업그레이드가 남는 “전승 요소” 설계  
- 각 루프에 다른 목표/제약 조건(타워 제한, 속성 제한 등) 부여  
- 실패 루프에도 의미 있는 보상 제공해 계속 돌고 싶게 설계  
- 다른 유저 배치 공유/추천으로 공략 발견 재미 제공

---

## game_0009 – 영웅 수집 카드 퍼즐

- `game_id` : game_0009  
- `repo` : mg-game-0009  
- `title_kr` : 영웅 수집 카드 퍼즐  
- `genre_tags` : ["card", "puzzle", "collection"]  
- `portfolio_role` :  
  - 카드 덱 + 퍼즐 연계 구조 실험  
  - 카드/시너지 공통 모듈 기반

### core_loop

1. 퍼즐 전투(블록 매칭)  
2. 카드/영웅 스킬 발동  
3. 카드 수집/강화  
4. 덱 구성/시너지 최적화

### fun_pillars

- 빌드/덱 (Build)  
- 수집 (Collection)  
- 퍼즐 판단 (Mastery)

### design_levers

- 블록 색/패턴과 카드 스킬 발동을 강하게 연결  
- 덱 속성/색 분포가 플레이 스타일(안정/폭발)에 영향을 주도록 설계  
- 카드 도감에서 추천 시너지/빌드 예시 제공  
- 퍼즐 성적(콤보/남은 턴)이 카드 보상/재료에 영향을 주게 설계

---

## game_0010 – 던전 샵 시뮬레이터

- `game_id` : game_0010  
- `repo` : mg-game-0010  
- `title_kr` : 던전 샵 시뮬레이터  
- `genre_tags` : ["management", "idle", "economy"]  
- `portfolio_role` :  
  - 경제/상점 시뮬 구조 실험  
  - 수요/공급/이벤트 경제 루프 공통 프레임 구축

### core_loop

1. 아이템 제작/매입  
2. 가격/배치 설정  
3. 손님 방문/구매/반응 확인  
4. 수익으로 상점/던전 지원 확대

### fun_pillars

- 경제 퍼즐 (Economy)  
- 성장/확장 (Growth)  
- 시장 읽기 (Prediction)

### design_levers

- 아이템별 수요/공급 곡선을 시각화하여 경제 퍼즐화  
- 손님 세그먼트(초보/고급/모험가 등)마다 선호/민감도 차별  
- 상점 판매 품목이 던전 원정대 성과에 영향을 미치게 설계  
- 이벤트/시즌별로 특정 아이템 수요 폭등 구조로 타이밍 게임 제공

---

## game_0011 – 요정 숲 힐링 아이들

- `game_id` : game_0011  
- `repo` : mg-game-0011  
- `title_kr` : 요정 숲 힐링 아이들  
- `genre_tags` : ["idle", "decoration", "social_light"]  
- `portfolio_role` :  
  - 힐링/꾸미기 특화 타이틀  
  - 라이트 소셜/꾸미기 공통 모듈 실험

### core_loop

1. 숲/요정과 상호작용  
2. 자원 수집 및 꾸미기/시설 배치  
3. 요정/환경 성장/해금  
4. 친구 숲 방문/가벼운 소셜 상호작용

### fun_pillars

- 힐링 (Healing)  
- 꾸미기 (Expression)  
- 소소한 상호작용 (Social Lite)

### design_levers

- 모든 상호작용에 부드러운 애니메이션/사운드 피드백 제공  
- 시간/날씨/이벤트에 따라 요정/환경 루틴 변화  
- 테마별 꾸미기 세트 미션으로 목표 있는 꾸미기 제공  
- 방문/좋아요 기반 가벼운 소셜 보상 구조 설계

---

## game_0012 – 연말 레이드 이벤트RPG

- `game_id` : game_0012  
- `repo` : mg-game-0012  
- `title_kr` : 연말 레이드 이벤트RPG  
- `genre_tags` : ["raid", "jrpg", "event"]  
- `portfolio_role` :  
  - 이벤트/레이드 중심 라이브 운영 실험 타이틀  
  - 크로스게임/시즌 이벤트 프레임 출발점

### core_loop

1. 레이드 준비(파티/장비 세팅)  
2. 레이드 전투 참여  
3. 개인/전체 보상 획득  
4. 이벤트 스토리/콘텐츠 소비

### fun_pillars

- 협동/공동 목표 (Co-op)  
- 단기 몰입 (Short-term Engagement)  
- 한정 보상 (Limited Reward)

### design_levers

- 역할 분담(탱/딜/힐/서포트)을 전제로 레이드 패턴 설계  
- 개인 기여도 보상 + 서버/길드 누적 진행도 보상을 동시에 제공  
- 연말/테마 연출과 스토리 컷신으로 단기 몰입 유도  
- 다른 게임 캐릭터/요소 카메오 등장으로 크로스 프로모션 허브 역할 설계

---

# Year 2 – 0013 ~ 0024

(Year 2 게임들도 동일한 형식으로 정의 – 라인업/전략 문서 기준으로 설계)

## game_0013 – 아레나 레전드: 용병 리그

- `game_id` : game_0013  
- `repo` : mg-game-0013  
- `title_kr` : 아레나 레전드: 용병 리그  
- `genre_tags` : ["auto_battle", "league", "pvp"]  
- `portfolio_role` :  
  - PVP/리그 시스템 심화  
  - `game_0006`에서 발전한 경쟁 타이틀

### fun_pillars

- 경쟁/리그 (Competition)  
- 시즌/메타 변화 (Seasonal Meta)  
- 빌드/카운터 (Counterplay)

### design_levers

- 리그/디비전 구조, 승강 시스템 명확화  
- 시즌마다 “핵심 테마 버프/너프” 부여로 메타 변화 유도  
- 카운터 빌드가 뚜렷이 존재하도록 스킬/역할 설계  
- 시즌 종료 보상 + 누적 명예 시스템 분리

---

## game_0014 – 마녀의 연구소: 실험형 퍼즐

- `game_id` : game_0014  
- `repo` : mg-game-0014  
- `title_kr` : 마녀의 연구소: 실험형 퍼즐  
- `genre_tags` : ["puzzle", "roguelike", "build"]  
- `portfolio_role` :  
  - 스킬 빌드/시너지 특화 퍼즐 로그라이크  
  - `game_0005` 확장 버전

### fun_pillars

- 빌드/시너지 (Build)  
- 실험/발견 (Experiment)  
- 재도전 (Retry)

### design_levers

- 매 판마다 스킬 조합이 완전히 달라지는 랜덤/드래프트 시스템 설계  
- 시너지 세트를 명시적으로 표현(아이콘/태그)해 전략 선택 지원  
- “실험 노트” 시스템으로 플레이어가 발견한 조합을 기록/추천

---

## game_0015 – 왕국 재건 프로젝트

- `game_id` : game_0015  
- `repo` : mg-game-0015  
- `title_kr` : 왕국 재건 프로젝트  
- `genre_tags` : ["idle", "city_build", "story"]  
- `portfolio_role` :  
  - 장기 성장형 방치 타이틀 후보  
  - 스토리/건설/이벤트 LiveOps 실험

### fun_pillars

- 장기 성장 (Long-term Growth)  
- 도시/왕국 꾸미기 (Expression)  
- 서사 (Narrative)

### design_levers

- 폐허→번영으로 이어지는 `city_stage` 단계를 명확히 구분  
- 건물/시설 업그레이드에 스토리/이벤트 연동  
- 선택형 스토리/정책으로 왕국 방향성 결정(성장 vs 복지 등)

---

## game_0016 – 덱빌딩 히어로즈

- `game_id` : game_0016  
- `repo` : mg-game-0016  
- `title_kr` : 덱빌딩 히어로즈  
- `genre_tags` : ["deck_building", "auto_battle", "jrpg"]  
- `portfolio_role` :  
  - 카드/덱 빌딩 공통 모듈 본격 실험  
  - 전투/카드 시스템의 브릿지

### fun_pillars

- 덱 구축 (Deckbuilding)  
- 시너지/콤보 (Synergy)  
- 전략 vs 운 (Strategy vs RNG)

### design_levers

- 전투를 카드 우선 관점으로 재설계 (카드가 곧 스킬/패턴)  
- 카드 풀/레어도/조합 제한으로 빌드 다양성 확보  
- “시드 덱” 예시 제공으로 진입 장벽 완화

---

## game_0017 – 던전 크래프트 타이쿤

- `game_id` : game_0017  
- `repo` : mg-game-0017  
- `title_kr` : 던전 크래프트 타이쿤  
- `genre_tags` : ["craft", "idle", "management"]  
- `portfolio_role` :  
  - 제작/레시피/경제 심화  
  - `game_0010`의 제작 측면 확장

### fun_pillars

- 제작/설계 (Crafting)  
- 경제/수요 대응 (Economy)  
- 파생 시스템 (Meta Systems)

### design_levers

- 레시피 계보(기본→고급→전설) 구조 설계  
- 재료/레시피 희귀도와 시장 가격의 상관 관계 표현  
- 다른 게임/던전과의 아이템 연계 가능성 열어두기

---

## game_0018 – 카툰 레이싱 RPG

- `game_id` : game_0018  
- `repo` : mg-game-0018  
- `title_kr` : 카툰 레이싱 RPG  
- `genre_tags` : ["racing", "rpg", "card"]  
- `portfolio_role` :  
  - 비전통 장르 믹스 실험 (레이싱+RPG)  
  - 새로운 유저층 탐색

### fun_pillars

- 스피드/피드백 (Speed)  
- 튜닝/커스터마이징 (Customization)  
- 카드/능력 관리 (Build)

### design_levers

- 짧은 레이스(1~2분) 중심으로 루프 설계  
- 튜닝/능력 카드가 레이스 스타일에 큰 영향(드리프트, 부스터 등)  
- 레이스 리플레이/고스트 경쟁 제공

---

## game_0019 – 길드 오브 방랑자들

- `game_id` : game_0019  
- `repo` : mg-game-0019  
- `title_kr` : 길드 오브 방랑자들  
- `genre_tags` : ["idle", "guild", "coop"]  
- `portfolio_role` :  
  - 길드/협동 시스템 본격 실험  
  - 방치RPG의 소셜 확장

### fun_pillars

- 협동 (Co-op)  
- 길드 성장 (Guild Growth)  
- 공동 목표 (Shared Goal)

### design_levers

- 길드 퀘스트/보스/상점 구조 설계  
- 개인 기여도와 길드 전체 보상을 균형있게 분배  
- 길드간 경쟁/협력 이벤트 설계

---

## game_0020 – 타임 슬립 탐험대

- `game_id` : game_0020  
- `repo` : mg-game-0020  
- `title_kr` : 타임 슬립 탐험대  
- `genre_tags` : ["exploration", "loop", "meta_growth"]  
- `portfolio_role` :  
  - 타임루프/회귀 메커닉 심화  
  - `game_0008` 확장판

### fun_pillars

- 탐험/발견 (Exploration)  
- 루프 성장 (Loop Growth)  
- 빌드/경로 선택 (Route Choice)

### design_levers

- 탐험 루트/시간 제한/루프 보상 구조 명확화  
- 회귀 후 시작 지점/능력/정보 차별화  
- “타임라인” UI로 플레이어의 선택/역사를 시각화

---

## game_0021 – 오염도 제로: 클리너 브리게이드

- `game_id` : game_0021  
- `repo` : mg-game-0021  
- `title_kr` : 오염도 제로: 클리너 브리게이드  
- `genre_tags` : ["puzzle", "defense", "theme_green"]  
- `portfolio_role` :  
  - 친환경/브랜딩 친화 타이틀  
  - 브랜드/ESG 콜라보 가능성 실험

### fun_pillars

- 퍼즐/디펜스 (Puzzle/Defense)  
- 테마 몰입 (Theme)  
- 미션/목표 (Mission)

### design_levers

- 오염도/청소율을 직관적으로 보여주는 시각 효과  
- 친환경 메시지를 재미로 소화할 수 있는 퀘스트/이벤트 설계  
- 브랜드/기관 콜라보를 끼워 넣을 수 있는 미션 구조 확보

---

## game_0022 – 몬스터 레벨업 파티

- `game_id` : game_0022  
- `repo` : mg-game-0022  
- `title_kr` : 몬스터 레벨업 파티  
- `genre_tags` : ["minigame", "idle", "party"]  
- `portfolio_role` :  
  - 미니게임 기반 라이브 운영/이벤트 실험  
  - 크로스 게임 이벤트 전용 미니게임 플랫폼

### fun_pillars

- 짧은 세션 (Short Sessions)  
- 다양한 규칙 (Variety)  
- 성장/파티 (Growth/Party)

### design_levers

- 30초~1분 내 끝나는 미니게임 여러 개 디자인  
- 미니게임마다 스코어/보상/성장 연결  
- 다른 게임 이벤트에 재사용 가능한 미니게임 포맷 정의

---

## game_0023 – 콜로니 프론티어

- `game_id` : game_0023  
- `repo` : mg-game-0023  
- `title_kr` : 콜로니 프론티어  
- `genre_tags` : ["colony_sim", "idle", "survival"]  
- `portfolio_role` :  
  - 하드코어/코어 게이머 타깃  
  - 깊이 있는 시스템 실험

### fun_pillars

- 생존/위기 관리 (Survival)  
- 기지/식민지 설계 (Base Design)  
- 장기 목표 (Long-term Goals)

### design_levers

- 자원/위험/환경 요소 간 상호작용 세밀하게 설계  
- 이벤트/위기(폭풍, 괴생물체, 반란 등)로 긴장 유지  
- 난이도 프리셋/커스텀 룰로 고인물 유저 대응

---

## game_0024 – 레전드 페스티벌: 크로스이벤트

- `game_id` : game_0024  
- `repo` : mg-game-0024  
- `title_kr` : 레전드 페스티벌: 크로스이벤트  
- `genre_tags` : ["event_hub", "raid", "crossover"]  
- `portfolio_role` :  
  - 크로스 게임 메타/이벤트 허브  
  - Year1~2 게임 자산 재활용과 프로모션 중심

### fun_pillars

- 이벤트/축제 (Festival)  
- 크로스오버 (Crossover)  
- 단기 집중 (Burst Engagement)

### design_levers

- 여러 게임의 캐릭터/보스/규칙이 한 곳에 모이는 이벤트 구조 설계  
- 기간 한정 레이드/미션/미니게임 묶음 제공  
- 다른 게임 플레이/성과에 따라 이 페스티벌에서 버프/보상 제공

---

이 문서는 24개 게임 전체의 개요/역할/재미 설계 방향을 한 번에 볼 수 있는 참조용이며,  
게임별 상세 GDD(`gdd_game_00XX.json`) 작성 시 `fun_pillars`, `design_levers` 필드를 그대로 가져다 쓸 수 있다.
