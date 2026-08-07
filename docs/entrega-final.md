# Gestión de Venta de Vehículos
> Documento de Diseño de Software — PSWE-04  
> Universidad Cenfotec · Maestría Profesional en Ingeniería del Software

---

## Tabla de contenidos

1. [Descripción del sistema y alcance](#1-descripción-del-sistema-y-alcance)
2. [Stakeholders](#2-stakeholders)
3. [Drivers arquitectónicos](#3-drivers-arquitectónicos)
4. [Requerimientos de calidad — Escenarios](#4-requerimientos-de-calidad--escenarios)
5. [Restricciones](#5-restricciones)
6. [Principios de diseño adoptados](#6-principios-de-diseño-adoptados)
7. [Vistas arquitectónicas](#7-vistas-arquitectónicas)
8. [Estilo arquitectónico](#8-estilo-arquitectónico)
9. [Registro de decisiones — ADRs](#9-registro-de-decisiones--adrs)
10. [Diseño detallado de componentes](#10-diseño-detallado-de-componentes)
11. [Patrones de diseño aplicados](#11-patrones-de-diseño-aplicados)
12. [Principios y técnicas habilitadoras — evidencia](#12-principios-y-técnicas-habilitadoras--evidencia)
13. [Análisis de calidad del diseño](#13-análisis-de-calidad-del-diseño)
14. [Secciones específicas por tipo de sistema](#14-secciones-específicas-por-tipo-de-sistema)
15. [Tendencias y evolución del diseño](#15-tendencias-y-evolución-del-diseño)
16. [Glosario](#16-glosario)
17. [Referencias](#17-referencias)

---

# BLOQUE 1 — CONTEXTO Y PROBLEMA
*Hitos: Propuesta (S03) y Avance 1 (S07)*

---
## 1. Descripción del sistema y alcance

### 1.1 Descripción general

Gestión de Venta de Vehículos apoya la operación de un negocio que recibe vehículos de terceros para venderlos por consignación. El sistema registra el vehículo, el consignante, sus datos comerciales, el precio, las fotos, los documentos, los compradores interesados, las citas, las reservas y el cierre de la venta o el retiro del vehículo.

El problema que resuelve es la dispersión de información entre WhatsApp, hojas de cálculo, carpetas de archivos, calendarios y publicaciones externas. Esa dispersión produce precios contradictorios, seguimiento incompleto y poca claridad sobre la disponibilidad real. El valor del sistema consiste en mantener una fuente de verdad interna y hacer visible cualquier diferencia con los canales externos.

La solución no pretende reemplazar todos los canales. El sitio público, WhatsApp, Google Drive, Google Calendar y los marketplaces siguen cumpliendo funciones específicas, pero ninguna conversación, carpeta o publicación externa sustituye el registro operativo interno.

### 1.2 Contexto del negocio o dominio

La venta por consignación separa al **consignante**, que entrega el vehículo para su comercialización, del **comprador**, que consulta, visita, reserva o compra. El negocio debe mantener el vínculo con ambos, controlar quién puede cambiar el precio o la disponibilidad y conservar evidencia de las decisiones sensibles.

El vehículo atraviesa un ciclo comercial: ingreso, revisión documental y fotográfica, preparación de publicación, publicación, reserva, venta o retiro. Los clientes potenciales y las citas no forman parte del estado único del vehículo porque pueden existir varios en paralelo. La reserva sí afecta la disponibilidad y debe impedir una segunda reserva activa.

Los canales externos tienen capacidades diferentes. Algunos pueden admitir integración, otros requieren una operación manual y ninguno debe decidir el estado interno. Google Drive conserva archivos; Google Calendar recibe copias de citas; los marketplaces muestran publicaciones; WhatsApp funciona como canal de contacto. El sistema conserva metadatos, estados, permisos, auditoría y trabajo pendiente.

Los avances no documentan una evaluación jurídica completa ni una normativa específica obligatoria. Antes de una puesta en producción debe validarse el tratamiento legal aplicable a datos personales, documentos del vehículo, retención y acceso. Esta entrega define controles técnicos de privacidad, pero no sustituye esa revisión.

### 1.3 Alcance del sistema

**Dentro del alcance — el sistema HACE:**

- Registra vehículos en consignación, consignantes, datos comerciales, precio, disponibilidad y versión.
- Controla las transiciones del vehículo mediante estados, permisos, precondiciones y auditoría.
- Mantiene ciclos separados para clientes potenciales, citas y reservas.
- Permite registrar interesados desde sitio público, WhatsApp, redes sociales o marketplaces.
- Agenda, confirma, reprograma, cancela y registra el resultado de visitas.
- Conserva metadatos, clasificación y estado de revisión de fotos y documentos almacenados en Google Drive.
- Administra publicaciones por canal, incluyendo estados automáticos y tareas manuales verificables.
- Registra ventas, precio final, comisión y responsable.
- Genera auditoría funcional para cambios sensibles.
- Procesa efectos externos mediante Outbox, adaptadores, reintentos, dead-letter y recuperación.
- Aplica autenticación, autorización por rol y recurso, control de concurrencia e idempotencia.
- Expone inventario publicado mediante el sitio propio sin crear una fuente de datos independiente.

**Fuera del alcance — el sistema NO HACE:**

- No procesa pagos bancarios ni funciona como pasarela de pago.
- No ejecuta el traspaso legal completo del vehículo.
- No valida automáticamente información contra el Registro Nacional.
- No garantiza publicación automática en todos los marketplaces.
- No reemplaza un sistema contable.
- No realiza tasación oficial automática.
- No incorpora chatbot ni decisiones basadas en IA generativa.
- No convierte Google Drive, Calendar, WhatsApp o un marketplace en fuente de verdad.
- No sustituye una evaluación legal de privacidad, retención o cumplimiento normativo.

### 1.4 Usuarios y casos de uso principales

| Tipo de usuario | Casos de uso principales |
|---|---|
| Dueño del negocio | CU-01 consultar operación; CU-03 aprobar transición sensible; CU-08 revisar venta y comisión; CU-09 atender escalaciones. |
| Vendedor | CU-02 registrar o consultar vehículo; CU-04 registrar cliente potencial; CU-05 gestionar cita; CU-03 reservar o solicitar retiro autorizado. |
| Encargado de publicaciones | CU-06 crear, actualizar, verificar o retirar publicaciones; CU-09 atender tareas manuales y fallos funcionales. |
| Encargado de fotos y documentos | CU-07 cargar, clasificar, revisar y reconciliar documentos; atender alertas de permisos. |
| Responsable financiero | CU-08 cerrar venta, registrar precio final y comisión; consultar auditoría de cierre. |
| Administrador técnico | CU-09 administrar usuarios, credenciales, worker, dead-letter, replays y alertas. |
| Cliente vendedor o consignante | Consultar el avance comercial por medio del personal; autorizar cambios cuando corresponda; solicitar retiro. |
| Cliente comprador | CU-01 consultar inventario público; CU-04 registrar interés; CU-05 solicitar una visita. |

| ID | Caso de uso arquitectónicamente relevante |
|---|---|
| CU-01 | Consultar ficha, precio, estado y disponibilidad del vehículo. |
| CU-02 | Registrar y completar la ficha de un vehículo en consignación. |
| CU-03 | Cambiar el estado del vehículo, reservarlo, venderlo o retirarlo. |
| CU-04 | Registrar y dar seguimiento a un cliente potencial. |
| CU-05 | Solicitar, agendar, confirmar, reprogramar o cancelar una cita. |
| CU-06 | Crear y mantener publicaciones automáticas o manuales. |
| CU-07 | Gestionar documentos y fotos con control de acceso. |
| CU-08 | Registrar cierre de venta y comisión. |
| CU-09 | Operar integraciones, alertas, dead-letter, tareas manuales y recuperación. |

---

## 2. Stakeholders

| Stakeholder | Rol | Intereses principales | Preocupaciones o restricciones |
|---|---|---|---|
| Dueño del negocio | Responsable de la operación | Visibilidad de inventario, ventas, citas, comisiones y pendientes. | Información contradictoria, ventas sobre vehículos no disponibles y tareas externas sin atender. |
| Vendedor | Usuario operativo | Consultar datos confiables y registrar seguimiento con pocos pasos. | Lentitud, estados ambiguos y duplicidad de reservas. |
| Cliente vendedor o consignante | Propietario o representante que entrega el vehículo | Conocer publicación, interesados, cambios de precio y resultado. | Uso indebido de documentos y falta de trazabilidad. |
| Cliente comprador | Interesado en adquirir un vehículo | Ver precio, fotos y disponibilidad; solicitar visita. | Información desactualizada o citas para vehículos no disponibles. |
| Encargado de publicaciones | Responsable de canales | Saber qué debe publicar, corregir o retirar y conservar evidencia. | Marketplaces sin API confiable, credenciales vencidas y tareas vencidas. |
| Encargado de fotos y documentos | Responsable documental | Identificar faltantes y controlar revisión y acceso. | Enlaces públicos, permisos heredados, archivos maliciosos o metadatos desalineados. |
| Responsable financiero | Responsable del cierre | Registrar precio final, comisión y responsable. | Cierre sin autorización, datos incompletos o falta de auditoría. |
| Administrador técnico | Operación y soporte | Mantener usuarios, integraciones, métricas, alertas y recuperación. | Fallos silenciosos, secretos expuestos y replays duplicados. |
| Equipo de desarrollo | Evolución del software | Módulos comprensibles, pruebas aisladas y contratos estables. | Acoplamiento con proveedores y reglas duplicadas. |
| Operación del negocio | Continuidad operativa | Resolver fallos automáticos y manuales con responsables y SLA. | Trabajo pendiente oculto o sin dueño. |
| Proveedores externos | Drive, Calendar, WhatsApp, marketplaces y correo | Recibir solicitudes compatibles con sus capacidades. | Cuotas, cambios de contrato, indisponibilidad y autenticación. |
| Docente y equipo evaluador | Evaluación académica | Trazabilidad, consistencia, evidencia editable y justificación de decisiones. | Diagramas no editables, resultados no medidos presentados como hechos e inconsistencias entre vistas. |

---

## 3. Drivers arquitectónicos

### 3.1 Requerimientos funcionales clave

| ID | Requerimiento | Stakeholder | Por qué es un driver |
|---|---|---|---|
| RF-01 | Mantener inventario, precio, estado y disponibilidad en una fuente interna. | Dueño, vendedor, comprador | Obliga a centralizar las escrituras y evitar consultas en tiempo real a proveedores. |
| RF-02 | Controlar el ciclo de vida con transiciones válidas, permisos y precondiciones. | Dueño, vendedor | Impide resolver el dominio como un CRUD de estados libres. |
| RF-03 | Permitir varios clientes potenciales y citas sin cambiar por sí solos el estado del vehículo. | Vendedor, comprador | Exige separar agregados y ciclos de vida. |
| RF-04 | Actualizar publicaciones mediante integraciones automáticas o tareas manuales. | Encargado de publicaciones | Requiere puertos, adaptadores, estados por canal y consistencia eventual. |
| RF-05 | Confirmar una reserva o cambio interno aunque un proveedor falle. | Dueño, vendedor | Obliga a separar la transacción interna de los efectos externos. |
| RF-06 | Gestionar documentos en Drive sin delegar autorización ni reglas de negocio. | Consignante, encargado documental | Exige metadatos internos, acceso mediado, auditoría y reconciliación. |
| RF-07 | Registrar quién cambió precio, estado, documentos, cita o publicación. | Dueño, responsable financiero | Requiere auditoría funcional de solo inserción. |
| RF-08 | Recuperar integraciones fallidas sin repetir acciones ya exitosas. | Administrador técnico, operación | Requiere idempotencia, acciones por destino, dead-letter y replay controlado. |

### 3.2 Atributos de calidad prioritarios

| ID | Atributo | Importancia | Stakeholder | Justificación |
|---|---|---|---|---|
| QA-01 | Consistencia e integridad | Alta | Dueño, vendedor, comprador | Un estado o precio incorrecto puede producir una reserva o comunicación comercial inválida. |
| QA-02 | Resiliencia y operabilidad | Alta | Operación, administrador técnico | Los servicios externos pueden fallar y el trabajo pendiente debe permanecer visible y recuperable. |
| QA-03 | Seguridad y privacidad | Alta | Consignante, encargado documental | Se manejan documentos, teléfonos y datos comerciales sensibles. |
| QA-04 | Rendimiento | Alta | Vendedor, comprador | La ficha debe estar disponible durante la atención sin consultar proveedores externos. |
| QA-05 | Usabilidad | Media-alta | Vendedor, comprador | El registro de interesados y solicitudes debe ser rápido para evitar notas paralelas. |

### 3.3 Restricciones que actúan como drivers

| ID | Restricción | Tipo | Impacto en el diseño |
|---|---|---|---|
| REST-01 | No todos los marketplaces ofrecen una API confiable. | Técnica / externa | Se necesita una estrategia manual con SLA, evidencia y reconciliación. |
| REST-02 | Google Drive almacena los binarios, pero no controla el dominio. | Técnica | PostgreSQL conserva metadatos y la API media el acceso. |
| REST-03 | Google Calendar es un apoyo, no la agenda oficial. | Negocio / técnica | La cita se confirma primero en PostgreSQL y se sincroniza después. |
| REST-04 | El sistema no procesa pagos, traspasos legales ni contabilidad completa. | Alcance / negocio | Evita incorporar responsabilidades financieras o jurídicas ajenas al problema. |
| REST-05 | El sistema interno debe mantener la verdad oficial aunque un canal esté caído. | Negocio | Favorece consistencia local y consistencia eventual externa. |
| REST-06 | La línea tecnológica aprobada usa React, Next.js, Java 21, Spring Boot y PostgreSQL. | Técnica | Define contenedores, contratos y habilidades de implementación. |
| REST-07 | Los resultados de rendimiento o métricas de código no pueden declararse cumplidos sin pruebas ejecutadas. | Académica / evidencia | El documento separa diseño, criterio verificable y resultado medido. |

---

# BLOQUE 2 — REQUERIMIENTOS DE CALIDAD
*Hito: Avance 1 (S07)*

---

## 4. Requerimientos de calidad — Escenarios

### Escenario QS-01 — Rendimiento de consulta de inventario

| Elemento | Descripción |
|---|---|
| **Fuente del estímulo** | Vendedor o comprador desde el sitio público. |
| **Estímulo** | Consulta una ficha de vehículo durante la atención comercial. |
| **Entorno** | Horario comercial y carga normal. |
| **Artefacto** | API de consulta, proyección de inventario y PostgreSQL. |
| **Respuesta** | Retorna datos comerciales, precio, estado, disponibilidad, fotos autorizadas y estado resumido de publicaciones sin consultar proveedores externos. |
| **Medida de respuesta** | El 95 % de las consultas responde en menos de 3 segundos. |

*Tensión con:* QS-06, porque autorización, auditoría y controles adicionales consumen recursos; se evita aplicar acceso documental pesado a la consulta comercial.

### Escenario QS-02 — Cambio de precio y consistencia de publicaciones

| Elemento | Descripción |
|---|---|
| **Fuente del estímulo** | Dueño del negocio o vendedor autorizado. |
| **Estímulo** | Cambia el precio de un vehículo publicado. |
| **Entorno** | Existen uno o varios canales externos, automáticos o manuales. |
| **Artefacto** | Inventario, auditoría, publicaciones, Outbox y procesador. |
| **Respuesta** | Confirma el precio interno, conserva el valor anterior y crea una acción por publicación. |
| **Medida de respuesta** | Cambio interno en menos de 2 segundos; 100 % auditado; acciones visibles en menos de 1 minuto; intento automático antes de 20 minutos cuando el proveedor esté disponible. |

*Tensión con:* QS-04, porque priorizar la disponibilidad de la operación interna acepta un periodo de consistencia eventual con terceros.

### Escenario QS-03 — Reserva, venta o retiro consistente

| Elemento | Descripción |
|---|---|
| **Fuente del estímulo** | Vendedor o dueño autorizado. |
| **Estímulo** | Cambia un vehículo a reservado, vendido o retirado. |
| **Entorno** | Puede haber solicitudes concurrentes, clientes potenciales, citas y publicaciones activas. |
| **Artefacto** | Gestor del ciclo de vida, reservas, agenda, auditoría, publicaciones y PostgreSQL. |
| **Respuesta** | Valida transición, versión, permisos y precondiciones; impide nuevas operaciones incompatibles; genera efectos externos. |
| **Medida de respuesta** | Bloqueo interno en menos de 2 segundos; como máximo una reserva activa; ninguna transición sensible sin actor y motivo cuando aplique. |

*Tensión con:* QS-05, porque más validaciones y control de concurrencia agregan pasos, pero protegen la disponibilidad real.

### Escenario QS-04 — Falla de integración externa

| Elemento | Descripción |
|---|---|
| **Fuente del estímulo** | Drive, Calendar, WhatsApp, marketplace, correo o la red. |
| **Estímulo** | La integración no responde, rechaza la solicitud o confirma un resultado incierto. |
| **Entorno** | Operación normal con un proveedor parcial o totalmente indisponible. |
| **Artefacto** | Outbox, acción de integración, adaptador, dead-letter y tablero operativo. |
| **Respuesta** | Mantiene el cambio interno, clasifica el error, programa reintento o tarea manual, alerta y permite recuperación auditada. |
| **Medida de respuesta** | Error visible en menos de 1 minuto; toda acción conserva canal, entidad, intento, error sanitizado y responsable; alertas críticas a los umbrales definidos. |

*Tensión con:* QS-02, porque la recuperación asíncrona no garantiza actualización inmediata de todos los canales.

### Escenario QS-05 — Registro rápido de cliente potencial

| Elemento | Descripción |
|---|---|
| **Fuente del estímulo** | Vendedor o comprador desde el sitio público. |
| **Estímulo** | Registra un interés por un vehículo y una siguiente acción o solicitud de visita. |
| **Entorno** | El vendedor atiende varias conversaciones o el comprador usa un dispositivo móvil. |
| **Artefacto** | Sitio público, backoffice, LeadService y API. |
| **Respuesta** | Crea el cliente potencial con los datos mínimos y permite continuar el seguimiento. |
| **Medida de respuesta** | Registro básico en menos de 1 minuto y con no más de cinco campos obligatorios. |

*Tensión con:* QS-06, porque reducir campos mejora la usabilidad pero exige evitar recolectar datos innecesarios y validar acceso posterior.

### Escenario QS-06 — Acceso a documentos sensibles

| Elemento | Descripción |
|---|---|
| **Fuente del estímulo** | Vendedor, encargado documental, responsable financiero o administrador con excepción aprobada. |
| **Estímulo** | Intenta cargar, consultar o descargar un documento asociado a un vehículo. |
| **Entorno** | El contenido reside en Google Drive y puede ser confidencial o restringido. |
| **Artefacto** | API central, política de autorización, metadatos, inspección, auditoría y adaptador de Drive. |
| **Respuesta** | Valida rol, recurso, acción, finalidad, estado de revisión y permisos; media la transferencia y registra el resultado. |
| **Medida de respuesta** | El 100 % de accesos valida autorización; cero enlaces públicos para documentos internos, confidenciales o restringidos; todo acceso sensible queda auditado. |

*Tensión con:* QS-01 y QS-05, porque la protección añade validaciones; se limita el acceso documental a endpoints específicos.

---

## 5. Restricciones

| ID | Restricción | Tipo | Origen | Impacto en el diseño |
|---|---|---|---|---|
| REST-01 | Marketplaces con automatización desigual. | Técnica | Proveedores externos | Strategy + Adapter y proceso manual. |
| REST-02 | Binarios en Google Drive. | Técnica | Decisión de diseño aprobada | Metadatos y reglas en PostgreSQL; acceso mediado. |
| REST-03 | Agenda interna antes que Calendar. | Negocio | Operación comercial | Outbox para sincronización posterior. |
| REST-04 | Sin pagos ni traspaso legal. | Negocio / alcance | Definición del proyecto | Ventas registra cierre y comisión, no liquida dinero ni actos legales. |
| REST-05 | Consistencia fuerte dentro de una base transaccional. | Técnica / negocio | Drivers de estado y reserva | Monolito modular y PostgreSQL. |
| REST-06 | Consistencia eventual con terceros. | Técnica | Disponibilidad externa | Estados por acción, reintentos y tareas. |
| REST-07 | Protección de documentos y datos personales. | Seguridad | Stakeholders y escenario QS-06 | Autorización contextual, clasificación, auditoría e inspección. |
| REST-08 | No existe aún una implementación con resultados medidos. | Evidencia | Estado del proyecto | Los valores se presentan como metas y planes de prueba. |

---
