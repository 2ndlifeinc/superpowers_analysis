# Phase 1: 구현 프롬프트 - 자동 트리거 워크플로우

## 목적
**OpenClaw에 "1% Rule + 스킬 자동 트리거" 메커니즘 구현**

이 문서는 AI 에이전트 또는 개발자에게 전달할 **실행 가능한 프롬프트**입니다.

---

## 프롬프트 1: AGENTS.md 업데이트

### 작업 설명
OpenClaw의 `AGENTS.md` 파일에 다음 섹션을 추가하세요.

### 위치
`## Skills` 섹션을 찾아서 **전체 교체**

### 새 내용

```markdown
## Skills (mandatory recall step)

### 🚨 1% Rule (EXTREMELY IMPORTANT)
**If there is even a 1% chance a skill applies, you MUST invoke it.**

Before ANY response:
1. Check available skills list (<available_skills> in prompt)
2. If ANY skill might apply (even 1%) → Read that SKILL.md with `read` tool
3. After reading: decide if it truly applies
4. If yes → Follow it EXACTLY (see Rigid vs Flexible below)
5. If no → Proceed normally

**Why this matters**: Skipping skills leads to:
- Missed best practices
- Incomplete work
- Preventable bugs
- Team confusion

### Rigid vs Flexible Skills
- **Rigid**: Follow EXACTLY, no exceptions
  - Examples: TDD, Git workflows, Systematic Debugging
  - Reason: These exist to prevent common mistakes
- **Flexible**: Adapt to context, use judgment
  - Examples: Brainstorming, Planning
  - Reason: These guide your thinking, not dictate steps

*(Default: Rigid unless SKILL.md explicitly says "Type: Flexible")*

### Red Flags (Rationalization Blockers)
❌ If you think any of these, STOP and re-check the skill:

1. **"This doesn't quite match"** → Reality: 1% match means you MUST use it
2. **"I'll do it my way"** → Reality: Rigid skills have no exceptions
3. **"I'll skip this step"** → Reality: Every step exists for a reason
4. **"I know better"** → Reality: The skill embeds team wisdom
5. **"This seems excessive"** → Reality: Thoroughness prevents bugs
6. **"I'll combine steps"** → Reality: Steps are sequential for a reason
7. **"This is obvious"** → Reality: Obvious things get missed
8. **"I'll come back to this"** → Reality: You won't
9. **"The user won't notice"** → Reality: They will, eventually
10. **"This will take too long"** → Reality: Doing it wrong takes longer
11. **"I already know the answer"** → Reality: Verify anyway

### When Skill Triggered

#### 1. Announce (required)
```
Using `{skill-name}` to {purpose}
```
Example: "Using `github` to check issue status"

#### 2. Create Checklist (if skill has one)
- **File**: `memory/checklist-{YYYY-MM-DD-HHMM}-{skill-name}.md`
- **Content**: Copy checklist from SKILL.md + add progress log
- **Purpose**: Track completion, enable resume if interrupted

Example structure:
```markdown
# GitHub Skill - Check issue #123

**Started**: 2026-02-09 18:30 KST
**Context**: Slack thread https://...

---

## Checklist

- [x] Step 1: Identify repo
- [x] Step 2: Run `gh issue view 123`
- [ ] Step 3: Parse output
- [ ] Step 4: Report findings

---

## Progress Log

### 18:30 - Started
- Repo: openclaw/openclaw

### 18:32 - Step 2 complete
- Output: Issue is open, assigned to @user
```

#### 3. Report Progress (if long-running)
**Conditions for Slack reporting**:
- Skill has 5+ steps
- Estimated time > 10 minutes
- User explicitly asked for updates

**Report format**:
- Initial: "Using `{skill}`, {N} steps expected"
- Mid-progress (every 2-3 steps): "✅ Step X complete, next: Y"
- Final: "✅ `{skill}` complete, result: ..."

**Where to report**:
- If in Slack thread → reply to that thread
- If in main session → create new summary

#### 4. Execute Strictly (Rigid) or Adapt (Flexible)
- **Rigid**: Follow every step, don't skip, don't reorder
- **Flexible**: Use as guide, apply judgment

### Example: Skill Trigger Flow

**User message**: "Why isn't my cron job running?"

**Your thought process**:
1. Check available skills → see `systematic-debugging`
2. "Debugging" + "problem investigation" → 10% match? Yes!
3. Read `skills/systematic-debugging/SKILL.md`
4. Confirm: This is definitely a debugging task
5. Announce: "Using `systematic-debugging` to investigate cron issue"
6. Create checklist: `memory/checklist-2026-02-09-1830-systematic-debugging.md`
7. Execute Phase 1: Observe symptoms
   - Check cron logs
   - Verify cron schedule syntax
   - Test cron job manually
8. Report: "✅ Phase 1 complete, found issue: syntax error in schedule"
9. Execute Phase 2-4: ...
10. Final report with fix + prevention

**Without 1% Rule** (what NOT to do):
```
User: "Why isn't my cron job running?"
Agent: [immediately guesses] "Probably a syntax error, try this..."
Result: Missed root cause (permissions issue), problem recurs
```

**With 1% Rule** (correct):
```
User: "Why isn't my cron job running?"
Agent: "Using `systematic-debugging` to investigate..."
       → Phase 1: Observed 3 symptoms
       → Phase 2: Formed 2 hypotheses
       → Phase 3: Verified (was permissions, not syntax)
       → Phase 4: Fixed + added monitoring
Result: Root cause found, prevented recurrence
```

---

## Skills

Skills provide your tools. When you need one, check its `SKILL.md`. Keep local notes (camera names, SSH details, voice preferences) in `TOOLS.md`.

*(rest of existing Skills section...)*
```

### 검증 방법
1. 파일을 저장하세요
2. OpenClaw 재시작 (새 프롬프트 반영)
3. 테스트 메시지 전송: "Check GitHub issue #1 on user/repo"
4. 예상 응답: "Using `github` to check issue #1"

---

## 프롬프트 2: 기존 스킬에 메타데이터 추가

### 작업 설명
모든 SKILL.md 파일 상단에 표준 메타데이터를 추가하세요.

### 대상 파일
- `skills/github/SKILL.md`
- `skills/gog/SKILL.md`
- `skills/himalaya/SKILL.md`
- `skills/notion/SKILL.md`
- *(기타 모든 skills/ 폴더 내 SKILL.md)*

### 추가할 메타데이터 템플릿

```markdown
---
**Type**: Rigid | Flexible
**Trigger**: [1% 판단 기준 - 어떤 상황에서 이 스킬을 써야 하는가?]
**Checklist**: Yes | No
---

## When to Use (1% Rule Check)
- [ ] 조건 1
- [ ] 조건 2
- [ ] 조건 3

If ANY checkbox applies → USE THIS SKILL

---

*(기존 내용 이어짐)*
```

### 예시: GitHub Skill

#### Before
```markdown
# GitHub CLI Skill

Use `gh` CLI to interact with GitHub...
```

#### After
```markdown
# GitHub CLI Skill

---
**Type**: Rigid
**Trigger**: Any mention of GitHub repositories, issues, pull requests, or CI runs
**Checklist**: Yes
---

## When to Use (1% Rule Check)
- [ ] User mentions GitHub repo name
- [ ] User asks about issues or PRs
- [ ] User needs to check CI/Actions status
- [ ] Any git operation beyond local commits
- [ ] User mentions "merge", "review", "CI failed"

If ANY checkbox applies → USE THIS SKILL

---

## Checklist
1. [ ] Identify target repository (org/repo)
2. [ ] Determine command (`gh issue`, `gh pr`, `gh run`, etc.)
3. [ ] Execute command with proper flags
4. [ ] Parse output (JSON preferred: `--json`)
5. [ ] Report findings (Slack thread if applicable)

---

Use `gh` CLI to interact with GitHub...
*(rest of existing content)*
```

### 우선순위
**P0 (즉시 추가)**:
1. `github/SKILL.md`
2. `systematic-debugging/SKILL.md` (만약 있다면)
3. `healthcheck/SKILL.md`

**P1 (이후 추가)**:
4. 나머지 모든 skills

### 검증 방법
1. 메타데이터 추가 후 해당 스킬 테스트
2. "When to Use" 조건 중 하나라도 만족하면 스킬이 트리거되는지 확인

---

## 프롬프트 3: 체크리스트 생성 함수 (간단한 스크립트)

### 작업 설명
체크리스트 파일을 자동 생성하는 간단한 함수를 작성하세요.

### 위치
`scripts/create-checklist.js` (새 파일)

### 코드

```javascript
#!/usr/bin/env node
/**
 * Create a checklist file from a skill's checklist section
 * Usage: node scripts/create-checklist.js <skill-name> <purpose>
 */

const fs = require('fs');
const path = require('path');

function formatTimestamp(date) {
  const y = date.getFullYear();
  const m = String(date.getMonth() + 1).padStart(2, '0');
  const d = String(date.getDate()).padStart(2, '0');
  const h = String(date.getHours()).padStart(2, '0');
  const min = String(date.getMinutes()).padStart(2, '0');
  return `${y}-${m}-${d}-${h}${min}`;
}

function extractChecklist(skillContent) {
  const checklistMatch = skillContent.match(/## Checklist\n([\s\S]*?)(?=\n## |\n---|\n$)/);
  if (!checklistMatch) return null;
  
  return checklistMatch[1]
    .split('\n')
    .filter(line => line.trim().startsWith('-') || line.trim().startsWith('*'))
    .map(line => line.replace(/^[\s-*]+/, '- [ ] '))
    .join('\n');
}

function createChecklist(skillName, purpose, contextLink = '') {
  const skillPath = path.join(__dirname, '../skills', skillName, 'SKILL.md');
  
  if (!fs.existsSync(skillPath)) {
    console.error(`Skill not found: ${skillPath}`);
    process.exit(1);
  }
  
  const skillContent = fs.readFileSync(skillPath, 'utf-8');
  const checklist = extractChecklist(skillContent);
  
  if (!checklist) {
    console.log(`No checklist found in ${skillName}/SKILL.md`);
    return null;
  }
  
  const timestamp = formatTimestamp(new Date());
  const filename = `checklist-${timestamp}-${skillName}.md`;
  const filepath = path.join(__dirname, '../memory', filename);
  
  const content = `# ${skillName} - ${purpose}

**Started**: ${new Date().toISOString().replace('T', ' ').slice(0, 19)} KST
**Context**: ${contextLink || 'N/A'}

---

## Checklist

${checklist}

---

## Progress Log

### ${new Date().toTimeString().slice(0,5)} - Started
- Read SKILL.md
- Created this checklist

`;

  fs.writeFileSync(filepath, content, 'utf-8');
  console.log(`✅ Created: ${filepath}`);
  return filepath;
}

// CLI
if (require.main === module) {
  const [,, skillName, ...purposeParts] = process.argv;
  if (!skillName) {
    console.error('Usage: node create-checklist.js <skill-name> <purpose>');
    process.exit(1);
  }
  const purpose = purposeParts.join(' ');
  createChecklist(skillName, purpose);
}

module.exports = { createChecklist };
```

### 사용 예시

```bash
# 터미널에서 직접 실행
node scripts/create-checklist.js github "Check issue #123"

# 출력:
# ✅ Created: /path/to/memory/checklist-2026-02-09-1830-github.md
```

### 검증 방법
1. 스크립트 실행
2. `memory/` 폴더에 파일 생성 확인
3. 체크리스트 형식 검증

---

## 프롬프트 4: Slack 진행 보고 헬퍼 (기존 도구 활용)

### 작업 설명
Slack 진행 보고를 간편하게 할 수 있는 헬퍼 함수 작성

### 위치
`TOOLS.md` 또는 `memory/helpers/slack-progress.md` (새 파일)

### 내용

```markdown
# Slack 진행 보고 헬퍼

## 언제 보고할까?

**보고 필요**:
- [ ] 5단계 이상 스킬 실행 중
- [ ] 예상 소요 시간 > 10분
- [ ] 팀원이 명시적으로 진행 상황 요청
- [ ] 여러 서브에이전트가 협업하는 작업

**보고 불필요**:
- [ ] 3단계 이하 단순 작업
- [ ] 1-2분 안에 끝나는 작업
- [ ] 내부 자동화 (cron 등)

## 보고 템플릿

### 초기 보고
```markdown
🔄 **Using `{skill-name}`**

**목적**: {purpose}
**예상 단계**: {N}단계
**체크리스트**: memory/checklist-{timestamp}-{skill-name}.md

진행 상황은 이 스레드에 업데이트합니다 💪
```

### 중간 보고 (2-3단계마다)
```markdown
✅ {Step X} 완료
📝 발견사항: {key findings}
⏭️ 다음: {Step Y}
```

### 완료 보고
```markdown
✅ **`{skill-name}` 완료**

**소요 시간**: {X}분
**결과 요약**: {summary}
**산출물**: [링크]

체크리스트: memory/checklist-{timestamp}-{skill-name}.md
```

### 에러 보고
```markdown
❌ **`{skill-name}` 중단**

**단계**: Step {X}
**에러**: {error message}
**다음 조치**: {what to do next}

체크리스트: memory/checklist-{timestamp}-{skill-name}.md
```

## 실제 사용 예시

**시나리오**: GitHub issue 분석 (5단계)

**Step 0: 초기 보고**
```
🔄 **Using `github`**

**목적**: Analyze issue #123 dependencies
**예상 단계**: 5단계
**체크리스트**: memory/checklist-2026-02-09-1830-github.md

진행 상황은 이 스레드에 업데이트합니다 💪
```

**Step 2: 중간 보고**
```
✅ Step 2 완료
📝 발견사항: Issue has 3 linked PRs, 2 merged, 1 open
⏭️ 다음: Check CI status for open PR
```

**Step 5: 완료 보고**
```
✅ **`github` 완료**

**소요 시간**: 3분
**결과 요약**: Issue #123 is blocked by PR #456 (CI failing)
**산출물**: [GitHub link]

체크리스트: memory/checklist-2026-02-09-1830-github.md
```

## 구현 (message tool 활용)

```typescript
// 의사코드
async function reportProgress(
  stage: 'start' | 'progress' | 'complete' | 'error',
  details: {
    skillName: string;
    purpose?: string;
    steps?: { current: number; total: number; message: string };
    result?: string;
    checklistPath?: string;
  }
) {
  const threadId = getCurrentThreadId(); // Slack thread
  
  let message = '';
  switch (stage) {
    case 'start':
      message = `🔄 **Using \`${details.skillName}\`**\n\n` +
                `**목적**: ${details.purpose}\n` +
                `**체크리스트**: ${details.checklistPath}\n\n` +
                `진행 상황은 이 스레드에 업데이트합니다 💪`;
      break;
    
    case 'progress':
      message = `✅ Step ${details.steps.current}/${details.steps.total} 완료\n` +
                `📝 ${details.steps.message}`;
      break;
    
    case 'complete':
      message = `✅ **\`${details.skillName}\` 완료**\n\n` +
                `**결과**: ${details.result}\n` +
                `**체크리스트**: ${details.checklistPath}`;
      break;
    
    case 'error':
      message = `❌ **\`${details.skillName}\` 중단**\n\n` +
                `**에러**: ${details.result}\n` +
                `**체크리스트**: ${details.checklistPath}`;
      break;
  }
  
  await messageTool.send({
    channel: 'slack',
    channelId: getCurrentChannelId(),
    threadId,
    message
  });
}
```
```

### 검증 방법
1. 5단계 이상 스킬 실행
2. 각 단계마다 보고 함수 호출
3. Slack 스레드에 메시지 확인

---

## 완료 기준

### Phase 1 구현 완료 조건
- [x] `03-application-plan.md` 작성
- [x] `04-implementation-prompt.md` 작성 (이 파일)
- [ ] `AGENTS.md` 업데이트 (1% Rule + Red Flags)
- [ ] 2-3개 스킬에 메타데이터 추가 (Type/Trigger/Checklist)
- [ ] 체크리스트 생성 스크립트 작성 (`scripts/create-checklist.js`)
- [ ] Slack 보고 헬퍼 문서 작성 (`TOOLS.md` or `memory/helpers/`)
- [ ] 테스트 (GitHub skill 트리거 확인)
- [ ] Slack #개발-통합채널에 토론 요청 (Phase 1 피드백)

### 다음 Phase
**Phase 2**: Subagent-Driven Development (SDD) 패턴 분석
- 02-subagent-orchestration/ 폴더 생성
- 01-diagram.md ~ 04-implementation-prompt.md 작성

---

## 참고 링크
- Superpowers 원본: https://github.com/obra/Superpowers
- OpenClaw 코드: /Users/secondlife/deployments/openclaw
- 현재 AGENTS.md: /Users/secondlife/.openclaw/workspace/AGENTS.md
