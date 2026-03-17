# 06 - Marten: Tu Event Store profesional

Ya tenemos eventos y ya tenemos una base de datos. Ahora necesitamos una herramienta que sepa cómo guardar esos eventos de forma eficiente en PostgreSQL. 

Esa herramienta es **Marten**.

## 1. El reemplazo de la Lista
¿Recuerdas nuestra `List<object>` de la sección 03? Marten va a tomar ese lugar, pero guardando los datos en discos reales.

### Instalación
En la carpeta de tu proyecto, instala el paquete:
```bash
dotnet add package Marten
```

---

## 2. Configuración inicial
En tu `Program.cs`, configura el `DocumentStore` (es el objeto que sabe hablar con Postgres).

```csharp
using Marten;

var store = DocumentStore.For(opt =>
{
    opt.Connection("Host=localhost;Port=5432;Database=taller-marten;Username=postgres;Password=Marten123");
});
```

---

## 3. Guardando tu primer Stream
En Event Sourcing, los eventos se agrupan en **Streams** (líneas de tiempo). Cada Orden de Compra tendrá su propio Stream.

```csharp
var idOrden = Guid.NewGuid();

await using var session = store.LightweightSession();

// Abrimos el stream y guardamos los eventos iniciales
session.Events.StartStream<OrdenCompra>(idOrden, 
    new OrdenCreada(idOrden, "FAC-001"),
    new ProductoAgregado("Teclado Mecánico", 1, 85m)
);

await session.SaveChangesAsync();

Console.WriteLine("¡Felicidades! Tus eventos ahora son permanentes.");
```

---

## ¿Qué acaba de pasar?
Si reinicias tu app ahora, **tus eventos seguirán ahí**. Marten se encargó de:
1. Serializar tus records a JSON.
2. Crear las tablas en Postgres automáticamente.
3. Asegurar que los datos se guarden físicamente.

---

[⬅️ Volver a la sección anterior](./05-preparando-persistencia.md)

[➡️ Siguiente sección: El poder de las Proyecciones](./07-consultas-proyecciones.md)
