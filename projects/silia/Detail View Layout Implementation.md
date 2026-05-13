---
tags: [silia, dynamic-tables, detail-view, layout, implementation]
created: 2026-05-13
---

# Detail View Layout — Implementation Notes

## What Was Built

Backend sync between column visibility and detail view layout. When a column's visibility changes, the layout auto-updates.

## Storage Decision

**Campo en tabla existente** (not new table) — `detailViewConfig` stored as JSON field on `DynamicTablesTablesTable`.

## API

### GET /tables/{tableId}
Returns `detailViewConfig: null` when no layout configured (explicit null, not omitted).

### PATCH /tables/{tableId}
Accepts `{ detailViewConfig: { headerPropertyId, leftColumn, rightColumn, actionsState } }`.

**Validations:**
- Header type restricted: no select, multi-select, document, formula, button → `errorCode: INVALID_HEADER_TYPE`
- Buttons only in actionsState → `errorCode: BUTTON_OUTSIDE_ACTIONS`
- No duplicates across zones → `errorCode: DUPLICATE_PROPERTY`
- Non-existent columnIds: **silently filtered** (not 400) — handles concurrent column deletion gracefully
- `detailViewConfig: null` clears layout
- `updatedBy` set from authorizer userId

**Concurrency:** Last-write-wins (DynamoDB default, no ConditionExpression).

## Visibility Sync (auto on column PATCH)

When `visibility` changes on a column:
- `hide` or `table` → remove from all layout sections (header, left, right, actions)
- `detail_view` or `table_view` → add to layout if not present:
  - button type → actionsState with enabled: true
  - other types → end of leftColumn
- Uses TransactWriteItems for atomicity (column + table in one transaction)

## Column Visibility Enum

```typescript
type ColumnVisibility = 'hide' | 'table' | 'detail_view' | 'table_view';
```

Backwards compatible: `isVisible: true` → `table_view`, `isVisible: false` → `hide`.

## Design Decision: Non-existent Columns

Ticket said HTTP 400 for non-existent propertyIds. Frontend team asked to ignore silently. **Decision: ignore silently.** Reason: in multi-user scenarios, user A deletes a column while user B has layout open → B's PATCH has stale ID → 400 would block B from saving. Filtering is more robust and aligns with last-write-wins concurrency model.

## Key Files

| File | Purpose |
|------|---------|
| `DynamicTables/domain/types/tables.types.ts` | ColumnVisibility, DetailViewConfig types |
| `DynamicTables/domain/models/DynamicColumn.model.ts` | visibility field + toItem() |
| `DynamicTables/domain/models/DynamicTable.model.ts` | detailViewConfig field + toItem() |
| `DynamicTables/application/Columns/patch/layoutSync.ts` | Pure function: sync layout on visibility change |
| `DynamicTables/application/Columns/patch/updateColumn.ts` | TransactWriteItems atomic sync |
| `DynamicTables/application/Tables/patch/updateTable.ts` | PATCH with validations |
| `DynamicTables/application/Tables/get/getTableById.ts` | Returns detailViewConfig |
| `app/src/features/data-views/utils/viewConfig.ts` | Frontend ViewConfig (localStorage → API migration pending) |

## actionsState Default Behavior

Button columns not included in actionsState array are treated as `enabled: true` by default (frontend logic). Backend only stores explicitly configured buttons.