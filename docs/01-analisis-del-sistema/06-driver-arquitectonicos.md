# Drivers Arquitectónicos

Un driver arquitectónico es un requisito, atributo o restricción que obliga a tomar una decisión de arquitectura. No todos los requisitos son drivers (solo los que realmente cambian cómo se diseña el sistema).

Piénsalo así: si este driver no existiera, la arquitectura sería diferente.

| ID | Driver | Origen | ¿Por qué cambia la arquitectura? |
|---|---|---|---|
| DA-01 | El sistema debe operar sin internet hasta 5 horas continuas. | R-08, AC-01 | Obliga a diseñar Store-and-Forward con SQLite local. Sin este driver, una arquitectura cliente-servidor normal sería suficiente. |
| DA-02 | Ningún evento puede perderse silenciosamente. | AC-02, RNF-06 | Obliga a implementar at-least-once delivery con reintentos exponenciales y confirmación explícita del servidor. |
| DA-03 | La detección debe ser automática y pasiva, sin acción del conductor. | RF-01, AC-08 | Obliga a usar un modelo ML embebido en la app (TFLite) en lugar de reportes manuales. |
| DA-04 | El modelo ML debe clasificar en menos de 50 ms en móvil de gama media. | RNF-04, AC-03 | Obliga a usar inferencia local (Edge AI) en lugar de enviar la señal cruda al servidor para clasificar en la nube. |
| DA-05 | 300 dispositivos pueden sincronizar al mismo tiempo. | RNF-03 | Obliga a diseñar el backend con procesamiento asíncrono interno (Channels de .NET) para absorber picos de carga. |
| DA-06 | Una sola persona desarrolla y opera el sistema. | R-09 | Obliga a elegir monolito modular en lugar de microservicios. La complejidad operacional debe ser mínima. |
| DA-07 | La identidad del conductor no puede almacenarse ni inferirse. | R-06, AC-04 | Obliga a usar hash SHA-256 irreversible del installationID en lugar de cualquier identificador personal. |
| DA-08 | El costo operativo debe ser mínimo en este ciclo 2026-II. | R-04, RNF-16 | Obliga a elegir infraestructura Free Tier permanente (Oracle Cloud ARM, Cloudflare Free, UptimeRobot Free). |
| DA-09 | El sistema debe escalar a más módulos en el futuro (empresas, autoridades). | AC-06 | Obliga a separar el backend en bounded contexts internos desde el inicio, aunque todo corra en un solo proceso. |