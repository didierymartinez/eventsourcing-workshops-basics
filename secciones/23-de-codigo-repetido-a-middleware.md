# 23 - De código repetido a Middleware: descubriendo la composición de funciones

> 🎯 **Hacia dónde va:** dominamos **delegados (`Func`/`Action`), closures y composición de funciones** construyendo un middleware a mano. Es una sección de **Nivel Básico** que se lee **temprano** (antes de Inyección de Dependencias §10 y Wolverine §13): esas secciones darán por sabido lo que aquí construyes.

> 🧬 **Sección-arco (naive → dolor → refactor → nombre).** Aquí *descubrimos* el middleware en vez de presentarlo: partimos de código repetido y refactorizamos hasta la composición de funciones. Más adelante reconocerás que **esto es exactamente lo que Wolverine genera** por debajo (§13) — pero el concepto de C# es tuyo desde ya.

> [!NOTE]
> 🔤 Esta sección es **C# puro**: no usa base de datos, ni eventos, ni el event store. Solo operaciones de una mini-app (enviar un correo, generar un reporte…) representadas con `Console.WriteLine` y un `Task.Delay` que *simula* el trabajo. Concéntrate en el patrón **"código que se repite en cada método"** y en cómo lo factorizamos — eso es lo único que importa aquí.

## 🟢 Momento 1 — Lo ingenuo

Tu app tiene varias operaciones: enviar un correo de bienvenida, generar un reporte, exportar un CSV. En **todas** quieres lo mismo: **registrar un log**, **medir el tiempo** y **envolver en try/catch**. Lo natural es escribirlo en cada una:

```csharp
public record EnviarBienvenida(string Email);

public async Task EjecutarAsync(EnviarBienvenida cmd, CancellationToken ct)
{
    var sw = Stopwatch.StartNew();
    Console.WriteLine($"[INICIO] EnviarBienvenida {cmd.Email}");
    try
    {
        // —— lo único que de verdad cambia entre operaciones ——
        await Task.Delay(20, ct);                       // (simula el trabajo real: enviar el correo)
        Console.WriteLine($"¡Bienvenido, {cmd.Email}!");
        // ———————————————————————————————————————————————
    }
    catch (Exception ex)
    {
        Console.WriteLine($"[ERROR] {ex.Message}");
        throw;
    }
    finally
    {
        Console.WriteLine($"[FIN] EnviarBienvenida en {sw.ElapsedMilliseconds}ms");
    }
}
```

Y copias/pegas ese mismo envoltorio en `GenerarReporte`, `ExportarCsv`, etc. Funciona.

## 💥 Momento 2 — El dolor

Llega un requisito: *"el log ahora debe ser JSON estructurado y mandar la duración a métricas"*. Tienes que abrir **las 50 operaciones** y cambiar el mismo bloque 50 veces. Si olvidas una, queda inconsistente. Peor: el "qué hace de verdad" la operación (2-3 líneas) está **ahogado** entre 12 líneas de ruido transversal.

> El problema real: una **preocupación transversal** (logging, tiempo, errores) está **mezclada y duplicada** dentro de cada caso de uso.

## 🔧 Momento 3 — El refactor

¿Y si pudiéramos escribir el envoltorio **una sola vez** y "envolver" con él cualquier operación? Para eso necesitamos poder **pasar la operación como un dato**. En C#, una función-como-dato es un **delegado** (`Func<>`/`Action<>`).

Modelamos una operación como un delegado: "recibe un mensaje (aquí `EnviarBienvenida`), devuelve un `Task`".

```csharp
// Una operación, vista como función
public delegate Task Operacion<TCommand>(TCommand cmd, CancellationToken ct);

// El envoltorio transversal, escrito UNA vez. Recibe "lo que sigue" (next) y lo envuelve.
public static Operacion<T> ConLogging<T>(Operacion<T> next)
{
    // Devolvemos una NUEVA función que hace el ruido y por dentro llama a next
    return async (cmd, ct) =>
    {
        var sw = Stopwatch.StartNew();
        Console.WriteLine($"[INICIO] {typeof(T).Name}");
        try
        {
            await next(cmd, ct);              // ← aquí se ejecuta la operación real
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

Ahora la operación vuelve a tener **solo su lógica**, limpia:

```csharp
Operacion<EnviarBienvenida> enviar = async (cmd, ct) =>
{
    await Task.Delay(20, ct);                  // el trabajo real, sin ruido transversal
    Console.WriteLine($"¡Bienvenido, {cmd.Email}!");
};

// La "envolvemos" con la preocupación transversal:
var enviarConLog = ConLogging(enviar);

await enviarConLog(new EnviarBienvenida("jhon@mail.com"), ct); // se loguea + mide + atrapa errores
```

¿Y si quieres **varias** preocupaciones (validación, transacción, log)? Las **compones**, una envolviendo a la otra:

```csharp
var pipeline = ConLogging(ConValidacion(ConTransaccion(enviar)));
// Ejecuta: log → validación → transacción → operación real → y de vuelta
```

> [!NOTE]
> Fíjate en dos cosas de C# que hicieron esto posible:
> 1. **Delegado** (`Operacion<T>`): tratar una función como un valor que puedes pasar.
> 2. **Closure**: la función que devuelve `ConLogging` "recuerda" la variable `next` aunque `ConLogging` ya terminó. El compilador guardó `next` en un objeto generado por debajo. No es magia: es una clase con un campo.

## 🏷️ Momento 4 — El nombre

Lo que acabas de construir tiene nombre: **middleware** (o *pipeline behaviors*). Cada eslabón recibe un `next` (lo que sigue), hace algo antes/después, y decide si llamarlo. Encadenarlos es **composición de funciones**: `f(g(h(x)))`.

Y ahora una **semilla hacia adelante**: cuando llegues a Wolverine (§13), declararás esto mismo sin envolver a mano —

```csharp
// En Wolverine, en vez de envolver a mano, declaras el middleware una vez:
options.Policies.AddMiddleware<LoggingMiddleware>();
options.Policies.AddMiddleware<UnitOfWorkMiddleware>();   // ← el de Cosmos
```

— y Wolverine **armará esta misma cadena por ti**. Más aún (lo verás en §22): **no la resuelve por reflexión en cada llamada, genera el código** que compone los eslabones. Si corres `dotnet run -- codegen write`, verás exactamente este envoltorio escrito en C#, igual al que acabas de hacer a mano.

> [!IMPORTANT]
> 🪐 **Ancla Cosmos (te la encontrarás en §13).** El `UnitOfWorkMiddleware` + `AutoApplyTransactions()` de Cosmos **son exactamente esto**: un middleware que abre la transacción antes y hace `SaveChanges` después de tu operación. Cuando lo veas allá, no será un truco del framework — es la composición de funciones que acabas de escribir tú mismo aquí.

---

### El Descubrimiento
No "aprendiste middleware": lo **necesitaste**, lo **construiste** y luego le pusiste nombre. Esa es la diferencia entre usar `AddMiddleware<T>()` por costumbre y entender que es un `next` envuelto en un delegado con una closure. Cuando el framework "haga magia" con tu pipeline, ya sabes que por debajo hay funciones compuestas — y que puedes leer el código generado.

---

> 📍 **Ubicación de lectura:** esta es una sección de **Nivel Básico** (concepto de C#), aunque su archivo sea el 23. Sigue el orden del [🗺️ Roadmap](../ROADMAP.md), no el número. Retomarás lo que construiste aquí al llegar a **Inyección de Dependencias (§10)** y **Wolverine (§13)**.

[🗺️ Roadmap (orden de lectura)](../ROADMAP.md) · [🧬 Método de arcos](../EVOLUCION-POR-CODIGO.md)
