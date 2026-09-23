# ADR 003: Garantizar Throughput de 10,000 Solicitudes/Hora con SLA de 99.9%

## Contexto
El sistema debe procesar 10,000 solicitudes de préstamo por hora con un SLA de disponibilidad del 99.9%. Esto implica que el sistema debe ser altamente escalable y resiliente, capaz de manejar picos de carga sin degradar el rendimiento.

## Opciones Evaluadas

### Opción 1: Escalado vertical
- **Descripción**: Aumentar los recursos de los servidores (CPU, RAM) para manejar la carga.
- **Ventajas**: Simple de implementar.
- **Desventajas**: Costoso y limitado por la capacidad máxima de los servidores.
- **Riesgos**: No es escalable a largo plazo.

### Opción 2: Escalado horizontal con balanceo de carga
- **Descripción**: Distribuir la carga entre múltiples instancias del servicio usando un balanceador de carga.
- **Ventajas**: Escalable y resiliente.
- **Desventajas**: Mayor complejidad en la gestión de estado y sincronización.
- **Riesgos**: Dependencia del balanceador de carga.

### Opción 3: Arquitectura basada en eventos
- **Descripción**: Usar una arquitectura basada en eventos para desacoplar componentes y manejar la carga de manera asíncrona.
- **Ventajas**: Alta escalabilidad y resiliencia.
- **Desventajas**: Complejidad en el manejo de estados y garantía de entrega.
- **Riesgos**: Posible pérdida de eventos si no se implementa correctamente.

### Opción 4: Microservicios con isolation
- **Descripción**: Dividir el sistema en microservicios independientes, cada uno escalable horizontalmente.
- **Ventajas**: Alta escalabilidad y aislamiento de fallos.
- **Desventajas**: Mayor complejidad en la orquestación y monitoreo.
- **Riesgos**: Posible latencia en la comunicación entre servicios.

## Decisión
Se adopta la **Opción 2: Escalado horizontal con balanceo de carga** combinada con la **Opción 3: Arquitectura basada en eventos**.
- **Razón**: El escalado horizontal permite manejar la carga de manera eficiente, mientras que la arquitectura basada en eventos mejora la resiliencia y el throughput.
- **Implementación**:
  - El sistema se desplegará en múltiples instancias detrás de un balanceador de carga.
  - Se usará Kafka para manejar eventos asíncronos (ej. solicitudes al buró de crédito, notificaciones de auditoría).
  - Se implementarán réplicas de las bases de datos para garantizar alta disponibilidad.

## Consecuencias
- **Positivas**:
  - Cumple con el throughput y SLA requeridos.
  - Escalable para manejar picos de carga.
  - Resiliente ante fallos.
- **Negativas**:
  - Mayor complejidad en la implementación y monitoreo.
  - Requiere infraestructura adicional para Kafka y balanceadores de carga.
  - Posible latencia en la comunicación entre componentes.