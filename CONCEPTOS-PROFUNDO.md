# 🧠 Auditoría Conceptual Profunda

> Profundización de **todo lo conceptual** del workshop. Para cada concepto: **definición precisa**, **bajo el capó** (el mecanismo real, no la metáfora), **el malentendido común** (la media-verdad que tienen muchos devs), **tradeoff / cuándo NO**, y **ancla Cosmos**. Validado con docs oficiales de Microsoft, WolverineFx y Marten. Objetivo: que ningún concepto quede como "magia".
>
> *Criterio de "entender" un concepto: poder explicar el mecanismo, predecir cuándo falla, y decir cuándo no usarlo.*

---

# I. Fundamentos del lenguaje

## 1. Valor vs Referencia · Identidad vs Igualdad · Inmutabilidad
**Definición precisa.** Un *value type* (`struct`, `int`, `record struct`) se copia por valor; un *reference type* (`class`, `record class`) se copia por referencia (se copia el puntero). **Identidad** = ser el mismo objeto (misma dirección, `ReferenceEquals`). **Igualdad** = tener el mismo valor (`Equals`/`==`). Son cosas distintas.

**Bajo el capó (records).** `record` (class) sigue siendo **tipo de referencia**, pero el compilador genera: `Equals(T)` por valor, `GetHashCode()` coherente, `ToString()`, una propiedad oculta `EqualityContract` (por eso dos records de tipos distintos nunca son iguales aunque tengan los mismos campos), y un método `<Clone>$` que implementa `with`. Las propiedades posicionales son `init` (asignables solo en construcción).

**Malentendido común.** *"record = inmutable"*. Falso en dos niveles: (1) puedes declarar `record` con `set` mutable; (2) la inmutabilidad es **superficial**: `record Orden(List<Item> Items)` permite `orden.Items.Add(...)`. El record no cambia su referencia a la lista, pero la lista muta. Para inmutabilidad real: tipos inmutables dentro (`IReadOnlyList`, `ImmutableArray`) y/o `readonly record struct`.

**Tradeoff.** `record class` da igualdad por valor pero sigue asignando en heap (GC). `readonly record struct` evita heap y es verdadero value object, pero se copia por valor (cuidado con structs grandes).

**Ancla Cosmos.** Comandos y eventos son `record` (`OrdenDeCompraCreada(...) : IPrivateEvent`). La igualdad por valor es lo que hace funcionar el `Then(eventoEsperado)` de los tests.

## 2. Sistema de tipos: interface vs abstract vs default methods, varianza, genéricos
**Definición precisa.** Una **interface** es un contrato; una **clase abstracta** es un contrato + estado + implementación parcial. Desde **C# 8** las interfaces **pueden tener implementación por defecto** (DIM) — la vieja regla "interface = sin código" es obsoleta. La diferencia real: **herencia única** (una sola clase base, puede traer estado) vs **implementación múltiple** (muchas interfaces, sin estado de instancia).

**Bajo el capó.** Las llamadas a métodos virtuales/de interfaz se resuelven por **vtable** (tabla de métodos virtuales) en runtime. Los genéricos en .NET son **reificados**: el JIT especializa por value types (sin boxing) y comparte código entre reference types. Las **restricciones** (`where T : AggregateRoot`) le dan al compilador información para llamar miembros sin casts ni reflexión.

**Varianza.** `interface ICommandHandler<in TCommand>` — el `in` la hace **contravariante** en el parámetro (puedes usar un handler de un tipo base donde se espera uno derivado). `out` = covariante (salidas). Entender esto explica por qué ciertas asignaciones de genéricos compilan y otras no.

**Malentendido común.** *"Usa interface para todo lo del constructor"*. No: a veces inyectas tipos concretos, `IOptions<T>`, o configuración. La interface se justifica cuando hay (o habrá) **más de una implementación** o necesitas un **seam** para testear. Interface por interface es ceremonia vacía.

**Ancla Cosmos.** `IEventStore`, `ICommandHandlerAsync<TCommand>`, `GetAggregateRootAsync<TAggregateRoot> where TAggregateRoot : AggregateRoot`. Los genéricos con restricción son lo que permite una sola firma para todos los agregados.

## 3. Polimorfismo y despacho (el más malentendido)
**Definición precisa.** Hay tres despachos distintos:
- **Estático / por sobrecarga** (compile-time): el compilador elige el método por el **tipo declarado** de los argumentos.
- **Dinámico / virtual** (`virtual`/`override`): se elige por el **tipo real** del objeto en runtime, vía vtable. Es polimorfismo "normal" de OOP.
- **`dynamic`** (DLR): se *suspende* la verificación de tipos; en runtime se construye un **call site** que resuelve el miembro por reflexión-cacheada.

**Bajo el capó de `dynamic`.** El compilador genera un `CallSite<T>` con un *binder* del **DLR (Dynamic Language Runtime)**. La primera llamada resuelve y cachea; las siguientes reusan el cache. Coste real: indirección, posible boxing, y **`RuntimeBinderException` si no existe el miembro** — un error que en código tipado sería de compilación.

**Malentendido común.** *"`((dynamic)this).Apply((dynamic)ev)` es la forma limpia de despachar eventos"*. Es limpia visualmente pero: pierde seguridad de tipos, es más lenta, y falla en runtime. La alternativa moderna y segura es el **`switch` con pattern matching** (C# 8+), que da OCP razonable **con** verificación de compilación; o generación de código. **Marten/Wolverine evitan `dynamic` en rutas calientes** justamente por esto (ver §6).

**Tradeoff.** `dynamic` es aceptable en código pedagógico o de baja frecuencia; en un Event Store que reproduce millones de eventos, no.

## 4. Delegados, closures y composición de funciones
**Definición precisa.** Un **delegado** (`Func<>`, `Action<>`) es un puntero a función con tipo. Una **closure** captura variables del entorno léxico. La **composición** encadena funciones: `f(g(x))`.

**Bajo el capó.** Una lambda que captura variables hace que el compilador genere una **clase de display** con esos campos; la lambda se vuelve un método de esa clase. Por eso una closure puede "recordar" estado — no es magia, es un objeto generado.

**Por qué importa (desarma el middleware).** Un **pipeline de middleware** *es* composición de funciones: cada middleware recibe un `next` (un delegado a "lo que sigue") y decide si llamarlo, cuándo, y qué hacer antes/después. Entender delegados convierte el middleware de "hechicería del framework" en "composición de funciones que yo podría escribir".

**Ancla Cosmos.** El `UnitOfWorkMiddleware` y `AutoApplyTransactions()` de Wolverine son middlewares que envuelven tu handler. Wolverine, además, **no los compone por reflexión en runtime: genera el código** que los encadena (ver §6).

---

# II. Ejecución y concurrencia

## 5. async / await (a fondo)
**Definición precisa.** `async/await` es **concurrencia sin paralelismo obligatorio**: libera el hilo durante operaciones de I/O en lugar de bloquearlo. No crea hilos; los **ahorra**.

**Bajo el capó.** El compilador transforma un método `async` en una **máquina de estados** (`struct` que implementa `IAsyncStateMachine`). Cada `await` es un punto de suspensión: si la tarea no terminó, se registra una **continuación** y el método retorna; el hilo queda libre. Al completarse, la continuación reanuda en `MoveNext()`. El **`ExecutionContext`** (que lleva `AsyncLocal`, p. ej. el TenantId) fluye a través de los awaits. El **`SynchronizationContext`**, si existe, se captura para reanudar "en el mismo sitio".

**El deadlock real vs starvation (matiz crítico).** El deadlock clásico de `.Result`/`.Wait()` ocurre cuando bloqueas un hilo que el `SynchronizationContext` capturado necesita para reanudar la continuación → espera mutua. Esto pasa en **UI (WPF/WinForms) y ASP.NET clásico**. Pero **ASP.NET Core y Azure Functions NO tienen SynchronizationContext** → ahí `.Result` **no hace deadlock, pero bloquea un hilo del pool** (thread starvation): bajo carga, agotas el pool y la app se cae igual. Cosmos es ASP.NET Core/Functions ⇒ el riesgo real es **starvation**, no deadlock. (Ambos se evitan igual: async todo el camino, nunca `.Result`/`.Wait()`.)

**Detalles de nivel.** `ConfigureAwait(false)` en librerías evita capturar el SyncContext (irrelevante en ASP.NET Core, vital en libs reutilizables). `async void` solo para event handlers (sus excepciones no se pueden `await` → crashean el proceso). `ValueTask` evita asignar un `Task` cuando el resultado suele estar disponible síncronamente (micro-optimización; no lo guardes ni lo awaitees dos veces). **`CancellationToken`** debe propagarse siempre (todas las APIs de Marten/Wolverine lo reciben).

**Malentendido común.** *"async hace mi código más rápido / corre en otro hilo"*. No: una sola operación async no es más rápida; mejora **escalabilidad** (más peticiones concurrentes con los mismos hilos). Y `await` no "lanza un hilo" — para I/O no hay hilo esperando en absoluto.

## 6. Reflexión vs Generación de código (el corazón anti-magia)
**Definición precisa.** **Reflexión** = inspeccionar tipos y llamar miembros en runtime (`Type`, `MethodInfo.Invoke`). Flexible pero lenta y opaca. **Generación de código** = producir código fuente/IL (en build con *source generators*, o en runtime) que hace lo mismo **explícitamente**, rápido e inspeccionable.

**Bajo el capó.** Un mediator clásico (estilo MediatR) usa reflexión/registro en el contenedor para encontrar y ejecutar el handler **en cada llamada**. **Wolverine genera código** que llama tu handler directamente, resuelve dependencias e inserta el middleware — y ese código se puede **previsualizar** (`dotnet run -- codegen write`, carpeta `./Internal/Generated/...`). La doc oficial lo dice explícitamente: *"Wolverine depends much more on runtime generated code than the IoC container tricks that many other .NET frameworks do"*, y recomienda **pre-generar tipos** para cold-start en serverless.

**Por qué esto mata la magia.** Si no entiendes cómo un comando llega a tu handler, **lees el código generado** y ahí está, sin misterio. Es el ejemplo máximo del principio "construye/inspecciona la magia antes de usarla".

**Ancla Cosmos.** Cosmos corre en Functions (.NET isolated); por eso usa `ExtensionDiscovery.ManualOnly` y se beneficia de pre-generar código para el cold-start.

---

# III. Principios de diseño

## 7. SOLID con rigor (no como póster)
- **SRP** — "una sola razón para cambiar". No es "una clase hace una cosa", es **un solo actor/motivo** que provoca su modificación.
- **OCP** — abierto a extensión, cerrado a modificación. **Cómo se logra de verdad:** polimorfismo/inyección, no `if/switch` que crece. (El `Apply` por evento cumple OCP; la cadena de `if` no.)
- **LSP** — un subtipo debe ser sustituible sin romper expectativas (precondiciones no más fuertes, postcondiciones no más débiles). Es sobre **contratos de comportamiento**, no solo firmas.
- **ISP** — interfaces pequeñas y específicas; no obligues a implementar lo que no se usa.
- **DIP** — depende de **abstracciones**, no de concreciones; y las abstracciones no dependen de detalles.

**La confusión que importa (DIP ≠ IoC ≠ DI ≠ Contenedor):**
- **DIP** = principio de diseño (depende de `IEventStore`, no de `MartenEventStore`).
- **IoC** = patrón general: algo externo controla el flujo/creación (el framework te llama a ti).
- **DI** = una técnica de IoC: pasar dependencias por constructor/método. **Funciona sin framework** ("pure DI": tú haces los `new` en un solo lugar).
- **Contenedor IoC** = herramienta que automatiza el grafo de objetos. **Opcional.**

**Bajo el capó del contenedor.** No es magia de C#: es **un diccionario (interfaz→cómo crear) + reflexión del constructor + recursión** para resolver dependencias anidadas. Construir un mini-contenedor de ~20 líneas lo demuestra.

**Trampa de nivel (captive dependency).** Inyectar un servicio `Scoped` dentro de uno `Singleton` lo "captura": el singleton conserva la primera instancia scoped para siempre → estado obsoleto/condiciones de carrera. El scope validation en desarrollo lo detecta.

**Service Locator = anti-patrón.** Resolver con `GetRequiredService` por todo el código oculta dependencias y rompe testabilidad. Solo es válido en el **Composition Root** (el único lugar donde se arma el grafo).

**Ancla Cosmos / matiz Wolverine.** La doc de Wolverine es tajante: *"Wolverine is trying really hard not to use an IoC container at runtime"*, prefiere **inyección por método** sobre constructor y evita registros lambda `Scoped`/`Transient` opacos. Su troubleshooting muestra el problema del scope cautivo: un servicio resuelto por un `IServiceScope` distinto recibe un `IMessageContext` con **TenantId DEFAULT** (mal). Esto es DIP/lifetimes en la vida real.

## 8. Acoplamiento y cohesión (la métrica detrás de todo)
Casi todos los patrones existen para **bajar acoplamiento** (qué tanto un cambio aquí obliga a cambiar allá) y **subir cohesión** (qué tan relacionado está lo que vive junto). Comando, Repositorio, DI, eventos: todos son palancas de esto. **Pero hay un contra-argumento de nivel**: demasiadas capas/abstracciones *también* dañan la mantenibilidad. La doc de Wolverine lo dice sin rodeos: prefieren "A-Frame Architecture" y **call stacks cortos** sobre Onion/Clean con muchas capas. Madurez = saber dónde está el punto medio.

---

# IV. Domain-Driven Design

## 9. Lenguaje Ubicuo
El código usa **las palabras exactas del experto del dominio** (comandos, eventos, métodos). No es estético: reduce la "deuda de traducción" entre negocio y código. **Clave que faltaba:** el lenguaje ubicuo es **por Bounded Context** — "factura" no significa lo mismo en Contabilidad que en Impuestos.

## 10. Bounded Context
**Definición precisa.** Un límite explícito (lingüístico, de modelo y de equipo) dentro del cual un modelo y su lenguaje son consistentes. Fuera de él, los mismos términos cambian de significado. El **Context Map** describe cómo se relacionan (cliente/proveedor, anticorrupción, etc.).

**Ancla Cosmos.** Cosmos **está organizado por Bounded Contexts** (ObligacionesPorPagar, Contabilidad, Impuestos, Terceros). Cada uno es un repo/despliegue; se comunican por **eventos de integración** (no llamadas síncronas). Sin este concepto, no se entiende la separación de planos ni el público/privado de eventos.

## 11. Aggregate y Aggregate Root
**Definición precisa.** Un **Aggregate** es un grupo de objetos tratados como una unidad para garantizar **invariantes**. La **Aggregate Root** es la única puerta de entrada: todo cambio pasa por ella, que protege las reglas. Es la **frontera de consistencia transaccional**.

**Reglas de oro (que suelen faltar):**
1. **Una transacción = un agregado.** Si una operación debe modificar dos agregados atómicamente, casi siempre el diseño está mal; usa eventos/consistencia eventual entre ellos.
2. **Dimensiónalo por invariantes, no por datos.** Agregados pequeños. El error clásico es un agregado gigante ("God aggregate") que serializa toda la concurrencia.
3. **Identidad estable** (un `Id`), y **modelo rico, no anémico**: la lógica vive en el agregado, no en "services" externos que manosean propiedades públicas.

**Ancla Cosmos.** `AggregateRoot` con `Id`, `Version`, `_uncommittedEvents`. El método `Aprobar()` protege la invariante "no aprobar dos veces".

## 12. Value Objects vs Entities
**Entity:** identidad propia que persiste aunque cambien sus atributos (una `Persona`). **Value Object:** definido por sus atributos, sin identidad, idealmente inmutable (un `Dinero(monto, moneda)`, una `Direccion`). Modelar como value object lo que no tiene identidad elimina bugs de igualdad y mutación. (`readonly record struct` es ideal para esto — conecta con §1.)

## 13. Domain Events vs Integration Events (distinción crítica)
- **Domain event:** un hecho relevante **dentro** del mismo bounded context/proceso. Suele procesarse en la misma transacción.
- **Integration event:** un hecho que **cruza** a otros bounded contexts, por el bus, con consistencia eventual. Debe ser estable (contrato público) y versionable.

**Malentendido común.** Tratar todo evento igual y publicarlo al bus. Filtrar datos internos en un evento de integración acopla contextos y filtra el modelo.

**Ancla Cosmos.** Implementado **literalmente**: `IPrivateEvent` (interno) vs `IPublicEvent` (integración), y el agregado los separa con `GetPrivateEvents()` / `GetPublicEvents()`. `WolverinePrivateEventSender` vs `WolverinePublicEventSender`.

---

# V. Event Sourcing & CQRS

## 14. Evento como fuente de verdad; estado como proyección
**Definición precisa.** En Event Sourcing la **fuente de verdad es el log append-only de eventos**, no una fila de estado. El estado actual es una **función de reducción** (fold) sobre los eventos: `estado = eventos.Aggregate(inicial, Apply)`. Ganas auditoría total, time-travel y proyecciones múltiples; pagas con complejidad y consultas indirectas.

**Bajo el capó (Marten).** Guarda eventos en PostgreSQL; al rehidratar, hace `AggregateStreamAsync<T>` (lee el stream y aplica los `Apply`). No usa `dynamic` en la ruta caliente: compila el aplicador.

**Malentendido común.** *"Event Sourcing = tener una tabla de auditoría"*. No: la auditoría es derivada; aquí el evento **es** el dato primario y el estado se descarta/recalcula.

## 15. Concurrencia optimista y versionado
**Definición precisa.** Dos comandos sobre el mismo stream a la vez ⇒ conflicto. El control **optimista** asume que rara vez chocan: cada agregado tiene una **`Version`**; al guardar, se verifica que la versión esperada siga vigente; si no, **excepción de concurrencia** y se reintenta (recargar→reaplicar). No bloquea (a diferencia del pesimista).

**Ancla Cosmos.** `AggregateRoot.Version`. Marten valida versión esperada al append.

## 16. Snapshots
Reproducir 1 millón de eventos para rehidratar es caro. Un **snapshot** guarda el estado en la versión N; al cargar, partes del snapshot y aplicas solo los eventos posteriores. Tradeoff: complejidad y consistencia del snapshot vs latencia de replay. No lo agregues antes de medir.

## 17. Proyecciones / read models y consistencia eventual
**Definición precisa.** Una **proyección** transforma el stream de eventos en un **read model** optimizado para consultar (tabla plana, documento). Puede ser **inline** (en la misma transacción, consistente) o **async** (un proceso la actualiza después → **consistencia eventual**: el lector puede ver datos viejos por milisegundos).

**Malentendido común.** Esperar consistencia inmediata en read models async. La consistencia eventual es un **trade** consciente (escala y desacople a cambio de latencia de propagación), no un bug.

## 18. CQRS
**Definición precisa.** Separar el modelo de **escritura** (comandos → agregados → eventos) del de **lectura** (queries → read models). No exige Event Sourcing (son ortogonales) ni bases separadas.

**Cuándo NO.** La mayoría de CRUDs simples **no** necesitan CQRS; añade complejidad. Se justifica con asimetría lectura/escritura, modelos de lectura múltiples, o alta escala.

---

# VI. Mensajería y fiabilidad

## 19. Command vs Event vs Query (semántica)
- **Command:** imperativo, **1 destinatario**, puede ser **rechazado**. ("AprobarOrden")
- **Event:** hecho pasado, **0..N suscriptores**, **irrechazable**, el emisor no sabe quién escucha. ("OrdenAprobada")
- **Query:** pide datos, **no muta**, devuelve resultado.

Confundirlos genera acoplamiento: un "evento" con un solo dueño obligado en realidad es un comando disfrazado.

## 20. Garantías de entrega: el mito del "exactly-once"
**Definición precisa.** En sistemas distribuidos, **exactly-once delivery no existe** de forma realista. Lo alcanzable es **at-least-once** (puede haber duplicados) + **idempotencia** del consumidor ⇒ efecto *exactly-once en el procesamiento*. La idempotencia se logra con dedup por id de mensaje, o haciendo la operación naturalmente idempotente.

**Ancla Cosmos/Wolverine.** El Outbox garantiza at-least-once; Wolverine también ofrece soporte de **idempotencia/inbox**. Tus handlers deben tolerar reentregas.

## 21. Outbox / Inbox
- **Outbox:** guarda el mensaje a publicar en la **misma transacción** que el cambio de negocio; un relay lo envía después con reintentos. Evita el "mensajero muerto" (cambio guardado pero aviso perdido, o viceversa).
- **Inbox:** registra mensajes **recibidos** para no procesarlos dos veces (dedup) y para procesar de forma durable.

**Ancla Cosmos.** `.IntegrateWithWolverine()` activa Inbox/Outbox durable sobre PostgreSQL; el transporte real es Azure Service Bus.

## 22. Orden: FIFO, particiones y sesiones
El orden global no escala. Se ordena **por partición/clave**. En Azure Service Bus, las **sesiones** (`SessionId`) garantizan orden FIFO dentro de una sesión. Cosmos usa **`SessionId = TenantId`**: los eventos de un tenant se procesan en orden estricto, sin entrelazar con otros tenants → orden donde importa, paralelismo entre tenants.

## 23. Sagas / procesos largos
Cuando un proceso de negocio cruza varios agregados/servicios y no puede ser una transacción ACID, se usa una **saga**: una máquina de estados que reacciona a eventos y emite comandos, con **compensaciones** en vez de rollback. (Onboarding de tenant en Cosmos es candidato natural.)

---

# VII. Multi-tenancy
**Definición precisa.** Una sola instancia sirve a múltiples clientes (tenants) con **aislamiento** de datos. Estrategias: base por tenant, esquema por tenant, o discriminador por fila. El **TenantId** debe fluir por todo el request (a menudo desde el JWT) sin acoplar el dominio.

**Ancla Cosmos.** `ITenantResolver`/`TenantResolver`, `InvokeForTenantAsync(tenantId, command)`. El TenantId viaja en el `ExecutionContext`/`IMessageContext`; resolver un servicio por un scope equivocado da TenantId DEFAULT (el bug que documenta Wolverine).

---

# VIII. Mapa de malentendidos comunes

| Lo que muchos creen | La realidad |
|---|---|
| `record` = inmutable | inmutabilidad **superficial**; las colecciones internas mutan |
| interface = sin código | desde C# 8 hay default interface methods |
| `dynamic` es la forma "limpia" de despachar | pierde tipos, es lento y falla en runtime; usa `switch`/codegen |
| async corre en otro hilo / es más rápido | no lanza hilo en I/O; mejora **escalabilidad**, no velocidad puntual |
| `.Result` solo causa deadlock | en ASP.NET Core/Functions causa **starvation** (igual de letal) |
| DI = el contenedor | DI funciona sin framework; el contenedor solo automatiza |
| el contenedor es magia de C# | es diccionario + reflexión + recursión |
| Event Sourcing = tabla de auditoría | el evento **es** la fuente de verdad; el estado es derivado |
| exactly-once delivery | at-least-once + idempotencia |
| read models siempre consistentes | proyecciones async ⇒ consistencia eventual (a propósito) |
| más capas = mejor arquitectura | demasiadas capas dañan; Wolverine prefiere call stacks cortos |
| evento y comando son casi lo mismo | comando: 1 dueño, rechazable; evento: N suscriptores, irrechazable |

---

# IX. Cómo usar este documento
- Es la **referencia conceptual** detrás del workshop. Cada sección práctica debería poder apuntar aquí para el "por qué profundo".
- Úsalo como **checklist de validación** en los Loops: si no puedes explicar el "bajo el capó" y el "cuándo NO" de un concepto, aún no lo dominas.
- Documentos hermanos: [REVISION-CRITICA.md](./REVISION-CRITICA.md) (qué arreglar en cada sección) · [ROADMAP.md](./ROADMAP.md) (currículo vivo) · [MAPA-CONCEPTUAL.md](./MAPA-CONCEPTUAL.md) · [WOLVERINE-RUTA-EXPERTO.md](./WOLVERINE-RUTA-EXPERTO.md).

*Fuentes de validación: docs oficiales de Microsoft (C# language reference, async), WolverineFx (Best Practices, Code Generation, Durability, Azure Service Bus) y Marten (Event Store, optimistic concurrency).*
