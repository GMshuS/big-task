# AGENTS.md - Development Guidelines

## Project Overview

This repository contains a design for a custom command system focused on state persistence and task orchestration across AI conversations. The project is in early design phase.

---

## Build/Lint/Test Commands

### Current State

No build system, test framework, or linting configuration exists yet. When adding these, follow the commands below.

### Standard Commands (to be implemented)

```bash
# Install dependencies
npm install

# Build project
npm run build

# Run linter
npm run lint

# Fix linting errors automatically
npm run lint:fix

# Run type checker
npm run typecheck

# Run all tests
npm test

# Run a single test file
npm test -- path/to/test.spec.ts

# Run tests in watch mode
npm test -- --watch

# Run tests with coverage
npm test -- --coverage

# Format code
npm run format
```

### CI/CD Commands

```bash
npm run ci  # Runs lint, typecheck, and tests
```

---

## Code Style Guidelines

### General Principles

- Write clean, readable code that minimizes cognitive load
- Prefer explicit over implicit
- Keep functions small and focused (single responsibility)
- Avoid premature optimization

### TypeScript/JavaScript Conventions

#### Naming Conventions

| Type        | Convention                            | Example                                   |
| ----------- | ------------------------------------- | ----------------------------------------- |
| Variables   | camelCase                             | `userData`, `isLoading`                   |
| Functions   | camelCase                             | `fetchUserData()`                         |
| Classes     | PascalCase                            | `UserService`, `TaskManager`              |
| Interfaces  | PascalCase with `I` prefix (optional) | `User` or `IUser`                         |
| Types       | PascalCase                            | `TaskStatus`, `CommandResult`             |
| Constants   | UPPER_SNAKE_CASE                      | `MAX_RETRIES`, `API_BASE_URL`             |
| Enums       | PascalCase (members as well)          | `TaskStatus.Pending`                      |
| Files       | kebab-case                            | `user-service.ts`, `task-manager.spec.ts` |
| Directories | kebab-case                            | `src/utils/`, `tests/unit/`               |

#### Type Definitions

- Use explicit return types for public functions
- Avoid `any` type; use `unknown` when type is truly unknown
- Use `interface` for object shapes, `type` for unions/intersections
- Prefer `type` over `enum` for simple unions

```typescript
// Good
function processTask(taskId: string): Promise<TaskResult> { ... }
type Status = 'pending' | 'running' | 'completed';
interface TaskConfig { ... }

// Avoid
function processTask(taskId: any): any { ... }
```

#### Null/Undefined Handling

- Use optional chaining (`?.`) and nullish coalescing (`??`) when appropriate
- Prefer `undefined` over `null` for optional values
- Use definite assignment assertions (`!`) sparingly

### Import Organization

Order imports as follows, with blank lines between groups:

```typescript
// 1. Node.js built-ins
import fs from "fs";
import path from "path";

// 2. External packages
import chalk from "chalk";
import { z } from "zod";

// 3. Internal packages (workspace packages)
import "@project/shared-utils";

// 4. Relative imports - current directory
import { helper } from "./helper";
import { constants } from "./constants";

// 5. Relative imports - parent/sibling directories
import { config } from "../config";
import { utils } from "../../utils";
```

### Formatting

- Use 2 spaces for indentation
- Use single quotes for strings
- Use semicolons
- Maximum line length: 100 characters
- Add trailing commas in multiline structures
- Use template literals for string interpolation

```typescript
// Good
const message = `Hello, ${userName}!`;
const config = {
  option1: true,
  option2: "value",
};

// Avoid
const message = "Hello, " + userName + "!";
const config = { option1: true, option2: "value" };
```

### Error Handling

- Use typed errors when possible
- Include context in error messages
- Use try/catch with async/await
- Create custom error classes for domain-specific errors

```typescript
// Good
class TaskExecutionError extends Error {
  constructor(
    public readonly taskId: string,
    message: string,
  ) {
    super(`Task ${taskId}: ${message}`);
    this.name = "TaskExecutionError";
  }
}

// Usage
try {
  await executeTask(taskId);
} catch (error) {
  if (error instanceof TaskExecutionError) {
    logger.error(error.taskId, error.message);
  }
  throw error;
}
```

### Async/Await

- Always use async/await over raw Promises
- Handle errors with try/catch
- Avoid unhandled promise rejections

### Comments

- Write self-documenting code; add comments only for complex logic
- Use JSDoc for public APIs
- Avoid commented-out code

### Testing Guidelines

- Test file naming: `*.spec.ts` or `*.test.ts`
- Place tests alongside source files or in `tests/` directory
- Follow AAA pattern: Arrange, Act, Assert
- Test one thing per test case
- Use descriptive test names that explain the expected behavior

```typescript
describe("TaskManager", () => {
  describe("executeTask", () => {
    it("should retry failed tasks up to MAX_RETRIES times", async () => {
      // Arrange
      const manager = new TaskManager({ maxRetries: 3 });
      const failingTask = createFailingTask();

      // Act
      const result = await manager.executeTask(failingTask);

      // Assert
      expect(result.status).toBe("failed");
      expect(failingTask.attemptCount).toBe(3);
    });
  });
});
```

---

## File Structure

```
src/
  ├── index.ts          # Main entry point
  ├── commands/         # Command implementations
  ├── utils/            # Utility functions
  └── types/            # TypeScript types/interfaces
tests/
  ├── unit/             # Unit tests
  └── integration/      # Integration tests
config/                 # Configuration files
scripts/                # Build/utility scripts
```

---

## Documentation

- Update README.md when adding features
- Document breaking changes in CHANGELOG.md
- Use clear, concise commit messages (Conventional Commits)

---

## Task Orchestration System

This project uses custom commands for state persistence and task orchestration across AI conversations.

### Commands

| Command                                       | Description                                                         |
| --------------------------------------------- | ------------------------------------------------------------------- |
| `/bigtask-plan <project-name> <requirements>` | Split requirements into structured task plan for a specific project |
| `/bigtask-execute <project-name>/<task-id>`   | Execute a specific subtask by ID                                    |
| `/bigtask-status [project-name]`              | List all tasks and their status (optionally for a specific project) |
| `/bigtask-aggregate <project-name>`           | Aggregate all subtask outputs into final result                     |

### Workflow

1. **Task Planning** (`/bigtask-plan project-a build a user auth system`): Creates `tasks/project-a/task_plan.md` with task IDs, dependencies
2. **Task Execution** (`/bigtask-execute project-a/task-1`): Runs tasks in isolated `tasks/project-a/` directory
3. **Aggregation** (`/bigtask-aggregate project-a`): Creates `tasks/project-a/final-result.md` and `status-summary.json`

### State Files

- `tasks/<project-name>/task_plan.md` - Master task list with dependencies
- `tasks/<project-name>/<task-id>-state.json` - Execution state for each task
- `tasks/<project-name>/outputs/` - Task output files
- `tasks/<project-name>/final-result.md` - Aggregated output
- `tasks/<project-name>/status-summary.json` - Project execution status

### Key Features

- **Multi-project support**: Run multiple independent projects in parallel
- **Isolation**: Each project has its own `tasks/<name>/` directory
- **Independence**: Projects don't interfere with each other

---

## Editor Setup

Recommended extensions:

- ESLint
- Prettier
- TypeScript Hero (for imports)
