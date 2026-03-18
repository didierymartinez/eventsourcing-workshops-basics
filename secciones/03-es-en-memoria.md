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
Acabas de construir un **Agregado**, que es el concepto que lo une todo. Para que no haya duda, mapeemos la teoría a lo que realmente está pasando en tu código C#:

1.  **El Identificador (El `idOrden`)**: 
    Es la pieza más importante. Es el ancla que nos dice de quién es la historia que estamos leyendo. Sin este ID, los hechos estarían "sueltos" y no tendrían dueño.

2.  **La Entidad (El Concepto de Identidad)**: 
    En Event Sourcing, la **Entidad** es más conceptual que física. No es necesariamente una clase que instancies, sino la **identidad** que vive detrás del ID. Cuando dices "esta es la Orden #123", la Entidad es ese concepto de "Orden" que tiene ese ID único.

3.  **El Stream (La Biografía / `List<object>`)**: 
    Es la secuencia de hechos. Es el historial de vida de esa identidad conceptual. 

4.  **El Agregado (El "Agente" en el código)**: 
    Es la unidad lógica que **sí creas en tu código**. Es el objeto que toma el ID (Entidad conceptual) y carga su historial (Stream) para poder tomar decisiones.
    > El **Agregado** es quien usa su lógica para decir: *"Si mi historia dice que esta orden ya fue pagada, no puedo agregarle más productos"*. 

**En resumen**, el rompecabezas de este capítulo tiene 4 piezas:
1.  **El ID**: El ancla que une todo.
2.  **La Entidad**: El "Quién" conceptual.
3.  **El Stream**: La biografía (qué pasó y en qué orden).
4.  **El Agregado**: El agente vivo en tu código que carga el pasado para decidir el futuro.

Esta es la base técnica. Pero como viste al ejecutar el programa, esta historia vive solo en la memoria RAM. En el siguiente capítulo, veremos por qué esto es un riesgo enorme.

---

[⬅️ Volver a la sección anterior](./02-primer-proyecto.md)

[➡️ Siguiente sección: El gran problema](./04-el-problema.md)
