# 09 - El Tiempo de Espera: I/O, `async/await` y el riesgo de olvidar

## ⏱️ Fase 2: Transición a la Infraestructura .NET

Hasta este punto de la historia, nuestra Agencia de Biógrafos es un milagro de eficiencia. El Biógrafo (`CommandHandler`) toma los apuntes, va corriendo al `EventStream`, pide los archivos a nuestro `InMemoryEventStore`, y escribe los nuevos eventos en milisegundos.

Pero nuestra oficina mágica tiene un problema colosal: **cierra a las 6:00 PM (cuando la aplicación de consola se apaga), y el conserje bota todos los papeles a la basura**.

Todo nuestro sistema vive en la Memoria RAM. Es **volátil**. Para que las biografías sobrevivan al paso del tiempo (y a la caída de servidores), necesitamos enviar los papeles fuera del edificio: a una base de datos real. 

Pero abandonar la RAM tiene un costo. **Aparece el Tiempo de Espera (I/O).**

---

## 🚀 El Problema: CPU vs I/O

En programación existen dos tipos principales de operaciones:

1. **CPU Bound (Ligadas a Procesamiento):** Como sumar 1+1, o el método `Apply()` que vimos antes. El cerebro del computador (la CPU) tiene las variables ahí mismo en la memoria caché. Ocurren en nanosegundos (10⁻⁹).
2. **I/O Bound (Ligadas a Entrada/Salida):** Escribir en un disco duro físico, llamar a una API web, o enviar un comando a PostgreSQL en otro servidor. Toma milisegundos (10⁻³). 

> [!WARNING]
> La diferencia parece poca, pero para la CPU, un milisegundo es **una eternidad**. Si le decimos al procesador: *"Oye, ve y guarda estos eventos en la base de datos"* y lo obligamos a quedarse **congelado esperando** a que el disco duro termine de girar para responder *"OK, guardado"*, estamos desperdiciando recursos enormes. Toda la aplicación web colapsaría si llegan 100 usuarios al mismo tiempo.

---

## 🌊 La Solución de C#: `Task` y `async`/`await`

Para no congelar la oficina mientras el biógrafo va en bus hasta la base de datos, C# introdujo la programación asíncrona.

*   `async`: Le dice a C# "esta función contiene esperas largas hacia el mundo exterior".
*   `await`: Significa "ve a hacer el viaje, yo me descongelo para atender a otro cliente mientras tanto, avísame cuando vuelvas y seguimos justo aquí".
*   `Task`: Es la promesa de "te entregaré un resultado en el futuro".

En la vida real de Event Sourcing (y en Cosmos), **todo acceso al Event Store es asíncrono**. Absolutamente todo.

---

## 🛠️ Modificando nuestro `IEventStore` al mundo real

Nuestro viejo contrato era instantáneo (sincrónico):
```csharp
// ❌ Sincrónico (Bloqueante)
public interface IEventStore
{
    IEnumerable<EventoAlmacenado> GetEvents(Guid aggregateId);
    void AppendEvent(EventoAlmacenado evento);
}
```

Ahora evolucionamos el contrato envolviendo los retornos en una "Promesa" (`Task`):

```csharp
// ✅ Asincrónico (No Bloqueante)
using System.Threading.Tasks;

public interface IEventStore
{
    // Retorna una "Promesa" de entregar la lista en el futuro
    Task<IEnumerable<EventoAlmacenado>> GetEventsAsync(Guid aggregateId);

    // Retorna una "Promesa" de que la tarea terminará en el futuro
    Task AppendEventAsync(EventoAlmacenado evento);
}

// Implementaremos este nuevo contrato en PostgreSQL más adelante.
```

## 🛠️ Evolucionando el EventStream

Si el EventStore asincrónico es el archivero, el `EventStream` que habla con él también debe ser asíncrono. Observa cómo fluyen los `async/await` como una cascada hacia arriba:

```csharp
public class EventStream<T> where T : AggregateRoot, new()
{
    private readonly IEventStore _store;
    private readonly Guid _aggregateId;
    private int _version = 0;

    public EventStream(IEventStore store, Guid aggregateId) { ... }

    // Ya no es 'Get()', ahora devuelve un Task<T>
    public async Task<T> GetAsync()
    {
        var entidad = new T();
        
        // El 'await' libera la CPU mientras el disco responde
        var eventos = await _store.GetEventsAsync(_aggregateId);

        foreach (var ev in eventos)
        {
            entidad.Load(new[] { ev.EventData });
            _version = ev.Version;
        }
        return entidad; // (Rehidratación Asíncrona)
    }

    public async Task AppendAsync(object nuevoEvento)
    {
        _version++;
        var eventoAlmacenado = new EventoAlmacenado(
            AggregateId: _aggregateId,
            Version: _version,
            Timestamp: DateTime.UtcNow,
            EventData: nuevoEvento
        );

        // De nuevo, esperamos sin bloquear el hilo
        await _store.AppendEventAsync(eventoAlmacenado); 
    }
}
```

## 🛠️ El Command Handler Asíncrono

Finalmente, el controlador maestro (Handler) también tiene que adaptarse al nuevo paradigma de la agencia:

```csharp
public class MatrimonioSolicitadoHandler
{
    private readonly IEventStore _store;

    public MatrimonioSolicitadoHandler(IEventStore store)
    {
        _store = store;
    }

    public async Task HandleAsync(RegistrarMatrimonioCommand comando)
    {
        var stream = new EventStream<Persona>(_store, comando.PersonaId);
        
        // 1. CARGAR (Esperar I/O)
        var persona = await stream.GetAsync();

        // 2. ACTUAR (Reglas de CPU instantáneas)
        var nuevoEvento = persona.RegistrarMatrimonio(comando.NombrePareja);

        // 3. GUARDAR (Esperar I/O)
        await stream.AppendAsync(nuevoEvento);
    }
}
```

### El Descubrimiento Crítico

La arquitectura real *nunca* asume que la base de datos es instantánea. En .NET, la persistencia se separa lógicamente del procesamiento usando `.Wait()`, `Task` o `async`.

Ahora nuestro motor está preparado para el mundo real. Las interfaces ya no nos mienten. Pero, ¿quién implementará ese `Task<IEnumerable<EventoAlmacenado>> GetEventsAsync`? Para eso, en la sección 11 dejaremos atrás la tabla de madera y construiremos una **Bóveda Incombustible** impulsada por tecnología real. 

Pero antes, tenemos que despedir a los intermediarios que arman estas clases manualmente con `new()`. Conozcamos al recepcionista de C#: **El Service Provider**.

---

[⬅️ Volver a la sección anterior](./08-el-almacen-en-memoria.md)

[➡️ Siguiente sección: El Recepcionista (Inyección de Dependencias)](./10-inyeccion-de-dependencias.md)
