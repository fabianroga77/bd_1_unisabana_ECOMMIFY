# Evidencias de Rendimiento — PostgreSQL (Supabase)
**Ecommify | Unidad 5 — Optimización de Rendimiento**

> ⚠️ **Instrucciones:** Reemplazar las tablas de ejemplo con los valores reales obtenidos al ejecutar `EXPLAIN (ANALYZE, BUFFERS)` en Supabase. Los valores aquí son **plantilla** para documentar el proceso.

---

## Metodología de medición

1. Ejecutar query **sin** el índice (`DROP INDEX nombre_indice;`)
2. Ejecutar `EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT) <query>`
3. Registrar `Execution Time`, `Planning Time`, `Rows Removed`, nodos usados
4. Crear el índice (`CREATE INDEX ...`)
5. Ejecutar `ANALYZE <tabla>;` para actualizar estadísticas
6. Repetir `EXPLAIN ANALYZE` y registrar resultados

---

## Q1 — Pedidos entregados en 2018

**Query:**
```sql
SELECT order_id, customer_id, order_purchase_timestamp
FROM orders_v2
WHERE order_status = 'delivered'
  AND order_purchase_timestamp BETWEEN '2018-01-01' AND '2018-12-31';
```

| Métrica | Sin índice | Con índice `idx_orders_v2_status_purchase` | Mejora |
|---|---|---|---|
| Execution Time | ___ ms | ___ ms | ___% |
| Planning Time | ___ ms | ___ ms | — |
| Nodo de acceso | Seq Scan | Index Scan / Bitmap Index Scan | ✅ |
| Rows removed by filter | ___ | ___ | — |
| Buffers hit | ___ | ___ | — |
| Shared blocks read | ___ | ___ | ✅ |

**Captura EXPLAIN (pegar aquí):**
```
-- ANTES (sin índice):
[Pegar salida de EXPLAIN ANALYZE aquí]

-- DESPUÉS (con índice):
[Pegar salida de EXPLAIN ANALYZE aquí]
```

---

## Q2 — Productos por especificaciones JSONB

**Query:**
```sql
SELECT product_id, product_category_name
FROM products_v2
WHERE (product_specifications->>'weight_g')::int > 500;
```

| Métrica | Sin índice GIN | Con índice GIN `idx_products_v2_specifications` | Mejora |
|---|---|---|---|
| Execution Time | ___ ms | ___ ms | ___% |
| Nodo de acceso | Seq Scan | Bitmap Index Scan on GIN | ✅ |
| Rows estimated | ___ | ___ | — |

---

## Q3 — Revenue por vendedor (Covering Index)

**Query:**
```sql
SELECT seller_id, SUM(price), SUM(freight_value), COUNT(*)
FROM order_items
WHERE seller_id = '3442f8959a84dea7ee197c632cb2df15'
GROUP BY seller_id;
```

| Métrica | Sin covering index | Con `idx_order_items_seller_revenue_covering` | Mejora |
|---|---|---|---|
| Execution Time | ___ ms | ___ ms | ___% |
| Heap Fetches | ___ | 0 (Index-Only Scan) | ✅ |
| Nodo de acceso | Seq Scan / Index Scan | Index Only Scan | ✅ |

---

## Q4 — Búsqueda fuzzy de categoría (pg_trgm)

**Query:**
```sql
SELECT DISTINCT product_category_name
FROM products_v2
WHERE product_category_name % 'beleza';
```

| Métrica | Sin índice trigram | Con `idx_products_v2_category_trgm` | Mejora |
|---|---|---|---|
| Execution Time | ___ ms | ___ ms | ___% |
| Nodo de acceso | Seq Scan | Bitmap Index Scan on GIN trgm | ✅ |

---

## Resumen global PostgreSQL

| Query | Tiempo antes (ms) | Tiempo después (ms) | Reducción |
|---|---|---|---|
| Q1 — Pedidos entregados 2018 | | | |
| Q2 — Productos por JSONB peso | | | |
| Q3 — Revenue por vendedor | | | |
| Q4 — Búsqueda fuzzy categoría | | | |

> 📸 **Capturas de pantalla:** Agregar imágenes de Supabase Query Editor mostrando los planes de ejecución en `/evidencias/postgresql/capturas/`
