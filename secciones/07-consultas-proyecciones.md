# 07 - Consultas y vistas inteligentes

Ya guardamos hechos de forma profesional. Ahora el reto es: ¿cómo volvemos a leerlos sin tener que hacer todo el trabajo manual de la sección 03?

## 1. Reconstrucción automática del estado
¿Recuerdas el bucle `foreach` que escribiste para calcular el total? Marten ya sabe hacer eso por ti. Solo tienes que pedirle el "Agregado" (el objeto final).

```csharp
// Marten lee la historia y te devuelve la "foto" actual de la orden
var orden = await session.Events.AggregateStreamAsync<OrdenCompra>(idOrden);

Console.WriteLine($"Orden recuperada: {orden.Numero}");
```

---

## 2. El problema del rendimiento
Si nuestro negocio tiene 10 años de historia y millones de hechos, reconstruir el estado cada vez que alguien quiere ver un reporte sería muy lento. 

**¿La solución?** Ir creando "vistas" o "reportes" en tiempo real a medida que los hechos ocurren.

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

    // Cuando el hecho "ProductoAgregado" sucede, actualizamos el resumen
    public void Apply(ProductoAgregado ev, OrdenResumen resumen)
    {
        resumen.Total += ev.Precio;
    }
}
```

---

### El Descubrimiento Final
A estas "vistas inteligentes" que escuchan hechos y transforman la historia en tablas de consulta fáciles de leer, se les llama **Proyecciones**. 

Con esto, has cerrado el círculo:
1. Identificaste **Hechos** (Eventos).
2. Construiste la **Historia** (Event Sourcing).
3. Aseguraste la permanencia en un **Baúl** (Event Store).
4. Optimizaste la lectura con **Vistas** (Proyecciones).

---

## 🏁 ¡Felicidades!
Has descubierto los fundamentos de una arquitectura orientada a eventos. Ahora tienes las herramientas para construir sistemas que no olvidan nada.

---

[⬅️ Volver a la sección anterior](./06-marten-event-store.md)

[🏠 Volver al inicio](../README.md)
