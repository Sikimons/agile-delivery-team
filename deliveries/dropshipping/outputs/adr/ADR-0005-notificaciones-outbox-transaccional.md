# ADR-0005 · Notificaciones al cliente por patrón outbox transaccional
**Estado:** aceptado  
**Fecha:** 2026-06-20

## Contexto y fuerza

Las épicas E4 (US-13, US-14, US-15) requieren que el cliente reciba notificaciones automáticas por email en cada hito del pedido Dropshipping. El problema de consistencia es el siguiente: si la notificación se envía de forma síncrona durante el mismo request HTTP del proveedor y el SMTP falla, el pedido queda en un estado inconsistente (estado actualizado en la BD, cliente no notificado). A la inversa, si el SMTP responde pero la transacción de BD falla, el cliente recibe una notificación para un evento que no ocurrió.

OQ-1 fue resuelta: el canal de MVP es email.

Fuerzas:
- `user-stories.md` US-13: cliente recibe notificación automática al cambiar el estado del pedido.
- `user-stories.md` US-14: notificación proactiva cuando cambia la fecha de entrega.
- `user-stories.md` US-15: preaviso al cliente antes del contacto del proveedor.
- `requisitos.md` R-13: las notificaciones deben indicar el producto afectado y qué cambió; no mensajes genéricos.

## Decisión

Se adopta el **patrón transactional outbox**: en el mismo commit de base de datos que registra el cambio de estado en el Audit Log, la State Machine inserta un registro en la tabla `outbox_messages` con el contenido del email (destinatario, asunto, cuerpo ya renderizado). Esta inserción es atómica respecto al cambio de estado.

Un proceso worker independiente (`OutboxWorker`) lee periódicamente los mensajes pendientes de `outbox_messages`, los envía al proveedor de email configurado (SMTP o API SaaS) y marca cada mensaje como `SENT` con el timestamp de envío. Si el envío falla, el worker lo reintenta con backoff exponencial.

```
outbox_messages(id, order_id, recipient_email, subject, body_html, state, created_at, sent_at, retry_count)
-- state: PENDING → SENT | FAILED_PERMANENT
```

La garantía de entrega es "al menos una vez": si el worker falla entre el envío y el `UPDATE`, el mensaje se reenvía. Para email, esto es aceptable (el cliente puede recibir un duplicado ocasional, que es preferible a no recibir el email).

## Alternativas consideradas

- **Envío síncrono en el mismo request del proveedor** — descartado: si el SMTP falla, el request del proveedor falla con error 500 aunque la transición de estado se haya aplicado correctamente. La UX del proveedor no debe depender de la disponibilidad del SMTP. Además, los SMTP SaaS pueden tener latencias de 1-3 s que degradan la respuesta del portal.
- **Message broker externo (RabbitMQ, Kafka, SQS)** — descartado: añade infraestructura adicional (un broker) no justificada para el volumen de pedidos del MVP. El outbox pattern logra el mismo desacoplamiento usando la base de datos que ya existe como "broker implícito".
- **Webhook desde el estado de la orden (polling desde el cliente)** — descartado: el cliente final no gestiona webhooks; el modelo de notificación push (email) es el acordado.

## Consecuencias

**Ganamos:**
- Consistencia garantizada: el email se envía si y solo si la transición de estado se confirmó en la base de datos.
- El portal del proveedor no depende de la disponibilidad del SMTP para responder al usuario.
- Reintentos automáticos con visibilidad: los mensajes en estado `PENDING` o `FAILED` son auditables.

**Aceptamos:**
- Latencia entre la transición de estado y la notificación al cliente (segundos a minutos según la frecuencia del worker). Aceptable para este dominio.
- Si el volumen de pedidos crece significativamente, el worker single-thread puede convertirse en un bottleneck. Mitigación: escalar a un pool de workers o migrar a un message broker externo (OA-1 en `architecture.md`).
- Posibilidad de email duplicado en casos de fallo del worker post-envío. Mitigación en v2: deduplicación por `order_id + event_type` en el proveedor de email si lo soporta (ej. SendGrid).
