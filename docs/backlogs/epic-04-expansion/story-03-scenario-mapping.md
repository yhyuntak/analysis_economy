# Story 3: 이벤트-시나리오 매핑 및 자동 트리거

**Epic**: Epic 4 - Agent A 진화
**Points**: 5
**Priority**: Should
**Status**: Todo

---

## User Story

**As a** Agent A
**I want** 이벤트 감지 시 적합한 시나리오를 자동 실행
**So that** 사용자 개입 없이 분석이 완료된다

---

## Background & Context

### 문제 정의
- Story 2에서 이벤트는 감지했지만 아직 수동 대응
- 각 이벤트마다 어떤 시나리오 실행할지 정의 필요
- 사용자가 매번 트리거하는 건 자동화 목적에 맞지 않음

### 솔루션
- 이벤트 → 시나리오 매핑 설정 (YAML)
- 자동 트리거 로직
- 다중 시나리오 실행 관리

### 범위
- Phase 1: 순차 실행 (안정성 우선)
- Phase 2: 병렬 실행 (속도 우선)

---

## Acceptance Criteria

### 필수 조건

- [ ] **AC1**: 시나리오 매핑 설정 (YAML)
  - 각 이벤트별 실행할 시나리오 정의
  - 우선순위 순서 지정
  - 조건부 실행 (선택)

- [ ] **AC2**: 이벤트 → 시나리오 자동 트리거
  - 이벤트 감지 시 자동 실행
  - 실행 로그 기록
  - 성공/실패 처리

- [ ] **AC3**: 다중 시나리오 실행 관리
  - 순차 실행 (기본)
  - 병렬 실행 (옵션, max 3개)
  - 타임아웃 설정

- [ ] **AC4**: 실행 알림 (슬랙)
  - 이벤트 감지 알림
  - 시나리오 실행 시작 알림
  - 완료/실패 알림

- [ ] **AC5**: E2E 테스트 성공
  - 시뮬레이션 이벤트 투입
  - 30분 내 분석 완료
  - 리포트 생성 확인

---

## Tasks

### 1. 시나리오 매핑 설정

**config/scenario_mapping.yaml**:
```yaml
# 이벤트 → 시나리오 매핑
mappings:
  fed_rate_change:
    scenarios:
      - id: interest-rate-impact
        priority: 1
        required: true
      - id: financials-analysis
        priority: 2
        required: true
      - id: real-estate-analysis
        priority: 3
        required: true
    execution_mode: sequential  # sequential | parallel
    timeout_minutes: 30

  fed_speech:
    scenarios:
      - id: macro-sentiment-analysis
        priority: 1
        required: true
    execution_mode: sequential
    timeout_minutes: 15

  oil_price_surge:
    scenarios:
      - id: energy-sector-analysis
        priority: 1
        required: true
      - id: cross-sector-impact  # 항공, 화학 등
        priority: 2
        required: false
    execution_mode: sequential
    timeout_minutes: 30

  semiconductor_shortage:
    scenarios:
      - id: semiconductor-weekly
        priority: 1
        required: true
    execution_mode: sequential
    timeout_minutes: 20

  inflation_data:
    scenarios:
      - id: macro-economic-update
        priority: 1
        required: true
      - id: consumer-staples-analysis
        priority: 2
        required: true
    execution_mode: sequential
    timeout_minutes: 25

  gdp_release:
    scenarios:
      - id: macro-economic-update
        priority: 1
        required: true
    execution_mode: sequential
    timeout_minutes: 20

  unemployment_data:
    scenarios:
      - id: macro-economic-update
        priority: 1
        required: true
      - id: consumer-discretionary-analysis
        priority: 2
        required: false
    execution_mode: sequential
    timeout_minutes: 20

  fda_approval:
    scenarios:
      - id: healthcare-fda-analysis
        priority: 1
        required: true
    execution_mode: sequential
    timeout_minutes: 15

  geopolitical_risk:
    scenarios:
      - id: risk-assessment
        priority: 1
        required: true
      - id: safe-haven-analysis  # 금, 채권 등
        priority: 2
        required: false
    execution_mode: sequential
    timeout_minutes: 20

# 기본 설정
defaults:
  execution_mode: sequential
  timeout_minutes: 30
  max_parallel: 3
  retry_on_failure: true
  max_retries: 2
```

### 2. ScenarioMapper 구현

**agents/monitor/scenario_mapper.py**:
```python
# agents/monitor/scenario_mapper.py
import yaml
import time
from typing import List, Dict, Optional
from datetime import datetime
from pathlib import Path
import importlib
from concurrent.futures import ThreadPoolExecutor, TimeoutError

from src.utils.logger import setup_logger
from src.notifiers.slack_notifier import SlackNotifier

logger = setup_logger(__name__)

class ScenarioMapper:
    """이벤트 → 시나리오 매핑 및 실행"""

    def __init__(self, mapping_path: str = "config/scenario_mapping.yaml"):
        self.mapping_path = mapping_path
        self.mappings = {}
        self.defaults = {}
        self._load_mappings()

        self.notifier = SlackNotifier()

    def _load_mappings(self):
        """매핑 설정 로드"""
        with open(self.mapping_path, 'r', encoding='utf-8') as f:
            config = yaml.safe_load(f)

        self.mappings = config.get('mappings', {})
        self.defaults = config.get('defaults', {})

        logger.info(f"Loaded scenario mappings for {len(self.mappings)} events")

    def _load_scenario_class(self, scenario_id: str):
        """시나리오 클래스 동적 로드"""
        # 시나리오 ID → 모듈 경로 매핑
        scenario_map = {
            'semiconductor-weekly': 'src.scenarios.semiconductor.SemiconductorAnalyzer',
            'interest-rate-impact': 'src.scenarios.interest_rate.InterestRateAnalyzer',
            'financials-analysis': 'src.scenarios.financials.FinancialsAnalyzer',
            'real-estate-analysis': 'src.scenarios.real_estate.RealEstateAnalyzer',
            # 나머지 시나리오들...
        }

        module_path = scenario_map.get(scenario_id)

        if not module_path:
            raise ValueError(f"Unknown scenario: {scenario_id}")

        # 모듈과 클래스명 분리
        module_name, class_name = module_path.rsplit('.', 1)

        # 동적 import
        module = importlib.import_module(module_name)
        scenario_class = getattr(module, class_name)

        return scenario_class

    def _run_scenario(self, scenario_id: str, event: Dict, timeout_minutes: int = 30) -> Dict:
        """단일 시나리오 실행"""
        logger.info(f"Running scenario: {scenario_id} for event: {event['event_name']}")

        try:
            # 시나리오 클래스 로드
            scenario_class = self._load_scenario_class(scenario_id)

            # 인스턴스 생성 (이벤트 컨텍스트 전달)
            scenario = scenario_class(event_context=event)

            # 타임아웃 설정
            start_time = time.time()

            # 실행
            with ThreadPoolExecutor(max_workers=1) as executor:
                future = executor.submit(scenario.run)

                try:
                    result = future.result(timeout=timeout_minutes * 60)

                    duration = time.time() - start_time

                    logger.info(f"Scenario {scenario_id} completed in {duration:.1f}s")

                    return {
                        'scenario_id': scenario_id,
                        'status': 'success',
                        'result': result,
                        'duration': duration
                    }

                except TimeoutError:
                    logger.error(f"Scenario {scenario_id} timed out after {timeout_minutes} minutes")

                    return {
                        'scenario_id': scenario_id,
                        'status': 'timeout',
                        'error': f'Timeout after {timeout_minutes} minutes'
                    }

        except Exception as e:
            logger.error(f"Scenario {scenario_id} failed: {str(e)}", exc_info=True)

            return {
                'scenario_id': scenario_id,
                'status': 'failed',
                'error': str(e)
            }

    def _run_scenarios_sequential(self, scenarios: List[Dict], event: Dict, timeout_minutes: int) -> List[Dict]:
        """시나리오 순차 실행"""
        results = []

        for scenario_config in scenarios:
            scenario_id = scenario_config['id']
            required = scenario_config.get('required', True)

            result = self._run_scenario(scenario_id, event, timeout_minutes)
            results.append(result)

            # 필수 시나리오 실패 시 중단
            if required and result['status'] != 'success':
                logger.error(f"Required scenario {scenario_id} failed, stopping execution")
                break

        return results

    def _run_scenarios_parallel(self, scenarios: List[Dict], event: Dict, timeout_minutes: int) -> List[Dict]:
        """시나리오 병렬 실행"""
        max_parallel = self.defaults.get('max_parallel', 3)

        with ThreadPoolExecutor(max_workers=max_parallel) as executor:
            futures = []

            for scenario_config in scenarios:
                scenario_id = scenario_config['id']
                future = executor.submit(self._run_scenario, scenario_id, event, timeout_minutes)
                futures.append((scenario_id, future))

            results = []

            for scenario_id, future in futures:
                try:
                    result = future.result()
                    results.append(result)
                except Exception as e:
                    logger.error(f"Parallel execution failed for {scenario_id}: {str(e)}")
                    results.append({
                        'scenario_id': scenario_id,
                        'status': 'failed',
                        'error': str(e)
                    })

        return results

    def trigger_event(self, event: Dict) -> Dict:
        """이벤트에 대한 시나리오 트리거"""
        event_id = event['event_id']

        logger.info(f"=== Triggering scenarios for event: {event['event_name']} ===")

        # 매핑 조회
        mapping = self.mappings.get(event_id)

        if not mapping:
            logger.warning(f"No scenario mapping found for event: {event_id}")
            return {
                'event': event,
                'status': 'no_mapping',
                'scenarios': []
            }

        # 슬랙 알림: 이벤트 감지
        self.notifier.send_message(
            message=f"🔔 이벤트 감지: {event['event_name']}",
            context={
                'confidence': event['confidence'],
                'article_title': event['article']['title'],
                'article_url': event['article']['link'],
                'scenarios': [s['id'] for s in mapping['scenarios']]
            }
        )

        # 실행 모드
        execution_mode = mapping.get('execution_mode', self.defaults.get('execution_mode', 'sequential'))
        timeout_minutes = mapping.get('timeout_minutes', self.defaults.get('timeout_minutes', 30))

        # 시나리오 실행
        start_time = time.time()

        if execution_mode == 'parallel':
            results = self._run_scenarios_parallel(mapping['scenarios'], event, timeout_minutes)
        else:
            results = self._run_scenarios_sequential(mapping['scenarios'], event, timeout_minutes)

        total_duration = time.time() - start_time

        # 결과 요약
        success_count = sum(1 for r in results if r['status'] == 'success')
        failed_count = sum(1 for r in results if r['status'] == 'failed')

        logger.info(
            f"=== Event {event['event_name']} completed: "
            f"{success_count} success, {failed_count} failed in {total_duration:.1f}s ==="
        )

        # 슬랙 알림: 완료
        status_emoji = "✅" if failed_count == 0 else "⚠️"
        self.notifier.send_message(
            message=f"{status_emoji} 이벤트 분석 완료: {event['event_name']}",
            context={
                'success': success_count,
                'failed': failed_count,
                'duration': f"{total_duration:.1f}s",
                'results': results
            }
        )

        return {
            'event': event,
            'status': 'completed',
            'scenarios': results,
            'total_duration': total_duration
        }

if __name__ == "__main__":
    # 테스트
    mapper = ScenarioMapper()

    # 시뮬레이션 이벤트
    test_event = {
        'event_id': 'fed_rate_change',
        'event_name': '연준 기준금리 변경',
        'priority': 'HIGH',
        'confidence': 0.95,
        'article': {
            'title': 'Fed raises rates by 25bps',
            'link': 'https://example.com/article',
            'source': 'Federal Reserve'
        },
        'detected_at': datetime.now().isoformat()
    }

    result = mapper.trigger_event(test_event)

    print(f"\n=== Trigger Result ===")
    print(f"Status: {result['status']}")
    print(f"Scenarios executed: {len(result['scenarios'])}")
    for scenario_result in result['scenarios']:
        print(f"  - {scenario_result['scenario_id']}: {scenario_result['status']}")
```

### 3. FeedWatcher 통합

**agents/monitor/feeds/feed_watcher.py 수정**:
```python
from .scenario_mapper import ScenarioMapper

class FeedWatcher:
    def __init__(self, ...):
        # ... 기존 코드
        self.classifier = EventClassifier()
        self.mapper = ScenarioMapper()

    def fetch_all_feeds(self) -> List[Dict]:
        """모든 피드 가져오기 + 이벤트 감지 + 시나리오 트리거"""
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

                # 시나리오 자동 트리거
                for event in events:
                    try:
                        self.mapper.trigger_event(event)
                    except Exception as e:
                        logger.error(f"Failed to trigger scenarios for event {event['event_id']}: {str(e)}")

        return all_articles
```

### 4. E2E 테스트

**tests/test_scenario_mapping.py**:
```python
# tests/test_scenario_mapping.py
from agents.monitor.scenario_mapper import ScenarioMapper
from datetime import datetime

def test_e2e_trigger():
    """E2E 테스트: 이벤트 → 시나리오 실행 → 리포트 생성"""
    mapper = ScenarioMapper()

    # 시뮬레이션 이벤트
    test_event = {
        'event_id': 'semiconductor_shortage',
        'event_name': '반도체 공급 이슈',
        'priority': 'MEDIUM',
        'confidence': 0.85,
        'article': {
            'title': 'TSMC reports capacity shortage',
            'link': 'https://example.com/tsmc',
            'source': 'Reuters'
        },
        'detected_at': datetime.now().isoformat()
    }

    # 트리거
    result = mapper.trigger_event(test_event)

    # 검증
    assert result['status'] == 'completed'
    assert len(result['scenarios']) > 0
    assert result['scenarios'][0]['status'] == 'success'
    assert result['total_duration'] < 30 * 60  # 30분 이내

    print("✅ E2E 테스트 성공")

if __name__ == "__main__":
    test_e2e_trigger()
```

---

## Technical Notes

### 동적 모듈 로딩
- `importlib` 사용
- 시나리오 클래스 런타임 로드
- 확장성: 새 시나리오 추가 시 코드 수정 최소화

### 타임아웃 처리
- 시나리오별 타임아웃 설정
- ThreadPoolExecutor.result(timeout)
- 무한 루프 방지

### 순차 vs 병렬
- **순차**: 안전, 디버깅 쉬움, 느림
- **병렬**: 빠름, 리소스 사용 높음, 디버깅 어려움

초기엔 순차, 안정화 후 병렬

---

## Definition of Done

- [ ] scenario_mapping.yaml 작성
- [ ] ScenarioMapper 구현 완료
- [ ] FeedWatcher 통합
- [ ] E2E 테스트 성공
- [ ] 슬랙 알림 동작 확인
- [ ] 30분 내 분석 완료
- [ ] 문서 업데이트
- [ ] AC 모두 충족

---

## Dependencies

**Prerequisite**:
- Story 2 (이벤트 감지) 완료
- Epic 3 Story 2 (슬랙 알림) 완료
- Epic 1 (시나리오 프레임워크) 완료

**Blocks**:
- 없음 (Epic 4의 마지막 자동화 스토리)

---

## Estimated Time

- **매핑 설정**: 1시간
- **ScenarioMapper 구현**: 3시간
- **통합**: 1시간
- **E2E 테스트**: 2시간
- **Total**: 5 Story Points

---

## Risks & Mitigation

**Risk 1**: 시나리오 실행 실패로 시스템 멈춤
- **원인**: 예외 처리 미흡
- **완화**: try-except, 타임아웃, 재시도 로직

**Risk 2**: 병렬 실행 시 리소스 고갈
- **원인**: CPU/메모리 부족
- **완화**: max_parallel 제한, 순차 실행 우선

**Risk 3**: 순환 트리거
- **원인**: 시나리오가 새 이벤트 생성 → 무한 루프
- **완화**: 실행 이력 체크, 중복 방지

---

## Success Metrics

- ✅ 시뮬레이션 이벤트 → 30분 내 완료
- ✅ 슬랙 알림 도달률 100%
- ✅ 필수 시나리오 실행률 100%
- ✅ 타임아웃 발생률 < 5%

---

## References

- [Python importlib](https://docs.python.org/3/library/importlib.html)
- [ThreadPoolExecutor](https://docs.python.org/3/library/concurrent.futures.html)
- Story 2: 이벤트 감지
- Epic 3 Story 2: 슬랙 알림
