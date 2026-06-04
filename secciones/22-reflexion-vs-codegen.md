# 22 - Reflexión vs Generación de Código: la magia, desarmada

> 🌳 **Sección donde se domina** la semilla repetida en §04, §10, §12 y §13: *"el framework no es magia, genera código que puedes leer"*. Esta es la pieza que separa al que usa Wolverine/Marten "porque funciona" del que entiende **qué hace y por qué es rápido**.

## Dos formas de que un framework "llame a tu código"

Cuando envías un comando y "mágicamente" se ejecuta tu handler con sus dependencias inyectadas, el framework tuvo que resolver dos preguntas en algún momento: *¿qué handler?* y *¿cómo construyo sus dependencias?*. Hay dos maneras de responderlas.

### Opción A — Reflexión (en runtime, en cada llamada)
La **reflexión** es la capacidad de .NET de inspeccionar tipos y llamar miembros **en tiempo de ejecución**: `Type`, `MethodInfo.Invoke`, `Activator.CreateInstance`. Un mediator clásico (estilo MediatR) hace, simplificado:

```csharp
// Pseudocódigo de un mediator por reflexión
var handlerType = typeof(IHandler<>).MakeGenericType(command.GetType()); // averigua el tipo
var handler = serviceProvider.GetService(handlerType);                   // lo resuelve del contenedor
var method = handlerType.GetMethod("Handle");                            // busca el método
method.Invoke(handler, new[] { command });                              // lo invoca por reflexión
```

- ✅ Flexible, simple de implementar.
- ❌ **Costo en cada llamada** (la reflexión es lenta comparada con una llamada directa), y sobre todo: **opaco**. Si algo falla, no hay código que leer — todo ocurre "por dentro" del framework.

### Opción B — Generación de código (lo que hace Wolverine)
En lugar de resolver por reflexión cada vez, el framework **escribe código C# real** que llama tu handler **directamente**, construye sus dependencias **explícitamente** y encadena el middleware. Ese código se compila una vez y se ejecuta como cualquier otro.

```csharp
// Esbozo del tipo de código que Wolverine GENERA por ti (simplificado)
public class AprobarOrden_Handler
{
    public async Task Handle(AprobarOrden command, IDocumentSession session, CancellationToken ct)
    {
        var orden = await session.Events.FetchForWriting<Orden>(command.OrdenId, ct); // carga
        var eventos = AprobarOrdenHandler.Handle(command, orden.Aggregate);            // TU función
        foreach (var e in eventos) orden.AppendOne(e);                                 // append
        await session.SaveChangesAsync(ct);                                            // guarda
    }
}
```

- ✅ Rápido (llamadas directas, sin reflexión por mensaje).
- ✅ **Inspeccionable**: ese archivo existe y lo puedes leer. La doc oficial de Wolverine lo confirma: *"depends much more on runtime generated code than the IoC container tricks that many other .NET frameworks do"*.

---

## La consecuencia anti-magia (lo más importante de toda la sección)

> [!IMPORTANT]
> Si alguna vez no entiendes **qué hace Wolverine con tu mensaje**, no adivines: **lee el código que generó**.
> ```bash
> dotnet run -- codegen write
> ```
> Genera los archivos en `./Internal/Generated/WolverineHandlers/`. Ahí ves, en C# normal, exactamente: qué handler se llama, qué dependencias se resuelven y en qué scope, qué middleware se aplica y en qué orden. **La magia se convierte en un archivo que puedes leer.** Eso es justo lo que esta sección (y todo el workshop) persigue.

Este mismo comando es la herramienta de *troubleshooting* que recomienda la doc oficial: si sospechas un problema de scope (p. ej. un `IServiceScope` que rompe el `TenantId`, la semilla de §10), lo confirmas mirando el código generado.

---

## Los modos de generación (y por qué importan en serverless)

Wolverine puede generar el código de dos formas:
- **Dinámico (por defecto):** genera y compila la primera vez que se usa cada handler → arranque más lento (*cold start*), pago una sola vez.
- **Pre-generado / estático:** generas el código **en build** y lo compilas con la app → arranque instantáneo, ideal para **Azure Functions / serverless**.

> [!NOTE]
> 🪐 **Ancla Cosmos.** Cosmos corre en **Azure Functions (.NET isolated)**, donde el *cold start* duele. La best-practice oficial recomienda **pre-generar los tipos** justo para esto. Y recuerda (semilla de §13): Cosmos usa `AddWolverine(ExtensionDiscovery.ManualOnly, …)` para que la generación/descubrimiento no intente cargar DLLs inexistentes en el host serverless.

---

## ¿Y Marten? También genera código

No es solo Wolverine. Marten **descubre** tus métodos `Apply` y **compila** el aplicador (semilla de §12): por eso reconstruir un agregado no paga el costo de `dynamic` en cada evento. Es la misma filosofía del "Critter Stack": preferir **código generado e inspeccionable** sobre reflexión por llamada.

---

## El primo en compile-time: Source Generators

C# moderno tiene **source generators**: generación de código en el **momento de compilar** (no en runtime). Los usan `System.Text.Json`, validadores, etc. Misma idea —generar código en vez de reflexionar— pero más temprano (mejor para AOT). Conocerlos te da el marco mental completo: **reflexión = decidir en runtime; codegen = decidir antes y dejar código explícito**.

---

### El Descubrimiento
La sensación de "magia" de Wolverine/Marten viene de no ver el puente entre tu mensaje y tu handler. Pero ese puente **es código C# generado**, rápido y legible — no reflexión opaca. Con `codegen write` lo lees cuando quieras. Has desarmado la última caja negra del Critter Stack.

> Con esto cierras el bloque avanzado: Decider + Aggregate Handler (§18), Versionado (§19), CQRS/Proyecciones (§20), Testing (§21) y Codegen (§22). El [ROADMAP](../ROADMAP.md) sigue abierto para más temas (sagas, dead-letter, observabilidad…) cuando los necesites.

---

[⬅️ Volver a Testing sin mocks](./21-testing-sin-mocks.md) · [🗺️ Roadmap](../ROADMAP.md) · [🏛️ La Plantilla Cosmos](./17-plantilla-cosmos.md)
