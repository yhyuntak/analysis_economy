# Epic 5: 사용자 인터페이스 및 대시보드

## Epic 개요

**ID**: EPIC-005
**제목**: 웹 대시보드 및 히스토리 비교 기능
**Priority**: Should Have
**Story Points**: 21
**Target Release**: Release 3 (Month 7-9)
**Status**: 대기 중
**의존성**: Epic 3 완료 필요 (Epic 4는 선택)

---

## PM 관점

### 비즈니스 목표

1. **사용성 개선**: Markdown 파일 찾기 → 웹에서 즉시 조회
2. **데이터 활용도 증가**: 히스토리 비교로 트렌드 파악, 의사결정 품질 향상
3. **사용자 확장 가능성**: 웹 UI로 비개발자도 접근 가능 (향후)

### 사용자 가치

**Target User**: 데이터 기반 투자자

**Pain Point**:
- 과거 리포트 찾기 번거로움 (파일 시스템 탐색)
- 이번 주 vs 지난주 비교 수동으로 해야 함
- 섹터 점수 추이 시각화 없음
- 모바일에서 리포트 확인 불편

**Provided Value**:
- 웹 브라우저에서 모든 리포트 즉시 조회
- 반도체 섹터 점수 8주 추이 차트로 확인
- 시나리오별 필터링, 날짜 검색
- 모바일에서도 편하게 확인 (반응형)

### Success Metrics

- [ ] **리포트 조회 시간**: 30초 → 5초 단축
- [ ] **히스토리 비교 사용 빈도**: 주 2회 이상
- [ ] **모바일 접근률**: 전체 조회의 30% 이상
- [ ] **페이지 로딩 속도**: < 2초

---

## Designer 관점

### UX 목표

1. **직관적 네비게이션**: 3클릭 이내에 원하는 리포트 도달
2. **시각적 인사이트**: 차트로 트렌드 한눈에 파악
3. **모바일 친화적**: 출퇴근 중에도 확인 가능
4. **깔끔한 디자인**: 정보 과부하 방지, 핵심 데이터 강조

### 화면 구성

#### 1. 대시보드 홈

```
┌─────────────────────────────────────────────────┐
│  📊 섹터 분석 대시보드                           │
├─────────────────────────────────────────────────┤
│  [최신 리포트]                                   │
│  ┌───────────────────┐  ┌───────────────────┐  │
│  │ 반도체 주간 점검   │  │ PMI 발표 분석     │  │
│  │ 2025-11-18 09:05  │  │ 2025-11-15 10:22  │  │
│  │ ⬆ +2.3% 강세     │  │ ⬇ -0.5% 약세     │  │
│  │ [상세보기]        │  │ [상세보기]        │  │
│  └───────────────────┘  └───────────────────┘  │
│                                                 │
│  [섹터 점수 추이]                                │
│  📈 [8주 차트]                                  │
│                                                 │
│  [필터]  시나리오: [전체▼]  날짜: [최근 30일▼] │
└─────────────────────────────────────────────────┘
```

#### 2. 리포트 상세 페이지

```
┌─────────────────────────────────────────────────┐
│  ← 돌아가기                                      │
├─────────────────────────────────────────────────┤
│  반도체 섹터 주간 점검                           │
│  2025-11-18 09:05 | 시나리오 #1                 │
├─────────────────────────────────────────────────┤
│  [요약] [주요지표] [분석] [뉴스] [투자시사점]   │
│                                                 │
│  ## 요약                                        │
│  - NVIDIA +4.1%, AI 수요 지속 강세              │
│  - 삼성전자 -0.5%, 메모리 가격 약세             │
│  - 전반적 섹터 상승세, 단기 과열 신호            │
│                                                 │
│  ## 주요 지표                                   │
│  [표 렌더링]                                    │
│                                                 │
│  ## 섹터 동향                                   │
│  [LLM 분석 텍스트]                              │
│                                                 │
│  [이전 주 리포트와 비교] [PDF 다운로드]          │
└─────────────────────────────────────────────────┘
```

#### 3. 히스토리 비교 페이지

```
┌─────────────────────────────────────────────────┐
│  히스토리 비교: 반도체 주간 점검                 │
├─────────────────────────────────────────────────┤
│  기간 선택: [최근 8주▼]                         │
│                                                 │
│  📈 섹터 점수 추이                               │
│  [선 차트: 반도체 ETF, NVIDIA, 삼성전자]        │
│                                                 │
│  📊 주간 변동률 비교                             │
│  [막대 차트: 8주간 변동률]                      │
│                                                 │
│  📝 주요 이벤트 타임라인                         │
│  - 11/18: AI 수요 급증 뉴스                     │
│  - 11/11: 메모리 가격 하락                      │
│  - 11/04: NVIDIA 실적 발표                      │
└─────────────────────────────────────────────────┘
```

### 색상 및 디자인 시스템

- **Primary**: #2563EB (파랑) - 신뢰, 안정
- **Success**: #10B981 (초록) - 상승, 긍정
- **Warning**: #F59E0B (주황) - 중립, 주의
- **Danger**: #EF4444 (빨강) - 하락, 위험
- **Font**: Pretendard (한글), Inter (영문)

---

## Developer 관점

### 기술 목표

1. **경량 백엔드**: FastAPI로 빠른 API 개발
2. **모던 프론트엔드**: React + TypeScript (또는 Streamlit 대안)
3. **데이터베이스 통합**: PostgreSQL로 리포트 메타데이터 관리
4. **차트 라이브러리**: Recharts로 인터랙티브 시각화

### Architecture Overview

```
backend/
├── api/
│   ├── main.py                 # FastAPI 엔트리포인트
│   ├── routers/
│   │   ├── reports.py          # 리포트 CRUD
│   │   ├── scenarios.py        # 시나리오 목록
│   │   └── analytics.py        # 히스토리 분석
│   ├── models/
│   │   └── report.py           # SQLAlchemy 모델
│   ├── database.py             # DB 연결
│   └── utils/
│       └── markdown_parser.py  # Markdown → HTML

frontend/
├── src/
│   ├── components/
│   │   ├── Dashboard.tsx       # 대시보드 홈
│   │   ├── ReportCard.tsx      # 리포트 카드
│   │   ├── ReportDetail.tsx    # 리포트 상세
│   │   └── HistoryChart.tsx    # 히스토리 차트
│   ├── services/
│   │   └── api.ts              # API 클라이언트
│   ├── App.tsx
│   └── main.tsx

database/
└── schema.sql                  # PostgreSQL 스키마
```

### 데이터베이스 스키마

```sql
-- reports 테이블
CREATE TABLE reports (
    id SERIAL PRIMARY KEY,
    scenario_name VARCHAR(100) NOT NULL,
    scenario_type VARCHAR(50),          -- weekly, event, monthly
    title VARCHAR(255) NOT NULL,
    content TEXT NOT NULL,              -- Markdown 본문
    summary TEXT,                        -- 3줄 요약
    generated_at TIMESTAMP NOT NULL,
    analysis_period_start DATE,
    analysis_period_end DATE,
    metadata JSONB,                     -- 유연한 메타데이터
    created_at TIMESTAMP DEFAULT NOW()
);

-- metrics 테이블 (시계열 데이터)
CREATE TABLE metrics (
    id SERIAL PRIMARY KEY,
    report_id INT REFERENCES reports(id),
    metric_name VARCHAR(100),           -- SMH_ETF, NVDA, etc.
    metric_value FLOAT,
    metric_change_pct FLOAT,
    recorded_at TIMESTAMP
);

-- 인덱스
CREATE INDEX idx_reports_scenario ON reports(scenario_name);
CREATE INDEX idx_reports_date ON reports(generated_at);
CREATE INDEX idx_metrics_report ON metrics(report_id);
```

### FastAPI 엔드포인트

```python
# backend/api/routers/reports.py
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session
from typing import List

router = APIRouter(prefix="/reports", tags=["reports"])

@router.get("/", response_model=List[ReportSummary])
def list_reports(
    scenario: Optional[str] = None,
    limit: int = 20,
    offset: int = 0,
    db: Session = Depends(get_db)
):
    """리포트 목록 조회"""
    query = db.query(Report)
    if scenario:
        query = query.filter(Report.scenario_name == scenario)
    reports = query.order_by(Report.generated_at.desc()) \
                   .limit(limit).offset(offset).all()
    return reports

@router.get("/{report_id}", response_model=ReportDetail)
def get_report(report_id: int, db: Session = Depends(get_db)):
    """리포트 상세 조회"""
    report = db.query(Report).filter(Report.id == report_id).first()
    if not report:
        raise HTTPException(status_code=404, detail="Report not found")
    return report

@router.get("/{report_id}/html")
def get_report_html(report_id: int, db: Session = Depends(get_db)):
    """Markdown → HTML 변환하여 반환"""
    report = db.query(Report).filter(Report.id == report_id).first()
    html_content = markdown_to_html(report.content)
    return {"html": html_content}
```

### React 컴포넌트 예시

```typescript
// frontend/src/components/Dashboard.tsx
import React, { useEffect, useState } from 'react';
import { fetchReports } from '../services/api';
import ReportCard from './ReportCard';

const Dashboard: React.FC = () => {
  const [reports, setReports] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchReports().then(data => {
      setReports(data);
      setLoading(false);
    });
  }, []);

  if (loading) return <div>Loading...</div>;

  return (
    <div className="dashboard">
      <h1>섹터 분석 대시보드</h1>
      <div className="report-grid">
        {reports.map(report => (
          <ReportCard key={report.id} report={report} />
        ))}
      </div>
    </div>
  );
};

export default Dashboard;
```

### Implementation Complexity

**총 Story Points**: 21

- **Story 1**: FastAPI 백엔드 및 리포트 API (5pt) - 중간 난이도
- **Story 2**: React 프론트엔드 기본 구조 및 리포트 리스트 (8pt) - 높은 난이도
- **Story 3**: 리포트 뷰어 및 Markdown 렌더링 (5pt) - 중간 난이도
- **Story 4**: 히스토리 차트 및 비교 기능 (3pt) - 낮은 난이도

### 주요 기술 과제

1. **Markdown → HTML 렌더링**
   - **해결**: `markdown-it` (백엔드) 또는 `react-markdown` (프론트엔드)

2. **차트 성능 최적화**
   - **해결**: Recharts lazy loading, 데이터 페이지네이션

3. **인증 (선택)**
   - **해결**: 초기엔 불필요, 나중에 JWT 추가

4. **모바일 반응형**
   - **해결**: Tailwind CSS, mobile-first 디자인

### Dependencies

```txt
# 백엔드
fastapi==0.104.1
uvicorn==0.24.0
sqlalchemy==2.0.23
psycopg2-binary==2.9.9
python-multipart==0.0.6
markdown-it-py==3.0.0

# 프론트엔드 (package.json)
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "typescript": "^5.2.2",
    "recharts": "^2.10.0",
    "react-markdown": "^9.0.0",
    "axios": "^1.6.0",
    "tailwindcss": "^3.3.5"
  }
}
```

### Streamlit 대안 (빠른 프로토타입)

```python
# dashboard.py (Streamlit 버전)
import streamlit as st
import pandas as pd
from services.report_service import get_reports, get_report_detail

st.set_page_config(page_title="섹터 분석 대시보드", layout="wide")

st.title("📊 섹터 분석 대시보드")

# 사이드바 필터
scenario = st.sidebar.selectbox("시나리오", ["전체", "반도체", "PMI"])
date_range = st.sidebar.date_input("날짜 범위", [])

# 리포트 목록
reports = get_reports(scenario=scenario)

for report in reports:
    with st.expander(f"{report['title']} ({report['generated_at']})"):
        st.markdown(report['summary'])
        if st.button("상세보기", key=report['id']):
            detail = get_report_detail(report['id'])
            st.markdown(detail['content'])

# 히스토리 차트
st.subheader("섹터 점수 추이")
df = pd.DataFrame(get_metrics_history())
st.line_chart(df)
```

**Streamlit 장점**:
- Python만으로 빠른 개발 (Story Points 절반)
- 데이터 과학자에게 친숙
- 배포 간단 (Streamlit Cloud)

**Streamlit 단점**:
- 커스터마이징 제한적
- 복잡한 인터랙션 어려움
- 프로덕션 스케일링 어려움

**추천**: 초기엔 Streamlit으로 프로토타입, 나중에 React 마이그레이션

---

## User Stories

이 Epic은 4개의 User Story로 구성됩니다:

1. **[Story 1] FastAPI 백엔드 및 리포트 API** (5pt)
   - PostgreSQL 스키마 설계
   - FastAPI 프로젝트 초기화
   - 리포트 CRUD API
   - 리포트 생성 시 DB 저장 로직 추가

2. **[Story 2] React 프론트엔드 기본 구조 및 리포트 리스트** (8pt)
   - React + TypeScript 프로젝트 초기화
   - 대시보드 홈 페이지
   - 리포트 카드 컴포넌트
   - API 클라이언트 (axios)
   - 필터 및 검색 기능

3. **[Story 3] 리포트 뷰어 및 Markdown 렌더링** (5pt)
   - 리포트 상세 페이지
   - Markdown → HTML 렌더링
   - 목차 (TOC) 자동 생성
   - 이전/다음 리포트 네비게이션

4. **[Story 4] 히스토리 차트 및 비교 기능** (3pt)
   - Recharts 통합
   - 섹터 점수 추이 차트
   - 주간 변동률 막대 차트
   - 날짜 범위 선택기

---

## Acceptance Criteria

Epic 전체가 완료되었다고 판단하는 기준:

- [ ] 웹 대시보드 접속 시 최신 리포트 20개 표시
- [ ] 리포트 클릭 시 전체 내용 Markdown 렌더링
- [ ] 시나리오별 필터링 동작
- [ ] 섹터 점수 8주 추이 차트 표시
- [ ] 모바일 (iPhone)에서 정상 표시
- [ ] 페이지 로딩 < 2초

---

## Risks & Mitigation

| 리스크 | 영향 | 확률 | 완화 전략 |
|--------|------|------|-----------|
| 프론트엔드 개발 시간 초과 | 중간 | 높음 | Streamlit 대안, 최소 기능부터 |
| PostgreSQL 설정 복잡도 | 낮음 | 중간 | Docker Compose로 간소화 |
| 차트 성능 저하 | 낮음 | 낮음 | 데이터 페이지네이션, lazy loading |
| 디자인 완성도 | 낮음 | 높음 | Tailwind CSS 템플릿 활용 |

---

## Timeline Estimate

**총 예상 기간**: 3-4주 (1인 개발)

- **Week 1**: Story 1 (백엔드)
- **Week 2**: Story 2 (프론트엔드 기본)
- **Week 3**: Story 3 (리포트 뷰어)
- **Week 4**: Story 4 (차트) + 통합 테스트

**Streamlit 대안 선택 시**: 1-2주로 단축 가능

---

## 다음 단계

1. Epic 3 완료 후 진행 (Epic 4는 선택)
2. Streamlit vs React 결정 (프로토타입 속도 vs 확장성)
3. Story 1부터 순차 진행
4. Story 2 완료 후 디자인 리뷰
5. 배포 전략 수립 (로컬 호스팅 vs 클라우드)

---

**Epic Owner**: Developer (본인)
**Created**: 2025-11-18
**Last Updated**: 2025-11-18
