# 代码质量审查者提示词模板

分派代码质量审查子智能体时使用此模板。

**目的：** 验证实现可以编译且所有测试通过

**仅在规格合规性审查通过后才分派。**

```
Task tool (general-purpose):
  description: "验证任务 N 的编译和测试"
  prompt: |
    你正在验证一个实现的基本质量门禁。

    ## 任务摘要

    [任务摘要]

    ## Git 范围

    Base: [BASE_SHA]
    Head: [HEAD_SHA]

    ## 你的工作

    **仅执行以下两项检查，不做其他任何事：**

    1. **编译检查：** 运行项目的编译/构建命令，确认无编译错误
    2. **测试检查：** 运行项目的单元测试命令，确认所有测试通过

    常用命令（根据项目类型选择）：
    - TypeScript/JavaScript: `npm run build` 和 `npm test`
    - Python: 无显式编译步骤，运行 `pytest` 或 `python -m pytest`
    - Go: `go build ./...` 和 `go test ./...`
    - Java: `mvn compile` 和 `mvn test`
    - Rust: `cargo build` 和 `cargo test`

    如果不确定项目的构建/测试命令，检查 package.json、Makefile、
    Cargo.toml、pom.xml 等项目配置文件。

    ## 报告格式

    报告：
    - **编译：** ✅ 通过 / ❌ 失败（附错误信息）
    - **测试：** ✅ 通过 / ❌ 失败（附失败测试名称和错误信息）
    - **结论：** 通过（两者均通过）/ 不通过（任一失败）
```

**代码审查者返回：** 编译状态、测试状态、结论（通过/不通过）
