# 03 - Vivir el pasado: El motor Apply

En la sección anterior preparamos nuestro lienzo. Ahora, vamos a empezar a escribir los primeros "párrafos" de la vida de Juan usando C#.

## 🎯 El Objetivo
Imagina que quieres saber cuántos años tiene Juan hoy.
**No puedes simplemente guardar `Edad = 34`**, porque eso solo sirve para el presente. Si borras el pasado, no hay forma de volver atrás y saber cuántos años tenía en el año 2000.

¿Cómo construimos un sistema que nos permita "recalcular" su edad en cualquier momento simplemente leyendo su diario?

---

## 1. El Protagonista (La Identidad)
Todo diario necesita un dueño en la portada. Sin un ID, las páginas no son más que notas sueltas.

```csharp
var idPersona = Guid.NewGuid();
```

Este es el **ID (Identidad)**. Es lo que nos permite decir: *"Todos estos hechos le pertenecen a ESTA persona específica"*.

---

## 2. Las Vivencias (Records)
Usemos **Records** para representar hechos inmutables. 

```csharp
public record PersonaNacida(Guid PersonaId, string Nombre, DateTime FechaNacimiento);
public record CumpleañosCelebrado(Guid PersonaId);
```

---

## 3. El Diario (El Stream)
Guardamos los hechos en una lista para mantener el orden. Este es el **Stream**.

```csharp
var biografia = new List<object>();

// Hechos acumulados
biografia.Add(new PersonaNacida(idPersona, "Juan", new DateTime(1990, 5, 10)));
biografia.Add(new CumpleañosCelebrado(idPersona)); // 1 año
biografia.Add(new CumpleañosCelebrado(idPersona)); // 2 años
```

---

## 4. Reconstrucción: El Aggregate Root
Aquí es donde unimos todo. Creamos una clase que toma esa "biografía" y la procesa para saber el estado actual.

```csharp
public class Persona 
{
    public Guid Id { get; private set; }
    public string Nombre { get; private set; }
    public int Edad { get; private set; } // Estado RECONSTRUIDO

    public Persona(IEnumerable<object> eventos)
    {
        foreach (var evento in eventos)
        {
            Aplicar(evento);
        }
    }

    private void Aplicar(object evento)
    {
        if (evento is PersonaNacida n) 
        { 
            Id = n.PersonaId; 
            Nombre = n.Nombre; 
        }
        
        if (evento is CumpleañosCelebrado) 
        { 
            Edad++; // No guardamos el número, lo VOLVEMOS a contar
        }
    }
}

// ¡Uso!
var juan = new Persona(biografia);
Console.WriteLine($"{juan.Nombre} tiene {juan.Edad} años.");
```

---

### 🧠 El Descubrimiento: ¿Raíz o Agregado?

Acabas de crear un puente entre los datos "muertos" (la lista) y un objeto "vivo" (Juan). En DDD, esta diferencia es clave:

#### 1. Aggregate Root (La Raíz)
Es la **clase `Persona`**. 
- Es el objeto con el que hablas en el código.
- Es quien tiene la autoridad de decir: *"Cargaré mi historia y actualizaré mi Edad"*.
- **En el código**: Es la instancia `juan`.

#### 2. Agregado (Aggregate)
Es el **concepto completo**. Es la suma de:
**La Raíz (`Persona`) + Su historia (`Stream`) + Sus reglas de consistencia.**
- No es una línea de código específica, es la unidad que garantiza que Juan sea "válido" (ej: que no tenga edad si no ha nacido).
- **En la analogía**: Juan no es solo un cuerpo (clase); es su cuerpo más toda la historia que lo define y lo hace quien es.

> [!TIP]
> **Diferencia rápida**: El **Root** es la puerta (la clase `Persona`). El **Agregado** es todo lo que hay detrás de la puerta (la historia + la lógica + el ID).

---

[⬅️ Volver a la sección anterior](./02-preparando-el-lienzo.md)

[➡️ Siguiente sección: Decidir el futuro (Comandos)](./03b-decidir-el-futuro.md)
