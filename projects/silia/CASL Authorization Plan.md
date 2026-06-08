# CASL Authorization — Story 0 + Story 0.1 Implementation Plan

## Context
Two backend stories for Feature 0 of the PRD. Story 0 builds the authorization system. Story 0.1 applies it to existing endpoints. Both are backend-only — no frontend changes.

**Key constraint**: All permissions, roles, and their mappings live in the **database** (existing Role, Permission, Resource tables). No hardcoded permission constants in code.

**Phase 1 scope**: "All features will work for backoffice users specifically. For users consuming directly via API, all criteria will be enabled in Phase 2."

---

# PART 1: Story 0 — Sistema de autorización base

**Goal**: Build the `can(user, permission_id, resource)` module using existing DB tables, create 2 new endpoints, seed ~57 permissions in DB.

## Existing DB tables we use (no new tables needed)

```
Role table:       { id, name, status }                    — 6 roles already seeded
Permission table: { id, roleId, resourceId, permission }  — GSI: roleId-resourceId-index
Resource table:   { id, name, description, status }       — 57 frontend routes already seeded
```

Role IDs in DEV:
```
superuser:  8bb70118-0d07-43c8-9040-296c687941ee
admin:      81ceb00d-2438-4e00-9259-09517b617a66
supervisor: 066ad765-7975-4efc-964c-30d5f3c187d1
operator:   d69138f2-971f-401b-ac08-408bf92df486
seller:     a78117b7-f787-4255-9003-575a6a18d45f
user:       1e3b4c36-2999-4683-b1c6-a312b6b02d41
```

---

## Step 1.1: Expand Permission model for atomic permissions

**File**: `Roles/domain/models/Permission.model.ts`

### What to change:

1. Add `IsOptional` to imports:
```ts
import {IsEnum, IsInt, IsNotEmpty, IsOptional, IsString, IsUUID} from 'class-validator';
```

2. Expand `PermissionLevel` enum:
```ts
export enum PermissionLevel {
    // Read
    VIEW = 'view',
    // Write
    CREATE = 'create',
    UPDATE = 'update',
    DELETE = 'delete',
    EDIT = 'edit',
    // Execute
    EXECUTE = 'execute',
    // Share
    SHARE = 'share',
    // Full access
    MANAGE = 'manage',
    // No access
    NONE = 'none',
}
```

3. Add `PermissionCategory` enum:
```ts
export enum PermissionCategory {
    READ = 'read',
    WRITE = 'write',
    EXECUTE = 'execute',
    SHARE = 'share',
    MANAGE = 'manage',
}
```

4. Add fields to `PermissionI` interface:
```ts
export interface PermissionI {
    id: string;
    resourceId: string;
    roleId: string;
    permission: PermissionLevel;
    description?: string;       // NEW
    category?: PermissionCategory; // NEW
    createdAt?: number;
    updatedAt?: number;
    status?: string;
}
```

5. Add class properties with decorators:
```ts
@IsOptional()
@IsString()
description: string;

@IsOptional()
@IsEnum(PermissionCategory)
category: PermissionCategory;
```

6. Add to constructor:
```ts
this.description = data.description || '';
this.category = data.category || PermissionCategory.READ;
```

7. Add to `save()` Item:
```ts
Item: {
    ...existing fields,
    description: this.description,
    category: this.category,
}
```

No DB migration needed — DynamoDB is schemaless. New fields are optional.

---

## Step 1.2: Update Role enum

**File**: `shared/config/roles.ts`

### What to change:

```ts
export enum Role {
    SUPERUSER  = 'superuser',
    ADMIN      = 'admin',
    SUPERVISOR = 'supervisor',
    OPERATOR   = 'operator',
    USER       = 'user',
}

export const ELEVATED_ROLES: Role[] = [Role.SUPERUSER, Role.ADMIN];

export const DEFAULT_ROLES: Role[] = [Role.SUPERUSER, Role.ADMIN, Role.SUPERVISOR, Role.OPERATOR];
```

---

## Step 1.3: Create CASL ability builder (permissions.ts)

**File**: `shared/config/permissions.ts` (CREATE or UPDATE)

This file uses the existing 3 DB tables to build CASL abilities.

### Full implementation:

```ts
import { AbilityBuilder, createMongoAbility, MongoAbility } from '@casl/ability';
import { Role } from '@shared/config/roles';
import { RoleModel } from '../../Roles/domain/models/Role.model';
import { PermissionModel, PermissionLevel } from '../../Roles/domain/models/Permission.model';
import { ResourceModel } from '../../Roles/domain/models/Resource.model';

export type Action = 'view' | 'create' | 'update' | 'delete' | 'edit' | 'execute' | 'share' | 'manage';
export type Subject = string;
export type AppAbility = MongoAbility<[Action, Subject]>;

const CACHE_TTL_MS = 30_000;
const cache = new Map<string, { ability: AppAbility; expiry: number }>();
const roleIdCache = new Map<string, { id: string; expiry: number }>();

async function resolveRoleId(roleName: string): Promise<string | null> {
    const cached = roleIdCache.get(roleName);
    if (cached && Date.now() < cached.expiry) return cached.id;

    const roleInstance = new RoleModel({});
    const allRoles = await roleInstance.findAll();
    const role = allRoles.find((r: any) => r.name === roleName);
    if (!role) return null;

    roleIdCache.set(roleName, { id: role.id, expiry: Date.now() + CACHE_TTL_MS });
    return role.id;
}

export async function defineAbilitiesForRole(role: string): Promise<AppAbility> {
    // Superuser bypass — no DB call
    if (role === Role.SUPERUSER) {
        const { can, build } = new AbilityBuilder<AppAbility>(createMongoAbility);
        can('manage', 'all');
        return build();
    }

    // Check cache
    const cached = cache.get(role);
    if (cached && Date.now() < cached.expiry) return cached.ability;

    const { can, build } = new AbilityBuilder<AppAbility>(createMongoAbility);

    // Step 1: Role name → UUID
    const roleId = await resolveRoleId(role);

    if (roleId) {
        // Step 2: Get all permission rows for this role
        const permissions = await PermissionModel.findAllByRoleId(roleId);

        if (permissions && permissions.length > 0) {
            // Step 3: Get resource names
            const resourceIds = [...new Set(permissions.map(p => p.resourceId))];
            const resourceMap = await ResourceModel.findByIds(resourceIds);

            // Step 4: Build CASL rules
            for (const perm of permissions) {
                if (perm.permission === PermissionLevel.NONE) continue;
                const resource = resourceMap.get(perm.resourceId);
                if (!resource) continue;

                const subject = resource.name.charAt(0).toUpperCase() + resource.name.slice(1);
                can(perm.permission as Action, subject);
            }
        }
    }

    const ability = build();
    cache.set(role, { ability, expiry: Date.now() + CACHE_TTL_MS });
    return ability;
}

export function clearAbilityCache(role?: string): void {
    if (role) { cache.delete(role); roleIdCache.delete(role); }
    else { cache.clear(); roleIdCache.clear(); }
}
```

### How the 3-table lookup works:

```
JWT token contains: role = 'admin'
    │
    ▼
Step 1: Role Table
    Scan for name = 'admin' → id = '81ceb00d-...'
    │
    ▼
Step 2: Permission Table (GSI: roleId-resourceId-index)
    Query: roleId = '81ceb00d-...'
    Result:
        { resourceId: 'uuid-1', permission: 'view' }
        { resourceId: 'uuid-1', permission: 'create' }
        { resourceId: 'uuid-2', permission: 'view' }
    │
    ▼
Step 3: Resource Table
    Batch get by IDs:
        'uuid-1' → name: 'Agent'
        'uuid-2' → name: 'Billing'
    │
    ▼
Step 4: Build CASL Ability
    can('view', 'Agent')
    can('create', 'Agent')
    can('view', 'Billing')
```

---

## Step 1.4: Create requirePermission middleware

**File**: `shared/middleware/requirePermission.ts` (CREATE or UPDATE)

### Full implementation:

```ts
import middy from '@middy/core';
import { APIGatewayProxyEvent, APIGatewayProxyResult } from 'aws-lambda';
import { StatusCodes } from 'http-status-codes';
import { Utils } from '@shared/clases/Utils.class';
import { getCallerRole, getCallerContext } from '@shared/middleware/requireRole';
import { defineAbilitiesForRole, Action, Subject } from '@shared/config/permissions';

function buildForbiddenResponse(message: string): APIGatewayProxyResult {
    return {
        headers: Utils.getCorsHeaders(),
        statusCode: StatusCodes.FORBIDDEN,
        body: JSON.stringify({ error: 'permission_denied', message }),
    };
}

// ─── Middy middleware ─────────────────────────────────────────────────────────

export function requirePermission(action?: Action, subject?: Subject): middy.MiddlewareObj<APIGatewayProxyEvent> {
    return {
        before: async (request) => {
            const role = getCallerRole(request.event);
            const resolvedAction = action || process.env.CASL_ACTION as Action
                || { GET:'view', POST:'create', PUT:'update', PATCH:'update', DELETE:'delete' }[request.event.httpMethod]
                || 'view';
            const resolvedSubject = subject || process.env.CASL_SUBJECT || 'unknown';

            if (!role) {
                throw Object.assign(new Error('Forbidden'), { statusCode: 403 });
            }

            const callerCtx = getCallerContext(request.event);
            const ability = await defineAbilitiesForRole(role);
            const allowed = ability.can(resolvedAction as Action, resolvedSubject);

            // Audit logging
            if (!allowed) {
                console.log(JSON.stringify({
                    type: 'permission_denied',
                    userId: callerCtx.userId,
                    role,
                    action: resolvedAction,
                    subject: resolvedSubject,
                    endpoint: request.event.path,
                    method: request.event.httpMethod,
                    timestamp: Date.now(),
                }));

                const { context: ctx } = request;
                (ctx as any).earlyResponse = buildForbiddenResponse(
                    'You do not have permission to perform this action'
                );
                throw Object.assign(new Error('Forbidden'), { statusCode: 403 });
            }

            // Log mutations for superuser bypass
            if (role === 'superuser') {
                console.log(JSON.stringify({
                    type: 'super_admin_bypass',
                    userId: callerCtx.userId,
                    action: resolvedAction,
                    subject: resolvedSubject,
                    endpoint: request.event.path,
                    timestamp: Date.now(),
                }));
            }

            // Log allowed mutations
            if (['POST', 'PUT', 'PATCH', 'DELETE'].includes(request.event.httpMethod)) {
                console.log(JSON.stringify({
                    type: 'permission_granted_mutation',
                    userId: callerCtx.userId,
                    role,
                    action: resolvedAction,
                    subject: resolvedSubject,
                    endpoint: request.event.path,
                    timestamp: Date.now(),
                }));
            }

            (request.context as any).performedBy = {
                userId: callerCtx.userId,
                role: callerCtx.role,
                accountId: callerCtx.accountId,
            };
        },
    };
}

// ─── Callback utility ─────────────────────────────────────────────────────────

export async function assertPermission(
    event: APIGatewayProxyEvent,
    action: Action,
    subject: Subject
): Promise<APIGatewayProxyResult | null> {
    const role = getCallerRole(event);
    if (!role) return buildForbiddenResponse('You do not have permission to perform this action');

    const ability = await defineAbilitiesForRole(role);
    if (!ability.can(action, subject)) {
        const callerCtx = getCallerContext(event);
        console.log(JSON.stringify({
            type: 'permission_denied',
            userId: callerCtx.userId,
            role, action, subject,
            endpoint: event.path,
            timestamp: Date.now(),
        }));
        return buildForbiddenResponse('You do not have permission to perform this action');
    }
    return null;
}
```

### Auto-detection:
- **Action**: explicit param → `CASL_ACTION` env var → HTTP method mapping → `'view'`
- **Subject**: explicit param → `CASL_SUBJECT` env var → `'unknown'`

### Usage:
```ts
// Explicit (for non-standard mappings):
middy(handler).use(requirePermission('execute', 'Workflow'))

// Auto-derived (most handlers):
middy(handler).use(requirePermission())
// POST /agents → resolves to can('create', 'Agent') via HTTP method + CASL_SUBJECT env var
```

---

## Step 1.5: Seed backend resources in DEV Resource table

### Resources to add (alongside existing frontend routes):

```bash
PROFILE="silia-engineer-operator-817389378997"
TABLE="dev-app-silia-com-resource"
NOW=$(date +%s)

# Agent
aws dynamodb put-item --table-name $TABLE --profile $PROFILE --region us-east-1 \
  --item "{\"id\":{\"S\":\"res-agent\"},\"name\":{\"S\":\"Agent\"},\"description\":{\"S\":\"AI chatbot agents\"},\"status\":{\"S\":\"active\"},\"createdAt\":{\"N\":\"$NOW\"},\"updatedAt\":{\"N\":\"$NOW\"}}"

# Folder
aws dynamodb put-item --table-name $TABLE --profile $PROFILE --region us-east-1 \
  --item "{\"id\":{\"S\":\"res-folder\"},\"name\":{\"S\":\"Folder\"},\"description\":{\"S\":\"Workspace folders\"},\"status\":{\"S\":\"active\"},\"createdAt\":{\"N\":\"$NOW\"},\"updatedAt\":{\"N\":\"$NOW\"}}"

# Table
aws dynamodb put-item --table-name $TABLE --profile $PROFILE --region us-east-1 \
  --item "{\"id\":{\"S\":\"res-table\"},\"name\":{\"S\":\"Table\"},\"description\":{\"S\":\"Dynamic tables\"},\"status\":{\"S\":\"active\"},\"createdAt\":{\"N\":\"$NOW\"},\"updatedAt\":{\"N\":\"$NOW\"}}"

# Billing
aws dynamodb put-item --table-name $TABLE --profile $PROFILE --region us-east-1 \
  --item "{\"id\":{\"S\":\"res-billing\"},\"name\":{\"S\":\"Billing\"},\"description\":{\"S\":\"Billing and payments\"},\"status\":{\"S\":\"active\"},\"createdAt\":{\"N\":\"$NOW\"},\"updatedAt\":{\"N\":\"$NOW\"}}"

# Workflow
aws dynamodb put-item --table-name $TABLE --profile $PROFILE --region us-east-1 \
  --item "{\"id\":{\"S\":\"res-workflow\"},\"name\":{\"S\":\"Workflow\"},\"description\":{\"S\":\"N8n workflow automation\"},\"status\":{\"S\":\"active\"},\"createdAt\":{\"N\":\"$NOW\"},\"updatedAt\":{\"N\":\"$NOW\"}}"

# OrganizationUsers
aws dynamodb put-item --table-name $TABLE --profile $PROFILE --region us-east-1 \
  --item "{\"id\":{\"S\":\"res-org-users\"},\"name\":{\"S\":\"OrganizationUsers\"},\"description\":{\"S\":\"User management\"},\"status\":{\"S\":\"active\"},\"createdAt\":{\"N\":\"$NOW\"},\"updatedAt\":{\"N\":\"$NOW\"}}"

# OrganizationTeams
aws dynamodb put-item --table-name $TABLE --profile $PROFILE --region us-east-1 \
  --item "{\"id\":{\"S\":\"res-org-teams\"},\"name\":{\"S\":\"OrganizationTeams\"},\"description\":{\"S\":\"Team management\"},\"status\":{\"S\":\"active\"},\"createdAt\":{\"N\":\"$NOW\"},\"updatedAt\":{\"N\":\"$NOW\"}}"

# OrganizationRoles
aws dynamodb put-item --table-name $TABLE --profile $PROFILE --region us-east-1 \
  --item "{\"id\":{\"S\":\"res-org-roles\"},\"name\":{\"S\":\"OrganizationRoles\"},\"description\":{\"S\":\"Role management\"},\"status\":{\"S\":\"active\"},\"createdAt\":{\"N\":\"$NOW\"},\"updatedAt\":{\"N\":\"$NOW\"}}"

# AccountSettings
aws dynamodb put-item --table-name $TABLE --profile $PROFILE --region us-east-1 \
  --item "{\"id\":{\"S\":\"res-account-settings\"},\"name\":{\"S\":\"AccountSettings\"},\"description\":{\"S\":\"Account configuration\"},\"status\":{\"S\":\"active\"},\"createdAt\":{\"N\":\"$NOW\"},\"updatedAt\":{\"N\":\"$NOW\"}}"

# AccountChannels
aws dynamodb put-item --table-name $TABLE --profile $PROFILE --region us-east-1 \
  --item "{\"id\":{\"S\":\"res-account-channels\"},\"name\":{\"S\":\"AccountChannels\"},\"description\":{\"S\":\"Deployment channels\"},\"status\":{\"S\":\"active\"},\"createdAt\":{\"N\":\"$NOW\"},\"updatedAt\":{\"N\":\"$NOW\"}}"

# AccountIntegrations
aws dynamodb put-item --table-name $TABLE --profile $PROFILE --region us-east-1 \
  --item "{\"id\":{\"S\":\"res-account-integrations\"},\"name\":{\"S\":\"AccountIntegrations\"},\"description\":{\"S\":\"Third-party integrations\"},\"status\":{\"S\":\"active\"},\"createdAt\":{\"N\":\"$NOW\"},\"updatedAt\":{\"N\":\"$NOW\"}}"

# AuditLog
aws dynamodb put-item --table-name $TABLE --profile $PROFILE --region us-east-1 \
  --item "{\"id\":{\"S\":\"res-audit-log\"},\"name\":{\"S\":\"AuditLog\"},\"description\":{\"S\":\"Audit log viewer\"},\"status\":{\"S\":\"active\"},\"createdAt\":{\"N\":\"$NOW\"},\"updatedAt\":{\"N\":\"$NOW\"}}"

# Training
aws dynamodb put-item --table-name $TABLE --profile $PROFILE --region us-east-1 \
  --item "{\"id\":{\"S\":\"res-training\"},\"name\":{\"S\":\"Training\"},\"description\":{\"S\":\"AI model training\"},\"status\":{\"S\":\"active\"},\"createdAt\":{\"N\":\"$NOW\"},\"updatedAt\":{\"N\":\"$NOW\"}}"

# Integration
aws dynamodb put-item --table-name $TABLE --profile $PROFILE --region us-east-1 \
  --item "{\"id\":{\"S\":\"res-integration\"},\"name\":{\"S\":\"Integration\"},\"description\":{\"S\":\"Third-party service connections\"},\"status\":{\"S\":\"active\"},\"createdAt\":{\"N\":\"$NOW\"},\"updatedAt\":{\"N\":\"$NOW\"}}"

# Conversation
aws dynamodb put-item --table-name $TABLE --profile $PROFILE --region us-east-1 \
  --item "{\"id\":{\"S\":\"res-conversation\"},\"name\":{\"S\":\"Conversation\"},\"description\":{\"S\":\"Chat conversations\"},\"status\":{\"S\":\"active\"},\"createdAt\":{\"N\":\"$NOW\"},\"updatedAt\":{\"N\":\"$NOW\"}}"

# Metrics
aws dynamodb put-item --table-name $TABLE --profile $PROFILE --region us-east-1 \
  --item "{\"id\":{\"S\":\"res-metrics\"},\"name\":{\"S\":\"Metrics\"},\"description\":{\"S\":\"Analytics and metrics\"},\"status\":{\"S\":\"active\"},\"createdAt\":{\"N\":\"$NOW\"},\"updatedAt\":{\"N\":\"$NOW\"}}"

# RTA
aws dynamodb put-item --table-name $TABLE --profile $PROFILE --region us-east-1 \
  --item "{\"id\":{\"S\":\"res-rta\"},\"name\":{\"S\":\"RTA\"},\"description\":{\"S\":\"Real-time agent assistance\"},\"status\":{\"S\":\"active\"},\"createdAt\":{\"N\":\"$NOW\"},\"updatedAt\":{\"N\":\"$NOW\"}}"
```

---

## Step 1.6: Seed permission rows for all 4 default roles

### PRD Permission Matrix — what to seed

Each row below = one `put-item` to the Permission table.

**Admin permissions** (roleId: `81ceb00d-2438-4e00-9259-09517b617a66`):

| Resource | Actions |
|---|---|
| OrganizationUsers | view, create, edit, delete, deactivate, import |
| OrganizationTeams | view, create, edit, delete |
| OrganizationRoles | view, create, edit, delete, assign |
| Billing | view, edit |
| AccountSettings | view, edit |
| AccountChannels | view, edit |
| AccountIntegrations | view, edit |
| AuditLog | view |
| Folder | view, create, edit, move, delete, share_access |
| Agent | view, create, edit_metadata, delete, share_access |
| Agent (kb) | view, edit |
| Agent (workflows) | view, edit, delete, execute |
| Agent (inbox) | view, operate, reassign |
| Agent (metrics) | view_own, view_global |
| Agent (settings) | view, edit |
| Agent (business_rules) | view, edit |
| Agent (audiences) | view, edit |
| Agent (billing) | view |
| Agent (tables) | link |
| Table | view, create, edit_structure, edit_data, delete, share_access, export |
| Table (agents) | link |

**Supervisor permissions** (roleId: `066ad765-7975-4efc-964c-30d5f3c187d1`):

| Resource | Actions |
|---|---|
| Folder | view |
| Agent | view, edit_metadata |
| Agent (kb) | view, edit |
| Agent (workflows) | view, edit, execute |
| Agent (inbox) | view, operate, reassign |
| Agent (metrics) | view_own, view_global |
| Agent (settings) | view, edit |
| Agent (business_rules) | view, edit |
| Table | view, edit_data, export |

**Operator permissions** (roleId: `d69138f2-971f-401b-ac08-408bf92df486`):

| Resource | Actions |
|---|---|
| Folder | view |
| Agent | view |
| Agent (kb) | view |
| Agent (workflows) | view, execute |
| Agent (inbox) | view, operate |
| Agent (metrics) | view_own |
| Table | view |

**Superuser**: No rows needed — bypasses everything in code with `can('manage', 'all')`.

### Seed script

**File**: `scripts/seed-permissions.ts` (NEW)

This script reads the PRD matrix and inserts all permission rows. Run once per environment.

Example for one row:
```bash
aws dynamodb put-item --table-name dev-app-silia-com-permission --profile $PROFILE --region us-east-1 \
  --item "{\"id\":{\"S\":\"perm-admin-agent-view\"},\"roleId\":{\"S\":\"81ceb00d-2438-4e00-9259-09517b617a66\"},\"resourceId\":{\"S\":\"res-agent\"},\"permission\":{\"S\":\"view\"},\"description\":{\"S\":\"Ver agentes\"},\"category\":{\"S\":\"read\"},\"status\":{\"S\":\"active\"},\"createdAt\":{\"N\":\"$NOW\"},\"updatedAt\":{\"N\":\"$NOW\"}}"
```

---

## Step 1.7: Build GET /me/permissions endpoint

**File**: `Roles/application/permission/getMyPermissions.ts` (NEW)

```ts
import { APIGatewayProxyEvent, APIGatewayProxyResult } from 'aws-lambda';
import middy from '@middy/core';
import httpCors from '@middy/http-cors';
import httpErrorHandler from '@middy/http-error-handler';
import { StatusCodes } from 'http-status-codes';
import { getCallerRole, getCallerContext } from '@shared/middleware/requireRole';
import { RoleModel } from '../../domain/models/Role.model';
import { PermissionModel } from '../../domain/models/Permission.model';
import { ResourceModel } from '../../domain/models/Resource.model';
import { Role } from '@shared/config/roles';

const getMyPermissionsHandler = async (event: APIGatewayProxyEvent): Promise<APIGatewayProxyResult> => {
    const role = getCallerRole(event);
    const callerCtx = getCallerContext(event);

    if (!role) {
        return { statusCode: StatusCodes.FORBIDDEN, body: JSON.stringify({ message: 'No role' }) };
    }

    // Superuser gets all
    if (role === Role.SUPERUSER) {
        return {
            statusCode: StatusCodes.OK,
            body: JSON.stringify({
                role,
                permissions: [{ action: 'manage', subject: 'all' }],
            }),
        };
    }

    // Resolve role → permissions → resources
    const roleInstance = new RoleModel({});
    const allRoles = await roleInstance.findAll();
    const roleRecord = allRoles.find((r: any) => r.name === role);

    if (!roleRecord) {
        return { statusCode: StatusCodes.OK, body: JSON.stringify({ role, permissions: [] }) };
    }

    const permissions = await PermissionModel.findAllByRoleId(roleRecord.id);
    if (!permissions || permissions.length === 0) {
        return { statusCode: StatusCodes.OK, body: JSON.stringify({ role, permissions: [] }) };
    }

    const resourceIds = [...new Set(permissions.map(p => p.resourceId))];
    const resourceMap = await ResourceModel.findByIds(resourceIds);

    const result = permissions
        .filter(p => p.permission !== 'none' && resourceMap.has(p.resourceId))
        .map(p => ({
            action: p.permission,
            subject: resourceMap.get(p.resourceId)!.name,
            description: p.description || '',
            category: p.category || 'read',
        }));

    return {
        statusCode: StatusCodes.OK,
        body: JSON.stringify({ role, permissions: result }),
    };
};

exports.handler = middy(getMyPermissionsHandler)
    .use(httpCors({ credentials: true, origin: '*' }))
    .use(httpErrorHandler());
```

**SAM template** (`Roles/infrastructure/aws.template.yml`): Add function + API route:
```yaml
GetMyPermissionsFunction:
  Type: AWS::Serverless::Function
  Properties:
    Role: !Ref Role
    Runtime: nodejs18.x
    Handler: getMyPermissionsMin.handler
    CodeUri: ../dist/application/permission
    FunctionName: !Sub ${StackName}-get-my-permissions
    Events:
      api:
        Type: Api
        Properties:
          Path: /me/permissions
          Method: get
          RestApiId: !Ref Api
          Auth:
            Authorizer: LambdaTokenAuth
```

---

## Step 1.8: Build GET /permissions_catalog endpoint

**File**: `Roles/application/permission/getCatalog.ts` (NEW)

Returns all resources and their available actions (the full catalog for frontend tooltips and Phase 2 roles matrix).

```ts
const getCatalogHandler = async (event: APIGatewayProxyEvent): Promise<APIGatewayProxyResult> => {
    // Get all resources
    const resourceInstance = new ResourceModel({});
    const allResources = await resourceInstance.findAll();

    // Get all permissions (all roles) to determine which actions exist per resource
    // Or define a static list of possible actions per resource type

    const catalog = allResources.map((r: any) => ({
        id: r.id,
        name: r.name,
        description: r.description,
    }));

    return {
        statusCode: StatusCodes.OK,
        body: JSON.stringify({ catalog }),
    };
};
```

**SAM**: Add to `Roles/infrastructure/aws.template.yml`.

---

## Step 1.9: Protect default roles from modification

**File**: `Roles/application/role/update.ts`
**File**: `Roles/application/role/delete.ts`

Add at the top of the handler, after loading the role:
```ts
const DEFAULT_ROLE_NAMES = ['superuser', 'admin', 'supervisor', 'operator'];
if (DEFAULT_ROLE_NAMES.includes(role.name)) {
    return callback(null, {
        statusCode: 403,
        body: JSON.stringify({ error: 'default_role_immutable', message: 'Default roles cannot be modified' })
    });
}
```

---

## Step 1.10: Unit tests

**File**: `shared/config/__tests__/permissions.test.ts`

Mock the 3 DB models. Test:
- Superuser can manage all (no DB call)
- Admin with specific permissions (from mock data matching PRD matrix)
- Supervisor with limited permissions
- Operator with minimal permissions
- Unknown role = no permissions
- Caching works (second call skips DB)
- Cache invalidation forces fresh call

---

## Story 0 Deliverables Checklist

- [ ] 1.1: Permission.model.ts — expanded enum + description/category
- [ ] 1.2: shared/config/roles.ts — expanded Role enum
- [ ] 1.3: shared/config/permissions.ts — CASL ability builder
- [ ] 1.4: shared/middleware/requirePermission.ts — middleware with audit logging
- [ ] 1.5: Seed backend resources in DEV Resource table (~17 resources)
- [ ] 1.6: Seed permission rows in DEV Permission table (~57 per role × 3 roles)
- [ ] 1.7: GET /me/permissions endpoint
- [ ] 1.8: GET /permissions_catalog endpoint
- [ ] 1.9: Protect default roles from modification
- [ ] 1.10: Unit tests

**Estimated effort: 5 days**

---

# PART 2: Story 0.1 — Aplicar middleware a endpoints existentes

**Goal**: Apply `requirePermission` middleware to existing endpoints. Start in permissive mode, switch to strict after monitoring.

**Scope**: Not all ~280 Lambdas. Focus on endpoints with LambdaTokenAuth (~95). Split by domain.

---

## Step 2.1: Feature flag for permissive/strict mode

**File**: `shared/middleware/requirePermission.ts`

Add to the middleware `before` handler:
```ts
const STRICT_MODE = process.env.PERMISSION_STRICT_MODE === 'true';

if (!allowed) {
    if (STRICT_MODE) {
        // Return 403
    } else {
        // Permissive: log warning, allow through
        console.warn(JSON.stringify({
            type: 'permission_warning_permissive',
            userId, action, subject, endpoint: event.path, method: event.httpMethod,
            timestamp: Date.now(),
        }));
        // Don't throw — let request through
    }
}
```

Add `PERMISSION_STRICT_MODE: 'false'` to all SAM template `Globals.Function.Environment.Variables`.

---

## Step 2.2: Add CASL_SUBJECT env var to each module SAM template

For each module, add to Globals:
```yaml
Globals:
  Function:
    Environment:
      Variables:
        CASL_SUBJECT: "Agent"       # Module default
        PERMISSION_STRICT_MODE: 'false'
```

| Module | CASL_SUBJECT |
|---|---|
| Accounts | Account |
| User | OrganizationUsers |
| Roles | OrganizationRoles |
| Assistant | Agent |
| DynamicTables | Table |
| Messages | Conversation |
| RTA | RTA |
| Teams | OrganizationTeams |
| Flows | Workflow |
| Billing | Billing |
| Training | Training |
| Integrations | Integration |
| Workflows | Workflow |
| Metrics | Metrics |

Per-function overrides where the default doesn't fit (e.g., Assistant's billing endpoints use `Billing` not `Agent`).

---

## Step 2.3: Apply middleware — Batch 1: Accounts + User + Roles (32 functions)

**Effort**: 2 days

Already have LambdaTokenAuth. For each handler:

1. Convert callback handlers to Middy pattern (return `APIGatewayProxyResult`)
2. Add `.use(requirePermission())` to the middy chain
3. Set `CASL_SUBJECT` in SAM template
4. Override `CASL_ACTION` per function where HTTP method doesn't match

**Files to modify**:
```
Accounts/application/post/index.ts
Accounts/application/get/getAll.ts
Accounts/application/get/getById.ts
Accounts/application/put/index.ts
Accounts/application/patch/updateAccountStatus.ts
Accounts/application/delete/index.ts
Accounts/application/post/addNote.ts
Accounts/application/put/updateNote.ts
Accounts/application/delete/deleteNote.ts
Accounts/application/usageFees/getUsageFees.ts

User/application/post/index.ts
User/application/get/*.ts
User/application/put/*.ts
User/application/delete/*.ts

Roles/application/role/*.ts
Roles/application/permission/*.ts
```

---

## Step 2.4: Apply middleware — Batch 2: Assistant (54 functions)

**Effort**: 3 days

Largest module. Most handlers already have LambdaTokenAuth.

**Permission mapping**:
```
POST /chatbot           → CASL_ACTION=create, CASL_SUBJECT=Agent
GET /chatbot            → CASL_ACTION=view, CASL_SUBJECT=Agent
GET /chatbot/{id}       → CASL_ACTION=view, CASL_SUBJECT=Agent
PUT /chatbot/{id}       → CASL_ACTION=edit_metadata, CASL_SUBJECT=Agent
PATCH /chatbot/{id}     → CASL_ACTION=edit_metadata, CASL_SUBJECT=Agent
DELETE /chatbot/{id}    → CASL_ACTION=delete, CASL_SUBJECT=Agent

POST /knowledge         → CASL_SUBJECT=Agent, CASL_ACTION=edit (agent.kb.edit)
GET /knowledge          → CASL_SUBJECT=Agent, CASL_ACTION=view (agent.kb.view)

GET /inbox              → CASL_SUBJECT=Agent, CASL_ACTION=view (agent.inbox.view)
POST /inbox/message     → CASL_SUBJECT=Agent, CASL_ACTION=operate (agent.inbox.operate)

GET /metrics            → CASL_SUBJECT=Agent, CASL_ACTION=view (agent.metrics.view_global)
```

---

## Step 2.5: Apply middleware — Batch 3: DynamicTables (36 functions)

**Effort**: 2 days

```
POST /tables           → create, Table
GET /tables            → view, Table
PATCH /tables/{id}     → edit_structure, Table
DELETE /tables/{id}    → delete, Table
POST /rows             → edit_data, Table
PUT /rows/{id}         → edit_data, Table
DELETE /rows/{id}      → edit_data, Table
GET /rows              → view, Table
GET /export            → export, Table
```

---

## Step 2.6: Apply middleware — Batch 4: Messages + RTA + Teams + Flows (50 functions)

**Effort**: 3 days

---

## Step 2.7: Apply middleware — Batch 5: Billing + Integrations + Workflows (48 functions)

**Effort**: 3 days

These DON'T have LambdaTokenAuth yet. Steps:
1. Add `Auth` section to SAM API definition
2. Build Auth middleware for the module
3. Add `requirePermission()` to handlers

**Skip**: Stripe webhook, Freshdesk webhook, SQS-triggered, WebSocket handlers.

---

## Step 2.8: Document exempt endpoints

Endpoints that skip permission check:
```
POST /auth/login
POST /auth/refresh
POST /auth/logout
POST /webhooks/stripe
POST /webhooks/freshdesk
WebSocket connect/disconnect/subscribe
Healthcheck endpoints
SQS/scheduled triggers
```

---

## Step 2.9: Multi-resource validation pattern

For endpoints touching multiple resources (e.g., move between folders):
```ts
// Inside handler, after middleware primary check:
import { defineAbilitiesForRole } from '@shared/config/permissions';
const ability = await defineAbilitiesForRole(role);
const canSource = ability.can('edit', 'Folder'); // + check ownership
const canDest = ability.can('edit', 'Folder');   // + check ownership
if (!canSource || !canDest) return 403;
```

---

## Step 2.10: Async job permission capture

For SQS-enqueued jobs, persist permission at enqueue time:
```ts
// When enqueuing:
await sqs.sendMessage({
    MessageBody: JSON.stringify({
        ...payload,
        _permissionContext: { role, action, subject, evaluatedAt: Date.now() }
    })
});
// Worker uses _permissionContext, does NOT re-evaluate
```

---

## Story 0.1 Deliverables Checklist

- [ ] 2.1: Feature flag PERMISSION_STRICT_MODE in all SAM templates
- [ ] 2.2: CASL_SUBJECT env var in each module SAM template
- [ ] 2.3: Batch 1 — Accounts + User + Roles (32 functions)
- [ ] 2.4: Batch 2 — Assistant (54 functions)
- [ ] 2.5: Batch 3 — DynamicTables (36 functions)
- [ ] 2.6: Batch 4 — Messages + RTA + Teams + Flows (50 functions)
- [ ] 2.7: Batch 5 — Billing + Integrations + Workflows (48 functions)
- [ ] 2.8: Exempt endpoints documented
- [ ] 2.9: Multi-resource validation pattern documented
- [ ] 2.10: Async job permission capture pattern documented

**Estimated effort: 13 days**

---

## Total Effort

| Part | Work | Days |
|------|------|------|
| **Story 0** | Permission model + can() + 2 endpoints + seed DB + tests | 5 |
| **Story 0.1 Batch 1** | Accounts + User + Roles | 2 |
| **Story 0.1 Batch 2** | Assistant | 3 |
| **Story 0.1 Batch 3** | DynamicTables | 2 |
| **Story 0.1 Batch 4** | Messages + RTA + Teams + Flows | 3 |
| **Story 0.1 Batch 5** | Billing + Integrations + Workflows | 3 |
| **Total** | | **~18 days** |

---

## Verification

1. **Unit tests**: Parametric role × permission matrix (mock DB)
2. **sam local invoke**: Test admin vs operator on key endpoints
3. **Postman**: 3 scenarios per batch (admin pass, operator denied, superuser bypass)
4. **CloudWatch**: Monitor `permission_warning_permissive` logs in permissive mode
5. **Strict mode flip**: After zero warnings for 1 week, change `PERMISSION_STRICT_MODE` to `'true'`