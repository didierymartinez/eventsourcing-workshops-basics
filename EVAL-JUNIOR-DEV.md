# 🧑‍🎓 Evaluación "Junior-Dev": fugas de concepto y huecos de narrativa

> Informe de una lectura simulada del workshop **con ojos de un desarrollador junior-medio**: maneja C# (CRUD, APIs, clases, LINQ básico) y patrones básicos, pero es **nuevo en Event Sourcing, EDA, DDD, Marten y Wolverine**. Recorre fundamentals (Nivel 0) + principal en orden y marca dónde se perdería. **Informe primero — sin aplicar cambios todavía.**
>
> Leyenda: 🔴 me perdí / bloqueo · 🟡 término usado sin explicar · 💡 no entiendo *para qué* lo hago · 🔀 salto de ejemplo o narrativa · ✅ bien resuelto.

---

## Veredicto general (voz del junior)

> "El workshop es **muy bueno** — las metáforas y el ir 'a mano primero' me hacen entender de verdad. Donde más me cuesta es: (1) algunos **términos de C# y Azure** que aparecen como si ya los supiera; (2) en **fundamentals salto de ejemplo** en casi cada sección (Persona, carrito, hospital, factura, orden) y pierdo el hilo; (3) a veces hay **muchas promesas hacia adelante** ('lo veremos en §X') y me quedo con la duda colgando; (4) un par de veces no sé si la clase `Persona` se construye con eventos o vacía."

---

## A. Hallazgos en el `fundamentals-workshop`

| Sec | Tipo | Qué me confunde (como junior-medio) |
|-----|------|--------------------------------------|
| §02 | 🟡 | `ICommandHandler<in TCommand>`: **no sé qué hace el `in`** (contravarianza). Aparece sin una palabra. |
| §03 | 🟡 | "DLR (Dynamic Language Runtime)" se nombra; no sé qué es. Y `((dynamic)this).Apply(...)` — el doble cast `(dynamic)` me marea la primera vez. |
| §03 | 🔀 | El ejemplo es `Persona`/eventos, pero en §08-§12 cambia a carrito/hospital/factura. ¿Es el mismo proyecto o no? |
| §04 | 🟡 | `[ApiController]`, `IActionResult`, `[FromBody]`: asume que conozco ASP.NET MVC. Si vengo solo de APIs mínimas, dudo. |
| §05 | 💡 | Entiendo el Repositorio, pero "verás `IDocumentSession` en el workshop principal" me deja con la duda de qué es eso *ahora*. |
| §07 | 🔀 | Salta a un ejemplo de **hospital/triage** (`PacienteCriticoIngresado`). Bueno para el punto, pero otro dominio más. |
| §08-§09 | 🔀 | Ahora es un **carrito de compras** (`ProductoAgregadoAlCarrito`, `TotalDolares`). Tercer/cuarto dominio. |
| §08 | 🟡 | `TotalDolares` y "los iteradores" se mencionan como si ya existieran en un ejemplo que no vi completo. |
| §10 | 🔀🟡 | Ejemplo de **Paciente** otra vez + metáfora del cocinero. Y `SynchronizationContext`/"ASP.NET clásico" — no sé qué es "clásico" vs Core. |
| §11 | 🟡 | "Elasticsearch", "desnormalizada" — términos sueltos; un junior puede no saber qué es desnormalizar. |
| §12 | 🟡 | `.AddMartenOutbox()` / RabbitMQ aparecen sin contexto; ¿de dónde sale Rabbit? |

**Patrón #1 (importante):** *fundamentals no tiene un hilo único*. Cada sección usa un dominio distinto (Persona, hospital, carrito, factura, orden). Como junior, **gasto energía re-entendiendo el escenario** en vez del concepto. El principal sí tiene un solo hilo (Jhon) y se siente mucho más fácil.

---

## B. Hallazgos en el workshop principal

| Sec | Tipo | Qué me confunde |
|-----|------|------------------|
| §01b | 🔴 | Justo después del §01 (suave) me cae **mucho de golpe**: Bounded Context, DOS contextos, eventos de integración, ACL, Envelope (4 forward-refs). Entiendo que "no debo resolverlo aún", pero me intimida. |
| §03 | 💡 | La `Persona` se construye con `new Persona(biografia)` (constructor con eventos). En §05/§21 veo `new Persona()` + `Load(...)`. **¿Cuál es? ¿Cambió?** Nadie me dice que evolucionó. |
| §05 | 🟡 | `where T : AggregateRoot, new()` — el `new()` constraint no se explica (por qué exige constructor vacío). |
| §09 | 💡 | El handler async de ejemplo **no recibe `CancellationToken`**, pero la caja de abajo dice "siempre pásalo". El ejemplo contradice la regla. |
| §11/§13 | 🟡 | "Azure Service Bus", "Azure Functions (.NET isolated)", "sesiones FIFO", "DurabilityMode.Solo" — varios términos de Azure que un junior nuevo no ubica. |
| §13/§22 | 🟡 | "codegen", "DLR" otra vez, `dotnet run -- codegen write` — útil, pero asumo que tengo un proyecto montado para probarlo. |
| §18 | 🟡 | `MartenOps.StartStream`, `IStartStream`, `CreationResponse`, `FetchLatest` aparecen en el ejemplo de "iniciar stream" sin explicar qué son (sobre todo `CreationResponse`). |
| §18 | 🟡 | El handler usa `yield return`/`yield break`. Sé qué es un iterador, pero **¿por qué un handler devuelve un `IEnumerable` perezoso?** No me lo explican. |
| §20 | 🟡 | `ProjectionLifecycle.Async`, "flat-table", "async daemon": menciono que async daemon suena importante pero no sé qué es exactamente. |
| §24/§25 | ✅ | Muy claros: el arco ingenuo→dolor→refactor me hace *sentir* el porqué. Aquí no me pierdo. |

**Patrón #2:** **densidad de §01b**. Es el único punto temprano donde un junior siente "demasiado a la vez". Quizá moverlo un poco después, o aligerarlo a solo "hay un afuera; algunos hechos cruzan", dejando ACL/Envelope sin nombrar tan pronto.

**Patrón #3:** **la evolución de `Persona` (constructor) no está señalizada**. Pasa de `new Persona(eventos)` (§03-08) a `new Persona()` + `Load` (§05+). Un junior cree que se equivocó.

---

## C. Hallazgos transversales (todo el workshop)

1. 🟡 **Términos usados antes de definir / asumidos:**
   - *De C#:* varianza `in`/`out` (§02 fund.), `new()` constraint (§05), iteradores `yield` como retorno de handler (§18), `DLR`.
   - *De Azure:* Service Bus, Functions/.NET isolated, sesiones FIFO, RabbitMQ — un junior nuevo en la nube no los ubica. Falta una línea de "qué es esto" o un glosario.
2. 💡 **Propósito al inicio (tu observación):** ya mejoró con el mapa "🎯 hacia dónde va" del README de fundamentals. Faltaría que **cada sección** abra con una línea "🎯 al terminar sabrás/usarás esto para…", no solo el README.
3. 🔀 **Cohesión de ejemplos en fundamentals:** el mayor desgaste cognitivo del junior. Opciones: unificar a un solo dominio, o poner en cada sección un encabezado "Ejemplo: [dominio]" para avisar el cambio.
4. 🔮 **Densidad de forward-refs:** muchas semillas prometen "lo veremos en §X". Está bien (es el diseño en espiral), pero juntas se sienten muchas. Un junior agradecería que cada semilla diga en **una palabra** qué se llevará *ahora* (no solo "lo verás después").
5. ✅ **Lo que funciona excelente** (no tocar): los arcos naive→dolor→refactor (§07 idempotencia, §06 concurrencia, §10 mini-contenedor, §23 middleware, §24 ACL, §25 Envelope, §26 EDA). Ahí el junior "siente" el porqué y no se pierde.

---

> **✅ Estado (04/06): los 7 hallazgos fueron APLICADOS** (vía 3 agentes en paralelo para lo mecánico + arreglos directos para lo sensible a coherencia):
> 1. **Cohesión de ejemplos (fundamentals):** cada sección con otro dominio ahora abre con `📦 Ejemplo de esta sección: …`.
> 2. **Evolución de `Persona`:** nota en §05 ("a partir de aquí se crea vacía + `Load`, no por constructor").
> 3. **§01b denso:** TL;DR al inicio ("lo único que debes llevarte"); el resto es mapa para ojear.
> 4. **Términos asumidos:** nuevo `GLOSARIO.md` (C#/Azure/arquitectura) enlazado en el README + nota de `in` (contravarianza) en §02 fund.
> 5. **§09 sin `CancellationToken`:** añadido al handler de ejemplo + propagado.
> 6. **§18 términos sin explicar:** glosario rápido de `yield`/`MartenOps.StartStream`/`IStartStream`/`CreationResponse`.
> 7. **"🎯 hacia dónde va" por sección:** insertado en las 27 del principal + las 12 de fundamentals (+ mapa de destinos en el README de fundamentals).
> Verificado: sin drift de nombres ni headers duplicados.

## D. Top hallazgos priorizados (recomendación)

| # | Prioridad | Hallazgo | Arreglo sugerido |
|---|-----------|----------|------------------|
| 1 | 🔴 Alta | Cohesión de ejemplos en fundamentals (Persona/hospital/carrito/factura) | Encabezar cada sección con "**Ejemplo:** [dominio]" o unificar a uno |
| 2 | 🔴 Alta | Evolución de `Persona` (ctor con eventos → `new()`+`Load`) sin señalizar | Una nota en §05 que diga "a partir de aquí Persona se crea vacía y se rehidrata con `Load`" |
| 3 | 🟠 Media | §01b cae muy denso tras §01 | Aligerar: dejar BC + "hay un afuera"; mover ACL/Envelope a una mención más breve |
| 4 | 🟠 Media | Términos de Azure/C# asumidos (`in`, `new()`, Service Bus, FIFO, yield-handler) | Glosario corto + 1 línea la primera vez que aparece cada uno |
| 5 | 🟠 Media | §09 handler de ejemplo sin `CancellationToken` (contradice su propia regla) | Añadir `CancellationToken ct` al ejemplo |
| 6 | 🟢 Baja | `MartenOps.StartStream`/`CreationResponse`/`yield` en §18 sin explicar | 1-2 líneas de qué son |
| 7 | 🟢 Baja | Cada sección abra con "🎯 hacia dónde va" (no solo el README) | Encabezado de propósito por sección |

---

## D-bis. 2ª pasada (re-evaluación, 04/06) — veredicto

Se re-corrió el evaluador junior (ver [EVALUADOR-JUNIOR-DEV.md](./EVALUADOR-JUNIOR-DEV.md)) tras los arreglos:
- **Los 7 hallazgos: ✅ RESUELTOS** (verificados archivo por archivo).
- **Veredicto:** *"la narrativa temprana ya deja claro para qué/hacia dónde va cada concepto; el workshop pasó de 'muy bueno con fugas' a sólido y autoconsistente para junior-medio."* Sin hallazgos 🔴 ni 🔀 nuevos.
- **2 residuales cosméticos (🟢) — también cerrados:** (1) firmas `GetAsync`/`AppendAsync` en §09 ahora reciben `CancellationToken ct = default` (coinciden con el handler); (2) pointer inline al `GLOSARIO` la 1ª vez que aparecen términos Azure en §13.

## D-ter. 3ª pasada (perfil JUNIOR-REAL, 08/06) — bugs reales encontrados

Se corrió el evaluador con un perfil **más bajo** ("solo CRUD y APIs"; no domina genéricos, yield, async, delegados, records). Encontró cosas que el junior-medio no ve — **y varias eran bugs reales, no solo de nivel**. Aplicadas directamente (insertar/aclarar, sin renombrar ni reescribir código):

1. **Link roto §14 → `15-limites-busqueda.md`** (sección nunca escrita; su contenido se absorbió en §20). Reapuntado a §20 (CQRS y Proyecciones). Verificado: **todos** los links internos entre secciones resuelven.
2. **§04 "Refactor 1 → Refactor 3"** sin 2: la extracción de la clase base abstracta ahora tiene su encabezado **Refactor 2**.
3. **Excepciones de dominio nunca definidas** (`ConcurrencyException`, `ReglaDeNegocioException`, `EventoInvalidoException`): nota en §06 con la definición de 1 línea ("son clases tuyas, heredan de `Exception`; asume que existen").
4. **§08 no chequeaba el `null`** que §07 acababa de enseñar (`Casar` → `PersonaCasada?`, no-op por idempotencia): WARNING que reconcilia el cabo suelto + muestra el `if (e is null) return;` y remite a §18.
5. **§26 `RaiseEvent` aparecía sin avisar** (el agregado venía *devolviendo* eventos): NOTE que lo declara equivalente (`RaiseEvent(e)` ≡ `Apply(e)` + encolar) y remite a §27.
6. **Typos que parecían términos técnicos** en fundamentals: *resuelcan, ascendemente, repuesta, Receptorio, pre-cacullados, Dipendency, Aismlamiento* → corregidos.

**Quedan como decisión de producto (NO aplicadas):** ver §F.

## F. Decisiones de producto (08/06) — APLICADAS

Didier decidió:

1. **Hand-holding de C# → opción (B): Glosario C# + pointer.** Hecho:
   - `GLOSARIO.md` ampliado con una subsección **"Símbolos de sintaxis (los que damos por conocidos)"**: `T?`, `?.`, `??`, `!`, `=>` con cuerpo de expresión, lambdas, constructor primario (C# 12), retorno por tupla, `out var`, sufijo `m`, `[Fact]` (xUnit), `.Should()` (FluentAssertions/AwesomeAssertions).
   - Pointers al glosario insertados en los dos puntos donde más frena: **§07** (antes del bloque de tests: `[Fact]`/`.Should()`/lambdas) y **§06** (junto al primer `?.`/`??`). Audiencia sigue siendo junior-medio; el cuerpo no se engorda.

2. **Orden de §23 (delegados/closures) → adelantar.** Hecho (era una **referencia circular**, no solo de orden):
   - Antes: §13 decía de los delegados *"lo dominaremos"* (futuro) y §23 decía *"domina la semilla de §13"* (pasado) → contradicción.
   - Ahora: **§23 se lee temprano** (Nivel Básico, antes de §10/§13). Su intro se volvió autónoma y *forward-planta* Wolverine (§13) en vez de back-referenciarlo, con una nota de que el handler es solo *andamio*. La semilla de §13 ahora dice *"ya lo construiste en §23"*. El ROADMAP marca explícito que §23 se lee aquí pese a su número de archivo.
   - **Caveat cerrado (08/06):** el ejemplo de §23 se reescribió a **C# puro** — operaciones de una mini-app (`EnviarBienvenida`, `GenerarReporte`, `ExportarCsv`) con `Console.WriteLine` + `Task.Delay` que simula el trabajo. Cero event store, cero `Orden`/`_store`. El delegado se llama `Operacion<T>`; las anclas a Wolverine/§13/§22 ahora son **forward-refs** (no asumen secciones ya leídas) y el footer apunta al ROADMAP en vez de a §22. Ya se lee de verdad al inicio.

## E. Cómo se usó este informe
Simulación de lectura lineal con perfil junior-medio. **No se aplicó ningún cambio**; este documento es el insumo para que decidas qué arreglar. Si quieres, puedo: (a) aplicar el Top 1-2 (los de mayor impacto), (b) crear un **prompt reutilizable "evaluador junior-dev"** para volver a correr esto cuando el workshop cambie, o (c) profundizar la simulación en una sección concreta.
