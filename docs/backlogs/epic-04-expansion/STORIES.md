# Epic 4: Agent A 진화 - User Stories 개요

## Story 목록

### Story 1: RSS/뉴스 피드 모니터링
**Points**: 5 | **Priority**: Should

**User Story**:
As a Agent A, I want RSS 피드를 주기적으로 모니터링, So that 주요 경제 이벤트를 자동으로 감지할 수 있다

**Acceptance Criteria**:
- [ ] RSS 피드 파서 구현 (feedparser 라이브러리)
- [ ] FED, Reuters, Bloomberg 등 주요 소스 등록
- [ ] 5-10분 간격 폴링
- [ ] 새 기사 감지 및 저장
- [ ] 24시간 이상 안정 동작

**Tasks**:
1. `agents/monitor/feed_watcher.py` 구현
2. RSS 소스 설정 (`config/feed_sources.yaml`)
3. 폴링 스케줄러 (APScheduler 사용)
4. 중복 기사 필터링 (해시 기반)
5. 새 기사 DB 또는 JSON 저장
6. 24시간 안정성 테스트

---

### Story 2: 키워드 기반 이벤트 감지 로직
**Points**: 5 | **Priority**: Should

**User Story**:
As a Agent A, I want 키워드 매칭으로 경제 이벤트 분류, So that 관련 시나리오를 자동으로 트리거할 수 있다

**Acceptance Criteria**:
- [ ] 이벤트 규칙 정의 (YAML)
- [ ] 키워드 매칭 엔진 구현
- [ ] 이벤트 우선순위 (HIGH/MEDIUM/LOW) 분류
- [ ] 오탐률 < 15%
- [ ] 테스트 기사 10개 중 9개 정확히 분류

**Tasks**:
1. `config/event_rules.yaml` 작성
   ```yaml
   - name: fed_rate_change
     keywords: ["Federal Reserve", "기준금리", "FOMC"]
     priority: HIGH
   ```
2. `agents/monitor/event_classifier.py` 구현
3. 키워드 매칭 로직 (정규표현식)
4. 신뢰도 점수 계산 (키워드 개수, 출처 가중치)
5. 테스트 데이터셋 준비 (과거 뉴스 10건)
6. 분류 정확도 측정

---

### Story 3: 이벤트-시나리오 매핑 및 자동 트리거
**Points**: 5 | **Priority**: Should

**User Story**:
As a Agent A, I want 이벤트 감지 시 적합한 시나리오를 자동 실행, So that 사용자 개입 없이 분석이 완료된다

**Acceptance Criteria**:
- [ ] 시나리오 매핑 설정 (YAML)
- [ ] 이벤트 → 시나리오 자동 트리거
- [ ] 다중 시나리오 실행 관리 (순차 또는 병렬)
- [ ] 실행 로그 및 알림
- [ ] 30분 내 분석 완료

**Tasks**:
1. `config/scenario_mapping.yaml` 작성
   ```yaml
   fed_rate_change:
     scenarios:
       - interest-rate-impact
       - financials-analysis
       - real-estate-analysis
   ```
2. `agents/monitor/scenario_mapper.py` 구현
3. 자동 트리거 로직
   - 이벤트 감지 → 매핑 조회 → 시나리오 실행
4. 다중 실행 관리
   - 순차 실행 (기본)
   - 병렬 실행 (max 3개)
5. 이벤트 알림 포맷 (이벤트 맥락 포함)
6. E2E 테스트 (시뮬레이션)

---

### Story 4: 시나리오 #2 금리 인하 대응 추가
**Points**: 6 | **Priority**: Should

**User Story**:
As a 투자자, I want 금리 변화 시 금융/부동산 섹터 영향 분석, So that 금리 정책에 따른 투자 전략을 수립할 수 있다

**Acceptance Criteria**:
- [ ] 금리 데이터 수집 (FRED API)
- [ ] 금융 ETF (XLF) 및 부동산 ETF (VNQ) 데이터 수집
- [ ] LLM 기반 영향 분석
- [ ] 리포트 생성
- [ ] 시나리오 프레임워크 기반 구현

**Tasks**:
1. `src/scenarios/interest_rate_impact.py` 구현
2. 금리 데이터 수집기 (FRED API)
3. 금융/부동산 섹터 데이터 수집
4. LLM 프롬프트 작성
   - 금리 변화 해석 (인상/인하)
   - 섹터별 영향 분석
   - 투자 시사점
5. 리포트 템플릿
6. 테스트 (과거 금리 변화 데이터)

---

## Epic 4 완료 기준

- [ ] 모든 Story (1-4) 완료
- [ ] RSS 모니터링 24시간 안정 동작
- [ ] 이벤트 감지 정확도 90% 이상
- [ ] 시뮬레이션 이벤트 → 30분 내 분석 완료
- [ ] 시나리오 #1, #2, #6 모두 이벤트 트리거 동작

---

**다음 단계**: Epic 5 (UI) 또는 시나리오 3-7 추가
