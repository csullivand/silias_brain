---
tags: [project, silia, feature-9.2, agent-engine, architecture]
date: 2026-09-04
---

# Feature 9.2 — Ejecución por Intent: motor y boundary Skills↔Agent

Investigación de código + verificación adversarial (12/12 claims confirmados). Doc fuente en repo: `docs/feature-9.2-plan.md`.

## Insight durable
**9.2 NO es construir un motor de ejecución — el motor ya existe.** Está repartido en dos mundos que hoy no se hablan:
- **TS (Silia):** el loop de function-calling `CompletionService`(OpenAI+Bedrock) → `ToolsService.execute` es reutilizable; el ejecutor de custom integrations existe (triplicado — `Skills/domain/customIntegrations/execute.ts` es el canónico, pero `ToolsService:1042` usa una copia inline propia).
- **Python (`Agent/` submódulo):** `engine-poc` tiene agentic loop + tool exec + **version-pin + run-logging** (`store.py:150 record_run_start(flow_id, flow_ver)`); `pipeline/layer3._select_flow` hace **selección multi-flow por intent**. Son **dos subsistemas distintos, sin unir**.

## El gap real
- **`ProcessFlow`** (Skills, persistido en `Process-Flows`) **no lo lee ningún runtime**. El pipeline inyecta `chatbot.executionFlow.steps` (concepto viejo) con `flows[].intents:[]` **hardcodeados** (`chatbotToPayload.ts:121,150`).
- **Cero sync `ProcessFlow`→engine** — los flows del engine se siembran a mano (`engine-poc/scripts/seed_santi_v2.py`).
- Modo `ENGINE` (`PipelineMode`) delega vía `runAgentEngine` a un solo `flow_id` (`agent_engine_flow_id ?? AGENT_ENGINE_DEFAULT_FLOW_ID ?? 'santi-chat'`).

## Net-new (corazón de 9.2)
Puente/sync `ProcessFlow`→formato del engine + poblar intents + unir selección(pipeline)↔ejecución(engine-poc). Y lo que no existe en ningún lado: **Data Views enforcement en runtime** (`assertAgentCanOperate`; `AgentTableConnection` solo valida al guardar), **MCP `callTool`** (solo hay probe/listTools), y el **trigger de integraciones de entrada** (mecánica abierta del PRD).

## Arranque recomendado (In Progress)
Bloque TS-puro de tablas (AC3+AC4): dispatcher canónico→Data Views (`extractRuntimeCapabilities` ya da la forma estructurada) + `assertAgentCanOperate` runtime + tool con enums cerrados (substrato: `RowListingToolService`). No depende del boundary. En paralelo: decisión de boundary con equipo AGE.

## Decisión que gatea todo
Quién dueña el sync `ProcessFlow`→engine y cuál subsistema Python ejecuta (engine-poc vs pipeline layer3). Cerrar con AGE antes de estimar en firme.