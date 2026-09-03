---
tags: silia,skills,feature-7,frontend,integration,api
---
# Skill Assistant (7.2) — Guía de integración Frontend

> Backend del **Skill Assistant**: el asistente flotante del editor de skills que
> redacta **instrucciones** y **business rules**. Mantiene una conversación
> persistente por skill, conoce el contexto del agente, y responde con un texto
> + una **acción estructurada** opcional que el front aplica al borrador.
>
> **El servicio NUNCA escribe la skill.** Propone; el front aplica al borrador
> del editor; el usuario persiste con **Save** (`PUT /skills/agents/{agentId}/flows/{flowId}`).
>
> Módulo: `Skills` · Contrato backend: `Skills/application/Flows/Assistant/CONTRACT.md`.

---

## 1. Base, auth y cliente

- Base del módulo: **`/skills`** (mismo `createApiClient('skills')` que `skillsService.ts` / `mcpServersService.ts`).
- Autenticación: authorizer de plataforma (JWT). Roles `ADMIN`, `IMPLEMENTADOR`, `SUPERUSER`, `SUPERADMIN`; permiso CASL `agent.process_flows.edit` (el mismo gate que el CRUD de skills).
- **`agentId` === `chatbotId`**: el back lee el contexto del agente de la tabla `Chatbot` con `id = agentId`. **El front NO manda contexto del agente** — solo el `agentId` en la ruta y (opcional) el borrador de la skill.

Sugerencia de archivo: `app/src/features/agent-skills/services/skillAssistantService.ts` (mismo shape que `customIntegrationAssistantService.ts`).

---

## 2. Endpoints

| Método | Ruta | Uso |
|---|---|---|
| `GET`  | `/skills/agents/{agentId}/flows/{flowId}/assistant` | Restaurar la conversación (al abrir el widget). |
| `POST` | `/skills/agents/{agentId}/flows/{flowId}/assistant` | Enviar un mensaje (un turno). |
| `POST` | `/skills/agents/{agentId}/flows/{flowId}/assistant/reset` | Restart: borrar el contexto y empezar de cero. |

> El `agentId` es el id del agente/assistant (el `:AssistantID` de la ruta del editor). El `flowId` es el id de la skill.

---

## 3. `GET` — restaurar conversación

Llámalo al abrir el widget (FAB → chat). La conversación persiste entre tabs, guardados y días; **solo el reset la borra**.

**Response 200 — conversación existente**
```jsonc
{
  "id": "conv-1",
  "flowId": "flow-1",
  "status": "active",
  "createdAt": 1710000000000,
  "lastMessageAt": 1710000005000,
  "turns": [
    { "id": "t1", "role": "user", "content": "Quiero un proceso de reembolsos", "createdAt": 1710000000000 },
    {
      "id": "t2", "role": "assistant", "content": "Te propongo este primer paso.",
      "createdAt": 1710000005000,
      "action": { "type": "insert_instructions", "payload": { "segments": [ /* … */ ] } }
    }
  ]
}
```

**Response 200 — skill sin conversación todavía** (NO es 404)
```jsonc
{ "conversationId": null, "flowId": "flow-1", "status": "active", "turns": [] }
```
> Ojo a la asimetría (igual que el asistente de código): con historial el id viene en **`id`**; vacía viene en **`conversationId: null`**. Para “¿hay conversación?” usa `resp.id ?? resp.conversationId`. Las `turns` de tipo `assistant` pueden traer `action` — **re-píntalas** como propuestas aplicables.

---

## 4. `POST` — un turno

**Request**
```jsonc
{
  "message": "quiero que primero valide el correo del cliente",
  "mode": "flow",              // "flow" (tab Instructions) | "rules" (tab Business rules). Default: "flow".
  "conversationId": "conv-1",  // opcional; omítelo en el primer mensaje.
  "skill": {                    // borrador VIVO del editor (lo no-guardado). Opcional pero recomendado.
    "name": "Reembolsos",
    "description": "Procesa reembolsos",
    "intent": "Solicitudes de reembolso",
    "instructions": { "schema_version": 1, "segments": [ /* InstructionSegment[] */ ] },
    "rules": [ { "id": "r-1", "name": "Márgenes", "description": "tope", "content": "Máx 10%." } ]
  }
}
```

- **`mode`** = el tab activo. Determina la persona del asistente y **qué acciones puede devolver**:
  - `"flow"` → `insert_instructions` / `edit_instructions`
  - `"rules"` → `add_rule` / `edit_rule`
- **`skill`** es el estado actual del editor (puede diferir de lo guardado). El back lo usa para el prompt; **no lo persiste**. Mándalo para que el asistente sea consistente con lo que el usuario ve.
- El widget vive una sola conversación entre ambos tabs: manda el mismo `conversationId` aunque cambies de tab.

**Response 200**
```jsonc
{
  "conversationId": "conv-1",
  "userTurn":  { "id": "t3", "role": "user", "content": "quiero que primero valide el correo del cliente", "createdAt": 1710000010000 },
  "assistantTurn": {
    "id": "t4", "role": "assistant",
    "content": "Agrego un paso de validación al inicio.",   // el texto del chat
    "createdAt": 1710000012000,
    "action": {                                              // opcional
      "type": "insert_instructions",
      "payload": { "segments": [ { "type": "text", "content": "Paso 0: valida el correo del cliente." } ] }
    }
  }
}
```

- Pinta `assistantTurn.content` como la burbuja del asistente.
- Si viene `assistantTurn.action`, muestra el **preview** de la propuesta con un botón para aplicarla (ver §5). **No la apliques automáticamente.**
- Si **no** viene `action`, es un turno conversacional (pregunta / aclaración / “necesitas crear una integración X en su panel”).

---

## 5. Acciones (`action.type`) y cómo aplicarlas

Todas son **parches** sobre el borrador del editor. El usuario confirma (botón); el front aplica; luego **Save** persiste vía el `PUT` de skills existente.

### `insert_instructions` (mode `flow`)
```jsonc
{ "type": "insert_instructions", "payload": { "segments": [ /* InstructionSegment[] */ ] } }
```
**Aplicar:** **agregar** los `segments` al final del documento de instrucciones (con una línea en blanco de separación). **Nunca sobreescribe** el contenido existente.

### `edit_instructions` (mode `flow`)
```jsonc
{ "type": "edit_instructions", "payload": { "anchor": "texto viejo a localizar", "segments": [ /* … */ ] } }
```
**Aplicar:** localizar el fragmento `anchor` (texto exacto) en las instrucciones y **reemplazar solo ese fragmento** por `segments`, sin tocar el resto.
> ⚠️ **A confirmar contigo (FE 7.1):** el `anchor` es texto exacto a ubicar. Si tu editor prefiere direccionar por id/rango de segmento en vez de por texto, dinos y ajustamos el contrato — es el único punto abierto.

### `add_rule` (mode `rules`)
```jsonc
{ "type": "add_rule", "payload": { "rule": { "name": "Márgenes", "description": "tope de descuento", "content": "Nunca descuentes más del 10%." } } }
```
**Aplicar:** agregar una regla nueva a la lista (tú asignas el `id`). Actualiza el contador del tab.

### `edit_rule` (mode `rules`)
```jsonc
{ "type": "edit_rule", "payload": { "ruleId": "r-1", "changes": { "content": "Nuevo cuerpo." } } }
```
**Aplicar:** modificar **solo** la regla `ruleId`; `changes` trae únicamente los campos que cambian (`name?`, `description?`, `content?`).

> **Fuera de alcance (v1):** el asistente NO crea/edita custom integrations ni MCP servers vía acción. Si el proceso necesita una que no existe, lo dirá en el `content` (texto) y el usuario la crea en su panel (los modales que ya existen).

---

## 6. `POST /reset` — Restart

Botón **Restart** del widget (ícono de reinicio, consistente con los otros asistentes). Cierra la conversación activa y devuelve una vacía. **Es lo único que borra el contexto** — cambiar de tab, guardar o navegar NO lo borran.

**Response 200**
```jsonc
{ "id": "conv-2", "flowId": "flow-1", "status": "active", "createdAt": 1710000020000, "lastMessageAt": 1710000020000, "turns": [] }
```

---

## 7. Errores

| HTTP | Cuándo | Manejo sugerido |
|---|---|---|
| `400` | Falta `agentId`/`flowId`, body inválido, `mode` desconocido, `message` vacío | Error de programación; corrige el request |
| `401` | Sin sesión | Redirige a login |
| `403` | Rol/permiso denegado | Oculta el widget o muestra “sin permiso” |
| `409` | La conversación fue reiniciada en otra pestaña, o llegó al tope de turnos | Recarga la conversación (GET) o invita a Restart |
| `502` | El modelo de IA falló/timeout | “El asistente no está disponible, reintenta”. El turno del usuario **ya quedó guardado**: reintentar = re-enviar el mensaje |
| `500` | Error inesperado | Toast genérico |

Cuerpo de error: `{ "error": string, "details"?: unknown }`.

**Tope de turnos:** 100 por conversación. Al llegar, el POST devuelve `409` → el usuario debe hacer Restart. No hay resumen automático.

---

## 8. Tipos TypeScript (copiables)

```ts
export type AssistantMode = 'flow' | 'rules';

export type SkillAssistantActionType =
  | 'insert_instructions'
  | 'edit_instructions'
  | 'add_rule'
  | 'edit_rule';

export interface SkillAssistantAction {
  type: SkillAssistantActionType;
  payload: Record<string, unknown>; // ver §5 según type
}

export interface SkillAssistantTurn {
  id: string;
  role: 'user' | 'assistant';
  content: string;
  createdAt: number;               // Unix ms
  action?: SkillAssistantAction;   // solo en turnos assistant con propuesta
}

// GET / reset
export interface SkillAssistantConversation {
  id?: string;                     // presente cuando hay historial
  conversationId?: string | null;  // presente (null) cuando aún no hay conversación
  flowId: string;
  status: 'active' | 'closed';
  createdAt?: number;
  lastMessageAt?: number;
  turns: SkillAssistantTurn[];
}

// POST request
export interface SkillAssistantChatRequest {
  message: string;
  mode?: AssistantMode;            // default 'flow'
  conversationId?: string;
  skill?: {
    name?: string;
    description?: string;
    intent?: string;
    instructions?: InstructionDocument; // { schema_version: 1, segments: InstructionSegment[] }
    rules?: Array<{ id: string; name: string; description: string; content: string }>;
  };
}

// POST response
export interface SkillAssistantChatResult {
  conversationId: string;
  userTurn: SkillAssistantTurn;
  assistantTurn: SkillAssistantTurn;
}
```

`InstructionDocument` / `InstructionSegment` son los mismos que ya usa el editor
(`app/src/features/agent-skills/instructions`): segmentos `text` /
`integration_chip` / `mcp_chip` / `table_op_chip`. Las acciones de instrucciones
devuelven exactamente ese shape, listo para insertar en el editor.

---

## 9. Servicio sugerido

```ts
import { createApiClient } from '…';
const api = createApiClient('skills');

export const getSkillAssistant = (agentId: string, flowId: string) =>
  api.get(`agents/${agentId}/flows/${flowId}/assistant`);

export const postSkillAssistant = (agentId: string, flowId: string, body: SkillAssistantChatRequest) =>
  api.post(`agents/${agentId}/flows/${flowId}/assistant`, body);

export const resetSkillAssistant = (agentId: string, flowId: string) =>
  api.post(`agents/${agentId}/flows/${flowId}/assistant/reset`, {});
```

---

## 10. Flujo típico

1. Abre el widget → **`GET`** → pinta `turns` (re-pintando las `action` como propuestas aplicables).
2. Usuario escribe → **`POST`** con `mode` (tab activo), `conversationId`, y el `skill` (borrador vivo).
3. Pinta `assistantTurn.content`. Si hay `action`, muestra preview + botón **Insert / Add rule / Apply**.
4. Al confirmar → aplica el parche al borrador del editor (§5). El usuario sigue editando.
5. **Save** (el `PUT` de skills que ya existe) persiste la skill. El asistente nunca persiste nada de la skill.
6. **Restart** cuando quiera empezar el hilo de cero.

---

## 11. Punto abierto (para cerrar contigo)

- **`edit_instructions.anchor`**: hoy el backend manda el texto exacto a localizar y reemplazar. Si el editor direcciona párrafos por id/rango de segmento, avísanos y cambiamos el payload de esa acción. Todo lo demás del contrato es estable.
