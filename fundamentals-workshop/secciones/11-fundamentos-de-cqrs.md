# 11 - Divorciando Escritura y Lectura: Fundamentos de CQRS
> 🎯 **Hacia dónde va:** CQRS separa el universo de escritura (comandos, agregados, EventStore) del de lectura (proyecciones); es el marco en el que Event Sourcing graba eventos y luego los proyecta hacia modelos de lectura por consistencia eventual.
> 📦 **Ejemplo de esta sección:** Factura (reportes de finanzas).

Cualquier curso de programación clásica te enseña a hacer un sistema monolítico tradicional (El CRUD). Usas una tabla en la base de datos (Ej. `Facturas`), y la modelas con una clase `Factura` en C# para que sirva absolutamente para TODO.

**El Problema Mundial del CRUD**

1.  **La pantalla del usuario cambia:** El jefe de finanzas pide una tabla (Grilla) en el Frontend que muestre los totales de las facturas del mes, cruzadas con los nombres de los clientes, y la lista de todos sus pagos agrupados por banco.
2.  **Tú sufres:** Vas a tu base de datos y escribes consultas SQL larguísimas (con múltiples `JOIN`), intentando que encajen en tu clase `Factura`. Creas DTOs extraños porque no caben.
3.  **El rendimiento muere:** Cada vez que el gerente entra a esa página en la web, el servidor hace cálculos matemáticos intensos y cruce de datos masivos. La página carga lento.
4.  **Agregas Índices:** Agregas índices a las tablas SQL para que "lean más rápido", pero ahora, cada vez que alguien intenta crear (escribir) una simple nueva Factura, toma el triple de tiempo porque el disco duro tiene que actualizar todos esos gigantescos índices.

**El CRUD te encierra mentalmente en que "Los datos se leen usando la misma forma y la misma base de datos en la que se escriben".**

## ⚔️ La Separación: CQRS

CQRS significa **Segregación de Responsabilidades de Comandos y Consultas** (*Command Query Responsibility Segregation*). Sugiere lo impensable: partir tu arquitectura en dos universos.

### Lado 1: El Universo de Escritura (Commands)
El único objetivo del lado derecho del sistema es procesar Comandos, hacer cumplir reglas de negocio rígidas, mantener la consistencia transaccional y **Escribir/Guardar** que algo sucedió. 

En este lado:
- Se usan **Agregados** (Como `CarritoDeCompras` de la sección 08).
- Se usan **Comandos** y CommandHanders.
- Su modelo de estructura está totalmente normalizado para proteger reglas (No se preocupa si "es fácil buscar en él").
- Aquí ocurre toda nuestra teoría de **Event Sourcing** (La escritura graba Eventos Históricos en un EventStore).

### Lado 2: El Universo de Lectura (Queries)
El único objetivo del lado izquierdo del sistema es **Mostrar datos en una pantalla en 10 milisegundos**.

En este lado:
- NO existen objetos de la vida real como "Agregados". Existen "Proyecciones" o "Modelos de Pantalla" (Ej. `FilaReporteFinanzasGerencia`).
- No hay CommandHandlers. Hay **Queries** (Ej. `ObtenerReporteGerencia`).
- **Los datos ya están pre-calculados y aplanados (Denormalizados).** Si la pantalla pide cruzar clientes con pagos, la base de datos no calcula ningún JOIN. Carga una tabla chata tipo Excel especialmente construida para ESA vista y la escupe íntegra de golpe hacia el usuario. Consultas simples y ultrarrápidas `SELECT * FROM VistaFinanzasPagos`.

## 🔄 El Puente: "Consistencia Final"

Ahí es donde el novato grita: *"Pero espera, ¡si guardo el evento en la Base de Datos transaccional (El EventStore)... cómo es que llega al Lado de lectura de la tabla chata tipo Excel para los reportes sin escribirlo dos veces en el código!"*.

En Event Sourcing, los Eventos son la sangre.
Cuando Jhon crea la factura, el lado de Escritura graba el evento `FacturaCreada ($500)` en el Store principal e inmediatamente **cierra y devuelve el `HTTP 200 OK` al usuario.** "Transacción terminada con cero fricción".

Milésimas de segundo DESPUÉS, en el fondo, corren procesos subscritos "escuchando" (`EventHandlers` o `Projections`). Ellos capturan el evento recién publicado e insertan asincrónicamente el registro en una Base de Datos paralela optimizada solo para lectura (Ej: una tabla desnormalizada en SQL Server, en Elasticsearch o MongoDb). 

*"Consistencia Final"* significa que, por una fracción minúscula de tiempo, la pantalla de "Listar reportes" quizás aún no muestre la venta que acabas de hacer (está desactualizada por milisegundos). Pero está garantizado que *al final del segundo* lo estará. 

Este divorcio total es lo que permite que gigantes como Amazon o Netflix no se caigan en un Black Friday. La cola de escritura absorbe los millones de compras brutamente, y las bases de datos de lectura se van actualizando a su ritmo para la gente navegando.

> [!WARNING]
> **El "cuándo NO" (igual de importante que el cómo).** CQRS **no es gratis**: añade read models, sincronización y **consistencia eventual** (el usuario puede no ver su dato al instante). La **mayoría de los CRUDs simples NO lo necesitan** y se complicarían sin beneficio. Justifícalo cuando: las lecturas y escrituras tienen formas/escala muy distintas, necesitas **varios** modelos de lectura, o el reporting pesa sobre la base transaccional. Si tu duda es *"¿lo aplico?"*, probablemente todavía no. Además, **CQRS y Event Sourcing son ortogonales**: puedes hacer CQRS sin ES (y viceversa); no asumas que uno obliga al otro.

---
[⬅️ Volver a la sección anterior](./10-async-await-y-concurrencia.md) | [➡️ Siguiente sección: El Patrón Outbox](./12-el-patron-outbox.md)
