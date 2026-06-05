---
title: Plan de Curso
nav_order: 5
---

# Plan de Curso — Desarrollo de un Web Service de Pagos (Recaudadoras)

**De nivel inicial a intermedio: construir y entender el proyecto desde la raíz**

---

## 1. Ficha técnica del curso

| Dato | Detalle |
| --- | --- |
| **Nombre** | Desarrollo de Web Services de Pagos con Node.js, TypeScript y PostgreSQL |
| **Carga horaria total** | 60 horas |
| **Modalidad de cursada** | Martes y jueves, de 18:30 a 21:30 (3 h por clase) |
| **Duración** | 10 semanas (20 clases) |
| **Fecha de inicio** | Martes 23/06/2026 |
| **Fecha de cierre** | Jueves 27/08/2026 |
| **Nivel** | Inicial → Intermedio |
| **Proyecto integrador** | El propio web service de recaudadoras + su panel de administración |

> **Objetivo general:** que al finalizar, el participante pueda entender, mantener, extender y desplegar desde cero un web service de pagos real, comprendiendo cada capa (dominio, aplicación, infraestructura, presentación), la base de datos legada, la seguridad por firmas HMAC, el panel de administración en React y el despliegue en producción.

---

## 2. Perfil de ingreso y egreso

### Requisitos previos (nivel inicial)
- Manejo básico de computadora y navegación de archivos.
- Lógica de programación elemental (variables, condicionales, bucles) — **se repasa** en el módulo 1, no se asume dominio.
- No se requiere experiencia previa en Node.js, TypeScript ni bases de datos.

### Al egresar (nivel intermedio) el participante podrá:
1. Escribir TypeScript con tipado estático, POO e interfaces.
2. Construir una API REST con Express 5 (rutas, middlewares, manejo de errores).
3. Modelar y consultar bases de datos PostgreSQL, incluyendo funciones PL/pgSQL y transacciones.
4. Aplicar **Arquitectura Limpia**: separar dominio, casos de uso, repositorios y controladores.
5. Implementar seguridad: firmas HMAC SHA-512 y autenticación JWT.
6. Escribir pruebas unitarias y de integración con Jest y Supertest.
7. Construir un panel de administración con React + Vite + TanStack Query.
8. Documentar con Swagger y desplegar en un VPS con PM2 y Nginx.

---

## 3. Stack tecnológico que se enseña (extraído del proyecto real)

- **Backend:** Node.js, TypeScript, Express 5
- **Base de datos:** PostgreSQL, driver `pg`, Drizzle ORM/Kit, funciones PL/pgSQL legadas (`inscobranza`, `insitemcobranza`, `cerrarcobranza`, `anucobranza`)
- **Validación y config:** Zod, dotenv
- **Seguridad:** HMAC SHA-512 (`crypto`), JWT (`jsonwebtoken`)
- **Logging:** Winston + Morgan
- **Documentación:** Swagger UI
- **Testing:** Jest, ts-jest, Supertest
- **Calidad:** ESLint, Prettier
- **Frontend (admin-ui):** React 19, Vite, Tailwind CSS, TanStack Query, React Router, i18next, react-pdf
- **DevOps:** Git, PM2, Nginx, Let's Encrypt

---

## 4. Metodología

- **20% teoría / 80% práctica.** Cada concepto se baja inmediatamente a código sobre el proyecto real.
- Cada clase incluye: repaso (10 min) → teoría guiada → laboratorio práctico → tarea para casa.
- Se construye **una versión propia del proyecto** en paralelo, archivo por archivo, hasta replicar la arquitectura real.
- Uso del repositorio existente como “solución de referencia” para comparar.

---

## 5. Cronograma general (20 clases / 60 horas)

| # | Fecha | Módulo | Tema central |
| --- | --- | --- | --- |
| 1 | Mar 23/06 | M0 | Presentación, entorno y panorama del proyecto |
| 2 | Jue 25/06 | M1 | JavaScript moderno I — fundamentos |
| 3 | Mar 30/06 | M1 | JavaScript moderno II — asincronía y módulos |
| 4 | Jue 02/07 | M2 | TypeScript I — tipos e interfaces |
| 5 | Mar 07/07 | M2 | TypeScript II — POO, clases y genéricos |
| 6 | Jue 09/07 | M3 | Node.js, npm y estructura de un proyecto |
| 7 | Mar 14/07 | M4 | Express I — servidor, rutas y middlewares |
| 8 | Jue 16/07 | M4 | Express II — routers, errores y validación con Zod |
| 9 | Mar 21/07 | M5 | Bases de datos relacionales y SQL I |
| 10 | Jue 23/07 | M5 | PostgreSQL II — joins, transacciones, PL/pgSQL y driver `pg` |
| 11 | Mar 28/07 | M6 | Configuración, entorno seguro, logging y Drizzle |
| 12 | Jue 30/07 | M7 | Arquitectura Limpia — capas y dependencias |
| 13 | Mar 04/08 | M7 | Dominio del proyecto — recaudadoras, deudas y comisiones |
| 14 | Jue 06/08 | M8 | Caso de uso: consulta de deudas (GetInvoices) |
| 15 | Mar 11/08 | M8 | Casos de uso: procesar pago y reversa |
| 16 | Jue 13/08 | M9 | Seguridad: firmas HMAC SHA-512 y JWT |
| 17 | Mar 18/08 | M10 | Testing con Jest y Supertest |
| 18 | Jue 20/08 | M11 | Panel admin I — React + Vite + Tailwind |
| 19 | Mar 25/08 | M11 | Panel admin II — TanStack Query, API y reportes PDF |
| 20 | Jue 27/08 | M12 | Swagger, build y despliegue (PM2 + Nginx) + cierre |

---

## 6. Desarrollo detallado de cada clase

### 🟦 Módulo 0 — Fundamentos y entorno

#### Clase 1 — Martes 23/06 — Presentación, entorno y panorama del proyecto
**Objetivos:** entender qué es un web service de pagos y preparar el entorno.
- Qué es una recaudadora / pasarela de pagos (Bancard, Pago Express, Infonet) y qué rol cumple el “facturador”.
- Recorrido funcional del proyecto: consultar deudas → pagar → revertir → administrar.
- Instalación: Node.js LTS, Git, Visual Studio Code, extensiones, PostgreSQL.
- Terminal y línea de comandos básica. Introducción a Git (clone, add, commit, push).
- **Práctica:** clonar el repositorio de referencia, instalar dependencias (`npm install`), levantar `npm run dev` y abrir `/health` y `/admin`.
- **Tarea:** configurar cuenta de GitHub y subir un repositorio “hola-mundo”.

---

### 🟦 Módulo 1 — JavaScript moderno

#### Clase 2 — Jueves 25/06 — JavaScript moderno I (fundamentos)
**Objetivos:** dominar la base del lenguaje.
- Variables (`let`, `const`), tipos primitivos, operadores.
- Condicionales, bucles, funciones, arrow functions.
- Objetos y arrays; métodos clave (`map`, `filter`, `reduce`, `find`).
- Desestructuración y spread/rest.
- **Práctica:** mini-ejercicios de transformación de datos (listas de “deudas” simuladas).
- **Tarea:** resolver 10 katas de arrays/objetos.

#### Clase 3 — Martes 30/06 — JavaScript moderno II (asincronía y módulos)
**Objetivos:** entender el modelo asíncrono que usa todo Node.
- Callbacks, Promesas, `async`/`await`, manejo de errores con `try/catch`.
- El Event Loop explicado de forma simple.
- Módulos: `import`/`export` (ESM) vs `require` (CommonJS).
- JSON: serializar y parsear.
- **Práctica:** simular llamadas asíncronas a una “API de deudas” con `setTimeout`/`fetch`.
- **Tarea:** convertir un conjunto de funciones con callbacks a `async/await`.

---

### 🟦 Módulo 2 — TypeScript

#### Clase 4 — Jueves 02/07 — TypeScript I (tipos e interfaces)
**Objetivos:** comprender por qué el proyecto usa TypeScript.
- Tipos básicos, `union`, `literal`, `enum`, `type` vs `interface`.
- Tipado de funciones, parámetros opcionales y por defecto.
- Tipos en arrays y objetos; `unknown`, `any`, `never`.
- `tsconfig.json` y compilación (`tsc`).
- **Práctica:** tipar las entidades del dominio (`Collector`, `Deuda`, `Transaction`) inspirándose en `src/domain/entities/types.ts`.
- **Tarea:** modelar con interfaces una factura con sus ítems.

#### Clase 5 — Martes 07/07 — TypeScript II (POO, clases y genéricos)
**Objetivos:** la POO que sostiene la arquitectura del proyecto.
- Clases, constructores, modificadores (`public`, `private`, `readonly`).
- Herencia, `implements` de interfaces, clases abstractas.
- Genéricos (`<T>`) y utilidad práctica.
- Value Objects: ejemplo real `Money.ts` y `Signature.ts`.
- **Práctica:** implementar una clase `Money` que valide montos y evite errores de redondeo.
- **Tarea:** crear una interfaz `IRepository<T>` genérica.

---

### 🟦 Módulo 3 — Node.js y el proyecto

#### Clase 6 — Jueves 09/07 — Node.js, npm y estructura de un proyecto
**Objetivos:** entender el runtime y el ecosistema.
- Qué es Node.js; diferencias con el navegador.
- `npm`/`package.json`: dependencias, devDependencies, scripts.
- `node_modules`, versionado semántico, `package-lock.json`.
- Scripts del proyecto real (`dev`, `build`, `start`, `db:init`, `seed`, `test`).
- Módulos nativos: `fs`, `path`, `crypto`, `http`.
- **Práctica:** crear un proyecto Node desde cero con TypeScript y `ts-node-dev`.
- **Tarea:** escribir un script que lea un archivo y cuente líneas con `fs`.

---

### 🟦 Módulo 4 — API con Express

#### Clase 7 — Martes 14/07 — Express I (servidor, rutas y middlewares)
**Objetivos:** levantar una API REST.
- HTTP: métodos, status codes, headers, body, query params.
- Crear un servidor Express; `req`/`res`.
- Rutas `GET`/`POST`; parámetros de ruta y query.
- Concepto de middleware; `express.json()`, `morgan`.
- **Práctica:** reconstruir `src/app.ts` paso a paso (health check, JSON, logging).
- **Tarea:** crear endpoints CRUD en memoria para “productos”.

#### Clase 8 — Jueves 16/07 — Express II (routers, errores y validación con Zod)
**Objetivos:** organizar y blindar la API.
- Routers modulares (patrón `createInvoiceRoutes`, `createPaymentRoutes`).
- Middleware de manejo de errores centralizado (`errorHandler`).
- Validación de entrada con **Zod**.
- Patrón Controller (separar HTTP de la lógica).
- **Práctica:** crear rutas modulares + un `errorHandler` global + validar el body con Zod.
- **Tarea:** validar con Zod los parámetros de una consulta de deudas.

---

### 🟦 Módulo 5 — Base de datos

#### Clase 9 — Martes 21/07 — Bases de datos relacionales y SQL I
**Objetivos:** fundamentos de datos.
- Modelo relacional: tablas, columnas, claves primarias y foráneas.
- SQL: `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `WHERE`, `ORDER BY`.
- Tipos de datos, `NULL`, restricciones.
- Recorrido del esquema del proyecto (`collectors`, `transactions`, `query_logs`, `commission_rules`, `products`).
- **Práctica:** crear tablas y poblarlas en PostgreSQL (pgAdmin / psql).
- **Tarea:** escribir 8 consultas SELECT sobre el esquema dado.

#### Clase 10 — Jueves 23/07 — PostgreSQL II (joins, transacciones, PL/pgSQL y driver `pg`)
**Objetivos:** consultas avanzadas y conexión desde Node.
- `JOIN` (inner/left), agregaciones (`COUNT`, `SUM`, `GROUP BY`).
- Transacciones (`BEGIN`/`COMMIT`/`ROLLBACK`) y por qué importan en pagos.
- Funciones PL/pgSQL: las legadas `inscobranza`, `insitemcobranza`, `cerrarcobranza`, `anucobranza`.
- Conexión desde Node con el driver `pg` (pool, queries parametrizadas, anti SQL injection).
- **Práctica:** consultar la BD desde Node con `pg`, usando parámetros.
- **Tarea:** escribir una consulta con JOIN que liste pagos por recaudadora.

#### Clase 11 — Martes 28/07 — Configuración, entorno seguro, logging y Drizzle
**Objetivos:** preparar la base de infraestructura.
- Variables de entorno con `.env` y validación estricta con **Zod** (`src/config/env.ts`).
- Logging estructurado con **Winston** (`src/config/logger.ts`) y Morgan.
- Introducción a **Drizzle ORM/Kit**: schema, `drizzle:generate`, migraciones.
- Scripts `db:init`, `db:migrate`, `seed` del proyecto.
- **Práctica:** crear `env.ts` validado y `logger.ts`; correr una migración.
- **Tarea:** agregar una variable de entorno nueva validada con Zod.

---

### 🟦 Módulo 6/7 — Arquitectura Limpia y dominio

#### Clase 12 — Jueves 30/07 — Arquitectura Limpia: capas y dependencias
**Objetivos:** comprender el corazón estructural del proyecto.
- Las 4 capas: `domain` → `application` → `infrastructure` → `interfaces`.
- Regla de dependencia (siempre hacia adentro) y por qué.
- Interfaces de repositorio (`IDeudaRepository`, `ICollectorRepository`) e inversión de dependencias.
- Inyección de dependencias manual (composición raíz en `src/app.ts`).
- **Práctica:** diagramar y recorrer el flujo de una petición a través de las 4 capas.
- **Tarea:** explicar por escrito por qué el dominio no debe conocer a PostgreSQL.

#### Clase 13 — Martes 04/08 — Dominio del proyecto: recaudadoras, deudas y comisiones
**Objetivos:** entender el negocio.
- Entidades del dominio (`types.ts`): Collector, Deuda, Transaction, CommissionRule.
- Productos: `prd_id=1` (cuotas) vs `prd_id=2` (deudas varias).
- Reglas de comisión: fija vs porcentual por recaudadora/producto.
- Adaptadores de recaudadoras (`BancardAdapter`, `CollectorAdapterFactory`) — patrón Adapter/Factory.
- **Práctica:** implementar `CalculateCommissionUseCase` y sus reglas.
- **Tarea:** agregar una nueva regla de comisión y probarla.

#### Clase 14 — Jueves 06/08 — Caso de uso: consulta de deudas (GetInvoices)
**Objetivos:** primer flujo completo de punta a punta.
- `GetInvoicesUseCase`: orquestación de repos + cálculo de comisión.
- Repositorio `PostgresDeudaRepository`: consulta a la BD legada.
- Controlador `InvoiceController` y ruta `GET /invoices`.
- Auditoría: registro en `query_logs`.
- **Práctica:** implementar el flujo completo de consulta de deudas y probarlo con Postman.
- **Tarea:** extender la respuesta con un campo calculado.

#### Clase 15 — Martes 11/08 — Casos de uso: procesar pago y reversa
**Objetivos:** el flujo crítico del negocio.
- `ProcessPaymentUseCase`: cálculo de comisión + `inscobranza`/`insitemcobranza`/`cerrarcobranza` de forma atómica.
- `ReversePaymentUseCase`: anulación con `anucobranza()`.
- Idempotencia, manejo de errores de dominio (`DomainError`), transacciones.
- Endpoints `POST /payment` y `POST /reverse`.
- **Práctica:** implementar pago y reversa, verificando el cierre en la BD.
- **Tarea:** documentar el flujo de reversa paso a paso.

---

### 🟦 Módulo 8 — Seguridad

#### Clase 16 — Jueves 13/08 — Seguridad: firmas HMAC SHA-512 y JWT
**Objetivos:** proteger la API.
- Por qué se firman las peticiones; integridad vs confidencialidad.
- HMAC SHA-512 con `crypto`: firma de `TID + PRD_ID + SUB_ID + SECRET_KEY` (`Signature.ts`).
- Middleware de autenticación de recaudadoras (`authMiddleware`, header `X-Signature`).
- JWT para el panel admin: login, `adminAuthMiddleware`, expiración y secreto.
- **Práctica:** implementar verificación de firma y un login JWT.
- **Tarea:** generar y verificar una firma desde un script cliente.

---

### 🟦 Módulo 9 — Testing

#### Clase 17 — Martes 18/08 — Testing con Jest y Supertest
**Objetivos:** asegurar calidad y evitar regresiones.
- Pirámide de testing: unitario vs integración.
- Jest + ts-jest: `describe`, `it`, `expect`, mocks.
- Pruebas unitarias de casos de uso (`ProcessPaymentUseCase.test.ts`, `CalculateCommissionUseCase.test.ts`).
- Pruebas de integración de la API con **Supertest** (`tests/integration/api.test.ts`).
- **Práctica:** escribir tests unitarios para el cálculo de comisión y uno de integración para `/invoices`.
- **Tarea:** agregar tests al caso de uso de reversa.

---

### 🟦 Módulo 10 — Panel de administración (Frontend)

#### Clase 18 — Jueves 20/08 — Panel admin I: React + Vite + Tailwind
**Objetivos:** introducción al frontend del proyecto.
- Qué es una SPA; React: componentes, props, estado (`useState`, `useEffect`).
- Vite: dev server y build; estructura de `admin-ui/`.
- Tailwind CSS y componentes UI reutilizables (`Button`, `Input`, `Modal`).
- Routing con React Router y rutas protegidas (`ProtectedRoute`).
- **Práctica:** crear la página de Login y el Layout del panel.
- **Tarea:** crear un componente de tabla reutilizable.

#### Clase 19 — Martes 25/08 — Panel admin II: TanStack Query, API y reportes PDF
**Objetivos:** conectar el frontend con el backend.
- Cliente HTTP con Axios (`lib/api.ts`) y manejo del token JWT.
- TanStack Query: `useQuery`/`useMutation`, caché e invalidación.
- Páginas reales: Dashboard, Transacciones, Reportes, Recaudadoras.
- Internacionalización (i18next) y generación de PDF (react-pdf: comprobantes y reporte de recaudación).
- **Práctica:** consumir `/admin/stats` y mostrar el Dashboard con datos reales.
- **Tarea:** agregar un filtro por recaudadora a una página.

---

### 🟦 Módulo 11 — Documentación y despliegue

#### Clase 20 — Jueves 27/08 — Swagger, build y despliegue (PM2 + Nginx) + cierre
**Objetivos:** publicar el servicio y cerrar el curso.
- Documentación de la API con **Swagger UI** (`/api-docs`, `swagger.json`).
- Build de producción: `build:all` (backend + admin-ui) y `npm start`.
- Despliegue en VPS: Node, **PM2** (proceso persistente), **Nginx** como proxy inverso, HTTPS con **Let's Encrypt**.
- Variables de entorno de producción y buenas prácticas operativas.
- **Práctica:** build completo + simulación de despliegue con PM2 + proxy local.
- **Cierre:** presentación de proyectos finales y evaluación integradora.

---

## 7. Evaluación

| Instrumento | Peso | Descripción |
| --- | --- | --- |
| Tareas y laboratorios | 30% | Entregas de cada clase. |
| Evaluación parcial | 20% | Al cierre del Módulo 5 (clase 11): API + base de datos. |
| Proyecto integrador | 40% | Un endpoint nuevo completo (dominio→infra→interfaz) con tests + una página en el admin. |
| Participación y asistencia | 10% | Mínimo 70% de asistencia para aprobar. |

**Aprobación:** 60/100 puntos.

---

## 8. Proyecto integrador (entrega final)

Cada participante debe agregar al sistema una **funcionalidad nueva de punta a punta**, por ejemplo: un nuevo tipo de producto con su regla de comisión, su consulta, su pago, sus tests unitarios y de integración, y su gestión desde el panel admin. Se evalúa: respeto a la arquitectura, validación, seguridad (firma), pruebas y documentación.

---

## 9. Recursos y bibliografía

- Documentación oficial: [Node.js](https://nodejs.org/docs), [TypeScript](https://www.typescriptlang.org/docs/), [Express](https://expressjs.com/), [PostgreSQL](https://www.postgresql.org/docs/), [Drizzle](https://orm.drizzle.team/), [Zod](https://zod.dev/), [React](https://react.dev/), [Vite](https://vite.dev/), [TanStack Query](https://tanstack.com/query).
- Guías de este sitio: [Guía de Integración](integration), [Guía de Configuración](configuration), [Manual de Operaciones](deployment).
- Colección Postman: `bancard-api.postman_collection.json`.
- Repositorio del proyecto como solución de referencia.

---

## 10. Materiales y requisitos de aula

- Una computadora por participante (mínimo 8 GB RAM).
- Software: Node.js LTS, Git, VS Code, PostgreSQL, navegador moderno, Postman.
- Conexión a internet, proyector y acceso al repositorio.

---

*Plan distribuido en 20 clases de 3 horas (60 horas), martes y jueves de 18:30 a 21:30, del 23/06/2026 al 27/08/2026.*
