# Guardian — Universal Unattended Automated Coding Workflow

A complete automated coding workflow with 10 professional AI Agents, 10-stage pipeline processes, and human final approval.

## Directory Structure

```
guardian-skill/
├── SKILL.md                               # Skill definition and design philosophy
├── README.md                              # This file (Chinese)
├── README_en.md                           # This file (English)
├── agents/                                # Universal Agent prompts (platform-agnostic)
│   ├── guardian-runner.md                 # Primary Agent: workflow orchestration
│   ├── refactor.md                        # Primary Agent: safe refactoring
│   ├── review.md                          # Primary Agent: PR review coordination
│   ├── test-writer.md                     # Primary Agent: test generation
│   ├── prd-analyzer.md                    # Subagent: requirements analysis
│   ├── code-reviewer.md                   # Subagent: code review
│   ├── security-reviewer.md               # Subagent: security review
│   ├── test-case-writer.md                # Subagent: test case generation
│   ├── performance-checker.md             # Subagent: performance & stability review
│   └── log-diagnoser.md                   # Subagent: log diagnosis
│
└── references/                            # Reference documentation
    ├── workflow-stages.md                 # 10-stage workflow details
    ├── safety-principles.md               # Safety constraints and stop conditions
    ├── output-structure.md                # Output file specifications
    ├── agent-responsibilities.md          # Agent role definitions
    ├── risk-matrix.md                     # Risk levels and decision matrices
    └── integration-examples.md            # Platform integration guide
```

## Quick Start

### OpenCode (Recommended)

```bash
# Copy Agent files
cp guardian-skill/agents/*.md your-project/.opencode/agents/

# Configure opencode.jsonc
cat > your-project/opencode.jsonc << 'EOF'
{
  "default_agent": "guardian-runner",
  "model": "[select model]",
  "permission": {
    "edit": "ask",
    "bash": "ask"
  }
}
EOF

# Start Guardian workflow
cd your-project
opencode
# Select guardian-runner
# Paste task requirements
# Workflow runs automatically (stages 0-9)
# Generate verification report
```

### Claude Code / Hermes (Requires Adaptation)

1. Copy prompts from `agents/` to your platform
2. Create orchestration wrapper layer (see `references/integration-examples.md`)
3. Configure permission control and safety checks
4. Set temperature to 0.1 for all Agents

### Minimal Installation

```
1. Copy agent prompts from agents/ directory
2. Create primary Agent (guardian-runner)
3. Set up subagent invocation mechanism
4. Set output directory to .guardian/
5. Test on isolated branch
```

---

## 10 Agents — Role Distribution

### Primary Agents (Orchestration & Execution)

| Agent | Responsibility | Main Output |
|-------|---|---|
| **guardian-runner** | Workflow orchestration, stage management, safety gates | morning-report.md |
| **refactor** | Safe small-step refactoring, interface stability | refactor-plan.md |
| **review** | PR review coordination, multi-perspective synthesis | review-summary.md |
| **test-writer** | Test strategy design, test case generation & execution | test-result.md |

### Expert Agents (Professional Review)

| Agent | Domain | Output |
|-------|--------|--------|
| **prd-analyzer** | Requirements clarification, task card generation | task-card.md |
| **code-reviewer** | Code quality, maintainability, bug checking | code-review.md |
| **security-reviewer** | Security audit, vulnerability detection | security-review.md |
| **test-case-writer** | Test cases, automation scripts | test-plan.md |
| **performance-checker** | Performance, concurrency, resource leaks | performance-review.md |
| **log-diagnoser** | Log analysis, root cause diagnosis | diagnosis.md |

---

## 10-Stage Workflow

### Stage 0: Environment Protection
- Check current branch (cannot be main/master)
- Verify clean working directory
- Create isolated feature branch

### Stage 1: Requirements Analysis
- Call `@prd-analyzer`
- Generate task-card.md (clarify scope, goals, risks)
- Stop if Critical issues found

### Stage 2: Technical Plan
- Create implementation plan
- List modified files, steps, rollback plan
- Stop if new dependencies/database migrations required

### Stage 3: Code Changes
- Make small-step changes only (one issue category per step)
- Update change-summary.md
- Prohibit modifying .env, secrets, production configs

### Stage 4: Test Generation
- Call `@test-case-writer`
- Cover happy path, exceptions, permissions, boundaries, regressions
- Generate test-plan.md

### Stage 5: Test Execution
- Run low-risk test commands (npm test, pytest, etc.)
- Auto-fix failures up to 2 rounds maximum
- Update test-result.md

### Stage 6: Code Quality Review
- Call `@code-reviewer`
- Stop if Critical issues found
- Output code-review.md

### Stage 7: Performance Review
- Call `@performance-checker`
- Check N+1 queries, caching, concurrency, leaks
- Stop if High risk found
- Output performance-review.md

### Stage 8: Security Review
- Call `@security-reviewer`
- Stop if Critical/High issues found
- Output security-review.md

### Stage 9: Verification Report
- Synthesize all results
- List unresolved issues, blockers
- Generate morning-report.md
- Recommendation: "Recommend human review" or "Stop, reason: ..."

---

## Safety Principles

### 10 Immutable Rules

1. ❌ Never work directly on main/master — must use feature branch
2. ❌ Never auto-commit, auto-push, or auto-merge
3. ❌ Never modify .env, secrets, tokens, production configs
4. ❌ Never execute production environment commands
5. ❌ Never add unapproved dependencies
6. ❌ Never bypass tests, code review, or security review
7. 🛑 Critical/High security issues must stop
8. 🛑 Permission, billing, device control, ROM/driver changes must stop
9. 🛑 Stop if test failures exceed 2 auto-fix rounds
10. 📌 Mark all uncertain conclusions as "needs human confirmation"

### Output Files (9 Required)

```
.guardian/
├── task-card.md              # Requirements breakdown
├── plan.md                   # Technical plan
├── change-summary.md         # Code changes
├── test-plan.md              # Test strategy
├── test-result.md            # Test results
├── code-review.md            # Quality assessment
├── performance-review.md     # Performance & stability assessment
├── security-review.md        # Security review
└── morning-report.md         # Comprehensive report and recommendations
```

---

## Platform Configuration

### OpenCode (Native Support)
- ✅ Full support via `.opencode/agents/` + `opencode.jsonc`
- Agent invocation syntax: `@agent-name`
- Permissions configured in metadata (mode, edit, bash)
- Works out of the box

### Claude Code (Requires Wrapper)
- Copy prompts to `.claude/agents/`
- Use Agent tool in guardian-runner to call subagents
- Configure via CLAUDE.md or settings.json
- 🟡 Requires orchestration wrapper

### Hermes (Requires YAML Configuration)
- Copy prompts to platform
- Use native workflow orchestration
- Configure via hermes.yaml
- 🟡 Requires platform binding

---

## Recommended Launch Method

```bash
# Enter project directory
cd your-project

# Start OpenCode Guardian mode
opencode select guardian-runner

# Paste task requirements and constraints:
Enter Guardian mode and strictly follow guardian-runner workflow for the following task:

Requirements:
[Describe your requirements]

Constraints:
1. Only work on new branch ai/guardian-{task-name};
2. No commits, pushes, or merges;
3. No new dependencies;
4. No modifications to .env, production configs, or secrets;
5. Auto-fix test failures maximum 2 rounds;
6. Must generate .guardian/morning-report.md at completion;
7. If permission/data/device control risks involved, must stop and mark needs human confirmation.
```

---

## Guardian Acceptance Completion Criteria

Human review can be recommended only when all conditions are met:

- ✅ task-card.md generated with clear requirements
- ✅ plan.md generated with complete steps
- ✅ change-summary.md documents all changes
- ✅ test-plan.md covers happy path/exceptions/permissions/boundaries/regression
- ✅ test-result.md shows all tests passing
- ✅ code-review.md has no Critical issues
- ✅ performance-review.md has no blocking risks
- ✅ security-review.md has no Critical/High issues
- ✅ morning-report.md generated with clear recommendations

---

## Parameter Configuration

### Temperature Parameter

All Agents use **temperature: 0.1**:
- Low randomness, high reproducibility
- Minimal hallucination in reviews
- More stable code generation
- Optimized for engineering tasks (not creative tasks)

If your platform requires temperature configuration, set all Agents to 0.1.

---

## What This Skill Does NOT Include

- ❌ Model credentials (use your platform's authentication)
- ❌ Platform-specific configuration formats (adapt per integration guide)
- ❌ Production deployment or release (generates candidate implementation only)
- ❌ Automatic approval logic (requires human final decision)

---

## Integration Checklist

- [ ] Copy agents/ prompts to your platform
- [ ] Create primary Agent (guardian-runner)
- [ ] Connect subagent invocation mechanism (@agent or spawn)
- [ ] Set temperature to 0.1 for all Agents
- [ ] Configure output directory (.guardian/ or equivalent)
- [ ] Test on isolated branch with sample task
- [ ] Verify 9 output files generated
- [ ] Review morning-report.md before deciding to merge
- [ ] If needed, set up daily/nightly automated runs

---

## More Help

- Detailed workflow: see `references/workflow-stages.md`
- Safety constraints and stop conditions: see `references/safety-principles.md`
- Output file specifications: see `references/output-structure.md`
- Platform integration: see `references/integration-examples.md`
- Agent role definitions: see `references/agent-responsibilities.md`
- Risk level decisions: see `references/risk-matrix.md`

---

**Final Reminder:** Guardian is not fully automated—it's a hybrid model of AI assistance plus human final gatekeeping. Human review is the final and most critical safeguard.
