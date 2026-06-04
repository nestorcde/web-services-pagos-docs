---
title: Inicio
nav_order: 1
---

# Web Service de Recaudaciones

Documentación técnica del sistema de pagos y recaudaciones.

---

## Documentos disponibles

| Documento | Descripción |
| --- | --- |
| [Guía de Integración](integration) | Endpoints, autenticación y ejemplos de código para entidades recaudadoras |
| [Guía de Configuración](configuration) | Alta de recaudadoras, productos y reglas de comisión |
| [Manual de Operaciones](deployment) | Instalación, deploy, monitoreo y resolución de problemas |

---

## Inicio rápido

Para comenzar la integración necesitás:

1. Solicitar tus credenciales: **Product ID** (`prd_id`) y **Clave Secreta** (`secret_key`)
2. Leer la [Guía de Integración](integration)
3. Implementar la firma `X-Signature` (SHA-512)
4. Probar contra el entorno de staging antes de pasar a producción
