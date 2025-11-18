# Story 3: 실행 로그 및 에러 핸들링 (이메일 Fallback)

**Epic**: Epic 3 - Agent A 초기
**Points**: 5
**Priority**: Must
**Status**: Todo

---

## User Story

**As a** 시스템 관리자
**I want** 실행 실패 시 로그 및 이메일 알림
**So that** 문제를 빠르게 파악하고 대응할 수 있다

---

## Background & Context

### 문제 정의
- 자동 실행 시스템은 실패해도 사용자가 모를 수 있음
- 일시적 에러 (네트워크 끊김)와 심각한 에러 구분 필요
- 실패 이력 추적으로 시스템 안정성 개선

### 솔루션
- 상세한 로깅 시스템
- 지능적 재시도 로직 (지수 백오프)
- 최종 실패 시 이메일 알림

### 범위
- Python logging 기반 시스템
- 3회 재시도 로직
- SMTP 이메일 알림
- 로그 자동 정리 (30일)

---

## Acceptance Criteria

### 필수 조건

- [ ] **AC1**: 실행 히스토리 로그 파일 저장
  - `logs/execution_YYYYMMDD.log` 포맷
  - 모든 실행 시작/종료 기록
  - 타임스탬프, 시나리오 이름, 상태 (성공/실패) 포함

- [ ] **AC2**: 에러 발생 시 상세 로그 기록
  - 별도 에러 로그 파일 (`logs/errors_YYYYMMDD.log`)
  - 스택 트레이스 전체 포함
  - 에러 발생 시점의 시스템 상태 (메모리, 디스크)

- [ ] **AC3**: 3회 재시도 로직 (지수 백오프)
  - 1차 실패 → 2초 대기 → 재시도
  - 2차 실패 → 4초 대기 → 재시도
  - 3차 실패 → 8초 대기 → 재시도
  - 3회 모두 실패 → 이메일 알림

- [ ] **AC4**: 최종 실패 시 이메일 알림
  - 제목: `[실패] 자동 분석 실행 실패 - {시나리오명} - {날짜}`
  - 본문: 에러 내용, 재시도 횟수, 로그 파일 경로
  - 수신자: 개발자 이메일 (환경 변수)

- [ ] **AC5**: 로그 파일 30일 후 자동 삭제
  - 일별 cron job으로 정리
  - 30일 이전 로그 삭제
  - 에러 로그는 90일 보관

---

## Tasks

### 1. 로깅 시스템 구축

**파일 생성**:
```bash
mkdir -p logs
touch src/utils/logger.py
```

**logger.py 구현**:
```python
# src/utils/logger.py
import logging
import os
from datetime import datetime
from logging.handlers import TimedRotatingFileHandler

def setup_logger(name: str, level=logging.INFO):
    """
    프로젝트 전역 로거 설정

    Args:
        name: 로거 이름 (일반적으로 __name__)
        level: 로그 레벨

    Returns:
        logging.Logger
    """
    logger = logging.getLogger(name)
    logger.setLevel(level)

    # 이미 핸들러 있으면 스킵 (중복 방지)
    if logger.handlers:
        return logger

    # 포맷 정의
    formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s',
        datefmt='%Y-%m-%d %H:%M:%S'
    )

    # 1. 실행 로그 (INFO 이상)
    execution_log = TimedRotatingFileHandler(
        filename='logs/execution.log',
        when='midnight',
        interval=1,
        backupCount=30,  # 30일 보관
        encoding='utf-8'
    )
    execution_log.setLevel(logging.INFO)
    execution_log.setFormatter(formatter)
    execution_log.suffix = "%Y%m%d"

    # 2. 에러 로그 (ERROR 이상)
    error_log = TimedRotatingFileHandler(
        filename='logs/errors.log',
        when='midnight',
        interval=1,
        backupCount=90,  # 90일 보관
        encoding='utf-8'
    )
    error_log.setLevel(logging.ERROR)
    error_log.setFormatter(formatter)
    error_log.suffix = "%Y%m%d"

    # 3. 콘솔 출력
    console = logging.StreamHandler()
    console.setLevel(logging.INFO)
    console.setFormatter(formatter)

    # 핸들러 추가
    logger.addHandler(execution_log)
    logger.addHandler(error_log)
    logger.addHandler(console)

    return logger
```

### 2. 재시도 로직 구현

**파일 생성**:
```bash
touch src/utils/retry.py
```

**retry.py 구현**:
```python
# src/utils/retry.py
import time
import logging
from functools import wraps
from typing import Callable, Any

logger = logging.getLogger(__name__)

def retry_with_backoff(max_retries: int = 3, base_delay: float = 2.0):
    """
    지수 백오프를 사용한 재시도 데코레이터

    Args:
        max_retries: 최대 재시도 횟수
        base_delay: 기본 대기 시간 (초)

    Usage:
        @retry_with_backoff(max_retries=3, base_delay=2.0)
        def risky_function():
            # 실패할 수 있는 작업
            pass
    """
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        def wrapper(*args, **kwargs) -> Any:
            last_exception = None

            for attempt in range(max_retries):
                try:
                    logger.info(f"Attempt {attempt + 1}/{max_retries} for {func.__name__}")
                    result = func(*args, **kwargs)

                    if attempt > 0:
                        logger.info(f"{func.__name__} succeeded after {attempt + 1} attempts")

                    return result

                except Exception as e:
                    last_exception = e
                    logger.error(
                        f"{func.__name__} failed on attempt {attempt + 1}/{max_retries}: {str(e)}"
                    )

                    if attempt < max_retries - 1:
                        delay = base_delay * (2 ** attempt)  # 지수 백오프
                        logger.info(f"Retrying in {delay} seconds...")
                        time.sleep(delay)
                    else:
                        logger.error(
                            f"{func.__name__} failed after {max_retries} attempts",
                            exc_info=True
                        )

            # 모든 재시도 실패
            raise last_exception

        return wrapper
    return decorator
```

**scheduler.py 수정** (재시도 적용):
```python
# agents/monitor/scheduler.py
from src.utils.retry import retry_with_backoff
from src.utils.logger import setup_logger

logger = setup_logger(__name__)

@retry_with_backoff(max_retries=3, base_delay=2.0)
def run_scenario(scenario_name: str):
    """시나리오 실행 (재시도 로직 포함)"""
    logger.info(f"=== Starting scenario: {scenario_name} ===")

    # ... 기존 로직
```

### 3. EmailNotifier 구현

**파일 생성**:
```bash
touch src/notifiers/email_notifier.py
```

**email_notifier.py 구현**:
```python
# src/notifiers/email_notifier.py
import smtplib
import os
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from datetime import datetime
import logging

logger = logging.getLogger(__name__)

class EmailNotifier:
    """이메일 알림 발송"""

    def __init__(self):
        self.smtp_host = os.getenv('SMTP_HOST', 'smtp.gmail.com')
        self.smtp_port = int(os.getenv('SMTP_PORT', '587'))
        self.smtp_user = os.getenv('SMTP_USER')
        self.smtp_password = os.getenv('SMTP_PASSWORD')
        self.from_email = os.getenv('FROM_EMAIL', self.smtp_user)
        self.to_email = os.getenv('ALERT_EMAIL')

        if not all([self.smtp_user, self.smtp_password, self.to_email]):
            logger.warning("Email notification is not configured")
            self.enabled = False
        else:
            self.enabled = True

    def send_failure_alert(self, scenario_name: str, error_message: str, log_path: str):
        """실행 실패 알림 전송"""
        if not self.enabled:
            logger.warning("Email notification is disabled (missing credentials)")
            return False

        try:
            subject = f"[실패] 자동 분석 실행 실패 - {scenario_name} - {datetime.now().strftime('%Y-%m-%d')}"

            body = f"""
자동 분석 실행이 실패했습니다.

시나리오: {scenario_name}
발생 시각: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}
재시도: 3회 모두 실패

에러 내용:
{error_message}

로그 파일:
{log_path}

조치 사항:
1. 로그 파일 확인
2. 데이터 소스 API 상태 확인
3. 네트워크 연결 확인
4. 필요 시 수동 실행

---
이 메일은 자동으로 발송되었습니다.
"""

            msg = MIMEMultipart()
            msg['From'] = self.from_email
            msg['To'] = self.to_email
            msg['Subject'] = subject
            msg.attach(MIMEText(body, 'plain', 'utf-8'))

            with smtplib.SMTP(self.smtp_host, self.smtp_port) as server:
                server.starttls()
                server.login(self.smtp_user, self.smtp_password)
                server.send_message(msg)

            logger.info(f"Failure alert email sent to {self.to_email}")
            return True

        except Exception as e:
            logger.error(f"Failed to send email alert: {str(e)}")
            return False
```

**scheduler.py 수정** (이메일 알림 추가):
```python
# agents/monitor/scheduler.py
from src.notifiers.email_notifier import EmailNotifier

def main():
    email_notifier = EmailNotifier()

    try:
        run_scenario("semiconductor-weekly")
    except Exception as e:
        logger.error("All retries failed, sending email alert")
        email_notifier.send_failure_alert(
            scenario_name="semiconductor-weekly",
            error_message=str(e),
            log_path=f"logs/execution_{datetime.now().strftime('%Y%m%d')}.log"
        )
        sys.exit(1)
```

### 4. 로그 정리 스크립트

**파일 생성**:
```bash
touch scripts/cleanup_logs.sh
chmod +x scripts/cleanup_logs.sh
```

**cleanup_logs.sh**:
```bash
#!/bin/bash
# 30일 이상 된 로그 파일 삭제

LOG_DIR="/Users/yoohyuntak/workspace/analysis_economy/logs"

# 실행 로그: 30일 보관
find "$LOG_DIR" -name "execution.log.*" -type f -mtime +30 -delete

# 에러 로그: 90일 보관
find "$LOG_DIR" -name "errors.log.*" -type f -mtime +90 -delete

echo "Log cleanup completed at $(date)"
```

**Crontab 추가** (매일 새벽 3시 정리):
```
0 3 * * * /Users/yoohyuntak/workspace/analysis_economy/scripts/cleanup_logs.sh >> /Users/yoohyuntak/workspace/analysis_economy/logs/cleanup.log 2>&1
```

### 5. 환경 변수 설정

**.env 업데이트**:
```bash
# 이메일 알림 설정
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=your-app-password  # Gmail 앱 비밀번호
FROM_EMAIL=your-email@gmail.com
ALERT_EMAIL=your-alert-email@gmail.com
```

**Gmail 앱 비밀번호 생성**:
1. Google 계정 → 보안 → 2단계 인증 활성화
2. 앱 비밀번호 생성
3. .env에 입력

### 6. 테스트

**의도적 에러 발생 테스트**:
```python
# test_error_handling.py
from agents.monitor.scheduler import run_scenario
import os

# API 키 일시적으로 제거
os.environ['ANTHROPIC_API_KEY'] = 'invalid-key'

try:
    run_scenario('semiconductor-weekly')
except Exception as e:
    print(f"Expected error: {e}")
    # 이메일 알림 받았는지 확인
```

**로그 확인**:
```bash
# 실행 로그
tail -f logs/execution.log

# 에러 로그
tail -f logs/errors.log
```

---

## Technical Notes

### 로그 레벨 가이드
- **DEBUG**: 개발 중 상세 정보
- **INFO**: 정상 실행 흐름 (시작/종료/성공)
- **WARNING**: 경고 (재시도 중, API 응답 느림)
- **ERROR**: 에러 (실패, 예외)
- **CRITICAL**: 심각한 시스템 문제

### 재시도 전략
- **멱등성**: 재시도해도 결과가 같아야 함 (중복 리포트 생성 방지)
- **지수 백오프**: 네트워크/API 부하 분산
- **최대 재시도**: 3회 (2+4+8 = 14초 총 대기)

### 이메일 대안
- Gmail 외 SMTP: SendGrid, AWS SES
- 슬랙 웹훅 (Epic 3 Story 2와 통합 가능)
- Pushover, PagerDuty (운영 환경)

---

## Definition of Done

- [ ] logger.py 구현 완료
- [ ] retry.py 구현 완료
- [ ] EmailNotifier 구현 완료
- [ ] scheduler.py에 통합
- [ ] 로그 정리 스크립트 작성
- [ ] 의도적 에러 테스트 성공
- [ ] 이메일 알림 수신 확인
- [ ] 30일 후 로그 자동 삭제 확인 (또는 수동 시뮬레이션)
- [ ] 문서 업데이트
- [ ] AC 모두 충족

---

## Dependencies

**Prerequisite**:
- Story 1 (스케줄러) 완료
- .env 파일 설정

**Blocks**:
- 없음 (독립적)

---

## Estimated Time

- **로깅 시스템**: 2시간
- **재시도 로직**: 1.5시간
- **이메일 알림**: 2시간
- **로그 정리**: 0.5시간
- **테스트**: 1시간
- **Total**: 5 Story Points

---

## Risks & Mitigation

**Risk 1**: Gmail SMTP 차단
- **원인**: 보안 정책 위반, 일일 발송 제한
- **완화**: SendGrid 같은 전문 서비스, 또는 슬랙 알림만 사용

**Risk 2**: 로그 파일 디스크 공간 부족
- **원인**: 로그 정리 실패
- **완화**: 디스크 사용량 모니터링, 로그 압축

**Risk 3**: 재시도 시 중복 작업
- **원인**: 멱등성 미보장
- **완화**: 타임스탬프 체크, 기존 리포트 덮어쓰기 정책

---

## Success Metrics

- ✅ 의도적 에러 3회 재시도 후 이메일 수신
- ✅ 로그 파일에서 모든 재시도 확인 가능
- ✅ 30일 이상 로그 자동 삭제 동작
- ✅ 이메일 알림 도달 시간 < 1분

---

## References

- [Python logging](https://docs.python.org/3/library/logging.html)
- [smtplib](https://docs.python.org/3/library/smtplib.html)
- [Gmail App Passwords](https://support.google.com/accounts/answer/185833)
- Story 1: 스케줄러 구현
- Story 2: 슬랙 알림 (대안)
