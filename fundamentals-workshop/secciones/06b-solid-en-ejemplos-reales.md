# 06b - SOLID, pero con el código que ya escribiste

> 🎯 **Hacia dónde va:** no vas a "aprender SOLID" como cinco reglas sueltas. Vas a **ponerle nombre** a decisiones que ya tomaste en las secciones 04-06 (un handler por comando, depender de `IEventStore` y no de Postgres…). Reconocerlas es lo que te deja defender un diseño en Cosmos sin recitar siglas.

> 🧬 **Sección de consolidación.** Aquí no hay código nuevo: revisitamos el que ya construiste (Comando §04, Repositorio §05, DI §06) y descubrimos que **ya era SOLID**. Ese es el orden correcto: primero lo necesitaste, ahora le pones el nombre.

SOLID son cinco principios de diseño orientado a objetos (Robert C. Martin). La trampa de la mayoría de tutoriales es enseñarlos con ejemplos de juguete (`Animal`, `Shape`) que nunca usarás. Aquí cada uno aparece en **tu** código.

## S — Responsabilidad Única (SRP)
*"Una clase debe tener una sola razón para cambiar."*

En §04 **no** hiciste un "súper biógrafo" con `TramitarBoda`, `ProcesarMudanza`, `RegistrarHijo`… Hiciste **un handler por comando**: `RegistrarMatrimonioHandler`, `RegistrarMudanzaHandler`. Cada uno cambia por una sola razón (su regla de negocio). Eso **es** SRP.

```csharp
// ❌ Viola SRP: cambia por matrimonios, por mudanzas, por hijos…
public class PersonaService { void Casar(...){} void Mudar(...){} void RegistrarHijo(...){} }

// ✅ SRP: una razón de cambio cada uno
public class RegistrarMatrimonioHandler : ICommandHandler<RegistrarMatrimonio> { ... }
public class RegistrarMudanzaHandler    : ICommandHandler<RegistrarMudanza>    { ... }
```

## O — Abierto/Cerrado (OCP)
*"Abierto a extensión, cerrado a modificación."*

Cuando mañana aparezca el comando `RegistrarDivorcio`, ¿qué tocas? **Nada existente**: creas `RegistrarDivorcioHandler` y listo. No abres `RegistrarMatrimonioHandler` para meterle un `if`. El sistema se **extiende** (clase nueva) sin **modificar** lo que ya funciona. Eso es OCP — y es justo lo que hace posible el patrón Comando + un handler por comando.

## L — Sustitución de Liskov (LSP)
*"Un subtipo debe poder reemplazar a su tipo base sin romper nada."*

En §05/§06 tu handler pide un `IEventStore`. Le puedes pasar `InMemoryEventStore` (en tests) o `PostgresEventStore` (en producción) y **el handler no se entera ni se rompe**. Cualquier implementación de `IEventStore` es sustituible por otra. Si una implementación rompiera el contrato (p. ej. `Get` que a veces lanza en vez de devolver `null`), violaría LSP y tus tests mentirían.

## I — Segregación de Interfaces (ISP)
*"Muchas interfaces específicas son mejores que una general y gorda."*

`ICommandHandler<T>` tiene **un** método: `Handle`. No es un `IServicio` con 20 métodos donde cada implementación deja 18 vacíos. Interfaces pequeñas y enfocadas: quien las implementa cumple **solo** lo que de verdad usa. En Event Sourcing esto se ve también en separar `IEvent` privado vs público (lo verás en §09).

## D — Inversión de Dependencias (DIP)
*"Depende de abstracciones, no de implementaciones concretas."*

Es la columna vertebral de la sección §06. Tu handler depende de `IEventStore` (abstracción), **no** de `PostgresEventStore` (concreto). Por eso pudiste testear sin base de datos. La flecha de dependencia apunta a la **interfaz**, y tanto el handler como Postgres dependen de ella.

```csharp
public RegistrarMatrimonioHandler(IEventStore store)  // ✅ depende de la abstracción
```

> [!WARNING]
> 🧠 **No confundas DIP con DI.** *DIP* (la "D" de SOLID) es el **principio**: "depende de interfaces". *Inyección de Dependencias* (§06) es **una técnica** para cumplirlo: pasar la dependencia por el constructor. Y el *contenedor* de DI es solo una **herramienta** que automatiza esa técnica. Principio → técnica → herramienta. Puedes cumplir DIP sin contenedor (los `new` manuales de §06 ya lo cumplían).

> [!IMPORTANT]
> 🪐 **Ancla Cosmos.** `Cosmos.BuildingBlocks` es SOLID hecho plantilla: `ICommandHandlerAsync<T>` (SRP + ISP), un handler por comando (OCP), `IEventStore`/`MartenEventStore` intercambiables (LSP + DIP). Cuando leas la plantilla (§27) no verás "código que aplica SOLID": verás código **estructurado** así de fábrica, y ya sabrás por qué.

---

### El Descubrimiento
SOLID no es una checklist que aplicas al final. Es el **nombre** de las buenas decisiones que el patrón Comando, el Repositorio y la DI te empujaron a tomar de forma natural. Si algún día un diseño "huele mal", casi siempre es porque rompe uno de estos cinco — y ahora puedes señalar cuál.

---

[⬅️ Volver a la sección anterior](./06-inyeccion-de-dependencias.md) | [➡️ Siguiente Fase: Lenguaje Ubicuo (DDD)](./07-lenguaje-ubicuo.md)
