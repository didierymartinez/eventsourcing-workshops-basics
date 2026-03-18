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
```

### El orden preciso de su vida
Listo, ya tenemos los hitos, pero ahora nos enfrentamos a un problema: el tiempo. Un nacimiento después de un cumpleaños no tendría sentido. Necesitamos que las páginas del diario estén pegadas en el orden exacto en que ocurrieron:

```csharp
// Guardamos los hechos en una lista para asegurar el orden
var biografia = new List<object>();

biografia.Add(new PersonaNacida(idPersona, "Juan", new DateTime(1990, 5, 10)));
biografia.Add(new CumpleañosCelebrado(idPersona)); // 1 año
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

Imagina un gran **Navío en alta mar**. Para que el barco llegue a puerto de forma segura, no puedes hablar con cada madera del casco o cada grano de pólvora por separado; necesitas a alguien que responda por todo el conjunto y mantenga el rumbo.

-   **La Raíz (Aggregate Root - "El Capitán")**: Es la clase `Persona`. El Capitán es el único con el que el mundo exterior habla. Él tiene la **Identidad** (el ID), firma la bitácora y toma las decisiones finales basados en lo que ha vivido. Si quieres cambiar el rumbo, hablas con el Capitán, no con el mástil.
-   **El Agregado (Aggregate - "El Navío")**: Es el concepto completo. No es solo el Capitán, sino el Capitán **+** Su Tripulación **+** Sus Provisiones **+** Su Bitácora (Eventos) **+** Las Reglas de Navegación. El Agregado es la **Frontera de Consistencia**; asegura que si el barco gira a la derecha, toda la estructura gire con él y nadie se caiga al agua.

> [!IMPORTANT]
> En nuestro código, tú no manipulas la "Edad" o el "Nombre" desde afuera de forma aislada. Tú interactúas con la **Raíz (Juan)**, y él se encarga de que todo su historial (**El Agregado**) sea coherente y verídico. Si Juan dice que nació en 1990, el Agregado garantiza que no puedas celebrar su cumpleaños número 5 en 1992.

---

[⬅️ Volver a la sección anterior](./02-preparando-el-lienzo.md)

[➡️ Siguiente sección: Decidir el futuro (Comandos)](./03b-decidir-el-futuro.md)
