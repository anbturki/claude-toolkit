---
name: clean-code-naming
description: Apply intent-revealing names. Verb-phrase functions, question-form booleans, UPPER_SNAKE constants, one word per concept. Use when reviewing names or extracting variables.
user-invocable: true
allowed-tools: Read, Edit, Grep, Glob
argument-hint: "[file path or scope]"
---

# Clean Naming

> "There are only two hard things in Computer Science: cache invalidation and naming things." - Phil Karlton

## Scope

`$ARGUMENTS`

## Rules

### 1. Variable names reveal intent

`isEligible` not `flag`. `retryCount` not `n`. `userInput` not `data`.

### 2. Function names are verb phrases

`validateInput()` not `inputCheck()`. `fetchUser()` not `userData()`.

### 3. Boolean names are questions

`isLoading`, `hasPermission`, `canEdit`, `shouldRetry`.

### 4. Constants are UPPER_SNAKE_CASE

`MAX_RETRY_COUNT`, `API_TIMEOUT_MS`, `DEFAULT_PAGE_SIZE`.

### 5. Avoid abbreviations unless universally known

OK: `id`, `url`, `db`, `api`, `dto`, `ms`
NOT OK: `usr`, `mgr`, `cfg`, `btn`, `cmpt`, `qty`

### 6. One word per concept across the codebase

Pick one of `fetch` / `get` / `retrieve` and use it consistently. Mixing them creates the impression that they mean different things.

### 7. Names match their level of abstraction

A function in a service layer shouldn't have HTTP-flavored names (`handleRequest`). A database repository shouldn't have business-logic names (`approveOrder`).

## See also

- [[clean-code-srp]] - if you can't name a function with a simple verb phrase, it's doing too much
- [[clean-code-no-magic-values]] - naming the extracted constant is the point

**Source:** Clean Code, Ch. 2
