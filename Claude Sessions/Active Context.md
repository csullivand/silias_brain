# Active Context

## Sesión actual: [SL-1637 / 7.2] Deploy del Skill Assistant + investigación 9.2 (2026-09-04)
Nota: [[Claude Sessions/silia/Feature-7.2-skill-assistant-service/2026-09-04]]

**Estado: 7.2 desplegándose vía PR #2292 (bump del puntero de Skills). 9.2 solo investigado.**

### Hecho hoy
1. **Contract open item cerrado:** `edit_instructions.anchor` = `{ segmentIndex, start, end }` (no texto exacto). Backend emite `null` si el modelo no produce anchor válido; FE deja el borrador intacto. Landeado en Skills develop (`10a3e69`).
2. **Reviews (PR + adversarial):** código 7.2 correcto/seguro/testeado (5/5 lentes). Barrido del repo: era el único caso del anti-patrón exact-text-anchor.
3. **CI replicado localmente:** lint, tests (58/58 Assistant), compare-eslint, runtime-consistency, governance → verde. Checkov marcó 2 tablas PREEXISTENTES de Alejandro (CustomIntegrations/McpServers, sin PITR) — no nuestras.
4. **Corrección clave del deploy:** un pointer-bump se ve como gitlink → los jobs file-scoped (iac-scan/security-lint/runtime-consistency/secret-scan) SE SALTAN el submódulo. lint + tests sí cubren Skills y pasan. **No se espera rojo; Checkov ni mira el template.**
5. **Deploy:** rama `chore/SL-1637-bump-skills-pointer`, commit `2b51933b2` (gitlink `d856f8c → 6b9788e`), **PR #2292 → develop** abierto.
6. **Investigación 9.2** (`docs/feature-9.2-plan.md`): el motor de ejecución YA existe (loop TS `CompletionService`/`ToolsService` + submódulo Python `Agent`). El net-new es el PUENTE `ProcessFlow`(Skills)→engine por intent + sync; y Data Views enforcement runtime + MCP callTool + trigger de entrada. Boundary: modo `ENGINE` delega al engine Python (hoy un solo `agent_engine_flow_id`).

### Pendientes
1. Revisar/aprobar **PR #2292**; al mergear a develop dispara deploy (+ promote-envs).
2. 9.2: llevar §2 (boundary) del plan al equipo AGE/engine — quién dueña el sync `ProcessFlow`→engine y cómo unir `pipeline/layer3` (selección) con `engine-poc` (ejecución). Cerrar 3 open questions del PRD.
3. Cleanup 7.2 opcional (no bloqueante): `Post/index.ts:114` instanceof redundante; `errors.ts:40` msg default dice "code assistant"; cobertura 409-vía-appendTurn.

### Contexto previo
- Sesión anterior (2026-08-31): diseño del servicio 7.2 (plan BE). [[Claude Sessions/silia/Feature-7.2-skill-assistant-service/2026-08-28]]
- Docs de referencia (untracked en Silia): [[Skill Assistant 7.2 - Plan BE]], [[Skill Assistant 7.2 - FE Integration]], `docs/feature-9.2-plan.md`.