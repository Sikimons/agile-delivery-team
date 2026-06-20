# ADR-0004 · Autenticación del proveedor por magic link (token de un solo uso)
**Estado:** aceptado  
**Fecha:** 2026-06-20

## Contexto y fuerza

Los proveedores son actores externos sin cuenta preexistente en el sistema. El riesgo de adopción más crítico del MVP es que el proveedor abandone el portal y vuelva al correo si percibe demasiada fricción (`mvp-canvas.md`: Riesgo #1 "Adopción del proveedor"). Crear un flujo de registro con usuario/contraseña añade fricción de onboarding y soporte de recuperación.

El proveedor entrevistado opera hoy principalmente por correo electrónico. Es el canal que domina y espera.

Fuerzas:
- `mvp-canvas.md`: Riesgo #1: "si el portal tiene demasiados campos, el proveedor vuelve al correo."
- `requisitos.md` R-30: portal simple, flujo lineal, pocos campos por acción.
- `requisitos.md` R-23: el proveedor solo accede a sus propias órdenes; la autenticación debe garantizar aislamiento por proveedor.

## Decisión

Al generar la orden (US-01), el sistema envía al proveedor un email con un **magic link**: una URL que contiene un token UUID de un solo uso con TTL de 72 horas. El proveedor accede al portal con ese enlace sin necesidad de crear cuenta ni contraseña.

Tras expirar el token (o si el proveedor lo pierde), puede solicitar un nuevo enlace desde la pantalla de login ingresando su email registrado. El analista también puede regenerar el link manualmente desde la vista de pedido.

Cada magic link es específico a un proveedor y sesión. No puede ser usado por otro proveedor.

## Alternativas consideradas

- **Usuario/contraseña con registro** — descartado: añade fricción de onboarding (formulario de registro, verificación de email, recuperación de contraseña) que va contra R-30 y el riesgo #1 del MVP. El proveedor debe poder operar en su primera visita sin configurar nada.
- **OAuth del proveedor (Google, Microsoft)** — descartado: no todos los proveedores tienen cuentas corporativas activas en esos proveedores. Añade dependencia externa y un paso de autorización que puede confundir a proveedores menos técnicos.
- **URL con token directamente en query string (sin sesión)** — descartado: el token en la URL queda expuesto en logs de servidores proxy, sistemas de analytics, historial del navegador y cabeceras Referer. La sesión con cookie firmada es más segura.
- **OTP por SMS** — descartado: no está en alcance del MVP (canal SMS es OA-1, decisión aplazada). Requiere integración con proveedor de SMS.

## Consecuencias

**Ganamos:**
- Cero fricción de onboarding: el proveedor accede en su primera visita sin configurar nada.
- Sin gestión de contraseñas (no hay contraseñas que robar ni que recuperar).
- El correo ya es el canal de trabajo del proveedor; el magic link llega donde ya está.

**Aceptamos:**
- Si el proveedor no tiene acceso al correo en el momento de operar (ej. el conductor en campo necesita registrar una novedad), queda bloqueado. Mitigación: el analista puede regenerar el link manualmente; en v2 se puede explorar un PIN de 6 dígitos para acceso en campo.
- El TTL de 72 h es un balance entre seguridad y usabilidad. Se puede ajustar por configuración sin cambiar el código.
