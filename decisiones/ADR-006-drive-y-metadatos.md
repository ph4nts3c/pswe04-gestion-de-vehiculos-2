# ADR-006 — Archivos en Google Drive y metadatos en PostgreSQL

| Campo | Valor |
|---|---|
| **Estado** | Aceptado |
| **Fecha** | 2026-07-23 |
| **Autores** | Alejandro, William, Rodolfo |
| **Drivers relacionados** | Seguridad, privacidad, repositorio centralizado |
| **Escenarios S07** | 6 |

### Contexto

Google Drive es adecuado para almacenar archivos, pero un archivo o enlace aislado no permite aplicar reglas de negocio ni saber su estado dentro del proceso.

### Decisión

El archivo físico se conserva en Drive. PostgreSQL guarda:

- identificador interno;
- vehículo asociado;
- tipo de documento;
- referencia de Drive;
- estado de revisión;
- clasificación de sensibilidad;
- responsable y fechas;
- información de auditoría necesaria.

Las reglas de completitud documental consultan metadatos internos.

### Alternativas consideradas

1. guardar binarios en PostgreSQL;
2. usar solo carpetas de Drive;
3. implementar almacenamiento propio desde el inicio.

### Consecuencias positivas

- Drive no se convierte en motor de reglas;
- se puede saber si la documentación está completa sin consultar Drive en cada operación;
- se mantiene trazabilidad;
- la base transaccional no almacena binarios grandes.

### Consecuencias negativas

- hay que detectar enlaces rotos;
- permisos de Drive y aplicación deben mantenerse alineados;
- se requieren verificaciones periódicas para archivos relevantes.

---

## Enmienda de entrega final

El acceso a documentos sensibles se media por la API; no se permiten enlaces públicos permanentes. Clasificación, autorización, reconciliación y respuesta ante permisos excesivos se detallan en ADR-010.

**Revisión requerida si:** Drive deja de cumplir requisitos de seguridad, volumen, retención o recuperación y se requiere almacenamiento dedicado.
