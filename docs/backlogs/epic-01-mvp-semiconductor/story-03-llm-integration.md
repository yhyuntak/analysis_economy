# User Story 3: LLM 통합 및 인사이트 생성

## User Story

**As a** 반도체 섹터 투자자
**I want** Claude LLM이 수집/분석된 데이터를 바탕으로 맥락 있는 인사이트를 생성하는 기능
**So that** 단순한 데이터 나열이 아닌, 투자 의사결정에 도움이 되는 해석과 시사점을 얻을 수 있다

---

## Story Details

**Story ID**: EPIC-001-S3
**Epic**: Epic 1 - MVP 반도체 주간 점검 자동화
**Story Points**: 8
**Priority**: Must Have
**Assignee**: Developer (본인)
**Status**: 대기 중

---

## Acceptance Criteria

### AC1: Claude API 통합
- [ ] Anthropic API 키 설정 및 환경변수 관리 (.env)
- [ ] `anthropic` Python SDK 설치 및 기본 연동
- [ ] API 호출 성공 및 응답 받기
- [ ] API 에러 핸들링 (rate limit, timeout, invalid key 등)

### AC2: 프롬프트 엔지니어링
- [ ] 시스템 프롬프트 작성 (역할, 톤, 제약사항)
- [ ] 데이터 삽입 프롬프트 템플릿 설계
- [ ] Few-shot examples 포함 (일관된 출력 형식 유도)
- [ ] 출력 형식 지시 (섹션별 구조화)

### AC3: 인사이트 생성 품질
- [ ] 생성된 인사이트가 다음을 포함:
  - 주간 섹터 동향 요약 (3-5문장)
  - 주요 이슈 해석 (뉴스 기반)
  - 투자 시사점 (기회 또는 리스크)
  - 향후 주목할 포인트
- [ ] 환각(hallucination) 최소화: 제공된 데이터 범위 내에서만 분석
- [ ] 한국어 자연스러움 (문법, 어휘, 톤)

### AC4: 성능 및 비용 최적화
- [ ] API 호출 시간 < 10초
- [ ] 입력 토큰 < 2000 tokens (비용 절감)
- [ ] 출력 토큰 < 1000 tokens
- [ ] 토큰 사용량 로깅 (비용 추적)

### AC5: 재시도 및 안정성
- [ ] API 호출 실패 시 자동 재시도 (최대 3회)
- [ ] Rate limit 초과 시 대기 후 재시도
- [ ] Fallback: LLM 실패 시에도 기본 리포트 생성 가능

---

## Tasks

### Task 1: Anthropic API 설정
- [ ] `anthropic` 패키지 설치
  ```bash
  pip install anthropic
  ```
- [ ] `.env` 파일 생성 (gitignore에 추가)
  ```
  ANTHROPIC_API_KEY=sk-ant-api03-...
  ```
- [ ] `src/utils/config.py`에 API 키 로드 로직
  ```python
  import os
  from dotenv import load_dotenv

  load_dotenv()

  ANTHROPIC_API_KEY = os.getenv("ANTHROPIC_API_KEY")
  if not ANTHROPIC_API_KEY:
      raise ValueError("ANTHROPIC_API_KEY not found in .env")
  ```

### Task 2: LLMAnalyzer 클래스 구현
- [ ] `src/analyzers/llm_analyzer.py` 파일 생성
- [ ] `LLMAnalyzer` 클래스 기본 구조
  ```python
  import anthropic
  from typing import Dict

  class LLMAnalyzer:
      def __init__(self, api_key: str):
          self.client = anthropic.Anthropic(api_key=api_key)
          self.model = "claude-3-5-sonnet-20241022"

      def generate_insights(self, analysis_data: dict) -> dict:
          """
          Args:
              analysis_data: Story 2에서 생성한 분석 결과

          Returns:
              {
                  "summary": "3줄 요약",
                  "sector_analysis": "섹터 동향 해석",
                  "key_news": "주요 뉴스 해석",
                  "investment_implications": "투자 시사점"
              }
          """
          pass

      def _build_prompt(self, analysis_data: dict) -> str:
          """분석 데이터를 프롬프트로 변환"""
          pass

      def _parse_response(self, response_text: str) -> dict:
          """LLM 응답을 구조화된 dict로 파싱"""
          pass
  ```

### Task 3: 시스템 프롬프트 작성
- [ ] 역할 정의
  ```python
  SYSTEM_PROMPT = """
  당신은 글로벌 반도체 섹터를 전문으로 분석하는 베테랑 투자 애널리스트입니다.

  역할:
  - 제공된 주가 및 뉴스 데이터를 바탕으로 반도체 섹터의 주간 동향을 분석합니다.
  - 투자자가 의사결정에 활용할 수 있는 실용적인 인사이트를 제공합니다.
  - 데이터에 기반하여 객관적으로 분석하되, 맥락과 해석을 더해 가치를 창출합니다.

  제약사항:
  - 제공된 데이터 범위를 벗어난 추측이나 환각을 피하세요.
  - 투자 권유가 아닌, 정보 제공과 해석에 집중하세요.
  - 한국어로 자연스럽고 전문적인 어투로 작성하세요.
  - 단정적인 예측보다는 가능성과 시나리오를 제시하세요.
  """
  ```

### Task 4: 프롬프트 템플릿 설계
- [ ] 데이터 삽입 템플릿
  ```python
  def _build_prompt(self, analysis_data: dict) -> str:
      stocks = analysis_data["stocks"]
      news = analysis_data["news"]
      metadata = analysis_data["metadata"]

      prompt = f"""
  다음은 {metadata['period']} 기간 동안의 반도체 섹터 데이터입니다.

  ## 주가 데이터
  - 섹터 평균 주간 변동: {stocks['sector_average_weekly']}%
  - 섹터 평균 월간 변동: {stocks['sector_average_monthly']}%
  - 상승 종목: {stocks['up_count']}개
  - 하락 종목: {stocks['down_count']}개

  ### 상위 종목
  {self._format_top_performers(stocks['top_performers'])}

  ### 하위 종목
  {self._format_bottom_performers(stocks['bottom_performers'])}

  ## 뉴스 분석
  - 전체 뉴스: {news['total_count']}건
  - 긍정 뉴스: {news['positive_count']}건
  - 부정 뉴스: {news['negative_count']}건
  - 가장 많이 언급된 기업: {', '.join([f"{k}({v}회)" for k, v in list(news['top_mentions'].items())[:3]])}

  ### 주요 뉴스 (상위 5개)
  {self._format_important_news(news['important_news'])}

  ---

  위 데이터를 바탕으로 다음 항목을 작성해주세요:

  1. **요약** (3-5문장): 이번 주 반도체 섹터의 핵심 동향을 간결하게 정리
  2. **섹터 동향**: 주가 움직임의 의미와 배경 해석
  3. **주요 뉴스 해석**: 언급된 뉴스들이 섹터에 미치는 영향
  4. **투자 시사점**: 투자자가 주목해야 할 기회와 리스크

  각 섹션을 명확히 구분하여 작성해주세요.
  """
      return prompt
  ```

### Task 5: Few-shot Examples 추가 (Optional)
- [ ] 일관된 출력 형식을 위한 예시 추가
  ```python
  FEW_SHOT_EXAMPLE = """
  예시:

  1. 요약
  이번 주 반도체 섹터는 NVIDIA의 실적 발표와 AI 칩 수요 증가 소식에 힘입어 평균 +2.1% 상승했습니다.
  다만 삼성전자는 메모리 가격 하락 우려로 소폭 하락했습니다.
  전반적으로 AI 관련 기업들의 강세가 두드러진 한 주였습니다.

  2. 섹터 동향
  SMH ETF와 SOXX ETF 모두 2% 이상 상승하며...

  [이하 생략]
  """
  ```

### Task 6: API 호출 및 응답 처리
- [ ] Claude API 호출 로직
  ```python
  def generate_insights(self, analysis_data: dict) -> dict:
      prompt = self._build_prompt(analysis_data)

      try:
          message = self.client.messages.create(
              model=self.model,
              max_tokens=1024,
              temperature=0.3,  # 일관성 우선
              system=SYSTEM_PROMPT,
              messages=[
                  {"role": "user", "content": prompt}
              ]
          )

          response_text = message.content[0].text

          # 토큰 사용량 로깅
          self._log_token_usage(message.usage)

          # 응답 파싱
          insights = self._parse_response(response_text)

          return insights

      except anthropic.RateLimitError as e:
          # Rate limit 시 대기 후 재시도
          time.sleep(60)
          return self.generate_insights(analysis_data)

      except Exception as e:
          # 기타 에러 처리
          logger.error(f"LLM API error: {e}")
          return self._generate_fallback_insights(analysis_data)
  ```

### Task 7: 응답 파싱 로직
- [ ] LLM 응답을 구조화된 dict로 변환
  ```python
  def _parse_response(self, response_text: str) -> dict:
      """
      LLM이 생성한 텍스트를 섹션별로 분리

      Example response_text:
      '''
      1. 요약
      [내용]

      2. 섹터 동향
      [내용]

      3. 주요 뉴스 해석
      [내용]

      4. 투자 시사점
      [내용]
      '''
      """
      sections = {
          "summary": "",
          "sector_analysis": "",
          "key_news": "",
          "investment_implications": ""
      }

      # 정규식 또는 split으로 섹션 분리
      # 간단한 구현: 숫자 헤딩으로 split
      import re
      parts = re.split(r'\d+\.\s*\*?\*?', response_text)

      if len(parts) >= 5:
          sections["summary"] = parts[1].strip()
          sections["sector_analysis"] = parts[2].strip()
          sections["key_news"] = parts[3].strip()
          sections["investment_implications"] = parts[4].strip()

      return sections
  ```

### Task 8: Fallback 인사이트 생성
- [ ] LLM 실패 시 기본 인사이트 생성
  ```python
  def _generate_fallback_insights(self, analysis_data: dict) -> dict:
      """LLM API 실패 시 간단한 템플릿 기반 인사이트"""
      stocks = analysis_data["stocks"]

      return {
          "summary": f"반도체 섹터는 주간 {stocks['sector_average_weekly']:+.2f}% 변동했습니다.",
          "sector_analysis": "LLM 분석을 사용할 수 없습니다. 주가 데이터를 참고하세요.",
          "key_news": "뉴스 분석을 사용할 수 없습니다.",
          "investment_implications": "추가 분석이 필요합니다."
      }
  ```

### Task 9: 토큰 사용량 로깅
- [ ] 비용 추적을 위한 로깅
  ```python
  def _log_token_usage(self, usage):
      """
      usage.input_tokens, usage.output_tokens 로깅
      """
      logger.info(f"Token usage - Input: {usage.input_tokens}, Output: {usage.output_tokens}")

      # Optional: 비용 계산
      # Claude 3.5 Sonnet 가격 (2024년 기준 예시)
      input_cost = usage.input_tokens * 0.003 / 1000  # $0.003 per 1K tokens
      output_cost = usage.output_tokens * 0.015 / 1000  # $0.015 per 1K tokens
      total_cost = input_cost + output_cost

      logger.info(f"Estimated cost: ${total_cost:.4f}")
  ```

### Task 10: 통합 테스트
- [ ] `test_llm_integration.py` 작성
  ```python
  from analyzers.data_analyzer import DataAnalyzer
  from analyzers.llm_analyzer import LLMAnalyzer
  from collectors.stock_collector import StockCollector
  from collectors.news_collector import NewsCollector

  # 전체 파이프라인 테스트
  stock_data = StockCollector([...]).collect_prices()
  news_data = NewsCollector([...]).collect_news()

  analyzer = DataAnalyzer(stock_data, news_data)
  summary = analyzer.generate_summary()

  llm = LLMAnalyzer(ANTHROPIC_API_KEY)
  insights = llm.generate_insights(summary)

  print("=== LLM Insights ===")
  print(f"Summary: {insights['summary']}")
  print(f"Sector Analysis: {insights['sector_analysis']}")
  print(f"Key News: {insights['key_news']}")
  print(f"Investment Implications: {insights['investment_implications']}")
  ```

### Task 11: 프롬프트 최적화 및 A/B 테스트
- [ ] 여러 프롬프트 버전 테스트
- [ ] 온도(temperature) 파라미터 조정 (0.1 ~ 0.5)
- [ ] Few-shot examples 효과 검증
- [ ] 출력 품질 평가 (주관적 평가 또는 체크리스트)

---

## Technical Notes

### Anthropic API 기본 사용법
```python
import anthropic

client = anthropic.Anthropic(api_key="sk-ant-...")

message = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    temperature=0.3,
    system="You are a financial analyst...",
    messages=[
        {"role": "user", "content": "Analyze this data..."}
    ]
)

print(message.content[0].text)
print(f"Tokens used: {message.usage.input_tokens} in, {message.usage.output_tokens} out")
```

### 프롬프트 최적화 팁
1. **명확한 지시**: "다음 형식으로 작성하세요: 1. 요약 2. 분석..."
2. **제약사항 명시**: "제공된 데이터 범위 내에서만 분석하세요"
3. **톤 설정**: "전문적이지만 이해하기 쉬운 한국어로"
4. **Few-shot**: 원하는 출력 형식의 예시 1-2개 포함
5. **Temperature**: 일관성 중요 시 0.1-0.3, 창의성 필요 시 0.5-0.7

### 에러 핸들링 패턴
```python
import time
from anthropic import RateLimitError, APIError

def call_with_retry(func, max_retries=3):
    for attempt in range(max_retries):
        try:
            return func()
        except RateLimitError:
            wait_time = 2 ** attempt  # Exponential backoff
            logger.warning(f"Rate limit hit. Waiting {wait_time}s...")
            time.sleep(wait_time)
        except APIError as e:
            logger.error(f"API error: {e}")
            if attempt == max_retries - 1:
                raise
    return None
```

### 비용 추정
- Claude 3.5 Sonnet (2024년 기준 예시):
  - Input: $0.003 per 1K tokens
  - Output: $0.015 per 1K tokens
- 예상 사용량 (1회 실행):
  - Input: ~1500 tokens → $0.0045
  - Output: ~800 tokens → $0.012
  - Total: ~$0.017 per report
- 월간 비용 (주 1회 실행): ~$0.07

---

## Definition of Done

- [ ] 모든 Acceptance Criteria 충족
- [ ] 모든 Tasks 완료
- [ ] Claude API 연동 성공 및 안정적 응답
- [ ] 생성된 인사이트 품질 검증 (3회 이상 테스트)
- [ ] 프롬프트 최적화 완료
- [ ] 에러 핸들링 및 재시도 로직 동작 확인
- [ ] 토큰 사용량 로깅 확인
- [ ] 전체 파이프라인 (Story 1-3) 통합 테스트 성공
- [ ] 코드 리뷰 및 문서화

---

## Dependencies

**선행 작업**:
- Story 1: 데이터 수집
- Story 2: 분석 로직 (LLM 입력 데이터 제공)

**후속 작업**:
- Story 4: 리포트 생성 (LLM 인사이트를 리포트에 포함)

---

## Risks

| 리스크 | 완화 전략 |
|--------|-----------|
| LLM 환각 (Hallucination) | 데이터 기반 분석 강조, 프롬프트에 제약사항 명시 |
| API 비용 초과 | 토큰 사용량 로깅, 월 예산 설정, 입력 데이터 최소화 |
| 프롬프트 품질 낮음 | A/B 테스트, 반복 개선, Few-shot examples |
| Rate Limit 초과 | Exponential backoff, 재시도 로직 |
| API 장애 | Fallback 인사이트 생성 |

---

## Estimated Time

**Story Points**: 8

**예상 시간**: 2-3일
- Task 1-2 (API 설정, 클래스 구현): 2-4시간
- Task 3-5 (프롬프트 엔지니어링): 6-8시간
- Task 6-9 (API 호출, 파싱, 에러 핸들링): 4-6시간
- Task 10-11 (통합 테스트, 최적화): 4-6시간

---

## Notes

- 프롬프트는 반복적으로 개선 (처음부터 완벽할 수 없음)
- 초기 몇 번의 실행 결과를 보고 프롬프트 조정
- Few-shot examples는 선택사항 (없어도 충분히 동작 가능)
- 비용 모니터링 중요: 예상 외 과금 방지
- LLM 응답 형식이 일정하지 않을 수 있음 → 파싱 로직 견고하게

---

**Created**: 2025-11-18
**Last Updated**: 2025-11-18
**Status**: Blocked (Story 1-2 완료 대기)
