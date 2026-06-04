# 19 - Versionado de Eventos: cómo cambiar lo que es inmutable

> 🌳 **Sección donde se domina** la semilla de §03 ("los eventos son eternos") y §12 ("Marten los guarda como JSON"). Este es el problema que rompe sistemas en producción y que casi nadie enseña.

## El problema que tarde o temprano vas a tener

En la Sección 03 grabaste a fuego: **un evento es un contrato inmutable con el futuro**. `PersonaNacida` quedó guardado como JSON en miles de streams:

```json
{ "Nombre": "Jhon", "FechaNacimiento": "1990-05-10", "Ciudad": "Bogotá" }
```

Seis meses después, el negocio pide: *"ahora necesitamos el **País** de nacimiento"*. Tu instinto es editar el record:

```csharp
public record PersonaNacida(string Nombre, DateTime FechaNacimiento, string Ciudad, string Pais); // ⚠️
```

Compila. Pero el día que arranques la app y Marten intente **deserializar los millones de eventos viejos** (que no tienen `Pais`), te enfrentas a la realidad: **no puedes editar el pasado**. El JSON viejo no tiene ese campo.

> [!IMPORTANT]
> Regla de oro del Event Sourcing: **un evento publicado nunca se modifica ni se borra.** Solo puedes *evolucionar cómo lo lees*. Esto se llama **versionado de eventos**.

---

## Estrategia 1 — Esquema débil (weak schema): la más barata

Si el cambio es **aditivo** (agregas un campo opcional), no necesitas casi nada: que el campo nuevo tenga un **valor por defecto** para los eventos viejos.

```csharp
// El campo nuevo es opcional; los eventos viejos lo deserializan como null/valor por defecto
public record PersonaNacida(string Nombre, DateTime FechaNacimiento, string Ciudad, string? Pais = null);
```

Al leer un evento viejo, `Pais` será `null`; al escribir uno nuevo, lo llenas. **Regla:** los cambios aditivos y opcionales son seguros. Esto resuelve el 80% de los casos.

> [!WARNING]
> Lo que **NO** es seguro: **renombrar** un campo, **cambiar su tipo**, **quitar** un campo que el código aún usa, o **cambiar el significado** de un evento. Eso es un cambio *de ruptura* → necesitas la Estrategia 2.

---

## Estrategia 2 — Upcasting: transformar el evento viejo al leerlo

Cuando el cambio rompe el esquema, mantienes el evento viejo como un tipo `V1` y defines un **upcaster**: una función que transforma la versión vieja en la nueva **en el momento de leer**, sin tocar lo guardado.

```csharp
// Conservas la forma vieja...
public record PersonaNacidaV1(string Nombre, DateTime FechaNacimiento, string Ciudad);
// ...y la nueva, que es la que usa tu dominio hoy
public record PersonaNacida(string Nombre, DateTime FechaNacimiento, string Ciudad, string Pais);
```

En Marten registras el **upcaster** (transformación de `V1` → actual). Conceptualmente:

```csharp
builder.Services.AddMarten(options =>
{
    // Marten: al leer un PersonaNacidaV1 del JSON, transfórmalo a PersonaNacida
    options.Events.Upcast<PersonaNacidaV1, PersonaNacida>(
        old => new PersonaNacida(old.Nombre, old.FechaNacimiento, old.Ciudad, Pais: "Desconocido"));
});
```

Ahora tu dominio **solo conoce `PersonaNacida`**; los eventos viejos se "elevan" (upcast) al vuelo. El JSON en disco sigue intacto. (Marten documenta esto en *Event Versioning*; soporta upcasting por tipo y por transformación de JSON crudo para casos avanzados.)

---

## Estrategia 3 — Nombres de evento estables (event type aliases)

Marten serializa el **nombre del tipo** junto al evento. Si renombras la clase C# (`PersonaNacida` → `ClienteRegistrado`), Marten ya no sabría mapear el JSON viejo. Solución: **fijar un alias estable** desacoplado del nombre de la clase, para poder refactorizar el código sin romper la lectura.

```csharp
options.Events.MapEventType<PersonaNacida>("persona_nacida"); // alias estable en disco
```

Así renombras la clase libremente; el nombre persistido no cambia.

---

## La dimensión crítica en Cosmos: eventos públicos

Recoge la semilla de §01 (domain vs integration). Versionar un **evento de dominio** (privado) es asunto tuyo. Pero versionar un **evento de integración** (`IPublicEvent`) es **un cambio de contrato con otros Bounded Contexts** que ya lo consumen:

> [!IMPORTANT]
> Un `IPublicEvent` es una **API pública**. Otros equipos (Contabilidad, Impuestos) dependen de su forma. Cambiarlo de ruptura sin coordinar **rompe sus consumidores**. Reglas:
> - Para públicos, prefiere **solo cambios aditivos** (esquema débil).
> - Si hay ruptura, **versiona el evento** (`OrdenAprobadaV2`) y publica ambos un tiempo, o usa upcasting del lado consumidor.
> - Nunca reutilices un nombre de evento público para significar algo distinto.

---

## Checklist de supervivencia

| Cambio | ¿Seguro? | Qué hacer |
|--------|----------|-----------|
| Agregar campo opcional | ✅ | Esquema débil (valor por defecto) |
| Renombrar la **clase** C# | ✅ con cuidado | Alias estable (`MapEventType`) |
| Renombrar/quitar/cambiar tipo de un **campo** | ❌ ruptura | Upcaster `V1`→actual |
| Cambiar el **significado** del evento | ❌ nunca | Crea un evento nuevo distinto |
| Borrar un evento del store | ❌ jamás | Está prohibido; archiva/compacta si hace falta |

---

### El Descubrimiento
"Inmutable" no significa "no puede evolucionar": significa que **evolucionas la lectura, no lo escrito**. Esquema débil para lo aditivo, upcasting para lo de ruptura, alias para refactorizar nombres. Y en eventos públicos, trata cada cambio como lo que es: **una versión de una API**.

**Siguiente:** ya sabes escribir y evolucionar el lado de escritura. Ahora el lado de lectura a escala — **CQRS y proyecciones** (la semilla de §06).

---

[⬅️ Volver a Decider y Aggregate Handler](./18-decider-y-aggregate-handler.md) · [🗺️ Roadmap](../ROADMAP.md)
