---
tags: [silia, automation, prompt]
created: 2026-06-15
---

# Auto-Improvement Prompt (Weekly CI)

Used by GitHub Actions claude-improve.yml workflow.

## Task
Analyze the codebase and ship ONE high-impact improvement.

## Focus Areas
contract-tests, error-handling, performance (configurable via workflow input)

## Critical Constraints
1. Contract tests MUST pass
2. Maintain backward compatibility
3. Follow existing code patterns
4. Max 3 files changed per run
5. Prioritize high-impact, low-risk
6. If nothing meaningful found, do nothing

## Workflow
1. Analyze: scan handlers, contract tests, TODO/FIXME comments
2. Apply the change(s)
3. Validate with contract tests
4. Branch: chore/SL-00-claude-weekly-{timestamp}
5. Commit with Conventional Commits
6. Push
7. Open PR against develop

## Where Its Used
- GitHub Actions: .github/workflows/claude-improve.yml
- CLAUDE.local.md: section 'Auto-Improvement Prompt'