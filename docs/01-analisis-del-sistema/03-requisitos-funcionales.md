# Requisitos Funcionales

¿Qué debe hacer el sistema? Cada requisito viene directamente de las historias de usuario (un HU puede tener varios requisitos, pero un requisito no puede pertenecer a dos o más HUs).

## RF-01 a RF-05 — App Android (Detección)

| ID | Requisito funcional |
|---|---|
| RF-01 | El sistema debe capturar lecturas del acelerómetro y giroscopio a 50 Hz cuando la velocidad GPS supera 5 km/h de forma sostenida. |
| RF-02 | El sistema debe clasificar automáticamente cada ventana de 500 ms usando el modelo TFLite embebido, retornando categoría, severidad (1–5) y confidence (0.0–1.0). |
| RF-03 | El sistema debe guardar en SQLite local todo evento con confidence ≥ 0.75, junto con timestamp, coordenadas GPS y hash del dispositivo. |
| RF-04 | El sistema debe detectar cuando hay señal de internet y sincronizar automáticamente los eventos pendientes con el servidor. |
| RF-05 | El sistema debe funcionar completamente sin internet — detección y almacenamiento local continúan sin interrupciones. |

## RF-06 a RF-10 — App Android (Sincronización y Mapa)

| ID | Requisito funcional |
|---|---|
| RF-06 | El sistema debe enviar eventos al servidor en lotes de hasta 50, marcándolos como sincronizados solo después de recibir confirmación del servidor. |
| RF-07 | El sistema debe reintentar la sincronización con backoff exponencial (30s → 2min → 10min → 30min → 2h) si falla la conexión. |
| RF-08 | El sistema debe mostrar al conductor no registrado solo el tramo demo Ayacucho–Vinchos (~65 km). |
| RF-09 | El sistema debe mostrar al conductor registrado el mapa completo del corredor de 333 km con anomalías, timestamp y severidad. |
| RF-10 | El sistema debe emitir alertas anticipadas (sonido + visual) cuando el conductor se aproxima a una anomalía conocida, usando solo GPS sin necesidad de internet. |

## RF-11 a RF-15 — Backend (Ingesta y Validación)

| ID | Requisito funcional |
|---|---|
| RF-11 | El sistema debe recibir lotes de eventos desde la app mediante el endpoint POST /api/v1/anomalies/batch y validar su estructura. |
| RF-12 | El sistema debe rechazar eventos con confidence < 0.75, GPS con precisión > 20 m, o timestamp fuera del rango válido. |
| RF-13 | El sistema debe clasificar cada anomalía como PERSISTENTE (aparece en ≥ 3 días distintos en 15 días) o EPISÓDICA (temporal). |
| RF-14 | El sistema debe exponer el endpoint GET /api/v1/anomalies?bbox= que retorna anomalías en formato GeoJSON. |
| RF-15 | El sistema debe registrar en un log append-only toda acción sensible: alta de usuario, cambio de rol, acceso a datos, exportaciones. |

## RF-16 a RF-20 — Panel Administrativo

| ID | Requisito funcional |
|---|---|
| RF-16 | El sistema debe permitir al administrador iniciar sesión con correo y contraseña, emitiendo un JWT con expiración de 8 horas. |
| RF-17 | El sistema debe mostrar al administrador el mapa completo con todas las anomalías, filtros por tipo, severidad y fecha. |
| RF-18 | El sistema debe mostrar al administrador un dashboard con: eventos por día, usuarios activos, cobertura del corredor y latencia del sistema. |
| RF-19 | El sistema debe permitir al administrador crear, editar el rol y desactivar usuarios. |
| RF-20 | El sistema debe permitir al administrador exportar el dataset de anomalías en formato CSV y GeoJSON. |

## Relación entre historias de usuario y requisitos funcionales

| Historia de usuario | Requisitos funcionales relacionados |
|---|---|
| HU-01 — Detección automática | RF-01, RF-02, RF-03 |
| HU-02 — Tipos de anomalía | RF-02, RF-03 |
| HU-03 — Operación sin señal | RF-05, RF-03 |
| HU-04 — Sin pérdida de eventos | RF-06, RF-07 |
| HU-05 — Validación de calidad | RF-11, RF-12 |
| HU-06 — Persistente vs episódica | RF-13 |
| HU-07 — Mapa tramo demo | RF-08 |
| HU-08 — Mapa completo | RF-09 |
| HU-09 — Alertas anticipadas | RF-10 |
| HU-10 — Historial de recorridos | RF-09 |
| HU-11 — Login administrador | RF-16 |
| HU-12 — Mapa admin | RF-17, RF-20 |
| HU-13 — Dashboard estadísticas | RF-18 |
| HU-14 — Gestión de usuarios | RF-19 |
| HU-15 — Log de auditoría | RF-15 |