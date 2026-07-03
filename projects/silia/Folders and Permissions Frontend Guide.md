# Folders & Permissions Frontend Integration Guide

This guide covers the backend features for the Folders module, DynamicTables refactor, and Access Grants system. Use this to implement the frontend UI.

---

## Table of Contents

1. [[#Feature Overview]]
2. [[#API Endpoints]]
3. [[#Data Models]]
4. [[#Permission Inheritance]]
5. [[#UI Implementation Guide]]
6. [[#Error Handling]]

---

## Feature Overview

| Feature | Description | Endpoint |
|---------|-------------|----------|
| Folders CRUD | Create, rename, move, delete folders | `POST/GET/PATCH/DELETE /folders/folders` |
| Unified Listing | List folders + agents + tables together | `GET /folders/folders?parentFolderId=` |
| Visibility Filter | Show only objects user has access to | `GET /folders/folders?visible_to=me` |
| Access Grants | Grant users/teams permissions on objects | `POST/GET/PATCH/DELETE /access/grants` |
| Effective Permissions | Check user's permissions on an object | `GET /access/effective` |
| Tables in Folders | Tables can be placed in folders | `PATCH /dynamic-tables/tables/{id}` |

---

## API Endpoints

### 1. Folders CRUD

#### Create Folder

```http
POST /folders/folders
Content-Type: application/json
Authorization: Bearer {token}

{
  "name": "Projects",
  "parentFolderId": null
}
```

**Response (201):**
```json
{
  "id": "folder-abc123",
  "name": "Projects",
  "parentFolderId": null,
  "path": "/folder-abc123",
  "depth": 1,
  "itemCounts": { "folders": 0, "agents": 0, "tables": 0 },
  "createdAt": 1719849600000,
  "updatedAt": 1719849600000,
  "createdBy": "user-xyz"
}
```

**Notes:**
- `parentFolderId: null` → creates at ROOT level
- `parentFolderId: "folder-id"` → creates inside that folder
- Max depth: 5 levels

#### Get Folder by ID

```http
GET /folders/folders/{folderId}
```

#### Rename Folder

```http
PATCH /folders/folders/{folderId}
{
  "name": "Renamed Folder"
}
```

#### Move Folder

```http
PATCH /folders/folders/{folderId}/move
{
  "parentFolderId": "folder-newparent"
}
```

#### Delete Folder

```http
DELETE /folders/folders/{folderId}
```

---

### 2. Unified Folder Listing

#### List ROOT Contents

```http
GET /folders/folders?parentFolderId=ROOT
```

**Response:**
```json
{
  "data": [
    {
      "id": "folder-abc",
      "type": "folder",
      "name": "Projects",
      "parentFolderId": null,
      "itemCounts": { "folders": 2, "agents": 5, "tables": 3 }
    },
    {
      "id": "agent-xyz",
      "type": "agent",
      "name": "Support Bot",
      "folderId": null,
      "status": "active"
    },
    {
      "id": "table-123",
      "type": "table",
      "name": "Customer Data",
      "folderId": null,
      "rowCount": 1542
    }
  ],
  "count": 3,
  "nextPageToken": null
}
```

#### List Subfolder Contents

```http
GET /folders/folders?parentFolderId={folderId}
```

---

### 3. Visibility Filter (`visible_to=me`)

Filter results to only show objects the user has explicit permissions on.

```http
GET /folders/folders?parentFolderId=ROOT&visible_to=me
```

**Behavior:**

| User Role | Behavior |
|-----------|----------|
| Admin / Superuser / Superadmin | Returns ALL objects (bypass) |
| Supervisor / Operator | Returns only objects with grants |
| No grants | Returns empty array |

**What makes an object visible:**

1. **Direct grant** on the object itself
2. **Team grant** on the object (user is team member)
3. **Folder inheritance** - grant on parent/ancestor folder (up to 5 levels)

---

### 4. Access Grants CRUD

#### Create Grant (User → Folder)

```http
POST /access/grants
{
  "subjectType": "user",
  "subjectId": "user-abc123",
  "objectType": "folder",
  "objectId": "folder-xyz",
  "roleId": "role-editor"
}
```

#### Create Grant (Team → Folder)

```http
POST /access/grants
{
  "subjectType": "team",
  "subjectId": "team-sales",
  "objectType": "folder",
  "objectId": "folder-xyz",
  "roleId": "role-admin"
}
```

#### List Grants

```http
GET /access/grants?object_id={folderId}&object_type=folder
```

#### Update Grant Role

```http
PATCH /access/grants/{grantId}
{
  "roleId": "role-viewer"
}
```

#### Delete Grant

```http
DELETE /access/grants/{grantId}
```

---

### 5. Effective Permissions

```http
GET /access/effective?object_id={objectId}&object_type=folder
```

**Response:**
```json
{
  "objectId": "folder-xyz",
  "objectType": "folder",
  "effectiveRole": "role-editor",
  "permissions": ["folder.view", "folder.edit", "agent.view"],
  "grantSources": [
    { "type": "direct", "grantId": "grant-123", "roleId": "role-editor" },
    { "type": "inherited", "fromObjectId": "folder-parent", "roleId": "role-viewer" }
  ]
}
```

---

### 6. Move Objects Between Folders

#### Move Agent

```http
PUT /chatbot/{chatbotId}
{ "folderId": "folder-xyz" }
```

#### Move Table

```http
PATCH /dynamic-tables/tables/{tableId}
{ "folderId": "folder-xyz" }
```

---

## Data Models

### FolderContentItem

```typescript
interface FolderContentItem {
  id: string;
  type: 'folder' | 'agent' | 'table';
  name: string;
  accountId: string;
  createdAt: number;
  updatedAt: number;

  // Folder-specific
  parentFolderId?: string | null;
  path?: string;
  depth?: number;
  itemCounts?: { folders: number; agents: number; tables: number };

  // Agent-specific
  folderId?: string | null;
  status?: 'active' | 'inactive' | 'draft';
  agentType?: 'conversational' | 'voice' | 'workflow';

  // Table-specific
  rowCount?: number;
}
```

### Available Roles

| Role ID | Name | Description |
|---------|------|-------------|
| `role-admin` | Admin | Full access |
| `role-supervisor` | Supervisor | View + manage team |
| `role-operator` | Operator | View + limited edit |
| `role-viewer` | Viewer | Read-only |
| `role-editor` | Editor | View + edit content |

---

## Permission Inheritance

### How Inheritance Works

```
/Projects (folder) ← User has "editor" grant here
├── /Team-A (subfolder) ← User can access (inherited)
│   ├── agent-1 ← User can access (inherited)
│   └── table-1 ← User can access (inherited)
├── /Team-B (subfolder) ← User can access (inherited)
│   └── agent-2 ← User can access (inherited)
└── agent-3 ← User can access (inherited)
```

### Inheritance Rules

1. **Depth limit**: 5 levels max
2. **Most permissive wins**: If user has `viewer` on folder and `editor` direct on agent, they get `editor`
3. **Team grants expand**: All team members inherit team's grants
4. **No upward inheritance**: Grant on `/Team-A` does NOT give access to `/Projects`

---

## UI Implementation Guide

### 1. Folder Tree View

- Lazy-load subfolder contents on expand
- Show `itemCounts` as badges
- Drag-and-drop to move objects
- Right-click context menu (rename, move, delete, share)

### 2. Visibility Toggle

```
[ ] Show only my items (visible_to=me)
```

- Hide toggle for Admin/Superuser/Superadmin

### 3. Share Dialog

- Load grants: `GET /access/grants?object_id={id}&object_type=folder`
- Add user/team with role selector
- Show inheritance note for folders

### 4. Permission Indicator Icons

| Icon | Meaning |
|------|---------|
| 🔑 | Direct grant |
| 👥 | Team grant |
| 📁 | Inherited from folder |

### 5. Empty States

**No items:** Show create buttons
**No visible items:** Show "Contact admin for access"

---

## Error Handling

| Status | Message | UI Action |
|--------|---------|-----------|
| 400 | Cannot move folder into itself | Error toast |
| 400 | Max depth exceeded | Error toast |
| 403 | Insufficient permissions | "Access Denied" |
| 404 | Folder not found | Redirect to parent |
| 409 | Folder not empty | Confirm bulk delete |

---

## Quick Reference

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/folders/folders?parentFolderId=ROOT` | List ROOT |
| GET | `/folders/folders?visible_to=me` | Filtered list |
| POST | `/folders/folders` | Create folder |
| PATCH | `/folders/folders/{id}` | Rename |
| PATCH | `/folders/folders/{id}/move` | Move |
| DELETE | `/folders/folders/{id}` | Delete |
| GET | `/access/grants` | List grants |
| POST | `/access/grants` | Create grant |
| GET | `/access/effective` | Check permissions |

---

Tags: #silia #frontend #api #folders #permissions #documentation