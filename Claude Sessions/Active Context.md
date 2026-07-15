---
tags: [claude-session, active-context]
updated: 2026-07-15
---

# Active Context

## Estado (2026-07-15) — 2 líneas de trabajo activas

### 1. Feature 5 access_grants (SL-1282)
- **Rama:** `fix/SL-1282-access-polimorficos` — COMPLETO, commiteado + pusheado (commit e28ef04e0). Reviews (pr + adversarial) en verde.
- Detalle: [[Claude Sessions/silia/feature-5-access-grants/2026-07-15]].
- Falta: abrir PR, correr seed, desplegar 6 módulos, review.

### 2. Counters + Embeds (SL-1281 follow-up)
- **Rama:** `feat/SL-1281-embed-teams-members` — commiteado + pusheado. 3 commits:
  1. embeds: `teams`[{id,name}] en /management (reemplaza teamNames) + `members` en /teams.
  2. (mismo)
  3. **fix IAM**: AccessGrant Query en `TeamsGetAllRole` (commit 1177e8406).
- **🐛 Bug de counters resuelto:** /teams daba 0 porque el rol real (`teams-get-all-role`) no tenía AccessGrant (el PR SL-1281 se lo puso a `TeamsGetByAccountRole`, rol equivocado). Fail-open lo tragaba. Fix pusheado. ⚠️ aplica al DESPLEGAR Teams. Ver [[Claude Sessions/silia/resource-counts-teams-users/2026-07-14]].
- /management counters OK (UserListOrgUsersRole bien wired).

## PENDIENTE
- Abrir PRs de ambas ramas.
- **Desplegar**: Feature 5 (6 módulos + seed) y Teams (fix de counters).
- Coordinar con front el cambio teamNames→teams (breaking).

## Related
- [[Claude Sessions/silia/feature-5-access-grants/2026-07-15]]
- [[Claude Sessions/silia/resource-counts-teams-users/2026-07-14]]
- [[concepts/Silia Access Permission System]]