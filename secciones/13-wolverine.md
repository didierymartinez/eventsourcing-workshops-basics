# 13 - El Correo Interno: Wolverine y el Bus de Mensajes

En la Sección 12 conectamos Marten y logramos que los eventos de Jhon se persistan realmente en PostgreSQL. 

Pero si observas el `Command Handler` que escribimos, hay un detalle que se vuelve problemático a escala:

```csharp
// Alguien tiene que instanciar este handler manualmente
var handler = new RegistrarPersonaHandler(session);
await handler.Handle(new RegistrarPersona("Jhon", "Bogotá"));
```

¿Quién llama a este Handler? ¿Un endpoint HTTP? ¿Otro servicio? ¿Qué pasa si hay 50 tipos de Comandos diferentes? ¿Creamos manualmente 50 instancias de Handlers?

Aquí es donde entra **Wolverine**: el sistema de mensajería interno de Sincosoft Cosmos.

---

## 🐺 ¿Qué es Wolverine?

Wolverine es un framework de .NET para el **enrutamiento de mensajes** (Comandos y Eventos) dentro de una aplicación. Se integra nativamente con Marten y funciona como el **cartero interno** del sistema:

- Tú publicas un mensaje (`IMessageBus.SendAsync(comando)`).
- Wolverine sabe automáticamente qué Handler debe procesar ese Comando.
- Wolverine inyecta las dependencias necesarias (como `IDocumentSession`) sin que tú lo hagas a mano.

> [!NOTE]
> **Wolverine vs MediatR**
> Si conoces MediatR, Wolverine cumple un rol similar pero con superpoderes adicionales: se integra con Marten para garantizar la **atomicidad transaccional** (ambos comparten la misma transacción de base de datos), e incorpora el patrón **Transactional Outbox** nativo — lo cual veremos en la Sección 14.

---

## 🗺️ El Mapa de la Traducción

| Sin Wolverine | Con Wolverine |
|---|---|
| Instanciar Handlers manualmente | Wolverine los descubre y registra automáticamente |
| Inyectar dependencias a mano | El DI Container de Wolverine lo hace |
| Coordinar la transacción tú mismo | Wolverine + Marten comparten una sola transacción |
| Acoplar emisor y receptor directamente | El emisor no sabe quién procesa el mensaje |

---

## 1. Instalación

```bash
dotnet add package WolverineFx
dotnet add package WolverineFx.Marten
```

El paquete `WolverineFx.Marten` es el que activa la integración entre ambos para compartir transacciones.

---

## 2. Configuración en el Program.cs

```csharp
using Wolverine;
using Wolverine.Marten;

var builder = WebApplication.CreateBuilder(args);

var connectionString = "Host=localhost;Port=5432;Database=eventstore_db;Username=admin;Password=Password123!";

builder.Services.AddMarten(options =>
{
    options.Connection(connectionString);
    options.AutoCreateSchemaObjects = AutoCreate.All;

    options.Events.AddEventType<PersonaNacida>();
    options.Events.AddEventType<CumpleañosCelebrado>();
    options.Events.AddEventType<HijoNacido>();
})
// Esta línea es la magia: le dice a Marten que comparta su sesión con Wolverine
.IntegrateWithWolverine();

builder.Host.UseWolverine(opts =>
{
    // Wolverine descubre automáticamente todos los Handlers del proyecto
    opts.Discovery.IncludeAssembly(typeof(Program).Assembly);
});
```

---

## 3. Definiendo un Comando

En Wolverine, un **Comando** es cualquier clase C#. No necesita implementar una interfaz especial. Por convención los nombramos como verbos en imperativo:

```csharp
// Un comando es simplemente un record con los datos necesarios para ejecutar la acción
public record RegistrarPersona(string Nombre, string Ciudad);
```

> [!TIP]
> En DDD, la distinción entre un **Comando** y un **Evento de Dominio** es fundamental:
> - **Comando** → Una **intención** de que algo ocurra. Puede ser rechazado (ej: "Registrar Persona con nombre vacío" falla la validación).
> - **Evento de Dominio** → Un **hecho** que ya ocurrió. Es inmutable e irrechazable (ej: `PersonaNacida` ya pasó).

---

## 4. El Handler al estilo Wolverine

Wolverine descubre los Handlers por **convención de nombres**: cualquier clase cuyo método se llame `Handle` y reciba un Comando como parámetro:

```csharp
public class RegistrarPersonaHandler
{
    // Wolverine inyecta IDocumentSession automáticamente
    // y garantiza que SaveChangesAsync() se llame al final del Handler
    public async Task Handle(RegistrarPersona comando, IDocumentSession session)
    {
        var idPersona = Guid.NewGuid();

        var evento = new PersonaNacida(idPersona, comando.Nombre, DateTime.UtcNow, comando.Ciudad);

        session.Events.StartStream<Persona>(idPersona, evento);

        // No necesitas llamar SaveChangesAsync() aquí.
        // Wolverine + Marten lo hacen automáticamente al finalizar el Handler.
    }
}
```

> [!IMPORTANT]
> Observa que **no llamamos `SaveChangesAsync()` explícitamente**. La integración Wolverine + Marten garantiza que, si el Handler termina sin lanzar una excepción, la sesión se confirma automáticamente en una sola transacción.
>
> Si el Handler lanza una excepción, Marten hace **rollback** completo: ningún evento llega a la base de datos.

---

## 5. Publicando el Comando

Para enviar un Comando, inyectamos `IMessageBus` (el cartero de Wolverine) en lugar de llamar al Handler directamente:

```csharp
app.MapPost("/personas", async (RegistrarPersona comando, IMessageBus bus) =>
{
    // El emisor no sabe qué Handler existe. Solo le dice al bus "ejecuta esto".
    await bus.InvokeAsync(comando);
    return Results.Ok();
});
```

> [!NOTE]
> `InvokeAsync` es la forma síncrona-over-async de Wolverine: espera que el Handler termine antes de continuar. También existe `SendAsync` para despacho en background (fuego y olvido).

---

### El Descubrimiento

Con Wolverine hemos eliminado el acoplamiento directo entre el **que pide** (el endpoint HTTP) y el **que ejecuta** (el Handler):

```
HTTP POST /personas
    └──> IMessageBus.InvokeAsync(comando)
              └──> [Wolverine descubre automáticamente]
                        └──> RegistrarPersonaHandler.Handle(comando, session)
                                  └──> Marten persiste en PostgreSQL (transacción compartida)
```

El endpoint no sabe nada de Marten, el Handler no sabe nada del endpoint. Son piezas independientes conectadas solo por el contrato del Comando.

**¿Pero qué pasa si el proceso falla justo después de guardar el evento en la BD, antes de notificar a otros sistemas?** Ese es el problema del "trabajo a medias", y tiene un nombre en la industria: **El Riesgo del Mensajero Muerto**. Para blindarse contra él, Wolverine incluye el patrón **Transactional Outbox** — lo veremos en la próxima sección.

---

[⬅️ Volver a la sección anterior](./12-introduccion-a-marten.md)

[➡️ Siguiente sección: El Compromiso Inquebrantable (Transactional Outbox)](./14-outbox.md)
