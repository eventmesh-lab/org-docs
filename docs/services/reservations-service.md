# Reservations Service

## 1. Descripción

Microservicio responsable de la **gestión de reservas temporales de tickets**, incluyendo bloqueo de asientos, expiración automática y liberación de inventario. Previene sobreventa mediante reservas con tiempo límite.

**Bounded Context:** Reservas y Ticketing

**Repository:** `eventmesh-lab/reservations-service`

---

## 2. Responsabilidades

- Crear reservas temporales con bloqueo de asientos
- Gestionar expiración automática de reservas no pagadas (10 minutos)
- Confirmar reservas tras pago exitoso
- Liberar inventario de reservas expiradas o canceladas
- Prevenir condiciones de carrera en selección de asientos
- Integrar con Redis para bloqueo distribuido

---

## 3. Modelo de Dominio

### 3.1 Agregado: Reserva

**Root Aggregate:** `Reserva`

#### Entidades

##### Reserva
```csharp
public class Reserva : AggregateRoot
{
    public Guid Id { get; private set; }
    public Guid EventoId { get; private set; }
    public Guid AsistenteId { get; private set; }
    public EstadoReserva Estado { get; private set; }
    public DateTime FechaCreacion { get; private set; }
    public DateTime FechaExpiracion { get; private set; }
    public DateTime? FechaConfirmacion { get; private set; }
    public IReadOnlyList<ItemReserva> Items { get; private set; }
    public decimal MontoTotal { get; private set; }
}
```

##### ItemReserva
```csharp
public class ItemReserva : Entity
{
    public Guid Id { get; private set; }
    public Guid SeccionId { get; private set; }
    public Guid? AsientoId { get; private set; }
    public string TipoTicket { get; private set; }
    public decimal Precio { get; private set; }
}
```

#### Value Objects

##### EstadoReserva
```csharp
public enum EstadoReserva
{
    Pendiente,    // Reserva creada, esperando pago
    Confirmada,   // Pago exitoso, tickets generados
    Expirada,     // Tiempo límite superado sin pago
    Cancelada     // Cancelada por el usuario
}
```

---

## 4. Comandos del Dominio

### CrearReserva
**Descripción:** Crea una reserva temporal con bloqueo de asientos.

**Input:**
```csharp
public record CrearReservaCommand
{
    public Guid EventoId { get; init; }
    public Guid AsistenteId { get; init; }
    public List<ItemReservaDto> Items { get; init; }
}

public record ItemReservaDto
{
    public Guid SeccionId { get; init; }
    public Guid? AsientoId { get; init; }
    public string TipoTicket { get; init; }
    public decimal Precio { get; init; }
}
```

**Validaciones:**
- El evento debe estar en estado `Publicado`
- Los asientos no deben estar ocupados o reservados
- La capacidad de la sección no debe estar agotada

**Lógica de bloqueo:**
1. Bloqueo distribuido en Redis con TTL de 15 segundos
2. Verificación de disponibilidad
3. Creación de reserva con expiración en 10 minutos
4. Liberación del bloqueo

**Emite:** `ReservaCreada`

**Estado resultante:** `Pendiente`

---

### ConfirmarReserva
**Descripción:** Confirma la reserva tras pago exitoso.

**Input:**
```csharp
public record ConfirmarReservaCommand
{
    public Guid ReservaId { get; init; }
    public Guid PagoId { get; init; }
}
```

**Validaciones:**
- La reserva debe estar en estado `Pendiente`
- La reserva no debe estar expirada
- El pago debe estar confirmado

**Emite:** `ReservaConfirmada`

**Estado resultante:** `Confirmada`

---

### CancelarReserva
**Descripción:** Cancela una reserva y libera los asientos.

**Input:**
```csharp
public record CancelarReservaCommand
{
    public Guid ReservaId { get; init; }
    public string Motivo { get; init; }
}
```

**Validaciones:**
- La reserva debe estar en estado `Pendiente`

**Emite:** `ReservaCancelada`

**Estado resultante:** `Cancelada`

---

### ExpirarReserva
**Descripción:** Marca la reserva como expirada (proceso automático).

**Input:**
```csharp
public record ExpirarReservaCommand
{
    public Guid ReservaId { get; init; }
}
```

**Emite:** `ReservaExpirada`

**Estado resultante:** `Expirada`

---

## 5. Eventos de Dominio

### ReservaCreada
```csharp
public record ReservaCreada : DomainEvent
{
    public Guid ReservaId { get; init; }
    public Guid EventoId { get; init; }
    public Guid AsistenteId { get; init; }
    public List<ItemReservaDto> Items { get; init; }
    public decimal MontoTotal { get; init; }
    public DateTime FechaExpiracion { get; init; }
}
```

**Suscriptores:**
- `payments-service`: Inicia proceso de pago
- `notifications-service`: Envía confirmación de reserva

---

### ReservaConfirmada
```csharp
public record ReservaConfirmada : DomainEvent
{
    public Guid ReservaId { get; init; }
    public Guid EventoId { get; init; }
    public Guid AsistenteId { get; init; }
    public List<ItemReservaDto> Items { get; init; }
}
```

**Suscriptores:**
- `tickets-service`: Genera tickets con QR
- `analytics-service`: Registra venta confirmada

---

### ReservaExpirada
```csharp
public record ReservaExpirada : DomainEvent
{
    public Guid ReservaId { get; init; }
    public Guid EventoId { get; init; }
    public List<Guid> AsientosLiberados { get; init; }
}
```

**Suscriptores:**
- `notifications-service`: Notifica al asistente
- `analytics-service`: Registra abandono de carrito

---

### ReservaCancelada
```csharp
public record ReservaCancelada : DomainEvent
{
    public Guid ReservaId { get; init; }
    public string Motivo { get; init; }
}
```

---

## 6. Reglas de Negocio

1. **Tiempo de expiración:** Las reservas expiran automáticamente a los 10 minutos si no se confirma el pago.

2. **Bloqueo optimista:** Se utiliza bloqueo distribuido en Redis para prevenir condiciones de carrera al seleccionar el mismo asiento simultáneamente.

3. **Liberación inmediata:** Al expirar o cancelar, los asientos se liberan inmediatamente para nuevas reservas.

4. **Reserva única activa:** Un asiento solo puede tener una reserva activa (Pendiente o Confirmada) a la vez.

5. **Límite de items:** Máximo 10 tickets por reserva.

6. **Reintento de confirmación:** Si falla la confirmación por problema técnico, se permiten reintentos antes de la expiración.

---

## 7. Jobs Programados (Hangfire)

### ExpirarReservasPendientes
- **Frecuencia:** Cada 1 minuto
- **Descripción:** Busca reservas en estado `Pendiente` con `FechaExpiracion` pasada y las marca como `Expiradas`

```csharp
public class ExpirarReservasJob
{
    public async Task Execute()
    {
        var reservasExpiradas = await _repository
            .ObtenerReservasPendientesExpiradas();
        
        foreach (var reserva in reservasExpiradas)
        {
            await _mediator.Send(new ExpirarReservaCommand 
            { 
                ReservaId = reserva.Id 
            });
        }
    }
}
```

---

## 8. Integración con Redis

### Bloqueo Distribuido

**Clave:** `reserva:asiento:{asientoId}`

**TTL:** 15 segundos

**Librería:** StackExchange.Redis + RedLock.net

```csharp
public async Task<bool> BloquearAsiento(Guid asientoId)
{
    var lockKey = $"reserva:asiento:{asientoId}";
    var lockValue = Guid.NewGuid().ToString();
    
    return await _redis.StringSetAsync(
        lockKey, 
        lockValue, 
        TimeSpan.FromSeconds(15),
        When.NotExists
    );
}
```

---

## 9. Integraciones

### Comunicación Asíncrona (RabbitMQ)

**Exchange:** `reservations.domain.events`

**Publica:**
- `ReservaCreada`
- `ReservaConfirmada`
- `ReservaExpirada`
- `ReservaCancelada`

**Consume:**
- `PagoConfirmado` (desde `payments-service`)

---

## 10. Persistencia

**Base de Datos:** PostgreSQL + Redis (cache)

### Tabla: reservas
```sql
CREATE TABLE reservas (
    id UUID PRIMARY KEY,
    evento_id UUID NOT NULL,
    asistente_id UUID NOT NULL,
    estado VARCHAR(20) NOT NULL,
    fecha_creacion TIMESTAMP NOT NULL,
    fecha_expiracion TIMESTAMP NOT NULL,
    fecha_confirmacion TIMESTAMP,
    monto_total DECIMAL(10,2) NOT NULL,
    version INT NOT NULL
);

CREATE INDEX idx_reservas_estado ON reservas(estado);
CREATE INDEX idx_reservas_expiracion ON reservas(fecha_expiracion) WHERE estado = 'Pendiente';
```

### Tabla: items_reserva
```sql
CREATE TABLE items_reserva (
    id UUID PRIMARY KEY,
    reserva_id UUID NOT NULL REFERENCES reservas(id),
    seccion_id UUID NOT NULL,
    asiento_id UUID,
    tipo_ticket VARCHAR(50) NOT NULL,
    precio DECIMAL(10,2) NOT NULL
);

CREATE INDEX idx_items_asiento ON items_reserva(asiento_id);
```

---

## 11. API Endpoints

### POST /api/reservas
Crea una nueva reserva.

**Request:**
```json
{
  "eventoId": "uuid",
  "asistenteId": "uuid",
  "items": [
    {
      "seccionId": "uuid",
      "asientoId": "uuid",
      "tipoTicket": "VIP",
      "precio": 150.00
    }
  ]
}
```

**Response:** `201 Created`

---

### GET /api/reservas/{id}
Obtiene detalles de una reserva.

### POST /api/reservas/{id}/confirmar
Confirma una reserva tras pago exitoso (endpoint interno).

### POST /api/reservas/{id}/cancelar
Cancela una reserva manualmente.

### GET /api/reservas/asistente/{asistenteId}
Lista reservas de un asistente.

---

## 12. Tecnologías

- **.NET 8** (Minimal APIs)
- **Entity Framework Core 8** (PostgreSQL)
- **StackExchange.Redis** (Bloqueo distribuido)
- **Hangfire** (Jobs de expiración)
- **MediatR** (CQRS)
- **RabbitMQ.Client**
- **Serilog**

---

## 13. Observabilidad

### Métricas
- `reservas_creadas_total`: Contador de reservas
- `reservas_confirmadas_total`: Contador de confirmaciones
- `reservas_expiradas_total`: Contador de expiraciones
- `tiempo_bloqueo_asiento_ms`: Latencia de bloqueo en Redis

---

## 14. Referencias

- [Lenguaje Ubicuo](../lenguaje-ubicuo.md)
- [Tickets Service](tickets-service.md)
- [Payments Service](payments-service.md)
- [Events Service](events-service.md)
