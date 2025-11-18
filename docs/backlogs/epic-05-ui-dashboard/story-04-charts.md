# Story 4: 히스토리 차트 및 비교 기능

**Epic**: Epic 5 - 사용자 인터페이스
**Points**: 3
**Priority**: Could
**Status**: Todo

---

## User Story

**As a** 투자자
**I want** 섹터 점수 추이 차트
**So that** 트렌드를 시각적으로 파악할 수 있다

---

## Background & Context

### 문제 정의
- Story 1-3에서 리포트 텍스트만 제공
- 시간에 따른 변화 추이 파악 어려움
- 예: 반도체 섹터 점수가 지난 8주간 어떻게 변했는지?

### 솔루션
- 시계열 차트로 지표 추이 시각화
- 여러 지표 동시 비교
- 인터랙티브 차트 (확대, 툴팁)

### 범위
- Phase 1: 선 차트, 막대 차트
- Phase 2: 히트맵, 상관관계 매트릭스

---

## Acceptance Criteria

### 필수 조건

- [ ] **AC1**: 시계열 선 차트 구현
  - X축: 날짜 (8주)
  - Y축: 지표 값
  - 여러 지표 동시 표시 (멀티 라인)

- [ ] **AC2**: 주간 변동률 막대 차트
  - 전주 대비 변동률
  - 양수(녹색), 음수(빨강) 색상

- [ ] **AC3**: 날짜 범위 선택기
  - 1개월, 3개월, 6개월, 전체
  - 커스텀 날짜 범위 (선택)

- [ ] **AC4**: 지표 선택 체크박스
  - SMH (반도체 ETF)
  - NVDA (엔비디아)
  - TSM (TSMC)
  - ... 기타 지표

- [ ] **AC5**: 차트 인터랙션
  - 툴팁 (마우스 오버)
  - 줌 인/아웃 (선택)
  - 데이터 다운로드 (CSV, 선택)

---

## Tasks

### 1. 의존성 추가

```bash
cd frontend
npm install recharts
npm install date-fns
```

### 2. Metrics API 엔드포인트 추가

**backend/api/routers/metrics.py** (새 파일):
```python
# backend/api/routers/metrics.py
from fastapi import APIRouter, Depends, Query
from sqlalchemy.orm import Session
from datetime import datetime, timedelta
from typing import Optional
from backend.database import get_db
from backend.models.report import Metric

router = APIRouter(prefix="/metrics", tags=["Metrics"])

@router.get("/")
def get_metrics(
    metric_name: Optional[str] = None,
    start_date: Optional[str] = None,
    end_date: Optional[str] = None,
    db: Session = Depends(get_db)
):
    """시계열 지표 조회"""
    query = db.query(Metric)

    # 필터링
    if metric_name:
        query = query.filter(Metric.metric_name == metric_name)

    if start_date:
        query = query.filter(Metric.metric_date >= start_date)

    if end_date:
        query = query.filter(Metric.metric_date <= end_date)

    # 날짜 순 정렬
    metrics = query.order_by(Metric.metric_date.asc()).all()

    # 그룹화 (metric_name별)
    result = {}
    for metric in metrics:
        if metric.metric_name not in result:
            result[metric.metric_name] = []

        result[metric.metric_name].append({
            'date': metric.metric_date.isoformat(),
            'value': metric.metric_value
        })

    return result
```

**backend/api/main.py 수정**:
```python
from backend.api.routers import reports, metrics

app.include_router(reports.router)
app.include_router(metrics.router)
```

**frontend/src/services/api.ts 수정**:
```typescript
export interface MetricDataPoint {
  date: string;
  value: number;
}

export interface MetricsResponse {
  [metricName: string]: MetricDataPoint[];
}

export const metricsApi = {
  /**
   * 시계열 지표 조회
   */
  getMetrics: async (params: {
    metric_name?: string;
    start_date?: string;
    end_date?: string;
  }): Promise<MetricsResponse> => {
    const response = await apiClient.get<MetricsResponse>('/metrics', {
      params,
    });
    return response.data;
  },
};
```

### 3. TimeSeriesChart 컴포넌트

**src/components/TimeSeriesChart.tsx**:
```typescript
// src/components/TimeSeriesChart.tsx
import {
  LineChart,
  Line,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  Legend,
  ResponsiveContainer,
} from 'recharts';
import { format, parseISO } from 'date-fns';

interface TimeSeriesChartProps {
  data: Array<{
    date: string;
    [key: string]: number | string;
  }>;
  metrics: string[];
  colors: string[];
}

export default function TimeSeriesChart({ data, metrics, colors }: TimeSeriesChartProps) {
  // 날짜 포맷팅
  const formattedData = data.map((item) => ({
    ...item,
    dateFormatted: format(parseISO(item.date), 'MM/dd'),
  }));

  return (
    <ResponsiveContainer width="100%" height={400}>
      <LineChart data={formattedData}>
        <CartesianGrid strokeDasharray="3 3" />
        <XAxis
          dataKey="dateFormatted"
          tick={{ fontSize: 12 }}
        />
        <YAxis tick={{ fontSize: 12 }} />
        <Tooltip
          contentStyle={{
            backgroundColor: 'white',
            border: '1px solid #ccc',
            borderRadius: '4px',
          }}
          labelFormatter={(label) => `날짜: ${label}`}
        />
        <Legend />

        {metrics.map((metric, index) => (
          <Line
            key={metric}
            type="monotone"
            dataKey={metric}
            stroke={colors[index % colors.length]}
            strokeWidth={2}
            dot={{ r: 4 }}
            activeDot={{ r: 6 }}
          />
        ))}
      </LineChart>
    </ResponsiveContainer>
  );
}
```

### 4. ChangeBarChart 컴포넌트

**src/components/ChangeBarChart.tsx**:
```typescript
// src/components/ChangeBarChart.tsx
import {
  BarChart,
  Bar,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  ResponsiveContainer,
  Cell,
} from 'recharts';

interface ChangeBarChartProps {
  data: Array<{
    date: string;
    change: number;
  }>;
}

export default function ChangeBarChart({ data }: ChangeBarChartProps) {
  return (
    <ResponsiveContainer width="100%" height={300}>
      <BarChart data={data}>
        <CartesianGrid strokeDasharray="3 3" />
        <XAxis dataKey="date" tick={{ fontSize: 12 }} />
        <YAxis tick={{ fontSize: 12 }} />
        <Tooltip
          contentStyle={{
            backgroundColor: 'white',
            border: '1px solid #ccc',
            borderRadius: '4px',
          }}
          formatter={(value: number) => `${value.toFixed(2)}%`}
        />
        <Bar dataKey="change">
          {data.map((entry, index) => (
            <Cell
              key={`cell-${index}`}
              fill={entry.change >= 0 ? '#10b981' : '#ef4444'}
            />
          ))}
        </Bar>
      </BarChart>
    </ResponsiveContainer>
  );
}
```

### 5. HistoryChart 페이지

**src/pages/HistoryChart.tsx**:
```typescript
// src/pages/HistoryChart.tsx
import { useState } from 'react';
import { useQuery } from '@tanstack/react-query';
import { subMonths, format } from 'date-fns';
import { metricsApi } from '../services/api';
import TimeSeriesChart from '../components/TimeSeriesChart';
import ChangeBarChart from '../components/ChangeBarChart';
import LoadingSpinner from '../components/LoadingSpinner';

const AVAILABLE_METRICS = [
  { id: 'SMH', name: 'SMH (반도체 ETF)', color: '#3b82f6' },
  { id: 'NVDA', name: 'NVDA (엔비디아)', color: '#10b981' },
  { id: 'TSM', name: 'TSM (TSMC)', color: '#f59e0b' },
];

const DATE_RANGES = [
  { label: '1개월', months: 1 },
  { label: '3개월', months: 3 },
  { label: '6개월', months: 6 },
];

export default function HistoryChart() {
  const [selectedMetrics, setSelectedMetrics] = useState<string[]>(['SMH']);
  const [selectedRange, setSelectedRange] = useState(3); // 3개월

  const startDate = format(subMonths(new Date(), selectedRange), 'yyyy-MM-dd');
  const endDate = format(new Date(), 'yyyy-MM-dd');

  const { data, isLoading } = useQuery({
    queryKey: ['metrics', startDate, endDate],
    queryFn: () => metricsApi.getMetrics({ start_date: startDate, end_date: endDate }),
  });

  const toggleMetric = (metricId: string) => {
    setSelectedMetrics((prev) =>
      prev.includes(metricId)
        ? prev.filter((m) => m !== metricId)
        : [...prev, metricId]
    );
  };

  if (isLoading) {
    return <LoadingSpinner />;
  }

  // 데이터 변환 (Recharts 포맷)
  const chartData = data ? transformData(data, selectedMetrics) : [];
  const changeData = data ? calculateChanges(data, selectedMetrics[0]) : [];

  return (
    <div className="max-w-7xl mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold text-gray-900 mb-6">지표 추이 분석</h1>

      {/* 컨트롤 */}
      <div className="bg-white rounded-lg shadow p-6 mb-6">
        {/* 날짜 범위 */}
        <div className="mb-4">
          <label className="block text-sm font-medium text-gray-700 mb-2">
            기간
          </label>
          <div className="flex gap-2">
            {DATE_RANGES.map((range) => (
              <button
                key={range.months}
                onClick={() => setSelectedRange(range.months)}
                className={`px-4 py-2 rounded ${
                  selectedRange === range.months
                    ? 'bg-blue-600 text-white'
                    : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
                }`}
              >
                {range.label}
              </button>
            ))}
          </div>
        </div>

        {/* 지표 선택 */}
        <div>
          <label className="block text-sm font-medium text-gray-700 mb-2">
            지표
          </label>
          <div className="flex flex-wrap gap-3">
            {AVAILABLE_METRICS.map((metric) => (
              <label
                key={metric.id}
                className="flex items-center gap-2 cursor-pointer"
              >
                <input
                  type="checkbox"
                  checked={selectedMetrics.includes(metric.id)}
                  onChange={() => toggleMetric(metric.id)}
                  className="w-4 h-4 text-blue-600 rounded"
                />
                <span className="text-sm text-gray-700">{metric.name}</span>
              </label>
            ))}
          </div>
        </div>
      </div>

      {/* 선 차트 */}
      <div className="bg-white rounded-lg shadow p-6 mb-6">
        <h2 className="text-xl font-semibold text-gray-900 mb-4">시계열 추이</h2>
        <TimeSeriesChart
          data={chartData}
          metrics={selectedMetrics}
          colors={AVAILABLE_METRICS.map((m) => m.color)}
        />
      </div>

      {/* 변동률 막대 차트 */}
      {selectedMetrics.length === 1 && (
        <div className="bg-white rounded-lg shadow p-6">
          <h2 className="text-xl font-semibold text-gray-900 mb-4">주간 변동률</h2>
          <ChangeBarChart data={changeData} />
        </div>
      )}
    </div>
  );
}

// 데이터 변환 헬퍼
function transformData(
  metricsData: any,
  selectedMetrics: string[]
): Array<{ date: string; [key: string]: number | string }> {
  const allDates = new Set<string>();

  // 모든 날짜 수집
  selectedMetrics.forEach((metric) => {
    if (metricsData[metric]) {
      metricsData[metric].forEach((item: any) => allDates.add(item.date));
    }
  });

  // 날짜별로 데이터 병합
  const result = Array.from(allDates)
    .sort()
    .map((date) => {
      const row: any = { date };

      selectedMetrics.forEach((metric) => {
        const dataPoint = metricsData[metric]?.find((d: any) => d.date === date);
        row[metric] = dataPoint ? dataPoint.value : null;
      });

      return row;
    });

  return result;
}

// 주간 변동률 계산
function calculateChanges(metricsData: any, metric: string) {
  if (!metricsData[metric]) return [];

  const data = metricsData[metric];
  const changes = [];

  for (let i = 1; i < data.length; i++) {
    const prev = data[i - 1].value;
    const curr = data[i].value;
    const change = ((curr - prev) / prev) * 100;

    changes.push({
      date: format(new Date(data[i].date), 'MM/dd'),
      change,
    });
  }

  return changes;
}
```

### 6. 라우터 추가

**src/App.tsx 수정**:
```typescript
import HistoryChart from './pages/HistoryChart';

<Routes>
  <Route path="/" element={<Dashboard />} />
  <Route path="/reports/:id" element={<ReportDetail />} />
  <Route path="/charts" element={<HistoryChart />} />
</Routes>
```

**src/components/Navbar.tsx 수정**:
```typescript
<Link
  to="/charts"
  className="text-gray-700 hover:text-blue-600 transition"
>
  차트
</Link>
```

---

## Technical Notes

### Recharts vs Chart.js
- **Recharts**: React 네이티브, 선언적, 반응형
- **Chart.js**: 성능 좋음, 커뮤니티 크지만 React 통합 복잡

Recharts 권장

### 시계열 데이터 저장
- Story 1의 Metric 모델 활용
- 리포트 생성 시 지표도 DB 저장
- BaseScenario에 `_save_metrics()` 추가 (향후)

---

## Definition of Done

- [ ] Metrics API 엔드포인트 완료
- [ ] TimeSeriesChart 구현
- [ ] ChangeBarChart 구현
- [ ] HistoryChart 페이지 완성
- [ ] 날짜 범위 선택 동작 확인
- [ ] 문서 업데이트
- [ ] AC 모두 충족

---

## Dependencies

**Prerequisite**:
- Story 1 (FastAPI 백엔드, Metric 모델) 완료

**Blocks**:
- 없음 (선택 기능)

---

## Estimated Time

- **Metrics API**: 0.5시간
- **차트 컴포넌트**: 1.5시간
- **페이지 구현**: 1시간
- **테스트**: 0.5시간
- **Total**: 3 Story Points

---

## Risks & Mitigation

**Risk 1**: 데이터 부족
- **원인**: 초기에는 리포트 몇 개뿐
- **완화**: 샘플 데이터 생성, 과거 데이터 백필

---

## Success Metrics

- ✅ 차트 렌더링 < 1초
- ✅ 인터랙션 부드러움
- ✅ 모바일 반응형

---

## References

- [Recharts 문서](https://recharts.org/)
- Story 1: FastAPI 백엔드
