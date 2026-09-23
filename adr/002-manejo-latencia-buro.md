# ADR 002: Manejo de Latencia del Buró de Crédito

## Contexto
El sistema debe integrarse con un buró de crédito externo que puede experimentar latencias de hasta 2 segundos en sus respuestas. Esta latencia no debe afectar el throughput del sistema (10,000 solicitudes/hora) ni el SLA de disponibilidad del 99.9%. La evaluación de crédito es un paso crítico en el flujo de aprobación de préstamos.

## Opciones Evaluadas

### Opción 1: Sincronía con timeout corto
- **Descripción**: Invocar al buró de crédito de manera síncrona con un timeout de 2 segundos.
- **Ventajas**: Simple de implementar, flujo lineal.
- **Desventajas**: Bloquea el hilo de procesamiento, afectando el throughput.
- **Riesgos**: Alto riesgo de fallos por timeout, especialmente en horas pico.

### Opción 2: Asincronía con cola de mensajes
- **Descripción**: Enviar las solicitudes al buró de crédito de manera asíncrona usando una cola de mensajes (ej. Kafka, SQS).
- **Ventajas**: Desacopla el procesamiento, mejora el throughput.
- **Desventajas**: Mayor complejidad en el manejo de estados y reintentos.
- **Riesgos**: Posible inconsistencia si la cola falla.

### Opción 3: Caché de respuestas del buró
- **Descripción**: Implementar una caché de respuestas del buró de crédito con TTL corto (ej. 5 minutos).
- **Ventajas**: Reduce la latencia para solicitudes repetidas.
- **Desventajas**: No resuelve el problema para solicitudes nuevas.
- **Riesgos**: Posible uso de datos obsoletos.

### Opción 4: Patrón Circuit Breaker
- **Descripción**: Usar un circuit breaker para manejar fallos del buró de crédito y degradar el servicio de manera controlada.
- **Ventajas**: Mejora la resiliencia del sistema.
- **Desventajas**: No resuelve el problema de latencia en sí.
- **Riesgos**: Requiere configuración cuidadosa para evitar falsos positivos.

## Decisión
Se adopta la **Opción 2: Asincronía con cola de mensajes** (Kafka) combinada con la **Opción 4: Patrón Circuit Breaker**.
- **Razón**: La asincronía permite manejar la latencia sin bloquear el flujo principal, mientras que el circuit breaker mejora la resiliencia.
- **Implementación**:
  - Las solicitudes al buró de crédito se enviarán a través de Kafka.
  - Se implementará un circuit breaker para manejar fallos del buró de crédito.
  - Se usará una caché con TTL corto (Opción 3) para reducir la carga en el buró.

## Consecuencias
- **Positivas**:
  - Mejora el throughput y la disponibilidad del sistema.
  - Reduce el impacto de la latencia del buró de crédito.
- **Negativas**:
  - Mayor complejidad en la implementación y monitoreo.
  - Requiere infraestructura adicional para Kafka.
  - Posible inconsistencia temporal en los datos.

---