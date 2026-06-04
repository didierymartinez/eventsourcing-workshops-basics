# 20 - CQRS y Proyecciones: el lado de lectura

> 🌳 **Sección donde se domina** la semilla de §06 ("el estado es solo una de muchas vistas"). Pasamos del lado de escritura (eventos) al lado de lectura (proyecciones), con sus tres sabores y sus tradeoffs reales.

## El dolor: leer reproduciendo eventos no escala

Para saber si Jhon está casado, `AggregateStreamAsync` reproduce su stream completo. Para **un** agregado, perfecto. Pero el negocio pregunta cosas como:

- *"Dame el listado de las personas casadas este mes, ordenadas por ciudad."*
- *"¿Cuántos matrimonios hubo por ciudad este año?"*

Responder eso reproduciendo **todos los streams de todas las personas** cada vez sería absurdo. El log de eventos es excelente para **escribir** y para **reconstruir un agregado**, pero pésimo para **consultas arbitrarias**.

---

## La idea: separar escribir de leer (CQRS)

**CQRS** (Command Query Responsibility Segregation) separa dos modelos que tradicionalmente mezclamos:

| | Lado de Escritura (Command) | Lado de Lectura (Query) |
|---|---|---|
| Fuente de verdad | el log de eventos | derivada de los eventos |
| Forma | agregados + eventos | vistas optimizadas para consultar |
| Optimizado para | aplicar reglas e invariantes | velocidad de lectura |
| En Cosmos | `IEventStore`, agregados | `IProjectionStore`, `IQueryRouter` |

> [!NOTE]
> CQRS **no exige** Event Sourcing ni bases de datos separadas. Son ideas ortogonales. Pero con Event Sourcing encajan de maravilla: como guardas todos los hechos, puedes construir **cuantas vistas de lectura quieras** a partir de ellos.

Una **proyección** es justamente eso: una estrategia para transformar el stream de eventos en una **vista de lectura** (un *read model*).

---

## Los tres sabores de proyección (Marten)

Esta es la decisión que más importa, porque define la **consistencia**:

### 1. Live (en vivo, bajo demanda)
Se calcula al momento, **sin almacenarse**. Es `AggregateStreamAsync` que ya conoces. Ideal para un solo agregado consultado ocasionalmente.
- ✅ Cero almacenamiento, siempre consistente.
- ❌ No sirve para consultar muchos a la vez.

### 2. Inline (en la misma transacción de escritura)
La vista se actualiza **dentro de la misma transacción** que guarda el evento. Cuando el comando termina, el read model **ya está actualizado**.
- ✅ **Consistencia fuerte**: lo que escribes, lo lees inmediatamente.
- ❌ Añade trabajo a cada escritura; acopla lectura y escritura en el tiempo.

### 3. Async (daemon en background)
Un proceso aparte —el **Async Daemon** de Marten— va leyendo los eventos nuevos y actualizando las vistas **después**, en segundo plano.
- ✅ Escala; no penaliza la escritura; permite reconstruir vistas pesadas.
- ❌ **Consistencia eventual**: hay un retraso (milisegundos a segundos) en el que la vista está "atrasada".

> [!IMPORTANT]
> 🌱→🌳 Aquí se cierra la semilla de §06: la **consistencia eventual** no es un bug, es un **trade consciente**. Eliges async cuando la escala importa más que ver el dato actualizadísimo al instante. Eliges inline cuando necesitas leer-tras-escribir consistente.

---

## Single-stream vs Multi-stream

- **Single-stream projection:** la vista resume **un** stream (ej. el estado actual de **una** persona). Suele usarse como **snapshot** del agregado.
- **Multi-stream projection:** la vista combina eventos de **muchos** streams (ej. "matrimonios por ciudad" cruza miles de personas). Marten las coordina en el daemon.

También existen las **flat-table projections** (proyectar a una tabla relacional plana para reporting con SQL puro).

```csharp
// Esbozo conceptual de una proyección de un solo stream (snapshot de la Persona)
public class PersonaResumen
{
    public Guid Id { get; set; }
    public string Nombre { get; set; }
    public string Ciudad { get; set; }
    public bool Casado { get; set; }

    public void Apply(PersonaNació e)   { Id = e.PersonaId; Nombre = e.Nombre; Ciudad = e.Ciudad; }
    public void Apply(PersonaCasada e)  { Casado = true; }
    public void Apply(PersonaMudada e)  { Ciudad = e.NuevaCiudad; }
}
// Registro: options.Projections.Add<PersonaResumenProjection>(ProjectionLifecycle.Async);
```

> [!NOTE]
> Operación real (Marten lo documenta): si **cambias** una proyección, la **reconstruyes** (`rebuild`) desde el evento cero — porque el read model es *derivado*, siempre puedes regenerarlo. Y las proyecciones se **testean** dando eventos de entrada y verificando la vista resultante.

---

## Cómo lo expone Cosmos

En `Cosmos.BuildingBlocks`:
- `Cosmos.EventSourcing.Abstractions/Queries/IProjectionStore.cs` — contrato del lado lectura.
- `Cosmos.EventSourcing.CritterStack/Queries/MartenProjectionStore.cs` — implementación.
- `WolverineQueryRouter` — enruta las queries (espejo del `WolverineCommandRouter`).
- `Cosmos.EventSourcing.Linq.Extensions/QueryableExtensions.cs` — consultas LINQ sobre los read models (`session.Query<PersonaResumen>().Where(...)`).

---

## ¿Cuándo NO usar CQRS?

> [!WARNING]
> La mayoría de los CRUDs **no** necesitan CQRS. Añade piezas (read models, sincronización, consistencia eventual) que cuestan complejidad. Justifícalo cuando: las lecturas y escrituras tienen formas/escala muy distintas, necesitas varios modelos de lectura, o el reporting pesa sobre la base transaccional. Si tu duda es "¿lo aplico?", probablemente todavía no.

---

### El Descubrimiento
El lado de escritura protege la verdad (eventos + invariantes); el lado de lectura la sirve rápido (proyecciones). La palanca clave es **inline (consistencia fuerte) vs async (consistencia eventual, escala)**. Y como las vistas son derivadas, son **desechables y reconstruibles**.

**Siguiente:** ya tienes escritura y lectura. Falta blindar todo con **pruebas** — testear agregados y handlers sin infraestructura (la semilla de §08).

---

[⬅️ Volver a Versionado de Eventos](./19-versionado-de-eventos.md) · [🗺️ Roadmap](../ROADMAP.md)
