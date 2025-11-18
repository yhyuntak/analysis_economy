# User Story 2: 공통 패턴 식별 및 베이스 클래스 설계

## User Story

**As a** 개발자
**I want** 시나리오 #1과 #6의 공통 패턴을 추출하여 베이스 클래스로 설계
**So that** 향후 시나리오 추가 시 개발 시간을 50% 단축할 수 있다

---

## Story Details

**Story ID**: EPIC-002-S2
**Epic**: Epic 2 - 공통 프레임워크 및 PMI 시나리오 추가
**Story Points**: 5
**Priority**: Must Have
**Assignee**: Developer (본인)
**Status**: 대기 중

---

## Acceptance Criteria

### AC1: 공통 패턴 문서화
- [ ] 시나리오 #1, #6 코드 비교 분석 완료
- [ ] 공통 로직 목록 작성 (데이터 수집 → 분석 → 리포트)
- [ ] 시나리오별 차이점 목록 작성
- [ ] `docs/architecture/framework-design.md` 문서 작성

### AC2: 베이스 클래스 설계
- [ ] `ScenarioBase` 추상 클래스 설계
- [ ] `DataCollectorBase` 추상 클래스 설계
- [ ] `AnalyzerBase` 추상 클래스 설계
- [ ] `ReporterBase` 추상 클래스 설계
- [ ] 클래스 다이어그램 작성

### AC3: ScenarioBase 구현
- [ ] `src/framework/base_scenario.py` 파일 생성
- [ ] 템플릿 메서드 패턴 구현 (`run()` 메서드)
- [ ] 추상 메서드 정의 (`setup_collectors()`, `setup_analyzer()`, `setup_reporter()`)
- [ ] 공통 로직 구현 (`_collect_data()`, `_analyze_data()`, `_generate_report()`)

### AC4: 베이스 클래스 테스트
- [ ] 간단한 테스트 시나리오 구현 (베이스 클래스 상속)
- [ ] 템플릿 메서드 패턴 동작 검증
- [ ] 추상 메서드 미구현 시 에러 발생 확인

---

## Tasks

### Task 1: 코드 비교 분석
- [ ] 시나리오 #1과 #6 코드를 나란히 놓고 비교
- [ ] 공통 부분 표시
  - 데이터 수집 → 분석 → 리포트 흐름
  - LLM API 호출 로직
  - 리포트 파일 저장 로직
  - 에러 핸들링
- [ ] 차이점 표시
  - 수집 데이터 종류 (주가 vs PMI)
  - LLM 프롬프트 내용
  - 리포트 섹션 구성

### Task 2: 프레임워크 설계 문서 작성
- [ ] `docs/architecture/framework-design.md` 생성
- [ ] 설계 원칙 작성
  - 템플릿 메서드 패턴 사용
  - SOLID 원칙 준수
  - 최소한의 추상화 (YAGNI)
- [ ] 클래스 구조 다이어그램
- [ ] 시나리오 추가 가이드 (예시 코드 포함)

### Task 3: ScenarioBase 구현
- [ ] `src/framework/` 디렉토리 생성
- [ ] `base_scenario.py` 구현
  ```python
  from abc import ABC, abstractmethod
  from typing import Dict, Any

  class ScenarioBase(ABC):
      def __init__(self, config: Dict[str, Any]):
          self.config = config
          self.collectors = []
          self.analyzer = None
          self.reporter = None

      @abstractmethod
      def setup_collectors(self):
          """데이터 수집기 초기화 (하위 클래스에서 구현)"""
          pass

      @abstractmethod
      def setup_analyzer(self):
          """분석기 초기화 (하위 클래스에서 구현)"""
          pass

      @abstractmethod
      def setup_reporter(self):
          """리포터 초기화 (하위 클래스에서 구현)"""
          pass

      def run(self) -> str:
          """
          시나리오 실행 (템플릿 메서드)
          """
          self.setup_collectors()
          self.setup_analyzer()
          self.setup_reporter()

          raw_data = self._collect_data()
          insights = self._analyze_data(raw_data)
          report_path = self._generate_report(raw_data, insights)

          return report_path

      def _collect_data(self) -> Dict[str, Any]:
          """공통 데이터 수집 로직"""
          data = {}
          for collector in self.collectors:
              data.update(collector.collect())
          return data

      def _analyze_data(self, data: Dict[str, Any]) -> str:
          """공통 분석 로직"""
          return self.analyzer.analyze(data)

      def _generate_report(self, data: Dict[str, Any], insights: str) -> str:
          """공통 리포트 생성 로직"""
          return self.reporter.generate(data, insights)
  ```

### Task 4: DataCollectorBase 구현
- [ ] `base_collector.py` 구현
  ```python
  from abc import ABC, abstractmethod

  class DataCollectorBase(ABC):
      @abstractmethod
      def collect(self) -> dict:
          """데이터 수집 (하위 클래스에서 구현)"""
          pass
  ```

### Task 5: AnalyzerBase 구현
- [ ] `base_analyzer.py` 구현 (LLMAnalyzer는 이미 구현되어 있으므로 인터페이스만)

### Task 6: ReporterBase 구현
- [ ] `base_reporter.py` 구현 (MarkdownReporter 인터페이스화)

### Task 7: 테스트 시나리오 구현
- [ ] `tests/test_framework.py` 생성
- [ ] 간단한 더미 시나리오 구현
  ```python
  class DummyScenario(ScenarioBase):
      def setup_collectors(self):
          self.collectors = [DummyCollector()]

      def setup_analyzer(self):
          self.analyzer = DummyAnalyzer()

      def setup_reporter(self):
          self.reporter = DummyReporter()
  ```
- [ ] `run()` 메서드 실행 테스트
- [ ] 각 단계 호출 순서 검증

---

## Technical Notes

### 템플릿 메서드 패턴

```
ScenarioBase.run()  [템플릿 메서드]
  ├── setup_collectors()     [추상 메서드 - 하위 구현]
  ├── setup_analyzer()       [추상 메서드 - 하위 구현]
  ├── setup_reporter()       [추상 메서드 - 하위 구현]
  ├── _collect_data()        [공통 로직]
  ├── _analyze_data()        [공통 로직]
  └── _generate_report()     [공통 로직]
```

### 설계 원칙

1. **단일 책임**: 각 베이스 클래스는 하나의 역할만
2. **개방-폐쇄**: 확장에는 열려있고, 수정에는 닫혀있게
3. **최소 추상화**: 두 시나리오에서 실제로 공통인 것만 추출

### 주의사항

- **과도한 추상화 방지**: 3개 시나리오 보기 전까진 완벽한 설계 불가
- **실용주의**: 완벽한 OOP보다 실용적인 코드
- **리팩토링 여지**: Epic 5에서 시나리오 5개 추가 후 재설계 가능

---

## Definition of Done

- [ ] 모든 Acceptance Criteria 충족
- [ ] 모든 Tasks 완료
- [ ] 베이스 클래스 구현 완료
- [ ] 테스트 시나리오로 동작 검증
- [ ] 설계 문서 작성 완료
- [ ] 코드 리뷰 (스스로 검토)

---

## Dependencies

**선행 작업**:
- Story 1 완료 (시나리오 #6 구현으로 비교 대상 확보)

**후속 작업**:
- Story 3: 시나리오 #1, #6을 프레임워크로 리팩토링

---

## Risks

| 리스크 | 완화 전략 |
|--------|-----------|
| 과도한 추상화 | 두 시나리오만 보고 설계, 단순하게 시작 |
| 설계 변경 필요성 | Epic 5에서 재설계 예상, 유연하게 접근 |
| 시간 소요 | 완벽한 설계보다 동작하는 설계 우선 |

---

## Estimated Time

**Story Points**: 5

**예상 시간**: 1-2일
- Task 1-2 (분석 + 문서): 3-4시간
- Task 3-6 (베이스 클래스 구현): 4-6시간
- Task 7 (테스트): 1-2시간

---

## Notes

- 이 Story는 Epic 2의 핵심
- 설계가 향후 5개 시나리오 추가 속도를 좌우
- 하지만 완벽 추구하지 말고, 두 시나리오만 보고 판단
- Story 3에서 실제로 리팩토링하며 설계 검증

---

**Created**: 2025-11-18
**Last Updated**: 2025-11-18
**Status**: Ready for Development
