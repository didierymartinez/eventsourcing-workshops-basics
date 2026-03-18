# 07 - Abstracción del Almacén: El Event Store

Nuestro flamante `PersonaCommandHandlers` ya controla el flujo, pero tiene un problema arquitectónico grave: ¡Conoce exactamente cómo y dónde se guardan los datos! (Inyectamos una ruda `List<object>` en su constructor).

Si decidimos cambiar la lista por una base de datos real o un archivo de texto, tendríamos que reescribir nuestro orquestador. Las dependencias están al revés.

## 🎯 El Objetivo
El Handler solo debería decirle a "alguien": *"Tráeme la historia de Jhon"* y *"Guarda estas nuevas historias"*. No debería importarle cómo ocurre la magia de I/O de archivos o conexión a base de datos.

Ese *alguien* es el **Event Store**.

---

## 1. El Contrato: Definiendo las funciones
Para desenchufar al Handler de la implementación física de la lista, diseñamos una interfaz que solo promete dos cosas esenciales para cualquier Agregado: **Obtener** y **Guardar**.

```csharp
public interface IPersonaEventStore
{
    // Recupera a Jhon de donde sea que esté guardado
    Persona GetPersona(Guid id);

    // Guarda los nuevos hitos generados
    void SaveEvents(Guid id, object nuevoEvento);
}
```

## 2. El Bibliotecario En Memoria
Ahora que tenemos la "política", escribiremos nuestra primera Base de Datos real, aunque viva puramente en memoria. A diferencia de todo el `Program.cs` usando una sola lista `List<object>`, usaremos un **Diccionario** para poder encontrar eficientemente la biografía de una persona filtrando por su `Guid`.

```csharp
public class InMemoryPersonaEventStore : IPersonaEventStore
{
    // El "Baúl Maestro". Un diccionario que tiene un compartimiento por ID de Persona
    private readonly Dictionary<Guid, List<object>> _storage = new();

    public Persona GetPersona(Guid id)
    {
        // Si no existe, entregamos una página en blanco
        if (!_storage.ContainsKey(id))
            return new Persona(new List<object>()); 

        // Entregamos la persona viva y reconstruida usando sus propios eventos
        return new Persona(_storage[id]);
    }

    public void SaveEvents(Guid id, object nuevoEvento)
    {
        // Si es el primer evento para esta persona, abrimos su compartimiento
        if (!_storage.ContainsKey(id))
            _storage[id] = new List<object>();

        // Agregamos el hecho a sus archivos
        _storage[id].Add(nuevoEvento);
    }
}
```

## 3. Plug & Play: Inyectando el Event Store

Volvamos a nuestro `PersonaCommandHandlers` y cambiemos la lista inyectable por nuestro nuevo contrato `IPersonaEventStore`. Notarás que el código se vuelve absurdamente natural de leer:

```csharp
public class PersonaCommandHandlers
{
    private readonly IPersonaEventStore _store;

    public PersonaCommandHandlers(IPersonaEventStore store)
    {
        _store = store; // Ahora dependemos de una interfaz abstraída
    }

    public void Handle(Guid personaId, string nombrePareja)
    {
        // 1. Cargamos desde la abstracción
        var jhon = _store.GetPersona(personaId);

        // 2. Actuamos pidiendo intenciones
        var eventoBoda = jhon.RegistrarMatrimonio(nombrePareja);

        // 3. Guardamos a través de la abstracción
        _store.SaveEvents(personaId, eventoBoda);
    }
}
```

### El Descubrimiento
Acabas de implementar el patrón **Repository/Store**. Tu código de orquestación ya no sabe nada del Mundo Exterior, bases de datos o diccionarios. Está "aislado y seguro", y al fin puedes manejar eficazmente a múltiples personas porque tu `InMemoryPersonaEventStore` clasifica por ID utilizando un Diccionario bajo el capó.

Pero la arquitectura nunca descansa. ¿Qué pasa si mañana introduces gatos, perros, empresas y carros al dominio? Copiar y pegar `IPersonaEventStore` e `InMemoryPersonaEventStore` de Agregado en Agregado es una atrocidad técnica. Necesitamos ir de lo específico a lo genérico.

---

[⬅️ Volver a la sección anterior](./06-el-command-handler.md)

[➡️ Siguiente sección: El Flujo Universal (EventStream)](./08-el-flujo-de-vida.md)
