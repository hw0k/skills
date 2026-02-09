# Engineering Principle Examples

Concrete examples for principles that benefit from illustration. Reference this file from CLAUDE.md when the generated `## Engineering Principles` section needs more context.

## No Premature Abstraction

### Bad: Abstracting after one occurrence

```tsx
// First and only form in the project. Too early to abstract.
function useGenericForm<T>(config: FormConfig<T>) {
  const [values, setValues] = useState<T>(config.initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  // 50 lines of generic form logic...
}

// Used once
const loginForm = useGenericForm<LoginFields>({ ... });
```

### Good: Wait for repetition, then extract the common pattern

```tsx
// After 3 similar forms exist (login, signup, profile edit),
// the shared pattern is now clear and stable.
// Only THEN extract the common parts.
function useFormValidation(rules: ValidationRule[]) {
  // Extract only the validated, repeated pattern
}
```

### Rule of thumb

- 1 occurrence: just write it inline
- 2 occurrences: note the duplication, tolerate it
- 3 occurrences: now the pattern is clear — extract it

## Atomic Commits

### Bad: Mixed concerns in one commit

```
commit: "refactor login page and add avatar upload"

 M src/components/LoginPage.tsx    ← rename + restructure
 M src/api/auth.ts                 ← refactor
 A src/components/AvatarUpload.tsx  ← new feature
 M src/api/user.ts                 ← new endpoint
 M src/styles/global.css            ← style fix
```

Cannot revert the avatar feature without also reverting the login refactor.

### Good: One concern per commit

```
commit 1: "refactor: rename LoginForm to AuthForm"
 M src/components/AuthForm.tsx (renamed)
 M src/pages/LoginPage.tsx (import update)

commit 2: "feat: add avatar upload"
 A src/components/AvatarUpload.tsx
 M src/api/user.ts

commit 3: "fix: correct global button padding"
 M src/styles/global.css
```

Each commit is independently revertible. Each passes tests.

### Conventional Commits quick reference

```
feat:     new feature
fix:      bug fix
refactor: code change that neither fixes nor adds
chore:    build, CI, dependency updates
docs:     documentation only
test:     adding or fixing tests
style:    formatting, no logic change
```

## Decision Authority

### Bad: Agent makes irreversible choice silently

```
I'll use Zustand for state management since it's lightweight.
[proceeds to install and integrate]
```

### Good: Agent presents comparison and asks

```
State management options for this project:

| Criteria       | Zustand        | Jotai          | Redux Toolkit  |
|---------------|----------------|----------------|----------------|
| Bundle size   | ~1KB           | ~2KB           | ~11KB          |
| Learning curve| Low            | Low            | Medium         |
| DevTools      | Basic          | Basic          | Excellent      |
| Middleware    | Minimal        | Minimal        | Rich ecosystem |

Given the project's scale (small~mid SPA), Zustand or Jotai both fit.
Which direction would you prefer?
```

### When to delegate

- Library/framework selection
- Database schema design that's hard to migrate
- API contract changes that affect other teams
- Deletion of code/data that can't be recovered
