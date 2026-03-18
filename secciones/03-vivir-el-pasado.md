# 03 - Vivir el pasado: El motor Apply

En la sección anterior preparamos nuestro lienzo. Ahora, vamos a empezar a escribir los primeros "párrafos" de la vida de Juan usando C#.

## 🎯 El Objetivo
Imagina que quieres saber en qué ciudad vive Juan y cuántos años tiene hoy.
**No puedes simplemente guardar `Edad = 34`**, porque eso solo sirve para el presente. Si borras el pasado, no hay forma de volver atrás y saber dónde vivía cuando tenía 20 años.

¿Cómo construimos un sistema que nos permita "viajar en el tiempo" y conocer su estado en cualquier momento simplemente leyendo su diario?

---

## 1. El Protagonista (La Identidad)
Todo diario necesita un dueño en la portada. Sin un ID, las páginas no son más que notas sueltas que no le pertenecen a nadie.

```csharp
// El ancla de nuestra historia
var idPersona = Guid.NewGuid();
```

En el mundo del software, a esto lo llamamos **Identidad**. Es como tu huella digital o tu número de pasaporte: no cambia nunca. Es lo que nos permite decir: *"Todos estos hechos le pertenecen a ESTA persona específica y a ninguna otra"*.

---

## 2. Las Vivencias (Los Hechos)
Ahora que tenemos un protagonista, necesitamos definir **qué puede pasarle**. Pero cuidado: en Event Sourcing no guardamos "acciones" (como "Mudarse"), guardamos **hechos consumados** (pasado: "Se mudó").

### ¿Cómo representamos un hecho en C#?
Para que un hecho sea útil, debe ser **inmutable** (el pasado no cambia) y **fácil de comparar**. Por eso usamos **Records** en lugar de clases:

1.  **Inmutabilidad**: Un `record` no se puede modificar accidentalmente. Refleja fielmente que lo que ya pasó está "escrito en piedra".
2.  **Igualdad por valor**: Dos registros son iguales si sus datos son iguales. Esto es **vital para el testeo**: puedes validar que tu sistema generó el hecho correcto simplemente comparando: `hechoGenerado == hechoEsperado`, sin ir campo por campo.

```csharp
// Solo registramos lo que afecta el estado que nos interesa (Event Storming)
public record PersonaNacida(Guid PersonaId, string Nombre, DateTime FechaNacimiento);
public record PersonaSeMudo(Guid PersonaId, string Ciudad);
```

> [!TIP]
> **PersonaId** es el vínculo. Aunque el hecho viva solo, siempre sabe quién es su dueño.

---

## 3. El Diario (El Stream)
Tener hechos sueltos no sirve de nada si no sabemos en qué orden ocurrieron. Un nacimiento después de una mudanza no tendría lógica.

```csharp
// Usamos una Lista para garantizar la secuencia cronológica
var biografia = new List<object>();

biografia.Add(new PersonaNacida(idPersona, "Juan", new DateTime(1990, 5, 10)));
biografia.Add(new PersonaSeMudo(idPersona, "Madrid"));
biografia.Add(new PersonaSeMudo(idPersona, "Barcelona"));
```

A esta secuencia organizada de hechos la llamamos **Stream** (Flujo). El Stream es la **Fuente de la Verdad**. Si el diario dice que Juan vivió en Madrid, Juan vivió en Madrid. No hay otra realidad posible.

---

## 4. Reconstruyendo a Juan (El Aggregate Root)
Aquí es donde unimos todo. Para saber quién es Juan "hoy", creamos una clase que toma su biografía y la "vive" de principio a fin.

```csharp
public class Persona 
{
    public Guid Id { get; private set; }
    public string Nombre { get; private set; }
    public string CiudadActual { get; private set; }

    // El Agregado se reconstruye leyendo su pasado
    public Persona(IEnumerable<object> eventos)
    {
        foreach (var evento in eventos)
        {
            Aplicar(evento);
        }
    }

    // El método 'Aplicar' transforma un hecho en estado actual
    private void Aplicar(object evento)
    {
        if (evento is PersonaNacida n) { Id = n.PersonaId; Nombre = n.Nombre; }
        if (evento is PersonaSeMudo m) { CiudadActual = m.Ciudad; }
    }
}

// ¡Uso!
var juan = new Persona(biografia);
Console.WriteLine($"{juan.Nombre} vive actualmente en {juan.CiudadActual}");
```

### El Descubrimiento
Acabas de crear un puente entre los datos muertos y un objeto vivo:

- **Aggregate Root (La Raíz)**: Es la clase `Persona`. Es el "jefe" que posee el ID y encapsula la lógica para entender la biografía.
- **Agregado**: Es el conjunto completo (El Jefe + Sus Hechos + Su Lógica).
- **Apply (Aplicar)**: Es el motor. Es lo que permite que el Aggregate Root se "despierte" y se ponga al día con su pasado.

---

[⬅️ Volver a la sección anterior](./02-preparando-el-lienzo.md)

[➡️ Siguiente sección: Decidir el futuro (Comandos)](./03b-decidir-el-futuro.md)
