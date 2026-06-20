# ADR-0002 · Audit Log append-only en lugar de event sourcing
**Estado:** aceptado  
**Fecha:** 2026-06-20

## Contexto y fuerza

US-11 y R-05 exigen un historial **inmutable** de cambios de estado por pedido, con actor y timestamp exactos. El acceso a este historial es la herramienta principal del analista para resolver reclamos o discrepancias sin depender de correos o llamadas (`personas.md`: dolor `evidencia-entrega-faltante`).

Dos patrones cumplen la inmutabilidad: un log append-only clásico, o event sourcing completo (el estado del sistema se reconstruye íntegramente desde los eventos almacenados). El segundo ofrece más poder expresivo, pero con una curva de implementación significativamente mayor.

Fuerzas:
- `requisitos.md` R-05: "los cambios deben agregarse al historial sin sobrescribir el estado anterior."
- `user-stories.md` US-11: historial en línea de tiempo cronológica, actor, timestamp, inmutable.
- `mvp-canvas.md`: principio de "lo más simple que funcione."

## Decisión

Se adopta un **Audit Log append-only**: una tabla `order_history` en la base de datos relacional donde solo se permiten operaciones INSERT. Los UPDATE y DELETE quedan prohibidos mediante un trigger de base de datos o una política a nivel de rol de usuario.

El estado actual del pedido se almacena **desnormalizado** en la tabla `orders` (campo `current_state`) para lecturas rápidas. Este campo es la proyección del último registro en `order_history` y siempre es consistente porque toda mutación pasa por la State Machine en la misma transacción.

Esquema simplificado:
```sql
-- orders: estado actual (lectura rápida)
orders(id, current_state, provider_id, created_at, ...)

-- order_history: historial inmutable
order_history(id, order_id, previous_state, new_state, actor_id, actor_type, reason, evidence_url, recorded_at)
-- trigger: BEFORE UPDATE OR DELETE → RAISE EXCEPTION 'Historial es inmutable'
```

## Alternativas consideradas

- **Event sourcing completo** — descartado: reconstruir el estado desde eventos requiere frameworks específicos (Axon, EventStore, Marten) y una curva de aprendizaje alta que no está justificada para un MVP de 17 historias. Además, las consultas de auditoría simples (dame el historial de esta orden) son más directas con SQL estándar que con proyecciones.
- **Historial como campo JSONB de la orden** — descartado: almacenar el historial en un array JSON dentro de la fila de la orden no permite consultas de auditoría eficientes (requiere parsear el JSON) y no garantiza inmutabilidad a nivel de base de datos.

## Consecuencias

**Ganamos:**
- Implementación con SQL estándar, sin frameworks adicionales.
- Consultas de auditoría directas y eficientes (`SELECT * FROM order_history WHERE order_id = X ORDER BY recorded_at`).
- Inmutabilidad garantizada por la base de datos, no solo por convención.

**Aceptamos:**
- El estado actual del pedido está duplicado (tabla `orders` + último registro de `order_history`). La State Machine es la única autorizada a mutar `orders.current_state`; ningún otro módulo debe hacerlo directamente.
- Si en el futuro se necesita reconstruir cualquier proyección del sistema desde el historial (event sourcing), la migración de este log a un event store es directa: cada fila de `order_history` es un evento.
