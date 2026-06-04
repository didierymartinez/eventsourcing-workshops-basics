# 04 - Separar la Intención de la Ejecución: El Patrón Comando

Si alguna vez has escrito código dentro del "botón Guardar" (`OnClick`) de un formulario de escritorio, o has puesto consultas a base de datos del tipo `dbContext.Users.Add(...)` directamente dentro de un Controlador HTTP en una web, has violado el principio más sagrado de la arquitectura limpia: responsabilidades mezcladas.

La capa de presentación (la UI o el Controlador de la API) no debe saber **CÓMO** se hacen las cosas. Solo debe saber **QUÉ** se quiere hacer.

## 📨 El Formulario de Intención (Command)

Un Comando es un simple objeto inmutable diseñado para transportar datos. Simboliza una intención imperativa generada por un usuario del sistema.

```csharp
// Un Comando siempre obedece a un verbo imperativo ("Haz algo")
public record RegistrarMatrimonio(Guid PersonaId, string NombrePareja);
```
> [!NOTE]
> Observa que es un `record` puro. No tiene métodos, no valida reglas de la iglesia, no se conecta a base de datos. Es literal y metafóricamente **un formulario en papel** rellenado.

## 🤵 El Empleado Asignado (Command Handler)

Una vez que el Controlador Web recibe el formulario del usuario, se lo entrega a un experto. A este obrero lo conocemos como el **Manejador del Comando (Command Handler)**.

```csharp
// Solo hace y sabe hacer una cosa: Casar Personas
public class RegistrarMatrimonioHandler
{
    private readonly IPersonaRepository _repo;

    public RegistrarMatrimonioHandler(IPersonaRepository repo)
    {
        _repo = repo;
    }

    // El punto de entrada único de la operación
    public void Handle(RegistrarMatrimonio comando)
    {
        // 1. Obtiene la entidad del mundo real
        var persona = _repo.Get(comando.PersonaId);

        // 2. Ejecuta la lógica central puramente sobre la entidad
        persona.Casar(comando.NombrePareja);

        // 3. Guarda los cambios de nuevo en la infraestructura
        _repo.Save(persona);
    }
}
```

## 🔌 El Controlador Desacoplado

¿Qué logra esta división? Que tu aplicación sea inmortal frente a cambios de tecnología de presentación. 
Tu lógica de negocio está salvaguardada en un bloque de código puro.

Mira cómo queda tu Controlador (o tu API):

```csharp
[ApiController]
public class MatrimonioController : ControllerBase
{
    private readonly RegistrarMatrimonioHandler _handler;

    [HttpPost("/api/matrimonios")]
    public IActionResult CrearMatrimonio([FromBody] RegistrarMatrimonio comando)
    {
        // La API no sabe cómo se casa a la gente ni dónde se guardan.
        // Solo delega el formulario completo al obrero.
        _handler.Handle(comando);
        return Ok("Felicidades por la boda!");
    }
}
```

### El Superpoder en Event Sourcing

En una aplicación clásica (CRUD), a veces los Programadores saltan este paso por "pereza" y escriben el CRUD en el controlador.

Pero en **Event Sourcing y CQRS**, el patrón Comando no es opcional, es el cimiento absoluto. Toda tu aplicación se tratará de enviar comandos, que generarán validaciones, que a su vez emitirán Eventos para registrar lo sucedido. Dominar esta mensajería es el primer paso.

---
[⬅️ Volver a la Fase anterior](./03-polimorfismo-y-dynamic-dispatching.md) | [➡️ Siguiente sección: El Patrón Repositorio](./05-el-patron-repositorio.md)
