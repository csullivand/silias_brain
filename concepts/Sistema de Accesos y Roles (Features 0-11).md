---
tags: [silia, access-control, roles, permissions, arquitectura]
---
# Sistema de Accesos, Roles y Permisos (Features 0-11)

Documento completo en el repo: `docs/access-control-system-overview.md`. Relacionado: [[Claude Sessions/silia/feature-10-roles/2026-07-16]].

## Estado por feature
- **F0** permisos atómicos + roles default: ✅ motor DB-driven (sin enum hardcoded; matriz PRD pendiente). SL-1271.
- **F1** objetos folders/agents/tables: ✅ (path, 5 niveles, N:N agente-tabla). SL-1273/1274.
- **F2** home visible_to=me: ✅ fail-closed. SL-1278.
- **F3** users CRUD: ✅ (⚠️ email invitación NO). SL-1281.
- **F4** teams: ✅ (⚠️ team=skill RTA NO). SL-1281.
- **F5** Share Access/grants: ✅ CRUD (⚠️ sin validar roleId, grants heredados borrables, audit solo CloudWatch). SL-0/1282.
- **F6** herencia: ✅ tiempo real + regla de unión. SL-1284.
- **F7** drawer access de user: ❌ no hay endpoint.
- **F9** linkeo agente-tabla: ✅ (junction N:N + tools; flag off). SL-1274.
- **F10** roles custom: ✅ construido, SIN MERGEAR (rama fix/SL-1289).
- **F11** asignación custom: ⚠️ backend resuelve (S12), falta cerrar dropdowns + guardar roleId en user.

## Los 6 conceptos clave
1. Objeto de primer orden (folder/agent/table, con folderId).
2. Permiso atómico = string = name de un Resource en DB (~102).
3. Rol = combinación de permisos (5 default globales + custom por cuenta).
4. Grant (AccessGrant) = sujeto(user/team) + rol + objeto. Lo crea Share Access.
5. Herencia = compartir folder padre baja acceso a hijos, calculado al leer por path (no materializado).
6. Rol/permiso efectivo = UNIÓN de rol global + roles vía teams + grant directo + heredados. Gana el que da más.

## Motor de permisos (F0)
3 tablas: Role, Resource (name=permiso), Permission (=role_permissions). assertPermission → CASL can(allowed, permiso). PERMISSION_STRICT_MODE OFF por default (advierte, no bloquea). Elevados (superadmin/superuser) bypass. Caché 30s memoria + 5min Redis (efectivos).

## Resolución efectiva (F6) = la pregunta central
resolveEffectivePermissions(user, account, role, objectType, objectId) = unión de direct + team + inherited (ancestros por path, 5 niveles). Cache Redis 5min, invalidación sync/SQS>100.

## Pendientes clave
- Producto: matriz ~102 permisos validada, email provider, "Implementador" vs enum.
- Código: F5 validar roleId + grants heredados inmutables + audit table; F7 drawer; F11 guardar roleId en user + asignación; F10 deploy+PR.


## Flujos detallados (diagramas de secuencia + endpoints paso a paso)
Documento: `docs/access-control-flows.md`. Cubre 13 flujos:
- F0 Login/bootstrap (GET /me/permissions)
- F1 Chequeo de un permiso (assertPermission, permisivo por default)
- F2 Resolución efectiva (resolveEffectivePermissions = unión direct+team+inherited) ← el corazón
- F3 Crear rol custom (GET catalog → POST /role)
- F4 Editar permisos de rol → invalidación de efectivos (sync≤100/SQS>100)
- F5 Clonar rol
- F6 Crear usuario (POST /user/management)
- F7 Crear team + members (POST /teams/, POST /teams/{id}/members)
- F8 Share Access (POST /access/grants — team sobre folder)
- F9 Operador navega home con herencia (GET /folders?visible_to=me, fail-closed)
- F10 Mover objeto → recalcula herencia por path
- F11 Linkeo agente-tabla (POST /tables/{id}/agents, doble permiso + tools)
- F12 Borrados/cascadas (rol/user/team)

Basepaths: Access=access, Folders=folders, Roles=permissions, Teams=teams, User=user, DynamicTables=dynamic-tables.