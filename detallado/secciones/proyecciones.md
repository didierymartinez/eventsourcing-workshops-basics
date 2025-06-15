# Proyecciones y consultas derivadas

En Event Sourcing, una **proyección** es una vista materializada o consulta derivada que se construye a partir de los eventos almacenados. Las proyecciones permiten transformar la secuencia de eventos en modelos de lectura optimizados para consultas específicas, como totales, listados o reportes.

## ¿Por qué usar proyecciones?

- Permiten obtener información agregada o resumida a partir de los eventos.
- Facilitan la consulta eficiente sin tener que reconstruir todo el agregado cada vez.
- Son ideales para construir modelos de lectura (Read Models) o vistas especializadas.

## Ejemplo: Proyección de total de orden de compra

Supón que quieres mantener actualizado el total de cada orden de compra a medida que se agregan productos. Puedes definir una proyección que escuche los eventos y actualice un documento de total.

### Ejercicio guiado

1. **Crea una clase para el modelo de proyección:**

```csharp
public class OrdenCompraTotal
{
    public Guid Id { get; set; }
    public decimal ValorTotal { get; set; }
}
```

2. **Crea una proyección que procese los eventos:**

```csharp
using Marten.Events.Projections;

public class OrdenesTotalProjection : EventProjection
{
    public OrdenesTotalProjection()
    {
        Project<OrdenCreada>((e, operation) =>
        {
            operation.Store(new OrdenCompraTotal { Id = e.IdOrden });
        });

        ProjectAsync<ProductoAgregado>(async (e, operation, token) =>
        {
            var ordenCompra = await operation.LoadAsync<OrdenCompraTotal>(e.IdProducto);
            ordenCompra.ValorTotal += e.Precio;
            operation.Store(ordenCompra);
        });
    }
}
```

- `Project<TEvento>` define cómo reaccionar ante un evento y actualizar el modelo de proyección.
- `operation.Store` guarda o actualiza el documento de proyección.
- Puedes usar `ProjectAsync` para lógica asíncrona.

3. **Registra la proyección en la configuración de Marten:**

```csharp
options.Projections.Add<OrdenesTotalProjection>(ProjectionLifecycle.Inline);
```

#### ¿Por qué usar `Inline`?

- El modo `Inline` ejecuta la proyección inmediatamente cuando se guardan los eventos, asegurando que el modelo de lectura esté actualizado al finalizar la transacción.
- Es útil cuando necesitas que los datos proyectados estén disponibles de inmediato después de guardar los eventos.
- Para escenarios donde la consistencia inmediata es importante (por ejemplo, mostrar el total actualizado en la misma operación), `Inline` es la mejor opción.
- Otros modos como `Async` procesan las proyecciones en segundo plano y pueden tener un pequeño retraso.

#### ¿Por qué cambiar la sesión a `IdentitySession`?

- En el ejemplo, se usa `store.IdentitySession()` en vez de `LightweightSession()`.
- `IdentitySession` permite que Marten rastree las entidades cargadas y devuelva la misma instancia si se solicita varias veces durante la sesión.
- Esto es importante en proyecciones que actualizan documentos existentes (como sumar al total), ya que garantiza que los cambios se apliquen sobre la misma instancia y se almacenen correctamente.
- `LightweightSession` no realiza este seguimiento y puede causar inconsistencias si se actualizan documentos varias veces en la misma sesión.

> **Intenta implementar estas clases en tu proyecto y observa cómo Marten mantiene actualizada la proyección automáticamente a medida que se agregan eventos.**

## ¿Qué ocurre en la base de datos con las proyecciones?

Cuando defines una proyección en Marten, también se crea una tabla en PostgreSQL para almacenar el modelo de lectura (read model) resultante de la proyección. Por ejemplo, si tienes la clase `OrdenCompraTotal`, Marten creará una tabla llamada `orden_compra_totals` (o similar, según la convención de nombres) donde se almacenarán los totales de cada orden.

- Cada vez que se procesan eventos y se actualiza la proyección, Marten inserta o actualiza filas en esta tabla.
- Así puedes consultar directamente los datos proyectados usando SQL estándar o desde tu código C# como documentos.

> Las proyecciones permiten tener modelos de consulta optimizados y siempre actualizados en la base de datos, sin tener que reconstruir el agregado desde los eventos cada vez.

---

[⬅️ Volver a la sección anterior](./consultar-eventos-y-reconstruir-agregado.md)
[⬆️ Volver al índice](../README.md)
