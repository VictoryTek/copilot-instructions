# Project Audit & Ideas Prompt

Paste this prompt into Claude Code at the start of a session.
No modifications needed — it is project-agnostic.

---

## PROMPT

Perform a full project audit in two parts. Do not make any changes to any files.
This is analysis and reporting only.

---

### PART 1: PROJECT HEALTH AUDIT

Read the entire repository. Then produce a structured report covering every
section below. For each finding, include the affected file path and line number
where relevant.

#### 1.1 Structure & Architecture
- Does the project follow a consistent architectural pattern?
- Are there files or directories that are misplaced, misnamed, or orphaned?
- Are there any circular dependencies or import cycles?
- Is there dead code — unused functions, modules, exports, or variables?
- Are there files that have grown too large and should be split?

#### 1.2 Correctness & Bugs
- Are there logic errors, off-by-one errors, or incorrect conditionals?
- Are there unhandled error paths, missing null checks, or silent failures?
- Are there race conditions, unsafe assumptions about state, or timing issues?
- Are there hardcoded values that should be constants or configuration?
- Are there TODO, FIXME, HACK, or XXX comments — list every one with its location?

#### 1.3 Consistency
- Are naming conventions consistent across files (variables, functions, files, types)?
- Are formatting and style consistent, or do different files follow different patterns?
- Are similar problems solved in different ways in different parts of the codebase?
- Are error messages and log output consistent in format and tone?

#### 1.4 Configuration & Environment
- Are there missing, redundant, or conflicting configuration entries?
- Are secrets, tokens, or credentials at risk of being committed?
- Are environment-specific values properly separated from code?
- Are dependency versions pinned, unpinned, or conflicting?
- Are there deprecated dependencies that should be updated?

#### 1.5 Documentation
- Are there undocumented public functions, modules, or APIs?
- Is existing documentation accurate, or does it contradict the current implementation?
- Is the README present, complete, and up to date?
- Are setup, build, and deployment instructions present and correct?

#### 1.6 Tests
- What is the apparent test coverage? Are critical paths untested?
- Are there tests that are incomplete, skipped, or always passing trivially?
- Are test names descriptive and consistent?
- Are there missing edge case tests for known risky areas of the code?

#### 1.7 Security
- Are there injection risks, unsafe deserialization, or input validation gaps?
- Are there overly permissive file permissions, exposed endpoints, or open redirects?
- Are dependencies known to have published vulnerabilities?
- Are authentication and authorization applied consistently?

#### 1.8 Performance
- Are there obvious inefficiencies — N+1 queries, unnecessary loops, redundant I/O?
- Are expensive operations cached where they should be?
- Are there memory leaks or unbounded growth patterns?

---

### PART 1 SUMMARY

After all sections, produce:

**Issue Severity Table**

| Severity | Count | Examples |
|----------|-------|---------|
| 🔴 Critical | X | brief description |
| 🟠 High | X | brief description |
| 🟡 Medium | X | brief description |
| 🔵 Low | X | brief description |
| ⚪ Info | X | brief description |

**Overall Health Score: X / 10**

Include one paragraph explaining the score.

---

### PART 2: IDEAS & RECOMMENDATIONS

Based on your analysis of the codebase, its apparent purpose, and its current state,
produce the following. These should be specific to THIS project — not generic advice.

#### 2.1 Quick Wins (low effort, high value)
List 3–5 improvements that could be made in under an hour each.
For each: what it is, why it matters, and which file(s) it affects.

#### 2.2 Feature Recommendations
List 5–10 features that would meaningfully improve this project.
For each:
- Feature name
- What problem it solves or value it adds
- Rough implementation complexity (Low / Medium / High)
- Which existing parts of the codebase it would build on or interact with

#### 2.3 Refactoring Opportunities
List 3–5 structural improvements that would make the codebase easier to maintain.
For each: what to change, why, and what the risk of doing it is.

#### 2.4 Tooling & Workflow Suggestions
Are there missing tools, scripts, or automation that would improve the development
experience for this specific project? (e.g. missing linter config, absent pre-commit
hooks, no local dev script, missing CI step)

---

### OUTPUT FORMAT

- Use the section headers exactly as written above
- Use file paths and line numbers for every specific finding
- Do not suggest changes outside this report — this is read-only analysis
- Do not truncate any section — if a section has no findings, write "None identified"
- Save the full report to:

```
.github/docs/subagent_docs/project_audit_[YYYY-MM-DD].md
```

Replace [YYYY-MM-DD] with today's date.

- This prompt is agent-agnostic. Before beginning the audit, check for the
  active instruction file in this order:
  1. `CLAUDE.md` (Claude Code / VS Code extension)
  2. `.github/copilot-instructions.md` (GitHub Copilot)
  - If found: read it and follow all absolute rules, forbidden commands,
    resource constraints, and project-specific constraints defined there
    throughout this audit. This is a read-only task — do not trigger the
    full workflow — but all safety rules still apply.
  - If neither is found: proceed using general best practices.
