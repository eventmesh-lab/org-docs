# Guía técnica: Arquitectura y buenas prácticas (.NET Core 8)

## Arquitectura Hexagonal en .NET Core 8
La arquitectura hexagonal (puertos y adaptadores) coloca el núcleo de la aplicación —la lógica de negocio— en el centro, aislado de las tecnologías externas. Alrededor van “puertos” (interfaces) y “adaptadores” (implementaciones) para I/O: llamadas a base de datos, APIs externas, UI, etc. El núcleo no “sabe” cómo se procesan o reciben los datos; todas las reglas y procesos residen en él, lo que facilita pruebas y mantenimiento.

En .NET Core 8 se suele implementar separando la solución en proyectos/librerías (por ejemplo: Domain, Application, Infrastructure, API). Cada capa se comunica solo con las necesarias; el dominio debe ser POCO (Plain Old CLR Objects) y no depender de Entity Framework ni de capas superiores. Un puerto es una interfaz inyectada en el dominio y el adaptador es su implementación concreta (p. ej. repositorios con EF Core, clientes HTTP, etc.).

Beneficios: desacople, testeo fácil (mocking de adaptadores), mejor mantenibilidad y claridad en dependencias. Plantillas y ejemplos open-source (p. ej. Amitpnk/Hexagonal-architecture-ASP.NET-Core) muestran soluciones prácticas.

## Domain-Driven Design (DDD) y Microservicios
DDD promueve modelar el software según la realidad del negocio y dividir el dominio en bounded contexts, cada uno con su propio modelo y lenguaje ubicuo. En arquitecturas basadas en microservicios, lo habitual es que cada bounded context se implemente como un microservicio independiente.

Conceptos clave dentro de cada contexto:
- Entidades y objetos de valor (Value Objects)
- Agregados con raíz de agregado que controla invariantes
- Casos de uso orquestados desde la capa de aplicación, delegando reglas al dominio

Capas típicas:
- Dominio: corazón del negocio, persistencia-ignorante (POCOs).
- Aplicación: casos de uso, orquestación, sin lógica de negocio pesada.
- Infraestructura: implementaciones concretas (persistencia, colas, integraciones).

Principio: la capa de Dominio NO conoce Infraestructura ni Aplicación.

## Especificación de Requerimientos (ERS / SRS)
La ERS/SRS es el documento formal que describe cómo debe comportarse el sistema. Debe incluir:
- Propósito y alcance
- Requerimientos funcionales y no funcionales
- Interfaces externas (APIs, UI, hardware)
- Criterios de aceptación y escenarios de ejemplo
- Apéndices: glosario, diagramas, casos de uso

Buenas prácticas: usar lenguaje claro y verificable, criterios de aceptación medibles, Specification by Example para escenarios, y plantillas estándar (IEEE 29148/830) para trazabilidad.

## Documentación técnica estructurada
Usar un marco coherente (p. ej. Diátaxis) para organizar: Tutoriales, Guías (how-to), Referencia y Explicaciones. Mantener secciones separadas para API Reference, Quickstart, guías de operación, troubleshooting y changelog. Documentar procesos de despliegue, configuración, runbooks y procedures de recuperación.

## Tecnologías clave (ejemplos y cómo encajan)
- Keycloak (IAM): proveedor de identidad (OpenID Connect/OAuth2). Integración con ASP.NET Core mediante middleware OpenID Connect o librerías cliente.
- RabbitMQ (mensajería): broker AMQP para eventos y colas; integrable con MassTransit o EasyNetQ.
- Hangfire (jobs en background): encolar y ejecutar tareas recurrentes o diferidas con dashboard integrado.
- SignalR: comunicaciones en tiempo real (WebSockets) para notificaciones y dashboards.
- YARP (API Gateway / reverse proxy): enrutamiento y transformación de peticiones dentro de soluciones .NET.
- Firebase Storage: almacenamiento de objetos (imágenes, vídeos) para assets generados por usuarios.
- React (frontend SPA): frontend desacoplado que consume APIs REST/GraphQL del backend .NET.
- Pruebas unitarias: xUnit/NUnit/MSTest con herramientas de cobertura (Coverlet, ReportGenerator). Priorizar tests en rutas críticas; la métrica de cobertura es una guía, no un objetivo absoluto.

## Buenas prácticas y recomendaciones
- Mantener el dominio puro: sin referencias a infra ni frameworks.
- Diseñar puertos finos y explícitos; preferir interfaces bien definidas a acoplamientos amplios.
- Aplicar pruebas de integración en adaptadores (repositorios, clientes HTTP) y pruebas unitarias en casos de uso y dominio.
- Versionar APIs y documentarlas (OpenAPI/Swagger). Mantener backward compatibility en contratos públicos.
- Usar CI/CD para lint, tests y despliegues automatizados; añadir gates para calidad.

## Recursos y lectura adicional
- Arquitectura Hexagonal: Lo esencial — https://www.netmentor.es/entrada/arquitectura-hexagonal
- Designing a DDD-oriented microservice - .NET | Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/ddd-oriented-microservice
- GitHub - Amitpnk/Hexagonal-architecture-ASP.NET-Core — https://github.com/Amitpnk/Hexagonal-architecture-ASP.NET-Core
- Hexagonal Architecture with .NET | Paulovich.NET — https://paulovich.net/hexagonal-architecture-dot-net/
- How to Write a Software Requirements Specification (SRS) Document | Perforce — https://www.perforce.com/blog/alm/how-write-software-requirements-specification-srs-document
- 9 Software Documentation Best Practices + Real Examples | Atlassian — https://www.atlassian.com/blog/loom/software-documentation-best-practices
- Keycloak — https://www.keycloak.org/
- Hangfire — https://www.hangfire.io/
- Real-time ASP.NET with SignalR | .NET — https://dotnet.microsoft.com/en-us/apps/aspnet/signalr
- YARP Getting Started | Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/yarp/getting-started?view=aspnetcore-9.0

---
_Documento técnico para guiar decisiones de arquitectura y elecciones tecnológicas en la plataforma basada en .NET Core 8._
