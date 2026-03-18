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

### ¿Qué es un Stream realmente?
A esta secuencia ininterrumpida de hechos la llamamos **Stream** (Flujo). 

Piénsalo así: **Un Stream es la biografía de la entidad**. 
> Es el registro de todo lo que te ha pasado: naces, creces, estudias, te casas... Esa sucesión de hechos, en ese orden exacto, es lo que permite conocer tu pasado. Pero la biografía por sí sola es solo papel; falta el protagonista.

---

## 4. Reconstruyendo la realidad (El Agregado)
Para saber el estado actual (por ejemplo, tu saldo bancario), "reproducimos" el Stream de arriba hacia abajo.

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
Acabas de construir un **Agregado**, que es el concepto que lo une todo. Para que no haya confusión, definamos las piezas usando la analogía de una **Persona**:

1.  **El Identificador (ID / Pasaporte)**: 
    Es el número único que nos permite saber de quién estamos hablando. Sin ID, no podemos buscar la historia en el Event Store.

2.  **La Entidad (El Sujeto)**: 
    Es el concepto del objeto (ej. "Persona" o "Orden de Compra"). Es el "quién" que posee ese ID.

3.  **El Stream (La Biografía)**: 
    Es la lista de hechos que le han pasado a esa Entidad específica. Es su pasado escrito en el tiempo.

4.  **El Agregado (La Unidad de Decisión)**: 
    Es el **Agente Vivo**. Eres **tú** (Entidad) + **tu ID** + **tu historia** (Stream) + **tu contexto** (familia, dirección, reglas).
    > El **Agregado** es la frontera de consistencia. Es la entidad "cargada" con toda su historia y lista para responder a una petición. Es quien dice: *"Dado que en mi biografía dice que ya estoy casado, no puedo registrar el hecho 'Casarse' otra vez"*.

**En resumen**: La **Entidad** nos da la identidad, el **Stream** nos da el pasado, y el **Agregado** es el objeto que cobra vida para validar reglas y decidir el futuro.

Al lugar donde guardamos permanentemente estos Streams se le conoce como **Event Store**.

---

[⬅️ Volver a la sección anterior](./02-primer-proyecto.md)

[➡️ Siguiente sección: El gran problema](./04-el-problema.md)
