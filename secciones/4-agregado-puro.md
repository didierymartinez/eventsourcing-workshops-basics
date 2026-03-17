# 04 - Construir el Agregado (Puro C#)

En este paso, aprenderás a crear un **Agregado** y cómo "reconstruir" su estado a partir de eventos usando solo C# nativo. Esto te permitirá entender la lógica antes de que Marten lo haga por ti.

## 1. ¿Qué es un Agregado?

Un **Agregado** es la entidad principal que protege las reglas de negocio. En Event Sourcing, el estado del Agregado no se guarda en una tabla; se deriva de los eventos.

### La regla de oro:
> El estado actual = Eventos del pasado + Funciones de Aplicación.

---

## 2. Ejercicio: El record `OrdenCompra`

Modifica tu `Program.cs` para incluir el Agregado. Usaremos métodos estáticos `Create` y `Apply` para mantener la lógica pura e inmutable.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

// 1. Definimos el Agregado
public record OrdenCompra(Guid Id, string Numero, List<string> Productos)
{
    // Función para el primer evento (Creación)
    public static OrdenCompra Create(OrdenCreada ev)
    {
        return new OrdenCompra(ev.Id, ev.NumeroFactura, new List<string>());
    }

    // Función para eventos posteriores (Modificación)
    public static OrdenCompra Apply(ProductoAgregado ev, OrdenCompra estadoActual)
    {
        // Creamos una nueva versión del agregado (Inmutabilidad)
        var nuevaLista = new List<string>(estadoActual.Productos) { ev.Nombre };
        return estadoActual with { Productos = nuevaLista };
    }
}

// 2. Simulamos la Reconstrucción del Estado
var eventos = new List<object>
{
    new OrdenCreada(Guid.NewGuid(), "INV-2024-001"),
    new ProductoAgregado("Laptop", 1, 1200m),
    new ProductoAgregado("Mouse", 2, 25m)
};

// Reconstrucción Manual
OrdenCompra? miOrden = null;

foreach (var ev in eventos)
{
    miOrden = ev switch
    {
        OrdenCreada c => OrdenCompra.Create(c),
        ProductoAgregado p => OrdenCompra.Apply(p, miOrden!),
        _ => miOrden
    };
}

Console.WriteLine($"Orden: {miOrden.Numero} | Productos: {string.Join(", ", miOrden.Productos)}");
```

---

## 3. ¿Por qué este patrón?

1. **Inmutabilidad**: Usar el operador `with` asegura que nunca modificamos el objeto original.
2. **Predecibilidad**: Si tienes la misma lista de eventos, siempre obtendrás el mismo Agregado.
3. **Desacoplamiento**: El Agregado no sabe nada de bases de datos, solo sabe de lógica de negocio.

---

> [!IMPORTANT]
> Entender este bucle `foreach` de reconstrucción es la base de todo. Cuando usemos Marten, él hará este bucle por nosotros de forma automática y eficiente.

---

[⬅️ Volver a la sección anterior](./3-modelado-eventos.md)

[➡️ Siguiente sección: Persistencia con Marten](./5-persistencia-marten.md)
