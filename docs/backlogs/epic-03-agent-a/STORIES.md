# Epic 3: Agent A 초기 - User Stories 개요

## Story 목록

### Story 1: Cron 스케줄링 설정 및 정기 실행
**Points**: 3 | **Priority**: Must

**User Story**:
As a 바쁜 투자자, I want 매주 월요일 오전 9시 자동으로 분석 실행, So that 수동 실행을 잊어버리는 일이 없다

**Acceptance Criteria**:
- [ ] crontab에 스케줄 등록 (0 9 * * MON)
- [ ] 스케줄러 스크립트 (`agents/monitor/scheduler.py`) 구현
- [ ] 4주 연속 정상 실행 검증
- [ ] 실행 로그 파일 생성

**Tasks**:
1. `agents/monitor/scheduler.py` 구현
   - 시나리오 실행 로직
   - 실행 시작/종료 로깅
2. crontab 설정
   ```
   0 9 * * MON cd /path/to/project && /path/to/venv/bin/python agents/monitor/scheduler.py --scenario semiconductor-weekly
   ```
3. 로그 디렉토리 생성 (`logs/`)
4. 테스트 실행 (수동)
5. 4주 모니터링

---

### Story 2: 슬랙 웹훅 통합 및 알림 포맷 ✅
**Points**: 5 | **Priority**: Must
- [상세 문서](./story-02-slack-integration.md)
- 슬랙 Incoming Webhook 생성
- NotifierBase 클래스 설계
- 리포트 메타데이터 추출 및 알림 전송

---

### Story 3: 실행 로그 및 에러 핸들링 (이메일 Fallback)
**Points**: 5 | **Priority**: Must

**User Story**:
As a 시스템 관리자, I want 실행 실패 시 로그 및 이메일 알림, So that 문제를 빠르게 파악하고 대응할 수 있다

**Acceptance Criteria**:
- [ ] 실행 히스토리 로그 파일 저장
- [ ] 에러 발생 시 상세 로그 기록
- [ ] 3회 재시도 로직 (지수 백오프)
- [ ] 최종 실패 시 이메일 알림
- [ ] 로그 파일 30일 후 자동 삭제

**Tasks**:
1. 로깅 시스템 구축
   - Python `logging` 모듈 설정
   - 일별 로그 파일 (`logs/execution_YYYYMMDD.log`)
   - 에러 전용 로그 (`logs/errors_YYYYMMDD.log`)
2. 재시도 로직 구현
   ```python
   def run_with_retry(scenario, max_retries=3):
       for attempt in range(max_retries):
           try:
               return scenario.run()
           except Exception as e:
               logger.error(f"Attempt {attempt + 1} failed: {e}")
               if attempt < max_retries - 1:
                   time.sleep(2 ** attempt)
       raise Exception("All retries failed")
   ```
3. EmailNotifier 구현
   - SMTP 설정 (Gmail 또는 SendGrid)
   - 에러 알림 이메일 전송
4. 로그 정리 스크립트
   - 30일 이상 로그 삭제 (cron)
5. 테스트
   - 의도적 에러 발생 → 재시도 → 이메일 알림 확인

**Notes**:
- 이메일 알림은 최후 수단 (슬랙 실패 + 실행 실패)
- 로그는 디버깅의 핵심, 상세하게 기록

---

## Epic 3 완료 기준

- [ ] 모든 Story (1-3) 완료
- [ ] 4주 연속 정기 실행 성공
- [ ] 슬랙 알림 도달률 100%
- [ ] 실행 실패 시 이메일 알림 동작 검증
- [ ] 로그에서 히스토리 확인 가능

---

**다음 단계**: Epic 4 (이벤트 감지) 또는 Epic 5 (UI)
