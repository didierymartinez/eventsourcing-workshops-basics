# 12 - El Bibliotecario Experto: Introducción a Marten

En la Sección 11 encendimos nuestra bóveda de concreto (PostgreSQL en Docker) y dejamos al `InMemoryEventStore` listo para ser jubilado.

Y si recordamos el dolor de las Secciones 05 y 06, construimos a mano: el sobre (`EventoAlmacenado`), el gestor de versiones, el diccionario en RAM (`InMemoryEventStore`) y el envoltorio lógico (`EventStream<T>`).

Imagina tener que repetir ahora todo ese trabajo pero contra una base de datos real: escribir sentencias SQL a mano, manejar conexiones, serializar JSON, controlar transacciones...

Aquí es donde entra **Marten**: la librería que hace todo ese trabajo sucio por ti con elegancia de cirujano.

---

## 🚀 ¿Qué es Marten?

Marten es una librería de .NET que convierte a PostgreSQL en dos cosas al mismo tiempo:

1. **Una Base de Datos Documental** — guarda objetos C# como JSON directamente (sin mapear tablas).
2. **Un Event Store nativo de producción** — reemplaza todo lo que construimos a mano en las Secciones 05 y 06.

Marten es el "Bibliotecario Experto": conoce el idioma de PostgreSQL, aplica el patrón Unit of Work, y nos entrega los Agregados rehidratados sin que escribamos ni una línea de SQL.

---

## 🗺️ El Mapa de la Traducción

Antes de tocar código, veamos exactamente qué piezas nuestras reemplaza Marten:

| Lo que construimos a mano | Lo que Marten nos da |
|---|---|
| `EventoAlmacenado` (el sobre) | Marten lo maneja internamente |
| `InMemoryEventStore` (el diccionario) | `IDocumentStore` (motor de Marten) |
| `EventStream<T>.Get()` (rehidratar) | `session.Events.AggregateStreamAsync<T>(id)` |
| `EventStream<T>.Append()` (guardar) | `session.Events.Append(id, events)` |
| `SaveChangesAsync()` | `session.SaveChangesAsync()` |
| `IEventStore` (la interfaz) | `IDocumentSession` (la súper-interfaz) |

> [!NOTE]
> **¿Recuerdas la nota filosófica de la Sección 06?**
> Aquí lo ves en la práctica: Marten fusiona el Repositorio Lógico y el Store Físico en una sola interfaz (`IDocumentSession`). No tienes que escribir ninguna clase puente intermediaria.

---

## 1. Instalación

En tu proyecto de consola (.NET 8+), instala el paquete de Marten:

```bash
dotnet add package Marten
```

---

## 2. Configuración en el Program.cs

Reemplazamos el registro manual del `InMemoryEventStore` por Marten. Todo en unas pocas líneas:

```csharp
using Marten;
using Weasel.Core;

var builder = WebApplication.CreateBuilder(args);

// La misma ConnectionString de la Sección 11
var connectionString = "Host=localhost;Port=5432;Database=eventstore_db;Username=admin;Password=Password123!";

builder.Services.AddMarten(options =>
{
    options.Connection(connectionString);

    // Para desarrollo: Marten crea y actualiza las tablas automáticamente.
    // No necesitas escribir ni una migración SQL.
    options.AutoCreateSchemaObjects = AutoCreate.All;

    // Le decimos a Marten qué tipos son Eventos de Dominio para que los trate correctamente
    options.Events.AddEventType<PersonaNacida>();
    options.Events.AddEventType<CumpleañosCelebrado>();
    options.Events.AddEventType<HijoNacido>();
});
```

> [!TIP]
> `AutoCreate.All` es equivalente a que Marten ejecute automáticamente `CREATE TABLE IF NOT EXISTS` al arrancar la app. En producción se recomienda usar `AutoCreate.None` y correr las migraciones de forma controlada.

---

## 3. Guardando el primer evento (Append)

En los Command Handlers, en vez de inyectar `IEventStore`, ahora inyectamos `IDocumentSession`.

Así se ve registrar el nacimiento de Jhon contra PostgreSQL:

```csharp
public class RegistrarPersonaHandler
{
    private readonly IDocumentSession _session;

    public RegistrarPersonaHandler(IDocumentSession session)
    {
        _session = session;
    }

    public async Task Handle(RegistrarPersona comando)
    {
        var idJhon = Guid.NewGuid();

        // Creamos el Evento de Dominio puro (igual que antes)
        var evento = new PersonaNacida(idJhon, comando.Nombre, DateTime.UtcNow, comando.Ciudad);

        // Marten recibe el evento y lo asocia al stream de ese ID
        // (internamente lo empaqueta en su propio "sobre" con Version y Timestamp)
        _session.Events.StartStream<Persona>(idJhon, evento);

        // Aquí es donde ocurre la magia: Marten serializa el evento a JSON
        // y lo persiste en PostgreSQL dentro de una transacción ACID
        await _session.SaveChangesAsync();
    }
}
```

> [!NOTE]
> `StartStream` se usa cuando el stream para ese ID **no existe aún** (primera vez que creamos a Jhon).
> Para eventos posteriores (bodas, mudanzas), usaremos `Append` en su lugar — lo veremos en la Sección 13.

---

## 4. Rehidratando el Agregado (Get / AggregateStreamAsync)

Aquí viene el momento más satisfactorio: ver cómo Marten replays todos los eventos de Jhon y nos entrega el objeto `Persona` completamente rehidratado — sin que nosotros escribamos ni un solo bucle `foreach`:

```csharp
public class ConsultarPersonaHandler
{
    private readonly IDocumentSession _session;

    public ConsultarPersonaHandler(IDocumentSession session)
    {
        _session = session;
    }

    public async Task<Persona> Handle(Guid personaId)
    {
        // Marten va a PostgreSQL, trae todos los eventos del stream de ese ID,
        // instancia una Persona vacía (new Persona()) y le aplica cada evento
        // llamando a los métodos Apply() que ya definimos. Igual que nuestro Load().
        var persona = await _session.Events.AggregateStreamAsync<Persona>(personaId);

        if (persona is null)
            throw new InvalidOperationException($"Persona {personaId} no encontrada.");

        return persona;
    }
}
```

> [!IMPORTANT]
> Para que `AggregateStreamAsync<Persona>` funcione, la clase `Persona` debe:
> 1. Tener un **constructor vacío** (`public Persona() {}`) o ser instanciable sin parámetros.
> 2. Mantener sus métodos `Apply(TipoDeEvento)` que definimos en la Sección 04.
>
> Marten usa exactamente el mismo mecanismo de `Apply` con `dynamic dispatch` que diseñamos a mano. ¡No fue en vano!

---

## 5. Actualizando la clase Persona

El único ajuste que necesita nuestra `Persona` es añadir el constructor vacío requerido por Marten:

```csharp
public class Persona : AggregateRoot
{
    public string Nombre { get; private set; }
    public string Ciudad { get; private set; }
    public int Edad { get; private set; }
    public List<string> Hijos { get; private set; } = new();

    // Requerido por Marten para instanciar el objeto antes de hacer el Replay
    public Persona() { }

    protected void Apply(PersonaNacida n)   { Id = n.PersonaId; Nombre = n.Nombre; Ciudad = n.Ciudad; }
    protected void Apply(CumpleañosCelebrado e) { Edad++; }
    protected void Apply(HijoNacido h)      { Hijos.Add(h.NombreHijo); }
}
```

> [!TIP]
> Observa que **eliminamos el constructor que recibía `IEnumerable<object> eventos`**. Marten se encarga del Replay internamente — ya no necesitamos inicializarlo a mano desde el `Program.cs`.

---

### El Descubrimiento

Con tres cambios quirúrgicos hemos pasado de un diccionario en RAM a una base de datos empresarial lista para producción:

1. ✅ Instalamos `Marten` vía NuGet.
2. ✅ Registramos Marten en el DI con la ConnectionString (4 líneas).
3. ✅ Reemplazamos `IEventStore` por `IDocumentSession` en los Handlers.

El resto — serialización JSON, transacciones ACID, control de versiones, tablas en PostgreSQL — Marten lo hizo 100% automáticamente en el fondo.

**¿Pero en un sistema real, cómo le llegan los Comandos a los Handlers? ¿Tiene que existir un endpoint HTTP que instancie al Handler manualmente?**

En sistemas de alta escala, los Comandos se envían a través de un **Bus de Mensajes interno** que enruta cada Comando a su Handler correcto de forma desacoplada. Para eso necesitamos al siguiente actor de nuestro workshop: **Wolverine**.

---

[⬅️ Volver a la sección anterior](./11-docker-postgres.md)

[➡️ Siguiente sección: El Correo Interno (Wolverine)](./13-wolverine.md)
