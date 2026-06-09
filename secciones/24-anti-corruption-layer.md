# 24 - Anti-Corruption Layer: traducir el evento de otro a tu propio idioma

> 🎯 **Hacia dónde va:** resolvemos una decisión clave de producción en Cosmos —cuando llega el evento de otro servicio, ¿manejarlo directo o traducirlo?— construyendo un Anti-Corruption Layer que protege tu propio idioma.

> 🌳 **Sección donde se domina** la semilla de §01/§14 (domain vs integration). Aquí resolvemos la pregunta real de producción: *cuando llega un evento de OTRO servicio, ¿lo manejo directo o lo traduzco?* Es una de las decisiones de diseño más importantes de Cosmos.

## El escenario
Seguimos con nuestro elenco (§01b). El BC **Registro Civil** escucha una cola. Llega un evento **público** del BC **Biografías**: `MatrimonioCelebrado`. Registro Civil quiere reaccionar (inscribir el matrimonio oficialmente).

## 🟢 Lo ingenuo: manejar el evento público directamente

```csharp
// En Registro Civil: handler que reacciona DIRECTAMENTE al evento de Biografías
public static async Task Handle(MatrimonioCelebrado evento, IDocumentSession session)
{
    var acta = await session.Events.FetchForWriting<ActaMatrimonio>(evento.PersonaId);
    acta.Aggregate.Inscribir(evento.NombrePareja, evento.Fecha); // uso campos del evento ajeno
    // ...
}
```

Funciona hoy. Pero acabas de **acoplar el dominio de Registro Civil a la firma de un evento que no controla** (lo controla Biografías).

## 💥 El dolor
El equipo de Biografías —otro equipo, otro repo, otro ciclo de despliegue— decide renombrar `NombrePareja` a `NombreConyuge`, o partir `MatrimonioCelebrado` en dos eventos. **Registro Civil se rompe** y ni se enteró del cambio. Peor: el evento podría llegar **mal formado** (un `PersonaId` vacío) y tu agregado lo procesa igual, corrompiendo tu estado.

> El problema de fondo: un **evento público es un contrato de OTRO**. Si tu lógica de negocio depende directamente de su forma, cada cambio de ellos es un riesgo para ti.

## 🔧 El refactor: una capa que valida y traduce

Metemos una capa intermedia cuyo único trabajo es: **recibir el evento externo, validarlo, y traducirlo a un comando de TU propio dominio**. Tu lógica de negocio nunca ve el evento ajeno; solo ve *tu* comando.

```csharp
// 1. TU comando interno, en TU lenguaje ubicuo (Registro Civil habla de "Ciudadano", no "Persona")
public record InscribirMatrimonio(Guid CiudadanoId, string Conyuge, DateOnly FechaActa);

// 2. La Anti-Corruption Layer: traduce el evento externo -> comando interno
public static class BiografiasAcl
{
    public static async Task Handle(MatrimonioCelebrado externo, IMessageBus bus)
    {
        // a) VALIDA que el evento externo venga bien informado
        if (externo.PersonaId == Guid.Empty || string.IsNullOrEmpty(externo.NombrePareja))
            throw new EventoInvalidoException("MatrimonioCelebrado mal formado"); // no contaminamos el dominio

        // b) TRADUCE a TU idioma y despacha un comando interno
        await bus.InvokeAsync(new InscribirMatrimonio(
            CiudadanoId: externo.PersonaId,   // "Persona" (Biografías) -> "Ciudadano" (Registro Civil)
            Conyuge:     externo.NombrePareja,      // el mapeo vive AQUÍ, aislado
            FechaActa:   externo.Fecha));
    }
}

// 3. TU handler de negocio solo conoce TU comando — desacoplado del mundo exterior
public static IEnumerable<object> Handle(InscribirMatrimonio cmd, [Aggregate] ActaMatrimonio acta)
{
    if (acta.Inscrita) yield break;
    yield return new MatrimonioInscrito(cmd.CiudadanoId, cmd.Conyuge, cmd.FechaActa);
}
```

Ahora, si Biografías renombra su campo, **solo cambias una línea en la ACL** (el mapeo). El dominio de Registro Civil ni se entera.

## 🏷️ El nombre y la regla
Esa capa intermedia es el **Anti-Corruption Layer (ACL)**, un patrón de DDD: una frontera que impide que el modelo/lenguaje de otro contexto "se cuele" y corrompa el tuyo. La regla práctica del equipo de Cosmos:

| Origen del evento | ¿Manejar directo? | Por qué |
|---|---|---|
| **Evento privado** (de tu propio BC) | ✅ Sí, handler directo | Controlas la firma; mismo equipo, mismo repo |
| **Evento público** (de otro BC) | ❌ No: pásalo por una **ACL** | No controlas su contrato; debes validar y traducir a comando interno |

> [!IMPORTANT]
> 🪐 **Postura de Cosmos.** Los eventos **privados** (`IPrivateEvent`) pueden tener Event Handlers directos. Los **públicos** (`IPublicEvent`) deberían pasar por una ACL que valide y traduzca a un comando interno — *"para evitar que se nos metan cosas que no estamos esperando"*. Así, el acoplamiento entre Bounded Contexts queda confinado a una capa fina y explícita, no esparcido por el dominio.

## ¿Y la sobrecarga de "siempre traducir"?
Sí, traducir evento→comando es un paso extra. Pero compra **independencia**: tu dominio evoluciona a su ritmo y los cambios de otros equipos no te rompen. Para lo privado te lo ahorras (handler directo); para lo público, el costo vale la tranquilidad.

---

### El Descubrimiento
La ACL responde la pregunta detonante: *no manejes directamente el evento de otro servicio*. Recíbelo en una capa-frontera que **valida** (que venga bien informado) y **traduce** a un comando de tu propio lenguaje. Tu núcleo de negocio nunca toca firmas ajenas. Es la versión "mensajería" de la misma idea que ya viste: **proteger el dominio de la infraestructura y de lo externo**.

**Siguiente:** si el comando interno se despacha por la cola, ¿cómo viaja el *tenant* y el *usuario* sin meterlos en el mensaje? Eso es el **Envelope** (§25).

---

[⬅️ Volver a Outbox](./14-outbox.md) · [🗺️ Roadmap](../ROADMAP.md) · [🏛️ La Plantilla Cosmos](./27-plantilla-cosmos.md)
