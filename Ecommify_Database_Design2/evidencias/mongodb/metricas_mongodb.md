# Evidencias de Rendimiento — MongoDB Atlas
**Ecommify | Unidad 5 — Optimización de Rendimiento**

> ⚠️ **Instrucciones:** Reemplazar los valores `___` con los resultados reales de `.explain("executionStats")` en MongoDB Atlas. Agregar capturas del Performance Advisor.

---

## Metodología de medición

```javascript
// Comando para obtener stats de ejecución
db.orders_reviews.explain("executionStats").find({ review_score: { $lte: 3 } });

// Métricas clave a registrar del output:
// - executionStats.executionTimeMillis
// - executionStats.totalDocsExamined
// - executionStats.totalKeysExamined
// - executionStats.nReturned
// - queryPlanner.winningPlan.stage  (COLLSCAN vs IXSCAN)
```

**Efficiency Ratio** = `nReturned / totalDocsExamined` (ideal: cercano a 1.0)

---

## M1 — Índice compuesto ESR: `idx_reviews_score_created_answered`

**Query de prueba:**
```javascript
db.orders_reviews.find({
  review_score: 5,
  review_creation_date: { $gte: "2018-01-01", $lte: "2018-06-30" }
}).sort({ review_creation_date: 1 });
```

| Métrica | Sin índice (COLLSCAN) | Con índice ESR (IXSCAN) | Mejora |
|---|---|---|---|
| executionTimeMillis | ___ ms | ___ ms | ___% |
| totalDocsExamined | ___ | ___ | ___% |
| totalKeysExamined | — | ___ | — |
| nReturned | ___ | ___ | = |
| Efficiency Ratio | ___ | ___ | ✅ |
| Stage ganador | COLLSCAN | IXSCAN | ✅ |

---

## M2 — Índice parcial: `idx_reviews_negative_partial`

**Query de prueba:**
```javascript
db.orders_reviews.find({
  review_score: { $lte: 3 },
  review_creation_date: { $gte: "2018-01-01" }
});
```

| Métrica | Sin índice parcial | Con índice parcial | Mejora |
|---|---|---|---|
| executionTimeMillis | ___ ms | ___ ms | ___% |
| totalDocsExamined | ___ | ___ | ___% |
| Tamaño del índice | N/A | ___ KB | — |
| Reducción vs índice completo | — | ~60% | ✅ |

---

## M3 — Full-Text Search: `idx_reviews_fulltext`

**Query de prueba:**
```javascript
db.orders_reviews.find({
  $text: { $search: "produto ruim entrega atrasada" }
}, {
  score: { $meta: "textScore" }
}).sort({ score: { $meta: "textScore" } });
```

| Métrica | Sin índice texto | Con índice texto | Mejora |
|---|---|---|---|
| executionTimeMillis | ___ ms | ___ ms | ___% |
| totalDocsExamined | ___ | ___ | ___% |
| Stage | COLLSCAN + filter | TEXT + IXSCAN | ✅ |

---

## M4 — Aggregation Pipeline: `01_pipeline_sales_analytics.js`

**Comando:**
```javascript
db.orders_reviews.explain("executionStats").aggregate([...pipeline...], { allowDiskUse: true });
```

| Stage | Docs entrada | Docs salida | Tiempo (ms) | Índice usado |
|---|---|---|---|---|
| $match | ~100,000 | ~45,000 | ___ | idx_reviews_score_created_answered |
| $lookup | ~45,000 | ~45,000 | ___ | — |
| $unwind | ~45,000 | ~45,000 | ___ | — |
| $addFields | ~45,000 | ~45,000 | ___ | — |
| $group | ~45,000 | ~500 | ___ | — |
| $facet | ~500 | 1 doc | ___ | — |
| $sort | 1 doc | 1 doc | ___ | — |
| **TOTAL** | | | **___ ms** | |

**Comparación pipeline no optimizado vs optimizado:**

| Versión | executionTimeMillis | docsExamined | Descripción |
|---|---|---|---|
| Sin optimizar | ___ ms | ~100,000 | $lookup sin filtro, $group antes de $match |
| Optimizado | ___ ms | ~45,000 | $match primero, $lookup con pipeline interno |
| **Mejora** | **___%** | **___%** | |

---

## M5 — MongoDB Atlas Performance Advisor

> 📸 Agregar captura de pantalla del Performance Advisor en `/evidencias/mongodb/capturas/`

**Índices sugeridos por el Advisor (registrar aquí):**

| Colección | Índice sugerido | Impact Score | Acción tomada |
|---|---|---|---|
| orders_reviews | `{ review_score: 1, review_creation_date: 1 }` | ___ | Implementado como ESR |
| geolocation | `{ geolocation_state: 1 }` | ___ | Implementado con zip_prefix |

---

## Resumen Global MongoDB

| Optimización | Antes | Después | Reducción |
|---|---|---|---|
| Query score + fecha (ESR index) | ___ ms | ___ ms | ___% |
| Query reseñas negativas (partial) | ___ ms | ___ ms | ___% |
| Full-text search comentarios | ___ ms | ___ ms | ___% |
| Pipeline analítico completo | ___ ms | ___ ms | ___% |

### Métricas de índices (`db.orders_reviews.stats()`)
```
totalIndexSize: ___ MB
indexSizes:
  _id_: ___ KB
  idx_reviews_score_created_answered: ___ KB
  idx_reviews_negative_partial: ___ KB
  idx_reviews_fulltext: ___ KB
  idx_reviews_order_score: ___ KB
```

> **Index Hit Ratio** (queries que usan índice vs COLLSCAN): ___%
> Objetivo: > 95% en queries de producción.

> 📸 **Capturas requeridas:**
> - `capturas/atlas_performance_advisor.png`
> - `capturas/slow_query_log.png`
> - `capturas/explain_before.png`
> - `capturas/explain_after.png`
