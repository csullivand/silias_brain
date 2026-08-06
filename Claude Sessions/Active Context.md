# Active Context

## Sesión actual: Análisis escalaciones + plan escalación forzada (2026-08-06)
Nota: [[Claude Sessions/silia/escalacion-forzada-admin/2026-08-06]] · Doc: [[projects/Escalacion Forzada Admin - Analisis y Plan]]

**Tarea:** analizar escalaciones del inbox y planear "escalación forzada". Análisis + plan **hechos** (doc con diagramas mermaid). Falta implementar.

### Hallazgos clave
- escalación == ticket (`EscalationRecord`, tabla `Escalations`). Hoy solo escala el LLM; no hay creación manual.
- El gate de horario vive SOLO en `EscalationEvaluatorService`; `createEscalation` NO valida horario → la feature = llamar `createEscalation` directo detrás de un endpoint nuevo.

### Decisiones (REVISADAS)
- **Permiso para TODOS** (no admin): gate con `agent.inbox.view`. Sin permiso nuevo.
- **NO se auto-asigna**: el ticket entra `requires_takecontrol` SIN asignar; se asigna MANUAL desde tickets con el endpoint `assign` existente.
- Bloquea la conversación (bot deja de responder). 409 si ya hay escalación activa (+FE oculta botón).
- **+2 columnas de auditoría** (se quedan): `isForced` (bool) y `forcedBy`/`forcedByName` (quién la forzó, del token; solo auditoría, no asigna).

### Plan
1. Modelo/interface: `isForced`, `forcedBy`, `forcedByName` (retrocompatible).
2. Servicio `forceEscalate()` = guard duplicado + createEscalation(forzada, bloquea). SIN assign.
3. Handler `Force/index.ts` (permiso `agent.inbox.view`, forcedBy del token, 409).
4. Sin permiso CASL nuevo (reusa `agent.inbox.view`).
5. Infra: ruta `POST /{chatbotId}/escalations/force` + build entry.
6. Tests servicio + handler.

### Pendiente
- `userName` para `forcedByName` (claims vs lookup) — no bloqueante.
- Nº de ticket para rama; decidir implementar ahora.

### Contexto previo (KPI cards)
- SL-1528 Feature 6 KPI aggregations — commit `943b6902b`, sin push. [[Claude Sessions/silia/SL-1528-kpi-aggregations/2026-08-05]]
- SL-1526 Feature 5 — mergeado a develop. Duda: max cards 6 vs 4. BUG: develop no despliega KpiPrefs (faltan build entries).
- Nota: `GH_PACKAGES_TOKEN=dummy npx jest`.