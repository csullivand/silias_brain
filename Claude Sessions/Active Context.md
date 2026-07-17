# Active Context

## Sesión actual: Feature 0 / Fase 3 — cablear permisos atómicos a endpoints (2026-07-17)
Nota completa: [[Claude Sessions/silia/feature-0-permissions-wiring/2026-07-17]]

**Rama:** `feat/SL-0-permissions-catalog` (~9 commits de Feature 0, PR NO abierto).
**Estado:** Fase 3 COMPLETA en código. 172 handlers cableados en 14 módulos + matriz de inbox validada por PM. Modo permisivo (no bloquea aún). Falta DEPLOY + SEED + PR + (Fase 6) flip a strict mode.

### Qué se hizo
- **Fase 3 (cablear can() a endpoints):** assertPermission/assertObjectPermission (o CASL_PERMISSION env-var en Billing) en 14 módulos: Access/Folders/Roles/Teams/User/DynamicTables (previos) + Billing/Metrics/Messages/Workflows/Flows/Accounts/Training/Assistant (esta sesión). Piso assertRole preservado.
- **Doc** `docs/permission-mapping-for-pms.md`: propuesta tentativa → estado de cableado real por módulo (✅/🟡/🔴/⚪) + decisiones de PM.
- **Decisiones PM aplicadas:** ChunkMetadata (guard por método), Metrics /get+/aggregated (agent.metrics.view cuenta), PUT /conversation (→agent.inbox.reassign); audiences/Integrations/ExecuteWorkflow = solo assertRole; varios confirmados as-is.
- **Matriz inbox Operador VALIDADA:** view/take_control/send_reply/resolve_ticket ✅, reassign ❌. Flags _review removidos, test actualizado.

### Continuar por (en orden)
1. DEPLOY Lambdas actualizados.
2. Correr seeds EN ORDEN por env: seed-default-roles (B1) → seed-permissions-catalog (B2) → seed-role-permissions (B4). Sin B2+B4 el permisivo no tiene contra qué resolver.
3. Abrir PR feat/SL-0-permissions-catalog → develop.
4. Fase 6 (después): PERMISSION_STRICT_MODE=true (rollout coordinado) → recién ahí aplican los 403.

### Sin cablear a propósito (decidido)
- audiences (authorized-users, 9) · Integrations (25) · ExecuteWorkflow → solo assertRole (NOTE en código).
- Latentes sin endpoint: platform.accounts.switch, platform.implementers.*.

### Aprendizaje operativo (importante)
- Subagentes + `git stash` (repo-wide) + hooks auto-commit RTA = pérdida de trabajo sin commitear en módulos en carrera. Regla: **commitear por módulo apenas termina cada agente**; excluir submódulos (RTA/Agent/Voice) del commit.

### Sesión anterior (relacionada)
- [[Claude Sessions/silia/feature-10-roles/2026-07-16]] — SL-1289 Feature 10 (roles custom). Fase 3 depende del catálogo/matriz de Feature 0.