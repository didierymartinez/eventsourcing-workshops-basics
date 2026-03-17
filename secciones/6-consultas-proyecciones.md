# 06 - Consultas y Proyecciones

En la etapa final, aprenderemos a pedirle a Marten que haga el trabajo sucio por nosotros: reconstruir el estado y generar vistas optimizadas.

## 1. Reconstrucción Automática (AggregateStream)

¿Recuerdas el bucle `foreach` que hicimos en la sección 04? Marten lo hace automáticamente con una sola línea.

```csharp
// Reconstruye el estado aplicando todos los eventos guardados
var orden = await session.Events.AggregateStreamAsync<OrdenCompra>(idOrden);

Console.WriteLine($"Orden recuperada: {orden.Numero}");
Console.WriteLine($"Cantidad de productos: {orden.Productos.Count}");
```

## 2. Proyecciones (ReadOnly Models)

A veces no quieres reconstruir todo el agregado solo para saber una cosa (ej: el total de ventas). Para eso usamos **Proyecciones**. Una proyección escucha eventos y actualiza una tabla de consulta.

### Ejemplo: Totalizador de Orden

```csharp
public class OrdenResumen
{
    public Guid Id { get; set; }
    public decimal Total { get; set; }
}

public class OrdenResumenProjection : Marten.Events.Projections.EventProjection
{
    // Cuando ocurra una OrdenCreada, creamos el resumen
    public OrdenResumen Create(OrdenCreada ev) => new OrdenResumen { Id = ev.Id, Total = 0 };

    // Cuando ocurra un ProductoAgregado, sumamos al total
    public void Apply(ProductoAgregado ev, OrdenResumen resumen)
    {
        resumen.Total += ev.Precio;
    }
}
```

### Registro en Marten
Debes registrar la proyección al configurar el store:

```csharp
var store = DocumentStore.For(opt =>
{
    opt.Connection(connectionString);
    // Registrar la proyección para que se ejecute en tiempo real (Inline)
    opt.Projections.Add<OrdenResumenProjection>(ProjectionLifecycle.Inline);
});
```

---

## 🏁 ¡Fin del Workshop!

Has aprendido el ciclo completo de Event Sourcing:
1. Entender los **fundamentos** con listas nativas.
2. **Modelar** eventos inmutables.
3. **Construir** la lógica del agregado.
4. **Persistir** en PostgreSQL con Marten.
5. **Consultar** y proyectar datos.

---

[⬅️ Volver a la sección anterior](./5-persistencia-marten.md)

[🏠 Volver al inicio](../README.md)
