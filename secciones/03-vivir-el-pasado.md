# 03 - Vivir el pasado: El motor Apply

En la sección anterior preparamos nuestro lienzo. Ahora, vamos a empezar a escribir los primeros "párrafos" de la vida de Juan usando C#.

## 🎯 El Objetivo
Imagina que quieres saber cuántos años tiene Juan hoy.
**No puedes simplemente guardar `Edad = 34`**, porque eso solo sirve para el presente. Si borras el pasado, no hay forma de volver atrás y saber qué edad tenía en el año 2000.

¿Cómo construimos un sistema que nos permita "recalcular" su realidad en cualquier momento simplemente leyendo su diario?

---

## El Protagonista y su Identidad
Todo diario necesita un dueño en la portada. Sin un nombre o una huella digital, las páginas no son más que notas sueltas que no le pertenecen a nadie.

```csharp
// El ancla de nuestra historia
var idPersona = Guid.NewGuid();
```

Este es el **ID**. En el mundo real es tu huella; en el software es lo que nos permite decir: *"Todos estos hechos le pertenecen a ESTA persona específica y a ninguna otra"*.

---

## Los Hechos: Escribiendo en Piedra
Una vez que tenemos al protagonista, registramos lo que le pasa. Pero en Event Sourcing no guardamos "acciones" (como "Mudarse"), guardamos **hechos consumados** (pasado: "Se mudó").

Para esto usamos **Records** en C#. Un hecho del pasado no puede cambiar (es inmutable) y dos nacimientos con los mismos datos son el mismo nacimiento (igualdad por valor).

```csharp
public record PersonaNacida(Guid PersonaId, string Nombre, DateTime FechaNacimiento);
public record CumpleañosCelebrado(Guid PersonaId);
```

---

## La Biografía: El Orden de los Factores
Tener hechos sueltos no sirve de nada si no sabemos cuándo ocurrieron. Un nacimiento después de un cumpleaños no tendría lógica. Por eso, guardamos todo en una lista organizada.

```csharp
// El Stream (Diario)
var biografia = new List<object>();

biografia.Add(new PersonaNacida(idPersona, "Juan", new DateTime(1990, 5, 10)));
biografia.Add(new CumpleañosCelebrado(idPersona)); // 1 año
biografia.Add(new CumpleañosCelebrado(idPersona)); // 2 años
```

A esta secuencia la llamamos **Stream**. Es la **Fuente de la Verdad**: si no está en el diario, no pasó.

---

## Recordar es volver a vivir (Replay)
Aquí es donde ocurre la magia. Para saber quién es Juan "hoy", no consultamos una base de datos de "estado actual". Simplemente "leemos" su diario de principio a fin.

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

Acabas de hacer un **Replay**. Has reconstruido la realidad a partir de hechos pasados. 

---

## El Descubrimiento: La clase (Aggregate Root)
Hacer ese `foreach` suelto por todo el código cada vez que queramos saber algo de Juan es desordenado. Necesitamos un **Jefe** que centralice esa biografía y sepa cómo interpretarla.

A este jefe lo llamamos **Aggregate Root (Raíz del Agregado)**.

```csharp
public class Persona 
{
    public Guid Id { get; private set; }
    public string Nombre { get; private set; }
    public int Edad { get; private set; } 

    // El Agregado se "despierta" leyendo su pasado
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

### 🧠 ¿Raíz o Agregado? (La diferencia final)

- **Aggregate Root (La Raíz)**: Es la clase `Persona`. El objeto físico con el que hablas para pedirle datos.
- **Agregado (Aggregate)**: Es el concepto completo. Es **Juan (el objeto) + Su diario (Stream) + Su lógica**. 

> [!IMPORTANT]
> El **Aggregate Root** es la puerta de entrada (la clase). El **Agregado** es todo lo que hay detrás de la puerta cuidando que la historia de Juan sea siempre coherente.

---

[⬅️ Volver a la sección anterior](./02-preparando-el-lienzo.md)

[➡️ Siguiente sección: Decidir el futuro (Comandos)](./03b-decidir-el-futuro.md)
