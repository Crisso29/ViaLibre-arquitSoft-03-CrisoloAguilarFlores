# Patrones de diseño

## Objetivo

Documentar los patrones de diseño aplicados en el backend de KillaUru, indicando dónde se usan, qué problema resuelven y con qué tecnología se implementan, para que cualquier retoma del código (incluso por la misma persona, meses después) siga un criterio uniforme.

## Contexto

- **Estilo:** monolito modular (ASP.NET Core 8 / C#).
- **Enfoque interno:** Clean Architecture (ver `diseño-interno-de-modulos.md`).
- **Base:** diagrama de componentes — `componentes-arquitectonicos.md` (C4 nivel 3).

## Patrones aplicados

| Patrón | Dónde se usa | Problema que resuelve | Se implementa con | Archivos en el código |
|---|---|---|---|---|
| Repository | Todos los módulos → PostgreSQL/PostGIS | El negocio no debe conocer SQL ni PostGIS | Dapper + Npgsql | `IAnomaliaRepository.cs`, `PostgresAnomaliaRepository.cs` (y su equivalente en cada módulo) |
| Decorator | Consulta de Mapa → Caché en memoria | Responder consultas del mapa sin golpear PostgreSQL en cada petición | `IMemoryCache` de .NET | `CachedAnomaliaRepository.cs` |
| Specification | Ingesta de Anomalías → Validación de Calidad | Evaluar RN-11 (confidence, precisión GPS, ventana de tiempo) de forma extensible | Clase C# pura, sin librería externa | `EspecificacionCalidadMinima.cs` |
| Domain Service | Ingesta de Anomalías → Clasificación | Aplicar RN-12 sin forzarla dentro de la entidad `Anomalia`, porque depende de varias anomalías a la vez | Clase estática C# pura | `ClasificadorPersistencia.cs` |
| Factory Method | Entidades de dominio (`Anomalia`, `Usuario`, etc.) | Garantizar que una entidad nunca exista en un estado inválido | Constructor privado + método estático `Crear(...)` | `Anomalia.cs`, `Usuario.cs` |
| Command | Todos los casos de uso | Transportar la intención del cliente como datos inmutables, sin exponer el request HTTP | `record` de C# | `IngestarLoteAnomaliasCommand.cs`, `CrearUsuarioCommand.cs` |

## Cómo funciona cada patrón

| Patrón | En pocas palabras |
|---|---|
| Repository | El negocio dice `guardar(anomalia)`; el repositorio escribe el SQL/PostGIS. |
| Decorator | Busca primero en `IMemoryCache`; si no está, consulta PostgreSQL y lo guarda en caché para la próxima vez. |
| Specification | Encapsula una regla de calidad en un objeto con un solo método `EsSatisfechaPor(...)`; agregar una regla nueva es agregar una clase nueva, no modificar la existente. |
| Domain Service | Las reglas que necesitan datos de varias anomalías (no de una sola) viven en un servicio sin estado propio, no dentro de la entidad. |
| Factory Method | El constructor de la entidad es privado; solo `Crear(...)` puede producirla, y ahí se valida todo de una vez. |
| Command | El controlador arma un objeto inmutable con los datos de la petición y se lo entrega al caso de uso. |

## Beneficio

- Cambiar el motor de persistencia (PostgreSQL) o el mecanismo de caché solo afecta al repositorio, nunca a las reglas de negocio.
- Agregar una nueva regla de calidad (por ejemplo, validar la velocidad del vehículo en el momento del evento) no obliga a tocar el caso de uso ni las reglas ya existentes.
- Al ser un proyecto de una sola persona (DA-06), cada regla vive en un archivo pequeño con una sola responsabilidad — más fácil de retomar que un único archivo con todas las validaciones mezcladas.

## Patrones previstos para siguientes iteraciones

| Patrón | Dónde se aplicaría | Cuándo |
|---|---|---|
| Adapter | Usuarios → SendGrid | Al implementar el correo de verificación de registro (HU-11b, prioridad Could) |
| Observer | Ingesta de Anomalías → Alertas en tiempo real | Si se agrega notificación push desde el servidor, además del geofencing local actual (v1.5+) |
| Strategy | Clasificación de anomalías | Si se agrega un segundo modelo de clasificación (por ejemplo, uno más robusto corriendo en el servidor) junto al TFLite embebido en la app |
| Facade | API comercial autoservicio para Empresas/Autoridades | Al activar los módulos de Empresa suscriptora y Autoridad Vial (v1.5+/v2.0+), como única puerta de entrada para clientes externos que pagan por datos agregados |