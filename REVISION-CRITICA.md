# 🔬 Revisión Crítica y Replanteamiento del Workshop

> Evaluación crítica avanzada del `fundamentals-workshop` y del workshop principal, contra documentación oficial (Microsoft C#, WolverineFx, Marten) y buenas prácticas. Objetivo: subir el listón de "uso de plantillas/patrones sin entender" a **comprensión de primeros principios**. Filosofía: *si parece magia, es que falta una sección.*

---

## A. Veredicto general

El material es **sólido y bien pensado**: las metáforas (el Diario de Jhon, el Recepcionista, el Cocinero congelado) construyen intuición real, y ya hay conciencia de OCP y DIP. Para un nivel básico-medio, cumple.

Pero para el objetivo declarado —llevar C# + Wolverine a nivel experto y evitar el "lo uso sin comprender"— hay tres tipos de problema:

1. **La magia no se desarma.** Varias secciones (DI, dynamic dispatch, async) *describen* el comportamiento pero no muestran el mecanismo por debajo. Un dev sale sabiendo *usar* el contenedor, no *cómo* funciona — exactamente lo que quieres evitar.
2. **Afirmaciones sin tradeoffs.** Se presentan soluciones (`dynamic`, Repository, "todo en constructor es interface") como verdades absolutas, sin el costo ni el "cuándo NO".
3. **Faltan conceptos vitales** que son justo los que separan al que copia plantillas del que entiende (SOLID explícito, genéricos, delegados/composición, reflexión vs generación de código, bounded context, unit of work, testing, idempotencia, concurrencia optimista).

A continuación, lo concreto.

---

## B. Hallazgos por sección (con severidad)

### 01 — Records e Inmutabilidad · 🟡 Medio
- **Falta el matiz clave:** la inmutabilidad de un `record` es **superficial (shallow)**. Un `record Persona(List<string> Hijos)` permite `persona.Hijos.Add(...)` — el record es inmutable, su contenido no. Sin esto, un dev cree que "record = inmutable garantizado" y se quema.
- `record` (class) **sigue siendo tipo de referencia** con igualdad por valor; no es lo mismo que `record struct` (C# 10) ni `readonly record struct` (verdadero value object inmutable). Vale la pena nombrarlo.
- `with` hace **copia superficial** (mismo caveat de colecciones).
- **Corrección de enfoque:** enseñar "inmutabilidad ≠ profundidad" y cuándo usar `readonly record struct`.

### 02 — Interfaces vs Clases Abstractas · 🟠 Alto (desactualizado)
- **Dato obsoleto:** desde **C# 8 (2019)** las interfaces **sí pueden tener implementación** (default interface methods). La premisa "la interfaz no tiene código" ya no es cierta y la diferencia real es otra: estado/herencia única vs contrato múltiple.
- Regla demasiado dogmática: *"todo lo que vaya en un constructor debería ser una interfaz"*. No siempre — a veces inyectas tipos concretos, `Options<T>`, o `record` de configuración.
- **Falta:** principio **composición sobre herencia** e **ISP** (Interface Segregation). La elección abstract-vs-interface es realmente una decisión de diseño, no una tabla.

### 03 — Polimorfismo y `dynamic` dispatch · 🔴 Crítico — ✅ REPLANTEADO (04/06)
> Reescrita: ahora presenta `switch` con pattern matching como la opción recomendada (segura en compilación), `dynamic` como alternativa **con sus costos citando la doc oficial de Microsoft** (bypass de type-checking, overload resolution en runtime vía DLR, `RuntimeBinderException`), y cierra con "producción genera el dispatch (Marten/codegen, §22)". Título actualizado.
- **El problema más serio del workshop.** Se promueve `((dynamic)this).Apply((dynamic)ev)` como "la solución limpia" sin **ninguno** de sus costos:
  - `dynamic` usa el **DLR (Dynamic Language Runtime)**: tiene overhead real por call-site, hace boxing y **rompe en runtime** (`RuntimeBinderException`) si no existe un `Apply` para ese tipo — sin error de compilación.
  - Pierdes seguridad de tipos, refactors seguros y análisis estático.
- **Falta lo importante:** las alternativas modernas y por qué la industria las prefiere:
  - **`switch` con pattern matching** (C# 8+): despacho con seguridad de compilación y mucho mejor rendimiento.
  - **Generación de código / expresiones**: frameworks de producción como **Marten y Wolverine evitan `dynamic` en las rutas calientes** precisamente por el costo; generan código o usan expression trees. (Ver sección nueva propuesta "Reflexión vs Codegen".)
- **Corrección de enfoque:** mostrar `dynamic` como recurso pedagógico/rápido, pero enseñar el `switch` tipado como lo correcto en producción, con el tradeoff explícito.

### 04 — Patrón Comando · 🟡 Medio
- Confunde dos cosas con el mismo nombre: el **Command Pattern de GoF** (objeto que encapsula una acción con `Execute()`) y el **comando de mensajería/CQRS** (DTO de intención que un handler procesa). Son distintos; conviene aclararlo.
- **Falta:** dónde va la **validación** del comando (no en el record), y la distinción **Command vs Query** (se posterga a CQRS pero debería sembrarse aquí).

### 05 — Patrón Repositorio · 🟡 Medio
- Honesto al admitir la "herejía" de Marten 👏. Pero **falta el contrapunto crítico:** poner un Repository genérico encima de un ORM que **ya es** Unit of Work + Repository (EF Core) es a menudo un **anti-patrón** (abstracción redundante que oculta capacidades). El "cuándo NO" importa.
- No se nombra el **Unit of Work** como concepto —y es vital, porque Cosmos usa `UnitOfWorkMiddleware` y `AutoApplyTransactions()`.

### 06 — Inyección de Dependencias · 🔴 Crítico (el caso que mencionaste)
- Llama a la resolución **"la magia absoluta de C#"** — y la deja como magia. Ese es exactamente el problema que quieres resolver.
- **Confunde 4 conceptos distintos** bajo un mismo nombre. Hay que separarlos:
  - **DIP** (principio SOLID): depende de abstracciones.
  - **IoC** (patrón): que algo externo controle el flujo/creación.
  - **DI** (técnica): pasar dependencias por constructor — *funciona sin ningún framework* ("pure DI").
  - **Contenedor IoC** (herramienta): automatiza el grafo de objetos. **No es obligatorio.**
- **Falta desarmar la magia:** el contenedor no es magia de C#, es **un registro + reflexión + recursión**. La forma de entenderlo de verdad es **construir un mini-contenedor de 20 líneas** (lo propongo abajo). Cuando lo construyes, deja de ser mágico para siempre.
- **Faltan trampas de nivel:**
  - **Captive dependency**: inyectar un `Scoped` dentro de un `Singleton` lo "captura" y rompe el ciclo de vida (bug real y silencioso).
  - **Service Locator como anti-patrón**: `GetRequiredService` por todos lados (solo es aceptable en el *composition root*).
  - **Composition Root**: el único lugar donde se arma todo.
- **Conexión Wolverine:** Wolverine **no resuelve por reflexión en cada llamada** (como un mediator clásico); **genera código** que hace la resolución explícita. Esto es lo que de verdad mata la "magia" — y enlaza con la sección nueva de codegen.

### 07 — Lenguaje Ubicuo · 🟢 Bien
- Buena sección. **Falta** introducir **Bounded Context**: el lenguaje ubicuo es *por contexto acotado* (la misma palabra significa cosas distintas en Contabilidad vs Impuestos). Y Cosmos **está organizado literalmente por Bounded Contexts** — es vital.

### 08 — Aggregate y Aggregate Root · 🟢 Bien
- Buen ataque al **modelo anémico**. **Faltan** dos reglas de oro: (1) **una transacción = un agregado** (frontera transaccional); (2) cómo **dimensionar** el agregado (pequeño, por invariantes) — el error clásico es agregados gigantes.

### 09 — Eventos de Dominio · 🟠 Alto
- **Falta la distinción crítica Domain Event vs Integration Event:**
  - **Domain event**: dentro del mismo bounded context/proceso.
  - **Integration event**: cruza a otros bounded contexts (por el bus).
  - Cosmos lo implementa **exactamente** así: `WolverinePrivateEventSender` (interno) vs `WolverinePublicEventSender` (público/integración). Sin esta distinción no se entiende el diseño real.

### 10 — Async/Await · 🟠 Alto
- **El título promete "El Bloqueo Mortal" pero nunca explica el deadlock.** Hay que cerrarlo:
  - El deadlock clásico (`.Result`/`.Wait()`) ocurre por captura del **SynchronizationContext** (UI / ASP.NET *clásico*). **Matiz importante y correcto:** **ASP.NET Core y Functions NO tienen SynchronizationContext**, así que ahí `.Result` no hace *deadlock* pero **sigue bloqueando hilos** (thread starvation). Cosmos es ASP.NET Core/Functions → el problema real es starvation, no deadlock.
- **Faltan, y son nivel medio-avanzado obligatorio:**
  - `async/await` compila a una **máquina de estados** (no hay un hilo "esperando").
  - **`CancellationToken`** (todas las APIs de Marten/Wolverine lo reciben).
  - **`ConfigureAwait(false)`** en librerías.
  - **`async void`** (peligroso, solo para event handlers).
  - `ValueTask` (cuándo).

### 11 — Fundamentos de CQRS · 🟡 Medio
- **Faltan:** la **consistencia eventual** como consecuencia (no solo separar clases), el **read model/proyección** como ciudadano de primera, y el **"cuándo NO usar CQRS"** (la mayoría de CRUDs no lo necesitan).

### 12 — Outbox · 🟢 Bien
- Buena explicación del doble fallo. **Faltan:** la entrega es **at-least-once** → el consumidor necesita **idempotencia/dedup**; y el patrón hermano **Inbox** (que Wolverine también trae).

---

## C. Conceptos vitales ausentes (los que hacen la diferencia)

Estos no están y son justo los que separan "usar la plantilla" de "entender":

1. **SOLID explícito** — sobre todo **DIP** (base de DI) y **OCP** (base del dispatch). Hoy se mencionan de pasada.
2. **Genéricos y restricciones (`where T : ...`)** — base de `IEventStore`, `AggregateStreamAsync<T>`, y del descubrimiento de handlers `Handle<TCommand>`.
3. **Delegados, `Func`/`Action` y composición de funciones** — **desarma la "magia" del middleware**: un pipeline de middleware *es* composición de funciones (`next()`). Sin esto, Wolverine middleware parece hechicería.
4. **Reflexión vs Generación de código** 🔴 — el corazón anti-magia. Por qué MediatR usa reflexión por-llamada y **Wolverine genera código C# inspeccionable** (`codegen`), mejor rendimiento y *sin* sorpresas en runtime. Se puede **previsualizar el código generado**.
5. **Bounded Context** (DDD estratégico) — Cosmos *es* bounded contexts.
6. **Domain vs Integration events** — el público/privado de Cosmos.
7. **Unit of Work** — el `UnitOfWorkMiddleware` de Cosmos.
8. **Testing** — el *porqué último* de la DI es la **testabilidad**. Unit-testear agregados (puro, sin mocks) y handlers (con dobles). Sin esta sección, la DI parece burocracia.
9. **Manejo de errores / Result / Railway** — Wolverine lo trae nativo; evita el `try/catch` ciego.
10. **Idempotencia y entrega at-least-once** — realidad de toda mensajería.
11. **Concurrencia optimista y versionado** — en ES, dos comandos sobre el mismo stream → conflicto de versión. Marten lo maneja; hay que entenderlo.

---

## D. Replanteamiento propuesto (currículo por niveles)

Unificar los dos workshops en **un solo viaje con 3 niveles**, insertando los conceptos faltantes y aplicando el principio *"primero a mano, luego la herramienta"*.

### 🟢 NIVEL BÁSICO — El lenguaje y el porqué
1. Inmutabilidad real (records, shallow vs deep, `readonly record struct`)
2. Tipos: interface vs abstract **vs default interface methods** + composición sobre herencia
3. **Genéricos y restricciones** *(nuevo)*
4. **Delegados, `Func`/`Action` y composición** *(nuevo — semilla del middleware)*
5. **SOLID en 5 ejemplos reales** *(nuevo, reemplaza menciones sueltas)*
6. Polimorfismo: `switch` tipado **primero**, `dynamic` después con sus costos

### 🟡 NIVEL MEDIO — Estructura y dominio
7. Patrón Comando (GoF vs CQRS-command) + dónde validar
8. Repositorio + **Unit of Work** + cuándo es anti-patrón
9. **DI de raíz**: DIP→IoC→DI→Contenedor + **construir un mini-contenedor** + lifetimes + captive dependency + composition root
10. Lenguaje Ubicuo + **Bounded Context** *(ampliado)*
11. Aggregate Root + frontera transaccional + dimensionamiento
12. **Domain events vs Integration events** *(ampliado)*
13. Async de verdad: máquina de estados, starvation vs deadlock, CancellationToken, ConfigureAwait, async void

### 🔴 NIVEL AVANZADO — Producción y "la magia desarmada"
14. **Reflexión vs Generación de código** *(nuevo, central)* — previsualizar el código que genera Wolverine
15. CQRS + proyecciones + consistencia eventual + cuándo NO
16. Event Store, replay, snapshots, **concurrencia optimista** *(ampliado)*
17. Outbox + **Inbox** + **idempotencia** *(ampliado)*
18. **Testing** del dominio y de handlers *(nuevo)*
19. Wolverine en serio: middleware como composición, durabilidad, Azure Service Bus + FIFO por tenant, serverless/codegen
20. Cierre: reconstruir un Bounded Context real de Cosmos de punta a punta

> Las secciones actuales encajan casi todas; lo *nuevo* es: genéricos, delegados/composición, SOLID, mini-contenedor, reflexión-vs-codegen, testing, y los *ampliados* (bounded context, domain-vs-integration, async profundo, concurrencia, idempotencia).

---

## E. Principios pedagógicos (la filosofía anti-cargo-cult)

1. **Construye la magia antes de usarla.** Mini-contenedor de DI (20 líneas), mini-mediator, mini-outbox a mano. Luego ves cómo el framework lo automatiza — y deja de ser magia.
2. **Muestra el código generado.** Con Wolverine, previsualizar el código que genera convierte el "no sé qué pasa" en "leo exactamente qué pasa".
3. **Cada herramienta resuelve un dolor que YA sentiste.** Nunca introducir Marten/Wolverine/DI sin haber sufrido primero el problema a mano.
4. **Siempre el tradeoff y el "cuándo NO".** Ningún patrón es gratis. Madurez = saber cuándo no usarlo.
5. **Validación por checkpoint.** No se avanza de sección sin poder *explicar de memoria* el mecanismo (no solo la sintaxis). Esto enlaza con tu protocolo de Loops.

---

## F. Recomendación de ejecución

Sugiero este orden (en secuencia, con commits tuyos en el repo):
1. **Arreglos críticos primero** (🔴): reescribir la Sección 03 (dynamic + alternativas) y la 06 (DI desarmada con mini-contenedor). Son los de mayor impacto.
2. **Insertar las 3 secciones-semilla** del nivel básico: genéricos, delegados/composición, SOLID.
3. **Sección central nueva:** Reflexión vs Codegen (la que más eleva el nivel).
4. **Ampliar** async, eventos (domain vs integration), bounded context, concurrencia, idempotencia.
5. **Añadir Testing.**
6. Validar cada una con un **Loop** (foco + estudio + preguntas).

> Dime por dónde arrancamos. Mi recomendación: la **Sección 06 (DI) reescrita con el mini-contenedor**, porque es el ejemplo exacto que diste de "magia que no se entiende" y marca el tono de todo el replanteamiento.
