# Svelte Todo List - 技术设计

> 本文档定义系统如何实现（HOW），基于规格说明中的需求。

**关联规格：** `spec.md`

## Context

构建一个简单的待办事项列表应用，使用 Svelte 框架。需要支持创建、完成、删除待办事项，并提供 localStorage 持久化。

## Goals / Non-Goals

### Goals
- 提供简洁的待办事项管理界面
- 通过 localStorage 实现数据持久化
- 支持按完成状态筛选

### Non-Goals
- 不做用户认证/多用户支持
- 不做云端同步
- 不做拖拽排序

## Decisions

### 前端框架选择
- **选择：** 使用 Svelte
- **理由：** 轻量、响应式、编译时优化
- **备选方案：** React（更重）、Vanilla JS（手动 DOM 管理繁琐）

### 状态管理
- **选择：** 使用 Svelte store（writable）
- **理由：** 内置于 Svelte，无需额外依赖，适合简单应用
- **备选方案：** Redux（过度工程化）

### 持久化方案
- **选择：** localStorage
- **理由：** 无需后端，满足单用户本地需求
- **备选方案：** IndexedDB（对于简单数据过于复杂）

## Architecture

### 用户界面

```
┌─────────────────────────────────────────┐
│  Svelte Todos                           │
├─────────────────────────────────────────┤
│  [________________________] [Add]       │
├─────────────────────────────────────────┤
│  [ ] Buy groceries                  [x] │
│  [✓] Walk the dog                   [x] │
│  [ ] Write code                     [x] │
├─────────────────────────────────────────┤
│  2 items left                           │
│  [All] [Active] [Completed]  [Clear ✓]  │
└─────────────────────────────────────────┘
```

### 组件结构

```
src/
  App.svelte           # Main app, state management
  lib/
    TodoInput.svelte   # Text input + Add button
    TodoList.svelte    # List container
    TodoItem.svelte    # Single todo with checkbox, text, delete
    FilterBar.svelte   # Filter buttons + clear completed
    store.ts           # Svelte store for todos
    storage.ts         # localStorage persistence
```

### 数据模型

```typescript
interface Todo {
  id: string;        // UUID
  text: string;      // Todo text
  completed: boolean;
}

type Filter = 'all' | 'active' | 'completed';
```

## Risks / Trade-offs

- localStorage 容量限制（~5MB）：对于待办事项列表足够 → 无需额外缓解
- 无云端备份：用户清除浏览器数据会丢失 → Non-Goal，不做云端同步
