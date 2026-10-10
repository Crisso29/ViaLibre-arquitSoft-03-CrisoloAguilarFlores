# Restricciones

Las restricciones son condiciones que no podemos cambiar. No son decisiones, son límites fijos que la arquitectura debe respetar sí o sí, establece el alcance del sistema.

| ID | Tipo | Restricción |
|---|---|---|
| R-01 | Tecnología | La app móvil debe ser Android nativo (Kotlin). No se permite desarrollo híbrido (Flutter, React Native) porque se necesita acceso directo a los sensores a 50 Hz. |
| R-02 | Tecnología | El backend debe desarrollarse en ASP.NET Core (.NET 8). Es la tecnología principal del autor y del proyecto. |
| R-03 | Tecnología | La base de datos debe ser PostgreSQL con la extensión PostGIS. Es el único motor open source con soporte geoespacial serio para producción. |
| R-04 | Infraestructura | El servidor debe ser Oracle Cloud ARM (Always Free). Restricción de presupuesto: costo operativo mensual cercano a cero. |
| R-05 | Infraestructura | El dominio debe ser vialibre.pe registrado en Punto.pe. El TLD .pe comunica identidad peruana del proyecto. |
| R-06 | Legal | El sistema debe cumplir la Ley N° 29733 (Protección de Datos Personales del Perú). La identidad del conductor nunca puede almacenarse ni inferirse. |
| R-07 | Legal | Los datos agregados son propiedad intelectual de CRISSO DEV S.A.C. No se exponen en internet abierto. |
| R-08 | Conectividad | El sistema debe operar en zonas sin cobertura celular durante 4 a 5 horas continuas. La arquitectura no puede depender de conexión permanente. |
| R-09 | Equipo | El proyecto es desarrollado por una sola persona. La arquitectura debe ser simple de operar y desplegar. |
| R-10 | Tiempo | El sistema debe estar desplegado en producción en un plazo de 6 a 8 semanas para la sustentación del curso IS488. |