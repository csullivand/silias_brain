# Active Context

## Sesión actual: [SL-1637 / 7.2 deploy] + [Feature 9.2 investigación VERIFICADA] (2026-09-04)
Nota: [[Claude Sessions/silia/Feature-7.2-skill-assistant-service/2026-09-04]]

**Estado: 7.2 desplegándose vía PR #2292 (BLOQUEADO por build de sharp). 9.2 investigado y verificado; In Progress, sin código.**

### Hecho
1. **7.2:** anchor fix cerrado (`{segmentIndex,start,end}`), reviews PASS, CI replicado. Deploy vía bump de puntero de Skills → **PR #2292** (`d856f8c → 6b9788e`).
2. **BLOQUEO de deploy (NO es 7.2):** `nx build Skills` falla en `SaveCustomIntegration` — webpack no parsea el `.so` de sharp (cadena Chatbot.model → VectorDuplicationService → transformers → sharp). Fix = externals de sharp/@img/@xenova en `webpack.config.js`. **Otro equipo lo toma** (PR infra aparte). 7.2 no despliega hasta ese fix.
3. **Feature 9.2:** `docs/feature-9.2-plan.md` — investigación (5 agentes) + **2ª pasada adversarial (3 verificadores): 12/12 claims CONFIRMED**. Reframe clave: el motor de ejecución YA existe (loop TS + engine Python con selección por intent, version-pin, run-log); el net-new es el PUENTE `ProcessFlow`→engine + sync, más Data Views enforcement runtime + MCP callTool + trigger de entrada. Boundary Skills↔Agent es la decisión que gatea.

### Pendientes
1. **Deploy 7.2:** esperar el fix de webpack (otro equipo); luego mergear PR #2292 → develop dispara deploy.
2. **9.2 arranque:** bloque TS-puro de tablas (AC3+AC4) — dispatcher canónico→Data Views + `assertAgentCanOperate` runtime + tool enums cerrados (reuso alto, no depende del boundary). En paralelo: cerrar boundary con AGE (quién dueña sync ProcessFlow→engine; qué subsistema ejecuta) + 3 open questions del PRD.
3. Cleanup 7.2 opcional: `Post/index.ts:114`, `errors.ts:40`, cobertura 409-vía-appendTurn.

### Referencia
- [[Skill Assistant 7.2 - Plan BE]] · [[Skill Assistant 7.2 - FE Integration]] · `docs/feature-9.2-plan.md`
- Sesión previa (2026-08-31): diseño 7.2. [[Claude Sessions/silia/Feature-7.2-skill-assistant-service/2026-08-28]]