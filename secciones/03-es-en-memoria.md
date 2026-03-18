# 03 - Escribiendo la biografía en código

En la sección anterior preparamos nuestro lienzo. Ahora, vamos a anotar los primeros hechos de Juan usando C#.

## 🎯 El Objetivo
Imagina que quieres saber en qué ciudad vive Juan y cuántos años tiene. 
**No puedes guardar variables fijas**, porque perderías su historia de mudanzas y crecimiento. 

¿Cómo construimos un objeto que sepa quién es Juan simplemente leyendo su diario?

---

## 1. El Protagonista (La Identidad)
Todo diario necesita un dueño.

```csharp
var idPersona = Guid.NewGuid();
```

Este es el **ID**. Es el ancla que une todos los hechos de Juan.

---

## 2. Las Vivencias (Records)
Usemos **Records** para representar hechos inmutables. 

```csharp
public record PersonaNacida(Guid PersonaId, string Nombre, DateTime FechaNacimiento);
public record PersonaSeMudo(Guid PersonaId, string Ciudad);
```

---

## 3. El Diario (El Stream)
Guardamos los hechos en una lista para mantener el orden. Este es el **Stream**.

```csharp
var biografia = new List<object>();

biografia.Add(new PersonaNacida(idPersona, "Juan", new DateTime(1990, 5, 10)));
biografia.Add(new PersonaSeMudo(idPersona, "Madrid"));
biografia.Add(new PersonaSeMudo(idPersona, "Barcelona"));
```

---

## 4. El Motor de Reconstrucción (El Método Apply)
Aquí es donde ocurre la magia. Para saber quién es Juan "hoy", creamos una clase que se "traga" los hechos uno a uno.

```csharp
public class Persona 
{
    public Guid Id { get; private set; }
    public string Nombre { get; private set; }
    public string CiudadActual { get; private set; }

    public Persona(IEnumerable<object> eventos)
    {
        foreach (var evento in eventos)
        {
            Aplicar(evento);
        }
    }

    private void Aplicar(object evento)
    {
        if (evento is PersonaNacida n) 
        { 
            Id = n.PersonaId; 
            Nombre = n.Nombre; 
        }
        
        if (evento is PersonaSeMudo m) 
        { 
            CiudadActual = m.Ciudad; 
        }
    }
}

// ¡Uso!
var juan = new Persona(biografia);
Console.WriteLine($"{juan.Nombre} vive en {juan.CiudadActual}");
```

### El Descubrimiento
- **Aggregate Root (La Raíz)**: Es la clase `Persona`. Es quien tiene el ID y encapsula la lógica para entender la biografía.
- **Agregado**: Es la unión completa (Persona + Biografía + Lógica de reconstrucción). 
- **Apply (Aplicar)**: Es el mecanismo que transforma un hecho del pasado en estado actual del objeto.

---

[⬅️ Volver a la sección anterior](./02-primer-proyecto.md)

[➡️ Siguiente sección: Tomando decisiones (Comandos)](./03b-ciclo-comandos.md)
