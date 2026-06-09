# 07 - Decidir el futuro: Emitir eventos

> 🎯 **Hacia dónde va:** damos el salto de solo leer el pasado a decidir el futuro, aprendiendo a emitir nuevos eventos desde el agregado para agregar capítulos a la historia de Jhon.

Jhon ya tiene una biografía y sabe quién es. Pero la vida sigue, y queremos agregar nuevos capítulos a su historia. Hasta ahora solo hemos visto cómo rehidratar el pasado, pero ¿cómo agregamos nuevos hitos en el presente?

## 🎯 El Objetivo
Jhon sigue viviendo y experimentando nuevos hitos: bodas, mudanzas, nuevos trabajos. Nuestro objetivo es aprender cómo registrar estos nuevos sucesos a través de nuestro objeto `Persona`.

---

## 1. El Nuevo Hito
En Event Sourcing, todo lo que altera la historia se registra como un evento adicional. Vamos a definir un par de nuevos eventos para la vida de Jhon:

```csharp
public record PersonaCasada(Guid PersonaId, string NombrePareja);
public record PersonaMudada(Guid PersonaId, string NuevaCiudad);
```

## 2. Es quien autoriza agregar eventos a su biografía
Aquí hay un detalle clave (y muy sutil) en Event Sourcing: La clase `Persona` es la dueña de las **reglas** de su historia, pero sorprendentemente **NO almacena la lista físicamente**. Si te fijas, su constructor lee los eventos del pasado para rehidratarse, pero nunca guarda una referencia directa a la `Lista<object> biografia`.

Por lo tanto, a Jhon no le "insertamos" eventos a la fuerza; nosotros le pedimos que evalúe realizar una acción y él, a cambio, **emite un nuevo evento** como resultado. Alguien más en el sistema será el encargado de tomar ese evento y guardarlo en el baúl para el futuro.

Por ahora, como estamos en el "camino feliz", asumimos que Jhon acepta generar cualquier evento nuevo sin poner condiciones. Añadamos esta lógica a la clase `Persona`:

```csharp
public class Persona : AggregateRoot
{
    public string NombrePareja { get; private set; }
    // Asumimos que la propiedad Ciudad ya existe desde la refactorización anterior

    // ... propiedades, constructor y métodos Apply básicos ...

    public PersonaCasada Casar(string nombrePareja)
    {
        // Generamos un nuevo hito para la biografía
        return new PersonaCasada(this.Id, nombrePareja);
    }

    public PersonaMudada Mudarse(string nuevaCiudad)
    {
        // En una aplicación real, aquí validaríamos las reglas de negocio usando el estado rehidratado
        return new PersonaMudada(this.Id, nuevaCiudad);
    }

    // El motor actualiza el estado cuando ESTOS eventos ocurren en el pasado
    private void Apply(PersonaCasada c) => NombrePareja = c.NombrePareja;
    private void Apply(PersonaMudada m) => Ciudad = m.NuevaCiudad;
}
```

## 3. Enseñando al Stream a escribir

En las secciones anteriores, diseñamos nuestro `EventStream<T>` exclusivamente como un experto lector para rehidratar el pasado. Pero ahora que nuestra `Persona` genera nuevos Eventos de Dominio en el presente, necesitamos que el Stream tenga la capacidad inversa: **escribir y guardar**.

Añadamos por primera vez el método `Append` a nuestro `EventStream<T>`. Este método recibirá un Evento de Dominio "puro", lo empaquetará en su respectivo sobre (`EventoAlmacenado`) protegiendo el orden de la historia (`_version++`), y se lo entregará firmemente al vigilante (`IEventStore`):

```csharp
public class EventStream<T> where T : AggregateRoot, new()
{
    // ... código de lectura previo (Get) y variables (_store, _aggregateId, _version) ...

    public void Append(object domainEvent)
    {
        _version++; // Nueva página/versión en la biografía

        var eventoAlmacenado = new EventoAlmacenado(
            AggregateId: _aggregateId,
            Version: _version,
            Timestamp: DateTime.UtcNow,
            EventData: domainEvent
        );

        _store.AppendEvent(eventoAlmacenado);
    }
}
```

## 4. 🛠️ El Ciclo de Vida en el Program.cs
Vamos a integrar finalmente esta capacidad bidireccional de lectura y escritura:

```csharp
// 0. Instanciamos el almacén (asumimos que 'store' ya existe)
var streamJhon = new EventStream<Persona>(store, idJhon);

// 1. CARGAMOS: Traemos a Jhon a la vida desde el almacén central
var jhon = streamJhon.Get();
Console.WriteLine($"[ANTES] Jhon vive en {jhon.Ciudad}");

// 2. ACTUAMOS: Le pedimos a Jhon que registre un nuevo hito en su vida
var eventoMudanza = jhon.Mudarse("Nueva York");

// 3. GUARDAMOS: Enviamos el evento de vuelta al stream para que se guarde
streamJhon.Append(eventoMudanza);

// Para ver el resultado final, rehidratamos a Jhon DESDE CERO 
var streamDeVerificacion = new EventStream<Persona>(store, idJhon);
var jhonActualizado = streamDeVerificacion.Get();

Console.WriteLine($"[DESPUÉS] Jhon vive ahora en {jhonActualizado.Ciudad}.");
```

### El Descubrimiento
Acabas de ver el flujo básico para interactuar con el dominio:
1. **Cargar**: Recuperas el pasado (la biografía).
2. **Rehidratar**: Pones al objeto en su estado actual (haciendo el Replay de su historia).
3. **Actuar**: Le pides al objeto que realice una acción y genere un **nuevo evento**.

> [!TIP]
> Intuitivamente, cada acción que le pides a Jhon (ej. pedirle que se case) es una petición que le haces al sistema. A esa intención de hacer algo se le llama **Comando** (lo formalizaremos como `RegistrarMatrimonio` en §08); el método del agregado que la ejecuta es `Casar`.
> Aquí vemos una regla de oro: **Los Comandos son los encargados de generar los Eventos** (siempre a través del Aggregate Root).

> [!NOTE]
> 🌱 **Semilla — los tres tipos de mensaje (no los confundas).** Desde ahora vas a manejar tres cosas distintas que es fácil mezclar. Grábate la diferencia:
>
> | Tipo | Qué es | Cardinalidad | ¿Se puede rechazar? | Tiempo verbal |
> |------|--------|--------------|---------------------|---------------|
> | **Comando** | una *intención* ("haz esto") | **1** destinatario (un handler) | **Sí** (puede fallar una regla) | imperativo: `RegistrarMatrimonio` |
> | **Evento** | un *hecho* que ya ocurrió | **0..N** interesados | **No** (ya pasó, es irrechazable) | pasado: `PersonaCasada` |
> | **Query** | una *pregunta* (pedir datos) | 1, devuelve resultado | n/a (no muta nada) | `ObtenerPersona` |
>
> El error clásico: tratar un "evento" que en realidad tiene **un solo dueño obligado** → eso era un comando disfrazado. Las **queries** las veremos a fondo en CQRS (§20); por ahora basta saber que **leer ≠ escribir**.

> [!NOTE]
> 🌱 **Semilla — Acabas de escribir la función `decide`.** En la Sección 03 viste `evolve` (estado + evento → estado). Aquí `Mudarse` (y `Casar`) hacen la otra mitad: **`decide`** (estado + comando → eventos). Juntas forman el **patrón Decider**, el modelo funcional del Event Sourcing: `decide` valida y *decide qué pasó*, `evolve` *aplica lo que pasó*. Ambas son puras → se testean sin base de datos. Marten + Wolverine se montan justo sobre este par.

---

## 🧬 Evolucionemos el código: ¿y si el comando llega dos veces?

Hasta aquí, `Casar` está en el "camino feliz": **siempre** emite el evento. Probemos qué pasa en la vida real, donde un comando puede reintentarse (la red falló, el usuario hizo doble clic):

```csharp
// 🟢 Lo ingenuo (lo que tenemos ahora)
public PersonaCasada Casar(string nombrePareja)
{
    return new PersonaCasada(this.Id, nombrePareja);
}
```

```csharp
// 💥 El dolor: el mismo comando llega dos veces
jhon.Casar("María");   // emite PersonaCasada
jhon.Casar("María");   // emite PersonaCasada OTRA VEZ
// La biografía de Jhon ahora dice que se casó dos veces con María.
// Al rehidratar, Apply(PersonaCasada) corre dos veces → estado corrupto.
```

El agregado es el **guardián de las reglas** (lo dijimos arriba). Así que la defensa nace donde debe: dentro de la `Persona`, validando su estado **antes** de emitir.

```csharp
// 🔧 El refactor: el agregado protege sus reglas ANTES de emitir
public class Persona : AggregateRoot
{
    public bool Casado { get; private set; }          // ← estado que vigilamos
    public int  Edad   { get; private set; }          // ← lo actualiza Apply(CumpleañosCelebrado)

    public PersonaCasada? Casar(string nombrePareja)
    {
        // (a) VALIDACIÓN — regla de negocio violada → se RECHAZA (esto sí es un error)
        if (Edad < 18)
            throw new ReglaDeNegocioException("No se puede casar a un menor de edad.");

        // (b) IDEMPOTENCIA — comando repetido sobre un estado ya alcanzado → NO-OP (no es un error)
        if (Casado)
            return null;   // ya está casado: no emitimos un evento duplicado

        return new PersonaCasada(this.Id, nombrePareja);
    }

    private void Apply(PersonaCasada c) { NombrePareja = c.NombrePareja; Casado = true; }
}
```

> [!IMPORTANT]
> 🏷️ **Dos motivos distintos para NO emitir un evento — no los confundas:**
> - **Validación (rechazar):** la operación es **inválida** (un menor no puede casarse). Es un **error**: lanzas una excepción (o devuelves un fallo) para que el llamador se entere. *No* debes "tragarte" una regla violada.
> - **Idempotencia (no-op):** la operación es **válida pero redundante** (ya estaba casado; el comando llegó dos veces). **No es un error**: simplemente no emites un evento duplicado y sigues.
>
> Ambas viven en el agregado (el **guardián de las reglas**), pero se comportan distinto: una grita, la otra calla. Más adelante, en mensajería, veremos la **segunda línea** de la idempotencia: deduplicación por *id de mensaje* (Inbox).

---

> [!NOTE]
> 🌱 **Semilla — devolver el evento en vez de publicarlo: "cascading messages".** Fíjate en un detalle de estilo: `Casar` **devuelve** el evento; no lo guarda ni lo publica por su cuenta. Eso es deliberado y Wolverine lo eleva a patrón con el nombre **cascading messages**: tu handler **devuelve** los mensajes/eventos que deben ocurrir, y el framework se encarga de publicarlos. ¿Por qué importa? Porque mantiene la lógica **pura** (no inyectas el bus, no escondes envíos en el fondo del call stack) y hace evidente, leyendo el método, *qué efectos* produce. Lo veremos a fondo en el Aggregate Handler (§18).

---

## 🧪 Empieza a testear DESDE YA (no lo dejes para el final)

`RegistrarMatrimonio` es una **función pura**: recibe el estado (eventos pasados) y un comando, y decide un evento. Eso significa que **ya puedes testearla** — sin base de datos, sin mocks, en microsegundos. Adopta el hábito desde esta sección:

> [!NOTE]
> 🔤 **¿Sintaxis nueva en los tests?** `[Fact]` marca un método como test (xUnit) y `resultado.Should()...` son aserciones fluidas (AwesomeAssertions). Si alguno de estos símbolos te frena —`[Fact]`, `.Should()`, lambdas `() => ...`, `params`— están en el [GLOSARIO](../GLOSARIO.md). Lo profundizamos en §21; aquí solo léelos como "monto la historia → ejecuto → verifico".

```csharp
// La edad de Jhon se deriva de sus cumpleaños (§03); para "Jhon adulto" montamos su historia.
static Persona JhonAdulto(params object[] extra)
{
    var historia = new List<object> { new PersonaNacida("Jhon", new DateTime(1990,5,10), "Bogotá") };
    historia.AddRange(Enumerable.Repeat<object>(new CumpleañosCelebrado(), 18)); // → 18 años
    historia.AddRange(extra);
    return new Persona(historia);
}

[Fact]
public void Casar_a_un_adulto_soltero_emite_PersonaCasada()
{
    var jhon = JhonAdulto();                          // Given
    var evento = jhon.Casar("María");   // When (decide)
    evento.Should().BeOfType<PersonaCasada>();        // Then
}

[Fact]
public void Casar_a_un_menor_es_RECHAZADO()           // validación: la regla se rechaza
{
    var jhon = new Persona(new object[] { new PersonaNacida("Jhon", new DateTime(1990,5,10), "Bogotá") }); // Edad 0
    var act = () => jhon.Casar("María");
    act.Should().Throw<ReglaDeNegocioException>();
}

[Fact]
public void Casar_a_alguien_ya_casado_no_emite_nada() // idempotencia: no es error, es no-op
{
    var jhon = JhonAdulto(new PersonaCasada(idJhon, "María"));
    jhon.Casar("Ana").Should().BeNull();
}
```

> [!TIP]
> 🌱 Este patrón **Given → When → Then** (historia previa → ejecutar → verificar el evento) será tu forma de testear todo el workshop. No esperes a una "fase de testing": cada vez que escribas un `decide` o un `evolve`, **escribe su test al lado**. Lo profundizamos en §21 (incluida la base `CommandHandlerTestBase` de Cosmos), pero el hábito empieza aquí.

---

[⬅️ Volver a la sección anterior](./06-el-almacen-en-memoria.md)

[➡️ Siguiente sección: El Command Handler](./08-el-command-handler.md)
