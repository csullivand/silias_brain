# Move Objects Between Folders

## Overview
Objects (agents, tables) can be moved between folders via API. The folder counters are automatically updated.

---

## Agents (Chatbots)

### Create Agent in Folder
```bash
POST /chatbot
{
  "name": "My Agent",
  "folderId": "<folder-id>",  # or null/omit for ROOT
  ...
}
```

### Move Agent to Folder
```bash
PUT /chatbot/{chatbotId}
{
  "folderId": "<new-folder-id>"  # or null for ROOT
}
```

**Handler:** `Assistant/application/Put/index.ts`
- Validates folderId via Zod schema
- Captures `oldFolderId` before update
- Calls `FolderModel.safeIncrementItemCount` to adjust counters

---

## Tables (DynamicTables)

### Create Table in Folder
```bash
POST /tables
{
  "name": "My Table",
  "folderId": "<folder-id>",  # or null/omit for ROOT
  "chatbotId": "<optional>"
}
```

### Move Table to Folder
```bash
PATCH /tables/{tableId}
{
  "folderId": "<new-folder-id>"  # or null for ROOT
}
```

**Handler:** `DynamicTables/application/Tables/patch/updateTable.ts`
- Added `folderId` to `UpdateTableDTO` (2026-06-25)
- Captures `oldFolderId` before update
- Calls `FolderModel.safeIncrementItemCount` to adjust counters

---

## Counter Updates

When an object is moved:
1. Old folder: `-1` on the relevant counter (agents or tables)
2. New folder: `+1` on the relevant counter

Uses `FolderModel.safeIncrementItemCount(folderId, accountId, field, delta)`:
- Validates folder ownership (accountId match)
- Atomic DynamoDB counter update
- Fire-and-forget (errors logged, don't block operation)

---

## Files

| Object | Create Handler | Move Handler |
|--------|---------------|--------------|
| Agent | `Assistant/application/Post/index.ts` | `Assistant/application/Put/index.ts` |
| Table | `DynamicTables/application/Tables/post/createTable.ts` | `DynamicTables/application/Tables/patch/updateTable.ts` |
| Folder | `Folders/application/Folders/post/createFolder.ts` | `Folders/application/Folders/patch/moveFolder.ts` |

---

Tags: #silia #folders #api #dynamictables #agents