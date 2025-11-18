# Epic 3: Agent A 초기 - 정기 실행 및 알림

## Epic 개요

**ID**: EPIC-003
**제목**: Agent A 초기 형태 - 정기 스케줄링 및 슬랙 알림
**Priority**: Must Have
**Story Points**: 13
**Target Release**: Release 2 (Month 4-6)
**Status**: 대기 중
**의존성**: Epic 2 완료 필요

---

## PM 관점

### 비즈니스 목표

1. **자동화 레벨 1 달성**: 사용자 수동 실행 → 자동 정기 실행
2. **사용자 개입 최소화**: 주 1회 실행 확인 → 월 1회 점검으로 감소
3. **정보 접근성 향상**: 리포트 생성 즉시 슬랙으로 알림 수신

### 사용자 가치

**Target User**: 바쁜 투자자

**Pain Point**:
- 매주 월요일 스크립트 실행 잊어버림
- 리포트 생성 여부 확인하려고 터미널 열어야 함
- 리포트 파일 위치 찾기 번거로움

**Provided Value**:
- 매주 월요일 오전 9시 자동 분석 실행
- 슬랙에서 핵심 요약 즉시 확인
- 전체 리포트는 링크 클릭으로 접근

### Success Metrics

- [ ] **자동 실행 성공률**: 95% 이상 (4주 관찰)
- [ ] **알림 도달 시간**: 리포트 생성 후 1분 내
- [ ] **사용자 만족도**: 리포트 확인 시간 30% 감소 (주관 평가)
- [ ] **안정성**: 실패 시 이메일 Fallback 동작

---

## Designer 관점

### UX 목표

1. **Zero Friction**: 사용자 액션 없이 정보 전달
2. **적절한 알림**: 핵심 정보만, 너무 길지 않게
3. **알림 피로 방지**: 주 1-2회로 제한, 중요도 필터링

### 슬랙 알림 포맷

```
🔔 반도체 섹터 주간 점검 완료

📅 분석 기간: 2025-11-11 ~ 2025-11-18
⏰ 생성 시각: 2025-11-18 09:05

📊 핵심 요약:
1. NVIDIA +4.1%, AI 수요 지속 강세
2. 삼성전자 -0.5%, 메모리 가격 약세
3. 전반적 섹터 상승세, 단기 과열 신호

📄 전체 리포트: [링크]
```

### 이메일 Fallback 포맷

```
Subject: [자동분석] 반도체 섹터 주간 점검 완료

안녕하세요,

반도체 섹터 주간 점검이 완료되었습니다.

생성 일시: 2025-11-18 09:05:23
리포트 경로: reports/semiconductor_weekly_20251118.md

핵심 요약:
- NVIDIA +4.1%, AI 수요 지속 강세
- 삼성전자 -0.5%, 메모리 가격 약세
- 전반적 섹터 상승세, 단기 과열 신호

전체 리포트를 확인하세요.

---
이 메일은 자동 생성되었습니다.
```

### 에러 알림 포맷

```
⚠️ 반도체 섹터 주간 점검 실패

🕐 시도 시각: 2025-11-18 09:00
❌ 에러: API rate limit exceeded (Claude API)

🔄 재시도: 30분 후 자동 재실행
📞 문제 지속 시: [관리자 연락처]
```

---

## Developer 관점

### 기술 목표

1. **Cron 통합**: macOS/Linux cron 또는 APScheduler 선택
2. **슬랙 웹훅 통합**: Incoming Webhook으로 메시지 전송
3. **로그 시스템 구축**: 실행 히스토리, 에러 추적
4. **에러 핸들링**: 재시도 로직, Fallback 알림

### Architecture Overview

```
agents/
└── monitor/                    # Agent A (초기 형태)
    ├── scheduler.py            # 스케줄링 로직
    ├── notifier.py             # 알림 발송 (슬랙, 이메일)
    └── executor.py             # 시나리오 실행 오케스트레이터

logs/
├── execution_YYYYMMDD.log      # 실행 로그
└── errors_YYYYMMDD.log         # 에러 로그

config/
└── schedule.yaml               # 스케줄 설정
```

### 스케줄 설정 예시

```yaml
# config/schedule.yaml
schedules:
  - scenario: semiconductor-weekly
    cron: "0 9 * * MON"         # 매주 월요일 09:00
    enabled: true
    notify:
      - slack
      - email

  - scenario: pmi-analysis
    cron: "0 10 1 * *"          # 매월 1일 10:00
    enabled: true
    notify:
      - slack
```

### Cron 설정 (macOS/Linux)

```bash
# crontab -e
0 9 * * MON cd /path/to/project && /path/to/venv/bin/python agents/monitor/scheduler.py --scenario semiconductor-weekly >> logs/cron.log 2>&1
```

### 슬랙 웹훅 통합

```python
# agents/monitor/notifier.py
import requests
from typing import Dict, Any

class SlackNotifier:
    def __init__(self, webhook_url: str):
        self.webhook_url = webhook_url

    def send(self, report_metadata: Dict[str, Any], summary: str):
        """
        슬랙으로 리포트 알림 전송
        """
        blocks = [
            {
                "type": "header",
                "text": {
                    "type": "plain_text",
                    "text": f"🔔 {report_metadata['title']} 완료"
                }
            },
            {
                "type": "section",
                "fields": [
                    {"type": "mrkdwn", "text": f"*분석 기간:*\n{report_metadata['period']}"},
                    {"type": "mrkdwn", "text": f"*생성 시각:*\n{report_metadata['timestamp']}"}
                ]
            },
            {
                "type": "section",
                "text": {
                    "type": "mrkdwn",
                    "text": f"*📊 핵심 요약:*\n{summary}"
                }
            },
            {
                "type": "actions",
                "elements": [
                    {
                        "type": "button",
                        "text": {"type": "plain_text", "text": "전체 리포트 보기"},
                        "url": report_metadata['report_url']
                    }
                ]
            }
        ]

        response = requests.post(
            self.webhook_url,
            json={"blocks": blocks}
        )

        return response.status_code == 200
```

### 실행 로그 예시

```
2025-11-18 09:00:00 [INFO] Scheduler started
2025-11-18 09:00:01 [INFO] Running scenario: semiconductor-weekly
2025-11-18 09:00:05 [INFO] Data collection completed (3 sources)
2025-11-18 09:00:45 [INFO] LLM analysis completed (42 tokens)
2025-11-18 09:01:02 [INFO] Report generated: reports/semiconductor_weekly_20251118.md
2025-11-18 09:01:05 [INFO] Slack notification sent
2025-11-18 09:01:05 [INFO] Execution completed successfully (65s)
```

### Implementation Complexity

**총 Story Points**: 13

- **Story 1**: Cron 스케줄링 설정 (3pt) - 낮은 난이도
- **Story 2**: 슬랙 웹훅 통합 및 알림 포맷 (5pt) - 중간 난이도
- **Story 3**: 실행 로그 및 에러 핸들링 (5pt) - 중간 난이도

### 주요 기술 과제

1. **Cron vs APScheduler 선택**
   - **Cron**: 간단, OS 의존적, 프로세스 외부
   - **APScheduler**: Python 내장, 동적 스케줄 변경 가능
   - **결정**: 초기엔 Cron (단순), 나중에 필요 시 APScheduler

2. **슬랙 웹훅 보안**
   - **해결**: .env에 SLACK_WEBHOOK_URL 저장, gitignore

3. **재시도 로직 설계**
   - **해결**: 3회 재시도 (지수 백오프), 최종 실패 시 이메일

4. **로그 관리**
   - **해결**: 일별 로그 파일, 30일 후 자동 삭제

### Dependencies

```txt
# 추가 의존성
slack-sdk==3.23.0       # 또는 간단히 requests 사용
schedule==1.2.0         # APScheduler 선택 시 대체
```

---

## User Stories

이 Epic은 3개의 User Story로 구성됩니다:

1. **[Story 1] Cron 스케줄링 설정 및 정기 실행** (3pt)
   - crontab 설정
   - 스케줄 설정 파일 (YAML)
   - 정기 실행 검증

2. **[Story 2] 슬랙 웹훅 통합 및 알림 포맷** (5pt)
   - 슬랙 Incoming Webhook 생성
   - NotifierBase 클래스 설계
   - 슬랙 알림 포맷 구현
   - 리포트 메타데이터 추출

3. **[Story 3] 실행 로그 및 에러 핸들링** (5pt)
   - 로깅 시스템 구축
   - 재시도 로직 구현
   - 이메일 Fallback
   - 실행 히스토리 저장

---

## Acceptance Criteria

Epic 전체가 완료되었다고 판단하는 기준:

- [ ] 매주 월요일 09:00에 반도체 시나리오 자동 실행 (4주 검증)
- [ ] 리포트 생성 후 1분 내 슬랙 알림 수신
- [ ] 알림에 핵심 요약 3-5줄 포함
- [ ] 전체 리포트 링크 클릭 시 파일 접근 가능
- [ ] 실행 실패 시 3회 재시도 후 이메일 알림
- [ ] 실행 로그에서 4주간 히스토리 확인 가능

---

## Risks & Mitigation

| 리스크 | 영향 | 확률 | 완화 전략 |
|--------|------|------|-----------|
| Cron 실행 실패 (macOS 재부팅) | 높음 | 낮음 | 시스템 시작 시 cron 재등록 스크립트 |
| 슬랙 웹훅 만료 | 중간 | 낮음 | 에러 발생 시 이메일 Fallback |
| 알림 피로 | 중간 | 중간 | 주 1회로 제한, 중요도 필터링 |
| 로그 파일 용량 증가 | 낮음 | 높음 | 30일 후 자동 삭제, 압축 저장 |

---

## Timeline Estimate

**총 예상 기간**: 1-2주 (1인 개발)

- **Week 1**: Story 1-2 (스케줄링 + 슬랙 통합)
- **Week 2**: Story 3 + 4주 안정성 테스트 시작

---

## 다음 단계

1. Epic 2 완료 후 진행
2. Story 1부터 순차 진행
3. Story 2 완료 후 슬랙 알림 테스트 (실제 워크스페이스)
4. 4주간 안정성 모니터링
5. Epic 4 (이벤트 감지)로 진화 고려

---

**Epic Owner**: Developer (본인)
**Created**: 2025-11-18
**Last Updated**: 2025-11-18
