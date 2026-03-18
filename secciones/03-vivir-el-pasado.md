# 03 - Vivir el pasado: El motor Apply

En la sección anterior preparamos nuestro lienzo. Ahora, vamos a empezar a escribir los primeros "párrafos" de la vida de Juan usando C#.

## 🎯 El Objetivo
Imagina que quieres saber cuántos años tiene Juan hoy. **No puedes simplemente guardar `Edad = 34`**, porque eso solo sirve para el presente. Si borras el pasado, no hay forma de volver atrás y saber qué edad tenía en el año 2000. 

¿Cómo construimos un sistema que nos permita "recalcular" su realidad en cualquier momento simplemente leyendo su diario?

---

Para que este diario sea real, lo primero es ponerle un dueño en la portada. Sin un ID, las páginas no son más que notas sueltas. Por eso, empezamos definiendo nuestra "huella digital":

```csharp
// El ancla de nuestra historia
var idPersona = Guid.NewGuid();
```

Este **ID** es lo que nos permite agrupar todos los hechos bajo una sola persona. Es su **Identidad**, y no cambiará sin importar cuántas cosas le pasen a Juan en el futuro. Es el hilo que une todas las páginas de su biografía.

Ahora que sabemos de quién es el diario, necesitamos registrar qué le ha pasado. Pero atención: aquí no guardamos "acciones" (como "Mudarse"), sino **hechos consumados**. En el código, estos hitos del pasado se representan mejor usando **Records**. 

¿Por qué Records y no clases normales? Porque el pasado está "escrito en piedra". Los Records nos garantizan **Inmutabilidad**, asegurando que nadie pueda "editar" la fecha de nacimiento de Juan por error. Además, nos permiten comparar hitos basándonos en sus datos (**Igualdad por valor**), lo cual es fundamental para saber si lo que guardamos es exactamente lo que pasó:

```csharp
public record PersonaNacida(Guid PersonaId, string Nombre, DateTime FechaNacimiento);
public record CumpleañosCelebrado(Guid PersonaId);
```

Listo, ya tenemos los hitos, pero ahora nos enfrentamos a un problema: el tiempo. Un nacimiento después de un cumpleaños no tendría sentido. Necesitamos que las páginas del diario estén pegadas en el orden exacto en que ocurrieron. Por eso, creamos una biografía organizada cronológicamente:

```csharp
// Guardamos los hechos en una lista para asegurar el orden
var biografia = new List<object>();

biografia.Add(new PersonaNacida(idPersona, "Juan", new DateTime(1990, 5, 10)));
biografia.Add(new CumpleañosCelebrado(idPersona)); // 1 año
biografia.Add(new CumpleañosCelebrado(idPersona)); // 2 años
```

> [!IMPORTANT]
> **El Descubrimiento: El Stream**
> Acabas de crear un **Stream** (Flujo). En el mundo de los eventos, el Stream es la **Fuente de la Verdad única**. No es solo una lista; es la biografía completa de Juan. Si un hecho no está aquí, para el sistema nunca sucedió. El orden de este flujo define la realidad del presente.

Pero, ¿cómo usamos esta biografía para saber su edad? Aquí es donde ocurre la magia. Ya no consultamos una tabla estática; simplemente nos sentamos a "leer" su historia de principio a fin para reconstruir su realidad:

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

Este proceso de lectura se llama **Replay**. Has "vuelto a vivir" el pasado para entender el presente. Funciona perfecto para un ejemplo pequeño, pero a medida que nuestra aplicación crezca y manejemos a miles de personas, tener estos bucles repartidos por todo el código se volverá un caos. Para eso necesitamos un personaje más en esta historia.

## El jefe de la historia: El Aggregate Root
Para poner orden, necesitamos un personaje que centralice esa biografía y sepa cómo interpretarla. Alguien que reciba el diario y se encargue de "despertar" con su estado actualizado. A este jefe lo llamamos **Aggregate Root (Raíz del Agregado)**.

```csharp
public class Persona 
{
    public Guid Id { get; private set; }
    public string Nombre { get; private set; }
    public int Edad { get; private set; } 

    // El Agregado se reconstruye leyendo su pasado
    public Persona(IEnumerable<object> eventos)
    {
        foreach (var ev in eventos)
        {
            Aplicar(ev);
        }
    }

    // El motor que transforma hechos en estado
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

En este modelo, la clase `Persona` es la encargada de cuidar que la historia de Juan sea siempre coherente. Es quien "posee" la verdad de su biografía.

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
