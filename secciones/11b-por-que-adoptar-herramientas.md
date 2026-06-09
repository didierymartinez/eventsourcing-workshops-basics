# 11b - El momento de adoptar herramientas: Marten + Wolverine, sin magia

> 🎯 **Hacia dónde va:** hacemos el recuento honesto del motor que construiste a mano y justificamos por qué y qué adoptar de Marten y Wolverine, el puente hacia las herramientas de producción sin tratarlas como magia.

> 🧭 **Punto de inflexión del workshop.** Hasta aquí construiste **todo el motor a mano**. Antes de adoptar librerías, paramos a responder dos preguntas honestas: *¿por qué adoptarlas (necesidad real)?* y *¿qué hace cada una y cómo (claridad)?* — para que **no sean "una librería que alguien recomendó y no sé cómo funciona"**, sino herramientas cuyo trabajo ya hiciste tú.

## 1. Lo que ya construiste a mano (recuento honesto)

En las secciones 01-11 escribiste, pieza por pieza, un motor de Event Sourcing completo:

| Pieza tuya | Qué hace | Sección |
|---|---|---|
| Eventos `record` | hechos inmutables | §03 |
| `Apply` / Replay (`evolve`) | reconstruir estado desde eventos | §03-04 |
| `AggregateRoot` | motor base + frontera de consistencia | §04 |
| `EventStream<T>` | repositorio lógico de un stream | §05 |
| `EventoAlmacenado` + `Version` | sobre con metadatos + orden | §05-06 |
| `IEventStore` / `InMemoryEventStore` | almacén (diccionario en RAM) | §06 |
| `decide` (métodos del agregado) | reglas de negocio que emiten eventos | §07 |
| `CommandHandler` | orquesta cargar→actuar→guardar | §08 |
| `async` / DI | I/O no bloqueante + ensamblaje | §09-10 |
| PostgreSQL + JSONB | persistencia real | §11 |

**No es poca cosa: ya entiendes Event Sourcing de raíz.** Esa es justo la condición para adoptar herramientas sin volverte dependiente de la magia.

## 2. El dolor: por qué no quieres mantener esto a mano

Llevar tu motor a producción significaría **reescribir, y mantener para siempre**, cosas difíciles y aburridas:

| Tu pieza a mano | El dolor en producción |
|---|---|
| `InMemoryEventStore` | escribir SQL, serializar/deserializar JSON, manejar conexiones y transacciones contra Postgres |
| `Version` | implementar concurrencia optimista correcta (detectar y rechazar conflictos) |
| `CommandHandler` (instanciado a mano) | enrutar 50 comandos a 50 handlers, inyectar dependencias, abrir/cerrar transacción |
| publicar eventos | garantizar entrega (Outbox), reintentos, orden, dead-letter |
| consultar | construir read models y mantenerlos (proyecciones) |

Esto son **meses** de trabajo no diferenciador (no es tu negocio; es plomería). Aquí nace la **necesidad**: no adoptas una librería por moda, la adoptas porque **ya sentiste el dolor que resuelve**.

## 3. Qué es cada herramienta (claridad) y cómo lo hace

Cosmos usa el **"Critter Stack"**: dos librarías del mismo autor (JasperFx) que se integran nativamente. No son intercambiables ni opcionales una de otra; cada una resuelve **una mitad**.

### 🗄️ Marten — la **persistencia**
- **Qué es:** convierte **PostgreSQL** en dos cosas: una **base documental** (guarda objetos C# como JSON) y un **Event Store nativo de producción**.
- **Qué reemplaza de lo tuyo:** `EventoAlmacenado`, `InMemoryEventStore`, `EventStream<T>`, la serialización y la concurrencia optimista.
- **Cómo lo hace (sin magia):** guarda cada evento como una fila **JSONB** en Postgres; para rehidratar, lee el stream y **aplica tus métodos `Apply`** (el mismo `evolve` que escribiste) — y **compila** ese aplicador en vez de usar reflexión lenta. La concurrencia optimista la hace con la **versión esperada** del stream (lo que tú hacías con `Version`).

### 🐺 Wolverine — la **mensajería y el runtime**
- **Qué es:** un **mediator + bus de mensajes** para .NET. Enruta comandos/eventos a sus handlers y gestiona el ciclo de ejecución.
- **Qué reemplaza de lo tuyo:** instanciar handlers a mano, inyectar dependencias, abrir/commitear la transacción, publicar eventos con garantía.
- **Cómo lo hace (sin magia):** **descubre** tus handlers por convención, y **genera código C# inspeccionable** (no reflexión por llamada) que llama tu handler, resuelve sus dependencias y aplica middleware (transacción, etc.). Integrado con Marten, **comparten la misma transacción** y te dan **Outbox/Inbox** durables.

### Cómo encajan
```
        TU CÓDIGO (lógica pura: decide / evolve)
                     │
        ┌────────────┴────────────┐
        ▼                         ▼
   🐺 WOLVERINE              🗄️ MARTEN
   (enruta, ejecuta,         (persiste eventos en
    publica, transacción)     Postgres, rehidrata,
        │                      proyecciones)
        └──────── comparten transacción ────────┘
                     │
              PostgreSQL (JSONB)
```

## 4. El principio (anti-cargo-cult)

> [!IMPORTANT]
> No adoptas Marten/Wolverine porque "un blog las recomienda". Las adoptas porque:
> 1. **Sentiste la necesidad** — ya sufriste a mano el problema que resuelven.
> 2. **Sabes qué hacen** — reemplazan piezas concretas que tú escribiste.
> 3. **Sabes cómo lo hacen** — JSONB + Apply compilado (Marten), descubrimiento + código generado (Wolverine).
> 4. **Puedes depurarlas** — si algo falla, lees el código generado (`dotnet run -- codegen write`, §22) o el SQL que emite Marten; no es una caja negra.
>
> Esa es la diferencia entre *usar* una librería y *dominarla*.

## 5. El trato: qué ganas y qué cedes
- **Ganas:** dejas de mantener plomería; te concentras en `decide`/`evolve` (tu negocio).
- **Cedes:** una dependencia y una "forma de hacer las cosas". Por eso importa entenderlas: cuando la herramienta no haga lo que esperas, sabrás por qué.

---

### El Descubrimiento
El mejor momento para adoptar una herramienta es **después** de haber construido su versión ingenua. Ahora Marten y Wolverine no son magia: son tu `InMemoryEventStore` y tu `CommandHandler` hechos por expertos, listos para producción. En las próximas secciones los conectamos y verás que **cada pieza tiene un equivalente en lo que ya hiciste**.

**Siguiente:** §12 — Marten en detalle (reemplaza tu store) · §13 — Wolverine en detalle (reemplaza tu enrutamiento) · §22 — cómo lo hacen por dentro (codegen).

---

[⬅️ Volver a Docker y PostgreSQL](./11-docker-postgres.md) · [➡️ Introducción a Marten](./12-introduccion-a-marten.md) · [🗺️ Roadmap](../ROADMAP.md)
