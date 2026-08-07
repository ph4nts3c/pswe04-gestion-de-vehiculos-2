# ADR-010 — Seguridad y privacidad de documentos almacenados en Google Drive

| Campo | Valor |
|---|---|
| **Estado** | Aceptado |
| **Fecha** | 2026-08-06 |
| **Autores** | Alejandro, William, Rodolfo |
| **Versión** | 1.1 |
| **Drivers** | Seguridad, privacidad, trazabilidad |
| **Escenarios S07** | 6 |

## Contexto

Los documentos del vehículo y del consignante contienen información sensible. Guardarlos en Drive reduce la carga de binarios en PostgreSQL, pero un enlace público, permiso heredado, archivo malicioso o acceso técnico amplio puede saltarse las reglas de la aplicación.

## Decisión

Drive conserva el contenido y PostgreSQL los metadatos. La API central media la carga y lectura síncrona después de autorizar por rol, recurso, acción y finalidad. El Procesador de integraciones ejecuta reconciliación de existencia y permisos.

No se permiten enlaces públicos permanentes para información interna, confidencial o restringida. Todo archivo debe superar validación de tipo, tamaño y contenido antes de quedar disponible. Si la inspección no está disponible, el archivo permanece `PENDIENTE_REVISION`.

Los accesos distinguen autorización concedida, transferencia completada y transferencia fallida. El administrador técnico no recibe acceso general al contenido; una excepción requiere documento específico, aprobación, motivo, vencimiento y auditoría. Reducir la clasificación de un documento restringido requiere dos roles distintos.

## Alternativas consideradas

1. compartir enlaces de Drive directamente;
2. usar únicamente permisos de carpetas como autorización;
3. guardar todos los binarios en PostgreSQL;
4. permitir acceso completo al administrador técnico;
5. aceptar cargas sin inspección cuando el validador no esté disponible.

## Consecuencias positivas

- control uniforme desde la API;
- menor riesgo de enlaces reutilizables;
- detección de desalineación entre permisos internos y Drive;
- archivos no verificados no quedan disponibles;
- evidencia precisa del acceso y la entrega.

## Consecuencias negativas

- la descarga depende de la API y del adaptador;
- se necesita reconciliación, inspección y respuesta a incidentes;
- clasificación, finalidades y retención deben mantenerse por tipo documental.

**Revisión requerida si:** Cambia el proveedor o una evaluación legal o de seguridad exige controles adicionales.
