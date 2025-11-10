# Invoicing Service

## 1. Descripción

Microservicio encargado de la **generación, emisión y custodia de facturas** asociadas a eventos y compras de tickets. Implementa el agregado `Factura` y asegura cumplimiento fiscal según configuración regional.

**Bounded Context:** Facturación y Cumplimiento

**Repository:** `eventmesh-lab/invoicing-service`

---

## 2. Responsabilidades

- Generar facturas a partir de pagos confirmados
- Validar datos fiscales del emisor y receptor
- Aplicar reglas tributarias según país y tipo de operación
- Emitir documentos electrónicos (PDF/XML) y almacenarlos
- Integrarse con servicios externos de timbrado/certificación
- Gestionar cancelaciones y notas de crédito

---

## 3. Modelo de Dominio

### 3.1 Agregado: Factura

**Root Aggregate:** `Factura`

#### Entidades

##### Factura
```csharp
public class Factura : AggregateRoot
{
    public Guid Id { get; private set; }
    public string Folio { get; private set; }
    public Guid PagoId { get; private set; }
    public Guid ClienteId { get; private set; }
    public DatosFiscales Emisor { get; private set; }
    public DatosFiscales Receptor { get; private set; }
    public ICollection<LineaFactura> Lineas { get; private set; }
    public Totales Totales { get; private set; }
    public EstadoFactura Estado { get; private set; }
    public DateTime FechaEmision { get; private set; }
    public string UrlDocumento { get; private set; }
}
```

##### LineaFactura
```csharp
public class LineaFactura : Entity
{
    public int Numero { get; private set; }
    public string Concepto { get; private set; }
    public decimal Cantidad { get; private set; }
    public decimal PrecioUnitario { get; private set; }
    public decimal Impuesto { get; private set; }
    public decimal Total { get; private set; }
}
```

#### Value Objects

##### DatosFiscales
```csharp
public record DatosFiscales
{
    public string RazonSocial { get; init; }
    public string IdentificadorFiscal { get; init; }
    public string Direccion { get; init; }
    public string Pais { get; init; }
}
```

##### Totales
```csharp
public record Totales
{
    public decimal Subtotal { get; init; }
    public decimal Impuestos { get; init; }
    public decimal Total { get; init; }
    public string Moneda { get; init; }
}
```

##### EstadoFactura
```csharp
public enum EstadoFactura
{
    Generada,
    Timbrada,
    Cancelada,
    NotaCredito
}
```

---

## 4. Comandos del Dominio

### GenerarFactura
**Descripción:** Genera una factura tras un pago confirmado.

**Input:**
```csharp
public record GenerarFacturaCommand
{
    public Guid PagoId { get; init; }
    public Guid ClienteId { get; init; }
    public IEnumerable<LineaFacturaDto> Conceptos { get; init; }
}
```

**Validaciones:**
- El pago debe estar en estado `Confirmado`
- El cliente debe tener datos fiscales completos
- Cada línea debe tener cantidad y precios positivos

**Emite:** `FacturaGenerada`

---

### TimbrarFactura
**Descripción:** Envía la factura a certificación fiscal.

**Input:**
```csharp
public record TimbrarFacturaCommand
{
    public Guid FacturaId { get; init; }
}
```

**Emite:** `FacturaTimbrada`

---

### CancelarFactura
**Descripción:** Cancela una factura timbrada y genera nota de crédito.

**Input:**
```csharp
public record CancelarFacturaCommand
{
    public Guid FacturaId { get; init; }
    public string Motivo { get; init; }
}
```

**Validaciones:**
- Debe estar en estado `Timbrada`
- Motivo requerido por normativa

**Emite:** `FacturaCancelada`

---

## 5. Eventos de Dominio

### FacturaGenerada
```csharp
public record FacturaGenerada : DomainEvent
{
    public Guid FacturaId { get; init; }
    public Guid PagoId { get; init; }
    public decimal Total { get; init; }
    public string Moneda { get; init; }
}
```

**Suscriptores:**
- `notifications-service`: Envía factura al cliente

---

### FacturaTimbrada
```csharp
public record FacturaTimbrada : DomainEvent
{
    public Guid FacturaId { get; init; }
    public string FolioFiscal { get; init; }
    public DateTime FechaTimbrado { get; init; }
}
```

**Suscriptores:**
- `compliance-service`: Archiva evidencia regulatoria

---

### FacturaCancelada
```csharp
public record FacturaCancelada : DomainEvent
{
    public Guid FacturaId { get; init; }
    public string Motivo { get; init; }
    public DateTime FechaCancelacion { get; init; }
}
```

**Suscriptores:**
- `payments-service`: Marca reembolso o reversión

---

## 6. Reglas de Negocio

1. **Serie y folio únicos:** Control interno por país.
2. **Impuestos configurables:** Soporta IVA, IGV, Sales Tax.
3. **Retención de documentos:** 5 años según normativa base.
4. **Zona horaria:** Fechas en UTC almacenadas, localizadas en presentación.
5. **Integridad con pagos:** Una factura por pago confirmado.

---

## 7. Integraciones

### Comunicación Asíncrona (RabbitMQ)

**Exchange:** `invoicing.domain.events`

**Publica:**
- `FacturaGenerada`
- `FacturaTimbrada`
- `FacturaCancelada`

**Consume:**
- `PagoConfirmado` (desde `payments-service`)
- `PagoReembolsado` (desde `payments-service`)

---

## 8. Persistencia

**Base de Datos:** PostgreSQL

### Tabla: facturas
```sql
CREATE TABLE facturas (
    id UUID PRIMARY KEY,
    folio VARCHAR(50) NOT NULL,
    pago_id UUID NOT NULL,
    cliente_id UUID NOT NULL,
    emisor JSONB NOT NULL,
    receptor JSONB NOT NULL,
    totales JSONB NOT NULL,
    estado VARCHAR(20) NOT NULL,
    fecha_emision TIMESTAMP NOT NULL,
    url_documento TEXT
);
```

### Tabla: facturas_lineas
```sql
CREATE TABLE facturas_lineas (
    factura_id UUID NOT NULL REFERENCES facturas(id),
    numero SMALLINT NOT NULL,
    concepto TEXT NOT NULL,
    cantidad DECIMAL(10,2) NOT NULL,
    precio_unitario DECIMAL(10,2) NOT NULL,
    impuesto DECIMAL(10,2) NOT NULL,
    total DECIMAL(10,2) NOT NULL,
    PRIMARY KEY (factura_id, numero)
);
```

---

## 9. API Endpoints

### POST /api/facturas
Genera una factura manual.

**Request:**
```json
{
  "pagoId": "uuid",
  "clienteId": "uuid",
  "conceptos": [
    {
      "concepto": "Publicación de evento",
      "cantidad": 1,
      "precioUnitario": 49.99,
      "impuesto": 8.50
    }
  ]
}
```

---

### GET /api/facturas/{id}
Obtiene detalles, estado y URLs de descarga.

---

### POST /api/facturas/{id}/cancelar
Cancela una factura timbrada.

---

## 10. Observabilidad

- **Métricas:** `facturas_generadas_total`, `facturas_fallidas_total`
- **Tracing:** Propaga `PagoId` como span attribute
- **Alertas:** Reintentos de timbrado > 3 generan alerta de soporte

---

## 11. Referencias

- [Payments Service](payments-service.md)
- [Notifications Service](notifications-service.md)
- [Reservas Service](reservations-service.md)
- [Events Service](events-service.md)
