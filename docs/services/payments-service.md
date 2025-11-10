# Payments Service

## 1. Descripción

Microservicio responsable del **procesamiento de transacciones, integración con pasarelas de pago y gestión del ciclo de vida de pagos**. Implementa el agregado `Pago` según el lenguaje ubicuo.

**Bounded Context:** Pagos y Facturación

**Repository:** `eventmesh-lab/payments-service`

---

## 2. Responsabilidades

- Procesar pagos de publicación de eventos
- Procesar pagos de compra de tickets
- Integrar con pasarelas de pago externas (Stripe, PayPal simulado)
- Gestionar reintentos automáticos ante fallos
- Confirmar o rechazar transacciones
- Emitir eventos de dominio para coordinación con otros servicios
- Registrar auditoría completa de transacciones

---

## 3. Modelo de Dominio

### 3.1 Agregado: Pago

**Root Aggregate:** `Pago`

#### Entidades

##### Pago
```csharp
public class Pago : AggregateRoot
{
    public Guid Id { get; private set; }
    public Monto Monto { get; private set; }
    public MetodoPago MetodoPago { get; private set; }
    public EstadoPago Estado { get; private set; }
    public Guid UsuarioId { get; private set; }
    public TipoPago Tipo { get; private set; }
    public string ReferenciaExterna { get; private set; } // ID de la pasarela
    public Guid? EventoId { get; private set; }
    public Guid? ReservaId { get; private set; }
    public DateTime FechaCreacion { get; private set; }
    public DateTime? FechaConfirmacion { get; private set; }
    public int IntentosRealizados { get; private set; }
    public string MensajeError { get; private set; }
}
```

#### Value Objects

##### Monto
```csharp
public record Monto
{
    public decimal Valor { get; init; }
    public string Moneda { get; init; } // USD, EUR, MXN
    
    // Validaciones: Valor debe ser positivo
}
```

##### EstadoPago
```csharp
public enum EstadoPago
{
    Pendiente,      // Pago iniciado, esperando procesamiento
    Confirmado,     // Pago exitoso
    Fallido,        // Pago rechazado por la pasarela
    Reembolsado,    // Pago devuelto al cliente
    Procesando      // En proceso de validación
}
```

##### MetodoPago
```csharp
public enum MetodoPago
{
    TarjetaCredito,
    TarjetaDebito,
    PayPal,
    Transferencia,
    Efectivo
}
```

##### TipoPago
```csharp
public enum TipoPago
{
    PagoPublicacion,  // Tarifa para publicar evento
    PagoEntradas      // Compra de tickets
}
```

---

## 4. Comandos del Dominio

### IniciarPago
**Descripción:** Inicia un nuevo proceso de pago.

**Input:**
```csharp
public record IniciarPagoCommand
{
    public decimal Monto { get; init; }
    public string Moneda { get; init; }
    public Guid UsuarioId { get; init; }
    public MetodoPago MetodoPago { get; init; }
    public TipoPago Tipo { get; init; }
    public Guid? EventoId { get; init; }
    public Guid? ReservaId { get; init; }
}
```

**Validaciones:**
- El monto debe ser mayor a 0
- El usuario debe existir
- Si es `PagoPublicacion`, debe existir el evento
- Si es `PagoEntradas`, debe existir la reserva

**Emite:** `PagoIniciado`

**Estado resultante:** `Procesando`

---

### ConfirmarPago
**Descripción:** Marca el pago como confirmado tras respuesta exitosa de la pasarela.

**Input:**
```csharp
public record ConfirmarPagoCommand
{
    public Guid PagoId { get; init; }
    public string ReferenciaExterna { get; init; }
}
```

**Validaciones:**
- El pago debe estar en estado `Procesando`
- La referencia externa debe ser válida

**Emite:** `PagoConfirmado`

**Estado resultante:** `Confirmado`

---

### FallarPago
**Descripción:** Marca el pago como fallido y programa reintentos si aplica.

**Input:**
```csharp
public record FallarPagoCommand
{
    public Guid PagoId { get; init; }
    public string Razon { get; init; }
}
```

**Validaciones:**
- El pago debe estar en estado `Procesando`

**Emite:** `PagoFallido`

**Estado resultante:** `Fallido`

**Lógica de reintentos:**
- Hasta 3 reintentos automáticos con backoff exponencial (1min, 5min, 15min)
- Después del tercer fallo, notifica al usuario

---

### ReembolsarPago
**Descripción:** Procesa el reembolso de un pago confirmado.

**Input:**
```csharp
public record ReembolsarPagoCommand
{
    public Guid PagoId { get; init; }
    public string Motivo { get; init; }
}
```

**Validaciones:**
- El pago debe estar en estado `Confirmado`
- No debe haberse reembolsado previamente

**Emite:** `PagoReembolsado`

**Estado resultante:** `Reembolsado`

---

## 5. Eventos de Dominio

### PagoIniciado
```csharp
public record PagoIniciado : DomainEvent
{
    public Guid PagoId { get; init; }
    public decimal Monto { get; init; }
    public TipoPago Tipo { get; init; }
    public Guid UsuarioId { get; init; }
}
```

---

### PagoConfirmado
```csharp
public record PagoConfirmado : DomainEvent
{
    public Guid PagoId { get; init; }
    public Guid UsuarioId { get; init; }
    public TipoPago Tipo { get; init; }
    public Guid? EventoId { get; init; }
    public Guid? ReservaId { get; init; }
    public decimal Monto { get; init; }
    public DateTime FechaConfirmacion { get; init; }
}
```

**Suscriptores:**
- `events-service`: Publica evento si es `PagoPublicacion`
- `tickets-service`: Confirma tickets si es `PagoEntradas`
- `invoicing-service`: Genera factura
- `notifications-service`: Notifica al usuario

---

### PagoFallido
```csharp
public record PagoFallido : DomainEvent
{
    public Guid PagoId { get; init; }
    public string Razon { get; init; }
    public int IntentosRealizados { get; init; }
}
```

**Suscriptores:**
- `notifications-service`: Notifica al usuario
- `reservations-service`: Puede cancelar reserva si excede reintentos

---

### PagoReembolsado
```csharp
public record PagoReembolsado : DomainEvent
{
    public Guid PagoId { get; init; }
    public Guid UsuarioId { get; init; }
    public decimal Monto { get; init; }
    public string Motivo { get; init; }
}
```

**Suscriptores:**
- `tickets-service`: Cancela tickets
- `notifications-service`: Notifica reembolso exitoso

---

## 6. Reglas de Negocio

1. **Transacción atómica:** Un pago solo puede estar en un estado a la vez.

2. **Idempotencia:** Múltiples llamadas con el mismo `PagoId` no deben crear pagos duplicados.

3. **Reintentos limitados:** Máximo 3 intentos automáticos antes de marcar como fallido definitivo.

4. **Auditoría completa:** Todos los cambios de estado y respuestas de pasarela deben quedar registrados.

5. **Timeout de procesamiento:** Si un pago queda en estado `Procesando` por más de 10 minutos, se marca como fallido.

6. **Reembolso parcial no soportado:** Solo se permiten reembolsos totales en la versión inicial.

---

## 7. Servicios de Dominio

### ServicioDePagos
```csharp
public interface IServicioDePagos
{
    Task<Result<string>> ProcesarPagoConPasarela(Guid pagoId, MetodoPago metodo);
    Task<Result> ReintentarPagoFallido(Guid pagoId);
    Task<Result> ProcesarReembolso(Guid pagoId, string motivo);
}
```

---

## 8. Integración con Pasarelas de Pago

### Adaptador para Stripe (Simulado)
```csharp
public interface IPasarelaPago
{
    Task<TransaccionResultado> ProcesarPago(PagoRequest request);
    Task<TransaccionResultado> ConsultarEstado(string referenciaExterna);
    Task<TransaccionResultado> ProcesarReembolso(string referenciaExterna, decimal monto);
}
```

**Simulación:** En ambiente de desarrollo, se simula respuesta exitosa con probabilidad del 90% y fallo del 10%.

---

## 9. Jobs Programados (Hangfire)

### ReintentarPagosFallidos
- **Frecuencia:** Cada 5 minutos
- **Descripción:** Busca pagos en estado `Fallido` con menos de 3 intentos y los reprocesa

### LimpiarPagosExpirados
- **Frecuencia:** Cada hora
- **Descripción:** Marca como fallidos los pagos en estado `Procesando` por más de 10 minutos

---

## 10. Integraciones

### Comunicación Asíncrona (RabbitMQ)

**Exchange:** `payments.domain.events`

**Publica:**
- `PagoConfirmado`
- `PagoFallido`
- `PagoReembolsado`

**Consume:**
- `PagoPublicacionIniciado` (desde `events-service`)
- `ReservaCreada` (desde `reservations-service`)

---

## 11. Persistencia

**Base de Datos:** PostgreSQL

### Tabla: pagos
```sql
CREATE TABLE pagos (
    id UUID PRIMARY KEY,
    monto DECIMAL(10,2) NOT NULL,
    moneda VARCHAR(3) NOT NULL,
    metodo_pago VARCHAR(50) NOT NULL,
    estado VARCHAR(20) NOT NULL,
    usuario_id UUID NOT NULL,
    tipo VARCHAR(50) NOT NULL,
    referencia_externa VARCHAR(200),
    evento_id UUID,
    reserva_id UUID,
    fecha_creacion TIMESTAMP NOT NULL,
    fecha_confirmacion TIMESTAMP,
    intentos_realizados INT DEFAULT 0,
    mensaje_error TEXT,
    version INT NOT NULL
);

CREATE INDEX idx_pagos_usuario ON pagos(usuario_id);
CREATE INDEX idx_pagos_estado ON pagos(estado);
CREATE INDEX idx_pagos_evento ON pagos(evento_id);
CREATE INDEX idx_pagos_reserva ON pagos(reserva_id);
```

### Tabla: auditoria_pagos
```sql
CREATE TABLE auditoria_pagos (
    id UUID PRIMARY KEY,
    pago_id UUID NOT NULL REFERENCES pagos(id),
    estado_anterior VARCHAR(20),
    estado_nuevo VARCHAR(20) NOT NULL,
    fecha TIMESTAMP NOT NULL,
    detalles TEXT,
    usuario_responsable UUID
);
```

---

## 12. API Endpoints

### POST /api/pagos
Inicia un nuevo pago.

**Request:**
```json
{
  "monto": 50.00,
  "moneda": "USD",
  "usuarioId": "uuid",
  "metodoPago": "TarjetaCredito",
  "tipo": "PagoEntradas",
  "reservaId": "uuid"
}
```

**Response:** `201 Created`
```json
{
  "pagoId": "uuid",
  "estado": "Procesando",
  "fechaCreacion": "2025-11-10T10:30:00Z"
}
```

---

### GET /api/pagos/{id}
Consulta el estado de un pago.

### POST /api/pagos/{id}/confirmar
Endpoint interno para confirmar pago (callback de pasarela).

### POST /api/pagos/{id}/reembolsar
Solicita el reembolso de un pago.

### GET /api/pagos/usuario/{usuarioId}
Lista todos los pagos de un usuario.

---

## 13. Tecnologías

- **.NET 8** (Minimal APIs)
- **Entity Framework Core 8** (PostgreSQL)
- **MediatR** (CQRS)
- **Hangfire** (Jobs de reintento)
- **Polly** (Políticas de resiliencia)
- **RabbitMQ.Client**
- **Serilog**

---

## 14. Observabilidad

### Métricas
- `pagos_iniciados_total`: Contador de pagos iniciados
- `pagos_confirmados_total`: Contador de pagos exitosos
- `pagos_fallidos_total`: Contador de pagos fallidos
- `tiempo_procesamiento_pago_ms`: Latencia de procesamiento

### Logs
- `PagoIniciado`: Nivel INFO
- `PagoConfirmado`: Nivel INFO
- `PagoFallido`: Nivel WARNING
- `ErrorPasarela`: Nivel ERROR

---

## 15. Referencias

- [Lenguaje Ubicuo](../lenguaje-ubicuo.md)
- [Invoicing Service](invoicing-service.md)
- [Events Service](events-service.md)
- [Tickets Service](tickets-service.md)
