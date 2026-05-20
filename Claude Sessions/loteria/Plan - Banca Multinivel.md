# Plan: Banca Multinivel (Admin → Sub-admin → Vendor)

**Rama:** `bancaMultinivel`
**Estado:** Planificado, pendiente implementación

## Estructura (3 niveles máximo)
```
Admin principal (dueño de la banca)
├── Sub-admin A (le paga al principal)
│   ├── Vendedor A1 (le paga a Sub-admin A)
│   └── Vendedor A2 (le paga a Sub-admin A)
├── Sub-admin B (le paga al principal)
│   ├── Vendedor B1 (le paga a Sub-admin B)
│   └── Vendedor B2
└── Vendedor directo (le paga al principal)
```

## Reglas
1. Admin ve TODO
2. Sub-admin ve solo sus vendedores y sus propias ventas
3. Sub-admin puede: vender, registrar resultados, ver reportes (filtrados), ver premios, manejar sus vendedores
4. Sub-admin NO puede: configurar sorteos, cambiar topes, ver suscripción, configurar banca
5. Sub-admin también puede vender (como vendor con poderes extra)
6. Vendedor sigue igual

## Cambios principales

### Migración 012
- `parent_id` en users (nullable, FK a users)
- Role `subadmin` agregado al CHECK

### Backend: Helper centralizado `user-scope.js`
- `getVisibleVendorIds(db, user)` → null (admin), [ids] (subadmin), [userId] (vendor)
- `vendorScopeSQL(user, alias)` → genera WHERE clause según rol
- Se aplica en: tickets, payments, reports, results, vendors

### Backend: Rutas modificadas
- tickets.js — scope filter en GET, void, pay
- payments.js — scope filter
- reports.js — sales-detail, lista, commissions
- results.js — my-winners
- vendors.js — GET filtrado, POST con parent_id auto, PUT/DELETE con scope check
- requireRole('admin', 'subadmin') donde aplique

### Frontend
- Router: subadmin usa AdminLayout con menú filtrado
- Layout: ocultar Sorteos, Topes, Banca, Suscripción para subadmin
- Vendedores: jerarquía visual, crear sub-admin/vendor
- Agregar Vender al menú del sub-admin

### Pagos entre niveles
- Vendedor → paga al sub-admin
- Sub-admin → paga al admin
- Estado de cuenta refleja la jerarquía

## Archivos clave
- Crear: migrations/012_multinivel.sql, lib/user-scope.js
- Modificar: ~13 archivos (routes, frontend, auth)

## Testing
1. Admin ve todo
2. Sub-admin ve solo sus vendors
3. Sub-admin crea vendor → parent_id automático
4. Sub-admin NO ve config de sorteos/topes/banca
5. Sub-admin puede vender
6. Reportes filtrados correctamente
7. Pagos reflejan jerarquía