---
name: project-bootstrap
description: >
  Generate or update a project's CLAUDE.md with the user's engineering principles and project-specific context.
  Use when: (1) bootstrapping a new project and need to create CLAUDE.md,
  (2) the user asks to set up or update CLAUDE.md with their engineering principles,
  (3) the user says "bootstrap", "init project", or "프로젝트 초기 셋업".
  This skill analyzes the codebase to understand the tech stack, then produces
  a CLAUDE.md tailored to the project with actionable engineering guidelines baked in.
---

# Project Bootstrap

Generate or update a project's CLAUDE.md that embeds the user's engineering philosophy as concrete, actionable instructions for Claude.

## Workflow

### 1. Analyze the Project

Before writing CLAUDE.md, understand the project context:

- Detect tech stack (package.json, tsconfig.json, Cargo.toml, etc.)
- Identify existing tooling (linter, formatter, test framework, CI)
- Check for existing CLAUDE.md (update vs. create)
- Note the project structure (monorepo, single app, library, etc.)

### 2. Generate CLAUDE.md

Write CLAUDE.md at the project root. Structure it as follows:

```markdown
# CLAUDE.md

## Project Overview
[1-2 sentences: what this project is, tech stack summary]

## Development Commands
[Common commands: build, test, lint, format, dev server — only what exists]

## Engineering Principles

### Decision Authority
- The final decision-maker for all decisions is the user.
- When an irreversible decision is needed, do not judge right or wrong on your own. Instead, provide comparison materials and delegate the decision to the user.

### No Premature Abstraction
- Do not abstract preemptively. Wait until there are concrete, repeated use cases before introducing abstractions.

### Atomic Commits
- Every commit should target the smallest unit of change.
- Every commit must be in a state where execution and tests pass.
- All commits must follow Conventional Commits.

### Single-Concern Pull Requests
- Every Pull Request should address exactly one concern.

### Prefer Static Verification over Self-Review
- Instead of ambiguous self-review, set up and rely on CI, Lint, Test, and other static checks.

## Code Style
[Only if detectable from existing config — do not invent rules]
```

For concrete good/bad examples of **Decision Authority**, **No Premature Abstraction**, and **Atomic Commits**, see [references/principle-examples.md](references/principle-examples.md). When generating CLAUDE.md for a project, include or adapt relevant examples from that file to make the principles actionable.

### 3. Adaptation Rules

- **Existing CLAUDE.md**: Merge principles into the existing file. Do not overwrite project-specific content (commands, architecture notes) that is already there.
- **Monorepo**: Add a note about per-package structure if relevant.
- **Non-frontend projects**: The principles are stack-agnostic. Adapt examples to the detected stack.
- **Development Commands**: Only list commands that actually exist in the project. Run detection (e.g., check `package.json` scripts) rather than guessing.

### 4. Keep It Concise

CLAUDE.md must be **under 200 lines**. Content beyond this should be split into `agent-references/` and referenced from CLAUDE.md. Every line must justify its token cost.
