# Epic 2: 공통 프레임워크 - User Stories 개요

## Story 목록

### Story 1: 시나리오 #6 PMI 분석 구현 (복붙 방식) ✅
**Points**: 3 | **Priority**: Must
- [상세 문서](./story-01-pmi-scenario.md)
- 시나리오 #1 코드 복사 → PMI 로직으로 수정
- 빠른 검증, 하드코딩 허용

### Story 2: 공통 패턴 식별 및 베이스 클래스 설계 ✅
**Points**: 5 | **Priority**: Must
- [상세 문서](./story-02-base-classes.md)
- 두 시나리오 코드 비교 분석
- ScenarioBase, CollectorBase 등 추상 클래스 설계
- 템플릿 메서드 패턴 구현

### Story 3: 시나리오 #1, #6을 프레임워크로 리팩토링
**Points**: 3 | **Priority**: Must

**User Story**:
As a 개발자, I want 기존 시나리오들을 베이스 클래스 기반으로 리팩토링, So that 프레임워크 실효성을 검증하고 향후 시나리오 추가 시 참고할 수 있다

**Acceptance Criteria**:
- [ ] SemiconductorWeeklyScenario가 ScenarioBase 상속
- [ ] PMIAnalysisScenario가 ScenarioBase 상속
- [ ] 리팩토링 전후 리포트 동일성 검증
- [ ] 코드 중복률 < 30%

**Tasks**:
1. SemiconductorWeeklyScenario 리팩토링
   - ScenarioBase 상속
   - `setup_collectors()`, `setup_analyzer()`, `setup_reporter()` 구현
   - 기존 로직을 메서드로 분리
2. PMIAnalysisScenario 리팩토링 (동일)
3. Before/After 리포트 diff 비교
4. 단위 테스트 업데이트
5. 통합 테스트 실행

**Notes**:
- 기능 변경 없음, 구조만 변경
- 테스트로 회귀 방지 필수
- 리팩토링 중 개선 아이디어 기록 (향후 Epic에서)

---

### Story 4: 새 시나리오 추가 가이드 문서화
**Points**: 2 | **Priority**: Should

**User Story**:
As a 개발자, I want 새 시나리오 추가 가이드 문서, So that 향후 시나리오를 30분 내에 추가할 수 있다

**Acceptance Criteria**:
- [ ] `docs/guides/add-new-scenario.md` 문서 작성
- [ ] 단계별 가이드 (1. 데이터 수집 정의 → 2. 프롬프트 작성 → 3. 시나리오 클래스 구현)
- [ ] 템플릿 코드 제공
- [ ] 실제로 가상 시나리오를 30분 내 추가 가능한지 테스트

**Tasks**:
1. 가이드 문서 작성
   - 프레임워크 개념 설명
   - 시나리오 추가 단계
   - 체크리스트
   - 예시 코드 (주석 포함)
2. 템플릿 코드 작성 (`templates/scenario_template.py`)
3. 가이드에 따라 더미 시나리오 추가 (예: 달러 인덱스 분석)
4. 소요 시간 기록 → 가이드 개선

**Notes**:
- 이 문서가 Epic 5 (시나리오 2-7 추가) 속도를 좌우
- 가능한 한 상세하게, 코드 복붙으로 80% 완성되게

---

## Epic 2 완료 기준

- [ ] 모든 Story (1-4) 완료
- [ ] 시나리오 #1, #6 모두 프레임워크 기반 동작
- [ ] 코드 재사용률 70% 이상
- [ ] 새 시나리오 추가 가이드 검증 완료
- [ ] Epic 회고 작성 (배운 점, 개선점)

---

**다음 단계**: Epic 3 (자동화) 또는 시나리오 1-2개 더 추가 (Epic 2 검증)
