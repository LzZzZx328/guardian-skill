# Guardian Risk Matrix — Classification & Handling

## Severity Levels

### Critical 🔴

**Definition:** Must be fixed before merge; blocks production.

**Examples:**
- SQL injection vulnerability
- Hardcoded API key or password
- Unrestricted authorization (any user can access any resource)
- XSS vulnerability in user input
- Authentication bypass
- Data breach risk (PII leaked in logs)
- Production config modified

**Action:**
- 🛑 **STOP immediately**
- Flag in morning report
- Do not recommend merge
- Await human decision

**CVSS Score:** 9.0-10.0 (Critical)

---

### High 🟠

**Definition:** Should be fixed before merge; significant security/stability risk.

**Examples:**
- Missing input validation
- Weak password hashing
- Missing authentication on sensitive endpoint
- Unchecked privilege escalation
- Race condition in critical path
- N+1 query in hot path (10x+ slower)
- Memory leak in long-running process
- Missing exception handling in core logic

**Action:**
- 🛑 **STOP immediately** (for security issues)
- ⚠️ **Report in morning report** (for performance/code quality)
- Request human review
- Do not recommend merge

**CVSS Score:** 7.0-8.9 (High)

---

### Medium 🟡

**Definition:** Should be addressed, but not a blocker for merge.

**Examples:**
- Weak cryptography (e.g., MD5 for passwords)
- Missing audit logging
- Incomplete error handling
- Code duplication in non-critical path
- Slow API response (1-2 seconds)
- Missing test for edge case
- Deprecated API usage

**Action:**
- ℹ️ **Report in morning report**
- Include in fix recommendations
- Can merge if no Critical/High issues
- Schedule follow-up PR to address

**CVSS Score:** 4.0-6.9 (Medium)

---

### Low 🟢

**Definition:** Nice-to-have improvements; does not block merge.

**Examples:**
- Code style inconsistency
- Unused variable
- Over-commented code
- Refactor suggestion
- Minor performance optimization
- Documentation improvement

**Action:**
- 💡 **Suggest in morning report**
- Optional to fix
- Can merge immediately

**CVSS Score:** 0.0-3.9 (Low)

---

## By Category

### Security Issues

| Issue | Severity | Action |
|-------|----------|--------|
| SQL injection | Critical | STOP |
| Command injection | Critical | STOP |
| XSS vulnerability | Critical | STOP |
| Hardcoded credentials | Critical | STOP |
| Auth bypass | Critical | STOP |
| Privilege escalation | High | STOP |
| Weak password hashing | High | STOP |
| Missing input validation | High | STOP |
| Insecure deserialization | High | STOP |
| Missing audit logging | Medium | REPORT |
| Weak encryption | Medium | REPORT |
| Sensitive data in logs | Critical | STOP |

### Performance Issues

| Issue | Severity | Action |
|-------|----------|--------|
| N+1 query (10x+ slower) | High | STOP |
| Memory leak in critical path | High | STOP |
| Main-thread blocking (>100ms) | High | STOP |
| Deadlock risk | High | STOP |
| Cache miss strategy (penetration) | Medium | REPORT |
| Slow query (1-2 sec) | Medium | REPORT |
| Resource leak (not critical) | Medium | REPORT |
| Code optimization opportunity | Low | SUGGEST |

### Code Quality Issues

| Issue | Severity | Action |
|-------|----------|--------|
| API breaking change | Critical | STOP |
| Missing exception handling (core) | High | STOP |
| Null pointer risk | High | STOP |
| Test failure | High | STOP |
| Code duplication (core) | Medium | REPORT |
| Missing test | Medium | REPORT |
| Code style | Low | SUGGEST |

### Environment/Config Issues

| Issue | Severity | Action |
|-------|----------|--------|
| .env file modified | Critical | STOP |
| Secrets touched | Critical | STOP |
| Production config changed | Critical | STOP |
| New dependency added | High | STOP |
| Database migration required | High | STOP |

---

## Stop Condition Workflow

```
Stage 0: Environment Check
  ❌ On main/master? → STOP
  ❌ Dirty working dir? → STOP
  ✅ Clean feature branch? → Continue

Stage 1: Requirements
  ❌ Critical unknown? → STOP
  ✅ Clear scope? → Continue

Stage 2: Plan
  ❌ New dependency? → STOP
  ❌ Core logic change? → STOP
  ❌ DB migration? → STOP
  ✅ Implementable? → Continue

Stage 3: Code
  ❌ Touches .env/secrets? → STOP
  ❌ Large rewrite? → STOP
  ❌ Deletes test? → STOP
  ✅ Small-step changes? → Continue

Stage 5: Tests
  ❌ Failure after 2 rounds? → STOP
  ✅ All pass? → Continue

Stage 6: Code Review
  ❌ Critical issue? → STOP
  ✅ No Critical? → Continue

Stage 7: Performance
  ❌ High-risk issue? → STOP
  ✅ No High-risk? → Continue

Stage 8: Security
  ❌ Critical/High issue? → STOP
  ✅ No Critical/High? → Continue

Stage 9: Morning Report
  ✅ Synthesize all → Output recommendation
```

---

## Decision Matrix

### Should We Recommend Merge?

```
Test Results    Code Review    Security    Performance    Recommendation
PASS            ✅ OK          ✅ OK       ✅ OK          ✅ MERGE
PASS            ✅ OK          ✅ OK       ⚠️ Medium      ✅ MERGE
PASS            ✅ OK          ⚠️ Medium   ✅ OK          ✅ MERGE
PASS            ✅ OK          ⚠️ Medium   ⚠️ Medium      ✅ MERGE
PASS            ⚠️ Major       ✅ OK       ✅ OK          ⚠️ FIX ISSUES
PASS            ⚠️ Major       ✅ OK       ⚠️ Medium      ⚠️ FIX ISSUES
PASS            ✅ OK          ⚠️ Medium   ⚠️ Medium      ✅ MERGE
---
PASS            ❌ Critical    *           *              ❌ DO NOT MERGE
PASS            *              ❌ Critical *              ❌ DO NOT MERGE
PASS            *              ❌ High     *              ❌ DO NOT MERGE
PASS            *              *           ❌ High        ❌ DO NOT MERGE
FAIL            *              *           *              ❌ DO NOT MERGE
STOPPED         *              *           *              ❌ STOPPED AT [STAGE]
```

---

## Escalation Path

### Automatic Escalation (No Human Needed)
- ✅ Merge recommendation if all gates pass
- ✅ Merge blockers if Critical/High issues found

### Human Confirmation Needed
- ⚠️ Merge-safe but uncertain (mark for engineer review)
- ⚠️ Design decisions (architectural choices)
- ⚠️ Risk acceptance (proceed despite Medium-risk issues)
- ⚠️ Scope clarification (requirement ambiguity)

---

## Metrics & Thresholds

### Code Coverage
| Level | Action |
|-------|--------|
| < 50% | Critical flag |
| 50-70% | High flag |
| 70-80% | Medium flag |
| > 80% | OK |

### Performance Regression
| Regression | Action |
|-----------|--------|
| > 50% | Critical flag |
| 20-50% | High flag |
| 10-20% | Medium flag |
| < 10% | OK |

### Test Pass Rate
| Rate | Action |
|------|--------|
| 100% | OK |
| 90-99% | High flag |
| < 90% | Critical flag |

---

## Risk Acceptance Process

**Only humans can accept risk.** AI can only:
1. Identify risk
2. Flag severity
3. Provide mitigation options
4. Request human decision

**Example:**
```
Security Review: Medium Risk
- Issue: Weak password hashing (MD5)
- Recommendation: Use bcrypt instead
- Risk Acceptance: Developer may choose to proceed with fix in next PR

Morning Report: 
- "Acceptable to merge. Follow-up: Switch to bcrypt (Low priority)"
```

---

## Common False Positives to Avoid

### Not Critical
- ❌ Missing comment or documentation
- ❌ Verbose variable name
- ❌ Code could be more DRY
- ❌ Future-proofing suggestion

### Genuinely Critical
- ✅ SQL injection
- ✅ Auth bypass
- ✅ Data loss risk
- ✅ Production config modified

---

## Severity Inflation Risk

**Avoid over-reporting:** Reserve Critical/High for actual blockers.

| Classification | ✅ Correct | ❌ Inflation |
|---------------|----------|------------|
| Critical | Exploitable vulnerability, data loss risk, auth bypass | Code style, naming suggestion |
| High | Missing validation, race condition, resource leak | Could be refactored better |
| Medium | Code duplication, slow query | Missing comment |
| Low | Code style, optimization idea | Theoretical future issue |

---

## Integration with Morning Report

All findings feed into the final recommendation:

```markdown
# Morning Report

## Issues Summary
- Critical: 0
- High: 1 (N+1 query in getUserById)
- Medium: 2
- Low: 3

## Recommendation
Not recommended for merge due to High-risk performance issue.

**Next Steps:**
1. Fix N+1 query in getUserById
2. Re-run tests
3. Re-submit for review
```
