# 04 - Refactorizando el motor: El AggregateRoot

En la sección anterior logramos reconstruir la vida de Jhon leyendo su diario directamente en el constructor. Sin embargo, a medida que Jhon viva más hitos (bodas, mudanzas, trabajos), ese constructor se llenará de un `if` interminable.

Vamos a limpiar nuestra arquitectura siguiendo el principio de **"Separación de Responsabilidades"**.

## 🛠️ Refactor 1: Extraer la lógica de "Traducción"

Lo primero es sacar el procesamiento de los eventos de en medio del constructor. Queremos que el constructor solo "orqueste" la lectura, pero que otro método se encargue de saber qué hacer con cada hito.

A este método lo llamaremos **`Aplicar`** (o *Apply* en inglés):

```csharp
public class Persona 
{
    public Guid Id { get; private set; }
    public string Nombre { get; private set; }
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
        if (ev is PersonaNacida n) { Id = n.PersonaId; Nombre = n.Nombre; }
        if (ev is CumpleañosCelebrado) { Edad++; }
        if (ev is HijoNacido h) { Hijos.Add(h.NombreHijo); }
    }
}
```

## 🏗️ Refactor 2: Generalizando con AggregateRoot

¿Qué pasará cuando tengamos una clase `Mascota` o `Empresa`? Ambas necesitarán un ID y un método para cargar eventos. No queremos repetir este código siempre.

Vamos a crear una **Clase Abstracta** que defina qué es un **Aggregate Root**:

```csharp
public abstract class AggregateRoot
{
    public Guid Id { get; protected set; }

    protected AggregateRoot() { }

    // El motor genérico de reconstrucción
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
    public int Edad { get; private set; }
    public List<string> Hijos { get; private set; } = new();

    public Persona(IEnumerable<object> eventos)
    {
        Load(eventos);
    }

    protected override void Aplicar(object ev)
    {
        if (ev is PersonaNacida n) { Id = n.PersonaId; Nombre = n.Nombre; }
        if (ev is CumpleañosCelebrado) { Edad++; }
        if (ev is HijoNacido h) { Hijos.Add(h.NombreHijo); }
    }
}
```

## 🚀 Refactor 3: El toque final con Overloads (Elegancia Pro)

Ese `if (ev is ...)` sigue pareciendo un poco rudimentario. En sistemas más avanzados, aprovechamos que C# puede seleccionar qué método ejecutar según el tipo de dato.

Podemos dejar el método `Aplicar(object ev)` vacío (o que use `dynamic`) y simplemente sobrecargar el método para cada tipo de evento:

```csharp
public class Persona : AggregateRoot
{
    // ... propiedades ...

    public Persona(IEnumerable<object> eventos) => Load(eventos);

    protected override void Aplicar(object ev)
    {
        // Simulamos la elegancia con sobrecargas:
        ((dynamic)this).Apply((dynamic)ev);
    }

    // Sobrecargas específicas: Corto, claro y tipado.
    private void Apply(PersonaNacida n) { Id = n.PersonaId; Nombre = n.Nombre; }
    private void Apply(CumpleañosCelebrado e) => Edad++;
    private void Apply(HijoNacido h) => Hijos.Add(h.NombreHijo);
}
```

> [!TIP]
> Al separar cada evento en su propio método `Apply`, hemos eliminado la complejidad cognitiva. Si el sistema crece, solo añades un método `Apply` nuevo y listo.

---

[⬅️ Volver a la sección anterior](./03-vivir-el-pasado.md)

[➡️ Siguiente sección: Decidir el futuro (Comandos)](./05-decidir-el-futuro.md)
