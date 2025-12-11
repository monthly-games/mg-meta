# 개발 프로세스

## 브랜치 전략

### 브랜치 구조
- `main` - 프로덕션 배포 브랜치
- `develop` - 개발 통합 브랜치
- `feature/*` - 기능 개발 브랜치
- `hotfix/*` - 긴급 수정 브랜치
- `release/*` - 릴리즈 준비 브랜치

### 브랜치 네이밍
```
feature/GAME-0001-123-tutorial-stage
hotfix/GAME-0001-urgent-crash-fix
release/v1.0.0
```

## 커밋 컨벤션

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type
- `feat`: 새로운 기능
- `fix`: 버그 수정
- `docs`: 문서 변경
- `style`: 코드 포맷팅
- `refactor`: 리팩토링
- `test`: 테스트 추가
- `chore`: 빌드/설정 변경

## PR 프로세스

1. Feature 브랜치 생성
2. 개발 및 테스트
3. PR 생성 (템플릿 사용)
4. 코드 리뷰
5. CI 통과 확인
6. Merge to develop
