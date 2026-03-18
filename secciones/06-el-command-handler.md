# 06 - El Command Handler

En la sección anterior vimos que podíamos instanciar a Jhon, pedirle que ejecute acciones y guardar esos eventos en nuestra lista. 

Todo este "flujo de trabajo" (Cargar -> Actuar -> Guardar) sucedía en un solo cajón desordenado: nuestro `Program.cs`.

## 📖 La Analogía del Biógrafo Oficial
Imagina que **Jhon (nuestra clase `Persona`)** toma constantemente decisiones sobre su vida: con quién se casa, a dónde se muda, o si tiene hijos. Él dicta lo que sucede (emite los eventos). Pero, ¿acaso Jhon anda literalmente cargando su propia enciclopedia de vida debajo del brazo? ¿Acaso él mismo va a los archivos a guardar los papeles cada vez que hace algo nuevo? 

No. Para eso tiene a su **Biógrafo Oficial (El Command Handler)**.

Cuando surge una nueva situación (un Comando), el Biógrafo:
1. Va a la hemeroteca y trae todos los apuntes pasados sobre la vida de Jhon (los eventos).
2. Se los presenta a Jhon para que recuerde exactamente quién es y dónde está (reconstrucción).
3. Le presenta la nueva situación o decisión.
4. Escucha el veredicto de Jhon (el nuevo evento emitido) y **el Biógrafo es quien va a archivarlo al estante correcto**.

Jhon se concentra puramente en "vivir" y hacer cumplir sus propias reglas. El Biógrafo se encarga de todo el trabajo logístico de almacenamiento y recuperación de la biografía.

---

## 1. Implementando el Command Handler
Un **Command Handler** es una clase puramente técnica. Su única responsabilidad es recibir un Comando específico, buscar el Agregado correcto (Jhon), pedirle que ejecute la acción, y guardar el resultado. No tiene "lógica de negocio", no coordina múltiples sistemas ni flujos complejos; simplemente atiende una única petición.

Vamos a abstraer el `Program.cs` moviendo este trabajo a una clase nueva. Además, para formalizar verdaderamente estas "peticiones", vamos a empaquetarlas en pequeños **Comandos** (como llenar un formulario de trámite).

### 🛠️ Paso 1: Los Formularios (Comandos)

```csharp
// 1. Definimos los "Formularios de Petición" (Comandos)
public record MatrimonioSolicitado(Guid PersonaId, string NombrePareja);
public record MudanzaSolicitada(Guid PersonaId, string NuevaCiudad);
```

> [!NOTE]
> **¿Por qué usamos `record` (en lugar de `class`) para los Comandos?**
> A diferencia de una clase normal, un `record` en C# es **Inmutable** por defecto. Una vez que un usuario u otro sistema genera una intención ("Mudarse a Madrid") de tipo Comando, ese paquete de datos viaja por el sistema y **no debe ser alterado por nadie en tránsito** hasta que llega a su Handler final. Operan como un puro y simple vehículo de transporte de datos (DTO).

### 🛠️ Paso 2: El Trabajo del Biógrafo (El Handler Simple)

Primero, construyamos a nuestro Biógrafo combinando todos los Handlers relacionados con nuestra `Persona` en una única clase, valiéndonos de la sobrecarga de métodos:

```csharp
// 2. El Manejador que actúa como nuestro Biógrafo Oficial
public class PersonaCommandHandlers
{
    // Por ahora le pasamos la "base de datos" por el constructor
    private readonly List<object> _biografia;

    public PersonaCommandHandlers(List<object> biografia)
    {
        _biografia = biografia;
    }

    // Nuestro Biógrafo anota un matrimonio con el método que se nos ocurra
    public void TramitarBoda(MatrimonioSolicitado comando)
    {
        // 1. Cargar: Recuperar el estado actual de Jhon
        var jhon = new Persona(_biografia);

        // 2. Actuar: Le pasamos la intención verificada al Agregado
        var eventoBoda = jhon.RegistrarMatrimonio(comando.NombrePareja);

        // 3. Guardar: Almacenar los nuevos hechos
        _biografia.Add(eventoBoda);
    }

    // Nuestro Biógrafo anota una mudanza
    public void ProcesarMudanza(MudanzaSolicitada comando)
    {
        var jhon = new Persona(_biografia);
        var eventoMudanza = jhon.RegistrarMudanza(comando.NuevaCiudad);
        _biografia.Add(eventoMudanza);
    }
}

// ----------------------------------------------------
// ¿Cómo se usa esta clase en nuestro programa principal?
// ----------------------------------------------------

var biografo = new PersonaCommandHandlers(biografia);

var comandoBoda = new MatrimonioSolicitado(idPersona, "María");
// Llamamos al método por su nombre específico
biografo.TramitarBoda(comandoBoda); 

// ¡Ahora la _biografia tiene el nuevo evento!
// Para VER el nuevo estado, reconstruimos a Jhon desde la lista actualizada.
var jhonActualizado = new Persona(biografia);
Console.WriteLine($"Jhon está casado con: {jhonActualizado.NombrePareja}");
```

> [!NOTE]
> **¿Cómo modifica el Biógrafo la lista "real"?**
> En C#, `List<object>` es un **tipo por referencia**. Cuando haces `new PersonaCommandHandlers(biografia)`, NO se copia la lista; se pasa un "apuntador" a la misma lista en memoria.
>
> Esto significa que cuando el Handler ejecuta `_biografia.Add(eventoBoda)`, está añadiendo el evento a **la misma lista** que vive en el `Program.cs`. Son el mismo objeto físico en RAM.
>
> Por eso al reconstruir a Jhon después con `new Persona(biografia)`, la lista ya tiene los nuevos eventos y su estado refleja las últimas acciones.
>
> Sin embargo, este mecanismo de "compartir lista" es muy frágil y difícil de escalar. ¿Qué pasa cuando tenemos múltiples personas, o cuando la aplicación se reinicia? Eso es exactamente lo que resolveremos en la siguiente sección: el **Event Store**.

### 🛠️ Paso 3: Abstraer al Biógrafo (La Interfaz Genérica)

Lo anterior funciona, pero oculta un problema de diseño grave a medida que crecemos: **el acoplamiento**. Si mañana creamos un controlador de API Web, este tendría que saberse de memoria que para casar a Jhon debe llamar a `TramitarBoda` y para mudarlo a `ProcesarMudanza`. 

El emisor del Comando (la API o la Interfaz de Usuario) no debería tener que saber cómo el Biógrafo nombra sus rutinas de trabajo interno. Para lograr esta **Inversión de Dependencias**, definimos un **contrato genérico estandarizado**. 

Así, quien envía el comando solo necesita buscar "alguien" que cumpla el contrato `ICommandHandler<T>` y llamar a su único método central e inequívoco: `Handle`.

Además, en estándares modernos de la industria, **se crea una clase Handler diferente por cada Comando** para cumplir a raja tabla con el Principio de Responsabilidad Única (SRP). En lugar de tener un "súper biógrafo" gigante, tenemos especialistas:

```csharp
// 1. Definimos una Interfaz Universal para cualquier Handler
public interface ICommandHandler<in TCommand>
{
    void Handle(TCommand comando);
}

// 2. Un Biógrafo especializado exclusivamente en matrimonios
public class MatrimonioSolicitadoHandler : ICommandHandler<MatrimonioSolicitado>
{
    private readonly List<object> _biografia;

    public MatrimonioSolicitadoHandler(List<object> biografia) => _biografia = biografia;

    public void Handle(MatrimonioSolicitado comando)
    {
        var jhon = new Persona(_biografia);
        var eventoBoda = jhon.RegistrarMatrimonio(comando.NombrePareja);
        _biografia.Add(eventoBoda);
    }
}

// 3. Otro Biógrafo especializado exclusivamente en mudanzas
public class MudanzaSolicitadaHandler : ICommandHandler<MudanzaSolicitada>
{
    private readonly List<object> _biografia;

    public MudanzaSolicitadaHandler(List<object> biografia) => _biografia = biografia;

    public void Handle(MudanzaSolicitada comando)
    {
        var jhon = new Persona(_biografia);
        var eventoMudanza = jhon.RegistrarMudanza(comando.NuevaCiudad);
        _biografia.Add(eventoMudanza);
    }
}
```

Al heredar de la interfaz `ICommandHandler<>` por cada comando, nuestra clase plural adquiere poderes arquitectónicos formales.

## 2. Un Program.cs Elegante
Ahora, el código de "arranque" de nuestra aplicación (ya sea una API REST o una consola) solo despacha el comando al Handler correcto:

```csharp
// Llenamos el formulario de la primera acción
var comandoBoda = new MatrimonioSolicitado(idPersona, "María");
var handlerBoda = new MatrimonioSolicitadoHandler(biografia);
handlerBoda.Handle(comandoBoda);

// Llenamos otro formulario para una segunda acción
var comandoMudanza = new MudanzaSolicitada(idPersona, "Madrid");
var handlerMudanza = new MudanzaSolicitadaHandler(biografia);
handlerMudanza.Handle(comandoMudanza);

// Para probar que los Handlers realmente hicieron su trabajo, reconstruimos a Jhon desde el historial
var jhonFinal = new Persona(biografia);

Console.WriteLine($"[VERIFICACIÓN] {jhonFinal.Nombre} ahora está casado con {jhonFinal.NombrePareja} y vive en {jhonFinal.Ciudad}.");
```

---

### El Descubrimiento
Acabamos de abstraer la intermediación de acciones únicas. Nuestro **Aggregate Root** (`Persona`) cuida las reglas vitales puras, mientras que el **Command Handler** (`PersonaCommandHandlers`) se ocupa del trabajo sucio de infraestructura: despertar a la persona y guardar sus memorias para ese evento particular.

Pero espera... Sigue habiendo un cable suelto. El Handler depende directamente de una cruda `List<object>`. ¿Qué pasa si Jhon tiene 1 millón de eventos? ¿El Handler tiene que pasarlos todos por valor? ¿Qué pasa si Jhon no está solo?

Es hora de formalizar un almacén oficial: El Event Store.

---

[⬅️ Volver a la sección anterior](./05-decidir-el-futuro.md)

[➡️ Siguiente sección: Abstracción del Almacén](./07-el-almacen-en-memoria.md)
