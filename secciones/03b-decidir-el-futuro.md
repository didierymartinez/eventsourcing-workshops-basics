# 03b - Tomando decisiones (Comandos)

Jhon ya tiene una biografía y sabe quién es. Pero la vida sigue, y Jhon quiere tomar decisiones.

## 🎯 El Objetivo
Jhon quiere cambiar su nombre. Pero tenemos una regla: **Una vez que naces con un nombre, no puedes cambiarlo en este sistema.**

¿Cómo usamos lo que ya sabemos para proteger esta regla?

---

## 1. La Intención (El Comando)
Un **Hecho** dice lo que pasó (`PersonaNacida`). Un **Comando** dice lo que *queremos* que pase.

```csharp
public record CambiarNombre(Guid PersonaId, string NuevoNombre);
```

---

## 2. El Ciclo de Decisión (El Aggregate Root)
El Aggregate Root (`Persona`) no solo sirve para leer el pasado, sino para **decidir el futuro**.

Añade este método a tu clase `Persona`:

```csharp
public object DecidirCambioNombre(string nuevoNombre)
{
    // Usamos el estado que RECONSTRUIMOS en la sección anterior
    if (!string.IsNullOrEmpty(this.Nombre))
    {
        Console.WriteLine("REGLA: El nombre es inmutable después del nacimiento.");
        return null; // No generamos ningún hecho nuevo
    }

    // Si fuera válido, generaríamos un nuevo hecho
    return new PersonaNacida(this.Id, nuevoNombre, DateTime.Now);
}
```

---

## 3. 🛠️ ¡Pruébalo!
Intenta forzar un cambio de nombre en tu `Program.cs`:

```csharp
var comando = new CambiarNombre(idPersona, "Jhon Sebastian");

// 1. Reconstruimos a Jhon desde el pasado
var juan = new Persona(biografia);

// 2. Le pedimos que decida basado en su estado actual
var resultado = juan.DecidirCambioNombre(comando.NuevoNombre);

if (resultado != null)
{
    biografia.Add(resultado);
}
```

### El Descubrimiento
Acabas de ver la "Unidad de Consistencia". El Agregado (`Persona`) usó su estado reconstruido para proteger una regla de negocio. 

Este es el ciclo real de Event Sourcing:
1.  **Cargar** el pasado (Stream).
2.  **Reconstruir** el estado (Apply).
3.  **Decidir** el futuro (Comandos).
4.  **Registrar** el resultado (Nuevos eventos).

---

[⬅️ Volver a la sección anterior](./03c-refactorizando-el-motor.md)

[➡️ Siguiente sección: El riesgo de olvidar](./04-el-riesgo-de-olvidar.md)
