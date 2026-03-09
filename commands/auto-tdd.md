---
description: 自动化 TDD 开发工作流。接收技术方案，自动完成需求分析、架构设计、测试驱动开发、质量保证和文档更新。最大化并行执行，自动处理失败，确保 80%+ 覆盖率。
---

# 自动化 TDD 命令

此命令提供完整的端到端 TDD 开发自动化，从需求分析到最终交付。

## 此命令的作用

1. **需求分析与架构设计** - 并行执行 architect + planner
2. **测试驱动开发** - 循环执行 tdd-guide (RED → GREEN → REFACTOR)
3. **质量保证** - 并行执行 code-reviewer + security-reviewer + e2e-runner
4. **优化与文档** - 并行执行 refactor-cleaner + doc-updater
5. **生成最终报告** - 汇总所有变更和测试结果

## 何时使用

在以下情况下使用 `/auto-tdd`：

* 需要实现完整的功能特性
* 需要重构现有系统
* 需要添加新的模块或组件
* 需要确保代码质量和测试覆盖率
* 需要自动化端到端开发流程

## 使用方法

```bash
# 标准模式 - 完整四阶段工作流
/auto-tdd I need to implement user authentication with JWT

# 快速模式 - 跳过阶段 4（优化与文档）
/auto-tdd --quick Add a search feature with pagination

# 继续模式 - 失败后继续执行
/auto-tdd --continue Refactor the database layer

# 详细模式 - 显示每个智能体的详细输出
/auto-tdd --verbose Implement payment integration
```

## 工作流程

### 阶段 1：需求分析与架构设计（并行）

```
用户输入技术方案
        ↓
    ┌───┴───┐
    ↓       ↓
┌────────┐ ┌────────┐
│architect│ │planner │
│ 5 min  │ │ 7 min  │
└────────┘ └────────┘
    ↓       ↓
    └───┬───┘
        ↓
   [交接文档]
```

**触发智能体**:
- **architect**: 分析架构影响，设计系统方案，创建 ADRs
- **planner**: 分析需求，分解任务，识别依赖和风险

**输出**:
- 架构设计文档
- 详细实施计划
- 风险评估报告

**质量门控**:
- [ ] 架构设计包含至少 1 个 ADR
- [ ] 实施计划包含完整的任务列表
- [ ] 风险评估已覆盖关键风险

### 阶段 2：测试驱动开发（循环）

```
    [交接文档]
         ↓
┌─────────────────────┐
│   tdd-guide 循环    │
│                     │
│  ┌─────┐           │
│  │RED  │ 编写测试   │
│  └──┬──┘           │
│     ↓              │
│  ┌─────┐           │
│  │GREEN│ 实现代码   │
│  └──┬──┘           │
│     ↓              │
│  ┌────────┐       │
│  │REFACTOR│ 优化   │
│  └───┬────┘       │
│      ↓             │
│  ┌──────────┐     │
│  │覆盖率≥80%│     │
│  └──────────┘     │
└─────────────────────┘
         ↓
    [实现完成]
```

**使用 commands**
- **/tdd**: 执行 TDD 开发命令

**触发智能体**:
- **tdd-guide**: 执行 TDD 循环，编写测试和实现

**并行策略**:
- 独立模块并行开发
- 最多同时运行 5 个 tdd-guide 实例

**输出**:
- 测试代码
- 实现代码
- 覆盖率报告

**质量门控**:
- [ ] 测试覆盖率 ≥ 80%
- [ ] 所有测试通过
- [ ] 严格遵循 TDD 循环

### 阶段 3：质量保证（并行）

```
    [实现完成]
         ↓
    ┌────┼────┐
    ↓    ↓    ↓
┌──────┐┌──────┐┌──────┐
│ code ││security││ e2e │
│review││reviewer││runner│
│ 5min ││  3min ││ 8min │
└──────┘└──────┘└──────┘
    ↓    ↓    ↓
    └────┼────┘
         ↓
   [QA报告]
```

**触发智能体**:
- **code-reviewer**: 代码质量审查
- **security-reviewer**: 安全漏洞检测
- **e2e-runner**: 端到端测试

**审查清单**:

**安全性（CRITICAL）**:
- [ ] 硬编码凭据
- [ ] SQL 注入
- [ ] XSS 漏洞
- [ ] 路径遍历

**代码质量（HIGH）**:
- [ ] 超大函数（>50 行）
- [ ] 过深嵌套（>4 层）
- [ ] 缺少错误处理

**质量门控**:
- [ ] 无 CRITICAL 问题
- [ ] HIGH 问题 ≤ 3 个
- [ ] 所有 E2E 测试通过

### 阶段 4：优化与文档（并行）

```
    [QA报告]
        ↓
    ┌───┴───┐
    ↓       ↓
┌────────┐ ┌────────┐
│refactor│ │  doc   │
│cleaner │ │updater │
│  3min  │ │  4min  │
└────────┘ └────────┘
    ↓       ↓
    └───┬───┘
        ↓
   [最终报告]
```

**触发智能体**:
- **refactor-cleaner**: 清理死代码，消除重复项
- **doc-updater**: 生成代码映射图，更新文档

**输出**:
- 重构报告
- 更新的文档
- 代码映射图

**最终验证**:
```bash
npm run build          # 构建检查
npm run test           # 所有测试
npm run test:coverage  # 覆盖率检查
npm run lint           # 代码检查
```

## 使用示例

### 示例 1：实现用户认证系统

```
User: /auto-tdd I need to implement user authentication with JWT and OAuth2

Agent:
# AUTO-TDD WORKFLOW STARTED
Task: Implement user authentication with JWT and OAuth2

## Phase 1: Analysis & Architecture (Parallel)

### Architect Agent
- Analyzing current architecture...
- Identifying affected components...
- Designing authentication system...
- Creating ADRs...

ADR-001: Use JWT for stateless authentication
- Status: Accepted
- Decision: JWT with RS256 signing
- Consequences:
  - ✅ Stateless, scalable
  - ✅ Easy to verify
  - ❌ Token revocation needs Redis

### Planner Agent
- Analyzing requirements...
- Breaking down tasks...
- Identifying dependencies...

Implementation Plan:
- Phase 1: Core authentication (3 tasks)
- Phase 2: OAuth2 integration (2 tasks)
- Phase 3: Security hardening (2 tasks)

Risk Assessment:
- Risk: Token revocation latency
  - Mitigation: Use Redis blacklist
- Risk: OAuth2 provider downtime
  - Mitigation: Fallback to local auth

## Phase 2: TDD Development

### Task 1: JWT Token Generation
1. ✍️ Writing tests... (RED)
2. 🔍 Running tests... Expected failure ✓
3. 💻 Implementing... (GREEN)
4. ✅ Tests passing
5. 🔧 Refactoring...
6. 📊 Coverage: 92%

### Task 2: Token Verification
... [similar process]

### Task 3: OAuth2 Integration
... [similar process]

## Phase 3: Quality Assurance (Parallel)

### Code Review
- Issues: 2 HIGH, 3 MEDIUM
- Status: ✅ Passed (all issues documented)

### Security Review
- Vulnerabilities: 0 CRITICAL, 1 HIGH
- Status: ✅ Passed
- Recommendation: Add rate limiting to /auth endpoints

### E2E Testing
- Tests: 5/5 passed
- Coverage: Login, Logout, OAuth, Token Refresh

## Phase 4: Optimization & Documentation

### Refactor Cleaner
- Removed: 3 unused functions
- Consolidated: 2 duplicate helpers

### Doc Updater
- Updated: README.md
- Generated: docs/CODEMAPS/auth.md

## Final Report

✅ READY TO SHIP

Duration: 32 minutes
Files Changed: 12 new, 5 modified
Test Coverage: 89%
Security: Passed

Next Steps:
1. Review changes
2. Run manual testing
3. Commit and push
```

### 示例 2：快速模式

```
User: /auto-tdd --quick Add pagination to the user list API

Agent:
# AUTO-TDD WORKFLOW (QUICK MODE)
Skipping Phase 4 (Optimization & Documentation)

## Phase 1: Analysis
[Architect + Planner output...]

## Phase 2: TDD Development
[tdd-guide output...]

## Phase 3: Quality Assurance
[Parallel QA output...]

✅ READY TO SHIP (Quick Mode)
Duration: 18 minutes
```

### 示例 3：继续模式

```
User: /auto-tdd --continue Previous task failed at Phase 3

Agent:
# AUTO-TDD WORKFLOW (CONTINUE MODE)
Resuming from Phase 3 checkpoint...

## Phase 3: Quality Assurance (Retry)
[Retrying failed agents...]

Issue Fixed: SQL injection vulnerability removed
Re-running security review...
✅ Security review passed

Continuing to Phase 4...

## Phase 4: Optimization & Documentation
[doc-updater + refactor-cleaner output...]

✅ READY TO SHIP
```

## 参数说明

### $ARGUMENTS

| 参数 | 说明 | 示例 |
|------|------|------|
| `description` | 技术方案描述（必需） | `Implement user authentication` |
| `--quick` | 快速模式，跳过阶段 4 | `/auto-tdd --quick Add feature` |
| `--continue` | 继续模式，从失败点继续 | `/auto-tdd --continue` |
| `--verbose` | 详细模式，显示智能体详细输出 | `/auto-tdd --verbose Refactor X` |

## 失败处理

### 自动修复

- **构建失败**: build-error-resolver 自动介入
- **测试失败**: tdd-guide 自动重试（最多 3 次）
- **覆盖率不足**: 自动补充测试

### 用户介入

以下情况需要用户决策：
- CRITICAL 问题 > 3 个
- 自动修复失败 3 次
- 架构冲突
- 安全审查 CRITICAL 问题

## 配置选项

### 技能加载

根据项目类型自动加载技能：

- **Vue 项目**: `vue-expert`, `vue-testing-best-practices`
- **数据库操作**: `dbhub`
- **Docker 集成**: `docker-patterns`

## 与其他命令的集成

- **先使用 `/plan`**: 对于复杂功能，可以先使用 `/plan` 创建规划
- **使用 `/tdd`**: 阶段 2 内部使用 `/tdd` 命令
- **使用 `/code-review`**: 阶段 3 内部使用 `/code-review` 命令
- **使用 `/build-fix`**: 构建失败时自动调用 `/build-fix`

## 成功指标

- ✅ 测试覆盖率 ≥ 80%
- ✅ 所有测试通过
- ✅ 无 CRITICAL 问题
- ✅ HIGH 问题 ≤ 3 个
- ✅ 安全审查通过
- ✅ 文档已更新

## 最佳实践

1. **提供清晰的描述** - 详细说明功能需求和技术约束
2. **使用快速模式** - 对于简单功能，使用 `--quick` 加速
3. **监控进度** - 使用 `--verbose` 查看详细执行过程
4. **处理失败** - 使用 `--continue` 从失败点继续
5. **审查结果** - 仔细阅读最终报告

## 注意事项

- 阶段 1 并行执行，时间取决于最慢的智能体
- 阶段 2 可并行多个 tdd-guide 实例（独立模块）
- 阶段 3 并行执行，节省约 50% 时间
- 阶段 4 可通过 `--quick` 跳过

## 相关文档

- **智能体编排**: `AGENTS.md`
- **TDD 工作流**: `commands/tdd.md`
- **代码审查**: `commands/code-review.md`
- **构建修复**: `commands/build-fix.md`
