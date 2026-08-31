# Active Context

## Sesión actual: [Feature 7.2 BE] Servicio del Skill Assistant (2026-08-31)
Nota: [[Claude Sessions/silia/Feature-7.2-skill-assistant-service/2026-08-28]]

**Rama develop. Sin código aún — fase de diseño. Entregable: docs/skill-assistant-7.2-plan.md (nuevo, sin commit).**

### Hecho
1. **Review completo** del ticket 7.2 vs código del submódulo Skills/. El code assistant de custom integrations (Skills/application/CustomIntegrations/Assistant/) es el TEMPLATE — mismo patrón conversación+persistencia+LiteLLM+function-calling.
2. **6 decisiones cerradas con dev (Daniel Rubiano):**
   - Contexto del agente lo LEE el back de la tabla ${StackName}-Chatbot (estilo Voice, con chatbotId). Front NO lo manda; solo chatbotId + skill. No existe flow_context en repo. Falta env CHATBOT_TABLE + grant IAM read.
   - El servicio NUNCA escribe el skill; propone, front aplica al borrador, Save persiste (PUT /flows/{flowId}).
   - action = PARCHE (qué regla/qué párrafo), no diff ni doc regenerado.
   - Alcance v1 = 4 acciones: insert_instructions, edit_instructions, add_rule, edit_rule. Integration/MCP FUERA (ya en modales VOX-187/188/199).
   - Conversación: tope 100 turnos, sin resumen, sin TTL, borra solo con Reset (POST .../reset).
   - IA: mismo proxy LiteLLM que ya usa Skills (inference-ui + SSM /{StackName}/ai/rta/litellm/master-key). No OpenAI directo.
3. **Plan escrito** (docs/skill-assistant-7.2-plan.md): contrato API, modelo de datos (tabla FlowAssistantConversations, key userFlowKey=userId#flowId), agentContext.ts (el único trozo nuevo, read Chatbot estilo Voice replicado — Skills submódulo no puede importar Assistant/), system prompt per-tab, function-calling tools mode-gated, infra SAM, checklist.

### Pendientes
1. Usuario revisa/aprueba el plan.
2. Confirmar con FE (7.1): locator de edit_instructions (propuesto 'anchor' textual) + shape del body.
3. Si aprueba: implementar según checklist §12 del plan.

### Contexto previo
- Feature 7 SL-1477 Kanban laneSorts per-user — commits pusheados. [[Claude Sessions/silia/SL-1477-kanban-lanesorts-per-user/2026-08-21]]