# 智能体编排

## 1. 智能体概览

### 1.1 智能体快速参考表

| 智能体 | 职责 | 触发时机 | 默认技能 | 优先级 | 阶段 |
|--------|------|----------|----------|--------|------|
| planner | 实现规划 | 复杂功能、重构 | planning-with-files | HIGH | 1 |
| architect | 系统设计 | 架构决策 | - | HIGH | 1 |
| tdd-guide | 测试驱动开发 | 新功能、错误修复 | tdd-workflow | CRITICAL | 2 |
| code-reviewer | 代码审查 | 编写代码后 | coding-standards | HIGH | 3 |
| security-reviewer | 安全分析 | 提交前 | security-review | CRITICAL | 3 |
| build-error-resolver | 修复构建错误 | 构建失败时 | - | CRITICAL | 2,3 |
| e2e-runner | 端到端测试 | 关键用户流程 | - | MEDIUM | 3 |
| refactor-cleaner | 清理死代码 | 代码维护 | coding-standards | LOW | 4 |
| doc-updater | 文档 | 更新文档时 | - | LOW | 4 |

### 1.2 智能体详细说明

#### planner

- **配置文件**: `agents/planner.md`
- **核心能力**: 分析需求、创建实施计划、分解任务、识别风险
- **默认技能**: `planning-with-files`
- **场景技能**: `search-first`（研究新功能）、`iterative-retrieval`（复杂查询）
- **输出**: 结构化实施计划（包含阶段、步骤、依赖、风险）

#### architect

- **配置文件**: `agents/architect.md`
- **核心能力**: 系统架构设计、技术决策、可扩展性分析
- **默认技能**: 无
- **场景技能**: `docker-patterns`（Docker 集成）、`web-design-guidelines`（前端架构）
- **输出**: 架构设计文档、架构决策记录（ADRs）

#### tdd-guide

- **配置文件**: `agents/tdd-guide.md`
- **核心能力**: TDD 循环指导、测试编写、覆盖率验证
- **默认技能**: `tdd-workflow`
- **场景技能**:
  - Vue 项目: `vue-expert`, `vue-testing-best-practices`
  - 数据库相关: `dbhub`
- **输出**: 测试代码、实现代码、覆盖率报告

#### code-reviewer

- **配置文件**: `agents/code-reviewer.md`
- **核心能力**: 代码质量审查、安全检查、最佳实践验证
- **默认技能**: `coding-standards`
- **场景技能**:
  - Vue 项目: `vue-best-practices`, `vueuse-functions`
- **输出**: 代码审查报告（包含问题分类和修复建议）

#### security-reviewer

- **配置文件**: `agents/security-reviewer.md`
- **核心能力**: 安全漏洞检测、密钥泄露检测、OWASP Top 10 检查
- **默认技能**: `security-review`
- **场景技能**: 无
- **输出**: 安全审查报告（包含漏洞严重性分级）

#### build-error-resolver

- **配置文件**: `agents/build-error-resolver.md`
- **核心能力**: TypeScript 错误修复、构建错误修复、依赖问题解决
- **默认技能**: 无
- **场景技能**: 无
- **输出**: 修复后的代码、修复说明

#### e2e-runner

- **配置文件**: `agents/e2e-runner.md`
- **核心能力**: E2E 测试生成、执行、不稳定测试管理
- **默认技能**: 无
- **场景技能**: 无
- **输出**: E2E 测试报告、测试工件（截图、视频、追踪）

#### refactor-cleaner

- **配置文件**: `agents/refactor-cleaner.md`
- **核心能力**: 死代码检测、重复项消除、安全重构
- **默认技能**: `coding-standards`
- **场景技能**: 无
- **输出**: 重构报告、清理的代码清单

#### doc-updater

- **配置文件**: `agents/doc-updater.md`
- **核心能力**: 代码映射图生成、文档更新、API 文档提取
- **默认技能**: 无
- **场景技能**: 无
- **输出**: 更新的文档、代码映射图

---

## 2. 技能加载配置

### 2.1 默认技能集（启动时自动加载）

```yaml
default_skills:
  planner:
    - planning-with-files

  tdd-guide:
    - tdd-workflow

  code-reviewer:
    - coding-standards

  security-reviewer:
    - security-review

  refactor-cleaner:
    - coding-standards

  architect: []
  build-error-resolver: []
  e2e-runner: []
  doc-updater: []
```

### 2.2 场景化技能加载规则

#### 场景 1：Vue 项目开发

**触发条件**:
- 检测到 `package.json` 中包含 `"vue"` 依赖
- 或检测到 `*.vue` 文件

**加载规则**:
```yaml
project_type: vue
agent_skill_map:
  tdd-guide:
    add:
      - vue-expert
      - vue-testing-best-practices
    priority: HIGH

  code-reviewer:
    add:
      - vue-best-practices
      - vueuse-functions
    priority: MEDIUM
```

#### 场景 2：数据库操作

**触发条件**:
- 检测到数据库配置文件（`prisma/schema.prisma`, `drizzle.config.ts`）
- 或用户提到 "数据库"、"SQL"

**加载规则**:
```yaml
project_type: database
agent_skill_map:
  tdd-guide:
    add:
      - dbhub
    priority: HIGH

  code-reviewer:
    add:
      - dbhub
    priority: MEDIUM
```

#### 场景 3：Docker 集成

**触发条件**:
- 检测到 `Dockerfile` 或 `docker-compose.yml`

**加载规则**:
```yaml
project_type: docker
agent_skill_map:
  architect:
    add:
      - docker-patterns
    priority: HIGH
```

### 2.3 技能加载优先级

``` text
优先级顺序：
1. CRITICAL - 必须加载
2. HIGH - 强烈建议加载
3. MEDIUM - 建议加载
4. LOW - 可选加载
```

---

## 即时智能体使用

无需用户提示：

1. 复杂的功能请求 - 使用 **planner** 智能体
2. 刚编写/修改的代码 - 使用 **code-reviewer** 智能体
3. 错误修复或新功能 - 使用 **tdd-guide** 智能体
4. 架构决策 - 使用 **architect** 智能体

## 自动化 TDD 工作流

对于完整的功能开发，使用 **`/auto-tdd`** 命令：

```
/auto-tdd Implement user authentication with JWT
```

这将自动执行：

1. **阶段 1**: 需求分析与架构设计（并行: architect + planner）
2. **阶段 2**: 测试驱动开发（循环: tdd-guide）
3. **阶段 3**: 质量保证（并行: code-reviewer + security-reviewer + e2e-runner）
4. **阶段 4**: 优化与文档（并行: refactor-cleaner + doc-updater）

详见: [commands/auto-tdd.md](../commands/auto-tdd.md)

## 并行任务执行

对于独立操作，**始终**使用并行任务执行：

```markdown
# 良好：并行执行
同时启动 3 个智能体：
1. 智能体 1：认证模块的安全分析
2. 智能体 2：缓存系统的性能审查
3. 智能体 3：工具类的类型检查

# 不良：不必要的顺序执行
先智能体 1，然后智能体 2，最后智能体 3

```

## 多视角分析

对于复杂问题，使用拆分角色的子智能体：

* 事实审查员
* 高级工程师
* 安全专家
* 一致性审查员
* 冗余检查器
