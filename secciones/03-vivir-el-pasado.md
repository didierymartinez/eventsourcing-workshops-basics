# 03 - Vivir el pasado: El motor Apply

En la sección anterior preparamos nuestro lienzo. Ahora, vamos a empezar a escribir los primeros "párrafos" de la vida de Jhon usando C#.

## 🎯 El Objetivo
Imagina que quieres saber cuántos años tiene Jhon hoy. **No puedes simplemente guardar `Edad = 34`**, porque eso solo sirve para el presente. Si borras el pasado, no hay forma de volver atrás y saber qué edad tenía en el año 2000. 

¿Cómo construimos un sistema que nos permita "recalcular" su realidad en cualquier momento simplemente leyendo su diario?

---

### Escribiendo los hitos en piedra
Ahora que sabemos de quién es el diario, necesitamos registrar qué le ha pasado. Pero atención: aquí no guardamos "acciones" (como "Mudarse"), sino **hechos consumados**. 

En el código, estos hitos del pasado se representan mejor usando **Records**. ¿Por qué? Porque el pasado está "escrito en piedra". Los Records nos garantizan **Inmutabilidad**, asegurando que nadie pueda "editar" la fecha de nacimiento de Jhon por error, y nos facilitan comparar hitos basándonos en sus datos (**Igualdad por valor**):

```csharp
public record PersonaNacida(string Nombre, DateTime FechaNacimiento, string Ciudad);
public record CumpleañosCelebrado();
public record HijoNacido(string NombreHijo);
```

### El orden preciso de su vida
Listo, ya tenemos los hitos, pero ahora nos enfrentamos a un problema: el tiempo. Un nacimiento después de un cumpleaños no tendría sentido. Necesitamos que las páginas del diario estén pegadas en el orden exacto en que ocurrieron:

```csharp
// Guardamos los hechos en una lista para asegurar el orden
var biografia = new List<object>();

biografia.Add(new PersonaNacida("Jhon", new DateTime(1990, 5, 10), "Bogotá"));

// Celebramos sus primeros 30 cumpleaños
for (int i = 0; i < 30; i++)
{
    biografia.Add(new CumpleañosCelebrado());
}

biografia.Add(new HijoNacido("Pedro")); // ¡A los 30 años, Jhon es padre!
biografia.Add(new CumpleañosCelebrado()); // Cumple 31 años
```

Al unir todas estas páginas en el orden exacto, acabas de crear un **Stream** (Flujo). En el mundo de los eventos, el **Stream** no es solo una lista; es la **Fuente de la Verdad única**. Si un hito no está en este flujo, para nuestro sistema simplemente nunca sucedió.

### Recordar es volver a vivir
¿Cómo usamos esta biografía para saber su edad? Aquí es donde ocurre la magia. Ya no consultamos una tabla estática; simplemente nos sentamos a "leer" su historia de principio a fin para reconstruir su realidad (a este proceso la industria lo llama **Rehidratar** el estado o **State Rehydration**):

```csharp
string nombre = "";
int edad = 0;
string ciudad = "";
List<string> hijos = new();

foreach (var hito in biografia)
{
    if (hito is PersonaNacida n) { nombre = n.Nombre; ciudad = n.Ciudad; }
    if (hito is CumpleañosCelebrado) { edad++; }
    if (hito is HijoNacido h) { hijos.Add(h.NombreHijo); }
}

Console.WriteLine($"Persona: {nombre} tiene {edad} años, nació en {ciudad} y tiene {hijos.Count} hijos.");
```

Este proceso de lectura se llama **Replay**. Has "vuelto a vivir" el pasado para rehidratar el presente.

¿Notaste cómo tuvimos que usar variables sueltas (`nombre`, `edad`, `hijos`) para capturar la información? **Saltar de este código imperativo a un código de objetos significa conceptualizar estas variables como propiedades de un objeto llamado `Persona`.**

### De variables sueltas a una clase que estructura el estado

Las variables `nombre`, `edad`, `ciudad` e `hijos` son útiles para un script puntual, pero en una aplicación real necesitamos algo mejor: una **clase** que agrupe todas esas variables como propiedades tipadas, que pueda ser instanciada múltiples veces (para Jhon, para Ana, para 1.000 personas), y que centralice la lógica de cómo interpretar la historia.

En diseño de software orientado al dominio (DDD), a esta clase se le da un nombre muy específico: **Aggregate Root** (Raíz del Agregado). El nombre tiene dos partes con significados distintos:

- **Agregado** (*Aggregate*): "Clase" es un concepto del lenguaje de programación — puede ser un helper, un servicio, un DTO. "Agregado" en cambio es un **concepto del dominio del negocio**: describe una frontera de consistencia, es decir, un grupo de objetos que las *reglas del negocio* exigen tratar como una unidad indivisible. Jhon no es solo un nombre y una edad — para el negocio, Jhon es el conjunto de su historia, sus hijos y sus datos, y ninguna parte de ese conjunto puede modificarse de forma independiente sin romper la coherencia del todo. El Agregado es esa frontera; existe en el diseño *antes* de que abramos el editor.
- **Raíz** (*Root*): porque dentro de ese conjunto, esta clase es el **único punto de entrada autorizado**. Nadie puede tocar las ramas (por ejemplo, la lista de hijos) sin pasar por la raíz. Es como la raíz de un árbol: sostiene y controla todo lo que cuelga de ella.


```csharp
public class Persona 
{
    public string Nombre { get; private set; }
    public string Ciudad { get; private set; }
    public int Edad { get; private set; } 
    public List<string> Hijos { get; private set; } = new();

    // El Aggregate Root se rehidrata leyendo su pasado (Replay)
    public Persona(IEnumerable<object> eventos)
    {
        foreach (var ev in eventos)
        {
            // Al principio, procesamos todo aquí directamente
            if (ev is PersonaNacida n) { Nombre = n.Nombre; Ciudad = n.Ciudad; }
            if (ev is CumpleañosCelebrado) { Edad++; }
            if (ev is HijoNacido h) { Hijos.Add(h.NombreHijo); }
        }
    }
}
```

```csharp

var jhon = new Persona(biografia);

// ¡La transición a objetos es evidente aquí!
// Ya no imprimimos variables sueltas, sino que accedemos a las propiedades del objeto Persona.
Console.WriteLine($"Persona: {jhon.Nombre} tiene {jhon.Edad} años, nació en {jhon.Ciudad} y tiene {jhon.Hijos.Count} hijos.");
```

En este modelo, la clase `Persona` es la encargada de cuidar que la historia de Jhon sea siempre coherente. Es quien "posee" la verdad de su biografía.

---

### 🧠 ¿Cuál es la diferencia entre Aggregate Root y Aggregate?

Para no confundirnos entre el código y la arquitectura, vamos a separar el **Rol** de la **Frontera**:

-   **El Aggregate Root (Punto de Acceso)**: Es la clase **`Persona`**. En la arquitectura, el Aggregate Root es la "puerta de entrada" y el único punto de contacto autorizado para realizar cambios. Incluso si quieres añadir un hijo, debes pedírselo al objeto `Persona`, no puedes manipular la lista de hijos por separado.
-   **El Aggregate (Frontera de Consistencia)**: Es el concepto total. Es el perímetro invisible que envuelve a Jhon, su pasado, sus hijos y sus leyes internas. El Aggregate NO es una clase; es la garantía de que nada de lo que le pase a Jhon (el Aggregate Root) o a sus relaciones (sus hijos) rompa la coherencia de su historia.

---

[⬅️ Volver a la sección anterior](./02-preparando-el-lienzo.md)

[➡️ Siguiente sección: Refactorizando el motor](./04-refactorizando-el-motor.md)
