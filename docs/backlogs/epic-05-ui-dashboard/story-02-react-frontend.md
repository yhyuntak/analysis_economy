# Story 2: React 프론트엔드 기본 구조 및 리포트 리스트

**Epic**: Epic 5 - 사용자 인터페이스
**Points**: 8
**Priority**: Should
**Status**: Todo

---

## User Story

**As a** 사용자
**I want** 웹 대시보드에서 리포트 목록 조회
**So that** 터미널 없이 편하게 리포트를 확인할 수 있다

---

## Background & Context

### 문제 정의
- 현재는 CLI 또는 파일 시스템으로만 리포트 확인
- 비기술 사용자는 Markdown 파일 열기 어려움
- 리포트 검색, 필터링 불편

### 솔루션
- React + TypeScript로 모던 웹 UI 구축
- 반응형 디자인 (데스크톱 + 모바일)
- 직관적인 카드 기반 레이아웃

### 범위
- Phase 1: 리포트 목록, 필터, 검색
- Phase 2: 대시보드 차트, 통계

---

## Acceptance Criteria

### 필수 조건

- [ ] **AC1**: React + TypeScript 프로젝트 초기화
  - Vite 사용 (빠른 빌드)
  - TypeScript 설정
  - ESLint, Prettier

- [ ] **AC2**: 대시보드 홈 페이지 구현
  - 네비게이션 바
  - 리포트 카드 그리드
  - 로딩 상태 표시

- [ ] **AC3**: 리포트 카드 컴포넌트
  - 제목, 날짜, 시나리오명
  - 요약 미리보기 (200자)
  - 상세 페이지 링크

- [ ] **AC4**: API 클라이언트 (axios)
  - 리포트 목록 조회
  - 에러 처리
  - 로딩 상태 관리

- [ ] **AC5**: 필터 및 검색 기능
  - 시나리오별 필터 (드롭다운)
  - 날짜 범위 검색 (date picker)
  - 키워드 검색

- [ ] **AC6**: 반응형 디자인
  - 모바일 (< 768px)
  - 태블릿 (768-1024px)
  - 데스크톱 (> 1024px)

---

## Tasks

### 1. React 프로젝트 초기화

```bash
# Vite로 프로젝트 생성
npm create vite@latest frontend -- --template react-ts

cd frontend
npm install

# 추가 의존성
npm install axios react-router-dom
npm install @tanstack/react-query
npm install tailwindcss postcss autoprefixer
npm install date-fns
npm install lucide-react  # 아이콘
```

**Tailwind CSS 설정**:
```bash
npx tailwindcss init -p
```

**tailwind.config.js**:
```js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

**src/index.css**:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### 2. 프로젝트 구조

```
frontend/
├── src/
│   ├── components/
│   │   ├── Dashboard.tsx
│   │   ├── ReportCard.tsx
│   │   ├── Navbar.tsx
│   │   ├── FilterBar.tsx
│   │   └── LoadingSpinner.tsx
│   ├── services/
│   │   └── api.ts
│   ├── types/
│   │   └── report.ts
│   ├── App.tsx
│   └── main.tsx
├── package.json
└── vite.config.ts
```

### 3. TypeScript 타입 정의

**src/types/report.ts**:
```typescript
// src/types/report.ts
export interface Report {
  id: number;
  scenario_id: string;
  scenario_name: string;
  report_path: string;
  created_at: string;
  event_context?: {
    event_name: string;
    article_title?: string;
    [key: string]: any;
  };
  summary?: string;
}

export interface ReportListResponse {
  total: number;
  page: number;
  page_size: number;
  reports: Report[];
}

export interface ReportFilters {
  scenario_id?: string;
  page: number;
  page_size: number;
}
```

### 4. API 클라이언트

**src/services/api.ts**:
```typescript
// src/services/api.ts
import axios from 'axios';
import { ReportListResponse, ReportFilters, Report } from '../types/report';

const API_BASE_URL = import.meta.env.VITE_API_URL || 'http://localhost:8000';

const apiClient = axios.create({
  baseURL: API_BASE_URL,
  headers: {
    'Content-Type': 'application/json',
  },
});

export const reportApi = {
  /**
   * 리포트 목록 조회
   */
  getReports: async (filters: ReportFilters): Promise<ReportListResponse> => {
    const response = await apiClient.get<ReportListResponse>('/reports', {
      params: filters,
    });
    return response.data;
  },

  /**
   * 리포트 상세 조회
   */
  getReport: async (id: number): Promise<Report> => {
    const response = await apiClient.get<Report>(`/reports/${id}`);
    return response.data;
  },
};

export default apiClient;
```

**.env.local**:
```
VITE_API_URL=http://localhost:8000
```

### 5. ReportCard 컴포넌트

**src/components/ReportCard.tsx**:
```typescript
// src/components/ReportCard.tsx
import { format } from 'date-fns';
import { FileText, Calendar, Tag } from 'lucide-react';
import { Report } from '../types/report';

interface ReportCardProps {
  report: Report;
  onClick: () => void;
}

export default function ReportCard({ report, onClick }: ReportCardProps) {
  const formattedDate = format(new Date(report.created_at), 'yyyy-MM-dd HH:mm');

  return (
    <div
      onClick={onClick}
      className="bg-white rounded-lg shadow hover:shadow-lg transition-shadow cursor-pointer p-6 border border-gray-200"
    >
      {/* 헤더 */}
      <div className="flex items-start justify-between mb-3">
        <div className="flex items-center gap-2">
          <FileText className="w-5 h-5 text-blue-600" />
          <h3 className="text-lg font-semibold text-gray-900 truncate">
            {report.scenario_name}
          </h3>
        </div>
      </div>

      {/* 이벤트 컨텍스트 */}
      {report.event_context && (
        <div className="mb-3 p-2 bg-yellow-50 rounded border-l-4 border-yellow-400">
          <p className="text-sm text-yellow-800">
            🔔 이벤트: {report.event_context.event_name}
          </p>
        </div>
      )}

      {/* 요약 */}
      {report.summary && (
        <p className="text-gray-600 text-sm mb-4 line-clamp-3">
          {report.summary}
        </p>
      )}

      {/* 메타 정보 */}
      <div className="flex items-center gap-4 text-sm text-gray-500">
        <div className="flex items-center gap-1">
          <Calendar className="w-4 h-4" />
          <span>{formattedDate}</span>
        </div>

        <div className="flex items-center gap-1">
          <Tag className="w-4 h-4" />
          <span className="px-2 py-0.5 bg-blue-100 text-blue-800 rounded">
            {report.scenario_id}
          </span>
        </div>
      </div>
    </div>
  );
}
```

### 6. FilterBar 컴포넌트

**src/components/FilterBar.tsx**:
```typescript
// src/components/FilterBar.tsx
import { Search } from 'lucide-react';

interface FilterBarProps {
  scenarioId?: string;
  onScenarioChange: (scenarioId: string) => void;
}

const SCENARIOS = [
  { id: '', name: '전체' },
  { id: 'semiconductor-weekly', name: '반도체 주간' },
  { id: 'interest-rate-impact', name: '금리 영향' },
  { id: 'energy-sector-analysis', name: '에너지 섹터' },
  // ... 추가 시나리오
];

export default function FilterBar({ scenarioId, onScenarioChange }: FilterBarProps) {
  return (
    <div className="bg-white p-4 rounded-lg shadow mb-6">
      <div className="flex flex-col md:flex-row gap-4">
        {/* 시나리오 필터 */}
        <div className="flex-1">
          <label className="block text-sm font-medium text-gray-700 mb-1">
            시나리오
          </label>
          <select
            value={scenarioId}
            onChange={(e) => onScenarioChange(e.target.value)}
            className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            {SCENARIOS.map((scenario) => (
              <option key={scenario.id} value={scenario.id}>
                {scenario.name}
              </option>
            ))}
          </select>
        </div>

        {/* 검색 (향후 구현) */}
        <div className="flex-1">
          <label className="block text-sm font-medium text-gray-700 mb-1">
            검색
          </label>
          <div className="relative">
            <Search className="absolute left-3 top-2.5 w-5 h-5 text-gray-400" />
            <input
              type="text"
              placeholder="리포트 검색..."
              className="w-full pl-10 pr-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
              disabled
            />
          </div>
        </div>
      </div>
    </div>
  );
}
```

### 7. Dashboard 컴포넌트

**src/components/Dashboard.tsx**:
```typescript
// src/components/Dashboard.tsx
import { useState } from 'react';
import { useQuery } from '@tanstack/react-query';
import { useNavigate } from 'react-router-dom';
import { reportApi } from '../services/api';
import ReportCard from './ReportCard';
import FilterBar from './FilterBar';
import LoadingSpinner from './LoadingSpinner';

export default function Dashboard() {
  const navigate = useNavigate();
  const [filters, setFilters] = useState({
    scenario_id: '',
    page: 1,
    page_size: 20,
  });

  const { data, isLoading, isError } = useQuery({
    queryKey: ['reports', filters],
    queryFn: () => reportApi.getReports(filters),
  });

  if (isLoading) {
    return <LoadingSpinner />;
  }

  if (isError) {
    return (
      <div className="flex items-center justify-center h-64">
        <p className="text-red-600">리포트를 불러오는데 실패했습니다.</p>
      </div>
    );
  }

  return (
    <div className="max-w-7xl mx-auto px-4 py-8">
      {/* 헤더 */}
      <div className="mb-6">
        <h1 className="text-3xl font-bold text-gray-900">분석 리포트</h1>
        <p className="text-gray-600 mt-2">
          총 {data?.total || 0}개의 리포트
        </p>
      </div>

      {/* 필터 */}
      <FilterBar
        scenarioId={filters.scenario_id}
        onScenarioChange={(scenarioId) =>
          setFilters({ ...filters, scenario_id: scenarioId, page: 1 })
        }
      />

      {/* 리포트 그리드 */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        {data?.reports.map((report) => (
          <ReportCard
            key={report.id}
            report={report}
            onClick={() => navigate(`/reports/${report.id}`)}
          />
        ))}
      </div>

      {/* 빈 상태 */}
      {data?.reports.length === 0 && (
        <div className="text-center py-12">
          <p className="text-gray-500">리포트가 없습니다.</p>
        </div>
      )}

      {/* 페이지네이션 (향후 구현) */}
      {data && data.total > filters.page_size && (
        <div className="mt-8 flex justify-center">
          <p className="text-gray-500 text-sm">
            페이지네이션 (향후 구현)
          </p>
        </div>
      )}
    </div>
  );
}
```

### 8. LoadingSpinner 컴포넌트

**src/components/LoadingSpinner.tsx**:
```typescript
// src/components/LoadingSpinner.tsx
export default function LoadingSpinner() {
  return (
    <div className="flex items-center justify-center h-64">
      <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600" />
    </div>
  );
}
```

### 9. App 및 라우터 설정

**src/App.tsx**:
```typescript
// src/App.tsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import Dashboard from './components/Dashboard';
import Navbar from './components/Navbar';

const queryClient = new QueryClient();

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <BrowserRouter>
        <div className="min-h-screen bg-gray-50">
          <Navbar />
          <Routes>
            <Route path="/" element={<Dashboard />} />
            <Route path="/reports/:id" element={<div>리포트 상세 (Story 3)</div>} />
          </Routes>
        </div>
      </BrowserRouter>
    </QueryClientProvider>
  );
}
```

**src/components/Navbar.tsx**:
```typescript
// src/components/Navbar.tsx
import { Link } from 'react-router-dom';
import { BarChart3 } from 'lucide-react';

export default function Navbar() {
  return (
    <nav className="bg-white shadow">
      <div className="max-w-7xl mx-auto px-4 py-4">
        <div className="flex items-center justify-between">
          <Link to="/" className="flex items-center gap-2">
            <BarChart3 className="w-8 h-8 text-blue-600" />
            <span className="text-xl font-bold text-gray-900">
              Analysis Economy
            </span>
          </Link>

          <div className="flex items-center gap-4">
            <Link
              to="/"
              className="text-gray-700 hover:text-blue-600 transition"
            >
              대시보드
            </Link>
          </div>
        </div>
      </div>
    </nav>
  );
}
```

### 10. 개발 서버 실행

```bash
# 프론트엔드
cd frontend
npm run dev

# 백엔드 (별도 터미널)
cd backend
uvicorn api.main:app --reload
```

브라우저에서 http://localhost:5173 접속

---

## Technical Notes

### Vite vs CRA
- **Vite**: 빠른 HMR, 모던 빌드, ESM
- **CRA**: 안정적, 커뮤니티 크지만 느림

Vite 권장

### React Query
- 서버 상태 관리
- 캐싱, 재시도, 무효화 자동
- useQuery, useMutation

### Tailwind CSS
- 유틸리티 퍼스트
- 빠른 개발, 작은 번들
- 커스터마이징 쉬움

---

## Definition of Done

- [ ] React 프로젝트 초기화 완료
- [ ] Dashboard 페이지 구현
- [ ] ReportCard 컴포넌트 완성
- [ ] API 클라이언트 통합
- [ ] 필터링 동작 확인
- [ ] 반응형 디자인 테스트
- [ ] 문서 업데이트
- [ ] AC 모두 충족

---

## Dependencies

**Prerequisite**:
- Story 1 (FastAPI 백엔드) 완료

**Blocks**:
- Story 3 (리포트 뷰어) - 이 스토리가 목록 담당

---

## Estimated Time

- **프로젝트 초기화**: 1시간
- **컴포넌트 구현**: 4시간
- **API 통합**: 1.5시간
- **스타일링**: 2시간
- **테스트**: 1.5시간
- **Total**: 8 Story Points

---

## Risks & Mitigation

**Risk 1**: CORS 에러
- **원인**: 백엔드 CORS 설정 누락
- **완화**: FastAPI CORS 미들웨어 확인

**Risk 2**: 모바일 레이아웃 깨짐
- **원인**: Tailwind 반응형 클래스 미사용
- **완화**: 모바일 우선 디자인, 테스트

---

## Success Metrics

- ✅ 페이지 로딩 < 2초
- ✅ 모바일 정상 표시
- ✅ API 에러율 < 1%

---

## References

- [Vite 문서](https://vitejs.dev/)
- [React Query 문서](https://tanstack.com/query/latest)
- [Tailwind CSS 문서](https://tailwindcss.com/)
