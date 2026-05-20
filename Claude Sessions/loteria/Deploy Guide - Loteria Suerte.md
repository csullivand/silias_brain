# Deploy Guide - Loteria Suerte

## Server Details
- **IP:** 157.151.226.186
- **User:** opc
- **SSH:** `ssh opc@157.151.226.186`
- **OS:** Oracle Linux x86_64 (Oracle Cloud Always Free)
- **RAM:** 498MB + 2.5GB swap
- **App path:** /opt/loteria
- **DB path:** /opt/loteria/backend/data/db.sqlite
- **Node:** v22.22.3 via nvm (`/home/opc/.nvm/versions/node/v22.22.3/bin`)
- **No hay git en el servidor** — se sube con rsync
- **Oracle Cloud Console:** para reboots cuando la VM se cuelga (OOM)

---

## Software instalado en el servidor

| Software | Versión | Cómo se instaló | Notas |
|----------|---------|-----------------|-------|
| Node.js | v22.22.3 | nvm | Path: ~/.nvm/versions/node/v22.22.3 |
| PM2 | latest | npm global | Startup systemd: pm2-opc.service |
| Caddy | v2.11.3 | Binary directo (curl desde caddyserver.com/api/download?os=linux&arch=amd64) | NO usar dnf install (mata la VM por OOM) |
| better-sqlite3 | via npm ci | Dependencia del backend | Usa prebuilds |
| gzip | built-in | Ya viene con Oracle Linux | Para comprimir backups |

**NO instalar con dnf** en esta VM — los paquetes grandes causan OOM. Preferir binarios directos.

---

## Deploy Process (paso a paso)

### Desde la máquina local:

```bash
# 0. Estar en el directorio raíz del proyecto
cd /Users/sulli/Documents/PersonalWork/Loteria/loteria-suerte

# 1. Build frontend LOCALMENTE (servidor no aguanta npm ci + vite build)
cd frontend && npm run build && cd ..

# 2. Sync backend (excluir node_modules y data para no borrar la DB)
rsync -avz --exclude='node_modules' --exclude='data' --exclude='.git' --exclude='.claude' backend/ opc@157.151.226.186:/opt/loteria/backend/

# 3. Sync frontend dist (ya compilado)
rsync -avz frontend/dist/ opc@157.151.226.186:/opt/loteria/frontend/dist/

# 4. En el servidor: instalar deps backend, migrar, reload
ssh opc@157.151.226.186 "cd /opt/loteria/backend && npm ci --omit=dev && node src/migrations/runner.js && pm2 reload loteria-api"
```

### Verificar que funciona:
```bash
ssh opc@157.151.226.186 "curl -s -o /dev/null -w '%{http_code}' http://localhost:80/"
# Debe devolver 200
```

---

## Servicios (systemd)

### PM2 (backend Node.js)
- **Service:** pm2-opc.service
- **Configuración:** /opt/loteria/ecosystem.config.cjs
- **Logs:** /var/log/loteria/{out,err}.log
- **Startup:** configurado con `pm2 startup systemd`
- **Dump:** /home/opc/.pm2/dump.pm2

```bash
# Comandos útiles
pm2 list                          # Ver estado
pm2 reload loteria-api            # Reload sin downtime
pm2 logs loteria-api --lines 20   # Ver logs
pm2 start ecosystem.config.cjs    # Iniciar si no está corriendo
pm2 save                          # Guardar estado para resurrect
```

**⚠️ Problema conocido:** Tras reboot, PM2 a veces no hace resurrect automáticamente. Fix:
```bash
ssh opc@157.151.226.186 "cd /opt/loteria && pm2 start ecosystem.config.cjs && pm2 save"
```

### Caddy (reverse proxy + static files)
- **Service:** caddy.service
- **Binary:** /usr/bin/caddy (descargado directo, NO via dnf)
- **Config:** /etc/caddy/Caddyfile
- **Puerto:** :80 (HTTP, sin HTTPS por ahora — acceso por IP)

Caddyfile actual:
```
:80 {
  encode zstd gzip
  root * /opt/loteria/frontend/dist
  try_files {path} /index.html
  file_server
  handle /api/* {
    reverse_proxy localhost:4000
  }
}
```

**⚠️ SELinux:** Después de instalar/actualizar Caddy, SIEMPRE correr:
```bash
sudo restorecon -v /usr/bin/caddy
sudo setcap cap_net_bind_service=+ep /usr/bin/caddy
sudo systemctl restart caddy
```

---

## Cosas que NO hacer

| ❌ NO hacer | Por qué | Alternativa |
|-------------|---------|-------------|
| `npm ci` del frontend en servidor | OOM (498MB RAM) | Build local + rsync dist/ |
| `dnf install` paquetes grandes | OOM | Descargar binarios directos |
| rsync de `data/` | Borra la DB de producción | Siempre `--exclude='data'` |
| rsync de `node_modules/` | darwin vs linux binarios | `npm ci --omit=dev` en servidor |
| `node src/seed.js` con datos reales | Sobreescribe usuarios/datos | Solo en DB vacía |
| Operaciones pesadas simultáneas | OOM crash | De a una, esperar que termine |

---

## Problemas comunes y soluciones

### 1. VM no responde / SSH timeout
**Causa:** OOM — la VM se quedó sin memoria
**Fix:**
1. Ir a Oracle Cloud Console → Compute → Instances
2. Click en la instancia → Reboot (o Stop + Start)
3. Esperar ~2 min, verificar SSH
4. Levantar servicios:
```bash
ssh opc@157.151.226.186 "cd /opt/loteria && pm2 start ecosystem.config.cjs && pm2 save"
# Caddy debería levantar solo (systemd)
ssh opc@157.151.226.186 "sudo systemctl status caddy"
```

### 2. PM2 vacío tras reboot (tabla sin apps)
**Causa:** PM2 no hizo resurrect del dump
**Fix:**
```bash
ssh opc@157.151.226.186 "cd /opt/loteria && pm2 start ecosystem.config.cjs && pm2 save"
```

### 3. Caddy "Permission denied" / "Failed at step EXEC"
**Causa:** SELinux marca el binario como user_tmp_t (no ejecutable como servicio)
**Fix:**
```bash
ssh opc@157.151.226.186 "sudo restorecon -v /usr/bin/caddy && sudo setcap cap_net_bind_service=+ep /usr/bin/caddy && sudo systemctl restart caddy"
```

### 4. "Cannot find package 'express'" (o cualquier módulo)
**Causa:** Se subió backend sin node_modules y no se corrió npm ci
**Fix:**
```bash
ssh opc@157.151.226.186 "cd /opt/loteria/backend && npm ci --omit=dev && pm2 reload loteria-api"
```

### 5. Login "Usuario o contraseña incorrectos" en DB nueva
**Causa:** Migraciones crean esquema pero no datos. DB está vacía.
**Fix:**
```bash
ssh opc@157.151.226.186 "cd /opt/loteria/backend && node src/seed.js"
# SOLO en DB vacía. Crea admin/admin + vendedores demo
```

### 6. Frontend muestra versión vieja
**Causa:** Caddy sirve cache o no se subió el nuevo dist/
**Fix:** Verificar que se hizo build local + rsync del dist/, luego hard refresh (Cmd+Shift+R)

### 7. Caddy no está instalado tras reboot
**Causa:** Se instaló con dnf pero la VM murió antes de completar
**Fix:** Instalar como binario directo:
```bash
ssh opc@157.151.226.186 "curl -sL 'https://caddyserver.com/api/download?os=linux&arch=amd64' -o /tmp/caddy && sudo mv /tmp/caddy /usr/bin/caddy && sudo chmod +x /usr/bin/caddy && sudo restorecon -v /usr/bin/caddy && sudo setcap cap_net_bind_service=+ep /usr/bin/caddy && sudo systemctl restart caddy"
```

---

## Backup Automático (configurado 2026-05-19)

- **Script:** `/opt/loteria/scripts/backup.cjs` (usa better-sqlite3 .backup())
- **Cron:** cada 6 horas (`0 */6 * * *`)
- **Destino:** `/opt/loteria/backups/db-YYYY-MM-DD-HH-MM.sqlite.gz`
- **Retención:** últimos 30 backups (~5 días)
- **Log:** `/opt/loteria/backups/backup.log`

**Nota:** Se usa Node (backup.cjs) en vez de sqlite3 CLI porque la versión de glibc del servidor es muy vieja para el binario de sqlite3.

### Verificar backups
```bash
ssh opc@157.151.226.186 "ls -la /opt/loteria/backups/ && cat /opt/loteria/backups/backup.log | tail -5"
```

### Restore manual
```bash
ssh opc@157.151.226.186
pm2 stop loteria-api
# Elegir el backup más reciente
ls /opt/loteria/backups/
gunzip /opt/loteria/backups/db-YYYY-MM-DD-HH-MM.sqlite.gz -k
cp /opt/loteria/backend/data/db.sqlite /opt/loteria/backend/data/db.broken.sqlite
cp /opt/loteria/backups/db-YYYY-MM-DD-HH-MM.sqlite /opt/loteria/backend/data/db.sqlite
pm2 start loteria-api
```

### Descargar backup a local
```bash
scp opc@157.151.226.186:/opt/loteria/backups/db-YYYY-MM-DD-HH-MM.sqlite.gz ./
```

---

## Migraciones

- Path: `backend/src/migrations/`
- Runner: `node src/migrations/runner.js`
- Son **aditivas** (ALTER TABLE, CREATE TABLE) — NO borran datos
- El runner lleva control en tabla `_migrations` — solo aplica las nuevas
- Migraciones actuales: 001 (init) → 007 (paid_and_fund_moves)

---

## Checklist post-deploy

- [ ] `curl http://157.151.226.186` devuelve 200
- [ ] Login funciona
- [ ] PM2 status = online
- [ ] Caddy status = active
- [ ] Migraciones aplicadas (verificar en logs)

---

## TODO
- Configurar dominio + HTTPS (Caddy auto-TLS con Let's Encrypt)
- Subir backups a Cloudflare R2 para protección offsite
- Considerar migrar a Hetzner CX22 (4GB RAM, /mes) si OOM persiste