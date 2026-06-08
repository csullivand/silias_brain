---
tags: [claude-session, active-context]
updated: 2026-06-04
---

# Active Context

## Current Session
- **Project:** Silia
- **Topic:** CASL Authorization POC
- **Session notes:** [[Claude Sessions/silia/casl-authorization-poc/2026-06-03]]
- **Branch:** feat/SL-1318-filter-bar-configuration-model

## What Was Done
- CASL POC fully validated: 21 unit tests + live DynamoDB
- Postman collection created with 7 requests
- Local auth setup (LocalAuthFunction + env.json)
- Presentation doc complete (14 sections)

## Current State
- POC complete, not committed
- Local API changes are for dev only (don't commit auth changes)

## How to Continue
1. Start local API: cd POC/infrastructure && yarn POC
2. Test with Postman using admin/user JWT tokens
3. Commit POC code (exclude local auth changes)
4. Roll out to all endpoints