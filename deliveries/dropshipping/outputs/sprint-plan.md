# Sprint 1 — El proveedor opera los cuatro hitos del ciclo Dropshipping desde el portal

**Capacidad:** 20 pts · **Comprometido:** 20 pts  
**Épicas cubiertas:** E1 (completa) · E2 (parcial: 4/5 historias)

---

## Sprint Goal

> Al finalizar este sprint, el proveedor puede aceptar, rechazar, registrar el despacho y confirmar la entrega con evidencia desde el portal web, y las órdenes se generan automáticamente con información completa al confirmar la venta en el eCommerce. El analista ya no necesita crear ni enviar órdenes manualmente.

---

## Historias comprometidas

| Historia | Título | Pts | Épica | Prioridad | Dependencias |
|----------|--------|-----|-------|-----------|--------------|
| US-01 | Generación automática de orden al confirmar venta | 3 | E1 | 1 | — |
| US-03 | Validación de completitud de la orden antes de enviarla | 2 | E1 | 2 | US-01 |
| US-02 | Reserva automática de inventario al confirmar pedido | 2 | E1 | 3 | US-01 |
| US-04 | Ver y aceptar órdenes asignadas con fecha estimada | 5 | E2 | 4 | US-01, US-03 |
| US-05 | Rechazar una orden indicando el motivo | 2 | E2 | 5 | US-04 |
| US-06 | Registrar despacho con guía o transporte alternativo | 3 | E2 | 6 | US-04 |
| US-07 | Confirmar entrega con evidencia adjunta | 3 | E2 | 7 | US-06 |
| **Total** | | **20** | | | |

---

## Valor entregado al finalizar el sprint

### E1 — Generación y Asignación de Órdenes (7 pts · completa)

- **US-01** · El sistema crea y envía la orden al proveedor automáticamente en < 1 min desde que se confirma el pago. El analista deja de hacer esto manualmente.
- **US-03** · La orden llega al proveedor con todos los campos obligatorios (producto, cantidad, dirección, contacto, fecha esperada, condiciones especiales). Se elimina el dolor `pedido-incompleto-al-recibir`.
- **US-02** · La unidad queda reservada en el inventario integrado al momento de confirmar el pedido. Se elimina el riesgo de doble venta (`stock-doble-canal`).

### E2 — Portal del Proveedor (13 pts · 4 de 5 historias)

- **US-04** · El proveedor ve sus órdenes en el portal con toda la información y puede aceptarlas en ≤ 3 acciones con fecha estimada de despacho. Canal único activo.
- **US-05** · El proveedor puede rechazar una orden con motivo. El equipo comercial recibe alerta inmediata.
- **US-06** · El proveedor registra el despacho con guía de transportadora o datos de transporte propio. El analista ve el tracking sin solicitarlo.
- **US-07** · El proveedor confirma la entrega adjuntando evidencia (foto, firma o nombre del receptor). El pedido cierra de forma verificable.

---

## Fuera de este sprint (Sprint 2+)

| Historia | Título | Pts | Épica | Motivo |
|----------|--------|-----|-------|--------|
| US-08 | Registrar novedad de entrega desde el campo | 3 | E2 | Capacidad agotada; completa E2 en Sprint 2. |
| US-09 | Vista en tiempo real de pedidos activos (analista) | 5 | E3 | Inicia E3 en Sprint 2. |
| US-10 | Alerta por pedido sin actualización en plazo | 3 | E3 | Depende de US-09. |
| US-11 | Historial inmutable con actor y timestamp | 3 | E3 | Depende de US-09. |
| US-12 | Registro manual para proveedores por correo | 2 | E3 | Depende de US-11. |
| US-13 | Notificaciones automáticas al cliente en hitos | 5 | E4 | E4 inicia en Sprint 2/3. |
| US-14 | Notificación proactiva cambio de fecha | 2 | E4 | Depende de US-13, US-12. |
| US-15 | Preaviso al cliente antes del contacto del proveedor | 2 | E4 | Depende de US-13. |
| US-16 | Dashboard de KPIs operativos Dropshipping | 5 | E5 | Depende de datos de sprints anteriores. |
| US-17 | Indicadores diferenciados del proveedor | 5 | E5 | Depende de US-16. |
| **Restante** | | **35 pts** | | |

---

## Definition of Ready — verificación

Todas las historias del sprint cumplen:

| Criterio | US-01 | US-02 | US-03 | US-04 | US-05 | US-06 | US-07 |
|----------|-------|-------|-------|-------|-------|-------|-------|
| Como/Quiero/Para completo | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Criterios de aceptación (Gherkin) | ✓ 3 | ✓ 4 | ✓ 3 | ✓ 4 | ✓ 3 | ✓ 4 | ✓ 4 |
| Estimación ≤ 8 pts | 3 | 2 | 2 | 5 | 2 | 3 | 3 |
| Dependencias en el sprint | — | US-01 ✓ | US-01 ✓ | US-01, US-03 ✓ | US-04 ✓ | US-04 ✓ | US-06 ✓ |
| Sin preguntas abiertas | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

---

## Notas del Scrum Master

**Sobre la secuencia:** US-01 desbloquea US-02, US-03 y (junto con US-03) US-04. El orden de implementación sugerido es US-01 → US-03 → US-02 (paralelo) → US-04 → US-05 / US-06 (paralelo) → US-07. La State Machine (ADR-0006) debe implementarse como parte de US-04, antes de US-05, US-06 y US-07.

**Sobre el portal:** El magic link (ADR-0004) y el módulo de autenticación del proveedor son prerequisitos de US-04. Si el equipo estima que la autenticación requiere más esfuerzo del esperado, puede tratarse como una tarea técnica dentro de US-04 o elevarse como historia técnica antes del sprint planning formal.

**Riesgo de adopción del proveedor:** El `mvp-canvas.md` señala explícitamente que sin adopción del proveedor el MVP no entrega valor. Se recomienda realizar una demo/piloto con el proveedor entrevistado antes de cerrar el sprint, usando datos reales de órdenes. Este feedback puede modificar US-04 (el hito más crítico).

**Qué NO entra este sprint por diseño:** La vista del analista (E3) y las notificaciones al cliente (E4) dependen de que el portal del proveedor esté funcionando. La secuencia E1 → E2 → E3 → E4 no es solo una priorización por valor: es la cadena causal del MVP. Sin proveedor operando en el portal (E2), no hay actualizaciones de estado que mostrar al analista (E3) ni hitos que notificar al cliente (E4).
