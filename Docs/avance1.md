# Avance 1

## 1. Descripción y alcance

### 1.1 Nombre del sistema

**Gestión de venta de vehículos**

Plataforma para apoyar la venta de vehículos por consignación, con control de inventario, clientes, publicaciones, citas, documentos y comisiones.

### 1.2 Descripción general

Gestión de venta de vehículos apoya la operación diaria de un negocio que recibe vehículos de terceros para venderlos por consignación. El sistema registra el vehículo, sus datos comerciales, su estado, las fotos, los documentos, los clientes interesados, las citas y el cierre de la venta o el retiro del vehículo.

Hoy la información queda repartida entre WhatsApp, hojas de cálculo, carpetas de fotos, publicaciones en plataformas externas y mensajes separados con clientes. Esa forma de trabajo provoca datos duplicados, precios desactualizados, seguimiento incompleto de clientes potenciales y poca claridad sobre el estado real de cada vehículo.

El sistema no busca reemplazar todos esos canales. Su función principal es ordenar la operación y dejar un repositorio centralizado. Los canales externos siguen existiendo, pero el negocio deja de depender de cada conversación o publicación como si fuera el registro oficial.

### 1.3 Problema arquitectónico central

El problema principal no es solo administrar vehículos. El punto crítico es mantener información confiable sobre cada vehículo mientras esa información se consulta, copia o actualiza en varios canales: WhatsApp Business, Google Drive, Google Calendar, CRAutos, Facebook Marketplace, Encuentra24 y el sitio web propio.

La arquitectura resuelve cómo conservar un repositorio centralizado cuando algunos canales permiten integración automática y otros dependen de tareas manuales. También debe mostrar cuándo una publicación externa ya no coincide con el precio, estado o disponibilidad registrados en el sistema.

Para evitar que el sistema termine siendo solo un CRUD de inventario convencional, se toman las siguientes reglas de base:

- El sistema interno es un repositorio centralizado del precio, estado comercial, disponibilidad, datos del vehículo, clientes potenciales, citas, documentos registrados y comisiones.
- Google Drive guarda archivos, pero el sistema guarda el vínculo, el estado del archivo, quién lo cargó y a qué vehículo pertenece.
- Google Calendar puede apoyar la agenda, pero la cita comercial queda registrada en el sistema.
- WhatsApp Business es un canal de contacto. El cliente potencial y su seguimiento quedan registrados en el sistema.
- CRAutos, Facebook Marketplace y Encuentra24 se tratan como canales externos. Cuando no exista integración confiable, el sistema registra el estado de publicación y genera tareas manuales.
- El sitio web propio se considera parte de la solución cuando muestra el inventario desde la fuente de verdad interna.

### 1.4 Reglas de consistencia con canales externos

| Situación | Respuesta esperada del sistema |
|---|---|
| Cambio de precio | Actualiza el precio interno de inmediato, guarda el valor anterior y marca las publicaciones externas como actualizadas, pendientes o fallidas según el canal. |
| Publicación externa desactualizada | Muestra una alerta al vendedor y al encargado de publicaciones. Si el canal es manual, crea una tarea para corregir la publicación. |
| Vehículo reservado | Bloquea nuevas reservas, advierte al vendedor y marca las publicaciones como pendientes de actualización de disponibilidad. |
| Vehículo vendido o retirado | Cierra el ciclo comercial del vehículo, bloquea nuevas citas y genera tareas para retirar o actualizar publicaciones externas. |
| Falla de integración | No revierte el cambio interno. Registra el error, programa reintento cuando aplique y deja visible la tarea pendiente. |
| Datos recibidos por WhatsApp | Permite convertir la conversación en cliente potencial, asociarlo a un vehículo y registrar la siguiente acción. |

### 1.5 Alcance dentro del sistema

- Registro y administración de vehículos en consignación.
- Control del estado del vehículo mediante un ciclo de vida definido.
- Gestión diferenciada de cliente vendedor o consignante y cliente comprador.
- Registro de clientes potenciales provenientes de WhatsApp, redes sociales, sitio web o plataformas externas.
- Agenda de citas para ver vehículos y seguimiento de la cita.
- Gestión de fotos y documentos mediante enlaces y metadatos asociados al vehículo.
- Control de publicaciones externas con estado por canal.
- Notificaciones internas para vendedores, administrador y encargado de publicaciones.
- Reportes básicos de inventario, clientes potenciales, citas, ventas y comisiones.
- Registro de comisiones o montos ligados a la venta.
- Auditoría de cambios de precio, estado, publicación, documentos y citas.

### 1.6 Alcance fuera del sistema

- Procesamiento directo de pagos bancarios.
- Traspaso legal completo del vehículo.
- Validación automática con Registro Nacional.
- Publicación automática garantizada en todas las plataformas externas.
- Sistema contable completo.
- Chatbot avanzado con IA generativa.
- Tasación oficial automática del valor del vehículo.

### 1.7 Justificación del alcance

El alcance se concentra en la operación comercial y administrativa de la consignación. La solución ordena el trabajo de inventario, clientes potenciales, citas, publicaciones, documentos y comisiones, sin asumir tareas legales, bancarias o contables que pertenecen a otros procesos.

Esta delimitación permite tratar problemas arquitectónicos reales: consistencia entre canales, historial de cambios, ciclo de vida del vehículo, permisos por rol, integración por adaptadores y manejo de fallas externas.

## 2. Ciclo de vida del vehículo

La lógica del ciclo de vida del vehículo es una regla central del dominio. Cada cambio de estado debe tener un responsable, una transición válida y un registro de auditoría. El estado no es solo una etiqueta visual; controla qué acciones puede hacer el negocio.

### 2.1 Estados principales

| Estado | Descripción | Acciones permitidas |
|---|---|---|
| Ingresado | El vehículo fue registrado por primera vez. | Completar datos básicos, asignar consignante, iniciar revisión. |
| Pendiente de documentación | Faltan documentos requeridos para seguir el proceso. | Cargar documentos, solicitar datos al consignante, bloquear publicación. |
| Pendiente de fotos | Los datos mínimos existen, pero faltan fotos comerciales. | Cargar fotos, asignar tarea al encargado de fotos, bloquear publicación. |
| Listo para publicar | El vehículo tiene datos, fotos y documentos mínimos. | Preparar texto, definir canales, aprobar publicación. |
| Publicado | El vehículo aparece en el sitio web propio o en canales externos. | Registrar clientes potenciales, agendar citas, cambiar precio, pausar publicación. |
| Con cliente potencial activo | Existe al menos un comprador con seguimiento pendiente. | Registrar interacción, agendar cita, descartar cliente potencial, mantener publicación. |
| Cita agendada | Hay una visita o revisión coordinada con un comprador. | Confirmar cita, reprogramar, cancelar, registrar resultado. |
| Reservado | El vehículo está apartado por un comprador. | Bloquear nuevas reservas, actualizar disponibilidad, preparar cierre. |
| Vendido | La venta fue cerrada. | Cerrar publicación, registrar comisión, bloquear nuevos clientes potenciales y citas. |
| Retirado | El consignante retiró el vehículo o el negocio dejó de ofrecerlo. | Cerrar publicación, bloquear nuevos clientes potenciales y citas, guardar motivo. |

### 2.2 Flujo base

Flujo principal:

```text
ingresado -> pendiente de documentación -> pendiente de fotos -> listo para publicar -> publicado -> con cliente potencial activo -> cita agendada -> reservado -> vendido
```

Rutas alternas permitidas:

- ingresado -> retirado, cuando el vehículo no cumple las condiciones para venderse.
- publicado -> retirado, cuando el consignante decide sacarlo de la venta.
- reservado -> publicado, cuando la reserva se cae y el vehículo vuelve a estar disponible.
- con cliente potencial activo -> publicado, cuando el cliente potencial se descarta y no queda seguimiento pendiente.
- cita agendada -> publicado, cuando la cita no concreta y no hay reserva.

### 2.3 Reglas de transición

| Transición | Quién puede hacerla | Regla de control | Evento auditado |
|---|---|---|---|
| Ingresado -> pendiente de documentación | Vendedor o encargado de documentos | Debe existir consignante asociado. | Cambio de estado, usuario, fecha y campos faltantes. |
| Pendiente de documentación -> pendiente de fotos | Encargado de documentos | Deben estar marcados los documentos mínimos. | Documentos completados y responsable. |
| Pendiente de fotos -> listo para publicar | Encargado de fotos o vendedor autorizado | Debe existir una galería mínima de fotos. | Fotos agregadas, fecha y responsable. |
| Listo para publicar -> publicado | Encargado de publicaciones o dueño | Debe existir precio aprobado y descripción comercial. | Canales seleccionados, texto usado y fecha. |
| Publicado -> con cliente potencial activo | Vendedor | Debe registrarse canal de origen y vehículo de interés. | cliente potencial creado, canal, fecha y siguiente acción. |
| Con cliente potencial activo -> cita agendada | Vendedor | Debe existir fecha, hora y responsable de atención. | Cita creada, comprador y vehículo. |
| Cita agendada -> reservado | Vendedor autorizado o dueño | Debe confirmarse comprador y monto o condición de reserva. | Reserva creada, fecha, usuario y condiciones. |
| Reservado -> vendido | Dueño o responsable financiero | Debe registrarse cierre y comisión. | Venta cerrada, monto, comisión y responsable. |
| Publicado / reservado -> retirado | Dueño o vendedor autorizado | Debe indicarse el motivo del retiro. | Retiro, motivo, fecha y responsable. |

### 2.4 Transiciones prohibidas y bloqueos

- No se puede publicar un vehículo si faltan documentos o fotos.
- No se puede vender un vehículo que nunca fue reservado o aprobado por el dueño.
- No se pueden crear nuevas citas para un vehículo vendido o retirado.
- No se pueden crear nuevas reservas para un vehículo reservado, vendido o retirado.
- No se puede cambiar el precio sin guardar valor anterior, valor nuevo, usuario y fecha.
- No se puede borrar una publicación externa del historial; solo se cambia su estado.

### 2.5 Efecto sobre publicaciones externas

| Cambio en el vehículo | Efecto en publicaciones |
|---|---|
| Pasa a listo para publicar | Habilita creación de publicación interna y tareas por canal externo. |
| Pasa a publicado | Registra canal, fecha, enlace si existe, responsable y estado inicial. |
| Cambia el precio | Marca cada publicación como actualizada, pendiente de actualización o fallida. |
| Pasa a reservado | Marca publicaciones como pendientes de cambio de disponibilidad y alerta a ventas. |
| Pasa a vendido | Genera tarea de retiro o cierre por canal y bloquea nuevas citas. |
| Pasa a retirado | Genera tarea para pausar o eliminar publicaciones y guarda motivo. |
