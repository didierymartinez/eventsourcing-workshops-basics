# 05 - Continuar la historia: Agregando eventos

Jhon ya tiene una biografía y sabe quién es. Pero la vida sigue, y queremos agregar nuevos capítulos a su historia. Hasta ahora solo hemos visto cómo rehidratar el pasado, pero ¿cómo agregamos nuevos hitos en el presente?

## 🎯 El Objetivo
Jhon sigue viviendo y experimentando nuevos hitos: bodas, mudanzas, nuevos trabajos. Nuestro objetivo es aprender cómo registrar estos nuevos sucesos a través de nuestro objeto `Persona`.

---

## 1. El Nuevo Hito
En Event Sourcing, todo lo que altera la historia se registra como un evento adicional. Vamos a definir un par de nuevos eventos para la vida de Jhon:

```csharp
public record PersonaCasada(Guid PersonaId, string NombrePareja);
public record PersonaMudada(Guid PersonaId, string NuevaCiudad);
```

## 2. Es quien autoriza agregar eventos a su biografía
Aquí hay un detalle clave (y muy sutil) en Event Sourcing: La clase `Persona` es la dueña de las **reglas** de su historia, pero sorprendentemente **NO almacena la lista físicamente**. Si te fijas, su constructor lee los eventos del pasado para rehidratarse, pero nunca guarda una referencia directa a la `Lista<object> biografia`.

Por lo tanto, a Jhon no le "insertamos" eventos a la fuerza; nosotros le pedimos que evalúe realizar una acción y él, a cambio, **emite un nuevo evento** como resultado. Alguien más en el sistema será el encargado de tomar ese evento y guardarlo en el baúl para el futuro.

Por ahora, como estamos en el "camino feliz", asumimos que Jhon acepta generar cualquier evento nuevo sin poner condiciones. Añadamos esta lógica a la clase `Persona`:

```csharp
public class Persona : AggregateRoot
{
    public string NombrePareja { get; private set; }
    // Asumimos que la propiedad Ciudad ya existe desde la refactorización anterior

    // ... propiedades, constructor y métodos Apply básicos ...

    public PersonaCasada RegistrarMatrimonio(string nombrePareja)
    {
        // Generamos un nuevo hito para la biografía
        return new PersonaCasada(this.Id, nombrePareja);
    }

    public PersonaMudada RegistrarMudanza(string nuevaCiudad)
    {
        // En una aplicación real, aquí validaríamos las reglas de negocio usando el estado rehidratado
        return new PersonaMudada(this.Id, nuevaCiudad);
    }

    // El motor actualiza el estado cuando ESTOS eventos ocurren en el pasado
    private void Apply(PersonaCasada c) => NombrePareja = c.NombrePareja;
    private void Apply(PersonaMudada m) => Ciudad = m.NuevaCiudad;
}
```

## 3. 🛠️ El Ciclo de Vida en el Program.cs
Vamos a ejecutar esto de la forma más sencilla posible en nuestra consola:

```csharp
// 1. Traemos a Jhon a la vida (rehidratamos su estado desde la lista)
var jhon = new Persona(biografia);

Console.WriteLine($"[ANTES] {jhon.Nombre} vive en {jhon.Ciudad} y su pareja es: {(string.IsNullOrEmpty(jhon.NombrePareja) ? "Nadie" : jhon.NombrePareja)}");

// 2. Le pedimos a Jhon que registre nuevos eventos en su vida
var eventoBoda = jhon.RegistrarMatrimonio("María");
var eventoMudanza = jhon.RegistrarMudanza("Madrid");

// 3. Guardamos estos nuevos hitos en su biografía oficial (esto sucede fúera de la clase Persona)
biografia.Add(eventoBoda);
biografia.Add(eventoMudanza);

// Para ver el resultado final, rehidratamos a Jhon DESDE CERO leyendo su biografía actualizada.
// ¡Esta es la clave! Persona no maneja la lista de su vida, alguien más (el sistema) lo hace.
var jhonActualizado = new Persona(biografia);

Console.WriteLine($"[DESPUÉS] {jhonActualizado.Nombre} ahora está casado con {jhonActualizado.NombrePareja} y vive en {jhonActualizado.Ciudad}.");
```

### El Descubrimiento
Acabas de ver el flujo básico para interactuar con el dominio:
1. **Cargar**: Recuperas el pasado (la biografía).
2. **Rehidratar**: Pones al objeto en su estado actual (haciendo el Replay de su historia).
3. **Actuar**: Le pides al objeto que realice una acción y genere un **nuevo evento**.

> [!TIP]
> Intuitivamente, cada acción que le pides a Jhon (como llamar a `RegistrarMatrimonio`) es una petición que le haces al sistema. En diseño de software, a esta intención de hacer algo se le llama **Comando**.
> Aquí vemos una regla de oro: **Los Comandos son los encargados de generar los Eventos** (siempre a través del Aggregate Root).

---

[⬅️ Volver a la sección anterior](./04-refactorizando-el-motor.md)

[➡️ Siguiente sección: El Command Handler](./06-el-command-handler.md)
