# Avance 1

## Descripción y alcance

## Nombre del sistema

Gestión de venta de vehículos

Plataforma de gestión para venta de vehículos por consignación.

## Descripción general

Gestión de venta de vehículos es una plataforma para apoyar la operación de un negocio de venta de vehículos por consignación. El sistema permite administrar el inventario de vehículos, registrar clientes interesados, controlar publicaciones en canales externos, coordinar citas, guardar fotos/documentos y dar seguimiento al proceso comercial hasta la venta o retiro del vehículo.

El problema actual es que muchas tareas se realizan de forma manual usando WhatsApp, hojas de cálculo, carpetas de fotos, publicaciones en diferentes plataformas y seguimiento informal con clientes. Esto genera duplicación de información, pérdida de control sobre el estado real de cada vehículo, dificultad para saber cuáles publicaciones están actualizadas y demoras al responder a compradores.

La propuesta inicial ya define que el negocio maneja vehículos en local o con sus dueños, publicaciones en varias plataformas, fotos en Drive, contactos por WhatsApp, citas con compradores y cambios de precio. También identifica como stakeholders al dueño, vendedores, compradores, encargado de redes sociales.

## Problema que resuelve

El sistema busca resolver la falta de control centralizado en la operación de consignación de vehículos.

Actualmente, un mismo vehículo puede tener información en varios lugares: una hoja de cálculo, una carpeta de fotos, una publicación en CR Autos, Marketplace, Encuentra24, sitio web y conversaciones separadas por WhatsApp. Esto hace difícil saber si el precio está actualizado, si el vehículo tiene fotos suficientes, si ya fue publicado, si hay citas pendientes o si un comprador quedó esperando respuesta.

## Alcance dentro del sistema

El sistema sí incluye:

- Registro y administración de vehículos en consignación.
- Control del estado del vehículo: recibido, pendiente de fotos, publicado, reservado, vendido, retirado.
- Gestión de clientes vendedores y compradores.
- Registro y seguimiento de leads provenientes de WhatsApp, redes sociales o plataformas externas.
- Agenda de citas para ver vehículos.
- Gestión de fotos y documentos asociados a cada vehículo.
- Control de publicaciones en canales externos.
- Notificaciones internas para vendedores o administrador.
- Reportes básicos de inventario, leads, citas y ventas.
- Registro de comisiones o montos asociados a la venta.

## Alcance fuera del sistema

El sistema no incluye:

- Procesamiento directo de pagos bancarios.
- Traspaso legal completo del vehículo.
- Validación automática con Registro Nacional.
- Publicación automática garantizada en todas las plataformas externas.
- Sistema contable completo.
- Chatbot inteligente avanzado con IA generativa.
- Tasación automática oficial del valor del vehículo.

## Justificación del alcance

El alcance se limita a la operación comercial y administrativa del negocio de consignación. No se intenta crear un ERP completo ni reemplazar sistemas legales, bancarios o contables. Esto ayuda a mantener el proyecto delimitado, pero con suficiente complejidad para tomar decisiones arquitectónicas importantes.

## Stakeholders y drivers

En diseño de software, una decisión no se justifica solo porque "funciona", sino porque responde a un problema, a restricciones y a atributos de calidad.

## Stakeholders principales

| Stakeholder | Interés principal | Necesidad o expectativa |
| --- | --- | --- |
| Dueño del negocio | Control general del negocio | Ver inventario, leads, ventas, citas y comisiones desde un solo lugar. |
| Vendedor | Atender clientes rápido | Consultar datos del vehículo, fotos, precio y disponibilidad sin buscar en varias fuentes. |
| Comprador | Recibir información clara | Obtener fotos, precio, disponibilidad y agendar visita fácilmente. |
| Encargado de redes sociales | Publicar vehículos | Saber cuáles autos tienen fotos completas y texto listo para publicar. |

## Drivers arquitectónicos

| Driver | Descripción | Impacto en el diseño |
| --- | --- | --- |
| Centralizar información | El negocio necesita una sola fuente de verdad para vehículos, clientes, publicaciones y citas. | Se requiere separar inventario, leads, publicaciones y agenda como responsabilidades claras. |
| Integración con canales externos | El negocio usa WhatsApp, Drive, Calendar y plataformas de anuncios. | El sistema debe diseñarse con interfaces para integraciones externas. |
| Evitar duplicación y datos desactualizados | Un cambio de precio o estado debe reflejarse claramente. | Se necesita control de estados e historial de cambios. |
| Respuesta rápida a compradores | Los vendedores necesitan datos disponibles en segundos. | El sistema debe priorizar usabilidad y rendimiento en consultas comunes. |
| Trazabilidad comercial | Se necesita saber qué pasó con cada vehículo y cada lead. | El sistema debe registrar historial de interacciones, cambios y citas. |
| Seguridad y privacidad | Hay datos personales, teléfonos, documentos y posiblemente información sensible. | Se deben definir permisos por rol y protección de datos. |
| Modularidad | El negocio puede agregar nuevos canales de publicación o automatizaciones. | Se recomienda bajo acoplamiento entre módulos internos y servicios externos. |

## Escenarios de calidad

## Escenario 1 - Disponibilidad para consulta de inventario

| Elemento | Descripción |
| --- | --- |
| Atributo de calidad | Disponibilidad |
| Fuente | Vendedor |
| Estímulo | El vendedor necesita consultar un vehículo mientras atiende a un comprador. |
| Entorno | Horario comercial, desde navegador o dispositivo móvil. |
| Artefacto afectado | Módulo de inventario y API de consulta. |
| Respuesta | El sistema muestra la ficha del vehículo con datos principales, fotos, precio, estado y disponibilidad. |
| Medida | El 95% de las consultas debe responder en menos de 3 segundos. |

Justificación: si el sistema no responde rápido, el vendedor vuelve a usar WhatsApp o archivos manuales, perdiendo el valor principal de la plataforma.

## Escenario 2 - Integridad de datos al cambiar precio

| Elemento | Descripción |
| --- | --- |
| Atributo de calidad | Integridad de datos |
| Fuente | Dueño del negocio o vendedor autorizado |
| Estímulo | Se actualiza el precio de un vehículo. |
| Entorno | Vehículo ya publicado en una o varias plataformas externas. |
| Artefacto afectado | Inventario, historial de cambios y módulo de publicaciones. |
| Respuesta | El sistema actualiza datos, este registra quién hizo el cambio, que se cambió y actualiza el vehículo en las otras plataformas. |
| Medida | Actualizar los atributos en todas las plataformas en menos de 20 minutos. Todo cambio debe quedar registrado con usuario, fecha, valor anterior y valor nuevo. |

Justificación: el precio es un dato crítico. Si se pierde el historial o no se sabe qué publicación quedó desactualizada, se pueden dar precios incorrectos al comprador.

## Escenario 3 - Interoperabilidad con servicios externos

| Elemento | Descripción |
| --- | --- |
| Atributo de calidad | Interoperabilidad |
| Fuente | Sistema externo, como Google Drive, Google Calendar o WhatsApp Business |
| Estímulo | El sistema necesita consultar fotos, crear una cita o registrar un contacto. |
| Entorno | Operación normal con servicios externos disponibles. |
| Artefacto afectado | Módulo de integraciones. |
| Respuesta | El sistema se comunica mediante interfaces definidas y registra si la operación fue exitosa o fallida. |
| Medida | El sistema debe registrar el resultado de cada integración externa. |

Justificación: la propuesta del proyecto menciona integración con WhatsApp Business, Google Calendar, Google Drive, Facebook Marketplace, CR Autos o Encuentra24. Por eso, la arquitectura debe evitar que cada módulo se conecte de forma desordenada con servicios externos.

## Escenario 4 - Usabilidad para registrar un lead

| Elemento | Descripción |
| --- | --- |
| Atributo de calidad | Usabilidad |
| Fuente | Vendedor |
| Estímulo | Entra un comprador interesado por WhatsApp o redes sociales. |
| Entorno | El vendedor está atendiendo varias conversaciones. |
| Artefacto afectado | Módulo de leads. |
| Respuesta | El sistema permite registrar nombre o usuario, teléfono, vehículo de interés, canal de origen y siguiente acción. |
| Medida | Un lead básico debe registrarse en menos de 1 minuto. |

Justificación: si registrar un lead es lento o complicado, los vendedores no usarán el sistema y seguirán trabajando manualmente.

## Vista de contexto C4

## Descripción de la vista

La vista de contexto C4 muestra a Gestión de venta de vehículos como el sistema central que apoya la operación del negocio de vehículos por consignación.

El sistema centraliza la información de vehículos, compradores, leads, publicaciones, citas, fotos, documentos, ventas y comisiones. A su alrededor se encuentran los actores humanos que utilizan el sistema y los servicios externos con los que se integra o intercambia información.

Esta vista ayuda a entender los límites del sistema: qué queda dentro de la plataforma y qué depende de herramientas externas como WhatsApp, Google Drive, Google Calendar y plataformas de publicación.

## Actores principales

| Actor | Relación con el sistema |
| --- | --- |
| Dueño del negocio | Consulta inventario, leads, ventas, citas, reportes y comisiones. |
| Vendedor | Registra leads, consulta vehículos, actualiza datos y agenda citas con compradores. |
| Comprador | Solicita información de vehículos, consulta disponibilidad y agenda visitas. |
| Encargado de redes sociales | Revisa vehículos listos para publicar y da seguimiento a publicaciones externas. |

## Sistemas externos

| Sistema externo | Relación con el sistema |
| --- | --- |
| WhatsApp Business | Canal donde llegan compradores interesados y se da seguimiento a conversaciones. |
| Google Drive | Almacena fotos y documentos relacionados con cada vehículo. |
| Google Calendar | Permite coordinar y consultar citas para ver vehículos. |
| CR Autos | Plataforma externa donde se publican vehículos. |
| Facebook Marketplace | Plataforma externa de publicación y captación de interesados. |
| Encuentra24 | Plataforma externa donde también se pueden publicar vehículos. |
| Sitio web | Canal propio donde se puede mostrar el inventario disponible. |
| Correo / notificaciones | Medio para enviar recordatorios o alertas internas. |

## Diagrama C4 de contexto: 
[Diagramas/C4_Contexto.mmd](Diagramas/C4_Contexto.mmd)

## Leyenda del diagrama

| Elemento | Tipo | Descripción |
| --- | --- | --- |
| Gestión de venta de vehículos | Sistema principal | Plataforma que centraliza la operación comercial y administrativa del negocio. |
| Dueño del negocio | Actor humano | Supervisa inventario, ventas, citas, leads, reportes y comisiones. |
| Vendedor | Actor humano | Atiende compradores, registra leads, consulta vehículos y agenda citas. |
| Comprador | Actor humano externo | Persona interesada en comprar un vehículo. |
| Encargado de redes sociales | Actor humano | Revisa vehículos listos para publicación y da seguimiento a publicaciones externas. |
| WhatsApp Business | Sistema externo | Canal principal para recibir interesados y dar seguimiento comercial. |
| Google Drive | Sistema externo | Repositorio de fotos y documentos de vehículos. |
| Google Calendar | Sistema externo | Herramienta externa para coordinar citas. |
| CR Autos | Sistema externo | Plataforma donde se publican vehículos. |
| Facebook Marketplace | Sistema externo | Canal de publicación y captación de compradores. |
| Encuentra24 | Sistema externo | Plataforma de publicación de vehículos. |
| Sitio web | Sistema externo | Canal propio para mostrar inventario disponible. |
| Correo / Notificaciones | Sistema externo | Servicio para alertas internas y recordatorios. |

## Justificación de la vista

Esta vista muestra que el sistema no funciona aislado. Su valor principal es centralizar información que actualmente se encuentra distribuida entre hojas de cálculo, WhatsApp, carpetas de fotos, publicaciones externas y seguimiento manual.

También deja clara la frontera del sistema: Gestión de venta de vehículos no reemplaza a WhatsApp, Google Drive, Calendar ni las plataformas de publicación, sino que se integra con ellas o registra el estado de lo que ocurre en esos canales.

Esto ayuda a reducir duplicación de datos, mejorar la trazabilidad comercial y evitar que cada vendedor maneje información diferente sobre precios, fotos, citas o estado de publicación.
