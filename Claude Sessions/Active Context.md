---
tags: [active-context]
---

# Active Context

## Current Session
- **Note:** [[Claude Sessions/loteria/Setup Session Tracking/2026-05-13]]
- **Project:** loteria
- **Working directory:** /Users/sulli/Documents/PersonalWork/Loteria/loteria-suerte
- **Last updated:** 2026-05-14

## What's Happening
- APP DEPLOYED to Oracle Cloud: http://157.151.226.186:4000
- Oracle Linux 9, x86, 945MB RAM + 2.5GB swap
- Node 22 via nvm, PM2, frontend built locally and uploaded
- Fixed: CORS origin for IP access, cookie secure flag for HTTP
- Server serves frontend static files + API on same port

## Current State
- **Git:** 3 files modified (server.js, auth/routes.js, vite.config.js), uncommitted
- **Server:** PM2 running, health OK, user testing in browser
- **Pending:** Commit deploy fixes, test in browser, setup PM2 startup