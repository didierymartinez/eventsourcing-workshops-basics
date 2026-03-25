# 05 - El flujo de vida: El EventStream

En la sección anterior, logramos construir un **motor de rehidratación** elegante dentro de la clase base `AggregateRoot`. 

Sin embargo, para "despertar" a Jhon, todavía dependemos de pasarle una lista cruda desde nuestro programa principal:

```csharp
var listaCrudaDeJhon = new List<object> { new PersonaNacida(...) };
var jhon = new Persona(listaCrudaDeJhon);
```

Manipular listas crudas (`List<object>`) directamente tiene graves problemas arquitectónicos:
1. **Falta de Responsabilidad (Acoplamiento)**: Quien quiera consultar a Jhon tiene que instanciar manualmente el objeto `new Persona(...)`. Saber *cómo* rehidratar un objeto no debería ser responsabilidad del programa consumidor.
2. **Basura Lógica**: Al ser una lista cruda desprotegida, cualquiera podría accidentalmente hacer `listaCrudaDeJhon.Add("Hola Mundo")` sin darnos cuenta.

Necesitamos un envoltorio técnico (un Patrón Repositorio) que proteja esa lista y centralice la lógica para interactuar con ella.

---

## 🌊 La Solución: El "EventStream"

El **EventStream** es el envoltorio lógico para **una sola línea temporal** (la historia exclusiva de UNA entidad).

### 🛠️ Paso 1: El Sobre (El Envoltorio de Metadatos)

Antes de crear el Stream, vamos a solucionar el problema más grave: que cualquiera pueda insertar cualquier cosa, y peor aún, que no sabemos cuándo pasó ni a quién le pertenece.

En lugar de obligar a nuestros hermosos Eventos de Dominio (`PersonaNacida`, `MatrimonioRegistrado`) a heredar de interfaces raras (lo cual contamina el dominio), **vamos a crear un sobre**.

Cuando un Evento de Dominio quiere entrar a la historia, lo metemos dentro de un sobre oficial llamado `EventoAlmacenado`:

```csharp
// El "sobre" oficial que envuelve cualquier evento crudo
public record EventoAlmacenado(
    Guid     AggregateId,  // ¿A quién le pasó?
    DateTime Timestamp,    // ¿Cuándo pasó?
    object   EventData     // El Evento de Dominio (PersonaNacida, etc.)
);
```

A partir de ahora, abandonamos la `List<object>` cruda para siempre. La colección interna de una entidad será estrictamente una `List<EventoAlmacenado>`.

---

### 🛠️ Paso 2: Encapsulando la lista en el EventStream

Ahora construimos la clase envoltorio genérica. Observa cómo "esconde" la lista y expone, por ahora, su capacidad de lectura a través de un método limpio de alto nivel:

> [!NOTE]
> **¿Qué es el `<T>` en C#?**
> El `<T>` (de *Type*) le permite a esta clase trabajar con cualquier Agregado. 
> La restricción `where T : AggregateRoot, new()` garantiza que el tipo que metamos aquí sí o sí tenga el método `Load` y pueda instanciarse vacío (`new()`).

```csharp
using System.Linq;

public class EventStream<T> where T : AggregateRoot, new()
{
    // La lista base ahora es estrictamente de Sobres Oficiales.
    // Ocultaremos esta lista al mundo exterior para proteger su integridad.
    private readonly List<EventoAlmacenado> _eventosInternos;
    
    // Guardamos el ID específico de la entidad (ej: el idJhon).
    // Lo necesitamos para "sellar" cada nuevo sobre de memoria que crearemos en el futuro,
    // garantizando que, si esta lista se guarda en una base de datos global, 
    // nunca se nos mezclen los eventos de Jhon con los eventos de Ana.
    private readonly Guid _aggregateId;

    public EventStream(List<EventoAlmacenado> listaBase, Guid aggregateId)
    {
        _eventosInternos = listaBase;
        _aggregateId = aggregateId;
    }

    // RESPONSABILIDAD 1: Entregar el objeto rehidratado
    public T Get()
    {
        var entidad = new T(); // Instanciamos cualquier T en blanco
        
        // Extraemos solo el contenido (EventData) de cada sobre
        var domainEvents = _eventosInternos.Select(e => e.EventData);
        
        // Gracias a la restricción "where T : AggregateRoot" en la definición de la clase,
        // el compilador sabe con certeza que 'entidad' (ya sea Persona o Mascota) 
        // obligatoriamente posee el motor base y podemos llamar al método .Load() 
        // para rehidratarlo de una sentada.
        entidad.Load(domainEvents); 
        return entidad;
    }
}
```

Así se usa en la vida real:

```csharp
// 1. La lista base de sobres sigue existiendo (en Memoria por ahora)
var listaBaseJhon = new List<EventoAlmacenado>();

// 2. Envolvemos la lista en nuestro Stream.
// ¿La razón? El Stream actuará como nuestro "embudo oficial" o "guardaespaldas".
// En lugar de hacer .Add() directos, sueltos y peligrosos a la lista plana, 
// a partir de ahora, todo paso de lectura o escritura tendrá que pasar a través
// de la estricta regla del método Get del Stream.
var flujoJhon = new EventStream<Persona>(listaBaseJhon, idJhon);

// 3. El uso es maravillosamente limpio y seguro
// El stream se encarga de instanciar la Persona y rehidratarla.
// Ya no tenemos que instanciar variables puras sueltas.
var jhon = flujoJhon.Get(); 

Console.WriteLine($"Jhon vive en {jhon.Ciudad}");
```

---

### El Descubrimiento

El `EventStream<T>` es el guardián de **una línea temporal individual**. Centralizó la instanciación de entidades (`new T()`), limpió nuestros objetos de no heredar jerarquías forzadas, y nos dio un método limpio y controlado: `Get`. Todo el proceso de hidratación y lectura está completamente dominado.

**¿Pero adivina qué? Todavía tenemos un cuello de botella logístico.**

Si a la agencia llegan 1.000 clientes, tendríamos que instanciar 1.000 colecciones internas (`new List<EventoAlmacenado>()`) flotando sueltas en el programa para poder inyectarlas en 1.000 `EventStream`s.

¿Cómo archivamos, organizamos y agrupamos eficientemente miles de líneas de vida diferentes sin tener variables globales sueltas?

Para eso necesitamos el **Event Store**.

---

[⬅️ Volver a la sección anterior](./04-refactorizando-el-motor.md)

[➡️ Siguiente sección: El Almacén en Memoria (Event Store)](./06-el-almacen-en-memoria.md)
