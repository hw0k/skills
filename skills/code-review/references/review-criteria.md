# Review Criteria Examples

Concrete good/bad examples for each review principle. Load this when you need to illustrate a finding during review.

## Dead Code

### Bad

```ts
import { useState, useEffect, useCallback } from "react"; // useCallback unused

function processOrder(order: Order) {
  // const oldLogic = order.items.reduce((sum, i) => sum + i.price, 0);
  const total = calculateTotal(order);
  return total;
}
```

Three violations: unused import (`useCallback`), commented-out code (`oldLogic`), unnecessary intermediate variable (`total` could be returned directly — though this alone is a NIT, combined with the rest it shows carelessness).

### Good

```ts
import { useState, useEffect } from "react";

function processOrder(order: Order) {
  return calculateTotal(order);
}
```

## Naming

### Bad

```ts
const d = new Date(); // what is d?
const flag = user.role === "admin"; // flag of what?
function proc(data: unknown[]) { /* ... */ } // proc what?
function getUsers(onlyActive: boolean) { /* ... */ } // boolean flag changes behavior
```

### Good

```ts
const createdAt = new Date();
const isAdmin = user.role === "admin";
function sanitizeInputRecords(records: unknown[]) { /* ... */ }
function getActiveUsers() { /* ... */ }
function getAllUsers() { /* ... */ }
```

## Function Design

### Bad: Too many responsibilities

```ts
async function handleSubmit(form: FormData) {
  const errors = validate(form);
  if (errors.length > 0) {
    setErrors(errors);
    trackEvent("form_validation_failed", { errors });
    return;
  }
  setLoading(true);
  try {
    const response = await api.post("/users", form);
    setUser(response.data);
    trackEvent("user_created", { id: response.data.id });
    toast.success("Account created!");
    router.push("/dashboard");
  } catch (err) {
    if (err.status === 409) {
      setErrors(["Email already exists"]);
    } else {
      setErrors(["Something went wrong"]);
      reportError(err);
    }
  } finally {
    setLoading(false);
  }
}
```

Validation, API call, analytics, navigation, error handling, UI updates — all in one function (~25 lines). Hard to test, hard to modify.

### Good: Separated concerns

```ts
async function handleSubmit(form: FormData) {
  const errors = validate(form);
  if (errors.length > 0) {
    onValidationFailed(errors);
    return;
  }
  await createUser(form);
}

async function createUser(form: FormData) {
  setLoading(true);
  try {
    const user = await api.post("/users", form);
    onUserCreated(user.data);
  } catch (err) {
    onCreateUserFailed(err);
  } finally {
    setLoading(false);
  }
}
```

Each function has one job. Easier to test and modify independently.

### Bad: Boolean flag parameter

```ts
function formatDate(date: Date, includeTime: boolean) {
  if (includeTime) {
    return date.toISOString();
  }
  return date.toISOString().split("T")[0];
}
```

### Good: Two focused functions

```ts
function formatDate(date: Date) {
  return date.toISOString().split("T")[0];
}

function formatDateTime(date: Date) {
  return date.toISOString();
}
```

## Error Handling

### Bad

```ts
try {
  await saveDocument(doc);
} catch (e) {
  // ignore
}

try {
  const data = await fetchUser(id);
} catch (error) {
  throw new Error("Something went wrong");
}
```

First: swallowed error. Second: generic message that loses context.

### Good

```ts
try {
  await saveDocument(doc);
} catch (error) {
  logger.error("Failed to save document", { documentId: doc.id, error });
  throw new DocumentSaveError(doc.id, { cause: error });
}

try {
  const data = await fetchUser(id);
} catch (error) {
  throw new UserNotFoundError(id, { cause: error });
}
```

## Immutability

### Bad

```ts
function addTag(item: Item, tag: string) {
  item.tags.push(tag); // mutates argument
  return item;
}

let count = 0; // mutable, declared with let unnecessarily
items.forEach((item) => {
  if (item.active) count++;
});
```

### Good

```ts
function addTag(item: Item, tag: string): Item {
  return { ...item, tags: [...item.tags, tag] };
}

const count = items.filter((item) => item.active).length;
```

## Magic Values

### Bad

```ts
if (password.length < 8) { /* ... */ }
setTimeout(retry, 3000);
if (user.role === "admin") { /* ... */ }
const tax = price * 0.1;
```

### Good

```ts
const MIN_PASSWORD_LENGTH = 8;
if (password.length < MIN_PASSWORD_LENGTH) { /* ... */ }

const RETRY_DELAY_MS = 3000;
setTimeout(retry, RETRY_DELAY_MS);

const ROLE = { ADMIN: "admin", USER: "user" } as const;
if (user.role === ROLE.ADMIN) { /* ... */ }

const TAX_RATE = 0.1;
const tax = price * TAX_RATE;
```

## Security

### Bad

```ts
const query = `SELECT * FROM users WHERE id = '${userId}'`; // SQL injection
const html = `<div>${userInput}</div>`; // XSS
const filePath = path.join(uploadDir, req.params.filename); // path traversal
```

### Good

```ts
const query = "SELECT * FROM users WHERE id = $1";
const result = await db.query(query, [userId]);

const sanitized = DOMPurify.sanitize(userInput);

const filename = path.basename(req.params.filename); // strip directory components
const filePath = path.join(uploadDir, filename);
```

## Test Quality

### Bad: Testing implementation

```ts
test("handleClick", () => {
  const spy = jest.spyOn(component, "setState");
  component.handleClick();
  expect(spy).toHaveBeenCalledWith({ count: 1 });
});
```

### Good: Testing behavior

```ts
test("increments counter when button is clicked", () => {
  render(<Counter />);
  fireEvent.click(screen.getByRole("button", { name: "Increment" }));
  expect(screen.getByText("Count: 1")).toBeInTheDocument();
});
```

### Bad: Duplicating production logic

```ts
test("calculates total correctly", () => {
  const items = [{ price: 10 }, { price: 20 }];
  // This duplicates the production reduce logic — if the production
  // code has a bug, this test has the same bug
  const expected = items.reduce((sum, i) => sum + i.price, 0);
  expect(calculateTotal(items)).toBe(expected);
});
```

### Good: Assert against known values

```ts
test("calculates total correctly", () => {
  const items = [{ price: 10 }, { price: 20 }];
  expect(calculateTotal(items)).toBe(30);
});
```
