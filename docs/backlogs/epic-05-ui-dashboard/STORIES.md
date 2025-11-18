# Epic 5: 사용자 인터페이스 - User Stories 개요

## Story 목록

### Story 1: FastAPI 백엔드 및 리포트 API
**Points**: 5 | **Priority**: Should

**User Story**:
As a 개발자, I want FastAPI로 리포트 CRUD API 구현, So that 프론트엔드에서 리포트 데이터를 조회할 수 있다

**Acceptance Criteria**:
- [ ] PostgreSQL 스키마 설계 및 생성
- [ ] FastAPI 프로젝트 초기화
- [ ] 리포트 목록 조회 API (`GET /reports`)
- [ ] 리포트 상세 조회 API (`GET /reports/{id}`)
- [ ] 리포트 생성 시 DB 저장 로직 추가
- [ ] API 문서 자동 생성 (Swagger)

**Tasks**:
1. PostgreSQL 설치 및 스키마 생성
   ```sql
   CREATE TABLE reports (...);
   CREATE TABLE metrics (...);
   ```
2. FastAPI 프로젝트 구조
   ```
   backend/
   ├── api/
   │   ├── main.py
   │   ├── routers/reports.py
   │   └── models/report.py
   └── database.py
   ```
3. SQLAlchemy ORM 모델 정의
4. 리포트 CRUD API 구현
5. 리포트 생성 시 DB 저장 로직 추가
6. API 테스트 (Postman 또는 curl)

---

### Story 2: React 프론트엔드 기본 구조 및 리포트 리스트
**Points**: 8 | **Priority**: Should

**User Story**:
As a 사용자, I want 웹 대시보드에서 리포트 목록 조회, So that 터미널 없이 편하게 리포트를 확인할 수 있다

**Acceptance Criteria**:
- [ ] React + TypeScript 프로젝트 초기화
- [ ] 대시보드 홈 페이지 구현
- [ ] 리포트 카드 컴포넌트 (제목, 날짜, 요약)
- [ ] API 클라이언트 (axios)
- [ ] 시나리오별 필터링
- [ ] 날짜 범위 검색
- [ ] 반응형 디자인 (모바일 지원)

**Tasks**:
1. React 프로젝트 초기화 (`create vite@latest`)
2. 디렉토리 구조
   ```
   frontend/
   ├── src/
   │   ├── components/
   │   │   ├── Dashboard.tsx
   │   │   └── ReportCard.tsx
   │   ├── services/api.ts
   │   └── App.tsx
   ```
3. API 클라이언트 구현
4. Dashboard 컴포넌트 (리포트 목록)
5. ReportCard 컴포넌트 (카드 UI)
6. 필터 및 검색 기능
7. Tailwind CSS 스타일링
8. 모바일 반응형 테스트

---

### Story 3: 리포트 뷰어 및 Markdown 렌더링
**Points**: 5 | **Priority**: Should

**User Story**:
As a 사용자, I want 리포트 상세 페이지에서 전체 내용 확인, So that Markdown 파일을 직접 열지 않아도 된다

**Acceptance Criteria**:
- [ ] 리포트 상세 페이지 구현
- [ ] Markdown → HTML 렌더링 (react-markdown)
- [ ] 목차 (TOC) 자동 생성
- [ ] 이전/다음 리포트 네비게이션
- [ ] 코드 블록 하이라이팅
- [ ] PDF 다운로드 (선택)

**Tasks**:
1. ReportDetail 컴포넌트 구현
2. `react-markdown` 통합
3. 목차 생성 로직 (헤딩 추출)
4. 네비게이션 버튼 (이전/다음)
5. 코드 하이라이팅 (`react-syntax-highlighter`)
6. 인쇄 최적화 CSS
7. PDF 다운로드 (선택, html2pdf.js)

---

### Story 4: 히스토리 차트 및 비교 기능
**Points**: 3 | **Priority**: Could

**User Story**:
As a 투자자, I want 섹터 점수 추이 차트, So that 트렌드를 시각적으로 파악할 수 있다

**Acceptance Criteria**:
- [ ] 섹터 점수 시계열 차트 (Recharts)
- [ ] 8주 추이 선 차트
- [ ] 주간 변동률 막대 차트
- [ ] 날짜 범위 선택기
- [ ] 여러 지표 동시 비교

**Tasks**:
1. HistoryChart 컴포넌트 구현
2. Recharts 통합
   - LineChart (시계열)
   - BarChart (변동률)
3. API 엔드포인트 (`GET /metrics`)
4. 날짜 범위 선택기 (date picker)
5. 지표 선택 체크박스 (SMH, NVDA 등)
6. 차트 인터랙션 (툴팁, 줌)

---

## Epic 5 완료 기준 (기본)

- [ ] Story 1-3 완료 (Story 4는 선택)
- [ ] 웹 대시보드 로컬 실행 성공
- [ ] 리포트 목록 조회 및 상세 확인 가능
- [ ] 모바일에서 정상 표시
- [ ] 페이지 로딩 < 2초

---

## Epic 5 완료 기준 (확장)

- [ ] Story 4 포함 전체 완료
- [ ] 히스토리 차트 동작
- [ ] 배포 (로컬 Docker 또는 클라우드)

---

## Streamlit 대안

React 대신 Streamlit 선택 시:
- **Story 2-4를 하나로 통합** (Story Points: 8 → 4)
- Python만으로 빠른 개발
- 프로토타입 2-3일 완성 가능
- 단점: 커스터마이징 제한, 프로덕션 스케일링 어려움

**추천**: 빠른 검증 우선이면 Streamlit, 확장성 중요하면 React

---

**다음 단계**: Epic 6 (추가 시나리오) 또는 Epic 7 (고도화)
