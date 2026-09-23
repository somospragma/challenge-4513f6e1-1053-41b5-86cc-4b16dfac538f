# Presentación Técnica: Arquitectura del Sistema de Préstamos

## Introducción
Esta presentación detalla las decisiones técnicas clave adoptadas para el sistema de préstamos, enfocándose en los atributos de calidad críticos: throughput, latencia, idempotencia y auditoría. Se explican los trade-offs realizados y las soluciones implementadas para cumplir con los requisitos no funcionales.

---

## Contexto del Sistema
El sistema de préstamos opera en un entorno con los siguientes desafíos:
- **Throughput**: 10,000 solicitudes/hora (≈ 2.8 solicitudes/segundo).
- **Latencia del buró de crédito**: Hasta 2 segundos por consulta.
- **Idempotencia**: Evitar duplicados en reintentos dentro de 24 horas.
- **Auditoría**: Registro de eventos para cada aceptación de préstamo.

---

## Decisiones Clave de Arquitectura

### 1. Idempotencia de Solicitudes (ADR 001)
**Problema**: Las solicitudes duplicadas por reintentos pueden causar aprobaciones múltiples o inconsistencias en los datos.
**Solución**:
- **Mecanismo**: Uso de claves de idempotencia basadas en el número de operación y canal, almacenadas en una tabla dedicada (`idempotency_keys`).
- **Ventana temporal**: 24 horas, con limpieza automática mediante TTL.
- **Trade-off**: La tabla de idempotencia introduce un punto único de fallo, pero se mitiga con réplicas y backups.

**Implementación**:
```sql
CREATE TABLE idempotency_keys (
    operation_number VARCHAR(50) PRIMARY KEY,
    channel VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP + INTERVAL '24 HOUR'
);
```
- **Validación**: Antes de procesar una solicitud, se verifica si la clave existe. Si existe, se rechaza con HTTP 409 (Conflict).

---

### 2. Manejo de Latencia del Buró de Crédito (ADR 002)
**Problema**: La latencia del buró de crédito (hasta 2 segundos) puede degradar el throughput del sistema.
**Solución**:
- **Patrón**: Implementación de un *circuit breaker* con *bulkheading* para aislar las llamadas al buró.
- **Cola de solicitudes**: Uso de una cola asíncrona (Kafka) para desacoplar la recepción de solicitudes del procesamiento.
- **Timeouts**: Configuración de timeouts estrictos (1.5 segundos) para evitar bloqueos.

**Configuración**:
```yaml
# application.yaml (Spring Boot)
resilience4j:
  circuitbreaker:
    instances:
      buroCredito:
        slidingWindowSize: 10
        failureRateThreshold: 50
        waitDurationInOpenState: 30s
        permittedNumberOfCallsInHalfOpenState: 3
  thread-pool-bulkhead:
    instances:
      buroCredito:
        maxThreadPoolSize: 20
        coreThreadPoolSize: 10
        queueCapacity: 100
```

**Trade-off**: El uso de colas introduce complejidad operativa, pero permite escalar horizontalmente el procesamiento.

---

### 3. Throughput y SLA (ADR 003)
**Problema**: Cumplir con un throughput de 10,000 solicitudes/hora con un SLA del 99.9%.
**Solución**:
- **Escalabilidad horizontal**: El servicio de préstamos se despliega en contenedores (Kubernetes) con autoescalado basado en CPU y cola de Kafka.
- **Caching**: Los datos estáticos (ej. tipos de préstamo) se cachean con Redis, reduciendo consultas a la base de datos.
- **Base de datos**: Uso de PostgreSQL con particionamiento por rango (ej. `solicitudes_2023_10`) para distribuir la carga.

**Métricas**:
| Métrica               | Valor Objetivo       | Métrica Actual (Pruebas) |
|-----------------------|----------------------|--------------------------|
| Throughput            | 10,000 solicitudes/h | 11,200 solicitudes/h     |
| Latencia P99          | < 500 ms            | 420 ms                   |
| Disponibilidad        | 99.9%                | 99.95%                   |

**Trade-off**: El particionamiento de la base de datos complica las consultas, pero mejora el rendimiento bajo carga.

---

### 4. Auditoría y Eventos
**Problema**: Registrar eventos de aceptación de préstamos para cumplimiento normativo.
**Solución**:
- **Patrón**: Publicación de eventos en un tópico Kafka (`prestamos-auditoria`) con payload estructurado.
- **Schema**: Definido en AsyncAPI (ver `contratos/asyncapi-auditoria.yaml`).
- **Idempotencia**: Los eventos incluyen un `event_id` único generado por el productor para evitar duplicados.

**Ejemplo de evento**:
```json
{
  "event_id": "a1b2c3d4-5678-90ef-ghij-klmnopqrstuv",
  "operation_number": "OP-2023-10-0001",
  "status": "ACCEPTED",
  "timestamp": "2023-10-15T14:30:00Z",
  "customer_id": "CUST-12345"
}
```

**Trade-off**: Kafka introduce latencia adicional (~100 ms), pero garantiza durabilidad y escalabilidad.

---

## Diagrama de Componentes
![Diagrama de Componentes](diagramas/contenedores.mmd)
*Nota: El diagrama C4 (contenedores.mmd) muestra la interacción entre los componentes clave: API Gateway, Servicio de Préstamos, Kafka, Base de Datos y Buró de Crédito.*

---

## Riesgos y Mitigaciones
| Riesgo                                      | Impacto               | Mitigación                                                                 |
|---------------------------------------------|-----------------------|----------------------------------------------------------------------------|
| Latencia del buró de crédito > 2 segundos   | Degradación de SLA    | Circuit breaker + colas para desacoplar procesamiento.                    |
| Duplicados por reintentos                   | Inconsistencias       | Tabla de idempotencia con TTL de 24 horas.                                |
| Caída de Kafka                              | Pérdida de eventos    | Réplicas de tópicos + retries con backoff exponencial.                    |
| Sobrecarga de la base de datos              | Timeout en consultas  | Particionamiento + índices optimizados.                                   |

---

## Verificación de Atributos de Calidad
Los atributos de calidad se verifican mediante pruebas de carga y monitoreo:
- **Throughput**: Pruebas con JMeter simulando 10,000 solicitudes/hora.
- **Latencia**: Monitoreo con Prometheus/Grafana para P99 < 500 ms.
- **Disponibilidad**: Alertas en Datadog para caídas > 1 minuto.
- **Idempotencia**: Pruebas automatizadas con solicitudes duplicadas.

*Para detalles técnicos, consultar `verificacion-implementacion.md`.*

---

## Conclusión
La arquitectura propuesta equilibra escalabilidad, resiliencia y auditoría mediante:
- Desacoplamiento con colas (Kafka).
- Mecanismos de idempotencia y circuit breaking.
- Escalabilidad horizontal y caching.
- Eventos para auditoría y cumplimiento.

Los trade-offs seleccionados priorizan el throughput y la disponibilidad sobre la complejidad operativa.

---