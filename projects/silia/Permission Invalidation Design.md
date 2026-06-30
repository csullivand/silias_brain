---
tags: [project, silia, architecture, permissions]
created: 2026-06-30
---

# Permission Invalidation Design

## Overview

When objects (folders, agents, tables) move between folders, inherited permissions change. This service automatically invalidates affected users' permission cache so they see updated permissions immediately.

## Flow

```
Object Move (folder/agent/table)
        │
        ▼
PermissionInvalidationService.onObjectMoved()
        │
        ├── Same location? → No-op
        │
        ├── Calculate affected users
        │   ├── Old folder grants (losing access)
        │   ├── New folder grants (gaining access)
        │   └── Expand team grants → user IDs
        │
        ├── Build invalidation set
        │   └── users × objects (including descendants)
        │
        └── Execute
            ├── <100 entries → Sync (Redis DEL)
            └── >100 entries → SQS async
```

## Key Files

| File | Purpose |
|------|---------|
| `Access/domain/services/PermissionInvalidationService.ts` | Core service |
| `Access/application/Invalidation/processor/processInvalidation.ts` | SQS worker |
| `Access/domain/services/PermissionCache.ts` | Redis cache operations |

## Integration Points

| Module | File | Hook Location |
|--------|------|---------------|
| Folders | `moveFolder.ts` | After successful move + audit log |
| Assistant | `Put/index.ts` | After folderId change + counter update |
| DynamicTables | `updateTable.ts` | After folderId change (2 places) |

## Environment Variables

```yaml
PERMISSION_INVALIDATION_QUEUE_URL: !ImportValue -permission-invalidation-queue-url
ACCESS_GRANT_TABLE: !Sub -AccessGrant
TEAM_USERS_TABLE: !Sub -TeamUser
REDIS_URL_SSM_PARAM: !Sub "//redis/url"
```

## Design Decisions

### Fail-Open
Errors are logged but don't block moves. Cache entries expire via 5-minute TTL.

### Sync vs Async Threshold
- <100 invalidations: Sync Redis DEL (fast path)
- >100 invalidations: SQS FIFO queue (background processing)

### Team Expansion
Team grants are expanded to individual user IDs via `TeamUserModel.findByTeamId()`.

### Ancestor Lookup
Folder hierarchy checked up to 5 levels (matches `FolderService.MAX_NESTING_DEPTH`).

### Deduplication
`Set<string>` ensures each user is invalidated only once, even if they have multiple grants.

## SQS Configuration

```yaml
PermissionInvalidationQueue:
  Type: AWS::SQS::Queue
  Properties:
    QueueName: !Sub -permission-invalidation.fifo
    FifoQueue: true
    VisibilityTimeoutSeconds: 960
    KmsMasterKeyId: alias/aws/sqs
    RedrivePolicy:
      deadLetterTargetArn: !GetAtt PermissionInvalidationDLQ.Arn
      maxReceiveCount: 3
```

## Cache Key Pattern

```
permissions:{userId}:{accountId}:{objectType}:{objectId}
```

Example: `permissions:user-123:acme-corp:agent:bot-456`

## Related

- [[Dynamic Tables Refactor Plan]]
- [[Access Module Design]]
- [[Tables Refactor Deployment Runbook]]