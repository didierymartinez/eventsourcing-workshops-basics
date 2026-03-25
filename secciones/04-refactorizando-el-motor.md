# 04 - Refactorizando el motor: El AggregateRoot

En la sección anterior logramos rehidratar la vida de Jhon leyendo su diario directamente en el constructor. Sin embargo, a medida que Jhon viva más hitos (bodas, mudanzas, trabajos), ese constructor se llenará de un `if` interminable.

Vamos a limpiar nuestra arquitectura siguiendo el principio de **"Separación de Responsabilidades"**.

## 🛠️ Refactor 1: Extraer la lógica de "Traducción"

Lo primero es sacar el procesamiento de los eventos de en medio del constructor. Queremos que el constructor solo "orqueste" la lectura, pero que otro método se encargue de saber qué hacer con cada hito.

A este método lo llamaremos **`Aplicar`** (o *Apply* en inglés):

```csharp
public class Persona 
{
    public Guid Id { get; private set; }
    public string Nombre { get; private set; }
    public string Ciudad { get; private set; }
    public int Edad { get; private set; }
    public List<string> Hijos { get; private set; } = new();

    public Persona(IEnumerable<object> eventos)
    {
        foreach (var ev in eventos)
        {
            Aplicar(ev); // Limpito
        }
    }

    private void Aplicar(object ev)
    {
        if (ev is PersonaNacida n) { Id = n.PersonaId; Nombre = n.Nombre; Ciudad = n.Ciudad; }
        if (ev is CumpleañosCelebrado) { Edad++; }
        if (ev is HijoNacido h) { Hijos.Add(h.NombreHijo); }
    }
}
```

## 🏗️ Refactor 2: Generalizando con AggregateRoot

Si mañana llega un nuevo cliente a nuestra agencia para documentar la vida de una `Mascota` o una `Empresa`, tendríamos que copiar y pegar en cada nueva clase la propiedad `Id` y todo el método `Load` con su bucle `foreach` para recorrer eventos.

Aunque la lista siga siendo de tipo `object` (un problema de tipado que resolveremos más adelante en la Sección 07), la mecánica fundamental de **"leer una lista de historia e ir aplicando cada evento uno por uno"** es universal para cualquier entidad en Event Sourcing.

Vamos a crear una **Clase Base Abstracta** que comparta esta "mecánica" (el motor) con todos:

> [!NOTE]
> **¿Por qué una Clase Abstracta y no una Interfaz (`IAggregateRoot`)?**
> 
> En C#, tienes dos opciones principales para estandarizar objetos:
> 1.  **Interfaz (`interface`)**: Solo define el *contrato* (ej: "todos deben tener un método `Load`"). Si usáramos una interfaz, tendríamos que **copiar y pegar** el código del bucle `foreach` y la propiedad `Id` dentro de `Persona`, `Mascota` y `Empresa`.
> 2.  **Clase Abstracta (`abstract class`)**: Define el contrato Y ADEMÁS te permite escribir **código real y reutilizable**. Una clase abstracta es una clase "incompleta" que no puedes instanciar sola (`new AggregateRoot()` da error). 
> 
> Elegimos la **Clase Abstracta** porque el motor de rehidratación (leer la historia línea por línea en el método `Load`) es **idéntico** para todos los Agregados del mundo. Así lo escribimos una sola vez, y cualquier clase que lo herede obtiene el motor gratis.

```csharp
public abstract class AggregateRoot
{
    // El ID se asigna cuando la clase hija procesa su primer evento de creación 
    // (ej: cuando Persona aplica PersonaNacida)
    public Guid Id { get; protected set; }

    protected AggregateRoot() { }

    // El motor genérico de rehidratación (State Rehydration)
    public void Load(IEnumerable<object> eventos)
    {
        foreach (var ev in eventos)
        {
            Aplicar(ev);
        }
    }

    // Cada Agregado sabrá cómo aplicar sus propios eventos
    protected abstract void Aplicar(object ev);
}
```

Ahora Jhon (nuestra `Persona`) es mucho más elegante:

```csharp
public class Persona : AggregateRoot
{
    public string Nombre { get; private set; }
    public string Ciudad { get; private set; }
    public int Edad { get; private set; }
    public List<string> Hijos { get; private set; } = new();

    public Persona(IEnumerable<object> eventos)
    {
        Load(eventos);
    }

    protected override void Aplicar(object ev)
    {
        if (ev is PersonaNacida n) { Id = n.PersonaId; Nombre = n.Nombre; Ciudad = n.Ciudad; }
        if (ev is CumpleañosCelebrado) { Edad++; }
        if (ev is HijoNacido h) { Hijos.Add(h.NombreHijo); }
    }
}
```

## 🚀 Refactor 3: El toque final con Overloads (Elegancia Pro)

Ese `protected override void Aplicar(object ev)` con su `if (ev is ...)` sigue pareciendo rudimentario. Como bien podrías preguntarte: *¿Por qué obligar a cada Agregado a escribir ese método si todos van a hacer lo mismo?*

En arquitecturas maduras, aprovechamos que C# permite **sobrecargar métodos** (múltiples métodos con el mismo nombre) y movemos la magia del ruteo directamente a la clase base `AggregateRoot` usando la palabra clave `dynamic`.

¡Podemos **eliminar** por completo el método abstracto `Aplicar`!

Así queda el motor definitivo en **`AggregateRoot`**:

```csharp
public abstract class AggregateRoot
{
    public Guid Id { get; protected set; }

    protected AggregateRoot() { }

    // El motor definitivo de rehidratación
    public void Load(IEnumerable<object> eventos)
    {
        foreach (var ev in eventos)
        {
            // El motor base hace el enrutamiento mágico por ti
            ((dynamic)this).Apply((dynamic)ev);
        }
    }
}
```

Y así queda de hermosa y limpia nuestra **`Persona`**:

```csharp
public class Persona : AggregateRoot
{
    // ... propiedades ...

    public Persona(IEnumerable<object> eventos) => Load(eventos);

    // Ya no hay `if`. Solo hay sobrecargas limpias.
    // OJO: Deben ser 'protected' o 'internal' para que la clase base (el dynamic) las encuentre, 
    // pero se mantengan ocultas del mundo exterior.
    protected void Apply(PersonaNacida n) { Id = n.PersonaId; Nombre = n.Nombre; Ciudad = n.Ciudad; }
    protected void Apply(CumpleañosCelebrado e) { Edad++; }
    protected void Apply(HijoNacido h) { Hijos.Add(h.NombreHijo); }
}
```

> [!NOTE]
> **¿Cómo funciona la magia de `dynamic`? (Dynamic Dispatching)**
> 
> Normalmente (y por defecto), C# tiene un **tipado estricto** en tiempo de compilación. Eso significa que si le pasas una variable declarada como `object` a un método `Apply`, el compilador siempre buscará ejecutar la firma `Apply(object)`, sin importar si tú sabes que por debajo ese objeto es realmente un `PersonaNacida`.
> 
> Pero al castear a `(dynamic)`, le estamos diciendo a C#: *"Apaga la verificación estricta del compilador y espérate a que el programa esté corriendo. Cuando llegue el momento de ejecutar esta línea, fíjate de qué tipo exacto es la variable `ev` en la memoria (ej: `PersonaNacida`) y busca si existe un método llamado `Apply` que reciba exactamente ese tipo."*
> 
> Así es como en tiempo de ejecución (Runtime), el código enruta automáticamente cada evento a su método `Apply(TipoDeEvento)` adecuado sin necesidad de escribir un enorme bloque `if / else`.

> [!TIP]
> Al separar cada evento en su propio método `Apply(TipoDeEvento)`, hemos eliminado la "complejidad cognitiva" del gran `if`. Si la vida de Jhon crece con 50 eventos nuevos, simplemente añades 50 métodos privados `Apply` aislados y tu clase `Persona` seguirá siendo hermosamente fácil de leer.

---

[⬅️ Volver a la sección anterior](./03-vivir-el-pasado.md)

[➡️ Siguiente sección: Decidir el futuro (Comandos)](./05-decidir-el-futuro.md)
