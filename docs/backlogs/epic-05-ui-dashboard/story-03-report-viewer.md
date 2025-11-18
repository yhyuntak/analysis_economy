# Story 3: 리포트 뷰어 및 Markdown 렌더링

**Epic**: Epic 5 - 사용자 인터페이스
**Points**: 5
**Priority**: Should
**Status**: Todo

---

## User Story

**As a** 사용자
**I want** 리포트 상세 페이지에서 전체 내용 확인
**So that** Markdown 파일을 직접 열지 않아도 된다

---

## Background & Context

### 문제 정의
- Story 2에서 리포트 목록만 구현
- 실제 리포트 내용은 여전히 파일로 열어야 함
- Markdown 렌더링, 네비게이션 필요

### 솔루션
- Markdown → HTML 변환 및 스타일링
- 목차 (TOC) 자동 생성
- 이전/다음 리포트 네비게이션

### 범위
- Phase 1: 기본 뷰어, TOC, 네비게이션
- Phase 2: PDF 다운로드, 인쇄 최적화

---

## Acceptance Criteria

### 필수 조건

- [ ] **AC1**: 리포트 상세 페이지 구현
  - URL: `/reports/:id`
  - 제목, 날짜, 메타데이터 표시
  - Markdown 본문 렌더링

- [ ] **AC2**: Markdown → HTML 렌더링
  - react-markdown 사용
  - 코드 블록 하이라이팅
  - 표, 리스트 스타일링

- [ ] **AC3**: 목차 (TOC) 자동 생성
  - H2, H3 헤딩 추출
  - 스크롤 시 자동 하이라이트
  - 클릭 시 해당 섹션 이동

- [ ] **AC4**: 이전/다음 리포트 네비게이션
  - 날짜 순서 기반
  - 키보드 단축키 (←, →)

- [ ] **AC5**: 인쇄 최적화 (선택)
  - 인쇄 시 깔끔한 레이아웃
  - 목차 페이지 구분

---

## Tasks

### 1. 의존성 추가

```bash
cd frontend
npm install react-markdown
npm install remark-gfm  # GitHub Flavored Markdown
npm install react-syntax-highlighter
npm install @types/react-syntax-highlighter
```

### 2. Markdown 콘텐츠 API 엔드포인트 추가

**backend/api/routers/reports.py 수정**:
```python
from fastapi import APIRouter
from pathlib import Path

@router.get("/{report_id}/content")
def get_report_content(report_id: int, db: Session = Depends(get_db)):
    """리포트 Markdown 콘텐츠 조회"""
    report = db.query(Report).filter(Report.id == report_id).first()

    if not report:
        raise HTTPException(status_code=404, detail="Report not found")

    # 파일 읽기
    report_file = Path(report.report_path)

    if not report_file.exists():
        raise HTTPException(status_code=404, detail="Report file not found")

    with open(report_file, 'r', encoding='utf-8') as f:
        content = f.read()

    return {
        "report_id": report.id,
        "content": content
    }
```

**frontend/src/services/api.ts 수정**:
```typescript
export const reportApi = {
  // ... 기존 메서드

  /**
   * 리포트 Markdown 콘텐츠 조회
   */
  getReportContent: async (id: number): Promise<string> => {
    const response = await apiClient.get<{ content: string }>(
      `/reports/${id}/content`
    );
    return response.data.content;
  },
};
```

### 3. TableOfContents 컴포넌트

**src/components/TableOfContents.tsx**:
```typescript
// src/components/TableOfContents.tsx
import { useEffect, useState } from 'react';

interface Heading {
  id: string;
  text: string;
  level: number;
}

interface TableOfContentsProps {
  content: string;
}

export default function TableOfContents({ content }: TableOfContentsProps) {
  const [headings, setHeadings] = useState<Heading[]>([]);
  const [activeId, setActiveId] = useState<string>('');

  useEffect(() => {
    // Markdown에서 헤딩 추출
    const headingRegex = /^(#{2,3})\s+(.+)$/gm;
    const matches = [...content.matchAll(headingRegex)];

    const extractedHeadings = matches.map((match, index) => {
      const level = match[1].length;
      const text = match[2];
      const id = `heading-${index}`;

      return { id, text, level };
    });

    setHeadings(extractedHeadings);
  }, [content]);

  useEffect(() => {
    // 스크롤 시 활성 헤딩 업데이트
    const handleScroll = () => {
      const headingElements = headings.map((h) =>
        document.getElementById(h.id)
      );

      for (let i = headingElements.length - 1; i >= 0; i--) {
        const element = headingElements[i];
        if (element && element.getBoundingClientRect().top < 100) {
          setActiveId(headings[i].id);
          break;
        }
      }
    };

    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, [headings]);

  const scrollToHeading = (id: string) => {
    const element = document.getElementById(id);
    if (element) {
      element.scrollIntoView({ behavior: 'smooth', block: 'start' });
    }
  };

  return (
    <nav className="sticky top-4 bg-white rounded-lg shadow p-4 max-h-[calc(100vh-2rem)] overflow-y-auto">
      <h3 className="font-semibold text-gray-900 mb-3">목차</h3>
      <ul className="space-y-2">
        {headings.map((heading) => (
          <li
            key={heading.id}
            className={`
              ${heading.level === 3 ? 'ml-4' : ''}
              ${activeId === heading.id ? 'text-blue-600 font-medium' : 'text-gray-600'}
            `}
          >
            <button
              onClick={() => scrollToHeading(heading.id)}
              className="text-left hover:text-blue-600 transition text-sm"
            >
              {heading.text}
            </button>
          </li>
        ))}
      </ul>
    </nav>
  );
}
```

### 4. MarkdownRenderer 컴포넌트

**src/components/MarkdownRenderer.tsx**:
```typescript
// src/components/MarkdownRenderer.tsx
import ReactMarkdown from 'react-markdown';
import remarkGfm from 'remark-gfm';
import { Prism as SyntaxHighlighter } from 'react-syntax-highlighter';
import { vscDarkPlus } from 'react-syntax-highlighter/dist/esm/styles/prism';

interface MarkdownRendererProps {
  content: string;
}

export default function MarkdownRenderer({ content }: MarkdownRendererProps) {
  // 헤딩에 ID 추가 (TOC 연동)
  let headingIndex = 0;

  return (
    <div className="prose prose-lg max-w-none">
      <ReactMarkdown
        remarkPlugins={[remarkGfm]}
        components={{
          // 헤딩 커스터마이징
          h2: ({ node, children, ...props }) => {
            const id = `heading-${headingIndex++}`;
            return (
              <h2 id={id} className="scroll-mt-20" {...props}>
                {children}
              </h2>
            );
          },
          h3: ({ node, children, ...props }) => {
            const id = `heading-${headingIndex++}`;
            return (
              <h3 id={id} className="scroll-mt-20" {...props}>
                {children}
              </h3>
            );
          },

          // 코드 블록
          code({ node, inline, className, children, ...props }) {
            const match = /language-(\w+)/.exec(className || '');

            return !inline && match ? (
              <SyntaxHighlighter
                style={vscDarkPlus}
                language={match[1]}
                PreTag="div"
                {...props}
              >
                {String(children).replace(/\n$/, '')}
              </SyntaxHighlighter>
            ) : (
              <code
                className="bg-gray-100 text-red-600 px-1 py-0.5 rounded"
                {...props}
              >
                {children}
              </code>
            );
          },

          // 표
          table: ({ node, children, ...props }) => (
            <div className="overflow-x-auto">
              <table className="min-w-full border-collapse" {...props}>
                {children}
              </table>
            </div>
          ),

          // 링크
          a: ({ node, children, ...props }) => (
            <a
              className="text-blue-600 hover:underline"
              target="_blank"
              rel="noopener noreferrer"
              {...props}
            >
              {children}
            </a>
          ),
        }}
      >
        {content}
      </ReactMarkdown>
    </div>
  );
}
```

### 5. ReportDetail 페이지

**src/pages/ReportDetail.tsx**:
```typescript
// src/pages/ReportDetail.tsx
import { useParams, useNavigate } from 'react-router-dom';
import { useQuery } from '@tanstack/react-query';
import { format } from 'date-fns';
import { ArrowLeft, ChevronLeft, ChevronRight, Calendar, Tag } from 'lucide-react';
import { reportApi } from '../services/api';
import MarkdownRenderer from '../components/MarkdownRenderer';
import TableOfContents from '../components/TableOfContents';
import LoadingSpinner from '../components/LoadingSpinner';
import { useEffect } from 'react';

export default function ReportDetail() {
  const { id } = useParams<{ id: string }>();
  const navigate = useNavigate();
  const reportId = parseInt(id || '0');

  // 리포트 메타데이터
  const { data: report, isLoading: isLoadingReport } = useQuery({
    queryKey: ['report', reportId],
    queryFn: () => reportApi.getReport(reportId),
  });

  // 리포트 콘텐츠
  const { data: content, isLoading: isLoadingContent } = useQuery({
    queryKey: ['reportContent', reportId],
    queryFn: () => reportApi.getReportContent(reportId),
  });

  // 키보드 단축키 (←, →)
  useEffect(() => {
    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.key === 'ArrowLeft') {
        // 이전 리포트 (향후 구현)
        console.log('Previous report');
      } else if (e.key === 'ArrowRight') {
        // 다음 리포트 (향후 구현)
        console.log('Next report');
      }
    };

    window.addEventListener('keydown', handleKeyDown);
    return () => window.removeEventListener('keydown', handleKeyDown);
  }, [reportId]);

  if (isLoadingReport || isLoadingContent) {
    return <LoadingSpinner />;
  }

  if (!report || !content) {
    return (
      <div className="flex items-center justify-center h-64">
        <p className="text-red-600">리포트를 불러올 수 없습니다.</p>
      </div>
    );
  }

  const formattedDate = format(new Date(report.created_at), 'yyyy-MM-dd HH:mm');

  return (
    <div className="max-w-7xl mx-auto px-4 py-8">
      {/* 뒤로 가기 */}
      <button
        onClick={() => navigate('/')}
        className="flex items-center gap-2 text-gray-600 hover:text-gray-900 mb-6"
      >
        <ArrowLeft className="w-5 h-5" />
        목록으로
      </button>

      {/* 헤더 */}
      <div className="bg-white rounded-lg shadow p-6 mb-6">
        <h1 className="text-3xl font-bold text-gray-900 mb-3">
          {report.scenario_name}
        </h1>

        <div className="flex flex-wrap items-center gap-4 text-sm text-gray-600">
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

        {report.event_context && (
          <div className="mt-4 p-3 bg-yellow-50 rounded border-l-4 border-yellow-400">
            <p className="text-sm text-yellow-800">
              🔔 이벤트 트리거: {report.event_context.event_name}
            </p>
          </div>
        )}
      </div>

      {/* 콘텐츠 + TOC */}
      <div className="grid grid-cols-1 lg:grid-cols-4 gap-6">
        {/* 목차 (데스크톱만) */}
        <aside className="hidden lg:block">
          <TableOfContents content={content} />
        </aside>

        {/* 본문 */}
        <main className="lg:col-span-3 bg-white rounded-lg shadow p-8">
          <MarkdownRenderer content={content} />
        </main>
      </div>

      {/* 네비게이션 */}
      <div className="flex justify-between items-center mt-8">
        <button
          className="flex items-center gap-2 px-4 py-2 bg-gray-200 text-gray-700 rounded hover:bg-gray-300 transition disabled:opacity-50"
          disabled
        >
          <ChevronLeft className="w-5 h-5" />
          이전 리포트
        </button>

        <button
          className="flex items-center gap-2 px-4 py-2 bg-gray-200 text-gray-700 rounded hover:bg-gray-300 transition disabled:opacity-50"
          disabled
        >
          다음 리포트
          <ChevronRight className="w-5 h-5" />
        </button>
      </div>
    </div>
  );
}
```

### 6. 라우터 업데이트

**src/App.tsx 수정**:
```typescript
import ReportDetail from './pages/ReportDetail';

<Routes>
  <Route path="/" element={<Dashboard />} />
  <Route path="/reports/:id" element={<ReportDetail />} />
</Routes>
```

### 7. Tailwind Typography 플러그인 추가

```bash
npm install @tailwindcss/typography
```

**tailwind.config.js 수정**:
```js
export default {
  // ...
  plugins: [
    require('@tailwindcss/typography'),
  ],
}
```

### 8. 인쇄 최적화 CSS (선택)

**src/index.css 추가**:
```css
@media print {
  /* 목차 숨김 */
  aside {
    display: none !important;
  }

  /* 네비게이션 숨김 */
  nav, button {
    display: none !important;
  }

  /* 본문만 출력 */
  main {
    box-shadow: none !important;
    border: none !important;
  }
}
```

---

## Technical Notes

### react-markdown vs dangerouslySetInnerHTML
- **react-markdown**: 안전, 컴포넌트 기반, 커스터마이징 쉬움
- **dangerouslySetInnerHTML**: XSS 위험

react-markdown 권장

### 코드 하이라이팅
- react-syntax-highlighter
- 다양한 테마 지원
- 성능 최적화 (Prism vs Highlight.js)

### 목차 자동 생성
- Markdown 파싱으로 헤딩 추출
- Intersection Observer (고급)
- 스크롤 이벤트 (간단)

---

## Definition of Done

- [ ] ReportDetail 페이지 구현 완료
- [ ] Markdown 렌더링 확인
- [ ] TOC 동작 확인
- [ ] 코드 하이라이팅 테스트
- [ ] 인쇄 최적화 확인 (선택)
- [ ] 문서 업데이트
- [ ] AC 모두 충족

---

## Dependencies

**Prerequisite**:
- Story 1 (FastAPI 백엔드) 완료
- Story 2 (리포트 목록) 완료

**Blocks**:
- Story 4 (차트) - 독립적

---

## Estimated Time

- **API 엔드포인트**: 0.5시간
- **Markdown 렌더링**: 2시간
- **TOC 구현**: 1.5시간
- **스타일링**: 1시간
- **테스트**: 1시간
- **Total**: 5 Story Points

---

## Risks & Mitigation

**Risk 1**: 대용량 Markdown 렌더링 느림
- **원인**: 복잡한 리포트 (이미지, 표 많음)
- **완화**: 가상화, 레이지 로딩

**Risk 2**: TOC 스크롤 버그
- **원인**: 헤딩 ID 중복
- **완화**: 고유 ID 생성, 테스트

---

## Success Metrics

- ✅ 렌더링 시간 < 1초
- ✅ TOC 정확도 100%
- ✅ 인쇄 레이아웃 깔끔

---

## References

- [react-markdown](https://github.com/remarkjs/react-markdown)
- [Tailwind Typography](https://tailwindcss.com/docs/typography-plugin)
- [react-syntax-highlighter](https://github.com/react-syntax-highlighter/react-syntax-highlighter)
