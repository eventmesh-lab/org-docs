## ⚙️ Tecnologías y Estándares

A continuación se presenta el stack tecnológico recomendado junto con una breve descripción y enlaces a la documentación oficial y recursos útiles.

| Capa / Componente | Tecnología | Descripción |
|---|---|---|
| Backend | .NET 8, MediatR | Implementación modular orientada a dominios (arquitectura limpia / DDD). |
| Mensajería | RabbitMQ | Procesamiento asíncrono de eventos y colas confiables. |
| Jobs | Hangfire | Tareas programadas y background jobs dentro del mismo ecosistema .NET. |
| Gateway | YARP | Enrutamiento y balanceo de peticiones para microservicios (reverse proxy). |
| Seguridad | Keycloak | Gestión de identidad y autorización (OpenID Connect / OAuth2). |
| Persistencia | PostgreSQL, MongoDB | Bases de datos relacional (Postgres) y documental (MongoDB) según necesidades. |
| Frontend | React | Aplicación web dinámica y componentes reutilizables. |
| Notificaciones | SignalR / WebSockets | Comunicación en tiempo real entre cliente y servidor. |
| Almacenamiento | Firebase Storage | Almacenamiento de archivos multimedia y comprobantes. |
| Despliegue | Docker / CI-CD | Contenerización e integración/despliegue continuo (pipelines). |

---

## Documentación y recursos oficiales

Aquí están las páginas oficiales (o repositorios/documentación principal) para cada tecnología, con una breve nota de uso:

- .NET (Microsoft) — Documentación oficial y guías de migración / novedades para .NET 8:
  - https://learn.microsoft.com/dotnet/
  - Nota: usar SDK .NET 8 y seguir recomendaciones de rendimiento y publicación.

- MediatR — Biblioteca para patrones mediator en .NET (CQRS, separación de concerns):
  - https://github.com/jbogard/MediatR

- RabbitMQ — Documentación oficial del broker de mensajería:
  - https://www.rabbitmq.com/

- Hangfire — Framework para background jobs en .NET:
  - https://www.hangfire.io/

- YARP (Yet Another Reverse Proxy) — Proxy reverso de Microsoft para .NET:
  - https://microsoft.github.io/reverse-proxy/

- Keycloak — Servidor de identidad y gestión de accesos (OpenID/OAuth2):
  - https://www.keycloak.org/

- PostgreSQL — Documentación oficial de la base relacional:
  - https://www.postgresql.org/docs/

- MongoDB — Documentación oficial de la base documental:
  - https://www.mongodb.com/docs/

- React — Documentación y guía oficial (sitio principal de React):
  - https://react.dev/

- SignalR (ASP.NET Core) — Soporte de comunicación en tiempo real:
  - https://learn.microsoft.com/aspnet/core/signalr/

- WebSockets (MDN) — Referencia general sobre la API WebSockets en navegadores:
  - https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API

- Firebase Storage — Guía para almacenamiento de archivos en Firebase:
  - https://firebase.google.com/docs/storage

- Docker — Documentación oficial para contenedores y mejores prácticas:
  - https://docs.docker.com/

- CI/CD (ejemplo: GitHub Actions) — Documentación de pipelines e integración continua:
  - https://docs.github.com/actions

---

## Notas de integración rápida

- Autenticación/Autorización: integrar aplicaciones con Keycloak usando OIDC/OAuth2; para APIs .NET usar middleware de autenticación JWT.
- Mensajería: usar RabbitMQ con exchanges y routing adecuados; considerar patrones de retry y DLQ.
- Jobs: emplear Hangfire para procesos que deban ejecutarse dentro del dominio .NET; para cargas masivas considerar colas externas.
- Gateway: usar YARP para configuración dinámica de rutas y balanceo entre servicios.
- Persistencia: preferir PostgreSQL para datos transaccionales y MongoDB para datos flexibles/documentales.
- Observabilidad: instrumentar con logs estructurados, métricas (Prometheus) y trazas distribuidas (OpenTelemetry).

---



## Tutoriales y guías recomendadas

Enlaces prácticos para aprender e implementar las piezas clave del stack:

- Docker + .NET (Best practices): https://docs.docker.com/samples/dotnet/
- Docker + React (multi-stage build): https://nodejs.org/en/docs/guides/nodejs-docker-webapp/
- GitHub Actions: guías oficiales y ejemplos: https://docs.github.com/actions
- CI/CD para .NET con GitHub Actions: https://learn.microsoft.com/dotnet/architecture/modernize-with-azure-containers/ci-cd-github-actions
- YARP examples & getting started: https://microsoft.github.io/reverse-proxy/articles/getting-started.html
- Keycloak quickstarts: https://www.keycloak.org/getting-started
- RabbitMQ tutorials: https://www.rabbitmq.com/getstarted.html
- Hangfire quickstart (.NET): https://docs.hangfire.io/en/latest/getting-started.html
- Firebase Storage guides: https://firebase.google.com/docs/storage
- Observabilidad con OpenTelemetry (.NET): https://opentelemetry.io/docs/instrumentation/net/

---