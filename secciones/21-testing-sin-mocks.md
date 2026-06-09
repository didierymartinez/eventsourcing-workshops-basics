# 21 - Testing sin mocks: el superpoder oculto del Event Sourcing

> 🎯 **Hacia dónde va:** aprovechamos las funciones puras del patrón Decider para escribir tests Given-When-Then sin mocks, mostrando que el Event Sourcing bien hecho es de lo más testeable que existe.

> 🌳 **Sección donde se domina** la semilla de §08 ("este diseño es un regalo para los tests"). El Event Sourcing, bien hecho, es de lo más testeable que existe — y casi **sin mocks**.

## Por qué el Event Sourcing es tan testeable

Recoge el patrón Decider (§18): tu lógica de negocio son **funciones puras**.
- `decide(estado, comando) -> eventos`
- `evolve(estado, evento) -> nuevo estado`

Una función pura es **un sueño para testear**: mismas entradas → mismas salidas, sin base de datos, sin red, sin reloj, sin aleatoriedad. No necesitas simular (mock) infraestructura porque **no hay infraestructura en la lógica**.

Y como todo en Event Sourcing se expresa en eventos, el test tiene una forma natural y legible: **Given-When-Then** (Dado-Cuando-Entonces), el lenguaje del BDD.

```
Given  → la historia previa (eventos que ya ocurrieron)
When   → ejecuto un comando
Then   → espero que se emitan ciertos eventos (y/o que el estado quede así)
```

---

## Nivel 1 — Testear el agregado directamente (cero dependencias)

El agregado no toca infraestructura, así que se testea instanciándolo y verificando los eventos que emite:

```csharp
// Helper: la edad de Jhon se deriva de sus cumpleaños (§03), así que para "Jhon adulto"
// montamos su historia con suficientes CumpleañosCelebrado.
static Persona JhonAdulto(params object[] extra)
{
    var historia = new List<object> { new PersonaNacida(id, "Jhon", new DateTime(1990,5,10), "Bogotá") };
    historia.AddRange(Enumerable.Repeat<object>(new CumpleañosCelebrado(), 18)); // → 18 años
    historia.AddRange(extra);
    var jhon = new Persona();
    jhon.Load(historia); // rehidratar (evolve)
    return jhon;
}

[Fact]
public void Casar_a_un_adulto_soltero_emite_PersonaCasada()
{
    var jhon = JhonAdulto();                          // Given: Jhon adulto y soltero
    var evento = jhon.Casar("María");   // When: lo casamos (decide)
    evento.Should().BeOfType<PersonaCasada>();        // Then: el hecho correcto
}

[Fact]
public void Casar_a_un_menor_es_RECHAZADO()           // validación (regla de negocio)
{
    var jhon = new Persona();
    jhon.Load(new object[] { new PersonaNacida(id, "Jhon", new DateTime(1990,5,10), "Bogotá") }); // Edad = 0

    var act = () => jhon.Casar("María");
    act.Should().Throw<ReglaDeNegocioException>();     // se rechaza, no se traga la regla
}

[Fact]
public void Casar_a_alguien_ya_casado_NO_emite_nada() // idempotencia (no es error)
{
    var jhon = JhonAdulto(new PersonaCasada(id, "María"));
    jhon.Casar("Ana").Should().BeNull(); // no-op silencioso
}
```

Sin mocks. Sin base de datos. Microsegundos por test. Fíjate que los **tres** casos —éxito, validación rechazada, e idempotencia— se prueban igual de fácil porque `decide` es puro.

---

## Nivel 2 — Testear el handler con un store de prueba

El handler sí orquesta (carga/guarda), pero no necesitas Postgres: usas un **event store en memoria de prueba**. La plantilla de Cosmos lo trae listo en `Cosmos.EventSourcing.Testing.Utilities` con la clase base **`CommandHandlerTestBase`**:

```csharp
public class RegistrarMatrimonioHandlerTests : CommandHandlerTestBase
{
    [Fact]
    public async Task Casar_a_Jhon_emite_PersonaCasada()
    {
        // Given: historia previa en el TestStore
        Given(new PersonaNacida(GuidAggregateId, "Jhon", new DateTime(1990,5,10), "Bogotá"));

        // When: ejecuto el handler real
        var handler = new RegistrarMatrimonioHandler(EventStore);
        await handler.HandleAsync(new RegistrarMatrimonio(GuidAggregateId, "María"), default);

        // Then: el evento esperado fue emitido
        Then(new PersonaCasada(GuidAggregateId, "María"));

        // And: (opcional) el estado proyectado quedó así
        And<Persona, bool>(p => p.Casado, true);
    }
}
```

Bajo el capó, `CommandHandlerTestBase` usa un `TestStore`, un `TestPrivateEventSender` y un `TestPublicEventSender` que capturan lo emitido, y compara con **AwesomeAssertions** sobre **xUnit** (las dependencias declaradas en la plantilla). `Given` precarga eventos; `Then` verifica los eventos nuevos; `And<T,P>` verifica una propiedad del agregado rehidratado.

---

## Por qué evitamos los mocks (no es dogma, es la best-practice oficial)

La documentación de Wolverine es explícita: *prefiere funciones puras y evita el uso de mocks*. ¿Por qué?

> [!NOTE]
> Un test con muchos mocks termina verificando **cómo** se llamó a las dependencias (interacciones), no **qué** resultó. Eso lo hace frágil: cualquier refactor interno rompe el test aunque el comportamiento sea correcto. Testear con **eventos de entrada → eventos de salida** verifica el **comportamiento observable**, que es lo que de verdad importa. El Event Sourcing te da esto gratis porque el resultado de una operación *son* eventos.

Esto conecta con la "A-Frame Architecture" que recomienda Wolverine: la decisión (pura) en el centro, la infraestructura en los bordes. Testeas el centro sin tocar los bordes.

---

## Nivel 3 — Testear proyecciones

Una proyección (§20) también es determinista: dados unos eventos, produce una vista. Se testea igual: alimentas eventos, reconstruyes la vista, verificas.

```csharp
[Fact]
public void Resumen_refleja_el_estado_tras_casarse()
{
    var resumen = new PersonaResumen();
    resumen.Apply(new PersonaNacida(id, "Jhon", new DateTime(1990,5,10), "Bogotá"));
    resumen.Apply(new PersonaCasada(id, "María"));

    resumen.Casado.Should().BeTrue();
}
```

> [!TIP]
> Para tests de integración reales (con Postgres efímero en Docker), Marten documenta utilidades específicas; pero el **grueso** de tu cobertura debe ser de estos tests puros, rápidos y sin infraestructura. Los de integración son la guinda, no la base.

---

### El Descubrimiento
El Event Sourcing convierte el testing en algo natural: **eventos entran, eventos salen**. Si mantienes la lógica en agregados puros y handlers delgados (§08, §18), pruebas casi todo sin mocks ni base de datos, con la legibilidad del Given-When-Then. La plantilla de Cosmos ya te da la base (`CommandHandlerTestBase`); úsala.

**Siguiente:** el último velo de "magia" por levantar — cómo el framework hace todo esto sin reflexión en cada llamada: **Reflexión vs Generación de Código**.

---

[⬅️ Volver a CQRS y Proyecciones](./20-cqrs-y-proyecciones.md) · [🗺️ Roadmap](../ROADMAP.md)
