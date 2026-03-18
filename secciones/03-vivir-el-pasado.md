# 03 - Vivir el pasado: El motor Apply

En la sección anterior preparamos nuestro lienzo. Ahora, vamos a empezar a escribir los primeros "párrafos" de la vida de Juan usando C#.

## 🎯 El Objetivo
Imagina que quieres saber cuántos años tiene Juan hoy. **No puedes simplemente guardar `Edad = 34`**, porque eso solo sirve para el presente. Si borras el pasado, no hay forma de volver atrás y saber qué edad tenía en el año 2000. 

¿Cómo construimos un sistema que nos permita "recalcular" su realidad en cualquier momento simplemente leyendo su diario?

---

### Poniendo nombre al diario de Juan
Todo diario requiere un dueño en la portada. Sin un ID, las páginas no son más que notas sueltas. Por eso, empezamos definiendo nuestra "huella digital", lo que llamamos su **Identidad**:

```csharp
// El ancla de nuestra historia
var idPersona = Guid.NewGuid();
```

Este **ID** es el hilo que une todas las páginas de su biografía sin importar cuántas cosas le pasen en el futuro.

### Escribiendo los hitos en piedra
Ahora que sabemos de quién es el diario, necesitamos registrar qué le ha pasado. Pero atención: aquí no guardamos "acciones" (como "Mudarse"), sino **hechos consumados**. 

En el código, estos hitos del pasado se representan mejor usando **Records**. ¿Por qué? Porque el pasado está "escrito en piedra". Los Records nos garantizan **Inmutabilidad**, asegurando que nadie pueda "editar" la fecha de nacimiento de Juan por error, y nos facilitan comparar hitos basándonos en sus datos (**Igualdad por valor**):

```csharp
public record PersonaNacida(Guid PersonaId, string Nombre, DateTime FechaNacimiento);
public record CumpleañosCelebrado(Guid PersonaId);
public record HijoNacido(Guid PersonaId, string NombreHijo);
```

### El orden preciso de su vida
Listo, ya tenemos los hitos, pero ahora nos enfrentamos a un problema: el tiempo. Un nacimiento después de un cumpleaños no tendría sentido. Necesitamos que las páginas del diario estén pegadas en el orden exacto en que ocurrieron:

```csharp
// Guardamos los hechos en una lista para asegurar el orden
var biografia = new List<object>();

biografia.Add(new PersonaNacida(idPersona, "Juan", new DateTime(1990, 5, 10)));
biografia.Add(new CumpleañosCelebrado(idPersona)); // 1 año
biografia.Add(new HijoNacido(idPersona, "Pedro")); // ¡Juan ahora es padre!
biografia.Add(new CumpleañosCelebrado(idPersona)); // 2 años
```

Al unir todas estas páginas en el orden exacto, acabas de crear un **Stream** (Flujo). En el mundo de los eventos, el **Stream** no es solo una lista; es la **Fuente de la Verdad única**. Si un hito no está en este flujo, para nuestro sistema simplemente nunca sucedió.

### Recordar es volver a vivir
¿Cómo usamos esta biografía para saber su edad? Aquí es donde ocurre la magia. Ya no consultamos una tabla estática; simplemente nos sentamos a "leer" su historia de principio a fin para reconstruir su realidad:

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

Este proceso de lectura se llama **Replay**. Has "vuelto a vivir" el pasado para entender el presente. Funciona perfecto para un ejemplo pequeño, pero a medida que nuestra aplicación crezca, tener estos bucles repartidos por todo el código se volverá un caos.

### El protagonista de la historia: El Aggregate Root
Necesitamos a un **Protagonista** que centralice esa biografía y sepa cómo interpretarla, alguien que reciba el diario y se encargue de "despertar" con su estado actualizado. A este personaje principal lo llamamos **Aggregate Root (Raíz del Agregado)**:

```csharp
public class Persona 
{
    public Guid Id { get; private set; }
    public string Nombre { get; private set; }
    public int Edad { get; private set; } 
    public List<string> Hijos { get; private set; } = new();

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
        if (ev is HijoNacido h) { Hijos.Add(h.NombreHijo); }
    }
}

// ¡Uso profesional!
var juan = new Persona(biografia);
Console.WriteLine($"{juan.Nombre} tiene {juan.Edad} años y {juan.Hijos.Count} hijos.");
```

En este modelo, la clase `Persona` es la encargada de cuidar que la historia de Juan sea siempre coherente. Es quien "posee" la verdad de su biografía.

---

### 🧠 ¿Cuál es la diferencia entre Raíz y Agregado?

Para no confundirnos entre el código y la arquitectura, vamos a separar el **Rol** de la **Frontera**:

-   **La Raíz (Aggregate Root - "El Punto de Acceso")**: Es la clase **`Persona`**. En la arquitectura, la Raíz es la "puerta de entrada". Cuando en tu código creas una instancia como `var juan = new Persona()`, ese objeto específico es quien asume el rol de **CEO**: es el único punto de contacto autorizado para realizar cambios. Incluso si quieres añadir un hijo, hablas con Juan (`Persona`), no con la lista de hijos directamente.
-   **El Agregado (Aggregate - "La Frontera de Consistencia")**: Es el concepto total. Es el perímetro invisible que envuelve a Juan, su pasado, sus hijos y sus reglas. El Agregado NO es una clase; es la garantía de que nada de lo que le pase a Juan (la Raíz) o a sus relaciones (sus hijos) rompa las reglas de su vida.

> [!IMPORTANT]
> **La Raíz (`Persona`)** es el objeto que "da la cara". 
> **El Agregado** es el sistema completo (incluyendo sus hijos y su historia) que ese objeto protege. 
> 
> Tú le pides cosas a la **Raíz**, y ella se asegura de que se cumplan las reglas de todo el **Agregado**. Es una unidad indivisible: no puedes modificar un "Hijo" sin que la Raíz lo valide y lo registre en la biografía completa.

---

[⬅️ Volver a la sección anterior](./02-preparando-el-lienzo.md)

[➡️ Siguiente sección: Decidir el futuro (Comandos)](./03b-decidir-el-futuro.md)
