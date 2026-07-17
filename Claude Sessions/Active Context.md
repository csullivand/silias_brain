# Active Context

## Sesión actual: SL-1289 — Feature 10 roles + contadores + visibilidad + logging (2026-07-16)
Nota completa: [[Claude Sessions/silia/feature-10-roles/2026-07-16]]

**Rama:** fix/SL-1289-default-roles-migration (12 commits, PR NO abierto).
**Estado:** implementación completa; review+adversarial = PASS ✅. Falta DEPLOY + SEED + verificar.

### Qué se hizo
- Feature 10 backend completo (12 subtareas): modelo roles custom por cuenta (accountId nullable/isDefault/soft-delete + GSI), CRUD validado, clone, category, audit dedicado, resolución account-aware (S12 desbloquea Feature 11), invalidación efectivos, seed idempotente.
- Contadores EFECTIVOS con herencia en management (listOrgUsers) y teams (getTeamsByAccount).
- Fix TDZ "Cannot access uh before initialization" en listOrgUsers (import dinámico concurrente → carga memoizada).
- Visibilidad fail-closed en listFolders (era fail-open = fuga).
- Logging estructurado getLambdaLogger en grants+roles CRUD.
- Teams: Folder/Chatbot via dynamic import memoizado (evita ciclo).

### Continuar por (en orden)
1. DEPLOY infra (Roles GSI+RoleAuditLog+clone+envs+IAM dedicado; Teams envs FOLDERS/CHATBOT) + código Lambdas. Verificar cloneMin.js.
2. Correr scripts/migrations/seed-default-roles.ts (pobla isDefault).
3. Verificar: management counts≠0, teams herencia, log visible_to=me con operador.
4. Abrir PR a develop con reportes review+adversarial.

### Bloqueos de producto (datos, no código)
- Matriz ~102 permisos validada → seed rol→permiso + category.
- "Implementador" (PRD) vs superuser/superadmin (enum código).
- Canal Slack para alerta >20 roles.

### Pendiente futuro
- Feature 11 (asignación custom) — S12 lo habilita.
- KMS Opción A para conteo de grants en delete de rol.