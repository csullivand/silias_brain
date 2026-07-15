---
tags: [claude-session, active-context]
updated: 2026-07-15
---

# Active Context

## Current Session
- **Project:** Silia
- **Topic:** [Feature 5] access_grants polimórfico (SL-1282)
- **Branch:** fix/SL-1282-access-polimorficos (basada en develop, SIN COMMITEAR)

## Estado (2026-07-15)
Feature 5 (modelo AccessGrant + endpoints /access) — **desarrollo COMPLETO**, sin commitear. Detalle completo en [[Claude Sessions/silia/feature-5-access-grants/2026-07-15]].

### Implementado (todos los AC + extras)
- S1 idempotencia, S2 validar role, S5 permiso share (object-level), S6 audit log (tabla dedicada), S7 cascada user/team, S8 cascada objeto, S9 invalidar miembros team, S3 herencia en listGrants (is_inherited+origen), S4 bloquear revocar heredado (403), S11 soft delete (reemplaza hard delete), S12 admin_objects (era stub).
- 8 tests nuevos, verdes.

### Decisiones de producto
- D1: share solo admin (supervisor/operator = futuro).
- D2: PERMISSION_STRICT_MODE apagado (rollout coordinado aparte) → hoy el check share advierte pero no bloquea a no-admins.

### Diferido
- S10 dashboard (no existe la entidad backend).
- Cleanup strict-tsc (~30 chatbotId + colisiones de tests; el repo no compila con tsc estricto, build real lo ignora).

## PENDIENTE
1. Commit + push + PR a develop.
2. Correr seed-access-resources.ts (crea folder/agent/table.share).
3. Desplegar 6 módulos: Access (tabla AccessAuditLog + soft delete), Teams, User, Folders, DynamicTables, Assistant.
4. NO verificado: suite completa, dev real, review.
5. 2 peticiones fuera de scope que el usuario mencionó (aún sin detallar).

## Cómo continuar
- Correr suite completa de módulos tocados → confirmar.
- Commit → push → PR a develop.
- Seed + deploy + prueba dev.

## Related
- [[Claude Sessions/silia/feature-5-access-grants/2026-07-15]]
- [[Claude Sessions/silia/resource-counts-teams-users/2026-07-14]]