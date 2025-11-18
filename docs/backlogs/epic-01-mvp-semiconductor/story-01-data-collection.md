# User Story 1: 반도체 섹터 데이터 수집

## User Story

**As a** 반도체 섹터 투자자
**I want** ETF, 주요 기업 주가, 관련 뉴스 데이터를 자동으로 수집하는 기능
**So that** 매주 수동으로 여러 사이트를 방문하지 않고도 필요한 모든 데이터를 한 번에 확보할 수 있다

---

## Story Details

**Story ID**: EPIC-001-S1
**Epic**: Epic 1 - MVP 반도체 주간 점검 자동화
**Story Points**: 5
**Priority**: Must Have
**Assignee**: Developer (본인)
**Status**: 준비 완료

---

## Acceptance Criteria

### AC1: ETF 가격 데이터 수집
- [ ] SMH, SOXX ETF의 최근 1주일 일별 종가 데이터 수집
- [ ] 각 ETF의 주간 변동률, 월간 변동률 계산
- [ ] 데이터는 Python dict 또는 pandas DataFrame으로 반환

### AC2: 주요 기업 주가 데이터 수집
- [ ] 다음 기업들의 최근 1주일 일별 종가 수집:
  - 삼성전자 (005930.KS)
  - TSMC (TSM)
  - NVIDIA (NVDA)
  - Intel (INTC)
  - ASML (ASML)
- [ ] 각 종목의 주간 변동률, 월간 변동률 계산

### AC3: 뉴스 데이터 수집
- [ ] "semiconductor", "chip", "반도체" 키워드로 최근 7일 뉴스 검색
- [ ] 최소 10건 이상의 뉴스 수집
- [ ] 각 뉴스는 제목, 출처, 날짜, URL 포함
- [ ] 중복 뉴스 제거

### AC4: 에러 핸들링
- [ ] API 호출 실패 시 명확한 에러 메시지 출력
- [ ] 네트워크 오류 시 재시도 로직 (최대 3회)
- [ ] 일부 데이터 수집 실패 시에도 다른 데이터는 정상 수집

### AC5: 성능 요구사항
- [ ] 전체 데이터 수집 시간 < 30초
- [ ] API Rate Limit 준수 (요청 간 딜레이 추가)

---

## Tasks

### Task 1: 프로젝트 구조 및 환경 설정
- [ ] Python 가상환경 생성 (`python -m venv venv`)
- [ ] 필요한 패키지 설치 (`requirements.txt` 작성)
  ```txt
  requests==2.31.0
  beautifulsoup4==4.12.2
  yfinance==0.2.28
  python-dotenv==1.0.0
  ```
- [ ] 프로젝트 디렉토리 구조 생성
  ```
  src/
  ├── collectors/
  │   ├── __init__.py
  │   ├── stock_collector.py
  │   └── news_collector.py
  └── utils/
      ├── __init__.py
      ├── config.py
      └── logger.py
  ```

### Task 2: StockCollector 구현
- [ ] `src/collectors/stock_collector.py` 파일 생성
- [ ] `StockCollector` 클래스 구현
  ```python
  class StockCollector:
      def __init__(self, symbols: list[str]):
          self.symbols = symbols

      def collect_prices(self, period: str = "1mo") -> dict:
          """주가 데이터 수집"""
          pass

      def calculate_returns(self, prices: dict) -> dict:
          """변동률 계산"""
          pass
  ```
- [ ] `yfinance` 라이브러리를 사용하여 ETF 및 주식 데이터 수집
- [ ] 주간/월간 변동률 계산 로직 구현
- [ ] 에러 핸들링 (심볼이 없거나 데이터 없을 경우)

### Task 3: NewsCollector 구현
- [ ] `src/collectors/news_collector.py` 파일 생성
- [ ] `NewsCollector` 클래스 구현
  ```python
  class NewsCollector:
      def __init__(self, keywords: list[str]):
          self.keywords = keywords

      def collect_news(self, days: int = 7) -> list[dict]:
          """뉴스 수집"""
          pass

      def deduplicate(self, news: list[dict]) -> list[dict]:
          """중복 제거"""
          pass
  ```
- [ ] Google News RSS 또는 간단한 웹 스크래핑 구현
  - Option A: Google News RSS (`https://news.google.com/rss/search?q={keyword}`)
  - Option B: BeautifulSoup으로 Google News 페이지 파싱
- [ ] 날짜 필터링 (최근 7일)
- [ ] 중복 뉴스 제거 (제목 유사도 기반)

### Task 4: 설정 및 유틸리티
- [ ] `src/utils/config.py` 작성
  ```python
  # 환경변수 및 설정 관리
  STOCK_SYMBOLS = {
      "etfs": ["SMH", "SOXX"],
      "stocks": ["005930.KS", "TSM", "NVDA", "INTC", "ASML"]
  }
  NEWS_KEYWORDS = ["semiconductor", "chip", "반도체"]
  ```
- [ ] `src/utils/logger.py` 작성 (기본 로깅 설정)

### Task 5: 통합 테스트
- [ ] 테스트 스크립트 작성 (`tests/test_collectors.py`)
- [ ] StockCollector 단위 테스트
  ```python
  def test_stock_collector():
      collector = StockCollector(["SMH", "NVDA"])
      data = collector.collect_prices()
      assert "SMH" in data
      assert "NVDA" in data
  ```
- [ ] NewsCollector 단위 테스트
- [ ] 전체 데이터 수집 통합 테스트 (실제 API 호출)

### Task 6: CLI 테스트 스크립트
- [ ] `test_collection.py` 작성 (수동 실행용)
  ```python
  # 데이터 수집 테스트 및 결과 출력
  if __name__ == "__main__":
      stock_collector = StockCollector(STOCK_SYMBOLS["etfs"] + STOCK_SYMBOLS["stocks"])
      stock_data = stock_collector.collect_prices()
      print("Stock Data:", stock_data)

      news_collector = NewsCollector(NEWS_KEYWORDS)
      news_data = news_collector.collect_news(days=7)
      print(f"Collected {len(news_data)} news articles")
  ```

---

## Technical Notes

### yfinance 사용 예시
```python
import yfinance as yf

# ETF 데이터 수집
ticker = yf.Ticker("SMH")
hist = ticker.history(period="1mo")  # 최근 1개월

# 주간 변동률 계산
current_price = hist['Close'][-1]
week_ago_price = hist['Close'][-5]  # 5 trading days ago
weekly_return = ((current_price - week_ago_price) / week_ago_price) * 100
```

### Google News RSS 사용 예시
```python
import requests
from bs4 import BeautifulSoup
from datetime import datetime, timedelta

def fetch_google_news(keyword, days=7):
    url = f"https://news.google.com/rss/search?q={keyword}&hl=ko&gl=KR&ceid=KR:ko"
    response = requests.get(url)
    soup = BeautifulSoup(response.content, 'xml')

    articles = []
    cutoff_date = datetime.now() - timedelta(days=days)

    for item in soup.find_all('item'):
        title = item.title.text
        link = item.link.text
        pub_date = datetime.strptime(item.pubDate.text, '%a, %d %b %Y %H:%M:%S %Z')

        if pub_date >= cutoff_date:
            articles.append({
                'title': title,
                'url': link,
                'date': pub_date.strftime('%Y-%m-%d'),
                'source': 'Google News'
            })

    return articles
```

### Rate Limiting 고려사항
- yfinance: 초당 1-2개 요청 권장
- Google News: 크롤링 간격 2-3초 권장
- 요청 간 `time.sleep(2)` 추가

### 중복 제거 로직
```python
def deduplicate_news(news_list):
    seen_titles = set()
    unique_news = []

    for news in news_list:
        # 제목 정규화 (소문자, 공백 제거)
        normalized = news['title'].lower().strip()
        if normalized not in seen_titles:
            seen_titles.add(normalized)
            unique_news.append(news)

    return unique_news
```

---

## Definition of Done

Story가 완료되었다고 판단하는 체크리스트:

- [ ] 모든 Acceptance Criteria 충족
- [ ] 모든 Tasks 완료
- [ ] 단위 테스트 작성 및 통과
- [ ] 통합 테스트 (실제 API 호출) 성공
- [ ] 에러 핸들링 검증 (네트워크 오류, 잘못된 심볼 등)
- [ ] 코드 리뷰 완료 (본인 검토)
- [ ] 간단한 문서화 (주석, docstring)

---

## Dependencies

**선행 작업**: 없음 (Epic 1의 첫 Story)

**후속 작업**:
- Story 2: 분석 로직 구현 (수집된 데이터를 입력으로 사용)

---

## Risks

| 리스크 | 완화 전략 |
|--------|-----------|
| yfinance API 변경 | 공식 문서 참조, 에러 발생 시 빠른 대응 |
| Google News 크롤링 차단 | User-Agent 설정, 요청 간격 늘림, RSS 사용 우선 |
| 한국 주식 (삼성전자) 데이터 부족 | Yahoo Finance Korea 심볼 확인 (005930.KS) |
| API Rate Limit 초과 | 요청 간 딜레이, 캐싱 고려 |

---

## Estimated Time

**Story Points**: 5

**예상 시간**: 1-2일
- Task 1-2 (StockCollector): 4-6시간
- Task 3 (NewsCollector): 4-6시간
- Task 4-6 (유틸리티, 테스트): 2-4시간

---

## Notes

- 초기 구현은 최소한의 기능만 포함 (Over-engineering 주의)
- 캐싱은 Epic 2 (프레임워크 구축)에서 고려
- 데이터 저장은 현재 메모리 내에서만 (DB는 향후)
- 삼성전자 데이터가 yfinance에서 잘 수집되는지 초기 확인 필요

---

**Created**: 2025-11-18
**Last Updated**: 2025-11-18
**Status**: Ready for Development
