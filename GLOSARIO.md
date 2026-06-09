# 📖 Glosario

Referencia rápida de términos que quizá no conozcas todavía. Aquí solo hay definiciones de 1-2 líneas para no asumir nada; **cada término se explica en contexto en su sección correspondiente del workshop**.

---

## C#

- **Varianza `in` / `out` (contravarianza / covarianza)** — Modificadores en interfaces y delegados genéricos. `out` (covarianza) permite usar un tipo más derivado donde se espera uno más general (devuelve `T`); `in` (contravarianza) permite lo contrario (consume `T`).
- **Restricción `new()`** — En un genérico (`where T : new()`) exige que `T` tenga constructor público sin parámetros, para poder hacer `new T()` dentro del método.
- **Iteradores (`yield return` / `yield break`)** — `yield return` devuelve los elementos de uno en uno sin construir toda la lista; `yield break` corta la secuencia. Generan un `IEnumerable` **perezoso** (lazy): los valores se calculan solo al iterarlos.
- **`IEnumerable` perezoso** — Secuencia que no produce sus elementos hasta que la recorres (p. ej. con `foreach`); ahorra memoria y permite secuencias infinitas o diferidas.
- **`dynamic` y el DLR (Dynamic Language Runtime)** — `dynamic` desactiva la comprobación de tipos en compilación y la resuelve en ejecución mediante el DLR (la capa de tiempo de ejecución dinámica de .NET).
- **`record`** — Tipo de referencia pensado para datos inmutables, con igualdad por valor y `ToString()` legible generados automáticamente.
- **`init`** — Modificador de propiedad que solo permite asignarla durante la inicialización del objeto; después queda de solo lectura.
- **`readonly record struct`** — `record` de tipo valor (`struct`) e inmutable: combina semántica de valor, igualdad por valor e inmutabilidad, sin asignar en el heap.

### Símbolos de sintaxis (los que damos por conocidos)
- **Tipo anulable `T?`** (p. ej. `PersonaCasada?`, `string?`) — La variable puede contener un valor **o** `null`. Devolver `null` suele significar "no pasó nada" (un no-op).
- **`?.` (acceso condicional)** — `a?.B` devuelve `null` si `a` es `null` en vez de lanzar `NullReferenceException`. Encadena seguro: `lista.LastOrDefault()?.Version`.
- **`??` (coalescencia nula)** — `a ?? b` devuelve `a` si no es `null`, si no `b`. Útil para valores por defecto: `... ?? 0`.
- **`!` (null-forgiving)** — `x!` le dice al compilador "confía, esto no es null aquí"; solo silencia el aviso, no cambia el comportamiento.
- **`=>` con cuerpo de expresión** — Forma corta de un método o constructor de una sola línea: `public int Doble(int n) => n * 2;` equivale a `{ return n * 2; }`.
- **Lambda `x => ...`** — Función anónima en línea. En LINQ, `.Where(p => p.Edad > 18)` aplica esa mini-función a cada elemento. (La construimos a fondo como **delegado** en §23.)
- **Constructor primario (C# 12)** — Parámetros entre paréntesis tras el nombre de la clase: `class Router(IBus bus) : IRouter { ... }`. Esos parámetros (`bus`) quedan disponibles en toda la clase, sin declarar campos a mano.
- **Retorno por tupla** — Un método puede devolver varios valores a la vez: `(CreationResponse, IStartStream) Handle(...)` devuelve **dos** cosas en un solo `return`.
- **`out var`** — Declara una variable al pasarla como parámetro de salida: `dict.TryGetValue(k, out var valor)` crea `valor` con el resultado.
- **Sufijo `m` (decimal)** — `1000m` es un literal `decimal` (precisión exacta para dinero), distinto de `1000` (`int`) o `1000.0` (`double`).
- **`[Fact]` (xUnit)** — Atributo que marca un método como **test** automatizado. xUnit lo detecta y lo ejecuta.
- **`.Should()...` (FluentAssertions / AwesomeAssertions)** — API fluida de aserciones para tests: `resultado.Should().BeOfType<PersonaCasada>()` o `act.Should().Throw<...>()` se leen casi como inglés.

## Azure / infraestructura

- **Azure Service Bus** — Servicio de mensajería de Azure (colas y temas) para comunicar componentes de forma asíncrona y fiable.
- **Azure Functions (.NET isolated)** — Funciones serverless que ejecutan código por eventos. El modelo *isolated* corre tu código en un proceso aparte del host, dándote control total sobre el runtime de .NET.
- **Sesiones FIFO / `SessionId`** — En Service Bus, agrupar mensajes con el mismo `SessionId` garantiza que se procesen en orden (First-In-First-Out) dentro de esa sesión.
- **RabbitMQ** — Broker de mensajería de código abierto, alternativa común a Service Bus para colas y enrutamiento de mensajes.
- **Cold start (arranque en frío)** — Latencia extra la primera vez que se invoca una función serverless inactiva, porque debe iniciar su entorno antes de responder.

## Datos / arquitectura

- **Desnormalizar** — Duplicar o precombinar datos a propósito (en vez de normalizarlos) para acelerar las lecturas, a costa de redundancia.
- **JSONB** — Tipo de PostgreSQL que almacena JSON en formato binario indexable y consultable, ideal para guardar eventos y documentos.
- **Consistencia eventual** — Tras un cambio, las distintas vistas o réplicas convergen al mismo estado "con el tiempo", no de forma instantánea.
- **Idempotencia** — Propiedad de una operación que, repetida varias veces, produce el mismo resultado que ejecutarla una sola vez (clave para reintentos seguros).
- **Outbox / Inbox** — Patrones de mensajería fiable: *Outbox* guarda el mensaje en la misma transacción que los datos para enviarlo luego sin perderlo; *Inbox* registra los mensajes ya recibidos para no procesarlos dos veces.
- **Bounded Context** — En DDD, una frontera explícita dentro de la cual un modelo y su lenguaje tienen un significado único y consistente.
- **Aggregate Root** — Entidad principal de un agregado (grupo de objetos que cambian juntos); es la única puerta de entrada para modificar ese grupo y garantizar sus reglas.
- **CritterStack (Marten + Wolverine)** — Nombre del ecosistema que une **Marten** (Event Sourcing y documentos sobre PostgreSQL) y **Wolverine** (mensajería y manejo de comandos) en .NET.
- **Proyección / read model** — Vista derivada de los eventos, optimizada para leer. La *proyección* es el proceso que transforma eventos en ese *read model* (modelo de lectura).
