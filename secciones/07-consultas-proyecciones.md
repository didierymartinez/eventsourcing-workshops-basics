# 07 - Consultas y vistas inteligentes

Llegamos a la última etapa. Ya guardamos la historia, pero ahora necesitamos leerla de forma eficiente.

## 🎯 El Objetivo
Si una orden tiene miles de hechos, reconstruir el total sumándolos todos cada vez que el cliente refresca la página es ineficiente y lento.

¿Cómo podemos tener una "foto" instantánea del total sin perder el historial de los hechos?

---

## 1. Reconstrucción automática
Antes de pasar a la solución avanzada, veamos cómo la herramienta automatiza lo que hicimos en la sección 03.

```csharp
// Le pedimos a la herramienta que reconstruya el estado final por nosotros
var orden = await session.Events.AggregateStreamAsync<OrdenCompra>(idOrden);

Console.WriteLine($"Orden recuperada: {orden.Numero}");
```

---

## 2. El poder de las vistas en tiempo real
Para cumplir nuestro objetivo de eficiencia, crearemos un modelo que se actualice **automáticamente** cada vez que un hecho ocurre.

```csharp
public class OrdenResumen
{
    public Guid Id { get; set; }
    public decimal Total { get; set; }
}

public class OrdenResumenProjection : Marten.Events.Projections.EventProjection
{
    // Cuando el hecho "OrdenCreada" sucede, iniciamos un resumen
    public OrdenResumen Create(OrdenCreada ev) => new OrdenResumen { Id = ev.Id, Total = 0 };

    // Cuando el hecho "ProductoAgregado" sucede, actualizamos el resumen instantáneamente
    public void Apply(ProductoAgregado ev, OrdenResumen resumen)
    {
        resumen.Total += ev.Precio;
    }
}
```

---

### El Descubrimiento Final
A estas tablas optimizadas para lectura, que se mantienen sincronizadas con la historia sin intervención manual, las llamamos **Proyecciones**. 

Ahora tienes un sistema que:
1. No olvida nada (Historia).
2. Es permanente (Almacenamiento).
3. Es increíblemente rápido al leer (Proyecciones).

---

## 🏁 ¡Felicidades!
Has completado el viaje. Has pasado de anotar en listas manuales a entender cómo funcionan los sistemas de eventos más avanzados de la industria.

---

[⬅️ Volver a la sección anterior](./06-marten-event-store.md)

[🏠 Volver al inicio](../README.md)
