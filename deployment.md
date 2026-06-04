---
title: Manual de Operaciones
nav_order: 4
---

# Manual de Operaciones y Despliegue

**Web Service Pagos - Facturador REST API**

---

## 1. Arquitectura y Requisitos Previos

El sistema está construido en **Node.js + TypeScript** usando Clean Architecture, e interactúa directamente con **PostgreSQL** mediante el driver `pg` nativo para invocar procedimientos almacenados legados.

### Requisitos de Infraestructura

- **Runtime**: Node.js **v18.x LTS** o superior
- **Gestor de paquetes**: npm v9+
- **Base de datos**: PostgreSQL **v13+** con funciones PL/pgSQL pre-existentes
- **Reverse Proxy**: Nginx o similar (para terminación SSL)
- **Gestor de procesos**: PM2

---

## 2. Instalación en servidor limpio (Debian/Ubuntu)

```bash
# Node.js 20 LTS
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt install -y nodejs

# PM2
npm install -g pm2
```

---

## 3. Variables de entorno (`.env`)

El sistema usa **Zod** en tiempo de inicio para validar todas las variables. Si falta alguna, aplica *Fail-Fast* y no inicia.

```env
PORT=3000
NODE_ENV=production

DATABASE_URL=postgresql://<usuario>:<password>@localhost:5432/<nombre_db>

LOG_LEVEL=info

ADMIN_USER=admin
ADMIN_PASSWORD=<contraseña_segura>
ADMIN_JWT_SECRET=<secreto_jwt_largo>
```

---

## 4. Preparación de la base de datos

```bash
# Crea todas las tablas del servicio.
# Seguro de re-ejecutar (usa IF NOT EXISTS y ON CONFLICT DO NOTHING).
npm run db:init

# Aplica solo migraciones nuevas (actualizaciones incrementales).
npm run db:migrate
```

---

## 5. Puesta en Producción

### Opción A: VPS con PM2 (recomendado)

```bash
# 1. Compilar
npm run build
npm run build:admin

# 2. Correr migraciones pendientes
npm run db:migrate

# 3. Iniciar con PM2
pm2 start dist/app.js --name web-services-pagos
pm2 save
pm2 startup
```

### Opción B: Distribución sin código fuente

```bash
# En tu máquina: compilar y empaquetar
npm run build && npm run build:admin
tar -czf web-services-pagos-prod.tar.gz dist/ admin-ui/dist/ package.json package-lock.json

# En el servidor: descomprimir e instalar solo deps de runtime
tar -xzf web-services-pagos-prod.tar.gz -C /opt/web-services-pagos
cd /opt/web-services-pagos
npm install --omit=dev
pm2 start dist/app.js --name web-services-pagos
```

### Nginx (reverse proxy)

```nginx
server {
    listen 443 ssl http2;
    server_name api.recaudaciones.tudominio.com;

    ssl_certificate /etc/letsencrypt/live/tudominio/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/tudominio/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

---

## 6. Panel de Administración

Accesible en `https://tudominio.com/admin`.

### Funcionalidades

- **Dashboard**: estadísticas del día, contadores y últimas transacciones
- **Productos**: alta, edición, activar/desactivar
- **Recaudadoras**: alta, edición, asignación de productos
- **Reglas de comisión**: tarifas por recaudadora y producto
- **Operaciones**: consultar deudas, registrar y revertir pagos manualmente
- **Transacciones**: historial con impresión de comprobante PDF
- **Reportes**: reporte de recaudación por período en PDF
- **Query Log**: auditoría completa de requests

---

## 7. Monitoreo y Logs

```bash
# Logs en tiempo real
pm2 logs web-services-pagos

# Métricas CPU/memoria
pm2 monit
```

---

## 8. Resolución de Problemas

| Error | Origen | Solución |
| --- | --- | --- |
| `InvalidSignature` | Middleware HMAC | Verificar `secret_key` y el orden estricto `TID+PRD_ID+SUB_ID+SECRET_KEY` sin espacios. |
| `SubscriberWithoutDebt` | DB sin filas | El abonado no existe o no tiene deudas pendientes. |
| `inscobranza` falla | Constraint PL/pgSQL | Revisar logs de Postgres. Puede ser FK faltante o estado incorrecto. |
| Admin UI devuelve 401 | JWT expirado | Verificar variables `ADMIN_USER`, `ADMIN_PASSWORD`, `ADMIN_JWT_SECRET` en `.env`. |
| `db:migrate` no crea tabla | Error de conexión | Verificar `DATABASE_URL` y permisos del usuario de DB. |
