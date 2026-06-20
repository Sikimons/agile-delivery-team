# ADR-0001 · Monolito modular como arquitectura base
**Estado:** aceptado  
**Fecha:** 2026-06-20

## Contexto y fuerza

El MVP del Portal Dropshipping tiene 17 historias y 55 puntos distribuidas en 5 épicas. El equipo es pequeño. La épica E1 (Generación de Órdenes) requiere consistencia transaccional entre tres operaciones que deben ejecutarse en un solo commit: crear la orden, reservar el inventario y registrar la entrada en el Audit Log. Esta coordinación entre módulos es directa y sin overhead en un monolito; en microservicios requeriría transacciones distribuidas (Saga, 2PC) que añaden complejidad sin aportar valor en este momento.

Fuerzas:
- MVP de alcance acotado y equipo pequeño (se itera rápido con un solo despliegue).
- E1: transacciones ACID entre Order Manager, Inventory Lock y Audit Log.
- `mvp-canvas.md`: "maximiza el trabajo no hecho"; la complejidad operativa de microservicios no está justificada.

## Decisión

Se adopta una arquitectura de **monolito modular**: un único proceso/contenedor desplegable, con módulos internos claramente delimitados por interfaces (no HTTP entre procesos). Los módulos son: Dropshipping Core (Order Manager, State Machine, Audit Log, Inventory Lock, Outbox Worker, KPI Read Model), Frontend (Portal del Proveedor, Vista del Analista).

La modularidad interna garantiza que cada módulo pueda extraerse a un servicio independiente si el volumen o la complejidad del equipo lo justifica en el futuro.

## Alternativas consideradas

- **Microservicios desde el inicio** — descartado: la complejidad operativa (service discovery, distributed transactions, despliegues coordinados) supera el beneficio en MVP con 17 historias. Las transacciones ACID de E1 serían especialmente costosas de coordinar.
- **Serverless functions (FaaS)** — descartado: la State Machine y el Audit Log requieren consistencia transaccional que es difícil de garantizar entre funciones independientes; el cold start añade latencia inaceptable en el portal del proveedor.

## Consecuencias

**Ganamos:**
- Transacciones ACID nativas entre módulos (Order Manager + Inventory Lock + Audit Log en un commit).
- Despliegue simple: un contenedor, una pipeline CI/CD.
- Desarrollo rápido en MVP: no hay overhead de comunicación entre servicios.

**Aceptamos:**
- Si el portal del proveedor crece en complejidad o tráfico, puede necesitar extraerse a un servicio separado. La delimitación clara de módulos facilita esa extracción sin reescribir lógica de negocio.
- El monolito puede convertirse en un "big ball of mud" si no se respetan las interfaces entre módulos. Mitigación: revisiones de arquitectura periódicas verifican que los módulos no se acoplen ilegítimamente.
