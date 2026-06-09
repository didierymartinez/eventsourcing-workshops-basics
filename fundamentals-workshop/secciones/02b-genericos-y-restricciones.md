# 02b - Un molde para cualquier tipo: Genéricos y restricciones (`where T`)

> 🎯 **Hacia dónde va:** los genéricos son lo que permite escribir **una sola** clase `EventStream<T>` y **un solo** método `GetAggregateRootAsync<T>(id)` que sirven para `Persona`, `Orden` o cualquier agregado — sin copiar/pegar. Y las **restricciones** (`where T : ...`) son las que le dan al compilador permiso para usar `T` con seguridad. Es la base del store de Cosmos.

> 🧬 **Sección-arco (naive → dolor → nombre).** Llegamos a los genéricos por necesidad: primero duplicamos código, sentimos el dolor, y luego descubrimos el molde con hueco.

## 🟢 Momento 1 — Lo ingenuo

En la sección anterior definiste un repositorio para `Persona`. Mañana necesitas uno para `Mascota`, y luego para `Empresa`. Lo natural es copiar/pegar:

```csharp
public class PersonaRepository
{
    private readonly Dictionary<Guid, Persona> _data = new();
    public Persona? Get(Guid id) => _data.GetValueOrDefault(id);
    public void Save(Persona p) => _data[p.Id] = p;
}

public class MascotaRepository
{
    private readonly Dictionary<Guid, Mascota> _data = new();   // ← idéntico, solo cambia el tipo
    public Mascota? Get(Guid id) => _data.GetValueOrDefault(id);
    public void Save(Mascota m) => _data[m.Id] = m;
}
```

## 💥 Momento 2 — El dolor

`PersonaRepository` y `MascotaRepository` son **la misma clase**: solo cambia el tipo que guardan. Si mejoras el `Get` (añadir caché, logging, lo que sea), tienes que cambiarlo en cada copia. La lógica es idéntica; lo único que varía es **un tipo**. C# tiene una herramienta exacta para "lo único que cambia es un tipo": los **genéricos**.

## 🔧 Momento 3 — El molde con hueco

Un genérico es una clase (o método) con un **hueco para un tipo**, escrito como `<T>`. Lo rellenas al usarlo:

```csharp
public class Repository<T>                       // T = el hueco
{
    private readonly Dictionary<Guid, T> _data = new();
    public T? Get(Guid id) => _data.GetValueOrDefault(id);
    public void Save(Guid id, T item) => _data[id] = item;
}

// Al usarlo, rellenas el hueco:
var personas = new Repository<Persona>();        // T = Persona
var mascotas = new Repository<Mascota>();        // T = Mascota
```

Una clase, todos los tipos. El compilador genera la versión concreta de cada `T` que uses, con **chequeo de tipos completo** (no es `object` con casts: `personas.Get(id)` devuelve un `Persona`, no un `object`).

## 🚧 El problema: `T` es demasiado libre

Hay un detalle. Dentro de `Repository<T>`, `T` podría ser **cualquier cosa** — `int`, `string`, `Persona`… Eso significa que el compilador **no te deja** asumir nada sobre `T`. Mira qué pasa si intentas usar `item.Id`:

```csharp
public void Save(T item) => _data[item.Id] = item;  // ❌ ERROR: 'T' no tiene 'Id'
```

El compilador tiene razón: si `T` fuera `int`, no hay `Id`. Necesitamos **prometerle** que `T` siempre tendrá ciertas características. Esa promesa es una **restricción** (`where`).

## 🏷️ El nombre: restricciones (`where T : ...`)

```csharp
public class Repository<T> where T : AggregateRoot, new()
{
    public void Save(T item) => _data[item.Id] = item;   // ✅ ahora T SIEMPRE tiene Id (es un AggregateRoot)
    public T Crear() => new T();                          // ✅ y SIEMPRE se puede construir vacío
}
```

Las restricciones más usadas en Event Sourcing:

| Restricción | Qué promete | Para qué la necesitas |
|---|---|---|
| `where T : AggregateRoot` | `T` hereda de `AggregateRoot` | poder usar `item.Id` y `item.Load(...)` dentro del genérico |
| `where T : new()` | `T` tiene constructor vacío público | poder hacer `new T()` para rehidratar desde cero |
| `where T : IEvent` | `T` cumple un contrato (interfaz) | restringir a "solo eventos", no cualquier objeto |
| `where T : class` | `T` es tipo referencia | permitir `null` y comparaciones por referencia |

> [!NOTE]
> 🔤 **Recordatorio de §02:** en `ICommandHandler<in TCommand>`, el `in` es **varianza** (contravarianza), algo distinto de las restricciones. La varianza dice *cómo se sustituyen* los tipos genéricos entre sí; la restricción (`where`) dice *qué características* debe cumplir `T`. Dos herramientas, dos propósitos.

## 🧰 Genéricos en métodos (no solo en clases)

El hueco `<T>` también va en un método suelto. Así nace el método estrella del store:

```csharp
// "Dame el agregado de tipo T con este id". UN método, todos los agregados.
public async Task<T> GetAggregateRootAsync<T>(Guid id, CancellationToken ct = default)
    where T : AggregateRoot, new()
{
    var eventos = await LeerEventosAsync(id, ct);
    var agg = new T();           // gracias a new()
    agg.Load(eventos);           // gracias a AggregateRoot
    return agg;
}

// Uso: el compilador infiere o tú especificas el tipo
var jhon  = await store.GetAggregateRootAsync<Persona>(idJhon, ct);
var orden = await store.GetAggregateRootAsync<Orden>(idOrden, ct);
```

> [!IMPORTANT]
> 🪐 **Ancla Cosmos.** Esto **no es teoría**: `IEventStore.GetAggregateRootAsync<T>(...)` con `where T : AggregateRoot, new()` es exactamente la firma del store en `Cosmos.BuildingBlocks`. Las dos restricciones son las que viste arriba: `AggregateRoot` (para hacer `Load`) y `new()` (para construir el agregado vacío antes de rehidratarlo). Cuando llegues al workshop principal (§05), construirás esta misma clase `EventStream<T>` a mano.

---

### El Descubrimiento
Un genérico no es "código avanzado": es **el mismo molde para muchos tipos**, y la restricción `where T` es la **promesa mínima** que le haces al compilador para poder usar `T` con seguridad. Sin genéricos, el store sería un `PersonaStore`, un `OrdenStore`, un `MascotaStore`… todos iguales. Con genéricos, es **uno solo**.

---

[⬅️ Volver a la sección anterior](./02-interfaces-vs-abstract-classes.md) | [➡️ Siguiente sección: La Magia del Enrutamiento (Polimorfismo)](./03-polimorfismo-y-dynamic-dispatching.md)
