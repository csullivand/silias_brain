---
tags: [concept, silia, access, permissions, casl, authorization]
---

# Silia — Access / Permission System (cómo funciona)

Conocimiento no obvio del módulo Access, aprendido implementando [[Claude Sessions/silia/feature-5-access-grants/2026-07-15|Feature 5]].

## AccessGrant (grants polimórficos)
- Tabla `${StackName}-AccessGrant` (CMK-encriptada con AccessGrantTableKmsKey).
- Campos: id, accountId, subjectType (user|team), subjectId, objectType (folder|agent|table), objectId, roleId, grantedBy, createdAt, updatedAt. (+ deletedAt/ttl tras Feature 5 soft delete).
- GSIs: objectId-subjectId-index, subjectId-objectType-index, accountId-createdAt-index.
- Modelo: Access/domain/models/AccessGrant.model.ts. Métodos: findByObjectId, findBySubjectId, findByAccountId, findByObjectAndSubject, findByObjectIds, deleteByObjectId, deleteAllBySubject, softDelete.

## Roles son GLOBALES (no por cuenta)
- La tabla Role (`${StackName}-role`) NO tiene accountId. Roles: superadmin, superuser, admin, supervisor, operator (Role enum en shared/config/roles.ts, valores en MINÚSCULAS).
- ELEVATED_ROLES = [SUPERADMIN, SUPERUSER, ADMIN] — bypassan checks object-level.
- Validar 'role pertenece a cuenta' = imposible; se valida existencia+activo (RoleModel.findById).

## El permiso object-level = NOMBRE del Resource
- PermissionResolver.getPermissionsForRole → mapea Permission(roleId→resourceId) al **resource.name** directamente. El campo 'permission'/'category' del Permission NO se usa en la resolución.
- Entonces el permiso 'folder.share' = un **Resource llamado 'folder.share'**. Igual 'folder.view', 'agent.view', etc.
- assertObjectPermission(event, 'folder.share', 'folder', objectId) → canOnObject → hasPermissionOnObject → resolveEffectivePermissions → .permissions.includes('folder.share').
- Seed: scripts/migrations/seed-access-resources.ts crea Resources + concede a roles.

## Modo PERMISIVO por defecto (¡importante!)
- Sistema object-level corre en modo permisivo salvo PERMISSION_STRICT_MODE=true.
- En permisivo: si el check FALLA → **loguea warning y DEJA PASAR** (no 403). Admins bypassan.
- El flag es GLOBAL (afecta folders, agents, tables — todos los endpoints object-level). Prenderlo = rollout coordinado.
- Implicación: un check object-level 'nuevo' no bloquea a nadie hasta strict mode.

## Reads de tablas CMK cross-stack NO requieren kms:Decrypt del caller
- DynamoDB descifra server-side con su grant de servicio. Precedente: Folders lee AccessGrant (CMK) con solo dynamodb:Query, sin KMS. (Verificado en Feature 5.)

## Role compartido con dynamodb:* 
- Los Lambdas del módulo Access corren bajo un **Role compartido** (Role: !Ref Role, definido en template.yaml raíz) con `dynamodb:[Query,GetItem,PutItem,Scan,DeleteItem,UpdateItem,BatchWriteItem] on "*"`. Por eso no necesitan IAM per-tabla.
- Otros módulos (Teams, User, Folders, DynamicTables) usan **roles dedicados** por función → sí necesitan agregar IAM por tabla al leer AccessGrant cross-módulo.

## Cache de permisos
- PermissionCache (Redis): invalidateForObject, invalidateForUser, invalidateForFolderTree, invalidateForAccount. Redis requiere VpcConfig + REDIS_URL_SSM_PARAM (solo módulo Access lo tiene wired).
- Feature 6 = PermissionInvalidationService + processor SQS.

## Herencia (folder ancestry)
- VisibilityResolver + PermissionResolver: un objeto hereda grants de carpetas ancestro. Path format: '/anc1/anc2/folderId'. Agent/table heredan de su folderId + ancestros de ese folder.


---

## ⚠️ CORRECCIÓN (2026-07-15): leer AccessGrant CMK SÍ requiere kms:Decrypt del caller

La sección de arriba decía que 'reads de tablas CMK cross-stack NO requieren kms:Decrypt del caller'. **ESO ES INCORRECTO** — un error de producción lo probó (`teams-get-all-role` → KMS AccessDenied al leer AccessGrant).

**Lo correcto:** leer una tabla DynamoDB cifrada con **CMK** requiere que el rol caller tenga:
1. `dynamodb:Query/GetItem` sobre la tabla + GSI, **Y**
2. `kms:Decrypt` (via-service dynamodb) en IAM, **Y**
3. que el **KEY POLICY del CMK** lo permita (delegar a root vía DynamoDB = escalable).

**AccessGrant CMK:** su key policy solo permitía el Role compartido → los roles dedicados fallaban. Fix (Opción A): statement `AllowAccountRolesViaDynamoDB` en el key policy (delega a root vía DynamoDB) + kms:Decrypt IAM en cada rol lector.

**FoldersTable CMK:** ya delega a root (`EnableIAMUserPermissions: kms:*`) → solo faltaba el kms:Decrypt IAM en el rol.

Ningún módulo leía AccessGrant cross-stack bien; los que lo intentaban (counters, listFolders?visible_to=me) fallaban silenciosamente (fail-open). Ver [[Claude Sessions/silia/resource-counts-teams-users/2026-07-14]] (BUG #2).