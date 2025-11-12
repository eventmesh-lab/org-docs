# Microservicios de la Plataforma

Este directorio contiene la documentación detallada de cada uno de los microservicios que componen la plataforma de gestión y publicación de eventos.

## 📋 Arquitectura General

La plataforma está organizada en **12 bounded contexts** implementados mediante **20 microservicios** independientes. Para una visión completa de la arquitectura, consulta:

- [ADR-0005: Definición de Microservicios y Bounded Contexts](../adr/0005-definicion-microservicios.md)

---

## 🎯 Bounded Contexts y Servicios

### 1. Gestión de Eventos
- **[Events Service](events-service.md)** - Gestión del ciclo de vida completo de eventos
- **[Venues Service](venues-service.md)** - Gestión de recintos físicos, zonas y asientos

### 2. Reservas y Ticketing
- **[Reservations Service](reservations-service.md)** - Bloqueo temporal de asientos, expiración automática
- **[Tickets Service](tickets-service.md)** - Generación y validación de tickets con código QR
- **Access Control Service** *(pendiente)* - Validación de acceso en puerta

### 3. Pagos y Facturación
- **[Payments Service](payments-service.md)** - Procesamiento de transacciones, reintentos automáticos
- **Invoicing Service** *(pendiente)* - Generación de facturas PDF, conciliación

### 4. Identidad y Acceso
- **Identity Service** *(pendiente)* - Integración con Keycloak
- **Users Service** *(pendiente)* - Gestión de perfiles y roles

### 5. Streaming y Contenido Digital
- **[Streaming Service](streaming-service.md)** - Transmisiones en vivo, control de acceso
- **Content Service** *(pendiente)* - Almacenamiento de grabaciones

### 6. Comunidad y Moderación
- **[Forums Service](forums-service.md)** - Foros de discusión por evento
- **Moderation Service** *(pendiente)* - Auditoría y moderación de contenido

### 7. Servicios Complementarios
- **Ancillary Service** *(pendiente)* - Orquestación de servicios externos (transporte, catering)
- **Providers Adapter Service** *(pendiente)* - Simulación de APIs de proveedores

### 8. Notificaciones
- **Notifications Service** *(pendiente)* - SignalR hub, correos, push notifications

### 9. Reportes y Analítica
- **Analytics Service** *(pendiente)* - Reportes, métricas, dashboards
- **Recommendations Service** *(pendiente)* - Algoritmos de recomendación

### 10. Marketing y Promociones
- **Promotions Service** *(pendiente)* - Cupones, descuentos, campañas

### 11. Archivos y Multimedia
- **Storage Service** *(pendiente)* - Proxy a Firebase Storage

### 12. Gateway y Orquestación
- **[API Gateway](api-gateway.md)** - YARP / svc_yarp_api-gateway (documentado en este portal)

---

## 🗂️ Estructura de Documentación

Cada microservicio documentado incluye:

1. **Descripción** - Propósito y responsabilidades
2. **Modelo de Dominio** - Agregados, entidades y value objects
3. **Comandos del Dominio** - Operaciones principales
4. **Eventos de Dominio** - Eventos publicados y consumidos
5. **Reglas de Negocio** - Invariantes y políticas
6. **Integraciones** - Comunicación con otros servicios
7. **Persistencia** - Esquema de base de datos
8. **API Endpoints** - Contratos HTTP
9. **Tecnologías** - Stack técnico específico
10. **Observabilidad** - Métricas, logs y traces

---

## 📊 Tabla de Resumen

| Microservicio | Bounded Context | Base de Datos | Estado Documentación |
|---|---|---|---|
| `events-service` | Gestión de Eventos | PostgreSQL | ✅ Completo |
| `venues-service` | Gestión de Eventos | PostgreSQL | ✅ Completo |
| `tickets-service` | Ticketing | PostgreSQL | ✅ Completo |
| `reservations-service` | Ticketing | PostgreSQL + Redis | ✅ Completo |
| `access-control-service` | Ticketing | PostgreSQL | ⏳ Pendiente |
| `payments-service` | Pagos | PostgreSQL | ✅ Completo |
| `invoicing-service` | Pagos | PostgreSQL | ⏳ Pendiente |
| `identity-service` | Identidad | Keycloak | ⏳ Pendiente |
| `users-service` | Identidad | PostgreSQL | ⏳ Pendiente |
| `streaming-service` | Streaming | MongoDB | ✅ Completo |
| `content-service` | Streaming | Firebase Storage | ⏳ Pendiente |
| `forums-service` | Comunidad | MongoDB | ✅ Completo |
| `moderation-service` | Comunidad | MongoDB | ⏳ Pendiente |
| `ancillary-service` | Servicios Complementarios | PostgreSQL | ⏳ Pendiente |
| `providers-adapter-service` | Servicios Complementarios | N/A | ⏳ Pendiente |
| `notifications-service` | Notificaciones | MongoDB | ⏳ Pendiente |
| `analytics-service` | Analítica | MongoDB | ⏳ Pendiente |
| `recommendations-service` | Analítica | MongoDB | ⏳ Pendiente |
| `promotions-service` | Marketing | PostgreSQL | ⏳ Pendiente |
| `storage-service` | Archivos | Firebase Storage | ⏳ Pendiente |
| `api-gateway` | Gateway | N/A | ✅ Documentado (básico) |

---

## 🔗 Referencias

- [Lenguaje Ubicuo](../lenguaje-ubicuo.md)
- [ADR-0004: Microservice Template](../adr/0004-microservice-template.md)
- [Stack Tecnológico](../stack-tecnologico.md)
- [Guía Técnica](../guia-tecnica.md)

---

## 📝 Nota sobre Sincronización

Este portal centraliza la documentación que vive en cada repositorio de servicio.

- Cada carpeta dentro de `docs/services/<servicio>` proviene del `docs/` del repositorio correspondiente.
- No edites estos archivos aquí; realiza los cambios en el repositorio del servicio y deja que la sincronización los replique.
- Cada servicio puede complementar su `docs/index.md` con enlaces, diagramas y ADRs locales.

> Fuente de datos: listado en `repos.json`. Ajusta ese archivo para agregar o quitar servicios.
