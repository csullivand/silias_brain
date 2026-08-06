# Active Context

## Sesión actual: Manual escalation (SL-1579 / Feature 1) — IMPLEMENTADO (2026-08-06)
Nota: [[Claude Sessions/silia/escalacion-forzada-admin/2026-08-06]] · Doc: [[projects/Escalacion Forzada Admin - Analisis y Plan]]

**Rama `feat/SL-1579-force-escalation` — feature implementada, tests verdes, SIN commitear.**

Endpoint `POST /{chatbotId}/escalations/force`: operador escala manual una conversación a Tickets y TOMA control, a cualquier hora.

### Comportamiento (idempotente)
- Sin ticket activo → crea (forzada, bloquea bot) + asigna al operador → `in_progress`.
- Ya tiene ticket → toma control del existente (assign si libre, handoff si de otro, no-op si del mismo). Sin 409.
- Permiso NUEVO `agent.inbox.force_escalate` (todos los roles del inbox). +2 columnas auditoría `isForced` + `forcedBy`/`forcedByName`.
- Identidad del token. Respuesta trae `status: in_progress` + `assignedTo`.

### Archivos tocados
- Assistant: IEscalationRecord.interface.ts, EscalationRecord.model.ts, EscalationRecordService.ts (forceEscalate), schemas/escalationRecord.schema.ts, Escalations/Lifecycle/Force/index.ts (nuevo), aws.template.yml + package.json (build entry).
- Permiso: scripts/migrations/permissions-catalog.json (+total 102→103) + role-permissions.json + app catalog.ts + tests bumped.
- Tests: 23/23 verdes (Force handler + forceEscalate servicio + permisos).

### Pendientes
- ⚠️ Sembrar el permiso en la BD (seed) ANTES de habilitar el endpoint (si no, 403 para todos salvo superuser).
- Commitear (excluye docs + submódulo Agent) + pr-review/adversarial.
- FE: implementar con la guía `docs/escalacion-forzada-frontend-integration.md`.

### Contexto previo (KPI cards)
- SL-1528 Feature 6 KPI aggregations — commit `943b6902b`. [[Claude Sessions/silia/SL-1528-kpi-aggregations/2026-08-05]]
- SL-1526 Feature 5 KPI prefs — en develop. Duda: max cards 6 vs 4. BUG: develop no despliega KpiPrefs (build entries).
- Nota: `GH_PACKAGES_TOKEN=dummy npx jest`.