# Estilo Arquitectónico

## ¿Qué es un estilo arquitectónico?

Un estilo arquitectónico define la forma global en que se organiza y despliega un sistema. No describe qué hace el sistema, sino cómo está estructurado internamente y cómo se relacionan sus partes. Elegir el estilo correcto es una de las decisiones más importantes del arquitecto de software, porque condiciona todo lo que viene después: los módulos, las tecnologías, el despliegue y la capacidad de evolución.

---
## Estilo seleccionado: Monolito Modular con Arquitectura en Capas

KillaUru adopta el **Monolito Modular** como estilo arquitectónico global. Todo el backend corre en un único proceso ASP.NET Core (.NET 8), pero organizado internamente en módulos con responsabilidades claramente delimitadas. Cada módulo tiene sus propias capas internas siguiendo Clean Architecture, lo que permite extraerlo como servicio independiente en versiones futuras sin reescribir su lógica de negocio.

> **Capas = organización lógica interna. Monolito = unidad de despliegue.**

### ¿Por qué Monolito Modular?

| Razón | Explicación |
|---|---|
| **Un solo desarrollador** | No hay equipo que paralelice el trabajo en múltiples servicios. La complejidad operacional debe ser mínima. |
| **Plazo académico** | 6 a 8 semanas para tener el sistema en producción. Un monolito se despliega con un solo comando. |
| **Presupuesto cero** | Oracle Cloud ARM Free Tier tiene recursos limitados. Un solo proceso consume menos memoria que varios servicios corriendo en paralelo. |
| **Evolución ordenada** | Los módulos internos tienen interfaces claras desde el inicio. Cuando el sistema crezca, cada módulo puede convertirse en un servicio independiente sin tocar los demás. |

---

## Diagrama de arquitectura del sistema

```mermaid
flowchart TD

    subgraph ACTORES["👤 ACTORES"]
        CONDUCTOR["Conductor\n(Android)"]
        ADMIN["Administrador\n(Panel web)"]
    end

    subgraph PRESENTACION["🖥️ CAPA DE PRESENTACIÓN"]
        APP["App Android\nKotlin + TFLite"]
        PANEL["Panel Web\nReact + Leaflet"]
    end

    subgraph BACKEND["⚙️ BACKEND — Monolito Modular (.NET 8 / ASP.NET Core)"]

        MIDDLEWARE["Middlewares transversales\nJWT Auth · Rate Limiting · Logger · Error Handling"]

        subgraph MODULO_DETECCION["Módulo: Detección"]
            DET_CTRL["SensorController"]
            DET_UC["ClasificarAnomaliaUseCase"]
            DET_ENT["Anomalia · Severidad · Confidence"]
        end

        subgraph MODULO_SYNC["Módulo: Sincronización"]
            SYNC_CTRL["SyncController"]
            SYNC_UC["SincronizarEventosUseCase"]
            SYNC_ENT["EventoPendiente · EstadoSync"]
        end

        subgraph MODULO_ANOMALIAS["Módulo: Gestión de Anomalías"]
            ANOM_CTRL["AnomaliaController"]
            ANOM_UC["ValidarAnomaliaUseCase\nClasificarPersistenciaUseCase"]
            ANOM_ENT["AnomaliaVial · TipoAnomalia"]
        end

        subgraph MODULO_AUTH["Módulo: Autenticación y Accesos"]
            AUTH_CTRL["AuthController"]
            AUTH_UC["LoginUseCase · GestionUsuariosUseCase"]
            AUTH_ENT["Usuario · Rol · Token"]
        end

        subgraph MODULO_INFRA["Módulo: Infraestructura"]
            CHANNELS[".NET Channels\n(procesamiento asíncrono)"]
            REPOS["Repositorios\n(PostGIS · Redis · SQLite)"]
            HEALTH["HealthController\n/health · /metrics"]
        end

        MIDDLEWARE --> MODULO_DETECCION
        MIDDLEWARE --> MODULO_SYNC
        MIDDLEWARE --> MODULO_ANOMALIAS
        MIDDLEWARE --> MODULO_AUTH

        DET_CTRL --> DET_UC --> DET_ENT
        SYNC_CTRL --> SYNC_UC --> SYNC_ENT
        ANOM_CTRL --> ANOM_UC --> ANOM_ENT
        AUTH_CTRL --> AUTH_UC --> AUTH_ENT

        MODULO_DETECCION --> MODULO_INFRA
        MODULO_SYNC --> CHANNELS
        CHANNELS --> MODULO_ANOMALIAS
        MODULO_ANOMALIAS --> REPOS
        MODULO_AUTH --> REPOS
    end

    subgraph DATOS["🗄️ DATOS"]
        SQLITE["SQLite\nBuffer local en dispositivo"]
        POSTGRES["PostgreSQL + PostGIS\nAnomalías geolocalizadas"]
        REDIS["Redis\nCaché del mapa público"]
    end

    subgraph INFRA["☁️ INFRAESTRUCTURA"]
        DOCKER["Docker Compose\nOracle Cloud ARM"]
        CF["Cloudflare\nCDN · SSL · DNS"]
        UR["UptimeRobot\nMonitoreo 24/7"]
    end

    CONDUCTOR --> APP
    ADMIN --> PANEL

    APP -->|HTTPS / REST| MIDDLEWARE
    PANEL -->|HTTPS / REST| MIDDLEWARE

    APP -->|SQLite local| SQLITE
    SQLITE -->|Sync batch| MODULO_SYNC

    REPOS --> POSTGRES
    REPOS --> REDIS

    BACKEND --> DOCKER
    DOCKER --> CF
    CF --> UR
```

---

## Reglas fundamentales de la arquitectura

Estas reglas no son sugerencias — son los límites que mantienen el sistema ordenado conforme crece:

**1. Invocación descendente por capas**
Cada capa solo invoca a la capa inmediatamente inferior. El controlador llama al caso de uso, el caso de uso llama al repositorio. Nunca al revés.

**2. Aislamiento entre módulos**
Un módulo no puede acceder directamente al repositorio o a las tablas de otro módulo. Si el módulo de Anomalías necesita datos de Autenticación, lo hace a través de la interfaz del servicio correspondiente, nunca tocando su base de datos directamente.

**3. Dominio sin dependencias externas**
Las entidades del dominio (Anomalia, EventoPendiente, Usuario) no conocen nada de ASP.NET, PostgreSQL ni ningún framework. Son clases puras de C# que pueden probarse sin levantar ningún servicio.

**4. Despliegue único**
Todo el backend corre en un único contenedor Docker conectado a una única base de datos PostgreSQL. No hay orquestación de múltiples servicios en v1.0.

**5. Evolución controlada**
Cuando el sistema crezca y un módulo necesite escalar de forma independiente, puede extraerse como servicio sin modificar los demás, porque sus interfaces ya están definidas desde el inicio.

---

## Stack tecnológico

| Componente | Tecnología | Justificación |
|---|---|---|
| App móvil | Kotlin + TFLite | Acceso directo a sensores a 50 Hz. TFLite para inferencia local < 50 ms. |
| Backend | ASP.NET Core .NET 8 | Tecnología principal del autor. Alto rendimiento, soporte nativo para async/await y .NET Channels. |
| Base de datos | PostgreSQL + PostGIS | Único motor open source con soporte geoespacial serio para producción. |
| Caché | Redis | Respuestas del mapa público en menos de 250 ms sin consultar PostgreSQL en cada petición. |
| Panel web | React + Leaflet | React para la interfaz administrativa. Leaflet para renderizar el mapa sobre OpenStreetMap. |
| Infraestructura | Docker Compose + Oracle Cloud ARM | Costo operativo cero. Un solo comando levanta todo el sistema. |
| CDN y SSL | Cloudflare Free | TLS 1.3, calificación A+ en SSL Labs, caché de assets estáticos sin costo. |
| Monitoreo | UptimeRobot Free | Alerta por correo ante caídas, monitoreo cada 5 minutos de los 4 subdominios. |