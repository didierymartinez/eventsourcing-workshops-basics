## Ejercicio: Modelando eventos en Program.cs

En este ejercicio trabajarás en el archivo `Program.cs` para modelar un flujo básico de eventos usando C# y Marten.

### ¿Qué debes hacer?

1. **Define los eventos** que representan acciones importantes en el dominio de una orden de compra. Por ejemplo:
   - `OrdenCreada`: cuando se crea una orden.
   - `ProductoAgregado`: cuando se agrega un producto a la orden.

2. **Implementa estos tipos usando `record`** para los eventos. Los `record` en C# son ideales para modelar datos inmutables y facilitan la comparación y manipulación de datos.

### Diferencia entre `class` y `record`

- **`class`**: Es el tipo de referencia tradicional en C#. Sus instancias son mutables por defecto y la igualdad se basa en la referencia (a menos que se sobreescriba).
- **`record`**: Introducido en C# 9, es ideal para modelos inmutables. La igualdad se basa en el valor de sus propiedades, lo que es útil para eventos y estados en DDD/Event Sourcing.

### Ejemplo orientativo

```csharp
public record OrdenCreada(string IdOrden, string Numero);
public record ProductoAgregado(string IdProducto, string Nombre, decimal Precio);
```

### ¿Por qué modelar así?

- Los **eventos** (`OrdenCreada`, `ProductoAgregado`) representan hechos que ocurrieron en el sistema y deben ser inmutables.

> Intenta implementar estos tipos en tu `Program.cs` y reflexiona sobre cómo los eventos pueden modificar el estado del sistema.

---

[⬅️ Volver a la sección anterior](./configuracion-inicial.md)

[➡️ Siguiente sección: Persistencia de eventos con Marten](./persistencia-eventos-marten.md)
