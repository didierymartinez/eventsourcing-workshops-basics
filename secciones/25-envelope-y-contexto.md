# 25 - El Envelope: cómo viaja el contexto (tenant, usuario) y por qué no se procesa en memoria

> 🎯 **Hacia dónde va:** resolvemos cómo viajan el tenant y el usuario cuando un mensaje cruza la cola usando el Envelope, en vez de mezclar el contexto dentro del payload —pieza clave del multi-tenancy.

> 🌳 **Sección donde se domina** la semilla de multi-tenancy (§27) y conecta con el bug del `TenantId = DEFAULT` que vimos en §10. Resuelve: *cuando despacho un comando/evento por la cola, ¿cómo viajan el tenant y el usuario?*

## El escenario
En una petición HTTP tienes un **contexto** completo: el request trae el tenant, el usuario, sus permisos, todo. Pero cuando publicas un mensaje a una cola (Outbox → Service Bus), ese mensaje viaja **solo** hasta otro proceso, quizá minutos después. ¿Cómo sabe el handler del otro lado *de qué tenant* es el mensaje?

## 🟢 Lo ingenuo: meter el contexto dentro del mensaje

```csharp
// Metemos tenant y usuario DENTRO del payload de cada comando
public record InscribirMatrimonio(
    string TenantId,        // ⚠️ contexto mezclado con datos
    string UsuarioId,       // ⚠️
    Guid   CiudadanoId,
    string Conyuge);
```

Funciona… hasta que lo repites en los **50 comandos** del sistema, y el `TenantId` empieza a "ensuciar" cada firma, cada validación, cada test. Y si un comando olvida incluirlo, te quedas sin tenant.

## 💥 El dolor
- **Repetición:** tenant/usuario en el payload de *todos* los mensajes.
- **Fuga de responsabilidad:** datos de *infraestructura/seguridad* mezclados con datos de *negocio*. Tu `InscribirMatrimonio` no debería hablar de TenantId; eso es contexto, no negocio.
- **Frágil:** olvidar el campo en un mensaje = sin tenant = datos cruzados entre clientes (¡catástrofe multi-tenant!).

## 🔧 El refactor: separar el contenido del sobre

Un mensaje tiene **dos partes**, igual que una carta:

```
┌─────────────────────────────────────────┐
│  ENVELOPE (el sobre) — metadatos          │  ← TenantId, UsuarioId, MessageId,
│  ┌─────────────────────────────────────┐ │     CorrelationId, timestamp, remitente
│  │  PAYLOAD (el contenido) — el mensaje│ │  ← InscribirMatrimonio(CiudadanoId, Conyuge)
│  └─────────────────────────────────────┘ │     SOLO datos de negocio
└─────────────────────────────────────────┘
```

El **payload** lleva solo datos de negocio. El **envelope** lleva el contexto (quién, qué tenant, correlación). Tu comando queda limpio:

```csharp
// El comando vuelve a ser PURO negocio
public record InscribirMatrimonio(Guid CiudadanoId, string Conyuge);

// Al publicar, "firmas el envelope" con el contexto (no lo metes en el payload)
await bus.PublishAsync(new InscribirMatrimonio(ciudadanoId, conyuge)); // Wolverine adjunta tenant/usuario al envelope
```

Todos los transportes (Azure Service Bus, RabbitMQ, SQS…) soportan esto con **headers/propiedades** del mensaje — ahí viaja el envelope. Del otro lado, el handler **lee el contexto del envelope**, no del payload.

## 🏷️ El nombre
Esto es el **patrón Envelope** (sobre): separar el **contenido del mensaje** de sus **metadatos de transporte/contexto**. En Wolverine, el contexto vive en el **`MessageContext`/Envelope**, y para multi-tenancy publicas con `InvokeForTenantAsync(tenantId, comando)` — el tenant viaja en el sobre, firmado.

> [!TIP]
> Regla de oro: el envelope **crece** a medida que necesitas más contexto (correlación, causalidad, usuario), pero **con mesura** — no conviertas el sobre en un basurero. Lo de negocio va en el payload; lo de "quién/cuándo/de parte de quién" va en el sobre.

## 💣 La consecuencia crítica: por qué NO se procesa "en memoria"

Aquí se cierra el círculo con la semilla de §10 (el `TenantId = DEFAULT`). Wolverine resuelve el contexto del mensaje de **dos formas**:
1. Desde el **contexto HTTP** (si llegó por un request web).
2. Desde el **envelope de la cola** (si llegó por un transporte).

```csharp
// Pseudocódigo del resolver de contexto de Wolverine
contexto = esHttp        ? DesdeRequestHttp()       // tiene tenant/usuario
         : esCola         ? DesdeEnvelopeDelMensaje() // tiene tenant/usuario (en el sobre)
         : /* en memoria */ ContextoVacío();          // 💥 NO hay envelope → tenant = DEFAULT
```

Si publicas un evento **en memoria** (procesamiento *in-process*, sin pasar por la cola), **no hay envelope** → no hay tenant ni usuario → el handler revienta o, peor, procesa con `TenantId = DEFAULT` y **cruza datos de tenants**.

> [!IMPORTANT]
> 🪐 **Postura de Cosmos (versión técnica de una regla filosófica).** Un evento que debe procesarse en otro handler **debe viajar por la cola** (Outbox → transporte), no "en memoria", **precisamente para que lleve su envelope con el contexto**. Un evento privado configurado sin canal de salida que termina procesándose en memoria es un bug latente: cuando el handler intente leer el tenant, no estará. Por eso "no se debe hacer eso".

---

### El Descubrimiento
El contexto (tenant, usuario) **no es un dato de negocio**: viaja en el **envelope**, no en el payload. Y como el contexto se reconstruye desde el HTTP request o desde el envelope de la cola, **procesar en memoria deja al handler sin contexto** → tenant DEFAULT → desastre multi-tenant. La solución es filosóficamente limpia (separar contenido de contexto) y técnicamente obligatoria (el envelope es la única fuente del contexto fuera del HTTP).

**Siguiente:** con ACL (§24) + Envelope (§25) ya sabes recibir y enviar mensajes entre Bounded Contexts de forma segura. El cierre natural: **sagas** para procesos que cruzan varios.

---

[⬅️ Volver a Anti-Corruption Layer](./24-anti-corruption-layer.md) · [🗺️ Roadmap](../ROADMAP.md)
