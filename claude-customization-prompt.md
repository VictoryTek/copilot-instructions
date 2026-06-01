# CLAUDE.md Customization Prompt

I am providing you with a CLAUDE.md Orchestrator Agent template.
Your task: Customize it for THIS repository without changing its logic, structure, rules, or workflow.

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

3. Fill in the Resource Constraints section:
   - Detect available RAM using the appropriate method for this OS:
     - Linux: read `/proc/meminfo`
     - macOS: run `sysctl hw.memsize`
     - Windows: run `wmic OS get TotalVisibleMemorySize`
     - If all three fail or the environment is unknown: ask the user before proceeding
   - Identify any commands that evaluate all targets in parallel and assess
     whether they are safe given the available RAM
   - Document any disk, network, or CI environment constraints

4. Fill in the FORBIDDEN COMMANDS section:
   - For each build or test command you would normally add, assess its resource cost:
     a. Does it evaluate all project targets in parallel?
     b. Could it exhaust available RAM or disk on this machine?
     c. Does it have destructive or irreversible side effects?
   - If YES to any of the above: add it to FORBIDDEN COMMANDS with a clear reason,
     and find a safe alternative for the Test Commands section.
   - If NO to all: it is safe to add to Test Commands.
   - If no forbidden commands are identified: write "None identified" and remove the placeholder lines.

5. Add project-specific build validation steps to the PHASE 3 section
   using ONLY the safe commands identified in step 4.

6. DO NOT modify:
   - Workflow phases
   - Phase execution logic
   - Absolute rules
   - Refinement loop logic
   - Escalation rules (2-cycle limit + report to user)
   - Scoring system
   - Conventional Commits format in Phase 7
   - FORBIDDEN COMMANDS enforcement logic
   - Context7 dependency policy

7. DO NOT simplify or restructure anything.

8. Only replace placeholders and inject project-specific technical details.

9. Output the finalized version ready to save as `CLAUDE.md` at the repository root.

---

Here is the template:

[PASTE CLAUDE.md TEMPLATE HERE]
