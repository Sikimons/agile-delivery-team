# ADR-0006 · Estado del pedido como máquina de estados explícita
**Estado:** aceptado  
**Fecha:** 2026-06-20

## Contexto y fuerza

El ciclo de vida del pedido Dropshipping tiene estados con restricciones de transición estrictas derivadas directamente del discovery:

- US-07 / R-07: no se puede marcar un pedido como `ENTREGADO` sin adjuntar evidencia (firma, foto o nombre del receptor).
- US-08 / R-08: un pedido con novedad no puede quedar en estado genérico `PENDIENTE`; debe estar en `NOVEDAD` con incidencia asignada.
- US-05: el estado `RECHAZADO` solo es válido desde `PENDIENTE_ACEPTACION`, no desde `EN_TRANSITO`.
- US-03: la orden no debe enviarse al proveedor si los campos obligatorios están incompletos.

Sin un mecanismo centralizado que aplique estas reglas, cada módulo (portal del proveedor, vista del analista, registro manual) necesita duplicar las validaciones, lo que inevitablemente lleva a inconsistencias.

Fuerzas:
- `user-stories.md` US-07, US-08, US-05.
- `requisitos.md` R-07, R-08.
- `mvp-canvas.md`: "historial inmutable de cambios"; las transiciones ilegales no deben ser registrables.

## Decisión

Se implementa una **máquina de estados explícita** como componente central del Dropshipping Core. Toda mutación del estado del pedido pasa obligatoriamente por este componente. Ningún módulo escribe directamente en la tabla `orders.current_state` ni en `order_history`.

**Estados válidos:**
```
PENDIENTE_ACEPTACION → ACEPTADO
PENDIENTE_ACEPTACION → RECHAZADO
ACEPTADO             → EN_TRANSITO
ACEPTADO             → CANCELADO
EN_TRANSITO          → ENTREGADO      (requiere: evidence_url != null)
IN_TRANSITO          → NOVEDAD
NOVEDAD              → EN_TRANSITO    (reintento de entrega)
NOVEDAD              → CANCELADO
ENTREGADO            → (estado terminal)
RECHAZADO            → (estado terminal)
CANCELADO            → (estado terminal)
```

Cada transición define sus **precondiciones** (campos requeridos, permisos del actor). Si no se cumplen, la State Machine lanza un error de dominio antes de tocar la base de datos.

## Alternativas consideradas

- **Validaciones dispersas en cada módulo (if/else por cada endpoint)** — descartado: duplica lógica en el portal del proveedor, la vista del analista y el registro manual. Cualquier cambio en las reglas (ej. añadir un estado nuevo) requiere modificar múltiples lugares, con alta probabilidad de que alguno quede desactualizado.
- **Workflow engine externo (Temporal, Camunda, AWS Step Functions)** — descartado: sobrecarga operativa y curva de aprendizaje no justificadas para 7 estados y ~10 transiciones en MVP. Un workflow engine aporta valor cuando las transiciones involucran esperas largas, actores externos múltiples o compensaciones complejas; aquí las transiciones son síncronas y el actor es un usuario humano.
- **Restricciones solo en la capa de base de datos (CHECK constraints, triggers)** — descartado como alternativa única: las restricciones de BD no pueden expresar lógica de negocio condicional compleja (ej. "ENTREGADO requiere evidence_url no nulo solo si el pedido pasó por IN_TRANSITO"). La State Machine en código es más expresiva y testeable.

## Consecuencias

**Ganamos:**
- Las reglas de transición están en un único lugar; cambiarlas no requiere modificar múltiples módulos.
- La State Machine es la unidad más testeable del sistema: cada transición y su precondición se prueba unitariamente sin infraestructura.
- El Audit Log siempre refleja transiciones válidas; no puede registrar un estado imposible.

**Aceptamos:**
- Toda escritura al estado del pedido pasa por un componente centralizado, que puede convertirse en un cuello de botella bajo alta concurrencia. Para el volumen esperado del MVP no es un problema; si escala, se puede distribuir por `order_id` con particionamiento.
- La State Machine debe ser el primer módulo que el equipo de desarrollo entienda profundamente: es el corazón del sistema y su incorrecta evolución puede generar estados inconsistentes en producción.
