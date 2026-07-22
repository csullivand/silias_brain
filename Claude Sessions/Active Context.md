# Active Context

## Sesión actual: Feature 0 CASL — IAM grants + CORS + public-endpoint bug (2026-07-22)
Nota completa: [[Claude Sessions/silia/feature-0-casl-iam-grants/2026-07-22]]

**Rama:** `feat/SL-1282-permissions` (HEAD 4d3d172e0). CASL IAM grants (12 módulos) YA commiteados. Fixes de hoy SIN commitear: webpack devtool, logger refactor, getWebChatConfig, Workflows/Integrations CORS.

### Estado por problema
1. **CASL IAM grants** → aplicados a los 12 módulos (~91 roles). Commiteados. Falta DESPLEGAR.
2. **Seeds** → STAGING (A+B) y DEV (B) corridos. DEV: role=7, resource=190, permission=572.
3. **Workflows/Integrations CORS** → era 401 en OPTIONS preflight. Fix: `AddDefaultAuthorizerToCorsPreflight: false`. Sin commitear/desplegar.
4. **Accounts deploy** → bloqueado por (a) tabla huérfana `dev-app-silia-com-FolderAssistantCreditAllocations` (borrarla) y (b) `ResetAssistantCreditsFunction` >250MB (fix webpack devtool→false, sin commitear).
5. **/chatbot/{id} 403** → BUG REAL (no deploy stale): endpoint PÚBLICO (sin authorizer) al que Jul-17 le metieron `assertPermission`. Quité el guard de `getWebChatConfig.ts`. Sin commitear/desplegar.
6. **Logger** → requirePermission.ts migrado a Pino securityLog. Sin commitear.

### Continuar por
1. Borrar tabla huérfana → redeploy Accounts (CI Deploy Service hace build limpio).
2. Commitear fixes de hoy + redeploy Assistant/Workflows/Integrations/Metrics via GitHub Actions.
3. Auditoría fiable (sub-agents por módulo) de OTRAS rutas públicas con guard erróneo.
4. Añadir ACAO a DEFAULT_4XX/5XX gateway responses.

### Aprendizaje clave
- CI (`deploy-service.yml`) ya hace `npx nx reset` + runner efímero → builds FRESCOS. La 'stale deploy' NO era la causa; los bugs eran de config/código (authorizer faltante, guard en endpoint público, source-maps >250MB).
- `assertPermission` (con accountId) necesita Query en `-role/accountId-name-index`; super short-circuit oculta gaps.

### Sesiones previas
- [[Claude Sessions/silia/feature-0-permissions-wiring/2026-07-17]] — cableado permisos.