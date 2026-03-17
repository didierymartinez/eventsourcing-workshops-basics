# 06 - Automatizando el historial

Ya tenemos los hechos y ya tenemos el baúl (el contenedor). Pero en la sección 03 vimos que guardar en una lista y hacer bucles `foreach` manualmente es mucho trabajo y puede fallar.

## 1. Buscando eficiencia
¿Y si existiera algo que tome nuestros hechos y los guarde directamente en el baúl sin que tengamos que preocuparnos por cómo se guardan?

Para eso, vamos a pedir ayuda a una librería externa.

### Instalación
En la terminal, dentro de la carpeta de tu proyecto, ejecuta:
```bash
dotnet add package Marten
```

---

## 2. Conectando el proyecto al baúl
En tu `Program.cs`, vamos a configurar cómo nuestra app debe hablar con el contenedor que creamos.

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
La librería que acabamos de instalar nos permite agrupar hechos relacionados. Por ejemplo, todos los hechos de la misma Orden de Compra.

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
Acabas de simplificar todo tu trabajo manual. La librería que está haciendo todo este "trabajo sucio" de serializar, conectar y guardar los hechos se llama **Marten**. 

Marten es la que convierte a PostgreSQL en un **Event Store** profesional para tus aplicaciones .NET.

---

[⬅️ Volver a la sección anterior](./05-preparando-persistencia.md)

[➡️ Siguiente sección: Consultas y vistas inteligentes](./07-consultas-proyecciones.md)
