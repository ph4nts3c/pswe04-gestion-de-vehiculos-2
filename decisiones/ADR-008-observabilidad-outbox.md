# ADR-008 — Observabilidad, dead-letter y recuperación del Outbox

| Campo | Valor |
|---|---|
| **Estado** | Aceptado |
| **Fecha** | 2026-08-06 |
| **Autores** | Alejandro, William, Rodolfo |
| **Versión** | 1.1 |
| **Drivers** | Resiliencia, operabilidad, trazabilidad |
| **Escenarios S07** | 2 y 4 |

## Contexto

El Outbox evita perder intenciones externas, pero una fila pendiente sin alertas, responsable ni procedimiento de recuperación puede permanecer sin atención. También existe riesgo de declarar una operación terminada cuando solo se creó una tarea manual, o de convertir un evento completo en fallo aunque otros canales ya hayan terminado correctamente.

## Decisión

Se separan tres ciclos:

- evento Outbox: `PENDING`, `PROCESSING`, `RETRY_SCHEDULED`, `WAITING_MANUAL`, `PROCESSED`, `DEAD_LETTER`, `CANCELLED`;
- acción de integración: `PENDIENTE`, `EN_PROCESO`, `EN_REINTENTO`, `TAREA_MANUAL_ABIERTA`, `EXITOSA`, `DEAD_LETTER`, `CANCELADA`;
- estado de la entidad externa, como publicación o cita.

Dead-letter se controla por acción. El estado del evento se calcula a partir de sus acciones: una tarea manual abierta produce `WAITING_MANUAL`; una acción no resuelta en dead-letter produce `DEAD_LETTER`; el evento llega a `PROCESSED` solo cuando todas las acciones están `EXITOSA` o `CANCELADA`.

Los errores temporales usan espera creciente. Toda recuperación requiere permiso, motivo, auditoría, verificación del estado externo e idempotencia. El replay se dirige a acciones concretas y no repite acciones exitosas.

Se exponen métricas de edad, volumen, tasa de éxito, tareas manuales vencidas, dead-letter y replays. El administrador técnico responde por credenciales o worker; el responsable funcional atiende contenido, agenda o tareas manuales.

## Alternativas consideradas

1. reintentar indefinidamente;
2. revisar la tabla Outbox solo cuando alguien reporte un problema;
3. marcar el evento procesado al crear una tarea manual;
4. mover todo el evento a dead-letter por el fallo de un único canal;
5. borrar eventos fallidos y recrearlos manualmente.

## Consecuencias positivas

- fallos visibles y asignables;
- estados coherentes entre trabajo automático y manual;
- conservación de éxitos parciales;
- recuperación reproducible y auditada;
- menor riesgo de duplicar operaciones.

## Consecuencias negativas

- más estados, métricas y reglas de agregación;
- se necesita mantener un runbook y responsables;
- dead-letter y tareas manuales requieren disciplina de cierre.

**Revisión requerida si:** Cambian los SLO, aumenta significativamente el volumen o la operación requiere automatizar más decisiones de recuperación.
