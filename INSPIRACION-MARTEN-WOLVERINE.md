# 📚 Qué enseñan Marten y Wolverine que deberíamos introducir

> Barrido de la documentación oficial de **Marten** (martendb.io) y los **tutoriales de Wolverine** (wolverinefx.net/tutorials) para detectar conceptos y patrones que ellos explican y que elevarían nuestro workshop. Cada ítem incluye: qué enseñan, por qué vale, y dónde encaja en el [ROADMAP](./ROADMAP.md).
>
> Fuentes vivas para subir a NotebookLM: Marten `https://martendb.io/llms-full.txt` · Wolverine `https://www.wolverinefx.io/llms-full.txt`

---

## 🟥 Lo más valioso: patrones productivos que cambian cómo escribimos handlers

Nuestro workshop (y la plantilla en Sección 27) enseña el flujo **manual**: `GetAggregateRootAsync` → método del agregado → `Save` → `SaveChangesAsync`. El tutorial oficial **Event Sourcing & CQRS with Marten** de Wolverine enseña un nivel por encima que deberíamos introducir:

### 1. Aggregate Handler Workflow (`[Aggregate]` + `FetchForWriting`) 🔴
Wolverine + Marten **eliminan el código repetitivo** de cargar/guardar el agregado. Tú escribes solo la decisión:
```csharp
[AggregateHandler] // o [Aggregate] en el parámetro
public static IEnumerable<object> Handle(AprobarOrden cmd, OrdenDeCompra orden)
{
    if (orden.Aprobada) yield break;
    yield return new OrdenDeCompraAprobada(cmd.OrdenId, cmd.AprobadorId); // solo devuelves eventos
}
```
- `FetchForWriting<T>(id)` carga el agregado **con concurrencia optimista** (captura la versión esperada) y, al terminar el handler, Marten **appendea los eventos devueltos** en la misma transacción.
- **Por qué vale:** es el patrón real de producción del "Critter Stack". Conecta con la best-practice de Wolverine de **funciones puras sin mocks**. Hace tu handler testeable y sin infraestructura.
- **Encaje:** nueva sección avanzada *"El handler que no toca la base: Aggregate Handler Workflow"*.

### 2. El patrón Decider (Event Sourcing funcional) 🔴
La forma académica y testeable de modelar: dos funciones puras.
```
decide(command, state)  -> eventos        // qué pasó (no muta nada)
evolve(state, event)    -> nuevo estado    // cómo cambia el estado (= nuestro Apply)
```
- **Por qué vale:** separa decisión de evolución; ambas puras → tests triviales (given eventos, when comando, then eventos). Es el fundamento detrás del Aggregate Handler. Nuestro `Apply` ya es el `evolve`; falta nombrar el `decide`.
- **Encaje:** sección conceptual en Nivel Medio, antes del Aggregate Handler.

### 3. `FetchLatest` y eventos en cascada (`MartenOps.StartStream`) 🟠
- `FetchLatest<T>(id)` devuelve el agregado **incluyendo los eventos recién emitidos** (para responder con el estado actualizado).
- `MartenOps.StartStream<T>(id, evento)` se **devuelve** como side-effect desde un handler puro para iniciar un stream — sin inyectar la sesión.
- **Por qué vale:** muestra cómo mantener handlers puros incluso al crear streams. Conecta con "Side Effects" de Wolverine.

---

## 🟧 Marten: áreas completas que no cubrimos

La sección **Event Store** de Marten documenta mucho más que "append + aggregate". Candidatos a sección:

| Tema Marten | Qué enseñan | Por qué introducirlo |
|---|---|---|
| **Event Versioning** (`events/versioning`) | cómo evolucionar el esquema de un evento sin romper el pasado (upcasting) | 🔴 **Vital y ausente**: los eventos son inmutables y eternos; cambiarlos es un problema real de producción |
| **Tipos de proyección** (`projections/*`) | Inline vs Async (daemon) vs Live; Single-stream, **Multi-stream**, Flat-table, Event projections | nuestra Sección 16 (CQRS) debe distinguirlos; cada uno tiene tradeoffs distintos |
| **Async Daemon** | proceso en background que reconstruye proyecciones con consistencia eventual + health checks | explica la consistencia eventual de forma concreta y operable |
| **Single Stream Projections & Snapshots** | snapshot del agregado como proyección | nuestra sección 💡 Snapshots se apoya aquí |
| **Rebuilding & Testing Projections** | reconstruir un read model desde cero; testear proyecciones | operación real: si cambias una proyección, la reconstruyes |
| **Event Subscriptions** | reaccionar a eventos ya guardados (vs proyectar) | base para integraciones y side-effects |
| **Optimistic Concurrency** (`documents/concurrency`, `events/...`) | versión esperada, `ConcurrencyException`, reintento | confirma y profundiza nuestra sección de concurrencia |
| **Archiving / Stream Compacting / Protection (GDPR)** | archivar streams viejos, compactar, borrar info protegida | temas de producción/cumplimiento que nadie enseña |
| **Dynamic Consistency Boundary (DCB)** | fronteras de consistencia dinámicas (Marten v8) | avanzado/novedoso; candidato 💡 |
| **Querying Events / Metadata** | consultar el stream crudo, metadata (timestamp, version, correlationId) | útil para auditoría y debugging |

---

## 🟨 Wolverine: tutoriales que aportan conceptos nuevos

| Tutorial Wolverine | Concepto que añade | Encaje |
|---|---|---|
| **Railway Programming** | manejar éxito/fallo como flujo de datos (sin `try/catch` ciego); `Result`-like | Nivel Medio: manejo de errores |
| **Dealing with Concurrency** | estrategias de concurrencia más allá del optimista | Nivel Avanzado |
| **Idempotency in Messaging** | cómo Wolverine ayuda con dedup/idempotencia | refuerza nuestra Sección 14 (Outbox) |
| **Dead Letter Queues** | qué pasa con mensajes que fallan repetidamente | operación real de mensajería |
| **Multi-Tenancy (holístico)** | tenancy across HTTP + messaging + persistence | conecta con `Cosmos.MultiTenancy` |
| **Vertical Slice Architecture** / **A-Frame** | organizar por caso de uso, no por capas; menos ceremonia | contrapunto a Clean/Onion (best-practice oficial) |
| **Custom Middleware** | construir middleware propio (= composición de funciones) | conecta con la sección nueva de delegados |
| **Modular Monolith** | empezar monolito modular antes de microservicios | decisión arquitectónica madura |

---

## 🎯 Recomendación de incorporación (priorizada)

**Imprescindibles (suben el nivel de inmediato):**
1. **Aggregate Handler Workflow + Decider** — cambia cómo se escriben los handlers; es el patrón real del Critter Stack.
2. **Event Versioning / Upcasting** — hueco crítico: nadie piensa en cómo evolucionar eventos hasta que rompe producción.
3. **Tipos de proyección + Async Daemon** — completa la Sección 16 (CQRS) con sustancia real.

**Importantes:**
4. Rebuilding & Testing projections.
5. Railway programming (manejo de errores).
6. Idempotency + Dead Letter Queues (operación de mensajería).

**Avanzados / candidatos:**
7. Snapshots (single-stream projections), Subscriptions, Archiving/Compacting/GDPR, DCB.

> Estos ítems ya quedaron **sembrados en el [ROADMAP](./ROADMAP.md)** como 🧩 planificadas / 💡 candidatas. La filosofía se mantiene: introducir cada uno solo tras sentir el dolor que resuelve, y mostrar el patrón manual antes del atajo del framework.
