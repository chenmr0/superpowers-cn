# SDD 工作流增强 实现计划

> **面向 AI 代理的工作者：** 必需子技能：使用 superpowers:subagent-driven-development（推荐）或 superpowers:executing-plans 逐任务实现此计划。步骤使用复选框（`- [ ]`）语法来跟踪进度。

**目标：** 增强 superpowers-zh 的 SDD 工作流：新增 SPEC.md 生成、简化代码审查为 UT+编译验证、TDD 流程级强制、精简非核心 skills。

**架构：** 渐进式改造，修改现有 skill 文件内容，删除非核心 skill 目录和 deprecated 命令文件，不引入新的基础设施。

**技术栈：** Markdown skill 文件编辑，Git 版本控制

---

## 文件结构

| 操作 | 文件 | 职责 |
|------|------|------|
| 修改 | `skills/brainstorming/SKILL.md` | 新增 SPEC 输出阶段 |
| 修改 | `skills/writing-plans/SKILL.md` | 强制每个任务包含 UT 先行步骤 |
| 修改 | `skills/executing-plans/SKILL.md` | 更新审查逻辑，增加 TDD 强制 |
| 修改 | `skills/subagent-driven-development/SKILL.md` | 简化代码质量审查为 UT+编译，TDD 强制 |
| 修改 | `skills/subagent-driven-development/code-quality-reviewer-prompt.md` | 简化审查模板 |
| 修改 | `skills/verification-before-completion/SKILL.md` | 增加 UT 存在性检查 |
| 修改 | `skills/requesting-code-review/SKILL.md` | 简化审查模板 |
| 修改 | `skills/requesting-code-review/code-reviewer.md` | 简化审查清单 |
| 修改 | `skills/using-superpowers/SKILL.md` | 更新 skill 清单，移除中文技能路由 |
| 修改 | `agents/code-reviewer.md` | 简化审查范围为 UT+编译 |
| 修改 | `package.json` | 更新描述，反映精简后的 skill 数量 |
| 删除 | `skills/chinese-code-review/` | 非核心 skill |
| 删除 | `skills/chinese-git-workflow/` | 非核心 skill |
| 删除 | `skills/chinese-documentation/` | 非核心 skill |
| 删除 | `skills/chinese-commit-conventions/` | 非核心 skill |
| 删除 | `skills/mcp-builder/` | 非核心 skill |
| 删除 | `skills/workflow-runner/` | 非核心 skill |
| 删除 | `commands/brainstorm.md` | deprecated 别名 |
| 删除 | `commands/execute-plan.md` | deprecated 别名 |
| 删除 | `commands/write-plan.md` | deprecated 别名 |

---

### Task 1: 删除非核心 skills 和 deprecated 命令

**文件：**
- 删除：`skills/chinese-code-review/SKILL.md`
- 删除：`skills/chinese-git-workflow/SKILL.md`
- 删除：`skills/chinese-documentation/SKILL.md`
- 删除：`skills/chinese-commit-conventions/SKILL.md`
- 删除：`skills/mcp-builder/SKILL.md`
- 删除：`skills/workflow-runner/SKILL.md`
- 删除：`commands/brainstorm.md`
- 删除：`commands/execute-plan.md`
- 删除：`commands/write-plan.md`

- [ ] **Step 1: 删除 6 个中文原创 skill 目录**

```bash
rm -rf skills/chinese-code-review skills/chinese-git-workflow skills/chinese-documentation skills/chinese-commit-conventions skills/mcp-builder skills/workflow-runner
```

- [ ] **Step 2: 删除 3 个 deprecated 命令文件**

```bash
rm -rf commands/brainstorm.md commands/execute-plan.md commands/write-plan.md
```

- [ ] **Step 3: 如果 commands 目录为空，删除目录**

```bash
rmdir commands 2>/dev/null; echo "done"
```

- [ ] **Step 4: 验证删除结果**

```bash
ls skills/ && echo "---" && ls commands/ 2>/dev/null || echo "commands dir removed"
```

预期：skills 下只有 14 个保留的 skill 目录，commands 目录已删除或为空。

- [ ] **Step 5: Commit**

```bash
git add -A skills/chinese-* skills/mcp-builder skills/workflow-runner commands/
git commit -m "chore: 删除非核心 skills 和 deprecated 命令别名

删除 6 个非核心 skill: chinese-code-review, chinese-git-workflow,
chinese-documentation, chinese-commit-conventions, mcp-builder, workflow-runner
删除 3 个 deprecated 命令: brainstorm, execute-plan, write-plan"
```

---

### Task 2: 修改 brainstorming skill — 新增 SPEC 输出阶段

**文件：**
- 修改：`skills/brainstorming/SKILL.md`

- [ ] **Step 1: 更新检查清单（第 22-32 行区域）**

将检查清单从 9 项改为 11 项，插入 SPEC 相关步骤：

将原检查清单中的第 5-6 项：
```
5. **展示设计** — 按复杂度分节展示，每节展示后获得用户批准
6. **编写设计文档** — 保存到 `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` 并 commit
```

替换为：
```
5. **展示规格说明（SPEC）** — 逐节展示 SPEC 内容（目的、需求、场景、约束、验收标准），每节获得用户批准
6. **编写规格文档** — 保存 SPEC 到 `docs/superpowers/specs/YYYY-MM-DD-<topic>-spec.md` 并 commit
7. **展示设计（DESIGN）** — 基于 SPEC 逐节展示技术设计方案，每节获得用户批准
8. **编写设计文档** — 保存 DESIGN 到 `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` 并 commit
```

后续项编号顺延（原 7→9, 8→10, 9→11），最终检查清单为：

```
1. **探索项目上下文** — 检查文件、文档、最近的 commit
2. **提供视觉伴侣**（如果主题涉及视觉问题）— 这是一条独立的消息，不要与澄清问题合并。参见下方的"视觉伴侣"部分。
3. **提出澄清问题** — 每次一个，了解目的/约束/成功标准
4. **提出 2-3 种方案** — 附带权衡分析和你的推荐
5. **展示规格说明（SPEC）** — 逐节展示 SPEC 内容（目的、需求、场景、约束、验收标准），每节获得用户批准
6. **编写规格文档** — 保存 SPEC 到 `docs/superpowers/specs/YYYY-MM-DD-<topic>-spec.md` 并 commit
7. **展示设计（DESIGN）** — 基于 SPEC 逐节展示技术设计方案，每节获得用户批准
8. **编写设计文档** — 保存 DESIGN 到 `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` 并 commit
9. **规格自检** — 快速内联检查占位符、矛盾、模糊性、范围、SPEC 与 DESIGN 一致性（详见下方）
10. **用户审查书面规格** — 在继续之前请用户审查 SPEC 和 DESIGN 文件
11. **过渡到实现** — 调用 writing-plans 技能创建实现计划
```

- [ ] **Step 2: 更新流程图（第 36-64 行区域）**

将原流程图中的：
```
"分节展示设计" [shape=box];
"用户批准设计?" [shape=diamond];
"编写设计文档" [shape=box];
```

替换为：
```
"分节展示 SPEC" [shape=box];
"用户批准 SPEC?" [shape=diamond];
"编写 SPEC 文档" [shape=box];
"分节展示 DESIGN" [shape=box];
"用户批准 DESIGN?" [shape=diamond];
"编写 DESIGN 文档" [shape=box];
```

并更新连线：
```
"提出 2-3 种方案" -> "分节展示 SPEC";
"分节展示 SPEC" -> "用户批准 SPEC?";
"用户批准 SPEC?" -> "分节展示 SPEC" [label="否，修改"];
"用户批准 SPEC?" -> "编写 SPEC 文档" [label="是"];
"编写 SPEC 文档" -> "分节展示 DESIGN";
"分节展示 DESIGN" -> "用户批准 DESIGN?";
"用户批准 DESIGN?" -> "分节展示 DESIGN" [label="否，修改"];
"用户批准 DESIGN?" -> "编写 DESIGN 文档" [label="是"];
"编写 DESIGN 文档" -> "规格自检\n（内联修复）";
```

- [ ] **Step 3: 更新"展示设计"章节（第 86-92 行区域）**

在原"展示设计"章节之前，新增"展示规格说明"章节：

```markdown
**展示规格说明（SPEC）：**

- 一旦你认为理解了要构建的内容，先展示规格说明
- SPEC 使用 RFC 2119 关键词（SHALL/MUST/SHOULD）和 GIVEN/WHEN/THEN 场景格式
- SPEC 结构：
  - 目的：一段话描述功能要解决的问题
  - 需求：每个需求包含名称、描述（SHALL/MUST/SHOULD）和多个场景（GIVEN/WHEN/THEN）
  - 约束：非功能性约束（性能、安全、兼容性等）
  - 验收标准：可检查的完成条件清单
- 每个部分展示后询问是否正确
- 获得用户对 SPEC 的完全批准后，再进入 DESIGN 阶段

**展示设计（DESIGN）：**
```

原"展示设计"的内容保留，作为 DESIGN 阶段的指导。将原开头 "一旦你认为理解了要构建的内容，就展示设计" 改为 "基于已批准的 SPEC，展示技术设计方案"。

- [ ] **Step 4: 更新"设计之后"章节的文档部分（第 107-114 行区域）**

将原文档部分从只输出 design.md 改为输出两个文件：

```markdown
**文档：**

- 将验证通过的规格说明写入 `docs/superpowers/specs/YYYY-MM-DD-<topic>-spec.md`
- 将验证通过的设计文档写入 `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`
  - （用户对规格位置的偏好优先于此默认值）
- 如果可用，使用 elements-of-style:writing-clearly-and-concisely 技能
- 将两个文档 commit 到 git
```

- [ ] **Step 5: 更新"规格自检"章节（第 116-124 行区域）**

在自检 4 项后增加第 5 项：

```markdown
5. **SPEC-DESIGN 一致性：** DESIGN 中的技术方案是否覆盖了 SPEC 中的所有需求？是否有 SPEC 中未提及的设计决策？
```

- [ ] **Step 6: 更新"用户审查关卡"（第 126-131 行区域）**

将审查提示从单个文件改为两个文件：

```markdown
> "SPEC 已写入 `<spec-path>`，DESIGN 已写入 `<design-path>`。请审查两个文件，如果在我们开始编写实现计划之前你想做任何修改，请告诉我。"
```

- [ ] **Step 7: 验证修改后的文件格式**

运行：`head -70 skills/brainstorming/SKILL.md`
预期：检查清单包含 11 项，流程图包含 SPEC 和 DESIGN 两个阶段。

- [ ] **Step 8: Commit**

```bash
git add skills/brainstorming/SKILL.md
git commit -m "feat(brainstorming): 新增 SPEC 输出阶段，流程改为 SPEC → DESIGN

brainstorming 现在输出两个文档：
- SPEC.md: 需求规格（RFC 2119 + GIVEN/WHEN/THEN）
- DESIGN.md: 技术设计
审批流程：先审批 SPEC，再审批 DESIGN"
```

---

### Task 3: 修改 writing-plans skill — TDD 流程级强制

**文件：**
- 修改：`skills/writing-plans/SKILL.md`

- [ ] **Step 1: 在"任务结构"模板中增加 TDD 强制说明**

在文件头部"概述"区域（第 10 行附近），在 `DRY。YAGNI。TDD。频繁 commit。` 之后增加：

```markdown

<TDD-ENFORCEMENT>
计划的每个实现任务必须包含两个子步骤：
1. **先写失败的单元测试** — 步骤必须包含完整测试代码
2. **再写最少实现代码** — 步骤必须包含完整实现代码

不包含 UT 先行步骤的实现任务是无效计划，必须修正。
纯配置、文档、样式类任务可豁免，但必须明确标注"本任务不涉及业务逻辑"。
</TDD-ENFORCEMENT>
```

- [ ] **Step 2: 更新"小步骤任务粒度"章节**

将原文（第 38-43 行）：
```markdown
**每步是一个操作（2-5 分钟）：**
- "编写失败的测试" - 一步
- "运行它确认失败" - 一步
- "实现最少代码让测试通过" - 一步
- "运行测试确认通过" - 一步
- "Commit" - 一步
```

改为：
```markdown
**每步是一个操作（2-5 分钟）：**
- "编写失败的测试" - 一步（必须）
- "运行它确认失败" - 一步（必须，确认测试因功能缺失而非拼写错误失败）
- "实现最少代码让测试通过" - 一步（必须）
- "运行测试确认通过" - 一步（必须）
- "Commit" - 一步

**TDD 强制：** 以上红-绿循环是每个实现任务的标准模板。如果任务涉及业务逻辑，跳过任何一步都是计划缺陷。
```

- [ ] **Step 3: 更新自检章节增加 TDD 检查项**

在"自检"章节的 3 项检查后增加第 4 项：

```markdown
**4. TDD 合规检查：** 每个涉及业务逻辑的任务是否都包含"先写测试 → 验证失败 → 写实现 → 验证通过"的完整步骤？是否有任务跳过了 UT 直接写实现？
```

- [ ] **Step 4: 验证修改**

运行：`grep -n "TDD" skills/writing-plans/SKILL.md`
预期：至少 4 处提到 TDD（概述、粒度、模板、自检）。

- [ ] **Step 5: Commit**

```bash
git add skills/writing-plans/SKILL.md
git commit -m "feat(writing-plans): TDD 流程级强制 — 每个实现任务必须先写 UT

- 新增 TDD-ENFORCEMENT 硬门控
- 任务粒度强调红-绿循环为标准模板
- 自检增加 TDD 合规检查项"
```

---

### Task 4: 修改 executing-plans skill — 简化审查 + TDD 强制

**文件：**
- 修改：`skills/executing-plans/SKILL.md`

- [ ] **Step 1: 在步骤 2"执行任务"中增加 TDD 强制检查**

在"执行实现"子项（第 51 行附近）后增加：

```markdown
5. **验证 TDD 顺序** — 如果任务涉及业务逻辑，确认测试先于实现存在。如果发现先写了业务代码但无对应 UT，停止执行该任务并上报。
```

- [ ] **Step 2: 更新"批量审查检查点"（第 83-85 行区域）**

将原文：
```markdown
**批量审查检查点：**
- 每完成 3 个任务后，暂停回顾：整体方向还对吗？有没有偏离计划？
- 如果发现前面的实现有问题，先修复再继续，不要带着问题往下走
```

改为：
```markdown
**批量审查检查点：**
- 每完成 3 个任务后，暂停回顾：
  - 整体方向还对吗？有没有偏离计划？
  - 每个任务是否遵循了 TDD（UT 先行）？
  - 运行项目编译和单元测试，确认全部通过
- 如果发现前面的实现有问题，先修复再继续，不要带着问题往下走
```

- [ ] **Step 3: 验证修改**

运行：`grep -n "TDD\|UT\|单元测试" skills/executing-plans/SKILL.md`
预期：至少 2 处提到 TDD/UT 检查。

- [ ] **Step 4: Commit**

```bash
git add skills/executing-plans/SKILL.md
git commit -m "feat(executing-plans): 增加 TDD 顺序验证和编译/UT 检查点

- 执行任务时验证 TDD 顺序（UT 先行）
- 批量审查检查点增加编译和单元测试验证"
```

---

### Task 5: 修改 subagent-driven-development — 简化审查 + TDD 强制

**文件：**
- 修改：`skills/subagent-driven-development/SKILL.md`
- 修改：`skills/subagent-driven-development/code-quality-reviewer-prompt.md`

- [ ] **Step 1: 更新 SKILL.md 的两阶段审查描述**

将文件开头区域（第 8 行附近）的：
```markdown
通过为每个任务分派一个全新的子智能体来执行计划，每个任务完成后进行两阶段审查：先审查规格合规性，再审查代码质量。
```

改为：
```markdown
通过为每个任务分派一个全新的子智能体来执行计划，每个任务完成后进行两阶段审查：先审查规格合规性，再进行构建验证（UT+编译）。
```

将核心原则（第 12 行）：
```markdown
**核心原则：** 每个任务一个全新子智能体 + 两阶段审查（先规格后质量）= 高质量、快速迭代
```

改为：
```markdown
**核心原则：** 每个任务一个全新子智能体 + 两阶段审查（先规格合规，后构建验证）= 高质量、快速迭代
```

- [ ] **Step 2: 更新流程图中的标签**

将流程图中（第 55-58 行区域）：
```
"分派代码质量审查子智能体 (./code-quality-reviewer-prompt.md)" [shape=box];
"代码质量审查子智能体通过?" [shape=diamond];
"实现子智能体修复质量问题" [shape=box];
```

改为：
```
"分派构建验证子智能体 (./code-quality-reviewer-prompt.md)" [shape=box];
"构建验证通过?" [shape=diamond];
"实现子智能体修复构建问题" [shape=box];
```

对应的连线也更新：
```
"规格审查子智能体确认代码匹配规格?" -> "分派构建验证子智能体 (./code-quality-reviewer-prompt.md)" [label="是"];
"分派构建验证子智能体 (./code-quality-reviewer-prompt.md)" -> "构建验证通过?";
"构建验证通过?" -> "实现子智能体修复构建问题" [label="否"];
"实现子智能体修复构建问题" -> "分派构建验证子智能体 (./code-quality-reviewer-prompt.md)" [label="重新验证"];
"构建验证通过?" -> "在 TodoWrite 中标记任务完成" [label="是"];
```

- [ ] **Step 3: 更新红线区域**

将红线区域中的：
```markdown
- 跳过审查（规格合规性或代码质量）
```

改为：
```markdown
- 跳过审查（规格合规性或构建验证）
```

将：
```markdown
- **在规格合规性审查通过之前开始代码质量审查**（顺序错误）
```

改为：
```markdown
- **在规格合规性审查通过之前开始构建验证**（顺序错误）
```

- [ ] **Step 4: 在红线区域末尾增加 TDD 强制红线**

在红线列表中增加：

```markdown
- **在实现子智能体指令中省略 TDD 要求**（每个实现子智能体必须遵循 TDD）
- **允许子智能体跳过"先写测试"步骤**（业务代码必须有先行的失败测试）
```

- [ ] **Step 5: 更新 implementer-prompt.md 中的 TDD 强制**

将 implementer-prompt.md 中的（第 33 行附近）：
```markdown
2. 编写测试（如果任务要求则遵循 TDD）
```

改为：
```markdown
2. **严格遵循 TDD**：先写失败的单元测试 → 验证失败 → 写最少实现代码 → 验证通过。如果任务涉及业务逻辑，跳过此顺序是不可接受的。
```

- [ ] **Step 6: 重写 code-quality-reviewer-prompt.md**

将 `skills/subagent-driven-development/code-quality-reviewer-prompt.md` 全文替换为：

```markdown
# 构建验证审查者提示词模板

分派构建验证子智能体时使用此模板。

**目的：** 验证实现是否能正确编译并通过所有单元测试

**仅在规格合规性审查通过后才分派。**

```
Task tool (general-purpose):
  description: "构建验证：任务 N"
  prompt: |
    你正在验证一个实现的构建状态。

    ## 实现者声称构建了什么

    [来自实现者的报告]

    ## 你的工作

    执行以下两项验证：

    ### 1. 编译验证
    - 运行项目构建命令（如 `npm run build`、`cargo build`、`go build` 等）
    - 确认构建成功，无编译错误
    - 记录构建输出

    ### 2. 单元测试验证
    - 运行相关单元测试（如 `npm test`、`cargo test`、`go test` 等）
    - 确认所有测试通过
    - 记录测试结果

    ## 报告格式

    - ✅ 构建验证通过（如果编译成功且所有 UT 通过）
    - ❌ 构建失败：[具体列出编译错误]
    - ❌ 测试失败：[具体列出失败的测试和错误信息]
```
```

- [ ] **Step 7: 验证修改**

运行：`grep -n "构建验证\|TDD" skills/subagent-driven-development/SKILL.md`
预期：至少 3 处提到构建验证或 TDD。

- [ ] **Step 8: Commit**

```bash
git add skills/subagent-driven-development/
git commit -m "feat(subagent-driven): 简化代码质量审查为构建验证 + TDD 强制

- Phase 2 从代码质量审查改为构建验证（UT+编译）
- 更新流程图和红线区域
- implementer-prompt 强制 TDD 顺序
- code-quality-reviewer-prompt 重写为构建验证模板"
```

---

### Task 6: 修改 verification-before-completion — 增加 UT 存在性检查

**文件：**
- 修改：`skills/verification-before-completion/SKILL.md`

- [ ] **Step 1: 在"常见失败模式"表格中增加 UT 检查行**

在表格末尾（第 50 行附近）追加一行：

```markdown
| TDD 合规 | 业务代码有对应 UT，且 UT 先于实现存在 | 只有业务代码无 UT、UT 后补 |
```

- [ ] **Step 2: 在"关键模式"区域增加 TDD 验证模式**

在"需求"模式块之后（第 100 行附近）增加：

```markdown
**TDD 合规：**
```
✅ 检查每个实现文件 → 确认存在对应测试文件 → 确认测试覆盖关键行为 → "TDD 合规"
❌ "写了代码，应该没问题"（没有 UT 覆盖验证）
```
```

- [ ] **Step 3: 验证修改**

运行：`grep -n "TDD\|UT" skills/verification-before-completion/SKILL.md`
预期：至少 3 处提到 TDD 或 UT。

- [ ] **Step 4: Commit**

```bash
git add skills/verification-before-completion/SKILL.md
git commit -m "feat(verification): 增加 TDD 合规和 UT 存在性验证

- 失败模式表格增加 TDD 合规检查行
- 关键模式增加 TDD 合规验证模板"
```

---

### Task 7: 修改 requesting-code-review 和 code-reviewer — 简化审查范围

**文件：**
- 修改：`skills/requesting-code-review/SKILL.md`
- 修改：`skills/requesting-code-review/code-reviewer.md`
- 修改：`agents/code-reviewer.md`

- [ ] **Step 1: 简化 requesting-code-review/SKILL.md**

将"如何请求"章节中（第 36 行附近）的占位符说明之后，增加审查范围说明：

```markdown
**审查范围：** 代码质量审查聚焦于构建验证——确认 UT 通过和编译成功。不再检查代码风格、架构、安全性等。
```

- [ ] **Step 2: 简化 requesting-code-review/code-reviewer.md**

将 `skills/requesting-code-review/code-reviewer.md` 中的审查清单（第 31-61 行）替换为：

```markdown
## 审查清单

**构建验证：**
- 项目是否能成功编译/构建？
- 构建命令是什么？运行结果如何？

**单元测试：**
- 所有单元测试是否通过？
- 测试命令是什么？运行结果如何？
- 有无跳过的测试？如有，原因是什么？

**需求（来自规格合规审查）：**
- 是否满足了规格中的所有需求？（此部分已在 Phase 1 规格合规审查中覆盖，此处仅做二次确认）
```

- [ ] **Step 3: 简化 agents/code-reviewer.md**

将 `agents/code-reviewer.md` 中的"代码质量评估"、"架构和设计审查"、"文档和标准"部分（第 19-35 行）替换为精简版：

```markdown
2. **构建验证**：
   - 运行项目构建命令，确认编译成功
   - 运行相关单元测试，确认全部通过
   - 记录构建和测试的完整输出

3. **需求确认**：
   - 快速确认所有计划的功能是否都已实现
   - 此步骤与 Phase 1 规格合规审查互补，不重复深入分析
```

删除第 4 部分"文档和标准"。

- [ ] **Step 4: 验证修改**

运行：`grep -c "架构\|安全性\|性能\|SOLID" skills/requesting-code-review/code-reviewer.md agents/code-reviewer.md`
预期：输出为 0 或极低（这些关键词已从审查范围移除）。

- [ ] **Step 5: Commit**

```bash
git add skills/requesting-code-review/ agents/code-reviewer.md
git commit -m "feat(code-review): 简化审查范围为构建验证（UT+编译）

- requesting-code-review 增加审查范围说明
- code-reviewer 模板简化为构建验证清单
- code-reviewer agent 去除架构/安全/文档审查"
```

---

### Task 8: 修改 using-superpowers — 更新 skill 清单

**文件：**
- 修改：`skills/using-superpowers/SKILL.md`

- [ ] **Step 1: 删除"中国特色技能路由"整个章节**

将第 109-127 行的整个"中国特色技能路由"章节删除：

```markdown
## 中国特色技能路由

当检测到以下场景时，**必须**优先调用对应的中国特色技能：

| 场景 | 调用技能 |
|------|---------|
| 代码审查且团队使用中文沟通 | **superpowers:chinese-code-review** |
| 使用 Gitee/Coding/极狐 GitLab | **superpowers:chinese-git-workflow** |
| 编写中文技术文档或 README | **superpowers:chinese-documentation** |
| 编写 git commit message（中文项目） | **superpowers:chinese-commit-conventions** |
| 构建 MCP 服务器/工具 | **superpowers:mcp-builder** |

**判断依据：**
- 项目中有中文注释、中文 README、或 .gitee 目录 → 启用中文系列技能
- commit 历史中有中文 → 使用中文提交规范
- 用户用中文交流 → 所有输出使用中文，优先考虑中国特色技能

中国特色技能与翻译技能**叠加使用**，不互斥。例如：做代码审查时，同时使用 requesting-code-review（流程）+ chinese-code-review（风格）。
```

- [ ] **Step 2: 更新"技能优先级"中的例子**

将（第 104 行附近）：
```markdown
2. **实现技能其次**（前端设计、mcp-builder）- 这些指导执行
```

改为：
```markdown
2. **实现技能其次**（test-driven-development、writing-plans）- 这些指导执行
```

- [ ] **Step 3: 验证修改**

运行：`grep -c "chinese-\|mcp-builder\|workflow-runner" skills/using-superpowers/SKILL.md`
预期：0（所有已删除 skill 的引用已移除）。

- [ ] **Step 4: Commit**

```bash
git add skills/using-superpowers/SKILL.md
git commit -m "feat(using-superpowers): 移除已删除技能的引用和中文技能路由

- 删除中国特色技能路由章节
- 更新技能优先级示例"
```

---

### Task 9: 更新 package.json

**文件：**
- 修改：`package.json`

- [ ] **Step 1: 更新 description 字段**

将 description（第 7 行）：
```json
"description": "AI 编程超能力中文增强版 — superpowers（99k+ ⭐）完整汉化 + 6 个中国原创 skills，支持 Claude Code / Copilot CLI / Hermes Agent / Cursor / Claw Code / Windsurf / Kiro / Gemini CLI 等 17 款工具"
```

改为：
```json
"description": "AI 编程超能力中文增强版 — superpowers（99k+ ⭐）完整汉化，聚焦 SDD 开发流程，支持 Claude Code / Copilot CLI / Hermes Agent / Cursor / Claw Code / Windsurf / Kiro / Gemini CLI 等 17 款工具"
```

- [ ] **Step 2: 验证 package.json 合法性**

运行：`node -e "JSON.parse(require('fs').readFileSync('package.json','utf8')); console.log('valid')"`
预期：输出 "valid"。

- [ ] **Step 3: Commit**

```bash
git add package.json
git commit -m "chore: 更新 package.json 描述，反映精简后的定位"
```

---

### Task 10: 最终验证

**文件：**
- 无修改，只做验证

- [ ] **Step 1: 验证 skills 目录只包含 14 个保留 skill**

```bash
ls -d skills/*/ | sed 's|skills/||;s|/||' | sort
```

预期输出（14 个目录）：
```
brainstorming
dispatching-parallel-agents
executing-plans
finishing-a-development-branch
receiving-code-review
requesting-code-review
subagent-driven-development
systematic-debugging
test-driven-development
using-git-worktrees
using-superpowers
verification-before-completion
writing-plans
writing-skills
```

- [ ] **Step 2: 验证 commands 目录已删除**

```bash
ls commands/ 2>/dev/null && echo "FAIL: commands dir still exists" || echo "OK: commands dir removed"
```

预期：输出 "OK: commands dir removed"

- [ ] **Step 3: 验证所有 skill 的 frontmatter 格式正确**

```bash
for f in skills/*/SKILL.md; do echo "=== $f ==="; head -5 "$f"; echo; done
```

预期：每个 SKILL.md 都有 `---` 开头的 frontmatter，包含 name 和 description 字段。

- [ ] **Step 4: 验证不再引用已删除的 skills**

```bash
grep -r "chinese-code-review\|chinese-git-workflow\|chinese-documentation\|chinese-commit-conventions\|mcp-builder\|workflow-runner" skills/ agents/ 2>/dev/null || echo "OK: no references to deleted skills"
```

预期：输出 "OK: no references to deleted skills"

- [ ] **Step 5: 验证 TDD 强制在 3 个环节都存在**

```bash
echo "=== writing-plans ===" && grep -c "TDD" skills/writing-plans/SKILL.md
echo "=== executing-plans ===" && grep -c "TDD\|UT" skills/executing-plans/SKILL.md
echo "=== subagent-driven ===" && grep -c "TDD" skills/subagent-driven-development/SKILL.md
echo "=== verification ===" && grep -c "TDD\|UT" skills/verification-before-completion/SKILL.md
```

预期：每个文件至少有 2 处提到 TDD 或 UT。

- [ ] **Step 6: 验证 SPEC 生成在 brainstorming 中存在**

```bash
grep -c "SPEC" skills/brainstorming/SKILL.md
```

预期：至少 5 处提到 SPEC。

- [ ] **Step 7: 输出最终验证报告**

```bash
echo "=== Skills count ===" && ls -d skills/*/ | wc -l
echo "=== Git log ===" && git log --oneline -10
```
