# Construir el agregado (Aggregate)

En este paso del workshop vas a crear el agregado principal de tu dominio y aprenderás cómo aplicar los eventos para reconstruir su estado.

## ¿Qué es un agregado?

Un **agregado** es la entidad central que representa el estado y las reglas de negocio de un concepto principal en tu dominio (por ejemplo, una orden de compra). En Event Sourcing, el estado del agregado se reconstruye aplicando la secuencia de eventos que le han ocurrido.

## Paso a paso para construir el agregado

1. **Define el record del agregado**
   - El agregado debe ser inmutable, es decir, cada vez que cambia su estado, se crea una nueva instancia en vez de modificar la existente. Esto ayuda a evitar errores y hace más fácil razonar sobre el flujo de eventos.
   - Por ejemplo:
     > Crea un record llamado `OrdenCompra` con las propiedades necesarias (IdOrden, Numero, Lista de productos).

2. **Agrega un método estático `Create`**
   - Este método recibirá el evento de creación (por ejemplo, `OrdenCreada`) y devolverá una nueva instancia del agregado con el estado inicial.
   - Así, el estado inicial del agregado siempre se deriva de un evento.

3. **Agrega un método estático `Apply`**
   - Este método recibirá un evento (por ejemplo, `ProductoAgregado`) y el estado anterior del agregado, y devolverá una nueva instancia con el estado actualizado.
   - De esta forma, puedes aplicar cada evento en orden y reconstruir el estado final del agregado.

4. **¿Por qué usar métodos estáticos?**
   - Los métodos estáticos permiten construir y evolucionar el agregado sin depender de un estado mutable interno.
   - Esto refuerza la inmutabilidad y hace que el proceso de reconstrucción a partir de eventos sea claro y predecible.

5. **¿Por qué inmutabilidad?**
   - La inmutabilidad significa que una vez creado un objeto, su estado no cambia. Cada vez que ocurre un evento, se crea una nueva versión del agregado.
   - Esto facilita el debugging, la concurrencia y la trazabilidad de los cambios en el sistema.

> **Nota importante sobre la propiedad `Id`**
>
> Marten requiere que el agregado tenga una propiedad pública llamada exactamente `Id` (no `IdOrden`, `OrderId`, etc.) para poder asociar correctamente el stream de eventos con el agregado. Si usas otro nombre, Marten no podrá reconstruir el agregado automáticamente desde los eventos.

## Ejemplo orientativo

A continuación se muestra un ejemplo completo del record `OrdenCompra` y sus métodos estáticos, siguiendo la convención de Marten:

```csharp
public record OrdenCompra(Guid Id, string Numero, List<Producto> Productos)
{
    public static OrdenCompra Create(OrdenCreada ordencreada)
    {
        return new OrdenCompra(ordencreada.IdOrden, ordencreada.Numero, []);
    }

    public static OrdenCompra Apply(ProductoAgregado productoAgregado, OrdenCompra ordenCompraAnterior)
    {
        var producto = new Producto(productoAgregado.IdProducto, productoAgregado.Nombre, productoAgregado.Precio);
        var listaProductos = ordenCompraAnterior.Productos.Append(producto).ToList();
        return ordenCompraAnterior with { Productos = listaProductos };
    }
}

public record Producto(string IdProducto, string Nombre, decimal Precio);
```

- El primer parámetro del record debe llamarse `Id` para que Marten pueda mapear el stream correctamente.
- El método `Create` inicializa el agregado a partir del evento de creación.
- El método `Apply` aplica un evento de producto agregado y devuelve una nueva instancia del agregado con el producto añadido.

---

[⬅️ Volver a la sección anterior](./persistencia-eventos-marten.md) | [➡️ Siguiente sección: Consultar eventos y reconstruir el agregado](./consultar-eventos-y-reconstruir-agregado.md)
[⬆️ Volver al índice](../README.md)