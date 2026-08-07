# ADR-002 — Separar el ciclo de vida del vehículo de clientes potenciales y citas

| Campo | Valor |
|---|---|
| **Estado** | Aceptado |
| **Fecha** | 2026-07-23 |
| **Autores** | Alejandro, William, Rodolfo |
| **Drivers relacionados** | Ciclo de vida, consistencia de estado, trazabilidad |
| **Escenarios S07** | 3 y 5 |

### Contexto

S07 describió `CON_CLIENTE_POTENCIAL_ACTIVO` y `CITA_AGENDADA` como hitos del proceso. En diseño detallado, un vehículo puede tener varios interesados y varias citas de manera concurrente.

### Decisión

El estado del vehículo representa solamente su condición comercial:

```text
INGRESADO -> PENDIENTE_DOCUMENTACION -> PENDIENTE_FOTOS -> LISTO_PARA_PUBLICAR -> PUBLICADO -> RESERVADO -> VENDIDO
```

`RETIRADO` funciona como salida permitida desde estados definidos.

Los clientes potenciales, citas y reservas tienen ciclos de vida propios y se relacionan con el vehículo por identificador.

### Alternativas consideradas

1. mantener todas las actividades en un único enum de `Vehicle.state`;
2. calcular el estado del vehículo únicamente a partir de citas, leads y reservas;
3. usar un motor BPM para modelar todo el proceso.

### Consecuencias positivas

- permite varios interesados y varias citas simultáneas;
- el estado del vehículo expresa disponibilidad real;
- reduce transiciones artificiales como `CITA_AGENDADA -> PUBLICADO`;
- simplifica reglas de publicación y reserva.

### Consecuencias negativas

- aumenta la cantidad de entidades y estados;
- algunas pantallas deben combinar información de vehículo, leads y citas;
- los reportes de “etapa comercial” requieren una proyección, no solo leer `Vehicle.state`.

---

**Revisión requerida si:** El negocio adopta un proceso configurable que requiera un motor de workflow o reglas dinámicas por tipo de vehículo.
