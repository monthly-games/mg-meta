# AI 개발 방법론

## 원칙

1. **구조화된 컨텍스트 전달**
   - 이슈/PR에 `spec_json` 형식으로 명세 포함
   - 관련 파일 경로 명시
   - 제약사항 명시

2. **일관된 템플릿 사용**
   - GDD, 이슈, PR 템플릿 준수
   - JSON 형식으로 구조화된 정보 전달

3. **점진적 구현**
   - 작은 단위로 나눠서 구현
   - 각 단계별 검증

## AI 에이전트 활용

### 설계 단계
- GDD 검토 및 피드백
- 아키텍처 제안
- 기술 스택 검토

### 구현 단계
- 코드 생성
- 테스트 작성
- 문서화

### 리뷰 단계
- 코드 리뷰
- 보안 검토
- 성능 분석

## spec_json 형식

```json
{
  "task_id": "GAME-0001-123",
  "game_id": "game_0001",
  "area": "game",
  "type": "feature",
  "spec_json": {
    "goal": "목표 설명",
    "acceptance_criteria": ["AC1", "AC2"],
    "target_files": ["path/to/files/**"],
    "non_goals": ["제외 사항"],
    "related_docs": ["관련 문서"],
    "constraints": ["제약 사항"]
  }
}
```
