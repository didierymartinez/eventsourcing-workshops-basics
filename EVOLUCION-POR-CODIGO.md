# 🧬 Evolución por Código: de la semilla al patrón

> El método más potente del workshop: **escríbelo ingenuamente → siente el dolor en carne propia → refactoriza → ahí *nace* el patrón**. El concepto no se presenta como teoría, se *descubre* como solución a un problema que el alumno ya sufrió en su propio código.
>
> Este documento revisa **cada semilla** y decide: ¿conviene volverla un arco de código? ¿con qué naive → dolor → refactor?

---

## ⚠️ Principio rector: el ejemplo es el hilo conductor (no solo texto)

Una semilla **no debe quedarse en una caja de texto** si el concepto se puede **mostrar en código evolucionando el ejemplo** (Jhon/`Persona`, y luego `Orden` en Cosmos). La regla:

- **Si es codeable en el ejemplo ahora → arco de código** (ingenuo → bug → refactor en la misma `Persona`/`Orden`). El concepto se *descubre*, no se anuncia.
- **Solo queda como nota 🌱** cuando el concepto depende de algo que aún no existe en el hilo (p. ej. versionado de eventos necesita Marten/JSON; codegen necesita el framework). Y aun así, cuando llega su sección, se muestra con código.

> El hilo conductor manda: cada concepto nuevo **mueve el mismo ejemplo un paso adelante**, no abre un ejemplo de juguete aparte.

### Estado de conversión (semilla-texto → arco-de-código)
| Concepto | Antes | Ahora |
|---|---|---|
| Idempotencia (§07) | nota | ✅ **arco de código** sobre `Persona.RegistrarMatrimonio` (naive→doble boda→guard) |
| Concurrencia optimista (§06) | nota | ✅ **arco de código** en `AppendEvent` (dos procesos→lost update→chequeo de `Version`/`ConcurrencyException`) |
| Domain vs Integration (§14) | nota | ✅ **arco de código** (publicar todo→acoplamiento→marcar `IPrivate/IPublicEvent` + `GetPublic/PrivateEvents()`) |
| DI / mini-contenedor (§10) | nota | ✅ **arco de código** (mini-contenedor de ~20 líneas: diccionario+reflexión+recursión) |
| Cascading messages (§07) | nota | ✅ ya es código (el agregado devuelve el evento) |
| Middleware (§23) | — | ✅ arco completo |
| Versionado (§19), Codegen (§22), Outbox (§14) | — | ✅ ya con código en su sección |

---

## El patrón del arco (plantilla)

Cada arco tiene 4 momentos:
1. **🟢 Lo ingenuo:** el código que escribiría cualquiera sin saber el patrón. Funciona… aparentemente.
2. **💥 El dolor:** un escenario concreto (con código o traza) donde el ingenuo falla o duele.
3. **🔧 El refactor:** la transformación mínima que resuelve el dolor.
4. **🏷️ El nombre:** recién aquí se revela "esto que acabas de hacer se llama *X*", y se conecta con el framework.

> La columna del workshop (Jhon) ya usa esto: §03 variables sueltas → §04 `AggregateRoot`; §08 handler en `Program.cs` → `ICommandHandler<T>`. Lo extendemos a las semillas avanzadas.

---

## Revisión semilla por semilla

| Semilla | ¿Arco de código? | Naive → Dolor → Refactor (resumen) | Dónde |
|---------|------------------|-----------------------------------|-------|
| **Polimorfismo / dispatch** | ✅ **Ideal** | `if/else is` por tipo → cadena interminable + olvidas un caso → `switch` tipado / `Apply` por sobrecarga | fund. §03 / main §04 (ya parcial) |
| **Idempotencia** | ✅ **Ideal** | handler emite siempre → doble clic = Jhon casado dos veces → guard por estado + dedup por Id | main §07 → arco |
| **Concurrencia optimista** | ✅ **Ideal** | dos handlers leen v5, ambos escriben v6 → *lost update* → chequeo de `Version` / `FetchForWriting` | main §06 → arco |
| **Composición / Middleware** | ✅ **Insignia** | log+validación copiado en cada handler → cambias el log y tocas 50 archivos → pipeline con `Func` → "esto es middleware" | nueva sección |
| **Domain vs Integration events** | ✅ Bueno | publicas TODO evento al bus → otro BC se acopla a un campo interno → separar `IPrivateEvent`/`IPublicEvent` | main §09/§01 → arco |
| **DI / contenedor** | ✅ (ya existe) | `new` anidados → infierno al añadir dependencia → contenedor; + mini-contenedor a mano (20 líneas) | main §10 (ampliar con mini-contenedor) |
| **Event versioning** | ✅ (ya en §19) | editas el record → revienta al deserializar lo viejo → upcaster | §19 (ya tiene código) |
| **Outbox** | ✅ (ya en §14) | guardar + publicar en dos pasos → falla a la mitad → outbox transaccional | §14 (ya tiene código) |
| **CQRS / proyección** | ✅ Bueno | consulta reproduciendo todos los streams → lento/imposible → proyección | §06/§20 → arco corto |
| **Records / inmutabilidad** | ✅ (ya en fund.§01) | `class` con setters mutada en tránsito → bug → `record` | fund. §01 (ya) |
| **async / starvation** | ⚠️ Difícil de "sentir" en demo | mostrar con un benchmark de hilos (opcional) | main §09 (mantener como aviso + mini-demo opcional) |
| **Genéricos** | ✅ (ya en §05) | `object` + casts → error en runtime → `<T>` con restricción | §05 (ya) |
| **Reflexión vs codegen** | ✅ (ya en §22) | mediator por reflexión → "no veo qué pasa" → leer código generado | §22 (ya tiene código) |
| **Bounded context** | ❌ Conceptual | difícil de codear; mantener como mapa/diagrama | §01/§10-bis |
| **Lenguaje ubicuo** | ❌ Conceptual | se muestra con naming, no con arco | fund. §07 (ya) |

**Veredicto:** los arcos de mayor impacto que faltan por construir con código completo son: **Composición/Middleware** (insignia, nueva sección), **Idempotencia** (§07), **Concurrencia** (§06), **Domain vs Integration** (§09), y el **mini-contenedor de DI a mano** (§10). El resto ya tiene código o es mejor dejarlo conceptual.

---

## Orden recomendado de implementación
1. 🏷️ **Composición → Middleware** *(insignia — ver sección nueva 23)* ✅ construida como demostración.
2. Idempotencia (arco en §07).
3. Concurrencia optimista (arco en §06).
4. Mini-contenedor de DI a mano (ampliar §10).
5. Domain vs Integration (arco en §09).

> Estado: la sección insignia ya está escrita como ejemplo del método. Las demás se irán implementando con el mismo arco de 4 momentos.
