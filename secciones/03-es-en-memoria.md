# 03 - Guardando hechos en el código

Ahora que tenemos nuestro lienzo en blanco, vamos a empezar a anotar lo que sucede. 

## 🎯 El Objetivo
Imagina que el dueño del negocio te pide saber cuánto dinero ha vendido hoy. Sin embargo, **no tienes permiso de crear una tabla llamada "Ventas"**. Solo puedes anotar en un diario cada cosa que pase (compras, cancelaciones, cambios).

¿Cómo harías para darle el total al final del día usando solo esas notas?

---

## 1. Definiendo los hechos
Independientemente del lenguaje de programación, lo que buscamos es que algo que **ya pasó** no se pueda alterar. En C#, la herramienta ideal para representar este concepto de inmutabilidad es el **Record**, ya que es muy breve de escribir y protege los datos por naturaleza.

Escribe esto en tu `Program.cs`:

```csharp
// Definimos los hechos que pueden ocurrir en nuestra Orden de Compra
public record OrdenCreada(Guid Id, string NumeroFactura);
public record ProductoAgregado(string Nombre, int Cantidad, decimal Precio);
```

### ¿Por qué `record` en vez de `class`?

Usar `record` es fundamental por tres razones que refuerzan este principio:

1.  **Inmutabilidad**: Como mencionamos, un hecho es pasado. El pasado no se edita. Los `record` aseguran que nadie pueda cambiar accidentalmente los datos de un evento una vez que ha sido registrado.
2.  **Igualdad por Valor**: Dos registros son iguales si sus datos son iguales. Esto facilita enormemente las pruebas: puedes comparar si el sistema generó el hecho correcto simplemente comparando los objetos, sin revisar propiedad por propiedad.
3.  **Semántica**: Al leer `record`, el programador entiende de inmediato que este objeto es un transporte de datos inmutable, no una entidad con comportamiento complejo.

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

## 3. Reconstruyendo la realidad (El Desafío)
Para cumplir nuestro objetivo de dar el total, vamos a leer nuestra lista de arriba hacia abajo.

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
2. Reconstruir el estado leyendo la secuencia completa.

Al tener la historia guardada como una **secuencia de eventos**, tenemos el poder de reflejar el estado del sistema en **cualquier momento**. A la estructura donde guardamos estas secuencias para consultarlas después, se le conoce conceptualmente como un **Event Store**.

---

[⬅️ Volver a la sección anterior](./02-primer-proyecto.md)

[➡️ Siguiente sección: El gran problema](./04-el-problema.md)
