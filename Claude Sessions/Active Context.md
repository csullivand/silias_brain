# Active Context

## Sesión actual: SL-1283 subject grants inheritance + SL-1278 org mutations (2026-07-28)
Nota completa: [[Claude Sessions/silia/SL-1283-subject-grants-inheritance/2026-07-28]]

**Rama:** `fix/SL-1283-subject-grants-inheritance` — 3 commits pushed (HEAD `c928a14c7`).

### Qué se hizo hoy (todo pusheado)
| Commit | Qué |
|--------|-----|
| `3e0c7624f` | Org mutation handlers cross-account (SL-1278): CREATE usa resolveTargetAccount; MODIFY/DELETE by id autoriza contra la cuenta de la ENTIDAD (id→entity→account, sin account del cliente). 4 team-modify handlers + patch/deleteOrgUser. |
| `6ad775a26` | Fix: object_type=table&include_inherited ya NO mezcla filas folder/agent (guard temprano). |
| `c928a14c7` | **Core SL-1283:** GET /access/grants ahora devuelve grants heredados (subfolders/agents) + derivados de TEAM, igual que el COUNT (computeUserCounts). Filas heredadas read-only, id sintético no-revocable. FE AccessModal: color-only (resalta como granted, SIN botón, SIN badge de texto). |

### Hallazgos / decisiones clave
- Herencia es **opt-in** (`?include_inherited=true`) para no romper consumidores que revocan por id. Solo AccessModal la activa; useAccessGrants sigue direct-only.
- Filas heredadas = READ-ONLY, id sintético `inherited:<type>:<id>` → el FE nunca borra un grant real.
- Revoke safety en AccessModal: grantIdByObject + sets toggleables se siembran SOLO de grants directos; inheritedFolderIds/inheritedAgentIds separados para display.
- Keyed off LIVE folder/agent set → phantom (borrados) no inflan; direct gana sobre inherited; descendiente hereda del ancestro más profundo.
- Usuario eligió **color-only** sobre un label "Inherited" (se quitó el badge de texto + i18n).

### Estado
- BE + app tsc clean; 45 tests org-mutations + 15 tests grants pasan.
- Working tree: solo untracked scripts/docs/env.json (pre-existentes, no de este trabajo).

### Continuar por
1. Abrir PR fix/SL-1283-subject-grants-inheritance → develop (bundlea SL-1278 mutations + SL-1283 grants + org-page reads previos).
2. Post-merge: redeploy Access + User + Teams.
3. Live-verify: subject con acceso solo-por-team ve subfolders/agents heredados en AccessModal (resaltados, sin botón), reconciliando con los badges de count.

### Contexto previo (2026-07-27): 3 PRs access/counters + escalation-inbox staging
Nota: [[Claude Sessions/silia/access-grants-counters/2026-07-27]]
- #1700 (elevated counts SL-1281), #1701 (Teams counts + IAM SL-1281), #1714 (cross-account grants SL-1278) — pendientes merge+redeploy.
- ⚠️ escalation-inbox STAGING 500: deploy gap, redeploy Assistant→staging: `gh workflow run deploy-service.yml -f service_name=Assistant -f ref=staging -f code_ref=staging` [[project_casl_iam_grants_gap]]

### Sesiones previas
- [[Claude Sessions/silia/access-grants-counters/2026-07-27]] — counters SL-1281 + cross-account SL-1278 + escalation-inbox.
- [[Claude Sessions/silia/SL-1281-temas-counter/2026-07-23]] — inicio SL-1281.