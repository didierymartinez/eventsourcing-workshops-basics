# 07 - El flujo de vida: El EventStream

En la sección anterior vimos que el `Command Handler` maneja el flujo:

**Cargar → Actuar → Guardar**

Pero aún hay un problema práctico: ¿cómo garantizamos la integridad de esa historia? 

## 🚀 El Problema: Una lista expuesta y sin reglas

En la sección anterior, nuestro `PersonaCommandHandler` recibía una `List<object>` cruda desde el exterior:

```csharp
var eventosJhon = new List<object>();
var handlerJhon = new MatrimonioSolicitadoHandler(eventosJhon);
```

Pasar una lista cruda (`List<object>`) directamente a las clases de negocio tiene dos graves problemas:
1. **Falta de Tipado**: Al ser `object`, cualquiera podría hacer `eventosJhon.Add("Hola Mundo")` o `eventosJhon.Add(42)`. La historia se corrompería con cosas que ni siquiera son eventos.
2. **Falta de Responsabilidad (Acoplamiento excesivo)**: El Command Handler tiene que recibir la lista y saber que para despertar a Jhon, debe hacer `var persona = new Persona()` e invocar el método `persona.Load(eventos)`. El Handler sabe "demasiado" sobre el ciclo de vida y cómo se reconstruye la memoria de un Agregado. Su único trabajo debería ser entregarle el comando.

Necesitamos un envoltorio técnico (un Patrón Repositorio) que proteja esa lista cruda y centralice las reglas para interactuar con ella.

---

## 🌊 La Solución: El "EventStream"

El **EventStream** es simplemente el envoltorio lógico para **una sola línea temporal** (la historia exclusiva de UNA entidad).

### 🛠️ Paso 1: El Sobre (El Envoltorio de Metadatos)

Antes de crear el Stream, vamos a solucionar el problema más grave: que cualquiera pueda insertar cualquier cosa, y peor aún, que no sabemos cuándo pasó ni a quién le pertenece.

En lugar de obligar a nuestros hermosos y puros eventos de dominio (`PersonaNacida`, `MatrimonioRegistrado`) a heredar de interfaces raras (lo cual contamina el dominio), **vamos a crear un sobre**.

Cuando un evento puro quiere entrar a la historia, lo metemos dentro de un sobre oficial llamado `EventoAlmacenado`:

```csharp
// El "sobre" oficial que envuelve cualquier evento crudo
public record EventoAlmacenado(
    Guid     AggregateId,  // ¿A quién le pasó?
    DateTime Timestamp,    // ¿Cuándo pasó?
    object   EventData     // El evento puro y duro (PersonaNacida, etc.)
);
```

A partir de ahora, abandonamos la `List<object>` cruda para siempre. La historia física de una entidad será estrictamente una `List<EventoAlmacenado>`.

---

### 🛠️ Paso 2: Encapsulando la lista en el EventStream

Ahora construimos la clase envoltorio genérica. Observa cómo "esconde" la lista y solo expone dos métodos limpios de alto nivel:

> [!NOTE]
> **¿Qué es el `<T>` en C#?**
> El `<T>` (de *Type*) le permite a esta clase trabajar con cualquier Agregado. 
> La restricción `where T : AggregateRoot, new()` garantiza que el tipo que metamos aquí sí o sí tenga el método `Load` y pueda instanciarse vacío (`new()`).

```csharp
using System.Linq;

public class EventStream<T> where T : AggregateRoot, new()
{
    // El stream físico ahora es una lista de Sobres Oficiales
    private readonly List<EventoAlmacenado> _eventosFisicos;
    
    // Necesitamos saber de qué ID específico estamos hablando
    private readonly Guid _aggregateId;

    public EventStream(List<EventoAlmacenado> storageDeEsteId, Guid aggregateId)
    {
        _eventosFisicos = storageDeEsteId;
        _aggregateId = aggregateId;
    }

    // RESPONSABILIDAD 1: Entregar el objeto rehidratado
    public T Get()
    {
        var entidad = new T(); // Instanciamos cualquier T en blanco
        
        // Extraemos solo el contenido (EventData) de cada sobre
        var eventosPuros = _eventosFisicos.Select(e => e.EventData);
        
        entidad.Load(eventosPuros); 
        return entidad;
    }

    // RESPONSABILIDAD 2: Empujar eventos puros a la historia
    public void Append(object nuevoEventoPuro)
    {
        // El Stream se encarga del trabajo sucio: empacar el evento en su sobre oficial
        var sobre = new EventoAlmacenado(
            AggregateId: _aggregateId,
            Timestamp: DateTime.UtcNow,
            EventData: nuevoEventoPuro
        );

        _eventosFisicos.Add(sobre); 
    }
}
```

Así se usa en la vida real:

```csharp
// 1. La lista física de sobres sigue existiendo (en Memoria por ahora)
var listaFisicaJhon = new List<EventoAlmacenado>();

// 2. Envolvemos la lista en nuestro Stream
var flujoJhon = new EventStream<Persona>(listaFisicaJhon, idJhon);

// 3. El uso es limpio, seguro y enfocado al dominio. 
// Le pasamos el objeto puro y el Stream lo empaca internamente.
var jhon = flujoJhon.Get();
var eventoBoda = jhon.RegistrarMatrimonio("María"); // Retorna el record puro

flujoJhon.Append(eventoBoda); 
```

---

### El Descubrimiento

El `EventStream<T>` es el guardián de **una línea temporal individual**. Centralizó la instanciación de entidades (`new T()`), limpió nuestros objetos de no heredar jerarquías forzadas, y aseguró que el Command Handler solo vea dos métodos puros: `Get` y `Append`. Todo lo que entra al Stream sale empacado en un `EventoAlmacenado` listo para la base de datos.

**¿Pero adivina qué? Todavía tenemos un cuello de botella logístico.**

Si a la agencia llegan 1.000 clientes, tendríamos que instanciar 1.000 listas físicas (`new List<EventoAlmacenado>()`) flotando sueltas en el programa para poder inyectarlas en 1.000 `EventStream`s.

¿Cómo archivamos, organizamos y agrupamos eficientemente miles de líneas de vida diferentes sin tener variables globales sueltas?

Para eso necesitamos el **Event Store**.

---

[⬅️ Volver a la sección anterior](./06-el-command-handler.md)

[➡️ Siguiente sección: El Almacén (Event Store)](./08-el-almacen-en-memoria.md)
