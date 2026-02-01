---
name: test-skill
version: 1.0.0
author: hw0k
description: A simple test skill to understand skill structure
tags: [test, example]
---

# Test Skill

This is a test skill to demonstrate the basic structure.

## Purpose

This skill shows:
- How to structure a skill file
- How metadata is defined
- How content is organized
- How skills are invoked

## Core Principles

### 1. Example Principle One
- Always use meaningful names
- Keep functions small and focused
- Write tests for critical paths

### 2. Example Principle Two
- Separate concerns
- Minimize dependencies
- Document public APIs

## When Applied

Claude considers this skill when:
- User explicitly invokes it with `/test-skill`
- User asks for test skill examples

## Example Usage

```javascript
// Good example
function calculateTotal(items) {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// Bad example
function calc(x) {
  let t = 0;
  for (let i = 0; i < x.length; i++) {
    t = t + x[i].p;
  }
  return t;
}
```

## Checklist

When reviewing code:
- [ ] Names are descriptive
- [ ] Functions have single purpose
- [ ] Code has tests
- [ ] No magic numbers
