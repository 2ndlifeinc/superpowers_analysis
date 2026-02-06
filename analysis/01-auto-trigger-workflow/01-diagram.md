# Phase 1: 자동 트리거 워크플로우 - 개념 시각화

## 핵심 개념

Superpowers의 핵심은 **"상황 감지 → 스킬 자동 활성화 → 프로세스 강제 실행"** 흐름입니다.

---

## 전체 흐름 다이어그램

```mermaid
graph TB
    A[사용자 메시지 수신] --> B{스킬 적용 가능성<br/>1% 이상?}
    B -->|YES| C[🔍 스킬 검색 & 호출]
    B -->|NO| Z[일반 응답]
    
    C --> D{스킬 우선순위<br/>판단}
    D -->|Process Skills| E[Process Skill 실행<br/>brainstorming, debugging]
    D -->|Implementation Skills| F[Implementation Skill 실행<br/>frontend, mcp-builder]
    
    E --> G{체크리스트 있음?}
    F --> G
    
    G -->|YES| H[📝 TodoWrite 생성<br/>작업 항목 추적]
    G -->|NO| I[스킬 내용 직접 실행]
    
    H --> I
    I --> J[스킬 규칙 강제 실행]
    J --> K{스킬 타입}
    
    K -->|Rigid<br/>TDD, debugging| L[정확히 따름<br/>예외 없음]
    K -->|Flexible<br/>patterns| M[원칙 적용<br/>컨텍스트 반영]
    
    L --> N[결과 산출]
    M --> N
    N --> O{다음 단계<br/>스킬 필요?}
    
    O -->|YES| C
    O -->|NO| P[✅ 완료]
    
    style A fill:#e1f5ff
    style C fill:#fff3cd
    style J fill:#ffe6e6
    style P fill:#d4edda
```

---

## 스킬 트리거 조건 (예시: brainstorming)

```mermaid
graph LR
    A[사용자 요청] --> B{키워드 감지}
    
    B -->|"create feature"| C[✅ brainstorming<br/>트리거]
    B -->|"build component"| C
    B -->|"add functionality"| C
    B -->|"modify behavior"| C
    B -->|"help me plan"| C
    
    C --> D[프로세스 실행:<br/>1. 이해<br/>2. 접근법 제안<br/>3. 디자인 제시]
    
    D --> E[문서 저장:<br/>docs/plans/design.md]
    
    E --> F{다음 단계?}
    
    F -->|구현 계속| G[using-git-worktrees<br/>트리거]
    F -->|일단 멈춤| H[대기]
    
    G --> I[writing-plans<br/>트리거]
    
    style C fill:#ffe6e6
    style D fill:#fff3cd
    style G fill:#ffe6e6
    style I fill:#ffe6e6
```

---

## Red Flags 감지 시스템 (합리화 방지)

```mermaid
graph TB
    A[에이전트 응답 생성 시도] --> B{Red Flag 감지}
    
    B -->|"This is just a<br/>simple question"| C[❌ STOP<br/>질문도 작업임]
    B -->|"I need more<br/>context first"| D[❌ STOP<br/>스킬 먼저 확인]
    B -->|"Let me explore<br/>codebase first"| E[❌ STOP<br/>스킬이 탐색법 제공]
    B -->|"This doesn't need<br/>a formal skill"| F[❌ STOP<br/>스킬 있으면 사용]
    B -->|"I remember<br/>this skill"| G[❌ STOP<br/>스킬은 진화함]
    
    C --> H[🔄 스킬 호출로 돌아감]
    D --> H
    E --> H
    F --> H
    G --> H
    
    B -->|Red Flag 없음| I[✅ 스킬 호출 진행]
    
    style C fill:#ffcccc
    style D fill:#ffcccc
    style E fill:#ffcccc
    style F fill:#ffcccc
    style G fill:#ffcccc
    style I fill:#d4edda
```

---

## 스킬 체인 실행 (예: 전체 개발 워크플로우)

```mermaid
graph TD
    A[사용자: "Let's build X"] --> B[🔍 brainstorming<br/>트리거]
    
    B --> C[디자인 문서 생성:<br/>docs/plans/design.md]
    
    C --> D{사용자 승인?}
    
    D -->|YES| E[🔍 using-git-worktrees<br/>트리거]
    D -->|NO| B
    
    E --> F[격리된 브랜치 생성]
    
    F --> G[🔍 writing-plans<br/>트리거]
    
    G --> H[상세 계획 생성:<br/>docs/plans/plan.md]
    
    H --> I[🔍 subagent-driven-development<br/>또는 executing-plans<br/>트리거]
    
    I --> J[작업 단위별 실행]
    
    J --> K{각 작업마다:<br/>🔍 test-driven-development<br/>트리거}
    
    K --> L[RED → GREEN → REFACTOR]
    
    L --> M[🔍 requesting-code-review<br/>트리거]
    
    M --> N{리뷰 통과?}
    
    N -->|NO| O[수정]
    N -->|YES| P{더 작업?}
    
    O --> M
    
    P -->|YES| J
    P -->|NO| Q[🔍 finishing-a-development-branch<br/>트리거]
    
    Q --> R[✅ 완료:<br/>merge/PR/keep/discard]
    
    style B fill:#ffe6e6
    style E fill:#ffe6e6
    style G fill:#ffe6e6
    style I fill:#ffe6e6
    style K fill:#ffe6e6
    style M fill:#ffe6e6
    style Q fill:#ffe6e6
    style R fill:#d4edda
```

---

## 강제 실행 메커니즘 (Rigid Skills)

```mermaid
graph TB
    A[Rigid Skill 트리거<br/>예: TDD] --> B[철칙 제시:<br/>"NO CODE WITHOUT FAILING TEST"]
    
    B --> C{에이전트 응답}
    
    C -->|규칙 위반<br/>코드 먼저 작성| D[❌ 강제 삭제 지시]
    
    D --> E[처음부터 다시 시작]
    
    C -->|규칙 준수<br/>테스트 먼저 작성| F[✅ 진행 허용]
    
    F --> G[RED → watch fail]
    
    G --> H[GREEN → minimal code]
    
    H --> I[watch pass]
    
    I --> J[REFACTOR]
    
    J --> K{테스트 여전히<br/>통과?}
    
    K -->|NO| L[❌ 리팩토링 롤백]
    K -->|YES| M[✅ commit]
    
    L --> J
    M --> N{다음 기능?}
    
    N -->|YES| G
    N -->|NO| O[✅ 완료]
    
    style D fill:#ffcccc
    style L fill:#ffcccc
    style F fill:#d4edda
    style M fill:#d4edda
    style O fill:#d4edda
```

---

## 핵심 원칙 요약

### 1. **무조건 트리거 (1% Rule)**
- 1% 가능성이라도 있으면 스킬 호출
- "나중에" 또는 "필요 없을 것 같은데" 금지

### 2. **스킬 우선순위**
```
Process Skills (brainstorming, debugging)
          ↓
Implementation Skills (frontend, mcp-builder)
```

### 3. **Red Flags (합리화 방지)**
- "간단한 질문이니까" → 질문도 작업
- "컨텍스트 먼저 확인" → 스킬이 확인법 제공
- "스킬 기억나는데" → 스킬은 진화함

### 4. **Rigid vs Flexible**
- **Rigid**: 정확히 따름 (TDD, debugging)
- **Flexible**: 원칙 적용 (patterns)

### 5. **체크리스트 자동 생성**
- 스킬에 체크리스트 있으면 → TodoWrite 자동 생성
- 진행 상황 추적 가능

---

## openclaw 적용 시사점

### 1. **AGENTS.md 규칙 기반 자동 트리거**
- `AGENTS.md`에 "상황 → 스킬" 매핑 정의
- 예: Slack에 "에러" 키워드 → systematic-debugging 자동 실행

### 2. **Slack 메시지 패턴 감지**
```
Slack 메시지 수신 → 패턴 분석 → cron 트리거 → sub-agent 시작
```

### 3. **GitHub issue 라벨 기반**
```
Issue 라벨 "bug" → debugging skill 자동 할당
Issue 라벨 "feature" → brainstorming → plan → implement 체인
```

### 4. **Red Flags를 IDENTITY.md 규칙으로**
- "절대 금지" 섹션과 유사
- 위반 감지 → 자동 경고 → 재시도 강제
