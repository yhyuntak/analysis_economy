# 산업 섹터 분석 자동화 프로젝트 백로그

## 프로젝트 비전

AI 에이전트를 활용하여 11개 GICS 섹터의 실시간 모니터링, 분석, 인사이트 도출을 자동화하는 시스템 구축. 투자 의사결정을 위한 시의성 있고 신뢰할 수 있는 정보를 지속적으로 제공합니다.

**궁극적 목표**:
- **Agent A (모니터링)**: 24/7 세계 경제/뉴스/주식 센싱 → 이벤트 트리거
- **Agent B (분석)**: 트리거 수신 → 에이전트 루프 실행 → 심층 분석 → 리포트 생성

## 개발 철학

- **점진적 학습 중심 개발**: 작게 시작하여 실제 사용하면서 배우고 확장
- **MVP First**: 동작하는 최소 기능부터 검증
- **Code First**: 완벽한 설계보다 실행 가능한 코드
- **YAGNI**: 필요할 때 추상화 (시나리오 1: 하드코딩 OK, 시나리오 2 추가 시 패턴 추출)

## Epic 개요

### Release 1: Foundation (Month 1-3)
**목표**: MVP 검증 및 확장 가능한 기반 구축

| Epic | 제목 | Priority | Story Points | 비즈니스 가치 |
|------|------|----------|--------------|--------------|
| Epic 1 | MVP - 반도체 주간 점검 자동화 | Must | 21 | 핵심 가치 검증, 사용자 피드백 수집 |
| Epic 2 | 공통 프레임워크 구축 | Must | 13 | 확장성 확보, 개발 속도 향상 |

**핵심 마일스톤**:
- [ ] 첫 번째 자동 리포트 생성 성공
- [ ] 2-3개 시나리오 추가 구현으로 프레임워크 검증
- [ ] 공통 패턴 추출 완료

### Release 2: Automation (Month 4-6)
**목표**: 완전 자동화 및 시나리오 확장

| Epic | 제목 | Priority | Story Points | 비즈니스 가치 |
|------|------|----------|--------------|--------------|
| Epic 3 | 모니터링 에이전트 (Agent A) | Must | 13 | 실시간 감시, 수동 작업 제거 |
| Epic 4 | 분석 에이전트 고도화 (Agent B) | Should | 21 | 분석 품질 향상, 자율성 증가 |
| Epic 5 | 추가 시나리오 구현 (2-7) | Must | 34 | 커버리지 확대, 다양한 시장 상황 대응 |

**핵심 마일스톤**:
- [ ] 정기 자동 실행 시스템 가동 (Cron)
- [ ] 이벤트 기반 트리거 동작
- [ ] 7개 시나리오 모두 운영 중

### Release 3: Integration (Month 7-9)
**목표**: 사용자 경험 개선 및 통합

| Epic | 제목 | Priority | Story Points | 비즈니스 가치 |
|------|------|----------|--------------|--------------|
| Epic 6 | 알림 시스템 | Should | 8 | 정보 접근성, 즉각적인 대응 |
| Epic 7 | 웹 대시보드 | Should | 21 | 가시성, 히스토리 비교, 의사결정 지원 |

**핵심 마일스톤**:
- [ ] 슬랙 알림 통합
- [ ] 웹 대시보드 첫 배포
- [ ] 히스토리 비교 기능 제공

### Release 4: Intelligence (Month 10+)
**목표**: 지능형 시스템으로 진화

| Epic | 제목 | Priority | Story Points | 비즈니스 가치 |
|------|------|----------|--------------|--------------|
| Epic 8 | 고도화 (백테스팅, 학습) | Could | 34 | 정확도 개선, 신뢰도 향상 |

**핵심 마일스톤**:
- [ ] 과거 예측 정확도 측정
- [ ] 피드백 루프 구축
- [ ] 자율 개선 시스템

---

## Epic 상세 구조

### Epic 1: MVP - 반도체 주간 점검 자동화 (Must, 21pt)

**PM 관점 (비즈니스 목표)**:
- 실제 사용 가능한 최소 기능으로 프로젝트 가치 검증
- 매주 반도체 섹터 점검 시간 3시간 → 5분 단축
- 투자 의사결정에 실질적 도움 제공

**Designer 관점 (UX 목표)**:
- 간단한 CLI 명령어 하나로 실행 (python run_semiconductor.py)
- 읽기 쉬운 Markdown 리포트 (섹션 구조, 불릿 포인트)
- 핵심 인사이트를 3-5개 요약으로 즉시 파악

**Developer 관점 (기술 목표)**:
- 데이터 수집 → LLM 분석 → 리포트 생성 파이프라인 구축
- 하드코딩 허용 (빠른 검증 우선)
- 에러 핸들링 기본 수준 (재시도 로직, 로깅)

**User Stories**: 5개
- Story 1: 프로젝트 초기 설정 및 환경 구성 (3pt)
- Story 2: 반도체 섹터 데이터 수집 자동화 (5pt)
- Story 3: Claude API 통합 및 분석 프롬프트 (5pt)
- Story 4: Markdown 리포트 생성 엔진 (5pt)
- Story 5: CLI 인터페이스 및 엔드투엔드 테스트 (3pt)

### Epic 2: 공통 프레임워크 및 PMI 시나리오 추가 (Must, 13pt)

**PM 관점 (비즈니스 목표)**:
- 신규 시나리오 추가 개발 시간 3일 → 1일 단축
- PMI 시나리오 추가로 산업재 섹터 커버리지 확보
- 프레임워크 검증으로 향후 5개 시나리오 추가 리스크 감소

**Designer 관점 (UX 목표)**:
- 모든 시나리오에서 일관된 리포트 구조 (Overview → 데이터 → 분석 → 인사이트)
- 시나리오 간 전환 시 학습 곡선 제로
- 리포트 메타데이터 (생성일, 시나리오 타입) 표준화

**Developer 관점 (기술 목표)**:
- 시나리오 1 코드 분석 → 공통 패턴 추출
- 베이스 클래스 설계 (ScenarioBase, DataCollector, Analyzer, Reporter)
- 시나리오 2 (PMI) 구현으로 프레임워크 검증
- 코드 재사용률 70% 이상 달성

**User Stories**: 4개
- Story 1: 시나리오 #6 PMI 분석 구현 (복붙 방식, 3pt)
- Story 2: 공통 패턴 식별 및 베이스 클래스 설계 (5pt)
- Story 3: 시나리오 #1과 #6을 프레임워크로 리팩토링 (3pt)
- Story 4: 새 시나리오 추가 가이드 문서화 (2pt)

### Epic 3: Agent A 초기 - 정기 실행 및 알림 (Must, 13pt)

**PM 관점 (비즈니스 목표)**:
- 수동 실행 제거 → 사용자 개입 주 1회 → 월 1회로 감소
- 매주 월요일 오전 9시 자동 분석 실행
- 리포트 완성 즉시 슬랙 알림으로 5분 내 확인 가능

**Designer 관점 (UX 목표)**:
- "실행 잊어버림" 문제 해결
- 슬랙 알림 포맷: 제목 + 핵심 인사이트 3줄 + 전체 리포트 링크
- 알림 피로 방지: 중요도 필터링, 주 1-2회로 제한

**Developer 관점 (기술 목표)**:
- APScheduler 또는 cron 통합 (초기엔 cron 선호, 단순)
- 정기 실행: 매주 월요일 09:00 (cron: 0 9 * * MON)
- 슬랙 웹훅 통합 (Incoming Webhook)
- 실행 로그 및 실패 시 재시도 로직

**User Stories**: 3개
- Story 1: Cron 스케줄링 설정 및 정기 실행 (3pt)
- Story 2: 슬랙 웹훅 통합 및 알림 포맷 (5pt)
- Story 3: 실행 로그 및 에러 핸들링 (이메일 Fallback, 5pt)

### Epic 4: Agent A 진화 - 이벤트 감지 및 다중 시나리오 (Should, 21pt)

**PM 관점 (비즈니스 목표)**:
- 정기 실행 → 이벤트 기반 실행으로 진화 (대응 시간 1주 → 1시간)
- 금리 발표, FOMC 등 주요 이벤트 감지
- 5-7개 시나리오 동시 운영으로 전 섹터 커버리지

**Designer 관점 (UX 목표)**:
- 이벤트 알림: 긴급도 표시 (High/Medium/Low)
- 다중 리포트 관리: 날짜/시나리오별 폴더 구조
- 이벤트 요약: "금리 0.5% 인하 발표 → 금융/부동산 섹터 영향 분석 완료"

**Developer 관점 (기술 목표)**:
- RSS 피드 모니터링 (FED, 주요 뉴스)
- 키워드 기반 이벤트 감지 (금리, FOMC, GDP, PMI 등)
- 이벤트 → 시나리오 매핑 로직 (금리 이벤트 → 시나리오 #2 실행)
- 동시 실행 관리 (멀티 프로세싱 또는 순차 실행)

**User Stories**: 4개
- Story 1: RSS/뉴스 피드 모니터링 (5pt)
- Story 2: 키워드 기반 이벤트 감지 로직 (5pt)
- Story 3: 이벤트-시나리오 매핑 및 자동 트리거 (5pt)
- Story 4: 시나리오 #2 금리 인하 대응 추가 (6pt)

### Epic 5: 사용자 인터페이스 및 대시보드 (Should, 21pt)

**PM 관점 (비즈니스 목표)**:
- 리포트 접근성 향상 (Markdown 파일 찾기 → 웹에서 즉시 조회)
- 히스토리 비교로 트렌드 파악 (이번 주 vs 지난주 반도체 점수 변화)
- 투자 의사결정 속도 30% 향상

**Designer 관점 (UX 목표)**:
- 직관적인 대시보드: 최신 리포트 카드 뷰
- 시나리오별 필터링 및 검색
- 차트 시각화: 섹터 점수 추이, 주요 지표 트렌드
- 반응형 디자인 (모바일에서도 조회 가능)

**Developer 관점 (기술 목표)**:
- 백엔드: FastAPI (리포트 CRUD API)
- 프론트엔드: React + TypeScript (또는 Streamlit으로 빠른 프로토타입)
- 데이터베이스: PostgreSQL (리포트 메타데이터 + 본문)
- 차트: Recharts 또는 Chart.js
- 인증: 초기엔 불필요, 나중에 간단한 토큰 인증

**User Stories**: 4개
- Story 1: FastAPI 백엔드 및 리포트 API (5pt)
- Story 2: React 프론트엔드 기본 구조 및 리포트 리스트 (8pt)
- Story 3: 리포트 뷰어 및 Markdown 렌더링 (5pt)
- Story 4: 히스토리 차트 및 비교 기능 (3pt)

**대안**: Streamlit으로 초기 프로토타입 (Story Points 절반으로 감소)
- Streamlit은 Python만으로 빠르게 UI 구축 가능
- 복잡한 프론트엔드 불필요 시 추천
- 나중에 React로 마이그레이션 가능

### Epic 6: 알림 시스템 (Should, 8pt)

**비즈니스 목표**: 정보 접근성 향상, 즉각적인 의사결정 지원

**UX 목표**: 적절한 채널로 적시에 알림 (알림 피로 방지)

**기술 목표**: 이메일, 슬랙 통합, 조건부 알림 로직

**User Stories**: 3개
- Story 1: 슬랙 웹훅 통합 (3pt)
- Story 2: 이메일 알림 (2pt)
- Story 3: 조건부 알림 로직 (우선순위, 필터링) (3pt)

### Epic 7: 웹 대시보드 (Should, 21pt)

**비즈니스 목표**: 사용자 경험 향상, 데이터 활용도 증가

**UX 목표**: 직관적인 시각화, 히스토리 비교, 트렌드 파악

**기술 목표**: React/Vue + FastAPI, 차트 라이브러리, DB 통합

**User Stories**: 4개
- Story 1: 백엔드 API 구축 (FastAPI) (5pt)
- Story 2: 프론트엔드 기본 구조 (React) (8pt)
- Story 3: 리포트 뷰어 및 히스토리 (5pt)
- Story 4: 차트 시각화 (3pt)

### Epic 8: 고도화 (Could, 34pt)

**비즈니스 목표**: 시스템 신뢰도 향상, 지속적인 개선

**UX 목표**: 정확도 정보 제공, 투명성 증대

**기술 목표**: 백테스팅 프레임워크, 정확도 추적, 학습 루프

**User Stories**: 5개
- Story 1: 백테스팅 시스템 (8pt)
- Story 2: 정확도 측정 및 대시보드 (5pt)
- Story 3: 피드백 수집 인터페이스 (5pt)
- Story 4: 학습 데이터 파이프라인 (8pt)
- Story 5: 자율 개선 로직 (8pt)

---

## 우선순위 정책 (MoSCoW)

### Must Have (출시 필수)
- Epic 1: MVP 검증 - 프로젝트 존재 이유
- Epic 2: 프레임워크 - 확장성 기반
- Epic 3: 모니터링 - 자동화의 핵심
- Epic 5: 시나리오 - 실질적 가치 제공

### Should Have (중요하지만 우회 가능)
- Epic 4: 분석 고도화 - 품질 향상
- Epic 6: 알림 - 편의성 증대
- Epic 7: 웹 대시보드 - UX 개선

### Could Have (여유 있으면)
- Epic 8: 고도화 - 장기적 투자

### Won't Have (현 범위 밖)
- 모바일 앱
- 실시간 트레이딩 연동
- 다국어 지원 (한국어 우선)

---

## 기술 스택 로드맵

### Phase 1 (Release 1)
- **Language**: Python 3.9+
- **LLM**: Claude 3.5 Sonnet (Anthropic API)
- **Data**: requests, beautifulsoup4, yfinance
- **Environment**: venv
- **Output**: Markdown files

### Phase 2 (Release 2)
- **Scheduling**: APScheduler or cron
- **Storage**: SQLite (히스토리 저장)
- **Logging**: Python logging, structlog

### Phase 3 (Release 3)
- **Backend**: FastAPI
- **Frontend**: React + TypeScript
- **Database**: PostgreSQL
- **Notification**: Slack SDK, smtplib
- **Charts**: Recharts or Chart.js

### Phase 4 (Release 4)
- **ML/Analytics**: pandas, numpy, scikit-learn
- **Vector DB**: Chroma (RAG 확장 시)
- **Deployment**: Docker, Docker Compose

---

## 리스크 및 가정

### 주요 리스크
1. **LLM API 비용**: 사용량 증가 시 비용 폭증 가능
   - **완화**: 캐싱, 배치 처리, API 사용량 모니터링

2. **데이터 소스 신뢰성**: 외부 API 변경, 서비스 중단
   - **완화**: 다중 소스, fallback 로직, 에러 핸들링

3. **분석 정확도**: LLM 환각, 잘못된 해석
   - **완화**: 프롬프트 엔지니어링, 휴먼 피드백, 백테스팅

4. **범위 확장 (Scope Creep)**: 기능 추가 욕심
   - **완화**: MVP 원칙 고수, 각 Release 명확한 목표

### 가정
- Claude API는 안정적으로 사용 가능
- 공개 데이터로 충분한 분석 가능 (유료 데이터 불필요)
- 1인 개발 (학습 중심)
- 초기 사용자는 개발자 본인 (실 투자 의사결정 지원)

---

## Success Metrics

### Release 1
- [ ] MVP 리포트 생성 성공률 > 95%
- [ ] 리포트 생성 시간 < 2분
- [ ] 2-3개 추가 시나리오 구현으로 프레임워크 검증

### Release 2
- [ ] 7개 시나리오 모두 자동 실행
- [ ] 정기 실행 안정성 > 99%
- [ ] 이벤트 감지 정확도 측정

### Release 3
- [ ] 웹 대시보드 첫 배포
- [ ] 슬랙 알림 사용률 측정
- [ ] 히스토리 비교 기능 사용

### Release 4
- [ ] 백테스팅 30일 이상
- [ ] 예측 정확도 정량화
- [ ] 피드백 수집 10건 이상

---

## 다음 단계

1. **Epic 1 착수**: `docs/backlogs/epic-01-mvp-semiconductor/` 참조
2. **개발 환경 설정**: Python venv, Anthropic API 키
3. **첫 Story 시작**: Story 1 - 반도체 섹터 데이터 수집
4. **데일리 진행 체크**: 각 Story 완료 시 Acceptance Criteria 검증

---

**문서 버전**: 1.0
**최종 업데이트**: 2025-11-18
**작성자**: Architecture Advisor Agent
