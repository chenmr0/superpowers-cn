# Svelte Todo List - 规格说明

> 本文档定义系统应该做什么（WHAT），不涉及如何实现。

## ADDED Requirements

### Requirement: 添加待办事项
系统 SHALL 允许用户通过在输入框中输入文本并按 Enter 键或点击 Add 按钮来添加新的待办事项。

#### Scenario: 通过 Enter 键添加待办
- **WHEN** 用户在输入框中输入"Buy groceries"并按 Enter 键
- **THEN** 系统创建一个新的未完成待办事项，文本为"Buy groceries"，并清空输入框

#### Scenario: 通过 Add 按钮添加待办
- **WHEN** 用户在输入框中输入"Walk the dog"并点击 Add 按钮
- **THEN** 系统创建一个新的未完成待办事项，文本为"Walk the dog"，并清空输入框

### Requirement: 切换待办完成状态
系统 SHALL 允许用户通过点击复选框来切换待办事项的完成状态。

#### Scenario: 标记待办为已完成
- **WHEN** 用户点击一个未完成待办事项的复选框
- **THEN** 该待办事项变为已完成状态

#### Scenario: 取消待办的完成状态
- **WHEN** 用户点击一个已完成待办事项的复选框
- **THEN** 该待办事项变为未完成状态

### Requirement: 删除待办事项
系统 SHALL 允许用户通过点击 X 按钮来删除待办事项。

#### Scenario: 删除单个待办
- **WHEN** 用户点击某个待办事项的 X 按钮
- **THEN** 该待办事项从列表中移除

### Requirement: 按状态筛选待办
系统 SHALL 提供筛选功能，允许用户按 All / Active / Completed 查看待办事项。

#### Scenario: 筛选 Active 待办
- **WHEN** 用户点击 Active 筛选按钮
- **THEN** 列表只显示未完成的待办事项

#### Scenario: 筛选 Completed 待办
- **WHEN** 用户点击 Completed 筛选按钮
- **THEN** 列表只显示已完成的待办事项

#### Scenario: 显示全部待办
- **WHEN** 用户点击 All 筛选按钮
- **THEN** 列表显示所有待办事项（已完成和未完成）

### Requirement: 显示剩余待办数量
系统 SHALL 显示未完成待办事项的数量。

#### Scenario: 显示正确计数
- **WHEN** 列表中有 3 个待办事项，其中 1 个已完成
- **THEN** 显示"2 items left"

### Requirement: 清除已完成的待办
系统 SHALL 允许用户一键清除所有已完成的待办事项。

#### Scenario: 清除所有已完成待办
- **WHEN** 用户点击"Clear completed"按钮
- **THEN** 所有已完成的待办事项从列表中移除

### Requirement: 本地持久化
系统 SHALL 将待办事项持久化到 localStorage，使其在页面刷新后仍然存在。

#### Scenario: 刷新后数据保留
- **WHEN** 用户添加了待办事项后刷新页面
- **THEN** 所有待办事项及其完成状态保持不变

### Requirement: 空状态提示
系统 SHALL 在没有待办事项时显示帮助信息。

#### Scenario: 无待办时显示提示
- **WHEN** 待办列表为空
- **THEN** 显示有用的空状态提示信息
