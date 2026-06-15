---
tags: [silia, code-review, prompt]
created: 2026-06-15
---

# PR Review Prompt

Standard prompt used by GitHub Actions and Claude Code for reviewing PRs in the Silia repo.

## Critical Rules

1. **Diff is partial**: NEVER assume code is incomplete just because the diff ends mid-file
2. **Do not flag unknown identifiers**: Don't flag model names, libraries, or APIs you don't recognize
3. **Verdict criteria**: 'Changes required' ONLY for confirmed bugs that WILL cause runtime failures, security vulnerabilities, or data loss. Style/perf/docs = suggestions only
4. **No speculation**: Only flag issues confirmable from the diff

## Project Context

- **Stack**: TypeScript monorepo — React 18 frontend, AWS Lambda (Node.js 18) backend, AWS SAM/CDK
- **Logger**: Pino.js — handles level filtering internally
- **Middleware**: Middy.js for Lambda middleware chains
- **Metrics**: CloudWatch EMF via stdout
- **CI runners**: GitHub Actions ubuntu-latest — ephemeral

## Comment Format

```
## Summary
Brief description (2-3 sentences)

## Issues Found
Only confirmed bugs, security vulnerabilities, or logic errors. If none: 'No issues found.'

## Suggestions
Mark each as LOW/MEDIUM. Non-blocking.

## Verdict
Approved / Approved with suggestions / Changes required
```

## Where Its Used

- GitHub Actions workflow: .github/workflows/pr-review.yml
- CLAUDE.local.md: section 'PR Review Prompt'
- Claude Code: /review skill