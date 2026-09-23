# Riesgos Técnicos y Mitigaciones

## Priorización de Riesgos

Los riesgos se priorizan según **impacto** (1-5) y **probabilidad** (1-5), calculando el **riesgo total** como `impacto × probabilidad`.

| Riesgo                                                                 | Impacto | Probabilidad | Riesgo Total | Prioridad |
|------------------------------------------------------------------------|---------|--------------|--------------|-----------|
| Fallo del Buró de Crédito                                              | 5       | 3            | 15           | Alta      |
| Saturación del throughput (más de 10,000 solicitudes/hora)              | 4       | 4            | 16           | Alta      |
| Pérdida de eventos de auditoría                                        | 3       | 2            | 6            | Media     |
| Inconsistencia en datos de solicitudes (duplicados o perdidos)         | 4       | 2            | 8            | Media     |
| Latencia excesiva en Loan Approval Service                             | 3       | 3            | 9            | Media     |
| Fallo de la caché de idempotencia (Redis)                              | 2       | 3            | 6            | Baja      |

## Riesgos Priorizados

### 1. Fallo del Buró de Crédito

#### Impacto
- **Operativo**: El Loan Request Service no puede evaluar solicitudes, causando rechazos automáticos.
- **Negocio**: Pérdida de ingresos por préstamos no aprobados y mala experiencia de cliente.

#### Causas
- Caída del servicio externo.
- Latencia superior al timeout (2 segundos).
- Bloqueo por rate limiting del Buró.

#### Mitigaciones
| Mitigación                                                                 | Responsable          | Estado       |
|---------------------------------------------------------------------------|----------------------|--------------|
| Implementar caché de respuestas del Buró (TTL: 5 minutos).                | Equipo de Desarrollo | Implementado |
| Usar circuit breaker (Resilience4j) para fallar rápido.                   | Equipo de Desarrollo | Implementado |
| Fallback a score crediticio histórico (últimos 30 días) si el Buró falla. | Equipo de Desarrollo | Implementado |
| Monitorear disponibilidad del Buró con alertas en Datadog.                | Equipo de Operaciones | Pendiente    |

#### Métricas de Éxito
- **Tiempo de recuperación**: ≤ 1 minuto después de fallo del Buró.
- **Porcentaje de solicitudes procesadas**: ≥ 95% durante fallo del Buró.

---

### 2. Saturación del Throughput

#### Impacto
- **Operativo**: Degradación del rendimiento o caída de servicios.
- **Negocio**: Incumplimiento del SLA de 10,000 solicitudes/hora.

#### Causas
- Pico de demanda inesperado.
- Cuellos de botella en Loan Request Service o bases de datos.
- Latencia alta en llamadas al Buró.

#### Mitigaciones
| Mitigación                                                                 | Responsable          | Estado       |
|---------------------------------------------------------------------------|----------------------|--------------|
| Escalado horizontal automático en EKS (basado en CPU/memoria).            | Equipo de Operaciones | Implementado |
| Optimizar queries a la base de datos (índices, consultas preparadas).     | Equipo de Desarrollo | Implementado |
| Implementar rate limiting en API Gateway (100 solicitudes/segundo/cliente).| Equipo de Desarrollo | Implementado |
| Usar particiones adicionales en Kafka para manejar volumen de eventos.    | Equipo de Operaciones | Pendiente    |

#### Métricas de Éxito
- **Throughput**: ≥ 10,000 solicitudes/hora durante picos.
- **Latencia p95**: ≤ 3 segundos.

---

### 3. Pérdida de Eventos de Auditoría

#### Impacto
- **Operativo**: Falta de trazabilidad para auditorías internas.
- **Negocio**: Incumplimiento de regulaciones (ej. SOX).

#### Causas
- Fallo en Kafka (broker caído).
- Problemas en el consumidor (Audit Service).
- Commit manual de offsets fallido.

#### Mitigaciones
| Mitigación                                                                 | Responsable          | Estado       |
|---------------------------------------------------------------------------|----------------------|--------------|
| Configurar Kafka con replicación (factor de replicación: 3).              | Equipo de Operaciones | Implementado |
| Implementar dead letter queue para eventos no procesados.                 | Equipo de Desarrollo | Implementado |
| Monitorear lag de consumidores con alertas en Prometheus.                 | Equipo de Operaciones | Pendiente    |
| Backups diarios de la base de datos de auditoría (MongoDB).               | Equipo de Operaciones | Implementado |

#### Métricas de Éxito
- **Eventos perdidos**: 0 eventos perdidos por mes.
- **Tiempo de recuperación**: ≤ 5 minutos para reprocesar eventos.

---

## Riesgos de Prioridad Media

### 4. Inconsistencia en Datos de Solicitudes

#### Impacto
- **Operativo**: Solicitudes duplicadas o perdidas.
- **Negocio**: Riesgo de fraude o errores en aprobaciones.

#### Causas
- Fallo en la caché de idempotencia (Redis).
- Race conditions en el Loan Request Service.
- Errores en transacciones de la base de datos.

#### Mitigaciones
- Usar Redis con persistencia (RDB + AOF).
- Implementar transacciones ACID en PostgreSQL.
- Monitorear métricas de Redis (evictions, latencia).

---

### 5. Latencia Excesiva en Loan Approval Service

#### Impacto
- **Operativo**: Tiempo de respuesta total > 3 segundos.
- **Negocio**: Experiencia de usuario pobre.

#### Causas
- Lógica de aprobación compleja.
- Cuellos de botella en la base de datos de reglas.

#### Mitigaciones
- Optimizar reglas de negocio (usar reglas precalculadas).
- Cachear reglas de aprobación en memoria (Caffeine).
- Escalar horizontalmente el Loan Approval Service.

---

## Plan de Mitigación General

1. **Monitoreo Proactivo**
   - Implementar dashboards en Grafana para métricas clave (throughput, latencia, disponibilidad).
   - Configurar alertas en Datadog para umbrales críticos.

2. **Pruebas de Carga**
   - Ejecutar pruebas de carga mensuales con JMeter para validar el throughput.
   - Simular fallos del Buró de Crédito para probar circuit breakers.

3. **Documentación Operativa**
   - Crear runbooks para cada riesgo priorizado (ej. "Qué hacer si falla el Buró").
   - Capacitar al equipo de operaciones en los procedimientos de mitigación.

4. **Revisión Trimestral**
   - Reevaluar riesgos cada 3 meses y ajustar prioridades.
   - Actualizar mitigaciones según cambios en el sistema o requisitos.