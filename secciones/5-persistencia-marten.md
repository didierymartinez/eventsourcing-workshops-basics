# 05 - Persistencia de eventos con Marten

Ahora que entiendes cómo funciona el Event Sourcing de forma manual, vamos a reemplazar nuestra lista `List<object>` por una base de datos real diseñada para esto: **Marten**.

## 1. Instalar Marten

Primero, añade el paquete NuGet a tu proyecto `TallerMarten.OrdenCompra`:

```bash
dotnet add package Marten
```

## 2. Configurar el `DocumentStore`

El `DocumentStore` es el objeto principal de Marten que gestiona la conexión a PostgreSQL.

```csharp
using Marten;

// Configuramos la conexión a nuestro contenedor de Docker
var store = DocumentStore.For(opt =>
{
    opt.Connection("Host=localhost;Port=5432;Database=taller-marten;Username=postgres;Password=Marten123");
});
```

## 3. Guardar eventos (StartStream)

En lugar de hacer un `.Add()` a una lista, usaremos `StartStream` para abrir una "línea de tiempo" (o Stream) para nuestra orden.

```csharp
var idOrden = Guid.NewGuid();

await using var session = store.LightweightSession();

// Iniciamos un stream para una nueva orden
session.Events.StartStream<OrdenCompra>(idOrden, 
    new OrdenCreada(idOrden, "FAC-001"),
    new ProductoAgregado("Teclado Mecánico", 1, 80m)
);

await session.SaveChangesAsync();
Console.WriteLine("Eventos guardados en PostgreSQL!");
```

### ¿Qué acaba de pasar?
- Marten creó automáticamente las tablas necesarias en PostgreSQL.
- Guardó los eventos serializados como JSON.
- Asoció los eventos al ID único de la orden.

---

[⬅️ Volver a la sección anterior](./4-agregado-puro.md)

[➡️ Siguiente sección: Consultas y Proyecciones](./6-consultas-proyecciones.md)