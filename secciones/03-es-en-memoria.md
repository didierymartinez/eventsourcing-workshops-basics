# 03 - Guardando hechos en el código

Vamos a empezar a anotar lo que sucede en nuestro negocio usando solo C#.

## 🎯 El Objetivo
Imagina que el dueño del negocio te pide saber cuánto dinero ha vendido hoy. Sin embargo, **no tienes permiso de crear una tabla llamada "Ventas"**. Solo puedes anotar en un diario cada cosa que pase (compras, cancelaciones, cambios).

¿Cómo harías para darle el total al final del día usando solo esas notas?

---

## 1. Identificando al protagonista (La Entidad)
Cuando procesamos información, siempre lo hacemos sobre algo específico. En nuestro caso, es una **Orden de Compra**. 

Para que el sistema sepa de qué orden estamos hablando, necesitamos un identificador único. 

```csharp
// Identificador único para nuestra entidad
var idOrden = Guid.NewGuid();
```

En el mundo del software, a este objeto con identidad propia lo llamamos **Entidad**. Podemos tener la Entidad A y la Entidad B, cada una con su propia historia.

---

## 2. Definiendo los hechos (Records)
Independientemente del lenguaje de programación, lo que buscamos es que algo que **ya pasó** no se pueda alterar. En C#, la herramienta ideal para representar este concepto de inmutabilidad es el **Record**.

```csharp
// Definimos los hechos vinculados a nuestra entidad
public record OrdenCreada(Guid Id, string NumeroFactura);
public record ProductoAgregado(string Nombre, int Cantidad, decimal Precio);
```

> [!NOTE]
> Nota que cada hecho incluye el ID de la entidad. Un hecho siempre pertenece a una sola entidad a la vez.

---

## 3. El flujo de la historia (El Stream)
Para que la historia tenga sentido, guardamos estos hechos en el orden exacto en que ocurrieron. 

```csharp
// Nuestra secuencia cronológica de hechos
var historial = new List<object>();

historial.Add(new OrdenCreada(idOrden, "FAC-2024-001"));
historial.Add(new ProductoAgregado("Laptop Pro", 1, 1500m));
historial.Add(new ProductoAgregado("Mouse Inalambrico", 1, 45m));
```

A esta secuencia ininterrumpida de hechos que pertenecen a una misma entidad la llamamos **Stream** (Flujo). Una entidad equivale a un Stream.

---

## 4. Reconstruyendo la realidad (El Agregado)
Para saber el estado actual (en este caso, el total), simplemente "reproducimos" el Stream de arriba hacia abajo.

```csharp
decimal total = 0;

foreach (var hecho in historial)
{
    if (hecho is ProductoAgregado p)
    {
        total += (p.Precio * p.Cantidad);
    }
}

Console.WriteLine($"Total calculado: ${total}");
```

### El Descubrimiento
Acabas de construir un **Agregado**. 

Un **Agregado** no es solo el dato guardado; es la unión de:
1. La **Entidad** (el ID que nos identifica).
2. El **Stream** (la lista de hechos que son nuestra fuente de verdad).
3. La **Lógica** (el código que recorre los hechos para calcular el estado).

Al lugar donde guardamos permanentemente estos Streams para que nunca se pierdan, se le conoce conceptualmente como **Event Store**.

---

[⬅️ Volver a la sección anterior](./02-primer-proyecto.md)

[➡️ Siguiente sección: El gran problema](./04-el-problema.md)
