# User Story 4: 리포트 생성 및 출력

## User Story

**As a** 반도체 섹터 투자자
**I want** 수집/분석/인사이트 데이터를 읽기 쉬운 Markdown 리포트로 생성하는 기능
**So that** 명령어 하나로 완성된 주간 점검 리포트를 받아 투자 의사결정에 활용할 수 있다

---

## Story Details

**Story ID**: EPIC-001-S4
**Epic**: Epic 1 - MVP 반도체 주간 점검 자동화
**Story Points**: 3
**Priority**: Must Have
**Assignee**: Developer (본인)
**Status**: 대기 중

---

## Acceptance Criteria

### AC1: Markdown 리포트 생성
- [ ] Story 1-3의 결과를 통합하여 완전한 Markdown 파일 생성
- [ ] 리포트 파일명: `semiconductor_weekly_YYYYMMDD.md`
- [ ] 파일 저장 위치: `reports/` 디렉토리
- [ ] 리포트 내용:
  - 헤더 (제목, 생성일시, 분석기간, 데이터 출처)
  - 요약 (LLM 생성)
  - 주요 지표 테이블
  - 섹터 동향 (LLM 생성)
  - 주요 뉴스
  - 투자 시사점 (LLM 생성)
  - 푸터 (면책 조항)

### AC2: CLI 실행 인터페이스
- [ ] `run_scenario.py` 스크립트 작성
- [ ] 명령어: `python run_scenario.py --scenario semiconductor-weekly`
- [ ] 진행 상황 표시 (데이터 수집 → 분석 → LLM → 리포트 생성)
- [ ] 완료 시 리포트 파일 경로 출력
- [ ] 소요 시간 표시

### AC3: 출력 형식 및 가독성
- [ ] GitHub Flavored Markdown 준수
- [ ] 테이블, 리스트, 강조 적절히 사용
- [ ] 한국어 자연스러움
- [ ] 단락 구분 명확
- [ ] 코드 블록 없음 (데이터 시각화는 텍스트/테이블로)

### AC4: 에러 핸들링 및 사용자 경험
- [ ] 파이프라인 중간 실패 시 명확한 에러 메시지
- [ ] 부분 성공 시에도 가능한 리포트 생성
- [ ] 로그 파일 생성 (디버깅용)
- [ ] 진행 상황을 시각적으로 표시 (progress bar 또는 단계별 체크)

### AC5: 성능 요구사항
- [ ] 전체 파이프라인 실행 시간 < 2분
- [ ] 리포트 생성 자체는 < 1초

---

## Tasks

### Task 1: MarkdownReporter 클래스 구현
- [ ] `src/reporters/` 디렉토리 생성
- [ ] `src/reporters/markdown_reporter.py` 파일 생성
- [ ] `MarkdownReporter` 클래스 기본 구조
  ```python
  from datetime import datetime
  from typing import Dict

  class MarkdownReporter:
      def __init__(self, output_dir: str = "reports"):
          self.output_dir = output_dir

      def generate_report(
          self,
          stock_data: dict,
          news_data: list,
          analysis_summary: dict,
          llm_insights: dict
      ) -> str:
          """
          전체 데이터를 통합하여 Markdown 리포트 생성

          Returns:
              생성된 파일 경로
          """
          pass

      def _build_header(self, metadata: dict) -> str:
          """헤더 섹션 생성"""
          pass

      def _build_summary(self, llm_insights: dict) -> str:
          """요약 섹션 (LLM)"""
          pass

      def _build_metrics_table(self, analysis_summary: dict) -> str:
          """주요 지표 테이블"""
          pass

      def _build_sector_analysis(self, llm_insights: dict) -> str:
          """섹터 동향 (LLM)"""
          pass

      def _build_news_section(self, news_data: list, analysis_summary: dict) -> str:
          """주요 뉴스 섹션"""
          pass

      def _build_investment_implications(self, llm_insights: dict) -> str:
          """투자 시사점 (LLM)"""
          pass

      def _build_footer(self) -> str:
          """푸터 (면책 조항)"""
          pass
  ```

### Task 2: 리포트 템플릿 작성
- [ ] 헤더 템플릿
  ```python
  def _build_header(self, metadata: dict) -> str:
      return f"""# 반도체 섹터 주간 점검

  **생성 일시**: {metadata['analysis_date']}
  **분석 기간**: {metadata['period']}
  **데이터 소스**: Yahoo Finance, Google News

  ---
  """
  ```

- [ ] 주요 지표 테이블 템플릿
  ```python
  def _build_metrics_table(self, analysis_summary: dict) -> str:
      stocks = analysis_summary["stocks"]

      # 테이블 헤더
      table = "## 주요 지표\n\n"
      table += "| 지표 | 현재가 | 주간 변동 | 월간 변동 |\n"
      table += "|------|---------|-----------|-----------|\\n"

      # ETF 데이터
      for etf, data in stocks["etf_data"].items():
          table += f"| {etf} | ${data['price']:.2f} | {data['weekly_return']:+.2f}% | {data['monthly_return']:+.2f}% |\n"

      # 개별 주식 데이터
      for stock, data in stocks["stock_data"].items():
          table += f"| {stock} | {data['price']} | {data['weekly_return']:+.2f}% | {data['monthly_return']:+.2f}% |\n"

      return table
  ```

- [ ] 뉴스 섹션 템플릿
  ```python
  def _build_news_section(self, news_data: list, analysis_summary: dict) -> str:
      news = analysis_summary["news"]

      section = "## 주요 뉴스\n\n"
      section += f"**전체 {news['total_count']}건** (긍정 {news['positive_count']}, 부정 {news['negative_count']}, 중립 {news['neutral_count']})\n\n"

      # 중요 뉴스 상위 5개
      for i, article in enumerate(news["important_news"][:5], 1):
          section += f"{i}. **[{article['title']}]({article['url']})** ({article['source']}, {article['date']})\n"
          if "summary" in article:
              section += f"   - {article['summary']}\n"
          section += "\n"

      return section
  ```

- [ ] 푸터 템플릿
  ```python
  def _build_footer(self) -> str:
      return """
  ---

  ## 참고사항

  - 본 리포트는 AI(Claude)가 생성한 분석이며, 투자 권유가 아닙니다.
  - 실제 투자 결정은 본인의 판단과 책임하에 이루어져야 합니다.
  - 데이터는 공개 소스에서 수집되었으며, 정확성을 보장하지 않습니다.
  """
  ```

### Task 3: CLI 스크립트 구현
- [ ] 프로젝트 루트에 `run_scenario.py` 생성
- [ ] argparse로 CLI 인터페이스 구현
  ```python
  import argparse
  import sys
  from datetime import datetime
  from src.scenarios.semiconductor_weekly import SemiconductorWeeklyScenario

  def main():
      parser = argparse.ArgumentParser(description="산업 섹터 분석 자동화")
      parser.add_argument(
          "--scenario",
          type=str,
          required=True,
          choices=["semiconductor-weekly"],
          help="실행할 시나리오"
      )
      args = parser.parse_args()

      if args.scenario == "semiconductor-weekly":
          scenario = SemiconductorWeeklyScenario()
          scenario.run()
      else:
          print(f"Unknown scenario: {args.scenario}")
          sys.exit(1)

  if __name__ == "__main__":
      main()
  ```

### Task 4: Scenario 통합 클래스 구현
- [ ] `src/scenarios/` 디렉토리 생성
- [ ] `src/scenarios/semiconductor_weekly.py` 파일 생성
- [ ] Story 1-3의 컴포넌트를 통합하는 Scenario 클래스
  ```python
  import time
  from datetime import datetime, timedelta
  from src.collectors.stock_collector import StockCollector
  from src.collectors.news_collector import NewsCollector
  from src.analyzers.data_analyzer import DataAnalyzer
  from src.analyzers.llm_analyzer import LLMAnalyzer
  from src.reporters.markdown_reporter import MarkdownReporter
  from src.utils.config import ANTHROPIC_API_KEY, STOCK_SYMBOLS, NEWS_KEYWORDS

  class SemiconductorWeeklyScenario:
      def __init__(self):
          self.start_time = None

      def run(self):
          """전체 파이프라인 실행"""
          self.start_time = time.time()

          print("=" * 60)
          print("반도체 섹터 주간 점검 시작")
          print("=" * 60)
          print()

          try:
              # Step 1: 데이터 수집
              stock_data, news_data = self._collect_data()

              # Step 2: 데이터 분석
              analysis_summary = self._analyze_data(stock_data, news_data)

              # Step 3: LLM 인사이트 생성
              llm_insights = self._generate_insights(analysis_summary)

              # Step 4: 리포트 생성
              report_path = self._generate_report(
                  stock_data, news_data, analysis_summary, llm_insights
              )

              # 완료 메시지
              self._print_completion(report_path)

          except Exception as e:
              print(f"\\n❌ 오류 발생: {e}")
              import traceback
              traceback.print_exc()
              sys.exit(1)

      def _collect_data(self):
          """Step 1: 데이터 수집"""
          print("[1/4] 데이터 수집 중...")
          # 구현
          pass

      def _analyze_data(self, stock_data, news_data):
          """Step 2: 데이터 분석"""
          print("[2/4] 데이터 분석 중...")
          # 구현
          pass

      def _generate_insights(self, analysis_summary):
          """Step 3: LLM 인사이트 생성"""
          print("[3/4] LLM 인사이트 생성 중...")
          # 구현
          pass

      def _generate_report(self, stock_data, news_data, analysis_summary, llm_insights):
          """Step 4: 리포트 생성"""
          print("[4/4] 리포트 생성 중...")
          # 구현
          pass

      def _print_completion(self, report_path):
          """완료 메시지 출력"""
          elapsed_time = time.time() - self.start_time
          minutes = int(elapsed_time // 60)
          seconds = int(elapsed_time % 60)

          print()
          print("=" * 60)
          print("리포트 생성 완료")
          print("=" * 60)
          print()
          print(f"📄 경로: {report_path}")
          print(f"⏱️  소요 시간: {minutes}분 {seconds}초")
          print()
          print("다음 명령어로 리포트 확인:")
          print(f"  cat {report_path}")
          print()
  ```

### Task 5: 진행 상황 표시 개선
- [ ] 각 단계별 세부 진행 상황 표시
  ```python
  def _collect_data(self):
      print("[1/4] 데이터 수집 중...")

      # ETF 데이터
      print("  - ETF 가격 데이터 수집 중...", end="", flush=True)
      etf_data = stock_collector.collect_etfs()
      print(" ✓")

      # 주식 데이터
      print("  - 주요 기업 주가 수집 중...", end="", flush=True)
      stock_data = stock_collector.collect_stocks()
      print(" ✓")

      # 뉴스 데이터
      print("  - 뉴스 수집 중...", end="", flush=True)
      news_data = news_collector.collect_news()
      print(f" ✓ ({len(news_data)}건)")

      print()
      return stock_data, news_data
  ```

### Task 6: 에러 핸들링 및 부분 실패 처리
- [ ] 각 단계별 try-except
- [ ] 부분 실패 시에도 리포트 생성
  ```python
  def _collect_data(self):
      stock_data = {}
      news_data = []

      try:
          stock_data = stock_collector.collect_prices()
      except Exception as e:
          print(f"  ⚠️  주가 데이터 수집 실패: {e}")

      try:
          news_data = news_collector.collect_news()
      except Exception as e:
          print(f"  ⚠️  뉴스 수집 실패: {e}")

      if not stock_data and not news_data:
          raise Exception("모든 데이터 수집 실패")

      return stock_data, news_data
  ```

### Task 7: 로깅 설정
- [ ] `src/utils/logger.py` 구현
  ```python
  import logging
  from datetime import datetime

  def setup_logger():
      logger = logging.getLogger("sector_analysis")
      logger.setLevel(logging.INFO)

      # 파일 핸들러
      fh = logging.FileHandler(f"logs/run_{datetime.now():%Y%m%d_%H%M%S}.log")
      fh.setLevel(logging.DEBUG)

      # 포맷터
      formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
      fh.setFormatter(formatter)

      logger.addHandler(fh)

      return logger
  ```

### Task 8: 통합 테스트
- [ ] 전체 파이프라인 end-to-end 테스트
- [ ] `python run_scenario.py --scenario semiconductor-weekly` 실행
- [ ] 생성된 리포트 품질 검증
- [ ] 에러 케이스 테스트 (네트워크 오류, API 실패 등)

### Task 9: 문서화
- [ ] README.md 업데이트 (사용 방법)
  ```markdown
  ## 사용 방법

  ### 환경 설정
  1. 가상환경 생성 및 활성화
     ```bash
     python -m venv venv
     source venv/bin/activate  # Windows: venv\\Scripts\\activate
     ```

  2. 패키지 설치
     ```bash
     pip install -r requirements.txt
     ```

  3. API 키 설정
     ```bash
     cp .env.example .env
     # .env 파일에 ANTHROPIC_API_KEY 입력
     ```

  ### 실행
  ```bash
  python run_scenario.py --scenario semiconductor-weekly
  ```

  ### 리포트 확인
  생성된 리포트는 `reports/` 디렉토리에서 확인할 수 있습니다.
  ```

---

## Technical Notes

### Markdown 테이블 포맷팅
```python
def format_table_row(columns: list, widths: list) -> str:
    """컬럼 너비를 맞춰 테이블 행 생성"""
    return "| " + " | ".join(
        str(col).ljust(width) for col, width in zip(columns, widths)
    ) + " |"
```

### 파일명 생성
```python
from datetime import datetime

def generate_filename(scenario: str, date: datetime = None) -> str:
    if date is None:
        date = datetime.now()
    return f"{scenario}_{date:%Y%m%d}.md"

# Example: semiconductor_weekly_20251118.md
```

### Progress Bar (Optional)
```python
# Simple progress indicator
import sys

def print_progress(step: int, total: int, message: str):
    bar_length = 40
    progress = step / total
    filled = int(bar_length * progress)

    bar = "█" * filled + "░" * (bar_length - filled)
    print(f"\r[{bar}] {step}/{total} {message}", end="", flush=True)

    if step == total:
        print()  # New line when complete
```

---

## Definition of Done

- [ ] 모든 Acceptance Criteria 충족
- [ ] 모든 Tasks 완료
- [ ] `python run_scenario.py --scenario semiconductor-weekly` 실행 성공
- [ ] 생성된 Markdown 리포트가 모든 섹션 포함
- [ ] 리포트 형식이 읽기 쉽고 전문적
- [ ] 전체 실행 시간 < 2분
- [ ] 에러 핸들링 동작 확인
- [ ] 로그 파일 생성 확인
- [ ] README 문서화 완료
- [ ] 3회 연속 실행 성공 (안정성 검증)

---

## Dependencies

**선행 작업**:
- Story 1: 데이터 수집
- Story 2: 분석 로직
- Story 3: LLM 통합

**후속 작업**: 없음 (Epic 1의 마지막 Story)

---

## Risks

| 리스크 | 완화 전략 |
|--------|-----------|
| 파이프라인 중간 실패 | 부분 실패 허용, Fallback 리포트 생성 |
| Markdown 형식 오류 | 템플릿 검증, 테스트 |
| 파일 저장 실패 | 디렉토리 존재 확인, 권한 체크 |
| CLI 사용성 낮음 | 명확한 메시지, 진행 상황 표시 |

---

## Estimated Time

**Story Points**: 3

**예상 시간**: 1일
- Task 1-2 (MarkdownReporter 구현): 2-3시간
- Task 3-4 (CLI, Scenario 통합): 3-4시간
- Task 5-7 (진행 표시, 에러 핸들링, 로깅): 2-3시간
- Task 8-9 (통합 테스트, 문서화): 1-2시간

---

## Notes

- Markdown 리포트는 간단하고 읽기 쉽게 (과도한 포맷팅 지양)
- CLI 출력은 사용자 경험 중요 (진행 상황 명확히)
- 로그는 디버깅용 (사용자는 CLI 출력만 봄)
- Epic 1 완료 후 실제 사용하며 피드백 수집
- Epic 2 (프레임워크 구축) 전에 배운 패턴 정리

---

**Created**: 2025-11-18
**Last Updated**: 2025-11-18
**Status**: Blocked (Story 1-3 완료 대기)
