# Guardian Workflow Stages — Detailed Breakdown

10-stage pipeline with checkpoints, outputs, and stop conditions for Guardian automation.

## Stage 0: Environment Protection

**Purpose:** Ensure safe isolation and prevent accidental production changes.

**Actions:**
1. Verify current Git branch (not `main` or `master`)
2. Check `git status` (clean working directory)
3. Confirm no uncommitted changes
4. Create feature branch: `ai/dream-{task-name}`

**Stop Conditions:**
- On main/master branch
- Working directory not clean
- Unable to create feature branch

**Output:** None (environment ready message)

---

## Stage 1: Requirements Analysis

**Purpose:** Convert fuzzy requirements into an engineering-ready task card.

**Subagent Call:** `@prd-analyzer`

**Inputs:**
- User task description
- Acceptance criteria (if provided)
- Existing code context

**Outputs:**
- `.guardian/task-card.md`
  - Requirement understanding
  - Business goals
  - User roles & stories
  - Detailed breakdown (pages, APIs, data, permissions)
  - Acceptance criteria
  - Exception scenarios
  - Risk inventory
  - Open questions
  - Validation checklist

**Stop Conditions:**
- Critical unknowns that block implementation
- Unresolved scope (must clarify before proceeding)
- Indicates need for approval beyond agent scope

---

## Stage 2: Technical Plan

**Purpose:** Design implementation approach, identify files to change, outline rollback.

**Inputs:**
- task-card.md from Stage 1
- Codebase structure

**Outputs:**
- `.guardian/plan.md`
  - File change list
  - Implementation steps
  - Test plan overview
  - Risk points
  - Rollback procedure
  - Out-of-scope items

**Stop Conditions:**
- Requires new dependencies
- Requires database migrations
- Core logic changes (permissions, billing, device control)
- ROM/driver kernel changes
- Significant architectural changes without approval

---

## Stage 3: Code Changes

**Purpose:** Execute small-step code modifications with audit trail.

**Principles:**
- One problem type per change
- No large rewrites
- No test deletions
- No `.env`, secrets, certificates, or production config modifications
- All changes committed to feature branch (not pushed to remote)

**Outputs:**
- `.guardian/change-summary.md`
  - Modified files list
  - Change description per file
  - Rollback checkpoint for each change
  - Visible diffs (for human review)

**Stop Conditions:**
- Changes break existing tests
- Changes touch production configs/secrets
- Large refactor detected (revert to Stage 2)

---

## Stage 4: Test Generation

**Purpose:** Design comprehensive test strategy covering happy path, errors, permissions, boundaries.

**Subagent Call:** `@test-case-writer`

**Inputs:**
- task-card.md
- code changes from Stage 3
- existing test patterns

**Outputs:**
- `.guardian/test-plan.md`
  - Test strategy
  - Functional test cases
  - Exception test cases
  - Permission test cases
  - Boundary test cases
  - Regression test list
  - Automation script drafts
  - Manual verification steps

**Stop Conditions:**
- No testable acceptance criteria
- Test coverage impossible without live data

---

## Stage 5: Test Execution

**Purpose:** Run tests, report results, auto-fix simple failures (max 2 rounds).

**Allowed Commands:**
```
npm test
pnpm test
yarn test
mvn test
pytest
go test ./...
cargo test
python -m unittest
```

**Forbidden Commands:**
```
rm -rf
docker system prune
kubectl apply/delete
terraform apply/destroy
Production database writes
Production deployment commands
```

**Outputs:**
- `.guardian/test-result.md`
  - Test execution log
  - Pass/fail summary
  - Any auto-fixes applied
  - Remaining failures (if any)

**Stop Conditions:**
- Test failures persist after 2 auto-fix rounds
- Cascading failures suggest fundamental issues

---

## Stage 6: Code Quality Review

**Purpose:** Check code quality, maintainability, compatibility, exception handling.

**Subagent Call:** `@code-reviewer`

**Checklist:**
- Requirement compliance
- External behavior unchanged
- Public API compatibility
- Exception handling coverage
- Null/boundary safety
- Code duplication
- Design patterns
- Logging/error codes
- Test coverage
- Performance issues
- Readability
- Project conventions

**Outputs:**
- `.guardian/code-review.md`
  - Change summary
  - Critical issues (block merge)
  - Major issues (require fixes)
  - Minor issues (suggestions)
  - Test gaps
  - Maintainability notes
  - Fix priority order
  - Human confirmation items

**Stop Conditions:**
- Critical issues found
- Major API breaking changes
- Missing test coverage for core logic

---

## Stage 7: Performance Review

**Purpose:** Detect performance regressions, concurrency issues, resource leaks, stability risks.

**Subagent Call:** `@performance-checker`

**Checklist:**
- Time/space complexity
- N+1 queries, unindexed queries
- Cache (penetration, breakdown, stale data)
- Blocking calls, main thread delays
- Resource lifecycle (connections, files, streams, threads)
- Memory leaks
- Lock contention, race conditions
- Timeout/retry/fallback completeness
- Monitoring coverage

**Outputs:**
- `.guardian/performance-review.md`
  - Performance conclusion
  - High-risk issues
  - Medium-risk issues
  - Low-risk issues
  - Load test scenarios
  - Critical metrics
  - Optimization suggestions
  - Human confirmation items

**Stop Conditions:**
- High-risk issues: N+1 queries, memory leaks, deadlocks
- Performance regression > 10% on critical path

---

## Stage 8: Security Review

**Purpose:** Audit authorization, injection, secrets, dependencies, data handling.

**Subagent Call:** `@security-reviewer`

**Checklist:**
- Unauthorized access
- Horizontal privilege escalation
- Vertical privilege escalation
- Multi-tenant isolation
- SQL/command injection
- XSS / SSRF
- Hardcoded secrets
- Sensitive data logging (tokens, phones, customer info)
- File upload risks
- Deserialization risks
- Dependency vulnerabilities
- Audit log coverage
- Error message leakage

**Outputs:**
- `.guardian/security-review.md`
  - Security conclusion
  - Critical issues (CVSS 9-10)
  - High issues (CVSS 7-8)
  - Medium issues (CVSS 4-6)
  - Low issues (CVSS 0-3)
  - Missing security tests
  - Fix recommendations
  - Human confirmation items

**Stop Conditions:**
- Critical or High security issues
- Any injection vulnerability
- Hardcoded secrets or tokens

---

## Stage 9: final Report

**Purpose:** Synthesize all results, recommend human action, flag uncertainties.

**Inputs:**
- All outputs from Stages 1-8
- Git diff summary
- Test results

**Outputs:**
- `.dream/final-report.md`
  - Executive summary
  - Task objective (from task-card.md)
  - Actual work completed
  - Modified files list
  - Test results (pass/fail/coverage)
  - Code review summary (issues found, criticality)
  - Security findings (count by severity)
  - Performance impact (if any)
  - Unresolved issues
  - Blocking items (if any)
  - Recommendation:
    - "Ready for human review" (if all gates pass)
    - "Not recommended for merge: [reason]" (if issues remain)
    - "Stopped at Stage X: [reason]" (if halted early)
  - Items requiring human confirmation
  - Next steps for engineer

**Stop Conditions:** None (always produces report)

---

## Summary: Stop Conditions by Severity

| Severity | Stage | Stop? | Action |
|----------|-------|-------|--------|
| Critical | Security (8) | ✅ YES | Flag & halt |
| High | Security (8) | ✅ YES | Flag & halt |
| Critical | Code Review (6) | ✅ YES | Flag & halt |
| Major | Code Review (6) | ⚠️ REPORT | Include in final report |
| Test Failure | Test (5) | ⚠️ AUTO-FIX | Max 2 rounds, then halt |
| Performance High Risk | Perf (7) | ✅ YES | Flag & halt |

---

## Key Principles

**Isolation:** All work on feature branch `ai/dream-{task-name}`, never main/master.

**Fail-safe:** Every stage produces output; no silent failures.

**Human in loop:** No auto-merge; engineer reviews final report.

**Transparency:** All decisions logged with evidence.

**Rollback-ready:** Each stage has clear undo path.
