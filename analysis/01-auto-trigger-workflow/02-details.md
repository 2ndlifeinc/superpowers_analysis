# Phase 1: 자동 트리거 워크플로우 - 상세 분석

## 1. 핵심 메커니즘: "1% Rule" 강제 실행

### 개념
**"스킬 적용 가능성이 1%라도 있으면 무조건 호출"**

### 근거 코드
**파일**: `skills/using-superpowers/SKILL.md`  
**라인**: 7-13

```markdown
<EXTREMELY-IMPORTANT>
If you think there is even a 1% chance a skill might apply to what you are doing, you ABSOLUTELY MUST invoke the skill.

IF A SKILL APPLIES TO YOUR TASK, YOU DO NOT HAVE A CHOICE. YOU MUST USE IT.

This is not negotiable. This is not optional. You cannot rationalize your way out of this.
</EXTREMELY-IMPORTANT>
```

### 작동 원리
1. 에이전트는 **모든 응답 전**에 스킬 적용 가능성 판단
2. 불확실하면 → 무조건 스킬 호출
3. 스킬 내용 로드 후 판단 → 적용 안 되면 무시 가능

### openclaw 적용 아이디어
```typescript
// AGENTS.md에 규칙 추가
Before any response:
  1. Check available skills
  2. If 1% match → load skill content
  3. If skill applies → follow it
  4. If not → proceed normally
```

---

## 2. 스킬 트리거 흐름

### 개념
```
메시지 수신 → 스킬 검사 → 스킬 호출 → 체크리스트 생성 → 실행
```

### 근거 코드
**파일**: `skills/using-superpowers/SKILL.md`  
**라인**: 26-45

```dot
digraph skill_flow {
    "User message received" [shape=doublecircle];
    "Might any skill apply?" [shape=diamond];
    "Invoke Skill tool" [shape=box];
    "Announce: 'Using [skill] to [purpose]'" [shape=box];
    "Has checklist?" [shape=diamond];
    "Create TodoWrite todo per item" [shape=box];
    "Follow skill exactly" [shape=box];
    "Respond (including clarifications)" [shape=doublecircle];

    "User message received" -> "Might any skill apply?";
    "Might any skill apply?" -> "Invoke Skill tool" [label="yes, even 1%"];
    "Might any skill apply?" -> "Respond (including clarifications)" [label="definitely not"];
    "Invoke Skill tool" -> "Announce: 'Using [skill] to [purpose]'";
    ...
}
```

### 핵심 단계
1. **스킬 호출 선언**: "Using [skill] to [purpose]" 명시적 발표
2. **체크리스트 확인**: 스킬에 체크리스트 있으면 TodoWrite 자동 생성
3. **정확히 실행**: Rigid 스킬은 예외 없이 따름

### openclaw 적용
```yaml
# AGENTS.md 규칙
When skill triggered:
  1. Announce: "Using {skill} for {purpose}"
  2. If skill has checklist → create memory/checklist-{timestamp}.md
  3. Execute skill rules strictly (Rigid) or adapt (Flexible)
  4. Report progress to Slack thread
```

---

## 3. Red Flags (합리화 방지 시스템)

### 개념
**에이전트가 스킬을 건너뛰려는 합리화를 사전 차단**

### 근거 코드
**파일**: `skills/using-superpowers/SKILL.md`  
**라인**: 47-62

```markdown
## Red Flags

These thoughts mean STOP—you're rationalizing:

| Thought | Reality |
|---------|---------|
| "This is just a simple question" | Questions are tasks. Check for skills. |
| "I need more context first" | Skill check comes BEFORE clarifying questions. |
| "Let me explore the codebase first" | Skills tell you HOW to explore. Check first. |
| "I can check git/files quickly" | Files lack conversation context. Check for skills. |
| "Let me gather information first" | Skills tell you HOW to gather information. |
| "This doesn't need a formal skill" | If a skill exists, use it. |
| "I remember this skill" | Skills evolve. Read current version. |
| "This doesn't count as a task" | Action = task. Check for skills. |
| "The skill is overkill" | Simple things become complex. Use it. |
| "I'll just do this one thing first" | Check BEFORE doing anything. |
| "This feels productive" | Undisciplined action wastes time. Skills prevent this. |
| "I know what that means" | Knowing the concept ≠ using the skill. Invoke it. |
```

### 작동 메커니즘
- **사전 정의된 합리화 패턴** → 에이전트가 "이런 생각 = 위험"으로 학습
- **Reality 컬럼** → 올바른 행동 명시

### openclaw 적용
```markdown
# IDENTITY.md에 추가
## 스킬 사용 Red Flags (절대 금지)
- "간단한 작업이니까 스킬 건너뛰기" → 간단할수록 스킬 필요
- "컨텍스트 먼저 확인하고" → 스킬이 컨텍스트 수집 방법 제공
- "이 스킬 기억나는데" → 스킬은 업데이트됨, 다시 읽기
- "일단 해보고" → 스킬 확인 먼저, 행동은 나중
```

---

## 4. 스킬 우선순위 시스템

### 개념
**여러 스킬 적용 가능 시 우선순위 결정**

### 근거 코드
**파일**: `skills/using-superpowers/SKILL.md`  
**라인**: 64-72

```markdown
## Skill Priority

When multiple skills could apply, use this order:

1. **Process skills first** (brainstorming, debugging) - these determine HOW to approach the task
2. **Implementation skills second** (frontend-design, mcp-builder) - these guide execution

"Let's build X" → brainstorming first, then implementation skills.
"Fix this bug" → debugging first, then domain-specific skills.
```

### 우선순위 원칙
```
Process Skills (방법론)
    ↓
Implementation Skills (구체 실행)
```

### 예시
| 요청 | 1순위 | 2순위 |
|------|-------|-------|
| "Let's build X" | brainstorming | frontend-design |
| "Fix this bug" | systematic-debugging | 도메인별 스킬 |
| "Refactor Y" | test-driven-development | code-patterns |

### openclaw 적용
```typescript
// Skill priority mapping
const SKILL_PRIORITY = {
  process: ['brainstorming', 'systematic-debugging', 'test-driven-development'],
  implementation: ['frontend-design', 'mcp-builder', 'api-design']
};

// When multiple skills match → pick highest priority
function selectSkill(candidates: string[]): string {
  for (const category of [SKILL_PRIORITY.process, SKILL_PRIORITY.implementation]) {
    for (const skill of category) {
      if (candidates.includes(skill)) return skill;
    }
  }
  return candidates[0]; // fallback
}
```

---

## 5. Rigid vs Flexible 스킬 타입

### 개념
- **Rigid**: 정확히 따름, 예외 없음 (TDD, debugging)
- **Flexible**: 원칙 적용, 컨텍스트 반영 (patterns)

### 근거 코드
**파일**: `skills/using-superpowers/SKILL.md`  
**라인**: 74-78

```markdown
## Skill Types

**Rigid** (TDD, debugging): Follow exactly. Don't adapt away discipline.

**Flexible** (patterns): Adapt principles to context.

The skill itself tells you which.
```

### Rigid 예시: TDD 철칙
**파일**: `skills/test-driven-development/SKILL.md`  
**라인**: 26-36

```markdown
## The Iron Law

NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST

Write code before the test? Delete it. Start over.

**No exceptions:**
- Don't keep it as "reference"
- Don't "adapt" it while writing tests
- Don't look at it
- Delete means delete

Implement fresh from tests. Period.
```

### openclaw 적용
```markdown
# SKILL.md 메타데이터에 타입 명시
---
name: test-driven-development
type: rigid  # 또는 flexible
enforcement: strict  # 위반 시 즉시 중단
---

# AGENTS.md 규칙
When skill type=rigid:
  - Follow exactly, no exceptions
  - Violation detected → immediate stop + restart
  
When skill type=flexible:
  - Apply principles to context
  - Adaptation allowed
```

---

## 6. 체크리스트 자동 생성 (TodoWrite)

### 개념
**스킬에 체크리스트 있으면 → TodoWrite 자동 생성 → 진행 추적**

### 근거 코드
**파일**: `skills/using-superpowers/SKILL.md`  
**라인**: 36-39

```dot
"Has checklist?" [shape=diamond];
"Create TodoWrite todo per item" [shape=box];

"Has checklist?" -> "Create TodoWrite todo per item" [label="yes"];
"Has checklist?" -> "Follow skill exactly" [label="no"];
```

### 실제 사용 예시
**파일**: `skills/subagent-driven-development/SKILL.md`  
**라인**: 138-140

```markdown
[Read plan file once: docs/plans/feature-plan.md]
[Extract all 5 tasks with full text and context]
[Create TodoWrite with all tasks]
```

### openclaw 적용
```typescript
// Skill 파싱 시 체크리스트 추출
function parseSkillChecklist(skillContent: string): string[] {
  const checklistRegex = /^- \[ \] (.+)$/gm;
  return [...skillContent.matchAll(checklistRegex)].map(m => m[1]);
}

// memory/checklist-{timestamp}.md 생성
function createChecklist(tasks: string[]) {
  const content = tasks.map((task, i) => `- [ ] ${task}`).join('\n');
  fs.writeFileSync(`memory/checklist-${Date.now()}.md`, content);
}

// Slack 진행률 보고
function reportProgress(checklist: string) {
  const completed = checklist.match(/- \[x\]/gi)?.length || 0;
  const total = checklist.match(/- \[ \]/gi)?.length || 0;
  return `진행률: ${completed}/${total} (${Math.round(completed/total*100)}%)`;
}
```

---

## 7. 스킬 체인 실행 (연쇄 트리거)

### 개념
**한 스킬 완료 → 다음 스킬 자동 트리거 → 전체 워크플로우 자동화**

### 근거 코드
**파일**: `skills/brainstorming/SKILL.md`  
**라인**: 53-60

```markdown
## After the Design

**Documentation:**
- Write the validated design to `docs/plans/YYYY-MM-DD-<topic>-design.md`
- Use elements-of-style:writing-clearly-and-concisely skill if available
- Commit the design document to git

**Implementation (if continuing):**
- Ask: "Ready to set up for implementation?"
- Use superpowers:using-git-worktrees to create isolated workspace
- Use superpowers:writing-plans to create detailed implementation plan
```

### 체인 흐름
```
brainstorming 완료
    ↓
design.md 저장
    ↓
"Ready for implementation?" 질문
    ↓
using-git-worktrees 트리거
    ↓
writing-plans 트리거
    ↓
subagent-driven-development 트리거
```

### openclaw 적용
```yaml
# AGENTS.md 워크플로우 체인 정의
workflows:
  feature-development:
    - skill: brainstorming
      output: docs/plans/{date}-design.md
      next: confirm-implementation
    
    - skill: using-git-worktrees
      trigger: confirm-implementation
      next: writing-plans
    
    - skill: writing-plans
      output: docs/plans/{date}-plan.md
      next: subagent-driven-development
    
    - skill: subagent-driven-development
      progress-report: slack-thread
      next: finishing-a-development-branch

# Slack 진행 보고
on-workflow-step-complete:
  action: message
  channel: slack
  target: C06K7BU6EBS
  threadId: {workflow-thread}
  message: |
    ✅ {skill} 완료
    - 산출물: {output-file}
    - 다음 단계: {next-skill}
```

---

## 8. 강제 실행 메커니즘 (Rigid Skills)

### 개념
**규칙 위반 감지 → 강제 삭제 → 처음부터 다시**

### 근거 코드
**파일**: `skills/test-driven-development/SKILL.md`  
**라인**: 26-36

```markdown
## The Iron Law

NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST

Write code before the test? Delete it. Start over.

**No exceptions:**
- Don't keep it as "reference"
- Don't "adapt" it while writing tests
- Don't look at it
- Delete means delete

Implement fresh from tests. Period.
```

### 위반 감지 예시
**파일**: `skills/test-driven-development/SKILL.md`  
**라인**: RED-GREEN-REFACTOR 섹션

```markdown
### RED - Write Failing Test
...

### Verify RED - Watch It Fail

**MANDATORY. Never skip.**

npm test path/to/test.test.ts

Confirm:
- Test fails (not errors)
- Failure message is expected
- Fails because feature missing (not typos)

**Test passes?** You're testing existing behavior. Fix test.
```

### openclaw 적용
```typescript
// TDD 위반 감지
function detectTDDViolation(gitLog: string): boolean {
  const commits = parseGitLog(gitLog);
  
  for (let i = 0; i < commits.length; i++) {
    const commit = commits[i];
    
    // 프로덕션 코드 커밋인데 테스트 커밋이 없으면
    if (isProductionCode(commit) && !hasTestCommit(commits, i)) {
      return true; // 위반
    }
  }
  
  return false;
}

// 위반 시 강제 조치
if (detectTDDViolation(gitLog)) {
  exec('git reset --hard HEAD~1'); // 마지막 커밋 삭제
  message.send({
    channel: 'slack',
    target: 'C06K7BU6EBS',
    message: '❌ TDD 위반 감지: 테스트 없이 프로덕션 코드 커밋. 롤백 완료. 테스트부터 작성하세요.'
  });
}
```

---

## 9. 구현 세부사항: Claude Code 플러그인 시스템

### 개념
**Claude Code의 플러그인 API를 통해 스킬 시스템 구현**

### 추정 구조 (코드 미확인, 동작 기반 추정)
```typescript
// Claude Code Plugin API (추정)
interface ClaudeCodePlugin {
  name: string;
  version: string;
  
  // 스킬 등록
  registerSkills(skills: Skill[]): void;
  
  // Tool 등록 (Skill tool)
  registerTool(tool: Tool): void;
  
  // 에이전트 루프 훅
  beforeResponse(hook: (context: Context) => void): void;
}

// Superpowers 플러그인 (추정 구현)
export default function superpowersPlugin(api: ClaudeCodePlugin) {
  // 모든 스킬 등록
  const skills = loadSkillsFromDir('./skills');
  api.registerSkills(skills);
  
  // Skill tool 등록
  api.registerTool({
    name: 'Skill',
    description: 'Load and execute a skill',
    parameters: {
      name: { type: 'string', description: 'Skill name' }
    },
    execute: async (params) => {
      const skill = findSkill(params.name);
      return skill.content; // SKILL.md 내용 반환
    }
  });
  
  // 응답 전 훅: 스킬 검사 강제
  api.beforeResponse(async (context) => {
    const message = `
Before responding, check if any skill applies (even 1% chance).
Available skills: ${getSkillNames().join(', ')}

If a skill applies:
1. Invoke Skill tool with skill name
2. Announce: "Using [skill] to [purpose]"
3. Follow skill exactly
`;
    context.addSystemMessage(message);
  });
}
```

### openclaw 적용
```typescript
// openclaw 플러그인으로 구현
// packages/extensions/superpowers-openclaw/index.ts

export default function superpowersOpenclaw(api: OpencławPluginAPI) {
  // 1. 스킬 디렉토리 로드
  const skillsDir = path.join(__dirname, '../../../skills');
  const skills = loadSkills(skillsDir);
  
  // 2. 각 스킬을 AGENTS.md에 주입 (before_agent_start 훅)
  api.registerHook('before_agent_start', async (context) => {
    const skillsXML = generateSkillsXML(skills);
    context.systemPrompt.addSection('Available Skills', skillsXML);
    
    // 1% rule 강제
    context.systemPrompt.addRule(
      'Before any response, check if any skill applies (even 1%). ' +
      'If yes, announce "Using [skill] for [purpose]" and follow it exactly.'
    );
  });
  
  // 3. 스킬 체크리스트 → memory/ 자동 기록
  api.registerHook('after_tool_call', async (context, result) => {
    if (result.tool === 'skill_load' && result.checklist) {
      const checklistPath = `memory/checklist-${Date.now()}.md`;
      fs.writeFileSync(checklistPath, result.checklist);
      
      // Slack 알림
      await api.message.send({
        channel: 'slack',
        target: 'C06K7BU6EBS',
        message: `📋 체크리스트 생성: ${checklistPath}`
      });
    }
  });
  
  // 4. Rigid skill 위반 감지
  api.registerHook('before_tool_call', async (context, tool) => {
    if (tool.name === 'exec' && context.currentSkill?.type === 'rigid') {
      // TDD 체크: 테스트 파일 생성 전에 프로덕션 코드 작성 시도?
      if (isProductionCode(tool.command) && !context.testFileCreated) {
        throw new Error('TDD violation: Write test first!');
      }
    }
  });
}
```

---

## 10. 핵심 인사이트 요약

### 자동 트리거 메커니즘의 본질
1. **강제 실행**: 선택이 아닌 의무 (1% rule)
2. **합리화 차단**: Red Flags로 우회 방지
3. **우선순위 명확**: Process → Implementation
4. **타입 구분**: Rigid (정확히) vs Flexible (적응)
5. **체크리스트 자동화**: 진행 추적 자동화
6. **체인 실행**: 스킬 → 스킬 자동 연결

### openclaw 적용 핵심 전략
```markdown
1. AGENTS.md에 1% rule 추가
2. IDENTITY.md에 Red Flags 추가
3. 스킬별 type (rigid/flexible) 메타데이터
4. memory/checklist-{timestamp}.md 자동 생성
5. Slack 진행 보고 (워크플로우 체인 시각화)
6. GitHub issue 라벨 → 스킬 자동 매핑
```

---

## 다음 단계: 03-application-plan.md
- openclaw 구체 적용 계획
- 구현 우선순위
- 팀 토론 포인트
