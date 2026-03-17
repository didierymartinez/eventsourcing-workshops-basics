# 01 - Fundamentos de Event Sourcing

Antes de usar herramientas como Marten, es vital entender qué estamos resolviendo. 

## 1. ¿Qué es Event Sourcing?

En el desarrollo tradicional (CRUD), guardamos el **estado actual** de los datos. Si una orden cambia de "Pendiente" a "Pagada", sobrescribimos el valor en la base de datos. Perdemos el historial de *cómo* llegamos ahí.

En **Event Sourcing**, no guardamos el estado. Guardamos la **secuencia de eventos** (hechos) que ocurrieron. El estado actual es simplemente el resultado de procesar todos esos eventos desde el principio.

### Analogía: Tu cuenta bancaria
- **CRUD:** Solo ves el saldo actual: `$100`.
- **Event Sourcing:** Es el libro mayor (*ledger*) de movimientos:
  1. Depósito de `$200` (+)
  2. Compra de café `$50` (-)
  3. Pago de internet `$50` (-)
  **Saldo actual:** `$100` (calculado).

---

## 2. Ejercicio: Event Sourcing "Nativo" con Listas

Vamos a simular un Event Store usando una simple lista de C#. Esto te ayudará a ver que Event Sourcing es un patrón de diseño, no solo una librería.

### ¿Qué debes hacer?
En tu `Program.cs`, intenta el siguiente flujo:

1. Crea una lista de objetos para guardar los eventos.
2. Agrega algunos eventos manualmente.
3. Recorre la lista para "reconstruir" el estado.

```csharp
using System;
using System.Collections.Generic;

// 1. Nuestros Eventos (Hechos del pasado)
public record OrdenCreada(string Id, string Cliente);
public record ProductoAgregado(string Producto, decimal Precio);

// 2. Simulamos el "Event Store" con una lista nativa
var historialDeEventos = new List<object>();

// Guardamos hechos
historialDeEventos.Add(new OrdenCreada("ORD-001", "Juan Perez"));
historialDeEventos.Add(new ProductoAgregado("Laptop", 1200m));
historialDeEventos.Add(new ProductoAgregado("Mouse", 25m));

// 3. Reconstrucción del Estado (Proyección en memoria)
decimal total = 0;
string cliente = "";

foreach (var evento in historialDeEventos)
{
    if (evento is OrdenCreada e) cliente = e.Cliente;
    if (evento is ProductoAgregado p) total += p.Precio;
}

Console.WriteLine($"Orden de: {cliente}");
Console.WriteLine($"Total acumulado: ${total}");
```

---

### Conclusión del ejercicio
Como puedes ver, **no guardamos el total en ningún lado**. El total se calculó recorriendo la historia. Esto nos da una trazabilidad perfecta.

> [!TIP]
> Si mañana el negocio pregunta: "¿Cuánto dinero gastó el cliente en mouses?", solo tenemos que recorrer la lista buscando el evento específico. Con CRUD, esa información se habría perdido en el total.

---

[➡️ Siguiente sección: Configuración del entorno](./2-configuracion.md)
