# CLAUDE.md Customization Prompt

I am providing you with a CLAUDE.md Orchestrator Agent template.
Your task: Customize it for THIS repository without changing its logic, structure, rules, or workflow.
You must also create a `.claude/settings.json` for this project.

## Instructions

1. Analyze the repository structure.

2. Detect and fill in:
   - Project name
   - Primary languages
   - Frameworks
   - Build system
   - Test commands (safe ones only — see step 4)
   - Package manager
   - Architecture patterns
   - Key directories
   - Special constraints

3. Fill in the Resource Constraints section — project-structural facts only:
   - Do NOT detect or record host RAM, free disk, or CPU counts. These vary per developer
     machine and would be wrong on any other device the project is developed on.
   - Instead, identify constraints that are permanently true about the codebase itself:
     - Does the build system or workspace layout make certain commands unsafe on any machine?
       (e.g. bare `cargo build` fails because the workspace has no default member)
     - Are any targets or features OS-specific or require optional system libraries?
       (e.g. a crate that only compiles on Linux due to libpam-dev)
     - Does building require a specific toolchain or target not universally available?
       (e.g. WASM target requires wasm-pack; cross-compilation requires a sysroot)
     - What is the CI environment type? (GitHub Actions free tier, self-hosted, Docker-based, local only)
     - Are there any commands with large disk side-effects?
       (e.g. Docker multi-stage builds produce multi-GB layers)

4. Fill in the FORBIDDEN COMMANDS section:
   - For each build or test command you would normally add, assess whether it is structurally unsafe:
     a. Does it evaluate all project targets in parallel in a way that is known to cause failures
        (e.g. evaluates 30 NixOS configurations simultaneously)?
     b. Does it have destructive or irreversible side effects?
     c. Is it broken by the project's workspace or build layout on any machine?
   - If YES to any of the above: add it to FORBIDDEN COMMANDS with a clear reason,
     and find a safe alternative for the Test Commands section.
   - If NO to all: it is safe to add to Test Commands.
   - If no forbidden commands are identified: write "None identified" and remove the placeholder lines.

5. Add project-specific build validation steps to the PHASE 3 section
   using ONLY the safe commands identified in step 4.

6. Create `.claude/settings.json` at the repository root with the following content:
   - Always include `"effortLevel": "medium"` as the baseline
   - Under `"permissions"`:
     - `"allow"`: list the safe build, test, git, and read commands identified in steps 4 and 5
       that Claude will run repeatedly during normal workflow execution — so the extension
       does not prompt for approval on every tool call
     - `"deny"`: list every command from FORBIDDEN COMMANDS so the extension enforces
       the same prohibition at the tool-call level, as a second layer of protection
       beyond the CLAUDE.md instruction
   - Use the pattern `"Bash(<command-prefix>:*)"` for each entry
   - Example structure:
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
       }
     }
     ```
   - `rm -rf` must always be in `deny` regardless of project type
   - `git add`, `git commit`, `git push`, and `git stash` must always be in `deny` — git
     operations that stage, commit, push, or stash are the user's responsibility, never Claude's
   - Do not include commands in `allow` that appear in `deny`
   - After writing the file, confirm its path: `.claude/settings.json`

7. DO NOT modify:
   - Workflow phases
   - Phase execution logic
   - Absolute rules
   - Engineering Principles (all four)
   - Refinement loop logic
   - Escalation rules (2-cycle limit + report to user)
   - Scoring system
   - Conventional Commits format in Phase 7
   - FORBIDDEN COMMANDS enforcement logic
   - Context7 dependency policy
   - "Verify Before Asserting" section

8. DO NOT simplify or restructure anything.

9. Only replace placeholders and inject project-specific technical details.

10. Output the finalized `CLAUDE.md` ready to save at the repository root,
    then confirm that `.claude/settings.json` has been written.

---

Here is the template:

[PASTE CLAUDE.md TEMPLATE HERE]
