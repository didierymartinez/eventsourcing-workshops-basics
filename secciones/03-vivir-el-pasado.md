# 03 - Vivir el pasado: El motor Apply

En la sección anterior preparamos nuestro lienzo. Ahora, vamos a empezar a escribir los primeros "párrafos" de la vida de Juan usando C#.

## 🎯 El Objetivo
Imagina que quieres saber cuántos años tiene Juan hoy. **No puedes simplemente guardar `Edad = 34`**, porque eso solo sirve para el presente. Si borras el pasado, no hay forma de volver atrás y saber qué edad tenía en el año 2000.

¿Cómo construimos un sistema que nos permita "recalcular" su realidad en cualquier momento simplemente leyendo su diario?

---

### Poniendo nombre al diario de Juan
Todo diario necesita un dueño en la portada. Sin un ID, las páginas no son más que notas sueltas que no le pertenecen a nadie. Por eso, lo primero que necesitamos es definir una "huella digital":

```csharp
// El ancla de nuestra historia
var idPersona = Guid.NewGuid();
```

Este **ID** es lo que nos permite decir que todos los hechos que siguen le pertenecen a una persona específica y a ninguna otra. Es lo que formalmente llamamos una **Identidad**.

### Escribiendo los hitos en piedra
Ahora que sabemos de quién es el diario, necesitamos registrar qué le ha pasado. Pero cuidado: aquí no guardamos "acciones" (como "Mudarse"), sino **hechos consumados** (pasado: "Se mudó").

Para que estos hechos sean útiles, usamos **Records** en C#. Al ser inmutables, nos aseguran que el pasado no se puede "editar" por accidente; y al tener igualdad por valor, nos facilitan la vida al testear si lo que pasó es lo que esperábamos:

```csharp
public record PersonaNacida(Guid PersonaId, string Nombre, DateTime FechaNacimiento);
public record CumpleañosCelebrado(Guid PersonaId);
```

### El orden en que ocurrió su vida
Listo, ya tenemos los hitos de Juan, pero ahora necesitamos saber el orden en que ocurrieron. Un nacimiento después de un cumpleaños no tendría lógica. Por eso, necesitamos crear una **Biografía** donde guardamos todo en una secuencia organizada:

```csharp
// Guardamos todo en una lista para mantener la cronología
var biografia = new List<object>();

biografia.Add(new PersonaNacida(idPersona, "Juan", new DateTime(1990, 5, 10)));
biografia.Add(new CumpleañosCelebrado(idPersona));
biografia.Add(new CumpleañosCelebrado(idPersona));
```

A esta secuencia de hechos organizada por tiempo la llamamos **Stream**. Es nuestra fuente de la verdad absoluta: si no está en el diario, no pasó.

### Recordar es volver a vivir
Aquí es donde ocurre la magia. Para saber quién es Juan "hoy", ya no consultamos una tabla de estado actual. Simplemente nos sentamos a "leer" su biografía de principio a fin para reconstruir su realidad en el presente:

```csharp
string nombre = "";
int edad = 0;

foreach (var hito in biografia)
{
    if (hito is PersonaNacida n) { nombre = n.Nombre; }
    if (hito is CumpleañosCelebrado) { edad++; }
}

Console.WriteLine($"{nombre} tiene {edad} años.");
```

Este proceso de lectura se llama **Replay**. Funciona perfecto para un ejemplo pequeño, pero a medida que nuestra aplicación crezca y tengamos miles de personas, tener estos bucles sueltos por todos lados se volverá inmanejable.

### El Jefe de la Historia
Para poner orden, necesitamos un **Jefe** que centralice esa biografía y sepa cómo interpretarla. Alguien que reciba el diario y se encargue de "despertar" con su estado actualizado. A este jefe lo llamamos **Aggregate Root (Raíz del Agregado)**.

```csharp
public class Persona 
{
    public Guid Id { get; private set; }
    public string Nombre { get; private set; }
    public int Edad { get; private set; } 

    public Persona(IEnumerable<object> eventos)
    {
        foreach (var ev in eventos)
        {
            Aplicar(ev);
        }
    }

    private void Aplicar(object ev)
    {
        if (ev is PersonaNacida n) { Id = n.PersonaId; Nombre = n.Nombre; }
        if (ev is CumpleañosCelebrado) { Edad++; }
    }
}

// ¡Uso profesional!
var juan = new Persona(biografia);
Console.WriteLine($"{juan.Nombre} tiene {juan.Edad} años.");
```

---

### 🧠 ¿Cuál es la diferencia entre Raíz y Agregado?

Aunque a veces los usamos como sinónimos, hay una jerarquía importante:

- **Aggregate Root (La Raíz)**: Es la clase `Persona`. El objeto físico con el que hablas en tu código (el "Capitán"). Es quien tiene el ID.
- **Agregado (Aggregate)**: Es el concepto completo. Es **Juan + Su Diario + Su Lógica**. Es la frontera invisible que asegura que toda su vida sea coherente.

> [!IMPORTANT]
> La **Raíz** es la puerta de entrada. El **Agregado** es todo lo que vive detrás de esa puerta cuidando que no se rompa ninguna regla de la historia de Juan.

---

[⬅️ Volver a la sección anterior](./02-preparando-el-lienzo.md)

[➡️ Siguiente sección: Decidir el futuro (Comandos)](./03b-decidir-el-futuro.md)
