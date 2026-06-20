# ADR-0003 · Portal del proveedor como módulo del frontend existente
**Estado:** aceptado  
**Fecha:** 2026-06-20

## Contexto y fuerza

El proveedor necesita una interfaz web para operar los cuatro hitos de entrega (US-04 a US-08). La restricción crítica es R-30 y la advertencia directa del proveedor entrevistado: _"si para cada pedido tengo que llenar demasiados campos, al final la gente vuelve al correo"_ (`mvp-canvas.md`: Nota de implementación "Simplicidad del portal del proveedor"). El portal debe ser simple y accesible.

OQ-2 fue resuelta: el portal es una URL web pública, no una integración con el ERP/sistema del proveedor.

Fuerzas:
- `requisitos.md` R-30: portal simple, pocos campos, flujo lineal por pedido.
- `mvp-canvas.md`: "≤ 3 acciones por hito"; "validar con al menos un proveedor piloto antes de lanzar."
- `requisitos.md` R-23: el proveedor solo recibe los datos del cliente estrictamente necesarios para ejecutar la entrega.

## Decisión

El portal del proveedor se implementa como un **módulo del frontend existente** bajo la ruta `/proveedor/*`, con su propio contexto de autenticación (magic link — ver ADR-0004). Comparte la infraestructura de hosting, el dominio y la librería de componentes de la plataforma.

El módulo es deliberadamente restringido: solo muestra las órdenes propias del proveedor autenticado. No tiene acceso a módulos internos del sistema ni a datos de otros proveedores o pedidos ajenos.

## Alternativas consideradas

- **Aplicación web standalone separada** — descartado: requiere infraestructura de despliegue adicional (CDN, certificados TLS, pipeline CI/CD propia) sin añadir valor en MVP. Complejidad operativa injustificada.
- **Progressive Web App (PWA) dedicada** — descartado: no hay evidencia en el discovery de que los proveedores necesiten acceso offline o instalación en dispositivo. Una PWA standalone añade complejidad (service workers, manifest) sin respaldo en las entrevistas.
- **Aplicación nativa móvil** — descartado: fuera de alcance del MVP. No hay evidencia de que los proveedores operen desde móvil como canal principal.

## Consecuencias

**Ganamos:**
- Un solo despliegue para toda la plataforma.
- Reutilización de la librería de componentes y estilos (consistencia visual, desarrollo más rápido).
- El piloto con el proveedor puede empezar sin provisionar infraestructura adicional.

**Aceptamos:**
- El portal del proveedor comparte el dominio con el eCommerce interno. Si en el futuro se requiere aislamiento total de dominio (ej. por razones de seguridad o branding), habrá que extraerlo a un subdominio o aplicación separada.
- Si el volumen de tráfico del portal supera la capacidad del frontend existente, se puede servir el módulo desde un CDN dedicado sin cambiar el código.
