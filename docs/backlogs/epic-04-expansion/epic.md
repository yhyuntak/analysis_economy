# Epic 4: Agent A 진화 - 이벤트 감지 및 다중 시나리오

## Epic 개요

**ID**: EPIC-004
**제목**: Agent A 진화 - 이벤트 기반 트리거 및 다중 시나리오 운영
**Priority**: Should Have
**Story Points**: 21
**Target Release**: Release 2 (Month 4-6)
**Status**: 대기 중
**의존성**: Epic 3 완료 필요

---

## PM 관점

### 비즈니스 목표

1. **반응 속도 향상**: 정기 실행 → 이벤트 발생 시 즉시 분석 (1주 → 1시간)
2. **섹터 커버리지 확대**: 5-7개 시나리오 동시 운영으로 전 시장 대응
3. **경쟁 우위 확보**: 주요 경제 이벤트 발생 시 자동 분석으로 시장 대응 시간 단축

### 사용자 가치

**Target User**: 적극적 투자자

**Pain Point**:
- 금리 발표, FOMC 등 중요 이벤트 발생 시 수동으로 분석 실행
- 뉴스 확인하고 → 관련 시나리오 찾고 → 실행하는 번거로움
- 이벤트 놓치면 시장 대응 늦음

**Provided Value**:
- FED 금리 인하 발표 → 30분 내 금융/부동산 섹터 영향 분석 완료
- PMI 발표 → 자동으로 산업재 분석 트리거
- 사용자는 슬랙 알림만 확인하면 됨

### Success Metrics

- [ ] **이벤트 감지 정확도**: 90% 이상 (테스트 이벤트 10개 중 9개 감지)
- [ ] **대응 시간**: 이벤트 발생 → 분석 완료 30분 이내
- [ ] **오탐률**: 15% 이하 (불필요한 알림 최소화)
- [ ] **시나리오 실행 성공률**: 95% 이상

---

## Designer 관점

### UX 목표

1. **이벤트 맥락 제공**: 왜 이 분석이 실행되었는지 명확히
2. **긴급도 표시**: High/Medium/Low로 우선순위 전달
3. **다중 리포트 관리**: 날짜/시나리오별 체계적 정리

### 이벤트 알림 포맷

```
🚨 긴급 이벤트 감지 (HIGH)

📰 이벤트: FED 기준금리 0.5% 인하 발표
🕐 감지 시각: 2025-11-18 03:15 (동부 시간 02:15)
🔗 출처: Federal Reserve 공식 발표

🤖 자동 분석 시작:
  ✓ 금리 인하 대응 시나리오 (시나리오 #2)
  ✓ 금융 섹터 영향 분석
  ✓ 부동산 섹터 영향 분석

⏳ 예상 완료: 30분 이내
```

```
✅ 이벤트 분석 완료

📰 이벤트: FED 기준금리 0.5% 인하
⏰ 분석 완료: 2025-11-18 03:42 (소요 27분)

📊 핵심 인사이트:
1. 금융 섹터: 은행주 단기 악재, 자산운용사 수혜 예상
2. 부동산: REITs 급등 가능성, 주택 건설주 긍정
3. 투자 전략: 방어적 배당주 → 성장주 로테이션 고려

📄 상세 리포트: [링크1] [링크2]
```

### 리포트 관리 구조

```
reports/
├── 2025-11-18/
│   ├── event_fed_rate_cut/
│   │   ├── interest_rate_impact_20251118_0342.md
│   │   ├── financials_analysis_20251118_0342.md
│   │   └── real_estate_analysis_20251118_0342.md
│   └── scheduled_semiconductor_weekly_20251118_0900.md
└── 2025-11-11/
    └── ...
```

---

## Developer 관점

### 기술 목표

1. **RSS/뉴스 모니터링**: 실시간 경제 뉴스 피드 감시
2. **키워드 기반 이벤트 감지**: NLP 또는 규칙 기반 이벤트 분류
3. **이벤트-시나리오 매핑**: 이벤트 타입별 적합한 시나리오 자동 선택
4. **다중 실행 관리**: 여러 시나리오 순차 또는 병렬 실행

### Architecture Overview

```
agents/
└── monitor/
    ├── event_detector.py       # 이벤트 감지 엔진
    ├── feed_watcher.py         # RSS/뉴스 피드 모니터링
    ├── event_classifier.py     # 이벤트 분류 및 우선순위
    ├── scenario_mapper.py      # 이벤트-시나리오 매핑
    └── executor.py             # 다중 시나리오 실행

config/
├── event_rules.yaml            # 이벤트 감지 규칙
└── scenario_mapping.yaml       # 이벤트-시나리오 매핑
```

### 이벤트 감지 규칙 예시

```yaml
# config/event_rules.yaml
event_rules:
  - name: fed_rate_change
    keywords:
      - "Federal Reserve"
      - "기준금리"
      - "FOMC"
      - "interest rate"
    priority: HIGH
    sources:
      - https://www.federalreserve.gov/feeds/
      - https://www.reuters.com/finance/rss
    scenarios:
      - interest-rate-impact
      - financials-analysis
      - real-estate-analysis

  - name: pmi_release
    keywords:
      - "PMI"
      - "제조업 지수"
      - "Manufacturing Index"
    priority: MEDIUM
    sources:
      - https://www.markiteconomics.com/rss
    scenarios:
      - pmi-analysis
      - industrials-analysis

  - name: oil_price_surge
    keywords:
      - "유가"
      - "oil price"
      - "crude oil"
    threshold: "+5%"           # 일일 변동 5% 이상
    priority: MEDIUM
    scenarios:
      - energy-analysis
      - materials-analysis
```

### 이벤트 감지 로직

```python
# agents/monitor/event_detector.py
import feedparser
import re
from typing import List, Dict, Any
from datetime import datetime

class EventDetector:
    def __init__(self, rules: Dict[str, Any]):
        self.rules = rules
        self.detected_events = []

    def watch_feeds(self):
        """
        RSS 피드 모니터링 (무한 루프)
        """
        for rule in self.rules:
            for source in rule['sources']:
                feed = feedparser.parse(source)

                for entry in feed.entries:
                    if self._is_event_match(entry, rule):
                        event = self._create_event(entry, rule)
                        self.detected_events.append(event)
                        self._trigger_scenarios(event)

    def _is_event_match(self, entry, rule) -> bool:
        """
        키워드 기반 이벤트 매칭
        """
        text = entry.title + " " + entry.summary
        for keyword in rule['keywords']:
            if keyword.lower() in text.lower():
                return True
        return False

    def _create_event(self, entry, rule) -> Dict[str, Any]:
        """
        이벤트 객체 생성
        """
        return {
            'event_id': self._generate_event_id(),
            'name': rule['name'],
            'title': entry.title,
            'source': entry.link,
            'detected_at': datetime.now(),
            'priority': rule['priority'],
            'scenarios': rule['scenarios']
        }

    def _trigger_scenarios(self, event: Dict[str, Any]):
        """
        이벤트에 매핑된 시나리오 실행
        """
        from agents.monitor.executor import ScenarioExecutor

        executor = ScenarioExecutor()
        for scenario_name in event['scenarios']:
            executor.run_scenario(
                scenario=scenario_name,
                context={'event': event}
            )
```

### 다중 시나리오 실행 전략

```python
# agents/monitor/executor.py
from concurrent.futures import ThreadPoolExecutor
import time

class ScenarioExecutor:
    def __init__(self, max_parallel: int = 3):
        self.max_parallel = max_parallel

    def run_multiple_scenarios(self, scenarios: List[str], context: Dict):
        """
        여러 시나리오 병렬 또는 순차 실행
        """
        if len(scenarios) <= 2:
            # 소수면 순차 실행 (리소스 절약)
            for scenario in scenarios:
                self.run_scenario(scenario, context)
        else:
            # 다수면 병렬 실행 (속도 우선)
            with ThreadPoolExecutor(max_workers=self.max_parallel) as executor:
                futures = [
                    executor.submit(self.run_scenario, s, context)
                    for s in scenarios
                ]
                results = [f.result() for f in futures]
```

### Implementation Complexity

**총 Story Points**: 21

- **Story 1**: RSS/뉴스 피드 모니터링 (5pt) - 중간 난이도
- **Story 2**: 키워드 기반 이벤트 감지 로직 (5pt) - 중간 난이도
- **Story 3**: 이벤트-시나리오 매핑 및 자동 트리거 (5pt) - 중간 난이도
- **Story 4**: 시나리오 #2 금리 인하 대응 추가 (6pt) - 중간 난이도

### 주요 기술 과제

1. **RSS 피드 모니터링 효율성**
   - **해결**: 폴링 간격 5-10분, 중복 이벤트 필터링

2. **오탐 (False Positive) 방지**
   - **해결**: 키워드 조합, 신뢰도 임계값, 화이트리스트 소스

3. **다중 실행 리소스 관리**
   - **해결**: LLM API 호출 제한, 병렬 실행 3개로 제한

4. **이벤트 중복 감지**
   - **해결**: 이벤트 해시, 24시간 내 동일 이벤트 무시

### Dependencies

```txt
# 추가 의존성
feedparser==6.0.10          # RSS 파싱
newspaper3k==0.2.8          # 뉴스 추출 (선택)
```

---

## User Stories

이 Epic은 4개의 User Story로 구성됩니다:

1. **[Story 1] RSS/뉴스 피드 모니터링** (5pt)
   - RSS 피드 파서 구현
   - 폴링 스케줄러
   - 피드 소스 설정 (FED, Reuters, Bloomberg)

2. **[Story 2] 키워드 기반 이벤트 감지 로직** (5pt)
   - 이벤트 규칙 정의 (YAML)
   - 키워드 매칭 엔진
   - 이벤트 분류 및 우선순위

3. **[Story 3] 이벤트-시나리오 매핑 및 자동 트리거** (5pt)
   - 시나리오 매핑 설정
   - 자동 실행 오케스트레이션
   - 다중 실행 관리 (순차/병렬)

4. **[Story 4] 시나리오 #2 금리 인하 대응 추가** (6pt)
   - 금리 데이터 수집
   - 금융/부동산 섹터 ETF 분석
   - LLM 프롬프트 최적화

---

## Acceptance Criteria

Epic 전체가 완료되었다고 판단하는 기준:

- [ ] RSS 피드 모니터링 24시간 이상 안정 동작
- [ ] 테스트 이벤트 10개 중 9개 이상 정확히 감지
- [ ] 금리 발표 이벤트 → 30분 내 3개 시나리오 분석 완료
- [ ] 오탐률 15% 이하 (불필요한 알림 최소화)
- [ ] 이벤트 알림에 맥락 정보 포함 (이벤트 제목, 출처, 우선순위)
- [ ] 다중 리포트 날짜/이벤트별 폴더에 체계적 저장

---

## Risks & Mitigation

| 리스크 | 영향 | 확률 | 완화 전략 |
|--------|------|------|-----------|
| 오탐 과다 (알림 피로) | 높음 | 높음 | 키워드 정교화, 신뢰도 임계값, 사용자 피드백 |
| RSS 피드 변경/중단 | 중간 | 중간 | 다중 소스 백업, fallback 로직 |
| LLM API 비용 급증 | 높음 | 중간 | 이벤트당 실행 시나리오 제한 (최대 3개) |
| 이벤트 누락 (미감지) | 중간 | 중간 | 정기 실행 병행, 중요 소스 추가 |

---

## Timeline Estimate

**총 예상 기간**: 2-3주 (1인 개발)

- **Week 1**: Story 1-2 (피드 모니터링 + 이벤트 감지)
- **Week 2**: Story 3 (매핑 + 자동 트리거)
- **Week 3**: Story 4 (금리 시나리오) + 통합 테스트

---

## 다음 단계

1. Epic 3 완료 및 4주 안정성 검증 후 진행
2. Story 1부터 순차 진행
3. Story 2 완료 후 1주일 오탐률 모니터링
4. Story 3 완료 후 이벤트 시뮬레이션 테스트
5. Epic 5 (UI) 또는 추가 시나리오 고려

---

**Epic Owner**: Developer (본인)
**Created**: 2025-11-18
**Last Updated**: 2025-11-18
