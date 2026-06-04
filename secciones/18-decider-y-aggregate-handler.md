# 18 - El Decider y el Aggregate Handler Workflow

> 🌳 **Sección donde se dominan las semillas** de §03 (`evolve`), §07 (`decide`), §04 (el motor automatizado), §06 y §12 (`FetchForWriting`/concurrencia). Aquí juntamos todo en el patrón real de producción del Critter Stack.

En la Sección 08 escribiste el Command Handler completo: cargar → actuar → guardar. Funciona, pero si te fijas, **cada handler repite el mismo ritual**:

```csharp
public async Task HandleAsync(RegistrarMatrimonio cmd, CancellationToken ct)
{
    var stream = new EventStream<Persona>(_store, cmd.PersonaId);
    var persona = await stream.GetAsync();        // 1. cargar (boilerplate)
    var evento  = persona.RegistrarMatrimonio(cmd.NombrePareja); // 2. la ÚNICA línea que importa
    await stream.AppendAsync(evento);             // 3. guardar (boilerplate)
}
```

Las líneas 1 y 3 son **siempre iguales**. Lo único que cambia entre handlers es la línea 2: **la decisión**. ¿Y si pudiéramos escribir *solo* la decisión?

---

## 1. Primero, el concepto puro: el patrón Decider

Recoge las semillas de §03 y §07. Todo agregado en Event Sourcing se puede expresar con **dos funciones puras**:

```
decide(estado, comando) -> eventos      // ¿qué debe pasar? (valida reglas, NO muta nada)
evolve(estado, evento)  -> nuevo estado  // ¿cómo cambia el estado? (= tu método Apply)
```

- **`decide`** mira el estado actual y un comando, aplica las reglas de negocio, y **devuelve los eventos** que deben ocurrir (o lanza/rechaza si la regla no se cumple). No toca base de datos ni red.
- **`evolve`** toma el estado y un evento ya ocurrido, y produce el nuevo estado. Es exactamente el `Apply` que construiste en la Sección 03.

Ambas son **puras**: mismas entradas → mismas salidas, sin efectos secundarios. En nuestra `Persona`, `RegistrarMatrimonio` es el `decide` y `Apply(PersonaCasada)` es el `evolve`:

```csharp
public class Persona : AggregateRoot
{
    public bool Casado { get; private set; }

    // decide: estado + intención -> evento(s), validando la regla
    public PersonaCasada RegistrarMatrimonio(string pareja)
    {
        if (Casado) throw new InvalidOperationException("Jhon ya está casado."); // invariante
        return new PersonaCasada(Id, pareja);
    }

    // evolve: estado + hecho -> nuevo estado
    private void Apply(PersonaCasada e) => Casado = true;
}
```

> [!TIP]
> **Por qué esto es oro para los tests** (semilla de §08): como `decide` y `evolve` son puras, las pruebas son triviales y **sin mocks**: *dado* unos eventos pasados, *cuando* ejecuto un comando, *entonces* espero ciertos eventos. Nada de base de datos.

---

## 2. El dolor que falta resolver: el boilerplate y la concurrencia

El patrón Decider es hermoso, pero alguien tiene que: (a) **cargar** el estado reproduciendo eventos, (b) llamar a `decide`, (c) **guardar** los eventos resultantes, y (d) hacerlo **sin perder la concurrencia optimista** (semilla de §06/§12: si otro proceso escribió primero, debe fallar y reintentar).

Escribir eso a mano en 50 handlers es repetitivo y propenso a olvidar la versión esperada. Aquí entra el framework — pero ahora ya **sabes exactamente qué hace por debajo**.

---

## 3. El Aggregate Handler Workflow (Wolverine + Marten)

Wolverine + Marten generan el ritual cargar/guardar por ti. Tú escribes **solo el `decide`**, y devuelves los eventos:

```csharp
public record RegistrarMatrimonio(Guid PersonaId, string Pareja);

public static class RegistrarMatrimonioHandler
{
    // [Aggregate] le dice a Wolverine: carga este agregado con FetchForWriting
    // (captura la versión esperada para concurrencia optimista)
    public static IEnumerable<object> Handle(RegistrarMatrimonio cmd, [Aggregate] Persona persona)
    {
        if (persona.Casado) yield break;                       // regla de negocio (decide)
        yield return new PersonaCasada(cmd.PersonaId, cmd.Pareja); // evento emitido
    }
}
```

¿Qué hace Wolverine automáticamente alrededor de esa función pura?
1. **Carga** la `Persona` con `session.Events.FetchForWriting<Persona>(cmd.PersonaId)` — que reproduce los eventos **y captura la versión esperada**.
2. Ejecuta tu `Handle` (el `decide`).
3. **Appendea** los eventos que devolviste al stream.
4. Hace `SaveChangesAsync` en una transacción; si la versión cambió entre tanto → **`ConcurrencyException`** (la concurrencia optimista que sembramos), con reintento configurable.

> [!NOTE]
> Compara con la Sección 08: **desaparecieron** las líneas de cargar y guardar. No porque sean "magia", sino porque Wolverine **genera ese código** (recuerda la semilla de §10/§13: codegen, no reflexión). Puedes verlo con `dotnet run -- codegen write`.

### Variante: iniciar un stream nuevo
Cuando el agregado aún no existe (primer evento), no usas `[Aggregate]`; devuelves un side-effect que inicia el stream:

```csharp
public static (CreationResponse, IStartStream) Handle(RegistrarPersona cmd)
{
    var nacio = new PersonaNació(cmd.PersonaId, cmd.Nombre, cmd.Ciudad);
    var start = MartenOps.StartStream<Persona>(cmd.PersonaId, nacio); // side-effect puro
    return (new CreationResponse(cmd.PersonaId), start);
}
```

Y para responder con el estado ya actualizado (incluyendo los eventos recién emitidos), Marten ofrece `FetchLatest<Persona>(id)`.

---

## 4. ¿Cuándo el workflow y cuándo el `IEventStore` manual?

La plantilla de Cosmos (`Cosmos.BuildingBlocks`, Sección 17) expone **ambos** estilos:
- **`IEventStore` explícito** (`GetAggregateRootAsync` → método del agregado → `Save` → `SaveChangesAsync`): más verboso, pero control total del flujo. Útil cuando un handler coordina algo más que un solo agregado.
- **Aggregate Handler Workflow** (`[Aggregate]` + `FetchForWriting`): mínima ceremonia, ideal para el caso común "un comando muta un agregado". Es el patrón que más verás.

> [!TIP]
> Regla práctica (alineada a la best-practice de Wolverine): **handler delgado, lógica pura en el agregado**. Si tu handler crece en `if`s e llamadas a servicios, casi siempre la lógica debería estar en el `decide` del agregado, no en el handler.

---

> [!NOTE]
> 🌱 **Semilla — ¿y cuando un proceso cruza varios agregados?** El Aggregate Handler asume **un comando → un agregado** (la frontera transaccional, §08/CONCEPTOS-PROFUNDO). Pero algunos procesos de negocio abarcan varios pasos y agregados a lo largo del tiempo: *aprovisionar un tenant* (crear cuenta → aprovisionar infra → notificar) es el caso real de Cosmos. Para coordinar eso **sin** una transacción gigante existe la **saga** (o *process manager*): una máquina de estados que reacciona a eventos, emite los siguientes comandos, y si algo falla aplica **compensaciones** (no *rollback*, sino "deshacer con un nuevo evento"). Wolverine + Marten las soportan nativamente. Guárdalo: cuando un caso de uso "no cabe" en un solo agregado, probablemente es una saga.

### El Descubrimiento
El patrón Decider (`decide` + `evolve`) es el modelo mental; el Aggregate Handler Workflow es su forma productiva en el Critter Stack. Ahora cuando escribas `[Aggregate]` y devuelvas eventos, no es un truco: es **el ritual cargar/decidir/guardar que ya construiste a mano**, generado y con concurrencia optimista incluida.

**Pendiente que esto abre:** ¿qué pasa el día que necesites cambiar la forma de un evento que ya está guardado? Eso es **versionado de eventos / upcasting** (la semilla de §03 y §12), nuestra próxima sección dedicada.

---

[⬅️ Volver a Outbox](./14-outbox.md) · [🗺️ Roadmap](../ROADMAP.md) · [🏛️ La Plantilla Cosmos](./17-plantilla-cosmos.md)
