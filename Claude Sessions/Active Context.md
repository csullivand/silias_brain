---
tags: [claude-session, active-context]
updated: 2026-07-15
---

# Active Context

## Estado (2026-07-15) — 2 ramas activas

### 1. Feature 5 access_grants (SL-1282)
- **Rama:** `fix/SL-1282-access-polimorficos` — código COMPLETO, commiteado+pusheado (e28ef04e0). Reviews en verde.
- **⚠️ PENDIENTE INMEDIATO: aplicar Opción A (KMS) a esta rama** — sus lectores cross-módulo de AccessGrant (roles dedicados: TeamsDeleteTeamRole, UserDeleteOrgUserRole, FolderWriteRole, DtTableWriteRole, RoleChatbotWrite, TeamsGetUsersRole) necesitan `kms:Decrypt` IAM (via-service) o fallarán al desplegar. El delegate del key policy llega vía el merge de la rama de counters.
- Detalle: [[Claude Sessions/silia/feature-5-access-grants/2026-07-15]].

### 2. Counters + Embeds (SL-1281 follow-up)
- **Rama:** `feat/SL-1281-embed-teams-members` — pusheada. 4 commits:
  1. embeds `teams`[{id,name}] + `members`
  2. fix IAM AccessGrant en TeamsGetAllRole (bug #1 DynamoDB)
  3. **Opción A** (bug #2 KMS): delegate CMK AccessGrant + kms:Decrypt en TeamsGetAllRole/UserListOrgUsersRole
  4. logs de resumen (no per-item)
- **🐛 2 bugs de counters resueltos:** (1) IAM DynamoDB al rol equivocado; (2) KMS — leer AccessGrant CMK requiere kms:Decrypt del caller (mi suposición de Feature 5 era errónea). Ver [[Claude Sessions/silia/resource-counts-teams-users/2026-07-14]].
- Falta: **desplegar Access+Teams+User juntos**.

## PENDIENTE
1. **Aplicar Opción A a fix/SL-1282** (en curso).
2. Abrir PRs de ambas ramas.
3. Desplegar: counters (Access+Teams+User), Feature 5 (6 módulos + seed).
4. Coordinar con front el breaking teamNames→teams.

## Related
- [[Claude Sessions/silia/feature-5-access-grants/2026-07-15]]
- [[Claude Sessions/silia/resource-counts-teams-users/2026-07-14]]
- [[concepts/Silia Access Permission System]]