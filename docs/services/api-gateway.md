# API Gateway (svc_yarn_api-gateway)

Este documento resume la información principal del servicio API Gateway `svc_yarn_api-gateway` del proyecto EventMesh.

## Propósito

El API Gateway actúa como punto de entrada y proxy para eventos y rutas relacionadas. Centraliza las peticiones hacia los distintos adaptadores/servicios y ofrece control de rutas, contratos OpenAPI y endpoints mock para desarrollo.

## Arquitectura y Capas

- Implementación basada en arquitectura Hexagonal (Ports & Adapters).
- Capas:
  - Domain: Entidades y puertos.
  - Application: Casos de uso.
  - Infrastructure: Repositorios y adaptadores (ej. repositorio en memoria para pruebas).
  - Api: Controladores REST, Swagger/OpenAPI y Program.cs (punto de entrada).

## Ubicación del código

`Services/eventmesh-frontend/svc_yarn_api-gateway/src/`

## Ejecutar localmente

1. Restaurar dependencias:

```bash
dotnet restore
```

2. Compilar:

```bash
dotnet build
```

3. Ejecutar la API:

```bash
dotnet run --project src/svc_yar_api-gateway.Api
```

La API expone documentación Swagger en `/swagger` cuando está en ejecución.

## Endpoints y controladores principales

- `EventsProxyController`: Proxy para eventos.
- `MockEventosController`: Endpoints de mock para facilitar el desarrollo y pruebas.

Para detalles de la especificación, revisar `src/svc_yar_api-gateway.Api/openapi.yaml`.

## Configuración

- `appsettings.json` y `appsettings.Development.json` en la carpeta `src/svc_yar_api-gateway.Api/`.

## Pruebas

Ejecutar todas las pruebas del repo:

```bash
dotnet test
```

Hay pruebas unitarias y de integración bajo `tests/`.
