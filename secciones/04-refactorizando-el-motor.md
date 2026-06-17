# 04 - Refactorizando el motor: El AggregateRoot

> 🎯 **Hacia dónde va:** extraemos el motor de rehidratación a una clase base `AggregateRoot` reutilizable, sentando la arquitectura limpia sobre la que vivirán todos los agregados.

En la sección anterior logramos rehidratar la vida de Jhon leyendo su diario directamente en el constructor. Sin embargo, a medida que Jhon viva más hitos (bodas, mudanzas, trabajos), ese constructor se llenará de un `if` interminable.

Vamos a limpiar nuestra arquitectura siguiendo el principio de **"Separación de Responsabilidades"**.

## 🛠️ Refactor 1: Extraer la lógica de "Traducción"

Ahora sí, refactorizamos el motor:

A este método lo llamaremos **`Aplicar`** (o *Apply* en inglés):

```csharp
public class Persona 
{
    public string Nombre { get; private set; }
    public string Ciudad { get; private set; }
    public int Edad { get; private set; }
    public List<string> Hijos { get; private set; } = new();

    public Persona(IEnumerable<object> eventos)
    {
        foreach (var ev in eventos)
        {
            Aplicar(ev);
        }
    }

    private void Aplicar(object ev)
    {
        if (ev is PersonaNacida n) { Nombre = n.Nombre; Ciudad = n.Ciudad; }
        if (ev is CumpleañosCelebrado) { Edad++; }
        if (ev is HijoNacido h) { Hijos.Add(h.NombreHijo); }
    }
}
```

Si mañana llega un nuevo cliente a nuestra agencia para documentar la vida de una `Mascota` o una `Empresa`, tendríamos que copiar y pegar en cada nueva clase el método `Aplicar` con su bucle `foreach`.

Aunque la lista siga siendo de tipo `object` (un problema de tipado que resolveremos más adelante en la Sección 05), la mecánica fundamental de **"leer una lista de historia e ir aplicando cada evento uno por uno"** es universal para cualquier entidad en Event Sourcing.

> [!NOTE]
> 🌱 **Semilla — Este motor que escribes a mano, luego lo automatiza el framework.** El bucle "cargar historia → aplicar evento por evento → guardar lo nuevo" es tan universal que **Marten + Wolverine lo generan por ti** (el *Aggregate Handler Workflow*: tú solo escribes la decisión y devuelves los eventos; ellos cargan, aplican y guardan con concurrencia optimista). Lo construyes a mano ahora para que ese atajo, más adelante, **no sea magia**: sabrás exactamente qué hace por debajo.

## 🛠️ Refactor 2: Un motor compartido (la Clase Base Abstracta)

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
        if (ev is PersonaNacida n) { Nombre = n.Nombre; Ciudad = n.Ciudad; }
        if (ev is CumpleañosCelebrado) { Edad++; }
        if (ev is HijoNacido h) { Hijos.Add(h.NombreHijo); }
    }
}
```

Ahora Jhon tiene una estructura profesional, y si quisiéramos crear una `Mascota`, solo tendríamos que heredar de `AggregateRoot` y reusar el motor `Load`.

## 🚀 Refactor 3: El toque final con Overloads (Elegancia Pro)

Ese `protected override void Aplicar(object ev)` con su `if (ev is ...)` sigue pareciendo rudimentario. Como bien podrías preguntarte: *¿Por qué obligar a cada Agregado a escribir ese método si todos van a hacer lo mismo?*

En arquitecturas maduras, aprovechamos que C# permite **sobrecargar métodos** (múltiples métodos con el mismo nombre) y movemos la magia del ruteo directamente a la clase base `AggregateRoot` usando la palabra clave `dynamic`.

¡Podemos **eliminar** por completo el método abstracto `Aplicar`!

Así queda el motor definitivo en **`AggregateRoot`**:

```csharp
public abstract class AggregateRoot
{
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
    // ⚠️ public (no protected/private): el motor que las invoca vive en la clase base — ver la nota de abajo.
    public void Apply(PersonaNacida n) { Nombre = n.Nombre; Ciudad = n.Ciudad; }
    public void Apply(CumpleañosCelebrado e) { Edad++; }
    public void Apply(HijoNacido h) { Hijos.Add(h.NombreHijo); }
}
```

> [!WARNING]
> 🪤 **Trampa real de `dynamic` y la accesibilidad (te vas a topar con esto).** Si declaras los `Apply` como `protected` o `private`, en tiempo de ejecución verás:
> ```
> RuntimeBinderException: 'Persona.Apply(HijoNacido)' is inaccessible due to its protection level
> ```
> ¿Por qué? Porque `dynamic` **sí respeta** las reglas de accesibilidad, evaluadas desde **el lugar donde está escrita la llamada dinámica**. Y `((dynamic)this).Apply(...)` vive en `AggregateRoot.Load`, o sea, en la **clase base**. Un miembro `protected` es accesible desde la clase que lo declara y desde sus **subclases**; pero `AggregateRoot` es la **superclase** de `Persona`, no una subclase, así que **no puede ver** los `Apply` protegidos de `Persona`.
> La regla, en una frase: **el método al que despacha el motor de la base tiene que ser visible desde la base.** Eso lo garantiza `public` (o `internal`, si todo está en el mismo proyecto). Si quieres mantenerlos ocultos, el motor tendría que usar reflexión con `BindingFlags.NonPublic` — más adelante (§22) verás por qué eso es justo lo que evitamos.

> [!NOTE]
> **¿Cómo funciona la magia de `dynamic`? (Dynamic Dispatching)**
> 
> Normalmente (y por defecto), C# tiene un **tipado estricto** en tiempo de compilación. Eso significa que si le pasas una variable declarada como `object` a un método `Apply`, el compilador siempre buscará ejecutar la firma `Apply(object)`, sin importar si tú sabes que por debajo ese objeto es realmente un `PersonaNacida`.
> 
> Pero al castear a `(dynamic)`, le estamos diciendo a C#: *"Apaga la verificación estricta del compilador y espérate a que el programa esté corriendo. Cuando llegue el momento de ejecutar esta línea, fíjate de qué tipo exacto es la variable `ev` en la memoria (ej: `PersonaNacida`) y busca si existe un método llamado `Apply` que reciba exactamente ese tipo."*
> 
> Así es como en tiempo de ejecución (Runtime), el código enruta automáticamente cada evento a su método `Apply(TipoDeEvento)` adecuado sin necesidad de escribir un enorme bloque `if / else`.

> [!TIP]
> Al separar cada evento en su propio método `Apply(TipoDeEvento)`, hemos eliminado la "complejidad cognitiva" del gran `if`. Si la vida de Jhon crece con 50 eventos nuevos, simplemente añades 50 métodos `Apply` aislados (públicos, por lo que vimos arriba) y tu clase `Persona` seguirá siendo hermosamente fácil de leer.

---

[⬅️ Volver a la sección anterior](./03-vivir-el-pasado.md)

[➡️ Siguiente sección: El flujo de vida (EventStream)](./05-el-flujo-de-vida.md)
