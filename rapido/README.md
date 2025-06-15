# Fast Workshop Marten + .NET

Guía rápida para Event Sourcing con Marten y PostgreSQL en .NET.

---

## 1. Requisitos

- [.NET 9.0 SDK](https://dotnet.microsoft.com/download)
- [PostgreSQL 12+](https://www.postgresql.org/download/)  
  (o Docker: `docker run --name postgres-marten -e POSTGRES_PASSWORD=Marten123 -p 5432:5432 -d postgres`)
- Crea la base de datos `marten-demo`.

---

## 2. Proyecto y dependencias

```bash
dotnet new console -n MartenDemo
cd MartenDemo
dotnet add package Marten
```

---

## 3. Define eventos

```csharp
public record OrdenCreada(Guid IdOrden, string Numero);
public record ProductoAgregado(Guid IdOrden, string IdProducto, string Nombre, decimal Precio);
```

---

## 4. Configura Marten

```csharp
using Marten;

var store = DocumentStore.For(opt =>
    opt.Connection("User ID=postgres;Password=Marten123;Host=localhost;Port=5432;Database=marten-demo")
);
```

---

## 5. Persistir eventos

```csharp
await using var session = store.IdentitySession();

var idOrden = Guid.NewGuid();
var ordenCreada = new OrdenCreada(idOrden, "ORD-001");
var productoAgregado = new ProductoAgregado(idOrden, "1", "Laptop", 1500m);

session.Events.StartStream<OrdenCompra>(idOrden, ordenCreada);
session.Events.Append(idOrden, productoAgregado);
await session.SaveChangesAsync();
```

---

## 6. Define el agregado

```csharp
public record OrdenCompra(Guid Id, string Numero, List<Producto> Productos)
{
    public static OrdenCompra Create(OrdenCreada e) =>
        new(e.IdOrden, e.Numero, []);
    public static OrdenCompra Apply(ProductoAgregado e, OrdenCompra prev) =>
        prev with { Productos = prev.Productos.Append(new Producto(e.IdProducto, e.Nombre, e.Precio)).ToList() };
}
public record Producto(string IdProducto, string Nombre, decimal Precio);
```

---

## 7. Reconstruir el agregado

```csharp
await using var session = store.LightweightSession();
var orden = await session.Events.AggregateStreamAsync<OrdenCompra>(idOrden);
// También puedes reconstruir hasta una versión específica:
// var ordenV2 = await session.Events.AggregateStreamAsync<OrdenCompra>(idOrden, 2);
Console.WriteLine(orden);
```

---

## 8. Consultar eventos (opcional)

```csharp
var eventos = await session.Events.FetchStreamAsync(idOrden);
foreach (var ev in eventos)
    Console.WriteLine($"{ev.EventType.Name}: {ev.Data}");
```

---

## 9. Proyecciones (Read Models)

```csharp
public class OrdenCompraTotal
{
    public Guid Id { get; set; }
    public decimal ValorTotal { get; set; }
}

public class OrdenesTotalProjection : Marten.Events.Projections.EventProjection
{
    public OrdenesTotalProjection()
    {
        Project<OrdenCreada>((e, op) => op.Store(new OrdenCompraTotal { Id = e.IdOrden }));
        ProjectAsync<ProductoAgregado>(async (e, op, token) =>
        {
            var orden = await op.LoadAsync<OrdenCompraTotal>(e.IdOrden);
            orden.ValorTotal += e.Precio;
            op.Store(orden);
        });
    }
}
```

Agrega la proyección en la configuración:

```csharp
options.Projections.Add<OrdenesTotalProjection>(Marten.Events.Projections.ProjectionLifecycle.Inline);
```

Usa `IdentitySession` para que Marten rastree las entidades y las proyecciones funcionen correctamente.

---

## 10. Tablas en PostgreSQL

- `mt_events`: eventos
- `mt_streams`: streams (guarda el tipo si usas streams tipados)
  - Si usas `StartStream<T>`, Marten asocia el tipo en la columna `type`.

---

¡Listo! Sigue estos pasos para tener Event Sourcing y proyecciones básicas con Marten.
