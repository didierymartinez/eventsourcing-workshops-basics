# Workshop de Marten con .NET

Bienvenido al workshop práctico para aprender a usar [Marten](https://martendb.io/) con .NET y PostgreSQL.

## 🎯 Objetivo

Aprenderás a modelar eventos, almacenarlos y consultarlos usando Event Sourcing con Marten y PostgreSQL en .NET, siguiendo un enfoque paso a paso y práctico.

> [!TIP]
> **¿Por dónde empiezo?** Este es el **workshop principal** (el viaje con Jhon). Si aún no dominas los fundamentos de C#/diseño (records, interfaces, polimorfismo, patrón comando/repositorio, inyección de dependencias, DDD básico, async), pasa primero por el **Nivel 0 — Nivelación**: [`fundamentals-workshop/`](./fundamentals-workshop/README.md). Si ya los dominas, arranca aquí; cuando un tema asuma un fundamento, lo enlazaremos a su sección de nivelación en vez de repetirlo.

## 🧠 ¿Qué aprenderás?

Este workshop no es una lista de instrucciones, es un viaje donde descubrirás:
- Cómo gestionar la **historia** de tu negocio en lugar de solo el estado actual.
- Cómo transformar hechos pasados en información útil para el presente.
- Cómo asegurar que el rastro de lo que sucede nunca se borre.
- Cómo automatizar y profesionalizar todo este proceso con herramientas de última generación.

## 📚 Secciones del workshop (Hoja de Ruta)

**Fase 1: El Motor Puro en Memoria**
1. [01 - 🧠 El diario de Jhon: Biografía vs Foto](./secciones/01-el-diario-de-jhon.md)
2. [02 - 🚀 Preparando el lienzo](./secciones/02-preparando-el-lienzo.md)
3. [03 - 📝 Vivir el pasado: El motor Apply](./secciones/03-vivir-el-pasado.md)
4. [04 - 🏗️ Refactorizando el motor: El AggregateRoot](./secciones/04-refactorizando-el-motor.md)
5. [05 - 🌊 El flujo de vida: EventStream](./secciones/05-el-flujo-de-vida.md)
6. [06 - 📦 El Almacén en Memoria: El Event Store](./secciones/06-el-almacen-en-memoria.md)
7. [07 - 💡 Decidir el futuro: Emitir eventos](./secciones/07-decidir-el-futuro.md)
8. [08 - 🛠️ El Command Handler](./secciones/08-el-command-handler.md)

**Fase 2: Transición a la Infraestructura .NET**
9. [09 - ⏳ El tiempo de espera: I/O, async/await y el riesgo de olvidar](./secciones/09-el-riesgo-de-olvidar.md)
10. [10 - 🛎️ El Recepcionista (Dependency Injection)](./secciones/10-inyeccion-de-dependencias.md)

**Fase 3: Persistencia Avanzada y Event Sourcing como Profesional**
11. [11 - 🐳 El Baúl Incombustible (Docker, PostgreSQL y el tipo de dato JSONB)](./secciones/11-docker-postgres.md)
12. [12 - 🗄️ El Bibliotecario Experto (Introducción a Marten)](./secciones/12-introduccion-a-marten.md)

**Fase 4: Desacoplamiento (Wolverine)**
13. [13 - 📨 El Correo Interno: Wolverine](./secciones/13-wolverine.md) ✅
14. [14 - 🤝 El Compromiso Inquebrantable: Outbox](./secciones/14-outbox.md) ✅

**Fase 5: Consultas (CQRS)** → cubierto por [§20 - CQRS y Proyecciones](./secciones/20-cqrs-y-proyecciones.md) ✅

**Fase 6: Profundización y Plantilla** → §18-§26 (Decider, Versionado, Testing, Codegen, ACL, Envelope, dos BCs) y [§17 - La Plantilla Cosmos.BuildingBlocks](./secciones/17-plantilla-cosmos.md) ✅

> ⚠️ Este listado numérico es **histórico**. El workshop es **evolutivo**: el **orden de aprendizaje real** y las secciones que vienen están en el [**ROADMAP.md**](./ROADMAP.md) — síguelo a él, no a los números de archivo.

---

## 🧭 Documentos de madurez y dominio
- [**ROADMAP.md**](./ROADMAP.md) — 🗺️ currículo vivo por niveles (básico→avanzado); crece con el tiempo.
- [**MAPA-CONCEPTUAL.md**](./MAPA-CONCEPTUAL.md) — vista de pájaro: cada concepto → su lugar en el código real de Cosmos.
- [**WOLVERINE-RUTA-EXPERTO.md**](./WOLVERINE-RUTA-EXPERTO.md) — ruta por niveles para dominar WolverineFx, anclada a Cosmos.
- [**CONCEPTOS-PROFUNDO.md**](./CONCEPTOS-PROFUNDO.md) — 🧠 auditoría conceptual profunda: cada concepto con su mecanismo bajo el capó, malentendidos y tradeoffs.
- [**REVISION-CRITICA.md**](./REVISION-CRITICA.md) — revisión crítica y replanteamiento (huecos y mejoras pendientes).
- [**INSPIRACION-MARTEN-WOLVERINE.md**](./INSPIRACION-MARTEN-WOLVERINE.md) — 📚 qué enseñan Marten y Wolverine que deberíamos introducir.
- [**INTEGRACION-PROGRESIVA.md**](./INTEGRACION-PROGRESIVA.md) — 🌱 currículo en espiral: dónde se siembra cada concepto.
- [**EVOLUCION-POR-CODIGO.md**](./EVOLUCION-POR-CODIGO.md) — 🧬 método naive→dolor→refactor→patrón y qué semillas volver arcos de código.

---

El workshop avanza por el [ROADMAP](./ROADMAP.md), no por el número de archivo.

