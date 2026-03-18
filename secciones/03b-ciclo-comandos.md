# 03b - El ciclo de vida de un cambio

Ya sabemos cómo reconstruir a Juan leyendo su biografía. Pero, ¿qué pasa cuando Juan quiere hacer algo nuevo? ¿Cómo pasamos de una "intención" a un "hecho registrado"?

## 🎯 El Objetivo
Juan se quiere mudar de ciudad. Sin embargo, tenemos una regla de negocio: **Juan no puede mudarse si no ha nacido todavía.**

Tu reto es implementar el ciclo completo de un comando para asegurar que esta regla se cumpla.

---

## 1. La intención (El Comando)
A diferencia de los hechos (que están en pasado), los **Comandos** representan una intención de futuro. Suelen llamarse en imperativo: `Mudarse`.

```csharp
public record Mudarse(Guid PersonaId, string NuevaCiudad);
```

---

## 2. El Ciclo de Vida: Las 5 Estaciones
Para procesar este comando, siempre seguiremos los mismos pasos que viste en la teoría:

1.  **Localizar**: Buscar el Stream (la lista `biografia`) usando el `PersonaId` del comando.
2.  **Cargar y Reconstruir**: Crear una instancia de `Persona` (nuestro Aggregate Root) pasándole todos los hechos del pasado.
3.  **Ejecutar Lógica**: Llamar a un método dentro de la clase `Persona` para que ella decida si la mudanza es válida.
4.  **Generar el Hecho**: Si todo está bien, la clase `Persona` devuelve un nuevo hecho: `MudanzaRealizada`.
5.  **Persistir**: Añadir ese nuevo hecho al final de la lista `biografia`.

---

## 3. 🛠️ ¡Manos a la obra! (Tu Turno)

Actualiza tu clase `Persona` en `Program.cs` para que sea capaz de procesar mudanzas. 

### Paso A: Definir el nuevo hecho
```csharp
public record MudanzaRealizada(Guid PersonaId, string Ciudad);
```

### Paso B: Implementar la lógica de decisión
Modifica la clase `Persona` para que tenga un método `DecidirMudanza`. **Importante:** La mudanza solo es válida si la persona ya tiene un nombre (ha nacido).

```csharp
public class Persona 
{
    // ... estado anterior ...
    public string CiudadActual { get; private set; }

    public MudanzaRealizada DecidirMudanza(string nuevaCiudad)
    {
        if (string.IsNullOrEmpty(Nombre)) 
            throw new Exception("¡No puedes mudarte si no has nacido!");

        return new MudanzaRealizada(Id, nuevaCiudad);
    }

    private void Aplicar(object hito)
    {
        // ... otros hilos ...
        if (hito is MudanzaRealizada m) { CiudadActual = m.Ciudad; }
    }
}
```

### Paso C: Procesa el comando
Ahora, intenta ejecutar este flujo en tu `Main`:

```csharp
// 1. Recibimos el comando
var comando = new Mudarse(idPersona, "Medellín");

// 2. Reconstruimos el agregado (Load & Replay)
var juan = new Persona(biografia);

// 3. El Agregado decide (Execute)
var nuevoHecho = juan.DecidirMudanza(comando.NuevaCiudad);

// 4. Guardamos el resultado del futuro (Append)
biografia.Add(nuevoHecho);

Console.WriteLine($"Juan ahora vive en: {juan.CiudadActual} (Espera... ¡falta algo!)");
```

---

## ❓ El gran dilema
Si ejecutas el código anterior, verás que `juan.CiudadActual` sigue estando vacía o desactualizada inmediatamente después del `Add`.

**¿Por qué?** Porque acabas de guardar el hecho en el diario, pero ¡olvidaste "vivirlo"! Para que el objeto `juan` se entere del cambio, también tiene que `Aplicar` ese nuevo hecho sobre sí mismo.

Este ciclo de: **Recibir -> Reconstruir -> Decidir -> Guardar** es el corazón de Event Sourcing. 

---

[⬅️ Volver a la sección anterior](./03-es-en-memoria.md)

[➡️ Siguiente sección: El gran problema](./04-el-problema.md)
