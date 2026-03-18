# 03 - Vivir el pasado: El motor Apply

En la sección anterior preparamos nuestro lienzo. Ahora, vamos a empezar a escribir los primeros "párrafos" de la vida de Juan usando C#.

## 🎯 El Objetivo
Imagina que quieres saber cuántos años tiene Juan hoy. **No puedes simplemente guardar `Edad = 34`**, porque eso solo sirve para el presente. Si borras el pasado, no hay forma de volver atrás y saber qué edad tenía en el año 2000. 

¿Cómo construimos un sistema que nos permita "recalcular" su realidad en cualquier momento simplemente leyendo su diario?

---

## El dueño del diario (Identidad)
Todo diario requiere un dueño. Sin un nombre o una huella digital, las páginas no son más que notas sueltas que no le pertenecen a nadie. En el mundo del software, lo primero que necesitamos es definir este vínculo inmutable:

```csharp
// El ancla de nuestra historia
var idPersona = Guid.NewGuid();
```

Este **ID** es lo que nos permite agrupar todos los hechos bajo una sola persona. Es su **Identidad**, y no cambiará sin importar cuántas cosas le pasen a Juan en el futuro.

## Escribiendo los hitos (Los Hechos)
Ahora que sabemos de quién es el diario, registramos lo que le ha pasado. Pero atención: aquí no guardamos "acciones" (como "Mudarse"), sino **hechos consumados** (pasado: "Se mudó"). 

En C# usamos **Records** para esto por dos razones que nos salvarán la vida:
- **Inmutabilidad**: El pasado no se edita. Una vez que Juan nació, ese hecho está "escrito en piedra".
- **Igualdad por valor**: Nos permite comparar si dos hitos son iguales basándonos en sus datos, lo cual es oro puro para hacer pruebas automáticas.

```csharp
public record PersonaNacida(Guid PersonaId, string Nombre, DateTime FechaNacimiento);
public record CumpleañosCelebrado(Guid PersonaId);
```

## El orden de la historia (El Stream)
Listo, ya tenemos los hitos, pero ahora necesitamos saber en qué orden ocurrieron. Un nacimiento después de un cumpleaños no tendría ningún sentido. Para mantener la lógica, creamos una biografía organizada cronológicamente:

```csharp
// Guardamos los hechos en una lista para asegurar el orden
var biografia = new List<object>();

biografia.Add(new PersonaNacida(idPersona, "Juan", new DateTime(1990, 5, 10)));
biografia.Add(new CumpleañosCelebrado(idPersona)); // 1 año
biografia.Add(new CumpleañosCelebrado(idPersona)); // 2 años
```

A esta secuencia organizada la llamamos **Stream**. Es nuestra fuente de la verdad absoluta: si algo no está en este flujo, simplemente nunca sucedió en la vida de Juan.

## Recordar es volver a vivir (El Replay)
Aquí es donde ocurre la magia. Para saber quién es Juan "hoy", ya no consultamos una tabla de estado actual. Simplemente nos sentamos a "leer" su biografía de principio a fin para reconstruir su realidad:

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

Este proceso de lectura se llama **Replay**. Funciona perfecto para un ejemplo pequeño, pero a medida que nuestra aplicación crezca, tener estos bucles repartidos por todo el código será un caos de mantenimiento.

## El jefe de la historia (Aggregate Root)
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

En este modelo, la clase `Persona` es la encargada de cuidar que la historia de Juan sea siempre coherente.

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
