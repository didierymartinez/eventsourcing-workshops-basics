# 🐺 Ruta de Experto en Wolverine

> Plan de dominio de **WolverineFx** basado en la documentación oficial ([wolverinefx.net](https://wolverinefx.net/tutorials/)), ordenado de fundamentos a experto y **anclado a cómo Cosmos lo usa de verdad** (CritterStack en `Cosmos.BuildingBlocks`). La meta no es leer docs, es poder diseñar, depurar y explicar el bus de mensajes de Cosmos.
>
> 📄 Docs LLM-friendly (para pegar en NotebookLM): https://www.wolverinefx.io/llms-full.txt
> Método: cada nivel se estudia con un **Loop** (foco + skim → práctica → validación). Marca ✅ al validar.

---

## Nivel 0 — Fundamentos y mentalidad ⬜
*Meta: entender qué problema resuelve Wolverine y cómo piensa.*
- [What is Wolverine?](https://wolverinefx.net/introduction/what-is-wolverine)
- [Getting Started](https://wolverinefx.net/introduction/getting-started)
- [Wolverine for MediatR Users](https://wolverinefx.net/introduction/from-mediatr) — clave: vienes mentalmente de MediatR.
- [Best Practices](https://wolverinefx.net/introduction/best-practices) · [Basic Concepts](https://wolverinefx.net/guide/basics)

**Cosmos:** Wolverine es el `ICommandRouter`/`IQueryRouter`. Ya lo viste en `WolverineCommandRouter.cs`.
**Checkpoint:** explicar Wolverine vs MediatR y por qué Cosmos lo eligió (atomicidad + outbox nativos).

## Nivel 1 — Mediator y Handlers ⬜  → *Sección 13 del workshop*
*Meta: dominar el descubrimiento por convención y el ciclo de un handler.*
- [Wolverine as Mediator](https://wolverinefx.net/tutorials/mediator)
- [Message Handlers](https://wolverinefx.net/guide/handlers/) · [Discovery](https://wolverinefx.net/guide/handlers/discovery) · [Return Values](https://wolverinefx.net/guide/handlers/return-values)
- [Cascading Messages](https://wolverinefx.net/guide/handlers/cascading) · [Side Effects](https://wolverinefx.net/guide/handlers/side-effects)
- [Railway Programming](https://wolverinefx.net/tutorials/railway-programming)

**Cosmos:** descubrimiento de handlers vía `options.Discovery.IncludeAssembly(dominioAssembly)` en `WolverineExtensions`.
**Checkpoint:** escribir un handler que reciba comando + `IDocumentSession` sin instanciarlo a mano.

## Nivel 2 — Middleware y validación ⬜
*Meta: interceptar el pipeline (lo que Cosmos usa para el UnitOfWork).*
- [Custom Middleware (tutorial)](https://wolverinefx.net/tutorials/middleware) · [Middleware (guide)](https://wolverinefx.net/guide/handlers/middleware)
- [Fluent Validation Middleware](https://wolverinefx.net/guide/handlers/fluent-validation)
- [Execution Timeouts](https://wolverinefx.net/guide/handlers/timeout) · [Rate Limiting](https://wolverinefx.net/guide/handlers/rate-limiting)

**Cosmos:** `options.Policies.AddMiddleware<UnitOfWorkMiddleware>()` + `AutoApplyTransactions()`. Este es el corazón de por qué no llamas `SaveChangesAsync` a mano.
**Checkpoint:** explicar qué hace el `UnitOfWorkMiddleware` de Cosmos en el pipeline.

## Nivel 3 — Durabilidad: Inbox / Outbox con Marten 🔴 (núcleo de Cosmos) ⬜  → *Sección 14*
*Meta: el pilar anti-"mensajero muerto". Lo más importante para tu trabajo.*
- [Durable Inbox and Outbox Messaging](https://wolverinefx.net/guide/durability/)
- [Marten Integration](https://wolverinefx.net/guide/durability/marten/) · [Transactional Middleware](https://wolverinefx.net/guide/durability/marten/transactional-middleware)
- [Transactional Outbox](https://wolverinefx.net/guide/durability/marten/outbox) · [Transactional Inbox](https://wolverinefx.net/guide/durability/marten/inbox)
- [Aggregate Handlers and Event Sourcing](https://wolverinefx.net/guide/durability/marten/event-sourcing)
- [Event Forwarding to Wolverine](https://wolverinefx.net/guide/durability/marten/event-forwarding) · [Sagas](https://wolverinefx.net/guide/durability/marten/sagas)

**Cosmos:** `.IntegrateWithWolverine()`, `DurabilityMode.Solo` (serverless), y los `WolverinePublicEventSender` / `WolverinePrivateEventSender`. ⚠️ Aquí vive tu bug del `TypeLoadException` en `WolverinePublicEventSender`.
**Checkpoint:** dibujar el flujo Outbox (tabla → relay → Service Bus) y decir qué garantiza la transacción.

## Nivel 4 — Azure Service Bus: el transporte real de Cosmos 🔴 ⬜
*Meta: dominar el transporte que Cosmos usa en producción.*
- [Azure Service Bus](https://wolverinefx.net/guide/messaging/transports/azureservicebus/) · [Publishing](https://wolverinefx.net/guide/messaging/transports/azureservicebus/publishing) · [Listening](https://wolverinefx.net/guide/messaging/transports/azureservicebus/listening)
- [Session Identifiers and FIFO Queues](https://wolverinefx.net/guide/messaging/transports/azureservicebus/session-identifiers) — **directamente** tu orden FIFO por tenant.
- [Dead Letter Queues](https://wolverinefx.net/guide/messaging/transports/azureservicebus/deadletterqueues) · [Conventional Routing](https://wolverinefx.net/guide/messaging/transports/azureservicebus/conventional-routing)
- [Multi-Tenancy (ASB)](https://wolverinefx.net/guide/messaging/transports/azureservicebus/multi-tenancy) · [Emulator](https://wolverinefx.net/guide/messaging/transports/azureservicebus/emulator)

**Cosmos:** el patrón Forwarding (billingTopic/provisioningTopic → onboardingQueue) y las sesiones FIFO por tenant que documentaste en `cosmos_project_history.md` (el ajuste de `sessionHandlerOptions`, 8→48 msgs/min).
**Checkpoint:** explicar por qué `SessionId` = TenantId garantiza orden y no entrelaza eventos.

## Nivel 5 — Patrones de producción (experto) ⬜
*Meta: resiliencia y escala.*
- [Dealing with Concurrency](https://wolverinefx.net/tutorials/concurrency) · [Idempotency](https://wolverinefx.net/tutorials/idempotency)
- [Dead Letter Queues (tutorial)](https://wolverinefx.net/tutorials/dead-letter-queues)
- [Multi-Tenancy (holístico)](https://wolverinefx.net/tutorials/multi-tenancy) · [Handlers Multi-Tenancy](https://wolverinefx.net/guide/handlers/multi-tenancy)
- [Serverless Hosting](https://wolverinefx.net/guide/serverless) — por qué Cosmos usa `ExtensionDiscovery.ManualOnly` en Functions.
- [Sagas](https://wolverinefx.net/guide/durability/sagas) · [Leader Election and Agents](https://wolverinefx.net/tutorials/leader-election)

**Cosmos:** `InvokeForTenantAsync(tenantId, command)` (multi-tenant), idempotencia con versiones de agregado en Marten, y el caveat serverless de `WolverineExtensions`.
**Checkpoint:** explicar el workaround serverless de Cosmos y por qué `UseWolverine()` rompía.

## Nivel 6 — Arquitectura y entrega ⬜
*Meta: ver el bus en el contexto de toda la app.*
- [Vertical Slice Architecture](https://wolverinefx.net/tutorials/vertical-slice-architecture) · [Modular Monoliths](https://wolverinefx.net/tutorials/modular-monolith)
- [CQRS and Event Sourcing with Marten](https://wolverinefx.net/tutorials/cqrs-with-marten) — el "full Critter Stack".
- [HTTP Services with Wolverine](https://wolverinefx.net/guide/http/) · [Test Automation Support](https://wolverinefx.net/guide/testing) · [Diagnostics](https://wolverinefx.net/guide/diagnostics)

**Checkpoint final de experto:** tomar un Bounded Context real de Cosmos (ej. `ObligacionesPorPagar.Radicacion`) y explicar de punta a punta: comando → router → handler → Marten event store → outbox → Service Bus → otro BC.

---

## 📌 Cómo avanzar (anti-procrastinación)
- 1 nivel ≈ varios Loops. **Prioridad 🔴: Niveles 3 y 4** (son el día a día de Cosmos y donde está tu bug).
- Sube la fuente `llms-full.txt` a NotebookLM como "cerebro" de consulta de Wolverine.
- Al terminar cada nivel, valida el **Checkpoint** conmigo antes de marcar ✅.
- Conecta siempre con código real: ten abierto `Cosmos.BuildingBlocks/Cosmos.EventSourcing.CritterStack`.
