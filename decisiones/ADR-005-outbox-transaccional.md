# ADR-005 — Publicaciones con estado propio y Outbox transaccional

| Campo | Valor |
|---|---|
| **Estado** | Aceptado |
| **Fecha** | 2026-07-23 |
| **Autores** | Alejandro, William, Rodolfo |
| **Drivers relacionados** | Consistencia con publicaciones externas, resiliencia |
| **Escenarios S07** | 2, 3 y 4 |

### Contexto

Una reserva, venta, retiro o cambio de precio debe quedar confirmado aunque un marketplace no responda.

### Decisión

Cada publicación externa tiene estado propio:

```text
ACTUALIZADA
PENDIENTE
EN_REINTENTO
MANUAL
FALLIDA_PERMANENTE
CERRADA
```

La API escribe el cambio de negocio y el evento Outbox en la misma transacción. El procesador de integraciones ejecuta después la actualización externa.

### Alternativas consideradas

1. esperar la API externa dentro de la solicitud del usuario;
2. confirmar el dominio y publicar un mensaje después sin garantía transaccional;
3. introducir un broker de mensajería adicional desde el inicio.

### Consecuencias positivas

- el negocio no depende del tiempo de respuesta de terceros;
- el evento no se pierde entre `COMMIT` y publicación;
- se soportan reintentos e idempotencia;
- el pendiente queda visible.

### Consecuencias negativas

- existe consistencia eventual;
- se necesita un worker y monitoreo;
- aparecen estados operativos adicionales.

---

## Enmienda de entrega final

La operación se completa con estados de evento, dead-letter, alertas y recuperación definidos en ADR-008.

**Revisión requerida si:** El volumen o la latencia requerida supera el sondeo sobre PostgreSQL y justifica un broker de mensajería.
