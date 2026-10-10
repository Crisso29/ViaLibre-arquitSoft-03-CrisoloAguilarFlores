# Escenarios de atributos de calidad de KillaUru

Este documento traduce los ocho atributos de calidad identificados en `04-atributos-de-calidad.md` en escenarios verificables, siguiendo el formato de seis partes de atributos de calidad (fuente, estímulo, entorno/condición, respuesta, medida de respuesta) utilizado en el método ATAM, con columnas adicionales de verificación y estado para seguimiento del proyecto. Cada escenario queda trazado a sus requisitos no funcionales (RNF), restricciones (R) y drivers arquitectónicos (DA) de origen.

## EQ-01. Disponibilidad offline: operación sin cobertura celular

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Disponibilidad offline |
| **Fuente** | Conductor en tránsito por la Vía Los Libertadores |
| **Estímulo** | El conductor circula por un tramo sin señal celular (más del 50 % del corredor, hasta 4–5 horas continuas). |
| **Condición** | La app KillaUru está instalada y en ejecución, con GPS activo; no hay conexión a datos móviles ni WiFi. |
| **Respuesta esperada** | La app sigue detectando anomalías con el modelo TFLite embebido, guarda los eventos en SQLite local con estado `PENDING`, y el mapa continúa funcionando con los datos descargados previamente. |
| **Medida** | 100 % de los eventos detectados durante el tramo sin señal quedan almacenados localmente sin pérdida; al recuperar señal, la sincronización se inicia automáticamente sin intervención del conductor (RF-04). |
| **Verificación** | Prueba de campo en modo avión en el tramo Ayacucho–Vinchos durante un recorrido completo; comparar el conteo de eventos en SQLite antes y después de reactivar los datos móviles. |
| **Estado** | Definido — fundamentado en R-08, AC-01, DA-01. |

## EQ-02. Confiabilidad del dato: entrega garantizada al servidor

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Confiabilidad del dato |
| **Fuente** | Módulo Synchronizer de la app Android |
| **Estímulo** | La sincronización de un lote de eventos falla a mitad de camino (corte de señal, timeout del servidor, error HTTP). |
| **Condición** | Existen eventos con estado `PENDING` en SQLite local esperando confirmación del backend. |
| **Respuesta esperada** | El sistema reintenta la sincronización con backoff exponencial (30 s → 2 min → 10 min → 30 min → 2 h), sin eliminar los eventos del buffer local hasta recibir confirmación HTTP 200 del servidor. |
| **Medida** | 0 % de eventos válidos perdidos en el flujo app → backend → base de datos; garantía *at-least-once* verificable por auditoría (RNF-06). |
| **Verificación** | Prueba de interrupción de red simulada a mitad de una sincronización; comparar el conteo de eventos enviados por la app contra los registrados en PostgreSQL. |
| **Estado** | Definido — fundamentado en AC-02, RNF-06, DA-02. |

## EQ-03. Rendimiento en móvil: clasificación del modelo ML

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Rendimiento en móvil |
| **Fuente** | Sensores del celular (acelerómetro + giroscopio) |
| **Estímulo** | Se completa una ventana de muestreo de 500 ms a 50 Hz mientras el vehículo circula a más de 5 km/h de forma sostenida. |
| **Condición** | Celular Android de gama media, app en primer o segundo plano, modelo TFLite ya cargado en memoria. |
| **Respuesta esperada** | El modelo TFLite clasifica la ventana (categoría, severidad 1–5, confidence 0.0–1.0) localmente, sin enviar la señal cruda al servidor. |
| **Medida** | Clasificación en menos de 50 ms en el 95 % de las ventanas procesadas, con precisión ≥ 90 % validada en campo en el tramo Ayacucho–Vinchos (RNF-04). |
| **Verificación** | Benchmark de inferencia en al menos tres modelos de celular de gama media; medir latencia p95 y precisión contra el dataset etiquetado del tramo demo. |
| **Estado** | Definido — fundamentado en AC-03, RNF-04, DA-04. |

## EQ-04. Seguridad y privacidad: protección de la identidad del conductor

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Seguridad y privacidad |
| **Fuente** | Usuario sin sesión válida o actor malicioso que intenta acceder a datos protegidos |
| **Estímulo** | Se intenta acceder al panel administrativo sin JWT válido, o se intenta reconstruir la identidad de un conductor a partir de los datos almacenados. |
| **Condición** | El backend expone `api.killauru.pe` y `panel.killauru.pe` bajo HTTPS/TLS 1.3; el `installationID` se almacena como hash SHA-256 irreversible. |
| **Respuesta esperada** | El sistema rechaza el acceso no autorizado sin exponer información sensible, y ningún dato almacenado permite reconstruir la identidad del conductor. |
| **Medida** | 100 % de los intentos no autorizados rechazados en las pruebas definidas; 0 % de reidentificación posible en auditoría de la base de datos; calificación A+ en SSL Labs; cumplimiento de la Ley N.º 29733 (RNF-08, RNF-10, R-06). |
| **Verificación** | Pruebas de acceso sobre el panel admin (sin token, token expirado, fuerza bruta en login); auditoría del esquema de base de datos para confirmar ausencia de PII reversible; escaneo SSL Labs. |
| **Estado** | Definido — fundamentado en AC-04, RNF-08 a RNF-11, R-06. |

## EQ-05. Mantenibilidad: cambios aislados por módulo

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Mantenibilidad |
| **Fuente** | Equipo de desarrollo (una sola persona: Crisólogo Aguilar Flores) |
| **Estímulo** | Se modifica una regla de negocio (por ejemplo, el umbral de confidence) o se necesita sustituir un componente interno del backend. |
| **Condición** | El backend está organizado como monolito modular, con *bounded contexts* internos (Anomalías, Autenticación, Auditoría, etc.). |
| **Respuesta esperada** | El cambio se concentra en el módulo responsable sin afectar funciones no relacionadas, y el sistema completo puede desplegarse con un único comando. |
| **Medida** | Las pruebas del módulo modificado y las pruebas de regresión del resto del sistema se aprueban sin fallos; despliegue reproducible con `docker compose up -d`; rollback a la versión anterior en menos de 5 minutos ante una falla crítica (RNF-17, RNF-18). |
| **Verificación** | Revisión de código para confirmar que el cambio no cruza límites de *bounded context* sin justificación; ejecución de la suite de pruebas y medición del tiempo de rollback en un entorno de staging. |
| **Estado** | Definido — fundamentado en AC-05, RNF-17 a RNF-19, DA-06, DA-09. |

## EQ-06. Escalabilidad: sincronización concurrente de la flota

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Escalabilidad |
| **Fuente** | Flota de dispositivos Android sincronizando simultáneamente |
| **Estímulo** | Hasta 300 dispositivos intentan sincronizar sus eventos pendientes en una ventana de tiempo corta (por ejemplo, al recuperar señal simultáneamente en una zona de cobertura). |
| **Condición** | El backend corre como monolito modular sobre un único servidor Oracle Cloud ARM (Free Tier), con procesamiento asíncrono interno (Channels de .NET). |
| **Respuesta esperada** | El sistema absorbe el pico de carga sin degradarse, procesando y confirmando los lotes entrantes dentro de los límites de tiempo acordados. |
| **Medida** | El sistema soporta 300 dispositivos sincronizando al mismo tiempo sin degradación; el backend procesa y guarda un evento entrante en menos de 100 ms (RNF-02, RNF-03). |
| **Verificación** | Prueba de carga simulando 300 clientes concurrentes enviando lotes de hasta 50 eventos; medir latencia p95, tasa de error y uso de CPU/memoria del servidor. |
| **Estado** | Definido — fundamentado en AC-06, RNF-02, RNF-03, DA-05, DA-09. |

## EQ-07. Observabilidad: detección temprana de fallas

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Observabilidad |
| **Fuente** | Componente de infraestructura (backend, base de datos, proxy) |
| **Estímulo** | Un subdominio deja de responder, el endpoint `/health` retorna un código distinto de 200, o se produce una caída del servicio. |
| **Condición** | UptimeRobot monitorea los subdominios cada 5 minutos; existe backup automático diario de la base de datos. |
| **Respuesta esperada** | El sistema notifica la caída al equipo de desarrollo antes de que los usuarios la reporten, y el estado del servicio queda visible públicamente. |
| **Medida** | Alerta por correo enviada dentro de los 5 minutos posteriores a la caída; disponibilidad mensual registrada en `status.killauru.pe` con historial de 30 días; backup diario con retención de 30 días (RNF-05, RNF-07). |
| **Verificación** | Simular una caída forzada del endpoint `/health` y medir el tiempo hasta la alerta; verificar la existencia y restaurabilidad de un backup reciente. |
| **Estado** | Definido — fundamentado en AC-07, RNF-05, RNF-07. |

## EQ-08. Usabilidad: operación pasiva sin distracción del conductor

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Usabilidad |
| **Fuente** | Conductor en tránsito |
| **Estímulo** | El conductor circula por un tramo de la vía mientras la app está en ejecución. |
| **Condición** | La app está instalada y en ejecución (primer o segundo plano); el conductor no interactúa activamente con la pantalla durante el trayecto. |
| **Respuesta esperada** | La detección de anomalías ocurre de forma completamente automática y pasiva, sin requerir ninguna acción manual del conductor; cuando sí interactúa (por ejemplo, para ver el mapa), las acciones principales son accesibles en pocos toques. |
| **Medida** | 0 interacciones manuales requeridas para la detección durante el trayecto; las acciones principales de la interfaz no requieren más de 3 toques; la app está completamente en español; las anomalías se muestran con tiempo relativo legible ("hace 2 horas") (RNF-12, RNF-14). |
| **Verificación** | Prueba de usuario con un conductor real completando un recorrido sin tocar el celular; revisión de la interfaz contando los toques necesarios para cada acción principal. |
| **Estado** | Definido — fundamentado en AC-08, RNF-12, RNF-14, DA-03. |

## Resumen de escenarios

| Código | Atributo de calidad | Prioridad | Qué se busca comprobar |
|---|---|---|---|
| EQ-01 | Disponibilidad offline | 🔴 Crítica | Que la detección y el mapa sigan funcionando sin cobertura celular. |
| EQ-02 | Confiabilidad del dato | 🔴 Crítica | Que ningún evento válido se pierda entre la app y el servidor. |
| EQ-03 | Rendimiento en móvil | 🔴 Crítica | Que el modelo ML clasifique dentro del tiempo y la precisión establecidos. |
| EQ-04 | Seguridad y privacidad | 🔴 Crítica | Que la identidad del conductor y el panel admin queden protegidos. |
| EQ-05 | Mantenibilidad | 🟠 Alta | Que los cambios se concentren en un módulo sin romper el resto del sistema. |
| EQ-06 | Escalabilidad | 🟠 Alta | Que el sistema soporte el pico de sincronización de hasta 300 dispositivos. |
| EQ-07 | Observabilidad | 🟠 Alta | Que una falla en producción se detecte antes de que la reporten los usuarios. |
| EQ-08 | Usabilidad | 🟡 Media | Que el conductor no necesite interactuar con el celular para que el sistema funcione. |