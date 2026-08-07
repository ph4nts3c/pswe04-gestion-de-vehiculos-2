# ADR-007 — Auditoría de solo inserción para cambios sensibles

| Campo | Valor |
|---|---|
| **Estado** | Aceptado |
| **Fecha** | 2026-07-23 |
| **Autores** | Alejandro, William, Rodolfo |
| **Drivers relacionados** | Trazabilidad comercial, seguridad |
| **Escenarios S07** | 2, 3 y 6 |

### Contexto

Cambios de precio, estado, reserva, documentos, citas y publicaciones deben conservar responsable, fecha y motivo cuando aplique.

### Decisión

Se utiliza una bitácora funcional de solo inserción. Cada registro conserva como mínimo:

- entidad e identificador;
- acción;
- actor;
- fecha/hora;
- valor anterior y nuevo cuando corresponda;
- motivo;
- origen;
- `correlationId`.

La bitácora no reemplaza los logs técnicos.

### Alternativas consideradas

1. confiar únicamente en logs de aplicación;
2. usar solo `updated_at` y `updated_by`;
3. adoptar Event Sourcing completo.

### Consecuencias positivas

- permite reconstruir cambios sensibles;
- facilita investigar diferencias de precio o estado;
- soporta trazabilidad sin exigir Event Sourcing.

### Consecuencias negativas

- aumenta el volumen de datos;
- hay que evitar guardar secretos o datos personales innecesarios;
- se debe controlar quién puede consultar la bitácora.

---

**Revisión requerida si:** Una auditoría externa exige almacenamiento WORM, firma criptográfica o reconstrucción completa mediante Event Sourcing.
