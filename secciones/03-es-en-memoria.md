# 03 - Guardando hechos en el código

Vamos a empezar a anotar lo que sucede en nuestro negocio usando solo C#.

## 1. Definiendo los hechos
En C#, la mejor forma de representar algo que ya pasó es mediante un **Record**. Es inmutable (una vez creado, no cambia) y muy breve de escribir.

Escribe esto en tu `Program.cs`:

```csharp
// Definimos los hechos que pueden ocurrir en nuestra Orden de Compra
public record OrdenCreada(Guid Id, string NumeroFactura);
public record ProductoAgregado(string Nombre, int Cantidad, decimal Precio);
```

---

## 2. Nuestro diario de anotaciones
Para que la historia tenga sentido, necesitamos guardar estos hechos **en el orden exacto en que ocurrieron**. Un hecho que sucede después de otro puede cambiar el resultado final.

Por eso, usaremos la estructura más simple que nos permite mantener ese orden cronológico: una **Lista**.

```csharp
// Un lugar temporal para anotar todo lo que pase (en orden)
var historial = new List<object>();

// Comienzan a ocurrir hechos en nuestro negocio
var idOrden = Guid.NewGuid();

historial.Add(new OrdenCreada(idOrden, "FAC-2024-001"));
historial.Add(new ProductoAgregado("Laptop Pro", 1, 1500m));
historial.Add(new ProductoAgregado("Mouse Inalambrico", 1, 45m));

Console.WriteLine($"Se han registrado {historial.Count} hechos en el historial.");
```

---

## 3. Reconstruyendo la realidad
Como no tenemos una tabla que diga el "Total", vamos a leer nuestra lista de arriba hacia abajo para calcularlo.

```csharp
decimal total = 0;

foreach (var hecho in historial)
{
    if (hecho is ProductoAgregado p)
    {
        total += (p.Precio * p.Cantidad);
    }
}

Console.WriteLine($"El total de la orden calculada es: ${total}");
```

---

### El Descubrimiento
Acabas de implementar los dos pilares de este enfoque:
1. Usar hechos inmutables para guardar la verdad.
2. Reconstruir el estado leyendo el historial.

A esa "Lista" donde guardamos todos los hechos para consultarlos después, se le conoce conceptualmente como un **Event Store**.

---

[⬅️ Volver a la sección anterior](./02-primer-proyecto.md)

[➡️ Siguiente sección: El gran problema](./04-el-problema.md)
