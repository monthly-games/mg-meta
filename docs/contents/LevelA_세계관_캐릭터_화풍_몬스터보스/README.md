# Monthly Games — Level A 라인업 세계관·캐릭터·화풍·몬스터/보스 구성 패키지 v1.0
- 작성일: 2025-12-12
- 포함: 12개 타이틀 각각의 **world/style/characters/enemies/bosses**(YAML) + 공통 **style tokens/prompt kit/spine 규칙** + JSON Schema

## 폴더 구조
- world/meta_universe.yaml : 공통 우주관(샤드라인)
- art/style_tokens.yaml : 공통 화풍 토큰
- art/comfyui_prompt_kit.yaml : 생성 프롬프트 키트(안전/일관성)
- animation/spine2d_style.yaml : Spine2D 모션 언어
- configs/lineup_levelA_12months.yaml : 12개월 라인업 메타
- schemas/*.schema.json : world/character 검증 스키마
- games/<game_id>/
  - world.yaml : 세계관/조직/사건/톤
  - style.yaml : 팔레트/렌더/UI/프롬프트 오버라이드
  - characters.yaml : 코어 12명(역할/속성/무기/관계 트리거/코스튬 테마)
  - enemies.yaml : 일반 8 + 엘리트 4
  - bosses.yaml : 중간보스 1 + 최종보스 1

## 사용 가이드(추천 워크플로우)
1) configs/lineup_levelA_12months.yaml에서 이번 달 game_id 선택
2) games/<game_id>/style.yaml의 palette/토큰을 ComfyUI Prompt Kit에 주입
3) characters.yaml을 기준으로 캐릭터 아트/코스튬/대사 생성(관계 트리거 활용)
4) enemies.yaml의 visual_tags를 이용해 몬스터 스프라이트/보스 키비주얼 생성
5) bosses.yaml의 signature_mechanics로 보스 패턴 설계(전투 변주 1개와 결합)
