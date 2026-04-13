# AGENTS.md - Development Guidelines

## Project Overview

This repository contains a design for a custom command system focused on state persistence and task orchestration across AI conversations. The project is in early design phase.

---

## File Structure

```
.opencode/              # opencode工具私有目录
  ├── commands          # 存放commands的目录
task_design.md          # commands设计的原型
task_readme.md          # commands使用说明
```

---

## Documentation

- Update task_design.md and task_readme.md when adding features
- Document breaking changes in CHANGELOG.md
- Use clear, concise commit messages (Conventional Commits)

---

## Task Orchestration System

This project uses custom commands for state persistence and task orchestration across AI conversations.

### Commands

| Command                                       | Description                                                         |
| --------------------------------------------- | ------------------------------------------------------------------- |
| `/bigtask-brainstorm <project-name> <主题>`   | 任务头脑风暴阶段：在bigtask-plan之前探索需求、方案思路、技术选型等  |
| `/bigtask-plan <project-name> <requirements>` | Split requirements into structured task plan for a specific project |
| `/bigtask-execute <project-name>/<task-id>`   | Execute a specific subtask by ID                                    |
| `/bigtask-status [project-name]`              | List all tasks and their status (optionally for a specific project) |
| `/bigtask-aggregate <project-name>`           | Aggregate all subtask outputs into final result                     |

### Workflow

1. **头脑风暴** (`/bigtask-brainstorm project-a 用户认证系统`): 探索需求、收集方案思路、评估技术选型，生成 `tasks/project-a/brainstorm.md`
2. **任务规划** (`/bigtask-plan project-a build a user auth system`): 在头脑风暴基础上创建任务计划 `task_plan.md`
3. **任务执行** (`/bigtask-execute project-a/task-1`): 执行各子任务
4. **聚合结果** (`/bigtask-aggregate project-a`): 整合所有输出

### State Files

- `tasks/<project-name>/brainstorm.md` - Brainstorming output (exploration, ideas, tech choices)
- `tasks/<project-name>/task_plan.md` - Master task list with dependencies
- `tasks/<project-name>/<task-id>-state.json` - Execution state for each task
- `tasks/<project-name>/outputs/` - Task output files
- `tasks/<project-name>/final-result.md` - Aggregated output
- `tasks/<project-name>/status-summary.json` - Project execution status

### Key Features

- **Multi-project support**: Run multiple independent projects in parallel
- **Isolation**: Each project has its own `tasks/<name>/` directory
- **Independence**: Projects don't interfere with each other
