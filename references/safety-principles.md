# Guardian Safety Principles — Constraints & Stop Conditions

## Top 10 Immutable Rules

### 1. Branch Isolation
**Rule:** Never work on `main`, `master`, or any production branches.

**Implementation:**
- Always create feature branch: `ai/guardian-{task-name}`
- Verify before each session: `git status`, current branch
- Refuse to proceed if on main/master

**Rationale:** Prevents accidental overwrites of stable code.

---

### 2. No Auto-Commit/Push/Merge
**Rule:** Agent completes work; human engineer approves and merges.

**Implementation:**
- Never call `git commit`
- Never call `git push`
- Never call `git merge` or `git rebase`
- Only prepare code changes; human runs merge

**Rationale:** Only humans own deployment and release decisions.

---

### 3. No Modification of Secrets/Config
**Rule:** Never touch `.env`, API keys, certificates, tokens, production config.

**Implementation:**
- Grep for `.env` files and refuse modification
- Block edits to:
  - `*.pem`, `*.key`, `*.crt` (certificates)
  - `secrets.json`, `config.prod.js`, etc.
  - Environment variable files
- Stage changes, do not deploy

**Rationale:** Prevents credential leaks and production incidents.

---

### 4. No Production Environment Commands
**Rule:** No live database writes, deployments, or ops commands.

**Forbidden:**
```
kubectl apply / delete
terraform apply / destroy
docker system prune
aws s3 rm -r
psql / mysql production database writes
systemctl restart
git push --force
```

**Allowed:**
```
npm test
pytest
go test ./...
grep, git log
cat, ls (read-only)
```

**Rationale:** Prevents live incidents.

---

### 5. No New Dependencies Without Approval
**Rule:** Never add packages, SDKs, or libraries.

**Implementation:**
- Refuse any `npm install`, `pip install`, `go get <new>`, `cargo add`
- If plan requires new dependency, stop and flag for human approval

**Rationale:** Dependencies introduce supply-chain risk.

---

### 6. Never Bypass Tests, Reviews, or Security Checks
**Rule:** Complete all workflow stages; no shortcuts.

**Implementation:**
- Run all required tests (Stage 5)
- Conduct all code reviews (Stage 6)
- Run security scan (Stage 8)
- If any stage fails, continue debugging or stop

**Rationale:** Testing and reviews catch bugs and security issues.

---

### 7. Stop on Critical/High Security Issues
**Rule:** Any security finding with CVSS 7+ or "Critical" designation halts progress.

**Examples:**
- SQL injection vulnerability
- Hardcoded API key
- Unrestricted authorization
- XSS in user input handling

**Implementation:**
- Security reviewer outputs severity level
- If Critical/High found, stop and report
- Do not proceed to merge recommendation

**Rationale:** Security issues block production.

---

### 8. Stop on Core Logic Changes
**Rule:** Refuse changes to permissions, billing, device control, ROM/kernel.

**Examples of STOP:**
- Adding permission bypass
- Modifying billing calculation
- Changing device driver calls
- ROM core firmware changes

**Examples of OK:**
- Fixing a UI bug
- Adding a logging statement
- Refactoring helper function

**Implementation:**
- Review plan (Stage 2) for core logic involvement
- If detected, stop and flag for human

**Rationale:** Core logic changes require deep domain expertise.

---

### 9. Limit Auto-Fix to 2 Rounds
**Rule:** If tests fail, auto-fix max 2 times, then stop.

**Implementation:**
- Fix 1: First attempt at root cause
- Fix 2: Second attempt at different root cause
- If still failing: Stop, report issue, wait for human

**Rationale:** Prevents infinite loops and hidden failures.

---

### 10. Mark All Uncertainties
**Rule:** Never assume; always flag uncertain conclusions.

**Examples:**
- "Assuming field X is optional — **needs human confirmation**"
- "Performance impact unknown without load test — **needs human confirmation**"
- "Regex coverage uncertain — **needs human confirmation**"

**Implementation:**
- Every agent flags uncertain items in output
- Morning report collects all items requiring human confirmation
- Engineer reviews these first

**Rationale:** Prevents bugs from uncertain assumptions.

---

## Stop Conditions by Category

### Environment Protection (Stage 0)
| Condition | Action |
|-----------|--------|
| On main/master | STOP, refuse to start |
| Working dir dirty | STOP, refuse to start |
| Cannot create branch | STOP, refuse to continue |

### Requirements (Stage 1)
| Condition | Action |
|-----------|--------|
| Critical unknowns | STOP, ask clarification |
| Scope ambiguity | STOP, request refinement |

### Planning (Stage 2)
| Condition | Action |
|-----------|--------|
| Requires new dependency | STOP, flag for approval |
| Database migration needed | STOP, flag for approval |
| Core permissions/billing/ROM | STOP, flag for approval |
| Architectural change | STOP, request design review |

### Code (Stage 3)
| Condition | Action |
|-----------|--------|
| Touches .env / secrets | STOP, revert |
| Large rewrite detected | STOP, revert to Stage 2 |
| Deletes test | STOP, revert |

### Testing (Stage 5)
| Condition | Action |
|-----------|--------|
| Test failure (1st round) | Auto-fix attempt 1 |
| Test failure (2nd round) | Auto-fix attempt 2 |
| Test failure (3rd+ rounds) | STOP, report failures |

### Code Review (Stage 6)
| Condition | Action |
|-----------|--------|
| Critical issue found | STOP, flag issue |
| Major issue found | REPORT in morning report |

### Performance (Stage 7)
| Condition | Action |
|-----------|--------|
| High-risk regression | STOP, flag issue |
| Medium risk | REPORT in morning report |

### Security (Stage 8)
| Condition | Action |
|-----------|--------|
| Critical issue (CVSS 9-10) | STOP immediately |
| High issue (CVSS 7-8) | STOP immediately |
| Medium/Low | REPORT in morning report |

---

## Permission Model

### Always Allowed
- Read files, code, logs
- Run tests (npm test, pytest, go test)
- Create branches
- Commit to feature branch
- Write to `.dream/` output directory

### Always Denied
- Modify `.env`, secrets, tokens
- Push to remote
- Merge or rebase
- Modify production config
- Execute production commands
- Create/delete/modify credentials

### Ask First (Permission Gate)
- Edit source code files
- Create new files
- Execute custom bash commands
- Run load tests / stress tests

---

## Escalation Rules

### Immediate Human Escalation (STOP)
- Security Critical/High
- Core logic changes
- New dependencies
- Database migrations
- Environment config changes

### Report in Morning (Continue)
- Code Review Major
- Performance Medium risk
- Uncertain assumptions
- Design suggestions
- Refactor opportunities

### Automatic Handling (Continue)
- Code formatting
- Test generation
- Documentation updates
- Small bug fixes

---

## Compliance Checklist

Before recommending "ready for merge":

- ✅ Environment: isolated branch, clean state
- ✅ Requirements: task-card.md clear, no Critical unknowns
- ✅ Plan: implementable, no core logic issues
- ✅ Code: changes small-step, no .env/secrets touched
- ✅ Tests: all pass (Stage 5)
- ✅ Code Review: no Critical issues (Stage 6)
- ✅ Performance: no High risk (Stage 7)
- ✅ Security: no Critical/High issues (Stage 8)
- ✅ Morning Report: generated, uncertainties flagged
- ✅ All 9 output files exist in `.dream/`

If any ✅ is ❌, recommendation is:
```
Not recommended for merge. Issues:
- [Issue 1]
- [Issue 2]
```

Or if stopped early:
```
Stopped at Stage X. Reason:
[Specific reason]
```
