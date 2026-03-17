# 07 - Consultas y Proyecciones

Hemos llegado a la meta. Ahora que tus eventos son inmutables y permanentes, vamos a ver cómo consultarlos de forma profesional.

## 1. Reconstrucción Automática
Ya no necesitas hacer el bucle `foreach` manual. Marten lo hace por ti pidiéndole el agregado.

```csharp
// Marten lee todos los eventos del stream y te devuelve el objeto final
var orden = await session.Events.AggregateStreamAsync<OrdenCompra>(idOrden);

Console.WriteLine($"Recuperada la orden {orden.Numero}");
```

---

## 2. Proyecciones (ReadOnly Models)
A veces no quieres la orden completa, solo quieres un reporte rápido de "Total de Ventas del Día". 

En Event Sourcing, creamos **Read Models** (Modelos de Lectura). Son tablas optimizadas para leer, que Marten actualiza cada vez que llega un evento.

```csharp
public class OrdenResumen
{
    public Guid Id { get; set; }
    public decimal Total { get; set; }
}

public class OrdenResumenProjection : Marten.Events.Projections.EventProjection
{
    public OrdenResumen Create(OrdenCreada ev) => new OrdenResumen { Id = ev.Id, Total = 0 };

    public void Apply(ProductoAgregado ev, OrdenResumen resumen)
    {
        resumen.Total += ev.Precio;
    }
}
```

---

## 🏁 ¡Misión Cumplida!

Has recorrido un largo camino:
1. Empezaste con **teoría** sobre CRUD vs Historia.
2. Experimentaste con **listas en memoria** y sentiste el miedo de perder datos.
3. Instalaste **Docker y Postgres** para dar seguridad a tu negocio.
4. Usaste **Marten** para simplificar el proceso y hacerlo profesional.

### Reto Final
¿Podrías agregar un evento `OrdenCancelada` y hacer que tu proyección ponga el total en $0? 

---

[⬅️ Volver a la sección anterior](./06-marten-event-store.md)

[🏠 Volver al índice principal](../README.md)
