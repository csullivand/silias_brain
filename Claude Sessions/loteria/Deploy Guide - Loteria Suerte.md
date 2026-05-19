# Deploy Guide - Loteria Suerte

## Server Details
- **IP:** 157.151.226.186
- **User:** opc
- **SSH:** `ssh opc@157.151.226.186`
- **OS:** Oracle Linux x86_64
- **RAM:** 498MB + 2.5GB swap
- **App path:** /opt/loteria
- **DB path:** /opt/loteria/backend/data/db.sqlite
- **Node:** v22 via nvm
- **Branch en servidor:** no hay git, se sube con rsync

---

## Deploy Process

```bash
# 1. Build frontend LOCALMENTE (servidor no aguanta npm ci + vite build)
cd frontend && npm run build

# 2. Sync backend (excluir node_modules y data para no borrar la DB)
rsync -avz --exclude='node_modules' --exclude='data' --exclude='.git' --exclude='.claude' backend/ opc@157.151.226.186:/opt/loteria/backend/

# 3. Sync frontend dist (ya compilado)
rsync -avz frontend/dist/ opc@157.151.226.186:/opt/loteria/frontend/dist/

# 4. En el servidor: instalar deps, migrar, reload
ssh opc@157.151.226.186 "cd /opt/loteria/backend && npm ci --omit=dev && node src/migrations/runner.js && pm2 reload loteria-api"
```

---

## Servicios en el servidor
- **PM2:** loteria-api (ecosystem.config.cjs). Startup systemd configurado (pm2-opc.service)
- **Caddy:** reverse proxy :80 → localhost:4000 para /api/*, file_server para frontend. Systemd service habilitado
- **SELinux:** Caddy necesita `sudo restorecon -v /usr/bin/caddy` después de instalarlo

---

## Cosas que NO hacer
- **NO correr `npm ci` del frontend en el servidor** — causa OOM (945MB RAM)
- **NO hacer rsync de `data/`** — borra la base de datos de producción
- **NO hacer rsync de `node_modules/`** — arquitectura puede diferir (local=darwin, server=linux)
- **NO correr `node src/seed.js` en producción con datos reales** — solo en DB nueva/vacía

---

## Problemas comunes

### VM no responde (OOM)
- Reiniciar desde Oracle Cloud Console: Compute → Instances → Reboot
- El swap de 2.5GB ayuda pero operaciones pesadas aún pueden matar la VM

### PM2 no levanta la app tras reboot
- PM2 startup está configurado pero a veces no hace resurrect
- Fix: `ssh opc@... "cd /opt/loteria && pm2 start ecosystem.config.cjs && pm2 save"`

### Caddy "Permission denied" tras instalar
- SELinux bloquea binarios descargados manualmente
- Fix: `sudo restorecon -v /usr/bin/caddy && sudo setcap cap_net_bind_service=+ep /usr/bin/caddy`

### "Cannot find package 'express'"
- Se subió el backend sin node_modules y no se corrió npm ci
- Fix: `cd /opt/loteria/backend && npm ci --omit=dev && pm2 reload loteria-api`

### Login no funciona después de migración desde cero
- La DB se recreó vacía (las migraciones solo crean esquema, no datos)
- Fix: `node src/seed.js` (SOLO si la DB está vacía, nunca en producción con datos)

---

## Migraciones y datos

Las migraciones en `backend/src/migrations/` son **aditivas** (ALTER TABLE, CREATE TABLE). No borran datos existentes. El runner solo aplica las que faltan.

**IMPORTANTE:** Si la DB de producción se borra accidentalmente:
1. Los datos se pierden (no hay backup automático configurado aún)
2. Se puede restaurar estructura con migraciones, pero los datos (ventas, tickets, usuarios) no se recuperan
3. **TODO:** Configurar backup automático con rclone a Cloudflare R2 (ver docs/09-deployment.md)

Para proteger la DB en deploys, el rsync SIEMPRE excluye `data/`:
```
--exclude='data'
```