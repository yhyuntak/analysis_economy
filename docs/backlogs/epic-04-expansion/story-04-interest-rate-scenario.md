# Story 4: 시나리오 #2 금리 인하 대응 추가

**Epic**: Epic 4 - Agent A 진화
**Points**: 6
**Priority**: Should
**Status**: Todo

---

## User Story

**As a** 투자자
**I want** 금리 변화 시 금융/부동산 섹터 영향 분석
**So that** 금리 정책에 따른 투자 전략을 수립할 수 있다

---

## Background & Context

### 문제 정의
- Epic 1에서 반도체 시나리오 1개만 구현됨
- 금리 변화는 다중 섹터(은행, 부동산, 기술주 등)에 광범위한 영향
- Story 3에서 자동 트리거 구현됐지만 실행할 시나리오 부족

### 솔루션
- 시나리오 #2 구현: 금리 인하 대응 분석
- Epic 2의 프레임워크 활용 (BaseScenario 상속)
- 실전 시나리오 문서 기반 분석 로직

### 범위
- Phase 1: 금융/부동산 섹터만
- Phase 2: 교차 영향 (기술주, 소비재 등)

---

## Acceptance Criteria

### 필수 조건

- [ ] **AC1**: 금리 데이터 수집 (FRED API)
  - 연준 기준금리 (Federal Funds Rate)
  - 10년물 국채 금리
  - 30년물 모기지 금리
  - 과거 12개월 추이

- [ ] **AC2**: 금융 섹터 데이터 수집
  - XLF ETF (Financial Select Sector SPDR)
  - 주요 은행 주가 (JPM, BAC, WFC)
  - NIM (순이자마진) 추정치
  - 부실률 데이터

- [ ] **AC3**: 부동산 섹터 데이터 수집
  - VNQ ETF (Vanguard Real Estate Index Fund)
  - REITs 주가
  - 주택 매매 지수
  - 모기지 신청 건수

- [ ] **AC4**: LLM 기반 영향 분석
  - 금리 변화 해석 (인상/인하, 폭)
  - 섹터별 영향 분석
  - 투자 시사점 도출

- [ ] **AC5**: 리포트 생성
  - `reports/interest_rate_impact_YYYYMMDD.md`
  - 금리 동향, 섹터 분석, 투자 전략 포함
  - 차트 (금리 추이, ETF 성과)

---

## Tasks

### 1. 프로젝트 구조

```bash
mkdir -p src/scenarios/interest_rate
mkdir -p src/data_sources/macro
touch src/scenarios/interest_rate/__init__.py
touch src/scenarios/interest_rate/interest_rate_analyzer.py
touch src/data_sources/macro/fred_client.py
```

### 2. FRED API 클라이언트 구현

**src/data_sources/macro/fred_client.py**:
```python
# src/data_sources/macro/fred_client.py
import os
import requests
from typing import Dict, List
from datetime import datetime, timedelta
from src.utils.logger import setup_logger

logger = setup_logger(__name__)

class FREDClient:
    """Federal Reserve Economic Data (FRED) API 클라이언트"""

    BASE_URL = "https://api.stlouisfed.org/fred/series/observations"

    def __init__(self):
        self.api_key = os.getenv('FRED_API_KEY')

        if not self.api_key:
            raise ValueError("FRED_API_KEY not found in environment")

    def get_series(self, series_id: str, months: int = 12) -> List[Dict]:
        """
        특정 시계열 데이터 조회

        Args:
            series_id: FRED 시리즈 ID (예: "DFF" - Federal Funds Rate)
            months: 조회할 과거 기간 (개월)

        Returns:
            [{'date': '2024-01-01', 'value': 5.33}, ...]
        """
        start_date = (datetime.now() - timedelta(days=months * 30)).strftime('%Y-%m-%d')

        params = {
            'series_id': series_id,
            'api_key': self.api_key,
            'file_type': 'json',
            'observation_start': start_date
        }

        try:
            response = requests.get(self.BASE_URL, params=params)
            response.raise_for_status()

            data = response.json()
            observations = data.get('observations', [])

            # 유효한 값만 필터링
            result = []
            for obs in observations:
                value = obs.get('value')

                if value and value != '.':
                    result.append({
                        'date': obs['date'],
                        'value': float(value)
                    })

            logger.info(f"Fetched {len(result)} observations for {series_id}")
            return result

        except Exception as e:
            logger.error(f"Failed to fetch FRED series {series_id}: {str(e)}")
            raise

    def get_federal_funds_rate(self, months: int = 12) -> List[Dict]:
        """연준 기준금리"""
        return self.get_series('DFF', months)

    def get_10year_treasury(self, months: int = 12) -> List[Dict]:
        """10년물 국채 금리"""
        return self.get_series('DGS10', months)

    def get_30year_mortgage(self, months: int = 12) -> List[Dict]:
        """30년물 모기지 금리"""
        return self.get_series('MORTGAGE30US', months)

if __name__ == "__main__":
    # 테스트
    client = FREDClient()

    fed_rate = client.get_federal_funds_rate(months=3)
    print(f"Federal Funds Rate (latest): {fed_rate[-1]}")

    treasury_10y = client.get_10year_treasury(months=3)
    print(f"10Y Treasury (latest): {treasury_10y[-1]}")
```

**.env 업데이트**:
```bash
# FRED API (무료)
# https://fred.stlouisfed.org/docs/api/api_key.html
FRED_API_KEY=your_fred_api_key_here
```

### 3. InterestRateAnalyzer 구현

**src/scenarios/interest_rate/interest_rate_analyzer.py**:
```python
# src/scenarios/interest_rate/interest_rate_analyzer.py
from datetime import datetime
from typing import Dict, Optional
import yfinance as yf

from src.scenarios.base_scenario import BaseScenario
from src.data_sources.macro.fred_client import FREDClient
from src.utils.logger import setup_logger

logger = setup_logger(__name__)

class InterestRateAnalyzer(BaseScenario):
    """금리 변화 영향 분석"""

    SCENARIO_NAME = "interest-rate-impact"

    def __init__(self, event_context: Optional[Dict] = None):
        super().__init__(event_context)
        self.fred_client = FREDClient()

    def collect_data(self) -> Dict:
        """데이터 수집"""
        logger.info("Collecting interest rate and sector data...")

        data = {}

        # 1. 금리 데이터 (FRED)
        data['fed_funds_rate'] = self.fred_client.get_federal_funds_rate(months=12)
        data['treasury_10y'] = self.fred_client.get_10year_treasury(months=12)
        data['mortgage_30y'] = self.fred_client.get_30year_mortgage(months=12)

        # 2. 금융 섹터 (Yahoo Finance)
        xlf = yf.Ticker("XLF")  # Financial Select Sector SPDR
        data['xlf_history'] = xlf.history(period="1y")

        jpm = yf.Ticker("JPM")  # JPMorgan Chase
        data['jpm_history'] = jpm.history(period="1y")

        # 3. 부동산 섹터
        vnq = yf.Ticker("VNQ")  # Vanguard Real Estate ETF
        data['vnq_history'] = vnq.history(period="1y")

        logger.info("Data collection completed")
        return data

    def analyze(self, data: Dict) -> Dict:
        """LLM 기반 분석"""
        logger.info("Analyzing interest rate impact...")

        # 금리 변화 계산
        fed_rate_current = data['fed_funds_rate'][-1]['value']
        fed_rate_3m_ago = data['fed_funds_rate'][-60]['value'] if len(data['fed_funds_rate']) >= 60 else fed_rate_current
        fed_rate_change = fed_rate_current - fed_rate_3m_ago

        # ETF 성과
        xlf_current = data['xlf_history']['Close'].iloc[-1]
        xlf_3m_ago = data['xlf_history']['Close'].iloc[-60] if len(data['xlf_history']) >= 60 else xlf_current
        xlf_return = (xlf_current / xlf_3m_ago - 1) * 100

        vnq_current = data['vnq_history']['Close'].iloc[-1]
        vnq_3m_ago = data['vnq_history']['Close'].iloc[-60] if len(data['vnq_history']) >= 60 else vnq_current
        vnq_return = (vnq_current / vnq_3m_ago - 1) * 100

        # LLM 프롬프트
        prompt = f"""
다음 데이터를 기반으로 금리 변화가 금융/부동산 섹터에 미치는 영향을 분석하세요.

# 금리 동향
- 현재 연준 기준금리: {fed_rate_current:.2f}%
- 3개월 전 대비 변화: {fed_rate_change:+.2f}%p
- 10년물 국채 금리: {data['treasury_10y'][-1]['value']:.2f}%
- 30년물 모기지 금리: {data['mortgage_30y'][-1]['value']:.2f}%

# 섹터 성과 (3개월)
- XLF (금융 ETF): {xlf_return:+.2f}%
- VNQ (부동산 ETF): {vnq_return:+.2f}%
- JPM (JPMorgan): {(data['jpm_history']['Close'].iloc[-1] / data['jpm_history']['Close'].iloc[-60] - 1) * 100:+.2f}%

# 분석 요청
1. 금리 변화 해석 (인상/인하 사이클, 속도)
2. 금융 섹터 영향
   - NIM (순이자마진) 전망
   - 대출 수요 변화
   - 주가 밸류에이션
3. 부동산 섹터 영향
   - 모기지 수요 변화
   - REITs 매력도
   - 상업용/주거용 차별화
4. 투자 전략
   - 매수/매도/관망 추천
   - 주목할 개별 종목
   - 리스크 요인

한국어로 전문 애널리스트 관점에서 작성하세요.
"""

        response = self.call_llm(prompt)

        analysis_result = {
            'rate_direction': '인하' if fed_rate_change < 0 else '인상',
            'rate_change_3m': fed_rate_change,
            'xlf_return_3m': xlf_return,
            'vnq_return_3m': vnq_return,
            'llm_analysis': response
        }

        logger.info("Analysis completed")
        return analysis_result

    def generate_report(self, data: Dict, analysis: Dict) -> str:
        """리포트 생성"""
        logger.info("Generating interest rate impact report...")

        report = f"""# 금리 변화 영향 분석 리포트

**생성 일시**: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}
**시나리오**: 금리 인하 대응 분석

---

## 1. 금리 동향 요약

- **현재 연준 기준금리**: {data['fed_funds_rate'][-1]['value']:.2f}%
- **3개월 변화**: {analysis['rate_change_3m']:+.2f}%p ({analysis['rate_direction']})
- **10년물 국채**: {data['treasury_10y'][-1]['value']:.2f}%
- **30년물 모기지**: {data['mortgage_30y'][-1]['value']:.2f}%

---

## 2. 섹터 성과 (3개월)

| 섹터 | 수익률 | 해석 |
|------|--------|------|
| XLF (금융) | {analysis['xlf_return_3m']:+.2f}% | {'긍정' if analysis['xlf_return_3m'] > 0 else '부정'} |
| VNQ (부동산) | {analysis['vnq_return_3m']:+.2f}% | {'긍정' if analysis['vnq_return_3m'] > 0 else '부정'} |

---

## 3. 심층 분석

{analysis['llm_analysis']}

---

## 4. 데이터 소스

- FRED (Federal Reserve Economic Data)
- Yahoo Finance
- LLM: Claude 3.5 Sonnet

---

**면책 조항**: 이 리포트는 정보 제공 목적이며 투자 권유가 아닙니다.
"""

        logger.info("Report generation completed")
        return report

if __name__ == "__main__":
    # 테스트
    analyzer = InterestRateAnalyzer()
    result = analyzer.run()

    print(f"\n=== Report saved to: {result['report_path']} ===")
```

### 4. BaseScenario 확인 및 업데이트

Epic 2에서 구현된 BaseScenario를 확인하고, 필요 시 `event_context` 파라미터 추가.

### 5. 의존성 추가

**requirements.txt**:
```
# 기존 의존성...
yfinance==0.2.38
```

### 6. 테스트

**수동 테스트**:
```bash
# 환경 변수 설정
export FRED_API_KEY=your_key

# 실행
python src/scenarios/interest_rate/interest_rate_analyzer.py

# 리포트 확인
cat reports/interest_rate_impact_$(date +%Y%m%d).md
```

**자동 트리거 테스트** (Story 3 통합 후):
```python
# tests/test_interest_rate_trigger.py
from agents.monitor.scenario_mapper import ScenarioMapper

event = {
    'event_id': 'fed_rate_change',
    'event_name': '연준 기준금리 변경',
    # ...
}

mapper = ScenarioMapper()
result = mapper.trigger_event(event)

assert result['scenarios'][0]['scenario_id'] == 'interest-rate-impact'
assert result['scenarios'][0]['status'] == 'success'
```

---

## Technical Notes

### FRED API
- 무료, API 키 필수
- 일일 요청 제한: 120회
- 다양한 경제 지표 제공

### 금융/부동산 상관관계
- 금리 ↓ → 금융주 NIM ↓ (단기 부정)
- 금리 ↓ → 대출 수요 ↑ (중장기 긍정)
- 금리 ↓ → REITs 매력 ↑ (채권 대비)

### 시나리오 프레임워크 활용
- BaseScenario 상속
- collect_data, analyze, generate_report 구현
- 재사용 가능한 구조

---

## Definition of Done

- [ ] FREDClient 구현 완료
- [ ] InterestRateAnalyzer 구현 완료
- [ ] BaseScenario 통합
- [ ] 수동 테스트 성공
- [ ] Story 3 자동 트리거 테스트 성공
- [ ] 리포트 품질 확인
- [ ] 문서 업데이트
- [ ] AC 모두 충족

---

## Dependencies

**Prerequisite**:
- Epic 2 (프레임워크) 완료
- Story 3 (시나리오 매핑) 완료

**Blocks**:
- 없음

---

## Estimated Time

- **FRED 클라이언트**: 1.5시간
- **InterestRateAnalyzer**: 3시간
- **테스트**: 1시간
- **통합 및 디버깅**: 1.5시간
- **Total**: 6 Story Points

---

## Risks & Mitigation

**Risk 1**: FRED API 장애
- **원인**: 서비스 다운, 요청 제한 초과
- **완화**: 캐싱, 대체 소스 (ECB, BOK)

**Risk 2**: 금리 데이터 지연
- **원인**: FRED 업데이트 주기 (일별)
- **완화**: 사용자에게 데이터 시점 명시

---

## Success Metrics

- ✅ 리포트 생성 시간 < 2분
- ✅ 데이터 수집 성공률 > 95%
- ✅ LLM 분석 품질 (사용자 만족도)

---

## References

- [FRED API 문서](https://fred.stlouisfed.org/docs/api/)
- [yfinance 문서](https://pypi.org/project/yfinance/)
- 실전 시나리오 #2: 금리 인하 대응
- Epic 2: 프레임워크
- Story 3: 시나리오 매핑
