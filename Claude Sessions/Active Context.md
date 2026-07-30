# Active Context

## Sesión actual: SL-1545 role list enrichment + seed Implementador (2026-07-30)
Nota completa: [[Claude Sessions/silia/SL-1545-role-count/2026-07-30]]

**Rama:** `feat/SL-1545-role-count` — pusheada, 15 tests verdes, tsc limpio. HEAD merge `b213abae4`.

### Qué se hizo hoy
- **GET /permissions/role (list)** ahora enriquece cada rol con:
  - `userCount`: usuarios de la cuenta con ese rol (por NOMBRE via user.role, no borrados) → UserModel.countByRole.
  - `sections`: los MÓDULOS PRD donde el rol tiene ≥1 permiso (level ≠ NONE).
- **sections = 6 módulos PRD**: `platform, account, organization, folder, agent, table` (allow-list PRD_MODULES). Colapsa el primer segmento del name del resource (agent.* → agent). EXCLUYE no-PRD: access.* (interno), dashboard, chatbot-configuration (legacy sin punto). Sub-secciones de agent (inbox, kb, flows, etc.) colapsan a `agent`.
  - Por rol: superadmin/implementador → los 6; admin → 5 (sin platform); supervisor/operator → 3 (folder, agent, table).
- Implementación en `Roles/application/role/get.ts` (`resolveSections`) + test. Phantom-safe, per-role Promise.all, tenant isolation intacta.

### Seed DEV — rol Implementador
- Descubierto en dry-run: el rol **implementador NO existía en DEV** → sus ~99 permisos se saltaban.
- Corrido real: `seed-default-roles.ts` creó `implementador` (id `0db6bed5-43ea-4552-b81c-a1fdb3537ba2`), luego `seed-role-permissions.ts` creó **99 filas** de permiso (verificado con COUNT en el GSI roleId-resourceId-index).
- AWS_PROFILE=silia-engineer-operator-817389378997, SSO session 'silia', DEV solo. Tablas dev-app-silia-com-{role,resource,permission}, user=dev-app-silia-com-User.
- ⚠️ matriz role-permissions.json PENDIENTE de validación celda-por-celda de PMs.

### Conflicto resuelto
- Merge de develop (PR #1749 wizard) metió la versión VIEJA de get.ts/get.test.ts (sections = nombres de resource). Se quedó HEAD (versión PRD-modules, superset de tests). Merge commit `b213abae4` pusheado.

### Continuar por
1. PR feat/SL-1545-role-count → develop.
2. Seed Implementador en STAGING/PROD cuando toque (misma secuencia, dry-run primero, tablas por env).
3. Decisión: ¿UI quiere sub-secciones finas de agent (agent.inbox, agent.kb) en vez de colapsar a `agent`?

### Sesiones previas
- [[Claude Sessions/silia/SL-1283-subject-grants-inheritance/2026-07-28]] — grants heredados + team en /access/grants; AccessModal color-only + isInternalAgent fix.
- [[Claude Sessions/silia/access-grants-counters/2026-07-27]] — counters SL-1281 + cross-account SL-1278 + escalation-inbox staging.

### Pendientes de sesiones previas (aún abiertos)
- escalation-inbox STAGING 500: redeploy Assistant→staging (`gh workflow run deploy-service.yml -f service_name=Assistant -f ref=staging -f code_ref=staging`). [[project_casl_iam_grants_gap]]