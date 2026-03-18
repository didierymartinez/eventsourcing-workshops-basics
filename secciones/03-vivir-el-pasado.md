# 03 - Vivir el pasado: El motor Apply

En la sección anterior preparamos nuestro lienzo. Ahora, vamos a empezar a escribir los primeros "párrafos" de la vida de Juan usando C#.

## 🎯 El Objetivo
Imagina que quieres saber cuántos años tiene Juan hoy.
**No puedes simplemente guardar `Edad = 34`**, porque eso solo sirve para el presente. Si borras el pasado, no hay forma de volver atrás y saber qué edad tenía en el año 2000.

¿Cómo construimos un sistema que nos permita "recalcular" su realidad en cualquier momento simplemente leyendo su diario?

---

## 1. El Protagonista (La Identidad)
Todo diario necesita un dueño en la portada. Sin un ID, las páginas no son más que notas sueltas.

```csharp
// El ancla de nuestra historia
var idPersona = Guid.NewGuid();
```

En el mundo del software, a esto lo llamamos **Identidad**. Es como tu huella digital o tu número de pasaporte: no cambia nunca. Es lo que nos permite decir: *"Todos estos hechos le pertenecen a ESTA persona específica"*.

---

## 2. Las Vivencias (Los Hechos)
Ahora definimos **qué puede pasarle**. Pero en Event Sourcing no guardamos "acciones", guardamos **hechos consumados**.

### ¿Cómo representamos un hecho en C#?
Usamos **Records** en lugar de clases por dos razones vitales:

1.  **Inmutabilidad**: Un `record` no se puede modificar. El pasado está "escrito en piedra".
2.  **Igualdad por valor**: Dos registros son iguales si sus datos coinciden. Esto es clave para validar que el sistema generó el hecho correcto (`hechoGenerado == hechoEsperado`).

```csharp
public record PersonaNacida(Guid PersonaId, string Nombre, DateTime FechaNacimiento);
public record CumpleañosCelebrado(Guid PersonaId);
```

---

## 3. El Diario (El Stream)
Tener hechos sueltos no sirve de nada sin orden. Un nacimiento después de un cumpleaños no tendría lógica.

```csharp
// Usamos una Lista para garantizar la secuencia cronológica
var biografia = new List<object>();

biografia.Add(new PersonaNacida(idPersona, "Juan", new DateTime(1990, 5, 10)));
biografia.Add(new CumpleañosCelebrado(idPersona));
biografia.Add(new CumpleañosCelebrado(idPersona));
```

A esta secuencia la llamamos **Stream** (Flujo). El Stream es la **Fuente de la Verdad**.

---

## 4. Recalculando la Realidad (Proceso Manual)
Para saber quién es Juan "hoy", no consultamos una base de datos de "estado actual". Consultamos el diario y procesamos los hechos uno por uno.

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

Este proceso de leer el pasado para entender el presente es lo que llamamos **Replay**. Funciona, pero si tenemos muchas personas, tener este `foreach` suelto por todos lados será un caos.

---

## 5. El Descubrimiento: La clase (Aggregate Root)
Necesitamos un **Jefe** que centralice esa lógica y proteja esos datos. Alguien que reciba la biografía y se encargue de reconstruirse a sí mismo.

A este jefe lo llamamos **Aggregate Root (Raíz del Agregado)**.

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

### 🧠 ¿Raíz o Agregado? (La diferencia final)

- **Aggregate Root (La Raíz)**: Es la clase `Persona`. El objeto físico en memoria con el que interactúas.
- **Agregado (Aggregate)**: Es el concepto completo. Es **Juan (objeto) + Su diario (Stream) + Su lógica**. 

> [!IMPORTANT]
> El **Aggregate Root** es la puerta de entrada (la clase). El **Agregado** es todo lo que hay detrás de la puerta cuidando que la historia de Juan sea consistente.

---

[⬅️ Volver a la sección anterior](./02-preparando-el-lienzo.md)

[➡️ Siguiente sección: Decidir el futuro (Comandos)](./03b-decidir-el-futuro.md)
