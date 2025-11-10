# Notifications Service

## 1. Descripción

Microservicio encargado de **orquestar comunicaciones multicanal** (correo, SMS, webhooks internos) en respuesta a eventos del dominio. Implementa el agregado `NotificacionPendiente` y coordina plantillas dinámicas por tipo de evento.

**Bounded Context:** Comunicaciones y Alertas

**Repository:** `eventmesh-lab/notifications-service`

---

## 2. Responsabilidades

- Recibir eventos de dominio y mapearlos a plantillas de notificación
- Gestionar preferencias de usuario (canales habilitados, idioma)
- Programar, reintentar y auditar entregas
- Emitir eventos de confirmación o fallo a servicios consumidores
- Exponer APIs para notificaciones ad-hoc o campañas masivas

---

## 3. Modelo de Dominio

### 3.1 Agregado: NotificacionPendiente

**Root Aggregate:** `NotificacionPendiente`

#### Entidades

##### NotificacionPendiente
```csharp
public class NotificacionPendiente : AggregateRoot
{
    public Guid Id { get; private set; }
    public Guid DestinatarioId { get; private set; }
    public CanalPreferido Canal { get; private set; }
    public string PlantillaId { get; private set; }
    public IDictionary<string, string> Datos { get; private set; }
    public EstadoNotificacion Estado { get; private set; }
    public int Intentos { get; private set; }
    public DateTime FechaCreacion { get; private set; }
    public DateTime? UltimoIntento { get; private set; }
}
```

##### PreferenciasUsuario
```csharp
public class PreferenciasUsuario : Entity
{
    public Guid UsuarioId { get; private set; }
    public bool EmailHabilitado { get; private set; }
    public bool SmsHabilitado { get; private set; }
    public bool PushHabilitado { get; private set; }
    public string Idioma { get; private set; }
}
```

#### Value Objects

##### EstadoNotificacion
```csharp
public enum EstadoNotificacion
{
    Pendiente,
    EnProceso,
    Entregada,
    Fallida,
    Cancelada
}
```

##### Canal
```csharp
public enum Canal
{
    Email,
    Sms,
    Push,
    Webhook
}
```

---

## 4. Comandos del Dominio

### CrearNotificacion
**Descripción:** Registra una notificación a partir de un evento recibido.

**Input:**
```csharp
public record CrearNotificacionCommand
{
    public Guid DestinatarioId { get; init; }
    public string Evento { get; init; }
    public IDictionary<string, string> Datos { get; init; }
}
```

**Validaciones:**
- Debe existir plantilla configurada para el evento
- El destinatario debe tener al menos un canal habilitado

**Emite:** `NotificacionCreada`

---

### MarcarNotificacionEntregada
**Descripción:** Confirma la entrega satisfactoria.

**Input:**
```csharp
public record MarcarNotificacionEntregadaCommand
{
    public Guid NotificacionId { get; init; }
    public string ProveedorMensajeId { get; init; }
}
```

**Emite:** `NotificacionEntregada`

---

### ReintentarNotificacion
**Descripción:** Reprograma una notificación fallida.

**Input:**
```csharp
public record ReintentarNotificacionCommand
{
    public Guid NotificacionId { get; init; }
}
```

**Validaciones:**
- Máximo 5 intentos
- Si falló por preferencia inválida, no reintentar

**Emite:** `NotificacionReintentada`

---

## 5. Eventos de Dominio

### NotificacionCreada
```csharp
public record NotificacionCreada : DomainEvent
{
    public Guid NotificacionId { get; init; }
    public Guid DestinatarioId { get; init; }
    public Canal Canal { get; init; }
}
```

**Suscriptores:**
- `deliveries-worker`: Envía mensaje al proveedor externo

---

### NotificacionEntregada
```csharp
public record NotificacionEntregada : DomainEvent
{
    public Guid NotificacionId { get; init; }
    public Guid DestinatarioId { get; init; }
    public DateTime FechaEntrega { get; init; }
    public string Canal { get; init; }
}
```

**Suscriptores:**
- `analytics-service`: Actualiza métricas de engagement
- `events-service`: Puede avanzar flujo de publicación

---

### NotificacionFallida
```csharp
public record NotificacionFallida : DomainEvent
{
    public Guid NotificacionId { get; init; }
    public string Motivo { get; init; }
    public int Intentos { get; init; }
}
```

**Suscriptores:**
- `support-service`: Abre ticket si supera umbral

---

## 6. Reglas de Negocio

1. **Idempotencia por evento:** Se evita duplicidad usando `Evento+Destinatario` como llave de negocio.
2. **Prioridad de canales:** Orden configurable (ej. email → push → SMS).
3. **Horarios silenciosos:** No enviar SMS entre 23:00 y 07:00 en huso del usuario.
4. **Plantillas versionadas:** Cambios se auditan y requieren aprobación.
5. **Retención:** Datos personales se eliminan tras 30 días.

---

## 7. Integraciones

### Comunicación Asíncrona (RabbitMQ)

**Exchange:** `notifications.domain.events`

**Publica:**
- `NotificacionEntregada`
- `NotificacionFallida`

**Consume:**
- `PagoConfirmado` (desde `payments-service`)
- `HiloPublicado` (desde `forums-service`)
- `TransmisionIniciada` (desde `streaming-service`)
- `TicketConfirmado` (desde `tickets-service`)

---

## 8. Persistencia

**Base de Datos:** PostgreSQL

### Tabla: notificaciones
```sql
CREATE TABLE notificaciones (
    id UUID PRIMARY KEY,
    destinatario_id UUID NOT NULL,
    canal VARCHAR(20) NOT NULL,
    plantilla_id VARCHAR(100) NOT NULL,
    estado VARCHAR(20) NOT NULL,
    intentos INT NOT NULL,
    datos JSONB NOT NULL,
    fecha_creacion TIMESTAMP NOT NULL,
    ultimo_intento TIMESTAMP
);
```

### Tabla: preferencias_usuario
```sql
CREATE TABLE preferencias_usuario (
    usuario_id UUID PRIMARY KEY,
    email_habilitado BOOLEAN NOT NULL,
    sms_habilitado BOOLEAN NOT NULL,
    push_habilitado BOOLEAN NOT NULL,
    idioma VARCHAR(10) NOT NULL,
    canales_bloqueados JSONB
);
```

---

## 9. API Endpoints

### POST /api/notificaciones
Crea una notificación manual.

**Request:**
```json
{
  "destinatarioId": "uuid",
  "evento": "PagoConfirmado",
  "datos": {
    "monto": "50.00",
    "moneda": "USD"
  }
}
```

---

### GET /api/notificaciones/{id}
Obtiene detalle y tracking de una notificación.

---

### PUT /api/preferencias/{usuarioId}
Actualiza preferencias y canales habilitados.

---

## 10. Observabilidad

- **Métricas Prometheus:** `notificaciones_en_cola_total`, `notificaciones_fallidas_total`
- **Tracing:** Propaga contexto W3C desde origen del evento
- **Alertas:** Umbral de fallos consecutivos > 5 genera alerta crítica

---

## 11. Referencias

- [Lenguaje Ubicuo](../lenguaje-ubicuo.md)
- [Payments Service](payments-service.md)
- [Forums Service](forums-service.md)
- [Streaming Service](streaming-service.md)
- [Tickets Service](tickets-service.md)
