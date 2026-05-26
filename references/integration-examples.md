# Integration Examples — Platform Adaptation

How to integrate Guardian skill into different platforms.

## OpenCode Integration (Native)

**Status:** ✅ Fully supported (original platform)

### Setup

```bash
# Option 1: Install from skill
mkdir -p your-project/.opencode/agents
cp guardian-skill/agents/*.md your-project/.opencode/agents/

# Option 2: Use automated script
./install-guardian.sh your-project

# Option 3: Manual symlink
ln -s $(pwd)/guardian-skill/agents/*.md your-project/.opencode/agents/
```

### Configuration

Create `your-project/opencode.jsonc`:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "default_agent": "guardian-runner",
  "model": "[your-chosen-model]",  // e.g., "gpt-4-turbo"
  "permission": {
    "edit": "ask",
    "bash": "ask"
  },
  "agents_dir": ".opencode/agents"
}
```

### Usage

```bash
cd your-project
opencode

# Select guardian-runner from agent menu
# Paste your task requirements
# Dream runner handles the rest
```

### Agent Calling Syntax

```markdown
# In guardian-runner.md

你需要调用 Subagent：

@prd-analyzer
@test-case-writer
@code-reviewer
@security-reviewer
@performance-checker
```

---

## Claude Code Integration

**Status:** 🟡 Requires wrapper

### Setup

```bash
# Copy agents
mkdir -p your-project/.claude/agents
cp dream-agents-skill/agents/*.md your-project/.claude/agents/

# Create wrapper (see below)
cp dream-agents-skill/examples/claude-code-wrapper.md your-project/.claude/agents/
```

### Configuration

**Option A: Via CLAUDE.md**

```markdown
# CLAUDE.md

/guardian
  spawn: guardian-runner-wrapper
  Initiate overnight Guardian-mode workflow on isolated branch.

/guardian-review
  spawn: review-agent-wrapper
  Organize comprehensive PR review via specialists.
```

**Option B: Via settings.json hooks**

```json
{
  "hooks": {
    "on-spawn-guardian-runner": {
      "spawn": "guardian-runner-wrapper"
    }
  }
}
```

### Wrapper Pattern

Since Claude Code doesn't have native @agent-name syntax, create a wrapper that orchestrates agents:

**claude-code-wrapper.md:**

```markdown
# Guardian Runner Wrapper (Claude Code)

You are the guardian-runner orchestrator for Claude Code.

## Your Role
- Coordinate with subagent prompts via SendMessage
- Manage workflow stages
- Aggregate results
- Generate morning report

## Stage 1: Call PRD Analyzer
Use the Agent tool to spawn a new agent with prd-analyzer prompt:

[Current stage: call Agent(description="...", prompt="...from agents/prd-analyzer.md")]

## How to Call Subagents

Instead of @prd-analyzer, use the Agent tool:

\`\`\`
Agent({
  description: "Requirements analysis",
  prompt: "[Full prd-analyzer.md prompt]"
})
\`\`\`

## Workflow
[Same as guardian-runner.md, but adapt agent calls to Agent tool]
```

### Agent Calling Syntax (Claude Code)

```python
from anthropic import Anthropic

# Instead of @agent-name, spawn agents explicitly
agent_tools = {
    'prd-analyzer': '...',
    'test-case-writer': '...',
    'code-reviewer': '...',
    # etc
}

# Call like:
response = client.messages.create(
    model="claude-opus-4-1",
    system=agent_tools['prd-analyzer'],  # Use agent prompt
    messages=[...]
)
```

---

## Hermes Integration

**Status:** 🟡 Requires YAML config

### Setup

```bash
# Copy agents
mkdir -p your-project/.hermes/agents
cp dream-agents-skill/agents/*.md your-project/.hermes/agents/

# Create config
cp dream-agents-skill/examples/hermes-config.yaml your-project/
```

### Configuration

**hermes.yaml:**

```yaml
agents:
  guardian-runner:
    role: primary-orchestrator
    model: gpt-4-turbo
    temperature: 0.1
    permissions:
      edit: ask
      bash: ask
    subagents:
      - prd-analyzer
      - test-case-writer
      - code-reviewer
      - security-reviewer
      - performance-checker
    workflow:
      - stage: 0
        action: environment_check
      - stage: 1
        action: spawn
        agent: prd-analyzer
      - stage: 2
        action: generate_plan
      - stage: 3
        action: code_changes
      - stage: 4
        action: spawn
        agent: test-case-writer
      - stage: 5
        action: run_tests
      - stage: 6
        action: spawn
        agent: code-reviewer
      - stage: 7
        action: spawn
        agent: performance-checker
      - stage: 8
        action: spawn
        agent: security-reviewer
      - stage: 9
        action: generate_morning_report

  prd-analyzer:
    role: subagent
    model: gpt-4-turbo
    temperature: 0.1
    permissions:
      edit: deny
      bash: deny

  code-reviewer:
    role: subagent
    model: gpt-4-turbo
    temperature: 0.1
    permissions:
      edit: deny
      bash: deny

  # ... other subagents
```

### Agent Calling Syntax (Hermes)

```yaml
guardian-runner:
  calls:
    - spawn: prd-analyzer
      input: task_description
      await: true
      output_to: task-card.md
    
    - spawn: test-case-writer
      input: code_diffs
      await: true
      output_to: test-plan.md
```

---

## Generic Platform Integration

If your platform isn't listed above, follow this pattern:

### Step 1: Copy Prompts

```bash
mkdir -p your-platform/.agents
cp dream-agents-skill/agents/*.md your-platform/.agents/
```

### Step 2: Create Orchestrator Wrapper

Adapt guardian-runner.md to your platform's agent-calling mechanism.

**Example patterns:**

```python
# Pattern A: Function calls
agents.call('prd-analyzer', input=task_description)

# Pattern B: Queue/event system
queue.publish('prd-analyzer-request', task_description)
results = queue.subscribe('prd-analyzer-response')

# Pattern C: HTTP API
response = requests.post(
    '/agents/prd-analyzer',
    json={"input": task_description}
)

# Pattern D: Process spawn
subprocess.run(['./agent', 'prd-analyzer', task_description])
```

### Step 3: Configure Output Directory

Ensure all `.md` files are written to `.agents/dream/` or equivalent.

### Step 4: Set Temperature

All agents must have temperature: 0.1

### Step 5: Test

Run on a sample task with non-production code.

---

## Comparison Table

| Feature | OpenCode | Claude Code | Hermes | Generic |
|---------|----------|-------------|--------|---------|
| Native support | ✅ Yes | 🟡 Wrapper | 🟡 Config | 🟡 Custom |
| Agent syntax | `@agent-name` | Agent tool | spawn() | Custom |
| Permissions | Built-in | settings.json | YAML | Custom |
| Workflow stages | Built-in | Manual | Built-in | Manual |
| Temperature control | Built-in | Default | YAML | Custom |
| Ease of setup | Easy | Hard | Medium | Hard |

---

## Minimal Viable Setup (Any Platform)

```
1. Copy agents/\*.md
2. Create orchestrator wrapper
3. Set temperature: 0.1 for all agents
4. Route outputs to .guardian/ directory
5. Implement stop conditions (see safety-principles.md)
6. Test on non-production code
```

---

## Troubleshooting

### Agent Not Called
- Check agent prompt file exists
- Verify agent calling syntax matches platform
- Check permissions (subagents need edit: deny, bash: deny)

### Temperature Not Applied
- Some platforms override temperature
- Check platform default settings
- Explicitly set in agent config if possible

### Output Files Not Generated
- Verify .guardian/ directory exists
- Check agent permissions to write files
- Ensure path is absolute, not relative

### Workflow Stops Unexpectedly
- Check stop condition logs
- Verify all 9 output files are requested
- Review morning report for details

---

## Best Practices

1. **Test First:** Always test on a staging branch first
2. **Monitor Logs:** Watch agent output for errors
3. **Version Agents:** Track agent versions separately from platform
4. **Temperature Lock:** Always set 0.1, never vary
5. **Review Before Merge:** Always human-review before merging
6. **Audit Trail:** Keep .guardian/ directory for 7+ days

---

## Example: Minimal OpenCode Setup

```bash
#!/bin/bash
# setup-guardian.sh

SKILL_PATH="guardian-skill"
PROJECT_PATH="$1"

# Create directories
mkdir -p "$PROJECT_PATH/.opencode/agents"
mkdir -p "$PROJECT_PATH/.guardian"

# Copy agents
cp "$SKILL_PATH/agents/"*.md "$PROJECT_PATH/.opencode/agents/"

# Create config
cat > "$PROJECT_PATH/opencode.jsonc" << 'EOF'
{
  "$schema": "https://opencode.ai/config.json",
  "default_agent": "guardian-runner",
  "model": "gpt-4-turbo",
  "permission": {
    "edit": "ask",
    "bash": "ask"
  }
}
EOF

echo "✅ Dream agents skill installed to $PROJECT_PATH"
```

Use:
```bash
./setup-dream-agents.sh ~/my-project
cd ~/my-project
opencode
```

---

## References

- See `safety-principles.md` for stop conditions
- See `workflow-stages.md` for detailed stage breakdown
- See `agent-responsibilities.md` for role definitions
