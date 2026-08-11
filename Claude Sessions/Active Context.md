# Active Context

## Sesión actual: SL-1579 Manual escalation — cadena de bugs (2026-08-07/11)
Nota: [[Claude Sessions/silia/SL-1579-manual-escalation-debug/2026-08-07]] · Plan: [[projects/Escalacion Forzada Admin - Analisis y Plan]]

**Rama `feat/SL-1579-force-escalation` — varios commits de fix, mayormente SIN push.**

Feature "Manual escalation" (SL-1579) ya en develop; esta sesión = arreglar los bugs al probar en dev.

### Fixes hechos (todos necesitan `sam deploy` real para surtir efecto)
1. `cf9aeb512` — resolver conversación por channelId (no solo id); 404 si no resuelve.
2. `8370e1782` — infra: `CHAT_TABLE` env + IAM sobre tabla `Chat` en RoleEscalationsLifecycle.
3. reactivación de conversaciones cerradas + reconciliación de memory-TTL (guard expectedStatus / closedAt / expirationTime).
4. `df325e34e` — idempotencia por la tabla de Escalaciones (anti-duplicado), no por el flag de la conversación.
5. `60434378a` — FE: ocultar botón "Manual escalation" si ya está escalada.

### ⚠️ Bloqueo actual
- El usuario subió CÓDIGO manual, pero eso NO aplica env/IAM (vienen del template). Agregué `CHAT_TABLE` por CLI; el **IAM sobre `Chat` NO** pude (mi SSO no tiene `iam:PutRolePolicy`). → sigue 404 hasta un `sam deploy` real.
- **Acción pendiente #1: push + sam deploy del Assistant a dev.**

### Comportamiento "si ya está escalada"
200 take-control (no 409, no duplica): mismo op → no-op; sin asignar → assign; otro op → handoff. Devuelve el ticket existente.

### Notas
- Regresión aparente `ConversationCoordinator.isHumanMode.AGE18` = pre-existente de develop (no mío).
- Perfil dev: `silia-engineer-operator-817389378997` (SSO, expira). Stack `dev-app-silia-com`.
- `GH_PACKAGES_TOKEN=dummy npx jest` para tests.

### Contexto previo
- SL-1528 Feature 6 KPI aggregations. [[Claude Sessions/silia/SL-1528-kpi-aggregations/2026-08-05]]
- SL-1526 Feature 5 KPI prefs (en develop).