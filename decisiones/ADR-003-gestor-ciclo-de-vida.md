# ADR-003 — El Gestor del ciclo de vida controla toda transición de vehículo

| Campo | Valor |
|---|---|
| **Estado** | Aceptado |
| **Fecha** | 2026-07-23 |
| **Autores** | Alejandro, William, Rodolfo |
| **Drivers relacionados** | Ciclo de vida, seguridad, auditoría |
| **Escenarios S07** | 2, 3 y 6 |

### Contexto

El estado del vehículo controla qué operaciones son válidas. Un `UPDATE state='VENDIDO'` directo permitiría saltarse permisos, precondiciones, auditoría y efectos sobre publicaciones.

### Decisión

Todo cambio de estado pasa por `VehicleLifecycleService`, que valida:

- transición permitida;
- autorización del actor;
- versión concurrente del vehículo;
- precondiciones del dominio;
- cambios relacionados;
- auditoría;
- evento Outbox cuando existan efectos externos.

No se expone un endpoint CRUD para modificar `state` directamente.

### Alternativas consideradas

1. validar únicamente en la interfaz;
2. distribuir las reglas entre controladores y servicios;
3. usar un motor de workflow desde la primera versión.

### Consecuencias positivas

- existe un único punto para proteger las transiciones;
- las reglas pueden probarse unitariamente;
- una pantalla o integración no puede saltarse el ciclo de vida;
- el cambio se coordina con auditoría y publicaciones.

### Consecuencias negativas

- el componente concentra reglas importantes;
- cambiar el ciclo de vida exige actualizar pruebas y políticas;
- debe evitarse convertir el servicio en una clase con demasiadas responsabilidades.

---

**Revisión requerida si:** Las políticas de transición dejan de ser estables, varían por unidad de negocio o el componente concentra responsabilidades ajenas al ciclo de vida.
