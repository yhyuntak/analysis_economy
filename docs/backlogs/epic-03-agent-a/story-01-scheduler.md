# Story 1: Cron 스케줄링 설정 및 정기 실행

**Epic**: Epic 3 - Agent A 초기
**Points**: 3
**Priority**: Must
**Status**: Todo

---

## User Story

**As a** 바쁜 투자자
**I want** 매주 월요일 오전 9시 자동으로 분석 실행
**So that** 수동 실행을 잊어버리는 일이 없다

---

## Background & Context

### 문제 정의
- 현재: 매주 월요일마다 CLI로 수동 실행 필요
- 문제: 출장, 휴가, 바쁜 일정으로 잊어버릴 수 있음
- 결과: 일관성 없는 분석, 투자 의사결정 공백

### 솔루션
- Cron 기반 정기 실행 시스템
- 별도 관리 불필요 (시스템 자동 실행)
- 로깅으로 실행 이력 추적

### 범위
- Phase 1: 반도체 주간 점검만 자동화
- Phase 2+: 다중 시나리오 스케줄링

---

## Acceptance Criteria

### 필수 조건
- [ ] **AC1**: crontab에 스케줄 등록 완료
  - 매주 월요일 오전 9시 실행 (0 9 * * MON)
  - Python 가상환경 경로 포함
  - 프로젝트 디렉토리 자동 이동

- [ ] **AC2**: 스케줄러 스크립트 구현
  - `agents/monitor/scheduler.py` 파일 생성
  - 시나리오 실행 로직
  - 실행 시작/종료 타임스탬프 로깅

- [ ] **AC3**: 4주 연속 정상 실행 검증
  - Week 1-4 매주 월요일 09:00 실행 확인
  - 생성된 리포트 파일 검증
  - 로그 파일에서 실행 이력 확인

- [ ] **AC4**: 실행 로그 파일 생성
  - `logs/scheduler_YYYYMMDD.log` 포맷
  - 시작 시간, 종료 시간, 소요 시간 기록
  - 에러 발생 시 스택 트레이스

### 선택 조건
- [ ] **AC5**: 수동 실행 명령어도 동일하게 작동
  - `python agents/monitor/scheduler.py --scenario semiconductor-weekly`
  - Cron과 동일한 로직 사용

---

## Tasks

### 1. 프로젝트 구조 준비
```bash
mkdir -p agents/monitor
mkdir -p logs
touch agents/monitor/__init__.py
touch agents/monitor/scheduler.py
```

### 2. scheduler.py 구현
**기본 구조**:
```python
# agents/monitor/scheduler.py
import logging
from datetime import datetime
import sys
import os

# 프로젝트 루트를 sys.path에 추가
sys.path.insert(0, os.path.dirname(os.path.dirname(os.path.dirname(__file__))))

from src.scenarios.semiconductor import SemiconductorAnalyzer

def setup_logging():
    """로깅 설정"""
    log_file = f"logs/scheduler_{datetime.now().strftime('%Y%m%d')}.log"
    logging.basicConfig(
        level=logging.INFO,
        format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
        handlers=[
            logging.FileHandler(log_file),
            logging.StreamHandler()
        ]
    )
    return logging.getLogger(__name__)

def run_scenario(scenario_name: str):
    """시나리오 실행"""
    logger = setup_logging()
    logger.info(f"=== Scheduler started for scenario: {scenario_name} ===")

    start_time = datetime.now()

    try:
        if scenario_name == "semiconductor-weekly":
            analyzer = SemiconductorAnalyzer()
            result = analyzer.run()
            logger.info(f"Analysis completed successfully")
            logger.info(f"Report saved to: {result['report_path']}")
        else:
            raise ValueError(f"Unknown scenario: {scenario_name}")

        end_time = datetime.now()
        duration = (end_time - start_time).total_seconds()
        logger.info(f"=== Scheduler completed in {duration:.2f} seconds ===")

        return True

    except Exception as e:
        logger.error(f"Scheduler failed: {str(e)}", exc_info=True)
        return False

if __name__ == "__main__":
    import argparse

    parser = argparse.ArgumentParser(description='Run scheduled analysis')
    parser.add_argument('--scenario', required=True, help='Scenario name')
    args = parser.parse_args()

    success = run_scenario(args.scenario)
    sys.exit(0 if success else 1)
```

### 3. Crontab 설정
```bash
# crontab 편집
crontab -e

# 다음 라인 추가 (실제 경로로 수정 필요)
0 9 * * MON cd /Users/yoohyuntak/workspace/analysis_economy && /Users/yoohyuntak/workspace/analysis_economy/venv/bin/python agents/monitor/scheduler.py --scenario semiconductor-weekly
```

**주의사항**:
- 절대 경로 사용 (상대 경로 X)
- Python 가상환경 경로 정확히 지정
- 프로젝트 디렉토리로 cd 먼저

### 4. 테스트 실행
```bash
# 수동 테스트
python agents/monitor/scheduler.py --scenario semiconductor-weekly

# 로그 확인
cat logs/scheduler_$(date +%Y%m%d).log

# Cron 동작 확인 (1분 후 실행으로 테스트)
# crontab에 임시로 추가:
# */1 * * * * cd /path/to/project && /path/to/venv/bin/python agents/monitor/scheduler.py --scenario semiconductor-weekly
```

### 5. 4주 모니터링
- **Week 1**: 첫 실행 후 즉시 확인
- **Week 2-3**: 주간 확인
- **Week 4**: 최종 검증 및 AC3 완료 처리

---

## Technical Notes

### Cron vs APScheduler
**초기 선택: Cron**
- ✅ 장점: OS 레벨 안정성, 추가 의존성 없음, 프로세스 종료 시 영향 없음
- ⚠️ 단점: 설정 변경 시 crontab 수동 수정

**나중에 APScheduler**:
- Python 코드로 스케줄 관리
- 동적 스케줄 변경 가능
- 하지만 데몬 프로세스 필요

### 로그 관리
- **보관 기간**: 90일 (3개월)
- **정리 스크립트**: Epic 3 Story 3에서 구현
- **로그 레벨**:
  - INFO: 실행 시작/종료, 성공
  - ERROR: 실패, 예외
  - DEBUG: 상세 디버깅 (개발 시에만)

### 타임존
- 한국 시간 (KST, UTC+9) 기준
- 서버 타임존 확인: `date`
- 필요시 TZ 환경변수 설정

---

## Definition of Done

- [ ] Code 작성 완료 (`scheduler.py`)
- [ ] Crontab 설정 완료
- [ ] 수동 테스트 성공
- [ ] Cron 테스트 실행 성공 (1주차)
- [ ] 4주 연속 자동 실행 검증
- [ ] 로그 파일 생성 및 내용 확인
- [ ] 문서 업데이트 (README에 cron 설정 방법)
- [ ] Story AC 모두 충족

---

## Dependencies

**Prerequisite**:
- Epic 1 완료 (SemiconductorAnalyzer 구현됨)

**Blocks**:
- Story 2 (슬랙 알림) - 이 스토리 완료 후 알림 추가 가능

---

## Estimated Time

- **구현**: 2시간
- **테스트**: 1시간
- **4주 모니터링**: 패시브 (주 5분 체크)
- **Total**: 3 Story Points

---

## Risks & Mitigation

**Risk 1**: Cron이 실행되지 않음
- **원인**: 잘못된 경로, 권한 문제
- **완화**: 수동 테스트 먼저, cron 로그 확인 (`grep CRON /var/log/syslog`)

**Risk 2**: Python 환경 문제
- **원인**: 가상환경 경로 변경, 패키지 누락
- **완화**: 절대 경로 사용, requirements.txt 최신화

**Risk 3**: 서버 재부팅 시 cron 유실
- **원인**: crontab 백업 안 함
- **완화**: `crontab -l > crontab_backup.txt` 정기 백업

---

## Success Metrics

- ✅ 4주 연속 100% 실행 성공률
- ✅ 로그 파일에서 모든 실행 이력 확인 가능
- ✅ 리포트 파일 생성 시간 편차 < 5분 (09:00-09:05 사이)

---

## References

- [Cron Syntax](https://crontab.guru/)
- [Python logging](https://docs.python.org/3/library/logging.html)
- Epic 1 Story 4: 리포트 생성 로직
