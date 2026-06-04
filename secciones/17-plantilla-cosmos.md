# 17 - La Plantilla Cosmos: Construir un Producto sobre `Cosmos.BuildingBlocks`

> Aquí se junta todo. `Cosmos.BuildingBlocks` es la **plantilla** sobre la que se construyen los productos del ERP (Órdenes de Compra, Obligaciones por Pagar, Contabilidad…). Esta guía tiene dos objetivos: (1) **entender su anatomía** y (2) darte la **receta productiva** para crear un Bounded Context nuevo sin reinventar nada. Filosofía: ya construiste a mano el motor (Secciones 1-16); aquí ves el motor de producción ya ensamblado.

> [!NOTE]
> 🔀 **Cambio de hilo (a propósito).** Hasta aquí el ejemplo fue **Jhon / Biografías**. A partir de esta sección saltamos a un **producto real de Cosmos** (`OrdenDeCompra`) para que veas el patrón en su hábitat. No es un ejemplo nuevo: es **exactamente lo mismo que ya dominas** con `Persona` —agregado, eventos, comando, handler— aplicado a un dominio del ERP. Si en algún punto te pierdes, traduce mentalmente `OrdenDeCompra` ↔ `Persona`.

---

## 🎯 Lo que la plantilla te resuelve (y por qué existe)
Todo lo que sufriste a mano en el workshop, BuildingBlocks ya lo encapsula:

| Lo que hiciste a mano | La plantilla lo da como |
|---|---|
| `AggregateRoot` con replay | `Cosmos.EventSourcing.Abstractions.AggregateRoot` |
| `IEventStore` / `InMemoryEventStore` | `IEventStore` + `MartenEventStore` |
| Enrutar comandos | `ICommandRouter` / `WolverineCommandRouter` |
| Publicar eventos a otros sistemas | `IPublicEventSender` / `IPrivateEventSender` |
| Outbox/Inbox a mano | transportes `...CritterStack.AzureServiceBus` / `.RabbitMQ` |
| Tests Given-When-Then | `Cosmos.EventSourcing.Testing.Utilities` |
| Multi-tenant | `Cosmos.MultiTenancy` |

---

## 🧩 Anatomía de la plantilla (paquete por paquete)

```
Cosmos.BuildingBlocks/
├── Cosmos.EventDriven.Abstractions/        ← contratos EDA: IEvent, IPublicEvent, IPrivateEvent, I*EventSender
├── Cosmos.EventDriven.CritterStack/        ← Wolverine*EventSender (publican eventos)
├── Cosmos.EventDriven.CritterStack.AzureServiceBus/  ← transporte real de Cosmos (Outbox/Inbox + FIFO)
├── Cosmos.EventDriven.CritterStack.RabbitMQ/         ← transporte alterno
├── Cosmos.EventSourcing.Abstractions/      ← AggregateRoot, IEventStore, ICommandHandler, IQueryHandler
│   ├── Commands/  (ICommandHandler, ICommandRouter, IEventStore)
│   └── Queries/   (IQueryHandler, IQueryRouter, IProjectionStore)
├── Cosmos.EventSourcing.CritterStack/      ← impl con Marten+Wolverine (MartenEventStore, routers, UnitOfWork)
├── Cosmos.EventSourcing.Linq.Extensions/   ← QueryableExtensions para el lado lectura
├── Cosmos.MultiTenancy/                    ← ITenantResolver, TenantResolver
└── Cosmos.EventSourcing.Testing.Utilities/ ← CommandHandlerTestBase (Given/When/Then), TestStore
```

**Patrón de diseño general:** cada capacidad se separa en **Abstractions** (los contratos, sin dependencias de framework) + **CritterStack** (la implementación con Marten/Wolverine). Tu producto depende de los *contratos*, no de la implementación — DIP en estado puro.

### Los contratos que de verdad usarás

```csharp
// AggregateRoot: tu entidad emite eventos, que se acumulan sin commitear
public abstract class AggregateRoot
{
    public string Id { get; protected set; }
    public int Version { get; protected set; }                 // ← concurrencia optimista
    public IReadOnlyList<object> UncommittedEvents { get; }
    public IPrivateEvent[] GetPrivateEvents();                  // intra-bounded-context
    public IPublicEvent[] GetPublicEvents();                    // integración entre BCs
}

// El event store que inyectas en tus handlers
public interface IEventStore : IAggregateRootReader
{
    void StartStream(AggregateRoot aggregateRoot);
    void AppendEvent(string aggregateId, object eventData);
    Task<bool> ExistsAsync<TAggregateRoot>(string id, CancellationToken ct) where TAggregateRoot : AggregateRoot;
    Task SaveChangesAsync(CancellationToken ct);
    // + GetAggregateRootAsync<T>(id, ct) heredado de IAggregateRootReader
}

// Tus handlers implementan una de estas
public interface ICommandHandlerAsync<TCommand> { Task HandleAsync(TCommand c, CancellationToken ct); }
public interface ICommandHandlerAsync<TCommand, TResult> { Task<TResult> HandleAsync(TCommand c, CancellationToken ct); }

// Tus eventos se marcan según su alcance
public interface IPrivateEvent : IEvent { }   // se queda en tu BC
public interface IPublicEvent  : IEvent { }   // cruza a otros BC por el bus
```

> [!IMPORTANT]
> La separación `GetPrivateEvents()` / `GetPublicEvents()` es la clave del diseño de Cosmos: el mismo agregado emite ambos, y la infraestructura enruta los privados dentro del BC y los públicos al Service Bus. Por eso **marcar bien cada evento** (`: IPrivateEvent` o `: IPublicEvent`) es una decisión de arquitectura, no un detalle.

---

## 🍳 Receta productiva: crear un producto nuevo (ej. Órdenes de Compra)

Esto es lo que harías para un Bounded Context nuevo. Cada paso usa exactamente la plantilla.

**1. Define los eventos** (marca su alcance):
```csharp
public record OrdenDeCompraCreada(string Id, string Proveedor, decimal Total) : IPrivateEvent;
public record OrdenDeCompraAprobada(string Id, string AprobadorId)            : IPublicEvent; // otros BC se enteran
```

**2. Define el Aggregate Root** (la regla de negocio vive aquí):
```csharp
public class OrdenDeCompra : AggregateRoot
{
    public string Proveedor { get; private set; } = "";
    public decimal Total { get; private set; }
    public bool Aprobada { get; private set; }

    public static OrdenDeCompra Crear(string id, string proveedor, decimal total)
    {
        var orden = new OrdenDeCompra();
        orden.RaiseEvent(new OrdenDeCompraCreada(id, proveedor, total));
        return orden;
    }

    public void Aprobar(string aprobadorId)
    {
        if (Aprobada) throw new InvalidOperationException("La orden ya está aprobada."); // invariante
        RaiseEvent(new OrdenDeCompraAprobada(Id, aprobadorId));
    }

    // Apply por cada evento — Marten los reaplica al rehidratar (replay)
    private void Apply(OrdenDeCompraCreada e) { Id = e.Id; Proveedor = e.Proveedor; Total = e.Total; }
    private void Apply(OrdenDeCompraAprobada e) { Aprobada = true; }
}
```
*(`RaiseEvent` = `Apply(e)` + `_uncommittedEvents.Add(e)`, como en el patrón de la plantilla.)*

**3. Implementa el Command Handler** (orquesta: cargar → actuar → guardar):
```csharp
public record AprobarOrdenDeCompra(string OrdenId, string AprobadorId);

public class AprobarOrdenDeCompraHandler(IEventStore eventStore) : ICommandHandlerAsync<AprobarOrdenDeCompra>
{
    public async Task HandleAsync(AprobarOrdenDeCompra cmd, CancellationToken ct)
    {
        var orden = await eventStore.GetAggregateRootAsync<OrdenDeCompra>(cmd.OrdenId, ct);
        orden!.Aprobar(cmd.AprobadorId);
        eventStore.Save(orden);
        await eventStore.SaveChangesAsync(ct);   // Wolverine+Marten: transacción + Outbox automáticos
    }
}
```

**4. Testea con Given-When-Then** (sin base de datos, puro):
```csharp
public class AprobarOrdenTests : CommandHandlerTestBase
{
    [Fact]
    public async Task Aprobar_emite_evento_publico()
    {
        Given(new OrdenDeCompraCreada(AggregateId, "ACME", 1000m));          // historia previa
        var handler = new AprobarOrdenDeCompraHandler(EventStore);
        await handler.HandleAsync(new AprobarOrdenDeCompra(AggregateId, "user-1"), default);
        Then(new OrdenDeCompraAprobada(AggregateId, "user-1"));              // evento esperado
    }
}
```

**5. Registra en `Program.cs`** (Wolverine descubre handlers, Marten persiste, transporte publica). Es la config de `WolverineExtensions` que ya viste en la Sección 13.

Eso es **todo**. No escribes SQL, ni manejo de transacciones, ni Outbox, ni serialización. La plantilla lo aporta.

---

## 📊 Matriz de conceptos por prioridad (productividad primero)

Todos los conceptos que necesitas, ordenados por **qué te hace productivo YA** vs qué profundizar después. Esta es la respuesta a "nómbralos todos, pero con criterio de productividad".

### 🟢 Imprescindibles — para construir un BC hoy
| Concepto | Por qué / dónde en la plantilla |
|---|---|
| **Records inmutables** | comandos y eventos (`record … : IPrivateEvent`) |
| **Aggregate Root: emitir + `Apply`** | tu regla de negocio; `RaiseEvent` |
| **`IPrivateEvent` vs `IPublicEvent`** | marcar el alcance de cada evento |
| **`ICommandHandlerAsync<T>` + `IEventStore`** | el corazón de cada caso de uso |
| **`async/await` + `CancellationToken`** | toda API de la plantilla lo exige |
| **Genéricos básicos** | `GetAggregateRootAsync<OrdenDeCompra>(…)` |
| **Inyección por constructor** | recibes `IEventStore` (no lo construyes) |
| **Lenguaje ubicuo + naming** | comando imperativo, evento en pasado |
| **Testing Given-When-Then** | `CommandHandlerTestBase` |

### 🟡 Importantes — para no romper cosas y entender el porqué
| Concepto | Por qué |
|---|---|
| **DIP / IoC** | por qué dependes de `IEventStore` y no de Marten |
| **Unit of Work / transacción por handler** | `UnitOfWorkMiddleware`, `AutoApplyTransactions()` |
| **Outbox/Inbox + at-least-once → idempotencia** | entrega garantizada pero duplicable |
| **Domain vs Integration events** | decidir privado vs público correctamente |
| **Bounded Context** | dónde vive y qué expone tu producto |
| **Concurrencia optimista (`Version`)** | dos comandos al mismo stream → conflicto |
| **Multi-tenancy** | `ITenantResolver`, `InvokeForTenantAsync` |
| **Dispatch tipado (`switch`) y replay** | cómo Marten reconstruye el agregado |

### 🔵 Profundizar después — maestría y operación
| Concepto | Por qué |
|---|---|
| **Reflexión vs generación de código** | Wolverine genera código inspeccionable (no magia) |
| **Azure Service Bus: FIFO por tenant, dead-letter** | el transporte real; tu bug de `WolverinePublicEventSender` |
| **CQRS: proyecciones / read models** | `IProjectionStore`, `QueryableExtensions` |
| **Snapshots** | optimizar replays largos |
| **SOLID completo, composición > herencia** | diseño sostenible |
| **Delegados / composición de funciones** | desarma el middleware de Wolverine |
| **`ConfigureAwait`, `ValueTask`, `async void`** | afinado async |
| **Durability modes / serverless** | por qué `DurabilityMode.Solo` en Functions |

---

## 🧭 Cómo estudiar la plantilla (anti-cargo-cult)
1. Abre `Cosmos.EventSourcing.Abstractions` **primero** — son contratos, se leen en minutos y definen el "qué".
2. Luego `Cosmos.EventSourcing.CritterStack` — el "cómo" con Marten/Wolverine. Reconoce cada pieza del workshop.
3. Toma un BC real (`ObligacionesPorPagar.Radicacion`) y rastrea **un** caso de uso: comando → handler → eventStore → evento privado/público → bus.
4. Valida con un **Loop**: explica de memoria la receta de 5 pasos sin mirar.

> Meta de maestría: poder crear un Bounded Context nuevo en Cosmos partiendo solo de los contratos de `BuildingBlocks`, y explicar por qué cada pieza existe.

---

[⬅️ Volver al inicio del workshop](../README.md)
