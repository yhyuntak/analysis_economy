# Epic 2: 공통 프레임워크 및 PMI 시나리오 추가

## Epic 개요

**ID**: EPIC-002
**제목**: 공통 프레임워크 구축 및 시나리오 #6 PMI 분석 추가
**Priority**: Must Have
**Story Points**: 13
**Target Release**: Release 1 (Month 1-3)
**Status**: 대기 중
**의존성**: Epic 1 완료 필요

---

## PM 관점

### 비즈니스 목표

1. **개발 효율성 극대화**: 신규 시나리오 추가 시간 3일 → 1일로 단축
2. **확장성 검증**: PMI 시나리오 추가를 통해 프레임워크 실효성 입증
3. **산업재 섹터 커버리지**: 제조업 PMI 분석으로 경기 선행 지표 확보

### 사용자 가치

**Target User**: 경기 사이클 투자자

**Pain Point**:
- 새로운 시나리오 추가 시 중복 코드 작성으로 시간 낭비
- 시나리오 간 일관성 부족으로 혼란
- PMI 같은 중요 지표 놓침

**Provided Value**:
- 일관된 리포트 구조로 학습 곡선 제로
- PMI 발표 시점에 산업재/소재 섹터 영향 즉시 파악
- 향후 5개 시나리오 빠르게 추가 가능

### Success Metrics

- [ ] **개발 속도**: 시나리오 #6 구현 시간 < 1일 (프레임워크 사용)
- [ ] **코드 재사용률**: 70% 이상
- [ ] **일관성**: 두 시나리오 리포트 구조 동일
- [ ] **확장성 검증**: 시나리오 #2 추가 가이드 문서 작성 완료

---

## Designer 관점

### UX 목표

1. **일관된 경험**: 모든 시나리오에서 동일한 리포트 구조
2. **메타데이터 표준화**: 생성일, 시나리오 타입, 데이터 소스 명시
3. **예측 가능성**: 사용자가 어떤 시나리오든 즉시 이해 가능

### 리포트 표준 구조

```markdown
# [시나리오 제목]

**생성 일시**: YYYY-MM-DD HH:MM
**분석 기간**: YYYY-MM-DD ~ YYYY-MM-DD
**데이터 소스**: [소스 나열]
**시나리오 타입**: [weekly/event/monthly]

---

## 요약

[3줄 요약]

---

## 주요 지표

[표 형식 데이터]

---

## 분석

[LLM 생성 인사이트]

---

## 투자 시사점

[투자 관점 해석]

---

## 참고사항

[면책 조항]
```

### CLI 일관성

```bash
# 모든 시나리오 동일한 인터페이스
python run_scenario.py --scenario semiconductor-weekly
python run_scenario.py --scenario pmi-analysis

# 공통 옵션
--date YYYY-MM-DD       # 분석 기준일 (기본값: 오늘)
--output-dir DIR        # 출력 경로 (기본값: reports/)
--verbose               # 상세 로그 출력
```

---

## Developer 관점

### 기술 목표

1. **패턴 추출**: 시나리오 1에서 공통 로직 식별
2. **베이스 클래스 설계**: 재사용 가능한 추상 클래스 구현
3. **프레임워크 검증**: 시나리오 2로 설계 유효성 입증
4. **리팩토링**: 시나리오 1을 프레임워크로 마이그레이션

### Architecture Design

```
src/
├── framework/
│   ├── base_scenario.py          # 시나리오 베이스 클래스
│   ├── base_collector.py         # 데이터 수집기 베이스
│   ├── base_analyzer.py          # 분석기 베이스
│   └── base_reporter.py          # 리포터 베이스
├── scenarios/
│   ├── semiconductor_weekly.py   # ScenarioBase 상속
│   └── pmi_analysis.py           # ScenarioBase 상속 (NEW)
├── collectors/
│   ├── stock_collector.py        # 재사용
│   ├── news_collector.py         # 재사용
│   └── economic_data_collector.py # PMI용 (NEW)
├── analyzers/
│   └── llm_analyzer.py            # 재사용
└── reporters/
    └── markdown_reporter.py       # 재사용
```

### 베이스 클래스 인터페이스

```python
# framework/base_scenario.py
from abc import ABC, abstractmethod
from typing import Dict, Any

class ScenarioBase(ABC):
    """
    모든 분석 시나리오의 베이스 클래스
    """

    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.collectors = []
        self.analyzer = None
        self.reporter = None

    @abstractmethod
    def setup_collectors(self):
        """데이터 수집기 초기화"""
        pass

    @abstractmethod
    def setup_analyzer(self):
        """분석기 초기화"""
        pass

    @abstractmethod
    def setup_reporter(self):
        """리포터 초기화"""
        pass

    def run(self) -> str:
        """
        시나리오 실행 (템플릿 메서드)
        """
        self.setup_collectors()
        self.setup_analyzer()
        self.setup_reporter()

        # 1. 데이터 수집
        raw_data = self._collect_data()

        # 2. 분석
        insights = self._analyze_data(raw_data)

        # 3. 리포트 생성
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

### PMI 시나리오 구현 예시

```python
# scenarios/pmi_analysis.py
from framework.base_scenario import ScenarioBase
from collectors.economic_data_collector import EconomicDataCollector
from collectors.stock_collector import StockCollector
from analyzers.llm_analyzer import LLMAnalyzer
from reporters.markdown_reporter import MarkdownReporter

class PMIAnalysisScenario(ScenarioBase):
    """
    제조업 PMI 발표 시 산업재/소재 섹터 영향 분석
    """

    def setup_collectors(self):
        # PMI 데이터
        self.collectors.append(
            EconomicDataCollector(indicators=['PMI_US', 'PMI_CHINA'])
        )

        # 산업재/소재 ETF
        self.collectors.append(
            StockCollector(symbols=['XLI', 'XLB', 'IYJ'])
        )

    def setup_analyzer(self):
        prompt_template = """
        다음 PMI 데이터와 산업재/소재 섹터 주가를 분석하시오:

        {data}

        분석 요청:
        1. PMI 변화의 의미 (확장/위축)
        2. 섹터 주가 반응 해석
        3. 향후 투자 시사점
        """

        self.analyzer = LLMAnalyzer(prompt_template=prompt_template)

    def setup_reporter(self):
        self.reporter = MarkdownReporter(
            template_name='pmi_analysis',
            output_dir='reports/'
        )
```

### Implementation Complexity

**총 Story Points**: 13

- **Story 1**: PMI 시나리오 구현 (복붙, 3pt) - 낮은 난이도
- **Story 2**: 공통 패턴 식별 및 베이스 클래스 설계 (5pt) - 중간 난이도
- **Story 3**: 리팩토링 (3pt) - 낮은 난이도
- **Story 4**: 가이드 문서화 (2pt) - 낮은 난이도

### 주요 기술 과제

1. **추상화 수준 결정**: 너무 추상적 vs 너무 구체적 균형
   - **해결**: 두 시나리오만 보고 판단, 3번째에서 재조정

2. **하위 호환성**: 시나리오 1 리팩토링 시 기능 유지
   - **해결**: 통합 테스트, Before/After 리포트 비교

3. **PMI 데이터 소스**: 무료 API 찾기
   - **해결**: Trading Economics API (무료 티어), FRED API

---

## User Stories

이 Epic은 4개의 User Story로 구성됩니다:

1. **[Story 1] 시나리오 #6 PMI 분석 구현** (3pt)
   - 시나리오 1 코드 복사 후 PMI 로직으로 수정
   - 빠른 검증용 하드코딩 허용

2. **[Story 2] 공통 패턴 식별 및 베이스 클래스 설계** (5pt)
   - 두 시나리오 코드 비교 분석
   - ScenarioBase, DataCollectorBase 등 설계

3. **[Story 3] 시나리오 #1과 #6을 프레임워크로 리팩토링** (3pt)
   - 베이스 클래스 상속 구조로 변경
   - 기능 동일성 검증

4. **[Story 4] 새 시나리오 추가 가이드 문서화** (2pt)
   - 개발자 가이드 작성
   - 템플릿 코드 제공

---

## Acceptance Criteria

Epic 전체가 완료되었다고 판단하는 기준:

- [ ] 시나리오 #1, #6 모두 프레임워크 기반으로 동작
- [ ] 두 시나리오 실행 시 리포트 구조 동일
- [ ] 시나리오 #6 구현 시간 < 1일 (개발자 주관 평가)
- [ ] 코드 중복률 < 30% (공통 코드 70% 이상)
- [ ] 새 시나리오 추가 가이드 문서 완성
- [ ] 가상의 시나리오 #2를 가이드 따라 30분 내 추가 가능 (프로토타입 수준)

---

## Risks & Mitigation

| 리스크 | 영향 | 확률 | 완화 전략 |
|--------|------|------|-----------|
| 과도한 추상화 | 중간 | 높음 | 두 사례만 보고 설계, YAGNI 원칙 |
| 리팩토링 중 버그 | 높음 | 중간 | 통합 테스트, 리포트 diff 비교 |
| PMI 데이터 접근 어려움 | 중간 | 중간 | 여러 API 조사, Fallback 계획 |
| 스코프 확장 욕심 | 낮음 | 높음 | "일단 2개만" 원칙 고수 |

---

## Timeline Estimate

**총 예상 기간**: 1-2주 (1인 개발)

- **Week 1**: Story 1-2 (PMI 구현 + 패턴 추출)
- **Week 2**: Story 3-4 (리팩토링 + 문서화)

---

## 다음 단계

1. Epic 1 완료 후 1주일 실사용 (피드백 수집)
2. Story 1부터 순차 진행
3. Story 2 완료 시 설계 리뷰 (과도한 추상화 방지)
4. Story 3 완료 후 통합 테스트
5. Epic 3 (자동화) 진행 또는 시나리오 1-2개 더 추가 고려

---

**Epic Owner**: Developer (본인)
**Created**: 2025-11-18
**Last Updated**: 2025-11-18
