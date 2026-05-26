# Guardian Skill — Overview

This skill enables **unattended coding workflows** — when a developer leaves, the AI orchestrates implementation, testing, reviews, and verification on an isolated branch, leaving only human approval at the end.

## Core Purpose

Transform a software task from fuzzy requirements to human-reviewed, tested, and audited code by:
- Analyzing requirements into engineering task cards
- Generating technical implementation plans  
- Executing safe, small-step code changes
- Generating and running tests
- Conducting code, security, and performance reviews
- Producing a comprehensive verification report

## Key Operating Principles

**Not automation, but verification:** The system does not auto-commit, auto-push, or auto-merge. A human engineer reviews and approves at the end.

**Staged safety gates:** Each workflow stage includes environment checks, risk detection, and explicit stop points for critical/high-severity findings.

**Isolation and rollback:** All work happens in isolated branches (`ai/guardian-{task-name}`) or worktrees; no production configs, secrets, or deployments are touched.

**Multi-agent coordination:** A primary orchestrator (guardian-runner) calls specialized subagents—PRD analyzer, test writer, code reviewer, security reviewer, performance checker—and synthesizes results into a verification report.

**Temperature: 0.1 always:** All agents run with low randomness and high stability, optimized for engineering tasks, not creativity.

## Output Template

```
Guardian Verification Report
├── Task Card
├── Implementation Plan
├── Code Changes Summary
├── Test Plan & Results
├── Code Review
├── Security Review
├── Performance Review
└── Risk Flags & Next Steps
```

## Workflow Stages (10 Steps)

```
Stage 0: Environment Protection (branch, git status, clean checks)
  ↓
Stage 1: Requirements Analysis → task-card.md
  ↓
Stage 2: Technical Plan → plan.md
  ↓
Stage 3: Code Changes → change-summary.md
  ↓
Stage 4: Test Generation → test-plan.md
  ↓
Stage 5: Test Execution → test-result.md
  ↓
Stage 6: Code Review → code-review.md
  ↓
Stage 7: Performance Review → performance-review.md
  ↓
Stage 8: Security Review → security-review.md
  ↓
Stage 9: Verification Report → morning-report.md
  ↓
Human Approval
```

## When to Use This Skill

- Overnight task queues with clear, bounded requirements
- Nightly refactoring or tech-debt cleanup
- Automated PR reviews before human examination
- Continuous testing and quality gates on feature branches
- Teams needing strict safety gates + multi-perspective reviews

## Key Guarantees

✅ **Never auto-commit, auto-push, or auto-merge**  
✅ **Never touch `.env`, secrets, production configs, or deployments**  
✅ **All risky actions explicitly stopped and flagged**  
✅ **All uncertain conclusions marked "needs human confirmation"**  
✅ **All work isolated to feature branches / worktrees**  
✅ **All outputs go to `.guardian/` directory for human review**

---

**Ready to configure?** See SKILL agents in `agents/` and workflow references in `references/`.
