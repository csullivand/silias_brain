# Active Context

## Sesión actual: SL-1281 temas-counter — PR review + PR description (2026-07-23)
Nota completa: [[Claude Sessions/silia/SL-1281-temas-counter/2026-07-23]]

**Rama:** `feat/SL-1281-temas-counter` (base develop). SIN commits — todo el trabajo está en el working tree sin commitear. Por eso scripts/pr-review.sh y scripts/adversarial-verify.sh dan diff VACÍO (develop...HEAD).

### Qué hace el branch
1. **Phantom-grant fix** en `expandEffectiveCounts` (duplicada en User/application/get/listOrgUsers.ts EXPORTED y Teams/application/get/getTeamsByAccount.ts privada): antes sembraba el set con los IDs otorgados crudos → contaba folders/agents borrados. Ahora arranca vacío y sólo cuenta grants que resuelven a un recurso VIVO + herencia.
2. **Elevated short-circuit removido** en computeUserCounts (listOrgUsers.ts): admin/superuser ya NO devuelven el total de la cuenta; ahora cuentan sus propios grants (super sin grants → 0). Import ELEVATED_ROLES eliminado.
3. +3 tests de regresión phantom-grant.

### Estado
- Tests: 8 (listOrgUsers.effectiveCounts) + 2 (getTeamsByAccount) → verde.
- Review: Approved with suggestions. Adversarial: PASS (1 nota INFO).

### Continuar por
1. **Commitear** los 3 archivos de código para que scripts/CI tengan diff.
2. (Opc) extraer expandEffectiveCounts a helper compartido + espejar tests phantom para Teams.
3. Confirmar con FE que usuarios elevados muestren conteo per-grant (no total de cuenta).
4. Abrir PR contra develop con la descripción ya redactada.

### Sesión previa (otro tema)
- [[Claude Sessions/silia/feature-0-casl-iam-grants/2026-07-22]] — Feature 0 CASL IAM grants (rama feat/SL-1282-permissions).