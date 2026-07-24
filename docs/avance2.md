# Avance 2

## Gestión de venta de vehículos

> Documento de Diseño de Software — PSWE-04  
> Universidad Cenfotec · Maestría Profesional en Ingeniería del Software

| Campo | Detalle |
|---|---|
| **Sistema** | Gestión de venta de vehículos |
| **Hito** | Avance 2 |
| **Versión** | 0.3.1 — revisión de diseño |
| **Fecha** | 2026-07-23 |
| **Base del diseño** | Avance 1: alcance, ciclo de vida, stakeholders, drivers, escenarios de calidad y vista de contexto C4 |

---

# 1. Propósito y ajustes de diseño respecto al Avance 1

El Avance 1 definió el problema arquitectónico alrededor de una fuente de verdad interna, el ciclo de vida del vehículo, la consistencia con publicaciones externas, la trazabilidad y el uso de servicios como Google Drive y Google Calendar sin convertirlos en registros oficiales del negocio.

Este Avance 2 convierte esas decisiones en estructura de software. La propuesta se apoya en cuatro ideas:

1. el inventario, el precio y la disponibilidad oficial se mantienen dentro del sistema;
2. el ciclo de vida del vehículo se controla mediante reglas de dominio y no mediante un campo editable libremente;
3. los canales externos se desacoplan mediante adaptadores y procesamiento asíncrono;
4. los cambios sensibles generan auditoría y efectos de publicación de forma trazable.

## 1.1 Refinamiento del ciclo de vida definido en S07

En S07 se usaron `CON_CLIENTE_POTENCIAL_ACTIVO` y `CITA_AGENDADA` como hitos del proceso comercial. Para el diseño detallado se separan de los estados propios del vehículo.

La razón es práctica: un vehículo publicado puede tener varios clientes potenciales y varias citas al mismo tiempo. Si una cita se almacena como estado único del vehículo, el modelo no puede representar correctamente esa concurrencia de actividades.

Por eso, en este avance se distinguen cuatro conceptos:

### Estado del vehículo

```text
INGRESADO
PENDIENTE_DOCUMENTACION
PENDIENTE_FOTOS
LISTO_PARA_PUBLICAR
PUBLICADO
RESERVADO
VENDIDO
RETIRADO
```

### Estado de cliente potencial

```text
NUEVO
EN_SEGUIMIENTO
DESCARTADO
CONVERTIDO
```

### Estado de cita

```text
SOLICITADA
AGENDADA
CONFIRMADA
REALIZADA
CANCELADA
NO_ASISTIO
```

### Estado de reserva

```text
ACTIVA
CANCELADA
CONVERTIDA_VENTA
```

Este ajuste no cambia el alcance funcional de S07. Lo vuelve implementable sin mezclar la disponibilidad del vehículo con actividades que pueden existir en paralelo.

---

# 2. Vista de contenedores C4

## 2.1 Delimitación de la solución

El sitio web propio se modela como un **contenedor separado** del backoffice. Ambos consumen la misma API central, pero tienen permisos y casos de uso distintos.

El sitio público puede:

- consultar inventario publicado;
- ver información comercial autorizada;
- registrar un cliente potencial;
- enviar una solicitud de visita.

El sitio público no:

- escribe directamente en inventario;
- cambia estados del vehículo;
- confirma reservas;
- confirma una cita por sí mismo;
- modifica publicaciones externas.

Una solicitud de visita entra a la API y pasa al módulo de agenda. La cita queda registrada internamente y solo después se sincroniza con Google Calendar cuando corresponda.

El backoffice es utilizado por el personal del negocio para administrar inventario, clientes potenciales, citas, documentos, publicaciones, reservas, ventas, comisiones y tareas pendientes.

## 2.2 Diagrama de contenedores

<img src="../diagramas/diagrama_vista_de_contenedores_c4_avance2.png">
*Figura 1. Vista C4 de contenedores de Gestión de venta de vehículos.*

## 2.3 Contenedores y responsabilidades

| Contenedor | Tecnología | Responsabilidad |
|---|---|---|
| **Backoffice web** | React + TypeScript | Interfaz privada para vendedores y personal administrativo. Consume la API central; no implementa reglas de transición por su cuenta. |
| **Sitio web público** | Next.js + TypeScript | Muestra únicamente vehículos autorizados para publicación, registra interesados y solicitudes de visita. No mantiene inventario propio. |
| **API central** | Java 21 + Spring Boot + Spring Security | Punto de entrada de las reglas de negocio. Administra inventario, ciclo de vida, clientes potenciales, citas, reservas, documentos, publicaciones, ventas, comisiones, autorización y auditoría. |
| **Procesador de integraciones** | Java 21 + Spring Boot | Procesa eventos Outbox, ejecuta adaptadores externos, aplica reintentos, maneja idempotencia y actualiza el estado de sincronización. |
| **PostgreSQL** | PostgreSQL | Fuente de verdad de los datos internos, auditoría, metadatos de documentos, publicaciones externas, reservas y eventos Outbox. |

## 2.4 Relaciones y protocolos

| Origen | Destino | Protocolo | Información / operación |
|---|---|---|---|
| Usuario interno | Backoffice | HTTPS | Operación privada del negocio. |
| Cliente comprador | Sitio público | HTTPS | Consulta de vehículos y envío de interés. |
| Backoffice | API central | HTTPS + REST/JSON | Operaciones autenticadas del negocio. |
| Sitio público | API central | HTTPS + REST/JSON | Inventario público, cliente potencial y solicitud de visita. |
| API central | PostgreSQL | JDBC + SQL sobre TCP/TLS | Persistencia transaccional. |
| Procesador de integraciones | PostgreSQL | JDBC + SQL sobre TCP/TLS | Lectura de Outbox, reintentos y actualización de sincronizaciones. |
| Procesador de integraciones | Google Drive | HTTPS REST + OAuth 2.0 | Archivos y enlaces. |
| Procesador de integraciones | Google Calendar | HTTPS REST + OAuth 2.0 | Sincronización de citas. |
| Procesador de integraciones | CRAutos | HTTPS/API cuando exista; tarea manual en caso contrario | Publicación y actualización de disponibilidad/precio. |
| Procesador de integraciones | Facebook Marketplace | HTTPS/API cuando exista; tarea manual en caso contrario | Publicación y actualización de disponibilidad/precio. |
| Procesador de integraciones | Encuentra24 | HTTPS/API cuando exista; tarea manual en caso contrario | Publicación y actualización de disponibilidad/precio. |
| Procesador de integraciones | WhatsApp Business | HTTPS API / webhook según capacidades disponibles | Captación o seguimiento de contactos. |
| Procesador de integraciones | Correo/notificaciones | HTTPS API o SMTP | Alertas y recordatorios. |

## 2.5 Relación del sitio propio con los módulos internos

| Función del sitio público | Módulo interno que la resuelve | Regla |
|---|---|---|
| Consultar vehículos | Inventario | Solo expone vehículos `PUBLICADO` y campos comerciales permitidos. |
| Consultar precio y disponibilidad | Inventario | Lee el valor oficial de PostgreSQL; no consulta marketplaces en tiempo real. |
| Registrar interés | Clientes potenciales | Crea un `Lead` con canal `SITIO_WEB`. |
| Solicitar visita | Clientes potenciales + Agenda | Crea una solicitud asociada al interesado; la cita confirmada se controla en Agenda. |
| Consultar fotos | Documentos/medios | La API entrega únicamente referencias autorizadas para exposición pública. |
| Mostrar si está reservado | Inventario/publicación interna | La política del sitio decide si muestra “Reservado” o si oculta el vehículo; el estado siempre se deriva de la fuente interna. |

## 2.6 Consistencia con la vista de contexto de S07

- El sistema interno sigue siendo la fuente oficial del precio, estado y disponibilidad.
- El sitio web propio forma parte de la solución y consume la misma API que el backoffice.
- Google Drive almacena archivos, pero las reglas sobre documentos se apoyan en metadatos internos.
- Google Calendar replica o apoya una cita; la cita comercial existe primero dentro del sistema.
- WhatsApp, CRAutos, Facebook Marketplace y Encuentra24 son canales externos y no controlan el estado del vehículo.
- Una falla externa no revierte una reserva, venta, retiro o cambio de precio ya confirmado internamente.

---

# 3. Estilo arquitectónico

## 3.1 Estilo seleccionado

Se selecciona un **monolito modular para la API central**, con límites inspirados en puertos y adaptadores, acompañado por un **procesador asíncrono de integraciones basado en Outbox**.

La API se despliega como una sola aplicación, pero internamente se divide en módulos con dependencias controladas:

```text
Inventario y ciclo de vida
Clientes potenciales
Agenda
Reservas
Publicaciones
Documentos
Ventas y comisiones
Auditoría
Integraciones
```

Las reglas del negocio permanecen dentro de la API. Las APIs de Google, WhatsApp o marketplaces se ocultan detrás de puertos. La transacción principal no espera la respuesta de esos proveedores.

## 3.2 Razón principal de la elección

El problema central requiere **consistencia fuerte dentro del sistema** y tolera **consistencia eventual con canales externos**.

Por ejemplo, al reservar un vehículo deben quedar coordinados en una sola transacción:

- estado del vehículo;
- reserva activa;
- auditoría;
- estado interno de publicaciones;
- evento pendiente de sincronización.

La actualización de CRAutos, Facebook Marketplace o Encuentra24 ocurre después. Si un canal falla, el sistema sigue sabiendo que el vehículo está reservado y qué publicación quedó pendiente.

## 3.3 Tácticas asociadas a los escenarios de calidad de S07

| Escenario de S07 | Medida definida | Táctica arquitectónica | Cómo se verifica |
|---|---|---|---|
| Consulta rápida de inventario | p95 < 3 s | Lectura desde PostgreSQL, índices por criterios de búsqueda y proyección de ficha sin consultar servicios externos en tiempo real | Prueba de carga sobre búsquedas y ficha de vehículo |
| Cambio de precio | Cambio interno < 2 s; 100% auditado | Transacción local en API + auditoría en la misma transacción | Prueba de integración que mida tiempo y compruebe auditoría |
| Estado reservado/vendido/retirado | Bloqueo interno < 2 s | Máquina de estados + restricción de reserva activa + transacción local | Pruebas de concurrencia y transición |
| Tareas de publicación | creadas < 1 min | Outbox transaccional + worker de integración | Prueba con canal simulado fuera de servicio |
| Falla de integración | error visible < 1 min | Reintentos, estado por publicación y registro de último error | Prueba de resiliencia con adaptador que devuelve error |
| Registro de cliente potencial | < 1 min y máximo 5 campos obligatorios | Endpoint específico de alta rápida, sin depender de APIs externas | Prueba de usabilidad y tiempo de tarea |
| Acceso a documentos | 100% valida rol | Autorización en API y auditoría de accesos sensibles | Prueba automatizada por rol |

## 3.4 Comparación con alternativas

| Criterio | Monolito modular + Outbox | Microservicios desde el inicio | Monolito CRUD tradicional |
|---|---|---|---|
| Consistencia interna | **Alta**: una transacción local puede coordinar vehículo, reserva, auditoría y Outbox | Media: requiere coordinación distribuida o sagas | Alta técnicamente, pero reglas pueden dispersarse |
| Complejidad operativa | **Media-baja** | Alta: varios despliegues, tracing, contratos y autenticación entre servicios | Baja |
| Aislamiento frente a proveedores externos | **Alto** con puertos/adaptadores | Alto | Bajo si controladores/servicios llaman APIs directamente |
| Facilidad para mantener reglas de estado | **Alta** con módulo de dominio | Alta, pero con costo de coordinación entre servicios | Baja si el estado queda como CRUD |
| Escalamiento independiente | Medio | **Alto** | Bajo |
| Adecuación al tamaño inicial del sistema | **Alta** | Baja-media | Media |
| Riesgo de sobrearquitectura | Medio-bajo | **Alto** | Bajo, pero con riesgo de deuda de diseño |

## 3.5 Alternativa rechazada: microservicios desde el inicio

### Ventajas

- despliegue independiente por dominio;
- escalamiento por servicio;
- aislamiento de fallas de procesos internos;
- equipos separados podrían evolucionar servicios de forma autónoma.

### Razones para rechazarla

Una venta o reserva cruza varias reglas que deben quedar consistentes. Si inventario, reservas, publicaciones, auditoría y ventas fueran servicios separados, el equipo tendría que introducir desde el inicio sagas, compensaciones, mensajería entre servicios, observabilidad distribuida y contratos versionados.

Ese costo no está justificado por el tamaño esperado del sistema ni por el equipo actual.

### Trade-off aceptado

Se renuncia a despliegues independientes por módulo a cambio de transacciones internas simples y reglas de dominio más fáciles de verificar. Los límites internos se conservan para permitir una separación futura si el volumen o el equipo lo justifican.

## 3.6 Alternativa rechazada: monolito CRUD por capas

### Ventajas

- implementación inicial rápida;
- menor cantidad de abstracciones;
- aprendizaje sencillo para operaciones básicas.

### Razones para rechazarla

Un CRUD tradicional facilita que distintos controladores o servicios escriban directamente el estado del vehículo. Eso contradice el driver principal de S07: el estado debe controlar acciones, permisos, auditoría y efectos sobre publicaciones.

### Trade-off aceptado

La propuesta modular requiere más clases y contratos internos, pero reduce el riesgo de reglas duplicadas o inconsistentes.

## 3.7 Costo aceptado por la consistencia eventual

El diseño no promete que un marketplace quede actualizado en el mismo instante que PostgreSQL.

Después de una reserva puede existir durante algunos minutos esta situación:

```text
Sistema interno: RESERVADO
Facebook Marketplace: todavía disponible
Estado interno de publicación: PENDIENTE
```

Ese desfase se acepta porque el sistema conserva la verdad comercial, bloquea nuevas reservas y muestra el pendiente al responsable. Es preferible a revertir la reserva porque un tercero no respondió.

---