# User Story 1: 시나리오 #6 PMI 분석 구현 (복붙 방식)

## User Story

**As a** 경기 사이클 투자자
**I want** PMI 발표 시 산업재/소재 섹터 영향을 자동 분석하는 기능
**So that** 제조업 경기 선행 지표를 통해 섹터 로테이션 기회를 포착할 수 있다

---

## Story Details

**Story ID**: EPIC-002-S1
**Epic**: Epic 2 - 공통 프레임워크 및 PMI 시나리오 추가
**Story Points**: 3
**Priority**: Must Have
**Assignee**: Developer (본인)
**Status**: 대기 중

---

## Acceptance Criteria

### AC1: PMI 데이터 수집
- [ ] 미국 제조업 PMI (ISM Manufacturing Index) 최근 데이터 수집
- [ ] 중국 제조업 PMI (Caixin Manufacturing PMI) 최근 데이터 수집
- [ ] 전월 대비 변화량 계산

### AC2: 산업재/소재 섹터 데이터 수집
- [ ] 산업재 ETF (XLI) 주간/월간 변동률
- [ ] 소재 ETF (XLB) 주간/월간 변동률
- [ ] 운송 ETF (IYT) 주간/월간 변동률

### AC3: LLM 분석
- [ ] PMI 데이터와 섹터 주가를 Claude API로 분석
- [ ] PMI 확장/위축 해석
- [ ] 섹터 주가 반응 분석
- [ ] 투자 시사점 도출

### AC4: 리포트 생성
- [ ] Markdown 리포트 생성
- [ ] 리포트 구조는 시나리오 #1과 동일
- [ ] 파일명: `pmi_analysis_YYYYMMDD.md`

### AC5: CLI 실행
- [ ] `python run_scenario.py --scenario pmi-analysis` 명령어로 실행
- [ ] 실행 시간 < 2분

---

## Tasks

### Task 1: 시나리오 #1 코드 복사
- [ ] `src/scenarios/semiconductor_weekly.py` 복사
- [ ] `src/scenarios/pmi_analysis.py`로 이름 변경
- [ ] 클래스명 변경: `PMIAnalysisScenario`

### Task 2: PMI 데이터 수집기 구현
- [ ] `src/collectors/economic_data_collector.py` 생성
- [ ] Trading Economics API 또는 FRED API 조사
- [ ] PMI 데이터 수집 로직 구현
  ```python
  class EconomicDataCollector:
      def collect_pmi_data(self) -> dict:
          """미국 및 중국 PMI 데이터 수집"""
          pass
  ```

### Task 3: 산업재/소재 ETF 수집
- [ ] 기존 `StockCollector` 재사용
- [ ] 심볼 리스트: `["XLI", "XLB", "IYT"]`

### Task 4: LLM 프롬프트 작성
- [ ] PMI 분석용 프롬프트 템플릿 작성
  ```
  다음 PMI 데이터와 산업재/소재 섹터 주가를 분석하시오:

  미국 PMI: {us_pmi} (전월 대비 {change})
  중국 PMI: {china_pmi}
  산업재 ETF (XLI): {xli_change}%
  소재 ETF (XLB): {xlb_change}%

  분석 요청:
  1. PMI 변화의 의미 (확장/위축, 50 기준)
  2. 섹터 주가 반응 해석
  3. 향후 투자 시사점
  ```

### Task 5: 리포트 템플릿 수정
- [ ] 반도체 → PMI로 제목 변경
- [ ] 주요 지표 테이블 수정 (ETF → PMI + 섹터 ETF)

### Task 6: 통합 테스트
- [ ] CLI 명령어 실행 테스트
- [ ] 리포트 생성 확인
- [ ] 리포트 내용 품질 검증

---

## Technical Notes

### PMI 데이터 소스 옵션

**Option 1: FRED API (추천)**
```python
import requests

def get_us_pmi():
    # ISM Manufacturing PMI (NAPM)
    url = "https://api.stlouisfed.org/fred/series/observations"
    params = {
        "series_id": "NAPM",
        "api_key": FRED_API_KEY,
        "file_type": "json",
        "sort_order": "desc",
        "limit": 2  # 최근 2개월
    }
    response = requests.get(url, params=params)
    data = response.json()
    return data['observations']
```

**Option 2: Trading Economics API**
- 무료 티어 제한적
- 더 많은 국가 PMI 제공

**Option 3: 수동 입력 (임시)**
- 초기 테스트용으로 PMI 값 하드코딩
- Epic 2 완료 후 API 통합

### PMI 해석 기준
- **PMI > 50**: 제조업 확장
- **PMI < 50**: 제조업 위축
- **PMI 상승**: 산업재/소재 긍정적
- **PMI 하락**: 방어적 섹터로 전환 고려

---

## Definition of Done

- [ ] 모든 Acceptance Criteria 충족
- [ ] 모든 Tasks 완료
- [ ] PMI 시나리오 리포트 생성 성공
- [ ] 리포트 품질 확인 (수동 검토)
- [ ] 실행 시간 < 2분 검증

---

## Dependencies

**선행 작업**:
- Epic 1 완료 (StockCollector, LLMAnalyzer 재사용)

**후속 작업**:
- Story 2: 공통 패턴 식별 (이 시나리오와 #1 비교 분석)

---

## Risks

| 리스크 | 완화 전략 |
|--------|-----------|
| PMI 데이터 API 접근 어려움 | 수동 입력으로 우선 구현, API는 나중에 |
| 중국 PMI 데이터 신뢰성 | 미국 PMI만으로도 분석 가능하게 설계 |
| LLM 프롬프트 품질 | 반도체 프롬프트 참고, 반복 테스트 |

---

## Estimated Time

**Story Points**: 3

**예상 시간**: 4-8시간 (0.5-1일)
- Task 1-2 (복사 + PMI 수집): 2-3시간
- Task 3-4 (ETF 수집 + 프롬프트): 1-2시간
- Task 5-6 (리포트 + 테스트): 1-3시간

---

## Notes

- **하드코딩 허용**: 빠른 검증이 목표, PMI 값 직접 입력 OK
- **API 통합은 나중에**: Epic 2 완료 후 개선 가능
- **중요한 건 구조**: 시나리오 1과 얼마나 유사한지 파악
- **프레임워크 설계 준비**: 이 Story 완료 후 두 시나리오를 비교 분석

---

**Created**: 2025-11-18
**Last Updated**: 2025-11-18
**Status**: Ready for Development
