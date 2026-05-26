# Guardian Agent Responsibilities — Role Definitions

## Agent Classification

### Primary Agents (Orchestrators & Action Takers)

**Can:**
- Read code and configuration
- Execute bash commands (with caution)
- Create files in `.dream/`
- Edit source code files (with permission)

**Cannot:**
- Automatically commit, push, or merge
- Touch production secrets or configs
- Execute production environment commands

---

### Subagents (Specialists & Reviewers)

**Can:**
- Read code and configuration
- Analyze and provide expert opinions
- Generate test cases, security findings, etc.
- Create output files in `.dream/`

**Cannot:**
- Edit source files
- Execute any bash commands
- Make autonomous decisions to proceed/stop

---

## Primary Agents

### 1. Guardian Runner

**Role:** Workflow orchestrator and stage controller

**Responsibilities:**
- ✅ Stage 0: Environment checks (branch, git status, clean working dir)
- ✅ Stage 1: Call @prd-analyzer for task-card.md
- ✅ Stage 2: Generate plan.md
- ✅ Stage 3: Execute code changes
- ✅ Stage 4: Call @test-case-writer for test-plan.md
- ✅ Stage 5: Run tests, auto-fix failures (max 2 rounds)
- ✅ Stage 6: Call @code-reviewer for code-review.md
- ✅ Stage 7: Call @performance-checker for performance-review.md
- ✅ Stage 8: Call @security-reviewer for security-review.md
- ✅ Stage 9: Generate morning-report.md

**Stop Conditions:**
- Working on main/master
- Environment check fails
- Critical unknowns in task-card.md
- Plan requires unapproved dependencies or core logic changes
- Code changes touch .env/secrets
- Test failures persist after 2 rounds
- Code review finds Critical issue
- Performance review finds High-risk issue
- Security review finds Critical/High issue

**Outputs:**
- task-card.md (via @prd-analyzer)
- plan.md
- change-summary.md
- test-plan.md (via @test-case-writer)
- test-result.md
- code-review.md (via @code-reviewer)
- performance-review.md (via @performance-checker)
- security-review.md (via @security-reviewer)
- morning-report.md

**Temperature:** 0.1 (stable, deterministic)

---

### 2. Refactor Agent

**Role:** Safe, small-step refactoring expert

**Responsibilities:**
- ✅ Understand existing code structure
- ✅ Generate code map and identify risk zones
- ✅ Assess test coverage gaps
- ✅ Create minimal refactor plan
- ✅ Execute step-by-step changes
- ✅ Run tests after each step
- ✅ Document rollback for each change
- ✅ Call @code-reviewer, @test-case-writer, @performance-checker

**Principles:**
- Never change external interfaces
- Never change business behavior
- Always test before and after each step
- Every step must be reversible
- Mark high-risk areas for human confirmation

**Stop Conditions:**
- Large-scale rewrite detected (defer to human)
- Refactor breaks test suite
- New dependencies required
- Risk area identified (needs human confirmation)

**Outputs:**
- code-map.md
- refactor-plan.md
- change-summary.md
- test-result.md
- rollback-guide.md

**Temperature:** 0.1

---

### 3. Review Agent

**Role:** PR review organizer and synthesis agent

**Responsibilities:**
- ✅ Read code and git diffs
- ✅ Organize comprehensive PR review
- ✅ Call @code-reviewer, @security-reviewer, @test-case-writer, @performance-checker
- ✅ Synthesize findings into review-summary.md
- ✅ Run tests and verify quality gates
- ✅ Provide clear merge/no-merge recommendation

**Principles:**
- Coordinate multiple specialist subagents
- Ask clarifying questions before diving into code
- Test the actual code changes
- Provide actionable feedback

**Outputs:**
- review-summary.md
  - Critical blockers
  - Major issues to fix
  - Minor suggestions
  - Test gaps
  - Security/performance summary
  - Merge recommendation

**Temperature:** 0.1

---

### 4. Test Writer Agent

**Role:** Testing strategy and test generation expert

**Responsibilities:**
- ✅ Design comprehensive test strategy
- ✅ Generate test cases (functional, exception, permission, boundary, regression)
- ✅ Create automation script drafts
- ✅ Identify manual test steps
- ✅ Flag test data and credential needs
- ✅ Mark uncertain business assertions

**Principles:**
- Cover happy path AND error scenarios
- Test permissions thoroughly
- Test boundary conditions (empty, null, max, etc.)
- Ensure regression tests for existing features
- All test data must be anonymized

**Stop Conditions:**
- Cannot define acceptance criteria
- Test automation impossible without live data
- Business assertions too uncertain

**Outputs:**
- test-plan.md
- test-case-writer.md (subagent output)
- automation-scripts.md (drafts)

**Temperature:** 0.1

---

## Subagents

### 1. PRD Analyzer

**Role:** Requirement clarity specialist

**Responsibility:**
- Transform fuzzy requirements into engineering task card
- Clarify business goals, user roles, user stories
- Identify all stakeholders and their needs
- Break down into pages, APIs, data models, permissions
- List exception scenarios
- Identify risks and unknowns
- Flag items requiring human confirmation

**Principles:**
- Never assume fields, APIs, or permission rules
- Never invent data structures
- Always ask for clarification on ambiguous requirements
- Mark all uncertain items

**Outputs:**
- task-card.md

**Permissions:** Read-only (no code edits, no bash)

**Temperature:** 0.1

---

### 2. Code Reviewer

**Role:** Code quality and maintainability expert

**Responsibility:**
- Assess code quality against standards
- Check for bugs, edge cases, null safety
- Verify API compatibility
- Identify code duplication
- Assess performance issues at code level
- Review exception handling
- Check test coverage
- Verify compliance with project conventions

**Checklist:**
- ✅ Requirement compliance
- ✅ Behavior unchanged (external)
- ✅ API compatibility
- ✅ Exception handling
- ✅ Null/boundary safety
- ✅ Code duplication
- ✅ Design patterns
- ✅ Logging/error codes
- ✅ Test coverage
- ✅ Performance issues
- ✅ Readability
- ✅ Project conventions

**Severity Levels:**
- **Critical:** Blocks merge (major bug, security issue, API break)
- **Major:** Should fix before merge (missing exception handling, test gap)
- **Minor:** Nice to have (code style, refactor suggestion)

**Outputs:**
- code-review.md

**Permissions:** Read-only (no code edits, no bash)

**Temperature:** 0.1

---

### 3. Security Reviewer

**Role:** Security vulnerability auditor

**Responsibility:**
- Audit code for security vulnerabilities
- Check authorization and authentication
- Identify injection risks (SQL, command, XSS, SSRF)
- Verify secrets are not hardcoded
- Check for sensitive data in logs
- Assess file upload risks
- Review dependency security
- Verify audit logging

**OWASP Coverage:**
- A01: Broken Access Control
- A02: Cryptographic Failures
- A03: Injection
- A04: Insecure Design
- A05: Security Misconfiguration
- A06: Vulnerable & Outdated Components
- A07: Identification & Authentication Failures
- A08: Data Integrity Failures
- A09: Logging & Monitoring Failures
- A10: SSRF

**Severity Levels:**
- **Critical (CVSS 9-10):** Exploitable vulnerability, blocks merge
- **High (CVSS 7-8):** Serious issue, should fix before merge
- **Medium (CVSS 4-6):** Should address
- **Low (CVSS 0-3):** Minor hardening

**Outputs:**
- security-review.md

**Permissions:** Read-only (no code edits, no bash)

**Temperature:** 0.1

---

### 4. Test Case Writer

**Role:** Test design specialist

**Responsibility:**
- Generate detailed test cases from requirements and code changes
- Cover functional, exception, permission, boundary, regression scenarios
- Create automation script drafts
- Identify manual test steps
- Flag test data and credential needs
- Mark uncertain business assertions

**Test Types:**
- Functional: Happy path
- Exception: Error scenarios
- Permission: Role-based access
- Boundary: Edge cases (empty, null, max)
- Regression: Existing features still work
- Security: Auth, injection, XSS
- Compatibility: Version compatibility

**Outputs:**
- test-plan.md
- test-case-table.md
- automation-scripts.md (drafts)
- regression-checklist.md

**Permissions:** Read-only (no code edits, no bash)

**Temperature:** 0.1

---

### 5. Performance Checker

**Role:** Performance and stability auditor

**Responsibility:**
- Detect performance regressions
- Check for N+1 queries, unindexed queries
- Assess caching strategy (penetration, breakdown)
- Identify blocking calls and main-thread delays
- Review resource lifecycle (connections, files, threads)
- Detect memory leaks, lock contention, race conditions
- Verify timeout/retry/fallback completeness
- Assess monitoring coverage

**Risk Levels:**
- **High:** N+1 queries, memory leaks, deadlocks, main-thread blocking
- **Medium:** Cache issues, minor performance regression, missing metrics
- **Low:** Code style, future optimization

**Outputs:**
- performance-review.md
- load-test-scenarios.md
- monitoring-checklist.md

**Permissions:** Read-only (no code edits, no bash)

**Temperature:** 0.1

---

### 6. Log Diagnoser

**Role:** Log analysis and root-cause diagnosis expert

**Responsibility:**
- Diagnose crashes, errors, panics from logs
- Extract key log patterns
- Suggest candidate root causes with evidence
- Differentiate facts from hypotheses
- Recommend next investigation steps
- Suggest fixes
- Create regression test plan

**Log Domains:**
- App crashes and stack traces
- Android logcat
- Linux dmesg, kernel logs
- Backend service errors
- Database errors
- Network timeouts

**Principles:**
- Evidence-based reasoning only
- Mark unproven hypotheses as "assumption"
- Focus on diagnostic next steps
- Never directly modify ROM/kernel/driver (human decision)

**Outputs:**
- diagnosis.md
- investigation-plan.md
- fix-recommendations.md

**Permissions:** Read-only (no code edits, no bash)

**Temperature:** 0.1

---

## Temperature Setting

**All agents use temperature: 0.1**

**Why:**
- ✅ Low randomness, high reproducibility
- ✅ Minimal hallucination in code and reviews
- ✅ Deterministic output for engineering tasks
- ✅ NOT suitable for creative writing, brainstorming

---

## Collaboration Pattern

```
guardian-runner (orchestrator)
├── calls @prd-analyzer → task-card.md
├── calls @test-case-writer → test-plan.md
├── calls @code-reviewer → code-review.md
├── calls @performance-checker → performance-review.md
├── calls @security-reviewer → security-review.md
└── synthesizes all → morning-report.md
```

---

## Permission Model

| Agent | Type | Read Code | Edit Code | Run Bash |
|-------|------|-----------|-----------|----------|
| dream-runner | Primary | ✅ Yes | ✅ Yes (with ask) | ✅ Yes (with ask) |
| refactor | Primary | ✅ Yes | ✅ Yes (with ask) | ✅ Yes (with ask) |
| review | Primary | ✅ Yes | ❌ No | ✅ Yes (with ask) |
| test-writer | Primary | ✅ Yes | ✅ Yes (with ask) | ✅ Yes (with ask) |
| prd-analyzer | Subagent | ✅ Yes | ❌ No | ❌ No |
| code-reviewer | Subagent | ✅ Yes | ❌ No | ❌ No |
| security-reviewer | Subagent | ✅ Yes | ❌ No | ❌ No |
| test-case-writer | Subagent | ✅ Yes | ❌ No | ❌ No |
| performance-checker | Subagent | ✅ Yes | ❌ No | ❌ No |
| log-diagnoser | Subagent | ✅ Yes | ❌ No | ❌ No |

---

## Common Handoffs

### dream-runner → @prd-analyzer
**Input:** Task description, acceptance criteria  
**Output:** task-card.md  
**Gate:** No Critical unknowns

### dream-runner → @test-case-writer
**Input:** task-card.md, code diffs  
**Output:** test-plan.md  
**Gate:** Design finalized

### dream-runner → @code-reviewer
**Input:** Code diffs, change-summary.md  
**Output:** code-review.md  
**Gate:** Tests passing

### dream-runner → @performance-checker
**Input:** Code diffs, change-summary.md  
**Output:** performance-review.md  
**Gate:** Tests passing

### dream-runner → @security-reviewer
**Input:** Code diffs, change-summary.md  
**Output:** security-review.md  
**Gate:** Tests passing
