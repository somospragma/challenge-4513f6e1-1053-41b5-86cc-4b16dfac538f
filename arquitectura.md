# Arquitectura del Sistema de Préstamos

## Vista de Componentes

El sistema de préstamos se estructura en los siguientes componentes principales, cada uno con límites claros y responsabilidades definidas:

### 1. **API Gateway (Punto de Entrada)**
- **Responsabilidad**: Routing de solicitudes, autenticación, rate limiting y validación básica de payloads.
- **Límites**:
  - Externo: Clientes (apps móviles, portales web).
  - Interno: Servicio de Solicitudes.
- **Contratos**:
  - Entrada: `POST /loans/requests` (OpenAPI `contratos/openapi-prestamos.yaml`).
  - Salida: Eventos de auditoría (AsyncAPI `contratos/asyncapi-auditoria.yaml`).

### 2. **Servicio de Solicitudes (Loan Request Service)**
- **Responsabilidad**: Orquestación del flujo de solicitud de préstamos (validación, evaluación, aprobación).
- **Límites**:
  - Externo: API Gateway, Buró de Crédito, Servicio de Aprobación.
  - Interno: Base de Datos de Solicitudes.
- **Contratos**:
  - Entrada: Payload validado desde API Gateway.
  - Salida: Llamadas a Buró de Crédito (API REST) y Servicio de Aprobación (gRPC).
  - Eventos: Publicación de eventos de auditoría (Kafka).

### 3. **Servicio de Aprobación (Loan Approval Service)**
- **Responsabilidad**: Lógica de aprobación basada en reglas de negocio (score crediticio, historial).
- **Límites**:
  - Externo: Servicio de Solicitudes.
  - Interno: Base de Datos de Reglas.
- **Contratos**:
  - Entrada: Datos del cliente y solicitud (gRPC).
  - Salida: Decisión de aprobación/rechazo.

### 4. **Buró de Crédito (External Credit Bureau)**
- **Responsabilidad**: Proveer datos crediticios del cliente.
- **Límites**: Externo (tercerizado).
- **Contratos**:
  - Entrada: `GET /credit-scores/{clientId}` (API REST).
  - Salida: Score crediticio y historial.

### 5. **Servicio de Auditoría (Audit Service)**
- **Responsabilidad**: Registro de eventos críticos (aprobaciones, rechazos, errores).
- **Límites**:
  - Externo: Todos los componentes internos.
  - Interno: Base de Datos de Auditoría.
- **Contratos**:
  - Entrada: Eventos desde Kafka (AsyncAPI `contratos/asyncapi-auditoria.yaml`).

### 6. **Base de Datos (Databases)**
- **Componentes**:
  - **Solicitudes**: Almacena solicitudes pendientes/aprobadas (PostgreSQL).
  - **Reglas**: Almacena reglas de aprobación (PostgreSQL).
  - **Auditoría**: Almacena eventos (MongoDB).
- **Límites**: Internos (accesibles solo por servicios correspondientes).

## Vista de Despliegue

```mermaid
C4Deployment
  title Vista de Despliegue del Sistema de Préstamos

  Deployment_Node(cloud, "AWS Cloud", "Region: us-east-1") {
    Deployment_Node(api_gw, "API Gateway", "Kong") {
      Container(api, "API Gateway", "Kong", "Routing, autenticación, rate limiting")
    }

    Deployment_Node(eks, "EKS Cluster", "Kubernetes") {
      Container(loan_request, "Loan Request Service", "Java/Spring Boot", "Orquestación de solicitudes")
      Container(loan_approval, "Loan Approval Service", "Java/Spring Boot", "Lógica de aprobación")
      ContainerDb(db_loans, "PostgreSQL", "Base de Datos de Solicitudes")
      ContainerDb(db_rules, "PostgreSQL", "Base de Datos de Reglas")
      Container(kafka, "Kafka", "Event Streaming")
    }

    Deployment_Node(audit, "Audit Service", "Node.js") {
      Container(audit_svc, "Audit Service", "Node.js", "Registro de eventos")
      ContainerDb(db_audit, "MongoDB", "Base de Datos de Auditoría")
    }
  }

  System_Ext(credit_bureau, "Buró de Crédito", "Servicio externo")

  Rel(api_gw, loan_request, "HTTP")
  Rel(loan_request, credit_bureau, "HTTP (timeout: 2s)")
  Rel(loan_request, loan_approval, "gRPC")
  Rel(loan_request, db_loans, "JDBC")
  Rel(loan_approval, db_rules, "JDBC")
  Rel(loan_request, kafka, "Producción de eventos")
  Rel(kafka, audit_svc, "Consumo de eventos")
  Rel(audit_svc, db_audit, "MongoDB")
```

## Contratos entre Componentes

### 1. **API Gateway ↔ Loan Request Service**
- **Protocolo**: HTTP/REST.
- **Contrato**: `contratos/openapi-prestamos.yaml` (ejemplo de payload):
  ```yaml
  paths:
    /loans/requests:
      post:
        requestBody:
          content:
            application/json:
              schema:
                type: object
                properties:
                  operationNumber:
                    type: string
                    example: "OP-2023-0001"
                  clientId:
                    type: string
                    example: "CL-12345"
                  amount:
                    type: number
                    example: 5000.00
                  channel:
                    type: string
                    enum: [MOBILE, WEB, BRANCH]
        responses:
          202:
            description: Solicitud aceptada para procesamiento.
  ```

### 2. **Loan Request Service ↔ Credit Bureau**
- **Protocolo**: HTTP/REST.
- **Contrato**:
  ```yaml
  paths:
    /credit-scores/{clientId}:
      get:
        parameters:
          - name: clientId
            in: path
            required: true
            schema:
              type: string
        responses:
          200:
            content:
              application/json:
                schema:
                  type: object
                  properties:
                    score:
                      type: integer
                      example: 750
                    history:
                      type: array
                      items:
                        type: object
  ```

### 3. **Loan Request Service ↔ Loan Approval Service**
- **Protocolo**: gRPC.
- **Contrato**: Definido en `.proto` (no incluido aquí, pero referenciado en el diseño).

### 4. **Loan Request Service → Audit Service**
- **Protocolo**: Kafka (eventos).
- **Contrato**: `contratos/asyncapi-auditoria.yaml`:
  ```yaml
  channels:
    loan-events:
      subscribe:
        message:
          payload:
            type: object
            properties:
              eventId:
                type: string
                example: "EVT-2023-0001"
              operationNumber:
                type: string
                example: "OP-2023-0001"
              status:
                type: string
                enum: [APPROVED, REJECTED, ERROR]
              timestamp:
                type: string
                format: date-time
  ```

## Límites de Responsabilidad

| Componente               | Responsabilidades                                                                 | Límites (Externo)                     | Límites (Interno)                     |
|---------------------------|----------------------------------------------------------------------------------|---------------------------------------|---------------------------------------|
| API Gateway               | Routing, autenticación, rate limiting.                                            | Clientes                               | Loan Request Service                  |
| Loan Request Service      | Orquestación, validación, idempotencia, llamadas a Buró y Aprobación.            | API Gateway, Credit Bureau, Approval  | Bases de datos, Kafka                 |
| Loan Approval Service     | Lógica de aprobación basada en reglas.                                             | Loan Request Service                   | Base de Datos de Reglas               |
| Credit Bureau              | Proveer datos crediticios.                                                         | Loan Request Service                   | Ninguno                               |
| Audit Service             | Registro de eventos críticos.                                                      | Kafka                                  | Base de Datos de Auditoría            |

## Trade-offs Arquitectónicos

1. **Idempotencia vs. Complejidad**
   - **Decisión**: Implementar idempotencia mediante caché de operaciones (24 horas).
   - **Trade-off**: Aumenta complejidad en el Loan Request Service, pero elimina duplicados.
   - **Justificación**: Requerimiento explícito del negocio (ADR `adr/001-idempotencia-solicitudes.md`).

2. **Latencia del Buró de Crédito**
   - **Decisión**: Timeout de 2 segundos para llamadas al Buró.
   - **Trade-off**: Puede causar rechazos prematuros si el Buró responde lento.
   - **Justificación**: SLA de throughput (10,000 solicitudes/hora) no permite esperas largas (ADR `adr/002-manejo-latencia-buro.md`).

3. **Throughput vs. Consistencia**
   - **Decisión**: Uso de Kafka para eventos de auditoría (eventual consistency).
   - **Trade-off**: Desacople entre Loan Request Service y Audit Service, pero posible inconsistencia temporal.
   - **Justificación**: Priorizar disponibilidad sobre consistencia fuerte (ADR `adr/003-throughput-sla.md`).

---