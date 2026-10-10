# Decisiones Arquitectónicas

Un ADR (Architecture Decision Record) documenta una decisión importante de arquitectura: el contexto que la motivó, las alternativas evaluadas, la decisión tomada y sus consecuencias.

## Resumen de decisiones

| ID | Decisión arquitectónica | Driver relacionado | Justificación | Resultado |
|---|---|---|---|---|
| **ADR-001** | **Monolito modular** | DA-06, DA-09 | Organizar el backend en módulos independientes dentro de un mismo proceso desplegable, reduciendo la complejidad operacional para un solo desarrollador. | Módulos de **Detección, Sincronización, Anomalías, Autenticación e Infraestructura**. |
| **ADR-002** | **Clean Architecture** | DA-06, DA-09 | Separar las reglas de negocio de los detalles tecnológicos para que el dominio no dependa de frameworks, base de datos ni sensores. | Capas de **Dominio, Aplicación, Adaptadores e Infraestructura**. |
| **ADR-003** | **Store-and-Forward con SQLite** | DA-01, DA-02 | Garantizar que ningún evento se pierda en los tramos sin cobertura celular, que representan más del 50% del corredor. | **Buffer local SQLite** con estados PENDING/SYNCED y reintentos con backoff exponencial. |
| **ADR-004** | **Edge AI con TFLite embebido** | DA-03, DA-04 | Clasificar anomalías en el dispositivo sin depender de internet, en menos de 50 ms y sin acción del conductor. | **Modelo MLP cuantizado** embebido en el APK con inferencia local. |

---

## ADR-001 — Monolito Modular

**Driver relacionado:** DA-06, DA-09

**Contexto y problema**
KillaUru lo desarrolla y opera una sola persona con un plazo de 6 a 8 semanas. Elegir microservicios implicaría gestionar múltiples servicios, bases de datos independientes y una infraestructura compleja que haría inviable el proyecto en ese tiempo, además no es óptimo cuando un proyecto recién inicia.

**Opciones consideradas**

| Opción | Pros | Contras |
|---|---|---|
| Microservicios | Escalabilidad independiente por módulo | Complejidad operacional inviable para un solo desarrollador |
| Monolito tradicional | Simple de desarrollar | Difícil de escalar y mantener conforme crece |
| **Monolito modular** | Simple de desplegar, módulos separados, puede evolucionar hacia servicios independientes | Requiere disciplina para mantener los límites entre módulos |

**Decisión**
Monolito modular en ASP.NET Core (.NET 8). Todo el backend corre en un solo proceso organizado en bounded contexts internos con interfaces claras entre ellos.

**Consecuencias**
- ✅ Un solo `docker compose up -d` levanta todo el sistema.
- ✅ Cada módulo puede extraerse como servicio independiente en versiones futuras sin reescribir la lógica de negocio.
- ⚠️ Requiere disciplina para que los módulos no se acoplen directamente entre sí.

---

## ADR-002 — Clean Architecture como enfoque interno

**Driver relacionado:** DA-06, DA-09

**Contexto y problema**
Con un solo desarrollador, el código debe ser fácil de modificar y probar. Si la lógica de negocio se mezcla con los controladores o la base de datos, cualquier cambio tecnológico futuro rompe todo el sistema.

**Opciones consideradas**

| Opción | Pros | Contras |
|---|---|---|
| MVC tradicional | Simple y conocido | Lógica de negocio acoplada a frameworks y BD |
| **Clean Architecture** | Dominio independiente de tecnología, fácil de probar | Requiere más carpetas y disciplina inicial |
| Hexagonal (Ports & Adapters) | Similar a Clean Architecture | Terminología menos familiar para el contexto académico |

**Decisión**
Clean Architecture con cuatro capas: Dominio, Aplicación, Adaptadores e Infraestructura. Las dependencias siempre apuntan hacia adentro — el dominio no conoce nada del exterior.

**Consecuencias**
- ✅ La lógica de clasificación de anomalías puede probarse sin base de datos ni sensores reales.
- ✅ Cambiar de PostgreSQL a otro motor solo afecta la capa de infraestructura.
- ⚠️ La estructura inicial requiere más archivos que un MVC simple.

---

## ADR-003 — Store-and-Forward con SQLite como buffer offline

**Driver relacionado:** DA-01, DA-02

**Contexto y problema**
Más del 50% del corredor PE-28A no tiene cobertura celular. Si la app depende de conexión permanente para registrar anomalías, el sistema no cumple su propósito principal en la mayor parte de la vía.

**Opciones consideradas**

| Opción | Pros | Contras |
|---|---|---|
| Envío directo al servidor | Simple | No funciona sin internet — inutilizable en la mayoría del corredor |
| Archivo local en disco | Simple de implementar | Sin garantías de integridad ni control de estado por evento |
| **SQLite + Store-and-Forward** | Persistencia garantizada, control PENDING/SYNCED por evento, nativo en Android | Requiere lógica de sincronización con reintentos |

**Decisión**
SQLite como buffer local en el dispositivo. Cada evento se guarda con estado PENDING y solo se marca SYNCED tras recibir confirmación HTTP 200 del servidor. La sincronización usa backoff exponencial: 30s → 2min → 10min → 30min → 2h.

**Consecuencias**
- ✅ La app funciona completamente sin internet durante todo el trayecto.
- ✅ Garantía at-least-once delivery — ningún evento se pierde silenciosamente.
- ⚠️ El servidor debe implementar deduplicación para manejar eventos que lleguen más de una vez.

---

## ADR-004 — Edge AI con TFLite para clasificación local

**Driver relacionado:** DA-03, DA-04

**Contexto y problema**
El conductor no puede tocar el celular mientras maneja. La clasificación debe ser automática, en tiempo real y sin internet (descarta cualquier solución que envíe datos crudos a la nube para clasificar).

**Opciones consideradas**

| Opción | Pros | Contras |
|---|---|---|
| Enviar señal cruda al servidor | Modelo más potente en nube | Requiere internet permanente, latencia inaceptable en ruta |
| Reglas manuales con umbrales fijos | Sin dependencia de ML | Baja precisión, no distingue tipos de anomalía |
| **TFLite embebido en la app** | Inferencia local < 50 ms, sin internet, modelo < 5 MB | Actualizar el modelo requiere nueva versión de la app |

**Decisión**
Modelo MLP cuantizado convertido a TFLite, embebido directamente en el APK. Clasifica ventanas de 500 ms con confidence ≥ 0.75 para registrar el evento.

**Consecuencias**
- ✅ Clasificación en menos de 50 ms sin necesidad de internet.
- ✅ El modelo ya tiene validación de campo con datos del tramo Ayacucho–Vinchos.
- ⚠️ Actualizar el modelo requiere publicar una nueva versión de la app.