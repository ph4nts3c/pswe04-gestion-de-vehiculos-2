# ADR-001 — PostgreSQL como fuente de verdad interna

| Campo | Valor |
|---|---|
| **Estado** | Aceptado |
| **Fecha** | 2026-07-23 |
| **Autores** | Alejandro, William, Rodolfo |
| **Drivers relacionados** | Repositorio centralizado, trazabilidad, respuesta rápida |
| **Escenarios S07** | 1, 2, 3, 4 y 6 |

### Contexto

Precio, disponibilidad y estado pueden aparecer en varios canales. Los servicios externos pueden quedar temporalmente desactualizados o no estar disponibles.

### Decisión

PostgreSQL mantiene la versión oficial de vehículos, precios, estados, clientes potenciales, citas, reservas, publicaciones, metadatos de documentos, ventas, comisiones, auditoría y eventos Outbox.

Todos los cambios se realizan por la API central.

### Alternativas consideradas

1. usar cada sistema externo como fuente de verdad de su área;
2. conservar inventario en hojas de cálculo;
3. mantener una base separada por módulo desde la primera versión.

### Consecuencias positivas

- existe un único lugar para determinar la situación real del vehículo;
- las consultas no dependen de servicios externos;
- se facilita la auditoría;
- sitio público y backoffice consumen los mismos datos.

### Consecuencias negativas

- PostgreSQL se vuelve crítico para la operación;
- se necesitan respaldos, monitoreo y recuperación;
- hay que sincronizar cambios hacia canales externos.

---

**Revisión requerida si:** La disponibilidad, el volumen o el crecimiento del equipo exigen partición, réplicas o separación de datos por servicio.
