# Story 1: FastAPI 백엔드 및 리포트 API

**Epic**: Epic 5 - 사용자 인터페이스
**Points**: 5
**Priority**: Should
**Status**: Todo

---

## User Story

**As a** 개발자
**I want** FastAPI로 리포트 CRUD API 구현
**So that** 프론트엔드에서 리포트 데이터를 조회할 수 있다

---

## Background & Context

### 문제 정의
- 현재 리포트는 Markdown 파일로만 저장
- 웹 대시보드에서 조회하려면 API 필요
- 검색, 필터링, 정렬 등 기능 구현 어려움

### 솔루션
- PostgreSQL DB에 리포트 메타데이터 저장
- FastAPI로 RESTful API 구현
- 리포트 생성 시 자동 DB 저장

### 범위
- Phase 1: 기본 CRUD (생성, 조회, 목록)
- Phase 2: 고급 기능 (검색, 통계, 비교)

---

## Acceptance Criteria

### 필수 조건

- [ ] **AC1**: PostgreSQL 스키마 설계 및 생성
  - reports 테이블 (리포트 메타데이터)
  - metrics 테이블 (시계열 지표)
  - events 테이블 (감지된 이벤트)

- [ ] **AC2**: FastAPI 프로젝트 초기화
  - 프로젝트 구조
  - SQLAlchemy ORM
  - Pydantic 스키마

- [ ] **AC3**: 리포트 CRUD API
  - `GET /reports` - 리포트 목록 (페이징, 필터링)
  - `GET /reports/{id}` - 리포트 상세
  - `POST /reports` - 리포트 생성
  - `DELETE /reports/{id}` - 리포트 삭제 (선택)

- [ ] **AC4**: 리포트 생성 시 자동 DB 저장
  - BaseScenario에 DB 저장 로직 추가
  - 파일 + DB 동시 저장

- [ ] **AC5**: API 문서 자동 생성
  - Swagger UI (`/docs`)
  - ReDoc (`/redoc`)

---

## Tasks

### 1. PostgreSQL 설치 및 스키마 생성

**PostgreSQL 설치** (macOS):
```bash
brew install postgresql@15
brew services start postgresql@15

# DB 생성
createdb analysis_economy
```

**스키마 정의** (`backend/schema.sql`):
```sql
-- backend/schema.sql
CREATE TABLE IF NOT EXISTS reports (
    id SERIAL PRIMARY KEY,
    scenario_id VARCHAR(100) NOT NULL,
    scenario_name VARCHAR(255) NOT NULL,
    report_path TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    event_context JSONB,  -- 트리거한 이벤트 정보
    metadata JSONB,  -- 추가 메타데이터
    summary TEXT  -- 요약 (프론트엔드 미리보기용)
);

CREATE TABLE IF NOT EXISTS metrics (
    id SERIAL PRIMARY KEY,
    report_id INTEGER REFERENCES reports(id) ON DELETE CASCADE,
    metric_name VARCHAR(100) NOT NULL,
    metric_value FLOAT NOT NULL,
    metric_date DATE NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS events (
    id SERIAL PRIMARY KEY,
    event_id VARCHAR(100) NOT NULL,
    event_name VARCHAR(255) NOT NULL,
    priority VARCHAR(20) NOT NULL,
    confidence FLOAT NOT NULL,
    article_title TEXT,
    article_url TEXT,
    detected_at TIMESTAMP DEFAULT NOW(),
    triggered_reports INTEGER[]  -- 트리거된 리포트 ID 배열
);

-- 인덱스
CREATE INDEX idx_reports_scenario ON reports(scenario_id);
CREATE INDEX idx_reports_created ON reports(created_at DESC);
CREATE INDEX idx_metrics_report ON metrics(report_id);
CREATE INDEX idx_events_detected ON events(detected_at DESC);
```

**스키마 적용**:
```bash
psql -d analysis_economy -f backend/schema.sql
```

### 2. FastAPI 프로젝트 구조

```bash
mkdir -p backend
mkdir -p backend/api/routers
mkdir -p backend/models
mkdir -p backend/schemas
touch backend/api/__init__.py
touch backend/api/main.py
touch backend/api/routers/__init__.py
touch backend/api/routers/reports.py
touch backend/models/__init__.py
touch backend/models/report.py
touch backend/schemas/__init__.py
touch backend/schemas/report.py
touch backend/database.py
touch backend/config.py
```

**구조**:
```
backend/
├── api/
│   ├── main.py          # FastAPI 앱
│   └── routers/
│       └── reports.py   # 리포트 라우터
├── models/
│   └── report.py        # SQLAlchemy 모델
├── schemas/
│   └── report.py        # Pydantic 스키마
├── database.py          # DB 연결
└── config.py            # 설정
```

### 3. 데이터베이스 연결

**backend/config.py**:
```python
# backend/config.py
import os
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    DATABASE_URL: str = os.getenv(
        'DATABASE_URL',
        'postgresql://localhost/analysis_economy'
    )
    API_TITLE: str = "Analysis Economy API"
    API_VERSION: str = "1.0.0"

    class Config:
        env_file = ".env"

settings = Settings()
```

**backend/database.py**:
```python
# backend/database.py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from backend.config import settings

engine = create_engine(settings.DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

def get_db():
    """DB 세션 의존성"""
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### 4. SQLAlchemy 모델

**backend/models/report.py**:
```python
# backend/models/report.py
from sqlalchemy import Column, Integer, String, Text, DateTime, JSON, Float, Date, ARRAY
from sqlalchemy.sql import func
from backend.database import Base

class Report(Base):
    __tablename__ = "reports"

    id = Column(Integer, primary_key=True, index=True)
    scenario_id = Column(String(100), nullable=False, index=True)
    scenario_name = Column(String(255), nullable=False)
    report_path = Column(Text, nullable=False)
    created_at = Column(DateTime(timezone=True), server_default=func.now(), index=True)
    event_context = Column(JSON, nullable=True)
    metadata = Column(JSON, nullable=True)
    summary = Column(Text, nullable=True)

class Metric(Base):
    __tablename__ = "metrics"

    id = Column(Integer, primary_key=True, index=True)
    report_id = Column(Integer, nullable=False, index=True)
    metric_name = Column(String(100), nullable=False)
    metric_value = Column(Float, nullable=False)
    metric_date = Column(Date, nullable=False)
    created_at = Column(DateTime(timezone=True), server_default=func.now())

class Event(Base):
    __tablename__ = "events"

    id = Column(Integer, primary_key=True, index=True)
    event_id = Column(String(100), nullable=False)
    event_name = Column(String(255), nullable=False)
    priority = Column(String(20), nullable=False)
    confidence = Column(Float, nullable=False)
    article_title = Column(Text, nullable=True)
    article_url = Column(Text, nullable=True)
    detected_at = Column(DateTime(timezone=True), server_default=func.now(), index=True)
    triggered_reports = Column(ARRAY(Integer), nullable=True)
```

### 5. Pydantic 스키마

**backend/schemas/report.py**:
```python
# backend/schemas/report.py
from pydantic import BaseModel, Field
from datetime import datetime
from typing import Optional, Dict, Any

class ReportBase(BaseModel):
    scenario_id: str
    scenario_name: str
    report_path: str
    event_context: Optional[Dict[str, Any]] = None
    metadata: Optional[Dict[str, Any]] = None
    summary: Optional[str] = None

class ReportCreate(ReportBase):
    pass

class ReportResponse(ReportBase):
    id: int
    created_at: datetime

    class Config:
        from_attributes = True

class ReportListResponse(BaseModel):
    total: int
    page: int
    page_size: int
    reports: list[ReportResponse]
```

### 6. API 라우터

**backend/api/routers/reports.py**:
```python
# backend/api/routers/reports.py
from fastapi import APIRouter, Depends, HTTPException, Query
from sqlalchemy.orm import Session
from typing import Optional
from backend.database import get_db
from backend.models.report import Report
from backend.schemas.report import ReportResponse, ReportListResponse, ReportCreate

router = APIRouter(prefix="/reports", tags=["Reports"])

@router.get("/", response_model=ReportListResponse)
def get_reports(
    page: int = Query(1, ge=1),
    page_size: int = Query(20, ge=1, le=100),
    scenario_id: Optional[str] = None,
    db: Session = Depends(get_db)
):
    """리포트 목록 조회 (페이징, 필터링)"""
    query = db.query(Report)

    # 필터링
    if scenario_id:
        query = query.filter(Report.scenario_id == scenario_id)

    # 총 개수
    total = query.count()

    # 페이징
    reports = query.order_by(Report.created_at.desc()) \
        .offset((page - 1) * page_size) \
        .limit(page_size) \
        .all()

    return {
        "total": total,
        "page": page,
        "page_size": page_size,
        "reports": reports
    }

@router.get("/{report_id}", response_model=ReportResponse)
def get_report(report_id: int, db: Session = Depends(get_db)):
    """리포트 상세 조회"""
    report = db.query(Report).filter(Report.id == report_id).first()

    if not report:
        raise HTTPException(status_code=404, detail="Report not found")

    return report

@router.post("/", response_model=ReportResponse, status_code=201)
def create_report(report: ReportCreate, db: Session = Depends(get_db)):
    """리포트 생성"""
    db_report = Report(**report.dict())
    db.add(db_report)
    db.commit()
    db.refresh(db_report)

    return db_report

@router.delete("/{report_id}", status_code=204)
def delete_report(report_id: int, db: Session = Depends(get_db)):
    """리포트 삭제 (선택)"""
    report = db.query(Report).filter(Report.id == report_id).first()

    if not report:
        raise HTTPException(status_code=404, detail="Report not found")

    db.delete(report)
    db.commit()

    return None
```

### 7. FastAPI 메인 앱

**backend/api/main.py**:
```python
# backend/api/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from backend.config import settings
from backend.api.routers import reports

app = FastAPI(
    title=settings.API_TITLE,
    version=settings.API_VERSION,
    description="Analysis Economy Backend API"
)

# CORS 설정 (프론트엔드 허용)
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173"],  # Vite 개발 서버
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 라우터 등록
app.include_router(reports.router)

@app.get("/")
def root():
    return {"message": "Analysis Economy API", "version": settings.API_VERSION}

@app.get("/health")
def health_check():
    return {"status": "healthy"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### 8. BaseScenario DB 저장 로직 추가

**src/scenarios/base_scenario.py 수정**:
```python
# BaseScenario 클래스에 추가
from backend.database import SessionLocal
from backend.models.report import Report

class BaseScenario:
    # ... 기존 코드

    def _save_to_db(self, report_path: str):
        """리포트 DB 저장"""
        db = SessionLocal()

        try:
            db_report = Report(
                scenario_id=self.SCENARIO_NAME,
                scenario_name=self.__class__.__name__,
                report_path=report_path,
                event_context=self.event_context,
                summary=self._extract_summary(report_path)
            )

            db.add(db_report)
            db.commit()
            db.refresh(db_report)

            logger.info(f"Report saved to DB with ID: {db_report.id}")

        except Exception as e:
            logger.error(f"Failed to save report to DB: {str(e)}")
            db.rollback()

        finally:
            db.close()

    def _extract_summary(self, report_path: str) -> str:
        """리포트 요약 추출 (첫 200자)"""
        with open(report_path, 'r', encoding='utf-8') as f:
            content = f.read()
            return content[:200] + "..."

    def run(self) -> Dict:
        # ... 기존 로직
        report_path = self.save_report(report_content)

        # DB 저장 추가
        self._save_to_db(report_path)

        # ...
```

### 9. 의존성 추가

**requirements.txt**:
```
fastapi==0.110.0
uvicorn[standard]==0.27.1
sqlalchemy==2.0.27
psycopg2-binary==2.9.9
pydantic-settings==2.2.1
```

### 10. 테스트

**서버 실행**:
```bash
cd backend
uvicorn api.main:app --reload --port 8000
```

**API 테스트**:
```bash
# 헬스 체크
curl http://localhost:8000/health

# 리포트 목록
curl http://localhost:8000/reports

# API 문서
open http://localhost:8000/docs
```

---

## Technical Notes

### PostgreSQL vs SQLite
- **PostgreSQL**: 프로덕션 권장, JSONB 지원, 동시성
- **SQLite**: 개발용, 간단, 파일 기반

초기엔 PostgreSQL로 시작

### ORM vs Raw SQL
- **ORM (SQLAlchemy)**: 타입 안전, 마이그레이션, 생산성
- **Raw SQL**: 성능, 유연성

ORM으로 시작, 필요 시 Raw SQL 혼용

### API 버저닝
- URL: `/v1/reports` (명확)
- Header: `Accept: application/vnd.api+json;version=1` (유연)

초기엔 버저닝 없이, 나중에 추가

---

## Definition of Done

- [ ] PostgreSQL 스키마 생성 완료
- [ ] FastAPI 프로젝트 구조 완성
- [ ] 리포트 CRUD API 구현
- [ ] BaseScenario DB 저장 로직 추가
- [ ] Swagger 문서 확인
- [ ] API 테스트 성공
- [ ] 문서 업데이트
- [ ] AC 모두 충족

---

## Dependencies

**Prerequisite**:
- Epic 2 (BaseScenario) 완료
- PostgreSQL 설치

**Blocks**:
- Story 2 (React 프론트엔드) - API 의존

---

## Estimated Time

- **PostgreSQL 스키마**: 1시간
- **FastAPI 구조**: 1.5시간
- **CRUD API**: 2시간
- **DB 저장 로직**: 1시간
- **테스트**: 1시간
- **Total**: 5 Story Points

---

## Risks & Mitigation

**Risk 1**: PostgreSQL 연결 실패
- **원인**: 설정 오류, 권한 문제
- **완화**: 상세 에러 로깅, 환경 변수 체크

**Risk 2**: ORM 성능 이슈
- **원인**: N+1 쿼리, 비효율적 조인
- **완화**: 쿼리 프로파일링, 인덱스 최적화

---

## Success Metrics

- ✅ API 응답 시간 < 200ms
- ✅ DB 저장 성공률 > 99%
- ✅ Swagger 문서 완성도

---

## References

- [FastAPI 문서](https://fastapi.tiangolo.com/)
- [SQLAlchemy 문서](https://www.sqlalchemy.org/)
- [PostgreSQL 문서](https://www.postgresql.org/docs/)
