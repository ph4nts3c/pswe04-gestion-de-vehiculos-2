# ADR-009 — Operación controlada de canales manuales

| Campo | Valor |
|---|---|
| **Estado** | Aceptado |
| **Fecha** | 2026-08-06 |
| **Autores** | Alejandro, William, Rodolfo |
| **Versión** | 1.1 |
| **Drivers** | Consistencia con publicaciones, trazabilidad, canales heterogéneos |
| **Escenarios S07** | 2, 3 y 4 |

## Contexto

CRAutos, Facebook Marketplace o Encuentra24 pueden no ofrecer una API confiable para todas las operaciones. Una anotación informal no permite saber quién debe actualizar, cuándo vence, qué valor debe quedar visible ni cómo se confirma el resultado.

## Decisión

`ManualPublicationAdapter` crea una tarea idempotente con vehículo, publicación, acción, valor esperado, responsable, SLA, evidencia y verificador. Mientras la tarea está abierta, la acción de integración queda `TAREA_MANUAL_ABIERTA` y el evento Outbox `WAITING_MANUAL`; crear la tarea no equivale a completar la sincronización.

La verificación de evidencia cambia la acción a `EXITOSA`, actualiza la publicación y recalcula el evento. La tarea permanece visible al vencer. Las acciones críticas de precio o disponibilidad requieren verificación por una persona distinta. Una reconciliación periódica compara tareas y publicaciones con la fuente interna.

## Alternativas consideradas

1. manejar el trabajo por WhatsApp o correo;
2. marcar la acción como completada al crear la tarea;
3. excluir canales sin API del alcance;
4. automatizar con navegación no soportada o frágil.

## Consecuencias positivas

- la consistencia eventual incluye trabajo humano trazable;
- se pueden medir vencimientos y tiempos de ejecución;
- se evita prometer automatización inexistente;
- los cambios posteriores cancelan tareas incompatibles.

## Consecuencias negativas

- depende de disciplina operativa;
- requiere tablero, asignación y verificación;
- el marketplace todavía puede moderar o retrasar el cambio fuera del control del negocio.

**Revisión requerida si:** El canal obtiene una API estable o el SLA manual deja de ser sostenible.
