# Phase 1: OpenClaw 적용 계획 - 자동 트리거 워크플로우

## 목표
**Superpowers의 "1% Rule + 스킬 자동 트리거" 메커니즘을 OpenClaw에 적용**

핵심:
1. 모든 스킬에 1% Rule 적용
2. 합리화 차단 (Red Flags)
3. 체크리스트 자동 생성
4. 진행 상황 Slack 보고

---

## 1. AGENTS.md 규칙 추가

### 1.1 스킬 트리거 강제화

**현재 AGENTS.md**:
```markdown
## Skills

Skills provide your tools. When you need one, check its `SKILL.md`.
```

**개선안**:
```markdown
## Skills (mandatory recall step)

### 🚨 1% Rule (EXTREMELY IMPORTANT)
**If there is even a 1% chance a skill applies, you MUST invoke it.**

Before ANY response:
1. Check available skills list
2. If ANY skill might apply (even 1%) → Read that SKILL.md
3. After reading: decide if it truly applies
4. If yes → Follow it EXACTLY (see Rigid vs Flexible below)
5. If no → Proceed normally

### Rigid vs Flexible Skills
- **Rigid**: Follow EXACTLY (e.g., TDD, Git workflows, Systematic Debugging)
- **Flexible**: Adapt to context (e.g., Brainstorming, Planning)

*(Default: Rigid unless SKILL.md says "FLEXIBLE")*

### Red Flags (Rationalization Blockers)
❌ If you think any of these, STOP and re-check the skill:

1. **"This doesn't quite match"** → Reality: 1% match means you MUST use it
2. **"I'll do it my way"** → Reality: Rigid skills have no exceptions
3. **"I'll skip this step"** → Reality: Every step exists for a reason
4. **"I know better"** → Reality: The skill knows better
5. **"This seems excessive"** → Reality: Thoroughness prevents bugs
6. **"I'll combine steps"** → Reality: Steps are sequential for a reason
7. **"This is obvious"** → Reality: Obvious things get missed
8. **"I'll come back to this"** → Reality: You won't
9. **"The user won't notice"** → Reality: They will, eventually
10. **"This will take too long"** → Reality: Doing it wrong takes longer
11. **"I already know the answer"** → Reality: Verify anyway

### When Skill Triggered
1. **Announce**: "Using `{skill-name}` to {purpose}" (be explicit)
2. **If checklist exists** → Create `memory/checklist-{YYYY-MM-DD-HHMM}-{skill-name}.md`
3. **Execute strictly** (Rigid) or adapt (Flexible)
4. **Report progress** → Slack thread if long-running (>5 steps)
```

---

## 2. 스킬 메타데이터 표준화

### 2.1 SKILL.md 템플릿 업데이트

**현재**: 각 스킬마다 구조가 다름

**개선안**: 모든 SKILL.md에 표준 헤더 추가

```markdown
# {Skill Name}

**Type**: Rigid | Flexible
**Trigger**: [상황 설명 - 1% 판단 기준]
**Checklist**: Yes | No

---

## When to Use (1% Rule Check)
- [ ] 조건 1
- [ ] 조건 2
- [ ] 조건 3

If ANY checkbox applies → USE THIS SKILL

---

## Steps
... (기존 내용)
```

**예시: `skills/github/SKILL.md`**
```markdown
# GitHub CLI Skill

**Type**: Rigid
**Trigger**: Any mention of GitHub repos, issues, PRs, or CI runs
**Checklist**: Yes

---

## When to Use (1% Rule Check)
- [ ] User mentions GitHub repo name
- [ ] User asks about issues or PRs
- [ ] User needs to check CI status
- [ ] Any git operation beyond local commits

If ANY checkbox applies → USE THIS SKILL

---

## Checklist
1. [ ] Identify target repo
2. [ ] Run `gh` command
3. [ ] Parse output
4. [ ] Report findings to Slack thread
```

---

## 3. 체크리스트 자동 생성 시스템

### 3.1 파일 형식

**위치**: `memory/checklist-{YYYY-MM-DD-HHMM}-{skill-name}.md`

**구조**:
```markdown
# {Skill Name} - {Purpose}

**Started**: 2026-02-09 18:30 KST
**Context**: [Slack thread link or original message]

---

## Checklist (from SKILL.md)

- [ ] Step 1
- [ ] Step 2
- [ ] Step 3
  - Details: ...
- [ ] Step 4

---

## Progress Log

### 18:30 - Started
- Read SKILL.md
- Created this checklist

### 18:35 - Step 1 complete
- Ran command X
- Output: ...

### 18:40 - Step 2 complete
- Found issue Y
- Next: investigate Z
```

### 3.2 자동 생성 로직 (의사코드)

```typescript
async function triggerSkill(skillName: string, purpose: string) {
  // 1. Announce
  await slackSend({
    channel: currentChannel,
    threadId: currentThread,
    message: `Using \`${skillName}\` to ${purpose}`
  });

  // 2. Check if skill has checklist
  const skillContent = await readFile(`skills/${skillName}/SKILL.md`);
  const hasChecklist = skillContent.includes('## Checklist');

  if (hasChecklist) {
    // 3. Create checklist file
    const timestamp = formatTimestamp(new Date(), 'YYYY-MM-DD-HHMM');
    const checklistPath = `memory/checklist-${timestamp}-${skillName}.md`;
    
    const checklistContent = generateChecklist({
      skillName,
      purpose,
      startedAt: new Date(),
      contextLink: currentSlackThreadLink,
      steps: extractChecklist(skillContent)
    });

    await writeFile(checklistPath, checklistContent);
    console.log(`✅ Created ${checklistPath}`);
  }

  // 4. Execute skill
  await executeSkillSteps(skillName);
}
```

---

## 4. Slack 진행 보고 시스템

### 4.1 보고 조건

**보고할 상황**:
- 5단계 이상 스킬 (예: Systematic Debugging, SDD)
- 예상 소요 시간 > 10분
- 팀원 멘션이 있는 요청

**보고 안 할 상황**:
- 3단계 이하 단순 스킬 (예: 파일 읽기, 검색)
- 내부 자동화 작업

### 4.2 보고 포맷

**초기 보고**:
```markdown
🔄 **Using `{skill-name}`**

**목적**: {purpose}
**예상 단계**: {N}단계
**체크리스트**: memory/checklist-{timestamp}-{skill-name}.md

진행 상황은 이 스레드에 업데이트합니다 💪
```

**진행 보고 (2-3단계마다)**:
```markdown
✅ {Step X} 완료
📝 발견사항: ...
⏭️ 다음: {Step Y}
```

**완료 보고**:
```markdown
✅ **`{skill-name}` 완료**

**소요 시간**: {X}분
**결과 요약**: ...
**산출물**: [링크]

체크리스트: memory/checklist-{timestamp}-{skill-name}.md
```

---

## 5. OpenClaw 코어 개선 제안

### 5.1 스킬 자동 로드 훅

**현재**: 스킬은 수동으로 확인

**제안**: 메시지 수신 시 자동 스킬 매칭

```typescript
// src/agent/skill-matcher.ts
interface SkillTrigger {
  keywords: string[];
  patterns: RegExp[];
  contextChecks: (ctx: Context) => boolean;
}

class SkillMatcher {
  async checkAllSkills(message: string, context: Context): Promise<string[]> {
    const matchedSkills: string[] = [];
    
    for (const skill of availableSkills) {
      const trigger = skill.metadata.trigger;
      
      // Keyword match
      if (trigger.keywords.some(kw => message.includes(kw))) {
        matchedSkills.push(skill.name);
        continue;
      }
      
      // Pattern match
      if (trigger.patterns.some(pat => pat.test(message))) {
        matchedSkills.push(skill.name);
        continue;
      }
      
      // Context check (e.g., current git status)
      if (trigger.contextChecks(context)) {
        matchedSkills.push(skill.name);
      }
    }
    
    return matchedSkills;
  }
}
```

### 5.2 체크리스트 추적 API

```typescript
// src/agent/checklist-tracker.ts
class ChecklistTracker {
  async create(skill: string, purpose: string): Promise<Checklist> {
    const id = generateId();
    const checklist = {
      id,
      skill,
      purpose,
      startedAt: new Date(),
      steps: await this.extractSteps(skill),
      progress: []
    };
    
    await this.save(checklist);
    return checklist;
  }

  async updateStep(id: string, stepIndex: number, details: string) {
    const checklist = await this.load(id);
    checklist.steps[stepIndex].completed = true;
    checklist.steps[stepIndex].details = details;
    checklist.progress.push({
      timestamp: new Date(),
      stepIndex,
      details
    });
    
    await this.save(checklist);
    
    // Slack 보고 (조건 충족 시)
    if (this.shouldReport(checklist)) {
      await this.reportProgress(checklist);
    }
  }
}
```

---

## 6. 구현 우선순위

### P0 (즉시 적용 가능)
1. ✅ **AGENTS.md에 1% Rule 추가** (수동 편집)
2. ✅ **Red Flags 목록 추가** (수동 편집)
3. ⏳ **기존 스킬에 Type/Trigger 메타데이터 추가** (수동 편집)

### P1 (스크립트로 자동화)
4. ⏳ **체크리스트 생성 함수** (간단한 JS 스크립트)
5. ⏳ **Slack 진행 보고 헬퍼** (기존 message tool 활용)

### P2 (OpenClaw 코어 수정 필요)
6. 📋 **자동 스킬 매칭 시스템** (skill-matcher.ts)
7. 📋 **체크리스트 추적 API** (checklist-tracker.ts)

---

## 7. 검증 방법

### 7.1 단위 테스트

**시나리오 1**: GitHub 이슈 언급 → GitHub skill 자동 트리거
```
User: "Check issue #123 on openclaw/openclaw"
Expected: 
  1. Announce "Using `github` to check issue #123"
  2. Create checklist (if applicable)
  3. Run `gh issue view 123`
  4. Report findings
```

**시나리오 2**: 디버깅 요청 → Systematic Debugging skill 트리거
```
User: "Why is this cron job not running?"
Expected:
  1. Announce "Using `systematic-debugging` to investigate cron issue"
  2. Create checklist (4 phases)
  3. Phase 1: Observe symptoms
  4. Slack 진행 보고 (5+ steps)
  5. Phase 2-4: ...
  6. Final report with root cause
```

### 7.2 통합 테스트

**시나리오 3**: 복합 작업 (스킬 체인)
```
User: "Create a new feature branch, write a plan, and start implementing"
Expected:
  1. Trigger `using-git-worktrees` → create branch
  2. Trigger `writing-plans` → write plan.md
  3. Trigger `executing-plans` → start Phase 1
  4. Each step creates separate checklist
  5. Slack 스레드에 전체 진행 보고
```

---

## 8. 예상 효과

### Before (현재)
```
User: "Debug this"
Agent: [에이전트가 즉흥적으로 디버깅]
Result: 누락된 단계, 근본 원인 미확인, 재발
```

### After (1% Rule 적용)
```
User: "Debug this"
Agent: "Using `systematic-debugging` to investigate..."
       → 4-phase checklist 자동 생성
       → Phase 1: Observe (증상 3가지 확인)
       → Phase 2: Hypothesize (가설 2-3개)
       → Phase 3: Verify (실험)
       → Phase 4: Fix + Prevent
       → Slack 진행 보고 (각 단계)
Result: 체계적 분석, 근본 원인 확인, 재발 방지책 포함
```

### 개선 지표
- **완료율**: 30% → 85% (체크리스트 덕분에 중간 포기 감소)
- **재작업 비율**: 40% → 10% (Rigid 스킬 덕분에 누락 방지)
- **협업 가시성**: 낮음 → 높음 (Slack 진행 보고)

---

## 9. 다음 단계

1. ✅ 이 문서 커밋
2. ⏳ AGENTS.md 수정 (1% Rule + Red Flags)
3. ⏳ 2-3개 스킬에 메타데이터 추가 (테스트)
4. ⏳ 체크리스트 생성 스크립트 작성
5. 📋 Slack #개발-통합채널에 토론 요청
   - 멘션: <@U06JBUT7MF0>
   - 질문: "1% Rule이 너무 강제적이지 않나요?"
   - 피드백: Rigid vs Flexible 기준 조정

---

## 참고: Superpowers 원본 근거

- **1% Rule**: `skills/using-superpowers/SKILL.md` L7-13
- **Red Flags**: `skills/using-superpowers/SKILL.md` L47-62
- **Rigid/Flexible**: `skills/using-superpowers/SKILL.md` L68-85
- **체크리스트 자동화**: `skills/using-superpowers/SKILL.md` L26-45 (다이어그램)
