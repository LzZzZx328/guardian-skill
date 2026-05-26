# Guardian — 通用无人值守自动化工作流

完整的自动化编码工作流，包含 10 个专业 AI Agent、10 阶段流程管道，以及人工最终审批。

## 目录结构

```
guardian-skill/
├── SKILL.md                               # Skill 定义与设计理念
├── README.md                              # 本文件
├── STRUCTURE.md                           # 完整结构说明
├── agents/                                # 通用 Agent 提示词（平台无关）
│   ├── guardian-runner.md                 # 主控 Agent：工作流总控
│   ├── refactor.md                        # 主控 Agent：安全重构
│   ├── review.md                          # 主控 Agent：PR 审查协调
│   ├── test-writer.md                     # 主控 Agent：测试生成
│   ├── prd-analyzer.md                    # Subagent：需求分析
│   ├── code-reviewer.md                   # Subagent：代码审查
│   ├── security-reviewer.md               # Subagent：安全审查
│   ├── test-case-writer.md                # Subagent：测试用例生成
│   ├── performance-checker.md             # Subagent：性能稳定性审查
│   └── log-diagnoser.md                   # Subagent：日志诊断
│
└── references/                            # 参考文档
    ├── workflow-stages.md                 # 10 阶段流程详解
    ├── safety-principles.md               # 安全约束与停止条件
    ├── output-structure.md                # 输出文件规范
    ├── agent-responsibilities.md          # Agent 角色定义
    ├── risk-matrix.md                     # 风险分级与决策
    └── integration-examples.md            # 平台集成指南
```

## 快速开始

### OpenCode（推荐）

```bash
# 复制 Agent 文件
cp guardian-skill/agents/*.md your-project/.opencode/agents/

# 配置 opencode.jsonc
cat > your-project/opencode.jsonc << 'EOF'
{
  "default_agent": "guardian-runner",
  "model": "[选择模型]",
  "permission": {
    "edit": "ask",
    "bash": "ask"
  }
}
EOF

# 启动 Guardian 工作流
cd your-project
opencode
# 选择 guardian-runner
# 粘贴任务需求
# 工作流自动运行（阶段 0-9）
# 生成验收报告
```

### Claude Code / Hermes（需要适配）

1. 复制 `agents/` 中的提示词到你的平台
2. 创建编排包装层（参考 `references/integration-examples.md`）
3. 配置权限管制和安全检查
4. 设置所有 Agent 的 temperature 为 0.1

### 最小化安装

```
1. 复制 agents/ 目录的提示词
2. 创建主控 Agent (guardian-runner)
3. 设置 subagent 调用机制
4. 输出目录设为 .guardian/
5. 在隔离分支上测试
```

---

## 10 个 Agent — 角色分工

### 主控 Agent（编排与行动）

| Agent | 职责 | 主要输出 |
|-------|------|---------|
| **guardian-runner** | 工作流总控、阶段管理、安全门禁 | morning-report.md |
| **refactor** | 安全小步重构、保持接口稳定 | refactor-plan.md |
| **review** | PR 审查协调、多视角综合 | review-summary.md |
| **test-writer** | 测试策略设计、用例生成执行 | test-result.md |

### 专家 Agent（专业审查）

| Agent | 专业领域 | 输出 |
|-------|---------|------|
| **prd-analyzer** | 需求澄清、任务卡生成 | task-card.md |
| **code-reviewer** | 代码质量、可维护性、Bug 检查 | code-review.md |
| **security-reviewer** | 安全审计、漏洞检测 | security-review.md |
| **test-case-writer** | 测试用例、自动化脚本 | test-plan.md |
| **performance-checker** | 性能、并发、资源泄漏 | performance-review.md |
| **log-diagnoser** | 日志分析、根因诊断 | diagnosis.md |

---

## 10 阶段工作流

### Stage 0：环境保护
- 检查当前分支（不能是 main/master）
- 验证工作区干净
- 创建隔离特性分支

### Stage 1：需求分析
- 调用 `@prd-analyzer`
- 生成 task-card.md（明确范围、目标、风险）
- Critical 问题则停止

### Stage 2：技术方案
- 制定实现方案
- 列出修改文件、步骤、回滚方案
- 需要新增依赖/数据库迁移则停止

### Stage 3：代码修改
- 只做小步修改（一次改一类问题）
- 更新 change-summary.md
- 禁止修改 .env、密钥、生产配置

### Stage 4：测试生成
- 调用 `@test-case-writer`
- 覆盖正常路径、异常、权限、边界、回归
- 生成 test-plan.md

### Stage 5：测试执行
- 运行低风险测试命令（npm test、pytest 等）
- 最多自动修复 2 轮失败
- 更新 test-result.md

### Stage 6：代码质量审查
- 调用 `@code-reviewer`
- Critical 问题则停止
- 输出 code-review.md

### Stage 7：性能审查
- 调用 `@performance-checker`
- 检查 N+1 查询、缓存、并发、泄漏
- High 风险则停止
- 输出 performance-review.md

### Stage 8：安全审查
- 调用 `@security-reviewer`
- Critical/High 问题则停止
- 输出 security-review.md

### Stage 9：验收报告
- 综合所有结果
- 列出未解决问题、阻断项
- 生成 final-report.md
- 给出建议："建议人工审查" 或 "停止，原因：……"

---

## 安全原则

### 10 条铁律

1. ❌ 禁止在 main/master 直接工作 — 必须用特性分支
2. ❌ 禁止自动提交、自动推送、自动合并
3. ❌ 禁止修改 .env、密钥、Token、生产配置
4. ❌ 禁止执行生产环境命令
5. ❌ 禁止新增未批准的依赖
6. ❌ 禁止绕过测试、Code Review、安全审查
7. 🛑 Critical/High 安全问题必须停止
8. 🛑 权限、计费、设备控制、ROM/驱动变更必须停止
9. 🛑 测试失败超过 2 轮自动修复则停止
10. 📌 所有不确定结论标注"需人工确认"

### 输出文件（9 个必须生成）

```
.guardian/
├── task-card.md              # 需求拆解
├── plan.md                   # 技术方案
├── change-summary.md         # 代码修改
├── test-plan.md              # 测试策略
├── test-result.md            # 测试结果
├── code-review.md            # 质量评估
├── performance-review.md     # 性能稳定性评估
├── security-review.md        # 安全审查
└── final-report.md         # 综合报告与建议
```

---

## 平台配置

### OpenCode（原生支持）
- ✅ 通过 `.opencode/agents/` + `opencode.jsonc` 完全支持
- Agent 调用语法：`@agent-name`
- 权限在元数据中配置（mode, edit, bash）
- 开箱即用

### Claude Code（需要包装层）
- 复制提示词到 `.claude/agents/`
- 使用 Agent 工具在 guardian-runner 中调用 subagent
- 通过 CLAUDE.md 或 settings.json 配置
- 🟡 需要编排包装

### Hermes（需要 YAML 配置）
- 复制提示词到平台
- 使用原生工作流编排
- 通过 hermes.yaml 配置
- 🟡 需要平台绑定

---

## 推荐启动方式

```bash
# 进入项目目录
cd your-project

# 启动 OpenCode Guardian 模式
opencode select guardian-runner

# 粘贴任务需求与限制条件：
请进入 Guardian 模式，严格按照 guardian-runner 工作流执行以下任务：

需求：
[描述你的需求]

限制：
1. 只允许在新分支 ai/guardian-{task-name} 上工作；
2. 不允许提交、推送、合并；
3. 不允许新增依赖；
4. 不允许修改 .env、生产配置、密钥；
5. 测试失败最多自动修复 2 轮；
6. 结束后必须生成 .guardian/final-report.md；
7. 如果涉及权限、数据、设备控制风险，必须停止并标注需人工确认。
```

---

## Guardian 验收完成标准

同时满足以下条件，才可建议进入人工 Review：

- ✅ task-card.md 已生成且需求清晰
- ✅ plan.md 已生成且步骤完整
- ✅ change-summary.md 记录所有修改
- ✅ test-plan.md 覆盖正常/异常/权限/边界/回归
- ✅ test-result.md 显示所有测试通过
- ✅ code-review.md 无 Critical 问题
- ✅ performance-review.md 无阻断风险
- ✅ security-review.md 无 Critical/High 问题
- ✅ morning-report.md 已生成且给出明确建议

---

## 参数设置

### 温度参数（Temperature）

所有 Agent 使用 **temperature: 0.1**：
- 低随机性，高可重现性
- 审查中极少幻觉
- 代码生成更稳定
- 专为工程任务优化（不适合创意任务）

如果你的平台需要配置温度，所有 Agent 都设为 0.1。

---

## Skill 不包含

- ❌ 模型凭证（使用你的平台认证）
- ❌ 平台特定配置格式（按集成指南自适配）
- ❌ 生产部署或上线（仅生成候选实现）
- ❌ 自动审批逻辑（必须人工最终决策）

---

## 集成检查清单

- [ ] 复制 agents/ 提示词到你的平台
- [ ] 创建主控 Agent (guardian-runner)
- [ ] 连接 subagent 调用机制（@agent 或 spawn）
- [ ] 所有 Agent 设置 temperature: 0.1
- [ ] 配置输出目录（.guardian/ 或等价）
- [ ] 在隔离分支上用样本任务测试
- [ ] 验证生成 9 个输出文件
- [ ] 审查 morning-report.md 再决定合并
- [ ] 如需要，设置每日/每夜自动运行

---

## 更多帮助

- 详细工作流：参考 `references/workflow-stages.md`
- 安全约束与停止条件：参考 `references/safety-principles.md`
- 输出文件规范：参考 `references/output-structure.md`
- 平台集成方案：参考 `references/integration-examples.md`
- Agent 角色分工：参考 `references/agent-responsibilities.md`
- 风险分级决策：参考 `references/risk-matrix.md`

---

**最后提醒：** Guardian 不是完全自动化，而是 AI 辅助 + 人工最终把关的混合模式。人工 Review 是最后也是最关键的一道防线。
