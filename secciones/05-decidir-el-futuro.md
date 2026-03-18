# 05 - Decidir el futuro: Comandos y Reglas

Jhon ya tiene una biografía y sabe quién es. Pero la vida sigue, y Jhon quiere tomar decisiones. Hasta ahora solo hemos visto el pasado, pero ¿cómo manejamos las intenciones de cambio en el presente?

## 🎯 El Objetivo
Jhon quiere cambiar su nombre. Pero tenemos una regla sagrada: **Una vez que naces con un nombre, no puedes cambiarlo en este sistema.**

Vamos a aprender cómo usar el estado que ya tenemos para proteger esta regla.

---

## 1. La Intención (El Comando)
En Event Sourcing, diferenciamos claramente entre un hecho pasado y una intención presente.
- **Hito (Evento)**: Algo que YA pasó (`PersonaNacida`). Es inmutable.
- **Intención (Comando)**: Algo que QUEREMOS que pase. Puede ser rechazado.

Definamos nuestro primer comando como un `record` simple:

```csharp
// "Quiero cambiar mi nombre a este nuevo valor"
public record CambiarNombre(Guid PersonaId, string NuevoNombre);
```

## 2. ¿Quién toma la decisión?
La clase `Persona` no solo sirve para "recordar" su edad. Su función principal es ser el **Guardián de las Reglas**. 

Para decidir si un comando es válido, Jhon necesita mirar su estado actual (el que reconstruyó en la sección anterior). Añadamos esta lógica a la clase `Persona`:

```csharp
public class Persona 
{
    // ... propiedades y constructor ...

    public object Decidir(CambiarNombre comando)
    {
        // 1. Validamos la regla: ¿Ya tiene un nombre?
        if (!string.IsNullOrEmpty(this.Nombre))
        {
            Console.WriteLine("REGLA: El nombre es inmutable. No se puede cambiar.");
            return null; // Decisión: No pasa nada
        }

        // 2. Si es válido, generamos un nuevo hito
        return new PersonaNacida(this.Id, comando.NuevoNombre, DateTime.Now);
    }
}
```

## 3. 🛠️ El Ciclo de Decisión en el Program.cs
Por ahora, vamos a ejecutar esto de la forma más sencilla posible en nuestra consola:

```csharp
// 1. Alguien envía un comando
var comando = new CambiarNombre(idPersona, "Jhon Sebastian");

// 2. Traemos a Jhon a la vida (reconstruimos su estado desde la lista)
var jhon = new Persona(biografia);

// 3. Jhon decide qué hacer con el comando
var resultado = jhon.Decidir(comando);

// 4. Si Jhon generó un hito nuevo, lo guardamos en su biografía
if (resultado != null)
{
    biografia.Add(resultado);
}
```

### El Descubrimiento
Acabas de ver el corazón del diseño basado en el dominio:
1. **Cargar**: Recuperas el pasado.
2. **Reconstruir**: Pones al objeto en su estado actual.
3. **Decidir**: El objeto valida el comando y, si es correcto, emite un **nuevo evento**.

---

[⬅️ Volver a la sección anterior](./04-refactorizando-el-motor.md)

[➡️ Siguiente sección: El flujo de vida (EventStream)](./06-el-flujo-de-vida.md)
