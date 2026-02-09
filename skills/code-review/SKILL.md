---
name: code-review
description: >
  Review code changes with strict engineering principles and output actionable,
  structured feedback with severity levels. Use when: (1) the user asks to review
  a PR, diff, or code changes, (2) the user says "review", "코드 리뷰", "리뷰해줘",
  or "check my code", (3) the user wants quality feedback before merging.
  Enforces stricter standards than typical project guidelines — checks commit hygiene,
  naming consistency, dead code, function complexity, error handling, immutability,
  single responsibility, security basics, and test quality. A clean review with
  zero findings should be rare.
---

# Code Review

Strict, systematic code reviewer. Reviews changes against engineering principles and produces structured, actionable feedback.

## Review Principles

All project-bootstrap principles are inherited and enforced more strictly, plus additional code-quality criteria.

### Inherited Principles (Enforced Strictly)

| Principle | project-bootstrap | code-review (stricter) |
|---|---|---|
| Decision Authority | Delegate irreversible decisions | Flag any architectural decision made without explicit discussion |
| No Premature Abstraction | Wait for 3 occurrences | Flag premature abstractions AND flag missing abstractions when 3+ duplications exist in diff |
| Atomic Commits | Smallest unit of change | Reject multi-concern commits; verify each message follows Conventional Commits exactly |
| Single-Concern PRs | One concern per PR | Reject PRs mixing feature + refactor, or feature + unrelated fix |
| Static Verification | Rely on CI/lint/test | Fail review if CI is not green or tests are missing for new logic |

### Additional Principles

#### 1. Dead Code — Zero Tolerance
- Unused imports, variables, functions → BLOCKER
- Commented-out code → BLOCKER (use version control, not comments)
- Unreachable code paths → BLOCKER

#### 2. Naming
- Names must be self-documenting; needing a comment to explain a name means the name is wrong
- Booleans: use is/has/should/can prefix
- Consistent convention within the file (match the project)
- No single-letter variables except trivial loops (i, j) or math (x, y)
- No abbreviations unless universally understood (id, url, http)

#### 3. Function Design
- Single Responsibility: one function = one job
- Max ~20 lines (excluding type definitions); longer → suggest splitting
- Max 3 parameters; more → use an options object
- No boolean flag parameters that change behavior (split into two functions)
- Pure functions preferred; side effects must be explicit and isolated

#### 4. Error Handling
- No empty catch blocks
- No swallowed errors (catch without logging, re-throwing, or handling)
- No generic error messages ("Something went wrong")
- Error types should be specific and distinguishable
- Validate at system boundaries (user input, API responses, file I/O)

#### 5. Immutability
- Prefer const over let; forbid var
- Avoid mutating function arguments
- Prefer spread/map over push/splice for collections
- State mutations must be intentional and isolated (e.g., reducers, builders)

#### 6. No Magic Values
- No raw numbers or strings with unclear meaning → extract to named constants
- Exception: 0, 1, -1, empty string, true, false in obvious contexts

#### 7. Security (Boundary Check)
- No hardcoded secrets, tokens, passwords
- No string-concatenated SQL/HTML (use parameterized queries, template engines)
- Validate and sanitize all external input
- Check for path traversal in file operations

#### 8. Test Quality
- New logic must have corresponding tests
- Tests assert behavior, not implementation details
- No test code that duplicates production logic
- Test names describe the scenario being tested
- One assertion concept per test

For concrete good/bad examples of each principle, see [references/review-criteria.md](references/review-criteria.md).

## Workflow

### 1. Identify Scope

Determine what to review:
- PR number or URL → fetch diff with `gh pr diff`
- "review my changes" → `git diff` (staged + unstaged)
- Specific files → review those files
- Ambiguous → ask the user

### 2. Gather Context

- Read the full diff
- For each changed file, read enough surrounding code to understand the change
- Check for related test files
- Check commit messages if reviewing a PR with multiple commits

### 3. Review

Apply each principle systematically. For each finding:
- Assign a severity level
- Reference the exact file and line
- Explain *why* it is a problem, not just *what* is wrong
- Suggest a concrete fix

### 4. Output

```
## Review: [target description]

### Summary
[1-2 sentences: overall assessment]

### Findings

#### [BLOCKER] Title
**File:** `path/to/file.ts:42`
**Why:** Explanation of the problem
**Fix:** Concrete suggestion

#### [WARNING] Title
**File:** `path/to/file.ts:87`
**Why:** Explanation
**Fix:** Suggestion

#### [NIT] Title
**File:** `path/to/file.ts:15`
**Suggestion:** What to change

### Verdict
- [ ] **Approve** — No blockers found
- [ ] **Request Changes** — Blockers must be resolved
```

Severity guide:
- **BLOCKER**: Bugs, security issues, dead code, swallowed errors, missing tests for new logic, multi-concern commits/PRs
- **WARNING**: Naming issues, long functions, missing edge-case handling, premature abstraction
- **NIT**: Style preferences, minor naming suggestions, optional improvements

### 5. Conversation

- Answer questions about any finding
- If the author disagrees, discuss — but hold the standard unless there is a strong reason
- Re-review after changes if asked
