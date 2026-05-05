---
tags: [silia, rta, architecture, reference]
created: 2026-05-05
---

# RTA Module Architecture

## Overview
The RTA (Real-Time Agent) module manages agent configurations (playbooks with knowledge, processes, metadata) and call sessions (tracking agent-user interactions). Follows Clean Architecture with strict separation: domain entities, repository interfaces, and infrastructure adapters.

## Entity: RtaConfig

```typescript
RtaConfig {
  id: string              // UUID, auto-generated
  agentId: string         // = chatbotId from Assistant module
  language: 'es' | 'en'
  topics: string[]
  isActive: boolean       // single active per agentId
  createdAt: number
  authorizedUsers: string[]  // user IDs with widget access
  metadata: { manual, version, date, scenario, language, source?, notes? }
  knowledge: { processes: [{ code, name, description, steps }], objections?: [{ triggers, nba }] }
  agentConfiguration?: { language?, active_skills[], additional_processes, code_overrides, custom }
}
```

## API Endpoints (18 Lambdas)

### Lifecycle
| Method | Endpoint | Function |
|--------|----------|----------|
| POST | /rta-configs | Create config (isActive=false) |
| GET | /rta-configs/{id} | Get by ID |
| GET | /rta-configs | List (paginated) |
| GET | /rta-configs/by-agent?agentId= | List by agentId |
| GET | /rta-configs/by-user?userId= | List by authorized user |
| GET | /rta-configs/active?agentId= or ?userId= | Get active config |
| POST | /rta-configs/{id}/activate | Activate (deactivates others for same agentId) |

### Configuration Updates
| Method | Endpoint | Function |
|--------|----------|----------|
| PATCH | /rta-configs/{id}/knowledge | Replace full knowledge |
| PUT | /rta-configs/{id}/knowledge/processes/{code} | Upsert single process |
| PATCH | /rta-configs/{id}/metadata | Replace metadata |
| PATCH | /rta-configs/{id}/language | Change language |
| PATCH | /rta-configs/{id}/topics | Replace topics |
| PATCH | /rta-configs/{id}/agent-configuration | Replace agentConfiguration |
| PUT | /rta-configs/{id}/agent-configuration/code-overrides/{code} | Upsert code override |
| PUT | /rta-configs/{id}/agent-configuration/additional-processes/{processId} | Upsert additional process |

### Authorized Users
| Method | Endpoint | Function |
|--------|----------|----------|
| PUT | /rta-configs/{id}/users | Replace full list |
| POST | /rta-configs/{id}/users/{userId} | Add user (idempotent) |
| DELETE | /rta-configs/{id}/users/{userId} | Remove user |

### Call Sessions
| Method | Endpoint | Function |
|--------|----------|----------|
| POST | /call-sessions | Record finished session |
| GET | /call-sessions?agentId= | List sessions |
| GET | /call-sessions/{id} | Get by ID |
| GET | /call-sessions/metrics?agentId= | Dashboard metrics |

## DynamoDB Tables
| Table | PK | GSI | Purpose |
|-------|-----|-----|---------|
| RtaConfigs | id | agentId-createdAt-index | Agent configurations |
| CallSessions | id | userId-startedAt, agentId-startedAt, rtaId-startedAt | Call history |
| MetricsSummary | pk + sk | agentId-sk-index | Aggregated daily/user metrics |

## Key Files
- Entity: `RTA/packages/rta-configs/domain/entities/RtaConfig.ts`
- Repository interface: `RTA/packages/rta-configs/domain/repositories/IRtaConfigRepository.ts`
- DynamoDB adapter: `RTA/packages/rta-configs/infrastructure/adapters/DynamoRtaConfigRepository.ts`
- SAM template: `RTA/apps/infrastructure/aws.template.yml`
- Postman: `RTA/docs/postman/RTA_Full.postman_collection.json`
- Lambda handlers: `RTA/apps/lambda/RtaConfigs/` and `RTA/apps/lambda/CallSessions/`

See also: [[RTA Billing Flow End-to-End]]