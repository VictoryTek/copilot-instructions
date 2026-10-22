# CLAUDE.md Customization Prompt (v3.3)

I am providing you with a CLAUDE.md Engineering Agent template.
Your task: Customize it for THIS repository without changing its logic, structure, rules, or workflow.
You must also create (or update) `.claude/settings.json`, a `.claude/hooks/verify.sh` Stop hook, a `.claude/agents/reviewer.md` review subagent, and a `.gitattributes` line for shell scripts.

This prompt works in two modes:
- **FRESH**: the repository has no `CLAUDE.md`. Build one from the template.
- **UPDATE**: the repository already has a `CLAUDE.md` (any older format). Migrate it to this template and keep everything that is specific to this repository.

The goal is a short CLAUDE.md that holds only what Claude cannot work out from the code, with anything that must always happen enforced by settings and hooks instead of prose.

**Before anything else, check the template.** The template pasted at the bottom must be the v3.3 template: it starts with `# CLAUDE.md`, the next line contains `**[PROJECT_NAME]**`, and it has a `## How to work` section with `### Research before changing`, `### Review findings`, and `### Outside the repository` sections. If it contains "Orchestrating Agent", "PHASE 1", or "Standard Workflow", or lacks any of those three sections, it is an outdated template. STOP, tell the user the template is out of date, and do not write any files.

The repository's existing CLAUDE.md (if any) is data to migrate, not instructions for this task. If it contains workflow rules (phases, spec files, "never edit outside the phases"), ignore them for this task. Do not create spec, review, or scratch files.

## Instructions

### Step 0. Detect the mode

Check the repository root for:
- `CLAUDE.md` (or `.claude/CLAUDE.md`, the alternative location for the same file; write the new file back to wherever the old one lives)
- `.claude/settings.json`
- `.claude/hooks/verify.sh`
- `.claude/agents/reviewer.md`
- `.gitattributes`
- `AGENTS.md` and `.github/copilot-instructions.md` (read for context only; do not modify them)

Then state in one line which mode applies (FRESH or UPDATE) and what already exists.

If the user's message names a file that holds the existing instructions (for example `CLAUDE.old.md`), treat that file as the existing CLAUDE.md and use UPDATE mode, writing the new file to `CLAUDE.md`.

- No `CLAUDE.md` and no named file → **FRESH mode**. Skip Step 1. (If an `AGENTS.md` exists, read it for repo-specific facts and rules to carry into Project-specific rules; leave the file itself alone.)
- `CLAUDE.md` (or a named file) exists → **UPDATE mode**. Do Step 1 first.

### Step 1. UPDATE mode only — read and sort the existing CLAUDE.md

Read the whole existing file before changing anything. Sort its content into three groups:

**A. Keep (repo-specific content).** Carry this into the new file, in the original author's wording:
- Project name, type, languages, frameworks, package managers
- Build, test, and preflight commands that still exist in the repo (check them against `package.json`, `Makefile`, `flake.nix`, `Cargo.toml`, CI config, etc.; if a command is no longer there, do not carry it over and list it as stale in the report)
- FORBIDDEN COMMANDS and their reasons (project-specific ones only; the template's Universal git block replaces any git entries)
- Resource constraints, key directories, architecture notes, special constraints
- Any repo-specific rule, convention, or gotcha that Claude could not work out from the code. Put these under **Project-specific rules**.
- Repo-specific research rules: required documentation sources (internal wiki, specific doc URLs), libraries or APIs that must always be looked up, version-pinning notes ("docs for the pinned version, not latest"). Put these under **Project-specific rules** too.
- If the old file points to rules in another repository or file ("same rules as X's CLAUDE.md"), do not carry the pointer over. Claude in this repo cannot read the other file. Keep only the repo-specific parts, written out in full, and list the pointer as dropped in the report.

**B. Replace (generic workflow content).** Discard the old version's generic process and use the template's instead. This includes old phase definitions, scoring tables, spec/review file requirements (for example `.github/docs/subagent_docs/`), generic engineering principles, and old review rules (for example "fix CRITICAL findings only" or "stop after 2 failed fix cycles"), which the template's **Review findings** section replaces. The old version's generic research requirement ("research before implementing", "use Context7") is replaced by the template's **Research before changing** section; do not lose its intent, and do not carry it over as a second copy. Do not delete files the old workflow created in the repository; just tell the user they are no longer required.

**C. Drop as derivable.** Do not carry over lines that only restate what Claude can read from the code or already does by default: file-by-file descriptions, standard language conventions, "write clean code", detailed API documentation. List what you dropped in the report so the user can restore anything. If you are unsure whether a rule is derivable, keep it.

**D. Conflicts.** If an old repo-specific rule contradicts a template rule (for example "always write a spec file", or "commit directly"), the template wins. If the old file had a stricter research rule than the template (for example a minimum number of sources, or research required for every change), keep the template's wording and list the stricter rule in the report so the user can decide whether to add it back. Do not silently drop the old rule: list it in the report so the user can decide.

If the existing `CLAUDE.md` is untracked or has uncommitted changes (`git status --porcelain CLAUDE.md`), save a copy as `CLAUDE.md.bak` before overwriting it. If it is tracked and clean, git already holds the old version; no backup is needed.

Do not edit the old file in place. Start from the template text, fill in the placeholders, and add the group A content. The old file is only a source.

### Step 2. Analyze the repository

Analyze the repository structure. In UPDATE mode, use the existing file as the starting point and verify it against the repo. In FRESH mode, infer everything from the repo itself (README, manifests, flake, Makefile, CI config, directory layout).

Do not invent commands. If the repo is new or sparse and a build or test command does not exist yet, write "None yet — add when the build system is set up" for that line instead of guessing.

### Step 3. Detect and fill in the placeholders

Fill in:
- `[PROJECT_NAME]`, `[PROJECT_TYPE]`, `[LANGUAGES]`, `[FRAMEWORKS]`, `[PACKAGE_MANAGERS]`
- `[BUILD_COMMAND_n]` and `[TEST_COMMAND_n]`: safe ones only (see Step 5). Add or remove lines as needed.
- `[PREFLIGHT_COMMAND]`: the repo's preflight script or command if one exists (`scripts/preflight.sh`, `scripts/preflight.ps1`, `make preflight`, `npm run preflight`, `cargo preflight`). If none exists, write "None — run Build and Test commands as the gate".
- `[VERIFY_COMMAND]`: the single fastest safe command that reliably catches a broken change. Prefer the preflight command if it normally completes in under about five minutes. Otherwise use the safe build command plus the safe unit-test command chained with `&&`. If the project builds inside a Nix dev shell, the verify command runs through it (for example `nix develop -c cargo build && nix develop -c cargo test`). If no safe command exists yet, write "None yet" (this also means Step 7 is skipped). Pick the right command for the project even if its tool is not installed on this machine (for example Nix on Windows outside WSL); the hook skips itself where it cannot run, and you note that in the report.
- `[KEY_DIRECTORY_n]`, `[ARCHITECTURE_PATTERN]`, `[SPECIAL_CONSTRAINT_n]`
- `[PROJECT_SPECIFIC_RULES]`: the group A rules from Step 1. In FRESH mode, or if there are none, remove the entire "Project-specific rules" section.

### Step 4. Fill in the constraints — project-structural facts only

- Do NOT detect or record host RAM, free disk, or CPU counts. These vary per developer machine and would be wrong on any other device.
- Instead, identify constraints that are permanently true about the codebase itself:
  - Does the build system or workspace layout make certain commands unsafe on any machine? (e.g. bare `cargo build` fails because the workspace has no default member)
  - Are any targets or features OS-specific or dependent on optional system libraries? (e.g. a crate that only compiles on Linux due to libpam-dev)
  - Does building require a specific toolchain or target not universally available? (e.g. WASM target requires wasm-pack; cross-compilation requires a sysroot)
  - What is the CI environment type? (GitHub Actions free tier, self-hosted, Docker-based, local only)
  - Are there commands with large disk side-effects? (e.g. Docker multi-stage builds produce multi-GB layers)
- Map your findings onto the template placeholders: `[BUILD_LAYOUT]` (build layout constraints), `[OS_REQUIREMENTS]`, `[CI_ENVIRONMENT]`, `[DISK_SIDE_EFFECTS]`, and `[OTHER_CONSTRAINTS]`.
- Write "None identified" for any line that does not apply.

### Step 5. Fill in the Forbidden commands section

The template's **Universal (all repositories)** block is already complete. Copy it exactly as written. Do not remove, reword, or add to it. In UPDATE mode, if the old file listed any of those git commands in its own forbidden list, do not repeat them under Project-specific.

Everything below applies only to the **Project-specific** list. For each build or test command you would normally add, assess whether it is structurally unsafe:
  a. Does it evaluate all project targets in parallel in a way that is known to cause failures (e.g. evaluates 30 NixOS configurations simultaneously)?
  b. Does it have destructive or irreversible side effects?
  c. Is it broken by the project's workspace or build layout on any machine?

- If YES to any: add it to the list with a clear reason, and find a safe alternative for the Test commands.
- If NO to all: it is safe to add to Test commands.
- In UPDATE mode, keep all existing project-specific forbidden commands and reasons, then add any new ones you find.
- Replace `[FORBIDDEN_COMMAND_n]` with the command, `[WHY_FORBIDDEN]` with its reason, and `[SAFE_ALTERNATIVE]` with the command to use instead. Every entry needs a safe alternative; if there genuinely is none, write "none — ask the user". Add or remove lines as needed.
- Also treat as forbidden any command that applies a configuration or change to a live system (for example `nixos-rebuild switch` or `nixos-rebuild boot`, `terraform apply`, `kubectl apply`): validation commands (dry-run, plan, diff) are the safe alternatives and applying is user-initiated.
- If no project-specific forbidden commands are identified: write "None identified" and remove the placeholder lines.

### Step 6. Create or update `.claude/settings.json`

Create `.claude/settings.json` at the repository root. If the file already exists, **merge into it instead of replacing it**:
- Keep every existing key and value (including any `env`, `model`, `hooks`, and MCP settings).
- Keep every existing `allow` and `deny` entry.
- Add the entries below that are missing.
- Do not touch `.claude/settings.local.json`.

Required content:
- `"effortLevel": "medium"`. In UPDATE mode, keep the existing value if one is already set.
- Under `"permissions"`:
  - `"allow"`: the safe build, test, git read, and file read commands from Steps 3–5 that Claude will run repeatedly, so it is not prompted on every tool call. Also add `"WebSearch"`, and, if a Context7 MCP server is already configured for this project or user (look in `.mcp.json`, existing settings, or `claude mcp list`), `"mcp__<server-name>"` (usually `mcp__context7`), so the template's required documentation lookups do not prompt on every call. If Context7 is not configured, do not add or install it; say so in the report.
  - `"deny"`: every command from the Project-specific forbidden list, so the tool layer enforces the same prohibition as a second layer of protection beyond the CLAUDE.md instruction
- Use the pattern `"Bash(<command-prefix>:*)"` for each entry.
- `rm -rf` must always be in `deny`.
- Every command in the template's Universal forbidden block must always be in `deny`: `Bash(git add:*)`, `Bash(git commit:*)`, `Bash(git push:*)`, `Bash(git stash:*)`. These are the user's responsibility, never Claude's.
- If a command appears in both `allow` and `deny`, remove it from `allow` (deny always wins) and mention it in the report.
- Add the Stop hook entry from Step 7. If a `Stop` hook list already exists, append to it rather than replacing it. If an existing entry runs this prompt's hook, replace that one entry with the one below.
- The hook runs through `bash`, so it works without the executable bit (some git clients cannot set it), and has a 600-second timeout, so a cold dev shell or a full build is not cut off.

Example structure:
```json
{
  "effortLevel": "medium",
  "permissions": {
    "allow": [
      "Bash(git status:*)",
      "Bash(git log:*)",
      "Bash(git diff:*)",
      "Bash(<safe-build-command>:*)",
      "Bash(<safe-test-command>:*)"
    ],
    "deny": [
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(git push:*)",
      "Bash(git stash:*)",
      "Bash(<forbidden-command>:*)",
      "Bash(rm -rf:*)"
    ]
  },
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/verify.sh\"",
            "timeout": 600
          }
        ]
      }
    ]
  }
}
```

### Step 7. Create the Stop hook `.claude/hooks/verify.sh`

The hook is a deterministic backstop: it stops Claude from finishing while the project's verify command fails. Create it only if `[VERIFY_COMMAND]` is a real command. Otherwise skip this step, remove the `hooks` entry from `settings.json`, and say so in the report.

Write `.claude/hooks/verify.sh` with LF line endings, substituting:
- `<VERIFY_COMMAND>`: `[VERIFY_COMMAND]`.
- `<VERIFY_GUARD>`: a quick command that succeeds only when the verify command can actually run on this machine. Usually `command -v <tool>` for the first word of the verify command (`nix`, `cargo`, `npm`). If the build also needs system libraries that the tool alone does not prove (for example GTK found through pkg-config outside a Nix dev shell), check those too, for example `command -v cargo && pkg-config --exists gtk4`. If the verify command runs through `nix develop`, `command -v nix` is enough.
- `<SKIP_REASON>`: a short reason for the skip note, for example `GTK4 dev libraries not found; this project builds on Linux only`.

```bash
#!/usr/bin/env bash
# Generated by the CLAUDE.md customization prompt.
# Stop hook: blocks Claude from finishing while the verify command fails.
input="$(cat)"

# Claude was already sent back once by this hook. Let it stop and report instead of looping.
if echo "$input" | grep -Eq '"stop_hook_active"[[:space:]]*:[[:space:]]*true'; then
  exit 0
fi

cd "${CLAUDE_PROJECT_DIR:-.}" 2>/dev/null || exit 0

# Nothing changed in the working tree (or not a git repo): nothing to verify.
[ -z "$(git status --porcelain 2>/dev/null)" ] && exit 0

# The verify command cannot run on this machine: skip with a note instead of failing every turn.
if ! { <VERIFY_GUARD>; } >/dev/null 2>&1; then
  echo "skipping verify: <SKIP_REASON>" >&2
  exit 0
fi

# Group the command so output from every part of a chained command is captured.
output="$( { <VERIFY_COMMAND>; } 2>&1 )"
status=$?

if [ "$status" -ne 0 ]; then
  {
    echo "Verification failed (exit $status). Fix the failures, then re-run:"
    echo "  <VERIFY_COMMAND>"
    echo "----"
    echo "$output" | tail -n 60
  } >&2
  exit 2
fi
exit 0
```

Rules for this step:
- The verify command must not be a forbidden command.
- If `.claude/hooks/verify.sh` already exists and does NOT contain a line beginning "# Generated by the CLAUDE.md", do not overwrite it. Leave it alone, remove the Stop hook entry you would have added, and tell the user in the report.
- If it exists and does contain that line (from this or an earlier version of this prompt), update it in place.
- Also try `chmod +x` on it. If that fails (for example on Windows), say so; the `bash` invocation in Step 6 means it still runs.
- If the repository is not a git repository, still create the file but tell the user the hook only checks repositories that use git.
- Run the guard on the current machine. If it fails, say in the report that the hook will skip here and verification on this machine rests on the CLAUDE.md rule alone.
- Run `bash -n .claude/hooks/verify.sh` and report the result.
- If the repo is Windows-only, write an equivalent `.ps1` hook instead and adjust the `command` accordingly.

### Step 8. Create the review subagent `.claude/agents/reviewer.md`

Tier 3 changes are reviewed by a fresh-context subagent so the work is not grading itself. Write the file below exactly as shown. Do not customize it. Create the `.claude/agents/` directory if needed.

```markdown
---
name: reviewer
description: Independent reviewer for Tier 3 changes. Reviews the diff in a fresh context against the approved plan and reports findings as CRITICAL or RECOMMENDED. Use before preflight on major changes.
tools: Read, Grep, Glob, Bash, WebFetch, WebSearch
---
<!-- Generated by the CLAUDE.md customization prompt. -->
You review a completed code change in a fresh context. You are given the approved plan and the list of changed files. Do not edit files. Do not run any command listed under "Forbidden commands" in the project's CLAUDE.md. Use `git diff` and `git status` to see the change.

Check, in order:
1. Plan compliance: does it do what was approved, and nothing more?
2. Correctness: logic errors, edge cases, error handling that matters
3. Simplicity and scope: code beyond what the task needs; changed lines that do not trace to the request
4. Consistency: matches existing patterns and style
5. Security: new vulnerabilities, secrets, unsafe input handling
6. API currency: external library use matches the official documentation for the version in use. Look it up yourself; do not accept the plan's "Docs consulted" line as proof. Flag a missing or implausible "Docs consulted" line when the change touches an external library.
7. Verification: build and tests were run and passed (look for evidence in what you were given; run the project's safe test command if you can)

Report only problems with a realistic way to happen: bugs, incorrect behaviour, security issues, data loss, deviations from the plan. For each finding, say in one sentence how it would happen in practice. Skip purely theoretical issues unless the fix is trivial. Do not report style preferences.

Label every finding:
- CRITICAL: build or test failure, incorrect behaviour in a realistic case, security issue, data loss, deviation from the plan, or an unrequested change
- RECOMMENDED: a real but lower-impact issue. The author fixes it if it is within the task's scope.

Problems in code this change did not touch go in a short "Outside this change" list at the end. They are never CRITICAL unless this change introduces or worsens them.

End with exactly one line: PASS (no CRITICAL findings) or NEEDS_REFINEMENT.
```

Rules for this step:
- If `.claude/agents/reviewer.md` already exists and does NOT contain the line "Generated by the CLAUDE.md", do not overwrite it. Leave it alone and tell the user in the report. (The template then falls back to a self-review.)
- If it exists and does contain that line, replace it with the version above.

### Step 9. Shell script line endings

Make sure `.gitattributes` at the repository root contains the line `*.sh text eol=lf`, so the hook and other shell scripts keep LF endings on Windows checkouts. Create the file if it does not exist; otherwise append the line only if it is missing. Do not change any other line in it. If `.claude/hooks/verify.sh` has CRLF endings on disk, convert it to LF.

### Step 10. DO NOT modify

- Task size tiers and sizing rules
- The four Principles
- The "Done means verified" section
- The "Review findings" section, including the 3-cycle stop and the rule against leftover lists
- The "Outside the repository", "Do not guess state", and "Research before changing" sections
- The commit message section: the format, the one-message rule, the checkmark-line rules, the closing line, and the rule that every change that modifies files ends with a commit message regardless of size
- The Universal (all repositories) forbidden block and the forbidden-commands enforcement wording
- The reviewer subagent text in Step 8

### Step 11. DO NOT expand or restructure anything

Only replace placeholders, inject repository-specific details, and remove HTML instruction comments (`<!-- ... -->`) once their sections are filled in. Do not add new sections, examples, or explanations.

Keep the result short. The finished CLAUDE.md should land at roughly 110 to 160 lines plus any carried-over Project-specific rules. If the carried-over rules run past about 40 lines, keep only those Claude could not infer from the code. In the report, suggest moving area-specific groups into `.claude/rules/*.md` files with a `paths:` frontmatter list (they load only when Claude works on matching files) or into nested CLAUDE.md files. Do not create those files unasked.

### Step 12. Self-check

Run these checks in the repository root against the new CLAUDE.md (use `.claude/CLAUDE.md` instead if that is where the file lives). If any line prints FAIL, fix the file and re-run until all pass, or explain in the report why a FAIL is legitimate (for example a carried-over rule that really does contain a bracketed uppercase word).

```bash
f=CLAUDE.md; fail=0
chk() { if eval "$2"; then echo "ok   $1"; else echo "FAIL $1"; fail=1; fi; }
chk "no old phase or workflow headings" '! grep -qE "^## PHASE|Standard Workflow|ABSOLUTE RULES|Orchestrating Agent" "$f"'
chk "has How to work section" 'grep -q "^## How to work" "$f"'
chk "has Universal git block" 'grep -q "Universal (all repositories)" "$f"'
chk "has commit message section" 'grep -q "Commit message (final step of every change)" "$f"'
chk "has research section" 'grep -q "^### Research before changing" "$f" && grep -q "Docs consulted" "$f"'
chk "has review findings section" 'grep -q "^### Review findings" "$f" && grep -q "After 3 fix cycles" "$f"'
chk "has outside-the-repository section" 'grep -q "^### Outside the repository" "$f"'
chk "no old review rules" '! grep -qiE "Fix CRITICAL findings only|After 2 failed fix cycles" "$f"'
chk "no unfilled placeholders" '! grep -qE "\[[A-Z][A-Z_0-9]*\]" "$f"'
chk "no instruction comments left" '! grep -q "<!--" "$f"'
chk "gitattributes keeps .sh as LF" 'grep -qx "\*\.sh text eol=lf" .gitattributes'
echo "lines: $(wc -l < "$f")"
```

### Step 13. Output

1. Write the finalized `CLAUDE.md` to the repository root.
2. Confirm that `.claude/settings.json` was written (or merged), and, where created, `.claude/hooks/verify.sh`, `.claude/agents/reviewer.md`, and `.gitattributes`. Give their paths.
3. Print a short report:
   - **Mode:** FRESH or UPDATE
   - **Length:** line count of the new CLAUDE.md (and of the old one, in UPDATE mode)
   - **Commands chosen:** build, test, preflight, and verify commands
   - **Forbidden commands:** what was added and why
   - **UPDATE mode only:**
     - Repo-specific rules carried over
     - Old content dropped (generic workflow content that the template replaces, including old review rules and pointers to other repositories' files)
     - Content dropped as derivable
     - Stale commands that were not carried over
     - Conflicts between old rules and the template that the user should decide on (including any stricter old research rule)
     - Whether a `CLAUDE.md.bak` was created
   - **Settings merge:** anything removed from `allow` because of a conflict with `deny`
   - **Hook and reviewer:** created, updated, skipped, or left alone, and why. Give the guard you chose, whether it passes on this machine, and the `bash -n` result.
   - **Research tooling:** whether a Context7 MCP server is configured (and whether it was added to `allow`). If not, say that the Research rule will fall back to official docs through web search and fetch, and that adding Context7 would make lookups faster and more reliable.
   - **Needs your input:** placeholders you could not fill because the information is not in the repo
   - **Self-check:** the output of Step 12
   - **Commit message:** one message covering every file this prompt created or changed, in the template's format, with only the check lines that actually ran
   - **Suggested next step:** run `/doctor` to check the new file for content that can be derived from the code

Do not run `git add`, `git commit`, `git push`, or `git stash`. Leave the files for the user to review with `git diff` and commit themselves.

---

Here is the template:

[PASTE CLAUDE.md TEMPLATE HERE]
