# Consultar eventos y reconstruir el agregado

En esta sección aprenderás cómo consultar los eventos almacenados en la base de datos usando Marten y cómo reconstruir el estado actual de tu agregado (por ejemplo, una orden de compra).

## 1. Reconstruir el agregado automáticamente con AggregateStreamAsync

Marten facilita la reconstrucción del agregado a partir de los eventos de un stream usando el método `AggregateStreamAsync<T>`. Este método aplica automáticamente todos los eventos del stream en orden y devuelve una instancia del agregado reconstruido.

### Ejemplo de uso

```csharp
using Marten;
using Dometrain.Marten.Domain; // Asegúrate de tener el namespace correcto

// Suponiendo que ya tienes el store y el id del agregado
await using var session = store.LightweightSession();

// Reconstruye el agregado OrdenCompra a partir de los eventos del stream
var orden = await session.Events.AggregateStreamAsync<OrdenCompra>(idOrden);

if (orden != null)
{
    Console.WriteLine($"Orden reconstruida: {orden}");
}
else
{
    Console.WriteLine("No se encontró el stream o no hay eventos para este id.");
}
```

- `AggregateStreamAsync<OrdenCompra>(idOrden)` aplica todos los eventos del stream identificado por `idOrden` y devuelve el agregado reconstruido.
- El tipo genérico `<OrdenCompra>` debe coincidir con tu record de agregado y debe tener una propiedad pública llamada exactamente `Id`.

> **Nota:** Marten requiere que el record del agregado tenga una propiedad pública llamada `Id` para poder asociar correctamente el stream de eventos.

## 2. ¿Qué sucede internamente?

- Marten obtiene todos los eventos del stream y aplica los métodos estáticos `Create` y `Apply` definidos en tu record de agregado.
- El estado final del agregado refleja todos los cambios realizados a través de los eventos.

---

[⬅️ Volver a la sección anterior](./construir-agregado.md) | 
[⬆️ Volver al índice](../README.md)