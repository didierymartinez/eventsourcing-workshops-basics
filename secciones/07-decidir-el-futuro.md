# 07 - Decidir el futuro: Emitir eventos

Jhon ya tiene una biografía y sabe quién es. Pero la vida sigue, y queremos agregar nuevos capítulos a su historia. Hasta ahora solo hemos visto cómo rehidratar el pasado, pero ¿cómo agregamos nuevos hitos en el presente?

## 🎯 El Objetivo
Jhon sigue viviendo y experimentando nuevos hitos: bodas, mudanzas, nuevos trabajos. Nuestro objetivo es aprender cómo registrar estos nuevos sucesos a través de nuestro objeto `Persona`.

---

## 1. El Nuevo Hito
En Event Sourcing, todo lo que altera la historia se registra como un evento adicional. Vamos a definir un par de nuevos eventos para la vida de Jhon:

```csharp
public record PersonaCasada(Guid PersonaId, string NombrePareja);
public record PersonaMudada(Guid PersonaId, string NuevaCiudad);
```

## 2. Es quien autoriza agregar eventos a su biografía
Aquí hay un detalle clave (y muy sutil) en Event Sourcing: La clase `Persona` es la dueña de las **reglas** de su historia, pero sorprendentemente **NO almacena la lista físicamente**. Si te fijas, su constructor lee los eventos del pasado para rehidratarse, pero nunca guarda una referencia directa a la `Lista<object> biografia`.

Por lo tanto, a Jhon no le "insertamos" eventos a la fuerza; nosotros le pedimos que evalúe realizar una acción y él, a cambio, **emite un nuevo evento** como resultado. Alguien más en el sistema será el encargado de tomar ese evento y guardarlo en el baúl para el futuro.

Por ahora, como estamos en el "camino feliz", asumimos que Jhon acepta generar cualquier evento nuevo sin poner condiciones. Añadamos esta lógica a la clase `Persona`:

```csharp
public class Persona : AggregateRoot
{
    public string NombrePareja { get; private set; }
    // Asumimos que la propiedad Ciudad ya existe desde la refactorización anterior

    // ... propiedades, constructor y métodos Apply básicos ...

    public PersonaCasada RegistrarMatrimonio(string nombrePareja)
    {
        // Generamos un nuevo hito para la biografía
        return new PersonaCasada(this.Id, nombrePareja);
    }

    public PersonaMudada RegistrarMudanza(string nuevaCiudad)
    {
        // En una aplicación real, aquí validaríamos las reglas de negocio usando el estado rehidratado
        return new PersonaMudada(this.Id, nuevaCiudad);
    }

    // El motor actualiza el estado cuando ESTOS eventos ocurren en el pasado
    private void Apply(PersonaCasada c) => NombrePareja = c.NombrePareja;
    private void Apply(PersonaMudada m) => Ciudad = m.NuevaCiudad;
}
```

## 3. Enseñando al Stream a escribir

En las secciones anteriores, diseñamos nuestro `EventStream<T>` exclusivamente como un experto lector para rehidratar el pasado. Pero ahora que nuestra `Persona` genera nuevos Eventos de Dominio en el presente, necesitamos que el Stream tenga la capacidad inversa: **escribir y guardar**.

Añadamos por primera vez el método `Append` a nuestro `EventStream<T>`. Este método recibirá un Evento de Dominio "puro", lo empaquetará en su respectivo sobre (`EventoAlmacenado`) protegiendo el orden de la historia (`_version++`), y se lo entregará firmemente al vigilante (`IEventStore`):

```csharp
public class EventStream<T> where T : AggregateRoot, new()
{
    // ... código de lectura previo (Get) y variables (_store, _aggregateId, _version) ...

    public void Append(object domainEvent)
    {
        _version++; // Nueva página/versión en la biografía

        var eventoAlmacenado = new EventoAlmacenado(
            AggregateId: _aggregateId,
            Version: _version,
            Timestamp: DateTime.UtcNow,
            EventData: domainEvent
        );

        _store.AppendEvent(eventoAlmacenado);
    }
}
```

## 4. 🛠️ El Ciclo de Vida en el Program.cs
Vamos a integrar finalmente esta capacidad bidireccional de lectura y escritura:

```csharp
// 0. Instanciamos el almacén (asumimos que 'store' ya existe)
var streamJhon = new EventStream<Persona>(store, idJhon);

// 1. CARGAMOS: Traemos a Jhon a la vida desde el almacén central
var jhon = streamJhon.Get();
Console.WriteLine($"[ANTES] Jhon vive en {jhon.Ciudad}");

// 2. ACTUAMOS: Le pedimos a Jhon que registre un nuevo hito en su vida
var eventoMudanza = jhon.RegistrarMudanza("Nueva York");

// 3. GUARDAMOS: Enviamos el evento de vuelta al stream para que se guarde
streamJhon.Append(eventoMudanza);

// Para ver el resultado final, rehidratamos a Jhon DESDE CERO 
var streamDeVerificacion = new EventStream<Persona>(store, idJhon);
var jhonActualizado = streamDeVerificacion.Get();

Console.WriteLine($"[DESPUÉS] Jhon vive ahora en {jhonActualizado.Ciudad}.");
```

### El Descubrimiento
Acabas de ver el flujo básico para interactuar con el dominio:
1. **Cargar**: Recuperas el pasado (la biografía).
2. **Rehidratar**: Pones al objeto en su estado actual (haciendo el Replay de su historia).
3. **Actuar**: Le pides al objeto que realice una acción y genere un **nuevo evento**.

> [!TIP]
> Intuitivamente, cada acción que le pides a Jhon (como llamar a `RegistrarMatrimonio`) es una petición que le haces al sistema. En diseño de software, a esta intención de hacer algo se le llama **Comando**.
> Aquí vemos una regla de oro: **Los Comandos son los encargados de generar los Eventos** (siempre a través del Aggregate Root).

---

[⬅️ Volver a la sección anterior](./06-el-almacen-en-memoria.md)

[➡️ Siguiente sección: El Command Handler](./08-el-command-handler.md)
