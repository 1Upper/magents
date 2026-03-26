---
name: csharp-review
description: >
  Code review skill for C# pull requests. Examines correctness, style,
  security, performance, and maintainability using C#/.NET conventions.
version: 1.0.0
---

# C# Code Review Skill

You are performing a code review on a C# pull request. Follow the steps below to produce a thorough, actionable review.

## Step 1: Gather PR Information

1. Retrieve the PR diff:
   ```
   gh pr diff
   ```
2. Retrieve the PR description and metadata:
   ```
   gh pr view
   ```
3. Note all changed files, paying attention to `.cs`, `.csproj`, and `.sln` files.

## Step 2: Analyse the Diff

Review every changed `.cs` file against the following checklist. For each finding, record:

- **File and line number**
- **Severity**: `critical` | `major` | `minor` | `suggestion`
- **Category** (see below)
- **Description** of the issue and a recommended fix

### Review Categories

#### Correctness
- Logic errors, off-by-one errors, incorrect null checks.
- Improper use of value types vs reference types.
- Race conditions, thread-safety issues, or improper use of `lock`.
- `async`/`await` pitfalls: missing `await`, `.Result`/`.Wait()` causing deadlocks, fire-and-forget exceptions.
- Incorrect `IDisposable`/`using` patterns that could cause resource leaks.

#### C# Conventions & Style
- Naming: `PascalCase` for types, methods, and properties; `camelCase` for local variables and parameters; `_camelCase` for private fields; `I` prefix for interfaces.
- Prefer `var` when the type is obvious from the right-hand side; avoid it when it obscures the type.
- Use expression-bodied members, pattern matching, and other modern C# features where they improve clarity.
- File-scoped namespaces (C# 10+) preferred when the project targets .NET 6+.
- Remove unused `using` directives and dead code.

#### Nullable Reference Types
- Ensure nullable annotations (`?`) are correct and consistent with the project's `<Nullable>` setting.
- Null-dereference risks where nullability is not properly handled.
- Prefer `??`, `?.`, and null-coalescing assignment over explicit null checks where idiomatic.

#### Exception Handling
- Avoid catching `Exception` or `SystemException` without re-throwing or a clear justification.
- Do not swallow exceptions silently (empty `catch` blocks).
- Prefer specific exception types; avoid using exceptions for control flow.
- Ensure `finally` / `using` blocks always release resources.

#### Security
- SQL injection: raw string interpolation into queries instead of parameterised queries or an ORM.
- Path traversal: unsanitised user input used in file paths.
- Sensitive data (passwords, tokens, keys) hard-coded or logged.
- Improper use of `[Authorize]` / missing authentication/authorisation checks in ASP.NET Core controllers.
- Insecure deserialization (e.g., `BinaryFormatter`, unrestricted `JsonSerializer` type handling).
- Missing input validation or model-binding validation (`[Required]`, `ModelState.IsValid`).

#### Performance
- LINQ queries executed multiple times over the same enumerable; prefer materialising with `.ToList()` / `.ToArray()` where appropriate.
- Unnecessary boxing/unboxing or allocations in hot paths.
- `string` concatenation in loops; prefer `StringBuilder` or interpolation.
- Synchronous I/O on async-capable paths.
- Missing `ConfigureAwait(false)` in library code.
- Large objects kept in memory longer than necessary.

#### Maintainability & Design
- SOLID principles: single responsibility, open/closed, Liskov substitution, interface segregation, dependency inversion.
- Methods or classes that are too long or do too much; recommend extraction.
- Magic numbers/strings; prefer named constants or enumerations.
- Missing or inadequate XML doc comments on public API members.
- Test coverage: are new public methods covered by unit tests?

## Step 3: Summarise Findings

After completing the analysis, produce a structured review summary in the following format:

```
## C# Code Review Summary

### Critical Issues
<list or "None">

### Major Issues
<list or "None">

### Minor Issues & Suggestions
<list or "None">

### Positive Observations
<brief notes on what was done well>

### Overall Recommendation
[ ] Approve
[ ] Approve with minor comments
[ ] Request changes
```

## Step 4: Post the Review

Post the summary as a PR review comment:

```
gh pr review --comment --body "<summary>"
```

If there are `critical` or `major` issues, use `--request-changes` instead of `--comment`.
