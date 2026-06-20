# ADR-0007 · Evidencias en object storage S3-compatible
**Estado:** aceptado  
**Fecha:** 2026-06-20

## Contexto y fuerza

US-07 exige que al confirmar la entrega el proveedor adjunte evidencia (foto, firma digital o nombre del receptor). US-08 exige que al registrar una novedad el proveedor pueda adjuntar una foto del estado en campo. Ambas evidencias deben:

1. Estar accesibles desde múltiples módulos (portal del proveedor al subir, vista del analista al consultar, KPI read model para auditoría).
2. Ser referenciables desde el Audit Log de forma inmutable (el registro de historial cita la URL de la evidencia).
3. Sobrevivir a reinicios del servidor y despliegues (no pueden estar en memoria o en el filesystem local de un proceso efímero).

Fuerzas:
- `user-stories.md` US-07: "el archivo queda vinculado al historial y el cliente recibe la confirmación."
- `user-stories.md` US-08: "la imagen queda vinculada a la incidencia como evidencia de campo."
- `requisitos.md` R-07: "evidencia como condición para marcar el pedido como entregado."

## Decisión

Las evidencias (fotos, firmas) se almacenan en un **bucket de object storage S3-compatible** (AWS S3 en producción; MinIO para desarrollo local). El Audit Log almacena la URL de cada evidencia, no el binario.

El flujo de subida es:
1. El cliente (portal del proveedor) solicita una URL prefirmada (presigned URL) al backend.
2. El proveedor sube el archivo directamente al bucket usando esa URL (sin pasar por el backend, reduciendo latencia y carga del servidor).
3. El backend recibe confirmación de la subida y procede con la transición de estado en la State Machine, registrando la URL en el Audit Log.

Retención: indefinida en MVP. La política de lifecycle (ej. archivar a almacenamiento frío después de N meses) se define en v2 cuando haya claridad legal sobre el plazo de retención de evidencias de entrega.

## Alternativas consideradas

- **Almacenar el binario en la base de datos (BLOB/BYTEA)** — descartado: degrada el rendimiento de las consultas sobre la tabla `order_history`; los backups de base de datos se vuelven significativamente más pesados; las consultas de historial que no necesitan la imagen cargan el BLOB igualmente en algunos ORMs.
- **Filesystem local del servidor** — descartado: no es compatible con despliegues multi-instancia (si hay dos réplicas del monolito, la imagen subida a la réplica A no está disponible en la réplica B); los archivos se pierden en despliegues si el sistema de archivos no es persistente.
- **Base64 inline en el campo JSON del Audit Log** — descartado: idénticos problemas al almacenamiento en BLOB, con el agravante de que el campo JSONB se vuelve ilegible para consultas de auditoría y las imágenes Base64 tienen un overhead de ~33 % de tamaño.

## Consecuencias

**Ganamos:**
- Separación de concerns: la base de datos gestiona datos estructurados, el object storage gestiona binarios.
- Las URLs de evidencia son inmutables (si se usa versionado del bucket, la URL siempre apunta al mismo archivo original).
- El proveedor sube directamente al bucket (presigned URL), sin que el backend sea el proxy de datos binarios.
- MinIO permite replicar el entorno de producción localmente sin costo.

**Aceptamos:**
- Se añade una dependencia de infraestructura adicional (el bucket). Requiere configurar credenciales y políticas de acceso del bucket desde el primer sprint de E2.
- La URL de la evidencia en el Audit Log asume que el bucket tiene retención indefinida. Si en el futuro se decide borrar evidencias antiguas, las URLs en el historial quedan rotas. Mitigación: definir la política de lifecycle antes de activar cualquier regla de eliminación automática.
- En caso de fallo de la subida al bucket (después de que el backend dio por válida la confirmación), puede quedar una transición de estado en el Audit Log con una URL inválida. Mitigación: el backend valida la existencia del objeto antes de confirmar la transición; si no existe, rechaza el hito con error descriptivo.
