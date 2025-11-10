# Venues Service

## 1. Descripción

Microservicio responsable de la **gestión de recintos físicos (venues)**, incluyendo zonas, aforo y disponibilidad. Administra los espacios donde se realizan los eventos físicos.

**Bounded Context:** Gestión de Eventos

**Repository:** `eventmesh-lab/venues-service`

---

## 2. Responsabilidades

- Registrar y administrar recintos físicos
- Gestionar zonas y distribución de asientos
- Controlar disponibilidad y calendario de ocupación
- Proveer información de capacidad y características
- Validar compatibilidad venue-evento

---

## 3. Modelo de Dominio

### 3.1 Agregado: Venue

**Root Aggregate:** `Venue`

#### Entidades

##### Venue
```csharp
public class Venue : AggregateRoot
{
    public Guid Id { get; private set; }
    public string Nombre { get; private set; }
    public Direccion Ubicacion { get; private set; }
    public int CapacidadTotal { get; private set; }
    public TipoVenue Tipo { get; private set; }
    public EstadoVenue Estado { get; private set; }
    public IReadOnlyList<Zona> Zonas { get; private set; }
    public IReadOnlyList<Caracteristica> Caracteristicas { get; private set; }
}
```

##### Zona
```csharp
public class Zona : Entity
{
    public Guid Id { get; private set; }
    public string Nombre { get; private set; }
    public int Capacidad { get; private set; }
    public TipoDistribucion Distribucion { get; private set; } // Numerado, General
    public IReadOnlyList<Asiento> Asientos { get; private set; }
}
```

##### Asiento
```csharp
public class Asiento : Entity
{
    public Guid Id { get; private set; }
    public string Identificador { get; private set; } // "A-12", "B-5"
    public string Fila { get; private set; }
    public int Numero { get; private set; }
    public bool Accesible { get; private set; }
}
```

#### Value Objects

##### Direccion
```csharp
public record Direccion
{
    public string Calle { get; init; }
    public string Ciudad { get; init; }
    public string Estado { get; init; }
    public string CodigoPostal { get; init; }
    public string Pais { get; init; }
    public Coordenadas Geolocalizacion { get; init; }
}
```

##### TipoVenue
```csharp
public enum TipoVenue
{
    Teatro,
    Estadio,
    Arena,
    SalonEventos,
    CentroConvenciones,
    ClubNocturno,
    AireLibre
}
```

##### EstadoVenue
```csharp
public enum EstadoVenue
{
    Activo,
    Inactivo,
    EnMantenimiento
}
```

##### TipoDistribucion
```csharp
public enum TipoDistribucion
{
    Numerado,    // Asientos específicos
    General      // Entrada general sin asiento asignado
}
```

---

## 4. Comandos del Dominio

### RegistrarVenue
**Descripción:** Registra un nuevo recinto en el sistema.

**Input:**
```csharp
public record RegistrarVenueCommand
{
    public string Nombre { get; init; }
    public DireccionDto Ubicacion { get; init; }
    public TipoVenue Tipo { get; init; }
    public List<ZonaDto> Zonas { get; init; }
    public List<string> Caracteristicas { get; init; }
}
```

**Validaciones:**
- El nombre debe ser único
- La capacidad total debe coincidir con la suma de capacidades de zonas
- Cada zona debe tener al menos 1 asiento

**Emite:** `VenueRegistrado`

---

### ActualizarVenue
**Descripción:** Actualiza información del venue.

**Input:**
```csharp
public record ActualizarVenueCommand
{
    public Guid VenueId { get; init; }
    public string Nombre { get; init; }
    public DireccionDto Ubicacion { get; init; }
    public List<string> Caracteristicas { get; init; }
}
```

**Emite:** `VenueActualizado`

---

### AgregarZona
**Descripción:** Añade una nueva zona al venue.

**Input:**
```csharp
public record AgregarZonaCommand
{
    public Guid VenueId { get; init; }
    public string Nombre { get; init; }
    public int Capacidad { get; init; }
    public TipoDistribucion Distribucion { get; init; }
    public List<AsientoDto> Asientos { get; init; }
}
```

**Validaciones:**
- El nombre de zona debe ser único dentro del venue
- Si es distribución numerada, debe incluir todos los asientos

**Emite:** `ZonaAgregada`

---

### DesactivarVenue
**Descripción:** Marca el venue como inactivo.

**Input:**
```csharp
public record DesactivarVenueCommand
{
    public Guid VenueId { get; init; }
    public string Razon { get; init; }
}
```

**Validaciones:**
- No debe tener eventos futuros programados

**Emite:** `VenueDesactivado`

---

## 5. Eventos de Dominio

### VenueRegistrado
```csharp
public record VenueRegistrado : DomainEvent
{
    public Guid VenueId { get; init; }
    public string Nombre { get; init; }
    public int CapacidadTotal { get; init; }
}
```

**Suscriptores:**
- `analytics-service`: Registra nuevo venue en estadísticas

---

### ZonaAgregada
```csharp
public record ZonaAgregada : DomainEvent
{
    public Guid VenueId { get; init; }
    public Guid ZonaId { get; init; }
    public string Nombre { get; init; }
    public int Capacidad { get; init; }
}
```

---

## 6. Consultas (Queries)

### ObtenerVenuePorId
Retorna toda la información del venue incluyendo zonas y asientos.

### ListarVenuesDisponibles
Lista venues activos con filtros por tipo, ubicación y capacidad.

### VerificarDisponibilidad
Valida si un venue está disponible para una fecha específica.

---

## 7. Reglas de Negocio

1. **Capacidad coherente:** La capacidad total del venue debe ser igual a la suma de capacidades de todas sus zonas.

2. **Distribución numerada:** Si una zona es de tipo `Numerado`, debe tener definidos todos los asientos con sus identificadores únicos.

3. **Disponibilidad:** Un venue puede tener múltiples eventos, pero no en horarios solapados.

4. **Estado activo:** Solo venues en estado `Activo` pueden ser asignados a nuevos eventos.

---

## 8. Persistencia

**Base de Datos:** PostgreSQL

### Tabla: venues
```sql
CREATE TABLE venues (
    id UUID PRIMARY KEY,
    nombre VARCHAR(200) NOT NULL UNIQUE,
    direccion_calle VARCHAR(200),
    direccion_ciudad VARCHAR(100),
    direccion_estado VARCHAR(100),
    direccion_codigo_postal VARCHAR(20),
    direccion_pais VARCHAR(100),
    latitud DECIMAL(10, 8),
    longitud DECIMAL(11, 8),
    capacidad_total INT NOT NULL,
    tipo VARCHAR(50) NOT NULL,
    estado VARCHAR(20) NOT NULL,
    fecha_creacion TIMESTAMP NOT NULL
);
```

### Tabla: zonas
```sql
CREATE TABLE zonas (
    id UUID PRIMARY KEY,
    venue_id UUID NOT NULL REFERENCES venues(id),
    nombre VARCHAR(100) NOT NULL,
    capacidad INT NOT NULL,
    distribucion VARCHAR(20) NOT NULL,
    UNIQUE(venue_id, nombre)
);
```

### Tabla: asientos
```sql
CREATE TABLE asientos (
    id UUID PRIMARY KEY,
    zona_id UUID NOT NULL REFERENCES zonas(id),
    identificador VARCHAR(20) NOT NULL,
    fila VARCHAR(10),
    numero INT,
    accesible BOOLEAN DEFAULT FALSE,
    UNIQUE(zona_id, identificador)
);
```

---

## 9. API Endpoints

### POST /api/venues
Registra un nuevo venue.

### GET /api/venues/{id}
Obtiene detalles completos de un venue.

### GET /api/venues
Lista venues con filtros.

**Query Params:**
- `tipo`: Filtra por tipo de venue
- `ciudad`: Filtra por ciudad
- `capacidadMinima`: Filtra por capacidad
- `estado`: Filtra por estado (Activo, Inactivo)

### PUT /api/venues/{id}
Actualiza información del venue.

### POST /api/venues/{id}/zonas
Agrega una nueva zona al venue.

### GET /api/venues/{id}/disponibilidad
Verifica disponibilidad en un rango de fechas.

**Query Params:**
- `fechaInicio`
- `fechaFin`

---

## 10. Tecnologías

- **.NET 8** (Minimal APIs)
- **Entity Framework Core 8** (PostgreSQL)
- **MediatR** (CQRS)
- **FluentValidation**
- **NetTopologySuite** (Geolocalización)
- **Serilog**

---

## 11. Referencias

- [Lenguaje Ubicuo](../lenguaje-ubicuo.md)
- [Events Service](events-service.md)
