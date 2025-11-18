# Story 1: RSS/뉴스 피드 모니터링

**Epic**: Epic 4 - Agent A 진화
**Points**: 5
**Priority**: Should
**Status**: Todo

---

## User Story

**As a** Agent A
**I want** RSS 피드를 주기적으로 모니터링
**So that** 주요 경제 이벤트를 자동으로 감지할 수 있다

---

## Background & Context

### 문제 정의
- Agent A는 현재 Cron 스케줄만 실행 (정기 분석)
- 갑작스러운 경제 이벤트에 실시간 대응 불가
- 예: FED 긴급 금리 결정, 지정학 리스크 급발, 기업 주요 공시

### 솔루션
- RSS 피드 모니터링으로 뉴스 자동 수집
- 5-10분 간격 폴링으로 준실시간 감지
- 중복 필터링 및 구조화된 저장

### 범위
- Phase 1: 주요 경제 뉴스 소스 5-10개
- Phase 2+: API 기반 뉴스 (Bloomberg, Reuters API)

---

## Acceptance Criteria

### 필수 조건

- [ ] **AC1**: RSS 피드 파서 구현
  - feedparser 라이브러리 사용
  - 제목, 내용, 게시 시간, URL 추출
  - 파싱 에러 처리

- [ ] **AC2**: 주요 소스 등록 (최소 5개)
  - Federal Reserve (연준 뉴스)
  - Reuters Economics
  - Bloomberg Markets
  - Yahoo Finance
  - MarketWatch

- [ ] **AC3**: 주기적 폴링 (5-10분 간격)
  - APScheduler 사용
  - 백그라운드 실행
  - 로깅 포함

- [ ] **AC4**: 중복 기사 필터링
  - URL 기반 해시 생성
  - 이미 수집한 기사 스킵
  - 24시간 내 중복 체크

- [ ] **AC5**: 24시간 안정 동작 검증
  - 1440회 (24h × 60min / 10min) 폴링 성공
  - 메모리 누수 없음
  - 에러 발생 시 재시작

---

## Tasks

### 1. 프로젝트 구조 준비

```bash
mkdir -p agents/monitor/feeds
mkdir -p config
mkdir -p data/articles
touch agents/monitor/feeds/__init__.py
touch agents/monitor/feeds/feed_watcher.py
touch config/feed_sources.yaml
```

### 2. RSS 소스 설정

**config/feed_sources.yaml**:
```yaml
feeds:
  - name: "Federal Reserve"
    url: "https://www.federalreserve.gov/feeds/press_all.xml"
    category: "central_bank"
    priority: "HIGH"

  - name: "Reuters Economics"
    url: "https://www.reutersagency.com/feed/?taxonomy=best-topics&post_type=best"
    category: "news"
    priority: "HIGH"

  - name: "Bloomberg Markets"
    url: "https://www.bloomberg.com/feed/podcast/etf-report.xml"
    category: "markets"
    priority: "MEDIUM"

  - name: "Yahoo Finance"
    url: "https://finance.yahoo.com/news/rssindex"
    category: "markets"
    priority: "MEDIUM"

  - name: "MarketWatch Economic Report"
    url: "https://www.marketwatch.com/rss/topstories"
    category: "news"
    priority: "MEDIUM"
```

### 3. FeedWatcher 구현

**agents/monitor/feeds/feed_watcher.py**:
```python
# agents/monitor/feeds/feed_watcher.py
import feedparser
import hashlib
import json
import yaml
from datetime import datetime
from pathlib import Path
from typing import List, Dict
from apscheduler.schedulers.background import BackgroundScheduler
from src.utils.logger import setup_logger

logger = setup_logger(__name__)

class FeedWatcher:
    """RSS 피드 모니터링"""

    def __init__(self, config_path: str = "config/feed_sources.yaml"):
        self.config_path = config_path
        self.feeds = self._load_feeds()
        self.seen_hashes = set()
        self.articles_dir = Path("data/articles")
        self.articles_dir.mkdir(parents=True, exist_ok=True)

        # 기존 해시 로드
        self._load_seen_hashes()

    def _load_feeds(self) -> List[Dict]:
        """피드 소스 로드"""
        with open(self.config_path, 'r', encoding='utf-8') as f:
            config = yaml.safe_load(f)
        return config['feeds']

    def _load_seen_hashes(self):
        """이미 본 기사 해시 로드 (24시간 내)"""
        hash_file = self.articles_dir / "seen_hashes.json"

        if hash_file.exists():
            with open(hash_file, 'r') as f:
                data = json.load(f)
                # 24시간 이내 해시만 유지
                cutoff = datetime.now().timestamp() - 86400
                self.seen_hashes = {
                    h for h, ts in data.items()
                    if ts > cutoff
                }
                logger.info(f"Loaded {len(self.seen_hashes)} seen hashes")

    def _save_seen_hashes(self):
        """본 기사 해시 저장"""
        hash_file = self.articles_dir / "seen_hashes.json"

        # 해시와 타임스탬프 저장
        data = {h: datetime.now().timestamp() for h in self.seen_hashes}

        with open(hash_file, 'w') as f:
            json.dump(data, f)

    def _get_article_hash(self, url: str) -> str:
        """기사 URL로 해시 생성"""
        return hashlib.md5(url.encode()).hexdigest()

    def fetch_feed(self, feed_config: Dict) -> List[Dict]:
        """단일 피드 가져오기"""
        try:
            logger.info(f"Fetching feed: {feed_config['name']}")

            feed = feedparser.parse(feed_config['url'])

            if feed.bozo:
                logger.warning(f"Feed parsing warning for {feed_config['name']}: {feed.bozo_exception}")

            articles = []

            for entry in feed.entries[:20]:  # 최신 20개만
                article_hash = self._get_article_hash(entry.link)

                # 중복 체크
                if article_hash in self.seen_hashes:
                    continue

                article = {
                    'hash': article_hash,
                    'title': entry.get('title', ''),
                    'link': entry.link,
                    'published': entry.get('published', ''),
                    'summary': entry.get('summary', ''),
                    'source': feed_config['name'],
                    'category': feed_config['category'],
                    'priority': feed_config['priority'],
                    'fetched_at': datetime.now().isoformat()
                }

                articles.append(article)
                self.seen_hashes.add(article_hash)

            if articles:
                logger.info(f"Found {len(articles)} new articles from {feed_config['name']}")

            return articles

        except Exception as e:
            logger.error(f"Failed to fetch feed {feed_config['name']}: {str(e)}")
            return []

    def fetch_all_feeds(self) -> List[Dict]:
        """모든 피드 가져오기"""
        all_articles = []

        for feed_config in self.feeds:
            articles = self.fetch_feed(feed_config)
            all_articles.extend(articles)

        if all_articles:
            self._save_articles(all_articles)
            self._save_seen_hashes()

        logger.info(f"Total new articles: {len(all_articles)}")
        return all_articles

    def _save_articles(self, articles: List[Dict]):
        """기사 JSON 저장"""
        today = datetime.now().strftime('%Y%m%d')
        output_file = self.articles_dir / f"articles_{today}.json"

        # 기존 기사 로드
        existing = []
        if output_file.exists():
            with open(output_file, 'r', encoding='utf-8') as f:
                existing = json.load(f)

        # 새 기사 추가
        existing.extend(articles)

        # 저장
        with open(output_file, 'w', encoding='utf-8') as f:
            json.dump(existing, f, indent=2, ensure_ascii=False)

        logger.info(f"Saved {len(articles)} articles to {output_file}")

    def start_watching(self, interval_minutes: int = 10):
        """백그라운드 모니터링 시작"""
        scheduler = BackgroundScheduler()

        # 즉시 1회 실행
        self.fetch_all_feeds()

        # 주기적 실행
        scheduler.add_job(
            self.fetch_all_feeds,
            'interval',
            minutes=interval_minutes,
            id='feed_watcher'
        )

        scheduler.start()
        logger.info(f"Feed watcher started (interval: {interval_minutes} minutes)")

        return scheduler

if __name__ == "__main__":
    watcher = FeedWatcher()

    # 테스트: 1회 실행
    articles = watcher.fetch_all_feeds()

    print(f"\n=== Fetched {len(articles)} new articles ===")
    for article in articles[:5]:
        print(f"[{article['source']}] {article['title']}")
```

### 4. 백그라운드 실행 스크립트

**agents/monitor/start_feed_watcher.py**:
```python
# agents/monitor/start_feed_watcher.py
import time
from feeds.feed_watcher import FeedWatcher
from src.utils.logger import setup_logger

logger = setup_logger(__name__)

def main():
    logger.info("Starting Feed Watcher...")

    watcher = FeedWatcher()
    scheduler = watcher.start_watching(interval_minutes=10)

    try:
        # 무한 실행
        while True:
            time.sleep(60)
    except (KeyboardInterrupt, SystemExit):
        logger.info("Shutting down Feed Watcher...")
        scheduler.shutdown()

if __name__ == "__main__":
    main()
```

### 5. 의존성 추가

**requirements.txt 업데이트**:
```
feedparser==6.0.11
APScheduler==3.10.4
PyYAML==6.0.1
```

### 6. 테스트

**수동 테스트**:
```bash
# 1회 실행 테스트
python agents/monitor/feeds/feed_watcher.py

# 결과 확인
cat data/articles/articles_$(date +%Y%m%d).json

# 백그라운드 실행 테스트 (10분 간격)
python agents/monitor/start_feed_watcher.py

# 로그 모니터링
tail -f logs/execution.log
```

**24시간 안정성 테스트**:
```bash
# 백그라운드 실행
nohup python agents/monitor/start_feed_watcher.py > /dev/null 2>&1 &

# 24시간 후 확인
ps aux | grep start_feed_watcher  # 프로세스 살아있는지
wc -l data/articles/articles_*.json  # 기사 수집됐는지
grep ERROR logs/execution.log  # 에러 없는지
```

---

## Technical Notes

### RSS vs API
- **RSS**: 무료, 간단, 지연 5-10분
- **API** (Bloomberg/Reuters): 실시간, 유료, 복잡한 인증

초기에는 RSS로 충분, 나중에 API 추가

### 폴링 간격
- **5분**: 준실시간, API 부하 높음
- **10분**: 권장 (하루 144회 폴링)
- **30분**: 배치성 뉴스만

### 중복 처리
- URL 해시 기반 (MD5)
- 같은 뉴스를 다른 소스가 보도해도 URL 다르면 별도 저장
- 제목 유사도 체크는 Story 2 (이벤트 감지)에서

### 데이터 보관
- JSON 파일 (초기)
- 나중에 DB 마이그레이션 (Epic 5)

---

## Definition of Done

- [ ] FeedWatcher 클래스 구현 완료
- [ ] 5개 이상 RSS 소스 등록
- [ ] APScheduler 통합
- [ ] 중복 필터링 동작 확인
- [ ] 24시간 안정 실행 테스트 성공
- [ ] 로그에서 폴링 이력 확인 가능
- [ ] 기사 JSON 저장 확인
- [ ] 문서 업데이트
- [ ] AC 모두 충족

---

## Dependencies

**Prerequisite**:
- Epic 3 Story 3 (로깅 시스템) 완료
- config/, data/ 디렉토리 존재

**Blocks**:
- Story 2 (이벤트 감지) - 이 스토리가 기사 수집 담당

---

## Estimated Time

- **RSS 소스 조사**: 1시간
- **FeedWatcher 구현**: 3시간
- **테스트**: 1시간
- **24시간 안정성 검증**: 패시브
- **Total**: 5 Story Points

---

## Risks & Mitigation

**Risk 1**: RSS 피드 URL 변경 또는 폐쇄
- **원인**: 뉴스 사이트 정책 변경
- **완화**: 여러 소스 등록, 정기 체크

**Risk 2**: 폴링 빈도로 인한 차단
- **원인**: 사이트에서 봇으로 인식
- **완화**: User-Agent 설정, 10분 간격 준수

**Risk 3**: 메모리 누수
- **원인**: seen_hashes 무한 증가
- **완화**: 24시간 내 해시만 유지, 주기적 정리

---

## Success Metrics

- ✅ 24시간 동안 144회 폴링 성공
- ✅ 수집된 기사 > 50개 (하루 평균)
- ✅ 중복률 < 5%
- ✅ 메모리 사용량 < 100MB

---

## References

- [feedparser 문서](https://feedparser.readthedocs.io/)
- [APScheduler 문서](https://apscheduler.readthedocs.io/)
- [RSS Best Practices](https://www.rssboard.org/rss-specification)
- Epic 3 Story 3: 로깅 시스템
