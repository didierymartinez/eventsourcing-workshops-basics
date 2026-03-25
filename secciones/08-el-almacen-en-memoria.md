# 08 - El Almacén en Memoria: El Event Store

En la sección anterior logramos que el `EventStream<T>` manejara perfectamente el flujo de vida de un solo individuo aislando su `List<IEvent>` física.

Pero nos dimos cuenta de un problema logístico masivo: si tenemos 1.000 clientes, tendríamos 1.000 listas flotando en la memoria del programa. Necesitamos un archivero general que agrupe y custodie todos esos flujos individuales.

**El Event Store es el sistema de almacenamiento centralizado que contiene todos los EventStreams.**

---

## 🎯 El Objetivo

Necesitamos un **contrato formal** que defina cómo se guardan y recuperan los eventos de cualquier Agregado en ese archivero central. Los EventStreams le hablarán a este contrato, delegando la responsabilidad de *dónde* y *cómo* se guardan físicamente las listas.

---

## 1. Añadiendo Control de Concurrencia al Sobre

En la sección anterior creamos el sobre `EventoAlmacenado` para guardar los eventos puros. Pero el Archivero es estricto: no puede simplemente meter sobres al cajón sin orden. 

Necesitamos añadirle un metadato vital a nuestro sobre: la **Versión**.

```csharp
// Actualizamos el "sobre" que creamos en la sección anterior
public record EventoAlmacenado(
    Guid     AggregateId,  // ¿A qué Agregado pertenece?
    int      Version,      // ¿En qué versión exacta va dentro de ese stream?
    DateTime Timestamp,    // ¿Cuándo se archivó?
    object   EventData     // El evento puro ("PersonaNacida", etc.)
);
```

> [!NOTE]
> La `Version` es crucial: garantiza que, aunque miles de eventos lleguen al almacén al mismo tiempo, siempre conserven su estricto orden cronológico dentro de su propia biografía. En la industria, a esto se le conoce como **Control de Concurrencia Optimista**.

---

## 2. El Contrato: El IEventStore

El `IEventStore` es la recepcionista del archivo. No sabe nada de `Persona` o `Mascota`, solo entiende de `Guid` (dame el cajón de este ID) y de cajitas `EventoAlmacenado`.

```csharp
public interface IEventStore
{
    // Dame todos los eventos archivados de esta persona
    IEnumerable<EventoAlmacenado> GetEvents(Guid aggregateId);

    // Archiva un nuevo evento al final del cajón de esta persona
    void AppendEvent(EventoAlmacenado evento);
}
```

---

## 3. La Implementación: El Diccionario en Memoria

Aquí es donde entra la estructura de datos para agrupar las listas que nos faltaba. La primera implementación concreta de nuestro almacén vive solo en RAM (perfecta para desarrollo). 

Internamente es un `Dictionary`, donde la llave es el `Guid` de la persona, y el valor es su flujo ordenado de `EventoAlmacenado`:

```csharp
public class InMemoryEventStore : IEventStore
{
    // EL ARCHIVERO REAL: Cada Guid tiene su propio compartimiento de eventos
    private readonly Dictionary<Guid, List<EventoAlmacenado>> _storage = new();

    public IEnumerable<EventoAlmacenado> GetEvents(Guid aggregateId)
    {
        if (!_storage.ContainsKey(aggregateId))
            return Enumerable.Empty<EventoAlmacenado>();

        return _storage[aggregateId];
    }

    public void AppendEvent(EventoAlmacenado evento)
    {
        // Si el cajón no existe, lo creamos
        if (!_storage.ContainsKey(evento.AggregateId))
            _storage[evento.AggregateId] = new List<EventoAlmacenado>();

        // Anexamos el evento al cajón
        _storage[evento.AggregateId].Add(evento);
    }
}
```

---

## 4. Actualizando el EventStream

Ahora que el EventStore existe, modificamos el `EventStream<T>` de la sección anterior. Ya no guarda una `List<IEvent>` interna; ahora se conecta al `IEventStore` genérico para que él le traiga los eventos o se los guarde.

Además, nuestro stream calculará la versión / secuencia del objeto:

```csharp
public class EventStream<T> where T : AggregateRoot, new()
{
    private readonly IEventStore _store;
    private readonly Guid _aggregateId;
    
    // El stream ahora lleva la cuenta de en qué página del historial vamos
    private int _version = 0;

    public EventStream(IEventStore store, Guid aggregateId)
    {
        _store       = store;
        _aggregateId = aggregateId;
    }

    // Ya no usa la lista interna, pide los eventos al almacén central
    public T Get()
    {
        var entidad = new T();
        
        var eventos = _store.GetEvents(_aggregateId);

        foreach (var ev in eventos)
        {
            entidad.Load(new[] { ev.EventData });
            _version = ev.Version; // Nos actualizamos a la última versión leída
        }
        return entidad; // A este proceso de reconstruir el estado se le llama "Rehidratar" (Rehydrate)
    }

    // Empaqueta el evento de negocio y lo envía al almacén
    public void Append(object nuevoEvento)
    {
        _version++; // Nueva página/versión en la biografía

        var eventoAlmacenado = new EventoAlmacenado(
            AggregateId: _aggregateId,
            Version: _version,
            Timestamp: DateTime.UtcNow,
            EventData: nuevoEvento
        );

        _store.AppendEvent(eventoAlmacenado);
    }
}
```

---

## 5. Todo Junto: El Store administra los Streams

Ahora el programa arquitectónico está completo y escalable para cualquier número de personas sin ensuciar la memoria con listas sueltas:

```csharp
// Un solo Event Store (Diccionario) para TODA la aplicación
IEventStore store = new InMemoryEventStore();

// Abrimos una ventana (Stream) enfocada solo a Jhon
var flujoJhon = new EventStream<Persona>(store, idJhon);

// Abrimos una ventana (Stream) enfocada solo a Ana
var flujoAna  = new EventStream<Persona>(store, idAna);

// Registramos la vida de Jhon
var jhon = flujoJhon.Get();
var eventoBoda = jhon.RegistrarMatrimonio("María"); // Retorna el record puro
flujoJhon.Append(eventoBoda);

// Registramos la vida de Ana (el Store garantiza que no se mezclen los IDs)
var ana = flujoAna.Get();
var eventoMudanza = ana.RegistrarMudanza("Medellín");
flujoAna.Append(eventoMudanza);

// VERIFICAR MÁGIA: reconstruimos a Jhon desde el Store
var jhonFinal = flujoJhon.Get();
Console.WriteLine($"{jhonFinal.Nombre} está casado con {jhonFinal.NombrePareja}");
```

---

### El Descubrimiento

La arquitectura refleja por fin la verdad del patrón **Event Sourcing**:

```
IEventStore  (Storage centralizado que agrupa a todos)
    ├── EventStream<Persona> (idJhon) → Solo ve los eventos de Jhon
    ├── EventStream<Persona> (idAna)  → Solo ve los eventos de Ana
    └── EventStream<Mascota> (idRex)  → Solo ve los eventos de Rex
```

Pero todavía hay una fragilidad enorme de la que tenemos que hacernos cargo: si el servidor se reinicia, el `InMemoryEventStore` pierde todo su glorioso diccionario. 

En la siguiente sección exploramos las consecuencias de esa volatilidad temporal y las reglas necesarias para solucionarla (Input/Output).

---

[⬅️ Volver a la sección anterior](./07-el-flujo-de-vida.md)

[➡️ Siguiente sección: El Riesgo de Olvidar](./09-el-riesgo-de-olvidar.md)
