# Active Context

## Sesión actual: Access/counters multi-PR + escalation-inbox staging (2026-07-27)
Nota completa: [[Claude Sessions/silia/access-grants-counters/2026-07-27]]

**3 PRs abiertas, todas verdes, pendientes de merge + redeploy:**
| PR | Qué | Servicio |
|----|-----|----------|
| #1700 | Elevated roles cuentan grants reales en /user/management (SL-1281) | User |
| #1701 | Teams cuenta grants reales en /teams + IAM Folders/Chatbot (SL-1281) | Teams |
| #1714 | Gestionar grants en cuenta cambiada / cross-account (SL-1278) | Access + FE |

### Hallazgos clave (verificados en vivo con token)
- **#1700**: regresión de MERGE — #1692 quitó el short-circuit elevated en computeUserCounts; #1696 (Implementador, rama vieja) lo re-metió; el deploy lo shippeó. David mostraba 23/92, debe ser 3/2.
- **#1701**: Teams 0/0 porque fetchAccountResources usa el ChatbotModel completo (tokenizer NO bundleado en Teams Lambda) → throw → fail-open vacío → phantom-fix da 0. Fix: query raw a CHATBOT_TABLE + IAM Folders/Chatbot al role. [[project_teams_lambda_chatbotmodel_bundling]]
- **role-editor**: es un slug SOLO del frontend (AccessModal DEFAULT_GRANT_ROLE_ID). El backend siembra roles con UUID; editor/viewer no existen en ningún seed/matriz → grants colgantes. Se guardaron 15-16 jul, ANTES de que Feature 5 (#1594, 20 jul) añadiera la validación RoleModel.findById en createGrant.
- **#1714 (SL-1278)**: el JWT NUNCA se re-emite al cambiar de cuenta (AccountContext.tsx); token.accountId = cuenta HOME siempre. Los 4 endpoints de grants usaban token account → rotos cross-account. Fix: helper resolveTargetAccount + canAccessAccount; FE manda accountId (viewing account). [[project_multi_account_jwt_home_account]]

### ⚠️ Pendiente inmediato: escalation-inbox en STAGING falla (500)
- Lambda: **staging-app-silia-com-escalation-inbox** (SAM GetEscalationInbox, Assistant/application/Escalations/Lifecycle/Inbox).
- Error: AccessDenied dynamodb:Query en staging-app-silia-com-role/index/accountId-name-index. Caller supervisor → assertPermission('agent.inbox.view') → findByAccountAndName.
- Causa: DEPLOY GAP. El grant CASL YA está en la plantilla (aws.template.yml:1555-1564, commit b7f1d0795 del 21-jul) y en la rama staging, pero Assistant NO se ha redeployado a staging desde entonces → IAM stale. [[project_casl_iam_grants_gap]]
- Fix: `gh workflow run deploy-service.yml -f service_name=Assistant -f ref=staging -f code_ref=staging`

### Continuar por
1. Merge #1700/#1701/#1714 + redeploy User/Teams/Access.
2. Redeploy Assistant→STAGING (arregla escalation-inbox, sin cambio de código).
3. Re-verificar en vivo: David 3/2, QA Escalations 2/2, inbox 200 para supervisor.
4. (Opc) auditar otras Lambdas Assistant no-super en staging por el mismo muro de IAM stale.

### Sesiones previas
- [[Claude Sessions/silia/SL-1281-temas-counter/2026-07-23]] — inicio SL-1281 (review + descripción).
- [[Claude Sessions/silia/feature-0-casl-iam-grants/2026-07-22]] — Feature 0 CASL IAM grants.