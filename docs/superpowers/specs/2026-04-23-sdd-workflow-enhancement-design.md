# SDD 工作流增强设计

日期: 2026-04-23

## 概述

对 superpowers-zh 的 SDD（Spec-Driven Development）工作流进行四项增强：

1. 新增 SPEC.md 生成阶段
2. 简化代码质量审查为 UT+编译验证
3. TDD 流程级强制
4. 精简非核心 skills

## 1. SPEC.md 生成

### 流程变更

当前流程:
```
brainstorming → design.md → plan.md
```

新流程:
```
brainstorming → SPEC.md → DESIGN.md → plan.md
```

### SPEC.md 结构

参考 OpenSpec 的规格格式，使用 RFC 2119 关键词和 GIVEN/WHEN/THEN 场景：

```markdown
# [Feature Name] 规格说明书

## 目的
[一段话描述功能要解决的问题]

## 需求

### 需求: [需求名称]
[描述，使用 SHALL/MUST/SHOULD]

#### 场景: [场景名称]
- GIVEN [前置条件]
- WHEN [操作]
- THEN [预期结果]

## 约束
[非功能性约束：性能、安全、兼容性等]

## 验收标准
[可检查的完成条件清单]
```

### brainstorming skill 修改

1. "Present design" 阶段拆分为 SPEC 审批 + DESIGN 审批
2. 输出两个文件：
   - `docs/superpowers/specs/YYYY-MM-DD-<topic>-spec.md`
   - `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`
3. Spec self-review 增加 SPEC 一致性检查（SPEC 与 DESIGN 无矛盾）
4. 用户审批门控：先审批 SPEC，再审批 DESIGN

## 2. 两阶段审查简化

### Phase 1: Spec 合规性审查（不变）

检查代码是否符合 plan/spec 要求。

### Phase 2: 构建验证（简化）

当前检查项（代码组织、命名、模式、测试覆盖、错误处理、安全性、性能、文档）简化为：

1. **UT 通过**: 运行相关单元测试，全部通过
2. **编译通过**: 项目能成功构建，无编译错误

审查模板：
```
## Phase 2: 构建验证
- [ ] 单元测试全部通过: `[测试命令]`
- [ ] 项目编译成功: `[构建命令]`
如有失败，记录错误信息并要求修复。
```

### 涉及修改的 skills

- **subagent-driven-development**: 代码质量审查模板只保留 UT+编译
- **requesting-code-review**: review agent 检查项模板简化
- **executing-plans**: checkpoint 验证逻辑更新

## 3. TDD 流程级强制

在三个环节增加流程级强制，确保 UT 先于业务代码：

### 环节 1: writing-plans（计划阶段）

- 每个实现任务的模板必须包含两个子步骤：
  - Step 1: 为 [功能] 编写失败的单元测试
  - Step 2: 实现最小代码使测试通过
- HARD-GATE 增加：计划中每个实现任务必须包含 UT 先行步骤，否则拒绝该计划

### 环节 2: executing-plans / subagent-driven-development（执行阶段）

- 执行任务时强制按 "先写 UT → 验证失败 → 再写业务代码" 顺序
- 任务执行模板禁止跳过 UT 步骤
- HARD-GATE：如果任务涉及业务代码但无 UT 步骤，拒绝执行并报错

### 环节 3: verification-before-completion（验证阶段）

- 增加检查项：每个实现任务是否有对应的 UT
- 如果发现业务代码无 UT，不标记完成

### 设计决策

- 不做自动回滚，而是阻止继续并要求补写 UT
- 强制点嵌入流程各阶段，而非单独检测，降低模型跳过的概率

## 4. Skills 精简

### 删除列表

| Skill | 原因 |
|-------|------|
| chinese-code-review | 非核心 SDD 流程 |
| chinese-git-workflow | 非核心 SDD 流程 |
| chinese-documentation | 非核心 SDD 流程 |
| chinese-commit-conventions | 非核心 SDD 流程 |
| mcp-builder | 非核心 SDD 流程，特定领域 |
| workflow-runner | 非核心 SDD 流程 |

### 保留列表（14 skills）

| 分类 | Skills |
|------|--------|
| 核心 SDD | brainstorming, writing-plans, executing-plans, test-driven-development, using-superpowers |
| 执行/工作树 | dispatching-parallel-agents, subagent-driven-development, using-git-worktrees, finishing-a-development-branch |
| 审查/验证 | requesting-code-review, receiving-code-review, verification-before-completion |
| 调试/技能编写 | systematic-debugging, writing-skills |

### 附加清理

删除 3 个 deprecated 别名：
- `superpowers:brainstorm` → 应使用 `superpowers:brainstorming`
- `superpowers:execute-plan` → 应使用 `superpowers:executing-plans`
- `superpowers:write-plan` → 应使用 `superpowers:writing-plans`

## 影响范围

### 需要修改的 skills

1. **using-superpowers** — 更新 skill 清单，移除已删除 skills 和 deprecated 别名
2. **brainstorming** — 增加 SPEC 输出阶段，修改审批流程
3. **writing-plans** — 强制每个任务包含 UT 先行步骤
4. **executing-plans** — 简化审查，增加 TDD 强制检查
5. **subagent-driven-development** — 简化代码质量审查为 UT+编译，增加 TDD 强制
6. **verification-before-completion** — 增加 UT 存在性检查
7. **requesting-code-review** — 简化审查模板

### 需要删除的 skills

1. chinese-code-review
2. chinese-git-workflow
3. chinese-documentation
4. chinese-commit-conventions
5. mcp-builder
6. workflow-runner

### 需要删除的 deprecated 别名

1. superpowers:brainstorm
2. superpowers:execute-plan
3. superpowers:write-plan

### 配置文件更新

- **package.json**: 更新 skill 元数据，移除已删除 skills
- **agents/**: 如果有 code-reviewer agent 引用了已删除 skills，需要更新
