# 🌱 Integración Progresiva (Currículo en Espiral)

> Los conceptos avanzados **no van al final**: se **siembran** desde las primeras secciones (mención ligera, con vocabulario correcto), **crecen** en las intermedias, y se **dominan** en las avanzadas. Así, cuando el alumno llega al patrón completo, ya lo ha visto 3 veces en contexto. Este documento mapea dónde se toca cada concepto.

**Convención de profundidad:** 🌱 Semilla (se nombra y se siembra la intuición) · 🌿 Crece (se usa de forma básica) · 🌳 Domina (sección dedicada, a fondo).

---

## Mapa de espiral por concepto

| Concepto | 🌱 Semilla (desde el inicio) | 🌿 Crece | 🌳 Domina |
|---|---|---|---|
| **Decider (`decide`/`evolve`)** | §03 — el `Apply` se nombra como `evolve`; "decidir vs evolucionar" | §07-08 — emitir eventos = `decide` | §nueva — Decider puro + Aggregate Handler |
| **Event Versioning / Upcasting** | §01/§03 — "el evento es eterno: ¿qué pasa si cambia?" (caja de aviso) | §12 — al serializar con Marten | §nueva — upcasting |
| **Idempotencia** | §01 — comando rechazable vs evento irrechazable; "¿y si llega dos veces?" | §13 — al enrutar | §14 — Outbox at-least-once + dedup |
| **Concurrencia optimista (`Version`)** | §03 — el orden importa; "¿y si dos editan a la vez?" | §06 — event store | §16/nueva — `FetchForWriting`, `ConcurrencyException` |
| **Domain vs Integration events** | §01 — "¿quién necesita enterarse?" | §09 — eventos de dominio | §nueva — `IPrivateEvent`/`IPublicEvent` (Cosmos) |
| **Proyecciones / CQRS** | §03 — "leer no es lo mismo que escribir" | §06 — el estado es una vista | §15-16 — tipos de proyección, async daemon |
| **Aggregate Handler Workflow** | §04 — "este motor lo automatiza Marten/Wolverine" | §12 — Marten | §nueva — `[Aggregate]` + `FetchForWriting` |
| **Funciones puras / testabilidad** | §03 — `evolve` es pura (sin I/O) | §08 — handler orquesta | §nueva — testing sin mocks |
| **Reflexión vs codegen** | §04 — "el `dynamic` tiene un costo; hay mejores formas" | §06 DI | §nueva — codegen Wolverine |

---

## Cómo se ve una "semilla" (patrón de escritura)

Una semilla es **una caja corta** que nombra el concepto, despierta la pregunta y enlaza hacia adelante — sin desviar la sección. Ejemplo (para event versioning, en §03):

> [!NOTE]
> 🌱 **Semilla — Los eventos son eternos.** `PersonaNacida` quedará escrito para siempre. ¿Qué pasa el día que el negocio quiera añadirle un campo `País`? No puedes editar el pasado. Ese problema tiene solución (se llama *versionado/upcasting*) y lo veremos a fondo más adelante. Por ahora, quédate con la idea: **un evento es un contrato inmutable con el futuro**.

Reglas de la semilla:
1. **Corta** (3-5 líneas). No explica la solución completa, planta la pregunta.
2. **Vocabulario correcto desde el día 1** (`evolve`, `decide`, `upcasting`, `idempotente`).
3. **Enlace hacia adelante** a la sección donde se domina.
4. **No rompe la narrativa** de Jhon.

---

## Principio
> Cuando un concepto avanzado aparece por primera vez como semilla, el alumno lo "ancla" sin carga cognitiva. Al reencontrarlo más profundo, ya tiene un percha mental. Esto es exactamente lo contrario a "dejar todo lo difícil para los últimos capítulos".

Las ediciones concretas de cada sección se registran aquí a medida que se hacen (ver estado en [ROADMAP](./ROADMAP.md)).

---

## ✅ Estado de integración (semillas plantadas)

| Sección | Semillas añadidas |
|---------|-------------------|
| §01 El diario de Jhon | 🌱 Domain vs Integration events ("¿quién necesita enterarse?") → enlaza a §01b |
| §01b El mapa de contextos | 🟢 **Encuadre temprano**: Bounded Context + 2º BC (Registro Civil) + dentro/fuera, anticipando §14/§24/§25/§26 |
| §03 Vivir el pasado | 🌱 Event versioning (eventos eternos) · 🌱 `evolve` como función pura (Decider) |
| §04 Refactorizando el motor | 🌱 Aggregate Handler Workflow (Marten/Wolverine automatizan este motor) |
| §06 El almacén en memoria | 🌱 Concurrencia optimista (conflicto → `ConcurrencyException`/reintento) · 🌱 Proyecciones/CQRS (el estado es una vista) |
| §07 Decidir el futuro | 🌱 `decide` (completa el par Decider) · 🌱 Idempotencia (¿y si el comando llega dos veces?) |
| §08 El Command Handler | 🌱 Funciones puras / testing sin mocks (Given-When-Then) |
| §09 El riesgo de olvidar | 🌱 Async de producción (no `.Result`/`.Wait()`→starvation, `CancellationToken`) + corrección del `.Wait()` |
| §02 Preparando el lienzo | 🌱 Meta: "cómo leer las semillas" (explica el enfoque en espiral al alumno) |
| §05 El flujo de vida | 🌱 Genéricos + restricciones (seguridad de tipos, sin casts/boxing) |
| §06 El almacén en memoria | 🌱 Concurrencia optimista (conflicto → `ConcurrencyException`/reintento) · 🌱 Proyecciones/CQRS (el estado es una vista) |
| §07 Decidir el futuro | 🌱 `decide` (completa el par Decider) · 🌱 Idempotencia (¿y si el comando llega dos veces?) |
| §08 El Command Handler | 🌱 Funciones puras / testing sin mocks (Given-When-Then) |
| §09 El riesgo de olvidar | 🌱 Async de producción (no `.Result`/`.Wait()`→starvation, `CancellationToken`) + corrección del `.Wait()` |
| §10 Inyección de Dependencias | 🌱 Desarma la "magia": DIP≠IoC≠DI≠Contenedor, pure DI, captive dependency, service locator, Wolverine-codegen |
| §11 Docker + PostgreSQL | 🌱 ACID→Outbox · 🌱 JSONB indexable→proyecciones/CQRS |
| §12 Introducción a Marten | 🌱 Versionado/upcasting al serializar JSON · 🌱 Reflexión vs codegen (Marten no usa `dynamic`) · 🌱 `FetchForWriting`/concurrencia |
| §13 Wolverine | 🌱 Middleware = composición de funciones (delegados/closures) + codegen |

**✅ Cobertura completa:** las 14 secciones del workshop principal tienen sembrados los conceptos avanzados desde temprano. Lo que queda es **escribir las secciones 🧩 dedicadas** del [ROADMAP](./ROADMAP.md) donde cada concepto se **domina** (Decider+Aggregate Handler, Event Versioning, CQRS/proyecciones, Testing, Reflexión-vs-codegen, etc.).

---

## 🐺 Espiral de conceptos propios de Wolverine

Conceptos que aparecen en la documentación de Wolverine y que también se siembran desde temprano (no solo al final):

| Concepto Wolverine | 🌱 Semilla | 🌳 Domina |
|---|---|---|
| **A-Frame / Vertical Slice** (lógica pura al centro, infra a los bordes, call stacks cortos) | §02 (cómo leer) — filosofía guía | §nueva / best-practices |
| **Cascading messages** (el handler *devuelve* mensajes; el framework los publica) | §07 (ya devuelves el evento) | §18 Aggregate Handler |
| **Railway / manejo de errores** (éxito/fallo como flujo, no try/catch ciego) | §08 (al introducir el handler) | §nueva Railway |
| **Method injection vs constructor injection** (Wolverine prefiere por método) | §10 (DI) | best-practices |
| **Dead Letter Queue** (mensajes que fallan repetidamente) | §14 (Outbox/mensajería) | §nueva DLQ |
| **Idempotencia en mensajería** | §07/§14 (ya) | §14 |
| **Sagas / process managers** (coordinar varios agregados, compensación) | §18 (un comando = un agregado → ¿y si son varios?) | §nueva Sagas |
| **Multi-tenancy** (`InvokeForTenantAsync`, FIFO por tenant) | §27 (ya) / CONCEPTOS-PROFUNDO | §25 Envelope / §nueva |
| **Anti-Corruption Layer** (evento público → comando interno) | §14 (semilla) | ✅ §24 |
| **Envelope / contexto** (payload vs sobre; in-memory sin contexto) | §10 (TenantId DEFAULT) / §14 | ✅ §25 |

**Estado:** semillas de Wolverine plantadas en §02, §07, §08, §10, §14, §18 (esta pasada). Las secciones que las *dominan* (Railway, DLQ, Sagas, Multi-tenancy) entran al ROADMAP como 🧩/💡.
