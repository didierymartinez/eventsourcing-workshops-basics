# 06 - Automatizando el historial

Tenemos los hechos y tenemos el baúl (PostgreSQL). Pero todavía nos falta algo: no queremos escribir código manual para traducir nuestros records a tablas de base de datos.

## 🎯 El Objetivo
Queremos una herramienta que tome nuestros hechos tal cual están en C# y los guarde en el baúl sin que tengamos que preocuparnos por abrir conexiones, escribir SQL o serializar datos. 

Buscamos "conectar y listo" para que podamos enfocarnos en el negocio.

---

## 1. Buscando eficiencia

En lugar de construir nuestro propio motor de persistencia, vamos a usar una herramienta diseñada específicamente para esto.

### Instalación
En la terminal, dentro de la carpeta de tu proyecto, ejecuta:
```bash
dotnet add package Marten
```

---

## 2. Conectando el proyecto al baúl
En tu `Program.cs`, vamos a configurar cómo nuestra app debe hablar con el almacén.

```csharp
using Marten;

// Configuramos la conexión automática
var store = DocumentStore.For(opt =>
{
    opt.Connection("Host=localhost;Port=5432;Database=taller-hechos;Username=postgres;Password=Marten123");
});
```

---

## 3. Guardando hechos de forma profesional

```csharp
var idOrden = Guid.NewGuid();

await using var session = store.LightweightSession();

// Enviamos los hechos al baúl
session.Events.StartStream(idOrden, 
    new OrdenCreada(idOrden, "FAC-999"),
    new ProductoAgregado("Monitor Curvo 34", 1, 450m)
);

await session.SaveChangesAsync();

Console.WriteLine("¡Hechos guardados de forma segura y permanente!");
```

---

### El Descubrimiento
Acabas de automatizar todo lo que antes era manual. La herramienta que está haciendo este trabajo inteligente de convertir tus records en historia persistente se llama **Marten**. 

Marten es quien permite que tu PostgreSQL actúe como un **Event Store** de alto nivel.

---

[⬅️ Volver a la sección anterior](./05-preparando-persistencia.md)

[➡️ Siguiente sección: Consultas y vistas inteligentes](./07-consultas-proyecciones.md)
