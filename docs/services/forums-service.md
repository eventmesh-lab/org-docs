# Forums Service

## 1. Descripción

Microservicio responsable de la **gestión de foros y comunidades por evento**, permitiendo la interacción entre asistentes y organizadores mediante hilos de discusión y comentarios. Implementa el agregado `Foro` según el lenguaje ubicuo.

**Bounded Context:** Comunidad y Moderación

**Repository:** `eventmesh-lab/forums-service`

---

## 2. Responsabilidades

- Crear foros automáticamente al publicar eventos
- Gestionar hilos de discusión y comentarios
- Permitir publicación, edición y eliminación de contenido
- Implementar sistema de reacciones y votos
- Gestionar notificaciones de respuestas
- Proveer datos para moderación
- Auditar todas las publicaciones

---

## 3. Modelo de Dominio

### 3.1 Agregado: Foro

**Root Aggregate:** `Foro`

#### Entidades

##### Foro
```csharp
public class Foro : AggregateRoot
{
    public Guid Id { get; private set; }
    public Guid EventoId { get; private set; }
    public string Titulo { get; private set; }
    public string Descripcion { get; private set; }
    public EstadoForo Estado { get; private set; }
    public DateTime FechaCreacion { get; private set; }
    public IReadOnlyList<HiloDiscusion> Hilos { get; private set; }
    public ConfiguracionForo Configuracion { get; private set; }
}
```

##### HiloDiscusion
```csharp
public class HiloDiscusion : Entity
{
    public Guid Id { get; private set; }
    public Guid AutorId { get; private set; }
    public string Titulo { get; private set; }
    public string Contenido { get; private set; }
    public DateTime FechaPublicacion { get; private set; }
    public DateTime? FechaEdicion { get; private set; }
    public bool Destacado { get; private set; }
    public bool Bloqueado { get; private set; }
    public IReadOnlyList<Comentario> Comentarios { get; private set; }
    public IReadOnlyList<Reaccion> Reacciones { get; private set; }
}
```

##### Comentario
```csharp
public class Comentario : Entity
{
    public Guid Id { get; private set; }
    public Guid AutorId { get; private set; }
    public string Contenido { get; private set; }
    public DateTime FechaPublicacion { get; private set; }
    public DateTime? FechaEdicion { get; private set; }
    public Guid? ComentarioPadreId { get; private set; }
    public bool Eliminado { get; private set; }
    public IReadOnlyList<Reaccion> Reacciones { get; private set; }
}
```

#### Value Objects

##### EstadoForo
```csharp
public enum EstadoForo
{
    Activo,       // Foro abierto para publicaciones
    Cerrado,      // Solo lectura, no se permiten nuevas publicaciones
    Archivado     // Foro archivado tras finalizar el evento
}
```

##### Reaccion
```csharp
public record Reaccion
{
    public Guid UsuarioId { get; init; }
    public TipoReaccion Tipo { get; init; }
    public DateTime Fecha { get; init; }
}

public enum TipoReaccion
{
    MeGusta,
    MeEncanta,
    Util,
    Celebrar
}
```

##### ConfiguracionForo
```csharp
public record ConfiguracionForo
{
    public bool PermitirComentariosAnonimos { get; init; }
    public bool RequiereAprobacion { get; init; }
    public bool NotificarNuevosComentarios { get; init; }
    public int MaximoCaracteresComentario { get; init; }
}
```

---

## 4. Comandos del Dominio

### CrearForo
**Descripción:** Crea un foro para un evento (usualmente automático al publicar).

**Input:**
```csharp
public record CrearForoCommand
{
    public Guid EventoId { get; init; }
    public string Titulo { get; init; }
    public string Descripcion { get; init; }
    public ConfiguracionForoDto Configuracion { get; init; }
}
```

**Validaciones:**
- El evento debe existir y estar publicado
- No debe existir otro foro para el mismo evento

**Emite:** `ForoCreado`

**Estado resultante:** `Activo`

---

### PublicarHilo
**Descripción:** Un asistente crea un nuevo hilo de discusión.

**Input:**
```csharp
public record PublicarHiloCommand
{
    public Guid ForoId { get; init; }
    public Guid AutorId { get; init; }
    public string Titulo { get; init; }
    public string Contenido { get; init; }
}
```

**Validaciones:**
- El foro debe estar en estado `Activo`
- El autor debe tener un ticket confirmado para el evento
- El contenido no debe exceder los límites configurados
- El contenido no debe contener palabras prohibidas (filtro básico)

**Emite:** `HiloPublicado`

---

### PublicarComentario
**Descripción:** Un usuario publica un comentario en un hilo.

**Input:**
```csharp
public record PublicarComentarioCommand
{
    public Guid HiloId { get; init; }
    public Guid AutorId { get; init; }
    public string Contenido { get; init; }
    public Guid? ComentarioPadreId { get; init; }  // Para respuestas anidadas
}
```

**Validaciones:**
- El hilo no debe estar bloqueado
- El autor debe tener acceso al foro
- Máximo 3 niveles de anidación en respuestas

**Emite:** `ComentarioPublicado`

---

### EliminarComentario
**Descripción:** El autor o un moderador elimina un comentario.

**Input:**
```csharp
public record EliminarComentarioCommand
{
    public Guid ComentarioId { get; init; }
    public Guid UsuarioId { get; init; }
    public string Razon { get; init; }
}
```

**Validaciones:**
- El usuario debe ser el autor o tener rol de moderador
- El comentario no debe haber sido eliminado previamente

**Emite:** `ComentarioEliminado`

---

### ReaccionarAHilo
**Descripción:** Un usuario añade una reacción a un hilo.

**Input:**
```csharp
public record ReaccionarAHiloCommand
{
    public Guid HiloId { get; init; }
    public Guid UsuarioId { get; init; }
    public TipoReaccion Tipo { get; init; }
}
```

**Emite:** `ReaccionAgregada`

---

### DestacaHilo
**Descripción:** Un moderador marca un hilo como destacado.

**Input:**
```csharp
public record DestacarHiloCommand
{
    public Guid HiloId { get; init; }
    public Guid ModeradorId { get; init; }
}
```

**Emite:** `HiloDestacado`

---

### CerrarForo
**Descripción:** Cierra el foro (solo lectura) tras finalizar el evento.

**Input:**
```csharp
public record CerrarForoCommand
{
    public Guid ForoId { get; init; }
}
```

**Emite:** `ForoCerrado`

**Estado resultante:** `Cerrado`

---

## 5. Eventos de Dominio

### ForoCreado
```csharp
public record ForoCreado : DomainEvent
{
    public Guid ForoId { get; init; }
    public Guid EventoId { get; init; }
    public DateTime FechaCreacion { get; init; }
}
```

**Suscriptores:**
- `notifications-service`: Notifica al organizador

---

### HiloPublicado
```csharp
public record HiloPublicado : DomainEvent
{
    public Guid HiloId { get; init; }
    public Guid ForoId { get; init; }
    public Guid AutorId { get; init; }
    public string Titulo { get; init; }
}
```

**Suscriptores:**
- `notifications-service`: Notifica a suscriptores del foro
- `moderation-service`: Analiza contenido

---

### ComentarioPublicado
```csharp
public record ComentarioPublicado : DomainEvent
{
    public Guid ComentarioId { get; init; }
    public Guid HiloId { get; init; }
    public Guid AutorId { get; init; }
    public Guid? ComentarioPadreId { get; init; }
}
```

**Suscriptores:**
- `notifications-service`: Notifica al autor del hilo o comentario padre
- `moderation-service`: Analiza contenido

---

### ComentarioEliminado
```csharp
public record ComentarioEliminado : DomainEvent
{
    public Guid ComentarioId { get; init; }
    public Guid UsuarioId { get; init; }
    public string Razon { get; init; }
    public bool EliminadoPorModerador { get; init; }
}
```

**Suscriptores:**
- `moderation-service`: Registra en auditoría
- `notifications-service`: Notifica al autor si fue eliminado por moderador

---

## 6. Reglas de Negocio

1. **Acceso por ticket:** Solo asistentes con tickets confirmados pueden publicar en el foro del evento.

2. **Edición limitada:** Los autores pueden editar sus publicaciones durante las primeras 24 horas. Después, solo moderadores pueden editar.

3. **Eliminación suave:** Los comentarios eliminados se marcan como tal pero no se borran físicamente para auditoría.

4. **Moderación automática:** Filtros básicos de contenido inapropiado antes de publicar.

5. **Notificaciones opcionales:** Los usuarios pueden suscribirse a hilos específicos para recibir notificaciones de nuevos comentarios.

6. **Cierre automático:** Los foros se cierran automáticamente 7 días después de finalizar el evento.

---

## 7. Integraciones

### Comunicación Asíncrona (RabbitMQ)

**Exchange:** `forums.domain.events`

**Publica:**
- `ForoCreado`
- `HiloPublicado`
- `ComentarioPublicado`
- `ComentarioEliminado`

**Consume:**
- `EventoPublicado` (desde `events-service`) - Crea foro automáticamente
- `EventoFinalizado` (desde `events-service`) - Programa cierre del foro

---

## 8. Persistencia

**Base de Datos:** MongoDB

### Colección: foros
```json
{
  "_id": "ObjectId",
  "foroId": "uuid",
  "eventoId": "uuid",
  "titulo": "string",
  "descripcion": "string",
  "estado": "Activo|Cerrado|Archivado",
  "fechaCreacion": "ISODate",
  "configuracion": {
    "permitirComentariosAnonimos": false,
    "requiereAprobacion": false,
    "notificarNuevosComentarios": true,
    "maximoCaracteresComentario": 2000
  },
  "hilos": [
    {
      "hiloId": "uuid",
      "autorId": "uuid",
      "titulo": "string",
      "contenido": "string",
      "fechaPublicacion": "ISODate",
      "fechaEdicion": "ISODate",
      "destacado": false,
      "bloqueado": false,
      "comentarios": [
        {
          "comentarioId": "uuid",
          "autorId": "uuid",
          "contenido": "string",
          "fechaPublicacion": "ISODate",
          "comentarioPadreId": "uuid",
          "eliminado": false,
          "reacciones": []
        }
      ],
      "reacciones": [
        {
          "usuarioId": "uuid",
          "tipo": "MeGusta",
          "fecha": "ISODate"
        }
      ]
    }
  ]
}
```

---

## 9. API Endpoints

### POST /api/foros
Crea un nuevo foro (usualmente automático).

---

### GET /api/foros/evento/{eventoId}
Obtiene el foro de un evento específico.

---

### POST /api/foros/{id}/hilos
Publica un nuevo hilo de discusión.

**Request:**
```json
{
  "titulo": "¿Cómo llegar al venue?",
  "contenido": "Alguien sabe cuál es la mejor ruta..."
}
```

---

### GET /api/foros/{id}/hilos
Lista hilos del foro con paginación.

**Query Params:**
- `page`: Número de página
- `pageSize`: Tamaño de página
- `ordenar`: `recientes|populares|destacados`

---

### POST /api/hilos/{id}/comentarios
Publica un comentario en un hilo.

---

### DELETE /api/comentarios/{id}
Elimina un comentario.

---

### POST /api/hilos/{id}/reacciones
Añade una reacción a un hilo.

**Request:**
```json
{
  "tipo": "MeGusta"
}
```

---

### PUT /api/hilos/{id}/destacar
Marca un hilo como destacado (solo moderadores).

---

## 10. Tecnologías

- **.NET 8** (Minimal APIs)
- **MongoDB Driver** (Persistencia)
- **MediatR** (CQRS)
- **RabbitMQ.Client**
- **Serilog**
- **OpenTelemetry**

---

## 11. Observabilidad

### Métricas
- `foros_creados_total`: Contador de foros
- `hilos_publicados_total`: Contador de hilos
- `comentarios_publicados_total`: Contador de comentarios
- `comentarios_eliminados_total`: Contador de eliminaciones

---

## 12. Referencias

- [Lenguaje Ubicuo](../lenguaje-ubicuo.md)
- [Events Service](events-service.md)
- [Notifications Service](notifications-service.md)
- Moderation Service *(pendiente)*
