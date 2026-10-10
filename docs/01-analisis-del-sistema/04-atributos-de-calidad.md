# Atributos de Calidad

¿Cómo debe comportarse el sistema? Estos atributos no describen funciones, sino la calidad con la que el sistema debe funcionar.

## Rendimiento

| ID | Requisito |
|---|---|
| RNF-01 | El 95% de las consultas al mapa deben responder en menos de 300 ms desde Lima. |
| RNF-02 | El backend debe procesar y guardar un evento entrante en menos de 300 ms. |
| RNF-03 | El sistema debe soportar 300 dispositivos sincronizando al mismo tiempo sin degradarse. |
| RNF-04 | El modelo TFLite debe clasificar una ventana de muestras en menos de 50 ms en un celular Android de gama media. |

## Disponibilidad

| ID | Requisito |
|---|---|
| RNF-05 | El sistema debe estar disponible al menos el 99% del tiempo medido mensualmente (máximo 7 horas de caída al mes). |
| RNF-06 | Ningún evento válido debe perderse en el flujo app → backend → base de datos, verificable por auditoría. |
| RNF-07 | El sistema debe tener backup automático diario de la base de datos con retención de 30 días. |

## Seguridad

| ID | Requisito |
|---|---|
| RNF-08 | El 100% del tráfico debe ir cifrado con TLS 1.3. Calificación objetivo en SSL Labs: A+. |
| RNF-09 | El panel administrativo debe aplicar headers de seguridad HTTP: HSTS, CSP, X-Frame-Options, X-Content-Type-Options. |
| RNF-10 | La identidad del conductor nunca debe poder reconstruirse a partir de los datos almacenados (hash SHA-256 irreversible). |
| RNF-11 | El sistema debe aplicar rate limiting diferenciado: 60 req/hora para público, 1,000 req/hora para conductor registrado. |

## Usabilidad

| ID | Requisito |
|---|---|
| RNF-12 | La app Android debe estar completamente en español. Las acciones principales no deben requerir más de 3 toques. |
| RNF-13 | El panel web debe funcionar correctamente en Chrome, Firefox y Edge (últimas 2 versiones). |
| RNF-14 | Las anomalías deben mostrarse con tiempo relativo legible: "hace 2 horas", "ayer a las 15:30". |

## Eficiencia de recursos

| ID | Requisito |
|---|---|
| RNF-15 | La app debe consumir menos del 8% de batería por hora activa, menos de 1 MB de datos por hora y menos de 100 MB de almacenamiento tras 30 días de uso. |
| RNF-16 | El costo operativo mensual de infraestructura no debe superar el 40% del ingreso recurrente mensual. |

## Mantenibilidad

| ID | Requisito |
|---|---|
| RNF-17 | El sistema debe poder desplegarse completo con un solo comando: `docker compose up -d`. |
| RNF-18 | El sistema debe poder volver a la versión anterior en menos de 5 minutos ante una falla crítica post-despliegue. |
| RNF-19 | Cada módulo del backend debe tener su propio README con propósito, endpoints y ejemplos de uso. |

## Resumen de prioridades

| ID | Atributo | ¿Por qué es importante en ViaLibre? | Prioridad |
|---|---|---|---|
| AC-01 | Disponibilidad offline | Más del 50% del corredor no tiene señal celular. Si el sistema no funciona sin internet, no sirve para la vía. | 🔴 Crítica |
| AC-02 | Confiabilidad del dato | Un bache mal registrado o perdido silenciosamente daña la confianza del sistema. Cada evento debe llegar sí o sí al servidor. | 🔴 Crítica |
| AC-03 | Rendimiento en móvil | El modelo ML debe clasificar en menos de 50 ms sin agotar la batería del conductor. Un sistema lento o que consume batería será desinstalado. | 🔴 Crítica |
| AC-04 | Seguridad y privacidad | La Ley 29733 obliga a proteger la identidad del conductor. Un sistema que filtra datos personales tiene consecuencias legales. | 🔴 Crítica |
| AC-05 | Mantenibilidad | El sistema lo desarrolla y mantiene una sola persona. Si el código es difícil de modificar, el proyecto muere. | 🟠 Alta |
| AC-06 | Escalabilidad | El sistema debe poder crecer: más conductores, más tramos, más módulos. La arquitectura debe permitirlo sin reescribir todo. | 🟠 Alta |
| AC-07 | Observabilidad | Si algo falla en producción, debe detectarse antes de que los usuarios lo reporten. Monitoreo y logs son indispensables. | 🟠 Alta |
| AC-08 | Usabilidad | El conductor no puede distraerse mientras maneja. La app debe ser completamente pasiva y la detección, automática. | 🟡 Media |