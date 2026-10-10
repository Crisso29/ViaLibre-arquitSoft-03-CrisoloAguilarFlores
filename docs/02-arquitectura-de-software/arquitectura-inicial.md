# Arquitectura Inicial del Sistema

ViaLibre nace como un **monolito modular**. Todo el backend corre en un solo proceso, pero organizado en bounded contexts internos con responsabilidades separadas. Cada módulo puede extraerse como servicio independiente conforme el sistema crezca, sin reescribir la lógica de negocio.

## Diagrama de arquitectura

```mermaid
flowchart TD

    subgraph ACTORES["👤 ACTORES"]
        C["Conductor"]
        A["Administrador"]
    end

    subgraph PRESENTACION["🖥️ PRESENTACIÓN"]
        APP["App Android\nKotlin + TFLite"]
        WEB["Panel Web\nReact + Leaflet"]
    end

    subgraph NEGOCIO["⚙️ LÓGICA DE NEGOCIO — Monolito Modular"]
        DET["Detección y Clasificación\n(TFLite local)"]
        SYNC["Sincronización\nStore-and-Forward"]
        ING["Ingesta de Eventos\n.NET Channels"]
        GES["Gestión de Anomalías\nPersistente / Episódica"]
        AUTH["Autenticación y Accesos\nJWT + RBAC"]
    end

    subgraph DATOS["🗄️ DATOS"]
        SQL["SQLite\nBuffer local (dispositivo)"]
        PG["PostgreSQL + PostGIS\nAnomalías geolocalizadas"]
        RD["Redis\nCaché del mapa público"]
    end

    subgraph INFRA["☁️ INFRAESTRUCTURA"]
        OC["Oracle Cloud ARM\nDocker Compose"]
        CF["Cloudflare\nCDN + SSL + DNS"]
        UR["UptimeRobot\nMonitoreo 24/7"]
    end

    C --> APP
    A --> WEB

    APP --> DET
    APP --> SYNC
    WEB --> GES
    WEB --> AUTH

    DET --> SQL
    SYNC --> ING
    ING --> GES
    GES --> PG
    GES --> RD
    AUTH --> PG

    NEGOCIO --> OC
    OC --> CF
    CF --> UR
```

## Descripción de capas

**Presentación:** la app Android es el punto de entrada del conductor-detecta anomalías en segundo plano sin que el conductor toque la pantalla. El panel web es la interfaz del administrador para monitorear el sistema, gestionar usuarios y exportar datos.

**Lógica de negocio:** cinco módulos con responsabilidades separadas dentro de un solo proceso. Detección y Clasificación corren localmente en el dispositivo con TFLite. Sincronización implementa el patrón Store-and-Forward con backoff exponencial. Ingesta de Eventos usa .NET Channels para absorber hasta 300 dispositivos sincronizando simultáneamente. Gestión de Anomalías aplica las reglas de validación y distingue anomalías persistentes de episódicas. Autenticación y Accesos controla el ingreso al panel con JWT y tres niveles de rol.

**Datos:** SQLite actúa como buffer local en el dispositivo del conductor durante los tramos sin señal. PostgreSQL con PostGIS almacena todas las anomalías geolocalizadas. Redis gestiona la caché del mapa público para responder en menos de 250 ms.

**Infraestructura:** todo corre sobre Oracle Cloud ARM con Docker Compose, con costo operativo mínimo. Cloudflare gestiona el CDN, SSL y DNS. UptimeRobot monitorea los cuatro subdominios cada 5 minutos.