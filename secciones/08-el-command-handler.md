# 08 - El Command Handler

En la sección anterior vimos que podíamos instanciar a Jhon, pedirle que ejecute acciones y guardar esos eventos en nuestra lista. 

Todo este "flujo de trabajo" (Cargar -> Actuar -> Guardar) sucedía en un solo cajón desordenado: nuestro `Program.cs`.

## 📖 La Analogía del Biógrafo Oficial
Imagina que **Jhon (nuestra clase `Persona`)** toma constantemente decisiones sobre su vida: con quién se casa, a dónde se muda, o si tiene hijos. Él dicta lo que sucede (emite los eventos). Pero, ¿acaso Jhon anda literalmente cargando su propia enciclopedia de vida debajo del brazo? ¿Acaso él mismo va a los archivos a guardar los papeles cada vez que hace algo nuevo? 

No. Para eso tiene a su **Biógrafo Oficial (El Command Handler)**.

Cuando surge una nueva situación (un Comando), el Biógrafo:
1. Va a la hemeroteca y trae todos los apuntes pasados sobre la vida de Jhon (los eventos).
2. Se los presenta a Jhon para que recuerde exactamente quién es y dónde está (rehidratación).
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
    // Le pasamos el Archivero Central
    private readonly IEventStore _store;

    public PersonaCommandHandlers(IEventStore store)
    {
        _store = store;
    }

    // Nuestro Biógrafo anota un matrimonio
    public void TramitarBoda(MatrimonioSolicitado comando)
    {
        // 1. Cargar: Abrimos el cajón de Jhon en el archivero
        var stream = new EventStream<Persona>(_store, comando.PersonaId);
        var jhon = stream.Get();

        // 2. Actuar: Le pasamos la intención verificada al Agregado
        var eventoBoda = jhon.RegistrarMatrimonio(comando.NombrePareja);

        // 3. Guardar: Almacenar el nuevo hecho en el stream
        stream.Append(eventoBoda);
    }

    // Nuestro Biógrafo anota una mudanza
    public void ProcesarMudanza(MudanzaSolicitada comando)
    {
        var stream = new EventStream<Persona>(_store, comando.PersonaId);
        var jhon = stream.Get();
        
        var eventoMudanza = jhon.RegistrarMudanza(comando.NuevaCiudad);
        
        stream.Append(eventoMudanza);
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
// Para VER el nuevo estado, rehidratamos a Jhon desde la lista actualizada.
var jhonActualizado = new Persona(biografia);
Console.WriteLine($"Jhon está casado con: {jhonActualizado.NombrePareja}");
```

> [!NOTE]
> **El Aismlamiento Perfecto**
> Al pasarle el `IEventStore` al `CommandHandler`, hemos logrado desacoplar completamente las reglas de negocio de la infraestructura de almacenamiento.
> A la clase `PersonaCommandHandlers` no le importa si el `IEventStore` guarda los datos en RAM, en un disco duro, o en una red distribuida. Simplemente le pide el Stream, actúa y guarda.

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
    private readonly IEventStore _store;

    public MatrimonioSolicitadoHandler(IEventStore store) => _store = store;

    public void Handle(MatrimonioSolicitado comando)
    {
        var stream = new EventStream<Persona>(_store, comando.PersonaId);
        var jhon = stream.Get();
        
        var eventoBoda = jhon.RegistrarMatrimonio(comando.NombrePareja);
        
        stream.Append(eventoBoda);
    }
}

// 3. Otro Biógrafo especializado exclusivamente en mudanzas
public class MudanzaSolicitadaHandler : ICommandHandler<MudanzaSolicitada>
{
    private readonly IEventStore _store;

    public MudanzaSolicitadaHandler(IEventStore store) => _store = store;

    public void Handle(MudanzaSolicitada comando)
    {
        var stream = new EventStream<Persona>(_store, comando.PersonaId);
        var jhon = stream.Get();
        
        var eventoMudanza = jhon.RegistrarMudanza(comando.NuevaCiudad);
        
        stream.Append(eventoMudanza);
    }
}
```

Al heredar de la interfaz `ICommandHandler<>` por cada comando, nuestra clase plural adquiere poderes arquitectónicos formales.

## 2. Un Program.cs Elegante
Ahora, el código de "arranque" de nuestra aplicación (ya sea una API REST o una consola) solo despacha el comando al Handler correcto:

```csharp
// Llenamos el formulario de la primera acción
var comandoBoda = new MatrimonioSolicitado(idPersona, "María");
var handlerBoda = new MatrimonioSolicitadoHandler(store); // Inyectamos el EventStore
handlerBoda.Handle(comandoBoda);

// Llenamos otro formulario para una segunda acción
var comandoMudanza = new MudanzaSolicitada(idPersona, "Madrid");
var handlerMudanza = new MudanzaSolicitadaHandler(store);
handlerMudanza.Handle(comandoMudanza);

// Para probar que los Handlers realmente hicieron su trabajo
var streamFinal = new EventStream<Persona>(store, idPersona);
var jhonFinal = streamFinal.Get();

Console.WriteLine($"[VERIFICACIÓN] {jhonFinal.Nombre} ahora está casado con {jhonFinal.NombrePareja} y vive en {jhonFinal.Ciudad}.");
```

---

### El Descubrimiento
Acabamos de abstraer la intermediación de acciones únicas. Nuestro **Aggregate Root** (`Persona`) cuida las reglas vitales puras, mientras que el **Command Handler** (`MudanzaSolicitadaHandler`) se ocupa del trabajo sucio de la arquitectura: conseguir las herramientas (`EventStream`), despertar a la persona y guardar sus memorias para ese evento particular.

¡Felicidades! Acabas de construir desde cero el flujo arquitectónico completo y profesional de Event Sourcing en Memoria.

Pero todavía hay una fragilidad enorme de la que tenemos que hacernos cargo: si el servidor se apaga, todo desaparece. En la siguiente sección, enfrentaremos el mundo real: persistencia e I/O.

---

[⬅️ Volver a la sección anterior](./07-decidir-el-futuro.md)

[➡️ Siguiente sección: El Tiempo de Espera (I/O y Task)](./09-el-riesgo-de-olvidar.md)
