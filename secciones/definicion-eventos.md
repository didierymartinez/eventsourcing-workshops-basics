# 🏷️ Definición de eventos

En Event Sourcing, los eventos son la fuente de la verdad. Representan hechos inmutables que ya ocurrieron en tu negocio.

## Ejercicio: Modelando tus primeros eventos

En este primer paso, definirás cómo se ven los eventos de una **Orden de Compra** en código C#.

### 1. ¿Qué debes hacer?

Abre el archivo `Program.cs` de tu proyecto `TallerMarten.OrdenCompra` y borra todo su contenido. Luego, escribe las siguientes definiciones de eventos:

```csharp
using System;

// Definición de eventos como registros (records) inmutables
public record OrdenCreada(Guid Id, string NumeroFactura);
public record ProductoAgregado(string Nombre, int Cantidad, decimal Precio);
```

### 2. Reto: ¡Ve un paso más allá!

El dominio de una **Orden de Compra** tiene más situaciones. Intenta definir otros `records` que representen estos cambios de estado:
- ¿Cómo llamarías al evento cuando se elimina un producto? (ej: `ProductoEliminado`)
- ¿Y cuando la orden se cancela o se marca como enviada?

> Escribe al menos dos eventos más por tu cuenta en `Program.cs`.

### 3. ¿Por qué usamos `record`?

En C#, un `record` es ideal para eventos porque:
- **Inmutabilidad**: Una vez creado, un evento no debe cambiar (el pasado no se puede editar).
- **Simplicidad**: Permiten definir estructuras de datos en una sola línea.
- **Igualdad por valor**: Dos eventos son iguales si sus datos son iguales.

---

> [!IMPORTANT]
> Los eventos siempre deben nombrarse en **pasado** (`Creada`, `Agregado`, `Enviado`) porque representan algo que ya sucedió.

---

[⬅️ Volver a la sección anterior](./configuracion-inicial.md)

[➡️ Siguiente sección: Persistencia de eventos con Marten](./persistencia-eventos-marten.md)
