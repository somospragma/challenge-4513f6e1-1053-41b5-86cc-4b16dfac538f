# ADR 001: Idempotencia en Solicitudes de Préstamo

## Contexto
El sistema de préstamos debe garantizar que cada solicitud de préstamo sea procesada exactamente una vez, incluso si se reciben múltiples reintentos debido a fallos en la red o en el cliente. Esto es crítico para evitar duplicados en desembolsos y registros contables. Las solicitudes se identifican por un número de operación único por canal (web, móvil, sucursal) y deben ser idempotentes durante un periodo de 24 horas.

## Opciones Evaluadas

### Opción 1: Idempotencia basada en clave compuesta (número de operación + canal)
- **Descripción**: Usar una clave única compuesta por el número de operación y el canal para garantizar idempotencia.
- **Ventajas**: Simple de implementar, bajo overhead en almacenamiento.
- **Desventajas**: Requiere almacenamiento persistente de las claves durante 24 horas.
- **Riesgos**: Posible colisión si el número de operación no es único por canal.

### Opción 2: Idempotencia basada en tokens distribuidos
- **Descripción**: Generar un token único para cada solicitud y almacenarlo en una base de datos centralizada.
- **Ventajas**: Mayor garantía de unicidad, escalable.
- **Desventajas**: Mayor complejidad y latencia debido a la generación y validación de tokens.
- **Riesgos**: Dependencia de un servicio externo para la generación de tokens.

### Opción 3: Idempotencia basada en caché distribuida
- **Descripción**: Almacenar las claves de idempotencia en una caché distribuida (ej. Redis) con TTL de 24 horas.
- **Ventajas**: Bajo overhead en almacenamiento, escalable y rápido.
- **Desventajas**: Requiere infraestructura adicional para la caché.
- **Riesgos**: Posible pérdida de datos si la caché falla.

## Decisión
Se adopta la **Opción 3: Idempotencia basada en caché distribuida** (Redis) con TTL de 24 horas.
- **Razón**: Ofrece un equilibrio entre simplicidad, escalabilidad y garantía de idempotencia.
- **Implementación**: La clave será una combinación del número de operación y el canal, almacenada en Redis con un TTL de 24 horas.

## Consecuencias
- **Positivas**:
  - Garantiza idempotencia sin afectar el throughput del sistema.
  - Escalable para manejar el volumen de solicitudes.
- **Negativas**:
  - Dependencia de Redis para garantizar la idempotencia.
  - Requiere monitoreo del TTL para evitar duplicados después del periodo de 24 horas.

---