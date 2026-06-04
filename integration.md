---
title: Guía de Integración
nav_order: 2
---

# Guía de Integración API - Web Service Recaudadoras

Bienvenido a la Guía de Integración Oficial para Entidades Recaudadoras. Esta documentación provee todo lo necesario para consumir los servicios de consulta, pago y anulación de deudas mediante nuestra API RESTful.

---

## 1. Información General

- **Arquitectura:** RESTful API
- **Formato de datos:** JSON (`application/json`)
- **Codificación:** UTF-8
- **Idempotencia:** garantizada mediante validación de IDs de transacción (`tid`)
- **Entornos:**
  - Test / Staging: `https://recaudaciones-staging.tudominio.com`
  - Producción: `https://api.recaudaciones.tudominio.com`

> **IMPORTANTE:** Para operar, su Entidad Recaudadora recibirá un **Product ID** (`prd_id`) y una **Clave Secreta** (`secret_key`). La clave secreta **nunca** viaja en las peticiones.

---

## 2. Autenticación (Firma Criptográfica)

Todos los endpoints requieren autenticación mediante el header HTTP `X-Signature`. Esta firma garantiza la integridad del mensaje y autentica a la recaudadora.

### Generación de la firma

La firma es el **hash SHA-512** del siguiente string concatenado **sin espacios ni separadores**:

```text
TID + PRD_ID + SUB_ID + SECRET_KEY
```

Donde:

- `TID`: Número de transacción único generado por la recaudadora.
- `PRD_ID`: Identificador de producto proporcionado en sus credenciales.
- `SUB_ID`: Cédula o identificador del abonado.
- `SECRET_KEY`: Su clave secreta (nunca viaja en la petición).

### Ejemplos de código

#### Node.js / TypeScript

```typescript
import { createHash } from 'crypto';

function generateSignature(tid: string, prdId: string, subId: string, secretKey: string): string {
  return createHash('sha512').update(`${tid}${prdId}${subId}${secretKey}`).digest('hex');
}
```

#### Python

```python
import hashlib

def generate_signature(tid, prd_id, sub_id, secret_key):
    raw = f"{tid}{prd_id}{sub_id}{secret_key}"
    return hashlib.sha512(raw.encode('utf-8')).hexdigest()
```

#### PHP

```php
function generate_signature($tid, $prd_id, $sub_id, $secret_key) {
    return hash('sha512', $tid . $prd_id . $sub_id . $secret_key);
}
```

---

## 3. Endpoints Disponibles

### 3.1 Estado del Servicio (Health Check)

Permite verificar si el servicio está operativo. **No requiere firma.**

- **Ruta:** `GET /health`

#### Respuesta de health check (200 OK)

```json
{
  "status": "success",
  "timestamp": "2026-03-22T20:15:00.000Z"
}
```

---

### 3.2 Consulta de Deuda

Obtiene las deudas o cuotas pendientes de un abonado.

- **Ruta:** `GET /invoices`
- **Headers:** `X-Signature: <firma>`

| Parámetro | Tipo | Requerido | Descripción |
| --- | --- | --- | --- |
| `prd_id` | Integer | Sí | Identificador del producto (`1` = cuotas académicas, `2` = deudas varias). |
| `sub_id` | String | Sí | Cédula o identificador del abonado. |
| `tid` | BigInt | No | ID de trazabilidad de la consulta. |

**Respuesta exitosa — `prd_id=1` (cuotas académicas)**

```json
{
  "status": "success",
  "invoices": [
    {
      "inv_id": "202501:6:1-1",
      "dsc": "Cuota 1 - ANALISIS DE SISTEMAS - 202501",
      "amt": 80000,
      "curr": "PYG",
      "due": "2025-02-14"
    },
    {
      "inv_id": "202501:6:1-2",
      "dsc": "Cuotas 1 al 2 - ANALISIS DE SISTEMAS - 202501",
      "amt": 160000,
      "curr": "PYG",
      "due": "2025-03-14"
    }
  ]
}
```

> Las cuotas se devuelven acumuladas (cuota 1, cuotas 1-2, …, cuotas 1-N) para prevenir pagos salteados. Si el alumno cursa dos carreras, se devuelven grupos separados por carrera.

**Respuesta exitosa — `prd_id=2` (deudas varias)**

```json
{
  "status": "success",
  "invoices": [
    {
      "inv_id": "DEUDA:1042",
      "dsc": "Multa por mora. Geometría Analítica",
      "amt": 250000,
      "curr": "PYG",
      "due": "2025-01-15"
    }
  ]
}
```

> Cada deuda `i1_deuda` es una factura independiente con `inv_id` en formato `DEUDA:{id}`.

---

### 3.3 Pago de Deuda

Procesa un cobro en tiempo real.

- **Ruta:** `POST /payment`
- **Headers:** `X-Signature: <firma>`, `Content-Type: application/json`

#### Request de pago

```json
{
  "tid": 111122223333,
  "prd_id": 1,
  "sub_id": "4543390",
  "inv_id": "202501:6:1-1",
  "amt": 80000,
  "curr": "PYG",
  "trn_dat": "2026-03-22",
  "trn_hou": "15:30:00"
}
```

> Para `prd_id=2` el `inv_id` tiene formato `DEUDA:{id}`, por ejemplo `"inv_id": "DEUDA:1042"`.

#### Respuesta de pago exitoso (200 OK)

```json
{
  "status": "success",
  "tid": 111122223333,
  "tkt": 99876,
  "cm_amt": 1500,
  "prnt_msg": [
    "GRACIAS POR SU PAGO EN RECAUDADORA TUDOMINIO",
    "Ticket Operación: 99876."
  ],
  "messages": []
}
```

---

### 3.4 Reversión / Anulación de Pago

Anula un pago del mismo día.

- **Ruta:** `POST /reverse`
- **Headers:** `X-Signature: <firma>`, `Content-Type: application/json`

> **Regla crítica:** No se puede revertir una cuota inferior si ya existe una cuota superior pagada en el sistema.

#### Request de reversión

```json
{
  "tid": 444455556666,
  "original_tid": 111122223333,
  "prd_id": 1,
  "sub_id": "4543390",
  "inv_id": "202501:6:1-1",
  "amt": 80000,
  "curr": "PYG",
  "trn_dat": "2026-03-22",
  "trn_hou": "15:35:00",
  "original_trn_dat": "2026-03-22"
}
```

#### Respuesta de reversión exitosa (200 OK)

```json
{
  "status": "success",
  "tid": 444455556666,
  "messages": [
    {
      "level": "success",
      "key": "ReversalProcessed",
      "dsc": ["El reverso comercial ha sido procesado de manera exitosa en el core."]
    }
  ]
}
```

---

## 4. Códigos de Error

| HTTP | Key | Causa | Solución |
| --- | --- | --- | --- |
| `401` | `InvalidSignature` | Firma HMAC-SHA512 inválida. | Verificar `secret_key` y el orden estricto `TID+PRD_ID+SUB_ID+SECRET_KEY` sin espacios. |
| `403` | `SubscriberWithoutDebt` | El abonado no tiene deudas pendientes. | Informar al cliente que no registra deudas en el sistema. |
| `403` | `HigherCuotaAlreadyPaid` | Se intenta anular una cuota inferior cuando ya existe una superior pagada. | Rechazar la anulación por inconsistencia de período. |
| `409` | `UnknownCollector` | El `prd_id` no corresponde a ninguna recaudadora habilitada. | Verificar que el `prd_id` esté autorizado en las credenciales de la recaudadora. |
| `409` | `InvalidAmount` | El monto no coincide con la factura + comisión calculada. | Usar el `amt` devuelto exactamente por `GET /invoices`. |
| `409` | `TransactionNotFound` | Se intenta revertir un `original_tid` que nunca fue procesado. | No forzar reversiones de transacciones no registradas. |
| `500` | `QueryFailed` | Error de base de datos. | Reintentar con backoff exponencial (5 segundos). |

---

> **¿Dudas o solicitud de acceso Sandbox?**
> Contactar al equipo de integración para obtener credenciales de prueba con abonados pre-cargados.
