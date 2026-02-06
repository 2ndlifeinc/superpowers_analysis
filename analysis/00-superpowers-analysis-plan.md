# obra/Superpowers 분석 계획 (openclaw 관점)

## 목표
**Claude Code의 Superpowers를 openclaw 생태계에 적용 가능한 패턴으로 추상화**

- 핵심: "상황별 자동 트리거 + 서브에이전트 오케스트레이션"
- openclaw 고유 특성 활용: GitHub issue 토론, Slack 장기 협업, cron 자율 실행, peekaboo macOS 제어

---

## Superpowers 핵심 특징 (초기 파악)

### 1. 자동 워크플로우 트리거
```
상황 감지 → 관련 스킬 자동 활성화 → 프로세스 강제 실행
(Claude Code 플러그인 시스템 기반)
```

### 2. Subagent-Driven Development (SDD)
```
Plan 작성 → 작업 단위로 분할 → 각 작업마다 fresh subagent 생성
         → 2단계 리뷰 (spec compliance + code quality) 
         → 다음 작업으로 진행
```

### 3. 체계적 프로세스
```
brainstorming → git worktree → write plan → execute/SDD
            → code review → finish branch
```

### 4. TDD 강제
```
RED (failing test) → watch fail → GREEN (minimal code)
                  → watch pass → commit → refactor
```

---

## 분석 접근법 (openclaw_analysis 차용)

### 원칙
1. **문서 우선 (Document-First)**: README → SKILL.md → 구현 코드 (필요시만)
2. **추상화 집중**: Claude Code 한정 기능이 아니라, **범용 패턴** 추출
3. **openclaw 특성 매핑**: 
   - GitHub issue → 장기 토론 (Slack 채널과 동일 역할)
   - Subagent → openclaw의 `sessions_spawn`
   - Cron → 자율 실행 (인간 없이 작업 진행)
   - Peekaboo → macOS 앱 제어로 워크플로우 확장

---

## 분석 순서 (Phase별)

### Phase 1: 워크플로우 자동 트리거 메커니즘 ⭐ (최우선)
**목표**: 상황 감지 + 스킬 자동 활성화 패턴 파악

**읽을 문서**:
- `README.md` (전체 흐름 파악 완료)
- `skills/using-superpowers/SKILL.md` (스킬 시스템 소개)
- `skills/brainstorming/SKILL.md` (트리거 조건 예시)
- `skills/test-driven-development/SKILL.md` (TDD 강제 로직)
- `.claudecode/plugin.json` (플러그인 매니페스트)

**코드 확인 (필요시)**:
- 플러그인 훅 구현 (Claude Code 플러그인 API 사용 부분)
- 스킬 트리거 조건 로직

**출력물**:
```
analysis/01-auto-trigger-workflow/
├── 01-diagram.md              # 트리거 메커니즘 시각화
├── 02-details.md              # 상세 + 근거 코드
├── 03-application-plan.md     # openclaw 적용 방안
└── 04-implementation-prompt.md # 구현 프롬프트
```

**openclaw 적용 아이디어**:
- `AGENTS.md` 규칙 기반 자동 실행
- Slack 메시지 패턴 감지 → cron 트리거
- GitHub issue 라벨/키워드 감지 → sub-agent 자동 dispatch

---

### Phase 2: Subagent-Driven Development (SDD) 패턴 ⭐
**목표**: 서브에이전트 오케스트레이션 + 2단계 리뷰 구조 파악

**읽을 문서**:
- `skills/subagent-driven-development/SKILL.md` (핵심)
- `skills/dispatching-parallel-agents/SKILL.md` (병렬 실행)
- `skills/requesting-code-review/SKILL.md` (리뷰 체크리스트)
- `tests/subagent-driven-dev/` (실제 사용 예시)

**코드 확인 (필요시)**:
- Subagent 생성/관리 로직
- 작업 큐 구조
- 리뷰 결과 병합 로직

**출력물**:
```
analysis/02-subagent-orchestration/
├── 01-diagram.md              # SDD 흐름 시각화
├── 02-details.md              # 2단계 리뷰 상세
├── 03-application-plan.md     # openclaw sessions_spawn 활용
└── 04-implementation-prompt.md
```

**openclaw 적용 아이디어**:
- `sessions_spawn` + Slack 스레드 실시간 진행 보고
- GitHub issue에 작업 단위별 체크리스트 + sub-agent 자동 할당
- cron으로 주기적 진행 체크 + 멈춤 감지 → Slack 알림

---

### Phase 3: 체계적 디버깅 (Systematic Debugging)
**목표**: 4단계 근본 원인 분석 프로세스 파악

**읽을 문서**:
- `skills/systematic-debugging/SKILL.md`
- `skills/verification-before-completion/SKILL.md` (완료 검증)
- `skills/systematic-debugging/*.md` (서브 문서들)

**출력물**:
```
analysis/03-systematic-debugging/
├── 01-diagram.md              # 디버깅 프로세스 흐름
├── 02-details.md              # 4단계 상세
├── 03-application-plan.md     # Slack 토론 + 로그 분석 연계
└── 04-implementation-prompt.md
```

**openclaw 적용 아이디어**:
- Slack 에러 보고 → 자동 디버깅 sub-agent 시작
- GitHub issue에 디버깅 타임라인 기록
- Peekaboo로 macOS 앱 상태 캡처 + 로그 수집

---

### Phase 4: Git Worktree + Branch 관리
**목표**: 격리된 작업 환경 + 완료 처리 패턴

**읽을 문서**:
- `skills/using-git-worktrees/SKILL.md`
- `skills/finishing-a-development-branch/SKILL.md`

**출력물**:
```
analysis/04-git-workflow/
├── 01-diagram.md
├── 02-details.md
├── 03-application-plan.md     # GitHub PR 자동 생성 + Slack 리뷰 요청
└── 04-implementation-prompt.md
```

**openclaw 적용 아이디어**:
- 작업 시작 시 자동 worktree 생성
- 완료 시 GitHub PR 자동 생성 + Slack에 리뷰 요청 멘션

---

### Phase 5: Plan 작성 + 실행 패턴
**목표**: 상세 계획 작성 기준 + 실행 전략

**읽을 문서**:
- `skills/writing-plans/SKILL.md`
- `skills/executing-plans/SKILL.md`
- `commands/write-plan.md`
- `commands/execute-plan.md`

**출력물**:
```
analysis/05-planning-execution/
├── 01-diagram.md
├── 02-details.md
├── 03-application-plan.md     # Notion/GitHub issue 계획 연동
└── 04-implementation-prompt.md
```

**openclaw 적용 아이디어**:
- Notion 페이지 → plan.md 자동 변환
- GitHub issue 체크리스트 → 실행 단위 자동 분할
- Slack에 진행률 실시간 업데이트

---

### Phase 6: Brainstorming + Design 프로세스 (선택)
**목표**: 소크라테스식 질문 + 점진적 설계 방법

**읽을 문서**:
- `skills/brainstorming/SKILL.md`
- `docs/plans/*.md` (실제 설계 문서 예시)

**출력물**:
```
analysis/06-design-process/
├── 01-diagram.md
├── 02-details.md
├── 03-application-plan.md     # Slack 토론 + Notion 문서화
└── 04-implementation-prompt.md
```

---

## openclaw 고유 특성 활용 계획

### 1. GitHub Issue를 "장기 토론 + 작업 추적" 중심으로
```
Issue 생성 → 라벨 감지 → sub-agent 자동 할당
          → 체크리스트 자동 생성 → 진행 상황 주기적 업데이트
          → 완료 시 PR 자동 생성
```

### 2. Slack을 "실시간 협업 + 알림" 중심으로
```
주요 발견 → Slack 스레드에 요약 + 팀원 멘션
작업 멈춤 → cron 감지 → Slack 알림
리뷰 필요 → GitHub PR 링크 + Slack에 멘션
```

### 3. Cron을 "자율 실행 + 진행 체크" 중심으로
```
30분마다 → 할당된 작업 상태 체크
         → 멈춤 감지 시 Slack 알림
         → 다음 작업 자동 시작 (조건 충족 시)
```

### 4. Peekaboo를 "macOS 앱 통합" 중심으로
```
디버깅 시 → 앱 상태 캡처
리뷰 시 → 스크린샷 자동 첨부
워크플로우 → Xcode/VS Code 자동 제어
```

---

## 팀 토론 전략

### 토론 시점
1. **Phase 1 완료 후**: 자동 트리거 메커니즘 타당성 검증
2. **Phase 2 완료 후**: SDD 패턴 openclaw 적용 방안 리뷰
3. **Phase 3 완료 후**: 디버깅 프로세스 실전 적용 계획
4. **최종 통합 plan 완성 후**: 전체 워크플로우 검토

### 멘션 대상 (개발 지식 추정)
- <@U06JBUT7MF0> (박재영) - 필수
- Slack #개발-통합채널 활동자 중 코드/기술 토론 참여자

### 멘션 제외 시간
- KST 23:00 ~ 08:00 (새벽 시간대)

---

## 작업 멈춤 감지 Cron 설정

### Cron 조건
```yaml
schedule:
  kind: every
  everyMs: 1800000  # 30분마다

payload:
  kind: agentTurn
  message: |
    superpowers_analysis 작업이 진행 중인지 확인.
    - 마지막 커밋 시간 확인 (24시간 이상 경과 시 알림)
    - 현재 세션에서 이 작업 진행 중인지 확인
    - 멈춤 감지 시 Slack #개발-통합채널 스레드에 알림

sessionTarget: isolated
```

### 알림 포맷
```
⚠️ superpowers_analysis 작업 점검

상태: 24시간 이상 진행 없음
마지막 커밋: 2026-02-05 23:30 (25시간 전)
현재 phase: Phase 2 (subagent-orchestration)

확인 필요: <@U06JBUT7MF0>
```

---

## 검증 방법

### 각 Phase 완료 후
1. **다이어그램 검증**: 핵심 흐름이 명확한가?
2. **근거 코드 확인**: 추상화가 실제 구현 기반인가?
3. **적용 계획 실현성**: openclaw로 구현 가능한가?
4. **팀 토론**: 실전 적용 시 문제점은?

### 최종 통합 plan 검증
1. **전체 워크플로우 시뮬레이션**: A→B→C 흐름이 자연스러운가?
2. **openclaw 특성 활용**: GitHub/Slack/cron/peekaboo가 유기적으로 연결되는가?
3. **구현 우선순위**: 어떤 순서로 개발할 것인가?

---

## 예상 산출물 (최종)

```
analysis/
├── 00-superpowers-analysis-plan.md         # 이 파일
├── 01-auto-trigger-workflow/
│   ├── 01-diagram.md
│   ├── 02-details.md
│   ├── 03-application-plan.md
│   └── 04-implementation-prompt.md
├── 02-subagent-orchestration/
│   ├── 01-diagram.md
│   ├── 02-details.md
│   ├── 03-application-plan.md
│   └── 04-implementation-prompt.md
├── 03-systematic-debugging/
│   ├── 01-diagram.md
│   ├── 02-details.md
│   ├── 03-application-plan.md
│   └── 04-implementation-prompt.md
├── 04-git-workflow/
│   ├── 01-diagram.md
│   ├── 02-details.md
│   ├── 03-application-plan.md
│   └── 04-implementation-prompt.md
├── 05-planning-execution/
│   ├── 01-diagram.md
│   ├── 02-details.md
│   ├── 03-application-plan.md
│   └── 04-implementation-prompt.md
├── 06-design-process/ (선택)
│   ├── 01-diagram.md
│   ├── 02-details.md
│   ├── 03-application-plan.md
│   └── 04-implementation-prompt.md
└── 99-integrated-workflow/
    ├── 01-full-workflow-diagram.md         # 전체 통합 워크플로우
    ├── 02-implementation-roadmap.md       # 구현 우선순위 + 로드맵
    └── 03-openclaw-enhancement-proposals.md # openclaw 개선 제안
```

---

## 다음 단계

1. ✅ 이 plan을 GitHub에 커밋
2. ✅ Cron 등록 (작업 멈춤 감지)
3. 🔄 Phase 1 시작: 자동 트리거 메커니즘 분석
4. 🔄 Slack에 토론 스레드 생성 (Phase 1 완료 후)
