# 03 - Vivir el pasado: El motor Apply

En la sección anterior preparamos nuestro lienzo. Ahora, vamos a empezar a escribir los primeros "párrafos" de la vida de Juan usando C#.

## 🎯 El Objetivo
Imagina que quieres saber cuántos años tiene Juan hoy.
**No puedes simplemente guardar `Edad = 34`**, porque eso solo sirve para el presente. Si borras el pasado, no hay forma de volver atrás y saber qué edad tenía en el año 2000.

¿Cómo construimos un sistema que nos permita "recalcular" su realidad en cualquier momento simplemente leyendo su diario?

---

## 1. El Protagonista y sus Vivencias
Todo diario necesita un dueño en la portada (ID) y unos hechos registrados (Records).

```csharp
var idPersona = Guid.NewGuid();

// Definimos qué puede pasar
public record PersonaNacida(Guid PersonaId, string Nombre, DateTime FechaNacimiento);
public record CumpleañosCelebrado(Guid PersonaId);

// El Diario (Stream)
var biografia = new List<object>();

biografia.Add(new PersonaNacida(idPersona, "Juan", new DateTime(1990, 5, 10)));
biografia.Add(new CumpleañosCelebrado(idPersona)); // 1 año
biografia.Add(new CumpleañosCelebrado(idPersona)); // 2 años
```

---

## 2. Recalculando la Realidad (El foreach)
Para saber quién es Juan "hoy", no consultamos una base de datos de "estado actual". Consultamos el diario y procesamos los hechos uno por uno.

```csharp
string nombre = "";
int edad = 0;

foreach (var hito in biografia)
{
    if (hito is PersonaNacida n) 
    { 
        nombre = n.Nombre; 
    }
    
    if (hito is CumpleañosCelebrado) 
    { 
        edad++; // ¡No guardamos el número, lo VOLVEMOS a contar!
    }
}

Console.WriteLine($"{nombre} tiene {edad} años.");
```

Este proceso de leer el pasado para entender el presente es lo que llamamos **Replay** (Re-ejecución).

---

## 3. El Descubrimiento: La Clase (Aggregate Root)
Hacer un `foreach` suelto por todo el código cada vez que queramos saber la edad de Juan es desordenado. Necesitamos un **Jefe** que centralice esa lógica y proteja esos datos. 

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
            if (ev is PersonaNacida n) { Id = n.PersonaId; Nombre = n.Nombre; }
            if (ev is CumpleañosCelebrado) { Edad++; }
        }
    }
}

// ¡Uso profesional!
var juan = new Persona(biografia);
Console.WriteLine($"{juan.Nombre} tiene {juan.Edad} años.");
```

### ¿Cuál es la diferencia real?

- **Aggregate Root (La Raíz)**: Es la clase `Persona`. Es el objeto físico que vive en tu memoria RAM. Es el "Capitán" con el que hablas para pedirle datos.
- **Agregado (Aggregate)**: Es el concepto completo. Es **Juan (el objeto) + Su diario (la lista) + La lógica de conteo**. Es la unidad de verdad que asegura que todo tenga sentido (ej: que no puedas cumplir años sin haber nacido).

> [!IMPORTANT]
> El **Aggregate Root** es la puerta de entrada. El **Agregado** es todo lo que hay detrás de la puerta cuidando que la historia de Juan sea consistente.

---

[⬅️ Volver a la sección anterior](./02-preparando-el-lienzo.md)

[➡️ Siguiente sección: Decidir el futuro (Comandos)](./03b-decidir-el-futuro.md)
