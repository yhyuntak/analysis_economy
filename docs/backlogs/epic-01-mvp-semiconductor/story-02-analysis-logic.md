# User Story 2: 분석 로직 구현

## User Story

**As a** 반도체 섹터 투자자
**I want** 수집된 데이터를 의미 있는 지표로 변환하고 정리하는 분석 로직
**So that** LLM이 효과적으로 인사이트를 생성할 수 있는 구조화된 정보를 제공받을 수 있다

---

## Story Details

**Story ID**: EPIC-001-S2
**Epic**: Epic 1 - MVP 반도체 주간 점검 자동화
**Story Points**: 5
**Priority**: Must Have
**Assignee**: Developer (본인)
**Status**: 대기 중

---

## Acceptance Criteria

### AC1: 주가 데이터 분석
- [ ] ETF 및 개별 주식의 주간/월간 변동률 계산
- [ ] 섹터 전체 평균 변동률 계산 (가중 평균 또는 단순 평균)
- [ ] 상승/하락 종목 개수 집계
- [ ] 변동률 기준 상위 3개, 하위 3개 종목 식별

### AC2: 뉴스 데이터 분석
- [ ] 뉴스를 긍정/부정/중립으로 분류 (간단한 키워드 기반)
- [ ] 가장 많이 언급된 기업/키워드 추출 (빈도 분석)
- [ ] 뉴스를 날짜별로 그룹화
- [ ] 중요도 점수 계산 (출처 신뢰도, 언급 빈도 등)

### AC3: 데이터 정규화
- [ ] 모든 수치 데이터를 일관된 형식으로 변환 (소수점 2자리)
- [ ] 날짜 형식 통일 (YYYY-MM-DD)
- [ ] 통화 단위 명시 (USD, KRW)
- [ ] Missing data 처리 (기본값 또는 표시)

### AC4: LLM 입력용 데이터 구조화
- [ ] 분석 결과를 JSON 또는 dict 형태로 구조화
- [ ] LLM 프롬프트에 삽입하기 쉬운 텍스트 형식 변환 함수
- [ ] 데이터 요약 통계 생성 (평균, 중앙값, 표준편차 등)

### AC5: 성능 및 안정성
- [ ] 데이터 분석 시간 < 5초
- [ ] 빈 데이터 또는 불완전한 데이터에 대한 에러 핸들링
- [ ] 분석 결과 검증 로직 (이상치 탐지)

---

## Tasks

### Task 1: DataAnalyzer 클래스 설계
- [ ] `src/analyzers/` 디렉토리 생성
- [ ] `src/analyzers/data_analyzer.py` 파일 생성
- [ ] `DataAnalyzer` 클래스 기본 구조
  ```python
  class DataAnalyzer:
      def __init__(self, stock_data: dict, news_data: list[dict]):
          self.stock_data = stock_data
          self.news_data = news_data

      def analyze_stocks(self) -> dict:
          """주가 데이터 분석"""
          pass

      def analyze_news(self) -> dict:
          """뉴스 데이터 분석"""
          pass

      def generate_summary(self) -> dict:
          """전체 요약 생성"""
          pass
  ```

### Task 2: 주가 분석 로직 구현
- [ ] 변동률 계산 함수
  ```python
  def calculate_returns(self, prices: dict) -> dict:
      """
      Returns:
          {
              "weekly": {"SMH": 2.3, "NVDA": 4.1, ...},
              "monthly": {"SMH": 5.1, "NVDA": 9.2, ...}
          }
      """
  ```
- [ ] 섹터 평균 계산
  ```python
  def calculate_sector_average(self, returns: dict) -> float:
      """ETF와 주식의 가중 평균 또는 단순 평균"""
  ```
- [ ] Top/Bottom 종목 식별
  ```python
  def identify_outliers(self, returns: dict) -> dict:
      """
      Returns:
          {
              "top_performers": [("NVDA", 4.1), ("TSMC", 3.2), ...],
              "bottom_performers": [("삼성전자", -0.5), ...]
          }
      """
  ```
- [ ] 상승/하락 종목 카운트

### Task 3: 뉴스 분석 로직 구현
- [ ] 키워드 기반 감성 분류
  ```python
  def classify_sentiment(self, title: str) -> str:
      """
      긍정 키워드: "surge", "growth", "record", "강세", "성장"
      부정 키워드: "fall", "decline", "loss", "약세", "하락"
      """
      # 간단한 규칙 기반 (LLM 사용 X, 비용 절감)
  ```
- [ ] 기업 언급 추출
  ```python
  def extract_mentions(self, news_list: list[dict]) -> dict:
      """
      Returns:
          {"NVIDIA": 5, "TSMC": 3, "Intel": 2, ...}
      """
  ```
- [ ] 날짜별 그룹화
  ```python
  def group_by_date(self, news_list: list[dict]) -> dict:
      """
      Returns:
          {
              "2025-11-18": [news1, news2, ...],
              "2025-11-17": [news3, ...]
          }
      """
  ```
- [ ] 중요도 점수 계산 (간단한 휴리스틱)
  ```python
  def calculate_importance(self, news: dict) -> float:
      """
      출처 신뢰도 + 제목 길이 + 키워드 매칭
      Returns: 0-100 점수
      """
  ```

### Task 4: 데이터 정규화 유틸리티
- [ ] 수치 포맷팅
  ```python
  def format_percentage(self, value: float) -> str:
      """2.345 -> "+2.35%" """
      return f"{value:+.2f}%"

  def format_price(self, value: float, currency: str) -> str:
      """245.30 -> "$245.30" """
  ```
- [ ] 날짜 파싱 및 통일
  ```python
  from datetime import datetime

  def normalize_date(self, date_str: str) -> str:
      """다양한 형식 -> YYYY-MM-DD"""
  ```
- [ ] Missing data 처리
  ```python
  def handle_missing(self, value, default="N/A"):
      return value if value is not None else default
  ```

### Task 5: LLM 입력 데이터 구조화
- [ ] 분석 결과를 dict로 통합
  ```python
  def generate_summary(self) -> dict:
      """
      Returns:
          {
              "stocks": {
                  "sector_average_weekly": 2.1,
                  "sector_average_monthly": 5.3,
                  "top_performers": [...],
                  "bottom_performers": [...],
                  "up_count": 4,
                  "down_count": 1
              },
              "news": {
                  "total_count": 12,
                  "positive_count": 7,
                  "negative_count": 2,
                  "neutral_count": 3,
                  "top_mentions": {"NVIDIA": 5, "TSMC": 3},
                  "important_news": [...]  # 상위 5개
              },
              "metadata": {
                  "analysis_date": "2025-11-18",
                  "period": "2025-11-11 ~ 2025-11-18"
              }
          }
      """
  ```
- [ ] LLM 프롬프트용 텍스트 변환
  ```python
  def to_prompt_text(self, summary: dict) -> str:
      """
      dict를 읽기 쉬운 텍스트로 변환

      Example output:
      '''
      [주가 분석]
      - 섹터 평균 주간 변동: +2.1%
      - 섹터 평균 월간 변동: +5.3%
      - 상승 종목: 4개, 하락 종목: 1개

      [상위 종목]
      1. NVIDIA: +4.1%
      2. TSMC: +3.2%
      3. SMH ETF: +2.3%

      [뉴스 분석]
      - 전체 뉴스: 12건
      - 긍정: 7건, 부정: 2건, 중립: 3건
      - 가장 많이 언급된 기업: NVIDIA (5회), TSMC (3회)
      '''
      """
  ```

### Task 6: 단위 테스트 작성
- [ ] `tests/test_data_analyzer.py` 작성
- [ ] 주가 분석 로직 테스트
  ```python
  def test_calculate_returns():
      analyzer = DataAnalyzer(sample_stock_data, [])
      returns = analyzer.analyze_stocks()
      assert "weekly" in returns
      assert returns["weekly"]["SMH"] > 0  # 예상값 검증
  ```
- [ ] 뉴스 분석 로직 테스트
- [ ] 데이터 정규화 테스트
- [ ] 엣지 케이스 테스트 (빈 데이터, 이상치 등)

### Task 7: 통합 테스트
- [ ] `test_analysis.py` 스크립트 작성
  ```python
  # Story 1의 collector와 통합 테스트
  from collectors.stock_collector import StockCollector
  from collectors.news_collector import NewsCollector
  from analyzers.data_analyzer import DataAnalyzer

  stock_data = StockCollector([...]).collect_prices()
  news_data = NewsCollector([...]).collect_news()

  analyzer = DataAnalyzer(stock_data, news_data)
  summary = analyzer.generate_summary()
  print(summary)

  prompt_text = analyzer.to_prompt_text(summary)
  print(prompt_text)
  ```

---

## Technical Notes

### 감성 분류 키워드 예시
```python
POSITIVE_KEYWORDS = [
    "surge", "soar", "gain", "growth", "record", "breakthrough",
    "강세", "상승", "성장", "기록", "돌파", "호조"
]

NEGATIVE_KEYWORDS = [
    "fall", "decline", "drop", "loss", "slump", "concern",
    "약세", "하락", "감소", "손실", "우려", "부진"
]

def classify_sentiment(title: str) -> str:
    title_lower = title.lower()
    positive_count = sum(1 for kw in POSITIVE_KEYWORDS if kw in title_lower)
    negative_count = sum(1 for kw in NEGATIVE_KEYWORDS if kw in title_lower)

    if positive_count > negative_count:
        return "positive"
    elif negative_count > positive_count:
        return "negative"
    else:
        return "neutral"
```

### 기업 언급 추출 예시
```python
COMPANY_KEYWORDS = {
    "NVIDIA": ["nvidia", "엔비디아"],
    "TSMC": ["tsmc", "taiwan semiconductor"],
    "Intel": ["intel", "인텔"],
    "Samsung": ["samsung", "삼성전자", "삼성"],
    "ASML": ["asml"]
}

def extract_mentions(news_list: list[dict]) -> dict:
    mentions = {company: 0 for company in COMPANY_KEYWORDS.keys()}

    for news in news_list:
        title_lower = news['title'].lower()
        for company, keywords in COMPANY_KEYWORDS.items():
            if any(kw in title_lower for kw in keywords):
                mentions[company] += 1

    # 언급 없는 기업 제거
    return {k: v for k, v in mentions.items() if v > 0}
```

### 데이터 검증 로직
```python
def validate_summary(summary: dict) -> bool:
    """분석 결과의 무결성 검증"""
    required_keys = ["stocks", "news", "metadata"]
    if not all(k in summary for k in required_keys):
        return False

    # 변동률이 합리적인 범위인지 (-50% ~ +50%)
    sector_avg = summary["stocks"].get("sector_average_weekly", 0)
    if abs(sector_avg) > 50:
        print(f"Warning: Unusual sector average: {sector_avg}%")

    return True
```

---

## Definition of Done

- [ ] 모든 Acceptance Criteria 충족
- [ ] 모든 Tasks 완료
- [ ] 단위 테스트 작성 및 통과 (커버리지 > 80%)
- [ ] Story 1과 통합 테스트 성공
- [ ] 분석 결과가 올바른 형식으로 출력됨 (dict, text)
- [ ] 엣지 케이스 처리 검증 (빈 데이터, 이상치)
- [ ] 코드 리뷰 완료
- [ ] 함수/클래스에 docstring 추가

---

## Dependencies

**선행 작업**:
- Story 1: 반도체 섹터 데이터 수집 (수집된 데이터를 입력으로 사용)

**후속 작업**:
- Story 3: LLM 통합 (분석 결과를 프롬프트에 사용)

---

## Risks

| 리스크 | 완화 전략 |
|--------|-----------|
| 감성 분류 정확도 낮음 | 초기는 간단한 규칙 기반, 추후 LLM 사용 고려 |
| 한글/영문 혼재로 키워드 매칭 실패 | 양쪽 언어 키워드 모두 정의 |
| 이상치로 인한 분석 왜곡 | 변동률 범위 제한, 이상치 필터링 |
| Missing data 처리 미흡 | 모든 계산에 None 체크, 기본값 설정 |

---

## Estimated Time

**Story Points**: 5

**예상 시간**: 1-2일
- Task 1-2 (주가 분석): 4-6시간
- Task 3 (뉴스 분석): 4-6시간
- Task 4-7 (정규화, 구조화, 테스트): 3-5시간

---

## Notes

- 초기 감성 분류는 간단한 키워드 기반으로 시작 (LLM 비용 절감)
- 뉴스 중요도 점수는 간단한 휴리스틱 사용 (추후 정교화)
- 분석 결과는 메모리에만 유지 (DB 저장은 Epic 3 이후)
- LLM 프롬프트 텍스트 형식은 Story 3에서 최적화 가능

---

**Created**: 2025-11-18
**Last Updated**: 2025-11-18
**Status**: Blocked (Story 1 완료 대기)
