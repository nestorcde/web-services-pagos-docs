---
title: Guía de Configuración
nav_order: 3
---

# Guía de Configuración de Entidades Recaudadoras

Este documento detalla cómo configurar el sistema para habilitar nuevas **Entidades Recaudadoras** (como Bancard, Infonet, PagoExpress, etc.), gestionar productos y definir reglas de comisión.

Toda la configuración puede realizarse desde el **Panel de Administración** (`/admin`) o directamente via SQL en la base de datos PostgreSQL.

---

## 1. Tabla `products`

Define los tipos de cobro disponibles. Cada recaudadora se autoriza a operar uno o más productos.

### Campos

| Campo | Tipo | Descripción |
| --- | --- | --- |
| `id` | `serial` (PK) | Identificador del producto (`prd_id`). |
| `name` | `text` | Nombre descriptivo. Ej: `'Cuotas académicas'`. |
| `description` | `text` | Descripción opcional del producto. |
| `is_active` | `boolean` | Habilita/deshabilita el producto. |

### Productos iniciales

| id | name | Fuente en DB |
| --- | --- | --- |
| `1` | Cuotas académicas | Vista `cuota_pagar` |
| `2` | Deudas varias | Vista `deuda` / tabla `i1_deuda` |

Para agregar nuevos productos, usar el panel `/admin/products` o:

```sql
INSERT INTO products (name, description) VALUES ('Nuevo producto', 'Descripción');
```

---

## 2. Tabla `collectors`

Define las credenciales y comportamiento de cada entidad recaudadora.

### Campos

| Campo | Tipo | Descripción | Ejemplo |
| --- | --- | --- | --- |
| `id` | `serial` (PK) | Identificador interno único (autogenerado). | `1` |
| `name` | `text` | Nombre de la recaudadora. | `'Bancard'` |
| `secret_key` | `text` | Clave secreta para validar firmas `X-Signature` HMAC SHA-512. **Nunca viaja en requests.** | `'clave_larga...'` |
| `product_ids` | `integer[]` | IDs de productos autorizados (referencia a `products.id`). | `[1, 2]` |
| `add_commission_to_debt` | `boolean` | `true`: la comisión se suma al monto y la absorbe el cliente. `false`: comisión interna, no altera el monto. | `true` |
| `cobranza_usuario_id` | `integer` | ID del usuario/cajero en el sistema legado al que se asignan las cobranzas del canal digital. | `1` |
| `is_active` | `boolean` | `false` → HTTP 401 en todos los requests. | `true` |

### Ejemplo SQL

```sql
INSERT INTO collectors (
  name, secret_key, product_ids, add_commission_to_debt, cobranza_usuario_id, is_active
) VALUES (
  'PagoExpress',
  'mi_clave_secreta_super_segura_pagoexpress',
  ARRAY[1, 2],
  true,
  2,
  true
);
```

---

## 3. Tabla `commission_rules`

Define las tarifas de comisión por recaudadora y producto, con soporte para rangos de monto.

### Campos

| Campo | Tipo | Descripción | Ejemplo |
| --- | --- | --- | --- |
| `id` | `serial` (PK) | Identificador único. | `1` |
| `collector_id` | `integer` | FK a `collectors.id`. | `1` |
| `product_id` | `integer` | FK a `products.id`. | `1` |
| `range_from` | `integer` | Monto mínimo (inclusivo) para que aplique la regla. | `0` |
| `range_to` | `integer` | Monto máximo (inclusivo). `NULL` = sin tope superior. | `100000` |
| `calc_type` | `enum` | `'FIXED'` (monto fijo) o `'PERCENTAGE'` (%). | `'FIXED'` |
| `value` | `numeric` | Valor de la comisión. Si es `FIXED`: guaraníes. Si es `PERCENTAGE`: porcentaje. | `5000` / `2.5` |
| `min_commission` | `integer` | Tope mínimo en Gs. Útil para % que resultarían en montos muy bajos. | `3000` |
| `max_commission` | `integer` | Tope máximo en Gs. `NULL` = sin límite. | `20000` |
| `is_active` | `boolean` | Reglas inactivas no se evalúan. | `true` |

### Flujo de Cálculo

Cuando la API procesa `GET /invoices` o `POST /payment`:

1. Identifica el `collector_id` y `prd_id` del request.
2. Busca reglas activas para esa combinación.
3. Evalúa en qué rango (`range_from`–`range_to`) cae el monto de la deuda.
4. Aplica el cálculo (`PERCENTAGE` o `FIXED`), respetando `min_commission` y `max_commission`.

### Ejemplos SQL

#### Tarifa plana

```sql
INSERT INTO commission_rules (
  collector_id, product_id, range_from, range_to,
  calc_type, value, min_commission, is_active
) VALUES (
  1, 1, 0, NULL,
  'FIXED', 5000, 5000, true
);
```

#### Comisión porcentual con topes

```sql
INSERT INTO commission_rules (
  collector_id, product_id, range_from, range_to,
  calc_type, value, min_commission, max_commission, is_active
) VALUES (
  1, 2, 0, NULL,
  'PERCENTAGE', 2.5, 3000, 25000, true
);
```

#### Comisión por escalas

```sql
-- Hasta 100.000 Gs → tarifa fija de 3.000 Gs
INSERT INTO commission_rules (
  collector_id, product_id, range_from, range_to,
  calc_type, value, min_commission, max_commission, is_active
) VALUES (1, 1, 0, 100000, 'FIXED', 3000, 3000, 3000, true);

-- Más de 100.000 Gs → 3%
INSERT INTO commission_rules (
  collector_id, product_id, range_from, range_to,
  calc_type, value, min_commission, max_commission, is_active
) VALUES (1, 1, 100001, NULL, 'PERCENTAGE', 3.0, 3000, NULL, true);
```

---

## Resumen: Integración de una nueva Recaudadora

1. Crear el producto en `/admin/products` si aún no existe.
2. Generar un `secret_key` largo y aleatorio (mínimo 64 caracteres).
3. Dar de alta la recaudadora en `/admin/collectors` asignando los `product_ids` autorizados.
4. Definir las reglas de comisión en `/admin/commission-rules` para cada producto habilitado.
5. Compartir el `secret_key` y la documentación de la API HTTP con la entidad para que implementen la firma `X-Signature`.
