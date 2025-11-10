---
Title: Adoptar stack tecnológico principal y estándares
Status: Accepted
Date: 2025-11-10
Authors: Equipo de arquitectura
---

# ADR 0003: Adoptar stack tecnológico principal y estándares

## Context

El proyecto descrito en `docs/proyecto-base.md` requiere una plataforma integral para gestión de eventos, reservas y servicios que soporte:

- Arquitectura basada en DDD y Hexagonal.
- Múltiples módulos/servicios que se comunican asincrónicamente.
- Procesamiento en background, jobs programados y reintentos.
- Gestión centralizada de seguridad, roles y autorización.
- Soporte para almacenamiento de archivos y notificaciones en tiempo real.
- Despliegue mediante contenedores y pipelines de CI/CD.

Para asegurar coherencia entre diseño y herramientas, se propone formalizar el stack recomendado en `docs/stack-tecnologico.md` como la decisión de referencia para el proyecto.

## Decision

Se adopta el siguiente stack tecnológico y estándares (resumen; el documento completo con enlaces está en `docs/stack-tecnologico.md`):

- Backend: .NET 8 con MediatR (patrón mediator, soporte para CQRS y separación de concerns).
- Mensajería: RabbitMQ (exchanges, colas, patrones de routing; soporte de DLQ y retries).
- Jobs / Background: Hangfire (ejecución de trabajos programados y background jobs dentro del ecosistema .NET).
- API Gateway: YARP (reverse proxy y enrutamiento entre microservicios).
- Seguridad: Keycloak (OpenID Connect / OAuth2 para autenticación y autorización centralizada).
- Persistencia: PostgreSQL (principal, datos transaccionales) y MongoDB (documental para logs/auditoría y datos flexibles).
- Frontend: React (aplicación web SPA con builds estáticos y despliegue en contenedor/hosting estático).
- Notificaciones en tiempo real: SignalR / WebSockets.
- Almacenamiento de archivos: Firebase Storage (archivos multimedia y comprobantes).
- Despliegue: Docker para contenerización y CI/CD (ej. GitHub Actions) para pipelines de build / push / deployment.

Esta decisión se formaliza como la fuente de referencia para nuevas incorporaciones tecnológicas y plantillas de servicios.

## Justificación (vinculada a `docs/proyecto-base.md`)

Las siguientes necesidades del proyecto (ver `docs/proyecto-base.md`) motivan la elección del stack:

- Arquitectura Hexagonal y DDD: .NET 8 + MediatR facilita la separación de capas y la implementación de patrones de dominio y CQRS. Esto encaja con el objetivo explícito de evitar soluciones CRUD simples y construir módulos independientes.
- Comunicación asincrónica entre módulos: RabbitMQ permite publicar eventos de negocio (reservas, pagos, confirmaciones) y manejar integraciones con proveedores externos sin acoplamiento fuerte.
- Jobs y expiraciones automáticas: Hangfire ofrece una forma integrada en .NET para ejecutar jobs programados (expiración de reservas, conciliaciones, reportes) sin depender de infra adicional.
- Gateway y seguridad: YARP centraliza el enrutamiento y Keycloak provee gestión de identidad y autorización (roles: Usuario, Organizador, Administrador), lo cual es requerido por las reglas de acceso y validación de tokens indicadas en el enunciado.
- Persistencia híbrida: PostgreSQL cubre datos transaccionales (reservas, pagos, usuarios) mientras MongoDB permite almacenar logs, auditoría y esquemas flexibles (encuestas, historiales), tal como recomienda la sección de auditoría de `proyecto-base.md`.
- Notificaciones en tiempo real: SignalR/WebSockets cumple los requisitos de notificaciones y paneles en tiempo real (disponibilidad, liberación de asientos, recordatorios).
- Despliegue y reproducibilidad: Docker + CI/CD permite cumplir la exigencia de despliegue fuera del entorno de desarrollo y facilita pipelines automatizados para builds, tests y despliegues (requerido en entregas).

## Consecuencias

Pros:

- Coherencia con los requisitos: cada herramienta responde a necesidades explícitas del enunciado (jobs, mensajería, seguridad, notificaciones).
- Ecosistema .NET consolidado: facilita la integración entre componentes, reuse de bibliotecas y consistencia operacional.
- Buenas prácticas para producción: uso de contenedores y pipelines facilita despliegues y escalado.

Contras / Riesgos:

- Complejidad operativa: mantener Keycloak, RabbitMQ, PostgreSQL, MongoDB y el orquestador/CI incrementa la superficie operativa.
- Curva de aprendizaje: el equipo debe dominar DDD, MediatR, YARP, Keycloak y observabilidad (OpenTelemetry).
- Costos de infraestructura: servicios gestionados (Firebase Storage, instancias de bases de datos) pueden incurrir en costos operativos.

Mitigaciones:

- Proveer plantillas y ejemplos (Dockerfile, workflows de CI) en el repositorio para estandarizar despliegues.
- Definir procedimientos operacionales mínimos (runbooks) y medición de observabilidad (logs, métricas, trazas).
- Priorizar despliegue de entorno de staging con las piezas básicas (Keycloak, RabbitMQ, Postgres) antes de la puesta en producción.

## Enlaces

- Documento de decisión (esta ADR): `docs/adr/0003-stack-tecnologico.md`
- Documento de referencia con el detalle del stack y enlaces oficiales: `docs/stack-tecnologico.md`
- Enunciado / requisitos del proyecto: `docs/proyecto-base.md`

---

Fecha de revisión propuesta: 2026-01-15 (revisar posibles cambios en dependencias y versiones antes de escalado a producción).
