# Codebase Maintenance Skill

Use this skill when modifying, debugging, or reviewing the Yggdrasil codebase.

## Goal

Make small, safe, reviewable engineering changes.

## Workflow

1. Inspect the repository.
2. Identify relevant files.
3. Explain the intended change.
4. Make the smallest safe patch.
5. Run formatter, linter, and tests when available.
6. Summarize the change.
7. Mention risks and follow-up tasks.

## Rules

Do not rewrite large parts of the system unless explicitly asked.

Do not change unrelated files.

Do not hide failing tests.

Do not claim production deployment succeeded unless verified.

Do not expose secrets.

## Before Running Risky Commands

Explain the command and risk before running commands that:

- delete files
- modify databases
- restart services
- change firewall rules
- remove Docker volumes
- force push Git history
- alter production data

## Preferred Output

For code changes, summarize:

```text
Changed:
- file A: reason
- file B: reason

Validation:
- command run
- result

Risks:
- known risk or none

Next:
- recommended follow-up
