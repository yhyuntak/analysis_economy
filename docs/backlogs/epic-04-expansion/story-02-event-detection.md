# Story 2: 키워드 기반 이벤트 감지 로직

**Epic**: Epic 4 - Agent A 진화
**Points**: 5
**Priority**: Should
**Status**: Todo

---

## User Story

**As a** Agent A
**I want** 키워드 매칭으로 경제 이벤트 분류
**So that** 관련 시나리오를 자동으로 트리거할 수 있다

---

## Background & Context

### 문제 정의
- Story 1에서 뉴스 기사는 수집했지만 무작위
- 모든 기사가 중요한 건 아님 (80%는 노이즈)
- 특정 경제 이벤트만 분석 트리거 해야 함

### 솔루션
- 키워드 규칙 기반 이벤트 분류
- 우선순위 (HIGH/MEDIUM/LOW) 자동 할당
- LLM 없이 빠른 필터링 (비용 절감)

### 범위
- Phase 1: 단순 키워드 매칭
- Phase 2: LLM 기반 의미 분석 (나중에)

---

## Acceptance Criteria

### 필수 조건

- [ ] **AC1**: 이벤트 규칙 YAML 정의
  - 최소 10개 이벤트 타입 정의
  - 각 이벤트별 키워드 리스트
  - 우선순위 (HIGH/MEDIUM/LOW)

- [ ] **AC2**: 키워드 매칭 엔진 구현
  - 정규표현식 지원
  - 대소문자 무시
  - 다국어 지원 (영어/한국어)

- [ ] **AC3**: 신뢰도 점수 계산
  - 키워드 매칭 개수 기반
  - 소스 가중치 반영 (FED > Yahoo)
  - 임계값 이상만 이벤트 트리거

- [ ] **AC4**: 오탐률 < 15%
  - 테스트 기사 100개 준비
  - 수동 레이블링
  - 정확도 측정

- [ ] **AC5**: 이벤트 저장 및 로깅
  - 감지된 이벤트 JSON 저장
  - 타임스탬프, 기사 링크, 신뢰도 포함

---

## Tasks

### 1. 이벤트 규칙 정의

**config/event_rules.yaml**:
```yaml
events:
  - id: fed_rate_change
    name: "연준 기준금리 변경"
    keywords:
      - "Federal Reserve"
      - "FOMC"
      - "interest rate"
      - "기준금리"
      - "금리 인상"
      - "금리 인하"
    priority: HIGH
    min_confidence: 0.7

  - id: fed_speech
    name: "연준 의장 발언"
    keywords:
      - "Jerome Powell"
      - "파월 의장"
      - "Fed Chair"
    priority: HIGH
    min_confidence: 0.6

  - id: oil_price_surge
    name: "유가 급등"
    keywords:
      - "oil price"
      - "crude oil"
      - "WTI"
      - "Brent"
      - "유가"
      - "원유"
    conditions:
      - keyword: "surge|급등|soar|spike"
        required: true
    priority: HIGH
    min_confidence: 0.7

  - id: semiconductor_shortage
    name: "반도체 공급 이슈"
    keywords:
      - "semiconductor"
      - "chip shortage"
      - "반도체"
      - "칩 부족"
      - "TSMC"
      - "삼성전자"
    priority: MEDIUM
    min_confidence: 0.6

  - id: inflation_data
    name: "인플레이션 지표 발표"
    keywords:
      - "CPI"
      - "inflation"
      - "소비자물가"
      - "인플레이션"
      - "물가 상승"
    priority: HIGH
    min_confidence: 0.7

  - id: gdp_release
    name: "GDP 발표"
    keywords:
      - "GDP"
      - "economic growth"
      - "경제성장률"
    priority: MEDIUM
    min_confidence: 0.6

  - id: unemployment_data
    name: "고용 지표"
    keywords:
      - "unemployment"
      - "jobless"
      - "nonfarm payrolls"
      - "실업률"
      - "고용"
    priority: MEDIUM
    min_confidence: 0.6

  - id: fda_approval
    name: "FDA 신약 승인"
    keywords:
      - "FDA approval"
      - "FDA 승인"
      - "drug approval"
      - "신약"
    priority: MEDIUM
    min_confidence: 0.7

  - id: geopolitical_risk
    name: "지정학 리스크"
    keywords:
      - "war"
      - "conflict"
      - "sanctions"
      - "전쟁"
      - "분쟁"
      - "제재"
    priority: HIGH
    min_confidence: 0.6

  - id: earnings_surprise
    name: "어닝 서프라이즈"
    keywords:
      - "earnings beat"
      - "earnings miss"
      - "실적 발표"
      - "깜짝 실적"
    priority: LOW
    min_confidence: 0.5

# 소스 가중치
source_weights:
  "Federal Reserve": 1.5
  "Reuters Economics": 1.3
  "Bloomberg Markets": 1.2
  "Yahoo Finance": 1.0
  "MarketWatch Economic Report": 1.0
```

### 2. EventClassifier 구현

**agents/monitor/feeds/event_classifier.py**:
```python
# agents/monitor/feeds/event_classifier.py
import re
import yaml
from typing import List, Dict, Optional
from datetime import datetime
from pathlib import Path
from src.utils.logger import setup_logger

logger = setup_logger(__name__)

class EventClassifier:
    """키워드 기반 이벤트 분류기"""

    def __init__(self, rules_path: str = "config/event_rules.yaml"):
        self.rules_path = rules_path
        self.event_rules = []
        self.source_weights = {}
        self._load_rules()

    def _load_rules(self):
        """이벤트 규칙 로드"""
        with open(self.rules_path, 'r', encoding='utf-8') as f:
            config = yaml.safe_load(f)

        self.event_rules = config['events']
        self.source_weights = config.get('source_weights', {})

        logger.info(f"Loaded {len(self.event_rules)} event rules")

    def _calculate_keyword_score(self, text: str, keywords: List[str]) -> float:
        """키워드 매칭 점수 계산"""
        text_lower = text.lower()
        matched = 0

        for keyword in keywords:
            # 정규표현식 패턴 생성 (단어 경계 고려)
            pattern = r'\b' + re.escape(keyword.lower()) + r'\b'

            if re.search(pattern, text_lower):
                matched += 1

        # 매칭된 키워드 비율
        return matched / len(keywords) if keywords else 0.0

    def _check_conditions(self, text: str, conditions: List[Dict]) -> bool:
        """추가 조건 체크"""
        if not conditions:
            return True

        text_lower = text.lower()

        for condition in conditions:
            keyword_pattern = condition['keyword']
            required = condition.get('required', False)

            if re.search(keyword_pattern, text_lower):
                return True
            elif required:
                return False

        return not any(c.get('required', False) for c in conditions)

    def classify_article(self, article: Dict) -> Optional[Dict]:
        """단일 기사 분류"""
        # 제목 + 요약 결합
        text = f"{article.get('title', '')} {article.get('summary', '')}"

        best_match = None
        best_score = 0.0

        for event_rule in self.event_rules:
            # 키워드 점수
            keyword_score = self._calculate_keyword_score(text, event_rule['keywords'])

            # 추가 조건 체크
            if 'conditions' in event_rule:
                if not self._check_conditions(text, event_rule['conditions']):
                    continue

            # 소스 가중치 반영
            source_weight = self.source_weights.get(article.get('source', ''), 1.0)
            final_score = keyword_score * source_weight

            # 최소 신뢰도 체크
            min_confidence = event_rule.get('min_confidence', 0.5)

            if final_score >= min_confidence and final_score > best_score:
                best_score = final_score
                best_match = {
                    'event_id': event_rule['id'],
                    'event_name': event_rule['name'],
                    'priority': event_rule['priority'],
                    'confidence': round(final_score, 2),
                    'article': article,
                    'detected_at': datetime.now().isoformat()
                }

        if best_match:
            logger.info(
                f"Detected event: {best_match['event_name']} "
                f"(confidence: {best_match['confidence']}) "
                f"from article: {article.get('title', '')[:50]}"
            )

        return best_match

    def classify_articles(self, articles: List[Dict]) -> List[Dict]:
        """여러 기사 분류"""
        events = []

        for article in articles:
            event = self.classify_article(article)

            if event:
                events.append(event)

        logger.info(f"Classified {len(events)} events from {len(articles)} articles")

        return events

    def save_events(self, events: List[Dict], output_dir: str = "data/events"):
        """이벤트 저장"""
        output_path = Path(output_dir)
        output_path.mkdir(parents=True, exist_ok=True)

        today = datetime.now().strftime('%Y%m%d')
        output_file = output_path / f"events_{today}.json"

        # 기존 이벤트 로드
        import json
        existing = []
        if output_file.exists():
            with open(output_file, 'r', encoding='utf-8') as f:
                existing = json.load(f)

        # 새 이벤트 추가
        existing.extend(events)

        # 저장
        with open(output_file, 'w', encoding='utf-8') as f:
            json.dump(existing, f, indent=2, ensure_ascii=False)

        logger.info(f"Saved {len(events)} events to {output_file}")

if __name__ == "__main__":
    # 테스트
    classifier = EventClassifier()

    # 샘플 기사
    test_articles = [
        {
            'title': 'Federal Reserve raises interest rates by 0.25%',
            'summary': 'The FOMC decided to increase the federal funds rate...',
            'source': 'Federal Reserve'
        },
        {
            'title': 'Oil prices surge 15% amid Middle East tensions',
            'summary': 'WTI crude oil jumped to $95 per barrel...',
            'source': 'Reuters Economics'
        },
        {
            'title': 'Apple reports record earnings',
            'summary': 'Tech giant beats Wall Street estimates...',
            'source': 'Yahoo Finance'
        }
    ]

    events = classifier.classify_articles(test_articles)

    print(f"\n=== Detected {len(events)} events ===")
    for event in events:
        print(f"[{event['priority']}] {event['event_name']} (confidence: {event['confidence']})")
        print(f"  Article: {event['article']['title']}")
```

### 3. FeedWatcher 통합

**agents/monitor/feeds/feed_watcher.py 수정**:
```python
# FeedWatcher 클래스에 추가
from .event_classifier import EventClassifier

class FeedWatcher:
    def __init__(self, ...):
        # ... 기존 코드
        self.classifier = EventClassifier()

    def fetch_all_feeds(self) -> List[Dict]:
        """모든 피드 가져오기 + 이벤트 감지"""
        all_articles = []

        for feed_config in self.feeds:
            articles = self.fetch_feed(feed_config)
            all_articles.extend(articles)

        if all_articles:
            self._save_articles(all_articles)
            self._save_seen_hashes()

            # 이벤트 분류
            events = self.classifier.classify_articles(all_articles)

            if events:
                self.classifier.save_events(events)
                logger.info(f"Detected {len(events)} events")

        return all_articles
```

### 4. 테스트 데이터셋 준비

**tests/test_event_classifier.py**:
```python
# tests/test_event_classifier.py
import json
from agents.monitor.feeds.event_classifier import EventClassifier

def load_test_articles():
    """테스트 기사 로드 (수동 레이블링)"""
    # 실제로는 과거 뉴스 100개 수집 후 수동 레이블링
    # 여기서는 샘플만
    return [
        {
            'title': 'Fed raises rates by 25 basis points',
            'summary': 'FOMC meeting results...',
            'source': 'Federal Reserve',
            'expected_event': 'fed_rate_change'
        },
        # ... 99개 더
    ]

def test_accuracy():
    """분류 정확도 테스트"""
    classifier = EventClassifier()
    test_data = load_test_articles()

    correct = 0
    total = len(test_data)

    for article in test_data:
        event = classifier.classify_article(article)

        if event and event['event_id'] == article['expected_event']:
            correct += 1
        elif not event and not article.get('expected_event'):
            correct += 1

    accuracy = correct / total
    print(f"Accuracy: {accuracy * 100:.1f}%")

    assert accuracy >= 0.85, f"Accuracy too low: {accuracy}"

if __name__ == "__main__":
    test_accuracy()
```

---

## Technical Notes

### 키워드 vs LLM
- **키워드**: 빠름, 저렴, 단순
- **LLM**: 느림, 비쌈, 정교함

초기에는 키워드로 필터링 → LLM은 Story 3에서 사용

### 정규표현식 주의
- 단어 경계 (`\b`) 사용
- 예: "rate"가 "separate"에 매칭 방지

### 다국어 지원
- 영어/한국어 키워드 혼재
- 나중에 번역 API 추가 가능

---

## Definition of Done

- [ ] event_rules.yaml 작성 (10개 이상 이벤트)
- [ ] EventClassifier 구현 완료
- [ ] FeedWatcher 통합
- [ ] 테스트 데이터셋 준비
- [ ] 정확도 85% 이상 달성
- [ ] 이벤트 JSON 저장 확인
- [ ] 문서 업데이트
- [ ] AC 모두 충족

---

## Dependencies

**Prerequisite**:
- Story 1 (RSS 모니터링) 완료

**Blocks**:
- Story 3 (시나리오 매핑) - 이 스토리가 이벤트 감지 담당

---

## Estimated Time

- **이벤트 규칙 정의**: 2시간
- **EventClassifier 구현**: 2시간
- **테스트 데이터 준비**: 1시간
- **정확도 개선**: 2시간
- **Total**: 5 Story Points

---

## Risks & Mitigation

**Risk 1**: 오탐률 높음
- **원인**: 키워드 모호성
- **완화**: 조건 추가, 소스 가중치, LLM 검증 (나중에)

**Risk 2**: 이벤트 놓침
- **원인**: 키워드 리스트 불완전
- **완화**: 정기적 키워드 업데이트, 로그 분석

---

## Success Metrics

- ✅ 정확도 85% 이상
- ✅ 오탐률 15% 이하
- ✅ 처리 속도 < 1초/기사
- ✅ HIGH 우선순위 이벤트 놓침률 0%

---

## References

- [정규표현식 문서](https://docs.python.org/3/library/re.html)
- Story 1: RSS 모니터링
- Story 3: 시나리오 매핑
