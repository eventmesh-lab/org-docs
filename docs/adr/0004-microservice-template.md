---
Title: Crear y adoptar template para microservicios (Hexagonal + TDD)
Status: Accepted
Date: 2025-11-10
Authors: Equipo de arquitectura
---

# ADR 0004: Crear y adoptar template para microservicios (Hexagonal + TDD)

## Context

Para favorecer la consistencia entre equipos y acelerar el desarrollo de los módulos descritos en `docs/proyecto-base.md`, se ha desarrollado un template de microservicio que implementa las convenciones de Arquitectura Hexagonal y una estrategia de desarrollo guiada por pruebas (TDD).

El template está publicado en: https://github.com/eventmesh-lab/microservice-hexagonal-template

El objetivo del template es proporcionar una base estandarizada —carpeta, convenciones, pruebas y artefactos— que permita a los equipos arrancar servicios nuevos con práctica y coherencia con las decisiones arquitectónicas ya tomadas (ver ADR 0003: stack tecnológico).

## Decision

Se adopta el template ubicado en el repositorio anterior como plantilla oficial para crear microservicios en este proyecto. Todos los nuevos servicios deben iniciar desde este template y seguir sus convenciones (naming, estructura de capas, convenciones de tests, configuración de CI, Dockerfile y observabilidad mínima).

## Estructura del template

La estructura que usa el template es la siguiente (se incluye para referencia dentro de la ADR):

- src/
  - Microservice.Domain/
  - Microservice.Application/
  - Microservice.Infrastructure/
  - Microservice.Api/
- tests/
  - Microservice.Domain.Tests/
  - Microservice.Application.Tests/
  - Microservice.Infrastructure.IntegrationTests/
- .template.config/template.json

Descripción breve de cada carpeta:

- Microservice.Domain: entidades, value objects, interfaces de dominio y reglas de negocio (pure domain). No depende de infra.
- Microservice.Application: casos de uso, comandos/queries (MediatR), DTOs y orquestación entre domain e infra.
- Microservice.Infrastructure: implementaciones concretas (repositorios, adaptadores a RabbitMQ, Hangfire, EF Core / MongoDB drivers, clientes externos).
- Microservice.Api: capa de exposición (controllers, endpoints, configuración de middleware, autenticación con Keycloak, etc.).
- tests: proyectos de pruebas unitarias y de integración organizados para fomentar TDD y validar contratos.
- .template.config/template.json: configuración para dotnet new / plantillas que facilita crear nuevos proyectos con el nombre del servicio.

## Justificación

- Consistencia: asegura que todos los servicios sigan las mismas convenciones (Hexagonal, separación de concerns, tests), lo que facilita mantenimiento y revisión de código.
- Productividad: reduce el trabajo repetitivo de bootstrap y configuración inicial (Dockerfile, CI, wiring de dependencias), permitiendo al equipo enfocarse en la lógica de negocio.
- Calidad: al imponer TDD y una estructura de tests, se eleva la calidad del software y se facilita la integración continua (tests automáticos en pipelines).
- Alineamiento con decisiones previas: el template incorpora y refleja las decisiones del ADR 0003 (uso de .NET 8, MediatR, Hangfire, YARP, Keycloak, Postgres/Mongo, SignalR), facilitando que los servicios se integren con la plataforma central.

## Consecuencias

Pros:

- Plantilla compartida disminuye errores de configuración y acelera onboarding de nuevos desarrolladores.
- Facilita la creación de librerías y convenciones compartidas (por ejemplo: logging, OpenTelemetry, manejo de excepciones).

Contras / Riesgos:

- Puede generar rigidez si el template no se mantiene actualizado con nuevas necesidades o dependencias.
- Requiere disciplina para actualizar el template y versionarlo adecuadamente; de lo contrario, los servicios divergirán.

Mitigaciones:

- Mantener el repo del template con versionado semántico y changelog; publicar notas de migración.
- Definir un proceso de actualización: PRs desde `microservice-hexagonal-template` a los servicios que indiquen cambios rompientes y un plan de migración.
- Añadir CI que verifique que el template compila y que los ejemplos de creación de servicio funcionan.

## Recomendaciones operativas

- Al crear un nuevo servicio, usar `dotnet new` o el flujo del template para inicializar el código y renombrar namespaces conforme al dominio.
- Ejecutar los tests (unit e integración) localmente antes de abrir PRs.
- Revisar periódicamente el template (por ejemplo, cada sprint o antes de cada entrega mayor) para actualizar dependencias y prácticas (por ejemplo, versiones de .NET, dependencias de seguridad).
- Documentar en el README del template las convenciones de nombres, secrets necesarios (Keycloak, DB), y ejemplos de CI/Docker que ya incluye el template.

## Enlaces

- Repositorio del template: https://github.com/eventmesh-lab/microservice-hexagonal-template
- ADR relacionada: `docs/adr/0003-stack-tecnologico.md`
- Requisitos del proyecto: `docs/proyecto-base.md`

---

Fecha para revisión: 2026-01-15 (alineada con revisión del stack y ADR 0003).
