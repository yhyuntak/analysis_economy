# User Story 2: 슬랙 웹훅 통합 및 알림 포맷

## User Story

**As a** 바쁜 투자자
**I want** 리포트 생성 완료 시 슬랙으로 핵심 요약을 받고 싶다
**So that** 터미널이나 파일 시스템에 접근하지 않고도 모바일에서 즉시 확인할 수 있다

---

## Story Details

**Story ID**: EPIC-003-S2
**Epic**: Epic 3 - Agent A 초기 (정기 실행 및 알림)
**Story Points**: 5
**Priority**: Must Have
**Assignee**: Developer (본인)
**Status**: 대기 중

---

## Acceptance Criteria

### AC1: 슬랙 Incoming Webhook 설정
- [ ] 슬랙 워크스페이스에서 Incoming Webhook 생성
- [ ] Webhook URL을 `.env`에 안전하게 저장
- [ ] 테스트 메시지 전송 성공

### AC2: 알림 메시지 포맷 구현
- [ ] 리포트 제목, 생성 일시 포함
- [ ] 핵심 요약 3-5줄 표시
- [ ] 전체 리포트 링크 (또는 파일 경로) 제공
- [ ] 이모지 사용으로 가독성 향상

### AC3: NotifierBase 클래스 설계
- [ ] 알림 서비스 추상화 (슬랙, 이메일 등 확장 가능)
- [ ] `SlackNotifier` 클래스 구현
- [ ] 알림 전송 성공/실패 로깅

### AC4: 리포트 메타데이터 추출
- [ ] 리포트 생성 시 메타데이터 JSON 파일 생성
  - 제목, 생성일시, 시나리오명, 요약 3줄
- [ ] 메타데이터를 슬랙 알림에 사용

### AC5: 통합 테스트
- [ ] 시나리오 실행 → 리포트 생성 → 슬랙 알림 E2E 테스트
- [ ] 알림 도달 시간 < 1분
- [ ] 슬랙 알림 포맷 가독성 검증

---

## Tasks

### Task 1: 슬랙 Incoming Webhook 생성
- [ ] 슬랙 워크스페이스 로그인
- [ ] Apps → Incoming Webhooks 검색
- [ ] "Add to Slack" → 채널 선택 (#투자-리포트 추천)
- [ ] Webhook URL 복사
- [ ] `.env`에 추가
  ```
  SLACK_WEBHOOK_URL=https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXX
  ```

### Task 2: SlackNotifier 클래스 구현
- [ ] `src/notifiers/` 디렉토리 생성
- [ ] `base_notifier.py` 구현
  ```python
  from abc import ABC, abstractmethod
  from typing import Dict, Any

  class NotifierBase(ABC):
      @abstractmethod
      def send(self, report_metadata: Dict[str, Any]) -> bool:
          """알림 전송"""
          pass
  ```

- [ ] `slack_notifier.py` 구현
  ```python
  import requests
  from typing import Dict, Any

  class SlackNotifier(NotifierBase):
      def __init__(self, webhook_url: str):
          self.webhook_url = webhook_url

      def send(self, report_metadata: Dict[str, Any]) -> bool:
          """
          슬랙으로 리포트 알림 전송
          """
          blocks = self._build_message_blocks(report_metadata)

          response = requests.post(
              self.webhook_url,
              json={"blocks": blocks}
          )

          return response.status_code == 200

      def _build_message_blocks(self, metadata: Dict[str, Any]) -> list:
          """
          슬랙 Block Kit 메시지 구성
          """
          return [
              {
                  "type": "header",
                  "text": {
                      "type": "plain_text",
                      "text": f"🔔 {metadata['title']} 완료"
                  }
              },
              {
                  "type": "section",
                  "fields": [
                      {
                          "type": "mrkdwn",
                          "text": f"*분석 기간:*\n{metadata['period']}"
                      },
                      {
                          "type": "mrkdwn",
                          "text": f"*생성 시각:*\n{metadata['timestamp']}"
                      }
                  ]
              },
              {
                  "type": "section",
                  "text": {
                      "type": "mrkdwn",
                      "text": f"*📊 핵심 요약:*\n{metadata['summary']}"
                  }
              },
              {
                  "type": "divider"
              },
              {
                  "type": "context",
                  "elements": [
                      {
                          "type": "mrkdwn",
                          "text": f"📄 전체 리포트: `{metadata['report_path']}`"
                      }
                  ]
              }
          ]
  ```

### Task 3: 리포트 메타데이터 추출
- [ ] `MarkdownReporter`에 메타데이터 추출 메서드 추가
  ```python
  def generate(self, data, insights) -> tuple[str, dict]:
      """
      리포트 생성 및 메타데이터 반환
      """
      report_path = self._save_report(content)

      metadata = {
          'title': '반도체 섹터 주간 점검',
          'timestamp': datetime.now().strftime('%Y-%m-%d %H:%M'),
          'period': f"{start_date} ~ {end_date}",
          'summary': self._extract_summary(insights),
          'report_path': report_path
      }

      return report_path, metadata
  ```

### Task 4: 시나리오에 알림 통합
- [ ] `ScenarioBase.run()` 메서드에 알림 로직 추가
  ```python
  def run(self) -> str:
      # ... (기존 로직)

      report_path, metadata = self._generate_report(raw_data, insights)

      # 알림 전송
      if self.config.get('enable_notification', True):
          self._send_notifications(metadata)

      return report_path

  def _send_notifications(self, metadata: dict):
      """알림 전송"""
      for notifier in self.notifiers:
          try:
              notifier.send(metadata)
          except Exception as e:
              logger.error(f"Notification failed: {e}")
  ```

### Task 5: 설정 파일 업데이트
- [ ] `config.yaml`에 알림 설정 추가
  ```yaml
  notifications:
    slack:
      enabled: true
      webhook_url_env: SLACK_WEBHOOK_URL
    email:
      enabled: false
  ```

### Task 6: 테스트 스크립트 작성
- [ ] `tests/test_slack_notification.py` 작성
  ```python
  def test_slack_notification():
      notifier = SlackNotifier(os.getenv('SLACK_WEBHOOK_URL'))

      test_metadata = {
          'title': '테스트 리포트',
          'timestamp': '2025-11-18 10:00',
          'period': '2025-11-11 ~ 2025-11-18',
          'summary': '- 테스트 1\n- 테스트 2\n- 테스트 3',
          'report_path': '/path/to/report.md'
      }

      result = notifier.send(test_metadata)
      assert result == True
  ```

### Task 7: E2E 통합 테스트
- [ ] 시나리오 #1 실행 → 슬랙 알림 확인
- [ ] 알림 메시지 포맷 검증
- [ ] 모바일에서 슬랙 알림 확인 (가독성)

---

## Technical Notes

### 슬랙 Block Kit

슬랙은 Block Kit을 사용하여 풍부한 메시지 포맷 제공:
- **Header**: 제목
- **Section**: 주요 내용, 필드
- **Divider**: 구분선
- **Context**: 부가 정보

Block Kit Builder에서 미리 디자인 가능:
https://api.slack.com/block-kit/building

### 요약 추출 로직

```python
def _extract_summary(self, llm_insights: str) -> str:
    """
    LLM 생성 텍스트에서 핵심 요약 추출
    """
    # Option 1: "요약" 섹션에서 추출
    if "## 요약" in llm_insights:
        summary_section = llm_insights.split("## 요약")[1].split("##")[0]
        lines = [line.strip() for line in summary_section.split('\n') if line.strip() and line.startswith('-')]
        return '\n'.join(lines[:5])  # 최대 5줄

    # Option 2: 전체 텍스트 첫 3줄
    lines = [line for line in llm_insights.split('\n') if line.strip()]
    return '\n'.join(lines[:3])
```

### 알림 실패 처리

```python
def send_with_retry(self, metadata: dict, max_retries: int = 3) -> bool:
    for attempt in range(max_retries):
        try:
            return self.send(metadata)
        except Exception as e:
            logger.warning(f"Attempt {attempt + 1} failed: {e}")
            if attempt < max_retries - 1:
                time.sleep(2 ** attempt)  # 지수 백오프
    return False
```

---

## Definition of Done

- [ ] 모든 Acceptance Criteria 충족
- [ ] 모든 Tasks 완료
- [ ] 슬랙 알림 E2E 테스트 성공
- [ ] 알림 포맷 가독성 검증 (실제 슬랙에서)
- [ ] 모바일에서 알림 확인
- [ ] 에러 핸들링 테스트 (Webhook URL 잘못됨 등)

---

## Dependencies

**선행 작업**:
- Epic 2 완료 (ScenarioBase 프레임워크 사용)

**후속 작업**:
- Story 3: 이메일 Fallback 구현

---

## Risks

| 리스크 | 완화 전략 |
|--------|-----------|
| 슬랙 Webhook 만료/변경 | .env에 저장, 재생성 가이드 문서화 |
| 알림 과부하 (너무 자주) | Epic 1에서 주 1회로 제한 |
| 메시지 포맷 가독성 | Block Kit Builder로 사전 디자인 |
| 네트워크 오류 | 재시도 로직, 이메일 Fallback (Story 3) |

---

## Estimated Time

**Story Points**: 5

**예상 시간**: 1일
- Task 1-2 (Webhook 설정 + Notifier 구현): 2-3시간
- Task 3-4 (메타데이터 + 통합): 2-3시간
- Task 5-7 (설정 + 테스트): 2-3시간

---

## Notes

- 슬랙 Block Kit은 직관적이지만 처음 사용 시 시행착오 예상
- Block Kit Builder에서 미리 디자인 후 JSON 복사 추천
- 알림 포맷은 모바일에서 확인 필수 (출퇴근 시 확인이 주 사용 케이스)
- 요약 추출 로직이 핵심: LLM 출력에서 깔끔하게 추출해야 함

---

**Created**: 2025-11-18
**Last Updated**: 2025-11-18
**Status**: Ready for Development
