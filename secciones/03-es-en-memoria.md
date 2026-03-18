# 03 - Escribiendo la biografía en código

En la sección anterior preparamos nuestro lienzo. Ahora, vamos a escribir los primeros "párrafos" de la vida de Juan usando C#.

## 🎯 El Objetivo
Imagina que quieres saber exactamente qué edad tenía Juan en el año 2005. 
**No puedes simplemente guardar `Edad = 34`**, porque eso solo sirve para el presente. Si borras el pasado, no hay forma de volver atrás.

¿Cómo construimos un sistema que nos permita "viajar en el tiempo" y conocer su estado en cualquier momento?

---

## 1. El Protagonista (La Identidad)
Todo diario necesita un dueño. Sin un nombre o un ID en la portada, las páginas no son más que notas sueltas.

```csharp
// El ancla de nuestra historia
var idPersona = Guid.NewGuid();
```

En el mundo del software, a esto lo llamamos **Identidad**. Es como tu huella digital: no cambia nunca. Es lo que nos permite decir: "Todos estos hechos le pertenecen a ESTA persona y a ninguna otra".

---

## 2. Las Vivencias (Los Hechos)
Ahora que tenemos un protagonista, necesitamos definir **qué puede pasarle**. Pero en Event Sourcing no guardamos "acciones", guardamos **hechos consumados**.

### ¿Cómo representamos un hecho en C#?
Para que un hecho sea útil, debe ser **inmutable** (el pasado no cambia) y **fácil de comparar**. Por eso usamos **Records** en lugar de clases:

1.  **Inmutabilidad**: Un `record` no se puede modificar una vez creado. Refleja fielmente que el pasado está "escrito en piedra".
2.  **Igualdad por valor**: Si dos registros tienen la misma fecha e ID, son el mismo hecho. Esto es vital para el testeo: `hechoGenerado == hechoEsperado` funciona sin tener que comparar campo por campo.

```csharp
// Definimos los hitos que realmente afectan la vida de Juan
// Usamos 'PersonaId' para que sea claro que este hecho le pertenece a él
public record PersonaNacida(Guid PersonaId, string Nombre, DateTime FechaNacimiento);
public record CumpleañosCelebrado(Guid PersonaId, DateTime Fecha);
```

> [!NOTE]
> Usar un nombre descriptivo como **PersonaId** en lugar de solo `Id` es una buena práctica. En un sistema real, podrías tener muchos IDs (ID del evento, ID de la dirección, etc.). Llamarlo `PersonaId` deja claro que este es el vínculo con el protagonista.

---

## 3. El Diario (El Stream)
Tener hitos sueltos no es suficiente. Para que una vida tenga sentido, los hechos deben estar **en orden**. Un nacimiento después de un cumpleaños no tendría lógica.

```csharp
// Usamos una Lista para garantizar el orden cronológico
var biografia = new List<object>();

biografia.Add(new PersonaNacida(idPersona, "Juan", new DateTime(1990, 5, 10)));
biografia.Add(new CumpleañosCelebrado(idPersona, new DateTime(1991, 5, 10)));
biografia.Add(new CumpleañosCelebrado(idPersona, new DateTime(1992, 5, 10)));
```

A esta secuencia organizada de hechos la llamamos **Stream** (Flujo). El Stream es la **Fuente de la Verdad**. Si el diario dice que Juan se mudó, Juan se mudó. No hay otra realidad.

---

## 4. El Resultado (El Agregado)
Ahora que tenemos la Identidad (ID), los Hechos (Records) y el Diario (Stream), podemos resolver el desafío: Calcular su edad.

```csharp
int edadCalculada = 0;

// Reconstruimos la realidad leyendo el diario de principio a fin
foreach (var hito in biografia)
{
    if (hito is CumpleañosCelebrado)
    {
        edadCalculada++;
    }
}

Console.WriteLine($"Juan tiene {edadCalculada} años.");
```

### El Descubrimiento: El Agregado
Este "todo" que acabas de construir es lo que en DDD llamamos un **Agregado**.

*   **Aggregate Root (La Raíz)**: Es el concepto de "Persona". Es el jefe que tiene el ID y quien decide qué hechos se anotan en el diario.
*   **Agregado**: Es la unión de la Raíz + su Stream de eventos + la Lógica para reconstruir el estado.

> [!IMPORTANT]
> El Agregado es una **frontera de consistencia**. Es quien asegura que las reglas se cumplan (ej: no puedes cumplir años si no has nacido). Él lee el pasado para decidir el futuro.

---

[⬅️ Volver a la sección anterior](./02-primer-proyecto.md)

[➡️ Siguiente sección: El gran problema](./04-el-problema.md)
