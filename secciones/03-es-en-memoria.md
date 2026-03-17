# 03 - Event Sourcing en Memoria

Ahora vamos a implementar nuestra primera versión de Event Sourcing. Lo haremos de la forma más sencilla posible: usando una **Lista** de C#.

## 1. Definiendo los Eventos
En C#, la mejor forma de representar eventos es mediante **Records**. Son inmutables (no cambian) y muy breves.

Escribe esto en tu `Program.cs`:

```csharp
using System;
using System.Collections.Generic;

// Definimos los hechos que pueden ocurrir
public record OrdenCreada(Guid Id, string NumeroFactura);
public record ProductoAgregado(string Nombre, int Cantidad, decimal Precio);
```

---

## 2. Guardando la historia
En lugar de una tabla de base de datos, usaremos una lista para guardar todo lo que pase.

```csharp
// Nuestro "Event Store" temporal
var historial = new List<object>();

// Ocurren cosas en el negocio
var idOrden = Guid.NewGuid();
historial.Add(new OrdenCreada(idOrden, "FAC-2024-01"));
historial.Add(new ProductoAgregado("Laptop", 1, 1200m));
historial.Add(new ProductoAgregado("Mouse", 2, 25m));

Console.WriteLine($"Hemos guardado {historial.Count} eventos en memoria.");
```

---

## 3. Reconstruyendo el presente
Como no tenemos una tabla con el "Total", debemos calcularlo recorriendo el historial.

```csharp
decimal total = 0;

foreach (var evento in historial)
{
    if (evento is ProductoAgregado p)
    {
        total += (p.Precio * p.Cantidad);
    }
}

Console.WriteLine($"El total de la orden es: ${total}");
```

---

## ¿Qué hemos aprendido?
1. Los eventos son **Records** (inmutables).
2. El estado no se guarda, se **calcula**.
3. La lista es nuestro "Event Store".

---

[⬅️ Volver a la sección anterior](./02-primer-proyecto.md)

[➡️ Siguiente sección: El gran problema](./04-el-problema.md)
