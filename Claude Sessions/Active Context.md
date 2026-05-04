---
tags: [active-context]
---

# Active Context

## Current Session
- **Note:** [[SL-1146 Metering Minutes/2026-05-04]]
- **Branch:** feat/SL-1146-metering-minutes
- **Project:** silia
- **Last updated:** 2026-05-04

## What's Happening
- Implemented usage tracking: AudioStream records call duration → BillingUsage DynamoDB table
- BillingUsage model created with all fields (chatbotId, accountId, sessionId, agentType, unit, durationSeconds, minutes, reportedToStripe)
- DynamoDB table with ChatbotIdTimestampIndex GSI for batch job queries
- IAM configured for both AudioStream (PutItem) and BillingGeneralRole (full CRUD for batch job)

## Current State
- **Uncommitted:** 13 modified + 2 new files, 303 insertions
- **Done:** Usage recording on call end, DynamoDB infra, IAM, env vars
- **Remaining:** Batch job Lambda, minutesUsed transformer, retry/alerting

## How to Continue
1. Implement batch job Lambda (reportUsageToStripe handler)
2. Wire minutesUsed in transformers.ts:161 (replace placeholder)
3. Add retry/alerting to batch job
4. Commit all changes
5. Test end-to-end