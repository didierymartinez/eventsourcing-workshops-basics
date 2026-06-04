# 23 - De código repetido a Middleware: descubriendo la composición de funciones

> 🧬 **Sección-arco (naive → dolor → refactor → nombre).** Aquí *descubrimos* el middleware en vez de presentarlo. Domina la semilla de §13 ("el middleware es composición de funciones") construyéndolo tú mismo, paso a paso, hasta llegar a lo que Wolverine genera.

## 🟢 Momento 1 — Lo ingenuo

Tienes tres handlers. En todos quieres **registrar un log**, **medir el tiempo** y **envolver en try/catch**. Lo natural es escribirlo en cada uno:

```csharp
public async Task Handle(AprobarOrden cmd, CancellationToken ct)
{
    var sw = Stopwatch.StartNew();
    Console.WriteLine($"[INICIO] AprobarOrden {cmd.OrdenId}");
    try
    {
        // —— lo único que de verdad cambia entre handlers ——
        var orden = await _store.GetAggregateRootAsync<Orden>(cmd.OrdenId, ct);
        orden!.Aprobar(cmd.AprobadorId);
        _store.Save(orden);
        await _store.SaveChangesAsync(ct);
        // ———————————————————————————————————————————————
    }
    catch (Exception ex)
    {
        Console.WriteLine($"[ERROR] {ex.Message}");
        throw;
    }
    finally
    {
        Console.WriteLine($"[FIN] AprobarOrden en {sw.ElapsedMilliseconds}ms");
    }
}
```

Y copias/pegas ese mismo envoltorio en `CrearOrden`, `RechazarOrden`, etc. Funciona.

## 💥 Momento 2 — El dolor

Llega un requisito: *"el log ahora debe ser JSON estructurado y mandar la duración a métricas"*. Tienes que abrir **los 50 handlers** y cambiar el mismo bloque 50 veces. Si olvidas uno, queda inconsistente. Peor: el "qué hace de verdad" el handler (3 líneas) está **ahogado** entre 12 líneas de ruido transversal.

> El problema real: una **preocupación transversal** (logging, tiempo, errores) está **mezclada y duplicada** dentro de cada caso de uso.

## 🔧 Momento 3 — El refactor

¿Y si pudiéramos escribir el envoltorio **una sola vez** y "envolver" con él cualquier handler? Para eso necesitamos poder **pasar el handler como un dato**. En C#, una función-como-dato es un **delegado** (`Func<>`/`Action<>`).

Modelamos un handler como un delegado: "recibe un comando, devuelve un Task".

```csharp
// Un handler, visto como función
public delegate Task HandlerDelegate<TCommand>(TCommand cmd, CancellationToken ct);

// El envoltorio transversal, escrito UNA vez. Recibe "lo que sigue" (next) y lo envuelve.
public static HandlerDelegate<T> ConLogging<T>(HandlerDelegate<T> next)
{
    // Devolvemos una NUEVA función que hace el ruido y por dentro llama a next
    return async (cmd, ct) =>
    {
        var sw = Stopwatch.StartNew();
        Console.WriteLine($"[INICIO] {typeof(T).Name}");
        try
        {
            await next(cmd, ct);              // ← aquí se ejecuta el handler real
        }
        catch (Exception ex)
        {
            Console.WriteLine($"[ERROR] {ex.Message}");
            throw;
        }
        finally
        {
            Console.WriteLine($"[FIN] {typeof(T).Name} en {sw.ElapsedMilliseconds}ms");
        }
    };
}
```

Ahora el handler vuelve a tener **solo su lógica**, limpia:

```csharp
HandlerDelegate<AprobarOrden> aprobar = async (cmd, ct) =>
{
    var orden = await _store.GetAggregateRootAsync<Orden>(cmd.OrdenId, ct);
    orden!.Aprobar(cmd.AprobadorId);
    _store.Save(orden);
    await _store.SaveChangesAsync(ct);
};

// Lo "envolvemos" con la preocupación transversal:
var aprobarConLog = ConLogging(aprobar);

await aprobarConLog(new AprobarOrden(id, "user-1"), ct); // se loguea + mide + atrapa errores
```

¿Y si quieres **varias** preocupaciones (validación, transacción, log)? Las **compones**, una envolviendo a la otra:

```csharp
var pipeline = ConLogging(ConValidacion(ConTransaccion(aprobar)));
// Ejecuta: log → validación → transacción → handler real → y de vuelta
```

> [!NOTE]
> Fíjate en dos cosas de C# que hicieron esto posible:
> 1. **Delegado** (`HandlerDelegate<T>`): tratar una función como un valor que puedes pasar.
> 2. **Closure**: la función que devuelve `ConLogging` "recuerda" la variable `next` aunque `ConLogging` ya terminó. El compilador guardó `next` en un objeto generado (semilla de la sección de delegados). No es magia: es una clase con un campo.

## 🏷️ Momento 4 — El nombre

Lo que acabas de construir tiene nombre: **middleware** (o *pipeline behaviors*). Cada eslabón recibe un `next` (lo que sigue), hace algo antes/después, y decide si llamarlo. Encadenarlos es **composición de funciones**: `f(g(h(x)))`.

Y ahora la conexión con el framework (semilla de §13):

```csharp
// En Wolverine, en vez de envolver a mano, declaras el middleware una vez:
options.Policies.AddMiddleware<LoggingMiddleware>();
options.Policies.AddMiddleware<UnitOfWorkMiddleware>();   // ← el de Cosmos
```

Wolverine **arma esta misma cadena por ti** — y recuerda (§22): **no la resuelve por reflexión en cada llamada, genera el código** que compone los eslabones. Si corres `dotnet run -- codegen write`, verás exactamente este envoltorio escrito en C#, igual al que acabas de hacer a mano.

> [!IMPORTANT]
> 🪐 **Ancla Cosmos.** El `UnitOfWorkMiddleware` + `AutoApplyTransactions()` que viste en §13 **son exactamente esto**: un middleware que abre la transacción antes y hace `SaveChanges` después de tu handler. Ahora sabes que no es un truco del framework — es composición de funciones que tú mismo podrías escribir.

---

### El Descubrimiento
No "aprendiste middleware": lo **necesitaste**, lo **construiste** y luego le pusiste nombre. Esa es la diferencia entre usar `AddMiddleware<T>()` por costumbre y entender que es un `next` envuelto en un delegado con una closure. Cuando el framework "haga magia" con tu pipeline, ya sabes que por debajo hay funciones compuestas — y que puedes leer el código generado.

---

[⬅️ Volver a Reflexión vs Codegen](./22-reflexion-vs-codegen.md) · [🧬 Método de arcos](../EVOLUCION-POR-CODIGO.md) · [🗺️ Roadmap](../ROADMAP.md)
