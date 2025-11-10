# Streaming Service

## 1. Descripción

Microservicio responsable de la **gestión de transmisiones en vivo y control de acceso a sesiones digitales**. Implementa el agregado `Transmision` según el lenguaje ubicuo, permitiendo que eventos digitales o híbridos sean transmitidos con acceso restringido por ticket.

**Bounded Context:** Streaming y Contenido Digital

**Repository:** `eventmesh-lab/streaming-service`

---

## 2. Responsabilidades

- Iniciar y finalizar transmisiones en vivo
- Generar tokens de acceso únicos por asistente
- Validar acceso a sesiones de streaming mediante tickets confirmados
- Registrar asistencia digital (quién se unió y cuándo)
- Controlar calidad y estado de la transmisión
- Integrar con plataforma de streaming (simulada o real)
- Almacenar metadata de sesiones

---

## 3. Modelo de Dominio

### 3.1 Agregado: Transmision

**Root Aggregate:** `Transmision`

#### Entidades

##### Transmision
```csharp
public class Transmision : AggregateRoot
{
    public Guid Id { get; private set; }
    public Guid EventoId { get; private set; }
    public string Titulo { get; private set; }
    public EstadoTransmision Estado { get; private set; }
    public DateTime? FechaInicio { get; private set; }
    public DateTime? FechaFin { get; private set; }
    public string UrlStreaming { get; private set; }
    public IReadOnlyList<SesionAsistente> Asistentes { get; private set; }
    public ConfiguracionStreaming Configuracion { get; private set; }
}
```

##### SesionAsistente
```csharp
public class SesionAsistente : Entity
{
    public Guid Id { get; private set; }
    public Guid AsistenteId { get; private set; }
    public Guid TicketId { get; private set; }
    public DateTime FechaIngreso { get; private set; }
    public DateTime? FechaSalida { get; private set; }
    public TimeSpan TiempoConectado { get; private set; }
    public string TokenAcceso { get; private set; }
}
```

#### Value Objects

##### EstadoTransmision
```csharp
public enum EstadoTransmision
{
    Programada,     // Transmisión creada, esperando inicio
    EnVivo,         // Actualmente transmitiendo
    Pausada,        // Temporalmente pausada
    Finalizada,     // Transmisión terminada
    Cancelada       // Transmisión cancelada
}
```

##### ConfiguracionStreaming
```csharp
public record ConfiguracionStreaming
{
    public string Resolucion { get; init; }  // 1080p, 720p, 480p
    public int Bitrate { get; init; }
    public bool PermitirChat { get; init; }
    public bool GrabarSesion { get; init; }
    public int MaximoAsistentes { get; init; }
}
```

---

## 4. Comandos del Dominio

### IniciarTransmision
**Descripción:** El organizador activa una sesión de streaming en vivo.

**Input:**
```csharp
public record IniciarTransmisionCommand
{
    public Guid EventoId { get; init; }
    public string Titulo { get; init; }
    public ConfiguracionStreamingDto Configuracion { get; init; }
}
```

**Validaciones:**
- El evento debe existir y estar en estado `EnCurso` o `Publicado`
- El organizador debe ser el propietario del evento
- No debe haber otra transmisión activa para el mismo evento

**Emite:** `TransmisionIniciada`

**Estado resultante:** `EnVivo`

---

### UnirseATransmision
**Descripción:** El asistente accede a una transmisión validando su ticket.

**Input:**
```csharp
public record UnirseATransmisionCommand
{
    public Guid TransmisionId { get; init; }
    public Guid AsistenteId { get; init; }
    public Guid TicketId { get; init; }
}
```

**Validaciones:**
- La transmisión debe estar en estado `EnVivo`
- El ticket debe estar en estado `Confirmado`
- El ticket debe corresponder al evento de la transmisión
- No se debe haber alcanzado el máximo de asistentes simultáneos

**Emite:** `AsistenteUnidoATransmision`

**Retorna:** Token de acceso único con TTL de 4 horas

---

### FinalizarTransmision
**Descripción:** El organizador finaliza la transmisión en vivo.

**Input:**
```csharp
public record FinalizarTransmisionCommand
{
    public Guid TransmisionId { get; init; }
}
```

**Validaciones:**
- La transmisión debe estar en estado `EnVivo` o `Pausada`

**Emite:** `TransmisionFinalizada`

**Estado resultante:** `Finalizada`

---

### RegistrarSalidaAsistente
**Descripción:** Registra cuando un asistente abandona la transmisión.

**Input:**
```csharp
public record RegistrarSalidaAsistenteCommand
{
    public Guid TransmisionId { get; init; }
    public Guid AsistenteId { get; init; }
}
```

**Emite:** `AsistenteSalioDeTransmision`

---

## 5. Eventos de Dominio

### TransmisionIniciada
```csharp
public record TransmisionIniciada : DomainEvent
{
    public Guid TransmisionId { get; init; }
    public Guid EventoId { get; init; }
    public DateTime FechaInicio { get; init; }
    public string UrlStreaming { get; init; }
}
```

**Suscriptores:**
- `notifications-service`: Notifica a asistentes con tickets
- `analytics-service`: Registra inicio de transmisión

---

### AsistenteUnidoATransmision
```csharp
public record AsistenteUnidoATransmision : DomainEvent
{
    public Guid TransmisionId { get; init; }
    public Guid AsistenteId { get; init; }
    public DateTime FechaIngreso { get; init; }
}
```

**Suscriptores:**
- `analytics-service`: Registra asistencia digital en tiempo real

---

### TransmisionFinalizada
```csharp
public record TransmisionFinalizada : DomainEvent
{
    public Guid TransmisionId { get; init; }
    public Guid EventoId { get; init; }
    public DateTime FechaFin { get; init; }
    public int TotalAsistentes { get; init; }
    public TimeSpan Duracion { get; init; }
}
```

**Suscriptores:**
- `analytics-service`: Genera reporte de asistencia
- `content-service`: Procesa grabación si está habilitada

---

## 6. Reglas de Negocio

1. **Acceso restringido por ticket:** Solo asistentes con tickets confirmados pueden unirse a la transmisión.

2. **Token de acceso único:** Cada asistente recibe un token único que no puede ser compartido. El token expira al finalizar la transmisión o después de 4 horas.

3. **Concurrencia limitada:** Se puede configurar un máximo de asistentes simultáneos para controlar el ancho de banda.

4. **Validación temporal:** El acceso solo es válido durante el periodo de transmisión activa.

5. **Registro de asistencia:** Se registra hora de ingreso, salida y tiempo total conectado de cada asistente.

6. **Un stream por evento:** Solo puede haber una transmisión activa por evento a la vez.

---

## 7. Servicios de Dominio

### ServicioDeStreaming
```csharp
public interface IServicioDeStreaming
{
    Task<string> GenerarTokenAcceso(Guid transmisionId, Guid asistenteId);
    Task<bool> ValidarTokenAcceso(string token);
    Task<string> ObtenerUrlStreamingSegura(Guid transmisionId, string token);
}
```

---

## 8. Integración con SignalR

### Hub de Streaming
```csharp
public class StreamingHub : Hub
{
    // Cliente se conecta con token de acceso
    public async Task JoinStream(string token)
    {
        // Validar token
        // Agregar a grupo de transmisión
        await Groups.AddToGroupAsync(Context.ConnectionId, transmisionId);
    }
    
    // Enviar mensaje de chat (si está habilitado)
    public async Task SendChatMessage(string message)
    {
        await Clients.Group(transmisionId).SendAsync("ReceiveMessage", user, message);
    }
    
    // Cliente se desconecta
    public override async Task OnDisconnectedAsync(Exception exception)
    {
        // Registrar salida del asistente
    }
}
```

---

## 9. Integraciones

### Comunicación Asíncrona (RabbitMQ)

**Exchange:** `streaming.domain.events`

**Publica:**
- `TransmisionIniciada`
- `AsistenteUnidoATransmision`
- `TransmisionFinalizada`

**Consume:**
- `EventoIniciado` (desde `events-service`)
- `EventoFinalizado` (desde `events-service`)

---

## 10. Persistencia

**Base de Datos:** MongoDB

### Colección: transmisiones
```json
{
  "_id": "ObjectId",
  "transmisionId": "uuid",
  "eventoId": "uuid",
  "titulo": "string",
  "estado": "EnVivo|Finalizada|...",
  "fechaInicio": "ISODate",
  "fechaFin": "ISODate",
  "urlStreaming": "string",
  "configuracion": {
    "resolucion": "1080p",
    "bitrate": 5000,
    "permitirChat": true,
    "grabarSesion": true,
    "maximoAsistentes": 1000
  },
  "asistentes": [
    {
      "asistenteId": "uuid",
      "ticketId": "uuid",
      "fechaIngreso": "ISODate",
      "fechaSalida": "ISODate",
      "tiempoConectado": "duration",
      "tokenAcceso": "hashed"
    }
  ]
}
```

### Colección: sesiones_activas (Redis o MongoDB TTL)
Para tracking en tiempo real de usuarios conectados:
```json
{
  "_id": "ObjectId",
  "transmisionId": "uuid",
  "asistenteId": "uuid",
  "tokenAcceso": "string",
  "ultimaActividad": "ISODate",
  "expireAt": "ISODate"  // TTL Index
}
```

---

## 11. API Endpoints

### POST /api/streaming/transmisiones
Inicia una nueva transmisión.

**Request:**
```json
{
  "eventoId": "uuid",
  "titulo": "Concierto Live",
  "configuracion": {
    "resolucion": "1080p",
    "bitrate": 5000,
    "permitirChat": true,
    "grabarSesion": true,
    "maximoAsistentes": 1000
  }
}
```

---

### POST /api/streaming/transmisiones/{id}/unirse
Un asistente solicita acceso a la transmisión.

**Request:**
```json
{
  "asistenteId": "uuid",
  "ticketId": "uuid"
}
```

**Response:**
```json
{
  "tokenAcceso": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "urlStreaming": "https://stream.example.com/live/abc123?token=xyz",
  "expiraEn": "2025-11-10T18:00:00Z"
}
```

---

### PUT /api/streaming/transmisiones/{id}/finalizar
Finaliza la transmisión.

---

### GET /api/streaming/transmisiones/{id}
Obtiene detalles de la transmisión.

---

### GET /api/streaming/transmisiones/{id}/asistentes
Lista asistentes conectados (en tiempo real).

---

### SignalR Hub: `/hubs/streaming`
Conexión WebSocket para chat y actualizaciones en tiempo real.

---

## 12. Integración con Firebase Storage

Para almacenar grabaciones de transmisiones:

```csharp
public interface IContentStorageService
{
    Task<string> SubirGrabacion(Guid transmisionId, Stream videoStream);
    Task<string> ObtenerUrlGrabacion(Guid transmisionId);
}
```

---

## 13. Tecnologías

- **.NET 8** (Minimal APIs)
- **MongoDB Driver** (Persistencia)
- **SignalR** (Comunicación en tiempo real)
- **Firebase Admin SDK** (Almacenamiento)
- **JWT** (Tokens de acceso)
- **RabbitMQ.Client**
- **Serilog**
- **OpenTelemetry**

---

## 14. Observabilidad

### Métricas
- `transmisiones_iniciadas_total`: Contador de transmisiones
- `asistentes_conectados_gauge`: Asistentes conectados actualmente
- `tiempo_sesion_minutos`: Histograma de duración de sesiones
- `transmisiones_finalizadas_total`: Contador de finalizaciones

### Logs
- `TransmisionIniciada`: Nivel INFO
- `AsistenteUnido`: Nivel INFO
- `ErrorAcceso`: Nivel WARNING
- `TransmisionFinalizada`: Nivel INFO

---

## 15. Referencias

- [Lenguaje Ubicuo](../lenguaje-ubicuo.md)
- [Events Service](events-service.md)
- [Tickets Service](tickets-service.md)
- [Notifications Service](notifications-service.md)
