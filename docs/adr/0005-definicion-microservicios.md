# ADR-0005: Definición de Microservicios y Bounded Contexts

## Estado
Propuesto

## Fecha
2025-11-10

## Contexto
La plataforma de gestión y publicación de eventos requiere una arquitectura de microservicios que refleje los bounded contexts del dominio. Cada microservicio debe tener responsabilidades claramente definidas, siguiendo principios de Domain-Driven Design (DDD) y manteniendo bajo acoplamiento con alta cohesión.

Se necesita establecer la lista completa de microservicios, sus tecnologías base, estrategias de persistencia y patrones de comunicación para garantizar escalabilidad, mantenibilidad y alineación con el lenguaje ubicuo.

## Decisión

### Bounded Contexts Identificados

Se han identificado **12 bounded contexts** principales que agrupan la funcionalidad del dominio:

1. **Gestión de Eventos** - Creación y publicación de eventos, gestión de venues
2. **Reservas y Ticketing** - Reservas temporales, generación de tickets, validación de acceso
3. **Pagos y Facturación** - Procesamiento de transacciones, emisión de facturas
4. **Identidad y Acceso** - Autenticación, autorización, gestión de usuarios
5. **Streaming y Contenido Digital** - Transmisiones en vivo, almacenamiento de contenido
6. **Comunidad y Moderación** - Foros, comentarios, moderación
7. **Servicios Complementarios** - Integración con proveedores externos (transporte, catering)
8. **Notificaciones** - Notificaciones en tiempo real, correos, recordatorios
9. **Reportes y Analítica** - Métricas, dashboards, recomendaciones
10. **Marketing y Promociones** - Cupones, descuentos
11. **Archivos y Multimedia** - Gestión de imágenes y documentos
12. **Gateway y Orquestación** - Enrutamiento, seguridad centralizada

### Arquitectura de Microservicios

Se definen **20 microservicios** distribuidos en los bounded contexts identificados:

| Microservicio | Bounded Context | Base de Datos | Tecnologías Clave |
|---|---|---|---|
| `events-service` | Gestión de Eventos | PostgreSQL | .NET 8, MediatR, EF Core |
| `venues-service` | Gestión de Eventos | PostgreSQL | .NET 8, EF Core |
| `tickets-service` | Ticketing | PostgreSQL | .NET 8, QR Generator |
| `reservations-service` | Ticketing | PostgreSQL + Redis (cache) | .NET 8, Hangfire (expiración) |
| `access-control-service` | Ticketing | PostgreSQL | .NET 8, SignalR |
| `payments-service` | Pagos | PostgreSQL | .NET 8, Hangfire (reintentos) |
| `invoicing-service` | Pagos | PostgreSQL | .NET 8, PDF Generator |
| `identity-service` | Identidad | Keycloak | Wrapper .NET |
| `users-service` | Identidad | PostgreSQL | .NET 8 |
| `streaming-service` | Streaming | MongoDB | .NET 8, SignalR, Firebase |
| `content-service` | Streaming | Firebase Storage | .NET 8 |
| `forums-service` | Comunidad | MongoDB | .NET 8 |
| `moderation-service` | Comunidad | MongoDB | .NET 8 |
| `ancillary-service` | Servicios Complementarios | PostgreSQL | .NET 8, RabbitMQ |
| `providers-adapter-service` | Servicios Complementarios | N/A (simulado) | .NET 8, RabbitMQ |
| `notifications-service` | Notificaciones | MongoDB (logs) | .NET 8, SignalR, SMTP |
| `analytics-service` | Analítica | MongoDB | .NET 8, Hangfire |
| `recommendations-service` | Analítica | MongoDB | .NET 8, ML.NET |
| `promotions-service` | Marketing | PostgreSQL | .NET 8 |
| `storage-service` | Archivos | Firebase Storage | .NET 8 |
| `api-gateway` | Gateway | N/A | YARP, Keycloak |

### Estrategia de Persistencia

#### Bases de Datos Transaccionales (PostgreSQL)
Utilizadas para microservicios que requieren consistencia fuerte, transacciones ACID y relaciones complejas:
- Gestión de eventos, venues, usuarios
- Pagos, facturación
- Reservas, tickets
- Servicios complementarios
- Promociones

#### Bases de Datos Documentales (MongoDB)
Utilizadas para microservicios que manejan datos no estructurados, alta escalabilidad de lectura y flexibilidad de esquema:
- Streaming (sesiones, logs de transmisión)
- Foros y comunidad (hilos, comentarios)
- Notificaciones (historial)
- Analítica (métricas agregadas)

#### Almacenamiento de Archivos (Firebase Storage)
Para contenido multimedia de gran tamaño:
- Imágenes de eventos
- Grabaciones de streaming
- Documentos y comprobantes

#### Cache Distribuido (Redis)
Para bloqueo optimista y cache de alta velocidad:
- Bloqueo de asientos en `reservations-service`
- Cache de consultas frecuentes

### Patrones de Comunicación

#### Comunicación Síncrona (HTTP/REST)
- Cliente → `api-gateway` → Microservicios
- Consultas de lectura (queries)
- Validaciones en tiempo real

#### Comunicación Asíncrona (RabbitMQ)
- Eventos de dominio entre microservicios
- Procesamiento de comandos diferidos
- Integración con proveedores externos
- Patrón Publish/Subscribe

#### Comunicación en Tiempo Real (SignalR)
- `notifications-service`: Push notifications a clientes
- `streaming-service`: Chat y actualizaciones de transmisión
- `access-control-service`: Validación de tickets en tiempo real

### Principios de Diseño

1. **Autonomía**: Cada microservicio es independiente y puede desplegarse por separado
2. **Base de datos por servicio**: No se comparten bases de datos entre microservicios
3. **Comunicación asíncrona preferida**: Para reducir acoplamiento temporal
4. **API Gateway centralizado**: Único punto de entrada para clientes externos
5. **Eventual consistency**: Aceptado entre bounded contexts mediante eventos de dominio
6. **Circuit breaker**: Implementado en integraciones críticas
7. **Observabilidad**: OpenTelemetry en todos los servicios

### Mapa de Comunicación

```
┌─────────────────┐
│   API Gateway   │
│     (YARP)      │
└────────┬────────┘
         │
         ├─────────────────────────────────────────────┐
         │                                             │
         ▼                                             ▼
┌─────────────────┐                          ┌──────────────────┐
│  Events Service │◄────RabbitMQ────────────►│ Payments Service │
└─────────────────┘                          └──────────────────┘
         │                                             │
         │                                             ▼
         ▼                                    ┌──────────────────┐
┌─────────────────┐                          │ Invoicing Service│
│  Venues Service │                          └──────────────────┘
└─────────────────┘                                   
         
         ▼                                             ▼
┌─────────────────┐                          ┌──────────────────┐
│Reservations Svc │◄────RabbitMQ────────────►│  Tickets Service │
└─────────────────┘                          └──────────────────┘
         │                                             │
         │ (Redis)                                     ▼
         │                                    ┌──────────────────┐
         │                                    │Access Control Svc│
         │                                    └──────────────────┘
         │
         ▼
┌─────────────────┐                          ┌──────────────────┐
│Notifications Svc│◄────RabbitMQ────────────►│ Streaming Service│
└─────────────────┘                          └──────────────────┘
         │                                             │
         │ (SignalR)                                   │ (SignalR)
         │                                             ▼
         │                                    ┌──────────────────┐
         │                                    │  Content Service │
         │                                    └──────────────────┘
         │
         ▼
┌─────────────────┐                          ┌──────────────────┐
│  Forums Service │◄────RabbitMQ────────────►│Moderation Service│
└─────────────────┘                          └──────────────────┘

┌─────────────────┐                          ┌──────────────────┐
│ Ancillary Svc   │◄────RabbitMQ────────────►│ Providers Adapter│
└─────────────────┘                          └──────────────────┘

┌─────────────────┐
│ Analytics Svc   │ (Query all services)
└─────────────────┘
```

## Consecuencias

### Positivas
- **Escalabilidad independiente**: Cada servicio puede escalar según su carga específica
- **Despliegue autónomo**: Reducción de riesgo en deployments
- **Tecnologías específicas**: Elección de base de datos óptima por caso de uso
- **Equipos autónomos**: Facilitación de desarrollo paralelo
- **Resiliencia**: Fallo en un servicio no afecta al resto del sistema
- **Alineación con DDD**: Bounded contexts claros reflejados en arquitectura

### Negativas
- **Complejidad operacional**: Requiere orquestación de contenedores (Kubernetes)
- **Transacciones distribuidas**: Necesidad de implementar Saga pattern
- **Consistencia eventual**: Requiere manejo cuidadoso de estados intermedios
- **Duplicación de datos**: Algunos datos pueden replicarse entre servicios
- **Debugging complejo**: Trazabilidad distribuida requiere herramientas especializadas
- **Latencia de red**: Comunicación entre servicios añade overhead

### Riesgos Identificados
1. **Cascada de fallos**: Mitigado con circuit breakers y timeouts
2. **Inconsistencia de datos**: Mitigado con eventos de dominio idempotentes
3. **Versionado de APIs**: Requiere estrategia de versionado semántico
4. **Observabilidad**: Requiere inversión en OpenTelemetry y logging centralizado

## Alternativas Consideradas

### Monolito Modular
- **Ventaja**: Menor complejidad operacional
- **Desventaja**: Escalabilidad limitada, deployments de alto riesgo
- **Razón de rechazo**: No cumple requisitos de escalabilidad independiente

### Microservicios + CQRS con Event Sourcing
- **Ventaja**: Trazabilidad completa, proyecciones optimizadas
- **Desventaja**: Complejidad de implementación muy alta
- **Razón de rechazo**: Overkill para la fase inicial del proyecto

### Arquitectura Serverless
- **Ventaja**: Escalabilidad automática, costos por uso
- **Desventaja**: Vendor lock-in, latencias cold-start
- **Razón de rechazo**: Requiere rediseño completo para Azure Functions/AWS Lambda

## Referencias
- [ADR-0002: Lenguaje Ubicuo Global](./0002-lenguaje-ubiquo-global.md)
- [ADR-0003: Stack Tecnológico](./0003-stack-tecnologico.md)
- [ADR-0004: Microservice Template](./0004-microservice-template.md)
- [ADR-0005: Arquitectura de Microservicios](./0005-definicion-microservicios.md)
- [Lenguaje Ubicuo](../lenguaje-ubicuo.md)
- [Guía Técnica](../guia-tecnica.md)

## Notas
Este ADR debe ser revisado y aprobado antes de iniciar el desarrollo de cada microservicio. La documentación detallada de cada servicio se mantiene en `docs/services/`.
