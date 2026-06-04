# 🗺️ Roadmap Evolutivo del Workshop

> Este workshop **no es una lista cerrada**: es un currículo vivo que crece. Las secciones se añaden a medida que aparecen conceptos nuevos (del código real de Cosmos, de dolores que surgen al construir, o de temas que merecen su propia sección). Aquí está el mapa completo —lo que existe y lo que viene— organizado por nivel, no por número fijo.

**Estados:** ✅ escrita · ✍️ en progreso · 🧩 planificada (hueco identificado) · 💡 candidata (idea por madurar)

**Principio rector:** *construye la magia antes de usarla* — cada herramienta (DI, Marten, Wolverine, Outbox) se introduce solo después de sufrir a mano el problema que resuelve.

---

## 🟢 Nivel Básico — El lenguaje y el porqué
*Meta: las herramientas finas de C# y los principios que sostienen todo lo demás.*

| Estado | Sección | Concepto núcleo |
|--------|---------|-----------------|
| ✅ | Inmutabilidad y Records | shallow vs deep, `readonly record struct` *(ampliar)* |
| ✅ | Interfaces vs Clases Abstractas | + default interface methods (C# 8), composición > herencia *(ampliar)* |
| 🧩 | **Genéricos y restricciones (`where T`)** | base de `GetAggregateRootAsync<T>` y discovery |
| 🧩 | **Delegados, `Func`/`Action` y composición** | semilla del middleware (pipeline = composición) |
| 🧩 | **SOLID en ejemplos reales** | DIP y OCP como columna vertebral |
| ✅ | Polimorfismo y dispatch | `switch` tipado **primero**, `dynamic` con sus costos *(replantear)* |

## 🟡 Nivel Medio — Estructura y dominio
*Meta: desacoplar el negocio de la tecnología y modelar bien.*

| Estado | Sección | Concepto núcleo |
|--------|---------|-----------------|
| ✅ | Patrón Comando | GoF vs CQRS-command + dónde validar *(ampliar)* |
| ✅ | Patrón Repositorio | + Unit of Work + cuándo es anti-patrón *(ampliar)* |
| ✅ | Inyección de Dependencias | DIP→IoC→DI→Contenedor + **mini-contenedor a mano** + captive deps *(replantear)* |
| ✅ | Lenguaje Ubicuo | + **Bounded Context** *(ampliar)* |
| ✅ | Aggregate Root | + frontera transaccional + dimensionamiento *(ampliar)* |
| ✅ | Eventos de Dominio | + **Domain vs Integration events** (público/privado Cosmos) *(ampliar)* |
| ✅ | Async/Await | + máquina de estados, starvation vs deadlock, `CancellationToken` *(ampliar)* |

## 🔴 Nivel Avanzado — Producción y "la magia desarmada"
*Meta: entender el motor de producción y operarlo.*

| Estado | Sección | Concepto núcleo |
|--------|---------|-----------------|
| ✅ | EventStream | el flujo de eventos |
| ✅ | Event Store en memoria | + **concurrencia optimista (`Version`)** *(ampliar)* |
| ✅ | Emitir eventos / Command Handler | cargar → actuar → guardar |
| ✅ | Docker + PostgreSQL (JSONB) | persistencia real |
| ✅ | Introducción a Marten | el bibliotecario experto |
| ✅ | [Wolverine](./secciones/13-wolverine.md) | bus interno + integración Marten |
| ✅ | [Outbox transaccional](./secciones/14-outbox.md) | + **Inbox** + **idempotencia** *(ampliar)* |
| ✅ | [**Reflexión vs Generación de código**](./secciones/22-reflexion-vs-codegen.md) | Wolverine genera código inspeccionable (núcleo anti-magia) |
| ✅ | [**Aggregate Handler Workflow + Decider**](./secciones/18-decider-y-aggregate-handler.md) | `[Aggregate]` + `FetchForWriting`; handlers puros (Critter Stack real) |
| ✅ | [**Event Versioning / Upcasting**](./secciones/19-versionado-de-eventos.md) | evolucionar eventos inmutables sin romper el pasado |
| ✅ | [CQRS + Proyecciones + consistencia eventual](./secciones/20-cqrs-y-proyecciones.md) | Inline vs Async (daemon) vs Live; single/multi-stream/flat-table |
| ✅ | [**Testing del dominio y de handlers**](./secciones/21-testing-sin-mocks.md) | funciones puras sin mocks (best-practice oficial) |
| 💡 | Rebuilding & Testing de proyecciones | reconstruir read models; testear proyecciones |
| 💡 | Railway programming / manejo de errores | éxito/fallo como flujo, sin try/catch ciego |
| 💡 | Idempotencia + Dead Letter Queues | reentregas, dedup, fallos repetidos |
| 💡 | Snapshots (single-stream projections) | optimizar replays largos |
| 💡 | Subscriptions / Event forwarding | reaccionar a eventos ya guardados |
| 💡 | Sagas / procesos largos | coordinación entre agregados + compensación |
| 💡 | Azure Service Bus a fondo | FIFO por tenant, dead-letter, forwarding |
| 💡 | Archiving / Compacting / GDPR | operación y cumplimiento del event store |
| 💡 | Observabilidad (OpenTelemetry) | trazas en EDA |

## 🏛️ Capstone — La Plantilla
| Estado | Sección | Concepto núcleo |
|--------|---------|-----------------|
| ✅ | [Plantilla Cosmos.BuildingBlocks](./secciones/17-plantilla-cosmos.md) | construir un BC real sobre la plantilla |
| 💡 | Reconstruir un BC real de Cosmos | OxP.Radicación de punta a punta |

---

## 🔭 Cómo evoluciona este roadmap
- Cuando aparezca un concepto nuevo (en el código de Cosmos o al construir), se agrega como **🧩 planificada** o **💡 candidata** aquí.
- Al escribir una sección, pasa a ✅ y se enlaza.
- Las marcas *(ampliar)* / *(replantear)* vienen de [`REVISION-CRITICA.md`](./REVISION-CRITICA.md) — son mejoras pendientes sobre secciones que ya existen.
- Documentos de apoyo: [Mapa Conceptual](./MAPA-CONCEPTUAL.md) · [Conceptos a Profundidad](./CONCEPTOS-PROFUNDO.md) · [Ruta de Experto en Wolverine](./WOLVERINE-RUTA-EXPERTO.md) · [Revisión Crítica](./REVISION-CRITICA.md) · [Inspiración Marten/Wolverine](./INSPIRACION-MARTEN-WOLVERINE.md).

> El orden numérico de los archivos en `secciones/` es solo histórico. **El orden de aprendizaje es este roadmap por niveles.**
