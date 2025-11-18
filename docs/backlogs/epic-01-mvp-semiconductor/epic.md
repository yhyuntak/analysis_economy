# Epic 1: MVP - 반도체 주간 점검 자동화

## Epic 개요

**ID**: EPIC-001
**제목**: 반도체 섹터 주간 점검 자동화 (시나리오 #1)
**Priority**: Must Have
**Story Points**: 21
**Target Release**: Release 1 (Month 1-3)
**Status**: 준비 완료

---

## PM 관점 🎯

### 비즈니스 목표

1. **프로젝트 가치 검증**: 전체 프로젝트의 핵심 가치 제안을 실제 동작하는 기능으로 증명
2. **사용자 피드백 수집**: 실제 사용 경험을 통해 개선점 및 추가 요구사항 파악
3. **학습 기반 확립**: 첫 번째 시나리오 구축 과정에서 배운 패턴을 문서화하여 이후 시나리오 개발 가속화

### 사용자 가치

**Target User**: 반도체 섹터에 관심 있는 투자자 (본인)

**Pain Point**:
- 매주 반도체 섹터 동향을 수동으로 체크하는 데 2-3시간 소요
- 여러 소스를 확인해야 하는 번거로움
- 중요한 시그널을 놓칠 위험

**Provided Value**:
- 명령어 하나로 종합적인 주간 리포트 생성 (< 2분)
- 주요 지표, 뉴스, 시장 동향을 한눈에 파악
- LLM 기반 인사이트로 맥락 있는 해석 제공

### Success Metrics

- [ ] **완료율**: 리포트 생성 성공률 > 95%
- [ ] **속도**: 실행부터 리포트 생성까지 < 2분
- [ ] **품질**: 생성된 리포트가 실제 투자 판단에 활용 가능 (정성 평가)
- [ ] **학습**: 3개 시나리오 추가 구현 시 개발 시간 50% 단축 (프레임워크 효과 측정)

### MVP Scope

**In Scope**:
- 반도체 섹터 ETF (SMH, SOXX) 가격 데이터 수집
- 주요 반도체 기업 (삼성전자, TSMC, NVIDIA, Intel, ASML) 주가 수집
- 최근 1주일 반도체 관련 뉴스 수집 (Google News)
- LLM 기반 종합 분석 (주간 동향, 주요 이슈, 투자 시사점)
- Markdown 형식 리포트 생성

**Out of Scope** (향후 Epic에서):
- 자동 스케줄링 (Epic 3)
- 슬랙 알림 (Epic 6)
- 웹 UI (Epic 7)
- 히스토리 비교 (Epic 7)
- 차트 생성 (Epic 7)

---

## Designer 관점 🎨

### UX 목표

1. **즉시 사용 가능**: 복잡한 설정 없이 첫 실행에서 바로 결과 확인
2. **읽기 쉬운 아웃풋**: 투자 의사결정에 바로 활용할 수 있는 구조화된 리포트
3. **신뢰성**: 데이터 출처, 생성 시각 명시로 정보 신뢰도 확보

### User Journey

```
[사용자 액션] → [시스템 반응] → [사용자 경험]

1. 터미널에서 명령어 실행
   python run_scenario.py --scenario semiconductor-weekly

2. 진행 상황 표시
   "데이터 수집 중... ✓"
   "LLM 분석 중... ✓"
   "리포트 생성 중... ✓"

3. 완료 메시지 및 파일 경로
   "리포트 생성 완료: reports/semiconductor_weekly_20251118.md"

4. 리포트 열어서 확인
   - 요약 (3줄)
   - 주요 지표 테이블
   - 섹터 동향 해석
   - 주요 뉴스 요약
   - 투자 시사점
```

### CLI Output Format

```
========================================
반도체 섹터 주간 점검 시작
========================================

[1/4] 데이터 수집 중...
  - ETF 가격 데이터 수집 완료 (SMH, SOXX)
  - 주요 기업 주가 수집 완료 (5개 기업)
  - 뉴스 수집 완료 (12건)

[2/4] 데이터 전처리 중...
  - 주간 변동률 계산 완료
  - 뉴스 중복 제거 및 필터링 완료

[3/4] LLM 분석 중...
  - 프롬프트 생성 완료
  - Claude API 호출 중...
  - 인사이트 생성 완료

[4/4] 리포트 생성 중...
  - Markdown 파일 작성 완료

========================================
리포트 생성 완료
========================================

경로: reports/semiconductor_weekly_20251118.md
생성 시각: 2025-11-18 14:30:25
소요 시간: 1분 23초

다음 명령어로 리포트 확인:
  cat reports/semiconductor_weekly_20251118.md
```

### Report Template Structure

```markdown
# 반도체 섹터 주간 점검

**생성 일시**: 2025-11-18 14:30
**분석 기간**: 2025-11-11 ~ 2025-11-18
**데이터 소스**: Yahoo Finance, Google News

---

## 요약

[LLM이 생성한 3줄 요약]

---

## 주요 지표

| 지표 | 현재가 | 주간 변동 | 월간 변동 |
|------|---------|-----------|-----------|
| SMH ETF | $245.30 | +2.3% | +5.1% |
| SOXX ETF | $512.40 | +1.8% | +4.7% |
| 삼성전자 | 71,200원 | -0.5% | +2.3% |
| TSMC | $178.50 | +3.2% | +7.8% |
| NVIDIA | $495.20 | +4.1% | +9.2% |

---

## 섹터 동향

[LLM이 생성한 종합 분석]

---

## 주요 뉴스

1. **[제목]** (출처, 날짜)
   - 요약 내용

2. **[제목]** (출처, 날짜)
   - 요약 내용

---

## 투자 시사점

[LLM이 생성한 투자 관점 해석]

---

## 참고사항

- 본 리포트는 AI가 생성한 분석이며, 투자 권유가 아닙니다.
- 실제 투자 결정은 본인의 판단과 책임하에 이루어져야 합니다.
```

### 접근성 및 일관성

- **파일명 규칙**: `{scenario}_{frequency}_{YYYYMMDD}.md`
- **한국어 우선**: 모든 리포트는 한국어로 작성
- **Markdown 표준**: GitHub Flavored Markdown 준수
- **에러 메시지**: 명확한 원인 및 해결 방법 제시

---

## Developer 관점 💻

### 기술 목표

1. **End-to-End 파이프라인 구축**: 데이터 수집 → 분석 → 리포트 생성 전 과정 자동화
2. **LLM 통합 검증**: Claude API를 활용한 실제 인사이트 생성 가능성 확인
3. **확장 가능한 구조**: 이후 시나리오 추가 시 재사용 가능한 모듈 설계

### Architecture Overview

```
src/
├── scenarios/
│   └── semiconductor_weekly.py       # 시나리오 메인 로직
├── collectors/
│   ├── stock_collector.py            # 주가 데이터 수집
│   └── news_collector.py             # 뉴스 수집
├── analyzers/
│   └── llm_analyzer.py                # LLM 기반 분석
├── reporters/
│   └── markdown_reporter.py           # 리포트 생성
└── utils/
    ├── config.py                      # 설정 관리
    └── logger.py                      # 로깅

reports/                               # 생성된 리포트 저장
tests/                                 # 단위 테스트
requirements.txt                       # 의존성
.env                                   # API 키 (gitignore)
run_scenario.py                        # CLI 엔트리포인트
```

### Tech Stack

- **Python**: 3.9+
- **HTTP Client**: `requests`
- **HTML Parsing**: `beautifulsoup4`
- **Stock Data**: `yfinance`
- **LLM**: Anthropic Claude API (`anthropic` SDK)
- **Environment**: `python-dotenv`
- **CLI**: `argparse`

### 데이터 플로우

```
1. run_scenario.py 실행
   ↓
2. SemiconductorWeeklyScenario 인스턴스 생성
   ↓
3. StockCollector.collect()
   - SMH, SOXX ETF 가격 (yfinance)
   - 삼성전자, TSMC, NVIDIA, Intel, ASML 주가
   ↓
4. NewsCollector.collect()
   - Google News API (또는 RSS)
   - "semiconductor" 키워드 검색
   - 최근 7일 필터링
   ↓
5. LLMAnalyzer.analyze()
   - 수집된 데이터를 프롬프트로 변환
   - Claude API 호출
   - 인사이트 텍스트 반환
   ↓
6. MarkdownReporter.generate()
   - 템플릿 + 데이터 + LLM 인사이트 결합
   - Markdown 파일 저장
   ↓
7. 결과 출력 (파일 경로)
```

### Implementation Complexity

**총 Story Points**: 21

- **Story 1**: 데이터 수집 (5pt) - 중간 난이도 (API 통합 2개)
- **Story 2**: 분석 로직 (5pt) - 중간 난이도 (데이터 가공)
- **Story 3**: LLM 통합 (8pt) - 높은 난이도 (프롬프트 엔지니어링, API 호출 최적화)
- **Story 4**: 리포트 생성 (3pt) - 낮은 난이도 (템플릿 렌더링)

### 주요 기술 과제

1. **API Rate Limiting**: yfinance, Google News 호출 빈도 관리
   - **해결**: 요청 간 딜레이, 캐싱 고려

2. **LLM 프롬프트 최적화**: 일관되고 유용한 인사이트 생성
   - **해결**: Few-shot examples, 명확한 지시문

3. **에러 핸들링**: 외부 API 장애 시 graceful degradation
   - **해결**: try-except, fallback 로직, 로깅

4. **API 비용 관리**: Claude API 토큰 사용량 모니터링
   - **해결**: 입력 토큰 최소화, 출력 길이 제한

### Dependencies

```txt
# requirements.txt
requests==2.31.0
beautifulsoup4==4.12.2
yfinance==0.2.28
anthropic==0.7.0
python-dotenv==1.0.0
```

### Configuration

```python
# .env
ANTHROPIC_API_KEY=sk-ant-...
STOCK_SYMBOLS=005930.KS,TSM,NVDA,INTC,ASML
ETF_SYMBOLS=SMH,SOXX
NEWS_KEYWORDS=semiconductor,chip,반도체
REPORT_OUTPUT_DIR=reports/
```

---

## User Stories

이 Epic은 4개의 User Story로 구성됩니다:

1. **[Story 1] 반도체 섹터 데이터 수집** (5pt)
   - ETF 및 주요 기업 주가 데이터 수집
   - 관련 뉴스 수집

2. **[Story 2] 분석 로직 구현** (5pt)
   - 주간/월간 변동률 계산
   - 데이터 정규화 및 전처리

3. **[Story 3] LLM 통합 및 인사이트 생성** (8pt)
   - Claude API 통합
   - 프롬프트 엔지니어링
   - 인사이트 생성 로직

4. **[Story 4] 리포트 생성 및 출력** (3pt)
   - Markdown 템플릿 작성
   - 리포트 파일 생성
   - CLI 출력 포맷팅

---

## Acceptance Criteria

Epic 전체가 완료되었다고 판단하는 기준:

- [ ] `python run_scenario.py --scenario semiconductor-weekly` 실행 시 리포트 생성 성공
- [ ] 리포트에 모든 섹션 포함 (요약, 주요 지표, 섹터 동향, 뉴스, 투자 시사점)
- [ ] 생성 시간 < 2분
- [ ] 3회 연속 실행 시 모두 성공 (안정성 검증)
- [ ] LLM 생성 인사이트가 실제로 유용함 (정성 평가)
- [ ] 에러 발생 시 명확한 메시지 출력

---

## Risks & Mitigation

| 리스크 | 영향 | 확률 | 완화 전략 |
|--------|------|------|-----------|
| LLM API 비용 초과 | 높음 | 중간 | 토큰 사용량 로깅, 월 예산 설정 |
| 데이터 소스 장애 | 중간 | 낮음 | 다중 소스, fallback 로직 |
| 프롬프트 품질 낮음 | 높음 | 중간 | 반복 테스트, Few-shot examples |
| 스코프 확장 욕심 | 중간 | 높음 | MVP 원칙 고수, Out of Scope 명확히 |

---

## Timeline Estimate

**총 예상 기간**: 2-3주 (1인 개발, 학습 포함)

- **Week 1**: Story 1-2 (데이터 수집 및 분석 로직)
- **Week 2**: Story 3 (LLM 통합, 프롬프트 최적화)
- **Week 3**: Story 4 + 통합 테스트 + 문서화

---

## 다음 단계

1. Story 1부터 순차적으로 진행
2. 각 Story 완료 시 Acceptance Criteria 체크
3. Story 3 완료 후 전체 통합 테스트
4. 실제 사용하며 피드백 수집
5. Epic 2 (프레임워크 구축)로 이동 전에 학습한 패턴 문서화

---

**Epic Owner**: Developer (본인)
**Created**: 2025-11-18
**Last Updated**: 2025-11-18
